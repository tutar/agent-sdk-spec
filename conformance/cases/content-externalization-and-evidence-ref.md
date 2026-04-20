# Case: Content Externalization And Evidence Ref

## 目标

验证超长 payload 可以被 externalize，并在当前轮及后续恢复中保持 preview + stable evidence ref 语义。

## Preconditions

- 至少一个工具、任务或资源读取路径会产生超长 payload
- runtime 支持 `EvidenceRef` 或语义等价的外部引用对象
- session 支持 compact / resume 或语义等价恢复

## Ingress

1. 触发一个产生超长 payload 的调用
2. 让 runtime 对该 payload 执行 externalization
3. 在当前轮继续使用 preview 参与模型上下文
4. 再触发一次 compact 或 resume

## Expected Runtime Semantics

- 当前轮模型可见上下文应保留 preview 或语义等价摘要
- 完整 payload 不应被要求继续内联在 message stream 中
- runtime 应通过 stable ref 继续追溯同一外部结果

## Expected Persistent Effects

- compact / resume 后仍能定位到同一外部结果
- 不应要求重新生成原始 payload 才能恢复引用

## Allowed Variance

- ref 可以指向本地文件、对象存储、任务输出或语义等价的远程句柄
- preview 的长度和结构可以不同

## Failure Conditions

- externalization 后完整 payload 仍被强制内联在每轮上下文中
- compact / resume 后 evidence ref 漂移到不同结果
- 只能通过重跑原调用才能恢复同一 payload 引用
