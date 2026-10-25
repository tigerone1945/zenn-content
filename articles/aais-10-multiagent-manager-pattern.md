---
title: "第10回 Agentを役割分担する──Manager PatternでMulti-Agent化する"
emoji: "👥"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - MultiAgent
  - Python
  - SQLite
published: true
published_at: "2026-10-27 07:00"
---

# はじめに

第9回では、売上データを、

```text
orders
customers
products
```

の3 Tableへ分けました。

さらに、

```text
list
↓
inspect
↓
suggest join
↓
validate
↓
preview
↓
aggregate
```

というJoin Validationを追加しました。

ここまでのAgentは、1つのAgentが、

```text
質問理解
Tool選択
Data確認
集計
結果説明
```

をすべて担当しています。

小規模なうちは問題ありません。

しかし機能が増えると、

> **1つのAgentの責務が広すぎる**

状態になります。

そこで第10回では、

> **Agent自体を役割分担する**

ところまで進めます。

---

# 今回のゴール

今回作る構成は、

```text
Manager Agent
├─ Data Quality Agent
├─ Sales Analyst Agent
└─ Business Insight Agent
```

です。

Manager Agentがユーザーとの会話を担当し、必要に応じて専門Agentを呼び出します。

---

# Multi-Agentにする理由

1つのAgentへ、

```text
データ品質も見て
売上分析もして
示唆も考えて
DBも理解して
```

と詰め込むことはできます。

しかし、Instructionsが大きくなり、

```text
何を担当するAgentなのか
```

が曖昧になります。

役割分担すると、

```text
Data Quality Agent
＝データ品質

Sales Analyst Agent
＝数値分析

Business Insight Agent
＝業務的な意味づけ
```

と責務を明確にできます。

---

# Agents as ToolsとHandoffs

OpenAI Agents SDKには、代表的に2つのMulti-Agentパターンがあります。

```text
Agents as Tools
Handoffs
```

違いは重要です。

## Agents as Tools

```text
User
↓
Manager Agent
↓
Specialist Agent
↓
Manager Agent
↓
User
```

Managerが会話の主導権を持ち続けます。

## Handoffs

```text
User
↓
Triage Agent
↓
Specialist Agent
↓
User
```

途中で会話の主担当そのものがSpecialistへ切り替わります。

---

# 今回はManager Patternを採用する

売上分析Applicationでは、

```text
複数の専門Agentの結果をまとめる
```

ことがあります。

そのため今回は、

> **Agents as Tools**

を使います。

Managerが最終回答をまとめる構成です。

---

# Data Quality Agent

最初の専門Agentです。

```python
from agents import Agent


data_quality_agent = Agent(
    name="Data Quality Agent",
    instructions="""
あなたは売上データの品質確認を担当します。

確認対象:
- 欠損
- 重複
- 不正値
- Join Key
- Table間整合性

ユーザー向けの最終回答は作らず、
Manager Agentが利用しやすい形で
簡潔に結果を返してください。
""",
    tools=[
        list_tables,
        inspect_table,
        validate_join,
        preview_join,
    ],
)
```

---

# Sales Analyst Agent

```python
sales_analyst_agent = Agent(
    name="Sales Analyst Agent",
    instructions="""
あなたは売上データの数値分析を担当します。

利用可能なToolを使って、
実データに基づく集計結果だけを返してください。

数値を自分で推測しないでください。
""",
    tools=[
        get_sales_summary,
        get_sales_by_customer_type,
        get_sales_by_product_category,
        get_sales_by_region,
    ],
)
```

---

# Business Insight Agent

このAgentは、集計結果から業務上の意味を整理します。

```python
business_insight_agent = Agent(
    name="Business Insight Agent",
    instructions="""
あなたは売上分析結果を
業務上の示唆へ変換する担当です。

入力された分析結果から、
- 重要な発見
- 注意点
- 次に確認すべきこと

を簡潔に整理してください。

数値を新しく作らないでください。
入力にない事実を断定しないでください。
""",
)
```

---

# Specialist AgentをTool化する

OpenAI Agents SDKでは、

```python
Agent.as_tool()
```

を使って、Agentを別のAgentから呼べるToolにできます。

```python
data_quality_tool = (
    data_quality_agent.as_tool(
        tool_name="check_data_quality",
        tool_description=(
            "売上データの品質や"
            "Join整合性を確認する"
        ),
    )
)
```

Sales Analystも同様です。

```python
sales_analysis_tool = (
    sales_analyst_agent.as_tool(
        tool_name="analyze_sales_data",
        tool_description=(
            "SQLiteの売上データを"
            "集計・分析する"
        ),
    )
)
```

Business Insight AgentもTool化します。

```python
business_insight_tool = (
    business_insight_agent.as_tool(
        tool_name="create_business_insight",
        tool_description=(
            "分析結果から"
            "業務上の示唆を整理する"
        ),
    )
)
```

---

# Manager Agentを作る

```python
manager_agent = Agent(
    name="Sales Analysis Manager",
    instructions="""
あなたは売上分析チームのManagerです。

ユーザーの質問を理解し、
必要に応じて専門AgentをToolとして呼んでください。

役割:
- Data Quality Agent
  データ品質やJoin整合性

- Sales Analyst Agent
  実データの集計・分析

- Business Insight Agent
  分析結果から示唆を整理

Manager自身で売上数値を作らないでください。

必要な専門Agentの結果を統合し、
最後はあなたがユーザー向けの回答を作ってください。
""",
    tools=[
        data_quality_tool,
        sales_analysis_tool,
        business_insight_tool,
    ],
)
```

