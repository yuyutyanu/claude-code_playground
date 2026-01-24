---
name: vitest-testing
description: Vitestを使用したテスト作成ガイド。ユニットテスト、統合テスト、Reactコンポーネントテスト、モック、カバレッジに関する指示。テストを書く、テストコードを生成する、テスト戦略を立てる際に使用。
---

# Vitest Testing

Vitestを使用した高速で効果的なテスト作成。

## テスト基本構造

### AAAパターン

```typescript
import { describe, it, expect } from 'vitest'

describe('機能名', () => {
  it('期待される動作を説明', () => {
    // Arrange（準備）
    const input = 'test'

    // Act（実行）
    const result = someFunction(input)

    // Assert（検証）
    expect(result).toBe('expected')
  })
})
```

## ユニットテスト

```typescript
// lib/utils.test.ts
import { describe, it, expect } from 'vitest'
import { formatDate, calculateTotal } from './utils'

describe('formatDate', () => {
  it('should format date correctly', () => {
    const date = new Date('2024-01-01')
    expect(formatDate(date)).toBe('2024年1月1日')
  })
})
```

## Reactコンポーネントテスト

```typescript
import { describe, it, expect, vi } from 'vitest'
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from './Button'

describe('Button', () => {
  it('should call onClick when clicked', async () => {
    const handleClick = vi.fn()
    const user = userEvent.setup()

    render(<Button onClick={handleClick}>Click me</Button>)
    await user.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

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
  fetchUser: vi.fn().mockResolvedValue({ id: 1, name: 'Test' }),
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
    vi.mocked(fetch).mockResolvedValueOnce({
      ok: true,
      json: async () => ({ id: 1, name: 'Test' }),
    } as Response)

    const result = await fetchUser(1)
    expect(result).toEqual({ id: 1, name: 'Test' })
  })
})
```

## Server ComponentsとAPIのテスト

```typescript
// Next.js Server Component
import { describe, it, expect, vi } from 'vitest'
import { render } from '@testing-library/react'
import UserList from './UserList'

global.fetch = vi.fn()

describe('UserList', () => {
  it('should render user list', async () => {
    vi.mocked(fetch).mockResolvedValueOnce({
      json: async () => [{ id: 1, name: 'User 1' }],
    } as Response)

    const { container } = render(await UserList())
    expect(container).toHaveTextContent('User 1')
  })
})
```

## ベストプラクティス

### 1. 明確なテスト名

```typescript
✅ it('should return empty array when no items exist', () => {})
❌ it('test 1', () => {})
```

### 2. 1テスト1アサーション（可能な限り）

```typescript
✅
it('should have correct name', () => {
  expect(user.name).toBe('Test')
})

it('should have correct age', () => {
  expect(user.age).toBe(20)
})

❌
it('should handle user', () => {
  expect(user.name).toBe('Test')
  expect(user.age).toBe(20)
  expect(user.email).toBe('test@example.com')
})
```

### 3. テストデータはテスト内で定義

```typescript
✅
it('should format user name', () => {
  const user = { firstName: 'John', lastName: 'Doe' }
  expect(formatUserName(user)).toBe('John Doe')
})
```

### 4. 非同期テストは適切に処理

```typescript
it('should fetch user data', async () => {
  const user = await fetchUser(1)
  expect(user.id).toBe(1)
})
```

## テストコマンド

```bash
npm run test              # すべてのテスト実行
npm run test:watch        # ウォッチモード
npm run test:coverage     # カバレッジレポート生成
```

## カバレッジ設定

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', '**/*.config.ts'],
    },
  },
})
```

## アンチパターン（避ける）

❌ 過度なモック（本当に必要な部分のみモック）
❌ 曖昧なテスト名
❌ 複数の概念を1つのテストに詰め込む
❌ テストデータの外部定義（テストが理解しにくくなる）

## 詳細リファレンス

より詳細な情報は `.claude/skills/vitest.md` を参照してください。
