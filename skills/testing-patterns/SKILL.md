---
name: testing-patterns
description: テストの設計パターン集。モック戦略、非同期テスト、コンポーネントテスト、APIテスト、テストデータ管理の実践的ガイド。
model: sonnet-4-5
---

# テストパターン

## モック戦略

### 関数モック

```typescript
import { vi } from "vitest";

// 基本的なモック
const mockFn = vi.fn();
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ id: "1", name: "テスト" });

// 実装を指定
const mockCalc = vi.fn((a: number, b: number) => a + b);
```

### モジュールモック

```typescript
// モジュール全体をモック
vi.mock("@/lib/db", () => ({
  getUser: vi.fn().mockResolvedValue({ id: "1", name: "太郎" }),
}));

// 一部だけモック（残りは実際の実装）
vi.mock("@/lib/utils", async (importOriginal) => {
  const actual = await importOriginal<typeof import("@/lib/utils")>();
  return { ...actual, sendEmail: vi.fn() };
});
```

### fetchモック

```typescript
// グローバルfetchをモック
const mockFetch = vi.fn();
global.fetch = mockFetch;

mockFetch.mockResolvedValueOnce({
  ok: true,
  json: () => Promise.resolve({ data: [] }),
});
```

## コンポーネントテスト

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

describe("LoginForm", () => {
  it("メールアドレスとパスワードを入力してログインできる", async () => {
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await userEvent.type(screen.getByLabelText("メール"), "test@example.com");
    await userEvent.type(screen.getByLabelText("パスワード"), "password123");
    await userEvent.click(screen.getByRole("button", { name: "ログイン" }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: "test@example.com",
      password: "password123",
    });
  });

  it("バリデーションエラーを表示する", async () => {
    render(<LoginForm onSubmit={vi.fn()} />);
    await userEvent.click(screen.getByRole("button", { name: "ログイン" }));
    expect(screen.getByText("メールアドレスは必須です")).toBeInTheDocument();
  });
});
```

## APIルートテスト

```typescript
import { GET, POST } from "@/app/api/users/route";
import { NextRequest } from "next/server";

describe("GET /api/users", () => {
  it("ユーザー一覧を返す", async () => {
    const req = new NextRequest("http://localhost/api/users");
    const res = await GET(req);
    const body = await res.json();
    expect(res.status).toBe(200);
    expect(body.data).toBeInstanceOf(Array);
  });
});

describe("POST /api/users", () => {
  it("バリデーションエラーで400を返す", async () => {
    const req = new NextRequest("http://localhost/api/users", {
      method: "POST",
      body: JSON.stringify({ name: "" }),
    });
    const res = await POST(req);
    expect(res.status).toBe(400);
  });
});
```

## テストデータ

### Factory関数

```typescript
function createUser(overrides: Partial<User> = {}): User {
  return {
    id: "test-id",
    name: "テストユーザー",
    email: "test@example.com",
    createdAt: new Date("2024-01-01"),
    ...overrides,
  };
}

// 使用
const admin = createUser({ role: "admin" });
const inactive = createUser({ status: "inactive" });
```

## テスト設計の原則

1. **AAAパターン**: Arrange（準備）→ Act（実行）→ Assert（検証）
2. **振る舞いをテスト**: 実装の詳細ではなく、ユーザーから見える結果
3. **セマンティックなセレクタ**: `getByRole`, `getByLabelText` を優先
4. **テストの独立性**: 各テストは他のテストに依存しない

## アンチパターン

- 実装の詳細をテスト（state変数の値を直接検証等）
- テスト間の依存（順序に依存するテスト）
- 過剰なモック（可能なら実物を使う）
- `data-testid`への過度な依存（セマンティックセレクタを優先）
