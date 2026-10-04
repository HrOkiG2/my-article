---
title: "Javaのオーバーロードとオーバーライドの違いをまとめた"
emoji: "🔁"
type: "tech"
topics: ["java", "overload", "override", "oop"]
published: false
published_at: "2021-09-23 10:08"
---

> 初出: 2021年9月23日(旧ブログの記事を移行・加筆)

Javaの、オーバーロードとオーバーライドの違いをまとめました。
名前が似ていて混同しがちな、Java初学者の方に向けた内容です。

## 結論

| 用語 | 意味 | 条件 |
| --- | --- | --- |
| オーバーライド(Override) | 親クラスのメソッドを、子クラスで書き換えます。 | メソッド名、引数、型がすべて同じです。 |
| オーバーロード(Overload) | 同じ名前のメソッドを、引数の違いで複数用意します。 | メソッド名は同じで、引数の数や型が違います。 |

## オーバーライド

親クラスで定義したメソッドを、子クラスで、子クラス専用の処理に書き換えることです。
Javaでは継承をよく使うので、親クラスのメソッドを、子クラス用にアレンジしたいときに使います。

次のコードは、`Animal` クラスを継承した `Dog` クラスで、`Animal` の `speak` メソッドを書き換えています。

```java
public class Animal {
    public void speak() {
        System.out.println("......");
    }
}

public class Dog extends Animal {
    // オーバーライド
    @Override
    public void speak() {
        System.out.println("わんわん");
    }

    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.speak(); // わんわん が出力される
    }
}
```

:::message
メソッド名、引数、戻り値の型が、すべて同じでないと、オーバーライドできません。
`@Override` を付けると、書き換えのミスをコンパイル時に検出できます。
:::

## オーバーロード

引数の違いによって、同じ名前のメソッドで、違う処理をさせる方法です。
本来、処理内容が違えば、別のメソッドを用意しなければいけません。
オーバーロードを使えば、呼び出すときの引数に違いを持たせるだけで、使い分けられます。

次のコードは、引数が1つの場合と2つの場合の処理です。

```java
public class Animal {
    public void speak(int i) {
        System.out.println("一人で黙々と眠る");
    }

    public void speak(int i, int y) {
        System.out.println("二人で仲良く眠る");
    }

    public static void main(String[] args) {
        Animal animal = new Animal();
        animal.speak(1);    // 一人で黙々と眠る が出力される
        animal.speak(1, 2); // 二人で仲良く眠る が出力される
    }
}
```

:::message
オーバーロードでは、引数の**型や並び順**が違っても、別の処理を書けます。
メソッドを呼び出すときは、引数の型や順番に気をつけてください。
:::

## まとめ

* オーバーライドは、親クラスのメソッドを、子クラスで書き換えることです。
* オーバーロードは、同じ名前のメソッドを、引数の違いで使い分けることです。
* オーバーライドは「継承」、オーバーロードは「引数の違い」と覚えましょう。
