# Case: Auto Memory Resident Index And Bounded Recall

## 目标

验证默认 auto-memory runtime 不会把整个 durable store 无界注入每轮 turn，而是通过 resident index 与 bounded payload recall 分层进入上下文。

## Preconditions

- session / memory subsystem 支持 auto-memory 或语义等价的默认 durable runtime
- durable memory store 中存在 `entrypoint/index + topic payloads` 的语义分层
- runtime 支持在 context assembly 中引入 durable memory

## Ingress

1. 预先准备一个包含 resident index 与多条 durable payload 的 auto-memory 空间
2. 发起一轮只需要其中少量 durable memory 的请求
3. 观察 resident index、header selection 与 recalled payload 的进入方式

## Expected Runtime Semantics

- resident index 与 durable payload 必须语义分离
- resident index 的典型映射可以是 `MEMORY.md`，durable payload 的典型映射可以是 topic memory records / files
- runtime 不应把整个 auto-memory store 无界注入当前 turn
- recall 必须 bounded，例如：
  - selected payload count
  - total bytes / token budget
  - already surfaced dedupe
- selection 可先经过 header / manifest 层，但 resident index 本身不应承载 payload 正文
- recalled durable memory 进入 context plane，而不是 transcript
- stale / freshness 至少应允许被建模，而不是把 recalled durable memory 视为当前真相

## Expected Persistent Effects

- resident index 继续作为 durable store 的入口存在
- recall 不应通过改写 transcript 模拟成功
- consolidation 之后，新的 durable payload 仍应遵守同样的 resident-index / bounded-recall 语义

## Allowed Variance

- resident index 可以是 `MEMORY.md`、manifest 或语义等价结构
- durable payload 可以是 topic memory files、records 或语义等价 durable units
- selection 可以使用规则、检索器或模型辅助路径
- recalled payload 可以进入 attachment-like context、structured context 或专用 memory plane

## Failure Conditions

- 每轮自动加载全部 durable payload 正文
- resident index 与 payload 无法区分
- recall 没有任何 budget / count / dedupe 语义
- recalled memory 被伪装成 transcript 原消息
