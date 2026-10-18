---
title: "第9回 複数Tableを安全に扱う──Join ValidationでAIに自由なSQLを書かせない"
emoji: "🔗"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - SQLite
  - SQL
  - Python
published: true
published_at: "2026-10-20 07:00"
---

# はじめに

第8回では、これまでCSVで扱っていた売上・受注データをSQLiteへ移行しました。

構成は、

```text
Agent
↓
Function Tools
↓
SQLite
↓
sales table
```

です。

ただし、現在の`sales` Tableには、

```text
注文
顧客
商品
地域
担当者
売上金額
```

がすべて1つに入っています。

学習用としては分かりやすいですが、業務システムへ近づけるなら、

```text
orders
customers
products
```

のようにTableを分ける方が自然です。

しかし、複数Tableにした瞬間に新しい問題が生まれます。

> **AIに自由にJOINを書かせてよいのか？**

そこで第9回では、複数Tableを安全に扱うために、

```text
list
↓
inspect
↓
suggest join
↓
validate key
↓
preview
↓
aggregate
```

というJoin Validationの流れを作ります。

---

# 今回のゴール

今回の構成は次の通りです。

```text
Agent
↓
Function Tools
↓
SQLite
├─ orders
├─ customers
└─ products
```

そして、

```text
orders.customer_id
↓
customers.customer_id

orders.product_id
↓
products.product_id
```

というJoinだけを安全に許可します。

AIへ、

```sql
SELECT *
FROM ...
JOIN ...
```

を自由生成させるのではなく、

> **許可されたJoinだけをTool経由で実行する**

構成にします。

---

# なぜ1 Tableから分けるのか

現在の`sales` Tableには、

```text
customer_name
customer_type
product_name
category
```

などが注文ごとに繰り返し入っています。

たとえば同じ顧客が20回注文すれば、

```text
customer_name
customer_type
```

も20回繰り返されます。

そこで、

```text
customers
products
orders
```

に分けます。

---

# 今回のTable構成

## customers

```text
customer_id
customer_name
customer_type
```

## products

```text
product_id
product_name
category
```

## orders

```text
order_id
order_date
customer_id
product_id
unit_price
quantity
discount_rate
subtotal
total_amount
sales_amount
region
salesperson
status
review_flag
```

---

# フォルダー構成

```text
ai-agent-practice/
├── data/
│   ├── ai_agent_practice_sales.csv
│   ├── conversations.db
│   └── sales.db
├── scripts/
│   └── normalize_sales_db.py
├── .env
├── .gitignore
├── agent_service.py
├── app.py
├── pyproject.toml
└── uv.lock
```

---

# まず3 Tableへ分割する

`scripts/normalize_sales_db.py`を作ります。

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
    ]

    with sqlite3.connect(DB_PATH) as conn:
        customers.to_sql(
            "customers",
            conn,
            if_exists="replace",
            index=False,
        )

        products.to_sql(
            "products",
            conn,
            if_exists="replace",
            index=False,
        )

        orders.to_sql(
            "orders",
            conn,
            if_exists="replace",
            index=False,
        )

        conn.execute(
            """
            CREATE UNIQUE INDEX IF NOT EXISTS
            idx_customers_customer_id
            ON customers(customer_id)
            """
        )

        conn.execute(
            """
            CREATE UNIQUE INDEX IF NOT EXISTS
            idx_products_product_id
            ON products(product_id)
            """
        )

        conn.execute(
            """
            CREATE UNIQUE INDEX IF NOT EXISTS
            idx_orders_order_id
            ON orders(order_id)
            """
        )

        conn.execute(
            """
            CREATE INDEX IF NOT EXISTS
            idx_orders_customer_id
            ON orders(customer_id)
            """
        )

        conn.execute(
            """
            CREATE INDEX IF NOT EXISTS
            idx_orders_product_id
            ON orders(product_id)
            """
        )

    print(
        "orders / customers / products "
        "を作成しました。"
    )


if __name__ == "__main__":
    main()
```

実行します。

```bash
uv run python scripts/normalize_sales_db.py
```

---

# Joinを「許可リスト」で管理する

今回のポイントです。

任意のTable同士をJoinさせず、許可した関係だけを定義します。

```python
ALLOWED_JOINS = {
    (
        "orders",
        "customers",
    ): (
        "customer_id",
        "customer_id",
    ),
    (
        "orders",
        "products",
    ): (
        "product_id",
        "product_id",
    ),
}
```

これで、

```text
orders ↔ customers
orders ↔ products
```

だけを許可します。

---

# Step 1：Table一覧を確認する

まずToolとしてTable一覧を取得します。

```python
from agents.decorators import tool


