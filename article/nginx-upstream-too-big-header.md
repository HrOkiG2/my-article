---
title: "Nginxの「upstream sent too big header」エラーの解決方法(Laravel + PHP-FPM)"
emoji: "📦"
type: "tech"
topics: ["nginx", "laravel", "php", "phpfpm", "ubuntu"]
published: false
published_at: "2025-06-24 00:04"
---

> 初出: 2025年6月24日(旧ブログの記事を移行・加筆)

Nginx + PHP-FPM + Laravelの環境で出る「upstream sent too big header」エラーの原因と、解決方法をまとめました。
ある日突然サイトが表示されなくなって困っている、Laravel開発者・インフラ担当の方に向けた内容です。

## 結論

Nginxの `fastcgi_buffer_size` と `fastcgi_buffers` を大きくすれば、エラーは解消します。
ただし、ヘッダーが大きくなる原因(セッションやCookieの肥大化など)は、Laravel側でも見直しましょう。

## 環境

* Ubuntu 24.04
* PHP 8.2
* Laravel 11.x

## 起こったこと

NginxをWebサーバーにして、LaravelとPHP-FPMを連携している環境で、ある日突然、Nginxのエラーログに次のエラーが出て、サイトが正常に表示されなくなりました。

```
upstream sent too big header while reading response header from upstream
```

調べてみると、このエラーは、**PHP-FPM(アップストリーム)が返そうとしたレスポンスヘッダーのサイズが、Nginxが受け入れられる上限を超えたとき**に発生します。
Laravelのような動的なアプリケーションでは、セッション情報やCookieの肥大化が原因で起きやすいようです。

## エラーが起きる原因

NginxはクライアントからのリクエストをPHP-FPM(FastCGI)に渡して処理してもらいます。
PHP-FPMがNginxに返すレスポンスには、ヘッダー情報が含まれます。
このヘッダー情報が、Nginxのバッファサイズを超えると、エラーになります。

主な原因は、次のようなものです。

* **セッションデータの肥大化**: セッションをCookieベースで管理している場合、セッションに多くの情報を保存したり、カートに大量の商品を入れたりすると、Cookieが大きくなります。
* **多数のCookieやカスタムヘッダー**: Cookieを大量に設定していたり、独自ヘッダーを大量に追加していたりすると、ヘッダー全体が大きくなります。
* **リダイレクトループ**: 不適切なリダイレクト設定で、リダイレクトのたびにCookieやヘッダーが増え、最終的にヘッダーが大きくなることがあります。

## 解決策: Nginxの設定を調整する

一番手軽なのは、Nginxが受け入れられるヘッダーバッファのサイズを増やすことです。

Nginxの設定ファイル(`/etc/nginx/nginx.conf` か、サイト固有の `.conf`)を開いて、`http` ブロックか、対象の `server` ブロックに、次を追加します。
`http` ブロックに書くと、すべてのバーチャルホストに適用されます。

```nginx
http {
    # ...他の設定...

    # upstream sent too big header エラー対策
    fastcgi_buffer_size 128k;   # FastCGIヘッダーバッファの初期サイズ(デフォルトは4k/8k)
    fastcgi_buffers 4 256k;     # 256KBのバッファを4つ(合計1MB)

    # プロキシとして使う場合は、こちらも設定
    # proxy_buffer_size 128k;
    # proxy_buffers 4 256k;
    # proxy_busy_buffers_size 256k;
}
```

### 各ディレクティブの意味

* `fastcgi_buffer_size`: FastCGIからのレスポンスヘッダーを読み込む、最初のバッファのサイズです。デフォルト(通常は `4k` か `8k`)より大きくすると、大きなヘッダーにも対応できます。
* `fastcgi_buffers`: レスポンス全体をバッファリングするための、バッファの数とサイズです。上の例では、256KBのバッファを4つで、合計1MBです。

:::message
値を必要以上に大きくすると、Nginxのメモリ使用量が増えて、サーバーを圧迫します。
まずは、エラーが出なくなる最小限の値から試して、負荷を見ながら調整しましょう。
:::

### 設定変更後の手順

1. 設定ファイルを保存します。
2. 構文エラーがないか、テストします。`test is successful` と出ればOKです。

   ```bash
   sudo nginx -t
   ```

3. Nginxをリロードして、設定を反映します。

   ```bash
   sudo systemctl reload nginx
   ```

これで、Nginxが大きなヘッダーを処理できるようになり、エラーは解消されるはずです。

## 根本的な解決策: Laravel側を最適化する

Nginxの設定でエラーは抑えられますが、**Laravel側で不必要に大きなヘッダーが作られている**なら、そこも見直しましょう。

* **セッションデータの見直し**
  * Cookieに保存しているセッションデータが大きすぎる場合は、DBやRedisなどの**サーバーサイドのストレージにセッションを保存**するよう変更しましょう。
  * Cookieにはセッションidだけが保存されるので、サイズを大幅に減らせます。設定は `config/session.php` で変更できます。
* **Cookieの管理**
  * 設定しているCookieの数や、1つ1つのサイズを減らせないか、確認しましょう。不要なCookieは削除します。
* **カスタムヘッダーの確認**
  * レスポンスに含めているカスタムヘッダーが、多すぎたり、大きすぎたりしないか確認しましょう。

## まとめ

* このエラーは、PHP-FPMが返すレスポンスヘッダーが、Nginxのバッファを超えると発生します。
* `fastcgi_buffer_size` と `fastcgi_buffers` を増やせば、当面のエラーは解消できます。
* 根本対策として、セッションをサーバーサイドに移す、Cookieやヘッダーを減らすなど、Laravel側の見直しも行いましょう。
