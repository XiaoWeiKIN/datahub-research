# 01 — Context Architecture Landscape

## DataHub、Atlan、OpenMetadata、Collibra、dbt Semantic Layer 与 Knowledge Graph 路线到底在解决哪一层问题？

**Snapshot:** 2026-09-29  
**Goal:** Architecture comparison, not product ranking

---

# 0. 当前结论

2026 年一个明显趋势是：

> **Metadata / Catalog vendors 正在向 Agent-facing Context Layer 收敛。**

DataHub、Atlan、OpenMetadata、Collibra 当前都已经支持 MCP，并都在把：

- metadata；
- lineage；
- ownership；
- glossary / business semantics；
- quality；
- governance；

暴露给 AI clients。

但它们不是完全相同的架构。

真正差异在于：

> **它们从哪个历史中心原语出发？**

~~~mermaid
flowchart TB
    DH[DataHub<br/>Metadata Graph / Active Metadata]
    AT[Atlan<br/>Data Catalog / Governance + Enterprise Data Graph]
    OM[OpenMetadata<br/>Unified Metadata Knowledge Graph]
    CO[Collibra<br/>Governance / Operating Model]
    DBT[dbt<br/>Executable Semantic Layer]
    KG[Knowledge Graph<br/>General Graph Semantics]

    DH --> CTX[Agent Context]
    AT --> CTX
    OM --> CTX
    CO --> CTX
    DBT --> CTX
    KG --> CTX
~~~

它们最终都可以向 Agent 提供 context，但“什么是系统中心”不同。

---

# 1. Six Architectural Centers

## DataHub

中心原语：

> **Metadata Graph + Active Metadata + Context Lifecycle**

## Atlan

中心原语：

> **Enterprise Data Graph + Catalog/Governance + Agent-facing Context Layer**

## OpenMetadata

中心原语：

> **Open unified metadata knowledge graph + APIs/MCP**

## Collibra

中心原语：

> **Governance operating model + business assets / workflows / policy / catalog**

## dbt

中心原语：

> **Executable analytics semantics + transformation graph**

## Knowledge Graph / GraphRAG

中心原语：

> **General entity-relation graph + graph reasoning / retrieval**

这些起点决定每个平台天然强在哪种责任。

---

# 2. High-level Responsibility Matrix

| Capability | DataHub | Atlan | OpenMetadata | Collibra | dbt Semantic Layer | Knowledge Graph / GraphRAG |
|---|---|---|---|---|---|---|
| Stable data asset identity | Strong | Strong | Strong | Strong | Strong within dbt project | Model-dependent |
| Metadata graph / lineage | Strong | Strong | Strong | Strong | Strong for transformation graph | Strong, if modeled |
| Business glossary / semantics | Strong | Strong | Strong | Very central | Metrics / dimensions central | Ontology/domain-model dependent |
| Data quality / operational context | Strong | Strong | Strong | Product-dependent / integrated | Tests / freshness | Must integrate externally |
| Human governance workflow | Strong, Cloud richer | Strong | Strong | Very central | Code review / project workflow | Must build / integrate |
| Context generation | DataHub Context Intelligence | Context Agents Studio | AI tooling / Collate agents varies | AI semantic mapping / Copilot / Maestro direction | AI/dev tooling, not central context lifecycle | LLM extraction pipelines |
| Proposal / publication lifecycle | Explicit Context Hub | AI enrichment + existing-value preservation / certification | Metadata CRUD/workflows; no equivalent unified lifecycle found | Workflow-heavy governance | PR/code workflow | Must build |
| MCP | Yes | Yes | Yes | Yes | Yes | Can expose via MCP |
| Agent metadata / governance | Agent Registry + Agents | AI/context tooling; governance surface | MCP + AI SDK; Collate has AI Studio | AI Governance + agent/model ingestion | dbt agents/dev workflows | Must model |
| Semantic query execution | External | External / integrations | External | Data Notebook / external runtimes | Core responsibility | External |
| Runtime data authorization | External / integrate | Atlan access policies + external runtime varies | Catalog RBAC; actual source enforcement external | Strong governance + integrations, runtime still source-dependent | Access controls at semantic layer | External |
| Context temporal / assertion model | Partial | Product-specific | Metadata entity lifecycle | Workflow / asset lifecycle oriented | Model/version oriented | Can model explicitly |
| General graph reasoning | Moderate / metadata-focused | Moderate / metadata-focused | Moderate / metadata-focused | Asset/relationship governance | Limited to semantic/project graph | Core strength |

