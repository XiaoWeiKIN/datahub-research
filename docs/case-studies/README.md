# Phase 3 — Concrete Case Studies

前两阶段分别回答：

- **Phase 1:** Context Layer 应该是什么？
- **Phase 2:** DataHub 今天实现到了哪里？

Phase 3 不再扩概念，而是用真实类型的 Agent 任务验证整个架构。

## Case Studies

1. [Analytics Agent — Enterprise Customer Net Revenue](01-analytics-agent-net-revenue.md)

## 方法

每个 case 都区分：

- **Synthetic Fixture**：为了推演而设定的业务定义 / 表 / 指标，不声称来自真实企业；
- **DataHub Current Capability**：截至快照日期有官方资料支持的能力；
- **Reference Architecture Ideal**：我们认为 production Context Layer 更理想的设计；
- **Gap**：DataHub 当前产品与 Reference Architecture 的差距。

每个 case 尽量完整走过：

~~~text
Intent
-> Context
-> Authority
-> Freshness / Quality
-> Semantic
-> Policy
-> Data Execution
-> Evidence
-> Answer
-> Write-back
-> Audit
~~~

## 目标

Case Study 不追求证明“DataHub 能自动解决一切”。

真正要检验的是：

> **Context / Semantic / Policy / Data / Agent 五个 Plane 的边界是否足以解释一个 production Agent 的正确性。**
