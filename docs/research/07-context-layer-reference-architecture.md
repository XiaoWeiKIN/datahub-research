# 07 — Context Layer Reference Architecture

## 从 DataHub 学习成果收束成一套 AI-native Context Layer 参考架构

**Status:** 第一版  
**Focus:** architecture synthesis / responsibilities / trust boundaries / SLO / failure modes  
**Updated:** 2026-09-28

---

# 0. 目标

前六篇分别回答了：

1. 为什么需要 Context Layer；
2. Context Graph 与 Knowledge Graph 的关系；
3. Context Layer 与 Semantic Layer 的边界；
4. Freshness / Provenance / Invalidation；
5. Agent Read / Write Context；
6. Human + Agent Shared Truth Plane。

这一篇不再比较术语。

目标是：

> **如果今天不看 DataHub 的具体产品形态，而从零设计一个 production-grade、AI-native Context Layer，它应该由哪些组件组成？**

我们希望得到一套可以用来：

- 理解 DataHub；
- 比较其他 Context / Metadata / Knowledge Graph 产品；
- 设计自有 Context Platform；
- 审查 Agent architecture；

的 reference architecture。

---

# 1. 总体架构：五个 Plane

最终模型不是“所有东西都塞进 Context Platform”。

而是五个职责平面：

~~~mermaid
flowchart TB
    subgraph DP[Data / Execution Plane]
        WH[Warehouse / Lakehouse]
        OPS[Operational Systems]
        DOCS[Docs / SaaS / Repos]
    end

    subgraph EP[Epistemic / Context Plane]
        OBS[Observation & Ingestion]
        ID[Identity / Entity Resolution]
        ASSERT[Context Assertions]
        GRAPH[Context Graph]
        TRUST[Trust / Provenance / Freshness]
        REC[Reconciliation / Invalidation]
        PUB[Proposal / Publication]
        PROJ[Projection / Retrieval]
    end

    subgraph SP[Semantic Execution Plane]
        MODEL[Semantic Models]
        COMP[Metric / Query Compiler]
    end

    subgraph PP[Identity / Policy Plane]
        PRINCIPAL[Principal / Delegation]
        PDP[Policy Decision]
        PEP[Policy Enforcement]
    end

    subgraph AP[Agent Plane]
        AG[Agents]
        TASK[Tasks]
        MEM[Private / Task Memory]
        TOOL[Tools / MCP Clients]
    end

    DP --> OBS
    OBS --> ID --> ASSERT --> GRAPH
    GRAPH --> TRUST
    TRUST --> REC
    REC --> PUB
    PUB --> PROJ

    MODEL <--> GRAPH
    PROJ --> AG

    AG --> COMP
    COMP --> PDP
    PRINCIPAL --> PDP
    PDP --> PEP --> DP

    AP --> ASSERT
~~~

一句话总结：

> **Context Plane 决定“知道什么、为什么相信、当前是否适用”；Semantic Plane 决定“怎么算”；Policy Plane 决定“是否允许”；Data Plane 负责“真实存储/执行”；Agent Plane 负责“解释、规划和行动”。**

---

# 2. Context Plane 内部的核心数据流

Context Plane 自己可以收缩成：

~~~mermaid
flowchart LR
    S[Sources]
    O[Observe]
    I[Identity]
    A[Assertion]
    G[Graph]
    T[Trust Envelope]
    R[Reconcile]
    P[Publish]
    V[Projection]
    C[Consumer]

    S --> O --> I --> A --> G --> T --> R --> P --> V --> C
~~~

这里每一步解决不同问题。

如果一个 Context 产品只解决其中一两步，它未必构成完整 Context Layer。

---

# 3. Component 1 — Source & Observation Plane

## 职责

从企业真实系统持续获取 context evidence。

来源至少包括：

### Technical Sources

- warehouses；
- lakes；
- dbt；
- orchestration；
- BI；
- semantic layers；
- APIs；
- code repositories。

### Operational Sources

- quality / observability；
- incidents；
- query logs；
- usage；
- pipeline state；
- freshness state。

### Business / Knowledge Sources

- Notion；
- Confluence；
- Google Docs；
- GitHub docs；
- policies；
- runbooks；
- decision records。

### Organizational Sources

- HR / directory；
- team ownership；
- IAM；
- domain mapping。

---

# 4. Observation 和 Truth 必须分开

Source connector 看到的只是：

