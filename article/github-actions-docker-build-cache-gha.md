---
title: "GitHub ActionsでDockerイメージのビルドキャッシュに type=gha を選んだ理由"
emoji: "⚡"
type: "tech"
topics: ["githubactions", "docker", "ecs", "cicd", "buildx"]
published: false
published_at: "2026-02-12 15:41"
---

> 初出: 2026年2月12日(旧ブログの記事を移行・加筆)

GitHub Actionsで、ECSへのデプロイワークフローを実装したときの、Dockerイメージのビルドキャッシュ戦略をまとめました。
CIのビルド時間を短縮したい方や、キャッシュの置き場所で迷っている方に向けた内容です。

## 結論

キャッシュのバックエンドには、**GitHub Actions Cache(`type=gha`)**を選びました。
理由は、**利用料(ランニングコスト)と、実装(設定)コストが、最も低かった**からです。

## 背景

業務で、ECSへのデプロイワークフローを実装することになりました。
主な流れは、次のとおりです。

1. フロントエンドのビルド
2. **Dockerイメージのビルド**
3. ECRへのログイン
4. ECRへのPush
5. ECSのサービス更新

この中でボトルネックになるのが、**2. Dockerイメージのビルド**です。
毎回フルビルドすると時間がかかりすぎるので、キャッシュの利用は必須です。

Docker Buildxのキャッシュバックエンドには、大きく3つの選択肢があります。

1. **GitHub Actions Cache**(`type=gha`)
2. **GitHub Container Registry**(`type=registry`)
3. **Amazon ECR**(`type=registry`)

## 比較検討: 3つのキャッシュ戦略

### 1. Amazon ECR(`type=registry`)

AWS環境に統一できるメリットはありますが、次のデメリットがあります。

* **実装コスト**: IAMロールの設定と、`aws-actions/amazon-ecr-login` の設定が必要です。
* **利用料**
  * **データ転送量**: GitHub Actions(インターネット)とAWS ECRの間で、通信が発生します。
  * **ストレージ料金**: キャッシュイメージの分だけ、ECRのストレージ料金がかかります。
  * **実行時間**: ネットワーク越しにPush/Pullするので、ビルド時間が伸びて、GitHub Actionsの課金対象時間(分)も消費します。

### 2. GitHub Container Registry(`type=registry`)

GitHubのエコシステム内で完結しますが、こちらも考慮が必要です。

* **実装コスト**: `docker/login-action` での認証設定や、パッケージの読み書き権限の設定が必要です。
* **利用料**: Organizationのプランによっては、GHCRのストレージ容量やデータ転送量に、制限や課金が発生する場合があるようです。

### 3. GitHub Actions Cache(`type=gha`)

今回採用した方法です。

* **実装コスト**: **認証設定が一切不要**です。YAMLに数行書くだけです。
* **利用料**
  * **通信費ゼロ**: GitHub Actionsの内部ネットワークで完結するので、外部への転送コストがかかりません。
  * **ストレージは無料**: リポジトリあたり10GBまで使えます。超過分は、古いものから自動で削除されるので、追加課金のリスクがありません。
  * **実行時間の短縮**: ネットワークが速く、キャッシュのリストアが速いので、結果としてActionsの実行時間を節約できます。

## 比較まとめ

| キャッシュ戦略 | 実装(設定)コスト | 利用料(ランニングコスト) | 速度 |
| --- | --- | --- | --- |
| **GHA Cache**(`type=gha`) | ◎(認証不要) | ◎(基本無料) | ◎(最速) |
| **GHCR**(`type=registry`) | △(Token設定が必要) | ◯(プラン依存) | ◯ |
| **ECR**(`type=registry`) | △(IAM設定が必要) | △(通信費・保管費) | △(通信が発生) |

## なぜ type=gha を選んだか

今回の要件で重視したのは、次の2点です。

1. **ワークフローの利用料を抑えたい**
   * 無駄なデータ転送費やストレージ料金を払いたくありません。
   * ビルド時間を短縮して、GitHub Actionsの利用枠(分)を節約したいです。
2. **実装コストを最小限にしたい**
   * キャッシュのためだけに、IAM権限を調整したり、Secretsを管理したりする手間を省きたいです。
   * シンプルに保って、メンテナンスしやすくしたいです。

この2点を満たすのが、`type=gha` でした。
「ローカル開発環境でも、同じキャッシュを使いたい」という要件があれば、`registry` 系が候補に挙がります。
ですが、今回は「CIの高速化」が主目的だったので、迷わずこちらを選びました。

## 実装コード

設定は、驚くほどシンプルです。
`docker/build-push-action` に、`cache-from` と `cache-to` を追加するだけです。

```yaml
# Buildxのセットアップ(必須)
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

# ビルド & Push
- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ${{ steps.login-ecr.outputs.registry }}/my-app:latest
    # ★ ここがポイント
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### ポイント: mode=max

`cache-to: type=gha,mode=max` を指定すると、最終的なイメージだけでなく、**中間レイヤー**(`npm install` などを行ったレイヤー)もキャッシュされます。
これで、コードを少し変更しただけで、フルビルドが走るのを防げます。

## まとめ

* 技術選定では、高機能さも大事ですが、「コストがかからない」「設定が楽」は、運用を続けるうえで正義です。
* ECSデプロイのビルド時間に悩んでいるなら、まずは、一番手軽で財布に優しい `type=gha` から試してみましょう。
* `mode=max` で、中間レイヤーもキャッシュしましょう。
