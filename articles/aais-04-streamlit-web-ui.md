---
title: "第4回 StreamlitでAIデータ分析AgentのWeb画面を作る──CLIから使えるApplicationへ"
emoji: "🖥️"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - Streamlit
  - pandas
  - CSV
published: true
published_at: "2026-09-29 07:00"
---

# はじめに

第3回では、売上分析Agentの出力をStructured Outputへ変えました。

ここまでの構成は、

```text
CSV
↓
pandas
↓
Function Tools
↓
Agent
↓
SalesAnalysisResult
```

です。

しかし、まだ操作はCLIです。

第4回では、

> **Streamlitを使って、ブラウザから売上分析Agentへ質問できるようにする**

ところまで進めます。

---

# 今回のゴール

完成イメージは次の通りです。

```text
Browser
↓
Streamlit
↓
User Question
↓
Agent
↓
Function Tool
↓
pandas
↓
CSV
↓
Structured Output
↓
Streamlit
↓
Browser
```

画面では、

```text
分析したい内容を入力してください

[ カテゴリ別の売上を分析してください ]

[ 分析する ]
```

のように質問できます。

分析結果は、

```text
分析概要
重要な発見
次のアクション
```

に分けて表示します。

---

# 今回のフォルダー構成

今回はUIとAgent処理を分けます。

```text
ai-agent-practice/
├── data/
│   └── ai_agent_practice_sales.csv
├── .env
├── .gitignore
├── agent_service.py
├── app.py
├── pyproject.toml
└── uv.lock
```

役割は、

```text
agent_service.py
＝Agent / Tools / pandas / Structured Output

app.py
＝Streamlit UI
```

です。

---

# Streamlitを追加する

```bash
uv add streamlit
```

ここまでで必要なライブラリは、

```text
openai-agents
python-dotenv
pandas
pydantic
streamlit
```

です。

---

# Agent処理を`agent_service.py`へ分ける

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

    return "\n".join(
        f"{row.category}: {int(row.sales_amount):,}円"
        for row in result.itertuples()
    )


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

    return "\n".join(
        f"{row.region}: {int(row.sales_amount):,}円"
        for row in result.itertuples()
    )


agent = Agent(
    name="売上分析アシスタント",
    instructions="""
あなたは売上データ分析を支援するAIアシスタントです。

ユーザーの質問に応じて、
必要なFunction Toolを選んでください。

売上集計は自分で計算せず、
必ずToolの結果を使用してください。

最終結果はSalesAnalysisResult形式で返してください。

analysis_typeには分析種別を短い英語識別子で設定してください。
key_findingsには重要な発見を2〜4件入れてください。
next_actionには次に確認すべき分析を1つ提案してください。
""",
    tools=[
        get_sales_summary,
        get_sales_by_category,
        get_sales_by_region,
    ],
    output_type=SalesAnalysisResult,
)


def run_sales_agent(
    question: str,
) -> SalesAnalysisResult:
    result = Runner.run_sync(
        agent,
        question,
    )

    return result.final_output
```

---

# Streamlit UIを作る

`app.py`を作ります。

```python
import streamlit as st

from agent_service import run_sales_agent


st.set_page_config(
    page_title="AI売上分析アシスタント",
    page_icon="📊",
)

st.title("AI売上分析アシスタント")
st.caption("OpenAI Agents SDK + pandas + Streamlit")

st.write("質問例:")

st.markdown(
    """
- 売上全体をまとめてください
- カテゴリ別の売上を分析してください
- 地域別の売上を比較してください
"""
)

with st.form("analysis_form"):
    question = st.text_area(
        "分析したい内容を入力してください",
        value="カテゴリ別の売上を分析してください。",
        height=100,
    )

    submitted = st.form_submit_button(
        "分析する"
    )


if submitted:
    if not question.strip():
        st.warning(
            "分析したい内容を入力してください。"
        )
    else:
        with st.spinner(
            "AIエージェントが分析しています..."
        ):
            try:
                result = run_sales_agent(
                    question=question.strip(),
                )

            except Exception as e:
                st.error(
                    "分析中にエラーが発生しました。"
                )
                st.exception(e)

            else:
                st.success(
                    "分析が完了しました。"
                )

                st.subheader("分析概要")
                st.write(
                    result.result_summary
                )

                st.subheader("重要な発見")
                for finding in result.key_findings:
                    st.markdown(
                        f"- {finding}"
                    )

                st.subheader("次のアクション")
                st.info(
                    result.next_action
                )

                with st.expander(
                    "Structured Outputを確認"
                ):
                    st.json(
                        result.model_dump()
                    )
