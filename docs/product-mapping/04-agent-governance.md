# 04 — Agent Governance Product Mapping

## DataHub 如何把 Agent 变成可治理、可限制、可审计的组织级 Actor？

**Snapshot:** 2026-09-28  
**Scope:** Agent Registry / Agents / Views / MCP / Tasks / Decisions / Write-back / Audit

---

# 0. 当前结论

DataHub 当前已经形成两套互补但职责不同的 Agent 能力：

~~~mermaid
flowchart LR
    EXT[External / Native Agent]
    REG[Agent Registry<br/>Catalog / Govern]
    RUN[Agents<br/>Build / Execute]
    VIEW[View / Scope]
    TOOL[Tools / Plugins]
    TASK[Tasks]
    DEC[Decisions]
    AUDIT[Audit / Lineage / Versions]

    EXT --> REG
    RUN --> REG
    RUN --> VIEW
    RUN --> TOOL --> TASK --> DEC --> AUDIT
~~~

## Agent Registry

解决：

> **这个 Agent 是谁、用什么、读什么、谁负责、版本是什么？**

它把 Agent、Skill、Tool、MCP Service 放进 metadata / lineage graph。

## Agents

解决：

> **这个 Agent 如何执行任务？**

它定义：

- instructions；
- tools；
- external AI Plugins；
- View scope；
- Tasks；
- Decisions。

这两者的关系是：

> **Registry governs the actor; Agents governs the execution workflow.**

当前最重要的成熟能力：

- Agent 作为 first-class metadata entity；
- skills / tools / MCP services 建模；
- data lineage to Agent；
- ownership / version / change history；
- classification propagation；
- View-based discovery scope；
- tool allow-list；
- OAuth / service account identity options；
- task run history；
- full tool-call trace；
- Decision-based human checkpoint。

当前最明显的边界与风险：

1. Custom Agents 仍是 **Private Beta**；
2. Tasks 当前以 **创建 Task 的用户身份**运行；
3. View scope 限制的是 DataHub search/discovery，不应被误解成最终 runtime authorization；
4. MCP mutation 中部分操作可直接提交，proposal workflow 还不是所有 write 的统一必经路径；
5. Agent Registry 的 audit 很强，但尚不能证明可以完整 replay 某次 Agent decision 使用的所有 context versions；
6. Scoped MCP Servers 当前官方 Docs 仍标记为 **Private Beta**。

---

# 1. Agent Registry 和 Agents 是两个不同产品层

DataHub 当前文档明确区分：

~~~text
Agent Registry
= catalog and govern agents as metadata

Agents
= build agents that execute tasks
~~~

这是一个非常正确的边界。

如果只做 Agent Runtime：

> Agent 执行完以后，很难纳入企业 metadata governance。

如果只做 Agent Registry：

> 只能看 inventory，不能治理真实 automation lifecycle。

因此两者结合后：

~~~mermaid
flowchart TB
    META[Governed Metadata Graph]
    REG[Agent Registry]
    EXEC[Agent Runtime]

    META --> REG
    EXEC --> REG
    REG --> META
~~~

Agent 自己变成 enterprise asset。

Sources:

- https://docs.datahub.com/docs/features/feature-guides/agent-registry
- https://docs.datahub.com/docs/features/feature-guides/agents

---

# 2. Agent Registry — Agent 是 First-class Metadata Entity

DataHub Agent Registry 当前引入：

- AI Agent；
- Skill；

并复用 Service Catalog 的：

- API；
- Service；

表示 Tool 和 MCP Server。

官方当前 graph 可以概括为：

~~~text
repository
-> service (MCP)
-> api (tool)

aiAgent
-> invokes -> api
-> adopts -> skill
-> reads -> dataset

skill
-> requires -> api
~~~

因此：

> Agent 不是一行 registry record，而是 graph 中的 actor。

这让普通 impact analysis 也能回答：

- 哪些 Agent 调用这个 API？
- 哪些 Agent 读取这个 dataset？
- 某个 Agent 依赖哪些 tools？
- 哪个 Skill 被哪些 Agent 复用？

**Status: Implemented.**

---

# 3. Agent Registry 的治理复用了已有 Metadata Governance

官方强调：

因为 Agent 在 graph 里，它自动获得：

- ownership；
- tags；
- glossary terms；
- documentation；
- lineage；
- incidents；
- change history。