@tool
def list_tables() -> str:
    """
    利用可能な業務Table一覧を返す。
    """

    with get_connection() as conn:
        rows = conn.execute(
            """
            SELECT name
            FROM sqlite_master
            WHERE type = 'table'
              AND name IN (
                'orders',
                'customers',
                'products'
              )
            ORDER BY name
            """
        ).fetchall()

    return "\n".join(
        row["name"]
        for row in rows
    )
```

---

# Step 2：Schemaを確認する

次に、Tableの列を確認します。

```python
@tool
def inspect_table(
    table_name: str,
) -> str:
    """
    指定TableのSchemaを返す。
    """

    allowed = {
        "orders",
        "customers",
        "products",
    }

    if table_name not in allowed:
        return "許可されていないTableです。"

    with get_connection() as conn:
        rows = conn.execute(
            f"PRAGMA table_info({table_name})"
        ).fetchall()

    return "\n".join(
        (
            f'{row["name"]} | '
            f'{row["type"]}'
        )
        for row in rows
    )
```

ここではTable名を許可リストで制限しています。

---

# Step 3：Join候補を提案する

```python
@tool
def suggest_join(
    left_table: str,
    right_table: str,
) -> str:
    """
    2つのTable間で許可されたJoin Keyを返す。
    """

    key = (
        left_table,
        right_table,
    )

    reverse_key = (
        right_table,
        left_table,
    )

    if key in ALLOWED_JOINS:
        left_key, right_key = (
            ALLOWED_JOINS[key]
        )

        return (
            f"{left_table}.{left_key} = "
            f"{right_table}.{right_key}"
        )

    if reverse_key in ALLOWED_JOINS:
        right_key, left_key = (
            ALLOWED_JOINS[reverse_key]
        )

        return (
            f"{left_table}.{left_key} = "
            f"{right_table}.{right_key}"
        )

    return "許可されたJoinはありません。"
```

---

# Step 4：Join Keyを検証する

Joinで怖いのは、

> **Many-to-Manyで行数が爆発すること**

です。

そのため、Join前にKeyの重複を確認します。

```python
def count_duplicates(
    conn: sqlite3.Connection,
    table_name: str,
    column_name: str,
) -> int:
    row = conn.execute(
        f"""
        SELECT COUNT(*) AS cnt
        FROM (
            SELECT {column_name}
            FROM {table_name}
            GROUP BY {column_name}
            HAVING COUNT(*) > 1
        )
        """
    ).fetchone()

    return int(row["cnt"])
```

そしてValidation Toolを作ります。

```python
@tool
def validate_join(
    left_table: str,
    right_table: str,
) -> str:
    """
    許可されたJoinについて、
    Keyの重複状態を検証する。
    """

    relation = resolve_join(
        left_table,
        right_table,
    )

    if relation is None:
        return "許可されていないJoinです。"

    left_key, right_key = relation

    with get_connection() as conn:
        left_dup = count_duplicates(
            conn,
            left_table,
            left_key,
        )

        right_dup = count_duplicates(
            conn,
            right_table,
            right_key,
        )

    return (
        f"{left_table}.{left_key} "
        f"重複Key数: {left_dup}\n"
        f"{right_table}.{right_key} "
        f"重複Key数: {right_dup}"
    )
```

---

# Many-to-Manyを警戒する

今回の想定では、

```text
customers.customer_id
products.product_id
```

は一意です。

一方、

```text
orders.customer_id
orders.product_id
```

は複数回登場します。

したがって、

```text
customers 1 : N orders
products  1 : N orders
```

です。

これは正常です。

しかし両側に重複がある場合は、

```text
N : M
```

の可能性があります。

その場合は、自動実行せず警告する方が安全です。

---

# Step 5：Join Previewを出す

実際の集計前に、数件だけPreviewします。

```python
@tool
def preview_join(
    left_table: str,
    right_table: str,
    limit: int = 5,
) -> str:
    """
    許可されたJoinのPreviewを返す。
    """

    relation = resolve_join(
        left_table,
        right_table,
    )

    if relation is None:
        return "許可されていないJoinです。"

    left_key, right_key = relation

    safe_limit = max(
        1,
        min(limit, 20),
    )

    sql = f"""
    SELECT *
    FROM {left_table} AS l
    JOIN {right_table} AS r
      ON l.{left_key} = r.{right_key}
    LIMIT ?
    """

    with get_connection() as conn:
        rows = conn.execute(
            sql,
            (safe_limit,),
        ).fetchall()

    if not rows:
        return "Join結果は0件です。"

    return "\n".join(
        str(dict(row))
        for row in rows
    )
```

---

# Step 6：安全な集計Toolを用意する

たとえば顧客種別別売上なら、

```python
@tool
def get_sales_by_customer_type() -> str:
    """
    顧客種別別の売上を集計する。
    """

    with get_connection() as conn:
        rows = conn.execute(
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
        ).fetchall()

    return "\n".join(
        (
            f'{row["customer_type"]}: '
            f'{int(row["sales"]):,}円'
        )
        for row in rows
    )
