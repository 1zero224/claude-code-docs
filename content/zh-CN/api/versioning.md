---
title: API 版本
url: https://platform.claude.com/docs/zh-CN/api/versioning
description: "在发出 API 请求时，您必须发送 `anthropic-version` 请求头。例如，`anthropic-version: 2026-10-21`。如果您使用的是[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)，它会为您设置此请求头。"
---

对于任何给定的 API 版本，Anthropic 会保留：

* 现有的输入参数
* 现有的输出参数

但是，Anthropic 可能会执行以下操作：

* 添加额外的可选输入
* 向输出中添加额外的值
* 更改特定错误类型的触发条件
* 向类枚举的输出值添加新的变体（例如，流式传输事件类型）

一般而言，如果您按照本参考文档中所述的方式使用 API，Anthropic 不会破坏您的使用方式。

## 版本历史

Anthropic 建议尽可能使用最新的 API 版本。

* `2026-10-21`

  * [迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21) 介绍了如何升级。

  * 可在 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。在 Amazon Bedrock 上，它适用于 Claude 4.7 及更高版本的模型。对于 Bedrock 上较早的 Claude 模型，请[继续使用您当前使用的 API 版本](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#availability)。

  * Messages API：

    * 将 `document` 块直接放在 `assistant` 消息中的请求会返回 400 错误。
    * 在 `user` 消息中发送带有非空 `citations` 数组的 `text` 块（无论是直接发送还是放在 `search_result` 块内）都会返回 400 错误。
    * `temperature`、`top_p` 和 `top_k` 请求参数已被移除。发送其中任何一个参数的请求在所有模型上都会返回 400 错误。

  * Admin API：

    * [RBAC 组](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/list)不再返回 `roles`。请改用 `role_ids`。
    * [RBAC 组成员](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/members/list)以及移除成员时的响应不再返回 `group_id`。请改用 `rbac_group_id`。
    * [RBAC 角色](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_roles/list)不再返回 `name`。请改用 `display_name`。
    * [组织](https://platform.claude.com/docs/zh-CN/api/organization/rate_limits/list)和[工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/rate_limits/list)的速率限制条目，以及[设置工作区速率限制](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/rate_limits/update)的响应，不再返回 `group_type`。请改用 `group.type`。
    * [API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/list)不再返回 `workspace_id`。请改用 `scope.workspace_id`。它包含密钥所属工作区的 ID，包括默认工作区（Default Workspace）。适用于所有工作区的密钥没有 `scope.workspace_id`。

  * Claude Enterprise Analytics API：

    * 按用户的[使用量](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/user_usage_report/list)和[成本](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/user_cost_report/list)报告不再包含 `actor.email`。请改用 `actor.email_address`。
    * [活动摘要](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/summaries/list)响应不再包含 `summaries`。请改用 `data`。

* `2023-06-01`

  * [流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)服务器发送事件（SSE）的新格式：

    * 补全内容是增量式的。例如，`" Hello"`、`" my"`、`" name"`、`" is"`、`" Claude." `，而不是 `" Hello"`、`" Hello my"`、`" Hello my name"`、`" Hello my name is"`、`" Hello my name is Claude."`。
    * 所有事件均为[命名事件](https://developer.mozilla.org/en-US/Web/API/Server-sent%5Fevents/Using%5Fserver-sent%5Fevents#named%5Fevents)，而非[仅数据事件](https://developer.mozilla.org/en-US/Web/API/Server-sent%5Fevents/Using%5Fserver-sent%5Fevents#data-only%5Fmessages)。
    * 移除了不必要的 `data: [DONE]` 事件。

  * 移除了响应中遗留的 `exception` 和 `truncated` 值。

* `2023-01-01`：初始版本。
