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
  - payload
```

推荐 `event_type` 至少包括：

- `assistant_delta`
- `assistant_message`
- `tool_started`
- `tool_progress`
- `tool_result`
- `requires_action`
- `task_notification`
- `turn_completed`
- `turn_failed`

## TerminalState

用于表达一次 turn 或 task 的终止结果。

推荐最小字段：

```text
TerminalState
  - status
  - reason
  - retryable?
  - summary?
```

推荐 `status` 至少包括：

- `completed`
- `stopped`
- `blocked`
- `failed`
- `budget_exhausted`

## RequiresAction

用于表达 runtime 被结构化阻塞的状态。

推荐最小字段：

```text
RequiresAction
  - action_type
  - session_id
  - agent_id?
  - task_id?
  - tool_name?
  - description
  - input?
  - request_id?
```

## ToolResult

用于统一 tool 或 tool-like action 的结果返回语义。

推荐最小字段：

```text
ToolResult
  - tool_name
  - success
  - content
  - structured_content?
  - metadata?
  - persisted_ref?
  - truncated?
```

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
  - output_ref?
  - start_time
  - end_time?
  - metadata?
```

## 规范结论

- `RuntimeEvent`、`TerminalState`、`RequiresAction`、`ToolResult`、`CapabilityView`、`TaskRecord` 应作为跨语言共享对象保留
- 这些对象的语义必须独立于具体文件组织和类型系统
