---
title: Azure AI Document Intelligenceを使ったドキュメント処理の可能性
tags:
  - Azure
  - rag
  - LLM
  - AzureAIDocumentIntelligence
private: false
updated_at: '2025-01-30T20:48:54+09:00'
id: 0b4ccf659df8970a0aef
organization_url_name: null
slide: false
ignorePublish: false
---
本記事ではAzure AI Document Intelligence(AADI)の概要と、ADIを利用して外部のファイルをどれだけ取り込めるかについて調査した結果を共有します。

# 1.Azure AI Document Intelligence(AADI)の概要


## 1-1. AADIの基本
AADIは、「構造を持つドキュメントの処理を支援する」AIサービスです。
スキャンされた文書、画像、PDF などからテキストや構造化データを抽出し、ビジネスプロセスの自動化を支援します。このサービスは、異なる種類の文書を解析し、必要な情報を抽出できるように設計されています。

https://learn.microsoft.com/ja-jp/azure/ai-services/document-intelligence/?view=doc-intel-4.0.0

## 1-2. 3種類のモデル

ADIを使う際は読み取りたいドキュメントの形式・内容によって、以下3モデルのいずれかを選択します。

### a. ドキュメント分析モデル

基本的なドキュメントの構造を解析し、テキスト、テーブル、チェックボックスなどの要素を識別したいときは「ドキュメント分析モデル」を選択します。
このモデルを利用することで、スキャンされた請求書、領収書、契約書などから必要な情報を効率的に取得できます。
と言いつつ、このモデルは後述の「事前構築済みモデル」の1つだそう。一旦公式HPにも記載されているため、記述しておきます。

↓読み取りモデル：印刷されたテキストや手書きテキストを抽出。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/c312c3f9-68ba-3a78-a9a4-5ea4bef9cb6f.png)

↓レイアウトモデル：テキスト、テーブル、ドキュメントの構造を抽出。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/9e6ee338-0e97-9355-df3f-d21901b3159a.png)


### b. 事前構築済みモデル

A特定のユースケース向けには、すでにトレーニング済みの「事前構築済みモデル」が用意されています。例えば、請求書や領収書、パスポート、ID カードなどの処理に特化したモデルがあり、ユーザーはカスタムトレーニングなしでこれらの文書を解析できます。
ただ現在のところ、基本的には米国のファイルに対応したものばかりで、日本で使えるモデルはまだ少なそうです。今後に期待。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/5b71a7e3-5b4b-8a42-d6e0-567fb8e3cbca.png)

https://learn.microsoft.com/ja-jp/azure/ai-services/document-intelligence/overview?view=doc-intel-4.0.0#prebuilt-models

### c. カスタムモデル

事前構築済みモデルで対応できない場合は「カスタムモデル」を使用します。
カスタムモデルでは、特定の文書フォーマットに対して独自のトレーニングを行い、業務に適したデータ抽出が可能になります。
例えば、企業独自の帳票や業務報告書などに対応するために活用できます。


↓カスタムモデルを作成する様子

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/1b4c3c9a-e8b5-1ec7-66c4-2205d8c776ef.png)


そしてこのカスタムモデルのなかでも、以下の2種類があります。


#### c1. カスタムテンプレートモデル

特定のレイアウトや構造に基づいたドキュメントを処理するためのルールベースのモデルです。手動でフィールドを定義し、指定した位置から情報を取得します。
事前構築モデルには含まれない固定フォーマットの契約書や税務書類など、決まったレイアウトのドキュメントを解析する場合に向いています。

#### c2. カスタムニューラルモデル

Azureのニューラルネットワークを活用したモデルで、ラベル付きデータを使ってトレーニングすることで、非構造化・半構造化ドキュメントの情報を抽出できます。
フォームや請求書など、さまざまなフォーマットのドキュメントからテキストや表データを抽出したい場合に適しています。


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/458babb2-dbba-fcb2-05aa-f7d7e497261d.png)


#### カスタムモデルの作成（ラベル付け・トレーニング）
名前の通り、もちろんカスタムなので、モデルを作成するにはそれなりの知識と工数が必要となります。
ここでいう作成方法は、**「データセットのラベル付け」「モデルのトレーニング」** にあたります。
どのデータをサンプルとして用意しどうラベルを付けるかによって、トレーニング済みモデルの精度が変動します。



https://learn.microsoft.com/ja-jp/azure/ai-services/document-intelligence/train/custom-labels?view=doc-intel-4.0.0


