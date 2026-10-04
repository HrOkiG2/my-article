---
title: "ファイルのinputにはaccept属性を指定しよう(HTML Tips)"
emoji: "📎"
type: "tech"
topics: ["html", "frontend", "ux", "tips"]
published: false
published_at: "2024-01-21 16:22"
---

> 初出: 2024年1月21日(旧ブログの記事を移行・加筆)

HTMLの `input type="file"` に指定できる、`accept` 属性の使い方をまとめました。
ファイルアップロードのUXを、手軽に改善したい方に向けたTipsです。

## 結論

`accept` 属性を指定すると、ファイル選択ダイアログで、**選べるファイルの種類を絞れます**。
ただし、フロントの制限は回避できるので、**バックエンドのバリデーションは必須**です。

## accept属性とは

`accept` 属性は、フォームの `<input>` タグで使います。
ユーザーがファイルを選ぶときに、どの種類のファイルを選べるかを、ブラウザに指示します。

指定できる値は、主に次のとおりです。

* ファイルタイプ指定(`image/*` など)
* MIMEタイプ(`image/jpeg` など)

「CSVだけ受け付けたい」「画像は、JPEGとPNGだけ受け付けたい」という要件が出たときに、フロントでできる施策です。

## 使用例

### ファイルタイプの指定

```html
<!-- すべての画像ファイル形式 -->
<input type="file" accept="image/*">

<!-- すべての音声ファイル形式 -->
<input type="file" accept="audio/*">

<!-- すべてのビデオファイル形式 -->
<input type="file" accept="video/*">
```

### MIMEタイプの指定

```html
<!-- JPEG形式の画像ファイルのみを受け入れる -->
<input type="file" accept="image/jpeg">

<!-- application/pdf は、PDFファイル -->
<input type="file" accept="application/pdf">
```

### 複数の指定

カンマ区切りで、複数指定できます。

```html
<input type="file" accept="image/png, image/jpeg">
```

## acceptを指定した場合の挙動

ローカルのファイル選択ダイアログで、指定した種類のファイルだけが選べるようになります。
たとえば、次のように指定すると、CSVファイルだけが選べます。

```html
<input type="file" accept="text/csv">
```

## 目的

セキュリティへの効果はありませんが、UXの向上が狙えます。
ただし、フロントで制限しても、サーバーにアップロードする方法は、いくらでもあります。
**バックエンドでの、バリデーションは必ず行ってください。**

## 参考

* [accept - HTML: ハイパーテキストマークアップ言語(MDN)](https://developer.mozilla.org/ja/docs/Web/HTML/Attributes/accept)

## まとめ

* `accept` で、ファイル選択ダイアログの選択肢を絞れます。
* ファイルタイプ(`image/*`)か、MIMEタイプ(`image/jpeg`)で指定します。
* UX向上が目的で、セキュリティの代わりにはなりません。バックエンドの検証も必須です。
