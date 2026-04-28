---
name: task-close
description: Close an important task by deciding what should be preserved and writing a raw task note for later ingest into the LLM Wiki.
---

# Task Close

本 skill 用于在一个重要任务结束时，判断本次任务是否值得沉淀，并把值得保留的内容写入 `raw/task-notes/`，作为后续 `ingest` 的原始资料。

它默认**不直接写 `wiki/`**。知识层更新应由 `ingest` 处理。

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

- **必需：** 本次任务的目标、过程或对话上下文。
- **可选：** 用户指定的任务名、日期、相关页面、是否需要立即写入 raw。

## 触发场景

当用户说出以下意图时使用：

- `task-close`
- 总结一下这次任务
- 判断是否值得沉淀到 wiki
- 生成任务记录
- 更新我的 LLM-wiki 的原始资料

## 必读（开始前）

1. `schema/AGENTS.md`
2. `wiki/index.md`

读取它们是为了遵守仓库边界和判断本次任务可能影响哪些现有知识；不要在本 skill 中直接修改 `wiki/`。

## 判断标准

值得沉淀的内容包括：

- 新结论、新流程、新决策
- 可复用命令、配置、排错步骤
- 重要取舍和失败原因
- 会影响后续任务启动的背景
- 对 `schema/` 或 `skills/` 的改进建议

不建议沉淀的内容包括：

- 临时闲聊
- 一次性中间输出
- 没有复用价值的完整聊天记录
- 无法回忆上下文或无法验证的推测

## 默认写入位置

默认新增一份原始任务记录：

```text
raw/task-notes/YYYY-MM-DD-<task-slug>.md
```

文件名规则：

- 日期使用当前日期。
- `<task-slug>` 使用简短 kebab-case 英文或拼音。
- 若无法安全命名，先在输出中给出建议路径并等待用户确认。

## Raw 记录模板

```md
# {{任务标题}}

## Metadata

- Date: {{YYYY-MM-DD}}
- Type: task-note
- Status: raw
- Suggested ingest target: {{建议的 wiki 落点或待判断}}

## Task Goal

## What Happened

## Reusable Conclusions

## Commands / Artifacts

## Decisions

## Open Questions

## Suggested Next Step

- Run `ingest raw/task-notes/{{文件名}}` if this should enter the wiki knowledge layer.
```

## 输出格式

```md
# Task Close: {{任务名}}

## 1. 是否值得沉淀

## 2. 建议写入位置

## 3. 已写入或建议写入的 raw 内容

## 4. 不建议写入的内容

## 5. 下一步 ingest 建议

## 6. Git commit message
```

## 写入边界

- 默认允许写入：`raw/task-notes/`
- 默认不直接写入：`wiki/`
- 默认不修改：`schema/`、`skills/`、`logs/`

若本次任务本身就是修改协议、schema 或 skill，则可以按用户任务要求修改对应文件；否则只把建议写入 raw 记录。

## 验证

完成后确认：

- 若值得沉淀，`raw/task-notes/` 下已有或已有建议路径。
- 未直接修改 `wiki/`。
- 输出中包含下一步 `ingest` 建议。
- 输出中包含建议 git commit message。
