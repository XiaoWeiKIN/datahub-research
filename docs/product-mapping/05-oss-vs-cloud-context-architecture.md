# 05 — OSS vs Cloud Context Architecture

## DataHub Core 能做到哪里？DataHub Cloud 真正增加的 Context / Agent 层是什么？

**Snapshot:** 2026-09-28  
**Scope:** DataHub Core (OSS / self-hosted) vs DataHub Cloud

---

# 0. 当前结论

最容易产生的误解有两个：

### 误解 A

> Context / Agent 能力都是 DataHub Cloud，OSS 只是传统 Catalog。

不准确。

DataHub Core 已经拥有 Context Platform 最重要的一批 substrate：

- Entity / Aspect / URN metadata standard；
- metadata graph；
- lineage；
- ownership；
- domains；
- glossary；
- Context Documents；
- search；
- policies；
- incidents；
- data contracts；
- APIs / SDK；
- self-hosted MCP Server；
- MCP read/write tools；
- service accounts / Views；
- extensible metadata model。

### 误解 B

> DataHub Cloud 只是托管版 OSS。

也不准确。

DataHub Cloud 现在明显增加了一层：

- managed Context Intelligence；
- generated Context lifecycle；
- evals；
- Data Expert / SME proposal review；
- Context Activation workflow；
- Ask DataHub；
- query-time Search Access Controls；
- richer Observability automation；
- Agent Registry；
- custom Agents / Tasks / Decisions；
- managed OAuth MCP；
- scoped MCP servers；
- AI Tool Audit；
- notifications / managed operational tooling。

所以更准确的架构：

~~~mermaid
flowchart TB
    subgraph CORE[DataHub Core / Open Metadata Substrate]
        MODEL[Entity / Aspect / URN]
        GRAPH[Graph / Lineage]
        GOV[Ownership / Tags / Glossary / Domains]
        DOC[Context Documents]
        SEARCH[Search / API]
        OBS[Core Assertions / Incidents / Contracts]
        MCP[Self-hosted MCP]
    end

    subgraph CLOUD[DataHub Cloud Context / Agent Layer]
        INTEL[Context Intelligence]
        EVAL[Evals / Proposal Review]
        ASK[Ask DataHub]
        SAC[Query-time Search Access Controls]
        REG[Agent Registry]
        AG[Agents / Tasks / Decisions]
        MMCP[Managed / Scoped MCP]
        CLOUDOBS[Advanced Observability / Automations]
    end

    CORE --> CLOUD
~~~

一句话：

> **Core 提供 Context substrate；Cloud 正在产品化 Context lifecycle、Agent governance 和 managed operational control surfaces。**

---

# 1. DataHub Core 的核心身份：Open Metadata Standard + Runtime

DataHub 当前文档把 Entity / Aspect / URN / metadata events 明确放在：

> Open Source DataHub Metadata Standard

这一层是 Context Platform 的根。

Core 拥有：

- schema-first metadata model；
- typed entities；
- typed aspects；
- relationships；
- MCP / MCL；
- APIs / SDKs；
- self-hosted deployment。

所以：

> DataHub Cloud 的 Context Platform 不是绕开 Core 另建一套 AI context database。

而是建立在同一 metadata substrate 之上。

---

# 2. Core 能不能做 Context Documents？

可以。

当前 Context Documents 文档的 Feature Availability 同时列出：

- Self-Hosted DataHub；
- DataHub Cloud。

Context Documents 在 DataHub 中是一等实体，可以：

- 创建 runbook / FAQ / policy / decision log；
- 加 owner / tag / domain / glossary term；
- 关联 dataset / dashboard / chart；
- Draft / Published；
- version history；
- import Notion / Confluence / GitHub；
- Python SDK；
- MCP access。

其中一个明确 Cloud-only 差异：

> GitHub document sync-back is DataHub Cloud only.

此外：

> Ask DataHub integration is Cloud only.

所以：

**Context Document entity / manual knowledge substrate：Core + Cloud。  
Managed AI consumption surfaces：Cloud 更完整。**

Source:

