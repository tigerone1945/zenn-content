---
title: "第11回 AIエージェントをProduction品質へ育てる"
emoji: "🛡️"
type: "tech"
topics:
  - AIエージェント
  - SystemSpecification
  - Guardrails
  - AI設計
  - AI_Native
published: true
published_at: "2026-09-08 07:00"
---

# PoCが動くだけでは業務では使えない

本記事は「AIエージェント設計実践シリーズ」の第11回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

前回はCode Agentを題材に、実装しやすいSystem Specificationについて解説しました。

今回は、AIエージェントを本番運用するために必要な**Production品質**をSystem Specificationへどのように反映するかをレビューします。

---

# PoCとProductionの違い

PoCでは

```text
入力
 ↓
LLM
 ↓
回答
```

でも動作します。

しかし本番運用では、それだけでは不十分です。

---

# As-Is

```text
入力
 ↓
LLM
 ↓
回答
```

## 課題

- 誤回答をそのまま返す
- 個人情報が漏れる可能性がある
- 障害時の対応が定義されていない
- 品質を継続的に改善できない

---

# To-Be

```text
入力
 ↓
Guardrails
 ↓
LLM
 ↓
Verification
 ↓
Monitoring
 ↓
回答
```

---

# レビュー① Guardrails

AIが実行してはいけないことを定義します。

例

- 個人情報を出力しない
- 禁止業務は実行しない
- 危険な指示は拒否する

---

# レビュー② Verification

AIの出力をそのまま信用しません。

確認項目

- 回答形式
- 必須項目
- 信頼度
- 業務ルール違反

必要に応じてHuman in the Loopへ引き継ぎます。

---

# レビュー③ Monitoring

本番運用では継続的な改善が必要です。

監視例

- エラー率
- Human Review率
- 応答時間
- 成功率

これらを記録し、System Specificationの改善へつなげます。

---

# 改善後のSystem Specification

## Guardrails

```yaml
PIIを出力しない
禁止業務を実行しない
```

## Verification

```yaml
confidence < 80% → HumanReview
回答形式を検証
```

## Monitoring

```yaml
response_time
error_rate
human_review_rate
```

---

# チェックポイント

- Guardrailsを定義しているか
- Verificationを定義しているか
- Monitoring項目を決めているか
- 継続改善の仕組みがあるか

---

# まとめ

Production品質は、実装後に追加するものではありません。

**Guardrails・Verification・Monitoringまで含めてSystem Specificationとして設計すること**で、AIエージェントは業務で安心して利用できるようになります。

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

最終回は

**「AI Native Enterpriseへのロードマップ」**

として、本シリーズ全体を振り返り、設計からWorkflow、実装、本番運用へ発展する学習ロードマップをまとめます。
