---
title: "Ubuntu 20.04でLAMP環境を作るときにハマりやすいポイント"
emoji: "🛠️"
type: "tech"
topics: ["php", "mysql", "apache", "ubuntu", "lamp"]
published: false
published_at: "2023-02-17 11:00"
---

> 初出: 2023年2月17日(旧ブログの記事を移行・加筆)

Ubuntu 20.04にLAMP環境(Linux / Apache / MySQL / PHP)を構築したときに、つまずきやすいポイントをまとめました。
普段はXAMPPで開発していて、初めてテスト環境をサーバーに構築する方に向けた備忘録です。

## 結論

手順そのものは参考記事のとおりに進めれば大丈夫です。
ハマりやすいのは、次の3つです。

* ディストリビューションの違い
* ApacheのSSL化
* MySQLの外部接続まわり(ファイアウォール・バインド・ユーザー)

## 構築する環境

* Ubuntu 20.04 LTS
* Apache
* PHP 8.0系
* MySQL

使ったツールはこちらです。

* MySQL Workbench(MySQLのGUI)
* さくらのクラウド(サーバー)
* FileZilla

参考にしたサイトです。

* [Linux、Apache、MySQL、PHP(LAMP)スタックをUbuntu 20.04にインストールする方法](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04-ja)
* [Ubuntu 20.04でLet's Encryptを使用してApacheを保護する方法](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04-ja)

## Linuxの注意点

### ディストリビューションを把握しておく

まず、自分が構築するディストリビューションをしっかり把握しておきましょう。
今回はUbuntuですが、ディストリビューションが違うと構築方法も多少変わります。
たとえばUbuntuとCentOSは別物なので、手順をそのまま流用するとハマります。

### コマンドを打つ環境を整える

Windows上でUnixコマンドを使う必要があったので、**Git Bash** を導入しました。
自分が慣れているターミナルがあれば、それを使ってもOKです。

## Apacheの注意点

気をつけるのは **SSL化** だけです。
それ以外は、参考記事のとおりに進めれば問題なく構築できます。

## MySQLの注意点

MySQLで気をつけたいのは、次の3つです。

* ファイアウォール
* バインドの設定
* ユーザーの設定

### ファイアウォールを開ける

初期状態では、外からMySQLに接続できません。
次のコマンドで、MySQLのポートを開放しましょう。

```bash
# ファイアウォールの確認
sudo ufw status

# MySQLの開放
sudo ufw allow 3306
```

### バインドの設定を変える

MySQLは標準だとローカルホストからの接続のみ許可しています。
`/etc/mysql/my.cnf` で `127.0.0.1` を LISTEN しているためです。

外部から接続したいので、次の項目をコメントアウトします。

```
#bind-address            = 127.0.0.1
```

私はここにLinuxのIPアドレスを指定してしまい、接続できずに詰まりました。
理由まで詳しくは調べていませんが、`bind-address` をコメントアウトすると、すべてのアクセスを受け付けるようになります。

### 外部接続用のユーザーを作る

root以外の外部接続用ユーザーを作成します。

```sql
-- DBの作成
CREATE DATABASE データベース名;

-- ユーザーの作成
-- '%' はワイルドカードです。IPアドレスを指定してもOKです
CREATE USER 'ユーザー名'@'%' IDENTIFIED BY 'パスワード';

-- ユーザーに権限付与
GRANT ALL PRIVILEGES ON *.* TO 'ユーザー名'@'%';

-- 更新
FLUSH PRIVILEGES;
```

ここまで設定すれば、MySQL Workbenchから外部接続できるようになります。
phpMyAdminでも何でもいいので、GUIツールを使える状態にしておくと便利ですよ。

なお、`ALL PRIVILEGES ON *.*` はかなり強い権限です。
本番環境では、必要なDBと権限だけに絞りましょう。

## まとめ

* ディストリビューションの違いを把握してから手順を選びます。
* ApacheはSSL化だけ注意すれば大丈夫です。
* MySQLの外部接続は「ファイアウォール → バインド → ユーザー」の3点をセットで確認します。
* PHPerとして、LAMP環境は自分で構築できるようになっておくと安心ですね。
