---
title: "第2回 LLMに集計させない──Function ToolsとpandasでCSVを分析する"
emoji: "🛠️"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - pandas
  - CSV
  - Python
published: true
published_at: "2026-09-22 07:00"
---

# はじめに

第1回では、売上CSVのDataset ProfileをAgentへ渡し、

```text
CSV
↓
Dataset Profile
↓
Agent
↓
分析観点を提案
```

という最小構成を作りました。

ただし、まだAgentは実際の売上金額を計算していません。

第2回では、いよいよCSVを実際に集計します。

今回のテーマは、

> **LLMに集計させない**

です。

AIエージェントが売上分析を行う場合でも、

```text
売上合計
カテゴリ別集計
地域別集計
```

のような処理をLLM自身に計算させる必要はありません。

そこで、

```text
LLM
＝質問を理解する
＝必要なToolを選ぶ
＝結果を説明する

pandas
＝CSVを読む
＝集計する
＝数値を計算する
```

と責務を分けます。

---

# 今回のゴール

第1回の構成を、

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
↓
User
```

へ発展させます。

今回は3つのToolを作ります。

```text
get_sales_summary()
get_sales_by_category()
get_sales_by_region()
```

利用者が、

> カテゴリ別の売上を教えてください

と質問すると、Agentが適切なToolを選び、pandasがCSVを集計します。

---

# なぜLLMに集計させないのか

たとえば300行のCSVをLLMへ渡して、

> 売上合計を計算してください

と依頼することもできます。

しかし、業務Applicationでは、

```text
数値計算
集計
フィルタ
ソート
```

は、通常のコードで行う方が扱いやすくなります。

今回の基本原則は、

```text
LLM
＝Reasoning / Interpretation

Code
＝Calculation / Aggregation
```

です。

---

# 今回使うCSV

ファイルは前回と同じです。

```text
data/ai_agent_practice_sales.csv
```

ここで特に重要な列が、

```text
total_amount
sales_amount
status
```

です。

このCSVでは、

```text
total_amount
＝注文時点の金額

sales_amount
＝実売上として計上する金額
```

という違いがあります。

キャンセル注文は、

```text
status = キャンセル
sales_amount = 0
```

です。

したがって、

> **売上を集計する場合は`sales_amount`を使う**

という業務ルールをPython側に持たせます。

これも、

> **LLMに業務ルールを曖昧に解釈させず、コードで明示する**

例です。

---

# フォルダー構成

今回は次の構成にします。

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

必要なライブラリは、

```bash
uv add openai-agents
uv add python-dotenv
uv add pandas
```

です。

---

# 完成コード

`main.py`を次のようにします。

```python
from pathlib import Path

import pandas as pd
from dotenv import load_dotenv
from agents import Agent, Runner
from agents.decorators import tool

load_dotenv()

CSV_PATH = Path("data/ai_agent_practice_sales.csv")


def load_sales_data() -> pd.DataFrame:
    return pd.read_csv(CSV_PATH)


@tool
def get_sales_summary() -> str:
    """
    売上データ全体の基本集計を返す。
    売上金額にはsales_amountを使用する。
    """
    df = load_sales_data()

    total_sales = int(df["sales_amount"].sum())
    order_count = len(df)
    cancelled_count = int((df["status"] == "キャンセル").sum())
    review_count = int((df["review_flag"] == "要レビュー").sum())

    return f"""
総レコード数: {order_count}
売上合計: {total_sales:,}円
キャンセル件数: {cancelled_count}
要レビュー件数: {review_count}
""".strip()


@tool
def get_sales_by_category() -> str:
    """
    商品カテゴリ別の売上を集計する。
    売上金額にはsales_amountを使用する。
    """
    df = load_sales_data()

    result = (
        df.groupby("category", as_index=False)["sales_amount"]
        .sum()
        .sort_values("sales_amount", ascending=False)
    )

    lines = [
        f"{row.category}: {int(row.sales_amount):,}円"
        for row in result.itertuples()
    ]

    return "\n".join(lines)


@tool
def get_sales_by_region() -> str:
    """
    地域別の売上を集計する。
    売上金額にはsales_amountを使用する。
    """
    df = load_sales_data()

    result = (
        df.groupby("region", as_index=False)["sales_amount"]
        .sum()
        .sort_values("sales_amount", ascending=False)
    )

    lines = [
        f"{row.region}: {int(row.sales_amount):,}円"
        for row in result.itertuples()
    ]

    return "\n".join(lines)


agent = Agent(
    name="売上分析アシスタント",
    instructions="""
あなたは売上データ分析を支援するAIアシスタントです。

ユーザーの質問に応じて、
必要なFunction Toolを選択してください。

売上集計が必要な場合、
自分で数値計算せず、必ずToolの結果を使用してください。

Toolが返した結果を、
簡潔で分かりやすい日本語で説明してください。
""",
    tools=[
        get_sales_summary,
        get_sales_by_category,
        get_sales_by_region,
    ],
)

result = Runner.run_sync(
    agent,
    "カテゴリ別の売上を教えてください。",
)

print(result.final_output)
```

---

# 3つのToolを作った

今回は、

```text
売上全体
カテゴリ別
地域別
```

という3つの分析だけをTool化しました。

Agentが持つToolは、

```text
売上分析アシスタント
├─ get_sales_summary
├─ get_sales_by_category
└─ get_sales_by_region
```

です。

---

# `load_sales_data()`を分ける理由

CSV読込を、

```python
def load_sales_data() -> pd.DataFrame:
    return pd.read_csv(CSV_PATH)
