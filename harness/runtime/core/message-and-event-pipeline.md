# Message And Event Pipeline

## 目标

本文定义 `Harness.Runtime.Core` 的消息与流式输出分层。

它关注的是：

- runtime 处理什么输入
- runtime 内部如何维护 turn 消息
- runtime 如何发出 stream-level 过程事实
- `tool_use / tool_result` 如何闭环

它不定义：

- raw channel protocol
- gateway egress projection
- viewer / UI message model

## 三层对象

runtime core 至少应区分三层：

- `NormalizedInbound`
- `RuntimeInternalMessage`
- `StreamEvent`

### NormalizedInbound

`NormalizedInbound` 是 gateway 已经归一化后的输入。

它可以来自：

- Feishu
- WeChat
- Telegram
- TUI
- CLI bridge
- API client

但 runtime core 不应看见这些渠道的 raw payload 差异。

### RuntimeInternalMessage

`RuntimeInternalMessage` 是 turn 内部的语义对象。

它至少应能承载：

- user input
- assistant message
- attachment-like injected message
- tool use summary
- tool result reinsertion
- compact boundary
- interruption / synthetic recovery message

它服务于：

- turn progression
- transcript append
- continuation
- tool pairing

### StreamEvent

`StreamEvent` 是 runtime 向上层暴露的流式过程事实。

它至少应允许表达：

- request start
- assistant delta
- assistant message complete
- tool progress
- compact / recovery boundary
- requires_action bookend

`StreamEvent` 不是 channel-specific egress，也不是 transcript。

## 推荐分层关系

```text
Raw Channel Input
  -> Gateway
  -> NormalizedInbound
  -> AgentRuntime
  -> RuntimeInternalMessage
  -> StreamEvent
  -> Runtime terminal state
```

对外投影到 host/channel 的行为，见 `runtime/projection/` 和 `gateway/`。

## Tool Use / Tool Result 闭环

这是 runtime core 的核心约束之一。

runtime 必须显式维护：

- 每个 `tool_use` 都有对应的最终 `tool_result` 或语义等价终结对象
- `tool_result` 必须能映射回对应 `tool_use_id`
- assistant continuation 在 `tool_result` 回接后继续，而不是开启一个新的无关 turn

允许：

- progress 先于最终 result
- 缺失 result 时由 runtime 插入结构化错误结果或语义等价修复对象
- 大结果以稳定引用方式进入 working messages

不允许：

- `tool_use` 悬空
- `tool_result` 脱离原 `tool_use_id`
- compact / recovery 后 pairing 漂移

## Ordering 约束

runtime 至少应保证这些顺序关系稳定：

- 同一 `tool_use_id` 的 `tool_result` 不能先于 `tool_use`
- tool result 回接后 assistant 才能继续推进该分支
- interruption / synthetic recovery message 不能破坏 pairing 约束
- compact boundary 是 runtime control fact，不应被误当成普通 assistant output

## Boundary And Control Messages

runtime 可以存在不直接进入 transcript 的 control message 或 boundary message。

例如：

- request start
- compact boundary
- synthetic recovery message
- stop hook 相关控制对象

约束：

- control message 可以存在
- 但不能与 user/assistant semantic message 混淆

## 与 Gateway 的边界

gateway 负责：

- raw channel ingress
- normalized inbound construction
- runtime facts 的 channel projection

runtime core 负责：

- normalized inbound 之后的内部消息与 stream pipeline

所以：

- `ExternalRuntimeEvent` 不是 runtime core 的主对象
- 它是 projection/gateway 的消费面

## 规范结论

- runtime core 应显式分层 `NormalizedInbound`、`RuntimeInternalMessage`、`StreamEvent`
- raw channel protocol 不应进入 runtime core
- `tool_use / tool_result` pairing 与 ordering 是核心约束
- stream event 不等于 transcript，也不等于 channel egress
