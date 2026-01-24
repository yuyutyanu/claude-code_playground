# Vitest テストガイド

このドキュメントは、Vitestを使用したテスト戦略とベストプラクティスを提供します。

## 概要

- **フレームワーク**: Vitest
- **テストランナー**: Viteベースの高速実行
- **API**: Jest互換
- **React Testing**: @testing-library/react

---

## セットアップ

### 基本設定

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    css: true,
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'vitest.setup.ts',
        '**/*.config.ts',
        '**/*.d.ts',
      ],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './'),
    },
  },
})
```

### セットアップファイル

```typescript
// vitest.setup.ts
import '@testing-library/jest-dom'
import { cleanup } from '@testing-library/react'
import { afterEach } from 'vitest'

// 各テスト後にクリーンアップ
afterEach(() => {
  cleanup()
})
```

---

## テストの構造

### ファイル命名規則

```
src/
├── components/
│   ├── Button.tsx
│   └── Button.test.tsx        # コンポーネントテスト
├── lib/
│   ├── utils.ts
│   └── utils.test.ts          # ユニットテスト
└── app/
    └── api/
        └── users/
            ├── route.ts
            └── route.test.ts  # APIテスト
```

### 基本テンプレート

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest'

describe('FeatureName', () => {
  beforeEach(() => {
    // テスト前のセットアップ
  })

  afterEach(() => {
    // テスト後のクリーンアップ
  })

  it('should do something', () => {
    // Arrange (準備)
    const input = 'test'

    // Act (実行)
    const result = someFunction(input)

    // Assert (検証)
    expect(result).toBe('expected')
  })
})
```

---

## ユニットテスト

### 基本的な例

```typescript
// lib/utils.test.ts
import { describe, it, expect } from 'vitest'
import { formatDate, calculateTotal } from './utils'

describe('formatDate', () => {
  it('should format date correctly', () => {
    const date = new Date('2024-01-01')
    expect(formatDate(date)).toBe('2024年1月1日')
  })

  it('should handle invalid date', () => {
    expect(() => formatDate(null)).toThrow()
  })
})

describe('calculateTotal', () => {
  it('should calculate total with tax', () => {
    const items = [
      { price: 100, quantity: 2 },
      { price: 200, quantity: 1 },
    ]
    expect(calculateTotal(items, 0.1)).toBe(440) // (200 + 200) * 1.1
  })

  it('should return 0 for empty array', () => {
    expect(calculateTotal([], 0.1)).toBe(0)
  })
})
```

---

## Reactコンポーネントテスト

### 基本的なコンポーネントテスト

```typescript
// components/Button.test.tsx
import { describe, it, expect, vi } from 'vitest'
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from './Button'

describe('Button', () => {
  it('should render with text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument()
  })

  it('should call onClick when clicked', async () => {
    const handleClick = vi.fn()
    const user = userEvent.setup()

    render(<Button onClick={handleClick}>Click me</Button>)

    await user.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### Server Componentのテスト

```typescript
// app/components/UserList.test.tsx
import { describe, it, expect, vi } from 'vitest'
import { render, screen } from '@testing-library/react'
import UserList from './UserList'

// fetchをモック
global.fetch = vi.fn()

describe('UserList', () => {
  it('should render user list', async () => {
    vi.mocked(fetch).mockResolvedValueOnce({
      json: async () => [
        { id: 1, name: 'User 1' },
        { id: 2, name: 'User 2' },
      ],
    } as Response)

    const { container } = render(await UserList())

    expect(screen.getByText('User 1')).toBeInTheDocument()
    expect(screen.getByText('User 2')).toBeInTheDocument()
  })
})
```

### カスタムフックのテスト

```typescript
// hooks/useCounter.test.ts
import { describe, it, expect } from 'vitest'
import { renderHook, act } from '@testing-library/react'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter())

    expect(result.current.count).toBe(0)

    act(() => {
      result.current.increment()
    })

    expect(result.current.count).toBe(1)
  })

  it('should decrement counter', () => {
    const { result } = renderHook(() => useCounter(10))

    act(() => {
      result.current.decrement()
    })

    expect(result.current.count).toBe(9)
  })
})
```

---

## モックとスタブ

### 関数のモック

```typescript
import { vi } from 'vitest'

// 関数をモック
const mockFn = vi.fn()
mockFn.mockReturnValue('mocked value')
mockFn.mockResolvedValue('async mocked value')

// モジュールをモック
vi.mock('./api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ id: 1, name: 'Test User' }),
}))
```

### fetchのモック

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest'

describe('API calls', () => {
  beforeEach(() => {
    global.fetch = vi.fn()
  })

  it('should fetch user data', async () => {
    const mockUser = { id: 1, name: 'Test User' }

    vi.mocked(fetch).mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser,
    } as Response)

    const result = await fetchUser(1)

    expect(fetch).toHaveBeenCalledWith('/api/users/1')
    expect(result).toEqual(mockUser)
  })
})
```

### Server Actionsのモック

```typescript
import { vi } from 'vitest'

// Server Actionをモック
vi.mock('@/app/actions', () => ({
  createUser: vi.fn().mockResolvedValue({ success: true }),
  deleteUser: vi.fn().mockResolvedValue({ success: true }),
}))
```

