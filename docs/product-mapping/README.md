# Phase 2 — DataHub Product Mapping

第一阶段建立了厂商无关的 Context Layer Reference Architecture。

第二阶段逐项检查：

> **DataHub 今天到底实现到了哪里？**

## Mapping

1. [Reference Architecture → DataHub Current Product](01-reference-architecture-to-datahub.md)
2. [Metadata Model as Context Substrate](02-metadata-model-as-context-substrate.md)
3. [Context Lifecycle Product Mapping](03-context-lifecycle.md)
4. [Agent Governance Product Mapping](04-agent-governance.md)
5. [OSS vs Cloud Context Architecture](05-oss-vs-cloud-context-architecture.md)

## Phase 2 完成

当前产品映射形成：

~~~mermaid
flowchart TB
    CORE[DataHub Core<br/>Open Metadata / Context Substrate]
    CLOUD[DataHub Cloud<br/>Context Lifecycle / Agent Operating Layer]
    EXT[External Runtime<br/>Semantic / Policy / Warehouse / Tools]

    CORE --> CLOUD --> EXT
    CORE --> EXT
~~~

### DataHub Core

已经能提供：

- identity / graph / lineage；
- business + technical metadata；
- Context Documents；
- API / SDK；
- self-hosted MCP；
- metadata mutation；
- quality / incidents / contracts；
- policies / Views。

### DataHub Cloud

进一步产品化：

- Context Intelligence；
- eval / proposal / SME review；
- publication / activation；
- Ask DataHub；
- query-time search access control；
- Agent Registry；
- Custom Agents / Tasks / Decisions；
- managed / scoped MCP；
- AI audit；
- advanced observability automation。

## 下一阶段

建议进入：

> **Phase 3 — Concrete Case Studies**

用真实 Agent 任务验证整个 architecture。
