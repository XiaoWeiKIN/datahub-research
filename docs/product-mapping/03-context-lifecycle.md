# 03 — Context Lifecycle Product Mapping

## 从 Query History 到 Agent Consumption：DataHub Context Platform 的真实状态机

**Snapshot:** 2026-09-28  
**Scope:** DataHub Cloud Context Platform lifecycle  
**Availability:** Public Beta

---

# 0. 当前结论

DataHub 当前的 Context Lifecycle 已经不是一个概念 demo，而是一个相对完整的产品闭环：

~~~mermaid
flowchart LR
    SRC[Metadata / Query History / BI / dbt]
    GEN[Context Generation]
    DOC[Generated Context Document]
    PROP[Proposal]
    EVAL[Metadata Evals]
    SME[Data Expert / SME Review]
    PUB[Published Context]
    MCP[MCP / Search / Ask DataHub]
    AG[Agent]
    FB[Feedback / Re-generation]

    SRC --> GEN --> DOC --> PROP --> EVAL --> SME --> PUB
    PUB --> MCP --> AG
    AG --> FB --> GEN
~~~

但要准确理解，它不是“自动生成企业知识”。

更接近：

> **AI-assisted context bootstrapping + governed promotion workflow + controlled activation**

其中最重要的产品边界是：

1. 生成和发布分离；
2. Human review 是正式状态转换；
3. Auto-publish 默认关闭；
4. Published 才是 Agent-visible；
5. Regeneration 当前是 full refresh；
6. Human edits 在 regeneration 中保留并优先；
7. Evals 用来验证 context 对 Agent 行为的影响，而不是只做文档质量检查；
8. Activation 通过 MCP / Ask DataHub / Agent skills 发生。

---

# 1. Availability

截至 2026-09-28：

> DataHub Context Platform is Public Beta.

DataHub Cloud v2.2 把 Context Platform 向所有 DataHub Cloud customers 开放 Public Beta。

Custom Agents 仍是 Private Beta。

所以需要区分：

~~~text
Context lifecycle:
  Public Beta

Custom Agent runtime:
  Private Beta
~~~

Sources:

- https://datahub.com/blog/datahub-cloud-v2-2/
- https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation

---

# 2. Lifecycle Overview

当前产品生命周期可以还原成：

~~~mermaid
stateDiagram-v2
    [*] --> GenerationConfigured
    GenerationConfigured --> Generating: Run
    Generating --> UnpublishedDocument: Generated
    UnpublishedDocument --> Proposal: Routed for review
    Proposal --> Evaluated: Run evals
    Evaluated --> Proposal: Needs edits
    Evaluated --> Published: Publish
    Evaluated --> Rejected: Reject
    Published --> UnpublishedDocument: Unpublish
    Published --> Regenerated: Full refresh
    Regenerated --> Published: Human edits preserved
~~~

注意：

这不是官方明确给出的 formal state machine。

它是根据当前产品文档重建出来的状态流。

---

# 3. Stage 1 — Configuration

Admin 在：

~~~text
Settings -> Context
~~~

配置 Context Generation Job。

当前关键配置：

### Name

generation job identity。

### Scope

必须至少选择：

- Domain；
- 或 Container（database / schema）。

当前文档明确要求 context generation 被 scope 到 domain/container：

- 聚焦有 SME owner 的资产；
- 控制成本；
- 让 review routing 可管理。

### Publishing Settings

默认：

~~~text
Auto-publish documents = disabled
~~~

因此 generated context：

- 不会自动进入 Agent；
- 不会自动进入 search；
- 先等待 Data Expert 验证。

这其实就是一个非常清晰的：

> **default fail-safe publication policy**

---

# 4. 为什么 Scope 是 Lifecycle 的核心，而不只是 Cost Control？

官方说明 scope 主要用于：

- 聚焦有 owner / SME 的资产；
- keep costs manageable。

但从我们的 Reference Architecture 看，还有第三个意义：

> **控制 context publication blast radius。**

如果第一次 context generation 就跑整个 enterprise：

- proposals 会爆炸；
- SME review 无法管理；
- eval coverage 不够；
- wrong context 影响范围更大。

所以 Domain / Container Scope 同时承担：

~~~text
cost boundary
review boundary
authority boundary
blast-radius boundary
~~~

这也是为什么 lifecycle 不能只理解成“AI 自动写文档”。

---

# 5. Stage 2 — Source Evidence

Context Generation 当前明确分析：

- query history；
- schema metadata；
- dbt models；
- downstream Looker charts / dashboards。

v2.2 release 进一步概括为：

> metadata from 150+ sources + semantic meaning from query history。

