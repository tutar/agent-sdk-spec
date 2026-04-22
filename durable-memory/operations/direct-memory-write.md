# Direct Memory Write

## 职责

`DirectMemoryWrite` 定义 foreground path 如何显式写入 durable memory。

它覆盖：

- explicit remember / forget
- foreground topic update
- append-first variant 下的 foreground capture

## 核心结论

direct write 是 durable memory 写侧的第一条正式路径。

它不等于：

- turn-end extraction
- dream consolidation
- transcript rewrite

## 稳定语义

### 1. foreground direct write

主 runtime 应允许模型直接创建或更新 durable memory。

典型场景：

- 用户明确要求 remember
- 当前 turn 已形成稳定 durable knowledge

默认 durable target 通常是 topic memory records，而不是 resident entrypoint 本体。

### 2. explicit remember / forget

`remember this` / `forget this` 不是新的 pipeline，而是 direct write 的显式触发场景。

### 3. assistant-style append-first variant

某些 long-lived host profile 可将 default runtime 切换为 append-first variant：

- 新记忆先进入 append-only capture stream
- resident index 与 topic records 延后由 dream 更新
- delayed consolidation 不应破坏 recall / continuity / viewer reattach

这是一种 operating mode 变化，不是新的 durable memory plane。

## 与其它子规范的边界

- 与 [extract-memories.md](extract-memories.md)
  extract 是补位路径；本页只定义 foreground write
- 与 [auto-dream.md](auto-dream.md)
  dream 是 cross-session consolidation，不是 foreground direct write
- 与 [../../harness/agent-profiles/assistant-agent.md](../../harness/agent-profiles/assistant-agent.md)
  assistant agent profile 可以偏好 append-first / consolidate-later，但不拥有 durable memory 本体
- 与 [../durable-memory-scopes-and-overlays.md](../durable-memory-scopes-and-overlays.md)
  team memory / agent memory 是 scoped extension，不改变这些写入路径的上层语义
- 与 [../storage/topic-memory.md](../storage/topic-memory.md)
  foreground path 的默认 durable payload target 见该页
- 与 [../storage/memory-md.md](../storage/memory-md.md)
  resident index 可以被这些路径更新，但不应被误当作 durable正文层

## 默认本地映射

当前默认本地映射通常表现为：

- prompt-taught direct write
- assistant-style append-only daily log + later dream
- topic payload 作为 durable target
- `MEMORY.md` 或等价物作为 resident entrypoint

## 规范结论

- direct write 是 durable memory 写侧的第一条正式路径
- `remember / forget` 属于 direct write 的显式触发场景
- direct write 不应与 extract 或 dream 混写
