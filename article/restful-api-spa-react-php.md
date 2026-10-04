---
title: "RESTful APIとSPAの関係をReactとPHPで解説"
emoji: "🔗"
type: "tech"
topics: ["php", "react", "restapi", "spa", "javascript"]
published: false
published_at: "2024-02-25 21:40"
---

> 初出: 2024年2月25日(旧ブログの記事を移行・加筆)

RESTful APIとSPA(シングルページアプリケーション)の関係を、PHPとReactの簡単な例で解説します。
SPAを使い始めたい方や、APIとフロントエンドの連携を整理したい方に向けた内容です。

## 結論

SPAは、画面の更新に必要なデータを、RESTful APIから非同期に取得します。
つまり、**RESTful APIは、SPAのデータ通信の土台**です。

## RESTful APIとは？

RESTful APIは、WebアプリケーションやWebサービスで使われる、プログラミングインターフェースです。
REST(Representational State Transfer)の原則に基づいて設計され、Web上のリソースへのアクセスや操作を、統一的な方法で提供します。

* リソースは、URLで識別します。
* HTTPメソッド(GET、POST、PUT、DELETEなど)を使って、アクセスします。

## SPA(シングルページアプリケーション)とは？

SPAは、ユーザーが操作するときに、ページを再読み込みせず、必要なデータだけをサーバーから非同期に取得して、動的にページを更新するWebアプリケーションの形式です。
従来のマルチページアプリケーションに比べて、**ユーザー体験が向上します。**

私のチームでも、ページのリロードが嫌だというユーザーの声から、SPAへの移行が少しずつ始まっています。
世間でもSPAは流行っています。(流行っているから偉い、というわけではありません。知っておくことが大事です。)

## RESTful APIとSPAの関係

SPAの流れは次のとおりです。

1. ページの初回読み込み時に、アプリケーションのコード(HTML、CSS、JavaScript)をすべてロードします。
2. ユーザーの操作でデータが必要になると、RESTful API経由で、サーバーからデータを非同期に取得します。
3. データはJSONやXML形式で返ってきます。
4. JavaScriptでクライアント側で処理して、ページの一部だけを動的に更新します。

この仕組みで、SPAはスムーズで反応の良い画面を提供できます。
**RESTful APIは、その裏側にあるデータ通信の基盤**になっています。

## PHPとReactを使った例

PHPで書いた簡単なRESTful APIと、Reactで作ったSPAの例です。

### PHPによるRESTful API

GETリクエストを受け取って、簡単なJSONデータを返します。

```php
<?php
header('Content-Type: application/json');
$response = ['message' => 'Hello, world!'];
echo json_encode($response);

// Response
// {"message": "Hello, world!"}
```

このスクリプトをサーバーに置いてアクセスすると、JSONが返ります。

### ReactによるSPA

上のAPIからデータを取得して、表示します。

```js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    axios.get('http://yourserver.com/index.php')
      .then(response => {
        setMessage(response.data.message);
      })
      .catch(error => console.error('There was an error!', error));
  }, []);

  return (
    <div>
      <button>{message}</button>
    </div>
  );
}

export default App;
```

このコードは、コンポーネントがマウントされたあとに、APIからデータを取得して、メッセージを表示します。
React(クライアント側)とPHP(サーバー側)による、RESTful APIとSPAの基本的な連携です。

今回は、データを取得して表示するだけですが、イベント処理を加えれば、リロードなしで画面を動的に変更できます。

:::message
フロントとAPIのオリジンが違う場合は、CORSの設定が必要です。
:::

## セキュリティとパフォーマンスの考慮

SPAとRESTful APIを使うときは、セキュリティとパフォーマンスの両方に配慮が必要です。

### セキュリティ

1. **HTTPSを使う**: APIとの通信は、常にHTTPSにします。中間者攻撃(MITM)のリスクを減らせます。
2. **CORSポリシーを適切に設定する**: 信頼できるオリジンからのリクエストだけを許可します。設定が不適切だと、セキュリティリスクになります。
3. **認証と認可**: JWT(JSON Web Tokens)やOAuthなどの堅牢な認証の仕組みでAPIへのアクセスを管理します。ユーザーの権限に基づく認可も重要です。
4. **入力の検証とサニタイズ**: SQLインジェクションやXSSを防ぐため、サーバー側とクライアント側の両方で、入力値を検証・サニタイズします。
5. **依存関係のセキュリティ**: ライブラリやフレームワークの脆弱性に注意して、定期的に更新します。

### パフォーマンス

1. **コード分割**: ユーザーが必要なコードだけをダウンロードするようにして、初期ロード時間を短縮します。
2. **キャッシュ戦略**: 静的リソースやAPIレスポンスのキャッシュを活用します。ブラウザのキャッシュやCDNが有効です。
3. **遅延ローディング(Lazy Loading)**: 画像やコンポーネントは、必要になるまでロードしません。
4. **APIの最適化**: レスポンスをできるだけ小さくします。フィルタリングやページネーションもAPI側でサポートします。
5. **フロントエンドの最適化**: 不要な再レンダリングを避けて、メモ化(memoization)や仮想DOMを効率的に使います。

セキュリティとパフォーマンスは、設計や開発の初期段階から考慮しておきましょう。

## まとめ

* RESTful APIは、URLとHTTPメソッドでリソースを操作する設計です。
* SPAは、初回に画面のコードを読み込み、以降はAPIからデータだけを取得して画面を更新します。
* PHPでJSONを返すAPIを作り、Reactから `axios` で取得する構成が基本です。
* HTTPS、CORS、認証・認可などのセキュリティと、パフォーマンスの考慮も忘れないようにしましょう。
