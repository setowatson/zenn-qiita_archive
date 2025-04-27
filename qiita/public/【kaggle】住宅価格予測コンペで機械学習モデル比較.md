---
title: 【kaggle】住宅価格予測コンペで機械学習モデル比較
tags:
  - Python
  - 初心者
  - 機械学習
  - Kaggle
private: false
updated_at: '2024-08-08T07:38:59+09:00'
id: 6ed212afc155e9c23de2
organization_url_name: null
slide: false
ignorePublish: false
---

# 1.今回の分析コンペの概要
「House Prices: Advanced Regression Techniques」はアメリカ合州国アイオワ州エイムズ市の戸建て住宅の価格を79個の変数から予測する、テーブルデータの回帰予測問題です。

https://www.kaggle.com/c/house-prices-advanced-regression-techniques

学習データ ： 1460件
予測データ ： 1459件


ちなみにアイオワ州エイムズ市はこのあたり。
５大湖自然豊かでトウモロコシや畜産が盛んな地域のようです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/ac404ef2-2df1-6ed0-e2b0-34476183a158.png)

以下引用：

https://www.google.com/maps/place/%E3%82%A2%E3%83%A1%E3%83%AA%E3%82%AB%E5%90%88%E8%A1%86%E5%9B%BD+%E3%82%A2%E3%82%A4%E3%82%AA%E3%83%AF%E5%B7%9E+%E3%82%A8%E3%82%A4%E3%83%A0%E3%82%BA/@39.5982578,-97.8835819,5z/data=!4m6!3m5!1s0x87ee70624634a06b:0x273156083cc75200!8m2!3d42.0307812!4d-93.6319131!16zL20vMG5tag?entry=ttu

https://www.gousa.jp/state/iowa



# 2.データのインポート・傾向確認

まずは必要となる基本的なライブラリをインポートします。

```ruby:ライブラリのインポート.py
import sklearn
import pandas as pd
import matplotlib.pyplot as plt　
%matplotlib inline
import numpy as np
import warnings
warnings.filterwarnings('ignore') #不要なログを省略

import os
for dirname, _, filenames in os.walk('/kaggle/input'):
    for filename in filenames:
        print(os.path.join(dirname, filename))

```

次に対象となる説明変数・目的変数を読み込んでいきます。
kaggleにもともと用意されていますので、pandasを用いてcsv形式で読み込みます。
また、最初にどんなカラムがあるのか確認しておきます。

```ruby:データの読み込み.py
# train_dfとして、/kaggle/input/house-prices-advanced-regression-techniques/train.csvをpandasで読み込み
train_df = pd.read_csv('/kaggle/input/house-prices-advanced-regression-techniques/train.csv')

# test_dfとして、/kaggle/input/house-prices-advanced-regression-techniques/test.csvをpandasで読み込み
test_df = pd.read_csv('/kaggle/input/house-prices-advanced-regression-techniques/test.csv')

# Idのカラムは不要のため削除
# inplaceオプションはデフォルトがFlaseで、その場合dfを変更できません。
train_Id = train_df['Id']
test_Id = test_df['Id']
train_df.drop(columns=['Id'], inplace=True)
test_df.drop(columns=['Id'], inplace=True)

# train_dfとtest_dfを連結してデータフレーム形式でcombined_dfという変数に格納・出力
# set_optionにより列を省略せずに出力
pd.set_option('display.max_columns', 80)
combined_df = pd.concat((train_df, test_df))
combined_df

>>> 出力結果
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/9e347909-bebd-41a9-40ab-e758c8240e22.png)

＊画面に映らないカラムは省略
データ件数は2919、カラムは80あることが確認できます。
Excelではなかなか扱いづらい件数ですね。

今回set_optionを使用しているので、上記ですべてのカラムは目視できますが、
念の為、もう１度カラムを確認しておきます。

```ruby:カラムを確認.py
from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"

#1. train_dfの特徴量を出力
print(train_df.columns.values)

>>>出力結果
['MSSubClass' 'MSZoning' 'LotFrontage' 'LotArea' 'Street' 'Alley'
 'LotShape' 'LandContour' 'Utilities' 'LotConfig' 'LandSlope'
 'Neighborhood' 'Condition1' 'Condition2' 'BldgType' 'HouseStyle'
 'OverallQual' 'OverallCond' 'YearBuilt' 'YearRemodAdd' 'RoofStyle'
 'RoofMatl' 'Exterior1st' 'Exterior2nd' 'MasVnrType' 'MasVnrArea'
 'ExterQual' 'ExterCond' 'Foundation' 'BsmtQual' 'BsmtCond' 'BsmtExposure'
 'BsmtFinType1' 'BsmtFinSF1' 'BsmtFinType2' 'BsmtFinSF2' 'BsmtUnfSF'
 'TotalBsmtSF' 'Heating' 'HeatingQC' 'CentralAir' 'Electrical' '1stFlrSF'
 '2ndFlrSF' 'LowQualFinSF' 'GrLivArea' 'BsmtFullBath' 'BsmtHalfBath'
 'FullBath' 'HalfBath' 'BedroomAbvGr' 'KitchenAbvGr' 'KitchenQual'
 'TotRmsAbvGrd' 'Functional' 'Fireplaces' 'FireplaceQu' 'GarageType'
 'GarageYrBlt' 'GarageFinish' 'GarageCars' 'GarageArea' 'GarageQual'
 'GarageCond' 'PavedDrive' 'WoodDeckSF' 'OpenPorchSF' 'EnclosedPorch'
 '3SsnPorch' 'ScreenPorch' 'PoolArea' 'PoolQC' 'Fence' 'MiscFeature'
 'MiscVal' 'MoSold' 'YrSold' 'SaleType' 'SaleCondition' 'SalePrice']
