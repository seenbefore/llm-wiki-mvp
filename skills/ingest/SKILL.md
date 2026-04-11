---
name: ingest
description: Run repository-local ingest workflow for this LLM Wiki.
---

# Ingest

本 skill 用于把一个或一组 `raw/` 来源增量编译进当前仓库的 `wiki/`。

## Wiki 根路径（LLM_WIKI_ROOT）

本 skill 中的相对路径（如 `schema/AGENTS.md`、`wiki/`、`raw/`、`logs/`）默认相对于 **当前 Cursor 工作区根目录**。若你在其他项目仓库中打开会话，且需要把读写到本机某份 `llm-wiki-mvp` 克隆，请设置环境变量：

- **变量名：** `LLM_WIKI_ROOT`
- **含义：** 本机 `llm-wiki-mvp` 仓库根的绝对路径（该根下须存在 `schema/AGENTS.md`、`wiki/` 等）。

**解析规则：**

1. 若 `LLM_WIKI_ROOT` 已设置且非空，本 skill 中所有相对路径均相对于该路径解析（Windows：将子路径与 `$env:LLM_WIKI_ROOT` 拼接；macOS/Linux：等价路径拼接）。
2. 若未设置，相对路径相对于当前工作区根目录（与直接在本仓库根打开工作区时使用本 skill 的行为一致）。

**校验（强制执行）：** 当依赖 `LLM_WIKI_ROOT` 时，在首次读写该仓库内文件之前，必须先确认该根下存在 `schema/AGENTS.md`。若不存在，停止执行并提示用户：`LLM_WIKI_ROOT` 指向的目录不是有效的 llm-wiki-mvp 仓库；**不得**在该根下新建 `wiki/` 等目录。

**配置示例（Windows PowerShell，用户级持久，重启 Cursor 或新终端后生效）：**

```powershell
[System.Environment]::SetEnvironmentVariable('LLM_WIKI_ROOT', 'E:\path\to\llm-wiki-mvp', 'User')
```

将示例中的路径改为你本机克隆路径。会话级仅当前进程可用：`$env:LLM_WIKI_ROOT = 'E:\path\to\llm-wiki-mvp'`。

**快速验证（PowerShell）：** `Test-Path (Join-Path $env:LLM_WIKI_ROOT 'schema/AGENTS.md')` 应为 `True`。

## 输入

- 最小输入：一个或一组 `raw/` 来源路径
- 可选上下文：用户指定的主题、实体、概念或预期落点

## 输出

- 默认输出：新增或更新一个 `wiki/sources/` 来源页、更新 `wiki/index.md`、追加 `logs/log.md`
- 可选输出：按需增量更新 `wiki/topics/`、`wiki/concepts/`、`wiki/entities/`

## 必读规则

开始执行前，必须先读取以下规则文件：

1. `schema/AGENTS.md`
2. `schema/workflows.md` 的 ingest 部分
3. `schema/templates/source-template.md`
4. `schema/templates/topic-template.md`
5. `schema/templates/entity-template.md`
6. `schema/templates/concept-template.md`

## 执行顺序

1. 读取 `schema/AGENTS.md`
2. 读取 `schema/workflows.md` 的 ingest 部分
3. 读取相关模板
4. 先生成或更新 `wiki/sources/` 页面
5. 再决定是否更新 `wiki/topics/`、`wiki/concepts/`、`wiki/entities/`
6. 更新 `wiki/index.md`
7. 追加 `logs/log.md`

## 工作要求

- 先做来源页，再做综合页；信息不足时只落来源页
- 综合结论优先回指来源页，而不是直接回指 `raw/`
- 对已有知识页只做增量更新，不整页重写
- 需要更新主题、概念、实体时，遵循 `schema/templates/` 的最小结构
- 新建页面后，确保它被 `wiki/index.md` 或上层页面链接，避免孤儿页

## 失败处理与边界

- 不修改 `raw/`
- 信息不足时只落来源页
- 冲突未解时标 `冲突/待核实`
- 不能回指来源的推测，不写入综合结论
- 严格遵守 `[[文件名|显示文本]]` 与 `aliases` 规范
- 只有真实文件名或显式 `aliases` 完全匹配时，才能使用 `[[目标名]]`

## 验证

完成后至少确认以下结果：

- `wiki/sources/` 页面已创建或已更新
- `wiki/index.md` 已反映本次新增或重要更新
- `logs/log.md` 已追加一条 ingest 记录
- 若有冲突，页面中明确标出 `冲突/待核实`
