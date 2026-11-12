---
title: OpenTelemetry span 参考
url: https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans
description: Claude API 和 Anthropic SDK 发出的 OpenTelemetry span 及 span 属性。
featureMetadata:
  topic:
    title: OpenTelemetry
    url: https://platform.claude.com/docs/en/manage-claude/opentelemetry
  status: beta
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
    Amazon Bedrock: beta
    Amazon Bedrock (bedrock-runtime): beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Anthropic managed): beta
    Claude Models on Google Cloud (Google managed): beta
    Microsoft Foundry: beta
---

<Note>
  OpenTelemetry 导出目前处于测试阶段。它根据您与 Anthropic 之间的协议提供，包括[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)中的 Beta Services 部分。Span 名称和属性在测试期间可能会发生变化。
</Note>

要获取 span，请参阅[在 Anthropic SDK 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk) 和[在 Claude API 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api)。

## Span 名称和类型

| 属性                           | SDK span                                                                                       | API span                              |
| ---------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------- |
| `service.name`               | 应用程序的 `service.name`                                                                           | `claude-platform`                     |
| Instrumentation scope（插桩作用域） | `com.anthropic.sdk.<language>` 或 `com.anthropic.sdk.cli`，在云平台的客户端上再加上 `.<platform>`，并附带该软件包的版本 | `com.anthropic.claude-platform`       |
| Span kind（span 类型）           | 请求对应的 span 上为 `CLIENT`，其他为 `INTERNAL`                                                          | 请求对应的 span 上为 `SERVER`，其他为 `INTERNAL` |

* **请求的 span**（"span"（跨度））以请求的 API 方法命名：`anthropic.` 后接该方法在 API 参考中的路径，用点代替斜杠，且不含 `beta`。例如，`client.beta.sessions.events.send()` (csharp, go: `client.Beta.Sessions.Events.Send()`; java: `client.beta().sessions().events().send()`; php: `$client->beta->sessions->events->send()`; ruby: `client.beta.sessions.events.send_()`) 会发送一个[发送会话事件](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events/send)的请求，其 span 名称为 [`anthropic.sessions.events.send`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-events-send)。
* **工具 span** 的名称由固定名称、一个空格和工具名称组成，例如 [`anthropic.tool_use bash`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)。

## Span 状态

* **`OK`：** 该步骤已完成。
* **`ERROR`：** 该步骤失败。
* **未设置：** 应用程序取消了该步骤，例如当它发送了 [`user.interrupt`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_user_interrupt_event) 事件时。

流的 span 在流结束时结束。如果应用程序提前关闭流，SDK span 的状态为未设置，因此永不完成的流（例如 [`anthropic.sessions.events.stream`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-method)）的 SDK span 状态为未设置或 `ERROR`。

## 属性

