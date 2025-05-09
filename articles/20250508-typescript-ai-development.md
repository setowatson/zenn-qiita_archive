---
title: "なぜ生成AIはTypeScriptでコードを書くのか？AI駆動開発に必要な最低限のTypeScript知識"
emoji: "🌱"
type: "tech"
topics: ["typeScript", "ai", "AI駆動開発", "JavaScript", "contest2025ts"]
published: false
publication_name: "headwaters"
---

# TypeScriptはAI駆動開発に必須？

最近、生成AIに「こんな感じのWebアプリ作ってみて〜」と頼むと、**TypeScript**でコードが返ってくることが多くなってきました。

- 「JavaScriptじゃダメなの？」
- 「型ってやつが難しそう…」
- 「記号が多くて読む気がしない…」

そんな疑問や抵抗感、ありませんか？

でも実は、**TypeScriptはAIにとって非常に扱いやすい言語**なのです。
そして、AIと一緒に開発する私たち人間にとっては、最低限のTypeScript知識があるだけで、**コードの質もスピードも大きく変わってきます**。

もちろん知識は多いに越したことはありませんが、「全く知らずに進める状態」と「これだけは知っている状態」では結果に大きな差が出る、という前提でお話しします。

この記事では、
「なぜ開発AIがTypeScriptを使いがちなのか？」「AI駆動開発で必要なTypeScriptのポイントは何か？」を解説します。


# なぜ生成AIはTypeScriptを好むのか？

## 1. 型情報が「人間の意図」を伝えやすくするから

生成AIにコードを書かせるとき、「何をしたいのか・何が欲しいのか」を明確に伝えることが重要です。

そのための強力な手段が「型（Type）」です。
生成AIにとって型は、コードの意味や期待される動作を読み取るための大きなヒントになります。

### 型があるとAIが「意味」を理解しやすいという例

例えば、以下のようなコードをAIに渡したとします。


```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

function createUser(user: User): User {
  return user;
}
```
ここでは、`User`という **インターフェース（構造の定義）** を使って、「ユーザー`User`は `id`=数値, `name`=文字列, `email`=文字列の3つを持つよ！」とはっきり記述しています。

AIにとってはこれは仕様書のような情報になり、関数の入出力や構造が明確になるため、理解しやすくなります。つまり、AIにとって仕様書のような役割を果たすのです。

### 型がなければ、AIは推測に頼るという例
一方、同じ関数をJavaScriptで書いたらこうなります。

```javascript
function createUser(user) {
  return user;
}
```

このコードでは、`user` が何なのか？どんなフィールドを持っているのか？AIにはわかりません。補完するとしても、似た例から推測するしかないです。
この「推測」はときに便利ですが、ときにバグの温床になります。

## 2. 構造化されたコードを生成しやすいから

TypeScriptには、クラス（class）やインターフェース（interface）、ユニオン型（`|` で複数型を許容）など、構造を明示する手段がたくさんあります。

AIにとっては、これらが設計図のような役割を果たしやすく、応用も自動化しやすいというTypeScriptならではの恩恵があります。


### 例えばAPIを扱うクラスを定義すると…

```typescript
class UserService {
  // コンストラクタで ApiClient を受け取る
  constructor(private apiClient: ApiClient) {}

  // ユーザーを取得するメソッド
  async getUser(id: number): Promise<User> {
    // apiClient を使って非同期でデータを取得
    return this.apiClient.get(`/users/${id}`);
  }
}

```

このコードから、AIは以下のような情報を推論できます。

* `UserService` は「ユーザー情報を取得するサービス」
* コンストラクタには `ApiClient` が注入される（依存性注入のパターン）
* `getUser` メソッドは `number` 型の `id` を受け取り、`User` 型を返す非同期関数

これにより、テストコードの生成、他メソッドの補完、型安全性の確保などがスムーズに行えます。

### JavaScriptでは構造が曖昧になることも

同じことをJavaScriptで書いても動きますが、こうなります。

```javascript
class UserService {
  // コンストラクタで apiClient を受け取る
  constructor(apiClient) {
    this.apiClient = apiClient;
  }

  // ユーザーを取得するメソッド
  async getUser(id) {
    // apiClient を使って非同期でデータを取得
    return this.apiClient.get(`/users/${id}`);
  }
}
```

一見変わらないように見えますが、AIにとっては次のような「不明点」が増えます。

* `apiClient` に `get` メソッドがある保証がない
* `id` が数値か文字列か不明
* `getUser` が返すのが「User型」なのかどうか不明

こうした曖昧さは、AIの後続のコード生成精度を下げる要因になります。

