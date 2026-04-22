# Instruction Include Expansion

## 职责

`AgentRuntime` 在 instruction loading 的解析阶段调用 `InstructionIncludeExpansion`，展开 instruction markdown 中的 `@include` 或语义等价 include 机制。

它回答的是：

- include 在什么时候被解析
- include 可以引用什么
- 递归、去重和安全边界如何工作

## 核心结论

include expansion 是 instruction loading 的解析子步骤，不是独立 discovery system。

也就是说：

- 只有当主 instruction file 被读取时，include 才会在同一次解析中展开
- include file 属于被主文件带入的附属 source，不单独构成新的 durable memory store

## 稳定语义

### 1. parse-time expansion

include 应在主 instruction file 解析时展开，而不是由后台扫描器预先全盘发现。

### 2. text-only applicability

include 应仅对普通文本内容生效，不应默认在 code block、inline code 或语义等价的 literal section 中触发。

### 3. bounded recursion

实现必须显式限制：

- 最大递归深度
- 循环引用
- 重复展开

### 4. missing target tolerance

若 include target 不存在，宿主可以：

- 忽略
- 记录诊断
- 降级加载

但不应因此破坏整个 session 的 restore 或普通 turn 可继续性。

### 5. external include safety

若 include target 超出当前批准的工作范围，宿主应允许：

- 安全确认
- deny / skip
- policy-gated expansion

## 推荐接口

```text
InstructionIncludeResolver
  - extract_include_refs(parsed_markdown) -> include_refs
  - resolve_include(ref, parent_source) -> included_source?
  - expand(parent_source, include_refs) -> expanded_sources[]
```

## 与其它子规范的边界

- 与 [instruction-markdown-loading.md](instruction-markdown-loading.md)
  include expansion 是 loading 的解析阶段，不替代 source discovery / precedence
- 与 [conditional-instruction-rules.md](conditional-instruction-rules.md)
  conditional rules 决定 instruction applicability；include expansion 决定正文如何展开
