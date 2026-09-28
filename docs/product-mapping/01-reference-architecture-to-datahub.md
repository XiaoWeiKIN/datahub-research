# 01 — Reference Architecture → DataHub Current Product

## DataHub 到底实现到了哪里？

**Snapshot:** 2026-09-28  
**Reference:** [Context Layer Reference Architecture](../research/07-context-layer-reference-architecture.md)

---

# 0. Summary

先给出当前判断。

DataHub 已经非常明确地实现了 Context Layer 的“骨架”：

~~~mermaid
flowchart LR
    ING[Ingestion]
    GRAPH[Metadata / Context Graph]
    GOV[Governance + Quality + Lineage]
    PUB[Context Proposal / Publish]
    MCP[MCP / API]
    AG[Agents]

    ING --> GRAPH --> GOV --> PUB --> MCP --> AG
~~~

当前成熟度最高的是：

- metadata ingestion；
- stable entity / URN model；
- graph relationships；
- lineage；
- search / discovery；
- ownership / glossary / tags / domains；
- quality / assertions / incidents；
- API / SDK；
- MCP；
- Agent-facing scoped retrieval；
- Agent Registry。

2026 年新增、但仍在 Beta 阶段的是：

- Context Intelligence / generation；
- Context proposal / validation / publication；
- Context Activation；
- custom Agents / Tasks / Decisions。

而我们 Reference Architecture 中更严格的几个 primitive，目前从官方文档**不能证明 DataHub 已经作为统一通用机制实现**：

- generalized Context Assertion model；
- assertion-level valid time；
- typed multi-dimensional authority；
- generic dependency invalidation semantics；
- conflict-aware retrieval；
- source silence → epistemic state propagation；
- full Agent decision replay；
- cross-plane reconciliation。

因此目前最准确的判断不是：

> DataHub 已经完整实现 Context Layer Reference Architecture。

而是：

> **DataHub 已经实现了一个强 metadata/context substrate，并正在把 proposal lifecycle、Agent activation 和 Agent governance 加到这个 substrate 上；更严格的 epistemic control-plane primitives 仍有一部分是我们的目标架构推论。**

---

# 1. High-Level Matrix

