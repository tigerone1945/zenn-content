---
title: "第5回 SessionsとTracingで「会話できて追跡できる」AIエージェントにする"
emoji: "🧠"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - Sessions
  - Tracing
  - Streamlit
published: true
published_at: "2026-10-02 07:00"
---

# はじめに

第4回では、売上分析AgentへStreamlitのWeb UIを追加しました。

ここまでの構成は、次のようになっています。

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

これで、利用者はブラウザから、

```text
カテゴリ別の売上を分析してください
```

のように質問できるようになりました。

しかし、まだ1つ大きな課題があります。

現在の構成では、

```text
1回質問する
↓
Agentが回答する
↓
終了
```

です。

たとえば最初に、

```text
地域別の売上を分析してください。
```

と質問したあと、

```text
では、売上が最も高かった地域は？
```

と聞いても、前の会話を引き継ぐ仕組みがなければ、

> 「何の地域別売上について話していたのか」

という文脈を保持できません。

そこで第5回では、

> **Sessionsで会話を継続し、TracingでAgentの実行を追跡する**

ところまで進めます。

今回扱う2つは、似て見えますが役割がまったく違います。

```text
Sessions
＝会話を覚える

Tracing
＝実行を追跡する
```

この違いを、実装しながら確認します。

---

# 今回のゴール

第4回の売上分析Applicationを、

```text
Browser
↓
Streamlit
↓
Session
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
Tracing
```

へ発展させます。

今回できるようにすることは、主に次の4つです。

1. 同じ会話の文脈を次の質問へ引き継ぐ
2. Streamlit上で会話履歴を表示する
3. Agentの実行をTracingで確認する
4. SessionとTracingの責務を区別する

---

# SessionsとTracingは何が違うのか

最初に、この2つを整理します。

## Sessions

Sessionsは、

> **複数回のAgent実行にまたがって会話履歴を保持する仕組み**

です。

イメージは、

```text
1回目
User:
地域別の売上を分析してください

Agent:
関東が最も高く……

        ↓ Sessionに保存

2回目
User:
では、売上が最も高かった地域は？

        ↓ 前回の会話を参照

Agent:
前回の分析では関東です
```

です。

## Tracing

Tracingは、

> **Agentが内部でどのように実行されたのかを追跡する仕組み**

です。

たとえば、

```text
User Input
↓
Agent
↓
get_sales_by_region
↓
Tool Result
↓
Model
↓
Final Output
```

という実行の流れを確認できます。

OpenAI Agents SDKではTracingが標準で組み込まれており、通常はデフォルトで有効です。

---

# つまり「Memory」と「Observability」

少し別の言い方をすると、

```text
Sessions
＝Memory

Tracing
＝Observability
```

です。

Sessionsは、

> **Agentが何を覚えているか**

に関係します。

Tracingは、

> **Agentが何をしたか**

に関係します。

この2つを混同しないことが、第5回の重要なポイントです。

---

# 今回も同じCSVを使う

これまでと同じ、

```text
ai_agent_practice_sales.csv
```

を使用します。

主な列は、

```text
order_date
customer_name
product_name
category
sales_amount
region
salesperson
status
review_flag
```

などです。

今回も業務データ自体はCSVのままです。

---

# 「SQLiteSession」と第8回のSQLiteは別物

今回、Sessionsの保存には、

```python
SQLiteSession
```

を使います。

ここで注意が必要です。

第8回では、

> **売上CSVを業務データベースとしてSQLiteへ移行する**

予定です。

今回のSQLiteSessionは、それとは別です。

整理すると、

```text
第5回 SQLiteSession
＝会話履歴を保存するためのSQLite

第8回 SQLite
＝売上・受注データを保存する業務DB
```

です。

同じSQLiteを使いますが、目的が違います。

---

# 今回のフォルダー構成

第4回の構成に、会話履歴DBが加わります。

```text
ai-agent-practice/
├── data/
│   ├── ai_agent_practice_sales.csv
│   └── conversations.db
├── .env
├── .gitignore
├── agent_service.py
├── app.py
├── pyproject.toml
└── uv.lock
```

`conversations.db`は、Agentの会話履歴を保存するために使います。

---

# 必要なライブラリ

第4回までの環境をそのまま利用できます。

```bash
uv add openai-agents
uv add python-dotenv
uv add pandas
uv add pydantic
uv add streamlit
```

今回の`SQLiteSession`はOpenAI Agents SDK側から利用します。

---

# Agent側をSessions対応にする

まず`agent_service.py`を修正します。

