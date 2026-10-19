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
業務AIエージェントを構築する
↓
Udemy講座4
BUILD

        ↓

STEP 6
PoCとして検証する
↓
Kindle 5（PoC-Kindle）
VALIDATE Design

＋

Udemy講座5
VALIDATE Implementation

        ↓

STEP 7
AWS本番環境へ育てる
↓
Kindle 6（Prod-Kindle）
OPERATE Design

＋

Udemy講座6
OPERATE Implementation
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

2026年10月20日（火）時点で、**第1〜9回は公開済み、第10〜21回は公開予定**です。第8回以降は原則として**毎週火曜日に1回**公開します（第1〜7回は火曜・金曜中心の公開実績）。今後の公開予定は変更される場合があります。

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
| 第10回 | Agentを役割分担する | Multi-Agent | 公開予定 |
| 第11回 | AIエージェントをAPI化する | FastAPI | 公開予定 |
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
- **OpenAI-SDK-Kindle（v2.0／予約受付中）**：正式タイトル候補は『OpenAI Agents SDKで育てるAIエージェント』、副題は「売上データ分析からMulti-Agent・Docker・AWS本番運用まで」。**2026年10月29日（木）発売予定、価格1,250円、ASIN：B0HNGDT575**（10月20日時点でKDP「予約注文（オンライン）」）。[Amazonの予約注文ページ](https://www.amazon.co.jp/dp/B0HNGDT575)。Zenn第1〜16回相当の技術を中心に、完成コード・再現手順・検証を体系化します。**Amazon Bedrock AgentCoreは本書v2.0の対象外**です。
- **トリアージUdemy講座4〜6**：問い合わせ業務をBUILD→VALIDATE→OPERATEまで育てる、**売上分析21回とは別題材**の実践講座です。
- **Udemy講座7（AgentCore／別企画）**：トリアージ講座6のCompleted Source Snapshotから始め、Bedrock AgentCoreで本番運用する内容です。**OpenAI-SDK-Kindleには含めないAgentCoreの実装・運用を扱う別題材の講座**であり、書籍の動画版ではありません。

FW別Udemyは書籍発売に合わせて自動的に作りません。開始・途中・完成スナップショット、障害再現、デバッグ、書籍にない改修演習、テスト・正解例、独立受講可能性を満たした場合に制作判断を行います。

---

# STEP 5：Udemy講座4 ― BUILD

Udemy講座4では、

> **完成済みSDDをコードへ変換する**

ことを中心にします。

題材は、問い合わせトリアージAIエージェントです。

```text
Existing SDD
↓
Requirements
↓
Design
↓
Tasks
↓
Python
↓
OpenAI Agents SDK
↓
Test
```

ここでは新たにSDDを一から作ることより、

> **Requirements → Design → Tasks → Code**

の対応を確認しながら実装することを重視します。

---

# STEP 6：PoCとしてVALIDATEする

動くAgentができたら、次はPoCです。

ここでは、

> **動くか**

だけではなく、

> **業務で価値があるか**

を検証します。

---

# Kindle 5（PoC-Kindle）：VALIDATE Design

Kindle 5（PoC-Kindle）では、

> **既存システムをPoCへ育てるため、SDDをどう更新するか**

を扱います。

```text
Existing Implementation
↓
Current Implementation Analysis
↓
PoC Change Request
↓
Updated System Specification
↓
Updated Technology Constraints
↓
Updated Requirements
↓
Updated Design
↓
Updated Tasks
↓
Regression Test Plan
↓
PoC Evaluation Plan
```

PoC化とは、単にWeb画面を追加することではありません。

> **検証可能性をシステムへ追加すること**

だと考えています。

---

# Udemy講座5：VALIDATE Implementation

Kindle 5（PoC-Kindle）で作成したUpdated SDDに従い、

```text
FastAPI
↓
Streamlit
↓
SQLite
↓
Web PoC
↓
Evaluation
↓
Failure Analysis
↓
Go / No-Go
```

まで実際に実装します。

Kindleでは、

> **なぜそう変更するのか**

Udemyでは、

> **その変更をどう実装するのか**

を扱います。

---

# STEP 7：本番環境へOPERATEする

PoCで価値を確認できたら、本番化へ進みます。

---

# Kindle 6（Prod-Kindle）：OPERATE Design

Kindle 6（Prod-Kindle）では、

> **PoCをProductionへ育てるため、SDDをどう更新するか**

を扱います。

```text
Validated PoC
↓
Production Readiness Gap
↓
Production Change Request
↓
Updated System Specification
↓
Updated Technology Constraints
↓
Updated Requirements
↓
Updated Design
↓
Updated Tasks
↓
Regression Test Plan
↓
Production Architecture
```

本番化とは、

> **PoCをそのままAWSへ載せること**

ではありません。

必要になるのは、

- Security
- Reliability
- Authentication / Authorization
- Secret Management
- Persistent Database
- Backup / Recovery
- Logging
- Monitoring
- Failure Handling
- Version Management
- Rollback
- Operations

などです。

---

# Udemy講座6：OPERATE Implementation

Udemy講座6では、

```text
Docker
↓
PostgreSQL
↓
Terraform
↓
ECR
↓
ECS / Fargate
↓
RDS
↓
Secrets Manager
↓
CloudWatch
↓
Regression Test
↓
E2E Test
```

まで実際に構築します。

Zennで個別に学んだ技術を、

> **実際の業務システムへ統合する**

のがUdemyの役割です。

---

# 同じ問い合わせトリアージを育て続ける

Udemy講座4・5・6では、基本的に同じ問い合わせトリアージシステムを利用します。

```text
講座4
Python Agent
BUILD
        ↓
講座5
Web PoC
VALIDATE
        ↓
講座6
AWS Production
OPERATE
```

毎回別のサンプルを作らないことで、

> **1つの業務システムが成長する過程**

そのものを学べるようにします。

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

# 次の技術シリーズ：CrewAIとフレームワーク選定

OpenAI Agents SDKの実装21回を最優先で完走します。**CrewAI連載を途中へ割り込ませません。**

| 教材・連載 | 位置づけ | 公開・制作方針 |
|---|---|---|
| **CrewAI-Kindle** | 売上データ分析を題材に、Crew・Flow・Productionまで扱う実装書 | **2026年10月31日（土）発売予定、価格1,250円、ASIN：B0HNG2K4C8**。KDP予約注文（オンライン）。[Amazonで予約](https://www.amazon.co.jp/dp/B0HNG2K4C8)。Zenn21回完結を待たず、別商品として発売予定 |
| **CrewAIのZenn短期連載** | ①Agent/Task/Tool ②Role・Crew ③Process ④Flow・状態・承認 ⑤Trace・Evaluation | 21回完結後に開始する5回案。2027年1月19日（火）開始は**暫定・条件付き** |
| **CrewAI-Udemy** | 構築・失敗再現・改修を含む実践演習 | Kindleを読み上げる動画にはせず、需要と演習独自性が整った場合だけ制作 |
| **Google ADK / LangGraph** | 次の新規FW候補 | 実装検証と比較の後で、採用する側だけを教材化。発売未決定 |

**FWが人気だからという理由だけで商品を増やさない**ことを基本とします。完成コードの品質、学習成果、需要、保守負担を確認します。

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
| 売上分析Agentの第1〜16回相当の実装を体系化 | [OpenAI-SDK-Kindle（v2.0）](https://www.amazon.co.jp/dp/B0HNGDT575) | 10/29（木）発売予定・予約受付中。AgentCoreは対象外 |
| 問い合わせトリアージを構築・PoC・AWS本番化 | Kindle 4〜6／Udemy講座4〜6 | 対応する教材の公開状況を確認してリンクを追加 |
| AgentCoreでトリアージAgentを運用 | Udemy講座7 | OpenAI-SDK-Kindleとは別題材・別範囲。公開後リンク追加 |
| CrewAIの体系的実装 | [CrewAI-Kindle](https://www.amazon.co.jp/dp/B0HNG2K4C8) | 10/31（土）発売予定・予約受付中。Udemyは条件付き |
| データ分析から施策・検証計画 | [データ分析noteハブ](https://note.com/loyal_hyssop7944/n/nb88b1b78960c)／データ分析Kindle | 独立した商品ライン。実装21回とは混同しない |

※ **『ChatGPTで学ぶ問題解決型データ分析 実践入門』は2026年10月27日（火）発売予定、価格1,250円、ASIN：B0HL4PS2QCで予約受付中**です（10月20日時点でKDP「予約注文（オンライン）」／[Amazon商品ページ](https://www.amazon.co.jp/dp/B0HL4PS2QC)）。300件の受注データを使ったChatGPT中心の分析・施策立案・効果評価計画を扱います。**OpenAI-SDK-Kindle・CrewAI-Kindleとは別の商品**です。主導線は**データ分析noteハブ**とします。

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

→ **Udemy講座4：BUILD**

SDDから問い合わせトリアージAIエージェントを実装します。

---

## AIエージェントをPoC化したい

→ **Kindle 5（PoC-Kindle）：VALIDATE Design**

→ **Udemy講座5：VALIDATE Implementation**

設計を更新し、Web PoCとして実際に評価します。

---

## AIエージェントを本番化したい

### 汎用AWS Applicationとして本番化する

→ **Kindle 6（Prod-Kindle）：OPERATE Design**

→ **Udemy講座6：OPERATE Implementation**

Production要件を設計し、

```text
Docker
PostgreSQL
Terraform
AWS
```

を使って本番システムへ発展させます。

### Agent専用Managed Serviceとして本番化する

→ **Zenn 第17回〜第21回：Amazon Bedrock AgentCore**

```text
Runtime
↓
Memory
↓
Gateway
↓
Identity
↓
Observability / Evaluations
```

ECS / Fargate版とAgentCore版を、2つのProduction Routeとして比較できる構成にします。

---

# 今後の更新予定

このハブは**新しい記事・商品が公開されたときに更新する常設の総合入口**です。

1. **最優先**：OpenAI Agents SDK第1〜21回の公開状況と個別リンクを追加（火曜連載を完走）。
2. 対応する水曜noteが公開されたら、各技術のWhy記事と相互リンクする。
3. **OpenAI-SDK-Kindle（v2.0）**は10月29日（木）、**CrewAI-Kindle**は10月31日（土）の発売予定として**予約受付中**（10月20日時点でKDP「予約注文（オンライン）」）。各書籍のAmazonリンクは上記に掲載。発売後は状態を再確認する。OpenAI-SDK-KindleにAgentCoreは含めない。
4. **CrewAI-Kindle**の発売予定に合わせ、原稿・コードの検証を継続する。CrewAI Zenn5回案は21回完走後に開始条件を確認する。
5. 必要に応じて**CrewAI実装ハブ**を設け、本ハブから誘導する。Google ADK・LangGraphは採用決定前にハブを量産しない。
6. トリアージUdemy講座4〜7と対応Kindleは、公開した商品だけリンクを追加する。
7. **[AIデータ分析・課題解決noteハブ](https://note.com/loyal_hyssop7944/n/nb88b1b78960c)**は別ラインとして案内し、売上分析Kindleへの導線はそちらへ集約する。

**運用原則**：主CTAは読者の学習目的に合うリンクを1つに絞る。**予約受付中と発売中を区別**し、Amazonの商品ページの実際の表示と整合させる。既存の記事URL・slugは維持する。

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

**第1〜9回は公開済みです。** 上記各回の見出しから内容と記事URLを確認できます。

[第1回：Agent / Runner](https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data) ／ [第8回：SQLite](https://zenn.dev/tigerone1945/articles/aais-08-csv-to-sqlite) ／ [第9回：Join Validation](https://zenn.dev/tigerone1945/articles/aais-09-multitable-join-validation)

---

## Udemy

問い合わせトリアージAIエージェントを題材に、

```text
BUILD
↓
VALIDATE
↓
OPERATE
```

と、1つの業務システムを段階的に育てる実装シリーズを予定しています。

※ 公開後リンク追加