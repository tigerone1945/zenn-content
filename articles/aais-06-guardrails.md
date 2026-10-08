---
title: "第6回 GuardrailsでAIエージェントの「やってはいけない」を設計する"
emoji: "🛡️"
type: "tech"
topics:
  - AIエージェント
  - OpenAIAgentsSDK
  - Guardrails
  - Streamlit
  - Python
published: true
published_at: "2026-10-06 07:00"
---

# はじめに

第5回では、売上分析AgentへSessionsとTracingを追加しました。

ここまでで、

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
```

という流れができています。

さらにTracingによって、

```text
どのAgentが動いたか
どのToolが呼ばれたか
どのように実行されたか
```

も確認できるようになりました。

しかし、業務用AIエージェントにはもう1つ重要な問題があります。

それは、

> **Agentに「やってはいけないこと」をどう守らせるか**

です。

Instructionsへ、

```text
売上分析以外の質問には答えないでください
```

と書くだけでも一定の制御はできます。

ただし、業務Applicationでは、

> **振る舞いのお願い**

と、

> **システムとしての安全境界**

を分けて考える必要があります。

そこで第6回では、OpenAI Agents SDKのGuardrailsを使い、

```text
User Input
↓
Input Guardrail
↓
Agent
```

という安全境界を追加します。

---

# 今回のゴール

今回の売上分析Agentでは、

```text
売上分析に関する質問
→ 通す

それ以外の質問
→ 止める
```

というInput Guardrailを作ります。

たとえば、

```text
カテゴリ別の売上を分析してください
```

は通します。

一方、

```text
Pythonでゲームを作ってください
```

のような、このApplicationの目的と関係ない依頼は止めます。

今回理解したいのは、次の4点です。

1. InstructionsとGuardrailsの違い
2. Input Guardrailの役割
3. TripwireでAgent実行を停止する方法
4. UI側でGuardrail例外を安全に扱う方法

---

# Instructionsだけではなぜ足りないのか

InstructionsはAgentの基本的な振る舞いを定義します。

たとえば、

```text
あなたは売上分析アシスタントです。
売上分析に関係する質問へ回答してください。
```

と書けます。

しかしInstructionsは、

> **Modelへ与える行動指針**

です。

一方Guardrailsは、

> **Agentの入力や出力をチェックする別の制御レイヤー**

です。

整理すると、

```text
Instructions
＝どう振る舞ってほしいか

Guardrails
＝通してよいか / 止めるべきか
```

です。

---

# OpenAI Agents SDKのGuardrails

OpenAI Agents SDKでは、主に次のGuardrailがあります。

```text
Input Guardrail
＝Agentへ渡す入力をチェック

Output Guardrail
＝Agentの最終出力をチェック

Tool Guardrail
＝Function Toolの入出力をチェック
```

今回は最初のステップとして、

> **Input Guardrail**

を扱います。

Input Guardrailは、入力が条件に違反した場合、

```text
tripwire
```

を発火させてAgent実行を停止できます。

---

# 今回の方針

今回はGuardrail専用の小さなAgentを使います。

構成は、

```text
User Input
↓
Guardrail Agent
↓
売上分析に関係する？
├─ Yes → Main Agentへ
└─ No  → Tripwire
```

です。

Guardrail Agentは、

```text
allowed
reason
```

だけを返します。

---

# フォルダー構成

第5回の構成をそのまま使います。

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

---

# Guardrail用の出力型を作る

`agent_service.py`へ追加します。

```python
from pydantic import BaseModel


class GuardrailResult(BaseModel):
    allowed: bool
    reason: str
