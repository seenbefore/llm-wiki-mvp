# AGENTS

这个文件定义 Agent 在本仓库中的工作边界和默认流程。

## 目标

把 `raw/` 中的原始资料持续编译进 `wiki/`，形成可浏览、可追溯、可增量维护的知识库。

重要 AI 任务应接入任务生命周期：开始前用 `task-start` 从 `wiki/` 提取上下文，结束时用 `task-close` 把值得保留的结果写入 `raw/task-notes/`，再按需要通过 `ingest` 进入知识层。

## 目录边界

- `raw/` 是原始资料层。Agent 不直接改写已有原始内容；`task-close` 可在 `raw/task-notes/` 新增任务记录。
- `wiki/` 是知识层。Agent 主要在这里新增和更新页面。
- `schema/` 是规则层。仅在工作流或模板需要演进时更新。
- `logs/` 是时间线层。重要 ingest、query 落档和 lint 都要记一笔。

## 默认行为

1. 重要任务开始前，先用 `task-start` 只读 `wiki/`，生成任务上下文启动包。
2. 重要任务结束时，用 `task-close` 判断是否值得沉淀；默认只新增 `raw/task-notes/` 原始任务记录。
3. 处理新资料或 task note 时，先在 `wiki/sources/` 生成来源摘要页。
4. 再判断是否需要更新 `wiki/topics/`、`wiki/entities/`、`wiki/concepts/`。
5. 若用户问题形成可复用结论，把结果沉淀到 `wiki/analyses/`。
6. 每次重要变更后更新 `wiki/index.md` 和 `logs/log.md`。

## 写作规则

- 优先做增量更新，不整页重写已有知识页。
- 结论尽量附上来源页链接，避免“无出处综合”。
- 对来源不足或置信度不够的内容，明确写出不确定性。
- 对互相冲突的说法，不直接覆盖旧结论，要标注 `冲突/待核实`。
- 保持页面可读性，避免把聊天口吻原样写入 Wiki。

## 链接约定

- 页面之间尽量使用相对路径或 Obsidian 风格双链。
- 文件名统一优先使用 `kebab-case`，例如 `llm-wiki.md`、`andrej-karpathy.md`。
- 当显示名和文件名不一致时，使用 `[[文件名|显示名]]`，例如 `[[llm-wiki|LLM Wiki]]`。
- 只有当笔记的真实文件名或显式 `aliases` 与链接目标完全匹配时，才能直接写 `[[目标名]]`。
- 不要把纯展示标题直接当作链接目标，否则 Obsidian 在找不到同名笔记时会创建空白文件。
- 来源优先链接到 `wiki/sources/` 页面，再由来源页回指 `raw/` 文件。
- 新建页面后，确保至少被 `wiki/index.md` 或某个上层主题页链接到，避免孤儿页。

## 页面选择原则

- `sources/`：单一来源摘要。
- `topics/`：跨来源的长期综合主题。
- `entities/`：稳定对象，如人、组织、工具、作品。
- `concepts/`：抽象概念、方法、术语。
- `analyses/`：高价值问答沉淀、比较、阶段性结论。

## 低置信度处理

遇到以下情况时，不要强行写进综合页：

- 资料过薄，只有零散片段
- 来源之间明显矛盾且未核实
- 只能做推测，不能回指来源

此时可以只生成 `sources/` 页，并在结尾注明后续需要补什么资料。

## 工作顺序

1. 阅读 `schema/workflows.md`
2. 参考 `schema/templates/`
3. 执行 task-start、task-close、ingest、query 或 lint
4. 按对应工作流判断是否需要更新 `wiki/index.md`
5. 按对应工作流判断是否需要记录 `logs/log.md`

## Skill 调度

当任务明确属于 task-start、task-close、ingest、query、lint 时，优先调用对应本地 skill，而不是临时自由发挥。