```python
from pathlib import Path

import pandas as pd
from dotenv import load_dotenv
from pydantic import BaseModel

from agents import (
    Agent,
    Runner,
    SQLiteSession,
    trace,
)
from agents.decorators import tool


load_dotenv()

CSV_PATH = Path(
    "data/ai_agent_practice_sales.csv"
)

SESSION_DB_PATH = Path(
    "data/conversations.db"
)


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

    total_sales = int(
        df["sales_amount"].sum()
    )

    order_count = len(df)

    cancelled_count = int(
        (df["status"] == "キャンセル").sum()
    )

    review_count = int(
        (
            df["review_flag"]
            == "要レビュー"
        ).sum()
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
        f"{row.category}: "
        f"{int(row.sales_amount):,}円"
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
        f"{row.region}: "
        f"{int(row.sales_amount):,}円"
        for row in result.itertuples()
    )


agent = Agent(
    name="売上分析アシスタント",
    instructions="""
あなたは売上データ分析を支援する
AIアシスタントです。

ユーザーの質問に応じて、
必要なFunction Toolを選んでください。

売上集計は自分で計算せず、
必ずToolの結果を使用してください。

会話履歴がある場合は、
直前までの質問と回答を踏まえて
Follow-upの質問に回答してください。

最終結果は
SalesAnalysisResult形式で返してください。

analysis_typeには、
分析種別を短い英語識別子で
設定してください。

key_findingsには、
重要な発見を2〜4件入れてください。

next_actionには、
次に確認すべき分析を
1つ提案してください。
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
    session_id: str,
) -> SalesAnalysisResult:

    session = SQLiteSession(
        session_id,
        str(SESSION_DB_PATH),
    )

    with trace(
        workflow_name="sales_analysis_chat",
        group_id=session_id,
    ):
        result = Runner.run_sync(
            agent,
            question,
            session=session,
        )

    return result.final_output
```

今回の重要な変更点は、

```python
SQLiteSession
```

と、

```python
session=session
```

です。

---

# SQLiteSessionを作る

次の部分です。

```python
session = SQLiteSession(
    session_id,
    str(SESSION_DB_PATH),
)
```

ここで、

```text
session_id
＝どの会話かを識別するID

conversations.db
＝会話履歴を保存するDB
```

を指定しています。

同じ`session_id`を使えば、次回の`Runner.run_sync()`でも前の会話履歴が利用されます。

---

# RunnerへSessionを渡す

第4回までは、

```python
result = Runner.run_sync(
    agent,
    question,
)
```

でした。

今回は、

```python
result = Runner.run_sync(
    agent,
    question,
    session=session,
)
```

とします。

これだけで、RunnerがSessionから会話履歴を取得し、新しい質問と合わせてAgentへ渡してくれます。

実行後の新しい会話内容もSessionへ保存されます。

---

# Tracingはすでに有効

OpenAI Agents SDKでは、Tracingは通常デフォルトで有効です。

`Runner.run()`、`Runner.run_sync()`、`Runner.run_streamed()`などの実行について、Agent実行、Model generation、Function Tool callなどがTraceとして記録されます。

今回のコードでは、さらに、

```python
with trace(
    workflow_name="sales_analysis_chat",
    group_id=session_id,
):
```

としています。

Tracing自体はデフォルトでも行われますが、今回は`workflow_name`と`group_id`を明示するために`trace()`を使います。

---

# `group_id`で同じ会話をまとめる

```python
group_id=session_id
```

とすることで、複数のTraceを同じ会話として関連付けやすくなります。

```text
Session ID
＝会話履歴の識別

Trace group_id
＝Trace上で会話を関連付ける識別
```

として、今回は同じIDを利用します。

---

# StreamlitをChat UIへ変更する

第4回では、`text_area`とSubmitボタンを使っていました。

今回は会話を続けるため、Chat形式のUIへ変更します。

`app.py`を次のようにします。

```python
from uuid import uuid4

import streamlit as st

from agent_service import run_sales_agent


st.set_page_config(
    page_title="AI売上分析アシスタント",
    page_icon="📊",
)

st.title("AI売上分析アシスタント")
st.caption(
    "OpenAI Agents SDK "
    "+ Sessions + Tracing"
)


if "session_id" not in st.session_state:
    st.session_state.session_id = (
        f"sales_{uuid4().hex}"
    )


if "messages" not in st.session_state:
    st.session_state.messages = []


with st.sidebar:
    st.subheader("Conversation")

    st.code(
        st.session_state.session_id
    )

    if st.button(
        "新しい会話を開始"
    ):
        st.session_state.session_id = (
            f"sales_{uuid4().hex}"
        )
        st.session_state.messages = []
        st.rerun()


st.write("質問例:")

st.markdown(
    """
- 地域別の売上を分析してください
- カテゴリ別の売上を分析してください
- 売上全体をまとめてください
"""
)


for message in st.session_state.messages:
    with st.chat_message(
        message["role"]
    ):
        st.markdown(
            message["content"]
        )


question = st.chat_input(
    "売上データについて質問してください"
)


if question:

    st.session_state.messages.append(
        {
            "role": "user",
            "content": question,
        }
    )

    with st.chat_message("user"):
        st.markdown(question)

    with st.chat_message("assistant"):

        with st.spinner(
            "AIエージェントが"
            "分析しています..."
        ):
            try:
                result = run_sales_agent(
                    question=question,
                    session_id=(
                        st.session_state
                        .session_id
                    ),
                )

            except Exception as e:
                st.error(
                    "分析中にエラーが"
                    "発生しました。"
                )
                st.exception(e)

            else:
                response = (
                    f"### 分析概要\n"
                    f"{result.result_summary}\n\n"
                    f"### 重要な発見\n"
                    + "\n".join(
                        f"- {item}"
                        for item
                        in result.key_findings
                    )
                    + "\n\n"
                    f"### 次のアクション\n"
                    f"{result.next_action}"
                )

                st.markdown(response)

                with st.expander(
                    "Structured Output"
                ):
                    st.json(
                        result.model_dump()
                    )

                st.session_state.messages.append(
                    {
                        "role": "assistant",
                        "content": response,
                    }
                )
```

