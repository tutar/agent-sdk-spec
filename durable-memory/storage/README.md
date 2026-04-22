# Durable Memory Storage

`storage/` 定义 durable memory 的主要落地形态。

这一组回答：

- resident index 落在哪里
- durable payload 落在哪里
- append-first capture stream 如何表示
- recall candidate metadata 如何暴露

固定分层为：

- [memory-md.md](memory-md.md)
  resident index
- [topic-memory.md](topic-memory.md)
  typed memory 的主要 durable payload 载体
- [daily-logs.md](daily-logs.md)
  append-first capture stream
- [frontmatter-and-header-manifest.md](frontmatter-and-header-manifest.md)
  recall manifest / header metadata

这几层必须分离：

- resident index 不是 payload
- daily logs 不是最终 typed memory
- header manifest 不是 payload truth source
