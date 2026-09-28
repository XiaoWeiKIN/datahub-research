# 06 — Human + Agent Shared Truth Plane

## 当人和 Agent 都依赖同一套 Context，Context Platform 会不会成为企业 AI 的 Control Plane？

**Status:** 第一版  
**Focus:** shared truth / distributed authority / human-agent symmetry / control plane boundary  
**Updated:** 2026-09-28

---

# 0. 当前结论

前五篇逐步建立了：

- Context Layer；
- Context Graph；
- Semantic Layer boundary；
- freshness / provenance；
- Agent write-back governance。

现在可以问一个更大的问题：

> **如果 Human 和 Agent 都依赖同一个 Context Layer，它是否正在成为企业 AI 的 Control Plane？**

我们的答案是：

> **部分是，但不能直接等同。**

更准确的定义是：

> **Context Platform 正在成为 enterprise AI 的 epistemic control plane：它管理“系统认为哪些企业事实、定义、关系和信任信号可以被认知和使用”；但它不应该取代 IAM、runtime policy enforcement、semantic execution 或 data plane。**

换句话说，它管理的是：

- what is known
- what it means
- why it is trusted
- who is authoritative
- what is applicable now

而不是独自管理：

- who may execute what
- where workload runs
- how SQL executes
- how network / data access is enforced

---

# 1. 为什么会出现 Shared Truth Plane？

传统企业里，人和机器通常走不同的信息路径。

~~~mermaid
flowchart TB
    REAL[Enterprise Reality]

    REAL --> CAT[Catalog / Wiki for Humans]
    REAL --> RAG[Vector Store for Agent A]
    REAL --> DB[Bespoke Context DB for Agent B]
    REAL --> PROMPT[Hard-coded Prompt for Agent C]

    CAT --> H[Humans]
    RAG --> A1[Agent A]
    DB --> A2[Agent B]
    PROMPT --> A3[Agent C]
~~~

这会形成多个 truth islands：

- Catalog 的定义已经更新；
- Agent A 的 embedding index 仍是旧版本；
- Agent B 使用另一套 metric mapping；
- Agent C 把规则写死在 prompt；
- 没有人知道哪个 Agent 看到了哪个版本。

当 Agent 数量很少时，这只是 integration debt。

当企业部署几十、几百个 Agent 时，它变成：

> **organizational epistemic fragmentation**

每个 Agent 都拥有自己的“小型企业现实”。

---

# 2. DataHub 当前明确主张 Human 和 Agent 共用同一 Context Graph

DataHub 当前 Context Platform 的公开定位是：

> technical metadata、operational context 和 business context 被统一进入同一可查询 Context Graph，并从同一个 governed source of truth 提供给 humans 和 AI agents。

这个方向背后的架构逻辑是：

~~~mermaid
flowchart TB
    REAL[Enterprise Reality]
    CTX[Shared Governed Context Plane]

    REAL --> CTX

    CTX --> H[Human]
    CTX --> A1[Analytics Agent]
    CTX --> A2[Governance Agent]
    CTX --> A3[Engineering Agent]
~~~

关键不是“大家使用 DataHub UI”。

而是：

> **所有 consumer 尽量从相同的 underlying assertions、provenance、authority 和 freshness state 出发。**

---

# 3. Shared Truth 不等于 Same Experience

Human 和 Agent 不应该看到完全相同的 presentation。

同一个 metric，对 Human 可能需要：

- business-friendly description；
- owner；
- dashboard examples；
- certification；
- documentation；
- lineage visualization。

对 Agent 可能需要：

- stable identifier；
- executable semantic definition；
- valid joins；
- freshness state；
- provenance；
- policy hints；
- structured relationships；
- epistemic status。

更合理的是：

~~~mermaid
flowchart TB
    T[Shared Truth Plane]
    HP[Human Projection]
    AP[Agent Projection]
    PP[Policy Projection]

    T --> HP --> H[Human]
    T --> AP --> A[Agent]
    T --> PP --> P[Policy Engine]
~~~

核心原则：

> **Same underlying truth, different projections.**

不是 same UI / same payload / same privileges。

---

# 4. Shared Truth 也不等于 Everyone Sees Everything

