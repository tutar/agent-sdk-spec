# Case: Gateway Ingress Vs Runtime Core Boundary

## 目标

验证 raw channel ingress 属于 gateway，runtime core 只消费 normalized inbound。

## Preconditions

- 至少存在一个 channel-facing ingress path
- gateway 能把 channel input 归一化为 runtime 可消费的对象

## Ingress

- 来自任意 channel 的一条用户输入

## Expected Runtime Semantics

- channel-specific parsing、identity、delivery metadata 处理止步于 gateway
- runtime core 只看到 normalized inbound 或语义等价对象
- 相同语义输入从不同 channel 进入时，runtime core 行为应等价
- turn progression 不依赖 Feishu、WeChat、Telegram、TUI 或其它特定 channel 的协议细节

## Expected Persistent Effects

- session/transcript 记录的是归一化后的语义输入，而不是 channel wire payload

## Allowed Variance

- gateway 可在本进程或远端运行
- normalized inbound 的字段名可不同，只要语义稳定

## Failure Conditions

- runtime core 直接解析 raw channel payload
- 不同 channel 导致 runtime core 语义漂移
- channel delivery state 混入 runtime core 的 turn-local execution state
