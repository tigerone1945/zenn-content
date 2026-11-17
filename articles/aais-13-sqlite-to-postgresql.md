---
title: "第13回 SQLiteからPostgreSQLへ──PoCのDBを本番構成へ近づける"
emoji: "🐘"
type: "tech"
topics:
  - AIエージェント
  - PostgreSQL
  - FastAPI
  - Docker
  - Python
published: true
---

# はじめに

第12回では、売上分析ApplicationをDocker化しました。

構成は、

```text
Docker Compose
├─ ui
│  └─ Streamlit
└─ api
   └─ FastAPI + Agent
```

です。

これで、

> 「自分のPCでは動く」

状態から、

> **同じContainer環境で再現できる**

状態へ進みました。

しかし、Data LayerはまだSQLiteです。

SQLiteは学習用・PoC用として非常に便利ですが、本番システムを意識すると、

```text
複数Process
同時接続
Server運用
Backup
権限管理
Cloud DB
```

といった要件が出てきます。

そこで第13回では、

> **SQLiteからPostgreSQLへ移行する**

ところまで進めます。

---

# 今回のゴール

第12回までの、

```text
api Container
↓
SQLite
```

を、

```text
api Container
↓
PostgreSQL Container
```

へ変えます。

Docker Compose全体は、

```text
Docker Compose
├─ ui
│  └─ Streamlit
│
├─ api
│  └─ FastAPI + Agent
│
└─ db
   └─ PostgreSQL
```

となります。

そして、Agent側のToolは、

```text
get_sales_summary
get_sales_by_region
get_sales_by_customer_type
```

のようなInterfaceをなるべく変えずに、Data Layerだけを差し替えます。

---

# なぜPostgreSQLへ移行するのか

SQLiteは、

```text
1ファイル
セットアップが簡単
ローカル開発向き
```

という大きなメリットがあります。

一方、PostgreSQLはDatabase Serverとして動きます。

本番を意識すると、

```text
複数接続
Transaction
Role / Permission
Network越しの接続
Managed DBへの移行
```

といった構成を取りやすくなります。

FastAPI公式ドキュメントでも、SQLiteは簡単な例に向き、本番ApplicationではPostgreSQLのようなDatabase Serverを利用する選択肢が示されています。

---

# 今回の方針

今回は、DBアクセスにSQLAlchemyを使います。

理由は、

> **SQLiteとPostgreSQLの差をConnection層へ寄せやすい**

からです。

構成を、

```text
Agent
↓
Function Tool
↓
Repository
↓
SQLAlchemy
↓
PostgreSQL
```

へ近づけます。

---

# 必要なPackageを追加する

```bash
uv add sqlalchemy
uv add "psycopg[binary]"
```

PostgreSQL Driverとして`psycopg`を利用します。

---

# フォルダー構成

```text
ai-agent-practice/
├── api/
│   └── main.py
├── ui/
│   └── app.py
├── data/
│   └── ai_agent_practice_sales.csv
├── scripts/
│   └── import_csv_to_postgres.py
├── agent_service.py
├── db.py
├── repository.py
├── Dockerfile.api
├── Dockerfile.ui
├── compose.yaml
├── .env
├── pyproject.toml
└── uv.lock
```

---

# `.env`へDB接続情報を追加する

```env
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx

POSTGRES_DB=ai_agent
POSTGRES_USER=app_user
POSTGRES_PASSWORD=change-me

DATABASE_URL=postgresql+psycopg://app_user:change-me@db:5432/ai_agent
```

注意点は、

> **本番ではPasswordを`.env`のまま運用しない**

ことです。

第16回でSecrets Managerへ移します。

---

# `compose.yaml`へPostgreSQLを追加する

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"
        ]
      interval: 5s
      timeout: 5s
      retries: 10

  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      DATABASE_URL: ${DATABASE_URL}
    depends_on:
      db:
        condition: service_healthy
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

volumes:
  postgres_data:
