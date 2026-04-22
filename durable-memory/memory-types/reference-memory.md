# Reference Memory

## 职责

`ReferenceMemory` 定义 durable memory taxonomy 中的外部入口层。

它回答的是：

- 去哪里找外部信息
- 这些外部资源是做什么用的

它不回答：

- 外部系统当前内容本体是什么
- 项目为什么现在这样做
- 用户是谁
- 以后具体该怎么做

## 保存什么

`ReferenceMemory` 应保存 authoritative external source 的 pointer 与用途，例如：

- bug tracker project
- dashboard
- runbook
- communication channel
- project directory 之外的其它信息入口

核心要求：

- 保存的是 lookup path
- 同时保存 purpose / lookup cue

只有地址没有用途通常不够，因为未来 recall 需要知道“什么时候该想到它”。

## 不保存什么

`ReferenceMemory` 不应用来保存：

- 外部系统的当前事实快照
- 易腐烂的外部状态本体
- 项目内部背景事实
- 用户画像
- 行为 guidance

如果一个信息主要回答的是：

- “这个项目为什么现在这样做”
  应归 `ProjectMemory`
- “以后怎么做”
  应归 `FeedbackMemory`
- “这个用户是谁”
  应归 `UserMemory`

## 何时应写入

推荐在学到以下信息时写入：

- 某个 external resource 的存在
- 该 resource 的 purpose
- 该 resource 与哪类问题或场景相关

典型场景：

- 某类 bug 在特定 Linear project 里跟踪
- 某个 dashboard 是 oncall 真正看的面板
- 某个 Slack channel 是反馈入口

## 如何使用

`ReferenceMemory` 的使用目标不是直接回答事实问题，而是更快找到 authoritative external source。

推荐使用方式：

1. 先 recall `ReferenceMemory`
2. 再去外部系统获取当前事实
3. 必要时重新验证和更新该 pointer

因此 `ReferenceMemory` 更像 pointer memory，而不是 fact memory。

## Scope 语义

在支持 shared/private split 的实现中，`ReferenceMemory` 应通常偏 shared/team。

原因：

- 大多数 external resource 是项目共享 lookup surface
- dashboard、tracker、runbook、feedback channel 通常不属于个人私有偏好

private `ReferenceMemory` 只适用于确实属于个人工作流或个人访问面、且不应进入共享 durable memory 的 pointer。

## 推荐最小记录结构

推荐最小对象：

```text
ReferenceMemoryRecord
  - memory_ref
  - resource_pointer
  - purpose
  - scope
  - authority_kind?
  - updated_at?
```

字段意图：

- `resource_pointer`
  - 标识 external source 在哪里
- `purpose`
  - 说明该 source 解决什么查找问题
- `authority_kind`
  - 可选标识 dashboard、tracker、runbook、channel 等语义类别

## 记录结构要求

`ReferenceMemory` 不强制要求像 `feedback` / `project` 那样必须带 `why + how_to_apply`，但至少应同时表达：

- pointer 在哪里
- 这个 pointer 有什么用

如果没有 `purpose`，recall 很容易退化成一堆无上下文链接。

## Staleness 与 Consolidation

`ReferenceMemory` 虽然常比 `ProjectMemory` 更稳定，但 external pointer 仍可能失效、迁移或不再 authoritative。

因此实现应允许：

- pointer validity re-check
- consolidation 中合并同义入口
- 删除失效 pointer
- 保留最 authoritative 的少量 external references

若 recall 到的 pointer 已坏掉或不再可信，runtime 不应继续把它当有效 external source 使用。

## 与 Recall 的关系

当 `ReferenceMemory` 被 recall 时，runtime 应把它当作 lookup cue，而不是答案本体。

推荐效果：

- 帮助定位当前 authoritative external source
- 引导后续读取当前状态
- 不把 recalled pointer 直接等价成当前事实

## 与其它类型的边界

- 与 [project-memory.md](project-memory.md)
  - `ProjectMemory` 记项目背景事实
  - `ReferenceMemory` 记去哪里找外部信息
- 与 [feedback-memory.md](feedback-memory.md)
  - `FeedbackMemory` 记行为 guidance
  - `ReferenceMemory` 记 lookup path
- 与 [user-memory.md](user-memory.md)
  - `UserMemory` 记 personalization
  - `ReferenceMemory` 记 external source pointer

## Default Local Mapping

当前默认本地映射中，`ReferenceMemory` 的权威语义包括：

- 保存外部入口及其用途
- 不保存外部系统当前内容本体
- scope 通常偏 shared/team

它同样会进入：

- turn-end extraction
- dream consolidation

并在这些路径中被去重、修 stale 和收紧。
