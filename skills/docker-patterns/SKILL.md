---
name: docker-patterns
description: Docker/コンテナ開発パターン。Dockerfile最適化、docker-compose設定、マルチステージビルド、開発環境構築。
model: sonnet-4-5
---

# Dockerパターン

## Dockerfileのベストプラクティス

### マルチステージビルド（Node.js）

```dockerfile
# ---- ビルドステージ ----
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ---- 実行ステージ ----
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# 非rootユーザーで実行
RUN addgroup -S app && adduser -S app -G app
COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER app
EXPOSE 3000
CMD ["node", "server.js"]
```

### レイヤーキャッシュの活用

```dockerfile
# 依存関係を先にコピー（変更頻度が低い）
COPY package*.json ./
RUN npm ci

# ソースコードは後にコピー（変更頻度が高い）
COPY . .
```

## docker-compose（開発環境）

```yaml
version: "3.8"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules  # node_modulesはコンテナ内を使用
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

## .dockerignore

```
node_modules
.next
.git
*.md
.env*
coverage
dist
```

## セキュリティ

1. **非rootユーザー**: `USER app` で実行
2. **最小イメージ**: `alpine`ベースを使用
3. **秘密情報**: ビルド引数やイメージに含めない（実行時に環境変数で渡す）
4. **脆弱性スキャン**: `docker scout` や `trivy` で定期チェック

## よく使うコマンド

```bash
# ビルド＆起動
docker compose up -d --build

# ログ確認
docker compose logs -f app

# コンテナに入る
docker compose exec app sh

# 全停止＆削除
docker compose down -v
```

## アンチパターン

- rootユーザーでの実行
- `.dockerignore`の未設定（node_modules等がコピーされる）
- `latest`タグの使用（バージョンを固定する）
- 1コンテナに複数プロセス
- 秘密情報をDockerfileに記述
