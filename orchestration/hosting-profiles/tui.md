# TUI Hosting Profile

## 目标

`TUI Hosting Profile` 描述交互式终端 agent host 的标准职责分布。

典型形态包括：

- Claude Code
- Codex CLI
- 其他长期运行、可持续交互的终端 agent shell

这里的重点不是“一次性 CLI 命令”，而是：

- 持续会话
- 本地 interaction loop
- 本地权限交互
- 本地 session 与本地任务编排

## 角色分布

在 TUI 场景下，推荐的职责分布是：

- `ChannelAdapter`
  终端输入输出、键盘事件、TUI 组件
- `IngressGateway`
  本地输入标准化、session binding、control routing
- `Harness`
  本地 turn evaluation
- `Session`
  本地 durable transcript + restore
- `Orchestration`
  本地 task / subagent / verifier / background task
- `Sandbox`
  以 `Execution Sandbox` 为主

## 典型特征

- REPL-like interaction loop 是主路径
- session 往往本地持久化
- permission prompt 通常可直接本地交互
- background task 与 subagent 常由本机进程承载
- verifier 常作为本地独立 agent/task 执行

## 对 orchestration 的要求

- 支持本地 task-first orchestration
- 支持 foreground / background agent 切换
- 支持本地 verifier 作为标准后置步骤
- 支持本地 permission / requires_action 流
- 支持 TUI 关闭后 resume 本地 session

## 默认实现映射

当前代码库中的默认 TUI hosting profile 主要映射到：

- [screens/REPL.tsx](../../../screens/REPL.tsx)
- [QueryEngine.ts](../../../QueryEngine.ts)
- [utils/sessionStorage.ts](../../../utils/sessionStorage.ts)
- [tasks/LocalAgentTask/LocalAgentTask.tsx](../../../tasks/LocalAgentTask/LocalAgentTask.tsx)
- [tools/AgentTool/built-in/verificationAgent.ts](../../../tools/AgentTool/built-in/verificationAgent.ts)

## 规范结论

- TUI profile 应被视为本地 agent host，而不是简单 CLI 命令包装
- 在该 profile 中，orchestration 通常以本地 task、subagent、permission 交互为中心
