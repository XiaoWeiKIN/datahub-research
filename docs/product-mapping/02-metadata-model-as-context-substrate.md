# 02 — Metadata Model as Context Substrate

## 为什么 Entity / Aspect / URN / Relationship / Timeseries 能承载 Context Platform？又在哪里开始吃力？

**Snapshot:** 2026-09-28  
**Scope:** DataHub Core metadata model as the substrate beneath Context Platform

---

# 0. 当前结论

DataHub 的 Context Platform 并不是建立在一个全新的 AI-specific data model 上。

它实际上大量复用了 DataHub 已有的 metadata primitives：

~~~mermaid
flowchart LR
    URN[URN / Key]
    ENTITY[Entity]
    ASPECT[Aspect]
    REL[Relationship]
    VER[Versioned Aspect]
    TS[Timeseries Aspect]
    MCP[MCP]
    MCL[MCL]

    URN --> ENTITY
    ENTITY --> ASPECT
    ASPECT --> REL
    ASPECT --> VER
    ASPECT --> TS
    MCP --> ASPECT
    ASPECT --> MCL
~~~

这套模型非常适合 Context Platform 的原因是：

1. **稳定身份**：URN 把 enterprise objects 变成可引用实体；
2. **可组合 facets**：Aspect 允许同一 entity 独立演进 ownership、schema、tags、docs、quality 等维度；
3. **关系原生化**：@Relationship 把引用直接变成可双向遍历的 graph edge；
4. **状态历史**：Versioned Aspect 能保留旧版本；
5. **运行时信号**：Timeseries Aspect 能表达 profile / usage / quality 等时间序列事实；
6. **事件驱动变化**：MCP / MCL 把 metadata mutation 变成事件流；
7. **schema-first**：storage / indexing / serving / ingestion 都直接建立在 metadata schema 上。

这解释了为什么 DataHub 可以从 Metadata Platform 扩展到 Context Platform，而不用先重建一整套 graph / identity / event infrastructure。

但它也有一个核心限制：

> **DataHub metadata model 的中心原语仍然是“Entity 的某个 Aspect 当前是什么”；AI-era Context Layer 更自然的原语则是“某条带 scope、authority、evidence、valid-time 的 Assertion 是否成立”。**

这两者有很大重叠，但不是同一个抽象。

---

# 1. DataHub Metadata Model 的真正中心是什么？

DataHub 官方当前文档明确说它采用：

> schema-first metadata modeling

并且 storage、serving、indexing、ingestion 都直接构建在 metadata model 上。

核心抽象：

~~~text
Entity
= type
+ unique identifier (URN)
+ aspects
~~~

而 Aspect 是：

> 描述 Entity 某一个 facet 的一组 attributes，也是 DataHub 最小的原子写入单元。

因此 DataHub 不是：

~~~text
document store + search index
~~~

而是：

~~~text
typed entity graph + independently mutable facets
~~~

Source:

- https://docs.datahub.com/docs/metadata-modeling/metadata-model

---

# 2. 为什么 Entity + Aspect 是一个很强的长期模型？

考虑一个 Dataset。

它可能同时拥有：

~~~text
Dataset
├── DatasetProperties
├── SchemaMetadata
├── Ownership
├── GlobalTags
├── GlossaryTerms
├── UpstreamLineage
├── Status
├── InstitutionalMemory / Documents
├── Usage
└── Quality / Assertions
~~~

这些 facet：

- 生命周期不同；
- 更新频率不同；
- 来源不同；
- owner 不同。

如果整个 entity 是一个 monolithic JSON document：

> 修改 ownership 很容易与 schema / description / lineage 写入产生耦合。

Aspect model 把它拆成多个 atomic write units。

~~~mermaid
flowchart TB
    D[Dataset Entity]

    D --> S[Schema Aspect]
    D --> O[Ownership Aspect]
    D --> T[Tags Aspect]
    D --> L[Lineage Aspect]
    D --> DOC[Documentation Aspect]
    D --> Q[Quality Aspect]
