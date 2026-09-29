# Case 04 — Cross-case Architecture Synthesis

## 从 Analytics、Incident Repair、Governance 三类 Agent 中抽出不可约运行时架构

**Snapshot:** 2026-09-29  
**Scope:** Synthesis of Case 01–03

---

# 0. 为什么要做 Cross-case Synthesis？

前三个 case 分别覆盖：

- **Analytics Agent**：如何选对定义、算对数字、解释为什么可信；
- **Incident Agent**：如何定位影响、修复、人工决策、验证与回写；
- **Governance / PII Agent**：如何在不泄露敏感 context 的前提下发现风险并整改。

任务完全不同，但三个 case 最终都收敛到同一套运行时骨架：

~~~mermaid
flowchart LR
    SCOPE[Scope]
    CTX[Trusted Context]
    AUTH[Authority]
    DET[Deterministic System]
    VERIFY[Verification]
    AUDIT[Audit]

    SCOPE --> CTX --> AUTH --> DET --> VERIFY --> AUDIT
~~~

这说明：

> **Production Agent Architecture 的真正复用单位不是某个 Prompt，而是一套 Context / Authority / Execution / Verification Contract。**

---

# 1. 三个 Case 在解决什么不同的不确定性？

| Case | 主要不确定性 |
|---|---|
| Analytics | “该用哪个定义 / metric / dataset？” |
| Incident | “真正受影响范围是什么？改什么才安全？” |
| Governance | “谁能看到什么？谁有权采取什么行动？” |

但继续向下分解，会发现每个 case 都需要回答六类问题：

1. **What is in scope?**
2. **What is true / trusted now?**
3. **Who has authority?**
4. **Which deterministic system should execute?**
5. **How do we verify actual state?**
6. **Can we reconstruct why this happened later?**

---

# 2. Unified Runtime Model

~~~mermaid
flowchart TB
    USER[Human / Event / Scheduler]

    subgraph AGENT[Agent Plane]
        INTENT[Interpret Intent]
        PLAN[Plan]
        DECIDE[Escalate / Continue]
    end

    subgraph CONTEXT[Context Plane]
        SCOPE[Scoped Projection]
        ASSERT[Trusted Assertions]
        PROV[Provenance / Freshness]
        AUTH[Authority]
    end

    subgraph SEMANTIC[Semantic Plane]
        SEM[Semantic Model / Compiler]
    end

    subgraph POLICY[Identity / Policy Plane]
        ID[Principal / Delegation]
        PDP[Policy Decision]
        PEP[Enforcement]
    end

    subgraph EXEC[Execution Plane]
        DATA[Warehouse / SaaS / Git / Runtime]
    end

    subgraph VERIFY[Verification Plane]
        READBACK[Independent Read-back]
        EVAL[Behavioral Eval]
        STATE[Observed Outcome]
    end

    subgraph AUDIT[Audit Plane]
        TRACE[Decision Trace]
        VERSION[Context / Agent Versions]
    end

    USER --> INTENT --> PLAN
    SCOPE --> PLAN
    ASSERT --> PLAN
    PROV --> PLAN
    AUTH --> DECIDE

    PLAN --> SEM
    PLAN --> ID --> PDP
    SEM --> PDP
    PDP --> PEP --> DATA

    DATA --> READBACK --> STATE
    STATE --> EVAL
    EVAL --> DECIDE

    DECIDE --> TRACE
    VERSION --> TRACE
~~~

这个模型比“Agent + MCP + Warehouse”多了三个关键层：

- **Authority**
- **Verification**
- **Audit**

这三层决定系统能不能进入生产。

---

# 3. Pattern 1 — Scope Before Retrieval

三个 case 都证明：

> Agent 不应该先搜整个企业，再靠 LLM 自己判断哪些结果相关。

更合理的路径：

~~~text
principal / task / domain
-> scoped projection
-> retrieval
-> reasoning
~~~

## Analytics

Finance reporting 问题应该优先进入 Finance-approved context。

## Incident

Schema incident 应限制在受影响 lineage / repository / environment。

## Governance

PII investigation 更需要严格 scope，否则 metadata discovery 本身就可能泄露敏感资产存在性。

所以：

> **Scope 是 retrieval contract，不只是 UI filter。**

DataHub Views / scoped MCP / Agent View 已经提供这个方向的现实 primitive。

---

# 4. Pattern 2 — Context Before Action

三个 case 都不允许：

~~~text
user request
-> LLM guesses
-> tool call
~~~

而要求：

~~~text
request
-> context resolution
-> trust check
-> action
~~~

Agent 至少需要知道：

- entity identity；
- relevant relationships；
- current operational state；
- business meaning；
- provenance；
- authority；
- conflicts。

所以：

> **Agent action should be downstream of context qualification.**

---

