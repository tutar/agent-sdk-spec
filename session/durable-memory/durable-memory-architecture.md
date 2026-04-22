# Durable Memory Architecture

## 职责

`DurableMemory` 定义 `Session` 域中的跨 session durable knowledge plane。

它回答的是：

- 哪些信息属于 durable knowledge
- durable knowledge 如何被组织、召回、整理和共享
- durable memory 与 short-term memory、transcript、restore 的边界是什么

它不等于：

- transcript 或 event log
- session restore snapshot
- current turn working state
- short-term continuity summary

## 核心结论

durable memory 不应被建模成单一文件、单一路径或单一算法。

推荐至少区分五层：

1. `durable payload taxonomy`
   - `user / feedback / project / reference`
2. `resident entrypoint/index`
   - 常驻、低成本、目录化 durable surface
3. `bounded recall pipeline`
   - manifest/header scan、candidate selection、payload load
4. `write and consolidation pipeline`
   - direct write、extraction、cross-session consolidation
5. `scope overlays`
   - `user / project / team / agent / local` 等可见性与共享维度

这五层必须语义分离。

## Durable Memory 保存什么

durable memory 保存未来回合仍可能有价值、且不应从 transcript 重新恢复的长期知识，例如：

- 用户画像与稳定偏好
- 行为 guidance 与 validated workflow feedback
- 不能从 code / git 直接推导的项目背景
- 外部 authoritative source 的 pointer

durable memory 不应承担：

- session restore
- 当前 turn tool continuation
- compact working set
- 短期 continuity summary
- 纯代码结构、git 历史、瞬时 task 进度

## 默认体系化理解

在当前默认本地映射中：

- `auto-memory/`
  定义 durable memory 的默认 local-first runtime
- `memory-types/`
  定义 durable payload 的标准 taxonomy
- `MEMORY.md`
  代表 resident entrypoint/index 的典型本地映射
- topic memory files
  代表 durable payload 的典型本地映射
- header/frontmatter manifest
  代表 recall candidate layer 的典型本地映射

规范不强制这些具体文件名，但这些层次必须可区分。

## 生命周期

推荐 durable memory 生命周期：

1. interaction 或 background path 产生 durable signal
2. durable signal 进入 direct write、extraction 或 append-first capture
3. durable payload 被写入或更新
4. resident index / manifest 被 tighten、refresh 或重建
5. 后续 turn 通过 bounded recall 读取
6. 周期性 consolidation / dream 合并重复项、修 stale、压实索引

## 常见误建模与本规范修正

本子域容易出现的误建模包括：

- 把 `dream` 写成独立 memory plane
- 把 harness-level instruction markdown loading 混成 durable recall
- 把 `team memory`、`agent memory` 写成新的 durable mechanism
- 把 `memory-types` 写成 runtime，而不是 payload taxonomy

本规范统一采用以下口径：

- `dream` 是 consolidation mode
- `AGENTS.md` / `rules` 一类 instruction markdown loading 归 `Harness.ContextEngineering`
- `team` / `agent` memory 是 scope overlay
- `user / project` 这类同名项若同时出现在 taxonomy 与 scope 轴上，必须显式消歧
- `memory-types` 参与 recall 和 consolidation，但不定义 runtime

## 与其它子规范的边界

- 与 [../short-term-memory/README.md](../short-term-memory/README.md)
  short-term memory 服务 per-session continuity；本子域服务 cross-session durable knowledge
- 与 [durable-memory-recall-pipeline.md](durable-memory-recall-pipeline.md)
  recall 只定义 durable 读侧
- 与 [durable-memory-write-and-consolidation.md](durable-memory-write-and-consolidation.md)
  写入、提炼、整合见该页
- 与 [dream-consolidation.md](dream-consolidation.md)
  dream 是 consolidation 的后台/跨 session 模式
- 与 [durable-memory-scopes-and-overlays.md](durable-memory-scopes-and-overlays.md)
  scope overlay 语义见该页
- 与 [scopes/README.md](scopes/README.md)
  具体 overlay family 见该子目录
- 与 [../../harness/context-engineering/instruction-markdown/README.md](../../harness/context-engineering/instruction-markdown/README.md)
  `AGENTS.md` / `rules` 一类 file-backed instruction source 属于 `Harness.ContextEngineering`，不属于 durable memory
- 与 [auto-memory/README.md](auto-memory/README.md)
  默认 local-first runtime 见该子目录
- 与 [memory-types/README.md](memory-types/README.md)
  payload taxonomy 见该子目录

## 规范结论

- durable memory 是 `Session` 域内独立于 transcript 和 short-term memory 的 durable knowledge plane
- durable memory 至少应区分 taxonomy、entrypoint/index、recall、write/consolidation、scope 五层
- `AutoMemory` 是默认 local-first runtime，不是 durable memory 的唯一规范名
- `TeamMemory`、`AgentMemory` 是 overlay，不是新的 durable plane
