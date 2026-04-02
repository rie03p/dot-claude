---
name: standup
description: |
  git logとLinearから直近の作業サマリーを生成するスキル。以下のような場面で積極的に使うこと:
  - 「昨日何やったっけ」「今日の作業まとめて」「スタンドアップ」
  - 「日報書きたい」「週報の材料が欲しい」「作業報告して」
  - 「最近何してたか振り返りたい」「進捗まとめて」
  - 「/standup」「/standup --days 3」のように直接呼び出されたとき
  - 朝会やデイリースクラムの準備をしているとき
  - 「今週の成果をまとめて」「先週やったことリストアップして」
  作業の振り返り、報告、サマリー作成を求められたら、このスキルを使うこと。
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
# 月曜は過去3日にすることで、金曜の作業も含まれるようにする
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

Git のコミットと Linear のチケットを突き合わせて、作業内容を統合する（コミットだけでは文脈がわからず、チケットだけでは具体的な作業量がわからないため、両方を組み合わせる）。

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

`--format` に応じて調整する:
- **markdown**: そのまま表示（デフォルト）
- **slack**: Slack の mrkdwn 形式に変換する（Slack は独自の記法を使うため、`**text**` を `*text*` に変換する等の調整が必要）
- **plain**: 装飾なしのプレーンテキスト

## 使用例

**例1**: ユーザーが朝「昨日何やったっけ」と言った場合
→ 過去1日分（月曜なら3日分）のコミットとLinearチケットを取得し、作業サマリーを表示する。

**例2**: ユーザーが「週報を書きたいので今週の作業をまとめて」と言った場合
→ `--days 5` で過去5日分を取得し、markdown形式で出力する。

**例3**: ユーザーが「/standup --format slack」と呼び出した場合
→ 直近の作業をSlack形式でまとめて出力する。そのままSlackに貼り付けられる形式にする。