```

ブログで伝わりやすいように、日本語にしておきます。
和訳ご希望の方は折りたたみを表示ください。
一番最後の「SalePrice: 米ドルでの販売価格」が目的変数です。

<details><summary>各カラムの和訳</summary>

SalePrice: 米ドルでの販売価格.

MSSubClass: 物件の等級.

MSZoning: 一般的な区画の分類

LotFrontage: 土地が面している道の長さ

LotArea: 土地の大きさ

Street: 道路アクセスのタイプ

Alley: 路地アクセスのタイプ

LotShape: 物件の一般的な形状

LandContour: 物件の平坦性

Utilities: 利用可能なユーティリティのタイプ

LotConfig: ロット構成

LandSlope: 物件の傾斜

Neighborhood: エイムス市内の制限内の物理的な場所

Condition1: 主要道路または鉄道に近接している

Condition2: 幹線道路または鉄道に近接している（2番目がある場合）

BldgType: 住居のタイプ

HouseStyle: 住居のスタイル

OverallQual: 全体的な材料と仕上げの品質

OverallCond: 全体的な状態の評価

YearBuilt: 元の建設日

YearRemodAdd: 改築日

RoofStyle: 屋根のタイプ

RoofMatl: 屋根の素材

Exterior1st: 家の外装

Exterior2nd: 家の外装（複数の材料の場合）

MasVnrType: 石積みのベニヤのタイプ

MasVnrArea: 平方フィートの石積みベニア面積

ExterQual: 外装材の品質

ExterCond: 外装材の現状

Foundation: 基礎部分の種類

BsmtQual: 地下室の高さ

BsmtCond: 地下室の一般的な状態

BsmtExposure: ウォークアウトまたは庭園レベルの地下壁

BsmtFinType1: 地下の仕上がり面積

BsmtFinSF1: Type 1 仕上げの平方フィート

BsmtFinType2: Type 2 仕上げした領域の品質（存在する場合）

BsmtFinSF2: Type 2仕上げの平方フィート

BsmtUnfSF: 地下室の未仕上げの平方フィート

TotalBsmtSF: 地下面積の合計平方フィート

Heating: 暖房の種類

HeatingQC: 暖房の品質と状態

CentralAir: セントラルエアコン

Electrical: 電気システム

1stFlrSF: 1階の平方フィート

2ndFlrSF: 2階の平方フィート

LowQualFinSF: 低品質仕上げの平方フィート（すべてのフロア）

GrLivArea: 地上のリビングエリアの平方フィート

BsmtFullBath: 地下のフルサイズバスルーム

BsmtHalfBath: 地下のハーフサイズバスルーム

FullBath: 地上のフルサイズバスルーム

HalfBath: 地上のハーフサイズバスルーム

Bedroom: 地上階のベッドルームの数

Kitchen: キッチンの数

KitchenQual: キッチンの品質

TotRmsAbvGrd: 地上の部屋数の合計（バスルームは含まれない）

Functional: 家の機能の評価

Fireplaces: 暖炉の数

FireplaceQu: 暖炉の品質

GarageType: ガレージの場所

GarageYrBlt: ガレージが建設された年

GarageFinish: ガレージの内部仕上げ

GarageCars: 車の容量でのガレージのサイズ

GarageArea: ガレージの平方フィートでのサイズ

GarageQual: ガレージの品質

GarageCond: ガレージの状態

PavedDrive: 舗装された私道

WoodDeckSF: 平方フィートのウッドデッキ領域

OpenPorchSF: 平方フィートのオープンポーチエリア

EnclosedPorch: 平方フィートで囲まれたポーチエリア

3SsnPorch: 平方フィートのスリーシーズンポーチエリア

ScreenPorch: 平方フィートのスクリーンポーチエリア

PoolArea: 平方フィートのプール面積

PoolQC: プールの品質

Fence: フェンスの品質

MiscFeature: 他のカテゴリではカバーされていないその他の機能

MiscVal: その他の機能の米ドル価値

MoSold: 販売月

YrSold: 販売年

SaleType: 販売のタイプ

SaleCondition: 販売の条件


</details>

一般的に考えると「TotalBsmtSF: 地下面積の合計平方フィート」は、SalePriceと相関がありそうですよね。中には暖炉や地下室など、The・アメリカの大きなお家！というような情報もありますね。

```ruby:グラフの表示.py
# TotalBsmtSFとSalePriceの散布図を作成(x軸にTotalBsmtSF、y軸にSalePriceを指定)
train_df.plot.scatter(x='TotalBsmtSF',y='SalePrice')
>>>出力結果
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/d8aa87fb-e21d-99c0-ed43-01f7c5ee3d9c.png)

やはり正の強い相関がありそうですね。
ここでは取り扱いませんが、右側に外れ値があるのも気になります。
ダントツで一番広い家なのに安い。事故物件か？
ちなみに6000平方フィート＝540平米らしいですよ。アメリカすごい。

