---
title: "AmazonLinux2023でImagickのインストールに失敗した原因はPHP 8.3との互換性"
emoji: "🖼️"
type: "tech"
topics: ["php", "imagick", "amazonlinux", "aws", "pecl"]
published: false
published_at: "2024-08-25 22:30"
---

> 初出: 2024年8月25日(旧ブログの記事を移行・加筆)

AmazonLinux 2023で、PHPのImagick拡張が入らなかった問題の、原因と解消方法をまとめました。
AmazonLinux 2023にImagickを入れようとして、`make` で失敗した方に向けた備忘録です。

## 結論

PHPを **8.3から8.2にダウングレード**してからインストールすると、正常に終了しました。
原因は、PHPのバージョンと、Imagickの互換性だったようです。

## 症状

Imagickを使いたい、ただそれだけでした。
次のエラーで、インストールに失敗しました。

```
make: *** [Makefile:196: /var/tmp/imagick/Imagick_arginfo.h] Error 1
ERROR: `make INSTALL_ROOT="/var/tmp/pear-build-rootEHkAY7/install-imagick-3.7.0" install' failed
```

Imagickのインストールに必要な、次のパッケージがインストールされていることは確認しました。

```
php-devel php-pear gcc make
ImageMagick ImageMagick-devel
```

`make` できていないことが問題だと考えて、ソースからのビルドも試しましたが、うまくいきませんでした。
何より悔しかったのは、**何が原因で落ちているのか分からない**ことでした。

## 解決(追記: 2024-08-26)

PHPのバージョンを **8.3から8.2にダウングレード**したあとで、Imagickをインストールしたところ、正常に終了しました。
バージョンの互換性に問題があったようです。
インストール時に使われているパーサーなどにも変化があったので、おそらく、互換性の問題だと考えます。
AmazonLinux 2023は、何も悪くありませんでした。

## まとめ

* Imagickの `make` が失敗するときは、PHPのバージョンとの互換性を疑いましょう。
* 私の環境では、PHP 8.2にダウングレードすると、インストールできました。
* 原因が分からないときは、バージョンを変えて試すのも1つの手です。
