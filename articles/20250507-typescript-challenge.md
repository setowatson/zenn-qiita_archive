---
title: "TypeScriptで実装する型安全なAPIクライアント【実践編】"
emoji: "🌱"
type: "tech"
topics: ["typescript", "api", "型安全", "zod", "contest2025ts"]
published: false
---

# はじめに

TypeScriptを使ったAPIクライアントの実装では、型安全性を確保することが重要です。最初は「型」という概念が難しく感じるかもしれませんが、開発を進めるうちにその恩恵を実感する場面が増えてきます。特に、APIクライアントの実装において型安全を意識することで、開発中のエラーを防ぎ、より信頼性の高いコードを実現できます。

私もTypeScriptを学び始めた頃、型の概念に戸惑いを感じていました。しかし、実際のプロジェクトで使い続けるうちに、型がどれほど開発効率を上げ、エラーを減らすかを実感しました。本記事では、Zodを使って型安全なAPIクライアントを実装する方法を実践的に紹介します。

## この記事で学べること
- TypeScriptの型システムの基礎知識
- Zodを使用した型定義とバリデーション
- 型安全なAPIクライアントの実装方法
- 実践的なエラーハンドリング

## 前提知識
- TypeScriptの基本的な型の概念
- JavaScriptの非同期処理（Promise, async/await）
- RESTful APIの基本的な理解

## 用語解説
- TypeScript: JavaScriptに型の概念を追加したプログラミング言語
- APIクライアント: サーバーと通信するためのプログラム
- 型安全性: プログラムの実行時に型の不一致によるエラーを防ぐ仕組み
- Zod: データの型を定義し、検証するためのライブラリ
- バリデーション: データが正しい形式であるかを確認する処理

# なぜ型安全なAPIクライアントが必要か

APIクライアントを実装する際、開発者はさまざまな問題に直面します。例えば、レスポンスの型が不明確であったり、ランタイムエラーが発生したり、型の不一致によるバグに悩まされることがあります。

これらの問題を未然に防ぐために、「型安全を確保したAPIクライアント」が非常に重要です。

## 1. レスポンスの型が不明確
```typescript
// 型が不明確な例
const response = await axios.get('/api/users/1');
console.log(response.data.name); // 型エラーになる可能性
```

この例では、`response.data`の型が不明確なため、`name`プロパティにアクセスする際にエラーが発生する可能性があります。

## 2. ランタイムエラーの発生
```typescript
// ランタイムエラーの例
const user = await fetchUser(1);
if (user.age > 18) { // ageプロパティが存在しない可能性
  console.log('成人ユーザー');
}
```

この例では、`user`オブジェクトに`age`プロパティが存在しない場合にエラーが発生します。実行時に初めてエラーに気づくというのは、開発者にとって大きなストレスですよね。

## 3. 型の不一致によるバグ
```typescript
// 型の不一致によるバグの例
interface User {
  id: number;
  name: string;
}

const user: User = {
  id: "1", // 文字列を数値型に代入しようとしている
  name: "John"
};
```

この例では、`id`プロパティに文字列を代入しようとしていますが、型定義では数値型を期待しているためエラーが発生します。このような型の不一致は、バグの原因となりやすいです。

## 4. 開発時の補完機能の欠如
```typescript
// 補完機能が効かない例
const response = await axios.get('/api/users/1');
response.data. // ここで補完が効かない
```

この例では、`response.data`の型が不明確なため、プロパティの補完機能が効きません。開発効率が下がってしまいますね。

これらの課題を解決するために、型安全なAPIクライアントの実装が重要です。

# TypeScriptの型システムの基礎

TypeScriptの型システムは、最初は複雑に感じるかもしれませんが、実際に使ってみると非常に便利です。私も最初は戸惑いましたが、今では型の恩恵を実感しています。

## 基本的な型
```typescript
// 基本的な型の例
let name: string = "John"; // 文字列型
let age: number = 25; // 数値型
let isActive: boolean = true; // 真偽値型
let hobbies: string[] = ["読書", "プログラミング"]; // 文字列の配列型
let user: { name: string; age: number } = { name: "John", age: 25 }; // オブジェクト型
```

## 型エイリアスとインターフェース
```typescript
// 型エイリアスの例
type User = {
  id: number;
  name: string;
  email: string;
};

// インターフェースの例
interface User {
  id: number;
  name: string;
  email: string;
}
```

型エイリアスとインターフェースは、オブジェクトの型を定義するための方法です。どちらも同じように使用できますが、インターフェースは拡張性が高いという特徴があります。インターフェースの方がより柔軟な実装が可能な印象です。

## ジェネリクス
```typescript
// ジェネリクスの例
function getFirst<T>(items: T[]): T {
  return items[0];
}

const numbers = getFirst<number>([1, 2, 3]); // 数値型の配列
const strings = getFirst<string>(["a", "b", "c"]); // 文字列型の配列
```