# 5. Pattern 3 — Authority Is Separate from Relevance

一个 context 很 relevant，不代表它 authoritative。

### Analytics

Growth 的 Active Customer 定义可能很相关，但 Board Reporting 应由 Finance definition 决定。

### Incident

Agent 可以识别修复建议，但 code owner 才有 merge authority。

### Governance

DataHub 可以发现 classification 风险，但 Security / Privacy policy 才决定具体治理要求。

所以所有 Agent 应区分：

~~~text
relevance
!=
authority
~~~

这也是为什么普通 RAG ranking 无法代替 Context Governance。

---

# 6. Pattern 4 — Deterministic Systems Should Execute Deterministic Logic

前三个 case 都反复验证：

> 不应该让 LLM 重新推导已经有确定性系统负责的逻辑。

## Analytics

Metric / join / aggregation：

> Semantic Runtime。

## Incident

Code change：

> Git / CI / tests。

## Governance

Access：

> IAM / runtime policy enforcement。

Agent 负责：

- intent；
- selection；
- plan；
- coordination；
- exception handling。

Deterministic system 负责：

- compute；
- enforce；
- validate structure；
- execute transaction。

原则：

> **Agents orchestrate; deterministic systems enforce.**

---

# 7. Pattern 5 — Human Decision Is a Runtime Primitive

三个 case 都有不能通过更多模型推理安全解决的 ambiguity。

例如：

- 两个业务定义都有依据；
- 两个潜在 owner 都合理；
- sensitive classification 需要 policy interpretation；
- 修复可能造成 breaking behavior。

这时系统不应该：

> 让 Agent 继续猜。

而应该：

~~~mermaid
sequenceDiagram
    participant A as Agent
    participant H as Human Authority
    participant C as Context

    A->>C: inspect evidence
    C-->>A: unresolved ambiguity
    A->>H: Decision request + evidence + impact
    H-->>A: authoritative choice
    A->>C: continue with scoped decision
~~~

DataHub Tasks / Decisions 正好体现这一 pattern。

---

# 8. Pattern 6 — Write Must Be Asymmetric

Read 和 Write 不应该对称。

~~~text
Read trusted context
Write proposal / bounded mutation
Publish authoritative context
~~~

而不是：

~~~text
GET entity
PATCH entity
~~~

## Analytics

通常 read-heavy，write 可能只是保存 evidence / explanation。

## Incident

可以写修复 PR、incident resolution、updated lineage，但需要验证。

## Governance

高风险 classification / policy change 更适合 proposal + authority。

因此 write path 需要：

- risk class；
- evidence；
- authority；
- verification；
- rollback。

---

# 9. Pattern 7 — Independent Verification Is Mandatory

三个 case 都不能把：

~~~text
tool returned success
~~~

当成：

~~~text
desired state achieved
~~~

## Analytics

需要检查 query result / semantic model / freshness。

## Incident

需要 CI、read-back、lineage refresh、dashboard validation。

## Governance

需要重新读取 classification / policy enforcement / Agent scope。

所以：

~~~mermaid
flowchart LR
    INTENT[Intended Change]
    WRITE[Action]
    READ[Independent Read-back]
    CMP[Compare]
    OK[Verified]

    INTENT --> WRITE --> READ --> CMP --> OK
~~~

原则：

> **Success response is evidence of acceptance, not evidence of convergence.**

---

# 10. Pattern 8 — Audit Must Capture the Context Used at Decision Time

如果只保存：

- chat transcript；
- 当前 context；
- 当前 Agent config；

仍然无法复现过去 decision。

三个 case 都需要：

~~~text
DecisionTrace
├── principal
├── agent + version
├── task
├── retrieved context ids + versions
├── provenance / freshness
├── authority decision
├── semantic model / policy version
├── tool calls
├── human decisions
├── mutations
└── observed outcome
~~~

因此：

> **Audit is temporal context capture, not just logging.**

---

# 11. 三个 Case 对五 Plane 的映射

| Plane | Analytics | Incident | Governance |
|---|---|---|---|
| Context | metric / definitions / freshness | lineage / schema / incidents | classification / lineage / owners |
| Semantic | metric execution | usually minor | usually minor |
| Policy | user access | repo / system permissions | central |
| Data / Execution | warehouse | Git / CI / orchestration | warehouse / IAM / policy runtime |
| Agent | interpret / orchestrate | diagnose / coordinate repair | discover / propose remediation |

由此可以看出：

> 同一个 Context Platform 不需要执行所有工作，但必须能为不同 execution systems 提供可信 context。

---

# 12. Context Platform 的不可约核心

从三个 case 反推，我们可以删掉很多 feature，最后仍必须保留：

## 1. Stable Identity

知道“哪个对象”。

## 2. Relationships

知道“与什么有关”。

