---
title: "【第7回】SpecificationからDify Workflowを設計する"
emoji: "🔄"
type: "tech"
topics:
  - AI
  - Dify
  - SDD
  - 設計
  - 生成AI
published: true
published_at: "2026-07-21 08:00"
---

# SpecificationからDify Workflowを設計する

前回の記事では、Specificationについて説明しました。

重要だったのは、

- 入力
- 出力
- 判断ルール
- 制約

を定義することでした。

そして、SpecificationはDify専用でもPython専用でもないという話もしました。

今回は、そのSpecificationをDify Workflowへ変換する考え方について説明します。

---

# Dify Workflowは仕様ではない

最初に誤解を解いておきます。

Dify Workflowは仕様ではありません。

Dify Workflowは仕様を実現するための実装形態です。

ここを混同すると、後からワークフローの修正が大変になります。

---

# よくある失敗

例えば、問い合わせ分類AIを作るとします。

多くの場合、最初にDifyを開きます。

そして、

- LLMノード
- IFノード
- テンプレート

を追加し始めます。

すると、気が付けばワークフローは完成します。

しかし数週間後、次のような問題が発生します。

---

# なぜそのノードがあるのか分からない

ワークフローを見ると、

- なぜこの分岐なのか
- なぜこのプロンプトなのか
- なぜこの判定基準なのか

が分からなくなります。

実装は残っています。

しかし仕様は消えています。

---

# SDDでは順番が逆になる

SDDでは最初にWorkflowを作りません。

先にSpecificationを作ります。

業務の現状と理想

（As-Is / To-Be）

↓

要求

↓

Specification

↓

Dify Workflow

です。

---

# 問い合わせ分類AIで考える

例えば前回のSpecificationが次のような内容だったとします。

入力：問い合わせ本文

出力：category, priority, summary, reply_draft

分類：問い合わせ、依頼、クレーム、その他

優先度：高、中、低

制約：summaryは30文字以内、reply_draftは100文字以内

ここにはDifyという言葉は一度も出てきません。

---

# Workflowへ変換する

しかし、このSpecificationを見ると必要な処理が見えてきます。

問い合わせ本文を受け取る

↓

AIへ判定依頼する

↓

分類結果を返す

↓

要約を返す

↓

返信案を返す

これだけです。

---

# ノードは仕様から決まる

Difyでは、つい「どのノードを使うか」から考えがちです。

しかしSDDでは逆です。

まず仕様を見る。

その後、仕様を実現するために必要なノードを決めます。

Workflowは仕様の結果です。

---

# Specificationはそのまま実装指示書になる

ここが重要です。

Difyで実装する場合、

LLMノードのシステムプロンプトに、

Specificationをそのままルールとして渡せます。

新しいプロンプトを考えるのではありません。

仕様書を実装へ変換しているだけです。

これがSDDの考え方です。

---

# Workflow設計の4ステップ

## Step1

入力を確認する

## Step2

出力を確認する

## Step3

判断ルールを確認する

## Step4

必要なノードへ分解する

---

# Difyの価値は変わらない

私はDifyを否定しているわけではありません。

むしろDifyは非常に優秀です。

ただし、仕様の代わりにはなりません。

Workflowは作れても、業務ルールまでは管理できません。

だからSpecificationが必要になります。

---

# 仕様があると変更に強くなる

例えば、分類を4種類から6種類へ変更したい。

優先度を3段階から5段階へ変更したい。

そんな時、最初に見るべきはWorkflowではありません。

Specificationです。

仕様を修正し、その結果としてWorkflowを修正します。

これがSDDです。

---

# Difyは設計図ではなく建物である

Specification = 設計図

Dify Workflow = 建物

建物だけ残っていても、なぜそう作ったのかは分かりません。

だから設計図が必要なのです。

---

# Difyだけの話ではない

今回はSpecificationをDify Workflowへ変換しました。

しかし、同じSpecificationから、Python Agentを設計することもできます。

出口が変わっても、正本であるSpecificationは変わりません。

これがSDDの最大の強みです。

---

# 次回予告

ここまでで、SpecificationからDify Workflowへ変換する考え方を説明しました。

では、Difyで作ることと、仕様駆動開発（SDD）は何が違うのでしょうか。

次回は、「Difyと仕様駆動開発は何が違うのか」をテーマに、両者の役割の違いを整理してみます。

---

## 📚 SDD連載

▶ **SDDシリーズ一覧（ハブ記事）**
https://zenn.dev/tigerone1945/articles/sdd-hub-ai-sdd-series

◀ 第6回
Specificationを書く
https://zenn.dev/tigerone1945/articles/sdd-06-writing-a-specification

▶ 第8回


---

## 📖 関連書籍

### Kindle
『AIエージェント・ストラテジスト：組織を動かす変革の戦略』
https://www.amazon.co.jp/dp/B0H3GGSZH1

---

## 🎓 実践講座（Udemy）

・講座1（GAS×Dify）
https://www.udemy.com/course/gas-dify-ai-agent-1/

---
