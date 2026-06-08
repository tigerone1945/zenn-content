---
title: "AIを作る時代から『AI Native』を設計する時代へ"
emoji: "🚀"
type: "idea"
topics:
  - AIエージェント
  - SDD
  - AI_Native
  - DX
  - AICX
published: true
---

# AIを作る時代から「AI Native」を設計する時代へ

生成AIの進化によって、AIを作ること自体は難しくなくなりました。

Difyを使えばノーコードでAIアプリを作れます。

ChatGPTを使えば高度な分析や文章生成ができます。

Claude CodeやKiroを使えば、仕様書からコードを生成することもできます。

しかし企業の現場を見ると、多くのAIプロジェクトは次のような課題に直面しています。

- PoCで終わる
- 一部の担当者しか使わない
- 現場に定着しない
- 運用できない
- 継続改善につながらない

AIを作ることと、AIを組織へ定着させることは全く別の問題です。

私はこれまでDX推進、RPA導入、AIエージェント開発、AWS基盤構築に関わる中で、

**「AIを作る人」よりも「AIを定着させる人」が重要になる**

と感じてきました。

そこで現在は、AIエージェント・ストラテジストという考え方で学習ロードマップを整理しています。

---

# AI導入は設計から始まる

多くの人は

```text
Dify
↓
Python
↓
AWS
```

という順番で学ぼうとします。

しかし実際のAI導入は、いきなりツール選定から始まるわけではありません。

まず必要なのは業務を理解することです。

例えば問い合わせ対応業務を考えてみます。

## As-Is（現状）

```text
問い合わせ受信
↓
担当者確認
↓
FAQ検索
↓
回答作成
↓
送信
```

この状態では

- 回答が遅い
- FAQ検索が手作業
- 担当者依存が大きい

といった課題があります。

次に理想状態を考えます。

## To-Be（理想）

```text
問い合わせ受信
↓
AI分類
↓
FAQ検索
↓
回答案生成
↓
担当者承認
↓
送信
```

ここで初めて

- 何をAI化するのか
- どこを人間が担当するのか
- どのKPIを改善するのか

が見えてきます。

つまりAI導入は

```text
As-Is
↓
To-Be
↓
Gap分析
↓
KPI
↓
BPR
```

から始まります。

私はこの工程こそが、AI導入で最も重要だと考えています。

---

# AICXとの対応

AICXの資格体系を整理してみると、AI導入の考え方が非常に体系的に整理されていることに気付きました。

特にChapter5の5Dモデルは興味深いです。

| AICX 5D | 実務での意味 | ロードマップ |
|----------|----------|----------|
| Discovery | As-Is分析 | 業務分析 |
| Definition | To-Be・KPI・BPR | 業務分析 |
| Design | 仕様設計 | SDD |
| Development | 実装 | Phase B / Phase C |
| Deployment | 運用 | Phase D |

つまりAI導入とは

```text
業務分析
↓
SDD
↓
実装
↓
運用
```

という流れなのです。

AICXはAI導入の考え方を体系化したものです。

私はその考え方を、実際に学習・実践できるロードマップへ落とし込みました。

それが次に説明するロードマップです。

---

# AI活用からAI Nativeまでのロードマップ

現在整理しているロードマップは以下です。

```text
業務分析
↓
SDD
仕様駆動開発

↓

Phase B
Dify Agent

↓

Phase C
Python Agent

↓

Phase D
AI Native

----------------

別軸
AIデータ分析
```

ポイントは、いきなりPythonやAWSへ進まないことです。

まず業務改善から始める。

次に設計を学ぶ。

その後に本格開発へ進む。

そして最後に運用まで考える。

この順番が重要だと考えています。

---

# Phase A
## Low Code AIエージェント

最初のステップです。

利用技術

- Dify
- GAS
- Google Workspace

テーマ例

- 問い合わせ分類
- メール自動化
- FAQ検索
- 議事録生成

ここでは

**「AIを業務へ組み込む力」**

を学びます。

現在は

- AIエージェント連載 https://zenn.dev/tigerone1945/articles/e0f4fcba591fb2
- AIエージェント無料Book https://zenn.dev/tigerone1945/books/7cbbd41a9953cd
- Udemy講座1 https://www.udemy.com/course/gas-dify-ai-agent-1/?referralCode=BDF338CD99C64780F085

で展開しています。

---

# なぜ次にSDDなのか

業務分析が終わると、次に必要になるのは設計です。

AI時代になると、コードを書く能力だけでは不十分になります。

必要になるのは

- 課題を定義する力
- 要求を整理する力
- 仕様を設計する力

です。

だからこそ私は

**SDD（仕様駆動開発）**

に注目しています。

---

# SDD
## 仕様駆動開発

利用技術

- Kiro
- Claude Code
- ChatGPT

学ぶ内容

- 要求定義
- 仕様設計
- AIとの協調開発

成果物のイメージ

```text
requirements.md

↓

specification.md

↓

tasks.md
```

現在は

- SDD連載
- Kindle
- Udemy

として今後展開を予定しています。

---

# Phase B
## Dify Agent

設計を学んだ後は、より高度なAgent構築へ進みます。

利用技術

- Workflow
- Agent
- Tool
- RAG

テーマ例

- 問い合わせトリアージ
- 営業支援Agent
- 社内ナレッジAgent

ここでは

**業務エージェントを設計・実装する力**

を学びます。

---

# Phase C
## Python Agent

Difyの外へ出る段階です。

利用技術

- Python
- OpenAI SDK
- LangGraph

テーマ例

- 問い合わせ分析Agent
- 在庫監視Agent
- IoT異常検知Agent

ここでは

**本格的なAIエージェント開発**

を学びます。

---

# Phase D
## AI Native

AIを本番運用する段階です。

利用技術

- AWS
- Serverless
- AgentOps
- Guardrail

多くのPoCはここで止まります。

しかし本当に重要なのは

**AIを運用し続けること**

です。

現在は

- AI基盤記事 https://zenn.dev/tigerone1945/articles/e0f4fcba591fb2
- AI基盤無料Book https://zenn.dev/tigerone1945/books/why-ai-poc-fails
- Kindle
- Udemy（予定）

として展開しています。

---

# もう一つの軸
## AIデータ分析

ここまでのロードマップはAIエージェントの世界です。

一方で私は別軸としてAIデータ分析も展開しています。

利用技術

- ChatGPT
- Python
- Google Colab

テーマ例

- 売上分析
- 顧客分析
- 問い合わせ分析
- 人事分析

目的は

**AIで意思決定を支援すること**

です。

---

# おわりに

AIを作れる人は増えています。

しかし、

- どの業務へ適用するか
- どの技術を選択するか
- どう運用するか
- どう定着させるか

まで考えられる人はまだ多くありません。

これから求められるのは

**AIを作る人ではなく、AIを組織へ定着させる人**

です。

私はそのためのロードマップとして

```text
AICX（戦略）
↓
SDD（設計）
↓
Phase B
↓
Phase C
↓
Phase D
```

という体系を整理しています。

本シリーズでは、このロードマップを順番に掘り下げていきます。

次回からはAICXシリーズとして、

**業務分析・KPI・BPR・5Dモデル**

を通じて、AI導入の考え方を整理していきます。

## 関連コンテンツ

AI導入や組織変革、AI Nativeについては、

Kindle
『AIエージェント・ストラテジスト
組織を動かす変革の戦略』

でも整理しています。
