# Sidechain And Subagent Transcripts

## 职责

`Sidechain And Subagent Transcripts` 定义 session 的 branch tracing 扩展层。

它记录主 session 之外、但仍附属于该 session 的 durable conversation branches。

它不负责：

- cross-session durable memory
- task output storage
- mailbox routing
- gateway projection

## 最小对象

推荐最小对象：

```text
BranchRef
  - branch_id
  - parent_session_id
  - branch_kind
  - transcript_ref
  - metadata_ref?
```

```text
SidechainTranscript
  - branch_ref
  - transcript_entries
  - parent_linkage
  - created_at
```

`branch_kind` 至少应允许：

- `subagent`
- `sidechain`
- `remote_session`

## 稳定语义

### 1. session branch tracing, not another memory store

sidechain / subagent transcript 仍然属于 session 的历史层。

它们不应被建模成：

- another durable memory plane
- task-only artifact
- unrelated nested session system

### 2. parent linkage must be durable

主 session 与 branch transcript 的关系必须可追溯。

实现至少应能表达：

- parent session
- branch kind
- transcript ref
- optional sidecar metadata

### 3. main transcript and sidechain transcript must be distinguishable

恢复主会话时，系统必须能够区分：

- main transcript chain
- sidechain / subagent leaf

否则 resume 可能错误地把 sidechain leaf 当成主链末端。

### 4. restore must preserve branch linkage

resume / wake 时，如果 session 拥有 child refs 或 branch refs，
这些 linkage 应作为 `ResumeSnapshot` 或等价对象的一部分恢复。

branch transcript 可以不直接进入当前 prompt window，
但它的追溯关系不得丢失。

## 与相邻页面的边界

- 与 [transcript.md](transcript.md)
  本页定义 branch tracing；transcript 总边界见该页
- 与 [resume-and-restore.md](resume-and-restore.md)
  本页定义 branch linkage 的 durable semantics；恢复要求见该页
- 与 [../durable-memory/README.md](../durable-memory/README.md)
  sidechain transcript 属于 session branch history，不属于 cross-session durable knowledge

## 规范结论

- `SidechainTranscript` / `BranchRef` 应作为 `Session` 的独立 durable layer 建模
- sidechain / subagent transcript 不得退化成 task output 或 durable memory 的替代物
- 恢复主 session 时，branch linkage 必须可追溯且主链可与 sidechain 区分
