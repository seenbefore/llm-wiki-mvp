---
name: task-start
description: Start an important task by reading the local LLM Wiki and producing a reusable context package before execution.
---

# Task Start

本 skill 用于在一个重要任务开始前，先从当前 LLM Wiki 中提取相关上下文，避免重复解释、重复推理和遗忘已有结论。

它支持两种模式：

- `web`：生成一份可直接复制到 ChatGPT 网页版或其他网页 AI 的干净 Markdown 上下文包；输出后停止，不执行本地任务。
- `local`：生成给当前本地 agent / Codex 会话使用的任务启动上下文；若用户任务本身是执行型，启动后可以继续进入计划、实现或验证流程。

无论哪种模式，启动阶段都只读 `wiki/` 和相关仓库上下文，不写入 `raw/`、`wiki/` 或 `logs/`。

## Wiki 根路径（用户配置文件）

本 skill 中的相对路径（如 `schema/AGENTS.md`、`wiki/`、`raw/`、`logs/`）必须先通过用户目录配置文件解析到本机 `llm-wiki-mvp` 仓库根。

- **配置文件：** `%USERPROFILE%\.config\llm-wiki-mvp\config.json`
- **字段名：** `root`
- **含义：** 本机 `llm-wiki-mvp` 仓库根的绝对路径（不是 `wiki/` 子目录；该根下须存在 `schema/AGENTS.md`、`wiki/` 等）。

配置示例：

```json
{
  "root": "E:\\code\\llm-wiki-mvp"
}
```

**解析规则：**

1. 必须先读取 `%USERPROFILE%\.config\llm-wiki-mvp\config.json`。
2. 若配置文件存在、JSON 有效，且 `root` 为非空字符串，本 skill 中所有相对路径均相对于 `root` 解析。
3. 若配置文件不存在、JSON 无法解析、`root` 缺失或为空，停止执行并提示用户创建或修复该配置文件。
4. 旧环境变量 `LLM_WIKI_ROOT` 已弃用；即使存在，也不得作为路径来源。若检测到它，请提示用户迁移到 `%USERPROFILE%\.config\llm-wiki-mvp\config.json`。

**校验（强制执行）：** 在首次读写仓库内文件之前，必须确认 `root` 下同时存在 `schema/AGENTS.md` 和 `wiki/`。若任一缺失，停止执行并提示：配置文件中的 `root` 不是有效的 `llm-wiki-mvp` 仓库根；不得自动创建 `wiki/`、`raw/`、`schema/` 等目录。

**配置示例（Windows PowerShell）：**

```powershell
$configDir = Join-Path $env:USERPROFILE '.config\llm-wiki-mvp'
New-Item -ItemType Directory -Force -Path $configDir | Out-Null
$config = @{ root = 'E:\code\llm-wiki-mvp' } | ConvertTo-Json
Set-Content -Encoding UTF8 (Join-Path $configDir 'config.json') $config
```

**快速验证（PowerShell）：**

```powershell
$config = Get-Content (Join-Path $env:USERPROFILE '.config\llm-wiki-mvp\config.json') -Raw | ConvertFrom-Json
Test-Path (Join-Path $config.root 'schema/AGENTS.md')
Test-Path (Join-Path $config.root 'wiki')
```

## 输入

- **必需：** 任务名称或任务描述。
- **可选：** 用户指定的关键词、相关页面、项目路径、完成标准或约束。
- **可选模式：** `web` 或 `local`。

## 模式选择

- 用户明确提到 `ChatGPT 网页`、`网页版 AI`、`复制给网页`、`Projects`、`网页上下文包` 时，使用 `web` 模式。
- 用户在 Codex、本地仓库或当前工作区中要求继续做任务、实现、验证、修复、写代码时，使用 `local` 模式。
- 用户未指定模式时，默认使用 `local` 模式；但若任务描述明显是给网页 AI 准备输入，则使用 `web` 模式。

## 触发场景

当用户说出以下意图时使用：

- `task-start`
- `task-start web`
- `task-start local`
- 我要开始一个新任务
- 先查一下我的 wiki
- 这个任务和以前的某个主题有关
- 请先帮我做任务启动
- 为这个任务生成一份给 ChatGPT 网页版使用的上下文包

## 必读（开始前）

1. `schema/AGENTS.md`
2. `wiki/index.md`

随后按任务需要读取相关页面；不要泛读无关长文。

## 检索顺序

1. 先从 `wiki/index.md` 定位可能相关的入口。
2. 优先检索 `wiki/topics/`、`wiki/concepts/`、`wiki/entities/`、`wiki/analyses/`。
3. 必要时查阅 `wiki/sources/`，用于确认来源或补充上下文。
4. 若用户提供关键词，结合关键词在 `wiki/` 中补充搜索。
5. 若找不到相关内容，明确说明缺口，不编造 wiki 中不存在的结论。

## `web` 模式输出格式

`web` 模式的输出必须是一份可直接复制到网页 AI 的 Markdown。不要包含本地 agent 的执行日志、工具调用说明、文件系统细节或“我接下来会执行”的话术。

```md
# 给 ChatGPT 网页版的任务上下文包：{{任务名}}

## 项目背景

## 当前任务

## 相关 Wiki 页面 / 仓库模块

## 已有约束

## 设计原则

## 已知坑

## 验收标准

## 希望你帮我做什么
```

`web` 模式输出后必须停止，不进入本地实现、验证或写入流程。

## `local` 模式输出格式

```md
# Task Start: {{任务名}}

## 1. 任务理解

## 2. 相关 wiki 页面

## 3. 已有上下文

## 4. 可复用结论

## 5. 当前缺口

## 6. 本次建议上下文包

## 7. 下一步建议
```

`local` 模式的输出可以面向当前 agent 使用，允许包含本地执行建议、相关文件路径和下一步计划。若用户已经要求执行任务，完成启动上下文后可以继续进入计划或实现流程，不强制暂停。

## 工作要求

- `web` 模式只输出网页 AI 上下文包，不直接开始执行任务。
- `local` 模式输出本地任务启动上下文；如果用户任务是执行型，可以继续推进后续工作。
- 结论优先回指已有 wiki 页面。
- 对覆盖不足、冲突或低置信度内容明确标出。
- 建议上下文包应短而可用；`web` 模式尤其要避免塞入不能复用的过程信息。

## 禁止写入

执行本 skill 时不得创建或修改：

- `raw/`
- `wiki/`
- `logs/`
- `schema/`
- `skills/`

若任务启动阶段发现值得沉淀的新内容，只在输出中建议后续使用 `task-close` 或 `ingest`。
