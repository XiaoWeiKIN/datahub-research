# Phase 3 — Concrete Case Studies

前两阶段分别回答：

- **Phase 1:** Context Layer 应该是什么？
- **Phase 2:** DataHub 今天实现到了哪里？

Phase 3 用真实类型的 Agent 任务验证整个架构。

## Case Studies

1. [Analytics Agent — Enterprise Customer Net Revenue](01-analytics-agent-net-revenue.md)
2. [Schema Change Incident Agent](02-schema-change-incident-agent.md)

## Coverage

~~~mermaid
flowchart LR
    C1[Case 01<br/>Read-heavy Analytics]
    C2[Case 02<br/>Operational Repair]
    C3[Next<br/>Governance / PII]

    C1 --> C2 --> C3
~~~

### Case 01

验证：

~~~text
Intent
-> Context
-> Semantic
-> Policy
-> Query
-> Evidence
-> Answer
~~~

### Case 02

验证：

~~~text
Incident
-> Fresh Context
-> Lineage Blast Radius
-> Ownership
-> Repair Plan
-> Human Decision
-> PR / CI
-> Independent Verification
-> Context Write-back
~~~

## 方法

每个 case 区分：

- **Synthetic Fixture**
- **DataHub Current Capability**
- **Reference Architecture Ideal**
- **Gap / Failure Test**

## 下一 Case

3. **Governance / PII Agent**

重点验证 Context Visibility 与 Runtime Authorization 的边界。
