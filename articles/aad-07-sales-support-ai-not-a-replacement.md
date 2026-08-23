---
title: "第7回 営業支援AIは営業担当者の代わりではない"
emoji: "💼"
type: "tech"
topics:
  - AIエージェント
  - SystemSpecification
  - SDD
  - AI設計
  - AI_Native
published: true
published_at: "2026-08-25 07:00"
---

# 営業支援AIは「提案する」だけでは価値にならない

本記事は「AIエージェント設計実践シリーズ」の第7回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

前回は問い合わせトリアージAIを例に、優先度判定とHuman in the Loopの重要性を解説しました。

今回は営業支援AIを題材に、営業活動を支援するために必要なSystem Specificationをレビューします。

---

# As-Is

```text
問い合わせ受信
   ↓
営業担当者が内容確認
   ↓
過去の商談履歴を確認
   ↓
提案内容を作成
   ↓
顧客へ返信
```

## 課題

- 提案内容が担当者の経験に依存する
- 過去の商談履歴を探すのに時間がかかる
- 対応品質にばらつきがある

---

# To-Be

```text
問い合わせ受信
   ↓
AIが内容を分析
   ↓
過去の商談・FAQを検索
   ↓
提案案を作成
   ↓
営業担当者がレビュー
   ↓
顧客へ返信
```

AIは営業担当者を置き換えるのではなく、提案作成を支援します。

---

# レビュー前のSystem Specification

## Input

```yaml
customer_message
```

## Output

```yaml
proposal
```

この仕様では実運用には不足しています。

---

# レビュー① Input不足

営業提案には問い合わせ内容だけでなく、

- customer_profile
- opportunity
- sales_history

などの情報も必要です。

---

# レビュー② Output不足

提案文だけでは営業活動は進みません。

必要な出力例

- proposal
- recommended_products
- next_action
- confidence

---

# レビュー③ 判断ルール不足

例えば、

- 新規顧客 → 製品紹介を優先
- 既存顧客 → 過去提案を参照
- 高額案件 → 上長レビュー

などのルールを仕様へ定義します。

---

# レビュー④ Human in the Loop

営業提案はAIだけで確定しません。

```text
AIが提案作成
   ↓
営業担当者レビュー
   ↓
必要に応じて修正
   ↓
顧客へ送信
```

営業担当者の最終判断を仕様へ組み込みます。

---

# 改善後のSystem Specification

## Input

```yaml
customer_message
customer_profile
sales_history
opportunity
```

## Output

```yaml
proposal
recommended_products
next_action
confidence
```

## Constraints

```yaml
高額案件は上長レビュー
confidence < 80% は要確認
契約条件はAIのみで確定しない
```

---

# チェックポイント

- 顧客情報は十分か
- 過去の商談履歴を利用するか
- AIが提案しない条件を定義しているか
- 営業担当者のレビュー工程があるか

---

# まとめ

営業支援AIの価値は、提案文を自動生成することではありません。

**営業担当者がより良い判断を行える情報を提供し、人とAIが協調して営業活動を進められる仕様を設計すること**です。

System Specificationでレビュー観点を明確にしておくことで、実装技術が変わっても品質を維持できます。

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

**「Python AIエージェントのSystem Specificationレビュー」**

をテーマに、設計から実装へ橋渡しするための考え方を解説します。
