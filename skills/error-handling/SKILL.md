---
name: error-handling
description: フロントエンド・バックエンドのエラーハンドリングパターン。カスタムエラークラス、境界処理、ユーザー通知、ログ戦略。
model: sonnet-4-5
---

# エラーハンドリング

## 基本原則

1. エラーは握りつぶさない
2. ユーザーには分かりやすいメッセージを表示
3. 開発者には詳細なログを残す
4. 適切なレイヤーでキャッチする

## カスタムエラークラス

```typescript
class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = "AppError";
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super("NOT_FOUND", `${resource}が見つかりません`, 404);
  }
}

class ValidationError extends AppError {
  constructor(public details: Record<string, string[]>) {
    super("VALIDATION", "入力内容に問題があります", 400);
  }
}
```

## バックエンドのエラーハンドリング

```typescript
// 集約エラーハンドラ
function handleApiError(error: unknown): NextResponse {
  if (error instanceof AppError && error.isOperational) {
    return NextResponse.json(
      { error: { code: error.code, message: error.message } },
      { status: error.statusCode }
    );
  }

  // 予期しないエラー（プログラミングエラー等）
  console.error("予期しないエラー:", error);
  return NextResponse.json(
    { error: { code: "INTERNAL", message: "サーバーエラーが発生しました" } },
    { status: 500 }
  );
}
```

## フロントエンドのエラーハンドリング

### React Error Boundary

```tsx
// Next.js App Router
// app/error.tsx
"use client";

export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div role="alert">
      <h2>エラーが発生しました</h2>
      <p>{error.message}</p>
      <button onClick={reset}>再試行</button>
    </div>
  );
}
```

### API呼び出しのエラー処理

```typescript
async function apiCall<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) {
    const body = await res.json().catch(() => null);
    throw new AppError(
      body?.error?.code ?? "API_ERROR",
      body?.error?.message ?? `APIエラー (${res.status})`,
      res.status
    );
  }
  return res.json();
}
```

## 非同期エラーのパターン

### Result型（例外を使わない）

```typescript
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

async function safeAsync<T>(fn: () => Promise<T>): Promise<Result<T>> {
  try {
    return { ok: true, value: await fn() };
  } catch (error) {
    return { ok: false, error: error instanceof Error ? error : new Error(String(error)) };
  }
}

// 使用例
const result = await safeAsync(() => fetchUser(id));
if (!result.ok) {
  // エラー処理
}
```

## ログ戦略

| レベル | 用途 |
|--------|------|
| error | 即時対応が必要なエラー |
| warn | 正常だが注意が必要な状態 |
| info | 重要なビジネスイベント |
| debug | 開発時のデバッグ情報（本番では無効） |

## アンチパターン

- 空のcatchブロック（`catch (e) {}`）
- エラーメッセージに内部情報を露出（スタックトレース、DB構造等）
- すべてのエラーを同じように扱う
- console.logでのエラーログ（構造化ログを使う）
