# Policy Engine

## 职责

`PolicyEngine` 负责在工具真正执行前做策略判定，并在需要时把会话推进到 `requires_action`。

权限不应被实现为简单弹窗，而应是一个可组合的策略系统。

## 必须覆盖的判定层

- 工具级 allow/deny
- 模式级限制
- 路径/工作目录限制
- sandbox 升级
- hook/classifier 审查
- async/background 会话降级策略

## 标准结果

```text
PolicyDecision
  - allow
  - deny(reason)
  - ask(reason, requires_action)
  - passthrough(reason)
```

其中：

- `passthrough`
  仅用于策略链内部，表示当前检查器不做最终裁决，继续交给后续检查器
- 对外最终结果不应直接暴露 `passthrough`

## `requires_action` 规范

阻塞不是异常字符串，而应是结构化对象。

```text
RequiresAction
  - tool_name
  - action_description
  - tool_use_id
  - request_id
  - input
```

## 设计要求

- 可见工具过滤和执行时授权必须是两层逻辑
- 权限原因必须可解释
- 审批请求必须可序列化
- 非交互会话必须支持自动拒绝或安全降级
- 即使底层模型原生支持某些敏感能力，权限裁决仍应由 SDK 控制
- `ask` 与 `requires_action` 必须分层：前者是策略决策，后者是 session 阻塞态投影
- 拒绝过多时应允许从自动拒绝退化到 prompting
- 工具中断语义不应混进权限结果，而应由执行层单独建模

## 默认策略顺序

1. 静态 deny rule
2. tool 输入校验
3. tool-specific permission check
4. hook/classifier
5. sandbox/working directory 检查
6. ask/deny/allow 决策

推荐补充：

7. denial tracking / fallback-to-prompting

## 拒绝追踪

当策略系统包含 classifier / hook / 自动拒绝时，推荐引入 denial tracking。

最小要求：

```text
DenialTrackingState
  - consecutive_denials
  - total_denials
```

```text
DenialFallbackPolicy
  - record_denial(state) -> state
  - record_success(state) -> state
  - should_fallback_to_prompting(state) -> boolean
```

其职责是：

- 防止系统无限自动拒绝
- 在连续拒绝过多时退回到 prompting / ask 语义

## 与中断语义的边界

权限决策只回答：

- 能否执行
- 是否需要审批

它不回答：

- 执行后能否被取消
- 用户新输入到来时是 cancel 还是 block

这些应由 tool execution 层通过 `interruptBehavior` 或等价语义单独建模。

## 当前仓库映射

- 权限主逻辑见 [utils/permissions/permissions.ts](../../cc/utils/permissions/permissions.ts)
- 拒绝追踪见 [utils/permissions/denialTracking.ts](../../cc/utils/permissions/denialTracking.ts)
- 会话阻塞态见 [utils/sessionState.ts](../../cc/utils/sessionState.ts)
- 工具中断语义见 [Tool.ts](../../cc/Tool.ts) 和 [services/tools/StreamingToolExecutor.ts](../../cc/services/tools/StreamingToolExecutor.ts)

## 规范结论

- 权限系统必须结构化
- `passthrough` 可作为内部策略链语义，但不应成为最终对外结果
- `requires_action` 必须是一等 runtime 状态
- `ask` 与 `requires_action` 不能混成一个概念
- denial tracking 应被视为策略退化机制
- 所有策略判定都必须可追踪、可解释
- 权限责任不能下放给底层模型
