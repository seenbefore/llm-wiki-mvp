# LLM Wiki MVP

这是一个通过 OpenSkills 分发的 `LLM Wiki` 仓库，提供 3 个可直接安装和使用的本地 skill：`ingest`、`query`、`lint`。

## Quick Start

推荐先安装 skill，再让 agent 按 skill 工作；不要先手工阅读 `schema/` 再自行摸索流程。

```powershell
git clone https://github.com/seenbefore/llm-wiki-mvp
cd llm-wiki-mvp
openskills install https://github.com/seenbefore/llm-wiki-mvp
openskills sync
npx openskills read ingest
```

执行上面的命令后：

- skill 会被安装到当前项目的 `.claude/skills/`
- `openskills sync` 会生成或更新根级 `AGENTS.md`
- agent 之后就可以按已加载的 skill 指令执行 ingest、query、lint

## How To Use

### 1. Ingest

先读取 skill：

```powershell
npx openskills read ingest
```

然后让 agent 使用 `ingest` 处理一个或一组 `raw/` 来源，例如：

- ingest `raw/articles/2026-04-07-karpathy-llm-wiki.md`
- ingest `raw/articles/`

默认目标是先生成或更新 `wiki/sources/` 页面，再按需要更新主题页、概念页、实体页，并同步 `wiki/index.md` 与 `logs/log.md`。

### 2. Query

先读取 skill：

```powershell
npx openskills read query
```

然后直接基于现有 wiki 提问，例如：

- `LLM Wiki` 和 `RAG` 的核心区别是什么？
- 当前仓库里的 Obsidian 链接规范是什么？

`query` 会优先从 `wiki/index.md` 和现有 wiki 页面回答，而不是默认回到 `raw/`。

### 3. Lint

先读取 skill：

```powershell
npx openskills read lint
```

然后让 agent 对全库或局部页面做巡检，例如：

- lint 全库
- lint `wiki/concepts/`
- lint 最近更新过的页面

`lint` 默认检查孤儿页、无来源结论、索引遗漏、错链、frontmatter 问题、未标记冲突，以及不符合 Obsidian 规范的双链。

## Repo Structure

- `raw/`：原始资料，只读层。
- `wiki/`：知识中间层，由 Agent 增量维护。
- `schema/`：工作流、模板和约束规则。
- `logs/`：摄入、问答沉淀和巡检日志。
- `skills/`：仓库内的 skill 源定义。
- `.claude/skills/`：通过 `openskills install` 安装到当前项目后的 skill 副本。

## Notes

- 这 3 个 skill 的源定义位于 `skills/ingest/`、`skills/query/`、`skills/lint/`。
- 根级 `AGENTS.md` 由 `openskills sync` 生成或更新，用来把已安装 skill 暴露给 agent。
- 如果你直接把仓库作为 Obsidian vault 打开，`wiki/` 是日常浏览主区域。
- 文件名优先使用 `kebab-case`，正文优先使用 `[[文件名|显示文本]]`。
- 如果一个页面长期会以多个名字被引用，把这些名字写进 YAML `aliases`，不要直接依赖裸显示标题。