```

へ分けています。

各Toolの中で毎回、

```python
pd.read_csv(...)
```

を書くこともできます。

しかし、

```text
データ読込
↓
分析処理
```

を分けておくと、後でCSVからSQLiteへ変えるときにも変更しやすくなります。

---

# Tool 1：売上全体を取得する

```python
@tool
def get_sales_summary() -> str:
```

では、

```text
総レコード数
売上合計
キャンセル件数
要レビュー件数
```

を取得します。

売上合計は、

```python
df["sales_amount"].sum()
```

です。

`total_amount`ではありません。

ここに、業務ルールをコードとして明示しています。

---

# Tool 2：カテゴリ別売上

```python
df.groupby(
    "category",
    as_index=False,
)["sales_amount"].sum()
```

でカテゴリ別に集計します。

Agentは、

```text
カテゴリ別に集計する方法
```

を考える必要がありません。

Toolの役割は、

> **確実にカテゴリ別売上を返す**

ことです。

---

# Tool 3：地域別売上

地域別も同じです。

```python
df.groupby(
    "region",
    as_index=False,
)["sales_amount"].sum()
```

で集計します。

Agentは利用者の質問を理解して、

```text
カテゴリ別？
地域別？
全体？
```

を判断します。

---

# 実行する

```bash
uv run python main.py
```

質問を、

```text
カテゴリ別の売上を教えてください。
```

としているので、Agentは`get_sales_by_category`を選びます。

処理は、

```text
User
↓
Agent
↓
get_sales_by_category
↓
pandas
↓
CSV
↓
集計結果
↓
Agent
↓
説明
```

です。

---

# 質問を変えて試す

たとえば、

```python
result = Runner.run_sync(
    agent,
    "売上全体の状況を教えてください。",
)
```

とすれば、`get_sales_summary`が使われます。

また、

```python
result = Runner.run_sync(
    agent,
    "地域別に売上を比較してください。",
)
```

とすれば、`get_sales_by_region`が候補になります。

---

# Function Toolの本質

今回重要なのは、

> Python関数をAIに呼ばせる

ことだけではありません。

本質は、

> **Agentの責務と、決定的処理の責務を分ける**

ことです。

```text
Agent
＝何を調べるか決める

Tool
＝決められた処理を実行する
```

という関係です。

---

# Tool名はInterface

Tool名は、

```text
get_sales_summary
get_sales_by_category
get_sales_by_region
```

としています。

これは単なるPython関数名ではありません。

Agentが、

> **何のためのToolなのか**

を判断するためのInterfaceでもあります。

そのため、

```text
tool1()
calc()
do_it()
```

のような名前より、目的が明確な名前の方が適しています。

---

# docstringも重要

たとえば、

```python
"""
商品カテゴリ別の売上を集計する。
売上金額にはsales_amountを使用する。
"""
```

と書いています。

これによって、

```text
このToolは何をするのか
どの列を売上とするのか
```

を明確にしています。

Tool設計では、

```text
Function Name
Type Hint
Docstring
```

をセットで考えます。

---

# Toolに任意SQLを渡さない

今後DBへ発展すると、

```text
AgentにSQLを書かせればよいのでは？
```

という考え方も出てきます。

しかしこのシリーズでは、まず、

```text
目的を限定したTool
```

を優先します。

たとえば、

```text
get_sales_by_category
```

の方が、

```text
execute_any_sql
```

より責務が明確です。

この考え方は、第9回のMulti-tableにもつながります。

---

# 第1回からの変化

第1回：

```text
CSV
↓
Dataset Profile
↓
Agent
↓
分析観点
```

第2回：

```text
CSV
↑
pandas
↑
Function Tool
↑
Agent
↑
User Question
```

第2回で初めて、

> **Agentが実データを集計できる**

構成になりました。

---

# 第2回で理解しておきたい5つのこと

```text
1. Function Tool
Python処理をAgentから利用できる

2. pandas
CSV集計を確実に実行する

3. Tool Selection
Agentが必要なToolを選ぶ

4. Business Rule
sales_amountを売上として使う

5. Responsibility
LLMとコードの責務を分ける
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Agent / Runner | ○ | 第1回 |
| Function Tools | ○ | 第2回 |
| pandas集計 | ○ | 第2回 |
| Structured Output | × | 第3回 |
| Streamlit | × | 第4回 |
| Sessions / Tracing | × | 第5回 |
| Guardrails | × | 第6回 |
| HITL | × | 第7回 |
| SQLite | × | 第8回 |
| Multi-table | × | 第9回 |
| Multi-Agent | × | 第10回 |

---

# まとめ

今回は、Function Toolsとpandasを使って、AIエージェントがCSVを集計できるようにしました。

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
Result
↓
Agent
```

です。

中心メッセージは、

> **LLMに集計させない**

です。

LLMには、

```text
質問理解
Tool選択
説明
```

を任せます。

数値集計は、

```text
Python / pandas
```

へ任せます。

---

# 次回予告

## 第3回：Structured Outputで分析結果を安定させる

次回は、Agentの最終回答を、

```text
自然言語
```

から、

```text
Pydantic Model
```

へ変えます。

つまり、

> **分析結果をApplicationから扱えるデータにする**

ところまで進めます。

---

# 関連記事・教材

## 第1回

**最小構成から始めるAIエージェント実装──CSV売上データを題材にAgentとRunnerを理解する**

https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data

---

## Kindle：AI時代の仕様駆動開発

https://www.amazon.co.jp/dp/B0HBQ32KCQ

---

## Kindle：AIエージェント開発のためのケーススタディ設計

https://www.amazon.co.jp/dp/B0HJMM4JF7
