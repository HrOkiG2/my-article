---
title: "PHPのスプレッド構文の使い方と使いどころ"
emoji: "🌿"
type: "tech"
topics: ["php", "spread", "array", "javascript"]
published: false
published_at: "2023-05-28 12:15"
---

> 初出: 2023年5月28日(旧ブログの記事を移行・加筆)

PHPのスプレッド構文(`...`)の基本と、使いどころをまとめました。
`array_merge()` で配列を結合している方や、JavaScriptのスプレッド構文と何が違うのか気になる方に向けた内容です。

## 結論

スプレッド構文を使うと、配列の要素を展開して、別の配列や関数の引数に組み込めます。
配列の結合は `array_merge()` の代わりに `[...$a, ...$b]` で書けるので、今後は `array_merge()` を使う機会が減りそうです。

## スプレッド構文とは

配列の要素を展開するための構文です。
配列を、より簡潔で柔軟に操作できます。

```php
$test = [1, 2, 3];
$testUnion = [...$test, 4, 5, 6];

var_dump($testUnion);
// [1, 2, 3, 4, 5, 6]
```

文字列キーを持つ連想配列も展開できます(PHP 8.1以降)。

```php
$person = ['name' => 'tarou', 'age' => 30];
$testUnion = ['city' => 'Gifu', ...$person];

var_dump($testUnion);
// ['city' => 'Gifu', 'name' => 'tarou', 'age' => 30]
```

:::message
JavaScriptと違い、PHPのスプレッド構文で展開できるのは、配列と `Traversable` です。
JavaScriptのように、オブジェクトのプロパティを展開する使い方はできません。
:::

## 使いどころ

### 複数の配列を結合する

```php
$test1 = [1, 2, 3];
$test2 = [4, 5, 6];
$testUnion = [...$test1, ...$test2];

var_dump($testUnion);
// [1, 2, 3, 4, 5, 6]
```

`array_merge()` とやっていることは同じです。
なので、基本的に `array_merge()` を使う機会は、今後少なくなりそうです。

### 配列の一部の要素を変更する

```php
$original = [1, 2, 3, 4, 5];
$modified = [...array_slice($original, 0, 2), 6, ...array_slice($original, 3)];

var_dump($modified);
// array(5) {
//   [0]=> int(1)
//   [1]=> int(2)
//   [2]=> int(6)
//   [3]=> int(4)
//   [4]=> int(5)
// }
```

### 関数の引数として使う

```php
function sum(...$numbers) {
    return array_sum($numbers);
}

$result = sum(1, 2, 3, 4, 5);

var_dump($result);
// int(15)
```

「配列をそのまま渡せばいいのでは？」と思いました。
ですが、呼び出し元で渡す引数がいくら増えても対応できるので、意外と便利かもしれません。

### 配列のコピーを作る

```php
$original = [1, 2, 3];
$copy = [...$original];

var_dump($copy);
// [1, 2, 3]
```

## 思うところ

JavaScriptでも、ES2015からスプレッド構文が使えます。

* [スプレッド構文 - MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax)

PHPのほうが先に機能として出ていたのですが、なんと言ってもJavaScriptは大御所です。
JSを使うプロジェクトなら、JSと同じ感覚で書けるのは魅力かもしれません。

## まとめ

* スプレッド構文(`...`)は、配列の要素を展開する構文です。
* 配列の結合は、`array_merge()` の代わりに使えます。
* 関数の引数にも使えて、引数の数が増えても対応できます。
* JSと違い、PHPでオブジェクトのプロパティは展開できません。