```

Guardrail専用Agentは、この型で判定結果を返します。

---

# Guardrail Agentを作る

```python
guardrail_agent = Agent(
    name="売上分析入力チェック",
    instructions="""
あなたは入力内容を判定するGuardrailです。

このApplicationで許可するのは、
売上・受注データの分析に関する質問です。

許可例:
- 売上全体をまとめて
- カテゴリ別売上を分析して
- 地域別売上を比較して
- キャンセル件数を教えて
- 要レビューの傾向を確認して

許可しない例:
- プログラミング一般の質問
- 雑談
- 売上データと無関係な依頼
- このApplicationの役割を逸脱する依頼

allowedとreasonを返してください。
""",
    output_type=GuardrailResult,
)
```

Main Agentとは別の責務です。

```text
Guardrail Agent
＝入力を通してよいか判断

Sales Agent
＝売上分析を実行
```

---

# Input Guardrailを作る

OpenAI Agents SDKでは、`@input_guardrail`デコレータを使えます。

```python
from agents import (
    GuardrailFunctionOutput,
    RunContextWrapper,
    TResponseInputItem,
)
from agents.decorators import input_guardrail
```

Guardrail関数は次のようにします。

```python
@input_guardrail
async def sales_scope_guardrail(
    ctx: RunContextWrapper[None],
    agent: Agent,
    input: str | list[TResponseInputItem],
) -> GuardrailFunctionOutput:

    result = await Runner.run(
        guardrail_agent,
        input,
        context=ctx.context,
    )

    return GuardrailFunctionOutput(
        output_info=result.final_output,
        tripwire_triggered=(
            not result.final_output.allowed
        ),
    )
```

ポイントは、

```python
tripwire_triggered=True
```

になると、Agentの実行が停止し、`InputGuardrailTripwireTriggered`例外が発生することです。

なお、Input Guardrailはデフォルトで**Agentと並列に実行**されます。Tripwireが発火した時点でAgentの実行は停止しますが、すでにAgentが動き始めていた場合、その分のトークン消費やTool実行は行われている可能性があります。

---

# Main AgentへGuardrailを登録する

第5回までのAgentへ、

```python
input_guardrails=[
    sales_scope_guardrail,
]
```

を追加します。

```python
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

