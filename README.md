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
8. **[08 — Context Platform Failure Modes](docs/research/08-context-platform-failure-modes.md)**
9. 下一篇：**DataHub Design Philosophy — Synthesis**

## 当前参考架构

~~~mermaid
flowchart TB
    REAL[Enterprise Reality]

    subgraph EP[Epistemic / Context Plane]
        OBS[Observation]
        ASSERT[Assertions]
        GRAPH[Context Graph]
        TRUST[Trust Envelope]
        REC[Reconciliation / Invalidation]
        PUB[Publication]
        RET[Scoped Retrieval]
    end

    subgraph OTHER[Execution Planes]
        SEM[Semantic: compute]
        POLICY[Policy: allow]
        DATA[Data: execute]
        AG[Agent: reason / act]
    end

    REAL --> OBS --> ASSERT --> GRAPH --> TRUST --> REC --> PUB --> RET --> AG
    AG --> SEM --> POLICY --> DATA
    DATA --> REAL
~~~

## 当前研究形成的核心原则

> **Semantic Layer makes meaning executable. Context Layer makes meaning situationally trustworthy.**

> **Context Graph 的关键不只是 representation，而是 dependency-aware maintenance。**

> **Freshness 不是 updated_at；Provenance 不是 audit log。**

> **Agents propose. Evidence verifies. Authorities publish.**

> **Shared truth means same governed substrate, not same access, same presentation, or one centralized author.**

> **Context Platform 更接近 Enterprise AI 的 Epistemic Control Plane，而不是完整 AI Control Plane。**

> **Context informs policy; policy governs execution.**

> **Unknown must remain a state; absence of evidence must not silently become false.**

> **A shared Context Plane reduces duplication but increases blast radius, so rollback and scoped publication are first-class reliability features.**

## Reliability View

~~~mermaid
flowchart LR
    SOURCE[Source]
    OBS[Observe]
    TRUST[Qualify]
    PUB[Publish]
    AG[Agent]
    ACT[Action]

    SOURCE --> OBS --> TRUST --> PUB --> AG --> ACT

    SOURCE -.silence.-> OBS
    TRUST -.conflict.-> PUB
    PUB -.bad rollout.-> AG
    AG -.poisoned write-back.-> TRUST
~~~

Context Platform 的可靠性不应只按 ingest/search/MCP 衡量，而应按：

- detection
- qualification
- validation
- scoping
- publication
- invalidation
- reconciliation
- rollback
- audit

衡量。

## 研究方法

**原文链接 → 中文释义 → 厂商主张 → 架构拆解 → 外部对照 → 我们的工作定义 → Failure-oriented validation**

关注：

- architecture boundaries
- trust / authority
- temporal validity
- provenance
- source health
- context security
- blast radius
- rollback
- decision reproducibility