https://learn.microsoft.com/ja-jp/azure/ai-services/document-intelligence/train/custom-label-tips?view=doc-intel-4.0.0


# 2.ドキュメント処理の検証

## 2-1. 調査対象

今回は、以下のファイルを2つの形式を検索エンジンのインデックスにインポートする方法を調査しました。

ⅰ. Excelファイル（.xlsx）
ⅱ. Excelファイル（.pdf）
ⅲ. フローチャート（.png）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/7d555537-294f-be6f-0852-e4d052f96afd.png)


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/902f677b-853c-a253-bb3d-6752724dbc05.png)


## 2-2.Azure AI Searchでのデータ取り込み
検索エンジンのインデックスにインポートするにあたって、Azure AI Searchを使用します。
Azure AI Searchについては以下をご覧ください。


https://learn.microsoft.com/ja-jp/azure/search/search-what-is-azure-search

https://qiita.com/setowatson/items/4493f354aa09460cb1b8


## 2-3.事前構築済みモデル「レイアウトモデル」の概要

今回の検証には事前構築済みモデルの1つである「レイアウトモデル」を使用します。
レイアウトモデルは、PDF、JPEG/JPG、PNG、BMP、TIFF、HEIF、DOCX、XLSX、PPTX形式のドキュメントに対応しており、テキストやテーブル情報を抽出するモデルです。

↓公式より。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/710a3e4f-450f-86ed-f3e0-00c5f36f341c.png)

https://learn.microsoft.com/ja-jp/azure/ai-services/document-intelligence/prebuilt/layout?view=doc-intel-4.0.0&tabs=sample-code

## 2-4.検証結果

### ⅰ.Excelファイル（.xlsx）の検証

<details><summary>出力結果</summary>

```ruby:ⅰ.Excelファイル（.xlsx）の検証

ID

チェック

金額（円）

コメント

101

田中 太郎

☑

15000	

縦書きテスト

102

佐藤 花子

☐

20000

要確認

鈴木 一郎

103

☑

18000	

再審査

2

53000
```

</details>

結果:
- テキスト抽出は可能だが、計算式（SUM等）は抽出されない。
- テーブルの構造が一部破壊される
- テーブルの抽出は困難。

### ⅱ.Excelファイル（.png）の検証

<details><summary>出力結果</summary>

```ruby:ⅱ.Excelファイル（.pdf）の検証
Table # 0 has 5 rows and 4 columns
Table # 0 location on page: 1 is [10, 10, 500, 10, 500, 200, 10, 200]
...Cell[0][0] has text 'ID'
...content on page 1 is within bounding polygon '[10, 10, 100, 10, 100, 30, 10, 30]'
...Cell[0][1] has text 'チェック'
...content on page 1 is within bounding polygon '[110, 10, 200, 10, 200, 30, 110, 30]'
...Cell[0][2] has text '金額(円)'
...content on page 1 is within bounding polygon '[210, 10, 300, 10, 300, 30, 210, 30]'
...Cell[0][3] has text 'コメント'
...content on page 1 is within bounding polygon '[310, 10, 400, 10, 400, 30, 310, 30]'
...Cell[1][0] has text '101'
...content on page 1 is within bounding polygon '[10, 40, 100, 40, 100, 60, 10, 60]'
...Cell[1][1] has text '田中 太郎 ✔'
...content on page 1 is within bounding polygon '[110, 40, 200, 40, 200, 60, 110, 60]'
...Cell[1][2] has text '15000'
...content on page 1 is within bounding polygon '[210, 40, 300, 40, 300, 60, 210, 60]'
...Cell[1][3] has text ''
...content on page 1 is within bounding polygon '[310, 40, 400, 40, 400, 60, 310, 60]'
...Cell[2][0] has text '102'
...content on page 1 is within bounding polygon '[10, 70, 100, 70, 100, 90, 10, 90]'
...Cell[2][1] has text '佐藤 花子 □'
...content on page 1 is within bounding polygon '[110, 70, 200, 70, 200, 90, 110, 90]'
...Cell[2][2] has text '20000'
...content on page 1 is within bounding polygon '[210, 70, 300, 70, 300, 90, 210, 90]'
...Cell[2][3] has text '要確認'
...content on page 1 is within bounding polygon '[310, 70, 400, 70, 400, 90, 310, 90]'
...Cell[3][0] has text '103'
...content on page 1 is within bounding polygon '[10, 100, 100, 100, 100, 120, 10, 120]'
...Cell[3][1] has text '鈴木 一郎 ✔'
...content on page 1 is within bounding polygon '[110, 100, 200, 100, 200, 120, 110, 120]'
...Cell[3][2] has text '18000'
...content on page 1 is within bounding polygon '[210, 100, 300, 100, 300, 120, 210, 120]'
...Cell[3][3] has text '再審査'
...content on page 1 is within bounding polygon '[310, 100, 400, 100, 400, 120, 310, 120]'
...Cell[4][0] has text ''
...content on page 1 is within bounding polygon '[10, 130, 100, 130, 100, 150, 10, 150]'
...Cell[4][1] has text '2'
...content on page 1 is within bounding polygon '[110, 130, 200, 130, 200, 150, 110, 150]'
...Cell[4][2] has text '53000'
...content on page 1 is within bounding polygon '[210, 130, 300, 130, 300, 150, 210, 150]'
...Cell[4][3] has text ''
...content on page 1 is within bounding polygon '[310, 130, 400, 130, 400, 150, 310, 150]'
```

