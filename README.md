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
- [02 — Metadata Model as Context Substrate](docs/product-mapping/02-metadata-model-as-context-substrate.md)
- **[03 — Context Lifecycle Product Mapping](docs/product-mapping/03-context-lifecycle.md)**
- 下一篇：**Agent Governance Product Mapping**

当前快照时间：**2026-09-28**

## DataHub Context Lifecycle

~~~mermaid
flowchart LR
    SRC[Metadata / Query History / BI]
    GEN[Context Generation]
    DOC[Context Document]
    PROP[Proposal]
    EVAL[Eval]
    SME[SME Review]
    PUB[Publish]
    MCP[MCP / Search]
    AG[Agent]

    SRC --> GEN --> DOC --> PROP --> EVAL --> SME --> PUB --> MCP --> AG
~~~

当前官方文档确认：

- Context Platform 是 Public Beta；
- Auto-publish 默认关闭；
- generated context 默认 unpublished；
- human-edited context 在 full refresh 中保留，并优先于 agent-generated metadata；
- 只有 published context 对 Agent 和 search 可见；
- reviewer 可以运行 eval 和 Ask DataHub preview；
- 官方建议从单 domain / 小规模 context 开始渐进发布。

这意味着 DataHub 已经有真实的：

> **Generation Plane -> Governance Plane -> Activation Plane**

但当前 lifecycle 的主要对象仍然是 **Context Document**，不是我们 Reference Architecture 中更通用的 assertion-level lifecycle。

## 当前 Product Mapping 主结论

> **DataHub Metadata Model 是优秀的 Context Substrate；Context Hub 已经提供一个完整度较高的 Document-level Governed Context Lifecycle。**

下一步研究 Agent 本身如何进入这个治理图。
