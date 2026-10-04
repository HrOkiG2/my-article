---
title: "PHPでInterfaceを使う理由: 一貫性・DI・テスト・拡張性"
emoji: "🧩"
type: "tech"
topics: ["php", "interface", "oop", "phpunit", "designpattern"]
published: false
published_at: "2024-07-27 10:51"
---

> 初出: 2024年7月27日(旧ブログの記事を移行・加筆)

PHPでInterfaceを使う理由と、その利点を具体例つきでまとめました。
Interfaceの使いどころがピンとこないPHPerの方に向けた内容です。

## 結論

Interfaceを使うと、コードの一貫性、依存関係の管理、テストのしやすさ、拡張性が向上します。
PHPで保守しやすいコードを書くための、強力なツールです。

## コードの一貫性を保つ

プロジェクト全体で、一貫したメソッドシグネチャを保てます。
複数のクラスが同じ操作をする場合、それぞれに同じメソッドを実装させられます。
読むときに、メソッドの名前や引数の形式に迷わなくなります。

```php
interface LoggerInterface {
    public function log(string $message): void;
}

class FileLogger implements LoggerInterface {
    public function log(string $message): void {
        // ファイルにログを記録する処理
    }
}

class DatabaseLogger implements LoggerInterface {
    public function log(string $message): void {
        // データベースにログを記録する処理
    }
}
```

## 依存関係の注入(DI)を容易にする

特定のクラスではなく、Interfaceに依存させると、依存関係の注入が簡単になります。
将来クラスを変更しても、コード全体への影響を小さくできます。

```php
class UserController {
    private LoggerInterface $logger;

    public function __construct(LoggerInterface $logger) {
        $this->logger = $logger;
    }

    public function createUser(string $name): void {
        // ユーザー作成の処理
        $this->logger->log("User created: $name");
    }
}
```

## テストを容易にする

モックオブジェクトやスタブで、依存関係を簡単に置き換えられるので、ユニットテストが書きやすくなります。

モックができないと、ユニットテストが書きにくい場面がたくさんあります。
正直、そのためだけにInterfaceを使っていると言っても、過言ではありません。

```php
class MockLogger implements LoggerInterface {
    public function log(string $message): void {
        // テスト用のモック処理
    }
}

$logger = new MockLogger();
$controller = new UserController($logger);
$controller->createUser('John Doe');
// ここでログが正しく動作するかをテスト
```

## 拡張性の向上

新しい機能を追加しやすくなります。
たとえば、新しい種類のLoggerを追加するときは、既存のコードを変えずに、新しいクラスを追加するだけで済みます。

```php
class EmailLogger implements LoggerInterface {
    public function log(string $message): void {
        // メールにログを送信する処理
    }
}
```

## まとめ

* Interfaceを使うと、メソッドの形が揃い、コードの一貫性が保てます。
* Interfaceに依存させると、DIがしやすくなります。
* モックに置き換えられるので、テストが書きやすくなります。
* 新しい実装を、既存コードを変えずに追加できます。
