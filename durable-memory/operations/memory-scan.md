# Memory Scan

## 职责

`MemoryScan` 定义 runtime 在 recall 前如何扫描 durable store 的 candidate metadata。

它负责：

- bounded header/frontmatter scan
- manifest candidate production
- scan limit 和 freshness metadata 暴露

它不负责：

- 最终 relevance selection
- payload 全量读取
- durable write 或 consolidation

## 核心结论

scan 是 read-side 的第一步。

runtime 不应为 recall 直接读取全部 payload，而应先读取 bounded metadata，例如：

- header
- frontmatter
- title / description / type
- mtime / updated_at

## 最小对象

```text
MemoryScanRequest
  - scope_selector?
  - scan_limit
  - freshness_policy?
```

```text
MemoryScanResult
  - manifest_entries[]
  - truncated
  - scanned_at?
```

## 与其它页面的边界

- 与 [relevant-memory-selection.md](relevant-memory-selection.md)
  scan 只生成 candidate manifest；selection 决定哪些 memory 进入当前 turn
- 与 [../storage/frontmatter-and-header-manifest.md](../storage/frontmatter-and-header-manifest.md)
  storage 页定义 manifest metadata；本页定义 runtime 如何消费它
- 与 [../storage/topic-memory.md](../storage/topic-memory.md)
  topic payload 不应在 scan 阶段被无界读取

## 规范结论

- scan 必须先于 payload read
- scan 必须 bounded
- manifest metadata 必须和 payload truth source 分离
