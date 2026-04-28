---
type: source
title: Aliases - Obsidian Help
aliases: [Obsidian Aliases]
source_kind: article
source_path: raw/articles/2026-04-07-obsidian-aliases.md
created: 2026-04-07
updated: 2026-04-07
tags: [obsidian, aliases, yaml, wikilinks]
---

# Aliases - Obsidian Help

## 来源信息

- 原始文件：`raw/articles/2026-04-07-obsidian-aliases.md`
- 作者 / 发布方：Obsidian Help
- 发布时间：未在抓取内容中明确标注
- 链接：https://help.obsidian.md/aliases

## 核心摘要

- Alias 是笔记的备用名字，应写在 YAML frontmatter 的 `aliases` 列表中。
- 当用户通过 alias 选中目标笔记时，Obsidian 生成的不是 `[[AI]]`，而是 `[[Artificial Intelligence|AI]]` 这种格式。
- Obsidian 明确区分“单次显示文本变化”和“可复用的替代命名”这两种需求。
- Alias 适合缩写、昵称和多语言名称。

## 关键细节

- `aliases` 必须是 YAML 列表，而不是单个自由文本字段。
- 如果只是某一个地方想展示不同名称，更适合用自定义显示文本，而不是新增 alias。
- Obsidian 采用 `[[真实目标|别名]]` 的原因之一是为了更好的互操作性。

## 可复用结论

- 当前仓库应该把人类可读标题放到 `title` 和 `aliases` 里，把链接目标稳定地绑定到真实文件名。
- 当页面长期会以多个名字被引用时，应该补 `aliases`，而不是直接写容易歧义的裸链接。
- 这条规则正好可以解释为什么之前的 `[[LLM Wiki]]` 会误建空白页。

## 待跟进

- 后续可以补充 alias 与 backlinks、重构命名、国际化命名之间的实践经验。

## 关联页面

- [[obsidian-linking-conventions|Obsidian 链接规范]]
- [[2026-04-07-obsidian-internal-links|Internal links - Obsidian Help]]
