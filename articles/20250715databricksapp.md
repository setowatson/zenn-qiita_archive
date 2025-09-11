---
title: "DatabricksでWebアプリを構築からデプロイまでしてみた"
emoji: "🌱"
type: "tech"
topics: ["databricks","アプリ開発","streamlit"]
published: true
publication_name: "headwaters"
---
# Databricksでアプリがデプロイできるようになたよ

今月からDatabricks Appが日本リージョンでも利用できるようになりました。

ーDatabricks Appとは？ー

> Databricks Appsは、データやAIを活用したアプリケーションをDatabricksプラットフォーム上で直接構築・デプロイできる新機能です。従来必要だった個別のインフラストラクチャ管理が不要となり、Unity CatalogやDatabricks SQLなどの既存サービスとシームレスに統合できます。PythonやNode.jsの主要フレームワークに対応し、インタラクティブなダッシュボードからRAGチャットアプリまで、幅広い内部ツールの開発が可能です。サーバレス環境でホストされるため、セキュリティやコンプライアンスの心配も軽減され、開発者はビジネスロジックに集中できます。

https://qiita.com/taka_yayoi/items/98028baf19e7a0f89daf


簡単に言うと、**「databricksだけでアプリの開発・デプロイができるようになったよ！」** ということですね。


# 社員の資格取得状況を見える化するダッシュボードアプリ

ということでさっそくアプリを作ってデプロイしてみました。
今回作成したのは、以下のような社内向けアプリです。

* **社員ごとの保有資格を一覧・フィルター表示**
* **部署別・月別・資格ランク別などでグラフ分析**
* **資格取得を申請できるフォーム機能**
* **資格マスタの登録や編集も可能**

フロントエンドには Streamlit を使い、
バックエンドはすべて Databricks の Delta Table で構築しました。
Cluade Code様のお力を借りながら3～4時間程度で完成しました。

![](https://storage.googleapis.com/zenn-user-upload/5c87edc4a7b1-20250716.png)
*タブ1：部署別の集計ダッシュボード*

![](https://storage.googleapis.com/zenn-user-upload/4b542cf54a35-20250716.png)
*タブ2：資格別の集計ダッシュボード*

![](https://storage.googleapis.com/zenn-user-upload/ce127c5fdc38-20250716.png)
*タブ3：月別の累計資格取得者数*

![](https://storage.googleapis.com/zenn-user-upload/69c093da5271-20250716.png)
*タブ4:資格取得一覧。編集や削除も可能。*

![](https://storage.googleapis.com/zenn-user-upload/3a821f67b80e-20250716.png)
*タブ5:資格取得時の申請（テーブル追加）*

![](https://storage.googleapis.com/zenn-user-upload/04e53b878fce-20250716.png)
*タブ6：対象資格の追加（テーブル追加）*

こんな感じです。
Appを試してみたかっただけなので、特別なUIやAI機能などは作成していません。
（機械学習モデルなんかを使うと、Databricksの強みがもっと出るかも）

アプリのURLを貼ろうか迷ったのですが、
まだ少し不具合があるのとセキュリティ対策を何もしていないので、
一旦スクショとコードのみの共有とさせてください。

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

また、Databricks環境は **Databricks Free Edition** を使用しています。
無料の範囲内でここまでいろんな検証ができるのは本当にありがたいです。

https://docs.databricks.com/aws/ja/getting-started/free-edition



# Databricksでアプリを開発するメリット


## 1. **Delta Tableをそのまま可視化・操作できる**

このアプリでは、事前にDatabricks上に保存されている以下のテーブルを活用しています。

* `sample`：資格取得の実績（社員・取得資格・取得日付）
* `certificate_master`：資格のランクマスタ
* `department_master`：部署マスタ

![](https://storage.googleapis.com/zenn-user-upload/6f7864906dc5-20250716.png)
*sampleテーブルはこんな感じ*


これらを直接`pandas.read_sql()`を使い読み込んで可視化に利用していますので、
**わざわざBIツールにデータ連携する必要がありません**。

また、このアプリでは新規追加、編集・削除などの機能がありますが、
Databricks上のDelta Tableに対して、直接実行しています。
つまり、以下のようなデータ操作が**追加のAPIやバッチ処理を用意せずに実現**できます。

* 表示（`SELECT`）：資格一覧や月別の集計グラフ表示
* 登録（`INSERT`）：新規資格申請・資格マスタ追加
* 更新（`UPDATE`）：資格情報の編集
* 削除（`DELETE`）：誤登録の資格削除

REST APIやバックエンドの複雑な実装が不要で、SQL一発で完結します。

## 2. デプロイもCLI操作もDatabricksでできる

Databricksでは、Streamlitアプリをノートブックではなく本番用アプリ（Databricks Apps）としてそのまま公開できます。

Databricks CLIの設定だけ最初に必要ですが、
`app.py` + `app.yaml` さえあれば、ワンクリック or CLIで即デプロイできました。

https://docs.databricks.com/aws/ja/dev-tools/cli/commands


つまり、**アプリ開発 → デプロイ → 共有 → 運用 まですべて1つの基盤で完結** します。
特に「社内向けアプリ」や「業務効率化ダッシュボード」を素早く回したいチームにとって、大きな武器になると思います。


![](https://storage.googleapis.com/zenn-user-upload/e29bbb0655b7-20250716.png)
*Databricksアプリのデプロイ画面*

CLIでコードをアップしたら、「デプロイ」ボタンを押すだけで公開できます。
個人的には `dbx sync`による同期がとても簡単で助かりました。

## 3. **権限・ログ・運用もDatabricks内で一貫管理できる**

* ワークスペースでの**Unity Catalogに基づくアクセス制御が可能**
* 操作ログ・データ変更ログも**監査しやすい**
* 「テーブル＋アプリ」が1つの環境にあることで、**ガバナンスと開発の両立**がしやすい

今回は個人開発ですし実運用も想定していないので考慮していないですが、
大規模でプロダクトレベルのアプリを開発する際、このあたりは必須の設定かなと思っています。
そうした設定が簡単なのは安心ですね。

![](https://storage.googleapis.com/zenn-user-upload/0dc02feb5319-20250716.png)
*「アプリ」→「権限」で設定できる画面*


# こんなケースにおすすめ

* **すでにDatabricksにデータがある**
  　→ Delta TableやSQL Warehouseをそのまま使いたい人に最適です。

* **部門ごとにアクセス制御したい／URLで簡単に共有したい**
  　→ Unity CatalogやDatabricks Appsと組み合わせれば、**セキュリティと利便性の両立**が可能です。

* **他システム連携ではなく、Databricks内で完結した仕組みを作りたい**
  　→ 外部APIの開発や疎通テストを省き、**シンプルかつ高速に運用開始**できます。

* **非エンジニアでもアイデアをアプリにしたい**
  　→ バックエンドの設定やデプロイシーンなど非エンジニアにとって難しい箇所の負担が軽減されるので、**誰でも気軽にアプリ開発に取り組むこと** ができます。


正直社内用途であれば、わざわざアプリ化しなくてもDatabricks上のダッシュボードで完結するよねーと最後に気づきました。
でもエンジニアでなくてもここまでできるぞ！ということで、
敷居の低い入門編としてとりあえず魅力を感じることはできました。
みなさんもぜひ気軽にチャレンジしてみてはいかがでしょうか？


## 参考リンク

https://www.databricks.com/jp/product/databricks-apps


https://docs.databricks.com/en/dev-tools/python-sql-connector.html


https://docs.databricks.com/aws/ja/dev-tools/databricks-apps/tutorial-streamlit
