# Reference Memory

## 职责

`ReferenceMemory` 定义 durable memory 中的外部入口层。

它负责保存：

- 去哪里找外部信息
- 外部资源的用途

而不是外部系统当前内容本体。

## 核心语义

- `ReferenceMemory` 是 pointer memory，而不是事实快照
- 它应帮助 future recall 找到 authoritative external source
- 它通常更适合 shared/team scope

## 与其它类型的边界

- 与 [project-memory.md](project-memory.md)
  `ReferenceMemory` 记外部入口；`ProjectMemory` 记项目背景与动机
