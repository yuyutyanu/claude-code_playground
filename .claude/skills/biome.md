# Biome 設定ガイド

このドキュメントは、Biomeを使用したコード品質管理のガイドラインを提供します。

## 概要

- **ツール**: Biome
- **目的**: Linting + Formatting（ESLint + Prettierの代替）
- **特徴**: Rustベースの高速実行、統一されたツールチェーン

---

## セットアップ

### インストール

```bash
npm install --save-dev @biomejs/biome
```

### 基本設定

```json
// biome.json
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
    "formatWithErrors": false,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100,
    "lineEnding": "lf"
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "jsxQuoteStyle": "double",
      "trailingComma": "all",
      "semicolons": "asNeeded",
      "arrowParentheses": "always",
      "bracketSpacing": true,
      "bracketSameLine": false
    }
  },
  "json": {
    "formatter": {
      "enabled": true
    }
  }
}
```

### package.jsonスクリプト

```json
{
  "scripts": {
    "lint": "biome lint .",
    "lint:fix": "biome lint --apply .",
    "format": "biome format --write .",
    "format:check": "biome format .",
    "check": "biome check .",
    "check:fix": "biome check --apply ."
  }
}
```

---

## 設定詳細

### Linterルール

#### 推奨ルールセット

```json
{
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "complexity": {
        "noExtraBooleanCast": "error",
        "noMultipleSpacesInRegularExpressionLiterals": "error",
        "noUselessCatch": "error",
        "noWith": "error"
      },
      "correctness": {
        "noConstAssign": "error",
        "noConstantCondition": "error",
        "noEmptyCharacterClassInRegex": "error",
        "noEmptyPattern": "error",
        "noGlobalObjectCalls": "error",
        "noInnerDeclarations": "error",
        "noInvalidConstructorSuper": "error",
        "noNewSymbol": "error",
        "noNonoctalDecimalEscape": "error",
        "noPrecisionLoss": "error",
        "noSelfAssign": "error",
        "noSetterReturn": "error",
        "noSwitchDeclarations": "error",
        "noUndeclaredVariables": "error",
        "noUnreachable": "error",
        "noUnreachableSuper": "error",
        "noUnsafeFinally": "error",
        "noUnsafeOptionalChaining": "error",
        "noUnusedLabels": "error",
        "noUnusedVariables": "error",
        "useIsNan": "error",
        "useValidForDirection": "error",
        "useYield": "error"
      },
      "suspicious": {
        "noAsyncPromiseExecutor": "error",
        "noCatchAssign": "error",
        "noClassAssign": "error",
        "noCompareNegZero": "error",
        "noControlCharactersInRegex": "error",
        "noDebugger": "error",
        "noDoubleEquals": "error",
        "noDuplicateCase": "error",
        "noDuplicateClassMembers": "error",
        "noDuplicateObjectKeys": "error",
        "noDuplicateParameters": "error",
        "noEmptyBlockStatements": "error",
        "noExplicitAny": "warn",
        "noExtraNonNullAssertion": "error",
        "noFallthroughSwitchClause": "error",
        "noFunctionAssign": "error",
        "noGlobalAssign": "error",
        "noImportAssign": "error",
        "noMisleadingCharacterClass": "error",
        "noPrototypeBuiltins": "error",
        "noRedeclare": "error",
        "noShadowRestrictedNames": "error",
        "noUnsafeNegation": "error",
        "useGetterReturn": "error",
        "useValidTypeof": "error"
      },
      "style": {
        "noArguments": "error",
        "noVar": "error",
        "useConst": "error",
        "useTemplate": "error"
      }
    }
  }
}
```

#### TypeScript固有のルール

```json
{
  "linter": {
    "rules": {
      "suspicious": {
        "noExplicitAny": "warn"
      }
    }
  }
}
```

### Formatterルール

```json
{
  "formatter": {
    "enabled": true,
    "formatWithErrors": false,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100,
    "lineEnding": "lf"
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "jsxQuoteStyle": "double",
      "trailingComma": "all",
      "semicolons": "asNeeded",
      "arrowParentheses": "always",
      "bracketSpacing": true,
      "bracketSameLine": false
    }
  }
}
```

### ファイル除外設定

```json
{
  "files": {
    "ignore": [
      "node_modules",
      "dist",
      "build",
      ".next",
      "coverage",
      "*.config.js",
      "*.config.ts"
    ]
  }
}
```

---

## 使用方法

### コマンドラインから実行

```bash
# Lintチェック
npx @biomejs/biome lint .

# Lint自動修正
npx @biomejs/biome lint --apply .

# フォーマットチェック
npx @biomejs/biome format .

# フォーマット適用
npx @biomejs/biome format --write .

# Lint + Format + Import整理を一括実行
npx @biomejs/biome check .

# 自動修正付き
npx @biomejs/biome check --apply .
```

