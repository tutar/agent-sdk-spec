# Hook Runtime

## 职责

`HookRuntime` 负责在不改写核心 runtime loop 的前提下，为 turn、tool、permission、session-start、compact、stop 等生命周期边界提供可插拔控制点。

它是 runtime 的 hook plane，不是泛化的“扩展杂项”。

## 应覆盖的事件面

规范上至少应允许以下几类事件：

- session start / end
- turn stop / stop failure
- subagent start / stop
- teammate idle / task completed
- pre-tool / post-tool / post-tool-failure
- permission request / permission denied
- pre-compact / post-compact
- notification / config change

## 稳定接口

```text
HookRegistry
  - register_hook(scope, event, matcher, hook)
  - remove_hook(scope, hook_id)
  - list_hooks(scope, event?)
  - resolve_matching_hooks(event, input, scope) -> matched_hooks

HookRuntime
  - execute_hooks(event, input, runtime_context) -> hook_results
  - emit_hook_events(event_execution) -> external_events

HookResult
  - outcome: success | blocking | non_blocking_error | cancelled
  - message?
  - blocking_error?
  - system_message?
  - additional_context?
  - metadata
```

## 设计要求

- hook 与 tool 必须分层
- hook 执行与 hook 结果投影必须分层
- hook 不应直接改写 session transcript，而应通过结构化结果影响 runtime
- hook 必须支持 session-scoped 注册
- hook 可影响 continuation、approval、blocking，但不得绕过 policy 与 session lifecycle 语义
- hook batch 可并行执行，但最终结果必须结构化聚合

## 结果投影原则

hook 运行结果可以投影到多个平面，但必须保持语义分层：

- 对 message pipeline 的投影
- 对 policy chain 的投影
- 对 session lifecycle 的投影
- 对独立 hook event stream 的投影

另外，lifecycle 扩展产出的 `additional_context` 不应被视为可以直接绕过 context engineering 的隐式输入。

## 与其它模块的边界

- 不等于 `Tools`
- 不等于 `PolicyEngine`
- 不等于 `SessionLifecycleState`
- 不等于 `Plugin`

它是：

- runtime 的 hook execution plane
- policy、session、tools 的旁路控制面

## 规范结论

- hooks 应被建模为 runtime hook plane
- hook runtime 应支持匹配、执行、聚合和投影
- lifecycle 产物必须先进入 context engineering，再进入模型输入
- hook 是运行时控制面，不是模型能力面
