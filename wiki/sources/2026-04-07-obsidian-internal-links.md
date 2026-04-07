---
type: source
title: Internal links - Obsidian Help
aliases: [Obsidian Internal Links]
source_kind: article
source_path: raw/articles/2026-04-07-obsidian-internal-links.md
created: 2026-04-07
updated: 2026-04-07
tags: [obsidian, links, wikilinks, markdown]
---

# Internal links - Obsidian Help

## 来源信息

- 原始文件：`raw/articles/2026-04-07-obsidian-internal-links.md`
- 作者 / 发布方：Obsidian Help
- 发布时间：未在抓取内容中明确标注
- 链接：https://help.obsidian.md/links

## 核心摘要

- Obsidian 支持两类内部链接格式：Wikilink 和 Markdown 链接。
- 默认情况下，Obsidian 更倾向使用 Wikilink。
- 如果只想改变某一次显示效果，应使用 `[[真实目标|显示文本]]` 这样的显示文本语法。
- 标题链接和块链接分别通过 `#` 与 `#^` 追加到目标后面实现。
- Obsidian 允许在重命名文件后自动更新内部链接。

## 关键细节

- Wikilink 示例是 `[[Three laws of motion]]`。
- 修改显示文本的示例是 `[[Example|Custom name]]`。
- Markdown 链接在目标里需要 URL 编码。
- 一些特殊字符不适合作为链接目标的一部分，文件名最好尽量安全、稳定。

## 可复用结论

- 当前仓库若继续使用 kebab-case 文件名，就应通过 `[[文件名|显示文本]]` 来保留人类友好的展示名。
- 对于本仓库而言，链接规范应优先建立在真实文件名之上，而不是展示标题之上。
- 标题与块链接能力后续可以用于更精细的知识组织，但当前 MVP 先统一基本双链格式更重要。

## 待跟进

- 后续可以补充 Obsidian 关于重命名链接更新、文件与链接设置项的更细文档。
- 后续可以评估是否需要在本仓库中启用 Markdown links 而不是 Wikilinks。

## 关联页面

- [[obsidian-linking-conventions|Obsidian 链接规范]]
- [[2026-04-07-obsidian-aliases|Aliases - Obsidian Help]]