| Reference Capability | DataHub Status | Availability | Current Evidence / Interpretation |
|---|---|---|---|
| Observation & Ingestion | **Implemented** | Core + Cloud | 广泛 connector / query history / usage / schema / lineage ingestion |
| Stable Identity | **Implemented** | Core + Cloud | Entity + key aspect + URN |
| Cross-source Entity Resolution | **Partial / Unclear** | Core + Cloud | 有 logical models / relationships，但未发现通用 entity-resolution authority workflow |
| General Context Assertion Model | **Partial** | Core + Cloud | Aspect/entity model很强，但不是我们定义的 assertion + scope + authority + valid-time 通用模型 |
| Metadata / Relationship Graph | **Implemented** | Core + Cloud | @Relationship、lineage、impact analysis |
| Technical Context | **Implemented** | Core + Cloud | schema / lineage / ownership / usage / query history |
| Operational Context | **Implemented / Cloud richer** | Core + Cloud | incidents、profiles；Cloud assertions/anomaly/health 更完整 |
| Business Context | **Implemented** | Core + Cloud / Cloud Context richer | glossary、domains、documents、semantic entities |
| Context Generation | **Beta** | Cloud Public Beta | Context Intelligence 从 query history / schema / BI/dbt 生成 semantic anchors |
| Proposal / Validation / Publication | **Beta** | Cloud Public Beta | proposal、eval、SME review、published-only activation |
| Provenance | **Partial** | Core + Cloud | lineage、change history、source relationships较强；通用 assertion derivation chain 未证实 |
| Temporal Validity | **Partial / Unclear** | Core + Cloud | time-based lineage/filter/history存在；通用 valid_from / invalidated_at assertion semantics 未证实 |
| Freshness / Source Health | **Partial** | Core + Cloud | freshness assertions、sync/ingestion health；通用 epistemic source-health propagation 未证实 |
| Authority Model | **Partial** | Core + Cloud | ownership、roles、policies、Data Expert review；typed authority hierarchy 未证实 |
| Reconciliation / Invalidation | **Partial / Unclear** | Cloud + Core primitives | automations、propagation、regeneration存在；通用 dependency invalidation engine 未证实 |
| Search / Retrieval | **Implemented** | Core + Cloud | search / GraphQL / MCP search |
| Scoped Projection | **Implemented** | Core + Cloud, Cloud richer | Views、service-account default View、scoped MCP、Agent View |
| Search Access Control | **Implemented / differentiated** | Cloud stronger | Cloud query-time filtering；OSS entity-page gating |
| MCP Activation | **Implemented** | Core + Cloud | self-hosted MCP + managed Cloud MCP |
| MCP Mutation | **Implemented** | Core + Cloud versions | tags / terms / owners / domains / descriptions / properties / docs 等写操作 |
| Governed Agent Proposals | **Implemented / Partial** | MCP + Cloud workflows | proposal tools和 Context Proposal workflow存在，但并非所有 mutation 都统一走 proposal |
| Semantic Model Catalog | **Implemented** | Core/Cloud metadata surface | Metric / semantic models 已是一等实体，v2.2 增强 lineage |
| Semantic Query Execution | **External** | External runtime | DataHub catalog/contextualize semantics；真正 metric/query runtime 在 dbt/Snowflake/Cube 等系统 |
| Metadata Policy / IAM | **Implemented** | Core + Cloud | DataHub metadata authorization / roles / policies |
| Runtime Data Authorization | **External** | External systems | Context 可提供信号，但 warehouse / policy engine 执行最终数据访问控制 |
| Agent Registry | **Implemented** | Cloud v2.1 | Agents / skills / tools 作为 governed versioned entities |
| Agent Runtime | **Private Beta** | Cloud | Agents / Tasks / Decisions |
| Human Decision Checkpoint | **Private Beta** | Cloud | Decisions |
| Agent Context Read | **Implemented** | Core + Cloud | MCP / Ask DataHub / scoped Views |
| Agent Context Write | **Implemented / Beta workflows** | Core MCP + Cloud | MCP mutation + proposal / review / publish |
| Agent Tool Audit | **Partial** | Cloud | Ask DataHub audit / Agent history；完整 decision-context snapshot 未证实 |
| Historical Decision Replay | **Unclear** | — | 未找到官方文档证明可按原 context versions 完整重放 Agent decision |
| Conflict-aware Retrieval | **Unclear** | — | glossary/version等能力存在，但没有找到通用 conflict-resolution retrieval contract |
| Context SLO Framework | **Partial / Unclear** | — | ingestion status / freshness / eval schedules存在，但没有统一 Context end-to-end SLO model |

---

# 2. Observation & Ingestion — Implemented

DataHub 最成熟的部分之一。

Context Generation 文档明确依赖 ingestion 提供：

- query entities；
- query usage statistics；
- schema metadata；
- dbt context；
- downstream Looker dashboards/charts。

DataHub Cloud 2.2 当前宣称 Context Platform 从 150+ sources 获取 metadata。

这和 Reference Architecture 的：

~~~text
Enterprise Reality
-> Observation
~~~

高度吻合。

## 但要注意

Ingestion 并不等于 truth。

DataHub 当前 architecture 仍主要把 source metadata 映射到 entity/aspect。

我们提出的：

> Observation -> Evidence -> Assertion

是比现有 DataHub metadata model 更严格的一层抽象。

**Status: Implemented substrate; assertion semantics Partial.**

Sources:

- https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation
- https://datahub.com/blog/datahub-cloud-v2-2/

---

# 3. Identity — Implemented

DataHub 的 entity model 本身非常适合 Context Platform。

核心结构：

~~~text
Entity
├── Key Aspect
├── URN
└── Aspects
~~~

Key Aspect 唯一识别 entity，并序列化为 URN。

Relationship annotation 会把 URN reference 变成 graph edge。

这解决了：

- stable entity identity；
- typed entity；
- cross-entity reference；
- relationship graph。

这也是 DataHub 从 Catalog 向 Context Graph 扩张时无需重建基础 identity system 的原因之一。

## Gap: Entity Resolution

Stable identity 不等于：

