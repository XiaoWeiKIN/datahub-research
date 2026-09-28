# 09 — DataHub Design Philosophy — Synthesis

## 为什么在 AI / Agent 时代值得系统学习 DataHub？

**Status:** 第一版  
**Focus:** architectural philosophy / long-term design choices / AI-era reinterpretation  
**Updated:** 2026-09-28

---

# 0. 最终判断

经过前八篇研究，我们可以把整个项目最初的问题重新表述：

> **DataHub 值得学习，不是因为它今天有一个叫 Context Platform 的产品，而是因为它长期选择把 metadata 当成一个实时、可编程、关系化、可治理的基础设施层。AI Agent 的出现，让这些原本服务 Discovery / Governance / Lineage 的设计选择突然成为企业 AI 的 Context Infrastructure。**

所以 DataHub 的演进不是：

~~~text
Data Catalog
-> 加一个 Chatbot
-> 改名 Context Platform
~~~

更像：

~~~mermaid
flowchart LR
    CAT[Catalog]
    GRAPH[Metadata Graph]
    ACTIVE[Active / Event-driven Metadata]
    GOV[Governed Metadata Platform]
    CONTEXT[Context Graph]
    AGENT[Agent-ready Context Platform]

    CAT --> GRAPH --> ACTIVE --> GOV --> CONTEXT --> AGENT
~~~

这里真正连续的是底层哲学：

> **Metadata is operational infrastructure, not documentation.**

AI 时代改变的是：

> metadata 的主要 consumer 从 Human 扩展成 Human + Machine。

---

# 1. Philosophy 1 — Metadata Is Infrastructure, Not Documentation

传统 catalog 的隐含模型：

~~~text
real system
-> people manually document
-> catalog shows documentation
~~~

DataHub 更接近：

~~~text
real system
-> continuously emits / ingests metadata
-> metadata platform maintains operational model
-> humans and systems consume it
~~~

这是最根本的差异。

如果 metadata 只是 documentation：

- stale 很正常；
- missing owner 很正常；
- lineage 不完整也只是 UX 问题；
- API 不够强也没关系。

如果 metadata 是 infrastructure：

- freshness 是 SLO；
- identity 是系统契约；
- provenance 是审计基础；
- schema/lineage changes 是事件；
- machine consumers 是一等公民；
- consistency / rollback / availability 都成为工程问题。

AI 时代真正发生的是：

> Agent 把 metadata quality 从“用户体验问题”升级成“执行正确性问题”。

---

# 2. Philosophy 2 — Relationships Matter More Than Inventory

传统 Catalog 首先解决：

> 我们拥有哪些数据资产？

Graph-first 系统进一步解决：

> 它们怎么连接？

~~~mermaid
graph LR
    SOURCE[Source]
    PIPE[Pipeline]
    TABLE[Dataset]
    METRIC[Metric]
    DASH[Dashboard]
    OWNER[Owner]
    POLICY[Policy]
    AGENT[Agent]

    SOURCE --> PIPE --> TABLE --> METRIC --> DASH
    OWNER --> METRIC
    POLICY --> TABLE
    TABLE --> AGENT
~~~

关系让以下能力成为自然结果：

- impact analysis；
- root cause；
- provenance；
- governance propagation；
- blast-radius analysis；
- context invalidation；
- Agent dependency analysis。

DataHub 2026 自己对 metadata knowledge graph 的解释也明确强调：

> load-bearing word is graph。

真正价值不是 UI 画出节点和线。

而是：

> relationships 是 underlying model 的一部分。

AI 时代，这一点进一步升级：

> Agent 通常需要的不是“一个事实”，而是一条 evidence / dependency / authority path。

---

# 3. Philosophy 3 — Active Beats Static

DataHub 很早就围绕 active metadata / event-driven metadata 强调：

> Metadata 要反映正在运行的系统，而不是昨天扫描后的快照。

AI 时代，这个哲学变得更加重要。

静态 context 的失败模式：

~~~mermaid
flowchart LR
    REAL[Reality Changes]
    STATIC[Static Documentation]
    AG[Agent]
    ACT[Wrong Action]

    REAL -.not propagated.-> STATIC
    STATIC --> AG --> ACT
~~~

所以：

