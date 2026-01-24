# Next.js 開発ガイド

このドキュメントは、Next.jsを使用した開発におけるベストプラクティスとガイドラインを提供します。

## バージョン

- **Next.js**: 14以降（App Routerを使用）
- **React**: 18以降

---

## プロジェクト構造

```
app/
├── (routes)/           # ルートグループ
│   ├── page.tsx       # ページコンポーネント
│   ├── layout.tsx     # レイアウトコンポーネント
│   └── loading.tsx    # ローディング状態
├── api/               # API Routes
│   └── route.ts       # Route Handlers
└── components/        # 共有コンポーネント
    ├── ui/           # UIコンポーネント
    └── features/     # 機能別コンポーネント

lib/
├── utils/            # ユーティリティ関数
├── hooks/            # カスタムフック
└── types/            # TypeScript型定義

public/               # 静的ファイル
```

---

## Server ComponentsとClient Components

### 基本方針

**デフォルトはServer Component**
- すべてのコンポーネントはデフォルトでServer Component
- 必要な場合のみClient Componentを使用

**Client Componentが必要な場合:**
- `useState`, `useEffect`などのReact Hooksを使用
- ブラウザAPIを使用（`window`, `localStorage`など）
- イベントハンドラーを使用
- インタラクティブな機能

### 実装例

```tsx
// Server Component (デフォルト)
// app/page.tsx
async function getData() {
  const res = await fetch('https://api.example.com/data')
  return res.json()
}

export default async function Page() {
  const data = await getData()
  return <div>{data.title}</div>
}
```

```tsx
// Client Component
// app/components/Counter.tsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

### コンポーネント分割のベストプラクティス

1. **Server ComponentでClient Componentをラップ**
   ```tsx
   // Server Component
   export default async function Page() {
     const data = await getData()
     return (
       <div>
         <ServerSideContent data={data} />
         <ClientSideInteraction />
       </div>
     )
   }
   ```

2. **Client Componentを最小限に保つ**
   - インタラクティブな部分のみClient Component化
   - 残りはServer Componentとして実装

3. **プロップスのシリアライズ可能性に注意**
   - Server ComponentからClient Componentに渡すプロップスはシリアライズ可能である必要がある
   - 関数やクラスインスタンスは渡せない

---

## データフェッチング

### fetch API（Server Component）

```tsx
// キャッシュあり（デフォルト）
const data = await fetch('https://api.example.com/data')

// キャッシュなし
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store'
})

// 再検証付きキャッシュ
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 } // 1時間ごとに再検証
})
```

### Server Actions

```tsx
// app/actions.ts
'use server'

export async function createUser(formData: FormData) {
  const name = formData.get('name')
  // データベース操作など
  return { success: true }
}
```

```tsx
// app/components/UserForm.tsx
'use client'

import { createUser } from '@/app/actions'

export function UserForm() {
  return (
    <form action={createUser}>
      <input name="name" />
      <button type="submit">Submit</button>
    </form>
  )
}
```

---

## ルーティング

### ファイルベースルーティング

```
app/
├── page.tsx              # /
├── about/
│   └── page.tsx         # /about
├── blog/
│   ├── page.tsx         # /blog
│   └── [slug]/
│       └── page.tsx     # /blog/:slug
└── (marketing)/
    └── pricing/
        └── page.tsx     # /pricing (ルートグループ)
```

### 動的ルート

```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default function BlogPost({ params }: PageProps) {
  return <div>Post: {params.slug}</div>
}

// 静的生成
export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```

### Parallel RoutesとIntercepting Routes

**Parallel Routes** - 複数のページを同時に表示
```
app/
├── @team/
│   └── page.tsx
├── @analytics/
│   └── page.tsx
└── layout.tsx
```

**Intercepting Routes** - モーダル表示など
```
app/
├── photo/
│   └── [id]/
│       └── page.tsx
└── (..)photo/
    └── [id]/
        └── page.tsx
```

---

## メタデータとSEO

### 静的メタデータ

```tsx
// app/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'My App',
  description: 'Welcome to my app',
}
```

### 動的メタデータ

```tsx
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.image],
    },
  }
}
```

---

## パフォーマンス最適化

### 画像最適化

```tsx
import Image from 'next/image'

