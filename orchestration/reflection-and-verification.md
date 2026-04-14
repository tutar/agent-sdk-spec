# Reflection And Verification

## 职责

`Reflection And Verification` 负责把“完成后再检查一次”从提示词技巧提升为标准编排能力。

它至少覆盖两类能力：

- `verification`
  对主 agent 产物做独立复核
- `reflection task`
  对已产生的结果、证据或会话状态做后置加工

规范上要明确：

- verification 不等于主 agent 自己再想一遍
- verification 应允许独立 agent、独立工具集和独立 verdict
- reflection 不应与 compact、memory consolidation、post-turn hooks 混为同一对象

## 稳定接口

推荐最小接口：

```text
VerificationRequest
  - target_session
  - target_agent?
  - original_task
  - changed_artifacts?
  - evidence_scope?
  - verification_policy?

VerificationResult
  - verdict: pass | fail | partial
  - evidence[]
  - findings[]
  - limitations[]

ReflectionOrchestrator
  - spawn_verifier(request) -> verifier_handle
  - await_verifier(handle) -> verification_result
  - attach_verification(session_or_task, verification_result)
```

可选补充：

```text
ReflectionTask
  - kind: verification | critique | review
  - trigger: explicit | automatic
  - status
  - output_ref?
```

## 默认实现

当前代码库的默认实现重点在 verifier agent：

- [tools/AgentTool/built-in/verificationAgent.ts](../../tools/AgentTool/built-in/verificationAgent.ts)
  内置 verification specialist，要求独立运行命令、拿到证据、输出 `VERDICT`
- [tools/AgentTool/builtInAgents.ts](../../tools/AgentTool/builtInAgents.ts)
  注册 built-in verifier
- [commands/init-verifiers.ts](../../commands/init-verifiers.ts)
  初始化项目级 verifier skills
- [tools/TodoWriteTool/TodoWriteTool.ts](../../tools/TodoWriteTool/TodoWriteTool.ts)
  在任务收尾阶段推动 verifier 进入标准流程
- [tools/TaskUpdateTool/TaskUpdateTool.ts](../../tools/TaskUpdateTool/TaskUpdateTool.ts)
  在多任务收尾时要求验证步骤

当前默认实现表明：

- verification 被建模成独立 agent
- verifier 默认与主 agent 分离
- verifier 的 prompt、工具边界和输出格式独立于主 agent

## 要解决的问题

- 如何避免主 agent 既实现又自判正确
- 如何为非平凡任务提供独立 verdict
- 如何把 verification 纳入标准交付链，而不是依赖人工约定
- 如何让 verifier 在只读或受限工具集下运行
- 如何区分 correctness verification 与 memory consolidation / compact

## 规范结论

- verification 应作为 orchestration 中的标准模式存在
- verifier 最好是独立 agent，而不是主 agent 的附加 prompt 段
- verification 输出应结构化，至少包含 verdict 与 evidence
- reflection-and-verification 不应吞并 compact、memory consolidation 或 stop hooks 的职责