> **Context quality is temporal.**

一个定义是否正确，不仅取决于内容，还取决于：

- source 是否活着；
- observation 是否及时；
- dependency 是否变化；
- validation 是否仍有效；
- publication 是否更新。

这就是为什么我们最终把 Context Layer 看成：

> reactive dependency system

而不只是 knowledge store。

---

# 4. Philosophy 4 — Metadata Changes Are Events, Not Just CRUD

这一点很容易被忽略。

普通 metadata database：

~~~text
UPDATE owner = 'finance'
~~~

事件模型：

~~~text
owner changed
who changed it
when
from what
to what
what depended on it
what should happen next
~~~

后者允许：

- audit；
- history；
- downstream propagation；
- reconciliation；
- point-in-time reconstruction；
- Agent decision replay。

DataHub 当前关于 metadata lineage 的材料进一步把 metadata 本身视为 versioned / lineage-tracked object。

这对 AI 有直接意义：

> 如果 Agent 的答案依赖 Context v17，就必须能够在未来重建 Context v17，而不是只看到今天的最新 state。

---

# 5. Philosophy 5 — Governance Belongs Inside the Information Model

传统系统经常把 governance 设计成：

~~~text
data
+
separate governance portal
~~~

DataHub 的方向更接近：

~~~text
asset
+ owner
+ domain
+ classification
+ policy
+ quality
+ lineage
+ certification
~~~

这些都是 context graph 的一部分。

为什么重要？

因为 Agent 不只需要回答：

> 有什么数据？

还要回答：

- 哪个 trusted？
- 谁负责？
- 是否 certified？
- 当前质量正常吗？
- 属于哪个 domain？
- 哪个定义 authoritative？

因此：

> **Governance is not post-processing. Governance is context.**

这也是 DataHub 当前 Context Platform 强调 governance prerequisite 的原因。

---

# 6. Philosophy 6 — Shift Left: Context 应尽量在 Source 附近产生

DataHub 早期 metadata philosophy 里一个重要思想是 Shift Left：

> 尽量在数据/代码产生的地方声明和发射 metadata。

例如：

- ownership 跟代码；
- contract 跟 schema；
- classification 跟 source；
- glossary / semantic declarations 进入 version control；
- metadata-as-code。

这个哲学解决一个现实问题：

> 如果 context creation 依赖“以后有人来 Catalog 填表”，它一定 stale。

AI 时代可以进一步扩展：

~~~text
Source of Change
-> Source of Metadata
-> Source of Context Evidence
~~~

也就是：

> **越靠近事实产生的地方捕获 context，越少依赖事后人工重建。**

---

# 7. Philosophy 7 — One Substrate, Many Use Cases

传统 data management 工具容易形成：

~~~text
Discovery graph
Governance graph
Observability graph
AI context store
~~~

每个系统复制一份资产世界。

DataHub 一贯更强调 unified metadata graph。

这带来：

~~~mermaid
flowchart TB
    GRAPH[Shared Metadata / Context Graph]

    GRAPH --> DISC[Discovery]
    GRAPH --> GOV[Governance]
    GRAPH --> OBS[Observability]
    GRAPH --> IMP[Impact Analysis]
    GRAPH --> AI[Agents]
~~~

AI 时代进一步变成：

> shared governed substrate for humans and machines.

这个哲学的真正价值不是“少买几个工具”。

而是：

> **同一条 correction / ownership / lineage / policy 更新可以同时影响多个 use case。**

当然代价也是：

> shared substrate increases shared blast radius.

所以越共享，越需要 publication / rollback / authority。

---

# 8. Philosophy 8 — Human and Machine Should Share Truth, Not Interface

传统 catalog 是 UI-first。

DataHub 现在明确把 Agent 视为 first-class consumer，并通过 MCP / API / SDK 暴露 graph。

但最值得学习的不是 MCP。

而是：

> Human 和 Agent 应尽量依赖同一个 underlying governed truth。

~~~mermaid
flowchart TB
    T[Governed Context Substrate]
    HVIEW[Human Projection]
    AVIEW[Agent Projection]

    T --> HVIEW --> H[Human]
    T --> AVIEW --> A[Agent]
~~~

Human 需要：

