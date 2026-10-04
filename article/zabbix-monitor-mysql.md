---
title: "ZabbixでMySQLを監視する設定(Ping監視とスロークエリ)"
emoji: "🐬"
type: "tech"
topics: ["zabbix", "mysql", "monitoring", "ubuntu"]
published: false
published_at: "2023-02-17 11:30"
---

> 初出: 2023年2月17日(旧ブログの記事を移行・加筆)

ZabbixでMySQLを監視する設定を、備忘録としてまとめました。
Zabbixで、MySQLの死活監視やスロークエリの監視をしたい方に向けた内容です。

## 結論

監視される側のMySQLに、監視用ユーザーと `.my.cnf` を用意します。
`userparameter_mysql.conf` でキーとコマンドを定義して、Zabbixサーバー側でアイテムとトリガーを設定します。

## 前提

### Zabbixのインストール

次の2つを済ませて、Zabbixが使える状態にしておきます。

* 監視する側に、Zabbixサーバーをインストールします。
* 監視される側に、Zabbixエージェントをインストールします。(詳しくは、別記事「Zabbixで外部サーバーを監視する設定」を参照してください。)

### MySQLのインストール

監視される側のサーバーには、MySQLをインストールしておきます。

## 監視するための準備

### 監視される側のサーバー

#### ユーザーの作成

Zabbixエージェントが監視できるように、監視される側のMySQLに、ユーザーを作ります。

```sql
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
```

:::message
元の手順では `grant all privileges` を使っていましたが、監視用なので、必要最小限の権限にとどめるのがおすすめです。
権限は、状況に合わせて調整してください。
:::

#### .my.cnf ファイルの作成

`/var/lib/zabbix/.my.cnf` を作ります。
デフォルトではこのファイルが存在しないので、`touch` から行います。

```ini
[client]
host=<監視される側のサーバーのIP>
user=zabbix
password='password'
```

`host` には `localhost` を指定しても動くはずですが、私の環境では、IPを指定しないと動きませんでした。
パスワードが入るファイルなので、権限(`chmod 600`)にも気をつけましょう。

#### userparameter_mysql.conf の設定

`/etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf` を編集します。
このファイルでやりたいことは、次の2つです。

* `--defaults-extra-file` オプションで、MySQLの設定ファイルを指定します。
* `mysql.ping`、`mysql.status` などの、キーとコマンドを指定します。

```
UserParameter=mysql.ping[*], mysqladmin --defaults-extra-file=/var/lib/zabbix/.my.cnf ping
UserParameter=mysql.status[*], echo "show global status where Variable_name='$1';" | mysql --defaults-extra-file=/var/lib/zabbix/.my.cnf -N | awk '{print $$2}'
UserParameter=mysql.get_status_variables[*], mysql --defaults-extra-file=/var/lib/zabbix/.my.cnf -sNX -e "show global status"
UserParameter=mysql.version[*], mysqladmin --defaults-extra-file=/var/lib/zabbix/.my.cnf -s version
UserParameter=mysql.db.discovery[*], mysql --defaults-extra-file=/var/lib/zabbix/.my.cnf -sN -e "show databases"
UserParameter=mysql.dbsize[*], mysql --defaults-extra-file=/var/lib/zabbix/.my.cnf -sN -e "SELECT SUM(DATA_LENGTH + INDEX_LENGTH) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='$3'"
UserParameter=mysql.replication.discovery[*], mysql --defaults-extra-file=/var/lib/zabbix/.my.cnf -sNX -e "show slave status"
UserParameter=mysql.slave_status[*], mysql --defaults-extra-file=/var/lib/zabbix/.my.cnf -sNX -e "show slave status"
```

設定ファイルのパスや、中身(ユーザー名やパスワード)が間違っていると、正常に監視できず、エラーになります。
リファレンス: [ユーザーパラメータ(Zabbix ドキュメント)](https://www.zabbix.com/documentation/2.2/jp/manual/config/items/userparameters)

## 監視する側のサーバーでの監視設定

### Ping監視

画像がなくて恐縮ですが、次の手順で設定できます。

1. Zabbixサーバーにログインして、Webインターフェースを開きます。
2. 監視対象のMySQLサーバーを選んで、「設定」→「ホスト」を選びます。
3. 監視対象のMySQLサーバーの設定ページで、「アイテム」タブを選びます。
4. 「アイテムの追加」をクリックして、次の設定を入力します。
   * キー: `mysql.ping`
   * 名前: `MySQL Ping`
   * タイプ: Zabbixエージェント
5. 「監視」→「グラフ」で、グラフを作ります。

この設定で、ZabbixからMySQLサーバーへのPingリクエストを、定期的に送れます。
「MySQLが生きているか」を監視できるようになります。
グラフで、Ping応答時間の変化も、視覚的に確認できます。

### スロークエリ

#### MySQL側の設定

MySQLの設定ファイルに、次を設定して、スロークエリの検出を有効にします。

```ini
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 1
```

:::message
**スロークエリとは**

MySQLのスロークエリログは、長時間実行されたクエリを特定して、パフォーマンスの問題を見つけるための、便利な仕組みです。

**MySQLの設定ファイルの違い**

`my.ini` はWindows用、`my.cnf` はWindows以外のOS用の設定ファイルです。
:::

#### Zabbix側の設定

次の手順で設定します。

1. Zabbixサーバーにログインして、Webインターフェースを開きます。
2. 監視対象のMySQLサーバーを選んで、「設定」→「ホスト」を選びます。
3. 「アイテム」タブで、「アイテムの追加」をクリックして、次の設定を入力します。
   * キー: `mysql.slowqueries`
   * 名前: `MySQL Slow Queries`
   * タイプ: Zabbixエージェント
   * 更新間隔: 30秒(必要に応じて変更します)
4. 「トリガー」を作ります。
   * 名前: `MySQL Slow Queries`
   * 条件: `mysql.slowqueries` の最新値が、0より大きい

この設定で、スロークエリを監視できます。

## まとめ

* 監視用のMySQLユーザーと、`.my.cnf` を用意します。
* `userparameter_mysql.conf` で、キーとコマンドを定義します。
* Ping監視で死活を、スロークエリの監視で性能の問題を検知できます。
