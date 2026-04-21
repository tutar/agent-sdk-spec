# Feedback Memory

## 职责

`FeedbackMemory` 定义 durable memory taxonomy 中的长期行为指导层。

它回答的是：

- 以后该怎么做
- 以后不要怎么做
- 哪些不显然的方法已经被验证有效

它不回答：

- 用户是谁、知识背景如何
- 项目当前的背景事实与动机
- 去哪里找外部信息

## 保存什么

`FeedbackMemory` 应保存会影响未来工作方式的 durable guidance，例如：

- 操作方式
- 协作偏好
- 回复风格约束
- 项目级行为规范
- 已被确认有效的非显然工作方法

核心要求：

- 既要记录 correction，也要记录 validated success
- 不应只记录“不要做什么”，也应记录“继续这样做”

否则实现容易出现：

- 只会避免过去错误
- 却逐渐偏离用户已经验证有效的方法
- 最终变得过于保守

## 不保存什么

`FeedbackMemory` 不应用来保存：

- 用户角色、职责、知识背景
- 项目背景事实、deadline、initiative 本体
- 外部系统入口
- code pattern、架构、git 历史、文件路径
- 只适用于当前 turn 的临时任务细节

若某条信息主要回答的是：

- “这个用户是谁”
  应归 `UserMemory`
- “项目为什么这样做”
  应归 `ProjectMemory`
- “去哪里查当前信息”
  应归 `ReferenceMemory`

## 何时应写入

推荐至少在以下两类场景中写入：

### 1. correction

当用户明确纠正做法时，例如：

- `no not that`
- `don't`
- `stop doing X`

### 2. validated success

当用户确认一个不显然的方法是对的时，例如：

- `yes exactly`
- `perfect, keep doing that`
- 接受一个 unusual choice 且没有 pushback

约束：

- correction 容易被发现；confirmation 更容易漏掉
- 实现不应把 `FeedbackMemory` 退化成“只记批评”的负面缓存
- 只有对未来会话仍然有帮助、且不显然的信息才值得写入

## 如何使用

`FeedbackMemory` 的使用目标是让未来行为被 durable guidance 稳定约束。

它应至少影响：

- 做事方式
- 回复方式
- 选择默认策略时的偏向
- 对相似工作场景的默认判断

它不应只在用户显式提到该规则时才生效。  
如果该 guidance 仍然有效，后续同类场景应默认受它影响。

## Scope 语义

在支持 shared/private split 的实现中，`FeedbackMemory` 应默认 private。

只有当 guidance 明显是 project-wide convention 时，才应上升为 shared/team。

典型区分：

- “回复不要再总结 diff”
  - private
- “集成测试必须打真实数据库”
  - shared/team

进一步要求：

- private feedback 不应静默覆盖已存在的 shared/team feedback
- 若两者冲突，实现至少应允许：
  - 不保存新的 private feedback
  - 显式记录 override
  - 或采用语义等价的冲突处理策略

## 推荐最小记录结构

推荐最小对象：

```text
FeedbackMemoryRecord
  - memory_ref
  - rule
  - why
  - how_to_apply
  - scope
  - validation_kind?: correction | confirmation
  - updated_at?
```

字段意图：

- `rule`
  - 直接给出应遵循的 guidance
- `why`
  - 保存用户给出的原因、历史事故、强偏好或验证依据
- `how_to_apply`
  - 说明在什么条件下触发、如何落到未来行为
- `validation_kind`
  - 区分来自 correction 还是 validated success

实现不要求公开同名字段，但这些语义应稳定存在。

## 记录结构要求

推荐 body structure：

1. 先写 rule 本身
2. 再写 `Why:`
3. 再写 `How to apply:`

原因：

- 只有规则本身不够
- 没有 `why`，实现很难判断 edge case
- 没有 `how_to_apply`，实现很难知道触发边界

因此 `FeedbackMemory` 不应只是松散备注，更接近半结构化的 policy record。

## Staleness 与 Consolidation

`FeedbackMemory` 虽然通常比 `ProjectMemory` 更稳定，但仍然可能过时或被后续事实推翻。

因此实现应允许：

- merge 相似规则
- 删除已被否定的 guidance
- 降权已长期不再适用的 personal preference
- 在 consolidation 中把重复 guidance 合并为单一 durable record

若后续 shared/team convention 已形成，旧 private feedback 也应允许被吸收、覆盖或显式失效。

## 与 Recall 的关系

当 `FeedbackMemory` 被 recall 时，runtime 不应把它当作普通背景事实，而应把它视为行为 guidance。

推荐效果：

- 先影响行动和表达策略
- 再影响次级解释风格

如果 recalled `FeedbackMemory` 与当前观察到的现实冲突，应优先相信当前现实，并允许后续更新或删除该 memory。

## 与其它类型的边界

- 与 [user-memory.md](user-memory.md)
  - `UserMemory` 记“你在和谁协作”
  - `FeedbackMemory` 记“以后该怎么做 / 别怎么做”
- 与 [project-memory.md](project-memory.md)
  - `ProjectMemory` 记项目背景与动机
  - `FeedbackMemory` 记未来行动规则
- 与 [reference-memory.md](reference-memory.md)
  - `ReferenceMemory` 记查找入口
  - `FeedbackMemory` 记行为 guidance

## Default Local Mapping

当前默认本地映射中，`FeedbackMemory` 的权威语义来自 memory taxonomy prompt：

- 同时记录 correction 与 validated success
- shared/private split 下默认 private
- 使用 `rule + why + how_to_apply` 的 body structure

这一定义还会被：

- turn-end extraction
- dream consolidation

沿用到 durable memory 的补写与整合路径中。
