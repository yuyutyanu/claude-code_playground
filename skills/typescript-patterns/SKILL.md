---
name: typescript-patterns
description: TypeScriptの実用的な型パターン集。ユーティリティ型、型ガード、ジェネリクス、判別共用体など、日常的に使える型テクニック。
model: sonnet-4-5
---

# TypeScriptパターン

## ユーティリティ型の活用

```typescript
// Partial: すべてのプロパティをオプショナルに
type UpdateUser = Partial<User>;

// Pick: 必要なプロパティだけ抽出
type UserSummary = Pick<User, "id" | "name">;

// Omit: 不要なプロパティを除外
type CreateUser = Omit<User, "id" | "createdAt">;

// Record: キーと値の型を定義
type StatusMap = Record<Status, string>;

// Required: すべてのプロパティを必須に
type CompleteConfig = Required<Config>;
```

## 判別共用体（Discriminated Union）

```typescript
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

function handleResult(result: Result<User>) {
  if (result.success) {
    // result.data にアクセス可能（型安全）
    console.log(result.data.name);
  } else {
    // result.error にアクセス可能
    console.error(result.error);
  }
}
```

## 型ガード

```typescript
// カスタム型ガード
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "id" in value && "name" in value;
}

// in演算子による絞り込み
function handle(event: MouseEvent | KeyboardEvent) {
  if ("key" in event) {
    // KeyboardEvent
  }
}
```

## ジェネリクス

```typescript
// 汎用的なAPIレスポンス型
interface ApiResponse<T> {
  data: T;
  meta: { total: number; page: number };
}

// 制約付きジェネリクス
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// デフォルト型パラメータ
interface PaginatedList<T, M = { total: number }> {
  items: T[];
  meta: M;
}
```

## const assertionとsatisfies

```typescript
// as const: リテラル型として推論
const ROUTES = {
  home: "/",
  about: "/about",
  users: "/users",
} as const;

type Route = (typeof ROUTES)[keyof typeof ROUTES]; // "/" | "/about" | "/users"

// satisfies: 型チェックしつつリテラル型を保持
const config = {
  port: 3000,
  host: "localhost",
} satisfies Record<string, string | number>;
// config.port は number型（Record<string, string | number>ではない）
```

## テンプレートリテラル型

```typescript
type EventName = `on${Capitalize<"click" | "focus" | "blur">}`;
// "onClick" | "onFocus" | "onBlur"

type Getter<T extends string> = `get${Capitalize<T>}`;
// Getter<"name"> → "getName"
```

## Branded Types（名前付き型）

```typescript
type UserId = string & { readonly __brand: "UserId" };
type PostId = string & { readonly __brand: "PostId" };

function createUserId(id: string): UserId {
  return id as UserId;
}

// UserIdとPostIdを混同できない
function getUser(id: UserId) { ... }
getUser(postId); // 型エラー
```

## アンチパターン

- `any`の多用 → `unknown`を使って型ガードで絞り込む
- 過剰な型アサーション（`as`） → 型ガードを使う
- 巨大なインターフェース → 小さな型に分割して合成
- 型とロジックの重複 → `typeof`、`keyof`、`ReturnType`で導出