This table describes architectural emphasis, not absolute feature completeness.

---

# 3. DataHub — Metadata-first Context Platform

DataHub currently describes DataHub Cloud as an enterprise Context Platform that unifies:

- technical metadata；
- business knowledge；
- documentation；
- lineage；
- quality；
- access/policy context；

and exposes them via MCP / APIs / SDKs.

Its strongest architectural continuity is:

~~~text
Metadata Graph
-> Active Metadata
-> Context Graph
-> Context Lifecycle
-> Agent Activation
~~~

Distinctive emphasis:

### Context Hub

AI-generated context is not automatically treated as published truth.

There is an explicit:

~~~text
generate
-> review
-> validate
-> publish
-> activate
~~~

lifecycle.

### Agent Registry

Agents, skills and tools can themselves become governed graph entities.

### Event / Metadata Change Substrate

Context Platform is built on the existing event-driven metadata architecture.

So DataHub's architecture is best understood as:

> **metadata operations expanded into context operations.**

Source:
https://datahub.com/products/context-platform/

---

# 4. Atlan — Converging Very Close to the Same Context-layer Category

Atlan now explicitly calls itself:

> **the context layer for AI**

and says metadata, semantics, lineage and business knowledge come together in its Enterprise Data Graph.

Its MCP server currently exposes governed tools across:

- discovery；
- lineage；
- metadata；
- glossary；
- quality；
- domains；
- knowledge/artifacts；
- lifecycle；

and write tools preview changes and wait for approval before writing.

This is important because Atlan has independently converged on several principles we derived from DataHub:

~~~text
shared graph
+ governed context
+ MCP
+ human approval for writes
+ AI metadata enrichment
~~~

Atlan's Context Agents Studio also generates:

- descriptions；
- READMEs；
- SQL intelligence；

using metadata, lineage, query history, glossary and knowledge files.

A notable governance rule:

> existing values are not overwritten by these context agents.

This is conceptually similar to a “machine contribution must not silently replace authoritative existing context” principle.

Sources:
- https://docs.atlan.com/get-started/what-is-atlan
- https://docs.atlan.com/product/capabilities/atlan-ai/references/mcp-tools
- https://docs.atlan.com/product/capabilities/governance/context-agents-studio/concepts/agents

---

# 5. DataHub vs Atlan — Convergence More Than Category Difference

Architecturally, DataHub and Atlan in 2026 are closer than the traditional:

~~~text
DataHub = metadata platform
Atlan = data catalog
~~~

description suggests.

Both now have:

- connected metadata graph；
- lineage；
- business glossary / semantics；
- governance；
- quality context；
- AI-generated enrichment；
- MCP；
- write operations；
- human approval / certification concepts；
- Human + Agent context consumption。

The difference is better investigated through **lifecycle mechanics and core model**, not category labels.

## DataHub emphasis

More explicit product model around:

~~~text
Context Intelligence
-> Context Hub
-> Context Activation
~~~

and Agent Registry / custom Agents.

## Atlan emphasis

Very strong:

~~~text
catalog / governance
-> Context Agents Studio
-> Enterprise Data Graph
-> MCP
~~~

with rich MCP write-tool approval semantics and asset-level governance.

A future comparative deep dive should therefore compare:

- generated-context state model；
- authority precedence；
- temporal invalidation；
- Agent identity / registry；
- write governance；
- evaluation model；

rather than basic “supports context?” questions.