```

商品カテゴリ別も、

```python
@tool
def get_sales_by_product_category() -> str:
    """
    商品カテゴリ別売上を集計する。
    """

    with get_connection() as conn:
        rows = conn.execute(
            """
            SELECT
                p.category,
                SUM(o.sales_amount) AS sales
            FROM orders AS o
            JOIN products AS p
              ON o.product_id = p.product_id
            GROUP BY p.category
            ORDER BY sales DESC
            """
        ).fetchall()

    return "\n".join(
        (
            f'{row["category"]}: '
            f'{int(row["sales"]):,}円'
        )
        for row in rows
    )
```

---

# `resolve_join()`を作る

上のToolで使うHelperです。

```python
def resolve_join(
    left_table: str,
    right_table: str,
) -> tuple[str, str] | None:

    direct = (
        left_table,
        right_table,
    )

    reverse = (
        right_table,
        left_table,
    )

    if direct in ALLOWED_JOINS:
        return ALLOWED_JOINS[direct]

    if reverse in ALLOWED_JOINS:
        right_key, left_key = (
            ALLOWED_JOINS[reverse]
        )
        return (
            left_key,
            right_key,
        )

    return None
```

---

# Agentへ与えるTool

```python
agent = Agent(
    name="売上分析アシスタント",
    instructions="""
複数Tableを扱う場合は、
いきなり集計を推測しないでください。

必要に応じて、

1. list_tables
2. inspect_table
3. suggest_join
4. validate_join
5. preview_join
6. 集計Tool

の順で確認してください。

任意SQLを生成して実行してはいけません。
""",
    tools=[
        list_tables,
        inspect_table,
        suggest_join,
        validate_join,
        preview_join,
        get_sales_by_customer_type,
        get_sales_by_product_category,
    ],
)
```

---

# なぜAIに自由なSQLを書かせないのか

自由SQLは便利です。

しかし、

```text
存在しないColumn
誤ったJoin Key
Many-to-Many
過剰取得
UPDATE / DELETE
```

などのリスクがあります。

業務Applicationでは、

> **SQL生成能力**

より、

> **安全に実行できる範囲を設計すること**

が重要です。

---

# 安全なJoin Workflow

今回の中心です。

```text
1. list
Tableを確認

2. inspect
Columnを確認

3. suggest
許可されたJoinを確認

4. validate
Keyの重複を確認

5. preview
数件でJoin結果を確認

6. aggregate
目的限定Toolで集計
```

この順番を覚えておくと、AIにDBを扱わせるときの事故をかなり減らせます。

---

# 今回のArchitecture

```text
Agent
↓
Join Validation Tools
├─ list_tables
├─ inspect_table
├─ suggest_join
├─ validate_join
└─ preview_join
↓
Purpose-specific Tools
├─ get_sales_by_customer_type
└─ get_sales_by_product_category
↓
SQLite
├─ orders
├─ customers
└─ products
```

---

# 第9回で理解しておきたい5つのこと

```text
1. Normalization
1 Tableを役割ごとに分ける

2. Allowed Join
Join関係を許可リスト化する

3. Validation
Join Keyの重複を確認する

4. Preview
本集計前に結果を確認する

5. Purpose-specific Query
自由SQLより目的限定Toolを優先する
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| 複数Table | ○ | 第9回 |
| Join Validation | ○ | 第9回 |
| 自由SQL | × | 採用しない |
| Multi-Agent | × | 第10回 |
| FastAPI | × | 第11回 |
| Docker | × | 第12回 |

---

# まとめ

今回は、SQLiteを3 Tableへ分け、

```text
orders
customers
products
```

を安全に扱うためのJoin Validationを追加しました。

中心メッセージは、

> **AIに自由なSQLを書かせる前に、Joinできる範囲をシステム側で定義する**

です。

複数Tableを扱うときは、

```text
list
↓
inspect
↓
suggest
↓
validate
↓
preview
↓
aggregate
```

の順で確認します。

---

# 次回予告

## 第10回：Agentを役割分担する──Manager PatternでMulti-Agent化する

次回は、1つのAgentにすべてを任せず、

```text
Manager Agent
├─ Data Quality Agent
├─ Sales Analyst Agent
└─ Business Insight Agent
```

へ分けます。

OpenAI Agents SDKの`Agent.as_tool()`を使い、

> **Managerが会話の主導権を持ったまま専門Agentを呼ぶ**

構成を作ります。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

OpenAI Agents SDK Tools：

https://openai.github.io/openai-agents-python/tools/

Python sqlite3：

https://docs.python.org/3/library/sqlite3.html
