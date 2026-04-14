# SDK Event Transport

## 职责

`SdkEventTransport` 负责把 runtime 内部的关键状态变化投影给外部 SDK 客户端。

它独立于主消息流，面向：

- session lifecycle
- task lifecycle
- bg task / subagent UI
- turn completion / idle detection

## 稳定接口

```text
SdkEventTransport
  - publish(event)
  - drain() -> events

SdkRuntimeEvent
  - type
  - subtype
  - session_id
  - uuid
  - payload
```

推荐事件：

- `task_started`
- `task_progress`
- `task_notification`
- `session_state_changed`

## 设计要求

- 不能与 transcript 混用
- 不能与模型消息流混用
- 允许多个子系统写入
- 允许外部客户端单独消费
- 默认可为 best-effort queue，但语义应稳定

## 默认实现映射

- 队列实现见 [utils/sdkEventQueue.ts](../../../cc/utils/sdkEventQueue.ts)
- session state 投影见 [utils/sessionState.ts](../../../cc/utils/sessionState.ts)
- task progress / lifecycle 投影见 [utils/task/framework.ts](../../../cc/utils/task/framework.ts) 和 [utils/task/sdkProgress.ts](../../../cc/utils/task/sdkProgress.ts)

## 规范结论

- SDK client 应有独立系统事件通道
- runtime state projection 不应强塞进主消息流
- 本地 direct-call API 可以聚合这些事件，但不应创造事件通道之外的独有语义