> 自动知道两个 source object 是否代表同一个 conceptual entity。

Logical Models 可以把多个 physical datasets 链接到一个 logical parent，是一种明确的 identity abstraction。

但没有找到证据表明 DataHub 当前存在一个通用：

~~~text
candidate equivalence
-> evidence
-> merge approval
-> split history
~~~

的 entity-resolution subsystem。

**Status: Identity Implemented; generalized resolution Partial / Unclear.**

Sources:

- https://docs.datahub.com/docs/metadata-modeling/extending-the-metadata-model
- https://docs.datahub.com/docs/features/feature-guides/logical-models/overview

---

# 4. Context Assertion Model — Partial

DataHub 的 Aspect Model 很强：

~~~text
entity
+ aspects
+ searchable fields
+ relationships
+ timeseries aspects
~~~

它可以表达 ownership、description、tags、schema、quality 等大量 context。

但我们 Reference Architecture 的 ContextAssertion 要求：

~~~text
subject
predicate
value
scope
authority
evidence
valid_from
valid_until
epistemic_state
publication_state
dependencies
~~~

当前官方材料没有证明这些字段存在一个统一、跨 context 类型的 assertion primitive。

DataHub 更接近：

> typed metadata aspects + specialized entities / workflows

而不是：

> universal epistemic assertion store.

因此：

**Status: Partial.**

这是一个很重要的差异，不应该因为 DataHub 有 graph 就忽略。

---

# 5. Graph & Relationships — Implemented

DataHub 原生 metadata model 的 @Relationship 会在 ingestion 时创建 entity edges。

Lineage 当前支持：

- table-level；
- column-level；
- pipelines；
- cross-platform；
- upstream/downstream；
- impact analysis；
- API / SDK。

Agent Registry 进一步把：

- aiAgent；
- Skill；
- API tool；
- MCP Service；
- Dataset；

连接到同一个 lineage graph。

所以我们提出的：

> relationships are first-class

在 DataHub 中不是愿景，而是成熟产品能力。

**Status: Implemented.**

Sources:

- https://docs.datahub.com/docs/features/feature-guides/lineage
- https://docs.datahub.com/docs/features/feature-guides/agent-registry

---

# 6. Technical / Operational / Business Context

## Technical Context — Implemented

包括：

- schema；
- lineage；
- query history；
- usage；
- ownership；
- service/API metadata。

## Operational Context — Implemented, Cloud 更丰富

DataHub 把 observability 定义为 metadata platform 的一等能力。

当前包括：

- Assertions；
- anomaly detection；
- incidents；
- data contracts；
- profiles；
- health dashboards；
- external quality integrations。

其中 active / ingestion-driven / anomaly assertions 等较丰富能力是 DataHub Cloud。

## Business Context — Implemented

已有：

- Business Glossary；
- Domains；
- Documents；
- logical models；
- metrics；
- semantic models；
- ownership；
- custom properties。

Context Platform 又加入 Context Documents 和 generated semantic anchors。

**Status: Implemented, but maturity varies by context type.**

Sources:

- https://docs.datahub.com/docs/features/feature-guides/observe
- https://docs.datahub.com/docs/features/feature-guides/context/
- https://datahub.com/blog/datahub-cloud-2-1/

---

# 7. Context Generation — Cloud Public Beta

截至 2026-09-28：

> DataHub Context Platform 是 Public Beta。

Context Generation 会分析：

- query history；
- schema；
- dbt；
- Looker / BI context；

生成 context documents / semantic anchors。

关键设计：

- 默认 auto-publish disabled；
- generated context 默认 unpublished；
- 可配置 eval；
- 可以按 domain 运行。

这对应我们的：

~~~text
Observation
-> Candidate Context
~~~

**Status: Implemented, Public Beta, DataHub Cloud.**

Sources:

- https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation
- https://datahub.com/blog/datahub-cloud-v2-2/

---

# 8. Proposal / Validation / Publication — Cloud Public Beta

这个 mapping 非常直接。

Data Expert / SME 可以：

- 查看 proposal；
- 运行 eval；
- 编辑；
- comment；
- publish / reject；
- unpublish。

而且：

