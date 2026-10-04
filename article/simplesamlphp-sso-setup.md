---
title: "SimpleSAMLphpでSAML認証のSSOを構築する手順メモ"
emoji: "🔑"
type: "tech"
topics: ["php", "simplesamlphp", "saml", "sso", "apache"]
published: false
published_at: "2023-02-17 11:30"
---

> 初出: 2023年2月17日(旧ブログの記事を移行・加筆)

SimpleSAMLphpを使って、SAML認証方式のSSO(シングルサインオン)を構築したときの手順をまとめました。
PHPでSSOを実装したい方や、IdPとSPの設定の流れをざっくり掴みたい方に向けた備忘録です。

## 結論

SimpleSAMLphpでのSSO構築は、次の流れで進めます。

1. ApacheにSimpleSAMLphpのエイリアスを設定します。
2. SP(新規)を `authsources.php` に登録します。
3. IdPとSPで、お互いのメタデータを登録します。
4. PHPファイルから呼び出して、動作を確認します。

## SSOとは

SSOは Single Sign-On の略で、**1回のログインで複数のシステムにログインできる**仕組みです。
GoogleアカウントやTwitterアカウントで、別のサービスにログインできるものをイメージすると分かりやすいと思います。

### 認証方式

SSOの認証方式には、次のようなものがあります。

* 代行認証方式
* リバースプロキシ方式
* エージェント方式
* SAML認証方式

今回はSimpleSAMLphpを使うので、**SAML認証方式**です。

### 構成要素

SAML認証は、IdPとSPという2つのシステムで構成されます。

| 名称 | 役割 |
| --- | --- |
| IdP | 大元のシステムに組み込みます。ログイン認証するユーザーの情報や、SPのメタデータを保持します。 |
| SP | 各システムに組み込みます。IdPのメタデータを保持します。 |

この2つのシステム間で「アサーション」という証明書をやり取りして、SSOを実現しています。
アサーションは、ログインの証跡のようなものです。

## SimpleSAMLphpの実装手順

### Apacheのエイリアスを設定する

SSL設定時に生成した、`/etc/apache2/sites-available/ドメイン名.conf` に次を追記します。

```apache
# ...略...
Alias /simplesaml /var/www/simplesamlphp/www
<Directory /var/www/simplesamlphp/www>
    Require all granted
</Directory>
# ...略...
```

設定が済んだら、SimpleSAMLphpのコントロールパネルにアクセスしましょう。
メタデータの確認や、SimpleSAMLphpの設定ができます。

#### セキュリティ面の注意

このエイリアス設定で、コントロールパネルがWeb上に公開されます。
IdPとSPの両方にメタデータが必要な認証方式なので、仮にメタデータが漏れても大きな問題にはなりにくいと考えています。
(SimpleSAMLphp側で、ログインIDとパスワードも設定できます。)

とはいえ、不必要な情報を公開する意味はありません。
**コントロールパネルには、開発者のIPだけに絞るなどのアクセス制限を入れておきましょう。**

#### 管理者パスワードを変更する

`simplesamlphp/config/config.php` の `auth.adminpassword` を、推測されにくい複雑な値に変更してください。
この設定に加えて、サーバー自体にDoS対策を入れておけば、ひとまず安全です。

### SPを新規登録する

`config/authsources.php` の `default-sp` をコピーして、新しいSP(名前は任意)として登録します。

```php
'test-sp' => [ // ←任意で変更(SP名を指定)
    'saml:SP',

    // The entity ID of this SP.
    // Can be NULL/unset, in which case an entity ID is generated based on the metadata URL.
    'entityID' => null,

    // The entity ID of the IdP this SP should contact.
    // Can be NULL/unset, in which case the user will be shown a list of available IdPs.
    'idp' => 'https://test.com/simplesaml/saml2/idp/metadata.php', // ←任意で変更(IdPのエンドポイントを指定)

    // The URL to the discovery service.
    // Can be NULL/unset, in which case a builtin discovery service will be used.
    'discoURL' => null,

    /*
     * The attributes parameter must contain an array of desired attributes by the SP.
     * ...(略)...
     */
],
```

登録したら、コントロールパネルからSPのメタデータ(XML形式)をダウンロードします。
IdPがあるサーバーは、基本的にSPとは別なので、**ダウンロードしたSPメタデータをIdP側のサーバーに登録**します。

### メタデータを相手側に登録する

次のURLから、メタデータのパース済みデータを取得します。

```
https://test.com/simplesaml/module.php/core/frontpage_federation.php
```

管理画面には、XMLをPHPの配列に変換してくれるツールがあります。
取得した内容を `metadata/saml20-idp-remote.php` に貼り付けましょう。

```php
$metadata['https://test.com/simplesaml/module.php/saml/sp/metadata.php/test-sp'] = [
    'SingleLogoutService' => [
        [
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect',
            'Location' => 'https://test.com/simplesaml/module.php/saml/sp/saml2-logout.php/test-sp',
        ],
    ],
    'AssertionConsumerService' => [
        [
            'index' => 0,
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST',
            'Location' => 'https://test.com/simplesaml/module.php/saml/sp/saml2-acs.php/test-sp',
        ],
        [
            'index' => 1,
            'Binding' => 'urn:oasis:names:tc:SAML:1.0:profiles:browser-post',
            'Location' => 'https://test.com/simplesaml/module.php/saml/sp/saml1-acs.php/test-sp',
        ],
        [
            'index' => 2,
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Artifact',
            'Location' => 'https://test.com/simplesaml/module.php/saml/sp/saml2-acs.php/test-sp',
        ],
        [
            'index' => 3,
            'Binding' => 'urn:oasis:names:tc:SAML:1.0:profiles:artifact-01',
            'Location' => 'https://test.com/simplesaml/module.php/saml/sp/saml1-acs.php/test-sp/artifact',
        ],
    ],
    'contacts' => [
        [
            'emailAddress' => 'test@gmail.com', // 管理者用メールアドレス
            'contactType' => '',                // 管理者用
            'givenName' => 'admin',
        ],
    ],
];
```

### 動作を確認する

任意のPHPファイルを作って、SimpleSAMLphpを呼び出します。

```php
<?php
require_once('./../../../simplesamlphp/lib/_autoload.php');

$auth = new SimpleSAML_Auth_Simple('test-sp');
$auth->requireAuth();
$attributes = $auth->getAttributes();

var_dump($attributes);
?>
```

ブラウザの開発者ツールでネットワークタブを開いた状態でアクセスしてみてください。
IdPサーバーとSPサーバーの間で、リダイレクトが起きる様子が確認できます。

最終的にアサーションの取得に成功すると、デバッグ出力した `$attributes` の値がブラウザに表示されるはずです。

## まとめ

* SSOは、1回のログインで複数のシステムを使える仕組みです。
* SimpleSAMLphpは、SAML認証方式のIdP / SPとして使えます。
* コントロールパネルは公開されるので、IP制限と管理者パスワードの変更を忘れずに行いましょう。
* IdPとSPでメタデータを相互に登録して、アサーションのやり取りができれば完成です。
