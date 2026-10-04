---
title: "LaravelからMinIOにputできない原因はエンドポイントのサービス名指定"
emoji: "🪣"
type: "tech"
topics: ["laravel", "minio", "docker", "s3", "php"]
published: false
published_at: "2024-08-17 17:06"
---

> 初出: 2024年8月17日(旧ブログの記事を移行・加筆)

Docker上のLaravelから、S3の代わりのMinIOに画像を保存できなかった問題の、原因と解消方法をまとめました。
ローカルでDockerを使って、MinIOを開発用のS3として使っている方に向けた備忘録です。

## 結論

`.env` の `AWS_ENDPOINT` には、`localhost` ではなく、**`docker-compose.yml` のサービス名**を指定する必要があります。

## 概要

個人開発で使っている次のスタックで、画像を保存できない問題が起きました。

* Laravel 10
* MinIO(S3の代わり)
* Intervention Image 3

## 調査

### Intervention Image

Intervention ImageがVersion 2から3で大きく変わったので、使い方を疑いました。
ですが、問題はありませんでした。デバッグで値を見ても、問題なしです。

```php
/**
 * 画像をリサイズする。
 */
public function resize(UploadedFile $file): ImageInterface
{
    return $this->ImageManager
        ->read($file)
        ->resize(Media::BLOG_IMAGE_WIDTH, Media::BLOG_IMAGE_HEIGHT);
}

/**
 * 画像をS3にPutする。
 */
public function putForS3(string $path, UploadedFile $file): bool
{
    $convertFile = $this->ImageManager->resize($file)->toPng();
    return $this->diskS3->put($path, $convertFile);
}
```

### MinIOへの通信

ローカル環境をDockerで開発していたので、ローカルネットワークが怪しいと思いました。
次のコマンドでネットワークを確認すると、同じネットワーク内にいたので、コンテナ間の通信はできているはずでした。

```bash
docker network ls
docker network inspect <ネットワーク名>
```

ですが、MinIOに向けた `curl` が返ってこないことが分かりました。

## 原因

調べていくと、**サービス名に対して、エンドポイントを指定しないといけない**ことが分かりました。

```yaml
minio:
  image: quay.io/minio/minio:RELEASE.2023-01-18T04-36-38Z
  container_name: 'discovery-gem_minio'
```

今回は `minio` というサービス名で登録していました。
そのため、`.env` のエンドポイントを、次のように設定する必要がありました。

```
AWS_ENDPOINT=http://minio:<docker-compose.ymlで指定したポート>
AWS_USE_PATH_STYLE_ENDPOINT=true
```

MinIOでは、`AWS_USE_PATH_STYLE_ENDPOINT=true` も合わせて設定するのが一般的です。

たったこれだけのことで悩んだのは、悔しいです。

## まとめ

* Dockerのコンテナ間では、`localhost` ではなく、サービス名で通信します。
* `AWS_ENDPOINT` には、サービス名とポートを指定します。
* MinIOでは、パススタイルのエンドポイントも有効にしましょう。
