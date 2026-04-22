# Attachment Assembly

## 职责

`AgentRuntime` 在完成 per-turn assembly 的 attachment 阶段时，调用 `AttachmentAssembly` 的稳定语义。

`AttachmentAssembly` 只定义 attachment-specific 的装配语义。

它回答的不是：

- attachment 是否属于独立 context plane
- attachment 在整体 assembly pipeline 的第几步出现
- attachment fragment 从哪个 provider 来

这些问题分别属于：

- [context-input-model.md](context-input-model.md)
- [context-assembly-pipeline.md](context-assembly-pipeline.md)
- [context-provider.md](context-provider.md)

本页只回答：

- attachment assembled 后最小应具有什么 envelope
- attachment 的 ordering 为什么必须 deterministic
- thread / agent scope 为什么必须显式建模
- audience 为什么不能和 model-visible 输入混在一起

## 最小对象

```text
AttachmentEnvelope
  - source
  - audience
  - thread_scope
  - ordering_class
  - payload_ref | inline_payload
  - provenance
```

## Ordering Classes

推荐最小 ordering classes：

- `user_input`
  用户在当前输入中显式附加的文件、资源、图片、mentions
- `all_thread`
  主线程与 worker/subagent 都可能需要的运行时附加上下文
- `main_thread_only`
  只对主线程或 leader 有意义的附加上下文
- `agent_scoped`
  只对某个特定 worker、teammate 或 agent task 可见的附加上下文

推荐稳定顺序：

1. `user_input`
2. `all_thread`
3. `main_thread_only`
4. `agent_scoped`

实现可以在类内进一步细分，但不应把 attachment 当作无序集合。

## Scope 语义

attachment 至少应支持：

- main session only
- all threads
- single worker / single agent scope

约束：

- 不是所有 attachment 都应被所有线程看到
- 不同 scope 的 attachment 可见性必须 deterministic
- `agent_scoped` attachment 不得被默认广播到全部 worker

## Audience 语义

同一条 attachment payload 可能面向：

- model-visible projection
- debug / audit only
- UI-only projection

约束：

- 不要把所有附加信息都直接送给模型
- audience 必须是 envelope 级语义，而不是 host 的隐含约定
- 同一 payload 可以有多个 projection，但它们的 audience 必须可区分

## Payload Form

attachment 应优先支持：

- `payload_ref`
- 有限 `inline_payload`

推荐语义：

- 大文件、长工具结果、诊断输出、task output 优先以 `payload_ref` 暴露
- 当前轮模型可见输入通常只拿 preview 或摘要
- 必要时再通过稳定 ref 二次读取

不要求具体 ref 形态，但要求其保持 transport-neutral。

## 与 Transcript 的边界

attachment 是运行时装配面，不等于 durable transcript 本体。

它可以被投影进消息流，但其：

- 来源
- 可见性
- ordering
- audience
- 治理策略

都独立于 transcript durable ownership。

## Mapping Guidance

本地默认实现里：

- `payload_ref` 常映射到本地文件、工具结果持久化路径、本地资源句柄
- `thread_scope` 常来自 runtime state、agent task state 或当前视图状态

云端兼容实现里：

- `payload_ref` 可以是远程对象、资源句柄、blob ref
- ordering、scope 和 audience 语义必须保持不变

## 规范结论

- `AttachmentAssembly` 只定义 attachment envelope 与装配规则
- attachment 必须有稳定 ordering、明确 scope、明确 audience 和可追溯 provenance
- attachment 不应被建模成无序文件列表
- attachment 不等于 transcript durable store
