# User Memory Overlay

## 职责

`UserMemoryOverlay` 定义 user-scoped durable memory binding。

它回答的是：

- durable memory 是否绑定到用户私有 durable store
- 是否跨项目保留
- 是否只对当前用户可见

它不是：

- `memory-types/user` 的同义词
- team/shared memory
- host-local machine binding

## 核心语义

`UserMemoryOverlay` 的默认语义是：

- private
- user-owned
- non-team-shared
- 可跨项目或 user-root 复用

它允许承载多种 payload type，而不只是一条 `UserMemory` payload。

例如：

- private `feedback`
- private `reference`
- private `project`
- private `user`

## 不改变什么

`UserMemoryOverlay` 不改变：

- payload taxonomy
- topic payload / entrypoint / manifest 的基本结构
- recall 必须 bounded 的要求
- consolidation 的 merge / dedupe / stale repair 职责

## 与其它 overlays 的边界

- 与 [project-memory.md](project-memory.md)
  `project overlay` 关注项目绑定；`user overlay` 关注用户私有归属
- 与 [local-memory.md](local-memory.md)
  `local overlay` 关注 host/workspace locality；`user overlay` 关注 user ownership
- 与 [team-memory.md](team-memory.md)
  `team overlay` 是 shared；`user overlay` 是 private

## 与 payload taxonomy 的边界

- 与 [../memory-types/user-memory.md](../memory-types/user-memory.md)
  `memory-types/user` 定义用户画像 payload；本页定义用户私有 storage binding

一条 `memory-types/user` payload 可以落在 `UserMemoryOverlay` 中，但两者不是同一概念。

## Default Local Mapping

默认本地映射中，`UserMemoryOverlay` 常见为：

- user-level durable backing root
- user-private agent memory backing

规范不强制具体路径，只要求 user-private、non-team-shared 的语义稳定。
