---
title: "SQLのテーブル結合の種類まとめ: 内部結合・外部結合・自己結合・クロス結合など"
emoji: "🧩"
type: "tech"
topics: ["sql", "oracle", "mysql", "join", "database"]
published: false
published_at: "2022-01-15 16:19"
---

> 初出: 2022年1月15日(旧ブログの記事を移行・加筆)

SQLのテーブル結合の種類を、サンプルデータとあわせてまとめました。
参考書で結合の名前がたくさん出てきて、こんがらがった初学者の方や、SQL資格を勉強している方に向けた内容です。

## 結論

実務では、**内部結合と外部結合**を知っていれば、ほぼやっていけます。
ほかの結合は、「そういう結合もあるんだ」くらいの理解で大丈夫です。

私の経験では、SQLの実務経験が6ヶ月(2022年1月時点)ですが、内部結合と外部結合がほとんどでした。(自己結合と非等価結合は、1、2回です。)

## 結合の種類

1. 内部結合
2. 外部結合(左外部結合、右外部結合)
3. 完全外部結合
4. 自己結合
5. クロス結合
6. 自然結合
7. 非等価結合

## サンプルデータ

```sql
SQL> select * from testJoin;

        NO NAME       P        AGE T
---------- ---------- - ---------- -
         1 michel     1         20 1
         2 jordan     2         30 1
         3
         4 Lebron     1         25 1
         5 hanamiti   2         18 2
         0 Popovich   0         80 2

SQL> select * from testJoin2;

        NO POSITION
---------- ----------
         1 PG
         2 SG
         3 Substitute
```

## 内部結合

**結合キーが一致する行だけ**を取得します。
次のSQLでは、`testJoin2` に「0」が存在しないので、`testJoin` の「No:0、Popovich」は取得されません。
マスタデータがないデータを除外したいときによく使います。

```sql
select t1.name, t2.POSITION
from testJoin t1
inner join testJoin2 t2 on t2.NO = t1.P;

NAME       POSITION
---------- ----------
michel     PG
jordan     SG
Lebron     PG
hanamiti   SG
```

## 外部結合(左外部結合、右外部結合)

**基準になるテーブルのデータをすべて残した**うえで、結合します。
次のSQLは、`testJoin` が基準なので、結合キーに一致しない「No:0」と「No:3」も取得できます。

```sql
select t1.no, t1.name, t2.POSITION
from testJoin t1
left join testJoin2 t2 on t2.NO = t1.P
order by t1.no;

        NO NAME       POSITION
---------- ---------- ----------
         0 Popovich
         1 michel     PG
         2 jordan     SG
         3
         4 Lebron     PG
         5 hanamiti   SG
```

## 完全外部結合

**結合キーが一致しなくても、両方のテーブルのすべてのデータ**を取得します。
一致しない「No:0、Popovich」「No:3」と、「Substitute」の行も取得されます。
正直、使ったことはありません。

```sql
select t1.no, t1.name, t2.POSITION
from testJoin t1
full outer join testJoin2 t2 on t2.NO = t1.P
order by t1.no;

        NO NAME       POSITION
---------- ---------- ----------
         0 Popovich
         1 michel     PG
         2 jordan     SG
         3
         4 Lebron     PG
         5 hanamiti   SG
                      Substitute
```

## 自己結合

**同じテーブルを、別のテーブルのように扱って**、データを取得します。
別名を付ける必要があります。

次のSQLは、同じチームに所属している選手同士を出力しています。
`WHERE` 句で名前の比較をして、同じ組み合わせ(AとB、BとA)を除外しています。

```sql
select t1.no, t1.name, t1.team_id, t2.no, t2.name, t2.team_id
from testJoin t1
inner join testJoin t2 on t1.team_id = t2.team_id
where t1.name > t2.name
order by t1.no;
```

## クロス結合

結合するテーブルのデータ数を**掛け算**した行数を返します。
`testJoin` が6行、`testJoin2` が3行なので、6 × 3 = 18行が取得されます。

```sql
select t1.name, t2.POSITION
from testJoin t1, testJoin2 t2
order by t1.no, t2.no;
```

## 自然結合

**結合するテーブル同士で、同じ名前・同じデータ型のカラムを、SQLが判断して結合キーにしてくれる**構文です。
次のSQLでは、両方のテーブルにある `NO` 列が、結合キーになります。
実務では、使うことはあまりないと思います。

```sql
select no, t1.name, t2.POSITION
from testJoin t1
natural left join testJoin2 t2;
```

## 非等価結合

結合条件に、**`=` を使わない**結合です。
`=` を使わないだけなので、難しく考える必要はありません。

```sql
select *
from testJoin t1
inner join testJoin2 t2 on t1.P <> t2.NO
where t1.no < 3
order by t1.no;
```

## まとめ

* 実務では、内部結合と外部結合が中心です。
* 内部結合は「一致する行のみ」、外部結合は「基準テーブルを全部残す」です。
* 完全外部結合、自己結合、クロス結合、自然結合、非等価結合は、用途が限られます。
