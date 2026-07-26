---
title: "【第9回】SpecificationからPython Agentを設計する"
emoji: "🐍"
type: "tech"
topics:
  - AI
  - Python
  - SDD
  - ClaudeCode
  - 生成AI
published: true
published_at: "2026-07-28 08:00"
---

# SpecificationからPython Agentを設計する

ここまでの連載で、

- なぜ仕様が必要なのか
- SDDとは何か
- Specificationを書く方法
- SpecificationからDify Workflowを設計する方法
- DifyとSDDの違い

を説明してきました。

そして前回、重要な結論にたどり着きました。

それは、

**Specificationが正本である**

という考え方です。

今回は、そのSpecificationからPython Agentを設計する方法について考えてみます。

---

# Python Agentは仕様ではない

最初に誤解を解いておきます。

Python Agentは仕様ではありません。

Python AgentもDify Workflowと同じです。

仕様を実現するための実装形態です。

第7回では、

```text
Specification
↓
Dify Workflow
```

を扱いました。

今回は、

```text
Specification
↓
Python Agent
```

を扱います。

違うのは出口だけです。

正本はどちらもSpecificationです。

---

# 多くの人が逆から考える

AI開発を始めると、多くの人は最初にClaude CodeやCursor、ChatGPTを開きます。

そして、

「問い合わせ分類AIを作って」

と依頼します。

すると数分後には動くコードが生成されます。

これはAI時代の大きな変化です。

しかし、ここで一つの問題があります。

---

# 何を作るのかは決めてくれない

AIは

「どう作るか」

は得意です。

しかし、

「何を作るか」

は決めてくれません。

例えば問い合わせ分類AIなら、

- 何種類に分類するのか
- 優先度は何段階か
- 要約は何文字か
- 返信案は必要か

を決める必要があります。

これがSpecificationです。

---

# 問い合わせ分類AIで考える

例えば、Specificationが次のようになっているとします。

```text
入力

問い合わせ本文

出力

category
priority
summary
reply_draft

分類

問い合わせ
依頼
クレーム
その他

優先度

高
中
低

制約

summaryは30文字以内

reply_draftは100文字以内
```

ここにはPythonという言葉はありません。

しかし、この仕様を見るだけで、必要な処理はかなり明確になります。

---

# Agent設計へ変換する

このSpecificationを見ると、必要な役割が見えてきます。

```text
入力受付

↓

分類判定

↓

優先度判定

↓

要約生成

↓

返信案生成

↓

出力
```

これはまだ実装ではありません。

仕様を分解した結果です。

そして、この分解結果をもとにAgent設計を行います。

---

# Agentは仕様から生まれる

多くの人は、

- 分類Agent
- 要約Agent
- 返信Agent

を先に考えます。

しかしSDDでは逆です。

まずSpecificationを見る。

その後で、

必要ならAgentへ分割する。

つまり、

Agentは仕様の結果です。

Agentから仕様を考えるのではありません。

---

# Claude Codeの役割

ここでClaude Codeが登場します。

Claude Codeの役割は、

仕様を実装へ変換することです。

例えば、

```text
requirements.md

spec.md

を読み込み、

Pythonで実装してください。
```

と依頼できます。

するとClaude Codeは実装案を生成します。

つまり、

Claude Codeは設計者ではなく、

実装担当です。

---

# Claude Codeは設計者ではない

Claude Codeは非常に優秀です。

しかし設計責任者ではありません。

分類を4種類にするか。

6種類にするか。

優先度を3段階にするか。

5段階にするか。

それは業務側の判断です。

つまり人間の仕事です。

AIが進化するほど、

人間はコードを書く人ではなく、

仕様を定義する人になっていきます。

---

# Difyとの共通点

実は、第7回で扱ったDify Workflowと今回のPython Agentは本質的には同じです。

どちらもSpecificationから生まれます。

```text
Specification
↓
Dify Workflow
```

も、

```text
Specification
↓
Python Agent
```

も、

同じ構造です。

違うのは実装形態だけです。

---

# 実装は交換可能になる

ここがSDDの最大の価値です。

もし将来、

DifyからPythonへ移行したくなったら。

あるいは、

Pythonから別のツールへ移行したくなったら。

修正するべきなのはコードではありません。

Specificationです。

仕様が正本だからです。

実装は交換できます。

しかし仕様は交換できません。

---

# AI時代の人間の仕事

AIがコードを書く時代になりました。

だからこそ、人間の仕事は変わります。

実装を書くことではありません。

何を作るかを決めることです。

そして、その判断をSpecificationとして残すことです。

これがAI時代の設計者の役割です。

---

# シリーズ総括

第1回から第9回まで、

私たちは仕様駆動開発（SDD）について考えてきました。

重要なのはツールではありません。

Difyでもありません。

Claude Codeでもありません。

Specificationです。

仕様があるから実装を変えられる。

仕様があるから変更に強くなる。

仕様があるからAIを活用できる。

これが、私が考えるAI時代の仕様駆動開発です。

---

# その先にあるもの

仕様書からコードを生成することは、以前よりはるかに簡単になりました。

しかし、その先には新しい課題があります。

生成されたコードを、

どこで動かすのか。

どう運用するのか。

どう監視するのか。

どう安全に使い続けるのか。

実装の壁が低くなったからこそ、

運用の重要性はむしろ高まっています。

それは次のテーマです。

---

# おわりに

AIはますます進化していきます。

新しいツールも登場するでしょう。

しかし、

何を作るのか。

なぜそう作るのか。

それを決めるのは、

これからも人間です。

そして、その意思決定を残すものがSpecificationです。

AI時代だからこそ、

仕様を書く力が重要になる。

それが、このシリーズを通じて伝えたかったことです。

---

## 📚 SDD連載

▶ **SDDシリーズ一覧（ハブ記事）**
https://zenn.dev/tigerone1945/articles/sdd-hub-ai-sdd-series

◀ 第8回
【第8回】Difyと仕様駆動開発は何が違うのか
https://zenn.dev/tigerone1945/articles/sdd-08-dify-vs-spec-driven-development


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
