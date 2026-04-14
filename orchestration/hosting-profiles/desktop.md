# Desktop Hosting Profile

## 目标

`Desktop Hosting Profile` 描述桌面端 agent host 的标准职责分布。

典型形态包括：

- 本地 GUI + agent daemon
- 桌面应用内嵌 agent runtime
- 本地 MCP bundle / extension host

## 角色分布

在 Desktop 场景下，推荐的职责分布是：

- `ChannelAdapter`
  GUI 事件、桌面通知、文件拖拽、系统集成
- `IngressGateway`
  UI 事件标准化、session binding、control routing
- `Harness`
  本地 daemon 或嵌入式 runtime 中的 turn evaluation
- `Session`
  本地 durable session store
- `Orchestration`
  本地 daemon task orchestration + UI/runtime 分工
- `Sandbox`
  同时消费 `Execution Sandbox` 与 `Environment Sandbox`

## 典型特征

- UI 进程与 runtime 进程常常分离
- permission / approval 可能通过 GUI 弹窗完成
- bundle install / update / load 是重要的本地生命周期问题
- 本地资源访问比云端更丰富，但安全边界也更复杂

## 对 orchestration 的要求

- 支持 UI host 与 runtime host 的分工
- 支持本地 daemon 的任务生命周期
- 支持桌面通知、后台任务和恢复
- 支持 MCP bundle / local extension 生命周期进入标准控制面
- 支持 GUI 关闭后 session 继续或后台运行

## 与 sandbox 的关系

Desktop profile 通常比 TUI 更需要显式表达：

- 本地进程级 sandbox
- 本地 workspace/container 级 sandbox
- 本地 credential boundary

因此它通常是最需要同时使用 `Execution Sandbox` 和 `Environment Sandbox` 的 profile。

## 规范结论

- Desktop profile 应把 GUI host、runtime host、local daemon 的职责边界写清
- 在该 profile 中，orchestration 不只是 agent/task，还要覆盖本地 host lifecycle