---

# 6. OpenMetadata — Open Knowledge Graph + MCP Route

OpenMetadata explicitly describes its MCP server as exposing its:

> unified knowledge graph

to AI assistants.

Current MCP / AI SDK tooling includes:

- metadata search；
- semantic search；
- entity details；
- lineage；
- glossary creation；
- metadata patch；
- quality test creation；
- root-cause analysis。

It also supports OAuth / PAT identity propagation into MCP access.

Architectural center:

~~~text
open metadata model
+ unified knowledge graph
+ data quality
+ search
+ MCP / AI SDK
~~~

This makes OpenMetadata structurally close to DataHub Core.

The main difference visible from current public docs is that OpenMetadata strongly exposes the metadata graph and actions to AI, while a DataHub-style dedicated:

~~~text
generated context
-> proposal
-> eval
-> SME publication
-> activation
~~~

product lifecycle is not equally explicit in the reviewed OpenMetadata docs.

That does not mean OpenMetadata cannot implement it.

It means:

> **the currently documented product abstraction is more “AI access to the metadata knowledge graph” than “separate governed Context Publication lifecycle.”**

Sources:
- https://docs.open-metadata.org/v1.12.x/how-to-guides/mcp
- https://docs.open-metadata.org/v1.12.x/api-reference/sdk/ai-sdk

---

# 7. DataHub vs OpenMetadata — Similar Substrate, Different Current Product Abstraction

Both have:

- open metadata models；
- lineage；
- ownership / tags / glossary；
- quality；
- search；
- APIs；
- MCP；
- metadata mutations。

So at substrate level:

~~~text
DataHub Core
≈
OpenMetadata
~~~

in architectural category: both are metadata-first graph/catalog infrastructures.

But at 2026 product-story level:

### DataHub Cloud

puts a dedicated Context lifecycle on top:

~~~text
Context Intelligence
-> Proposal
-> Eval
-> Publish
-> Agent activation
~~~

### OpenMetadata

puts AI / MCP directly on the unified metadata knowledge graph and AI SDK.

This is one of the clearest examples of:

> same substrate class, different orchestration layer.

---

# 8. Collibra — Governance-first Route to Agent Context

Collibra historically centers:

- Business Glossary；
- stewardship；
- workflows；
- privacy；
- governance；
- policy；
- traceability。

Its 2026 platform now also has:

- MCP；
- AI Copilot / Maestro direction；
- semantic mapping；
- AI model / Agent metadata ingestion；
- technical lineage；
- data quality / privacy；
- business + technical asset relations。

The Collibra MCP server lets AI agents:

- discover assets；
- explore lineage；
- query glossary；
- take actions on assets；

through OAuth-governed access.

This route is architecturally distinct because the starting point is:

> **organizational governance model**

rather than primarily:

> metadata graph operations.

Sources:
- https://productresources.collibra.com/docs/collibra/latest/Content/ModelContextProtocol/co_mcp.htm
- https://productresources.collibra.com/docs/collibra/2026.02/Content/BusinessGlossary/to_business-glossary.htm
- https://productresources.collibra.com/docs/collibra/latest/Content/Catalog/IntegrateAIModels/to_integrating-ai-models.htm

---

# 9. Collibra's Natural Strength: Authority / Stewardship Semantics

Our Reference Architecture places large importance on:

- authority；
- ownership；
- publication；
- workflow；
- governance state。

Those are areas where Collibra's historical governance-first model is naturally aligned.

Its Business Glossary is tightly integrated with:

- business asset types；
- domains；
- workflows；
- traceability；
- technical assets。

Semantic Mapping can link:

~~~text
Business Terms / Measures
-> Data Attributes
-> Physical Columns
~~~

using AI-assisted suggestions with human acceptance.

This resembles the:

> machine proposal -> human authority

pattern from a different starting point.

---

# 10. dbt — Semantic-execution-first, Not Broad Context-management-first

dbt Semantic Layer solves a narrower but extremely important problem:

> **How should analytical meaning be computed consistently?**

Its center is:

- models；
- metrics；
- dimensions；
- joins；
- MetricFlow；
- SQL/query execution；
- access control。

The dbt MCP server exposes:

- Semantic Layer；
- metadata discovery；
- lineage；
- freshness；
- SQL execution；
- dbt platform APIs。

So dbt also delivers governed context to Agent clients.

But its context is concentrated around:

> transformation + analytics semantics.

That makes dbt complementary to broad Context Platforms.

~~~mermaid
flowchart LR
    CTX[DataHub / Atlan / Catalog Context]
    DBT[dbt Semantic Layer]
    AG[Agent]
    WH[Warehouse]

    CTX --> AG
    AG --> DBT --> WH
~~~

Principle:

> **Context Platform helps select applicable trusted meaning; dbt makes analytical meaning executable.**

Sources:
- https://docs.getdbt.com/docs/use-dbt-semantic-layer/dbt-sl
- https://docs.getdbt.com/docs/dbt-ai/about-mcp

---

# 11. Why dbt Should Not Be Compared as “Another Catalog”

dbt is solving a different responsibility.

If a user asks:

> Net Revenue by Enterprise Customer

dbt's strongest responsibility is:

~~~text
metric formula
+ dimensions
+ joins
+ query generation
~~~

A broad Context Layer adds:

~~~text
which definition applies
+ authority
+ upstream health
+ policy
+ documentation
+ organizational context
~~~

The two systems can overlap in lineage / metadata / MCP, but their architecture centers remain different.

---

# 12. Knowledge Graph / GraphRAG — General Reasoning Substrate

A Knowledge Graph route starts from:

> entities + relationships + semantics

rather than specifically:

> enterprise data governance lifecycle.

Neo4j's current GraphRAG materials emphasize:

- multi-hop reasoning；
- graph + vector retrieval；
- provenance / source tracing；
- agent reasoning across connected data；
- zero-copy / virtual graph directions。

This gives it maximum modeling flexibility.

A knowledge graph can theoretically model:

- assets；
- people；
- policy；
- authority；
- provenance；
- time；
- agents；
- decisions。

But the architecture burden shifts to the implementer:

> Who maintains freshness?  
> Who owns the ontology?  
> How are conflicts reviewed?  
> How do connector failures propagate?  
> How is context published?  
> How do data quality signals enter the graph?

So:

> **Knowledge Graph is a powerful substrate, but Context Management requires an operating model on top.**

Sources:
- https://neo4j.com/graphrag/
- https://neo4j.com/blog/graph-database/introducing-neo4j-virtual-graph-graph-reasoning-on-the-data-you-already-have/

---

# 13. RAG / Vector DB Route — Retrieval-first

The simplest Agent architecture often begins:

~~~text
documents
-> chunks
-> embeddings
-> vector search
-> LLM
~~~

Strengths:

- fast to build；
- good for unstructured text；
- flexible；
- low schema cost。

But it does not natively solve:

- stable enterprise identity；
- lineage；
- authority；
- conflicting definitions；
- temporal validity；
- source health；
- structured policy；
- write governance。

So RAG is better treated as:

> **retrieval mechanism**

inside Context Engineering, not as the whole enterprise Context Layer.

Most mature Context Platforms now themselves use embeddings / semantic search, but embeddings are one retrieval index over a larger governed substrate.

---

# 14. The Most Important Comparative Axis: What Is the Source of Authority?

This is more revealing than “supports MCP?”

## DataHub

Authority is expressed through:

- ownership；
- published Context；
- Data Expert review；
- policies；
- certification / governance metadata。

## Atlan

Authority through:

- ownership；
- glossary；
- certification；
- governance policies；
- approved changes / existing-value protection。

## OpenMetadata

Authority through:

- ownership；
- domains；
- glossary；
- RBAC / governance metadata；
- metadata entity state。

## Collibra

Authority is especially explicit via:

