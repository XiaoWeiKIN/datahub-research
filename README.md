# DataHub Research

研究 DataHub 的**架构思想、设计哲学，以及它如何从 Metadata Platform 演化为 AI 时代的 Context Layer / Context Platform**。

官方文档翻译是学习手段，不是研究终点。

> DataHub: https://datahub.com/  
> Documentation: https://docs.datahub.com/

## Start Here

1. **[01 — Why Context Layer?](docs/research/01-why-context-layer.md)**
2. **[02 — Context Graph vs Knowledge Graph](docs/research/02-context-graph-vs-knowledge-graph.md)**
3. **[03 — Context Layer vs Semantic Layer](docs/research/03-context-layer-vs-semantic-layer.md)**
4. **[04 — Context Freshness & Provenance](docs/research/04-context-freshness-and-provenance.md)**
5. **[05 — Agent Read / Write Context](docs/research/05-agent-read-write-context.md)**
6. **[06 — Human + Agent Shared Truth Plane](docs/research/06-human-agent-shared-truth-plane.md)**
7. 下一篇：**Context Layer Reference Architecture**

## 当前总模型

~~~mermaid
flowchart TB
    subgraph EP[Epistemic / Context Plane]
        CG[Context Graph]
        ASSERT[Assertions]
        AUTH[Authority]
        PROV[Provenance]
        FRESH[Freshness]
        PUB[Proposal / Publication / Reconciliation]
    end

    subgraph SP[Semantic Execution Plane]
        SM[Semantic Models]
        COMP[Metric / Query Compiler]
    end

    subgraph PP[Identity / Policy Plane]
        ID[Identity / Delegation]
        PDP[Policy Decision]
        PEP[Policy Enforcement]
    end

    subgraph AP[Agent Plane]
        AG[Agents]
        TASK[Tasks]
        MEM[Private / Task Memory]
    end

    subgraph DP[Data / Execution Plane]
        WH[Warehouse / Lakehouse]
        OPS[Operational Systems]
    end

    EP --> AG
    AG --> SP
    AG --> PP
    SP --> PP
    PP --> DP
    DP --> EP
    SP --> EP
    AP --> EP
~~~

当前研究形成的核心原则：

> **Semantic Layer makes meaning executable. Context Layer makes meaning situationally trustworthy.**

> **Context Graph 的关键不只是 representation，而是 dependency-aware maintenance。**

> **Freshness 不是 updated_at；Provenance 不是 audit log。**

> **Agents propose. Evidence verifies. Authorities publish.**

> **Shared truth means same governed substrate, not same access, same presentation, or one centralized author.**

> **Context Platform 更接近 Enterprise AI 的 Epistemic Control Plane，而不是完整 AI Control Plane。**

## 为什么叫 Epistemic Control Plane？

它主要管理：

- 系统知道什么；
- 什么定义适用；
- 哪些 context current；
- 为什么相信；
- 谁拥有 authority；
- 哪些 assertions 已发布；
- Human / Agent 消费的是哪个版本。

它不应该取代 warehouse / data plane、semantic execution、IAM、runtime policy enforcement、agent runtime 或 model serving。

## 研究方法

**原文链接 → 中文释义 → 厂商主张 → 架构拆解 → 外部对照 → 我们的工作定义 → 开放问题**

关注：

- architectural responsibility
- system boundaries
- information model
- authority / trust model
- temporal model
- provenance
- reconciliation
- AI / Agent security
- human-machine governance
