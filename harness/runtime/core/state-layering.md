# State Layering

## 目标

本文定义 `Harness.Runtime.Core` 与相邻子系统之间的状态所有权边界。

重点不是列字段，而是回答：

- 哪些状态属于 runtime core
- 哪些状态属于 gateway
- 哪些状态属于 session durable layer
- 哪些状态属于 task / mcp / plugin 等领域对象

## 推荐分层

建议至少显式区分：

- `ProcessState`
- `RuntimeExecutionState`
- `GatewayInteractionState`
- `DomainStateRegistry`
- `SessionDurableState`

## ProcessState

`ProcessState` 是当前 harness 进程或 worker 的运行态。

例如：

- environment snapshot
- runtime caches
- counters
- feature latches
- provider / host level temporary cache

它通常是：

- 本地的
- 易失的
- 不直接等于一次 turn 的执行状态

## RuntimeExecutionState

`RuntimeExecutionState` 是 runtime core 自己拥有的 turn-local state。

例如：

- 当前 normalized inbound
- working messages
- continuation reason
- active tool loop state
- current provider request context
- compact / retry guards
- pending action / blocked reason

这层状态只服务 turn progression。

## GatewayInteractionState

`GatewayInteractionState` 属于 gateway，而不属于 runtime core。

例如：

- chat identity
- channel-specific delivery state
- reply target / thread target
- inbound dedup state
- webhook ack / retry cursor
- TUI / bot / webhook side的 interaction bookkeeping

这层存在的原因是：

- agent-spec 的 agent 不自带 UI
- Feishu / WeChat / Telegram / TUI / API client 都只是 channel 入口
- 与这些 channel 有关的交互状态应止步于 gateway

## DomainStateRegistry

`DomainStateRegistry` 是各子系统自己的状态集合。

例如：

- tasks
- mcp connection state
- plugin / skill runtime state
- notification queue

约束：

- runtime core 可以读取它们需要的事实
- 但不应吞并这些子系统的 source of truth

## SessionDurableState

`SessionDurableState` 属于 `Session`。

例如：

- transcript
- checkpoint
- resume snapshot
- short-term continuity
- durable memory linkage

约束：

- runtime core 消费 session durable state
- 但 durable ownership 仍在 session 子域

## 推荐抽象

```text
ProcessState
  - environment
  - runtime_caches
  - counters
  - feature_latches
```

```text
RuntimeExecutionState
  - normalized_input
  - working_messages
  - continuation_reason?
  - tool_loop_state?
  - pending_action?
  - compact_guard?
```

```text
GatewayInteractionState
  - channel_identity
  - chat_binding
  - delivery_state
  - inbound_dedup
  - control_interaction_state
```

```text
DomainStateRegistry
  - tasks
  - mcp
  - plugins
  - notifications
  - other_subsystems
```

## 边界结论

- runtime core 不应默认拥有 UI / chat interaction state
- chat/channel 相关交互状态属于 gateway
- task、mcp、plugin 等状态属于 domain registry，而不是 runtime core 自己的字段集合
- durable transcript / checkpoint / resume state 属于 session

## 规范结论

- `ProcessState`、`RuntimeExecutionState`、`GatewayInteractionState`、`DomainStateRegistry`、`SessionDurableState` 应显式分层
- runtime core 只拥有 turn-local execution state
- gateway 拥有 channel/chat 交互状态
- session 拥有 durable state
