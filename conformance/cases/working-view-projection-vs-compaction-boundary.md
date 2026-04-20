# Case: Working-View Projection Vs Compaction Boundary

## 目标

验证 working-view projection 与 transcript-visible compaction rewrite 是两种不同语义，且恢复边界稳定。

## Preconditions

- runtime 支持一种不改写 transcript 的 working-view reduction 机制，或语义等价机制
- runtime 支持 compact boundary 或语义等价的 transcript-visible rewrite
- session 支持 resume 或语义等价恢复

## Ingress

1. 构造一段较长历史，使 runtime 先进入 working-view projection
2. 观察此时 transcript 是否仍保持原始历史
3. 继续增加预算压力，使 runtime 进入 compaction rewrite
4. 触发一次 resume / restore

## Expected Runtime Semantics

- working-view projection 不等于 transcript rewrite
- projection 可以改变当前轮模型可见上下文，但不应要求 durable transcript 同步重写
- compaction rewrite 必须建立显式边界或语义等价边界

## Expected Persistent Effects

- projection state 必须可恢复或可重建
- rewrite 后旧 projection state 不得跨 boundary 错误污染新历史

## Allowed Variance

- projection state 可以存于 transcript metadata、side store 或等价恢复结构
- compact boundary 的具体字段可以不同，只要求语义稳定

## Failure Conditions

- projection 和 rewrite 在语义上无法区分
- rewrite 后旧 projection state 仍继续作用于 post-boundary history
- resume 后无法重建 projection 或识别 rewrite 边界
