# Context Editing

## 职责

`AgentRuntime` 在 governance 决定触发后，调用 `ContextEditing` 对模型可见上下文执行 externalization、projection 或 rewrite。

它回答的不是“上下文来自哪里”，而是：

- 上下文何时应该被 externalize、折叠、重写
- working view 和 durable transcript 如何区分
- 从轻量编辑到强重写应如何分层退化
- compact / overflow recovery 后需要保留哪些连续性语义

它不负责：

- durable transcript 本体
- tool 执行本身
- prompt cache provider 细节
- memory consolidation 本体
- budget 分析与 compact 触发决策

## 核心结论

`ContextEditing` 不是单一 compact。

推荐至少区分三类稳定语义：

- `ContentExternalization`
  - 把超长 payload 从 inline content 转成 preview + durable ref
- `WorkingViewProjection`
  - 在不改写 transcript 的前提下，为当前轮投影更短的模型可见 working view
- `CompactionRewrite`
  - 在必须强重写时，显式建立 compact boundary 或语义等价边界

实现可以用不同算法完成这些能力，但不应把它们混成一种行为。

## 推荐分层

### 1. Content Externalization

当工具结果、资源读取结果或诊断输出过长时，runtime 应优先：

- 外置完整 payload
- 在当前轮只保留 preview
- 通过 `EvidenceRef` 或语义等价引用保留可恢复关联

这一步是最低损的上下文编辑。

它不应改变：

- 原始 `tool_use_id` / result identity
- 后续 resume / branch restore 的追溯关系

### 2. Lightweight Editing

在 externalization 之后，runtime 可以再执行轻量级上下文收缩，例如：

- 历史裁剪
- request-local 微型 compact
- cache-aware compact

这类编辑的特点应是：

- 尽量不重写 transcript
- 尽量维持当前 turn 结构
- 主要作用于体积，而不是 durable 会话边界

### 3. Working-View Projection

runtime 应允许在不改写 transcript 的前提下，为模型投影一个更短的 working view。

这一层的核心要求是：

- durable transcript 仍保持原始历史或语义等价历史
- 当前轮模型看到的是 projection，而不是 transcript 原文
- projection state 必须可恢复或可重建

这类机制适合：

- 折叠旧历史跨度
- 用 summary 代表一个 archived span
- 在真实 overflow 前先做增量恢复

### 4. Compaction Rewrite

当前面的手段不足以恢复预算时，runtime 可以进入 transcript-visible rewrite。

此时应：

- 建立明确的 compact boundary
- 生成 post-compact working set
- 保留最小连续性状态
- 让 resume 后能识别 rewrite 边界

这一层是最高损的编辑形式，因此应晚于 externalization 和 working-view projection。

## 最小稳定语义

### 1. Progressive Degradation

实现不应只支持“上下文超限时一次性 full compact”。

推荐语义是：

- 先低损编辑
- 再 working-view projection
- 最后 transcript rewrite

这能减少不必要的 summary churn，也更利于 evidence fidelity。

### 2. Working View 不等于 Transcript

runtime 允许：

- transcript 保留较完整历史
- 模型只看到 projection 后的 working view

但实现必须保持二者可解释、可恢复，且不能让 projection 与 transcript 混成同一种 durable 记录。

### 3. Externalized Payload 必须保持稳定引用

若某个 payload 被 externalize，则：

- 当前轮应保留 preview 或语义等价摘要
- 后续轮应能通过 stable ref 找回同一结果
- compact / resume / branch restore 后不应要求重新生成原始结果

### 4. Rewrite 必须有显式边界

当 runtime 进入强重写模式时，应产生 compact boundary 或语义等价边界，用于：

- 标识 pre-edit / post-edit 工作集的切换
- 恢复 preserved segment
- 阻止旧 projection state 跨 boundary 污染新历史

### 5. Compact Summary 不是 Truth Source

无论是 lightweight editing 还是 full rewrite，summary 都只负责 continuity shaping。

summary 不应被当作：

- transcript 本体
- 原始证据的唯一来源
- persisted result 的替代物

## 恢复要求

`ContextEditing` 的结果必须进入恢复语义，而不是只在当前 turn 生效。

至少需要满足：

- externalization 的 stable ref 可在 resume 后重新引用
- working-view projection 的 state 可恢复或可重建
- compact boundary 可在恢复时被识别
- preserved segment 或语义等价 continuity window 可在 rewrite 后重新建立

实现可以使用：

- transcript metadata
- side store
- compact state snapshot
- lazy replay

但恢复效果必须稳定。

## 与其它子域的边界

### 与 Session 的边界

- `Session`
  - 保存 durable transcript、event log、checkpoint、resume 素材
- `ContextEditing`
  - 只塑形模型可见上下文

因此：

- transcript 完整性属于 `Session`
- working view、externalization、compact rewrite 语义属于 `Harness`

### 与 Task / Tools 的边界

- tool result、task output、resource read 可能成为 `ContentExternalization` 的 source
- 但它们的 identity 和 lifecycle 仍由 `Tools` / `Task` 各自拥有

`ContextEditing` 只决定这些高体积内容如何进入模型可见上下文。

### 与 Prompt Cache Strategy 的边界

- `PromptCacheStrategy`
  - 负责稳定前缀、动态后缀、cache break 语义
- `ContextEditing`
  - 负责上下文本身如何被 externalize、折叠、重写

二者会协同，但不应混成同一能力。

## 规范结论

- `ContextEditing` 是 `Harness.ContextEngineering` 的独立能力，不等于单次 compact
- payload externalization、working-view projection、compaction rewrite 必须语义分离
- runtime 应优先采用 progressive degradation，而不是直接 full compact
- edited context 的结果必须可恢复、可追溯，并与 durable transcript 分层