> Only published context documents are visible to agents.

Human-edited business metadata 在 regeneration 中优先于 agent-generated metadata。

这已经非常接近我们的：

~~~text
Generation Plane != Truth Plane
~~~

和：

~~~text
Agents propose.
Evidence verifies.
Authorities publish.
~~~

**Status: Implemented, Public Beta.**

Sources:

- https://docs.datahub.com/docs/managed-datahub/context/review-context-proposals
- https://docs.datahub.com/docs/managed-datahub/context/activate-context

---

# 9. Provenance — Strong Substrate, Partial General Model

DataHub 有很强的 provenance-related primitives：

- lineage；
- version history；
- change timeline；
- source / ingestion context；
- Agent Registry dependencies；
- query lineage；
- glossary term versions。

Agent Registry 当前甚至记录：

- agent versions；
- ownership changes；
- eval changes；
- docs；
- lineage to datasets。

但是 Reference Architecture 要求的是：

> 每个重要 assertion 都能表达完整 evidence / derivation / transformation chain。

当前官方材料不足以证明 DataHub 有一个统一：

~~~text
Assertion
-> generated by activity
-> derived from evidence
-> validated by authority
~~~

的 generalized provenance object model。

**Status: Partial.**

---

# 10. Temporal Validity — Partial / Unclear

DataHub 有：

- aspect timestamps；
- change history；
- timeseries aspects；
- lineage edge更新时间；
- glossary versions；
- Agent version history。

但 Lineage docs 明确说明：

> time picker 过滤的是 latest lineage 中 edge 的 last-updated 时间，并不是 historical lineage snapshot。

这正说明：

> version/change-time support != full bitemporal context model.

我们 Reference Architecture 的：

~~~text
valid_from
valid_until
observed_at
invalidated_at
~~~

没有在当前公开资料中表现成统一 assertion-level model。

**Status: Partial / Unclear.**

Source:

- https://docs.datahub.com/docs/features/feature-guides/lineage

---

# 11. Freshness & Source Health — Partial

DataHub 有成熟的 data freshness / observability primitives：

- freshness assertions；
- anomaly detection；
- sync / ingestion state；
- incidents；
- operation events。

但 Context Freshness 更严格：

~~~text
source reality
-> source telemetry
-> ingestion
-> processing
-> context generation
-> publication
-> agent visibility
~~~

当前 DataHub Support 也明确承认 metadata processing 具有 eventual consistency。

我们没有看到一个统一的：

> Context Assertion inherits SourceHealth -> STALE / UNVERIFIED

机制。

所以：

**Status: Partial.**

Data quality freshness 已实现；epistemic context freshness 尚不能证明完整实现。

---

# 12. Authority — Partial

DataHub 已经有多个 authority primitive：

- owner；
- ownership type；
- domain；
- Data Expert / Editor；
- policies；
- roles；
- proposal reviewer；
- human-edited context precedence。

但我们的 Reference Architecture 区分：

~~~text
Source Authority
Domain Authority
Business Authority
Policy Authority
Publication Authority
Execution Authority
~~~

当前 DataHub 并没有公开一个统一 typed-authority model。

因此：

**Status: Partial.**

现实中 authority 由不同 features 分散表达。

---

# 13. Reconciliation / Invalidation — Partial / Unclear

DataHub 已经有一些 reconciliation-like capability：

- lineage metadata propagation；
- automations；
- context full refresh；
- human-edited context preservation；
- proposal regeneration；
- quality incidents；
- event-triggered Agents。

但我们 Reference Architecture 要求：

~~~text
source change
-> dependency graph
-> affected assertions
-> typed invalidation
-> recompute / review
~~~

目前公开文档不足以证明存在一个 generalized invalidation engine。

尤其没有明确看到：

~~~text
schema change
-> semantic context marked SUSPECT
-> business declaration routed for revalidation
~~~

作为通用平台 primitive。

**Status: Partial / Unclear.**

---

# 14. Retrieval & Projection — Implemented

DataHub 当前 retrieval surface 很成熟：

- Search；
- GraphQL；
- Python SDK；
- MCP；
- Ask DataHub。

Views 可以限定：

- entity type；
- platform；
- domain；
- tags；
- owners；

