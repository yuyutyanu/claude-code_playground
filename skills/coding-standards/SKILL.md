---
name: coding-standards
description: TypeScript/JavaScript/Reactプロジェクトのコーディング規約。命名規則、型安全性、エラーハンドリング、コード構成の統一基準。
model: sonnet-4-5
---

# コーディング規約

## 基本原則

1. **可読性優先**: コードは書くより読む回数が多い
2. **KISS**: 最もシンプルな解決策を選ぶ
3. **DRY**: 同じロジックを繰り返さない（ただし早すぎる抽象化は避ける）
4. **YAGNI**: 今必要ないものは作らない

## 命名規則

| 対象 | 規則 | 例 |
|------|------|-----|
| コンポーネント | PascalCase | `UserProfile.tsx` |
| 関数・変数 | camelCase | `getUserById` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 型・インターフェース | PascalCase | `UserResponse` |
| ファイル（非コンポーネント） | camelCase | `authUtils.ts` |
| テストファイル | 対象と同名 + `.test` | `authUtils.test.ts` |

## 型安全性

```typescript
// NG: any型
function process(data: any) { ... }

// OK: 適切な型定義
interface UserData {
  id: string;
  name: string;
  email: string;
}
function process(data: UserData) { ... }
```

## イミュータブルな更新

```typescript
// NG: 直接変更
user.name = "新しい名前";
items.push(newItem);

// OK: スプレッド演算子
const updated = { ...user, name: "新しい名前" };
const newItems = [...items, newItem];
```

## エラーハンドリング

```typescript
// 非同期処理は必ずtry-catchで囲む
async function fetchUser(id: string): Promise<User> {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  } catch (error) {
    console.error("ユーザー取得失敗:", error);
    throw error;
  }
}
```

## 非同期処理の最適化

```typescript
// NG: 逐次実行
const users = await getUsers();
const posts = await getPosts();

// OK: 並列実行（依存関係がない場合）
const [users, posts] = await Promise.all([getUsers(), getPosts()]);
```

## コードの臭い（避けるべきパターン）

- 50行を超える関数
- 5段階以上のネスト
- 説明のないマジックナンバー
- `any`型の多用
- コメントアウトされたコード
- 未使用のimport・変数

## Reactコンポーネント

```typescript
// 関数コンポーネント + 適切な型
interface Props {
  title: string;
  onSubmit: (value: string) => void;
}

export function SearchForm({ title, onSubmit }: Props) {
  const [query, setQuery] = useState("");

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    onSubmit(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>{title}</h2>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <button type="submit">検索</button>
    </form>
  );
}
```
