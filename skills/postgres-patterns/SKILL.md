---
name: postgres-patterns
description: PostgreSQLの設計・運用パターン。スキーマ設計、インデックス戦略、クエリ最適化、RLS、マイグレーション。
model: sonnet-4-5
---

# PostgreSQLパターン

## データ型の選択

| 用途 | 推奨型 | 備考 |
|------|--------|------|
| ID | `bigint` / `uuid` | 自動採番なら`bigint`、分散なら`uuid` |
| 文字列 | `text` | `varchar(n)`は特別な理由がない限り不要 |
| 日時 | `timestamptz` | タイムゾーン付き必須 |
| 金額 | `numeric` | 浮動小数点は使わない |
| フラグ | `boolean` | |
| JSON | `jsonb` | `json`ではなく`jsonb` |

## インデックス戦略

| クエリパターン | インデックス種別 |
|----------------|------------------|
| `WHERE col = ?` | B-tree |
| `WHERE col IN (?, ?)` | B-tree |
| `WHERE col BETWEEN ? AND ?` | B-tree |
| `WHERE jsonb_col @> ?` | GIN |
| 全文検索 | GIN + `tsvector` |
| 時系列データ | BRIN |

### 複合インデックスの順序

```sql
-- 等値条件のカラムを先に、範囲条件を後に
CREATE INDEX idx_orders_status_date
  ON orders (status, created_at);

-- 部分インデックス（特定条件のみ）
CREATE INDEX idx_active_users
  ON users (email) WHERE status = 'active';

-- カバリングインデックス（テーブルルックアップ回避）
CREATE INDEX idx_users_email_name
  ON users (email) INCLUDE (name);
```

## クエリ最適化

### N+1問題の防止

```sql
-- NG: ループ内で個別クエリ
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE id = 2;

-- OK: バッチ取得
SELECT * FROM users WHERE id = ANY($1::bigint[]);
```

### カーソルベースのページネーション

```sql
-- OFFSETベース（遅い: O(n)）
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 1000;

-- カーソルベース（速い: O(1)）
SELECT * FROM posts WHERE id > $1 ORDER BY id LIMIT 20;
```

## Row Level Security (RLS)

```sql
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "ユーザーは自分の投稿のみ閲覧可能"
  ON posts FOR SELECT
  USING (user_id = auth.uid());

CREATE POLICY "ユーザーは自分の投稿のみ更新可能"
  ON posts FOR UPDATE
  USING (user_id = auth.uid());
```

## トランザクション

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
  UPDATE accounts SET balance = balance + 1000 WHERE id = 2;
COMMIT;
```

ORMの場合:

```typescript
await prisma.$transaction([
  prisma.account.update({ where: { id: 1 }, data: { balance: { decrement: 1000 } } }),
  prisma.account.update({ where: { id: 2 }, data: { balance: { increment: 1000 } } }),
]);
```

## アンチパターン検出

```sql
-- 未インデックスの外部キー
SELECT c.conrelid::regclass, a.attname
FROM pg_constraint c
JOIN pg_attribute a ON a.attnum = ANY(c.conkey) AND a.attrelid = c.conrelid
WHERE c.contype = 'f'
AND NOT EXISTS (
  SELECT 1 FROM pg_index i WHERE i.indrelid = c.conrelid AND a.attnum = ANY(i.indkey)
);
```

## アンチパターン

- `varchar(255)` の無意味な長さ制限
- タイムゾーンなしの`timestamp`
- 金額に`float`/`double`を使用
- OFFSETベースのページネーション（大量データ時）
- インデックスなしの外部キー
