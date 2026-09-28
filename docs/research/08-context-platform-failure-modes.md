# 08 — Context Platform Failure Modes

## 用失败场景反向检验 Context Layer Reference Architecture

**Status:** 第一版  
**Focus:** reliability / security / epistemic failure / governance failure / blast radius  
**Updated:** 2026-09-28

---

# 0. 为什么要从 Failure Modes 反推架构？

Reference Architecture 很容易在“正常路径”下看起来合理：

~~~text
source
-> ingest
-> context graph
-> validate
-> publish
-> retrieve
-> agent
~~~

真正决定 Context Platform 是否可靠的是异常路径：

- source 沉默；
- context stale；
- 定义冲突；
- identity 错配；
- Agent 把错误写回；
- policy 与 context 不一致；
- publication 错误扩大到多个 consumer。

所以这一篇采用：

> **failure-oriented design**

不是问：

> 系统有哪些 feature？

而是问：

> **当某一个假设失效时，系统会如何失败？失败能不能被检测、限制、解释和恢复？**

---

# 1. Failure Taxonomy

我们把 Context Platform 的失败分成六组：

~~~mermaid
mindmap
  root((Context Platform Failures))
    Freshness
      Stale Context
      Source Silence
      Propagation Lag
    Semantics
      Conflicting Truth
      Semantic Drift
      Wrong Applicability
    Identity
      Entity Collision
      Duplicate Identity
      Broken Resolution
    Trust
      False Provenance
      Authority Collapse
      Unknown as False
    Agent
      Context Poisoning
      Self Reinforcement
      Memory Promotion
    Platform
      Context Islands
      Over Centralization
      Blast Radius
      Policy Mismatch
~~~

这些 failure 不是独立的。

真正危险的事故通常是多个 failure chain 在一起。

---

# 2. Failure 1 — Stale-but-Confident Context

## 场景

业务定义已经改变，但旧文档仍是 published。

Agent：

1. 正确检索；
2. 正确读取；
3. 正确推理；
4. 得到错误答案。

最危险之处：

> **没有明显异常。**

DataHub 自己把 stale context 视为生产 Agent 的典型失败：schema、metric 或文档变化后，Agent 仍可能根据旧 context 自信回答。

## Root Cause

- freshness 只看 `updated_at`；
- 没有 dependency invalidation；
- human-authored context 没有 revalidation；
- source change 没触发影响分析。

## Required Controls

~~~text
valid_time
+ source health
+ dependency graph
+ stale state
+ revalidation policy
~~~

## Detection Signal

- source asset version > context evidence version；
- dependency changed after last validation；
- eval regression；
- declared context 与 observed behavior divergence。

---

# 3. Failure 2 — Source Silence Interpreted as Stability

## 场景

lineage connector 已停止运行。

Context Graph 仍显示：

~~~text
A -> B -> C
~~~

系统错误地解释成：

> lineage 没变化。

真实状态其实是：

> unknown。

## Root Cause

系统只有：

~~~text
last_value
~~~

没有：

~~~text
source_health
expected_observation_frequency
~~~

## Required Control

~~~text
HEALTHY
LAGGING
SILENT
FAILED
PARTIAL
~~~

Context assertion 应继承 source health。

原则：

> **No new evidence != evidence of no change.**

---

# 4. Failure 3 — Eventual Consistency Hidden from Agent

DataHub 官方支持材料明确说明 DataHub 是 eventually consistent：metadata ingestion acknowledgement 后，还有异步 indexing、relationship/lineage 更新和 graph processing；大规模 ingestion 时 visibility 可以滞后数小时。

这意味着：

~~~text
write accepted
!=
context visible
~~~

## Failure

Agent：

1. 修改 metadata；
2. API 返回 success；
3. 立即检索；
4. 读到旧状态；
5. 基于旧状态继续行动。

## Required Controls

- mutation version / operation id；
- read-after-write consistency strategy；
- independent read-back；
- minimum visible version；
- propagation status；
- timeout / retry policy。

原则：

> **Agent-reported write success is not proof of state convergence.**

---

# 5. Failure 4 — Conflicting Truth Hidden by Ranking

## 场景

Finance：

~~~text
Active Customer = paid in last 90 days
~~~

Growth：

~~~text
Active Customer = logged in last 30 days
~~~

Vector retrieval 因相似度选择 Finance 定义。

Agent 不知道还存在冲突。

## Root Cause

retrieval 把：

> ranking

误当成：

> truth resolution。

## Required Controls

