# Instruction Markdown Loading

## 职责

`AgentRuntime` 在 startup、resume 或 target-sensitive turn 前调用 `InstructionMarkdownLoading`，发现、排序、适用并装配 file-backed instruction sources。

它回答的是：

- 哪些 instruction files 会被加载
- 它们按什么顺序累积
- subtree / target-path applicability 如何工作
- 它们进入哪个 context plane

## 核心结论

instruction markdown loading 是 harness-level context assembly input，不是 durable memory。

它的默认本地映射可使用：

- user-level `AGENTS.md`
- workdir/project-level `AGENTS.md`
- subtree-scoped `AGENTS.md`
- local override file

实现不要求使用同名文件，但必须保留：

- deterministic precedence
- cumulative loading
- later overrides earlier
- subtree isolation

## 稳定接口

推荐最小接口：

```text
InstructionMarkdownLoader
  - discover_sources(runtime_context) -> sources
  - resolve_applicable_sources(target_path?) -> ordered_sources
  - load_sources(sources) -> loaded_instruction_markdown
```

推荐最小对象：

```text
InstructionMarkdownSource
  - source_id
  - kind: agents_file | local_override | subtree_file
  - scope: user | project | subtree | local
  - file_path
  - applies_to?
  - precedence
```

```text
LoadedInstructionMarkdown
  - source_ref
  - content
  - priority
  - applicable_target?
```

## 默认顺序语义

宿主若声明支持 `AGENTS.md` instruction loading，推荐按以下顺序发现并加载：

1. user-level `AGENTS.md`
2. workdir / project-level `AGENTS.md`
3. 从 workdir 到目标子目录路径上的各级 `AGENTS.md`
4. local override file

约束：

- 采用累积加载
- 后加载者优先级高于前者
- subtree file 仅影响对应子树
- sibling 子树不得污染当前 target

## 与 Context Plane 的关系

instruction markdown source 进入的是 context plane，而不是 transcript plane。

因此必须保持：

- 不伪装成 assistant / user transcript message
- compact / resume 后可重新解析源文件或复用已解析缓存
- 不通过 durable recall store 来实现加载成功

## 与其它子规范的边界

- 与 [../assembly/context-assembly-pipeline.md](../assembly/context-assembly-pipeline.md)
  本页定义 instruction source loading；assembly pipeline 定义它如何进入模型输入
- 与 [instruction-include-expansion.md](instruction-include-expansion.md)
  include expansion 是 loading 的子机制
- 与 [conditional-instruction-rules.md](conditional-instruction-rules.md)
  conditional rules 定义 `paths` / target-path matching
- 与 [../../../durable-memory/README.md](../../../durable-memory/README.md)
  durable memory 是 store-backed durable knowledge；instruction markdown loading 不属于 durable memory