# 3.前処理

ここからデータの前処理フェーズに移行します。
まずは欠損値の確認。

```ruby:欠損値の個数を見える化.py
pd.set_option('display.max_rows', 500)
#1. train_dfの全ての特徴量について、`isnull`メソッドと`sum`メソッドを利用して欠損値の有無を確認
train_df.isnull().sum()
#2. test_dfの全ての特徴量について、`isnull`メソッドと`sum`メソッドを利用して欠損値の有無を確認
test_df.isnull().sum()

>>>出力結果
```
<details><summary>出力結果（データが多いため折りたたんでいます。）</summary>

MSSubClass          0
MSZoning            0
LotFrontage       259
LotArea             0
Street              0
Alley            1369
LotShape            0
LandContour         0
Utilities           0
LotConfig           0
LandSlope           0
Neighborhood        0
Condition1          0
Condition2          0
BldgType            0
HouseStyle          0
OverallQual         0
OverallCond         0
YearBuilt           0
YearRemodAdd        0
RoofStyle           0
RoofMatl            0
Exterior1st         0
Exterior2nd         0
MasVnrType        872
MasVnrArea          8
ExterQual           0
ExterCond           0
Foundation          0
BsmtQual           37
BsmtCond           37
BsmtExposure       38
BsmtFinType1       37
BsmtFinSF1          0
BsmtFinType2       38
BsmtFinSF2          0
BsmtUnfSF           0
TotalBsmtSF         0
Heating             0
HeatingQC           0
CentralAir          0
Electrical          1
1stFlrSF            0
2ndFlrSF            0
LowQualFinSF        0
GrLivArea           0
BsmtFullBath        0
BsmtHalfBath        0
FullBath            0
HalfBath            0
BedroomAbvGr        0
KitchenAbvGr        0
KitchenQual         0
TotRmsAbvGrd        0
Functional          0
Fireplaces          0
FireplaceQu       690
GarageType         81
GarageYrBlt        81
GarageFinish       81
GarageCars          0
GarageArea          0
GarageQual         81
GarageCond         81
PavedDrive          0
WoodDeckSF          0
OpenPorchSF         0
EnclosedPorch       0
3SsnPorch           0
ScreenPorch         0
PoolArea            0
PoolQC           1453
Fence            1179
MiscFeature      1406
MiscVal             0
MoSold              0
YrSold              0
SaleType            0
SaleCondition       0
SalePrice           0
dtype: int64
MSSubClass          0
MSZoning            4
LotFrontage       227
LotArea             0
Street              0
Alley            1352
LotShape            0
LandContour         0
Utilities           2
LotConfig           0
LandSlope           0
Neighborhood        0
Condition1          0
Condition2          0
BldgType            0
HouseStyle          0
OverallQual         0
OverallCond         0
YearBuilt           0
YearRemodAdd        0
RoofStyle           0
RoofMatl            0
Exterior1st         1
Exterior2nd         1
MasVnrType        894
MasVnrArea         15
ExterQual           0
ExterCond           0
Foundation          0
BsmtQual           44
BsmtCond           45
BsmtExposure       44
BsmtFinType1       42
BsmtFinSF1          1
BsmtFinType2       42
BsmtFinSF2          1
BsmtUnfSF           1
TotalBsmtSF         1
Heating             0
HeatingQC           0
CentralAir          0
Electrical          0
1stFlrSF            0
2ndFlrSF            0
LowQualFinSF        0
GrLivArea           0
BsmtFullBath        2
BsmtHalfBath        2
FullBath            0
HalfBath            0
BedroomAbvGr        0
KitchenAbvGr        0
KitchenQual         1
TotRmsAbvGrd        0
Functional          2
Fireplaces          0
FireplaceQu       730
GarageType         76
GarageYrBlt        78
GarageFinish       78
GarageCars          1
GarageArea          1
GarageQual         78
GarageCond         78
PavedDrive          0
WoodDeckSF          0
OpenPorchSF         0
EnclosedPorch       0
3SsnPorch           0
ScreenPorch         0
PoolArea            0
PoolQC           1456
Fence            1169
MiscFeature      1408
MiscVal             0
MoSold              0
YrSold              0
SaleType            1
SaleCondition       0
dtype: int64
</details>

いくつかのカラムで、欠損値の多さが目立ちますね。
欠損値を修正していきます。
まずこちらの修正のため、データの重複・最頻値等を確認します。

```ruby:データの確認.py
# train_dfにdescribeメソッドを利用して、データの重複を確認
# countはデータの個数（件数）、uniqueは重複を排除したデータ個数、topは最も多く含まれる値、freqはtopの値が含まれる個数
train_df.describe(include=['O'])

>>>出力結果
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/e938a63b-51ca-b7a8-149b-911c8edeb458.png)

特徴量には、「カテゴリ値」と「数値」が存在します。

- カテゴリ値…定性データ
Ex)  MSSubClass:物件の等級、OverallCond:全体的な状態の評価
- 数値…定量データ
Ex)  LotFrontage:土地が面している道の長さ

またカテゴリ値の中でも、「名義尺度」と「順序尺度」の２つがあります。
- 名義尺度...単に区別されるために用いられる尺度
Ex)  RoofStyle:屋根のスタイル

- 順序尺度...大小関係のみに意味がある尺度
Ex)  OverallQual:全体的な材料と仕上げの品質

これらの特徴量の種類によって、欠損値をどう扱うかが変わります。
以下は、特徴量の特徴に応じて、欠損値をどう穴埋めしているかを反映しています。
データの型をstr型(文字列)に変更するのも忘れず行っておきましょう。

```ruby:欠損値の穴埋め.py

