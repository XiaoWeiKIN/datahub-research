# AI Agent Context：Agent 所需的四层上下文，以及每一层缺失时会怎样失败

**类型：** 官方文章中文学习译注（非全文逐字翻译）  
**作者：** Sat Duggal  
**发布日期：** 2026-09-22  
**原文：** https://datahub.com/blog/ai-agent-context/

> 说明：本文保留原文的核心结构与主要论点，并结合本仓库已有 Context Layer 研究做中文释义；完整原文请以官方链接为准。

---

## 一句话理解

AI Agent 的 context 不只是 prompt、memory 或 context window。对于企业数据 Agent，更重要的是它背后的 **technical、operational、business、organizational** 四层企业上下文是否统一、可信并保持最新。

如果缺少任何一层，Agent 都可能得到一个“技术上能运行、形式上很可信、业务上却错误”的答案。

---

## 四层 Context

| Layer | 中文理解 | 主要回答 |
|---|---|---|
| Technical Context | 技术上下文 | 正确的资产是什么？结构和依赖关系是什么？ |
| Operational Context | 运行上下文 | 这个资产现在是否新鲜、健康、可相信？ |
| Business Context | 业务上下文 | 企业在当前业务语境下究竟如何定义这个概念？ |
| Organizational Context | 组织上下文 | 谁拥有、谁有权限、谁对结果负责？ |

四层必须相互连接。单独知道 schema、freshness、业务术语或 owner 都不够；Agent 需要沿关系把这些信息组合起来。

---

## 缺失四层时的典型失败

~~~mermaid
flowchart TB
    Q[Business Question]

    Q --> T[Technical missing<br/>选错资产]
    Q --> O[Operational missing<br/>使用 stale / broken data]
    Q --> B[Business missing<br/>使用错误定义]
    Q --> G[Organizational missing<br/>权限 / ownership / accountability 不清]
~~~

这类失败最大的危险是 **silent failure**：查询往往成功、SQL 也合法，因此系统没有明显错误信号。

---

## Relevant / Reliable / Retained

原文进一步强调：Context 除了“包含什么”，还必须具备三个质量：

- **Relevant**：与当前任务、时间和 domain 有关；
- **Reliable**：能够追溯 provenance，并知道哪些来源可信；
- **Retained**：知识能够跨会话、跨 Agent 持续存在，而不是每次从零开始。

这和本仓库前面的结论基本一致：Context Layer 的价值不在“提供更多 token”，而在提供 **task-applicable、可追溯、可持续维护的企业事实**。

---

# 与本仓库研究框架的连接

这篇文章可以直接映射到我们的五 Plane 模型。

~~~mermaid
flowchart LR
    TECH[Technical]
    OPS[Operational]
    BUS[Business]
    ORG[Organizational]

    TECH --> CTX[Epistemic / Context Plane]
    OPS --> CTX
    BUS --> CTX
    ORG --> CTX

    CTX --> AG[Agent]
~~~

但要注意：

- Organizational Context 提供 ownership / role / authority 信号；
- 真正 runtime authorization 仍属于 Policy / IAM Plane；
- Business Context 不等于 Semantic Layer，后者还负责 executable metrics / query semantics；
- MCP 只是 Context Activation / delivery mechanism，不是 Context 本身。

---

## 这篇文章最值得保留的设计判断

### 1. Context failure 往往发生在进入 context window 之前

Context Engineering 可以很好地组织 prompt、tool、memory 和 retrieved documents，但如果 retrieval 背后的企业 context 本来就是错的，那么 window engineering 无法修复它。

因此：

> **Context Engineering 管理“这一轮给模型什么”；Context Management 管理“企业有什么可信东西可以给模型”。**

### 2. Context Layer 必须连接四层，而不是建立四个孤岛

真正的价值来自关系，例如：

~~~mermaid
graph LR
    TERM[Business Term]
    METRIC[Metric]
    TABLE[Dataset]
    QUALITY[Freshness / Quality]
    OWNER[Owner / Team]

    TERM --> METRIC --> TABLE
    TABLE --> QUALITY
    OWNER --> METRIC
~~~

Agent 的答案通常依赖一条 context path，而不是单个 metadata 字段。

### 3. Context-aware 不等于 Context-connected

Agent 能连接 warehouse、搜索 schema、调用 MCP，只说明它“能访问”。

它是否真正 context-aware，取决于 retrieval 结果是否同时携带：

- meaning；
- trust；
- freshness；
- ownership / authority；
- provenance。

### 4. Context Platform 的真正任务是降低 silent wrong answer

一个成熟 Context Platform 需要让 Agent 在回答前能够判断：

> 我选的是不是正确资产？  
> 数据现在是否健康？  
> 当前业务定义是否适用？  
> 谁有权定义和使用它？

这也是我们把 Context Layer 定义成 **Epistemic Infrastructure** 的原因。

---

## 与 Phase 1 的对应关系

- [01 — Why Context Layer?](../research/01-why-context-layer.md)
- [03 — Context Layer vs Semantic Layer](../research/03-context-layer-vs-semantic-layer.md)
- [04 — Context Freshness & Provenance](../research/04-context-freshness-and-provenance.md)
- [06 — Human + Agent Shared Truth Plane](../research/06-human-agent-shared-truth-plane.md)
- [07 — Context Layer Reference Architecture](../research/07-context-layer-reference-architecture.md)

---

## Source

DataHub, **AI Agent Context: The Four Layers Every Agent Needs (and How Each One Fails)**, 2026-09-22  
https://datahub.com/blog/ai-agent-context/