Town Hall 材料说明 Context Intelligence 特别关注高信号 queries，例如：

- multi-table joins；
- interactive sessions；
- high-frequency usage。

因此当前 Context Generation 的核心证据模型更偏：

> **behavioral / technical evidence -> semantic interpretation**

而不是：

> LLM blank-page authoring。

---

# 6. Query History 是重要输入，但自身也有 Freshness 边界

官方配置文档特别提醒：

某些源的 query history 本身有延迟。

例如 Snowflake ACCOUNT_USAGE 可能落后真实查询一段时间。

这意味着 lifecycle 的真实时间链：

~~~text
User Query
-> Warehouse Telemetry
-> DataHub Ingestion
-> Context Generation
-> Proposal
-> Review
-> Publish
-> Agent Retrieval
~~~

所以：

> Context Publication Time != Source Reality Time.

这是我们前面 temporal model 的现实产品证据。

---

# 7. Stage 3 — Context Generation

Context generation 会处理 scope 内 datasets：

- 分析 query history；
- 读取 schema；
- 结合 dbt；
- 结合 BI context；
- 生成 Context Documents。

官方也称这些 generated context documents 为：

> semantic anchors.

semantic anchor 的目标不是重新复制 schema。

而是补上 Agent 难从 schema 推出的东西：

- business meaning；
- common query patterns；
- relevant joins；
- semantic interpretation。

---

# 8. Context Intelligence 本质上是 Bootstrapping，不是 Authority

DataHub 当前 launch narrative 很明确：

Context Intelligence：

> mines raw signals and bootstraps semantic context.

然后 Context Hub 负责：

> review / validation / publication.

所以产品自身其实已经隐含：

~~~text
Inference
!=
Authority
~~~

这与我们 Phase 1 的设计原则一致。

---

# 9. Stage 4 — Generated Context Document

生成结果不是直接写回普通 description field。

DataHub 有专门的：

> Context Documents

v2.0 已经提供 Context Documents Home：

- 查看 context 来源；
- 看哪些 domain coverage sparse；
- 查看 native / agent-authored / imported documents；
- bulk publish / hide。

所以 Context Document 已经成为 Context Platform 的核心 content primitive。

它不仅是：

> 一个 Markdown 文件。

而是：

> 可以被生成、review、publish、hide、search、Agent consume 的 governed context object。

---

# 10. Native / Agent-authored / Imported Context

v2.0 产品材料明确提到 Context Documents 可以来自：

- native；
- agent-authored；
- imported from Notion；
- Confluence；
- GitHub。

这意味着 Context Hub 不只是“AI-generated documents review UI”。

它正在成为：

> **heterogeneous business knowledge management surface**

不同来源的文档最终统一进入：

~~~text
Document
-> publication lifecycle
-> agent activation
~~~

---

# 11. Stage 5 — Proposal

Data Expert / SME 在 Task Center 中处理 proposal。

当前权限要求：

- MANAGE_DOCUMENTS
- MANAGE_DOCUMENT_PROPOSALS
- MANAGE_EVALS

reviewer 不是普通 consumer。

它是：

> publication authority。

DataHub 当前角色模型把这一 responsibility 明确赋给 Editor / Data Expert / SME。

---

# 12. Proposal 与 Document 的关系

当前文档呈现出来的生命周期更接近：

~~~text
generated document
+ proposed change
-> review
-> published document
~~~

Proposal 是 review workflow object。

Document 是 context object。

这两个不能混淆。

~~~mermaid
flowchart LR
    DOC[Context Document]
    PROP[Proposal]
    REVIEW[Review Workflow]
    PUB[Published Document]

    DOC --> PROP --> REVIEW --> PUB
~~~

这是一个比较成熟的 content governance 模型。

---

# 13. Stage 6 — Metadata Evals

DataHub 当前提供两类明显用途的 eval：

## SQL Generation / Golden Question

验证：

- business question；
- expected SQL / semantic behavior；
- generated context 是否帮助 Agent 得到正确答案。

## Generic Catalog Q&A

验证：

- Agent 是否引用正确 assets；
- must-reference；
- must-not-reference；
- grading guidelines。

这意味着 eval 检查的是：

> **Context 对 Agent behavior 的实际影响。**

而不是只检查：

> 文档语言是否正确。

---

# 14. Evals 的本质：Behavioral Validation

这是 Context Platform 最值得学习的地方之一。

普通 documentation review：

~~~text
Is the document correct?
~~~

DataHub Evals 更接近：

~~~text
If this context becomes active,
does the Agent behave correctly?
~~~