---

# `st.session_state`とAgents SDK Sessionは別物

今回、2種類の「Session」が登場しています。

```text
Streamlit st.session_state

OpenAI Agents SDK SQLiteSession
```

役割は違います。

```text
st.session_state
＝UI表示用

SQLiteSession
＝Agentの会話Memory
```

です。

この2つを同じものと考えないようにします。

---

# Follow-upを試してみる

まず、次のように質問します。

```text
地域別の売上を分析してください。
```

Agentが地域別分析を返したら、続けて、

```text
では、売上が最も高かった地域は？
```

と入力します。

Sessionがあれば、前回の会話を参照できます。

さらに、

```text
次に確認するとしたら何を見るべき？
```

と続けることもできます。

このように、

```text
単発質問
```

から、

```text
対話型分析
```

へ変わります。

---

# Sessionsは「長期記憶」ではない

Sessionsを使うと会話履歴を保持できます。

ただし、何でも永久に覚える長期Memoryと考えるのは少し違います。

Sessionsは基本的に、ある会話の履歴を管理する仕組みです。

長い会話では、Context量やToken量も増えるため、本番Applicationでは履歴をどこまで利用するかという設計も必要になります。

---

# Tracingで何が見えるのか

Tracingでは、Agent実行の中で、

```text
Model Call
Tool Call
Tool Result
Agent Span
Generation
```

などを追跡できます。

今回の質問なら、

```text
地域別売上を分析してください
↓
Agent
↓
get_sales_by_region
↓
pandas集計結果
↓
Agent
↓
Structured Output
```

という流れを確認できます。

---

# Trace Viewerを開く

OpenAI PlatformのTrace Viewerから確認できます。

```text
https://platform.openai.com/traces
```

Agentを実行したあと、Traceを開くと、どのAgentが動いたか、どのToolが呼ばれたか、Modelが何回呼ばれたか、といった実行状況を確認できます。

---

# Tracingはデバッグに強い

たとえば、

> 「カテゴリ別売上を聞いたのに違うToolが呼ばれた」

という問題があったとします。

Tracingがあれば、

```text
User Input
↓
Model
↓
選択されたTool
↓
Tool Input
↓
Tool Output
↓
Final Output
```

を追えます。

AIエージェントでは、結果だけでなく、実行過程を確認できることが重要です。

---

# Application LogとTracingは違う

今後、本番化するとApplication Logも使います。

```text
Tracing
＝Agent Workflowの実行経路

Application Log
＝Application全体のイベント記録
```

たとえばHTTP Request、Error、DB ConnectionなどはApplication Log側で扱うことがあります。

第16回ではCloudWatchも含めて、この違いを改めて扱います。

---

# Sensitive Dataに注意する

Tracingでは、ModelのInput / OutputやFunction ToolのInput / Outputが記録される場合があります。

したがって、本番環境では、個人情報・顧客情報・機密情報をTraceへ含めてよいか検討が必要です。

学習用の今回のCSVは架空データですが、実データを利用する場合は、

> **Tracingに何を記録するか**

も設計対象になります。

---

# 今回のArchitecture

```text
Browser
↓
Streamlit
├─ st.session_state
│  └─ UI表示履歴
│
└─ session_id
   ↓
SQLiteSession
└─ Agent会話履歴
   ↓
Agent
↓
Function Tools
↓
pandas + CSV
↓
SalesAnalysisResult

同時に

Runner / Agent / Tool
        ↓
      Tracing
        ↓
 OpenAI Trace Viewer
```

---

# 第1〜5回の成長

## 第1回

```text
CSV
↓
Dataset Profile
↓
Agent
↓
分析観点
```

## 第2回

