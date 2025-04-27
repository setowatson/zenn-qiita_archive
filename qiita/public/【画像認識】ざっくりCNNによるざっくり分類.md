---
title: 【画像認識】ざっくりCNNによるざっくり分類
tags:
  - 初心者
  - Keras
  - CNN
  - 転移学習
  - VGG16
private: false
updated_at: '2024-06-12T07:18:21+09:00'
id: ffa9c60023965fe14e6c
organization_url_name: null
slide: false
ignorePublish: false
---
# 1.CNNのざっくり概要

CNNとは、Convolutional Neural Networkの略。
日本語で言うと、「畳み込みニューラルネットワーク」、
人間の脳の視覚野と似た構造を持つ 「畳み込み層」 という層を使い、特徴の抽出を行うニューラルネットワークです。

全結合層のみのニューラルネットワークと比べ、画像認識などの分野でより高い性能を発揮します。
今や当たり前となっているスマホの顔認証システムや、工場での不良品検出などにも使用されている技術です。

また多くの場合、畳み込み層と共に 「プーリング層」 という層が使われます。 
プーリング層で畳み込み層から得た情報を縮約し、最終的に画像の分類などを行います。

下の画像は、CNNの一例です。
左の写真をCNNを用いて、「Car」だと分類しています。



![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/5e226198-845f-4842-27ae-54b219556588.png)


出典:CS231n: Deep Learning for Computer Vision

https://cs231n.stanford.edu/


畳み込み層とは？プーリング層とは？などは本投稿では省略し、CNNの実装に移ります。
CNNの詳しい解説は、以下の記事がわかりやすいです。

https://qiita.com/icoxfog417/items/5fd55fad152231d706c2


# 2.かなり初歩的なCNNの実装


## 2-1. 使用するライブラリ
今回はTensorFlowからKerasライブラリを使って、かな〜り初歩的なCNNを実装します。

Kerasは機械学習ライブラリの上で非常に使い勝手の良いライブラリです。
QiitaにもKerasに関する記事は山ほどありますが、「Kerasって何？」という方にとってはこちらがわかりやすいかと思います。

https://tech-forward.io/magazine/keras#index_id1

私はTensorFlowとの違いがあまりわかっていなかったので有り難かったです。
いくら人気とは言え、実務上ではまだTensorFlowを使われていることも多いようです。
Tensorflowの一番いいところは「情報量」と下記記事でも解説されています。

https://qiita.com/tawago/items/c977c79b76c5979874e8


（こちらの記事、同じ初心者の立場でこんなに書けるのすごい。）


## 2-2. 使用するデータセット 
CIFER-10(サイファー・テン)というデータセットを使用し、画像の分類問題に取り組みます。CIFAR-10とは、下の写真のように10種類のオブジェクトが映った画像のデータセットです。
各画像はサイズが32ピクセル×32ピクセルで3チャンネル（R,G,B）のデータとなっており、airplane,automobileなどオブジェクトごとにそれぞれ0~9のクラスラベルがつけられています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/714786aa-a5a6-8e27-1d1a-18639dcd4ffa.png)


https://www.cs.toronto.edu/~kriz/cifar.html


## 2-3. 実装コード

まず必要なライブラリ・データセットをインポートしていきます。
描画用のために、numpy,matplotlibも忘れずに。

```ruby:ライブラリのインポート.py
from tensorflow import keras
from tensorflow.keras.datasets import cifar10
from tensorflow.keras.layers import Activation, Conv2D, Dense, Dropout, Flatten, MaxPooling2D
from tensorflow.keras.models import Sequential, load_model
from tensorflow.keras.utils import to_categorical
import numpy as np
import matplotlib.pyplot as plt
```


次にデータをロードしていきます。
訓練データとテストデータに分けますが、ここでデータを多くしすぎると時間が大量にかかるためまずはこの程度で実装…

```ruby:データロード.py
(X_train, y_train), (X_test, y_test) = cifar10.load_data()

# 今回は全データのうち、学習には300、検証には100個のデータを使用
X_train = X_train[:300]
X_test = X_test[:100]
y_train = to_categorical(y_train)[:300]
y_test = to_categorical(y_test)[:100]
```



 TensorFlowでは、まずモデルを管理する インスタンス を作り、addメソッドで層を一層ずつ定義していきます。

