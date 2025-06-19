---
title: "Databricksで頑張るAIエージェント入門"
emoji: "🌱"
type: "tech"
topics: ["databricks","AIエージェント"]
published: false
publication_name: "headwaters"
---


# 1. DatabricksでAIエージェントを作ってみたい

## 「Agent Bricks」とやらが発表された！…けど

絶賛Databricks勉強中のせっとです。
実際に触ったり情報集めたりとしているのですが、先週のサミットで以下の発表がありました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/a3a1bb56-b88d-4784-b43c-48c9552acd0b.png)

https://www.databricks.com/company/newsroom/press-releases/databricks-launches-agent-bricks-new-approach-building-ai-agents?utm_source=chatgpt.com

私もまだ理解が追い付いていない箇所がたくさんありますが、簡単に言うと
**「エージェントを簡単に作成できるプラットフォーム"Agent Brick"の提供が始めたよ！」** とのことです。

このAgent Bricksを使うメリットとして、

- **開発コスト削減**
- **デプロイまでのスピード感と運用効率**
- **アクセス制御や監査が容易**

などが大きく主張されています。
他にもサミットの動画や記事をいくつか拝見しましたが、モデル精度やスコアにこだわっているというよりは、とにかく **「ビジネス現場での利便性」** にこだわっている印象を受けました。

今回このAgent Bricksを使用してみたかったのですが、まだFree Editionだとツールバーに表示がありませんでした。
Beta版だからでしょうか？正確な情報は見つけ切れませんでした。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/f20dced2-523f-4422-b603-d33b909d90ec.png)
*本当はこんな感じで表示があるらしい。*


<iframe width="560" height="315" src="https://www.youtube.com/embed/VEYYgy0HrqM?si=QPuVH4_02mpZziRj" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


ない。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/d4d6c732-f686-49f8-861e-fc9ccaa2f9f6.png)


## 「Mosaic AI エージェント」ならできそう

ただ、Databricksには他にもエージェントを構築できるフレームワークがあるようで、それがこの **「Mosaic AIエージェントフレームワーク」**

https://www.databricks.com/jp/blog/build-autonomous-ai-assistant-mosaic-ai-agent-framework

昨年11月にリリースされているのですかね。勉強不足でした。
ざっくり一言でいうと **「組織専用のChatGPTみたいなものを、安全に・正確に動かすためのフレームワーク」** のようです。

セキュリティの不安を解消してくれるのは嬉しいですね。
今回はこのMosaic AIエージェントをプチ構築してみたいと思います。

## 本記事でできるようになること

結論、こんな感じ。
どちらもdatabricks上のplaygroundで動かしているものですが、このチャットをAzureやAWSで簡単にデプロイできる。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/5e46ef30-c07f-4b8d-ad02-7300f232fa52.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/c0e7f990-1449-4c7c-8ff4-cc2c5030260e.png)


「でもこの会話なら普通のチャットボットと変わらんやん」と言われそうですが、実際そうでして…。
今回は下記クイックスタートガイドを参考に実行したのですが、このチュートリアルだけですと、**MLflowとChatAgentを最小構成で使った「チャットボット」** に近い状態です。

https://docs.databricks.com/aws/en/generative-ai/tutorials/agent-quickstart


そのため「AIエージェント感（＝判断・外部情報・動作の連鎖）」はほぼありません。

ただ「AIエージェント感」は薄くても、Databricksにおける生成AI／エージェントの実運用に必要な重要コンセプトを学べるポイントはいくつかあると思います。
そのため、一旦は上記ガイドを実践しながら調べながら、いろいろ書いてみようと思います。

# 2. ざっくり全体構成

ざっくりと全体構成を見ると、こんな感じです。

```scss
[1] LLM選定      → [2] エージェント定義 → [3] モデル登録        → [4] 本番デプロイ
(使えるモデル確認)    (ChatAgent実装)       (MLflow + Unity Catalog) (REST API化)
```

改めて特徴を挙げるとこんな感じかと思います。
- エージェントの定義→登録→運用を **すべてDatabricksで完結できる**
- MLflowとUnity Catalogによる **再現性とガバナンスの担保**
- エンドポイント経由で社内エージェントを **ノーコードで** 呼び出せる

# 3.実装ステップ

では実際にここからはガイドを実行してみたいと思います。

##  (1) LLMエンドポイントの選択

最初に、Databricks上に登録されたLLMの中から今使えるLLMを自動的に選ぶ処理を行います。
なんだか不思議な処理な気がしたのですが、どうやらDatabricksでは、
**LLMは「モデル」ではなく「エンドポイント名」で指定**し、
**候補リストを順に試して、最初に成功したものを採用する**
という仕組みがよく使われるようです。

「本番環境からはモデルを入れ替えたい」や「実験的に複数LLMを比較したい」ということが容易にできるのですね。

