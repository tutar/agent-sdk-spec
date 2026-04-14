# Case: MCP Tool Adaptation

## 目标

验证 MCP 接入后的能力映射语义。

## Preconditions

- runtime 支持 MCP
- 至少存在一个 MCP server

## Ingress

- 枚举 MCP 暴露的能力
- 触发一次 MCP tool 调用
- 若实现支持 prompt / skill 映射，也同时检查枚举结果

## Expected Mapping

必须保持以下角色区分：

- MCP tools -> `Tool`
- MCP prompts -> `Command` 或等价共享对象模型
- MCP skills -> `Skill` 语义对象，默认载体可为 `Command`

## Expected Runtime Behavior

- MCP tool 可通过标准 tool execution 语义调用
- MCP prompt 不应被错误暴露成 tool
- MCP skill 不应被错误退化成普通 prompt 列表项

## Allowed Variance

- 不同语言实现可使用不同 MCP client
- Desktop host 可增加 bundle / install / trust 流程

## Failure Conditions

- 把 MCP prompt 映射成 tool
- 丢失 MCP skill 与 MCP prompt 的语义差别
- MCP tool 无法参与标准 tool lifecycle
