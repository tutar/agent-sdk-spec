# Continuity Summary

## 职责

`Continuity Summary` 定义 short-term memory 如何为 compact、resume 和 handoff 提供低成本连续性表示。

## 核心结论

continuity summary 应被视为 short-term memory 的主要内容形态，而不是 transcript 的替代品。

它至少应支持：

- compact-aware continuation
- resume 后快速继续
- away / handoff summary

## 稳定语义

### 1. summary serves continuity

continuity summary 用于帮助 runtime 在不重放全量 transcript 的情况下恢复工作连续性。

典型包含：

- 当前目标
- 阶段性进展
- 关键未完成事项
- 最近关键决策与风险

### 2. summary is not transcript

continuity summary 不应伪装成 transcript 原消息。

因此：

- 不通过改写 transcript 模拟 continuity summary
- 不把它建模成长期 durable payload
- 不将其作为 agent-global memory 共享

### 3. compact-aware

compact 后的 continuation 可以消费 continuity summary，但仍应保留对 compact boundary 前 durable history 的可追溯性。

## 与其它子规范的边界

- 与 [short-term-memory-model.md](short-term-memory-model.md)
  本页定义 summary 形态；上页定义 short-term memory 总模型
- 与 [coverage-boundary-and-stability.md](coverage-boundary-and-stability.md)
  summary 的 coverage 与稳定性约束见该页