- visualization；
- prose；
- ownership；
- lineage navigation。

Agent 需要：

- stable ids；
- structured assertions；
- provenance；
- freshness；
- semantic relations。

因此：

> **same truth, different projection.**

这避免两套事实系统长期漂移。

---

# 9. Philosophy 9 — Context Is a Lifecycle, Not a Corpus

传统 RAG 的 mental model：

~~~text
documents
-> chunk
-> embed
-> retrieve
~~~

DataHub Context Platform 更接近：

~~~mermaid
flowchart LR
    ING[Ingest Evidence]
    GEN[Generate / Derive]
    EVAL[Evaluate]
    REV[Review]
    PUB[Publish]
    USE[Use]
    OBS[Observe Outcomes]
    INV[Invalidate / Refresh]

    ING --> GEN --> EVAL --> REV --> PUB --> USE --> OBS --> INV --> GEN
~~~

这说明：

> Context management 的单位不是 document。

而是：

> **maintained knowledge lifecycle.**

AI 时代最大的 Context 问题不是“第一次生成难”。

而是：

> 第 100 天它还对不对？

---

# 10. Philosophy 10 — Generated Knowledge Must Not Collapse into Truth

Agent 可以生成：

- description；
- glossary mapping；
- semantic context；
- tag；
- ownership suggestion；
- query pattern。

但机器生成 ≠ 企业事实。

DataHub 当前 Context Hub 的 proposal / review / publish 模式体现了一个很成熟的思想：

> **Generation Plane != Truth Plane**

~~~mermaid
flowchart LR
    AG[Agent]
    CAND[Candidate]
    EVAL[Evidence / Eval]
    AUTH[Authority]
    PUB[Published Context]

    AG --> CAND --> EVAL --> AUTH --> PUB
~~~

我们最终把它压缩为：

> **Agents propose. Evidence verifies. Authorities publish.**

这可能是 Agent 时代 context governance 最重要的设计原则之一。

---

# 11. Philosophy 11 — Authority Must Be Federated

一个统一 Context Platform 不应该意味着：

> 中央团队拥有所有语义。

真正 authority 本来就分布：

- Warehouse 对当前 schema 有 source authority；
- Finance 对 Revenue 有 business authority；
- Security 对 classification 有 policy authority；
- HR 对组织关系有 source authority；
- Domain Expert 对业务定义有 publication authority。

所以：

~~~text
central substrate
+
federated authority
~~~

比：

~~~text
central source of all truth
~~~

更合理。

DataHub 自己关于 context ownership 的公开观点也逐渐走向 distributed custodian model。

---

# 12. Philosophy 12 — Truth Is Scoped, Versioned, and Contextual

“Single Source of Truth”不能字面理解。

成熟系统应该允许：

~~~text
Active Customer
├── Finance definition
├── Growth definition
└── Customer Success definition
~~~

它们可以同时正确。

区别来自：

- scope；
- domain；
- purpose；
- valid time；
- authority。

所以我们的最终工作模型不是：

~~~text
one concept -> one value
~~~

而是：

~~~text
assertion
+ scope
+ authority
+ temporal validity
+ provenance
~~~

Context Platform 的工作不是消灭 ambiguity。

而是：

> **make ambiguity governable.**

---

# 13. Philosophy 13 — Unknown Must Remain Unknown

这是一个非常基础但经常违反的原则。

如果系统没看到 lineage：

不能直接推出：

> no lineage.

如果 source 没有返回 owner：

不能直接推出：

> no owner.

如果 Agent 没找到相关 policy：

不能直接推出：

> no policy.

因此：

~~~text
unknown
unverified
partial
stale
conflicted
~~~

必须是一等 epistemic state。

AI 最大的问题之一是会把信息缺失填成一个流畅答案。

Context Layer 必须反过来：

> **preserve uncertainty structurally.**

---

# 14. Philosophy 14 — Deterministic Structure Should Constrain Probabilistic Reasoning

这是整个研究里超出 DataHub 本身的一条更普适原则。

LLM 不应该重新猜：

- metric formula；
- join path；
- access rule；
- known lineage；
- certified owner；
- active policy。

这些应该尽量由 deterministic / governed systems 提供。

