# Case: Harness Deployment Binding Equivalence

## 目标

验证 `Local` 与 `Cloud` 下，`Harness` 作为一个完整整体保持相同的运行时语义；变化的只是它调用其它核心模块的 binding。

## Preconditions

- 至少存在一个 `Local` hosting profile
- 至少存在一个 `Cloud` 或语义等价的远端 hosting profile
- harness 能与 `Session`、`Tools`、`Sandbox` 协作完成一轮标准 turn

## Ingress

- 一条可在 `Local` 与 `Cloud` 下语义等价处理的普通 turn 输入

## Expected Runtime Semantics

- `Local` 与 `Cloud` 下都应表现为同一个完整 harness
- runtime、context engineering、task、permission、multi-agent 仍属于 harness 内部
- `Local` 可 direct-call `Session` / `Tools` / `Sandbox`
- `Cloud` 可通过 RPC / remote binding 调用这些模块
- binding 变化不应改变：
  - turn semantics
  - terminal state
  - requires_action semantics
  - tool continuation semantics
  - task ownership boundary

## Expected Persistent Effects

- 同一语义输入在 `Local` / `Cloud` 下，session durable effects 应语义等价
- 不应要求 `Cloud` 下把 harness 自身拆成多个独立远程组件才能成立

## Allowed Variance

- `Local` 可为单进程、同机多进程或 local IPC
- `Cloud` 可为 remote worker、service process 或语义等价远端部署
- 远端调用协议和字段名可以不同，只要语义一致

## Failure Conditions

- `Cloud` 下 harness 被错误建模成一组独立远程子组件
- binding 变化导致 runtime / context / task / permission 语义漂移
- `Local` 与 `Cloud` 的差异超出部署位置和调用方式本身