~~~mermaid
flowchart LR
    CTX[Candidate Context]
    AG[Agent]
    Q[Golden Question]
    OUT[Agent Output]
    EVAL[Evaluation]

    CTX --> AG
    Q --> AG
    AG --> OUT --> EVAL
~~~

这使：

> context correctness

开始变成：

> downstream behavior correctness.

---

# 15. Eval 也不是 Truth Oracle

官方 FAQ 明确指出：

一个看起来正确的 context document 也可能因为：

> pass criteria 比预期更严格

而 eval fail。

例如 criteria 指定某张 table，但其实多个 table 都是合法答案。

因此：

~~~text
Eval Fail
!= Context Wrong

Eval Pass
!= Context Authoritative
~~~

Eval 是 evidence。

Authority 仍然来自 reviewer / policy / domain owner。

---

# 16. Stage 7 — Human Review

Reviewer 可以：

- 运行 eval；
- 编辑；
- 留 comment；
- publish；
- reject；
- cancel / 保留 proposal。

Best Practices 明确建议：

1. 先跑 eval；
2. 用 Ask DataHub 测试；
3. 用 comments 协作；
4. incremental publish。

因此 review 本质上是：

> **evidence-assisted authority decision**

而不是：

> 手工 proofread AI 文档。

---

# 17. Test on Ask DataHub 是很重要的 Preview Primitive

当前 reviewer 可以在发布前：

> Test on Ask DataHub

比较 context 生效前后的真实 Agent response。

这是一个很好的设计：

~~~mermaid
flowchart LR
    CAND[Candidate Context]
    TEST[Preview / Ask DataHub]
    BEFORE[Without Context]
    AFTER[With Context]
    REVIEW[Reviewer Decision]

    CAND --> TEST
    TEST --> BEFORE
    TEST --> AFTER
    BEFORE --> REVIEW
    AFTER --> REVIEW
~~~

它把 publish 从：

> static document review

升级成：

> behavioral preview.

---

# 18. Stage 8 — Publish

Publish 后 Context Document 才进入：

- Agent-visible context；
- search；
- Document Library。

官方 Activation 文档明确：

> Only published context documents are visible to agents.

Unpublished document：

- Agent 看不到；
- search 看不到；
- 但并没有被删除。

所以 publication state 是一个真正的：

> activation boundary.

---

# 19. Publish 与 Visibility 是强绑定的

这是 DataHub 当前 lifecycle 最清晰的产品语义之一：

~~~text
Unpublished
-> exists
-> can be reviewed
-> not discoverable by normal Agent/search

Published
-> discoverable
-> Agent-visible
-> activated through MCP/search
~~~

因此我们 Phase 1 中提出的：

> Proposal Plane / Truth Plane

在产品里已经有非常真实的对应。

---

# 20. Unpublish 是重要 Recovery Primitive

官方 FAQ：

> Unpublish 后 document 会从 Agent 和 search 隐藏，但不会删除。

这意味着：

~~~text
publish
<-> unpublish
~~~

比：

~~~text
create
-> irreversible truth
~~~

更安全。

但还要注意：

> 文档 unpublish 不一定意味着 downstream Agent cache / derived context 自动回滚。

官方公开材料没有证明有一个统一 dependency rollback mechanism。

所以：

**Publication rollback: Implemented at document visibility level.  
Dependency rollback: Unclear.**

---

# 21. Stage 9 — Regeneration

当前官方 FAQ 明确：

> Context generation is currently a full refresh.

这非常重要。

它意味着 regeneration 不是：

~~~text
only changed assets
-> incremental recompute
~~~

而更接近：

~~~text
scope
-> regenerate current generated context
~~~

---

# 22. Human Edits Survive Full Refresh

官方明确保证：

> human-edited context is preserved on regeneration.

并且：

> human-edited business metadata takes precedence over agent-generated metadata.

这其实是一个非常明确的 authority rule：

~~~text
Human-reviewed context
>
Agent-generated refresh
~~~

因此 regeneration 不会简单：

> AI 再跑一遍，把 SME 修正覆盖掉。

---

# 23. 这个 Precedence Rule 很关键，但仍然是 Specialized Authority

它很好地处理：

> human vs agent

但我们的 Reference Architecture 还需要更一般的 conflict：

~~~text
Finance human
vs
Growth human

Policy source
vs
business SME

System-of-record
vs
manual override
~~~

当前 Context lifecycle 文档主要证明：

> human-edited > agent-generated.

还不能证明它已经具备通用 typed authority resolution。

---

# 24. Stage 10 — Activation

发布后的 context 可以通过 DataHub MCP 被 Agent 检索。

官方 Activation 文档推荐 Analytics Agent 使用：

