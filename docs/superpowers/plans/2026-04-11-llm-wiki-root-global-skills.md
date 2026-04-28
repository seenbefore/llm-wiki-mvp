# LLM_WIKI_ROOT 全局 Skill 路径解析 — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在仓库内三份 skill 源文件（`skills/*/SKILL.md`）中增加与规格一致的路径解析说明，并在根 `README.md` 增加跨项目使用提示；合并后用户通过 `openskills install -g -y https://github.com/seenbefore/llm-wiki-mvp` 安装或更新用户级全局 skill。

**Architecture:** 规格见 `docs/superpowers/specs/2026-04-11-explicit-wiki-root-env-design.md`。实现上仅在 Markdown 中增加**同一语义**的一节（三份 skill 文字等价），不引入脚本或代码。Agent 执行 skill 时按节内规则在首次读写前校验 `schema/AGENTS.md`。

**Tech Stack:** Markdown、`rg`（ripgrep）、PowerShell（本机验证可选）

---

## 文件结构（变更锁定）


| 文件                       | 职责                                 |
| ------------------------ | ---------------------------------- |
| `skills/ingest/SKILL.md` | ingest 的路径根解析与校验说明                 |
| `skills/query/SKILL.md`  | query 同上                           |
| `skills/lint/SKILL.md`   | lint 同上                            |
| `README.md`              | 贡献者可见的一句「跨项目请设 `LLM_WIKI_ROOT`」及指向 |


**不在本计划内：** 修改用户目录下的全局副本（由各人 `openskills install -g` 或 `openskills update` 从本仓库拉取）；不新增 CI。

---

### 共用段落：插入到三个 SKILL.md 的 Markdown 块

以下整段插入到每个文件的 **一级标题与 `## 输入` 之间**（即紧接在首段说明文字之后、`## 输入` 之前）。三份文件**逐字相同**。

```markdown
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
```

---

### Task 1: `skills/ingest/SKILL.md`

**Files:**

- Modify: `skills/ingest/SKILL.md`（在 `# Ingest` 下首段说明与 `## 输入` 之间插入「共用段落」）
- **Step 1: 插入 Wiki 根路径一节**

在下列两行之间插入上面「共用段落」全文（含外层 `## Wiki 根路径` 至末尾验证行）：

```markdown
本 skill 用于把一个或一组 `raw/` 来源增量编译进当前仓库的 `wiki/`。

## 输入
```

插入后，`## 输入` 紧跟共用段落的最后一行（空行后接 `## 输入` 亦可，保持与仓库其余 Markdown 空行风格一致即可）。

- **Step 2: 校验文件包含关键字**

Run:

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
rg -n "LLM_WIKI_ROOT|schema/AGENTS.md" "skills\ingest\SKILL.md"
```

Expected: 至少两行匹配，且文件中存在「校验（强制执行）」。

- **Step 3: Commit**

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
git add skills/ingest/SKILL.md
git commit -m "docs(skill): ingest 说明 LLM_WIKI_ROOT 路径解析"
```

---

### Task 2: `skills/query/SKILL.md`

**Files:**

- Modify: `skills/query/SKILL.md`（在首段与 `## 输入` 之间插入同一「共用段落」）
- **Step 1: 插入位置**

在下列两行之间插入与 Task 1 **完全相同**的「共用段落」：

```markdown
本 skill 用于基于当前仓库已有 `wiki/` 页面回答问题，并在需要时把高价值结论沉淀回知识库。

## 输入
```

- **Step 2: 校验**

Run:

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
rg -n "LLM_WIKI_ROOT" "skills\query\SKILL.md"
```

Expected: 至少一行匹配。

- **Step 3: Commit**

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
git add skills/query/SKILL.md
git commit -m "docs(skill): query 说明 LLM_WIKI_ROOT 路径解析"
```

---

### Task 3: `skills/lint/SKILL.md`

**Files:**

