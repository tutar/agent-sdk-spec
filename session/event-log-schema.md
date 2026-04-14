# Session Event Log Schema

## 目标

本文件定义 `Session` 的事件日志对象。

它关注 durable event log 的接口语义，而不是某种具体文件格式。

## SessionEvent

用于表达 session 中的一条持久化事件。

```text
SessionEvent
  - event_id
  - session_id
  - parent_event_id?
  - event_type
  - timestamp
  - agent_id?
  - task_id?
  - payload
```

推荐 `event_type` 至少包括：

- `user_message`
- `assistant_message`
- `system_message`
- `attachment`
- `compact_boundary`
- `task_notification`
- `memory_link`
- `metadata_update`

## EventSelector

用于表达读取 event log 的选择器。

```text
EventSelector
  - cursor?
  - range?
  - window?
  - event_types?
  - branch?
```

## WakeRequest

用于表达从 durable session 恢复 runtime 的请求。

```text
WakeRequest
  - session_id
  - target_branch?
  - restore_mode?
  - cursor?
```

## ResumeSnapshot

用于表达从 event log 恢复后的运行时快照。

```text
ResumeSnapshot
  - session_id
  - runtime_state
  - transcript_slice
  - working_state
  - short_term_memory?
  - child_refs?
```

## 规范结论

- `SessionEvent`、`EventSelector`、`WakeRequest`、`ResumeSnapshot` 应作为 session event log 的标准对象
- event log 接口应独立于 JSONL、数据库行或消息数组实现
