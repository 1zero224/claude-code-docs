---
title: Ruby SDK
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/ruby
description: 安装和配置 Anthropic Ruby SDK，包括 Sorbet 类型、流式传输辅助工具和连接池
---

Anthropic Ruby 库为任何 Ruby 3.2.0+ 应用程序提供了便捷访问 Claude API 的方式。它附带了 YARD、RBS 和 RBI 格式的完整类型和文档字符串。该库使用标准库的 `net/http` 作为 HTTP 传输层，并通过 `connection_pool` gem 实现 “connection pooling”（连接池）。

<Info>
  有关包含代码示例的 API 功能文档，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)。本页介绍 Ruby 特有的 SDK 功能和配置。
</Info>

## 安装

使用 Bundler 将该 gem 添加到您应用程序的 `Gemfile` 中：

```bash
bundle add anthropic
```

## 要求

Ruby 3.2.0 或更高版本。

## 用法

```ruby
anthropic = Anthropic::Client.new(
  api_key: ENV["ANTHROPIC_API_KEY"] # This is the default and can be omitted
)

message = anthropic.messages.create(
  max_tokens: 1024,
  messages: [{role: "user", content: "Hello, Claude"}],
  model: Anthropic::Model::CLAUDE_OPUS_5_5
)

message.content.each do |block|
  puts block.text if block.type == :text
end
```

有关包括 Workload Identity Federation（工作负载身份联合）在内的身份验证选项，请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。如果您的 API 密钥是未限定到某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)，请在 `anthropic-workspace-id` 请求标头中设置工作区 ID；[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)介绍了此 SDK 的按请求设置选项。

## 流式传输

SDK 支持使用 “Server-Sent Events”（服务器发送事件），即 SSE，进行 “streaming”（流式传输）响应。

```ruby
anthropic = Anthropic::Client.new
stream = anthropic.messages.stream(
  max_tokens: 1024,
  messages: [{role: "user", content: "Hello, Claude"}],
  model: Anthropic::Model::CLAUDE_OPUS_5_5
)

stream.each do |message|
  puts(message.type)
end
```

### 流式传输辅助工具

该库为流式传输消息提供了多种便捷功能，例如：

```ruby
anthropic = Anthropic::Client.new
stream = anthropic.messages.stream(
  max_tokens: 1024,
  messages: [{role: :user, content: "Say hello there!"}],
  model: Anthropic::Model::CLAUDE_OPUS_5_5
)

stream.text.each do |text|
  print(text)
end
```

使用 `anthropic.messages.stream(...)` 进行流式传输时，会提供多种辅助工具，包括累积功能和 SDK 特有的事件。

## 输入模式和工具调用

SDK 提供了辅助机制，用于为工具定义结构化数据类，并让 Claude 自动运行它们。有关工具使用模式（包括工具运行器）的详细文档，请参阅[工具运行器（SDK）](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)。

```ruby
anthropic = Anthropic::Client.new
class CalculatorInput < Anthropic::BaseModel
  required :lhs, Float
  required :rhs, Float
  required :operator, Anthropic::InputSchema::EnumOf[:+, :-, :*, :/]
end

class Calculator < Anthropic::BaseTool
  input_schema CalculatorInput

  def call(expr)
    expr.lhs.public_send(expr.operator, expr.rhs)
  end
end

# 自动处理工具执行循环
anthropic.beta.messages.tool_runner(
  model: Anthropic::Model::CLAUDE_OPUS_5_5,
  max_tokens: 1024,
  messages: [{role: "user", content: "What's 15 * 7?"}],
  tools: [Calculator.new]
).each_message { |message| puts message.content }
```

## 结构化输出

