---
title: "SQLのサブクエリは単一行のみ？複数行を返すときのIN・EXISTS・ANY"
emoji: "🔎"
type: "tech"
topics: ["sql", "oracle", "mysql", "subquery"]
published: false
published_at: "2022-01-16 11:21"
---

> 初出: 2022年1月16日(旧ブログの記事を移行・加筆)

WHERE句のサブクエリが複数行を返したときのエラーと、対処法をまとめました。
SQLを学び始めて、`single-row subquery returns more than one row` のエラーに出会った方に向けた内容です。

## 結論

`=` で比較するサブクエリは、**結果を1行にする**必要があります。
複数行を受け取りたいときは、`IN`、`EXISTS`、`ANY` などの述語を使います。

## 複数行を返すとエラーになる

`WHERE` 句のサブクエリの結果が2行以上になると、エラーが返ってきます。

```sql
SQL> select * from test
  2  where no = (select no from test);

ERROR at line 2:
ORA-01427: single-row subquery returns more than one row
```

サブクエリの結果が複数になると、**`no` 列との比較に、どの値を使えばよいか分からない**ためのエラーです。
そのため、サブクエリが返す値を、1つにする必要があります。
サブクエリを使うときは、ケースバイケースで、返ってくる件数を意識しましょう。

```sql
-- 条件で1行に絞る
SQL> select * from test
  2  where no = (select no from test where no = 1);

        NO NAME
---------- ----------
         1 aaaaa

-- 集約関数で1つにする
SQL> select * from test
  2  where no = (select max(no) from test);

        NO NAME
---------- ----------
         5 youo
```

## 複数行を受け取れる述語

サブクエリが複数行を返すようにしたい場合は、`IN`、`EXISTS`、`ANY` などの述語を使います。
これらを使うと、**サブクエリの結果のどれかと一致するもの**を、条件にできます。

```sql
-- IN
SQL> select * from test
  2  where no IN (select no from test0);

        NO NAME
---------- ----------
         1 aaaaa
         2 iiiii
         2 uouoo

-- EXISTS
SQL> select * from test
  2  where exists (select no from test0 where test.no = test0.no);

        NO NAME
---------- ----------
         1 aaaaa
         2 iiiii
         2 uouoo

-- ANY
SQL> select * from test
  2  where no = ANY (select no from test0);

        NO NAME
---------- ----------
         1 aaaaa
         2 iiiii
         2 uouoo
```

## まとめ

* `=` で比較するサブクエリは、1行だけを返す必要があります。
* 複数行を返すときは、`IN`、`EXISTS`、`ANY` を使います。
* サブクエリが返す件数を、意識して書きましょう。