# "PoolQC", "MiscFeature", "Alley", "Fence", "FireplaceQu"の欠損値を”None”で穴埋め
combined_df["PoolQC"] = combined_df["PoolQC"].fillna("None")
combined_df["MiscFeature"] = combined_df["MiscFeature"].fillna("None")
combined_df["Alley"] = combined_df["Alley"].fillna("None")
combined_df["Fence"] = combined_df["Fence"].fillna("None")
combined_df["FireplaceQu"] = combined_df["FireplaceQu"].fillna("None"
                                                              )
# for文を使って'GarageType', 'GarageFinish', 'GarageQual', 'GarageCond'の欠損値を’None’で穴埋め
for col in ('GarageType', 'GarageFinish', 'GarageQual', 'GarageCond','BsmtQual', 'BsmtCond', 'BsmtExposure', 'BsmtFinType1', 'BsmtFinType2',"MasVnrType"):
    combined_df[col] = combined_df[col].fillna('None')

# "LotFrontage"を"Neighborhood"ごとの"LotFrontage"の中央値で穴埋め
combined_df["LotFrontage"] = combined_df.groupby("Neighborhood")["LotFrontage"].transform(
    lambda x: x.fillna(x.median()))

# 'BsmtFinSF1', 'BsmtFinSF2', 'BsmtUnfSF','TotalBsmtSF', 'BsmtFullBath', 'BsmtHalfBath', 'GarageYrBlt', 'GarageArea', 'GarageCars'の欠損値を0で穴埋め
for col in (
    'BsmtFinSF1', 'BsmtFinSF2', 'BsmtUnfSF',
    'TotalBsmtSF', 'BsmtFullBath', 'BsmtHalfBath',
    'GarageYrBlt', 'GarageArea', 'GarageCars',"MasVnrArea"
    ):
    combined_df[col] = combined_df[col].fillna(0)

# 'MSZoning'の欠損値を最頻値で穴埋め
combined_df['MSZoning'] = combined_df['MSZoning'].fillna(combined_df['MSZoning'].mode()[0])
# "Functional"の欠損値を"Typ"で穴埋め
combined_df["Functional"] = combined_df["Functional"].fillna("Typ")

# Utilities'列を削除
combined_df = combined_df.drop(['Utilities'], axis=1)

#  'Electrical', 'KitchenQual', 'Exterior1st', 'Exterior2nd', 'SaleType'の欠損値を最頻値で穴埋め
combined_df['Electrical'] = combined_df['Electrical'].fillna(combined_df['Electrical'].mode()[0])
combined_df['KitchenQual'] = combined_df['KitchenQual'].fillna(combined_df['KitchenQual'].mode()[0])
combined_df['Exterior1st'] = combined_df['Exterior1st'].fillna(combined_df['Exterior1st'].mode()[0])
combined_df['Exterior2nd'] = combined_df['Exterior2nd'].fillna(combined_df['Exterior2nd'].mode()[0])
combined_df['SaleType'] = combined_df['SaleType'].fillna(combined_df['SaleType'].mode()[0])

#  'MSSubClass'の欠損値を”None”で穴埋めして、データ型をstr型に変更
combined_df['MSSubClass'] = combined_df['MSSubClass'].fillna("None")

#  'OverallCond', 'YrSold', 'MoSold'のデータ型をstr型に変更
combined_df['OverallCond'] = combined_df['OverallCond'].astype(str)
combined_df['YrSold'] = combined_df['YrSold'].astype(str)
combined_df['MoSold'] = combined_df['MoSold'].astype(str)
```

　簡単にざっとしているように見えますが、この欠損値の扱いによって、今後のモデルの精度に大きく影響します。だいぶ時間がかかりました。
途中で練習がてら、for構文やlambda構文を使ってみました。
ひとつひとつ変更するほうが作業自体は楽なのですが、
こうしてコードを書くたびに１つでも新しいコードに触れたいですね。




　次に、カテゴリ変数に対して、ラベルエンコーディングをします。
ラベルエンコーディングとは、「カテゴリ変数を数値化する方法」です。
たとえば、Red,Blue,Yellowという3つのカテゴリをもつ変数に対して、Redは0, Blueは1,Yellowは2というような変換をします。

ラベルエンコーディングの実装はscikit-learnのLabelEncoderが便利です。


```ruby:ラベルエンコーティング.py
#ラベルエンコーディング(カテゴリ変数の数値化)

from sklearn.preprocessing import LabelEncoder

mapping5_0 = {'Ex': 5, 'Gd': 4, 'TA': 3, 'Fa': 2, 'Po': 1, 'None': 0}
cols_5_0 = ('ExterQual', 'ExterCond','HeatingQC', 'PoolQC', 'KitchenQual')
for c in cols_5_0:
    combined_df[c] = combined_df[c].map(mapping5_0)