- https://docs.datahub.com/docs/features/feature-guides/context/context-documents

---

# 3. Core 能不能做 MCP？

可以，而且这是非常重要的边界。

DataHub MCP 官方文档明确：

~~~text
Managed MCP Server
-> DataHub Cloud

Self-Hosted MCP Server
-> DataHub Core
~~~

Self-hosted MCP：

- 连接 GMS endpoint；
- 使用 DataHub PAT；
- 可服务 Claude / Cursor / 其他 MCP clients；
- 可以 search；
- get entities；
- lineage；
- schema；
- query history；
- SQL context；
- documents；
- governance proposals；
- mutation。

所以：

> **MCP 本身不是 Cloud-only Context capability。**

一个 DataHub Core 用户完全可以：

~~~text
DataHub Core
-> self-hosted MCP
-> Claude / Cursor / custom agent
~~~

构建 machine-consumable metadata/context layer。

Source:

- https://docs.datahub.com/docs/features/feature-guides/mcp

---

# 4. Core MCP 甚至可以 Write

DataHub MCP mutation tools 从 mcp-server-datahub v0.5.0+ 可用。

包括：

- tags；
- glossary terms；
- owners；
- domains；
- descriptions；
- structured properties；
- lifecycle stages；
- documents；
- glossary authoring；
- proposals。

通过：

~~~text
TOOLS_IS_MUTATION_ENABLED=true
~~~

启用。

这意味着：

> Agent Read / Write Metadata 也不是 Cloud-exclusive primitive。

Cloud 的差异更多发生在：

- managed endpoint；
- OAuth；
- UI-managed scoped MCP；
- Ask DataHub；
- integrated audit；
- Context proposal lifecycle；
- custom Agent runtime。

---

# 5. Cloud 的 MCP 增值：Managed Identity

DataHub Cloud v1.0.2+ 提供：

> OAuth2 + Dynamic Client Registration

每个 interactive user：

- 用自己的 DataHub account 登录；
- 可以走企业 SSO；
- token scoped to signed-in user；
- 自动 refresh。

而 self-hosted Core MCP 当前文档路径主要是：

> GMS URL + PAT。

所以 MCP 层的一个明确 Cloud 增值是：

~~~text
Managed MCP
+ per-user OAuth
+ SSO identity propagation
~~~

而不仅是“少部署一个服务”。

---

# 6. Autonomous MCP 在 Core 仍然可做

官方 MCP 文档对 unattended workflows 推荐：

> Service Account

Service account token 可以用于 MCP。

并且从：

- DataHub Cloud v1.0.0+
- DataHub Core v1.6.0+

都支持：

> Service Account Default View

用于限制 MCP search 到某个 View。

因此一个 Core 架构完全可以：

~~~mermaid
flowchart LR
    SA[Service Account]
    VIEW[Default View]
    MCP[Self-hosted MCP]
    AG[External Agent]

    SA --> MCP
    VIEW --> MCP
    MCP --> AG
~~~

这已经具备相当实用的 Agent context scoping。

---

# 7. Core 的 Context Scope：Views 有用，但 Search ACL 较弱

Core 有：

- policies；
- View Entity；
- Views；
- service-account default View。

但 Search Access Controls 文档明确区分：

## DataHub Cloud

支持：

> query-time search filtering

search / browse / direct entity access 根据 View Entity policy 过滤。

## DataHub Core

可以：

~~~text
VIEW_AUTHORIZATION_ENABLED=true
~~~

做：

> entity page gating / post-search masking

但：

> 不提供 Cloud 同等级 query-time search filtering。

因此：

~~~text
Core:
  metadata authorization exists
  but search result filtering is weaker

Cloud:
  query-time default-deny search authorization
~~~

这对 Agent 很重要，因为：

> Agent 的第一步往往是 search / discovery。

Source:

- https://docs.datahub.com/docs/features/feature-guides/search-access-controls

---

# 8. Core 的 Observability 并不弱

不能把：

> Data Quality = Cloud only

这样理解。

当前 Observability 文档明确：

### Core 可用

