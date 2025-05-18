---
title: "MCPで実現できる手軽なメモ帳 with Claude"
emoji: "🌱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["MCP", "shadcn", "ai", "notion"]
published: false
---

# TL;DR

ClaudeにMarkItDown-MCPを取り入れることで、PDF、PowerPoint、Word文書などのテキスト抽出を効率よく実行できます！


# MarkItDownとは？


MarkItDownは、**Microsoftが開発したPythonベースのツール** で、多様な形式のファイルをMarkdown形式に変換する機能を提供しています。
対応フォーマットには以下が含まれます。

- ドキュメント: PDF、Word、PowerPoint、Excel
- メディア: 画像（OCR機能付き）、音声（文字起こし機能）
- Web: HTML、YouTubeのURL
- アーカイブ: ZIPファイル内のコンテンツ
- 電子書籍: EPUB


![markitdown1](/images/markitdown1.png)


通常はPythonライブラリの1つとしてコードの中で実行されますが、MCP連携によりコードを書かなくても実行できるようになりました。

ちなみにMCPを使わない通常のMarkItDownの使い方はこちらの記事がわかりやすかったです。

https://note.com/robothink/n/n3e41df09d174

## markitdown-mcpのgithub

READMEに導入方法が記載されています。

https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp


# MarkItDown-MCPでできること

MarkItDownをMCP連携したNotionから操作するイメージを、先にご紹介します。

## 処理対象ファイル

IPAが公式に共有している以下のpptxをマークダウン形式に変換します。


![image.png](/images/markiddown2.png)


引用：

https://www.ipa.go.jp/security/sme/f55m8k0000001wpz-att/outline_guidance_incident.pdf


## Claude Desktopでの操作

```md
/md "対象ファイルのパス名"
```
と入力するだけです。
今回はローカルDocumentsフォルダに保存したファイルパスを入力しています。

![image.png](/images/markitdown2.avif)


## 処理結果

上記ファイルの読み取り結果をマークダウン形式で返してくれています。

```

I'll help you analyze the PDF document. Let me convert it to a more readable format first.

セキュリティインシデント対応の手引きの内容を日本語で要約いたします。

このドキュメントは「中小企業のためのセキュリティインシデント対応の手引き」で、2024年7月に独立行政法人情報処理推進機構(IPA)のセキュリティセンターによって発行されたものです。

このガイドは以下の3つの主要なステップでインシデント対応を説明しています：

## ステップ1: 検知・初動対応
このステップでは以下の内容が含まれています：
- 検知と連絡受付：インシデントの兆候や発生を発見した場合は情報セキュリティ責任者に報告
- 対応体制の構築：情報セキュリティ責任者は経営者に報告し、対応のための体制を立ち上げる
- 初動対応：外部からのアクセスを遮断、情報や機器の隔離、システムやサービスの停止など

## ステップ2: 報告・公表
このステップでは：
- 第一報：状況をウェブサイトやメディアを通じて公表し、必要に応じて問い合わせ窓口を開設
- 第二報以降・最終報：被害者や影響を受けた取引先に対応状況や再発防止策を報告し、必要に応じて関係機関に届け出る

## ステップ3: 復旧・再発防止
このステップには次の内容が含まれます：
- 調査・対応：5W1Hの観点で状況を調査し、必要な修復を実施
- 証拠保全：訴訟対応などのために証拠を保全
- 復旧：修復確認後、停止したシステムやサービスを復旧
- 再発防止：根本原因を分析し、新たな技術的対策や運用改善などを実施

ガイドラインには以下の事象ごとの対応ポイントも含まれています：
- ウイルス感染・ランサムウェア感染の場合：感染機器の使用停止、ネットワークからの分離、バックアップの重要性
- 情報漏えいの場合：不正アクセス、内部犯行、誤送信、誤公開、紛失・置忘れなどへの対応
- システム停止の場合：様々な原因に対する対応と事業継続計画（BCP）の必要性

また、インシデント発生時に整理すべき事項のリストや、相談窓口・報告先の情報も提供されています：
- IPA（情報処理推進機構）：技術的相談、ウイルス・不正アクセスの届出
- 警察のサイバー犯罪相談窓口
- JPCERT/CC（一般社団法人JPCERTコーディネーションセンター）：インシデント対応依頼
- 個人情報保護委員会：個人情報・マイナンバー漏えいの報告

さらに、インシデント対応に役立つ参考情報として、「サイバー攻撃被害に係る情報の共有・公表ガイダンス」、「インシデントハンドリングマニュアル」、「ランサムウェア対策特設ページ」が紹介されており、中小企業の情報セキュリティ対策ガイドラインも参照されています。

このガイドラインは中小企業がセキュリティインシデントに適切に対応し、被害を最小限に抑え、事業継続を確保するための実践的なフレームワークを提供しています。

```


