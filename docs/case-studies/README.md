# Phase 3 — Concrete Case Studies

前两阶段分别回答：

- **Phase 1:** Context Layer 应该是什么？
- **Phase 2:** DataHub 今天实现到了哪里？

Phase 3 用真实类型的 Agent 任务验证整个架构。

## Case Studies

1. [Analytics Agent — Enterprise Customer Net Revenue](01-analytics-agent-net-revenue.md)
2. [Schema Change Incident Agent](02-schema-change-incident-agent.md)
3. [Governance / PII Agent](03-governance-pii-agent.md)

## Coverage

~~~mermaid
flowchart LR
    C1[Case 01<br/>Analytics]
    C2[Case 02<br/>Operations]
    C3[Case 03<br/>Governance]
    C4[Next<br/>Cross-case Synthesis]

    C1 --> C2 --> C3 --> C4
~~~

### Case 01

测试正确 business meaning / semantic definition。

### Case 02

测试安全行动、Human Decision、独立验证与 write-back。

### Case 03

测试 Context visibility 与 Runtime authorization 的边界。

## 下一篇

4. **Cross-case Architecture Synthesis**

从三个 case 中抽出每次都重复出现的不可约组件。
