---
title: "第11回 AIエージェントをFastAPIでAPI化する──UIとAgentの境界を分離する"
emoji: "🌐"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - FastAPI
  - Python
  - API
published: true
published_at: "2026-11-03 07:00"
---

# はじめに

第10回では、売上分析AgentをMulti-Agent化しました。

```text
Manager Agent
├─ Data Quality Agent
├─ Sales Analyst Agent
└─ Business Insight Agent
```

ここまでStreamlitは、Python関数を直接呼んでいます。

```text
Streamlit
↓
run_sales_agent()
↓
Manager Agent
```

小さなApplicationなら、この構成でも動きます。

しかし本番を意識すると、

```text
Frontend
AI Processing
Data Layer
```

を分離したくなります。

そこで第11回では、

> **FastAPIを使ってAIエージェントをWeb API化する**

ところまで進めます。

---

# 今回のゴール

構成を、

```text
Streamlit
↓
Python Function
↓
Agent
```

から、

```text
Streamlit
↓
HTTP
↓
FastAPI
↓
Agent Service
↓
Manager Agent
```

へ変えます。

これによって、

> **UIとAIエージェントを別Applicationとして扱える**

ようになります。

---

# なぜAPI化するのか

API化すると、同じAgentを、

```text
Streamlit
Web Frontend
Mobile App
Batch
別システム
```

から利用できます。

つまり、

```text
AI Agent
```

を、

```text
Application内部の関数
```

から、

```text
Service
```

へ変えることになります。

---

# 今回のフォルダー構成

```text
ai-agent-practice/
├── data/
│   ├── sales.db
│   └── conversations.db
├── api/
│   └── main.py
├── ui/
│   └── app.py
├── agent_service.py
├── .env
├── pyproject.toml
└── uv.lock
```

---

# FastAPIを追加する

```bash
uv add fastapi
uv add "uvicorn[standard]"
uv add httpx
```

`httpx`はStreamlit側からAPIを呼ぶために使います。

---

# Request / Response Schemaを作る

APIでは、入出力を明確にします。

```python
from pydantic import BaseModel


class AnalysisRequest(BaseModel):
    question: str
    session_id: str


class AnalysisResponse(BaseModel):
    analysis_type: str
    result_summary: str
    key_findings: list[str]
    next_action: str
```

---

# FastAPI Applicationを作る

`api/main.py`です。

```python
from fastapi import FastAPI, HTTPException

from agent_service import (
    run_sales_agent_async,
)


app = FastAPI(
    title="AI Sales Analysis API",
    version="1.0.0",
)


class AnalysisRequest(BaseModel):
    question: str
    session_id: str


class AnalysisResponse(BaseModel):
    analysis_type: str
    result_summary: str
    key_findings: list[str]
    next_action: str


@app.get("/health")
async def health() -> dict[str, str]:
    return {
        "status": "ok",
    }


@app.post(
    "/analysis",
    response_model=AnalysisResponse,
)
async def analyze(
    request: AnalysisRequest,
) -> AnalysisResponse:

    if not request.question.strip():
        raise HTTPException(
            status_code=400,
            detail="question is required",
        )

    try:
        result = await run_sales_agent_async(
            question=request.question,
            session_id=request.session_id,
        )

    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail="analysis failed",
        ) from e

    return AnalysisResponse(
        analysis_type=result.analysis_type,
        result_summary=result.result_summary,
        key_findings=result.key_findings,
        next_action=result.next_action,
    )
```

---

# `run_sales_agent_async()`を作る

FastAPIはasyncと相性がよいので、Agent Service側もasync版にします。

```python
async def run_sales_agent_async(
    question: str,
    session_id: str,
) -> SalesAnalysisResult:

    session = SQLiteSession(
        session_id,
        str(SESSION_DB_PATH),
    )

    with trace(
        workflow_name="sales_analysis_api",
        group_id=session_id,
    ):
        result = await Runner.run(
            manager_agent,
            question,
            session=session,
        )

    return result.final_output
```

---

# APIを起動する

```bash
uv run uvicorn api.main:app --reload
```

通常は、

```text
http://127.0.0.1:8000
```

で起動します。

---

# Health Check

ブラウザまたはcurlで、

```bash
curl http://127.0.0.1:8000/health
```

とすると、

```json
{
  "status": "ok"
}
```

が返ります。

---

# Analysis APIを呼ぶ

```bash
curl \
  -X POST \
  http://127.0.0.1:8000/analysis \
  -H "Content-Type: application/json" \
  -d '{
    "question": "カテゴリ別売上を分析してください",
    "session_id": "demo-session"
  }'
```

レスポンス例：

```json
{
  "analysis_type": "category_sales",
  "result_summary": "カテゴリ別売上を分析しました。",
  "key_findings": [
    "売上上位カテゴリがあります",
    "カテゴリ間で売上差があります"
  ],
  "next_action": "上位カテゴリの商品別内訳を確認してください"
}
```

