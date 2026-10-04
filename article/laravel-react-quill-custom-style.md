---
title: "Laravel + React-Quillに独自のスタイルを適用する方法"
emoji: "✍️"
type: "tech"
topics: ["laravel", "react", "quill", "css", "inertia"]
published: false
published_at: "2024-01-05 20:53"
---

> 初出: 2024年1月5日(旧ブログの記事を移行・加筆)

React-Quillのスタイルを、少しだけ変更したいときのTipsをまとめました。
Laravel + ReactでリッチテキストエディタのUIを整えたい方に向けた内容です。
スタイルを当てることに焦点を置いているので、UIのデザイン自体は気にしないでください。

## 結論

Quill用のCSSを用意して、Quillを使うコンポーネントで、**`quill.snow.css` のあとに**インポートします。
デフォルトのスタイルを上書きできます。

## なぜEditorに独自スタイルを当てるのか

ITリテラシーが低いユーザーに使ってもらうのが、一番の理由です。
殺風景なUIだと、その時点でユーザーが離れてしまうことに、最近気づきました。
プログラマーやエンジニアには理解しにくいかもしれませんが、リテラシーが低い人は、とことん低いです。

* 画面が味気ない
* よく分からないボタンがある
* 英語で書かれている

少し考えれば分かるだろう、と言いたくなりますが、**使ってもらえなければ、ただの使われないものを作ってしまうだけ**なので、工夫が必要です。

## CSSの定義

LaravelのCSS用ディレクトリに、Quill用のCSSを用意します。

* `resources/css/Quill/style.css`
* Quill用にディレクトリを分けておくのが、おすすめです。

```css
.quill {
    .ql-toolbar {
        background-color: rgb(148 163 184); /* bg-slate-400 */
    }

    .ql-editor {
        h1, h2, h3 {
            padding-left: 0.5rem;
            border-radius: 0.5rem; /* rounded-lg */
            background-color: rgb(56 189 248); /* bg-sky-400 */
            color: white;
            margin-bottom: 1.2rem;
        }

        p {
            font-size: 1rem;
            margin-bottom: 1.2rem;
        }
    }
}
```

## Quillのコンポーネントで読み込む

定義したCSSを、Quillを使っているコンポーネントでインポートします。
Quillのデフォルトスタイルを上書きするために、`react-quill/dist/quill.snow.css` の**後**にインポートします。

```jsx
import React from 'react';
import ReactQuill from 'react-quill';
import 'react-quill/dist/quill.snow.css';
import '@/../css/Quill/style.css';

export default function QuillEditor({ handleValueChange, value }) {
    const customToolbarOptions = [
        ['bold', 'italic', 'underline', 'strike'],
        ['link', 'image'],
        ['blockquote', 'code-block'],
        [{ 'header': 1 }, { 'header': 2 }],
        [{ 'list': 'ordered' }, { 'list': 'bullet' }],
        [{ 'script': 'sub' }, { 'script': 'super' }],
        [{ 'indent': '-1' }, { 'indent': '+1' }],
        [{ 'direction': 'rtl' }],
        [{ 'size': ['small', false, 'large', 'huge'] }],
        [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
        [{ 'color': [] }, { 'background': [] }],
        [{ 'font': [] }],
        [{ 'align': [] }],
        ['clean'],
    ];

    // 親コンポーネントから受け取った関数で、入力値を渡す
    const handleContentValueChange = (content) => {
        handleValueChange(content);
    };

    return (
        <ReactQuill
            theme="snow"
            value={value} // 親コンポーネントから受け取った値を設定
            onChange={handleContentValueChange}
            modules={{
                toolbar: customToolbarOptions,
            }}
        />
    );
}
```

## まとめ

* Quill用のCSSは、専用のディレクトリにまとめます。
* CSSのインポート順は、`quill.snow.css` のあとにします。
* ツールバーやエディタ内の見出しなどに、独自のスタイルを当てられます。
* ReactQuillを、メンテナンスしてくれている方々に感謝です。
