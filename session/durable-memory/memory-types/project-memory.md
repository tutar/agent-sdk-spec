# Project Memory

## 职责

`ProjectMemory` 定义 durable memory 中的项目背景层。

它负责保存：

- ongoing work
- goals
- initiatives
- bugs / incidents
- constraints / deadlines / stakeholder asks

前提是这些信息不能从 code 或 git history 直接推导。

## 核心语义

- `ProjectMemory` 保存非代码态项目背景
- 它应用于 future suggestions 的项目上下文增强
- relative dates 在写入时应允许转为 absolute dates 或语义等价形式
- 它通常比其它 durable types 更快 decay，因此推荐支持：
  - fact_or_decision
  - why
  - how_to_apply
- 在 shared/private split 存在时，应 strongly bias toward shared/team

## 与其它类型的边界

- 与 [reference-memory.md](reference-memory.md)
  `ProjectMemory` 保存背景事实；`ReferenceMemory` 保存去哪里找外部信息
