# Session Event Log Schema

## 职责

本文件定义 `Session` transcript / event log 的标准对象模型。

它关注的是 durable event semantics，而不是某种具体文件格式。

runtime 负责追加与读取这些对象；
`Session` 负责它们的 durable ownership。

## SessionEvent

用于表达 session 中的一条 durable transcript / event entry。

```text
SessionEvent
  - event_id
  - session_id
  - parent_event_id?
  - logical_parent_event_id?
  - event_type
  - timestamp
  - branch_ref?
  - agent_id?
  - task_id?
  - payload
```

推荐 `event_type` 至少包括：

- `user_message`
- `assistant_message`
- `system_message`
- `attachment`
- `tool_result_reference`
- `compact_boundary`
- `branch_link`
- `resume_marker`
- `task_notification`
- `metadata_update`

约束：

- durable transcript 只应记录 durable semantics，而不是 UI 临时态
- 高频 progress / spinner / ephemeral status 默认不应直接进入 event log
- 若 tool result 被外存化，event log 中应持有稳定 reference，而不是要求重新生成原结果

## EventSelector

用于表达读取 transcript / event log 的选择器。

```text
EventSelector
  - cursor?
  - range?
  - window?
  - event_types?
  - branch?
```

## SessionCheckpoint

用于表达一次 durable append 的提交位置。

```text
SessionCheckpoint
  - session_id
  - last_event_id
  - cursor
  - committed_at
  - branch_id?
  - event_count?
```

## WakeRequest

用于表达从 durable session 恢复 runtime 的请求。

```text
WakeRequest
  - session_id
  - target_branch?
  - restore_mode?
  - cursor?
  - pending_action_ref?
```

## ResumeSnapshot

用于表达 restore-and-rebind 的恢复产物。

```text
ResumeSnapshot
  - session_id
  - wake_id
  - checkpoint
  - transcript_slice
  - runtime_state
  - working_state
  - lifecycle_state?
  - pending_action?
  - short_term_memory?
  - branch_refs?
```

## RuntimeState

`runtime_state` 推荐至少包括：

```text
RuntimeState
  - active_branch?
  - last_checkpoint
  - resume_cursor?
  - pending_continuation?
  - compact_boundary_ref?
```

## WorkingState

`working_state` 推荐至少包括：

```text
WorkingState
  - active_messages?
  - pending_action_binding?
  - content_replacements?
  - tool_execution_state?
  - working_view_projection?
  - continuation_inputs?
```

## ShortTermMemoryCoverageBoundary

用于表达 short-term memory 覆盖到 transcript 的哪个位置。

```text
ShortTermMemoryCoverageBoundary
  - session_id
  - covered_until_event_id?
  - covered_until_cursor?
  - generated_at
```

约束：

- `ShortTermMemoryCoverageBoundary` 必须能与 `SessionCheckpoint` 对齐
- `wake()` 时若消费 short-term memory，应能判断其是否覆盖待恢复 checkpoint

## 稳定语义

- transcript / event log 是 session durable truth source，而不是 continuity summary
- compact boundary、branch link、resume marker 都应以 durable event semantics 表达
- `ResumeSnapshot` 必须足以支撑 compact 后恢复，而不依赖旧进程内状态
- branch / sidechain / persisted tool result reference 应通过 durable event 语义进入 session 边界

## 与相邻页面的边界

- [transcript.md](transcript.md)
  定义 transcript 的 durable truth-source 语义
- [resume-and-restore.md](resume-and-restore.md)
  定义 restore-and-rebind 的行为语义
- [working-state.md](working-state.md)
  定义 working state 作为 continuation context 的语义

## 规范结论

- `SessionEvent`、`EventSelector`、`WakeRequest`、`ResumeSnapshot` 应作为 `Session` 的标准对象
- event log 接口应独立于 JSONL、数据库行或消息数组实现
- transcript / event log 不得退化成 memory store、UI state store 或 lifecycle projection store
