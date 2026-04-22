# Durable Memory Scopes And Overlays

## 职责

`DurableMemoryScopesAndOverlays` 定义 durable memory 的 scope axis。

它回答的是：

- durable memory 绑定到哪里
- 谁可以看到和更新这条 durable memory
- 是否共享、是否同步、是否与某个 agent identity 绑定

它不回答：

- 这条 durable memory 记的是什么语义
- recall pipeline 如何做 bounded selection
- consolidation 如何 merge 或 prune payload

因此本页讨论的是 overlay / binding，不是 payload taxonomy。

## 核心结论

durable memory 至少有两条正交维度：

1. `payload taxonomy`
   - `user / feedback / project / reference`
2. `scope / overlay binding`
   - `user / project / team / agent / local`

这两条轴可以出现同名项，但语义不同。

例如：

- `memory-types/user`
  - 回答“这条 memory 是不是用户画像”
- `scopes/user-memory`
  - 回答“这条 memory 是否落在用户私有 durable store 中”

- `memory-types/project`
  - 回答“这条 memory 是否是项目背景”
- `scopes/project-memory`
  - 回答“这条 memory 是否绑定到当前项目 durable store”

scope 改变的是 durable memory 的：

- visibility
- sharing
- storage root or persistence binding
- sync surface
- loading policy

scope 不改变的是 durable memory 的：

- payload taxonomy
- topic payload model
- recall pipeline 的基本分层
- consolidation 的核心职责

## Overlay Family

推荐把 durable memory 的 overlay family 至少建模为：

- `user`
  - 用户私有、跨项目或 user-root capable 的 overlay
- `project`
  - 当前项目绑定的默认 durable overlay
- `team`
  - 团队共享 overlay
- `agent`
  - agent identity / agent type scoped overlay
- `local`
  - host-local 或 workspace-local private overlay

这些 overlay 都不是新的 durable plane。

它们复用同一 durable-memory：

- payload taxonomy
- resident entrypoint / index
- bounded recall pipeline
- write and consolidation semantics

## Overlay 影响什么

overlay 影响：

- durable payload 写到哪个 durable store
- recall 时哪些 candidates 可见
- 是否允许跨成员共享
- 是否有 sync surface
- 是否与特定 agent identity、项目或本地环境绑定

overlay 不应改变：

- 这条 payload 是 `user`、`feedback`、`project` 还是 `reference`
- topic payload 与 resident entrypoint/index 的结构分层
- direct write、extraction、dream consolidation 的核心职责

## 默认本地映射

当前默认本地映射通常可理解为：

- `project overlay`
  - 默认 durable runtime root
- `team overlay`
  - 位于 project durable store 之上的 shared subtree 或语义等价层
- `agent overlay`
  - agent identity scoped subtree、root 或语义等价绑定
- `user overlay`
  - user-private durable backing root
- `local overlay`
  - host-local / workspace-local private backing root

规范不强制具体目录结构，但要求这些 overlay 差异在语义上可区分。

## 与 Recall / Consolidation / Instruction Loading 的关系

- scope 会影响 recall 的 candidate source、visibility 与 selection boundary
- scope 会影响 write destination、sharing surface 与 sync boundary
- scope 不等于 instruction source

因此：

- `AGENTS.md` / `rules`
  - 属于 harness-level instruction markdown loading
- durable memory overlays
  - 属于 store-backed durable knowledge 的 binding 与 sharing 模型

二者不应混成同一条机制。

## 与其它子规范的边界

- 与 [durable-memory-architecture.md](durable-memory-architecture.md)
  `durable-memory-architecture` 定义 durable memory 总模型；本页只定义 scope axis
- 与 [memory-types/README.md](memory-types/README.md)
  `memory-types/` 定义 payload taxonomy；本页定义 storage / visibility overlays
- 与 [durable-memory-recall-pipeline.md](durable-memory-recall-pipeline.md)
  recall 负责在 overlay 边界内选择和读取 durable memory；本页不定义 recall 算法
- 与 [durable-memory-write-and-consolidation.md](durable-memory-write-and-consolidation.md)
  consolidation 可以写入不同 overlay，但不拥有 overlay 定义本身
- 与 [scopes/README.md](scopes/README.md)
  各具体 overlay 页见该子目录

## 规范结论

- durable memory 必须把 `payload taxonomy` 与 `scope / overlay binding` 作为两条不同轴建模
- `team`、`agent`、`local` 以及 `user`、`project` 一类 binding 都属于 overlays，不是新的 durable plane
- 同名项如 `user`、`project` 出现在两条轴上时，必须明确消歧
- 具体 overlay family 见 [scopes/README.md](scopes/README.md)
