# Runtime

## 职责

`Runtime` 是 `Harness` 内部直接负责 turn 推进、hook 执行、状态投影和 turn 结束后处理的子域。

它回答的不是“整个 harness 是什么”，而是：

- turn state machine 如何运行
- 内部消息、外部事件与终止态如何统一
- lifecycle hooks 如何插入 runtime
- runtime state、progress、trace、系统事件如何投影
- turn stop 之后的后处理如何触发

## 为什么不再叫 `runtime-core`

当前这一组内容已经不只包含 core loop。

除了 turn state machine 本身，源码里还存在几条稳定的 runtime 子平面：

- `core`
  turn state machine、terminal state、message pipeline、state layering
- `hooks`
  lifecycle hook runtime
- `projection`
  observability 与 runtime event transport
- `post-turn`
  stop boundary 之后的处理面

因此这里更适合统一称为 `Runtime`，而不是 `Runtime Core`。

## 子域分层

### Core

- [core/agent-runtime.md](core/agent-runtime.md)
- [core/ralph-loop.md](core/ralph-loop.md)
- [core/failure-and-terminal-states.md](core/failure-and-terminal-states.md)
- [core/state-layering.md](core/state-layering.md)
- [core/message-and-event-pipeline.md](core/message-and-event-pipeline.md)

这一组回答：

- runtime lifecycle、session binding 与 turn loop 如何分层
- turn-local loop 如何推进
- terminal state、error class、retryability 如何统一
- runtime 内部状态如何分层
- normalized ingress、internal message、stream event 如何分开
- 为什么 chat/channel 语义不属于 runtime core，而属于 gateway

### Hooks

- [hooks/hook-runtime.md](hooks/hook-runtime.md)

这一页回答：

- lifecycle hooks 如何注册、匹配、执行和投影
- hook runtime 与 tools、permission、session 的边界是什么

### Projection

- [projection/runtime-observability.md](projection/runtime-observability.md)
- [projection/runtime-state-projection.md](projection/runtime-state-projection.md)

这一组回答：

- runtime 的进度、usage、latency、trace 如何被观测
- 哪些 runtime-visible facts 应被对外投影
- observability 与 state projection 的职责如何分层

### Post-Turn

- [post-turn/post-turn-processing.md](post-turn/post-turn-processing.md)

这一页回答：

- turn 结束后哪些处理属于 runtime
- stop hook、prompt suggestion、memory extract、dream 等如何触发
- 为什么 post-turn processing 不是 core loop、不是 verification、也不是 task runtime

## 设计结论

- `Runtime` 应被视为 `Harness` 的主运行时子域
- `core / hooks / projection / post-turn` 应显式分层
- `extension-and-projection` 这类泛目录不适合继续保留