## 3. Trust Metadata

知道“为什么相信”。

## 4. Scope / Authority

知道“在哪个场景、谁说了算”。

## 5. Freshness / State

知道“现在还能不能用”。

## 6. Proposal / Publication

知道“机器生成的信息什么时候升级为 shared truth”。

## 7. Activation

让 Agent 能以结构化方式读取。

## 8. Audit

让组织能重建决策。

这八项比“有没有聊天 UI”更能定义 Context Platform。

---

# 13. DataHub 在三个 Case 中扮演的共同角色

DataHub 并不是：

> Agent runtime 本身。

它更像：

~~~mermaid
flowchart TB
    REALITY[Enterprise Systems]
    DH[DataHub Context / Metadata Plane]
    AG[Agent]
    EXTERNAL[Semantic / Policy / Git / Warehouse]

    REALITY --> DH
    DH --> AG
    AG --> EXTERNAL
    EXTERNAL --> REALITY
    AG -.proposals / metadata.-> DH
~~~

它提供：

- graph；
- lineage；
- ownership；
- business context；
- quality；
- scoped retrieval；
- Agent Registry；
- MCP；
- proposal / governance primitives。

执行依赖外部系统。

这正支持前面的定位：

> **DataHub = Context / Epistemic Infrastructure, not universal execution runtime.**

---

# 14. Analytics Case 给出的设计原则

Analytics Agent 的核心不是 text-to-SQL。

而是：

~~~text
business intent
-> applicable context
-> semantic selection
-> policy
-> deterministic query
-> evidence-backed answer
~~~

核心原则：

> **Context chooses; semantics computes.**

---

# 15. Incident Case 给出的设计原则

Incident Agent 的核心不是 autonomous code change。

而是：

~~~text
detect
-> graph impact
-> propose repair
-> human decision
-> deterministic CI
-> verify
-> write back
~~~

核心原则：

> **Automation without independent verification is incomplete automation.**

---

# 16. Governance Case 给出的设计原则

Governance Agent 的核心不是发现 PII。

而是：

~~~text
classification context
-> scoped discovery
-> delegated identity
-> runtime policy
-> remediation
-> audit
~~~

核心原则：

> **Context visibility != runtime authorization.**

---

# 17. 三个 Case 合起来得到的 Production Agent Contract

~~~text
ProductionAgentContract
├── Identity
│   ├── agent_id
│   ├── version
│   └── invoking_principal
├── Scope
│   ├── domain
│   ├── view
│   └── purpose
├── Context
│   ├── assertions
│   ├── provenance
│   ├── freshness
│   └── conflicts
├── Authority
│   ├── owner
│   ├── policy
│   └── human_decision
├── Execution
│   ├── tool
│   ├── deterministic_runtime
│   └── expected_effect
├── Verification
│   ├── read_back
│   ├── eval
│   └── observed_state
└── Audit
    ├── trace
    ├── versions
    └── outcome
~~~

这个 Contract 可以作为以后比较任何 Agent Platform 的 checklist。

---

# 18. 一个更深的结论：Agent Reliability 是跨 Plane 属性

Agent 错误不能只归因于：

> model hallucination.

实际错误来源可能是：

~~~text
wrong context
wrong semantic model
wrong authority
wrong policy
stale data
wrong identity
write not converged
missing verification
~~~

所以：

~~~mermaid
flowchart LR
    C[Context]
    S[Semantic]
    P[Policy]
    D[Data]
    A[Agent]
    R[Reliability]

    C --> R
    S --> R
    P --> R
    D --> R
    A --> R
~~~

> **Agent Reliability is a cross-plane property.**

这可能是前三个 case 最重要的共同结论。

---

# 19. 为什么 Context Layer 会成为 Shared Infrastructure？

三个 case 的 Agent 完全不同：

- Finance Analytics Agent；
- Data Incident Agent；
- Governance Agent。

但它们都重复需要：

- entity identity；
- lineage；
- ownership；
- classification；
- quality；
- semantic meaning；
- provenance。

如果每个 Agent 单独维护：

~~~text
vector DB
prompt rules
entity mapping
ownership mapping
freshness logic
~~~

组织会快速形成 context islands。

所以 Context Platform 的 compounding value 来自：

> **same governed substrate reused by different Agent classes.**

---

# 20. 但 Shared Infrastructure 同时放大 Failure Blast Radius

同一条错误 context 可能影响：

- Analytics answers；
- Incident remediation；
- Governance decisions。

因此 Context Platform 的 reliability 要求比普通 Catalog 更高。

它必须拥有：

- publish boundary；
- rollback；
- provenance；
- source health；
- conflict states；
- audit；
- blast-radius-aware review。

这再次验证 Phase 1 Failure Modes。

---

# 21. 三种 Loop

