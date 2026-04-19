# Runtime State Projection

## 职责

`RuntimeStateProjection` 定义 `Harness.Runtime` 如何把内部可观察事实投影给外部观察者。

它回答的问题是：

- 哪些 runtime-visible facts 需要对外暴露
- 哪些事实属于 runtime 自身，哪些来自 task 等子系统
- 这些事实如何与 transcript、message stream、task registry 分离
- 外部 host、gateway、bridge、observer 应如何消费这些事实

## 为什么这里不再叫 `Transport`

规范真正需要稳定的不是某一种队列或 SDK 通道，而是：

- 哪些事实应被对外投影
- 这些事实与其它平面的边界
- 对外消费者看到的语义是否稳定

实现可以用不同方式暴露这些投影：

- event stream
- queue + drain
- callback / subscribe
- polled status surface
- bridge message

`transport` 只是默认实现映射；规范层真正稳定的是 `projection` 语义。

## 投影对象

### Session / Turn State Projection

runtime 至少应允许对外投影：

- `running`
- `idle`
- `requires_action`
- turn terminal bookends

这些对象描述的是 turn 级运行事实，而不是 durable transcript。

### Task-Derived Projection

runtime 也可以对外投影由 task 子系统派生出来的 observer-facing facts，例如：

- `task_started`
- `task_progress`
- `task_notification`

这些投影用于让外部观察者知道：

- 后台执行对象是否已经开始
- 当前是否仍在推进
- 是否已经进入 terminal bookend

### Runtime Boundary Signals

runtime 还可以投影由 runtime 自己控制的边界信号，例如：

- `requires_action`
- `compact_boundary_created`
- 其它 stop / resume / turn-boundary system signal

## 与 Task 的关系

这一页最重要的边界是：

- `Harness.Task`
  拥有 task lifecycle 的 source of truth
- `Harness.Runtime`
  拥有 observer-facing state projection

也就是说：

- task state 存在于 `TaskRegistry`
- task 的标准状态机、输出句柄、terminal dedupe 由 `Harness.Task` 定义
- runtime 不拥有 task 状态机
- runtime 只负责把 runtime-managed async work 的关键事实投影给外部观察者

因此 task 事件出现在 runtime projection 中是合理的，但这不改变 task 的所有权。

### Ownership Boundary

`Harness.Task` 负责：

- task registration
- task lifecycle state
- output handle / cursor
- terminal notification dedupe
- retention / eviction semantics

`Harness.Runtime` 负责：

- task-derived fact 的外部可见投影
- 这些投影与 transcript / message stream 的分离
- observer-facing publication timing
- session / turn / task facts 的统一对外暴露面

## 稳定接口

```text
RuntimeStateProjection
  - project(fact)
  - attach_observer(observer) -> subscription?
  - list_channels()?
```

推荐直接复用 [../../../object-model.md](../../../object-model.md) 中的 `RuntimeEvent` 作为事件型投影对象。

约束：

- projection 语义必须独立于 transcript
- projection 语义必须独立于主消息流
- task-derived projection 不得被误当作 task registry 本体
- event stream 只是实现映射，不是唯一标准接口

## 与其它 runtime 页面边界

- [../core/message-and-event-pipeline.md](../core/message-and-event-pipeline.md)
  负责 internal message、stream event、external runtime event 的分层
- [runtime-observability.md](runtime-observability.md)
  负责 metrics、trace、progress 等“观测什么”的语义
- 本页
  负责哪些 runtime-visible facts 应对外投影，以及这些 facts 与 task 的边界

## Default Local Mapping

默认本地实现通常会同时存在两类 outward channels：

- 结构化 system-event 输出
- 内部 notification queue 经 runtime 转投影

在这种映射里：

- runtime 自己可以直接投影 session / turn state
- task 子系统先产生 lifecycle facts
- runtime 再把这些 facts 变成 observer-facing projection

这是一种常见本地实现，但规范不要求其它 host 也必须使用 queue + drain。

## 规范结论

- runtime 应有独立的 state projection 平面
- `task_started / task_progress / task_notification` 可以作为 runtime projection surfaces 暴露
- task-derived projection 不等于 task lifecycle ownership
- projection 必须与 transcript、message stream、task registry 显式分离
- 外部观察者可以通过事件流、状态 API 或语义等价机制消费这些 projection