</details>


結果:
- Markdown記法を用いることが可能に。
- 元の表データ構造を保持しながら、テーブルの内容を抽出。
- 各セルのバウンディングボックスも抽出。


xlsx形式よりも高精度なテーブル抽出ができました。
画像のほうが上手く解析できるんですね。

### ⅲ.フローチャート（.png）の検証

<details><summary>出力結果</summary>

```ruby:ⅲ.フローチャート（.png）の検証
<figure>
開始

処理A

判断

はい

いいえ

処理Ｂ

終了

<figure>
```

</details>

結果:


- チャート図形内の文字のみを抽出。
- 関係性や構造の意味は失われる。

このレイアウトモデルではチャートの処理は難しそうです。

## 2-5. 結果・考察

AADIのレイアウトモデルを使用して、検証しました。
画像形式に変換した表ファイル（スクリーンショット等）は比較的正確に情報を抽出できることがわかりました。一方、フローチャートは図形に意味を見出すことまでは解析できませんでした。
検証はキリがないので、いったんここで打ち切っています。

これはなんとなくみなさんにも理解していただけると思うのですが、おそらくExcel自体がOCR解析には向いていません。
セルを丁寧に扱っている表や前処理をしきった表データ等であれば、読み取ることができますが、世のExcelはそんな綺麗な使われ方ばかりされていません。
むしろ社内マニュアルや議事録などをExcelを１ボードとして使用している企業も多いと思います。

そして生成AIを活用したいという企業は、そうしたExcelを渡してきたりします。（この結論にもっていきたかったのd)


生成AI活用の世界においては、もはやあるあるになってきていると思いますが、「AIに何を与えるか」というフェーズは非常に重要です。
AADIを活用する場合も同じということですね。


# 3.まとめ
Azure AI Document Intelligence は、多様なドキュメント処理を自動化できる強力なツールです。PDF、JPEG、PNG などの一般的なファイル形式に対応し、請求書や契約書、領収書などの解析に活用できます。特に、構造化されたフォームや表が含まれる文書の処理に適しており、高精度で情報を抽出できます。

ただし、米国のみの対応もあるのが現状です。
また手書き文字が崩れている文書や、Excelをはじめとする複雑なレイアウトの文書では精度が低下する可能性があります（もちろんExcelは使い方に大きく左右される。）
また、暗号化された PDF や極端に圧縮された画像も解析が難しい場合があるようです。

いずれにしても、事前に適切な形式でドキュメントを準備することが重要です。
これはAADIに限った話ではなく、LLMやRAGを作成する際にどのモデルを使用しようとしても避けては通れない道です。現在のところ、の話ですが。

AADIは、機能性はもちろんその他のAzureサービスとの連携やセキュリティレベルにおいてとても優れたサービスです。
ぜひAADIを活用することで、企業は手作業によるデータ入力の負担を軽減し、多くの組織が効率的なドキュメント管理を実現いただければと思います。


#### 補足・注意

- このブログで参照されている、Microsoft、Azure、Azure OpenAI、PowerAppsその他のマイクロソフト製品およびサービスは、米国およびその他の国におけるマイクロソフトの商標または登録商標です。
- 私は、Microsoftとは直接関係のない個人として本ブログを記載しています。
- この記事の情報は公開時点のものであり、サービスは随時アップデートされています。Azureのサービスは進化が早いため、最新情報は公式ＨＰをご覧ください。
