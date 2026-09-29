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
- [Case 02 — Schema Change Incident Agent](docs/case-studies/02-schema-change-incident-agent.md)
- [Case 03 — Governance / PII Agent](docs/case-studies/03-governance-pii-agent.md)
- **[Case 04 — Cross-case Architecture Synthesis](docs/case-studies/04-cross-case-architecture-synthesis.md)**

## Cross-case Runtime Architecture

~~~mermaid
flowchart LR
    SCOPE[Scope]
    CTX[Trusted Context]
    AUTH[Authority]
    EXEC[Deterministic Execution]
    VERIFY[Verification]
    AUDIT[Audit]

    SCOPE --> CTX --> AUTH --> EXEC --> VERIFY --> AUDIT
~~~

Analytics、Incident Repair、Governance 三类 Agent 最终都依赖相同闭环。

### Analytics

> Context chooses; semantics computes.

### Incident

> Automation without independent verification is incomplete automation.

### Governance

> Context visibility != runtime authorization.

### Cross-case

> **Agent Reliability is a cross-plane property.**

模型只是其中一个变量；context、semantic definition、identity、authority、policy、data state 和 verification 共同决定最终可靠性。

## 当前总模型

~~~mermaid
flowchart TB
    subgraph CONTEXT[Shared Context Plane]
        ID[Identity]
        GRAPH[Relationships]
        TRUST[Trust / Freshness]
        AUTH[Authority]
    end

    subgraph AGENT[Agent Plane]
        PLAN[Interpret / Plan]
        DEC[Human Decision]
    end

    subgraph EXEC[Execution Planes]
        SEM[Semantic Runtime]
        POLICY[Policy Runtime]
        TOOLS[Warehouse / Git / SaaS]
    end

    subgraph REL[Reliability]
        VERIFY[Independent Verification]
        AUDIT[Decision Audit]
    end

    CONTEXT --> AGENT
    AGENT --> EXEC
    EXEC --> REL
    REL -.feedback.-> CONTEXT
~~~

## Next

建议进入：

> **Phase 4 — Comparative Architecture**

不比较产品“谁最好”，而比较各条技术路线分别把 Identity、Graph、Context Lifecycle、Authority、Semantic Execution、Policy、Agent Activation、Write-back 与 Audit 放在哪里。
