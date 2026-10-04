---
title: "BladeからVue3コンポーネントに配列を渡す方法(Laravel)"
emoji: "🧩"
type: "tech"
topics: ["laravel", "php", "vue", "vuejs", "blade"]
published: false
published_at: "2023-03-12 22:36"
---

> 初出: 2023年3月12日(旧ブログの記事を移行・加筆)

LaravelのBladeファイルから、Vue3のコンポーネントに配列を渡す方法をまとめました。
BladeとVue3を併用していて、データの受け渡しで迷っている方に向けた備忘録です。

## 結論

次の3ファイルを設定すれば、BladeからVue3に配列を渡せます。

1. `test.blade.php` で配列をJSONに変換して、コンポーネントに渡します。
2. `resources/js/app.js` で、`createApp` を使ってコンポーネントを登録します。
3. `Test.vue` で、`props` を定義して受け取ります。

## test.blade.php

配列はControllerから渡してもいいのですが、ここではBladeファイル内で作成しています。
配列をJSONに変換して、コンポーネントに渡します。

```php
{{-- Controllerから渡しても良い --}}
{{-- ここではBladeファイル内で配列を作成 --}}
@php
    $photes_data = array(
        array(
            'img_no' => '0',
            'path' => 'laundry.jpg',
            'alt' => 'サーフボード'
        ),
        array(
            'img_no' => '1',
            'path' => 'harusa.jpg',
            'alt' => '農業'
        ),
        array(
            'img_no' => '2',
            'path' => 'ht-beach.jpeg',
            'alt' => 'ht_1'
        )
    );
@endphp

{{-- コンポーネント作成 --}}
<div id="phote_gallery">
    {{-- 配列をJSONに変換して渡す --}}
    <template-photegallery v-bind:photes_data="'{{ json_encode($photes_data) }}'"></template-photegallery>
</div>
```

## resources/js/app.js

```js
import { createApp } from 'vue';
import Test from './components/Test.vue';

if (document.getElementById('phote_gallery')) {
    console.log("test.blade.php")
    createApp({})
        .component("template-photegallery", Test)
        .mount('#phote_gallery');
}
```

Laravelでは、`resources/js/app.js` がVue.jsのエントリーポイントになります。
このファイルでVue.jsのコンポーネントをインスタンス化しています。

ただし、このファイルはすべてのBladeファイルで読み込まれます。
そのままだと、コンポーネントを使わないページでもインスタンス化されてしまいます。

そこで、`id="phote_gallery"` があるときだけインスタンス化するように、JSでHTML内のID属性を取得して判定しています。

また、Vue3では、Vueアプリを作成するときに `createApp({})` を使います。
**Vue2の書き方では動作しませんでした。**

Vue2とVue3の違いが分からない方は、次の記事がとても参考になるので、ぜひ読んでみてください。

* [Vue 3の新しい機能と変更点・全11件](https://blog.capilano-fw.com/?p=6393)

## Test.vue

```vue
<script setup>
import { defineProps } from "vue";
const props = defineProps({
    photes_data: {
        type: String,
        required: true,
    },
});
console.log(props.photes_data);
</script>
```

`props` を定義して、`test.blade.php` で作った `<template-photegallery>` の `template-photegallery` を呼び出します。
これで、BladeからVue3へのデータ連携は完了です。

:::message
`props` は **JSON文字列** として届くので、配列として使うときは `JSON.parse(props.photes_data)` で変換してください。
また、`<script setup>` では `defineProps` はimport不要です(コンパイラマクロなので、importしても問題はありません)。
:::

## まとめ

* Bladeで配列を `json_encode` して、Vueコンポーネントの `props` に渡します。
* `app.js` では、Vue3の `createApp` でコンポーネントを登録します。
* 特定のIDがあるページだけインスタンス化すると、不要なページに影響しません。
* Vue側の `props` は文字列で届くため、`JSON.parse` で配列に戻して使いましょう。
