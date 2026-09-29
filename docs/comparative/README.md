# Phase 4 — Comparative Architecture

Phase 4 不做产品排名。

目标是比较不同技术路线如何分配下面这些责任：

- Identity
- Graph / Relationships
- Context Lifecycle
- Authority / Governance
- Freshness / Operational State
- Semantic Execution
- Policy / Authorization
- Agent Activation
- Write-back
- Audit / Provenance

## Comparative Notes

1. [Context Architecture Landscape — DataHub / Atlan / OpenMetadata / Collibra / dbt / Knowledge Graph](01-context-architecture-landscape.md)

## 比较原则

不是问：

> 谁的 feature 更多？

而是问：

> **每条路线把“企业事实、语义、信任、执行和 Agent access”分别放在哪一层？**

## Archetypes

~~~mermaid
flowchart TB
    META[Metadata / Catalog-first]
    GOV[Governance-first]
    SEM[Semantic-execution-first]
    KG[Knowledge-graph-first]
    RAG[RAG / Memory-first]

    META --> CTX[Enterprise AI Context]
    GOV --> CTX
    SEM --> CTX
    KG --> CTX
    RAG --> CTX
~~~

我们会重点观察不同路线最终是否正在收敛到同一类 Context Infrastructure，以及它们保留了哪些不同的“中心原语”。
