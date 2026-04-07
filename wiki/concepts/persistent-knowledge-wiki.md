---
type: concept
title: Persistent Knowledge Wiki
aliases: [Persistent Knowledge Wiki]
created: 2026-04-07
updated: 2026-04-07
tags: [llm, wiki, synthesis]
---

# Persistent Knowledge Wiki

## 定义

`Persistent Knowledge Wiki` 指一个由 LLM 持续维护的、可交叉引用的 Markdown 知识层。它位于原始资料与问答之间，会随着新资料和新问题不断增量演化。

## 边界

- 它不是原始资料仓库，原始资料仍保留在 `raw/`。
- 它也不是一次性总结文档，而是长期维护的中间知识层。
- 它和普通笔记库的区别在于：写入、补链、修订、冲突标注主要由 Agent 持续完成。

## 关键要点

- 重点不只是“存下摘要”，而是维护结构关系和历史演化。
- 交叉引用、冲突标注、索引和日志都是这个概念的重要组成部分。
- 高价值问答结果也应作为新页面回写，而不是只存在聊天记录中。

## 相关页面

- [[llm-wiki|LLM Wiki]]
- [[RAG]]
- [[andrej-karpathy|Andrej Karpathy]]

## 主要来源

- [[2026-04-07-karpathy-llm-wiki]]
