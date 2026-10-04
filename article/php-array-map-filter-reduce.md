---
title: "PHPでよく使う配列関数 array_map・array_filter・array_reduce"
emoji: "🧮"
type: "tech"
topics: ["php", "array", "arraymap", "arrayfilter", "arrayreduce"]
published: false
published_at: "2023-05-27 11:28"
---

> 初出: 2023年5月27日(旧ブログの記事を移行・加筆)

PHPの配列を加工するときによく使う、`array_map`・`array_filter`・`array_reduce` の使い方をまとめました。
`foreach` で配列を1つずつ処理していた方が、もっとスッキリ書きたいときの参考になれば幸いです。

## 結論

配列の加工は、`foreach` ではなく専用の関数に任せると、短く読みやすく書けます。

| 関数 | できること |
| --- | --- |
| `array_map` | 配列の各要素に処理をして、新しい配列を返します。 |
| `array_filter` | 条件に合う要素だけを残します。 |
| `array_reduce` | 配列を1つの値にまとめます。 |

これまでは、配列から1つずつ要素を取り出して処理していました。
PHPにこんな便利な関数があると知ってからは、もう手放せません。

## array_map

配列の各要素に、決まった処理を行う関数です。
かなりよく使うようになりました。だって便利なんです。
私はCSVを読み込んだあとの、文字コードの変換などでよく使っています。

```php
array_map(コールバック関数, 配列)
```

```php
$test = ['ジョーダン', 'ジョンソン', 'ジャクソン'];

$testMap = array_map(function($t) {
    return 'マイケル' . $t;
}, $test);

var_dump($testMap);
// array(3) {
//   [0]=> string(27) "マイケルジョーダン"
//   [1]=> string(27) "マイケルジョンソン"
//   [2]=> string(27) "マイケルジャクソン"
// }
```

コールバックで配列を返すと、結果は多次元配列になります。
ここだけは注意してください。

```php
$test = ['ジョーダン', 'ジョンソン', 'ジャクソン'];

$testMap = array_map(function($t) {
    return ["test" => 'マイケル' . $t];
}, $test);

var_dump($testMap);
// array(3) {
//   [0]=> array(1) { ["test"]=> string(27) "マイケルジョーダン" }
//   [1]=> array(1) { ["test"]=> string(27) "マイケルジョンソン" }
//   [2]=> array(1) { ["test"]=> string(27) "マイケルジャクソン" }
// }
```

## array_filter

名前のとおり、配列を絞り込む関数です。
コールバックが `true` を返した要素だけが残ります。

コールバックを省略すると、`empty()` と同じ判定になります。
そのため、**配列から空の要素を取り除く**ことができます。

```php
$test = ['ジョーダン', '', 'ジャクソン'];

$testFilter = array_filter($test);

var_dump($testFilter);
// array(2) {
//   [0]=> string(15) "ジョーダン"
//   [2]=> string(15) "ジャクソン"
// }
```

結果のキーは、元の配列のまま残ります(上の例では `0` と `2`)。
キーを振り直したいときは、`array_values()` と組み合わせましょう。

## array_reduce

配列の要素を、1つの値に加工してくれる関数です。
たとえば、チェックボックスで指定された検索条件(配列)を、SQLの条件式にまとめるときに使えます。

```php
$test = ['1', '2', '3'];

$query = '1=1';
$testFilter = array_reduce($test, function($carry, $item) {
    $carry = $carry . ' OR test_column = ' . $item;
    return $carry;
}, $query);

var_dump($testFilter);
// string(60) "1=1 OR test_column = 1 OR test_column = 2 OR test_column = 3"
```

:::message alert
上の例は、動きを説明するために、SQLの文字列をそのまま連結しています。
実際のアプリで外部入力を使うときは、SQLインジェクションの原因になるので、プレースホルダを使ってください。
:::

`array_reduce()` に渡した要素がなくなるまで、次の処理が繰り返されます。

1. `$test` のインデックス0の値を取り出して、クロージャの `$item` に入れ、処理をします。
2. 1つの処理が終わったら、結果を `$carry` に保持します。
3. 最後に `$carry` を返します。

第3引数の、最初に使う初期値(上の例では `$query`)は省略できます。
省略した場合、`$carry` は最初 `null` になります。

```php
$test = ['1', '2', '3'];

$testFilter = array_reduce($test, function($carry, $item) {
    $carry = $carry . ' OR test_column = ' . $item;
    return $carry;
});

var_dump($testFilter);
// string(57) " OR test_column = 1 OR test_column = 2 OR test_column = 3"
```

## まとめ

* `array_map` は、各要素を加工した新しい配列を返します。
* `array_filter` は、条件に合う要素だけを残します。コールバックを省略すると、空の要素を除去できます。
* `array_reduce` は、配列を1つの値にまとめます。初期値は省略できます。
* `foreach` で書いていた処理は、これらに置き換えると読みやすくなります。