> observation

不是自动的 truth。

例如：

~~~text
Observation:
Table orders_v2 was queried 8,000 times this month.
~~~

这不等于：

~~~text
Truth:
orders_v2 is the canonical orders table.
~~~

因此 observation layer 应该尽量保留：

- producer；
- event time；
- observed time；
- source identity；
- raw evidence reference；
- connector health；
- source version。

原则：

> **Ingestion produces evidence before it produces knowledge.**

---

# 5. Component 2 — Identity & Entity Resolution

所有 Context Graph 问题最终都会碰到 identity。

例如：

~~~text
snowflake://prod/finance/orders
dbt://model/orders
looker://explore/orders
notion://page/orders-definition
metric://net_revenue
~~~

这些对象可能描述同一业务概念，也可能只是相关。

Context Layer 需要稳定 identity：

~~~text
Entity Identity
├── canonical id
├── source ids
├── aliases
├── entity type
├── namespace
├── lifecycle state
└── merge / split history
~~~

没有 identity，Graph 只会变成：

> connected duplicates.

---

# 6. Identity Resolution 不能过度自动化

“两个名字相似的对象是同一个东西”是高风险推断。

例如：

~~~text
customer
customers
customer_360
active_customer
customer_metric
~~~

它们可能不是同一 entity。

所以 entity resolution 应区分：

- deterministic match；
- declared equivalence；
- inferred similarity；
- human-approved merge。

Agent / embedding 的相似度只能产生：

> candidate identity relation

不能直接制造 canonical identity。

---

# 7. Component 3 — Context Assertion Store

我们前面已经得到：

> Context 的基本单位更适合是 Assertion，而不是裸属性。

参考模型：

~~~text
ContextAssertion
├── assertion_id
├── subject
├── predicate
├── object / value
├── scope
├── domain
├── source
├── evidence
├── producer
├── observed_at
├── valid_from
├── valid_until
├── generated_at
├── authority
├── confidence
├── epistemic_state
├── publication_state
├── dependencies
└── supersedes
~~~

例如：

~~~text
subject: metric/net_revenue
predicate: authoritative_for
object: board_reporting

domain: finance
authority: finance-controller
valid_from: 2026-07-01
publication_state: published
~~~

---

# 8. 为什么 Assertion 比 Property 更重要？

普通属性：

~~~text
owner = finance
~~~

无法表达：

- 谁说的？
- 什么时候成立？
- 对什么 scope 成立？
- 有没有冲突？
- 现在是否还有效？

Assertion model 允许：

~~~mermaid
graph LR
    M[Net Revenue]
    A1[Assertion: Finance owns metric]
    A2[Assertion: Growth uses alternate definition]
    A3[Assertion: Board reporting uses Finance definition]

    M --> A1
    M --> A2
    M --> A3
~~~

企业现实里的 plurality 才能被保留下来。

---

# 9. Component 4 — Context Graph

Context Graph 的职责不是简单可视化关系。

它至少承担三种 graph：

## Semantic Graph

~~~text
metric -> dataset
term -> metric
document -> entity
owner -> asset
~~~

## Dependency Graph

~~~text
schema -> semantic model -> metric -> dashboard -> agent skill
~~~

用于 impact / invalidation。

## Provenance Graph

~~~text
source evidence -> generation activity -> proposal -> validation -> publication
~~~

因此更完整的 Context Graph 是：

~~~mermaid
flowchart TB
    SEM[Semantic Relationships]
    DEP[Dependency Relationships]
    PROV[Provenance Relationships]
    ORG[Authority / Organizational Relationships]

    SEM --> G[Context Graph]
    DEP --> G
    PROV --> G
    ORG --> G
~~~

---

# 10. Component 5 — Provenance & Temporal Model

所有 assertion 都必须能回答：

### Provenance

- source 是什么？
- 谁生成？
- 使用了什么 evidence？
- 经过什么 transform / agent？
- 谁批准？

### Temporal

- 现实中何时开始成立？
- Context Layer 何时观察到？
- 何时生成？
- 何时发布？
- 何时失效？

参考：

~~~text
valid_time       = reality time
observed_time    = system learned it
generated_time   = derived result produced
published_time   = became shared context
invalidated_time = stopped being usable
~~~

原则：

> **updated_at 不足以支持 Agent audit。**

---

# 11. Component 6 — Trust Envelope

Context retrieval 不应该只返回 value。

