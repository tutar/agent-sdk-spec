# Conformance Cases

## 目标

本目录提供跨语言、跨宿主共享的合规测试资产。

这些资产不绑定具体语言实现，也不绑定单一宿主形态，而是用于验证：

- `TUI / Desktop / Cloud` 三种 host 下的外部行为是否等价
- 不同语言 SDK 是否保持相同的 runtime 语义
- `Harness / Session / Tools / Sandbox / Orchestration` 五个模块的接口语义是否被正确实现

## 目录结构

- [cases/basic-turn.md](cases/basic-turn.md)
- [cases/tool-call-roundtrip.md](cases/tool-call-roundtrip.md)
- [cases/requires-action-approval.md](cases/requires-action-approval.md)
- [cases/session-resume.md](cases/session-resume.md)
- [cases/background-agent.md](cases/background-agent.md)
- [cases/hosting-profile-equivalence.md](cases/hosting-profile-equivalence.md)
- [cases/mcp-tool-adaptation.md](cases/mcp-tool-adaptation.md)
- [cases/sandbox-deny.md](cases/sandbox-deny.md)
- [cases/memory-recall-and-consolidation.md](cases/memory-recall-and-consolidation.md)
- [cases/desktop-bundle-lifecycle.md](cases/desktop-bundle-lifecycle.md)
- [cases/cloud-wake-and-reprovision.md](cases/cloud-wake-and-reprovision.md)
- [cases/prompt-cache-stable-prefix.md](cases/prompt-cache-stable-prefix.md)
- [cases/prompt-cache-dynamic-suffix.md](cases/prompt-cache-dynamic-suffix.md)
- [cases/prompt-cache-fork-sharing.md](cases/prompt-cache-fork-sharing.md)
- [cases/prompt-cache-break-detection.md](cases/prompt-cache-break-detection.md)
- [cases/prompt-cache-strategy-equivalence.md](cases/prompt-cache-strategy-equivalence.md)
- [golden/basic-turn.events.json](golden/basic-turn.events.json)
- [golden/tool-call-roundtrip.events.json](golden/tool-call-roundtrip.events.json)
- [golden/requires-action-approval.events.json](golden/requires-action-approval.events.json)
- [golden/session-resume.event-log.json](golden/session-resume.event-log.json)
- [golden/sandbox-deny.events.json](golden/sandbox-deny.events.json)
- [golden/memory-recall-and-consolidation.json](golden/memory-recall-and-consolidation.json)
- [golden/desktop-bundle-lifecycle.json](golden/desktop-bundle-lifecycle.json)
- [golden/cloud-wake-and-reprovision.json](golden/cloud-wake-and-reprovision.json)
- [golden/prompt-cache-stable-prefix.json](golden/prompt-cache-stable-prefix.json)
- [golden/prompt-cache-dynamic-suffix.json](golden/prompt-cache-dynamic-suffix.json)
- [golden/prompt-cache-fork-sharing.json](golden/prompt-cache-fork-sharing.json)
- [golden/prompt-cache-break-detection.json](golden/prompt-cache-break-detection.json)
- [golden/prompt-cache-strategy-equivalence.json](golden/prompt-cache-strategy-equivalence.json)

## 测试层次

### 1. Canonical Cases

`cases/` 下的文档定义标准场景。

每个场景至少描述：

- preconditions
- ingress
- expected lifecycle
- expected runtime events
- expected transcript or event log effects
- allowed implementation variance

### 2. Golden Artifacts

`golden/` 下的文件提供可共享的输入输出样本。

这些样本用于：

- replay
- snapshot comparison
- adapter conformance
- cross-host equivalence checks

### 3. Host Equivalence

同一 canonical case 可以在：

- `TUI`
- `Desktop`
- `Cloud`

三种 host profile 下运行。

允许不同 host 的进程边界、部署形态、存储位置不同，但不允许外部可观察语义漂移。

## 最低覆盖要求

一个语言 SDK 若要宣称实现本规范，至少应能通过：

- `basic-turn`
- `tool-call-roundtrip`
- `requires-action-approval`
- `session-resume`

若要宣称支持 agent orchestration，还应通过：

- `background-agent`

若要宣称支持跨宿主一致性，还应通过：

- `hosting-profile-equivalence`

若要宣称支持 MCP compatibility，还应通过：

- `mcp-tool-adaptation`

若要宣称支持 sandbox、memory consolidation、desktop bundle host、cloud managed-agent profile，还应通过：

- `sandbox-deny`
- `memory-recall-and-consolidation`
- `desktop-bundle-lifecycle`
- `cloud-wake-and-reprovision`

若要宣称支持 prompt-cache-aware harness，还应通过：

- `prompt-cache-stable-prefix`
- `prompt-cache-dynamic-suffix`
- `prompt-cache-fork-sharing`
- `prompt-cache-break-detection`

若要宣称支持跨 provider prompt cache strategy adapter，还应通过：

- `prompt-cache-strategy-equivalence`

## 规范结论

- `conformance` 测试的重点是行为语义，而不是内部实现结构
- `golden artifacts` 应保持语言无关、宿主无关
- host profile 可以改变部署方式，但不能改变规范定义的外部行为
