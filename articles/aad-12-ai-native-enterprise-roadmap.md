---
title: "第12回 AI Native Enterpriseへのロードマップ"
emoji: "🧭"
type: "tech"
topics:
  - AIエージェント
  - AI_Native
  - SystemSpecification
  - SDD
  - AI設計
published: true
published_at: "2026-09-11 07:00"
---

# AIエージェント設計の先にあるもの

本記事は「AIエージェント設計実践シリーズ」の最終回です。

シリーズ全体の概要は、ハブ記事をご覧ください。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

これまで11回にわたり、

**AIエージェントを業務で使うために、何をどう設計すべきか**

を考えてきました。

第1回、第2回では設計の基本を整理し、

第3回から第7回では具体的な業務ユースケースを通して、
System Specificationをレビューしてきました。

第8回から第10回では、
Python、マルチエージェント、Code Agentなど、
実装につながる設計へと範囲を広げました。

そして第11回では、

**PoCとして動くAIを、どう本番運用できるAIへ育てるか**

を扱いました。

最終回では、そのさらに先を考えます。

テーマは、

**AI Native Enterprise**

です。

---

# このシリーズ全体の流れ

ここまでの流れを整理すると、次のようになります。

```text
第1回〜第2回
AIエージェント設計の基礎
        ↓
第3回〜第7回
業務ユースケース別の
System Specificationレビュー
        ↓
第8回〜第10回
実装へつながる
System Specification
        ↓
第11回
PoCから本番運用へ
        ↓
第12回
AI Native Enterpriseへ
```

このシリーズで一貫して扱ってきたのは、

**AIに何をさせるのかを、実装前に設計すること**

です。

AIエージェントは、
モデルやツールを選べば完成するものではありません。

業務目的、入力、出力、判断基準、制約、例外、
そして人間との役割分担まで定義する必要があります。

その中心にあるのが、

**System Specification**

です。

---

# AIエージェントは業務システムの一部になる

AIエージェントを試すだけなら、
チャット画面や簡単なPoCでも十分です。

しかし、実際の業務へ導入すると状況が変わります。

例えば問い合わせトリアージAIであれば、

```text
問い合わせ受信
      ↓
内容を確認
      ↓
問い合わせ種別を判断
      ↓
優先度を判断
      ↓
担当部署を決定
      ↓
必要に応じて人間へ引き継ぐ
      ↓
結果を記録
```

という業務プロセスの一部になります。

このときAIエージェントだけを見るのではなく、

**業務システム全体の中で、AIがどの役割を担うのか**

を考える必要があります。

つまり、

```text
AI Agent
```

ではなく、

```text
Human
  +
AI Agent
  +
Business Rule
  +
Existing System
  +
Data
  +
Monitoring
```

を一つのシステムとして設計する必要があります。

---

# 人間とAIの役割を分ける

AI Nativeな業務とは、

すべてをAIへ任せる業務ではありません。

むしろ重要なのは、

**AIが判断する部分と、人間が判断する部分を明確に分けること**

です。

例えば、

```text
通常ケース
    ↓
AIが処理

例外ケース
    ↓
Human in the Loop

重大ケース
    ↓
人間が最終判断
```

という設計が考えられます。

ここで重要になるのが、

- AIが自動実行してよい範囲
- AIが判断できない条件
- 人間へ引き継ぐ条件
- 人間が承認すべき処理
- 実行してはいけない処理

です。

AI Native Enterpriseでは、

**AIを増やすことより、AIと人間の境界を設計すること**

が重要になります。

---

# 複数のAIエージェントをどう管理するか

業務が複雑になると、
1つのAIエージェントですべてを処理するのは難しくなります。

例えば、

```text
問い合わせ受付Agent
        ↓
分類Agent
        ↓
優先度判定Agent
        ↓
回答Agent
        ↓
エスカレーションAgent
```

のように、
複数のAgentが協調する構成も考えられます。

しかし、

Agentを増やせばよいわけではありません。

必要なのは、

- 各Agentの役割
- 入出力
- 実行順序
- 判断権限
- 失敗時の処理
- Human in the Loop
- 全体の管理方法

を明確にすることです。

つまり、

**Multi-Agentでも設計の中心はSystem Specification**

