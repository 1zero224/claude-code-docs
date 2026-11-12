---
title: Claude Platform 发布说明
url: https://platform.claude.com/docs/zh-CN/release-notes/overview
description: Claude Platform 的更新，包括 Claude API、客户端 SDK 和 Claude Console。
---

Claude Platform 发布说明列出了 Claude API、客户端 SDK 和 Claude Console 的变更，按时间从新到旧排列。

<Tip>
  有关 Claude Apps 的发布说明，请参阅 [Claude 帮助中心中的 Claude Apps 发布说明](https://support.claude.com/en/articles/12138966-release-notes)。

  有关 Claude Code 的更新，请参阅 `claude-code` 仓库中的[完整 CHANGELOG.md](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)。
</Tip>

### 2026 年 11 月 9 日

* 我们推出了 **Claude Fable 5.2**（`claude-fable-5-2`），专为高要求的推理和长周期智能体工作而打造。**Claude Mythos 5.2**（`claude-mythos-5-2`）提供相同的能力，但仅面向通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织。两个模型都具有 [1M 令牌的上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)、128k 最大输出令牌数，以及始终开启的[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。它们的价格为每 MTok 输入 $8 美元、每 MTok 输出 $40 美元（Claude Fable 5.1 和 Claude Mythos 5.1 为每 MTok 输入 $10 美元、每 MTok 输出 $50 美元）。Claude Fable 5.2 可在 Claude API、[Amazon Bedrock（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。有关 API 变更和迁移指南，请参阅 [Claude Fable 5.2 的新功能](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2)。有关特定于模型的提示模式，请参阅[为 Claude Fable 5.2 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2)。
* Claude Fable 5.2 和 Claude Mythos 5.2 对每个账户（无论其创建时间）都强制执行 [“prefix check”（前缀检查）](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)。Claude Fable 5.1 默认对 2026 年 8 月 31 日或之后创建的账户返回该错误（请参阅[未设置 `prefix_mismatch_behavior` 时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)）。如果您的代码编辑了 `system` 提示、`tools` 或较早的消息，然后又发回来自后续轮次的 Claude Fable 5.2 或 Claude Mythos 5.2 思考块，该请求将返回 400 错误。在切换之前，请[检查您的代码是否编辑了前缀](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-preserved-thinking)。
* 在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 和 Claude Mythos 5.2 可以读取其他模型的思考块。因此在这些平台上，转移到这两个模型中任一模型的对话会保留其推理。这些其他模型包括 Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5。
* 没有其他模型可以读取来自 Claude Fable 5.2 或 Claude Mythos 5.2 的思考块。当对话从这两个模型中的任一模型转移到这对模型之外的模型时，API 会丢弃这些思考块。这包括转移到 Claude Opus 5.5 或 Claude Fable 5.1。请求仍会成功。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。
* Claude Fable 5.2 和 Claude Mythos 5.2 通过 `computer_toolset_20260801` 工具集支持 “computer use”（计算机使用）。声明较早的 `computer_20251124` 工具的请求会返回 400 错误（适用于除 Amazon Bedrock 以外的所有平台，在 Amazon Bedrock 上较早的工具仍然可用）。请参阅[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)。
* 使用 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)（该工具目前处于 beta 阶段）时，Claude Fable 5.2 或 Claude Mythos 5.2 执行器仅接受 Claude Fable 5.2 或 Claude Mythos 5.2 作为其顾问。任何其他顾问都会返回 400 错误，因此当您将执行器迁移到这两个模型时，请将顾问的 `model` 设置为其中之一。每个执行器模型都接受 Claude Fable 5.2 作为其顾问。请参阅[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#model-compatibility)。
* 与 Claude Fable 5.1 和 Claude Mythos 5.1 一样，这两个模型都需要 30 天的数据保留。除非获得 Anthropic 的明确授权，否则它们在 “zero data retention”（零数据保留）下不可用。请参阅[特定模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。
* Claude Fable 5.2 和 Claude Mythos 5.2 生成的思考块仅在生成它们的账户或与其关联的账户中有效。请参阅[保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* [Dreams](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)（研究预览版）现已支持 Claude Fable 5.2。请参阅[支持的模型](https://platform.claude.com/docs/zh-CN/managed-agents/dreams#feature-availability)。
* 我们已在 Claude API 上的 Messages API 请求中为 Claude Fable 5.2 推出了[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。快速模式的每秒输出令牌数最高可提升 2.5 倍，价格为每 MTok 输入 $16 美元、每 MTok 输出 $80 美元。在您的请求中设置 `speed: "fast"` 即可，无需 beta 标头。

### 2026 年 11 月 5 日

* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 现已包含 Security posture 数据，面向 Claude Enterprise 组织提供 beta 版本。[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)会记录 claude.ai **组织设置**中 **Security posture** 页面上对声明的更改。`GET /v1/compliance/organizations/{organization_id}/security_posture` 端点会返回组织当前的 Security posture 结果。请参阅 [Compliance API 中的 Security posture](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview)。
* 您现在可以在 Claude Console 中为您的组织设置 “IP allowlist”（IP 允许列表），该功能处于 beta 阶段。来自不在您列表中的地址、发往受覆盖服务的请求将被拒绝，从而限制泄露的 API 密钥可在何处使用。您可以添加 IPv4 和 IPv6 地址或 CIDR 范围，每个组织最多 1,000 个。请参阅 [IP 允许列表](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist)，了解覆盖范围以及设置方法。
* 您现在可以使用 Admin API 在所有位置将您的 Claude Enterprise（claude.ai）组织成员登出 Claude。此功能处于 beta 阶段。请参阅 [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization)。该成员会保留在组织中，并且可以重新登录。有关 beta 标头以及该请求不会结束的内容，请参阅[在所有位置登出成员](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#sign-a-member-out-everywhere)。

### 2026 年 11 月 3 日

* 在 [Claude 如何标记 AI 生成的内容](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)中列出的平台上，Claude Mythos 5、Claude Opus 4.7 和 Claude Sonnet 4.6 生成的文本带有 Anthropic 的文本水印。在 Claude Sonnet 4.6 上，如果请求将 `temperature`、`top_p` 或 `top_k` 设置为非默认值，则为该请求生成的文本可能不带有水印。请参阅[文本水印](https://platform.claude.com/docs/zh-CN/build-with-claude/text-watermark)。

### 2026 年 11 月 2 日

* Messages API 上的长对话和智能体循环现在可以通过 “message threads”（[消息线程](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads)）发送更小的请求并缩短首个令牌的响应时间，目前为 beta 版，需使用 `message-threads-2026-08-12` beta 标头。API 会在服务器上保留对话，因此之后的每个请求只需发送新消息。线程会在其最后一次请求的 5 分钟后过期，并且最迟会在其最后一次使用的 1 小时后被删除。消息线程适用于 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、Claude Models on Google Cloud (Anthropic managed) 以及 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)。

### 2026 年 10 月 30 日

* 我们已弃用 Claude Opus 4.8 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)，并将于 2026 年 11 月 30 日移除。移除后，对 `claude-opus-4-8` 发送的带有 `speed: "fast"` 的请求将返回错误。如需继续使用快速模式，请迁移到 [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8)。详情请参阅[快速模式的模型可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#supported-models)。

### 2026 年 10 月 29 日

* 我们推出了 beta 版[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)，使用 `analytics-usage-cost-2026-08-24` beta 标头。其 `/v1/organizations/analytics/usage` 和 `/v1/organizations/analytics/cost` 端点报告 Claude Console 和 Claude Enterprise 组织的令牌用量和成本。您可以按模型、工作区、API 密钥以及每个请求背后的用户或服务账户等维度筛选和分组结果。
* [用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api) 现已成为旧版 API，[Claude Enterprise Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api) 的 `usage_report`、`cost_report`、`user_usage_report` 和 `user_cost_report` 端点也是如此。它们仍然全部受支持。要迁移现有集成，请参阅[从用量和成本 API 迁移](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#migrate-from-usage-cost-api)或[从 Claude Enterprise 用量和成本报告迁移](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#migrate-from-enterprise-analytics-reports)。

### 2026 年 10 月 28 日

* 我们已将第一方 Claude API 上 Messages API 和令牌计数 API 的[最大请求大小](https://platform.claude.com/docs/zh-CN/api/overview#request-size-limits)从 32 MB 提高到 128 MB。

### 2026 年 10 月 23 日

* [Claude Tag API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api) 现已面向 Claude Enterprise 组织推出 beta 版，需使用 `experimental-claude-tag-2026-10-21` beta 标头。其 `/v1/organizations/claude_tag/` 端点可读取和更改 Slack 频道以及 Slack 工作区中的 [Claude Tag](https://claude.com/docs/claude-tag/overview) 设置，显示某个频道中适用哪些设置以及每项设置的来源，并可附加访问包。请使用由您的主要所有者在 claude.ai 中创建、具有 `read:claude_tag` 或 `write:claude_tag` 范围的 Admin API 密钥来调用它。Claude Platform（Claude Console）组织无法使用此 API。

### 2026 年 10 月 22 日

* 我们推出了 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)，这是由 Anthropic 运营、通过您的 Google Cloud 项目访问的 Claude API。Google Cloud IAM 负责控制访问权限，Google Cloud Marketplace 负责处理计费。
* [快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)现已在 Claude Models on Google Cloud (Anthropic managed) 上可用于 Claude Opus 5.5。

### 2026 年 10 月 21 日

* 我们发布了 API 版本 `2026-10-21`。[迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21) 介绍了如何升级。
* 我们发布了新版本的[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 和 [`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart)，它们默认发送 API 版本 `2026-10-21`：Python SDK 2.0.0、TypeScript SDK 1.0.0、C# SDK 13.0.0、Go SDK 2.0.0、Java SDK 3.0.0、PHP SDK 1.0.0、Ruby SDK 2.0.0 以及 `ant` CLI 2.0.0。请参阅[切换到 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。
* 新的 SDK 和 CLI 版本还包含不依赖于 API 版本的破坏性变更，并且它们采用 Apache 2.0 许可证，而不再采用 MIT 许可证。请参阅 [SDK 和 CLI 变更](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sdk-changes)。
* 我们为[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)添加了 `enforcement_status` 和 `enforcement_actions` 字段，以便您查看 Anthropic 是否已限制或封禁您的某个终端客户。要查找存在未解除的限制或封禁的配置文件，请使用新的 `enforcement_status[]` 筛选器[列出您的配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/list)。请参阅[检查 Anthropic 是否已限制或封禁某个配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#check-whether-anthropic-has-restricted-or-banned-a-profile)。

### 2026 年 10 月 20 日

* [推理钩子](https://platform.claude.com/docs/zh-CN/manage-claude/inference-hooks)已面向 Claude Enterprise 组织结束 beta 阶段。
* 我们推出了[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)，让您可以告诉 Anthropic 每个 API 请求是为您的哪个终端客户发出的。这样，如果您运营平台或网关，或转售 API 访问权限，安全执行措施就可以仅作用于单个终端客户，而不是您的整个组织。SDK 和 `ant` CLI 支持新的 [`/v1/user_profiles` 端点](https://platform.claude.com/docs/zh-CN/api/user_profiles)。
* 我们在 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 中添加了[组织 webhook](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks)，现为 beta 版，需使用 `webhooks-2026-10-07` beta 标头。其 [`/v1/organizations/webhooks` 端点](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks)可为两种事件类型注册 webhook 端点：一种在 Anthropic 限制或封禁您组织的某个用户配置文件时触发，另一种在该限制或封禁结束时触发。如需申请访问权限，请联系您的 Anthropic 客户团队。
* 组织管理员现在可以在 Claude Console 的 [**功能**](https://platform.claude.com/settings/capabilities) 页面上为其组织关闭 “web fetch”（网页抓取）。关闭网页抓取后，包含网页抓取工具的 Messages API 请求会失败并返回 400 错误。在 Claude Managed Agents 会话中，`web_fetch` 调用则会向智能体返回错误结果。开启网页抓取时，管理员还可以在同一页面上为网页抓取设置允许或阻止的域名。请参阅[如何使用网页获取](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#how-to-use-web-fetch)和[域名过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#domain-filtering)。
* 在 Claude Managed Agents 中，[`web_fetch` 工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#available-tools)现在遵循您的组织在 Claude Console 的 [**功能**](https://platform.claude.com/settings/capabilities) 页面上设置的网页抓取域名限制。URL（包括通过重定向到达的 URL）必须同时通过组织的列表以及智能体 `web_fetch` 工具上的 `allowed_domains` 或 `blocked_domains` 列表。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。
* 在 Claude Managed Agents 中，会话遵循您的组织在 Claude Console 的 [**功能**](https://platform.claude.com/settings/capabilities) 页面上的网页搜索设置：自 10 月 6 日起遵循**允许网页搜索**设置，自 10 月 14 日起遵循其网页搜索域名限制。智能体只会收到组织的域名限制和智能体 [`web_search` 工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#available-tools)上的 `allowed_domains` 或 `blocked_domains` 列表都允许的搜索结果。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

### 2026 年 10 月 15 日

* 我们在 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 中添加了 `capabilities.thinking.types.between_tools`。`GET /v1/models` 和 `GET /v1/models/{model_id}` 现在会报告每个模型是否接受用于关闭前置思考的 `thinking: {type: "between_tools"}`。请参阅[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api)。
* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 现在会返回每位组织成员在 claude.ai 中保存为 **Claude 指令**以及在 Cowork 中保存为**全局指令**的文本，面向 Claude Enterprise 组织以 beta 形式提供。使用您现有的 Compliance Access Key 和 `read:compliance_user_data` 范围调用 `GET /v1/compliance/organizations/{org_uuid}/user_preferences`。请参阅[列出用户偏好](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-user-preferences)。
* Claude Opus 5.5 的 [Fast mode](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)（快速模式）已结束研究预览，您不再需要申请访问权限。对 Claude Opus 5.5 的请求不再需要 `fast-mode-2026-02-01` beta 标头，仍发送该标头的请求也可继续正常工作。Claude Opus 5.5 现在在 Start、Build 和 Scale 使用层级中都有默认的快速模式速率限制。Claude Opus 5 和 Claude Opus 4.8 的快速模式仍处于研究预览阶段。向这些模型发送快速模式请求时，请继续发送该 beta 标头。
* Claude Opus 5.5 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)现已在 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上可用，适用于托管在 Azure 上的部署。

### 2026 年 10 月 14 日

* 我们推出了 beta 版的 [OpenTelemetry 导出](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry)，让您可以在可观测性平台中追踪 Messages API 请求和 Claude Managed Agents 会话。Claude SDK 在 Python、TypeScript、C#、Go、Java、PHP 和 Ruby 中发出 span。要从 API 获取 span，请在 Claude Console 中[添加 OpenTelemetry 目标](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#add-a-destination)。
* Claude Managed Agents [计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)现已[在 Claude Platform on AWS 上可用](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#claude-managed-agents)。

### 2026 年 10 月 9 日

* 您现在可以让 Claude Managed Agents 智能体使用[动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#dynamic-workflows)，该功能为 beta 版，需使用 `managed-agents-2026-04-01` beta 标头。对于包含许多部分的工作（例如审阅数百份文档），智能体可以编写一个工作流。工作流是一个分阶段运行多个智能体并合并其结果的程序。服务器会在后台将其作为一次 “workflow run”（工作流运行）来执行。要[开启动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#turn-on-dynamic-workflows)，请将智能体的 `multiagent` 字段设置为 `{"type": "multiagent_20261001", "workflows": {"type": "enabled"}}`。要引导智能体，请在其系统提示中告诉它何时启动运行。通过会话事件流上的 `workflow_run.*` 事件跟踪每次运行。请参阅[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)。

### 2026 年 10 月 8 日

* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的聊天端点现在还会返回来自统一 Claude 体验的聊天，该功能面向 Claude Enterprise 组织提供 beta 版，使用您现有的 Compliance Access Key 即可。请参阅[检索和删除聊天、文件及项目](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-content-data)。
* Compliance API 现在可以将 Claude Docs 文档下载为 Word 文件，该功能为 beta 版。请参阅[检索文件和 artifacts](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-content-data#retrieve-files-and-artifacts)。

### 2026 年 10 月 7 日

* 我们已将 Claude Sonnet 5.5 上提示缓存读取的价格从每百万令牌 0.20 美元降至 0.10 美元：即基础输入价格的 0.05 倍，而非 0.1 倍。缓存写入及所有其他价格保持不变。请参阅[提示缓存定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#prompt-caching)。
* 我们推出了 **Claude Haiku 5.5**（`claude-haiku-5-5`），这是我们针对大批量和延迟敏感型工作调优的最强模型。它拥有 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)、128k 最大输出令牌，以及支持 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)的[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。它可在 Claude API、[Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。请参阅 [Claude Haiku 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5)。
* 为 Claude Haiku 4.5 编写的代码在 Claude Haiku 5.5 上可能会出错。手动扩展思考（`budget_tokens`）、采样参数 `temperature`、`top_p` 和 `top_k`，以及助手消息预填充，都可能返回 400 错误。`computer_20250124` 也会返回 400 错误：在 Claude API 和 Google Cloud 上请使用 `computer_toolset_20260801` 工具集，在 Amazon Bedrock 上请使用 `computer_20251124`。在更改 `system`、`tools` 或之前的轮次后回传思考块的请求也可能返回 400 错误。自适应思考默认开启，因此响应可能以 `thinking` 块开头，除非您将 `thinking.display` 设置为 `"summarized"`，否则其文本会被省略。Claude Haiku 5.5 还使用一种会为相同文本产生更多令牌的分词器。有关每项变更，请参阅 [Claude Haiku 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide)。有关特定于模型的提示模式，请参阅[为 Claude Haiku 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5)。
* Python SDK 1.12.0 和 TypeScript SDK 0.132.0 以 beta 形式为[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)和[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)添加了类。您只需继承其中一个类，并针对您自己的浏览器或桌面自动化为每个工具编写一个方法。SDK 会运行工具循环、您为浏览器设置的 URL 和文件策略，以及您的审批回调。请参阅[使用 SDK 工具集进行浏览器和计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk)。
* Claude Max、Ultra 和 Team 计划现在包含每月 API 额度。要了解如何领取，请参阅 [Max、Ultra 和 Team 计划的 API 额度](https://platform.claude.com/docs/zh-CN/about-claude/api-credits-for-subscribers)。
* 在 Claude Managed Agents 中，使用 `limited` 网络的云环境现在也会将其 `allowed_hosts` 应用于 `web_search` 和 `web_fetch` 工具。对 `allowed_hosts` 不匹配的主机上的 URL 进行 `web_fetch` 调用时，会向智能体返回 `url_not_allowed` 错误结果。`web_search` 会省略来自此类主机的结果。当 `allowed_hosts` 未列出任何主机时，这两个工具都不会返回页面或搜索结果。`allow_package_managers` 和 `allow_mcp_servers` 不会为这些工具添加任何主机。要让这些工具访问某个主机，请将其添加到 `allowed_hosts`，这也会向沙箱开放该主机。`unrestricted` 网络和自托管环境不会限制这些工具。请参阅[环境网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)。
* 使用 `limited` 网络时，如果已启用的 Web 工具的 `allowed_domains` 中有不在 `allowed_hosts` 范围内的条目，创建会话将失败并返回 400 错误。添加此类条目的会话更新也会如此。除非 `allowed_hosts` 条目以 `*.` 开头，否则它只匹配一个确切的主机，因此 `docs.example.com` 不在 `["example.com"]` 范围内。要修复该错误，请将该主机添加到 `allowed_hosts`，或从 `allowed_domains` 中删除该条目。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。
* 在 Claude Managed Agents 中，[`web_fetch` 工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#available-tools)现在只获取已在会话中出现过的 URL，例如出现在用户消息文本中、`web_search` 结果中，或 `web_fetch` 之前返回的页面中的 URL。这降低了数据外泄的风险。仅出现在 Claude 自身输出、智能体的系统提示、附加文档，或 `bash`、`read` 或 MCP 工具等工具输出中的 URL 不算在内：对其进行 `web_fetch` 调用会向智能体返回 `url_not_in_prior_context` 错误结果。要让智能体获取某个 URL，请在 `user.message` 事件的文本中发送该 URL。

### 2026 年 10 月 6 日

* 我们在 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 中添加了 `capabilities.server_tools`。`GET /v1/models` 和 `GET /v1/models/{model_id}` 现在会报告每个模型是否接受网页搜索和代码执行工具。要检查模型是否接受代码执行工具，请读取 `capabilities.server_tools.code_execution`。顶层的 `capabilities.code_execution` 报告代码是否可以调用您请求中的其他工具。请参阅[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api)。

### 2026 年 10 月 5 日

* 我们在 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 中添加了 `capabilities.thinking.types.disabled`。`GET /v1/models` 和 `GET /v1/models/{model_id}` 现在会报告每个模型是否接受用于关闭思考的 `thinking: {type: "disabled"}`。请参阅[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api)。

### 2026 年 10 月 1 日

* 我们在 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 中添加了 `line` 字段。`GET /v1/models` 和 `GET /v1/models/{model_id}` 现在会返回每个模型所属的模型系列。例如，Claude Opus 4.5 和 Claude Opus 4.6 都报告为 `opus`。使用 `line` 可以对模型进行分组，而无需解析其 ID。对于不属于任何系列的模型，`line` 为 `null`。请参阅[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api)。
* [梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)（研究预览版）现在支持 Claude Opus 5.5、Claude Fable 5.1 和 Claude Sonnet 5.5。请参阅[支持的模型](https://platform.claude.com/docs/zh-CN/managed-agents/dreams#feature-availability)。

### 2026 年 9 月 30 日

* 在 Python SDK 1.10.0、TypeScript SDK 0.130.0、C# SDK 12.52.0、Go SDK 1.77.0、Java SDK 2.67.0、PHP SDK 0.53.0、Ruby SDK 1.75.0 以及 `ant` CLI 1.37.0 版本中，用于组织信息、成员、邀请、工作区、API 密钥、速率限制、服务账户、工作负载身份联合、客户管理的加密密钥和合规设置的 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 方法已结束 beta：请在 `client.organization` 和 `ant organization` 下调用它们。这些版本也保留了 `client.beta.organization` 和 `ant beta:organization`。
* 我们宣布弃用 Claude Sonnet 4.5 模型（`claude-sonnet-4-5-20250929`），计划于 2026 年 11 月 30 日在 Claude API 上停用。我们建议迁移到 [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migrating-from-sonnet-45)。更多信息请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。

### 2026 年 9 月 28 日

* 我们推出了 **Claude Sonnet 5.5**（`claude-sonnet-5-5`）。它可在 Claude API、[Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。有关其上下文窗口、输出限制和价格，请参阅 [Claude Sonnet 5.5 模型页面](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview)。
* 为 Claude Sonnet 5 编写的代码在 Claude Sonnet 5.5 上可能会在五个方面出错。要关闭预先思考，请在 `high` 或更低的 effort 下发送 `thinking: {"type": "between_tools"}`，而不是 `"disabled"`。强制 “tool use”（工具使用）（`tool_choice` 类型 `any` 和 `tool`）会返回 400 错误。思考块与模型和对话绑定。在 Claude API 和 Google Cloud 上，不再接受早期的 `computer_20251124` 计算机使用工具。Advisor 工具拒绝将 Claude Opus 4.8、Claude Opus 4.7 和 Claude Sonnet 5 作为顾问。有关每项变更，请参阅 [Claude Sonnet 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5)；有关变更前后的请求对比，请参阅 [Claude Sonnet 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide)。有关特定于模型的提示模式，请参阅[为 Claude Sonnet 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5)。
* Claude Sonnet 5.5 生成的思考块仅在生成它们的账户或与其关联的账户中有效。当其他账户发送这些块之一时，API 会在模型看到之前丢弃该块，并且请求会成功。来自较早模型的块不受影响。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。

### 2026 年 9 月 24 日

* 对于 `stop_details.category` 为 `"bio"`、`"frontier_llm"` 或 `"reasoning_extraction"` 且在任何输出之前发生的"refusals"（拒绝），我们将恢复计费，这些类别是我们测得误报量较低的类别。流式传输中途的拒绝此前已经计费。根据此变更计费的拒绝与任何其他请求一样收费，按运行该请求的模型的费率计算。其他类别中在任何输出之前发生的拒绝仍不计费，回退额度保持不变。此变更适用于所有平台。请参阅[拒绝如何计费](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。
* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 本地会话端点已针对 Excel、PowerPoint、Word 和 Outlook 中的 Claude for Microsoft 365 会话（`product_surface` 值以 `office_agents` 开头）结束 beta 阶段。请参阅[用户机器上的会话](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-local-sessions)。
* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)不再返回文件名、项目文档名称或 artifact 标题。文件、项目文档和 artifact 活动上的 `filename` 和 `title` 字段现在始终为空或被省略，包括在此变更之前记录的活动。要通过活动上的 ID 查找名称或标题，请使用具有 `read:compliance_user_data` 范围的 Compliance Access Key。请参阅[了解 Activity 对象](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed#understand-the-activity-object)。

### 2026 年 9 月 23 日

* [“Cache diagnostics”（缓存诊断）](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)已在 Claude API 上结束 beta 阶段，不再需要 `cache-diagnosis-2026-04-07` beta 标头。在 Messages 请求中包含 `diagnostics` 对象即可选择启用；仍发送该标头的请求照常工作。来自 `POST /v1/messages` 的响应现在始终包含 `diagnostics` 字段，当请求未包含 `diagnostics` 对象时，该字段为 `null`。

### 2026 年 9 月 22 日

* 我们已推出 **Claude Opus 5.5**（`claude-opus-5-5`），这是一款面向长时间运行的智能体编码和知识工作的模型。它默认具有 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)、128k 最大输出令牌，以及始终开启的[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)，价格为每 MTok $4 / $20 美元（Claude Opus 5 为 $5 / $25 美元）。Claude Opus 5.5 可在 Claude API、[Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。有关功能、API 变更和迁移指导，请参阅 [Claude Opus 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5)。
* 在 Claude Opus 5.5 上，思考无法禁用：`thinking: {"type": "disabled"}` 和 `thinking: {"type": "enabled", ...}` 会返回 400 错误。请省略 `thinking` 字段，并通过 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)控制思考深度。与 Claude Fable 5.1 一样，`tool_choice` 类型 `any` 和 `tool` 也会返回 400 错误；请将 `auto` 与[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)结合使用。在 Claude API 和 Google Cloud 上，此模型的[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)需要 `computer_toolset_20260801` 工具集，早期的 `computer_20251124` 工具会返回 400 错误；在 Amazon Bedrock 上，`computer_20251124` 仍可继续使用。请参阅 [Claude Opus 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-5)。
* [“Fast mode”（快速模式）](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)（研究预览版）已在 Claude API 上可用于 Claude Opus 5.5。
* 现在可以在[对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#define-tools-in-a-message-beta)中定义工具，该功能在 Claude API 上处于 beta 阶段，需使用 `inline-tools-2026-09-15` beta 标头。`tool_addition` 块可以携带工具的完整定义（`tool: {"type": "tool_definition", "definition": {...}}`），因此您可以添加工具、更改其 schema，或将服务器工具迁移到较新版本，而无需编辑 `tools`，也不会使"prompt cache"（提示缓存）失效。同一标头也涵盖按引用添加和移除工具。如果同时使用 MCP 连接器的 `mcp-client-2026-09-15` beta 标头，定义可以是 MCP 工具集，并且响应会在 `mcp_tool_listing` 块中记录每个服务器获取到的工具列表，当您将该块发回时，它会固定该列表。

### 2026 年 9 月 18 日

* 对于[缓存诊断](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)，发送 `cache-diagnosis-2026-04-07` beta 标头的请求所对应的响应现在始终包含 `diagnostics` 字段。当请求未包含 `diagnostics` 对象时，该字段为 `null`。此前，在这种情况下该字段会被省略。
* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 本地会话端点现在还会返回 Claude in Chrome 会话（`product_surface` 值为 `claude_in_chrome`）的对话记录，该功能面向 Claude Enterprise 组织处于 beta 阶段，使用您现有的 Compliance Access Key 和 `read:compliance_user_data` 范围。请参阅[用户机器上的会话](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-local-sessions)。

### 2026 年 9 月 14 日

* Messages API 现在可以在 Claude API 上[按需压缩对话](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)，该功能处于 beta 阶段，需使用 `compact-2026-09-04` beta 标头。发送顶层 `compaction` 参数，API 会返回一个已签名的 `compaction` 块，用于总结您发送的消息。在后续请求中，请先发送该块，以代替这些消息。您可以选择何时压缩，请求可以在后台运行，并且您可以在摘要之后逐字保留最近的轮次。在支持保留思考的模型上，这些保留轮次中的思考可以保持有效。
* 使用 `thinking-binding-controls-2026-08-01` beta 标头时，`input_transformations` 响应字段新增第二种条目类型 `thinking_mismatch_allowed`。它指明在 API 不强制执行前缀检查的请求中未通过该检查的思考块：例如，在 Claude Fable 5.1 上，来自 2026 年 8 月 31 日之前创建的账户且未设置 `prefix_mismatch_behavior` 的请求。该块仍会原样到达模型。在选择启用强制执行之前，请记录这些条目，以便在生产流量中发现对历史记录的编辑。请参阅[设置不匹配行为并读取 `input_transformations`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#preserved-thinking-controls)。

### 2026 年 9 月 10 日

* Claude Managed Agents 权限策略现在包含 `auto`：服务器会评估每个智能体或 MCP 工具调用，并运行它、拒绝它或暂停以等待您的批准。`agent.tool_use` 和 `agent.mcp_tool_use` 事件会在 `evaluated_permission` 旁边的 `evaluation` 字段中报告每个调用的评估方式。请参阅[使用 `auto` 让服务器评估每个调用](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#let-the-server-evaluate-each-call-with-auto)。
* `ant` CLI 1.32.0 版本新增了 `ant beta:sessions connect`，可将您的终端连接到 Claude Managed Agents 会话。您可以实时跟踪会话、发送消息，以及允许或拒绝正在等待批准的工具调用。传入 `--web` 可在本地提供 Claude Console 的会话查看器，并改为在其中打开会话。请参阅[从终端连接到 Managed Agents 会话](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/sessions-connect)。

### 2026 年 9 月 9 日

* 对于[缓存诊断](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)，API 现在仅在请求包含 `diagnostics` 对象时才存储该请求的指纹。仅发送 `cache-diagnosis-2026-04-07` beta 标头的请求仍会被接受，但不会存储指纹。之后将 `previous_message_id` 指向该请求的轮次会报告 `previous_message_not_found`。请在每个轮次中都包含 `diagnostics`，并在第一个轮次中使用 `"previous_message_id": null`。

### 2026 年 9 月 3 日

* `ant` CLI 1.30.0 版本新增了 `ant apply`，可根据您仓库中的文件创建和更新智能体、环境、技能、记忆存储和部署。在文件中描述每个资源，运行 `ant apply`，然后批准它打印的计划。提交它写入的 `claude-lock.json` 锁文件，以便之后在您的机器上或 CI 中运行时更新相同的资源，而不是创建新资源。请参阅[使用 ant apply 以代码形式管理资源](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/apply)。
* [按消息更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#change-effort-mid-conversation-beta) 的功能（beta）现在也可在 [Google Cloud](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 上用于 Claude Fable 5.1、Claude Mythos 5.1 和 Claude Opus 5，使用相同的 `mid-conversation-output-config-2026-07-01` beta 标头。

### 2026 年 9 月 1 日

* 我们推出了 **Claude Fable 5.1**（`claude-fable-5-1`），它是 Claude Fable 5 的后继模型，适用于长时间运行的智能体编码、知识工作和研究。我们还面向 Project Glasswing 参与者推出了 **Claude Mythos 5.1**（`claude-mythos-5-1`）。这两个模型默认支持 [1M 令牌的 context window（上下文窗口）](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)、128k 最大输出令牌以及始终开启的 [adaptive thinking（自适应思考）](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。它们的价格为每 MTok $10 / $50 美元，与 Claude Fable 5 相同，缓存读取价格则降至每 MTok $0.25 美元。Claude Fable 5.1 可在 Claude API、[Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 和 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上使用。它还可在 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。有关规格，请参阅 [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview)。
* Claude Fable 5.1 和 Claude Mythos 5.1 上的提示缓存读取费用为每百万令牌 $0.25 美元：为基础输入价格的 0.025 倍，而其他模型为 0.1 倍。缓存写入价格保持不变。请参阅[提示缓存定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#prompt-caching)。
* 在 Claude Fable 5.1 和 Claude Mythos 5.1 上，不支持 `tool_choice` 类型 `any` 和 `tool`，使用时会返回 400 错误。`auto` 和 `none` 保持不变。要保证工具输入符合 schema，请使用[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)或["structured outputs"（结构化输出）](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。
* Claude Fable 5.1 和 Claude Mythos 5.1 生成的思考块仅对生成它们的模型或更新的模型保留：较早的模型无法读取它们，并且 API 会丢弃重放给较早模型的此类块。Claude Fable 5.1 接受来自 Claude Opus 5、Claude Fable 5、Claude Mythos 5 及更早 Claude 模型的思考块。在 Claude Fable 5.1 上，API 还会[检查块之前的内容是否未发生变化](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-in-conversation)：对于在 2026 年 8 月 31 日或之后创建的新账户，在 `system` 提示、`tools` 或较早的消息发生变化后重放此类块会返回 400 错误。使用 `thinking-binding-controls-2026-08-01` beta 标头时，被丢弃的块会在 `input_transformations` 响应字段中报告，并且可以通过 `thinking.block_binding.prefix_mismatch_behavior` 选择拒绝还是丢弃历史记录已更改的块。请参阅[保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-thinking)。
* 按消息更改 effort 的功能在 Claude API 上针对 Claude Fable 5.1、Claude Mythos 5.1 和 Claude Opus 5 处于 beta 阶段。在 `messages` 中添加一条带有 `output_config.effort` 的 `role: "system"` 消息，即可更改后续轮次的 effort，同时保留提示缓存。请在请求中包含 `mid-conversation-output-config-2026-07-01` beta 标头。请参阅[按消息更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#change-effort-mid-conversation-beta)。
* [轮次范围的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#turn-scoped-system-messages)处于 beta 阶段（`mid-conversation-system-clear-at-2026-08-21` 标头）。在对话中途的 `role: "system"` 消息上设置 `clear_at: "next_user_message"`，它将仅在当前轮次中呈现，之后保留在历史记录中且不消耗令牌。每轮提醒不会累积，也不会使提示缓存或后续思考块失效。
* `thinking.display` 接受第三个值 `"updates"`，该功能处于 beta 阶段（`thinking-display-updates-2026-08-18` 标头）。推理内容返回时 `thinking` 字段为空，与 `"omitted"` 相同，而 Claude Fable 5.1、Claude Mythos 5.1 和 Claude Fable 5 在工具调用之间编写的简短进度更新会以文本形式返回，每次工具调用之前最多一个 `thinking` 块。请参阅[工具调用之间的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)。
* Claude Fable 5.1 和 Claude Mythos 5.1 生成的文本带有 Anthropic 的文本水印；当您在 Claude API 上通过 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 检索 Claude 通过[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)生成的受支持图像、视频和音频文件时，这些文件带有 C2PA 内容凭证（Content Credentials）。标记无需对您的请求或响应处理进行任何更改。
* 与 Claude Fable 5 一样，这两款模型都需要 30 天数据保留，并且除非获得 Anthropic 明确授权，否则无法在"zero data retention"（零数据保留）下使用。请参阅[特定于模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。
* [Admin API 的 Claude Enterprise 端点](https://platform.claude.com/docs/zh-CN/api/beta/organization)（[用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)和[支出限制](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)）、[Claude Enterprise Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api) 以及 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的指南现在会显示 `anthropic-version` 标头；与 Claude API 的其余部分一样，请在向这些端点发出的每个请求中发送该标头。请参阅 [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)。

### 2026 年 8 月 27 日

* 在 Python SDK 1.2.0、TypeScript SDK 0.122.0、Go SDK 1.68.0、Java SDK 2.59.0、Ruby SDK 1.67.0 和 C# SDK 12.44.0 中，`client.beta.files` 和 `client.beta.skills` 不再发送 `files-api-2025-04-14` 和 `skills-2025-10-02` beta 标头，并返回与 `client.files` 和 `client.skills` 相同的结构。随着此变更，`client.beta.skills.delete()` 会删除 Skill 及其所有版本，并且 beta Messages 类型 `BetaSkill`（容器 Skill 引用）已重命名为 `BetaContainerSkill`。仍发送 beta 标头的请求将继续收到 beta 结构。请参阅[从 `files-api-2025-04-14` 迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/files#migrate-from-files-api-2025-04-14)和[从 `skills-2025-10-02` 迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide#migrate-from-skills-2025-10-02)。
* 您现在可以在 Claude Console 中创建**个人密钥**和**服务账户密钥**。它们以您本人或[服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#service-accounts)的身份运行，具有相同的权限，并在关联账户从组织中移除时停止工作。这让组织管理员能够更轻松地跟踪每个账户的使用情况，并确保密钥的使用是合法的。这些 API 密钥可以限定到特定工作区，也可以[在管理端点以及该账户有权访问的任何工作区中使用](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)。工作区 API 密钥作为旧版选项继续受支持。有关更多信息，请参阅[身份验证中的 API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#api-keys)。

### 2026 年 8 月 26 日

* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的会话端点已针对 Cowork 和 Claude Code 会话结束 beta。请参阅[检索会话记录](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions)。
* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的本地会话端点现在还会返回 Claude Science 会话（`product_surface` 值为 `claude_science`）以及 Excel、PowerPoint、Word 和 Outlook 中的 Claude for Microsoft 365 会话（以 `office_agents` 开头的 `product_surface` 值）的记录，该功能面向 Claude Enterprise 组织提供 beta 版，使用您现有的 Compliance Access Key 和 `read:compliance_user_data` 范围即可。请参阅[用户机器上的会话](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-local-sessions)。
* [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 现在可在 `ant` CLI 以及 Python、TypeScript、C#、Go、Java、PHP 和 Ruby SDK 中通过 `client.beta.organization` 使用。它们涵盖组织信息、成员、邀请、工作区和工作区成员、API 密钥、速率限制、服务账户、工作负载身份联合颁发者和规则，以及客户管理的加密密钥。使用量和成本报告以及 Claude Enterprise 用户管理和分析端点仍仅支持 curl。CLI 和 SDK 从 `ANTHROPIC_API_KEY` 读取 Admin API 密钥，或从 `ANTHROPIC_AUTH_TOKEN` 读取 `org:admin` OAuth 令牌。

### 2026 年 8 月 20 日

* 我们发布了 **[Python SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/python) v1.0**。SDK 的 HTTP 层从 `httpx` 迁移到 [httpx2](https://httpx2.pydantic.dev)，这是一个持续维护且与 API 兼容的分支：请使用 `httpx2` 构建自定义 `http_client`、`Timeout` 和传输对象（`DefaultHttpxClient` 辅助工具保持不变）；如果您依赖会修补 `httpx` 的追踪或模拟库，请在启动时调用 `httpx2.alias_httpx()`。v1.0 需要 Python 3.10 或更高版本，并移除了长期弃用的接口，包括旧版 Text Completions API、Messages 方法上的 `temperature`、`top_p` 和 `top_k` 参数，以及工具运行器的客户端 `compaction_control`。在异步客户端上，`.with_raw_response` 结果现在需要 `await response.parse()`，并且当未配置 AWS 区域时，`AnthropicBedrock` 现在会引发错误，而不是默认使用 `us-east-1`。有关每项变更及其前后代码片段，请参阅 [v1 迁移指南](https://github.com/anthropics/anthropic-sdk-python/blob/main/MIGRATION.md)。
* [计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和["browser use"（浏览器使用）](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)工具集（`computer_toolset_20260801` 和 `browser_toolset_20260801`）现已在 [Google Cloud](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 上可用于 Claude Fable 5、Claude Mythos 5、Claude Opus 5、Claude Sonnet 5 和 Claude Opus 4.8。请求使用与 Claude API 上相同的 `tools` 条目。

### 2026 年 8 月 19 日

* [计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)已在 Claude API 上以 `computer_toolset_20260801` 工具集的形式结束 beta：无需 beta 标头，支持批量操作（一个轮次中执行多个操作），默认启用 `zoom`，并可通过 `configs` 进行按成员配置。较早的 beta 版本仍然可用。升级现有集成会改变请求结构和工具处理方式；请参阅[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)。
* 我们推出了[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)（`browser_toolset_20260801`），这是一个用于驱动由您的应用程序托管的浏览器的客户端工具集。它在浏览器视口而非整个桌面中工作，直接读取页面本身（其无障碍树、元素、表单和标签页），并在截图加点击控制的基础上增加了元素引用、表单输入、标签页管理、下载报告以及可选启用的文件上传。
* 这两个工具集均已在 Claude API 上面向 Claude Fable 5、Claude Mythos 5、Claude Opus 5、Claude Sonnet 5 和 Claude Opus 4.8 提供。
* [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 已在 Claude API 上结束 beta。对 `/v1/files` 端点的请求以及引用已上传文件的 Messages API 请求不再需要 `files-api-2025-04-14` beta 标头。不带该标头发送的请求使用当前的响应格式：[文件过期](https://platform.claude.com/docs/zh-CN/build-with-claude/files#file-expiration)（上传文件时设置 `expires_in_seconds`；文件对象会报告 `expires_at`），以及在[列出文件](https://platform.claude.com/docs/zh-CN/build-with-claude/files#list-files)时使用 `page` 和 `next_page` [分页](https://platform.claude.com/docs/zh-CN/api/overview#pagination)和 `ids[]` 过滤器。仍发送 beta 标头的 `/v1/files` 请求会继续工作，并返回之前的响应格式。 要让现有集成不再使用该标头，请参阅[从 `files-api-2025-04-14` 迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/files#migrate-from-files-api-2025-04-14)。
* [Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview) 和 Skills API（`/v1/skills`）已在 Claude API 上结束 beta。请求不再需要 `skills-2025-10-02` beta 标头，包括通过 `container` 参数加载 Skills 的 Messages API 请求。仍发送该标头的请求会继续照常工作。请参阅[通过 API 使用 Agent Skills](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide)。 要让现有集成不再使用该标头，请参阅[从 `skills-2025-10-02` 迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide#migrate-from-skills-2025-10-02)。
* 面向 **Claude Enterprise**（claude.ai）组织的 [Admin API 用户管理端点](https://platform.claude.com/docs/zh-CN/api/beta/organization)（成员、邀请、群组和自定义角色）已结束 beta。群组和自定义角色请求不再需要 `anthropic-beta: ce-user-management-2026-07-13` 标头；仍发送该标头的请求会照常被接受。请参阅[用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)。
* 您现在可以限制 Claude Managed Agents 智能体的 `web_search` 和 `web_fetch` 工具可以访问哪些站点。在 `agent_toolset_20260401` 的 `configs` 数组中该工具的条目上设置 `allowed_domains` 或 `blocked_domains`；`web_fetch` 还接受 `max_content_tokens`，`web_search` 接受 `user_location`。每个 `configs` 条目通过其 `name` 标识，并通过可选的 `type` 指定类型，仅传递 `name`、`enabled` 和 `permission_policy` 的请求会继续工作；在类型化 SDK 中，`configs` 条目会成为[按工具划分的类型](https://platform.claude.com/docs/zh-CN/managed-agents/tools#config-entry-types-in-the-sdks)。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。
* 在[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)中运行的 Claude Managed Agents 会话现在可以挂载[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)。Python、TypeScript 和 Go SDK worker 会将每个挂载的存储下载到沙箱中的 `mount_path`，并将智能体的更改同步回存储。请参阅[自托管沙箱中的记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)。
* Claude Console 中的会话查看器已重新设计，新增了时间线缩略图、按模型请求分组的对话记录，以及一个 Inspector 面板，用于查看会话详情和成本、原始事件、按工具统计、已挂载资源以及按线程的活动。请参阅[在 Console 中检查会话](https://platform.claude.com/docs/zh-CN/managed-agents/session-observability#inspect-a-session-in-the-console)。

### 2026 年 8 月 18 日

* Workbench 现在在 Claude Console 中更名为 [**playground**](https://platform.claude.com/playground)。Playground 支持所有 Messages API 参数，并包含演示代码执行和网络搜索等 API 功能的模板。它会显示每次运行的完整 SDK 请求和 API 响应，帮助您理解 API 并基于它进行构建。有关更多信息，请参阅 [Claude 帮助中心](https://support.claude.com/en/articles/8606378-how-do-i-use-playground)，或在 [platform.claude.com/playground](https://platform.claude.com/playground) 上试用。

### 2026 年 8 月 11 日

* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 现在会返回在您用户的机器上运行的 Cowork 和 Claude Code 会话的对话记录，该功能面向 Claude Enterprise 组织处于 beta 阶段。`GET /v1/compliance/apps/sessions/local` 列出整个组织中的会话，`GET /v1/compliance/apps/sessions/local/{session_id}` 检索单个会话的元数据，`GET /v1/compliance/apps/sessions/local/{session_id}/messages` 返回其对话记录，所有这些都使用您现有的 Compliance Access Key 和 `read:compliance_user_data` 范围。请参阅[用户机器上的会话](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-local-sessions)。
* 我们已向 Claude API 添加了 `anthropic-workspace-id` 响应标头。它包含请求的 API 密钥或访问令牌所解析到的工作区的 ID（带 `wrkspc_` 前缀），包括您组织的默认工作区（Default Workspace）。请参阅[识别 API 响应背后的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#identify-the-workspace-behind-an-api-response)。

### 2026 年 8 月 10 日

* **Claude Sonnet 5** 的推广价格（每 MTok $2 / $10 美元）现已成为标准价格：原定于 2026 年 9 月 1 日上调至每 MTok $3 / $15 美元的计划将不会实施。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。

### 2026 年 8 月 7 日

* 您现在可以为 Claude Managed Agents 会话设置预算：这是会话支出的硬性上限，按公开标价计算。达到预算的会话会以 `budget_reached` 停止原因暂停，而不是发起新的模型请求；更改或移除预算会使其恢复。部署接受相同的预算，并将其应用于它们启动的每个会话。请参阅[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)。
* 您现在可以为 Claude Managed Agents 会话配置顾问（advisor）：一个能力至少与智能体自身模型相当的模型，会话的主线程可以在轮次中途向其咨询以获取战略指导。在智能体的多智能体名单中将其配置为 `{"type": "advisor"}` 条目，并指定要咨询的 `model`。请参阅[为会话配置顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。
* 您现在可以控制 Claude Managed Agents 智能体的模型推理在何处运行。在创建智能体时（请参阅[固定推理地理位置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#pin-the-inference-geo)）在 `model` 对象中设置 `inference_geo`，或[为单个会话覆盖该设置](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#pin-the-inference-geo-for-a-session)。有关可用的地理区域和定价，请参阅[数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)。
* Claude Managed Agents 会话现在可以[从 GitHub 仓库加载技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills#load-skills-from-a-github-repository)。当会话[挂载仓库](https://platform.claude.com/docs/zh-CN/managed-agents/github)时，其根目录 `.claude/skills` 中的所有技能都会在会话开始时被自动发现，并在该会话中供智能体使用。

### 2026 年 8 月 5 日

* **Inference hooks**（推理钩子）现已面向 Claude Enterprise 组织推出 beta 版。将 Claude 指向您组织的 AI 安全服务器后，claude.ai、Cowork 和 Claude Code 中每个受管控的提示都会被暂缓，等待服务器给出允许或拒绝的裁决后再进行推理。请求经过签名，故障处理方式可配置，并且每次拒绝都会记录在合规[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)中。请参阅[推理钩子](https://platform.claude.com/docs/zh-CN/manage-claude/inference-hooks)。
* 我们已停用 Claude Opus 4.1 模型（`claude-opus-4-1-20250805`）。现在，Claude API 上对该模型的所有请求都将返回错误。我们建议升级到 [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview)。研究人员可以通过[外部研究人员访问计划](https://support.claude.com/en/articles/9125743-what-is-the-external-researcher-access-program)申请持续访问。

### 2026 年 8 月 3 日

* [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 现在会返回在 claude.ai 网页版或移动端上启动的 Cowork 会话的对话记录，该功能面向 Claude Enterprise 组织处于 beta 阶段。`GET /v1/compliance/apps/sessions/remote` 列出会话，`GET /v1/compliance/apps/sessions/remote/{session_id}/messages` 返回单个会话的对话记录，使用您现有的具有 `read:compliance_user_data` 范围的 Compliance Access Key。请参阅[云端会话](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-remote-sessions)。

### 2026 年 8 月 1 日

* [梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)（研究预览版）现已支持 Claude Opus 5。请参阅[支持的模型](https://platform.claude.com/docs/zh-CN/managed-agents/dreams#feature-availability)。

### 2026 年 7 月 24 日

* 我们推出了 **Claude Opus 5**（`claude-opus-5`），相比 Claude Opus 4.8 实现了跨越式提升。Claude Opus 5 支持 [1M 令牌的 context window（上下文窗口）](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)（既是默认值也是最大值）和 128k 最大输出令牌，并默认开启 [thinking（思考）](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。其价格为每 MTok $5 / $25 美元，与 Claude Opus 4.8 定价相同。它可在 Claude API、[Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。有关完整规格，请参阅 [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview) 和[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)。
* 在 Claude Opus 5 上，仅当 effort 为 `high` 或更低时才允许禁用思考：在 effort 为 `xhigh` 或 `max` 时使用 `thinking: {"type": "disabled"}` 会返回 400 错误。这是相对于 Claude Opus 4.8 的 breaking change（破坏性变更）。请参阅[结合思考使用 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-with-thinking)。
* [Effort（努力程度）](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)是引导 Claude Opus 5 的主要控制手段：该模型支持完整的级别阶梯（`low`、`medium`、`high`、`xhigh`、`max`），其中 `max` 用于对能力要求极高的工作。
* “Mid-conversation tool changes”（对话中途工具变更）现已在 Claude Fable 5、Claude Mythos 5、Claude Opus 4.8 和 Claude Opus 5 上进入 beta 阶段：您可以在对话轮次之间添加或移除工具，同时保留提示缓存。请在请求中包含 `mid-conversation-tool-changes-2026-07-01` beta header（beta 标头）。
* `fallbacks` 参数现在支持 `"default"` 模式，该模式会按拒绝类别应用 Anthropic 推荐的回退模型。"Server-side fallback"（服务器端回退）目前处于 beta 阶段，`"default"` 模式需要 `server-side-fallback-2026-07-01` beta 标头。请参阅[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)。
* 我们已移除 Claude Opus 4.7 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。现在对 `claude-opus-4-7` 使用 `speed: "fast"` 的请求会返回错误；与 Claude Opus 4.6 不同，这些请求不会回退到标准速度。Claude Opus 4.7 本身仍可以标准速度使用。要继续使用快速模式，请迁移到 [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview) 或 Claude Opus 4.8。有关更多信息，请参阅[快速模式的模型可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#supported-models)。

### 2026 年 7 月 22 日

* 您现在可以在 Claude Managed Agents 智能体的模型配置中设置 `effort` 级别。在[创建智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#create-an-agent)时在 `model` 对象中传入 `effort`。有关各级别的作用，请参阅 [Effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-levels)。
* Claude Managed Agents 的 Webhooks 现在涵盖环境和记忆存储的生命周期：四种 `environment.*` 事件类型和三种 `memory_store.*` 事件类型。您无需轮询即可响应环境和记忆存储的生命周期变化。请参阅[支持的事件类型](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#supported-event-types)中的环境事件和记忆存储事件标签页。
* 创建 Claude Managed Agents 会话时，您现在可以[使用初始事件为其预置内容](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#seed-the-session-with-initial-events)。在 `POST /v1/sessions` 上传入 `initial_events`，最多包含 50 个 `user.message` 和 `user.define_outcome` 事件。非空列表会在同一调用中启动智能体循环，因此您无需单独发送事件请求来开始工作。
* [更新 Claude Managed Agents 智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-an-agent)时，`version` 字段现在是可选的。提供该字段可实现乐观并发控制（不匹配时返回 409 错误），省略则无条件应用更新。请参阅[更新语义](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-semantics)。
* Claude Managed Agents 会话线程事件流现在支持[事件增量](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas)。`GET /v1/sessions/{session_id}/threads/{thread_id}/stream` 接受与会话级流相同的 `event_deltas[]` 查询参数，因此您可以在模型生成子智能体文本时预览该文本。一个连接只预览它正在读取的线程。请参阅[预览会话线程事件](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#preview-session-thread-events)。

### 2026 年 7 月 17 日

* Claude Console 中的旧版 **Workbench**（[platform.claude.com/workbench](https://platform.claude.com/workbench)）即将停用，访问将于 2026 年 8 月 17 日结束。Console 中更新后的 [Workbench](https://platform.claude.com/playground) 不支持已保存的提示、变量和评估。您可以通过横幅或在 **Organizational Settings**（组织设置）下导出任何想要保留的数据。有关更多信息，请参阅 Claude 帮助中心的[如何使用 Workbench？](https://support.claude.com/en/articles/8606378-how-do-i-use-the-workbench)。
* 用于生成、改进和模板化提示的实验性提示工具 API（`/v1/experimental/generate_prompt`、`/v1/experimental/improve_prompt` 和 `/v1/experimental/templatize_prompt`）将于 2026 年 8 月 17 日随 Workbench 一同停用。移除后，对这些端点的请求将返回错误。

### 2026 年 7 月 15 日

* [对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)可在 Claude API、[Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 和 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 上的 Claude Fable 5、Claude Mythos 5 和 Claude Opus 4.8 中使用，无需 beta 标头。此条更正了之前的可用性说明。

### 2026 年 7 月 14 日

* 您现在可以使用 Admin API（请参阅 [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization)）管理 **Claude Enterprise**（claude.ai）组织中的人员。该功能以 beta 形式面向所有 Claude Enterprise 组织提供，支持以下操作：列出成员并按电子邮件地址查找成员、更改成员角色、移除成员、发送和撤回邀请、管理群组及其成员资格，以及读取自定义角色。群组和自定义角色请求需要 `anthropic-beta: ce-user-management-2026-07-13` beta 标头；成员和邀请请求则不需要 beta 标头。具有 `read:org_audit` 范围的 Admin API 密钥也可以调用所有用户管理 `GET` 端点。请参阅[用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)。

### 2026 年 7 月 10 日

* [梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)（研究预览版）现在支持 Claude Fable 5 和 Claude Sonnet 5。请参阅[支持的模型](https://platform.claude.com/docs/zh-CN/managed-agents/dreams#feature-availability)。
* 我们扩充了[访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency)文档中关于 `cmek_preserve` 事件的内容，新增了一个筛选示例、一个示例事件负载以及两个保留原因代码（`policy_violation_investigation`、`csae_report`）。该文档现在还阐明：无论保留操作是由人工审核员还是自动化安全流水线发起的，都会写入保留事件。请参阅 [CMEK 内容保留](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#cmek-content-preservation)。

### 2026 年 7 月 8 日

* 现在，您在 [Claude Console](https://platform.claude.com/settings/keys) 中创建 API 密钥或 Admin API 密钥时可以设置过期时间。您可以选择预设值、自定义时长或**从不**。对于有效期至少为 7 天的密钥，Anthropic 会在过期前向创建者发送电子邮件。现有密钥不受影响。Admin API 会在 [`expires_at`](https://platform.claude.com/docs/zh-CN/api/beta/organization/api_keys/list) 字段中报告每个密钥的过期时间。请参阅[身份验证中的密钥过期](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)。

### 2026 年 7 月 2 日

* 我们新增了 `agent-memory-2026-07-22` beta 标头，它会改变[列出记忆](https://platform.claude.com/docs/zh-CN/managed-agents/memory#list-memories)（`GET /v1/memory_stores/{memory_store_id}/memories`）的行为：结果以稳定的、由服务器定义的顺序返回，并忽略 `order_by` 和 `order` 参数；`depth` 仅接受 `0`、`1` 或省略（其他值返回 `400` 错误）；`path_prefix` 必须以 `/` 结尾，并匹配完整的路径段而非子字符串。未使用该标头时签发的分页游标在使用该标头后无效，因此在采用该标头时请从第一页重新开始。在记忆存储端点上，`agent-memory-2026-07-22` 取代了 `managed-agents-2026-04-01`；同时发送两者会返回 `400` 错误。2026 年 7 月 22 日起，`managed-agents-2026-04-01` 标头也将采用相同的列表行为。请参阅 [Beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers#endpoint-specific-headers)。
* Python (0.116.0)、TypeScript (0.110.0)、Go (1.56.0)、Java (2.48.0)、Ruby (1.55.0)、PHP (0.36.0)、C# (12.35.0) 和 CLI (1.16.0) SDK 现在在所有记忆存储调用中发送 `agent-memory-2026-07-22`，而不是 `managed-agents-2026-04-01`。如果您的代码在记忆存储调用中显式传入 `betas`，请在该处将 `managed-agents-2026-04-01` 替换为 `agent-memory-2026-07-22`，而不是添加第二个值。

### 2026 年 7 月 1 日

* 我们已恢复对 Claude Fable 5 和 Claude Mythos 5 的访问。有关更多信息，请参阅[我们的声明](https://www.anthropic.com/news/redeploying-fable-5)。

### 2026 年 6 月 30 日

* 我们推出了 **Claude Sonnet 5**（`claude-sonnet-5`），这是我们 Sonnet 模型系列的新一代产品，推广价格为每 MTok $2 / $10 美元（已于 2026 年 8 月 10 日成为标准价格）。Claude Sonnet 5 支持 [1M 令牌的上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)、128k 最大输出令牌，以及与 Claude Sonnet 4.6 相同的工具和平台功能集，但 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers#supported-models) 除外，该功能在 Claude Sonnet 5 上不可用。迁移时有三项行为变更：[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)现已默认开启；手动 extended thinking（扩展思考）（`thinking: {type: "enabled", budget_tokens: N}`）已被移除，使用时会返回 400 错误（它已在 Sonnet 4.6 上被弃用）；将采样参数（`temperature`、`top_p`、`top_k`）设置为非默认值会返回 400 错误。Claude Sonnet 5 还使用了一个新的分词器，对于相同文本会产生大约多 30% 的令牌。具体增幅取决于内容和工作负载的形态。有关详细信息，请参阅 [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview)。有关行为差异和特定于模型的提示模式，请参阅[为 Claude Sonnet 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5)。
* Claude Managed Agents 会话事件流现在支持[事件增量](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas)。通过在 `GET /v1/sessions/{session_id}/events/stream` 上使用 `event_deltas[]` 查询参数选择启用。`event_start` 和 `event_delta` 事件会在完整的 `agent.message` 事件到达之前，在智能体消息文本生成时对其进行预览。
* Claude Managed Agents 的[列出会话](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#listing-sessions)现在支持向后分页。`GET /v1/sessions` 会在 `next_page` 之外返回 `prev_page` 游标；将其作为 `page` 参数传入即可返回上一页。请参阅[分页](https://platform.claude.com/docs/zh-CN/api/overview#pagination)。
* 创建 Claude Managed Agents 会话时，您现在可以[为该会话覆盖智能体的配置](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)。传入带有 `type: "agent_with_overrides"` 的 `agent`，即可为单个会话替换模型、system prompt（系统提示）、工具、MCP 服务器或技能。智能体本身保持不变。
* Claude Managed Agents 保管库（vaults）现在在[环境变量凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults#add-a-credential)（环境变量选项卡）上支持 `injection_location` 设置。它控制凭据的值在出口处被替换到智能体出站请求的请求头、请求体还是两者中。
* Claude Managed Agents 的 Webhooks 现在涵盖智能体、部署和部署运行的生命周期。您无需轮询即可响应新发布的智能体版本、已暂停的部署或失败的计划运行。请参阅[支持的事件类型](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#supported-event-types)中的智能体事件、部署事件和部署运行事件标签页。

### 2026 年 6 月 29 日

* 我们已移除 Claude Opus 4.6 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。现在，使用 `speed: "fast"` 向 `claude-opus-4-6` 发出的请求不再以快速模式运行，也不再按溢价计费：这些请求会以标准速度运行、按标准费率计费，且不会返回错误。响应中的 `usage.speed` 字段会报告实际使用的速度。如需继续使用快速模式，请迁移到 [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)。在[快速模式的模型可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#supported-models)中了解更多信息。

### 2026 年 6 月 26 日

* 我们提高了整个 Claude API 的 [rate limits（速率限制）](https://platform.claude.com/docs/zh-CN/api/rate-limits)。现在，Claude Sonnet 和 Claude Haiku 在每个使用层级上的速率限制都与 Claude Opus 一致，并且使用层级已合并为三个：Start、Build 和 Scale。大多数组织会升至更高层级，所有组织的限制都不会低于以前，且您无需采取任何操作。您可以在 Claude Console 中[查看您的层级和当前限制](https://platform.claude.com/settings/limits)。

### 2026 年 6 月 25 日

* 我们已弃用 Claude Opus 4.7 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)，该功能将于 2026 年 7 月 24 日移除。移除后，使用 `speed: "fast"` 向 `claude-opus-4-7` 发出的请求将返回错误。请迁移到 Claude Opus 4.8 的快速模式。在[快速模式的模型可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#supported-models)中了解更多信息。

### 2026 年 6 月 22 日

* **MCP 隧道**（研究预览版）：管理 API 已从 Admin API 上的 `/v1/organizations/tunnels` 迁移到 Claude API 上的 `/v1/tunnels`。新接口使用 `anthropic-beta: mcp-tunnels-2026-06-22` 标头和 `workspace:manage_tunnels` WIF 范围。在迁移窗口期内，旧接口仍然可用。请参阅 [Tunnels API 参考](https://platform.claude.com/docs/zh-CN/api/beta/tunnels)。

### 2026 年 6 月 18 日

* Python、TypeScript、Go、Java、Ruby、PHP 和 C# SDK 现在支持 `code_execution_20260120`。这是[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)的一个版本，它增加了 REPL 状态持久化，也是使用[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)所需的最低版本。要采用该版本，请将工具的 `type` 设置为 `code_execution_20260120`，无需 beta 标头。该版本可在 Claude Fable 5、Claude Mythos 5、Claude Opus 4.5 及更新版本，以及 Claude Sonnet 4.5 及更新版本上使用；请参阅代码执行工具的[API 支持](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#availability-feature-support)部分。

### 2026 年 6 月 15 日

* 我们已停用 Claude Sonnet 4 模型（`claude-sonnet-4-20250514`）和 Claude Opus 4 模型（`claude-opus-4-20250514`）。现在，在 Claude API 上对这些模型的所有请求都将返回错误。我们建议分别升级到 [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison) 和 [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)。研究人员可以通过[外部研究人员访问计划](https://support.claude.com/en/articles/9125743-what-is-the-external-researcher-access-program)申请继续访问。

### 2026 年 6 月 11 日

* [代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)现在支持 `code_execution_20260521`。该版本会在工具描述中说明每个单元格 90 秒的执行时间限制，以便 Claude 为长时间运行的单元格合理规划时间。无需 beta 标头。
* [网络搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)和[网页抓取工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)现在支持 `web_search_20260318` 和 `web_fetch_20260318`。这两个版本新增了 `response_inclusion` 参数，可在智能体工作流中从 API 响应里删除已使用的结果块。无需 beta 标头。

### 2026 年 6 月 10 日

* `GET /v1/environments/{id}/work` 端点现已在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上可用，该端点用于列出[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)的待处理工作。有关授权该端点的 `GetEnvironment` 操作，请参阅 [Claude Platform on AWS 的 IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions)。

### 2026 年 6 月 9 日

* 我们推出了 **Claude Fable 5**（`claude-fable-5`），这是我们向所有客户开放的能力最强的模型，同时为 Project Glasswing 参与者推出了 **Claude Mythos 5**（`claude-mythos-5`）。两个模型均默认支持 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)、128k 最大输出令牌，以及始终开启的[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。有关规格、定价和可用性，请参阅 [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview)。
* Claude Fable 5 和 Claude Mythos 5 使用随 Claude Opus 4.7 引入的分词器。与 Claude Opus 4.7 之前的模型相比，相同文本产生的令牌约多 30%，确切增幅取决于内容和工作负载形态。您可以使用带有 `model: "claude-fable-5"` 的令牌计数 API 来衡量您的提示在新分词器下的令牌数。请参阅 [Claude Fable 和 Claude Mythos 模型上的令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting#token-counts-on-claude-fable-5)。
* Claude Fable 5 会对请求运行安全分类器，并在响应生成期间持续运行。当分类器拒绝请求时，Messages API 会返回 `stop_reason: "refusal"`。如果请求在生成任何输出之前就被拒绝，您不会为其付费。您可以选择启用 `fallbacks` 参数，在另一个模型上重新运行被拒绝的请求，并按回退模型的费率计费。该参数在 Claude API 和 Claude Platform on AWS 上处于 beta 阶段，Message Batches API 不支持该参数。请参阅[处理停止原因](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)。
* 在 Claude Fable 5 上，拒绝响应中的 [`stop_details.category`](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response) 字段现在包含 `"reasoning_extraction"`。当请求因违反 Anthropic 服务条款中关于逆向工程或复制模型输出的限制而被阻止时，会返回该值。现有的 `"cyber"` 和 `"bio"` 类别保持不变。无需 beta 标头。
* 在 Claude Fable 5 和 Claude Mythos 5 上，[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)是唯一的思考模式：不支持 `thinking: {"type": "disabled"}`，也不支持手动扩展思考预算和助手预填充（两者都会返回 400 错误）。
* 在 Claude Fable 5 和 Claude Mythos 5 上，`thinking.display` 默认为 `"omitted"`，与 Claude Opus 4.8、Claude Opus 4.7 和 Claude Mythos Preview 相同。设置 `display: "summarized"` 即可接收可读的思考摘要。原始思维链永远不会返回；在同一模型上进行多轮对话时，请原样传回思考块。请参阅 [Claude Fable 5 和 Claude Mythos 5 上的思考输出](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-output-on-claude-fable-5-and-claude-mythos-5)。
* Claude Fable 5 要求 30 天数据保留，不支持零数据保留。请参阅[特定于模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。
* Claude Managed Agents 现在支持[计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)，让您无需自行管理调度器即可按 cron 计划运行会话。
* Claude Managed Agents 保管库现在支持[环境变量凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults#add-a-credential)。对于通过环境变量进行身份验证的 CLI、SDK 和其他服务，您可以借此将密钥安全地注入智能体的沙箱。
* [合规 API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)（`GET /v1/compliance/activities`）现已在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上可用。有关授权它的 `ListComplianceActivities` 操作，请参阅 [Claude Platform on AWS 的合规 IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#compliance)。
* `session.thread_*` webhook 事件现在包含 `session_thread_id` 字段，用于标识触发该事件的多智能体线程。
* 我们发布了一个 beta 版 [Swift 包](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/apple-foundation-models)，它将 Claude 作为服务器端 `LanguageModel` 添加到 Apple 的 Foundation Models 框架中。在 iOS 27、macOS 27、visionOS 27 和 watchOS 27（beta）上，您可以通过与 Apple 设备端模型相同的 `LanguageModelSession` API 调用 Claude。

### 2026 年 6 月 5 日

* 我们宣布弃用 Claude Opus 4.1 模型（`claude-opus-4-1-20250805`），该模型计划于 2026 年 8 月 5 日在 Claude API 上停用。我们建议迁移到 [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)。在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。

### 2026 年 6 月 2 日

* [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)现已支持 `max_tokens` 参数，用于限制顾问模型每次调用的输出，从而为不需要完整长度顾问响应的工作负载降低延迟和输出令牌成本。请在 Advisor 工具定义上设置 `tools[].max_tokens`；请参阅[限制顾问输出](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#capping-advisor-output)。
* 在 Claude API 上，如果请求在 Claude 未生成任何输出的情况下返回 `stop_reason: "refusal"`，您将不再为该请求付费。有关如何检测和处理拒绝，请参阅[流式传输拒绝](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/handle-streaming-refusals)。

### 2026 年 5 月 29 日

* Claude Managed Agents 的 [webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks)、[多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)和[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)现已在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上可用。有关新的 IAM 操作和 `AnthropicSelfHostedEnvironmentAccess` 托管策略，请参阅 [Claude Platform on AWS 的 IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions)。

### 2026 年 5 月 28 日

* 我们推出了 **Claude Opus 4.8**（claude-opus-4-8），这是我们能力最强的模型。在 Claude API、Amazon Bedrock、Google Cloud 和 Microsoft Foundry 上，Claude Opus 4.8 默认支持 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)和 128k 最大输出令牌，并提供与 Claude Opus 4.7 相同的工具和平台功能集。有关基线设置、功能和迁移指南，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。
* 我们推出了[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)。在 Claude Opus 4.8 上，您可以在 `messages` 数组中的用户轮次之后发送 `role: "system"` 消息（须遵守[放置规则](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#limitations)）。这样，在长时间运行的会话中更改指令时，仍可保留提示缓存命中。无需 beta 标头。
* 拒绝响应中的 [`stop_details`](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response) 字段现已公开记录。该字段返回一个 `category`（`cyber`、`bio` 或 `null`）和一个人类可读的 `explanation`，以便您的应用程序将不同类别的拒绝路由到相应的后续步骤。无需 beta 标头。
* 在 Claude Opus 4.8 上，[effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)在所有平台（包括 Claude Code 和 Messages API）上均默认为 `high`。
* 在 Claude Opus 4.8 上，[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)的最小可缓存提示长度为 1,024 个令牌，低于 Claude Opus 4.7。
* 启用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)后，Claude Opus 4.8 仅在某个轮次需要时才会触发推理。与相同 effort 级别下的 Claude Opus 4.7 相比，这减少了浪费的思考令牌。
* Claude Opus 4.8 支持[高分辨率图像输入](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#high-resolution-image-support-on-claude-opus-4-7)（长边最多 2576 像素），与 Claude Opus 4.7 相同。
* [任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)现在支持 Claude Opus 4.8。
* [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)现在支持 Claude Opus 4.8。
* [计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)现在支持 Claude Opus 4.8。
* Claude Opus 4.8 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)以研究预览版形式提供，仅限 Claude API。
* 在 Claude Opus 4.8 上，将采样参数 `temperature`、`top_p` 或 `top_k` 设置为非默认值会返回 400 错误，与 Claude Opus 4.7 相同。有关详细信息，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。
* 在 Claude Code 中，我们已将 Auto 模式扩展到更多用户，用于长时间运行的任务。请参阅 [Claude Code 文档](https://code.claude.com/docs)。
* 在 Claude Code 中，Max 计划用户现在在 Claude Opus 4.8 上默认使用[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。请参阅 [Claude Code 文档](https://code.claude.com/docs)。
* 在 Claude Code 中，Workflows 以研究预览版形式提供，让您可以定义和运行多步骤的智能体计划。请参阅 [Claude Code 文档](https://code.claude.com/docs)。
* 我们已弃用 Claude Opus 4.6 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)，将在发布后大约 30 天移除。请迁移到 Claude Opus 4.8 或 Claude Opus 4.7 的快速模式。请在[快速模式模型可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#supported-models)中了解更多信息。
* 有关本次发布中 claude.ai、Cowork、Claude for Microsoft 365 和其他 Claude 应用的更新，请参阅 [Claude 应用的发布说明](https://support.claude.com/en/articles/12138966-release-notes)。

### 2026 年 5 月 27 日

* Messages API 响应现在包含 [`usage.output_tokens_details.thinking_tokens`](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#budget-rules-and-tuning)，用于报告计费输出令牌中有多少属于扩展思考。使用 streaming（流式传输）时，该细分数据仅出现在最终的 `message_delta` 事件中。无需 beta 标头。

### 2026 年 5 月 19 日

* [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)现已以研究预览版形式提供，让您可以连接到私有网络中的 MCP 服务器。
* Claude Managed Agents 现已提供自托管沙箱，作为在 Anthropic 基础设施中执行工具的替代方案。请参阅[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)。
* 借助 Claude Managed Agents，您现在可以更新与活动会话关联的智能体 MCP 服务器和工具配置。
* 借助 Claude Managed Agents，`agent_toolset` 和 MCP 工具产生的超过 100k 个字符（约 25k 个令牌）的大型输出现在会自动溢出到沙箱中的文件。模型会收到包含文件路径的截断预览，并可以从该文件读取完整内容。

### 2026 年 5 月 18 日

* [网络搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)现在会返回更丰富的 SEC 文件数据。这让金融研究智能体、财报分析和尽职调查工作流能够更轻松地以带引用的一手来源为依据。

### 2026 年 5 月 13 日

* 我们推出了处于 beta 阶段的[缓存诊断](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)。在 Messages 请求中传递 `diagnostics.previous_message_id`，API 会报告一个 `cache_miss_reason`，说明提示缓存前缀在何处与上一轮次产生分歧。请在您的请求中包含 `cache-diagnosis-2026-04-07` beta 标头。

### 2026 年 5 月 12 日

* [快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)（研究预览版）现在支持 Claude Opus 4.7。设置 `speed: "fast"`，并配合使用 `model: "claude-opus-4-7"` 和 `fast-mode-2026-02-01` beta 标头，即可以溢价获得显著更快的输出令牌生成速度。其定价、速率限制和访问方式与 Opus 4.6 快速模式相同；感兴趣的客户请加入[候补名单](https://claude.com/fast-mode)。

### 2026 年 5 月 11 日

* 我们推出了 **Claude Platform on AWS**，将 Claude API 引入可通过 AWS 访问的 Anthropic 托管基础设施，并支持 AWS 计费和 IAM 身份验证。您可以通过原生 AWS 端点访问完整的 Messages API、Files API、Message Batches API、Claude Managed Agents、Agent Skills、代码执行和工具使用功能。在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 中了解更多信息。

### 2026 年 5 月 6 日

* [多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)和[结果](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)现已在标准 `managed-agents-2026-04-01` beta 标头下进入 beta 阶段。
* Claude Managed Agents 保管库的凭据后台刷新功能现在支持 `mcp_oauth` 凭据。请参阅[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)。
* Claude Managed Agents 现已支持 webhooks。Webhook 事件类型包括会话和保管库的生命周期事件。请参阅[订阅 webhooks](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks)。
* Claude Managed Agents 现在支持更多筛选和排序选项：会话可以按状态筛选，事件可以按类型筛选。事件现在还可以按创建时间筛选。
* Claude Managed Agents 的[梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)现已以研究预览版形式提供。Dream 会读取现有的记忆存储以及过去的会话记录，并生成一个重新组织后的输出记忆存储：合并重复项、替换过时条目，并提炼出新的见解。Dream 端点需要使用 `dreaming-2026-04-21` beta 标头才能访问。如需试用，请[申请访问权限](https://claude.com/form/claude-managed-agents)。

### 2026 年 5 月 4 日

* 我们推出了 [Workload Identity Federation](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)（工作负载身份联合）。您可以使用来自自有身份提供商（AWS IAM、Google Cloud、GitHub Actions、Kubernetes、Microsoft Entra ID、Okta、SPIFFE 等）的短期 OIDC 令牌，向 Claude API 验证工作负载身份，而无需使用长期有效的静态 API 密钥。在 Claude Console 中配置颁发者和联合规则后，SDK 会自动处理令牌交换和刷新。请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。

### 2026 年 4 月 30 日

* 我们已停用 Claude Sonnet 4.5 和 Claude Sonnet 4 的 1M 令牌上下文窗口 beta（`context-1m-2025-08-07`）。该 beta 标头现在对这些模型不再起作用，超过标准 200k 令牌上下文窗口的请求会返回错误。如需使用 1M 上下文窗口，请迁移到 [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison) 或 [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison)。这两个模型以标准定价提供该功能，且无需 beta 标头。

### 2026 年 4 月 29 日

* 我们发布了 [Claude API 技能](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/claude-api-skill)，这是一个开源的 [Agent Skill](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview)。它为 Claude 提供最新的参考资料，帮助其使用 8 种语言基于 Messages API 和 Claude Managed Agents 进行构建。该技能已随 Claude Code 捆绑提供，也可在 [Anthropic skills 仓库](https://github.com/anthropics/skills/tree/main/skills/claude-api)中获取。

### 2026 年 4 月 24 日

* 我们发布了[速率限制 API](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api)，管理员可以通过它以编程方式查询为其组织和工作区配置的速率限制。

### 2026 年 4 月 23 日

* Claude Managed Agents 的记忆功能现已在标准 `managed-agents-2026-04-01` 标头下进入 beta 阶段。有关完整集成指南，请参阅[使用智能体记忆](https://platform.claude.com/docs/zh-CN/managed-agents/memory)。

### 2026 年 4 月 20 日

* 我们已停用 Claude Haiku 3 模型（`claude-3-haiku-20240307`）。现在，对该模型的所有请求都将返回错误。我们建议升级到 [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison)。

### 2026 年 4 月 16 日

* 我们推出了 [Claude Opus 4.7](https://www.anthropic.com/news/claude-opus-4-7)，这是我们用于复杂推理和智能体编码的最强大模型，定价与 Opus 4.6 相同，为每 MTok $5 / $25 美元。有关能力改进、新功能和更新的分词器，请参阅 [Claude Opus 4.7 的新功能](https://platform.claude.com/docs/zh-CN/about-claude/models/whats-new-claude-4-7)。与 Opus 4.6 相比，Opus 4.7 包含 API 破坏性变更；升级前请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。
* [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 现已向所有 Amazon Bedrock 客户开放。Claude Opus 4.7 和 Claude Haiku 4.5 可通过 Bedrock 控制台自助使用，经由位于 `/anthropic/v1/messages` 的 Messages API 端点访问，覆盖 27 个 AWS 区域，提供全球端点和区域端点。
* 我们在 Claude Opus 4.7 上推出了测试版的[任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)。您可以为完整的智能体循环（思考、工具调用、工具结果和输出）为 Claude 提供一个建议性的令牌预算，模型会看到一个实时倒计时，并据此确定工作优先级，在预算消耗时从容地完成任务。请在请求中包含 `task-budgets-2026-03-13` beta 标头。
* Claude Opus 4.7 支持[高分辨率图像输入](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#high-resolution-image-support-on-claude-opus-4-7)，将长边的最大图像分辨率从 1568 像素提高到 2576 像素，从而提升计算机使用、屏幕截图理解和文档分析方面的性能。高分辨率支持是自动启用的，无需 beta 标头；图像使用的图像令牌数量可能比之前的模型多出约 3 倍。
* 我们在 Claude Opus 4.7 上新增了 `xhigh` [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。`xhigh` 介于 `high` 和 `max` 之间，专为令牌预算达数百万的长时间运行（超过 30 分钟）的智能体和编码任务而调优。无需 beta 标头。

### 2026 年 4 月 14 日

* 我们宣布弃用 Claude Sonnet 4 模型（`claude-sonnet-4-20250514`）和 Claude Opus 4 模型（`claude-opus-4-20250514`），计划于 2026 年 6 月 15 日在 Claude API 上停用。我们建议分别迁移到 [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison) 和 [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)。请在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。

### 2026 年 4 月 9 日

* 我们推出了处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)。将速度更快的执行者模型与智能水平更高的顾问模型配对，由顾问模型在生成过程中提供战略指导，使长周期智能体工作负载接近顾问模型单独运行的质量，而大部分令牌生成则按执行者模型的费率进行。请在您的请求中包含 beta 标头 `advisor-tool-2026-03-01`。

### 2026 年 4 月 8 日

* 我们推出了处于 beta 阶段的 **Claude Managed Agents**，这是一个完全托管的智能体框架，用于将 Claude 作为自主智能体运行，提供安全沙箱、内置工具和服务器发送事件流式传输。通过 API 创建智能体、配置容器并运行会话。所有端点都需要 `managed-agents-2026-04-01` beta 标头。请在 [Claude Managed Agents 概述](https://platform.claude.com/docs/zh-CN/managed-agents/overview)中了解更多信息。
* 我们推出了 **`ant` CLI**，这是一个用于 Claude API 的命令行客户端，可实现与 Claude API 更快速的交互、与 Claude Code 的原生集成，以及在 YAML 文件中对 API 资源进行版本管理。请在 [CLI 快速入门](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart)中了解更多信息。

### 2026 年 4 月 7 日

* 我们宣布 [Claude Mythos Preview](https://anthropic.com/glasswing) 作为 [Project Glasswing](https://anthropic.com/glasswing) 的一部分，以受限研究预览的形式提供，用于防御性网络安全工作。仅限受邀访问。
* [Messages API](https://platform.claude.com/docs/zh-CN/api/messages) 现已作为研究预览在 Amazon Bedrock 上提供。位于 `/anthropic/v1/messages` 的全新 Claude in Amazon Bedrock 端点使用与第一方 Claude API 相同的请求结构，并运行在 AWS 托管的基础设施上，运营方零访问权限。目前在 `us-east-1` 中可用；请联系您的 Anthropic 客户经理申请访问权限。请在 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 中了解更多信息。

### 2026 年 3 月 30 日

* 我们已将 Claude Opus 4.6 和 Sonnet 4.6 的 `max_tokens` 上限[在 Message Batches API 上提高到 300k](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#extended-output-beta)。请包含 `output-300k-2026-03-24` beta 标头，以便为长篇内容、结构化数据和大型代码生成任务生成更长的单轮输出。
* 我们将于 **2026 年 4 月 30 日**停用 Claude Sonnet 4.5 和 Claude Sonnet 4 的 1M 令牌 “context window”（上下文窗口）测试版。在该日期之后，`context-1m-2025-08-07` beta 标头将对这些模型不再生效，超出标准 200k 令牌上下文窗口的请求将返回错误。如需继续使用 1M 上下文窗口，请迁移到 [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison) 或 [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison)，它们以标准定价支持完整的 1M 令牌上下文窗口，且无需 beta 标头。

### 2026 年 3 月 18 日

* 我们在 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 中新增了模型能力字段。`GET /v1/models` 和 `GET /v1/models/{model_id}` 现在会返回 `max_input_tokens`、`max_tokens` 以及一个 `capabilities` 对象。您可以查询 API 来了解每个模型支持哪些功能。

### 2026 年 3 月 16 日

* 我们为 “extended thinking”（扩展思考）推出了 `display` 字段，允许您在响应中省略思考内容，以实现更快的流式传输。设置 `thinking.display: "omitted"` 后，您将收到 `thinking` 字段为空的思考块，同时保留 `signature` 以确保多轮对话的连续性。计费方式不变。请在[控制思考显示](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)中了解更多信息。

### 2026 年 3 月 13 日

* Claude Opus 4.6 和 Sonnet 4.6 的 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)已结束测试，按标准定价提供。对于这些模型，超过 200k 令牌的请求会自动生效，无需 beta 标头。Claude Sonnet 4.5 和 Sonnet 4 的 1M 令牌上下文窗口仍处于测试阶段。
* 我们已取消所有受支持模型专用的 1M “rate limit”（速率限制）。您的标准账户限制现在适用于所有上下文长度。
* 使用 1M 令牌上下文窗口时，我们已将每个请求的媒体限制从 100 张图像或 PDF 页面提高到 600 张。

### 2026 年 2 月 19 日

* 我们为 Messages API 推出了**自动缓存**。在请求正文中添加单个 `cache_control` 字段，系统会自动缓存最后一个可缓存块，并随着对话增长向前移动缓存点。无需手动管理断点。可与现有的块级缓存控制配合使用，以实现细粒度优化。在 Claude API 和 Microsoft Foundry（预览版）上可用。请在[自动缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)中了解更多信息。
* 我们已停用 Claude Sonnet 3.7 模型（`claude-3-7-sonnet-20250219`）和 Claude Haiku 3.5 模型（`claude-3-5-haiku-20241022`）。所有对 Claude Sonnet 3.7 的请求现在都将返回错误。在 Claude API 上对 Claude Haiku 3.5 的请求现在将返回错误；该模型在 Amazon Bedrock 和 Google Cloud 上仍然可用。我们建议分别升级到 [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison) 和 [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison)。研究人员可以通过[外部研究人员访问计划](https://support.claude.com/en/articles/9125743-what-is-the-external-researcher-access-program)申请持续访问权限。
* 我们宣布弃用 Claude Haiku 3 模型（`claude-3-haiku-20240307`），计划于 2026 年 4 月 20 日停用。我们建议迁移到 [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison)。请在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。

### 2026 年 2 月 17 日

* 我们[推出了 Claude Sonnet 4.6](https://www.anthropic.com/news/claude-sonnet-4-6)，这是我们最新的平衡型模型，为日常任务兼顾速度与智能。Sonnet 4.6 在消耗更少令牌的同时提供了更好的智能体搜索性能。Sonnet 4.6 支持[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)和 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)（beta）。有关详细信息，请参阅[模型与定价](https://platform.claude.com/docs/zh-CN/models/overview)。
* API [代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)现在**与网络搜索或网页获取一起使用时免费**。沙箱化的代码执行可提升模型能力和令牌效率。有关单独使用的情况，请参阅[定价详情](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#usage-and-pricing)。
* [网络搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)和[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)现已可用，无需 beta 标头。网络搜索和网页获取现在支持[动态过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#dynamic-filtering)，它使用代码执行在结果进入上下文窗口之前对其进行过滤，以获得更好的性能并降低令牌成本。
* [代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)、[网页获取工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)、[工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)、["tool use"（工具使用）示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#providing-tool-use-examples)和[记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)不再需要 beta 标头。

### 2026 年 2 月 7 日

* 我们为 Opus 4.6 推出了研究预览版的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)，通过 `speed` 参数显著加快输出令牌的生成速度。快速模式的速度最高可达 2.5 倍，采用溢价定价。感兴趣的客户请加入[候补名单](https://claude.com/fast-mode)。

### 2026 年 2 月 5 日

* 我们[推出了 Claude Opus 4.6](https://www.anthropic.com/news/claude-opus-4-6)，这是我们用于复杂智能体任务和长周期工作的最智能模型。Opus 4.6 推荐使用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（`thinking: {type: "adaptive"}`）；手动思考（带 `budget_tokens` 的 `type: "enabled"`）已被弃用。Opus 4.6 不支持预填充助手消息。请在 [Claude 4.6 的新功能](https://platform.claude.com/docs/zh-CN/about-claude/models/whats-new-claude-4-6)中了解更多信息。
* [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)不再需要 beta 标头，并且现在支持 Claude Opus 4.6。在新模型上，effort 取代 `budget_tokens` 来控制思考深度。
* 我们推出了测试版的[压缩 API](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)，提供服务器端上下文摘要功能，实现实际上无限长的对话。在 Opus 4.6 上可用。
* 我们引入了[数据驻留控制](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)，允许您通过 `inference_geo` 参数指定模型推理的运行位置。对于 2026 年 2 月 1 日之后发布的模型，仅限美国的推理按 1.1 倍定价提供。
* [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)现已在 Claude Opus 4.6 上以测试版提供，此外 Sonnet 4.5 和 Sonnet 4 也支持。超过 200k 输入令牌的请求适用[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)。
* [细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming)在任何模型或平台上都不再需要 beta 标头。

### 2026 年 1 月 29 日

* 对于 Claude Sonnet 4.5、Claude Opus 4.5 和 Claude Haiku 4.5，[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)已在 Claude API 上结束 beta 阶段。此版本包括扩展的 schema 支持、改进的语法编译延迟，以及无需 beta 标头的简化集成路径。`output_format` 参数已移至 `output_config.format`。现有 beta 用户在过渡期内可以继续使用 beta 标头。结构化输出在 Amazon Bedrock 和 Microsoft Foundry 上仍处于 beta 阶段。

### 2026 年 1 月 12 日

* `console.anthropic.com` 现在会重定向到 `platform.claude.com`。作为 Claude 品牌整合的一部分，Claude Console 已迁移到新地址。现有的书签和链接将通过自动重定向继续有效。更多详情，请参阅 [2025 年 9 月 16 日的公告](https://platform.claude.com/docs/zh-CN/release-notes/overview#september-16-2025)。

### 2026 年 1 月 5 日

* 我们已停用 Claude Opus 3 模型（`claude-3-opus-20240229`）。所有对该模型的请求现在都将返回错误。我们建议升级到 [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison)，它以三分之一的成本提供显著提升的智能。研究人员可以通过[外部研究人员访问计划](https://support.claude.com/en/articles/9125743-what-is-the-external-researcher-access-program)申请在 API 上持续访问 Claude Opus 3。

### 2025 年 12 月 19 日

* 我们宣布弃用 Claude Haiku 3.5 模型。请在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。

### 2025 年 12 月 4 日

* [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)现已支持 Claude Haiku 4.5。

### 2025 年 11 月 24 日

* 我们[发布了 Claude Opus 4.5](https://www.anthropic.com/news/claude-opus-4-5)，这是我们最智能的模型，兼具最强能力与实用性能。非常适合复杂的专业任务、专业软件工程和高级智能体。它在视觉、编码和计算机使用方面实现了跨越式提升，且价格比以往的 Opus 模型更加亲民。请在[模型概述](https://platform.claude.com/docs/zh-CN/models/overview)中了解更多信息。
* 我们推出了 beta 版[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)，允许 Claude 在代码执行中调用工具，以减少多工具工作流中的延迟和令牌用量。
* 我们推出了 beta 版[工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)，使 Claude 能够从大型工具目录中按需动态发现和加载工具。
* 我们为 Claude Opus 4.5 推出了 beta 版 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)，允许您通过在响应详尽程度与效率之间进行权衡来控制令牌用量。
* 我们在 Python 和 TypeScript SDK 中新增了[客户端压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#client-side-compaction-sdk)功能，在使用 `tool_runner` 时通过摘要自动管理对话上下文。

### 2025 年 11 月 21 日

* 搜索结果内容块现已在 Amazon Bedrock 上可用，无需 beta 标头。请在[搜索结果](https://platform.claude.com/docs/zh-CN/build-with-claude/search-results)中了解更多信息。

### 2025 年 11 月 19 日

* 我们在 [platform.claude.com/docs](https://platform.claude.com/docs) 推出了**全新的文档平台**。我们的文档现在与 Claude Console 并列，提供统一的开发者体验。之前位于 docs.claude.com 的文档站点将重定向到新位置。

### 2025 年 11 月 18 日

* 我们推出了 **Microsoft Foundry 中的 Claude**，为 Azure 客户带来 Claude 模型，支持 Azure 计费和 OAuth 身份验证。您可以访问完整的 Messages API，包括扩展思考、提示缓存（5 分钟和 1 小时）、PDF 支持、Files API、Agent Skills 和工具使用。请在 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 中了解更多信息。

### 2025 年 11 月 14 日

* 我们推出了 beta 版[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)，确保 Claude 的响应符合架构（schema）。使用 JSON 输出获取结构化数据响应，或使用严格工具使用获取经过验证的工具输入。适用于 Claude Sonnet 4.5 和 Claude Opus 4.1。要启用此功能，请使用 beta 标头 `structured-outputs-2025-11-13`。

### 2025 年 10 月 28 日

* 我们宣布弃用 Claude Sonnet 3.7 模型。请在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。
* 我们已停用 Claude Sonnet 3.5 模型。所有对这些模型的请求现在都将返回错误。
* 我们扩展了上下文编辑功能，新增思考块清除（`clear_thinking_20251015`），可自动管理思考块。请在[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)中了解更多信息。

### 2025 年 10 月 16 日

* 我们[推出了 Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)（`skills-2025-10-02` beta），这是一种扩展 Claude 能力的新方式。Skills 是由指令、脚本和资源组成的有组织的文件夹，Claude 会动态加载它们以执行专门任务。首次发布包括：

  * **Anthropic 托管的 Skills：** 用于处理 PowerPoint（.pptx）、Excel（.xlsx）、Word（.docx）和 PDF 文件的预构建 Skills
  * **自定义 Skills：** 通过 Skills API（`/v1/skills` 端点）上传您自己的 Skills，以封装领域专业知识和组织工作流
  * Skills 需要启用[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)
  * 请在 [Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview) 和 [API 参考](https://platform.claude.com/docs/zh-CN/api/skills/create)中了解更多信息

### 2025 年 10 月 15 日

* 我们[发布了 Claude Haiku 4.5](https://www.anthropic.com/news/claude-haiku-4-5)，这是我们速度最快、最智能的 Haiku 模型，性能接近前沿水平。非常适合实时应用、大批量处理以及需要强大推理能力的成本敏感型部署。请在[模型概述](https://platform.claude.com/docs/zh-CN/models/overview)中了解更多信息。

### 2025 年 9 月 29 日

* 我们推出了 [Claude Sonnet 4.5](https://www.anthropic.com/news/claude-sonnet-4-5)，这是我们用于复杂智能体和编码的最佳模型，在大多数任务中具有最高的智能水平。请在[模型概述](https://platform.claude.com/docs/zh-CN/models/overview)中了解更多信息。
* 我们为 Amazon Bedrock 和 Vertex AI 引入了[全球端点定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)。Claude API（1P）定价不受影响。
* 我们引入了新的停止原因 `model_context_window_exceeded`，允许您在不计算输入大小的情况下请求尽可能多的令牌。请在[处理停止原因](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)中了解更多信息。
* 我们推出了测试版的记忆工具，使 Claude 能够跨对话存储和查阅信息。请在[记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)中了解更多信息。
* 我们推出了测试版的上下文编辑功能，提供自动管理对话上下文的策略。首个版本支持在接近令牌限制时清除较早的工具结果和调用。请在[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)中了解更多信息。

### 2025 年 9 月 17 日

* 我们为 Python 和 TypeScript SDK 推出了测试版的工具辅助功能，通过类型安全的输入验证以及用于在对话中自动处理工具的工具运行器，简化了工具的创建和执行。详情请参阅 [Python SDK 工具辅助功能](https://github.com/anthropics/anthropic-sdk-python/blob/main/tools.md)和 [TypeScript SDK 工具辅助功能](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/helpers.md#tool-helpers)的文档。

### 2025 年 9 月 16 日

* 我们已将开发者产品统一到 Claude 品牌之下。您会在我们的平台和文档中看到更新后的名称和 URL，但**我们的开发者接口将保持不变**。以下是一些值得注意的变化：

  * Claude Console（[console.anthropic.com](https://console.anthropic.com)）→ Claude Console（[platform.claude.com](https://platform.claude.com)）。在 2026 年 1 月 12 日之前，控制台可通过这两个 URL 访问。在该日期之后，[console.anthropic.com](https://console.anthropic.com) 将自动重定向到 [platform.claude.com](https://platform.claude.com)。
  * Anthropic Docs（[docs.anthropic.com](https://docs.anthropic.com)）→ Claude Docs（[docs.claude.com](https://docs.claude.com)）
  * Anthropic 帮助中心（[support.anthropic.com](https://support.anthropic.com)）→ Claude 帮助中心（[support.claude.com](https://support.claude.com)）
  * API 端点、标头、环境变量和 SDK 保持不变。您现有的集成无需任何更改即可继续工作。

### 2025 年 9 月 10 日

* 我们推出了测试版的网页获取工具，允许 Claude 从指定的网页和 PDF 文档中检索完整内容。请在[网页获取工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)中了解更多信息。
* 我们推出了 [Claude Code 分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-code-analytics-api)，使组织能够以编程方式访问 Claude Code 的每日汇总使用指标，包括生产力指标、工具使用统计和成本数据。

### 2025 年 9 月 8 日

* 我们发布了 [C# SDK](https://github.com/anthropics/anthropic-sdk-csharp) 的测试版。

### 2025 年 9 月 5 日

* 我们在 Console 的[用量](https://console.anthropic.com/settings/usage)页面中推出了[速率限制图表](https://platform.claude.com/docs/zh-CN/api/rate-limits#monitoring-your-rate-limits-in-the-console)，让您可以随时间监控 API 速率限制使用情况和缓存率。

### 2025 年 9 月 3 日

* 我们推出了在客户端工具结果中支持可引用文档的功能。请在[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)中了解更多信息。

### 2025 年 9 月 2 日

* 我们推出了 beta 版 [代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool) v2，用 Bash 命令执行和直接文件操作功能（包括使用其他语言编写代码）取代了原先仅支持 Python 的工具。

### 2025 年 8 月 27 日

* 我们发布了 [PHP SDK](https://github.com/anthropics/anthropic-sdk-php) 的测试版。

### 2025 年 8 月 26 日

* 我们提高了 Claude API 上 Claude Sonnet 4 的 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)的速率限制。
* 1M 令牌上下文窗口现已在 Vertex AI 上可用。更多信息，请参阅 [Vertex AI 上的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)。

### 2025 年 8 月 19 日

* 请求 ID 现在直接包含在错误响应正文中，与现有的 `request-id` 标头并存。请在[错误结构](https://platform.claude.com/docs/zh-CN/api/errors#error-shapes)中了解更多信息。

### 2025 年 8 月 18 日

* 我们发布了[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)，允许管理员以编程方式监控其组织的使用情况和成本数据。
* 我们在 Admin API 中新增了一个用于检索组织信息的端点。详情请参阅[组织信息 Admin API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization/retrieve)。

### 2025 年 8 月 13 日

* 我们宣布弃用 Claude Sonnet 3.5 模型（`claude-3-5-sonnet-20240620` 和 `claude-3-5-sonnet-20241022`）。这些模型将于 2025 年 10 月 28 日停用。我们建议迁移到 Claude Sonnet 4.5（`claude-sonnet-4-5-20250929`），以获得更好的性能和能力。请在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。
* 提示缓存的 1 小时缓存时长不再需要 beta 标头。请在 [1 小时缓存时长](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#1-hour-cache-duration)中了解更多信息。

### 2025 年 8 月 12 日

* 我们在 Claude API 和 Amazon Bedrock 上为 Claude Sonnet 4 推出了 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)的测试版支持。

### 2025 年 8 月 11 日

* 由于 API 的加速限制，部分客户在 API 使用量急剧增加后可能会遇到 [429（`rate_limit_error`）错误](https://platform.claude.com/docs/zh-CN/api/errors)。此前，在类似情况下会出现 529（`overloaded_error`）错误。

### 2025 年 8 月 8 日

* 搜索结果内容块已在 Claude API 和 Vertex AI 上结束测试。此功能为 “retrieval-augmented generation”（检索增强生成），即 RAG 应用提供带有正确来源归属的自然引用。不再需要 beta 标头 `search-results-2025-06-09`。请在[搜索结果](https://platform.claude.com/docs/zh-CN/build-with-claude/search-results)中了解更多信息。

### 2025 年 8 月 5 日

* 我们推出了 [Claude Opus 4.1](https://www.anthropic.com/news/claude-opus-4-1)，这是 Claude Opus 4 的增量更新，具有增强的能力和性能改进。Opus 4.1 不允许同时指定 `temperature` 和 `top_p` 参数。请只使用其中一个。请在[模型概述](https://platform.claude.com/docs/zh-CN/models/overview)中了解更多信息。

### 2025 年 7 月 28 日

* 我们发布了 `text_editor_20250728`，这是一个更新的文本编辑器工具，修复了之前版本中的一些问题，并新增了一个可选的 `max_characters` 参数，允许您在查看大文件时控制截断长度。

### 2025 年 7 月 24 日

* 我们提高了 Claude API 上 Claude Opus 4 的[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)，为您提供更多容量来使用 Claude 进行构建和扩展。对于适用[使用层级 1-4 速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)的客户，这些更改会立即应用到您的账户，无需任何操作。

### 2025 年 7 月 21 日

* 我们已停用 Claude 2.0、Claude 2.1 和 Claude Sonnet 3 模型。所有对这些模型的请求现在都将返回错误。请在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。

### 2025 年 7 月 17 日

* 我们提高了 Claude API 上 Claude Sonnet 4 的[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)，为您提供更多容量来使用 Claude 进行构建和扩展。对于适用[使用层级 1-4 速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)的客户，这些更改会立即应用到您的账户，无需任何操作。

### 2025 年 7 月 3 日

* 我们推出了测试版的搜索结果内容块，为 RAG 应用提供自然引用。工具现在可以返回带有正确来源归属的搜索结果，Claude 会在其响应中自动引用这些来源，引用质量与网络搜索相当。这消除了在自定义知识库应用中使用文档变通方案的需要。请在[搜索结果](https://platform.claude.com/docs/zh-CN/build-with-claude/search-results)中了解更多信息。如需启用此功能，请使用 beta 标头 `search-results-2025-06-09`。

### 2025 年 6 月 30 日

* 我们宣布弃用 Claude Opus 3 模型。请在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中了解更多信息。

### 2025 年 6 月 23 日

* 在 Console 中，具有 Developer 角色的用户现在可以访问[费用](https://console.anthropic.com/settings/cost)页面。此前，Developer 角色可以访问[用量](https://console.anthropic.com/settings/usage)页面，但不能访问成本页面。

### 2025 年 6 月 11 日

* 我们推出了 beta 版[细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming)，该功能使 Claude 能够在不进行缓冲 / JSON 验证的情况下流式传输工具使用参数。要启用细粒度工具流式传输，请使用 [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers) `fine-grained-tool-streaming-2025-05-14`。

### 2025 年 5 月 22 日

* 我们推出了 [Claude Opus 4 和 Claude Sonnet 4](https://www.anthropic.com/news/claude-4)，这是我们具备扩展思考能力的最新模型。请在[模型概述](https://platform.claude.com/docs/zh-CN/models/overview)中了解更多信息。
* Claude 4 模型中[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)的默认行为是返回 Claude 完整思考过程的摘要，完整的思考内容会被加密，并在 `thinking` 块输出的 `signature` 字段中返回。
* 我们推出了 beta 版[交错思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#interleaved-thinking)，该功能使 Claude 能够在工具调用之间进行思考。要启用交错思考，请使用 [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers) `interleaved-thinking-2025-05-14`。
* 我们推出了 beta 版 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)，使您能够上传文件并在 Messages API 和代码执行工具中引用它们。
* 我们推出了 beta 版[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)，该工具使 Claude 能够在安全的沙盒环境中执行 Python 代码。
* 我们推出了 beta 版 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)，该功能允许您直接从 Messages API 连接到远程 MCP 服务器。
* 为了提高回答质量并减少工具错误，我们已将 Messages API 中所有模型的 `top_p` [核采样](https://en.wikipedia.org/wiki/Top-p_sampling)参数的默认值从 0.999 更改为 0.99。如需撤销此更改，请将 `top_p` 设置为 0.999。 此外，启用扩展思考时，您现在可以将 `top_p` 设置为 0.95 到 1 之间的值。
* 我们的 [Go SDK](https://github.com/anthropics/anthropic-sdk-go) 已从测试版升级为首个稳定版本。
* 我们在 Console 的[用量](https://console.anthropic.com/settings/usage)页面中添加了分钟级和小时级的粒度，并在用量页面上显示 429 错误率。

### 2025 年 5 月 21 日

* 我们的 [Ruby SDK](https://github.com/anthropics/anthropic-sdk-ruby) 已从测试版升级为首个稳定版本。

### 2025 年 5 月 7 日

* 我们在 API 中推出了网络搜索工具，允许 Claude 从网络获取最新信息。请在[网络搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)中了解更多信息。

### 2025 年 5 月 1 日

* 缓存控制现在必须直接在 `tool_result` 和 `document.source` 的父级 `content` 块中指定。为了向后兼容，如果在 `tool_result.content` 或 `document.source.content` 的最后一个块上检测到缓存控制，它将被自动应用到父级块。在 `tool_result.content` 和 `document.source.content` 中的任何其他块上设置缓存控制将导致验证错误。

### 2025 年 4 月 9 日

* 我们发布了 [Ruby SDK](https://github.com/anthropics/anthropic-sdk-ruby) 的测试版。

### 2025 年 3 月 31 日

* 我们的 [Java SDK](https://github.com/anthropics/anthropic-sdk-java) 已从测试版升级为首个稳定版本。
* 我们已将 [Go SDK](https://github.com/anthropics/anthropic-sdk-go) 从 alpha 版升级为测试版。

### 2025 年 2 月 27 日

* 我们在 Messages API 中为图像和 PDF 新增了 URL 源块。您现在可以直接通过 URL 引用图像和 PDF，而无需对其进行 base64 编码。请在[视觉](https://platform.claude.com/docs/zh-CN/build-with-claude/vision)和 [PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)中了解更多信息。
* 我们在 Messages API 的 `tool_choice` 参数中新增了对 `none` 选项的支持，该选项可阻止 Claude 调用任何工具。此外，在包含 `tool_use` 和 `tool_result` 块时，您不再需要提供任何 `tools`。
* 我们推出了与 OpenAI 兼容的 API 端点，您只需在现有的 OpenAI 集成中更改 “API key”（API 密钥）、基础 URL 和模型名称，即可测试 Claude 模型。该兼容层支持核心的聊天补全功能。请在 [OpenAI SDK 兼容性](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk)中了解更多信息。

### 2025 年 2 月 24 日

* 我们推出了 [Claude Sonnet 3.7](https://www.anthropic.com/news/claude-3-7-sonnet)，这是我们迄今为止最智能的模型。Claude Sonnet 3.7 可以生成近乎即时的响应，也可以逐步展示其扩展思考过程。一个模型，两种思考方式。请在[模型概述](https://platform.claude.com/docs/zh-CN/models/overview)中了解所有 Claude 模型的更多信息。

* 我们为 Claude Haiku 3.5 新增了视觉支持，使该模型能够分析和理解图像。

* 我们发布了一种令牌高效的工具使用实现，提升了在 Claude 中使用工具时的整体性能。请在[使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)中了解更多信息。

* 为了与 API 中的默认 temperature 保持一致，我们已将 [Console](https://console.anthropic.com/workbench) 中新提示的默认 temperature 从 0 更改为 1。现有已保存的提示保持不变。

* 我们发布了工具的更新版本，将文本编辑工具和 bash 工具与计算机使用 “system prompt”（系统提示）解耦：

  * `bash_20250124`：功能与之前版本相同，但独立于计算机使用。不需要 beta 标头。
  * `text_editor_20250124`：功能与之前版本相同，但独立于计算机使用。不需要 beta 标头。
  * `computer_20250124`：更新后的计算机使用工具，新增了命令选项，包括 `hold_key`、`left_mouse_down`、`left_mouse_up`、`scroll`、`triple_click` 和 `wait`。此工具需要 `computer-use-2025-01-24` anthropic-beta 标头。 请在[使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)中了解更多信息。

### 2025 年 2 月 10 日

* 我们在所有 API 响应中新增了 `anthropic-organization-id` 响应标头。该标头提供与请求中所用 API 密钥关联的组织 ID。

### 2025 年 1 月 31 日

* 我们已将 [Java SDK](https://github.com/anthropics/anthropic-sdk-java) 从 alpha 阶段升级到 beta 阶段。

### 2025 年 1 月 23 日

* 我们在 API 中推出了"citations"（引用）功能，使 Claude 能够为信息提供来源归属。在[引用](https://platform.claude.com/docs/zh-CN/build-with-claude/citations)中了解更多信息。
* 我们在 Messages API 中添加了对纯文本文档和自定义内容文档的支持。

### 2025 年 1 月 21 日

* 我们宣布"deprecation"（弃用）Claude 2、Claude 2.1 和 Claude Sonnet 3 模型。在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中阅读更多信息。

### 2025 年 1 月 15 日

* 我们更新了["prompt caching"（提示缓存）](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)，使其更易于使用。现在，当您设置"cache breakpoint"（缓存断点）时，我们将自动从您之前缓存的最长前缀中读取。
* 现在，您在使用工具时可以预先替 Claude 写好部分回复内容。

### 2025 年 1 月 10 日

* 我们优化了对 [Message Batches API 中的提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#using-prompt-caching-with-message-batches)的支持，以提高"cache hit rate"（缓存命中率）。

### 2024 年 12 月 19 日

* 我们在 Message Batches API 中添加了对["delete endpoint"（删除端点）](https://platform.claude.com/docs/zh-CN/api/messages/batches/delete)的支持。

### 2024 年 12 月 17 日

* 以下功能现已在 Claude API 中可用，无需"beta header"（beta 标头）：

  * [Models API](https://platform.claude.com/docs/zh-CN/api/models/list)：查询可用模型、验证模型 ID，并将["model aliases"（模型别名）](https://platform.claude.com/docs/zh-CN/models/overview)解析为其规范模型 ID。
  * [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)：以标准 API 费用的 50% 异步处理大批量消息。
  * [Token counting API](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)（令牌计数 API）：在将消息发送给 Claude 之前计算其"token"（令牌）数量。
  * [Prompt Caching](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)（提示缓存）：通过缓存和重用提示内容，将成本降低多达 90%，并将"latency"（延迟）降低多达 80%。
  * [PDF support](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)（PDF 支持）：处理 PDF，以分析文档中的文本和视觉内容。

* 我们还发布了新的官方 SDK：

  * [Java SDK](https://github.com/anthropics/anthropic-sdk-java)（alpha）
  * [Go SDK](https://github.com/anthropics/anthropic-sdk-go)（alpha）

### 2024 年 12 月 4 日

* 我们在 [Developer Console](https://console.anthropic.com) 的[用量](https://console.anthropic.com/settings/usage)和[费用](https://console.anthropic.com/settings/cost)页面上添加了按 API 密钥分组的功能。
* 我们在 [Developer Console](https://console.anthropic.com) 的 [API 密钥](https://console.anthropic.com/settings/keys)页面上新增了**上次使用**和**费用**两列，并支持按任意列排序。

### 2024 年 11 月 21 日

* 我们发布了 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)，允许用户以编程方式管理其组织的资源。

### 2024 年 11 月 20 日

* 我们更新了 Messages API 的"rate limit"（速率限制）。我们已将每分钟令牌数速率限制替换为新的每分钟输入令牌数和每分钟输出令牌数速率限制。在[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)中阅读更多信息。
* 我们在 [Console Workbench](https://console.anthropic.com/workbench) 中添加了对["tool use"（工具使用）](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)的支持。

### 2024 年 11 月 13 日

* 我们为所有 Claude Sonnet 3.5 模型添加了 PDF 支持。在 [PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)中阅读更多信息。

### 2024 年 11 月 6 日

* 我们已停用 Claude 1 和 Instant 模型。在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中阅读更多信息。

### 2024 年 11 月 4 日

* [Claude Haiku 3.5](https://www.anthropic.com/claude/haiku) 现已作为纯文本模型在 Claude API 上提供。

### 2024 年 11 月 1 日

* 我们添加了 PDF 支持，可与新的 Claude Sonnet 3.5 配合使用。在 [PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)中阅读更多信息。
* 我们还添加了令牌计数功能，让您可以在将消息发送给 Claude 之前确定其中的令牌总数。在[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)中阅读更多信息。

### 2024 年 10 月 22 日

* 我们在 API 中添加了 Anthropic 定义的"computer use"（计算机使用）工具，可与新的 Claude Sonnet 3.5 配合使用。在[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)中阅读更多信息。
* Claude Sonnet 3.5 是我们迄今为止最智能的模型，它刚刚完成升级，现已在 Claude API 上提供。在 [Claude Sonnet 文档](https://www.anthropic.com/claude/sonnet)中阅读更多信息。

### 2024 年 10 月 8 日

* Message Batches API 现已推出 beta 版。在 Claude API 中异步处理大批量查询，成本降低 50%。在["batch processing"（批处理）](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)中阅读更多信息。
* 我们放宽了 Messages API 中对 `user`/`assistant` 轮次顺序的限制。连续的 `user`/`assistant` 消息将被合并为一条消息，而不会报错，并且我们不再要求第一条输入消息必须是 `user` 消息。
* 我们已弃用 Build 和 Scale 计划，改为提供标准功能套件（以前称为 Build），以及可通过销售团队获得的附加功能。在我们的 [API 定价信息](https://claude.com/platform/api)中阅读更多信息。

### 2024 年 10 月 3 日

* 我们添加了在 API 中禁用"parallel tool use"（并行工具使用）的功能。在 `tool_choice` 字段中设置 `disable_parallel_tool_use: true`，以确保 Claude 最多使用一个工具。在[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)中阅读更多信息。

### 2024 年 9 月 10 日

* 我们在 [Developer Console](https://console.anthropic.com) 中添加了 Workspaces（工作区）。工作区允许您设置自定义支出或速率限制、对 API 密钥进行分组、按项目跟踪用量，并通过用户角色控制访问权限。请在我们的 [Workspaces 博客文章](https://www.anthropic.com/news/workspaces)中了解更多信息。

### 2024 年 9 月 4 日

* 我们宣布弃用 Claude 1 模型。在[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)中阅读更多信息。

### 2024 年 8 月 22 日

* 我们通过在 API 响应中返回"Cross-Origin Resource Sharing"（跨源资源共享），即 CORS 标头，添加了对在浏览器中使用 SDK 的支持。在 SDK 实例化时设置 `dangerouslyAllowBrowser: true` 即可启用此功能。

### 2024 年 8 月 19 日

* Claude Sonnet 3.5 上的 8,192 令牌输出已结束 beta 阶段，不再需要 `max-tokens-3-5-sonnet-2024-07-15` 标头。

### 2024 年 8 月 14 日

* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)现已作为 beta 功能在 Claude API 中提供。缓存并重用提示，可将延迟降低多达 80%，并将成本降低多达 90%。

### 2024 年 7 月 15 日

* 使用新的 `anthropic-beta: max-tokens-3-5-sonnet-2024-07-15` 标头，从 Claude Sonnet 3.5 生成长度最多为 8,192 个令牌的输出。

### 2024 年 7 月 9 日

* 在 [Developer Console](https://console.anthropic.com) 中使用 Claude 为您的提示自动生成"test cases"（测试用例）。
* 在 [Developer Console](https://console.anthropic.com) 新的输出比较模式中并排比较不同提示的输出。

### 2024 年 6 月 27 日

* 在 [Developer Console](https://console.anthropic.com) 新的[用量](https://console.anthropic.com/settings/usage)和[费用](https://console.anthropic.com/settings/cost)标签页中，查看按美元金额、令牌数量和 API 密钥细分的 API 使用情况和账单。
* 在 [Developer Console](https://console.anthropic.com) 新的[速率限制选项卡](https://console.anthropic.com/settings/limits)中查看您当前的 API 速率限制。

### 2024 年 6 月 20 日

* [Claude Sonnet 3.5](https://www.anthropic.com/news/claude-3-5-sonnet) 是我们迄今为止最智能的模型，现已在 Claude API、Amazon Bedrock 和 Vertex AI 上提供。

### 2024 年 5 月 30 日

* [工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)已在 Claude API、Amazon Bedrock 和 Vertex AI 上结束 beta 阶段，无需 beta 标头。

### 2024 年 5 月 10 日

* 我们的提示生成器工具现已在 [Developer Console](https://console.anthropic.com) 中可用。提示生成器可帮助您引导 Claude 生成针对您特定任务量身定制的高质量提示。请在我们的[提示生成器博客文章](https://www.anthropic.com/news/prompt-generator)中了解更多信息。