DataHub 已经开始提供 scoped MCP server、Agent-specific scope 与治理角色。

Shared Truth Plane 应该理解为：

> 共享的逻辑事实基础。

而不是：

> 所有 consumer 拥有相同读取权限。

~~~mermaid
flowchart TB
    T[Shared Context Graph]

    T --> S1[Projection: Finance]
    T --> S2[Projection: Marketing]
    T --> S3[Projection: Agent X]

    S1 --> F[Finance Human]
    S2 --> M[Marketing Human]
    S3 --> A[Agent]
~~~

因此：

> **Shared Truth != Shared Access**

---

# 5. Truth Plane 保存的是 Assertions，不是“绝对真理”

“Single Source of Truth”容易产生错误期待。

企业知识本来就可能存在：

- domain-specific truth；
- conflicting definitions；
- temporal versions；
- policy exceptions；
- regional rules；
- scenario-specific meaning。

例如 Active Customer 在 Finance、Growth、Customer Success 可能有三个合法定义。

所以 Context Platform 不应该强迫：

one term -> one global value

而应该表达：

assertion + scope + authority + valid time + domain + provenance

更准确的目标不是 single truth，而是：

> **single governed truth model**

冲突本身也应该被建模。

---

# 6. Truth Plane 应该支持 Contextual Truth

~~~mermaid
graph TB
    TERM[Active Customer]
    D1[Definition A]
    D2[Definition B]
    FIN[Finance Domain]
    GROWTH[Growth Domain]

    TERM --> D1
    TERM --> D2
    D1 --> FIN
    D2 --> GROWTH
~~~

Agent 问“Active customers 有多少？”时，Context Layer 不应该简单返回 Definition A。

它应先判断：

- 当前 task 属于哪个 domain？
- 用户是谁？
- report 用途是什么？
- 时间范围是什么？
- 哪个 authority governs this use case？

因此：

> **Truth Plane 的职责不是消灭 ambiguity，而是 make ambiguity governable。**

---

# 7. Ownership 必须是 Federated，而不是 Centralized

DataHub 最近关于 Context Ownership 的材料提出：Context 没有单一 owner。

它至少来自三类 custodians：

### Data / Platform Teams

负责 schema、lineage、freshness、quality、operational metadata。

### Analysts / Domain Experts

负责 business semantics、metric meaning、definitions、SOP、domain knowledge。

### Governance / Compliance

负责 policy、access rules、classification、regulatory obligations。

~~~mermaid
flowchart TB
    CP[Shared Context Plane]

    DATA[Data / Platform Custodian] -->|technical context| CP
    SME[Domain / Analyst Custodian] -->|business context| CP
    GOV[Governance Custodian] -->|policy context| CP
~~~

重要原则：

> **Shared infrastructure does not imply centralized authorship.**

平台可以统一，authority 必须分布。

---

# 8. Context Platform 更像 Federation Layer

更合理的组织模型是：

central platform  
+ distributed custodians  
+ explicit authority boundaries

~~~mermaid
flowchart TB
    PLATFORM[Shared Platform]

    F1[Finance Authority]
    F2[Marketing Authority]
    F3[Security Authority]
    F4[Data Platform Authority]

    F1 --> PLATFORM
    F2 --> PLATFORM
    F3 --> PLATFORM
    F4 --> PLATFORM
~~~

这和 source control 很像：

- repository 是共享基础设施；
- CODEOWNERS 决定 authority；
- review workflow 协调贡献；
- history 记录责任链。

---

# 9. Context Platform 是 Control Plane 吗？

在 Kubernetes 中，control plane 管理 cluster overall state，接收 desired state，并通过 controllers 检测现实偏差、推动系统回到期望状态。

~~~mermaid
flowchart LR
    DES[Desired State]
    CP[Control Plane]
    DP[Data / Execution Plane]

    DES --> CP
    CP --> DP
    DP -->|observed state| CP
~~~

Context Platform 确实有 Control Plane 特征：

- 保存共享 state；
- 建模 desired governance state；
- 收集 observed state；
- 检测 drift；
- 路由 review；
- 给 Agent 提供 decision context；
- 记录 provenance；
- 管理 agent / asset relationships。

