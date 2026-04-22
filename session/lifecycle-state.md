# Session Lifecycle State

## 职责

本文档定义 `Session` 的 observer-facing lifecycle / progress state。

这里的 lifecycle state 不等于：

- transcript
- working state
- restore snapshot
- UI state
- process state

它解决的是：

- 当前 session 是否正在运行
- 当前 session 是否空闲
- 当前 session 是否阻塞在需要外部动作的节点

## 一、为什么需要单独建模

session durable store 解决的是“历史”和“恢复”。

但 gateway、UI、remote control、SDK 等外部消费者还需要知道：

- 当前是否正在推进 turn
- 当前是否等待 approval / input / selection
- 当前是否已结束当前活跃 turn

如果不把这些显式建模，就会退化成：

- 从 transcript/event stream 反推状态
- 从 UI 局部状态猜当前状态
- 用字符串消息近似表达 `requires_action`

因此 `SessionLifecycleState` 必须是一等对象。

## 二、推荐最小状态

```text
SessionLifecycleState
  - idle
  - running
  - requires_action
```

含义：

- `idle`
  当前没有活跃 turn 正在推进
- `running`
  当前正在推进 turn、执行 tool、等待模型流式返回或处理 continuation
- `requires_action`
  当前被阻塞，等待外部系统或用户提供动作

## 三、RequiresAction

`requires_action` 不应只是一个布尔值。

它应携带结构化上下文，至少包括：

```text
RequiresActionDetails
  - tool_name
  - action_description
  - tool_use_id
  - request_id
  - input?
```

这样外部系统才能：

- 展示阻塞原因
- 恢复具体动作上下文
- 直接驱动 approval / question / review UI

permission approval 的专门路由与恢复语义见 [../harness/permission/approval-and-resume.md](../harness/permission/approval-and-resume.md)。

## 四、与其它状态的边界

### 与 Transcript 的边界

- transcript 负责 durable history
- lifecycle state 负责当前可观察运行状态

### 与 WorkingState 的边界

- working state 负责 continuation 所需结构化运行上下文
- lifecycle state 只负责外部可观察状态

`requires_action` 的文本描述不是 working state；
pending action binding 则属于 `WorkingState` / `ResumeSnapshot` 的一部分。

### 与 ProcessState / UIState 的边界

- process state 关心当前进程内部状态
- UI state 关心某个客户端的界面状态
- lifecycle state 必须独立于具体 UI 和进程实例被消费

## 五、推荐接口

```text
SessionLifecycleController
  - get_state(session_id) -> lifecycle_state
  - set_state(session_id, lifecycle_state, details?)
  - subscribe(session_id, listener)
```

推荐补充：

```text
SessionExternalMetadata
  - pending_action?
  - task_summary?
  - post_turn_summary?
```

## 六、推荐语义约束

- lifecycle state 必须能独立于 UI 消费
- `requires_action` 必须带结构化 details
- `idle` 应在真实完成当前 turn 后发出，而不是在中间停顿时误发
- lifecycle state 不应替代 restore state；恢复时必须同时恢复 pending action 的结构化绑定
- lifecycle state 可被 runtime / gateway 投影消费，但 ownership 仍在 `Session`

## 七、规范结论

- `SessionLifecycleState` 应成为 `Session` 模块的稳定接口之一
- 它是 observer-facing projection，不应被埋进 UI state、process state 或 transcript parsing 逻辑里
- `requires_action` 必须是结构化对象，而不是字符串消息
- `SessionLifecycleState` 不得替代 `WorkingState` 或 `ResumeSnapshot`
