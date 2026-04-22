# Context Input Model

## 职责

`AgentRuntime` 在每轮 sampling 前会把来自 session、instruction、attachments、capability surface 和 per-turn dynamic context 的材料交给本页定义的输入对象模型。

关键结论：

- 最终模型输入不是一段 prompt 字符串
- tool schemas、attachments、request metadata 也属于模型可见上下文
- `system_context`、`user_context`、attachments、capability surface 不是同一种注入机制

它不负责：

- 装配顺序本身
- fragment provider 的收集方式
- attachment envelope 的排序与 scope 语义
- budget governance 和 context editing 触发

## 最小对象模型

```text
ContextAssemblyInput
  - transcript
  - bootstrap_prompt_sections
  - system_context[]
  - user_context[]
  - attachments[]
  - capability_surface
  - evidence_refs[]
  - request_metadata
```

```text
ContextPlane
  - system_prompt
  - message_stream
  - attachment_stream
  - capability_surface
  - request_metadata
```

```text
StructuredContext
  - scope: system | user
  - lifecycle: startup | per_turn | resume
  - payload
  - provenance
```

```text
CapabilityExposure
  - always_loaded[]
  - deferred[]
  - searchable[]
```

```text
EvidenceRef
  - kind
  - ref
  - preview?
  - provenance
```

## 语义要求

### 1. `system_prompt` 与 `message_stream` 必须区分

- `system_prompt`
  表达稳定角色、行为骨架、环境约束与系统级规则。
- `message_stream`
  表达真实 transcript、meta user context、tool-use continuity 与运行时消息。

实现不应把二者提前混成同一个字符串。

### 2. structured context 必须按 scope 分离

- `system_context`
  更接近 environment snapshot、runtime guardrail、cache breaker 一类上下文
- `user_context`
  更接近规则文件、日期、面向回答阶段的系统提醒

即使二者最后都进入模型输入，也不应共享同一个注入面。

### 3. attachments 是一等上下文输入

attachment 不只是“额外文件”。

它可以承载：

- 用户显式附加的文件/资源
- nested memory / relevant memories
- diagnostics / IDE selection
- delta announcements
- task / budget / mailbox 一类运行时投影

因此 attachment 必须被建模为独立 context plane，而不是 `user_context` 的一种别名。

attachment assembled 之后的 envelope、ordering、scope 与 audience 语义，见 [attachment-assembly.md](attachment-assembly.md)。

### 4. capability surface 属于上下文的一部分

模型可见的 tool schemas、deferred tool search surface、searchable capability listing 都会改变模型行为。

因此：

- capability exposure 必须进入 `ContextAssemblyInput`
- 它既影响工具可用性，也影响 token budget 与 cache stability

### 5. evidence 应优先以 ref 保留

长工具输出、task output、资源读取结果若需要被后续可靠引用，不应只保留 summary。

推荐语义：

- 当前轮只内联 preview
- 完整结果外部化
- 通过 `EvidenceRef` 在后续轮次中重新引用

`EvidenceRef` 也是 `ContextEditing` 中 content externalization 的稳定结果之一。

## 与相邻页面的边界

- 与 [context-assembly-pipeline.md](context-assembly-pipeline.md)
  本页定义输入对象；装配顺序与阶段边界见 `ContextAssemblyPipeline`
- 与 [context-provider.md](context-provider.md)
  provider 只产出 fragments；本页定义这些 fragments 最终进入的 object model
- 与 [attachment-assembly.md](attachment-assembly.md)
  本页只把 attachments 视为一类 context plane；assembled attachment 的 envelope 语义见 `AttachmentAssembly`
- 与 [../governance/context-governance.md](../governance/context-governance.md)
  budget、compact 和 editing 决策不在本页定义

## 规范结论

- 模型可见上下文必须按多平面对象建模
- `system_context`、`user_context`、attachments、capability surface 必须区分
- tool schemas 与 request metadata 也属于上下文语义的一部分
- evidence ref 必须保持 transport-neutral
