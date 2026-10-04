---
title: "ALBでHTTPS化したのに、なぜECSコンテナはポート80でOKなのか"
emoji: "🔀"
type: "tech"
topics: ["aws", "alb", "ecs", "cdk", "https"]
published: false
published_at: "2025-10-17 10:45"
---

> 初出: 2025年10月17日(旧ブログの記事を移行・加筆)

ALBでHTTPS化しているのに、後ろのECSコンテナがポート80(HTTP)で通信できる理由を、セキュリティグループとあわせて解説します。
ALB + ECSの構成を作っていて、この仕組みが腑に落ちていない方に向けた内容です。

## 結論

ALBが、**「リダイレクトの案内」と「リクエストの取り次ぎ(SSL/TLS終端)」**という2つの仕事をしているからです。
コンテナには、暗号化が解かれたHTTPリクエストだけが届くので、ポート80で待ち受ければ十分です。

## 目指す構成: 安全な3層アーキテクチャ

* **Web層**: ALBが、インターネットからのリクエストをすべて受け付けます。
* **AP層**: ECSコンテナが、プライベートな領域でアプリケーションを動かします。
* **DB層**: RDSが、さらに内側のプライベートな領域で、データを安全に保管します。

この構成のセキュリティの肝は、「**ECSはALBからの通信しか受け付けない**」というルールです。
これを実現するのが、セキュリティグループです。

## 門番の役割を担う「セキュリティグループ」

セキュリティグループは、リソースのドアを守る、バーチャルな門番です。今回は3つ配置します。

1. **ALBの門番**: インターネットからの訪問者(ポート80、443)を、全員受け入れます。
2. **ECSの門番**: **ALBの門番から紹介された人**(ALBからのトラフィック)**だけ**を通します。
3. **RDSの門番**: **ECSの門番から紹介された人**(ECSからのトラフィック)**だけ**を通します。

この連携で、攻撃者がECSに直接アクセスしようとしても、門前払いになります。

## ALBがこなす、2つの全く異なる仕事

ユーザーが `http://` でアクセスしたときに、ALBとコンテナの間では、**2段階の独立した処理**が行われます。

### ステップ1: ブラウザへの「リダイレクト指示」

最初のやり取りは、ユーザーのブラウザとALBの間だけで完結します。

1. **ユーザー**: ブラウザで `http://example.com`(ポート80)にアクセスします。
2. **ALB(ポート80の受付係)**: リクエストを受け取ると、中身は見ずに、「こちらの窓口は安全ではありません。向かいにある、安全な443番の窓口に行き直してください」と応答します。これが**HTTP 301リダイレクト**です。

**この時点では、リクエストはECSコンテナに、まだ一切届いていません。**

### ステップ2: コンテナへの「リクエスト転送」

ブラウザは、ALBの指示に従って、今度は最初から安全な窓口に向かいます。

1. **ユーザー**: ブラウザが、自動で `https://example.com`(ポート443)に、**新しいリクエスト**を送ります。
2. **ALB(ポート443の受付係)**:
   * 暗号化されたリクエストを受け取って、持っているSSL証明書で**通信を復号**します(SSL/TLS終端)。
   * 平文になったリクエストを、**ECSコンテナのポート80**に転送(フォワード)します。
3. **ECSコンテナ(ポート80)**: ALBから来た通常のHTTPリクエストを受け取って、処理します。

ALBは、**ユーザーを正しい入口に案内する「案内係」**と、**リクエストを内部の担当者に届ける「取次係」**という、2つの役割をこなしています。
コンテナから見ると、常にALBという信頼できる取次係から、暗号化が解かれたリクエストが届くだけです。だから、ポート80で待ち受ければよいのです。

## CDKでの実装

### セキュリティグループ

```ts
// ALB用セキュリティグループ
const albSecurityGroup = new ec2.SecurityGroup(this, 'AlbSg', { vpc });
albSecurityGroup.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(80), 'Allow HTTP');
albSecurityGroup.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(443), 'Allow HTTPS');

// ECS用セキュリティグループ
const ecsSecurityGroup = new ec2.SecurityGroup(this, 'EcsSg', { vpc });

// ALBからの通信のみを、コンテナのポート80で許可
ecsSecurityGroup.addIngressRule(
  albSecurityGroup,
  ec2.Port.tcp(80),
  'Allow traffic only from ALB'
);
```

### ALBリスナー

```ts
// ALB本体を作成
const alb = new elbv2.ApplicationLoadBalancer(this, 'MyAlb', {
  vpc,
  internetFacing: true,
  securityGroup: albSecurityGroup,
});

// HTTPリスナー(ポート80): HTTPSにリダイレクトするだけ
alb.addListener('HttpListener', {
  port: 80,
  defaultAction: elbv2.ListenerAction.redirect({
    protocol: 'HTTPS',
    port: '443',
    permanent: true,
  }),
});

// HTTPSリスナー(ポート443): ECSにリクエストを転送する
alb.addListener('HttpsListener', {
  port: 443,
  certificates: [certificate], // ACMで発行した証明書
  defaultAction: elbv2.ListenerAction.forward([ecsTargetGroup]), // ECSのターゲットグループ
});
```

## まとめ

* ALBのHTTP→HTTPSリダイレクトは、「リダイレクト指示」と「リクエスト転送」の2段階です。
* SSL/TLSの暗号化・復号は、ALBがすべて肩代わりします(SSL/TLS終端)。
* コンテナは、暗号化されていないHTTPリクエストを受け取るだけでよいです。
* セキュリティグループを正しく設定して、この安全な通信経路が完成します。

参考:

* [What is an Application Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
* [Listeners for your Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html)
