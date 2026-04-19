# Case: Runtime Tool Use And Tool Result Pairing

## 目标

验证 runtime core 显式维护 `tool_use / tool_result` pairing 与 ordering。

## Preconditions

- 有至少一个可用工具
- runtime 支持把工具结果回接到同一 turn

## Ingress

- 一条会触发至少一个 `tool_use` 的 normalized inbound

## Expected Runtime Semantics

- 每个 `tool_use` 必须有对应的最终 `tool_result` 或语义等价终结对象
- `tool_result` 必须能映射回原 `tool_use_id`
- assistant continuation 发生在 result 回接之后
- pairing 语义在 compact、retry、resume 后保持稳定

## Expected Persistent Effects

- transcript 或 working message view 中，tool result 与对应 tool use 的关联可恢复
- 若结果被外存化，应保留稳定引用

## Allowed Variance

- 允许 progress 先于最终 result
- 允许以结构化错误结果结束某次 tool use
- 允许大结果以引用替代内联内容

## Failure Conditions

- `tool_use` 悬空
- `tool_result` 脱离原 `tool_use_id`
- assistant 在 result 回接前错误继续
- compact / recovery 导致 pairing 漂移
