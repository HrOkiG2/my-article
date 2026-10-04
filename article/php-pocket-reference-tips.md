---
title: "PHP実務でよく使う構文・デバッグTips集"
emoji: "🐘"
type: "tech"
topics: ["php", "pdo", "debug", "tips"]
published: false
published_at: "2022-03-01 06:16"
---

> 初出: 2022年3月1日(旧ブログの記事を移行・加筆)

PHPの実務で繰り返し使う構文とデバッグ手法を、ポケットリファレンスとしてまとめました。
「あの書き方なんだっけ？」とその場で引きたいPHPエンジニアを対象にしています。

## 配列は array 系関数とスプレッド構文を使いこなす

`array_merge()` の代わりに、スプレッド構文で配列を結合できます。
配列操作は実務で頻出するため、`array_*` 系の関数と合わせて使えるようにしておくと便利です。

```php
$a = [1, 2];
$b = [3, 4];

$merged = [...$a, ...$b]; // [1, 2, 3, 4]
```

## 比較は「===」で厳格に行う

**結論**: 比較には原則として `===` を使います。

* `==` は値のみを比較します。
* `===` は値に加えて型まで比較します。
* 否定は `!==` です。

PHPUnitでは型まで厳格に比較する場面が多いため、普段から `===` に統一しておくとテストが書きやすくなります。
なお、PHPの `===` はJavaのような参照先の比較ではありません。

## NULL合体演算子で初期値を切り替える

変数が `NULL` の場合に別の値へ切り替えるには、NULL合体演算子 `??` を使います。
`if` 文で `NULL` 判定をするより、短く読みやすく書けます。

```php
$Hello = NULL;

echo $Hello ?? "Not Hello";

// 「Not Hello」と出力される
```

私自身は、文字出力以外の場面ではまだ使いどころを掴みきれていません。

## var_dump() は <pre> で囲むと見やすい

配列の中身を確認するために `var_dump()` を多用すると、ブラウザ上では改行されず読みづらくなります。
`<pre>` タグで囲むと、整形された状態で表示されます。

```php
echo '<pre>';
var_dump($test);
echo '</pre>';

/*
 * 結果
 * array(3) {
 *   [0]=>
 *   string(5) "apple"
 *   [1]=>
 *   string(6) "banana"
 *   [2]=>
 *   string(6) "orange"
 * }
 */
```

## PDFの表示とダウンロードは header() で制御する

PDFの表示とダウンロードは `header()` で切り替えます。

* 表示する場合: `header('Content-Disposition: inline; filename="' . $output_filename . '"');`
* ダウンロードする場合: `header('Content-Disposition: attachment; filename="' . $output_filename . '"');`

注意点は次の2つです。

* ブラウザ表示からダウンロードさせる場合は、キャッシュの制御が必要です。
* S3に保管したPDFを扱う場合は、メタデータを `echo` で出力します。

## PDOでエラー時に実行SQLを確認する

PDOでエラーが出たものの原因が分からないときは、`debugDumpParams()` で実行されたSQLとパラメータを確認できます。

```php
return $stmt->debugDumpParams();
```

## if文を1行で書いて出力する

短縮タグ `<?= ?>` と三項演算子を組み合わせると、条件分岐と出力を1行で書けます。

```php
<?=(条件式)?trueの場合:falseの場合?>
<?=(isset($a))?"trueだよ":"falseだよ"?>
```

フォームのバリデーションエラー表示で特によく使います。

```html
<form>
  <div class="form-group">
    <label for="exampleInputEmail1">Email address</label>
    <?=(isset($error))?$error:''?>
    <input type="email" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" placeholder="Enter email">
    <small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone else.</small>
  </div>
</form>
```

## まとめ

* 配列の結合はスプレッド構文を使います。
* 比較は `===` / `!==` で厳格に行います。
* `NULL` の切り替えは `??` で簡潔に書けます。
* `var_dump()` は `<pre>` で囲むと見やすくなります。
* PDFの表示とダウンロードは `Content-Disposition` で切り替えます。
* PDOのデバッグには `debugDumpParams()` が役立ちます。
* テンプレート内の条件出力は `<?= ?>` と三項演算子で1行にできます。