Context Retrieval 必须先检查：

~~~text
same concept
+ overlapping scope
+ multiple active assertions
~~~

然后返回：

~~~text
CONFLICTED
~~~

而不是自动选择第一名。

## Resolution Inputs

- task domain；
- purpose；
- authority；
- valid time；
- user / principal；
- publication state。

---

# 6. Failure 5 — Semantic Drift

## 场景

组织明确声明：

~~~text
Net Revenue -> Metric A
~~~

但实际 analysts 与 dashboards 已经大量使用 Metric B。

两者都没有技术错误。

这是：

> declared reality 与 observed reality 分叉。

## Risk

Agent 可能：

- 继续使用过时规范；
- 或把 observed behavior 自动当成新 truth。

两边都可能错。

## Required Control

~~~mermaid
flowchart LR
    DECL[Declared]
    OBS[Observed]
    DIFF[Drift Detection]
    SME[Authority Review]
    RES[Resolved Context]

    DECL --> DIFF
    OBS --> DIFF
    DIFF --> SME --> RES
~~~

原则：

> **Observed behavior is evidence, not automatic authority.**

---

# 7. Failure 6 — Wrong Applicability

Context 本身完全正确。

但被用在错误场景。

例如：

~~~text
Revenue Definition:
valid for US GAAP reporting
~~~

Agent 用它回答 APAC tax reporting。

这是：

> truth correct, applicability wrong。

## Required Assertion Fields

- domain；
- jurisdiction；
- audience；
- purpose；
- time interval；
- environment；
- business process。

所以 context trust 不只需要：

> Is it true?

还要：

> **Is it true for this task?**

---

# 8. Failure 7 — Entity Identity Collision

## 场景

系统自动把：

~~~text
customer
customer_360
customer_master
~~~

合并成一个 canonical entity。

其实它们是不同业务对象。

结果：

- lineage 错误；
- ownership 错误；
- business definitions 混合；
- Agent retrieval 被污染。

## Root Cause

entity resolution 过度依赖：

- string similarity；
- embedding similarity；
- LLM judgment。

## Required Controls

- deterministic identity rules；
- source namespace；
- candidate equivalence；
- human-approved merges；
- merge history；
- reversible split。

原则：

> **Similarity proposes identity; authority confirms identity.**

---

# 9. Failure 8 — Duplicate Identity / Fragmented Entity

反方向问题：

同一真实对象在 graph 中出现多个身份。

例如：

~~~text
dbt:model.orders
snowflake:PROD.ANALYTICS.ORDERS
semantic:orders
~~~

没有被正确连接。

结果：

- usage 分散；
- ownership 分散；
- quality signal 不完整；
- retrieval 认为它们是三个资产。

因此 identity failure 有两种：

~~~text
false merge
false split
~~~

都必须支持审计与修复。

---

# 10. Failure 9 — Authority Collapse

## 场景

Context Graph 里同时存在：

- Finance-approved definition；
- AI-generated summary；
- inferred query pattern；
- random Confluence page。

retrieval 把它们当同等级 evidence。

## 结果

Agent 可能选择：

> 内容更详细、embedding 更相似的 AI summary

而不是 Finance authority。

## Required Controls

每条 assertion 都应包含：

~~~text
authority_type
authority_scope
producer_type
publication_state
review_state
~~~

原则：

> **Information richness must never override authority silently.**

---

# 11. Failure 10 — False Provenance

## 场景

系统显示：

> Source: Finance Policy

但真实链路是：

~~~text
Finance Policy
-> old LLM summary
-> another Agent rewrite
-> published context
~~~

Agent 看到的 provenance 被过度压缩。

## Risk

用户以为它直接来自 system of record。

## Required Control

完整 derivation chain：

~~~mermaid
flowchart LR
    SRC[Source]
    A1[Agent Derivation]
    A2[Summary]
    REV[Human Review]
    PUB[Published Assertion]

    SRC --> A1 --> A2 --> REV --> PUB
~~~

Provenance UI / API 不应该只展示 root source。

还要展示：

> transformations between root and current assertion.

---

# 12. Failure 11 — Unknown Collapses into False

非常常见。

系统没有证据说明 dataset 有 downstream consumer。

API 返回：

~~~text
downstream = []
~~~

Agent解释：

> 没有 downstream。

真实含义可能是：

> lineage source unavailable。

正确状态应该是：

~~~text
known_empty
unknown
partial
~~~

三者不能合并。

原则：

> **Absence of evidence must be machine-readable.**

---