mapping_7 = {'GLQ': 6, 'ALQ': 5, 'BLQ': 4, 'Rec': 3, 'LwQ': 2, 'Unf': 1, 'None': 0}
cols_7 = ('BsmtFinType1', 'BsmtFinType2')
for c in cols_7:
    combined_df[c] = combined_df[c].map(mapping_7)

mapping_8 = {'Typ': 8, 'Min1': 7, 'Min2': 6, 'Mod': 5, 'Maj1': 4, 'Maj2': 3, 'Sev': 2, 'Sal': 1, 'None': 0}
combined_df['Functional'] = combined_df['Functional'].map(mapping_8)

mapping_5_1 = {'GdPrv': 4, 'MnPrv': 3, 'GdWo': 2, 'MnWw': 1, 'None': 0}
combined_df['Fence'] = combined_df['Fence'].map(mapping_5_1)


mapping_5_2 = {'Gd': 4, 'Av': 3, 'Mn': 2, 'No': 1, 'None': 0}
combined_df['BsmtExposure'] = combined_df['BsmtExposure'].map(mapping_5_2)

mapping_4_0 = {'Fin': 3, 'RFn': 2, 'Unf': 1, 'None': 0}
combined_df['GarageFinish'] = combined_df['GarageFinish'].map(mapping_4_0)

mapping_4_1 = {'Reg': 4, 'IR1': 3, 'IR2': 2, 'IR3': 1, 'None': 0}
combined_df['LotShape'] = combined_df['LotShape'].map(mapping_4_1)

mapping_3_0 = {'Gtl': 3, 'Mod': 2, 'Sev': 1, 'None': 0}
combined_df['LandSlope'] = combined_df['LandSlope'].map(mapping_3_0)

mapping_3_1 = {'Y': 3, 'P': 2, 'N': 1, 'None': 0}
combined_df['PavedDrive'] = combined_df['PavedDrive'].map(mapping_3_1)

mapping_3_2 = {'Grvl': 3, 'Pave': 2, 'N': 1, 'None': 0}
combined_df['Alley'] = combined_df['Alley'].map(mapping_3_2)

mapping_2_0 = {'Grvl': 2, 'Pave': 1, 'None': 0}
combined_df['Street'] = combined_df['Street'].map(mapping_2_0)

mapping_2_1 = {'Y': 2, 'N': 1, 'None': 0}
combined_df['CentralAir'] = combined_df['CentralAir'].map(mapping_2_1)

cols = ('MSSubClass', 'OverallCond', 'YrSold', 'MoSold','FireplaceQu', 'BsmtQual', 'BsmtCond', 'GarageQual', 'GarageCond')
for c in cols:
    lbl = LabelEncoder()
    lbl.fit(list(combined_df[c].values))
    combined_df[c] = lbl.transform(list(combined_df[c].values))
```

以上ですべての特徴量の欠損値を変換できました。
最後に欠損値がなくなったかを確認します。
モデル学習の際に、説明変数に欠損値があるとエラーが起きるからです。


```ruby:欠損値の確認.py
pd.set_option('display.max_rows', 500)
# combined_dfの特徴量について、説明変数の欠損値がすべてなくなっていることを確認
#モデル学習のさいに欠損値があるとエラーで実行できない。
combined_df.isnull().sum()

>>>出力結果
```

<details><summary>出力結果（データが多いため折りたたんでいます。）</summary>
MSSubClass          0
MSZoning            0
LotFrontage         0
LotArea             0
Street              0
Alley               0
LotShape            0
LandContour         0
LotConfig           0
LandSlope           0
Neighborhood        0
Condition1          0
Condition2          0
BldgType            0
HouseStyle          0
OverallQual         0
OverallCond         0
YearBuilt           0
YearRemodAdd        0
RoofStyle           0
RoofMatl            0
Exterior1st         0
Exterior2nd         0
MasVnrType          0
MasVnrArea          0
ExterQual           0
ExterCond           0
Foundation          0
BsmtQual            0
BsmtCond            0
BsmtExposure        0
BsmtFinType1        0
BsmtFinSF1          0
BsmtFinType2        0
BsmtFinSF2          0
BsmtUnfSF           0
TotalBsmtSF         0
Heating             0
HeatingQC           0
CentralAir          0
Electrical          0
1stFlrSF            0
2ndFlrSF            0
LowQualFinSF        0
GrLivArea           0
BsmtFullBath        0
BsmtHalfBath        0
FullBath            0
HalfBath            0
BedroomAbvGr        0
KitchenAbvGr        0
KitchenQual         0
TotRmsAbvGrd        0
Functional          0
Fireplaces          0
FireplaceQu         0
GarageType          0
GarageYrBlt         0
GarageFinish        0
GarageCars          0
GarageArea          0
GarageQual          0
GarageCond          0
PavedDrive          0
WoodDeckSF          0
OpenPorchSF         0
EnclosedPorch       0
3SsnPorch           0
ScreenPorch         0
PoolArea            0
PoolQC              0
Fence               0
MiscFeature         0
MiscVal             0
MoSold              0
YrSold              0
SaleType            0
SaleCondition       0
SalePrice        1459
dtype: int64
</details>

目的変数のSalesPrice以外、欠損値がなくなっていることが確認できました。
データの前処理はまだ続きます。
ここでより最後のモデル精度を上げるため、説明変数を追加します。
主に既存のカラムを組み合わせることで、新たな特徴量となります。
```ruby:説明変数の追加.py
combined_df['YrBltAndRemod']=combined_df['YearBuilt']+combined_df['YearRemodAdd']
combined_df['TotalSF']=combined_df['TotalBsmtSF'] + combined_df['1stFlrSF'] + combined_df['2ndFlrSF']
combined_df['Total_sqr_footage'] = (combined_df['BsmtFinSF1'] + combined_df['BsmtFinSF2'] +combined_df['1stFlrSF'] + combined_df['2ndFlrSF'])
combined_df['Total_Bathrooms'] = (combined_df['FullBath'] + (0.5 * combined_df['HalfBath']) +combined_df['BsmtFullBath'] + (0.5 * combined_df['BsmtHalfBath']))
combined_df['Total_porch_sf'] = (combined_df['OpenPorchSF'] + combined_df['3SsnPorch'] +combined_df['EnclosedPorch'] + combined_df['ScreenPorch'] +combined_df['WoodDeckSF'])
train_df = combined_df[:len(train_df)]
test_df = combined_df[len(train_df):].drop(columns=['SalePrice'])
```

またカテゴリ変数をモデルに組み込むために必要な操作「ダミー変数化」を行います。
機械学習の前処理によく用いられるようです。

```ruby:ダミー変数化.py
combined_df = pd.get_dummies(combined_df, drop_first=True)
train_df = combined_df[:len(train_df)]
test_df = combined_df[len(train_df):].drop(columns=['SalePrice'])
print(train_df.shape)
print(test_df.shape)

