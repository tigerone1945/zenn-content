---
title: "第12回 AIエージェントをDocker化する──「自分のPCでは動く」から卒業する"
emoji: "🐳"
type: "tech"
topics:
  - AIエージェント
  - Docker
  - FastAPI
  - Streamlit
  - Python
published: true
published_at: "2026-11-10 07:00"
---

# はじめに

第11回では、売上分析AgentをFastAPIでAPI化しました。

構成は、

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
↓
SQLite
```

です。

これでUIとAgent Processingを分離できました。

しかし、まだ実行環境はローカルPCに依存しています。

```text
PythonのVersion
uvの環境
OS
Library
起動コマンド
```

が人によって違えば、

> 「自分のMacでは動くけれど、別環境では動かない」

という問題が起こります。

そこで第12回では、

> **Dockerで実行環境そのものをContainer化する**

ところまで進めます。

---

# 今回のゴール

今回は、

```text
FastAPI
Streamlit
```

を別Containerにします。

最終構成は、

```text
Docker Compose
├─ api
│  └─ FastAPI + Agent
│
└─ ui
   └─ Streamlit
```

です。

SQLiteとCSVはVolumeとして共有します。

---

# なぜDockerを使うのか

Dockerを使うと、

```text
Python Version
Dependency
起動Command
OS Layer
```

をDockerfileへまとめられます。

つまり、

> **Applicationだけでなく、動作環境もコード化する**

ことができます。

---

# 今回のフォルダー構成

```text
ai-agent-practice/
├── api/
│   └── main.py
├── ui/
│   └── app.py
├── data/
│   ├── sales.db
│   └── conversations.db
├── agent_service.py
├── Dockerfile.api
├── Dockerfile.ui
├── compose.yaml
├── .dockerignore
├── .env
├── pyproject.toml
└── uv.lock
```

---

# `.dockerignore`を作る

まずDocker build contextへ不要なものを送らないようにします。

```dockerignore
.git
.venv
__pycache__
*.pyc
.env
```

`.env`をImageへ含めないのが重要です。

---

# API用Dockerfile

`Dockerfile.api`を作ります。

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY pyproject.toml uv.lock ./

RUN pip install --no-cache-dir uv \
    && uv sync --frozen --no-dev

COPY agent_service.py ./
COPY api ./api
COPY data ./data

ENV PATH="/app/.venv/bin:$PATH"

EXPOSE 8000

CMD [
  "uvicorn",
  "api.main:app",
  "--host",
  "0.0.0.0",
  "--port",
  "8000"
]
```

---

# UI用Dockerfile

`Dockerfile.ui`です。

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY pyproject.toml uv.lock ./

RUN pip install --no-cache-dir uv \
    && uv sync --frozen --no-dev

COPY ui ./ui

ENV PATH="/app/.venv/bin:$PATH"

EXPOSE 8501

CMD [
  "streamlit",
  "run",
  "ui/app.py",
  "--server.address=0.0.0.0",
  "--server.port=8501"
]
```

---

# Container間ではlocalhostを使わない

第11回のStreamlitは、

```python
API_URL = (
    "http://127.0.0.1:8000"
)
```

でした。

Docker Composeでは、UI Containerから見た`localhost`は、

> **UI Container自身**

です。

FastAPI Containerではありません。

そのため、

```text
api
```

というService名でアクセスします。

```python
import os


API_URL = os.getenv(
    "API_URL",
    "http://localhost:8000",
)
```

Docker Composeでは、

```text
http://api:8000
```

を設定します。

---

# `compose.yaml`を作る

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
    volumes:
      - ./data:/app/data
    ports:
      - "8000:8000"

  ui:
    build:
      context: .
      dockerfile: Dockerfile.ui
    environment:
      API_URL: http://api:8000
    depends_on:
      - api
    ports:
      - "8501:8501"
```

---

# API KeyはEnvironment Variableで渡す

Dockerfileへ、

```text
OPENAI_API_KEY
```

を直接書いてはいけません。

`.env`に、

```env
OPENAI_API_KEY=sk-xxxxxxxx
```

を置き、Composeから、

```yaml
OPENAI_API_KEY: ${OPENAI_API_KEY}
```

として渡します。

---

# Buildする

```bash
docker compose build
```

これで、

```text
api image
ui image
```

が作られます。

---

# 起動する

```bash
docker compose up
```

ブラウザから、

```text
http://localhost:8501
```

へアクセスします。

FastAPIは、

```text
http://localhost:8000
```

です。

Swagger UIは、

```text
http://localhost:8000/docs
```

です。

---

# Health Checkを確認する

```bash
curl http://localhost:8000/health
```

で、

```json
{
  "status": "ok"
}
```

が返れば、API Containerは動いています。

---

# Container同士の通信

構成は、

