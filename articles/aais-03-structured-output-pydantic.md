---
title: "第3回 Structured Outputで分析結果を安定させる──Pydanticで「文章」を「データ」に変える"
emoji: "🧱"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - Pydantic
  - pandas
  - StructuredOutput
published: true
published_at: "2026-09-25 07:00"
---

# はじめに

第2回では、Function Toolsとpandasを使って、CSV売上データを実際に集計できるようにしました。

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

でした。

ただし、最終回答はまだ自然言語です。

たとえば、

```text
IT機器の売上が最も高く、次にオフィス家具が続いています。
```

という文章だけでは、後続のApplicationから使いにくい場合があります。

そこで第3回では、

> **分析結果をStructured Outputへ変える**

ところまで進めます。

---

# 今回のゴール

第2回の構成を、

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
Agent
↓
SalesAnalysisResult
↓
Application
```

へ発展させます。

最終結果を、

```json
{
  "question": "カテゴリ別の売上を分析してください",
  "analysis_type": "category_sales",
  "result_summary": "カテゴリ別売上を比較しました。",
  "key_findings": [
    "IT機器が売上上位です",
    "オフィス家具も大きな比率を占めます"
  ],
  "next_action": "上位カテゴリの商品別内訳を確認します"
}
```

のような構造にします。

---

# なぜStructured Outputが必要なのか

人が読むだけなら自然言語で十分です。

しかし、今後は、

```text
Streamlit
SQLite
FastAPI
PostgreSQL
```

へ分析結果を渡します。

そのとき、

```text
分析タイプは何か
要点は何か
次のActionは何か
```

を文章から再び取り出すのは扱いにくくなります。

そこで、

```text
Text
↓
Structured Data
```

へ変えます。

---

# Pydanticモデルを作る

今回は次のモデルを使います。

```python
from pydantic import BaseModel


class SalesAnalysisResult(BaseModel):
    question: str
    analysis_type: str
    result_summary: str
    key_findings: list[str]
    next_action: str
```

Fieldの意味は次の通りです。

| Field | 内容 |
|---|---|
| `question` | ユーザーの質問 |
| `analysis_type` | 分析種別 |
| `result_summary` | 分析結果の要約 |
| `key_findings` | 重要な発見 |
| `next_action` | 次に見るべき分析 |

---

# フォルダー構成

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

Pydanticを追加します。

```bash
uv add pydantic
```

---

# 完成コード

```python
from pathlib import Path

import pandas as pd
from dotenv import load_dotenv
from pydantic import BaseModel
from agents import Agent, Runner
from agents.decorators import tool

load_dotenv()

CSV_PATH = Path("data/ai_agent_practice_sales.csv")


class SalesAnalysisResult(BaseModel):
    question: str
    analysis_type: str
    result_summary: str
    key_findings: list[str]
    next_action: str


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
    cancelled_count = int(
        (df["status"] == "キャンセル").sum()
    )
    review_count = int(
        (df["review_flag"] == "要レビュー").sum()
    )

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
        df.groupby(
            "category",
            as_index=False,
        )["sales_amount"]
        .sum()
        .sort_values(
            "sales_amount",
            ascending=False,
        )
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
        df.groupby(
            "region",
            as_index=False,
        )["sales_amount"]
        .sum()
        .sort_values(
            "sales_amount",
            ascending=False,
        )
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
必要なFunction Toolを利用してください。

売上集計は自分で計算せず、
必ずToolの結果を使用してください。

最終結果はSalesAnalysisResultの形式で返してください。

analysis_typeには、
今回行った分析種別を短い英語識別子で設定してください。

key_findingsには、
重要な発見を2〜4件入れてください。

