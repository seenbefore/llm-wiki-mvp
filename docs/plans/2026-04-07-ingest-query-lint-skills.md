# Ingest Query Lint Skills Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 为当前 `LLM Wiki` 仓库落地 3 个本地 skill：`ingest`、`query`、`lint`，把已有 workflow、模板、索引、日志和 Obsidian 链接规范固化成稳定调用入口。

**Architecture:** 本次实现不新增总控命令层，直接在仓库内创建 3 个平级 skill。每个 skill 只定义自身输入、输出、执行步骤、失败处理和验证方式，并明确依赖现有 `schema/AGENTS.md`、`schema/workflows.md`、`schema/templates/` 作为规则底座。

**Tech Stack:** Markdown、仓库本地 SKILL 规范、PowerShell 验证命令、Obsidian Wikilink/aliases 规范

---

### Task 1: 创建 skills 目录骨架

**Files:**

- Create: `skills/ingest/SKILL.md`
- Create: `skills/query/SKILL.md`
- Create: `skills/lint/SKILL.md`

**Step 1: 创建空目录并确认路径**

Run: `Get-ChildItem -Name "E:\code\llm-wiki-mvp"`
Expected: 输出中还没有 `skills`

**Step 2: 新建 3 个 skill 目录**

Run: `New-Item -ItemType Directory -Force "E:\code\llm-wiki-mvp\skills\ingest","E:\code\llm-wiki-mvp\skills\query","E:\code\llm-wiki-mvp\skills\lint"`
Expected: 3 个目录创建成功

**Step 3: 新建 3 个 `SKILL.md` 占位文件**

实现最小文件头：

```md
---
name: ingest
description: Run repository-local ingest workflow for this LLM Wiki.
---
```

同理为 `query` 和 `lint` 创建对应文件头。

**Step 4: 验证文件存在**

Run: `Get-ChildItem -Recurse -File -Name "E:\code\llm-wiki-mvp\skills"`
Expected: 至少看到 `ingest\SKILL.md`、`query\SKILL.md`、`lint\SKILL.md`

**Step 5: Commit**

```bash
git add skills
git commit -m "feat: add local skill scaffolds"
```

### Task 2: 实现 `ingest` skill 文档

**Files:**

- Modify: `skills/ingest/SKILL.md`
- Reference: `schema/AGENTS.md`
- Reference: `schema/workflows.md`
- Reference: `schema/templates/source-template.md`
- Reference: `schema/templates/topic-template.md`
- Reference: `schema/templates/entity-template.md`
- Reference: `schema/templates/concept-template.md`
- Reference: `wiki/index.md`
- Reference: `logs/log.md`

**Step 1: 写出 `ingest` skill 的职责与输入**

在 `skills/ingest/SKILL.md` 中明确：

```md
- 最小输入：一个或一组 `raw/` 来源路径
- 默认输出：来源页、索引更新、日志更新
- 可选输出：主题页/概念页/实体页增量更新
```

**Step 2: 写出执行顺序**

将以下顺序写入 skill：

```md
1. 读取 `schema/AGENTS.md`
2. 读取 `schema/workflows.md` 的 ingest 部分
3. 读取相关模板
4. 先生成 `wiki/sources/` 页面
5. 再决定是否更新主题/概念/实体页
6. 更新 `wiki/index.md`
7. 追加 `logs/log.md`
```

**Step 3: 写出失败处理与边界**

明确写入：

```md
- 不修改 `raw/`
- 信息不足时只落来源页
- 冲突未解时标 `冲突/待核实`
- 严格遵守 `[[文件名|显示文本]]` 与 `aliases` 规范
```

**Step 4: 运行内容校验**

Run: `rg "sources|index|logs/log.md|冲突/待核实" "E:\code\llm-wiki-mvp\skills\ingest\SKILL.md"`
Expected: 命中 `ingest` 的核心职责、索引、日志、冲突处理说明

**Step 5: Commit**

```bash
git add skills/ingest/SKILL.md
git commit -m "feat: define ingest skill workflow"
```

### Task 3: 实现 `query` skill 文档

**Files:**

- Modify: `skills/query/SKILL.md`
- Reference: `schema/AGENTS.md`
- Reference: `schema/workflows.md`
- Reference: `schema/templates/analysis-template.md`
- Reference: `wiki/index.md`
- Reference: `wiki/analyses/`
- Reference: `logs/log.md`

**Step 1: 写出 `query` 的默认入口行为**

在 `skills/query/SKILL.md` 中明确：

```md
- 最小输入：一个问题
- 默认先读 `wiki/index.md`
- 优先读现有 wiki 页面，而不是直接回到 `raw/`
```

**Step 2: 写出沉淀策略**

将以下规则写入 skill：

```md
- 普通回答：直接回答并引用已有 wiki 页面
- 高价值回答：新增或更新 `wiki/analyses/` 页面
- 若分析页改变结构结论：回写主题页或概念页
- 完成后更新索引和日志
```

**Step 3: 写出信息缺口与禁止行为**

明确写入：

