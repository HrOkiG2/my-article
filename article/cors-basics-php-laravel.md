---
title: "CORSの基本と、PHP・Laravelでの設定方法"
emoji: "🌐"
type: "tech"
topics: ["cors", "php", "laravel", "http", "security"]
published: false
published_at: "2024-01-28 15:53"
---

> 初出: 2024年1月28日(旧ブログの記事を移行・加筆)

Web開発でよく聞くけれど、意外とわかりにくいCORS(Cross-Origin Resource Sharing)の基本と、PHP・Laravelでの設定方法をまとめました。
CORSエラーに悩んだ経験のあるWeb開発者の方に向けた内容です。

## 結論

CORSは、「信頼できるオリジンからのリクエストだけを許可する」ための仕組みです。
サーバー側で許可するオリジンを設定し、ブラウザがその設定に従います。

## CORSって何？

CORSは、Webページが**異なるオリジン**(ドメイン、プロトコル、ポートのいずれかが違うもの)のリソースにアクセスするときの、セキュリティの仕組みです。
自分のサイトが、他のサイトのデータを安全に扱うためのルールだと考えてください。

まず、用語を整理します。

| 用語 | 説明 |
| --- | --- |
| ホスト(Host) | インターネット上で情報やリソースを提供するコンピューターやサーバーです。例: ウェブサイトをホストするサーバー |
| ドメイン(Domain) | インターネット上のアドレスの一部です。人間が理解しやすい形でサーバーを識別します。例: `example.com` |
| オリジン(Origin) | スキーム(プロトコル)、ホスト、ポートの組み合わせで定義されます。例: `https://example.com:443` |

## なぜCORSが必要なの？

Webはオープンにつながっていますが、同時にセキュリティも重要です。
たとえば、ログインしているサイトAのデータを、悪意のあるサイトBが盗もうとするリスクがあります。
CORSは、こうした不正なアクセスを防ぐために存在します。

## CORSの仕組み

基本は、**信頼できるオリジンからのリクエストのみを許可する**ことです。
サーバー側で設定して、ブラウザがその設定に従います。

1. リクエストが来たときに、サーバーが「このオリジンはOKか」をチェックします。
2. OKならリソースを提供し、ダメなら拒否します。

### 簡単な例

自分のサイト(`example.com`)が、別のサイト(`api.example.com`)からデータを取得したいとします。

* `api.example.com` が、CORSで `example.com` を許可していれば、問題なくやり取りできます。
* 許可されていなければ、「CORSエラー」が発生してアクセスできません。

### CORSをどう扱う？

開発者にとってCORSは頭痛の種ですが、適切に設定すれば、セキュリティを保ちつつデータ交換もできます。
サーバーの設定を見直したり、必要に応じて、許可するオリジンを追加したりしましょう。

## 設定方法

### 素のPHP

Webサーバーで設定することもできますが、ここではPHPでの実装を想定します。

```php
header("Access-Control-Allow-Origin: http://example.com");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, X-Requested-With");
```

この設定の意味は、次のとおりです。

* `http://example.com` からのアクセスを許可します。
* 許可するメソッドは、GET、POST、PUT、DELETEです。
* リクエストで許可するHTTPヘッダーは、次の2つです。
  * `Content-Type`(リソースのメディアタイプ)
  * `X-Requested-With`(通常は、Ajaxリクエストを識別するために使います)

### Laravel

Laravel 7.x以降では、デフォルトでCORSの設定が組み込まれています。
設定ファイルは `config/cors.php` です。

```php
return [
    'paths' => ['api/*'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['http://example.com'],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => false,
];
```

* `allowed_origins`: `http://example.com` からのリクエストを許可します。
* `allowed_methods`: `['*']` で、すべてのHTTPメソッドを許可します。特定のメソッドだけ許可したい場合は、`['GET', 'POST', 'PUT', 'DELETE']` のように指定します。
* `allowed_headers`: `['*']` で、すべてのHTTPヘッダーを許可します。特定のヘッダーだけ許可したい場合は、必要なヘッダーを配列に追加します。

## まとめ

* CORSは、異なるオリジンへのアクセスを制御する、Webの安全性を保つ仕組みです。
* 許可するオリジンは、サーバー側で設定します。
* PHPでは `header()` で、Laravelでは `config/cors.php` で設定します。
* 最初は複雑に感じますが、基本を理解すれば、安全で効果的な開発ができます。