~~~mermaid
flowchart TB
    DET[Deterministic / Governed Structure]
    PROB[Probabilistic Agent Reasoning]

    DET -->|constrain choice space| PROB
~~~

所以：

- Semantic Layer 固化 computational semantics；
- Context Layer 固化 trusted situational context；
- Policy Plane 固化 authorization；
- Agent 负责 unresolved reasoning / planning。

好的 AI infrastructure 不是让 Agent “更聪明地猜”。

而是：

> **让它需要猜的东西越来越少。**

---

# 15. Philosophy 15 — Protocol Is Important, Substrate Is More Important

MCP 是重要变化。

它解决：

- standard discovery；
- tool invocation；
- integration portability。

但：

> MCP 不会自动让 context 变可信。

如果 MCP 背后是：

- stale docs；
- wrong owner；
- conflicting definitions；
- incomplete lineage；

Agent 只是更高效地获取错误信息。

所以架构顺序应该是：

~~~text
trusted substrate
-> governed projection
-> protocol
-> agent
~~~

而不是：

~~~text
install MCP
-> context problem solved
~~~

DataHub 值得学的点不是“支持 MCP”。

而是它试图让 MCP 暴露的是长期维护的 context graph。

---

# 16. Philosophy 16 — Interoperability Beats Context Lock-in

如果每个 Agent vendor 都有自己的：

- memory store；
- vector DB；
- private ontology；
- proprietary context model；

企业最终会重新回到信息孤岛。

所以 Context Layer 必须尽量支持：

- stable identifiers；
- APIs；
- MCP；
- semantic standards；
- import/export；
- external policy systems；
- external semantic runtimes。

DataHub 当前把 metrics / semantic models 变成一等实体，并走向 semantic model interchange，也是这一方向的体现。

真正理想状态：

> Context Platform 是 shared substrate，而不是新的 silo。

---

# 17. Philosophy 17 — Architecture Boundaries Matter More as Platform Scope Expands

Context Platform 很容易变成 architecture blob。

因为它天然接触：

- identity；
- policy；
- semantics；
- data；
- Agent；
- knowledge；
- observability。

但“接触”不代表“接管”。

我们最终坚持五 Plane：

~~~text
Context  = know / trust
Semantic = compute
Policy   = allow
Data     = execute
Agent    = reason / act
~~~

DataHub 与 runtime policy vendor 的合作也进一步说明：

> trusted context 与 runtime enforcement 是 complementary responsibilities。

---

# 18. Philosophy 18 — Reconciliation Is More Important Than Initial Capture

企业 context 永远不会一次建完。

更现实：

~~~text
declared context
!= system reality
!= observed behavior
!= agent behavior
~~~

所以一个成熟 Context Platform 的核心循环应该是：

~~~mermaid
flowchart TB
    DECL[Declared]
    SYS[System]
    OBS[Observed Human Behavior]
    AG[Agent Behavior]

    DECL --> REC[Reconciliation]
    SYS --> REC
    OBS --> REC
    AG --> REC

    REC --> REVIEW[Review / Update]
~~~

这使 Context Platform 从：

> catalog of knowledge

走向：

> **reconciliation system for organizational reasoning.**

---

# 19. Philosophy 19 — Reliability Must Be Designed at Context Level

传统 Catalog KPI：

- documentation coverage；
- monthly active users；
- search clicks。

Agent-era Context Platform 还需要：

- freshness SLO；
- source health；
- conflict rate；
- invalidation latency；
- authority coverage；
- publication rollback；
- decision reproducibility；
- context incident MTTR。

因此我们提出：

> **Context Reliability Engineering**

因为 Context 一旦参与自动化决策，就必须拥有类似 production data / software infrastructure 的 reliability discipline。

---

# 20. Philosophy 20 — AI Makes Old Metadata Architecture More Valuable, Not Less

这是整篇最重要的历史判断。

AI 很容易让市场看起来像：

> 过去的数据治理全部失效，需要一套全新的 AI context stack。

但 DataHub 的演化提供了另一种解释：

~~~mermaid
flowchart LR
    OLD[Metadata Architecture]
    AI[AI Era]
    NEW[New Value]

    OLD -->|graph| NEW
    OLD -->|lineage| NEW
    OLD -->|ownership| NEW
    OLD -->|quality| NEW
    OLD -->|event model| NEW
    OLD -->|governance| NEW
    AI --> NEW
