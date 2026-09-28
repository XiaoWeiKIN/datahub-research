# DataHub Research

研究 DataHub 的**架构思想、设计哲学，以及它如何从 Metadata Platform 演化为 AI 时代的 Context Layer / Context Platform**。

官方文档翻译是学习手段，不是研究终点。

> DataHub: https://datahub.com/  
> Documentation: https://docs.datahub.com/

## Phase 1 — Architecture & Philosophy

1. [Why Context Layer?](docs/research/01-why-context-layer.md)
2. [Context Graph vs Knowledge Graph](docs/research/02-context-graph-vs-knowledge-graph.md)
3. [Context Layer vs Semantic Layer](docs/research/03-context-layer-vs-semantic-layer.md)
4. [Context Freshness & Provenance](docs/research/04-context-freshness-and-provenance.md)
5. [Agent Read / Write Context](docs/research/05-agent-read-write-context.md)
6. [Human + Agent Shared Truth Plane](docs/research/06-human-agent-shared-truth-plane.md)
7. [Context Layer Reference Architecture](docs/research/07-context-layer-reference-architecture.md)
8. [Context Platform Failure Modes](docs/research/08-context-platform-failure-modes.md)
9. [DataHub Design Philosophy — Synthesis](docs/research/09-datahub-design-philosophy-synthesis.md)

## Phase 2 — Product Mapping

- [Phase 2 Index](docs/product-mapping/README.md)
- [01 — Reference Architecture → DataHub Current Product](docs/product-mapping/01-reference-architecture-to-datahub.md)
- **[02 — Metadata Model as Context Substrate](docs/product-mapping/02-metadata-model-as-context-substrate.md)**
- 下一篇：**Context Lifecycle Product Mapping**

当前快照时间：**2026-09-28**

## 为什么 Metadata Model 是关键？

~~~mermaid
flowchart LR
    URN[URN / Identity]
    ENTITY[Entity]
    ASPECT[Typed Aspects]
    REL[Relationships]
    TIME[Versioned + Timeseries]
    EVENT[MCP / MCL]
    CONTEXT[Context Platform]

    URN --> ENTITY --> ASPECT
    ASPECT --> REL
    ASPECT --> TIME
    ASPECT --> EVENT
    REL --> CONTEXT
    TIME --> CONTEXT
    EVENT --> CONTEXT
~~~

DataHub 当前 metadata model 的长期优势：

- stable identity；
- typed facets；
- atomic writes；
- native graph edges；
- metadata history；
- operational timeseries；
- event-driven change stream；
- extensibility。

这解释了为什么 DataHub 可以在旧 Metadata Platform 上继续构建 Context Platform，而不是重新设计底层 substrate。

## 当前模型边界

DataHub 的核心原语仍然更接近：

> **Entity 的某个 Aspect 当前是什么。**

我们的 AI-era Reference Architecture 更进一步要求：

> **某条 Assertion 在什么 scope / authority / evidence / valid-time 下成立。**

因此：

> **DataHub Metadata Model 是优秀的 Context Substrate，但不等于完整的 Epistemic Model。**

一个可能的长期 hybrid：

~~~mermaid
flowchart TB
    E[Entity]
    A[Typed Aspects]
    C[Scoped Assertions]
    G[Context Graph]

    E --> A --> G
    E --> C --> G
~~~

Aspect 继续承载 schema、ownership、lineage 等强类型 metadata；Assertion 更适合承载可冲突、带 authority / evidence / temporal validity 的组织知识。

## 研究方法

**官方证据 → 当前能力 → Reference Architecture responsibility → Gap / boundary**

不把：

- version history 当 bitemporal truth；
- URN identity 当 entity resolution；
- relationship graph 当 epistemic graph；
- write provenance 当 knowledge provenance。
