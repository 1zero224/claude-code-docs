---
title: 适用于 Claude API 的 OpenTelemetry 追踪
url: https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry
description: 使用来自 Anthropic SDK 和 Claude API 的 OpenTelemetry span 追踪您的 Messages API 请求、工具使用和 Managed Agents 会话。
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
  OpenTelemetry 导出功能处于测试阶段。它根据您与 Anthropic 之间的协议提供，包括[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)中的 Beta Services 部分。
</Note>

<video aria-label="在 Claude Platform 上使用 OpenTelemetry 追踪智能体">
  <source src="https://media.platform.claude.com/docs/opentelemetry/tracing/overview-1080p.mp4" type="video/mp4" media="(min-width: 1280px)" />
  <source src="https://media.platform.claude.com/docs/opentelemetry/tracing/overview-720p.mp4" type="video/mp4" media="(min-width: 640px)" />
  <source src="https://media.platform.claude.com/docs/opentelemetry/tracing/overview-480p.mp4" type="video/mp4" />
</video>

Claude API 和 SDK 可以使用 [OpenTelemetry](https://opentelemetry.io/)（OTel）追踪您的 Messages API 请求、工具使用和 Managed Agents 会话。您可以在已有的可观测性平台中查看这些追踪。您可以使用它们调试失败或缓慢的请求、跟踪令牌消耗和延迟，并基于真实流量构建评估。您还可以在应用程序自身的追踪中看到每个请求，并将其关联到发送该请求的用户或功能。

OpenTelemetry 是一个大多数可观测性平台都支持的供应商中立标准。"trace"（追踪）是一个请求在您的系统中的完整记录。"span"（跨度）是追踪中的一个步骤，例如一次模型请求或一次工具使用。API 和 SDK 使用 [GenAI 语义约定](https://github.com/open-telemetry/semantic-conventions-genai)中的属性，因此支持这些约定的平台无需额外设置即可识别这些 span。

## 入门

您可以设置 SDK 追踪、API 追踪，或两者都设置。同时设置两者，即可在同一追踪中看到请求的 SDK span 和 API span，如[示例追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#example-traces-and-how-to-read-them)所示。

让 Claude Code 为您设置此功能

```text
使用 Claude API 和 Anthropic SDK 设置 OpenTelemetry 追踪
```

<CardGroup cols={3}>
  <Card title="设置 SDK 追踪" icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk" layout="horizontal">
    安装 SDK 并注册 tracer provider。
  </Card>

  <Card title="设置 API 追踪" icon="cloud" href="https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api" layout="horizontal">
    在 Claude Console 中添加目标。
  </Card>

  <Card title="Span 参考" icon="book" href="https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans" layout="horizontal">
    查阅 span 及其属性。
  </Card>
</CardGroup>

| 您获得的内容                                                                                                          | SDK 追踪                                                                       | API 追踪                                                                                      | 两者结合                                                                                        |
| --------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 请求                                   | ✅                                                                            | ✅                                                                                           | ✅                                                                                           |
| [Sessions API](https://platform.claude.com/docs/zh-CN/managed-agents/sessions) 请求                               | ✅                                                                            | ✅                                                                                           | ✅                                                                                           |
| 令牌用量（参见[令牌统计](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#count-tokens)）                 | ✅                                                                            | ✅                                                                                           | ✅                                                                                           |
| 请求中的每次模型运行                                                                                                      | —                                                                            | ✅                                                                                           | ✅                                                                                           |
| [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 轮次                             | —                                                                            | ✅                                                                                           | ✅                                                                                           |
| 由 API 运行的[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)                 | —                                                                            | ✅                                                                                           | ✅                                                                                           |
| 由[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)运行的客户端工具                  | ✅                                                                            | —                                                                                           | ✅                                                                                           |
| [自托管沙箱](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#trace-a-self-hosted-sandbox)中的工具 | ✅                                                                            | —                                                                                           | ✅                                                                                           |
| 其他 API 请求，例如上传文件                                                                                                | ✅                                                                            | —                                                                                           | ✅                                                                                           |
| 设置                                                                                                              | [代码](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk) | [Console](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api)           | 两者                                                                                          |
| 所需权限                                                                                                            | 无                                                                            | [管理员](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#what-you-need) | [管理员](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#what-you-need) |
| 覆盖范围                                                                                                            | 使用 SDK 的客户端                                                                  | 任何客户端                                                                                       | 任何客户端                                                                                       |
| 云平台支持                                                                                                           | 任何                                                                           | Claude Platform<br />Claude Platform on AWS                                                 | Claude Platform<br />Claude Platform on AWS                                                 |

对于 Claude Code、Agent SDK 或 Cowork，请参阅 [Claude Code 监控](https://code.claude.com/docs/en/monitoring-usage)、[Agent SDK 可观测性](https://code.claude.com/docs/en/agent-sdk/observability)或 [Cowork 活动监控](https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry)。

## 将追踪发送到合作伙伴平台

将追踪发送到任何接受 OTLP 的可观测性平台，包括以下合作伙伴：

<CardGroup cols={3}>
  <Card title="Arize AI" image="https://platform.claude.com/docs/images/partner-logos/arize.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/arize-dark.svg" imageOffsetLeft={2} href="https://arize.com/docs/ax/integrations/platforms/claude-managed-agents/claude-managed-agents-tracing">
    面向 AI 的统一可观测性与评估平台。
  </Card>

  <Card title="Braintrust" image="https://platform.claude.com/docs/images/partner-logos/braintrust.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/braintrust-dark.svg" imageHeight={25} imageOffsetLeft={1} href="https://www.braintrust.dev/docs/integrations/ai-providers/anthropic#export-opentelemetry-traces-from-the-claude-api">
    面向智能体的主动可观测性平台。
  </Card>

  <Card title="Databricks" image="https://platform.claude.com/docs/images/partner-logos/databricks.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/databricks-dark.svg" imageHeight={26} imageOffsetLeft={1} href="https://docs.databricks.com/aws/en/ai-gateway/unified-trace-table">
    面向可观测性与评估的统一 AI 控制平面。
  </Card>

  <Card title="Datadog" image="https://platform.claude.com/docs/images/partner-logos/datadog.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/datadog-dark.svg" imageHeight={28} imageOffsetLeft={2} href="https://docs.datadoghq.com/llm_observability/instrument/otel_instrumentation/?tab=python#using-claude-managed-agents">
    AI 驱动的可观测性与安全。
  </Card>

  <Card title="Dynatrace" image="https://platform.claude.com/docs/images/partner-logos/dynatrace.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/dynatrace-dark.svg" imageHeight={22} href="https://docs.dynatrace.com/docs/observe/dynatrace-for-ai-observability/integrations/claude-platform">
    为 AI 时代打造的可观测性。
  </Card>

  <Card title="Elastic" image="https://platform.claude.com/docs/images/partner-logos/elastic.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/elastic-dark.svg" imageHeight={32} href="https://www.elastic.co/docs/reference/opentelemetry/motlp">
    基于开源构建的可观测性。
  </Card>

  <Card title="Grafana Cloud" image="https://platform.claude.com/docs/images/partner-logos/grafana.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/grafana-dark.svg" imageHeight={34} imageOffsetLeft={2} href="https://grafana.com/docs/grafana-cloud/observe-and-act/agent-observability/guides/send-claude-platform-traces/">
    完全托管的开放可观测性平台。
  </Card>

  <Card title="Honeycomb" image="https://platform.claude.com/docs/images/partner-logos/honeycomb.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/honeycomb-dark.svg" imageHeight={36} imageOffsetLeft={4} href="https://docs.honeycomb.io/send-data/use-cases/claude-platform">
    面向软件新形态的可观测性平台。
  </Card>

  <Card title="Langfuse" image="https://platform.claude.com/docs/images/partner-logos/langfuse.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/langfuse-dark.svg" imageHeight={26} imageOffsetLeft={1} href="https://langfuse.com/integrations/model-providers/claude-platform">
    开源的智能体可观测性与评估平台。
  </Card>

  <Card title="LangSmith" image="https://platform.claude.com/docs/images/partner-logos/langchain.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/langchain-dark.svg" imageHeight={29} href="https://docs.langchain.com/langsmith/trace-anthropic">
    LangSmith 智能体工程平台。
  </Card>

  <Card title="New Relic" image="https://platform.claude.com/docs/images/partner-logos/new-relic.svg" imageHeight={27} imageOffsetLeft={1} href="https://docs.newrelic.com/docs/ai-monitoring/opentelemetry-ai-monitoring/#anthropic">
    智能可观测性平台。
  </Card>

  <Card title="PostHog" image="https://platform.claude.com/docs/images/partner-logos/posthog.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/posthog-dark.svg" imageHeight={29} href="https://posthog.com/docs/ai-observability/installation/anthropic?tab=Claude+Platform&utm_source=anthropic&utm_campaign=anthropic-otel">
    用于构建成功产品的一体化平台。
  </Card>

  <Card title="Pydantic Logfire" image="https://platform.claude.com/docs/images/partner-logos/pydantic.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/pydantic-dark.svg" imageHeight={24} href="https://pydantic.dev/docs/logfire/integrations/llms/claude-platform/">
    生产级 AI 与通用可观测性。
  </Card>

  <Card title="Sentry" image="https://platform.claude.com/docs/images/partner-logos/sentry.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/sentry-dark.svg" imageHeight={25} href="https://sentry.io/cookbook/trace-claude-platform-with-opentelemetry/">
    调试出错的代码和行为异常的智能体。
  </Card>

  <Card title="Snowflake" image="https://platform.claude.com/docs/images/partner-logos/snowflake.svg" imageHeight={30} imageOffsetLeft={2} href="https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-ai-gateway/observability#trace-claude-requests">
    内置可观测性与治理的智能体控制平面。
  </Card>

  <Card title="Splunk" image="https://platform.claude.com/docs/images/partner-logos/splunk.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/splunk-dark.svg" imageHeight={27} imageOffsetLeft={1} href="https://agent-observability-docs.splunk.com/sdk-api/third-party-integrations/claude">
    观测、评估并保护智能体。
  </Card>
</CardGroup>

## 内容模式

每个目标和每个 SDK 客户端都有一个内容模式，用于设置 span 是否包含模型提示和响应。请在[添加目标](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#add-a-destination)或[配置 SDK](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#configure-the-sdk) 时进行设置。

* **仅元数据**（SDK 中为 `metadata_only`）：默认值。span 包含标识符、时间、状态、令牌计数以及您选择的文本，例如您的智能体和工具的名称与描述，以及您的 MCP 服务器的名称。
* **完整内容**（SDK 中为 `content`）：可见性最高。span 包含模型提示和响应，以及"仅元数据"中的所有内容。

使用"完整内容"，您还可以查看运行失败或缓慢的原因、阅读完整对话，并基于真实追踪构建评估。在 [OpenTelemetry span 参考](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans)中，包含提示和响应的属性会标记为 <Badge>内容</Badge>。

<Info>
  您的可观测性平台会根据其自身的访问和保留设置存储这些内容，因此请检查它们是否满足您组织的要求。
</Info>

如果您的组织与 Anthropic 之间存在 HIPAA、FERPA 或 K-12 数据处理限制，则无论目标的内容模式如何，目标都只导出元数据。SDK span 不受影响。

启用[零数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)后，您仍然可以使用"完整内容"。对于 Messages API 请求，API 仅将导出的内容保留在内存中，直到将其传送到目标。[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 不符合零数据保留的条件，并且在您使用 OpenTelemetry 时，其数据保留方式保持不变。

### 内容限制

使用"完整内容"时，每个 span 默认最多保存 0.75 MB 的内容。这足以容纳 200k 令牌上下文窗口的英文文本，并且低于许多可观测性平台和代理的 1 MB 请求限制。如果您选择合作伙伴平台作为**提供商**，Console 会设置该平台可接受的限制。

当内容超过限制时，SDK 或 API 会缩短内容，并将 `anthropic.content.truncated` 设置为 `true`。请求 span 会先移除系统提示和最早的消息，工具 span 会从结果末尾移除块。图像和文档仅显示其类型。

在以下情况下更改此限制：

* **您的可观测性平台可接受更大的有效负载：** 提高或移除限制，以完整保留较长的提示和响应。填满 1M 令牌上下文窗口大约需要 4 MB。
* **您的可观测性平台或其前面的代理会拒绝较大的请求：** 降低限制。

在 SDK 中，请设置 `traces.max_content_bytes` (typescript, java, php: `traces.maxContentBytes`; csharp: `Traces.MaxContentBytes`; go: `option.WithOpenTelemetryTracesMaxContentBytes()`)。在 Console 中，请在[添加目标](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#add-a-destination)时将**内容模式**设置为**完整内容**，然后在**高级设置**下设置**最大内容大小（MB）**。

## 示例追踪及其解读方法

### Messages API 请求

在[创建消息](https://platform.claude.com/docs/zh-CN/api/messages/create)的请求的 API span 之下，API 会为每次模型请求添加一个 span，并为 API 运行的每次服务器工具或 MCP 连接器工具使用添加一个工具 span。

在此示例中，您的应用程序调用了[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)，即运行您的工具并将其结果发送回 Claude 的 SDK 辅助工具。工具运行器发送的每个请求都有一个来自 SDK 的客户端 span 和一个来自 API 的服务器 span。这些 span 按以下顺序出现：

1. 您的应用程序启动 `answer_question` span，并在其中调用 Messages 工具运行器（显示为 [`anthropic.messages.tool_runner`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-tool-runner) span）。
2. 工具运行器发送第一个请求（显示为第一个 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) 客户端 span 及其下的 span）。在此请求中，API 在两次模型请求之间运行网络搜索工具。
3. 在第二次模型请求中，模型请求使用自定义的 `get_weather` 工具，因此第一个请求结束。
4. 工具运行器运行 `get_weather`（显示为 `anthropic.custom_tool_use get_weather` span）。
5. 工具运行器发送带有工具结果的第二个请求（显示为第二个 `anthropic.messages.create` 客户端 span 及其下的 span）。
6. 在第二个请求中，模型给出最终答案。

![Messages 工具运行器 trace（追踪）示例：两个请求，包含客户端和服务器 span（跨度）、模型请求、web\_search 和 get\_weather](https://platform.claude.com/docs/zh-CN/images/opentelemetry-sdk-trace-shape.svg)

您应用程序中的 SDK 会发出 [`anthropic.messages.tool_runner`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-tool-runner) span、[`anthropic.custom_tool_use`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use) span 以及每个请求的客户端 span。要获取这些 span，请[设置 SDK 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

每个请求的令牌用量会出现在其多个 span 上。要确保每个令牌只计算一次，请参阅[令牌统计](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#count-tokens)。

使用[完整内容](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#choose-a-content-mode)时，`anthropic.messages.create` 服务器 span 包含提示和响应。系统提示位于 `gen_ai.system_instructions` 中，请求的消息位于 `gen_ai.input.messages` 中，响应位于 `gen_ai.output.messages` 中。

有关每个 span 及其属性，请参阅 [OpenTelemetry span 参考](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans)。

### Managed Agents 会话

会话中的每个 Sessions API 请求都有一个 span，每个智能体轮次都有一个轮次 span。由某个请求启动的轮次是该请求 span 的子级。一个轮次包含每个模型请求以及 API 运行的每个工具的 span，但[工具 span 参考中列出的情况](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)除外。它还包含每次结果评估的 span。

此示例跟踪一个聊天请求，在该请求中，您的应用程序向会话发送一条消息并运行一个自定义工具。该 trace 包含来自您的应用程序和 API 的 span，顺序如下：

1. 您的应用程序收到来自用户的 `POST /chat` 请求。
2. 您的应用程序启动 SDK 的会话工具运行器，该运行器列出并流式传输会话的事件（显示为 `anthropic.sessions.events.stream` 和 `anthropic.sessions.events.list` span）。
3. 您的应用程序通过调用 `sessions.events.send` 将用户的消息发送到会话。
4. 一个轮次开始，其中包括模型请求、一次 `bash` 工具使用，以及另一个线程上的一个轮次（显示为第一个 `anthropic.session.turn` span 及其下的 span）。
5. 模型请求使用[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools) `list_issues`，因此该轮次以停止原因 `requires_action` 结束。
6. 会话工具运行器运行 `list_issues`（显示为 `anthropic.custom_tool_use list_issues` span），并通过调用 `sessions.events.send` 发送其结果。
7. 第二个轮次以工具结果开始（显示为最后一个 `anthropic.session.turn` span）。

![Managed Agents 会话 trace（追踪）示例：POST /chat、四个会话事件请求、三个轮次、一次 bash 工具使用以及 list\_issues](https://platform.claude.com/docs/zh-CN/images/opentelemetry-trace-shape.svg)

您应用程序中的 SDK 会发出 [`anthropic.custom_tool_use`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use) span 以及每个请求的客户端 span。要获取这些 span，请[设置 SDK 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

一个会话的 span 可能分布在多个 trace 中，例如当您从新的根 trace 发送后续请求时。启用 [multiagent](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)（多智能体）后，一个会话还可以有多个线程。要找到您需要的 span，请按 ID 对它们进行分组。

* **单个线程：** 按 `gen_ai.conversation.id` 对 span 进行分组，该属性保存会话线程 ID。按开始时间对它们排序，即可将线程作为对话记录阅读。
* **整个会话及其所有线程：** 跨 trace 按 `anthropic.session.id` 对 span 进行分组。

使用[完整内容](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#choose-a-content-mode)时，您可以从会话的 span 中读取其内容。

* 要读取某个轮次的内容，请使用其[轮次 span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-turn)。系统提示位于线程第一个轮次 span 的 `gen_ai.system_instructions` 中，输入位于 `gen_ai.input.messages` 中（您的消息和工具结果、`system.message` 事件以及来自其他线程的消息），输出位于 `gen_ai.output.messages` 中（智能体的消息、工具使用和工具结果）。
* 要查看某次工具使用的参数和结果，请读取其[工具 span](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use)。参数位于 `gen_ai.tool.call.arguments` 中，结果位于 `anthropic.tool_result.content` 中。

有关每个 span 及其属性，请参阅 [OpenTelemetry span 参考](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans)。

## 令牌统计

要使每个令牌只计数一次，请对每个请求仅汇总一个来源的 span。当一个 Messages API 请求同时具有 SDK span 和 API span 时，请汇总其 API span。两者都命名为 `anthropic.messages.create`，因此请按类型区分它们：SDK span 为 `CLIENT`，其子级 API span 为 `SERVER`。

对于每个来源，请仅汇总以下 span，因为某些用量属性会在 API span 的多个层级上重复出现：

* **SDK span：** 汇总 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span。如果请求包含 [compaction](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)（压缩），还需加上该 span 的 `anthropic.usage.compaction.input_tokens` 和 `anthropic.usage.compaction.output_tokens`，因为 `gen_ai.usage.*` 属性不包含压缩令牌。如果请求由[备用模型](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)处理，其 span 仅包含实际提供响应的模型的用量。
* **Messages API 请求的 API span：** 汇总 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span 或其 [`anthropic.model_request`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-model-request) span，但不要两者都汇总。如果请求包含压缩，还需加上其 [`anthropic.compaction`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-compaction) span。不要再额外加上 `anthropic.messages.create` span 的 `anthropic.usage.compaction.*` 属性，因为它们包含相同的令牌。
* **Managed Agents 会话的 API span：** 汇总 `anthropic.model_request` 和 [`anthropic.session.outcome_evaluation`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-session-outcome-evaluation) span。

span 可能会多次到达，例如当投递被重试时。如果您的可观测性平台不会删除重复项，请按 trace ID 和 span ID 对每个 span 只计数一次。

`gen_ai.usage.input_tokens` 已包含从提示缓存读取和写入提示缓存的令牌，因此不要再将缓存属性加到其中。有关所有用量属性，请参阅[令牌用量](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#token-usage)。

<Note>
  导出的 span 并不保证是 API 用量的完整记录。span 的交付是[尽力而为](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#delivery-and-health)的，因此 span 可能会丢失。调用方还可以关闭请求的 [`traceparent` 标头](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#connect-api-spans-to-your-own-traces)中的采样标志，从而省略该请求的 span。要跟踪每个 API 密钥的用量（例如发现泄露的密钥），请使用[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 并按 `api_key_id` 分组。它也会统计没有 span 的请求的用量。用量通常会在 4 小时内显示在其中（请参阅[数据新鲜度](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#data-freshness)）。
</Note>

## 可用性

OpenTelemetry在所有模型上处于 beta 阶段。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态   | 云平台     | 模型                                                             |
| ---- | ------- | -------------------------------------------------------------- |
| Beta | 所有支持的平台 | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台     | 模型                                                             |
| ---- | ------- | -------------------------------------------------------------- |
| Beta | 所有支持的平台 | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |
