---
title: "DatabricksにおけるLiquid Clustering徹底解説"
emoji: "💧"
type: "tech"
topics: ["Databricks", "DeltaLake", "LiquidClustering", "データ基盤"]
published: false
publication_name: "headwaters"
---

# Liquid Clusteringとは？

Databricksの **Liquid Clustering** は、Delta Lake テーブルのデータを効率的に並び替える仕組みです。  
従来の「パーティション」や「Z-ORDER」に代わる **次世代のデータ配置最適化機能** として提供されています。  

ポイントは以下の3つです：  
- 物理パーティションを意識せずに、任意の列でデータをクラスタリングできる  
- データ更新や追加に応じて **自動でクラスタリングを維持**  
- Z-ORDER のように手動で `OPTIMIZE` を回さなくてもよい  

---

# 従来のパーティションの限界

従来の `PARTITIONED BY` は、指定した列ごとにディレクトリを分割してデータを配置する仕組みでした。  

```sql
CREATE TABLE sales_partitioned (
  order_id STRING,
  order_date DATE,
  customer_id STRING,
  amount DECIMAL(10,2)
)
USING delta
PARTITIONED BY (year(order_date), month(order_date));
````

* **メリット**

  * `WHERE year = 2025` のようなクエリでは劇的にスキャン量を削減できる

* **デメリット**

  * パーティション数が増えるとメタデータ管理コストが急増
  * 「日付＋ID」など複数条件のフィルタには弱い
  * 更新が多いテーブルでは再分割コストが発生

このため Databricks では、**大規模データや更新頻度が高いケースでは非推奨に近い扱い**となってきています。

---

# Liquid Clusteringの登場

Liquid Clusteringでは、物理パーティションではなく **データの物理的な並び順** によって効率化を実現します。

```sql
CREATE TABLE sales_liquid_clustered (
  order_id STRING,
  order_date DATE,
  customer_id STRING,
  amount DECIMAL(10,2)
)
USING delta
CLUSTER BY (order_date, customer_id);
```

* `order_date` や `customer_id` でクエリを絞ると効率的にスキャンできる
* 更新・追加があってもクラスタリングが自動的に保たれる
* 複合キーでの検索に強い

---

# ユースケース

## ✅ 1. 時系列データ

* IoTログ、アクセスログ、金融取引履歴
* 「日付＋ユーザーID」「日付＋地域」で頻繁にフィルタ → Liquid Clusteringで効果大

## ✅ 2. 更新が多いデータ

* ECサイト注文履歴、在庫データ
* MERGEやアップサートが多い場合でもクラスタリングが維持される

## ✅ 3. 多次元検索

* 顧客行動分析（顧客ID × 商品カテゴリ × 日付）
* パーティションでは難しかった複合条件に対応

## ✅ 4. 小ファイル問題の回避

* ストリーミング取り込みやマイクロバッチで大量の小ファイルが発生
* 自動クラスタリングで断片化を抑制

---

# 活用方法

## テーブル作成時に指定

```sql
CREATE TABLE transactions (
  id STRING,
  user_id STRING,
  region STRING,
  tx_date DATE,
  amount DOUBLE
)
USING delta
CLUSTER BY (tx_date, user_id);
```

## 既存テーブルに後付け

```sql
ALTER TABLE transactions
SET TBLPROPERTIES (
  'delta.liquidClustered.columns' = 'tx_date, user_id'
);
```

## パーティションとの組み合わせ

```sql
CREATE TABLE web_logs (
  log_id STRING,
  event_time TIMESTAMP,
  user_id STRING,
  region STRING
)
USING delta
PARTITIONED BY (year(event_time))
CLUSTER BY (event_time, user_id);
```

大規模データでは「粗いパーティション × Liquid Clustering」のハイブリッドが有効です。

---

# まとめ

* **パーティション**

  * 少数の粗い区切り（例：年単位）に向いているが、多用は非推奨

* **Liquid Clustering**

  * 複数列でのフィルタや更新が多いケースに強く、Databricks推奨の次世代手法

新規開発ではまず Liquid Clustering を検討し、必要に応じて粗いパーティションと組み合わせるのがベストプラクティスです。
