# Case: Session Working State Vs Lifecycle State

## 目标

验证 `WorkingState` 与 `SessionLifecycleState` 的边界。

## Preconditions

- session 支持 structured lifecycle state
- session 支持 resume snapshot / working-state restore

## Ingress

1. 启动一个 session 并推进到需要 tool continuation 的中间状态
2. 将 lifecycle state 设为 `running` 或 `requires_action`
3. 对 session 执行 resume / wake

## Expected Runtime Behavior

- lifecycle state 只表达外部可观察状态
- working state 仍保留 pending action binding、active working window 或等价 continuation context
- restore 不得仅依赖 lifecycle state 重建 continuation

## Failure Conditions

- implementation 用 `idle / running / requires_action` 代替 working state
- pending action binding 丢失，只剩一条 requires-action 文本
- restore 时只能恢复 lifecycle state，无法继续原 continuation