应该返回：

~~~text
TrustEnvelope
├── content
├── scope
├── authority
├── provenance
├── freshness
├── dependency_health
├── epistemic_state
├── publication_state
├── access_scope
└── evidence_refs
~~~

Agent-facing output：

~~~text
Metric: Net Revenue
Applicable scope: Board Reporting
Authority: Finance
Status: Published / Valid
Freshness: within SLA
Dependency health: healthy
Evidence:
  - finance-policy-v17
  - dbt metric net_revenue
  - certified dashboard ...
~~~

这比“confidence 0.94”更可操作。

---

# 12. Component 7 — Epistemic State Machine

Context 不能只有 exists / deleted。

至少需要：

~~~mermaid
stateDiagram-v2
    [*] --> Candidate
    Candidate --> Valid
    Candidate --> Rejected
    Valid --> Stale
    Valid --> Suspect
    Valid --> Conflicted
    Stale --> Valid: refreshed
    Suspect --> Valid: corroborated
    Conflicted --> Valid: authority resolution
    Valid --> Invalid
    Stale --> Invalid
    Valid --> Superseded
~~~

可以采用类似：

- CANDIDATE
- VALID
- STALE
- SUSPECT
- UNVERIFIED
- CONFLICTED
- INVALID
- SUPERSEDED

Agent 根据状态选择不同策略。

---

# 13. Component 8 — Reconciliation & Invalidation Engine

这是 Context Layer 最像 control plane 的部分。

输入：

- source change；
- policy change；
- schema change；
- quality incident；
- ownership change；
- human declaration；
- observed behavior；
- agent behavior。

处理：

~~~mermaid
flowchart LR
    CHANGE[Change Event]
    DEP[Dependency Traversal]
    IMPACT[Impact Set]
    CLASS[Classify Consequence]
    ACTION[Recompute / Invalidate / Review]

    CHANGE --> DEP --> IMPACT --> CLASS --> ACTION
~~~

不同依赖 edge 必须有不同 invalidation semantics。

例如：

~~~text
schema change
-> semantic model: MAY_INVALIDATE
-> metric: MAY_INVALIDATE
-> business definition: REVIEW
-> owner: NO_EFFECT
-> SQL example: LIKELY_INVALID
~~~

---

# 14. Context Layer 需要 Typed Dependency Semantics

普通 graph 只知道：

~~~text
A -> B
~~~

生产 Context Layer 还需要：

~~~text
A --[depends_on: schema_contract]--> B
A --[supports: evidence]--> C
A --[governed_by]--> D
A --[derived_from]--> E
~~~

并知道：

> 哪种 source change 会影响哪一种关系。

这接近 build graph / reactive system，而不仅是 knowledge graph。

---

# 15. Component 9 — Proposal & Publication Plane

前面已经形成：

> Generation Plane != Truth Plane

参考生命周期：

~~~mermaid
flowchart LR
    OBS[Observation]
    CAND[Candidate]
    PROP[Proposal]
    EVAL[Evidence / Evals]
    AUTH[Authority Review]
    PUB[Published]
    SUP[Superseded / Invalid]

    OBS --> CAND --> PROP --> EVAL --> AUTH --> PUB
    PUB --> SUP
~~~

Agent 可以生成 Candidate / Proposal。

只有符合 policy / authority 的内容进入 Published Context。

DataHub 当前的 context proposal / publish 模型就是这类架构的一个现实实现。其当前文档强调默认生成内容不自动暴露给 Agent，发布后才进入 Agent 可见范围。 

---

# 16. Component 10 — Authority Model

Authority 不能只有一个 owner 字段。

至少要区分：

~~~text
source authority
domain authority
business authority
policy authority
publication authority
execution authority
~~~

例如：

- Warehouse 对 schema 有 source authority；
- Finance 对 Net Revenue 有 business authority；
- Security 对 sensitivity 有 policy authority；
- Data Expert 对某 Context Document 有 publication authority；
- Runtime Policy Engine 对最终 allow / deny 有 execution authority。

原则：

> **Authority is typed and scoped.**

---

# 17. Component 11 — Projection & Retrieval Layer

Shared Truth Plane 不等于大家拿相同 payload。

Context Layer 应该支持：

~~~text
truth substrate
-> projection
-> retrieval
-> consumer
~~~

不同 projection：

### Human Projection

- description；
- charts；
- owners；
- documents；
- lineage UI。