- Incidents；
- Data Contracts；
- API/SDK assertion management；
- custom assertions；
- ingestion-driven profiles；
- external quality integrations；
- operations events。

### Cloud 增值

- anomaly detection；
- Data Health Dashboard；
- Monitoring Rules；
- Slack / Teams notification；
- observability agent；
- richer failure trend analysis。

所以：

> Core 有 quality data model / incident / contract substrate；Cloud 更偏自动化、智能检测、managed operations。

Source:

- https://docs.datahub.com/docs/features/feature-guides/observe

---

# 9. Core 的 Business Context 也已经相当完整

Core substrate 包括：

- ownership；
- glossary；
- domains；
- tags；
- structured properties；
- Context Documents；
- logical models；
- metrics / semantic model metadata；
- data products；
- service catalog primitives（具体 availability需按 feature核实）。

所以如果目标只是：

> 构建 enterprise context graph

Core 已经拥有非常多必要 building blocks。

真正 Cloud-heavy 的部分是：

> **Context 自动生产、验证、激活和 Agent workflow 产品化。**

---

# 10. Context Intelligence / Generation — Cloud

当前 Configure Context Generation 是：

> DataHub Context Public Beta

产品由 DataHub Cloud v2.2 向 Cloud customers 开放。

它负责：

- 分析 query history；
- schema；
- dbt；
- Looker / BI；
- 生成 semantic anchors / Context Documents；
- domain/container scoping；
- auto-publish config；
- Context Curator agent troubleshooting。

这是 Cloud Context Platform 的明确增值层。

Core 可以拥有：

> query entities / usage / documents / graph

但官方目前没有提供：

> self-hosted Context Intelligence generation workflow

作为 Core feature。

**Status: Cloud Public Beta.**

Sources:

- https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation
- https://datahub.com/blog/datahub-cloud-v2-2/

---

# 11. Context Proposal / Eval / SME Review — Cloud

当前 managed Context workflow：

~~~text
Generated Context
-> Proposal
-> Metadata Eval
-> Data Expert / SME
-> Publish
-> Agent
~~~

在 DataHub Cloud Context Public Beta 中提供。

包括：

- proposal Task Center；
- SQL Generation eval；
- Generic Catalog Q&A eval；
- must-reference / must-not-reference assets；
- comments；
- Test on Ask DataHub；
- publish / unpublish；
- human edits preserved across regeneration。

这是 Cloud 与 Core 最本质的差异之一：

> **Core 可以保存 Context Documents；Cloud 提供 managed AI-generated Context promotion lifecycle。**

---

# 12. Context Activation — 要拆成两层看

## Primitive Activation

~~~text
DataHub
-> MCP
-> Agent
~~~

Core 可以。

## Managed Context Activation

~~~text
Published generated context
-> MCP / Ask DataHub / managed skills
-> Agent
~~~

当前 DataHub Context Activation 是 Cloud Public Beta。

所以不能简单写：

> Context Activation = Cloud-only。

更准确：

> **MCP-based activation is Core-capable；managed validated Context lifecycle activation is Cloud product layer。**

---

# 13. Ask DataHub — Cloud

当前 Ask DataHub 文档明确：

- conversational AI assistant；
- grounded in metadata graph + organizational knowledge；
- UI / Slack / Teams 等 surface。

Context Documents 文档明确：

> Ask DataHub is available in DataHub Cloud only.

Ask DataHub 当前可：

- search trusted data；
- impact analysis；
- quality；
- policies / runbooks；
- SQL draft；
- context retrieval；
- 部分 governed write。

所以 Ask DataHub 是：

> Cloud-native human-facing / agentic consumption layer。

Core 用户可以用外部 MCP client 实现类似消费模式，但那不是 Ask DataHub 产品本身。

---

# 14. Agent Registry — 当前是 Cloud Product Feature

Agent Registry 是 DataHub Cloud v2.1 引入的产品能力。

官方当前文档描述：

- AI Agent；
- Skill；
- Tool/API；
- MCP Server/Service；
- lineage；
- ownership；
- versions；
- eval / reliability signals。

