# Context Platform Failure Modes — 官方材料学习笔记

**Status:** 第一轮  
**Updated:** 2026-09-28

> 本文不是单篇官方文档翻译，而是把 DataHub 当前关于 context failure、generation、validation、activation 与 eventual consistency 的官方材料集中整理。

---

# 1. DataHub 自己总结的五类 Context 问题

DataHub 当前材料把生产 Agent 的 Context 问题归纳为：

1. Discovery — Agent 找不到已有 context；
2. Trust — 无法追溯决策依据；
3. Freshness — context 落后于现实；
4. Governance — 人类时代政策无法直接适应 Agent；
5. Fragmentation — 每个团队自己构建 Context stack。

这个分类很适合用作平台 failure taxonomy 的第一层。

---

# 2. Context Platform 的四个对应机制

DataHub Town Hall 材料把主要问题进一步描述为：

- fragmented；
- inaccessible；
- unvalidated；
- stale。

对应四个产品支柱：

~~~mermaid
flowchart LR
    ING[Context Ingestion]
    INTEL[Context Intelligence]
    HUB[Context Hub]
    ACT[Context Activation]

    ING --> INTEL --> HUB --> ACT
~~~

其中最值得研究的是 Hub：

> Agent 使用 context 之前，需要先建立 validation / publication boundary。

---

# 3. DataHub 默认不是 Auto-publish

当前 Context Generation 文档明确：

- Auto-publish 默认 disabled；
- generated context 默认 unpublished；
- 未发布 context 对 Agent 和 search 不可见；
- Data Expert 验证后 publish；
- 官方建议先建立 metadata eval，再考虑 auto-publish。

这实际上是一个：

> fail-safe publication default。

---

# 4. 官方建议 Incremental Publication

Activate Context 文档建议：

- 先从小范围高置信 documents 开始；
- 在一个 domain 中测试；
- 用 Ask DataHub 比较发布前后 behavior；
- 运行 eval；
- 验证之后再扩大 rollout。

这可以理解成：

> Context canary deployment。

---

# 5. Eval 本身不是绝对 Truth

Validate Context Proposals 文档也提醒：

eval criteria 可能过于具体，导致“看起来正确”的 document fail。

因此官方机制本身承认：

> eval 是辅助质量信号，而不是自动 truth oracle。

这支持我们的结论：

~~~text
Eval Pass != Authority
~~~

---

# 6. Human Context Precedence

官方文档说明：

> human-edited business metadata 在 context regeneration 中保留，并优先于 agent-generated metadata。

这是非常重要的 conflict rule。

它避免 Context Curator 每次 full refresh 都覆盖已经经过 SME 修正的 context。

---

# 7. Eventual Consistency 是现实约束

DataHub Support 明确说明：

DataHub metadata 进入系统后会异步完成：

- Elasticsearch indexing；
- relationship / lineage update；
- aggregation；
- graph update。

因此：

> ingestion completed != UI/search immediately converged。

官方支持文档甚至给出大规模 ingestion 可延迟数小时的案例。

这意味着 Agent 设计不能假设：

~~~text
write -> immediate consistent read
~~~

---

# 8. Query History 自身也可能有 Source Lag

Context Generation 文档指出：

Snowflake ACCOUNT_USAGE query history 本身可能落后真实时间约 45 分钟到 3 小时。

因此 Context Freshness 的 lag 不一定来自 DataHub。

链路可能是：

~~~text
Reality
-> Source telemetry lag
-> DataHub ingestion
-> processing/indexing
-> Context generation
-> publication
-> Agent
~~~

这进一步证明需要 end-to-end freshness model，而不是只监控平台内部 ingestion。

---

# 9. Context Layer 的 Failure Chain

基于官方材料，可以构造：

~~~mermaid
flowchart LR
    SOURCE[Source Lag]
    ING[Ingestion Lag]
    GEN[Generated Context]
    WEAK[Weak Eval]
    PUB[Publish]
    MCP[MCP]
    AG[Agent]

    SOURCE --> ING --> GEN --> WEAK --> PUB --> MCP --> AG
~~~

每个组件都“正常运行”，最终仍可能得到错误 Agent answer。

这说明 production reliability 要看完整 chain。

---

# 10. Agents 的 Scope 是重要 Isolation Boundary

当前 Agents 文档允许每个 Agent配置：

- instructions；
- tools；
- external plugins；
- View scope。

View 限制 Agent 能发现的资产。

这为 Context Platform 提供一个重要的 blast-radius boundary：

> 不同 Agent 不需要看到完整 Context Graph。

---

# 11. Context Activation 明确区分 Published / Unpublished

当前文档非常明确：

> Only published context documents are visible to agents.

这可能是 DataHub Context Platform 最重要的 reliability design 之一。

它建立：

~~~text
generation state
!=
consumption state
~~~

---

# 12. 当前学习结论

DataHub 当前已经体现几个 failure-aware design：

- no default auto-publish；
- proposal / publish boundary；
- human precedence；
- eval before rollout；
- incremental activation；
- scoped agents；
- eventual consistency acknowledgment。

但官方公开材料仍不能证明平台已经统一解决：

- generalized context conflict；
- source silence propagation；
- assertion-level temporal validity；
- cross-plane policy drift；
- rollback propagation；
- historical decision replay。

这些应继续作为观察项，而不是假设已经实现。

---

# Related Research

- [08 — Context Platform Failure Modes](../research/08-context-platform-failure-modes.md)
- [07 — Context Layer Reference Architecture](../research/07-context-layer-reference-architecture.md)
- [04 — Context Freshness & Provenance](../research/04-context-freshness-and-provenance.md)