### Agent Projection

- canonical ids；
- structured assertions；
- evidence；
- freshness；
- policy context；
- machine-readable relations。

### Policy Projection

- classification；
- ownership；
- purpose；
- sensitivity；
- delegation context。

---

# 18. Retrieval 不是普通 Search

Context retrieval 的 ranking 应考虑：

~~~text
relevance
+ authority
+ freshness
+ domain applicability
+ epistemic state
+ policy scope
+ provenance quality
~~~

传统 vector similarity 只解决其中一个维度。

因此：

~~~text
Context Retrieval
!= Vector Search
~~~

更接近：

> constrained epistemic retrieval.

---

# 19. Component 12 — Context Activation

Activation 是 Context Layer 对外接口。

至少包含：

- Search / UI；
- API / SDK；
- MCP；
- event subscription；
- semantic interfaces。

DataHub 当前 Context Activation 明确采用：published context 通过 MCP 提供给 Agent，并建议先通过 eval 验证这些 context 是否真实改变 Agent 行为。citeturn530554search0turn530554search7

一个重要设计原则：

> **Activation 不只是 delivery，还应暴露 trust metadata。**

---

# 20. MCP 在 Reference Architecture 里的位置

~~~mermaid
flowchart LR
    CTX[Published Context]
    PROJ[Agent Projection]
    MCP[MCP Server]
    AG[Agent]

    CTX --> PROJ --> MCP --> AG
~~~

MCP 负责：

- discovery；
- tool invocation；
- structured access。

MCP 不负责：

- authority；
- freshness；
- truth；
- provenance；
- semantic correctness。

它只是 activation protocol。

---

# 21. Component 13 — Semantic Plane Integration

Context Layer 不应该重新实现 metric compiler。

它应该 ingest / reference：

- semantic model；
- metric；
- dimension；
- join；
- query semantics。

然后关联：

~~~mermaid
graph LR
    M[Metric]
    SM[Semantic Model]
    D[Dataset]
    Q[Quality]
    O[Owner]
    P[Policy]

    SM --> M
    M --> D
    D --> Q
    O --> M
    P --> M
~~~

Agent 路径：

~~~text
Context Plane:
select trusted metric

Semantic Plane:
execute metric
~~~

---

# 22. Component 14 — Identity / Policy Plane Integration

Context Plane 与 Policy Plane 的接口必须清晰。

Context 可以提供：

- asset classification；
- sensitivity；
- ownership；
- purpose；
- business domain；
- user / agent context。

Policy Engine 决定：

- allow；
- deny；
- mask；
- filter；
- require approval。

OPA 的经典架构将 Policy Decision Point 与实际 Policy Enforcement Point 分离，这正好说明“知道政策”与“执行政策”是不同职责。citeturn530554search3turn530554search6

原则：

> **Context informs policy; policy governs execution.**

---

# 23. Component 15 — Agent Read Contract

Agent 读取 context 时，请求应该携带：

~~~text
ContextReadRequest
├── agent_id
├── principal
├── task
├── purpose
├── domain
├── requested_entity / question
├── time
└── requested_evidence_level
~~~

返回：

~~~text
ContextReadResponse
├── assertions
├── provenance
├── authority
├── freshness
├── conflicts
├── epistemic_state
├── access_scope
└── retrieval_trace
~~~

这样才能支持后续 audit。

---

# 24. Component 16 — Agent Write Contract

写入时不应该直接 PATCH truth。

参考：

~~~text
ContextWriteProposal
├── agent_identity
├── principal
├── task_id
├── intended_change
├── semantic_risk_class
├── target
├── evidence
├── source_context_versions
├── expected_effect
├── verification_method
├── rollback_plan
└── required_authority
~~~

write 后需要：

~~~text
actual state
+ independent read-back
+ provenance event
+ publication / approval record
~~~

原则：

> **Agents propose. Evidence verifies. Authorities publish.**

---

# 25. Component 17 — Decision Provenance / Audit

Production Agent audit 不应该只保存 chat transcript。

需要保存：

~~~text
DecisionTrace
├── principal
├── agent_id / version
├── model
├── task
├── retrieved context ids + versions
├── semantic model version
├── policy decision
├── tool calls
├── human decisions
├── writes
└── outcome
~~~

这样才能回答：

> Agent 为什么在当时做了这个决定？

---

# 26. Component 18 — SLO Model