它复用 metadata graph，但：

> 当前发布 / enablement 语境明确是 DataHub Cloud v2.1。

公开材料没有建立完整 OSS parity。

因此在 Product Mapping 中：

**Agent Registry：Cloud product feature。**

但需要注意：

> 它使用的 entity / graph abstraction 与 DataHub open metadata substrate 同源。

也就是说未来不应把：

~~~text
Agent entity concept
~~~

和：

~~~text
Agent Registry Cloud UI / workflow
~~~

混为一谈。

Sources:

- https://docs.datahub.com/docs/features/feature-guides/agent-registry
- https://datahub.com/blog/datahub-cloud-2-1/

---

# 15. Custom Agents / Tasks / Decisions — Cloud-only Private Beta

这条边界非常明确。

当前 Agents Docs：

> Agents is a DataHub Cloud-only feature currently in Private Beta.

包含：

- custom agent persona；
- instructions；
- DataHub tools；
- AI Plugins；
- View scope；
- scheduled / event tasks；
- Decisions；
- run history；
- tool-call audit。

所以：

> DataHub Agent Runtime 不是 Core capability。

Core 能给 external agents context。

Cloud 才开始自己承载 Agent execution workflow。

Source:

- https://docs.datahub.com/docs/features/feature-guides/agents

---

# 16. Scoped MCP Servers — Cloud Context Layer

普通 MCP：

- Core self-host；
- Cloud managed。

Scoped MCP Server：

- 独立 URL；
- tool allow-list；
- instructions；
- View；
- domain-specific endpoint。

DataHub Cloud v2.1 把它作为 Context Platform feature 发布。

当前 feature guide / release material仍属于 Cloud Context program。

所以：

> **MCP protocol surface is shared；MCP control plane / endpoint productization is Cloud differentiation。**

---

# 17. Search Access Control 是一个非常典型的 Cloud Productization Example

底层：

- Policy；
- View Entity；
- metadata authorization

Core 有。

Cloud 添加：

> query-time search enforcement

把权限直接推入 search retrieval path。

这在 Agent context 场景非常重要：

~~~text
Context Graph
-> Search
-> Agent
~~~

如果 search 在 query-time 不 enforce：

> unauthorized entity name / metadata 可能先出现在 retrieval surface，再在 entity page 被挡。

因此 Cloud 的 Search Access Controls 实际上是：

> Agent-era retrieval security hardening.

---

# 18. Cloud 的 Observability 增值也是相似模式

Core 已经有：

- quality model；
- assertions；
- incidents；
- contracts；
- API integration。

Cloud 增加：

~~~text
automation
+ anomaly detection
+ fleet-wide health
+ notifications
+ agent-assisted root cause
~~~

同样说明：

> Cloud 的主要价值经常不是“发明 metadata entity”，而是“把 substrate 变成 managed operating system”。

---

# 19. OSS vs Cloud 不应该按 Feature List 理解

最合理的模型是：

~~~mermaid
flowchart TB
    subgraph OPEN[Open / Core Substrate]
        STANDARD[Metadata Standard]
        GRAPH[Graph]
        EVENTS[Events]
        SEARCH[Search]
        API[API / SDK]
        DOC[Documents]
        GOV[Governance Metadata]
        QUAL[Quality Metadata]
        MCP[MCP Server]
    end

    subgraph MANAGED[Cloud Operating Layer]
        MANAGE[Managed Service]
        INTEL[AI Context Generation]
        REVIEW[Proposal / Eval / Review]
        ACTIVATION[Managed Activation]
        SEARCHACL[Search-time ACL]
        AUDIT[AI Audit]
        REG[Agent Governance]
        RUNTIME[Agent Runtime]
        AUTO[Automation / Notifications]
    end

    OPEN --> MANAGED
~~~

所以 DataHub Cloud 更像：

> **managed + intelligent + governed operating layer on top of the open metadata substrate。**

---

# 20. 如果只有 DataHub Core，能不能自己构建 Context Layer？

答案是：

