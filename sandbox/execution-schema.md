# Execution Schema

## 目标

本文件定义 `sandbox / execution` 相关的标准对象。

它关注执行面接口，而不是具体实现为本地 shell、进程级沙箱、容器、远端节点还是 MCP host。

## HandDescriptor

用于描述一个可路由的 execution target。

```text
HandDescriptor
  - hand_id
  - kind
  - capabilities
  - location
  - trust_level
  - isolation_level
```

## SandboxHandle

用于描述一个被 provision 出来的执行实例。

```text
SandboxHandle
  - sandbox_id
  - hand_id
  - status
  - created_at
  - expires_at?
  - metadata?
```

## ProvisionRequest

用于表达创建或准备 execution target 的请求。

```text
ProvisionRequest
  - requested_capabilities
  - location_preference?
  - isolation_level?
  - resources?
  - ttl?
```

## ExecutionRequest

用于表达一次动作执行请求。

```text
ExecutionRequest
  - request_id
  - target_ref
  - action_type
  - input
  - timeout_ms?
  - streaming?
```

## ExecutionResult

用于统一执行层返回结果。

```text
ExecutionResult
  - success
  - status
  - output
  - structured_output?
  - error?
  - retryable?
  - requires_reprovision?
```

## 规范结论

- `HandDescriptor`、`SandboxHandle`、`ProvisionRequest`、`ExecutionRequest`、`ExecutionResult` 应作为执行层共享对象
- `ToolExecutor` 不应替代这些对象
- 这些对象既可服务 `Execution Sandbox`，也可服务 `Environment Sandbox`
