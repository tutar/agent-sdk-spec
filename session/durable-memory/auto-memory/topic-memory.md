# Auto Memory Topic Memory

## 职责

`AutoMemoryTopicMemory` 定义 auto-memory runtime 下 durable memory 的正文层。

它回答的是：

- 什么是 auto-memory 的 durable payload unit
- topic payload 与 resident index、header manifest 的边界是什么
- consolidation、team scope、agent scope 如何复用这层 payload 形态

它不定义：

- short-term session memory
- transcript restore
- resident index 的常驻预算
- header scan / manifest 选择策略

## 核心结论

在 auto-memory 的默认 local mapping 中，长期记忆正文应被建模为一组按主题组织的 durable payload units，而不是：

- transcript 片段
- 时间顺序日志
- resident index 本身

推荐将这层建模为：

- per-topic durable record
- semantic organization, not chronological organization
- update-existing / avoid-duplicates
- consolidation 的主要整理目标

## 稳定对象

```text
TopicMemoryRecord
  - memory_ref
  - type?
  - description?
  - payload_ref
  - scope?
  - updated_at?
```

实现不要求公开同名对象，但这层语义应稳定存在。

## 必须保持的语义

### 1. topic payload 是 durable正文

topic memory 应承载 durable memory 的正文内容，而不是索引提示。

它应允许：

- 保存更完整的长期知识
- 承载超出 resident index 预算的细节
- 作为 dream / consolidation 的主要 merge target

### 2. semantic by topic, not chronological

topic memory 应按主题组织，而不是按时间顺序堆叠。

推荐表现为：

- 每个 durable topic 对应一个正文单元
- 新信号优先 merge 到现有 topic，而不是追加重复 topic
- 主题语义比“发生时间”更重要

### 3. update-existing / avoid-duplicates

当 durable knowledge 与已有 topic 语义重合时，runtime 应允许：

- 更新现有 topic memory
- merge duplicate or near-duplicate topics
- 删除被证伪或过时的 topic payload

不应把长期记忆建模成不可整合的 append-only payload 集。

### 4. topic payload 不等于 transcript

topic memory 可以从 transcript 提炼而来，但它本身不是 transcript。

因此：

- topic memory 不应通过改写 transcript 来模拟存在
- transcript restore 不依赖 topic payload 才能成立
- topic payload 被召回时，应进入 context plane，而不是伪装成会话原消息

### 5. scope 可变，但 payload 形态不应漂移

project-scoped、team-scoped、agent-scoped durable memory 可以共享 topic memory 这一 payload 形态。

scope 可以改变：

- 可见性
- 所有权
- 同步或共享方式

但不应改变 topic payload 作为 durable正文层的基本语义。

## 与其它子规范的边界

- 与 [memory-entrypoint-index.md](memory-entrypoint-index.md)
  `MEMORY.md` 或等价 entrypoint 负责 resident index；本页定义正文层
- 与 [frontmatter-and-header-manifest.md](frontmatter-and-header-manifest.md)
  header / manifest 负责 recall candidate projection；本页定义被选择后的 durable payload
- 与 [auto-memory-write-paths.md](auto-memory-write-paths.md)
  direct write、background extraction、dream consolidation 的默认 durable target 通常是 topic memory
- 与 [../scoped-durable-memory.md](../scoped-durable-memory.md)
  scope 是共享边界；本页只定义 payload unit 语义

## Default Local Mapping

当前默认本地映射通常表现为：

- markdown topic files 作为 durable payload
- 每个 payload 有主题级 frontmatter/header
- direct write / extraction / dream 优先更新已有 topic files，而不是制造重复正文
