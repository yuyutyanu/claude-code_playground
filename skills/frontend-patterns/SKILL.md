---
name: frontend-patterns
description: React/Next.jsのフロントエンド設計パターン集。コンポーネント設計、カスタムフック、状態管理、パフォーマンス最適化、アクセシビリティ。
model: sonnet-4-5
---

# フロントエンドパターン

## コンポーネント設計

### Composition（合成）パターン

```tsx
// 柔軟に組み合わせ可能なカードコンポーネント
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}
Card.Header = ({ children }: { children: React.ReactNode }) => (
  <div className="card-header">{children}</div>
);
Card.Body = ({ children }: { children: React.ReactNode }) => (
  <div className="card-body">{children}</div>
);

// 使用例
<Card>
  <Card.Header>タイトル</Card.Header>
  <Card.Body>コンテンツ</Card.Body>
</Card>
```

### Compound Componentパターン

```tsx
const TabsContext = createContext<{ active: string; setActive: (v: string) => void } | null>(null);

function Tabs({ children, defaultTab }: { children: ReactNode; defaultTab: string }) {
  const [active, setActive] = useState(defaultTab);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
}

Tabs.Tab = ({ id, children }: { id: string; children: ReactNode }) => {
  const ctx = useContext(TabsContext)!;
  return <button onClick={() => ctx.setActive(id)} data-active={ctx.active === id}>{children}</button>;
};

Tabs.Panel = ({ id, children }: { id: string; children: ReactNode }) => {
  const ctx = useContext(TabsContext)!;
  return ctx.active === id ? <div>{children}</div> : null;
};
```

## カスタムフック

### useDebounce

```typescript
function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}
```

### useFetch

```typescript
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();
    fetch(url, { signal: controller.signal })
      .then((res) => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
    return () => controller.abort();
  }, [url]);

  return { data, error, loading };
}
```

## パフォーマンス最適化

### メモ化

```tsx
// 重い計算
const sorted = useMemo(() => items.sort((a, b) => a.price - b.price), [items]);

// コールバック
const handleClick = useCallback((id: string) => dispatch({ type: "select", id }), []);

// コンポーネント（propsが変わらなければ再レンダリングしない）
const MemoizedList = memo(({ items }: { items: Item[] }) => (
  <ul>{items.map((item) => <li key={item.id}>{item.name}</li>)}</ul>
));
```

### 遅延読み込み

```tsx
const HeavyChart = lazy(() => import("./HeavyChart"));

function Dashboard() {
  return (
    <Suspense fallback={<div>読み込み中...</div>}>
      <HeavyChart />
    </Suspense>
  );
}
```

## アクセシビリティ

- すべてのインタラクティブ要素にキーボード操作を実装
- 適切なARIA属性を使用
- フォーカス管理（モーダル、ドロップダウン）
- カラーコントラスト比4.5:1以上

```tsx
function Modal({ isOpen, onClose, children }: ModalProps) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (isOpen) ref.current?.focus();
  }, [isOpen]);

  if (!isOpen) return null;
  return (
    <div role="dialog" aria-modal="true" ref={ref} tabIndex={-1}
      onKeyDown={(e) => e.key === "Escape" && onClose()}>
      {children}
    </div>
  );
}
```

## エラーバウンダリ

```tsx
class ErrorBoundary extends Component<{ children: ReactNode; fallback: ReactNode }, { hasError: boolean }> {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  render() { return this.state.hasError ? this.props.fallback : this.props.children; }
}
```
