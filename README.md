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
- [01 — Reference Architecture → DataHub Current Product](docs/product-mapping/01-reference-architecture-to-datahub.md)
- [02 — Metadata Model as Context Substrate](docs/product-mapping/02-metadata-model-as-context-substrate.md)
- [03 — Context Lifecycle Product Mapping](docs/product-mapping/03-context-lifecycle.md)
- [04 — Agent Governance Product Mapping](docs/product-mapping/04-agent-governance.md)
- **[05 — OSS vs Cloud Context Architecture](docs/product-mapping/05-oss-vs-cloud-context-architecture.md)**

当前快照：**2026-09-28**

## DataHub 当前架构边界

~~~mermaid
flowchart TB
    subgraph CORE[DataHub Core — Open Context Substrate]
        MODEL[Metadata Standard]
        GRAPH[Graph / Lineage]
        DOC[Context Documents]
        GOV[Governance Metadata]
        QUAL[Quality / Incidents / Contracts]
        API[API / SDK]
        MCP[Self-hosted MCP]
    end

    subgraph CLOUD[DataHub Cloud — Context Operating Layer]
        INTEL[Context Intelligence]
        EVAL[Eval / Proposal / SME]
        ASK[Ask DataHub]
        ACL[Search-time Access Control]
        REG[Agent Registry]
        AG[Agents / Tasks / Decisions]
        MMCP[Managed / Scoped MCP]
        AUTO[AI Audit / Advanced Automation]
    end

    subgraph EXT[External Runtime]
        SEM[Semantic Runtime]
        POLICY[Runtime Authorization]
        DATA[Warehouse / SaaS / Tools]
    end

    CORE --> CLOUD
    CLOUD --> EXT
    CORE --> EXT
~~~

## Phase 2 核心结论

> **DataHub Core 已经是一个很强的 Open Metadata / Context Substrate。**

它并不缺：

- graph；
- lineage；
- documents；
- APIs；
- MCP；
- governance metadata；
- quality primitives。

DataHub Cloud 真正增加的是：

> **Context lifecycle + Agent governance + managed security / automation / operating model。**

因此不能简单理解：

~~~text
OSS = Catalog
Cloud = Context Platform
~~~

更准确：

~~~text
Core = Context Substrate
Cloud = Productized Context Operating Layer
~~~

## Phase 3

下一阶段停止继续拆 feature。

进入：

> **Concrete Case Studies**

第一篇建议：

**Analytics Agent：过去 90 天 Enterprise Customer Net Revenue 是多少？同比如何？为什么应该相信这个数字？**

用一个任务贯穿：

~~~text
Intent
-> Context
-> Semantic
-> Policy
-> Data
-> Evidence
-> Answer
-> Audit
~~~