~~~

这对 Context Platform 极其重要，因为：

> enterprise context 本来就不是一个统一刷新周期的数据对象。

---

# 3. Aspect 是 DataHub 从 Catalog 走向 Context 的关键 primitive

传统 catalog schema 常常把：

~~~text
name
description
owner
tags
columns
~~~

当成固定 asset record 的字段。

DataHub 的 Aspect model 更接近：

> 一个 Entity 可以持续挂载新的 context facets。

因此当产品从 metadata 扩展到：

- semantic models；
- Agent entities；
- skills；
- tools；
- Context Documents；
- quality；
- incidents；

底层不需要改变“什么是 Entity”。

只需要：

- 新 entity；
- 新 aspect；
- 新 relationships。

这是一种相当稳定的 extensibility model。

---

# 4. URN：Context Graph 的 Reference Integrity 基础

Key Aspect 用于唯一识别 Entity，并序列化成 URN。

例如：

~~~text
urn:li:dataset:(urn:li:dataPlatform:snowflake,finance.orders,PROD)
~~~

URN 的价值不是字符串格式本身。

而是：

> Context 中任何对象都可以稳定引用另一个对象，而不需要复制完整对象。

这让：

~~~text
Owner
Metric
Dataset
Dashboard
Agent
Skill
Glossary Term
Domain
~~~

都可以进入同一个 reference graph。

DataHub 还支持 @UrnValidation，对 URN field：

- 检查 existence；
- 做 strict validation；
- 限制 entity type。

这使 metadata model 已经具备一定 referential-integrity 思想。

---

# 5. URN 的优势：Machine-friendly Identity

Agent era 里 stable identity 特别重要。

如果 Agent 只拿：

~~~text
"orders"
~~~

它可能无法区分：

- prod vs dev；
- Snowflake vs BigQuery；
- logical vs physical；
- database/schema ambiguity。

URN 把 identity 结构化。

所以 DataHub 的一个长期优势是：

> **它从一开始就不是靠 display name 组织世界。**

这是 Context Graph 必须具备的能力。

---

# 6. 但 URN 解决的是 Identity，不是 Entity Resolution

这一点必须区分。

URN 可以唯一识别：

~~~text
Snowflake / finance.orders / PROD
~~~

但不能自动回答：

> BigQuery 的 `finance.orders` 是否与 Snowflake 的这个表代表同一个 logical concept？

DataHub Logical Models 开始解决一部分问题：

~~~mermaid
flowchart TB
    L[Logical Users Model]
    S[Snowflake Users]
    B[BigQuery AllUsers]
    H[Hive UsersAttributes]

    S -->|PhysicalInstanceOf| L
    B -->|PhysicalInstanceOf| L
    H -->|PhysicalInstanceOf| L
~~~

官方当前文档说明 logical model 是一个不绑定具体物理实例的 dataset concept，并允许多个 physical dataset 链接到同一 logical parent。

这是很重要的 Context-era evolution。

但它仍然是：

> 明确建模的 logical relationship

而不是通用 entity resolution engine。

Source:

- https://docs.datahub.com/docs/features/feature-guides/logical-models/overview

---

# 7. Relationship：DataHub 真正的 Graph Primitive

DataHub 中 relationship 不是 UI 层推断出来的。

@Relationship annotation 会在 Aspect field 引用另一个 URN 时创建 graph edge，并可双向遍历。

例如 Ownership：

~~~text
Dataset --OwnedBy--> CorpUser
~~~

或 Dashboard：

~~~text
Dashboard --Contains--> Chart
~~~

这意味着 graph edge 的源头仍然是：

> typed aspect field。

这是一个很实用的模型：

~~~mermaid
flowchart LR
    A[Typed Aspect Field]
    U[Referenced URN]
    E[Graph Edge]

    A --> U --> E
