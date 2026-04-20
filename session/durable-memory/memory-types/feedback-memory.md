# Feedback Memory

## 职责

`FeedbackMemory` 定义 durable memory 中的长期行为指导层。

它负责：

- 以后该怎么做
- 以后不要怎么做
- 哪些不显然的方法已经被验证有效

## 核心语义

- `FeedbackMemory` 既应记录 correction，也应记录 validated success
- 它的目标是减少重复纠正，并保持长期协作方式一致
- 推荐支持：
  - rule
  - why
  - how_to_apply
- 在 shared/private split 存在时，默认 private；只有 project-wide convention 才应上升为 team/shared

## 与其它类型的边界

- 与 [user-memory.md](user-memory.md)
  本页定义行为规则；`UserMemory` 定义用户画像
- 与 [project-memory.md](project-memory.md)
  本页回答“以后怎么做”；`ProjectMemory` 回答“为什么项目现在要这样做”
