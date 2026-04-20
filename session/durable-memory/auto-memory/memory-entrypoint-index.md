# Auto Memory Entrypoint Index

## 职责

`AutoMemoryEntrypointIndex` 定义 auto-memory runtime 中 resident entrypoint index 的语义。

典型默认映射是 `MEMORY.md`，但本页描述的是语义，不是具体文件名约束。

它负责：

- resident index 的职责与预算
- index 与 payload 的分层
- distilled index 在 assistant-style mode 下的语义

它不负责：

- topic payload 本体
- header manifest 的候选选择
- transcript restore

## 核心结论

resident entrypoint index 应被视为 durable memory 的常驻入口层，而不是 durable正文。

它至少应满足：

- 常驻进入 context plane
- 低成本、可预算
- 列出 durable topic 的 pointer / hook / summary
- 不承载完整 durable payload

## 稳定对象

```text
AutoMemoryEntrypointIndex
  - entrypoint_ref
  - resident
  - entry_budget
  - entry_shape
  - distilled?
```

实现不要求公开同名字段，但这层对象语义应存在。

## 必须保持的语义

### 1. index is not payload

resident entrypoint index 必须与 durable正文层分离。

它可以包含：

- pointers
- one-line hooks
- concise titles / summaries

但不应承载：

- 完整 durable payload
- transcript fragments
- 无界增长的正文内容

### 2. resident and budgeted

resident index 进入 context plane 时，应保持低成本和预算可控。

实现至少应允许对以下之一设预算：

- line count
- bytes
- token-equivalent size

预算约束不是实现细节，而是 resident layer 与 payload layer 分离的重要前提。

### 3. compact entry shape

resident index 的 entry 应偏向 compact pointer / hook，而不是正文段落。

实现不要求固定 markdown 形态，但应保持：

- entry 可作为 durable topic 的低成本入口
- entry 结构稳定到足以支持 orientation
- 细节应下沉到 topic payload

### 4. distilled index is allowed

某些 long-lived host profile 可把 resident index 变成 delayed / distilled index：

- 新信号先进入 append-only capture stream
- resident index 稍后由 consolidation 更新

这改变的是 operating mode，不改变 resident index 的 ownership 和语义角色。

## 与其它子规范的边界

- 与 [auto-memory-prompt-and-index.md](auto-memory-prompt-and-index.md)
  本页聚焦 resident entrypoint index；上层页继续定义 resident instructions + index 的组合语义
- 与 [topic-memory.md](topic-memory.md)
  topic memory 是 durable正文层；本页是常驻入口层
- 与 [frontmatter-and-header-manifest.md](frontmatter-and-header-manifest.md)
  header manifest 是 recall candidate layer；本页是 resident orientation layer
- 与 [auto-memory-write-paths.md](auto-memory-write-paths.md)
  direct write、extraction、dream 可以更新 index，但不改变其 resident-entrypoint 角色

## Default Local Mapping

当前默认本地映射通常表现为：

- `MEMORY.md` 作为 resident entrypoint index
- 每条 entry 是 pointer / one-line hook
- line / byte budget 控制常驻成本
- assistant-style mode 下可由 background consolidation 维护该 index
