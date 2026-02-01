---
name: jest-testing
description: Jestを使用したテスト作成ガイド。ユニットテスト、統合テスト、Reactコンポーネントテスト、モック、スナップショットテスト、カバレッジ設定。
model: sonnet-4-5
---

# Jest Testing

Jestを使用したテスト作成ガイド。

## テスト基本構造

### AAAパターン

```typescript
describe("機能名", () => {
  it("期待される動作を説明", () => {
    // Arrange（準備）
    const input = "test";

    // Act（実行）
    const result = someFunction(input);

    // Assert（検証）
    expect(result).toBe("expected");
  });
});
```

## セットアップ

### jest.config.ts（TypeScript）

```typescript
import type { Config } from "jest";

const config: Config = {
  preset: "ts-jest",
  testEnvironment: "jsdom", // React用。Node.jsなら "node"
  roots: ["<rootDir>/src"],
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/src/$1",
    "\\.(css|less|scss)$": "identity-obj-proxy",
  },
  setupFilesAfterSetup: ["<rootDir>/jest.setup.ts"],
  collectCoverageFrom: [
    "src/**/*.{ts,tsx}",
    "!src/**/*.d.ts",
    "!src/**/index.ts",
  ],
};

export default config;
```

### jest.setup.ts

```typescript
import "@testing-library/jest-dom";
```

### Next.js用設定

```typescript
import nextJest from "next/jest";

const createJestConfig = nextJest({ dir: "./" });

const config = {
  testEnvironment: "jsdom",
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/src/$1",
  },
  setupFilesAfterSetup: ["<rootDir>/jest.setup.ts"],
};

export default createJestConfig(config);
```

## ユニットテスト

```typescript
import { formatDate, calculateTotal } from "./utils";

describe("formatDate", () => {
  it("日付を正しくフォーマットする", () => {
    const date = new Date("2024-01-01");
    expect(formatDate(date)).toBe("2024年1月1日");
  });

  it("無効な日付でエラーをスローする", () => {
    expect(() => formatDate(null as any)).toThrow("無効な日付");
  });
});

describe("calculateTotal", () => {
  it("税込み合計を計算する", () => {
    const items = [
      { price: 1000, quantity: 2 },
      { price: 500, quantity: 1 },
    ];
    expect(calculateTotal(items, 0.1)).toBe(2750);
  });

  it("空配列で0を返す", () => {
    expect(calculateTotal([], 0.1)).toBe(0);
  });
});
```

## Reactコンポーネントテスト

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { LoginForm } from "./LoginForm";

describe("LoginForm", () => {
  it("メールとパスワードを入力してログインできる", async () => {
    const onSubmit = jest.fn();
    const user = userEvent.setup();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText("メール"), "test@example.com");
    await user.type(screen.getByLabelText("パスワード"), "password123");
    await user.click(screen.getByRole("button", { name: "ログイン" }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: "test@example.com",
      password: "password123",
    });
  });

  it("バリデーションエラーを表示する", async () => {
    render(<LoginForm onSubmit={jest.fn()} />);
    await userEvent.click(screen.getByRole("button", { name: "ログイン" }));
    expect(screen.getByText("メールアドレスは必須です")).toBeInTheDocument();
  });
});
```

### フック（カスタムフック）のテスト

```typescript
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

describe("useCounter", () => {
  it("初期値を設定できる", () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it("incrementで1増える", () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => result.current.increment());
    expect(result.current.count).toBe(1);
  });
});
```

## モックとスタブ

### 関数モック

```typescript
// 基本
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ id: "1", name: "テスト" });

// 実装を指定
const mockCalc = jest.fn((a: number, b: number) => a + b);

// 呼び出し検証
expect(mockFn).toHaveBeenCalledTimes(1);
expect(mockFn).toHaveBeenCalledWith("arg1", "arg2");
```

### モジュールモック

```typescript
// モジュール全体をモック
jest.mock("@/lib/db", () => ({
  getUser: jest.fn().mockResolvedValue({ id: "1", name: "太郎" }),
}));

// 一部だけモック
jest.mock("@/lib/utils", () => ({
  ...jest.requireActual("@/lib/utils"),
  sendEmail: jest.fn(),
}));
```

### fetchモック

```typescript
// グローバルfetchをモック
global.fetch = jest.fn();