---

# Architecture

```text
                User
                  │
                  ▼
        ┌──────────────────┐
        │  Manager Agent   │
        └───────┬──────────┘
                │
        ┌───────┼───────────┐
        │       │           │
        ▼       ▼           ▼
┌──────────┐ ┌──────────┐ ┌────────────┐
│Data      │ │Sales     │ │Business    │
│Quality   │ │Analyst   │ │Insight     │
│Agent     │ │Agent     │ │Agent       │
└──────────┘ └──────────┘ └────────────┘
        │       │           │
        └───────┴───────────┘
                │
                ▼
        ┌──────────────────┐
        │  Manager Agent   │
        └───────┬──────────┘
                │
                ▼
               User
```

---

# Managerが会話を持つ

`as_tool()`の重要なポイントは、

> **Specialist Agentへ会話そのものを渡して主担当を交代するわけではない**

ことです。

Managerは、

```text
必要なSubtaskだけをAgent Toolへ渡す
↓
結果を受け取る
↓
最終回答を作る
```

という動きをします。

---

# 例：カテゴリ別売上を分析する

ユーザーが、

```text
商品カテゴリ別の売上を分析して、
業務上の示唆も教えてください。
```

と質問した場合、Managerは、

```text
Sales Analyst Agent
↓
カテゴリ別売上

Business Insight Agent
↓
分析結果の意味づけ

Manager
↓
統合して回答
```

という流れにできます。

---

# 例：データ品質も確認する

ユーザーが、

```text
顧客種別別売上を出す前に、
Joinが安全か確認してください。
```

と聞いた場合、

```text
Data Quality Agent
↓
Join Validation

Sales Analyst Agent
↓
集計

Manager
↓
結果を統合
```

とできます。

---

# Handoffではなくas_tool()を使う理由

今回の目的は、

```text
専門家へ相談する
```

ことであって、

```text
会話の担当者を交代する
```

ことではありません。

そのため、

```text
Agent.as_tool()
```

の方が合っています。

---

# Handoffが向くケース

一方、たとえば問い合わせ窓口で、

```text
請求
返金
技術サポート
```

のように、

> **以降の会話を専門Agentへ任せたい**

場合はHandoffが向いています。

---

# Multi-Agentにすればよいわけではない

Agentを分けすぎると、

```text
Model Call増加
Latency増加
Cost増加
Traceが複雑
```

になります。

そのため、

> **責務が明確に分かれる場合だけAgentを分ける**

のが基本です。

---

# Toolで十分なものはToolのままにする

たとえば、

```text
get_sales_by_region
```

はAgentにする必要はありません。

これは決定的な処理なので、

```text
Function Tool
```

で十分です。

整理すると、

```text
決定的な処理
→ Function Tool

判断・分析・説明を伴う専門処理
→ Specialist Agent
```

という分け方が使えます。

---

# Sessions / Tracingとの関係

第5回で導入したSessionsとTracingは、Multi-Agentでもそのまま重要です。

Tracingを見ると、

```text
Manager
↓
Sales Analyst Agent
↓
Function Tool
↓
Manager
↓
Business Insight Agent
```

といった実行経路が分かります。

Multi-Agentになるほど、

> **Tracingの価値が上がる**

と考えると分かりやすいです。

---

# 第10回で理解しておきたい5つのこと

```text
1. Specialist Agent
責務ごとにAgentを分ける

2. Agent.as_tool()
AgentをToolとして呼ぶ

3. Manager Pattern
Managerが会話を保持する

4. Handoffとの違い
会話の主担当を移すかどうか

5. Cost / Complexity
分けすぎない
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Manager Pattern | ○ | 第10回 |
| Agents as Tools | ○ | 第10回 |
| Handoffs | △ | 比較のみ |
| FastAPI | × | 第11回 |
| Docker | × | 第12回 |
| PostgreSQL | × | 第13回 |

---

# まとめ

今回は、売上分析AgentをMulti-Agent化しました。

構成は、

```text
Manager Agent
├─ Data Quality Agent
├─ Sales Analyst Agent
└─ Business Insight Agent
```

です。

中心メッセージは、

> **Agentを増やすことが目的ではなく、責務を分けることが目的**

です。

今回のApplicationでは、Managerが最終回答を持つため、

```text
Agents as Tools
```

を採用しました。

---

# 次回予告

## 第11回：AIエージェントをFastAPIでAPI化する

次回は、

```text
Streamlit
↓
HTTP
↓
FastAPI
↓
Manager Agent
```

という境界を作ります。

これまで同一Process内で呼んでいたAgentを、

> **Web APIとして切り離す**

ところまで進めます。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

OpenAI Agents SDK Multi-Agent：

https://openai.github.io/openai-agents-python/multi_agent/

OpenAI Agents SDK Tools：

https://openai.github.io/openai-agents-python/tools/

公式ドキュメントでは、`Agent.as_tool()`を使うManager Patternと、HandoffsによるConversation Delegationが主要なMulti-Agentパターンとして説明されています。
