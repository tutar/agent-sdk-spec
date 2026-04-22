# Case: Durable Memory Taxonomy Vs Scope Axis

## 目标

验证 durable memory 的 payload taxonomy 与 scope overlay 是两条不同维度，而不是同一对象语义。

## Preconditions

- durable-memory package 支持 durable payload taxonomy
- 实现支持 scope、overlay 或语义等价的 binding 模型

## Ingress

1. 写入一条 `user`-typed durable payload
2. 把该 payload 绑定到 user-private、local 或语义等价 overlay
3. 写入一条 `project`-typed durable payload，并把它绑定到 project 或 team overlay
4. 触发一次 recall 与一次 consolidation

## Expected Runtime Semantics

- payload taxonomy 回答“这条 memory 记的是什么语义”
- scope overlay 回答“这条 memory 绑定到哪里、谁可见、是否共享”
- 同名项如 `user`、`project` 出现在两条轴上时，运行时语义必须可区分
- scope 可影响 visibility、sharing、candidate source 或 sync surface
- scope 不得改写 payload taxonomy 本身

## Expected Persistent Effects

- 同一 payload type 可落在不同 overlays 中
- 同一 overlay 可承载不同 payload types
- consolidation 可更新 payload 内容或 overlay 内的 durable binding 状态，但不得把 taxonomy 与 scope 合并成单一字段语义

## Allowed Variance

- scope 可以建模为字段、binding object、storage root selector 或语义等价结构
- taxonomy 可以通过 type tag、record class 或语义等价元数据表示

## Failure Conditions

- 把 `memory-types/user` 与 user-private overlay 混成同一概念
- 把 `memory-types/project` 与 project-scoped binding 混成同一概念
- scope 改写 payload taxonomy，或 taxonomy 隐式决定唯一 overlay
