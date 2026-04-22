# Case: Reference Vs User Memory Pointer And Private Boundary

## 目标

验证 `ReferenceMemory` 是 pointer memory，而 `UserMemory` 是始终 private 的 personalization layer，两者边界稳定。

## Preconditions

- session / memory subsystem 支持 `ReferenceMemory` 与 `UserMemory` 或语义等价类型
- durable memory subsystem 支持 shared/private split，或语义等价 scope 模型

## Ingress

1. 提供一条 external resource 及其 purpose
2. 提供一条用户 role / knowledge / stable preference 信息
3. 触发 durable memory 写入与后续 recall

## Expected Runtime Semantics

- `ReferenceMemory` 保存的是 lookup path 与 purpose，而不是外部系统当前事实快照
- recalled `ReferenceMemory` 应引导去 authoritative external source 查当前状态，而不是直接充当答案
- `UserMemory` 保存的是 personalization context，而不是 behavior rule
- `UserMemory` 在支持 shared/private split 的实现中必须保持 private
- `ReferenceMemory` 通常应偏 shared/team，但不应因此吞并 user profile 语义
- `UserMemory` 作为 payload semantics 必须与 user/private/local overlay 语义分离

## Expected Persistent Effects

- external pointer 会以 `ReferenceMemory` 或语义等价记录进入 durable store
- user profile 会以 private durable record 进入 durable store
- consolidation 可清理失效 external pointer，但不得把 user profile 外溢到 shared/team memory

## Allowed Variance

- `ReferenceMemory` 可额外建模 authority kind、purpose、resource type
- `UserMemory` 可额外建模 role、knowledge profile、stable preferences 的细分字段
- user-private overlay 可以存在，但不应与 `UserMemory` payload 语义混成同一字段

## Failure Conditions

- 外部系统当前状态被直接固化为 `ReferenceMemory`
- `ReferenceMemory` 缺失 purpose，导致 recall 只有无上下文链接
- `UserMemory` 被升级到 shared/team scope
- `UserMemory` 被当作 behavior rule 使用，或 `ReferenceMemory` 被当作 project background 使用
- `UserMemory` payload semantics 与 user/private/local overlay semantics 被混成单一概念
