# Case: Tool Call Roundtrip

## 目标

验证一次标准 `tool_use -> tool_result -> assistant completion` 回路。

## Preconditions

- session 初始状态为 `idle`
- 至少有一个可执行 tool 被暴露给 harness
- tool execution 不需要额外审批

## Ingress

- 一条会触发该 tool 的 user message

## Expected Lifecycle

- session lifecycle: `idle -> running -> idle`
- turn terminal state: `completed`

## Expected Runtime Events

至少应出现：

- `turn_started`
- `tool_started`
- `tool_result`
- `turn_completed`

允许出现：

- `assistant_delta`
- `tool_progress`
- `assistant_output`

## Expected Message Pairing

- 每个 `tool_use` 必须有且仅有一个最终 `tool_result`
- `tool_result` 必须能被映射回对应的 `tool_use_id`
- tool result 进入后，本轮 assistant 能继续完成

## Allowed Variance

- tool 可以流式返回 progress
- 具体工具输出内容不参与本 case 合规判断
- tool 调用次数允许为 1 次或多次，但每次配对必须完整

## Failure Conditions

- `tool_use` 与 `tool_result` 配对丢失
- tool 执行完成后 turn 无法继续
- 非并发安全工具被错误并发执行
