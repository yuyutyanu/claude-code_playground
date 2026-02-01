---
name: api-design
description: RESTful API設計のベストプラクティス。エンドポイント設計、リクエスト/レスポンス形式、バリデーション、バージョニング、ドキュメント。
model: sonnet-4-5
---

# API設計

## エンドポイント設計原則

1. **名詞を使う**: `/api/users`（動詞は使わない: `/api/getUsers`）
2. **複数形**: `/api/users`（単数形ではない: `/api/user`）
3. **ネスト**: `/api/users/:userId/posts`（ユーザーの投稿）
4. **2階層まで**: 深いネストは避ける

## HTTPメソッドとステータスコード

| メソッド | 用途 | 成功コード |
|----------|------|------------|
| GET | 取得 | 200 |
| POST | 作成 | 201 |
| PUT | 全体更新 | 200 |
| PATCH | 部分更新 | 200 |
| DELETE | 削除 | 204 |

### エラーコード

| コード | 意味 | 使用場面 |
|--------|------|----------|
| 400 | Bad Request | バリデーションエラー |
| 401 | Unauthorized | 未認証 |
| 403 | Forbidden | 権限不足 |
| 404 | Not Found | リソースなし |
| 409 | Conflict | 重複 |
| 429 | Too Many Requests | レート制限超過 |
| 500 | Internal Server Error | サーバーエラー |

## リクエスト/レスポンス設計

### ページネーション

```
GET /api/users?page=1&limit=20&sort=createdAt&order=desc
```

```json
{
  "data": [...],
  "meta": {
    "total": 150,
    "page": 1,
    "limit": 20,
    "totalPages": 8
  }
}
```

### フィルタリング

```
GET /api/users?status=active&role=admin&createdAfter=2024-01-01
```

### バリデーション（Zod）

```typescript
import { z } from "zod";

const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(["user", "admin"]).default("user"),
});

export async function POST(req: NextRequest) {
  const body = await req.json();
  const parsed = CreateUserSchema.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json(
      { error: { code: "VALIDATION", details: parsed.error.flatten() } },
      { status: 400 }
    );
  }
  // parsed.data は型安全
}
```

## バージョニング

```
/api/v1/users   # URLパスによるバージョニング
```

- 破壊的変更がある場合のみバージョンを上げる
- 旧バージョンは非推奨期間を設けてから廃止

## エラーレスポンスの統一

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "入力内容に問題があります",
    "details": [
      { "field": "email", "message": "有効なメールアドレスを入力してください" }
    ]
  }
}
```

## アンチパターン

- 動詞のエンドポイント（`/api/createUser`）
- すべて200で返してbodyにエラーを入れる
- ネストが深すぎるURL（3階層以上）
- レスポンス形式がエンドポイントごとにバラバラ
