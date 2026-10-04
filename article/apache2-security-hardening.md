---
title: "Apache2で最低限やっておきたいセキュリティ対策まとめ"
emoji: "🔐"
type: "tech"
topics: ["apache", "security", "ubuntu", "lamp", "php"]
published: false
published_at: "2023-04-04 08:50"
---

> 初出: 2023年4月4日(旧ブログの記事を移行・加筆)

Apache2で最低限やっておきたいセキュリティ対策を、設定例つきでまとめました。
自分でLAMP環境を構築して公開する方に向けた内容です。

## 結論

次の対策を、Apacheの設定ファイルに入れておきましょう。

* クリックジャッキング: `X-Frame-Options`
* XSS: アプリ側のエスケープ + CSP
* DoS/DDoS: `mod_evasive`
* 画像などの直リンク: リファラによる制限
* バージョン情報の非表示: `ServerTokens` / `expose_php`

## Apache2の設定ファイルについて

まず、どのファイルに書くのかを整理しておきます。

* `/etc/apache2/apache2.conf` は、Apacheの導入後に自動生成されるファイルです。
* `/etc/apache2/sites-available/ドメイン名.conf` は、SSL化などのときに自分で作るサイト用のファイルです。

違いは、次のとおりです。

* 全般的な設定は `apache2.conf` に書きます。
* サイトごとの設定は、サイト用の設定ファイルに書きます。

基本的には1サーバー1ドメインになることが多いので、自動生成ファイルに書くケースが多いと思います。
私のようにSSL化でサイト用ファイルを作った場合は、そちらに書いてもOKです。

## エラーが出る場合

これから紹介する設定を書いたときに、次のエラーが出ることがあります。

```
Invalid command 'Header', perhaps misspelled or defined by a module not included in the server configuration
```

これは、ApacheでHTTPヘッダーの操作が許可されていない、という意味です。
次のコマンドで、`headers` モジュールを有効にしましょう。

```bash
sudo a2enmod headers
sudo systemctl restart apache2
```

## クリックジャッキング

### どんな攻撃か

悪意のあるWebページや広告をユーザーが踏むことで、意図しない操作をさせられる攻撃です。
`iframe` やリンクなどを使って、既存のコンテンツの上に透明な悪意あるコンテンツを重ねます。
ユーザーは普通のボタンを押しただけなのに、個人情報が盗まれてしまうことがあります。

### 対策

HTTPヘッダーの `X-Frame-Options` で対策します。
これは、ブラウザに「埋め込みを許可するかどうか」を伝える設定です。

* `DENY`: どのサイトからも埋め込めないようにします。
* `SAMEORIGIN`: 同じオリジン(ドメイン・プロトコル・ポート番号が同じ)のサイトからのみ埋め込めるようにします。
* `ALLOW-FROM`: 特定のURLからのみ埋め込めるようにします。ただし、今は非推奨です。

ロードバランサーなどで複数サーバーを運用することも多いので、`SAMEORIGIN` を選ぶケースが多いと思います。

Apacheでは、設定ファイルに次のように追記します。

```apache
Header always set X-Frame-Options "DENY"
```

HTMLに書く場合は、`meta` タグでも指定できます。
ただし、WEBサーバー側での設定が推奨です。

```html
<meta http-equiv="X-FRAME-OPTIONS" content="DENY" />
```

## XSS

### どんな攻撃か

悪意のあるスクリプトを注入して、ユーザーのブラウザで実行させる攻撃です。
CookieやセッションID、個人情報を盗まれたり、偽のページを表示されたりします。
たとえば、GETパラメータに悪意のあるリンクを入れるなどで実現されます。

### 考え方

基本的には、**アプリケーション側で、文字列のエスケープやサニタイズをする**のが大前提です。

### Apacheでの対策

ブラウザのXSSフィルター機能を有効にします。
この機能は、リクエストパラメータやHTMLタグなどの入力値を検査して、悪意のあるスクリプトがあれば削除・エスケープします。

