---
title: "databricksの。"
emoji: "🌱"
type: "tech"
topics: [""]
published: false
publication_name: "headwaters"
---

databricksを触ってくぜ！でも何かしたらいいかわからん！という私のために書いています。

# この投稿でできるようになること

1. CTAS（Create Table As Select）を使って Delta Lake テーブルを作る
　**→ SELECT の結果から直接テーブルを作成して、データも同時に保存する。**

2. 既存のビューやテーブルを元に、新しいテーブルを作成する
　**→ 例えば一時ビューに読み込んだCSVのデータを、加工して保存用テーブルに変換する。**

3. 読み込んだデータに、追加の情報（メタデータ）を付け加える
　**→ 日付や処理時間などを列として追加し、分析や管理に役立つ情報を入れる。**

# セットアップ

早速かよって感じですが、セットアップの諸々はすでにわかりやすい解説があるため省略します。

- はじめてのDatabricks
https://qiita.com/taka_yayoi/items/8dc72d083edb879a5e5d


- Databricksのエクスプレスセットアップ
https://qiita.com/taka_yayoi/items/d180c3e65da26befe9de

- Databricksアカウントのセットアップとワークスペースの作成(実践編)
https://qiita.com/taka_yayoi/items/98edd2e9d06f5c1029a1


サンプルデータについては、Unity Catalogというdatabricksのカタログの中に、すぐに利用できるサンプルデータが格納されています。
自分たちでデータを準備する必要もないので、セットアップはさくっと終わらせましょう。

# Parquetファイルに対するクエリ

ここから基本的な操作や諸々気になったことをメモしていきます。
まずはParquet形式のサンプルファイルに対してSQLクエリを投げて、中身を確認するという処理です。

以下のSQLは、**「Parquetファイルに直接クエリ」** しています。
（出力結果は省略します）

```sql
SELECT * 
FROM parquet.`/Volumes/サンプルファイルのパス/` 
LIMIT 10;
```

- `parquet./Volumes/...`：「このディレクトリの中にあるParquetファイルにアクセスしますよ」という意味

- このパスにあるParquetファイルは、テーブルとして登録されていなくても、SQLで中身を読むことが可能

### Parquetとは？


Parquetファイルは、**列ごとにデータを保存する形式（列指向）**　のファイルです。

大規模データの分析向けで、Spark・Hadoopなどでもよく使われます。CSVが「行単位」で保存するのに対し、Parquetは「列単位」で保存するため、"特定の列だけ使うような処理"が速くなります。

ちなみに読み方は諸説ありますが、「パーケ」と読んでおけば間違いないです。

参照：

> Apache Parquet は、効率的なデータの保存と検索のために設計された、オープンソースの列指向データファイル形式です。
複雑なデータを一括処理するための効率的なデータ圧縮と符号化方式を提供し、パフォーマンスを向上させます。
Apache Parquet は、バッチとインタラクティブの両方のワークロードで共通の交換形式となるように設計されており、Hadoop で利用可能な他の列指向ストレージファイル形式である RCFile や ORC に似ています。

https://www.databricks.com/jp/glossary/what-is-parquet


### なぜテーブル登録していないのにクエリできるのか？

通常のデータベースシステムでは、CSVやParquetなどのデータファイルはテーブルとして登録しないとクエリできません。
ただし、Databricks（というよりSpark）では **「ファイルに直接クエリする機能」** があるため、テーブル登録をせずにファイルに直接クエリを実行することができます。

ただし本格的なデータ分析やパイプラインでは、きちんとDeltaテーブルとして登録するほうが良いです。さっと中身を確認したいときに使用するくらいの感覚がよさそうです。


### Volumesって何？

VolumesはDatabricksのボリュームストレージ領域で、学習用データなどを格納する専用のフォルダです。
読み込み用の場所であり、テーブルとして保存する場所ではないことに注意です。


参照：
> ボリュームは、表形式以外のデータセットに対するガバナンスを可能にする Unity Catalog オブジェクトです。 ボリュームは、クラウド・オブジェクト・ストレージ・ロケーション内のストレージの論理ボリュームを表します。 ボリュームは、ファイルへのアクセス、保存、管理、および整理の機能を提供します。

https://docs.databricks.com/aws/ja/volumes/

# Parquet→Deltaテーブル

上記はParquetデータを確認したのみで、まだテーブルは作成できていません。
ここから先ほど読み取ったParquetデータから、実際にDeltaテーブルを作成してみます。

そこで使用するのが、`CREATE TABLE AS SELECT` ステートメント、通称 **`CTAS`** です。

`sample_bronze`というテーブルを定義しつつ、SELECT文でParquetデータも同時に流し込むことができます。

```sql
CREATE OR REPLACE TABLE sample_bronze 
USING DELTA
AS
SELECT * 
FROM parquet.`/Volumes/サンプルファイルのパス/`;
```

- `USING DELTA`：テーブルの保存形式をDelta Lake形式に定義

- クエリからスキーマ（列名・データ型）を自動的に推論（カラム名やデータ型を書く必要がない）

- `sample_bronze`：作成するテーブルの名前（ブロンズテーブル）



DESCRIBE <table-name>を実行すると、列名とデータ型を見ることができます。このテーブルのスキーマは正しく見えます

### なぜ「ブロンズ」？

データレイクハウスの一般的な設計では、データの加工段階に応じて以下のようにレイヤーを分けます。

- Bronze：生データ（rawデータ）をそのまま格納

- Silver：整形・加工済みの中間データ

- Gold：分析・可視化用に最適化されたデータ

今回の `sample_bronze` は「ブロンズレイヤー」、つまり「Parquetから取り込んだだけの初期状態のデータ」です。


# CSV→Deltaテーブル

次にCSVデータをDeltaテーブルとして保存する方法を記載します。
上ではParquetデータをシンプルな`CTAS`で読み取りましたが、CSVデータを同じ方法で読み取ろうとすると、"|"(区切り)文字やヘッダーのせいで上手くテーブルを作成することができません。

そこで使用するのが `read_files()`という関数です。
`CTAS`のなかで`read_files()`を使用することで、CSVやJSONなどParquet以外の形式ファイルも読み込むことができます。

（出力結果は省略します）

```sql
CREATE TABLE sample_bronze2 
USING DELTA 
AS
SELECT * 
FROM read_files("/Volumes/サンプルファイル-csv/",
      format => "csv",
      sep => "|",
      header => true,
      mode => "FAILFAST");

SELECT * 
FROM sample_bronze2 
LIMIT 10;
```

次のパラメータを使用しています。

- `format => "csv"`：CSV形式のファイルを読み取る       
- `sep => "|" `：  "|"は区切り文字 
- `header => true` ：ファイルの1行目は列名とみなす         
- `mode => "FAILFAST"`：データに問題があったら即エラーにする


参照：

https://docs.databricks.com/aws/en/sql/language-manual/functions/read_files


# カタログ・スキーマ・テーブル

今回はデモのため省略していますが、実際の環境ではテーブルを「どのカタログ／スキーマにテーブルを作るか」を最初に指定する必要があります。カタログ・スキーマ・テーブルのざっくりの関係を書いておきます。

イメージ：
```
Catalog（カタログ）
 └── Schema（スキーマ／従来の「データベース」）
      └── Table（テーブル）
```

カタログ・スキーマを指定する例：
```sql
CREATE TABLE my_catalog.my_schema.sample_bronze
USING DELTA AS ...
```
`USE`を使用する方法もあるみたいですが、これが一番安全かなと思います。




