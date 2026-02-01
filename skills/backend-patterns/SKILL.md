---
name: backend-patterns
description: Node.js/Next.jsバックエンド設計パターン集。API設計、データベース操作、キャッシング、エラーハンドリング、認証・認可。
model: sonnet-4-5
---

# バックエンドパターン

## API設計

### RESTful規則

```
GET    /api/users          # 一覧取得
GET    /api/users/:id      # 詳細取得
POST   /api/users          # 作成
PUT    /api/users/:id      # 更新
DELETE /api/users/:id      # 削除
```

### レスポンス形式の統一

```typescript
// 成功
{ "data": { ... }, "meta": { "total": 100, "page": 1 } }

// エラー
{ "error": { "code": "NOT_FOUND", "message": "ユーザーが見つかりません" } }
```

### Next.js Route Handler

```typescript
import { NextRequest, NextResponse } from "next/server";

export async function GET(req: NextRequest) {
  try {
    const { searchParams } = new URL(req.url);
    const page = Number(searchParams.get("page") ?? "1");
    const data = await getUsers({ page });
    return NextResponse.json({ data });
  } catch (error) {
    return NextResponse.json({ error: { code: "INTERNAL", message: "サーバーエラー" } }, { status: 500 });
  }
}
```

## サービス層パターン

```typescript
// リポジトリ: データアクセスを抽象化
class UserRepository {
  async findById(id: string): Promise<User | null> {
    return db.user.findUnique({ where: { id } });
  }
}

// サービス: ビジネスロジック
class UserService {
  constructor(private repo: UserRepository) {}

  async getUser(id: string): Promise<User> {
    const user = await this.repo.findById(id);
    if (!user) throw new ApiError("NOT_FOUND", "ユーザーが見つかりません");
    return user;
  }
}
```

## エラーハンドリング

```typescript
class ApiError extends Error {
  constructor(public code: string, message: string, public status = 400) {
    super(message);
  }
}

// 集約エラーハンドラ
function handleError(error: unknown) {
  if (error instanceof ApiError) {
    return NextResponse.json({ error: { code: error.code, message: error.message } }, { status: error.status });
  }
  console.error("予期しないエラー:", error);
  return NextResponse.json({ error: { code: "INTERNAL", message: "サーバーエラー" } }, { status: 500 });
}
```

## リトライとバックオフ

```typescript
async function withRetry<T>(fn: () => Promise<T>, maxRetries = 3): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise((r) => setTimeout(r, 2 ** i * 1000));
    }
  }
  throw new Error("unreachable");
}
```

## キャッシング

```typescript
// Next.js fetch キャッシュ
const data = await fetch(url, { next: { revalidate: 3600 } }); // 1時間キャッシュ

// Redis（外部キャッシュ）
async function cached<T>(key: string, ttl: number, fn: () => Promise<T>): Promise<T> {
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit);
  const result = await fn();
  await redis.set(key, JSON.stringify(result), "EX", ttl);
  return result;
}
```

## 構造化ログ

```typescript
function log(level: "info" | "warn" | "error", message: string, meta?: Record<string, unknown>) {
  console.log(JSON.stringify({ timestamp: new Date().toISOString(), level, message, ...meta }));
}
```
