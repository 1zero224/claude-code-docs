---
title: 审查和监控智能体会话
url: https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring
description: 可选择使用离线作业审查每个会话的事件。关注需要处理的会话的 webhook，并了解 Managed Agents 会保留哪些内容以及保留多长时间。
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

您可以选择使用离线作业审查每个会话的事件。关注需要处理的会话的 webhook，并删除不再需要的会话。若要在智能体运行之前限制其可执行的操作，请参阅[保护您的 Managed Agents 部署](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment)。

## 可选：让离线审查器阅读每份会话记录

“Offline reviewer”（离线审查器）是您运行的一个作业，它独立于其所审查的会话，在每个会话的事件写入后读取这些事件。长会话的事件太多，人工无法逐一阅读，因此该作业让 Claude 来读取它们。审查会标记超出智能体被授予权限范围的行为。有关审查提示、设置步骤和可运行的代码，请参阅 Claude Cookbook 中的[安全审查示例](https://platform.claude.com/cookbook/managed-agents-cma-agent-security-review)。

## 监控会话并查看保留策略

订阅在会话需要处理时通知您的 webhook，并删除不再需要的会话。验证每次投递的签名。请参阅[验证签名](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#verify-the-signature)。

| 需要关注的内容            | 查看位置                                                                                                                                                                                          |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 进入空闲状态的会话，例如等待批准   | `session.status_idled` webhook。每当会话因任何原因进入空闲状态时都会触发                                                                                                                                           |
| 达到预算或终止的会话         | `session.budget_reached` 和 `session.status_terminated` webhook                                                                                                                                |
| 未能启动会话的计划运行        | `deployment_run.failed` 和 `deployment.paused` webhook                                                                                                                                         |
| 无法刷新的 MCP OAuth 凭据 | `vault_credential.refresh_failed` webhook                                                                                                                                                     |
| 服务器拒绝或暂停的工具调用      | `agent.tool_use` 和 `agent.mcp_tool_use` 事件上的 `evaluated_permission`。在多智能体会话中，还要列出每个线程的事件                                                                                                      |
| 智能体在会话中执行的操作       | Claude Console 会话查看器                                                                                                                                                                          |
| 超出智能体被授予权限的行为      | 您运行的可选离线审查作业，用于读取每个会话的事件。请参阅[可选：让离线审查器阅读每份会话记录](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring#optionally-have-an-offline-reviewer-read-every-transcript) |

要查看会话进入空闲状态的原因，请列出其事件，并将 `types` 设置为 `session.status_idle`、`order` 设置为 `desc`、`limit` 设置为 `1`。该事件名为 `session.status_idle`，而 webhook 名为 `session.status_idled`。如果 `stop_reason.type` 为 `requires_action`，则会话正在等待您处理，`stop_reason.event_ids` 会列出它正在等待的事件。要响应这些事件，请参阅[回答使会话暂停的工具调用](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#answer-tool-calls-that-pause-the-session)。在自托管沙箱中，`agent.tool_use` 事件可能正在等待您的沙箱返回工具结果。当工作流运行处于打开状态时，工具调用可能会在会话保持 `running` 状态的同时等待您的响应。请在每个调用的事件到达时进行响应。请参阅[运行处于打开状态时](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#while-a-run-is-open)。

webhook 投递可能会丢失，因此还要按计划列出您的会话并检查空闲的会话。

请参阅[订阅 webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks) 和[检查会话并跟踪用量](https://platform.claude.com/docs/zh-CN/managed-agents/session-observability#inspect-a-session-in-the-console)。

以下列表显示了 Managed Agents 会保留哪些内容以及保留多长时间：

* **会话历史：** 保留到您删除会话为止。归档会话会保留其历史。若要保留会话的事件，请在删除会话之前使用事件 API 将其复制到您自己的日志中。处于 `running` 状态的会话无法删除。请参阅[删除会话](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#deleting-a-session)。
* **沙箱状态：** 在沙箱创建后保留 30 天。
* **会话生成的文件：** 随会话一起删除。请先下载您想保留的文件。
* **您上传的文件、记忆存储和保管库：** 不会随会话一起删除。请单独删除它们。

使用自托管沙箱时，您自己基础设施上的工具输入和输出副本由您自行决定保留或删除。

Managed Agents 目前不符合零数据保留（Zero Data Retention）或 HIPAA 商业伙伴协议（BAA）覆盖的条件。请参阅 [API 与数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)。