- Modify: `skills/lint/SKILL.md`（在首段与 `## 输入` 之间插入同一「共用段落」）
- **Step 1: 插入位置**

在下列两行之间插入与 Task 1 **完全相同**的「共用段落」：

```markdown
本 skill 用于对当前 `wiki/` 做仓库级健康检查，并按风险等级决定直接修复还是先报告。

## 输入
```

- **Step 2: 校验**

Run:

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
rg -n "LLM_WIKI_ROOT" "skills\lint\SKILL.md"
```

Expected: 至少一行匹配。

- **Step 3: Commit**

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
git add skills/lint/SKILL.md
git commit -m "docs(skill): lint 说明 LLM_WIKI_ROOT 路径解析"
```

---

### Task 4: `README.md` 一句说明

**Files:**

- Modify: `README.md`（`## Notes` 小节）
- **Step 1: 在 Notes 中新增一条 bullet**

在 `## Notes` 下、现有列表**末尾**追加一条（保持 `-`  列表格式），内容为：

```markdown
- 若在**其他项目工作区**中调用已安装的 ingest/query/lint，需要读写本机某份 `llm-wiki-mvp` 克隆，请设置环境变量 `LLM_WIKI_ROOT` 指向该仓库根；详见各 skill 内「Wiki 根路径（LLM_WIKI_ROOT）」一节。
```

- **Step 2: 校验**

Run:

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
rg -n "LLM_WIKI_ROOT" "README.md"
```

Expected: 至少一行匹配。

- **Step 3: Commit**

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
git add README.md
git commit -m "docs: README 说明跨项目使用 LLM_WIKI_ROOT"
```

---

### Task 5: 合并后使用者操作（文档化验证，无新文件）

**Files:** 无

- **Step 1: 全库字符串一致性抽查**

Run:

```powershell
Set-Location "e:\icinfo\llm-wiki-mvp"
rg "LLM_WIKI_ROOT" skills/ README.md
```

Expected: `skills/ingest/SKILL.md`、`skills/query/SKILL.md`、`skills/lint/SKILL.md`、`README.md` 均出现匹配。

- **Step 2: （可选）本机环境验证**

在已设置 `LLM_WIKI_ROOT` 的 PowerShell 会话中：

```powershell
Test-Path (Join-Path $env:LLM_WIKI_ROOT 'schema/AGENTS.md')
```

Expected: `True`（仅当变量指向本仓库或有效克隆时）。

- **Step 3: 记录发布侧提醒（不提交代码）**

合并并发布后，使用者若从远程安装 skill，应执行：

```powershell
openskills install -g -y https://github.com/seenbefore/llm-wiki-mvp
```

已经全局安装过的使用者可执行 `openskills update` 拉取更新后的 `SKILL.md`。不要使用默认的项目安装；`openskills sync` 只用于在某个工作区生成或更新 `AGENTS.md`，不是安装步骤。

---

## Self-Review（对照规格）


| 规格要求（见 `2026-04-11-explicit-wiki-root-env-design.md`） | 对应任务                    |
| ----------------------------------------------------- | ----------------------- |
| 变量名 `LLM_WIKI_ROOT`、未设置则工作区根                          | Task 1–3 共用段落           |
| 设置时校验 `schema/AGENTS.md` 存在                           | Task 1–3 共用段落「校验（强制执行）」 |
| 失败提示语义                                                | Task 1–3 共用段落           |
| PowerShell 持久化示例                                      | Task 1–3 共用段落           |
| 可选 README / schema 简短说明                               | Task 4                  |


**Placeholder 扫描：** 无 TBD/TODO。  
**类型/命名一致性：** 全仓库统一使用 `LLM_WIKI_ROOT`，与规格一致。

---

## Execution Handoff

**Plan complete and saved to `docs/superpowers/plans/2026-04-11-llm-wiki-root-global-skills.md`. Two execution options:**

**1. Subagent-Driven (recommended)** — Dispatch a fresh subagent per task, review between tasks, fast iteration.

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints.

**Which approach?**