```md
- 覆盖不足时要明确说明信息缺口
- 除非用户明确要求，否则不要把重新读取 `raw/` 当主回答路径
```

**Step 4: 运行内容校验**

Run: `rg "wiki/index.md|wiki/analyses|logs/log.md|信息缺口|raw/" "E:\code\llm-wiki-mvp\skills\query\SKILL.md"`
Expected: 命中索引优先、分析页沉淀、日志写入、信息缺口和 raw 限制

**Step 5: Commit**

```bash
git add skills/query/SKILL.md
git commit -m "feat: define query skill workflow"
```

### Task 4: 实现 `lint` skill 文档

**Files:**

- Modify: `skills/lint/SKILL.md`
- Reference: `schema/AGENTS.md`
- Reference: `schema/workflows.md`
- Reference: `schema/lint-checklist.md`
- Reference: `wiki/index.md`
- Reference: `logs/log.md`

**Step 1: 写出 `lint` 的默认行为**

在 `skills/lint/SKILL.md` 中明确：

```md
- 最小输入：可为空
- 默认无输入时巡检全库
- 允许限定到某一目录或一批页面
```

**Step 2: 写出检查清单**

至少包含：

```md
- 孤儿页
- 无来源结论
- 重复概念/实体
- 索引遗漏
- 错链
- frontmatter 问题
- 未标记冲突
- 不符合 Obsidian 链接规范的双链
```

**Step 3: 写出风险分级**

明确写入：

```md
- 低风险：可直接修
- 中风险：结构回写，谨慎处理
- 高风险：删除页、合并概念、重写结论，必须报告
```

**Step 4: 运行内容校验**

Run: `rg "孤儿页|frontmatter|Obsidian|低风险|高风险" "E:\code\llm-wiki-mvp\skills\lint\SKILL.md"`
Expected: 命中检查清单和风险分级

**Step 5: Commit**

```bash
git add skills/lint/SKILL.md
git commit -m "feat: define lint skill workflow"
```

### Task 5: 补充仓库级发现入口

**Files:**

- Modify: `README.md`
- Modify: `schema/AGENTS.md`
- Modify: `docs/superpowers/specs/2026-04-07-ingest-query-lint-skills-design.md`

**Step 1: 在 `README.md` 加一段 skill 入口说明**

补充：

```md
## Local Skills

- `skills/ingest/`
- `skills/query/`
- `skills/lint/`
```

并说明这 3 个 skill 是当前仓库的稳定操作入口。

**Step 2: 在 `schema/AGENTS.md` 增加调度说明**

补充一段：

```md
当任务明确属于 ingest、query、lint 时，优先调用对应本地 skill，而不是临时自由发挥。
```

**Step 3: 在 spec 中标记实现入口位置**

在设计文档中补一句，说明实际实现目录是 `skills/ingest/`、`skills/query/`、`skills/lint/`。

**Step 4: 运行内容校验**

Run: `rg "skills/ingest|skills/query|skills/lint|本地 skill" "E:\code\llm-wiki-mvp\README.md" "E:\code\llm-wiki-mvp\schema\AGENTS.md" "E:\code\llm-wiki-mvp\docs\superpowers\specs\2026-04-07-ingest-query-lint-skills-design.md"`
Expected: 三处都能命中本地 skill 入口说明

**Step 5: Commit**

```bash
git add README.md schema/AGENTS.md docs/superpowers/specs/2026-04-07-ingest-query-lint-skills-design.md
git commit -m "docs: document local skill entrypoints"
```

### Task 6: 做一次仓库级 smoke check

**Files:**

- Verify: `skills/ingest/SKILL.md`
- Verify: `skills/query/SKILL.md`
- Verify: `skills/lint/SKILL.md`
- Verify: `README.md`
- Verify: `schema/AGENTS.md`

**Step 1: 检查 3 个 skill 是否都存在**

Run: `Get-ChildItem -Recurse -File -Name "E:\code\llm-wiki-mvp\skills"`
Expected: 看到 3 个 `SKILL.md`

**Step 2: 检查 3 个 skill 是否都引用了仓库规则**

Run: `rg "schema/AGENTS.md|schema/workflows.md|schema/templates|wiki/index.md|logs/log.md" "E:\code\llm-wiki-mvp\skills"`
Expected: 3 个 skill 都命中其依赖规则文件

**Step 3: 检查是否仍符合 Obsidian 链接规范**

Run: `rg "\[\[(LLM Wiki|Andrej Karpathy|Persistent Knowledge Wiki)\]\]" "E:\code\llm-wiki-mvp\skills" "E:\code\llm-wiki-mvp\README.md" "E:\code\llm-wiki-mvp\schema"`
Expected: 不应出现把展示标题直接作为仓库真实跳转目标的写法

**Step 4: 检查 lints**

Run: 使用 IDE diagnostics / `ReadLints`
Expected: 无新增诊断问题

**Step 5: Commit**

```bash
git add skills README.md schema/AGENTS.md
git commit -m "feat: add local ingest query lint skills"
```

