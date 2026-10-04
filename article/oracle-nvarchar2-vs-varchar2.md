---
title: "OracleのNVARCHAR2とVARCHAR2の違いと変換方法(TO_NCHAR)"
emoji: "🔤"
type: "tech"
topics: ["oracle", "sql", "nvarchar2", "database"]
published: false
published_at: "2021-09-23 09:11"
---

> 初出: 2021年9月23日(旧ブログの記事を移行・加筆)

OracleのNVARCHAR2とVARCHAR2の違い、そしてVARCHAR2からNVARCHAR2への変換方法をまとめました。
日本語を扱うテーブルを設計するときの、データ型選びで迷っている方に向けた内容です。

## 結論

* `NVARCHAR2` は、各国の国語文字を、**文字数**で管理できるデータ型です。
* `VARCHAR2` の長さは、(デフォルトでは)バイト数です。日本語は1文字が2〜3バイトなので、少ない文字数しか入りません。
* 型が違うときは、`TO_NCHAR` で変換します。

## NVARCHAR2とは

各国の国語文字を格納できるデータ型です。
「国語文字」とは、日本なら日本語、中国なら中国語のような、国固有の文字のことです。
`NVARCHAR2(5)` とすると、「あいうえお」が格納できます。

:::message
**文字コードについて**

パソコンは、0と1の2つの値で情報を判別しています。
英語圏では1バイトで1文字ですが、日本語は2〜3バイトで1文字を表します。
このように、文字コードは言語によって形式が異なるので、文字コードの設定は大切です。
:::

`VARCHAR2(5)` だと、文字コードの設定にもよりますが、日本語は1文字あたり2〜3バイト使うので、2〜3文字ほどしか入りません。
バイトではなく、日本語を**文字数**で管理したいなら、`NVARCHAR2` がおすすめです。

:::message alert
`NVARCHAR2` で、日本語1文字を「1」として扱えても、容量としては、2〜3バイトを消費している点に注意してください。
:::

## VARCHAR2をNVARCHAR2に変換するには

型が違うと、同じ文字列でも、異なる値として扱われます。
同じ値として扱いたいときは、型変換をしましょう。

`VARCHAR2` や `CHAR` を、`NVARCHAR2` や `NCHAR` に変換するときは、`TO_NCHAR` を使います。

```sql
TO_NCHAR(変換したい値)
```

変換したい値には、数値も指定できます。

### 具体的な使い方

システム改修で、新しいテーブルに `NVARCHAR2` のカラムを用意して、既存テーブルの `VARCHAR2` のカラムのデータを入れたい場面があったとします。
そのときは、型変換をして格納します。

```sql
CREATE TABLE newTable (
  id   NUMBER(2,0) NOT NULL,
  name NVARCHAR2(15)
);

INSERT INTO newTable (id, name)
SELECT id, TO_NCHAR(name)
FROM oldTable;
```

:::message
Oracleには、MySQLの `AUTO_INCREMENT` はありません。連番が必要な場合は、シーケンスや、IDENTITY列(12c以降)を使います。
:::

## まとめ

* `NVARCHAR2` は、国語文字を文字数で扱えるデータ型です。
* ただし、容量は、日本語1文字あたり2〜3バイトを消費します。
* `VARCHAR2` から `NVARCHAR2` への変換は、`TO_NCHAR` を使います。
