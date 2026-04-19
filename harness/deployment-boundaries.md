# Deployment Boundaries

## 目标

本文定义 `Harness` 在不同 hosting profile 下的部署边界。

它回答的问题是：

- `Harness` 在 `Local` 和 `Cloud` 下是否还是同一个整体
- `Harness` 与 `Session`、`Tools`、`Sandbox`、`Orchestration` 的调用边界如何变化
- 哪些差异属于 binding 变化，哪些差异不应改变语义

它不讨论：

- chat/channel 接入
- UI 形态
- provider-specific transport

这些分别属于 `Gateway` 与 `Model Provider` 子域。

## 核心结论

`Harness` 无论在 `Local` 还是 `Cloud` 下，都应被视为一个完整整体。

这意味着：

- runtime
- context engineering
- task
- permission
- multi-agent
- post-turn processing

这些能力都仍然属于 harness 内部，而不是在 `Cloud` 下被重新拆成一组独立远程组件。

`Cloud-compatible` 的真正含义是：

- `Harness` 可以整体部署到远端
- `Harness` 调用其它核心模块的 binding 可以从 direct-call 变成 RPC / remote service call
- 但 harness 自己的内部语义和模块边界不应变化

## Local 与 Cloud 的真实差异

### Local

`Local` 通常意味着：

- harness 在本机运行
- 与 `Session`、`Tools`、`Sandbox` 常见是 direct-call 或本地进程调用
- 默认以单机装配为主

`Local` 不要求单进程，但常见优化是：

- direct-call
- in-process cache
- local process / local IPC

### Cloud

`Cloud` 通常意味着：

- harness 整体部署为远端 worker / service
- `Session`、`Tools`、`Sandbox`、`Orchestration` 可以通过 RPC 或 remote handle 调用
- orchestrator 决定 harness 的唤醒、重建、路由和替换

但 `Cloud` 不应被理解成：

- harness 被拆碎成一堆远程小组件
- runtime / context / task / permission 各自变成独立顶层服务

## 稳定边界

无论 `Local` 还是 `Cloud`，以下边界都必须保持一致：

- `Harness` 负责 turn runtime、context engineering、task、permission、multi-agent 等运行时能力
- `Session` 负责 durable transcript、checkpoint、resume、session memory
- `Tools` 负责 capability declaration、tool execution、MCP、skills
- `Sandbox` 负责 execution boundary 与 capability/security model
- `Orchestration` 负责 cloud control plane、wake、reprovision、hosting profile

部署位置可以变化，语义所有权不能变化。

## Binding 是可变的，不是语义层级

`Harness` 与其它模块之间的 binding 可以变化：

- Local:
  - function call
  - local process call
  - local handle
- Cloud:
  - RPC
  - remote handle
  - service call

但是这些变化只应影响：

- latency
- deployment topology
- failure domain
- retry/transport 实现细节

不应影响：

- turn semantics
- terminal state
- tool continuation
- requires_action semantics
- task ownership
- context assembly ownership

## 对各模块关系的约束

### Harness vs Gateway

- `Gateway` 负责外部 chat/channel 接入
- `Harness` 负责内部运行时整体
- 在 `Cloud` 下，gateway 可以与远端 harness 交互
- 但不是去调用 harness 的“部分子组件”

### Harness vs Session / Tools / Sandbox

- `Harness` 依赖这些模块
- `Local` 下可以 direct-call
- `Cloud` 下可以 RPC 调用
- 这种变化不应把 harness 内部能力重新分类成外部服务

### Harness vs Orchestration

- `Orchestration` 决定 harness 的部署、唤醒、替换、重建
- 但不吞并 harness 的运行时语义
- `Cloud` 不是 orchestration 直接替代 harness

## Core Contract 与 Convenience API

`Local` 与 `Cloud` 不应演化成两套不同的 harness 语义。

更稳妥的约束是：

- 只有一套 core semantic contract
- Local 下允许 direct-call convenience API
- Cloud 下允许 remote binding
- 但 direct-call convenience API 仍必须可由同一核心语义严格推导

这意味着：

- `run_turn()` 不能因为是本地函数调用就拥有额外状态机
- `Cloud` 下的远端调用也不能弱化 turn/stream/terminal semantics

## Default Local Mapping

默认本地实现通常表现为：

- harness 与 session / tools / sandbox 同机协作
- 调用方式多为 direct-call
- task、verification、multi-agent 也在 harness 内部运行

这只是默认映射，不构成规范前提。

## Cloud-Compatible Mapping

cloud 形态下应理解为：

- harness 整体远端部署
- runtime / context engineering / task / permission / multi-agent 仍在 harness 内部
- harness 通过 remote binding 调用 `Session`、`Tools`、`Sandbox`
- orchestration 为 harness 提供 wake / reprovision / route

## 规范结论

- `Harness` 在 `Local` 和 `Cloud` 下都应被视为一个完整整体
- `Cloud-compatible` 不等于 harness 内部服务拆分
- 真正变化的是 harness 调用其它模块的 binding
- `Local` 下 direct-call、`Cloud` 下 RPC 都只是部署映射
- deployment topology 可以变化，但模块边界和运行时语义不应变化
