---
title: "Laravel+Inertia+ReactをECS Fargateで動かす: NginxとPHP-FPMのコンテナ構成"
emoji: "🐳"
type: "tech"
topics: ["aws", "ecs", "laravel", "nginx", "docker"]
published: false
published_at: "2025-10-28 14:24"
---

> 初出: 2025年10月28日(旧ブログの記事を移行・加筆)

Laravel + Inertia + Reactの構成を、AWSのECS Fargateで動かすときの、NginxとPHP-FPMのコンテナ構成をまとめました。
ローカルのDocker Composeでは動くのに、ECSへのデプロイで迷っている方に向けた内容です。

## 結論

* **1タスク2コンテナ**(Nginx + PHP-FPM)の構成にします。
* ECS Fargateでは、`fastcgi_pass 127.0.0.1:9000;` と指定します。
* ビルドしたJS/CSSは、**Nginxコンテナ**に置きます。
* `manifest.json` は、PHP-FPMコンテナにも必要です。

## fastcgi_pass とは

### NginxとPHP-FPMの橋渡し

Nginxは高性能なWebサーバーですが、PHPのコードは実行できません。
PHPを実行するのは、PHP-FPMというプロセスです。

NginxがPHPファイルへのリクエストを受けたら、PHP-FPMに処理を依頼します。
このとき、**どのPHP-FPMに依頼するかの宛先**を指定するのが、`fastcgi_pass` です。

```nginx
location ~ \.php$ {
    # ...
    # ↓ この1行が「橋渡し」の指定
    fastcgi_pass php-fpm-server:9000;
}
```

## ローカル開発(Docker Compose)の構成

ローカル開発では、「1コンテナ1責務」の原則に従って、NginxとPHP-FPMを別のコンテナにして、`docker-compose.yml` に定義するのが一般的です。

```yaml
# docker-compose.yml (抜粋)
services:
  # Nginxコンテナ
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./laravel-project:/var/www/html

  # PHP-FPMコンテナ
  php:
    build: . # PHP-FPMとLaravelコードを含むDockerfile
    volumes:
      - ./laravel-project:/var/www/html
```

この構成では、NginxコンテナとPHPコンテナは**別のネットワーク空間**にあります。
Docker Composeの内部ネットワークを通じて、サービス名で通信します。
そのため、Nginxからは `php` というサービス名で、PHPコンテナにアクセスします。

```nginx
# nginx.conf (Docker Compose用)
location ~ \.php$ {
    # ...
    # 'php' サービス(コンテナ)の9000番ポートに転送
    fastcgi_pass php:9000;
}
```

コンテナ間通信がDockerの内部ネットワークに限られていれば、TCP/IP通信による性能やセキュリティの懸念は、実用上ほとんど問題になりません。

## 本番(ECS Fargate)の構成: 1タスク2コンテナ

ECSでは、`docker-compose.yml` に相当するものが**タスク定義**です。
大事なポイントは、**1つのタスク定義の中に、NginxコンテナとPHP-FPMコンテナの2つを定義する**ことです。
この2つのコンテナが、1セットで「Webアプリケーションサーバー」として動きます。

### ECS Fargateでの fastcgi_pass

同じタスク内のコンテナは、**同じネットワーク空間を共有**します(ネットワークモード `awsvpc`)。
Nginxコンテナから見ると、PHP-FPMコンテナは「他人」ではなく、**自分自身(localhost)**に見えます。

そのため、ECS Fargate用のNginxイメージの `nginx.conf` は、サービス名ではなく `localhost` を指定します。

```nginx
# nginx.conf (ECS Fargate用)
location ~ \.php$ {
    # ...
    # サービス名ではなく、localhost (127.0.0.1) を指定する
    fastcgi_pass 127.0.0.1:9000;
}
```

### タスク定義のイメージ

* **コンテナA: `php-fpm-container`**
  * イメージ: Laravelコードを含むPHP-FPMイメージ
  * ポートマッピング: **なし**(外部に公開する必要がないためです)
* **コンテナB: `nginx-container`**
  * イメージ: 設定済みの `nginx.conf` と静的ファイルを含むNginxイメージ
  * ポートマッピング: **`80:80`**(ALBからのトラフィックを受けるためです)

