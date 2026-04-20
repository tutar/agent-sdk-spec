# Case: AGENTS Memory Loading Precedence

## 目标

验证 `AGENTS.md` 作为 file-backed durable memory injection 的加载顺序与作用域语义。

## Preconditions

- session / context assembly 支持 `AGENTS.md` memory injection
- 存在 `~/.openagent/AGENTS.md`
- 工作目录存在 `AGENTS.md`
- 目标子目录存在 `AGENTS.md`

## Ingress

1. 在目标位于该子目录时发起一轮 turn
2. 组装当前 turn 的 memory injection
3. 观察最终注入顺序与来源

## Expected Runtime Semantics

- 加载顺序为 `~/.openagent/AGENTS.md -> workdir/AGENTS.md -> subtree/AGENTS.md`
- 最终结果采用累积加载
- 冲突内容以后加载者为准
- sibling 子目录的 `AGENTS.md` 不应进入当前 target 的注入结果
- 注入结果进入 context plane，而不是 transcript
- 该注入语义独立于 durable memory recall，不要求把 `AGENTS.md` 先转成 durable memory record

## Expected Persistent Effects

- transcript / event log 不应记录伪造的 `AGENTS.md` message
- resume 后，宿主可重新解析 `AGENTS.md` 或使用语义等价缓存

## Allowed Variance

- 实现可以缓存已解析的 `AGENTS.md`
- 实现可以用结构化 context attachment 或等价方式承载注入结果

## Failure Conditions

- 优先级顺序不稳定
- sibling 子目录污染当前目标上下文
- `AGENTS.md` 被写入 transcript 伪装成普通消息
- 缺失某一层 `AGENTS.md` 导致 session restore 语义失效
