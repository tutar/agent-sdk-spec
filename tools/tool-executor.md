# Tool Executor

## 职责

`ToolExecutor` 负责把模型产出的一个或多个 `tool_use` 安全地变成运行结果。

重点不是“把函数调起来”，而是处理并发、取消、顺序、progress 和上下文一致性。

它描述的是 tool invocation orchestration，不等价于 sandbox lifecycle，也不等价于执行环境本身。

## 标准接口

```text
ToolExecutor
  - run_tools(tool_uses, assistant_messages, tool_use_context)
    -> tool_execution_events, tool_execution_summary
```

## 必须支持的能力

- 当模型支持原生并行 tool calling 时，优先沿用模型的调用结构
- 当模型不支持时，由 harness 负责批处理与并发调度
- 按 `isConcurrencySafe` 分批
- 并发执行只读或安全工具
- 串行执行非安全工具
- 先发射 progress，再发射最终结果
- 在执行后统一合并 context mutation
- 用户中断时按工具的 `interruptBehavior` 处理
- 某个并行工具失败时取消兄弟工具

## 不属于 ToolExecutor 的职责

- 创建 sandbox
- 恢复或重建失效 execution target
- 管理远端 VPC 连接
- 持有凭据或直连外部密钥系统

这些应由 sandbox / security / orchestration 层承担。

## 推荐执行模型

1. 解析 `tool_use`
2. 查找 tool definition
3. 校验 schema
4. 进行权限审查
5. 按并发安全性分批
6. 发射 `tool_started`
7. 发射 `tool_progress`
8. 回收 `tool_result`
9. 应用 context modifier

## 为什么 context mutation 要延后

并发工具可以同时产出结果，但不能同时修改共享上下文，否则会出现：

- 顺序不确定
- 工具结果覆盖
- message/thread state 被交叉污染

因此建议把 context 修改当作单独阶段处理。

## 取消模型

SDK 应至少区分：

- `cancel`
  用户新输入到来时可中止当前工具
- `block`
  当前工具必须跑完，新输入等待

## 当前仓库映射

- 批处理编排见 [services/tools/toolOrchestration.ts](../../cc/services/tools/toolOrchestration.ts)
- 流式执行器见 [services/tools/StreamingToolExecutor.ts](../../cc/services/tools/StreamingToolExecutor.ts)

## 规范结论

- 执行器必须独立于 tool 定义
- 并发和上下文修改必须分离
- 工具执行必须可取消、可观察、可恢复
- 工具执行语义必须不依赖某个模型厂商的原生协议
- tool executor 负责编排，不负责定义或承载 hands 本身
