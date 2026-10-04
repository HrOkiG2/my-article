---
title: "Javaのfinallyとtry-with-resources: ファイルを確実に閉じる例外処理"
emoji: "🔒"
type: "tech"
topics: ["java", "exception", "finally", "beginner"]
published: false
published_at: "2021-09-23 09:49"
---

> 初出: 2021年9月23日(旧ブログの記事を移行・加筆)

Javaの例外処理で、`finally` と `try-with-resources` が重要な理由をまとめました。
なんとなく `finally` を書いていた、Java初学者の方に向けた内容です。

## 結論

ファイルやDB接続など、**閉じなければいけないリソース**は、`finally` か `try-with-resources` で、確実に閉じましょう。
いまは、`try-with-resources` が推奨されています。

## finallyは重要なのか

例外処理でよく出てくるのが、ファイルの読み書きや、DB接続です。
ここでは、ファイルの書き込みを例に説明します。

まず、悪い例です。

```java
public static void main(String[] args) {
    try {
        FileWriter fw = new FileWriter("読み書きするファイル名", true);
        // ファイルに書き込み
        fw.write("aiueo");
        // 書き込みを実行
        fw.flush();
        // ファイルを閉じる
        fw.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

この書き方だと、`flush()` のあとに例外が起きたとき、`close()` が実行されず、ファイルを閉じられません。
その結果、ファイルを開けなくなったり、ファイルが壊れたりすることがあります。

そうならないように、`finally` に、ファイルを閉じる処理を書いて、**必ず閉じる**ようにします。

```java
public static void main(String[] args) {
    FileWriter fw = null;
    try {
        fw = new FileWriter("読み書きするファイル名", true);
        // ファイルに書き込み
        fw.write("aiueo");
        // 書き込みを実行
        fw.flush();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        // ここでファイルを閉じる
        try {
            if (fw != null) {
                fw.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

この処理なら、例外が起きても、最後に必ずファイルを閉じるので、ファイルの破損などの障害を防げます。

:::message
`flush()` は、「書き込んだ分を、今すぐ書き込め」という命令です。
これがないと、書き込みをまとめて実行するので、エラーが起きたときに書き込みに失敗するおそれがあります。
先輩いわく、書き込んだ分は `flush` で実行していくのが良いそうです。
:::

## try-with-resources

Javaには、閉じなければいけない処理を、もっと簡単に書ける書き方があります。
`try-with-resources` という構文です。

```java
public static void main(String[] args) {
    try (FileWriter fw = new FileWriter("記述したいテキスト名", true)) {
        // ファイルに書き込み
        fw.write("aiueo");
        // 書き込みを実行
        fw.flush();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

`try()` の中に、閉じなければいけないリソースを書くと、Javaが**自動で閉じてくれます。**
`try-catch-finally` と比べて、記述が少なく、例外が起きても起きなくても、必ず閉じてくれるので、推奨されています。

## まとめ

* ファイルやDB接続は、例外が起きても、必ず閉じる必要があります。
* `finally` に、閉じる処理を書けば、必ず実行されます。
* `try-with-resources` なら、自動で閉じてくれて、記述も少なく済みます。
