# Case: Auto Memory Write Paths And Consolidation

## 目标

验证 auto-memory runtime 中的 direct write、background extraction 与 dream consolidation 是三条不同但可协作的 durable write paths。

## Preconditions

- durable-memory package 支持 auto-memory 或语义等价的默认 durable runtime
- runtime 支持 foreground durable write
- runtime 支持 background extraction、dream consolidation 或语义等价机制

## Ingress

1. 发起一轮会形成 durable memory 的交互，并允许主路径直接写入 durable memory
2. 再完成一轮未显式写 memory、但可由 background extraction 补提取的交互
3. 之后触发一次 dream-style consolidation 或语义等价的 cross-session consolidation
4. 如实现支持 assistant-style operating mode，再观察 append-first / consolidate-later 变体

## Expected Runtime Semantics

- direct write、background extraction、dream consolidation 必须是可区分的路径
- 若主路径已写入 durable memory，background extraction 应允许跳过重复范围
- dream consolidation 不等于 transcript rewrite，也不等于普通 extraction
- assistant-style append-first mode 若存在，只改变 operating mode，不改变 durable memory 的 ownership 或 consolidation 语义

## Expected Persistent Effects

- direct write 会产生或更新 durable memory
- background extraction 可补充未被 foreground path 显式保存的 durable signal
- dream consolidation 可 merge / prune / tighten durable memory 与 resident index
- 三条写入路径的 durable target 仍应保持 topic payload + entrypoint/index 的语义分层
- consolidation 失败或延迟不应破坏 session restore 或提前暴露未提交 memory

## Allowed Variance

- extraction 与 consolidation 可以同步、异步、后台 task 或受限 subagent 执行
- assistant-style variant 可以使用 append-only logs、capture stream 或语义等价结构

## Failure Conditions

- 把三条 durable write paths 混成单一不可区分操作
- direct write 后 extraction 重复写入同一语义范围，且没有 skip / dedupe 语义
- dream consolidation 通过改写 transcript 伪装成功
- assistant-style append-first mode 改变了 durable memory 的 ownership 边界
