# Frontmatter And Header Manifest

## 职责

`FrontmatterAndHeaderManifest` 定义 header/frontmatter 与 recall manifest 的语义。

它负责：

- header scan 先于 payload read 的分层
- recall candidate metadata 的最小语义
- manifest 作为 bounded selection 输入的作用

它不负责：

- durable payload 本体
- resident entrypoint 的常驻预算
- selection algorithm 的具体实现

## 核心结论

durable memory 不应通过预读全部 payload 来做 recall。

推荐分层为：

- resident entrypoint index
- bounded header/frontmatter scan
- manifest-based relevance selection
- selected payload read

header / frontmatter 是 recall infrastructure，而不是 durable payload 本体。

## 必须保持的语义

### 1. header scan precedes payload read

runtime 应允许先读取 bounded 的 header/frontmatter，而不是直接读取所有 payload。

### 2. manifest drives bounded selection

header/frontmatter 应能被投影成 manifest 或语义等价结构，供 relevance selection 使用。

### 3. metadata is not full truth

header/manifest 只负责 recall candidate projection，不应替代 durable payload。

### 4. freshness should be representable

header 层应允许建模 freshness，例如：

- mtime
- updated_at
- stale hint

## 与其它页面的边界

- 与 [topic-memory.md](topic-memory.md)
  本页定义 candidate metadata layer；topic memory 定义 durable payload layer
- 与 [memory-md.md](memory-md.md)
  entrypoint index 是 resident orientation layer；header manifest 是 recall selection layer
- 与 [../operations/memory-scan.md](../operations/memory-scan.md)
  本页定义 storage metadata；scan 定义 runtime 如何读取它
- 与 [../operations/relevant-memory-selection.md](../operations/relevant-memory-selection.md)
  selection 消费 manifest，但 manifest 不是 selection algorithm 本身

## 规范结论

- header/frontmatter 必须先于 payload read
- manifest 只负责 candidate projection
- header metadata 必须和 payload truth source 分离
