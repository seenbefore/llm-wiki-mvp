---
type: concept
title: RAG
aliases: [RAG]
created: 2026-04-07
updated: 2026-04-07
tags: [llm, retrieval]
---

# RAG

## 定义

`RAG` 是一种在回答问题时，从外部资料中检索相关片段，再结合生成模型形成答案的模式。

## 边界

- 它关注“问题发生时如何检索资料”，不天然包含长期沉淀和持续维护知识页的过程。
- 它与 `[[persistent-knowledge-wiki|Persistent Knowledge Wiki]]` 的差别不在于是否使用资料，而在于资料是否先被编译进一个可持续演化的中间层。

## 关键要点

- 传统体验中，RAG 往往需要在每次提问时重新发现和拼接资料片段。
- 当问题需要跨多份资料综合时，RAG 的重复发现成本会变高。
- 在 `[[llm-wiki|LLM Wiki]]` 语境下，RAG 更像“原始资料入口”，而非最终知识组织形式。

## 相关页面

- [[llm-wiki|LLM Wiki]]
- [[persistent-knowledge-wiki|Persistent Knowledge Wiki]]

## 主要来源

- [[2026-04-07-karpathy-llm-wiki]]
