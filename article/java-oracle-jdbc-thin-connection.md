---
title: "JavaとOracleをJDBC(Thin接続)でつなぐ手順"
emoji: "🔌"
type: "tech"
topics: ["java", "oracle", "jdbc", "database"]
published: false
published_at: "2022-02-06 16:53"
---

> 初出: 2022年2月6日(旧ブログの記事を移行・加筆)

JavaからOracleDBに、JDBCのThin接続でつなぐ手順をまとめました。
JavaとDBを初めて接続する方に向けた内容です。

## 結論

OracleのJDBCドライバ(jarファイル)を用意して、`DriverManager.getConnection` で接続します。
**DBのバージョンとJDKのバージョンに合ったjarファイル**を使うことが大事です。

## 事前準備

### 用意するもの

jarファイルと、テストデータを用意します。
OracleDBに接続するので、jarファイルをOracleのサイトからダウンロードして、`WEB-INF/lib` ディレクトリの直下に配置します。

:::message
**DBのバージョンとJDKのバージョンに合ったjarファイル**でないと、接続がうまくいきません。(私は1日つぶれました。)
:::

テストデータは、次のテーブルを使います。

```sql
SQL> desc AAA;
 Name        Null?    Type
 ----------- -------- ----------------
 NO                   NUMBER
 NAME                 VARCHAR2(10)

SQL> SELECT * FROM AAA;

        NO NAME
---------- ----------
         1 aiueo
```

### ドライバの仕組み

ここで言うドライバは、事前に用意するjarファイルのことです。
ドライバを使うと、DBごとの特色を、可能な限り考えなくて済むようになり、プログラム(Java)とDBを連携しやすくなります。
難しく考えすぎず、**JavaとDBをつなぐときに必要なファイル**と覚えておけば十分です。

## 接続方法は2つある

JDBCドライバでの接続には、「OCI接続」と「Thin接続」の2つがあります。
違いは、**Oracle Clientがインストールされている必要があるかどうか**です。
Thin接続は、Oracle Clientが不要で、現在の主流の方法です。

## 接続手順

今回は、Thin接続を使います。
必要な手順は、次の2つです。

1. JDBCドライバを読み込みます。
2. データベースに接続します。

```java
// ① JDBCドライバの読み込み
final String path = "oracle.jdbc.OracleDriver";
Class.forName(path);

// ② データベース接続
final String URL = "jdbc:oracle:thin:usr1/mypassword@//localhost:1521/ORCLPDB1";
Connection con = DriverManager.getConnection(URL);
```

:::message
`oracle.jdbc.driver.OracleDriver` は古い指定です。最近のドライバでは、`oracle.jdbc.OracleDriver` を使います。
また、JDBC 4.0以降は、`Class.forName` を省略できる場合もあります。
:::

## データを取得する

データを取得する手順は、次のとおりです。

1. SQL文を、文字列の変数に格納します。
2. `PreparedStatement` に、変数を格納します。
3. `ResultSet` に、実行結果を格納します。
4. 結果を、ループ処理で出力します。

```java
// ①
String sql = "SELECT * FROM AAA";

// ②
PreparedStatement ps = con.prepareStatement(sql);

// ③
ResultSet rs = ps.executeQuery();

// ④
while (rs.next()) {
    System.out.println(rs.getInt("no"));
    System.out.println(rs.getString("name"));
}
```

使い終わったら、`ResultSet`、`PreparedStatement`、`Connection` を閉じます。`try-with-resources` で書くと、自動で閉じてくれます。

## まとめ

* JavaからOracleへは、JDBCドライバ(jarファイル)を使って接続します。
* Thin接続は、Oracle Clientが不要で、現在の主流です。
* DBとJDKのバージョンに合ったドライバを使いましょう。
* 接続したら、`PreparedStatement` と `ResultSet` でデータを取得します。
