# Runtime State Projection And Task Boundary

## 目标

验证 `Harness.Runtime` 能把 session、turn、task 的 observer-facing facts 对外投影，同时不混淆 `Harness.Task` 对 task lifecycle 的所有权。

## Preconditions

- host 支持 runtime-visible session / turn 状态
- host 支持至少一种 task-like async execution object
- host 至少存在一种独立于 transcript 的外部观察面

## Ingress

启动一个会产生后台 task 的 turn，并观察：

- turn 进入运行态
- task 被注册并开始推进
- task 产生 progress
- task 到达 terminal 状态
- turn 或 session 回到可观察的 stable state

## Expected Runtime Semantics

- observer 必须能区分 session / turn state 与 task-derived facts
- `task_started`、`task_progress`、`task_notification` 或语义等价对象可以作为 runtime projection surfaces 暴露
- 这些 projection 必须与 transcript / message stream 分离
- task-derived projection 不得被当作 task registry 本体
- runtime 对 task facts 的 outward projection 不得改变 `Harness.Task` 对 task lifecycle、output handle、terminal dedupe 的 ownership

## Expected Persistent Effects

- 若 task subsystem 提供 durable task state，source of truth 仍在 task side
- runtime projection surface 可以是 ephemeral event stream、status API 或语义等价机制，不要求自身变成 durable task store

## Allowed Variance

- 可以使用 event stream、queue + drain、callback、bridge message 或 polled status API
- 可以不使用 `task_started` / `task_progress` / `task_notification` 这些字面事件名，只要语义等价
- task-derived projection 可以与 runtime 自身状态共用一个外部观察通道，但语义必须仍可区分

## Failure Conditions

- 外部观察者无法区分 session / turn state 与 task-derived projection
- task progress 或 terminal facts 只能通过 transcript 猜测
- runtime projection 被实现成 task registry 的别名，导致 ownership boundary 消失
- task lifecycle 的 source of truth 被错误转移到 runtime projection surface
