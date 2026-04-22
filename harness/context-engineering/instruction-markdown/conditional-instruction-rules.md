# Conditional Instruction Rules

## 职责

`AgentRuntime` 在 instruction loading 与 target selection 之间调用 `ConditionalInstructionRules`，决定哪些 rules/frontmatter/`paths` 应进入当前上下文。

它回答的是：

- unconditional rule 和 conditional rule 的区别
- `paths` 或语义等价字段如何工作
- 为什么 rule matching 基于 target path，而不是简单 cwd

## 核心结论

rules 是 automatic instruction injection，不是 on-demand capability activation。

因此：

- rules 不等于 skills
- conditional rule 不等于“有条件的 skill”
- 规则命中后进入的是 instruction/context plane，而不是 workflow dispatch plane

## 稳定语义

### 1. unconditional vs conditional

- 无 `paths` 或等价条件字段
  - 视为 unconditional rule
- 有 `paths` 或等价条件字段
  - 视为 conditional rule

### 2. target-path matching

conditional rule 的匹配对象应是：

- 当前目标文件路径
- 当前工作目标路径
- 或语义等价的 target selector

而不是单纯依赖当前 cwd 文本。

### 3. frontmatter controls loading

rule frontmatter 主要用于控制匹配与加载，不要求作为正文内容进入模型输入。

### 4. subtree isolation

只匹配当前 target 的 rules 才应进入最终 instruction result。

兄弟子树的 conditional rules 不应污染当前 target。

## 推荐接口

```text
ConditionalInstructionRule
  - rule_ref
  - paths[]
  - base_dir
  - applicability
```

```text
InstructionRuleMatcher
  - list_rules(scope) -> rules
  - match(target_path, rules) -> matched_rules
```

## 与 Skills 的边界

- rules
  - automatic instruction injection
- skills
  - on-demand capability / workflow activation

所以：

- rules 适合必须自动生效的约束
- skills 适合按需触发的能力或流程
