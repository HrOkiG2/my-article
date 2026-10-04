---
title: "Apacheで特定ディレクトリへのアクセスを禁止する方法"
emoji: "🚫"
type: "tech"
topics: ["apache", "htaccess", "security", "php"]
published: false
published_at: "2023-01-18 19:43"
---

> 初出: 2023年1月18日(旧ブログの記事を移行・加筆)

Apacheで、特定のディレクトリに直接アクセスできないようにする方法をまとめました。
LAMP環境でちょっとしたセキュリティ対策をしたい方に向けた内容です。

## 結論

アクセス制限の方法は2つあります。

* `.htaccess` でディレクトリごとに制御する方法
* Apacheの設定ファイル(`.conf`)の `<Directory>` で制御する方法

基本的には、サーバー側の設定ファイルで管理するほうがおすすめです。

## .htaccessでアクセス管理する場合

### .htaccessとは

`.htaccess` は、Apache環境でディレクトリ単位の設定を書くファイルです。

* NGINXでは使えません。
* 基本的には、サーバー側の `.conf` で管理したほうが良いです。

### .htaccessをサーバーで有効にする

Ubuntuの場合は、`/etc/apache2/apache2.conf` を編集します。

```apache
<Directory "/var/www/test.com">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

やりたいことは次の2つです。

* `Directory` で、`.htaccess` を有効にしたいディレクトリを指定します。基本的には、ドメイン配下のディレクトリになると思います。
* `AllowOverride All` で有効化します。デフォルトは `AllowOverride None`(無効)です。

### 設定を書いた.htaccessを置く

アクセスを禁止したいディレクトリに、設定を書いた `.htaccess` を置きます。
たとえば、`test.com/test/` への直接アクセスをすべて禁止するには、次のように書きます。

```apache
Require all denied
```

ディレクトリ構成は、次のようになります。

```
test.com
└── test
    ├── .htaccess
    └── index.php
```

特定のIPだけアクセスを許可する場合は、こう書きます。

```apache
Require ip 指定のIP
Require ip 指定のIP
```

:::message
Apache 2.2までは `order deny,allow` / `deny from all` / `allow from IP` という書き方でした。
Apache 2.4では `Require` ディレクティブを使います。
バージョンによって書き方が違うので、使っている環境に合わせて調べてみてください。
:::

## 設定ファイル(.conf)で制御する場合

`/etc/apache2/apache2.conf`(または、サイトごとの `.conf`)に次を追記します。

```apache
<Directory /var/www/test.com/test/test>
    Require all denied
</Directory>
```

この設定をすると、`https://test.com/test/test` に直接アクセスしたときに **403 Forbidden** が返ります。

## まとめ

* ディレクトリごとのアクセス制限は、`.htaccess` か `.conf` の `<Directory>` で設定できます。
* `.htaccess` を使うには、`AllowOverride All` で有効化が必要です。
* 基本は、サーバー側の `.conf` で管理しましょう。
* Apache 2.4では `Require all denied` で禁止、`Require ip` で特定IPのみ許可できます。
