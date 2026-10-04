---
title: "Laravel + axiosで非同期通信が404になる原因は/apiの接頭辞"
emoji: "🔍"
type: "tech"
topics: ["laravel", "php", "axios", "vue", "javascript"]
published: false
published_at: "2023-03-06 15:03"
---

> 初出: 2023年3月6日(旧ブログの記事を移行・加筆)

Laravel(Vue)から `axios` で非同期通信をしたときに、`404` になって困った場合の原因と対処法をまとめました。
`routes/api.php` にルートを書いたのに、なぜか届かない、という方に向けた内容です。

## 結論

`routes/api.php` に定義したルートには、**デフォルトで `/api` の接頭辞が付きます。**
そのため、axiosで呼ぶURLは `/api/test` のように、`/api` から書く必要があります。

## エラー内容

レスポンスは次のとおりです。

```js
AxiosError {message: 'Request failed with status code 404', name: 'AxiosError', code: 'ERR_BAD_REQUEST', ...
```

考えられる原因は、次の2つです。

* ルーティングを間違えている
* 通信先のURLを間違えている

## 404になる原因

ルートの設定をしているのに、404が返ってきてしまいました。
エラーの文言どおり、**呼び出しているURLのページが存在しない**ことが原因です。

### エラーになるコード

`routes/api.php`

```php
Route::get(
    '/test',
    [Test::class, 'test_function']
);
```

`resources/js/components/EnduserSearchstore.vue`

```js
import axios from 'axios';

export default {
    data() {
        return {
            users: {},
        }
    },
    methods: {
        getBigCategory() {
            axios.get('/api_search_storeCategory')
                .then((response) => {
                    // this.users = response.data.users
                    console.log(response)
                })
        }
    },
    created() {
        this.getBigCategory()
    }
}
```

`routes/api.php` で定義しているのは `/test` なのに、Vue側では存在しない `/api_search_storeCategory` を呼んでいます。

### 正常に動作するコード

`routes/api.php` は同じです。

```php
Route::get(
    '/test',
    [Test::class, 'test_function']
);
```

`resources/js/components/EnduserSearchstore.vue` では、URLを `/api/test` に直します。

```js
import axios from 'axios';

export default {
    data() {
        return {
            users: {},
        }
    },
    methods: {
        getBigCategory() {
            axios.get('/api/test')
                .then((response) => {
                    // this.users = response.data.users
                    console.log(response)
                })
        }
    },
    created() {
        this.getBigCategory()
    }
}
```

## 修正内容

axiosでURLを指定している箇所を、`axios.get('/api/test')` に変えます。

Laravelでは、`routes/api.php` で定義したルートに、デフォルトで `/api` の接頭辞が付きます。
そのため、エラーになるコードでVue側から呼んでいたURLは、存在しない遷移先になっていました。

## まとめ

* `routes/api.php` のルートには、デフォルトで `/api` が付きます。
* axiosで呼ぶURLは、`/api` から始まる形で指定しましょう。
* 404のときは、まずルート定義とリクエストURLが一致しているか確認します。
* `php artisan route:list` を実行すると、実際に登録されているURLを確認できます。
