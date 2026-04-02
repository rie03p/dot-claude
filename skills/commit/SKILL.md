---
name: commit
description: gitでステージングされているファイルから、コミット文章を作成して承認されしだいコミットを行います。「コミットして」「/commit」「変更をまとめて」など、gitコミットを行いたい場面で使う。ステージ済みファイルがなければ何をステージするか提案もする。
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*), AskUserQuestion
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Your task

ステージされたファイルから変更内容を分析し、3つのコミットメッセージ候補を生成してユーザーに選択させます。

1. **変更の分析**: diffとgit statusから変更の種類・影響範囲・目的を把握
   - ステージ済みの変更がない場合は、unstaged changes や untracked files を確認し、何をステージするかユーザーに提案する。変更自体がなければ「コミットする変更がありません」と伝えて終了する
2. **コミット内容の確認**: 変更内容を確認し、違う作業種類の変更がまじっており分離可能であれば警告を出し、必要に応じて分割を促す（例: バグ修正と機能追加が混在している場合など）
3. **タイトル候補の生成**: 異なる視点で3つのタイトル候補を作成
   - Conventional Commits形式（feat/fix/docs/refactor/chore等）
   - モジュール名(frontend, backend, api等。たとえばパッケージ名やディレクトリ名を利用)を [xxx] の形式でプレフィックスにする
   - 日本語で記述（プレフィックスは英語）
   - 各候補は異なる詳細度や視点を提供（例: 技術的詳細重視、ビジネス価値重視、シンプル表現）

   まとめると、例えば `feat: [frontend] ユーザープロフィール画面のUI改善` や `fix: [api] 認証エラーの修正` のような形式になります。
4. **ユーザー選択**: AskUserQuestionで3候補を提示し選択を受ける
5. **本文を作成**: 選択されたタイトルに基づいて、変更内容を説明する本文を生成（必要に応じて箇条書きで変更点を列挙）。変更内容の概要、影響範囲、新たに出てきたTODOは必ず書く
6. **コミット実行**: 必要に応じてgit addし、選択されたメッセージでコミット

## Constraint

- Claude co-authorshipフッターは不要
- メッセージは日本語（プレフィックスのみ英語）