# 13. Failure 12 — Context Poisoning

这是 Read/Write Context 架构中最重要的安全风险之一。

攻击链：

~~~mermaid
flowchart LR
    INPUT[Untrusted Input]
    AG1[Agent]
    PROP[Context Write]
    GRAPH[Shared Context]
    AG2[Future Agents]

    INPUT --> AG1 --> PROP --> GRAPH --> AG2
~~~

一次 prompt injection / malicious document 可以升级成 persistent organizational memory poisoning。

MITRE ATLAS 已经把 AI Agent Context Poisoning 作为 Agent persistence 攻击技术；OWASP 也把持久 memory/context 视为重要 Agentic AI attack surface。

## Required Controls

- proposal/published boundary；
- source trust labels；
- no direct private-memory promotion；
- human / policy authority；
- evidence gates；
- rollback；
- provenance；
- suspicious producer isolation。

---

# 14. Failure 13 — Agent Self-Reinforcement

不是恶意攻击，也可能发生。

~~~text
Agent A generates X
-> X published
-> Agent B reads X
-> B creates Y citing X
-> Agent C reads X + Y
-> confidence increases
~~~

看起来 evidence 越来越多，但全部来自同一个生成源。

## Required Controls

provenance 需要计算：

> evidence independence

至少区分：

- system-of-record root；
- human authority root；
- runtime evidence；
- Agent-derived evidence。

原则：

> **More derived assertions do not create more independent evidence.**

---

# 15. Failure 14 — Memory Promotion Leak

Agent private memory：

> “用户经常喜欢用 table X。”

如果自动 promotion 到 enterprise graph：

> “table X is canonical。”

这是 trust-domain leak。

正确路径：

~~~mermaid
flowchart LR
    MEM[Private Memory]
    OBS[Observation]
    PROP[Proposal]
    AUTH[Validation]
    CTX[Shared Context]

    MEM --> OBS --> PROP --> AUTH --> CTX
~~~

原则：

> **Private memory cannot become shared truth without promotion gates.**

---

# 16. Failure 15 — Publication Blast Radius

Shared Context 的优势：

> one fix benefits many consumers.

对应风险：

> one bad publish harms many consumers.

DataHub 当前文档建议：

- auto-publish 默认关闭；
- 先建立 eval；
- 从少量高置信 context / 单个 domain 开始；
- 发布前用 Ask DataHub 测试；
- published 才对 Agent 可见。

这本质上就是：

> blast-radius-aware rollout.

## Required Controls

- domain-scoped publication；
- incremental rollout；
- canary agents；
- eval regression；
- publish history；
- instant unpublish / rollback。

---

# 17. Failure 16 — Bad Eval = False Confidence

Evals 本身也可能错。

DataHub 文档明确指出，一个看起来正确的 context document 也可能因为 pass criteria 过于具体而 eval fail。

反方向也一样：

> weak eval can pass bad context.

所以：

~~~text
Eval Pass != Truth
~~~

## Failure Types

- golden question coverage 太窄；
- trap assets 不完整；
- expected SQL 自身错误；
- eval 与真实 workload 不匹配；
- eval overfits known examples。

## Required Control

- eval versioning；
- domain ownership；
- production feedback；
- diverse question set；
- periodic eval review。

---

# 18. Failure 17 — Context Islands

每个 Agent 自己建立：

- vector DB；
- semantic mapping；
- copied documentation；
- prompt rules。

结果：

~~~mermaid
flowchart TB
    REAL[Enterprise Reality]
    C1[Context A]
    C2[Context B]
    C3[Context C]

    REAL --> C1
    REAL --> C2
    REAL --> C3
~~~

各自 drift。

DataHub 当前反复把 fragmentation 作为主要 context failure：context 分散、不可访问、未验证、过期。

## Required Control

shared substrate + scoped projection。

而不是：

> one context database per agent.

---

# 19. Failure 18 — Over-centralization

解决 Context Islands 时，很容易走向另一个极端：

> 一个中央 Context Team 管所有 business truth。

这会制造：

- review bottleneck；
- stale definitions；
- domain expertise 丢失；
- platform team 成为 semantic gatekeeper。

DataHub 当前 Context Ownership 模型也明确认为 technical、business、governance context 需要不同 custodians。

所以正确模式：

> **centralized infrastructure + federated authority**

---

# 20. Failure 19 — Context / Policy Mismatch

Context Graph 说：

~~~text
dataset X contains PII
~~~

但 runtime enforcement：

