# MCP

## 职责

`MCP` 子模块负责定义外部协议接入的稳定接口。

它的职责不是“提供一个 MCP tool”，而是：

- 连接 MCP server
- 枚举 MCP tools
- 枚举 MCP prompts
- 枚举 MCP resources
- 在资源语义上发现 MCP skills
- 把这些外部对象适配到本地 runtime 的 `tool / skill / resource` 模型

## 稳定接口

推荐最小接口：

```text
McpClient
  - connect(server_descriptor)
  - disconnect(server_id)
  - list_tools(server_id)
  - list_prompts(server_id)
  - list_resources(server_id)
  - call_tool(server_id, tool_name, input)
  - get_prompt(server_id, prompt_name, args)
  - read_resource(server_id, resource_uri)

McpToolAdapter
  - adapt_mcp_tool(server_id, remote_tool)

McpPromptAdapter
  - adapt_mcp_prompt(server_id, remote_prompt)

McpSkillAdapter
  - discover_skills_from_resources(server_id, resources)
  - adapt_mcp_skill(server_id, remote_skill)
```

这里必须区分：

- `MCP` 是协议层
- `McpToolAdapter` 只是把 MCP tools 适配成 runtime tools
- MCP prompts 和 MCP skills 不应被吞并进同一个抽象

推荐在规范里显式写出默认落点：

```text
mcp tools   -> Tool
mcp prompts -> Command
mcp skills  -> Skill (default carrier may be Command)
```

## 默认实现

当前代码库中的默认实现是：

- [services/mcp/client.ts](../../../cc/services/mcp/client.ts)
  MCP 主客户端，负责连接 server 并拉取 tools/prompts/resources/skills
- [services/mcp/utils.ts](../../../cc/services/mcp/utils.ts)
  处理 server 维度的筛选、命名和 prompt/skill 区分
- [tools/MCPTool/MCPTool.ts](../../../cc/tools/MCPTool/MCPTool.ts)
  作为默认 `McpToolAdapter` 的底座
- [tools/ListMcpResourcesTool/ListMcpResourcesTool.ts](../../../cc/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts)
  暴露 resources 枚举能力
- [tools/ReadMcpResourceTool/ReadMcpResourceTool.ts](../../../cc/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts)
  暴露 resource 读取能力

## 要解决的问题

- 如何把 MCP 协议接入和本地 tool/skill 抽象解耦
- 如何同时支持 tools、prompts、resources 三类协议对象
- 如何在 resources 之上发现 skill，而不是错误地把所有 prompt 都当成 skill
- 如何让不同 MCP server 的能力接入后仍保留来源边界
- 如何给权限系统、UI、tool execution 提供统一但不失真的对象模型

## 当前源码映射

源码里的实现已经明确分了几条链路：

- `fetchToolsForClient()` 在 [services/mcp/client.ts](../../../cc/services/mcp/client.ts) 中通过 `tools/list` 拉取工具，并包装成 `Tool`
- `fetchCommandsForClient()` 在 [services/mcp/client.ts](../../../cc/services/mcp/client.ts) 中通过 `prompts/list` 拉取 MCP prompts，并包装成 `Command(type='prompt')`
- `fetchResourcesForClient()` 在 [services/mcp/client.ts](../../../cc/services/mcp/client.ts) 中通过 `resources/list` 拉取资源
- `fetchMcpSkillsForClient()` 在 [services/mcp/client.ts](../../../cc/services/mcp/client.ts) 中被单独调用，注释明确写的是从 `skill://` resources 发现 skills

源码中的关键洞察：

- MCP prompt 和 MCP skill 都会进入 `mcp.commands`
- 区分方式不是 `type`，而是 `loadedFrom === 'mcp'`
- MCP prompt 的命名形态是 `mcp__<server>__<prompt>`
- MCP skill 的命名形态是 `<server>:<skill>`

这些规则集中体现在 [services/mcp/utils.ts](../../../cc/services/mcp/utils.ts)。

## 规范结论

- `MCP` 应在 `tools` 域下保留为独立稳定子接口
- MCP tools、prompts、resources、skills 必须分别建模，再在 runtime 中汇合
- 不能把“MCP 兼容”简化成“支持一批远端 tools”
- 不能把 `mcp prompts` 与 `mcp skills` 合并成同一个对象语义
