---
title: "第9回 マルチエージェントは「役割設計」で決まる"
emoji: "🤝"
type: "tech"
topics:
  - AIエージェント
  - マルチエージェント
  - SystemSpecification
  - SDD
  - AI_Native
published: true
published_at: "2026-09-01 07:00"
---

# マルチエージェントは「AIを増やす」ことではない

本記事は「AIエージェント設計実践シリーズ」の第9回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

前回はPython AIエージェントを例に、実装より前にSystem Specificationを整理する重要性を解説しました。

今回は、複数のAIエージェントが協調して動作する**マルチエージェント**を題材に、役割分担と連携方法をどのように仕様化するかをレビューします。

---

# As-Is

```text
問い合わせ受信
      ↓
1つのAIが
すべて処理
```

## 課題

- 責務が集中する
- プロンプトが肥大化する
- 保守・改善が難しい

---

# To-Be

```text
問い合わせ受信
      ↓
分類AI
      ↓
      ├─ FAQ AI
      ├─ 営業支援AI
      └─ 人へ引継ぎ
```

1つのAIに全てを任せるのではなく、役割ごとに分割します。

---

# レビュー前のSystem Specification

## Agent

```yaml
agent:
  role: "すべて処理する"
```

この仕様では責務が曖昧です。

---

# レビュー① 役割を明確にする

例

- Classifier Agent
- FAQ Agent
- Sales Agent
- Human Review

各エージェントの責務を定義します。

---

# レビュー② Agent間の入出力

```text
分類AI
    ↓
category
confidence
    ↓
FAQ AI
```

受け渡すデータを仕様として定義します。

---

# レビュー③ 引継ぎ条件

例えば

- confidence < 80%
- category = unknown
- Tool Error

などは、人または別エージェントへ引き継ぎます。

---

# 改善後のSystem Specification

## Agents

```yaml
ClassifierAgent
FAQAgent
SalesAgent
HumanReview
```

## Shared Output

```yaml
category
confidence
next_agent
reason
```

## Constraints

```yaml
confidence < 80% → HumanReview
unknown → HumanReview
Tool Error → Retry
```

---

# チェックポイント

- 各エージェントの責務は明確か
- 入出力は統一されているか
- Agent間の受け渡しデータを定義しているか
- Human in the Loopを考慮しているか

---

# まとめ

マルチエージェントの品質は、AIの数では決まりません。

**各エージェントの役割と連携をSystem Specificationとして定義すること**が重要です。

役割が明確であれば、個々のAIを改善しても全体の品質を維持できます。

---

## もっと体系的に学びたい方へ

Business Specification・System Specificationを体系的に学びたい方は、

📚 [AI時代の仕様駆動開発：Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書](https://www.amazon.co.jp/dp/B0HBQ32KCQ)

をご覧ください。

---

## シリーズ全体はこちら

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

---

## 次回予告

次回は

**「Code AgentのSystem Specificationレビュー」**

を題材に、AIがコードを生成・修正するために必要な仕様設計を解説します。