~~~text
allows unrestricted access
~~~

或者反过来：

Context 认为 public，但 policy engine 已升级为 confidential。

这是一种 cross-plane drift。

## Required Control

~~~mermaid
flowchart LR
    CTX[Context Classification]
    POLICY[Runtime Policy]
    DIFF[Drift Detector]
    FIX[Reconcile]

    CTX --> DIFF
    POLICY --> DIFF
    DIFF --> FIX
~~~

原则：

> **Context informs policy, but differences between context and enforcement must themselves be observable.**

---

# 21. Failure 20 — Semantic / Context Mismatch

Context Graph：

> Net Revenue = Finance-certified metric.

Semantic Runtime：

> metric definition 已改，但 context 未同步。

或者：

Context 更新了 business definition，但 semantic model 仍旧。

结果：

> Agent 选对 metric 名字，却算出旧逻辑。

因此：

~~~text
context version
semantic model version
~~~

必须能关联。

理想情况下 Agent answer trace 要记录两者。

---

# 22. Failure 21 — Agent Scope Leak

Agent 本应只看到 Marketing View。

但：

- MCP server 配错；
- plugin 使用 shared service account；
- retrieval 跳过 view scope；
- tool call 直接访问底层系统。

结果：

> Context scope 与 execution scope 不一致。

DataHub 当前 Agents 支持 View scope，Copilot 集成也推荐 OAuth，让交互式用户以自己的 DataHub identity 访问，而不是共享 PAT。

## Required Control

Agent request 应携带：

~~~text
agent_id
principal
delegation
purpose
view_scope
~~~

并且 context retrieval 与 runtime access 都要验证这些信息。

---

# 23. Failure 22 — Control Plane Becomes Architecture Blob

Context Platform 逐步承担：

- graph；
- IAM；
- policy；
- SQL execution；
- agent runtime；
- secrets；
- model serving。

最终：

> everything depends on one platform.

风险：

- replaceability 差；
- security boundary 混乱；
- duplicated capability；
- scaling model 混杂。

防护原则仍然是五 Plane：

~~~text
Context = know / trust
Semantic = compute
Policy = allow
Data = execute
Agent = reason / act
~~~

---

# 24. Failure 23 — Silent Partial Coverage

Context Graph 看起来很完整，但 connector 只覆盖 70% assets。

Agent 不知道剩下 30% 是：

> missing from graph

而不是：

> nonexistent.

这和 Unknown-as-False 是同类问题，但发生在 coverage 层。

Context retrieval 应暴露：

~~~text
coverage
source completeness
known gaps
~~~

尤其 lineage / usage / quality。

---

# 25. Failure 24 — False Canonicalization from Usage

Context Intelligence 从 query logs 发现：

> table X 最常用。

系统自动推断：

> X is canonical.

但最常用可能因为：

- legacy habit；
- old dashboard；
- bad copy-paste；
- easier permissions；
- undocumented workaround。

所以 observed context 必须区分：

~~~text
popular
trusted
authoritative
~~~

三者不能互推。

---

# 26. Failure 25 — Review Fatigue

Proposal model也会失败。

如果 Context Curator 每天产生几千个 proposals：

> SME 不再认真审。

最终 Approve 变成机械操作。

## Controls

- risk-based review；
- blast-radius prioritization；
- aggregate proposals；
- evidence summary；
- auto-approve only low-risk bounded changes；
- measure reviewer disagreement / correction rate。

Human-in-the-loop 不是无限资源。

---

# 27. Failure 26 — Rollback Without Dependency Rollback

Context Document 被 unpublish。

但：

- Agent cache 仍有旧内容；
- downstream derived context 未 invalid；
- vector index 未清理；
- semantic mapping 仍引用。

所以 rollback 不能只改 publication flag。

需要：

~~~text
rollback
-> dependency invalidation
-> cache invalidation
-> projection refresh
-> consumer notification
~~~

---

# 28. Failure 27 — No Decision Reproducibility

Agent 给出错误结果。

系统有：

- chat transcript；
- current Context Graph。

但没有：

> 当时 Agent 读取的 context version。

由于 graph 已经变化，无法复现。

## Required Audit

~~~text
decision trace
├── context assertion ids + versions
├── retrieval trace
├── semantic model version
├── policy decision
├── agent/model version
├── tool outputs
└── human decisions
~~~

原则：

> **Audit must reference historical context, not current context.**

---

# 29. Failure Chains 比单点 Failure 更重要

一个真实事故可能是：

