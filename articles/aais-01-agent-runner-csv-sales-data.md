---
title: "第1回 最小構成から始めるAIエージェント実装──CSV売上データを題材にAgentとRunnerを理解する"
emoji: "🤖"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - Python
  - CSV
  - AI開発
published: true
published_at: "2026-09-18 07:00"
---

# はじめに

今回から、**AIエージェント実装実践シリーズ**を始めます。

このシリーズでは、最初から複雑なシステムを作りません。

1つの小さなAIエージェントを出発点にして、

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
Human-in-the-Loop
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
Monitoring
```

と、少しずつ育てていきます。

そして今後は、同じ題材を使い続けます。

今回から使うのは、

> **売上・受注データをまとめたCSV**

です。

ファイル名は、

```text
ai_agent_practice_sales.csv
```

とします。

このCSVを、第1回では「Agentが扱う題材」として使い、第2回以降でFunction Tools、Structured Output、Streamlit、SQLite、Multi-Agentへ発展させます。

---

# 今回のゴール

第1回では、まだCSVをAgent自身に直接操作させません。

今回理解したいのは、

```text
Agent
Runner
```

というOpenAI Agents SDKの最小構成です。

構成は次の通りです。

```text
CSV
↓
PythonでDataset Profileを作る
↓
Runner
↓
Agent
↓
分析観点を提案
```

つまり、

> **CSVそのものをLLMへ丸投げするのではなく、Python側で簡単な情報を整理してAgentへ渡す**

ところから始めます。

---

# 使用するCSV

このシリーズでは、次のCSVを使います。

```text
ai_agent_practice_sales.csv
```

主な列は次の通りです。

| 列名 | 内容 |
|---|---|
| `order_id` | 注文ID |
| `order_date` | 注文日 |
| `customer_id` | 顧客ID |
| `customer_name` | 顧客名 |
| `customer_type` | 法人 / 個人 |
| `product_id` | 商品ID |
| `product_name` | 商品名 |
| `category` | 商品カテゴリ |
| `unit_price` | 単価 |
| `quantity` | 数量 |
| `discount_rate` | 割引率 |
| `subtotal` | 割引前金額 |
| `total_amount` | 注文金額 |
| `sales_amount` | 実売上金額 |
| `region` | 地域 |
| `salesperson` | 担当者 |
| `status` | 注文状態 |
| `review_flag` | 要レビュー判定 |

このCSVには、今後の実装で使いやすいように、

```text
売上
カテゴリ
地域
担当者
キャンセル
要レビュー
```

などの情報が入っています。

---

# なぜ最初からCSVを使うのか

AIエージェントの学習では、毎回別のサンプルを作ると、

```text
第1回
別の題材

第2回
別の題材

第3回
また別の題材
```

となり、技術の成長が見えにくくなります。

そこでこのシリーズでは、同じCSVを使い続けます。

```text
CSV
↓
Agent
↓
Tools
↓
Web UI
↓
DB
↓
Multi-Agent
↓
API
↓
AWS
```

と、1つのシステムが育っていく流れを追います。

---

# 第1回では何をしないのか

今回は、まだ次のことは行いません。

- pandasによる売上集計
- Function Tools
- Structured Output
- Streamlit
- SQLite
- Multi-Agent

第1回では、

> **AgentとRunnerの役割を理解すること**

に集中します。

---

# 開発環境

今回は以下を想定します。

```text
macOS
VS Code
Python
uv
OpenAI API Key
```

プロジェクトを作ります。

```bash
mkdir ai-agent-practice
cd ai-agent-practice
```

`uv`で初期化します。

```bash
uv init
```

必要なライブラリを追加します。

```bash
uv add openai-agents
uv add python-dotenv
uv add pandas
```

---

# フォルダー構成

今回は次のようにします。

```text
ai-agent-practice/
├── data/
│   └── ai_agent_practice_sales.csv
├── .env
├── .gitignore
├── main.py
├── pyproject.toml
└── uv.lock
```

CSVは`data`フォルダーへ配置します。

---

# API Keyを設定する

`.env`を作成します。

```env
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
```

`.gitignore`には、

```gitignore
.env
.venv/
__pycache__/
```

を追加します。

API KeyはGitHubへ公開しないようにします。

---

# まずPythonでCSVを確認する

第1回では、CSV全体をAgentへ渡しません。

Python側で、

```text
件数
列名
期間
```

だけを取得して、Dataset ProfileとしてAgentへ渡します。

`main.py`を作成します。

```python
from pathlib import Path

import pandas as pd
from dotenv import load_dotenv
from agents import Agent, Runner

load_dotenv()

CSV_PATH = Path("data/ai_agent_practice_sales.csv")

df = pd.read_csv(CSV_PATH)

dataset_profile = f"""
このCSVは売上・受注データです。

レコード数:
{len(df)}

列名:
{", ".join(df.columns)}

注文日の最小値:
{df["order_date"].min()}

注文日の最大値:
{df["order_date"].max()}
"""

agent = Agent(
    name="売上分析アシスタント",
    instructions="""
あなたは売上データ分析を支援するAIアシスタントです。

与えられたDataset Profileを読み、
最初に確認すべき分析観点を3つ提案してください。

まだ実データの数値集計は行わず、
どの観点を確認すべきかだけを説明してください。
""",
)

prompt = f"""
以下が分析対象データの概要です。

{dataset_profile}

このデータを分析するとしたら、
最初に確認すべき観点を3つ提案してください。
"""

result = Runner.run_sync(
    agent,
    prompt,
)

