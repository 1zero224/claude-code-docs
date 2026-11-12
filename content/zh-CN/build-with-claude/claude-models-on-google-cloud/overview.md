---
title: Claude Models on Google Cloud
url: https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/overview
description: 比较 Claude Models on Google Cloud (Anthropic managed) 和 Claude Models on Google Cloud (Google managed)，并了解您使用的是哪一种。
---

您可以通过两种方式在您的 Google Cloud 项目中使用 Claude：

* **[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)：** 由 Anthropic 运营 [Claude API](https://platform.claude.com/docs/zh-CN/api/overview)。
* **[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)：** 由 Google 将 Claude 模型作为 Google Cloud’s Agent Platform 的一部分进行运营。

## 两种 Google Cloud 产品的区别

下表比较了 Claude Models on Google Cloud (Anthropic managed) 与 Claude Models on Google Cloud (Google managed)。

| 方面                          | [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)                       | [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)                              |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **API 接口**                  | Claude API 端点<br />`/v1/{endpoint}`                                                                                                                                                 | Agent Platform 预测端点                                                                                                                                                                  |
| **请求格式**                    | 与 Claude API 相同<br />`model` 位于请求体中<br />`anthropic-version` 标头                                                                                                                     | `model` 位于 URL 中<br />`anthropic_version` 位于请求体中                                                                                                                                     |
| **Streaming（流式传输）**         | 在请求体中设置 `"stream": true`                                                                                                                                                            | 单独的 `:streamRawPredict` 端点                                                                                                                                                           |
| **位置**                      | `global`<br />请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。 | 全球、多区域或区域端点<br />请参阅[全球、多区域和区域端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#global-multi-region-and-regional-endpoints)。 |
| **早于 Claude 4.6 的模型的模型 ID** | Claude API ID<br />示例：`claude-haiku-4-5-20251001`                                                                                                                                   | Claude API ID，将快照日期前的连字符替换为 `@`<br />示例：`claude-haiku-4-5@20251001`<br />请参阅[模型 ID 和版本控制](https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions)。        |
| **需要在项目中启用的服务**             | 请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account)。                                          | Agent Platform API<br />`aiplatform.googleapis.com`                                                                                                                                  |
| **Anthropic 组织**            | 为您的项目创建一个新的 Anthropic 组织                                                                                                                                                            | 无                                                                                                                                                                                    |
| **计费**                      | Google Cloud Marketplace                                                                                                                                                            | Google Cloud，作为 Agent Platform 用量计费                                                                                                                                                  |
| **Rate limits（速率限制）**       | 由 Anthropic 管理                                                                                                                                                                      | 由 Google Cloud 管理                                                                                                                                                                    |

## 两种产品的共同点

* **凭据：** 两种产品都使用 Google Cloud 凭据，包括 Application Default Credentials。您需要通过各自的 IAM 角色分别授予对每种产品的访问权限。

  * [Claude Models on Google Cloud (Anthropic managed) 的凭据](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#authentication)
  * [Claude Models on Google Cloud (Google managed) 的凭据](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#before-making-api-calls)

* **计费：** 两种产品的费用都会显示在您的 Google Cloud 账单上。

  * [Claude Models on Google Cloud (Anthropic managed) 的计费](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#billing)
  * [Claude Models on Google Cloud (Google managed) 的计费](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#billing)

* **模型 ID：** Claude 4.6 及更高版本的模型在两种产品上都使用其 Claude API ID。请参阅[模型 ID 和版本控制](https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions)。

* **SDK：** Anthropic 的[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 支持两种产品。

## 在两种产品之间迁移

请按照您要迁移到的产品对应的步骤操作：

* [迁移到 Claude Models on Google Cloud (Anthropic managed) 的步骤](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#migrating-from-google-managed)
* [迁移到 Claude Models on Google Cloud (Google managed) 的步骤](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#migrating-from-anthropic-managed)

## 您使用的是哪种产品？

下表列出了用于识别 Claude Models on Google Cloud (Anthropic managed) 和 Claude Models on Google Cloud (Google managed) 的值。找到一个您已有的值。例如，查看您请求 URL 中的主机名或代码中的 SDK 客户端。该值所在列的标题就是您使用的产品。

| 检查项                            | [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)                                                                                           | [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)                                                                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **请求发送的目标**                    | `claude.googleapis.com`<br />URL 中包含您的工作区 ID。<br />URL 以 Claude API 路径结尾，例如 `.../invoke/v1/messages`。                                                                                                                                                   | 以下 Agent Platform 主机之一：<br />`aiplatform.googleapis.com`<br />`aiplatform.us.rep.googleapis.com`<br />`aiplatform.eu.rep.googleapis.com`<br />`{region}-aiplatform.googleapis.com`<br />URL 路径包含 `.../publishers/anthropic/models/{model}`。<br />URL 以 `:rawPredict` 或 `:streamRawPredict` 结尾。 |
| **代码中的 SDK 客户端**               | `AnthropicGoogleCloud` (csharp: `AnthropicGoogleCloudClient`; go: `googlecloud.NewClient()`; java: `GoogleCloudBackend`; php: `Anthropic\GoogleCloud\Client`; ruby: `Anthropic::GoogleCloudClient`)<br />在每个 SDK 中，客户端名称都包含 `GoogleCloud`（在 Go 中全部为小写）。 | `AnthropicVertex` (csharp: `AnthropicVertexClient`; go: `vertex.WithGoogleAuth()`; java: `VertexBackend`; php: `Anthropic\Vertex\Client`; ruby: `Anthropic::VertexClient`)<br />在每个 SDK 中，客户端名称都包含 `Vertex`（在 Go 中全部为小写）。                                                                      |
| **授予访问权限的 IAM 角色**             | `roles/claude.admin`<br />`roles/claude.user`<br />`roles/claude.viewer`                                                                                                                                                                                | Agent Platform IAM 角色                                                                                                                                                                                                                                                                          |
| **管理控制台**                      | Claude Console，位于 `platform.claude.com`<br />您可以从 Google Cloud 控制台打开它。                                                                                                                                                                                  | Google Cloud 控制台                                                                                                                                                                                                                                                                               |
| **每种产品在 Google Cloud 控制台中的位置** | 标记为 **Anthropic managed** 的选项<br />请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account)。                                                                           | [Model Garden](https://cloud.google.com/model-garden) 中的 Claude 模型                                                                                                                                                                                                                             |

如果您使用 API 密钥调用 `api.anthropic.com`，则您未使用这两种产品中的任何一种。要将这些调用迁移到 Claude Models on Google Cloud (Anthropic managed)，请参阅[从现有 Anthropic 组织迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#moving-from-an-existing-anthropic-organization)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="Claude Models on Google Cloud (Anthropic managed)" icon="cloud" href="https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed">
    设置您的账户并发出您的第一个请求。
  </Card>

  <Card title="Claude Models on Google Cloud (Google managed)" icon="cloud" href="https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed">
    安装 SDK 并发出您的第一个请求。
  </Card>

  <Card title="Claude Models on Google Cloud (Anthropic managed) 的 IAM 权限" icon="lock" href="https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions">
    查找权限、角色和审计日志类型。
  </Card>
</CardGroup>
