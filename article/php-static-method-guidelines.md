---
title: "PHPのstaticメソッドを使うべき場面と避けるべき場面"
emoji: "📌"
type: "tech"
topics: ["php", "static", "oop", "designpattern"]
published: false
published_at: "2024-09-12 16:50"
---

> 初出: 2024年9月12日(旧ブログの記事を移行・加筆)

PHPの `static` メソッドを使うべき場面と、避けるべき場面をまとめました。
`static` を便利に使いすぎて、「staticおじさん」になりかけているPHPerの方に向けた備忘録です。

## 結論

`static` メソッドは、**状態を持たない処理**にだけ使うのが基本です。
使いすぎると、オブジェクト指向の利点やテストのしやすさが損なわれます。

## staticメソッドとは

`static` メソッドは、インスタンスを生成せずに、クラスから直接呼び出せるメソッドです。

## 使うべきタイミング

### 状態を持たない処理

インスタンスの状態に依存しない、独立した処理に向いています。
たとえば、ユーティリティ関数やヘルパー関数です。

```php
class MathHelper {
    public static function add($a, $b) {
        return $a + $b;
    }
}

echo MathHelper::add(3, 4); // 出力: 7
```

### ファクトリーメソッド

クラスのインスタンスを生成するメソッド(ファクトリーメソッド)として使われることがあります。
`new` を隠して、インスタンス化の過程を制御できます。

```php
class User {
    public $name;

    private function __construct($name) {
        $this->name = $name;
    }

    public static function create($name) {
        return new self($name);
    }
}

$user = User::create('John Doe');
```

個人的には、あまり使いたくないですね。

### シングルトンパターン

特定のクラスのインスタンスが1つだけ存在することを保証する、シングルトンパターンの実装にも使われます。

```php
<?php

class Singleton
{
    private static $instance;

    private function __construct()
    {
        // プライベートコンストラクタ
    }

    public static function getInstance()
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}

$singleton = Singleton::getInstance();
```

## だめな使い方

### インスタンスの状態を操作する処理

インスタンスの状態に依存するメソッドに `static` を使ってはいけません。
`static` メソッドはクラスレベルで動くので、インスタンス固有の状態を持てません。

```php
class User {
    private $name;

    public static function setName($name) {
        $this->name = $name; // エラー: $this は使えない
    }
}
```

### 過度な使用

すべてのメソッドを `static` にするのは避けましょう。
多用すると、オブジェクト指向の基本である**カプセル化**や**多態性**が損なわれます。
特に、あとからメソッドをオーバーライドしたくなったときに、柔軟性を失います。

### 依存性注入を避けてしまう

`static` メソッドは、依存性注入と相性が悪く、テストやメンテナンスが難しくなります。
テストしやすいコードを保つには、インスタンスメソッドを使って、必要な依存関係をコンストラクタやセッターで注入するのが望ましいです。

## まとめ

* `static` は、状態を持たないユーティリティ処理に使います。
* インスタンスの状態に依存する処理に使ってはいけません。
* 多用すると、カプセル化や多態性が損なわれます。
* テストしやすさのために、依存関係はインスタンスメソッドと注入で扱いましょう。