```text
Browser
↓
localhost:8501
↓
ui Container
↓
http://api:8000
↓
api Container
↓
Agent
```

です。

Browserから`api`というHost名は見えません。

`api`はDocker Compose Network内のService名です。

---

# VolumeでDataを保持する

今回は、

```yaml
volumes:
  - ./data:/app/data
```

としています。

これによって、

```text
sales.db
conversations.db
```

はContainer削除後もHost側に残ります。

ただし、この構成はまだ開発用です。

本番ではSQLiteではなく、後のPostgreSQLへ移行します。

---

# `depends_on`は「起動完了待ち」ではない

```yaml
depends_on:
  - api
```

は、Container起動順の助けにはなります。

ただし、

> **FastAPIがReadyになるまで完全に待つ**

ことを保証するものではありません。

本番に近づけるなら、

```text
Healthcheck
Retry
```

も必要です。

今回はDocker Composeの基本に集中します。

---

# Logを見る

```bash
docker compose logs -f
```

でContainer Logを確認できます。

APIだけなら、

```bash
docker compose logs -f api
```

UIだけなら、

```bash
docker compose logs -f ui
```

です。

---

# Containerへ入る

問題がある場合、

```bash
docker compose exec api sh
```

でAPI Containerへ入れます。

たとえば、

```bash
ls -la
ls -la data
```

で、必要なファイルが見えるか確認できます。

---

# 停止する

```bash
docker compose down
```

で停止します。

VolumeとしてBind Mountしている`./data`はHost側に残ります。

---

# Docker化で何が変わったのか

第11回：

```text
Mac
├─ Streamlit Process
└─ FastAPI Process
```

第12回：

```text
Docker
├─ ui Container
└─ api Container
```

Applicationの実行環境が明確になりました。

---

# Dockerは本番環境そのものではない

Docker化すると、

> 「これで本番対応完了」

と思いがちです。

しかし、まだ、

```text
SQLite
Secrets
Authentication
Scaling
Monitoring
Deployment
```

などが残っています。

Dockerは、

> **再現可能な実行単位を作る**

ための段階です。

---

# なぜAPIとUIを別Containerにしたのか

1 Containerへ、

```text
Streamlit
FastAPI
```

を入れることもできます。

しかし今回は、第11回で作った、

```text
UI Boundary
API Boundary
```

を維持します。

```text
ui
＝Frontend

api
＝Backend / Agent Service
```

と分離した方が、AWSへ進むときにも分かりやすくなります。

---

# 今回のArchitecture

```text
┌──────────────────────────────┐
│        Docker Compose        │
│                              │
│  ┌────────────────────────┐  │
│  │      ui Container      │  │
│  │      Streamlit         │  │
│  └──────────┬─────────────┘  │
│             │ HTTP           │
│             ▼                │
│  ┌────────────────────────┐  │
│  │      api Container     │  │
│  │ FastAPI + Agent SDK    │  │
│  └──────────┬─────────────┘  │
│             │                │
│             ▼                │
│  ┌────────────────────────┐  │
│  │       ./data           │  │
│  │ SQLite / Session DB    │  │
│  └────────────────────────┘  │
└──────────────────────────────┘
```

---

# 第12回で理解しておきたい5つのこと

```text
1. Dockerfile
実行環境を定義する

2. Container
Applicationを再現可能にする

3. Docker Compose
複数Containerをまとめて動かす

4. Service Name
Container間通信ではlocalhostを使わない

5. Environment Variable / Volume
SecretsとDataをImageから分離する
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| FastAPI Container | ○ | 第12回 |
| Streamlit Container | ○ | 第12回 |
| Docker Compose | ○ | 第12回 |
| SQLite | ○ | まだ利用 |
| PostgreSQL | × | 第13回 |
| Terraform | × | 第14回 |
| AWS Deploy | × | 第15回 |
| CloudWatch | × | 第16回 |

---

# まとめ

今回は、AI売上分析ApplicationをDocker化しました。

構成は、

```text
Docker Compose
├─ ui
│  └─ Streamlit
└─ api
   └─ FastAPI + Agent
```

です。

中心メッセージは、

> **「コードがある」だけでなく、「同じ環境で動かせる」状態にする**

ことです。

これで、次はData Layerを本番向けに育てられます。

---

# 次回予告

## 第13回：SQLiteからPostgreSQLへ──PoCのDBを本番構成へ近づける

次回は、

```text
SQLite
↓
PostgreSQL
```

へ移行します。

Docker ComposeへPostgreSQL Containerを追加し、

```text
api
↓
PostgreSQL
```

という構成へ発展させます。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

Docker：

https://docs.docker.com/

Docker Compose：

https://docs.docker.com/compose/

FastAPI Docker deployment：

https://fastapi.tiangolo.com/deployment/docker/
