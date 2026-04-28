---
name: lint
description: Run repository-local lint workflow for this LLM Wiki.
---

# Lint

本 skill 用于对当前 `wiki/` 做仓库级健康检查，并按风险等级决定直接修复还是先报告。

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