并自动应用到：

- search；
- browse；
- Ask DataHub；
- MCP service account search。

DataHub Agents 也可以绑定 View 来限制 Agent discoverable scope。

这非常接近我们定义的：

> shared truth substrate + consumer-specific projection.

**Status: Implemented.**

Sources:

- https://docs.datahub.com/docs/features/feature-guides/views/overview
- https://docs.datahub.com/docs/features/feature-guides/agents
- https://docs.datahub.com/docs/features/feature-guides/mcp

---

# 15. Search Access Control — Implemented, Cloud 更完整

DataHub Cloud Search Access Controls 支持 query-time filtering：

- search results；
- browse；
- direct entity access。

OSS 可以开启 entity page gating，但官方文档明确说：

> OSS 不提供相同的 search query-time filtering。

所以：

**Status: Implemented with Core/Cloud capability difference.**

Source:

- https://docs.datahub.com/docs/features/feature-guides/search-access-controls

---

# 16. MCP Activation — Implemented

这是 DataHub 当前最明确的 Agent interface。

DataHub MCP Server 可以：

- search；
- get entity metadata；
- list schema；
- lineage；
- document search；
- query history；
- SQL context；
- governance proposals。

部署：

- DataHub Cloud managed MCP；
- DataHub Core self-hosted MCP。

Cloud 还支持 OAuth/DCR 让 interactive users 以自己的 DataHub identity 连接。

所以：

**Status: Implemented.**

Source:

- https://docs.datahub.com/docs/features/feature-guides/mcp

---

# 17. MCP Mutation — Implemented

当前 MCP mutation tools 已经相当广：

- add/remove tags；
- glossary terms；
- owners；
- domains；
- descriptions；
- structured properties；
- lifecycle stages；
- save document；
- glossary authoring。

还存在 proposal tools：

- propose glossary term；
- propose lifecycle stage；
- accept/reject proposals。

这说明：

> Agent write-back 已经是实际 capability。

不过不是所有 mutation 都强制走 proposal。

所以从我们“Proposal Plane”要求来看：

**Status: Mutation Implemented; universal governed-write contract Partial.**

Source:

- https://docs.datahub.com/docs/features/feature-guides/mcp

---

# 18. Semantic Models — Implemented as Context Entities, Execution External

v2.1 已经把：

- Metric；
- Semantic Model；

作为 first-class catalog entities，并基于 Apache Ossie spec。

v2.2 又增加 Semantic Model Container Lineage：

~~~text
Metric
-> Semantic Model Dataset
-> Physical Dataset
~~~

并支持/扩展来自：

- dbt；
- Snowflake；
- Databricks；
- Cube；

的 semantic metadata。

这说明我们的判断成立：

> DataHub 必须理解 Semantic Layer artifacts。

但 DataHub 并没有因此成为主要 metric query compiler。

真正计算仍然由 semantic/data runtime 完成。

因此：

- **Semantic context integration: Implemented / growing**
- **Semantic execution: External**

Sources:

- https://datahub.com/blog/datahub-cloud-2-1/
- https://datahub.com/blog/datahub-cloud-v2-2/

---

# 19. Policy / IAM — Metadata Plane Implemented, Runtime Data Enforcement External

DataHub 自身有：

- authentication；
- platform policies；
- metadata policies；
- roles；
- View Entity；
- search access controls；
- service accounts。

这足以控制：

> 谁能看到 / 编辑 DataHub 中的 metadata。

但它并不等价于：

> warehouse runtime row/column authorization。

所以我们的五 Plane 边界仍然正确：

~~~text
Context informs policy.
Policy governs execution.
~~~

DataHub 与 external runtime enforcement 系统结合，才形成完整 Agent data-access chain。

**Status: Metadata IAM Implemented; runtime data authorization External.**

---

# 20. Agent Registry — Implemented

v2.1 引入 Agent Registry。

AI Agent、Skill、Tool/MCP Service 被建模成：

> first-class, governed, versioned metadata entities.

DataHub 可以查看：

- instructions；
- skills；
- tools；
- model；
- owner；
- upstream datasets；
- versions；
- eval scores；
- invocation volume / latency。

