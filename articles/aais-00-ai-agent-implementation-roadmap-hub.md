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

AIエージェントを学び始めると、すぐに疑問が増えていきます。

- 最初にAgentを作ればよいのか
- Function Toolsはいつ必要になるのか
- Structured OutputやSessionsはなぜ必要なのか
- GuardrailsとHuman-in-the-Loopはどう使い分けるのか
- Multi-Agentにする判断基準は何か
- FastAPI、Docker、PostgreSQLはどの段階で必要になるのか
- AWSへDeployすれば本番化できたと言えるのか
- Amazon Bedrock AgentCoreはECS/Fargateと何が違うのか

個別の技術だけを追うと、

```text
Agent
Tools
RAG
Multi-Agent
FastAPI
Docker
AWS
AgentCore
```

と知識がばらばらになりがちです。

そこでこのハブでは、

> **業務課題から仕様を作り、AIエージェントを小さく実装し、Application化し、AWS / AgentCoreまで育てるための技術ロードマップ**

として全体を整理します。

このページは固定記事ではありません。  
公開したZenn記事・Kindle・学習資産に合わせて、リンクと位置づけを更新していきます。

---

# まず全体像

現在の学習ルートは、次の4段階で考えています。

```text
STEP 1
業務課題と仕様を整理する
↓
SDD / Case Study / System Specification

STEP 2
AIエージェントを小さく実装する
↓
Zenn AIエージェント実装実践 第1〜21回

STEP 3
本番運用の2ルートを比較する
↓
ECS / Fargate
または
Amazon Bedrock AgentCore

STEP 4
次の技術テーマへ進む
↓
Python-MIPによる数理最適化
```

ここで重要なのは、

> **設計・技術学習・業務PoC・本番運用を、同じものとして扱わない**

ことです。

---

# Zenn・note・Kindle・Udemyの役割

現在は、各媒体を次のように分けています。

| 媒体 | 役割 | 主に扱うこと |
|---|---|---|
| **Zenn** | 技術資産・How | 実装、コード、検証、技術選定 |
| **note** | 集客・Why / What | 業務課題、判断、設計、導入 |
| **Kindle** | 体系化 | 一つのテーマを順序立てて学ぶ |
| **Udemy** | ケース実践 | 実務システムを手を動かして作る |

Zennは「集客量」を主目的にせず、

> **技術を深く確認したい人が戻ってこられる資産**

として使います。

---

# STEP 1：業務課題から仕様へ

AIエージェント開発でも、最初からAgentを書くのではなく、

> **何を作るのか、なぜ作るのか、どこまでAIへ任せるのか**

を整理することが重要です。

考え方は、

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

この土台は、次のKindleで体系化しています。

## 『AI時代の仕様駆動開発』

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

という、AI時代の仕様駆動開発を扱います。

## 『AIエージェント開発のためのケーススタディ設計』

https://www.amazon.co.jp/dp/B0HJMM4JF7

```text
Business Problem
↓
As-Is / To-Be
↓
Case Study
↓
System Specification
↓
Kiro Spec
```

という、業務課題を仕様へ変換する流れを扱います。

## 『業務課題から学ぶ AIエージェント実践入門』

https://www.amazon.co.jp/dp/B0HJSTH8BP

業務課題からAgent Design、OpenAI Agents SDK、Evaluationへ進む考え方を体系化しています。

---

# STEP 2：AIエージェント実装実践シリーズ

ここからがZennの中心です。

一つの小さなサンプルを、

```text
Agent
↓
Function Tools
↓
Structured Output
↓
Streamlit
↓
Sessions / Tracing
↓
Guardrails
↓
HITL
↓
SQLite
↓
Multi-table
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
Monitoring / Evaluation
↓
AgentCore
```

と段階的に育てます。

最初から完成システムは作りません。

> **必要になった機能を、その理由と一緒に一つずつ追加する**

ことを重視します。

---

# AIエージェント実装実践シリーズ 全21回

