# Assistant Agent Profile Background Responsiveness

## 目标

验证 `AssistantAgentProfile` 下主线程偏响应式，长阻塞工作不会把主用户交互面整体冻结。

## Preconditions

- host 宣称支持 `Harness.AgentProfiles`
- 至少实现了一个 assistant-mode host profile
- 存在至少一种可能长时间阻塞的工作单元，例如 shell command、tool execution 或语义等价对象

## Ingress

1. 启动一个 assistant agent session
2. 在主交互路径中触发一个可能长时间运行的工作单元
3. 在该工作仍未结束时，继续向同一 session 注入新的用户输入或语义等价的交互刺激
4. 观察主交互面与后台工作之间的关系

## Expected Runtime Semantics

- assistant agent 的主交互面应保持响应式
- 长阻塞工作应被后台化、委派，或以语义等价方式移出主交互路径
- 用户交互不应被迫等待该工作完全结束才继续
- 主线程“保持响应式”是 profile 语义，不要求绑定某种特定 task 实现

## Expected Persistent Effects

- 若实现使用 task subsystem，后台化后的工作应保留 task-like 可观察事实
- 若实现不使用 task subsystem，也应存在语义等价的后台工作追踪面

## Allowed Variance

- 可以使用 task、async worker、subagent delegation 或其它语义等价机制
- 不要求所有实现都自动后台化；也可通过 profile-level explicit delegation 达成同等响应式结果

## Failure Conditions

- assistant agent 的主交互面会被长阻塞工作整体冻结
- 用户必须等待长工作完成后才能继续与同一 session 交互
- “assistant profile” 仅是显示层变化，没有体现响应式主线程语义
