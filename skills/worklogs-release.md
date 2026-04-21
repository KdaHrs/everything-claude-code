---
description: weekly-report (WORKLOGS) プロジェクトのリリースワークフロー。ブランチ作成→修正→build確認→バージョンバンプ→PR作成→codexレビュー→マージ。
user-invocable: true
---

# /release — WORKLOGS リリースワークフロー

## When to Use
weekly-report プロジェクト（/Users/hk/Documents/cc/everything/）で修正・機能追加をデプロイする時。

## 前提
- worktree: `/Users/hk/Documents/cc/everything/`
- リポジトリ: `KdaHrs/weekly-report`
- Node 22 必須（`nvm use 22`）
- main への直接 push 禁止

## ワークフロー

以下を **上から順に1つずつ実行** する。スキップ禁止。

### Step 1: ブランチ作成
```bash
cd /Users/hk/Documents/cc/everything
git checkout main && git pull origin main
git checkout -b <type>/<short-description>
```
- type: `fix/`, `feat/`, `refactor/`, `chore/`
- 例: `fix/publish-button-zindex`, `feat/report-template`

### Step 2: コード修正
- 修正方針をユーザーに確認してから実装
- 最小差分で修正（大幅書き換え禁止）

### Step 3: ビルド確認
```bash
export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh" && nvm use 22
cd /Users/hk/Documents/cc/everything && npx next build
```
- **ビルド失敗ならStep 2に戻る。pushしない。**

### Step 4: バージョンバンプ
変更内容に応じて1つ選択:
```bash
npm run version:patch   # バグ修正、typo
npm run version:minor   # 新機能追加、UI大幅改善
npm run version:major   # 破壊的変更（DBスキーマ等）
```

### Step 5: コミット & Push
```bash
git add <files>
git commit -m "<type>: <説明>"
git push -u origin <branch-name>
```
- コミットメッセージは日本語
- 関連Issueがあれば `closes #N` を含める

### Step 6: PR作成
```bash
gh pr create --repo KdaHrs/weekly-report --title "<タイトル>" --body "$(cat <<'EOF'
## 変更内容
- ...

## テスト
- [ ] `npx next build` 成功
- [ ] 本番動作確認

🤖 Generated with Claude Code
EOF
)"
```

### Step 7: Codexレビュー
- code-reviewer エージェントで diff 全体をレビュー
- CRITICAL/HIGH の指摘があれば修正 → 再push → 再レビュー
- レビュー結果をユーザーに報告
- **ユーザーの承認を得てから次へ進む**

### Step 8: マージ（= 自動デプロイ）
```bash
gh pr merge --squash --repo KdaHrs/weekly-report
```
- Vercel auto-deploy で本番反映

### Step 8.5: 本番画面確認（必須、スキップ禁止）
- [ ] 1〜2分待ってVercelデプロイ完了
- [ ] Playwright MCPで https://everything-henna.vercel.app にアクセスして screenshot
- [ ] トップページが正常表示されることを確認
- [ ] CSP/middleware/rewrite等の横断変更なら複数ページ確認
- [ ] **確認完了後、必ず `browser_close` でPlaywrightブラウザを閉じる**
- **画面が壊れていたら即hotfix（revertまたは最小修正）**

### Step 9: レポート登録（minor以上のみ）
- patch → 不要
- minor/major → `seed-report.ts` でDBにレポート登録
  - 投稿者: 北田博保
  - 本文末尾に `## 担当者の感想` セクション

### Step 10: 後片付け
```bash
git checkout main && git pull origin main
git branch -d <branch-name>
```
