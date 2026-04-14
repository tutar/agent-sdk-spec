# Tools

## 职责

`Tools` 模块负责把外部能力表达成模型可调用、系统可执行、策略可审查的能力单元。

这个模块包含三层：

- tool definition
- tool registry
- tool execution

同时，`Tools` 不是单一对象模型，而是一个能力域。当前规范在该目录下进一步区分三个稳定子接口和一个共享对象模型：

- `tool`
  执行型能力，面向 `call`、schema、权限和执行编排
- `skill`
  提示词/工作流型能力，面向发现、加载、激活和模型调用桥接
- `mcp`
  外部协议接入能力，面向 `tools / prompts / resources / skills` 的统一接入
- `command`
  `tools` 域内的共享对象模型，用来承载 prompt/local/ui 型入口对象及 skill 的默认运行时表示

这套能力域需要跨宿主形态保持一致：

- `TUI`
  工具常以本地 tool registry + 本地执行为主
- `Desktop`
  工具常与本地 bundle、MCP host、GUI 能力集成
- `Cloud`
  工具常与 remote tool proxy、remote execution、managed MCP 集成

同时，宿主可以把不同来源的 invocable capability 投影到统一命令面。

例如：

- built-in commands
- bundled skills
- plugin commands
- plugin skills

这些能力可以在同一个 command surface 中展示，但其内部来源语义仍需保留。共享规范见 [../capability-surface.md](../capability-surface.md)。

当前源码已经体现了这层区分：

- `Tool` 是 [Tool.ts](../../cc/Tool.ts)
- `Skill` 是独立能力语义接口，默认由 [types/command.ts](../../cc/types/command.ts) 中的 `Command(type='prompt')` 承载
- `Command` 的默认对象模型定义在 [types/command.ts](../../cc/types/command.ts)
- `MCP` 是 [services/mcp/client.ts](../../cc/services/mcp/client.ts) 驱动的协议接入层
- [tools/SkillTool/SkillTool.ts](../../cc/tools/SkillTool/SkillTool.ts) 只是把 `Skill` 暴露给模型调用的桥接 tool，不是 skill 本身

## 生态兼容要求

`Tools` 模块不是封闭接口，默认应满足外部生态兼容性，以保证采用本规范的 SDK 可以直接复用现有工具与技能生态。

推荐分为三层兼容：

- `Skills`
  兼容 Agent Skills 规范，使 SDK 能发现、加载和执行标准 skill 目录。
- `MCP`
  兼容 Model Context Protocol，使 SDK 能作为 MCP client 接入标准 MCP servers。
- `MCPB`
  若是桌面端实现，兼容 MCP Bundles（`.mcpb`），支持本地 MCP server 的分发、安装与加载。

这三层里：

- `Skills` 与 `MCP` 应视为默认兼容目标
- `MCPB` 是桌面端附加兼容目标

## 稳定接口

推荐最小接口：

```text
ToolDefinition
  - name
  - input_schema
  - description()
  - call()
  - check_permissions()
  - is_concurrency_safe()

ToolRegistry
  - list_tools()
  - resolve_tool(name)
  - filter_visible_tools(policy, runtime)

ToolExecutor
  - run_tools(tool_calls, context)
```

为满足生态兼容，推荐在 `Tools` 域下再补三组接口：

```text
CommandModel
  - resolve_command(name)
  - list_commands(scope)
  - invoke_command(name, args, context)
```

以及：

```text
SkillRegistry
  - discover_skills()
  - load_skill(skill_id)
  - activate_skill(skill_id, context)

McpClient
  - connect(server_descriptor)
  - list_tools()
  - list_resources()
  - list_prompts()
  - call_tool()
  - read_resource()
  - get_prompt()
```

桌面端可再补：

```text
McpBundleHost
  - install_bundle(bundle_file)
  - verify_bundle(bundle_file)
  - load_bundle(bundle_id)
  - update_bundle(bundle_id)
  - uninstall_bundle(bundle_id)
```

## 默认实现

当前代码库中的默认实现是：

- [Tool.ts](../../cc/Tool.ts)
  定义工具契约、权限接口、并发与中断语义
- [tools.ts](../../cc/tools.ts)
  组装 builtin tools、feature-gated tools、MCP tools，并注册 `SkillTool`
- [types/command.ts](../../cc/types/command.ts)
  定义默认 `Command` 共享对象模型
- [commands.ts](../../cc/commands.ts)
  组装本地命令、skills、MCP commands，并做可见性筛选
