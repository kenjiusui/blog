---
date: 2022-06-09T00:01:17+09:00
draft: false
title: "Cloud FunctionをCloud Composerから認証有りでHTTP起動する"
tags: ["cloudfunctions", "cloudcomposer", "gcp"]
categories: ["tech"]
---

Cloud ComposerからCloud Functionsにある関数を認証有りのHTTPトリガーで軌道する方法です。
公式のドキュメントに書いてありますが備忘録として書きます。
たぶん、これが一番簡単だと思いますが他に良い方法があったら教えてください。

# 環境
Airflow: 2.1.4

# やりかた
やることはわかってしまえば簡単です。

- 関数の実行権限をCloud Composerに付与する
- IDトークンを発行する
- HTTPリクエストのheaderにトークンを含めて送る

これだけです。

# 詳しいやり方
## 関数の実行権限をCloud Composerに付与する
GCPのコンソールからポチポチやるのが簡単だと思います。
具体的な手順はドキュメントを参照してください。

https://cloud.google.com/functions/docs/securing/authenticating#authenticating_function_to_function_calls

ポイントは受信する側の関数ごとにroles/cloudfunctions.invokerを設定するということです。
やっていないのでわかりませんがComposerのサービスアカウントにroles/cloudfunctions.invokerを付与するのではなく関数側から設定する必要がありそうです。
一括で付与とかはできないのかな？できそうな気もするのでわかる人いたら教えてください。

## IDトークンを発行する
ここからはComposerのDAGのコードになります。
DAGでHTTPリクエストを送るまえに認証に使用するIDトークンを発行します。

```
import google.auth.transport.requests
import google.oauth2.id_token

auth_req = google.auth.transport.requests.Request()
# endpoint はCloud Functionsにある関数のエンドポイントのURL
id_token = google.oauth2.id_token.fetch_id_token(auth_req, endpoint)
```

これで得られたid_tokenが目的のものになります。
`endpoint` はCloud Functionsにある関数のエンドポイントのURLです。
このトークンの発行に使っているGoogleのライブラリはComposerであればインストールされているはずなので簡単に利用できます。

## HTTPリクエストを送る
最後はComposerからHTTPリクエストを送って関数を実行するだけです。
単純に関数を実行するだけであればエンドポイントをPOSTするだけで良いのですが認証情報をheaderに入れて送ります。

```
from airflow.providers.http.operators.http import SimpleHttpOperator

end_point = "cloud functionsの関数のエンドポイント"
invoke_task = SimpleHttpOperator(
        task_id = "invoke_function",
        endpoint=end_point,
        headers={"Content-Type": "application/json", "Authorization": f"Bearer {id_token}"},
        dag=dag
    )
```

はい、これだけでOKです。
あとはよしなにDAGのなかにこのタスクを組み込んでください。
こういう単純なHTTPリクエストを送るのはAirflowの1系だと面倒だったのですが2系ではSimpleHttpOperatorという便利なものがあるのでこれを使えばOKです。

---------
Cloud Functionsを使うのが初めてだったのでこんな単純なことをやるのに結構苦労してしまったので、もし同じようなことをする人の役に立てば嬉しいです。
もし間違っていたり、より良い方法がありましたらコメントで教えてください。
