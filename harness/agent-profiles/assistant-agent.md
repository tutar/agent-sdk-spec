# Assistant Agent

## 职责

`AssistantAgentProfile` 定义一种 long-lived、chat/channel-facing、background-capable 的 harness host profile。

它负责描述：

- assistant-style 会话的长期运行形态
- 主线程为何必须偏响应式
- 用户可见输出为何应通过显式 user-facing surface 暴露
- 为什么长期记忆可采用 append-first、consolidate-later 的 operating mode
- 为什么 maintenance / check-in / dream-style background work 会成为宿主能力的一部分

它不负责：

- 定义 `MultiAgent` 机制本体
- 定义 `Gateway` channel adapter 协议
- 定义 `Task` lifecycle
- 定义 `Session.memory` 的对象模型

## 核心结论

assistant agent 不是单个 agent definition，也不是单个 tool。

它更像一种宿主形态：

- 会话是长驻的
- 用户可见输出优先走显式 user-facing channel
- 主线程更像 coordinator，而不是可长期阻塞的 shell driver
- 长期记忆更适合先累积、后蒸馏
- 后台 maintenance / dream-style consolidation 可以成为会话常驻能力
- 可预置 worker / team context，但不要求把 multi-agent 作为本体

## 稳定对象

推荐最小对象：

```text
AgentProfile
  - profile_name
  - interaction_style
  - session_shape
  - user_output_contract
  - background_work_policy
  - memory_write_policy
  - continuity_policy
  - channel_affinity?
```

```text
AssistantAgentProfile
  - profile_type: assistant
  - long_lived_session: true
  - requires_explicit_user_channel: true
  - prefer_responsive_main_thread: true
  - supports_viewer_attach: true
  - append_first_memory_mode?
  - scheduled_maintenance_mode?
  - preseeded_team_context?
```

实现不要求暴露完全相同的字段名，但这些行为面应稳定存在。

## 稳定语义

### 1. 长驻会话形态

assistant agent 应允许一个 session 持续存在，并在多个 turn 之间保持 assistant-style continuity。

这通常意味着：

- session 不依赖单次前台 REPL 生命周期
- 允许 reattach / viewer attach
- 宿主可以在无显式用户输入时继续维护 background work 或 maintenance state

### 2. 显式用户输出面

assistant agent 不应默认把普通终端文本视为唯一 user-facing output surface。

更稳定的语义是：

- 存在显式用户输出通道
- 运行时可以区分“内部协作输出”和“对用户可见输出”
- user-facing communication 可以由 gateway/channel surface 投影，但其语义由 profile 决定

### 3. 主线程响应式偏好

assistant agent 的主线程应偏响应式。

至少应允许：

- 长阻塞工作被后台化
- 或语义等价地被移出主交互路径
- 主用户交互面不因某个长时 shell/tool 调用而整体冻结

这个要求不强制具体 task 实现，但要求 profile 的外部行为体现“主线程不长期阻塞”。

### 4. continuity / viewer attach

assistant agent 应允许：

- 当前会话被 viewer / observer 重新附着
- 近期 history 被按需分页或语义等价方式重新展示
- continuity 不依赖 transcript 全量重放到当前 prompt

这是一种会话形态语义，不是 `Gateway` 或 `Session` 单方独占的能力。

### 5. append-first memory operating mode

assistant agent 可采用 append-first durable memory operating mode：

- 新记忆先写入 append-only capture stream
- 稍后再由 consolidation 流程蒸馏成 durable index / topic records

这与普通“直接维护长期记忆索引”的模式不同，但不改变 `Session.memory` 对 durable memory 的所有权。

### 6. scheduled maintenance

assistant agent 可支持一类宿主级 background maintenance：

- catch-up
- check-in
- dream-style consolidation
- 其他定期维护任务

这些任务属于 profile 的常见 operating mode，但并不改变 `Task` 或 `Session.memory` 的子域归属。

### 7. pre-seeded worker / team context

assistant agent 可以在宿主启动时预置 worker/team context，使 delegation 与 teammate execution 更自然。

但这仍然只是：

- profile 对 `MultiAgent` 的组合使用

而不是 `AssistantAgentProfile` 自己拥有一套新的 multi-agent 机制。

## 与其它子域的边界

### 与 `multi-agent/`

assistant agent 可以预置或强化 multi-agent 使用，但它不定义：

- delegated subagent
- teammate execution
- mailbox routing

这些仍归 [../multi-agent/README.md](../multi-agent/README.md)。

### 与 `gateway/`

assistant agent 常把 channel 作为主 user-facing surface，但它不拥有：

- `ChannelAdapter`
- `IngressNormalization`
- `GatewayInteractionState`

