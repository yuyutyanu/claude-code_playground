# Skills

このディレクトリには、[Anthropic Agent Skills](https://github.com/anthropics/skills)形式のスキルが含まれています。

## スキル形式

各スキルは以下の構造を持ちます：

```
skill-name/
└── SKILL.md          # YAML frontmatter + Markdown本文
```

## 利用可能なスキル

### Development Skills

- **skill-creator** - スキル作成ガイド
  - 効果的なスキルの作成方法
  - ベストプラクティスとパターン

- **nextjs** - Next.js開発スキル
  - App Router、Server/Client Components
  - データフェッチング、ルーティング

- **vitest** - Vitestテストスキル
  - ユニットテスト、統合テスト
  - モック、カバレッジ

- **biome** - Biomeスキル
  - Linting、Formatting
  - CI/CD統合

## 使用方法

### Claude Codeで使用

```bash
# このリポジトリをマーケットプレイスに追加
/plugin marketplace add yuyutyanu/claude-code_playground

# スキルをインストール
/plugin install nextjs@claude-code_playground
/plugin install vitest@claude-code_playground
/plugin install biome@claude-code_playground
```

### Claude.aiで使用

1. スキルフォルダをzipファイルに圧縮
2. Claude.aiの設定からスキルをアップロード
3. 対話でスキルを有効化

### Claude APIで使用

API経由でスキルをロードして使用できます。詳細は[Agent Skills API Guide](https://docs.claude.com/en/api/skills-guide)を参照してください。

## スキルと詳細ガイドの使い分け

- **skills/** (このディレクトリ) - 簡潔な指示、実行用（数十行）
- **.claude/skills/** - 詳細なリファレンス、参照用（数百行）

スキルは簡潔さを重視し、詳細な情報は`.claude/skills/`の対応するファイルを参照します。

## カスタムスキルの作成

新しいスキルを作成する場合は、`skill-creator`スキルを参照してください：

```
skills/skill-creator/SKILL.md
```

## 参考リソース

- [Anthropic Agent Skills Repository](https://github.com/anthropics/skills)
- [Agent Skills Documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Skills API Guide](https://docs.claude.com/en/api/skills-guide)
