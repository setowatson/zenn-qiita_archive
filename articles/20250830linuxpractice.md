---
title: "Linuxを安全に学び始めるための準備（VirtualBox＆Ubuntu）"
emoji: "🌱"
type: "tech"
topics: ["linux", "ubuntu", "virtualbox","zennfes2025free"]
published: true
publication_name: "headwaters"
---
# Linuxを学ぶ価値

この記事では、VirtualBoxとUbuntuを使って安全な仮想環境を構築し、
Linuxの基礎を **楽しく&安全に** 学ぶ方法を紹介します。

Linuxは、世界中のサーバーやクラウド、スマートフォン（Android）などの基盤として使われているOSです。以下のような理由から、IT実務での価値は非常に高いです。

- 開発環境としての柔軟性（Python, Node.js, Dockerなど）
- セキュリティ・ネットワークの学習に最適
- サーバー構築や運用の基礎スキル
- 世界中の技術者が使っているオープンソース文化

Linuxを知っているだけで、開発・インフラ・セキュリティなど、複数の分野に対応できるようになります。
コンサルタントとして基本的には上流工程に携わっている身ですが、
Linuxの基本くらいは知っておくべきだと思い、今回色々と触ってみてます。
同じような立場の方の参考になれば嬉しいです。

参考：
https://dad-union.com/what-is-linux-beginners-guide


# BanditでLinuxの基礎を学ぶ

OverTheWireの「Bandit」は、Linuxの基本操作をゲーム感覚で学べる最高の教材です。
Level0～34のクイズ形式教材があり、徐々に使用するコマンドや考え方の難易度が上がります。
ちなみにBanditは”強盗・盗賊”という意味です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/af15c4d4-5a3e-44fc-97d9-456ca4894ace.png)


https://overthewire.org/wargames/bandit/ 



## Banditの特徴

- SSHでログインして、各レベルの課題に挑戦できる！
- `ls`, `cat`, `cd` などの基本コマンドを使ってファイルを探索する！
- レベルが上がるごとに、ファイル操作・検索・パーミッションなどの知識が身につく！

Banditをプレイすることで、実際の業務に近い形でLinuxの操作を体験できます。
初学者はとりあえずレベル20くらいを目安に頑張るといいようです。

例えば、Level0は指定されたパスワードを入力するだけです。
（本記事では、Banditの具体的な攻略やLinuxコマンドについては記述しません）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/db8a1118-9c9c-44d2-8f52-5eee01797fb7.png)
*Bandit Level0のプレイイメージ。一部上から白モザイク掛けてます。*



# VirtualBox + Ubuntuで学習環境を構築

Linux学習で最初にぶつかる壁は「環境の準備」です。
もちろんWindowsでも学習を進められますが、Banditのような教材はターミナルからのUnixコマンド操作を前提としているため、後々不便さを感じることがあります。

その点、VirtualBox上にUbuntu環境を構築しておけば、PCの設定や環境設定を汚すことなく、安全にLinuxを試すことができます。

## 手順概要

### 1. VirtualBoxをインストール（公式サイトからダウンロード）

https://www.virtualbox.org/wiki/Downloads


### 2. Ubuntu Desktop 24.04 LTSのISOをダウンロード

https://jp.ubuntu.com/download

### 3. VirtualBoxで新規仮想マシンを作成

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/6586a715-4b57-4c5a-b23b-eb6db85db8b7.png)


### 4. ストレージ設定でISOファイルをマウント

ディスクマークを押して先ほどダウンロードしたubuntuファイルを選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/22329a4e-07e2-4dae-bfbb-235ee54b8aec.png)

その他の設定などはこちら

| カテゴリ | 項目 | 設定内容 |
|----------|------|----------|
| システム | メモリ | 4096 MB以上推奨 |
|  | CPUコア数 | 2コア以上推奨 |
|  | 起動順序 | ハードディスク → 光学ドライブ（ISO除去後） |
|  | アクセラレーション | ネステッドページング、KVM準仮想化 |
| ストレージ | IDE | ISOファイル（インストール時のみ使用） |
|  | SATA | ubuntu.vdi（25 GB） |
|  | ホストI/Oキャッシュ | 有効化推奨 |
|  ディスプレイ | グラフィックコントローラー | VMSVGA |
|  | ビデオメモリ | 64 MB以上推奨 |
|  | 3Dアクセラレーション | 無効（必要に応じて有効化） |
| ネットワーク | アダプター | Intel PRO/1000 MT デスクトップ |
| インストール時 | ネットワーク接続 | 今は接続しない |
|  | インストールタイプ | ディスクを消去してUbuntuをインストール |
|  | ソフトウェア選択 | 標準インストール or 最小インストール |
|  | アップデートとサードパーティ | チェックなし（後から追加可能） |


### 5. 仮想マシンを起動してUbuntuをインストール

環境が整えば仮想マシンを起動して、Ubuntuをインストールします。
私はこのインストールがなぜか上手く行かず、
再起動を繰り返しながら1時間程度かかってしまいました。
Ubuntuをインストールする際もいくつかセットアップが必要ですが、
ここは自分で調べながらやっていくことで新しい勉強も兼ねられると思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3780099/316cc81a-e321-4a9a-89a1-b021dfaedd71.png)

ここまできたらあとはBanditの内容に従い、Linuxコマンドを調べながら実践していく…ということが可能です！

# まとめ

VirtualBoxとUbuntuを使えば、誰でも安全にLinuxを学べます。
特にBanditのような学習ゲームを活用することで、楽しみながらスキルアップが可能です。

「Linuxを学びたいけど、何から始めればいいかわからない」という方は、ぜひこの方法を試してみてください。仮想環境なら失敗してもすぐにやり直せるので、安心して学習を進められます。

ぜひ一緒に学んでいきましょう！
