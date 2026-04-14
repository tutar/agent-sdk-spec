# Case: Prompt Cache Dynamic Suffix

## 目标

验证动态上下文只影响 suffix，不污染 cacheable prefix。

## Preconditions

- runtime 支持 `PromptCacheStrategy`
- 至少存在一类高波动上下文，例如：
  - session guidance
  - dynamic attachment
  - runtime delta
  - late-bound integration instruction

## Ingress

1. 运行一次基线 turn
2. 注入动态上下文变化
3. 再运行一次 turn

## Expected Semantics

- stable prefix 保持不变
- dynamic suffix 发生变化
- 变化不应通过重写 bootstrap prompt 静态部分来实现

## Expected Persistent Effects

- session / transcript 正常追加
- 不应因为 suffix 变化而触发对静态前缀的整体重建

## Failure Conditions

- 动态上下文写进 cacheable prefix
- suffix 变化导致 prefix hash 变化
- 动态上下文被错误建模成 transcript 历史改写
