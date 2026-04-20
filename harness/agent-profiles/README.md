# Agent Profiles

本目录收拢 `Harness` 域内与 agent host profile 直接相关的语义。

它回答的问题是：

- 什么是 `AgentProfile`
- 哪些行为属于一种长期运行形态，而不是某个单独 tool 或某条 runtime 细节
- profile 如何组合 `gateway`、`task`、`multi-agent`、`context-engineering`、`session.memory` 等既有子域能力

这里的 profile 不是 deployment hosting profile，也不是 cloud/local 责任分布。
deployment 相关语义见 [../deployment-boundaries.md](../deployment-boundaries.md)。

## 目录内文档

- [assistant-agent.md](assistant-agent.md)