这是一种很重要的架构选择：

> **不要为 Agent 再造一套 governance database。**

而是：

> reuse existing metadata governance substrate.

这和整个 DataHub “one substrate, many use cases” 哲学一致。

---

# 4. Skills：从 Prompt 片段升级成 Governed Capability

Agent Registry 把 Skill 建成 catalog entity。

官方定义：

> reusable capability bundle = prompts + tools + domain expertise.

Skill 还可以：

- 有 source-of-truth spec；
- 指定需要的 tools；
- 被多个 Agent adopt。

这比：

~~~text
prompt copied into every agent config
~~~

更成熟。

~~~mermaid
flowchart LR
    SKILL[Skill]
    A1[Agent A]
    A2[Agent B]
    TOOL[Required Tool]

    A1 --> SKILL
    A2 --> SKILL
    SKILL --> TOOL
~~~

它让 Agent capability 本身进入治理图。

---

# 5. Tools：不是字符串，而是 API Contract

Agent Registry 把 Agent Tool 建模成 API Entity。

因此 tool 可以拥有：

- typed inputs；
- typed outputs；
- OpenAPI / JSON Schema contract；
- versioned metadata。

MCP Server 则建模成 Service Entity，并保存 live tool list。

这非常重要：

> Agent tool dependency 不再只是 prompt 中的 tool name。

而是可以做：

- versioning；
- impact analysis；
- discovery；
- lineage；
- governance。

---

# 6. Agent Lineage：数据到 Agent 的治理传播

Agent 位于数据资产 downstream。

~~~mermaid
flowchart LR
    DATA[Highly Confidential Dataset]
    AG[Agent]
    INCIDENT[Incident]

    DATA -->|reads| AG
    DATA -->|classification propagates| AG
    AG --> INCIDENT
~~~

DataHub 当前官方案例：

如果 source table 标记为 Highly Confidential：

- classification 传播到 consuming Agent；
- Agent health 变红；
- incident 被创建。

这是 Agent Governance 中非常重要的能力：

> **governance can follow lineage into the AI layer.**

**Status: Implemented.**

---

# 7. Agent Versioning — Agent 本身有 Software-like Lifecycle

DataHub Agent Registry 当前支持：

- version set；
- latest version；
- 多版本并存；
- ownership changes；
- eval score changes；
- documentation；
- version milestones；
- full timeline。

这把 Agent 从：

> mutable config

提升成：

> versioned governed asset.

对 production AI 非常重要，因为：

~~~text
Agent v1
!=
Agent v2
~~~

两者可能：

- 用不同 model；
- 用不同 tool；
- 访问不同 data；
- 有不同 evaluation profile。

---

# 8. 但 Agent Versioning != Decision Replay

Agent Registry 能记录：

- version；
- dependency；
- change history。

Agents runtime 能记录：

- run history；
- tool calls；
- outputs。

但我们 Reference Architecture 的 Decision Replay 还要求：

~~~text
Agent Version
+ Model Version
+ Context Assertion Versions
+ Semantic Model Version
+ Policy Decision
+ Exact Tool Outputs
+ Human Decisions
~~~

当前公开文档不能证明这些全部被绑定到一个可重放的 decision snapshot。

所以：

> **Asset audit is strong; full epistemic decision replay remains Partial / Unclear.**

---

# 9. Agents — Runtime Config Model

DataHub Custom Agent 当前包含：

| Field | Governance Meaning |
|---|---|
| Name | stable human-visible identity |
| Description | declared purpose |
| Instructions | behavioral policy / prompt |
| Tools | DataHub tool allow-list |
| AI Plugins | external MCP integrations |
| View | discovery scope |
| Show in Ask DataHub | interaction surface |

这已经非常接近：

> configurable delegated actor.

~~~mermaid
flowchart TB
    A[Agent]
    I[Instructions]
    T[Tools]
    P[Plugins]
    V[View]

    I --> A
    T --> A
    P --> A
    V --> A
~~~

**Status: Private Beta, Cloud-only.**

---

# 10. Manage Agents 是配置权限，不是 Agent Runtime Identity

要管理：

- Agents；
- Tasks；
- Decisions；

用户必须拥有：

> Manage Agents platform privilege.

