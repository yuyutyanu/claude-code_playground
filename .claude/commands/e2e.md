E2Eテストを作成・実行する。

## 手順

1. **ユーザーフローの分析**
   - テスト対象の操作フローを特定
   - 前提条件・テストデータを整理

2. **テスト生成**
   - Page Object Modelパターンでテストを作成
   - `data-testid` 属性でUI要素を特定
   - 任意のsleepではなく、APIレスポンスの明示的な待機を使用

3. **テスト実行**
   - 複数ブラウザで実行（Chrome, Firefox, Safari）
   - 失敗時にスクリーンショット・動画・トレースを収集

4. **結果レポート**
   - 通過/失敗のサマリー
   - 失敗テストの詳細とスクリーンショット
   - 不安定なテストの検出・隔離

## テスト対象の優先度

1. 認証フロー（ログイン/ログアウト/登録）
2. 主要なビジネスフロー（決済、注文等）
3. CRUD操作
4. ナビゲーション・ルーティング

## Playwrightの例

```typescript
import { test, expect } from '@playwright/test';

test('ユーザーがログインできる', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('メール').fill('test@example.com');
  await page.getByLabel('パスワード').fill('password123');
  await page.getByRole('button', { name: 'ログイン' }).click();
  await expect(page.getByRole('heading', { name: 'ダッシュボード' })).toBeVisible();
});
```

## 重要

- 本番環境でE2Eテストを実行しない（staging/testnet環境を使用）
- テストデータは各テストで独立して管理する
- 不安定なテストは修正するか隔離する

$ARGUMENTS: テスト対象のフローや画面の説明
