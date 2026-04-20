# Case: Durable Memory Index And Bounded Recall

## 目标

验证长期记忆不会被整库无界注入当前 turn，而是通过常驻索引与 bounded recall 分层进入上下文。

## Preconditions

- session / memory subsystem 支持 durable memory recall
- durable memory store 中存在多条可供召回的长期记忆
- runtime 支持 context assembly 中的 memory 注入

## Ingress

1. 预先准备一个包含多条 durable memory 的长期记忆空间
2. 发起一轮只需要其中少量记忆的请求
3. 观察 memory index、候选选择和 recalled payload 的进入方式

## Expected Runtime Semantics

- runtime 不应把整个 durable memory store 无界注入当前 turn
- 应能区分：
  - 常驻索引或等价 manifest
  - 当前轮被选中的 recalled payload
- recall 必须有 bounded 语义，例如数量、体积或等价预算上界
- recalled memory 进入 context plane，而不是 transcript

## Expected Persistent Effects

- durable memory store 保持原有 durable 记录
- recall 不应通过改写 transcript 模拟成功
- consolidation 之后，新的 durable memory 仍应遵守相同的 bounded recall 语义

## Allowed Variance

- 索引可以是 `MEMORY.md`、manifest 文件、header cache 或语义等价结构
- recall 选择器可以是规则、检索器或模型辅助路径
- recalled memory 可以进入 attachment-like context 或 structured context

## Failure Conditions

- 每轮都加载全部 durable memory 正文
- 索引与 recalled payload 无法区分
- recall 没有任何数量或体积边界
- recalled memory 被写回 transcript 伪装成原始会话消息
