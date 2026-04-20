# Durable Memory Recall

## 职责

`DurableMemoryRecall` 定义 `Session.memory` 如何从长期记忆中选择、读取并注入当前 turn 所需的 durable memory。

它负责：

- 定义长期记忆的常驻索引与按需召回边界
- 定义 recall 的 bounded selection 语义
- 定义 header/index、recalled payload、durable store 三层关系
- 定义 freshness、dedupe、already-surfaced 一类 recall 控制语义

它不负责：

- short-term session memory
- transcript restore
- memory consolidation
- `AGENTS.md` 这类显式 injection source 的发现规则

## 核心结论

长期记忆不应被实现为“把整个 durable memory store 直接塞进每轮上下文”。

推荐至少分三层建模：

- `memory index`
  - 常驻、低成本、只提供目录/指针/摘要
- `recall candidates`
  - 当前轮可被选中的 durable memory headers
- `recalled payloads`
  - 真正进入当前 turn 的长期记忆内容

这三层必须语义分离。

## 稳定对象

推荐最小对象：

```text
DurableMemoryIndex
  - entrypoints[]
  - pointers[]
  - updated_at?
```

```text
DurableMemoryHeader
  - memory_ref
  - title?
  - description?
  - type?
  - scope?
  - mtime?
```

```text
DurableMemoryRecallRequest
  - session_ref
  - query
  - scope_selector?
  - already_surfaced_refs[]
  - recent_tools[]
  - max_results?
  - max_total_bytes?
```

```text
DurableMemoryRecallResult
  - selected_refs[]
  - selected_headers[]
  - selected_payloads[]
  - skipped_refs[]
```

实现不要求公开同名字段，但这几层语义应稳定存在。

## 推荐流程

### 1. 常驻索引

长期记忆系统应允许一个低成本常驻索引进入上下文。

典型用途：

- 告诉模型有哪些 durable memory 主题存在
- 提供 memory system 使用规则
- 避免每轮都重新加载全部正文

常驻索引不等于全部长期记忆正文。

### 2. Header Scan

在需要 recall 时，runtime 应先读取 bounded 的 header / manifest，而不是直接读取所有正文。

header 层至少应支持：

- 标识文件或 memory record
- 给出简短说明
- 提供 freshness / mtime 或语义等价信息

### 3. Relevance Selection

runtime 应根据：

- query
- runtime context
- recent tools
- already surfaced refs

来选择少量 recall candidates。

选择器可以是规则、检索器或模型辅助路径，但必须满足：

- bounded
- 可去重
- 不无界展开 durable store

### 4. Payload Read

只有被选中的 memories 才应读取正文并注入当前 turn。

这一步必须允许：

- 数量上限
- 总大小上限
- scope 过滤
- stale memory 提示

## 必须保持的语义

### 1. bounded recall

长期记忆召回必须是 bounded 的。

实现至少应能对以下一项或多项设上界：

- 召回数量
- 总字节数 / token 预算
- 每种 scope 的最大配额

### 2. recalled memory 不等于 transcript

durable memory 被召回后，应进入 context plane，而不是伪装成 transcript 原消息。

实现可以把 recalled memory 放入：

- structured context
- attachment-like context
- dedicated memory plane

但不得改写 transcript 来模拟 recall。

### 3. index 与 payload 分离

实现必须允许：

- index 常驻
- payload 按需读取

这两层不应混成“永远加载全部长期记忆”。

### 4. freshness / staleness 可建模

durable memory 是长期知识，不保证永远新鲜。

因此 recall 路径应允许建模：

- mtime
- freshness note
- stale warning
- revalidation hint

规范不强制具体文案，但不应把 recalled memory 视为当前真相。

### 5. already surfaced / dedupe

若同一条 durable memory 已在近几轮或本轮其它路径中 surfacing 过，runtime 应允许：

- 跳过再次召回
- 降低优先级
- 仅保留 ref 而不重复注入正文

## 与其它子规范的边界

### 与 [durable-memory-model.md](durable-memory-model.md)

- `durable-memory-model`
  定义 durable memory 的总模型
- 本页
  只定义 durable long-term memory 的 recall 语义

### 与 [scoped-durable-memory.md](scoped-durable-memory.md)

- `scope`
  定义长期记忆的可见性和共享边界
- 本页
  定义 recall 如何在这些 scope 内选择和读取

### 与 [memory-injection.md](memory-injection.md)

- file-backed injection source（如 `AGENTS.md`）
  属于显式 durable context injection
- durable memory recall
  属于长期记忆 store 的按需召回

二者不应混成同一机制。

### 与 [memory-consolidation.md](memory-consolidation.md)

- consolidation
  负责提炼、合并、整理 durable memory
- recall
  负责把 durable memory 注入当前 turn

consolidation 可以更新 recall 的输入，但不替代 recall 本身。

## Default Local Mapping

本地实现中，常见映射包括：

- `MEMORY.md` 或等价入口文件作为常驻索引
- markdown topic files 作为 durable payload
- 基于 header manifest 的相关记忆选择
- bounded 的每轮 recall 数量与大小限制

这只是默认映射，不构成对具体文件布局的强约束。

auto-memory 作为默认 local mapping 的更完整语义，见 [auto-memory/README.md](auto-memory/README.md)。

其中 resident entrypoint、topic payload、header manifest 的默认本地映射，见：

- [auto-memory/memory-entrypoint-index.md](auto-memory/memory-entrypoint-index.md)
- [auto-memory/topic-memory.md](auto-memory/topic-memory.md)
- [auto-memory/frontmatter-and-header-manifest.md](auto-memory/frontmatter-and-header-manifest.md)

## 规范结论

- durable memory recall 必须和 short-term session memory 分层
- 常驻索引与正文按需召回必须语义分离
- recall 必须 bounded、可去重、可建模 freshness
- recalled durable memory 进入 context plane，而不是 transcript
