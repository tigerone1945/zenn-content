---
title: "第8回 CSVからSQLiteへ──AIエージェントの業務データを永続化する"
emoji: "🗄️"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - SQLite
  - SQL
  - Python
published: true
published_at: "2026-10-13 07:00"
---

# はじめに

第1回から第7回まで、同じ売上データを使ってAIエージェントを育ててきました。

これまでは、

```text
ai_agent_practice_sales.csv
```

をpandasで読み込み、

```text
Agent
↓
Function Tools
↓
pandas
↓
CSV
```

という構成でした。

CSVは学習用データとして非常に扱いやすい形式です。

しかしApplicationとして考えると、

- データ件数が増える
- 条件検索したい
- 一部だけ取り出したい
- 更新したい
- 複数Tableへ発展したい

といった要求が出てきます。

そこで第8回では、

> **売上・受注データをCSVからSQLiteへ移行する**

ところまで進めます。

---

# 今回のゴール

これまでの、

```text
Function Tool
↓
pandas
↓
CSV
```

を、

```text
Function Tool
↓
SQLite
↓
SQL
```

へ変更します。

今回の全体構成は、

```text
Browser
↓
Streamlit
↓
Session
↓
Guardrail
↓
Agent
↓
Function Tools
↓
SQLite
↓
Structured Output
```

です。

---

# 第5回のSQLiteSessionとは別物

ここはもう一度整理します。

第5回では、

```text
conversations.db
```

を作りました。

これは、

> **Agentの会話履歴**

を保存するためのSQLiteです。

今回作るのは、

```text
sales.db
```

です。

こちらは、

> **売上・受注という業務データ**

を保存します。

```text
conversations.db
＝Agent Memory

sales.db
＝Business Data
```

です。

---

# なぜCSVからSQLiteへ移るのか

CSVは、

```text
全件を読み込む
↓
pandasで集計する
```

には便利です。

一方SQLiteなら、

```sql
SELECT
    category,
    SUM(sales_amount)
FROM sales
GROUP BY category;
```

のように、

> **必要なデータをDB側で集計**

できます。

また、第9回では、

```text
orders
customers
products
```

という複数Tableへ発展させます。

SQLite化は、その準備でもあります。

---

# 今回のフォルダー構成

```text
ai-agent-practice/
├── data/
│   ├── ai_agent_practice_sales.csv
│   ├── conversations.db
│   └── sales.db
├── scripts/
│   └── import_csv_to_sqlite.py
├── .env
├── .gitignore
├── agent_service.py
├── app.py
├── pyproject.toml
└── uv.lock
```

---

# まずCSVをSQLiteへ取り込む

`scripts/import_csv_to_sqlite.py`を作ります。

```python
import sqlite3
from pathlib import Path

import pandas as pd


CSV_PATH = Path(
    "data/ai_agent_practice_sales.csv"
)

DB_PATH = Path(
    "data/sales.db"
)


def main() -> None:
    df = pd.read_csv(CSV_PATH)

    with sqlite3.connect(DB_PATH) as conn:
        df.to_sql(
            "sales",
            conn,
            if_exists="replace",
            index=False,
        )

        conn.execute(
            """
            CREATE INDEX IF NOT EXISTS
            idx_sales_category
            ON sales(category)
            """
        )

        conn.execute(
            """
            CREATE INDEX IF NOT EXISTS
            idx_sales_region
            ON sales(region)
            """
        )

        conn.execute(
            """
            CREATE INDEX IF NOT EXISTS
            idx_sales_status
            ON sales(status)
            """
        )

    print(
        f"{len(df)}件を"
        f"{DB_PATH}へ取り込みました。"
    )


if __name__ == "__main__":
    main()
```

---

# Import Scriptを実行する

```bash
uv run python scripts/import_csv_to_sqlite.py
```

成功すると、

```text
300件をdata/sales.dbへ取り込みました。
```

のように表示されます。

これで、

```text
CSV
↓
SQLite sales table
```

への移行ができました。

---

# Tableを確認する

SQLite CLIがある場合は、

