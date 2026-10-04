---
title: "Oracleの日付型とタイムゾーン: DATE・TIMESTAMP・INTERVALの違い(Silver SQL対策)"
emoji: "🕐"
type: "tech"
topics: ["oracle", "sql", "date", "timestamp", "silver"]
published: false
published_at: "2021-11-28 13:39"
---

> 初出: 2021年11月28日(旧ブログの記事を移行・加筆)

Oracleの日付型の種類と、タイムゾーン、表示形式をまとめました。
ORACLE MASTER Silver SQLの試験勉強をしている方や、Oracleで日付を扱う方に向けた内容です。

## 結論

* Oracleの日付型は、`DATE`、`TIMESTAMP` 系、`INTERVAL` 系があります。
* `SYSDATE` と `CURRENT_DATE` では、基準になるタイムゾーンが違います。
* 表示形式は、`NLS_DATE_FORMAT` で変更できます。

## 日付型の種類

Oracleの日付型は、次の6種類です(期間データ型を含みます)。

| データ型 | 現在値を取得する関数 |
| --- | --- |
| `DATE` | `SYSDATE` |
| `TIMESTAMP` | `SYSTIMESTAMP` |
| `TIMESTAMP WITH TIME ZONE` | `CURRENT_TIMESTAMP` |
| `TIMESTAMP WITH LOCAL TIME ZONE` | `LOCALTIMESTAMP` |

| 期間データ型 | 例 |
| --- | --- |
| `INTERVAL YEAR TO MONTH` | `SYSDATE + INTERVAL '1-1' YEAR TO MONTH` |
| `INTERVAL DAY TO SECOND` | `SYSTIMESTAMP + INTERVAL '5 01:00:00' DAY TO SECOND` |

### 型ごとの違い

* **`DATE` と `TIMESTAMP`**: ミリ秒(小数秒)まで含むのが `TIMESTAMP` です。(先輩いわく、厳密なシステム以外は、基本的に `DATE` を使うそうです。)
* **`TIMESTAMP WITH TIME ZONE` と `TIMESTAMP WITH LOCAL TIME ZONE`**: どちらもタイムゾーンの情報を扱います。**`WITH LOCAL TIME ZONE` は、DBのタイムゾーンに変換されて保存されます。** たとえば、DBのタイムゾーンが `Asia/Tokyo` なら、そのタイムゾーンに変換されて保存されます。一方、`WITH TIME ZONE` は、タイムゾーンの情報がそのまま保存され、取得時にタイムゾーン名称も表示されます。
* **`INTERVAL`**: どちらも時刻の差分を管理します。**年月は `INTERVAL YEAR TO MONTH`**、**日と秒は `INTERVAL DAY TO SECOND`** です。

## タイムゾーンについて

Oracleは世界中で使われるRDBMSなので、地域に合わせた時刻を表示できます。

| 関数 | 基準になるタイムゾーン |
| --- | --- |
| `CURRENT_DATE` | セッションのタイムゾーン |
| `CURRENT_TIMESTAMP` | セッションのタイムゾーン |
| `SYSDATE` | データベースのタイムゾーン |
| `SYSTIMESTAMP` | データベースのタイムゾーン |
| `LOCALTIMESTAMP` | セッションのタイムゾーン |

```sql
SQL> select systimestamp from dual;

SYSTIMESTAMP
-------------------------------
21-11-28 03:41:28.349648000 GMT

SQL> alter session set time_zone='Asia/Tokyo';

SQL> select current_timestamp from dual;

CURRENT_TIMESTAMP
--------------------------------------
21-11-28 12:41:28.360894000 ASIA/TOKYO
```

次のSQLで、DBのタイムゾーン(世界標準時との差)を確認できます。

```sql
SELECT DBTIMEZONE FROM dual;
```

## 表示について

日付型は、数値型や文字列型のように、文字列を指定して表示形式を変えることができません。
「年月のみ」「月と日のみ」と表示したいときは、**`NLS_DATE_FORMAT`** で書式を変更します。

```sql
SQL> select sysdate from dual;

SYSDATE
---------
27-NOV-21

SQL> alter session set NLS_DATE_FORMAT='YYYY/MM/DD';

SQL> select sysdate from dual;

SYSDATE
----------
2021/11/27
```

日付型の書式は、次の設定にも影響を受けます。

* `NLS_DATE_LANGUAGE`
* `NLS_TERRITORY`

`NLS_DATE_LANGUAGE` が `AMERICAN` の場合は、次のようになります。

```sql
SQL> alter session set NLS_DATE_FORMAT='yyyy-mm-dd-day';
SQL> select sysdate from dual;

SYSDATE
--------------------
2021-11-28-sunday
```

`NLS_DATE_LANGUAGE` を `Japanese` に変更すると、次のようになります。

```sql
SQL> alter session set NLS_DATE_LANGUAGE='Japanese';
SQL> select sysdate from dual;

SYSDATE
--------------
2021-11-28-日曜日
```

こちらのサイトの解説が、とても分かりやすいです。

* [知らないとチョットつまづく RDS for Oracle の NLS パラメータ](https://dev.classmethod.jp/articles/rds-for-oracle-nls-param/)

## まとめ

* 日付型には、`DATE`、`TIMESTAMP` 系、`INTERVAL` 系があります。
* `SYSDATE`/`SYSTIMESTAMP` はDBのタイムゾーン、`CURRENT_*`/`LOCALTIMESTAMP` はセッションのタイムゾーンが基準です。
* 表示形式は `NLS_DATE_FORMAT` で変更でき、`NLS_DATE_LANGUAGE` などにも影響されます。
