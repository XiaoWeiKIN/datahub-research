# Phase 2 — DataHub Product Mapping

第一阶段建立了厂商无关的 Context Layer Reference Architecture。

第二阶段不再扩展理论，而是逐项检查：

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

## 下一步

3. **Context Lifecycle Product Mapping**
4. Agent Governance Product Mapping
5. OSS vs Cloud Context Architecture

## 研究规则

- 时间截点：**2026-09-28**
- 优先使用当前 DataHub Docs 和 2026 release notes。
- 区分 **DataHub Core (OSS)** 与 **DataHub Cloud**。
- 不把 blog 中的架构愿景自动视为已实现产品能力。
- “有相关 feature”不等于“完整实现了 Reference Architecture primitive”。
- 无法从当前官方材料确认的能力，标记为 **Unclear**，而不是猜测。