~~~

它同时保证：

- source entity 的 metadata schema；
- destination entity identity；
- relationship type；
- graph traversal；

保持一致。

---

# 8. 这个模型为什么适合 Context Graph？

因为 enterprise context 大量存在于关系中：

~~~text
metric -> dataset
dataset -> owner
dataset -> upstream
metric -> glossary term
agent -> dataset
agent -> tool
document -> asset
asset -> domain
~~~

DataHub 不需要后来增加一个完全独立的 graph store abstraction。

Aspect 中的 reference 已经能 materialize graph relationship。

所以：

> **Context Graph 对 DataHub 来说更像 Metadata Graph 的 scope expansion，而不是 architecture replacement。**

---

# 9. Relationship Model 的限制：Edge Semantics 不够“Context-aware”

Reference Architecture 中，我们希望 dependency edge 可以表达：

~~~text
depends_on
supports
derived_from
governed_by
authoritative_for
invalidates_when
~~~

并进一步表达：

> 哪种 upstream change 会让 downstream 哪个 assertion stale。

DataHub 的 @Relationship 可以很好表达：

~~~text
A --relationship_type--> B
~~~

但从公开 metadata model 文档看，它并没有天然提供一个统一 edge-level：

~~~text
invalidation_semantics
authority_semantics
valid_time
confidence
evidence_weight
~~~

模型。

这些语义通常需要：

- 特定 Aspect schema；
- 特定 Entity；
- 上层业务逻辑；

分别实现。

因此：

**Graph structure: strong.  
Epistemic edge semantics: domain-specific / partial.**

---

# 10. Versioned Aspect：非常重要，但它是 System Time History

DataHub Versioned Aspect：

- 每次 field change 生成新 version；
- backend 保存历史；
- latest version 通常是 version 0 查询入口；
- ownership / tags / glossary terms / description 等大量 metadata 使用此模式。

这给 Context Platform 一个非常好的基础：

> **metadata state 不是覆盖后消失，而可以保留历史版本。**

~~~mermaid
flowchart LR
    V3[Aspect v3]
    V2[Aspect v2]
    V1[Aspect v1]
    V0[Latest]

    V3 --> V2 --> V1 --> V0
~~~

不过严格来说，这更接近：

> system version history

而不是：

> real-world temporal validity。

---

# 11. Version Time != Valid Time

例如 Finance 在 9 月 1 日更改 policy，但 9 月 5 日才录入 DataHub。

Aspect history 可以告诉我们：

~~~text
DataHub 在 Sep 5 收到新版本
~~~

却未必天然表达：

~~~text
这条 policy 在现实世界从 Sep 1 开始有效
~~~

我们的 Reference Architecture 区分：

~~~text
valid_time
observed_time
system_time
published_time
invalidated_time
~~~

DataHub versioned Aspect 并没有自动等同于这套 temporal semantics。

所以：

> **Version history 是 Context temporal model 的重要基础，但不是完整 bitemporal truth model。**

---

# 12. Timeseries Aspect：DataHub 很早就区分了 State 与 Event-like Metadata

DataHub metadata model 明确区分：

## Versioned Aspect

表达：

> Entity 当前某个 facet 的 latest state + versions。

例如：

- ownership；
- tags；
- glossary terms；
- descriptions。

## Timeseries Aspect

表达：

> 随时间持续产生的 observations / events。

例如：

- dataset profile；
- usage statistics；
- daily quality results。

Timeseries Aspect：

- 每条有 timestamp；
- 可以按 time range 查询；
- 可被搜索 / relationship annotation；
- 存储路径与普通 versioned aspects 不同。

~~~mermaid
flowchart TB
    ENTITY[Dataset]

    ENTITY --> STATE[Versioned State<br/>Owner / Tags / Docs]
    ENTITY --> TS[Timeseries Observations<br/>Usage / Profiles / Quality]
