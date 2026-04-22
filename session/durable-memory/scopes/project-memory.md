# Project Memory Overlay

## 职责

`ProjectMemoryOverlay` 定义 project-scoped durable memory binding。

它回答的是：

- durable memory 是否绑定到当前项目 durable store
- 是否随项目上下文被装载
- 是否作为默认 project-level durable root

它不是：

- `memory-types/project` 的同义词
- team-shared sync 机制本体
- agent identity scoped overlay

## 核心语义

`ProjectMemoryOverlay` 的默认语义是：

- project-rooted
- project-visible
- 常作为默认 durable runtime root

它可以承载多种 payload type，而不只是一条 `ProjectMemory` payload。

例如：

- `project`
- `feedback`
- `reference`
- 某些 private durable payload 也可落在 project-bound store 中，只要 sharing 语义未被提升为 team

## 不改变什么

`ProjectMemoryOverlay` 不改变：

- payload taxonomy
- recall pipeline 的 index / manifest / payload layering
- consolidation 的核心职责

## 与其它 overlays 的边界

- 与 [user-memory.md](user-memory.md)
  `user overlay` 强调 user-private ownership；`project overlay` 强调 project binding
- 与 [team-memory.md](team-memory.md)
  `project overlay` 关注“绑定到哪个项目”；`team overlay` 关注“是否跨成员共享与同步”
- 与 [agent-memory.md](agent-memory.md)
  `agent overlay` 关注 agent identity；`project overlay` 关注 project-rooted durable store

## 与 payload taxonomy 的边界

- 与 [../memory-types/project-memory.md](../memory-types/project-memory.md)
  `memory-types/project` 定义项目背景 payload；本页定义项目级 durable binding

一条 `memory-types/project` payload 常见地落在 `ProjectMemoryOverlay` 中，但两者不是同一概念。

## Default Local Mapping

默认本地映射中，`ProjectMemoryOverlay` 常见为：

- default durable runtime root
- project-scoped durable store

规范不强制具体目录结构，只要求 project-rooted binding 语义稳定。
