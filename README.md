# LLM Wiki MVP

一个面向个人本地使用的 Markdown 知识库骨架。

这个仓库把原始资料、LLM 维护的知识页、约束规则和演化日志分开，目标是让 Agent 持续把 `raw/` 里的资料编译进 `wiki/`，而不是每次提问都重新从原始资料里摸索。

## 目录

- `raw/`：原始资料，只读层。
- `wiki/`：知识中间层，由 Agent 增量维护。
- `schema/`：Agent 维护规则、模板和工作流说明。
- `logs/`：摄入、问答沉淀和巡检日志。

## 推荐使用方式

1. 把网页剪藏、PDF、摘录文本、图片等放进 `raw/`。
2. 让 Agent 依据 `schema/AGENTS.md` 处理新资料。
3. 先生成 `wiki/sources/` 摘要页，再更新相关主题页、概念页或实体页。
4. 定期执行一次 `lint` 检查，修复断链、孤儿页、无来源结论和重复页面。

## Obsidian 友好约定

- 直接把本仓库文件夹作为一个 Obsidian vault 打开。
- `wiki/` 是日常浏览的主区域，适合配合双链和图谱视图使用。
- `.obsidian/` 下的工作区状态默认不提交，只保留必要说明。
- 原始资料建议按来源类型整理，并把附件统一放到 `raw/assets/`。
- 文件名优先使用 `kebab-case`，正文中优先使用 `[[文件名|显示文本]]`。
- 如果一个页面长期会以多个名字被引用，把这些名字写进 YAML `aliases`，不要直接依赖裸显示标题。

## 第一次使用

- 阅读 `schema/AGENTS.md`
- 参考 `schema/templates/` 中的最小模板
- 从 `schema/workflows.md` 的 `ingest` 流程开始
- 在 `logs/log.md` 记录每次重要变更

## Local Skills

- `skills/ingest/`
- `skills/query/`
- `skills/lint/`

这 3 个本地 skill 是当前仓库的稳定操作入口，分别对应 ingest、query、lint 三条核心工作流。
