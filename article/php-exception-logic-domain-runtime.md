---
title: "とりあえずExceptionにしてない？PHPの例外クラスの正しい使い分け"
emoji: "🚦"
type: "tech"
topics: ["php", "exception", "spl", "errorhandling"]
published: false
published_at: "2026-01-06 13:46"
---

> 初出: 2026年1月6日(旧ブログの記事を移行・加筆)

PHPの標準例外(SPL)のうち、`LogicException`、`DomainException`、`RuntimeException` の使い分けをまとめました。
「とりあえず `Exception` を投げている」PHPerの方に向けた内容です。

## 結論

例外クラスを使い分ける目的は、**「誰のせいなのか」をはっきりさせる**ことです。

| 系統 | 意味 | 対応 |
| --- | --- | --- |
| `LogicException` / `DomainException` | コードのバグ | コードを修正する(`catch` して無視しない) |
| `RuntimeException` | 実行時のトラブル(環境要因) | エラー画面やログで、運用でカバーする |

## なぜ使い分ける必要があるの？

エラーが起きたときに、次のどちらなのかで、対応が変わります。

* **プログラマーのミス(バグ)なら**: コードを修正しないといけません。
* **環境要因(運悪くサーバーが落ちたなど)なら**: リトライしたり、ユーザーに「混み合っています」と伝えたりします。

これをクラスの型で表現するために、使い分けが必要です。

## LogicException(ロジック例外)

**「これは開発者のミス！ コードを直して！」**

プログラムの論理がおかしいときに投げる例外です。
基本的に、**本番でこれが出たらアウト**で、すぐ修正が必要です。

たとえば、APIキーをセットしていないのに、リクエストを送ろうとしたケースです。

```php
class ApiClient
{
    private ?string $apiKey = null;

    public function setApiKey(string $key): void
    {
        $this->apiKey = $key;
    }

    public function request(): void
    {
        if ($this->apiKey === null) {
            // 開発者がセットアップ手順を間違えているため、LogicException
            throw new \LogicException('API Keyが設定されていません。request()の前にsetApiKey()を呼んでください。');
        }
        // リクエスト処理...
    }
}
```

この例外が出たら、`catch` して握りつぶさずに、コード自体を直しましょう。

## DomainException(ドメイン例外)

**「そのデータ、仕様としてありえないよ！」**

`LogicException` の親戚(サブクラス)です。
ロジックは動くけれど、渡されたデータが、ビジネスルール(仕様)としてありえない場合に使います。

たとえば「信号機の色」を扱うクラスに、「紫」が渡されたら困りますよね。

```php
class TrafficLight
{
    private const ALLOWED_COLORS = ['red', 'yellow', 'blue'];

    public function __construct(private string $color)
    {
        if (!in_array($color, self::ALLOWED_COLORS, true)) {
            // 定義外の値がコードから渡された！
            throw new \DomainException("色 '{$color}' は信号機に存在しません。");
        }
    }
}
```

ユーザーの入力ミスというより、**コード上で変な値を渡してしまったとき**に検知するためのものです。

## RuntimeException(実行時例外)

**「コードは合ってるけど、環境のせいで失敗しました…」**

いちばんよく使います。
コードは正しいけれど、「DBに繋がらない」「ファイルがない」「外部APIが落ちている」など、**実行時の状況**によって起きるエラーです。

```php
class ConfigLoader
{
    public function load(string $filePath): array
    {
        if (!file_exists($filePath)) {
            // パスは合っているはずだが、ファイルが消えている(実行時の事情)
            throw new \RuntimeException("設定ファイルが見つかりません: {$filePath}");
        }
        // 読み込み処理...
    }
}
```

このエラーが出たときは、`try-catch` で捕まえて、「ただいま混み合っております」のようなエラー画面を出してあげる必要があります。

## まとめ

* **Logic / Domain 系**: 「コードのバグ」です。コードを修正しましょう。`catch` して無視してはいけません。
* **Runtime 系**: 「実行時のトラブル」です。エラー画面やログで、運用でカバーします。

この意識を持つだけで、エラーが起きたときに「何をすればいいか」がすぐ分かるようになります。
ぜひ、明日からのコーディングで意識してみてください。
