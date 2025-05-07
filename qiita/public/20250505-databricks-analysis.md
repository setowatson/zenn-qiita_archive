---
title: 弊社の高PV記事の特徴をPythonで分析してみた。
publication_name: headwaters
private: false
tags:
  - databricks
  - 機械学習
  - データ分析
  - zenn
updated_at: '2025-05-07T23:03:01.091Z'
id: null
organization_url_name: null
slide: false
---
# はじめに

この記事では、Zennのpublicationプランにある「記事エクスポート機能」を活用し、Databricksのノートブック上で弊社ヘッドウォータースの人気投稿を分析した結果を紹介します。

https://zenn.dev/zenn/articles/publication-pro-features#%E7%B5%B1%E8%A8%88%E3%83%80%E3%83%83%E3%82%B7%E3%83%A5%E3%83%9C%E3%83%BC%E3%83%89%E6%A9%9F%E8%83%BD

## 結論

忙しい方のために結論だけ。

1. 平均PVが最も高い単語: **nvidia** 
2. 最もPVが高い技術カテゴリ: **ハードウェア** 
3. 最適なタイトル長: **10~20 文字**
4. 最もPVを取りやすくなる記事公開曜日： **水曜日**
5. 公開後のPVピークは公開から： **4 日目**

つまりPV数を稼ぎやすいのは、

**「"NVIDIA"の"ハードウェア"に関する記事を"10~20字前後のタイトル"で"水曜日"に公開すること」**

となります。


# 今回行う分析の概要

## 目的

- 人気記事の特徴を定量的に把握する
- 機械学習を用いて記事の傾向を分析する
- 今後の記事作成の参考となる知見を得る

## 対象

- 記事：ヘッドウォータースとして投稿された記事
- 期間：2024/11/1〜2025/4/30の半年間
- 投稿数：275個
- *対象記事数：61124個

![Zenn Publicationのエクスポート画面](/images/zenn2.png)

こちらはZennのpublicationページです。
半年間で合計約35万回も表示されています。すごいですね。

このページから期間を絞って、PV数などを記録したデータをCSV形式でダウンロードできます。

以下の列が含まれています。

- ユーザー名
- slug(記事ID)
- タイトル
- PV発生日*
- PV数
- 記事公開日

*便利なようで厄介なのが「PV発生日」です。
実際にこの期間に投稿されたのは275記事しかなくとも閲覧があったタイミングでカウントされるので、同じ投稿で複数の行になっていたり、上記期間から過去の記事も対象となります。
そのため、実際のデータとしては61,124行ありました。

![データの構造を示すスクリーンショット](/images/zenn4.png)

これはタイトルでまとめて処理をかけています。
2024/11/1以前の記事は対象外にすることや日づけ毎に分けて考えるなどの前処理も検討できると思いましたが、一旦今回は簡単な分析を目標にしているのでこのまま進めます。

## 方針

上記CSVデータを元に大きく下記2つの方針で分析しています。

1.概要分析（基本的な統計量）
2.機械学習モデルを用いた深層分析（ランダムフォレスト）

どちらも使用言語はPythonです。

# 1. 概要分析

まず、記事の基本的な統計量を確認しました。
データを整理したのみで、機械学習は使っていません。

## ノートブック①

実際のコードはこちらです。

https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/4372917459524351/2835289172474537/7861969920932136/latest.html


## 平均PVが高くなる単語トップ20のグラフ

![平均PVが高い単語トップ20のグラフ](/images/zenntop20.png)

1位：nvidia
2位：raspberry
3位：pi
4位：api
5位：azure
…

タイトルに含まれる単語を分析しています。
いくつか表示エラーが起きていますが、こんな感じです。
ヘッドウォータースの強みの領域も多く含まれています、反響があるのは嬉しいことですね。

ちなみに意味のない助詞や数字は除かれるようストップワードとして設定しています。

## 曜日別平均PV

![曜日別平均PVのグラフ](/images/zennday.png)