> **可以构建一个很强的 Context Layer substrate，但需要自己补很多 lifecycle / governance / AI operations。**

Core 已经给你：

~~~text
Identity
Graph
Lineage
Documents
Ownership
Glossary
Domains
Quality
Events
Search
API
MCP
Mutation
Service Accounts
Views
~~~

你完全可以在上面自己做：

~~~text
custom context generation
custom eval
custom proposal workflow
custom agent
custom policy integration
custom audit
~~~

技术上没有被阻止。

---

# 21. 但“能自己做”与“已经是同一个产品”不是一回事

例如 Core MCP + Documents 可以构成：

~~~text
manual curated context
-> self-hosted MCP
-> external agent
~~~

这已经是 Context Layer。

但 Cloud Context Platform 进一步提供：

~~~text
automated context generation
-> eval
-> domain SME proposal
-> publish
-> managed activation
-> audit
~~~

如果企业自己在 Core 上实现这些：

> 你实际上是在自己建设 Context Platform operating layer。

所以购买 / 使用 Cloud 的价值主要发生在：

> **operationalization, not conceptual possibility。**

---

# 22. Core-first Architecture

一个完全 OSS / self-managed 的 Agent context stack可以：

~~~mermaid
flowchart TB
    SOURCES[Data Sources]
    CORE[DataHub Core]
    DOC[Context Documents]
    MCP[Self-hosted MCP]
    EXT[External Agent]
    SEM[External Semantic Layer]
    AUTH[External IAM / Policy]

    SOURCES --> CORE
    DOC --> CORE
    CORE --> MCP --> EXT
    EXT --> SEM
    EXT --> AUTH
~~~

这种模型已经可以：

- trusted asset discovery；
- lineage；
- owner；
- docs；
- glossary；
- quality signals；
- SQL context；
- metadata write-back。

---

# 23. Cloud Context Architecture

Cloud 在这个基础上加入：

~~~mermaid
flowchart TB
    SOURCES[Sources]
    CORE[Metadata Substrate]
    GEN[Context Intelligence]
    REVIEW[Eval / SME Review]
    PUB[Published Context]
    MCP[Managed / Scoped MCP]
    ASK[Ask DataHub]
    REG[Agent Registry]
    AG[Agents / Tasks / Decisions]
    EXT[External Data / Tool Systems]

    SOURCES --> CORE --> GEN --> REVIEW --> PUB
    PUB --> MCP
    PUB --> ASK
    PUB --> AG

    REG --> AG
    AG --> EXT
~~~

这里 Context lifecycle 本身成为产品。

---

# 24. “Open Core” 和 “Cloud Differentiation”的战略结构

从架构角度看，DataHub 当前很聪明的一点是：

> 把长期 ecosystem / interoperability 价值放在 open metadata substrate。

开放层：

- entity model；
- graph；
- ingestion ecosystem；
- events；
- APIs；
- SDK；
- MCP self-host。

商业差异层更多在：

- automation；
- managed security；
- AI context generation；
- eval；
- human workflow；
- agent governance；
- operational UX。

这减少一个风险：

> Context Platform 变成 proprietary context silo。

至少 substrate 与大量 access surface 仍然是开放的。

---

# 25. 但 Cloud Context 也会形成新的 Product-level Lock-in Surface

即使底层 metadata standard 开放，Cloud 上层 lifecycle 仍可能形成：

- generated Context Documents；
- eval configs；
- Agent configs；
- Decisions；
- Task history；
- AI audit；
- proposal workflow；
- managed OAuth/scoping。

这些 operating state 是否：

- 可导出；
- 可迁移；
- 有稳定 API；
- 有开放 schema；

将决定 Context Platform 的真实 portability。

这是后续值得观察的问题。

---

# 26. Feature Matrix — Context / Agent Perspective

