# DataHub Research

研究 DataHub 的**架构思想、设计哲学，以及它如何从 Metadata Platform 演化为 AI 时代的 Context Layer / Context Platform**。

> DataHub: https://datahub.com/  
> Documentation: https://docs.datahub.com/

## 最新专题 — Metrics & Semantic Models

- **[02 — DataHub 如何管理业务语义，而不成为指标计算引擎](docs/research/02-metrics-and-semantic-models.md)**
- [官方材料释义与证据表](docs/official-docs/metrics-and-semantic-models.md)

核查日期：2026-09-28。研究定义目录与计算引擎的边界、语义模型与血缘、OSI / Apache Ossie、跨平台身份，以及 Agent 消费的现状与目标架构。此专题沿用对话编号 02，保留下面原有主线与案例。

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
- [05 — OSS vs Cloud Context Architecture](docs/product-mapping/05-oss-vs-cloud-context-architecture.md)

## Phase 3 — Concrete Case Studies

- [Phase 3 Index](docs/case-studies/README.md)
- [Case 01 — Analytics Agent: Enterprise Customer Net Revenue](docs/case-studies/01-analytics-agent-net-revenue.md)
- [Case 02 — Schema Change Incident Agent](docs/case-studies/02-schema-change-incident-agent.md)
- **[Case 03 — Governance / PII Agent](docs/case-studies/03-governance-pii-agent.md)**
- 下一篇：**Cross-case Architecture Synthesis**

## Case 03 Architecture

~~~mermaid
flowchart LR
    CLASS[Classification]
    GRAPH[Lineage / Agent Graph]
    DISC[Scoped Discovery]
    ID[Agent + Human Identity]
    POLICY[Runtime Policy]
    DATA[Data Enforcement]
    REM[Remediation]
    VERIFY[Verify]
    AUDIT[Audit]

    CLASS --> GRAPH --> DISC --> ID --> POLICY --> DATA --> REM --> VERIFY --> AUDIT
~~~

Case 03 的核心结论：

> **Context visibility != runtime authorization.**

DataHub Cloud Search Access Controls 能在 query-time 限制 metadata search / browse / direct entity discovery；这保护 Context/Metadata Plane。

DataHub Agent Registry 可以把 data classification 通过 lineage 传播到 consuming Agent，并触发 incident，帮助治理团队发现风险。

但最终某个 Agent / Human 是否能读取具体敏感字段，仍需要 runtime identity、purpose 与 query-time enforcement。

所以生产 Agent Governance 必须同时回答：

~~~text
What can the Agent discover?
What is the data classified as?
Who is invoking the Agent?
What is the purpose?
What may that principal actually read or do?
~~~

## Next

Cross-case synthesis 会从 Analytics、Incident Repair、Governance 三个案例中抽出 Context Platform 的不可约核心。
