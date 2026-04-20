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
- compact / wake 时是否可安全消费最近稳定摘要

### 2. stability

若 short-term memory 尚未稳定，`wake()` 或等价恢复路径应显式返回：

- wait
- timeout
- use last stable summary

而不是静默漂移到未稳定的 continuity 表示。

### 3. safe update boundary

short-term memory 更新应在安全边界进行，避免截断未闭合的 tool/use chain。

## 与其它子规范的边界

- 与 [short-term-memory-model.md](short-term-memory-model.md)
  本页补充 coverage 与稳定性约束
- 与 [../event-log-schema.md](../event-log-schema.md)
  transcript cursor / coverage boundary 的事件模型见该页
