# Case: Basic Turn

## 目标

验证一次无 tool、无审批、无中断的最小 turn 行为。

## Preconditions

- session 已存在，初始状态为 `idle`
- harness 可正常调用模型
- 当前 turn 不需要 tools

## Ingress

- 一条已经过 gateway 归一化的普通 user message

## Expected Lifecycle

- session lifecycle: `idle -> running -> idle`
- turn terminal state: `completed`

## Expected Runtime Events

至少应出现：

- `turn_started`
- `assistant_output` 或等价的 assistant terminal output event
- `turn_completed`

不应出现：

- `tool_started`
- `requires_action`
- `turn_failed`

## Expected Persistent Effects

- transcript append 一条 user message
- transcript append 一条 assistant output
- session 仍可继续接受下一轮 ingress

## Allowed Variance

- streaming 过程中可出现 `assistant_delta`
- assistant 输出的具体文本不参与本 case 合规判断
- usage / token 数值允许因模型而异

## Failure Conditions

以下任一情况均视为不合规：

- session 未回到 `idle`
- turn 被错误标记为 failure
- assistant output 未进入 transcript 或等价 durable log
- runtime 需要读取 raw channel payload 才能推进 turn
