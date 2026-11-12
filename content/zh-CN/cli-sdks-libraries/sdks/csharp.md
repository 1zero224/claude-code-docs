---
title: C# SDK
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/csharp
description: 为 .NET 应用程序安装和配置 Anthropic C# SDK，并集成 IChatClient
---

Anthropic C# SDK 为使用 C# 编写的应用程序提供了便捷访问 Claude API 的方式。

<Info>
  有关包含代码示例的 API 功能文档，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)。本页介绍 C# 特有的 SDK 功能和配置。
</Info>

<Warning>
  从版本 10+ 开始，`Anthropic` 包现在是 Anthropic 官方的 C# SDK。3.X 及以下的包版本此前用于 tryAGI 社区构建的 SDK，该 SDK 已迁移至 [`tryAGI.Anthropic`](https://www.nuget.org/packages/tryagi.Anthropic/)。如果您需要在项目中继续使用之前的客户端，请将包引用更新为 `tryAGI.Anthropic`。
</Warning>

## 安装

从 [NuGet](https://www.nuget.org/packages/Anthropic) 安装该包：

```bash
dotnet add package Anthropic
```

## 要求

此库需要 .NET Standard 2.0 或更高版本。

## 用法

```csharp
using System;
using Anthropic;
using Anthropic.Models.Messages;

AnthropicClient client = new();

MessageCreateParams parameters = new()
{
    MaxTokens = 1024,
    Messages = [new UserMessageParam { Content = "Hello, Claude" }],
    Model = Model.ClaudeOpus5_5,
};

var message = await client.Messages.Create(parameters);

foreach (var block in message.Content)
{
    if (block.TryPickText(out var textBlock))
    {
        Console.WriteLine(textBlock.Text);
    }
}
```

有关包括 Workload Identity Federation（工作负载身份联合）在内的身份验证选项，请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。如果您的 API 密钥是未限定到某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)，请在 `anthropic-workspace-id` 请求标头中设置工作区 ID；[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)介绍了此 SDK 的按请求设置选项。

## 客户端配置

使用环境变量配置客户端：

```csharp
using Anthropic;

// 使用 ANTHROPIC_API_KEY、ANTHROPIC_AUTH_TOKEN 和 ANTHROPIC_BASE_URL 环境变量进行配置
AnthropicClient client = new();
```

或手动配置：

```csharp
using Anthropic;

AnthropicClient client = new() { ApiKey = "my-anthropic-api-key" };
```

或结合使用这两种方法。

可用选项请参见下表：

| 属性          | 环境变量                   | 必需    | 默认值                           |
| ----------- | ---------------------- | ----- | ----------------------------- |
| `ApiKey`    | `ANTHROPIC_API_KEY`    | false | -                             |
| `AuthToken` | `ANTHROPIC_AUTH_TOKEN` | false | -                             |
| `BaseUrl`   | `ANTHROPIC_BASE_URL`   | true  | `"https://api.anthropic.com"` |

### 修改配置

要临时使用修改后的客户端配置，同时复用相同的连接池和线程池，请在任意客户端或服务上调用 `WithOptions`：

```csharp
using System;

var message = await client
    .WithOptions(options =>
        options with
        {
            BaseUrl = "https://example.com",
            Timeout = TimeSpan.FromSeconds(42),
        }
    )
    .Messages.Create(parameters);

Console.WriteLine(message);
```

您可以使用 [`with` 表达式](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression)来构造修改后的选项。

`WithOptions` 方法不会影响原始客户端或服务。

## 流式传输

