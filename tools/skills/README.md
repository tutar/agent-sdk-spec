# Skills

## 职责

`Skills` 子模块负责定义提示词型能力的稳定接口。

它解决的不是“执行一个 tool”，而是：

- 发现 skill
- 解析 skill 元数据
- 加载 skill 内容
- 在合适时机激活 skill
- 通过桥接层把 skill 暴露给模型调用

在当前规范中：

- `Skill` 属于 `tools` 域
- `Skill` 不是 `Tool`
- `Skill` 的默认运行时载体可以是 `Command`
- `Skill` 只服务 Agent Skills 生态

## 稳定接口

推荐最小接口：

```text
SkillDefinition
  - id
  - name
  - description
  - content
  - arguments
  - when_to_use
  - allowed_tools
  - metadata

SkillRegistry
  - discover_skills(scope)
  - load_skill(skill_id)
  - invalidate_skills(scope)

SkillActivator
  - activate_skill(skill_id, args, context)
  - render_skill_prompt(skill_id, args, context)

SkillInvocationBridge
  - list_model_invocable_skills()
  - invoke_skill(skill_id, args, runtime_context)
```

`SkillInvocationBridge` 是必要的，因为 skill 本身不是 tool；模型若只能调用 tools，需要桥接层把 skill 暴露成可调用入口。

如果某个实现内部使用 `Command` 作为 skill 的载体，该对象模型应视为默认实现细节，而不是把 `Skill` 降格为 `Command` 的别名。

## 默认实现

当前代码库中的默认实现是：

- [skills/loadSkillsDir.ts](../../../cc/skills/loadSkillsDir.ts)
  从 `skills/` 目录和相关来源加载 skill，并构造成 `Command(type='prompt')`
- [types/command.ts](../../../cc/types/command.ts)
  定义 `PromptCommand` / `Command`，这是 skill 的默认运行时对象
- [commands.ts](../../../cc/commands.ts)
  提供 `getSkillToolCommands()`、`getMcpSkillCommands()` 等筛选逻辑
- [tools/SkillTool/SkillTool.ts](../../../cc/tools/SkillTool/SkillTool.ts)
  作为默认 `SkillInvocationBridge`

源码里的关键判断是：

- skill 的一等对象不是 `Tool`
- skill 默认由 `Command(type='prompt')` 承载
- `SkillTool` 只是桥接层

## 要解决的问题

- 如何把 prompt/workflow 型能力做成稳定接口
- 如何避免把 skill 和 tool 混为一谈
- 如何支持本地 skill、bundled skill、plugin skill、managed skill、MCP skill
- 如何控制哪些 skill 对模型可见、哪些只对用户可见
- 如何在不破坏 skill 语义的前提下，通过桥接 tool 暴露给模型

## 当前源码映射

- `createSkillCommand()` 在 [skills/loadSkillsDir.ts](../../../cc/skills/loadSkillsDir.ts) 中把 skill 转成 `Command(type='prompt')`
- `loadedFrom` 用于区分 skill 来源，包含 `skills / plugin / managed / bundled / mcp`
- `getSkillToolCommands()` 在 [commands.ts](../../../cc/commands.ts) 中筛选模型可见 skill
- `SkillTool` 在 [tools/SkillTool/SkillTool.ts](../../../cc/tools/SkillTool/SkillTool.ts) 中调用 skill，并在必要时 fork sub-agent

因此，当前源码里的关系应理解为：

```text
Skill        -> 能力语义
Command      -> 默认对象载体
SkillTool    -> 模型调用桥接
```

## 规范结论

- `Skill` 是独立于 `Tool` 的稳定接口
- skill 的默认对象模型应允许 prompt/workflow 型表达
- skill 是否由模型调用，应由桥接层决定，而不是由 skill 定义本身承担
- `SkillTool` 不应被当成 `Skill` 本身
