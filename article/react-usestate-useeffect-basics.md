---
title: "ReactのuseStateとuseEffectの基本: jQueryから乗り換えて分かったこと"
emoji: "⚛️"
type: "tech"
topics: ["react", "javascript", "hooks", "usestate", "useeffect"]
published: false
published_at: "2023-08-18 01:17"
---

> 初出: 2023年8月18日(旧ブログの記事を移行・加筆)

ReactのuseStateとuseEffectの基本を、jQueryから乗り換えた視点でまとめました。
jQueryでDOM操作をしてきて、Reactを学び始めた方に向けた内容です。

## 結論

* `useState` は、コンポーネント内の**状態を管理**します。
* `useEffect` は、レンダリング後に実行する**副作用**を管理します。
* `useEffect` の第2引数を指定し忘れると、無限ループになるので注意が必要です。

## Reactの考え方

Reactでは、DOM要素を直接操作する代わりに、**状態管理とレンダリングの仕組み**を使って操作することが推奨されています。

## useState

`useState` は、状態(state)を、Reactコンポーネント内で管理するためのフックです。
状態は、コンポーネント内のデータの変化を追跡して、自動的に再レンダリングするために使います。

差分を取って、差分の分だけレンダリングする仕組みなので、楽で速いです。
Reactの基本理念である、状態管理による差分の検出が、`useState` の肝だと感じています。

## useEffect

コンポーネントのレンダリング後に実行する処理(データの取得など)を管理するためのフックです。

* 第1引数に、副作用の関数を渡します。
* 第2引数に、依存する値の配列を渡します。この配列の値が変更されると、副作用の関数が再実行されます。
* **空の配列**を渡すと、**初回のレンダリング時にだけ**、副作用の関数が実行されます。

```jsx
const [csrfToken, setCsrfToken] = useState(null);

useEffect(() => {
  // metaタグからトークンを取得
  const metaTag = document.querySelector('meta[name="csrf-token"]');
  if (metaTag) {
    setCsrfToken(metaTag.content);
  }
}, []);
```

なんとなく、`onload` のような感覚です。

### 注意点

**第2引数を何も指定しない**と、レンダリングのたびに副作用が実行されます。
副作用の中で、状態を更新していると、再レンダリングが繰り返されて、**無限ループ**になります。気をつけましょう。

## まとめ

* `useState` は状態を持ち、状態が変わると、自動で再レンダリングされます。
* `useEffect` は、レンダリング後の処理を担当します。
* 第2引数の依存配列を、忘れずに指定しましょう。