~~~mermaid
flowchart LR
    SILENT[Connector Silent]
    STALE[Stale Context]
    RET[Retriever Selects It]
    WRITE[Agent Writes Derived Summary]
    PUB[Weak Eval Auto-publishes]
    MANY[Many Agents Consume]

    SILENT --> STALE --> RET --> WRITE --> PUB --> MANY
~~~

单个组件看都没有 crash。

但最终形成严重 semantic incident。

所以 Context Platform reliability 需要：

> end-to-end failure chain analysis.

---

# 30. Failure Mode -> Required Architecture Mapping

| Failure | Required Primitive |
|---|---|
| Stale context | temporal validity + invalidation |
| Source silence | source health + unknown state |
| Eventual consistency | versioned read-back |
| Conflict hidden | conflict-aware retrieval |
| Semantic drift | reconciliation |
| Wrong applicability | scoped assertions |
| Identity collision | governed entity resolution |
| Authority collapse | typed authority |
| False provenance | full derivation chain |
| Context poisoning | proposal/publish boundary |
| Self reinforcement | evidence root tracking |
| Blast radius | scoped publication + rollback |
| Context islands | shared substrate |
| Over-centralization | federated authority |
| Policy mismatch | cross-plane drift detection |
| Review fatigue | risk-based promotion |
| Audit failure | historical decision provenance |

如果 Reference Architecture 缺少这些 primitive，对应 failure 通常不会被可靠处理。

---

# 31. Reliability Principle 1 — Fail Closed on Epistemic Uncertainty

在高风险任务中：

~~~text
UNKNOWN
SUSPECT
CONFLICTED
STALE
~~~

不应该被 Agent 自动转成：

> probably fine.

更合理：

- ask human；
- retrieve independent evidence；
- lower action scope；
- refuse authoritative action；
- mark answer caveat。

这叫：

> **epistemic fail-closed**

不代表所有问答都停止，而是高风险 action 不应建立在未解决的 context ambiguity 上。

---

# 32. Reliability Principle 2 — Separate Data Availability from Context Trust

Source reachable 不代表 context trustworthy。

Context trustworthy 也不代表 source currently available。

需要两个 health dimensions：

~~~text
Operational Health
Epistemic Health
~~~

例如：

| Operational | Epistemic | 含义 |
|---|---|---|
| Healthy | Valid | 正常 |
| Healthy | Conflicted | 数据可用但意义有冲突 |
| Failed | Previously Valid | context 可能进入 stale window |
| Healthy | Unknown | source 可访问但知识未建立 |

---

# 33. Reliability Principle 3 — Measure Context Debt

企业已经有 technical debt / data debt。

Context Platform 还需要：

> context debt.

可能指标：

- stale published documents；
- assertions without authority；
- assertions without provenance；
- unresolved conflicts；
- silent sources；
- unowned domains；
- failed evals；
- proposed-but-unreviewed context；
- duplicated identities；
- agents using unpublished/private context stores。

这类指标比“catalog documentation coverage”更贴近 Agent readiness。

---

# 34. Reliability Principle 4 — Blast Radius Must Be Explicit

每条 context 应该知道自己的 consumer / dependency scope。

例如：

~~~text
local workflow
domain
enterprise-wide
external-facing
regulated
~~~

发布权限与 eval 强度应该随 blast radius 增长。

原则：

> **Publication risk is a function of downstream consumers, not document size.**

---

# 35. Reliability Principle 5 — Recovery Is a First-class Feature

不能只设计：

> how context becomes published.

还要设计：

> how context becomes untrusted again.

恢复能力至少包括：

- unpublish；
- supersede；
- invalidate；
- rollback；
- consumer notification；
- cache purge；
- dependency re-evaluation；
- incident trace。

---

# 36. 对 DataHub 当前设计的验证

DataHub 当前已有一些与这些 failure 对应的机制：

### Context fragmentation

统一 Context Graph / MCP activation。

### Unvalidated generated context

默认 auto-publish 关闭；generated context 进入 proposal，由 Data Expert / SME review。

### Bad context quality

metadata eval、must-reference、must-not-reference、Ask DataHub testing。

### Publication blast radius

官方建议从小范围、高置信文档和单 domain 开始发布。

### Human-vs-Agent precedence

human-edited context 在 regeneration 中保留并优先。

### Agent scope

Agents 支持 scoped View。

### Runtime context lag

官方明确承认 ingestion / indexing 的 eventual consistency。

这些都说明 DataHub 当前设计已经触及 Context Platform 的真实 reliability problem，而不只是“metadata + AI search”。

