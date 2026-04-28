# Log

按时间倒序或顺序追加都可以，但整个仓库需要保持一致。

推荐格式：

```md
## [YYYY-MM-DD] ingest | 标题
- 新增：[[某来源页]]
- 更新：[[某主题页]]
- 备注：是否存在待核实点
```

## [2026-04-07] bootstrap | 初始化仓库
- 新增：README、目录骨架、模板、Agent 规则、工作流说明
- 备注：MVP 结构已可用于第一轮 ingest

## [2026-04-07] ingest | LLM Wiki
- 新增：[[2026-04-07-karpathy-llm-wiki]]
- 更新：[[llm-wiki|LLM Wiki]]、[[RAG]]、[[persistent-knowledge-wiki|Persistent Knowledge Wiki]]、[[andrej-karpathy|Andrej Karpathy]]、`wiki/index.md`
- 备注：本次为第一轮示例 ingest；当前综合结论仍主要来自单一概念来源，后续需要补实践型材料。

## [2026-04-07] ingest | Obsidian linking conventions
- 新增：[[2026-04-07-obsidian-internal-links|Internal links - Obsidian Help]]、[[2026-04-07-obsidian-aliases|Aliases - Obsidian Help]]、[[obsidian-linking-conventions|Obsidian 链接规范]]
- 更新：`README.md`、`schema/AGENTS.md`、`schema/templates/`、`wiki/index.md`
- 备注：这次 ingest 直接用于修正当前仓库的链接规范，核心结论是“真实目标与显示文本分离”，并通过 `aliases` 提供稳定别名。