```

---

# PostgreSQLを起動する

```bash
docker compose up -d db
```

状態を確認します。

```bash
docker compose ps
```

`db`が`healthy`になれば準備完了です。

---

# Database Connectionを共通化する

`db.py`を作ります。

```python
import os

from sqlalchemy import create_engine


DATABASE_URL = os.environ["DATABASE_URL"]

engine = create_engine(
    DATABASE_URL,
    pool_pre_ping=True,
)
```

`pool_pre_ping=True`を指定すると、Connection Poolから取得した接続が生きているか確認しやすくなります。

---

# CSVをPostgreSQLへ取り込む

`scripts/import_csv_to_postgres.py`を作ります。

```python
from pathlib import Path

import pandas as pd

from db import engine


CSV_PATH = Path(
    "data/ai_agent_practice_sales.csv"
)


def main() -> None:
    df = pd.read_csv(CSV_PATH)

    customers = (
        df[
            [
                "customer_id",
                "customer_name",
                "customer_type",
            ]
        ]
        .drop_duplicates()
        .sort_values("customer_id")
    )

    products = (
        df[
            [
                "product_id",
                "product_name",
                "category",
            ]
        ]
        .drop_duplicates()
        .sort_values("product_id")
    )

    orders = df[
        [
            "order_id",
            "order_date",
            "customer_id",
            "product_id",
            "unit_price",
            "quantity",
            "discount_rate",
            "subtotal",
            "total_amount",
            "sales_amount",
            "region",
            "salesperson",
            "status",
            "review_flag",
        ]
    ].copy()

    orders["order_date"] = pd.to_datetime(
        orders["order_date"]
    )

    customers.to_sql(
        "customers",
        engine,
        if_exists="replace",
        index=False,
    )

    products.to_sql(
        "products",
        engine,
        if_exists="replace",
        index=False,
    )

    orders.to_sql(
        "orders",
        engine,
        if_exists="replace",
        index=False,
    )

    print(
        f"{len(orders)}件を"
        "PostgreSQLへ取り込みました。"
    )


if __name__ == "__main__":
    main()
```

---

# Importを実行する

Compose全体を起動します。

```bash
docker compose up -d
```

API Container内からScriptを実行する場合は、

```bash
docker compose exec api \
  python scripts/import_csv_to_postgres.py
```

とします。

---

# Repositoryを作る

DBアクセスをAgent Toolからさらに分離します。

`repository.py`：

```python
from sqlalchemy import text

from db import engine


def fetch_sales_summary() -> dict:
    sql = text(
        """
        SELECT
            COUNT(*) AS order_count,
            SUM(sales_amount) AS total_sales,
            SUM(
                CASE
                    WHEN status = 'キャンセル'
                    THEN 1
                    ELSE 0
                END
            ) AS cancelled_count,
            SUM(
                CASE
                    WHEN review_flag = '要レビュー'
                    THEN 1
                    ELSE 0
                END
            ) AS review_count
        FROM orders
        """
    )

    with engine.connect() as conn:
        row = conn.execute(
            sql
        ).mappings().one()

    return dict(row)
```

---

# 地域別売上Repository

```python
def fetch_sales_by_region() -> list[dict]:
    sql = text(
        """
        SELECT
            region,
            SUM(sales_amount) AS sales
        FROM orders
        GROUP BY region
        ORDER BY sales DESC
        """
    )

    with engine.connect() as conn:
        rows = conn.execute(
            sql
        ).mappings().all()

    return [
        dict(row)
        for row in rows
    ]
```

---

# 顧客種別別売上Repository

```python
def fetch_sales_by_customer_type() -> list[dict]:
    sql = text(
        """
        SELECT
            c.customer_type,
            SUM(o.sales_amount) AS sales
        FROM orders AS o
        JOIN customers AS c
          ON o.customer_id = c.customer_id
        GROUP BY c.customer_type
        ORDER BY sales DESC
        """
    )

    with engine.connect() as conn:
        rows = conn.execute(
            sql
        ).mappings().all()

    return [
        dict(row)
        for row in rows
    ]