Context Platform 不应该只宣传 “real-time”。

应该定义端到端 SLO。

## Observation SLO

~~~text
source change -> detected
~~~

## Propagation SLO

~~~text
detected -> graph visible
~~~

## Invalidation SLO

~~~text
dependency change -> stale/invalid mark
~~~

## Publication SLO

~~~text
proposal -> authoritative publication
~~~

## Retrieval SLO

~~~text
agent query -> trusted context response
~~~

## Audit SLO

~~~text
decision -> trace available
~~~

---

# 27. Freshness SLO 必须按 Context Type

例如：

| Context | Target |
|---|---|
| Active incident | minutes |
| Pipeline / table health | minutes |
| Schema | minutes-hours |
| Usage | hours-days |
| Ownership | hours-days |
| Business definition | long-lived + periodic revalidation |
| Regulation / policy | immediate change propagation |

不是一个统一 “freshness = 5 min”。

---

# 28. Health Model 必须区分 Silence

Source 没变化和 source 挂了不能相同。

建议：

~~~text
SourceHealth
├── last_successful_observation
├── expected_frequency
├── lag
├── backlog
├── coverage
└── state
~~~

state：

- HEALTHY
- LAGGING
- SILENT
- FAILED
- PARTIAL

然后 assertion freshness 继承 source health。

---

# 29. Failure Mode 1 — Stale-but-Confident Context

症状：

> Agent 使用过期定义但回答非常肯定。

原因：

- freshness 未进入 retrieval；
- source silence 被当成 no-change；
- stale content 没 invalidation。

防护：

- epistemic states；
- freshness envelope；
- source health；
- dependency invalidation。

---

# 30. Failure Mode 2 — Authority Collapse

症状：

> Agent-generated proposal 与 Finance-approved definition 在 retrieval 中同权。

原因：

- graph 只有 content，没有 typed authority。

防护：

- authority hierarchy；
- publication state；
- domain scope；
- human / system-of-record precedence。

---

# 31. Failure Mode 3 — Context Poisoning

症状：

> 恶意或错误输入进入 persistent context，影响未来多个 Agent。

防护：

- proposal/truth boundary；
- provenance roots；
- no direct memory promotion；
- evidence gates；
- rollback；
- blast-radius-aware review。

---

# 32. Failure Mode 4 — Context Islands

症状：

- Agent A 用 DataHub；
- Agent B 自己建 Pinecone；
- Agent C 用 prompt hard-code；
- humans 看 Confluence。

结果：

> no shared debugging model.

防护：

- shared substrate；
- scoped projections；
- standard activation；
- versioned assertions。

---

# 33. Failure Mode 5 — Over-centralization

症状：

> 中央 Context Team 负责写所有业务定义。

结果：

- bottleneck；
- stale context；
- domain ownership 丢失。

防护：

> centralized platform + federated authority

---

# 34. Failure Mode 6 — Context Platform Becomes Architecture Blob

症状：

Context Platform 开始负责：

- IAM；
- query execution；
- model runtime；
- secrets；
- all policy；
- all business data。

结果：

- 责任边界模糊；
- impossible to replace；
- duplicated infrastructure。

防护：

严格保持 Plane boundary：

~~~text
Context = know/trust
Semantic = compute
Policy = allow
Data = execute
Agent = reason/act
~~~

---

# 35. Failure Mode 7 — Retrieval Without Conflict Awareness

症状：

系统找到一个高相似度 definition，但忽略同 domain 的冲突定义。

防护：

retrieval 需要：

- conflict detection；
- authority resolution；
- scope matching；
- temporal validity。

---

# 36. Failure Mode 8 — Unknown 被错误表达成 False

例如：

connector 没检测到 lineage。

系统输出：

~~~text
no upstream dependency
~~~

而真实情况是：

~~~text
unknown because source silent
~~~

这是危险的 epistemic bug。

所以 unknown / unverified 必须是一等状态。

---

# 37. DataHub 当前能力如何映射到 Reference Architecture？

注意：下面是架构映射，不是等价实现声明。