Denseは全結合層、Conv2Dは畳み込み層、MaxPooling2Dはプーリング層を追加しています。
各層のパラメーターは、抽出する種類の数やカーネル(畳み込みに使用する重み行列)の大きさを指定しています。基本的にデフォルト設定で進め、詳しい解説は省略します。



```ruby:モデルの定義.py
#インスタンスの作成
model = Sequential()
#層の追加
model.add(Conv2D(32, (3, 3), padding='same',
                 input_shape=X_train.shape[1:]))
model.add(Activation('relu'))
model.add(Conv2D(32, (3, 3)))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))

model.add(Conv2D(64, (3, 3), padding='same'))
model.add(Activation('relu'))
model.add(Conv2D(64, (3, 3)))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))

model.add(Flatten())
model.add(Dense(512))
model.add(Activation('relu'))
model.add(Dropout(0.5))
model.add(Dense(10))
model.add(Activation('softmax'))

```

オプティマイザと損失関数を設定してモデルをコンパイル…コンピュータが実行可能な形式に変換していきます。
損失関数などいろいろな種類があるようですが、初学者のうちは代表的なものを使っていく感じで大丈夫かなと思っています。
「lossを使って良い感じの誤差が出てoptimizerで良い感じに重みを更新してくれるのだろうな」程度の理解です、間違っているかもしれません。

```ruby:コンパイル.py
opt = keras.optimizers.RMSprop(learning_rate=0.0001)
model.compile(loss='categorical_crossentropy',
              optimizer=opt,
              metrics=['accuracy'])
```
次にモデルの学習を進めます。
後ほど言及するのですが、「バッチサイズ(batch_size)」と「エポック数(epocks)」が非常に重要です。
バッチサイズとは「モデルへ一度に入力するデータの数」、エポック数とは「繰り返しする学習の回数」、CNNだけでなくディープラーニングは全般的にこの２つによって結果が大きく異なります。
必ずしも大きいと良いということは無く、バッチサイズが大きすぎると全体のデータへの最適化が行われなくなる状態（局所解）から抜け出せなくなってしまったり、エポック数が大きすぎると学習を繰り返すことで損失関数を最小化させようとして過学習(過剰適合)を引き起こしてしまう可能性があります。
データとモデルごとに適切なバッチサイズ・エポック数の設定をする必要があります。

この２つを適切な値にするためにも学習の進捗を可視化したい…そのためにモデル学習のfit関数に、historyを定義しておきます。

```ruby:モデルの学習.py
history = model.fit(X_train, y_train, batch_size=100, epochs=10, validation_data=(X_test, y_test)) 
```

次にモデルの精度を評価するために、「正答率」と「損失関数の出力値」を表示させます。
損失関数とは、機械学習モデルが算出した予測値と実際の正解値の「ズレ」を計算するための関数。
出力される損失は、実際の正解値と予測値の距離や差の総量や平均とみなすことができます。
正解率だけを指標に評価することも可能ですが、それだけでは個々の出力データの正解・不正解といった細かい結果までは分かりません。そのため、損失関数の出力値も計算します。


```ruby:精度評価.py
scores = model.evaluate(X_test, y_test, verbose=1)
# 損失関数による損失の出力
print('Test loss:', scores[0])
# 正答率の出力
print('Test accuracy:', scores[1])
```

いよいよテストデータを用いて、実際に分類予測をしていきます。

```ruby:最初の10枚の予測.py
pred = np.argmax(model.predict(X_test[0:10]), axis=1)
print(pred)
#モデル構造の表を出力
model.summary()

```

上記で述べたように、エポック数による正答率と損失の変化を見ていきます。
モデル学習時にhistoryを用いていたのはこのときのためです。

```ruby:学習状況の表示.py
# 学習の経過を出力
print("Train Accuracy per epoch:", history.history['accuracy'])
print("Validation Accuracy per epoch:", history.history['val_accuracy'])
print("Train Loss per epoch:", history.history['loss'])
print("Validation Loss per epoch:", history.history['val_loss'])

# 学習の経過を出力・プロット
plt.figure(figsize=(12, 4))

# loss
plt.subplot(1, 2, 2)
plt.plot(history.history['loss'], label='Train Loss')
plt.plot(history.history['val_loss'], label='Validation Loss')
plt.legend()
plt.title('Epoch vs Loss')
plt.xlabel('Epochs')
plt.ylabel('Loss')

# accuracy
plt.subplot(1, 2, 1)
plt.plot(history.history['accuracy'], label='Train Accuracy')
plt.plot(history.history['val_accuracy'], label='Validation Accuracy')
plt.legend()
plt.title('Epoch vs Accuracy')
plt.xlabel('Epochs')
plt.ylabel('Accuracy')

plt.tight_layout()
plt.show()
```