```apache
Header set X-XSS-Protection "1; mode=block"
```

:::message
`X-XSS-Protection` は、最近のブラウザではすでに廃止されています。
今はCSP(次の項目)での対策が主流なので、あくまで古いブラウザ向けの補助だと考えてください。
:::

## CSP

### CSPとは

CSP(Content Security Policy)は、Webページに埋め込まれる外部リソースの扱いを制限する仕組みです。

### 対策

```apache
Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; object-src 'none'; frame-ancestors 'none'"
```

### unsafe-eval のエラーが出たとき

CSPを設定したあとに、次のエラーが出ることがあります。

```
Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' 'unsafe-inline'".
```

この場合、`script-src` に `'unsafe-eval'` を追加すれば動きます。
ただし、`eval` は文字列をJavaScriptとして実行できるため、**推奨されません。**

```js
// 文字列がそのままコードとして実行されてしまう
eval("alert('aiueo')");
```

追加するなら、ほかの対策も組み合わせましょう。
何重にも対策しておけば、ある程度は担保できます。

* XSSフィルター(サーバー側)
* 入力データの検証(アプリケーション側)

## DoS / DDoS

### どんな攻撃か

システムに大量のリクエストを送って、正常なリクエストを失敗させ、サービスを使えなくする攻撃です。

### 対策

`mod_evasive` というモジュールを使います。
Linuxのディストリビューションやバージョンによって、設定を書くファイルが異なります。

```apache
<IfModule mod_evasive20.c>
   DOSHashTableSize 3097
   DOSPageCount 10
   DOSSiteCount 50
   DOSPageInterval 1
   DOSSiteInterval 1
   DOSBlockingPeriod 10
   DOSLogDir "/var/log/mod_evasive"
   DOSEmailNotify admin@yourdomain.com
   DOSWhitelist 127.0.0.1
</IfModule>
```

## 画像の直リンク

### 直リンクされると困ること

案件によっては、画像やPDFに情報を詰めたデータを扱うことがあります(保険の明細や履歴書など)。
こうしたデータをバイナリ化してDBに入れず、特定のディレクトリに保管する場合、そのディレクトリに直接アクセスされると個人情報が公開されてしまいます。

### 対策

リファラを使って、直アクセスを防ぎます。

```apache
RewriteEngine On
RewriteCond %{HTTP_REFERER} !^$
RewriteCond %{HTTP_REFERER} !^http(s)?://(www\.)?ドメイン.com [NC]
RewriteRule \.(jpg|jpeg|png|gif)$ - [NC,F,L]
```

:::message
リファラは偽装できるため、これだけでは個人情報の保護として十分ではありません。
重要なデータは、公開ディレクトリに置かず、アプリ側で認証したうえで配信しましょう。
:::

`.htaccess` を使ったアクセス禁止の方法もあります。
詳しくは、別記事の「Apacheで特定ディレクトリへのアクセスを禁止する方法」を見てください。

## バージョン情報の非表示

バージョンが見えると、そのバージョンに合わせた攻撃や、セキュリティホールを突く攻撃をされるおそれがあります。
サーバーでコマンドを打てばバージョンは確認できてしまうので、外部には非表示にしましょう。

### Apacheのバージョンを非表示にする

```apache
ServerTokens Prod
ServerSignature Off
```

### PHPのバージョンを非表示にする

`/etc/php/{PHPのバージョン}/apache2/php.ini` を編集します。

```ini
expose_php = Off
```

## まとめ

* ヘッダー系の設定を使うときは、`a2enmod headers` でモジュールを有効にしておきます。
* クリックジャッキングは `X-Frame-Options`、XSSはアプリ側の対策とCSPで防ぎます。
* DoS対策には `mod_evasive`、画像の直リンクにはリファラ制限が使えます。
* ApacheとPHPのバージョン情報は、外に出さないようにしましょう。
* セキュリティ対策は、1つに頼らず、何重にも組み合わせるのが大切です。