| Capability | DataHub Core | DataHub Cloud |
|---|---|---|
| Entity / Aspect / URN | Yes | Yes |
| Metadata Graph | Yes | Yes |
| Lineage | Yes | Yes |
| Search | Yes | Yes |
| Ownership / Tags / Domains / Glossary | Yes | Yes |
| Context Documents | Yes | Yes |
| Document version history | Yes | Yes |
| Document import | Yes | Yes |
| GitHub document sync-back | No / not documented | Yes |
| API / SDK | Yes | Yes |
| MCP Server | Self-hosted | Managed + self-host possible |
| MCP Read Tools | Yes | Yes |
| MCP Mutation Tools | Yes, supported versions | Yes |
| Service Accounts | Yes | Yes |
| Service Account Default View | v1.6.0+ | v1.0.0+ |
| Policies / View Entity | Yes | Yes |
| Query-time Search Access Control | No | Yes |
| Entity-page view gating | Yes | Yes |
| Incidents | Yes | Yes |
| Data Contracts | Yes | Yes |
| Assertions / custom quality signals | Yes | Yes |
| Anomaly Detection | No | Yes |
| Data Health Dashboard / Monitoring Rules | No | Yes |
| Notifications | limited / no Cloud delivery surfaces | Yes |
| Ask DataHub | No | Yes |
| Context Intelligence / Generation | No official Core offering | Public Beta |
| Metadata Evals for Context | No official Core offering | Public Beta |
| Context Proposal / SME Review | No official Core offering | Public Beta |
| Managed Context Activation workflow | No | Public Beta |
| Agent Registry | No established Core parity | Cloud feature |
| Custom Agents / Tasks / Decisions | No | Private Beta |
| Managed OAuth MCP / DCR | No | Yes |
| Scoped MCP Servers | No official Core product | Cloud Context feature |
| AI Tool Audit Dashboard | No | Yes |
| Runtime warehouse authorization | External | External |
| Semantic query runtime | External | External |

> 注：这是 Context / Agent 架构视角的 mapping，不是 DataHub 官方定价比较表。

---

# 27. 一个重要细节：Docs “Feature Availability” 需要结合正文解释

DataHub Docs 的页面顶部经常同时显示：

~~~text
DataHub Core (OSS)
DataHub Cloud
~~~

但具体 capability 会在正文说明差异。

例如：

### MCP

明确：

- managed MCP = Cloud；
- self-hosted MCP = Core。

### Search Access Control

明确：

- query-time filtering = Cloud；
- OSS 只提供 entity page gating / masking。

### Context Documents

entity 本身支持 self-hosted + Cloud，但：

- Ask DataHub = Cloud；
- GitHub sync-back = Cloud-only。

因此不能只看页面上方 Availability badge。

必须看：

> **具体 capability 的 execution path。**

---

# 28. DataHub Core 是 Context Platform 吗？

这取决于“Context Platform”的定义。

如果定义是：

> 一个 graph-based substrate，连接 technical / operational / business context，并通过 API / MCP 提供给 Human/Agent。

那么：

> **DataHub Core 已经可以构成 Context Platform 的基础版本。**

如果定义是 DataHub 当前商业产品：

> automated context intelligence + eval + SME workflow + managed activation + Agents。

那么：

> **这是 DataHub Cloud Context Platform。**

所以正确区分：

~~~text
Context Architecture
!=
DataHub Cloud Context Platform Product
~~~

---

# 29. DataHub Cloud 的真正增值层：Operating Model

我们的最终判断：

DataHub Cloud 的核心差异不是：

> 有 graph，OSS 没 graph。

也不是：

> 有 MCP，OSS 没 MCP。

而是：

> **把 graph / metadata / MCP 组织成一个可运营的 enterprise Context + Agent lifecycle。**

主要包括：

- 自动生成；
- review queue；
- eval；
- publication；
- scoped managed access；
- human checkpoints；
- Agent governance；
- automation；
- audit；
- managed identity；
- managed reliability。

这和传统 SaaS “省运维”相比更深一层。

它是：

> productized operating model.

---

# 30. 对我们研究 Context Layer 的意义

这个 OSS / Cloud 边界验证了一个重要观点：

> Context Layer 的 substrate 和 operating layer 是两个不同问题。

## Substrate

回答：

- context 怎么表示？
- identity 怎么建立？
- relationships 怎么连接？
- change 怎么流动？
- API 怎么访问？

