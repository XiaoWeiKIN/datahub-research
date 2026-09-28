# Phase 2 — DataHub Product Mapping

第一阶段建立了厂商无关的 Context Layer Reference Architecture。

第二阶段逐项检查：

> **DataHub 今天到底实现到了哪里？**

## Status Legend

| Status | Meaning |
|---|---|
| **Implemented** | 官方当前文档明确存在，可作为现有产品能力 |
| **Beta** | 已实现，但处于 Public / Private Beta |
| **Partial** | 只覆盖 Reference Architecture 的一部分 |
| **External** | DataHub 提供 context / integration，但核心职责由外部系统执行 |
| **Unclear** | 官方材料不足以证明存在一个统一、通用的实现 |
| **Roadmap** | 官方明确描述为未来方向，而非当前能力 |

## Mapping

1. [Reference Architecture → DataHub Current Product](01-reference-architecture-to-datahub.md)
2. [Metadata Model as Context Substrate](02-metadata-model-as-context-substrate.md)
3. [Context Lifecycle Product Mapping](03-context-lifecycle.md)

## 下一步

4. **Agent Governance Product Mapping**
5. OSS vs Cloud Context Architecture

## 当前阶段判断

DataHub Product Mapping 目前呈现三层：

~~~mermaid
flowchart TB
    CORE[Metadata Substrate<br/>Entity · Aspect · URN · Graph · Events]
    LIFE[Context Lifecycle<br/>Generate · Eval · Review · Publish · Activate]
    AG[Agent Governance<br/>Registry · Scope · Tasks · Decisions]

    CORE --> LIFE --> AG
~~~

前两层已经有较清晰产品证据。

下一步开始验证 Agent 层。
