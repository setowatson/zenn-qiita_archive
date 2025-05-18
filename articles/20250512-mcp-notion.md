---
title: "MCPで実現できる手軽なメモ帳 with Claude"
emoji: "🌱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["MCP", "ai", "notion"]
published: false
publication_name: "headwaters"
---


# MCPで実現できるAIアシスタント付き学びメモ

今話題のMCPですが、先日NotionからClaude DesktopのようなMCPクライアントからNotionのページやデータベースを操作できる機能が展開されました。

https://github.com/makenotion/notion-mcp-server

この機能を活用することで、以下のような作業が可能になります。

- Claudeの支援を受けながらNotionコンテンツを作成・編集する
- Notionのデータベースやページ内容をClaudeに分析させる
- 複数のNotionページやデータベースの情報を統合して分析する
- 定型業務を自動化する

本記事では、この連携から具体的にできることの一例 
**「AIの力を借りながら、Notionをメモ帳・学びポートフォリオとして活用する例」** 
を紹介します。
「MCPを活用してみたい！」という方にもおすすめです。
（「MCPとは？」という記事は既存でたくさんあるため省略します。）

## 作成したNotionワークスペースはこちら

https://rogue-boat-75a.notion.site/1d2cf2b4168680a8b96ec0d5f7933225?pvs=4

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/1b852c8c-f57d-4de2-9a60-5b51b31aa120.png)

このワークスペースは、AIと技術に関する情報を収集・整理・発信するための個人的な知識管理システムです。日々の学びや考えを記録・体系化し、将来の参照や知識共有に活用することを目指しています。

基本的にNotionはノータッチで、すべてClaude Desktopへのチャットベース操作で記事や文章の作成・編集を行います。

# 主要機能

各機能は対応するページ（「今日のAIニュース」「学びメモ」「ブログ投稿」）と連携しており、情報を体系的に整理・管理することができます。これらの機能を活用することで、AI関連の情報収集、知識の整理、コンテンツ作成のワークフローを効率化できます。
## ① AIニュースのピックアップ・要約作成

最新のAI関連ニュースを収集して、日付、タイトル、内容、ソース、カテゴリの情報を含めてデータベースに追加します。
カテゴリは「研究」「ビジネス」「製品」「倫理」「その他」から選択されます。

使用例：
「今日のAIトレンドニュースを教えて」
「AIの研究に関する最新情報をニュースに入れておいて」

https://rogue-boat-75a.notion.site/AI-1d2cf2b4168681e3899ae8a02ee6ea4c


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/3469a506-235f-4431-bdf3-fdbf78aa9014.png)


## ② Q&A形式の学びメモ作成

特定のトピックについての知識やQ&Aを「学びメモ」に追加できます。
質問、回答、カテゴリ、タグ、学習日などの情報が整理されて保存されます。

使用例：
「OpenAIの最新モデルについて教えて」
「MCPに関する内容をQ&A形式で追加して」

https://rogue-boat-75a.notion.site/1d2cf2b4168680248423f961eb3aceaa

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/2bc122a0-6e4a-42df-b299-dd000811583c.png)


## ③ ブログ記事投稿アシスタント

この機能を使うと、ブログ記事の作成やアイデア管理ができます。
「」と指示すると、ブログ記事の下書きが作成されます。
「量子コンピューティングに関するテーマをブログ投稿アイデアに追加して」と指示すると、アイデアメモとしてデータベースに追加されます。

使用例：
「AIとビジネスの関係についてブログを書きたい」
「MCPに関するテーマをブログ投稿アイデアに追加して」
「アイデアにある内容からブログを作成して」

https://rogue-boat-75a.notion.site/1d2cf2b4168680e5a01cf69ac333ad1c

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/5c05f92f-ba10-4cb2-962c-ac5695752120.png)



# 使用手順

## 1.Claude Desktopに主要３機能いずれかのプロンプトを入力

- 主要機能①「今日のAIニュースを教えて」
→「今日のAIニュース」ページにニュース記事を追加する。