>>>出力結果
(1460, 206)
(1459, 205)
```
最後に、訓練用データとテスト用データの行数を表示しています。
テスト用は目的変数のSalesPriceのカラムを削除しています。

データの前処理は以上です。


# 4.評価用関数の作成

　ここから機械学習モデルを準備していきます。
本コンペの予測力を評価する指標はRMSLE(二乗平均平方誤差の対数)です。 
RMSLEはRMSEの対数をとったものです。定義式は下記のようになります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/45d4fff5-3cb6-0df0-0a05-2ea2c613e4f6.png)


実は、最初は対数を取らないRMSE（二乗平均平方誤差）を使用していました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/2ca3cb93-9cb0-e884-aa3f-81f2e829997f.png)

　ただこの先のモデルを評価していく中で、RSMEが５桁以上と非常に大きな値となり、誤差を測る関数として機能を果たせませんでした。
これは、特徴量のなかに外れ値が存在しているからだと考えます。

　RMSEは直感的にもわかりやすく人気があるようですが、
今回のように「外れ値に値が引きずられやすい」という欠点があるようです。
確かに定義式を確認しても２桁ばかりのデータの中に「1,000を900」と予測する誤差が1つあった場合だけで、
RSMEの値が大きく変わってしまいそうです。
モデルの評価で、モデルの判断やスコアが大きく変わることもあるようですので、大事ですね。



　モデルの作成に戻ります。
今回はkfold法を使用します。 KFold法では、用意した訓練データセットをk個に分割し、
そのうちの1つを評価用データ、残りのk-1個を学習データとして使用します。
そして学習と評価を繰り返して得られるk個のモデルと性能評価から平均性能を算出する、という手法ですね。

ホールドアウト法ではデータの分割方法によって偶然分布の偏りが生まれてしまい正確な評価を行えない可能性がありますが、KFold法では複数回データを変えて実行することで安定した評価を行うことができます。


```ruby:機械学習ライブラリのインポート.py
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVR, LinearSVR
from sklearn.ensemble import RandomForestRegressor
from sklearn.neural_network import MLPRegressor
from sklearn.linear_model import SGDRegressor
from sklearn.tree import DecisionTreeRegressor

# 評価用関数
from sklearn.metrics import mean_squared_error

# 1. 説明変数X_trainには、SalePriceを除いたtrain_dfを代入
X_train = train_df.drop("SalePrice", axis=1)
# 2. 目的変数Y_trainには、SalePriceのみが入ったtrain_dfを代入
y_train = train_df["SalePrice"]
# 3. X_testには、test_dfを代入
X_test  = test_df
# 4. X_train、Y_train、X_testの行数、列数を出力
print(X_train.shape, y_train.shape, X_test.shape)
```

```ruby:データの分割.py
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
# 訓練データと検証データに分割
X_trn, X_val, y_trn, y_val = train_test_split(X_train, y_train, test_size=0.33, random_state=42)
```

```ruby:kfold法の実装.py
from sklearn.model_selection import KFold
def run_cv(model):
    cv = KFold(n_splits=3, random_state=42, shuffle=True)
    rmsle_results = []
    models = []
    # 各foldごとに学習データのインデックスと評価データのインデックスが得られる
    for trn_index, val_index in cv.split(X_train):
        X_trn, X_val = X_train.loc[trn_index], X_train.loc[val_index]
        y_trn, y_val = y_train[trn_index], y_train[val_index]
        # モデルの学習
        model.fit(X_trn, y_trn)
        pred = model.predict(X_val)
        # モデルの精度をRSMLEで算出 
        rmsle = np.sqrt(mean_squared_log_error(y_val, pred))
        print("RMSlE:", rmsle)
        rmsle_results.append(rmsle)
        models.append(model)

    print(rmsle_results)
    print("Average:", np.mean(rmsle_results))
    return models