~~~

AI 没有让这些能力过时。

相反：

> Agent 比 Human 更依赖结构化、最新、可追溯、治理过的 metadata。

所以 Context Platform 很大一部分创新是：

> **AI revaluation of metadata infrastructure.**

---

# 21. 什么是 DataHub 真正“新的”部分？

必须区分旧基础和新扩张。

## 长期基础

- metadata graph；
- relationships；
- lineage；
- ownership；
- event-driven metadata；
- APIs；
- extensible metadata model；
- governance；
- machine-readable metadata。

## AI 时代扩张

- unstructured organizational knowledge；
- Context Documents；
- context intelligence；
- generated context proposals；
- eval-driven validation；
- context publication；
- MCP activation；
- Agent Registry；
- Agents / Tasks / Decisions；
- human-machine shared context；
- read/write Agent loops。

所以：

> Context Platform 不是从零设计。

而是：

> Metadata Platform + organizational knowledge + lifecycle + machine activation + Agent governance.

---

# 22. 什么不是 DataHub 独有创新？

研究中也必须避免产品崇拜。

以下概念并非 DataHub 发明：

### Knowledge Graph

来自更长的 semantic web / graph tradition。

### Provenance

W3C PROV 等已有成熟模型。

### Event-driven systems

是通用分布式系统架构。

### Semantic Layer

有 dbt / Looker / Cube 等独立传统。

### Policy Decision / Enforcement

有 IAM / OPA / security 系统。

### MCP

是开放协议，不属于 DataHub。

### Human review

也不是新概念。

DataHub 真正值得研究的不是单个 primitive。

而是：

> **它把这些 primitive 围绕 enterprise data context 组合成一个统一的 operating model。**

---

# 23. DataHub 最强的哲学不是 Graph，而是 Continuity

如果必须选一个词，我会选：

> **Continuity**

因为 DataHub 的架构思想有一条非常连续的主线：

~~~text
source reality
-> metadata event
-> graph state
-> governance / semantics
-> human use
-> machine use
-> feedback
-> graph state
~~~

数据资产、metadata、business meaning、Agent 都不是孤立 snapshot。

而是一个持续变化的系统。

Graph 提供关系。

Event 提供时间。

Governance 提供 trust。

Context management 提供 lifecycle。

Agent 提供新的 consumer / producer。

---

# 24. 从 Data Catalog 到 Epistemic Infrastructure

我们整个研究最终把 DataHub 的演进理解成：

~~~mermaid
flowchart LR
    C1[Inventory]
    C2[Discovery]
    C3[Metadata Graph]
    C4[Active Metadata]
    C5[Context Graph]
    C6[Shared Truth Plane]
    C7[Epistemic Infrastructure]

    C1 --> C2 --> C3 --> C4 --> C5 --> C6 --> C7
~~~

最后一个“Epistemic Infrastructure”是我们的研究术语，不是 DataHub 官方名称。

它想表达：

> 企业需要一个系统来维护“机器和人可以基于哪些知识做决定”。

这比传统 Catalog 的“哪里有什么数据”更深。

---

# 25. DataHub 的设计哲学可以压缩成 10 条

如果未来只保留一页，可以记住：

## 1. Metadata is infrastructure

不是静态文档。

## 2. Relationships are first-class

价值在 graph traversal，不只是 inventory。

## 3. Context must be active

stale context 比 missing context 更危险。

## 4. Governance is context

ownership / quality / provenance / policy 都应参与 reasoning。

## 5. Humans and machines share a substrate

但拥有不同 projection 和权限。

## 6. Generated context is proposal, not truth

机器生产知识不等于拥有 authority。

## 7. Authority is federated

不同 domain/source/policy 有不同解释权。

## 8. Unknown must remain explicit

系统不能把没有证据变成否定事实。

## 9. Reconciliation is continuous

声明、系统现实、行为、Agent 行为会持续漂移。

## 10. Deterministic infrastructure should constrain probabilistic agents

不要让 LLM 重猜系统本来就能确定的事实。

---

# 26. 为什么这套哲学特别适合 AI 时代？

因为 LLM 有几个固有倾向：

