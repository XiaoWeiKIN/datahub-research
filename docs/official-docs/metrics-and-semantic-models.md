# Metrics & Semantic Models — 官方材料释义与证据表

**核查日期：2026-09-28**  
**类型：中文释义、事实核查与研究伴读；不是全文翻译。**  
**研究正文：[02 — Metrics & Semantic Models](../research/02-metrics-and-semantic-models.md)。**

## 1. 最重要的官方定位

DataHub 把 Metrics 定义为**管理指标定义的目录**。它登记计算定义、维度语境、关系和治理信息，不承担业务指标值的计算或可视化。后两项仍在 BI 或语义层完成。[S1]

理解这句话后，读取其余材料就不容易把“可查询指标元数据”误解成“可查询指标计算结果”。

## 2. 当前公开能力与边界

| 核查项 | 公开证据支持什么 | 不应扩大成什么 | 来源 |
| --- | --- | --- | --- |
| 新对象 | Metric、Semantic Model 为独立实体；逻辑模型数据集复用 Dataset 类型 | DataHub 新增了三种完全独立的实体类型 | S1 |
| 发布状态 | 功能指南标 Beta；Cloud 从 2.1.0、Core 从 1.7.0 可用 | 任意版本都默认打开；整体已经 GA | S1、S2 |
| 原生采集 | 指南给出 Snowflake Semantic Views；其他来源可使用 SDK | 所有语义平台的新实体采集都已完整支持 | S1 |
| Cube | 2.1 发布说明列出 Cube 连接器 | 不经版本核查就认定其新实体映射与 Snowflake 等价 | S2 |
| 血缘模型 | 模型作为容器，指标经逻辑 Dataset 追到物理 Dataset | 模型本身是每个指标的计算中间节点 | S1、S3 |
| 影响分析 | 实体级影响可表达；完整列到指标影响仍列 roadmap | 任意列变化都能精确找到全部受影响指标 | S1 |
| AI Context | 可存同义词、指导和示例 | 全部 Agent 工具已消费这些字段并保证答案正确 | S1 |
| 指标专用 MCP | 指南的后续工作明确列有此项 | 已有通用 MCP 就代表新实体覆盖完成 | S1 |
| 指标身份 | 环境不是单独 URN 组成项；不同平台保持不同身份 | 自动区分所有环境；自动合并跨平台同名指标 | S1 |
| 计算数值 | 官方 FAQ 明确不由 DataHub Metrics 计算 | DataHub 是指标执行服务或 Metric Store | S1 |

这些是**文档级证据**，不是实例验收结果。本文没有运行连接器、SDK 或 MCP 工具。

## 3. 两组需要保留的资料差异

### 发布博客与功能指南的差异

July Town Hall 回顾用 certified definition 描述 Agent 场景的价值；当前指南仍将指标专用 MCP 和 AI-native discovery 写在下一步里。不能用预览文字覆盖指南限制，也不能由此推断通用 MCP 完全不可用。[S1、S4]

指南的 Lineage 章节已经描述 Semantic Model 容器，What's Coming Next 又继续列出容器视图相关改进。结合 2.2 的发布说明，能够确认容器方向已进入产品，但不能据此认定全部视觉改进均已完成。[S1、S3]

### 开发分支文档与正式发布的差异

所读指南还写到 Core v1.8.0、Cloud 2.3.0 起默认开启。但本次 GitHub Releases 的 `latest` 接口返回 **v1.7.0.1**，发布时间为 **2026-09-03T19:36:36Z**。[S1、S5]

因此本文仅报告：“当前文档描述了这些版本的默认行为”，**不把 v1.8.0 或 Cloud 2.3.0 已发布、已部署当作确认事实**。正式版本、是否启用和连接器输出是三个不同问题。

## 4. 时间线：区分会议月份、文章发布日期与版本发布

| 时间证据 | 事件 | 可以得出的结论 |
| --- | --- | --- |
| 2025-03-24 的 roadmap 文章 | 已提到 Metrics Catalog | 方向至少在这份规划中存在，不是 2026 才出现 |
| June 2026 Town Hall；回顾文章发表于 2026-07-24 | 讨论一等 Semantic Models / Metrics 与 OSI | 属于方向与近期计划，不等同发布 |
| 2026-08-03，Cloud 2.1 发布文章 | SDK、实体和 UI | 正式发布证据；原生连接器覆盖需另查 |
| July Town Hall；回顾文章发表于 2026-09-01 | Metrics Catalog 预览回顾 | 会议月份不能用作网页发布日期 |
| 2026-09-09，Cloud 2.2 发布文章 | Semantic Model container lineage | 容器化路径及重新采集／迁移要求 |

