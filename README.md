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
- **[Case 01 — Analytics Agent: Enterprise Customer Net Revenue](docs/case-studies/01-analytics-agent-net-revenue.md)**

Case 01 使用合成企业数据定义，不代表任何真实公司。

它验证：

~~~mermaid
flowchart LR
    INTENT[Intent]
    CTX[Context / Authority / Freshness]
    SEM[Semantic]
    POLICY[Policy]
    DATA[Data Execution]
    EVIDENCE[Evidence]
    ANSWER[Answer]
    AUDIT[Audit]

    INTENT --> CTX --> SEM --> POLICY --> DATA --> EVIDENCE --> ANSWER --> AUDIT
~~~

## Case 01 的核心判断

对于：

> “过去 90 天 Enterprise Customer Net Revenue 是多少？同比如何？为什么我应该相信这个数字？”

正确 Agent 不能从 schema 直接跳到 SQL。

它至少要先解决：

- Enterprise Customer 的定义；
- Net Revenue 的定义；
- authority / domain；
- join / query pattern；
- freshness / quality；
- runtime authorization。

DataHub 当前 Analytics Agent 的 reference implementation 也采用：

> context enrichment before SQL generation

并通过 DataHub MCP / Agent Context Kit 读取 schema、glossary、lineage、quality、usage 等 context，再生成 SQL。citeturn926690search1

DataHub 当前 Context Activation 还明确建议 analytics/text-to-SQL Agent 使用 `datahub-sql-workflow` skill，在回答自然语言业务问题时先搜索 published DataHub context。citeturn926690search0

## 下一 Case

**Schema Change Incident Agent**

验证 read-heavy analytics 之外的：

~~~text
Lineage
-> Incident
-> GitHub / Tool Action
-> Human Decision
-> Verification
-> Context Write-back
~~~
