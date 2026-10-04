---
title: "JavaのRandomクラスでランダムな整数を生成する方法"
emoji: "🎲"
type: "tech"
topics: ["java", "random", "beginner"]
published: false
published_at: "2021-09-23 09:54"
---

> 初出: 2021年9月23日(旧ブログの記事を移行・加筆)

JavaのRandomクラスで、ランダムな整数を生成する方法をまとめました。
乱数を使ったプログラムを書きたい、Java初学者の方に向けたメモです。

## 結論

`java.util.Random` をインスタンス化して、`nextInt(n)` を呼ぶと、`0` から `n - 1` までのランダムな整数を得られます。

## 使い方

1. `Random` クラスを、インポートしてインスタンス化します。

   ```java
   import java.util.Random;

   Random rm = new Random();
   ```

2. `nextInt()` メソッドを呼びます。

   ```java
   rm.nextInt();
   ```

3. 引数で、整数の範囲を指定します。引数に指定した数の範囲の数が生成されます。
   * 引数に `10` を指定すると、0〜9の10個の数字です。
   * 引数に `6` を指定すると、0〜5の6個の数字です。

   ```java
   rm.nextInt(10);
   ```

4. ランダムな数字を、プログラムに当てはめます。

## 例: 配列からランダムに出力する

500、100、50、10、5、1の6個の数字を入れた `int` 型の配列から、ランダムに1つ出力するプログラムです。

```java
import java.util.Random;

public class Main {
    public static void main(String[] args) {
        int[] coinCase = new int[6];
        coinCase[0] = 500;
        coinCase[1] = 100;
        coinCase[2] = 50;
        coinCase[3] = 10;
        coinCase[4] = 5;
        coinCase[5] = 1;

        Random rm = new Random();
        int no = rm.nextInt(6);

        System.out.println(coinCase[no]);
        // no に 0〜5 のランダムな数字が入る。
        // 配列のインデックスに指定することで、500〜1 の数字をランダムに出力できる
    }
}
```

## まとめ

* `Random` クラスの `nextInt(n)` で、`0` 〜 `n - 1` のランダムな整数が得られます。
* 配列のインデックスに使うと、要素をランダムに選べます。