| 回 | テーマ | 主な技術 |
|---:|---|---|
| 第1回 | 最小AIエージェントを作る | Agent / Runner |
| 第2回 | 処理をToolへ分離する | Function Tools |
| 第3回 | 出力を構造化する | Structured Output / Pydantic |
| 第4回 | Web画面を作る | Streamlit |
| 第5回 | 会話と実行を追跡する | Sessions / Tracing |
| 第6回 | 危険な処理を止める | Guardrails |
| 第7回 | 人の判断を入れる | HITL |
| 第8回 | 業務データを永続化する | SQLite |
| 第9回 | 複数Tableを安全に扱う | Multi-table / Join Validation |
| 第10回 | Agentを役割分担する | Multi-Agent |
| 第11回 | AIエージェントをAPI化する | FastAPI |
| 第12回 | 実行環境をContainer化する | Docker / Docker Compose |
| 第13回 | 本番DBへ移行する | PostgreSQL |
| 第14回 | AWS基盤をコード化する | Terraform |
| 第15回 | AWSへDeployする | ECR / ECS / Fargate / RDS |
| 第16回 | 本番環境を監視・評価する | CloudWatch / Secrets / Evaluation |
| 第17回 | Agent専用Runtimeへ載せる | AgentCore Runtime |
| 第18回 | 記憶を管理する | AgentCore Memory |
| 第19回 | Tool接続を外へ出す | AgentCore Gateway |
| 第20回 | 誰の権限で呼ぶかを設計する | AgentCore Identity |
| 第21回 | 監視・評価を改善Loopへ戻す | AgentCore Observability / Evaluations |

---

# Phase 1：Single AgentをApplicationへ近づける

第1〜9回です。

```text
Agent
↓
Tools
↓
Structured Output
↓
UI
↓
Session
↓
Guardrails
↓
HITL
↓
Database
```

ここで重要なのは、

> **LLMへすべて任せない**

ことです。

```text
LLM
＝Reasoning / Interpretation

Code
＝Calculation / Rule / Constraint
```

と分けます。

売上計算や閾値判定、Database Constraintのように決定的に処理できるものは、Codeへ寄せます。

---

# Phase 2：Multi-Agentは責務が分かれたときだけ使う

第10回です。

```text
Manager Agent
├─ Specialist Agent A
├─ Specialist Agent B
└─ Specialist Agent C
```

Multi-Agentの目的は、

> **Agentを増やすこと**

ではありません。

> **責務を分離すること**

です。

Single Agentで十分なら、無理に分けません。

---

# Phase 3：AgentをApplicationへ組み込む

第11〜13回です。

```text
Streamlit
↓
FastAPI
↓
Agent
↓
Repository
↓
PostgreSQL
```

ここで、

```text
Agent Design
≠
Application Design
```

がはっきりしてきます。

FastAPIはUIとAI処理を分け、Dockerは実行環境を固定し、PostgreSQLは共有Data Layerを持つために追加します。

---

# Phase 4：汎用AWS Applicationとして本番化する

第14〜16回です。

```text
Terraform
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
↓
CloudWatch
```

第14回ではInfrastructure as Code、第15回ではAWS Deploy、第16回ではSecrets・Monitoring・Tracing・Evaluationを扱います。

重要なのは、

> **Deployできたことと、運用できることは別**

という点です。

```text
Deploy
↓
Secrets
↓
Logs / Metrics / Tracing
↓
Evaluation
↓
Backup / Rollback
```

まで考えて、Production Readinessへ進みます。

---

# Phase 5：AgentCoreでAgent Nativeな本番ルートを学ぶ

第17〜21回です。

第16回までで、ECS / Fargateを使った汎用AWS Applicationとしての本番ルートを作ります。

そのうえで、第17回からは、

> **Agent専用Managed Serviceを使う別のProduction Route**

を学びます。

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

## 第17回：Runtime

```text
OpenAI Agents SDK
↓
BedrockAgentCoreApp
↓
AgentCore Runtime
```

既存Agent本体を書き直すのではなく、Runtime Adapterを追加します。

ここで比較するのは、

```text
汎用Application Runtime
vs
Agent Native Runtime
```

です。

## 第18回：Memory

```text
Session
＝今の会話

Long-term Memory
＝次回も使うKnowledge

Business Fact
＝Database
```

と責務を分けます。

何を覚えるかだけでなく、何を保存しないかも設計します。

## 第19回：Gateway

