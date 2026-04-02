---
name: review-pr
description: GitHub PRの差分を取得し、変更内容を整理して表示する。「PRを見せて」「このPRの内容は？」「/review-pr 123」など、PRの内容確認を求められた場面で使う。
argument-hint: "<PR番号 または PR URL>"
allowed-tools: Bash(gh *), Bash(git *), Read
---

# PR レビュー表示スキル

指定された GitHub PR の差分を取得し、変更内容を構造化して表示する。コメントの生成や投稿は行わない。

## 引数の解析

`$ARGUMENTS` から PR 番号または URL を読み取る。未指定ならユーザーに聞く。

## 手順

### 1. PR 情報の取得

```bash
# PR のメタ情報
gh pr view <PR番号> --json title,body,state,author,baseRefName,headRefName,labels,reviewRequests,additions,deletions,changedFiles,comments,reviews

# 差分の取得
gh pr diff <PR番号>
```

### 2. 変更の整理

以下の構成で表示する:

**概要**
- タイトル / 作者 / ベースブランチ → ヘッドブランチ
- 状態 / ラベル / レビュアー
- ファイル数 / 追加行数 / 削除行数

**変更ファイル一覧**
- ファイルごとに変更種別（追加/変更/削除）と変更行数を表示
- 論理的なグループに分類（例: フロントエンド/バックエンド/テスト/設定）

**変更の要約**
- 何がどう変わったかを箇条書きで簡潔に説明
- 影響範囲の概要

**レビューコメント・会話**（存在する場合）
- 既存のレビューコメントを要約

**気になるポイント**（あれば）
- 潜在的なリスクや確認すべき箇所を列挙
- ただし判断はユーザーに委ねる

### 3. 追加操作

ユーザーが求めた場合のみ:
- 特定ファイルの差分を詳細表示
- 関連する Issue やコミット履歴の表示