- community / domain model；
- stewardship；
- workflow；
- governance operating model；
- business asset lifecycle。

## dbt

Authority is primarily:

> code / semantic definitions under software-development workflow.

## Knowledge Graph

Authority must be explicitly designed into ontology / provenance / workflows.

## RAG

Authority is usually external to the vector index unless explicitly modeled.

This axis explains why Context Layer is fundamentally more than search.

---

# 15. Another Comparative Axis: Where Does Truth Become Executable?

| Architecture | Executable Truth |
|---|---|
| DataHub | Mostly external runtime; DataHub contextualizes |
| Atlan | Mostly external runtime; Atlan contextualizes/governs |
| OpenMetadata | Mostly external runtime; OpenMetadata contextualizes |
| Collibra | Mostly governance context, integrations execute |
| dbt | Semantic definitions directly compile/execute analytics queries |
| Knowledge Graph | Depends on application/tool layer |
| RAG | LLM/application decides action |

This is why Semantic Layer remains a distinct architectural plane.

---

# 16. Agent Write-back Patterns

## DataHub

MCP supports metadata mutations; Context lifecycle separates generated/published context for specific workflows.

## Atlan

MCP write tools preview the proposed change and wait for approval before writing.

This is a strong generic mutation guardrail.

## OpenMetadata

MCP / AI SDK exposes patch/create actions, glossary creation, lineage writes, quality test creation.

## Collibra

MCP includes read and action tools under OAuth / scopes; governance workflows remain central.

## dbt

AI tooling generally writes code/project changes that can be reviewed through development workflows; dbt Wizard shows diffs before persistence.

## Knowledge Graph / RAG

Write governance is application-defined.

This axis reveals an important distinction:

> **Some platforms govern the write transaction; others govern the resulting knowledge object; mature Context systems eventually need both.**

---

# 17. Convergence Around MCP Does Not Mean Architectural Convergence

By 2026:

- DataHub has MCP；
- Atlan has MCP；
- OpenMetadata has MCP；
- Collibra has MCP；
- dbt has MCP；
- Neo4j ecosystem supports MCP patterns。

So:

> “supports MCP” is no longer a meaningful differentiator by itself.

MCP only standardizes:

- tool discovery；
- invocation；
- transport / integration。

The important question is:

> **What governed substrate and lifecycle sits behind the MCP server?**

---

# 18. Five Architecture Archetypes

We can now group the landscape into five archetypes.

~~~mermaid
flowchart TB
    A[Metadata-first<br/>DataHub / OpenMetadata]
    B[Governance-first<br/>Collibra]
    C[Catalog-context-first<br/>Atlan]
    D[Semantic-execution-first<br/>dbt]
    E[Graph-reasoning-first<br/>Knowledge Graph / Neo4j]

    A --> X[Enterprise AI Context]
    B --> X
    C --> X
    D --> X
    E --> X
~~~

These categories overlap.

They describe:

> historical architectural center, not exclusive product identity.

---

# 19. What DataHub Appears Distinctive In

Without ranking, DataHub's current route is distinctive in the combination of:

~~~text
open metadata graph
+ event-driven metadata changes
+ lineage / quality / governance
+ explicit Context generation lifecycle
+ proposal / eval / publication
+ Agent Registry
+ managed + OSS context activation paths
~~~

Atlan now overlaps heavily on the Context Layer category.

OpenMetadata overlaps heavily on the open metadata graph + MCP category.

Collibra overlaps strongly on governance / authority / Agent metadata.

dbt owns a deeper executable-semantic responsibility.

Knowledge Graph systems offer more general graph semantics but require more context-operating-model assembly.

So DataHub's differentiation is best studied as:

> **integration of active metadata operations with governed context lifecycle**

rather than “it has graph” or “it has MCP.”

---

# 20. What Atlan Appears Distinctive In

Atlan's current architecture emphasizes:

