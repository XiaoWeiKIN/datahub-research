# Case 01 — Analytics Agent: Enterprise Customer Net Revenue

## “过去 90 天 Enterprise Customer Net Revenue 是多少？同比如何？为什么我应该相信这个数字？”

**Snapshot:** 2026-09-28  
**Case Type:** Synthetic enterprise scenario, grounded in current DataHub capabilities

---

# 0. Case 说明

用户问题：

> **过去 90 天 Enterprise Customer Net Revenue 是多少？与去年同期相比怎么样？为什么我应该相信这个数字？**

这句话看起来只需要一个 SQL。

实际上至少包含六类不确定性：

1. **Enterprise Customer 是什么？**
2. **Net Revenue 是什么？**
3. **应该用哪个数据集 / semantic model？**
4. **数据现在是否 fresh / healthy？**
5. **当前用户 / Agent 是否有权限执行？**
6. **答案最终基于哪些 evidence？**

所以这个 case 的目标不是展示“LLM 会写 SQL”。

而是展示：

> **一个 production Analytics Agent 如何逐步缩小不确定性，直到剩下可以交给 deterministic system 执行的部分。**

---

# 1. Synthetic Fixture

下面全部是为了架构推演而设定的虚构企业事实，不代表任何真实 DataHub customer。

假设企业有：

## Business Terms

### Enterprise Customer

Finance-approved 定义：

~~~text
customer_segment = 'enterprise'
AND account_status = 'active'
~~~

Growth 团队还有一个旧定义：

~~~text
annual_contract_value >= 100000
~~~

因此存在潜在 semantic conflict。

### Net Revenue

Finance-certified metric：

~~~text
gross_revenue
- refunds
- credits
~~~

仅统计：

~~~text
order_status = 'completed'
~~~

---

# 2. Synthetic Data Assets

假设相关资产：

~~~text
analytics.fact_orders
analytics.dim_customers
finance.fact_refunds
finance.fact_credits
semantic.finance_metrics
~~~

其中：

- `fact_orders`：订单事实；
- `dim_customers`：客户 segment / account status；
- `fact_refunds`：退款；
- `fact_credits`：credit adjustments；
- `semantic.finance_metrics`：Finance metric semantic definition。

---

# 3. Context Graph Fixture

我们希望 Context Layer 至少知道：

~~~mermaid
graph TB
    TERM[Enterprise Customer]
    METRIC[Net Revenue]
    SEM[Finance Semantic Model]
    ORDERS[fact_orders]
    CUSTOMERS[dim_customers]
    REFUNDS[fact_refunds]
    CREDITS[fact_credits]
    FIN[Finance Analytics]
    QUALITY[Freshness / Quality]
    POLICY[Board Reporting Policy]

    TERM --> CUSTOMERS
    METRIC --> SEM
    SEM --> ORDERS
    SEM --> REFUNDS
    SEM --> CREDITS
    FIN --> METRIC
    FIN --> TERM
    ORDERS --> QUALITY
    POLICY --> METRIC
~~~

注意：

> 这个 graph 本身还不够。

Agent 还要知道：

- 哪条 definition published；
- 哪条 authority 更高；
- freshness；
- policy；
- valid scope。

---

# 4. Naive Agent 的路径

最简单 Text-to-SQL Agent 可能：

~~~mermaid
flowchart LR
    Q[User Question]
    SCHEMA[Warehouse Schema]
    LLM[LLM]
    SQL[Generated SQL]
    DB[Warehouse]
    OUT[Answer]

    Q --> LLM
    SCHEMA --> LLM
    LLM --> SQL --> DB --> OUT
~~~

它可能生成语法完全正确的 SQL，但：

- 选旧 Enterprise definition；
- 把 gross revenue 当 net revenue；
- 漏 credits；
- join fan-out；
- 用 deprecated table；
- 用 stale data。

DataHub 自己发布 Analytics Agent 时，也把这类“SQL syntactically correct but semantically wrong”作为核心失败模式。

---

# 5. Production Agent 的目标路径

更合理的 architecture：

