# Case: Context Editing Progressive Degradation

## 目标

验证 runtime 在长会话或预算压力下，能够按从低损到高损的顺序执行 context editing，而不是只有单次 full compact。

## Preconditions

- runtime 支持至少两种以上上下文编辑策略或语义等价机制
- 其中至少一种为低损编辑，例如 payload externalization、history trimming、working-view projection
- session 支持 compact 或语义等价的强重写路径

## Ingress

1. 构造一个包含超长 payload、较长历史和连续工作状态的 session
2. 触发一次普通 turn，使 runtime 先尝试低损编辑
3. 继续增加上下文压力，直到需要更强编辑
4. 观察 runtime 的退化顺序

## Expected Runtime Semantics

- runtime 不应把“上下文过长”直接等同于 full compact
- runtime 应优先尝试低损编辑，再进入更强的 compaction rewrite
- editing 路径之间的职责必须可区分：
  - externalization / trimming
  - projection
  - transcript-visible rewrite

## Expected Persistent Effects

- 若低损 editing 已足够恢复预算，不应强制写入 compact boundary
- 若进入强重写，必须建立明确边界或语义等价边界

## Allowed Variance

- 低损 editing 的具体算法可以不同
- 不要求所有实现都暴露完全相同的中间状态，只要求退化顺序和边界稳定

## Failure Conditions

- runtime 只支持单次 full compact
- 低损 editing 与 transcript rewrite 被混成同一行为
- 本应通过低损 editing 恢复的场景被不必要地升级成强重写