next_actionには、
次に確認すべき分析を1つ提案してください。
""",
    tools=[
        get_sales_summary,
        get_sales_by_category,
        get_sales_by_region,
    ],
    output_type=SalesAnalysisResult,
)

question = "カテゴリ別の売上を分析してください。"

result = Runner.run_sync(
    agent,
    question,
)

output = result.final_output

print(output)
print(output.model_dump())
```

---

# `output_type`を追加する

今回の中心は、

```python
output_type=SalesAnalysisResult
```

です。

これによって、最終回答は単なる文字列ではなく、

```text
SalesAnalysisResult
```

として扱えます。

---

# final_outputを使う

```python
output = result.final_output
```

とすれば、

```python
output.question
output.analysis_type
output.result_summary
output.key_findings
output.next_action
```

へ直接アクセスできます。

---

# JSON / dictへ変換する

Pydanticなので、

```python
output.model_dump()
```

でdictへ変換できます。

また、

```python
output.model_dump_json(indent=2)
```

でJSON文字列にもできます。

これは第4回のStreamlit表示でも使います。

---

# Structured Outputは表示形式ではない

Structured Outputの目的は、

> JSONで見栄えよく表示すること

ではありません。

目的は、

> **AgentとApplicationのInterfaceを作ること**

です。

今回であれば、

```text
Agent
↓
SalesAnalysisResult
↓
Streamlit
```

という接続ができます。

---

# Function ToolとStructured Outputの違い

第2回と第3回は役割が違います。

Function Tool：

```text
Agent
↓
Code
```

Structured Output：

```text
Agent
↓
Application
```

つまり、

```text
Tool
＝Agentが処理を実行するための入口

Structured Output
＝Agentが結果を渡すための出口
```

です。

---

# Fieldを増やしすぎない

たとえば、

```text
confidence
priority
trend
risk
message
status
notes
```

などを大量に追加することもできます。

しかし、後続Applicationで使わないFieldまで増やすと、Schemaが複雑になります。

今回は、

```text
question
analysis_type
result_summary
key_findings
next_action
```

だけに絞ります。

---

# 次回のStreamlitへどうつながるか

第4回では、

```python
output.result_summary
```

を画面に表示し、

```python
output.key_findings
```

を箇条書きにし、

```python
output.next_action
```

を次の分析提案として表示できます。

つまり、第3回で構造化したことが、そのままUI設計へつながります。

---

# 第1〜3回の成長

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
↓
pandas
↓
Function Tool
↓
Agent
↓
Text
```

第3回：

```text
CSV
↓
pandas
↓
Function Tool
↓
Agent
↓
SalesAnalysisResult
↓
Application
```

ここまでで、AIエージェントの出力をApplicationへ渡せるようになりました。

---

# 第3回で理解しておきたい5つのこと

```text
1. Pydantic BaseModel
分析結果のSchemaを定義する

2. output_type
Agentの最終出力型を指定する

3. final_output
型付き結果を取得する

4. model_dump()
dictへ変換する

5. Interface
AgentとApplicationの境界として考える
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Agent / Runner | ○ | 第1回 |
| Function Tools | ○ | 第2回 |
| pandas集計 | ○ | 第2回 |
| Structured Output | ○ | 第3回 |
| Streamlit | × | 第4回 |
| Sessions / Tracing | × | 第5回 |
| Guardrails | × | 第6回 |
| HITL | × | 第7回 |
| SQLite | × | 第8回 |
| Multi-table | × | 第9回 |
| Multi-Agent | × | 第10回 |

---

# まとめ

今回は、売上分析Agentの最終回答をStructured Outputへ変えました。

```text
Text
↓
SalesAnalysisResult
```

としたことで、

```python
output.result_summary
output.key_findings
output.next_action
```

のように、後続Applicationから直接使えるようになりました。

中心メッセージは、

> **AIの分析結果を文章で終わらせず、Applicationが利用できるデータへ変える**

です。

---

# 次回予告

## 第4回：StreamlitでAIデータ分析AgentのWeb画面を作る

次回は、

```text
Browser
↓
Streamlit
↓
Agent
↓
Function Tools
↓
pandas
↓
CSV
↓
Structured Output
↓
Browser
```

へ発展させます。

CLIで動いていたAgentを、

> **利用者がブラウザから質問できるApplication**

へ変えます。

---

# 関連記事・教材

## 第1回

**最小構成から始めるAIエージェント実装──CSV売上データを題材にAgentとRunnerを理解する**

https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data

## 第2回

**LLMに集計させない──Function ToolsとpandasでCSVを分析する**

https://zenn.dev/tigerone1945/articles/aais-02-function-tools-pandas-csv-analysis