来源：S2、S3、S4、S6、S7。这里不试图证明最早出现 Metric 一词的历史，只纠正“2026 年才首次提出 Metrics Catalog”的推断。

## 5. 三个概念的中文释义

**Semantic Model：计算环境。** 它组织逻辑数据集、允许的关联以及可用的维度和指标。名称相同的对象在不同产品中的粒度未必相同，不能只按名称做一对一映射。[S1]

**Metric：命名的业务计算定义。** 它包含表达式、所处模型及治理信息，能够关联派生指标、数据来源和消费者。模型字段中保存表达式，不代表保存了每个时间区间的计算结果。[S1]

**Semantic Model Dataset：业务投影。** 它是具有对应 subtype 的 Dataset，拥有自己的 schema、治理与血缘。这解释了为何官方在新路径中保留逻辑层，而不直接把每个指标连到一个物理表。[S1]

## 6. OSI 的名称与版本也需要校准

OSI 现称 Apache Ossie（incubating），官方站点给出的更名公告日期为 2026-07-10。[S8]

本次所读 core spec 的版本历史将 `0.2.0.dev0` 标为未发布，并提醒 schema 仍可变化。规范的 main 分支不能代替某个已发布 DataHub / dbt / 连接器实际支持的规范版本。[S9]

多方言表达式与厂商扩展是表达机制，不是全自动转换或跨引擎计算等价证明。作为实现层面的交叉证据，Count 的 OSI 转换文档明确记录了未转换的 AI Context 和不支持的跨视图指标表达式。[S10]

## 7. 建议带着哪些问题阅读官方材料？

优先区分四种说法：**模型可表达、SDK 可写入、连接器可自动采集、Agent 可可靠消费**。它们需要不同证据，不能相互替代。

接着追问三个边界：公式由谁批准、数据由谁计算、权限由谁执行。最后再看“统一”到底指统一检索入口、统一描述格式，还是已经解决跨系统定义冲突。后者需要更强的产品与运行证据。

## 来源记录

[S1]: https://github.com/datahub-project/datahub/blob/15b128f568cec343d4eb15592aedf25c98995f7d/docs/features/feature-guides/metrics-and-semantic-models.md
[S2]: https://datahub.com/blog/datahub-cloud-2-1/
[S3]: https://datahub.com/blog/datahub-cloud-v2-2/
[S4]: https://datahub.com/blog/real-stories-in-production-july-2026-town-hall-highlights/
[S5]: https://github.com/datahub-project/datahub/releases/tag/v1.7.0.1
[S6]: https://datahub.com/blog/whats-next-for-datahub/
[S7]: https://datahub.com/blog/inside-the-context-platform-june-2026-town-hall-highlights/
[S8]: https://ossie.apache.org/
[S9]: https://github.com/apache/ossie/blob/main/core-spec/spec.md
[S10]: https://learn.count.co/docs/third-party-file-support/osi-support

- [S1 — 官方功能指南固定快照][S1]：GitHub 官方文档提交 `15b128f568cec343d4eb15592aedf25c98995f7d`；本次官网详情读取失败，使用官方仓库文档补充。
- [S2 — Cloud 2.1 发布说明][S2]，2026-08-03。
- [S3 — Cloud 2.2 发布说明][S3]，2026-09-09。
- [S4 — July Town Hall 回顾][S4]，网页日期 2026-09-01。
- [S5 — Core v1.7.0.1 正式 Release][S5]，本次 latest 接口返回的版本；UTC 发布时间 2026-09-03T19:36:36Z。
- [S6 — 2025 Roadmap][S6]，2025-03-24。
- [S7 — June Town Hall 回顾][S7]，网页日期 2026-07-24。
- [S8 — Apache Ossie 官方站点][S8]，含 2026-07-10 更名公告入口。
- [S9 — Ossie Core Specification][S9]，动态 main；所读版本历史为 `0.2.0.dev0 (Unreleased)`。
- [S10 — Count OSI 转换说明][S10]，只用于说明具体实现的兼容限制，不代表所有消费者。

所有来源的访问核查日期均为 2026-09-28。博客发布、开发文档、正式 Release 和运行验证分别记录；没有实测结果时不使用“已验证可用”的措辞。
