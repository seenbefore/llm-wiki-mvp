# 设计：通过用户配置文件指定 LLM Wiki 仓库根路径

**日期：** 2026-04-11  
**状态：** 已更新（Config Only，替代旧环境变量方案）

## 背景

Skill 文件可放在用户级目录（全局加载），但 `task-start` / `task-close` / `ingest` / `query` / `lint` 的读写目标应是 **llm-wiki-mvp** 仓库，而非当前打开的任意项目。过去依赖 `LLM_WIKI_ROOT` 环境变量；如果环境变量失效、未被子进程继承或被清理，skill 容易回退到错误工作区。

## 目标

- 在任意工作区打开会话时，Agent 都能通过一个稳定的用户目录配置文件找到真实的 `llm-wiki-mvp` 克隆路径。
- 未配置或配置错误时明确停止，避免静默写错项目目录。
- 配置方式可审计、跨 shell 稳定，不依赖环境变量继承。

## 方案概述

使用用户目录 JSON 配置文件指向 `llm-wiki-mvp` 仓库根目录。

- **配置文件：** `%USERPROFILE%\.config\llm-wiki-mvp\config.json`
- **字段名：** `root`
- **含义：** 本机上一份 `llm-wiki-mvp` 克隆的根目录（包含 `schema/AGENTS.md`、`wiki/`、`raw/` 等）。

配置示例：

```json
{
  "root": "E:\\code\\llm-wiki-mvp"
}
```

## 解析规则

1. Skill 必须先读取 `%USERPROFILE%\.config\llm-wiki-mvp\config.json`。
2. 若配置文件存在、JSON 有效，且 `root` 为非空字符串，所有相对路径均解释为 `root` 下的路径。
3. 若配置文件不存在、JSON 无法解析、`root` 缺失或为空，停止执行并提示用户创建或修复配置文件。
4. 旧环境变量 `LLM_WIKI_ROOT` 已弃用；即使存在，也不得作为路径来源。若检测到它，请提示用户迁移到用户配置文件。

## 校验与失败提示

首次读写仓库内文件之前，必须确认：

- `root/schema/AGENTS.md` 存在。
- `root/wiki/` 存在。

若任一缺失，停止执行并提示：配置文件中的 `root` 不是有效的 `llm-wiki-mvp` 仓库根。不得自动创建 `wiki/`、`raw/`、`schema/` 等目录。

错误提示应区分：

- 配置文件不存在。
- JSON 无法解析。
- `root` 缺失或为空。
- `root` 不是有效的 `llm-wiki-mvp` 仓库根，或误指向 `wiki/` 子目录。

## 用户文档示例

Windows PowerShell：

```powershell
$configDir = Join-Path $env:USERPROFILE '.config\llm-wiki-mvp'
New-Item -ItemType Directory -Force -Path $configDir | Out-Null
$config = @{ root = 'E:\code\llm-wiki-mvp' } | ConvertTo-Json
Set-Content -Encoding UTF8 (Join-Path $configDir 'config.json') $config
```

快速验证：

```powershell
$config = Get-Content (Join-Path $env:USERPROFILE '.config\llm-wiki-mvp\config.json') -Raw | ConvertFrom-Json
Test-Path (Join-Path $config.root 'schema/AGENTS.md')
Test-Path (Join-Path $config.root 'wiki')
```

## 安全与协作注意

- 配置文件不提交到仓库；每台机器自行配置。
- 路径指向的是用户本机克隆，无额外机密；仍需避免在公开聊天中粘贴含敏感信息的绝对路径。
- 多根工作区不参与根路径选择；配置文件是唯一有效来源。

## 后续实现项

1. 在 `task-start` / `task-close` / `ingest` / `query` / `lint` 的 `SKILL.md` 中增加「Wiki 根路径（用户配置文件）」规则。
2. 更新 `README.md` 的跨项目使用说明。
3. 保留 `LLM_WIKI_ROOT` 的弃用提示，但不再将其作为 fallback。
