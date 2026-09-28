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
- [05 — OSS vs Cloud Context Architecture](docs/product-mapping/05-oss-vs-cloud-context-architecture.md)

## Phase 3 — Concrete Case Studies

- [Phase 3 Index](docs/case-studies/README.md)
- [Case 01 — Analytics Agent: Enterprise Customer Net Revenue](docs/case-studies/01-analytics-agent-net-revenue.md)
- **[Case 02 — Schema Change Incident Agent](docs/case-studies/02-schema-change-incident-agent.md)**
- 下一篇：**Governance / PII Agent**

## Case 02 Architecture

~~~mermaid
flowchart LR
    CHANGE[Schema Change]
    CTX[Fresh Context]
    LINEAGE[Column-level Lineage]
    IMPACT[Blast Radius]
    OWNER[Owner / Repo]
    PLAN[Repair Plan]
    DEC[Human Decision]
    PR[PR / CI]
    VERIFY[Independent Verification]
    WRITE[Incident / Context Write-back]

    CHANGE --> CTX --> LINEAGE --> IMPACT --> OWNER --> PLAN --> DEC --> PR --> VERIFY --> WRITE
~~~

DataHub 2026 Agent Hackathon 已经出现多个类似生产模式：column-level lineage 算 blast radius、修 dbt/Airflow、开 PR、通过 evidence gate、用另一 API read-back 验证，再把 tags、incident、postmortem、lineage/docs 写回 DataHub。citeturn121928search0

Case 02 验证的新原则：

> **An agent cannot prove its own success by reporting success.**

> **Operational action must end in independent verification and governed shared context, not private Agent memory.**

> **Lineage reachability tells you what may be affected; it does not by itself tell you what must be changed.**

## Next

Case 03 将直接测试：

> **Context visibility != runtime authorization**

用 PII / governance propagation / Agent lineage 场景检验 Policy Plane。