~~~

这其实已经非常接近我们 Context Layer 中：

> declared state vs observed signals

的区分。

Source:

- https://docs.datahub.com/docs/metadata-modeling/metadata-model

---

# 13. Timeseries Model 对 AI Context 很有价值

Agent 不应该只知道：

~~~text
owner = Finance
~~~

还可能需要：

~~~text
usage trend
quality trend
profile changes
recent queries
incident history
~~~

Timeseries Aspect 给 DataHub 一个天然位置存这些 temporal signals。

它解释了为什么 DataHub 能自然把：

- operational context；
- usage；
- data quality；

连接到静态 metadata graph。

---

# 14. 但 Timeseries Aspect 不是 General Temporal Assertion

Timeseries aspect 是：

> 某一类 Aspect 的 timestamped records。

我们 Reference Architecture 的 assertion temporal model更广：

~~~text
Assertion:
  Revenue definition authoritative for Board Reporting
  valid_from: ...
  valid_until: ...
~~~

它不是“每天采样一次”的 signal。

也不是 ownership aspect 的历史版本就能完全表达。

所以：

> DataHub 已有两套时间机制——Versioned State + Timeseries Observation；  
> 但 AI-era Context 还可能需要第三类：Scoped Temporal Assertion。

这是当前 metadata model 与我们的目标模型之间最值得关注的差距之一。

---

# 15. MCP：Aspect-level Mutation 是一个非常优秀的原子操作

Metadata Change Proposal（MCP）表达：

> 请求改变某一个 Entity 的某一个 Aspect。

核心字段：

~~~text
entityUrn
entityType
changeType
aspectName
aspect
systemMetadata
~~~

支持：

- CREATE；
- UPSERT；
- DELETE；
- 部分 PATCH。

这个模型和 Aspect atomicity 完全一致。

~~~mermaid
flowchart LR
    CLIENT[Producer]
    MCP[Metadata Change Proposal]
    ASPECT[Entity Aspect]
    GRAPH[Metadata Graph]

    CLIENT --> MCP --> ASPECT --> GRAPH
~~~

这使 connector / SDK / Agent 不需要发送整个 entity snapshot。

只提交自己负责的 facet。

---

# 16. MCP 的设计哲学：Multiple Producers Can Own Different Facets

假设：

- Snowflake connector 写 schema；
- HR integration 写 owner mapping；
- dbt 写 lineage；
- human 写 description；
- Agent propose glossary mapping。

Aspect-level writes 让这些 producer 可以在一定程度上并存。

这是 Context Platform 特别需要的。

因为 Context 的来源天然是 federated 的。

如果所有 producer 都需要覆写整个 Entity：

> 最后写入者会不断破坏其他来源维护的 context。

Aspect atomicity 减少了这个冲突面。

---

# 17. 但 Aspect-level Ownership 不等于 Assertion-level Authority

例如 Ownership Aspect 可能包含多个 owners。

Agent 可能更新一个 owner。

Context Platform 真正还想知道：

- 这个 owner 是 HR source 声明的？
- Data steward 人工编辑的？
- Agent 推断的？
- 哪个更 authoritative？
- 哪个只在某 domain 有效？

MCP 有 systemMetadata，可以记录：

- lastObserved；
- runId；
- model registry；
- properties。

MCL 也记录：

- previous value；
- previous system metadata；
- actor；
- timestamp。

这些是很好的 provenance building blocks。

但仍不等于一个统一 authority calculus。

---

# 18. MCL：DataHub 真正 Event-driven 的关键

Metadata Change Log（MCL）代表：

> 已经成功提交到 Metadata Graph 的变化。

对 versioned 与 timeseries aspect 有不同 event streams。

MCL 包含：

- entity；
- aspect；
- current value；
- previous value；
- systemMetadata；
- previousSystemMetadata；
- actor；
- created time。

这非常重要。

