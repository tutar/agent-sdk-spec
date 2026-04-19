# Failure And Terminal States

## 目标

本文统一 `Harness.Runtime.Core` 中的 turn terminal semantics。

它只讨论：

- 一次 turn 如何结束
- 为什么结束
- 失败和阻塞如何分类

它不讨论：

- task status
- gateway delivery result
- channel adapter 的协议错误呈现

## 核心原则

runtime 至少要区分三层对象：

- `event`
  运行中的过程事实
- `terminal state`
  本次 turn 的结束归类
- `error class`
  导致失败、阻塞或恢复的原因类型

这三层不能混成一个自由文本结果。

## TerminalState

推荐最小终态：

```text
TerminalState
  - completed
  - failed
  - cancelled
  - requires_action
  - aborted
  - timed_out
```

语义：

- `completed`
  turn 达到稳定完成点
- `failed`
  turn 未达到语义目标并以错误结束
- `cancelled`
  存在明确的终止发起方
- `requires_action`
  turn 到达稳定阻塞点，等待外部恢复输入
- `aborted`
  当前执行上下文被中断、替换或撤销，不表达“用户有意取消”
- `timed_out`
  turn 因时间上限到达稳定终态

## RequiresAction 不是失败

`requires_action` 必须与 `failed` 分开。

它表示：

- runtime 已到达可持久化的稳定阻塞点
- 需要 gateway / host / operator 提供后续输入
- 当前 turn 可以在以后恢复，而不是必须重跑整轮语义

## Cancelled 与 Aborted 的边界

必须区分：

- `cancelled`
  有明确取消意图，例如用户或控制面发出的中止动作
- `aborted`
  执行被替换、丢弃或被更高层流程打断

例如：

- host 显式 interrupt 当前 turn，更接近 `cancelled`
- 某次中间恢复路径被新的执行分支取代，更接近 `aborted`

## Error Class

推荐最小错误分类：

```text
AgentErrorClass
  - auth_error
  - permission_denied
  - rate_limited
  - context_too_large
  - output_limit
  - validation_error
  - tool_protocol_error
  - dependency_unavailable
  - network_transient
  - execution_target_failed
  - conflict
  - not_found
  - internal_error
  - non_retryable
```

推荐错误对象：

```text
AgentError
  - class
  - message
  - retryable
  - terminal
  - source: model | tool | task | session | sandbox | transport | policy
  - code?
  - details?
```

## Retryability

`retryable` 必须是显式字段，而不是隐含在异常名、HTTP 状态码或 provider-specific text 中。

runtime 需要它来决定：

- 是否 continuation
- 是否 compact/recovery
- 是否直接 failed
- 是否转为 `requires_action`

## Runtime 与其它模块的边界

### Runtime

runtime 必须负责：

- 把 provider / tool / policy 失败归一化
- 依据错误与 stop reason 驱动 turn state machine
- 产出稳定 `TerminalState`

### Task

task status 不属于 runtime terminal taxonomy。

例如：

- `pending / running / completed / failed / killed`
  是 task lifecycle 状态
- 它们不是 turn terminal state 的子集或别名

task 若需要更细粒度 stop reason，应通过 task metadata 或 task event 表达。

### Gateway

gateway 可以把 runtime terminal state 投影到 channel，但不应重写 runtime taxonomy。

例如：

- Telegram 消息发送失败
- TUI 渲染中断
- Webhook delivery retry

这些是 gateway / channel delivery 问题，不应被混写成 runtime terminal state。

### Session

session 必须能够 durable 保留：

- 最近的 terminal-like stop reason
- `requires_action` 的结构化对象
- 恢复 turn 所需的最小状态

## 事件与终态的关系

事件流是过程事实，终态是结束归类。

推荐最小事件类型至少覆盖：

- `started`
- `progress`
- `requires_action`
- `completed`
- `failed`
- `cancelled`
- `aborted`
- `timed_out`

但：

- event 不等于 terminal state
- event 多次出现不意味着多次终态

## Convenience API 约束

若实现提供 direct-call API：

- 其返回结果、异常和终态必须可以从 core streaming contract 严格推导
- 不得因为 API 形态更简单，就把 `requires_action`、`cancelled`、`aborted` 折叠掉

## 规范结论

- `TerminalState` 只描述 turn/runtime execution unit 的结束方式
- `requires_action` 不是失败
- `cancelled` 不等于 `aborted`
- gateway delivery failure 不应重写 runtime terminal taxonomy
- task status 不能冒充 runtime terminal state
