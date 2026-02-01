---
name: security-review
description: Webアプリケーションのセキュリティレビューガイド。OWASP Top 10を中心に、認証・認可、入力検証、データ保護を体系的にチェックする。
model: sonnet-4-5
---

# セキュリティレビュー

## 秘密情報の管理

```typescript
// NG: ハードコード
const API_KEY = "sk-1234567890";

// OK: 環境変数
const API_KEY = process.env.API_KEY;
if (!API_KEY) throw new Error("API_KEY is required");
```

## 入力バリデーション

```typescript
import { z } from "zod";

const UserInput = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150),
});

// ファイルアップロード制限
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
const ALLOWED_TYPES = ["image/jpeg", "image/png", "image/webp"];
```

## SQLインジェクション防止

```typescript
// NG: 文字列結合
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// OK: パラメータ化クエリ
const result = await db.query("SELECT * FROM users WHERE id = $1", [userId]);

// OK: ORM使用
const user = await prisma.user.findUnique({ where: { id: userId } });
```

## XSS防止

- ユーザー入力をHTMLに埋め込む際はサニタイズ
- ReactはデフォルトでエスケープするがdangerouslySetInnerHTMLに注意
- Content Security Policyヘッダーを設定

```typescript
// Next.js next.config.js
const securityHeaders = [
  { key: "Content-Security-Policy", value: "default-src 'self'" },
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "X-Frame-Options", value: "DENY" },
  { key: "X-XSS-Protection", value: "1; mode=block" },
];
```

## 認証・認可

- トークンは`httpOnly` Cookieに保存（localStorageは使わない）
- CSRF対策としてSameSite属性を設定
- JWTの有効期限を適切に設定

```typescript
cookies().set("token", jwt, {
  httpOnly: true,
  secure: true,
  sameSite: "strict",
  maxAge: 60 * 60, // 1時間
});
```

## レート制限

```typescript
const WINDOW_MS = 15 * 60 * 1000; // 15分
const MAX_REQUESTS = 100;
```

API、ログイン、パスワードリセットなど重要なエンドポイントに適用。

## デプロイ前チェックリスト

- [ ] 秘密情報がコードに含まれていない
- [ ] すべてのユーザー入力がバリデーションされている
- [ ] SQLクエリがパラメータ化されている
- [ ] CSPヘッダーが設定されている
- [ ] 認証トークンがhttpOnly Cookieに保存されている
- [ ] レート制限が設定されている
- [ ] エラーメッセージに内部情報が含まれていない
- [ ] `npm audit` で重大な脆弱性がない
- [ ] ログに個人情報が出力されていない
