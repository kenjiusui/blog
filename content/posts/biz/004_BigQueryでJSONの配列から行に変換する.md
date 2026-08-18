---
date: 2022-12-07T11:30:18+09:00
draft: false
title: "BigQueryでJSONの配列から行に変換する"
tags: ["SQL", "BigQuery", "JSON"]
---

valueに配列を含むJSONから配列を抽出し行へ変換するクエリの書き方

# 前提
下記のようなJSONが "テーブル名：table_1" の "列名：item_logs" に格納されいてるとき

```
{
    "user_id": 1111,
    "item_names": [
        "foo",
        "bar",
        "hoge"
    ]
}
```

このようなテーブルに変換するためのクエリ

| user_id | item |
| ---- | ---- |
| 1111 | foo |
| 1111 | bar |
| 1111 | hoge |

環境はBigQuery

# 書き方

1. json_query_arrayでほしい配列が格納されているペアのkey "$.item_names" を指定し目的の配列をとってくる
2. unnestで配列を行に変換する（配列の順序が保証されないらしいので必要ならoffsetを使う）
3. 上記で行に変換したものと元のテーブルをcross joinして目的のテーブルが完成

```
select
    user_id,
    json_value(items) as item,
from
    table_1 as t_1
cross join
    unnest(json_query_array(t_1.item_logs, "$.item_names")) as items
```

これを使ってJSONから `item = "foo"` のときだけフラグを建てる処理も簡単に書ける。
JSON値は比較とかできないのでjson_valueで変換する。

```
select
    id,
    json_value(items) = "foo" as is_foo,
from
    table_1 as t_1
cross join
    unnest(json_query_array(t_1.item_logs, "$.item_names")) as items
```

変更があったり間違っていたらコメントください。

# Ref.
JSONの取り扱い
https://cloud.google.com/bigquery/docs/reference/standard-sql/json-data#query_json_data

json_query_array  
https://cloud.google.com/bigquery/docs/reference/standard-sql/json_functions#json_query_array

json_value  
https://cloud.google.com/bigquery/docs/reference/standard-sql/json_functions#json_value