### 特定のファイルのみ対象

```bash
# 特定のファイル
npx @biomejs/biome check src/components/Button.tsx

# 特定のディレクトリ
npx @biomejs/biome check src/components/

# 複数のパス
npx @biomejs/biome check src/ lib/
```

---

## エディタ統合

### VS Code

```bash
# VS Code拡張機能のインストール
code --install-extension biomejs.biome
```

#### settings.json

```json
{
  "[javascript]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "[json]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "editor.codeActionsOnSave": {
    "quickfix.biome": "explicit",
    "source.organizeImports.biome": "explicit"
  }
}
```

---

## CI/CD統合

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run check
```

### pre-commit hook（Husky + lint-staged）

```bash
# Huskyのインストール
npm install --save-dev husky lint-staged
npx husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx,json}": ["biome check --apply --no-errors-on-unmatched"]
  }
}
```

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

---

## ルール設定例

### React/Next.js プロジェクト向け

```json
{
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "a11y": {
        "useKeyWithClickEvents": "warn",
        "useAltText": "error"
      },
      "correctness": {
        "useExhaustiveDependencies": "warn",
        "useHookAtTopLevel": "error"
      },
      "suspicious": {
        "noExplicitAny": "warn"
      },
      "style": {
        "noNonNullAssertion": "warn",
        "useImportType": "error",
        "useNodejsImportProtocol": "error"
      }
    }
  }
}
```

### 厳格な設定

```json
{
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": {
        "noExplicitAny": "error"
      },
      "style": {
        "noNonNullAssertion": "error"
      },
      "correctness": {
        "noUnusedVariables": "error"
      }
    }
  }
}
```

---

## ベストプラクティス

### 1. 保存時の自動フォーマット

エディタで保存時に自動的にフォーマットを適用するよう設定します。

### 2. CI/CDでのチェック

プルリクエスト時に必ずlintとformatのチェックを実行します。

### 3. pre-commitフックの活用

コミット前に自動的にフォーマットとlintを実行し、問題を早期に発見します。

### 4. 段階的な導入

既存プロジェクトに導入する場合は、警告レベルから始めて徐々に厳格化します。

```json
{
  "linter": {
    "rules": {
      "suspicious": {
        "noExplicitAny": "warn" // 最初は警告
      }
    }
  }
}
```

### 5. チーム全体での設定統一

`.vscode/settings.json`をリポジトリに含めて、チーム全体で同じ設定を共有します。

---

## よくある問題

### 問題: 特定のファイルを除外したい

```json
{
  "files": {
    "ignore": ["generated/**", "legacy/**"]
  }
}
```

### 問題: 特定のルールを無効化したい

```json
{
  "linter": {
    "rules": {
      "suspicious": {
        "noExplicitAny": "off"
      }
    }
  }
}
```

### 問題: インラインでルールを無効化したい

```typescript
// biome-ignore lint/suspicious/noExplicitAny: Legacy code
const data: any = legacyFunction()

// 複数行
// biome-ignore lint/suspicious/noExplicitAny: <explanation>
// biome-ignore lint/correctness/noUnusedVariables: <explanation>
const unused: any = something()
```

### 問題: フォーマットとlintで競合する

Biomeはlintとformatが統合されているため、通常は競合しません。もし問題がある場合は設定を見直してください。

---

## マイグレーション

### ESLint + Prettierからの移行

1. 既存の設定を削除
   ```bash
   npm uninstall eslint prettier eslint-config-prettier
   rm .eslintrc.js .prettierrc
   ```

2. Biomeをインストール
   ```bash
   npm install --save-dev @biomejs/biome
   ```

3. 設定ファイルを作成
   ```bash
   npx @biomejs/biome init
   ```

4. package.jsonのスクリプトを更新

5. 一括フォーマット
   ```bash
   npm run format
   npm run lint:fix
   ```

---

## パフォーマンス

### 速度比較

Biomeは従来のツールと比較して非常に高速です：

- ESLint + Prettierの約10-20倍高速
- Rustで実装されており、並列処理を活用
- 大規模プロジェクトでも数秒で完了

### 最適化のヒント

1. **必要なファイルのみ対象**
   ```bash
   npx @biomejs/biome check src/ # node_modules等を除外
   ```

2. **並列実行**
   Biomeは自動的に並列処理を行いますが、手動で調整も可能です。

---

## 参考リソース

- [Biome公式ドキュメント](https://biomejs.dev/)
- [設定リファレンス](https://biomejs.dev/reference/configuration/)
- [Lintルール一覧](https://biomejs.dev/linter/rules/)
- [VS Code拡張](https://marketplace.visualstudio.com/items?itemName=biomejs.biome)