没有这个 privilege 的用户仍可以与被暴露到 Ask DataHub Chat 的 Agent 交互，但不能修改配置。

这体现：

~~~text
agent administration
!=
agent usage
~~~

这是正确的管理平面边界。

---

# 11. View Scope — 限制 Agent“能发现什么”

Agent 可以绑定一个 DataHub View。

View 可以按：

- entity type；
- platform；
- domain；
- tags；
- owner；

过滤可发现资产。

例如：

~~~text
Marketing Agent
-> Marketing Domain View

Snowflake Production Agent
-> Production Snowflake View
~~~

在 Agent 配置里，View 的语义是：

> scope the agent's search to a specific View, limiting which assets it can discover.

所以 View 是：

> **Context Discovery Scope**

而不是：

> Runtime Data Authorization.

---

# 12. 为什么不能把 View 当 Security Boundary？

DataHub Views 主要应用在：

- Search；
- Home；
- Browse；
- Ask DataHub；
- MCP search。

Service Account 也可以绑定 Default View，让 MCP searches 自动限定在该 View。

但是：

- CLI / GraphQL direct queries 需要显式传 view；
- external AI Plugin 有自己的权限模型；
- warehouse access 不由 View enforcement；
- Agent runtime 可能调用 external tools。

因此：

> **View controls context visibility / search scope, not universal execution authorization.**

这是 Agent Governance 最需要避免的误解之一。

---

# 13. Scoped MCP Server — Tool Scope + Context Scope + Instructions

DataHub Scoped MCP Servers 当前可以为不同 use case 创建独立 endpoint：

~~~text
/mcp/finance
/mcp/engineering
/mcp/executive
~~~

每个 server 可以配置：

- tool allow-list；
- base instructions；
- optional View；
- dedicated URL。

~~~mermaid
flowchart TB
    MCP[Scoped MCP Server]
    TOOLS[Selected Tools]
    VIEW[Selected View]
    INST[Instructions]

    TOOLS --> MCP
    VIEW --> MCP
    INST --> MCP
~~~

这是一个非常清晰的：

> Agent Capability Projection.

当前 Docs 标记：

> Private Beta, DataHub Cloud Context Platform.

---

# 14. Scoped MCP 与 Agent View 是两层 Scope

需要区分：

## Agent View

Agent runtime 自身的搜索 scope。

## Scoped MCP Server View

外部 MCP client 通过这个 server 看到的 search / lookup scope。

两者都是：

> context projection.

不是最终 warehouse authorization。

但它们非常适合减少：

- signal noise；
- accidental tool use；
- domain leakage；
- context overload。

---

# 15. Identity — Interactive User vs Autonomous Workflow

DataHub MCP 当前有两类身份模式。

## Interactive

推荐 OAuth + DCR。

每个用户：

- 用自己的 DataHub account 登录；
- 可以走 SSO；
- token scoped to signed-in user。

这很好地保留：

> invoking human identity.

## Autonomous

官方推荐：

> Service Account

用于：

- scheduled scripts；
- CI/CD；
- unattended agents。

Service Account 还可以设置 Default View。

~~~mermaid
flowchart LR
    HUMAN[Interactive Human]
    OAUTH[OAuth Identity]
    AGENT[Autonomous Agent]
    SA[Service Account]

    HUMAN --> OAUTH
    AGENT --> SA
~~~

这是合理的 identity split。

---

# 16. 但 DataHub Custom Tasks 当前不是 Service-account Runtime

这是当前非常重要的限制。

官方 Agents FAQ 明确：

> **Tasks today run as the user who created the task.**

因此 Task 会继承创建者：

- AI Plugins；
- privileges。

未来官方计划：

> support running as a specific service account.

这意味着当前：

~~~text
Agent Persona
!=
Runtime Principal
~~~

Agent 可以有自己的 Name / Instructions / View，但真实权限主体仍可能是：

> Task Creator.

这是当前 Private Beta Agent Governance 中最值得关注的风险。

---

# 17. 为什么“Run as Creator”有治理风险？

假设管理员创建了：

> Nightly Governance Agent

管理员本人拥有：

- 广泛 DataHub privilege；
- Snowflake admin plugin；
- GitHub write plugin。

Agent 后续 schedule 自动运行时：

> 自动继承创建者这些 access。

这意味着 privilege lifecycle 与 human account 绑定。