但它通常不直接负责：

- authentication；
- runtime authorization；
- network enforcement；
- data query enforcement；
- compute scheduling；
- model serving；
- secrets；
- sandboxing。

所以 Context Platform = Enterprise AI Control Plane 表达过强。

---

# 10. 更准确的术语：Epistemic Control Plane

Epistemic 关注：

> 系统知道什么，以及为什么相信。

Context Platform 负责的 state 类似：

- known assets；
- known relationships；
- known definitions；
- known owners；
- known quality state；
- known incidents；
- known policies；
- known provenance；
- known authorities；
- known agent dependencies。

然后 Agent 根据这些信息 interpret、plan、select、route、reason。

所以：

> **Context Layer controls the epistemic state available to AI, not the entire runtime state of AI.**

---

# 11. Context Plane 与 Policy Plane 必须分开

DataHub 与 SecuPi 的联合材料提供了一个很清楚的例子。

Context Layer 可以告诉 Agent：

- customer churn 需要哪些表；
- 哪些 join；
- metric 如何定义；
- dataset 是否 trusted。

但 Agent 是否有权看到 Social Security Number，是 authorization / enforcement 问题。

~~~mermaid
flowchart LR
    USER[User / Principal]
    CTX[Context Plane]
    AG[Agent]
    POLICY[Policy Decision / Enforcement Plane]
    DATA[Data Plane]

    USER --> AG
    CTX -->|what is relevant| AG
    AG -->|requested action| POLICY
    POLICY -->|allow / deny / filter| DATA
~~~

重要原则：

> **Knowing is not authorization.**

---

# 12. Context Plane 与 Semantic Plane 也必须分开

上一篇已经得到：

> Context chooses. Semantics computes.

放进 shared truth architecture：

~~~mermaid
flowchart TB
    CTX[Epistemic Context Plane]
    SEM[Semantic Execution Plane]
    POLICY[Policy / Authorization Plane]
    DATA[Data Plane]
    AG[Agent]

    CTX --> AG
    AG --> SEM
    SEM --> POLICY
    POLICY --> DATA
~~~

职责分别是：

- Context Plane：什么 relevant / trusted / applicable？
- Semantic Plane：怎么计算？
- Policy Plane：是否允许执行？
- Data Plane：真正存储和处理数据。

---

# 13. 一个更完整的 Enterprise AI Plane Model

~~~mermaid
flowchart TB
    subgraph EP[Epistemic / Context Plane]
        CG[Context Graph]
        AUTH[Authority]
        PROV[Provenance]
        STATE[Freshness / Quality]
    end

    subgraph SP[Semantic Plane]
        SM[Semantic Models]
        COMP[Query Compiler]
    end

    subgraph PP[Policy Plane]
        IAM[Identity]
        PDP[Policy Decision]
        PEP[Policy Enforcement]
    end

    subgraph AP[Agent Plane]
        AG[Agents]
        TASK[Tasks]
        MEM[Task / Agent Memory]
    end

    subgraph DP[Data / Execution Plane]
        WH[Warehouse]
        SAAS[SaaS / Operational Systems]
    end

    EP --> AG
    AG --> SP
    AG --> PP

    SP --> PP
    PP --> DP

    DP --> EP
    DP --> SP
    AP --> EP
~~~

---

# 14. 为什么 Humans 和 Agents 应该共享 Underlying Truth？

如果 Human 和 Agent 不共享：

human system of meaning != agent system of meaning

会出现：

### Debugging Gap

用户看到 catalog 定义 A，Agent 使用 context store 定义 B。

### Governance Gap

Human approval 更新了定义，但 Agent index 未同步。

### Trust Gap

人无法确认 Agent 实际使用了什么 context。

### Operational Gap

Agent 发现新问题，却写进自己的 memory，人类 catalog 看不到。

所以 shared truth plane 最大价值之一是：

> **organizational debuggability**

所有参与者都可以讨论：

> “我们究竟基于哪条 assertion 做了这个决定？”

---

# 15. Shared Truth 最大的价值可能是 Debuggability

传统软件 debugging 依赖：

code + input + state + logs

Agent decision debugging 还需要：

