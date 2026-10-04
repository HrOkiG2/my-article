---
title: "UbuntuにMySQLクライアントだけ入れてRDSに接続する方法"
emoji: "🔌"
type: "tech"
topics: ["ubuntu", "mysql", "rds", "aws", "ec2"]
published: false
published_at: "2023-12-17 20:56"
---

> 初出: 2023年12月17日(旧ブログの記事を移行・加筆)

UbuntuのEC2に、MySQL本体を入れずに、MySQLクライアント(コマンド)だけ入れて、RDSに接続する方法をまとめました。
EC2からRDSに接続しようとして、`mysql` コマンドが見つからずに困った方に向けた備忘録です。

## 結論

`mysql-client-core-8.x` のパッケージを入れれば、MySQL本体を入れなくても、`mysql` コマンドが使えます。

## 困ったこと

RDSを構築したあと、EC2から接続しようとして、つながらずに泣きたくなりました。
原因は単純で、**EC2にMySQLクライアントがなかった**ことです。

クラウド側(RDS)にMySQLがあるのに、わざわざEC2にMySQL本体を入れるのは、もったいないと思っていました。
少し工夫すれば、クライアントだけを入れて接続できると分かったので、共有します。

## 手順

1. パッケージ情報を更新します。

   ```bash
   sudo apt update
   ```

2. `mysql-client` 系のパッケージを検索します。

   ```bash
   apt search mysql-client
   ```

3. 最小構成のクライアントを入れます。使っているバージョンに合わせて指定してください。クライアントを入れれば `mysql` コマンドが使えます。

   ```bash
   sudo apt install mysql-client-core-8.0
   ```

4. RDSに接続します。

   ```bash
   mysql -h <RDSのエンドポイント> -P 3306 -u <ユーザー名> -p
   ```

## 失敗したときのエラー

### 権限不足

```
Permission denied
```

Ubuntuなので、なんとなく入ったように見えても、実は権限が足りていませんでした。
`sudo` を付けて実行しましょう。

### 接続失敗

```
ERROR 2003 (HY000): Can't connect to MySQL server on ...
```

次の2点を確認してください。

* RDSの設定漏れがないか
* EC2からの接続を許可しているか(RDSのセキュリティグループ)

## まとめ

* EC2からRDSに接続するには、MySQLクライアントが必要です。
* `mysql-client-core-8.x` で、クライアントだけを入れられます。
* つながらないときは、`sudo` の有無と、RDSのセキュリティグループを確認しましょう。