```


# 5.機械学習モデルの学習・選定 

ここからモデルを学習させていきます。
以下5つのモデルを使用し、上記の評価関数RSMLEの値を比較してみたいと思います。

1\. サポートベクターマシーン
2\. 多層パーセプトロン
3\. 決定木
4\. ランダムフォレスト
5\. LightGBM


```ruby:サポートベクターマシーン.py
#サポートベクターマシーンSVM…分類と回帰を行うアルゴリズム
#カーネル法と呼ばれる手法を使ってデータを非線形から線形へ変換しているため学習にかかる処理時間が短く済む
SVR_models = run_cv(SVR())

>>>出力結果
MSE: 0.41105424257991424
RMSE: 0.41438487295756743
RMSE: 0.3743149873002599
[0.41105424257991424, 0.41438487295756743, 0.3743149873002599]
Average: 0.39991803427924716
```

```ruby:多層パーセプトロン.py
#多層パーセプトロン（ニューラルネットワーク）…分類を行うアルゴリズム
#パーセプトロンを何層にも接続して、非線形分類問題を解けるようになり、回帰問題にも適用可能
MLP_models = run_cv(MLPRegressor())

>>>出力結果
RMSLE: 0.26541167791704723
RMSLE: 0.25534446945547296
RMSLE: 0.2317913827656762
[0.26541167791704723, 0.25534446945547296, 0.2317913827656762]
Average: 0.25084917671273216
```



```ruby:決定木.py
#決定木
#データから抽出したルールが木構造で表現。直感的に分かりやすく人に説明しやすいため、利用しやすいアルゴリズム
DT_models = run_cv(DecisionTreeRegressor())

>>>出力結果
RMSLE: 0.2112894487571786
RMSLE: 0.22729407936639096
RMSLE: 0.19761811990631895
[0.2112894487571786, 0.22729407936639096, 0.19761811990631895]
Average: 0.2120672160099628
```

```ruby:ランダムフォレスト.py
#ランダムフォレスト
#アンサンブル学習法（複数の分類器を集めて構成される識別器）の一つで、多数の決定木を構築
RF_models = run_cv(RandomForestRegressor())

>>>出力結果
RMSLE: 0.1469047138832375
RMSLE: 0.17414312372848556
RMSLE: 0.12922405093342998
[0.1469047138832375, 0.17414312372848556, 0.12922405093342998]
Average: 0.150090629515051
```

```ruby:LightGBM.py
#LightGBM
#木構造を用いた勾配ブースティングのフレームワーク
#１つ上の決定木の誤差を次の決定木の目的変数として決定木を構築しつつ、目的変数の最適化に勾配を用いて学習

import lightgbm as lgb

lgb_params = {
    "objective":"regression",
    "metric": "rmse",
    # 再現性確保のために一般的にシードは固定
    "random_state":42
    }

cv = KFold(n_splits=3, random_state=42, shuffle=True)
rmse_results = []
lgbm_models = []

# 検証データの予測値を保存するための配列
test_preds = np.zeros(len(X_test))

for trn_index, val_index in cv.split(X_train, y_train):
    X_trn, X_val = X_train.loc[trn_index], X_train.loc[val_index]
    y_trn, y_val = y_train[trn_index], y_train[val_index]

    train_lgb = lgb.Dataset(X_trn, y_trn)
    validation_lgb = lgb.Dataset(X_val, y_val)
    model = lgb.train(
        lgb_params, train_lgb,
        num_boost_round=1000, valid_sets=[validation_lgb],
        callbacks=[lgb.log_evaluation(period=10),lgb.early_stopping(100)]
        )
    pred = model.predict(X_val)
    rmse = np.sqrt(mean_squared_log_error(y_val, pred))
    print("RMSLE:", rmsle)
    rmsle_results.append(rmsle)
    lgbm_models.append(model)

    # 結果LightGBMの精度が一番良いようなので、LightGBMで予測する準備
    test_preds += model.predict(X_test) / cv.n_splits

print(rmsle_results)
print("Average:", np.mean(rmsle_results))


>>>出力結果
[0.13544007585218848, 0.1640756260799781, 0.12243970894837297]
Average: 0.1406518036268465

```
評価関数RSMLEが一番小さくなるのは、LightGBMでした。
LightGBMで予測した結果を提出したいと思います。


# 6.予測結果・提出


```ruby:予測結果の提出.py
# サンプル提出ファイルの予測値列の値を変えるだけで提出
submission = pd.read_csv('../input/house-prices-advanced-regression-techniques/sample_submission.csv')
submission['SalePrice']  = test_preds
display(submission.head(10))
# 提出ファイルを出力
submission.to_csv("first_submission.csv", index=False)
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/a97a61f2-5439-f790-77c6-0143826c1235.png)


さあ、結果はどうなのか…。

# 7.結果・考察

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/5081564a-1d27-dfa1-7410-06695a51b532.png)

スコア：0.13663
順位：1471/4659 （上位32%）
*2024/5/14時点

となりました。
「案外良いのでは？！」と思いました。

