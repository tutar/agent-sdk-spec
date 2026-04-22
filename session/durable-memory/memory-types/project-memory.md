# Project Memory

## 职责

`ProjectMemory` 定义 durable memory taxonomy 中的项目背景层。

它回答的是：

- 这个项目现在为什么这样推进
- 哪些非代码态背景、约束、deadline、incident 会长期影响建议

它不回答：

- 代码结构长什么样
- git 最近发生了什么
- 用户是谁
- 以后具体该怎么做

## 保存什么

`ProjectMemory` 应保存不能从 code 或 git history 直接推导的项目背景，例如：

- ongoing work 的非代码态背景
- goals / initiatives
- bugs / incidents 的项目层含义
- constraints / deadlines / stakeholder asks
- 会影响未来建议的组织或协调上下文

核心要求：

- 保存的是项目背景、动机和约束
- 不是 repo 结构摘要，也不是 changelog

## 不保存什么

`ProjectMemory` 不应用来保存：

- code pattern、架构、目录结构
- git 历史、谁改了什么
- 当前 turn 的临时任务状态
- 纯 lookup path
- 用户个性化画像
- 纯行为 guidance

如果一个信息主要回答的是：

- “以后怎么做”
  应归 `FeedbackMemory`
- “去哪里找外部信息”
  应归 `ReferenceMemory`
- “这个用户是谁”
  应归 `UserMemory`

## 何时应写入

推荐在学到以下信息时写入：

- 谁在做什么
- 为什么要做
- 需要在什么时间之前做
- 哪些非代码态约束会影响未来决策

特别要求：

- relative dates 在写入时应转成 absolute dates 或语义等价形式

例如：

- `Thursday`
  - 写成一个明确日期

原因是 `ProjectMemory` 天然高 staleness，时间锚点必须在写入时就被固定下来。

## 如何使用

`ProjectMemory` 的使用目标是为未来建议提供项目背景和动机上下文。

它应至少影响：

- 方案取舍
- 风险判断
- coordination awareness
- 对 deadline / compliance / stakeholder constraint 的敏感度

它不是 personal preference，也不是 action policy；它更像 future decision-making 的背景事实层。

## Scope 语义

在支持 shared/private split 的实现中，`ProjectMemory` 应允许 private 或 shared/team，但应 strongly bias toward shared/team。

原因：

- 大多数项目背景本就是团队共享上下文
- merge freeze、合规约束、initiative 背景通常不该只停留在私人 durable memory 中

private `ProjectMemory` 只适用于确实不应外溢到团队共享层的项目背景。

## 推荐最小记录结构

推荐最小对象：

```text
ProjectMemoryRecord
  - memory_ref
  - fact_or_decision
  - why
  - how_to_apply
  - scope
  - effective_date?
  - updated_at?
```

字段意图：

- `fact_or_decision`
  - 保存具体项目背景事实或关键决策
- `why`
  - 保存动机、constraint、deadline、stakeholder ask
- `how_to_apply`
  - 说明未来建议应如何受这条项目背景影响
- `effective_date`
  - 用于给时效性强的信息提供明确时间锚点

## 记录结构要求

推荐 body structure：

1. 先写 fact or decision
2. 再写 `Why:`
3. 再写 `How to apply:`

原因：

- `ProjectMemory` decay fast
- 只写事实，未来很难判断它是否仍然 load-bearing
- `why` 和 `how_to_apply` 决定它能否继续稳定影响后续建议

## Staleness 与 Consolidation

四类 taxonomy 中，`ProjectMemory` 的 staleness 风险应被视为最高之一。

因此实现应允许：

- 明确时间锚点
- stale warning 或 freshness note
- consolidation 中 merge 同一 initiative / incident 的多条记录
- 删除被后续事实推翻的项目背景
- 降权已不再 load-bearing 的旧约束

如果 recall 得到的 `ProjectMemory` 与当前代码、当前项目状态或用户最新输入冲突，应优先相信当前观察结果，并允许后续更新或删除旧 memory。

## 与 Recall 的关系

被 recall 的 `ProjectMemory` 不应伪装成 transcript 或 activity log。

它应作为 durable background context 进入当前 turn，并帮助 runtime：

- 理解用户请求的细微背景
- 预判 coordination 影响
- 输出更 informed 的建议

## 与其它类型的边界

- 与 [feedback-memory.md](feedback-memory.md)
  - `FeedbackMemory` 记以后该怎么行动
  - `ProjectMemory` 记为什么项目当前要这样做
- 与 [reference-memory.md](reference-memory.md)
  - `ReferenceMemory` 记外部入口
  - `ProjectMemory` 记项目内部背景和约束
- 与 [user-memory.md](user-memory.md)
  - `UserMemory` 记协作对象画像
  - `ProjectMemory` 记项目背景上下文

## 与 overlay 的边界

- 与 [../scopes/project-memory.md](../scopes/project-memory.md)
  - 本页定义 `project` 作为 payload semantics
  - overlay 页定义 `project` 作为 project-scoped durable binding

二者可以同时命中同一条 record，但不应被建模成同一字段语义。

## Default Local Mapping

当前默认本地映射中，`ProjectMemory` 的权威语义包括：

- 只保存不能从 code / git 派生的项目背景
- relative date 转 absolute date
- shared/team 强偏置
- 使用 `fact_or_decision + why + how_to_apply` 的 body structure

这一定义会在：

- foreground durable write
- turn-end extraction
- dream consolidation

中被统一复用。