- [services/tools/toolOrchestration.ts](../../cc/services/tools/toolOrchestration.ts)
  按并发安全性分批执行
- [services/tools/StreamingToolExecutor.ts](../../cc/services/tools/StreamingToolExecutor.ts)
  处理流式 tool use、取消传播和结果有序发射

默认执行平面上的代表工具包括：

- [tools/BashTool/BashTool.tsx](../../cc/tools/BashTool/BashTool.tsx)
  默认 shell 执行能力
- [tools/AgentTool/AgentTool.tsx](../../cc/tools/AgentTool/AgentTool.tsx)
  默认子 agent / orchestration 入口
- [tools/SkillTool/SkillTool.ts](../../cc/tools/SkillTool/SkillTool.ts)
  skill invocation bridge，负责把 skill 暴露成模型可调用入口

与外部生态的默认映射是：

- Agent Skills
  当前仓库已有 `skills/` 目录结构与 `SKILL.md` 驱动模式，默认实现落在 [skills/loadSkillsDir.ts](../../cc/skills/loadSkillsDir.ts)
- MCP
  当前仓库已有 MCP client、resource、prompt、tool 集成路径，默认实现落在 [services/mcp/client.ts](../../cc/services/mcp/client.ts)
- MCPB
  当前仓库暂无完整桌面 bundle host 规范实现，但规范上应为桌面端预留 bundle install/load/verify/update 能力

## 要解决的问题

- 如何让模型可靠地调用外部能力
- 如何在不同模型之间保持 tool 语义一致
- 如何把 schema、权限、并发、安全放进同一份能力声明
- 如何在长会话中处理 tool progress、tool failure、tool result persistence
- 如何避免把 `skill`、`mcp`、`tool` 三种不同来源混成一个抽象
- 如何在不新增顶层模块的前提下，把 `command` 这种共享对象模型写清楚
- 如何避免 SDK 自建一个与 Skills/MCP 生态割裂的封闭工具系统
- 如何让桌面端把本地 MCP server 分发与安装纳入同一套规范
- 如何让同一 tool/skill/mcp 语义同时适配 TUI、Desktop、Cloud

## 默认设计取向

- 工具不是简单函数，而是运行时能力对象
- registry 不是静态数组，而是 capability assembly
- executor 不负责 hand 生命周期，只负责调用编排
- tool result 应支持大结果外存化与恢复
- `tool / skill / mcp` 应在能力域内并列建模，再在模型可见能力层汇合
- `command` 是共享对象模型，不是新的顶层能力模块
- `SkillTool` 是桥接 tool，不是 skill 抽象本身
- 协议兼容优先于私有扩展

## 与其它模块的边界

- tool 定义不等于 sandbox
- tool executor 不等于 orchestration
- tool registry 不等于 session history
- skill registry 不等于 tool registry
- mcp client 不等于 mcp tool adapter
- command model 不等于顶层模块

## 目录内文档

- [builtin-tool-baseline.md](builtin-tool-baseline.md)
- [tool-definition.md](tool-definition.md)
- [tool-executor.md](tool-executor.md)
- [tool-streaming-execution.md](tool-streaming-execution.md)
- [policy-engine.md](policy-engine.md)
- [command-model.md](command-model.md)
- [skills/README.md](skills/README.md)
- [mcp/README.md](mcp/README.md)

## 外部规范

- Agent Skills specification: <https://agentskills.io/specification>
- Model Context Protocol specification: <https://modelcontextprotocol.io/specification/2025-06-18>
- MCP Bundles (`.mcpb`) for desktop hosts: <https://github.com/modelcontextprotocol/mcpb>

## 规范结论

- tools 是模型与外部世界之间的能力层
- tools 模块必须同时面向模型、运行时、策略和恢复设计
- 默认实现可变，但 `definition / registry / executor` 三层边界应保持稳定
- 采用本规范的 SDK 默认应兼容 Skills 与 MCP 生态
- 桌面端实现默认应进一步兼容 MCP Bundles 生态
- `skill` 与 `mcp` 应作为 `tools` 域下的独立稳定子接口保留
- `command` 应作为 `tools` 域内的共享对象模型进入 spec
- tools 语义应跨 TUI、Desktop、Cloud 保持一致，只允许 registry/host 形态变化
- 宿主侧统一 command surface 可以混合展示 built-in commands 与 bundled skills，但不能抹平其来源差异
- built-in tools 不应只写成抽象概念，baseline 应单独定义
