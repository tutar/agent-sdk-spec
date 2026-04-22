# Assistant Agent Profile Memory And Continuity

## 目标

验证 `AssistantAgentProfile` 可以采用 append-first durable memory accumulation，并保持 viewer / reattach continuity，而不改变 `durable-memory`、`Gateway`、`Task` 的 ownership boundary。

## Preconditions

- host 宣称支持 `Harness.AgentProfiles`
- 至少实现了一个 assistant-mode host profile
- session 支持 durable memory、continuity 或语义等价能力

## Ingress

1. 启动一个 assistant agent session
2. 让 session 累积多轮用户交互和长期值得保留的记忆信号
3. 观察 durable memory 是立即重写索引，还是先追加 capture stream 再延后 consolidation
4. 在会话持续期间触发 viewer attach、reattach，或语义等价的 continuity 恢复路径

## Expected Runtime Semantics

- assistant agent 可以采用 append-first durable memory capture
- durable memory 的 consolidation 可以延后，不要求每次新信号都立即重写长期记忆索引
- viewer / reattach continuity 必须与这种 memory accumulation 方式兼容
- assistant profile 不得把 `durable-memory`、`Gateway`、`Task` 的 ownership 吞并进 profile 自身

## Expected Persistent Effects

- 记忆 capture 流与 consolidated durable memory 应能被区分
- continuity 恢复不应要求把 append-first capture 伪装成 transcript 本体

## Allowed Variance

- append-first 可以是 daily log、capture stream、journal file 或语义等价对象
- continuity 可以通过 viewer attach、history paging、resume projection 或语义等价方式实现

## Failure Conditions

- assistant profile 强制把 durable memory 退化成“每次即时重写索引”的唯一模式
- assistant profile 的 continuity 依赖错误改写 transcript 或 durable memory ownership
- profile 语义与 `durable-memory`、`Gateway`、`Task` 的边界混淆
