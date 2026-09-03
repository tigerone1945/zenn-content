---
title: "第10回 Code Agentが正しく実装できるかは「仕様」で決まる"
emoji: "💻"
type: "tech"
topics:
  - AIエージェント
  - CodeAgent
  - SystemSpecification
  - SDD
  - AI_Native
published: true
published_at: "2026-09-04 07:00"
---

# Code Agentは仕様が曖昧だと期待どおりに動かない

本記事は「AIエージェント設計実践シリーズ」の第10回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

前回はマルチエージェントを題材に、役割分担と連携方法をSystem Specificationとして定義する重要性を解説しました。

今回はCode Agentを題材に、AIが実装しやすいSystem Specificationとは何かをレビューします。

---

# Code Agentでよくある失敗

```text
「問い合わせ対応AIを作って」
        ↓
Code Agentが実装
        ↓
期待と違うアプリが完成
```

原因の多くは、Code Agentではなく仕様の曖昧さです。

---

# As-Is

```text
要求
  ↓
Code Agentへ指示
  ↓
実装
```

## 課題

- 入出力が曖昧
- 判断ルールが不足
- 例外処理が未定義
- 実装結果が毎回変わる

---

# To-Be

```text
Business Specification
        ↓
System Specification
        ↓
Code Agent
        ↓
レビュー
```

Code Agentは設計書を基に実装する役割です。

---

# レビュー前のSystem Specification

## Input

```yaml
customer_message
```

## Output

```yaml
reply
```

これではCode Agentが実装方法を推測することになります。

---

# レビュー① データ構造を定義する

## Input

```yaml
customer_message
customer_profile
history
```

## Output

```yaml
reply
category
confidence
next_action
```

---

# レビュー② 処理フローを定義する

```text
入力
 ↓
分類
 ↓
ルール判定
 ↓
回答生成
 ↓
レビュー
```

処理順序を明確にします。

---

# レビュー③ 制約条件を定義する

```yaml
confidence < 80% → HumanReview
個人情報はログへ出力しない
エラー時は再試行
```

Code Agentが守るべきルールを仕様へ記述します。

---

# 改善後のSystem Specification

## Input

```yaml
customer_message
customer_profile
history
```

## Output

```yaml
reply
category
confidence
next_action
```

## Constraints

```yaml
PIIをログへ保存しない
信頼度が低い場合は人へ引継ぎ
エラー時はリトライ後に通知
```

---

# チェックポイント

- 入出力は十分か
- データ構造は明確か
- 処理フローは定義されているか
- 制約条件を記述しているか
- Human in the Loopを考慮しているか

---

# まとめ

Code Agentの品質は、プロンプトだけでは決まりません。

**Code Agentが迷わず実装できるSystem Specificationを用意すること**が重要です。

仕様が明確であれば、実装品質のばらつきを抑え、レビューもしやすくなります。

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

**「Production品質へ育てるSystem Specification」**

をテーマに、Guardrails・Verification・Monitoringを考慮した設計を解説します。
