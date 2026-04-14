# Environment Sandbox

## 目标

`Environment Sandbox` 规范工作区、容器、VM 或远端 workspace 级别的沙箱。

它面向的是更长生命周期的执行环境，而不是单次命令包装。

典型场景包括：

- 容器级 agent workspace
- remote execution node
- ephemeral VM
- desktop local isolated workspace

## 适用层级

这一层对应：

- container-level isolation
- workspace-level isolation
- remote execution environment

它与 `Execution Sandbox` 的区别在于：

- 生命周期更长
- 作用域更大
- 常与 provisioning / orchestration 绑定

## 稳定接口

推荐最小接口：

```text
EnvironmentSandbox
  - provision(provision_request) -> sandbox_handle
  - resume(sandbox_handle) -> status
  - inspect(sandbox_handle) -> sandbox_status
  - recycle(sandbox_handle)
  - destroy(sandbox_handle)
```

建议补充：

```text
EnvironmentSandboxStatus
  - sandbox_id
  - isolation_level
  - workspace_ref
  - network_profile
  - filesystem_profile
  - credential_boundary
  - lifecycle_state
```

## 设计要求

- `Environment Sandbox` 应与 orchestration 的 provisioning 能力协同
- 应允许懒创建、恢复、替换和销毁
- 应把 workspace / image / remote location 视为一等配置
- 不应假设与 harness 共机或同进程

## 默认实现映射

当前代码库没有完整、统一的 `Environment Sandbox` 顶层实现，但存在相关默认映射：

- [bridge/sessionRunner.ts](../../cc/bridge/sessionRunner.ts)
  本地/子进程 session 承载
- [tools/AgentTool/AgentTool.tsx](../../cc/tools/AgentTool/AgentTool.tsx)
  agent isolation routing
- [utils/teleport.tsx](../../cc/utils/teleport.tsx)
  remote/teleport 场景下的 workspace/environment 迁移

因此当前仓库更接近：

- `Execution Sandbox` 已较完整
- `Environment Sandbox` 仅有部分能力散落在 remote/workspace/orchestration 路径中

## 要解决的问题

- 如何为 agent 或 task provision 长生命周期隔离环境
- 如何把容器、VM、remote workspace 统一纳入 sandbox 语义
- 如何与 session/orchestration 配合做恢复和迁移
- 如何把 credential boundary、workspace boundary、network boundary 绑定到环境级实例

## 规范结论

- `Environment Sandbox` 应作为 sandbox 模块中的独立层级存在
- 它主要描述容器级、workspace级和远端级隔离
- 即使当前默认实现不完整，规范上也应保留这一层