以上を実行してみます。

```ruby:
出力結果>>
Test loss: 2.6474132537841797
Test accuracy: 0.20000000298023224
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/6e9d15f8-e2b7-e856-7de1-0dcf552dc83e.png)


実行するたびに数％変わるのですが、１度目の結果は正答率20％。
以前の私だったら嘆いて終わりでしたが、
最近ここで嘆くのは２流だと学びました。ここからが分析の本番。
ちなみにエポック数は4程度で十分なようですね。これ以上すると過学習になる恐れがありそうです。


# 3.正答率をあげるための修正　
上記の分類問題の正答率を上げるためにできそうなことを考えていきます。

- データ数を増やす
- エポック数を変更する
- バッチサイズを変更する
- 層を増やす
- パラメータを変更する
- 転移学習する

まず１番に考えられるのは「データ量を増やす」
上記の分類問題において使用したのは、何万とあるCIFER10の画像集のうち、学習では300、検証では100だけ。学習時間やデータ量を気にして少なくしていましたが、ここはやはりかなり大きな問題。
最後にテストしているのも１０枚だけです。ここはなんとなく予想していましたが、要改善。
次に手取り早くできることが、「エポック数・バッチサイズの修正」
特に今回はバッチサイズを100でやってみましたが、もともとデータサイズのセットも大きいので、バッチサイズを1000くらいにしてもよいかもしれません。ちなみにこれはやっていくなかで知ったのですが、バッチサイズは2の累乗でされることが多いようです。512,1024など。


## 3-1. 転移学習とは。VGG16とは。　
色々と試せることはありますが、上記で書いたような「〇〇を増やす」というやり方は、データや処理時間が大量に必要です。
そこで今回はより確実な方法を１つ発見。
それが「転移学習」
すでに公開されている学習済みのモデルを使って新たなモデルの学習を行うこと、です。

公開されているモデルには何種類かありますが、ここでは「VGG16」というモデルを例に説明します。

VGGモデルは、2014年のILSVRCという大規模な画像認識のコンペティションで2位になったオックスフォード大学VGG(Visual Geometry Group)チームが作成したネットワークモデルです。小さいフィルターを使った畳み込みを2〜4回連続で行いさらにプーリングする、というのを繰り返し、当時としてはかなり層を深くしてあるのが特徴です。
VGGモデルには、重みがある層(畳み込み層と全結合層)を16層重ねたものと19層重ねたものがあり、それぞれVGG16やVGG19と呼ばれます。VGG16は、畳み込み13層＋全結合層3層＝16層のニューラルネットワークになっています。


## 3-2. VGG16の転移学習の実装　

それではVGG16転移学習のコードを実装していきます。

```ruby:ライブラリのインポート〜データロード.py
from tensorflow import keras
from tensorflow.keras.datasets import cifar10
from tensorflow.keras.layers import Activation, Conv2D, Dense, Dropout, Flatten, MaxPooling2D
from tensorflow.keras.models import Sequential, load_model
from tensorflow.keras.utils import to_categorical
import numpy as np
import matplotlib.pyplot as plt

# データのロード
(X_train, y_train), (X_test, y_test) = cifar10.load_data()
```
ここまでは同じ。
次にinput_tensorの定義をして、vggのImageNetによる学習済みモデルを作成します。
入力は(32,32,3)である必要はないが、VGG16に準拠する形で一旦やってみる。（この辺誤っている可能性は高いです）
引数input_tensorにより、学習済みモデルの入力に新たなレイヤーを追加できます。
特徴量を抽出する全結合層も作成しておきます。
このあとVGG16と連結させます。

```ruby:学習済みモデルの作成.py
input_tensor =Input(shape=(32, 32, 3))
vgg16 = VGG16(include_top=False, weights='imagenet', input_tensor=input_tensor)
# 特徴量抽出部分のモデル（全結合層）も作成
top_model = Sequential()
top_model.add(Flatten(input_shape=vgg16.output_shape[1:]))
top_model.add(Dense(256, activation='sigmoid'))
top_model.add(Dropout(0.5))
top_model.add(Dense(10, activation='softmax'))
```

結合させる際、VGG16から転移させた15層目までは学習させないので、重みの更新を15層目までで止めておきます。

```ruby:VGG16と全結合層の結合.py
model =Model(inputs=vgg16.input, outputs=top_model(vgg16.output))
# 15層目までの重みを固定
for layer in model.layers[:15]:
    layer.trainable = False
