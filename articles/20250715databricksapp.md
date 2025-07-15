---
title: "Databricksでアプリをデプロイするメリットを考える"
emoji: "🌱"
type: "tech"
topics: ["databricks","アプリ開発","streamlit"]
published: false
publication_name: "headwaters"
---


# Databricksでアプリがデプロイできるようになりました

今月Databricks Appが日本でも利用できるようになりました。

ーDatabricks Appとは？ー

> Databricks Appsは、データやAIを活用したアプリケーションをDatabricksプラットフォーム上で直接構築・デプロイできる新機能です。従来必要だった個別のインフラストラクチャ管理が不要となり、Unity CatalogやDatabricks SQLなどの既存サービスとシームレスに統合できます。PythonやNode.jsの主要フレームワークに対応し、インタラクティブなダッシュボードからRAGチャットアプリまで、幅広い内部ツールの開発が可能です。サーバレス環境でホストされるため、セキュリティやコンプライアンスの心配も軽減され、開発者はビジネスロジックに集中できます。

https://qiita.com/taka_yayoi/items/98028baf19e7a0f89daf




簡単に言うと、「databricksだけでWebアプリの開発・デプロイができるようになったよ！」ということですね。


# アプリ：社員の資格取得状況を見える化するダッシュボード

ということでさっそくアプリを開発してみました。
今回作成したのは、以下のような社内向けアプリです。

* **社員ごとの保有資格を一覧・フィルター表示**
* **部署別・月別・資格ランク別などでグラフ分析**
* **資格取得を申請できるフォーム機能**
* **資格マスタの登録や編集も可能**

フロントエンドには Streamlit を使い、
バックエンドはすべて Databricks の Delta Table で構築しました。
開発所要時間は1人で3～4時間程度です。


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/4332ee23-aa9c-4f51-8194-8549130b3831.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/47cb4e9b-97a6-4849-8485-7ff03efbc3fc.png)


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/fde79843-9ba3-4ad7-8337-be210b094d89.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/a5a6f689-dc43-4a73-9b5a-dc04a639855e.png)


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/8f639216-c837-4b9f-a1a4-9cf5fabb7680.png)



![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/216b5abe-eee3-4a13-950c-8c99d5f16f9c.png)

こんな感じです。
Appを試してみたかっただけなので、特別AIを組み込んだ機能などは作成していません。

また、アプリのURLかGithubリポジトリを貼ろうか迷ったのですが、
まだ少し不具合があるのとセキュリティ対策を何もしていないので一旦スクショのみの共有とさせてください。

## アプリ構成


```markdown
SHIKAKU-MIERUKA/
├── .databricks/                  # Databricks CLI関連（任意）
├── .streamlit/                   # Streamlit設定フォルダ
│   └── secrets.toml              # Databricks SQL接続情報（※Git管理NG）
├── shikaku-mieruka2.git/         # Git設定情報（VSCode/Git拡張由来のフォルダ）
├── .gitignore                    # Gitで無視するファイル定義
├── app.py                        # Streamlitアプリ本体
├── app.yaml                      # Databricks App用の構成設定ファイル
├── README.md                     # 説明書き（使い方・概要など）
└── requirements.txt             # 使用ライブラリ一覧（pip install用）
```

| ファイル・フォルダ                 | 説明                                                     |
| ------------------------- | ------------------------------------------------------ |
| `.streamlit/secrets.toml` | Databricks SQL Connectorの認証情報。Git管理しない。       |
| `app.py`                  | Streamlitのメインスクリプト。UI・SQL実行・グラフ表示などを書く。 |
| `app.yaml`                | Databricks Appsでのアプリ起動時の定義ファイル（v2対応）   |
| `requirements.txt`        | `pip install -r requirements.txt` で依存パッケージを一括インストール |
| `.gitignore`              | `secrets.toml`や`.venv/`など、Gitに含めたくないものを除外    |
| `README.md`               | プロジェクトの概要・起動方法を記載するドキュメント          |

## 開発環境

このアプリは、VSCode上でPythonを使って開発しています。

- 使用している主なツール・ライブラリ
    - Python 3.9 以上
    　StreamlitやDatabricks Connectorが動作する環境

    - Streamlit 1.35+
　PythonだけでWebアプリを作れるライブラリ

    - Databricks SQL Connector for Python
