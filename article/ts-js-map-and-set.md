---
title: "JavaScript/TypeScriptのMapとSetの違いと使い方"
emoji: "🗺️"
type: "tech"
topics: ["javascript", "typescript", "map", "set"]
published: false
published_at: "2023-06-04 15:28"
---

> 初出: 2023年6月4日(旧ブログの記事を移行・加筆)

JavaScript・TypeScriptの `Map` と `Set` の違いを、使い方とあわせて整理しました。
配列やオブジェクトしか使ったことがなく、`Map` / `Set` の使いどころを知りたい方に向けた内容です。

## 結論

| 種類 | 特徴 |
| --- | --- |
| `Map` | 連想配列のように、キーと値のペアを持つ構造です。 |
| `Set` | 重複を認めない、一意の値を保持する構造です。 |

## データの格納方法

### Map

キーと値のペアを格納します。
キーは一意でなければなりませんが、任意のデータ型を指定できます。

```js
const myMap = new Map();

// キーと値の追加
myMap.set("key1", "value1");
myMap.set("key2", "value2");
```

### Set

一意の値の集合を格納します。
重複した値は持てません。要素は、任意のデータ型を指定できます。

```js
const mySet = new Set();

// 値の追加
mySet.add("value1");
mySet.add("value2");
mySet.add("value3");
mySet.add("value2"); // 重複した値は追加されません
```

## 要素へのアクセス方法

### Map

キーを使って、要素にアクセスします。`get()` で、キーに対応する値を取得できます。

```js
// 単一の取得
myMap.get("key1");

// 一括
myMap.forEach((value, key) => {
  console.log(key, value);
});
```

### Set

単一の値の集合なので、`has()` で存在を確認したり、`forEach` や `for...of` で順に取り出したりします。
配列に変換して、アクセスすることもできます。

* [Setを配列に変換する(TypeScript Book)](https://typescriptbook.jp/reference/builtin-api/set#set%E3%82%92%E9%85%8D%E5%88%97%E3%81%AB%E5%A4%89%E6%8F%9B%E3%81%99%E3%82%8B)

ただ、せっかく `Set` で値を用意したのに、配列に変換するのは野暮な感じがしますね。

## 順序の保持

`Map` も `Set` も、**要素を追加した順番を保持**します。
イテレーションも、追加した順番に行われます。

:::message
元の記事では「Setは順序を保持しない」と書いていましたが、誤りでした。
JavaScriptの `Set` は、要素の挿入順を保持します。ただし、`Set` にはインデックスでアクセスする手段がありません。
:::

## サイズの取得

どちらも、`size` プロパティで要素数を取得できます。

```js
console.log(myMap.size); // 2
console.log(mySet.size); // 3
```

## 重複の取り扱い

* **Map**: キーが一意なので、同じキーで要素を追加すると、後の値で上書きされます。
* **Set**: 重複した値を持たないので、同じ値を何度追加しても、1つしか保持されません。

## まとめ

* `Map` は、キーと値のペアを持ちます。キーは任意のデータ型を使えます。
* `Set` は、重複しない値の集合です。配列の重複削除などに便利です。
* どちらも、追加順を保持して、`size` で要素数を取得できます。
