# Continuity Summary

## 职责

`Continuity Summary` 定义 short-term memory 如何为 compact、resume 和 handoff 提供低成本连续性表示。

它辅助 runtime 在不重放全部 transcript 的情况下继续工作；
但它不成为 transcript 的替代物。

## 核心结论

continuity summary 应被视为 short-term memory 的主要内容形态，而不是 transcript dump。

它至少应支持：

- compact-aware continuation
- resume 后快速继续
- away / handoff summary
- current-state oriented continuity

## 稳定语义

### 1. summary serves continuity

continuity summary 的目标是帮助 runtime 维持工作连续性。

典型包含：

- 当前目标
- 阶段性进展
- 关键未完成事项
- 最近关键决策与风险
- current state / next-step oriented continuity

### 2. summary is not transcript

continuity summary 不应伪装成 transcript 原消息。

因此：

- 不通过改写 transcript 模拟 continuity summary
- 不把它建模成 cross-session durable payload
- 不将其作为 agent-global memory 共享

### 3. compact-aware but traceable

compact 后的 continuation 可以消费 continuity summary，
但仍应保留对 compact boundary 前 durable history 的可追溯性。

summary 只负责 continuity，不负责 truth-source replacement。

## 与其它子规范的边界

- 与 [short-term-memory-model.md](short-term-memory-model.md)
  本页定义 summary 形态；上页定义 short-term memory 总模型
- 与 [coverage-boundary-and-stability.md](coverage-boundary-and-stability.md)
  summary 的 coverage 与稳定性约束见该页
- 与 [../transcript.md](../transcript.md)
  transcript 是 durable truth source；summary 只是 continuity layer
