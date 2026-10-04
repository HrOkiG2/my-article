---
title: "SOLID原則の単一責任の原則(SRP)をPHPのコードで理解する"
emoji: "🧱"
type: "tech"
topics: ["php", "solid", "cleanarchitecture", "design"]
published: false
published_at: "2023-05-28 22:51"
---

> 初出: 2023年5月28日(旧ブログの記事を移行・加筆)

SOLID原則の1つ目、単一責任の原則(SRP)を、PHPのコードで整理しました。
設計の知識を増やしたい、PHPエンジニアの方に向けた内容です。

## 結論

**クラスは、単一の責務だけを持つべき**です。
1つのクラスが、1つの目的だけに集中するように設計します。

## SOLID原則とは

先輩から、クリーンアーキテクチャを特集した雑誌(Software Design 2023年6月号)を渡されたのが、学習のきっかけです。
SOLID原則は、次の5つです。

1. 単一責任の原則(Single Responsibility Principle, SRP)
2. オープン/クローズドの原則(Open/Closed Principle, OCP)
3. リスコフの置換原則(Liskov Substitution Principle, LSP)
4. インターフェース分離の原則(Interface Segregation Principle, ISP)
5. 依存性逆転の原則(Dependency Inversion Principle, DIP)

## 単一責任の原則

SRP(Single Responsibility Principle)は、一言で言うと、「クラスは、単一の責務を持つべき」という原則です。
もっと噛み砕くと、**1つのクラスが、1つの目的や責務にだけ集中する**ことを指します。

### 良くない設計

```php
<?php
class DB {
    private $host;
    private $dbName;
    private $userName;
    private $password;

    public function __construct() {
        $this->host = env('host');
        $this->dbName = env('dbName');
        $this->userName = env('userName');
        $this->password = env('password');

        try {
            $database_handler = new PDO(
                'mysql:host=' . $this->host . ';port=3306;dbname=' . $this->dbName . ';charset=utf8mb4',
                $this->userName,
                $this->password
            );
        } catch (PDOException $e) {
            echo $e->getMessage() . "\n";
            exit;
        }
    }

    public static function selectUser() {
        $db = new self();
        // 取得処理
    }

    public static function editAdmin() {
        $db = new self();
        // 編集処理
    }
}
```

このコードは、`DB` クラスに、「DB接続」「ユーザーの取得処理」「管理者の編集処理」を、まとめて書いています。
理想のクラス設計では、DB接続のクラス、ユーザーのクラス、管理者のクラスに分けるべきです。

### 改善後

**DB接続クラス**

```php
class DB {
    private $host;
    private $dbName;
    private $userName;
    private $password;

    public function __construct() {
        $this->host = env('host');
        $this->dbName = env('dbName');
        $this->userName = env('userName');
        $this->password = env('password');
        // PDOの接続処理
    }
}
```

**ユーザークラス**(管理者クラスも同様です)

```php
class User {
    public static function selectUser() {
        // 取得処理
    }
    // 他のユーザーに関する処理を書いていく
}
```

## 振り返り

「良くない設計」として出したコードは、以前の現場で書いていた手法です。(笑)
`DB` クラスの内容が1,000行以上になっていて、拡張性や保守性がかなり悪いことが、単一責任の原則を学んで、理解できました。
以前の現場のコードは、いつか爆発することが目に見えていましたが、爆発する前に離れることができました。

## まとめ

* 単一責任の原則は、「クラスは単一の責務を持つ」という原則です。
* DB接続、ユーザー処理、管理者処理は、別のクラスに分けましょう。
* 責務を分けると、拡張性と保守性が上がります。
