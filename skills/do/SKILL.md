---
name: do
description: 根据任务在本地 llm-wiki wiki/ 中只读检索最佳实践并在当前工作区执行；无依据则说明未找到，有冲突或不足则暂停待用户选择。
---

# Do

本 skill 在用户给出**任务描述**时，先在 **llm-wiki-mvp 本地 `wiki/`** 中检索与任务相关的最佳实践，再决定如何在**当前 Cursor 工作区**落地执行。对知识库侧**只读**：不写入、不修改 `wiki/`、`raw/`、`logs/`。

与 `query` 的区别：`query` 可回答问题并**沉淀**分析页等；`do` **不得**向 `wiki/` 写入任何内容。

## Wiki 根路径（LLM_WIKI_ROOT）

本 skill 中指向知识库的相对路径（如 `schema/AGENTS.md`、`wiki/`）默认相对于 **当前 Cursor 工作区根目录**。若你在其它项目仓库中打开会话，且需要读取本机某份 `llm-wiki-mvp` 克隆下的 `wiki/`，请设置环境变量：

- **变量名：** `LLM_WIKI_ROOT`
- **含义：** 本机 `llm-wiki-mvp` 仓库根的绝对路径（该根下须存在 `schema/AGENTS.md`、`wiki/` 等）。

**解析规则：**

1. 若 `LLM_WIKI_ROOT` 已设置且非空，本 skill 中所有上述相对路径均相对于该路径解析（Windows：将子路径与 `$env:LLM_WIKI_ROOT` 拼接；macOS/Linux：等价路径拼接）。
2. 若未设置，相对路径相对于当前工作区根目录（与直接在本仓库根打开工作区时使用本 skill 的行为一致）。

**校验（强制执行）：** 当依赖 `LLM_WIKI_ROOT` 读取知识库时，在**首次**读取该仓库内文件之前，必须先确认该根下存在 `schema/AGENTS.md`。若不存在，停止执行并提示用户：`LLM_WIKI_ROOT` 指向的目录不是有效的 llm-wiki-mvp 仓库；**不得**在该根下新建 `wiki/` 等目录。

**配置示例（Windows PowerShell，用户级持久，重启 Cursor 或新终端后生效）：**

```powershell
[System.Environment]::SetEnvironmentVariable('LLM_WIKI_ROOT', 'E:\path\to\llm-wiki-mvp', 'User')
```

将示例中的路径改为你本机克隆路径。会话级仅当前进程可用：`$env:LLM_WIKI_ROOT = 'E:\path\to\llm-wiki-mvp'`。

**快速验证（PowerShell）：** `Test-Path (Join-Path $env:LLM_WIKI_ROOT 'schema/AGENTS.md')` 应为 `True`。

## 输入

- **必需：** 任务描述（要做什么、完成标准或约束）。
- **可选：** 用户指定优先检索的 wiki 页面路径或关键词。

## 必读（开始执行前）

1. `schema/AGENTS.md`（仓库总约束）
2. `wiki/index.md`（检索入口）

随后按下方「检索顺序」展开；**不要**在未读 `schema/AGENTS.md` 前修改工作区或知识库文件。

## 检索顺序

1. 阅读 `wiki/index.md`，沿 Obsidian 链接跳到与任务关键词相关的主题/概念/分析页。
2. 按任务关键词在 `wiki/topics/`、`wiki/concepts/`、`wiki/entities/` 中补充检索（可用搜索/列表工具）。
3. 必要时查阅 `wiki/sources/`、`wiki/analyses/` 中与任务**直接相关**的条目。
4. 避免为凑结论而泛读无关长文；检索范围应与当前任务直接相关。

## 判定分支

| 情况 | 行为 |
|------|------|
| 可归纳出**明确、可执行**且 wiki 条目间**无冲突**的实践 | 在当前工作区执行任务（改代码、运行实践要求的校验命令等）；回复中简要注明依据的 wiki 路径或标题 |
| **无任何**相关实践 | 明确说明：**未在 wiki 中找到**与任务相关的最佳实践；不编造 wiki 中不存在的结论 |
| 仅有部分相关、或多页**冲突**、或信息**不足以安全执行** | **不要**继续改代码或跑破坏性命令；用列表说明：信息缺口、冲突点、可选方向；**等待用户回复选择后再继续** |

本 skill **默认不**在「未找到」后自动改用通用经验冒充 wiki 结论；若用户在同一对话中明确要求，可另作处理。

## 工作区执行原则

- 修改范围限于完成用户任务所必需；风格与当前工作区仓库约定一致。
- 若 wiki 实践页**明确要求**特定验证步骤（如测试、lint 命令），则执行；无 wiki 依据则不扩大范围。

## 禁止写入

在执行本 skill 过程中，**不得**创建或修改：`wiki/`、`raw/`、`logs/` 下任何文件（含 `wiki/analyses/`、`wiki/index.md`）。工作区内的修改仅用于完成用户任务。

## 与其它技能的关系

| 技能 | 与本技能关系 |
|------|----------------|
| `ingest` | 独立；`do` 不编译 `raw/` 入 `wiki/` |
| `query` | `query` 可写 wiki（如分析页）；`do` **只读** wiki |
| `lint` | 独立；知识库 lint 不由 `do` 承担 |

## 实现后验证（手工）

在设置 `LLM_WIKI_ROOT` 指向本仓库克隆时各走查一次：

1. **有明确实践：** 任务能被 wiki 支撑 → 工作区有预期变更，且回复引用了 wiki 依据。
2. **无相关实践：** 回复明确「未找到」，且未伪造 wiki 条目。
3. **需用户决策：** 存在冲突或信息不足 → 暂停改代码，列出选项，待「用户回复」后再继续。

并确认整个过程中**未对 `wiki/` 产生写操作**。

## 触发示例

- 「使用 `do`：任务：______」
- 「按 do 流程：在 wiki 里找最佳实践后实现：______」