です。

---

# AIを「仕様」で統制する

AI Native Enterpriseでは、
AIエージェントが業務のさまざまな場所で動くようになります。

このとき問題になるのが、

**AIの判断をどう統制するか**

です。

例えば、

```text
AIができること
AIができないこと
AIが判断してよいこと
AIが人間へ確認すべきこと
AIが絶対に実行してはいけないこと
```

を明確にする必要があります。

そこで重要になるのが、

- Business Specification
- System Specification
- Guardrails
- Verification
- Monitoring
- Logging
- Human in the Loop

です。

AIを自由に動かすのではなく、

**仕様の中で動かす**

という考え方です。

---

# AI Nativeな業務プロセス

従来の業務自動化では、

```text
人間の業務
      ↓
一部をシステム化
```

という考え方が中心でした。

AI Nativeでは、

最初から

```text
Human
  +
AI Agent
  +
System
```

を前提として業務を設計します。

例えば、

```text
As-Is
人間がすべて判断
        ↓
To-Be
AIが一次判断
        ↓
通常ケースは自動処理
        ↓
例外ケースだけ人間へ
        ↓
結果をMonitoring
        ↓
仕様を改善
```

という形です。

重要なのは、

**既存業務へAIを追加するだけではなく、
AIを前提としてTo-Beを再設計すること**

です。

---

# AI導入は段階的に進める

いきなり企業全体をAI Nativeにすることはできません。

段階的に進める必要があります。

```text
Step 1
業務課題を整理する
        ↓
Step 2
System Specificationを作る
        ↓
Step 3
AI Agentを実装する
        ↓
Step 4
Evaluationする
        ↓
Step 5
PoCで業務適合性を確認する
        ↓
Step 6
Productionへ展開する
        ↓
Step 7
Monitoringする
        ↓
Step 8
仕様を改善する
```

ここで重要なのは、

**PoCをゴールにしないこと**

です。

PoCは、

「AIが動くか」

を見るだけではありません。

本来は、

**このAIを業務で使えるのか**

を確認するための段階です。

---

# AIエージェントは「作る」のではなく「育てる」

第11回では、

- Guardrails
- Verification
- Monitoring
- Logging
- Human in the Loop
- Evaluation

などを扱いました。

これらはすべて、

AIエージェントを本番で使い続けるために必要な仕組みです。

つまりAIエージェントは、

```text
設計
 ↓
実装
 ↓
評価
 ↓
PoC
 ↓
Production
 ↓
Monitoring
 ↓
改善
```

というサイクルで育てていくものです。

一度実装して終わりではありません。

利用結果から問題を見つけ、
System Specificationを更新し、
再び実装へ反映します。

```text
System Specification
        ↓
Implementation
        ↓
Operation
        ↓
Monitoring
        ↓
Review
        ↓
System Specification更新
```

この循環が重要になります。

---

# これからの学習ロードマップ

ここまでのシリーズでは、

**AIエージェントをどう設計するか**

を中心に扱ってきました。

ここからは、その設計を実際のシステムへつなげていきます。

```text
AIエージェント設計
        ↓
SDD
        ↓
Agent実装
        ↓
Evaluation
        ↓
Web PoC
        ↓
Go / No-Go
        ↓
AWS Production
        ↓
Monitoring / 改善
```

中心ケースとして、
問い合わせトリアージAIエージェントを段階的に発展させます。

---

# 問い合わせトリアージAIを段階的に育てる

例えば問い合わせトリアージでは、
次のように学習を進めます。

## Agent実装

```text
Classification
      ↓
Priority
      ↓
Guardrails
      ↓
Routing
      ↓
Handoff
      ↓
Tracing
```

OpenAI Agents SDKなどを使いながら、
System SpecificationからAgentへ落とし込んでいきます。

## Web PoC

次に、

```text
Agent
 ↓
Streamlit
 ↓
FastAPI
 ↓
SQLite
 ↓
Human Escalation
 ↓
Evaluation
 ↓
Go / No-Go
```

と発展させます。

ここでは、
「動くか」だけではなく、

**業務導入できる品質か**

を評価します。

## AWS Production

さらに、

