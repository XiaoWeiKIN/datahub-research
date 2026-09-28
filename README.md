# DataHub Research

面向 DataHub 的系统化学习、官方文档中文学习笔记与技术实验仓库。

> 官方站点：https://datahub.com/
> 官方文档：https://docs.datahub.com/

## 目标

这个仓库不做官方文档的镜像。学习材料采用：

**官方文档 → 中文释义/摘要 → 概念拆解 → 问题 → 实验验证 → 结论**

重点回答：

1. DataHub 解决什么问题？
2. Metadata Graph 的核心抽象是什么？
3. Metadata Ingestion 如何工作？
4. Dataset、URN、Aspect、MCP 等概念如何组织？
5. Lineage、Search、Governance、Data Quality 如何实现？
6. Python SDK / API 如何使用？
7. DataHub Core 与 DataHub Cloud 有什么边界？
8. Context / Agents / MCP 等新能力如何与 metadata graph 结合？

## 学习入口

- [学习路线](docs/learning-path.md)
- [官方文档学习索引](docs/official-docs/README.md)
- [术语表](docs/glossary.md)
- [01 - What is DataHub?](docs/official-docs/01-what-is-datahub.md)
- [问题清单](notes/questions.md)
- [实验目录](experiments/README.md)

## 目录

```text
.
├── README.md
├── docs/
│   ├── learning-path.md
│   ├── glossary.md
│   └── official-docs/
│       ├── README.md
│       └── 01-what-is-datahub.md
├── notes/
│   └── questions.md
└── experiments/
    └── README.md
```

## 笔记原则

每篇文档尽量包含：

- Source：官方原文链接
- Status：待学习 / 学习中 / 已完成 / 待验证
- 中文释义：用自己的语言解释原文
- Key Concepts：关键概念
- Mental Model：概念之间的关系
- Questions：阅读后仍未解决的问题
- Verification：需要通过源码或实验验证的部分
- Takeaways：当前阶段结论

## License / Attribution

DataHub 名称、官方文档及相关内容归其各自权利人所有。本仓库主要保存个人研究笔记、中文释义和实验结果；需要查看完整、最新内容时，请以官方文档为准。
