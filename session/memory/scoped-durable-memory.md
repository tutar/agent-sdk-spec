# Scoped Durable Memory

## 职责

`Scoped Durable Memory` 负责定义 durable memory 的作用域语义。

它不是新的 memory plane，而是 `DurableMemory` 的组织、共享和可见性维度。

它不同于：

- `session transcript`
  可恢复事件历史
- `short-term session memory`
  服务 continuity 的会话级摘要
- `memory injection`
  显式 durable context source 的注入机制
- `memory consolidation`
  durable memory 的提炼与整合

## 作用域模型

`ScopedDurableMemory` 是 durable memory 的作用域化形态。

推荐至少支持：

- `user`
  对所有项目通用
- `project`
  在当前项目内共享
- `agent`
  某类 agent 的专用长期知识
- `local`
  仅当前环境或当前机器有效

约束：

- `agent-scoped memory` 属于 durable memory scope，而不是单独的新 memory 类型
- scope 会影响 sharing / visibility / recall candidate selection
- scope 不应改变 durable memory 作为 recall object 的核心语义
- scope 是组织维度，不是 ownership 单元
- 当前产品绑定关系仍是 `1 Agent = 1 Global Long-Term Memory`

## 与 Recall / Injection 的关系

- scope 会影响 recall 的候选边界与可见性
- scope 不等于 injection source
- 某个 injection source 可以带有 scope，但 injection 与 scope 不是同一层

例如：

- `AGENTS.md` 可以作为带 scope 的显式 injection source
- `DurableMemoryRecord.scope` 仍然是 durable memory record 的组织字段
- 即使存在 `user / project / agent / local` scope，也不意味着产品上存在多份 agent-owned long-term memory

## 与其它 memory 子规范的边界

- 与 [memory-model.md](memory-model.md)
  `memory-model` 只定义 3 个核心 memory layers；本文件定义 durable memory 的 scope 维度
- 与 [memory-injection.md](memory-injection.md)
  injection 负责把显式 source 注入当前 turn；scope 负责 durable memory 的共享/可见性边界
- 与 [memory-consolidation.md](memory-consolidation.md)
  consolidation 可以读写不同 scope 的 durable memory，但不拥有 scope 定义本身

## 要解决的问题

- 如何表达 durable memory 在不同宿主/项目/agent 间的共享边界
- 如何让 `user / project / agent / local` 保持稳定语义
- 如何让 scope 影响 recall 和 visibility，而不改变 durable memory 核心对象模型
- 如何避免把 scope 错写成新的 memory plane

## 规范结论

- `Scoped Durable Memory` 应作为 `Session.memory` 下的独立子规范存在
- scope 是 durable memory 的维度，不是新的 memory layer
- 默认 scope 语义推荐为 `user / project / agent / local`
- `DurableMemoryRecord.scope` 足以承载当前 canonical object 需求，无需新增独立 scoped object