```text
Docker
 ↓
PostgreSQL
 ↓
Terraform
 ↓
ECR
 ↓
ECS / Fargate
 ↓
RDS
 ↓
Secrets Manager
 ↓
CloudWatch
```

へ発展させます。

これによって、

```text
設計
 ↓
Agent
 ↓
PoC
 ↓
Production
```

を一つの業務システムとして学んでいきます。

---

# さまざまな業務へ展開する

問い合わせトリアージ以外にも、
業務によって必要なAgent設計は変わります。

| ユースケース | 主なテーマ |
|---|---|
| 会議後フォローアップ | Structured Output / ToDo生成 |
| 問い合わせトリアージ | Classification / Priority / Routing / Handoff |
| 社内ヘルプデスク・FAQ | RAG / File Search / Grounding |
| 経費・購買申請の事前審査 | Business Rule / Guardrails / Human in the Loop |
| 営業案件フォロー | Function Tools / CRM / Email / Calendar / Sessions |
| 営業・業務実績レポート | Multiple Tools / Data Analysis / Report Generation |
| 契約書レビュー | Document Retrieval / Risk Classification |
| クレーム対応 | Severity / Escalation / Handoff |
| IT障害一次対応 | State / Investigation / Tool Use |
| 業務KPI分析 | Data Agent / Multi-Agent |

ここでも共通するのは、

**ツールから考えるのではなく、業務から設計する**

という考え方です。

---

# AI Native Enterpriseへ

AI Native Enterpriseとは、

AIツールをたくさん導入した企業ではありません。

私が考えるAI Native Enterpriseは、

```text
Business
   ↓
Specification
   ↓
AI Agent
   ↓
Human
   ↓
System
   ↓
Monitoring
   ↓
Improvement
```

が一体となって動く組織です。

つまり、

**AIを前提として業務とシステムを設計できる組織**

です。

そのために必要なのは、

最新のLLMを知ることだけではありません。

- 業務を理解する
- AIの役割を決める
- 仕様を書く
- Humanとの境界を決める
- 評価する
- 監視する
- 改善する

という力です。

モデルやツールは変わっていきます。

しかし、

**「AIに何を判断させるのか」を設計する重要性は変わりません。**

---

# まとめ

このシリーズでは、

第1回から第2回でAIエージェント設計の基本を学び、

第3回から第7回で、
実際の業務ユースケースを使ってSystem Specificationをレビューしました。

第8回から第10回では、
Python、マルチエージェント、Code Agentへと範囲を広げ、

第11回では、
PoCから本番運用へ進むための設計を考えました。

そして最終回では、

**個別のAIエージェントから、AI Native Enterpriseへ**

視点を広げました。

```text
業務
 ↓
System Specification
 ↓
AI Agent
 ↓
PoC
 ↓
Production
 ↓
Monitoring
 ↓
改善
```

AIエージェント開発は、

AIを「作る」ことがゴールではありません。

**業務に合わせて設計し、
運用しながら育て続けること。**

これが、
AI Native時代のAIエージェント開発だと考えています。

---

# もっと体系的に学びたい方へ

Business Specification・System Specification、
そしてSDDの考え方を体系的に学びたい方は、

📚 [AI時代の仕様駆動開発：Kiro・Claude Codeで学ぶ System Specification Design（SDD）の教科書](https://www.amazon.co.jp/dp/B0HBQ32KCQ)

をご覧ください。

そして今後は、

**設計 → Agent実装 → Web PoC → AWS Production**

へと、より実装・運用に近い内容へ進んでいきます。

---

# シリーズ全体はこちら

「AIエージェント設計実践シリーズ」の全記事はこちらです。

https://zenn.dev/tigerone1945/articles/aad-00-hub-system-specification-series

---

# おわりに

ここまでシリーズをお読みいただき、
ありがとうございました。

第1回では、

**AIエージェントを作る前に、何を設計すべきなのか**

というところから始まりました。

そして最終回では、

**AIを前提として業務そのものをどう設計するのか**

まで視点を広げました。

AIエージェントを「作る」時代から、
AIエージェントを「設計する」時代へ。

そしてその先にあるのが、

**AI Native Enterprise**

です。

ここからは、
設計したAIエージェントを実際に実装し、
PoCで評価し、
Productionへ育てていきます。