```text
CSV
↓
pandas
↓
Function Tools
↓
Agent
```

## 第3回

```text
CSV
↓
Tools
↓
Agent
↓
Structured Output
```

## 第4回

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
```

## 第5回

```text
Browser
↓
Streamlit
↓
Session
↓
Agent
↓
Tools
↓
CSV

＋

Tracing
```

第5回で、単発の分析Applicationから、会話を続けられ、実行過程も追跡できるApplicationへ変わりました。

---

# 第5回で理解しておきたい6つのこと

```text
1. SQLiteSession
会話履歴を保持する

2. session_id
会話単位を識別する

3. Runner session=
過去の会話を次のRunへ引き継ぐ

4. Tracing
Agentの実行過程を記録する

5. group_id
同じ会話のTraceを関連付ける

6. UI StateとAgent Memory
役割は別
```

最も重要なのは、

```text
Sessions
＝会話のMemory

Tracing
＝実行のObservability
```

という違いです。

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Agent / Runner | ○ | 第1回 |
| Function Tools | ○ | 第2回 |
| Structured Output | ○ | 第3回 |
| Streamlit | ○ | 第4回 |
| Sessions | ○ | 第5回 |
| Tracing | ○ | 第5回 |
| Guardrails | × | 第6回 |
| HITL | × | 第7回 |
| 業務データSQLite化 | × | 第8回 |
| Multi-table | × | 第9回 |
| Multi-Agent | × | 第10回 |
| FastAPI | × | 第11回 |

次回は、Agentが「できること」を増やすのではなく、やってはいけないことを制御する方向へ進みます。

---

# まとめ

今回は、売上分析AgentへSessionsとTracingを追加しました。

Sessionsを使うことで、

```text
地域別売上を分析して
↓
では一番高い地域は？
```

というFollow-up Conversationが可能になります。

一方、Tracingを使うことで、

```text
どのAgentが動いたか
どのToolが呼ばれたか
どのように処理されたか
```

を確認できます。

つまり、

```text
Sessions
＝会話を継続する

Tracing
＝実行を追跡する
```

です。

この2つが入ることで、AIエージェントは単なる一問一答のツールから、

> **会話しながら分析でき、内部動作も確認できるApplication**

へ一段進みました。

---

# 次回予告

## 第6回：GuardrailsでAIエージェントの「やってはいけない」を設計する

次回は、

```text
入力
↓
Guardrail
↓
Agent
↓
Tool
↓
Structured Output
```

へ発展させます。

テーマは、

> **Instructionsに「やらないで」と書くだけで安全なのか？**

です。

AIエージェントを業務で使うために必要な、安全境界の作り方を扱います。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

## 第1回

**最小構成から始めるAIエージェント実装──CSV売上データを題材にAgentとRunnerを理解する**

https://zenn.dev/tigerone1945/articles/aais-01-agent-runner-csv-sales-data

## 第2回

**LLMに集計させない──Function ToolsとpandasでCSVを分析する**

https://zenn.dev/tigerone1945/articles/aais-02-function-tools-pandas-csv-analysis

## 第3回

**Structured Outputで分析結果を安定させる──Pydanticで「文章」を「データ」に変える**

https://zenn.dev/tigerone1945/articles/aais-03-structured-output-pydantic

## 第4回

**StreamlitでAIデータ分析AgentのWeb画面を作る──CLIから使えるApplicationへ**

https://zenn.dev/tigerone1945/articles/aais-04-streamlit-web-ui

## Kindle：AI時代の仕様駆動開発

**『AI時代の仕様駆動開発: Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書』**

https://www.amazon.co.jp/dp/B0HBQ32KCQ

## Kindle：AIエージェント開発のためのケーススタディ設計

**『AIエージェント開発のためのケーススタディ設計: As-Is / To-BeからSystem Specification・Kiro Specへつなぐ業務設計』**

https://www.amazon.co.jp/dp/B0HJMM4JF7

## AIエージェント実践入門 Kindle

**2026年9月末公開予定**

業務課題からSystem Specification、SDD、Agent Design、OpenAI Agents SDK、Evaluationまでを体系的に扱う予定です。

---

# 参考

OpenAI Agents SDK 公式ドキュメント：

https://openai.github.io/openai-agents-python/

Sessions：

https://openai.github.io/openai-agents-python/sessions/

Tracing：

https://openai.github.io/openai-agents-python/tracing/

OpenAI Trace Viewer：

https://platform.openai.com/traces

今回の実装では、OpenAI Agents SDKの`SQLiteSession`を使って会話履歴を保持し、`Runner.run_sync(..., session=session)`で複数Run間の文脈を引き継いでいます。

また、TracingはAgents SDKに標準で組み込まれており、Agent実行、Model generation、Function Tool callなどをTraceとして確認できます。
