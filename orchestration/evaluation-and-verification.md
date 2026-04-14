# Evaluation And Verification

## 职责

`EvaluationAndVerification` 定义 orchestration 层如何对 agent 结果做独立验证，而不是依赖主 agent 自评。

它负责：

- 在合适阶段触发 verifier
- 将验证视为独立 agent / task
- 为验证产出结构化 verdict
- 将验证结果回流主编排链

它不负责：

- 通用 telemetry
- session lifecycle 投影
- tool-level tracing

## 稳定接口

推荐最小接口：

```text
Verifier
  - evaluate(change_context) -> VerificationResult

VerificationResult
  - verdict: PASS | FAIL | PARTIAL
  - evidence[]
  - limitations[]
  - verifier_id
```

## 规范要求

### 1. 验证应由独立执行单元完成

不应将主 agent 自检视为充分验证。

### 2. verdict 必须结构化

至少支持：

- `PASS`
- `FAIL`
- `PARTIAL`

### 3. evidence 必须来自真实执行

代码阅读、推测和口头解释不能替代执行证据。

### 4. orchestration 应允许自动触发 verifier

对非平凡改动，编排层应允许自动或半自动进入 verification 阶段。

## 推荐默认策略

- 将 verifier 建模成独立 agent
- verifier 默认只读项目目录
- verifier 输出结构化 evidence + verdict
- 主 agent 在汇报完成前应等待 verifier 结果

## 默认实现映射

- built-in verifier 见 [tools/AgentTool/built-in/verificationAgent.ts](../../cc/tools/AgentTool/built-in/verificationAgent.ts)
- verifier agent 注册见 [tools/AgentTool/builtInAgents.ts](../../cc/tools/AgentTool/builtInAgents.ts)
- 主 prompt 中的验证契约见 [constants/prompts.ts](../../cc/constants/prompts.ts)
- 关闭任务后的验证提醒见 [tools/TodoWriteTool/TodoWriteTool.ts](../../cc/tools/TodoWriteTool/TodoWriteTool.ts) 和 [tools/TaskUpdateTool/TaskUpdateTool.ts](../../cc/tools/TaskUpdateTool/TaskUpdateTool.ts)
- verifier skill 初始化见 [commands/init-verifiers.ts](../../cc/commands/init-verifiers.ts)

## 规范结论

- agent 评估应优先通过独立 verifier 完成
- verifier 应作为 orchestration 中的标准模式，而不是临时技巧
- 监控与评估必须分开建模
