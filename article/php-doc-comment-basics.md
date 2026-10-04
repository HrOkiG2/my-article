---
title: "PHPのDocコメントの基本: 書き方・型・タグ一覧"
emoji: "📝"
type: "tech"
topics: ["php", "phpdoc", "phpstan", "comment"]
published: false
published_at: "2024-01-15 23:08"
---

> 初出: 2024年1月15日(旧ブログの記事を移行・加筆)

PHPのDocコメント(PHPDoc)の書き方と、よく使う型・タグをまとめました。
PHPer は技術者のレベル差が大きいと言われがちなので、基礎くらいは押さえておきたい、という方に向けた内容です。

## 結論

Docコメントは、可読性と静的解析ツールの精度を上げてくれます。
書くのは少し面倒ですが、メリットのほうが大きいので、きちんと書いておきましょう。

## Docコメントのメリット

### 可読性・視認性が高くなる

何をするのか、どんなデータを持っているのかが一目で分かります。
保守開発のときに、大いに役立ちます。

### ツールに影響する

PHPStanやPHPUnitなどのツールを使うときに、大きく役立ちます。
特に静的解析の精度が上がって、バグの早期発見にとても貢献してくれます。

## Docコメントのデメリット

### 面倒くさい

書くのが、猛烈に面倒な日もあります。
それでもメリットを理解しているので、そんな日もちゃんと書きます。みなさんも書きましょう。

### 内容が不適切だとつらい

書いてある内容が間違っていると、誤解を生んで、新たな仕事が発生することがあります。
最近、ある大手サービスのSDKのDocコメントが間違っていて、IDE上の警告が消えず、時間を使ってしまうことがありました。
(おそらく、Docコメントがメンテナンスされていなかっただけだと思います。)

## 書き方

メソッドとプロパティの例です。

```php
/**
 * 何をするメソッドなのか
 *
 * @param 型 $変数 説明
 * @return 戻り値の型 説明
 */
private function fuga(): void {}

/**
 * プロパティの説明
 *
 * @var string
 */
protected string $hogehoge;
```

## 型定義

| 型 | 説明 |
| --- | --- |
| string | 文字列 |
| integer / int | 整数 |
| boolean / bool | 真偽値 |
| float / double | 浮動小数点数 |
| object | 任意のクラスのインスタンス |
| mixed | 任意の型(不特定) |
| array | 配列 |
| resource | リソース型 |
| void | 値を返さない |
| null | NULL値 |
| callable | コールバック関数 |
| false / true | 真または偽 |
| self | 同じクラスのインスタンス |
| scalar | スカラー型(string, int, boolなど) |

参考: https://docs.phpdoc.org/3.0/guide/references/phpdoc/types.html

## タグの意味

| タグ | 意味 | 用法の例 |
| --- | --- | --- |
| `@param` | パラメータを記述します。メソッドのDocコメントで使います。 | `@param int $age 年齢` |
| `@return` | 戻り値を記述します。 | `@return string ユーザーの名前を返します` |
| `@var` | 変数の型を記述します。プロパティのDocコメントで使います。 | `@var string $name ユーザーの名前` |
| `@throws` | 例外を記述します。 | `@throws InvalidArgumentException 不正な引数の場合` |
| `@deprecated` | 非推奨を示します。 | `@deprecated 2.0.0バージョン以降で非推奨` |
| `@see` | 関連要素を参照します。 | `@see User::getName() 関連するメソッド` |
| `@author` | 作者名です。 | `@author 山田太郎` |
| `@since` | 導入されたバージョンです。 | `@since 1.0.0 初期バージョンから導入` |
| `@version` | バージョンです。 | `@version 1.2.3` |
| `@link` | 関連リンクです。 | `@link http://example.com 詳細情報` |
| `@example` | 例示です。 | `@example /path/to/example.php 例示コード` |
| `@category` | カテゴリ分類です。 | `@category HTTP クライアント関連` |

参考: https://docs.phpdoc.org/3.0/guide/references/phpdoc/index.html

## まとめ

* Docコメントは、可読性が上がり、PHPStanなどの静的解析の精度も上がります。
* 内容が間違っていると逆効果なので、メンテナンスが大切です。
* よく使うのは `@param` / `@return` / `@var` / `@throws` あたりです。
