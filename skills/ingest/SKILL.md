---
name: ingest
description: Run repository-local ingest workflow for this LLM Wiki.
---

# Ingest

本 skill 用于把一个或一组 `raw/` 来源增量编译进当前仓库的 `wiki/`。

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
