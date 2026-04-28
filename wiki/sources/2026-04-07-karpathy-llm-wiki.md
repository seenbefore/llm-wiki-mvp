---
type: source
title: LLM Wiki
aliases: [Karpathy LLM Wiki]
source_kind: article
source_path: raw/articles/2026-04-07-karpathy-llm-wiki.md
created: 2026-04-07
updated: 2026-04-07
tags: [llm, knowledge-base, obsidian, wiki]
---

# LLM Wiki

## 来源信息

- 原始文件：`raw/articles/2026-04-07-karpathy-llm-wiki.md`
- 作者 / 发布方：Andrej Karpathy
- 发布时间：未在原文中明确标注
- 链接：[https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

## 核心摘要

- 文章提出一种不同于传统 RAG 的知识组织模式：让 LLM 维护一个持续演化的 Markdown Wiki。
- 这个 Wiki 位于原始资料与问答之间，承担摘要、综合、交叉引用、冲突标注和结构化沉淀的职责。
- 新资料进入后，LLM 不只做检索索引，而是先消化，再回写到主题页、概念页和实体页中。
- 作者把 Obsidian 视为浏览和观察 Wiki 的 IDE，而把 LLM 视为维护 Wiki 的程序员。
- 文章强调 `index.md` 和 `log.md` 两个特殊文件的重要性，分别解决内容导航和时间线追踪问题。
- 文中把 `ingest`、`query`、`lint` 定义为维持知识库复利增长的三种核心操作。

## 关键细节

- 三层架构是 `Raw sources`、`The wiki`、`The schema`。
- 好的问答结果不应只留在聊天历史里，而应沉淀成新的 Wiki 页面。
- LLM 的核心价值不只是总结，而是承担长期维护成本接近零的“知识库运维”角色。
- 文中明确指出这是一个抽象模式，不绑定某个固定目录结构或单一工具链。

## 可复用结论

- 这篇文章可以作为 `[[llm-wiki|LLM Wiki]]` 主题页的第一份种子来源。
- 它直接支撑 `[[RAG]]` 与 `[[persistent-knowledge-wiki|Persistent Knowledge Wiki]]` 这两个概念页。
- 作者本人适合拥有一个实体页 `[[andrej-karpathy|Andrej Karpathy]]`，用于追踪其提出的相关方法或观点。

## 待跟进

- 需要补充更多关于实际落地方式的来源，而不仅是概念说明。
- 需要补充对搜索工具、模板组织和长期维护策略的实践型材料。
- 需要对“与传统 RAG 的边界”继续收集案例，否则容易停留在抽象口号。

## 关联页面

- [[llm-wiki|LLM Wiki]]
- [[RAG]]
- [[persistent-knowledge-wiki|Persistent Knowledge Wiki]]
- [[andrej-karpathy|Andrej Karpathy]]