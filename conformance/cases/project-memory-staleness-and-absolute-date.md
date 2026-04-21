# Case: Project Memory Staleness And Absolute Date

## 目标

验证 `ProjectMemory` 保存的是非代码态项目背景，并且对时间锚点与 staleness 有显式建模。

## Preconditions

- session / memory subsystem 支持 `ProjectMemory` 或语义等价类型
- durable memory subsystem 支持 date / freshness / staleness 的显式或语义等价建模

## Ingress

1. 提供一条包含相对日期的项目背景信息
2. 提供一条包含项目动机、constraint 或 stakeholder ask 的背景信息
3. 再提供一条仅能从 code / git 推导的候选信息
4. 触发 durable memory 写入、后续 recall 与 consolidation

## Expected Runtime Semantics

- `ProjectMemory` 应记录非代码态项目背景，而不是 code / git 派生事实
- 相对日期在写入时应转成绝对时间或语义等价时间锚点
- 记录内容应包含：
  - fact_or_decision
  - why
  - how_to_apply
  或语义等价结构
- recalled `ProjectMemory` 应作为 future suggestion 的背景上下文，而不是行为 rule
- stale / freshness 必须允许被建模；实现不应把旧 `ProjectMemory` 当永远正确的当前真相

## Expected Persistent Effects

- project background 会以正确类型进入 durable store
- 可从 code / git 推导的事实不会被误存为 `ProjectMemory`
- consolidation 应允许合并同一 initiative / incident 的重复背景，并修正 stale 项目事实

## Allowed Variance

- 绝对时间可以用不同字段或规范化文本表示
- staleness 可以通过 freshness note、mtime、revalidation hint 或语义等价机制表达

## Failure Conditions

- 相对日期原样进入 durable memory，无法在未来解释
- repo 结构 / git 历史被存成 `ProjectMemory`
- recalled `ProjectMemory` 被当作 action policy 使用
- consolidation 无法修 stale 或清理已失效项目背景