---

## 統合テスト

### API Routeのテスト

```typescript
// app/api/users/route.test.ts
import { describe, it, expect, vi } from 'vitest'
import { GET, POST } from './route'

describe('/api/users', () => {
  describe('GET', () => {
    it('should return users list', async () => {
      const request = new Request('http://localhost:3000/api/users')
      const response = await GET(request)
      const data = await response.json()

      expect(response.status).toBe(200)
      expect(Array.isArray(data)).toBe(true)
    })
  })

  describe('POST', () => {
    it('should create a new user', async () => {
      const request = new Request('http://localhost:3000/api/users', {
        method: 'POST',
        body: JSON.stringify({ name: 'New User', email: 'test@example.com' }),
      })

      const response = await POST(request)
      const data = await response.json()

      expect(response.status).toBe(201)
      expect(data).toHaveProperty('id')
      expect(data.name).toBe('New User')
    })

    it('should return 400 for invalid data', async () => {
      const request = new Request('http://localhost:3000/api/users', {
        method: 'POST',
        body: JSON.stringify({ name: '' }), // invalid
      })

      const response = await POST(request)
      expect(response.status).toBe(400)
    })
  })
})
```

---

## スナップショットテスト

```typescript
import { describe, it, expect } from 'vitest'
import { render } from '@testing-library/react'
import { UserCard } from './UserCard'

describe('UserCard', () => {
  it('should match snapshot', () => {
    const { container } = render(
      <UserCard user={{ id: 1, name: 'Test User', email: 'test@example.com' }} />
    )
    expect(container).toMatchSnapshot()
  })
})
```

---

## カバレッジ

### カバレッジレポートの生成

```bash
# カバレッジレポートを生成
npm run test:coverage

# カバレッジ閾値を設定（vitest.config.ts）
coverage: {
  thresholds: {
    lines: 80,
    functions: 80,
    branches: 80,
    statements: 80,
  },
}
```

### カバレッジから除外

```typescript
// vitest.config.ts
coverage: {
  exclude: [
    'node_modules/',
    '**/*.config.ts',
    '**/*.d.ts',
    '**/types/**',
    '**/__mocks__/**',
  ],
}
```

---

## ベストプラクティス

### 1. AAA パターンを使用

```typescript
it('should calculate total', () => {
  // Arrange - テストデータの準備
  const items = [{ price: 100 }]

  // Act - テスト対象の実行
  const result = calculateTotal(items)

  // Assert - 結果の検証
  expect(result).toBe(100)
})
```

### 2. 明確なテスト名

```typescript
// ❌ 悪い例
it('test 1', () => {})
it('works', () => {})

// ✅ 良い例
it('should return empty array when no items exist', () => {})
it('should throw error when user is not found', () => {})
```

### 3. 1テスト1アサーション（可能な限り）

```typescript
// ❌ 悪い例
it('should handle user', () => {
  expect(user.name).toBe('Test')
  expect(user.age).toBe(20)
  expect(user.email).toBe('test@example.com')
})

// ✅ 良い例
it('should have correct name', () => {
  expect(user.name).toBe('Test')
})

it('should have correct age', () => {
  expect(user.age).toBe(20)
})

it('should have correct email', () => {
  expect(user.email).toBe('test@example.com')
})
```

### 4. テストデータはテスト内で定義

```typescript
// ✅ 良い例
it('should format user name', () => {
  const user = { firstName: 'John', lastName: 'Doe' }
  expect(formatUserName(user)).toBe('John Doe')
})
```

### 5. 非同期テストは適切に処理

```typescript
// ✅ async/awaitを使用
it('should fetch user data', async () => {
  const user = await fetchUser(1)
  expect(user.id).toBe(1)
})
```

### 6. モックは必要最小限に

```typescript
// ❌ 過度なモック
vi.mock('./everything')

// ✅ 必要な部分のみモック
vi.mock('./api', () => ({
  fetchUser: vi.fn(),
}))
```

---

## テストコマンド

```bash
# すべてのテストを実行
npm run test

# ウォッチモード
npm run test:watch

# 特定のファイルをテスト
npm run test Button.test.tsx

# UIモード（ブラウザでテスト結果を確認）
npm run test:ui

# カバレッジレポート生成
npm run test:coverage
```

---

## トラブルシューティング

### 問題: `window is not defined`

```typescript
// vitest.config.ts
test: {
  environment: 'jsdom', // または 'happy-dom'
}
```

### 問題: モジュールが見つからない

```typescript
// vitest.config.ts
resolve: {
  alias: {
    '@': path.resolve(__dirname, './'),
  },
}
```

### 問題: 非同期テストがタイムアウト

```typescript
it('should handle long operation', async () => {
  // タイムアウトを延長
  await someVeryLongOperation()
}, 10000) // 10秒
```

---

## 参考リソース

- [Vitest公式ドキュメント](https://vitest.dev/)
- [Testing Library](https://testing-library.com/)
- [Jest互換API](https://vitest.dev/guide/migration.html)