| Reference Component | DataHub 当前对应能力 |
|---|---|
| Observation / Ingestion | Metadata ingestion, connectors, events |
| Identity | URN / entity model |
| Graph | Metadata / Context Graph |
| Technical context | schema, lineage, usage, quality |
| Business context | Context Documents, glossary, domains |
| Context generation | Context Intelligence / Curator |
| Proposal / publication | Context Hub proposals / publish |
| Activation | Search / API / SDK / MCP |
| Agent plane | Agents / Tasks / Decisions |
| Agent scope | Views / scoped agents |
| Trust signals | ownership, assertions, freshness, certification |
| Human review | Data Expert / SME review |
| Semantic integration | semantic models / metrics direction |
| Audit | traces / agent governance surfaces |

DataHub 文档目前明确将 Context generation 做 domain/container scope，默认 auto-publish 关闭，并建议先建立 eval 再逐步发布；这体现了 reference architecture 中 blast-radius control 与 proposal/publish boundary。citeturn530554search1turn530554search7

---

# 38. DataHub 当前仍不是整个 Reference Architecture

需要避免把研究模型倒推成：

> DataHub 已完整实现以上所有组件。

并没有足够证据支持这种说法。

特别需要继续验证：

- generalized assertion-level temporal model；
- typed invalidation semantics；
- conflict resolution primitives；
- multi-dimensional authority model；
- full decision provenance；
- external policy integration depth；
- Agent write transaction / rollback model；
- context dependency SLO；
- cross-domain federation。

Reference Architecture 是：

> 从 DataHub + 外部系统设计经验抽象出的目标模型。

不是 DataHub feature checklist。

---

# 39. 与 DataHub 当前 Agent Architecture 的连接

DataHub Agents 当前支持：

- custom instructions；
- tools；
- external MCP plugins；
- scoped View；
- manual/scheduled/event-triggered tasks；
- human Decisions。citeturn530554search2

这说明 Agent Plane 已经在向：

~~~text
scoped context
+ governed tools
+ human checkpoint
~~~

演化。

而 Context Activation 当前要求 published context 才对 Agent 可见，也说明 Agent Plane 与 Truth Plane 之间已经存在明确 publication boundary。citeturn530554search0turn530554search7

---

# 40. Reference Architecture 的核心原则

最终压缩成十条：

## 1. Evidence before Truth

Ingestion 先产生 evidence，再产生 assertion。

## 2. Assertions, not Naked Properties

所有重要 context 都要带 scope / provenance / time / authority。

## 3. Relationships are First-class

Context 的价值大量存在于关系和路径。

## 4. Freshness is a Contract

不是 updated_at。

## 5. Unknown is a State

不能把没有观察到等同于 false。

## 6. Authority is Typed

Finance、Security、Warehouse、Human Reviewer 拥有不同解释权。

## 7. Proposal != Truth

Agent-generated context 先进入 proposal plane。

## 8. Shared Truth != Shared Access

统一底座，按 consumer 投影视图。

## 9. Context != Authorization

Context 提供认知，Policy Plane 决定执行许可。

## 10. Reconciliation is Continuous

系统持续比较 declared / observed / system / agent reality。

---

# 41. 一个最小可行 Context Layer

如果从零建设，不需要一次做完所有组件。

## Phase 1 — Shared Metadata Graph

- stable identity；
- schema；
- lineage；
- ownership；
- API；
- search。

## Phase 2 — Trust

- quality；
- freshness；
- provenance；
- certification；
- source health。

## Phase 3 — Business Context

- glossary；
- metrics；
- docs；
- domain knowledge；
- authority。

## Phase 4 — Context Lifecycle

- proposal；
- publish；
- invalidation；
- revalidation；
- versioning。

## Phase 5 — Agent Activation

- MCP / API；
- scoped projection；
- context retrieval；
- trust envelope。

## Phase 6 — Agent Contribution

- write proposal；
- decisions；
- read-back verification；
- provenance；
- security boundaries。

## Phase 7 — Reconciliation Control Plane

- drift detection；
- typed dependencies；
- policy integration；
- enterprise-scale SLO / audit。

---

# 42. 最小架构图

~~~mermaid
flowchart LR
    SRC[Enterprise Sources]
    ING[Observe]
    GRAPH[Context Graph]
    GOV[Trust + Authority]
    PUB[Publish]
    MCP[MCP / API]
    AG[Agents]

    SRC --> ING --> GRAPH --> GOV --> PUB --> MCP --> AG
~~~

如果系统没有：

- trust；
- freshness；
- provenance；
- publication boundary；

那么它更像：

> metadata/search/RAG system

还不足以成为我们定义的 production Context Layer。

---

# 43. 完整架构图