```

---

# Function ToolはRepositoryを呼ぶだけにする

```python
from agents.decorators import tool

from repository import (
    fetch_sales_by_customer_type,
    fetch_sales_by_region,
    fetch_sales_summary,
)


@tool
def get_sales_summary() -> str:
    """
    売上全体の基本集計を返す。
    """

    row = fetch_sales_summary()

    return f"""
総レコード数: {row["order_count"]}
売上合計: {int(row["total_sales"]):,}円
キャンセル件数: {row["cancelled_count"]}
要レビュー件数: {row["review_count"]}
""".strip()
```

このように、

```text
Tool
↓
Repository
↓
Database
```

と責務を分けます。

---

# なぜRepositoryを挟むのか

第12回までは、

```text
Tool
↓
SQL
```

でも動きます。

しかし本番化すると、

```text
Connection
Transaction
Retry
DB変更
Test
```

の責務が増えてきます。

そこで、

```text
Agent Tool
＝AgentとのInterface

Repository
＝Data Access

Database
＝Storage
```

と分けます。

---

# SQLiteからPostgreSQLへ変えてもAgentは変えない

重要なのはここです。

Agentは、

```text
get_sales_summary
get_sales_by_region
```

というTool名だけ知っています。

その裏側が、

```text
SQLite
```

から、

```text
PostgreSQL
```

へ変わっても、Agent Instructionsは大きく変更しません。

これが、

> **責務分離の効果**

です。

---

# Migrationをどう考えるか

今回の教材では、CSVからTableを再作成しています。

しかし本番システムでは、

```text
ALTER TABLE
Schema Version
Migration History
Rollback
```

が必要になります。

その場合はAlembicなどのMigration Toolを使う選択肢があります。

今回は、

> **SQLite → PostgreSQLというData Layer移行**

に集中します。

---

# PostgreSQLだから安全になるわけではない

PostgreSQLへ移行しても、

```text
任意SQL
権限過多
Password流出
SQL Injection
```

などの問題は残ります。

第9回と同様に、

```text
目的限定Tool
Parameterized Query
Role制御
```

を続けます。

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
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Manager Agent   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Function Tools   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Repository     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   PostgreSQL     │
└──────────────────┘
```

---

# 第13回で理解しておきたい5つのこと

```text
1. PostgreSQL
PoCのSQLiteからServer型DBへ移行する

2. DATABASE_URL
接続情報をConfigurationとして外へ出す

3. SQLAlchemy
Connection層を抽象化する

4. Repository
Agent ToolからDBアクセスを分離する

5. Interface維持
DBが変わってもAgent Tool名は変えない
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| PostgreSQL | ○ | 第13回 |
| Docker Compose DB | ○ | 第13回 |
| Repository | ○ | 第13回 |
| Terraform | × | 第14回 |
| RDS | × | 第15回 |
| Secrets Manager | × | 第16回 |

---

# まとめ

今回は、

```text
SQLite
↓
PostgreSQL
```

へ移行しました。

同時に、

```text
Agent
↓
Tool
↓
Repository
↓
PostgreSQL
```

という責務分離を導入しました。

中心メッセージは、

> **本番DBへ移行するときも、AgentとStorageを直接結びつけない**

ことです。

---

# 次回予告

## 第14回：TerraformでAIエージェントのAWS基盤を作る──Infrastructure as Code入門

次回は、Applicationの実装からInfrastructureへ進みます。

```text
Docker
PostgreSQL
↓
Terraform
↓
AWS Infrastructure
```

という流れで、

```text
VPC
ECR
ECS
RDS
CloudWatch
```

などをコードで定義します。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

FastAPI SQL Database：

https://fastapi.tiangolo.com/tutorial/sql-databases/

PostgreSQL Docker Official Image：

https://hub.docker.com/_/postgres

SQLAlchemy：

https://docs.sqlalchemy.org/
