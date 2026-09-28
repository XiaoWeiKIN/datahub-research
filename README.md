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

第二阶段开始把 Reference Architecture 映射到 DataHub 当前真实产品能力。

- [Phase 2 Index](docs/product-mapping/README.md)
- **[01 — Reference Architecture → DataHub Current Product](docs/product-mapping/01-reference-architecture-to-datahub.md)**

当前快照时间：**2026-09-28**

## 当前 Product Mapping 结论

~~~mermaid
flowchart LR
    STRONG[Strong / Mature<br/>Ingestion · Graph · Lineage · Search · Governance]
    BETA[2026 Expansion<br/>Context Lifecycle · Agents · Semantic Entities]
    PARTIAL[Partial<br/>Temporal · Authority · Reconciliation · Decision Audit]
    EXT[External<br/>Semantic Execution · Runtime Data Authorization]

    STRONG --> BETA --> PARTIAL
    BETA --> EXT
~~~

DataHub 当前最成熟的是 metadata/context substrate。

Context lifecycle 已经真实存在，但 Context Platform 当前仍是 **Public Beta**；custom Agents 是 **Private Beta**。citeturn339781search0turn341575search1

我们的 Reference Architecture 中更严格的 assertion-level temporal validity、typed authority、generic invalidation 和 decision replay，目前不能从公开官方材料证明为完整统一能力。

## Final Thesis

> **AI Agent 的可靠性上限，不只由模型决定，而由它所处的企业认知基础设施决定。DataHub 最值得研究的地方，是它长期把 metadata 设计成实时、关系化、可治理、可编程的基础设施；这使它能够自然扩展为一种面向 Human 和 Agent 的 Context Platform。**

## 研究方法

**官方证据 → 当前能力 → Reference Architecture responsibility → Gap / boundary**

不把：

- vendor vision 当 current feature；
- 有相似 feature 当完整 primitive；
- Cloud capability 当 OSS capability；
- Agent context 当 runtime authorization。
