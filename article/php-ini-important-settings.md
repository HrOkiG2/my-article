---
title: "意外と見落としがちなphp.iniの重要な設定ポイント"
emoji: "⚙️"
type: "tech"
topics: ["php", "phpini", "security", "performance"]
published: false
published_at: "2024-05-12 15:13"
---

> 初出: 2024年5月12日(旧ブログの記事を移行・加筆)

php.iniの設定で、意外と見落とされがちな重要ポイントをまとめました。
PHPの動作をカスタマイズして、パフォーマンスとセキュリティを高めたい方に向けた内容です。

## 結論

次の5つは、必ず見直しておきましょう。

* `memory_limit`
* `max_execution_time`
* `error_reporting`
* `file_uploads`
* セッション関連のセキュリティ設定

## メモリ制限(memory_limit)

PHPスクリプトが消費できる、最大のメモリ量を指定します。
デフォルトは `128M` のことが多いですが、大規模なアプリケーションでは増やす必要があります。
足りないと、スクリプトが突然停止することがあるので、適切な量を設定しましょう。

```ini
memory_limit = 256M
```

## 実行時間の制限(max_execution_time)

スクリプトが実行を終えるまでの最大時間を、秒で設定します。
デフォルトは `30` 秒です。時間のかかる処理を扱う場合は、調整が重要です。

```ini
max_execution_time = 60
```

## エラーレポーティング(error_reporting)

開発中は、すべてのエラーを表示することが重要です。
一方で、本番環境では、エラーの表示を抑えるべきです。
この設定で、どのレベルのエラーを報告するかを制御できます。

```ini
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
```

本番環境では、あわせて画面への表示をオフにして、ログに出力するようにしましょう。

```ini
display_errors = Off
log_errors = On
```

## ファイルアップロード(file_uploads)

ファイルのアップロードを許可するかどうかを制御します。
セキュリティ上、使わない場合は無効にすることをおすすめします。

```ini
file_uploads = On
```

使わないなら `Off` にしてください。

## セッションのセキュリティ設定

セッションIDの扱いには、特に注意が必要です。
セッションハイジャックを防ぐために、次の設定をしておきましょう。

```ini
session.use_strict_mode = 1
session.cookie_secure = 1
session.cookie_httponly = 1
```

* `session.use_strict_mode`: サーバーが発行していない、未初期化のセッションIDを受け付けません。
* `session.cookie_secure`: CookieをHTTPS通信でのみ送ります。
* `session.cookie_httponly`: JavaScriptからCookieを読めないようにします。

ログイン後に `session_regenerate_id()` でセッションIDを再生成することも、あわせて行いましょう。

## まとめ

* `memory_limit` と `max_execution_time` は、アプリケーションの規模に合わせて調整しましょう。
* `error_reporting` は、開発と本番で使い分けます。本番は画面表示を `Off` にします。
* 使わない `file_uploads` は、無効にしておきます。
* セッションまわりは、`use_strict_mode`、`cookie_secure`、`cookie_httponly` を設定しましょう。
