# Bootstrap Prompts

## 职责

`AgentRuntime` 在进入主 loop 前调用 `BootstrapPrompts`，建立稳定的 `system prompt skeleton`。

它负责：

- 为 runtime 提供稳定的行为骨架
- 提供模型可见的身份、能力和环境说明
- 以 section 方式组织 system prompt
- 为 prompt caching 提供 static / dynamic boundary
- 为 override / agent / custom / append 等调用提供稳定合成规则

它不负责：

- startup-only lifecycle context
- attachment assembly
- tool result 注入
- memory recall 结果本身
- per-turn governance 与 editing

## 稳定接口

推荐最小接口：

```text
BootstrapPromptAssembler
  - build_default_prompt(runtime_capabilities, model_view) -> PromptSections
  - merge_prompt_layers(base, overrides) -> PromptSections
  - resolve_sections(sections, runtime_state) -> ResolvedPromptSections
  - split_static_dynamic(sections) -> PromptBlocks
  - invalidate_sections(reason) -> void
```

推荐最小对象：

```text
PromptSection
  - name
  - text | compute()
  - cache_policy
  - cache_breaking
  - dynamic
```

```text
PromptBlocks
  - attribution_prefix?
  - static_blocks[]
  - dynamic_blocks[]
```

## 规范要求

### 1. bootstrap prompt 必须是分段对象

实现不应将 bootstrap prompt 直接建模成单一字符串。

### 2. 必须支持覆盖与追加语义

至少应支持这些语义层：

- override prompt
- coordinator / orchestrator prompt
- agent prompt
- custom prompt
- default prompt
- append prompt

实现必须明确：

- 哪些层是替换
- 哪些层是追加
- 哪些层能与 default 共存

### 3. 必须与 structured context 分离

bootstrap prompt 不是 `system_context` 或 `user_context` 的别名。

推荐分层：

- bootstrap prompt
  system prompt 骨架
- structured context
  结构化 system/user/session context
- attachment context
  以 message 方式进入模型的动态上下文

### 4. 必须支持 section 级缓存与失效

实现应允许每个 prompt section 单独声明：

- 可缓存
- 不可缓存
- 何时失效
- 是否会破坏 prompt cache

### 5. 必须支持 static / dynamic boundary

bootstrap prompt 应支持将：

- 稳定前缀
- 会话动态部分

显式分开。

这个边界不只是优化项，而是 prompt caching 和 API projection 的稳定输入。

### 6. 动态 section 必须显式声明

下列内容通常属于动态 section：

- session guidance
- memory mechanics prompt
- environment info
- output style
- server / integration instructions
- token budget instructions

实现不应把这些动态段偷偷混入静态前缀中。

### 7. 必须区分 agent prompt 与 startup-only briefing

`agent prompt` 是 bootstrap prompt 内的稳定语义层，用来表达：

- identity / role
- operating mode
- domain-specific behavior contract
- 与 default prompt 的覆盖或叠加关系

首轮 kickoff、session-start、agent-start、resume-start 这类内容应走独立 startup context，见 [startup-and-turn-zero-context.md](startup-and-turn-zero-context.md)。

## 推荐默认策略

- 默认 prompt 由静态 section 和动态 section 两组组成
- 静态 section 尽量稳定，优先进入 cacheable prefix
- 动态 section 延后解析，并支持独立失效
- append prompt 永远在最终尾部
- custom prompt 应是完整替换，除非调用方明确要求 layered merge
- agent prompt 视为稳定 prompt layer；首轮额外引导默认走 startup context
- environment summary 可在 bootstrap prompt 中投影为 section，但其归属应先由 structured context 层定义

## 与相邻页面的边界

- 与 [../assembly/context-input-model.md](../assembly/context-input-model.md)
  本页只定义 stable system skeleton；完整输入对象模型见 `ContextInputModel`
- 与 [../assembly/context-provider.md](../assembly/context-provider.md)
  `BootstrapPrompts` 定义 skeleton；`ContextProvider` 负责向其它 context planes 提供结构化片段
- 与 [../assembly/context-assembly-pipeline.md](../assembly/context-assembly-pipeline.md)
  本页不定义完整装配顺序；per-turn assembly 顺序见 `ContextAssemblyPipeline`
- 与 [../governance/prompt-cache-strategy.md](../governance/prompt-cache-strategy.md)
  本页定义 stable skeleton 的结构；`PromptCacheStrategy` 定义这些结构如何组织成 cache-friendly 输入
- 与 [../governance/context-governance.md](../governance/context-governance.md)
  本页不负责预算与 compact 决策；这些治理职责属于 `ContextGovernance`
- 与 [../../model-provider/model-provider-adapter.md](../../model-provider/model-provider-adapter.md)
  `ModelProviderAdapter` 决定如何把 prompt blocks 投影到特定模型协议；不定义 bootstrap prompt 内容

## 规范结论

- runtime 应在进入主 loop 前建立 bootstrap prompt
- bootstrap prompt 应以 section 为基本单元，而不是单一字符串
- bootstrap prompt 应支持优先级覆盖、section 级缓存与 static/dynamic 边界
- bootstrap prompt 应与 startup-only context、structured context、attachments 和 governance 分开建模
