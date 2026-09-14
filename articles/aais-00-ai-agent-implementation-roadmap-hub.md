---
title: "AIエージェント実践ロードマップ──SDD・設計・実装・PoC・AWS本番まで"
emoji: "🧭"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - SDD
  - ClaudeCode
  - AWS
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
```

と知識がバラバラになりがちです。

そこでこの記事では、これまで公開してきたKindle・Zennシリーズと、これから公開するZenn実装シリーズ・Kindle・Udemyを、

> **AIエージェントを仕様から設計し、実装し、PoCを経てAWS本番環境へ育てるための学習ロードマップ**

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
2026年9月末公開予定

        ↓

STEP 4
AIエージェント技術を小さく実装する
↓
Zenn
AIエージェント実装実践シリーズ
第1回〜第16回

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
Kindle③
VALIDATE Design

＋

Udemy講座5
VALIDATE Implementation

        ↓

STEP 7
AWS本番環境へ育てる
↓
Kindle④
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

# Zenn・Kindle・Udemyの役割

私は、それぞれの媒体を次のように使い分けています。

| 媒体 | 役割 | 主に扱うこと |
|---|---|---|
| Zenn | 小さく理解する | 1記事1テーマの設計・実装 |
| Kindle | 体系化する | なぜその設計にするのか |
| Udemy | 実際に作る | 一連の業務システム実装 |

同じFastAPIを扱う場合でも、目的は違います。

Zennでは、

```text
Agent
↓
FastAPI
↓
APIとして呼び出す
```

ところを小さく実装します。

Kindleでは、

```text
なぜFrontendとAgentを分離するのか
なぜAPI Boundaryが必要なのか
```

を設計として考えます。

Udemyでは、

```text
FastAPI
↓
Endpoint
↓
Agent
↓
Error Handling
↓
Test
↓
Web UI
↓
Database
```

まで一連のApplicationとして実装します。

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

# AIエージェントKindleも公開予定

ZennのAIエージェント設計シリーズで扱ってきた内容を、さらに体系化したKindleも公開予定です。

## AIエージェント実践入門

**2026年9月末公開予定**

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

を体系的に扱う予定です。

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
| AIエージェント実践入門 | SDDをAIエージェント設計へ適用する | 2026年9月末公開予定 |

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
```

と、一つずつ機能を追加していきます。

---

# AIエージェント実装実践シリーズ 全16回

現時点では、次の構成を予定しています。

| 回 | テーマ | 主な技術 |
|---:|---|---|
| 第1回 | 最小AIエージェントを作る | Agent / Runner |
| 第2回 | 処理をToolへ分離する | Function Tools |
| 第3回 | 出力を構造化する | Structured Output / Pydantic |
| 第4回 | Web画面を作る | Streamlit |
| 第5回 | 会話と実行を追跡する | Sessions / Tracing |
| 第6回 | 危険な処理を止める | Guardrails |
| 第7回 | 人の判断を入れる | HITL |
| 第8回 | データを永続化する | SQLite |
| 第9回 | 複数Tableを安全に扱う | Join Validation |
| 第10回 | Agentを役割分担する | Multi-Agent |
| 第11回 | AIエージェントをAPI化する | FastAPI |
| 第12回 | 実行環境をContainer化する | Docker / Docker Compose |
| 第13回 | 本番DBへ移行する | PostgreSQL |
| 第14回 | AWS基盤をコード化する | Terraform |
| 第15回 | AWSへデプロイする | ECR / ECS / Fargate / RDS |
| 第16回 | 本番環境を監視する | CloudWatch / Secrets / Evaluation |

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

最後に、

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

最終的なイメージは、

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

---

# 実装シリーズは4つのPhaseで進む

16回を大きく分けると、次の4段階になります。

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

最後に、

> **ローカルで動くAgentをCloud上で運用できる構成へ育てる**

ところまで進みます。

---

# KindleとZenn実装シリーズの違い

KindleとZenn実装シリーズでは、主目的が違います。

## Kindle

```text
Business Problem
↓
System Specification
↓
Technology Constraints
↓
SDD
↓
Agent Design
```

> **何を、なぜ作るのか**

を中心に扱います。

## Zenn実装シリーズ

```text
Agent
↓
Tools
↓
Safety
↓
Database
↓
API
↓
Docker
↓
AWS
```

> **技術を一つずつどう実装するのか**

を中心に扱います。

---

# Zenn実装シリーズとUdemyの違い

ZennでAWSまで扱っても、Udemyとは役割が異なります。

Zennでは、

```text
技術を理解する
↓
小さく実装する
↓
動作を確認する
```

ことを目的とします。

Udemyでは、問い合わせトリアージAIエージェントという1つの業務システムを、

```text
BUILD
↓
VALIDATE
↓
OPERATE
```

まで育てます。

つまり、

```text
Zenn
＝技術を段階的に理解する

Kindle
＝設計を体系的に理解する

Udemy
＝業務システムとして完成させる
```

という役割分担です。

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

# Kindle③：VALIDATE Design

Kindle③では、

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

Kindle③で作成したUpdated SDDに従い、

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

# Kindle④：OPERATE Design

Kindle④では、

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

**2026年9月末公開予定**

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

第1回から第16回まで、

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
```

と段階的に進みます。

---

## 業務AIエージェントを実際に作りたい

→ **Udemy講座4：BUILD**

SDDから問い合わせトリアージAIエージェントを実装します。

---

## AIエージェントをPoC化したい

→ **Kindle③：VALIDATE Design**

→ **Udemy講座5：VALIDATE Implementation**

設計を更新し、Web PoCとして実際に評価します。

---

## AIエージェントをAWSで本番化したい

→ **Kindle④：OPERATE Design**

→ **Udemy講座6：OPERATE Implementation**

Production要件を設計し、

```text
Docker
PostgreSQL
Terraform
AWS
```

を使って本番システムへ発展させます。

---

# 今後の更新予定

このハブ記事は随時更新します。

今後追加予定：

- AIエージェント実践入門 Kindle
- AIエージェント実装実践 第1回〜第16回
- Udemy講座4 BUILD
- Kindle③ VALIDATE Design
- Udemy講座5 VALIDATE Implementation
- Kindle④ OPERATE Design
- Udemy講座6 OPERATE Implementation

公開した記事・教材は、このページからアクセスできるように順次リンクを追加していきます。

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

次回からは、新しい

# AIエージェント実装実践シリーズ

を開始します。

第1回は、

> **最小構成から始めるAIエージェント実装**

です。

まずは小さなAgentを1つ動かすところから始め、最終的には第16回でAWS上の運用までつなげていきます。

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

**2026年9月末公開予定**

業務課題・SDD・Agent Design・OpenAI Agents SDK・Evaluationまでを体系的に扱います。

---

## AIエージェント実装実践シリーズ

### 第1回
**最小構成から始めるAIエージェント実装**

※ 公開後リンク追加

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