风险包括：

- creator role 变化；
- creator 离职；
- accidental broad privileges；
- Agent purpose 与 creator permissions 不匹配。

更成熟模式应该是：

~~~text
Agent / Task
-> explicit runtime service account
-> least privilege
-> purpose-bound access
~~~

官方已经把它列为未来方向。

---

# 18. Tools — Capability Allow-list

Custom Agent 可以选择 DataHub Tools：

- search；
- lineage；
- mutations；
- 其他 platform tools。

这实现：

> capability-level allow-list.

它比只依赖 prompt：

~~~text
"Don't mutate metadata"
~~~

要可靠。

~~~mermaid
flowchart LR
    AG[Agent]
    R[Read Tools]
    W[Mutation Tools]

    R --> AG
    W --> AG
~~~

如果 Write Tool 未被启用，Agent 就没有该能力。

原则：

> **Capability restriction should be structural, not instructional.**

---

# 19. AI Plugins — External Capability Expansion

Agent 还可以连接 external MCP servers，例如：

- Snowflake；
- GitHub；
- 其他 systems。

这使 DataHub Agent 从：

> metadata assistant

扩展成：

> cross-stack operational agent.

但也引入一个重要治理问题：

> DataHub 能控制“是否给 Agent 这个 plugin”，但 plugin 内的真正权限仍由 external identity / provider policy 决定。

因此：

~~~text
DataHub Tool Scope
+
External Plugin Permissions
+
Runtime Principal
~~~

共同决定真实 action surface。

---

# 20. Task — Reusable Execution Unit

Task 不是 Agent 本身。

Task 是：

> repeatable instruction assigned to an Agent.

它继承：

- agent instructions；
- tools；
- plugin access；
- default View。

并添加：

- task-specific instructions；
- trigger；
- Allow Decisions。

这形成：

~~~mermaid
flowchart TB
    AG[Agent Persona]
    TASK1[Task A]
    TASK2[Task B]
    TASK3[Task C]

    AG --> TASK1
    AG --> TASK2
    AG --> TASK3
~~~

一个 Agent 可以执行多个不同工作流。

---

# 21. Task Trigger — 当前 Event Model 还比较早期

Task 支持：

- manual；
- hourly/daily/weekly schedule；
- event。

但当前 Event Trigger 官方文档列出的具体事件只有：

> Task Completion

用于 task chaining。

并明确：

> additional event types will be added in future releases.

所以“event-triggered Agent”目前产品能力需要谨慎表述。

**Status: Task event framework exists; event taxonomy currently limited.**

---

# 22. Decision — Human Judgment 进入 Runtime

Decision 是当前 Agent 产品最值得学习的治理 primitive。

当 Task 开启 Allow Decisions：

Agent 可以：

1. 暂停；
2. 提出问题；
3. 提供 choices / free text；
4. 等 Human response；
5. 使用答案继续执行。

~~~mermaid
sequenceDiagram
    participant A as Agent
    participant H as Human

    A->>A: detects ambiguity
    A->>H: Decision request
    H-->>A: answer
    A->>A: resume task
~~~

如果 Human dismiss：

> task run aborts.

这比最终审批更细。

它把 authority checkpoint 插进 reasoning/execution chain。

---

# 23. Decision 的优点：Human 不只是 Approver

传统 HITL：

~~~text
Agent finishes
-> Human approves whole action
~~~

Decision：

~~~text
Agent reaches ambiguity
-> Human supplies missing judgment
-> Agent continues
~~~

这更适合：

- ambiguous ownership；
- sensitive classification；
- business exceptions；
- unclear domain semantics。

因此：

> Decision is a runtime authority injection primitive.

---

# 24. Decision 的当前限制

官方文档目前只说明：

- question；
- options；
- free text；
- response；
- dismiss。

还没有证明 Decision object 本身拥有完整：

- authority type；
- blast radius；
- evidence package；
- approval policy；
- expiration；
- delegation；
- reviewer qualification；

模型。

所以：

> Human checkpoint exists; generalized decision-governance model remains Partial.

---

# 25. Mutation — Agent 可以直接改 Metadata

DataHub MCP 当前 Mutation Tools 支持：

- tags；
- glossary terms；
- ownership；
- domains；
- descriptions；
- structured properties；
- lifecycle stage；
- document authoring。

