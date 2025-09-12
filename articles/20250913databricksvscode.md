---
title: "VS Code × Databricks Bundleでノートブックをデプロイしてみた"
emoji: "🌱"
type: "tech"
topics: ["Databricks", "VSCode", "Bundle", "CLI"]
published: false
---

# Asset Bundleとは

Databricks の **Asset Bundle (DABs)** は、
**「Databricks 上のリソース（ノートブック・ジョブ・パイプラインなど）を
コードで一括管理し、デプロイできる仕組み」** です。

従来の手作業や Notebook ベースの管理に比べて、
- Git でのバージョン管理  
- CI/CD パイプラインでの自動デプロイ  
- 複数環境（dev/stg/prod）の切り替え  

といったソフトウェアエンジニアリングのベストプラクティスを取り込みやすくなっています。

今回は VS Code と組み合わせて、ローカルから Databricks ワークスペースに
ノートブックをアップロードする流れを試してみます。


https://learn.microsoft.com/ja-jp/azure/databricks/dev-tools/bundles/



## と言いつつ、最初に言い訳。

Asset Bundle 自体は、2023年ごろに CLI ベースの機能としてプレビュー提供され、  
当初は「ローカルマシンから `databricks bundle deploy` を実行する」使い方が中心でした。  

しかし 2025年5月のアップデート により、  
**Databricks ワークスペース上でも Asset Bundle を直接活用できるように進化**しました。  
これにより、ワークスペース上での共同編集と IaC 的なリソース管理を両立できるようになっています。

本来このタイミングで記事を書くなら、新しいほうの機能を取り扱うべきでした。
（記事を最後まで書ききった後、気づきました）

次回以降に書くとして、今回は「ローカル環境から CLI で実行する流れ」を紹介します。

https://learn.microsoft.com/ja-jp/azure/databricks/dev-tools/bundles/workspace-tutorial

（ただ、まだプレビュー機能のようです）

## 環境
- VS Code
- Databricks 拡張機能 (Marketplace からインストール)
- Databricks CLI v0.221.0 以降
- Azure Databricks ワークスペース

# 手順

## 1. VS Code に Databricks 拡張を導入
拡張を入れると、サイドバーに「Databricks」アイコンが出ます。  
最初はこのように「プロジェクトが検出されない」状態からスタートします。

![](/images/databricksdabs1.png =500x)



## 2. ワークスペースと認証を設定
`Create configuration` を押すと、ワークスペースの URL と認証方式を選ぶ画面が出ます。  

おすすめは **OAuth**。
ブラウザが立ち上がり、Azure AD (Entra ID) でサインインするだけで使えるようになります。



## 3. databricks.yaml を作成
プロジェクトルートに `databricks.yaml` を用意します。  

```yaml
bundle:
  name: "任意の名前"
  target: dev

targets:
  dev:
    workspace:
      host: https://adb-xxxxxxxx.azuredatabricks.net
      root_path: /Users/<ユーザー名>/<フォルダ名>

artifacts:
  notebooks:
    type: files
    path: notebooks
    files:
      - source: notebooks/hello.py
```

ポイント:

* `targets.dev.workspace` にワークスペース URL と配置先パスを指定
* `artifacts` でアップロードするファイルを定義
* `files` の下は `source:` で相対パスを書く必要がある


## 4. ディレクトリ構成

ローカルの構成はこんな感じ。

```
📂 プロジェクトルート
 ├─ databricks.yaml
 ├─ README.md
 └─ notebooks/
      └─ hello.py
```


## 5. デプロイ

準備ができたら以下を実行します。

```bash
databricks bundle deploy --profile <プロファイル名>
```

実際の出力:

```
Uploading notebooks/hello.py...
Uploading bundle files to /Workspace/Users/.../0912-vscode-test/files...
Deploying resources...
Deployment complete!
```

## 6. Databricks ワークスペースで確認


ワークスペースに `hello.py` がアップロードされているのを確認できました 🎉

![](databricksdabs2.png)

# Repoとどう違うんだっけ？

ここまで触っておいて、Repoの機能と似ている気がするけど、どうなんだっけ？と思い調べてみました。

結論、Databricks の Repo機能とBundleは似ているようで結構違います。

「どっちでもいい」わけではなくて、用途が分かれている感じです。



## Repo 機能

### 特徴

* **GitHub / Azure DevOps / GitLab と直接連携**
* Databricks Workspace 上に「Repo」としてチェックアウトされる
* Notebook を編集すると **Git にコミット/プッシュ**できる
* 基本的に「開発者が Databricks 上で直接コードを書いてバージョン管理する」スタイル

https://learn.microsoft.com/ja-jp/azure/databricks/repos/git-operations-with-repos

### 向いているケース

* チームで共同開発する場合
* Databricks ノートブックを直接編集したい場合
* 「GitOps」よりも「普段のGitフロー」に近い運用をしたい場合


## Databricks Bundle

### 特徴

* `databricks.yaml` に **デプロイ対象（artifacts, jobs, pipelines, clusters...）を定義**
* `databricks bundle deploy` で **ローカルから Databricks 環境に反映**
* IaC (Infrastructure as Code) 的に使える
* CI/CD パイプラインで自動デプロイしやすい

### 向いているケース

* 複数環境（dev/stg/prod）を同じコードで管理したい
* Job, Pipeline, Unity Catalog オブジェクトも含めて「環境構成ごと再現」したい
* Databricks を IaC 的に管理したい




# まとめ

- VS Code の拡張と CLI を組み合わせることで、ローカルのファイルを簡単に Databricks にデプロイできる  
- `databricks.yaml` の書き方に少し戸惑うかも。 `artifacts.files` 周りは最初にハマりやすいポイント 
- 一度仕組みを作れば、Job や Pipeline などのリソースも同じく `bundle deploy` で IaC 的に管理できる


といったことが体感できました。  

また、Asset Bundle は今後ワークスペース上でも活用できる方向に進化しており、
「ローカルからのデプロイ」だけでなく「Databricks 上で共同編集しながら IaC 管理する」といった使い方も広がりそうです。  

以上、VS Code × Databricks Bundle のハンズオンでした。