---
name: performance-optimization
description: Webアプリのパフォーマンス最適化ガイド。Core Web Vitals、バンドルサイズ、レンダリング最適化、データフェッチング、画像最適化。
model: sonnet-4-5
---

# パフォーマンス最適化

## Core Web Vitals

| 指標 | 目標 | 意味 |
|------|------|------|
| LCP | < 2.5秒 | 最大コンテンツの表示時間 |
| INP | < 200ms | インタラクションの応答性 |
| CLS | < 0.1 | レイアウトのズレ |

## バンドルサイズ最適化

### コード分割

```tsx
// ルートベースの分割（Next.js App Routerは自動）
// 手動での遅延読み込み
import dynamic from "next/dynamic";

const HeavyEditor = dynamic(() => import("@/components/Editor"), {
  loading: () => <p>読み込み中...</p>,
  ssr: false, // クライアントのみ
});
```

### Tree Shaking

```typescript
// NG: 名前空間全体のインポート
import * as lodash from "lodash";
lodash.debounce(...);

// OK: 個別インポート
import debounce from "lodash/debounce";
```

### バンドル分析

```bash
# Next.js
ANALYZE=true npm run build
# package.json に @next/bundle-analyzer の設定が必要
```

## レンダリング最適化

### 不要な再レンダリングを防ぐ

```tsx
// 1. stateを必要なコンポーネントに近づける
// NG: 親コンポーネントでstateを持つ
function Parent() {
  const [search, setSearch] = useState(""); // 子全体が再レンダリング
  return <><SearchBox value={search} onChange={setSearch} /><HeavyList /></>;
}

// OK: stateを分離
function Parent() {
  return <><SearchBox /><HeavyList /></>;
}
function SearchBox() {
  const [search, setSearch] = useState(""); // SearchBoxだけ再レンダリング
  return <input value={search} onChange={(e) => setSearch(e.target.value)} />;
}
```

### リスト仮想化

```tsx
// 大量リスト（1000件以上）は仮想化
import { useVirtualizer } from "@tanstack/react-virtual";

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: "auto" }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map((virtual) => (
          <div key={virtual.key} style={{ height: virtual.size, transform: `translateY(${virtual.start}px)` }}>
            {items[virtual.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 画像最適化

```tsx
// Next.js Image
import Image from "next/image";

<Image
  src="/hero.jpg"
  alt="説明テキスト"
  width={1200}
  height={630}
  priority       // LCPに含まれる画像
  placeholder="blur"
/>
```

- WebP/AVIF形式を使用
- 適切なsizes属性を設定
- ファーストビュー外の画像は遅延読み込み（デフォルト）

## データフェッチング最適化

```typescript
// 並列フェッチ
const [users, posts] = await Promise.all([
  fetch("/api/users"),
  fetch("/api/posts"),
]);

// Next.js: 適切なキャッシュ戦略
fetch(url, { next: { revalidate: 3600 } }); // ISR
fetch(url, { cache: "no-store" });            // 常に最新
```

## 計測ツール

- Lighthouse（Chrome DevTools）
- Web Vitals（`next/web-vitals`）
- Bundle Analyzer