MCP tool 标记：

> readOnlyHint: false

使 MCP client 可以在调用前要求 confirmation。

这说明：

> DataHub 已经明确允许 Agent 参与 metadata write path。

---

# 26. Proposal Workflow 并不是所有 Mutation 的统一 Gate

MCP 同时提供 proposal tools：

- propose_create_glossary_term；
- propose_lifecycle_stage；
- accept_or_reject_proposals。

官方说明：

> useful when an agent should suggest, not commit, metadata changes.

关键是：

> **Proposal 是可选的 governed workflow，不是当前所有 mutation 的统一必经状态。**

因此 Reference Architecture 里的：

~~~text
Agent Write
-> Proposal Plane
-> Authority
-> Published Truth
~~~

当前只在某些 write 类型 / product workflow 中实现。

---

# 27. Ask DataHub 和 Custom Agents 的 Write Guardrail 不完全相同

DataHub Cloud v2.1 对 Ask DataHub 的规则非常保守：

> every metadata edit requires a human approval before write.

同时提供：

- reasoning trace；
- AI Tool Audit Dashboard。

Custom Agents 则可以配置 mutation tools 并用于 autonomous tasks。

因此需要区分：

~~~text
Ask DataHub conversational writes
vs
Custom Agent automation writes
vs
MCP client writes
~~~

它们的 confirmation / principal / workflow 不完全相同。

不能笼统说：

> “DataHub 所有 Agent 写操作都必须 human approve。”

那并不准确。

---

# 28. Tool Audit — 当前 Runtime Audit 已经很实用

Task Run Detail 当前支持：

- status；
- duration；
- output；
- thinking；
- every tool call；
- inputs；
- outputs。

官方称：

> full trace of every tool call.

这对 operational audit 很有价值。

~~~mermaid
flowchart LR
    RUN[Task Run]
    THINK[Reasoning Trace]
    CALL[Tool Call]
    INPUT[Input]
    OUTPUT[Output]

    RUN --> THINK
    RUN --> CALL
    CALL --> INPUT
    CALL --> OUTPUT
~~~

**Status: Implemented in Agents Private Beta.**

---

# 29. 但 Tool Trace != Full Decision Provenance

它还不能自动等于：

> 为什么 Agent 相信了这条企业事实？

完整 Decision Provenance 还需要：

- exact retrieved context versions；
- context authority；
- freshness；
- semantic model version；
- policy result；
- Agent version；
- tool results；
- Human Decisions。

当前 Task tool trace 解决：

> execution trace

而不是完整：

> epistemic trace.

所以：

**Runtime Audit: strong.  
Decision Replay: still Partial / Unclear.**

---

# 30. Governance Propagation：Agent 风险来自 Data Dependencies

Agent Registry 的 governance across lineage 非常值得注意。

如果 Agent 读取：

> Highly Confidential Dataset

DataHub 可以：

- 把 classification propagate 到 Agent；
- Agent health 变红；
- 创建 incident。

这是一种：

> derived Agent risk state.

~~~mermaid
flowchart LR
    D[Dataset Risk]
    L[Lineage]
    A[Agent]
    R[Agent Risk / Incident]

    D --> L --> A --> R
~~~

这说明 DataHub 对 Agent Governance 的理解不是只看 Agent config。

还看：

> Agent consumes what.

---

# 31. Governance Propagation 的局限

Lineage propagation 很适合：

- classification；
- impact。

但不代表：

> classification automatically enforces runtime access.

例如：

Agent 被标记为 Highly Confidential：

这可以提醒治理团队。

但 warehouse 是否真正阻止 unauthorized query：

> 仍然属于 external runtime authorization / policy plane.

所以：

> governance visibility != policy enforcement.

---

# 32. Agent Governance 当前的四个 Scope

综合产品，可以区分：

## 1. Metadata Scope

Agent Registry / entity governance。

## 2. Context Scope

View / Scoped MCP 限定发现内容。

## 3. Capability Scope

Tool allow-list / AI Plugins。

## 4. Identity Scope

OAuth user / task creator / service account / external credentials。

~~~mermaid
flowchart TB
    AG[Agent]

    M[Metadata Scope] --> AG
    C[Context Scope] --> AG
    T[Tool Scope] --> AG
    I[Identity Scope] --> AG
~~~

