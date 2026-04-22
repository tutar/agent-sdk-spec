# Instruction Markdown

## 职责

`InstructionMarkdown` 定义 harness 如何发现、匹配、展开并注入 file-backed instruction markdown sources。

它覆盖：

- `AGENTS.md` 一类显式 instruction files
- local override / nested subtree instruction loading
- `@include` 或语义等价 include expansion
- `rules` / `paths` 一类 conditional instruction matching

它不覆盖：

- durable topic payload
- bounded durable memory recall
- memory consolidation / dream
- transcript / restore

## 核心结论

在 `agent-spec` 中，`AGENTS.md` 是通用命名。

默认本地映射里，它通常对应：

- project-level `AGENTS.md`
- local override file
- nested subtree `AGENTS.md`
- `.claude/rules/*.md`

但命名映射不改变 ownership：

- `AGENTS.md` / `rules`
  归 `Harness.ContextEngineering`
- `AutoMemory` / `TeamMemory` / `AgentMemory`
  归 `Session.durable-memory`

## 子页

- [instruction-markdown-loading.md](instruction-markdown-loading.md)
- [instruction-include-expansion.md](instruction-include-expansion.md)
- [conditional-instruction-rules.md](conditional-instruction-rules.md)
