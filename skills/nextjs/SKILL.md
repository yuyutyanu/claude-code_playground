---
name: nextjs-development
description: Next.js App Routerを使用した開発ガイド。Server Components、Client Components、データフェッチング、ルーティング、パフォーマンス最適化に関する指示。Next.jsコードを書く、Next.jsプロジェクトを作成する、Next.jsアプリケーションを最適化する際に使用。
---

# Next.js Development

Next.js 14以降（App Router）を使用した開発のベストプラクティス。

## コアルール

### 1. Server ComponentsとClient Components

**デフォルトはServer Component:**
- すべてのコンポーネントはデフォルトでServer Component
- `'use client'`は必要な場合のみ使用

**Client Componentが必要な場合:**
- useState、useEffect等のReact Hooks使用時
- ブラウザAPI使用時（window、localStorage等）
- イベントハンドラー使用時

**実装例:**
```tsx
// Server Component（デフォルト）
async function Page() {
  const data = await fetch('https://api.example.com/data')
  return <div>{data.title}</div>
}

// Client Component
'use client'
import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

### 2. データフェッチング

```tsx
// キャッシュあり（デフォルト、静的生成）
const data = await fetch('https://api.example.com/data')

// キャッシュなし（動的、リアルタイム）
const data = await fetch('https://api.example.com/data', { cache: 'no-store' })

// 再検証付きキャッシュ（ISR）
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }
})
```

### 3. ファイルベースルーティング

```
app/
├── page.tsx              # /
├── about/page.tsx        # /about
├── blog/[slug]/page.tsx  # /blog/:slug
└── (group)/page.tsx      # /page (ルートグループ)
```

### 4. メタデータ

```tsx
// 静的メタデータ
export const metadata = {
  title: 'Page Title',
  description: 'Page description',
}

// 動的メタデータ
export async function generateMetadata({ params }) {
  const data = await fetchData(params.id)
  return {
    title: data.title,
    description: data.description,
  }
}
```

### 5. パフォーマンス最適化

**画像:**
```tsx
import Image from 'next/image'

<Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />
```

**フォント:**
```tsx
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'] })

// layout.tsx
<html className={inter.className}>
```

**Dynamic Import:**
```tsx
import dynamic from 'next/dynamic'

const Heavy = dynamic(() => import('./Heavy'), {
  loading: () => <p>Loading...</p>,
})
```

## ベストプラクティス

1. **Server Componentをデフォルトに** - インタラクティビティが不要な場合は常にServer Component
2. **データフェッチングはサーバー側で** - SEO、セキュリティ、パフォーマンス向上
3. **適切なキャッシング戦略** - 静的/動的/ISRを適切に選択
4. **next/imageとnext/fontを活用** - 自動最適化を利用
5. **Suspenseでストリーミング** - ローディング状態を適切に管理

## アンチパターン（避ける）

❌ Server ComponentでクライアントサイドAPIを使用
❌ Client Componentで直接データベースアクセス
❌ 過度なClient Component化（必要な部分のみに限定）
❌ 不適切なキャッシング設定

## 詳細リファレンス

より詳細な情報は `.claude/skills/nextjs.md` を参照してください。