因为 DataHub 可以：

~~~text
proposal
-> committed state
-> change log
-> index / graph update
-> derived events / automation
~~~

所以它不是一个“被动 catalog DB”。

它有完整的 metadata change stream。

Source:

- https://docs.datahub.com/docs/what/mxe

---

# 19. MCL 是 Context Reconciliation Engine 的天然输入

我们 Reference Architecture 需要：

~~~text
source change
-> impacted context
-> invalidate / recompute / review
~~~

MCL 已经提供了：

~~~text
what changed
old value
new value
who
when
~~~

所以理论上：

~~~mermaid
flowchart LR
    MCL[Metadata Change Log]
    DEP[Dependency Graph]
    IMPACT[Impacted Context]
    ACTION[Invalidate / Recompute / Review]

    MCL --> DEP --> IMPACT --> ACTION
~~~

完全符合 DataHub 的事件模型。

这也是为什么我们说：

> DataHub 已经有 reconciliation substrate。

真正缺少证据的是：

> 是否已经存在一个 generalized Context-level invalidation engine。

---

# 20. Platform Event：从 Storage Change 到 Semantic Change

DataHub 除了 MCL，还有 Platform Event。

其中 Entity Change Event 会把一些变化表达成：

~~~text
OWNER ADD
TAG REMOVE
DEPRECATION CHANGE
...
~~~

也就是说：

> MCL 更接近 metadata storage/event model；  
> Platform Event 更接近 semantic business event。

这是很值得注意的分层。

~~~mermaid
flowchart LR
    MCP[Requested Aspect Change]
    MCL[Committed Metadata Change]
    PE[Semantic Platform Event]
    ACTION[Automation]

    MCP --> MCL --> PE --> ACTION
~~~

Context Platform 的未来 reconciliation / policy workflow 可以自然建立在这类 semantic event 上。

---

# 21. Schema-first Model 的强项：Strong Types Across the Stack

DataHub 当前 metadata model 文档明确强调：

> strong types 从 client 一直到 storage layer。

Aspect schema 可以同时驱动：

- validation；
- ingestion；
- storage；
- indexing；
- search；
- relationships；
- SDK types。

这避免 Context Platform 退化成：

~~~text
arbitrary JSON blob graph
~~~

对 Agent 来说也很重要。

因为 machine-readable context 的可靠性很大程度来自：

> context schema 是明确的。

---

# 22. Schema-first 的代价：新 Epistemic Semantics 需要显式建模

如果想增加：

~~~text
authority
validity scope
evidence
epistemic state
conflict state
publication state
~~~

schema-first system 不能只是“解释一下 existing text”。

它需要：

- Aspect schema；
- Entity；
- relationship；
- system logic；
- retrieval logic。

这使 DataHub 模型：

> 更可靠，但演进更显式。

对于 production infrastructure，这是合理 tradeoff。

但也意味着：

> AI-era Context Layer 的新抽象不会仅靠 LLM 或 embeddings 自然出现，必须进入 metadata standard。

---

# 23. Custom Model Extension：Context Platform 的可扩展性基础

DataHub metadata model 可以：

- 新增 Entity；
- 新增 Aspect；
- attach Aspect 到 existing Entity；
- 自定义 @Searchable；
- 自定义 @Relationship；
- 定义 timeseries aspects。

当前文档也强调希望更多 model extension 走 no-code / low-code / custom repository 路径，而不是长期维护 DataHub fork。

这意味着：

> 企业可以扩展自己的 context ontology。

不过需要注意：

> extension ability != semantic interoperability。

每家公司自定义不同 entity/aspect，也可能形成新的 context fragmentation。

所以未来开放 metadata / semantic standards仍然重要。

---

# 24. Structured Properties：不改 Core Schema 的轻量扩展

DataHub 还有 Structured Properties / custom properties 类能力，适合：

- custom business attributes；
- additional metadata；
- organization-specific fields。

