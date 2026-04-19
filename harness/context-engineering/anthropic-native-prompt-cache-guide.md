# Anthropic Native Prompt Cache Guide

## 定位

本页是 `Harness.ContextEngineering` 下的 **实现指南**。

它描述的是一种 provider-specific 的默认本地映射：

- 适用于 Anthropic Messages 风格的原生 prompt caching 能力
- 适用于支持 system block、content block cache marker 与 cache usage 观测的运行时

本页不是：

- 核心规范
- 标准扩展规范
- 单独 capability
- conformance target

跨 provider 的稳定语义仍以 [prompt-cache-strategy.md](prompt-cache-strategy.md) 为准。

## 适用前提

这条路线通常适用于满足以下条件的 provider/runtime：

- system prompt 可以按 block 投影
- tool schemas 或等价 capability surface 会进入模型可见输入
- message/content blocks 支持 cache marker 或语义等价结构
- response usage 中可观察到 cache read / write 或等价指标

## 为什么这不是一个简单开关

Anthropic native prompt caching 不是“打开 provider 缓存”就会自动高命中。

要获得稳定收益，harness 通常需要主动处理：

- system prompt block splitting
- static / dynamic boundary 放置
- block scope 选择
- tool schema cache marker 放置
- message-level marker 与 reference 顺序
- fork/subagent 的 cache-safe 参数稳定
- response-side cache break detection

因此，这更像一套 **输入组织策略**，而不是一个布尔开关。

## 推荐实现形态

推荐把 Anthropic native route 视为一条 strategy-local plan，而不是通用 object model：

```text
AnthropicNativePromptPlan
  - system_blocks
  - tool_blocks
  - message_cache_markers
  - global_cache_strategy
  - cache_write_enabled
```

```text
AnthropicSystemBlock
  - text
  - cache_scope: null | org | global
```

```text
AnthropicForkCachePlan
  - share_parent_prefix
  - cache_safe_params
  - skip_cache_write
```

这些对象只用于实现层设计，不应提升为跨 provider 的 canonical object model。

## 1. System Prompt Block Splitting

这条路线的核心前提是：**system prompt 必须被主动切块**。

推荐将以下部分分开建模：

- attribution header
- CLI / product prefix
- static content before dynamic boundary
- dynamic content after dynamic boundary

### 推荐边界

应显式设置一个 dynamic boundary marker，使 harness 能稳定区分：

- cacheable stable prefix
- per-turn dynamic suffix

### 推荐模式

#### Global cache mode with boundary

当 global scope 条件满足、且 prompt 内存在显式动态边界时，推荐分成：

- attribution header -> uncached
- CLI / product prefix -> uncached
- static content before boundary -> `global`
- dynamic content after boundary -> uncached

这条路径最适合：

- 稳定前缀很大
- 静态块跨 turn、高概率跨 session 复用
- 动态变化集中在 boundary 之后

#### Tool-based / org fallback

当全局 system caching 不适合时，推荐退回：

- attribution header -> uncached
- prefix -> `org`
- 其余 system content -> `org`

典型场景包括：

- tool surface 高频变化
- 连接态/服务态经常变
- 需要避免 global scope 被频繁 bust

## 2. Scope 选择建议

Anthropic native 路线通常会落到三类 scope：

- `null`
  不缓存
- `org`
  组织内稳定复用
- `global`
  条件更严格、复用范围更大

### 推荐选择原则

- attribution header 和高波动块通常不缓存
- 默认稳定前缀优先考虑 `org`
- 只有在边界稳定、动态部分被明确隔离时，才考虑把 boundary 前静态块提升到 `global`
- 如果系统发现某些连接态或工具态会持续改变 system prompt，则应主动退回 org-scoped plan

scope 变化本身是 cache-critical change。  
换言之，哪怕文本不变，仅 scope/TTL 策略翻转，也可能导致 cache break。

## 3. Tool Schemas And Capability Surface

Anthropic native prompt caching 不应只关注 system prompt。