这些仍归 [../gateway/gateway.md](../gateway/gateway.md) 与相关子页。

### 与 `task/`

assistant agent 可以要求主线程偏响应式，并因此倾向后台化长工作。

但它不拥有：

- task registry
- task lifecycle
- task notification

这些仍归 [../task/README.md](../task/README.md)。

### 与 `context-engineering/`

assistant agent 可以改变：

- memory accumulation operating mode
- user-facing output contract
- startup / continuity profile

但不改变：

- context planes
- assembly pipeline
- compact / editing / prompt cache 的核心语义

这些仍归 [../context-engineering/README.md](../context-engineering/README.md)。

### 与 `session.memory`

assistant agent 可以采用：

- append-first capture
- consolidate-later durable memory organization

但 durable memory 的 recall、scope、consolidation 对象模型仍归 `Session.memory`。

## Default Local Mapping

当前本地实现的 assistant mode 可以被视为 `AssistantAgentProfile` 的默认映射。

这个默认映射通常具有以下行为：

- 会话显式进入 assistant mode 后，主 user-facing output surface 切到显式 user message channel
- 长阻塞 shell work 会被自动后台化或鼓励委派
- viewer 可附着既有 session 并按需获取历史
- durable memory 改成 append-first capture，再由后续 consolidation 蒸馏
- 允许 built-in scheduled maintenance / dream-style background work
- 允许预置 team / teammate context

这些都是默认本地映射，不构成唯一合法实现。

## Prompt Appendix

本附录只保留默认本地映射中最关键的 prompt 原文。
这些原文帮助实现者理解该 profile 的默认行为，但不构成额外核心规范义务。

### A. Proactive-style host addendum

来源：

- `main.tsx`

作用：

- 为长驻主动运行形态追加 host-level behavioral contract

原文：

```text
# Proactive Mode

You are in proactive mode. Take initiative — explore, act, and make progress without waiting for instructions.

Start by briefly greeting the user.

You will receive periodic <tick> prompts. These are check-ins. Do whatever seems most useful, or call Sleep if there's nothing to do. ${briefVisibility}
```

### B. Assistant-mode daily-log memory prompt

来源：

- `memdir/memdir.ts`

作用：

- 把 durable memory 写入策略从“直接维护索引”切到“按日追加日志，再夜间蒸馏”

原文：

```text
# auto memory

You have a persistent, file-based memory system found at: `<memoryDir>`

This session is long-lived. As you work, record anything worth remembering by **appending** to today's daily log file:

`<memoryDir>/logs/YYYY/MM/YYYY-MM-DD.md`

Substitute today's date (from `currentDate` in your context) for `YYYY-MM-DD`. When the date rolls over mid-session, start appending to the new day's file.

Write each entry as a short timestamped bullet. Create the file (and parent directories) on first write if it does not exist. Do not rewrite or reorganize the log — it is append-only. A separate nightly process distills these logs into `MEMORY.md` and topic files.
```

以及紧随其后的“what to log”段：

```text
## What to log
- User corrections and preferences ("use bun, not npm"; "stop summarizing diffs")
- Facts about the user, their role, or their goals
- Project context that is not derivable from the code (deadlines, incidents, decisions and their rationale)
- Pointers to external systems (dashboards, Linear projects, Slack channels)
- Anything the user explicitly asks you to remember
```

### C. Auto-background explanatory text

来源：

- `BashTool.tsx`
- `PowerShellTool.tsx`

作用：

- 把 assistant-mode 的“主线程保持响应式”要求直接反馈给模型

原文：

```text
Command exceeded the assistant-mode blocking budget (${ASSISTANT_BLOCKING_BUDGET_MS / 1000}s) and was moved to the background with ID: ${backgroundTaskId}. It is still running — you will be notified when it completes. Output is being written to: ${outputPath}. In assistant mode, delegate long-running work to a subagent or use run_in_background to keep this conversation responsive.
```

### D. Dream-style consolidation prompt

来源：

- `services/autoDream/consolidationPrompt.ts`

作用：

- 说明 consolidate-later 的长期记忆 operating mode 如何被执行

原文开头：

```text
# Dream: Memory Consolidation

You are performing a dream — a reflective pass over your memory files. Synthesize what you've learned recently into durable, well-organized memories so that future sessions can orient quickly.
```

### E. Assistant addendum body

当前本地实现会调用一个 assistant-specific system prompt addendum provider。

在当前快照中：

- 调用点可见
- 但 addendum 正文本体不可见

因此本规范只确认：

- assistant agent profile 通常会附加一段 host-specific system prompt addendum
- 当前快照不足以把该段原文纳入附录
