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
- **[04 — Agent Governance Product Mapping](docs/product-mapping/04-agent-governance.md)**
- 下一篇：**OSS vs Cloud Context Architecture**

当前快照：**2026-09-28**

## Agent Governance 当前模型

~~~mermaid
flowchart TB
    REG[Agent Registry<br/>Identity · Ownership · Version · Lineage]
    SCOPE[Context Scope<br/>View / Scoped MCP]
    CAP[Capability Scope<br/>Tools / Plugins]
    ID[Runtime Identity<br/>OAuth / Creator / Service Account]
    RUN[Tasks / Decisions]
    AUDIT[Run History / Tool Trace]

    REG --> RUN
    SCOPE --> RUN
    CAP --> RUN
    ID --> RUN
    RUN --> AUDIT
~~~

当前最重要的 Product Mapping 结论：

> **DataHub 的 Agent governance graph 已经比 custom Agent runtime 本身成熟。**

Agent Registry 已经把 Agent / Skill / Tool / MCP Server 放进 metadata/lineage graph，并继承 ownership、versioning、classification propagation 与 incident governance。citeturn451627view0

Custom Agents 当前仍是 DataHub Cloud Private Beta，可配置 instructions、DataHub tools、external AI Plugins 和 View scope，并通过 Tasks / Decisions 执行工作流。citeturn263058view0

当前一个重要限制是：Task 以创建 Task 的用户权限运行；官方计划未来支持指定 service account。citeturn263058view1

因此：

> **Agent identity、Context scope、Tool scope、Runtime principal 是四个不同维度，不能混成一个“Agent 权限”。**

## 研究方法

**官方证据 → 当前能力 → Reference Architecture responsibility → Gap / boundary**
