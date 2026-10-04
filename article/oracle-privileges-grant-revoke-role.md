---
title: "Oracleの権限まとめ: システム権限・オブジェクト権限・ロール(Silver SQL対策)"
emoji: "🔑"
type: "tech"
topics: ["oracle", "sql", "privilege", "grant", "silver"]
published: false
published_at: "2021-11-21 00:43"
---

> 初出: 2021年11月21日(旧ブログの記事を移行・加筆)

Oracleの権限について、システム権限、オブジェクト権限、ロールをまとめました。
ORACLE MASTER Silver SQLの試験勉強をしている方や、Oracleの権限で迷った方に向けた内容です。

## 結論

* 権限は、ユーザーごとにできることを分けて、DBを安全に運用するための仕組みです。
* 付与は `GRANT`、剥奪は `REVOKE` です。
* 権限は、システム権限とオブジェクト権限の2種類あります。
* 複数の権限をまとめるのが、ロールです。

## 権限の目的

権限があるのは、**ユーザーごとにできることを分けて、データベースを安全に運用するため**です。

会社にたとえると、社長と新卒社員では、持つ権限が違います。
雑用で手一杯の新卒社員に、会社の最終決定権を持たせたら、大変なことになります。
データベースも同じで、「データを削除していい人」「スキーマを作っていい人」など、権限を切り分けることで、安全に運用できます。

別の面では、**システム運用のために権限を振り分ける**理由もあります。
会社に部署があるように、Aユーザーは、a業務のテーブルを操作でき、Bユーザーは、b業務とc業務のテーブルを操作できる、といった使い分けです。

## 権限の構文

```sql
-- 権限の付与には GRANT
GRANT SELECT, INSERT, UPDATE ON usr1.TEST TO usr2;

-- 権限の剥奪には REVOKE
REVOKE SELECT, INSERT, UPDATE ON usr1.TEST FROM usr2;
```

## 権限の種類

### システム権限

データベースに対する命令を行うための権限です。
「Oracleにお願いを聞いてもらうための装備」のようなものです。
社員証がないと会社に入れないように、命令ごとのシステム権限がないと、DBに対して何も実行できません。

代表的なものを紹介します。

| 権限 | 内容 |
| --- | --- |
| `CREATE USER` | ユーザーの作成 |
| `CREATE SESSION` | データベースへの接続 |
| `CREATE TABLE` | 表の作成 |

私は初めてOracleでユーザーを作ったとき、`CREATE SESSION` を付与し忘れて、DBに接続できず、3日ほど立ち往生しました。
最低限付与すべき権限もあるので、一度は目を通しておくとよいです。

* [Oracleシステム権限の一覧と確認](http://itref.fc2web.com/oracle/system-privilege.html)

#### ADMIN OPTION

他のユーザーからシステム権限を付与されて、それを別のユーザーに付与するには、`WITH ADMIN OPTION` が必要です。

```sql
-- usr2は接続権限を付与され、他人にも譲渡できる
GRANT CREATE SESSION TO usr2 WITH ADMIN OPTION;
```

### オブジェクト権限

**自分以外のオブジェクトに対して命令を実行できるようにする**ための権限です。
友達のパソコンや車を、許可なく使えないのと同じように、他のユーザーが持つオブジェクトには、権限がないと何も実行できません。
(自分が作ったオブジェクトには、権限は不要です。)

#### GRANT OPTION

他のユーザーからオブジェクト権限を付与されて、それを別のユーザーに付与するには、`WITH GRANT OPTION` が必要です。
この方法で付与された権限は、大元のユーザーが権限を剥奪すると、**連鎖的に削除**されます。

```sql
-- usr2は、別のユーザーに権限を付与できるようになる
GRANT SELECT, INSERT ON usr1.TEST TO usr2 WITH GRANT OPTION;

-- usr2が実行。usr3にも同じ権限を付与できる
GRANT SELECT, INSERT ON usr1.TEST TO usr3;

-- usr1が実行。usr2の権限が消えると同時に、usr3の権限も連鎖的に消える
REVOKE SELECT, INSERT ON usr1.TEST FROM usr2;
```

## ロール

ロールは、**権限をまとめたもの**です。プログラミングで言う、配列のようなものです。
一度定義しておけば、繰り返し呼び出せるので、実務でもよく使われます。
システム権限とオブジェクト権限を、**同時に格納できる**のも特徴です。

```sql
CREATE ROLE TEST_ROLE;
GRANT CREATE SESSION TO TEST_ROLE;        -- システム権限を付与
GRANT SELECT, INSERT ON usr1.TEST TO TEST_ROLE; -- オブジェクト権限を付与
GRANT TEST_ROLE TO usr1;
```

:::message
* ロールに、ロールを付与することもできます。
* すべてのユーザーに適用される、`PUBLIC` ロールがあります。
:::

## まとめ

* 権限は、ユーザーごとにできることを分けて、安全に運用するためのものです。
* システム権限は「DBへの命令」、オブジェクト権限は「他人のオブジェクトへの操作」の権限です。
* `WITH ADMIN OPTION` / `WITH GRANT OPTION` で、権限を他のユーザーに付与できるようになります。
* ロールで、権限をまとめて管理できます。
