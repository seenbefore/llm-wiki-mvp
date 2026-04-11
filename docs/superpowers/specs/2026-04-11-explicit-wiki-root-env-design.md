# 设计：通过环境变量显式指定 LLM Wiki 仓库根路径

**日期：** 2026-04-11  
**状态：** 草案（供全局 skill 与用户文档对齐）

## 背景

Skill 文件可放在用户级目录（全局加载），但 `ingest` / `query` / `lint` 的读写目标应是 **llm-wiki-mvp** 仓库，而非当前打开的任意项目（例如「甲项目」）。相对路径默认相对 **Cursor 工作区根**，在甲项目中会错误地指向甲目录。

## 目标

- 在任意工作区打开会话时，只要本机配置正确，Agent 仍能将 `wiki/`、`raw/`、`schema/`、`logs/` 解析到 **真实的 llm-wiki-mvp 克隆路径**。
- 配置方式简单、可审计：不依赖把 wiki 嵌进甲仓库，也不依赖多根工作区（可作为可选补充）。
- 未配置时行为明确：与「工作区即 wiki 根」一致，避免静默写错位置。

## 非目标

- 不在本规格中规定团队如何分发全局 `SKILL.md`（仅约定解析规则）。
- 不要求跨机器同步环境变量值（每人本机路径不同属正常）。

## 方案概述

使用 **单一环境变量** 指向 llm-wiki-mvp 仓库的**绝对路径**（目录级，无尾随反斜杠亦可）。

### 变量名

- **名称：** `LLM_WIKI_ROOT`
- **含义：** 本机上一份 `llm-wiki-mvp` 克隆的根目录（包含 `schema/AGENTS.md`、`wiki/`、`raw/` 等）。

备选：若未来与其它工具冲突，可加前缀（如 `ICINFO_LLM_WIKI_ROOT`）；在团队内统一即可。

### 解析规则（供全局 SKILL 引用）

1. **若** `LLM_WIKI_ROOT` 已设置且非空字符串：  
   - 所有 skill 中的相对路径（如 `schema/AGENTS.md`、`wiki/sources/`）均解释为：  
     `Join-Path $env:LLM_WIKI_ROOT '<相对路径>'`（Windows）或等价拼接（POSIX）。
2. **否则**：  
   - 相对路径解释为当前 **工作区根目录** 下的路径（与现有仓库内 skill 行为一致）。
3. **验证（建议强制执行）**：  
   - 当使用 `LLM_WIKI_ROOT` 时，在首次读写前检查路径  
     `Join-Path $env:LLM_WIKI_ROOT 'schema/AGENTS.md'`  
     是否存在。  
   - 若不存在：停止并提示用户变量指向了错误目录或未完整克隆，**禁止**在未知根下创建 `wiki/` 等目录。

### 失败与提示文案（建议）

- 变量已设置但 `schema/AGENTS.md` 缺失：  
  `LLM_WIKI_ROOT` 指向的目录不是有效的 llm-wiki-mvp 仓库，请检查路径或重新克隆。
- 变量未设置且在非 wiki 工作区执行 ingest：  
  可提示（可选）：若意图操作另一路径下的 wiki，请设置 `LLM_WIKI_ROOT`；否则请将工作区切换为 llm-wiki-mvp 根目录。

## 与用户文档的衔接

在面向贡献者的说明中增加 **Windows（PowerShell）** 持久化示例（用户级，重启后仍生效）：

```powershell
[System.Environment]::SetEnvironmentVariable('LLM_WIKI_ROOT', 'E:\icinfo\llm-wiki-mvp', 'User')
```

会话级（仅当前终端 / 当前 Cursor 子进程继承）可用：

```powershell
$env:LLM_WIKI_ROOT = 'E:\icinfo\llm-wiki-mvp'
```

Linux/macOS 可使用 `export LLM_WIKI_ROOT=/path/to/llm-wiki-mvp` 或写入 `~/.profile` 等，由团队文档统一即可。

## 安全与协作注意

- 环境变量**不**提交到仓库；每人本机路径不同，符合预期。
- 路径指向的是用户本机克隆，无额外机密；仍需避免在聊天中粘贴含敏感信息的绝对路径（常规操作习惯）。

## 与多根工作区关系

- **显式根路径**与**多根工作区**可并存：多根时若仍设置 `LLM_WIKI_ROOT`，以变量为准，避免「两个根里都有 `wiki/`」的歧义。
- 若未设置变量且使用多根工作区，全局 skill 必须额外规定「以哪一根为 wiki 根」——本规格推荐优先采用 `LLM_WIKI_ROOT` 消除歧义。

## 后续实现项（不在本文件内展开代码）

1. 在用户级 `ingest` / `query` / `lint` 的 `SKILL.md` 中增加一节「路径解析：LLM_WIKI_ROOT」，与上文规则逐字或等价对齐。
2. （可选）在本仓库 `README` 或 `schema/` 下增加简短「跨项目使用」说明，指向环境变量配置。
3. 实现或演练时可用 PowerShell 一行验证：  
   `Test-Path (Join-Path $env:LLM_WIKI_ROOT 'schema/AGENTS.md')`

## 自审（Spec Self-Review）

- **占位：** 无 TBD。
- **一致性：** 解析顺序与验证步骤一致；与「未设置则工作区根」兼容旧行为。
- **范围：** 仅约定根路径发现机制，不扩展 ingest 业务规则。
- **歧义：** 已明确「设置变量则以变量为准」；多根未设置变量时仍属未覆盖场景，建议用变量或切换单根 wiki 工作区。
