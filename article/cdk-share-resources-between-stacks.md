---
title: "CDKでスタック間のリソースを共有する: IDではなくオブジェクトを渡す"
emoji: "🏗️"
type: "tech"
topics: ["aws", "cdk", "typescript", "ecs", "lamp"]
published: false
published_at: "2026-01-14 15:26"
---

> 初出: 2026年1月14日(旧ブログの記事を移行・加筆)

AWS CDKで、スタックを分割したときの、スタック間のリソース共有の方法をまとめました。
LAMP環境のような実務の構成を、CDKで作ろうとしている方に向けた内容です。

## 結論

スタック間でVPCなどを共有するときは、**IDの文字列ではなく、オブジェクトそのもの**を渡しましょう。
型安全の恩恵を受けられて、記述も楽になります。

## 困りごと

ネット上の「CDKでECSを構築してみた！」系の記事は、VPCもECSもRDSも、1つの `lib/my-stack.ts` に書かれていることが多いです。
サンプルとしては分かりやすいのですが、実務でLAMP環境を作ろうとすると、スタックを分けたくなります。

* VPCは更新頻度が低いので、スタックを分けたい
* RDSはステートフルなので、独立させたい
* ECSはアプリのデプロイで頻繁に更新が入るかもしれない

でも、分割した途端に、「StackAで作ったVPCのIDを、どうやってStackBに渡すの?」という問題が出ます。

## IDではなくオブジェクトを渡す

VPCスタックとECSスタックを分けるときに、つい「**VPC IDをPropsで渡す**」方法を取りがちです。
Terraformに慣れていると、この発想になりやすいですよね。

ただ、この方法ではCDKの型安全を活かせません。
私は、**VPCオブジェクトそのもの**を渡すのがベストだと考えています。

構成は、エントリーポイントの `app` が、バケツリレーのようにオブジェクトを渡すイメージです。

1. **VpcStack**: VPCを作って、`public readonly vpc` でVPCオブジェクトを公開します。
2. **app**: VpcStackからVPCオブジェクトをもらって、EcsStackに渡します。
3. **EcsStack**: 受け取ったVPCオブジェクトで、クラスターを作ります。

## 型安全の恩恵を受ける

### VpcStack(渡す側)

`this.vpc` を、クラスのメンバ変数として公開します。

```ts
// lib/vpc-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import { Construct } from 'constructs';

export class VpcStack extends cdk.Stack {
  // 外部に公開するプロパティ(これが重要)
  public readonly vpc: ec2.IVpc;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    this.vpc = new ec2.Vpc(this, 'LampVpc', {
      maxAzs: 2,
    });
  }
}
```

### EcsStack(受け取る側)

ここがポイントです。
`vpcId: string` ではなく、`vpc: ec2.IVpc` というインターフェース型で受け取ります。

```ts
// lib/ecs-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import { Construct } from 'constructs';

// Propsの定義: 文字列ではなく「VPCの型」を指定する
interface EcsStackProps extends cdk.StackProps {
  vpc: ec2.IVpc;
}

export class EcsStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: EcsStackProps) {
    super(scope, id, props);

    // 文字列から検索する必要はなく、そのまま使える
    const cluster = new ecs.Cluster(this, 'LampCluster', {
      vpc: props.vpc,
    });
  }
}
```

### bin/app.ts(つなぐ場所)

最後に、エントリーポイントでひも付けます。

```ts
// bin/app.ts
import * as cdk from 'aws-cdk-lib';
import { VpcStack } from '../lib/vpc-stack';
import { EcsStack } from '../lib/ecs-stack';

const app = new cdk.App();

// 1. VPCを作る
const vpcStack = new VpcStack(app, 'VpcStack');

// 2. VPCオブジェクトをECSスタックに渡す
const ecsStack = new EcsStack(app, 'EcsStack', {
  vpc: vpcStack.vpc, // ここでバケツリレー
});
```

### この方法のメリット

一言で言うと、プログラミング言語による抽象化です。メリットは2つあります。

1. **圧倒的な型安全性**: 間違えてS3のオブジェクトを渡そうとすると、エディタがその場でエラーにしてくれます。「デプロイしたらIDが違って失敗した」という悲劇を、未然に防げます。
2. **記述がとても楽**: 受け取る側で `ec2.Vpc.fromLookup()` のようなインポート処理を書く必要がありません。渡された瞬間から、`props.vpc.addInterfaceEndpoint` のようにメソッドが使えます。

## 議論したいポイント

ここまで「これが正解」という顔で書いてきましたが、この構成には、はっきりしたデメリットもあります。
それは、**スタック同士が密結合になる**ことです。

CloudFormationの `Export/Import` 機能で強く結びつくため、次のような問題が出ます。

* スタック間の参照があるので、「VPCだけを作り直したい」ができません。

ただ、私はこの問題が出ても、LAMP環境という1つのアプリケーションの中では、VPCとECSは「運命共同体」なので、問題ないと考えています。
「ECSが動いているのに、VPCだけ消したい」という状況は、まれだと思います。
そのため、**この密結合は、あえて受け入れる仕様**だと割り切っています。

ただし、組織の基盤となる共通VPCを作る場合は、ライフサイクルが違うので、疎結合なID渡しのほうが正解かもしれません。

みなさんの構成も、ぜひ教えてください。

## まとめ

* スタックを分割するときは、VPCなどのリソースを、IDではなくオブジェクトで渡しましょう。
* 型安全の恩恵を受けられて、`fromLookup()` のような記述も不要になります。
* 代わりに、スタック間は密結合になります。1つのアプリ内なら割り切るのも手です。
* 組織共通の基盤リソースは、ライフサイクルが違うので、疎結合なID渡しも検討しましょう。
