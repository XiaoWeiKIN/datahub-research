# DataHub Research

研究 DataHub 的**架构思想、设计哲学，以及它如何从 Metadata Platform 演化为 AI 时代的 Context Layer / Context Platform**。

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
- [Reference Architecture → DataHub](docs/product-mapping/01-reference-architecture-to-datahub.md)
- [Metadata Model as Context Substrate](docs/product-mapping/02-metadata-model-as-context-substrate.md)
- [Context Lifecycle](docs/product-mapping/03-context-lifecycle.md)
- [Agent Governance](docs/product-mapping/04-agent-governance.md)
- [OSS vs Cloud](docs/product-mapping/05-oss-vs-cloud-context-architecture.md)

## Phase 3 — Concrete Case Studies

- [Phase 3 Index](docs/case-studies/README.md)
- [Analytics Agent](docs/case-studies/01-analytics-agent-net-revenue.md)
- [Schema Change Incident Agent](docs/case-studies/02-schema-change-incident-agent.md)
- [Governance / PII Agent](docs/case-studies/03-governance-pii-agent.md)
- [Cross-case Architecture Synthesis](docs/case-studies/04-cross-case-architecture-synthesis.md)

## Phase 4 — Comparative Architecture

- [Phase 4 Index](docs/comparative/README.md)
- **[01 — Context Architecture Landscape](docs/comparative/01-context-architecture-landscape.md)**
- 下一篇：**DataHub vs Atlan**

## Comparative Landscape

~~~mermaid
flowchart TB
    DH[DataHub<br/>Active Metadata + Context Lifecycle]
    AT[Atlan<br/>Enterprise Data Graph + Context/Governance]
    OM[OpenMetadata<br/>Open Metadata Knowledge Graph]
    CO[Collibra<br/>Governance Operating Model]
    DBT[dbt<br/>Executable Semantic Layer]
    KG[Knowledge Graph<br/>Graph Reasoning]

    CTX[Enterprise AI Context]

    DH --> CTX
    AT --> CTX
    OM --> CTX
    CO --> CTX
    DBT --> CTX
    KG --> CTX
~~~

2026 年这些路线正在明显收敛：DataHub、Atlan、OpenMetadata、Collibra、dbt 都已经通过 MCP 向 AI client 暴露不同类型的 governed context。

真正需要比较的已经不是：

> 谁支持 MCP / AI Search？

而是：

> **MCP 背后的 authoritative substrate 是什么？context 如何产生、验证、更新、执行和审计？**

## 当前比较框架

~~~text
Identity
Graph
Business Semantics
Operational Context
Authority
Provenance
Temporal Validity
Conflict Model
Context Lifecycle
Agent Activation
Write-back
Semantic Execution
Runtime Authorization
Decision Audit
Interoperability
~~~

不做总体排名，只比较 architecture responsibility。