　Databricksのテーブルに対して、PythonからSQLを実行するための公式ライブラリ

    - Plotly
    　棒グラフや折れ線グラフを描くのに使用



またDatabricks Free Editionを使用しています。
無料の範囲内でここまでの検証ができるのは本当にありがたい。


# Databricksでアプリを開発するメリット


## 1. **Delta Tableをそのまま可視化・操作できる**

このアプリでは、事前にDatabricks上に保存されている以下のテーブルを活用しています。

* `sample`：資格取得の実績（社員・取得資格・取得日付）
* `certificate_master`：資格のランクマスタ
* `department_master`：部署マスタ

sampleテーブルはこんな感じ。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/cebe576b-651a-471d-8b4f-de5266251175.png)


これらを直接`pandas.read_sql()`で読み込んで可視化に利用していますので、
**わざわざBIツールにデータ連携する必要がありません**。

また、このアプリではDatabricks上のDelta Tableに対して、
**直接 `SELECT`・`INSERT`・`UPDATE`・`DELETE` を実行**しています。
つまり、以下のようなデータ操作が**追加のAPIやバッチ処理を用意せずに実現**できます。

* 表示（`SELECT`）：資格一覧や月別の集計グラフ表示
* 登録（`INSERT`）：新規資格申請・資格マスタ追加
* 更新（`UPDATE`）：資格情報の編集
* 削除（`DELETE`）：誤登録の資格削除

REST APIやバックエンドの複雑な実装が不要で、SQL一発で完結します。

## 2. デプロイもCLI操作もDatabricksで一気通貫できる
Databricks では、Streamlitアプリをノートブックではなく本番用アプリ（Databricks Apps）としてそのまま公開できます。

Databricks CLIの設定だけ最初に必要ですが、それさえすれば、
app.py + app.yaml さえあれば、ワンクリック or CLIで即デプロイできました。

https://docs.databricks.com/aws/ja/dev-tools/cli/commands


つまり、アプリ開発 → デプロイ → 共有 → 運用まですべて1つの基盤で完結します。
特に「社内向けアプリ」や「業務効率化ダッシュボード」を素早く回したいチームにとって、大きな武器になると思います。

以下はDatabricksアプリのデプロイ画面です。
CLIでコードをアップしたら、「デプロイ」ボタンを押すだけで公開できます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/2bf1fdda-841a-4458-a9c0-ce14af869ebd.png)


## 3. **権限・ログ・運用もDatabricks内で一貫管理**

* ワークスペースでの**Unity Catalogに基づくアクセス制御が可能**
* 操作ログ・データ変更ログも**監査しやすい**
* 「テーブル＋アプリ」が1つの環境にあることで、**ガバナンスと開発の両立**がしやすい

今回は私個人で行い、実運用も想定していないのであまり考慮していないですが、
大規模でプロダクトレベルのアプリを開発する際、このあたりは必須の環境かなと思っています。
そこも簡単なのは安心ですね。

以下「権限」から設定できる画面。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/80b4bb9a-0103-4e5e-b86a-c3e4ec192619.png)


# こんなケースにおすすめ

* **すでにDatabricksにデータがある**
  　→ Delta TableやSQL Warehouseをそのまま使いたい人に最適です。

* **ExcelやBIツールでは限界を感じている**
  　→ 単なる可視化ではなく、申請や登録など、**業務フローそのものを操作できるアプリ**を作りたい場合にぴったり。

* **部門ごとにアクセス制御したい／URLで簡単に共有したい**
  　→ Unity CatalogやDatabricks Appsと組み合わせれば、**セキュリティと利便性の両立**が可能です。

* **「他システム連携」ではなく、Databricks内で完結した仕組みを作りたい**
  　→ 外部APIの開発や疎通テストを省き、**シンプルかつ高速に運用開始**できます。




今回のプチ開発でdatabricksでのアプリ開発にとても大きな魅力を感じました。上に当てはまる方はぜひ気軽にチャレンジしてみられてはいかがかでしょうか。


## 参考リンク

https://www.databricks.com/jp/product/databricks-apps


https://docs.databricks.com/en/dev-tools/python-sql-connector.html


https://docs.databricks.com/aws/ja/dev-tools/databricks-apps/tutorial-streamlit
