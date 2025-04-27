---
title: LLM効率化に必須の「LangChain Expression Language (LCEL)」のデバッグに注意する。
tags:
  - debug
  - LangChain
  - LLM
  - LCEL
private: false
updated_at: '2025-03-09T11:46:38+09:00'
id: fa2c23f677bed6da0eea
organization_url_name: null
slide: false
ignorePublish: false
---
# LangChain Expression Language (LCEL)とは

LangChain Expression Language (LCEL)とは、**LangChainの中で使用されるDSL（ドメイン固有の言語）** であり、データ処理やLLMのワークフローを直感的に記述できる仕組みです。

主に **LLMを活用したアプリケーション開発の効率化** を目的に使用されます。

今回はこのLCELの特徴的な記述方法、使い方をまとめています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/aafac8ca-8ec9-4411-ba7a-3385b0185d7f.png)


# 1. 簡単にパイプラインが構築できる！

LCELの `|` 記法は、LangChainの `Runnable` を活用して実装されています。
Runnable の特徴として以下が挙げられます。

- `invoke()`: 同期実行
- `ainvoke()`: 非同期実行（asyncio との親和性が高い）
- `batch()`: バッチ処理が可能（複数のプロンプトを一度に処理）



```例1：パイプラインの構築.py
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

# プロンプトテンプレート
prompt = PromptTemplate.from_template("Translate this into French: {input}")

# LLMモデル
llm = ChatOpenAI(model="gpt-4")

# LCELの `|` 記法を使ってパイプラインを構築
chain = prompt | llm

# 実行
output = chain.invoke({"input": "Hello, how are you?"})
print(output)

```
このコードでは、`prompt | llm` のように `|` を使うことで、
**プロンプトテンプレートの処理 → LLMの呼び出し** をシームレスにつなげています。


# 2. 並列処理で速度アップ！


LCELでは、`ainvoke()` を活用することで **複数の処理を並列実行** し、
大規模ワークフローの処理速度を最適化できます。
特に **LLMの呼び出しを並列化** することで、応答時間を大幅に短縮できるため、
リアルタイム処理が求められるアプリケーション（カスタマーサポート、データ解析など）で効果的です。

```例3：非同期処理の例.py
import asyncio
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

# プロンプトテンプレート
prompt = PromptTemplate.from_template("Provide a recipe for {dish} in Japanese.")

# LLMモデル
llm = ChatOpenAI(model="gpt-4")

# パイプラインを構築
chain = prompt | llm

# 非同期実行
async def async_task():
    result1 = chain.ainvoke({"dish": "カレー"})
    result2 = chain.ainvoke({"dish": "ラーメン"})
    results = await asyncio.gather(result1, result2)  # 並列処理
    print(results)

asyncio.run(async_task())

```



# 3. 知っておきたい「デバッグ」のポイント

LCELはシンプルな記法ながら、デバッグのためのツールも必要になります。
1でお伝えした `|` 記法を活用したパイプラインはシンプルに記述できる反面、エラーの特定が難しくなることがあります。

LCELは柔軟で強力ですが、**デバッグがしやすいコードを書いておくことが重要** です。



## エラー発生時のデバッグ方法
LCELでは、以下のようなエラーが発生することがあります。

| エラー | 原因 | 解決策 |
|--------|------|-------|
| `TypeError: 'str' object is not callable` | `Runnable` のメソッドを間違えて呼び出し | `invoke()` を適切に使用する |
| `KeyError: 'input'` | `invoke()` に渡す辞書のキーが間違っている | `print(input_data.keys())` で確認 |
| `asyncio.exceptions.TimeoutError` | LLM呼び出しがタイムアウト | `max_retries` を増やす・モデルの負荷を確認 |
| `AttributeError: 'NoneType' object has no attribute '...` | どこかで `None` が返っている | `print()` を挟んで `None` の発生箇所を特定 |

LCELに限らないことも多いですが、、、エラーが出たらの基本です。