```text
Agent
↓
Gateway
↓
Tool / API
```

外部Service、Side Effect、共有Tool、認証が必要なToolをAgent Codeから分離します。

Local ToolをすべてGateway化するわけではありません。

## 第20回：Identity

```text
User Identity
Agent Workload Identity
External Service Credential
```

を分けます。

AuthenticationとAuthorizationをPromptではなくSystem Designとして扱います。

## 第21回：Observability / Evaluations

```text
Production
↓
Failure Detection
↓
Trace Analysis
↓
Evaluation
↓
Failure Case
↓
Fix
↓
Regression Evaluation
↓
Deploy
```

という改善Loopを作ります。

見つけた失敗をEvaluation Caseへ戻すことが、継続改善の中心です。

---

# ECS/FargateとAgentCoreは「どちらが上か」ではない

このシリーズでは、二つを比較できるようにします。

| 観点 | ECS / Fargate | AgentCore |
|---|---|---|
| 中心 | 汎用Application | Agent Runtime |
| 実行単位 | Container / Task | Agent |
| Network / Runtime | 自分で設計 | Agent向けManaged |
| Memory / Gateway / Identity | 自分で組み合わせる | Agent向けCapability |
| 向くケース | 既存AWS Application標準がある | Agent中心のManaged構成を使いたい |

判断基準は、

> **何を自分で設計し、何をManaged Serviceへ任せるか**

です。

---

# 第1〜21回を通して学びたいこと

重要なのは技術名ではありません。

```text
責務を分ける
境界を作る
危険な処理を制御する
状態を管理する
実行を追跡する
評価できるようにする
```

ことです。

シリーズ全体の中心メッセージは、

> **AIエージェントを業務システムにするとは、LLMを呼び出すことではなく、LLMをシステムの中に安全に配置すること**

です。

---

# STEP 3：業務ケース実践はFDEシリーズへ分ける

以前は、問い合わせトリアージを題材に、

```text
BUILD
↓
VALIDATE
↓
OPERATE
```

とUdemy講座4〜6へ展開する計画を置いていました。

この計画は現在の方針では採用していません。

今後の業務ケース実践は、**FDE（Forward Deployed Engineer）型のケーススタディ**として再編します。

主な題材は、

- 請求書処理（支払前確認）
- 問い合わせトリアージ

です。

FDE-Udemyでは、

```text
Business Problem
↓
As-Is
↓
To-Be
↓
System Specification
↓
SDD
↓
Agent Implementation
↓
Test
↓
Local PoC
```

までを扱います。

技術構成は、

```text
Streamlit
＋
FastAPI
＋
SQLite
```

を基本とし、**Docker化・AWS Deploy・PostgreSQL本番運用はFDE-Udemyには含めません。**

クラウド運用は、Zenn第14〜21回や別の技術学習ルートで扱います。

> **技術を学ぶZennと、業務ケースを実装するFDEを分ける**

ことで、目的を明確にします。

---

# STEP 4：第21回の次はPython-MIPを優先する

AIエージェント実装実践シリーズ第1〜21回を完走したあと、新しい技術連載として優先する候補は、

> **Python-MIPによるビジネス数理最適化**

です。

CrewAIの新規Zenn連載を先に割り込ませるのではなく、既に原稿・検証資産があるPython-MIPを先に公開する方針です。

題材はホテル予約データです。

```text
データ分析
↓
課題発見
↓
原因仮説
↓
改善施策
↓
数理モデル
↓
最適化
↓
意思決定
```

へ進みます。

全6回の原稿は作成済みで、GitHubの

`python_mip/`

を正本として管理しています。

2027年の公開候補ですが、**公開開始日は現時点で固定しません。**

既存のZenn第1〜21回を完走し、投稿枠との整合を確認してから開始します。

---

# ホテル予約データ分析とPython-MIPの関係

ホテル予約データ分析は、

> **何が起きているか、なぜ起きている可能性があるか、何を改善するか**

を考える学習です。

その先のPython-MIPでは、

> **限られた予算・客室数・キャンセル率などの制約の中で、どの施策へどれだけ資源を配分するか**

を扱います。

つまり、

```text
Data Analysis
＝What / Why

Mathematical Optimization
＝Act / Decide under Constraints
```

