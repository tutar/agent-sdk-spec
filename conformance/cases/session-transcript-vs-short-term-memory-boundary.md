# Case: Session Transcript Vs Short-Term Memory Boundary

## 目标

验证 transcript truth source 与 short-term continuity summary 的边界。

## Preconditions

- session 支持 append-only transcript
- session 支持 short-term memory
- implementation 支持 compact 或等价 continuity compression

## Ingress

1. 在一个长 session 中写入多轮 transcript
2. 生成或更新 short-term continuity summary
3. 在 compact 后继续推进并执行 resume

## Expected Runtime Behavior

- transcript 仍然是 durable truth source
- short-term memory 只辅助 compact 后 continuation 与 resume continuity
- implementation 能表达 short-term memory 的 coverage boundary 与稳定性

## Failure Conditions

- short-term memory 被当成 transcript 替代物
- compact 后无法追溯 transcript truth source
- resume 只依赖 summary，不能恢复 transcript-backed session semantics
