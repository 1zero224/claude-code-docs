# 为 Office 代理配置自定义 OpenTelemetry 收集器

您可以将来自 Office 代理的完整审计遥测数据路由到您自己的 OpenTelemetry (OTEL) 收集器。这使您的组织能够完全控制数据保留、加密以及与 SIEM 或可观测性平台的集成。

本指南涵盖如何启用自定义收集器、您将接收的数据以及完整的跨度架构参考。

自定义 OTEL 收集器可供 Claude Enterprise 组织和直接提供商部署（Amazon Bedrock、Google Vertex AI 或网关）使用。

## 您将接收的内容

配置自定义收集器后，Office 代理会发送涵盖每个用户轮次的跟踪数据。每个轮次生成一个跨度树，捕获提示、模型调用、工具执行、文件上传和上下文压缩事件。

您的收集器接收所有跨度属性，包括那些包含用户生成内容的属性（提示文本、工具输入和输出、文档 URL 和文件名）。不会对任何属性进行编辑或过滤。助手响应文本不包含在发出的跨度数据中。您的组织拥有此数据。

**重要：**指标不会发送到自定义收集器。`office_agent.*` 计数器命名空间仅路由到 Anthropic。但是，每个计数器增量也会作为跨度事件出现在活跃跨度上，因此相同的信号在您的跟踪中可用。

遥测数据通过 OTLP/HTTP 发送到您的端点 `{your_url}/v1/traces`。不支持 gRPC，因为加载项在 Office WebView 中运行。

---

## 启用自定义收集器

设置因您的组织如何与 Claude 进行身份验证而异。

**重要：**配置自定义端点后，遥测数据将专门发送到您的收集器。跨度不会同时发送到 Anthropic。

### Claude Enterprise (OAuth) 组织