Agent 还进入 lineage graph：

~~~text
dataset
-> aiAgent
-> skill / tool
~~~

classification 可以沿 lineage 传播到 Agent 并触发 incident。

这非常接近我们 Reference Architecture 中：

> Agent becomes a governed organizational actor.

**Status: Implemented (Cloud capability).**

Source:

- https://docs.datahub.com/docs/features/feature-guides/agent-registry

---

# 21. Agents / Tasks / Decisions — Private Beta

截至当前文档：

> Agents 是 DataHub Cloud-only Private Beta。

Agent 包含：

- instructions；
- tools；
- AI Plugins；
- View scope。

Task 可以：

- manually run；
- schedule；
- event-trigger。

Decision：

> Agent 暂停执行，向 Human 请求判断，然后继续。

这正对应我们前面研究的：

> human judgment inside the reasoning path.

**Status: Implemented, Private Beta, Cloud-only.**

Source:

- https://docs.datahub.com/docs/features/feature-guides/agents

---

# 22. Decision Provenance / Audit — Partial

DataHub 已有：

- Agent Registry change history；
- Agent versions；
- eval history；
- Ask DataHub tool audit surfaces；
- task run history；
- lineage；
- ownership。

但我们 Reference Architecture 要求：

~~~text
DecisionTrace
├── exact context assertion versions
├── retrieval trace
├── semantic model version
├── policy decision
├── agent/model version
├── tool outputs
└── human decisions
~~~

没有找到官方证据证明现在可以把一次 Agent decision 按当时 context snapshot 完整重放。

**Status: Partial.**

---

# 23. Conflict-aware Retrieval — Unclear

DataHub 有：

- glossary versions；
- related glossary terms；
- lifecycle stages；
- proposal workflow；
- domains；
- scopes / Views。

但我们没有找到当前官方文档描述一个通用：

~~~text
multiple active assertions
-> detect conflict
-> resolve by authority/scope
-> expose CONFLICTED state to Agent
~~~

的 retrieval primitive。

因此不能把普通 search ranking / glossary versioning 当作 conflict-aware epistemic retrieval。

**Status: Unclear.**

---

# 24. Context SLO Framework — Partial / Unclear

DataHub 已有：

- ingestion status；
- assertion freshness；
- scheduled evals；
- task run status；
- sync status；
- incidents。

这些都是 reliability primitives。

但我们 Reference Architecture 的端到端 SLO：

~~~text
Observation SLO
Propagation SLO
Invalidation SLO
Publication SLO
Retrieval SLO
Decision Audit SLO
~~~

目前没有看到作为统一 Context Platform SLO model 的公开产品能力。

**Status: Partial / Unclear.**

---

# 25. Reference Architecture Coverage View

为了更直观看成熟度，可以分成四层。

## Layer A — Strongly Implemented

~~~text
Ingestion
Identity / URN
Entity / Aspect Model
Relationships
Lineage
Search
Ownership
Domains
Glossary
Quality / Incidents
API / SDK
MCP
Views / scoped retrieval
~~~

这是 DataHub 的传统强项。

## Layer B — Newly Implemented / Beta

~~~text
Context Documents
Context Intelligence
Context Proposals
Evals
Publication
Context Activation
Agent Registry
Agents / Tasks / Decisions
Metrics / Semantic Models
~~~

这是 2026 年的 Context Platform 扩张。

## Layer C — Partially Represented

~~~text
Assertion model
Provenance
Temporal validity
Authority
Freshness propagation
Reconciliation
Agent decision audit
~~~

DataHub 有多个相关 primitive，但不是我们 Reference Architecture 中的统一模型。

## Layer D — External / Not Evidenced

~~~text
Semantic query runtime
Runtime data policy enforcement
General conflict-aware retrieval
Generic typed invalidation engine
Full historical Agent decision replay
~~~

---

# 26. 一个重要结论：DataHub 的 Context Platform 是 Metadata-first，而不是 Knowledge-first

DataHub 的实现路线明显是：

~~~mermaid
flowchart LR
    META[Metadata Platform]
    KNOW[Business / Unstructured Context]
    LIFE[Context Lifecycle]
    AGENT[Agent Activation]

    META --> KNOW --> LIFE --> AGENT
