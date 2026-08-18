---
date: 2023-03-15T01:20:36+09:00
draft: false
title: "メッセージの往復回数をカウントするSQLクエリの書き方"
tags: ["sql", "bigquery"]
---

タイトルどおりメッセージの往復回数をカウントするクエリの書き方です。
正確には異なるユーザがメッセージを送った回数をカウントする方法なのでチャットルームが1vs1なら往復回数だし3人以上いたら発信者が切り替わった回数と見ることができます。
最近はチャットサポートが当たり前になってきましたしチャットを提供するサービスも一般化してきたので使う機会は割とあるのではないでしょうか。
環境はBigQueryを想定しています。

# データのイメージ
こういう感じのデータをイメージしています。
massagesというテーブルに下記のようなデータが入っているとします。

| id | room_id | user_id | created_at |
| ---- | ---- | ---- | ---- |
|  1  |  300  |  600  | 2023-01-01 00:00:00 |
|  2  |  300  |  700  | 2023-01-01 01:01:01 |
|  3  |  300  |  600  | 2023-01-01 08:01:02 |
|  4  |  400  |  800  | 2023-02-01 00:10:03 |
|  5  |  400  |  900  | 2023-03-01 01:00:04 |
|  6  |  400  |  900  | 2023-04-01 0500:05 |

idはmassageのユニークキーです。
room_idはチャットルームごとのユニークキーです。
room_idごとに何往復のチャットがやりとりされたのか数え上げることが目的です。
user_idはチャットルームに参加しているユーザのidでmassageを誰が送ったのかわかります。
ここではチャットルームに2人のユーザがいることを想定しています。

# 例文

```
with massages_with_lag as (
    select
        id,
        room_id,
        user_id,
        lag(user_id, 1) over (partition by room_id order by created_at) as lag_user_id
    from
        massages
)

select
    room_id,
    ceil(count(case when user_id <> lag_user_id then id end) / 2) as count_round_trips
from
    massages_with_lag
group by
    room_id
```

このクエリは2つにわかれています。
前半は各メッセージごとに1つ前のメッセージのuser_idを横並びにさせ、後半でuser_idを比較して違ったらカウントが1つ増え、それを半分で割って小数点以下を繰り上げることで往復回数としています。

さて、前半のmassages_with_lagの内容を解説します。
このクエリではlag_user_idにwindow関数を使って対象とするメッセージの1つ前のメッセージのuser_idを横並びにしています。
lagは指定した数だけ前の行のデータを取ってくる関数で、`order by creatd_at` でcreated_atの昇順にデータを並び替えlag関数で1つ前のデータをもってくるということです。
`partition by room_id` をつけることでこれがroom_idごとに計算されるということです。
ちなみに一番最初のメッセージは前のメッセージがないのでlag_user_idはnullになります。

後半は前半の結果を利用して実際にcountで数え上げます。
`count(case when user_id <> lag_user_id then id end)` はcase式を使ってuser_idとlag_user_idを比較して不一致であればidを返しそれをcountします。
つまり今の行と前の行でuser_idが違うならば1増えるので、これが2増えるごとに往復回数が1増えることになります。
なので2で割って往復回数にしています。
しかし、割った結果が小数点だと往復回数が0.5回というのは使いにくいですね。
往復回数が知りたいだけなので一方的にメッセージを送ったままのときは往復回数にカウントしないようにしたいです。
ceilという関数で小数点以下を切り上げる関数を使うことでうまく結果を整数にし直感的な往復回数としています。

以上で往復回数を集計することができます。
もし、間違いやもっと良い方法があれば教えてください。

# 参考
window関数

https://cloud.google.com/bigquery/docs/reference/standard-sql/window-function-calls?hl=ja

lag関数

https://cloud.google.com/bigquery/docs/reference/standard-sql/navigation_functions?hl=ja#lag
