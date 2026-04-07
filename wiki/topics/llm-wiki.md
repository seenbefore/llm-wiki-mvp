---
type: topic
title: LLM Wiki
aliases: [LLM Wiki]
created: 2026-04-07
updated: 2026-04-07
tags: [llm, wiki, knowledge-base]
source_count: 1
---

# LLM Wiki

## 当前结论

`LLM Wiki` 是一种把 LLM 从“按需回答器”变成“持续维护者”的知识库模式。它不把原始资料直接暴露给问答流程，而是在中间维护一个持续演化的 Markdown Wiki，用于承接摘要、综合、交叉引用、冲突标注和可复用分析。

## 关键观点

- 它强调“知识一次编译、持续更新”，而不是每次问题都从原始资料重新发现答案。
- 它把 `wiki/` 当成知识中间层，把 `raw/` 当成只读事实层。
- 它要求 LLM 维护 `index`、`log`、主题页和来源页之间的结构关系。
- 它适合长期积累型场景，例如研究、阅读、个人管理和团队知识沉淀。

## 分歧与不确定性

- 冲突 / 待核实：目前只有一篇概念性来源，关于长期规模化维护效果和实际操作成本仍缺少更多案例支持。

## 相关实体

- [[andrej-karpathy|Andrej Karpathy]]

## 相关概念

- [[RAG]]
- [[persistent-knowledge-wiki|Persistent Knowledge Wiki]]

## 主要来源

- [[2026-04-07-karpathy-llm-wiki]]

## 后续问题

- 增量维护在资料规模扩大后是否仍然比传统 RAG 更稳？
- 哪些工作最适合依赖模板，哪些应该留给 Agent 自主判断？
- 什么时候需要从索引文件升级到搜索工具？
