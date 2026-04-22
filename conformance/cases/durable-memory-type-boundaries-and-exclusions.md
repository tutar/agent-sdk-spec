# Case: Durable Memory Type Boundaries And Exclusions

## 目标

验证 durable memory taxonomy 的类型边界稳定，并且 “what not to save” 的排除规则不会被误归入 `user / feedback / project / reference` 四类。

## Preconditions

- durable-memory package 支持 durable memory taxonomy 或语义等价分类
- durable-memory package 支持至少 `user / feedback / project / reference` 四类中的语义等价边界

## Ingress

1. 提供一组可被保存的 durable memory 候选：
   - 用户画像
   - 行为 guidance
   - 非代码态项目背景
   - 外部信息入口
2. 再提供一组不应被保存为四类 taxonomy 的候选：
   - code patterns
   - architecture / file structure
   - git history
   - ephemeral task details
3. 触发 durable memory 写入或分类流程

## Expected Runtime Semantics

- `user / feedback / project / reference` 四类必须语义可区分
- code / git / ephemeral task details 不得被误存为以上任一类 durable memory
- taxonomy 的作用是 durable payload 分类，而不是定义新的 memory plane
- 分类结果必须能支持后续 recall / consolidation 继续保持同一类型边界

## Expected Persistent Effects

- 合法的 durable candidate 会以正确类型写入 durable store
- 被排除的候选不会因为“显式要求保存”而自动绕过 taxonomy 边界
- consolidation 后，非法类型内容不应通过 merge 被重新混入 taxonomy records

## Allowed Variance

- 实现可以使用不同字段名表达类型
- 实现可以在写入前做 rules-based、prompt-based 或 hybrid classification
- 非法候选可以被拒绝、降级为不保存、或要求用户进一步提炼“真正值得记住的部分”

## Failure Conditions

- code pattern 被写成 `project` 或 `feedback` durable memory
- git history 被写成 durable taxonomy record
- 当前 turn 临时任务状态被写成 `project` memory
- taxonomy 边界只体现在文档中，运行时无法区分