<details><summary>（折りたたみ）LLMエンドポイントの選択</summary>

```python
# 最初に使えるLLMエンドポイントを自動で選ぶための準備
LLM_ENDPOINT_NAME = None

from databricks.sdk import WorkspaceClient

# 指定したエンドポイントが使えるかどうかをチェックする関数
def is_endpoint_available(endpoint_name):
    try:
        # DatabricksのOpenAI互換クライアントを取得
        client = WorkspaceClient().serving_endpoints.get_open_ai_client()
        # テスト用のメッセージを送って、応答が返ってくるか確認
        client.chat.completions.create(
            model=endpoint_name,
            messages=[{"role": "user", "content": "AIとは何ですか？"}]
        )
        return True  # 応答があればOK
    except Exception:
        return False  # エラーが出たら使えない

# Databricksクライアントを初期化
client = WorkspaceClient()

# 候補のエンドポイントを順番にチェック
for candidate_endpoint_name in [
    "databricks-claude-3-7-sonnet", 
    "databricks-meta-llama-3-3-70b-instruct"
]:
    if is_endpoint_available(candidate_endpoint_name):
        # 最初に使えたエンドポイントを選ぶ
        LLM_ENDPOINT_NAME = candidate_endpoint_name

# どれも使えなかったらエラーを出す
assert LLM_ENDPOINT_NAME is not None, "LLM_ENDPOINT_NAMEを指定してください"
```
</details>

## (2) エージェントの定義

次は **「Databricks上で動作する生成AIエージェントのロジック」** を構築します。
これがAIエージェントの中枢（思考＋実行）に当たる部分です。

以下のコードで、Mosaic AI エージェントフレームワークで提供されているチャットエージェント用の抽象クラス`ChatAgent`を拡張して、

1. ユーザーとの対話を受け取り、
1. LLMを使って応答を生成し、
1. 応答を構造化された形式で返す

という処理ができるようになります。

<details><summary>（折りたたみ）エージェントの定義</summary>

```python
import uuid
import mlflow
from typing import Any, Optional

from mlflow.pyfunc import ChatAgent  # MLflowのチャットエージェントのベースクラス
from mlflow.types.agent import ChatAgentMessage, ChatAgentResponse, ChatContext

mlflow.openai.autolog()  # LLMの呼び出しやツール実行をMLflowに自動ログ

# ChatAgentを継承して、独自のエージェントを定義
class QuickstartAgent(ChatAgent):
    def predict(
        self,
        messages: list[ChatAgentMessage],  # ユーザーとの会話履歴
        context: Optional[ChatContext] = None,  # 会話の文脈（使わない場合もある）
        custom_inputs: Optional[dict[str, Any]] = None,  # カスタム入力（オプション）
    ) -> ChatAgentResponse:
        
        # 1. 最後のユーザーメッセージ（プロンプト）を取り出す
        prompt = messages[-1].content

        # 2. run_agent関数を呼び出して、LLMの応答を取得
        raw_msgs = run_agent(prompt)

        # 3. 応答メッセージをChatAgentMessage形式に変換して返す
        out = []
        for m in raw_msgs:
            out.append(ChatAgentMessage(id=uuid.uuid4().hex, **m))

        return ChatAgentResponse(messages=out)
```

</details>

上記のコードは、この全体のなかでも特にポイントだと思っています。
```python
class QuickstartAgent(ChatAgent):
    def predict(...):
        ...
```

この `predict`の中に「単にLLMを呼び出す」だけでなく、
**外部ツールの実行、状況に応じた処理分岐、他システムとの連携といった処理を組み込む** ことで、いわゆる AIエージェントらしい **“ふるまい”** を実現できます。

本来のAIエージェントとしての振る舞いを目指すのであれば、このクラスこそが最も重要な実装ポイントになります。
たとえば、ユーザーの質問に応じて自動的に社内のデータベースを検索したり、Pythonで計算やグラフを生成して返したりするような処理は、すべて `predict()` の中で制御できます。

この先の応用は一旦今回スキップして次回チャレンジしたいと思います。

## (3) Unity CatalogにMLflowモデルとして保存


次のフェーズは、Databricks上で構築したAIエージェントを **MLflowモデルとして登録・保存する処理** です。
単にPythonファイルを保存するだけでなく、**使用リソース（LLMやPython関数）ごとエージェントをパッケージ化して、組織的に管理・再利用可能にする** というのがポイントです。

<details><summary>（折りたたみ）Unity CatalogにMLflowモデルとして保存</summary>

