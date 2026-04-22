# Case: Instruction Rules And Include Expansion

## 目标

验证 conditional instruction rules 与 `@include` / 语义等价 include expansion 作为 harness-level instruction loading 子机制的稳定语义。

## Preconditions

- harness / context assembly 支持 rules 或语义等价的 conditional instruction source
- 宿主支持 instruction include expansion 或语义等价机制
- 至少存在一条 unconditional rule、一条 conditional rule、以及一个被主 instruction file 引用的 include target

## Ingress

1. 准备一个主 instruction markdown source，其中包含 include ref
2. 准备一组 rules，其中一部分带 `paths` 或语义等价条件字段
3. 发起一轮针对特定 target path 的 turn
4. 观察 include 是否在主文件解析时展开，以及哪些 rules 会进入最终 instruction result

## Expected Runtime Semantics

- include expansion 应在主 instruction source 解析时发生，而不是独立后台扫描
- include expansion 应允许递归深度限制、循环去重和外部路径安全边界
- unconditional rule 应总是可进入候选集
- conditional rule 应基于 target path 或语义等价 selector 匹配，而不是仅按 cwd 文本匹配
- frontmatter / 条件字段应控制加载与匹配，而不要求作为正文进入模型输入
- 这些语义属于 harness-level instruction loading，而不是 durable memory recall

## Expected Persistent Effects

- include target 不应被写回 transcript 伪装成普通消息
- rule matching 不应改写 durable memory store 来模拟成功
- conditional rule 的匹配结果可以缓存，但缓存失效不应破坏 session restore

## Allowed Variance

- 条件字段可以是 `paths` 或语义等价的 target selector
- include 语法可以是 `@include` 或语义等价机制
- include target 可以作为独立 source entry 或主 source 的 expanded fragment 注入

## Failure Conditions

- include 只靠全局预扫描生效，而不是随主文件解析展开
- conditional rule 不能表达 target-path applicability
- sibling / unrelated subtree rule 污染当前 target
- 将 rules 或 include expansion 混成 durable memory recall
