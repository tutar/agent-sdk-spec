# Ingress Gateway

## 职责

`IngressGateway` 负责把外部消息来源与内部 agent runtime 连接起来。

它位于 `ChannelAdapter` 与 `Harness` 之间，负责：

- channel-agnostic ingress
- message normalization
- control routing
- session binding
- egress projection

当前仓库里的默认实现主要是 remote-control bridge，但规范不应把它限制为单一渠道。

## 稳定接口

推荐最小接口：

```text
IngressGateway
  - register_channel(adapter)
  - receive_inbound(inbound_envelope) -> normalized_input
  - route_control(control_message) -> control_result
  - bind_session(channel_identity, session_identity) -> binding
  - project_egress(runtime_event) -> egress_event
```

推荐补充：

```text
NormalizedInboundMessage
  - channel
  - conversation_id
  - sender_id
  - message_id
  - content
  - attachments
  - metadata

ControlEnvelope
  - subtype
  - request_id
  - payload

SessionBinding
  - channel_identity
  - conversation_id
  - session_id
  - agent_id?
```

## 设计要求

- ingress gateway 不等于具体 transport
- ingress gateway 不等于 channel adapter
- ingress gateway 不等于 harness loop
- ingress gateway 必须支持 ingress normalization
- ingress gateway 必须支持 control flow 与 normal message flow 分层
- ingress gateway 必须支持多渠道扩展
- ingress gateway 必须允许把多个渠道映射到统一 session 语义
- egress projection 必须允许过滤内部 chatter，只暴露合适的外部事件

## 默认实现策略

推荐分为四层：

1. `ChannelAdapter`
   负责接具体外部渠道并做协议适配
2. `IngressNormalization`
   负责把外部输入转成内部消息
3. `ControlRouting`
   负责处理 interrupt / permission / mode-change 等控制流
4. `SessionBindingAndProjection`
   负责 channel 与 session/runtime 的绑定及输出投影

## 与其它模块的边界

- 不等于 `Session`
- 不等于 `Harness`
- 不等于 `Transport`
- 不等于 `Orchestration`
- 不等于 `ChannelAdapter`

它位于 `ChannelAdapter` 与 `Harness` 之间，承担统一入口边界。

## 默认实现映射

当前仓库中的默认实现主要是 remote-control 变体：

- 入口包装见 [bridge/initReplBridge.ts](../../../bridge/initReplBridge.ts)
- bridge core 见 [bridge/replBridge.ts](../../../bridge/replBridge.ts)
- env-less core 见 [bridge/remoteBridgeCore.ts](../../../bridge/remoteBridgeCore.ts)
- transport abstraction 见 [bridge/replBridgeTransport.ts](../../../bridge/replBridgeTransport.ts)
- ingress projection 见 [bridge/bridgeMessaging.ts](../../../bridge/bridgeMessaging.ts)
- inbound normalization 见 [bridge/inboundMessages.ts](../../../bridge/inboundMessages.ts)
- inbound attachments 见 [bridge/inboundAttachments.ts](../../../bridge/inboundAttachments.ts)
- local session adapter 见 [bridge/sessionRunner.ts](../../../bridge/sessionRunner.ts)

## 规范结论

- bridge 在规范层应被提升为 ingress gateway
- remote-control 只是当前默认实现，不应限制规范抽象
- 所有外部 chat/channel 接入都应优先复用这层网关边界
- harness 不应内建各 channel 的服务端逻辑
