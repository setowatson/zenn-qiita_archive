---
title: なぜ生成AIはTypeScriptでコードを書くのか？AI駆動開発に必要な最低限のTypeScript知識
publication_name: headwaters
private: false
tags:
  - typeScript
  - ai
  - AI駆動開発
  - JavaScript
  - contest2025ts
updated_at: '2025-05-18T02:09:51.949Z'
id: null
organization_url_name: null
slide: false
---
# TypeScriptはAI駆動開発に必須？

AI駆動スタイルでwebサイトやwebアプリを開発しようとした時に、**TypeScript**が即時に選ばれた経験ありませんか？

もちろん条件や環境によっては他言語が選ばれますが、私は技術選定をお任せにした上で個人開発を行ってみると、ほぼすべてのプロジェクトでTypeScriptが選ばれました。

![ts1![](https://storage.googleapis.com/zenn-user-upload/e89d99aba814-20250513.png)
](/images/ts1.png)

↑先日作ったwebアプリの言語選定(別途記事にする予定です)

私自身「JavaScriptをより扱いやすくした言語？」くらいの認識しかなかったのでこれを機に色々と調べてみたのですが、**TypeScriptはAIにとって非常に扱いやすい言語** のようです。

ではAIと一緒に開発を進める前提に立った時、私たち人間は「TypeScriptについて最低限何をどれくらい」知っておくべきなのでしょうか。


- 「なぜ開発AIがTypeScriptを使いがちなのか？」
- 「AI駆動開発で必要なTypeScriptのポイントは何か？」

を今回考えてみようと思います。

※もちろんすべてを知っているに越したことはありませんが、「コーディングAIと0から開発を進める」「その上で最低限知っていたほうがいい特徴」という条件で考えます。

# なぜ生成AIはTypeScriptを好むのか？

## 1. 型情報が「人間の意図」を伝えやすくするから

みなさんもご存知の通り、生成AIにコードを書かせるときはAIに **「何をしてほしいのか・何が欲しいのか」** を明確に伝えることが重要です。

その伝える手段の1つが **「型（Type）」** です。
生成AIにとって型は、コードの意味や期待される動作を読み取るための大きなヒントになるようです。

TspeScriptはこの **「型」** を共有しやすい言語のため、AIと人間の対話が安全に行われやすい要素があります。

### 型があるからAIも意味を理解しやすい例

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
ここでは、`User`という **インターフェース（構造の定義）** を使って、「ユーザー`User`は `id`=数値, `name`=文字列, `email`=文字列の3つを持つよ！」と記述しています。

AIにとってはこれは仕様書のような情報になり、関数の入出力や構造が明確になるため人間の意図をより理解しやすくなります。

### 型がないためAIが推測してしまう例

一方、同じ関数をJavaScriptであえて推測するように書いたらこうなります。

```javascript
function createUser(user) {
  return user;
}
```

このコードでは、`user` が何なのか？どんなフィールドを持っているのか？AIにはわかりません。補完するとしても、似た例から推測するしかないです。
この「推測」はときに便利ですが、ときにバグの温床になります。特にAIにコーディングを任せる領域が多いときはその傾向が強くなるようです。

「AIに任せる範囲が増えるほど、型の明示が価値を持つ」という視点は、AI時代のフロントエンドや業務コード設計でかなり重要となります

## 2. 構造化されたコードを理解し、出力しやすいから

TypeScriptには、`interface`や`class`、`union`など、構造を明示する型システムがたくさんあります。
これらの機能も、AIにとって設計図のような役割を果たしやすく、また意図が誤解されにくいというメリットがあります。


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

これにより、テストコードの生成、他メソッドの補完、型安全性の確保などがスムーズに行えます。これはAIに限らず、TypeScriptの大きな魅力の1つにもなっています。

### JavaScriptでは構造が曖昧になることもある

同じ内容をJavaScriptで書いても動きますが、こうなります。

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

* `apiClient` に `get` メソッドがある保証がない？
* `id` は数値？文字列？
* `getUser` が返すのは「User型」？

こうした曖昧さは、AIの後続のコード生成精度を下げる要因になります。

構造化されたコードのほうが理解しやすいのはどの場面でも間違いないですが、特にAIコーディングの場合は、
「自分自身も構造化データを出力しやすく、また出力を次の入力に加える…そのループが起きる」
というサイクルはAIととても相性が良いです。

1. `interface` や `class` を定義する
2. それに準拠した関数やクラスを自動生成する
3. その関数やクラスに基づくユニットテストやドキュメントを補完する
4. 修正・追加時にも型情報を参照して文脈を理解する
この循環を意識しやすいのがTypeScriptの特徴です。


# AI駆動開発におけるTypeScriptの活かし方

以上のようにAIと協働してコードを書くとき、TypeScriptは単なる型付き言語以上に「意図を伝えるための言語」になります。そのなかで生成AIとのやりとりの精度や効率が向上させるために意識できることを4つピックアップしてみました。

## 1. 型定義を「プロンプトの入口」にする

生成AIにコードを書かせるとき、いきなり関数やクラスをお願いするのではなく、まず**扱いたいデータの型（インターフェース）を提示すること**で、より正確なコード出力につながります。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}
```


### **✅プロンプト例：**

> `User`型のインターフェースを上のように定義します。この`User`を元に、ユーザー一覧に追加する関数`addUser`を実装してください。

このような型定義があると、AIは「User型のデータを操作する関数」や「Userを使うクラス」などを、**型に沿って整合性のある形で生成**しやすくなります。


## 2. 設定ファイルでプロジェクトの「意図」を明確にする

TypeScriptでは、 `tsconfig.json` や `package.json` などの**設定ファイル**がより重要です。

これはAI駆動の場面でも同様、もしくはそれ以上に重要で、特に`tsconfig.json` は型チェックのルールや対象スコープ、`strict` モードの有無など、**生成AIの出力コードの性質**に直結します。


### ✅AI駆動開発において `tsconfig.json` で特に重要な項目

| 項目                      | 重要な理由                                                      |
| ----------------------- | ---------------------------------------------------------- |
| `strict: true`          | AIに厳密な型前提でコードを生成させられる（バグ減・型補完向上）                           |
| `baseUrl` / `paths`     | モジュール解決をAIに伝えられ、絶対パスインポートや設計方針が反映される                       |
| `types`                 | JestやNodeの型が明示されていると、AIがその型に沿ったテストやコードを書く                  |
| `target`, `module`      | AIがどのJS構文（ES2020など）を想定すべきか判断できる                            |
| `esModuleInterop: true` | import構文の揺れ（`import * as fs` vs `import fs from 'fs'`）を避ける |


特に `strict` はAIにとって最も影響が大きく、曖昧な型を避けて「堅牢なコード」を出力させる前提になります。



### **✅プロンプト例：**

（Next.js + TypeScript を前提にしています）

> これからNext.js + TypeScriptのプロジェクトを進めます。以下はこのプロジェクトの `tsconfig.json` です。この設定に沿ったコードを書いてください。特に型の厳格さ (`strict: true`) や、`@/` を使ったパス解決、`jsx`の扱いなどに注意してください。

> ```json
> {
>   "compilerOptions": {
>     "lib": ["dom", "dom.iterable", "esnext"],
>     "allowJs": true,
>     "target": "ES6",
>     "skipLibCheck": true,
>     "strict": true,
>     "noEmit": true,
>     "esModuleInterop": true,
>     "module": "esnext",
>     "moduleResolution": "bundler",
>     "resolveJsonModule": true,
>     "isolatedModules": true,
>     "jsx": "preserve",
>     "incremental": true,
>     "plugins": [
>       {
>         "name": "next"
>       }
>     ],
>     "paths": {
>       "@/*": ["./*"]
>     }
>   },
>   "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
>   "exclude": ["node_modules"]
> }
> ```


## 3. ユニオン型や列挙型で「選択肢」を明確にする

生成AIにとって、TypeScriptのユニオン型や `enum` は **「この変数は、限られた値のどれかになるよ！というルール」** として非常に重要です。
AIは自由度が高すぎると意図を外しやすくなりますが、選択肢が明確になることで、意図を正しく読み取り、誤解の少ないコードを出力できます。

```typescript
type UserRole = 'admin' | 'member' | 'guest';