# AI駆動開発におけるTypeScript基礎知識

では、実際にAIと協働して開発を進めるとき、私たちはどのようにTypeScriptを使いこなせばよいのでしょうか？
実際にAIと協働するうえで知っておきたいTypeScriptの基本的な知識と、プロンプト設計のコツを紹介します。

## 1. 型定義を最初に用意して「意図」を明確にする

生成AIにコードを書かせるとき、いきなり関数やクラスをお願いするのではなく、まず**受け渡すデータの型情報（インターフェース）を用意する**のが効果的です。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}
```

このような型定義があると、AIは「User型のデータを操作する関数」や「Userを使うクラス」などを、**型に沿って整合性のある形で生成**してくれます。

### **プロンプト例：**

> 「User型のインターフェースを以下のように定義しています。このUserを受け取って、ユーザー一覧に追加する関数を実装してください。」

このように**型と合わせて目的を伝える**と、AIはより正確なコードを出力してくれます。



## 2. クラスや責務ごとの構造を用意すると、AIの出力も整理される

TypeScriptのクラスやモジュール構造は、AIにとっても「どこに何を書くべきか」のガイドになります。

```typescript
class UserService {
  constructor(private apiClient: ApiClient) {}

  async getUser(id: number): Promise<User> {
    return this.apiClient.get(`/users/${id}`);
  }
}
```

このようにクラスを用意しておくと、AIはその中に適切なメソッドを追加したり、依存注入された `apiClient` を使って拡張してくれます。

### **プロンプト例：**

> 「この `UserService` クラスに、ユーザーを新規作成するメソッド `createUser` を追加してください。ApiClientを使ってPOSTリクエストを送る形でお願いします。」



## 3. ユニオン型や列挙型で「選択肢」を明確にする

TypeScriptのユニオン型や `enum` は、AIに\*\*「この値はこの中から選ばれる」という制約\*\*を明示する手段になります。

```typescript
type UserRole = 'admin' | 'member' | 'guest';

interface User {
  id: number;
  name: string;
  role: UserRole;
}
```

こうしておけば、AIは `role` に `'admin'` や `'member'` 以外の文字列を使うことはありません。条件分岐の生成精度も上がります。

### **プロンプト例：**

> 「UserRoleが 'admin' のときは管理画面にリダイレクトし、それ以外は通常のダッシュボードに遷移するような関数を作ってください。」



## 4. 型に基づいた「型安全なテスト」もお願いできる

型情報が整っていれば、AIに対して**型に沿ったテストコードの生成**も頼めます。

```typescript
function isAdult(user: User): boolean {
  return user.age >= 18;
}
```

### **プロンプト例：**

> 「この `isAdult` 関数に対して、型に沿ったユニットテストコードを `jest` で書いてください。18歳境界のテストも含めてください。」



## 5. 出力されたコードを「型チェック」視点でレビューする

生成されたコードを鵜呑みにせず、**TypeScriptの型エラーやnull安全性を確認する習慣**を持つことが、AI駆動開発における人間の重要な役割です。

### **チェックポイント例：**

* `any` や型アサーション（as）で雑に処理していないか
* `undefined` や `null` に対して安全なアクセスになっているか
* 非同期関数の戻り値型が正しく `Promise<T>` になっているか

### **プロンプト例（出力後の確認用）：**

> 「このコードに型エラーがある場合、指摘と修正案を教えてください。」



# まとめ：AIとTypeScriptは「相性抜群」

生成AIとTypeScriptを組み合わせることで、**より安全で効率的な開発**が可能になります。ポイントは以下の通りです。

### なぜAIはTypeScriptを好むのか？

* **型はAIにとっての設計図**
  → 型があることで、AIは意図を正確に理解し、適切なコードを生成しやすくなります。

* **構造が明確で保守性が高い**
  → クラスやインターフェースなど、TypeScriptの構造化された記述は、AIにとって非常に扱いやすく、ミスの少ない出力に繋がります。

* **エラーの少ないコードを生成できる**
  → 型定義やエラーハンドリングにより、バグの少ない、安全なコード生成をサポートします。

### ＋αの知識でもOK
TypeScriptを完全に理解していなくても、「型ってこういうもの」という基本がわかるだけで、AIとの協働が格段にスムーズになります。

これからのAI時代に備えて、まずはTypeScriptの“最低限”から、気軽に触れてみましょう！

## 参考資料

* [TypeScript公式ドキュメント](https://www.typescriptlang.org/docs/)
* [ChatGPT APIドキュメント](https://platform.openai.com/docs/api-reference)
* [GitHub Copilotドキュメント](https://docs.github.com/ja/copilot)