~~~mermaid
flowchart TB
    USER[User Intent]

    subgraph CTX[Context / Epistemic Plane]
        DISC[Discover]
        RESOLVE[Resolve Meaning / Authority]
        TRUST[Freshness / Quality / Provenance]
    end

    subgraph SEM[Semantic Plane]
        MODEL[Metric / Join Definition]
        COMPILE[Compile Query]
    end

    subgraph POLICY[Identity / Policy Plane]
        AUTHZ[Authorize]
    end

    subgraph DATA[Data Plane]
        EXEC[Execute]
    end

    subgraph AG[Agent Plane]
        REASON[Interpret / Plan]
        ANSWER[Synthesize Answer]
    end

    USER --> REASON
    REASON --> DISC --> RESOLVE --> TRUST
    TRUST --> MODEL --> COMPILE
    COMPILE --> AUTHZ --> EXEC
    EXEC --> ANSWER
    TRUST --> ANSWER
~~~

核心顺序：

> **先决定应该相信和使用什么，再执行计算。**

---

# 6. Step 1 — Intent Parsing

Agent 先把问题拆成：

~~~text
Measure:
  Net Revenue

Population:
  Enterprise Customer

Window:
  trailing 90 days

Comparison:
  same period previous year

Output:
  total + YoY change + trust explanation
~~~

这一阶段仍然是 probabilistic reasoning。

但 Agent 不应该自己定义：

- Enterprise；
- Net Revenue；
- join；
- authority。

---

# 7. Step 2 — Context Discovery

当前 DataHub Context Activation 官方建议 analytics / text-to-SQL Agent 使用：

> `datahub-sql-workflow` skill

这个 skill 会在回答自然语言业务问题时先搜索 DataHub context for grounded truth。

DataHub Analytics Agent 的公开架构也明确：

> every question runs through a context-enrichment pipeline before SQL generation.

所以 Agent 的第一步不是：

~~~text
list warehouse tables
~~~

而是：

~~~text
search business context
search metric context
search relevant assets
~~~

Source:

- https://docs.datahub.com/docs/managed-datahub/context/activate-context
- https://datahub.com/blog/datahub-analytics-agent/

---

# 8. Step 3 — Resolve “Enterprise Customer”

Agent 搜到两个定义：

~~~text
Definition A
  source: Finance Context
  status: published
  authority: Finance
  definition:
    customer_segment='enterprise'
    AND account_status='active'

Definition B
  source: Growth legacy document
  definition:
    annual_contract_value >= 100000
~~~

一个 naive retriever 可能按照 embedding similarity 直接取一个。

Production path 应该：

~~~mermaid
flowchart LR
    A[Definition A]
    B[Definition B]
    CONFLICT[Conflict]
    SCOPE[Task Scope]
    AUTH[Authority]
    WIN[Applicable Definition]

    A --> CONFLICT
    B --> CONFLICT
    CONFLICT --> SCOPE --> AUTH --> WIN
~~~

任务是：

> Finance Net Revenue reporting

所以 Finance-approved definition 更 applicable。

---

# 9. DataHub Current Capability vs Ideal

## Current DataHub

DataHub 当前可以提供：

- Glossary / Context Documents；
- ownership；
- domains；
- published vs unpublished context；
- search / MCP；
- human-reviewed context。

## Reference Architecture Ideal

还应统一返回：

~~~text
epistemic_state = VALID
scope = finance_reporting
authority = Finance
conflicts = [Growth definition]
valid_from = ...
~~~

当前 DataHub 公开资料不能证明存在通用 conflict-aware retrieval contract。

所以：

> **Context discovery 已实现；通用 authority-based conflict resolution 仍属于 Reference Architecture。**

---

# 10. Step 4 — Resolve “Net Revenue”

Agent 不应该从 column names 自己猜：

~~~text
revenue
net_revenue
gross_revenue
booked_revenue
recognized_revenue
~~~

而应该找到：

- Finance-certified metric；
- semantic model；
- Context Document；
- glossary / metric definition；
- query patterns。

DataHub 2026 已把 Metric / Semantic Model 变成更明确的一等 context surface，并继续增加 metric-level lineage。

DataHub Analytics Agent 的官方定位也明确要求 glossary resolution / semantic context 先于 SQL generation。

---

# 11. Step 5 — Resolve Query Pattern / Join Path

假设 context 给出：

~~~text
dim_customers.customer_id
-> fact_orders.customer_id

fact_orders.order_id
-> fact_refunds.order_id

fact_orders.order_id
-> fact_credits.order_id
~~~

