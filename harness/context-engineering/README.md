# Context Engineering

## 职责

`Harness.ContextEngineering` 是 `Harness` 内部的上下文子域。

`AgentRuntime` 会在以下阶段调用它：

- startup / resume 前，建立稳定 instruction 与 bootstrap 输入
- 每轮 sampling 前，装配本轮模型可见输入
- pre-sampling request shaping 时，执行预算治理、编辑和 cache-aware 组织

它辅助 runtime 完成：

- 建模多平面模型输入，而不是只处理 prompt 字符串
- 区分 stable skeleton、startup-only context、per-turn dynamic context
- 装配 attachments、capability surface、instruction fragments、structured context
- 在预算压力下执行 governance、editing 和 prompt-cache-aware shaping

它不负责：

- runtime loop 本身
- session durable ownership
- tool 执行本身
- sandbox enforcement
- task lifecycle

## 设计定位

本子域采用 `Local-first + Cloud-compatible`：

- `Local-first`
  默认实现映射来自本地 harness 对 session、tools、sandbox 和 host state 的直接协作。
- `Cloud-compatible`
  `ContextEngineering` 仍由 harness 持有；变化的只是它消费 transcript、memory links、tool surface 和 environment context 的方式，而不是语义边界。

这意味着：

- `ContextEngineering` 属于 `Harness`
- 它被 `AgentRuntime` 调用，但不并入 `Runtime`
- 它不是单独远程服务，也不是第六个顶层模块

## Runtime 调用总图

```text
runtime startup / resume
-> instruction-markdown
-> entry
-> per-turn assembly
-> pre-sampling governance
-> final model input
```

## 子组

- [instruction-markdown/README.md](instruction-markdown/README.md)
  定义 file-backed instruction loading、include expansion 与 conditional instruction matching。
- [entry/bootstrap-prompts.md](entry/bootstrap-prompts.md)
  定义 runtime 在进入主 loop 前建立的 stable system skeleton。
- [entry/startup-and-turn-zero-context.md](entry/startup-and-turn-zero-context.md)
  定义 session-start、agent-start、turn-zero、resume-start 一类 lifecycle-scoped context。
- [assembly/context-input-model.md](assembly/context-input-model.md)
  定义模型可见输入的 canonical object model 与 context planes。
- [assembly/context-provider.md](assembly/context-provider.md)
  定义 fragment provider 如何为 runtime 提供结构化上下文片段。
- [assembly/context-assembly-pipeline.md](assembly/context-assembly-pipeline.md)
  定义 runtime 在一次 sampling 前如何按阶段装配最终模型输入。
- [assembly/attachment-assembly.md](assembly/attachment-assembly.md)
  定义 attachment envelope、ordering、scope 与 audience。
- [governance/context-governance.md](governance/context-governance.md)
  定义 runtime 如何在 pre-sampling 阶段做预算分析与治理决策。
- [governance/context-editing.md](governance/context-editing.md)
  定义 governance 触发后 runtime 可执行的 editing layers。
- [governance/prompt-cache-strategy.md](governance/prompt-cache-strategy.md)
  定义 harness 如何组织输入以提高 prompt cache reuse。
- [governance/anthropic-native-prompt-cache-guide.md](governance/anthropic-native-prompt-cache-guide.md)
  作为 provider-specific 的默认实现指南，不是规范能力本身。

## 规范结论

- `ContextEngineering` 是 `Harness` 内部的 runtime-facing 子域
- `AgentRuntime` 依赖它建立 startup input、per-turn assembly 和 pre-sampling shaping
- 模型可见输入必须按 object model、assembly、attachment、governance 分层
- startup-only context、per-turn context、editing 和 prompt cache strategy 不能混层
- `AGENTS.md` / `rules` 一类 instruction sources 属于 `Harness.ContextEngineering`，不是 `durable-memory`
