---
title: "第7回 Human-in-the-LoopでAIエージェントに「人の承認」を入れる"
emoji: "🙋"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - HumanInTheLoop
  - Streamlit
  - Python
published: true
published_at: "2026-10-09 07:00"
---

# はじめに

第6回ではGuardrailsを追加し、

```text
User
↓
Input Guardrail
↓
Agent
```

という安全境界を作りました。

これによって、

> **このApplicationで扱ってよい質問か**

を別の仕組みで判定し、問題があればAgentの実行を止められるようになりました。

しかし、業務AIエージェントでは、

```text
質問自体は問題ない
```

場合でも、

```text
その処理をAIだけで実行してよいとは限らない
```

ケースがあります。

たとえば、

- 注文キャンセル
- 顧客へのメール送信
- 返金
- 審査結果の確定
- レビュー依頼の登録

などです。

そこで第7回では、

> **Human-in-the-Loop（HITL）で、Sensitive Toolの実行前に人の承認を入れる**

ところまで進めます。

---

# 今回のゴール

今回のCSVには、

```text
review_flag
```

があります。

値は、

```text
通常
要レビュー
```

です。

今回は、Agentが要レビュー対象を分析したあと、

> **レビュー依頼を登録する**

Toolを追加します。

ただし、このToolは自動実行しません。

```text
Agent
↓
create_review_request
↓
Approval Required
↓
Human
├─ Approve → 実行
└─ Reject  → 実行しない
```

とします。

---

# Human-in-the-Loopとは

Human-in-the-Loopとは、

> **AIの処理フローの途中に人の判断を入れる設計**

です。

AIエージェントのゴールは、

```text
すべて自動化すること
```

ではありません。

業務では、

```text
Low Risk
→ 自動

High Risk
→ Human Review
```

と分ける方が適切なケースがあります。

---

# GuardrailsとHITLの違い

第6回のGuardrailsは、

```text
やってはいけない
→ Stop
```

でした。

今回のHITLは、

```text
実行してよいか不明
→ Humanへ確認
```

です。

整理すると、

```text
Guardrail
＝禁止境界

HITL
＝承認境界
```

です。

---

# 今回追加するTool

今回は、

```text
create_review_request()
```

を追加します。

このToolは、レビュー依頼を

```text
data/review_requests.csv
```

へ追記します。

つまり、読み取り専用だったこれまでと違い、

> **外部状態を変更するTool**

です。

だからこそ、Human Approvalを入れる意味があります。

---

# フォルダー構成

```text
ai-agent-practice/
├── data/
│   ├── ai_agent_practice_sales.csv
│   ├── conversations.db
│   └── review_requests.csv
├── .env
├── .gitignore
├── agent_service.py
├── app.py
├── pyproject.toml
└── uv.lock
```

`review_requests.csv`は初回実行時に作成します。

---

# 承認が必要なToolを作る

OpenAI Agents SDKでは、Function Toolへ、

```python
needs_approval=True
```

を指定できます。

```python
from agents.decorators import tool


@tool(needs_approval=True)
async def create_review_request(
    order_id: str,
    reason: str,
) -> str:
    ...
```

これによって、AgentがこのToolを呼ぼうとすると、

```text
Tool Call
↓
Pause
↓
Approval待ち
```

になります。

---

# `create_review_request()`を実装する

```python
import csv
from datetime import datetime
from pathlib import Path


REVIEW_REQUESTS_PATH = Path(
    "data/review_requests.csv"
)


@tool(needs_approval=True)
async def create_review_request(
    order_id: str,
    reason: str,
) -> str:
    """
    注文IDをレビュー依頼として登録する。
    このToolは人の承認後にのみ実行する。
    """

    file_exists = REVIEW_REQUESTS_PATH.exists()

    with REVIEW_REQUESTS_PATH.open(
        "a",
        newline="",
        encoding="utf-8-sig",
    ) as f:
        writer = csv.writer(f)

        if not file_exists:
            writer.writerow(
                [
                    "created_at",
                    "order_id",
                    "reason",
                ]
            )

        writer.writerow(
            [
                datetime.now().isoformat(
                    timespec="seconds"
                ),
                order_id,
                reason,
            ]
        )

    return (
        f"{order_id}をレビュー依頼として"
        "登録しました。"
    )
```

---

# AgentへToolを追加する

```python
agent = Agent(
    name="売上分析アシスタント",
    instructions="""
あなたは売上データ分析を支援する
AIアシスタントです。

分析には既存のFunction Toolを使用してください。

ユーザーが明示的にレビュー依頼の登録を求めた場合だけ、
create_review_requestを使用してください。

レビュー依頼の登録は人の承認が必要です。
""",
    tools=[
        get_sales_summary,
        get_sales_by_category,
        get_sales_by_region,
        create_review_request,
    ],
)
```

重要なのは、

> **分析しただけで勝手にレビュー依頼を登録しない**

ことです。

---

# Approval Flowはどう動くのか

`needs_approval=True`のToolが呼ばれると、Runはその場でPauseします。

結果には、

```python
result.interruptions
```

として承認待ちのTool Callが入ります。

流れは、

```text
Runner
↓
Agent
↓
Tool Call
↓
needs_approval
↓
interruptions
↓
Pause
```

です。

---

# RunStateへ変換する

PauseしたRunは、

```python
state = result.to_state()
```

で`RunState`へ変換できます。

このStateに対して、

```python
state.approve(interruption)
```

または、

```python
state.reject(interruption)
```

を行います。

そのあと、

```python
result = await Runner.run(
    agent,
    state,
)
```

として元のRunを再開します。

---

# 最小のApproval例

