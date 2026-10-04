---
title: "EC2のSSHを安全に運用する: EC2 Instance Connectで鍵管理をなくす"
emoji: "🔐"
type: "tech"
topics: ["aws", "ec2", "ssh", "security", "ec2instanceconnect"]
published: false
published_at: "2024-10-28 22:42"
---

> 初出: 2024年10月28日(旧ブログの記事を移行・加筆)

EC2へのSSH接続のセキュリティを高めつつ、運用の手間も減らす方法をまとめました。
EC2にSSH接続して作業している、バックエンド担当の方に向けた内容です。

## 結論

SSH鍵の管理には、**EC2 Instance Connect**を使いましょう。
鍵の発行・ローテーション・ユーザー管理から解放されます。
セキュリティグループは、EC2 Instance Connectのサービス用IP範囲だけに絞ると、さらに安心です。

## EC2のSSH鍵の扱い、どうしている？

複数人でプロジェクトを進めていると、サーバーで作業する人が複数出てきます。
そうなると、次の作業を**作業者の人数 × サーバーの台数**ぶん、行う必要があります。

1. 作業者のユーザーを登録する
2. 各作業者のローカルでSSH鍵を作り、サーバーに公開鍵を登録する

この時点で億劫です。
さらに、鍵の定期的なローテーションや、メンバーの入退場のたびのユーザー管理も発生します。
セキュリティに関心がないメンバーが「ローテーションはしなくていいのでは」と言い出すこともあり、とても面倒です。

## EC2 Instance Connectで解決する

鍵の管理もローテーションも、不要にする方法があります。
それが、AWSのコンソールから、対象のEC2にSSH接続できる**EC2 Instance Connect**です。

この機能は、`~/.ssh/authorized_keys` に**一時的なキー**を付与します。
そのため、ユーザーが鍵を管理する必要がなくなり、運用の手間をかなり減らせます。
サーバー運用では、**管理するものを減らすことが、絶大な効果を生みます。**

APIとCLIにも対応しているので、ちょっとした作業でも使えます。
基本的に、権限さえ付与すれば使える機能です。

### 「使いづらい」と言われたときのTips

「コンソールだとコマンド操作がやりづらい」という声は出てくると思います。
運用の手間を減らすためにも、次のような提案をして、乗り切りましょう。

* **tmux**: 画面分割と、セッションの保持ができます。
* **bash-completion**: 補完が効きます。
* **thefuck**: コマンドの打ち間違いを、いい感じに直してくれます。

参考: [EC2 Instance Connect でインスタンスに接続する](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html)

## セキュリティグループも絞っておく

EC2 Instance Connectを使うときは、セキュリティグループでSSHのポート `22` を開放する必要があります。
22番ポートを開けるのは、少し気が引けますよね。

そこで、最低限のセキュリティとして、**EC2が属するリージョンの、EC2 Instance Connect用のIP範囲だけ**に絞れます。
リージョンごと、サービスごとのIP範囲は、次のJSONにまとまっています。

* https://ip-ranges.amazonaws.com/ip-ranges.json

たとえば、`ap-northeast-1` のEC2のSSH接続を制限するなら、`service` が `EC2_INSTANCE_CONNECT` のレコードの `ip_prefix` を、セキュリティグループのIP範囲に指定します。

```json
{
  "ip_prefix": "3.112.23.0/29",
  "region": "ap-northeast-1",
  "service": "EC2_INSTANCE_CONNECT",
  "network_border_group": "ap-northeast-1"
}
```

これで、**IAMで認証されたユーザーで、かつ東京リージョンのEC2 Instance Connect経由**のSSH接続に制限できます。

:::message
IP範囲は、変更されることがあります。
設定するときは、最新の `ip-ranges.json` を確認してください。
:::

## まとめ

* SSH鍵の管理は、人数 × サーバー台数ぶんの手間がかかります。
* EC2 Instance Connectなら、一時キーが付与されるので、鍵の管理が不要になります。
* セキュリティグループは、EC2 Instance Connectのリージョン別IP範囲に絞りましょう。
* 運用負荷が下がると、幸せになれます。
