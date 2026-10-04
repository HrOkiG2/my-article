---
title: "Oracleで権限付与時に「ユーザーがいません」になる原因(PDBとCDB)"
emoji: "🏛️"
type: "tech"
topics: ["oracle", "sql", "pdb", "cdb", "multitenant"]
published: false
published_at: "2022-01-22 11:21"
---

> 初出: 2022年1月22日(旧ブログの記事を移行・加筆)

Oracleで権限を付与しようとしたら、「ユーザーがいません」というエラーになった原因と、解決方法をまとめました。
Oracle 12c以降のマルチテナント構成で、ユーザー作成や権限付与につまずいた方に向けた内容です。

## 結論

原因は、**接続先のコンテナがCDBとPDBで違っていた**ことでした。
SYSTEMユーザーの接続先を、対象ユーザーがいるPDBに切り替えれば、権限を付与できます。

## 問題

ローカルユーザーに権限を付与すると、「ユーザーがいません」というエラーが出ました。
SYSTEMユーザーでログインしているので、付与できないはずがないのですが、思わぬところに落とし穴がありました。

```sql
SQL> GRANT CREATE SEQUENCE TO usr1;
GRANT CREATE SEQUENCE TO usr1
                         *
ERROR at line 1:
ORA-01917: user or role 'USR1' does not exist
```

## 原因

OracleのマルチテナントというDB構造が原因でした。
**CDB**が基盤となるDBで、その上に**PDB**と呼ばれるDBが存在する、という構造です。(Dockerで、コンテナの上にイメージを作るようなイメージです。)

今回は、SYSTEMユーザーがCDBに接続していて、権限を付与されるUSR1ユーザーは、PDBにいました。
CDB上のSYSTEMユーザーからは、PDB上のUSR1に権限を付与できないので、SYSTEMユーザーをPDBに接続する必要があります。

## 解決方法

次の手順で解決できます。

1. 権限を付与する側のユーザー(SYSTEM)の接続先を確認します。

   ```sql
   SQL> show con_name

   CON_NAME
   ------------------------------
   CDB$ROOT
   ```

2. 権限を付与される側のユーザー(USR1)の接続先を確認します。

   ```sql
   SQL> show con_name

   CON_NAME
   ------------------------------
   ORCLPDB1
   ```

3. 権限を付与する側のユーザーの接続先を、USR1がいるPDBに切り替えます。

   ```sql
   SQL> alter session set container = orclPDB1;

   Session altered.
   ```

4. `GRANT` 文を発行します。

   ```sql
   SQL> GRANT CREATE SEQUENCE TO usr1;

   Grant succeeded.
   ```

## まとめ

* Oracleのマルチテナントでは、CDBの上にPDBがあります。
* CDBに接続しているユーザーから、PDBのローカルユーザーには、権限を付与できません。
* `show con_name` で接続先を確認して、`alter session set container = <PDB名>` で切り替えましょう。
