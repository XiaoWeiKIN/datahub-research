# DataHub Design Philosophy — 官方材料综合学习笔记

**Status:** 第一轮综合  
**Updated:** 2026-09-28

> 本文不是某一篇官方文档的翻译，而是把 2022–2026 DataHub 公开材料中的设计主线串起来。

---

# 1. 一个明显的连续性：Metadata 从来不只是 Catalog UI

早期 DataHub / Acryl 的公开材料已经强调：

- active metadata；
- governance as code；
- shift left；
- unified metadata；
- relationships / lineage；
- event-driven metadata。

这些概念出现得比 Context Platform 早很多。

2026 年 DataHub 把产品叙事升级成 Context Platform 后，它们重新被解释为：

- Agent freshness；
- Agent provenance；
- Agent governance；
- machine-readable context；
- shared enterprise truth。

因此 Context Platform 更像是长期 metadata architecture 的上层扩展。

---

# 2. Graph-first 是长期主线

DataHub 当前明确回顾：

> DataHub started as a metadata graph.

早期 graph 连接：

- dataset；
- pipeline；
- dashboard；
- owner；
- lineage。

当前 Context Graph 再加入：

- business definitions；
- documents；
- policies；
- decisions；
- Agent / AI assets。

结构没有被抛弃。

Graph scope 被扩大。

---

# 3. Event-driven / Active Metadata 也是长期主线

早期 active metadata 的问题：

> 如何让 metadata 跟现实变化？

2026 Context Platform 的问题：

> 如何让 Agent 不使用 stale context？

本质是同一个 temporal architecture 问题。

只是 consumer 从 Human 扩展成 Agent。

---

# 4. Shift Left 与 Agent Context

2022 的 Shift Left 强调：

> metadata 在 source / code 附近声明和产生。

AI 时代这意味着：

> 不应该主要依赖 Agent 或 LLM 事后从文档中猜出企业语义。

尽可能让：

- data contract；
- ownership；
- schema；
- semantic definitions；
- policy intent；

靠近真实 source 声明。

Agent inference 负责补充，不应取代 authoritative declaration。

---

# 5. Governance 从 Human Workflow 变成 Machine Context

过去：

> ownership / classification / domain

主要是给 catalog user 看。

现在：

> 这些字段直接影响 Agent 判断哪个数据可信、相关、可用。

因此 governance 从：

> UI metadata

变成：

> reasoning input.

---

# 6. Context Platform 的新增部分

2026 相比过去明显新增：

- unstructured Context Documents；
- Context Intelligence；
- Context Curator；
- proposals；
- evals；
- publication workflow；
- MCP；
- Agent Context Kit；
- Agent Registry；
- Agents / Tasks / Decisions。

也就是说：

> DataHub 不只是扩大 metadata scope，也增加了 context lifecycle 与 Agent lifecycle。

---

# 7. DataHub 当前对 Context-Aware Agent 的定义

官方最近提出一个重要观点：

> Context-awareness 是 underlying context infrastructure 的属性，而不是单纯模型 / prompt 的属性。

这是整个产品演进的逻辑中心。

一个 Agent 可以：

- 会调用数据库；
- 会写 SQL；
- 会检索表；

但如果不知道：

- certification；
- ownership；
- lineage；
- business meaning；
- freshness；

它只是：

> context-connected

还不是：

> context-aware.

---

# 8. Context Platform 当前的官方要求

DataHub 当前常见表述可以压缩为：

1. unified graph；
2. structured + unstructured context；
3. semantic retrieval；
4. agent-native protocol；
5. freshness；
6. governance / trust；
7. read/write feedback。

这些已经明显超出 legacy data catalog。

---

# 9. 官方叙事需要保持批判性

以下表达应当视为 vendor claim，而不是自动接受：

### “Real-time”

DataHub 自身也有 async processing / eventual consistency，所以应转成可度量 SLO。

### “Single source of truth”

企业有 domain-specific / temporal truth，应理解成 governed truth model。

### “Context Graph is new”

Graph / provenance / ontology 都有更长技术传统。

### “Context Platform replaces other layers”

Semantic / policy / runtime 等仍然有独立责任。

---

# 10. 我们从官方材料抽象出的长期设计哲学

~~~text
Metadata is infrastructure
Relationships are first-class
Context should be active
Governance belongs in the model
Capture context near the source
Use one substrate across use cases
Serve humans and machines
Keep generation separate from publication
Federate authority
Continuously reconcile reality and declarations
~~~

这十条比任何某一版产品 feature 更值得长期学习。

---

# Related Research

- [09 — DataHub Design Philosophy — Synthesis](../research/09-datahub-design-philosophy-synthesis.md)
- [07 — Context Layer Reference Architecture](../research/07-context-layer-reference-architecture.md)
- [08 — Context Platform Failure Modes](../research/08-context-platform-failure-modes.md)
