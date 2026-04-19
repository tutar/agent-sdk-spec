# Runtime Observability

## 职责

`RuntimeObservability` 定义 runtime 必须如何暴露运行状态、进度、usage、cost、latency 和 trace 信息。

它负责：

- turn 级 runtime metrics
- session lifecycle projection
- task / background agent progress projection
- tracing span 生命周期
- 对外 agent / IDE / dashboard 的状态投影

它不负责：

- 最终正确性判定
- 业务质量评分
- 具体 exporter / vendor 绑定

## 稳定接口

```text
RuntimeObservability
  - emit_runtime_metric(metric)
  - emit_progress(update)
  - emit_session_state(state)
  - start_span(type, attributes) -> span_handle
  - end_span(span_handle, result)
  - project_external_event(event)
```

## 规范要求

### 1. 必须支持三层可观测性

至少应区分：

- session-level signal
- task / background progress
- span-based tracing

### 2. session lifecycle 必须显式投影

最小状态至少包括：

- `idle`
- `running`
- `requires_action`

### 3. 后台 agent 必须支持 progress projection

至少应允许投影：

- token count
- tool use count
- duration
- last activity
- summary

### 4. 模型请求与工具调用必须可追踪

至少应能观测：

- input / output tokens
- cache tokens
- duration
- ttft
- success / error

## 与 `runtime-state-projection` 的边界

- `RuntimeObservability`
  定义“观测什么”
- `RuntimeStateProjection`
  定义“哪些 runtime-visible facts 对外暴露，以及这些 facts 如何与 transcript / task registry 分离”

## 规范结论

- runtime 必须有独立 observability 平面
- session signal、task progress、span tracing 必须分层
- observability 只回答“发生了什么”，不直接承担“是否通过”的评估职责