三个 case 其实代表三种 Agent loop。

## Query Loop

~~~text
question
-> context
-> semantic execution
-> answer
~~~

## Repair Loop

~~~text
incident
-> context
-> action
-> verify
-> update context
~~~

## Governance Loop

~~~text
policy / classification
-> context
-> inspect impact
-> enforce / remediate
-> audit
~~~

这三种 loop 覆盖了大量 enterprise Agent 场景。

---

# 22. 一个统一的 Closed-loop Agent Architecture

~~~mermaid
flowchart LR
    TRIGGER[Question / Event / Policy]
    CTX[Context]
    PLAN[Agent Plan]
    EXEC[Deterministic Execution]
    OBS[Observed Result]
    VERIFY[Verify]
    UPDATE[Context / Audit Update]

    TRIGGER --> CTX --> PLAN --> EXEC --> OBS --> VERIFY --> UPDATE
    UPDATE -.next cycle.-> CTX
~~~

如果只有前半段：

~~~text
Trigger -> Agent -> Tool
~~~

那只是 automation。

加入：

~~~text
Context + Verification + Update
~~~

之后才成为真正的 closed-loop system。

---

# 23. Phase 3 对 DataHub 的验证结果

前三个 case 基本验证了我们之前对 DataHub 的定位：

## DataHub 很适合做

- shared context substrate；
- lineage / impact graph；
- business / technical metadata；
- Agent discoverability；
- scoped retrieval；
- human-machine context coordination；
- metadata write-back / proposals。

## DataHub 不应该单独负责

- metric execution；
- code execution；
- warehouse authorization；
- identity source-of-record；
- CI；
- runtime sandbox；
- secrets；
- all policy enforcement。

这不是产品缺陷。

而是健康的 architecture boundary。

---

# 24. 对 Agent Platform 的通用评估 Checklist

以后评估任何 Agent Platform，可以问：

### Context

- 是否有 shared context substrate？
- 是否支持 provenance / freshness / authority？
- 是否能表达 conflict / unknown？

### Scope

- 是否能按 user / task / domain 投影 context？
- Agent 能不能越过 scope 调底层 tool？

### Execution

- deterministic work 是否交给 deterministic runtime？
- tool contract 是否 typed / auditable？

### Write

- Agent 是直接写 truth，还是 proposal？
- 有 rollback / read-back 吗？

### Human

- ambiguity 是否可以 runtime escalation？
- human decision 是否进入 trace？

### Audit

- 能否知道 Agent 当时看到的 context version？
- 能否重建 decision path？

如果这些问题回答不了：

> Agent 可能能 demo，但还不是可靠 production architecture。

---

# 25. Phase 3 最终架构

~~~mermaid
flowchart TB
    subgraph CONTEXT[Shared Context Plane]
        ID[Identity]
        GRAPH[Relationships]
        TRUST[Trust / Freshness]
        AUTH[Authority]
    end

    subgraph AGENT[Agent Plane]
        INTENT[Intent]
        PLAN[Plan]
        DECISION[Human Decision]
    end

    subgraph EXECUTION[Deterministic Execution]
        SEM[Semantic Runtime]
        POLICY[Policy Runtime]
        TOOLS[Git / Warehouse / SaaS]
    end

    subgraph RELIABILITY[Reliability]
        VERIFY[Independent Verification]
        AUDIT[Decision Audit]
    end

    CONTEXT --> AGENT
    AGENT --> EXECUTION
    EXECUTION --> RELIABILITY
    RELIABILITY -.feedback.-> CONTEXT
~~~

这就是三个案例共同推导出的最小生产闭环。

---

# 26. 下一阶段建议

到这里：

- Phase 1 解决了“Context Layer 应该是什么”；
- Phase 2 解决了“DataHub 当前实现到了哪里”；
- Phase 3 解决了“真实 Agent 任务如何使用这些能力”。

下一阶段不应继续堆 case。

建议进入：

## Phase 4 — Comparative Architecture

比较不同体系的责任边界：

- DataHub
- OpenMetadata
- Atlan
- Collibra
- dbt / Semantic Layer ecosystem
- Enterprise Knowledge Graph
- RAG / Agent Memory stack

比较维度不是 feature 数量，而是：

~~~text
Identity
Graph
Context lifecycle
Authority
Freshness
Semantic execution
Policy integration
Agent activation
Write-back
Audit
~~~

目标是回答：

> **DataHub 的 Context Platform 路线，与其他可能的企业 AI Context 架构相比，真正独特在哪里？**

---

# Related Cases

- [Case 01 — Analytics Agent](01-analytics-agent-net-revenue.md)
- [Case 02 — Schema Change Incident Agent](02-schema-change-incident-agent.md)
- [Case 03 — Governance / PII Agent](03-governance-pii-agent.md)