这提供了比 fork metadata model 更轻的扩展路径。

但如果一个 field 需要：

- 强 relationship semantics；
- dedicated lifecycle；
- temporal model；
- special authority；
- complex validation；

最终还是更适合专门 Aspect / Entity。

也就是说：

~~~text
custom property
!=
full context primitive
~~~

---

# 25. Logical Models：DataHub 开始从 Physical Identity 走向 Conceptual Identity

Logical Model 是这套 metadata substrate 在 Context 时代一个很有代表性的扩展。

它不是实际数据表。

而是：

> 不绑定单一物理实现的 table concept。

例如：

~~~mermaid
flowchart TB
    L[Logical Users]
    S[Snowflake Users]
    B[BigQuery Users]
    H[Hive Users]

    S --> L
    B --> L
    H --> L
~~~

这说明 DataHub 已经不再只 catalog“现实系统中实际存在的 object”。

它开始 catalog：

> conceptual data model.

这很接近 Context Layer 对 semantic abstraction 的需求。

---

# 26. Logical Model 也暴露了 Identity Model 的边界

官方当前文档明确：

- 目前 logical models 只支持 datasets / schema fields；
- logical model 实际仍被表示为一个 logical platform 上的 Dataset；
- breaking column changes 会拆掉受影响 column mappings；
- 没有 automatic relinking。

这说明 DataHub 的 abstract concept modeling 仍然建立在 existing Entity system 上进行复用。

优点：

- implementation reuse；
- UI/API reuse；
- ownership/tag/docs reuse。

代价：

> conceptual entity 有时需要借用已有 physical-oriented type。

这是一个值得长期观察的建模压力。

---

# 27. Metadata Model 为什么如此适合 DataHub 的 Context Platform 路线？

综合起来：

## 1. Stable references

URN 让 Context Graph 可连接。

## 2. Faceted mutation

Aspect 让多个 source 独立维护不同 context。

## 3. Graph-native references

@Relationship 让 context 成为 graph，不是 flat fields。

## 4. History

Versioned Aspect 保存 metadata state evolution。

## 5. Operational signals

Timeseries Aspect 容纳 high-frequency context。

## 6. Event stream

MCP / MCL / Platform Event 支持 continuous metadata。

## 7. Strong schema

Agent consumption 可以基于 typed metadata，而不是 arbitrary text。

## 8. Extensibility

Context scope 可以持续扩展。

这八点基本构成 DataHub 能走向 Context Platform 的底层技术理由。

---

# 28. Metadata Model 在 AI Context 时代开始吃力的地方

我们可以非常具体地列出。

## A. Aspect-centric vs Assertion-centric

DataHub：

~~~text
Entity -> Aspect -> Fields
~~~

Reference Context Model：

~~~text
Assertion
-> subject
-> predicate
-> object
-> scope
-> authority
-> evidence
-> temporal validity
~~~

Aspect 更适合：

> 描述一个 Entity 的一个 facet。

Assertion 更适合：

> 表达一个可冲突、可限定范围、可追溯的知识声明。

---

# 29. B. Version History vs Bitemporal Truth

DataHub 已有：

- aspect version；
- timeseries timestamp；
- systemMetadata lastObserved；
- created actor/time。

但我们仍需要区分：

~~~text
when reality changed
when source observed
when DataHub ingested
when context derived
when human validated
when published
when invalidated
~~~

现有 primitive 可以承载其中一部分。

但不是一个统一 temporal contract。

---

# 30. C. Relationships vs Epistemic Relationships

现有 graph edge 可以很好表达：

~~~text
OwnedBy
Contains
UpstreamOf
PhysicalInstanceOf
~~~

AI-era Context 可能需要：

~~~text
SupportsEvidenceFor
AuthoritativeFor
Contradicts
Supersedes
DerivedFrom
ApplicableIn
InvalidatedBy
~~~

