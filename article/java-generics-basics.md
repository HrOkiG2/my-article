---
title: "Javaのジェネリクスとは？型安全なコレクションを作る"
emoji: "🧪"
type: "tech"
topics: ["java", "generics", "collection", "beginner"]
published: false
published_at: "2021-09-18 17:38"
---

> 初出: 2021年9月18日(旧ブログの記事を移行・加筆)

Javaのジェネリクスの基本を、型安全の考え方から整理しました。
PHPのような動的型付け言語から、Javaに移ってきた方に向けた内容です。

## 結論

ジェネリクスは、**コレクションに入れられる型を指定する**仕組みです。
`List<String>` のように `<型>` を書くことで、型安全にコレクションを使えます。

## 前提: Javaの型安全

Javaには、「型安全」という考え方があります。
変数やメソッドに型を指定して、安全にプログラムを組み立てる、という考え方です。

たとえば、変数 `moji` に `1` を入れたとします。
型を指定していないと、文字の `1` なのか、数字の `1` なのか、判別できません。
この食い違いを防ぐために、Javaでは型を指定するのがルールです。

```java
public class Num {
    public static void main(String[] args) {
        String moji = "1";
        System.out.println(moji + moji); // 11(文字列の連結)

        int num = 1;
        System.out.println(num + num);   // 2(数値の加算)
    }
}
```

PHPのように、型を指定しない言語を**動的型付け言語**、Javaのように型を明示する言語を**静的型付け言語**と言います。
PHPから来た人は、ストレスを感じるところですが、少しずつ慣れていきましょう。

## ジェネリクスとは

ジェネリクスは、**コレクションに型を指定する方法**です。
コレクションは、値の追加や保持ができる、Java特有の配列のようなものです(厳密には配列ではありません)。

```java
// <> の部分にStringを指定。文字列しか入れられないコレクションになる
List<String> box = new ArrayList<String>();
box.add("hello");
box.add("bye");

for (String s : box) {
    System.out.println(s);
}
```

:::message
**配列とコレクションの違い**

どちらも複数の値を持てますが、配列は、定義するときに要素数を決める必要があり、途中で追加できません。
コレクションは、あとから値を追加できます。
:::

### ジェネリクスを使う目的

コレクションは、関連する情報をまとめて扱いたいときに使います。
たとえば、ポジション(ガード、フォワード、センター)のようなまとまりです。
そのため、ループ処理と一緒に使われることが多いです。

ここで問題になるのが、**コレクションには、何でも入れられてしまう**ことです。

```java
// 型を指定しないコレクション(生の型)
List box = new ArrayList();
box.add(1);
box.add("aiueo");
box.add("hello");
// box の中身: { 数字の1, 文字列の aiueo, 文字列の hello }

for (Object o : box) {
    String s = (String) o; // 数字の1を文字列にキャストしてしまい、実行時にエラー
}
```

様々な型が混ざっていると、ループの中で、同じ型として扱えません。
そこで、ジェネリクスで型を指定して、1つの型だけ入れられるコレクションにします。

```java
List<String> box = new ArrayList<String>(); // String型として扱う宣言
box.add("1");
box.add("aiueo");
box.add("hello");
// box.add(1); // コンパイルエラー。文字列以外は入れられない

for (String s : box) {
    System.out.println(s);
}
```

`<型名>` を書くことで、コレクションの扱い方を決められます。
1つの型しか入れられないので、**コンパイル時に型の間違いに気づけて**、型安全が保てます。

## 便利な使い方

クラスを作るときに、型の代わりに `E` や `T` などの文字を置いておき、使うときに型を決める方法があります。
この文字は、`T`、`E`、何でも構いません。「使うときに型を決めてね」という意味です。

```java
// 型をTとしておく
public class Box<T> {
    private T value;

    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

public class Main {
    public static void main(String[] args) {
        Box<String> a = new Box<String>();
        a.set("ジェネリクス");

        Box<Integer> b = new Box<Integer>(); // intではなくIntegerを指定
        b.set(10);
    }
}
```

自分で作ったクラス(たとえば `Hero`)を、型に指定することもできます。

:::message
ジェネリクスの型には、`int` などの基本型は指定できません。`Integer` などのラッパークラスを指定します。
:::

## まとめ

* ジェネリクスは、コレクションに入れられる型を指定する仕組みです。
* 型を指定すると、コンパイル時に型の間違いに気づけます。
* クラスに `<T>` を付けると、使うときに型を決められます。
* 基本型は指定できないので、ラッパークラスを使いましょう。
