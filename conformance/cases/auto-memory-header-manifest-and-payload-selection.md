# Case: Auto Memory Header Manifest And Payload Selection

## 目标

验证 auto-memory runtime 会先基于 header / manifest 做 bounded selection，再读取少量 durable payload，而不是预读整库正文。

## Preconditions

- session / memory subsystem 支持 auto-memory 或语义等价的默认 durable runtime
- durable memory store 中存在 resident entrypoint、topic payload 与 header metadata 的语义分层
- runtime 支持在 recall 前做 bounded candidate scan

## Ingress

1. 预先准备多条 durable payload，并为每条 payload 提供 type / description / freshness 等 header metadata 或语义等价信息
2. 发起一轮只需要其中少量长期记忆的请求
3. 观察 recall 是否先基于 manifest / header 做候选选择，再读取正文

## Expected Runtime Semantics

- header / frontmatter scan 与 payload read 必须是两层
- manifest 或语义等价结构必须能够承载 bounded selection 的输入
- selection 可以使用规则、检索器或模型辅助路径，但不应要求整库正文预读
- freshness / mtime / description 一类 metadata 至少应允许被建模
- selected payload 与 resident entrypoint 必须语义分离

## Expected Persistent Effects

- durable payload 继续作为 durable正文存在
- manifest / header 层不应改写 transcript 来模拟 recall 成功
- consolidation 之后，新的 payload 仍应通过同样的 header / manifest layer 参与 recall

## Allowed Variance

- header metadata 可以来自 frontmatter、sidecar manifest、header cache 或语义等价结构
- selection 可以是规则、检索器或模型辅助路径
- freshness 可以使用 `mtime`、`updated_at` 或语义等价字段

## Failure Conditions

- recall 必须预读全部 durable payload 正文才能工作
- header / manifest 与 payload 无法区分
- manifest 不能表达 selection 所需的最小 metadata
- selected payload 被伪装成 transcript 原消息
