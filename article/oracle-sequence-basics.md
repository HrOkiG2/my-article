---
title: "Oracleのシーケンスの使い方: 連番の発行とセッションごとの動き"
emoji: "🔢"
type: "tech"
topics: ["oracle", "sql", "sequence", "database"]
published: false
published_at: "2022-01-22 12:16"
---

> 初出: 2022年1月22日(旧ブログの記事を移行・加筆)

Oracleのシーケンスの基本を、作成から使い方、複数セッションでの動きまでまとめました。
Oracleで連番(主キーなど)を発行したい方に向けた内容です。

## 結論

シーケンスは、**連番を作るために、Oracleが用意しているオブジェクト**です。
`NEXTVAL` で次の番号を発行し、`CURRVAL` で現在の番号を取得します。

## シーケンスとは

連番を作り出すためのオブジェクトです。
1、2、3、4、5…と連続した値を発行するので、主キーやユニークキーによく使われます。
MySQLの `AUTO_INCREMENT` に似た機能です。

## シーケンスのコード

### CREATE文

```sql
CREATE SEQUENCE seqTest
  START WITH 1
  INCREMENT BY 1
  MAXVALUE 1000
  NOCYCLE;
```

### 取得

```sql
-- 連番を発行
SQL> select seqTest.nextval from dual;

   NEXTVAL
----------
         1

-- 現在の番号を取得
SQL> select seqTest.currval from dual;

   CURRVAL
----------
         1
```

### 使用例

```sql
SQL> desc TBLSEQ;
 Name                  Null?    Type
 --------------------- -------- ----------------
 NO                    NOT NULL NUMBER(3)
 NAME                           VARCHAR2(10)

-- test0テーブルの値を使って、連番付きで登録
SQL> INSERT INTO TBLseq
     select seqTest.nextval,
            concat(TO_CHAR(no), concat(name, TO_CHAR(no2)))
     from test0;

20 rows created.

SQL> select * from tblseq;

        NO NAME
---------- ----------
         1 0a0
         2 0b1
         3 0a2
       ...
        20 9b3

20 rows selected.
```

## 複数セッションでのシーケンスの扱い

上の例で20まで発行したシーケンスを、別のセッションから使うと、**続きの連番**から発行されます。

```sql
-- 別セッションに接続
SQL> sqlplus 〜〜〜

-- シーケンスを発行
SQL> select seqtest.nextval from dual;

   NEXTVAL
----------
        21
```

:::message
シーケンスは、ロールバックしても戻りません。また、複数のセッションで使うと、番号が飛ぶことがあります。
欠番が許されない用途には、向きません。
:::

## まとめ

* シーケンスは、連番を作るためのオブジェクトです。
* `NEXTVAL` で発行し、`CURRVAL` で現在の値を取得します。
* 複数のセッションで共有されるので、続きの番号が発行されます。
* ロールバックしても戻らず、欠番ができることがあります。
