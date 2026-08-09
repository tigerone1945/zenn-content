---
title: "第3回 問い合わせ分類AIはなぜ誤判定するのか"
emoji: "🔍"
type: "tech"
topics:
  - AIエージェント
  - SDD
  - SystemSpecification
  - AI設計
  - AI_Native
published: true
published_at: "2026-08-11 07:00"
---

本記事は「AIエージェント設計実践シリーズ」の第3回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

# 問い合わせ分類AIが誤判定する理由

問い合わせ分類AIを作ったものの、

「営業への問い合わせなのに経理へ振り分けられた」

という経験はないでしょうか。

多くの場合、

原因として疑われるのは

- プロンプト
- LLM
- RAG

です。

しかし実際には、

**System Specificationが曖昧だった**

というケースが少なくありません。

今回は、

問い合わせ分類AIを例に、

「どこをレビューすると品質が上がるのか」

を見ていきます。

---

# レビュー前のSystem Specification

```
Input
- subject
- body

Output
- category

Rule
営業なら営業部
請求なら経理
```

一見すると問題なさそうです。

しかし、

この仕様では品質は安定しません。

---

# レビュー① Inputは十分か

送信者情報（sender）がありません。

例えば、

- 社内からの問い合わせ
- 重要顧客からの問い合わせ

では優先度が変わるかもしれません。

**改善**

```
Input

subject
body
sender
```

---

# レビュー② Outputは業務で使えるか

categoryだけでは、

担当部署は分かっても業務は終わりません。

必要なのは

- category
- priority
- summary
- department

です。

---

# レビュー③ Ruleは具体的か

例えば、

```
営業なら営業部
```

だけでは、

「見積」

「価格」

「代理店」

は営業なのでしょうか。

判断できません。

改善後は

```
営業・見積・価格
↓

営業部
```

のように、

判断基準を具体化します。

---

# レビュー④ 例外処理はあるか

分類できない問い合わせは必ず発生します。

例えば、

```
分類不能

↓

要確認
```

というルールを定義します。

さらに

- 個人情報を要約しない
- 信頼度が低ければ人へ引き継ぐ

などの制約も必要です。

---

# 改善後のSystem Specification

```
Input
subject
body
sender

Output
category
priority
summary
department

Rules
営業・見積・価格 → 営業部
請求・支払 → 経理
障害・エラー・不具合 → 情報システム

Constraints
分類不能 → 要確認
個人情報は要約しない
信頼度が低い場合は人へ引き継ぐ
```

このように、

レビューによって

AIが判断するための情報が揃います。

---

# 実装技術は後から選べる

ここまで仕様が整理できれば、

- Dify
- Python
- Amazon Bedrock
- Vertex AI
- Azure AI Foundry

どれでも実装できます。

重要なのは、

実装ではなく

**レビューによって仕様を改善すること**です。

---

# チェックリスト

レビューでは次の点を確認します。

- Inputは十分か
- Outputは業務で使えるか
- Ruleは具体的か
- 例外処理はあるか
- 制約条件はあるか
- 人への引継ぎ条件はあるか
- 成功条件を評価できるか

---

# まとめ

今回紹介した誤判定の原因は、

LLMではありませんでした。

原因は、

**System Specificationが曖昧だったこと**です。

AIエージェントの品質は、

レビューによって大きく改善できます。

---

# もっと体系的に学びたい方へ

本記事では、

問い合わせ分類AIを例に

System Specificationレビューを紹介しました。

より実践で使える設計力を身につけるためには、

- Requirementsの整理方法
- 設計レビューの進め方
- AIが理解できる仕様書の書き方
- AI Native時代の設計思想

まで体系的に理解することが重要です。

これらは Kindle

📚 [AI時代の仕様駆動開発：Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書](https://www.amazon.co.jp/dp/B0HBQ32KCQ)

で詳しく解説しています。

---

# シリーズ全体はこちら

シリーズ全体の位置付けやロードマップは

ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

---

# 次回予告

次回は

**「FAQ AIはなぜ正しく回答できないのか」**

をテーマに、

RAGを前提としたFAQ AIの設計レビューを行います。
