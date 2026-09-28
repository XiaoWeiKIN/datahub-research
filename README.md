# DataHub Research

研究 DataHub 的**架构思想、设计哲学，以及它如何从 Metadata Platform 演化为 AI 时代的 Context Layer / Context Platform**。

官方文档翻译是学习手段，不是研究终点。

> DataHub: https://datahub.com/  
> Documentation: https://docs.datahub.com/

## 第一阶段：理论主线

1. **[01 — Why Context Layer?](docs/research/01-why-context-layer.md)**
2. **[02 — Context Graph vs Knowledge Graph](docs/research/02-context-graph-vs-knowledge-graph.md)**
3. **[03 — Context Layer vs Semantic Layer](docs/research/03-context-layer-vs-semantic-layer.md)**
4. **[04 — Context Freshness & Provenance](docs/research/04-context-freshness-and-provenance.md)**
5. **[05 — Agent Read / Write Context](docs/research/05-agent-read-write-context.md)**
6. **[06 — Human + Agent Shared Truth Plane](docs/research/06-human-agent-shared-truth-plane.md)**
7. **[07 — Context Layer Reference Architecture](docs/research/07-context-layer-reference-architecture.md)**
8. **[08 — Context Platform Failure Modes](docs/research/08-context-platform-failure-modes.md)**
9. **[09 — DataHub Design Philosophy — Synthesis](docs/research/09-datahub-design-philosophy-synthesis.md)**

## Final Thesis

> **AI Agent 的可靠性上限，不只由模型决定，而由它所处的企业认知基础设施决定。DataHub 最值得研究的地方，是它长期把 metadata 设计成实时、关系化、可治理、可编程的基础设施；这使它能够自然扩展为一种面向 Human 和 Agent 的 Context Platform。Context Platform 的真正任务不是给模型更多信息，而是持续维护一个可被组织信任、解释、修正和复用的企业现实模型。**

## DataHub 的长期设计主线

~~~mermaid
flowchart LR
    CAT[Data Catalog]
    GRAPH[Metadata Graph]
    ACTIVE[Active Metadata]
    GOV[Governed Metadata Platform]
    CONTEXT[Context Graph]
    AGENT[Agent-ready Context Platform]

    CAT --> GRAPH --> ACTIVE --> GOV --> CONTEXT --> AGENT
~~~

当前研究认为：AI 没有让传统 metadata architecture 失效，反而提高了它的价值。

- lineage → provenance / dependency reasoning
- ownership → authority
- quality → trust
- usage → observed context
- event-driven metadata → freshness
- graph → context traversal
- governance → agent grounding
- API/MCP → machine activation

## 当前参考架构

~~~mermaid
flowchart TB
    subgraph REALITY[Enterprise Reality]
        DATA[Data Systems]
        DOCS[Knowledge]
        PEOPLE[People / Org]
        OPS[Operational Signals]
    end

    subgraph CONTEXT[Epistemic Context Infrastructure]
        OBS[Observe]
        ID[Identity]
        ASSERT[Assertions]
        GRAPH[Graph]
        TRUST[Trust / Authority / Provenance]
        REC[Reconcile / Invalidate]
        PUB[Publish]
        PROJ[Project / Retrieve]
    end

    subgraph CONSUMERS[Consumers]
        HUMAN[Humans]
        AGENTS[Agents]
        APPS[Applications]
    end

    REALITY --> OBS --> ID --> ASSERT --> GRAPH
    GRAPH --> TRUST --> REC --> PUB --> PROJ
    PROJ --> CONSUMERS
    CONSUMERS -.feedback.-> ASSERT
~~~

## 核心设计原则

1. **Metadata is infrastructure, not documentation.**
2. **Relationships are first-class.**
3. **Context must stay active and temporally aligned with reality.**
4. **Governance is context, not post-processing.**
5. **Capture authoritative context near the source when possible.**
6. **Humans and machines should share a governed substrate, not necessarily the same view.**
7. **Generated context is proposal, not truth.**
8. **Authority should be federated and typed.**
9. **Unknown / stale / conflicted must remain explicit states.**
10. **Deterministic systems should constrain probabilistic Agent reasoning.**
11. **Context must be continuously reconciled, not authored once.**
12. **Protocol interoperability matters, but trusted substrate matters more.**

## 五个 Plane 的边界

| Plane | 主要问题 |
|---|---|
| Context / Epistemic | 知道什么？为什么相信？当前是否适用？ |
| Semantic | 怎么算？ |
| Identity / Policy | 当前主体是否允许？ |
| Data / Execution | 数据在哪里、如何真实执行？ |
| Agent | 如何解释、规划、调用工具和行动？ |

## 第一阶段之后

理论研究已形成闭环。下一阶段建议从三条路继续：

### Product Mapping

把 Reference Architecture 映射到 DataHub 当前真实能力：

- implemented
- partial
- roadmap
- unclear / external dependency

### Comparative Architecture

比较不同系统各自解决哪一层责任，不做总体排名。

### Case Studies

用具体 Agent 任务验证架构，例如：

> “给我过去 90 天 Enterprise Customer Net Revenue，并解释为什么这个数字可信。”

完整追踪 context → semantic → policy → query → evidence → answer → audit。