组织管理员在 Claude.ai 管理控制台的 **[组织设置 > Office 代理](https://claude.ai/admin-settings/office-agents)** 中设置收集器端点。该设置适用于整个组织。

| **设置**          | **描述**                                                   |
| --------------- | -------------------------------------------------------- |
| `otlp_endpoint` | 您的 OTLP 收集器的基础 URL。加载项会附加 /v1/traces。强烈建议使用 HTTPS。       |
| `otlp_headers`  | 可选的身份验证标头，按照 OpenTelemetry 规范格式化：key1=value1,key2=value2 |

协议必须是基于 HTTP 的 OTLP。gRPC 在配置时会被拒绝。

### 直接提供商部署（Amazon Bedrock、Google Vertex AI、网关）

对于针对您自己的模型提供商而非 Claude.ai 进行身份验证的部署，收集器端点通过三个配置通道之一提供。所有三个通道使用相同的两个密钥。

**推荐：**使用 **[claude-for-msft-365-install 插件](https://github.com/anthropics/financial-services/tree/main/claude-for-msft-365-install)**用于 Claude Code。它会引导您生成清单、注册 Entra 扩展属性以及建立预配置了 `otlp_endpoint` 和 `otlp_headers` 的引导端点。下面记录了三个通道供参考和手动设置。

| **密钥**          | **格式**                  | **描述**                                                            |
| --------------- | ----------------------- | ----------------------------------------------------------------- |
| `otlp_endpoint` | HTTPS URL               | 您的 OTLP 收集器的基础 URL。加载项会删除任何尾部斜杠并附加 /v1/traces。                    |
| `otlp_headers`  | key1=value1,key2=value2 | 可选的身份验证标头。格式与 OpenTelemetry OTEL\_EXPORTER\_OTLP\_HEADERS 环境变量相同。 |

如果 `otlp_endpoint` 未设置或为空，则不会配置自定义收集器，加载项会回退到默认行为。

**注意：**这些配置通道仅适用于 Microsoft Office 部署。Google Workspace 加载项的配置方式不同。

**通道 1：清单 URL 参数**

将密钥作为查询字符串参数附加到自定义加载项清单中的任务窗格 URL：

`https://<addin-host>/taskpane.html?otlp_endpoint=https://otel-collector.your-domain.com&otlp_headers=Authorization=Bearer%20<token>`

对值进行 URL 编码。这会将配置应用于安装清单的每个用户。

**通道 2：Azure Entra ID 目录扩展**

对于按用户配置，将密钥注册为 Entra ID 目录扩展属性，并通过 Microsoft Graph 分配它们。加载项使用嵌套应用身份验证 (NAA) 从用户的 ID 令牌中读取它们。

已颁发 ID 令牌中的声明名称遵循 Azure 的目录扩展格式：

| **声明**               | **映射到**         |
| -------------------- | --------------- |
| `extn.otlp_endpoint` | `otlp_endpoint` |
| `extn.otlp_headers`  | `otlp_headers`  |

使用针对用户对象的 Graph PATCH 按用户设置这些。Azure 在 ID 令牌中将目录扩展值编码为单元素数组；加载项会自动解包它们。此通道需要清单 URL 参数中的 `entra_sso=1` 来启用 NAA 令牌获取。

**通道 3：引导端点响应**

如果您的部署使用引导端点（您的组织托管的 JSON 端点，加载项在启动时调用），请在响应正文中包含这些密钥：

```
{
  "otlp_endpoint": "https://otel-collector.your-domain.com",
  "otlp_headers": "Authorization=Bearer <token>"
}
```

引导端点 URL 本身通过清单 URL 参数或 Entra `extn.bootstrap_url` 声明中的 `bootstrap_url` 配置。如果获取了 Entra ID 令牌，它会作为 Bearer 授权标头传递到引导端点，以便您的端点可以在返回按用户配置之前对用户进行身份验证。

**优先级**

当多个通道提供值时，后面的通道会覆盖前面的通道：首先读取清单参数，然后是 Entra 声明，然后是引导响应。引导响应优先。

如果您还没有，最快的方式是 **[claude-for-msft-365-install 插件](https://github.com/anthropics/financial-services/tree/main/claude-for-msft-365-install)**。

---

## 部署模式

自定义收集器导出在两种部署模式上都可用：

- Claude.ai 企业版 (OAuth)：完整的审计跟踪，包括用户身份 (`user.email`、`user.account_uuid`、`organization.id`)、MCP 服务器元数据和文件上传记录。

- 直接提供商 (Bedrock、Vertex AI、网关)：核心审计跟踪（提示、工具输入和输出、文档 URL），但没有用户身份、没有 MCP 元数据和没有文件上传跨度。用户归属需要将 `session.id` 与您自己的身份提供商日志进行关联。

核心审计有效负载在两种模式中是相同的。直接提供商部署缺少 Claude.ai 帐户上下文，因此从 Claude.ai 用户或组织配置文件派生的属性不存在。请参阅下面跨度架构中的 `[claude.ai-only]` 标签以获取完整列表。

## 表面和供应商标签

每个跨度都包含两个标签，用于标识生成它的 Office 应用程序和平台。将这些用作过滤和仪表板的主要维度。

| **标签**          | **值**                                                      |
| --------------- | ---------------------------------------------------------- |
| `agent.surface` | sheet (Excel)、doc (Word)、slide (PowerPoint)、mail (Outlook) |
| `agent.vendor`  | m (Microsoft)                                              |

---

## 跨度参考

每个用户轮次产生最多五种跨度类型的父/子树。标记为 [content] 的属性携带用户生成的数据；这些形成您的审计有效负载。标记为 [`claude.ai-only`] 的属性仅在用户使用 Claude 帐户登录时填充；它们在 Bedrock、Vertex AI 和网关部署上不存在。缺少的属性从跨度中完全省略（没有具有空值的密钥）。

`file.upload` 跨度和所有 `mcp.*` 属性也是 [`claude.ai-only`]，因为文件上传和 MCP 服务器连接是 Claude 平台功能。

对于直接提供商部署，用户身份应通过 `session.id` 和 `document.url` 进行关联，并与您的身份提供商的会话日志进行联接。

### 资源属性

这些属性出现在每个跨度上：

| **属性**            | **描述**           |
| ----------------- | ---------------- |
| `service.name`    | 固定值：office-agent |
| `service.version` | 固定值：1.0.0        |
| `git.sha`         | 构建提交标识符          |

### agent.query (根跨度)

每个用户轮次一个跨度。这是跨度树的根，携带会话身份、文档上下文和 MCP 服务器状态。SpanKind：INTERNAL。

| **属性**                                        | **描述**                                                    |
| --------------------------------------------- | --------------------------------------------------------- |
| `agent.surface`                               | sheet \| doc \| slide \| mail                             |
| `agent.vendor`                                | m                                                         |
| `user.message [content]`                      | 用户的提示（前 4000 个字符）                                         |
| `session.id`                                  | 不透明的会话标识符                                                 |
| `user.email [claude.ai-only]`                 | 用户的电子邮件地址                                                 |
| `user.bucket [claude.ai-only]`                | 确定性哈希桶（电子邮件的 SHA-256，模 30）                                |
| `user.account_uuid [claude.ai-only]`          | Claude 账户 UUID                                            |
| `document.url [content]`                      | 打开的 Office 文档的 URL                                        |
| `organization.id [claude.ai-only]`            | Claude 组织 UUID                                            |
| `org.rate_limit_tier [claude.ai-only]`        | Claude 订阅等级                                               |
| `org.type [claude.ai-only]`                   | Claude 组织类型                                               |
| `agent.selected_model`                        | 用户为此会话选择的模型                                               |
| `office.platform`                             | PC \| Mac \| OfficeOnline \| iOS \| Android \| Universal  |
| `office.version`                              | Office 内部版本号                                              |
| `mcp.configured_count [claude.ai-only]`       | 已配置的 MCP 服务器数量                                            |
| `mcp.connected_count [claude.ai-only]`        | 成功连接的 MCP 服务器数量                                           |
| `mcp.failed_count [claude.ai-only]`           | 连接失败的 MCP 服务器数量                                           |
| `mcp.fetch_status [claude.ai-only]`           | success \| error \| timeout \| no\_auth \| not\_attempted |
| `mcp.fetch_duration_ms [claude.ai-only]`      | MCP 注册表获取耗时（毫秒）                                           |
| `mcp.fetch_http_status [claude.ai-only]`      | MCP 注册表获取 HTTP 状态码                                        |
| `mcp.servers [content] [claude.ai-only]`      | 序列化的 MCP 服务器详情（名称、工具数量、错误消息）                              |
| `file.upload.count [claude.ai-only]`          | 附加到此轮次的文件数量                                               |
| `file.upload.total_bytes [claude.ai-only]`    | 上传的总字节数                                                   |
| `file.upload.image_count [claude.ai-only]`    | 图像附件数量                                                    |
| `file.upload.document_count [claude.ai-only]` | 文档附件数量                                                    |
| `file.upload.other_count [claude.ai-only]`    | 其他附件数量                                                    |
| `error.name`                                  | 异常类名称（失败时显示）                                              |
| `agent.query_phase`                           | 查询失败的阶段（失败时显示）                                            |

### agent.stream

每次 Claude API 调用对应一个 span，作为查询 span 的子级。SpanKind：CLIENT。

| **属性**                  | **描述**                                     |
| ----------------------- | ------------------------------------------ |
| `model`                 | 用于此调用的模型 ID                                |
| `max_tokens`            | 请求的最大输出令牌数                                 |
| `agent.message_count`   | 流开始时对话中的消息数                                |
| `input_tokens`          | 计费的输入令牌（来自 API 响应）                         |
| `output_tokens`         | 计费的输出令牌（来自 API 响应）                         |
| `cache_read_tokens`     | 从提示缓存提供的令牌                                 |
| `cache_creation_tokens` | 写入提示缓存的令牌                                  |
| `stop_reason`           | end\_turn \| tool\_use \| max\_tokens \| 等 |
| `request_id`            | Anthropic API request-id 标头，可用于支持关联        |

**关于提示缓存的说明：**该加载项无条件请求提示缓存。`cache_read_tokens` 和 `cache_creation_tokens` 属性从提供商的 API 响应设置，当响应不包含这些属性时将被省略。提示缓存适用于 Claude Platform；截至本文撰写时，Amazon Bedrock 和 Google Vertex AI 还不会通过该加载项使用的客户端返回这些字段。当您的提供商支持此功能时，这些属性将自动开始显示。

### agent.tool_execution

每个工具调用一个跨度，作为流跨度的子级。SpanKind：INTERNAL。这是模型对文档采取的操作的主要记录。

| **属性**                  | **描述**                                                           |
| ----------------------- | ---------------------------------------------------------------- |
| `tool_name`             | 工具标识符（例如 get\_cell\_ranges、execute\_office\_js、edit\_slide\_xml） |
| `tool.id`               | 此工具调用的唯一 ID                                                      |
| `tool.caller`           | server \| client                                                 |
| `tool.owner`            | first\_party \| mcp \| server                                    |
| `tool.read_write`       | read \| write \| read\_write                                     |
| `tool.accept_decision`  | manual \| auto\_accept \| deferred                               |
| `tool.input [content]`  | 序列化的工具输入（前 4000 个字符）                                             |
| `tool.success`          | 布尔值                                                              |
| `tool.output [content]` | 序列化的工具输出（前 4000 个字符）                                             |
| `tool.output_chars`     | 完整输出长度（以字符为单位）（用于检测截断）                                           |
| `tool.error_type`       | 错误分类（失败时显示）                                                      |
| `sheet.cells_read`      | 读取的单元格（仅限工作表表面）                                                  |
| `sheet.cells_written`   | 写入的单元格（仅限工作表表面）                                                  |
| `sheet.cells_copied`    | 复制的单元格（仅限工作表表面）                                                  |

**说明：**`tool.accept_decision` 属性记录权限决策的方式：`manual`（用户批准了此特定操作）、`auto_accept`（用户之前授予了常设批准）或 `deferred`（操作已排队等待稍后审查）。使用此功能审计整个组织的批准模式。

### agent.compaction

每个对话一个跨度的自动摘要，当上下文接近窗口限制时触发。SpanKind：CLIENT。

| **属性**                    | **描述**   |
| ------------------------- | -------- |
| `compaction.pre_tokens`   | 摘要前的令牌计数 |
| `compaction.post_tokens`  | 摘要后的令牌计数 |
| `compaction.tokens_saved` | 差值       |
| `compaction.success`      | 布尔值      |
| `compaction.trigger`      | 目前始终为反应式 |

此跨度还包含 `agent.surface`、`agent.vendor`、`session.id`、`user.email [claude.ai-only]`、`user.bucket [claude.ai-only]`、`office.platform` 和 `office.version`，从根跨度复制，以便您可以独立查询压缩事件。

### file.upload [claude.ai-only]

每个文件上传一个跨度，作为查询跨度的子级。SpanKind：CLIENT。此跨度类型仅在用户使用 Claude.ai 账户登录时出现。文件上传在直接提供商部署中不可用。

| **属性**                           | **描述**                  |
| -------------------------------- | ----------------------- |
| `file.upload.filename [content]` | 原始文件名                   |
| `file.upload.size_bytes`         | 文件大小                    |
| `file.upload.mime_type`          | MIME 类型                 |
| `file.upload.file_id`            | Anthropic Files API 标识符 |
| `file.upload.success`            | 布尔值                     |

---

## 跨度事件

跨度事件是附加到上述跨度的带时间戳的标记。它们捕获生命周期转换和计数器等效信号。

- `agent.query`：exception {exception.type}；file_upload {file_id, mime_type, content_category}

- `agent.stream`：first_token；stream_complete；stream_error {exception.type}

- `agent.tool_execution`：tool_init；tool_run；tool_result；tool_error {error_type}

- `agent.compaction`：compaction_start；compaction_complete；compaction_error {exception.type}

- `file.upload`：exception {exception.type}

每个内部产品计数器也在当前活跃的跨度上记录一个同名的跨度事件，在您的跟踪数据中提供等效的指标流。`office_agent.token.usage` 事件在每个 `agent.stream` 跨度上发出，每个非零令牌类型一次，属性为 {token_usage.type: input | output | cacheRead | cacheCreation, token_usage.model, token_usage.token_count}。这反映了其他 Anthropic 产品发出的 `*.token.usage` 计数器形状，因此单个收集器可以通过按 `service.name` 分组来聚合跨产品的令牌成本。

---

## 特定于表面的行为

遥测架构在所有表面上是一致的。这些是差异：

- Sheets (Excel)：`agent.tool_execution` 跨度包含 `sheet.cells_read`、`sheet.cells_written` 和 `sheet.cells_copied` 属性。当用户在单元格编辑中间而工具尝试写入时，会出现 `office_agent.cell_edit_collision_total` 跨度事件。

- Documents (Word)：文档编辑漏斗事件跟踪编辑生命周期：`office_agent.doc_edit_received_total`、`office_agent.doc_edit_parsed_total`、`office_agent.doc_edit_applied_total`、`office_agent.doc_proposed_edit_reviewed_total`。没有 `sheet.cells_*` 属性。

- Slides (PowerPoint)：除了通用架构外，没有特定于表面的属性或事件。

- Mail (Outlook)：除了通用架构外，没有特定于表面的属性或事件。

---

## 重建用户会话

### Claude.ai 部署

- 按 `user.email`（或 `user.account_uuid`）和 `session.id` 过滤跨度。

- 按开始时间戳排序 `agent.query` 跨度；每个都是一个用户轮次。

- 对于每个轮次，`user.message` 是提示，`document.url` 是正在处理的文件。

- 子 `agent.tool_execution` 跨度按时间戳排序，是采取的操作：`tool.input` 是尝试的内容，`tool.output` 是结果，`tool.accept_decision` 记录用户是否明确批准。

### 直接提供商部署

在此模式下，加载项没有 Claude.ai 用户身份，因此跨度不包含 `user.email` 或 `user.account_uuid`。要将活动归属于用户：

- 按 `session.id` 筛选跨度以隔离一个连续的加载项会话。

- 使用 `document.url` 识别正在处理的文件。

- 将会话与您的身份提供商的日志相关联：Entra 登录事件、网关访问日志或您的引导端点的请求日志（该日志在 Bearer 标头中接收用户的 Entra ID 令牌）。

- 将会话分配给用户后，每轮重建是相同的：按时间戳排序的 agent.query 跨度，其中 `user.message`、`tool.input`、`tool.output` 和 `tool.accept_decision` 提供审计跟踪。

这在两种部署模式中都生成了完整的、有序的交互记录。