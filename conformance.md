# Conformance

## 目标

本文件定义各语言 SDK 的一致性要求。

不同语言实现可以使用不同的并发模型、序列化库、HTTP 客户端和类型系统，但不能改变 agent 的核心行为语义。

## 一致性范围

需要跨语言保持一致的内容：

- 事件语义
- 终止状态语义
- tool 权限语义
- requires_action 语义
- context governance 触发语义
- 模型能力路由语义
- task 生命周期语义
- Skills / MCP / MCPB 兼容语义
- `tool / skill / command / mcp` 的角色语义

允许不同实现存在差异的内容：

- 内部类名、函数名、文件组织
- 线程、协程、异步任务实现方式
- 本地缓存、序列化、日志框架
- 网络重试的具体实现细节

## 强制一致的行为

### 1. Runtime Event Semantics

各语言实现必须对外暴露等价的事件分类。

例如：

- `assistant_delta`
- `tool_started`
- `tool_progress`
- `tool_result`
- `requires_action`
- `turn_completed`
- `turn_failed`

事件字段名可以按语言习惯调整，但事件含义不能变化。

### 2. Terminal State Semantics

各语言实现必须能区分：

- 正常完成
- 用户中止
- 权限阻塞
- 预算终止
- 运行失败

不能把这些终止类型混成单一 `error`。

### 3. Tool Execution Semantics

各语言实现必须保持：

- 并发安全工具可并发
- 非并发安全工具必须串行
- context mutation 不能并发乱序应用
- 用户中断行为必须尊重 tool 的中断语义

### 4. Policy Semantics

各语言实现必须保持：

- 权限决策有 `allow/deny/ask`
- `requires_action` 是结构化状态
- 非交互环境可自动拒绝
- 静态规则与运行时审查是两层逻辑

### 5. Context Governance Semantics

各语言实现必须保持：

- 超长上下文会触发治理
- 超长工具结果不能无约束进入上下文
- compact 与 overflow recovery 可被 runtime 调用
- budget continuation 有一致的停止条件

### 6. Ecosystem Compatibility Semantics

各语言实现若宣称实现 `Tools` 模块，应保持以下兼容语义：

- skill discovery / load / activate 的基本行为一致
- MCP tool / resource / prompt 的接入语义一致
- 桌面端实现的 bundle install / load / verify 语义一致
- `skill` 与 `tool` 是不同稳定接口
- `command` 是共享对象模型，不是第六顶层模块
- `mcp` 是协议接入层，不等于一组远端 tools

允许实现差异：

- 具体缓存方式
- 传输实现
- UI 呈现形式

不允许差异：

- skill 与 tool 的能力来源模型
- command 与 skill/tool 的角色分层
- MCP prompt/resource/tool 的基本角色定义
- MCP skill 与 MCP prompt 的区分语义
- 桌面 bundle 的安装后生命周期语义

## 一致性测试基线

每个语言 SDK 至少应具备以下测试类型。

### Contract Tests

验证标准输入输出契约：

- tool definition 契约
- model capability view 契约
- requires_action 契约
- runtime terminal state 契约

### Behavior Tests

验证行为一致性：

- 并发 tool 执行顺序
- 权限 ask/deny/allow 分支
- compact 触发与恢复
- 子 agent 生命周期

### Failure Tests

验证异常情况下的统一行为：

- 模型限流
- prompt too long
- tool 执行失败
- 用户中断
- 非交互环境拒绝审批

### Compatibility Tests

验证不同模型接入后语义不漂移：

- 支持原生 tool calling 的模型
- 不支持原生 tool calling 的模型
- 支持 structured output 的模型
- 不支持 structured output 的模型

还应包含生态兼容测试：

- Agent Skills skill 目录能被发现并激活
- MCP server 的 tools/resources/prompts 能被正确枚举与调用
- 桌面端 `.mcpb` bundle 能被安装、校验与加载
- 使用内部 command model 的实现，不会把 skill 或 mcp prompt 错误暴露成 tool

## 推荐测试工件

建议维护一套跨语言共享的测试工件：

- 标准事件录制样本
- 标准 tool 用例集
- 标准权限决策样本
- 标准 compact 场景样本
- 标准模型能力矩阵样本

这些工件应脱离具体语言实现，作为所有 SDK 的共同验收基线。

规范目录中的共享样例入口见：

- [conformance/README.md](conformance/README.md)
- [conformance/cases/basic-turn.md](conformance/cases/basic-turn.md)
- [conformance/cases/tool-call-roundtrip.md](conformance/cases/tool-call-roundtrip.md)
- [conformance/cases/requires-action-approval.md](conformance/cases/requires-action-approval.md)
- [conformance/cases/session-resume.md](conformance/cases/session-resume.md)
- [conformance/cases/background-agent.md](conformance/cases/background-agent.md)
- [conformance/cases/hosting-profile-equivalence.md](conformance/cases/hosting-profile-equivalence.md)
- [conformance/cases/mcp-tool-adaptation.md](conformance/cases/mcp-tool-adaptation.md)
- [conformance/cases/sandbox-deny.md](conformance/cases/sandbox-deny.md)
- [conformance/cases/memory-recall-and-consolidation.md](conformance/cases/memory-recall-and-consolidation.md)
- [conformance/cases/cloud-wake-and-reprovision.md](conformance/cases/cloud-wake-and-reprovision.md)
- [conformance/cases/prompt-cache-stable-prefix.md](conformance/cases/prompt-cache-stable-prefix.md)
- [conformance/cases/prompt-cache-dynamic-suffix.md](conformance/cases/prompt-cache-dynamic-suffix.md)
- [conformance/cases/prompt-cache-fork-sharing.md](conformance/cases/prompt-cache-fork-sharing.md)
- [conformance/cases/prompt-cache-break-detection.md](conformance/cases/prompt-cache-break-detection.md)
- [conformance/cases/prompt-cache-strategy-equivalence.md](conformance/cases/prompt-cache-strategy-equivalence.md)

## 版本兼容要求

规范升级时应明确：

- 哪些字段是新增可选字段
- 哪些行为是向后兼容变更
- 哪些行为变化属于破坏性变更

各语言 SDK 应对外声明自己实现的规范版本。

## 最低合规要求

一个语言 SDK 若要宣称“实现了本规范”，至少必须具备：

- 标准化模型适配层
- 标准化 runtime 事件流
- 标准化 tool 契约
- 标准化 policy 决策
- 标准化 context governance 基线

若要宣称“实现了本规范的 `Tools` 模块”，至少还应具备：

- Agent Skills compatibility
- MCP compatibility

若目标是桌面端，还应具备：

- MCPB compatibility

缺少其中任一项，只能称为局部实现。

## 规范结论

- 规范的目标是行为一致，不是代码结构一致
- 各语言 SDK 可以自由实现，但不能自由改变语义
- 合规测试应与语言实现解耦，作为共享验收标准存在
