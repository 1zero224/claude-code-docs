---
title: API 概述
url: https://platform.claude.com/docs/zh-CN/api/overview
description: 了解 Claude API 可用的端点、身份验证标头、客户端 SDK、分页、速率限制以及云平台访问选项。
---

Claude API 是位于 `https://api.anthropic.com` 的 RESTful API，提供对 Claude 模型和 Claude Managed Agents 的编程访问。

<Note>
  **初次使用 Claude？** 如需直接访问模型，请从[开始使用](https://platform.claude.com/docs/zh-CN/get-started)和[使用 Messages](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)开始。如需托管智能体基础设施，请参阅 [Claude Managed Agents 快速入门](https://platform.claude.com/docs/zh-CN/managed-agents/quickstart)。
</Note>

## 前提条件

要使用 Claude API，您需要：

* 一个 [Claude Console 账户](https://platform.claude.com/)
* 从 Console 获取的 [API 密钥](https://platform.claude.com/settings/keys)，或已配置的[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)规则

有关分步设置说明，请参阅[开始使用](https://platform.claude.com/docs/zh-CN/get-started)。

## 可用的 API

Claude API 包括以下 API：

* **[Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create)：** 向 Claude 发送消息以进行对话式交互（`POST /v1/messages`）
* **[Message Batches API](https://platform.claude.com/docs/zh-CN/api/messages/batches/create)：** 以异步方式处理大量 Messages 请求，成本降低 50%（`POST /v1/messages/batches`）
* **[Token Counting API](https://platform.claude.com/docs/zh-CN/api/messages/count_tokens)：** 在发送消息之前计算其中的令牌数，以管理成本和速率限制（`POST /v1/messages/count_tokens`）
* **[Models API](https://platform.claude.com/docs/zh-CN/api/models/list)：** 列出可用的 Claude 模型及其详细信息（`GET /v1/models`）
* **[Files API](https://platform.claude.com/docs/zh-CN/api/files/upload)：** 上传和管理文件，以便在多个 API 调用中使用（`POST /v1/files`、`GET /v1/files`）
* **[Skills API](https://platform.claude.com/docs/zh-CN/api/skills/create)：** 创建和管理自定义智能体技能（`POST /v1/skills`、`GET /v1/skills`）

以下 API 处于测试阶段：

* **[Agents API](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)：** 为 Claude Managed Agents 定义可复用、带版本的智能体配置（`POST /v1/agents`、`GET /v1/agents`）
* **[Sessions API](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)：** 在托管的云沙箱中运行有状态的智能体会话（`POST /v1/sessions`、`GET /v1/sessions/{id}/events/stream`）
* **[Environments API](https://platform.claude.com/docs/zh-CN/managed-agents/environments)：** 为智能体会话配置沙箱模板（`POST /v1/environments`、`GET /v1/environments`）

有关包含所有端点、参数和响应架构的完整 API 参考，请浏览导航中列出的 API 参考页面。要访问测试功能，请参阅 [Beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers)。

## 身份验证

有关每种身份验证方法及其使用时机的详细信息，请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。对 Claude API 的请求包含以下标头：

| 标头                       | 值                                                                                                                                                                                                                          | 是否必需                                                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `Authorization`          | `Bearer <token>`，其中 `<token>` 是您的 API 密钥，或通过[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)从 `POST /v1/oauth/token` 获取的短期访问令牌                                                | 是，除非已设置 `x-api-key`                                                                               |
| `x-api-key`              | 您在 Console 中获取的 API 密钥。`Authorization` 的旧版备用方式，仍受支持                                                                                                                                                                        | 否                                                                                                 |
| `anthropic-workspace-id` | 请求所运行的[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)的 ID（例如 `wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ`）。请参阅[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)。 | 使用未限定于某个工作区的 API 密钥时必需。对于其他 API 密钥为可选。为单个工作区创建的密钥在省略该标头时会在该工作区中运行。不适用于工作负载身份联合令牌，这类令牌在令牌交换时选择工作区。 |
| `anthropic-version`      | API 版本（例如 `2026-10-21`）                                                                                                                                                                                                    | 是                                                                                                 |
| `content-type`           | `application/json`                                                                                                                                                                                                         | 是                                                                                                 |

如果您使用[客户端 SDK](https://platform.claude.com/docs/zh-CN/api/overview#client-sdks)，SDK 会自动发送身份验证、版本和 content-type 标头；当您的密钥需要时，您需要自己传递 `anthropic-workspace-id`。有关 API 版本控制的详细信息，请参阅 [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)。

通过[云平台](https://platform.claude.com/docs/zh-CN/api/overview#claude-api-vs-cloud-platforms)访问 Claude 时，身份验证与云提供商的 IAM 系统集成。有关支持的凭据类型、所需标头和身份验证选项，请参阅特定平台的文档。

### 获取 API 密钥

API 通过网页版 [Console](https://platform.claude.com/) 提供。在 Console 中，您可以使用 [Playground](https://platform.claude.com/playground)（`platform.claude.com/playground`）在浏览器中试用 API，然后在 [API 密钥](https://platform.claude.com/settings/keys)页面（`platform.claude.com/settings/keys`）生成 API 密钥。要了解具体方法，请参阅[获取您的 Claude API 密钥](https://platform.claude.com/docs/zh-CN/get-api-key)。创建密钥时，您可以选择每个密钥的类型（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）及其[过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)。在 Console 中使用[工作区](https://platform.claude.com/settings/workspaces)来分隔环境，并按用例[控制支出](https://platform.claude.com/docs/zh-CN/api/rate-limits)。

## 客户端 SDK

Anthropic 提供官方 SDK，通过处理身份验证、请求格式化、错误处理等来简化 API 集成。

**优势：**

* 自动标头管理（身份验证、`anthropic-version`、`content-type`）
* 类型安全的请求和响应处理
* 内置重试逻辑和错误处理
* 流式传输支持
* 请求超时和连接管理

有关客户端 SDK 列表，请参阅 [SDK、CLI 和库](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。

## Claude API 与云平台

Claude 可通过第一方 Claude API 和云平台使用。请根据您的基础设施、功能可用性、合规要求和定价偏好进行选择。

### Claude API

* **直接访问**最新的模型和功能
* **Anthropic 计费和支持**
* **最适合：** 新集成、完整功能访问、与 Anthropic 的直接关系

### 云平台 API

通过 AWS、Google Cloud 或 Microsoft Azure 访问 Claude：

* 与云提供商的计费和 IAM **集成**
* **功能可用性因平台而异：** Anthropic 运营的平台包括 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 和 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)；合作伙伴运营的平台包括 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock。有关功能可用性和时间安排，请参阅每个平台的页面。
* **最适合：** 现有云承诺、特定合规要求、合并云计费

| 平台                                                | 提供商                           | 文档                                                                                                                                                                                                                                                                       |
| ------------------------------------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Claude Models on Google Cloud (Google managed)    | Google Cloud                  | [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)                                                                                                                  |
| Amazon Bedrock                                    | AWS                           | [Amazon Bedrock 中的 Claude（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)和 [Amazon Bedrock 上的 Claude（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy) |
| Claude Platform on AWS                            | AWS（Anthropic 运营）             | [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)                                                                                                                                                                |
| Claude Models on Google Cloud (Anthropic managed) | Google Cloud（Anthropic 运营）    | [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)                                                                                                            |
| Microsoft Foundry                                 | Microsoft Azure（Anthropic 运营） | [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                                      |

<Note>
  Claude Managed Agents 可通过第一方 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 和 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 使用。有关跨平台的功能可用性，请参阅[功能概述](https://platform.claude.com/docs/zh-CN/build-with-claude/overview)。
</Note>

## 请求和响应格式

### 请求大小限制

第一方 Claude API 强制执行以下请求大小限制：

| 端点                                                                                                       | 最大请求大小 |
| -------------------------------------------------------------------------------------------------------- | ------ |
| Messages、令牌计数                                                                                            | 128 MB |
| Message Batches API（请参阅[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)） | 256 MB |
| [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)                              | 500 MB |
| 会话、智能体、环境                                                                                                | 32 MB  |

如果超过这些限制，您将收到 413 `request_too_large` 错误。

<Note>
  由合作伙伴运营的平台有各自的请求大小限制：Bedrock 将请求限制为 20 MB，Claude Models on Google Cloud (Google managed) 将请求限制为 30 MB。Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed) 和 Microsoft Foundry 支持最大 32 MB 的 Messages 和令牌计数请求。有关当前数值，请查阅您所用平台的文档。
</Note>

### 响应标头

Claude API 在其响应中包含以下标头：

| 标头                          | 描述                                                                                                                                                                                                                                                                                                                                                   |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `request-id`                | 请求的全局唯一标识符，例如 `req_018EeWyXxfu5pfWkrYcMdjWG`。当您就特定请求联系支持时请包含它。请参阅[请求 ID](https://platform.claude.com/docs/zh-CN/api/errors#request-id)。                                                                                                                                                                                                              |
| `anthropic-organization-id` | 请求中使用的 API 密钥或访问令牌所属组织的 ID。                                                                                                                                                                                                                                                                                                                          |
| `anthropic-workspace-id`    | API 密钥或访问令牌解析到的[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)的 `wrkspc_` 前缀 ID，例如 `wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ`，包括当它是您组织的默认工作区时。当凭据未解析到工作区时（例如在 Admin API 请求上）或请求在身份验证完成前失败时，该标头不存在。请参阅[识别 API 响应背后的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#identify-the-workspace-behind-an-api-response)。 |

有关速率限制标头，请参阅速率限制中的[响应标头](https://platform.claude.com/docs/zh-CN/api/rate-limits#response-headers)。有关使用每个 SDK 按名称读取响应标头的示例，请参阅[识别 API 响应背后的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#identify-the-workspace-behind-an-api-response)。

<Note>
  Claude Platform on AWS 在标准 `request-id` 标头旁添加了 AWS 请求 ID（`x-amzn-requestid`）。有关双 ID 处理模式，请参阅[请求 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#request-ids)。
</Note>

## 分页

列表端点以页面形式返回结果。大多数较新的列表端点使用本节中描述的 `page` 和 `next_page` 游标方案。有些使用不同的方案；请参阅本节末尾的说明。使用 `limit` 查询参数控制页面大小，使用 `page` 查询参数获取相邻页面。每个响应都包含一个 `data` 数组以及用于在页面之间导航的游标字段。

| 名称          | 位置   | 描述                                                                      |
| ----------- | ---- | ----------------------------------------------------------------------- |
| `limit`     | 查询参数 | 每页返回的最大项目数。                                                             |
| `page`      | 查询参数 | 来自先前响应的不透明游标。在此处传递 `next_page` 或 `prev_page` 值以获取相邻页面。                  |
| `order`     | 查询参数 | 在支持排序的列表端点上，结果的排序方向（`asc` 或 `desc`）。`page` 游标仅对创建它时所用的 `order` 有效。      |
| `next_page` | 响应字段 | 下一页的游标，如果没有更多结果则为 `null`。                                               |
| `prev_page` | 响应字段 | 在支持向后分页的端点上（当前为 `GET /v1/sessions`）上一页的游标，如果您在第一页则为 `null`。其他列表端点省略此字段。 |

要返回上一页，请将 `prev_page` 作为 `page` 参数传递。当您在第一页时，`prev_page` 为 `null`。并非所有列表端点都支持 `prev_page`。只有 `GET /v1/sessions` 返回 `prev_page`；在不支持向后分页的列表端点上，该字段在响应中不存在，而不是 `null`。有关请求演练，请参阅[列出会话](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#listing-sessions)。

SDK 提供了一个自动分页迭代器，会替您跟随 `next_page`。例如，`for session in client.beta.sessions.list()` (typescript: `for await (const session of client.beta.sessions.list())`; go: `client.Beta.Sessions.ListAutoPaging()`; java: `client.beta().sessions().list().autoPager()`; csharp: `(await client.Beta.Sessions.List()).Paginate()`; php: `$client->beta->sessions->list()->pagingEachItem()`; ruby: `client.beta.sessions.list.auto_paging_each`) 会遍历每个会话。SDK 自动分页只能向前；要返回上一页，请从响应中读取 `prev_page`，并自行将其作为 `page` 参数传回。有关详细信息，请参阅 [SDK、CLI 和库](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。

<Note>
  部分列表端点使用不同的游标方案。Message Batches API（请参阅[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)）、[Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 以及若干 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 端点使用 `after_id` 和 `before_id` 查询参数，而不是 `page`。它们的响应返回 `has_more`、`first_id` 和 `last_id`，而不是 `next_page`。有关各端点的确切分页字段，请参阅相应的参考页面。
</Note>

## 速率限制和可用性

### 速率限制

该 API 强制执行速率限制和支出限制，以防止滥用并管理容量。限制按使用层级组织；您的组织会自动被分配到某个层级，并可随时间推移升级到更高层级。每个层级都有：

* **支出限制：** API 使用的每月最高费用
* **速率限制：** 每分钟最大请求数（RPM）和每分钟最大令牌数（TPM）

您可以在 [Claude Console 中的速率限制](https://platform.claude.com/usage/limits)下查看您的速率限制。您的支出限制位于 [Claude Console 中的支出限制](https://platform.claude.com/settings/spend-limits)（`platform.claude.com/settings/spend-limits`）下。如需更高的速率限制或更高的每月支出上限，请在**速率限制**页面上使用**申请提升等级**。

有关限制、层级以及用于速率限制的令牌桶算法的详细信息，请参阅[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)。

### 可用性

Claude API 在全球[许多国家和地区](https://platform.claude.com/docs/zh-CN/api/supported-regions)可用。请查看支持的地区页面以确认您所在位置的可用性。

## 后续步骤

<CardGroup cols={2}>
  <Card title="Messages API 参考" icon="book" href="https://platform.claude.com/docs/zh-CN/api/messages/create">
    直接模型交互的完整 API 规范
  </Card>

  <Card title="Claude Managed Agents 参考" icon="brain" href="https://platform.claude.com/docs/zh-CN/managed-agents/sessions">
    智能体、会话和环境端点
  </Card>

  <Card title="客户端 SDK" icon="code" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview">
    Python、TypeScript、C#、Go、Java、PHP 和 Ruby
  </Card>

  <Card title="速率限制" icon="gauge" href="https://platform.claude.com/docs/zh-CN/api/rate-limits">
    使用层级、请求更高限制以及令牌桶算法
  </Card>
</CardGroup>
