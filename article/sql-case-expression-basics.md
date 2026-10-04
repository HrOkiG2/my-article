---
title: "SQLのCASE式の使い方: SELECT・WHERE・ORDER BYで条件分岐する"
emoji: "🔀"
type: "tech"
topics: ["sql", "mysql", "oracle", "case", "beginner"]
published: false
published_at: "2021-09-18 20:04"
---

> 初出: 2021年9月18日(旧ブログの記事を移行・加筆)

SQLのCASE式の基本的な使い方を、SELECT句、WHERE句、ORDER BY句の例でまとめました。
SQLで条件分岐したい初学者の方や、Oracle Master試験の勉強をしている方に向けた内容です。

## 結論

CASE式は、SQLの中で条件分岐ができる式です。
SELECT句、WHERE句、ORDER BY句、GROUP BY句など、さまざまな場所で使えます。

## CASE式とは

SQLで条件分岐をするための式です。
JavaやPHPの条件分岐と考え方は同じで、「〇〇のときはAの処理」「〇〇のときはB」という指定ができます。

```sql
-- 単純CASE式(値の一致で判定)
CASE カラム名
  WHEN 値1 THEN '一致したときの値'
  WHEN 値2 THEN '一致したときの値'
  ELSE 'どれにも一致しないときの値'
END

-- 検索CASE式(条件式で判定)
CASE
  WHEN 条件式1 THEN '一致したときの値'
  WHEN 条件式2 THEN '一致したときの値'
  ELSE 'どれにも一致しないときの値'
END
```

不等号(`>` や `<`)を使うときは、**検索CASE式**を使います。

## 使い方

### SELECT句

取得した `age` の値を、成人か未成年かで表示する例です。

```sql
SELECT name,
  CASE
    WHEN age >= 20 THEN '成人'
    WHEN age <  20 THEN '未成年'
    ELSE '年齢不詳'
  END AS age_type
FROM syain;
```

### WHERE句

```sql
SELECT *
FROM sample
WHERE (CASE WHEN target_date > SYSDATE THEN '未来' ELSE '過去' END) = '未来';
```

### ORDER BY句

本来の並びが「総務部、開発部、サービス、営業」のときに、CASE式で、営業を一番上に取得できます。

```sql
SELECT *
FROM busyo
ORDER BY CASE busyo_name WHEN '営業' THEN 1 ELSE 2 END;
```

## 混同しがちなIF文

PL/SQLなどのプログラミング言語で使う**IF文**と、**CASE式**は、混同されがちです。
どちらも条件分岐のための予約語ですが、名前のとおり、違いがあります。

* **IF文(文)**: 複数の文の中で、処理を分岐させます。手続き型の言語(PL/SQLなど)で使います。
* **CASE式(式)**: 1つのSQL文の中で、値を分岐させます。SQLの中で使います。

## まとめ

* CASE式は、SQLの中で条件分岐できる式です。
* 値の一致なら単純CASE式、不等号などの条件なら検索CASE式を使います。
* SELECT句、WHERE句、ORDER BY句など、さまざまな場所で使えます。
* 先輩いわく、「上級者はWHERE句よりも、SELECT句でCASE式を使う」そうです。
