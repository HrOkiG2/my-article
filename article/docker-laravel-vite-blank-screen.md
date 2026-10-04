---
title: "Docker+Laravel+React+Viteで npm run dev すると画面が真っ白になる問題の解決"
emoji: "⚪"
type: "tech"
topics: ["docker", "laravel", "react", "vite", "php"]
published: false
published_at: "2023-07-22 14:19"
---

> 初出: 2023年7月22日(旧ブログの記事を移行・加筆)

Docker上にLaravel + React + Viteの環境を作り、`npm run dev` を実行したところ、画面が真っ白になってしまいました。
同じ構成でハマっている方に向けて、原因と解決方法をまとめます。

## 結論

次の2点を直すと解決しました。

* `docker-compose.yml` で、Vite用のポート `5173` を公開する
* `package.json` の `dev` スクリプトに `--host` を付ける

## 環境

| 項目 | バージョン |
| --- | --- |
| Docker | 20.10.6 |
| Laravel | 10.15.0 |
| React | 18.2.0 |

主なパッケージは次のとおりです。

```
@headlessui/react@1.7.15
@inertiajs/react@1.0.9
@tailwindcss/forms@0.5.4
@vitejs/plugin-react@3.1.0
autoprefixer@10.4.14
axios@1.4.0
laravel-vite-plugin@0.7.8
postcss@8.4.27
react-dom@18.2.0
react@18.2.0
```

## 症状

Docker+Laravel+React+Viteで環境を構築して `npm run dev` を実行すると、画面が真っ白になりました。

ブラウザの開発者ツール(Console)には、次のエラーが出ていました。

```
GET http://127.0.0.1:5173/@vite/client net::ERR_CONNECTION_REFUSED
GET http://127.0.0.1:5173/resources/js/app.jsx net::ERR_CONNECTION_REFUSED
GET http://127.0.0.1:5173/resources/js/Pages/Welcome.jsx net::ERR_CONNECTION_REFUSED
GET http://127.0.0.1:5173/@react-refresh net::ERR_CONNECTION_REFUSED
```

## 原因: ポートが公開されていない

ConsoleのリクエストURLを見て、ポートがおかしいことに気づきました。

Dockerのnginxには `8080:80` を指定していました。
それなのに、Viteのリクエスト先は `http://127.0.0.1:5173` になっています。

つまり、**ブラウザからコンテナ内のVite開発サーバー(5173)に接続できない**状態でした。
Laravel・Reactを入れているアプリケーションコンテナの設定が漏れていたわけです。

## 解決方法

### 1. docker-compose.yml にポートを追記する

アプリケーションコンテナに、Vite用のポートを開放します。

```yaml
app:
  build: ./docker/php
  volumes:
    - ./src:/data
  ports:
    - 5173:5173 # Vite用のポートを開放
```

### 2. package.json に --host を追記する

Viteがコンテナの外からアクセスできるように、`--host` を付けます。

```json
"scripts": {
    "dev": "vite --host",
    "build": "vite build"
}
```

`--host` がないと、Viteは `localhost` でしか待ち受けないため、Dockerの外から接続できません。

これで、画面が表示されるようになりました。

## まとめ

* Docker上でViteを使うときは、Vite用のポート(5173)の公開が必要です。
* あわせて、`vite --host` でコンテナ外からの接続を許可します。
* Consoleの `ERR_CONNECTION_REFUSED` とリクエスト先のポートは、原因を探す手がかりになります。
