<!-- 2026-11-09 ホテル予約Kindle予約受付・ASIN反映のReview。出版原稿にはこの変更確認部分を混入させない。 -->

# 変更確認：Zenn総合ハブ／ホテル予約Kindle予約注文

## 変更1：次期技術連載のKindle予約情報

**変更前**

- **ホテル予約データ分析Kindle**：2026年11月15日（日）発売予定。販売開始の確認までは発売済みと案内しません。数理最適化の教材用仮定値を実測効果と混同しません。

**変更後**

- **ホテル予約データ分析Kindle**：**『ChatGPTで学ぶホテル予約データ分析と業務課題解決：キャンセル率から改善施策・効果検証まで』は2026年11月15日（日）発売予定・[Amazonで予約注文受付中](https://www.amazon.co.jp/dp/B0HQYNZ8JF)**です。価格1,250円、ASIN `B0HQYNZ8JF`（2026年11月9日時点のAmazon・KDP表示）。**発売済みではありません。** 数理最適化の教材用仮定値を実測効果と混同しません。

## 変更2：商品選択表の予約リンク

**変更前**

| ホテル予約の数理最適化 | Python-MIP Zenn全6回／ホテル予約データ分析Kindle | 原稿v1.3保存、ローカル検証PASS報告済み。2027年公開日未定。Kindleは11/15発売予定 |

**変更後**

| ホテル予約の数理最適化 | Python-MIP Zenn全6回／[ホテル予約データ分析Kindle](https://www.amazon.co.jp/dp/B0HQYNZ8JF) | Zenn原稿v1.3保存・ローカル検証PASS報告済み。2027年公開日未定。Kindleは**11/15発売予定・予約注文受付中** |


---

## 更新後のハブ原稿

---
title: "AIエージェント実践ロードマップ──SDD・設計・実装・PoC・AWS・AgentCore本番まで"
emoji: "🧭"
type: "tech"
topics:
  - "aws"
  - "aiエージェント"
  - "sdd"
  - "claudecode"
  - "openaiagentssdk"
published: true
published_at: "2026-09-15 07:00"
---

AIエージェントを学び始めると、すぐにこんな疑問が出てきます。

- まずAgentを作ればいいのか
- SDDはどこで使うのか
- 業務課題をどうAIエージェント設計へ落とすのか
- Function ToolsやMulti-Agentはいつ使うのか
- PoCと本番は何が違うのか
- DockerやAWSはどの段階で必要になるのか
- AIエージェントを業務システムとしてどう育てるのか

個別の技術だけを追いかけていると、

```text
Agent
Function Tools
RAG
Multi-Agent
FastAPI
Docker
AWS
AgentCore
```

と知識がバラバラになりがちです。

そこでこの記事では、これまで公開してきたKindle・Zennシリーズと、これから公開するZenn実装シリーズ・Kindle・Udemyを、

> **AIエージェントを仕様から設計し、実装し、PoCを経てAWS / AgentCore本番環境へ育てるための学習ロードマップ**

として整理します。

この記事は固定記事ではありません。

今後公開するZenn・Kindle・Udemyに合わせてリンクを追加していく、**AIエージェント学習全体のハブ記事**として更新していきます。

---

# 全体ロードマップ

まずは全体像です。

```text
STEP 1
SDDを体系的に学ぶ
↓
Kindle
AI時代の仕様駆動開発
Kiro・Claude Codeで学ぶ
System Specification Design（SDD）の教科書

        ↓

STEP 2
業務課題をケーススタディへ落とす
↓
Kindle
AIエージェント開発のためのケーススタディ設計

As-Is / To-Be
→ Case Study
→ System Specification
→ Kiro Spec

        ↓

STEP 3
AIエージェント設計へ適用する
↓
Zenn
AIエージェント設計シリーズ

＋

Kindle
AIエージェント実践入門
公開済み

        ↓

STEP 4
AIエージェント技術を小さく実装する
↓
Zenn
AIエージェント実装実践シリーズ
第1回〜第21回

Agent
→ Tools
→ HITL
→ DB
→ Multi-Agent
→ API
→ Docker
→ PostgreSQL
→ Terraform
→ AWS
→ AgentCore

        ↓

STEP 5
FDE-Udemy：請求書処理・問い合わせトリアージ
As-Is → To-Be → SDD → ローカルPoC
Streamlit＋FastAPI＋SQLite

        ↓

STEP 6
AgentCore／クラウドは別枠で学習・検証
2027年3月に将来の商品化を審査
```

このロードマップで重要なのは、

> **設計・技術学習・業務実装・PoC・本番化を、同じものとして扱わない**

ことです。

それぞれ学習目的が違います。

---

# Zenn・note・Kindle・Udemyの役割（2026年10月改訂）

学習する技術と解決したい業務課題によって、入口と教材の役割を分けています。

| 媒体 | 主な役割 | 読者が得るもの |
|---|---|---|
| **Zenn** | 技術のHow | OpenAI Agents SDKなどを、1記事1テーマで小さく実装する手順 |
| **note（水曜）** | 設計のWhy | なぜその技術が必要か、どこに境界や責務を設けるか |
| **note（日曜）** | 業務課題・選定 | 導入判断、FW選択、データ分析から施策・効果検証まで |
| **Kindle（FW別）** | 体系的な実装教材 | 動く完成コード、設計、再現手順、検証結果を一冊で学べる |
| **Kindle（データ分析）** | 業務課題の解決 | ChatGPT中心の分析、原因仮説、改善施策、効果検証計画まで |
| **Udemy** | 独自の実践演習 | 構築の過程、エラー調査、自力での改修。需要と差別化がある場合に制作 |

**Kindle＝設計だけ、Udemy＝実装だけ、という区分ではありません。**
FW別Kindleは書籍だけで再現できる実装教材とし、Udemyは動画ならではのデバッグや要件変更演習で差別化します。

例えばFastAPIは、Zennで単独の実装要点を確認し、Kindleで構成・完成コードと再現方法を学び、必要ならUdemyで実装・失敗・改善を体験する関係です。

**このハブは「AIエージェントの実装・学習」の総合入口**です。業務課題・自動化の境界・設計判断の「Why」は、[AIエージェントは、何を設計すべきか？（note設計・活用ハブ）](https://note.com/loyal_hyssop7944/n/n111ba3f53507)へ。データ分析から施策立案までを扱うシリーズは、[データ分析は「集計した」で終わっていないか？（noteデータ分析・課題解決ハブ）](https://note.com/loyal_hyssop7944/n/nb88b1b78960c)を主導線とします。どちらのnoteハブも2026年10月19日（月）に公開済みです。

---

# STEP 1：まずSDDを体系的に学ぶ

AIエージェント開発でも、最初からAgentを書くのではなく、

> **何を作るのかを仕様として定義する**

ことが重要です。

その土台として位置づけているのが、公開済みのKindleです。

## 『AI時代の仕様駆動開発』

**Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書**

Amazon：

https://www.amazon.co.jp/dp/B0HBQ32KCQ

この本では、AI時代の仕様駆動開発として、

```text
System Specification
↓
Requirements
↓
Design
↓
Tasks
↓
Implementation
```

という流れを体系的に扱います。

AIコーディングツールを使えば、コード自体は以前より速く生成できます。

しかし、

> **何を作るのか**

が曖昧なままでは、生成速度が上がっても正しいシステムにはなりません。

そのため、AI時代ほど、

```text
Specification
↓
Design
↓
Implementation
```

という流れが重要になります。

この本は、後続のAIエージェント開発を理解するための**SDDの基礎編**に位置づけています。

---

# STEP 2：業務課題をケーススタディへ落とす

SDDの考え方を理解した次に必要なのが、

> **実際の業務課題を、どう仕様へ変換するのか**

という部分です。

そこで位置づけているのが、こちらの公開済みKindleです。

## 『AIエージェント開発のためのケーススタディ設計』

**As-Is / To-BeからSystem Specification・Kiro Specへつなぐ業務設計**

Amazon：

https://www.amazon.co.jp/dp/B0HJMM4JF7

この本では、

```text
Business Problem
↓
As-Is
↓
Pain Point
↓
To-Be
↓
Case Study
↓
System Specification
↓
Kiro Spec
```

という流れを扱います。

SDDを理解していても、

> 「では、実際の業務課題を何から書けばいいのか」

で止まることがあります。

そこで、

```text
現在の業務
↓
課題
↓
改善後の業務
↓
AIに任せる範囲
↓
システム仕様
```

へ変換していきます。

つまり、

### SDD-Kindle

では、

> **仕様駆動開発そのもの**

を学び、

### ケーススタディ設計Kindle

では、

> **業務課題をSDDの入力へ変換する方法**

を学ぶ、

という関係です。

---

# STEP 3：AIエージェント設計へ適用する

SDDとケーススタディ設計を理解したら、次はAIエージェントへ適用します。

ここで、これまでZennで公開してきた

# AIエージェント設計シリーズ

につながります。

このシリーズでは、

> **AIエージェントをどう設計するか**

を個別テーマに分けて扱ってきました。

考え方の基本は、

```text
Business Problem
↓
As-Is
↓
Pain Point
↓
To-Be
↓
System Specification
↓
Technology Constraints
↓
SDD
↓
Agent Design
```

です。

---

# AIエージェントはAgentから考え始めない

たとえば、

> 問い合わせ対応をAIエージェント化したい

という場合でも、いきなり、

```python
agent = Agent(...)
```

とはしません。

先に、

- 何が業務課題なのか
- 現在は誰が判断しているのか
- AIへ任せる範囲はどこか
- 人へ戻すべき判断は何か
- Toolが必要なのか
- Workflowで十分なのか
- Multi-Agentへ分ける必要があるのか

を整理します。

重要なのは、

> **AIエージェントありきではなく、業務課題ありき**

で設計することです。

---

# AIエージェントKindleも公開

ZennのAIエージェント設計シリーズで扱ってきた内容を、さらに体系化したKindleも公開しています。

## AIエージェント実践入門

https://www.amazon.co.jp/dp/B0HJSTH8BP

このKindleでは、

```text
Business Problem
↓
Case Study
↓
As-Is / To-Be
↓
System Specification
↓
Technology Constraints
↓
SDD
↓
Agent Design
↓
OpenAI Agents SDK
↓
Evaluation
```

という流れで、

> **業務課題を、実装可能なAIエージェントへ変換する方法**

を体系的に扱います。

単なるOpenAI Agents SDKの機能解説ではありません。

中心にあるのは、

> **技術を先に選ばず、業務課題から必要なAIエージェント構成を判断する**

という考え方です。

たとえば、

- Single Agentで十分なのか
- Function Toolsが必要なのか
- RAGが必要なのか
- Handoffを使うべきなのか
- Multi-Agentへ分割する必要があるのか
- Human-in-the-Loopをどこへ入れるのか

といった判断を扱います。

---

# 3冊のKindleの関係

ここまでのKindleを整理すると、次のようになります。

| Kindle | 主な役割 | 状況 |
|---|---|---|
| AI時代の仕様駆動開発 | SDDそのものを学ぶ | 公開済み |
| AIエージェント開発のためのケーススタディ設計 | 業務課題を仕様へ変換する | 公開済み |
| AIエージェント実践入門 | SDDをAIエージェント設計へ適用する | 公開済み |

学習順としては、

```text
SDDを理解する
↓
業務課題を仕様へ変換する
↓
AIエージェントへ適用する
```

となります。

---

# STEP 4：設計から「実装」へ

設計が理解できたら、次はコードへ落としていきます。

ここから始めるのが、新しい

# AIエージェント実装実践シリーズ

です。

このシリーズでは、一つの小さなサンプルを段階的に育てながら、OpenAI Agents SDKとApplication開発の主要技術を学びます。

最初から完成システムは作りません。

```text
Agent
↓
Function Tools
↓
Structured Output
↓
Web UI
↓
Sessions
↓
Tracing
↓
Guardrails
↓
HITL
↓
Database
↓
Multi-Agent
↓
FastAPI
↓
Docker
↓
PostgreSQL
↓
Terraform
↓
AWS
↓
Monitoring
↓
AgentCore
```

と、一つずつ機能を追加していきます。

---

# AIエージェント実装実践シリーズ 全21回

2026年11月5日（木）時点で、**第1〜11回は公開済み、第12〜21回は公開予定**です。第8回以降は原則として**毎週火曜日に1回**公開します（第1〜7回は火曜・金曜中心の公開実績）。今後の公開予定は変更される場合があります。

| 回 | テーマ | 主な技術 | 状況 |
|---:|---|---|---|
| 第1回 | 最小AIエージェントを作る | Agent / Runner | 公開済み |
| 第2回 | 処理をToolへ分離する | Function Tools | 公開済み |
| 第3回 | 出力を構造化する | Structured Output / Pydantic | 公開済み |
| 第4回 | Web画面を作る | Streamlit | 公開済み |
| 第5回 | 会話と実行を追跡する | Sessions / Tracing | 公開済み |
| 第6回 | 危険な処理を止める | Guardrails | 公開済み |
| 第7回 | 人の判断を入れる | HITL | 公開済み |
| 第8回 | データを永続化する | SQLite | 公開済み |
| 第9回 | [複数Tableを安全に扱う](https://zenn.dev/tigerone1945/articles/aais-09-multitable-join-validation) | Join Validation | 公開済み |
| 第10回 | [Agentを役割分担する](https://zenn.dev/tigerone1945/articles/aais-10-multiagent-manager-pattern) | Multi-Agent | 公開済み |
| 第11回 | [AIエージェントをAPI化する](https://zenn.dev/tigerone1945/articles/aais-11-fastapi-agent-api) | FastAPI | 公開済み |
| 第12回 | 実行環境をContainer化する | Docker / Docker Compose | 公開予定 |
| 第13回 | 本番DBへ移行する | PostgreSQL | 公開予定 |
| 第14回 | AWS基盤をコード化する | Terraform | 公開予定 |
| 第15回 | AWSへデプロイする | ECR / ECS / Fargate / RDS | 公開予定 |
| 第16回 | 本番環境を監視する | CloudWatch / Secrets / Evaluation | 公開予定 |
| 第17回 | AgentCore Runtimeへ載せる | AgentCore Runtime / AgentCore CLI | 公開予定 |
| 第18回 | 記憶を管理する | AgentCore Memory | 公開予定 |
| 第19回 | ToolをManaged化する | AgentCore Gateway | 公開予定 |
| 第20回 | 誰の権限で呼ぶかを設計する | AgentCore Identity | 公開予定 |
| 第21回 | 監視・評価を運用Loopにする | AgentCore Observability / Evaluations | 公開予定 |

---

# 第1回：最小AIエージェントを作る

最初は極力シンプルにします。

```text
User
↓
Agent
↓
Response
```

扱うのは、

- Agent
- Instructions
- Runner
- Result

です。

目的は、

> **OpenAI Agents SDKでAgentが動く最小構成を理解する**

ことです。

https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data

---

# 第2回：Function Toolsへ処理を分離する

AIエージェントに、すべてをLLMだけで処理させるわけではありません。

```text
Agent
↓
Function Tool
↓
Python
↓
Result
↓
Agent
```

と分けます。

重要なのは、

> **LLMが考える部分と、コードで確実に処理する部分を分離する**

ことです。

https://zenn.dev/tigerone1945/articles/aais-02-function-tools-pandas-csv-analysis

---

# 第3回：Structured Outputで出力を安定させる

自然言語だけでは、後続のApplicationから利用しにくくなります。

そこで、

```text
Agent
↓
Pydantic Model
↓
Structured Output
```

とします。

たとえば、

```text
result
confidence
reason
next_action
```

のように、出力構造を定義します。

https://zenn.dev/tigerone1945/articles/aais-03-structured-output-pydantic

---

# 第4回：StreamlitでWeb画面を作る

CLIだけでは、実際の利用者には使いにくい場合があります。

そこで、

```text
Browser
↓
Streamlit
↓
Agent
```

へ発展させます。

ここから、

> **Agentそのものだけでなく、人が使うApplication**

として考え始めます。

https://zenn.dev/tigerone1945/articles/aais-04-streamlit-web-ui

---

# 第5回：SessionsとTracingを追加する

次に、

> 前の会話を覚えてほしい

という要求が出てきます。

そこでSessionsを使います。

一方、

> Agentが内部で何をしたのか確認したい

場合に使うのがTracingです。

```text
Sessions
＝会話を継続する

Tracing
＝実行を追跡する
```

役割は異なります。

https://zenn.dev/tigerone1945/articles/aais-05-sessions-tracing

---

# 第6回：Guardrailsで危険な処理を止める

AIエージェントを業務で利用する場合、

> Instructionsに「やらないでください」と書く

だけでは不十分です。

そこでGuardrailsを導入します。

- 不適切な入力
- 対象外の要求
- 危険な出力
- 禁止された処理

などを制御します。

https://zenn.dev/tigerone1945/articles/aais-06-guardrails

---

# 第7回：Human-in-the-Loopを入れる

AIエージェントのゴールは、必ずしも完全自動化ではありません。

```text
AI
↓
Decision
↓
Low Risk → Automatic
High Risk → Human Review
```

という構成も重要です。

ここでは、

- Approval
- Reject
- Resume
- Human Decision

を扱います。

https://zenn.dev/tigerone1945/articles/aais-07-human-in-the-loop

---

# 第8回：SQLiteでデータを永続化する

次に、

```text
一回実行
↓
終了
```

ではなく、

```text
実行結果
↓
Database
↓
再利用
```

へ進みます。

ここではSQLiteを使います。

SQLiteは、小規模ApplicationやPoCで扱いやすいデータベースです。


[第8回：CSVからSQLiteへ──AIエージェントの業務データを永続化する](https://zenn.dev/tigerone1945/articles/aais-08-csv-to-sqlite)

---

# 第9回：複数Tableを安全に扱う

業務データは、1つのTableだけとは限りません。

```text
customers
orders
products
```

のような複数Tableを扱う場合、

```text
Join候補
↓
Join Key確認
↓
Validation
↓
Preview
↓
Join
```

という流れが必要になります。

重要なのは、

> **AIに自由なSQLを書かせるのではなく、安全な操作をToolとして提供する**

ことです。

[第9回：複数Tableを安全に扱う──Join ValidationでAIに自由なSQLを書かせない](https://zenn.dev/tigerone1945/articles/aais-09-multitable-join-validation)

---

# 第10回：Multi-Agentへ発展させる

Agentの責務が大きくなってきたら、役割を分けます。

```text
Manager Agent
├─ Specialist Agent A
├─ Specialist Agent B
└─ Specialist Agent C
```

ここでは、

- Manager Pattern
- Agents as Tools
- Agent Responsibility
- Multi-Agent Design

を扱います。

重要なのは、

> **最初からMulti-Agentにしない**

ことです。

Single Agentで責務が大きくなったときに、必要な部分だけ分離します。

[第10回：Agentを役割分担する──Manager PatternでMulti-Agent化する](https://zenn.dev/tigerone1945/articles/aais-10-multiagent-manager-pattern)

---

# 第11回：FastAPIでAPI化する

ここからAIエージェントをApplicationとして考えます。

Before：

```text
Streamlit
↓
Agent
```

After：

```text
Streamlit
↓
FastAPI
↓
Agent
```

とします。

これによりFrontendとAgent Runtimeを分離できます。

[第11回：AIエージェントをFastAPIでAPI化する──UIとAgentの境界を分離する](https://zenn.dev/tigerone1945/articles/aais-11-fastapi-agent-api)

扱うのは、

- FastAPI
- Endpoint
- Request / Response
- Pydantic
- Service Layer
- API Boundary

です。

---

# 第12回：Dockerで実行環境を固定する

ローカル環境だけで動いている状態から、

> 自分のPCでは動く

を卒業します。

```text
Docker Compose
├─ frontend
│  └─ Streamlit
│
└─ backend
   └─ FastAPI + Agents SDK
```

ここでの目的はDockerを極めることではありません。

> **AIエージェントを実行環境から独立させる**

ことです。

---

# 第13回：PostgreSQLへ移行する

SQLiteから、

```text
SQLite
↓
PostgreSQL
```

へ移行します。

Architectureも、

```text
Agent
↓
Tool
↓
Repository
↓
Database
```

とし、AgentがDB製品そのものへ強く依存しない構成を考えます。

ここから本番Applicationを意識します。

---

# 第14回：TerraformでAWS基盤を作る

ここからCloudへ進みます。

想定するAWS構成は、

```text
ECR
ECS / Fargate
RDS
Secrets Manager
CloudWatch
```

などです。

AWS Consoleだけで構築するのではなく、

> **TerraformでInfrastructure as Codeとして定義する**

ことを学びます。

---

# 第15回：AIエージェントをAWSへデプロイする

ローカルで育ててきたAIエージェントをAWSへ移します。

```text
Docker Image
↓
ECR
↓
ECS / Fargate
↓
FastAPI
↓
Agents SDK
↓
RDS PostgreSQL
```

第1回ではローカルで動くだけだったAgentが、

> **Cloud上で動くApplication**

になります。

この回がシリーズの大きな到達点です。

---

# 第16回：本番AIエージェントを監視する

AWSへデプロイできても、それだけでは運用できません。

そこで、

```text
CloudWatch
Secrets Manager
Tracing
Application Log
Error
Latency
Evaluation
```

を扱います。

ECS / Fargate版の最終的なイメージは、

```text
User
↓
Frontend
↓
FastAPI
↓
AI Agent
↓
Function Tools
↓
PostgreSQL / RDS
↓
Tracing / Logging
↓
Docker
↓
ECS / Fargate
↓
CloudWatch
```

です。

第1回の小さなAgentを、

> **運用を意識したAI Application**

まで育てます。

ここまでが、汎用的なAWS ApplicationとしてAIエージェントを本番化するルートです。

第17回からは、同じAgentを**Agent専用のManaged Service**で本番化するもう一つのルートを扱います。

---

# 第17回：AgentCore Runtimeへ載せる

第16回までのAgentを、Amazon Bedrock AgentCore Runtimeへ載せます。

```text
OpenAI Agents SDK
↓
BedrockAgentCoreApp
↓
AgentCore Runtime
```

既存のAgent本体は書き直さず、外側にRuntime Adapter（`app.py`）を追加するだけです。

Entrypointも外部入力の入口として、入力検証とエラー処理を入れます。

開発の流れは、

```text
agentcore dev
↓
agentcore deploy
↓
agentcore invoke
```

です。

ECS / Fargateと置き換えるのではなく、

> **汎用AWS Application**と**Agent向けManaged Service**を、2つのProduction Routeとして比較する

ことがこの回のテーマです。

AWS SDKからの呼び出し（Timeout・Retry・IAM）や、RDSへの接続経路も整理します。

---

# 第18回：AgentCore Memoryで記憶を管理する

第5回のSessionsは、同じ会話の文脈を保つ仕組みでした。

しかし、

> **「同じ会話を覚える」と「次の会話でも覚えている」は別**

です。

```text
Session
↓
Short-term Memory
↓
Long-term Memory
```

のように記憶を分け、何を保存し、何を保存しないかをMemory Policyとして設計します。

売上の数値のように変化するBusiness FactはDBを正とし、Memoryには好みなどの安定した情報だけを残します。

MemoryとRAGの違い、Memoryが取れなくてもAgentが動く設計も整理します。

---

# 第19回：AgentCore GatewayでToolをManaged化する

これまでToolはAgentのコード内にFunction Toolとして持っていました。

第19回では、Toolの接続をGatewayへ切り出します。

```text
Agent
↓
AgentCore Gateway
↓
Tool / API
```

GatewayのEndpointはMCP Serverとして扱えます。Side Effectを伴う`create_review_request`をLambda Toolとして切り出し、AgentからMCP経由で呼び出します。

ただし、Toolをすべて移す必要はありません。

```text
Local Tool
Gateway Tool
```

のどちらへ置くかを、判断基準として整理します。

第7回のHuman-in-the-Loopとのつなぎ方も扱います。

---

# 第20回：AgentCore Identityで「誰の権限で呼ぶか」を設計する

Toolを外部へつなぐと、

> **誰の権限で、そのToolを呼ぶのか**

が問題になります。

```text
Inbound Auth
誰がAgentを呼ぶか

Outbound Auth
Agentが誰として外部Toolを呼ぶか
```

を分けて考え、API Keyの直書きをやめてCredential Providerへ移します。

権限の判定はPromptではなくCodeで行う、という原則も扱います。

第16回のSecrets Managerとの違い、OAuth Consent、Workload Identity、Gateway・Identity・HITLの役割分担も整理します。

---

# 第21回：AgentCore ObservabilityとEvaluationsで監視・評価する

最後に、第16回で行った監視と評価を、AgentCoreのManaged Capabilityで拡張して比較します。

```text
Runtime
Memory
Gateway
Identity
↓
Observability
↓
Evaluations
↓
改善
```

Traceから遅延やErrorの原因の層（Runtime / Gateway / Memory / Auth）を絞り込みます。

Custom Evaluatorで売上分析Agent向けの評価軸を作り、Code-basedとLLM-as-a-Judgeを使い分けたうえで、

```text
観測する
評価する
改善する
```

というLoopを回します。見つけた失敗はEvaluation Caseへ戻します。

第1回の小さなAgentが、

> **ECS / Fargate版とAgentCore版、2つの本番ルートを比較できるAI Application**

まで育ちます。

---

# 実装シリーズは5つのPhaseで進む

21回を大きく分けると、次の5段階になります。

## Phase 1：Single Agent

```text
第1回 Agent
↓
第2回 Function Tools
↓
第3回 Structured Output
↓
第4回 Streamlit
↓
第5回 Sessions / Tracing
↓
第6回 Guardrails
↓
第7回 HITL
↓
第8回 SQLite
↓
第9回 Multi-table
```

まず、1つのAgentを利用可能なApplicationへ近づけます。

---

## Phase 2：Multi-Agent

```text
第10回
Multi-Agent
```

必要になった責務だけを分離します。

---

## Phase 3：Application

```text
第11回 FastAPI
↓
第12回 Docker
↓
第13回 PostgreSQL
```

ここから、

> **AIエージェント開発**

から、

> **AIエージェントを組み込んだApplication開発**

へ進みます。

---

## Phase 4：Production

```text
第14回 Terraform
↓
第15回 AWS Deploy
↓
第16回 Monitoring
```

> **ローカルで動くAgentをCloud上で運用できる構成へ育てる**

ところまで進みます。

---

## Phase 5：AgentCore

```text
第17回 Runtime
↓
第18回 Memory
↓
第19回 Gateway
↓
第20回 Identity
↓
第21回 Observability / Evaluations
```

同じAgentを、

> **Agent専用のManaged Serviceで本番化するもう一つのルート**

として比較します。

---

# Kindle・Zenn・Udemyは何が違うか

- **Zennの実装21回**：同じ売上・受注データを使い、Agent / Tools / Guardrails / Database / Multi-Agent / AWS / AgentCoreの実装を段階的に学びます。
- **OpenAI-SDK-Kindle（v2.0／販売中）**：正式タイトル候補は『OpenAI Agents SDKで育てるAIエージェント』、副題は「売上データ分析からMulti-Agent・Docker・AWS本番運用まで」。**2026年10月29日（木）発売、価格1,250円、ASIN：B0HNGDT575**（10月31日時点でKDP「販売中」）。[Amazonの商品ページ](https://www.amazon.co.jp/dp/B0HNGDT575)。Zenn第1〜16回相当の技術を中心に、完成コード・再現手順・検証を体系化します。**Amazon Bedrock AgentCoreは本書v2.0の対象外**です。
- **FDE-Udemy（請求書処理・問い合わせトリアージ／制作中）**：As-Is／To-Be → SDD → Agent実装・テスト → Streamlit＋FastAPI＋SQLiteの**ローカルPoC**を扱います。**Docker化・クラウドデプロイは本編に含めません**。2026年末〜2027年初頭の公開見込みですが確定日は未設定です。
- **AgentCore／クラウド移行Udemy（別枠の将来候補）**：2027年3月までは学習・AWS実機検証が中心。ローカルPoCと実機検証の結果からGO／HOLD／NO-GOを判断します。

FW別Udemyは書籍発売に合わせて自動的に作りません。開始・途中・完成スナップショット、障害再現、デバッグ、書籍にない改修演習、テスト・正解例、独立受講可能性を満たした場合に制作判断を行います。

---

# FDE型業務AI化｜請求書処理とトリアージ

旧「Kindle 4〜6／Udemy講座4〜7」の独立発売計画は採用せず、FDEの業務ケーススタディを優先します。

```text
As-Is・業務課題 → To-Be・業務ルール
↓
Case Study／Spec／Constraints
↓
SDD・Claude CodeによるAgent実装
↓
Streamlit＋FastAPI＋SQLite
↓
正常系・異常系・承認／Guardrailsの検証
↓
ローカルPoC・受講者向け教材
```

**FDE-Udemy本編にDocker・AWS/Azure/GCPへのデプロイは含めません。** 請求書処理とトリアージを優先し、2026年末〜2027年初頭の公開を目指します。追加ケースはその後に選びます。

## AgentCoreはクラウド学習・将来企画

2027年3月まではOpenAI Agents SDK×AgentCore Runtime等の学習と小規模AWS実機検証を優先します。Runtimeの動作、認証、監視、費用、削除手順を証跡付きで確認し、クラウド移行UdemyのGO／HOLD／NO-GOを判断します。**現時点でUdemy発売は未定**です。

---

# 技術を学ぶ順番と、業務システムを育てる順番は違う

AIエージェント技術だけを見ると、

```text
Agent
↓
Function Tools
↓
Structured Output
↓
Sessions
↓
Tracing
↓
Guardrails
↓
HITL
↓
Database
↓
Multi-Agent
↓
API
↓
Docker
↓
PostgreSQL
↓
AWS
↓
Monitoring
↓
AgentCore Runtime
↓
Memory / Gateway / Identity
↓
Observability / Evaluations
```

という順番で学べます。

これがZenn実装シリーズです。

一方、実際の業務システムでは、

```text
Business Problem
↓
Specification
↓
BUILD
↓
VALIDATE
↓
OPERATE
```

という流れになります。

こちらがKindle・Udemy側のストーリーです。

この2つは似ていますが、同じではありません。

---

# AIにすべて任せることがゴールではない

AIエージェントという言葉から、

> AIがすべて自動で処理する

というイメージを持つかもしれません。

しかし業務システムでは、

```text
AI
↓
Confidence / Risk
↓
Low Risk
→ Automatic

High Risk
→ Human Review
```

という構成が重要になることがあります。

特に、

- 金額に関わる判断
- 顧客への重要な回答
- 契約
- 審査
- 業務ルール
- 経営判断

では、

> **AIが正しく人へ戻せること**

も重要な能力です。

---

# AIが考える部分とコードで決める部分を分ける

シリーズ全体を通して重視する考え方の1つです。

LLMが得意なのは、

- 文脈理解
- 分類
- 意味解釈
- 仮説生成
- 要約
- 提案

です。

一方、

- 数値計算
- 閾値判定
- Business Rule
- Database Constraint
- Security Check

などは、決定論的コードの方が適しています。

```text
LLM
＝Reasoning / Interpretation

Code
＝Calculation / Rule / Constraint
```

この境界を意識します。

---

# 次の技術シリーズ：Python-MIPを先行

ZennのOpenAI Agents SDK全21回を完走した後は、CrewAIの新連載より**Python-MIPによるホテル予約の数理最適化（全6回）**を優先します。

- **原稿**：[GitHub `upload_to_zenn/python_mip/`](https://github.com/tigerone1945/upload_to_zenn/tree/main/python_mip) に**全6回v1.3保存済み**。Zennには未公開（`published: false`）。
- **検証**：2026年11月3日のローカル実行報告では全回PASS。公開前には検証時の版とv1.3の差分、校正・表示を確認します。
- **開始日**：2027年の公開候補。**2027年1月19日（火）は確定日ではありません**。既存のZenn・note枠との調整後に決定します。
- **ホテル予約データ分析Kindle**：**『ChatGPTで学ぶホテル予約データ分析と業務課題解決：キャンセル率から改善施策・効果検証まで』は2026年11月15日（日）発売予定・[Amazonで予約注文受付中](https://www.amazon.co.jp/dp/B0HQYNZ8JF)**です。価格1,250円、ASIN `B0HQYNZ8JF`（2026年11月9日時点のAmazon・KDP表示）。**発売済みではありません。** 数理最適化の教材用仮定値を実測効果と混同しません。
- **CrewAI**：Kindleは[販売中](https://www.amazon.co.jp/dp/B0HNG2K4C8)。CrewAIの新しいZenn連載はPython-MIPの後で検討します。

---

# noteと別ハブへの導線

水曜noteでは、火曜Zennのコードを転載せず、「なぜ必要か」「何をAIに任せてよいか」「業務のどこに境界を置くか」を説明します。2026年10月は入門編を継続し、11月4日（水）・11日（水）・18日（水）は過去のZenn回を複数まとめた設計記事、**11月25日（水）から火曜Zennと1対1同期する計画**です。

日曜noteは独立した業務設計・FW比較・データ分析の記事を扱います。**データ分析は別ライン**として、記事→「AIデータ分析・課題解決 noteハブ」→業務課題別Kindleへ誘導します。

公開済みのnoteハブ：

- [AIエージェントは、何を設計すべきか？｜業務課題・自動化・本番運用の実践ガイド](https://note.com/loyal_hyssop7944/n/n111ba3f53507)
- [データ分析は「集計した」で終わっていないか？｜AIデータ分析・課題解決ガイド](https://note.com/loyal_hyssop7944/n/nb88b1b78960c)

両ハブは2026年10月19日（月）公開済みです。今後の個別note記事は、公開後に正式URLを確認して追加します。

---

# 商品の選び方と現在の計画

| 学びたいこと | 主な教材・商品 | 状態・注意点 |
|---|---|---|
| 仕様駆動開発の考え方 | 『AI時代の仕様駆動開発』 | 公開済み（下部にリンク） |
| 業務課題をSDDへ変える | 『AIエージェント開発のためのケーススタディ設計』 | 公開済み（下部にリンク） |
| 業務AIエージェントの設計 | 『AIエージェント実践入門』 | 公開済み（下部にリンク） |
| Agents SDKを小さく実装 | Zenn実装実践シリーズ全21回 | 火曜連載中。公開回のみ順次リンク追加 |
| 売上分析Agentの第1〜16回相当の実装を体系化 | [OpenAI-SDK-Kindle（v2.0）](https://www.amazon.co.jp/dp/B0HNGDT575) | 10/29（木）発売・販売中。AgentCoreは対象外 |
| 請求書処理・トリアージの業務AI化 | FDE-Udemy（ローカルPoC） | Streamlit＋FastAPI＋SQLite。2026年末〜2027年初頭の公開見込み、日付未確定。Docker・クラウド対象外 |
| AgentCoreのクラウド学習 | AWS実機検証／将来の独立Udemy候補 | 2027年3月までの実機検証後に商品化判断 |
| ホテル予約の数理最適化 | Python-MIP Zenn全6回／[ホテル予約データ分析Kindle](https://www.amazon.co.jp/dp/B0HQYNZ8JF) | Zenn原稿v1.3保存・ローカル検証PASS報告済み。2027年公開日未定。Kindleは**11/15発売予定・予約注文受付中** |
| CrewAIの体系的実装 | [CrewAI-Kindle](https://www.amazon.co.jp/dp/B0HNG2K4C8) | 10/31（土）発売・販売中。Udemyは条件付き |
| データ分析から施策・検証計画 | [データ分析noteハブ](https://note.com/loyal_hyssop7944/n/nb88b1b78960c)／データ分析Kindle | 独立した商品ライン。実装21回とは混同しない |

※ **『ChatGPTで学ぶ問題解決型データ分析 実践入門』は2026年10月27日（火）発売、価格1,250円、ASIN：B0HL4PS2QCで販売中**です（10月31日時点でKDP「販売中」／[Amazon商品ページ](https://www.amazon.co.jp/dp/B0HL4PS2QC)）。300件の受注データを使ったChatGPT中心の分析・施策立案・効果評価計画を扱います。**OpenAI-SDK-Kindle・CrewAI-Kindleとは別の商品**です。主導線は**データ分析noteハブ**とします。

# このハブ記事の使い方

目的に応じて、読む場所を選んでください。

---

## SDDそのものを学びたい

### Kindle
**『AI時代の仕様駆動開発: Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書』**

https://www.amazon.co.jp/dp/B0HBQ32KCQ

```text
System Specification
↓
Requirements
↓
Design
↓
Tasks
↓
Implementation
```

を体系的に学びます。

---

## 業務課題を仕様へ変換したい

### Kindle
**『AIエージェント開発のためのケーススタディ設計: As-Is / To-BeからSystem Specification・Kiro Specへつなぐ業務設計』**

https://www.amazon.co.jp/dp/B0HJMM4JF7

```text
As-Is
↓
Pain Point
↓
To-Be
↓
Case Study
↓
System Specification
↓
Kiro Spec
```

を扱います。

---

## AIエージェントの設計を学びたい

→ **Zenn AIエージェント設計シリーズ**

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

---

## AIエージェント開発を体系的に学びたい

### Kindle
**AIエージェント実践入門**

https://www.amazon.co.jp/dp/B0HJSTH8BP

```text
Business Problem
↓
System Specification
↓
SDD
↓
Agent Design
↓
OpenAI Agents SDK
↓
Evaluation
```

までを体系的に扱います。

---

## AIエージェント技術を実装しながら学びたい

→ **Zenn AIエージェント実装実践シリーズ**

第1回から第21回まで、

```text
Agent
↓
Tools
↓
Web
↓
Safety
↓
HITL
↓
Database
↓
Multi-Agent
↓
FastAPI
↓
Docker
↓
PostgreSQL
↓
Terraform
↓
AWS
↓
Monitoring
↓
AgentCore
```

と段階的に進みます。

---

## 業務AIエージェントを実際に作りたい

→ **FDE-Udemy（請求書処理／問い合わせトリアージ）**。As-Is／To-Be、SDD、Streamlit＋FastAPI＋SQLiteを用いたローカルPoCを教材化します。公開日は未確定です。

## クラウドへの移行を学びたい

→ Zenn第17〜21回のAgentCore技術編を読み、実機ではRuntime等から段階的に検証します。クラウドデプロイUdemyは別企画とし、2027年3月に制作可否を判断します。

---

# 今後の更新予定

このハブは**新しい記事・商品が公開されたときに更新する常設の総合入口**です。

1. **最優先**：OpenAI Agents SDK第1〜21回の公開状況と個別リンクを追加（火曜連載を完走）。
2. 対応する水曜noteが公開されたら、各技術のWhy記事と相互リンクする。
3. **OpenAI-SDK-Kindle（v2.0）**は10月29日（木）、**CrewAI-Kindle**は10月31日（土）に発売しました（10月31日時点でKDP「販売中」）。Amazonの商品リンクは上記に掲載しています。OpenAI-SDK-KindleにAgentCoreは含めません。
4. **Python-MIP全6回**は原稿v1.3のGitHub保存済み・2026年11月3日のローカル検証完了報告済み。2027年の公開枠と校正を調整する。
5. **FDE-Udemy（請求書処理・トリアージ）**はローカルPoCと教材品質を優先して制作し、公開後に正式URLを追加する。
6. **AgentCoreのクラウド実機検証**は2027年3月まで実施し、独立UdemyのGO／HOLD／NO-GOを判断する。
7. **CrewAIの新規Zenn連載**はPython-MIPの後に検討する。データ分析noteハブとの導線は維持する。

**運用原則**：公開済み／予定、原稿完成／実機検証を区別し、既存の記事URLとslugを維持します。

---

# まとめ

AIエージェント開発は、

```text
Agentを作る
```

だけでは終わりません。

全体では、

```text
SDDを理解する
↓
業務課題を仕様へ変換する
↓
AIエージェントを設計する
↓
小さく実装する
↓
Tools / Safety / HITLを追加する
↓
Databaseへ接続する
↓
Multi-Agentへ発展させる
↓
API化する
↓
Docker化する
↓
PostgreSQLへ移行する
↓
TerraformでAWS基盤を作る
↓
AWSへデプロイする
↓
監視・運用する
↓
AgentCore Runtimeへ載せる
↓
Memory / Gateway / Identityを設計する
↓
Observability / Evaluationsで改善Loopを回す
```

まで続きます。

この学習ロードマップでは、

```text
SDD-Kindle
↓
ケーススタディ設計Kindle
↓
Zenn AIエージェント設計シリーズ
↓
AIエージェント実践入門 Kindle
↓
Zenn AIエージェント実装実践シリーズ
↓
Udemy BUILD
↓
Kindle / Udemy VALIDATE
↓
Kindle / Udemy OPERATE
```

という流れで、

> **AIエージェントを「作る」のではなく、業務システムとして「育てる」**

ことを目指します。

現在、

# AIエージェント実装実践シリーズ

を順次公開しています。

第1回の

> **最小構成から始めるAIエージェント実装**

から小さなAgentを育て、第16回でAWS上の運用へ、第17〜21回でAgentCoreを使ったもう一つの本番ルートへつなげていきます。

https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data

---

# 次に読む

## SDDを学ぶ

**『AI時代の仕様駆動開発: Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書』**

https://www.amazon.co.jp/dp/B0HBQ32KCQ

---

## 業務課題を仕様へ変換する

**『AIエージェント開発のためのケーススタディ設計: As-Is / To-BeからSystem Specification・Kiro Specへつなぐ業務設計』**

https://www.amazon.co.jp/dp/B0HJMM4JF7

---

## AIエージェント設計シリーズ

AIエージェントの設計から学びたい方はこちら。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

---

## AIエージェント実践入門 Kindle

https://www.amazon.co.jp/dp/B0HJSTH8BP

業務課題・SDD・Agent Design・OpenAI Agents SDK・Evaluationまでを体系的に扱います。

---

## AIエージェント実装実践シリーズ

**第1〜11回は公開済みです。** 上記各回の見出しから内容と記事URLを確認できます。

[第1回：Agent / Runner](https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data) ／ [第8回：SQLite](https://zenn.dev/tigerone1945/articles/aais-08-csv-to-sqlite) ／ [第9回：Join Validation](https://zenn.dev/tigerone1945/articles/aais-09-multitable-join-validation) ／ [第10回：Multi-Agent](https://zenn.dev/tigerone1945/articles/aais-10-multiagent-manager-pattern) ／ [第11回：FastAPI](https://zenn.dev/tigerone1945/articles/aais-11-fastapi-agent-api)

---

## Udemy

**FDE-Udemyでは請求書処理と問い合わせトリアージを題材に、As-Is／To-Be → SDD → 実装 → テスト → ローカルPoCまでの実践講座を制作中**です。Docker・クラウド化は別企画であり、発売日は未確定です。

※ 公開後リンク追加