---
title: "AIエージェント設計実践シリーズ ── AIを「作る」から「設計する」時代へ"
emoji: "📘"
type: "idea"
topics:
  - AIエージェント
  - SystemSpecification
  - AI設計
  - AI_Native
  - SDD
published: true
published_at: "2026-07-31 08:00"
---

# AIエージェントは「作る」時代から「設計する」時代へ

生成AIの進化によって、AIエージェントを作ること自体は難しくなくなりました。

Difyを使えばノーコードでAIエージェントを作れます。

Claude CodeやCodexを使えば、Pythonコードも短時間で生成できます。

しかし、実際の業務では

- 思ったような判断をしてくれない
- 人によって回答が変わる
- PoCは完成したのに業務で使われない

というケースが少なくありません。

多くの場合、その原因はAIの性能ではありません。

**AIに何を判断させるのか、その仕様が曖昧だからです。**

AIは人間のように空気を読んで判断することはできません。

だからこそ、

- どの情報を入力として使うのか
- 何を判断基準にするのか
- どのような結果を出力するのか
- 判断できない場合はどうするのか

を事前に設計する必要があります。

このシリーズでは、その設計書となる **System Specification（システム仕様）** を中心に、AIエージェントをどのように設計するのかを、実際のユースケースを題材に解説していきます。

---

# なぜ今、System Specificationなのか

AIエージェントに関する情報は急速に増えています。

しかし、多くの記事や動画では、

- プロンプトの書き方
- Difyの使い方
- Pythonの実装方法
- LLMの選び方

といった**実装方法**が中心です。

もちろん実装は重要です。

しかし、その前に考えなければならないことがあります。

> **AIに何を判断させるのか。**

ここが曖昧なままでは、どんなLLMを使っても、どんなフレームワークを使っても、期待したAIエージェントにはなりません。

AIの品質を決めるのは、モデルだけではなく、設計です。

私はこれまで、

- AICXシリーズ
- SDD（Specification Driven Development）シリーズ

を通して、

- As-Is
- To-Be
- Business Specification
- System Specification

という設計プロセスを紹介してきました。

本シリーズでは、その考え方をさらに一歩進め、

**実際のAIエージェントを題材に、System Specificationをレビューしながら設計する方法**を解説します。

---

# このシリーズで学べること

このシリーズの目的は、

**AIエージェントを作ることではありません。**

**AIエージェントを設計できるようになること**です。

毎回ひとつのAIエージェントを取り上げ、

- 業務の目的
- 入力
- 出力
- 判断ルール
- 制約条件
- 例外処理

といった観点からSystem Specificationをレビューしていきます。

実装方法は問いません。

- Dify
- Python
- LangGraph
- Code Agent
- AWS

どの技術を使う場合でも、

設計の考え方は共通です。

このシリーズでは、その「変わらない設計」を学びます。

---

# このシリーズはこんな方におすすめ

- AIエージェントを設計したい方
- Difyを使って業務自動化を進めたい方
- Claude CodeやCodexでAI開発を始めたい方
- PythonでAIエージェントを実装したい方
- DX・業務改善を担当している方
- AI導入プロジェクトに携わるエンジニア
- AIをPoCで終わらせず、実運用につなげたい方

---

# 共通の設計プロセス

本シリーズでは、すべての記事で次の設計プロセスを共通の考え方として扱います。

```text
As-Is
    ↓
To-Be
    ↓
Business Specification
    ↓
System Specification
    ↓
レビュー
    ↓
実装
```

本シリーズで扱うのは、

**System Specificationのレビューまで**です。

実装方法については、Kindle・Udemyで詳しく解説していきます。

---

# シリーズ一覧

## 第1回
### AIエージェントとは何か

AI Native時代に求められるAIエージェントの役割と、設計という考え方を整理します。

---

## 第2回
### Business SpecificationからSystem Specificationへ

