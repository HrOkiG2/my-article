---
title: "Javaのラッパークラスとは？基本型をオブジェクトとして扱う方法"
emoji: "🎁"
type: "tech"
topics: ["java", "wrapperclass", "beginner"]
published: false
published_at: "2021-09-19 23:52"
---

> 初出: 2021年9月19日(旧ブログの記事を移行・加筆)

Javaのラッパークラスの基本を、初心者向けにまとめました。
基本型の値を、変換したり、メソッドで加工したりしたいJava学習者の方に向けた内容です。

## 結論

ラッパークラスは、**基本型の値を包んで、オブジェクトとして扱えるようにするクラス**です。
難しく考えず、「基本型を包むと、便利なメソッドが使えるようになる」くらいの認識で大丈夫です。

## 概略

ラッパークラスは、基本型の値が入った変数を、オブジェクトとして扱うためのものです。
オブジェクトとして扱えるので、クラスが持つメソッドで、値を加工できます。

| 基本型 | ラッパークラス |
| --- | --- |
| `int` | `Integer` |
| `double` | `Double` |
| `boolean` | `Boolean` |
| `char` | `Character` |

## ラッパークラスとは

Javaは型に厳格な言語(型安全)です。
一度決めた型を、途中で勝手に変えることはできず、コンパイルエラーになります。

```java
public static void main(String[] args) {
    int i = 2021;
    // 文字列として扱いたい
    String s = i; // コンパイルエラー
}
```

そこで、ラッパークラスで基本型の値を包むと、さまざまな形に加工できます。

```java
int i = 1;                          // int型の変数
Integer itg = Integer.valueOf(i);   // ラッパークラスに変換(ボクシング)
String s = String.valueOf(i);       // 文字列に変換
System.out.println(s);              // 文字列の "1" が出力される

int n = Integer.parseInt("123");    // 文字列からintに変換
```

:::message
`new Integer(i)` は、Java 9以降は非推奨です。`Integer.valueOf(i)` を使いましょう。
また、`Integer.parseInt()` は、文字列をintに変換するメソッドです。intを文字列に変換するわけではありません。
:::

## 難しく考えない

基本型は、そのままではメソッドを持っていません。
メソッドを使うには、クラスが必要です。
そこで、さまざまな加工方法(メソッド)を持つラッパークラスで、基本型を包んで使います。

## まとめ

* ラッパークラスは、基本型を包んで、オブジェクトとして扱うためのクラスです。
* `Integer.valueOf()` で包み、`Integer.parseInt()` で文字列からintに変換できます。
* Java 9以降は、`new Integer()` ではなく `valueOf()` を使いましょう。
