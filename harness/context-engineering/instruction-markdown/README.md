# Instruction Markdown

## 职责

`AgentRuntime` 在 startup、resume 和 target-sensitive turn 前调用 `InstructionMarkdown` 子组，发现、匹配、展开并装配 file-backed instruction sources。

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
  归 `durable-memory`

## 与 Runtime 的关系

这组页面为 runtime 提供：

- startup / resume 前可加载的 instruction fragments
- subtree-scoped 或 target-scoped 的适用性判断
- include expansion 后可进入 context assembly 的稳定文本输入

它不负责：

- durable memory recall
- transcript replay
- session restore
- skill / workflow activation

## 子页

- [instruction-markdown-loading.md](instruction-markdown-loading.md)
- [instruction-include-expansion.md](instruction-include-expansion.md)
- [conditional-instruction-rules.md](conditional-instruction-rules.md)
