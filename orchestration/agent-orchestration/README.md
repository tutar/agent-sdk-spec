# Agent Orchestration

本目录收拢 `Orchestration` 域内与 agent / subagent / verifier / task / 分工协作 直接相关的通用编排语义。

它回答四类问题：

- agent 可以以哪些模式被编排
- runtime task 如何被托管、恢复和通知
- 多 agent 如何分工与表达阻塞关系
- verifier 如何作为特化的 agent/task 进入标准编排链

这组文档描述的是通用 agent 编排主线，不等于 Cloud deployment 下的托管控制面。
Cloud 侧的控制面语义见 [../cloud/README.md](../cloud/README.md) 与 [../cloud/managed-orchestration.md](../cloud/managed-orchestration.md)。

## 目录内文档

- [agent-orchestration-modes.md](agent-orchestration-modes.md)
- [task-manager.md](task-manager.md)
- [work-allocation.md](work-allocation.md)
- [evaluation-and-verification.md](evaluation-and-verification.md)