As-Is / To-Beから業務仕様を整理し、System Specificationへ落とし込む考え方を学びます。

---

## 第3回
### 問い合わせ分類AIのSystem Specificationレビュー

問い合わせ分類AIを例に、

- 入力
- 出力
- 判断ルール
- 制約条件

をレビューします。

---

## 第4回
### FAQ AIのSystem Specificationレビュー

RAGを活用したFAQ AIを題材に、

検索・回答・例外処理を設計します。

---

## 第5回
### 議事録AIのSystem Specificationレビュー

会議要約・タスク抽出・担当者割り当てをどのように仕様化するかを考えます。

---

## 第6回
### 問い合わせトリアージAIのSystem Specificationレビュー

優先度判定、担当部署への振り分け、エスカレーションなど、ワークフロー型AIを設計します。

---

## 第7回
### 営業支援AIのSystem Specificationレビュー

営業活動を支援するAIエージェントを例に、

CRM連携や商談管理を前提とした設計をレビューします。

---

## 第8回
### Python AIエージェントのSystem Specificationレビュー

Pythonによる実装を前提としたSystem Specificationを考えます。

---

## 第9回
### マルチエージェントのSystem Specificationレビュー

複数のAIエージェントが協調して動くための役割分担と連携方法を設計します。

---

## 第10回
### Code AgentのSystem Specificationレビュー

Code Agentによる開発を前提に、実装しやすいSystem Specificationを考えます。

---

## 第11回
### Production品質へ育てるSystem Specification

Guardrails、Verification、Monitoringなどを考慮し、本番運用を見据えた設計へ改善します。

---

## 第12回
### AI Native Enterpriseへのロードマップ

AI Native時代における企業システムの設計思想と、今後の展望をまとめます。

---

# Kindle・Udemyとの関係

このシリーズは「設計編」です。

各コンテンツは次のような役割を担っています。

| コンテンツ | 役割 |
|-----------|------|
| AICXシリーズ | AI導入・業務変革を考える |
| SDDシリーズ | Specification Driven Developmentの考え方を学ぶ |
| **System Specification実践シリーズ** | AIエージェントのSystem Specificationをレビューする |
| Kindle | System Specificationを体系的に学ぶ |
| Udemy | Dify・Python・AWSで実際に実装する |

つまり、

**設計と実装をつなぐ橋渡し**

が、このシリーズの役割です。

---

# おわりに

AIエージェント開発では、新しいLLMやフレームワークが次々と登場しています。

しかし、どれだけ技術が進化しても、

**AIに何を判断させるのか**

という設計の重要性は変わりません。

AIエージェントの品質は、

プロンプトだけでも、

モデルだけでも決まりません。

**良いSystem Specificationを書けるかどうか。**

それが、AIエージェントの品質を大きく左右します。

このシリーズでは、実際のAIエージェントを題材にしながら、System Specificationをレビューし、実務で役立つ設計力を身につけていきます。

---

# Kindleでもっと体系的に学びたい方へ

このシリーズでは、実際のAIエージェントを題材にしながら、System Specificationをレビューし、**「どう設計するか」**を実践的に解説していきます。

一方で、

- なぜSpecificationが重要なのか
- Business SpecificationとSystem Specificationの考え方
- Specification Driven Development（SDD）の全体像
- AI時代の仕様駆動開発の進め方

といった設計思想を体系的に学びたい方には、**2026年8月3日発売予定のKindle『AI時代の仕様駆動開発』**がおすすめです。

本書では、AICXからSDD、そしてSystem Specificationへとつながる設計アプローチを一冊にまとめています。

「なぜ仕様が重要なのか」を理解したうえで本シリーズを読むことで、各AIエージェントの設計意図をより深く理解できるはずです。

📚 **Kindle予約受付中（2026年8月3日発売予定）**

https://www.amazon.co.jp/dp/B0HBQ32KCQ

---

それでは、第1回から始めましょう。