---
title: "第8回 Python AIエージェントは「実装」ではなく「仕様」から設計する"
emoji: "🐍"
type: "tech"
topics:
  - AIエージェント
  - Python
  - SystemSpecification
  - SDD
  - AI_Native
published: true
published_at: "2026-08-28 07:00"
---

# Python AIエージェントはコードから作り始めてはいけない

本記事は「AIエージェント設計実践シリーズ」の第8回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

前回は営業支援AIを題材に、人とAIが協調するSystem Specificationをレビューしました。

今回はPython AIエージェントを例に、**実装より前に仕様を定義する重要性**を解説します。

---

# よくある実装の始め方

多くの開発では、

```text
Pythonプロジェクト作成
   ↓
LLM呼び出し
   ↓
動けばOK
```

という流れになりがちです。

しかし、これでは仕様変更に弱く、保守も難しくなります。

---

# As-Is

```text
要求
   ↓
すぐにPython実装
   ↓
動作確認
```

課題

- 入出力が曖昧
- 判断ルールがコードへ埋め込まれる
- テストしにくい

---

# To-Be

```text
Business Specification
        ↓
System Specification
        ↓
Python設計
        ↓
実装
```

---

# レビュー前のSystem Specification

## Input

```yaml
user_request
```

## Output

```yaml
response
```

この仕様ではPythonコードの責務が曖昧になります。

---

# レビュー① 入出力を明確にする

Input

```yaml
user_request
context
history
```

Output

```yaml
answer
reason
confidence
next_action
```

---

# レビュー② コンポーネントを分離する

System Specificationの段階で、

- LLM
- Prompt
- Tool
- Memory
- Validation

の責務を整理します。

---

# レビュー③ エラー処理を定義する

```text
LLMエラー
   ↓
リトライ
   ↓
失敗
   ↓
Human in the Loop
```

実装前に例外処理を仕様へ組み込みます。

---

# 改善後のSystem Specification

## Input

```yaml
user_request
context
history
```

## Output

```yaml
answer
reason
confidence
next_action
```

## Constraints

```yaml
confidence < 80% → 人へ引継ぎ
Tool失敗 → リトライ
最終失敗 → エラー通知
```

---

# チェックポイント

- 入出力は十分か
- モジュール責務は明確か
- エラー処理を定義しているか
- Human in the Loopを考慮しているか

---

# まとめ

Python AIエージェントでは、コードを書くことよりも先に、System Specificationを整理することが重要です。

仕様が明確であれば、Pythonだけでなく他の実装基盤へも展開できます。

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

**「マルチエージェントのSystem Specificationレビュー」**

を題材に、複数のAIエージェントが連携するための仕様設計を解説します。
