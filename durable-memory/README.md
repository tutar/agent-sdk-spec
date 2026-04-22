# Durable Memory

## 职责

`DurableMemory` 是 `agent-spec/` 下的 shared spec package。

它定义跨 turn、跨 session 的 durable knowledge 如何被：

- 存储
- 分类
- 扫描与选择
- 写入与整理
- 绑定到不同 overlay

它不是：

- 第六个核心模块
- transcript restore store
- short-term continuity summary
- harness-level instruction loading

## 所有权边界

`DurableMemory` 的所有权必须分开表达：

- `Session`
  提供 write-side 上游之一，尤其是 transcript 中的 model-visible conversation
- `Harness.AgentRuntime`
  持有 read-side orchestration，把相关 durable knowledge 注入当前 turn context
- `DurableMemory`
  自身定义 durable store、taxonomy、operations 与 overlays

因此：

- durable memory 不再作为 `Session` 的内嵌子目录建模
- 但它仍与 `Session` 和 `Harness` 都有稳定接口边界

## 核心结论

durable memory 不应被建模成单一文件、单一路径或单一算法。

推荐至少分四组：

- [storage/README.md](storage/README.md)
  resident index、topic payload、daily logs、header manifest
- [memory-types/README.md](memory-types/README.md)
  durable payload taxonomy
- [operations/README.md](operations/README.md)
  scan、selection、direct write、extract、dream consolidation
- [scopes/README.md](scopes/README.md)
  user / project / team / agent / local overlays

根级总览页固定为：

- [durable-memory-overview.md](durable-memory-overview.md)
- [auto-memory-runtime.md](auto-memory-runtime.md)
- [durable-memory-scopes-and-overlays.md](durable-memory-scopes-and-overlays.md)

## Read Side

read-side 的稳定语义是：

- durable memory 最终进入 current turn context
- 读侧由 `Harness.AgentRuntime` 调用
- 读侧不是由 `Session` 自己执行

细节见：

- [operations/memory-scan.md](operations/memory-scan.md)
- [operations/relevant-memory-selection.md](operations/relevant-memory-selection.md)

## Write Side

write-side 的稳定语义是：

- durable memory 不是 transcript 副本
- 写侧的主上游不是整个 session 对象
- 而是 session transcript 中的 model-visible conversation

这些 conversation signals 通过三条正式路径沉淀为 durable knowledge：

- [operations/direct-memory-write.md](operations/direct-memory-write.md)
- [operations/extract-memories.md](operations/extract-memories.md)
- [operations/auto-dream.md](operations/auto-dream.md)

## 规范结论

- `DurableMemory` 是 shared spec package，不是第六核心模块
- `Session` 不拥有 durable store，只参与 linkage 与 write-side 上游
- `Harness.AgentRuntime` 持有 read-side orchestration
- durable memory 必须明确区分 storage、taxonomy、operations 和 overlays
