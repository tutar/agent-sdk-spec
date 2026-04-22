# Coverage Boundary And Stability

## 职责

`Coverage Boundary And Stability` 定义 short-term memory 覆盖 transcript 的边界，以及 compact / wake / resume 前的稳定性语义。

## 核心结论

short-term memory 不只是“有无摘要”，还必须能表达：

- 它覆盖到 transcript 的哪里
- 当前摘要是否已稳定

## 稳定语义

### 1. coverage boundary

实现应能表达 short-term memory 覆盖到 transcript 的哪个位置。

这使 runtime 能判断：

- 当前 checkpoint 是否被 short-term memory 覆盖
- compact / wake / resume 时是否可安全消费最近稳定摘要

### 2. stability

若 short-term memory 尚未稳定，`wake()` 或等价恢复路径应显式返回：

- wait
- timeout
- use last stable summary

而不是静默漂移到未稳定的 continuity 表示。

### 3. safe update boundary

short-term memory 更新应在安全边界进行，避免截断未闭合的 tool-use chain 或 continuation chain。

### 4. latest stable summary

若实现允许读取“最近稳定摘要”，则必须明确：

- 返回的是 latest stable summary
- 不是任意最新但未稳定的 transient summary

## 与其它子规范的边界

- 与 [short-term-memory-model.md](short-term-memory-model.md)
  本页补充 coverage 与稳定性约束
- 与 [continuity-summary.md](continuity-summary.md)
  summary 内容形态见该页
- 与 [../event-log-schema.md](../event-log-schema.md)
  transcript cursor / coverage boundary 的标准对象见该页

## 规范结论

- coverage boundary 与 stability 必须成为 short-term memory 的显式语义
- compact / wake / resume 消费 short-term memory 时，不得静默跨越未稳定边界