以下例としてステップ１のスライドですが、
インデントの項目も上手く要約できていることがわかります。

![image.png](/images/markitdown4.avif)


めちゃくちゃ便利。すごい。


# Claude連携のメリット

上のようにMarkItDownはMCP（Model Context Protocol）サーバー機能を提供しており、Claude Desktopと直接連携できます。
この連携によって、以下のようなメリットが考えられます。

1. **シームレスなファイル処理**: 複雑なフォーマットのファイルを直接Claudeに理解させることができる
2. **コンテキストの維持**: 文書の構造や書式情報を保持したまま、AIに情報を伝えられる
3. **バッチ処理の効率化**: 複数のファイルをまとめて変換し、一括で分析できる
4. **マルチモーダル情報の統合**: 画像内のテキスト、音声の文字起こしなど、異なる形式の情報を統合できる


# MarkitDownの使用方法・Claudeとの連携方法

以下連携方法を記載していますが、基本はGithubリポジトリのREADMEに書いてある通りとなります。

## 1. インストール方法

まず、MarkItDownとそのMCP連携機能をローカル環境にインストールする必要があります。
以下をターミナルなどで実行します。

```bash
# リポジトリをクローン
git clone https://github.com/microsoft/markitdown.git
cd markitdown

# インストール (すべての機能を含む)
pip install -e 'packages/markitdown[all]'

# MCPパッケージも別途インストール
pip install -e 'packages/markitdown-mcp'
```

## 2. Claude Desktopの設定

次にClaude Desktopの設定操作です。
Claude Desktopでmarkitdown-mcpを使用するには、`claude_desktop_config.json`ファイルを設定する必要があります。 
このファイルの場所はClaude Desktopのインストール場所によって異なりますが、基本は`「設定」＞「開発者」＞「構成を編集」`から見つけることができると思います。

設定ファイルの例

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

特定のディレクトリをマウントする場合は以下のように設定します。

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-v",
        "/home/user/data:/workdir",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

### 3. 使用方法

上記で設定は完了です。
連携に問題がなければ、サーバーが以下のMCPコマンドに応答します。

- `/md <fileのパス>` → 指定されたファイルをMarkdownに変換

例えば、「このPDF文書の主要なポイントを3つにまとめて」「このプレゼン資料の具体的な提案内容を要約して」などのプロンプト指示により、MarkItDownの機能を使って出力を返してくれます。

# まとめ

MarkItDownとClaude Desktopの連携は、”単なるファイル変換ツール”以上の価値を提供しています。
設定さえ完了すればLLMアプリ上でチャットするだけで変換できるので。

また今回はここまで試していませんが、MarkItDownはプラグイン機能も備えているので、独自の変換機能も追加できます。これにより、業界特有のファイル形式や社内独自フォーマットにも対応することも可能です。

MarkItDownはAIの力を最大限に活用するため、今後のデジタルワークフローに欠かせない存在になりえます。皆さんも、ぜひMarkItDownとClaude Desktopの連携で、新たな業務効率化の可能性を探ってみてはいかがでしょうか。


## 参考リンク

- [MarkItDown GitHub リポジトリ](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp)
- [MarkItDown 公式ドキュメント](https://github.com/microsoft/markitdown/blob/main/README.md)
- [Claude Desktop 公式サイト](https://claude.ai/desktop)
