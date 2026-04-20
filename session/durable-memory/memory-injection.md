# Memory Injection

## 职责

`Memory Injection` 负责把长期有效、显式表达的 durable context source 注入当前 runtime。

它不同于：

- `session transcript`
  可恢复事件历史
- `short-term session memory`
  服务 continuity 的会话级摘要
- `durable memory recall`
  从长期记忆库中按 query 取回候选
- `memory consolidation`
  从 session 或跨 session 经验中提炼和整合 durable memory

它的核心目标是：

- 让显式记忆源进入当前 turn 的 context plane
- 保持这些记忆源与 transcript / restore / recall 的边界
- 为 file-backed durable memory injection 提供标准 discovery / ordering / applicability 语义

## 稳定接口

推荐最小接口：

```text
DurableMemoryInjectionLoader
  - discover_file_backed_sources(runtime_context) -> memory_sources
  - resolve_applicable_sources(target_path) -> ordered_memory_sources
  - load_injection_payloads(sources) -> memory_payloads
```

推荐标准对象：

```text
DurableMemoryInjectionSource
  - source_id
  - kind: agents_file
  - scope: user | project | subtree
  - file_path
  - applies_to
  - precedence
```

```text
LoadedMemoryInjection
  - source_ref
  - content
  - priority
  - applicable_target
```

这些对象的 canonical fields 见 [../../object-model.md](../../object-model.md)。

## File-Backed Durable Memory Injection

`FileBackedDurableMemoryInjection` 是 `DurableMemoryInjection` 的标准子形态。

它用于把显式记忆文件稳定注入当前 turn，但不把这些文件误建模为 transcript 或 recall record。

第一批标准载体包括：

- `AGENTS.md`

`AGENTS.md` 的默认语义：

- 是持久化记忆文件
- 属于 durable memory injection
- 不属于 session transcript
- 不自动进入 durable memory recall store
- 不承担 session restore 真源职责

## `AGENTS.md` Loading Order

当宿主声明支持 `AGENTS.md` memory injection 时，推荐按以下顺序发现并加载：

1. `~/.openagent/AGENTS.md`
2. 工作目录下的 `AGENTS.md`
3. 从工作目录到目标子目录路径上的各级 `AGENTS.md`

这里的“目标子目录”指当前 turn 主要工作目标所在的目录或语义等价目标路径。

约束：

- 按上述顺序依次加载
- 最终注入结果采用累积加载
- 若内容存在冲突，后加载者优先级高于前者
- 子目录 `AGENTS.md` 仅对其子树生效，不应影响兄弟目录
- 默认不要求扫描到文件系统根，也不要求支持额外规则目录

## 与 Recall / Transcript 的边界

`AGENTS.md` 进入当前 runtime 时，必须保持以下边界：

- 它进入 context injection plane，而不是 transcript plane
- 它不应伪装成 assistant / user message 写入 event log
- 它不应先被抽取成 `DurableMemoryRecord` 再参与本轮 recall，除非宿主额外声明这一扩展能力
- compact / resume 后，宿主可以重新解析源文件，或使用语义等价的已解析缓存，但不能从 transcript 反推这些文件内容

## 与其他 memory 子能力的边界

- 与 [durable-memory-model.md](durable-memory-model.md)
  `durable-memory-model` 定义 durable 侧的总模型；本文件定义 injection 这一层的具体行为
- 与 [durable-memory-recall.md](durable-memory-recall.md)
  durable recall 面向长期记忆 store；本文件面向显式 file-backed injection source
- 与 [scoped-durable-memory.md](scoped-durable-memory.md)
  scope 负责 durable memory 的共享/可见性边界；injection 负责显式 source 注入
- 与 [memory-consolidation.md](memory-consolidation.md)
  consolidation 负责提炼/整合 durable memory；injection 负责显式注入 durable context source
- 与 [../README.md](../README.md)
  `Session` 只声明 file-backed memory injection 属于 `Session.memory`，不在 transcript 内部建模

## 要解决的问题

- 如何把显式记忆文件稳定注入当前 turn
- 如何保持 source precedence deterministic
- 如何让 subtree-scoped memory 不污染 sibling 目录
- 如何防止实现把 injection 混成 transcript 或 recall
- 如何在 future injection source 增加时保持规范可扩展

## 规范结论

- `Memory Injection` 应作为 `Session.memory` 下的独立子规范存在
- `AGENTS.md` 应作为第一批标准 file-backed durable memory injection source
- injection source 的 ordering / applicability / override 语义必须 deterministic
- injection 进入 context plane，不进入 transcript plane
