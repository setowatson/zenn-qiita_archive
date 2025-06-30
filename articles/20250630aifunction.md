---
title: "Databricksだと簡単SQLだけで半年先の人口を予測できる"
emoji: "🌱"
type: "tech"
topics: ["databricks"]
published: false
publication_name: "headwaters"
---


 
# 出力イメージ
 
生成AIや自動予測が当たり前になりつつある中、**簡単なSQLだけで時系列予測ができるDatabricksの `ai_forecast()` 関数**が登場しました。
 
https://docs.databricks.com/gcp/ja/sql/language-manual/functions/ai_forecast
 

# 1. ai_forecast 関数とは？
 
Databricksの `ai_forecast()` は、**時系列データに対して将来の値を予測してくれるAI組込関数**です。
SQLで簡単に以下のことができます。
 
* 特定の期間までの将来予測
* グループごとの個別予測（例：市区別）
* 上限・下限（予測区間）の自動計算
 
> 公式ドキュメントはこちら
https://docs.databricks.com/gcp/ja/sql/language-manual/functions/ai_forecast
 

# 2. 使ったデータと目的
 
今回使ったのは、**Databricks上に保存された東京都の市区別・月次人口データ** です。
このデータには2020年〜2025年1月までの実測人口が入っており、これを元に**2025年2月〜7月までを予測**しました。
 
 
### 🟦 3. 予測テーブルの作成（SQLだけ）
 
以下のように、`ai_forecast()` を使って未来6ヶ月分の人口予測を生成し、Deltaテーブルとして保存します。
 
```sql
DROP TABLE IF EXISTS workspace.popu_featured.population_forecast_202502_to_202507;
 
CREATE TABLE workspace.popu_featured.population_forecast_202502_to_202507 AS
SELECT * FROM ai_forecast(
  TABLE (
    SELECT 
      `地域コード` AS area_code, 
      CAST(`年月` AS DATE) AS ds, 
      `人口` AS population
    FROM workspace.popu_featured.population_all_ver2
  ),
  horizon => '2025-08-01',
  time_col => 'ds',
  value_col => 'population',
  group_col => 'area_code',
  frequency => 'MS',
  parameters => '{"global_floor": 0}'
);
```
 
📝 補足：
 
* `group_col` には地域コードを指定（グループごとに予測）
* `frequency => 'MS'`：月初ごとに予測
* `parameters` で人口が0未満にならないよう制約を追加
 

 
# 4. ダッシュボードで可視化
 
作成した予測テーブルには以下の列が含まれます：
 
\| ds（日付） | area\_code（地域） | population\_forecast | population\_upper | population\_lower |
 
これを使って、Databricksダッシュボードで**予測線＋上限/下限帯グラフ** を作成します。
 
#### 📈 Visualization設定例：
 
| 設定項目                 | 値                                      |
| -------------------- | -------------------------------------- |
| Visualization Type   | Line                                   |
| X-axis               | `ds`                                   |
| Y-axis               | `population_forecast`                  |
| Optional Range Lines | `population_upper`, `population_lower` |
| Filter               | `area_code`（地域を切り替え表示可能）               |
 
# 5. 実測と予測の比較グラフも
 
さらに以下のSQLで、**実測人口（2024年7月以降）と予測人口を並べて比較**できます。
 
```sql
SELECT
  ds,
  area_code,
  '実測人口' AS `種類`,
  population AS `人口`
FROM (
  SELECT 
    CAST(`年月` AS DATE) AS ds,
    `地域コード` AS area_code,
    `人口` AS population
  FROM workspace.popu_featured.population_all_ver2
  WHERE `年月` >= '2024-07-01'
)
UNION ALL
SELECT
  ds,
  area_code,
  '予測人口' AS `種類`,
  population_forecast AS `人口`
FROM workspace.popu_featured.population_forecast_202502_to_202507
```
 
このクエリで出力された結果を「Lineグラフ（X軸：ds、色分け：種類）」にすると、**予測と実測の差異が一目瞭然**になります。
 
# 所感と活用ポイント
 
* SQLだけでここまでできるのは非常に強力。Pythonや機械学習モデルの知識がなくてもOK。
* 定期更新ジョブやAuto Loaderと組み合わせれば、**毎月新しいデータが来たら自動予測＆可視化も可能**
* 各地域の増減傾向や、将来の人口集中・過疎もすぐに発見できる

 
`ai_forecast()` 関数は、**Databricks SQLユーザーにとって「使わない理由がない」ほど便利な時系列予測ツール**です。
 
業務データ（売上・来客数・在庫）やオープンデータ（人口・交通量・気温）など、あらゆる時系列データに応用できます。

SQLだけで未来を予測し、意思決定に活かしたい人にはぜひ一度触ってみてほしい関数です。

