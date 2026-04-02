---
name: get-linear-issue
description: |
  LinearのIssueを取得し、親Issue・子Issue・関連Issueを含めて要点を整理して表示する。
  このスキルはユーザーが「チケットの内容を見せて」「PROJ-XXXXの詳細を教えて」「このチケットどうなってる？」「PROJ-XXXXって何？」「チケットの状況を確認して」「/get-linear-issue PROJ-XXXX」など、Linearチケットの情報確認を求めたときに積極的に使うこと。
  会話中にチケットID（PROJ-1234のような形式）が出てきたら、ユーザーがチケット情報を必要としている可能性が高いので、このスキルの使用を検討すること。Linearのチケット閲覧・参照・確認に関する依頼はすべてこのスキルで対応する。
argument-hint: "<チケットID (例: PROJ-1234)>"
---

# Linear Issue取得スキル

指定された Linear Issue の詳細を取得し、親Issue・子Issue・関連Issue（relation）まで含めて要点を整理して返す。

## 使用例

**例1**: 「GLU-456の詳細を教えて」
→ GLU-456 のチケット情報を取得し、親Issue・子Issue・関連Issueも含めて整理して表示する。

**例2**: 「このチケットどうなってる？ GLU-789」
→ GLU-789 のステータス、担当者、最新コメントなどを取得して報告する。

**例3**: 会話中に「GLU-123の実装を始めたい」と言われた場合
→ まずこのスキルで GLU-123 の情報を取得してから、次のアクションを提案する。

## 引数の解析

`$ARGUMENTS` からチケットID（例: `PROJ-1234`）を読み取る。未指定ならユーザーに聞く。

## 手順

### 1. Issue情報の取得

`linear api` で対象Issueを取得する。`searchIssues(term: ..., first: 1)` を使う。

```bash
linear api '
query {
  searchIssues(term: "<チケットID>", first: 1) {
    nodes {
      id
      identifier
      title
      description
      url
      priority
      estimate
      state { name type }
      assignee { name email }
      creator { name }
      project { id name }
      team { id key name }
      labels { nodes { name } }
      parent {
        id
        identifier
        title
        description
        state { name type }
      }
      children {
        nodes {
          id
          identifier
          title
          description
          state { name type }
          assignee { name }
        }
      }
      relations {
        nodes {
          id
          type
          issue {
            id
            identifier
            title
            description
            state { name type }
          }
          relatedIssue {
            id
            identifier
            title
            description
            state { name type }
          }
        }
      }
      comments(first: 5, orderBy: createdAt) {
        nodes {
          id
          body
          createdAt
          user { name }
        }
      }
    }
  }
}
'
```

### 2. 取得結果の検証

- `nodes` が空なら「Issueが見つからない」旨をユーザーに伝え、IDの再確認を依頼する
- 複数件返る場合は identifier 完全一致を優先し、先頭件を採用する
- `relations` が空でも正常。関連Issueなしとして扱う

### 3. 要点整理（ユーザーへの返却フォーマット）

取得後は以下の順で簡潔に整理して返す。

1. 対象Issue: identifier / title / state / assignee / priority / URL
2. 概要: description の要点（3〜5行）
3. 親Issue: identifier / title / state / 要点
4. 子Issue一覧: identifier / title / state / assignee
5. 関連Issue一覧: relation type / identifier / title / state
6. 直近コメント（最大3件）: 投稿者 / 日時 / 要点
7. 依存関係・懸念点（あれば）

### 4. 必要に応じた追加取得

ユーザーが必要とした場合のみ追加で実行する。

- 親Issueをさらに深掘りする（親の親、project情報）
- 関連Issueごとの詳細を個別取得する
- コメントの取得件数を増やす

## 注意

- Linear の情報取得は `linear api` を使う（MCP経由だと認証やレスポンス形式が異なり、スキルの手順と合わなくなるため）
- Issue更新はこのスキルの責務外。更新が必要なら別途ユーザー確認後に実行する
- description が長い場合は要点を抜粋する（全文をそのまま貼ると可読性が下がるため）
