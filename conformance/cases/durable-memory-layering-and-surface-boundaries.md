# Case: Durable Memory Layering And Surface Boundaries

## 目标

验证 durable memory 会区分 architecture、recall、write/consolidation、payload taxonomy 与 scope overlay，而不是把这些 surface 混成单一 memory 机制。

## Preconditions

- durable-memory package 支持跨 session durable knowledge
- 实现支持 durable recall、durable write/consolidation、payload taxonomy、scope 或语义等价机制
## Ingress

1. 准备一组可被 recall 的 durable payload
2. 触发一轮读取 durable knowledge 的 turn
3. 再触发一轮会更新 durable knowledge 的 interaction，并运行 consolidation 或语义等价流程
4. 观察 project / team / agent / local 等 scope 是否只改变可见性或 binding，而不改变 durable-memory 基本机制

## Expected Runtime Semantics

- durable memory 至少应区分：
  - payload taxonomy
  - bounded recall pipeline
  - write/consolidation pipeline
  - scope overlay
- dream 不等于独立 memory plane，而是 consolidation mode
- memory types 参与 recall 和 consolidation 的 interpretation，但不定义 runtime 本体
- team / agent memory 若存在，应被实现为 overlay 或语义等价层，而不是新的 durable plane
- 同名项如 `user / project` 若同时出现在 taxonomy 与 scope 轴上，必须可区分

## Expected Persistent Effects

- recall 不应改写 durable store 来模拟成功
- consolidation 可更新 durable payload 与 entrypoint/index，但不能破坏 session restore
- scope overlay 可以改变 durable binding / sharing，但不应破坏 recall 与 consolidation 的分层语义

## Allowed Variance

- default local mapping 可以是 `MEMORY.md` + topic files + header manifest，也可以是语义等价结构
- team / agent overlay 可以有不同存储根、同步协议或加载策略

## Failure Conditions

- 把 dream 当成独立 memory plane
- 把 team / agent memory 当成完全不同的 durable mechanism，而不是 overlay
- recall、write/consolidation 之间没有可观察的语义边界