ジェネリクスは、型を引数のように扱うことができる機能です。これにより、同じ関数を異なる型で使用することができます。最初は理解するのが難しいかもしれませんが、実際に使ってみると非常に便利です。

# Zodによる型定義とバリデーション

Zodは、TypeScriptの型定義とランタイムバリデーションを統合するライブラリです。私自身、Zodを使用することで、型の安全性とバリデーションの両方を効率的に実装できるようになりました。

## 基本的なスキーマ定義
```typescript
import { z } from 'zod';

// ユーザー情報のスキーマ定義
const UserSchema = z.object({
  id: z.number(), // 数値型
  name: z.string(), // 文字列型
  email: z.string().email(), // メールアドレス形式の文字列
  role: z.enum(['admin', 'user']), // 列挙型
  createdAt: z.string().datetime(), // 日時形式の文字列
});

// 型の導出
type User = z.infer<typeof UserSchema>;

// バリデーション
const validateUser = (data: unknown): User => {
  return UserSchema.parse(data);
};
```

## スキーマの拡張
```typescript
// スキーマの拡張例
const AdminSchema = UserSchema.extend({
  permissions: z.array(z.string()), // 文字列の配列
  lastLogin: z.string().datetime().optional(), // オプショナルな日時
});

type Admin = z.infer<typeof AdminSchema>;
```

# 型安全なAPIクライアントの実装

Zodを使用した型安全なAPIクライアントの実装例を示します。

## 基本的なAPIクライアント
```typescript
import { z } from 'zod';
import axios from 'axios';

// APIクライアントの型定義
type ApiClient = {
  get: <T>(url: string, schema: z.ZodType<T>) => Promise<T>;
  post: <T>(url: string, data: unknown, schema: z.ZodType<T>) => Promise<T>;
};

// 型安全なAPIクライアントの実装
const createApiClient = (baseURL: string): ApiClient => {
  const client = axios.create({ baseURL });

  return {
    get: async <T>(url: string, schema: z.ZodType<T>) => {
      const response = await client.get(url);
      return schema.parse(response.data);
    },
    post: async <T>(url: string, data: unknown, schema: z.ZodType<T>) => {
      const response = await client.post(url, data);
      return schema.parse(response.data);
    },
  };
};
```

# エラーハンドリングと型推論

以下はZodのエラーハンドリングと型推論の機能を活用した実装例です。

## カスタムエラークラス
```typescript
// カスタムエラークラス
class ValidationError extends Error {
  constructor(public errors: z.ZodError) {
    super('Validation Error');
  }
}

// エラーハンドリングを強化したAPIクライアント
const createSafeApiClient = (baseURL: string): ApiClient => {
  const client = axios.create({ baseURL });

  return {
    get: async <T>(url: string, schema: z.ZodType<T>) => {
      try {
        const response = await client.get(url);
        return schema.parse(response.data);
      } catch (error) {
        if (error instanceof z.ZodError) {
          throw new ValidationError(error);
        }
        throw error;
      }
    },
    post: async <T>(url: string, data: unknown, schema: z.ZodType<T>) => {
      try {
        const response = await client.post(url, data);
        return schema.parse(response.data);
      } catch (error) {
        if (error instanceof z.ZodError) {
          throw new ValidationError(error);
        }
        throw error;
      }
    },
  };
};
```

# 実践的なユースケース

実際のプロジェクトでの使用例を示します。

## APIクライアントの使用例
```typescript
// APIクライアントのインスタンス化
const api = createSafeApiClient('https://api.example.com');

// ユーザー情報の取得
const fetchUser = async (id: number) => {
  const user = await api.get(`/users/${id}`, UserSchema);
  return user;
};

// ユーザーの作成
const createUser = async (userData: unknown) => {
  const newUser = await api.post('/users', userData, UserSchema);
  return newUser;
};

// 使用例
const main = async () => {
  try {
    const user = await fetchUser(1);
    console.log(user.name); // 型安全なアクセス

    const newUser = await createUser({
      name: 'John Doe',
      email: 'john@example.com',
      role: 'user',
    });
    console.log(newUser.id); // 型安全なアクセス
  } catch (error) {
    if (error instanceof ValidationError) {
      console.error('Validation Error:', error.errors);
    } else {
      console.error('Unexpected Error:', error);
    }
  }
};
```

# まとめ

型安全なAPIクライアントの実装により、以下のメリットが得られます：

1. コンパイル時の型チェック
2. ランタイムでのバリデーション
3. 開発時の補完機能
4. エラーハンドリングの改善
5. コードの保守性向上

TypeScriptとZodを組み合わせることで、より安全で保守性の高いAPIクライアントを実装することができます。

# 参考資料

- [Zod公式ドキュメント](https://zod.dev/)
- [TypeScript公式ドキュメント](https://www.typescriptlang.org/docs/)
- [Axios公式ドキュメント](https://axios-http.com/docs/intro) 