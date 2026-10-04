---
title: "PHPとMySQLをPDOで接続してデータを取得する手順"
emoji: "🔌"
type: "tech"
topics: ["php", "mysql", "pdo", "database"]
published: false
published_at: "2022-03-01 06:14"
---

> 初出: 2022年3月1日(旧ブログの記事を移行・加筆)

PHPからMySQLへ接続し、データを取得するまでの最小手順をまとめました。
PHPとDBの接続が初めての方や、JavaのようにJARで接続する感覚で戸惑っている方を対象にしています。

## 結論

PHPとMySQLの接続は、PHP標準の `PDO` クラスを使えば数行で完結します。
JavaのようにドライバのJARを用意する必要はありません。

## 前提

MySQLのインストールが済んでいることを前提にします。

## DBとテーブルを用意する

まず、データベースとテーブルを作成します。

```sql
-- データベース作成
CREATE DATABASE study;

-- テーブル作成
CREATE TABLE studyContents(
  no int(2),
  name varchar(20) NOT NULL,
  PRIMARY KEY(no)
);

-- 確認
mysql> desc studycontents;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| no    | int(2)      | NO   | PRI | NULL    |       |
| name  | varchar(20) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

## PDOでMySQLに接続する

`PDO` は PHP Data Objects の略で、**DBへ簡単にアクセスするためにPHPが用意しているクラス**です。
インスタンス生成時に、次の3つを引数として渡します。

* データベース接続に必要な情報(DSN)
* ユーザー名
* パスワード

```php
try {
    // 定義
    $dsn = 'mysql:dbname=study;host=localhost;charset=utf8mb4';
    $user = 'root';
    $password = 'root';

    // 呼び出し
    $db = new PDO($dsn, $user, $password);
    echo "接続しました";
} catch (PDOException $e) {
    echo "データベースの接続に失敗しました" . $e->getMessage();
}
```

### 接続時の注意点

| 注意点 | 対処 |
| --- | --- |
| データベース名・ホスト名の記載ミス | `select user(), current_user();` で接続情報を確認します。 |
| `charset` の指定忘れ | MySQLとPHPで文字コードが異なると文字化けの原因になります。 |

## データを取得する

接続できたら、試しにデータを取得します。
手順は次のとおりです。

1. SELECT文を用意する
2. `prepare` メソッドにSELECT文をセットする
3. SQLインジェクション対策をする(今回は省略します)
4. `execute` メソッドでSQLを実行する
5. 結果を取得して出力する

```php
// 1. SELECT文の用意
$sql = "SELECT * FROM studyContents";

// 2. prepareメソッドにSELECT文をセット
$stmt = $db->prepare($sql);

// 4. executeメソッドでSQLを実行
$stmt->execute();

// 5. 結果の取得及び出力
$result = $stmt->fetchAll(PDO::FETCH_ASSOC);

foreach ($result as $r) {
    echo $r['no'] . ':' . $r['name'];
}
```

### fetch と fetchAll の使い分け

* 1行だけ取得する場合は `fetch` を使います。例: ログインユーザーの name、pw の取得
* 複数行を一度に取得する場合は `fetchAll` を使います。例: 今月売れた商品のデータ取得

取得形式のオプションは `PDO::FETCH_ASSOC` がデータ消費量が少なくおすすめです。
オプションは4種類あるため、用途に応じて公式ドキュメントで確認してください。

## まとめ

* PHPとMySQLの接続には `PDO` を使います。
* DSNには、DB名・ホスト名・`charset` を指定します。
* 取得は `prepare` → `execute` → `fetch` / `fetchAll` の流れです。
* 本番ではSQLインジェクション対策として、プレースホルダの利用が必須です。
