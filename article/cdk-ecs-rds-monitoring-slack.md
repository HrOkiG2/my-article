---
title: "CDKでECS/RDSの監視・オートスケール基盤を構築してSlack通知する"
emoji: "📣"
type: "tech"
topics: ["aws", "cdk", "ecs", "rds", "slack"]
published: false
published_at: "2025-11-19 13:44"
---

> 初出: 2025年11月19日(旧ブログの記事を移行・加筆)

AWS CDKで、ECSとRDSの監視アラーム、ECSのオートスケーリング、Slackへの通知基盤を、まとめて構築する方法をまとめました。
ECS FargateとRDSでPHPアプリを運用していて、監視と通知をIaCにしたい方に向けた内容です。

## 結論

CloudWatchアラームとSNSとAWS Chatbotで、アラームをSlackに集約します。
ECSのオートスケーリングと、そのイベント通知も、同じCDKスタックで定義できます。

## 監視・通知アーキテクチャの全体像

| 機能 | AWSサービス | 役割 |
| --- | --- | --- |
| 監視 | CloudWatch | メトリクスを収集して、アラームを発報します。 |
| 自動応答 | Application Auto Scaling | ECSの負荷に応じて、タスク数を自動調整します。 |
| 通知ルーティング | Amazon SNS | アラームやスケーリングイベントを中継します。 |
| 通知先 | AWS Chatbot → Slack | SNSからの通知を整形して、Slackに送ります。 |

## 事前準備: AWS ChatbotとSlackの連携

CDKでデプロイする前に、**AWS Chatbotのクライアント**を、手動で作っておく必要があります。

1. AWS Chatbotのコンソールで、**Slackワークスペース**と**チャンネル**を連携させます。
2. この設定で、通知先の**SlackチャンネルID**と**ワークスペースID**が取得できます。CDKのコードで使います。

:::message
**CDKだけでChatbotの連携が完結しない理由**

* **外部サービスとの認証(OAuth)が必要**: Slack側でAWS Chatbotアプリをインストールして、アクセスを許可する認証を、管理者がブラウザで行う必要があります。CDKは、Slack側の認証フローを、自動では実行できません。
* **Chatbotクライアントの事前設定が必要**: CDKの `aws-chatbot` モジュールで定義できるのは、連携済みのワークスペースを参照して、チャンネル設定(どのSNSトピックの通知を、どのチャンネルに送るか)をする部分だけです。ワークスペースIDは、コンソールで連携を完了して、初めてAWS側に登録されます。
:::

## 監視とオートスケール基盤の構築

ECSサービスとRDSインスタンスが、すでにあることを前提に、監視とスケールの設定を定義するCDKスタックを作ります。

### CDKスタックのセットアップ

```ts
// lib/monitoring-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as sns from 'aws-cdk-lib/aws-sns';
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch';
import * as actions from 'aws-cdk-lib/aws-cloudwatch-actions';
import * as events from 'aws-cdk-lib/aws-events';
import * as targets from 'aws-cdk-lib/aws-events-targets';
import * as autoscaling from 'aws-cdk-lib/aws-applicationautoscaling';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as chatbot from 'aws-cdk-lib/aws-chatbot';

// 環境依存の定数(適宜置き換える)
const CLUSTER_NAME = 'your-ecs-cluster-name';
const SERVICE_NAME = 'your-ecs-service-name';
const RDS_IDENTIFIER = 'your-rds-instance-identifier';
const SLACK_CHANNEL_ID = 'C01234567';   // 取得したチャンネルID
const SLACK_WORKSPACE_ID = 'T01234567'; // 取得したワークスペースID

export class MonitoringStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // 既存のECSサービスを参照
    const ecsService = ecs.FargateService.fromFargateServiceAttributes(this, 'ExistingEcsService', {
      serviceArn: cdk.Stack.of(this).formatArn({
        service: 'ecs',
        resource: 'service',
        resourceName: `${CLUSTER_NAME}/${SERVICE_NAME}`,
        arnFormat: cdk.ArnFormat.SLASH_RESOURCE_NAME,
      }),
      cluster: ecs.Cluster.fromClusterAttributes(this, 'ExistingCluster', {
        clusterName: CLUSTER_NAME,
        vpc: ({} as any), // 参照のための空オブジェクト
        securityGroups: [],
      }),
    });
```

### 通知用SNSトピックとChatbotの設定

アラームやイベントが通知を送るSNSトピックを作って、AWS Chatbotに連携させます。

