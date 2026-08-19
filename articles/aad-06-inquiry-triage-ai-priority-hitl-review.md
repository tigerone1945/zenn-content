---
title: "第6回 問い合わせAIは「優先度」と「人への引き継ぎ」をどう設計するか"
emoji: "🚦"
type: "tech"
topics:
  - AIエージェント
  - SystemSpecification
  - SDD
  - AI設計
  - AI_Native
published: true
published_at: "2026-08-21 07:00"
---

# 今回は問い合わせトリアージAIを題材に、System Specificationをレビューします

本記事は「AIエージェント設計実践シリーズ」の第6回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

前回は議事録AIを例に、要約だけではなくタスクや担当者まで仕様化する重要性を解説しました。

今回は問い合わせトリアージAIを題材に、**優先度判定・エスカレーション・Human in the Loop**をどのようにSystem Specificationへ落とし込むかをレビューします。

---

# As-Is

```text
問い合わせ受信
   ↓
担当者が内容確認
   ↓
緊急度を判断
   ↓
担当部署へ割り振り
```

課題

- 判断基準が担当者によって異なる
- 緊急案件への対応が遅れる
- エスカレーション基準が曖昧

---

# To-Be

```text
問い合わせ受信
   ↓
AIが分類
   ↓
AIが優先度判定
   ↓
高優先度は担当者へ即通知
   ↓
通常案件は担当部署へ振り分け
```

---

# レビュー前のSystem Specification

## Input

```yaml
subject
body
```

## Output

```yaml
category
priority
```

一見十分に見えますが、実運用では仕様不足です。

---

# レビュー① 優先度ルールが曖昧

「高・中・低」だけではAIは判断できません。

例

- 障害・サービス停止 → High
- 請求・契約 → Medium
- 一般問い合わせ → Low

---

# レビュー② エスカレーション条件がない

例えば、

- High
- 信頼度80%未満
- VIP顧客

は担当者へ即時通知するなど、条件を仕様化する必要があります。

---

# レビュー③ Human in the Loop

```text
AI判定
   ↓
High または 判定不能
   ↓
担当者レビュー
   ↓
対応開始
```

AIだけで完結させないことが重要です。

---

# 改善後のSystem Specification

## Input

```yaml
subject
body
sender
customer_type
```

## Output

```yaml
category
priority
department
confidence
escalation
```

## Constraints

```yaml
confidence < 80% → 要確認
High → 即時通知
VIP顧客 → 人が最終確認
```

---

# チェックポイント

- 優先度の基準は明確か
- エスカレーション条件はあるか
- Human in the Loopは定義されているか
- AIが判断しない条件はあるか

---

# まとめ

問い合わせトリアージAIでは、分類精度だけでは十分ではありません。

**「いつAIが判断し、いつ人へ引き継ぐか」まで仕様として定義すること**が、業務で利用できるAIエージェントを実現する鍵になります。

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

**「営業支援AIのSystem Specificationレビュー」**

を題材に、AIエージェントが営業活動を支援するために必要な仕様について解説します。
