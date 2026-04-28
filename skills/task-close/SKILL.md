---
name: task-close
description: Close an important task by deciding what should be preserved and writing a raw task note for later ingest into the LLM Wiki.
---

# Task Close

本 skill 用于在一个重要任务结束时，判断本次任务是否值得沉淀，并把值得保留的内容写入 `raw/task-notes/`，作为后续 `ingest` 的原始资料。

它默认**不直接写 `wiki/`**。知识层更新应由 `ingest` 处理。

## Wiki 根路径（LLM_WIKI_ROOT）

本 skill 中的相对路径（如 `schema/AGENTS.md`、`raw/`、`wiki/`）默认相对于 **当前工作区根目录**。若你在其他项目仓库中打开会话，且需要写入本机某份 `llm-wiki-mvp` 克隆，请设置环境变量：

- **变量名：** `LLM_WIKI_ROOT`
- **含义：** 本机 `llm-wiki-mvp` 仓库根的绝对路径（该根下须存在 `schema/AGENTS.md`、`raw/`、`wiki/` 等）。

**解析规则：**

1. 若 `LLM_WIKI_ROOT` 已设置且非空，本 skill 中所有相对路径均相对于该路径解析。
2. 若未设置，相对路径相对于当前工作区根目录。

**校验（强制执行）：** 当依赖 `LLM_WIKI_ROOT` 时，在首次读写该仓库内文件之前，必须先确认该根下存在 `schema/AGENTS.md`。若不存在，停止执行并提示用户：`LLM_WIKI_ROOT` 指向的目录不是有效的 llm-wiki-mvp 仓库；不得在该根下新建 `raw/`、`wiki/` 等目录。

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

