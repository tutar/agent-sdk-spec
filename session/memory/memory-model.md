# Session Memory Model

## 目标

本规范定义 `Session` 域中的长短期记忆模型。

这里的 `memory` 不是当前 prompt，也不是 transcript 的别名。规范上应至少区分三类核心对象：

- `session transcript`
  可恢复的事件历史
- `short-term session memory`
  当前 session 的连续性摘要
- `durable memory`
  跨 session 的可召回知识

本规范的目标是稳定以下语义：

- 哪些信息应进入短期记忆
- 哪些信息应进入长期记忆
- 短期记忆、长期记忆与 transcript 的边界
- 记忆的写入、召回、整合与恢复时机
- 记忆如何与 compact、resume、subagent 协同

## 一、核心原则

- transcript 是恢复依据，不是记忆层本身
- 短期记忆服务于 continuity，不服务于跨 session recall
- 长期记忆服务于 recall，不服务于 session restore
- 显式记忆文件属于独立的注入面，不属于 session transcript
- scope 属于 durable memory 的作用域维度，不属于新的 memory plane
- `1 Session = 1 Short-Term Memory`
- `1 Agent = 1 Global Long-Term Memory`

## 二、分层模型

### 1. Session Transcript

`SessionTranscript` 是会话的可恢复历史。

它负责：

- 保存原始 turn、tool use、tool result、system event
- 支撑 resume、replay、audit、restore
- 为短期记忆和长期记忆提供提炼源

它不负责：

- 作为 continuity summary
- 直接作为长期知识库
- 直接决定 recall 结果

### 2. Short-Term Session Memory

`ShortTermSessionMemory` 是当前 session 的 continuity object。

它负责：

- 维护当前会话的压缩连续性表示
- 为 compact 提供 continuity summary
- 为 resume 后的快速继续提供低成本状态
- 为长任务 handoff 提供会话级摘要

它不负责：

- 替代 transcript
- 承担跨 session recall
- 作为 durable knowledge 的存储

绑定约束：

- 一个 short-term memory 只能服务一个 session
- 一个 session 只能绑定一个 short-term memory
- harness 重启不应导致该 short-term memory 丢失

应进入短期记忆的信息包括：

- 当前任务目标与阶段性进展
- 尚未完成的中间结论
- 最近关键决策、约束和风险
- compact 后仍需保留的局部上下文

不应进入短期记忆的信息包括：

- 大量原始 stdout / file dump
- 可从 transcript 精确重放的细粒度历史
- 通用用户偏好或项目长期知识

### 3. Durable Memory

`DurableMemory` 是跨 session 的可召回知识层。

它负责：

- 保存用户偏好、项目知识、约束、经验、gotchas
- 保存对未来回合可能有价值的抽象信息
- 为后续 session 提供 bounded recall 候选

它不负责：

- 替代 session restore
- 保存每轮完整对话
- 充当当前 turn 的 working state

产品级绑定约束：

- agent 只有一个全局长期记忆空间
- 多个 session 可共享并沉淀到同一个 agent long-term memory
- 当前产品语义下，long-term memory 不是 per-user、per-chat 拆分

应进入长期记忆的信息包括：

- 稳定的用户偏好
- 项目级规则与约束
- 重复出现的问题及解决方式
- API / 环境 / workflow 的注意事项
- 值得跨会话复用的设计决策

不应进入长期记忆的信息包括：

- 仅对当前 turn 有价值的临时状态
- 纯粹可由 transcript 重新推导的细节
- 已经过时且没有保留价值的操作痕迹

durable memory 的 recall 语义见 [durable-memory-recall.md](durable-memory-recall.md)。

durable memory 的 scope 维度见 [scoped-durable-memory.md](scoped-durable-memory.md)。

显式 durable injection 的详细语义见 [memory-injection.md](memory-injection.md)。

## 三、生命周期

### 1. 短期记忆生命周期

推荐流程：

1. session 进行中积累 transcript
2. 到达阈值或自然停顿点
3. 从 transcript 增量提炼 continuity summary
4. 更新 short-term memory
5. compact / resume / away summary 消费该摘要

短期记忆更新应满足：

- 基于增量，而不是每次全量重算
- 在安全边界更新，避免截断未闭合的 tool chain
- 允许异步更新，但 compact 前应支持等待

### 2. 长期记忆生命周期

推荐流程：

1. 完整 turn 或 query loop 结束
2. 从 transcript 中提炼 durable memory 候选
3. 写入 durable memory store
4. 周期性执行 consolidate / cleanup
5. 后续 turn 通过 recall 取回相关 memory

长期记忆更新应满足：

- 以完整语义单元为边界，而不是任意中间状态
- 支持后台提炼
- 支持去重、合并、整合
- 支持 freshness / source metadata

### 3. 召回生命周期

推荐流程：

1. turn 开始前进行 memory recall prefetch
2. 基于 query、runtime context、tool context 选择候选
3. 对 recall 结果做数量与体积约束
4. 将 recall 结果作为 attachments / context sources 注入当前 turn

召回必须满足：

- bounded
- 可去重
- 可过滤已注入对象
- 不要求把整个 memory store 暴露给模型

## 四、与其他模块的边界

### 1. 与 Session 的边界

`Session` 管理：

- transcript
- runtime state
- restore
- short-term memory linkage

`Session` 不直接定义 durable memory store 的内部实现。

### 2. 与 Harness 的边界

`Harness` 负责：

- 决定何时触发记忆读写
- 将短期记忆和长期记忆接入当前 turn
- 在 compact / resume 时消费短期记忆

`Harness` 不应把 transcript 和 memory 混成一个对象。

### 3. 与 ContextProvider 的边界

`ContextProvider` 负责：

- 将 durable context source 变成当前 turn 的上下文输入

它不负责：

- durable memory 的提炼
- session transcript 的恢复

### 4. 与 ContextGovernance 的边界

`ContextGovernance` 负责：

- token budget
- compact 触发
- pruning 策略

它可以消费短期记忆，但不拥有短期记忆。

## 五、推荐最小接口

```text
ShortTermMemoryStore
  - load(session_id) -> short_term_memory | null
  - update(session_id, transcript_delta, current_memory) -> updated_memory
  - get_coverage_boundary(session_id) -> transcript_cursor | null
  - wait_until_stable(session_id, timeout_ms) -> ready | timed_out
```

```text
DurableMemoryStore
  - put(memory_record) -> memory_id
  - update(memory_id, patch) -> memory_record
  - delete(memory_id) -> deleted
  - list(selector) -> memory_index
  - read(memory_refs) -> memory_payloads
```

```text
DurableMemoryExtractor
  - extract(transcript_slice, existing_memory_context) -> memory_records
```

推荐 `memory_record` 至少包括：

```text
DurableMemoryRecord
  - memory_id?
  - scope
  - summary
  - source_session_id?
  - source_event_refs?
  - freshness?
  - metadata?
```

## 六、失败与延迟语义

`memory consolidation` 作为标准扩展，可以异步、延迟或后台执行，但必须满足：

- consolidation 失败不得破坏 transcript / checkpoint / resume
- recall 若读取到旧版本 durable memory，行为上允许暂时滞后，但不能造成 session restore 语义漂移
- 去重、合并、重写 durable memory 时，必须保留可追溯 source metadata 或语义等价引用
- consolidation 结果应体现在 durable memory store，而不是通过改写历史 transcript 模拟

## 七、规范结论

- `DurableMemoryRecord` 应作为 durable memory 的最小共享对象
- memory consolidation 是标准扩展，不是 session restore 的前提
- consolidation 可以失败或延迟，但不能让 session 丢失可恢复性
