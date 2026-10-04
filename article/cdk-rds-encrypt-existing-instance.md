---
title: "CDKで管理している既存のRDSを暗号化する手順(Retain→識別子変更→再作成)"
emoji: "🔒"
type: "tech"
topics: ["aws", "cdk", "rds", "cloudformation", "encryption"]
published: false
published_at: "2026-02-12 15:29"
---

> 初出: 2026年2月12日(旧ブログの記事を移行・加筆)

CDKで管理している既存のRDSを、暗号化済みのRDSに作り直した手順をまとめました。
「既存のRDSを暗号化したい」という要件が来て、CDKでの扱いに困っている方に向けた内容です。

## 結論

**CDKのデプロイを3回行う**ことで、CDK管理のまま、暗号化RDSに移行できました。

1. `RemovalPolicy.RETAIN` を設定して、既存のRDSを保護します。
2. `instanceIdentifier` を変更して、既存のRDSをCDKの管理から切り離します。
3. `storageEncrypted: true` を指定して、新しい暗号化RDSを作ります。

そのあと、`mysqldump` でデータを移行します。

## 今回の要件と制約

* 既存RDSの暗号化(最重要)
* 既存のデータを、そのまま移行できること
* アプリ側の変更を避けるため、**エンドポイント(DNS)が変わらないこと**
* **ダウンタイムは許容**(ALBでメンテナンス画面を出す前提です)
* バッチ処理が動いていない、夜間に作業すること
* **CDKコードの変更は最小限**にして、今後もCDKで管理し続けること

## なぜ困るのか

RDSは、**作成後に暗号化の設定を変更できません。**
既存のRDSを暗号化するには、「暗号化有効」で作り直す必要があります。

