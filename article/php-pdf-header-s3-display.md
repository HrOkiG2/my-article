---
title: "PHPでPDFをブラウザ表示する方法とS3保管PDFの扱い"
emoji: "📄"
type: "tech"
topics: ["php", "pdf", "aws", "s3"]
published: false
published_at: "2023-02-17 11:00"
---

> 初出: 2023年2月17日(旧ブログの記事を移行・加筆)

PHPでPDFをブラウザ表示・ダウンロードさせるときの書き方と注意点をまとめました。
サーバー上のPDFやS3に保管したPDFを、PHPから出力したい方を対象にしています。

## 結論

PDFの出力は、`header()` でContent-Typeなどを指定し、`readfile()` で中身を出力します。
ブラウザ表示からのダウンロードにはキャッシュ制御が、S3のPDFにはAWS SDK for PHPが必要です。

## PDFを表示する基本構文

```php
$output_filename = "表示させる時のファイル名称";
$output_file = "表示させたいファイル";

header('Content-Type: application/pdf');
header('Content-Disposition: inline; filename="' . $output_filename . '"');
header('Content-Length: ' . filesize($output_file));
readfile($output_file);
```

### 各ヘッダーの意味

* `Content-Type`: 扱うファイルの種類を指定します。拡張子ごとに指定名が決まっています。
  * 参考: [Content-Typeの一覧](https://qiita.com/AkihiroTakamura/items/b93fbe511465f52bffaa)
* `Content-Disposition`: ブラウザで表示するなら `inline`、ダウンロードさせるなら `attachment` を指定します。

### 注意点

**ファイルは相対パスで指定します**

URLを含む形で読み込むとエラーになります。

* OK: `./../xxx/yyy/zzz.pdf`
* NG: `https://xxx/yyy/zzz.pdf`

**header() の前に文字を出力しません**

`header()` より前に `echo` や `var_dump()` で出力すると、ファイルに余計な文字が混ざったり、読み込みに失敗したりします。

## ブラウザ表示からダウンロードできるようにする

`inline` でブラウザ表示した際に、ブラウザからダウンロードできない場合があります。
その場合は、ヘッダーでキャッシュを制御します。

```php
header('Cache-Control: private, max-age=90, must-revalidate'); // 追記
header('Content-Type: application/pdf');
header('Content-Disposition: inline; filename="' . $output_filename . '"');
header('Content-Length: ' . filesize($output_file));
readfile($output_file);
```

| 指定 | 内容 |
| --- | --- |
| `private` | キャッシュをプライベート状態で保持します。 |
| `max-age` | キャッシュを保持する時間です。短いほうが良いと考え、90秒にしています。 |
| `must-revalidate` | キャッシュが新鮮かどうかを検証します。 |

参考: https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Cache-Control

## S3から取得したPDFをブラウザ表示する

S3のPDFを表示するには、AWS SDK for PHPが必要です。

### 前提

* AWS SDK for PHPをインストール済みであること
* S3へのアクセス権があること(EC2やLambdaを使う場合は、そのサーバーからのアクセスのみ許可します)

### やりたいこと

* S3バケットからPDFのメタデータを取得します。
* `header` をPDFに指定して、取得した内容を出力します。
* ファイルが存在しない場合を考慮して、`try-catch` を使います。

### コード

```php
use Aws\S3\S3Client;
use Aws\S3\Exception\S3Exception;

try {
    $s3Client = S3Client::factory([
        'region' => '', // リージョン名
        'version' => 'latest',
    ]);

    // S3からオブジェクトを取得
    $file_obj = $s3Client->getObject([
        'Bucket' => '', // バケット名
        'Key' => '',    // ファイルパス(可変にする場合は変数を設定)
    ]);

    $read_file_name = [
        'contentType' => $file_obj['@metadata']['headers']['content-type'],
        'body' => $file_obj['Body']->getContents(),
    ];

    header('Cache-Control: private,max-age=180');
    header('Content-Type: application/pdf');
    header('Content-Disposition: inline; filename="' . $download_file_name . '"');

    while (ob_get_level()) {
        ob_end_clean();
    }

    echo $read_file_name['body'];
} catch (S3Exception $e) {
    echo $e->getMessage();
    echo '<br>';
    echo '<p class="text-center font-weight-bold">不正なデータです。</p>';
    die();
}
```

## まとめ

* PDFの出力は `header()` と `readfile()` で行います。
* `header()` の前に文字を出力しないでください。
* ブラウザ表示からダウンロードさせるには、`Cache-Control` の指定が必要です。
* S3のPDFは、AWS SDK for PHPで取得して出力します。