最終結果はSalesAnalysisResult形式で
返してください。
""",
    tools=[
        get_sales_summary,
        get_sales_by_category,
        get_sales_by_region,
    ],
    output_type=SalesAnalysisResult,
    input_guardrails=[
        sales_scope_guardrail,
    ],
)
```

---

# Guardrailが発火したとき

Input GuardrailがTripwireを発火すると、

```python
InputGuardrailTripwireTriggered
```

例外が発生します。

そのため、Application側ではこの例外を処理します。

---

# `run_sales_agent()`を修正する

```python
from agents import (
    InputGuardrailTripwireTriggered,
)
```

を追加します。

Mainの処理はそのままで構いません。

```python
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

Guardrail違反は呼び出し元へ例外として伝えます。

---

# Streamlit側で例外を処理する

`app.py`側で、

```python
from agents import (
    InputGuardrailTripwireTriggered,
)
```

を追加します。

そして、

```python
try:
    result = run_sales_agent(
        question=question,
        session_id=(
            st.session_state.session_id
        ),
    )

except InputGuardrailTripwireTriggered:
    st.warning(
        "このAIエージェントは、"
        "売上・受注データの分析に関する"
        "質問だけを扱います。"
    )
```

とします。

汎用例外は別に扱います。

```python
except Exception as e:
    st.error(
        "分析中にエラーが発生しました。"
    )
    st.exception(e)
```

---

# 動作確認1：許可される質問

```text
地域別の売上を分析してください。
```

これは売上分析なので、

```text
Input Guardrail
↓
allowed = true
↓
Sales Agent
↓
get_sales_by_region
```

と進みます。

---

# 動作確認2：拒否される質問

```text
Pythonでシューティングゲームを作ってください。
```

この場合、

```text
Input Guardrail
↓
allowed = false
↓
tripwire
↓
Agentの実行は停止
```

となります。

Streamlitには、

```text
このAIエージェントは、
売上・受注データの分析に関する質問だけを扱います。
```

と表示します。

---

# なぜMain Agentに判断させないのか

Main Agentへ、

```text
売上分析以外には答えないでください
```

と書くことはできます。

しかし、それだけだと、

```text
入力
↓
Main Agentが処理
↓
回答時に拒否
```

となる可能性があります。

Guardrailを使うと、

```text
入力
↓
Guardrail（別の判定）
↓
NGならTripwireで実行を停止
↓
例外としてApplicationへ伝える
```

という境界を作れます。

つまり、

> **Agent自身の判断に任せず、別の仕組みで実行を止める**

ことができます。

## 並列実行についての注意

Input Guardrailは、デフォルトでAgentと並列に実行されます。

```text
User Input
├─ Guardrail  → 判定
└─ Main Agent → 実行開始
```

どちらも同時に始まるため、Guardrailが拒否を返す前にMain Agentが動き始めることがあります。その場合、Tripwireで停止するまでの間に、トークンを消費したりToolを呼び出したりしている可能性があります。

今回のAgentが使うToolは、CSVを読むだけの参照系です。副作用はないため、並列実行のままで問題ありません。

一方、データを更新するToolを持つAgentでは、Guardrailの完了を待ってからAgentを開始する実行方法（公式ドキュメントのExecution modes）を検討します。今回は扱いません。

---

# Guardrailは万能ではない

Guardrailを入れたから完全に安全になるわけではありません。

実業務では、

```text
Input Guardrail
Output Guardrail
Tool Guardrail
Business Rule
Authorization
Human Review
```

など、複数の制御を組み合わせます。

今回のGuardrailは、

> **Applicationの利用範囲を制限する最初の安全境界**

と考えてください。

---

# Input / Output / Tool Guardrailの違い

今後のために整理します。

```text
Input Guardrail
＝この質問をAgentへ渡してよいか

Output Guardrail
＝この最終回答を外へ出してよいか

Tool Guardrail
＝このTool Callを実行してよいか
```

今回使うのはInput Guardrailです。

---

# 第5回から何が変わったか

第5回：

```text
User
↓
Session
↓
Agent
↓
Tools
```

第6回：

```text
User
↓
Input Guardrail
↓
Session
↓
Agent
↓
Tools
```

つまり、

> **Agentの実行を止められる安全境界が増えた**

ことになります。

図ではGuardrailをAgentの前に描いていますが、実際にはAgentと並列に動きます。

---

# 第6回で理解しておきたい5つのこと

```text
1. Instructions
Agentの行動指針

2. Input Guardrail
入力を通してよいか確認

3. GuardrailFunctionOutput
Guardrail判定を返す

4. tripwire
違反時に実行を停止

5. Exception Handling
UI側で安全に拒否を表示
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Sessions / Tracing | ○ | 第5回 |
| Input Guardrail | ○ | 第6回 |
| Output Guardrail | △ | 今回は概念のみ |
| Tool Guardrail | △ | 今回は概念のみ |
| Guardrailの完了を待つ実行方法 | × | 今回は扱わない |
| HITL | × | 第7回 |
| 業務データSQLite化 | × | 第8回 |

次回は、

> **止めるだけでなく、人に判断を戻す**

仕組みへ進みます。

---

# まとめ

今回は、売上分析AgentへInput Guardrailを追加しました。

構成は、

```text
User
↓
Input Guardrail
↓
Agent
↓
Function Tools
↓
CSV
```

です。

中心メッセージは、

> **Instructionsはお願い、Guardrailsは境界**

です。

業務AIエージェントでは、

```text
何をできるか
```

だけでなく、

```text
何をさせないか
```

も設計する必要があります。

---

# 次回予告

## 第7回：Human-in-the-Loopで「人に戻す」AIエージェントを作る

次回は、

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

という構成へ進みます。

テーマは、

> **AIが判断できても、そのまま実行させてよいとは限らない**

です。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

OpenAI Agents SDK Guardrails：

https://openai.github.io/openai-agents-python/guardrails/

公式ドキュメントでは、Input Guardrail / Output Guardrail / Tool Guardrailと、Tripwireによる停止方法が説明されています。