以及 approved query pattern：

~~~text
aggregate refunds and credits before joining to order grain
~~~

这非常重要。

否则 LLM 可能直接：

~~~sql
orders
JOIN refunds
JOIN credits
~~~

产生 fan-out duplication。

所以 Context Layer 的价值不只是：

> “这个表叫什么”。

还包括：

> **组织过去已经验证过的 query pattern。**

DataHub Context Intelligence 当前特别利用 query history 来提取 join / calculation pattern，这正是当前 Context Platform 的重点之一。

---

# 12. Step 6 — Check Freshness & Quality

在生成 SQL 前，Agent 查询：

~~~text
fact_orders
  freshness: healthy
  last update: within SLA
  incidents: none

fact_refunds
  freshness: healthy

fact_credits
  freshness: warning?
~~~

这里有三个结果模式。

## Case A — Healthy

继续执行。

## Case B — Stale but usable

例如：

> credits 数据延迟 20 分钟，但 Finance SLA 允许 60 分钟。

可以执行，但答案需要 disclosure。

## Case C — Invalid / open incident

例如：

> refunds pipeline broken for 36 hours.

Agent 不应该继续给 authoritative number。

应该：

~~~text
block / warn / ask user whether approximate answer acceptable
~~~

---

# 13. DataHub Current Capability

DataHub 已经有：

- lineage；
- Assertions；
- freshness；
- incidents；
- quality signals；
- external quality integrations。

DataHub Analytics Agent 官方文章也明确说它会读取：

- quality signals；
- usage；
- lineage；
- glossary；
- schema；

而不是只读 DDL。

所以这一段是 DataHub 当前真实能力方向，而不是纯理论。

Source:

- https://datahub.com/blog/datahub-analytics-agent/
- https://docs.datahub.com/docs/features/feature-guides/observe

---

# 14. Step 7 — Semantic Execution

这里必须保持 Phase 1 定义的边界。

Context Layer 决定：

~~~text
use Finance Net Revenue
use Enterprise Customer definition A
use these assets
use this join pattern
~~~

Semantic / execution layer决定：

~~~text
how exactly is metric compiled?
~~~

如果企业已有：

- dbt Semantic Layer；
- Snowflake semantic view；
- Cube；
- another governed semantic runtime；

Agent 应优先调用它。

如果没有：

> DataHub Analytics Agent current reference implementation 会利用 context 生成 SQL，然后直接对 warehouse 执行。

官方文章明确说它支持 Snowflake、BigQuery、Databricks、Redshift、Postgres，并在 context enrichment 后生成 SQL。

所以有两条合法架构：

~~~mermaid
flowchart TB
    CTX[Context]

    CTX --> SEM[Semantic Runtime]
    SEM --> DB1[Warehouse]

    CTX --> SQL[Context-grounded SQL Generation]
    SQL --> DB2[Warehouse]
~~~

第一条 deterministic 程度更高。

第二条是当前 Analytics Agent reference implementation 能展示的模式。

---

# 15. Step 8 — Runtime Authorization

Context Layer 可以知道：

- 哪张表 relevant；
- data classification；
- owner；
- business purpose。

但最终是否允许当前 user / Agent 查询：

~~~text
customer data
revenue data
sensitive dimensions
~~~

不能只由 Context Layer 决定。

正确模型：

~~~mermaid
flowchart LR
    USER[User Principal]
    AG[Agent]
    CTX[Context]
    POLICY[Runtime Authorization]
    DB[Warehouse]

    USER --> AG
    CTX --> AG
    AG --> POLICY --> DB
~~~

如果执行环境是 Snowflake：

> Snowflake role / policy 最终 enforce。

如果是其他 warehouse：

> 由该系统 IAM/policy enforce。

因此：

> **Context relevance != execution authorization.**

---

# 16. Step 9 — Generate SQL

假设没有独立 semantic compiler。

Agent 在 context-grounded constraints 下生成：

