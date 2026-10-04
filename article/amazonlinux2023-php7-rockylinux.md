---
title: "AmazonLinux2023ではPHP7系が使えない。移行先にRockyLinuxを選んだ話"
emoji: "🪨"
type: "tech"
topics: ["aws", "amazonlinux", "rockylinux", "php", "linux"]
published: false
published_at: "2024-08-20 07:29"
---

> 初出: 2024年8月20日(旧ブログの記事を移行・加筆)

CentOS 7のサポート終了に伴うサーバー移行で、AmazonLinux 2023ではPHP7系が入れられず、RockyLinuxを選んだ経緯をまとめました。
古いPHPのシステムを、サーバーごと移行する必要がある方に向けた内容です。

## 結論

AmazonLinux 2023では、PHP7系はインストールできません。
PHP7系が必須の場合は、Remiリポジトリが使えるRockyLinuxなどを選ぶ必要があります。

## 事の発端

セキュリティサポートが2022年末に終了したPHP7系で動いているシステムを、別のサーバーに移行するタスクが降ってきました。
きっかけは、CentOS 7のサポート終了です。

サポートが切れた技術を使い続けるのは、DevOps系の仕事をしている方なら分かる、かなりの怖さがあります。
そこで、CentOSを別のOSに移行することになりました。

## 困ったこと

クラウドはAWSを使っているので、AWS提供のOSであるAmazonLinux 2023を移行先に決めました。
単純なLAMP環境を作るタスクだったので、作業を進めていくと、1つ問題にぶつかりました。

**PHP7系がインストールできません。**

### RemiリポジトリもEPELも使えない

そもそも、PHP7系は2022年末にサポートが切れているので、AmazonLinuxがサポートしていないのは当然です。
ですが、今回の要件でPHP7系は外せませんでした。

それなら、Remiリポジトリから入れようと考えたのですが、Remiリポジトリも使えませんでした。
(EPELがそもそも使えないためです。)

関連するドキュメントはこちらです。

* [AWS ドキュメント: Fedora との関係](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/relationship-to-fedora.html)
* [AWS ドキュメント: AL2との比較(EPEL)](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/compare-with-al2.html#epel)
* [CentOSなどで使う、RemiRepositoryってなんだ？(Qiita)](https://qiita.com/charon/items/6d34ae798e9b05e8bd0a)

PHP7系をインストールするという要件は、AmazonLinux 2023では達成できないと分かった瞬間でした。

## OSの選定

CentOSの移行先として、次のOSがよく候補に挙がります。

* RedHat Enterprise Linux
* AlmaLinux
* RockyLinux

条件は、次の3つです。

* **PHP7系が使えること**
* **互換性があること**
* **ある程度の知名度があること**

これらを満たすOSとして、RockyLinuxを選びました。
最終的には、RockyLinuxでRemiリポジトリを使って、目的の環境を作れました。

## 感謝を捧げる

振り回された感はありましたが、誰が悪いという話ではないと思います。

* CentOSのサポートが切れたこと
* AmazonLinux 2023がPHP7をサポートしていないこと
* PHP7でいまだに運用しているシステムがあること

どれも、誰かが悪いわけではありません。(PHP7については、早くバージョンアップしてほしいですが…)

最終的に何とかなったので、RockyLinuxのコントリビューターの皆さんに感謝です。

## まとめ

* AmazonLinux 2023では、PHP7系はインストールできません。EPELも使えないため、Remiリポジトリも使えません。
* PHP7系が必須なら、RockyLinuxなど、Remiリポジトリが使えるOSを選びましょう。
* そもそも、PHP7系はサポートが切れているので、早めのバージョンアップをおすすめします。
