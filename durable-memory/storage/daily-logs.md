# Daily Logs

## 职责

`DailyLogs` 定义 append-first operating mode 下的 capture stream。

它负责：

- 先承接新产生的 durable signal
- 为后续 extract / dream 提供近期 capture source
- 允许 long-lived assistant-style host 先追加、后蒸馏

它不负责：

- resident index
- topic payload 的最终 durable shape
- bounded recall 的常驻入口

## 核心结论

daily logs 不是最终 typed memory 载体。

它们只表示：

- append-first capture stream
- 按时间追加的近期 durable signals
- 后续 distill / merge / dedupe 的上游输入

## 与其它页面的边界

- 与 [memory-md.md](memory-md.md)
  `MEMORY.md` 是 resident index；daily logs 不是 resident index
- 与 [topic-memory.md](topic-memory.md)
  topic memory 是最终 durable payload；daily logs 只是上游 capture stream
- 与 [../operations/direct-memory-write.md](../operations/direct-memory-write.md)
  append-first variant 可以先写 daily logs，再延后 consolidation
- 与 [../operations/auto-dream.md](../operations/auto-dream.md)
  dream 可以把 daily logs 蒸馏成 topic payload 和 tightened index

## 规范结论

- daily logs 只是一种 capture stream
- 它们不等于 durable payload
- append-first operating mode 只改变写入路径，不改变 durable memory 的所有权和分层