~~~mermaid
flowchart TB
    subgraph SOURCES[Enterprise Reality]
        DW[Data Platforms]
        BI[BI / Semantic]
        OBS[Quality / Observability]
        DOC[Docs / Repos]
        ORG[Org / IAM]
    end

    subgraph CONTEXT[Epistemic Context Plane]
        ING[Observation]
        ER[Entity Resolution]
        AS[Assertions]
        KG[Context Graph]
        PR[Provenance]
        TM[Temporal State]
        AU[Authority]
        INV[Invalidation]
        REC[Reconciliation]
        PROP[Proposal]
        PUB[Published Truth]
        RET[Scoped Retrieval]
    end

    subgraph AGENT[Agent Plane]
        A[Agent]
        TASK[Task]
        DEC[Human Decision]
        MEMORY[Private Memory]
    end

    subgraph SEMANTIC[Semantic Plane]
        SM[Semantic Model]
        Q[Query Runtime]
    end

    subgraph POLICY[Policy Plane]
        PID[Principal / Delegation]
        PDP[Decision]
        PEP[Enforcement]
    end

    SOURCES --> ING --> ER --> AS --> KG

    KG --> PR
    KG --> TM
    KG --> AU

    PR --> INV
    TM --> INV
    AU --> REC
    INV --> REC

    REC --> PROP --> PUB --> RET

    RET --> A
    A --> TASK
    TASK --> DEC
    MEMORY --> A

    A --> SM --> Q
    A --> PID --> PDP
    Q --> PDP --> PEP --> SOURCES

    SEMANTIC --> KG
    AGENT --> AS
~~~

---

# 44. DataHub 给我们的真正启发

经过七篇研究，DataHub 最值得学习的已经不是：

> 如何实现一个 Data Catalog。

而是它长期积累的几个选择，在 AI 时代突然组合成了新的架构意义：

### Graph-first metadata

让 relationships / lineage 成为自然基础。

### Active metadata

让 context 有机会持续反映现实。

### Governance as metadata

让 ownership / quality / policy 能进入 Agent context。

### Human + Machine consumers

把 UI-first catalog 推向 machine-consumable substrate。

### Context proposals / publication

开始建立 machine-generated knowledge 的 authority boundary。

### MCP / Agent activation

让 Context Plane 真正进入 runtime AI workflows。

所以 DataHub 的演进可以重新描述为：

~~~mermaid
flowchart LR
    CAT[Catalog]
    META[Metadata Platform]
    ACTIVE[Active Metadata Graph]
    CONTEXT[Context Platform]
    EP[Epistemic Control Plane]

    CAT --> META --> ACTIVE --> CONTEXT --> EP
~~~

最后一步目前仍然是我们的架构推论，不是 DataHub 官方产品定义。

---

# 45. 下一步

下一阶段不再扩主架构。

建议进入：

## 08 — Context Platform Failure Modes

用 failure-oriented design 检验上面的 reference architecture：

- stale context；
- authority conflict；
- source silence；
- poisoned context；
- semantic drift；
- context island；
- over-centralization；
- identity collision；
- false provenance；
- Agent self-reinforcement；
- policy/context mismatch；
- publication blast radius。

然后最后再做：

## 09 — DataHub Design Philosophy — Synthesis

把整个仓库压缩成一篇真正的“为什么 DataHub 值得在 AI 时代研究”。

---

# Sources

## DataHub

- DataHub Docs, **Configure Context Generation**  
  https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation

- DataHub Docs, **Validate Context Proposals**  
  https://docs.datahub.com/docs/managed-datahub/context/review-context-proposals

- DataHub Docs, **Activate Context**  
  https://docs.datahub.com/docs/managed-datahub/context/activate-context

- DataHub Docs, **Agents**  
  https://docs.datahub.com/docs/features/feature-guides/agents

- DataHub, **What Is a Context Graph?**  
  https://datahub.com/blog/context-graph/

## External architecture references

- Kubernetes, **Kubernetes Components / Control Plane**  
  https://kubernetes.io/docs/concepts/overview/components/

- Open Policy Agent, **How to Deploy OPA**  
  https://www.openpolicyagent.org/docs/deploy

- Open Policy Agent, **Management APIs and Architecture**  
  https://www.openpolicyagent.org/docs/management-introduction

- OpenLineage, **Object Model**  
  https://openlineage.io/docs/spec/object-model/
