---
name: git-workflow
description: Gitワークフローとコミット規約のガイド。ブランチ戦略、Conventional Commits、PR作成、コンフリクト解決の標準手順。
model: sonnet-4-5
---

# Gitワークフロー

## ブランチ戦略

```
main          ← 本番環境（直接コミットしない）
├── develop   ← 開発統合ブランチ（任意）
├── feat/*    ← 機能ブランチ
├── fix/*     ← バグ修正ブランチ
└── chore/*   ← メンテナンスブランチ
```

## Conventional Commits

```
<type>(<scope>): <subject>

<body>

<footer>
```

### タイプ一覧

| タイプ | 用途 | 例 |
|--------|------|-----|
| `feat` | 新機能 | `feat(auth): ログイン機能を追加` |
| `fix` | バグ修正 | `fix(cart): 数量が0になるバグを修正` |
| `docs` | ドキュメント | `docs: READMEを更新` |
| `refactor` | リファクタリング | `refactor(api): エラーハンドリングを統一` |
| `test` | テスト追加・修正 | `test(auth): ログインテストを追加` |
| `chore` | ビルド・ツール | `chore: 依存パッケージを更新` |
| `style` | フォーマット | `style: インデントを修正` |
| `perf` | パフォーマンス改善 | `perf(query): N+1クエリを解消` |

### コミットメッセージの原則

- **「何を」より「なぜ」** を書く
- 命令形で書く（「追加した」ではなく「追加」）
- 1行目は50文字以内
- 本文は72文字で折り返し

## PR作成のベストプラクティス

1. 変更を小さく保つ（レビューしやすいサイズ）
2. 1つのPRに1つの目的
3. テストを含める
4. スクリーンショットがあれば添付（UI変更時）

### PRテンプレート

```markdown
## 概要
何をしたか1-3行で

## 変更内容
- 変更点1
- 変更点2

## テスト
- [ ] ユニットテスト通過
- [ ] 手動テスト完了
```

## コンフリクト解決

```bash
# 最新のmainを取得
git fetch origin main

# リベース（コミット履歴をきれいに保つ）
git rebase origin/main

# コンフリクト解決後
git add <resolved-files>
git rebase --continue
```

## やってはいけないこと

- mainブランチへの直接プッシュ
- `--force` プッシュ（共有ブランチ）
- 秘密情報のコミット（`.env`等）
- 巨大なバイナリファイルのコミット