```python
#必要なライブラリをインポート
import mlflow
from mlflow.models.resources import DatabricksFunction, DatabricksServingEndpoint
from pkg_resources import get_distribution
from quickstart_agent import LLM_ENDPOINT_NAME  # 使うLLMのエンドポイント名を取得

# モデルを登録するカタログとスキーマを指定（デフォルトは current_catalog() と "default"）
catalog_name = spark.sql("SELECT current_catalog()").collect()[0][0]
schema_name = "default"
registered_model_name = f"{catalog_name}.{schema_name}.quickstart_agent"

# エージェントが使うリソース（LLMエンドポイントとPython実行ツール）を指定
resources = [
    DatabricksServingEndpoint(endpoint_name=LLM_ENDPOINT_NAME),
    DatabricksFunction(function_name="system.ai.python_exec"), 
]

# モデルの保存先をUnity Catalogに設定

mlflow.set_registry_uri("databricks-uc")  

# MLflowでモデルを保存・登録

with mlflow.start_run():
    logged_agent_info = mlflow.pyfunc.log_model(
        artifact_path="agent",  # モデルの保存先（相対パス）
        python_model="quickstart_agent.py",  # エージェントのPythonファイル
        extra_pip_requirements=[
            f"databricks-connect=={get_distribution('databricks-connect').version}"
        ],  # 必要なライブラリを指定
        resources=resources,  # エージェントが使うリソースを指定
        registered_model_name=registered_model_name,  # Unity Catalogに登録するモデル名
    )

```
</details>


この登録処理の後は、MLflow UIやAPIからこのエージェントを呼び出して使うことができます。


ここではリソースとして、以下2つを指定しています。
- `DatabricksServingEndpoint`：LLMを呼び出すためのエンドポイント
- `DatabricksFunction` ：　任意のPythonコードを実行できる関数（今回は `system.ai.python_exec`）


```py
resources = [
    DatabricksServingEndpoint(endpoint_name=LLM_ENDPOINT_NAME),
    DatabricksFunction(function_name="system.ai.python_exec"),
]
```

このように、LLMとツール（関数）を組み合わせることで、**エージェントに「外部と連携する力」を持たせる** ことができます。
さらにここに検索用の関数や、社内APIとの連携機能などを追加すれば、より高度で自律的なAIエージェントを構築することも可能です。



ちなみにCatalogの画面はこんな感じ。
一旦何もないですが、Unity Catalogを使うことで、
**モデル単位で「部門別」「用途別」に権限管理** が可能となり、
「A部門のエージェントはA部門しか呼び出せない」といった制御もできるようになります。



![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/65828fd2-10fd-4883-9900-1657e1bf8b71.png)


## (4) デプロイ

最後に、Databricks上に構築・登録されたAIエージェントを「REST API」としてデプロイします。

従来の「MLモデルのサービング」と違うのは、**チャット形式で応答できるAIエージェントが即座にAPIとして使える状態になる** という点かと思います。

<details><summary>（折りたたみ）デプロイ</summary>

```python
from databricks import agents

deployment_info = agents.deploy(
    model_name=registered_model_name,  # Unity Catalogに登録したモデル名
    model_version=logged_agent_info.registered_model_version,  # 登録されたモデルのバージョン
)
```

</details>

上記実行後、数分待って「サービング」を見てみるとこんな感じ。

![サービング.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/d1a46ab7-5a8c-468f-b51f-6771e501e6bf.png)

ここで右上の「使用」⇒「Playgroundで試す」で冒頭のチャット画面に飛ぶことができます。




# 4. まとめ：DatabricksのAIエージェントでできそうなこと

今回は Databricks上で動くAIエージェントのプチ構築を体験してみました。

ここまで紹介してきたように、Mosaic AI エージェントフレームワーク に含まれる `ChatAgent` クラスをベースに、LLMを組み込んだAIエージェントを比較的簡単に構築できます。

ただし、本質的な価値は「ただのチャット応答」ではなく、その先にある**実業務への応用**にあります。

以下は、今回のコードを出発点に、**実務で役立つエージェントに発展させる具体的な方向性**です。

1. **議事録からのToDo抽出＋タスク登録エージェント**
   　→ `predict()` に要約・ToDo抽出処理を組み込み、`DatabricksFunction` でAsanaやJira APIを呼び出してチケットを作成

2. **CSVアップロードで即集計・レポート生成エージェント**
   　→ `custom_inputs` 経由でCSVを受け取り、DatabricksでPython実行 → グラフ付きレポートを出力・保存

3. **Slack経由で回答する社内FAQボット**
   　→ `predict()` でRAG（検索拡張生成）を組み込み、Slack連携で「チャットで聞けば答えてくれる」社内ナレッジBotを実装


このように、DatabricksのAIエージェントは単なるチャット応答に留まらず、**業務フローに入り込むアシスタント**として活用できます。
コードの再利用性やMLflow/Unity Catalogによる一元管理も、企業導入に適した強みです。

DatabricksのAIエージェントは、**自然言語を窓口に、実務を支援する柔軟な自律システム**として活用できる可能性があります。
PoCから始めて段階的に育てていく開発スタイルとも相性が良いと感じました。