这些当然可以继续增加 relationship types / entities。

但问题会变成：

> 是否应该继续把所有 epistemic semantics 都编码成 domain-specific Aspects？

还是需要一个更通用 Context Assertion layer？

这是架构演进的重要问题。

---

# 31. D. Producer Provenance vs Knowledge Provenance

MCP / MCL system metadata 很擅长回答：

> 哪次 ingestion / 哪个 actor 写了这个 Aspect？

但 AI context 还要问：

> 这条自然语言定义基于哪些 evidence？
> 中间经过哪些 Agent summarization？
> 哪个 SME 批准？
> 哪些 assertion 支撑它？

也就是：

~~~text
write provenance
!=
knowledge provenance
~~~

DataHub 已经有很多 building blocks，但后者需要更高层 model。

---

# 32. E. Entity Ownership vs Typed Authority

Ownership 很适合：

> 谁负责这个 asset？

但 Context authority 更复杂：

~~~text
Finance:
  authority over metric meaning

Warehouse:
  authority over schema

Security:
  authority over classification

Data Expert:
  publication authority

Policy Engine:
  execution authority
~~~

这不是一个简单 Owner list 能完全表达的。

所以：

> ownership 是 authority 的输入，不是完整 authority model。

---

# 33. F. Search Ranking vs Epistemic Retrieval

DataHub search architecture已经能：

- 索引不同 aspects；
- consolidate entity-level fields；
- search tiers；
- exact/filter/fulltext；
- semantic search。

但 Context retrieval 还需要：

~~~text
relevance
+ authority
+ scope
+ freshness
+ conflict
+ publication state
+ evidence
~~~

所以：

> Search index infrastructure 是必要条件，但 Context Retrieval 是更高层 reasoning contract。

---

# 34. 一个非常重要的观察：DataHub 其实已经有“三种真相”

把 metadata primitives重新分类：

## Current State

Versioned Aspect latest version。

## Historical State

Older aspect versions。

## Observed Time-series

Timeseries Aspect records。

~~~mermaid
flowchart TB
    E[Entity]

    E --> C[Current State]
    E --> H[Historical Versions]
    E --> O[Observed Time-series]
~~~

AI Context 还会需要第四种：

## Scoped Assertion

~~~text
某命题
在某 domain / purpose / time
由某 authority
基于某 evidence
成立
~~~

所以 DataHub 的下一个模型演进问题可以表达成：

> **是否需要让 Assertion 本身成为 first-class entity / aspect primitive？**

这不是当前产品事实，而是我们的架构问题。

---

# 35. 为什么不应该轻易抛弃 Aspect Model？

即使 Assertion 更适合某些 AI context，也不意味着应该改成“所有 metadata 都是 triples”。

Aspect 模型有非常现实的优势：

- strong typing；
- atomic writes；
- schema validation；
- efficient materialization；
- API ergonomics；
- domain-specific structure；
- predictable search indexing。

如果全部变成：

~~~text
subject-predicate-object
~~~

很可能失去：

- application-friendly schemas；
- validation；
- compact state retrieval；
- stable SDK types。

因此更合理的未来模型可能是：

~~~mermaid
flowchart TB
    ENTITY[Entity]
    ASPECT[Typed Aspects]
    ASSERT[Epistemic Assertions]
    REL[Graph Relationships]

    ENTITY --> ASPECT
    ENTITY --> ASSERT
    ASPECT --> REL
    ASSERT --> REL
~~~

也就是：

> Aspect 与 Assertion 并存。

而不是一个取代另一个。

---

# 36. 一个可能的 Hybrid Model

对于确定性、结构化 metadata：

~~~text
SchemaMetadata
Ownership
DatasetProperties
SemanticModel
~~~

继续用 Aspect。

对于需要：

- scope；
- competing truth；
- authority；
- evidence；
- valid time；
- publication state；

的 context：

