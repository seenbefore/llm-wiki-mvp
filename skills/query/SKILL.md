---
name: query
description: Run repository-local query workflow for this LLM Wiki.
---

# Query

本 skill 用于基于当前仓库已有 `wiki/` 页面回答问题，并在需要时把高价值结论沉淀回知识库。

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
