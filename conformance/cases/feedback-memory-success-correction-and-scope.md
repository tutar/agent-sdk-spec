# Case: Feedback Memory Success Correction And Scope

## 目标

验证 `FeedbackMemory` 同时覆盖 correction 与 validated success，并且在 shared/private split 下保持正确的 scope 语义。

## Preconditions

- session / memory subsystem 支持 `FeedbackMemory` 或语义等价类型
- durable memory subsystem 支持 private / shared split，或语义等价 scope 模型

## Ingress

1. 提供一条明确的 correction guidance
2. 提供一条 validated success guidance
3. 分别构造：
   - personal style preference
   - project-wide convention
4. 触发 durable memory 写入与后续 recall

## Expected Runtime Semantics

- `FeedbackMemory` 必须能同时保存：
  - correction
  - confirmation / validated success
- personal style preference 应默认落到 private scope
- 明显的 project-wide convention 才应落到 shared/team scope
- 记录内容应包含 rule 本身，以及 `why` / `how_to_apply` 或语义等价结构
- recalled `FeedbackMemory` 应作为行为 guidance 使用，而不是用户画像或项目背景

## Expected Persistent Effects

- correction 与 validated success 都能进入 durable store
- private guidance 不应静默覆盖已存在 shared/team convention
- consolidation 应允许合并重复 guidance，并清理被后续事实推翻的旧 guidance

## Allowed Variance

- `validation_kind` 可显式建模，也可通过语义等价字段体现
- 冲突 guidance 可选择：
  - 不保存
  - 显式记录 override
  - 或其它语义等价冲突处理策略

## Failure Conditions

- 只有 correction 能被保存，validated success 被忽略
- personal style preference 被默认升级为 shared/team
- `FeedbackMemory` 缺失 `why` / `how_to_apply` 语义，导致无法判断触发边界
- recalled feedback 被错误当作 `UserMemory` 或 `ProjectMemory`
