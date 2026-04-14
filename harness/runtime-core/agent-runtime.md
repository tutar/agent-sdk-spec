# Agent Runtime

## 职责

`AgentRuntime` 负责管理一轮完整的 agent turn 状态机。

它的职责不是调用一次模型，而是维持从“输入消息”到“最终停机状态”的完整推进过程。

这里的 `AgentRuntime` 应明确理解为 harness 内部的 loop，而不是整个 managed-agent 系统的全部。它依赖 session 提供 durable history，依赖 sandbox/hands 提供执行能力。

## 标准接口

```text
TurnState
  - messages
  - turn_count
  - transition
  - requires_action

AgentRuntime
  - run_turn(run_turn_params) -> event_stream, terminal_state
```

## Runtime 必须负责的事情

- 基于模型能力视图选择 native path 或 harness path
- 标准化输入消息
- 组装 system/user context
- 发起模型请求并消费流式返回
- 检测 `tool_use`
- 调用 `ToolExecutor`
- 把 `tool_result` 重新接回消息链
- 在 budget、compact、stop hook 条件下继续或终止

## 不属于 AgentRuntime 的职责

- durable session storage
- sandbox provisioning 或重建
- 凭据持有与 vault 访问
- remote hand 的网络拓扑管理

这些应分别归属 `Session`、`Sandbox / Hands`、`Security Boundary` 和 `Managed Orchestration`。

## 状态机要求

- 必须显式记录当前 turn 的 `messages`
- 必须显式记录 continuation 原因
- 必须区分正常结束、权限阻塞、预算结束、失败结束
- 必须支持多次 API 请求共同组成一个 turn

## 终止状态

建议最小终止态：

- `completed`
- `requires_action`
- `failed`
- `aborted`

## 默认恢复能力

- `prompt_too_long` 后触发 compact/recovery
- `max_output_tokens` 后允许有限次数恢复
- tool 执行中断后回填错误或拒绝结果

## 设计原因

只把 agent 看成单次模型调用会丢失三类关键能力：

- 多轮工具调用
- 长 turn 自动继续
- 长上下文恢复

## 当前仓库映射

- 主循环实现见 [query.ts](../../../query.ts)
- turn 预算见 [query/tokenBudget.ts](../../../query/tokenBudget.ts)

## 规范结论

- `run_turn()` 必须是状态机
- 返回值必须是事件流
- runtime 必须拥有恢复和继续的责任
- runtime 必须把模型能力差异屏蔽在 adapter 层之后
- runtime 是 harness 的一部分，不应吞并 session 或 sandbox 的职责