// 推奨: next/imageを使用
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority // above the fold用
/>

// 外部画像の場合
<Image
  src="https://example.com/image.jpg"
  alt="External"
  width={800}
  height={400}
  // next.config.jsでドメインを許可
/>
```

### フォント最適化

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ja" className={inter.className}>
      <body>{children}</body>
    </html>
  )
}
```

### Dynamic Import（コード分割）

```tsx
import dynamic from 'next/dynamic'

const DynamicComponent = dynamic(() => import('@/components/HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false, // クライアントサイドのみ
})
```

### Suspenseとストリーミング

```tsx
// app/page.tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <div>
      <h1>My Page</h1>
      <Suspense fallback={<div>Loading posts...</div>}>
        <Posts />
      </Suspense>
    </div>
  )
}
```

---

## エラーハンドリング

### error.tsx

```tsx
// app/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

### not-found.tsx

```tsx
// app/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find requested resource</p>
    </div>
  )
}
```

---

## 環境変数

### 命名規則

- **クライアント側**: `NEXT_PUBLIC_` プレフィックス
- **サーバー側**: プレフィックスなし

```bash
# .env.local
DATABASE_URL=postgresql://...
NEXT_PUBLIC_API_URL=https://api.example.com
```

```tsx
// Server Component
const dbUrl = process.env.DATABASE_URL

// Client Component
const apiUrl = process.env.NEXT_PUBLIC_API_URL
```

---

## TypeScript

### 型定義

```tsx
// lib/types/index.ts
export interface User {
  id: string
  name: string
  email: string
}

export interface Post {
  id: string
  title: string
  content: string
  authorId: string
}
```

### ページとレイアウトの型

```tsx
// Next.jsの型を使用
import type { Metadata } from 'next'

interface PageProps {
  params: { id: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default function Page({ params, searchParams }: PageProps) {
  // ...
}
```

---

## ベストプラクティス

### 1. Server Componentをデフォルトに

インタラクティビティが不要な場合は常にServer Componentを使用。

### 2. データフェッチングはできるだけサーバー側で

- SEOの向上
- セキュリティの向上（APIキーなどをクライアントに露出しない）
- パフォーマンスの向上

### 3. キャッシング戦略を適切に設定

```tsx
// 静的データ
const data = await fetch('...', { cache: 'force-cache' })

// リアルタイムデータ
const data = await fetch('...', { cache: 'no-store' })

// 定期的に更新されるデータ
const data = await fetch('...', { next: { revalidate: 3600 } })
```

### 4. レイアウトで共通UIを共有

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <Header />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  )
}
```

### 5. loading.tsxでローディング状態を管理

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading dashboard...</div>
}
```

### 6. パス別名を使用

```tsx
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"]
    }
  }
}

// 使用例
import { Button } from '@/components/ui/Button'
import { formatDate } from '@/lib/utils'
```

---

## アンチパターン（避けるべきこと）

### ❌ Server ComponentでクライアントサイドのAPIを使用

```tsx
// NG
export default function Page() {
  const width = window.innerWidth // Error!
  return <div>{width}</div>
}
```

### ❌ Client Componentで直接データベースアクセス

```tsx
// NG
'use client'
import { db } from '@/lib/db'

export function Component() {
  const data = db.query(...) // Danger!
}
```

### ❌ 過度なClient Component化

```tsx
// NG - 全体をClient Component化
'use client'
export default function Page() {
  return (
    <div>
      <StaticContent />
      <InteractiveButton />
    </div>
  )
}

// OK - 必要な部分のみClient Component化
export default function Page() {
  return (
    <div>
      <StaticContent />
      <InteractiveButton /> {/* これだけがClient Component */}
    </div>
  )
}
```

---

## 参考リソース

- [Next.js公式ドキュメント](https://nextjs.org/docs)
- [Next.js App Router](https://nextjs.org/docs/app)
- [Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [Data Fetching](https://nextjs.org/docs/app/building-your-application/data-fetching)
