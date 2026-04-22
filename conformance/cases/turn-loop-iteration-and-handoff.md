# Case: Turn Loop Iteration And Handoff

## 目标

验证一次 turn 可以由 turn-local loop 跨多个 iteration 推进，并在 terminal 前进入稳定的 post-turn handoff boundary。

## Preconditions

- runtime 支持至少一次 continuation
- runtime 支持 tool loop
- runtime 支持 post-turn processing 或语义等价 handoff

## Ingress

1. 构造一条会触发 `tool_use` 的 normalized inbound
2. 让 runtime 至少完成一次 tool roundtrip
3. 让当前 turn 再进入一次 follow-up iteration
4. 最终完成该 turn

## Expected Runtime Semantics

- 一个 turn 可以包含多个 iteration
- `tool loop` 发生在单 iteration 内
- `recursive follow-up` 发生在 iteration 与 iteration 之间
- 每次新 iteration 前，context assembly 与 context editing 必须重新发生或语义等价重算
- turn core loop 结束后，可以进入稳定的 post-turn handoff boundary

## Expected Persistent Effects

- transcript 或语义等价 durable log 仍反映同一个 turn，而不是多个无关 turns
- handoff 触发的 post-turn effects 不得改写 turn-local core loop 的终态归类

## Allowed Variance

- handoff 可以同步或异步触发
- continuation 可由 tool results、compact recovery、output limit recovery、stop-hook retry 等触发

## Failure Conditions

- `tool loop` 与 `recursive follow-up` 无法区分
- 每次 continuation 被错误实现成新的独立 turn
- post-turn processing 被错误建模为 turn core loop 的额外 iteration
