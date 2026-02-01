---
name: prisma-patterns
description: Prisma ORMの実用パターン。スキーマ設計、クエリ最適化、リレーション、マイグレーション、型安全なDB操作。
model: sonnet-4-5
---

# Prismaパターン

## スキーマ設計

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  posts     Post[]
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("users")
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String   @map("author_id")
  tags      Tag[]
  createdAt DateTime @default(now()) @map("created_at")

  @@index([authorId])
  @@map("posts")
}
```

## クエリパターン

### リレーションの取得

```typescript
// Eager Loading（必要なリレーションだけ取得）
const user = await prisma.user.findUnique({
  where: { id },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: "desc" },
      take: 10,
    },
  },
});

// select で必要なフィールドだけ取得
const users = await prisma.user.findMany({
  select: { id: true, name: true, email: true },
});
```

### ページネーション

```typescript
// カーソルベース（推奨）
const posts = await prisma.post.findMany({
  take: 20,
  skip: 1, // カーソルの次から
  cursor: { id: lastPostId },
  orderBy: { createdAt: "desc" },
});

// オフセットベース（小規模データ向け）
const posts = await prisma.post.findMany({
  skip: (page - 1) * limit,
  take: limit,
});
```

### Upsert

```typescript
const user = await prisma.user.upsert({
  where: { email },
  update: { name },
  create: { email, name },
});
```

### トランザクション

```typescript
// 複数操作の一括実行
const [post, notification] = await prisma.$transaction([
  prisma.post.create({ data: { title, content, authorId } }),
  prisma.notification.create({ data: { userId: authorId, type: "POST_CREATED" } }),
]);

// インタラクティブトランザクション
await prisma.$transaction(async (tx) => {
  const sender = await tx.account.update({
    where: { id: senderId },
    data: { balance: { decrement: amount } },
  });
  if (sender.balance < 0) throw new Error("残高不足");
  await tx.account.update({
    where: { id: receiverId },
    data: { balance: { increment: amount } },
  });
});
```

## マイグレーション

```bash
# スキーマ変更後にマイグレーション作成
npx prisma migrate dev --name add_user_role

# 本番適用
npx prisma migrate deploy

# クライアント再生成
npx prisma generate
```

## アンチパターン

- `findMany`でリレーションを全取得（N+1の原因）→ `include`/`select`を使う
- スキーマに`@@index`を忘れる（外部キーには必須）
- `$queryRaw`の多用（型安全性が失われる）
- マイグレーションファイルの手動編集