model + prompt + tools + context + context versions + authority + provenance + policy

如果 Human 和 Agent 不共享 Context Plane：

> 人类甚至无法复现 Agent 当时看到的企业现实。

因此 Context Platform 应该让 enterprise reasoning 足够可重建、可审计。

---

# 16. Same Truth Plane 让一个 Fix 可以服务多个 Consumer

~~~mermaid
flowchart LR
    FIX[Finance updates definition]
    T[Shared Truth Plane]

    FIX --> T
    T --> H[Human Search]
    T --> A1[Analytics Agent]
    T --> A2[Governance Agent]
    T --> API[Applications]
~~~

共享底座的经济价值：

> **one governed correction propagates to many consumers.**

---

# 17. 但 Shared Truth 也制造更大的 Blast Radius

~~~mermaid
flowchart LR
    BAD[Bad Published Context]
    T[Shared Truth Plane]

    BAD --> T
    T --> H[Humans]
    T --> A1[Agent A]
    T --> A2[Agent B]
    T --> A3[Agent C]
~~~

因此：

> **Centralizing context value also centralizes context risk.**

这再次证明 proposal boundary、evidence gate、authority、provenance、rollback 不是可选 feature。

---

# 18. Shared Truth 应该做 Blast-Radius-aware Governance

不是所有 context 都应该同样容易 publish。

### Local Context

只影响一个 Agent / workflow。

### Domain Context

影响一个业务 domain，需要 domain authority。

### Enterprise Context

如 corporate metric、PII classification、global policy、canonical customer definition，需要更严格 review。

~~~mermaid
flowchart LR
    L[Local]
    D[Domain]
    E[Enterprise]

    L -->|higher blast radius| D --> E
~~~

随着 blast radius 增大：

- required evidence；
- required authority；
- review rigor；
- rollback discipline；

都应该增加。

---

# 19. Agent 数量增加后，治理对象会变化

过去治理对象主要是：

- dataset；
- dashboard；
- metric；
- user。

以后还包括：

- Agent；
- Agent version；
- skill；
- tool；
- memory；
- context scope；
- task；
- decision；
- generated assertion。

DataHub Agent Registry 已经在向这个方向走。

Context Platform 开始描述：

> **non-human organizational topology**

---

# 20. Agent 可以被理解成新的 Organizational Actor

~~~mermaid
graph LR
    PERSON[Person]
    TEAM[Team]
    AGENT[Agent]
    SKILL[Skill]
    DATA[Dataset]
    METRIC[Metric]

    PERSON --> TEAM
    TEAM --> AGENT
    AGENT --> SKILL
    AGENT --> DATA
    AGENT --> METRIC
~~~

Agent 在 governance 里越来越像 delegated organizational actor。

它拥有：

- owner；
- scope；
- responsibilities；
- permissions；
- dependencies；
- outputs；
- performance history。

这会让 Context Graph 与 Org Graph 越来越接近。

---

# 21. Agent 不能简单复制 Human IAM Model

Human 通常使用自己的 identity。

Agent 经常通过 service account / NHI / shared credentials 执行，于是系统可能丢失：

> 谁发起了这次请求、为了什么目的、应该继承谁的权限。

因此 Agent request context 应该包含：

agent identity  
+ invoking human / principal  
+ purpose  
+ task scope  
+ delegated authority

而不是只有 service account。

这连接了 Context Plane 与 Identity Plane，但二者不能合并。

---

# 22. Authority Model 应该是 Multi-dimensional

一条 assertion 是否 authoritative，至少可能包括：

- Domain Authority；
- Source Authority；
- Policy Authority；
- Temporal Authority；
- Publication Authority；
- Execution Authority。

例如：

- Finance 有权定义 Net Revenue；
- HR System 有权声明 employee/team membership；
- Security 有权定义 sensitivity policy；
- Semantic Runtime 有权声明 metric execution specification；
- Warehouse 有权声明当前 schema。

所以 shared truth 不是一个中央 database 说了算，而是：

> **平台把不同 authority domains 的声明连接并协调。**

---

# 23. 这更像 DNS / PKI / Git，而不是 Master Database

Context Platform 更可能是：

