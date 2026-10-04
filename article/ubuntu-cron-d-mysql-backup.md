---
title: "Ubuntuのcron.dでMySQLのバックアップを自動化する"
emoji: "⏰"
type: "tech"
topics: ["ubuntu", "cron", "mysql", "shellscript", "backup"]
published: false
published_at: "2023-02-16 23:00"
---

> 初出: 2023年2月16日(旧ブログの記事を移行・加筆)

Ubuntuの `cron.d` と `mysqldump` を使って、MySQLのバックアップを毎日自動で取る方法をまとめました。
LAMP環境などで、DBの定期バックアップを仕込みたい方に向けた内容です。

## 結論

* 定期実行は、`crontab` ではなく `/etc/cron.d/` 配下にファイルを置いて管理すると楽です。
* バックアップは、シェルスクリプト内で `mysqldump` を実行し、古いファイルも一緒に削除します。

## cronとは

「あの処理を定期実行したい！」というときに使う、OS搭載のアプリケーションです。
1日1回、1時間ごとに1回など、好きなタイミングでプログラムを動かせて便利です。

## cron.dを使う理由

Ubuntuには `crontab` と `cron.d` の2種類のcronがあります。
個人的には、`cron.d` のほうが圧倒的に管理しやすいと思います。

* `crontab` は、1つのファイルの中に定期実行の記述をまとめて書きます。
* `cron.d` は、ディレクトリ配下に機能ごとのファイルを置けます。

MySQLならMySQL用、PHPならPHP用とファイルを分けられるので、用途ごとに管理できます。

```
/etc/cron.d
├── certbot
├── mysql
└── php
```

:::message
`cron.d` 配下のファイル名には、拡張子をつけないでください。
仕様上、拡張子なしのファイル名にする必要があります。
:::

## 定期実行を作成する

### cronファイルを作成する

今回はMySQLのバックアップを取りたいので、`/etc/cron.d/mysql` を作成します。
このファイルで、実行するシェルと、実行結果の出力先を指定します。

```
# 分 時 日 月 曜日 ユーザー コマンド
00 00 * * * root /var/mysql_backup/mysql-backup.sh >> /var/mysql_backup/mysql-backup.log 2>&1
```

毎日0時0分に、`root` ユーザーでバックアップ用のシェルを実行する設定です。

### シェルスクリプトを作成する

やりたいことは、`mysqldump` でDBのバックアップを取得することです。

まず、バックアップファイルを保管するディレクトリを作ります。

```bash
mkdir /var/mysql_backup
```

次に、シェルを作成して、実行権限をつけます。

```bash
touch /var/mysql_backup/mysql-backup.sh
chmod +x /var/mysql_backup/mysql-backup.sh
nano /var/mysql_backup/mysql-backup.sh
```

シェルの中身は、次のとおりです。

```bash
#!/bin/bash

# 保管日数
storagedate=14

# ディレクトリ指定
basedir='/var/mysql_backup'

# ファイル名定義
fileprefix="mysql-dump-"
filedate=$(date +%y%m%d)
filename=${fileprefix}${filedate}

# dumpコマンド
mysqldump --single-transaction --skip-lock-tables --no-tablespaces \
  --password='password' -u 'user' -B 'DB_name' > ${basedir}/${filename}.sql

# 出力ファイルのパーミッション(任意)
chmod 775 ${basedir}/${filename}.sql

# 削除対象日
deletedate=$(date --date="${storagedate} days ago" +%y%m%d)

# 削除ファイル名の指定
deletefile=${basedir}/${fileprefix}${deletedate}.sql

# 削除
rm -f ${deletefile}
```

ポイントは次の3つです。

* `mysqldump` で、指定したDBを日付つきのファイル名で出力します。
* 保管日数(`storagedate`)より古いバックアップは、`rm -f` で削除します。
* パスワードをシェルに直書きしているので、ファイルの権限には気をつけましょう。

:::message alert
元の手順では、`date` コマンドの書式(`date +%y%m%d`)や、`$storagedate` の参照、削除ファイル名にプレフィックスが入っていない点などが誤っていたため、上記のように修正しています。
:::

## 動作確認

シェルを書き終えたら、cronを再読み込みして完了です。
うまく動かない場合は、ログを確認して、どこで失敗しているかを調べましょう。

```bash
# シェルを手動で実行してみる
sudo /var/mysql_backup/mysql-backup.sh

# 出力ログの確認
cat /var/mysql_backup/mysql-backup.log
```

## まとめ

* 定期実行は、`/etc/cron.d/` にファイルを分けて管理すると見通しが良くなります。
* `cron.d` のファイル名には拡張子をつけないでください。
* バックアップは `mysqldump` で取得し、古いファイルは同じシェルで削除します。
* うまくいかないときは、シェルを手動で実行して、ログから原因を探しましょう。
