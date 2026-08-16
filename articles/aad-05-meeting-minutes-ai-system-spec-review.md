---
title: "第5回 議事録AIのSystem Specificationレビュー ─ 要約AIではなく、業務を前に進めるAIを設計する"
emoji: "📝"
type: "tech"
topics:
  - AIエージェント
  - SystemSpecification
  - SDD
  - AI設計
  - AI_Native
published: true
published_at: "2026-08-18 07:00"
---

# 議事録AIは「要約できる」だけでは不十分

本記事は「AIエージェント設計実践シリーズ」の第5回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

前回はFAQ AIを例に、RAGの性能よりもSystem Specificationの品質が重要であることを解説しました。

今回は議事録AIを題材に、「要約するAI」ではなく「業務を前へ進めるAI」を設計するためのレビューを行います。

## As-Is

```
会議終了
 ↓
担当者が議事録作成
 ↓
タスク整理
 ↓
共有
```

課題

- 要約品質が担当者でばらつく
- タスク漏れが起きる
- 担当者・期限が曖昧になる

## To-Be

```
会議終了
 ↓
AIが要約
 ↓
AIがタスク抽出
 ↓
担当者・期限を整理
 ↓
レビュー後に共有
```

## レビュー前

Input

```yaml
transcript
```

Output

```yaml
summary
```

これでは「要約AI」は作れても「議事録AI」としては不十分です。

## レビュー

### Output不足

必要な出力

- summary
- action_items
- owner
- due_date
- decisions

### 判断ルール不足

- 「○○さんお願いします」→担当者
- 「来週まで」→期限
- 「決定します」→決定事項

### Human in the Loop

```
AI生成
 ↓
人がレビュー
 ↓
共有
```

## 改善後

Input

```yaml
transcript
participants
meeting_date
```

Output

```yaml
summary
action_items
owner
due_date
decisions
```

Constraints

```yaml
不明な担当者は未割当
期限が曖昧なら要確認
AIのみで確定しない
```

# まとめ

議事録AIで重要なのは要約精度だけではありません。

業務で利用できる情報を再利用可能な形で出力できることです。

---

## もっと体系的に学びたい方へ

Business Specification・System Specificationを体系的に学びたい方は

「AI時代の仕様駆動開発（SDD-Kindle）」

をご覧ください。

---

## シリーズ全体はこちら

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

---

## 次回予告

第6回では

「問い合わせトリアージAIのSystem Specificationレビュー」

を解説します。
