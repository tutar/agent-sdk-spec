# Auto Memory Prompt And Index

## 职责

`AutoMemoryPromptAndIndex` 定义 auto-memory operating instructions 与 resident entrypoint index 之间的关系。

它不定义：

- short-term continuity summary
- transcript restore
- instruction markdown loading precedence

## 核心结论

`AutoMemory` 不是“把整个 durable store 注入 prompt”。

推荐至少分两层：

- resident instructions
- resident entrypoint index

并与下列层语义分离：

- topic payloads
- header / manifest selection layer

## 稳定对象

```text
AutoMemoryIndex
  - entrypoint_ref
  - entry_limit
  - byte_limit
  - resident_instructions_ref?
```

## 必须保持的语义

### 1. resident instructions

auto-memory runtime 应向模型暴露 durable memory operating instructions。

这些 instructions 至少应覆盖：

- 什么值得保存
- 什么不值得保存
- 如何处理 explicit remember / forget
- 如何处理 ignore-memory
- recalled memory 可能 stale，应验证当前状态

### 2. resident index

resident index 应长期驻留在 context plane，但必须与 topic payload 分离。

resident index 常见用途：

- 列出有哪些 durable memory 主题存在
- 提供低成本入口
- 限制常驻 memory 的大小

### 3. index is not payload

`MEMORY.md`、manifest、header cache 或语义等价物都只能视为 index。

它们不应承载 durable memory 正文，也不应被当作 transcript。

## 与其它子规范的边界

- 与 [../durable-memory-recall-pipeline.md](../durable-memory-recall-pipeline.md)
  本页是 auto-memory 的默认 local mapping；上层 recall 语义仍以 `durable-memory-recall-pipeline.md` 为准
- 与 [../../../harness/context-engineering/instruction-markdown/README.md](../../../harness/context-engineering/instruction-markdown/README.md)
  `AGENTS.md` / `rules` 一类 instruction markdown loading 属于 harness-level context input，不属于 auto-memory
  resident index 属于 durable memory runtime，不等于显式 injection source
- 与 [memory-entrypoint-index.md](memory-entrypoint-index.md)
  本页定义 resident instructions + resident index 的组合语义；entrypoint 的细化约束见该页
- 与 [topic-memory.md](topic-memory.md)
  durable正文层见该页
- 与 [frontmatter-and-header-manifest.md](frontmatter-and-header-manifest.md)
  header scan、manifest 和 selection layer 的细化约束见该页

## Default Local Mapping

当前默认本地映射通常表现为：

- `MEMORY.md` 常驻
- topic payload 与 resident entrypoint 分层
- header / manifest 先于 payload read
