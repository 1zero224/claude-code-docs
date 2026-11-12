# 使用 OpenTelemetry 监控 Claude Cowork 活动

本文介绍如何使用 OpenTelemetry (OTel) 监控整个组织中的 Claude Cowork 活动。通过 OTel，您的安全和运营团队可以将 Cowork 事件流式传输到您已在使用的可观测性工具中，以跟踪使用情况、调查事件和分析性能。

## 可用性

Claude Cowork 的 OpenTelemetry 监控在 Team 和 Enterprise 计划中可用。它涵盖在云中运行的 Cowork 会话（在桌面、网络和移动设备上）以及本地桌面会话。监控云中的会话需要 Claude Desktop 版本 1.22209.3 或更高版本，监控本地桌面会话需要 Claude Desktop 版本 1.1.4173 或更高版本。

新的 Claude 体验结合了聊天和 Cowork，正在向 Pro、Max、Ultra 计划推出。在 Team 计划（测试版）和 Enterprise 计划中，当启用**聊天和 Cowork 统一**时可用。尚未启用此功能的组织将聊天和 Cowork 保持为单独的选项，因此本文仍然适用。Enterprise 所有者可以在**[为您的 Enterprise 组织设置和管理新的统一 Claude 体验](https://support.claude.com/en/articles/17444462)**中了解更多信息。

---

## 您可以监控的内容

将 Claude Cowork 连接到 OpenTelemetry 收集器时，Cowork 会流式传输涵盖以下内容的事件：

- **用户提示。**当内容捕获包括用户提示时，用户提交给 Cowork 的提示的完整文本。否则，每个提示都会被记录而不包含其文本。

- **工具和 MCP 调用。**Claude 在会话期间进行的每个工具调用，包括工具名称、成功或失败以及执行时间。当内容捕获包括工具详情时，事件还会携带工具的参数（对于 MCP 工具，包括 MCP 服务器和工具名称）以及工具失败时的错误消息。

- **文件访问。**用户连接到会话的文件夹。当内容捕获包括工具详情时，事件还会携带 Claude 读取、修改或以其他方式访问的文件路径，包括通过 MCP 访问的文件。

- **技能和插件。**Claude 在会话中使用的技能和插件。除非内容捕获包括工具详情，否则非 Anthropic 提供的技能和插件会显示在占位符名称下。

- **人工批准决定。**每个工具操作是由用户批准、由用户拒绝还是根据现有权限自动启动。

- **API 请求和错误。**每个请求的模型、令牌计数、估计成本、持续时间以及返回的任何错误。

共享的 `prompt.id` 属性链接由单个用户提示触发的每个事件，因此您可以重建 Claude 为响应一个输入而执行的所有操作。

有关事件类型和属性的完整列表，请参阅 **claude.com/docs** 上的 **[Cowork 监控参考](https://claude.com/docs/cowork/monitoring#events)**。

---

## 何时使用 OpenTelemetry

OpenTelemetry 为您提供结构化 Cowork 事件的实时流，您可以将其路由到现有的 SIEM 和可观测性工具中。它是安全监控和事件调查、跟踪整个组织中的工具和文件访问模式、成本和性能分析以及在现有管道中构建仪表板和警报的正确选择。

## 兼容的目标

Cowork 的 OpenTelemetry 输出适用于任何标准 OTel 收集器。常见目标包括：

- **SIEM 平台**，如 Splunk 和 Cribl

- **日志聚合系统**，如 Elasticsearch 和 Loki

- **列式存储**，如 ClickHouse

- **可观测性平台**，如 Honeycomb 和 Datadog

您可以通过相应地配置收集器，将事件路由到多个目标。

---

## 设置 OpenTelemetry 监控

要配置 Cowork 将事件导出到您的收集器：

1. 转到**组织设置 > Cowork**。

2. 输入您的 **OTLP 端点**（您的 OpenTelemetry 收集器 URL）。

3. 选择您的收集器使用的 **OTLP 协议**：HTTP/JSON 或 HTTP/protobuf。

4. 添加身份验证所需的任何 **OTLP 标头**，例如持有者令牌。

5. 保存您的设置。

事件立即开始流向您的收集器。身份验证标头在 Anthropic 服务器上进行了加密存储。

---

## 安全和隐私考虑

在启用 OpenTelemetry 导出之前，需要注意以下几点：

- **在当前版本的 Claude Desktop 中，提示文本、Claude 的响应和工具输入默认处于关闭状态。**除非您使用 `otlpContentCapture` 设备策略启用内容捕获（您可以使用设备管理工具部署该策略），否则事件仅携带元数据。如果您启用它，请通过满足您组织日志记录该内容的策略的收集器路由事件。在 **[Cowork 监控参考中了解有关内容捕获的更多信息](https://claude.com/docs/cowork/monitoring#content-capture)**。

- **某些较旧版本的 Claude Desktop 在未设置策略时会发送内容。**该内容可能包括提示文本、Claude 的响应和工具输入。这些版本在更新后会切换到仅元数据。在支持该策略的版本上，您可以通过将 `otlpContentCapture` 设置为空列表（写作两个字符 `[]`）来在此之前关闭内容。

- **该策略适用于从 Claude Desktop 启动的 Cowork 会话，无论它们是在用户的计算机上运行还是在云中运行。**它不适用于在云中运行的计划任务，或者在聊天和 Cowork 统一的情况下，Claude 移动到云中的聊天。这些会话仅携带元数据。在网络或移动应用中启动的 Cowork 会话仅携带元数据，该策略不会改变这一点。

- **工具参数可能包含敏感值。**当内容捕获包括工具详情时，文件路径、命令参数和其他工具输入会在 `tool_input` 和 `tool_parameters` 字段中导出。相应地规划您的保留和访问策略。

- **无论内容捕获设置如何，用户电子邮件地址、连接文件夹的路径和 API 错误消息都包括在内。**API 错误消息有时可能会重复导致它的请求的一部分，因此如果这是一个问题，请在您的收集器处过滤或编辑。

- **只有当管理员配置 OTLP 端点时，事件才会被导出。**默认情况下不会流动任何数据。

---

## 将 OpenTelemetry 数据与合规性 API 结合

合规性 API 涵盖 Cowork（通过 Claude、Claude Desktop 和 Claude Mobile）和 Claude Code（通过 CLI 和 Claude Desktop）以及 Claude 聊天，为您提供一个审计跟踪，每个会话都可归因于单个用户。已在使用 OpenTelemetry 的组织可以并行运行两者，OTel 覆盖范围也包括网络和移动上的 Cowork。在 **[合规性 API 中了解有关检索远程会话的更多信息](https://platform.claude.com/docs/en/manage-claude/compliance-content-data)**。