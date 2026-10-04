---
title: "CakePHPのCommandクラスでHTTPSのURLを生成する方法"
emoji: "🍰"
type: "tech"
topics: ["cakephp", "php", "https", "routing"]
published: false
published_at: "2024-04-20 19:07"
---

> 初出: 2024年4月20日(旧ブログの記事を移行・加筆)

CakePHPのCommandクラスで、HTTPSのURLを生成しようとしてハマったので、その解消方法を備忘録としてまとめました。
CakePHP 3から4へ移行したり、マニュアルどおりに動かなかったりして困っている方に向けた内容です。

## 結論

私の環境(CakePHP 4.4)では、マニュアルの `_https` ではHTTPSにならず、`_ssl` を指定すると、HTTPSのURLを生成できました。

## 環境

* CakePHP 4.4
* PHP 8.1

## やりたいこと

Commandクラスから、メールに添付するURLを生成したいです。
そのURLは、HTTPS化されたものにしたいです。

## 困ったこと

ControllerやViewでURLを生成するときは、現在のoriginを参照してくれます。
ですが、Commandクラスなどoriginを参照できない場所では、**オプションを明示的に指定する**必要があります。

CakePHP 4のマニュアルには、次のように書かれています。

> `_https` を `true` にすると、普通のURLから https に変換します。`false` にすると、強制的に http になります。
>
> (出典: [URL の生成](https://book.cakephp.org/4/ja/development/routing.html#id25))

マニュアルのとおりに書きました。

```php
$routes->url([
    'controller' => 'コントローラー名',
    'action' => 'アクション名',
    '_https' => true,
]);
```

これだけでHTTPSになるなら便利だと思ったのですが、HTTPSになりませんでした。
Routerクラスのインポートやスペルミスを確認しても、変わりません。

## 原因を探る

どうしてもHTTPS化が必須だったので、CakePHP 3のマニュアルも読んでみました。
すると、**Cake 3とCake 4で、オプションの名前が違う**ことが分かりました。

> `_ssl` を `true` にすると、普通のURLから https に変換します。`false` にすると、強制的に http になります。
>
> (出典: [URL の生成(CakePHP 3)](https://book.cakephp.org/3/ja/development/routing.html#id23))

## 解決方法

次のように `_ssl` を指定したところ、目的のHTTPSのURLを生成できました。

```php
$routes->url([
    'controller' => 'Articles',
    'action' => 'index',
    '_ssl' => true,
]);
```

:::message
CakePHPのバージョンやURL生成の方法によって、有効なオプションが異なる可能性があります。
うまくいかないときは、使っているバージョンのマニュアルとあわせて、`_ssl` と `_https` の両方を試してみてください。
:::

## まとめ

* Commandクラスなどoriginを参照できない場所では、URL生成のオプションを明示する必要があります。
* マニュアルのとおりに書いても動かないときは、旧バージョンのオプション名も試しましょう。
* 私の環境(CakePHP 4.4)では、`_ssl => true` でHTTPSになりました。
* 枯れた技術を使うのは、ちょっと手間がかかりますね。
