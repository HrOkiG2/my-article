---
title: "JavaでDBのDATE型・TIMESTAMP型を扱うときの注意点"
emoji: "📅"
type: "tech"
topics: ["java", "oracle", "jdbc", "sql", "date"]
published: false
published_at: "2021-09-20 11:52"
---

> 初出: 2021年9月20日(旧ブログの記事を移行・加筆)

JavaでDBの日付型(DATE、TIMESTAMP)を扱うときの注意点をまとめました。
JDBCでOracle DBなどを扱う、Java初学者の方に向けた内容です。

## 結論

JavaでSQLの値を扱うときは、**SQLの型に対応する型を、Java側で用意する**必要があります。
日付は、`DATE` なら `java.sql.Date`、`TIMESTAMP` なら `java.sql.Timestamp` を使います。

## 気をつけること

SQLから受け取った値を扱える型が、Java側にないと、DBの値を間違った形で取得してしまい、整合性のないプログラムになります。
まず、SQLとJavaの型の対応を見てみましょう。(Oracle DBでよく使うデータ型です。)

| SQL | Java |
| --- | --- |
| `CHAR`、`VARCHAR2` | `String` |
| `NUMBER` | `int`、`float`、`byte` など |
| `DATE` | `java.sql.Date` |
| `TIMESTAMP` | `java.sql.Timestamp` |

特に気をつけたいのが、日付を扱う `DATE` 型と `TIMESTAMP` 型です。
`DATE` 型は「年・月・日」を扱い、`TIMESTAMP` 型は日付と時刻を扱います。

```
-- DATE型(YYYY/MM/DD)
2021/08/20

-- TIMESTAMP型(YYYY/MM/DD HH:MM:SS)
2021/08/20 12:30:25
```

日付も、文字列のように `String` 1つで扱えたら楽ですが、Javaでは少し難しいです。
次のように書いて対応します。

```java
/* DATE */
String sql = "SELECT 登録日 FROM SAMPLE WHERE ROWNUM = 1";
PreparedStatement ps = con.prepareStatement(sql);
ResultSet rs = ps.executeQuery();
if (rs.next()) {
    java.sql.Date date = rs.getDate(1);
}

/* TIMESTAMP */
String sql2 = "SELECT 登録日 FROM SAMPLE WHERE ROWNUM = 1";
PreparedStatement ps2 = con.prepareStatement(sql2);
ResultSet rs2 = ps2.executeQuery();
if (rs2.next()) {
    java.sql.Timestamp ts = rs2.getTimestamp(1);
    java.util.Date date2 = new java.util.Date(ts.getTime());
}
```

## 補足: SQL側で日付を変換する

日付型の変換はよく使うので、紹介しておきます。
SQL側で変換すれば、Java側で型をあまり気にせずに値を受け取れます。

```sql
/* 日付型を文字列に変換 */
SELECT TO_CHAR(登録日, 'YYYY/MM/DD'),
       TO_CHAR(登録日, 'YYYY/MM/DD HH:MI:SS'),   -- 12時間形式
       TO_CHAR(登録日, 'YYYY/MM/DD HH24:MI:SS')  -- 24時間形式
FROM SAMPLE;

/* 文字列を日付型に変換 */
SELECT TO_DATE('2021/08/20', 'YYYY/MM/DD'),
       TO_DATE('2021/08/20 12:30:25', 'YYYY/MM/DD HH24:MI:SS')
FROM DUAL;
```

:::message
フォーマットの分は `MI` です。`MM` は月を表すので、時刻の分には使えません。
:::

## まとめ

* JavaでSQLの値を扱うときは、対応する型を用意します。
* `DATE` は `java.sql.Date`、`TIMESTAMP` は `java.sql.Timestamp` です。
* SQL側で `TO_CHAR` や `TO_DATE` で変換する方法もあります。