模型可见的 tool schemas 也会参与 cache key，因此实现上应把它们当成 cache plan 的一部分。

### 推荐做法

- 把 tool schema surface 纳入 prompt state snapshot
- 将 tool schema 的动态变化视为 cache break 风险
- 在 capability exposure 层尽量减少无意义漂移

### 常见高风险变化面

- advisor / tool search 开关
- deferred tools 切换
- capability listing 变化
- 连接态变化导致工具集变化

这些变化并不一定错误，但应视为 cache-sensitive input，而不是普通 UI 细节。

## 4. Message-Level Cache Marker And Reference Placement

Anthropic native 路线通常允许 content/message block 级 cache marker。

这里的重点不是具体字段名，而是 **placement responsibility**：

- marker 放在哪个 block 上，应由 harness 决定
- reference 与 marker 的相对顺序，应由 harness 保证合法
- adapter 不应在发送前临时猜测和修补

### 推荐约束

- message-level marker 数量保持极小
- marker 位置尽量稳定
- 若 provider 对 reference 与 marker 的相对顺序有限制，默认采用更严格、更稳定的 placement 策略

## 5. Fork/Subagent Cache Sharing

Anthropic native 路线很适合 fork/subagent 共享父前缀，但前提是 **cache-safe params 稳定**。

### 推荐共享原则

- fork/subagent 继承父前缀时，不改变 model
- 不随意改 effort / thinking / budget-critical 参数
- 不改变 tool surface，除非这是目标行为

### 推荐区分两个决策

- `share_parent_prefix`
  是否复用父前缀
- `skip_cache_write`
  是否为这次分支调用写入新的 cache entry

这两个决策不应绑死。

例如：

- side question
  可以共享父 cache
  但通常应避免写入新的 suffix cache

### 常见误区

- 为了“更快”临时改 thinking/effort，结果 bust 父前缀
- fork 后把工具面裁掉或扩展，导致 cache key 变化
- one-off query 既共享父前缀又写入不再复用的 cache entry

## 6. Cache Break Detection

Anthropic native 路线推荐把 cache break detection 当成策略的一部分，而不是额外调试脚本。

因为仅凭“read tokens 下降”通常不足以区分：

- 合理的 cache 失效
- scope/marker/tool surface 漂移
- compact/clear/delete 导致的预期 drop
- TTL 到期

### 推荐跟踪面

建议至少记录：

- system blocks hash
- tool schema hash
- cache-control-related hash
- model
- strategy mode
- betas / feature headers
- effort / extra body params

### 推荐判断方式

将：

- prompt state snapshot
- cache read / write usage
- 当前 query source

结合起来看，而不是只看单个指标。

### 预期下降应与异常 break 分开

实现应显式区分这些“正常 cache read 下降”的场景：

- compact
- cache deletion / cache edit
- TTL 到期
- 合法的 strategy fallback

这类情况应重置 baseline 或以等价方式避免误报。

## 7. 常见失效点

Anthropic native 路线中最常见的 cache bust 来源包括：

- dynamic context 混进静态前缀
- tool schema surface 漂移
- scope / TTL flips
- forked call 改了 cache-safe params
- one-off query 错误写 cache
- message marker 放置不稳定
- 将高波动服务状态塞回 system prompt static block

这些问题大多不是 provider 的问题，而是 harness 没有稳定组织输入。

## 与规范页的关系

- [prompt-cache-strategy.md](prompt-cache-strategy.md)
  定义跨 provider 的稳定策略接口与语义
- [bootstrap-prompts.md](bootstrap-prompts.md)
  定义 system skeleton 与 static/dynamic boundary 的上游结构
- [context-assembly-pipeline.md](context-assembly-pipeline.md)
  定义 cache boundary placement 所属的 assembly 阶段

## 规范外结论

- Anthropic native prompt caching 更像一条实现路线，而不是一条通用规范
- 它最核心的价值不在 provider 字段，而在 harness 是否主动稳定输入组织
- 若实现者不需要兼容多 provider，这条路线可以作为 prompt cache 的默认本地最佳实践
