# Agent Runtime

## 职责

`AgentRuntime` 是 `HarnessInstance` 内部的执行编排面。

它不只负责单次 turn state machine，还负责：

- startup bootstrap 之后的 active runtime 建立
- active session binding / resume / switch orchestration
- turn dispatch into per-turn loop
- post-turn handoff 的稳定触发
- process shutdown 边界上的 runtime cleanup orchestration

它消费的是 gateway 已归一化后的输入，而不是 raw channel payload。

## 它负责什么

`AgentRuntime` 的最小稳定职责应包括：

- 接收 normalized inbound 与当前 session working state
- 建立或恢复 active session binding
- 把一次 ingress dispatch 到单个 turn execution unit
- 组装本轮模型可见输入
- 发起一次或多次模型请求
- 消费 streaming 输出
- 检测 `tool_use`
- 调用工具执行
- 把 `tool_result` 接回同一 turn
- 在 compact、retry、stop hook、requires_action 等条件下继续或停止
- 触发 post-turn handoff 与 runtime-visible tail work
- 产出本轮 terminal state 与可投影的 runtime facts
- 在 clear / resume / continue / shutdown 时切换或结束当前 active binding

默认本地映射通常表现为 query loop，但规范只要求稳定语义，不要求具体实现形态。

## 它不负责什么

以下职责不属于 `AgentRuntime`：

- 外部 channel 协议接入
- chat / conversation identity
- raw ingress parsing
- session durable ownership
- durable transcript / checkpoint / resume snapshot store
- durable memory store
- task registry、task output、task retention / eviction
- sandbox provisioning 或 execution target lifecycle

这些能力分别归：

- `Gateway`
- `Session`
- `Session.MemoryConsolidation`
- `Harness.Task`
- `Sandbox`
- `Orchestration`

这里要区分两层：

- runtime **必须拥有** session binding / switching orchestration
- runtime **不拥有** session durable store 的 source of truth

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
  - bind_session(handle) -> active_binding
  - restore_and_bind(handle) -> active_binding
  - switch_session(binding_ref) -> active_binding
  - shutdown(context) -> runtime_shutdown_result
```

约束：

- `run_turn()` 必须是状态机，不是一次 provider RPC 的别名
- direct-call convenience API 如存在，也必须可严格还原为同一 streaming contract
- runtime 必须支持一次 turn 内的多次 continuation
- session binding 可以隐藏在实现内部，但语义上必须存在

## Runtime Lifecycle

`AgentRuntime` 至少应显式拥有这些生命周期阶段：

1. `startup ready`
   runtime 已完成 bootstrap，具备接受 ingress 与建立 active binding 的能力
2. `session bind`
   建立新的 active session binding，先于首轮模型采样
3. `session restore and rebind`
   基于既有 session durable state 恢复并重新建立 active binding
4. `turn dispatch`
   把一次 normalized inbound 推进成一个 turn execution unit
5. `post-turn handoff`
   在 turn terminal 或 stop boundary 上触发 post-turn processors
6. `session switch or end`
   clear current binding、restore another binding、或 startup-time continue existing binding
7. `process shutdown`
   在进程退出边界上结束 active runtime 并执行 cleanup orchestration

## Session Lifecycle 责任

runtime 必须显式拥有 session lifecycle orchestration。

至少应覆盖：

- `fresh session start`
  - 为当前 runtime 建立新的 active session binding
- `resumed session start`
  - restore-and-rebind，而不是 transcript replay
- `active turn during session`
  - runtime 消费 session transcript、continuity、restore inputs，但不拥有其 durable store
- `session switch`
  - clear current session and start fresh
  - restore and bind another session
  - startup-time continue / resume existing session
- `session end vs process shutdown`
  - session end 是 runtime orchestration 边界，不等于 OS/process exit

如果某实现把 session binding / switching 隐藏在 gateway、host 或 session 自身内部，它就不再是稳定的 runtime 边界。

## Turn Loop 责任

runtime 必须显式拥有 turn-local execution loop 责任。

至少应覆盖：

- turn entry
- context assembly
- context editing
- provider sampling
- tool loop
- recursive follow-up
- terminal or post-turn handoff

具体 turn / loop 规范见 [ralph-loop.md](ralph-loop.md)。

## Continuation 责任

runtime 必须显式拥有 continuation 责任。

至少应覆盖：

- `tool_use` 触发的 continuation
- compact 后 continuation
- `max_output_tokens` 或语义等价输出限制后的恢复
- prompt-too-long / overflow 后的恢复
- `requires_action` 后的稳定阻塞
- stop hook retry 或语义等价恢复

如果某实现把这些 continuation 隐藏在 adapter、channel 或 UI 层，它就不再是稳定的 runtime 边界。

## 与 Gateway 的边界

`Gateway` 负责：

- 外部 chat/channel 输入
- input normalization
- session/chat binding
- control routing
- runtime facts 的 egress projection

`AgentRuntime` 负责：

- active runtime lifecycle
- session binding / switching orchestration
- turn progression
- tool loop
- runtime terminal outcome

因此：

- Feishu、WeChat、Telegram、TUI、API client 都不是 runtime 的一部分
- runtime 不应直接处理 raw channel payload
- runtime 可以拥有 session binding orchestration，但不应拥有 channel/chat binding ownership

## 与 Session 的边界

`Session` 负责：

- transcript
- checkpoint / resume snapshot
- short-term continuity
- restore inputs
- sidechain / branch transcript

`AgentRuntime` 负责：

- 建立、恢复、切换、结束 active binding
- 在 turn 中读取 session durable state 所需的最小 slice
- 把 turn-derived durable updates 交给 session 或 post-turn 子平面

因此：

- runtime 不拥有 session durable store
- session 也不拥有 runtime execution loop

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

## 与 Post-Turn 的边界

`AgentRuntime` 必须拥有 post-turn handoff 的触发责任，  
但不应把所有 post-turn processor 吞并进 core loop。

这意味着：

- post-turn processing 可以异步
- 但触发边界必须由 runtime 稳定提供

具体后处理面见 [../post-turn/post-turn-processing.md](../post-turn/post-turn-processing.md)。

## 规范结论

- `AgentRuntime` 是 `HarnessInstance` 内部的执行编排面，而不只是单次 turn state machine
- 它消费 normalized ingress，而不是 raw channel input
- 它必须拥有 session binding / switching orchestration，但不拥有 session durable store
- 一次 turn 可以跨多次 provider call、tool loop 与 continuation 完成
- turn-local loop 细节应与 runtime lifecycle 总入口分离，详见 `ralph-loop.md`
- chat/channel/UI 不属于 runtime core，而属于 `Gateway`
