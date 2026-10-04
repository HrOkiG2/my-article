---
title: "SOLID原則の依存性逆転の原則(DIP)をPHPのInterfaceで理解する"
emoji: "🔄"
type: "tech"
topics: ["php", "solid", "interface", "cleanarchitecture", "design"]
published: false
published_at: "2023-06-06 00:19"
---

> 初出: 2023年6月6日(旧ブログの記事を移行・加筆)

SOLID原則の依存性逆転の原則(DIP)を、PHPのInterfaceを使って整理しました。
設計を学んでいる、PHPエンジニアの方に向けた内容です。(前回は、単一責任の原則を学びました。)

## 結論

**呼び出し元は、具体的なクラスではなく、Interface(抽象)に依存させる**のが、依存性逆転の原則です。
これで、クラス同士が疎結合になり、拡張しやすくなります。

## まずはInterfaceから

PHPのInterfaceは、クラスが実装するべきメソッドのシグネチャ(メソッド名、引数、戻り値の型)を、定義するための機能です。
Interfaceは抽象的な定義で、実際の実装は持ちません。(「じゃあ、いらないのでは?」と思ってしまいますよね。)

なぜ必要なのかというと、**クラスがInterfaceを実装すると、そのInterfaceで定義されたメソッドを、必ず実装しなければいけないから**です。

たとえば、ユーザーにメッセージを送る場面を考えてください。
どんなメッセンジャーを使うにしても、「**メッセージを生成する機能**」と「**メッセージを送信する機能**」の2つが必要です。
ここでは、メールで送る場合(Mailクラス)と、LINEで送る場合(Lineクラス)を想定します。

```php
interface Message {
    public function createMessage();
    public function sendMessage();
}
```

このように定義すると、MailクラスとLineクラスには、必ず同じメソッドがあることが、保証されます。(**クラスが、同じ振る舞いを持つことが保証される**ということです。)

```php
class Mail implements Message {
    public function createMessage() {
        // 処理
    }

    public function sendMessage() {
        // 処理
    }
}

class Line implements Message {
    public function createMessage() {
        // 処理
    }

    public function sendMessage() {
        // 処理
    }
}
```

もしSlackでメッセージを送りたくなったら、Slackクラスに、同じメソッドを実装するだけで済みます。
既存のクラス(Mail、Line)に影響を与えずに実装できるので、拡張性が高くなります。

## 依存性逆転の原則

呼び出し元の観点で、考えてみましょう。
Interfaceを実装すると、呼び出し元のコードは、**具体的な実装クラスではなく、Interfaceに依存**します。
そのため、コードの結合が低くなります(疎結合)。

先ほどのコードで言えば、呼び出し元は、Mailクラスを呼んでいるつもりでも、実態は、Interfaceを経由しています。

```php
class Notifier {
    public function __construct(private Message $message) {}

    public function notify() {
        $this->message->createMessage();
        $this->message->sendMessage();
    }
}

$notifier = new Notifier(new Mail());  // Line や Slack に差し替えても、Notifier は変更不要
```

Interfaceを使うと、クラス間の依存関係を抽象化して、実装ではなく、Interfaceに依存させられます。

* 呼び出し元が見ているのは、Interfaceです。
* Interfaceは、「定義する場所」です。
* Interfaceを実装したクラスには、必ず同じメソッドがあります。
* 結果として、呼び出し元と実装クラスの、**どちらもInterfaceに依存**します。

これで、依存性逆転の原則が達成されます。

## まとめ

* Interfaceは、実装クラスが必ず持つメソッドを定義します。
* 呼び出し元は、実装クラスではなく、Interfaceに依存させます。
* これで疎結合になり、新しい実装を、既存コードに影響なく追加できます。
