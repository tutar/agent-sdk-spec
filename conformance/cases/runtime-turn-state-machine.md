# Case: Runtime Turn State Machine

## 目标

验证一次 turn 可以由同一个 runtime state machine 跨多次 provider call 与 continuation 完成。

## Preconditions

- ingress 已经由 gateway 规范化
- harness 支持至少一次 continuation
- turn 允许触发 tool loop、compact 或语义等价恢复路径

## Ingress

- 一条会导致 runtime 至少经历一次 continuation 的 normalized inbound

## Expected Runtime Semantics

- 同一 turn 可以跨多次 provider call 推进
- continuation reason 必须是显式 runtime fact，而不是隐含在 channel 行为中
- runtime 最终产出单一 `TerminalState`
- direct-call facade 如存在，也必须能还原为同一 turn state machine

## Expected Persistent Effects

- transcript 或等价 durable log 反映的是同一 turn 的结果，而不是多个无关 turns
- session 在 terminal 后仍可继续接受下一轮 ingress

## Allowed Variance

- continuation 可由 tool loop、compact recovery、output limit recovery、stop hook retry 等触发
- provider call 次数可以不同

## Failure Conditions

- 一次 continuation 被错误实现为新的独立 turn
- terminal outcome 只能从 gateway/channel 侧拼装，runtime 自己没有稳定终态
- runtime 无法表达 continuation reason