interface User {
  id: number;
  name: string;
  role: UserRole;
}
```

こうしておけば、AIは `role` に `'admin'` や `'member'` 以外の文字列を使うことはなくなり、また以下のように「明確に役割ごとの分岐を記述」できます。

```typescript
function redirectUser(user: User) {
  if (user.role === 'admin') {
    return '/admin/dashboard';
  } else if (user.role === 'member') {
    return '/dashboard';
  } else {
    return '/welcome';
  }
}
```

### **✅プロンプト例：**

> UserRole は 'admin' | 'member' | 'guest' のユニオン型です。
User の role に応じてリダイレクト先を変える関数を実装してください。
不正な文字列は来ない前提で、全パターンを網羅してください。



## 4. 出力されたコードを「型の信頼性」視点でレビューする


TypeScriptの目的は“予測可能な安全なコード”を書くこと。その本来の価値を引き出すためには、**AIが型を守れているかを人間がレビューする視点**が不可欠です。

特にAI駆動開発では「とりあえず動くコード」はすぐに出ますが、それが安全で拡張性があるかどうかは人間が判断する必要があります。


### ✅AI駆動開発におけるTypeScriptレビュー用チェックリスト

| チェック項目                                   | 内容                     | よくあるNG例                                       | 修正アドバイス                                                | AIへのチェック依頼プロンプト                                                                 |
| ---------------------------------------- | ---------------------- | --------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------- |
| **① `any` が使われていないか**                    | 型安全性が失われるため極力使わない      | `const data: any = res.data;`                 | `User[]` など具体的な型を定義                                    | `このコード内に any を使っている箇所があれば教えてください。その理由と代替案もお願いします。`                              |
| **② 型アサーション（`as`）を濫用していないか**             | 本来の型チェックを飛ばすため、誤動作しやすい | `user as AdminUser`                           | 実行時に型を分岐して安全に処理する                                      | `過剰な as による型アサーションが使われていないか確認してください。代わりに型安全な判定ができるようにしてください。`                   |
| **③ `null` や `undefined` に安全にアクセスしているか** | `?.` や明示的な存在確認が必要      | `user.name.toUpperCase()`                     | `user?.name?.toUpperCase()` のように安全に処理                  | `null や undefined に対して危険なアクセスをしている箇所があれば教えてください。オプショナルチェーンや安全なガードを使って修正してください。` |
| **④ 非同期関数の戻り値型が正しいか**                    | `Promise<T>` など明示する    | `async function fetchData() { return data; }` | `Promise<Data>` のように型定義を追加                             | `非同期関数の戻り値型が明示されているか確認してください。Promise<T> の型が適切かも見てください。`                         |
| **⑤ 型の抜けがないか（引数・戻り値・stateなど）**           | 全体的な型定義の網羅性を確認         | `function submit(form) {}`                    | `submit(form: FormData): void` のように型付け                 | `関数の引数や戻り値、ReactコンポーネントのPropsなどに型の抜けがないか教えてください。`                               |
| **⑥ 適切なユニオン型や `enum` が使われているか**          | 入力値を明確に制限する            | `role: string`                                | `type UserRole = 'admin' \| 'member' \| 'guest'` などを使う | `このコードで、文字列などが任意の値になっていないか確認してください。必要に応じてユニオン型や enum を提案してください。`                |
| **⑦ JSXやReactコンポーネントの型定義がされているか**        | Props や state の型が正しいか  | `function Button(props) {}`                   | `interface ButtonProps { label: string }` を使う          | `このReactコンポーネントに型定義の不足がないか、PropsやStateの型が適切か確認してください。`                          |


人間の目で上記項目をチェックしますが、その補助として、AIにも各観点のレビューを依頼することはアリかと思っています。



# ただし、TypeScriptは「型安全にできる言語」という理解も必要

TypeScriptは型安全な設計が可能な言語ですが、**使い方を誤れば簡単に型の恩恵を失ってしまいます。**

柔軟に書けるがゆえに、「型を導入したつもりが、実は安全性が担保されていない」といった中途半端な状態になりやすく、それが批判される原因にもなっています。

AI自身がこのような設計ミスを頻発するわけではありませんが、TypeScriptの仕組みを理解しないままAIと人間の作業を混ぜて使うと、意図せぬ不整合が生まれることがあります。


## TypeScriptが「型安全じゃない」と言われる主な理由

### 1. `any`型の存在

`any`を使うと、型チェック(コンパイラー)が完全に無効化されます。本来は型がわからない外部コードを扱うときなど型安全性を一時的に放棄したいときに使用する機能ですが、この`any`の使い方を間違えると実行時エラーが爆増します。
雑に書こうと思えばTypeScriptはJavaScriptと変わらないほど緩くなる、という例の1つです。

```ts
let data: any = "hello";
data(); // 実行時エラー：TypeError: data is not a function
```

```ts
let data: string = "hello";
data(); // ❌ コンパイルエラー：'data' は関数ではありません
```

上の例では、文字列helloを変数`data`に代入しています。しかし、2行目の`data`は本来"関数"を呼び出す構文のため、実行するとエラーになります。このような単純なエラーは本来TypeScriptコンパイラーで発見できますが、`any`を設定した値についてはコンパイラーが警告しなくなります。

そもそも`any`を乱用するのであればTypeScriptを使う意味そのものがなくなります。

### 2. 型アサーションの乱用

また、TypeScriptには `as`構文のように型推論を“上書き”して無理やり別の型として扱う機能(型アサーション)もあります。こちらも使い方を誤ったり乱用したりすると、実際の値と型が食い違い、深刻なバグにつながります。

```ts
const value = "123" as unknown as number;
console.log(value + 1); // 実行結果: "1231"（数値ではなく文字列結合）
```

上の例で、"123" はもともと文字列ですが、`as unknown` でいったん「何者でもない」状態になり、`as number` で「これは数値や！」と強引にアサーションしています。
ただし、実際は `string` のままなので数値演算に失敗しておかしな挙動となってしまいます。

型アサーションを使うのは「本当に型が正しいことを確信している場合だけ」にしたいですね。


## 「TypeScriptはJavaScriptの上位互換」とは限らない

よく"TypeScriptはJavaScriptの進化版"っぽく言われますが、実際には静的型付けと動的言語という**本質的に異なる思想を無理やり同居させた設計** とも言えます。

そのためTypeScriptは、必ずしも“JavaScriptの上位互換”とは言えません。

例えば、JavaScriptでは当たり前に行われる「動的な構造変更」や「柔軟な型の使いまわし」は、TypeScriptでは警告や型定義の複雑さにつながることが多く「柔軟性が制限される」「型に振り回される」と感じる場面が出てきます。

JavaScript的な「自由に書ける柔軟さ」を維持したいときには、TypeScriptの「正確さ」「安全性」が返って邪魔になることもあるということです。

## AI駆動開発だからといって、必ずしもTypeScriptではない

上で述べているように型があることで、AIがコードの意味をより正確に把握しやすくなるのは事実です。
**だからといって「AI開発には必ずTypeScript」というわけではありません。**

プロトタイプを高速に作る、ルールがまだ決まっていない部分を試すなどの場面ではJavaScriptのほうがスムーズに進むこともあります。むしろ始めから型でガチガチにしてしまうと、柔軟な発想がしづらくなることもあります。

AI駆動開発で大事なのは「TypeScriptを使うかどうか」ではなく、**AIとどんな開発スタイルを取りたいか、そのために型が必要かどうかを判断すること** です。


# まとめ：AIとTypeScriptは「使い方を間違わなければ」相性抜群！

生成AIとTypeScriptを組み合わせることで、**より安全で効率的な開発**が可能になります。ポイントは以下の通りです。

### なぜAIはTypeScriptを好むのか？

* **型はAIにとっての設計図**
  → 型があることで、AIは意図を正確に理解し、適切なコードを生成しやすくなります。

* **構造が明確で保守性が高い**
  → クラスやインターフェースなど、TypeScriptの構造化された記述はAIにとって非常に扱いやすく、ミスの少ない出力に繋がります。

* **エラーの少ないコードを生成できる**
  → 型定義やエラーハンドリングにより、バグの少ない安全なコード生成をサポートします。

### ＋αの知識でもOK
TypeScriptを完全に理解していなくても、「型ってこういうもの」という基本がわかるだけでAIとの協働が格段にスムーズになります。

これからのAI時代に備えて、まずはTypeScriptの“最低限”から気軽に触れてみましょう！

## 参考資料

公式資料
* [TypeScript公式ドキュメント](https://www.typescriptlang.org/docs/)
- [サバイバルTypeScript](https://typescriptbook.jp/)

参考にしたブログ
- [tsconfig.jsonの主要オプションを理解する](https://qiita.com/ryokkkke/items/390647a7c26933940470)

- [AI駆動開発やバイブコーディングがOSSライブラリ開発や採用に与える影響について](https://qiita.com/SFITB/items/abe46878a0de2a595763#ai%E9%A7%86%E5%8B%95%E9%96%8B%E7%99%BA%E4%BB%A5%E5%89%8D%E3%81%AFoss%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E3%81%8C%E5%B7%A5%E6%95%B0%E5%89%8A%E6%B8%9B%E3%81%AB%E5%AF%84%E4%B8%8E%E3%81%97%E3%81%A6%E3%81%8D%E3%81%9F)

- [TypeScriptの危険性](https://qiita.com/MadakaHeri/items/45e514dfbbc85fd64c77)

- [TypeScriptは型安全じゃないからすばらしい](https://mametter.hatenablog.com/entry/2024/10/07/095302)

- [AIの時代だからこそ、型システムがより厳格な静的型付け言語（GoやRustなど）が良いのでは？という話](https://zenn.dev/dress_code/articles/9fcfe8a7288dbd)

- [TypeScriptのユーティリティ型を使いこなす：実践的なレシピ集🎯](https://zenn.dev/cloudcreatorlab/articles/a5bfbaaf7f761d)

今回TypeScriptに対して様々な立場を取られている方がいることを改めて知りました。調べる過程でとても勉強になりました。