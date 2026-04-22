# Durable Memory Types

`memory-types/` 定义 durable memory payload 的标准 taxonomy。

这些类型在默认 local mapping 中通常由 `auto-memory` prompt/runtime 引入，但语义层属于 `Session.durable-memory` 的通用类型系统，而不是某个具体 runtime 的私有实现。

它们不是 4 套独立机制，而是同一 durable memory runtime 下的 4 类 payload 语义：

- `user`
- `feedback`
- `project`
- `reference`

这 4 类的权威语义应至少覆盖：

- 存什么
- 不存什么
- 什么时候写
- 如何用于未来回合
- shared/private scope 倾向
- consolidation / staleness 特征

taxonomy 只定义 durable payload 的语义分类，不直接定义：

- payload 载体格式
- resident index
- header manifest
- recall transport
- runtime binding

taxonomy 也不等于 scope / overlay binding。

例如：

- `memory-types/user`
  - 表示用户画像 payload
- `scopes/user-memory`
  - 表示 user-private durable backing

- `memory-types/project`
  - 表示项目背景 payload
- `scopes/project-memory`
  - 表示 project-scoped durable binding

这些结构语义分别见：

- [../auto-memory/topic-memory.md](../auto-memory/topic-memory.md)
- [../auto-memory/memory-entrypoint-index.md](../auto-memory/memory-entrypoint-index.md)
- [../auto-memory/frontmatter-and-header-manifest.md](../auto-memory/frontmatter-and-header-manifest.md)
- [../scopes/README.md](../scopes/README.md)

taxonomy 参与 recall 与 consolidation 的 interpretation，但不定义 runtime。本子目录应与 durable-memory 的 architecture、recall pipeline、write-side 页面联合阅读。

taxonomy 还必须受 durable memory 的排除规则约束。  
例如：

- code patterns
- architecture
- git history
- file paths
- ephemeral task details

不应被误存为以下任一类型。

- [user-memory.md](user-memory.md)
- [feedback-memory.md](feedback-memory.md)
- [project-memory.md](project-memory.md)
- [reference-memory.md](reference-memory.md)