一个真正安全的 Agent 必须四者一致。

---

# 33. Scope Drift 是一个很现实的风险

例如：

~~~text
Agent View:
  Marketing only

External Snowflake Plugin:
  ACCOUNTADMIN

Task Principal:
  Admin creator

Tool Set:
  SQL Execute
~~~

表面上 Agent 是：

> Marketing-scoped

实际执行权限可能远大于 Marketing。

所以：

> **Context Scope cannot compensate for broad Runtime Identity.**

这是当前 DataHub Agents Private Beta 最需要用户理解的地方。

---

# 34. Agent Registry vs Runtime Agent 的 Identity Gap

Agent Registry 里的 Agent Entity 有：

- identity；
- version；
- owner；
- dependencies。

但 Agents runtime 当前 Task 的 runtime principal 是：

> creator user.

这意味着两个不同 identity：

~~~text
Governed Agent Identity
Runtime Security Principal
~~~

当前并没有完全统一。

~~~mermaid
flowchart LR
    META[Agent Metadata Identity]
    TASK[Task]
    USER[Creator Principal]
    ACTION[External Action]

    META --> TASK
    USER --> TASK --> ACTION
~~~

未来 service-account execution 会更接近：

~~~text
Agent / Task
-> dedicated principal
~~~

---

# 35. Interactive Agent 场景反而 Identity 更清晰

MCP OAuth + DCR 可以让：

> 每个 Human user 用自己的 DataHub login。

例如 Claude / ChatGPT / Cursor / Snowflake / Databricks 等支持时：

~~~text
Human Principal
-> OAuth
-> DataHub MCP
-> DataHub permissions
~~~

这保留：

- user identity；
- SSO；
- per-user token。

所以：

> Interactive Agent identity currently can be cleaner than autonomous Task identity.

这是一个有趣的现实差异。

---

# 36. Agent Governance Product Matrix

| Capability | Status | Notes |
|---|---|---|
| Agent Registry | Implemented | Cloud v2.1 |
| External Agent Cataloging | Implemented | SDK / CLI / API / frameworks |
| Agent Ownership | Implemented | normal metadata ownership |
| Skills as Entities | Implemented | reusable capability |
| Tools as API Entities | Implemented | typed signature |
| MCP Server as Service | Implemented | versioned tool contract |
| Agent-to-Data Lineage | Implemented | impact analysis |
| Agent Versioning | Implemented | version sets + timeline |
| Classification Propagation | Implemented | lineage automation + incident |
| Custom Agents | Private Beta | Cloud-only |
| Tool Allow-list | Private Beta | per Agent |
| AI Plugins | Private Beta | external MCP |
| Agent View Scope | Private Beta | discovery scope |
| Task Schedules | Private Beta | hourly/daily/weekly |
| Task Event Trigger | Partial | currently Task Completion |
| Decisions | Private Beta | human checkpoint |
| Task Tool-call Audit | Private Beta | inputs/outputs trace |
| Task Service-account Identity | Roadmap | currently creator user |
| Managed MCP OAuth per User | Implemented | Cloud v1.0.2+ |
| Autonomous MCP Service Account | Implemented | recommended |
| Service-account Default View | Implemented | Cloud v1.0.0+ / Core v1.6.0+ |
| Scoped MCP Servers | Private Beta | separate URL/tool/view/instructions |
| MCP Mutation | Implemented | Core/Cloud supported versions |
| MCP Proposal Tools | Partial | selected metadata types |
| Universal Proposal Gate | Not evidenced | direct mutation still exists |
| Full Decision Replay | Unclear | no complete context-version replay evidence |

---

# 37. DataHub Agent Governance 的核心 Strength

当前最强的地方不是 Agent builder UI。

而是：

> **Agent runtime 被放回 DataHub 原有的 metadata graph / lineage / ownership / incident / versioning 世界。**

换句话说：

~~~text
Agent governance
does not start from
Agent prompt config.

It starts from
enterprise metadata graph.
~~~

这是 DataHub 相比单独 Agent Framework 最值得研究的差异。

---

# 38. 当前最大的 Product Gap：Delegation Model

我们的 Reference Architecture 希望：

~~~text
Human Principal
-> delegates purpose-limited authority
-> Agent Identity
-> Task Principal
-> Tool / Data Access
~~~