> datahub-sql-workflow skill

它会在回答自然语言业务问题时：

> 搜索 DataHub context for grounded truth.

同时可以连接：

- Claude Code；
- 其他 MCP client；
- Snowflake Cortex Agent / Databricks Genie skill（部分 Private Beta）。

所以 lifecycle 最终不是停在“发布文档”。

而是：

~~~text
Published Context
-> Agent Retrieval
-> Agent Behavior
~~~

---

# 25. Activation 需要再次验证

官方明确建议发布后：

- 用 Test on Ask DataHub；
- 跑 metadata eval；
- 从少量高置信 documents 开始；
- 一个 domain 先 rollout。

这说明 DataHub 把 lifecycle 看成：

~~~text
generate
-> validate
-> publish
-> validate again in consumption
~~~

而不是一次性 publish。

---

# 26. v2.2 加强了 Eval Feedback Loop

DataHub Cloud v2.2 重新设计 Evals：

现在除了 pass/fail，还可以：

> trace failing eval to the exact context document that caused it.

然后：

- 修改 document；
- 重跑；
- 验证 fix。

这意味着 lifecycle 开始形成：

~~~mermaid
flowchart LR
    PUB[Published Context]
    AG[Agent]
    FAIL[Eval Failure]
    DOC[Responsible Document]
    FIX[Edit Context]
    REPUB[Republish]

    PUB --> AG --> FAIL --> DOC --> FIX --> REPUB --> PUB
~~~

这是 Context Reliability 非常关键的一步。

---

# 27. Context Curator Agent 把 Lifecycle 自动化

v2.2 内置 Context Curator Agent：

- 从 warehouse query logs；
- BI dashboards；

持续生成 semantic context；

- 随数据变化保持更新；
- 将 proposals 路由到正确 domain experts。

这使：

~~~text
Context Generation Job
~~~

正在进一步变成：

~~~text
Agent-operated Context Maintenance Workflow
~~~

---

# 28. 当前 Lifecycle 的真正 State Boundary

我们可以把当前产品压缩成三个 state planes：

~~~mermaid
flowchart TB
    subgraph GEN[Generation Plane]
        SRC[Evidence]
        AI[Context Intelligence]
        DOC[Generated Document]
    end

    subgraph GOV[Governance Plane]
        PROP[Proposal]
        EVAL[Eval]
        SME[SME]
        PUB[Published]
    end

    subgraph ACT[Activation Plane]
        SEARCH[Search]
        MCP[MCP]
        ASK[Ask DataHub]
        AG[External Agent]
    end

    SRC --> AI --> DOC --> PROP --> EVAL --> SME --> PUB
    PUB --> SEARCH
    PUB --> MCP --> AG
    PUB --> ASK
~~~

这基本对应：

> Generate -> Govern -> Activate.

---

# 29. Lifecycle 的核心 Strengths

## 1. Publication Boundary

AI output 不自动等于 shared truth。

## 2. SME Authority

明确 Human-in-the-loop role。

## 3. Behavior-based Eval

context correctness 通过 Agent outcome 验证。

## 4. Incremental Rollout

官方建议小范围 publish。

## 5. Human Edit Preservation

full refresh 不覆盖 SME edits。

## 6. Unpublish

可以从 Agent/search 移除 context，而不删除。

## 7. Activation Separation

只有 publish 后才进入 MCP/search。

这些已经构成一个很完整的 Context Governance lifecycle。

---

# 30. Lifecycle 当前明显的 Limits

## A. Full Refresh

目前 Context Generation 是 full refresh。

这意味着：

- 计算成本；
- re-evaluation scope；
- impact granularity；

都还不是真正 dependency-aware incremental rebuild。

## B. Document-centric

当前主要 lifecycle object 是 Context Document。

Reference Architecture 更希望：

> assertion-level lifecycle.

## C. Human > Agent Rule 仍较粗

没有证明存在通用 typed authority resolution。

## D. Publication Rollback 主要是 Visibility

没有看到明确 generalized downstream dependency invalidation。

## E. Eval Coverage 是 Configuration Responsibility

如果没有好 eval：

> proposal lifecycle 仍可能 publish wrong context.

## F. Source Freshness Chain 仍需外部理解

当前 UI/job status 不等于 end-to-end context freshness SLO。

---

# 31. Lifecycle 与我们的 Reference Architecture Mapping

