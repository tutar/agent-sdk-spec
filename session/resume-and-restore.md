# Resume And Restore

## 职责

`Resume And Restore` 定义 runtime 如何把某个 durable session 重新建立为可继续运行的 active binding。

runtime 触发这条路径，但 restore truth source 与 durable inputs 由 `Session` 持有。

它不负责：

- turn 内 query loop
- tool loop
- post-turn background worker
- durable-memory recall execution

## 最小对象

推荐最小对象：

```text
WakeRequest
  - session_id
  - target_branch?
  - restore_mode?
  - cursor?
  - pending_action_ref?
```

```text
ResumeSnapshot
  - session_id
  - wake_id
  - checkpoint
  - transcript_slice
  - runtime_state
  - working_state
  - short_term_memory?
  - lifecycle_state?
  - branch_refs?
```

## 稳定语义

### 1. restore-and-rebind, not replay

resume 的最小语义必须是：

- 定位要恢复的 session
- 加载 transcript / branch / sidecar durable inputs
- 重建可继续运行的 working state
- 重新把 runtime 绑定回该 session

因此 resume 不是 transcript replay，也不是仅恢复消息文本。

### 2. transcript 是核心输入，不是唯一恢复产物

resume 至少应读取：

- transcript slice
- checkpoint / cursor
- interruption markers
- branch refs

但恢复结果还应包括：

- working state
- pending action binding
- content replacement / projection state
- file-history / attribution / todo 等 sidecar state

### 3. interrupted state 必须可恢复

若 session 在 durable state 中停在：

- interrupted prompt
- interrupted turn
- requires_action

restore 后必须回到同一结构化阻塞语义，
而不能只恢复一条文本说明。

### 4. compact 后恢复仍需成立

compact 之后允许 runtime 不再重放全部历史消息，
但 restore 仍必须满足：

- compact boundary 可追溯
- checkpoint 可对齐
- short-term memory 只作 continuity 辅助
- transcript 仍是 durable truth source

### 5. fresh start 与 resumed start 必须区分

`Session` 至少应支持：

- fresh session start
- restore-and-rebind existing session
- clear current session and start fresh
- bind another existing session

这些路径的 binding 语义不同，不得被压成同一“新建或读日志”模型。

## 恢复什么，重建什么

直接恢复的 durable inputs 通常包括：

- transcript slice
- checkpoint / cursor
- pending action refs
- sidechain / branch refs
- sidecar restore inputs

restore 后重建的结果通常包括：

- current working state
- current continuation context
- resumed lifecycle projection
- current runtime binding

## 与相邻页面的边界

- 与 [transcript.md](transcript.md)
  transcript 是核心输入真源；本页定义如何基于它恢复运行状态
- 与 [working-state.md](working-state.md)
  本页定义恢复 working state 的语义；working state 本体见该页
- 与 [lifecycle-state.md](lifecycle-state.md)
  lifecycle state 可随 restore 恢复，但不是 restore truth source
- 与 [../harness/runtime/core/agent-runtime.md](../harness/runtime/core/agent-runtime.md)
  runtime 负责编排 binding / switching；`Session` 负责 durable restore semantics

## 规范结论

- resume 必须被定义为 restore-and-rebind，而不是 transcript replay
- `ResumeSnapshot` 必须足以恢复 compact 后 continuation 所需的 session state
- interrupted / requires-action semantics 必须以结构化方式恢复
