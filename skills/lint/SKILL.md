---
name: lint
description: Run repository-local lint workflow for this LLM Wiki.
---

# Lint

本 skill 用于对当前 `wiki/` 做仓库级健康检查，并按风险等级决定直接修复还是先报告。

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

- 最小输入：可为空
- 默认无输入时巡检全库
- 允许限定到某一目录或一批页面

## 必读规则

开始执行前，必须先读取以下规则文件：

1. `schema/AGENTS.md`
2. `schema/workflows.md` 的 lint 部分
3. `schema/lint-checklist.md`
4. `wiki/index.md`

## 检查清单

至少检查以下项目：

- 孤儿页
- 无来源结论
- 重复概念/实体
- 索引遗漏
- 错链
- frontmatter 问题
- 未标记冲突
- 不符合 Obsidian 链接规范的双链

## 工作要求

- 默认从 `wiki/index.md` 反查索引覆盖范围
- 结合 `schema/lint-checklist.md` 做巡检，不只看链接存在与否
- 发现问题后，先判断能否安全直接修复
- 任何修复或发现都应在需要时记录到 `logs/log.md`
- 涉及链接修复时，严格遵守 `[[文件名|显示文本]]` 与 `aliases` 规范

## 风险分级

- 低风险：可直接修
- 中风险：结构回写，谨慎处理
- 高风险：删除页、合并概念、重写结论，必须报告

## 处理边界

- 不把高风险操作当作默认修复动作
- 对是否重复、是否冲突存在不确定性时，先报告证据和信息缺口
- 若发现旧结论被新来源挑战但尚未标记，应补 `冲突/待核实` 或报告待处理

## 验证

完成后至少确认以下结果：

- 已覆盖 `schema/lint-checklist.md` 的核心检查项
- 若修复了问题，`wiki/index.md` 与相关页面保持一致
- 若发现高风险问题，输出中已明确报告而不是直接改写
- 若有重要巡检结论，已更新 `logs/log.md`
