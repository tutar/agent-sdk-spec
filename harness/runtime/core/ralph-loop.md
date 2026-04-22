# Ralph Loop

## 职责

`RalphLoop` 定义 `Harness.Runtime.Core` 中一次 turn 的本地执行回路。

这里的 `loop` 不是指某个具体函数名，而是指：

- 一个 turn 不等于一次 provider RPC
- 一个 turn 可以跨多个 iteration 推进
- `tool loop` 与 `recursive follow-up` 共同构成 turn-local continuation mechanism

它只讨论：

- turn 如何进入
- iteration 如何推进
- `tool loop` 与 `recursive follow-up` 的边界
- context editing 在 turn 中的正式位置
- terminal 之前如何进入 post-turn handoff

它不讨论：

- raw channel ingress
- session durable ownership
- task lifecycle
- post-turn processor 的内部实现

## 最小接口

```text
LoopIterationState
  - working_messages
  - normalized_input
  - continuation_reason?
  - pending_action?
  - active_tool_calls?

RalphLoop
  - run_iteration(state) -> iteration_events, next_state
  - continue_or_stop(state) -> continue | terminal
```

允许实现把这些接口内联到 `AgentRuntime`，但语义上必须能区分：

- runtime lifecycle
- turn-local loop progression

## Turn Execution Unit

一次标准 turn 至少应经过这些阶段：

1. `turn entry`
   runtime 接受 normalized inbound 并建立本轮 execution unit
2. `transcript append boundary`
   runtime 在 query 前为本轮新输入建立 durable append / checkpoint 边界
3. `context assembly`
   runtime 组装当前 iteration 的候选上下文
4. `context editing`
   runtime 对候选上下文执行 request-shaping
5. `model sampling`
   runtime 发起 provider 请求并消费 assistant streaming / stop facts
6. `post-sampling signals`
   runtime 处理 hook、verification recommendation、其它 turn-local signal
7. `tool loop`
   若出现 `tool_use`，runtime 执行工具并回接 `tool_result`
8. `recursive follow-up`
   runtime 判断是否以新的 iteration 继续当前 turn
9. `terminal or handoff`
   turn 结束或交给 post-turn processing

## Tool Loop 与 Recursive Follow-Up

这两层必须显式区分。

### Tool Loop

`tool loop` 是单个 iteration 内的工具执行阶段。

它至少应保证：

- 同一 assistant 输出中的多个 `tool_use` 可以在同一 iteration 内闭环
- 每个 `tool_use` 都有对应的最终 `tool_result` 或语义等价终结对象
- `tool_result` 回接到同一 turn，而不是开启无关新 turn

### Recursive Follow-Up

`recursive follow-up` 是 iteration 与 iteration 之间的 continuation mechanism。

它至少应保证：

- tool results 回接后可进入下一轮 provider call
- compact、output limit recovery、stop-hook retry、approval recovery 等 continuation reason 可被稳定建模
- continuation reason 是显式 runtime fact，而不是隐含在 UI 或 channel 行为中

因此：

- `tool loop` 发生在单 iteration 内
- `recursive follow-up` 发生在 iteration 与 iteration 之间

## Context Assembly

在真正进入 sampling 前，runtime 必须先完成 `context assembly`。

它至少应能组装：

- system prompt skeleton
- structured user/system context
- tool surface
- attachment-like injected inputs
- durable memory recall results 或语义等价输入
- runtime continuation 所需的 control messages

上下文 provider、assembly 与 governance 的细节，见 `Harness.ContextEngineering`。

## Context Editing

`context editing` 是 turn-local loop 中的正式阶段，不是实现细节。

它发生在：

- model sampling 之前
- 每个 iteration 内

也就是说，只要 turn 继续进入新的 follow-up iteration，runtime 就应重新执行这一阶段的语义等价逻辑。

### 分层退化要求

runtime 至少应支持从低损到高损的上下文整形分层，语义上至少覆盖：

1. `content externalization / replacement`
   大 payload 被稳定引用替代，而不是直接丢弃
2. `lightweight trimming`
   先尝试历史裁剪或等价低损缩减
3. `microcompact-like editing`
   在不进入 full rewrite 的前提下做局部上下文收缩
4. `working-view projection`
   允许只改变当前轮可见 working view，而不改写 transcript
5. `transcript-visible compaction rewrite`
   作为更重的最后手段

### 同步语义

对当前 iteration 来说，`context editing` 必须表现为同步 request-shaping：

- runtime 必须先拿到 editing 后的 working messages
- 才能进入本轮 provider sampling

但某些持久化副作用可以异步，只要：

- 当前 iteration 的输入语义已稳定
- 恢复与追溯语义保持一致

### 边界

`context editing` 不等于：

- post-turn processing
- durable memory consolidation
- verification worker

它是 turn-local loop 的一部分。

## Model Sampling

runtime 在 `model sampling` 阶段至少应能处理：

- assistant delta
- assistant message completion
- stop reason
- withheld recoverable errors
- fallback model 或语义等价降级路径

如果某 iteration 未产生 `tool_use`，turn 可以直接走向 terminal 或 handoff。  
如果产生 `tool_use`，则进入 `tool loop`。

## Post-Sampling Signals

采样结束后、tool loop 之前，runtime 可以处理 turn-local signals。

允许包括：

- post-sampling hooks
- verification recommendation
- attachment-like reminder signals

约束：

- recommendation signal 不等于独立 execution phase
- verification 若真正执行，仍应通过 normal tool invocation 进入 tool loop
- `reflection` 不应被建模为 turn 内独立 phase

## Transcript Boundary

runtime 必须在 query 前建立稳定的 transcript append / checkpoint boundary。

要求：

- 新输入一旦被 turn 接受，就应能被 durable 追溯
- 即使 provider 尚未返回，也不应丢失本轮起点
- 恢复时不得把同一 turn 错误拆成多个无关 turns

具体 transcript durable ownership 仍属于 `Session`。

## Terminal And Handoff

一次 turn 结束时，`RalphLoop` 至少应产出以下两类结果之一：

- `terminal`
  - 本轮达到稳定结束点
- `handoff`
  - 本轮 core loop 结束，并触发 post-turn processors

约束：

- post-turn processing 不属于 turn core loop 本体
- 但 handoff boundary 必须由 runtime core 稳定触发

## 规范结论

- `RalphLoop` 是一次 turn 的 turn-local execution loop，而不是 provider RPC 的别名
- 一个 turn 可以包含多个 iteration
- `tool loop` 与 `recursive follow-up` 必须显式分层
- `context editing` 是 turn 内正式阶段，且必须支持从低损到高损的分层退化
- recommendation signal 与 execution phase 不得混淆
- post-turn processing 发生在 turn terminal / handoff 之后，而不属于 `RalphLoop` 本体