```

ここまできたら残りは上記2と同じ流れ。
```ruby:コンパイル〜学習〜評価.py
#　学習の前に、モデル構造を確認
model.summary()

# コンパイル
model.compile(loss='categorical_crossentropy',
              optimizer=optimizers.SGD(learning_rate=1e-4, momentum=0.9),
              metrics=['accuracy'])


# バッチサイズ32、エポック数は3で学習
history = model.fit(X_train, y_train,validation_data=(X_test, y_test), batch_size=32, epochs=3 ) 


# 精度の評価
scores = model.evaluate(X_test, y_test, verbose=1)
print('Test loss:', scores[0])
print('Test accuracy:', scores[1])

# 学習状況の表示（Epoch vs Accuracy, Epoch vs Loss)

# 学習の経過を出力
print("Train Accuracy per epoch:", history.history['accuracy'])
print("Validation Accuracy per epoch:", history.history['val_accuracy'])
print("Train Loss per epoch:", history.history['loss'])
print("Validation Loss per epoch:", history.history['val_loss'])


# 学習の経過を出力・プロット
plt.figure(figsize=(12, 4))

# accuracy
plt.subplot(1, 2, 1)
plt.plot(history.history['accuracy'], label='Train Accuracy')
plt.plot(history.history['val_accuracy'], label='Validation Accuracy')
plt.legend()
plt.title('Epoch vs Accuracy')
plt.xlabel('Epochs')
plt.ylabel('Accuracy')

# loss
plt.subplot(1, 2, 2)
plt.plot(history.history['loss'], label='Train Loss')
plt.plot(history.history['val_loss'], label='Validation Loss')
plt.legend()
plt.title('Epoch vs Loss')
plt.xlabel('Epochs')
plt.ylabel('Loss')

plt.tight_layout()
plt.show()
```

```ruby:
出力結果>>
Test loss: 0.8562871813774109
Test accuracy: 0.7057999968528748
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/1899c3eb-aaae-4cdd-ccd6-4ab5fc708bff.png)


以上が今回の結果です。
転移学習を用いた結果、精度が70.5％まで上がりました。
学習の進捗を見ると、エポック数が1を超えたあたりから過学習ぎみになっていることがわかります。
これは転移学習ならではの動きであることが予想できます。
損失関数も学習が進むにつれて、０に近づいています。
0.85とまだ小さくなる余地はありそうですが、転移学習を使う前の2.6よりはかなりよくなりました。



# 4. 今回の感想・考察
CNNによる画像の分類問題を、ざっくりと体験できた。
飛行機・車・鳥・犬…などこれらを分類するのにこれだけ難しいのだから、顔認証システムなんてどうなっているのだろう？と思ってしまいます。

https://aismiley.co.jp/ai_news/detect-faces-from-an-image/


簡単そうに解説されていますがここには上記畳み込みニューラルネットワークの技術がつまっているわけですね。社会への影響も本当にすごい。

また、正答率を上げるための思考として、選択肢はいくつかありますが、そのなかでも
「転移学習」はかなり強力な武器。
賢い先人の努力を引用することができる。なんだかかっこいいと思いませんか。
kaggleをいくつか漁るだけでも転移学習はさまざまなところで使われいることがわかります。
エンジニアにはリサーチ能力いわゆる”ググる力”が必要とよく聞くが、これもその1つかもしれないと感じました。

# ほかに参考になりそうな記事

同じcifer-10で精度86％を出したkaggleでもゴールドを獲得しているコードです。
転移学習を使用していることがわかります。

https://www.kaggle.com/code/muhammadibrahimqasmi/cifar-10-cnn-image-classification


