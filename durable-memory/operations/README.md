# Durable Memory Operations

`operations/` 定义 durable memory 的运行链。

这一组固定回答：

- store 如何被扫描
- runtime 如何做 bounded selection
- foreground 如何直接写入 durable memory
- turn 结束后如何补提取 durable signal
- cross-session dream 如何做 consolidation

固定操作链为：

- [memory-scan.md](memory-scan.md)
- [relevant-memory-selection.md](relevant-memory-selection.md)
- [direct-memory-write.md](direct-memory-write.md)
- [extract-memories.md](extract-memories.md)
- [auto-dream.md](auto-dream.md)

这五条链必须可区分：

- scan != selection
- direct write != extract
- extract != dream