### 它会补全缺失信息

Context Layer 需要 preserve unknown。

### 它会把流畅性误当确定性

Context Layer 需要 provenance / authority / epistemic state。

### 它每次都可能重新推导逻辑

Semantic / policy / context layers 应固化 deterministic structure。

### 它缺少企业历史和组织权威

Context Graph 需要连接 institutional knowledge。

### 它可以高速执行错误

Freshness / governance / validation 必须进入 runtime context。

因此 Context Layer 不是为了让 LLM 获得更多 token。

而是：

> **给概率模型一个足够结构化、受约束、可追溯的企业现实。**

---

# 27. 为什么不直接做一个 Enterprise Knowledge Graph？

这是前面研究反复出现的问题。

答案不是：

> Knowledge Graph 不够先进。

而是：

> Context Platform 需要 operational requirements。

包括：

- source connectors；
- change detection；
- freshness；
- quality；
- usage；
- lineage；
- publication lifecycle；
- Agent activation；
- human review；
- SLO；
- rollback。

所以 Context Platform 更接近：

> operationalized enterprise knowledge graph for data + AI workflows.

---

# 28. 为什么不直接做一个 RAG Platform？

因为 RAG 主要解决：

> retrieve relevant information.

Context Layer 还要解决：

- authority；
- freshness；
- conflict；
- scope；
- provenance；
- lifecycle；
- invalidation；
- write-back；
- audit。

一句话：

> **RAG solves reach; Context Management solves trust.**

---

# 29. 为什么不只做 Semantic Layer？

因为 Semantic Layer 解决：

> 业务指标怎么算。

Context Layer 还要回答：

> 当前场景为什么应该用这个定义？它现在是否可信？谁负责？数据健康吗？有没有 policy / incident 影响它？

所以：

> **Semantic Layer makes meaning executable. Context Layer makes meaning situationally trustworthy.**

两者互补。

---

# 30. 为什么 Context Platform 不应该成为 Everything Platform？

因为 AI stack 仍然需要清晰职责：

~~~mermaid
flowchart LR
    CONTEXT[Context<br/>know / trust]
    SEM[Semantic<br/>compute]
    POLICY[Policy<br/>allow]
    DATA[Data<br/>execute]
    AG[Agent<br/>reason / act]

    CONTEXT --> AG
    AG --> SEM --> POLICY --> DATA
~~~

架构成熟度的一个标志不是：

> 一个产品做所有事。

而是：

> boundaries 清楚，context 可以跨系统流动。

---

# 31. DataHub 的长期战略价值可能在哪里？

如果企业未来拥有：

- 10 个 Agent；
- 100 个 Agent；
- 1,000 个 Agent；

最昂贵的问题可能不是模型 token。

而是：

> 如何让所有 Agent 不必分别学习一次公司是谁、数据是什么、规则是什么。

所以 Context Platform 的 compounding effect 是：

~~~text
first agent:
build context infrastructure

next agent:
reuse context infrastructure

next 1000 agents:
inherit same governed substrate
~~~

DataHub 当前 agent onboarding / context platform 叙事明确押注的就是这个方向。

这也是为什么 Context Layer 更像 shared infrastructure，而不是 application feature。

---

# 32. 我们对 DataHub 的最终定位

经过当前研究，我们暂时把 DataHub 定位成：

> **一个从 enterprise metadata graph 演进而来的 Context Infrastructure 平台，其长期价值在于把 technical metadata、business meaning、operational state、governance 和 Agent-facing context 连接在一个持续维护的 shared graph 中。**

它不是：

- 单纯 Catalog；
- 单纯 Knowledge Graph；
- 单纯 Semantic Layer；
- 单纯 RAG；
- 单纯 MCP Server；
- 完整 AI Control Plane。

更接近：

> **Enterprise Data / AI Epistemic Infrastructure**

这是我们的研究定义，不是官方 category。

---

# 33. 当前 Reference Model

