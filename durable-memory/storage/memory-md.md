# MEMORY.md

## 职责

`MEMORY.md` 是 resident entrypoint / index 的默认本地映射。

它负责：

- 提供低成本常驻入口
- 列出 durable topics 的 pointer / hook / summary
- 为 runtime 提供 orientation，而不是完整 payload

它不负责：

- durable payload 本体
- bounded selection algorithm
- transcript restore

## 核心结论

resident entrypoint index 应被视为 durable memory 的常驻入口层，而不是 durable payload。

它至少应满足：

- 常驻进入 context plane
- 低成本、可预算
- 列出 durable topic 的 pointer / hook / summary
- 不承载完整 durable payload

## 必须保持的语义

### 1. index is not payload

resident entrypoint index 必须与 durable payload 层分离。

### 2. resident and budgeted

resident index 进入 context plane 时，应保持低成本和预算可控。

### 3. distilled index is allowed

某些 long-lived host profile 可把 resident index 变成 delayed / distilled index：

- 新信号先进入 append-first capture stream
- resident index 稍后由 dream 更新

## 与其它页面的边界

- 与 [topic-memory.md](topic-memory.md)
  topic memory 是 durable payload；本页是常驻入口层
- 与 [frontmatter-and-header-manifest.md](frontmatter-and-header-manifest.md)
  header manifest 是 recall candidate layer；本页是 resident orientation layer
- 与 [../operations/direct-memory-write.md](../operations/direct-memory-write.md)
  direct write 可以更新 resident index，但 resident index 不是 direct write 的唯一 target
- 与 [../operations/auto-dream.md](../operations/auto-dream.md)
  dream 可以 tighten resident index，但 dream 不改变其 entrypoint 角色

## 规范结论

- `MEMORY.md` 只表达 resident index 的默认映射
- resident index 必须与 payload 分层
- resident index 可以被 delayed / distilled maintenance 更新
