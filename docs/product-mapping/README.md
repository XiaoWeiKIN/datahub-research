# Phase 2 — DataHub Product Mapping

第一阶段建立了厂商无关的 Context Layer Reference Architecture。

第二阶段逐项检查：

> **DataHub 今天到底实现到了哪里？**

## Mapping

1. [Reference Architecture → DataHub Current Product](01-reference-architecture-to-datahub.md)
2. [Metadata Model as Context Substrate](02-metadata-model-as-context-substrate.md)
3. [Context Lifecycle Product Mapping](03-context-lifecycle.md)
4. [Agent Governance Product Mapping](04-agent-governance.md)

## 下一步

5. **OSS vs Cloud Context Architecture**

## 当前产品结构

~~~mermaid
flowchart TB
    SUB[Metadata / Context Substrate]
    LIFE[Context Lifecycle]
    REG[Agent Registry]
    RUN[Agent Runtime]
    EXT[External Execution]

    SUB --> LIFE
    SUB --> REG
    LIFE --> RUN
    REG --> RUN
    RUN --> EXT
~~~

当前判断：

- Metadata substrate：成熟；
- Context lifecycle：Public Beta；
- Agent Registry：已落地；
- Custom Agent runtime：Private Beta；
- Agent runtime identity / delegation：仍在演进；
- runtime data authorization：外部职责。
