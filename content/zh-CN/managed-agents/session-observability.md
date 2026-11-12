---
title: 检查会话并跟踪用量
url: https://platform.claude.com/docs/zh-CN/managed-agents/session-observability
description: 在 Claude Console 中检查会话，读取其令牌用量和标价成本，并调试意外的智能体行为。
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
---

使用 Claude Console 中的会话查看器来检查智能体在会话中执行了哪些操作，无需编写任何代码。使用会话的 `usage` (csharp, go: `Usage`) 总计来查看这些工作消耗了多少资源。要在您的可观测性平台中查看会话的轮次、模型请求和工具使用，请设置 [OpenTelemetry 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry)。

## 在 Console 中检查会话

会话查看器仅对 Developer 和 Admin 开放。要打开它，请前往 Claude Console 中的[会话](https://platform.claude.com/sessions)页面（`platform.claude.com/sessions`）。该列表显示工作区中的每个会话及其状态、智能体、令牌用量、成本、创建者和创建时间。选择一个会话即可打开它。

会话查看器显示：

* **时间线缩略图：** 会话活动随时间变化的可缩放概览，在[多智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)会话中每个线程各占一条泳道。选择一条泳道可查看该线程，选择一个标记可跳转到其对应的事件。
* **转录文本：** 按模型请求分组的对话内容，包括思考过程、工具调用及其输入和结果，以及流式传输中的消息文本。您可以筛选事件，并将其复制或下载为 JSON。
* **检查器：** 一个可调整大小的侧边面板，通过五个选项卡展示会话的详细信息。

| 检查器选项卡  | 显示内容                                                                                                                                                                                                                                                                                                                                                              |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **会话**  | 会话的详细信息和元数据、其随时间累计的成本，以及在设置了会话[预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)时相对于该预算的支出。                                                                                                                                                                                                                                                      |
| **事件**  | 当前线程上的每个原始事件，按服务器发送的顺序排列。选择一个事件可查看其 JSON。在页面打开期间流式传输的消息还有一个**差异**视图，用于显示其[事件增量](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas)。                                                                                                                                                                                                              |
| **工具**  | 会话中智能体所配置的工具，以及调用次数、失败次数和中位持续时间。选择一个工具可查看其调用，并跳转到对话记录中的某次调用。                                                                                                                                                                                                                                                                                                      |
| **资源**  | 挂载在其容器路径上的[文件](https://platform.claude.com/docs/zh-CN/managed-agents/files)、[代码仓库](https://platform.claude.com/docs/zh-CN/managed-agents/github)和[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)，包括每个存储中的记忆以及本会话对它们所做的更改。还会列出智能体写入 `/mnt/session/outputs` 的文件，以及附加到会话智能体的[技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills)。 |
| **讨论串** | 每个线程及其状态、上下文大小和成本。选择一个线程可查看其详细信息，例如智能体、模型、上下文用量和成本。                                                                                                                                                                                                                                                                                                               |

在会话 URL 后追加 `?event={event_id}` 可在特定事件处打开该会话。

通过 `ant beta:sessions connect`，您可以从 `ant` CLI 打开同一个查看器，或在终端中跟踪会话。请参阅[从终端连接到 Managed Agents 会话](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/sessions-connect)。

## 跟踪用量

会话对象包含一个 `usage` 字段，其中记录了该会话的累计用量：令牌计数、服务器工具使用、活跃时间以及跟踪的标价成本。在会话进入空闲状态后获取该会话，即可读取最新的总计数据。

```json
{
  "id": "sesn_01...",
  "status": "idle",
  "usage": {
    "input_tokens": 5000,
    "output_tokens": 3200,
    "cache_read_input_tokens": 20000,
    "cache_creation": {
      "ephemeral_5m_input_tokens": 2000,
      "ephemeral_1h_input_tokens": 0
    },
    "list_cost": {
      "amount": "187",
      "currency": "USD"
    },
    "active_seconds": 342.5,
    "server_tool_use": {
      "web_search_requests": 3,
      "web_fetch_requests": 0
    }
  }
}
```

| 字段                        | 描述                                                                                      |
| ------------------------- | --------------------------------------------------------------------------------------- |
| `input_tokens`            | 会话中所有模型调用的未缓存输入令牌数。                                                                     |
| `output_tokens`           | 会话中所有模型调用的输出令牌总数。                                                                       |
| `cache_read_input_tokens` | 从提示缓存中读取的令牌数。                                                                           |
| `cache_creation`          | 缓存创建令牌数，按缓存生命周期细分（`ephemeral_5m_input_tokens` 和 `ephemeral_1h_input_tokens`）。           |
| `list_cost`               | 按公开标价计算的会话累计消耗，以字符串形式表示的整数美分数，并附带货币代码。                                                  |
| `active_seconds`          | 会话中至少有一个线程在运行的累计时间。并发线程的重叠活动只计算一次。会话的运行时成本按此时长计价。                                       |
| `server_tool_use`         | 服务器执行的工具请求计数，用于计价。网络搜索请求按每次请求计入标价成本。网页获取请求不收取按次费用，也不计量，因此 `web_fetch_requests` 显示为 `0`。 |

缓存条目默认使用 5 分钟的 “TTL”（生存时间），因此在该时间窗口内连续进行的轮次可以受益于缓存读取，从而降低每令牌成本。

会话的 `stats` 对象有其自己的 `active_seconds`，它将每个线程各自的活跃时间相加，而不是将重叠活动只计算一次。

### 每线程用量

每个[会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)自身的 `usage` (csharp, go: `Usage`) 同样包含 `list_cost` (java, php: `listCost`; csharp, go: `ListCost`) 和 `active_seconds` (java, php: `activeSeconds`; csharp, go: `ActiveSeconds`)。每个线程的数值是独立四舍五入的，并且不包含会话的运行时间成本，因此它们的总和与会话的 `list_cost` (java, php: `listCost`; csharp, go: `ListCost`) 并不完全相等。以会话的数值为准。

### 从流中读取用量

您无需轮询会话即可观察这些总计。`session.usage` 事件在会话流和事件历史中携带相同的累计快照。该快照包含 `usage` (csharp, go: `Usage`) 对象，如果会话设置了预算，还包含会话的 `budget` (csharp, go: `Budget`)。

该事件在空闲状态转换时发出，而不是按定时器发出：

* 会话在进入空闲状态之前会立即发出一个该事件，无论停止原因是什么。
* 当某个线程因达到[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)而暂停时，会话会发出一个该事件。

### 强制执行支出限制

要强制执行支出限制，请设置会话预算，而不是轮询用量并自行停止会话。平台会持续对会话的消耗进行计价，一旦标价成本达到上限，就会在每个线程的下一次模型请求之前将其暂停。请参阅[当会话达到其预算时](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#when-a-session-reaches-its-budget)，了解这在流中的表现。

### 查找整个组织的会话用量

会话用量会计入 Claude Console 中[用量](https://platform.claude.com/usage)页面（`platform.claude.com/usage`）和[费用](https://platform.claude.com/cost)页面（`platform.claude.com/cost`）上的组织总计。要按智能体查看令牌用量，请在**用量**页面上将**分组方式**设置为**智能体**。

要按智能体或按会话查看会话用量，请使用[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)，并按 `agent_id` 或 `session_id` 进行分组。会话用量还会出现在旧版[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api) 的[用量报告](https://platform.claude.com/docs/zh-CN/api/beta/organization/usage_report/retrieve_messages)中。要查看它记录在哪个账户下，请按以下字段之一对该报告进行分组：

* **`account_id`：** 启动会话的成员。这涵盖在 Console 中启动的会话，或使用限定于单个工作区的[个人密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)启动的会话。
* **`service_account_id`：** 一个由 Anthropic 管理的服务账户（`svac_...`）。这涵盖使用工作区密钥启动的会话或由[部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)启动的会话。对于此类用量，`api_key_id` 和 `account_id` 为 `null`。

Anthropic 会在每个运行会话的工作区中创建一个此类服务账户。它不会与您创建的服务账户一起列出，Admin API 的服务账户端点也不会返回它。在**用量**页面上，**账户**筛选器按其 ID 列出它。当您将**分组方式**设置为 **API 密钥**时，其用量标记为**托管智能体**。

要查看是谁启动了会话，请查看[会话](https://platform.claude.com/sessions)页面上的**创建者**列。

## 调试技巧

* **检查会话事件：** 会话通过 [`session.error`](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types) 事件报告错误。
* **查看工具结果：** 工具失败通常可以解释智能体的意外行为。检查器的**工具**选项卡会显示每个工具的失败情况。
* **使用系统提示：** 在系统提示中添加日志记录指令，让智能体总结它做了什么以及发现了什么。
* **预览故障排除：** 如果选择接收事件增量的流未按预期运行，请参阅[预览故障排除](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#troubleshoot-previews)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    发送事件、流式传输响应，并在会话运行时中断或重定向会话。
  </Card>

  <Card title="会话预算" icon="coins" href="https://platform.claude.com/docs/zh-CN/managed-agents/budgets">
    通过按公开标价执行的硬性美元预算来限制会话的支出。
  </Card>
</CardGroup>

## 可用性

会话查看器在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                         | HIPAA                             |
| ---- | ------------------------------------------------------------- | --------------------------------- |
| 状态   | 不具备资格                                                         | 不具备资格                             |
| 详细信息 | 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。 | 适用于所有 Managed Agents 子功能，包括自托管沙箱。 |

适用的端点：

* `/v1/agents`
* `/v1/sessions`
* `/v1/environments`