実はこの後さらにスコアが上がるよう、
前処理の方法を変えたり、説明変数を追加したり、LightGBMのパラメータを変えたり、「9.補足」で解説しているようなことを追加したり…いろいろしたのですが、初回提出のスコアが一番良かったです。
やればやるほど改善するというわけではないものの、スコアを上げるためにはTry&Errorを続けることが大事だと改めて痛感しました。
　また今回の分析は、データの前処理にかなり時間がかかってしまっています。提出に至るまでの工程を１００とすると、データの前処理だけに６０くらい使った感覚です。これも実際の職場や企業ではあるあるではないかなと思いました。どうしてもモデルをいじっているときは楽しいのですが、そこに行き着くまでの地道な作業があってこそですね。
 
 １つ、今回の分析で個人的にファインプレーだと思っているのは「評価用関数RSMLEを使用できたこと」です。RSMEの場合、なかなか参考にならず、どうしようか…とかなり焦りました。いろんなqiitaの投稿や諸先輩のブログを拝見し、RSMLEを使用することで上手く分析を進めることができました。
 エンジニアにはリサーチ能力が必須！とよく聞きますが、その通りだと実感しました。
 
 


　ただ、ここで注意。
kaggleのようなコンペでは、Public Leaderboardのスコアを信用しすぎないことが大事のようです。
ほとんどのコンペティションでは検証データの一部の成績のみがスコアとして公開されて、最終的な成績は残りの検証データのスコアで決まります。
そのためこのPublicLBのスコアを上げることに熱中しすぎると、
検証データの一部に過学習してしまい、結果的に残りの検証データに対する予測がうまくいかないといったことも多くあります。
あくまでモデルを評価するのは、自分で構築した評価手法(CV)で行うのが良いとされています。
もちろんPublicLBのスコアも自分のCVのスコアと比べてみたりすることで参考になることも多いです。提出してスコアや順位が上がることはコンペティションを続けるモチベーションの１つにもなりますね。


# 8.補足：精度向上のために追加で実践したこと

スコアを上げるために工夫したことを２つ紹介します。
ただどちらもスコアは上がらなかったため、こちらは提出しておりません。

```ruby:アンサンブル学習.py
from lightgbm import LGBMRegressor
from xgboost import XGBRegressor

# ホールドアウト法で評価
X_trn, X_val, y_trn, y_val = train_test_split(X_train, y_train, test_size=0.33, random_state=42)

models = [LGBMRegressor(),XGBRegressor(),RandomForestRegressor()]
predictions = []
for model in models:
    model.fit(X_trn, y_trn)
    # 各クラスに属する確率を予測
    pred = model.predict(X_val)
    # 各モデルの精度も表示
    print(f"{model.__class__.__name__}:", np.sqrt(mean_squared_error(y_val, pred)))
    predictions.append(pred)

# 3つのモデルの平均値をとる
mean_emsemble_proba = np.mean(predictions, axis=0)

print("\nmean emesemble RMSE", np.sqrt(mean_squared_error(y_val, mean_emsemble_proba)))
```


```ruby:スタッキング.py
# ホールドアウト法で評価
X_train_hold, X_val_hold, y_train_hold, y_val_hold = train_test_split(X_train, y_train, test_size=0.33, random_state=42)

X_train_hold = X_train_hold.reset_index(drop=True)
X_val_hold = X_val_hold.reset_index(drop=True)
y_train_hold = y_train_hold.reset_index(drop=True)
y_val_hold = y_val_hold.reset_index(drop=True)

# oofの方法を用いて学習データの予測値を取得
models = [LGBMRegressor(),XGBRegressor(),RandomForestRegressor()]
predictions = []
for model in models:
    cv = KFold(n_splits=3, random_state=42, shuffle=True)
    # 最初に、各foldのoofを保存するための配列を作成
    oof_predicts_by_model = np.zeros(len(y_train_hold))
    for trn_index, val_index in cv.split(X_train_hold):
        X_trn, X_val = X_train_hold.loc[trn_index], X_train_hold.loc[val_index]
        y_trn, y_val = y_train_hold[trn_index], y_train_hold[val_index]
        model.fit(X_trn, y_trn)
        # oof_predictionsのうち、検証データのインデックスの部分だけ予測して保存
        oof_predicts_by_model[val_index] = model.predict(X_val)
    predictions.append(oof_predicts_by_model)
# 学習データの予測値。2層目の学習データとして用いる
oof_train = pd.DataFrame({"lgbm_prediction":predictions[0], "xg_prediction":predictions[1], "rf_prediction":predictions[2]})
display(oof_train)



# 検証データの予測値を取得
predictions_test = []
for model in models:
    model.fit(X_train_hold, y_train_hold)
    predict_test = model.predict(X_val_hold)
    predictions_test.append(predict_test)

# 検証データの予測値。2層目の予測データとして用いる
oof_test = pd.DataFrame({"lgbm_prediction":predictions_test[0], "xg_prediction":predictions_test[1], "rf_prediction":predictions_test[2]})
display(oof_test)


# 2層目はLightGBMで予測
lgbm_model_2nd = LGBMRegressor()
lgbm_model_2nd.fit(oof_train, y_train_hold)
predict_stacking = lgbm_model_2nd.predict(oof_test)
score = np.sqrt(mean_squared_error(y_val_hold, predict_stacking))
print("stacking RMSE:", score)
```
