---
name: query
description: Run repository-local query workflow for this LLM Wiki.
---

# Query

本 skill 用于基于当前仓库已有 `wiki/` 页面回答问题，并在需要时把高价值结论沉淀回知识库。

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

- 最小输入：一个问题
- 可选上下文：指定页面、目录、主题范围或预期输出形式

## 默认入口行为

- 默认先读 `wiki/index.md`
- 优先读现有 `wiki/` 页面，而不是直接回到 `raw/`
- 按问题需要读取相关 `wiki/topics/`、`wiki/concepts/`、`wiki/entities/`、`wiki/sources/`、`wiki/analyses/`

## 必读规则

开始执行前，必须先读取以下规则文件：

1. `schema/AGENTS.md`
2. `schema/workflows.md` 的 query 部分
3. `schema/templates/analysis-template.md`

## 回答与沉淀策略

- 普通回答：直接回答并引用已有 wiki 页面
- 高价值回答：新增或更新 `wiki/analyses/` 页面
- 若分析页改变结构结论：回写主题页或概念页
- 完成后更新 `wiki/index.md` 和 `logs/log.md`

## 工作要求

- 先从 `wiki/index.md` 找入口，再展开到相关页面
- 回答中的结论优先回指已有来源页、主题页、概念页、实体页或分析页
- 若创建分析页，结构应遵循 `schema/templates/analysis-template.md`
- 若分析页形成新的长期结论，应同步更新对应主题页或概念页，保持索引可发现
- 严格遵守 Obsidian 链接规范，优先使用 `[[文件名|显示文本]]`

## 信息缺口与禁止行为

- 覆盖不足时要明确说明信息缺口
- 除非用户明确要求，否则不要把重新读取 `raw/` 当主回答路径
- 不能回指现有 wiki 证据的推测，不要包装成确定结论
- 若现有页面之间存在冲突而无法裁决，要保留分歧并指出待补资料

## 验证

完成后至少确认以下结果：

- `wiki/index.md` 仍是默认入口，并能定位本次使用或新增的页面
- 若新增或更新 `wiki/analyses/`，索引中能发现该页面
- `logs/log.md` 已记录本次高价值沉淀或结构回写
- 回答正文明确说明了任何关键的信息缺口
