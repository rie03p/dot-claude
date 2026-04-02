---
name: dependency-check
description: プロジェクトの依存パッケージの脆弱性チェックと更新確認を行う。「依存関係をチェックして」「脆弱性はない？」「パッケージの更新を確認して」「/dependency-check」など、依存管理を求められた場面で使う。
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

プロジェクトルートのファイルから自動検出する:

| ファイル | マネージャ |
|---|---|
| `package-lock.json` | npm |
| `yarn.lock` | yarn |
| `pnpm-lock.yaml` | pnpm |
| `requirements.txt` / `pyproject.toml` | pip |
| `Gemfile.lock` | bundler |
| `go.sum` | go modules |

### 2. 脆弱性チェック

検出したマネージャに応じてコマンドを実行:

```bash
# npm
npm audit --json

# yarn
yarn audit --json

# pip
pip audit --format json
```

結果を以下の形式で整理:

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

結果を整理:

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