まずCLIで仕組みだけ確認すると、次のようになります。

```python
import asyncio

from agents import Agent, Runner


async def main() -> None:
    result = await Runner.run(
        agent,
        """
注文 O0001 をレビュー対象として登録してください。
理由は、高額注文の確認です。
""",
    )

    while result.interruptions:
        state = result.to_state()

        for interruption in result.interruptions:
            print(
                "承認対象:",
                interruption.name,
            )
            print(
                "引数:",
                interruption.arguments,
            )

            answer = input(
                "承認しますか？ [y/N]: "
            ).strip().lower()

            if answer in {
                "y",
                "yes",
            }:
                state.approve(
                    interruption
                )
            else:
                state.reject(
                    interruption
                )

        result = await Runner.run(
            agent,
            state,
        )

    print(result.final_output)


asyncio.run(main())
```

---

# Approveした場合

```text
Agent
↓
create_review_requestを要求
↓
Pause
↓
Human: Approve
↓
Tool実行
↓
review_requests.csvへ追記
↓
Agent再開
```

となります。

---

# Rejectした場合

```text
Agent
↓
create_review_requestを要求
↓
Pause
↓
Human: Reject
↓
Toolは実行されない
↓
Agent再開
```

となります。

つまり、

> **AIは実行したいと提案できるが、最終的な実行権限は人が持つ**

設計です。

---

# なぜCSVへの追記を承認対象にしたのか

これまでのToolは、

```text
CSVを読む
集計する
説明する
```

だけでした。

つまりRead-onlyです。

今回のToolは、

```text
review_requests.csvを書き換える
```

ため、副作用があります。

業務システムでは、

```text
Read
```

と、

```text
Write / Action
```

を分けて考えることが重要です。

---

# すべてのToolを承認対象にしない

たとえば、

```text
get_sales_by_region
```

まで毎回承認を要求すると、操作性が悪くなります。

基本的には、

```text
Read-only
→ 自動実行

Side Effect / High Risk
→ Approval
```

のように分ける方が自然です。

---

# 条件付きApprovalもできる

`needs_approval`には、常に`True`だけでなく、関数も指定できます。

たとえば、

```text
一定条件だけ承認が必要
```

という設計も可能です。

今回の教材では、

> HITLの基本フローを明確にする

ため、`needs_approval=True`とします。

---

# SessionsとHITLを併用する場合

第5回でSessionsを使っています。

ApprovalでRunをPauseし、再開する場合でも、同じSessionを使い続けることが重要です。

概念的には、

```text
Session
↓
Run
↓
Interruption
↓
Human Approval
↓
同じRunをResume
↓
同じSessionへ履歴追加
```

です。

---

# Streamlitではどう設計するか

Streamlit UIでは、

```text
承認待ちTool
Tool名
Arguments
```

を表示し、

```text
[承認]
[却下]
```

ボタンを出す設計にできます。

ただし、Pauseした`RunState`を画面操作の間も保持する必要があります。

実運用では、

```text
RunStateをSerialize
↓
Session State / DBへ保存
↓
承認後に復元
↓
Resume
```

という構成が必要になります。

この第7回では、まずSDKのApproval Flowそのものを理解することを優先します。

---

# HITLは「AIの能力不足対策」だけではない

Human Reviewは、

> AIの精度が低いから人が確認する

だけではありません。

たとえAIの判断が高精度でも、

```text
金銭
契約
顧客通知
審査
削除
更新
```

などでは、

> **責任上、人の承認が必要**

というケースがあります。

HITLは精度対策ではなく、

> **Governance設計**

でもあります。

---

# 第6回との違い

第6回：

```text
禁止
↓
Guardrail
↓
Stop
```

第7回：

```text
実行候補
↓
Approval
↓
Human Decision
├─ Approve
└─ Reject
```

ここで、

> **止める**

から、

> **人へ戻す**

へ進みました。

---

# 第7回で理解しておきたい5つのこと

```text
1. needs_approval
Sensitive Toolを承認対象にする

2. interruptions
承認待ちTool Callを取得する

3. RunState
PauseしたRunを保持する

4. approve / reject
人が実行可否を決める

5. Resume
同じRunを途中から再開する
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Guardrails | ○ | 第6回 |
| HITL | ○ | 第7回 |
| RunState永続化 | △ | 概念のみ |
| 業務データSQLite化 | × | 第8回 |
| Multi-table | × | 第9回 |
| Multi-Agent | × | 第10回 |

次回は、これまでCSVファイルだった売上データ自体をSQLiteへ移します。

---

# まとめ

今回は、Human-in-the-Loopを追加しました。

構成は、

```text
Agent
↓
Sensitive Tool
↓
Approval Required
↓
Human
├─ Approve
└─ Reject
```

です。

中心メッセージは、

> **AIができることと、AIに実行させてよいことは別**

です。

Read-onlyな分析Toolは自動化しつつ、

```text
状態を変更する処理
```

にはHuman Approvalを入れる。

これが業務AIエージェントの重要な設計パターンです。

---

# 次回予告

## 第8回：CSVからSQLiteへ──AIエージェントの業務データを永続化する

次回は、

```text
CSV
↓
SQLite
```

へ移行します。

これまでpandasでCSVを直接読んでいたToolを、

```text
Agent
↓
Function Tool
↓
SQLite
```

へ変更します。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

OpenAI Agents SDK Human-in-the-loop：

https://openai.github.io/openai-agents-python/human_in_the_loop/

公式ドキュメントでは、`needs_approval`、`interruptions`、`RunState`、`approve()` / `reject()`、Runの再開というApproval Flowが説明されています。
