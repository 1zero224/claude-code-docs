---
title: 参考
url: https://platform.claude.com/docs/zh-CN/managed-agents/reference
description: Claude Managed Agents 的事件类型、自托管 worker CLI 标志、支持的 MCP 服务器类型、速率限制和品牌指南。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  status: beta
  betaHeader: managed-agents-2026-04-01
  zdr:
    eligibility: not-eligible
    note: 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-fable-5
    - claude-mythos-5
    - claude-mythos-preview
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-sonnet-5
    - claude-haiku-5-5
    - claude-haiku-4-5-20251001
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API: beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Anthropic managed): beta
---

本页汇集了 Claude Managed Agents 的参考资料。如需面向任务的指南，请点击各节中的链接。有关会话资源的操作，请参阅[会话操作](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations)。

## 事件类型

持久化的事件类型字符串遵循 `{domain}.{action}` 命名约定；仅限流的事件增量（请参阅**事件增量**选项卡）是例外。有关发送、流式传输和列出事件，请参阅[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。Webhook 事件类型单独列在[订阅 webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#supported-event-types) 中，其中一些名称与流中的名称不同（例如，`session.status_idled` 而不是 `session.status_idle`）。

<Tabs>
  <Tab title="用户事件">
    | 类型                        | 描述                                                                                                                                                            |
    | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `user.message`            | 包含文本、图像或文档内容的用户消息。                                                                                                                                            |
    | `user.interrupt`          | 在智能体运行时停止它。它不会结束任何[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#interrupt-a-session-with-runs-open)。                            |
    | `user.custom_tool_result` | 对智能体发起的自定义工具调用的响应。                                                                                                                                            |
    | `user.tool_confirmation`  | 当权限策略要求确认时，批准或拒绝智能体或 MCP 工具调用。                                                                                                                                |
    | `user.define_outcome`     | 定义一个供智能体努力达成的 [outcome（结果）](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)。                                                           |
    | `user.tool_result`        | 仅适用于使用 `self_hosted` [环境](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)的会话，您的集成负责提供 `agent_toolset` 结果。SDK 辅助工具和 CLI 会自动完成此操作。 |
  </Tab>

  <Tab title="智能体事件">
    | 类型                               | 描述                                                                                                                                                                                                                                                                                                      |
    | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `agent.message`                  | 代理响应内容块。                                                                                                                                                                                                                                                                                                |
    | `agent.thinking`                 | 表示代理正在通过扩展思考取得进展。这仅是一个进度信号，不包含思考内容。                                                                                                                                                                                                                                                                     |
    | `agent.tool_use`                 | 智能体调用预构建的智能体工具（bash、文件操作等）。携带 `evaluated_permission` (java, php: `evaluatedPermission`; csharp, go: `EvaluatedPermission`)，并且通常携带 `evaluation` (csharp, go: `Evaluation`)（请参阅[每次调用是如何被评估的](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#see-how-each-call-was-evaluated)）。 |
    | `agent.tool_result`              | 预构建代理工具的执行结果。                                                                                                                                                                                                                                                                                           |
    | `agent.mcp_tool_use`             | 智能体调用 MCP 服务器工具。携带 `evaluated_permission` (java, php: `evaluatedPermission`; csharp, go: `EvaluatedPermission`)，并且通常携带 `evaluation` (csharp, go: `Evaluation`)（请参阅[每次调用是如何被评估的](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#see-how-each-call-was-evaluated)）。            |
    | `agent.mcp_tool_result`          | MCP 工具的执行结果。                                                                                                                                                                                                                                                                                            |
    | `agent.custom_tool_use`          | 智能体调用您的某个自定义工具。请以 `user.custom_tool_result` 事件进行响应。                                                                                                                                                                                                                                                     |
    | `agent.thread_context_compacted` | 对话历史已被压缩以适应上下文窗口。                                                                                                                                                                                                                                                                                       |
    | `agent.thread_message_received`  | 在[多智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)会话中，来自另一个线程的消息到达了其流携带此事件的线程；在主线程上，表示某个智能体向会话所运行的智能体发送了报告或问题。                                                                                                                                                     |
    | `agent.thread_message_sent`      | 在[多智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)会话中，其流携带此事件的线程向另一个线程发送了消息；在主线程上，表示会话所运行的智能体向另一个智能体发送了任务或后续消息。                                                                                                                                                    |

    这些事件中的消息内容可能包含 `redacted` 内容块，即 `{"type": "redacted"}`：这是因 Anthropic 模型策略而被隐去的内容的占位符。该块不携带其他字段。Redacted 块仅出现在平台发出的内容中；包含此类块的用户事件将被拒绝并返回 400 错误。
  </Tab>

  <Tab title="会话事件">
    | 类型                                  | 描述                                                                                                                                                                                                                                                                                                                                                                                                |
    | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `session.status_running`            | 智能体正在积极处理。                                                                                                                                                                                                                                                                                                                                                                                        |
    | `session.status_idle`               | 智能体已完成当前任务，正在等待输入。包含一个 `stop_reason` (java, php: `stopReason`; csharp, go: `StopReason`)，指示智能体停止的原因。                                                                                                                                                                                                                                                                                              |
    | `session.status_rescheduled`        | 发生了瞬时错误，会话正在自动重试。                                                                                                                                                                                                                                                                                                                                                                                 |
    | `session.status_terminated`         | 会话已结束，原因是发生了不可恢复的错误或会话已被归档。                                                                                                                                                                                                                                                                                                                                                                       |
    | `session.deleted`                   | 会话已被删除。结束所有活动的事件流；此会话不会再发出任何事件。                                                                                                                                                                                                                                                                                                                                                                   |
    | `session.updated`                   | 会话更新请求更改了至少一个字段。仅包含已更改的字段。更新将在下一轮生效。                                                                                                                                                                                                                                                                                                                                                              |
    | `session.error`                     | 处理过程中发生错误。包含一个带类型的 `error` (csharp, go: `Error`) 对象，其中带有 `retry_status` (java, php: `retryStatus`; csharp, go: `RetryStatus`)。                                                                                                                                                                                                                                                                    |
    | `session.usage`                     | 会话累计用量和跟踪的标价成本的快照。携带会话的用量总计，以及会话的[预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)（如果有）。                                                                                                                                                                                                                                                                                        |
    | `session.thread_created`            | 已创建一个[多智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)线程。它包含 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`)：对于[工作流运行的线程](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#a-runs-threads)，其值为该运行的 ID；对于任何其他线程，其值为 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `""`)。 |
    | `session.thread_status_running`     | 某个会话线程开始执行。每个会话都会为其主线程发出此事件；在[多智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)会话中，子线程的状态转换也会交叉发布到主事件流。                                                                                                                                                                                                                                                       |
    | `session.thread_status_idle`        | 会话线程已完成其轮次，正在等待输入。包含 `stop_reason` (java, php: `stopReason`; csharp, go: `StopReason`)。                                                                                                                                                                                                                                                                                                           |
    | `session.thread_status_rescheduled` | 某个会话线程遇到瞬时错误，正在自动重试。                                                                                                                                                                                                                                                                                                                                                                              |
    | `session.thread_status_terminated`  | 某个会话线程已终止，不再接受任何输入，例如因为它已被归档或遇到了不可恢复的错误。顾问线程在其咨询结束时也会终止。服务器会自行归档工作流运行的线程。                                                                                                                                                                                                                                                                                                                         |

    在其智能体已开启[动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#dynamic-workflows)的会话中，以下 `workflow_run.*` 事件会到达会话的流：

    | 类型                            | 描述                                                                                                                                                                                                                                                                                                                                 |
    | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `workflow_run.created`        | 已创建一个工作流运行。包含 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`)、运行的 `name` (csharp, go: `Name`) 和 `description` (csharp, go: `Description`)，以及 `phases` (csharp, go: `Phases`)（为该运行声明的阶段）。随后会有一个状态事件，但在中断之后可能不会有。                                                                       |
    | `workflow_run.status_running` | 工作流运行正在运行。在运行开始执行时发送（这可能在 `workflow_run.created` 之后一段时间），并在因预算暂停后恢复时再次发送。中断后的恢复可能不会发送此事件。从一开始就处于空闲状态的运行可能会先收到 `workflow_run.status_idle`。包含 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`)。                                                                                          |
    | `workflow_run.status_idle`    | 工作流运行进入空闲状态：它已暂停，例如因达到会话的预算。中断后的暂停可能不会发送此事件。包含 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`)。                                                                                                                                                                                       |
    | `workflow_run.status_ended`   | 工作流运行已结束。包含 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`) 和 `result` (csharp, go: `Result`)，其 `type` (csharp, go: `Type`) 说明运行是如何结束的。                                                                                                                                               |
    | `workflow_run.error`          | 工作流运行的错误，或服务器拒绝的启动。以错误结束的运行会在其 `workflow_run.status_ended` 之前收到此事件，且错误相同。包含 `error` (csharp, go: `Error`) 和 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`)，当未创建任何运行时，后者为 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `""`)。                                   |
    | `workflow_run.phase_started`  | 工作流运行进入了其某个阶段。包含 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`) 和 `workflow_run_phase_id` (java: `workflowRunPhaseId`; php: `workflowRunPhaseID`; csharp, go: `WorkflowRunPhaseID`)。                                                                                                 |
    | `workflow_run.phase_ended`    | 工作流运行离开了一个阶段。包含 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`)、`workflow_run_phase_id` (java: `workflowRunPhaseId`; php: `workflowRunPhaseID`; csharp, go: `WorkflowRunPhaseID`) 和 `phase_started_id` (java: `phaseStartedId`; php: `phaseStartedID`; csharp, go: `PhaseStartedID`)。 |
  </Tab>

  <Tab title="Span 事件">
    Span 事件是可观测性标记，用于包裹活动以进行计时和用量跟踪。

    | 类型                                | 描述                                                                                                                                                                                                 |
    | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `span.model_request_start`        | 模型推理调用已开始。                                                                                                                                                                                         |
    | `span.model_request_end`          | 模型推理调用已完成。包含带有令牌计数的 `model_usage` (java, php: `modelUsage`; csharp, go: `ModelUsage`)。                                                                                                             |
    | `span.outcome_evaluation_start`   | [Outcome（结果）](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)评估已开始。                                                                                                         |
    | `span.outcome_evaluation_ongoing` | 正在进行的 [outcome（结果）](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)评估期间的心跳。                                                                                                 |
    | `span.outcome_evaluation_end`     | 一个 [outcome（结果）](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)评估周期已完成。`needs_revision` 结果表示随后还有另一个周期；`satisfied`、`max_iterations_reached`、`failed` 和 `interrupted` 为终止状态。 |
  </Tab>

  <Tab title="系统事件">
    | 类型               | 描述                                                                                                                                                                                                                         |
    | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `system.message` | 追加特权系统级上下文，该上下文适用于随附的轮次及所有后续轮次。有关接受此事件的模型，请参阅[受支持的模型](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#supported-models)。在不受支持的主模型上，该事件将被拒绝并返回 `model_does_not_support_mid_conversation_system`。 |
  </Tab>

  <Tab title="事件增量">
    事件增量是仅限流式传输的预览事件。它们在通过 `event_deltas` (curl: `event_deltas[]`; cli: `--event-delta`; java, php: `eventDeltas`; csharp, go: `EventDeltas`) 参数选择启用的流连接（会话级或按线程）上发出，并且永远不会持久化到会话的事件历史中。有关如何选择启用、累积和协调它们，请参阅[使用事件增量预览响应](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas)。

    | 类型            | 描述                                                                                               |
    | ------------- | ------------------------------------------------------------------------------------------------ |
    | `event_start` | 一个预览事件已开始生成。携带即将到来的事件的 `type` (csharp, go: `Type`) 和 `id` (csharp, go: `ID`)。仅限流式传输，永不持久化。       |
    | `event_delta` | 预览事件的增量内容，由 `event_id` (java: `eventId`; php: `eventID`; csharp, go: `EventID`) 标识。仅限流式传输，永不持久化。 |
  </Tab>
</Tabs>

## 自托管 worker

有关驱动 `self_hosted` 环境的预构建 worker 的 `ant beta:worker` CLI 标志、环境变量、文件系统路径和 SDK 辅助工具选项，请参阅[自托管 worker 参考](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference)。

## 支持的 MCP 服务器类型

Claude Managed Agents 可连接到公开 HTTP 端点的[远程 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/remote-mcp-servers)，或通过 [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)连接到私有 MCP 服务器。服务器应支持 MCP 协议的 streamable HTTP 传输；仅支持已弃用的 SSE 传输的服务器仍可通过自动回退正常工作。有关在智能体上声明服务器的信息，请参阅 [MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)。

有关 MCP 及构建 MCP 服务器的更多信息，请参阅 [MCP 文档](https://modelcontextprotocol.io)。

## 速率限制

Managed Agents 端点按组织进行速率限制：

| 操作                  | 限制            |
| ------------------- | ------------- |
| 创建类端点（例如智能体、会话和环境）  | 每分钟 300 个请求   |
| 读取类端点（例如检索、列出和流式传输） | 每分钟 1,200 个请求 |

组织级别的[支出限制和使用层级速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)同样适用。

## 品牌指南

对于集成 Claude Managed Agents 的合作伙伴，使用 Claude 品牌是可选的。在您的产品中提及 Claude 时：

**允许：**

* “Claude Agent”（下拉菜单的首选）
* “Claude”（当位于已标记为"Agents"的菜单中时）
* “\{YourAgentName} Powered by Claude”（如果您已有智能体名称）

**不允许：**

* “Claude Code” 或 “Claude Code Agent”
* “Claude Cowork” 或 “Claude Cowork Agent”
* Claude Code 品牌的 ASCII 艺术或模仿 Claude Code 的视觉元素

您的产品应保持自己的品牌，不应看起来像是 Claude Code、Claude Cowork 或任何其他 Anthropic 产品。如有关于品牌合规的问题，请联系 Anthropic [销售团队](https://www.anthropic.com/contact-sales)。
