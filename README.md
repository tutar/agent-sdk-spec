# Agent SDK Spec

本目录将规范重构为 5 个核心模块子目录，每个模块目录都以一个 `README.md` 作为主规范，统一回答四件事：

- 职责
- 稳定接口
- 默认实现
- 要解决的问题

目标不是沉淀某个具体语言或某一版 harness 的实现细节，而是形成一套可向下指导多语言 SDK、向上承接不同模型与基础设施能力的稳定接口规范。

## 五个核心模块

- [harness/README.md](harness/README.md)
- [session/README.md](session/README.md)
- [tools/README.md](tools/README.md)
- [sandbox/README.md](sandbox/README.md)
- [orchestration/README.md](orchestration/README.md)

其中 [harness/README.md](harness/README.md) 已进一步按 5 组子主题组织：

- Runtime Core
- Model Integration
- Context Assembly
- Ingress Gateway
- Extension And Projection

在 `Context Assembly` 子主题下，现已单独补充 bootstrap prompt 规范，用于稳定 system prompt skeleton、section cache 与 static/dynamic boundary。

另外，反思相关能力已按职责拆入现有模块，而不是新增顶层 `reflection` 模块：

- `orchestration` 下的 verification / reflection task
- `harness` 下的 post-turn processing
- `session` 下的 memory consolidation

此外，规范现在显式面向三种宿主场景：

- `TUI`
  交互式终端 agent host，例如 Claude Code、Codex CLI
- `Desktop`
  GUI + 本地 daemon / bundle host
- `Cloud`
  托管 control plane + remote execution

这三种场景不改变五个核心模块的边界，但会改变它们的部署位置、职责分布和默认实现映射。相关差异主要通过 `orchestration/hosting-profiles/` 表达。

## 共享文档

- [conformance.md](conformance.md)
- [conformance/README.md](conformance/README.md)
- [capability-surface.md](capability-surface.md)
- [extension-packaging.md](extension-packaging.md)
- [object-model.md](object-model.md)
- [schema-serialization.md](schema-serialization.md)
- [test-artifacts.md](test-artifacts.md)
- [managed-agent-conformance-scenarios.md](managed-agent-conformance-scenarios.md)
- [todo.md](todo.md)

## 设计原则

- 本规范首先稳定模块边界，再允许实现自由演进。
- 优先稳定 `Harness / Session / Tools / Sandbox / Orchestration` 五个对象。
- 优先复用模型原生能力，能力不足时由 harness 补齐。
- durable history、执行环境、工具编排、托管编排必须解耦。
- 同一套模块边界必须同时适配 TUI、Desktop、Cloud 三种宿主形态。
- 文档中的接口描述是跨语言语义，不是具体语言 API。
- `command` 会作为 `tools` 域内共享抽象出现，但不升级为第六个顶层模块。

## 与当前代码的默认映射

- Harness: [QueryEngine.ts](../QueryEngine.ts), [query.ts](../query.ts), [context.ts](../context.ts)
  Harness 侧还包含入口网关抽象，当前默认映射到 [bridge/initReplBridge.ts](../bridge/initReplBridge.ts), [bridge/replBridge.ts](../bridge/replBridge.ts), [bridge/bridgeMessaging.ts](../bridge/bridgeMessaging.ts), [bridge/replBridgeTransport.ts](../bridge/replBridgeTransport.ts)
- Session: [bootstrap/state.ts](../bootstrap/state.ts), [utils/sessionStorage.ts](../utils/sessionStorage.ts), [utils/sessionRestore.ts](../utils/sessionRestore.ts), [utils/sessionState.ts](../utils/sessionState.ts)
- Tools: [Tool.ts](../Tool.ts), [tools.ts](../tools.ts), [services/tools/toolOrchestration.ts](../services/tools/toolOrchestration.ts), [services/tools/StreamingToolExecutor.ts](../services/tools/StreamingToolExecutor.ts)
  `tools` 域下还包含 `skills`、`mcp` 两个稳定子接口，以及 `command` 共享对象模型；默认映射分别位于 [skills/loadSkillsDir.ts](../skills/loadSkillsDir.ts)、[services/mcp/client.ts](../services/mcp/client.ts)、[types/command.ts](../types/command.ts)
- Sandbox: [tools/BashTool/BashTool.tsx](../tools/BashTool/BashTool.tsx), [tasks/LocalShellTask/LocalShellTask.tsx](../tasks/LocalShellTask/LocalShellTask.tsx), [utils/sandbox/sandbox-adapter.ts](../utils/sandbox/sandbox-adapter.ts)
- Orchestration: [tasks.ts](../tasks.ts), [tasks/LocalAgentTask/LocalAgentTask.tsx](../tasks/LocalAgentTask/LocalAgentTask.tsx), [tasks/RemoteAgentTask/RemoteAgentTask.tsx](../tasks/RemoteAgentTask/RemoteAgentTask.tsx), [tools/AgentTool/runAgent.ts](../tools/AgentTool/runAgent.ts)
  `orchestration` 域下默认还包含标准 agent 编排模式，映射到 [tools/AgentTool/AgentTool.tsx](../tools/AgentTool/AgentTool.tsx)、[tasks/InProcessTeammateTask/InProcessTeammateTask.tsx](../tasks/InProcessTeammateTask/InProcessTeammateTask.tsx)、[tools/shared/spawnMultiAgent.ts](../tools/shared/spawnMultiAgent.ts)
