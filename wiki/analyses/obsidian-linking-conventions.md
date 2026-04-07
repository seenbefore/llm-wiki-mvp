---
type: analysis
title: Obsidian 链接规范
aliases: [Obsidian Linking Conventions]
created: 2026-04-07
updated: 2026-04-07
question: 当前仓库应该如何编写 Obsidian 内部链接，才能避免误建空白页并保持可读性？
tags: [obsidian, links, aliases, repository-conventions]
---

# Obsidian 链接规范

## 问题

当前仓库混用了“展示标题”和“真实文件名”，导致部分双链在 Obsidian 中点击时会新建空白笔记，而不是跳到已有文件。

## 简短结论

本仓库应统一采用以下规范：

- 文件名优先使用稳定的 `kebab-case`
- 正文中默认写 `[[文件名|显示文本]]`
- 只有当真实文件名或显式 `aliases` 与链接目标完全一致时，才允许直接写 `[[目标名]]`
- 页面若长期存在多个称呼，应在 YAML 中维护 `aliases` 列表

## 证据

- [[2026-04-07-obsidian-internal-links|Internal links - Obsidian Help]]
- [[2026-04-07-obsidian-aliases|Aliases - Obsidian Help]]
- [[llm-wiki|LLM Wiki]]

## 展开分析

Obsidian 的内部链接首先解析目标笔记，而不是优先解析你想展示给人看的标题。如果真实文件叫 `llm-wiki.md`，而正文写成 `[[LLM Wiki]]`，在没有 alias 精确匹配时，Obsidian 会把它视为“一个尚不存在但应当存在的笔记”，因此点击时会创建新文件。

官方文档给出的推荐方式很清楚：

- 需要单次改变显示文本时，用 `[[真实目标|显示文本]]`
- 需要长期支持多个名字时，在 YAML `aliases` 中定义别名
- 通过 alias 选中目标后，Obsidian 仍然会生成 `[[真实目标|别名]]`，而不是只写别名本身

因此，对这个仓库最稳妥的策略不是“让显示文本兼任链接目标”，而是把“解析目标”和“视觉展示”拆开处理。

## 仓库级约定

- 主题页：`wiki/topics/*.md`
- 实体页：`wiki/entities/*.md`
- 概念页：`wiki/concepts/*.md`
- 来源页：`wiki/sources/*.md`
- 文件名使用 `kebab-case`
- 显示标题写入 `title`
- 常用别称写入 `aliases`
- 正文中优先写 `[[file-name|Readable Title]]`

## 局限

- 当前规范主要覆盖基础双链，不包括标题块链接与块引用的大规模使用约定。
- 目前还没有为附件、图片和非 Markdown 文件建立完整的命名规范。

## 可回写页面

- [[llm-wiki|LLM Wiki]]
