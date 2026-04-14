# Case: Prompt Cache Stable Prefix

## 目标

验证 harness 能稳定构造 cacheable prefix，而不是每轮重组出不同前缀字节。

## Preconditions

- runtime 启用了某种 `PromptCacheStrategy`
- bootstrap prompt 与 structured context 已分层
- session 中存在至少两轮共享同一静态前缀的 turn

## Ingress

1. 发起第一轮普通 turn
2. 发起第二轮只改变动态后缀的 turn

## Expected Semantics

- 两轮 turn 的 stable prefix 必须等价
- 第二轮的变化只能体现在 dynamic suffix
- 实现可以使用 provider-native prompt cache、OpenClaw-mediated cache 或 fallback strategy

## Expected Runtime Effects

- 若 runtime 暴露 prefix hash 或等价可观测信息，两轮 stable prefix 应相同
- 若 runtime 暴露 cache policy，两轮 cache policy 应保持一致

## Allowed Variance

- provider 可决定是否实际命中缓存
- usage 数值和延迟不参与本 case 合规判断

## Failure Conditions

- 动态信息被错误混入 stable prefix
- 同一静态上下文在两轮生成不同 prefix bytes
- cache-critical 配置在无必要时发生漂移