1. エラーメッセージを読む
2. `print()` or `RunnableLambda` を挟んで途中のデータを確認
3. 非同期処理なら `try-except` でエラーハンドリング
4. 辞書のキーやデータ型が正しいか確認


以下、具体的な確認・対応方法を記述します。

##  3.1 `invoke()` でデータの流れを確認する。 
LCELのパイプラインは複数の `Runnable` をつなげるため、途中の処理がどうなっているのか確認しづらくなります。  
まずは、**個々のステップの出力を確認** してデータが正しく処理されているかをチェックしましょう。


```データの流れを個別に確認.py
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

# プロンプトとモデル
prompt = PromptTemplate.from_template("What is {topic}?")
llm = ChatOpenAI(model="gpt-4")

# LCELパイプライン
chain = prompt | llm

# 🔍 途中の処理を確認
test_input = {"topic": "quantum computing"}
print("Step 1: Prompt Output ->", prompt.invoke(test_input))
print("Step 2: LLM Output ->", chain.invoke(test_input))
```

- **`invoke()` を各ステップで個別に実行** し、データの流れをチェックする。
- LCELの `|` を使うと中間データが見えないので、**一部の処理だけ試す** ことでデバッグが容易になる。



## 3.2 `RunnableLambda` で途中のデータをログ出力する。

LangChainには `RunnableLambda` というユーティリティがあり、任意の処理を挟めます。  
**`RunnableLambda` を使ってデータの状態をログに出力** し、途中で変なデータになっていないか確認できます。


```データの流れをログ出力.py
from langchain_core.runnables import RunnableLambda

def log_data(x):
    print("🔍 Debug:", x)
    return x

debug_chain = RunnableLambda(log_data) | chain

# 実行
output = debug_chain.invoke({"topic": "AI ethics"})
print("Final Output:", output)
```

- `RunnableLambda` でデータを **ログに出力**
- 途中の処理がどうなっているかを確認しながらデバッグできる
- **エラー発生時のデータの状態** をチェックしやすくなる



## 3.3 `ainvoke()` のデバッグ（非同期処理）

`ainvoke()` を使っている場合、エラーメッセージが `asyncio` のスタックトレースに埋もれてしまい、**何が原因なのか分かりづらくなる** ことがあります。  
この場合、`try-except` を使って **エラーログを詳細に出力** すると、デバッグしやすくなります。


``` try-exceptでエラーハンドリング.py
import asyncio

async def debug_async():
    try:
        result1 = chain.ainvoke({"topic": "blockchain"})
        result2 = chain.ainvoke({"topic": "quantum physics"})
        results = await asyncio.gather(result1, result2)
        print("Results:", results)
    except Exception as e:
        print("🚨 Error occurred:", str(e))

asyncio.run(debug_async())
```

- `try-except` でエラーをキャッチし、原因を特定
- `asyncio.gather()` を使っている場合、複数の処理のどこでエラーが起きたかを把握しやすくする


## 3.4 `RunnableBranch` で条件分岐をデバッグ

LCELでは `RunnableBranch` を使うと、特定の条件に応じて異なる処理を実行できます。  
ただし、**条件分岐のロジックが間違っていると意図しない処理が実行される** ことがあるため、デバッグが必要です。


```RunnableBranchのデバッグ.py
from langchain_core.runnables import RunnableBranch

def is_math_topic(input_data):
    return "math" in input_data["topic"].lower()

# 条件分岐
branch_chain = RunnableBranch(
    (is_math_topic, prompt | llm),
    (lambda x: True, RunnableLambda(lambda x: "Not a math topic"))
)

# デバッグ実行
print(branch_chain.invoke({"topic": "math equations"}))  # LLMに渡される
print(branch_chain.invoke({"topic": "history"}))  # "Not a math topic" が返る
```


- **`RunnableBranch` の条件が意図通り動作しているかを確認**
- **条件関数 (`is_math_topic`) のロジックを細かくテスト** する
- **デフォルトの分岐 (`lambda x: True`) を設定しておくと、すべてのケースをカバーできる**


