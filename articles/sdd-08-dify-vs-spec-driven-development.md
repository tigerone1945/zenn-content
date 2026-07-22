---
title: "【第8回】Difyと仕様駆動開発は何が違うのか"
emoji: "⚖️"
type: "tech"
topics:
  - AI
  - Dify
  - SDD
  - 設計
  - 生成AI
published: true
published_at: "2026-07-24 08:00"
---

# Difyと仕様駆動開発は何が違うのか

ここまでの連載で、

- なぜ仕様が必要なのか
- SDDとは何か
- Kiroで仕様を書く方法
- Specificationを書く方法
- SpecificationからDify Workflowへ変換する方法

を説明してきました。

ここで一つの疑問が出てきます。

「結局、DifyとSDDは何が違うのか？」

です。

今回は、DifyとSDDの違いを整理してみます。

---

# 結論から言う

結論から言うと、

DifyとSDDは競合しません。

役割が違います。

私は次のように考えています。

SDD

↓

Dify

です。

逆ではありません。

---

# Difyは何を解決するのか

Difyは優れたAIアプリ開発基盤です。

ノーコードで

- Chatbot
- Workflow
- Agent

を作れます。

さらに、

- RAG
- API連携
- ツール呼び出し

も実現できます。

つまり、

Difyは

「どう作るか」

を支援するツールです。

---

# SDDは何を解決するのか

一方、

SDDが解決するのは別の問題です。

それは、

「何を作るか」

です。

具体的には、

- 何を入力するのか
- 何を出力するのか
- どのように判断するのか
- どのような制約があるのか

を整理します。

---

# なぜ混同されるのか

多くの人は、

Difyを開いた瞬間に開発を始めます。

すると、

いつの間にか

- プロンプト
- ノード
- 分岐

が増えていきます。

そして、

ワークフローは完成します。

しかし、

「なぜそう作ったのか」

は残りません。

---

# Difyの問題ではない

ここで誤解してはいけません。

これはDifyの問題ではありません。

もしPythonで作っても同じです。

これまでGASで自動化を作った時も、

RPAでシナリオを作った時も、

同じ問題にぶつかったはずです。

問題はツールではありません。

仕様が存在しないことです。

---

# Dify利用者がぶつかる壁

Difyを触り始めた頃は楽しいです。

すぐにAIアプリが作れます。

しかし運用が始まると、

別の問題が出てきます。

例えば、

- 分類ルールを変更したい
- 出力形式を変更したい
- 判定基準を変更したい

という要求です。

その時、

どこを直せばよいのか分からなくなります。

---

# 本当に修正したいもの

実は、

修正したいのは

Workflowではありません。

業務ルールです。

例えば、

「優先度は3段階」

というルール。

「要約は30文字以内」

というルール。

「クレームを最優先にする」

というルール。

本当に変更したいのは、

これらです。

---

# SDDは業務ルールを管理する

SDDでは、

業務ルールを

Specification

として管理します。

つまり、

Workflowより先に、

仕様が存在します。

だから変更が起きた時も、

最初に修正するのは

Specificationです。

---

# DifyとSDDの関係

私は次のように整理しています。

SDD

↓

Specification

↓

Dify Workflow

Difyは仕様を実現するための手段です。

仕様そのものではありません。

---

# 家づくりで考える

家づくりで考えると分かりやすいです。

Specification

＝ 設計図

Dify Workflow

＝ 建物

です。

建物だけ見ても、

なぜそう設計したかは分かりません。

しかし設計図があれば、

修正も増築もできます。

---

# Difyを使うほどSDDが必要になる

面白いことに、

Difyを本格運用するほど

SDDが必要になります。

小規模なPoCなら、

仕様なしでも動くかもしれません。

しかし、

- 利用者が増える
- ワークフローが増える
- Agentが増える

と、

必ず仕様管理が必要になります。

---

# DifyかSDDかではない

ここも重要です。

DifyかSDDか

ではありません。

正しくは、

SDDしてからDify

です。

私はこれがAI開発の自然な流れだと考えています。

---

# AI時代の開発プロセス

従来は、

要求

↓

設計

↓

実装

でした。

AI時代は、

要求

↓

Specification

↓

AI実装

になります。

Difyも、

Claude Codeも、

将来登場する新しいツールも、

この構造は変わりません。

---

# Difyだけの話ではない

設計図（Specification）さえしっかりしていれば、

大工道具はDifyでなくても構いません。

同じSpecificationから、

Dify Workflowを設計することもできます。

Python Agentを設計することもできます。

出口が変わっても、

正本であるSpecificationは変わりません。

これがSDDの最大の強みです。

---

# 次回予告

ここまでで、

Difyと仕様駆動開発の違いを整理しました。

では、

同じSpecificationから

Python Agentを設計するとどうなるのでしょうか。

次回は、

「SpecificationからPython Agentを設計する」

をテーマに、

Difyとは異なる実装形態への変換を考えてみます。

---

## 📚 SDD連載

▶ **SDDシリーズ一覧（ハブ記事）**
https://zenn.dev/tigerone1945/articles/sdd-hub-ai-sdd-series

◀ 第7回
【第7回】SpecificationからDify Workflowを設計する
https://zenn.dev/tigerone1945/articles/sdd-07-from-spec-to-dify-workflow

▶ 第9回
【第9回】SpecificationからPython Agentを設計する


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