~~~mermaid
flowchart TB
    subgraph REALITY[Enterprise Reality]
        DATA[Data Systems]
        DOCS[Knowledge]
        PEOPLE[People / Org]
        OPS[Operational Signals]
    end

    subgraph CONTEXT[Epistemic Context Infrastructure]
        OBS[Observe]
        ID[Identity]
        ASSERT[Assertions]
        GRAPH[Graph]
        TRUST[Trust / Authority / Provenance]
        REC[Reconcile / Invalidate]
        PUB[Publish]
        PROJ[Project / Retrieve]
    end

    subgraph CONSUMERS[Consumers]
        HUMAN[Humans]
        AGENTS[Agents]
        APPS[Applications]
    end

    REALITY --> OBS --> ID --> ASSERT --> GRAPH
    GRAPH --> TRUST --> REC --> PUB --> PROJ
    PROJ --> CONSUMERS
    CONSUMERS -.feedback.-> ASSERT
~~~

这是当前整个仓库的最终核心图。

---

# 34. 后续学习方向

主理论线到这里已经形成闭环。

接下来不应该继续无限增加概念文章。

建议进入三个更具体的方向：

## A. DataHub Product Mapping

把 Reference Architecture 逐项映射到 DataHub 当前实际产品能力：

- 已实现；
- 部分实现；
- roadmap；
- 未明确。

## B. Comparative Architecture

对比：

- DataHub；
- dbt / semantic ecosystem；
- Atlan；
- Collibra；
- OpenMetadata；
- enterprise knowledge graph；
- Agent memory / RAG stacks。

不做排名，只比较 responsibility / architecture。

## C. Concrete Case Studies

例如：

> “Analytics Agent 回答 Revenue 问题”

完整走一遍：

~~~text
user intent
-> context retrieval
-> authority resolution
-> semantic selection
-> policy
-> query
-> evidence
-> answer
-> write-back
-> audit
~~~

这会验证我们的架构是否真的能解释生产系统。

---

# 35. Final Thesis

如果整个仓库最后只能保留一段话，我会保留：

> **AI Agent 的可靠性上限，不只由模型决定，而由它所处的企业认知基础设施决定。DataHub 最值得研究的地方，是它长期把 metadata 设计成实时、关系化、可治理、可编程的基础设施；这使它能够自然扩展为一种面向 Human 和 Agent 的 Context Platform。Context Platform 的真正任务不是给模型更多信息，而是持续维护一个可被组织信任、解释、修正和复用的企业现实模型。**

---

# Sources

## DataHub — historical / architectural continuity

- DataHub, **Metadata Day 2022: 4 Principles of Modern Data Governance**  
  https://datahub.com/blog/metadata-day-round-up-4-principles-that-are-driving-modern-data-governance

- DataHub, **The 3 Must-Haves of Metadata Management — Shift Left Guide**, 2022  
  https://datahub.com/blog/the-3-must-haves-of-metadata-management-part-2/

## DataHub — current architecture / Context Platform

- DataHub, **What Is Metadata Management?**  
  https://datahub.com/blog/what-is-metadata-management/

- DataHub, **What Is a Metadata Knowledge Graph?**  
  https://datahub.com/blog/metadata-knowledge-graph/

- DataHub, **What Is a Context Graph?**  
  https://datahub.com/blog/context-graph/

- DataHub, **What Is Context Management?**  
  https://datahub.com/blog/context-management/

- DataHub, **The Context Layer for AI: What Enterprises Get Wrong**  
  https://datahub.com/blog/context-layer-for-ai/

- DataHub, **Announcing the DataHub Context Platform**  
  https://datahub.com/blog/announcing-datahub-context-platform/

- DataHub, **Context-Aware AI Agents: Why Most Aren't**  
  https://datahub.com/blog/context-aware-ai-agents/

- DataHub, **Agents in Production: How DataHub MCP Closes the Context Gap**  
  https://datahub.com/blog/agents-in-production-datahub-mcp/

- DataHub, **Inside the Context Platform — June 2026 Town Hall Highlights**  
  https://datahub.com/blog/inside-the-context-platform-june-2026-town-hall-highlights/

- DataHub, **Introducing DataHub Cloud v2.1**  
  https://datahub.com/blog/datahub-cloud-2-1/

- DataHub, **Introducing DataHub Cloud v2.2**  
  https://datahub.com/blog/datahub-cloud-v2-2/

- DataHub, **What Is Metadata Lineage?**  
  https://datahub.com/blog/metadata-lineage/
