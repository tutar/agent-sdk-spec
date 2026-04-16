# Case: Single Active Harness Lease

## 目标

验证同一 session 任一时刻只有一个 active harness lease。

## Preconditions

- session 支持 wake / resume
- harness 支持恢复或迁移

## Ingress

1. harness A 获取某个 session 的 active lease
2. harness A 在推进中退出或被替换
3. harness B 通过 wake / resume 接管同一个 session

## Expected Runtime Behavior

- 任一时刻只有一个 harness 可推进该 session
- resume 是 lease 交接，不是复制出第二个可写 session
- short-term memory 跟随 session 恢复

## Failure Conditions

- 两个 harness 同时推进同一个 session
- harness 重启导致 session short-term memory 丢失