有关包含 Ruby 示例的完整结构化输出文档，请参阅[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。

## 错误处理

当库无法连接到 API，或者 API 返回非成功状态码（即 4xx 或 5xx 响应）时，会抛出 `Anthropic::Errors::APIError` 的子类：

```ruby
anthropic = Anthropic::Client.new
begin
  message = anthropic.messages.create(
    max_tokens: 1024,
    messages: [{role: "user", content: "Hello, Claude"}],
    model: Anthropic::Model::CLAUDE_OPUS_5_5
  )
rescue Anthropic::Errors::APIConnectionError => e
  puts("The server could not be reached")
  puts(e.cause)  # an underlying Exception, likely raised within `net/http`
rescue Anthropic::Errors::RateLimitError => e
  puts("A 429 status code was received; we should back off a bit.")
rescue Anthropic::Errors::APIStatusError => e
  puts("Another non-200-range status code was received")
  puts(e.status)
end
```

错误代码如下：

| 原因          | 错误类型                       |
| ----------- | -------------------------- |
| HTTP 400    | `BadRequestError`          |
| HTTP 401    | `AuthenticationError`      |
| HTTP 403    | `PermissionDeniedError`    |
| HTTP 404    | `NotFoundError`            |
| HTTP 409    | `ConflictError`            |
| HTTP 422    | `UnprocessableEntityError` |
| HTTP 429    | `RateLimitError`           |
| HTTP >= 500 | `InternalServerError`      |
| 其他 HTTP 错误  | `APIStatusError`           |
| 超时          | `APITimeoutError`          |
| 网络错误        | `APIConnectionError`       |

## 重试

默认情况下，某些错误会自动重试 2 次，并采用短暂的指数退避。

连接错误（例如由于网络连接问题）、408 Request Timeout、409 Conflict、429 Rate Limit、>=500 内部错误以及超时，默认都会重试。

您可以使用 `max_retries` 选项来配置或禁用此功能：

```ruby
# 为所有请求配置默认值：
anthropic = Anthropic::Client.new(
  max_retries: 0 # default is 2
)

# 或者，按请求单独配置：
anthropic.messages.create(
  max_tokens: 1024,
  messages: [{role: "user", content: "Hello, Claude"}],
  model: :"claude-opus-5-5",
  request_options: {max_retries: 5}
)
```

## 超时

默认情况下，请求在 10 分钟后超时。您可以使用 `timeout` 选项进行配置：

```ruby
# 为所有请求配置默认值：
anthropic = Anthropic::Client.new(
  timeout: 20 # 20 seconds (default is 10 minutes)
)

# 或者，按请求单独配置：
anthropic.messages.create(
  max_tokens: 1024,
  messages: [{role: "user", content: "Hello, Claude"}],
  model: :"claude-opus-5-5",
  request_options: {timeout: 5}
)
```

超时时会抛出 `Anthropic::Errors::APITimeoutError`。

请注意，超时的请求默认会重试。

## 分页

Claude API 中的列表方法是分页的。

该库为每个列表响应提供自动分页迭代器，因此您无需手动请求后续页面：

```ruby
anthropic = Anthropic::Client.new
page = anthropic.messages.batches.list(limit: 20)

# 从页面中获取单个条目。
batch = page.data[0]
puts(batch.id)

# 根据需要自动获取更多页面。
page.auto_paging_each do |batch|
  puts(batch.id)
end
```

或者，您可以使用 `#next_page?` 和 `#next_page` 方法对页面进行更精细的控制。

```ruby
anthropic = Anthropic::Client.new
page = anthropic.messages.batches.list(limit: 20)
loop do
  page.data&.each { |batch| puts(batch.id) }
  break unless page.next_page?
  page = page.next_page
end
```

## 文件上传

与文件上传对应的请求参数可以作为原始内容、[`Pathname`](https://rubyapi.org/3.2/o/pathname) 实例、[`StringIO`](https://rubyapi.org/3.2/o/stringio) 等形式传递。

```ruby
anthropic = Anthropic::Client.new
require "pathname"

# 使用 `Pathname` 来发送文件名、避免将大文件载入内存，或两者兼顾：
file_metadata = anthropic.files.upload(file: Pathname("/path/to/file"))

# 或者，直接传入文件内容或 `StringIO`：
file_metadata = anthropic.files.upload(file: File.read("/path/to/file"))

# 或者，如需控制文件名、内容类型或两者：
file = Anthropic::FilePart.new(File.read("/path/to/file"), filename: "/path/to/file", content_type: "...")
file_metadata = anthropic.files.upload(file: file)

puts(file_metadata.id)
```

请注意，您也可以传递原始 `IO` 描述符，但这会禁用重试，因为库无法确定该描述符是文件还是管道（管道无法回退）。

## Sorbet

该库提供了完整的 [RBI](https://sorbet.org/docs/rbi) 定义，并且不依赖 sorbet-runtime。

您可以按如下方式提供类型安全的请求参数：

```ruby
anthropic = Anthropic::Client.new
anthropic.messages.create(
  max_tokens: 1024,
  messages: [Anthropic::MessageParam.new(role: "user", content: "Hello, Claude")],
  model: Anthropic::Model::CLAUDE_OPUS_5_5
)
```

或者，等效地：

```ruby
anthropic = Anthropic::Client.new
# 哈希可以使用，但不是类型安全的：
anthropic.messages.create(
  max_tokens: 1024,
  messages: [{role: "user", content: "Hello, Claude"}],
  model: Anthropic::Model::CLAUDE_OPUS_5_5
)

# 您也可以展开（splat）一个完整的 Params 类：
params = Anthropic::MessageCreateParams.new(
  max_tokens: 1024,
  messages: [Anthropic::MessageParam.new(role: "user", content: "Hello, Claude")],
  model: Anthropic::Model::CLAUDE_OPUS_5_5
)
anthropic.messages.create(**params)
```

### 枚举

由于该库不依赖 `sorbet-runtime`，因此无法提供 [`T::Enum`](https://sorbet.org/docs/tenum) 实例。取而代之的是，SDK 提供了"tagged symbols"（标记符号），它们在运行时始终是原始类型：

```ruby
# :auto
puts(Anthropic::MessageCreateParams::ServiceTier::AUTO)

# 揭示的类型：`T.all(Anthropic::MessageCreateParams::ServiceTier, Symbol)`
T.reveal_type(Anthropic::MessageCreateParams::ServiceTier::AUTO)
```

枚举参数具有"宽松"类型，因此您既可以传入枚举常量，也可以传入其字面值：

```ruby
anthropic = Anthropic::Client.new
# 使用枚举常量可保留带标签的类型信息：
anthropic.messages.create(
  max_tokens: 1024,
  messages: [{role: "user", content: "Hello, Claude"}],
  model: Anthropic::Model::CLAUDE_OPUS_5_5,
  service_tier: Anthropic::MessageCreateParams::ServiceTier::AUTO
)

# 也可以使用字面量值：
anthropic.messages.create(
  max_tokens: 1024,
  messages: [{role: "user", content: "Hello, Claude"}],
  model: :"claude-opus-5-5",
  service_tier: :auto
)
```

## BaseModel

所有参数和响应对象都继承自 `Anthropic::Internal::Type::BaseModel`，它提供了多种便捷功能，包括：

1. 所有字段（包括未知字段）都可以通过 `obj[:prop]` 语法访问，并且可以使用 `obj => {prop: prop}` 或模式匹配语法进行解构。

2. 基于结构等价性判断相等。如果两个 API 调用返回相同的值，使用 == 比较响应将返回 true。

3. 实例和类本身都可以进行美化打印。

4. 辅助方法，例如 `#to_h`、`#deep_to_h`、`#to_json` 和 `#to_yaml`。

## 并发和连接池

`Anthropic::Client` 实例是线程安全的，但只有在没有进行中的 HTTP 请求时才是 fork 安全的。

每个 `Anthropic::Client` 实例都有自己的 HTTP 连接池，默认大小为 99。因此，在大多数情况下，建议每个应用程序只创建一次客户端。

当连接池中的所有可用连接都被占用时，请求会等待新连接变为可用，排队时间计入请求超时。

除非另有说明，SDK 中的其他类没有用于保护其底层数据结构的锁。

## OpenTelemetry 追踪

当您的应用程序注册 OpenTelemetry tracer provider 时，SDK 会为其发送的每个请求发出一个 span。要将这些 span 发送到您的可观测性平台，请参阅[在 Anthropic SDK 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

## 发送自定义或未记录的请求

### 未记录的属性

您可以向任何端点发送未记录的参数，并读取未记录的响应属性，方法如下：

<Warning>
  同名的 `extra_` 参数会覆盖已记录的参数。出于安全原因，请仅对受信任的输入数据使用这些方法。
</Warning>

```ruby
anthropic = Anthropic::Client.new
value = "example"
message =
  anthropic.messages.create(
    max_tokens: 1024,
    messages: [{role: "user", content: "Hello, Claude"}],
    model: :"claude-opus-5-5",
    request_options: {
      extra_query: {my_query_parameter: value},
      extra_body: {my_body_parameter: value},
      extra_headers: {"my-header": value}
    }
  )

puts(message[:my_undocumented_property])
```

### 未记录的请求参数

如果您想显式发送额外参数，可以在发出请求时在 `request_options:` 下使用 `extra_query`、`extra_body` 和 `extra_headers`，如前面的示例所示。

### 未记录的端点

要向未记录的端点发出请求，同时保留身份验证、重试等功能的优势，您可以使用 `anthropic.request` 发出请求，方法如下：

```ruby
response = anthropic.request(
  method: :post,
  path: '/undocumented/endpoint',
  query: {"dog": "woof"},
  headers: {"useful-header": "interesting-value"},
  body: {"hello": "world"}
)
```

## 平台集成

<Note>
  有关包含代码示例的详细平台设置指南，请参阅：

  * [Amazon Bedrock（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)
  * [Amazon Bedrock（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)
  * [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)
  * [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)
  * [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)

  另请参阅 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)。
</Note>

Ruby SDK 支持以下平台：

* **Claude Models on Google Cloud (Google managed)：** `Anthropic::VertexClient`。需要 `googleauth` gem。
* **Bedrock：** `Anthropic::BedrockMantleClient`，或用于 `bedrock-runtime` 路径的 `Anthropic::BedrockClient`。`Anthropic::BedrockMantleClient` 需要 `aws-sdk-core` gem；`Anthropic::BedrockClient` 需要 `aws-sdk-bedrockruntime` gem。
* **Claude Platform on AWS：** 包含在主 `anthropic` gem 中（需要 `aws-sdk-core` gem）。提供 `Anthropic::AWSClient`。向构造函数传递 `workspace_id:` 或设置 `ANTHROPIC_AWS_WORKSPACE_ID` 环境变量（请参阅[工作区](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#workspaces)）。以 beta 形式提供。
* **Foundry：** 包含在主 `anthropic` gem 中。提供 `Anthropic::FoundryClient`。向构造函数传递 `resource:` 和 `api_key:`（对于 Microsoft Entra ID，则传递 `azure_ad_token_provider:`），或设置 `ANTHROPIC_FOUNDRY_RESOURCE` 和 `ANTHROPIC_FOUNDRY_API_KEY` 环境变量。

将 `Anthropic::BedrockMantleClient` 与 `bedrock-mantle` 端点配合使用，将 `Anthropic::BedrockClient` 与 `bedrock-runtime` 端点上的 `InvokeModel` API 配合使用。

Ruby SDK 还通过 `Anthropic::GoogleCloudClient` 支持 Claude Models on Google Cloud (Anthropic managed)。此客户端目前处于 beta 阶段。它包含在主 `anthropic` gem 中，并且需要 `googleauth` gem。请向构造函数传递 `workspace_id:`，或设置 `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID` 环境变量。有关客户端如何解析工作区 ID，请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。

## 语义化版本

此软件包遵循 [SemVer](https://semver.org/spec/v2.0.0.html) 约定。

此软件包将对（非运行时）`*.rbi` 和 `*.rbs` 类型定义的改进视为非破坏性变更。

对 Claude API 的每个请求都使用一个 API 版本。使用此 SDK 时，您的请求会使用 SDK 主版本发布时的当前 API 版本，因此您需要通过升级到新的主版本来迁移到新的 API 版本。版本 2.0.0 是首个使用 `2026-10-21` 的版本。有关如何升级，请参阅[迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。

## 其他资源

* [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-ruby)
* [YARD 文档](https://gemdocs.org/gems/anthropic)
* [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)
* [流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
