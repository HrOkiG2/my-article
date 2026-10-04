---
title: "Laravel(Inertia + React)にreact-image-galleryを導入する"
emoji: "🖼️"
type: "tech"
topics: ["laravel", "react", "inertia", "imagegallery", "tailwindcss"]
published: false
published_at: "2023-08-26 18:02"
---

> 初出: 2023年8月26日(旧ブログの記事を移行・加筆)

Laravel + Inertia + Reactのプロジェクトに、フォトギャラリー(スライダー)の `react-image-gallery` を導入した手順をまとめました。
HPにスライダーを入れたい、LaravelとReactの開発者の方に向けた内容です。

## 結論

`npm install react-image-gallery` で入れて、**CSSも必ずインポート**します。
Laravel側の画像データは、`items` として、Inertia経由でReactに渡します。

## 技術構成

| 項目 | バージョン |
| --- | --- |
| Laravel | Laravel Framework 10.15.0 |
| React | react-dom@18.2.0 |
| DaisyUI | daisyui@3.5.0 |
| Tailwind | @tailwindcss/forms@0.5.4 |

## いい感じのフォトギャラリー

HPを作っていると、スライダーの要望がよく出てきます。
以前、jQueryで生実装して、面倒な思いをした経験があったので、今回は生実装を避けたいと思い、Reactで使えるライブラリを探しました。
そこで出会ったのが、`react-image-gallery` です。

ほかの選択肢には、次のようなものがあります。

| ライブラリ | 概要 |
| --- | --- |
| [react-image-gallery](https://github.com/xiaolin/react-image-gallery) | 画像のスライドショーを作るための、高機能なライブラリです。 |
| [react-images](https://github.com/jossmac/react-images) | シンプルでカスタマイズできる、グリッドスタイルのギャラリーを作るライブラリです。 |
| [react-awesome-lightbox](https://github.com/photostory/react-awesome-lightbox) | モーダルウィンドウ内で、画像を表示するライブラリです。 |

## 導入手順

### インストール

```bash
npm install react-image-gallery
```

### インポート

```jsx
import ImageGallery from 'react-image-gallery';
import 'react-image-gallery/styles/css/image-gallery.css';
```

:::message
**2行目のCSSも、必ずインポートしてください。**
CSSをインポートすることで、ライブラリに含まれているスタイルが適用されます。
:::

これで、使う準備は完了です。あとは、自由に設定していくだけです。

### Laravel側(コントローラー)

今回は、Laravelからデータを渡すので、コントローラーから配列を渡します。(本当は、Modelでデータを定義して渡すべきですが。)

```php
$images = [
    [
        "original" => "/img/AsItFlows/asit_team.jpg",
        "thumbnail" => "/img/AsItFlows/asit_team.jpg",
        "originalAlt" => "チーム",
        "thumbnailAlt" => "チーム",
        "onErrorImageURL" => "/img/AsItFlows/logo.jpg",
    ],
    [
        "original" => "/img/AsItFlows/sunset.jpg",
        "thumbnail" => "/img/AsItFlows/sunset.jpg",
        "originalAlt" => "サンセット",
        "thumbnailAlt" => "サンセット",
        "onErrorImageURL" => "/img/AsItFlows/logo.jpg",
    ],
    // ...
];

shuffle($images);

return Inertia::render('Public/AsItFlows/Top', [
    'items' => $images,
]);
```

### React側(JSX)

最後に、JSX側で、`items` を渡して使います。

```jsx
<ImageGallery
  items={items}
  showNav={true}
  autoPlay={true}
  showFullscreenButton={false}
  useBrowserFullscreen={false}
  showPlayButton={false}
/>
```

素敵なギャラリーができました。

## まとめ

* スライダーを生実装せず、`react-image-gallery` を使うと楽です。
* CSSのインポートを忘れないようにしましょう。
* Laravelから、配列を `items` として渡して、`ImageGallery` に渡すだけで表示できます。
