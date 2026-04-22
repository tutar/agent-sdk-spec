# Durable Memory Scopes

`scopes/` 定义 durable memory 的 overlay family。

本目录回答的是：

- durable memory 绑定到哪里
- 谁可以看到和更新它
- 是否共享、是否同步、是否与 agent identity 或本地环境绑定

它不回答：

- 这条 memory 的 payload 语义是什么
- recall pipeline 如何做 bounded selection
- consolidation 如何 merge 或 prune payload

因此：

- `memory-types/`
  - 定义 payload taxonomy
- `scopes/`
  - 定义 storage / visibility / sharing overlays

这两条轴必须同时存在，但不能混成一个对象模型。

本目录包含：

- [user-memory.md](user-memory.md)
- [project-memory.md](project-memory.md)
- [team-memory.md](team-memory.md)
- [agent-memory.md](agent-memory.md)
- [local-memory.md](local-memory.md)

联合阅读：

- [../durable-memory-scopes-and-overlays.md](../durable-memory-scopes-and-overlays.md)
- [../durable-memory-recall-pipeline.md](../durable-memory-recall-pipeline.md)
- [../durable-memory-write-and-consolidation.md](../durable-memory-write-and-consolidation.md)
- [../memory-types/README.md](../memory-types/README.md)

规范要求：

- overlay 改变 binding、sharing、visibility、sync surface
- overlay 不改变 payload taxonomy、recall 基本分层或 consolidation 核心职责
- `user / project` 若同时出现在 taxonomy 与 scope 轴上，必须明确消歧
