---
title: "Laravel10でBreezeをインストールすると互換性エラーになる問題の解消"
emoji: "🌬️"
type: "tech"
topics: ["laravel", "php", "composer", "breeze"]
published: false
published_at: "2024-03-16 10:24"
---

> 初出: 2024年3月16日(旧ブログの記事を移行・加筆)

Laravel 10で Laravel Breeze をインストールしようとして、互換性エラーが出たときの解消方法をまとめました。
公式の手順どおりに進めたのにエラーになってしまった方に向けた備忘録です。

## 結論

Laravel 10では、Breezeの **1.x系** を、バージョンを指定してインストールすれば動きます。

```bash
composer require laravel/breeze:^1.29.1 --dev
```

## 環境

* PHP 8.2
* Laravel Framework 10.48.3
* MySQL 8.0
* nginx
* Docker

## エラー

公式の手順どおりにインストールしたところ、次のエラーになりました。

```
composer require laravel/breeze --dev
Using version ^2.0 for laravel/breeze
./composer.json has been updated
Running composer update laravel/breeze
Loading composer repositories with package information
Updating dependencies
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Root composer.json requires laravel/breeze ^2.0 -> satisfiable by laravel/breeze[v2.0.0].
    - laravel/breeze v2.0.0 requires illuminate/console ^11.0 -> found illuminate/console[v11.0.0, ..., v11.0.7] but these were not loaded, likely because it conflicts with another require.

Installation failed, reverting ./composer.json and ./composer.lock to their original content.
```

## 原因

バージョンを指定しないと、Composerは最新の `laravel/breeze` v2.0.0 を選びます。
ところが、v2.0.0 は Laravel 11 向け(`illuminate/console ^11.0` が必要)なので、Laravel 10のプロジェクトとは合いません。

## 解消手順

### 1. インストール可能なバージョンを確認する

```bash
composer show laravel/breeze --all | less
```

結果の `versions` に、インストールできるバージョンが一覧で表示されます。

```
versions : dev-master, 2.x-dev, v2.0.0, 1.x-dev, * v1.29.1, v1.29.0, v1.28.3, ...
```

### 2. 1.x系の最新を選んでインストールする

一覧から、1.x系の最新(`v1.29.1`)を選び、バージョンを指定してインストールします。

```bash
composer require laravel/breeze:^1.29.1 --dev
```

これで、無事にインストールできました。

## まとめ

* バージョンを指定しないと、最新の v2.x(Laravel 11向け)が選ばれます。
* Laravel 10では、Breezeの1.x系をバージョン指定でインストールしましょう。
* 使えるバージョンは、`composer show laravel/breeze --all` で確認できます。
