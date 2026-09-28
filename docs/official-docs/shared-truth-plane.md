# Human + Agent Shared Context — 官方材料中文学习笔记

**Status:** 第一轮  
**Focus:** same governed context / ownership / scoped activation / runtime governance  
**Updated:** 2026-09-28

> 本文是 DataHub 当前产品材料的中文释义与架构拆解，不是逐字翻译。

---

# 1. DataHub 当前的核心主张

DataHub Context Platform 明确主张：

> technical metadata、operational context 和 business context 统一到一个可查询 Context Graph，并从同一个 governed source of truth 提供给 humans 和 AI agents。

最重要的不是 Human 和 Agent 使用相同界面，而是两者尽量消费同一 underlying context truth。

---

# 2. DataHub Cloud 2.0：Context for Every Agent, Every Employee

产品路线包括：

- 从 Notion / Confluence / GitHub 等来源 ingest context；
- 从 observed usage 推导 context；
- routing 到人 review；
- MCP / API / SDK activation。

~~~mermaid
flowchart LR
    SOURCE[Enterprise Sources]
    CTX[Context Platform]
    H[Employees]
    A[Agents]

    SOURCE --> CTX
    CTX --> H
    CTX --> A
~~~

---

# 3. Same Context 不等于 Same Scope

2.1 开始增加：

- scoped MCP servers；
- Agent Registry；
- governed agent graph。

Agents docs 也允许 Agent 有 instructions、tools、plugins 和 scoped view of data ecosystem。

因此：

> same source of truth = same governed substrate + different scoped projections

---

# 4. Context Ownership 是 Distributed Model

DataHub 2026-05-11 的 Context Ownership 文章明确反对一个团队统一拥有所有 context。

它把 custodian 大致分成：

- Data Teams：technical / operational context；
- Analysts / SMEs：business semantics / domain knowledge；
- Governance / Compliance：policy / access / classification / regulation。

因此 Context Platform 的作用是：

> coordination infrastructure

而不是 central content team。

---

# 5. Governance Context 与 Runtime Enforcement 不是同一个问题

DataHub 与 SecuPi 2026-09 的文章做了一个重要区分。

Context Platform 可以让 Agent 知道：

- 哪张表 relevant；
- 什么 metric；
- lineage；
- ownership；
- quality。

但 Agent 是否能读取 SSN、account balance、healthcare attributes，是 runtime authorization 问题。

~~~mermaid
flowchart LR
    C[Context Platform]
    A[Agent]
    P[Runtime Policy / Enforcement]
    D[Data]

    C --> A
    A --> P --> D
~~~

所以 Context Plane 和 Authorization Plane 是 complementary layers。

---

# 6. Agent Registry 让 Human / Agent 开始进入同一治理图

DataHub Cloud 2.1 把 Agent、skills、tools、owner、datasets、versions、evals 放进同一个 lineage/context graph。

因此 Agent 自己也成为 metadata entity。

Context Graph 开始描述：

> enterprise actors — human + machine

而不再只描述 data assets。

---

# 7. DataHub Cloud 2.2 让 Agent 成为 Workflow Actor

2.2 增加：

- Agents；
- Tasks；
- Decisions。

这说明 Context Platform 正从“给 Agent context”向“在同一治理面管理 Agent 的上下文、任务和人类判断节点”扩张。

但 execution 本身仍然依赖 tools / plugins / external systems。

---

# 8. Shared Truth 的官方优势

DataHub 的产品叙事认为统一 context 可以减少：

- duplicate context stores；
- conflicting metric definitions；
- stale RAG indexes；
- inconsistent agent answers；
- repeated context engineering。

它希望把 many agent-specific context stores 变成：

> shared context management -> many agent-specific projections

---

# 9. “Single Source of Truth”不要字面理解

企业现实里：

- domain definitions 可以不同；
- temporal versions 可以共存；
- policy 会因 jurisdiction 变化；
- authority 是 distributed 的。

所以更合理的技术表达是：

> **single governed model for multiple scoped truths**

Context Graph 不应该消灭冲突，而应该表达 scope、authority、provenance、valid time，并在 retrieval 时选择适用 assertion。

---

# 10. Context Platform 与 Control Plane

借用 Kubernetes 的 control plane 概念：

- global state；
- desired state；
- observed state；
- reconciliation。

Context Platform 已经有类似元素：

- context state；
- ownership；
- policy；
- proposal / publication；
- drift / freshness；
- Agent Registry。

但它不独立负责 IAM、runtime authorization、model serving、compute、network 或 data query enforcement。

所以我们采用：

> **Epistemic Control Plane**

作为研究术语，而不是断言 DataHub 已经是完整 Enterprise AI Control Plane。

---

# 11. 当前学习结论

~~~mermaid
flowchart TB
    PEOPLE[Humans]
    AGENTS[Agents]
    CTX[Shared Governed Context Plane]
    POLICY[Runtime Policy Plane]
    DATA[Enterprise Data]

    CTX --> PEOPLE
    CTX --> AGENTS

    PEOPLE --> CTX
    AGENTS --> CTX

    AGENTS --> POLICY --> DATA
~~~

Context Platform 负责：

> meaning / trust / context / authority / provenance

Policy system 负责：

> runtime permission / enforcement

两者结合才更接近 production agent governance。

---

# Related Research

- [06 — Human + Agent Shared Truth Plane](../research/06-human-agent-shared-truth-plane.md)
- [05 — Agent Read / Write Context](../research/05-agent-read-write-context.md)
- [04 — Context Freshness & Provenance](../research/04-context-freshness-and-provenance.md)
