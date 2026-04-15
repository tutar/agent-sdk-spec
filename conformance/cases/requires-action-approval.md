# Case: Requires Action Approval

## 目标

验证权限阻塞、用户批准、turn 继续执行这一标准审批流。

## Preconditions

- 有一个 tool 会触发 `ask`
- runtime 支持结构化 `requires_action`
- session 初始状态为 `idle`

## Ingress

1. 一条会触发该 tool 的 user message
2. 一条 approval response，表示用户批准

## Expected Lifecycle

阶段一：

- `idle -> running -> requires_action`

阶段二：

- `requires_action -> running -> idle`

## Expected Runtime Events

阶段一至少应出现：

- `turn_started`
- `requires_action`

阶段二至少应出现：

- `tool_started`
- `tool_result`
- `turn_completed`

## Expected Requires Action Payload

至少应包含：

- action type 或等价字段
- `tool_name`
- `tool_use_id`
- 可供 gateway / UI 消费的描述性信息

## Allowed Variance

- approval response 可以通过 Local、Cloud 任一 gateway 注入
- 实现可以把审批恢复放在新 turn 或 continuation 中完成，但外部语义必须等价

## Failure Conditions

- `requires_action` 不是结构化对象
- 批准后无法恢复到执行态
- 批准结果没有绑定到原 `tool_use`
