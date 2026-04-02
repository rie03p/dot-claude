---
name: standup
description: git logとLinearから直近の作業サマリーを生成する。「昨日何やったっけ」「今日の作業まとめて」「スタンドアップ」「/standup」など、作業報告や振り返りを求められた場面で使う。
argument-hint: "[--days <日数>] [--format <slack|markdown|plain>]"
allowed-tools: Bash(git *), Bash(linear *), Bash(date*), Bash(gh *)
---

# スタンドアップサマリースキル

git log と Linear の情報から直近の作業内容をまとめて表示する。

## 引数の解析

- `--days <N>`: 過去N日分（デフォルト: 1、月曜なら3で金曜分を含む）
- `--format`: 出力形式（デフォルト: markdown）

## 手順

### 1. 期間の決定

```bash
# 今日が月曜なら過去3日、それ以外は過去1日
DAY_OF_WEEK=$(date +%u)
```

### 2. Git 情報の取得

現在のリポジトリと、作業ディレクトリ近辺の関連リポジトリから取得する。

```bash
# 現在のリポジトリのコミット
git log --author="$(git config user.name)" --since="<期間>" --oneline --no-merges

# ブランチの作成・マージ
git log --author="$(git config user.name)" --since="<期間>" --merges --oneline

# PR の作成・マージ
gh pr list --author=@me --state all --json number,title,state,createdAt,mergedAt
```

### 3. Linear 情報の取得

```bash
linear api '
query {
  viewer {
    assignedIssues(
      filter: { updatedAt: { gte: "<期間開始>" } }
      first: 20
      orderBy: updatedAt
    ) {
      nodes {
        identifier
        title
        state { name type }
        updatedAt
      }
    }
  }
}
'
```

### 4. サマリーの構成

```markdown
## 作業サマリー（YYYY/MM/DD〜YYYY/MM/DD）

### 完了した作業
- [PROJ-1234] タスクタイトル — コミットN件、PRマージ済み
- [PROJ-1235] タスクタイトル — 実装完了、レビュー待ち

### 進行中
- [PROJ-1236] タスクタイトル — ブランチ作成済み、コミットN件

### レビュー・PR
- PR #123「タイトル」— マージ済み
- PR #124「タイトル」— レビュー中

### コミット詳細
- repo-name: N件のコミット
  - abc1234 コミットメッセージ
  - def5678 コミットメッセージ
```

### 5. フォーマット出力

`--format` に応じて調整:
- **markdown**: そのまま表示（デフォルト）
- **slack**: Slack の mrkdwn 形式に変換（太字を `*text*` に）
- **plain**: 装飾なしのプレーンテキスト
