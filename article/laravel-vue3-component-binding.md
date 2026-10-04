---
title: "Laravelの一部にVue3コンポーネントを組み込む方法"
emoji: "🧱"
type: "tech"
topics: ["laravel", "vue", "vite", "php", "javascript"]
published: false
published_at: "2024-04-15 23:06"
---

> 初出: 2024年4月15日(旧ブログの記事を移行・加筆)

既存のLaravelプロジェクトの一部に、Vue3のコンポーネントを組み込む方法をまとめました。
jQueryを卒業して、VueやReactを導入したいLaravel開発者の方に向けた内容です。

## 結論

SPA化はコストが高いので、Laravelの一部だけにVueを導入しましょう。
手順は次のとおりです。

1. ViteのプラグインとしてVueをインストールします。
2. `vite.config.js` と `app.js` を設定します。
3. BladeにVueのマウント先を用意します。
4. Vueコンポーネントを作成して、Bladeから呼び出します。

## 環境

* PHP 8.2
* Laravel 10
* Vue 3
  * node v18.19.1
  * npm 10.5.0

## やりたいこと

Laravel 10の一部に、Vue.jsを適用します。

* Vueコンポーネントを定義する
* Vueコンポーネントをバインドする
* Vueコンポーネントに、バックエンドからデータを渡す

## 手順

### 1. Vueをインストールする

LaravelはビルドツールにViteを使っているので、ViteのプラグインとしてVueをインストールします。

```bash
npm install vue
npm install @vitejs/plugin-vue --save-dev
```

### 2. vite.config.js を設定する

インストールしたVueのプラグインを、設定ファイルに追記します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue'; // 追記

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/css/app.css', 'resources/js/app.js'],
      refresh: true,
    }),
    vue(), // 追記
  ],
});
```

### 3. app.js を編集する

Vueコンポーネントをグローバルコンポーネントとして登録するために、`app.js` を編集します。
やりたいことは、**コンポーネントの登録と、マウント**です。

```js
import './bootstrap';
import { createApp } from 'vue';
import Alpine from 'alpinejs';
import CsrfToken from './Components/Utility/CsrfToken.vue';
import VueDatePicker from './Components/Utility/DatePicker.vue';
import AddSalesListModal from './Components/AddSalesListModal.vue';

window.Alpine = Alpine;

Alpine.start();

const app = createApp({});

// グローバルコンポーネント
app.component('csrf-token', CsrfToken);
app.component('add-sales-list-modal', AddSalesListModal);
app.component('date-picker', VueDatePicker);

// #app にマウント
app.mount('#app');
```

### 4. マウント先を指定する

`app.js` で、マウント先を `id="app"` にしたので、Bladeファイルにもマウント先を書きます。

```html
<body id="app" class="font-sans antialiased flex flex-col min-h-screen">
  ...
</body>
```

### 5. コンポーネントを作成する

試しに、LaravelのCSRFトークンを扱うコンポーネントを作ってみます。
axiosで非同期通信をするときなどに重宝するので、作っておくと便利です。
(ここでは動作確認のために `p` タグに出力していますが、本来はトークンを画面に表示しません。)

```vue
<script setup>
import { ref, onMounted } from 'vue';

const csrfToken = ref('');

onMounted(() => {
    const token = document.querySelector('meta[name="csrf-token"]')?.getAttribute('content');
    if (token) {
        csrfToken.value = token;
    }
});
</script>

<template>
  <p>{{ csrfToken }}</p>
</template>
```

### 6. コンポーネントを呼び出す

Bladeに次のように書くと、Vueで取得したCSRFトークンが出力されます。

```html
<csrf-token></csrf-token>
```

### バックエンドのデータをVueに渡す

Vueにバックエンドのデータを渡したいときは、次のように書きます。

```php
{{-- Blade --}}
<csrf-token :test='@json($test)'></csrf-token>
```

```vue
<script setup>
import { defineProps } from "vue";

const props = defineProps({
    test: Object,
});
</script>
```

## まとめ

* Laravelの一部だけにVueを導入すれば、SPA化よりずっと低コストで済みます。
* `@vitejs/plugin-vue` を入れて、`vite.config.js` と `app.js` を設定します。
* `createApp` でコンポーネントを登録して、`id="app"` にマウントします。
* Bladeから `:prop='@json($data)'` で、Vueにデータを渡せます。
* 時間さえあれば、いつかSPA化したいですね。