shared graph  
+ federated authority  
+ versioned assertions  
+ explicit provenance  
+ promotion workflow

而不是：

central admin edits everything

这个类比也解释了为什么 federation 比 central ownership 更重要。

---

# 24. Control Plane 的真正特征：Reconciliation

Kubernetes Control Plane 的重要模式是：

> observed state 与 desired state 不一致时，触发 reconciliation。

Context Platform 如果真的成为 control-plane-like infrastructure，也需要类似能力：

~~~mermaid
flowchart LR
    DES[Desired Context State]
    OBS[Observed Context State]
    DIFF[Drift]
    REC[Reconcile]

    DES --> DIFF
    OBS --> DIFF
    DIFF --> REC
~~~

例如：

### Desired

Finance-approved Revenue definition 应该映射到 metric A。

### Observed

query behavior 大量使用 metric B。

Context Platform 应该：

- detect drift；
- flag conflict；
- ask SME；
- update context / policy；
- possibly update training/examples。

这就是 Declared vs Observed Context 的进一步演化。

---

# 25. Context Platform 可能成为 Enterprise Reasoning Reconciliation Plane

它持续比较：

- what organization says；
- what systems show；
- what people do；
- what agents do。

~~~mermaid
flowchart TB
    DECL[Declared Truth]
    SYS[System Reality]
    OBS[Observed Behavior]
    AG[Agent Behavior]

    DECL --> R[Reconciliation]
    SYS --> R
    OBS --> R
    AG --> R

    R --> REVIEW[Review / Update / Policy]
~~~

它可以发现：

- semantic drift；
- ownership drift；
- behavior drift；
- policy drift；
- agent drift。

这可能是 Context Platform 比传统 Catalog 更深的一层价值。

---

# 26. Governance 必须是 Cross-cutting

NIST AI RMF 把 Govern 视为横跨 AI lifecycle 的持续功能，而不是最终审批步骤。

这与我们的 Context Plane 模型一致：

- Data team 对 technical context 负责；
- Domain team 对 business meaning 负责；
- Security/Compliance 对 policy 负责；
- AI Platform 对 agent identity / execution governance 负责；
- Context Platform 负责协调、记录和传播。

所以：

> **Context governance is a coordination architecture.**

---

# 27. Shared Context Plane 至少需要什么？

## Identity

每个 asset / metric / document / agent 有稳定 identity。

## Relationships

依赖、lineage、ownership、semantic connections。

## Assertions

事实不是裸值，而是带 scope 的 assertion。

## Authority

知道谁对哪种 assertion 有解释权。

## Provenance

知道 assertion 如何产生。

## Temporal State

知道何时有效、是否 stale / invalid。

## Epistemic State

VALID / SUSPECT / CONFLICTED / UNVERIFIED 等。

## Publication State

draft / proposal / published / superseded。

## Projection

给不同 human / agent / policy consumer 不同视图。

## Audit

能重建某次 Agent decision 使用的 context。

---

# 28. Shared Truth Plane 不应该负责什么？

为了避免 Context Platform 变成 architecture blob，它不应该默认承担所有职责。

- 不替代 Warehouse；
- 不替代 Semantic Runtime；
- 不替代 IAM；
- 不替代 Runtime Policy Enforcement；
- 不替代 Agent Runtime；
- 不替代 Source Systems。

它更应该是：

> **logical coordination and trust layer**

---

# 29. DataHub 今天离“AI Control Plane”还有多远？

## 已经具备的 Control-plane-like 能力

- Context Graph；
- 多来源 context ingestion；
- ownership；
- provenance / lineage；
- quality / freshness；
- policies；
- context proposal / publication；
- Agent Registry；
- scoped MCP；
- Agent tasks / decisions；
- event-driven metadata；
- evals；
- audit surfaces。

## 仍依赖外部系统

- fine-grained runtime authorization；
- end-user identity propagation；
- policy enforcement at query time；
- semantic execution；
- data processing；
- model runtime；
- secrets；
- network isolation；
- full AI risk management。

因此最准确的判断仍然是：

> **DataHub 正在构建 Enterprise AI 的 Context / Epistemic Control Plane，而不是完整的 AI Control Plane。**