~~~sql
WITH enterprise_customers AS (
  SELECT customer_id
  FROM analytics.dim_customers
  WHERE customer_segment = 'enterprise'
    AND account_status = 'active'
),
order_revenue AS (
  SELECT
    order_id,
    customer_id,
    order_date,
    gross_revenue
  FROM analytics.fact_orders
  WHERE order_status = 'completed'
),
refunds AS (
  SELECT order_id, SUM(refund_amount) AS refund_amount
  FROM finance.fact_refunds
  GROUP BY 1
),
credits AS (
  SELECT order_id, SUM(credit_amount) AS credit_amount
  FROM finance.fact_credits
  GROUP BY 1
)
SELECT ...
~~~

这里 SQL 只是示意。

Case Study 不声称这是某个真实组织的正确 SQL。

真正重要的是：

> Agent 不应该自行发明这些业务约束。

它们来自 context / semantic definitions。

---

# 17. Step 10 — Query Execution

执行时需要保留：

~~~text
query_id
warehouse
principal
role
timestamp
sql_hash
semantic/context versions
~~~

如果执行失败：

> 这是 execution failure。

如果执行成功但结果错误：

> 更可能是 context / semantic failure。

区分这两种错误非常重要。

---

# 18. Step 11 — YoY Calculation

用户要求同比。

定义需要明确：

~~~text
current:
  trailing 90 days ending today

previous:
  exact corresponding date interval one year earlier
~~~

或者公司可能使用：

- fiscal calendar；
- comparable weeks；
- 13-week period。

Agent 不应该自己假设。

如果 Context Graph 存在 Finance fiscal calendar policy：

> 先 resolve period semantics。

这再次说明：

> 时间窗口本身也可能是 business context。

---

# 19. Step 12 — Evidence Package

最终答案不应该只返回：

~~~text
Net Revenue = $123.4M
YoY = +8.2%
~~~

更理想：

~~~text
Answer
├── result
├── comparison
├── metric definition
├── population definition
├── data sources
├── freshness / quality
├── authority
├── query / semantic execution ref
├── lineage
└── caveats
~~~

也就是：

> **Answer + Evidence Package**

---

# 20. Example Final Answer Shape

一个 Agent 可以返回：

~~~text
过去 90 天 Enterprise Customer Net Revenue:
  $123.4M

同比:
  +8.2%

使用定义:
  - Enterprise Customer: Finance-approved active enterprise segment
  - Net Revenue: Gross Revenue - Refunds - Credits

数据状态:
  - Orders: healthy
  - Refunds: healthy
  - Credits: within freshness SLA

Authority:
  Finance Analytics

Sources:
  fact_orders
  dim_customers
  fact_refunds
  fact_credits

Execution:
  query id: ...
~~~

如果其中某一项 unhealthy：

> disclosure 必须进入答案。

---

# 21. “为什么应该相信？”其实是独立输出

这是整个 case 最重要的地方。

传统 Agent 只回答：

> what is the number?

Context-aware Agent 还应该回答：

> why should I trust the number?

这个 trust explanation 至少包括：

~~~mermaid
flowchart LR
    DEF[Certified Definition]
    AUTH[Authority]
    LINEAGE[Lineage]
    QUALITY[Quality / Freshness]
    EXEC[Execution Evidence]

    DEF --> TRUST[Trust Explanation]
    AUTH --> TRUST
    LINEAGE --> TRUST
    QUALITY --> TRUST
    EXEC --> TRUST
~~~

这才是 Context Layer 真正提供的价值。

---

# 22. Step 13 — Write-back

假设 Agent 在执行中发现：

> Growth 的 Enterprise definition仍然存在，但已经与 Finance-certified definition冲突。

Agent 不应该直接删除它。

更合理的写回：

~~~text
propose:
  mark Growth definition as legacy
  link to Finance-approved definition
  ask Growth owner to confirm
~~~

或者：

> 发现一个 validated join pattern 还没有文档化。

Agent 可以：

- propose glossary context；
- propose documentation；
- save context document；
- create metadata proposal。

DataHub Analytics Agent 官方 reference implementation 明确说：

> 当它发现 undocumented tables 或 undefined terms，会把修正写回 DataHub，而不是静默 workaround。

Source:

- https://datahub.com/blog/datahub-analytics-agent/

---

# 23. Write-back 必须保持 Authority Boundary

正确：

~~~mermaid
flowchart LR
    AG[Agent Observation]
    PROP[Proposal]
    SME[Finance / Growth SME]
    PUB[Published Context]

    AG --> PROP --> SME --> PUB
~~~

错误：