“Streaming”（流式传输）：SDK 定义了返回响应"块"（chunk）流的方法，每个块在到达后即可单独处理，而无需等待完整响应。流式传输方法通常对应 [SSE](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) 或 [JSONL](https://jsonlines.org) 响应。

流式传输方法的名称中始终带有 `Streaming` 后缀，即使它没有非流式传输的变体。

这些流式传输方法返回 [`IAsyncEnumerable`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iasyncenumerable-1)：

```csharp
using System;
using Anthropic;
using Anthropic.Models.Messages;

AnthropicClient client = new();

MessageCreateParams parameters = new()
{
    MaxTokens = 1024,
    Messages = [new UserMessageParam { Content = "Hello, Claude" }],
    Model = Model.ClaudeOpus5_5,
};

await foreach (var message in client.Messages.CreateStreaming(parameters))
{
    Console.WriteLine(message);
}
```

## 错误处理

SDK 会抛出自定义的非受检异常类型：

* `AnthropicApiException`：API 错误的基类。各 HTTP 状态码对应抛出的异常子类请参见下表：

| 状态码 | 异常                                       |
| --- | ---------------------------------------- |
| 400 | `AnthropicBadRequestException`           |
| 401 | `AnthropicUnauthorizedException`         |
| 403 | `AnthropicForbiddenException`            |
| 404 | `AnthropicNotFoundException`             |
| 422 | `AnthropicUnprocessableEntityException`  |
| 429 | `AnthropicRateLimitException`            |
| 5xx | `Anthropic5xxException`                  |
| 其他  | `AnthropicUnexpectedStatusCodeException` |

此外，所有 4xx 错误都继承自 `Anthropic4xxException`。

* `AnthropicSseException`：在成功获得初始 HTTP 响应后，SSE 流式传输过程中遇到的错误。

* `AnthropicIOException`：I/O 网络错误。

* `AnthropicInvalidDataException`：无法解释已成功解析的数据。例如，访问本应为必需的属性，但 API 意外地在响应中省略了该属性。

* `AnthropicException`：所有异常的基类。

## 重试

SDK 默认自动重试 2 次，请求之间采用短暂的指数退避。

仅以下错误类型会被重试：

* 连接错误（例如，由于网络连接问题）
* 408 Request Timeout
* 409 Conflict
* 429 Rate Limit
* 5xx Internal

API 也可能明确指示 SDK 重试或不重试某个请求。

要设置自定义重试次数，请使用 `MaxRetries` 属性配置客户端：

```csharp
using Anthropic;

AnthropicClient client = new() { MaxRetries = 3 };
```

或使用 `WithOptions` 配置单个方法调用：

```csharp
using System;

var message = await client
    .WithOptions(options =>
        options with { MaxRetries = 3 }
    )
    .Messages.Create(parameters);

Console.WriteLine(message);
```

## 超时

请求默认在 10 分钟后超时。

要设置自定义超时，请使用 `Timeout` 选项配置客户端：

```csharp
using System;
using Anthropic;

AnthropicClient client = new() { Timeout = TimeSpan.FromSeconds(42) };
```

或使用 `WithOptions` 配置单个方法调用：

```csharp
using System;

var message = await client
    .WithOptions(options =>
        options with { Timeout = TimeSpan.FromSeconds(42) }
    )
    .Messages.Create(parameters);

Console.WriteLine(message);
```

## 分页

SDK 定义了返回分页结果列表的方法。它提供了便捷的方式，既可以一次访问一页结果，也可以跨所有页面逐项访问结果。

### 自动分页

要遍历所有页面中的所有结果，请使用 `Paginate` 方法，该方法会根据需要自动获取更多页面。该方法返回 [`IAsyncEnumerable`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iasyncenumerable-1)：

```csharp
using System;

var page = await client.Messages.Batches.List(parameters);
await foreach (var item in page.Paginate())
{
    Console.WriteLine(item);
}
```

### 手动分页

要访问单个页面的项目并手动请求下一页，请使用 `Items` 属性以及 `HasNext` 和 `Next` 方法：

```csharp
var page = await client.Messages.Batches.List();
while (true)
{
    foreach (var item in page.Items)
    {
        Console.WriteLine(item);
    }
    if (!page.HasNext())
    {
        break;
    }
    page = await page.Next();
}
```

## 响应验证

在极少数情况下，API 可能返回与预期类型不匹配的响应。默认情况下，SDK 在这种情况下不会抛出异常。只有当您直接访问该属性时，它才会抛出 `AnthropicInvalidDataException`。

如果您希望预先检查响应是否完全符合类型，可以调用 `Validate`：

```csharp
var message = await client.Messages.Create(parameters);
message.Validate();
```

或使用 `ResponseValidation` 选项配置客户端：

```csharp
using Anthropic;

AnthropicClient client = new() { ResponseValidation = true };
```

或使用 `WithOptions` 配置单个方法调用：

```csharp
using System;

var message = await client
    .WithOptions(options =>
        options with { ResponseValidation = true }
    )
    .Messages.Create(parameters);

Console.WriteLine(message);
```

## IChatClient 集成

SDK 提供了 `Microsoft.Extensions.AI.Abstractions` 库中 `IChatClient` 接口的实现。这使得 `AnthropicClient`（以及 `Anthropic.Services.IBetaService`）可以与集成了这些核心抽象的其他库一起使用。例如，MCP C# SDK（`ModelContextProtocol`）库中的工具可以直接与通过 `IChatClient` 公开的 `AnthropicClient` 一起使用。

```csharp
using Anthropic;
using Microsoft.Extensions.AI;
using ModelContextProtocol.Client;

// 使用 ANTHROPIC_API_KEY、ANTHROPIC_AUTH_TOKEN 和 ANTHROPIC_BASE_URL 环境变量进行配置
AnthropicClient client = new();

IChatClient chatClient = client.AsIChatClient("claude-opus-5-5")
    .AsBuilder()
    .UseFunctionInvocation()
    .Build();

// 使用 MCP C# SDK 中的 McpClient
McpClient learningServer = await McpClient.CreateAsync(
    new HttpClientTransport(new() { Endpoint = new("https://learn.microsoft.com/api/mcp") }));

ChatOptions options = new() { Tools = [.. await learningServer.ListToolsAsync()] };

Console.WriteLine(await chatClient.GetResponseAsync("Tell me about IChatClient", options));
```

## 请求和响应

要向 Claude API 发送请求，请构建一个 `Params` 类的实例，并将其传递给相应的客户端方法。收到响应后，它会被反序列化为一个 C# 类的实例。

例如，`client.Messages.Create` 接受 `MessageCreateParams` 的实例并返回 `Task<Message>` 的实例。

## 高级用法

### 二进制响应

SDK 定义了返回二进制响应的方法，这些方法用于不一定需要解析的 API 响应，例如非 JSON 数据。

这些方法返回 `HttpResponse`：

```csharp
using System;
using Anthropic.Models.Files;

FileDownloadParams parameters = new() { FileID = "file_id" };

var response = await client.Files.Download(parameters);

Console.WriteLine(response);
```

要将响应内容保存到文件或任意 [`Stream`](https://learn.microsoft.com/en-us/dotnet/api/system.io.stream)，请使用 [`CopyToAsync`](https://learn.microsoft.com/en-us/dotnet/api/system.io.stream.copytoasync) 方法：

```csharp
using System.IO;

using var response = await client.Files.Download(parameters);
using var contentStream = await response.ReadAsStream();
using var fileStream = File.Open(path, FileMode.OpenOrCreate);
await contentStream.CopyToAsync(fileStream); // Or any other Stream
```

### 原始响应

SDK 定义了将响应反序列化为 C# 类实例的方法。要访问响应头、状态码或原始响应体，请在客户端或服务上的任意 HTTP 方法调用前加上 `WithRawResponse`：

```csharp
var response = await client.WithRawResponse.Messages.Create(parameters);
var statusCode = response.StatusCode;
var headers = response.Headers;
```

也可以通过 `RawMessage` 属性访问原始的 `HttpResponseMessage`。

对于非流式传输响应，如有需要，您可以将响应反序列化为 C# 类的实例：

```csharp
using System;
using Anthropic.Models.Messages;

var response = await client.WithRawResponse.Messages.Create(parameters);
Message deserialized = await response.Deserialize();
Console.WriteLine(deserialized);
```

对于流式传输响应，如有需要，您可以将响应反序列化为 `IAsyncEnumerable`：

```csharp
using System;

var response = await client.WithRawResponse.Messages.CreateStreaming(parameters);
await foreach (var item in response.Enumerate())
{
    Console.WriteLine(item);
}
```

### 日志记录

<Warning>
  所有日志消息仅用于调试。日志消息的格式和内容可能会在不同版本之间发生变化。
</Warning>

通过设置环境变量启用调试日志记录：

```bash
export ANTHROPIC_LOG=debug
```

### OpenTelemetry 追踪

当您的应用程序使用 `AddSource("com.anthropic.sdk.csharp*")` 将 SDK 的源添加到其 OpenTelemetry tracer provider 时，SDK 会为其发送的每个请求发出一个 span。要将这些 span 发送到您的可观测性平台，请参阅[在 Anthropic SDK 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

### 未记录的 API 功能

SDK 针对已记录的 API 进行了类型化，以便于使用。不过，它也支持使用 API 中未记录或尚未支持的部分。

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

C# SDK 通过单独的 NuGet 包支持以下平台：

* **Claude Models on Google Cloud (Google managed)：** `Anthropic.Vertex`。有关客户端设置，请参阅 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)。

* **Bedrock：** `Anthropic.Bedrock`。使用哪个客户端取决于端点：

  * **`bedrock-mantle`：** `AnthropicBedrockMantleClient`。它接受一个可选的 `MantleAwsClientOptions` 配置对象。
  * **`bedrock-runtime`：** `AnthropicBedrockClient`，用于 `InvokeModel` API。它接受 `AnthropicBedrockCredentialsHelper.FromEnv()` 或显式凭据。

* **Claude Platform on AWS：** `Anthropic.Aws`。使用 `AnthropicAwsClient`；在客户端上设置 `WorkspaceId` 或设置 `ANTHROPIC_AWS_WORKSPACE_ID` 环境变量（请参阅[工作区](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#workspaces)）。以 beta 形式提供。

* **Foundry：** `Anthropic.Foundry`。将 `AnthropicFoundryClient` 与 `DefaultAnthropicFoundryCredentials.FromEnv()` 或显式凭据一起使用。

C# SDK 还通过 `Anthropic.GoogleCloud` 包中的 `AnthropicGoogleCloudClient` 支持 Claude Models on Google Cloud (Anthropic managed)。此客户端处于 beta 阶段。请在可选的 `GoogleCloudClientOptions` 值上设置 `WorkspaceId`，或设置 `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID` 环境变量。请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。

## 语义化版本控制

此包通常遵循 [SemVer](https://semver.org/spec/v2.0.0.html) 约定，但某些向后不兼容的更改可能会作为次要版本发布：

1. 对库内部的更改，这些内部内容在技术上是公开的，但并非有意或有文档说明供外部使用。
2. 预计在实践中不会影响绝大多数用户的更改。

我们非常重视向后兼容性，以确保您可以获得顺畅的升级体验。

对 Claude API 的每个请求都使用一个 API 版本。使用此 SDK 时，您的请求会使用 SDK 主版本发布时的当前 API 版本，因此您需要通过升级到新的主版本来迁移到新的 API 版本。版本 13.0.0 是首个使用 `2026-10-21` 的版本。有关如何升级，请参阅[迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。

## 其他资源

* [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-csharp)
* [NuGet 包](https://www.nuget.org/packages/Anthropic)
* [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)
* [流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