なんとダントツで水曜日が1位。
なぜ？と思い、いろいろ調べてみましたが分かりませんでした。
みなさん水曜日に投稿しがちなのか、バズった記事が水曜日に集中しているのかわかりませんが、おそらく後者な気がします。

Zennにアクセスしやすい時間帯や曜日まで確認できれば、もっと深く調査できそうです。


## 記事公開からの経過日数とPVの関係

![記事公開からの経過日数とPVの関係グラフ](/images/zenn30days.png)

1日あたりのPV数は、投稿してから4~5日でピークとなります。
ただし、30日を経っても反響がある記事もあるため、記事を書き続けることにも意味があるように思えます。

## タイトル長とPVの関係

![タイトル長とPVの関係グラフ](/images/zennlength.png)

50字前後のタイトルが良さそ...と思いましたが、
よく見ると外れ値が引っ張っている様子ですね。
全体のバランスを見ると、10〜20字と短いタイトルが良さそう？

## 技術カテゴリ分析

![技術カテゴリ分析のグラフ](/images/zenncategory.png)

主要カテゴリの定義は以下です。

- 'AI/ML': `['ai', 'ml', 'machine', 'learning', 'generative', 'gpt', 'llm', 'openai', 'chatgpt', 'bert', 'transformer']`
- 'Cloud': `['cloud', 'aws', 'azure', 'gcp', 'google', 'serverless', 'lambda', 'ec2', 's3']`
- 'Frontend': `['javascript', 'typescript', 'react', 'vue', 'angular', 'css', 'html', 'frontend', 'web']`
- 'Backend': `['backend', 'server', 'api', 'rest', 'graphql', 'database', 'sql', 'nosql', 'mongodb']`
- 'Mobile': `['mobile', 'ios', 'android', 'swift', 'kotlin', 'flutter', 'react-native']`
- 'DevOps': `['devops', 'docker', 'kubernetes', 'k8s', 'ci', 'cd', 'pipeline', 'jenkins', 'github']`
- 'Hardware': `['hardware', 'raspberry', 'pi', 'arduino', 'nvidia', 'gpu', 'processor', 'chip']`

平均PV数が1位なのはこちらもダントツでハードウェアでした。

## ユーザー分析

![ユーザー分析のグラフ](/images/zennuser.png)

こちらはトータルPV数。
1位は @hwstakechiyo さん

https://zenn.dev/hwstakechiyo

そもそもの記事数がダントツでした。見習います！

# 2. 機械学習による深層分析

次に、機械学習を用いて以下の分析を行いました。

- 平均PVが高い単語トップ20のグラフ
- 曜日別平均PV
- 記事公開からの経過日数とPVの関係
- タイトル長とPVの関係
- 技術カテゴリ分析

一つ一つ共有すると長くなってしまうのと結論はほぼ上と同じなので、ノートブックの共有のみにします。もしお時間ある方は↓のノートブックの内容をご覧ください。

## ノートブック②

https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/4372917459524351/2835289172474566/7861969920932136/latest.html

## 使用した機械学習モデル

- ランダムフォレストモデル

===== 機械学習モデルの構築 =====

- カテゴリ特徴量: 2個
- 数値特徴量: 25個
- テストデータでの平均二乗誤差: 238.26
- テストデータでの決定係数 (R²): 0.50
- 3分割交差検証のRMSE: 13.55

詳しい解説は参照を見ていただきたいですが、あまりいいモデルとは言えません。モデルとしてはもっと工夫の余地がありそうでした。
あとは日本語の形態素解析なんかやってみるとバズるタイトルをもっとわかりやすく出せるかもしれませんね。


# まとめ

1. **記事作成の黄金タイミング**
   - 水曜日の朝に投稿
   - 週末に向けての記事を意識

2. **タイトル作成のコツ**
   - 10~20文字前後を目安に
   - 内容を端的に表現
   - 触れるサービス名も含める
   
3. **カテゴリ戦略**
   - ハードウェア系の記事を中心に
   - NVIDIA関連の最新情報を積極的に
   - 実践的な内容を重視


