# Case: Prompt Cache Strategy Equivalence

## 目标

验证不同 prompt cache strategy 对上层暴露一致语义。

## Preconditions

- 至少存在两种策略实现：
  - `Anthropic Native Strategy`
  - `OpenClaw-Mediated Strategy`
  或其中一种加 `Fallback Strategy`

## Procedure

在不同策略下运行以下子场景：

- `prompt-cache-stable-prefix`
- `prompt-cache-dynamic-suffix`
- `prompt-cache-fork-sharing`
- `prompt-cache-break-detection`

## Expected Equivalence

不同策略必须对外保持以下语义一致：

- stable prefix 的定义
- dynamic suffix 的定义
- fork cache sharing 语义
- cache break 的结构化分类

## Allowed Variance

允许不同策略使用不同 provider 字段、不同 cache backend、不同 usage 计量方式。

不允许不同策略改变：

- 上层 runtime event 语义
- conformance case 的通过条件
- host-neutral 的外部行为

## Failure Conditions

- Anthropic-native 语义与 OpenClaw-mediated 语义不能对齐
- fallback strategy 对外暴露不同的 break 分类
- 策略切换要求上层业务同时修改语义判断
