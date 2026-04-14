# Case: Desktop Bundle Lifecycle

## 目标

验证桌面宿主下 bundle/package 型扩展的安装、校验、加载、停用语义。

## Preconditions

- host profile 为 `Desktop`
- runtime 支持 bundle/package 型扩展，例如 MCP bundle 或等价本地封装
- 桌面端具备本地 install / verify / enable / disable 机制

## Ingress

1. 安装一个 bundle
2. 校验并加载该 bundle
3. 使用其暴露的能力
4. 停用或卸载该 bundle

## Expected Lifecycle

- bundle 从 `installed -> verified -> active -> inactive` 或等价状态推进
- bundle 生命周期不应与 session lifecycle 混淆

## Expected Runtime Semantics

- bundle 激活后，其能力进入正确的 domain：
  - tool
  - skill
  - mcp
- bundle 校验失败时，不应进入 active state
- bundle 停用后，新的 turn 不应继续暴露其能力

## Expected Persistent Effects

- 桌面端本地 host 保存 bundle 安装状态
- 启动新 session 时，active bundle 状态可被重新装配

## Failure Conditions

- bundle 安装后未经校验直接进入 active
- 停用后能力仍暴露给 harness
- bundle lifecycle 被错误写入 session transcript