タイトル案として、

- NVIDIAの最新GPU解説（15字）
- NVIDIAの技術力とは（15字）
- 進化するNVIDIA製品（14字）
- NVIDIAのAI向けGPU（15字）
- NVIDIAチップの強み（14字）
- NVIDIA開発の裏側（14字）
- 次世代GPUとNVIDIA（14字）

なんて感じで書いてみてはいかがでしょうか？


今回の分析で、人気記事の特徴が定量的に把握できました。
これらの知見を活かして、私もより多くの方に読んでいただける記事を作成していきたいと思います。

（とはいえ、大事なのは記事の内容です。自分のためになってどこかの誰かのためになる質の高い記事作成に取り組んでいきます！）

## 今後やりたいこと

今回の分析では基本的な機械学習モデルを使用しましたが、Databricksの強力な機能を活用することで、より高度な分析が可能です。

- **MLflowの活用**
  - モデルのバージョン管理
  - 実験の追跡と比較
  - モデルのデプロイメント自動化

- **Feature Storeの活用**
  - 特徴量の管理と共有
  - 特徴量の再利用性向上
  - データの一貫性確保

- **AutoMLの活用**
  - 自動的な特徴量エンジニアリング
  - ハイパーパラメータの最適化
  - モデルの自動選択

- **Unity Catalogの活用**
  - データのガバナンス
  - アクセス制御
  - データのライニージ

これらの機能を活用することで、より精度の高い分析や予測が可能になると考えています。次回の分析では、これらの機能を積極的に活用していきたいと思います。

## 参考資料

### Databricks関連
- [Databricks ノートブック](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/4372917459524351/2835289172474537/7861969920932136/latest.html)
- [Databricks MLOps ガイド](https://docs.databricks.com/en/mlflow/index.html)
- [Delta Lake ドキュメント](https://docs.delta.io/latest/index.html)
- [Databricks Feature Store 入門](https://docs.databricks.com/en/machine-learning/feature-store/index.html)
- [Databricks AutoML チュートリアル](https://docs.databricks.com/en/machine-learning/automl/index.html)

### Zenn関連
- [Zenn Publication 機能]([https://zenn.dev/zenn/articles/what-is-publicatio](https://zenn.dev/zenn/articles/publication-pro-features#%E7%B5%B1%E8%A8%88%E3%83%80%E3%83%83%E3%82%B7%E3%83%A5%E3%83%9C%E3%83%BC%E3%83%89%E6%A9%9F%E8%83%BD)
- [Zennの記事作成ガイドライン](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Zennの記事公開のベストプラクティス](https://zenn.dev/zenn/articles/zenn-article-best-practices)

### データ分析・機械学習関連
- [時系列分析入門 - Prophet](https://facebook.github.io/prophet/docs/quick_start.html)
- [自然言語処理の基礎 - spaCy](https://spacy.io/usage/linguistic-features)
- [機械学習モデルの評価指標](https://scikit-learn.org/stable/modules/model_evaluation.html)
- [特徴量エンジニアリングのベストプラクティス](https://www.kaggle.com/learn/feature-engineering)

### 関連記事
- [Databricksで始めるMLOps](https://zenn.dev/databricks/articles/databricks-mlops-introduction)
- [Delta Lakeによるデータレイクハウスの構築](https://zenn.dev/databricks/articles/delta-lake-data-lakehouse)
- [Zenn記事のPV分析とSEO対策](https://zenn.dev/articles/zenn-pv-analysis-seo)
- [機械学習によるコンテンツ分析の実践](https://zenn.dev/articles/ml-content-analysis)

### ツール・ライブラリ
- [MLflow ドキュメント](https://mlflow.org/docs/latest/index.html)
- [scikit-learn チュートリアル](https://scikit-learn.org/stable/tutorial/index.html)
- [pandas データ分析入門](https://pandas.pydata.org/docs/getting_started/intro_tutorials/index.html)
- [matplotlib 可視化ガイド](https://matplotlib.org/stable/tutorials/index.html) 