```

---

# 実行する

```bash
uv run streamlit run app.py
```

ブラウザが開きます。

通常は、

```text
http://localhost:8501
```

のようなURLです。

---

# 画面で質問する

たとえば、

```text
カテゴリ別の売上を分析してください。
```

と入力します。

Agentは、

```text
質問を理解
↓
get_sales_by_categoryを選択
↓
pandasで集計
↓
結果を解釈
↓
SalesAnalysisResultで返す
```

という流れで処理します。

---

# Structured OutputをUIへ割り当てる

第3回で作ったFieldを、そのままUIへ割り当てています。

```text
result_summary
↓
分析概要

key_findings
↓
重要な発見

next_action
↓
次のアクション
```

つまり、

> **Structured Outputを作ったから、UIを安定して作れる**

という関係です。

---

# なぜ質問入力型にするのか

以前の単価・数量入力型では、

```text
商品単価
注文数量
```

を固定で入力していました。

今回からは、

```text
自然言語の質問
```

を入力します。

これによって、Agentが、

```text
全体集計
カテゴリ別
地域別
```

のどのToolを使うか判断できます。

ここで初めて、

> **AIエージェントらしいUI**

になってきます。

---

# 質問例を表示する

利用者に、

```text
何を聞けばよいか分からない
```

という問題があります。

そのため、

```text
売上全体をまとめてください
カテゴリ別の売上を分析してください
地域別の売上を比較してください
```

という例を表示します。

AI Applicationでは、

> **自由入力だけを置けばよいとは限らない**

ことも重要です。

---

# UIとAgentの責務を分ける

今回の構成は、

```text
app.py
＝Human Interface

agent_service.py
＝AI Processing
```

です。

UIは、

```text
入力
Submit
表示
```

に集中します。

Agent側は、

```text
Tool選択
CSV集計
分析
Structured Output
```

に集中します。

---

# 第11回のFastAPIにつながる

今は、

```text
Streamlit
↓
run_sales_agent()
↓
Agent
```

です。

第11回では、

```text
Streamlit
↓
HTTP
↓
FastAPI
↓
Agent
```

へ変えます。

今回からUIとAgentを別ファイルへ分けているのは、その準備でもあります。

---

# 今回のArchitecture

```text
┌──────────────────┐
│     Browser      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    Streamlit     │
│      app.py      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ run_sales_agent  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│      Agent       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Function Tools   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ pandas + CSV     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ SalesAnalysis    │
│     Result       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    Streamlit     │
└──────────────────┘
```

---

# 第1〜4回の成長

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
Function Tools
↓
Agent
```

第3回：

```text
CSV
↓
Tools
↓
Agent
↓
Structured Output
```

第4回：

```text
Browser
↓
Streamlit
↓
Agent
↓
Tools
↓
CSV
↓
Structured Output
↓
Browser
```

4回目で、

> **利用者がブラウザから分析できるApplication**

になりました。

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Agent / Runner | ○ | 第1回 |
| Function Tools | ○ | 第2回 |
| Structured Output | ○ | 第3回 |
| Streamlit | ○ | 第4回 |
| Sessions / Tracing | × | 第5回 |
| Guardrails | × | 第6回 |
| HITL | × | 第7回 |
| SQLite | × | 第8回 |
| Multi-table | × | 第9回 |
| Multi-Agent | × | 第10回 |
| FastAPI | × | 第11回 |

まだ、

> 「では関西だけでは？」

のようなFollow-up Conversationを継続する仕組みはありません。

次回はそこを追加します。

---

# まとめ

今回は、売上分析AgentへStreamlitのWeb UIを追加しました。

構成は、

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

です。

今回のポイントは、

> **Web画面を付けることは、Agentの能力を増やすことではない**

という点です。

追加したのは、

```text
Human Interface
```

です。

AgentとUIを分けたことで、今後のFastAPI化にもつながります。

---

# 次回予告

## 第5回：SessionsとTracingで「会話できて追跡できる」AIエージェントにする

次回は、

```text
「カテゴリ別売上は分かった。
では関西だけでは？」
```

のようなFollow-upを扱います。

同時に、

```text
どのToolが呼ばれたのか
Agentがどう実行されたのか
```

をTracingで追跡します。

つまり、

```text
Sessions
＝会話を継続する

Tracing
＝実行を追跡する
```

の違いを実装しながら確認します。

---

# 関連記事・教材

## 第1回

**最小構成から始めるAIエージェント実装──CSV売上データを題材にAgentとRunnerを理解する**

https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data

## 第2回

**LLMに集計させない──Function ToolsとpandasでCSVを分析する**

https://zenn.dev/tigerone1945/articles/aais-02-function-tools-pandas-csv-analysis

## 第3回

**Structured Outputで分析結果を安定させる──Pydanticで「文章」を「データ」に変える**

https://zenn.dev/tigerone1945/articles/aais-03-structured-output-pydantic

## Kindle：AI時代の仕様駆動開発

https://www.amazon.co.jp/dp/B0HBQ32KCQ

## Kindle：AIエージェント開発のためのケーススタディ設計

https://www.amazon.co.jp/dp/B0HJMM4JF7