当前产品更接近：

### Interactive MCP

~~~text
Human
-> OAuth
-> own permissions
~~~

很好。

### Autonomous MCP

~~~text
Agent
-> Service Account
~~~

也比较清晰。

### DataHub Custom Task

~~~text
Task
-> Creator User permissions
~~~

这里是明显的过渡状态。

所以 Agent Governance 下一步最关键的演进之一，很可能不是更多 tools。

而是：

> **explicit delegated runtime identity.**

---

# 39. 第二个 Gap：Proposal / Mutation Policy 还没有统一

DataHub 已经拥有：

- direct MCP mutations；
- proposal tools；
- Ask DataHub human approval；
- Context Proposal workflow；
- Decision checkpoint。

这些都是 governance primitives。

但目前它们仍然是：

> multiple workflows.

我们的 Reference Architecture 希望最终形成：

~~~text
semantic risk class
-> required write path
-> required authority
~~~

例如：

~~~text
usage observation
-> direct append

description suggestion
-> proposal

business definition
-> domain SME approval

classification
-> security authority

runtime data write
-> external policy
~~~

当前 DataHub 产品还没有从公开文档体现一个统一 risk-based write policy engine。

---

# 40. 第三个 Gap：Context Scope 与 Execution Scope 仍分离

这是架构上合理的。

但用户容易误解。

DataHub 能控制：

- Agent View；
- scoped MCP；
- DataHub Tools；
- plugin connection。

外部 system 控制：

- warehouse role；
- GitHub permission；
- SaaS scopes；
- runtime policy。

所以 production Agent governance 必须同时审计：

~~~text
What can Agent see?
What can Agent call?
Who is Agent running as?
What can that principal do?
~~~

任何一个过宽，整体就过宽。

---

# 41. Product Verdict

截至 2026-09-28：

## Agent Registry

**成熟度：高**

DataHub 已经很好地把 Agent 作为 governed metadata asset：

- identity；
- ownership；
- skill/tool dependency；
- data lineage；
- version；
- audit；
- propagated governance。

## Agent Runtime

**成熟度：早期但架构清晰**

Custom Agents / Tasks / Decisions 是 Private Beta。

已经拥有：

- scope；
- tool allow-list；
- external plugins；
- schedules；
- decisions；
- tool-call audit。

## Agent Security / Delegation

**成熟度：Partial**

尤其：

- Task currently runs as creator；
- View != authorization；
- external plugin permission independent；
- proposal workflow not universal。

因此最准确的判断：

> **DataHub 的 Agent governance graph 已经比 Agent runtime 本身成熟；它的 metadata governance foundation 很强，而 runtime identity / delegation / unified write-governance 仍处于明显演进阶段。**

---

# 42. 下一篇

## 05 — OSS vs Cloud Context Architecture

最后把边界彻底拆清：

~~~mermaid
flowchart LR
    OSS[DataHub Core]
    MCP[Open / Self-hosted MCP]
    CLOUD[DataHub Cloud]
    CTX[Context Platform]
    AG[Agents]

    OSS --> MCP
    OSS --> CLOUD
    CLOUD --> CTX --> AG
~~~

重点回答：

- Entity / Aspect / Graph / Lineage 哪些属于 OSS substrate？
- MCP 哪些是 Core 可用？
- Context Documents / generation / proposal / eval 哪些是 Cloud？
- Agent Registry / Agents 分别是什么 availability？
- 如果只使用 OSS，能否自己构建 Context Layer？
- Cloud 的真正增值层到底是什么？

---

# Sources

- DataHub Docs, Agent Registry  
  https://docs.datahub.com/docs/features/feature-guides/agent-registry

- DataHub Docs, Agents  
  https://docs.datahub.com/docs/features/feature-guides/agents

- DataHub Docs, MCP Server  
  https://docs.datahub.com/docs/features/feature-guides/mcp

- DataHub Docs, Scoped MCP Servers  
  https://docs.datahub.com/docs/features/feature-guides/scoped-mcp-servers

- DataHub Docs, Views  
  https://docs.datahub.com/docs/features/feature-guides/views/overview

- DataHub, Cloud v2.1  
  https://datahub.com/blog/datahub-cloud-2-1/

- DataHub, Cloud v2.2  
  https://datahub.com/blog/datahub-cloud-v2-2/
