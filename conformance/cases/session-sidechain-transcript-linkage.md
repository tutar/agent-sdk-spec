# Case: Session Sidechain Transcript Linkage

## 目标

验证主 session transcript 与 sidechain / subagent transcript 的 durable linkage。

## Preconditions

- session 支持 branch / sidechain transcript
- restore 支持 branch refs

## Ingress

1. 创建一个主 session transcript
2. 在该 session 下创建一个 sidechain 或 subagent transcript
3. 再次恢复主 session

## Expected Persistent Effects

- 主 transcript 与 child transcript 的 parent linkage 可追溯
- restore 主 session 时能区分 main transcript 和 sidechain leaf
- branch refs 或等价对象能进入 resume snapshot

## Failure Conditions

- sidechain transcript 被当成主 transcript 末端
- child transcript 与 parent session 的 linkage 不可追溯
- branch tracing 被错误降级成 durable memory 或 task output
