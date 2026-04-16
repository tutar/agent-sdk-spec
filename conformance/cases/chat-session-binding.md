# Case: Chat Session Binding

## 目标

验证 `1 Chat = 1 Session` 的绑定语义。

## Preconditions

- gateway 支持 channel / chat / session binding
- runtime 支持 supplement input

## Ingress

1. 某个 channel 上创建一个 chat
2. gateway 为该 chat 绑定一个 session
3. 对该 chat 连续发送多轮普通输入与 supplement input

## Expected Runtime Behavior

- 同一 chat 始终绑定同一个 session
- supplement input 不会创建第二个 session
- 一个 chat 不会同时绑定多个 session

## Failure Conditions

- 同一 chat 被错误分裂成多个 session
- supplement input 被当成新 chat 或新 session
