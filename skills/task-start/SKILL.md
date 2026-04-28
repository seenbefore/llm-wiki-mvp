---
name: task-start
description: Start an important task by reading the local LLM Wiki and producing a reusable context package before execution.
---

# Task Start

本 skill 用于在一个重要任务开始前，先从当前 LLM Wiki 中提取相关上下文，避免重复解释、重复推理和遗忘已有结论。

它只负责**任务启动**：读取 `wiki/`，生成上下文启动包；不执行任务本身，也不写入 `raw/`、`wiki/` 或 `logs/`。

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

## 触发场景

当用户说出以下意图时使用：

- `task-start`
- 我要开始一个新任务
- 先查一下我的 wiki
- 这个任务和以前的某个主题有关
- 请先帮我做任务启动

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

## 输出格式

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

## 工作要求

- 只输出任务启动上下文，不直接开始执行任务。
- 结论优先回指已有 wiki 页面。
- 对覆盖不足、冲突或低置信度内容明确标出。
- 建议上下文包应短而可用，适合作为后续任务执行的输入。

## 禁止写入

执行本 skill 时不得创建或修改：

- `raw/`
- `wiki/`
- `logs/`
- `schema/`
- `skills/`

若任务启动阶段发现值得沉淀的新内容，只在输出中建议后续使用 `task-close` 或 `ingest`。
