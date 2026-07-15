---
title: "【第6回】Specificationを書く"
emoji: "📝"
type: "tech"
topics:
  - AI
  - SDD
  - Kiro
  - 仕様駆動開発
  - 生成AI
published: true
published_at: "2026-07-17 08:00"
---

# Specificationを書く

前回の記事では、Kiroを使って要求と仕様を整理する流れを紹介しました。

requirements.md

spec.md

の2つのファイルを作り、

実装の前に考えるべきことを整理しました。

今回は、その中でも開発の正本（Source of Truth）となる

spec.md

つまり

Specification

について深掘りしていきます。

---

# 要求と仕様は違う

まず最初に理解しておきたいことがあります。

要求と仕様は同じではありません。

例えば、問い合わせ対応業務を考えてみます。

要求は次のようなものです。

- 問い合わせ対応を効率化したい
- 優先度判断を標準化したい
- 返信作成時間を短縮したい

これは業務側の願いです。

しかし、これだけではAIは実装できません。

---

# AIが欲しいのは仕様である

AIが欲しいのは仕様です。

例えば、

- 分類は4種類
- 優先度は3段階
- 要約は30文字以内
- 返信案は100文字以内

というような具体的なルールです。

AIは要求から実装を推測できます。

しかし推測にはブレがあります。

だから仕様が必要になります。

---

# Specificationとは何か

私が考えるSpecificationとは、

「実装者へ渡すための業務ルール集」

です。

実装者が人間でもAIでも変わりません。

重要なのは、

- 何を作るのか
- どのように判断するのか
- どのような制約があるのか

を明文化することです。

---

# 仕様の例

問い合わせ分類AIを例にしてみます。

```md
# 入力

問い合わせ本文

# 出力

category
priority
summary
reply_draft

# 分類

問い合わせ
依頼
クレーム
その他

# 優先度

高
中
低

# 制約

summaryは30文字以内

reply_draftは100文字以内
```

これだけでもAIの理解度は大きく変わります。

---

# 仕様に書くべきこと

Specificationに書くべき内容は大きく4つです。

## ① 入力

何を受け取るのか。

## ② 出力

何を返すのか。

## ③ 判断ルール

どのように分類するのか。

## ④ 制約

文字数や形式など。

例えば問い合わせ分類AIなら、

### 入力

問い合わせ本文

### 出力

- category
- priority
- summary
- reply_draft

### 判断ルール

- 4分類で判定する
- 優先度は3段階で判定する

### 制約

- 要約は30文字以内
- 返信案は100文字以内

となります。

---

# 仕様に書かなくて良いこと

逆に、実装方式は書かなくても構いません。

例えば、

- Pythonで実装する
- Difyで実装する
- AWSで動かす

これは後から変えられます。

Specificationは実装に依存しないことが重要です。

---

# なぜ実装から切り離すのか

多くの開発では、

仕様と実装が混ざっています。

すると実装を変更した瞬間に仕様も崩れます。

しかし本来、

業務ルール

と

実装方式

は別物です。

だからSDDではSpecificationを先に作ります。

---

# DifyでもPythonでも同じ

ここが今回の重要ポイントです。

SpecificationはDify専用でもPython専用でもありません。

同じSpecificationから、

- Dify Workflow
- Python Agent

の両方を設計できます。

つまりSpecificationが正本になります。

---

# AI時代の設計者の仕事

AIがコードを書く時代になるほど、

人間の仕事はコードから離れていきます。

代わりに重要になるのが、

- 要求を整理する
- 判断基準を決める
- 制約を定義する
- Specificationを書く

ことです。

私はこれがAI時代の設計者の仕事だと考えています。

---

# 次回予告

ここまででSpecificationの役割を説明しました。

Specificationの

- 入力
- 出力
- 判断ルール
- 制約

が整理されていると、

Difyのノード設計は驚くほど簡単になります。

では、このSpecificationをDifyへ落とし込むとどうなるのでしょうか。

次回は

Specification → Dify Workflow

の変換をテーマに、

仕様からワークフローを設計する考え方を解説します。

---

## 📚 SDD連載

▶ **SDDシリーズ一覧（ハブ記事）**
https://zenn.dev/tigerone1945/articles/sdd-hub-ai-sdd-series

◀ 第5回
【第5回】Kiroを使って仕様を書いてみる
https://zenn.dev/tigerone1945/articles/sdd-05-writing-a-spec-with-kiro

▶ 第7回
【第7回】SpecificationからDify Workflowを設計する

---

## 📖 関連書籍

### Kindle
『AIエージェント・ストラテジスト：組織を動かす変革の戦略』
https://www.amazon.co.jp/dp/B0H3GGSZH1

---

## 🎓 実践講座（Udemy）

・講座1（GAS×Dify）
https://www.udemy.com/course/gas-dify-ai-agent-1/
