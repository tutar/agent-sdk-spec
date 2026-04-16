# Case: Agent Global Long Memory

## 目标

验证 `1 Agent = 1 Global Long-Term Memory` 的产品语义。

## Preconditions

- 同一个 agent 下存在多个 session
- session 支持 durable memory linkage / recall

## Ingress

1. session A 沉淀一条 durable memory
2. 在同一个 agent 下启动 session B
3. session B 发起一次需要该长期记忆的请求

## Expected Runtime Behavior

- session B 可从同一个 agent long-term memory 中召回该记忆
- long-term memory 不要求按 user 或 chat 拆分 ownership
- short-term memory 仍然保持 session 独占

## Failure Conditions

- 同一 agent 下的 session 无法共享 global long-term memory
- long-term memory 被错误建模为 per-chat short-term continuity