ALB(ロードバランサー)からのトラフィックはNginxコンテナが受けて、必要に応じて `localhost:9000` 経由でPHP-FPMコンテナに渡します。

## Inertia/Reactのビルドファイルはどこに置く？

Inertia/React構成では、ここで迷いやすいです。

> InertiaはLaravel(PHP)がビューを配信する仕組みだから、`npm run build` で作られた `public/build` は、PHP-FPMコンテナだけに置けばいいのでは？

これはよくある誤解です。
**パフォーマンスと責務分離の面から、ビルドした静的ファイル(JS/CSS)はNginxコンテナに置くのが正解**です。
理由は、Inertiaのページが読み込まれる、2段階の流れを見ると分かります。

### ステップ1: HTMLのガワをリクエスト(PHP-FPMが処理)

1. ブラウザが `https://example.com/dashboard` にアクセスします。
2. ALB → Nginxコンテナが、リクエストを受けます。
3. NginxはPHPのリクエストだと判断して、`fastcgi_pass 127.0.0.1:9000` で**PHP-FPMコンテナ**に転送します。
4. Laravelが起動して、Inertiaは `app.blade.php` をレンダリングします。
5. このとき、PHPは `public/build/manifest.json` を読んで、HTMLに含めるJS/CSSのファイル名を特定します。
6. PHPは、次のようなHTMLの「ガワ」を生成して、Nginx経由でブラウザに返します。

```html
<html>
<head>
    <script src="/build/assets/app.12345.js" defer></script>
    <link rel="stylesheet" href="/build/assets/app.67890.css">
</head>
<body>
    <div id="app" data-page="..."></div>
</body>
</html>
```

### ステップ2: 静的ファイル(JS/CSS)をリクエスト(Nginxが処理)

1. ブラウザは、受け取ったHTMLを解析します。
2. 「`/build/assets/app.12345.js` と `/build/assets/app.67890.css` が必要だ」と判断して、**別に2回のリクエスト**を送ります。
3. このリクエストは、PHPとは関係ありません。
4. ALB → **Nginxコンテナ**が、この2つのリクエストを受けます。
5. Nginxは、自分の管理下(`public` フォルダ)にファイルがあるかを探します。
6. **Nginxコンテナに `public/build/` 以下のファイルがあれば**、PHP-FPMを起動せずに、静的ファイルを高速に返せます。

もしNginxコンテナにJS/CSSがないと、このリクエストもPHP-FPMに転送されます。
「JSファイルを取得するためだけに、Laravelを起動する」ことになり、深刻なパフォーマンスのボトルネックになります。

## CI/CDでのベストプラクティス

Dockerイメージをビルドするときは、次の構成が最適です。

1. `npm run build` を実行して、`public/build` を生成します。
2. **PHP-FPMイメージのビルド**
   * Laravelのコード(`app/`、`routes/` など)をコピーします。
   * ステップ1のために、`public/build/manifest.json` をコピーします(リンク生成に必要です)。
3. **Nginxイメージのビルド**
   * `nginx.conf` をコピーします。
   * ステップ2のために、`public` フォルダ全体(`index.php` と、`public/build` 以下の**すべてのJS/CSS**)をコピーします。

`manifest.json` はPHP-FPMコンテナに、JS/CSSの実体はNginxコンテナに必要です。
両方のイメージに `public` フォルダ(または `public/build`)全体をコピーするのが、いちばんシンプルで確実です。

## まとめ

1. **構成**: 「1タスク2コンテナ(Nginx + PHP-FPM)」にします。
2. **`fastcgi_pass`**: 同じタスク内なので、`127.0.0.1:9000` を指定します。
3. **静的ファイル**: ビルドしたJS/CSSはNginxコンテナに置いて、Nginxから直接配信します。
4. **`manifest.json`**: HTMLのガワを生成するために、PHP-FPMコンテナにも必要です。

PHP-FPMは動的な処理に専念して、Nginxは静的ファイルの高速配信に専念する。
「1コンテナ1責務」のメリットを活かした、スケーラブルな本番環境が作れます。