---

# 30. 最终参考模型

~~~mermaid
flowchart TB
    subgraph HUMAN[Human Organization]
        U[Users]
        SME[Domain Experts]
        GOV[Governance]
    end

    subgraph EP[Epistemic Context Plane]
        CG[Context Graph]
        ASSERT[Assertions]
        AUTH[Authority]
        PROV[Provenance]
        FRESH[Freshness]
        PUB[Publication / Reconciliation]
    end

    subgraph AP[Agent Plane]
        AG[Agents]
        TASK[Tasks]
        MEM[Private / Task Memory]
    end

    subgraph SP[Semantic Plane]
        SEM[Semantic Models]
        EXEC[Query Compiler]
    end

    subgraph PP[Identity / Policy Plane]
        ID[Identity / Delegation]
        PDP[Policy Decision]
        PEP[Enforcement]
    end

    subgraph DP[Data Plane]
        WH[Warehouses / Lakes]
        OP[Operational Systems]
    end

    SME --> EP
    GOV --> EP
    U --> AG

    EP --> U
    EP --> AG

    AG --> SP
    AG --> PP

    SP --> PP
    PP --> DP

    DP --> EP
    SP --> EP
    AP --> EP
~~~

---

# 31. 当前工作定义

## Shared Truth Plane

> **一个供人和机器共同依赖的逻辑企业事实层，其中 context 以带 scope、authority、provenance、temporal validity 和 governance state 的 assertions 表达；不同 consumer 可以获得不同 projection，但底层事实和责任链保持一致。**

## Epistemic Control Plane

> **管理企业 AI 可认知状态的基础设施：持续获取、连接、验证、发布和协调企业 context，使 Agent 能知道哪些事实 relevant、trusted、current 和 authoritative，但不取代 runtime authorization、semantic execution 或 data processing。**

---

# 32. 下一步

前六篇已经把主要概念分解完成。

下一篇停止继续“概念对比”，开始收束成：

## 07 — Context Layer Reference Architecture

将前六篇组合成一个可实现的 reference architecture：

1. Sources / observation plane
2. Identity & entity resolution
3. Context assertions
4. Context graph
5. Provenance / temporal model
6. Authority / ownership
7. Reconciliation & invalidation
8. Proposal / publication
9. Retrieval / projection
10. MCP / API activation
11. Semantic plane integration
12. Policy / IAM integration
13. Agent read/write contract
14. Audit / decision provenance
15. SLO / failure modes

这会是从“学习 DataHub”转向“我们自己能设计 Context Layer”的分界点。

---

# Sources

## DataHub

- DataHub, **Context Platform for AI Agents**  
  https://datahub.com/products/context-platform/

- DataHub, **Introducing DataHub Cloud 2.0**, 2026-06-30  
  https://datahub.com/blog/datahub-cloud-2-0/

- DataHub, **Introducing DataHub Cloud 2.1**, 2026-08-03  
  https://datahub.com/blog/datahub-cloud-2-1/

- DataHub, **Introducing DataHub Cloud 2.2**, 2026-09-09  
  https://datahub.com/blog/datahub-cloud-v2-2/

- DataHub, **Who Owns Context Management?**, 2026-05-11  
  https://datahub.com/blog/context-ownership/

- DataHub, **Enhancing Agent Governance with DataHub and SecuPi**, 2026-09-10  
  https://datahub.com/blog/agent-governance-with-datahub-and-secupi/

- DataHub Docs, **Activate Context**  
  https://docs.datahub.com/docs/managed-datahub/context/activate-context

- DataHub Docs, **Agents**  
  https://docs.datahub.com/docs/features/feature-guides/agents

## External architectural references

- Kubernetes, **Kubernetes Components / Control Plane**  
  https://kubernetes.io/docs/concepts/overview/components/

- Open Policy Agent, **Management APIs and Architecture**  
  https://www.openpolicyagent.org/docs/management-introduction

- Open Policy Agent, **Deployment / PDP-PEP model**  
  https://www.openpolicyagent.org/docs/deploy

- NIST, **AI Risk Management Framework / Govern**  
  https://airc.nist.gov/airmf-resources/airmf/5-sec-core/