~~~text
Agent noticed ambiguity
-> overwrote glossary
-> future agents see only its choice
~~~

Phase 1 中的原则在这里得到验证：

> **Agents propose. Evidence verifies. Authorities publish.**

---

# 24. Step 14 — Audit

要重建一次回答，理想 audit 包括：

~~~text
DecisionTrace
├── user / principal
├── agent + model version
├── question
├── retrieved context ids
├── published context versions
├── metric / semantic model version
├── freshness / incident state
├── policy decision
├── SQL
├── warehouse query id
├── tool calls
├── result
└── write-back proposals
~~~

DataHub 当前已经有：

- metadata lineage；
- Agent Registry versions；
- task/tool audit；
- context documents；
- query history；
- MCP traces in relevant products。

但完整 historical decision replay 仍然是我们 Reference Architecture 更强的目标。

---

# 25. Current DataHub Capability Mapping

| Stage | DataHub Current Capability | Status |
|---|---|---|
| Business context search | Search / MCP / Context Documents | Implemented |
| Published-only generated context | Context Activation | Cloud Public Beta |
| Metric / semantic discovery | Metrics / Semantic Models / Context | Implemented / growing |
| Schema grounding | Metadata model / MCP | Implemented |
| Glossary resolution | Glossary / Documents | Implemented |
| Query patterns | Query history / Context Intelligence | Cloud Public Beta for generation |
| Lineage | Table / column / cross-platform | Implemented |
| Freshness / quality | Assertions / Incidents / Observability | Implemented, Cloud richer |
| Context behavioral eval | Metadata Evals | Cloud Public Beta |
| SQL workflow skill | datahub-sql-workflow | Context Activation |
| Text-to-SQL reference agent | DataHub Analytics Agent | Open-source reference implementation |
| Warehouse execution | Agent / external runtime | External execution |
| Runtime authorization | Warehouse / policy system | External |
| Context write-back | MCP / proposals / documents | Implemented / workflow varies |
| Agent audit | Registry / task traces | Partial / Cloud richer |
| Full decision replay | Not evidenced | Unclear |

---

# 26. What DataHub Solves vs What It Does Not

## DataHub can help resolve

- what “Enterprise Customer” means；
- what “Net Revenue” means；
- which datasets are relevant；
- common / validated join patterns；
- who owns metric / data；
- lineage；
- quality / incidents；
- published business context；
- prior query behavior。

## DataHub should not be treated as automatically solving

- final warehouse authorization；
- every metric execution engine；
- all conflict resolution；
- fiscal-period ambiguity unless context exists；
- stale context if source/eval pipeline is incomplete；
- complete historical decision replay。

这个 distinction 很重要。

---

# 27. Failure Test A — Two Revenue Definitions

如果：

~~~text
Finance Net Revenue
Growth Net Revenue
~~~

都 published。

系统应该：

- expose conflict；
- ask / infer domain；
- resolve by authority / task purpose。

如果 search simply returns one：

> case fails epistemically even if SQL succeeds.

这是我们 Reference Architecture 超出 DataHub 当前公开产品保证的部分。

---

# 28. Failure Test B — Freshness Incident

如果：

~~~text
refunds freshness = FAILED
~~~

Agent 仍给一个 authoritative revenue number：

> case fails.

合理策略：

~~~text
high-risk finance reporting:
  block / disclose

exploratory analytics:
  maybe answer with caveat
~~~

这说明：

> action risk affects epistemic fail-closed policy.

---

# 29. Failure Test C — View Scope Correct, Runtime Principal Too Broad

Agent View 只允许 Finance data。

但 Snowflake plugin 使用 ACCOUNTADMIN。

如果 Agent 被 prompt injection 诱导访问 HR：

> View 本身不能保证 runtime protection.

这验证 Phase 2 的 Agent Governance 结论：

> Context scope != runtime authorization.

---

# 30. Failure Test D — Correct Context, Wrong Semantic Execution

Agent 找到正确 Net Revenue definition。

但生成 SQL 出现 fan-out。

结果错误。

说明：

> Context correctness != execution correctness.

这验证：

> Semantic Plane 不能被 Context Plane 替代。

---

# 31. Failure Test E — Correct Number, No Evidence

Agent 返回数字完全正确，但无法回答：

- metric version；
- tables；
- freshness；
- query id；
- authority。