```ts
    // 1. 通知用SNSトピックの作成
    const alarmTopic = new sns.Topic(this, 'AlarmTopic', {
      displayName: 'System-Monitoring-Notifications',
    });

    // 2. AWS ChatbotとSNSトピックの連携(Slackへのルーティング)
    new chatbot.SlackChannelConfiguration(this, 'ChatbotConfig', {
      slackChannelConfigurationName: 'EcsRdsMonitoring',
      slackWorkspaceId: SLACK_WORKSPACE_ID,
      slackChannelId: SLACK_CHANNEL_ID,
      notificationTopics: [alarmTopic], // このトピックへの通知が、Slackに送られる
    });
```

### ECSとRDSの監視アラーム

主要なメトリクスにCloudWatchアラームを作って、アクションに、上のSNSトピックを設定します。

```ts
    // --- ECS Fargate アラーム ---
    // ECSのCPU使用率が80%を超えたらアラーム
    const ecsCpuAlarm = new cloudwatch.Alarm(this, 'EcsCpuHighAlarm', {
      metric: cloudwatch.Metric.fromNamespaceAndName('AWS/ECS', 'CPUUtilization').with({
        dimensionsMap: { ClusterName: CLUSTER_NAME, ServiceName: SERVICE_NAME },
      }),
      threshold: 80,
      evaluationPeriods: 2,
      comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_OR_EQUAL_TO_THRESHOLD,
      alarmDescription: 'ECS Service CPU utilization is too high.',
    });
    ecsCpuAlarm.addAlarmAction(new actions.SnsAction(alarmTopic));
    // 復旧時も通知したい場合は、次を追加
    // ecsCpuAlarm.addOkAction(new actions.SnsAction(alarmTopic));

    // --- RDS アラーム ---
    // RDSのCPU使用率が70%を超えたらアラーム
    const rdsCpuAlarm = new cloudwatch.Alarm(this, 'RdsCpuHighAlarm', {
      metric: cloudwatch.Metric.fromNamespaceAndName('AWS/RDS', 'CPUUtilization').with({
        dimensionsMap: { DBInstanceIdentifier: RDS_IDENTIFIER },
      }),
      threshold: 70,
      evaluationPeriods: 3,
      comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_OR_EQUAL_TO_THRESHOLD,
      alarmDescription: 'RDS CPU utilization is too high.',
    });
    rdsCpuAlarm.addAlarmAction(new actions.SnsAction(alarmTopic));
```

### ECSサービスのオートスケーリング

ECSサービスを、CPU使用率60%を目標に、自動でスケーリングするよう設定します。

```ts
    // 3. ECSサービスのオートスケーリング設定
    // サービスのDesired Countをスケーリング対象にする
    const scalableTarget = autoscaling.ScalableTarget.forEcsService(this, 'EcsScalableTarget', {
      service: ecsService,
      minCapacity: 1, // 最小タスク数
      maxCapacity: 5, // 最大タスク数
    });

    // 目標CPU使用率60%を維持する、ターゲット追跡スケーリングポリシー
    scalableTarget.scaleToTrackMetric('CpuScalingPolicy', {
      metric: ecsService.metricCpuUtilization(),
      targetValue: 60,
      scaleOutCooldown: cdk.Duration.seconds(60),
      scaleInCooldown: cdk.Duration.seconds(300),
    });
```

### オートスケールイベントのSlack通知

スケーリング操作の成功イベントを、EventBridgeで捕捉して、Slackに通知します。

```ts
    // 4. オートスケールイベントを捕捉して、SNSトピックにルーティング
    new events.Rule(this, 'AutoScalingNotificationRule', {
      description: 'Notify Slack on successful ECS Auto Scaling events.',
      eventPattern: {
        source: ['aws.application-autoscaling'],
        detailType: ['Application Auto Scaling Policy Execution'],
        detail: {
          // 成功したスケーリング操作のみを通知
          statusCode: ['Successful'],
        },
      },
      // ターゲットは、既存のSNSトピック
      targets: [new targets.SnsTopic(alarmTopic)],
    });
  }
}
```

## まとめ

CDKを使うと、TypeScriptのコードで、次の運用基盤を構築できます。

1. ECS/RDSの重要なメトリクスの監視
2. CloudWatch AlarmsとChatbotによる、Slackへの即時通知
3. ECSサービスの負荷に応じた、自動スケーリング(スケールイン/アウト)
4. スケーリング操作の完了通知

インフラをコード化すると、再現性、バージョン管理、レビューができるようになり、システムの信頼性が大きく向上します。
