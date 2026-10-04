---
title: "例外の型でエラー通知の緊急度を自動で振り分けるPHP実装"
emoji: "🔔"
type: "tech"
topics: ["php", "exception", "slack", "errorhandling", "operations"]
published: false
published_at: "2026-01-06 13:55"
---

> 初出: 2026年1月6日(旧ブログの記事を移行・加筆)

例外の種類によって、Slack通知やエラー画面を自動で振り分ける方法をまとめました。
エラー通知が多すぎて、重要な通知を見逃しがちな開発チームの方に向けた実践編です。
(例外クラスの使い分けは、前回の記事「とりあえずExceptionにしてない？PHPの例外クラスの正しい使い分け」を参照してください。)

## 結論

`LogicException`(バグ)と `RuntimeException`(環境エラー)を使い分けて、例外ハンドラーで振り分けましょう。
Slack通知が来たら「即対応」と決められて、運用がぐっと楽になります。

## すべてのエラーを通知していませんか？

開発現場でよくあるのが、「エラーが出たら全部Slackに通知！」という運用です。
最初はいいのですが、だんだんこうなりませんか？

* 深夜に「DB接続タイムアウト」の通知で起こされる(でも、数秒後に自然復旧している)
* 通知が多すぎて、本当に重要なバグ報告を見逃す
* 結果、「通知チャンネルを誰も見なくなる」

原因は、**「バグ(緊急)」**と**「環境トラブル(要確認)」**を混ぜてしまっていることです。

## 例外の型で振り分ける実装

例外ハンドラー(エラーを一元管理するクラス)で、次のように振り分けます。

```php
use Psr\Log\LoggerInterface;

class AppExceptionHandler
{
    public function __construct(
        private LoggerInterface $logger,
        private SlackNotifier $slack
    ) {}

    public function handle(\Throwable $e): void
    {
        // パターン1: LogicException / DomainException
        // 意味: 「コードがおかしい(バグ)」
        if ($e instanceof \LogicException) {
            // これは緊急事態。開発者にすぐ知らせる
            $this->logger->critical($e->getMessage());
            $this->slack->send("🚨 緊急: 実装バグが発生！すぐ直して！: " . $e->getMessage());

            // ユーザーには「システムエラー」とだけ表示
            $this->renderErrorPage(500);
            return;
        }

        // パターン2: RuntimeException
        // 意味: 「外部要因で失敗(環境エラー)」
        if ($e instanceof \RuntimeException) {
            // ログには残すが、深夜に起こすほどではない
            $this->logger->error($e->getMessage());

            // ユーザーには「現在混み合っています」など、やんわり伝える
            $this->renderErrorPage(503);
            return;
        }

        // その他の予期せぬエラー
        $this->logger->error('Unknown Error: ' . $e->getMessage());
        $this->renderErrorPage(500);
    }
}
```

`DomainException` は `LogicException` のサブクラスなので、`instanceof \LogicException` でまとめて拾えます。

## この設計のメリット

1. **Slack通知が来たら「即対応」できる**: `LogicException` だけが通知されるので、「通知 = 自分たちのコードのミス」だと分かります。
2. **ログがきれいになる**: 一時的な接続エラーなどは、ログファイルに溜まるだけです。精神衛生上もよいです。(もちろん、定期的なチェックは必要です。)
3. **ユーザーへの案内が親切になる**: バグなら `500`、アクセス過多なら `503` と、状況に合ったHTTPステータスコードを返せます。

## 例外処理は「未来の自分」へのメッセージ

例外クラスの使い分けは、単なるルールではありません。

* `LogicException` を投げるときは、「これはバグだから、絶対直してね！」というメッセージです。
* `RuntimeException` を投げるときは、「運用でカバーしてね」というメッセージです。

意図を込めてコードを書けるようになると、一人前のエンジニアに近づけます。
エラーハンドリングは地味な部分ですが、こだわるとかっこいいですよ。

## まとめ

* 全エラーをSlack通知すると、重要な通知が埋もれます。
* `LogicException` だけを緊急通知して、`RuntimeException` はログとエラー画面で対応しましょう。
* 例外の型と、HTTPステータスコード(500/503)を対応させると、ユーザーへの案内も親切になります。
* ぜひ、自分たちのプロジェクトへの導入を検討してみてください。
