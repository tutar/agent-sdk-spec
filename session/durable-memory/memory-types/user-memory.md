# User Memory

## 职责

`UserMemory` 定义 durable memory taxonomy 中的用户画像层。

它回答的是：

- 你在和谁协作
- 这个用户的角色、目标、职责、知识背景和稳定偏好是什么

它不回答：

- 以后该怎么做
- 项目当前为什么这样做
- 去哪里找外部信息

## 保存什么

`UserMemory` 应保存会改变未来协作方式的用户信息，例如：

- role
- goals
- responsibilities
- knowledge / experience profile
- stable preferences relevant to future collaboration

核心要求：

- 这些信息必须与未来协作有持续价值
- 目标是 personalization，而不是泛化 profiling

## 不保存什么

`UserMemory` 不应用来保存：

- negative judgement
- 与当前工作无关的用户评价
- 项目背景事实
- 行为 guidance
- 外部 reference pointer
- code pattern、架构、git 历史、文件路径

如果一个信息主要回答的是：

- “以后该怎么做 / 别怎么做”
  应归 `FeedbackMemory`
- “项目为什么现在这样做”
  应归 `ProjectMemory`
- “去哪里找外部信息”
  应归 `ReferenceMemory`

## 何时应写入

推荐在学到以下信息时写入：

- 用户 role
- 用户 responsibilities
- 用户知识背景或经验水平
- 会长期影响协作方式的稳定偏好

触发不要求用户显式说“记住这个”。  
只要这些信息对后续协作持续有价值，就应允许写入 durable memory。

## 如何使用

`UserMemory` 的使用目标是 personalization。

它应至少影响：

- explanation style
- abstraction level
- analogy choice
- 对已有知识的默认假设
- 合作时的信息呈现方式

它不应直接替代行为规则系统。  
也就是说，`UserMemory` 更接近“如何与这个人解释和协作”，而不是“以后必须采取什么行动”。

## Scope 语义

在支持 shared/private split 的实现中，`UserMemory` 应始终 private。

这是一个强约束：

- 用户画像不应进入 shared/team durable memory
- personalization 与团队共享知识必须严格分离

即使实现支持 team memory 或其它 shared durable scopes，`UserMemory` 也不应被提升为 shared/team。

## 推荐最小记录结构

推荐最小对象：

```text
UserMemoryRecord
  - memory_ref
  - role?
  - goals?
  - responsibilities?
  - knowledge_profile?
  - stable_preferences?
  - scope: private
  - updated_at?
```

字段意图：

- `role`
  - 用户职业或协作身份
- `goals`
  - 长期目标或工作关注点
- `responsibilities`
  - 用户承担的职责边界
- `knowledge_profile`
  - 用户在相关领域的经验/熟悉度
- `stable_preferences`
  - 会长期影响协作方式的稳定偏好

## 记录结构要求

`UserMemory` 不要求固定的 `rule + why + how_to_apply` 形态，但至少应保留：

- user profile 本体
- 与 future collaboration 的相关性

如果只保存零散标签、不保存其协作意义，未来 recall 的价值会显著下降。

## Staleness 与 Consolidation

`UserMemory` 通常比 `ProjectMemory` 更稳定，但仍可能过时：

- 角色变化
- 当前关注点变化
- 新的知识背景覆盖旧假设

因此实现应允许：

- merge 重复画像
- 修正过时背景
- 删除明显已不适用的旧 profile

但 consolidation 不应把 `UserMemory` 泛化成 shared/team convention，也不应把 user profile 误改写成 behavior rule。

## 与 Recall 的关系

当 `UserMemory` 被 recall 时，runtime 应把它当作 personalization context，而不是行为规则。

推荐效果：

- 调整解释深度
- 选择更合适的类比
- 使用更贴近用户 perspective 的表达

如果 recalled `UserMemory` 与当前用户的最新表述冲突，应优先相信当前对话中的新信息，并允许后续更新旧 memory。

## 与其它类型的边界

- 与 [feedback-memory.md](feedback-memory.md)
  - `UserMemory` 记“你在和谁协作”
  - `FeedbackMemory` 记“以后该怎么做 / 别怎么做”
- 与 [project-memory.md](project-memory.md)
  - `ProjectMemory` 记项目背景和动机
  - `UserMemory` 记用户画像
- 与 [reference-memory.md](reference-memory.md)
  - `ReferenceMemory` 记外部入口
  - `UserMemory` 记 personalization

## Default Local Mapping

当前默认本地映射中，`UserMemory` 的权威语义包括：

- 保存 role / goals / responsibilities / knowledge
- personalization-oriented，而不是 broad profiling
- 不记录 negative judgement
- scope 始终 private

这一定义会被：

- foreground durable write
- turn-end extraction
- dream consolidation

统一沿用到 durable memory 的写入和整理路径中。
