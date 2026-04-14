# Canonical Object Model

## 目标

本文件定义跨语言共享的核心标准对象。

这些对象不是某个语言 SDK 的具体类型，而是所有实现都应保持语义一致的 canonical model。

## 设计原则

- 对象语义稳定优先于字段命名一致
- 允许语言实现调整大小写、类型系统映射和序列化库
- 不允许改变对象职责和状态含义
- 对象应最小但足以承载跨模块协作

## RuntimeEvent

用于描述 runtime 对外发射的标准事件。

推荐最小字段：

```text
RuntimeEvent
  - event_type
  - event_id
  - timestamp
  - session_id
  - agent_id?
  - task_id?
  - turn_id?
  - causation_id?
  - correlation_id?
  - payload
```

推荐 `event_type` 至少包括：

- `request_started`
- `assistant_delta`
- `assistant_message`
- `tool_started`
- `tool_progress`
- `tool_result`
- `tool_failed`
- `tool_cancelled`
- `requires_action`
- `task_notification`
- `turn_completed`
- `turn_failed`
- `turn_cancelled`
- `turn_aborted`
- `turn_timed_out`

约束：

- `RuntimeEvent` 是外部稳定协议，不等于内部消息对象
- `event_type` 应与 [harness/runtime-core/failure-and-terminal-states.md](harness/runtime-core/failure-and-terminal-states.md) 中的终止态和错误语义兼容
- `causation_id / correlation_id` 用于把 event 与 tool call、task、request 或外部动作关联起来

## TerminalState

用于表达一次 turn 或 task 的终止结果。

推荐最小字段：

```text
TerminalState
  - status
  - reason?
  - retryable
  - error?
  - completed_at
  - summary?
```

推荐 `status` 至少包括：

- `completed`
- `failed`
- `cancelled`
- `requires_action`
- `aborted`
- `timed_out`

约束：

- `status` 必须与 [harness/runtime-core/failure-and-terminal-states.md](harness/runtime-core/failure-and-terminal-states.md) 中的统一终止态保持一致
- `retryable` 必须显式
- `error` 如存在，应使用统一 `SdkError`

## SdkError

用于表达跨 runtime / tool / task / session 的统一错误对象。

```text
SdkError
  - class
  - message
  - retryable
  - terminal
  - source: model | tool | task | session | sandbox | transport | policy
  - code?
  - details?
```

推荐 `class` 至少包括：

- `auth_error`
- `permission_denied`
- `rate_limited`
- `context_too_large`
- `output_limit`
- `validation_error`
- `tool_protocol_error`
- `dependency_unavailable`
- `network_transient`
- `execution_target_failed`
- `conflict`
- `not_found`
- `internal_error`
- `non_retryable`

## RequiresAction

用于表达 runtime 被结构化阻塞的状态。

推荐最小字段：

```text
RequiresAction
  - action_type
  - session_id
  - agent_id?
  - task_id?
  - terminal_state: requires_action
  - tool_name?
  - description
  - input?
  - request_id
  - action_ref?
  - policy_decision_id?
  - resumable: boolean
  - details?
```

推荐 `action_type` 至少包括：

- `approval`
- `user_input`
- `selection`
- `plan_review`
- `external_resume`

约束：

- `RequiresAction` 是结构化阻塞对象，不是错误对象
- 其语义必须与 [session/lifecycle-state.md](session/lifecycle-state.md) 以及 [tools/policy-engine.md](tools/policy-engine.md) 一致

## ToolResult

用于统一 tool 或 tool-like action 的结果返回语义。

推荐最小字段：

```text
ToolResult
  - tool_name
  - success
  - tool_use_id?
  - content
  - structured_content?
  - error?
  - metadata?
  - persisted_ref?
  - truncated?
```

约束：

- 若 `success=false`，应优先通过 `error: SdkError` 表达失败原因
- tool 的 `progress/result/failed/cancelled` 过程语义应通过 `RuntimeEvent` 或 `ToolExecutionEvent` 表达，而不全部压缩进 `ToolResult`

## CapabilityView

用于表达某一时刻 runtime 可暴露给模型或上层系统的能力视图。

推荐最小字段：

```text
CapabilityView
  - tools
  - skills
  - commands?
  - mcp_servers?
  - hands?
  - policies?
```

## TaskRecord

用于表达 task-first orchestration 中的统一任务对象。

推荐最小字段：

```text
TaskRecord
  - task_id
  - type
  - status
  - description
  - session_id?
  - agent_id?
  - parent_task_id?
  - output_ref?
  - start_time
  - end_time?
  - terminal_state?
  - metadata?
```

推荐 `status` 至少与 task event 语义兼容：

- `pending`
- `running`
- `completed`
- `failed`
- `cancelled`
- `killed`

## TaskEvent

用于表达 task 生命周期中的增量事件。

```text
TaskEvent
  - task_id
  - event_id
  - timestamp
  - type: started | progress | notification | completed | failed | cancelled | killed
  - payload
  - terminal_state?
  - error?
```

## SessionCheckpoint

用于表达 session durable append 的提交位置。

```text
SessionCheckpoint
  - session_id
  - last_event_id
  - cursor
  - committed_at
```

## 规范结论

- `RuntimeEvent`、`TerminalState`、`SdkError`、`RequiresAction`、`ToolResult`、`CapabilityView`、`TaskRecord`、`TaskEvent`、`SessionCheckpoint` 应作为跨语言共享对象保留
- 这些对象的语义必须独立于具体文件组织和类型系统
- `object-model.md` 应成为各子模块引用的 canonical field table，而不是各自发明局部变体
