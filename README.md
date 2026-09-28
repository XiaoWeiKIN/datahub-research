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
7. **[07 — Context Layer Reference Architecture](docs/research/07-context-layer-reference-architecture.md)**
8. 下一篇：**Context Platform Failure Modes**

## 当前参考架构

~~~mermaid
flowchart TB
    subgraph DP[Data / Execution Plane]
        WH[Warehouse / Lakehouse]
        OPS[Operational Systems]
        DOC[Docs / SaaS / Repos]
    end

    subgraph EP[Epistemic / Context Plane]
        OBS[Observation]
        ID[Identity]
        ASSERT[Assertions]
        CG[Context Graph]
        TRUST[Provenance / Freshness / Authority]
        REC[Reconciliation / Invalidation]
        PUB[Proposal / Publication]
        RET[Scoped Retrieval]
    end

    subgraph SP[Semantic Plane]
        SM[Semantic Models]
        COMP[Metric / Query Compiler]
    end

    subgraph PP[Identity / Policy Plane]
        PRI[Principal / Delegation]
        PDP[Policy Decision]
        PEP[Policy Enforcement]
    end

    subgraph AP[Agent Plane]
        AG[Agents]
        TASK[Tasks]
        MEM[Private Memory]
    end

    DP --> OBS --> ID --> ASSERT --> CG
    CG --> TRUST --> REC --> PUB --> RET
    RET --> AG

    SM <--> CG
    AG --> COMP --> PDP
    PRI --> PDP
    PDP --> PEP --> DP

    AP --> ASSERT
~~~

## 当前研究形成的核心原则

> **Semantic Layer makes meaning executable. Context Layer makes meaning situationally trustworthy.**

> **Context Graph 的关键不只是 representation，而是 dependency-aware maintenance。**

> **Freshness 不是 updated_at；Provenance 不是 audit log。**

> **Agents propose. Evidence verifies. Authorities publish.**

> **Shared truth means same governed substrate, not same access, same presentation, or one centralized author.**

> **Context Platform 更接近 Enterprise AI 的 Epistemic Control Plane，而不是完整 AI Control Plane。**

> **Context informs policy; policy governs execution.**

## Context Layer 的核心数据模型

重要 context 不应只是 property，而应当是带 trust metadata 的 assertion：

~~~text
ContextAssertion
├── subject / predicate / value
├── scope / domain
├── source / evidence
├── authority
├── provenance
├── valid time / observed time
├── epistemic state
├── publication state
└── dependencies
~~~

## 五个 Plane 的边界

| Plane | 主要问题 |
|---|---|
| Context / Epistemic | 知道什么？为什么相信？当前是否适用？ |
| Semantic | 怎么算？ |
| Identity / Policy | 当前主体是否允许？ |
| Data / Execution | 数据在哪里、如何真实执行？ |
| Agent | 如何解释、规划、调用工具和行动？ |

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
- SLO / failure modes
- AI / Agent security
- human-machine governance
