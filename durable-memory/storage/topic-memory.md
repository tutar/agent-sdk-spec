# Topic Memory

## 职责

`TopicMemory` 定义 durable memory 的正文层。

它回答的是：

- 什么是 durable payload unit
- topic payload 与 resident index、header manifest 的边界是什么
- consolidation、team scope、agent scope 如何复用这层 payload 形态

## 核心结论

长期记忆正文应被建模为一组按主题组织的 durable payload units，而不是：

- transcript 片段
- 时间顺序日志
- resident index 本身

推荐将这层建模为：

- per-topic durable record
- semantic organization, not chronological organization
- update-existing / avoid-duplicates
- dream 的主要 merge target

## 必须保持的语义

### 1. topic payload 是 durable正文

topic memory 应承载 durable memory 的正文内容，而不是索引提示。

### 2. semantic by topic, not chronological

topic memory 应按主题组织，而不是按时间顺序堆叠。

### 3. update-existing / avoid-duplicates

当 durable knowledge 与已有 topic 语义重合时，应允许：

- 更新现有 topic memory
- merge duplicate or near-duplicate topics
- 删除被证伪或过时的 topic payload

### 4. topic payload 不等于 transcript

topic memory 可以从 transcript 提炼而来，但它本身不是 transcript。

### 5. scope 可变，但 payload 形态不应漂移

project、team、agent 等 overlays 可以共享 topic memory 这一 payload 形态。

## 与其它页面的边界

- 与 [memory-md.md](memory-md.md)
  resident index 负责常驻入口；本页定义正文层
- 与 [frontmatter-and-header-manifest.md](frontmatter-and-header-manifest.md)
  header / manifest 负责 candidate projection；本页定义被选择后的 durable payload
- 与 [../operations/direct-memory-write.md](../operations/direct-memory-write.md)
  direct write 的默认 durable target 通常是 topic memory
- 与 [../operations/extract-memories.md](../operations/extract-memories.md)
  extract 的默认 payload target 通常是 topic memory
- 与 [../durable-memory-scopes-and-overlays.md](../durable-memory-scopes-and-overlays.md)
  scope 是共享边界；本页只定义 payload unit 语义

## 规范结论

- topic memory 是 typed memory 的主要 durable payload 载体
- topic memory 必须与 resident index 和 header manifest 分层
- topic memory 应优先 update-existing，而不是 append-only duplicate payload