- 主要機能②「〇〇について教えて」「〇〇に関する内容をまとめて」
→ 学びメモにQ&A形式で追加する。

- 主要機能③「〇〇の内容でブログを書きたい」「〇〇に関するテーマをブログ投稿アイデアに追加して」
→ 下書きを作成する。アイデアを保管しておく。



## 2.Notionで作成を確認する。

Notionの該当ページへの反映を確認します。
例えば、ブログ記事管理はこんなふうに反映されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/21a2446d-180d-433d-9fee-5dadfe72ace6.png)


## 3. 学びの復習・他者への共有・ブログ記事の下書きリストとして活用する。

作って満足しがちな性格ですが、ここを活用するのが大事。
Notionならスマホでもどこでもアクセスできるので、しっかり学びポートフォリオとして活用したいですね。

## Claude Desktopのチャットイメージ

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/976199b6-6b72-451c-98c6-157971c87d42.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/9e44e764-01ad-4d47-a19f-08e58374b82b.png)





# このページの作成方法


## 0.事前準備

- Notionアカウントの作成（ある程度の概要・操作知識）
- Claudeアカウントの作成
- Claude Desktopのインストール

すべて課金の必要ありません。

## 1. Notionインテグレーションの作成・トークン取得

https://www.notion.so/profile/integrations

1. Notionの上記ページへアクセス
1. 「+ New integration」ボタンをクリック
1. 以下の情報を入力
    - Name: 任意の名前
    - Logo: 任意
    - Associated workspace: 連携したいワークスペースを選択
    - Type：Internal
    - Capabilities: 必要な権限を選択（ページの読み書き、コメント追加など）

作成後、「Internal Integration Token」が表示されます。
このトークンは機密情報ですので流出しないように気をつけてください。
トークンは後からでも参照できるのでメモする必要はありません。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/fdf38bac-8c3e-4222-abf6-3735e26fb3ae.png)

## 2.対象Notionページにインテグレーションを追加

1. 右上の「…」をクリック
1. 「接続」というコマンドにカーソルを合わせると「コネクトを探す」
1. 先ほど作ったインテグレーション(MCPserver)を選択

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/b34a117c-9408-49a8-8b3f-a1bc24a55e17.png)

## 3.Claude DeskTopにトークンを連携

1. Claude Desktopの「設定」を開く
1. 「開発者」から「構成を編集」
1. 「claude_desktop_config.json」ファイルを開く
1. 以下コードに編集

「トークンを挿入」の部分に先ほどのトークンをペーストしてください。
```claude_desktop_config.json
{
  "mcpServers": {
    "notionApi": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer トークンを挿入\", \"Notion-Version\": \"2022-06-28\" }"
      }
    }
  }
}

```

最後にファイルを保存してClaude Desktopを再起動。
「running」になっていることを確認。


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/04efcfaa-7f2c-40bb-b90e-5663c9b1c3e6.png)


完了！
これでNotionとClaudeがMCPによって連携されました。

# セキュリティと注意点

MCP連携は強力ですが、以下の点に注意が必要です：

- アクセス権限の適切な設定 - Claudeに必要最小限のアクセス権限のみを付与することをお勧めします
- 機密情報の取り扱い - 極めて機密性の高い情報はMCP連携の対象から除外することを検討しましょう
- 変更の確認 - Claudeによる変更は必ず人間が確認するプロセスを設けると安心です
- 監査ログの確認 - 定期的にアクセスログを確認し、想定外の操作がないか確認しましょう


# アイデアはたくさん！あなただけのClaude×Notionアイデアを！

今回NotionとClaude DesktopのMCP連携によって、学びメモを作成しました。
Claudeのチャットで簡単に学びを追加・アクセス・共有できるため、学習のモチベーションも上がりそうです。（学習としてのありかたはさておき）

ほかにもClaudeにアイデアを吹き込むことでさまざまなNotionAI活用アイデアがあると思います。
ぜひ試行錯誤しながらMCPブームに乗って、日常生活を豊かにしましょう。


