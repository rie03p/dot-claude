---
name: dependency-check
description: |
  プロジェクトの依存パッケージの脆弱性チェックと更新確認を行うスキル。以下のような場面で積極的に使うこと:
  - 「依存関係をチェックして」「脆弱性はない？」「パッケージの更新を確認して」
  - 「npm audit して」「outdated 確認して」「セキュリティ大丈夫？」
  - 「ライブラリのバージョン古くない？」「アップデートできるものある？」
  - 「/dependency-check」のように直接呼び出されたとき
  - 依存パッケージの脆弱性、更新状況、バージョン管理に関する質問が出たとき
  パッケージ・ライブラリ・依存関係のチェックや更新確認を求められたら、このスキルを使うこと。
argument-hint: "[--audit | --outdated | --all]"
allowed-tools: Bash(npm *), Bash(yarn *), Bash(pnpm *), Bash(pip *), Bash(cat*), Bash(jq*), Read, WebFetch
---

# 依存パッケージチェックスキル

プロジェクトの依存パッケージについて、脆弱性と更新状況を確認する。

## 引数の解析

`$ARGUMENTS` からモードを判定する:
- `--audit`: 脆弱性チェックのみ
- `--outdated`: 更新確認のみ
- `--all` または未指定: 両方実行

## 手順

### 1. パッケージマネージャの検出

プロジェクトルートのファイルから自動検出する（手動指定より自動検出の方が確実で、複数マネージャの混在も検出できるため）:

| ファイル | マネージャ |
|---|---|
| `package-lock.json` | npm |
| `yarn.lock` | yarn |
| `pnpm-lock.yaml` | pnpm |
| `requirements.txt` / `pyproject.toml` | pip |
| `Gemfile.lock` | bundler |
| `go.sum` | go modules |

### 2. 脆弱性チェック

検出したマネージャに応じてコマンドを実行する:

```bash
# npm
npm audit --json

# yarn
yarn audit --json

# pip
pip audit --format json
```

結果を以下の形式で整理する:

```
## 脆弱性レポート

### Critical (即座に対応が必要)
- [パッケージ名@バージョン] — 脆弱性の説明、修正バージョン

### High
- ...

### Moderate / Low
- ...

### 推奨アクション
1. `npm audit fix` で自動修正可能: X件
2. 手動対応が必要: Y件（Breaking changes あり）
```

### 3. 更新確認

```bash
# npm
npm outdated --json

# yarn
yarn outdated --json
```

結果を整理する:

```
## 更新可能なパッケージ

### メジャーアップデート（Breaking changes の可能性）
| パッケージ | 現在 | 最新 | 変更内容 |
|---|---|---|---|

### マイナー/パッチアップデート（安全に更新可能）
| パッケージ | 現在 | 最新 |
|---|---|---|
```

### 4. サマリー

- 脆弱性の総数と深刻度別の内訳
- 更新可能なパッケージ数
- 推奨する対応アクション（優先度順）

## 使用例

**例1**: ユーザーが「このプロジェクトの依存パッケージに脆弱性がないか調べて」と言った場合
→ lockfile からパッケージマネージャを検出し、`npm audit --json` 等を実行して結果を整理する。

**例2**: ユーザーが「更新できるパッケージある？」と言った場合
→ `npm outdated --json` 等を実行し、メジャー/マイナー/パッチに分類して表示する。

**例3**: ユーザーが「/dependency-check --audit」と呼び出した場合
→ 脆弱性チェックのみを実行し、深刻度別にレポートする。