在存在对应约定的情况下，属性使用 [OpenTelemetry 生成式 AI 语义约定](https://github.com/open-telemetry/semantic-conventions-genai)（`gen_ai.*`）。Anthropic 属性以 `anthropic.` 开头。这些 span 遵循 2026 年 9 月 24 日提交 [`e57c543b`](https://github.com/open-telemetry/semantic-conventions-genai/tree/e57c543b4889619eb2a05702471937db5119165d) 时的约定。这些约定仍在开发中。

### 通用属性

任何 span 都可以具有以下属性：

| 属性                          | 类型       | 描述                                                                                                                                                                                                                                                 |
| --------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.provider.name`      | `string` | 为请求提供服务的平台：Amazon Bedrock（包括 Mantle）上为 `aws.bedrock`，Google Cloud’s Agent Platform 上为 `gcp.vertex_ai`，其他所有情况下为 `anthropic`。出现在每个 span 上。                                                                                                           |
| `anthropic.organization.id` | `string` | 组织的 ID。                                                                                                                                                                                                                                            |
| `anthropic.workspace.id`    | `string` | 工作运行所在工作区的 ID。                                                                                                                                                                                                                                     |
| `user.id`                   | `string` | 您的最终用户的 ID，如果您在 W3C [`baggage`](https://www.w3.org/TR/baggage/) 标头中设置了它（例如 `baggage: user.id=4f2a`）。它出现在请求的 API span 以及其下的每个 API span 上，例如该请求启动的会话轮次。<br />请使用不透明的 ID，例如您数据库中该用户的键。<br />**请勿在此值中放入电子邮件地址。** 为保护用户隐私，API 会丢弃任何包含电子邮件地址的 `user.id`。 |

### 请求 span 属性

请求的每个 span 还具有以下属性，除非描述中另有说明：

| 属性                          | 类型       | 描述                                                 |
| --------------------------- | -------- | -------------------------------------------------- |
| `anthropic.request.id`      | `string` | `request-id` 响应标头的值。                               |
| `http.request.method`       | `string` | 请求的 HTTP 方法，例如 `POST`。                             |
| `http.route`                | `string` | 端点的路径，其中路径参数以占位符表示，例如 `/v1/sessions/{session_id}`。 |
| `http.response.status_code` | `int`    | 最终响应的 HTTP 状态码。对于流，为随标头发送的状态码。没有响应时不存在。            |
| `server.address`            | `string` | 请求发送到的主机，例如 `api.anthropic.com`。                   |
| `server.port`               | `int`    | 请求 URL 的端口；当 URL 没有端口时为其协议的默认端口。                   |
| `url.scheme`                | `string` | URL 的协议，例如 `https`。                                |
| `url.path`                  | `string` | URL 的路径，例如 `/v1/messages`。                         |
| `url.query`                 | `string` | URL 的查询字符串，仅包含端点定义的查询参数。没有此类参数时不存在。                |
| `url.full`                  | `string` | 请求的 URL。                                           |

对于通过云平台（例如 Claude Platform on AWS）发送的请求，API span 不包含 `server.address`、`server.port` 或 `url.full`。

### 会话属性

在 Managed Agents 会话中，span 具有以下属性。一个会话的 span 可能分布在多个 trace（追踪）中，因此要组装会话的视图（例如对话记录），请[按这些 ID 对其 span 进行分组](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#a-managed-agents-session)。

| 属性                       | 类型       | 描述                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `anthropic.session.id`   | `string` | 会话 ID。在不作用于单个会话的方法的 span 上不存在，例如 [`anthropic.sessions.list`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-method)。                                                                                                                                                                                                        |
| `gen_ai.conversation.id` | `string` | span 所属的会话线程的 ID。SDK 的 [`conversation_id` (typescript, java, php: `conversationId`; csharp: `ConversationId`; go: `option.WithOpenTelemetryConversationID()`) 选项](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#configure-the-sdk)会将其替换为您自己的会话 ID。在针对 Sessions API 请求（针对单个线程的请求除外）的 API span 上不存在；在没有线程的 SDK span 上，除非您设置了该选项，否则也不存在。 |

### 内容属性

标记为 <Badge>内容</Badge> 的属性仅存在于内容模式下的 span 上，SDK 和 API 均如此。请参阅[指南中内容模式的区别](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#choose-a-content-mode)。当这些内容过大时，SDK 和 API 会将其缩短以符合[内容限制](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#content-limits)，并将 `anthropic.content.truncated` 设置为 `true`。如果 Anthropic 的安全防护措施从响应中扣留了内容，API 的 span 中也不会包含这些内容。

消息属性是采用 OpenTelemetry GenAI 约定消息格式的 JSON 字符串。在 Messages API 以及 Managed Agents 消息事件（例如 `agent.message`）中，每个 Anthropic 内容块都会成为一个 GenAI 约定消息部分：

| Anthropic 内容块                                                          | GenAI 约定消息部分                                                                               |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `text`                                                                 | `text`                                                                                     |
| `tool_use`、`server_tool_use`                                           | `tool_call`，对于具有 `toolset_name`（例如 `computer`）的 `tool_use` 块，还具有 `anthropic_toolset_name`  |
| `mcp_tool_use`                                                         | `tool_call`，还具有 `anthropic_mcp_server_name`                                                |
| `tool_result`、`mcp_tool_result`、服务器工具的结果块（例如 `web_search_tool_result`） | `tool_call_response`                                                                       |
| `thinking`                                                             | `reasoning`，不包含其 `thinking` 块的签名。Managed Agents span 中不包含此项，因为 `agent.thinking` 事件不包含思考内容。 |
| 任何其他块，例如 `image`                                                       | 带有前缀 `anthropic_` 的块 `type`，例如 `anthropic_image`。该部分不包含其他字段。                               |

每个 Managed Agents 工具使用或工具结果事件也会成为一个 GenAI 约定消息部分：

| Managed Agents 事件                                                                        | GenAI 约定消息部分                                |
| ---------------------------------------------------------------------------------------- | ------------------------------------------- |
| `agent.tool_use`、`agent.custom_tool_use`                                                 | `tool_call`                                 |
| `agent.mcp_tool_use`                                                                     | `tool_call`，还具有 `anthropic_mcp_server_name` |
| `user.tool_result`、`user.custom_tool_result`、`agent.tool_result`、`agent.mcp_tool_result` | `tool_call_response`                        |

### 令牌用量

以下 span 具有令牌用量属性：[`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create)、[`anthropic.model_request`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-model-request)、[`anthropic.compaction`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-compaction) 和 [`anthropic.session.outcome_evaluation`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-outcome-evaluation)。这些值针对每个 span 的步骤，例如请求、模型请求或压缩。在 Messages API 请求的 API span 上，令牌计数是您的组织被计费的令牌，即使请求失败或应用程序取消了请求也是如此。SDK 的 `anthropic.messages.create` span 则具有响应报告的计数，因此对于失败或已取消的请求，这些计数可能低于 API span 上的计数，或者不存在。要使每个令牌只计数一次，请参阅[指南中如何计算令牌](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#count-tokens)。

| 属性                                                         | 类型    | 描述                                                                                                                                                                    |
| ---------------------------------------------------------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.usage.input_tokens`                                | `int` | 输入令牌，包括缓存读取和写入。                                                                                                                                                       |
| `gen_ai.usage.output_tokens`                               | `int` | 输出令牌。                                                                                                                                                                 |
| `gen_ai.usage.cache_write.input_tokens`                    | `int` | 写入提示缓存的令牌。在 [`anthropic.session.outcome_evaluation`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-outcome-evaluation) span 上不存在。  |
| `anthropic.usage.cache_creation.ephemeral_5m_input_tokens` | `int` | 写入 5 分钟缓存的令牌。仅存在于 Messages API 请求的 span 上。在流式传输响应的 SDK span 上，仅包含第一个输出令牌之前写入的令牌。                                                                                      |
| `anthropic.usage.cache_creation.ephemeral_1h_input_tokens` | `int` | 写入 1 小时缓存的令牌。仅存在于 Messages API 请求的 span 上。在流式传输响应的 SDK span 上，仅包含第一个输出令牌之前写入的令牌。                                                                                      |
| `gen_ai.usage.cache_read.input_tokens`                     | `int` | 从提示缓存读取的令牌。在 [`anthropic.session.outcome_evaluation`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-outcome-evaluation) span 上不存在。 |

## Span

span 可以来自应用程序中的 SDK，也可以来自 API，因此本参考中的每个 span 都列出了其来源。以 **Messages API：** 或 **Managed Agents：** 开头的描述适用于该产品的 span。以来源开头的属性描述（例如 **SDK：** 或 **API：**）适用于该来源的 span。

### `anthropic.messages.create`

* **来源：** SDK 和 API。
* **类型：** 来自 SDK 时为 `CLIENT`，来自 API 时为 `SERVER`。
* **SDK span 的父级：** 应用程序调用该方法时当前的 span。如果没有当前 span，则没有父级。
* **API span 的父级：** 请求的 `traceparent` 标头中的 span，当请求来自 SDK 时即为 SDK span。没有该标头时，API span 没有父级。请参阅[如何发送 `traceparent` 标头](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#connect-api-spans-to-your-own-traces)。
* **SDK span 的子级：** 该请求的 API span。
* **API span 的子级：** 每个模型请求对应一个 [`anthropic.model_request`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-model-request) span，API 在请求中运行的每次服务器工具或 MCP 连接器工具使用对应一个[工具 span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)，每次压缩对应一个 [`anthropic.compaction`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-compaction) span。

SDK 会为每个 Messages 请求发出此 span，无论是否为流式传输。

API 会为其认证的每个 [`POST /v1/messages`](https://platform.claude.com/docs/zh-CN/api/messages/create) 请求发出此 span。对于发往 [OpenAI 兼容端点](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk) `POST /v1/chat/completions` 的请求，它不会发出此 span。

API 为一个请求发出一个 span，即使该请求由[回退模型](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)提供服务也是如此。

该 span 具有[令牌用量属性](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#token-usage)。SDK span 还具有每个[请求的 SDK span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-resources-method) 的属性。

| 属性                                         | 类型         | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.operation.name`                    | `string`   | `chat`                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `gen_ai.request.model`                     | `string`   | 请求的 [`model`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.model)。                                                                                                                                                                                                                                                                                                                                                                               |
| `gen_ai.response.id`                       | `string`   | 响应的 [`id`](https://platform.claude.com/docs/zh-CN/api/messages/create#message)。                                                                                                                                                                                                                                                                                                                                                                                       |
| `gen_ai.response.model`                    | `string`   | 为请求提供服务的模型。当请求使用模型别名，或由[回退模型](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)提供服务时，它与请求的模型不同。                                                                                                                                                                                                                                                                                                                                         |
| `anthropic.message.stop_reason`            | `string`   | 响应的 [`stop_reason`](https://platform.claude.com/docs/zh-CN/api/messages#stop_reason)，例如 `end_turn`。                                                                                                                                                                                                                                                                                                                                                                   |
| `anthropic.response.service_tier`          | `string`   | 为请求提供服务的服务层级：响应的 [`usage.service_tier`](https://platform.claude.com/docs/zh-CN/api/messages#usage)。请参阅[请求如何被分配层级](https://platform.claude.com/docs/zh-CN/api/service-tiers#how-requests-get-assigned-tiers)。                                                                                                                                                                                                                                                          |
| `error.type`                               | `string`   | 失败请求的[错误类型](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)，例如 `rate_limit_error`。<br />**SDK：** 还包括[请求的 SDK span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-resources-method) 中的其他值。<br />**API：** 当 API 在没有错误响应的情况下结束请求时为 `api_error`。                                                                                                                                                                                |
| `gen_ai.response.time_to_first_chunk`      | `double`   | 仅存在于流式传输响应的 span 上。到流的第一个 `content_block_delta` 事件（如果有）的秒数。                                                                                                                                                                                                                                                                                                                                                                                                           |
| `gen_ai.usage.reasoning.output_tokens`     | `int`      | 思考令牌，它们是输出令牌的一部分。当思考令牌未与其他输出令牌分开计数时不存在，例如在响应不包含输出令牌详细信息的模型上。                                                                                                                                                                                                                                                                                                                                                                                                          |
| `anthropic.usage.compaction.input_tokens`  | `int`      | 请求的[压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)的输入令牌，包括缓存读取和写入。不属于 `gen_ai.usage.input_tokens`。当请求没有压缩时不存在。                                                                                                                                                                                                                                                                                                                              |
| `anthropic.usage.compaction.output_tokens` | `int`      | 请求的压缩的输出令牌。不属于 `gen_ai.usage.output_tokens`。当请求没有压缩时不存在。                                                                                                                                                                                                                                                                                                                                                                                                              |
| `gen_ai.request.max_tokens`                | `int`      | 请求的 [`max_tokens`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.max_tokens)。                                                                                                                                                                                                                                                                                                                                                                     |
| `gen_ai.request.stream`                    | `boolean`  | 当请求设置了 [`stream: true`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.stream) 时为 `true`。否则不存在。                                                                                                                                                                                                                                                                                                                                                    |
| `gen_ai.request.reasoning.level`           | `string`   | 请求的 [`output_config`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.output_config) 中的 [`effort`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.output_config.effort)。当请求未设置时不存在。                                                                                                                                                                                                                                              |
| `gen_ai.output.type`                       | `string`   | 当请求将输出格式设置为 JSON 时为 `json`。否则不存在。                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `anthropic.request.service_tier`           | `string`   | 请求的 [`service_tier`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.service_tier)。当请求未设置时不存在。                                                                                                                                                                                                                                                                                                                                                      |
| `anthropic.thinking.type`                  | `string`   | 请求的 [`thinking.type`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.thinking)。当请求未设置时不存在。                                                                                                                                                                                                                                                                                                                                                         |
| `anthropic.thinking.budget_tokens`         | `int`      | 请求的 [`thinking`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.thinking) 对象中的 [`budget_tokens`](https://platform.claude.com/docs/zh-CN/api/messages#thinking_config_enabled.budget_tokens)。当请求未设置时不存在。                                                                                                                                                                                                                                            |
| `anthropic.beta`                           | `string[]` | 请求的 [`anthropic-beta`](https://platform.claude.com/docs/zh-CN/api/beta/messages/create#create.betas) 标头中的 beta 名称。                                                                                                                                                                                                                                                                                                                                                    |
| `gen_ai.system_instructions`               | `string`   | <Badge>内容</Badge> 遵循 OpenTelemetry [`SystemInstructions`](https://github.com/open-telemetry/semantic-conventions-genai/blob/e57c543b4889619eb2a05702471937db5119165d/model/gen-ai/gen-ai-system-instructions.json) 模式的 JSON 字符串。请求的 [`system`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.system) 提示。                                                                                                                                          |
| `gen_ai.input.messages`                    | `string`   | <Badge>内容</Badge> 遵循 OpenTelemetry [`InputMessages`](https://github.com/open-telemetry/semantic-conventions-genai/blob/e57c543b4889619eb2a05702471937db5119165d/model/gen-ai/gen-ai-input-messages.json) 模式的 JSON 字符串。请求的 [`messages`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.messages)。                                                                                                                                                   |
| `gen_ai.output.messages`                   | `string`   | <Badge>内容</Badge> 遵循 OpenTelemetry [`OutputMessages`](https://github.com/open-telemetry/semantic-conventions-genai/blob/e57c543b4889619eb2a05702471937db5119165d/model/gen-ai/gen-ai-output-messages.json) 模式的 JSON 字符串。响应的 [`content`](https://platform.claude.com/docs/zh-CN/api/messages/create#message)，作为一条 `assistant` 消息。<br />**SDK：** 对于流，仅当应用程序按照[流追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#trace-a-stream)中所述的方式读取流时才存在。 |
| `gen_ai.request.stop_sequences`            | `string[]` | <Badge>内容</Badge> 请求的 [`stop_sequences`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.stop_sequences)。                                                                                                                                                                                                                                                                                                                                           |

请求的 span 不包含响应的服务器工具使用计数，例如 `web_search_requests`。相反，API 会为每次服务器工具使用发出一个[工具 span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)。

### `anthropic.messages.count_tokens`

* **来源：** SDK。API 不会为 [`POST /v1/messages/count_tokens`](https://platform.claude.com/docs/zh-CN/api/messages/count_tokens) 发出 span。
* **类型：** `CLIENT`。
* **父级：** 应用程序调用该方法时当前的 span；如果没有当前 span，则没有父级。
* **子级：** 无。

| 属性                                            | 类型       | 描述                             |
| --------------------------------------------- | -------- | ------------------------------ |
| `gen_ai.request.model`                        | `string` | 请求的模型。                         |
| `anthropic.message.count_tokens.input_tokens` | `int`    | 响应中的令牌计数。此计数不是用量。汇总令牌用量时请勿包含它。 |

### `anthropic.sessions.create`

与 [`anthropic.sessions.<method>`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-method) 相同，但以下子级和属性除外：

* **SDK span 的子级：** 该请求的 API span。
* **API span 的子级：** 当请求具有非空的 [`initial_events`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/create#create.initial_events) 时，第一个轮次的 [`anthropic.session.turn`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-turn) span。

| 属性                                 | 类型       | 描述                                      |
| ---------------------------------- | -------- | --------------------------------------- |
| `gen_ai.agent.id`                  | `string` | 创建该会话所用智能体的 ID。                         |
| `gen_ai.agent.name`                | `string` | 该智能体的名称。<br />**SDK：** 请求失败时不存在。        |
| `gen_ai.request.model`             | `string` | 该智能体的模型，它运行主线程。<br />**SDK：** 请求失败时不存在。 |
| `anthropic.session.environment.id` | `string` | 会话环境的 ID。                               |

### `anthropic.sessions.events.send`

与 [`anthropic.sessions.<method>`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-method) 相同，但 API span 的子级除外。

* **SDK span 的子级：** 该请求的 API span。
* **API span 的子级：** 因该请求的事件而启动的每个轮次的 [`anthropic.session.turn`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-turn) span。

### `anthropic.sessions.update`

与 [`anthropic.sessions.<method>`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-method) 相同，但 API span 的子级除外。

* **SDK span 的子级：** 该请求的 API span。
* **API span 的子级：** 因该请求提高或移除[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#resume-a-session-at-its-budget)而恢复的每个轮次的 [`anthropic.session.turn`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-turn) span。

### `anthropic.session.turn`

* **来源：** API，用于 Managed Agents 会话。

* **类型：** `INTERNAL`。

* **父级：** 取决于轮次开始的原因：

  * 对于使用非空 [`initial_events`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/create#create.initial_events) 创建的会话的第一个轮次，为 [`anthropic.sessions.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-create) span。
  * 对于因某个事件（例如 [`user.message`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_user_message_event) 事件）而开始的轮次，为发送该事件的请求的 [`anthropic.sessions.events.send`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-events-send) span。
  * 对于因请求提高或移除会话预算而恢复的轮次，为 [`anthropic.sessions.update`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-sessions-update) span。
  * 对于另一个线程上的轮次，为[会话所运行的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)启动此轮次时所处的该智能体轮次的 `anthropic.session.turn` span。
  * 对于因另一个线程上的智能体（例如由主智能体调用的智能体）回复而开始的轮次，为产生该回复的轮次的 `anthropic.session.turn` span。
  * 对于任何其他轮次（例如因某个[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)结束而开始的轮次），没有父级。

* **子级：**

  * 轮次中每个模型请求对应一个 [`anthropic.model_request`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-model-request) span。
  * API 每次在轮次中运行工具时对应一个工具 span，但[工具 span 部分中列出的情况](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)除外。
  * 另一个线程上在此轮次期间由会话所运行的智能体或由此轮次的回复启动的每个轮次，各对应一个 `anthropic.session.turn` span。
  * 此轮次启动的结果评估的每次迭代对应一个 [`anthropic.session.outcome_evaluation`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-outcome-evaluation) span。

轮次在其线程进入 `running` 状态时开始，在线程离开该状态时结束。

| 属性                                   | 类型       | 描述                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `gen_ai.operation.name`              | `string` | `invoke_agent`                                                                                                                                                                                                                                                                                                                                                                                         |
| `gen_ai.agent.id`                    | `string` | 运行该轮次线程的智能体的 ID。在[顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#advisor-threads)线程上不存在。                                                                                                                                                                                                                                                                           |
| `gen_ai.agent.name`                  | `string` | 该智能体的名称。在顾问线程上为 `anthropic.advisor`。                                                                                                                                                                                                                                                                                                                                                                   |
| `gen_ai.request.model`               | `string` | 该智能体的模型。在顾问线程上，为顾问的模型。                                                                                                                                                                                                                                                                                                                                                                                 |
| `anthropic.session.turn.stop_reason` | `string` | 轮次结束的原因：[`session.thread_status_idle`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_session_thread_status_idle_event) 事件的 [`stop_reason.type`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_session_thread_status_idle_event)，例如 `requires_action`。当线程未进入空闲状态时不存在。                                                       |
| `error.type`                         | `string` | 如果轮次失败，为结束该轮次的 [`session.error`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_session_error_event) 事件的 [`error.type`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_session_error_event)。                                                                                                                                 |
| `gen_ai.agent.description`           | `string` | 该智能体的描述。在顾问线程上或智能体没有描述时不存在。                                                                                                                                                                                                                                                                                                                                                                            |
| `gen_ai.system_instructions`         | `string` | <Badge>内容</Badge> 遵循 OpenTelemetry [`SystemInstructions`](https://github.com/open-telemetry/semantic-conventions-genai/blob/e57c543b4889619eb2a05702471937db5119165d/model/gen-ai/gen-ai-system-instructions.json) 模式的 JSON 字符串。该轮次生效的系统提示。仅存在于线程的第一个轮次上。在顾问线程上不存在。                                                                                                                                    |
| `gen_ai.input.messages`              | `string` | <Badge>内容</Badge> 遵循 OpenTelemetry [`InputMessages`](https://github.com/open-telemetry/semantic-conventions-genai/blob/e57c543b4889619eb2a05702471937db5119165d/model/gen-ai/gen-ai-input-messages.json) 模式的 JSON 字符串。该轮次的输入事件：用户消息和工具结果、[`system.message`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_system_message_event_params) 事件，以及来自其他线程的消息。在顾问线程上不存在。 |
| `gen_ai.output.messages`             | `string` | <Badge>内容</Badge> 遵循 OpenTelemetry [`OutputMessages`](https://github.com/open-telemetry/semantic-conventions-genai/blob/e57c543b4889619eb2a05702471937db5119165d/model/gen-ai/gen-ai-output-messages.json) 模式的 JSON 字符串。该轮次的输出事件：智能体的消息、工具使用和工具结果。                                                                                                                                                     |

### `anthropic.model_request`

* **来源：** API，用于 Messages API 请求和 Managed Agents 会话。

* **类型：** `INTERNAL`。

* **父级：**

  * **Messages API：** 请求的 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span。
  * **Managed Agents：** 发出该请求的轮次的 [`anthropic.session.turn`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-turn) span。

* **子级：** 无。

在 Messages API 上，每个 span 是 API 请求中的一个模型请求。

该 span 具有[令牌用量属性](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#token-usage)。

| 属性                      | 类型       | 描述                                                                                                                                                                                                                                                                                                                                                |
| ----------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.operation.name` | `string` | 仅存在于 Managed Agents 会话的 span 上，值为 `chat`。在 Messages API 上，请求的 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span 是 `chat` 操作。                                                                                                                                           |
| `gen_ai.request.model`  | `string` | 请求的模型。                                                                                                                                                                                                                                                                                                                                            |
| `gen_ai.response.model` | `string` | 仅存在于 Messages API 请求的 span 上。运行该模型请求的模型。                                                                                                                                                                                                                                                                                                          |
| `error.type`            | `string` | **Messages API：** 如果请求在此模型请求期间失败，为 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span 的 `error.type`。<br />**Managed Agents：** 如果模型请求失败，为其[错误类型](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)，例如 `rate_limit_error`。如果模型请求在没有响应的情况下失败，则为 `api_error`。 |

### `anthropic.tool_use`、`anthropic.mcp_tool_use` 和 `anthropic.custom_tool_use`

* **来源：** SDK 和 API。

* **名称：** `anthropic.tool_use <tool>`、`anthropic.mcp_tool_use <tool>` 或 `anthropic.custom_tool_use <tool>`，其中 `<tool>` 是工具的名称。

* **类型：** `INTERNAL`。

* **SDK span 的父级：** 工具启动时应用程序中当前的 span，在 Messages 工具运行器中即为 [`anthropic.messages.tool_runner`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-tool-runner) span。如果没有当前 span，则没有父级。

* **API span 的父级：**

  * **Messages API：** API 在其中运行该工具的请求的 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span。
  * **Managed Agents：** 工具运行所在轮次的 [`anthropic.session.turn`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-turn) span。对于等待 [`user.tool_confirmation`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_user_tool_confirmation_event) 事件的工具使用，即为因该确认而启动的轮次。
  * 对于来自另一个工具所运行代码的工具使用，为该工具的工具 span。在 Messages API 上，该工具是[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)。

* **SDK span 的子级：** 工具代码启动的 span。

* **API span 的子级：** 此工具运行的代码中每次工具使用对应一个工具 span。例如，代码执行工具运行的代码可能会使用 `web_search`。

span 名称的前缀取决于工具的类型：

* `anthropic.tool_use <tool>` 是 Messages API 上每次使用[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)或 Managed Agents 智能体工具集中工具的 span。API 运行智能体工具集中的工具，但在[自托管沙箱](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#trace-a-self-hosted-sandbox)中除外，此时由环境工作器运行它们。
* `anthropic.mcp_tool_use <tool>` 是在 SDK、Messages API 和 Managed Agents 中每次使用 MCP 工具时的 span。在 SDK 中，它是由 [Messages 工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner) 或[会话工具运行器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#session-tool-runner)运行的 MCP 辅助工具中的工具。在 Messages API 上，该工具来自 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)。在 Managed Agents 上，如果[与 MCP 服务器的连接失败](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector#handle-connection-and-authentication-failures)，span 名称不包含工具名称。
* `anthropic.custom_tool_use <tool>` 是每次使用由应用程序定义并由工具运行器运行的工具时的 span。只有 SDK 会发出此 span。

在以下情况下，API 不会发出工具 span：

* 模型请求使用由应用程序运行的工具，例如[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools)。如果由 SDK 中的工具运行器运行该工具，则由工具运行器发出其 span。
* **Managed Agents：** 智能体使用在自托管沙箱中运行的工具。运行该工具的 SDK 中的工作器会发出其 span。请参阅[如何追踪自托管沙箱](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#trace-a-self-hosted-sandbox)。
* **Managed Agents：** 智能体使用[多智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)工具或[顾问工具](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。
* **Managed Agents：** 智能体请求使用某个工具，但服务器根据 [`auto` 权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#let-the-server-evaluate-each-call-with-auto)拒绝了该使用，或应用程序在 [`user.tool_confirmation`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_user_tool_confirmation_event) 事件中拒绝了它。

| 属性                                    | 类型       | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.operation.name`               | `string` | `execute_tool`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `gen_ai.tool.name`                    | `string` | 工具的名称，例如 `bash`。在 API span 上，MCP 工具的名称不包含服务器名称。在 Managed Agents 上，如果与 MCP 服务器的连接失败，则为模型生成的名称，且仅出现在内容模式下的 span 上。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `gen_ai.tool.call.id`                 | `string` | **Messages API：** 模型在其中请求使用该工具的块的 `id`：[`tool_use`](https://platform.claude.com/docs/zh-CN/api/messages#tool_use_block)、[`server_tool_use`](https://platform.claude.com/docs/zh-CN/api/messages#server_tool_use_block) 或 [`mcp_tool_use`](https://platform.claude.com/docs/zh-CN/api/beta/messages#beta_mcp_tool_use_block) 块。<br />**Managed Agents：** 智能体在其中请求使用该工具的事件的 ID：[`agent.tool_use`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_agent_tool_use_event)、[`agent.mcp_tool_use`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_agent_mcp_tool_use_event) 或 [`agent.custom_tool_use`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events#beta_managed_agents_agent_custom_tool_use_event) 事件。 |
| `anthropic.mcp.server.name`           | `string` | 仅存在于 MCP 工具的 API span 上。该工具所属 MCP 服务器的名称。在 Messages API 上，为 [`mcp_tool_use`](https://platform.claude.com/docs/zh-CN/api/beta/messages#beta_mcp_tool_use_block) 块的 [`server_name`](https://platform.claude.com/docs/zh-CN/api/beta/messages#beta_mcp_tool_use_block.server_name)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `gen_ai.conversation.id`              | `string` | 会话线程 ID，如[会话属性](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#session-attributes)中所述。在 Messages 工具运行器中，为通过 [`conversation_id` (typescript, java, php: `conversationId`; csharp: `ConversationId`; go: `option.WithOpenTelemetryConversationID()`) 选项](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#configure-the-sdk)传入的对话 ID，未传入时不存在。在 Messages API 请求的 API span 上不存在。                                                                                                                                                                                                                                                                                                                                                                              |
| `anthropic.session.id`                | `string` | 会话的 ID。对于 Managed Agents 会话之外的工具使用（例如在 Messages API 上或在 Messages 工具运行器中）不存在。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `anthropic.session.thread.id`         | `string` | 仅存在于会话工具运行器的 SDK span 上。会话线程的 ID。对于主线程上的工具使用不存在。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `anthropic.tool.evaluated_permission` | `string` | 仅存在于 API 在 Managed Agents 会话中发出的 span 上。针对该工具使用评估[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)的结果：`allow` 或 `ask`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `error.type`                          | `string` | 如果工具使用失败，为错误类型：当工具代码引发错误或工具返回错误结果时为 `tool_error`。<br />**API：** 在 Messages API 上，服务器工具的错误结果改为使用其 [`error_code`](https://platform.claude.com/docs/zh-CN/api/messages#web_search_tool_result_error.error_code)，例如 `max_uses_exceeded`。在 Managed Agents 上，如果与 MCP 服务器的连接失败，认证失败时为 `mcp_authentication_failed_error`，否则为 `mcp_connection_failed_error`。如果请求在工具使用期间失败，则为请求的[错误类型](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)。                                                                                                                                                                                                                                                                                                                                                    |
| `gen_ai.tool.description`             | `string` | 仅存在于 SDK span 上。工具定义中的描述。描述为空时不存在。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `gen_ai.tool.type`                    | `string` | 仅存在于 SDK span 上。`function`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `gen_ai.tool.call.arguments`          | `string` | <Badge>内容</Badge> 遵循 OpenTelemetry [`ToolCallArguments`](https://github.com/open-telemetry/semantic-conventions-genai/blob/e57c543b4889619eb2a05702471937db5119165d/model/gen-ai/gen-ai-tool-call-arguments.json) 模式的 JSON 字符串。工具使用的 [`input`](https://platform.claude.com/docs/zh-CN/api/messages#tool_use_block)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `anthropic.tool_result.content`       | `string` | <Badge>内容</Badge> 包含内容块数组的 JSON 字符串。返回给模型的工具结果的 `content`。错误结果也包括在内，但在 Managed Agents 上与 MCP 服务器的连接失败时除外。字符串形式的 `content` 会变为包含一个 [`text`](https://platform.claude.com/docs/zh-CN/api/messages#text_block) 块的数组。<br />**API：** 在 Messages API 上，服务器工具结果中单个对象形式的 `content`（例如 `code_execution_result`）会变为包含该对象的数组。                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

### `anthropic.compaction`

* **来源：** API，用于 Messages API 请求。
* **类型：** `INTERNAL`。
* **父级：** 该请求的 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span。
* **子级：** 无。

该 span 具有压缩的[令牌用量属性](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#token-usage)。在 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) 上，压缩的用量位于 `anthropic.usage.compaction.*` 属性中，而不是 `gen_ai.usage.*` 总计中。

| 属性                      | 类型       | 描述                                                                                                                                             |
| ----------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.request.model`  | `string` | 为该压缩请求的模型，即该请求的 [`model`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.model)。                                            |
| `gen_ai.response.model` | `string` | 运行该压缩的模型。当请求使用模型别名，或由[回退模型](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)提供服务时，它与 `gen_ai.request.model` 不同。 |
| `error.type`            | `string` | 如果请求在压缩期间失败，则为该请求的[错误类型](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)。                                                       |

### `anthropic.session.outcome_evaluation`

* **来源：** API，用于 Managed Agents 会话。
* **类型：** `INTERNAL`。
* **父级：** 启动该迭代的轮次的 [`anthropic.session.turn`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-turn) span，即使该轮次在迭代之前结束也是如此。
* **子级：** 无。

当结果为 `interrupted` 时，状态未设置；对于任何其他结果，状态为 `OK`。

该 span 具有两个[令牌用量属性](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#token-usage)：`gen_ai.usage.input_tokens` 和 `gen_ai.usage.output_tokens`。

| 属性                                                 | 类型       | 描述                                                                                     |
| -------------------------------------------------- | -------- | -------------------------------------------------------------------------------------- |
| `anthropic.session.outcome.id`                     | `string` | 该迭代所评估的结果的 ID。                                                                         |
| `anthropic.session.outcome_evaluation.result`      | `string` | 该迭代的结果：`satisfied`、`needs_revision`、`max_iterations_reached`、`failed` 或 `interrupted`。 |
| `anthropic.session.outcome_evaluation.iteration`   | `int`    | 迭代编号。                                                                                  |
| `anthropic.session.outcome_evaluation.explanation` | `string` | <Badge>内容</Badge> 评分器对结果的解释，为纯文本。                                                      |

### `anthropic.messages.tool_runner`

* **来源：** SDK。
* **类型：** `INTERNAL`。
* **父级：** 循环开始时应用程序中的当前 span；如果没有当前 span，则为无。
* **子级：** 循环中每个请求的 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span，以及运行器运行的每次工具使用的[工具 span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)。

该 span 随循环结束而结束。如果应用程序停止运行器，则状态未设置。如果某个工具的代码引发错误，其工具 span 的状态为 `ERROR`，并且循环继续。该 span 没有用量属性，因此请从其 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) 子级读取用量。

| 属性                       | 类型       | 描述                                                                                                                                                                                                                                                       |
| ------------------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.operation.name`  | `string` | `invoke_agent`                                                                                                                                                                                                                                           |
| `gen_ai.request.model`   | `string` | 循环中第一个请求的模型。                                                                                                                                                                                                                                             |
| `gen_ai.conversation.id` | `string` | 通过 [`conversation_id` (typescript, java, php: `conversationId`; csharp: `ConversationId`; go: `option.WithOpenTelemetryConversationID()`) 选项](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#configure-the-sdk)传入的对话 ID。未传入时不存在。 |
| `error.type`             | `string` | 如果循环失败，则为结束循环的请求的[错误类型](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)。当运行器本身失败时为 `client_error`。                                                                                                                                        |
| `gen_ai.output.type`     | `string` | 当请求将输出格式设置为 JSON 时为 `json`。否则不存在。                                                                                                                                                                                                                        |

### `anthropic.messages.batches.<method>`

* **来源：** SDK，用于每个 Message Batches 方法。
* **类型：** `CLIENT`。
* **父级：** 应用程序调用该方法时的当前 span；如果没有当前 span，则为无。
* **子级：** 无。

这些 span 还具有每个[请求的 SDK span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-resources-method) 的属性。

| 属性                                      | 类型       | 描述                                                                                                                        |
| --------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `anthropic.message.batch.id`            | `string` | 该方法所作用的批次的 ID；对于 `create`，则为新批次的 ID。在 `list` 和失败的 `create` 上不存在。                                                          |
| `anthropic.message.batch.request_count` | `int`    | 仅在 `create` 上存在。[`requests`](https://platform.claude.com/docs/zh-CN/api/messages/batches/create#create.requests) 数组中的项目数。 |

### `anthropic.sessions.<method>`

* **来源：** SDK 和 API。
* **类型：** 来自 SDK 时为 `CLIENT`，来自 API 时为 `SERVER`。
* **SDK span 的父级：** 应用程序调用该方法时的当前 span；如果没有当前 span，则为无。
* **API span 的父级：** 请求的 [`traceparent` 标头](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#connect-api-spans-to-your-own-traces)中的 span，当请求来自 SDK 时即为 SDK span；没有该标头时为无。
* **SDK span 的子级：** 该请求的 API span。
* **API span 的子级：** 无。

API 会为每个发往 [Sessions API](https://platform.claude.com/docs/zh-CN/managed-agents/sessions) 端点的经过身份验证的请求发出一个 `anthropic.sessions.<method>` span。平台代表用户发送的请求（例如在 Claude Console 中发送的消息，或由[计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)启动的会话）也会有一个 span，每个都位于新的 trace 中。

<Accordion title="Span 名称和端点">
  | Span                                       | 端点                                                                                                                                           |
  | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
  | `anthropic.sessions.create`                | [`POST /v1/sessions`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/create)                                                       |
  | `anthropic.sessions.list`                  | [`GET /v1/sessions`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/list)                                                          |
  | `anthropic.sessions.retrieve`              | [`GET /v1/sessions/{session_id}`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/retrieve)                                         |
  | `anthropic.sessions.update`                | [`POST /v1/sessions/{session_id}`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/update)                                          |
  | `anthropic.sessions.delete`                | [`DELETE /v1/sessions/{session_id}`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/delete)                                        |
  | `anthropic.sessions.archive`               | [`POST /v1/sessions/{session_id}/archive`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/archive)                                 |
  | `anthropic.sessions.events.send`           | [`POST /v1/sessions/{session_id}/events`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events/send)                              |
  | `anthropic.sessions.events.list`           | [`GET /v1/sessions/{session_id}/events`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events/list)                               |
  | `anthropic.sessions.events.stream`         | [`GET /v1/sessions/{session_id}/events/stream`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/events/stream)                      |
  | `anthropic.sessions.resources.add`         | [`POST /v1/sessions/{session_id}/resources`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/resources/add)                         |
  | `anthropic.sessions.resources.list`        | [`GET /v1/sessions/{session_id}/resources`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/resources/list)                         |
  | `anthropic.sessions.resources.retrieve`    | [`GET /v1/sessions/{session_id}/resources/{resource_id}`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/resources/retrieve)       |
  | `anthropic.sessions.resources.update`      | [`POST /v1/sessions/{session_id}/resources/{resource_id}`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/resources/update)        |
  | `anthropic.sessions.resources.delete`      | [`DELETE /v1/sessions/{session_id}/resources/{resource_id}`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/resources/delete)      |
  | `anthropic.sessions.threads.list`          | [`GET /v1/sessions/{session_id}/threads`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/threads/list)                             |
  | `anthropic.sessions.threads.retrieve`      | [`GET /v1/sessions/{session_id}/threads/{thread_id}`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/threads/retrieve)             |
  | `anthropic.sessions.threads.archive`       | [`POST /v1/sessions/{session_id}/threads/{thread_id}/archive`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/threads/archive)     |
  | `anthropic.sessions.threads.events.list`   | [`GET /v1/sessions/{session_id}/threads/{thread_id}/events`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/threads/events/list)   |
  | `anthropic.sessions.threads.events.stream` | [`GET /v1/sessions/{session_id}/threads/{thread_id}/stream`](https://platform.claude.com/docs/zh-CN/api/beta/sessions/threads/events/stream) |
</Accordion>

该 span 具有[会话属性](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#session-attributes)。SDK span 还具有每个[请求的 SDK span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-resources-method) 的属性。

| 属性           | 类型       | 描述                                                                                                                                                                                                                                                                                                         |
| ------------ | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `error.type` | `string` | 失败请求的[错误类型](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)，例如 `rate_limit_error`。<br />**SDK：** 还包括[请求的 SDK span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-resources-method) 中的其他值。<br />**API：** 当 API 在没有错误响应的情况下关闭连接，或流式传输因 API 中的故障而结束时，为 `api_error`。 |

### `anthropic.environment.work`

* **来源：** CLI 以及 Python、TypeScript 和 Go 中的 SDK。
* **类型：** `INTERNAL`。
* **父级：** 环境工作器或工作轮询器获取工作项时应用程序中的当前 span；如果没有当前 span，则为无。在 `ant beta:worker poll` 中，该 span 是根 span。
* **子级：** 环境工作器在处理该工作项期间发送的请求的 span，以及它为该工作项运行的每个工具的[工具 span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)。

环境工作器和工作轮询器是在自托管沙箱中认领工作项的 [SDK 辅助工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#sdk-helpers)。它们为每个工作项发出一个 `anthropic.environment.work` span，从轮询返回该工作项开始，直到其处理结束。

| 属性                              | 类型       | 描述                                                                  |
| ------------------------------- | -------- | ------------------------------------------------------------------- |
| `anthropic.environment.id`      | `string` | 环境的 ID。                                                             |
| `anthropic.environment.work.id` | `string` | 工作项的 ID。                                                            |
| `anthropic.session.id`          | `string` | 该工作项所属会话的 ID。                                                       |
| `error.type`                    | `string` | 如果某个请求结束了处理，则为该失败请求对应的 span 的 `error.type`。任何其他失败均为 `client_error`。 |

### `anthropic.<resources>.<method>`

* **来源：** SDK。
* **类型：** `CLIENT`。
* **父级：** 应用程序调用该方法时的当前 span；如果没有当前 span，则为无。
* **子级：** 无。

每个发送请求且在本页中没有单独章节的 SDK 方法都会发出一个 `anthropic.<resources>.<method>` span，例如 `anthropic.models.list` 或 `anthropic.files.upload`。列表后续每一页的请求都有自己的 span。

一个 span 涵盖其整个请求（包括重试），并且当 SDK 拥有相应值时，具有 API 同名 span 的每个属性。SDK 在启动 span 时设置 `http.request.method` 和 `http.route`，以便采样器可以使用它们。

每个请求的 SDK span，包括 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create)、[`anthropic.messages.count_tokens`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-count-tokens) 和 [`anthropic.messages.batches.<method>`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-batches-method)，都具有以下属性：

| 属性                          | 类型       | 描述                                                                                                                                                                                                                                                                                                                                                                                      |
| --------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gen_ai.conversation.id`    | `string` | 通过 [`conversation_id` (typescript, java, php: `conversationId`; csharp: `ConversationId`; go: `option.WithOpenTelemetryConversationID()`) 选项](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#configure-the-sdk)传入的对话 ID。未传入时，为 span 所属的会话线程的 ID，如[会话属性](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#session-attributes)中所述；在没有线程的 span 上不存在。 |
| `anthropic.<resource>.id`   | `string` | 请求路径中的每个 ID 各对应一个属性，该方法创建的对象的 ID 也对应一个属性。名称来自路径，采用单数形式，例如 `anthropic.session.id` 或 `anthropic.session.thread.id`。                                                                                                                                                                                                                                                                       |
| `gen_ai.agent.id`           | `string` | 代理的 ID，位于代理方法上。                                                                                                                                                                                                                                                                                                                                                                         |
| `gen_ai.memory.store.id`    | `string` | 记忆存储的 ID，位于记忆存储方法上。                                                                                                                                                                                                                                                                                                                                                                     |
| `gen_ai.memory.record.id`   | `string` | 记忆的 ID，位于记忆方法上。                                                                                                                                                                                                                                                                                                                                                                         |
| `http.request.resend_count` | `int`    | SDK 出于任何原因（例如重试）重新发送请求的次数。SDK 未重新发送时不存在。                                                                                                                                                                                                                                                                                                                                                |
| `error.type`                | `string` | 失败调用的[错误类型](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)，例如 `rate_limit_error`。当错误响应没有错误类型时，为 HTTP 状态码，例如 `502`。当调用在 SDK 中失败时，为 `connection_error`、`client_timeout_error` 或 `client_error`。当服务器提前结束流式传输时，为 `api_error`。                                                                                                                                               |
| `server.address`            | `string` | 客户端基础 URL 的主机，默认为 `api.anthropic.com`。在云平台的客户端上，为该平台的主机。                                                                                                                                                                                                                                                                                                                                |
| `url.query`                 | `string` | URL 的查询字符串，仅包含端点所定义的查询参数。没有时不存在。在仅元数据模式下，当查询包含敏感参数（例如 `email`）时不存在。                                                                                                                                                                                                                                                                                                                     |
| `url.full`                  | `string` | 由 `url.scheme`、`server.address`、端口（当基础 URL 有端口时）、`url.path` 和 `url.query` 组成的 URL。在仅元数据模式下，只要该模式省略了 `url.query`，就不存在。                                                                                                                                                                                                                                                                   |
