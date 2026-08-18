---
date: 2023-04-28T23:18:34+09:00
draft: false
title: "SQLでカテゴリごとに最大・最小の値を持つ行を取得する"
tags: ["sql", "bigquery"]
---

SQLではよくあるケースだけど案外書くのが難しい書き方。
これが一番簡単だと思いますが、もっといい方法があったら教えてください。

# やりたいこと
なんらかのカテゴリーごとに順番で並べて一番数値が高い行の他のカラムのデータを取得するクエリです。
たとえば、下記のitemsというテーブルがあったときcategoryごとにpriceが最も高いnameを取得するという問題です。

itemsテーブル
|  category  |  name  |  price  |
| ---- | ---- | ---- |
|  belt  |  a  |  2000  |
|  belt  |  b  |  10000  |
|  belt  |  c  |  4000  |
|  wallet  |  d  |  3000  |
|  wallet  |  e  |  60000  |
|  wallet  |  f  |  15000  |
|  wallet  |  g  |  20000  |

求める結果
|  category  |  name  |  price  |
| ---- | ---- | ---- |
|  belt  |  b  |  10000  |
|  wallet  |  e  |  60000  |


# クエリの例
この問題はwindow関数でrow_number関数を使うことで記述する方法がオススメです。

```
select
    category,
    name,
    price
from (
    select
        category,
        name,
        price,
        row_number() over (partition by category order by price desc) as price_order
    from
        items
)
where
    price_order = 1
```

なにをやっているのか簡単に言うと `partition by category` でcategoryごとにわけて `order by price desc` でprice降順で並びかえたところに `row_number()` で順番に番号を割り当てています。
これによってpriceが最も高いとprice_orderカラムに1が入り降順で番号が割り当てられます。
これをサブクエリとして `price_order = 1` に絞り込むことで最大の値が手に入るという仕組みです。
注意点として同じpriceの行が複数存在する場合はどれか1行しか得られません。

ちなみに、最小の値がほしいならorder by で並び変えるところを降順でなく昇順にすればOKです。
さらに最後に絞り込むところで任意の数値を使うことで好きな順番の行を取得できます。
ので二番目に大きい行がほしいとかも同じ書き方で取得できます。

# BigQueryの便利ワザ
BigQueryだとQUALIFY句という便利なものがありサブクエリにしなくても一発で最大の値を取ることができます。

```
select
    category,
    name,
    price,
    row_number() over (partition by category order by price desc) as price_order
from
    items
qualify
    price_order = 1
```

qualify句はgroup byにおけるhaving句のwindow関数版です。
window関数が実行されたあとに評価されて絞り込まれます。
サブクエリを使わずとも1発でほしい結果が得られます。
たぶんBigQueryにしかないはず。

# 余談： 相関サブクエリ
今回はwindow関数を使いましたが同じ結果はサブクエリを使うことでも得られます。

```
select
    category,
    name,
    price_max,
from
    items as items_1
    inner join (
            select
                category,
                max(price) as price_max
            from
                items
            group by
                category
    ) as items_2
    on items_1.category = items_2.category
    and items_1.price = items_2.price
```

この書き方はwindow関数よりも読みにくく処理も遅くなるので非推奨です。
メリットとして同じpriceを持つ行が複数あっても取得することができます。