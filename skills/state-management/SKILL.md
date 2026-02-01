---
name: state-management
description: React状態管理パターン。useState/useReducer、Context、Zustand、Server State（TanStack Query）の使い分けと実装パターン。
model: sonnet-4-5
---

# 状態管理

## 状態の種類と選択基準

| 状態の種類 | スコープ | 推奨ツール |
|-----------|---------|-----------|
| UIローカル状態 | 1コンポーネント | `useState` |
| 複雑なローカル状態 | 1コンポーネント | `useReducer` |
| 共有UI状態 | 複数コンポーネント | Zustand / Context |
| サーバー状態 | API由来 | TanStack Query |
| URL状態 | URLパラメータ | `useSearchParams` |
| フォーム状態 | フォーム | React Hook Form |

## useState / useReducer

```typescript
// シンプルな状態
const [count, setCount] = useState(0);

// 複雑な状態（複数の関連するstate）
type Action =
  | { type: "add"; item: CartItem }
  | { type: "remove"; id: string }
  | { type: "clear" };

function cartReducer(state: CartItem[], action: Action): CartItem[] {
  switch (action.type) {
    case "add":
      return [...state, action.item];
    case "remove":
      return state.filter((item) => item.id !== action.id);
    case "clear":
      return [];
  }
}

const [cart, dispatch] = useReducer(cartReducer, []);
```

## Zustand（軽量グローバル状態）

```typescript
import { create } from "zustand";

interface AuthStore {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null }),
}));

// コンポーネントで使用
function Header() {
  const user = useAuthStore((s) => s.user); // セレクタで必要な部分だけ購読
  const logout = useAuthStore((s) => s.logout);
  return user ? <button onClick={logout}>ログアウト</button> : <LoginLink />;
}
```

### Zustand + Persist

```typescript
import { persist } from "zustand/middleware";

const useSettingsStore = create<SettingsStore>()(
  persist(
    (set) => ({
      theme: "light",
      setTheme: (theme: string) => set({ theme }),
    }),
    { name: "settings" } // localStorageのキー
  )
);
```

## TanStack Query（サーバー状態）

```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

// データ取得
function useUser(id: string) {
  return useQuery({
    queryKey: ["users", id],
    queryFn: () => fetch(`/api/users/${id}`).then((r) => r.json()),
    staleTime: 5 * 60 * 1000, // 5分間はキャッシュを使用
  });
}

// データ更新
function useUpdateUser() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: UpdateUser) =>
      fetch(`/api/users/${data.id}`, { method: "PUT", body: JSON.stringify(data) }),
    onSuccess: (_, variables) => {
      queryClient.invalidateQueries({ queryKey: ["users", variables.id] });
    },
  });
}
```

## 選択の指針

1. **まず`useState`で始める** — 必要になるまでグローバル状態を使わない
2. **サーバー状態はTanStack Query** — 自前でキャッシュ管理しない
3. **グローバルUI状態が必要ならZustand** — Contextより再レンダリングが少ない
4. **Contextは低頻度更新のみ** — テーマ、ロケール等

## アンチパターン

- すべてをグローバル状態に入れる
- サーバー状態を`useState`で管理（キャッシュ・再取得が困難）
- Contextに高頻度更新の値を入れる（全消費コンポーネントが再レンダリング）
- 状態の重複（同じデータを複数箇所で管理）