* 参考: [Amazon RDS DB インスタンスおよびクラスターの暗号化 - AWS Prescriptive Guidance](https://docs.aws.amazon.com/ja_jp/prescriptive-guidance/latest/patterns/automatically-remediate-unencrypted-amazon-rds-db-instances-and-clusters.html)

さらに、インフラを**AWS CDK**で管理しています。
単純にコードを書き換えてデプロイすると、CloudFormationの仕様で「リソースの置換」が発生して、ダウンタイムやデータ損失のおそれがあります。
一般的な「スナップショットから復元して差し替える」方法も、CDKのState管理との兼ね合いで、今回は採用できませんでした。

### ハマりポイント

CDKで管理しているリソースの `instanceIdentifier`(物理ID)を固定したまま、`storageEncrypted: true` に変更してデプロイすると、次のエラーで失敗します。

```
CloudFormation cannot update a stack when a custom-named resource requires replacing
```

「カスタム名(明示した名前)のついたリソースは、置換(削除して作成)できない」という、CloudFormationの安全装置です。
これを回避するために、あえて識別子を変えて、別のリソースとして認識させる必要があります。

## 手順の詳細

### 1. 削除ポリシーを変更する

まず、既存のRDSが、CDKのスタック操作で削除されないようにします。
これをやらないと、次の手順で「既存RDSの削除」が走って、データが消えます。(超重要です。)

```ts
new rds.DatabaseInstance(this, 'RdsInstance', {
  instanceIdentifier: 'my-prod-db', // 現在の識別子
  // ...
  removalPolicy: cdk.RemovalPolicy.RETAIN, // ここを追加
});
```

この状態で `cdk deploy` します。
これで、このRDSは、スタックから外れても、AWS上にリソースが残るようになります。

### 2. instanceIdentifier を変更する

次に、既存のRDSを、CDKの管理から外す(Orphanにする)作業です。
CDK上の識別子を、既存のものとは別の名前に書き換えます。

```ts
new rds.DatabaseInstance(this, 'RdsInstance', {
  instanceIdentifier: 'my-prod-db-temp', // 手順3と被らない、一時的な名前に変更
  // ...
  removalPolicy: cdk.RemovalPolicy.RETAIN,
});
```

この状態で `cdk deploy` します。

**何が起きるか**: CloudFormationは「`my-prod-db` を管理から外して、新しく `my-prod-db-temp` を作る」という動きをします。
手順1で `RETAIN` にしているので、**旧RDS(データ入り)は削除されず、そのまま残ります。**
この時点では、アプリはまだ旧RDSを見ています。

### 3. 暗号化RDSを構築する

最後に、本命の「暗号化されたRDS」を作ります。

```ts
new rds.DatabaseInstance(this, 'RdsInstance', {
  instanceIdentifier: 'my-prod-db', // 元の名前に戻す(変わるとエンドポイントが変わるので注意)
  storageEncrypted: true,           // 暗号化をON
  removalPolicy: cdk.RemovalPolicy.RETAIN,
  // ...
});
```

この状態で `cdk deploy` します。

:::message
手順1と同じ名前(`my-prod-db`)に戻したい場合、手順2のあとにも、AWS上に古い `my-prod-db` が残っているので、**名前が重複してエラー**になる可能性があります。
その場合は、**マネジメントコンソールまたはCLIから、古いRDS(手順2で切り離されたもの)の識別子を、`my-prod-db-old` などにリネーム**してから、デプロイしてください。
:::

これで、暗号化された空のRDSが立ち上がります。

## データ移行

新旧2つのRDSがある状態になりました。

* **旧RDS**: データあり、非暗号化、CDK管理外
* **新RDS**: データなし、暗号化済み、CDK管理下

データ量が、数GB〜数十GB程度と多くなかったので、シンプルに `mysqldump` で移行しました。
TB級になる場合は、AWS DMS(Database Migration Service)を検討してください。

```bash
# SSM Session Managerなどで、踏み台サーバーに接続して実行

# 1. 旧RDSからデータをエクスポート(パイプで圧縮して転送時間を短縮)
mysqldump -h <旧RDSエンドポイント> -u <ユーザー名> -p \
  --single-transaction --routines --triggers \
  --databases <対象のスキーマ> \
  | gzip > dump.sql.gz

# 2. 新RDSにインポート
zcat dump.sql.gz | mysql -h <新RDSエンドポイント> -u <ユーザー名> -p
```

データの移行が終わったら、アプリの接続先が新RDSを向いていることを確認して、メンテナンスを解除します。

## 後片付け

稼働確認が取れたら、不要なリソースを削除します。

1. **手順2で立ち上がった、一時的なRDS**(CDKの管理から外れているはずですが、念のため確認して削除します)
2. **手順1以前からあった旧RDS**(リネームした `my-prod-db-old` など)

スナップショットが取れていることを確認したうえで、マネジメントコンソールから削除しました。

## おまけ: 試したけどうまくいかなかったこと

### `cdk import` での既存リソースの取り込み

既存のリソースをリネームして、新しいスタックに取り込もうと、`cdk import` を試しました。
しかし、次のメッセージが出て、スキップされました。

```
TmpStack: no new resources compared to the currently deployed stack, skipping import.
```

また、定義と実リソースのプロパティ(暗号化の有無)が食い違っているので、インポート自体が整合性エラーになる可能性が高いです。

### `DatabaseInstanceFromSnapshot` の利用

スナップショットから復元するCDKのクラスですが、これをメインの定義にすると、常に「スナップショットから作られた状態」が正になります。
パラメータグループやバージョン更新などの運用で、ドリフト(設定の乖離)の管理が面倒になると考えて、今回は見送りました。

## まとめ

* RDSは、作成後に暗号化の設定を変更できません。
* CDK管理のRDSは、「Retainで保護 → Identifierを変更して管理から切り離し → 新規作成」の3回のデプロイで移行できます。
* RDSに限らず、ほかのステートフルなリソースにも応用できるテクニックです。
* RDSは、最初から暗号化しておきましょう。
