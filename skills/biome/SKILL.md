---
name: biome-linting
description: Biomeを使用したコード品質管理ガイド。linting、formatting、設定、CI/CD統合に関する指示。Biome設定ファイルを作成する、コードをフォーマットする、lintルールを設定する際に使用。
---

# Biome Linting & Formatting

Biomeを使用した高速で統一されたコード品質管理。

## 基本設定

### biome.json

```json
{
  "$schema": "https://biomejs.dev/schemas/1.5.3/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "jsxQuoteStyle": "double",
      "trailingComma": "all",
      "semicolons": "asNeeded"
    }
  }
}
```

## コマンド

```bash
# Lintチェック
biome lint .

# Lint自動修正
biome lint --apply .

# フォーマット適用
biome format --write .

# Lint + Format + Import整理を一括実行
biome check .

# 自動修正付き
biome check --apply .
```

## package.jsonスクリプト

```json
{
  "scripts": {
    "lint": "biome lint .",
    "lint:fix": "biome lint --apply .",
    "format": "biome format --write .",
    "check": "biome check .",
    "check:fix": "biome check --apply ."
  }
}
```

## 主要なLintルール

### TypeScript/JavaScript

```json
{
  "linter": {
    "rules": {
      "correctness": {
        "noUnusedVariables": "error",
        "noUndeclaredVariables": "error"
      },
      "suspicious": {
        "noExplicitAny": "warn",
        "noDoubleEquals": "error"
      },
      "style": {
        "noVar": "error",
        "useConst": "error",
        "useTemplate": "error"
      }
    }
  }
}
```

### React/Next.js向け

```json
{
  "linter": {
    "rules": {
      "a11y": {
        "useKeyWithClickEvents": "warn",
        "useAltText": "error"
      },
      "correctness": {
        "useExhaustiveDependencies": "warn",
        "useHookAtTopLevel": "error"
      }
    }
  }
}
```

## ファイル除外

```json
{
  "files": {
    "ignore": [
      "node_modules",
      "dist",
      "build",
      ".next",
      "coverage"
    ]
  }
}
```

## インラインでルールを無効化

```typescript
// biome-ignore lint/suspicious/noExplicitAny: Legacy code
const data: any = legacyFunction()

// 複数行
// biome-ignore lint/suspicious/noExplicitAny: <explanation>
// biome-ignore lint/correctness/noUnusedVariables: <explanation>
const unused: any = something()
```

## CI/CD統合

### GitHub Actions

```yaml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run check
```

### pre-commit (Husky + lint-staged)

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx,json}": ["biome check --apply --no-errors-on-unmatched"]
  }
}
```

## VS Code統合

```json
// .vscode/settings.json
{
  "[javascript]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "editor.codeActionsOnSave": {
    "quickfix.biome": "explicit",
    "source.organizeImports.biome": "explicit"
  }
}
```

## ベストプラクティス

1. **保存時の自動フォーマット** - エディタで保存時に自動適用
2. **CI/CDでチェック** - プルリクエスト時に必ず実行
3. **pre-commitフック** - コミット前に自動修正
4. **段階的導入** - 既存プロジェクトでは警告から開始

## ESLint + Prettierからの移行

```bash
# 既存ツールをアンインストール
npm uninstall eslint prettier eslint-config-prettier

# Biomeをインストール
npm install --save-dev @biomejs/biome

# 初期化
npx @biomejs/biome init

# 一括フォーマット
npm run format
npm run lint:fix
```

## アンチパターン（避ける）

❌ biome-ignoreの乱用（根本的な問題を解決する）
❌ 過度に寛容な設定（コード品質が低下）
❌ CI/CDでのチェックスキップ（品質を保証できない）

## 詳細リファレンス

より詳細な情報は `.claude/skills/biome.md` を参照してください。
