# Startup And Turn-Zero Context

## 职责

`AgentRuntime` 在 fresh start、resume、delegated worker start 和 first turn 之前调用本页定义的 startup context 语义。

它解决的核心问题是：不要把 `session_start`、`agent_start`、`turn_zero`、`resume_start` 混成同一种 prompt patch 或 prompt override。

## 最小对象

```text
StartupContext
  - kind
  - first_use_only
  - reentry_policy
  - transcript_visibility
  - dedup_policy
  - payload
```

## 推荐分类

- `session_start`
  主会话刚开始时注入的一次性或受 reentry 策略控制的上下文。
- `agent_start`
  delegated agent / teammate 首次启动时的 role-local briefing。
- `turn_zero`
  在第一轮模型调用前提供的一次性 kickoff 信息。
- `resume_start`
  从 restore / reentry 恢复时重新宣布的上下文。

## 语义要求

### 1. session-start 与 agent-start 必须分开

`session_start` 面向主会话连续性。

`agent_start` 面向：

- worker role-local briefing
- delegation additions
- team-local coordination bootstrap

二者不能互相污染。

### 2. startup context 不是 bootstrap prompt

bootstrap prompt 是 stable system skeleton。

startup context 是一次性或按 reentry 策略重放的运行时上下文。

### 3. turn-zero briefing 不是 agent prompt

首轮一次性说明不应通过修改 agent prompt 本体实现。

否则会导致：

- prompt cache churn
- 首轮/后续轮语义混淆
- agent identity 与 kickoff payload 混层

### 4. resume-start 必须显式建模

从 restore 或 UI reentry 回来时，某些上下文需要重新宣布：

- 当前执行模式
- pending action
- viewed transcript binding
- active worker identity

这些内容不能假设模型会从压缩后的历史中自动推断。

## 与相邻页面的边界

- 与 [bootstrap-prompts.md](bootstrap-prompts.md)
  `BootstrapPrompts` 定义稳定 skeleton；本页只定义 lifecycle-scoped injected context
- 与 [../assembly/context-assembly-pipeline.md](../assembly/context-assembly-pipeline.md)
  本页定义 startup-only input 的语义；per-turn 装配顺序见 `ContextAssemblyPipeline`
- 与 [../assembly/context-provider.md](../assembly/context-provider.md)
  provider 可以产出 startup fragments，但 startup lifecycle 分类由本页定义
- 与 [../governance/context-governance.md](../governance/context-governance.md)
  startup context 不是预算治理或 editing 触发面

## 规范结论

- startup-only context 必须是独立语义层
- `session_start`、`agent_start`、`turn_zero`、`resume_start` 必须区分
- runtime 不应通过污染 bootstrap prompt 或 agent prompt 本体来表达一次性 briefing