```bash
sqlite3 data/sales.db
```

で開けます。

```sql
.tables
```

すると、

```text
sales
```

が確認できます。

件数確認：

```sql
SELECT COUNT(*)
FROM sales;
```

---

# Agent ToolをSQLite版へ変更する

`agent_service.py`では、これまで、

```python
pd.read_csv(...)
```

を使っていました。

今回はSQLiteへ接続します。

```python
import sqlite3
from pathlib import Path


DB_PATH = Path(
    "data/sales.db"
)


def get_connection() -> sqlite3.Connection:
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    return conn
```

---

# Tool 1：売上全体

```python
@tool
def get_sales_summary() -> str:
    """
    売上データ全体の基本集計を返す。
    """

    with get_connection() as conn:
        row = conn.execute(
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
            FROM sales
            """
        ).fetchone()

    return f"""
総レコード数: {row["order_count"]}
売上合計: {int(row["total_sales"]):,}円
キャンセル件数: {row["cancelled_count"]}
要レビュー件数: {row["review_count"]}
""".strip()
```

---

# Tool 2：カテゴリ別売上

```python
@tool
def get_sales_by_category() -> str:
    """
    商品カテゴリ別の売上を集計する。
    """

    with get_connection() as conn:
        rows = conn.execute(
            """
            SELECT
                category,
                SUM(sales_amount) AS sales
            FROM sales
            GROUP BY category
            ORDER BY sales DESC
            """
        ).fetchall()

    return "\n".join(
        f'{row["category"]}: '
        f'{int(row["sales"]):,}円'
        for row in rows
    )
```

---

# Tool 3：地域別売上

```python
@tool
def get_sales_by_region() -> str:
    """
    地域別の売上を集計する。
    """

    with get_connection() as conn:
        rows = conn.execute(
            """
            SELECT
                region,
                SUM(sales_amount) AS sales
            FROM sales
            GROUP BY region
            ORDER BY sales DESC
            """
        ).fetchall()

    return "\n".join(
        f'{row["region"]}: '
        f'{int(row["sales"]):,}円'
        for row in rows
    )
```

---

# Tool 4：要レビュー注文を確認する

第7回でHITLを扱ったので、今回はレビュー対象をDBから検索するToolも追加します。

```python
@tool
def get_review_orders(
    limit: int = 10,
) -> str:
    """
    要レビュー注文を最大limit件返す。
    """

    safe_limit = max(
        1,
        min(limit, 50),
    )

    with get_connection() as conn:
        rows = conn.execute(
            """
            SELECT
                order_id,
                customer_name,
                product_name,
                total_amount,
                status,
                review_flag
            FROM sales
            WHERE review_flag = '要レビュー'
            ORDER BY total_amount DESC
            LIMIT ?
            """,
            (safe_limit,),
        ).fetchall()

    if not rows:
        return "要レビュー注文はありません。"

    return "\n".join(
        (
            f'{row["order_id"]} | '
            f'{row["customer_name"]} | '
            f'{row["product_name"]} | '
            f'{int(row["total_amount"]):,}円 | '
            f'{row["status"]}'
        )
        for row in rows
    )
```

---

# Parameterized Queryを使う

最後のToolでは、

```sql
LIMIT ?
```

とし、

```python
(safe_limit,)
```

を渡しています。

SQLへ値を直接文字列結合せず、

> **Parameterを分離する**

のが基本です。

今後条件検索を増やす場合も、この考え方を使います。

---

# 任意SQL Toolはまだ作らない

ここで、

```text
AgentにSQLを書かせて、
そのSQLをそのまま実行すればよいのでは？
```

と思うかもしれません。

このシリーズでは、まだそれは行いません。

理由は、

```text
DELETE
UPDATE
危険なJOIN
大量取得
意図しないQuery
```

などを防ぎにくいからです。

今回は、

```text
get_sales_summary
get_sales_by_category
get_sales_by_region
get_review_orders
```

のような、

> **目的限定Tool**

を使います。

---

# CSV版とSQLite版の違い

## CSV版

