# Agent Runtime

## 职责

`AgentRuntime` 是 `HarnessInstance` 内部的 turn state machine。

它只负责一件事：

- 把一次已经规范化的 ingress 推进成一个完整 turn

这里的 ingress 不是 Feishu、WeChat、Telegram、TUI 或 API client 的原始协议输入。  
这些外部 chat/channel 语义属于 `Gateway`，`AgentRuntime` 只消费 gateway 已经归一化后的输入。

## 它负责什么

`AgentRuntime` 的最小稳定职责应包括：

- 接收 normalized inbound 与当前 session working state
- 组装本轮模型可见输入
- 发起一次或多次模型请求
- 消费 streaming 输出
- 检测 `tool_use`
- 调用工具执行
- 把 `tool_result` 接回同一 turn
- 在 compact、retry、stop hook、requires_action 等条件下继续或停止
- 产出本轮 terminal state 与可投影的 runtime facts

源码中的默认本地映射是 query loop：一次 turn 可以跨多次 provider call 完成，而不是“一次输入对应一次模型调用”。

## 它不负责什么

以下职责不属于 `AgentRuntime`：

- 外部 channel 协议接入
- chat / conversation identity
- session binding
- egress projection 到 Feishu、WeChat、Telegram、TUI 或其它 channel
- durable transcript / checkpoint / resume snapshot
- task registry、task output、task retention / eviction
- sandbox provisioning 或 execution target lifecycle

这些能力分别归：

- `Gateway`
- `Session`
- `Harness.Task`
- `Sandbox`
- `Orchestration`

## 最小接口

```text
TurnState
  - normalized_input
  - working_messages
  - turn_count
  - continuation_reason?
  - pending_action?

AgentRuntime
  - run_turn(params) -> stream, terminal_state
```

约束：

- `run_turn()` 必须是状态机，不是一次 provider RPC 的别名
- direct-call convenience API 如存在，也必须可严格还原为同一 streaming contract
- runtime 必须支持一次 turn 内的多次 continuation

## Turn State Machine

一次标准 turn 至少会经过这些阶段：

1. `ingress accepted`
   gateway 已完成 input normalization，runtime 开始处理本轮输入
2. `context assembled`
   runtime 结合 session working state、context engineering、tool surface 组装模型输入
3. `provider streaming`
   runtime 消费 assistant delta / assistant message / stop reason
4. `tool loop`
   若出现 `tool_use`，runtime 调用工具并把 `tool_result` 接回本轮消息链
5. `continuation or stop`
   根据 tool result、stop hook、compact、permission 阻塞、预算和 provider error 决定继续或停止
6. `terminal`
   产出 `TerminalState`

## Continuation 责任

runtime 必须显式拥有 continuation 责任。

至少应覆盖：

- `tool_use` 触发的 continuation
- compact 后 continuation
- `max_output_tokens` 或语义等价输出限制后的恢复
- prompt-too-long / overflow 后的恢复
- `requires_action` 后的稳定阻塞

如果某实现把这些 continuation 隐藏在 adapter 或 channel 层，它就不再是稳定的 runtime 边界。

## 与 Gateway 的边界

`Gateway` 负责：

- 外部 chat/channel 输入
- input normalization
- session/chat binding
- control routing
- runtime facts 的 egress projection

`AgentRuntime` 负责：

- turn progression
- tool loop
- runtime terminal outcome

因此：

- Feishu / WeChat / Telegram / TUI / API client 都不是 runtime 的一部分
- 它们都是 `Gateway.ChannelAdapter` 的不同落地
- runtime 不应直接处理 raw channel payload

## 与 Task 的边界

`AgentRuntime` 可以触发后台 agent、workflow、shell 等 task-like execution object，  
但不拥有 task lifecycle。

runtime 负责：

- 在 turn 内触发它们
- 感知必要的 runtime-visible facts

task 子系统负责：

- task registration
- task state
- output handle
- retention / eviction
- terminal dedupe

## 规范结论

- `AgentRuntime` 是 `HarnessInstance` 内部的 turn state machine
- 它消费 normalized ingress，而不是 raw channel input
- 一次 turn 可以跨多次 provider call 与 tool loop 完成
- continuation、compact recovery、requires_action 都属于 runtime 核心职责
- chat/channel/UI 不属于 runtime core，而属于 `Gateway`