beforeEach(() => {
  (fetch as jest.Mock).mockClear();
});

it("ユーザーデータを取得する", async () => {
  (fetch as jest.Mock).mockResolvedValueOnce({
    ok: true,
    json: async () => ({ id: 1, name: "太郎" }),
  });

  const result = await fetchUser(1);
  expect(result).toEqual({ id: 1, name: "太郎" });
  expect(fetch).toHaveBeenCalledWith("/api/users/1");
});
```

### タイマーモック

```typescript
describe("debounce", () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it("指定時間後に関数を実行する", () => {
    const callback = jest.fn();
    const debounced = debounce(callback, 300);

    debounced();
    expect(callback).not.toHaveBeenCalled();

    jest.advanceTimersByTime(300);
    expect(callback).toHaveBeenCalledTimes(1);
  });
});
```

## スナップショットテスト

```tsx
import { render } from "@testing-library/react";
import { UserCard } from "./UserCard";

it("正しくレンダリングされる", () => {
  const { container } = render(
    <UserCard name="太郎" email="taro@example.com" />
  );
  expect(container).toMatchSnapshot();
});

// インラインスナップショット（小さいコンポーネント向け）
it("タイトルをレンダリングする", () => {
  const { getByRole } = render(<Header title="ダッシュボード" />);
  expect(getByRole("heading").textContent).toMatchInlineSnapshot(
    `"ダッシュボード"`
  );
});
```

**注意**: スナップショットは壊れやすいため、重要なUIの構造チェックに限定して使用する。

## 非同期テスト

```typescript
// async/await
it("データを取得する", async () => {
  const data = await fetchData();
  expect(data).toBeDefined();
});

// waitFor（DOMの変更を待つ）
it("読み込み後にデータを表示する", async () => {
  render(<UserProfile id="1" />);
  expect(screen.getByText("読み込み中...")).toBeInTheDocument();

  await waitFor(() => {
    expect(screen.getByText("太郎")).toBeInTheDocument();
  });
});

// findBy（waitForの省略形）
it("非同期にレンダリングされた要素を見つける", async () => {
  render(<UserProfile id="1" />);
  const name = await screen.findByText("太郎");
  expect(name).toBeInTheDocument();
});
```

## テストのライフサイクル

```typescript
describe("UserService", () => {
  let db: Database;

  beforeAll(async () => {
    // テストスイート全体で1回だけ実行
    db = await connectDatabase();
  });

  afterAll(async () => {
    await db.disconnect();
  });

  beforeEach(async () => {
    // 各テスト前に実行
    await db.clear();
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  it("...", () => {});
});
```

## テストコマンド

```bash
npx jest                      # すべてのテスト実行
npx jest --watch              # ウォッチモード
npx jest --coverage           # カバレッジレポート生成
npx jest path/to/file.test.ts # 特定ファイルのみ
npx jest -t "テスト名"         # 名前でフィルタ
npx jest --updateSnapshot     # スナップショット更新
```

## カバレッジ設定

```json
// package.json または jest.config.ts
{
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  }
}
```

## Vitest との違い

| 項目 | Jest | Vitest |
|------|------|--------|
| インポート | グローバル（設定不要） | `import { describe, it } from 'vitest'` |
| モック | `jest.fn()` | `vi.fn()` |
| モジュールモック | `jest.mock()` | `vi.mock()` |
| タイマー | `jest.useFakeTimers()` | `vi.useFakeTimers()` |
| 実行速度 | 普通 | 高速（Viteベース） |
| 設定 | `jest.config.ts` | `vitest.config.ts` |

新規プロジェクトではVitestを推奨。既存のJestプロジェクトは無理に移行しなくてよい。

## アンチパターン

- スナップショットテストの乱用（変更のたびに更新が必要になる）
- `jest.mock`の過剰使用（本物を使えるなら使う）
- テスト間の状態共有（`beforeEach`でリセットする）
- `toBeTruthy()`/`toBeFalsy()`の多用（`toBe(true)`等、具体的なマッチャーを使う）
- `done`コールバック（`async/await`を使う）
