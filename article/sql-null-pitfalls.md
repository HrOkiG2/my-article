---
title: "SQLのNULLがやっかいな理由: =が使えない・結合・並び順・サブクエリ"
emoji: "🕳️"
type: "tech"
topics: ["sql", "mysql", "oracle", "null", "database"]
published: false
published_at: "2021-10-17 10:14"
---

> 初出: 2021年10月17日(旧ブログの記事を移行・加筆)

SQLのNULLで、ハマりやすいポイントをまとめました。
SQLを学び始めた方や、NULLの扱いで意図しない結果になった経験のある方に向けた内容です。

## 結論

NULLは「値がない」状態を表すもので、0でも空文字でもありません。
そのため、比較、結合、並び替え、サブクエリで、注意が必要です。

## NULLの特徴

NULLは、0でも空文字でもない、値のことを指します。
(イメージは、「存在しない値」を分かりやすくするために、NULLというシールを貼っている状態です。)

たとえば、「財布の中身はいくら?」という問いに対して、次の場合は、値を持っています。

* 財布の中身が0円なら、「0円」という値を持っています。
* 10円が入っていれば、「10円」という値を持っています。

ですが、財布自体がなければ、10円も0円も持てません。
この「財布がない」状態を、SQLの世界ではNULLと呼びます。

## NULLのやっかいなところ

### 「=」が使えない

NULLは値ではないので、`=`、`<`、`>` などの比較演算子が使えません。
NULLかどうかを調べるには、**`IS NULL`** を使います。

```sql
-- 名前がない人の検索
SELECT name FROM family WHERE name IS NULL;

-- 名前がある人の検索
SELECT name FROM family WHERE name IS NOT NULL;
```

NULLに対して `=` ではなく `IS` を使うのは、NULLが値ではなく、「何もない状態を示す目印」だからです。

### 文字列結合ができない

MySQLで、NULLと文字列を `CONCAT` で結合すると、結果がNULLになります。
(数学で言う「数字 × 0 = 0」のようなものです。)

```
mysql> SELECT CONCAT(busyo_name,'hoge') FROM BUSYO;
+---------------------------+
| CONCAT(busyo_name,'hoge') |
+---------------------------+
| 開発部hoge                |
| デザイン部hoge            |
| NULL                      |
+---------------------------+
```

:::message
Oracleでは、**空文字がNULLとして扱われます。**
また、OracleのCONCAT関数では、NULLが空文字のように扱われます。`CONCAT('あいうえお', NULL)` は、`あいうえお` を返します。
MySQLとOracleで動きが違うので、注意が必要です。
:::

### 並び順

`ORDER BY` で並び替えるとき、NULLを含む列では、NULLが一番上(または一番下)に来ます。
NULLは値ではないので、比較して並べることができません。

Oracleでは、`NULLS FIRST`(または `NULLS LAST`)で、NULLの位置を指定できます。

```sql
SQL> select * from fruit2 order by name1;
NAME1      NAME2
---------- ----------
apple      orange
banana     banana
orange     orange
           orange       -- NULLが最後

SQL> select * from fruit2 order by name1 nulls first;
NAME1      NAME2
---------- ----------
           orange       -- NULLが最初
apple      orange
banana     banana
orange     orange
```

### 論理演算にNULLが紛れ込むと、やっかい

`WHERE` 句に、次のようなサブクエリを書いたとします。

```sql
SELECT * FROM TEST1
WHERE no < (SELECT MIN(no) FROM TEST2);
```

NULLに比較演算子は使えないので、サブクエリの結果がNULLになると、元のSQLは1行も返しません。
`TEST2` にNULLがないと思い込んで発行すると、想定外の結果になるので、注意が必要です。
対策としては、サブクエリの中で、NULLを変換する `COALESCE` 関数などを使うことをおすすめします。

## ほかにも

まだ学習が足りず、確認できていませんが、次の内容も追記していきたいです。

* インデックスとNULL
* NULLの便利な使い方

## まとめ

* NULLは、0でも空文字でもない、値がない状態です。
* 比較は `=` ではなく、`IS NULL` / `IS NOT NULL` を使います。
* 文字列結合、並び順、サブクエリで、意図しない動きになりやすいです。
* `COALESCE` や `NULLS FIRST/LAST` で、NULLを制御しましょう。
