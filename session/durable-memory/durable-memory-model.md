# Durable Memory Model

## 目标

本规范定义 `Session` 域中的 durable memory 模型。

这里的 durable memory 不是：

- transcript restore
- current turn working state
- short-term continuity summary

它是跨 session 的可召回知识层。

## 核心原则

- durable memory 服务 recall，不服务 restore
- `1 Agent = 1 Global Long-Term Memory`
- 多个 session 可以共享同一个 agent long-term memory
- scope 是 durable memory 的共享/可见性维度，不是新的 memory plane
- 显式 durable injection 属于 durable memory 输入面，而不是 transcript

## 稳定语义

### 1. durable recall layer

`DurableMemory` 负责保存未来回合可能有价值的抽象信息，例如：

- 用户偏好
- 项目规则与约束
- workflow gotchas
- 外部 reference pointers

### 2. not restore state

durable memory 不应承担：

- session restore
- 当前 turn working state
- per-session continuity

### 3. shared across sessions

多个 session 可以共享同一个 agent long-term memory。

当前产品语义下，long-term memory 不应被误建模为 per-chat short-term continuity。

## 生命周期

推荐 durable memory 生命周期：

1. 完整 turn 或 query loop 结束
2. 从 transcript 中提炼 durable candidate
3. 写入 durable store
4. 周期性 consolidate / cleanup
5. 后续 turn 通过 recall 读取

## 与其它子规范的边界

- 与 [../short-term-memory/README.md](../short-term-memory/README.md)
  short-term memory 服务 continuity；本页定义 cross-session recall layer
- 与 [durable-memory-recall.md](durable-memory-recall.md)
  recall 语义见该页
- 与 [memory-types/README.md](memory-types/README.md)
  durable payload 的标准 taxonomy 见该子目录
- 与 [scoped-durable-memory.md](scoped-durable-memory.md)
  scope 语义见该页
- 与 [memory-injection.md](memory-injection.md)
  file-backed durable injection 见该页
- 与 [auto-memory/README.md](auto-memory/README.md)
  default local-first durable runtime 见该子目录