这种结果：

> 可能有 factual correctness，但没有 enterprise trustability。

对于生产 Agent：

> answer correctness 与 answer auditability 是两个不同质量维度。

---

# 32. End-to-End Trust Model

我们最终可以把这个问题表示成：

~~~mermaid
flowchart LR
    I[Intent Correct]
    C[Context Correct]
    S[Semantic Correct]
    P[Policy Correct]
    D[Data Fresh]
    E[Execution Correct]
    A[Answer Correct]
    T[Trustable]

    I --> C --> S --> P --> D --> E --> A --> T
~~~

任何一层失败：

> 最终数字都可能错误或不可用。

所以 Analytics Agent 的可靠性不是：

~~~text
LLM accuracy
~~~

而是：

~~~text
End-to-end system correctness
~~~

---

# 33. DataHub 在这条链里的真实角色

DataHub 主要加强：

~~~text
Context Correctness
+
Trust Evidence
+
Discovery
+
Governance
+
Lineage
+
Operational Signals
~~~

它不是：

~~~text
entire analytics runtime
~~~

这反而是一个健康的架构边界。

---

# 34. DataHub Analytics Agent 为什么是一个有价值的 Reference Implementation？

它的重要性不在 UI。

官方自己也写：

> UI is designed to be replaced; the context patterns underneath are designed to be kept.

它展示的核心 pattern：

~~~text
Question
-> context enrichment via DataHub MCP / Agent Context Kit
-> SQL generation
-> warehouse execution
-> visualization
-> context write-back when gaps found
~~~

并且：

> works with DataHub OSS and DataHub Cloud.

这说明 Phase 2 的 OSS/Cloud 判断再次被验证：

> Core substrate 已足以支持一个真实 talk-to-data Agent；Cloud 可以通过更丰富 / validated context 提高这一模式的 operating maturity.

---

# 35. Case Verdict

这个 case 验证了我们 Phase 1 的五 Plane 模型。

~~~mermaid
flowchart TB
    CTX[Context Plane<br/>What should I trust/use?]
    SEM[Semantic Plane<br/>How should it be computed?]
    POL[Policy Plane<br/>Am I allowed?]
    DATA[Data Plane<br/>Execute]
    AG[Agent Plane<br/>Interpret / plan / explain]

    AG --> CTX
    AG --> SEM
    AG --> POL
    SEM --> DATA
    POL --> DATA
    DATA --> AG
~~~

其中：

> DataHub 的核心价值集中在 Context Plane，并提供一些 Agent Plane / governance surface。

这比把 DataHub 定义成“Text-to-SQL tool”更准确。

---

# 36. 最值得保留的 Case Insight

用户问：

> 为什么我应该相信这个数字？

实际上强迫架构回答：

~~~text
Meaning
Authority
Freshness
Lineage
Execution
Policy
Evidence
~~~

所以：

> **“Explain why I should trust this answer” 是测试 Context Platform 是否真实存在的最好问题之一。**

一个只有 RAG / schema / vector search 的 Agent 很难完整回答。

---

# 37. 下一 Case 建议

## Case 02 — Schema Change Incident Agent

场景：

~~~text
upstream schema changes
-> dashboard breaks
-> Agent investigates lineage
-> identifies owner
-> opens GitHub fix
-> verifies deployment
-> writes incident resolution back
~~~

它会重点验证：

- operational context；
- lineage；
- write actions；
- Decisions；
- policy；
- read-back verification；
- context write-back；
- incident provenance。

这个 case 会补足 Case 01 偏 analytics/read-heavy 的不足。

---

# Sources

- DataHub Docs, Activate Context  
  https://docs.datahub.com/docs/managed-datahub/context/activate-context

- DataHub, Introducing DataHub Analytics Agent  
  https://datahub.com/blog/datahub-analytics-agent/

- DataHub, Inside the Context Platform — June 2026 Town Hall  
  https://datahub.com/blog/inside-the-context-platform-june-2026-town-hall-highlights/

- DataHub, Context Platform for AI Agents  
  https://datahub.com/products/context-platform/

- DataHub Docs, Data Quality & Observability  
  https://docs.datahub.com/docs/features/feature-guides/observe

- DataHub Docs, MCP  
  https://docs.datahub.com/docs/features/feature-guides/mcp