---

# Swagger UIも使える

FastAPIでは、自動でAPIドキュメントが生成されます。

起動後、

```text
http://127.0.0.1:8000/docs
```

へアクセスします。

ここから、

```text
POST /analysis
```

を直接試せます。

---

# StreamlitからHTTPで呼ぶ

第10回までは、

```python
from agent_service import run_sales_agent
```

としていました。

第11回ではAgent Serviceを直接importしません。

`ui/app.py`からHTTPで呼びます。

```python
import httpx
import streamlit as st


API_URL = (
    "http://127.0.0.1:8000"
)


def call_analysis_api(
    question: str,
    session_id: str,
) -> dict:

    response = httpx.post(
        f"{API_URL}/analysis",
        json={
            "question": question,
            "session_id": session_id,
        },
        timeout=60.0,
    )

    response.raise_for_status()

    return response.json()
```

---

# UIからAgentを直接知らなくなる

これは大きな変化です。

第10回：

```text
Streamlit
↓
Agent Service
```

第11回：

```text
Streamlit
↓
HTTP
↓
FastAPI
↓
Agent Service
```

Streamlitは、

```text
Agentが何個あるか
どのToolを使うか
SQLiteがどこにあるか
```

を知る必要がありません。

---

# API Contractという考え方

UIが知るのは、

```text
POST /analysis
```

と、

```json
{
  "question": "...",
  "session_id": "..."
}
```

だけです。

返ってくるのは、

```json
{
  "analysis_type": "...",
  "result_summary": "...",
  "key_findings": [],
  "next_action": "..."
}
```

です。

これが、

> **API Contract**

です。

---

# Structured Outputがここで効く

第3回でStructured Outputを導入しました。

そのため、

```text
Agent Output
↓
Pydantic
↓
FastAPI Response
```

へ自然につながります。

もしAgentが自由文しか返さなければ、APIのResponse Schemaも安定しません。

---

# Error Handling

APIでは、内部Exceptionをそのまま返さない方が安全です。

今回は、

```python
raise HTTPException(
    status_code=500,
    detail="analysis failed",
)
```

としています。

詳細なExceptionは、

```text
Tracing
Application Log
```

側で確認します。

---

# Guardrail違反は400系に分けてもよい

第6回のGuardrailと組み合わせるなら、

```text
入力範囲外
```

は500ではなく、

```text
400 / 422
```

系として扱う設計もできます。

つまり、

```text
System Error
と
User Input Error
```

を分けます。

---

# CORSはいつ必要か

今回のStreamlitとFastAPIが同じ開発環境にある場合、まずは不要です。

しかし将来、

```text
Browser Frontend
↓
FastAPI
```

を別Originで動かすなら、CORS設定が必要になります。

今回はまだ追加しません。

---

# Authenticationもまだ追加しない

API化すると、

```text
誰でも呼べるのか？
```

という問題が出ます。

本番では、

```text
Authentication
Authorization
Rate Limit
```

が必要です。

ただし今回は、

> **API境界を作る**

ことに集中します。

---

# 今回のArchitecture

```text
┌──────────────────┐
│    Streamlit     │
└────────┬─────────┘
         │ HTTP
         ▼
┌──────────────────┐
│     FastAPI      │
│   /analysis      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Agent Service   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Manager Agent   │
└────────┬─────────┘
         │
   ┌─────┼─────┐
   ▼     ▼     ▼
 Data  Sales  Insight
 Agent Agent  Agent
         │
         ▼
      SQLite
```

---

# 第11回で理解しておきたい5つのこと

```text
1. FastAPI
AgentをWeb API化する

2. Request / Response Schema
API Contractを定義する

3. async
Runner.run()とAPI処理をつなぐ

4. HTTP Boundary
UIとAgentを分離する

5. Structured Output
API Responseの安定化に使う
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| FastAPI | ○ | 第11回 |
| HTTP API | ○ | 第11回 |
| Swagger | ○ | 第11回 |
| Docker | × | 第12回 |
| PostgreSQL | × | 第13回 |
| Authentication | × | 今後 |
| AWS | × | 第15回 |

---

# まとめ

今回は、売上分析AgentをFastAPIでAPI化しました。

構成は、

```text
Streamlit
↓
HTTP
↓
FastAPI
↓
Agent
```

です。

中心メッセージは、

> **AIエージェントをApplication内部の関数からServiceへ変える**

ことです。

これによって、UIとAI実装を独立して変更しやすくなります。

---

# 次回予告

## 第12回：AIエージェントをDocker化する──「自分のPCでは動く」から卒業する

次回は、

```text
Streamlit
FastAPI
Agent
SQLite
```

をContainerで動かします。

最終的には、

```text
Docker Compose
├─ api
└─ ui
```

という構成にします。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

FastAPI：

https://fastapi.tiangolo.com/

OpenAI Agents SDK Running Agents：

https://openai.github.io/openai-agents-python/running_agents/
