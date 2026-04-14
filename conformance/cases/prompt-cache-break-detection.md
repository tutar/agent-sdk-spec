# Case: Prompt Cache Break Detection

## 目标

验证 runtime 能把 cache break 识别成结构化语义，而不是隐性退化。

## Preconditions

- runtime 支持 cache usage、prefix hash、或其他等价可观测指标
- `PromptCacheStrategy` 支持 cache break detection

## Ingress

执行一轮会触发 cache break 的变更，例如：

- cache TTL 改变
- prompt bytes 改变
- tool surface 改变
- cache scope 改变

## Expected Semantics

- runtime 能区分：
  - expected cache miss
  - unexpected cache break
- cache break 应带有结构化原因分类

## Expected Runtime Effects

若实现提供观测事件，应至少能暴露：

- break detected
- reason
- previous baseline
- current baseline

## Allowed Variance

- 不同实现可使用 usage token、hash、或 prefix metadata 进行判断
- 具体阈值可不同，但外部 break classification 语义应一致

## Failure Conditions

- cache break 只能靠日志文本人工推断
- expected miss 与 unexpected break 混为一谈
- break 原因不可分类
