# Short-Term Memory

## 职责

`Short-Term Memory` 定义 `Session` 域中的 session-exclusive continuity memory。

runtime 在 compact 后 continuation、resume、away / handoff 等场景消费它；
它辅助 runtime 维持连续性，但 ownership 仍在 `Session`。

它负责：

- continuity summary
- compact-aware continuation
- away / handoff summary
- coverage boundary 与稳定性

它不负责：

- transcript truth
- restore truth source
- lifecycle projection
- cross-session durable recall
- durable memory consolidation

## 核心结论

short-term memory 是 `Session` 的 continuity object，而不是 transcript、prompt window 或长期知识库。

它至少要求：

- `1 Session = 1 Short-Term Memory`
- short-term memory 与 transcript 分层
- compact / resume 可以消费它，但不由它取代 transcript
- coverage boundary 与稳定性可显式建模

## 子页

- [short-term-memory-model.md](short-term-memory-model.md)
- [continuity-summary.md](continuity-summary.md)
- [coverage-boundary-and-stability.md](coverage-boundary-and-stability.md)