| Reference Primitive | DataHub Current Lifecycle |
|---|---|
| Observation | Ingestion / query history / schema / BI |
| Candidate Context | Generated Context Document |
| Proposal | Document Proposal |
| Evidence Gate | Metadata Eval |
| Human Authority | Data Expert / SME |
| Publication State | Published / Unpublished |
| Activation | MCP / Search / Ask DataHub |
| Rollback | Unpublish |
| Re-generation | Full refresh |
| Authority precedence | Human edits preserved over AI-generated |
| Context SLO | Partial / implicit |
| Typed conflict resolution | Not evidenced |
| Dependency invalidation | Partial / unclear |
| Assertion lifecycle | Document-centric / partial |

---

# 32. DataHub Context Lifecycle 的产品哲学

这套 lifecycle 背后的哲学其实非常清楚：

> **AI can bootstrap semantic context faster than humans can author it, but organizational authority still needs a governed promotion step.**

换句话说：

~~~text
AI = high-throughput context producer
Human = authority / validator
Evals = behavioral evidence
Context Hub = promotion boundary
MCP = activation channel
~~~

这是 DataHub Context Platform 当前最完整、也最有区分度的一段产品设计。

---

# 33. 一个重要问题：Context Document 是否会成为过粗的治理粒度？

如果一个 document 同时包含：

- metric meaning；
- join guidance；
- owner；
- business exception；
- query pattern；

其中只有一个部分 stale。

当前 publish/unpublish 的粒度如果主要是 Document：

> 可能需要整篇下线。

Reference Architecture 更细：

~~~text
Document
-> Assertions
-> each assertion has authority / validity / evidence
~~~

所以未来值得观察：

> DataHub 会不会从 Document-centric Context 进一步演进到 Assertion-centric Context？

当前没有官方证据说明会这么做。

这是我们的架构观察点。

---

# 34. 另一个重要问题：Full Refresh 与 Continuous Context 的张力

DataHub 产品叙事强调：

> continuously keeps context current.

但当前文档又明确：

> context generation is currently a full refresh.

两者并不矛盾。

“continuous”可以意味着：

> repeat full regeneration on schedule/event.

但从系统工程角度：

~~~text
continuous
!= incremental
~~~

真正 mature Context Engine 可能最终需要：

~~~text
specific source change
-> specific dependent assertions
-> targeted recompute / review
~~~

这正是我们 Reference Architecture 的 generalized invalidation engine。

---

# 35. Product Verdict

当前 Context Lifecycle 可以评价为：

## Product completeness

**High for document-level governed context lifecycle.**

已经具备：

- generation；
- proposal；
- eval；
- human review；
- publication；
- unpublish；
- activation；
- regeneration；
- human precedence；
- post-publish validation。

## Architectural generality

**Medium / evolving.**

因为目前主要仍是：

> Context Document lifecycle

而不是：

> generalized assertion / dependency / temporal lifecycle.

所以最准确的定位：

> **DataHub 已经实现了一个成熟雏形的 governed Context Document lifecycle；它非常接近我们 Reference Architecture 的 Proposal/Publication/Activation 链，但还没有证据表明已经抽象成通用 assertion-level epistemic lifecycle。**

---

# 36. 下一篇

## 04 — Agent Governance Product Mapping

将沿当前 DataHub Agent 产品链：

~~~mermaid
flowchart LR
    REG[Agent Registry]
    VIEW[Scoped View]
    INST[Instructions]
    TOOL[Tools / MCP Plugins]
    TASK[Tasks]
    DEC[Decisions]
    WRITE[Metadata Mutation]
    AUDIT[Audit / Version / Lineage]

    REG --> VIEW --> INST --> TOOL --> TASK --> DEC --> WRITE --> AUDIT
~~~

重点回答：

- Agent 在 DataHub 里到底是什么 Entity？
- skills / tools 怎么进入 graph？
- View scope 实际控制什么？
- Task / Decision 如何治理？
- Agent mutation 如何与 Context proposal 区分？
- Audit 到底能还原到什么程度？

---

# Sources

- DataHub Docs, Configure Context Generation  
  https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation

- DataHub Docs, Validate Context Proposals  
  https://docs.datahub.com/docs/managed-datahub/context/review-context-proposals

- DataHub Docs, Activate Context  
  https://docs.datahub.com/docs/managed-datahub/context/activate-context

- DataHub, Introducing DataHub Cloud v2.2  
  https://datahub.com/blog/datahub-cloud-v2-2/

- DataHub, Context to Action: May 2026 Town Hall Highlights  
  https://datahub.com/blog/datahub-may-2026-town-hall-highlights-context-to-action/

- DataHub, Introducing DataHub Context Platform for Analytics Agents  
  https://datahub.com/blog/announcing-datahub-context-platform/

- DataHub, Introducing DataHub Cloud 2.0  
  https://datahub.com/blog/datahub-cloud-2-0/
