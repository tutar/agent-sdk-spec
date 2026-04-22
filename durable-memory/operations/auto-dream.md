# Auto Dream

## 职责

`AutoDream` 是 durable memory 的后台 / 跨 session consolidation mode。

它的目标不是从单个 turn 提取记忆，而是对已有 durable store、近期 session 线索和 resident index 做一次反思式整理。

它通常负责：

- merge duplicate memory
- 修正已过时或被证伪的 memory
- 将近期 session 中的稳定信号整合进 topic memory
- 修剪和重建 resident index

## 核心结论

dream 不是独立 memory plane，而是 durable memory 的标准 consolidation mode。

它处理的是：

- recent sessions
- append-first capture stream
- existing durable payloads
- resident entrypoint / index

它产出的是：

- updated topic memories
- pruned or merged durable refs
- tightened or rebuilt entrypoint index

## 触发模式

`AutoDream` 至少应支持两种触发模式：

- `manual`
  用户显式触发
- `automatic`
  系统按策略自动触发

二者的差异主要在：

- 触发来源
- 调度方式
- 权限与执行面

而不应体现在：

- 基本 consolidation 语义
- 对 index / topic payload 的处理目标
- 最终结果对象的定义

## Dream 的内部动作

dream 至少应覆盖：

- `merge`
  将新信号优先并入已有 topic
- `dedupe`
  消除 direct write / extract / append-first capture 带来的重复
- `stale repair`
  修正被证伪、过时或不再 load-bearing 的 memory
- `distill`
  将 append-first capture stream 蒸馏成 topic payload 和 resident index
- `index tightening`
  去掉 stale pointer，重建低成本入口

## 稳定接口

推荐最小接口：

```text
DreamConsolidationRequest
  - trigger: manual | automatic
  - memory_scope
  - source_sessions[]
  - existing_memory_refs[]
  - index_ref?
  - consolidation_policy?

DreamConsolidationResult
  - updated_memory_refs[]
  - pruned_memory_refs[]
  - updated_index_ref?
  - summary
  - status: completed | no_change | failed | killed
  - trigger: manual | automatic
```

## 规范要求

### 1. automatic dream 必须显式建模调度门槛

自动 dream 不应在每轮 turn 无条件触发。

至少应允许：

- 最小时间间隔
- 最小 session 数量
- memory activity threshold
- in-progress lock

### 2. automatic dream 必须显式建模锁语义

若系统允许后台 consolidation，则必须处理：

- 并发 dream
- crash recovery
- aborted run rollback
- 下一次重试时机

automatic dream 的失败不得影响：

- session transcript restore
- durable memory 的已提交视图
- 后续普通 turn 的继续运行

### 3. assistant-style append-first mode 只改变 operating mode

append-first 模式下，系统通常会表现为：

- 新记忆先进入 append-first capture stream
- durable index / topic memory 的整理延后到 dream
- dream 更常由 scheduled/background path 触发

但这些变化只影响 operating mode，不改变：

- durable memory 的 shared-package 所有权边界
- `AutoDream` 的 request/result/lock 语义
- `Task`、`Gateway`、`Harness` 的职责边界

## 与其它模块的边界

- 与 [extract-memories.md](extract-memories.md)
  extract 是 turn-end write path；dream 是 cross-session consolidation mode
- 与 [../auto-memory-runtime.md](../auto-memory-runtime.md)
  dream 可以作为默认 runtime 的 consolidation path，但不等于 runtime 本体
- 与 [../durable-memory-overview.md](../durable-memory-overview.md)
  dream 主要作用于 durable memory，而不是 short-term session memory
- 与 [../durable-memory-scopes-and-overlays.md](../durable-memory-scopes-and-overlays.md)
  dream 可以操作不同 scope 的 durable memory，但 scope 语义不由 dream 定义
- 与 [../../harness/agent-profiles/assistant-agent.md](../../harness/agent-profiles/assistant-agent.md)
  assistant agent profile 可以把 dream 作为常见 maintenance operating mode，但 profile 不拥有 dream 的 consolidation 语义

## 规范结论

- `dream` 是 durable memory 的标准 consolidation mode
- 规范应同时覆盖 `manual dream` 与 `autoDream`
- `autoDream` 是调度模式，`dream` 是 consolidation 语义
- dream 的默认执行形态可以是受限 subagent + task surface
- append-first operating mode 可以改变 dream 的触发频率和输入来源，但不改变 dream 的 ownership 与结果语义
