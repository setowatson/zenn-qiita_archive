---
title: "databricks"
emoji: "🌱"
type: "tech"
topics: ["databricks","n8n"]
published: false
publication_name: "headwaters"
---

# 1. はじめに
	•	Databricks × n8n を組み合わせる背景
	•	既存のSaaS連携ツール（Zapier, Power Automate等）と何が違うのか
	•	「AIエージェント時代」に求められる自動化基盤とは


# 2. それぞれの特徴

## Databricksの強み
	•	データとAIを統合的に扱えるLakehouse基盤
	•	SQL, ML, Streaming, BIまで一元化できる
	•	Unity Catalogによるガバナンス／セキュリティ



## n8nの特徴
	•	ローコードでAPIやサービスをつなげられる自動化ツール
	•	Webhook／HTTP Requestを使った柔軟な拡張性
	•	Databricks Appsとしてセルフホストできる利点（データと近い場所に置ける）


## Databricks × n8nでできること
	•	データ活用API化：Databricks SQLをWebhook経由で即席APIに
	•	AI推論サービス化：MLflow/Model Servingを呼び出してAIエージェントを実装
	•	通知／アラート：ジョブ完了をTeams/メールに自動通知
	•	PoC支援：ノーコードで業務部門がAI活用を試せる

# 3. AIエージェント的な利用イメージ
	•	Root Agentが入力を解釈 → Databricksをツールとして呼び出す
	•	例：
	•	「東京の売上は？」→ Databricks SQLクエリに変換
	•	「この顧客の解約リスクは？」→ Databricksモデルを呼び出し
	•	Chat UI（Teams/Slack）との連携で自然な利用体験


## メリットと注意点
	•	メリット：データの近くで自動化、セキュリティ一元化、PoC容易
	•	注意点：運用時はDB/Redisが必要、Teams連携は権限設計に注意


# 4. 実際の操作
	•	コーヒー販売データをDatabricksに保存
	•	n8n Webhookで期間・店舗を入力 → Databricks集計 → Teamsに返答
	•	「AI Agent的に使える」ことが一目で伝わるサンプル


# 5. まとめと展望
	•	Databricks Appsとn8nで「AIエージェント基盤」が身近に
	•	今後はエンタープライズAI（RAGやマルチエージェント）との組み合わせが期待される
	•	「データ基盤＋自動化＋AI」＝業務変革のコア になる

