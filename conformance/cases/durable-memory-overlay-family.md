# Case: Durable Memory Overlay Family

## 目标

验证 `user / project / team / agent / local` 都属于 durable-memory overlays，而不是新的 durable planes。

## Preconditions

- session / durable-memory subsystem 支持 overlays 或语义等价 binding 模型
- 实现支持 durable recall 与 write/consolidation

## Ingress

1. 准备一组分别绑定到 `user / project / team / agent / local` 的 durable payloads
2. 在不同 visibility、sharing 或 agent identity 条件下触发 recall
3. 对其中一部分 payload 触发 direct write、extraction 或 consolidation

## Expected Runtime Semantics

- `user / project / team / agent / local` 改变的是：
  - visibility
  - sharing
  - storage binding
  - sync surface
- 它们不应被实现成新的 payload taxonomy
- 它们不应被实现成新的 recall pipeline
- 它们不应被实现成新的 consolidation mechanism
- `team` 与 `agent` 若存在，应是 overlays 或语义等价层

## Expected Persistent Effects

- 不同 overlays 中的 durable payload 仍复用同一 entrypoint / payload / manifest / consolidation 分层，或语义等价结构
- overlay 可以影响写入根、同步范围与 recall visibility
- overlay 不应破坏 bounded recall 与 durable write-side 的分层语义

## Allowed Variance

- `team` overlay 可有不同 sync protocol
- `agent` overlay 可绑定到 user、project、local 或其它语义等价 backing
- `local` overlay 可绑定到 host-local 或 workspace-local private store

## Failure Conditions

- 把 `team memory` 建模成新的 durable plane
- 把 `agent memory` 建模成新的 payload system
- 把 `local memory` 建模成单独 runtime，而不是 private overlay
