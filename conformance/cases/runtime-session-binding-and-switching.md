# Case: Runtime Session Binding And Switching

## 目标

验证 `Harness.Runtime` 拥有 session binding / switching orchestration，但不拥有 session durable store。

## Preconditions

- runtime 支持 fresh session start
- runtime 支持 resume 或语义等价 restore-and-rebind
- session durable state 与 runtime process state 分离

## Ingress

1. 启动一个 fresh session，并完成至少一轮普通 turn
2. clear 当前 session，并在同一 runtime 中开始一个 fresh binding
3. restore 并绑定另一个已有 session
4. 在 restored session 上继续执行一轮新的 turn

## Expected Runtime Semantics

- fresh session start 必须先建立 active session binding，再进入首轮 turn
- clear 必须结束当前 binding，并开始新的 binding
- resume 必须 restore-and-rebind，而不是创建一个与原 session 无关的新会话
- runtime 必须拥有 binding / switching orchestration
- session durable store 的 source of truth 不得转移到 runtime

## Expected Persistent Effects

- clear 后的新 binding 不得覆盖旧 session durable history
- restored session 的 durable history、resume inputs 与 side state 仍可追溯
- restored session 上的新消息必须追加到原有 durable history，而不是写入无关新会话

## Allowed Variance

- clear 可以在同一进程或新 worker 中完成，只要新的 binding 语义稳定
- resume source 可以来自 transcript、event log、数据库或语义等价 durable store

## Failure Conditions

- runtime 无法表达 fresh binding、clear-and-start-fresh、restore-and-rebind 的差异
- resume 只是 transcript replay，缺失 active binding 恢复
- runtime 把 session durable ownership 吞并为自身内部状态