---

# 37. 仍然需要观察的 DataHub 问题

我们不能从文档直接确认以下能力是否形成统一通用机制：

1. assertion-level temporal validity；
2. generalized source-health propagation；
3. conflict-aware retrieval；
4. entity merge/split governance；
5. multi-dimensional typed authority；
6. derived-evidence independence；
7. publication rollback dependency propagation；
8. historical context snapshot for decision replay；
9. cross-plane policy/context drift detection；
10. explicit context debt metrics。

这些更适合作为后续产品观察 checklist。

---

# 38. Context Incident Model

未来 Context Platform 应该像 Data Platform 一样拥有 incident 类型。

例如：

~~~text
ContextIncident
├── incident_id
├── affected assertions
├── affected consumers / agents
├── failure type
├── first bad version
├── source / producer
├── blast radius
├── mitigation
├── rollback
├── root cause
└── prevention
~~~

Context incident 例子：

> Finance Revenue definition v18 被错误 auto-publish，影响 14 个 Agents，在 2 小时内产生 37 次错误回答。

这种 incident model 会让 Context Reliability 真正成为可运营能力。

---

# 39. Context SRE

进一步推导：

如果 Context Layer 是生产 AI 基础设施，就需要类似 SRE 的 discipline：

~~~text
Context Reliability Engineering
~~~

关注：

- freshness SLO；
- propagation latency；
- coverage；
- source health；
- conflict rate；
- eval pass rate；
- invalidation latency；
- rollback time；
- context incident MTTR；
- decision reproducibility。

这比“Data Catalog adoption”是完全不同的运营模型。

---

# 40. 最终判断

Context Platform 的价值和风险来自同一个属性：

> **它是共享的。**

共享意味着：

- 一次治理可以服务多个 Agent；
- 一个定义可以统一人和机器；
- provenance 可以统一审计；
- context engineering 不再重复建设。

也意味着：

- 一个错误可以广泛传播；
- 一个 poisoned assertion 可以长期存在；
- 一个 identity merge 可以污染整个 graph；
- 一个弱 authority model 可以让 AI-generated text 伪装成企业事实。

因此 production Context Platform 的核心能力不应该只按：

~~~text
ingest
search
graph
MCP
~~~

衡量。

更应该按：

~~~text
detect
qualify
validate
scope
publish
invalidate
reconcile
rollback
audit
~~~

衡量。

---

# 41. 下一步

下一篇：

## 09 — DataHub Design Philosophy — Synthesis

将不再新增架构组件。

而是回答整个项目最初的问题：

> **为什么在 AI 时代值得学习 DataHub？**

会把研究压缩成几个长期设计哲学：

- Metadata as infrastructure
- Relationships over inventory
- Active over static
- Truth as governed assertions
- Human + machine symmetry
- Context as shared substrate
- Authority is federated
- AI should consume deterministic structure where possible
- Context must be continuously reconciled
- Agent contribution must not collapse into Agent authority

---

# Sources

## DataHub

- DataHub, **The Five Context Problems Data Teams Face**  
  https://datahub.com/blog/common-context-problems-data-teams-face/

- DataHub, **Continuous Context: Why Your AI Documentation Is Already Lying to You**  
  https://datahub.com/blog/continuous-context/

- DataHub, **The Data Engineer's Guide to Context Engineering**  
  https://datahub.com/blog/context-engineering/

- DataHub Docs, **Configure Context Generation**  
  https://docs.datahub.com/docs/managed-datahub/context/configure-context-generation

- DataHub Docs, **Validate Context Proposals**  
  https://docs.datahub.com/docs/managed-datahub/context/review-context-proposals

- DataHub Docs, **Activate Context**  
  https://docs.datahub.com/docs/managed-datahub/context/activate-context

- DataHub Docs, **Agents**  
  https://docs.datahub.com/docs/features/feature-guides/agents

- DataHub Support, **Large Dataset Ingestion Lag and Eventual Consistency**  
  https://support.datahub.com/hc/en-us/articles/42653624258459-Large-Dataset-Ingestion-Lag-and-Eventual-Consistency

## Security

- MITRE ATLAS, **AI Agent Context Poisoning**  
  https://atlas.mitre.org/techniques/AML.T0080/

- OWASP GenAI Security Project, **Memory Is a Feature. It Is Also an Attack Surface**  
  https://genai.owasp.org/2026/05/13/memory-is-a-feature-it-is-also-an-attack-surface/