という関係です。

ホテル予約データ分析Kindleは、2026年11月15日に販売開始しています。

**『ChatGPTで学ぶホテル予約データ分析と業務課題解決：キャンセル率から改善施策・効果検証まで』**

https://www.amazon.co.jp/dp/B0HQYNZ8JF

ただし、このZennハブの主目的は商品紹介ではなく、**技術学習ルートを整理すること**です。

---

# CrewAIはどうするか

CrewAIそのものをやめるわけではありません。

既存のCrewAI Kindleや学習資産は維持します。

ただし、新しいZenn技術連載の優先順位は、

```text
OpenAI Agents SDK 第1〜21回を完走
↓
Python-MIP
↓
CrewAI等の次Frameworkを再評価
```

とします。

Frameworkは流行だけで選ばず、

- 学習価値
- 既存資産
- 読者需要
- 実務ケースへの接続
- 制作負荷

を見て判断します。

---

# AgentCoreは「商品化を急ぐ」より先に検証する

AgentCoreは第17〜21回で学びます。

さらに2027年3月までは、

> **実機学習・小規模検証を優先**

します。

Runtime、Memory、Gateway、Identity、Observability / Evaluationsについて、実際に使える範囲と既存AWS構成との差を確認します。

この検証結果を見て、

- AgentCore特化Udemy
- Cloud Deployment講座
- FDEケースのCloud版

などをGo / Hold / No-Goで判断します。

最初から商品化を前提にしません。

---

# 現在の学習・制作ロードマップ

```text
2026
OpenAI Agents SDK 第1〜21回を継続
↓
ECS/Fargate本番ルート
＋
AgentCore本番ルート

並行
↓
FDEケース
請求書処理 / トリアージ
Local PoC

2027
↓
Python-MIP Zenn全6回を優先公開
↓
AgentCore実機検証を継続
↓
CrewAI / 次Framework / Cloud講座を再評価
```

---

# このハブの使い方

## AIエージェントの設計から学びたい

### Kindle
**『業務課題から学ぶ AIエージェント実践入門』**

https://www.amazon.co.jp/dp/B0HJSTH8BP

## SDDそのものを学びたい

### Kindle
**『AI時代の仕様駆動開発』**

https://www.amazon.co.jp/dp/B0HBQ32KCQ

## 業務課題を仕様へ変えたい

### Kindle
**『AIエージェント開発のためのケーススタディ設計』**

https://www.amazon.co.jp/dp/B0HJMM4JF7

## OpenAI Agents SDKを実装しながら学びたい

→ **このZenn実装実践シリーズ 第1〜21回**

第1回：

https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data

## 業務ケースを実装したい

→ **FDEケーススタディ**

請求書処理・問い合わせトリアージを、Streamlit + FastAPI + SQLiteのローカルPoCまで実装します。

## 数理最適化へ進みたい

→ **Python-MIPシリーズ**

ホテル予約データを題材に、分析結果を制約付き意思決定へつなげます。2027年公開候補です。

---

# 今後の更新

このハブでは今後、次を更新します。

- AIエージェント実装実践 第1〜21回の公開状況とリンク
- 第17〜21回 AgentCoreの実機検証結果
- Python-MIP全6回の公開開始日とリンク
- FDEケーススタディの公開状況
- 次Frameworkを採用する場合の判断結果

公開済み事実と計画は分けて記載します。

---

# まとめ

現在のロードマップは、

```text
Business Problem
↓
System Specification / SDD
↓
Agent Design
↓
OpenAI Agents SDK
↓
Tools / Safety / HITL
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
Monitoring / Evaluation
↓
AgentCore
```

と進みます。

ここまでが、

> **AIエージェントを業務Applicationとして育てる技術ルート**

です。

その後は、

```text
業務ケース実践
→ FDE

分析結果から意思決定へ
→ Python-MIP

Agent Native Cloud
→ AgentCore実機検証
```

と、目的別に分岐します。

大切なのは、技術を増やすことではありません。

> **何を解決したいのかに応じて、必要な設計・技術・運用を選ぶこと。**

このハブを、その判断に戻るための技術ロードマップとして更新していきます。