~~~text
Enterprise Data Graph
+ rich catalog/governance
+ Context Agents Studio
+ Context Engineering Studio
+ governed MCP tool surface
+ write preview / approval
~~~

Atlan's MCP itself is notably broad: current docs list dozens of governed tools across search, lineage, metadata, glossary, data quality, domains and knowledge.

This makes Atlan and DataHub the most direct architectural comparison in the “context layer for AI” category.

A useful future deep dive:

> **DataHub Context Hub vs Atlan Context Agents / Context Engineering Studio**

—not generic catalog feature comparison.

---

# 21. What OpenMetadata Appears Distinctive In

OpenMetadata's advantage as an architecture pattern is:

> open-source unified metadata knowledge graph exposed directly through APIs / MCP / AI SDK.

It has fewer conceptual layers between:

~~~text
metadata graph
-> MCP
-> Agent
~~~

This can be simpler and highly extensible.

The trade-off to investigate is whether complex Context lifecycle primitives live:

- inside OpenMetadata；
- in Collate commercial AI layers；
- or in downstream Agent applications。

So the interesting comparison is:

> **Context substrate vs Context operating lifecycle.**

---

# 22. What Collibra Appears Distinctive In

Collibra's historical center is organizational governance.

That gives it a natural model for:

- stewardship；
- business assets；
- workflow；
- privacy；
- compliance；
- policy；
- AI model / agent governance。

As Context Platforms mature, these “organizational authority” primitives may become increasingly important.

This supports a broader thesis from Phase 1:

> **The hardest Context Layer problem may eventually be authority, not retrieval.**

---

# 23. What dbt Appears Distinctive In

dbt provides the clearest answer to:

> **How does business meaning become executable?**

The Semantic Layer centralizes metrics and joins, and MCP exposes those semantics directly to Agent clients.

That makes dbt a strong complement to metadata Context Platforms rather than a substitute.

A mature enterprise stack may therefore be:

~~~mermaid
flowchart LR
    CATALOG[Context Platform]
    DBT[Semantic Runtime]
    POLICY[Policy Runtime]
    DATA[Warehouse]
    AG[Agent]

    CATALOG --> AG
    AG --> DBT --> POLICY --> DATA
~~~

---

# 24. What Knowledge Graph Appears Distinctive In

Knowledge Graph / GraphRAG provides maximum flexibility for:

- arbitrary entity types；
- arbitrary relationships；
- multi-hop reasoning；
- ontology-driven semantics；
- graph + vector retrieval。

The cost is:

> governance / lifecycle / freshness / authority are architectural responsibilities you must build or integrate.

So Knowledge Graph answers:

> how to represent and traverse connected knowledge.

Context Platform must additionally answer:

> how to operate that knowledge as trusted enterprise infrastructure.

---

# 25. A Better Comparison Framework

Future comparisons should use this matrix:

~~~text
1. Identity
2. Relationship Graph
3. Business Semantics
4. Operational Context
5. Authority
6. Provenance
7. Temporal Validity
8. Conflict Model
9. Context Generation
10. Human Validation
11. Publication
12. Agent Retrieval
13. Agent Write-back
14. Semantic Execution
15. Runtime Authorization
16. Decision Audit
17. Open / Interoperable Interfaces
~~~

This avoids meaningless comparisons like:

> Product A has AI search, Product B has AI search too.

---

# 26. Initial Comparative Positioning

The architecture landscape can be summarized as:

~~~mermaid
flowchart TB
    DATA[Enterprise Data / Systems]

    DH[DataHub<br/>Active Metadata + Context Lifecycle]
    AT[Atlan<br/>Enterprise Data Graph + Context/Governance]
    OM[OpenMetadata<br/>Open Metadata Knowledge Graph]
    CO[Collibra<br/>Governance Operating Model]
    DBT[dbt<br/>Executable Semantic Layer]
    KG[Knowledge Graph<br/>General Graph Reasoning]

    AG[Agents]

    DATA --> DH --> AG
    DATA --> AT --> AG
    DATA --> OM --> AG
    DATA --> CO --> AG
    DATA --> DBT --> AG
    DATA --> KG --> AG