```text
Tool
↓
pd.read_csv()
↓
DataFrame
↓
groupby
```

## SQLite版

```text
Tool
↓
sqlite3
↓
SELECT
↓
GROUP BY
```

Agentから見るとTool名はほぼ同じです。

つまり、

> **Agent Interfaceを保ったまま、Data Layerを差し替えられる**

構成です。

---

# これは重要な設計ポイント

もしAgentのInstructionsに、

```text
CSVのcategory列をpandasでgroupbyして……
```

と細かく書いていたら、SQLiteへ変えるとInstructionsまで修正が必要になります。

しかし、

```text
get_sales_by_category
```

というTool Interfaceで抽象化しておけば、

```text
CSV
↓
SQLite
```

へ変えてもAgent側への影響を小さくできます。

---

# Repository Layerへの第一歩

今回の構成は、

```text
Agent
↓
Tool
↓
SQLite
```

です。

さらに本格的にするなら、

```text
Agent
↓
Tool
↓
Repository
↓
Database
```

と分けられます。

第13回でPostgreSQLへ移行するとき、この考え方が重要になります。

---

# Streamlit側はほとんど変えなくてよい

UIから見ると、

```python
run_sales_agent(question, session_id)
```

を呼ぶだけです。

CSVがSQLiteになっても、

```text
Streamlit UI
```

は基本的に同じです。

これは責務を分離してきた効果です。

---

# 今回のArchitecture

```text
┌──────────────────────┐
│       Browser        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      Streamlit       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      Guardrail       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│       Session        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│        Agent         │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Function Tools     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│       SQLite         │
│      sales.db        │
└──────────────────────┘
```

---

# 第1〜8回の成長

```text
第1回
Agent / Runner
↓
第2回
Function Tools + pandas
↓
第3回
Structured Output
↓
第4回
Streamlit
↓
第5回
Sessions / Tracing
↓
第6回
Guardrails
↓
第7回
Human-in-the-Loop
↓
第8回
SQLite
```

第8回で、

> **ファイルを読む分析Agent**

から、

> **Databaseを利用するApplication**

へ一段進みました。

---

# 第8回で理解しておきたい5つのこと

```text
1. CSV → SQLite
業務データをDBへ移す

2. SELECT / GROUP BY
DB側で集計する

3. Parameterized Query
値をSQL文字列へ直接埋め込まない

4. Purpose-specific Tool
任意SQLではなく目的限定Toolを作る

5. Interface
AgentからData Layerを隠す
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| CSV → SQLite | ○ | 第8回 |
| 1 Table | ○ | 第8回 |
| 複数Table | × | 第9回 |
| Join Validation | × | 第9回 |
| Multi-Agent | × | 第10回 |
| FastAPI | × | 第11回 |
| PostgreSQL | × | 第13回 |

次回は、現在の1枚の`sales` Tableを、

```text
orders
customers
products
```

へ分けます。

---

# まとめ

今回は、

```text
CSV
↓
SQLite
```

へ移行しました。

これまで、

```text
Tool
↓
pandas
↓
CSV
```

だったData Layerが、

```text
Tool
↓
SQL
↓
SQLite
```

へ変わりました。

一方、Agentから見ると、

```text
get_sales_by_category
get_sales_by_region
```

というTool Interfaceは維持できます。

中心メッセージは、

> **AIエージェントとData Storageを直接結びつけず、Toolという境界を置く**

ことです。

---

# 次回予告

## 第9回：複数Tableを安全に扱う──Join ValidationでAIに自由なSQLを書かせない

次回は、

```text
sales
```

という1 Tableを、

```text
orders
customers
products
```

へ分割します。

そして、

```text
どのTableをJoinするか
どのKeyでJoinするか
Many-to-Manyにならないか
```

を確認する、

> **安全なJoin Workflow**

を作ります。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

Python sqlite3：

https://docs.python.org/3/library/sqlite3.html

SQLite：

https://www.sqlite.org/

次回以降も、Agentへ任意SQLを自由実行させるのではなく、目的限定ToolとValidationを積み上げていきます。