~~~text
Business Definition
Approved Interpretation
Context Claim
Policy Interpretation
Agent-generated Hypothesis
~~~

可以用 Assertion / Document + Evidence model。

这样：

~~~mermaid
flowchart LR
    SOURCE[Structured Sources]
    ASPECT[Typed Aspect]
    GRAPH[Context Graph]

    KNOW[Knowledge / Agent]
    ASSERT[Scoped Assertion]

    SOURCE --> ASPECT --> GRAPH
    KNOW --> ASSERT --> GRAPH
~~~

这与 DataHub 现在“technical metadata + Context Documents”的方向其实已经有一点相似，只是还没有被官方抽象成统一 assertion primitive。

---

# 37. Product Mapping Verdict

| Primitive | Why It Works for Context | AI-era Limitation |
|---|---|---|
| Entity | 稳定 object identity | conceptual/entity resolution 仍需上层机制 |
| URN / Key | machine-readable reference | 不解决 semantic equivalence |
| Aspect | typed faceted context + atomic writes | 不天然表达 scoped competing assertions |
| @Relationship | native graph edges | edge epistemic semantics需专门建模 |
| Versioned Aspect | current + history | 不等于 real-world valid time |
| Timeseries Aspect | operational observations | 不等于 arbitrary temporal assertion |
| MCP | facet-level federated mutation | mutation authority需上层治理 |
| MCL | committed change stream | 不自动做 context invalidation |
| Platform Event | semantic change trigger | event taxonomy仍是业务特定 |
| Logical Model | conceptual abstraction | 当前主要复用 Dataset type，范围有限 |
| Custom Model | 可扩 context ontology | interoperability / fragmentation 风险 |

---

# 38. 当前判断：DataHub Metadata Model 是 Context Platform 的“好底座”，不是最终模型

如果没有这套 metadata standard，DataHub 2026 再做 Context Platform，很可能需要重建：

- identity；
- graph；
- eventing；
- versioning；
- search；
- APIs；
- extensibility。

它没有重建。

而是在上面增加：

- Context Documents；
- Context Intelligence；
- proposal lifecycle；
- semantic entities；
- Agents；
- MCP。

这证明旧模型的长期质量很高。

但 AI-era context 又提出更高要求：

> assertion-level authority / scope / validity / evidence / conflict。

这些不完全落在传统 Entity + Aspect 世界里。

所以我们最终判断：

> **DataHub Metadata Model 是一个优秀的 Context Substrate，但“Context Substrate”与“完整 Epistemic Model”不是同一回事。**

---

# 39. 下一篇

## 03 — Context Lifecycle Product Mapping

下一步直接跟一条真实产品链路：

~~~mermaid
flowchart LR
    Q[Query History / Metadata]
    GEN[Context Generation]
    PROP[Proposal]
    EVAL[Eval]
    SME[SME Review]
    PUB[Publish]
    MCP[MCP]
    AG[Agent]

    Q --> GEN --> PROP --> EVAL --> SME --> PUB --> MCP --> AG
~~~

重点回答：

- Context Intelligence 到底输入什么？
- semantic anchors 是什么？
- proposal 如何存储和覆盖？
- eval 实际验证什么？
- publish 后 Agent 看到什么？
- regenerate 时 human edits 怎么处理？
- context lifecycle 的真正 state machine 是什么？

---

# Sources

- DataHub Docs, **The Metadata Model**  
  https://docs.datahub.com/docs/metadata-modeling/metadata-model

- DataHub Docs, **Metadata Events**  
  https://docs.datahub.com/docs/what/mxe

- DataHub Docs, **Logical Models**  
  https://docs.datahub.com/docs/features/feature-guides/logical-models/overview

- DataHub Docs, **Extending the Metadata Model**  
  https://docs.datahub.com/docs/metadata-modeling/extending-the-metadata-model

- DataHub Docs, **Search**  
  https://docs.datahub.com/docs/how/search
