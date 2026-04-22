# Working State

## 职责

`WorkingState` 定义 `Session` 中可恢复的 continuation runtime context。

runtime 在 turn 内推进它、在 restore 时重建它，但其 durable ownership 仍在 `Session`。

它不负责：

- durable transcript truth
- observer-facing lifecycle state
- cross-session recall
- task / gateway projection

## 最小对象

推荐最小对象：

```text
WorkingState
  - active_messages?
  - pending_action_binding?
  - tool_execution_state?
  - content_replacements?
  - working_view_projection?
  - compact_boundary_ref?
  - continuation_inputs?
```

## 稳定语义

### 1. continuation runtime state

working state 关心的是：

- 当前 turn / continuation 如何继续
- compact 后当前 working set 是什么
- pending action 如何绑定回运行态
- tool result replacement / projection 如何承接

它描述的是“如何继续”，而不是“历史里发生过什么”。

### 2. not transcript

transcript 记录 durable history；
working state 则记录恢复 continuation 所需的结构化状态。

因此 working state 不得通过扫描 transcript 文本临时拼装来替代 durable restore。

### 3. not lifecycle projection

`idle / running / requires_action` 是 observer-facing projection。

working state 则至少需要表达：

- pending action binding
- active working window
- content replacement / externalization state
- collapse / projection state

二者可以存于同一实现中，但语义上必须区分。

### 4. compact 与 projection 会重写 working state

compact、working-view projection、content externalization 不一定改写 transcript，
但会改变当前 continuation 所依赖的 working set。

因此 working state 必须支持：

- compact-aware continuation
- projection-aware restore
- content-replacement-aware resume

### 5. restore 的目标之一是重建 working state

resume / wake 成功的标准之一是：

- session 被恢复成可继续运行的 working state

而不是仅恢复出一条可显示的历史消息序列。

## 与相邻页面的边界

- 与 [transcript.md](transcript.md)
  transcript 是 durable truth source；working state 是 continuation context
- 与 [resume-and-restore.md](resume-and-restore.md)
  本页定义 working state 是什么；resume-and-restore 定义如何恢复它
- 与 [lifecycle-state.md](lifecycle-state.md)
  lifecycle state 是 observer-facing projection，不是 continuation state
- 与 [../harness/runtime/core/ralph-loop.md](../harness/runtime/core/ralph-loop.md)
  runtime turn loop 使用 working state，但不拥有它的 durable boundary

## 规范结论

- `WorkingState` 必须作为 `Session` 的独立层建模
- working state 不得退化成 transcript replay、lifecycle projection 或 short-term summary
- restore 成功的标准之一是 working state 可被重新建立
