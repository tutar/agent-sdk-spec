# Auto Memory Frontmatter And Header Manifest

## 职责

`AutoMemoryFrontmatterAndHeaderManifest` 定义 auto-memory runtime 下 header/frontmatter 与 manifest selection 的语义。

它负责：

- header scan 先于 payload read 的分层
- recall candidate metadata 的最小语义
- manifest 作为 bounded selection 输入的作用

它不负责：

- durable payload 本体
- resident entrypoint index 的常驻预算
- selection algorithm 的具体实现

## 核心结论

auto-memory 不应通过预读全部 durable payload 来做 recall。

推荐分层为：

- resident entrypoint index
- bounded header/frontmatter scan
- manifest-based relevance selection
- selected payload read

header / frontmatter 是 recall infrastructure，而不是 durable正文本体。

## 稳定对象

```text
AutoMemoryHeader
  - memory_ref
  - title?
  - type?
  - description?
  - mtime?
```

```text
AutoMemoryManifest
  - headers[]
  - scan_limit
  - selection_budget
```

## 必须保持的语义

### 1. header scan precedes payload read

在需要 recall 时，runtime 应允许先读取 bounded 的 header/frontmatter，而不是直接读取所有 durable payload。

这层至少应支持：

- 识别 durable payload unit
- 提供简短 semantic hint
- 提供 freshness / mtime 或语义等价信号

### 2. manifest drives bounded selection

header/frontmatter 应能被投影成 manifest 或语义等价结构，供 relevance selection 使用。

manifest selection 至少应允许：

- bounded candidate count
- already surfaced dedupe
- freshness-aware selection
- recent-tools / context-aware suppression 或语义等价能力

### 3. metadata is not full truth

header/manifest 只负责 recall candidate projection，不应替代 durable正文。

因此：

- description 不是完整 payload
- type 不是全部语义
- manifest 不应被当作 transcript 或 durable正文读取层

### 4. freshness should be representable

header 层应允许建模 freshness，例如：

- mtime
- updated_at
- stale hint

规范不强制具体字段名，但不应把 recall candidate 当作永远 current 的事实。

## 与其它子规范的边界

- 与 [topic-memory.md](topic-memory.md)
  本页定义 candidate metadata layer；topic memory 定义 durable正文层
- 与 [memory-entrypoint-index.md](memory-entrypoint-index.md)
  entrypoint index 是 resident orientation layer；header manifest 是 recall selection layer
- 与 [../durable-memory-recall.md](../durable-memory-recall.md)
  本页是 auto-memory 默认本地映射；上层 recall 分层语义仍以 durable-memory-recall 为准

## Default Local Mapping

当前默认本地映射通常表现为：

- topic payload 通过 frontmatter/header 暴露 `type`、`description`、`mtime` 等 metadata
- runtime 先做 bounded header scan
- 再把 headers 压成 manifest 交给 selection
- 被选中的少量 payload 才读取正文并进入 context plane