~~~

But the arrows hide the real distinction:

- DataHub / Atlan / OpenMetadata / Collibra largely **contextualize and govern**;
- dbt **computes governed analytical semantics**;
- Knowledge Graph **models and reasons across arbitrary relationships**;
- RAG **retrieves information into model context**.

Production Agent architectures may use several at once.

---

# 27. Important Conclusion: The Future Is Probably Compositional

The research so far does not support a clean:

> one platform owns the entire enterprise AI context stack

architecture.

More likely:

~~~mermaid
flowchart TB
    META[Metadata / Context Platform]
    SEM[Semantic Runtime]
    POLICY[Identity / Policy]
    KG[Domain Knowledge Graph]
    RAG[Retrieval / Memory]
    AG[Agent Runtime]

    META --> AG
    SEM --> AG
    POLICY --> AG
    KG --> AG
    RAG --> AG
~~~

The design problem becomes:

> **Which system is authoritative for which kind of context?**

That is fundamentally an authority/federation question.

---

# 28. DataHub Research Implication

This comparison sharpens why DataHub remains interesting.

It does not need to become:

- dbt；
- OPA；
- Neo4j；
- warehouse；
- Agent runtime。

Its strategic role can remain:

> **maintain the shared data/context graph and trust lifecycle that connects those systems.**

That is a narrower but potentially foundational architecture position.

---

# 29. Next Comparative Deep Dives

The most useful next comparisons are:

## 02 — DataHub vs Atlan

Because both explicitly position as Context Layer for AI.

Compare:

- context graph model；
- context generation；
- human validation；
- write-back；
- MCP tools；
- authority；
- freshness；
- lifecycle。

## 03 — DataHub vs OpenMetadata

Because both share open metadata-platform / graph roots.

Compare:

- metadata model；
- openness；
- MCP；
- context lifecycle；
- commercial-vs-OSS boundary。

## 04 — DataHub + dbt

Not “versus”.

Study:

> Context Plane + Semantic Execution Plane composition.

## 05 — DataHub + Knowledge Graph

Study when DataHub's metadata graph is sufficient and when a domain Knowledge Graph is still needed.

---

# Sources

## DataHub
- https://datahub.com/products/context-platform/
- https://docs.datahub.com/docs/features/feature-guides/agents
- https://datahub.com/blog/datahub-cloud-v2-2/

## Atlan
- https://docs.atlan.com/get-started/what-is-atlan
- https://docs.atlan.com/agents/faq/context-layer
- https://docs.atlan.com/product/capabilities/atlan-ai/references/mcp-tools
- https://docs.atlan.com/product/capabilities/governance/context-agents-studio/concepts/agents

## OpenMetadata
- https://docs.open-metadata.org/v1.12.x/how-to-guides/mcp
- https://docs.open-metadata.org/v1.12.x/api-reference/sdk/ai-sdk
- https://docs.open-metadata.org/v1.12.x/how-to-guides/mcp/semantic-search

## Collibra
- https://productresources.collibra.com/docs/collibra/latest/Content/ModelContextProtocol/co_mcp.htm
- https://productresources.collibra.com/docs/collibra/latest/Content/ModelContextProtocol/ref_mcp-tools.htm
- https://productresources.collibra.com/docs/collibra/2026.02/Content/BusinessGlossary/to_business-glossary.htm
- https://productresources.collibra.com/docs/collibra/latest/Content/Catalog/IntegrateAIModels/to_integrating-ai-models.htm

## dbt
- https://docs.getdbt.com/docs/use-dbt-semantic-layer/dbt-sl
- https://docs.getdbt.com/docs/dbt-ai/about-mcp

## Knowledge Graph / GraphRAG
- https://neo4j.com/graphrag/
- https://neo4j.com/blog/graph-database/introducing-neo4j-virtual-graph-graph-reasoning-on-the-data-you-already-have/