## Operating Layer

回答：

- 谁生成 context？
- 谁验证？
- 如何 publish？
- 如何限制 Agent？
- 如何 eval？
- 如何 rollback？
- 如何 audit？
- 如何持续维护？

DataHub Core 强在前者。

DataHub Cloud 正在产品化后者。

---

# 31. 这可能是 DataHub 最值得借鉴的产品战略

一个 Context Platform 如果把：

> context model + context lifecycle + agent runtime

全部做成封闭 SaaS：

企业会担心：

- lock-in；
- context portability；
- Agent portability；
- architecture dependency。

DataHub 当前路线则更接近：

~~~text
Open substrate
+
Commercial operating layer
~~~

这可能比“所有 Context 都锁在 proprietary AI memory system”更健康。

---

# 32. 但长期需要观察的边界

## 1. Cloud-generated context portability

Context Documents 可以 API/MCP 访问，但 eval / proposal / lifecycle metadata 是否有完整 export model？

## 2. Agent Registry portability

Agent / Skill / Tool entities能否完整通过开放 metadata standard 表达并被 self-hosted 系统消费？

## 3. Agent runtime portability

Tasks / Decisions 是否有 public stable API / schema？

## 4. Scoped MCP portability

能否把 server configuration 当 declarative config export？

## 5. Eval portability

golden questions / evaluation rules 是否可迁移？

这些将决定：

> Open substrate 是否真的能防止 upper-layer lock-in。

---

# 33. Phase 2 总结

Product Mapping 到这里形成完整链路：

~~~mermaid
flowchart LR
    REF[Reference Architecture]
    MODEL[Metadata Model]
    LIFE[Context Lifecycle]
    GOV[Agent Governance]
    BOUND[OSS vs Cloud Boundary]

    REF --> MODEL --> LIFE --> GOV --> BOUND
~~~

我们的当前结论：

### DataHub Core

是一个成熟的：

> **Open Metadata / Context Substrate**

### DataHub Cloud

正在成为：

> **Managed Context Lifecycle + Agent Governance Operating Layer**

这两者建立在同一个 metadata graph / standard 上。

---

# 34. 下一阶段建议

第二阶段到这里也可以收束。

下一步进入 Phase 3：

## Concrete Case Studies

不要再抽象。

选真实任务，把整个架构走一遍。

推荐第一个：

> **Case 01 — Analytics Agent: “过去 90 天 Enterprise Customer Net Revenue 是多少？同比如何？为什么我应该相信这个数字？”**

完整分析：

~~~text
Intent
-> Context Retrieval
-> Metric / Authority Resolution
-> Freshness / Quality
-> Semantic Execution
-> Runtime Authorization
-> Query
-> Evidence
-> Answer
-> Write-back / Audit
~~~

它会验证前两阶段的理论到底是不是可用的。

---

# Sources

- DataHub Docs, MCP Server  
  https://docs.datahub.com/docs/features/feature-guides/mcp

- DataHub Docs, Context Documents  
  https://docs.datahub.com/docs/features/feature-guides/context/context-documents

- DataHub Docs, Search Access Controls  
  https://docs.datahub.com/docs/features/feature-guides/search-access-controls

- DataHub Docs, Data Quality & Observability  
  https://docs.datahub.com/docs/features/feature-guides/observe

- DataHub Docs, Configure Context Generation  
  https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation

- DataHub Docs, Validate Context Proposals  
  https://docs.datahub.com/docs/managed-datahub/context/review-context-proposals

- DataHub Docs, Activate Context  
  https://docs.datahub.com/docs/managed-datahub/context/activate-context

- DataHub Docs, Agent Registry  
  https://docs.datahub.com/docs/features/feature-guides/agent-registry

- DataHub Docs, Agents  
  https://docs.datahub.com/docs/features/feature-guides/agents

- DataHub Cloud v2.1  
  https://datahub.com/blog/datahub-cloud-2-1/

- DataHub Cloud v2.2  
  https://datahub.com/blog/datahub-cloud-v2-2/
