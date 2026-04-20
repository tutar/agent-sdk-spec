# User Memory

## 职责

`UserMemory` 定义 durable memory 中的用户画像层。

它负责：

- 用户角色
- 知识背景
- 职责
- 与未来协作有关的稳定偏好

## 核心语义

- `UserMemory` 用于 personalization，不用于行为规则约束
- 它应帮助未来回答更贴合用户背景与 perspective
- 它不应用于负面 judgement 或与工作无关的 profiling
- 在 shared/private split 存在时，`UserMemory` 应默认 private

## 与其它类型的边界

- 与 [feedback-memory.md](feedback-memory.md)
  `UserMemory` 记“用户是谁”；`FeedbackMemory` 记“以后该怎么做”