print(result.final_output)
```

---

# 何をしているのか

今回の処理は次の流れです。

```text
CSV
↓
pandasで読み込む
↓
Dataset Profileを作る
↓
Runner
↓
Agent
↓
分析観点を返す
```

まだAgentはCSVを直接集計していません。

ここが大事です。

---

# Dataset Profileとは

今回は次のような情報だけをAgentへ渡します。

```text
レコード数
列名
期間
```

たとえば、

```text
このCSVは売上・受注データです。

レコード数:
300

列名:
order_id, order_date, customer_id, ...
```

のような情報です。

Agentはこの情報から、

```text
カテゴリ別売上
地域別売上
注文状態
```

などの分析観点を提案できます。

---

# なぜCSV全体をそのまま渡さないのか

300行のCSVを文字列にしてLLMへ渡すこともできます。

しかし、今回の目的は、

> **Agent / Runnerの最小構造を理解すること**

です。

また、今後のシリーズでは、

```text
LLM
＝分析意図を理解する

Python / pandas
＝集計・計算する
```

という役割分担を採用します。

そのため第1回では、あえてCSVの概要だけを渡します。

---

# Agentとは何か

今回のAgentは、

```python
agent = Agent(
    name="売上分析アシスタント",
    instructions="...",
)
```

です。

Agentは、

> **どのような役割で、どのように振る舞うか**

を定義します。

今回は、

```text
役割
＝売上分析支援

入力
＝Dataset Profile

出力
＝最初に見るべき分析観点
```

という責務にしています。

---

# Runnerとは何か

Agentを定義しただけでは、まだ実行されません。

```python
result = Runner.run_sync(
    agent,
    prompt,
)
```

でAgentを実行します。

流れは、

```text
Prompt
↓
Runner
↓
Agent
↓
Model
↓
Result
```

です。

---

# final_outputを取得する

最終回答は、

```python
result.final_output
```

から取得します。

```python
print(result.final_output)
```

とすれば、ターミナルへ表示できます。

---

# 実行する

次を実行します。

```bash
uv run python main.py
```

回答例は次のようなものになります。

```text
1. カテゴリ別売上
どの商品カテゴリが売上の中心なのか確認する。

2. 地域別売上
地域によって売上傾向に違いがあるか確認する。

3. 注文状態とレビュー対象
キャンセルや要レビューがどの程度発生しているか確認する。
```

ここで重要なのは、

> **まだ実際の売上金額を計算していない**

ことです。

今回は分析方針を考えただけです。

---

# 今回のArchitecture

```text
┌──────────────────────────┐
│ ai_agent_practice_sales  │
│          .csv            │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│        pandas            │
│   Dataset Profile作成    │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│         Runner           │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│          Agent           │
│   売上分析アシスタント   │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│      分析観点を提案       │
└──────────────────────────┘
```

---

# これはまだ「分析Agent」ではない

現時点では、

```text
CSVを集計する
```

能力はありません。

Agentができるのは、

```text
Dataset Profileを理解する
↓
分析観点を提案する
```

ところまでです。

つまり、

> **分析そのものではなく、分析方針を考えるAgent**

です。

---

# 次回で初めて実データを集計する

第2回では、Function Toolsを追加します。

たとえば、

```text
get_sales_summary
get_sales_by_category
get_sales_by_region
```

というToolを作ります。

構成は、

```text
User
↓
Agent
↓
Function Tool
↓
pandas
↓
CSV
↓
集計結果
↓
Agent
```

です。

これによって初めて、

> **Agentが必要な集計処理を選び、実データを分析する**

構成になります。

---

# 第1回で理解しておきたい4つのこと

```text
1. Agent
役割と振る舞いを定義する

2. Runner
Agentを実行する

3. final_output
最終回答を取得する

4. Dataset Profile
実データそのものではなく概要をAgentへ渡す
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Agent / Runner | ○ | 第1回 |
| CSV Dataset Profile | ○ | 第1回 |
| Function Tools | × | 第2回 |
| Structured Output | × | 第3回 |
| Streamlit | × | 第4回 |
| Sessions / Tracing | × | 第5回 |
| Guardrails | × | 第6回 |
| HITL | × | 第7回 |
| SQLite | × | 第8回 |
| Multi-table | × | 第9回 |
| Multi-Agent | × | 第10回 |
| FastAPI | × | 第11回 |
| Docker | × | 第12回 |
| PostgreSQL | × | 第13回 |
| Terraform / AWS | × | 第14回以降 |

---

# まとめ

今回は、CSV売上データを題材に、OpenAI Agents SDKの最小構成を作りました。

構成は、

```text
CSV
↓
Dataset Profile
↓
Agent
↓
分析観点
```

です。

重要なのは、

> **第1回ではまだCSVをAIへ集計させない**

ことです。

今回はAgent / Runnerという最小構成だけを理解しました。

次回から、AIエージェントが実際にCSVへアクセスして集計できるようにします。

---

# 次回予告

## 第2回：LLMに集計させない──Function ToolsとpandasでCSVを分析する

次回は、

```text
Agent
↓
Function Tool
↓
pandas
↓
CSV
↓
集計結果
↓
Agent
```

へ発展させます。

テーマは、

> **LLMに数値集計をさせず、pandasに確実に計算させる**

です。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

## Kindle：AI時代の仕様駆動開発

**『AI時代の仕様駆動開発: Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書』**

https://www.amazon.co.jp/dp/B0HBQ32KCQ

---

## Kindle：AIエージェント開発のためのケーススタディ設計

**『AIエージェント開発のためのケーススタディ設計: As-Is / To-BeからSystem Specification・Kiro Specへつなぐ業務設計』**

https://www.amazon.co.jp/dp/B0HJMM4JF7

---

## AIエージェント実践入門 Kindle

**2026年9月末公開予定**

業務課題からSystem Specification、SDD、Agent Design、OpenAI Agents SDK、Evaluationまでを体系的に扱う予定です。
