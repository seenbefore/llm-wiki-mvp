# LLM Wiki MVP

这是一个通过 OpenSkills 分发的 `LLM Wiki` 仓库，提供 5 个可直接安装和使用的本地 skill：`task-start`、`task-close`、`ingest`、`query`、`lint`。

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
- agent 之后就可以按已加载的 skill 指令执行 task-start、task-close、ingest、query、lint

## Updating A Private Knowledge Repo

如果你把这个公共仓库作为产品功能源，同时维护一个私人知识仓库用于实际沉淀内容，推荐结构是：

- 公共产品仓库：开发和发布 `skills/`、`schema/`、README 等功能。
- 私人知识仓库：保存个人的 `raw/`、`wiki/`、`.obsidian/`、`logs/` 内容，并定期合并公共功能更新。

在私人仓库中添加公共仓库作为只读上游：

```powershell
cd E:\path\to\your-private-llm-wiki
git remote add upstream https://github.com/seenbefore/llm-wiki-mvp.git
git remote set-url --push upstream DISABLED
```

之后每次更新公共功能：

```powershell
cd E:\path\to\your-private-llm-wiki
git fetch upstream
git merge upstream/master
git push private master
```

如果合并时出现冲突，通常按这个原则处理：

- `skills/`、`schema/`、产品 README：优先采用公共仓库版本，再补充你的私人说明。
- `raw/`、`wiki/`、`.obsidian/`、`logs/`：优先保留私人仓库版本。

不要从私人知识仓库推送到 `upstream`；私人仓库只推自己的 `private` remote。

## How To Use

### 1. Task Start

先读取 skill：

```powershell
npx openskills read task-start
```

然后在重要任务开始前让 agent 做任务启动，例如：

- task-start：我要开始一个新任务，主题是 AI 工具整合
- task-start：我要做 DGX Spark 本地模型部署，请先查 wiki

`task-start` 会只读 `wiki/`，输出相关页面、已有上下文、可复用结论、当前缺口和本次建议上下文包；它不直接执行任务，也不写入仓库。

### 2. Task Close

先读取 skill：

```powershell
npx openskills read task-close
```

然后在重要任务结束时让 agent 做任务收尾，例如：

- task-close：判断这次任务是否值得沉淀到 LLM-wiki
- task-close：把本次 AI 工具整合讨论整理成 raw task note

`task-close` 默认只写入 `raw/task-notes/YYYY-MM-DD-<task-slug>.md`，作为后续 `ingest` 的原始资料；它不直接改 `wiki/`。

### 3. Ingest

先读取 skill：

```powershell
npx openskills read ingest
```

然后让 agent 使用 `ingest` 处理一个或一组 `raw/` 来源，例如：

- ingest `raw/articles/2026-04-07-karpathy-llm-wiki.md`
- ingest `raw/articles/`

默认目标是先生成或更新 `wiki/sources/` 页面，再按需要更新主题页、概念页、实体页，并同步 `wiki/index.md` 与 `logs/log.md`。

### 4. Query

先读取 skill：

```powershell
npx openskills read query
```

然后直接基于现有 wiki 提问，例如：

- `LLM Wiki` 和 `RAG` 的核心区别是什么？
- 当前仓库里的 Obsidian 链接规范是什么？

`query` 会优先从 `wiki/index.md` 和现有 wiki 页面回答，而不是默认回到 `raw/`。

### 5. Lint

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

- 这 5 个 skill 的源定义位于 `skills/task-start/`、`skills/task-close/`、`skills/ingest/`、`skills/query/`、`skills/lint/`。
- 根级 `AGENTS.md` 由 `openskills sync` 生成或更新，用来把已安装 skill 暴露给 agent。
- 如果你直接把仓库作为 Obsidian vault 打开，`wiki/` 是日常浏览主区域。
- 文件名优先使用 `kebab-case`，正文优先使用 `[[文件名|显示文本]]`。
- 如果一个页面长期会以多个名字被引用，把这些名字写进 YAML `aliases`，不要直接依赖裸显示标题。
- 若在**其他项目工作区**中调用已安装的 task-start/task-close/ingest/query/lint，需要读写本机某份 `llm-wiki-mvp` 克隆，请设置环境变量 `LLM_WIKI_ROOT` 指向该仓库根；详见各 skill 内「Wiki 根路径（LLM_WIKI_ROOT）」一节。
