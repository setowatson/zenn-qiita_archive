---
title: 【Markdown AI】画像生成機能で背景を作成してみた。
tags:
  - CSS
  - 画像生成
  - MarkdownAI
private: false
updated_at: '2025-01-09T07:12:16+09:00'
id: 5fd1dc70b091d8ed5485
organization_url_name: null
slide: false
ignorePublish: false
---
# 完成品


https://mdown.ai/content/a2716694-a8cb-4cc6-90a2-3e70ace9004e

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/0ead7a8c-8958-5d38-473f-7b44f948aca3.png)

背景がめっちゃおしゃれになった…。

## 画像リンク

https://storage.googleapis.com/topdowncom/content/0WfMbNk0hPTkxRHxUzZN92jFE0J3/3c6abada-c804-4ec9-8ad2-f6ffccc9b661/cf868bd6-5675-441d-a94f-a688f3c78bcf

## 元のサイト・機能はこちらで投稿しています。

本投稿ではMarkdown AIの画像生成機能のみに注目します。
作成したサイトや詳細のコードについては以下の投稿をご覧ください。

https://qiita.com/setowatson/items/77086957763b46d9156d

## 本画像を作成したプロンプト

1. Size：square *仮名
1. Art Style：none
1. Background：Sea *仮名
1. Main Content：none
1. Expression：none
1. Pose：none
1. Additional Content 1：none
1. Additional Content 2：none
1. Additional Content 3：none
1. 自由記述部分
・The location is at the entrance of a movie theater.  
・Leave ample space in the background to suit the theme.  
・Use colors, but avoid overly flashy tones.  
（和訳）
・場所は映画館の入り口
・背景にふさわしいよう余白を多めにして
・カラーだけど派手すぎない色使いにして


以下、選択した状態。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/44e75df4-7610-72c8-58d2-8be7a3cadb33.png)

# 画像生成機能の使い方

### 1.上タブから「Insert」を選択

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/d3015621-b195-df80-7a29-6751216e74f3.png)

### 2.2番目「Insert an AI-generated image」を選択
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/e81b7342-e1e7-5d34-51f6-238a3aa6da08.png)

### 3.直感でいろいろ選択

満足いくまで色々試します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/15e898ff-585b-29fe-7918-3cab14722b70.png)

### 4.画像挿入

画像を決めたら「Insert」を押す。
Google storageに画像が保存されたリンクが生成されています。
このままでもいいのでしょうけど、私はリンクだけ使用しています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/e975278d-f378-930d-4409-fe620eb61a17.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/477d42ab-f308-180f-b9a7-42f3dcde971b.png)


# 感想

便利というより「楽しい」です（笑）。
もちろん便利なんですが、ここまで直感的でゲーム感覚で画像を作れるとなると、もはや「遊び」の域ですね。

特にいいなと思ったのは、選択肢だけでなく自由記述欄があること。
このおかげで、「これだけは避けてほしい」「これを必ず含めてほしい」といった具体的な指示が可能になります。

今回、私はサイトの背景画像を作りたかったので、あまりごちゃごちゃしていないデザインを望んでいました。それをプロンプトに入れたところ、しっかり意図を汲んでくれました。


ただ、難しい点もあって、それが **「AIっぽい画像になりがち」** という問題です。
これはMarkdownに限らず、最近のAI画像生成ツール全般に言えることかもしれませんが、なんとなく「AIが作りました感」が気になるんですよね。個人的なこだわりかもしれませんが、そういった印象をいかに抑えるかが重要だと思っています。

↓最初に出力されたのはこんな感じ。AIっぽい…。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/c4e444f0-c7ca-a750-d5b7-bb7e2c8b7be1.png)

いくつか試した結果、個人的におすすめのプロンプトが以下です。

- 「Location」を海や空などの“自然系”に設定する。
- 自由記述欄に「Use colors, but avoid overly flashy tones（和訳：カラーだけど派手すぎない色使いにして）」を加える。

これで、背景にふさわしい画像が生成される確率がぐっと上がると思います。

さらに、Googleクラウドに自動保存して、そのURLだけを出力できる機能もあり、使い勝手が非常に高いのもありがたいポイントです。

## 個人的な経緯

1 . Markdown AIなしで個人でサイトを普通に作る。Qiitaに投稿する。

https://qiita.com/setowatson/items/b942e972701774d08599

2 . 投稿した際のキャンペーンでMarkdown AIキャンペーンを知る。

https://qiita.com/official-events/52830497d1d63301da95

3 . Markdown AI使って、サイトを作成してみる。

私の1つ前の投稿。感動した。

https://qiita.com/setowatson/items/77086957763b46d9156d

4 . Markdown AIのなかに「画像生成AI(ロボットAI)」とやらがあるのを後から知る。

じゃあもう一個作るか〜と思っていたが、上記作成したサイトの背景を作れないかと思いつく（天才）

https://qiita.com/mdown_ai_jpn/items/f1fbbfd7a7b41b814d05

最初、Markdown AIの画像生成機能について読んだときは、「サイトのユーザーが画像を入手するための機能」だと思っていました。
例えば、映画の好みを聞いておすすめ映画を出力するように、指示を出すとそれに当てはまる画像を生成するモデルをサイト内に組み込める…と思っていたのです。

「それなら設計を考えるのが大変そうだな〜」と思っていたのですが、実際に触ってみると、これは「作成側がサイト内に画像を埋め込める機能」でした。

以前作ったサイトの背景画像はフリー素材を探して使っていたので、これは非常に助かりました。
好みに合うフリー素材を探すのもなかなかの労力ですし、著作権触れないかな大丈夫かなという心配もあります。


個人的には人やキャラクターで使うのではなく、背景として使うことで、AI感を少なくかつオシャレなサイトに仕上げることができるのではないかと思います。


# 参考

https://qiita.com/mdown_ai_jpn/items/f1fbbfd7a7b41b814d05
