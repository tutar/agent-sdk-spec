# Relevant Memory Selection

## 职责

`RelevantMemorySelection` 定义 `Harness.AgentRuntime` 如何从 durable memory 中选择、读取并注入当前 turn 所需的 knowledge。

它负责：

- 定义长期记忆的常驻索引与按需召回边界
- 定义 recall 的 bounded selection 语义
- 定义 header/index、recalled payload、durable store 三层关系
- 定义 freshness、dedupe、already-surfaced 一类 recall 控制语义

它不负责：

- short-term session memory
- transcript restore
- memory consolidation
- `AGENTS.md` / `rules` 一类 instruction markdown source 的发现规则

## 核心结论

长期记忆不应被实现为“把整个 durable memory store 直接塞进每轮上下文”。

推荐至少分三层建模：

- `resident entrypoint / index`
  常驻、低成本、只提供目录/指针/摘要
- `recall manifest / headers`
  当前轮可被选中的 durable memory headers
- `recalled payloads`
  真正进入当前 turn 的长期记忆内容

这三层必须语义分离。

## 稳定对象

```text
DurableMemoryEntrypointIndex
  - entrypoints[]
  - pointers[]
  - updated_at?
```

```text
DurableMemoryManifestEntry
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

## 推荐流程

1. resident entrypoint / index 常驻进入 context plane
2. `memory-scan` 读取 bounded header / manifest
3. runtime 基于 query、runtime context、recent tools、already surfaced refs 选择少量 candidates
4. 只有被选中的 memories 才读取正文并注入当前 turn

## 必须保持的语义

### 1. bounded recall

长期记忆召回必须是 bounded 的。

### 2. recalled memory 不等于 transcript

durable memory 被召回后，应进入 context plane，而不是伪装成 transcript 原消息。

### 3. index / manifest / payload 分离

实现必须允许：

- index 常驻
- manifest/header bounded scan
- payload 按需读取

### 4. freshness / staleness 可建模

recall 路径应允许建模：

- mtime
- freshness note
- stale warning
- revalidation hint

### 5. already surfaced / dedupe

若同一条 durable memory 已在近几轮或本轮其它路径中 surfacing 过，runtime 应允许：

- 跳过再次召回
- 降低优先级
- 仅保留 ref 而不重复注入正文

## 与其它子规范的边界

- 与 [../durable-memory-overview.md](../durable-memory-overview.md)
  本页只定义 read-side recall 语义
- 与 [../durable-memory-scopes-and-overlays.md](../durable-memory-scopes-and-overlays.md)
  scope 改变可见性和配额，不改变 read-side 分层
- 与 [../../harness/context-engineering/instruction-markdown/README.md](../../harness/context-engineering/instruction-markdown/README.md)
  instruction markdown loading 属于 harness-level context input，不属于 durable recall
- 与 [extract-memories.md](extract-memories.md)
  extract 更新 durable store，但不替代 recall 本身
- 与 [memory-scan.md](memory-scan.md)
  scan 先生成 candidates；本页定义 selection 和 payload load
- 与 [../storage/memory-md.md](../storage/memory-md.md)
  resident entrypoint 是常驻入口
- 与 [../storage/topic-memory.md](../storage/topic-memory.md)
  topic payload 是被选中的 durable正文层
- 与 [../storage/frontmatter-and-header-manifest.md](../storage/frontmatter-and-header-manifest.md)
  manifest/header 是 recall candidate layer

## 规范结论

- durable memory recall 必须和 short-term memory、transcript、instruction markdown loading 分层
- resident entrypoint、manifest/header、payload 三层必须语义分离
- recall 必须 bounded、可去重、可建模 freshness
- recalled durable memory 进入 context plane，而不是 transcript