~~~

而不是从：

~~~text
documents
-> ontology
-> enterprise knowledge graph
~~~

开始。

这解释了它为什么在以下方面天然较强：

- lineage；
- freshness；
- operational metadata；
- ownership；
- source integrations；
- data quality；
- Agent/data dependency。

同时也解释我们 Reference Architecture 中一些更 abstract epistemic primitive 为什么目前不是显式中心：

- generalized assertion semantics；
- arbitrary truth conflict；
- bitemporal validity；
- typed authority calculus。

---

# 27. Context Platform 当前真正的“产品边界”

截至 2026-09-28，可以把 DataHub 当前产品边界画成：

~~~mermaid
flowchart TB
    subgraph CORE[Metadata / Context Substrate]
        ING[Ingestion]
        MODEL[Entity / Aspect / Graph]
        LIN[Lineage]
        GOV[Ownership / Glossary / Domains / Policies]
        QUAL[Quality / Incidents]
        SEARCH[Search / API]
    end

    subgraph CLOUDCTX[Cloud Context Platform]
        GEN[Context Intelligence]
        DOC[Context Documents]
        EVAL[Evals]
        REVIEW[Review / Publish]
        ACT[MCP Activation]
    end

    subgraph AI[AI Layer]
        REG[Agent Registry]
        AG[Agents / Tasks / Decisions]
    end

    subgraph EXT[External Execution]
        SEM[Semantic Runtime]
        POLICY[Runtime Data Policy]
        WH[Warehouse]
    end

    CORE --> CLOUDCTX --> AI
    AI --> EXT
    CORE --> EXT
~~~

这个图比“DataHub 是一个 AI Control Plane”更接近当前事实。

---

# 28. 我们应该如何评价 DataHub 的成熟度？

不是打总分。

而应该分责任。

### Metadata Context Substrate

已经成熟。

### Context Lifecycle

真实存在，但仍处于 Public Beta。

### Agent Governance

Agent Registry 已落地；custom Agents 仍 Private Beta。

### Epistemic Semantics

有许多 building blocks，但我们定义的通用 assertion / authority / temporal / conflict model 仍属于更严格的目标架构。

### Runtime Execution

DataHub 明确依赖外部 semantic、warehouse、policy、tool systems。

这是合理架构边界，不应该被视为缺陷。

---

# 29. 下一步 Product Mapping

接下来值得继续拆成几个专题 mapping：

## 02 — Metadata Model as Context Substrate

深入 Entity / Aspect / URN / Relationship / Timeseries：

> 为什么 DataHub 现有 metadata model 能承载 Context Platform？哪些地方开始吃力？

## 03 — Context Lifecycle Product Mapping

完整追踪：

~~~text
query history
-> context generation
-> proposal
-> eval
-> SME
-> publish
-> MCP
-> Agent
~~~

## 04 — Agent Governance Product Mapping

追踪：

~~~text
Agent Registry
-> Agent scope
-> Tools
-> Tasks
-> Decisions
-> mutation
-> audit
~~~

## 05 — OSS vs Cloud Context Architecture

真正拆清：

> 哪些 Context primitive 在 OSS，哪些属于 Cloud product layer？

---

# Sources

Current official material used for this snapshot:

- https://docs.datahub.com/docs/features/feature-guides/mcp
- https://docs.datahub.com/docs/features/feature-guides/agent-registry
- https://docs.datahub.com/docs/features/feature-guides/agents
- https://docs.datahub.com/docs/features/feature-guides/lineage
- https://docs.datahub.com/docs/features/feature-guides/observe
- https://docs.datahub.com/docs/features/feature-guides/views/overview
- https://docs.datahub.com/docs/features/feature-guides/search-access-controls
- https://docs.datahub.com/docs/features/feature-guides/logical-models/overview
- https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation
- https://docs.datahub.com/docs/managed-datahub/context/review-context-proposals
- https://docs.datahub.com/docs/managed-datahub/context/activate-context
- https://datahub.com/blog/datahub-cloud-2-1/
- https://datahub.com/blog/datahub-cloud-v2-2/
