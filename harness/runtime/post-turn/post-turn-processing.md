# Post-Turn Processing

## 职责

`PostTurnProcessing` 负责在一次 turn 结束后，执行不属于主推理链但会影响后续行为的加工流程。

典型职责包括：

- stop hooks
- prompt suggestion
- session-level memory extraction trigger
- dream / consolidation trigger
- task completed / teammate idle post-processing

规范上要明确：

- post-turn processing 不属于主模型推理
- 它应在 turn terminal 或 stop 阶段触发
- 它可以异步执行，但必须有明确触发边界

## 稳定接口

```text
PostTurnContext
  - session_ref
  - turn_ref
  - messages
  - system_prompt
  - user_context
  - system_context
  - tool_use_context
  - query_source

PostTurnProcessor
  - should_run(context) -> boolean
  - run(context) -> post_turn_events

PostTurnRegistry
  - list_processors()
  - register(processor)
  - execute_all(context)
```

## 要解决的问题

- 如何把 turn 结束后的加工逻辑从主推理回路中拆出来
- 如何给 memory extraction、dream、hooks 提供统一触发平面
- 如何防止这些后处理逻辑污染主 query loop
- 如何在同步返回与后台任务之间平衡时延与完整性

## 与其它模块的边界

- 不是 core loop
- 不是 verification
- 不是 compact
- 不是 task runtime

它是 runtime 在 turn terminal / stop boundary 上的独立阶段。

turn-local loop 如何进入 terminal 或 handoff boundary，见 [../core/ralph-loop.md](../core/ralph-loop.md)。

## 规范结论

- post-turn processing 应作为 runtime 的独立子平面存在
- 它应由明确的 turn 边界触发，而不是散落在业务逻辑中
- post-turn processor 可以异步，但触发点和事件投影必须稳定
- post-turn processing 不是 verification，也不是 compact
