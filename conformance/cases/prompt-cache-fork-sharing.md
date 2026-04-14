# Case: Prompt Cache Fork Sharing

## 目标

验证 fork/subagent 在 cache-sharing 模式下继承 cache-critical 参数。

## Preconditions

- runtime 支持 forked subagent 或 side-query
- parent turn 已构造完成 cacheable prefix
- runtime 启用了 `PromptCacheStrategy`

## Ingress

1. 发起一个父 turn
2. 从该 turn fork 出 subagent / side-query

## Expected Semantics

- fork child 应继承 parent 的 cache-critical 参数
- 若实现支持 `skip_cache_write` 或等价语义，应能显式控制是否写入新缓存
- fork child 可以增加自己的动态 suffix，但不应破坏共享前缀

## Expected Runtime Effects

- fork 关系中应能观察到：
  - shared parent prefix
  - inherited cache-critical parameters
  - optional cache-write suppression

## Failure Conditions

- fork child 擅自变更 model identity、TTL、tool surface 等关键参数
- fork child 因无关配置漂移而失去共享前缀能力
- parent / child 的缓存共享语义不可观察、不可解释
