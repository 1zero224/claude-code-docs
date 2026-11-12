---
title: TypeScript SDK
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript
description: 为 Node.js、Deno、Bun 和浏览器环境安装和配置 Anthropic TypeScript SDK
---

此库提供了从 TypeScript 或 JavaScript 便捷访问 Claude API 的方式。

<Info>
  有关包含代码示例的 API 功能文档，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)。本页介绍 TypeScript 特有的 SDK 功能和配置。
</Info>

## 安装

```bash
npm install @anthropic-ai/sdk
```

## 要求

支持 TypeScript >= 5.0。

支持以下运行时：

* Node.js 20 LTS 或更高版本（[非 EOL](https://endoflife.date/nodejs)）。
* Deno v1.28.0 或更高版本。
* Bun 1.0 或更高版本。
* Cloudflare Workers。
* Vercel Edge Runtime。
* Jest 28 或更高版本，使用 `"node"` 环境（目前不支持 `"jsdom"`）。
* Nitro v2.6 或更高版本。
* Web 浏览器：默认禁用，以避免暴露您的秘密 API 凭据（请参阅 [API 密钥最佳实践](https://support.claude.com/en/articles/9767949-api-key-best-practices-keeping-your-keys-safe-and-secure)）。通过将 `dangerouslyAllowBrowser` 显式设置为 `true` 来启用浏览器支持。

请注意，目前不支持 React Native。

如果您对其他运行时环境感兴趣，请在 [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-typescript)上提交 issue 或为现有 issue 投票。

## 用法

```typescript
const client = new Anthropic({
  apiKey: process.env["ANTHROPIC_API_KEY"] // This is the default and can be omitted
});

const message = await client.messages.create({
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  model: "claude-opus-5-5"
});

for (const block of message.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```

有关包括 Workload Identity Federation（工作负载身份联合）在内的身份验证选项，请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。如果您的 API 密钥是未限定到某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)，请在 `anthropic-workspace-id` 请求标头中设置工作区 ID；[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)介绍了此 SDK 的按请求设置选项。

## 请求和响应类型

此库包含所有请求参数和响应字段的 TypeScript 定义。您可以按如下方式导入和使用它们：

```typescript
const client = new Anthropic({
  apiKey: process.env["ANTHROPIC_API_KEY"] // This is the default and can be omitted
});

const params: Anthropic.MessageCreateParams = {
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  model: "claude-opus-5-5"
};
const message: Anthropic.Message = await client.messages.create(params);
```

每个方法、请求参数和响应字段的文档都可以在文档字符串中找到，并且在大多数现代编辑器中悬停时会显示。

## 计算令牌

您可以通过 `usage` 响应属性查看给定请求的确切用量，例如：

```typescript
const message = await client.messages.create({
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  model: "claude-opus-5-5"
});
console.log(message.usage);
// { input_tokens: 25, output_tokens: 13 }
```

## 流式传输响应

SDK 支持使用 “Server-Sent Events”（服务器发送事件），即 SSE，进行 “streaming”（流式传输）响应。

```typescript
const client = new Anthropic();

const stream = await client.messages.create({
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  model: "claude-opus-5-5",
  stream: true
});
for await (const messageStreamEvent of stream) {
  console.log(messageStreamEvent.type);
}
```

如果您需要取消流，可以从循环中 `break` 或调用 `stream.controller.abort()`。

## 流式传输辅助工具

此库为流式传输消息提供了多种便利功能，例如：

```typescript
const anthropic = new Anthropic();

const stream = anthropic.messages
  .stream({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: "Say hello there!"
      }
    ]
  })
  .on("text", (text) => {
    console.log(text);
  });

const message = await stream.finalMessage();
console.log(message);
```

使用 `client.messages.stream(...)` 进行流式传输会提供各种便捷的辅助工具，包括事件处理程序和累积功能。

或者，您可以使用 `client.messages.create({ ..., stream: true })`，它只返回流中事件的异步可迭代对象，因此占用更少的内存（它不会为您构建最终的消息对象）。

## 工具辅助工具

此 SDK 提供了在 Messages API 中创建和运行工具的辅助函数。您可以使用 Zod schema 或 JSON Schema 来描述工具的输入。然后，您可以使用 `client.beta.messages.toolRunner()` 方法运行这些工具。此方法负责将所选模型生成的输入传递给正确的工具，并将结果传回模型。

有关 “tool use”（工具使用）的更多详细信息，请参阅[使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)。要使用此 SDK 运行浏览器使用工具或计算机使用工具，请参阅[使用 SDK 工具集进行浏览器和计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk)。

```typescript
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const anthropic = new Anthropic();

const weatherTool = betaZodTool({
  name: "get_weather",
  inputSchema: z.object({
    location: z.string()
  }),
  description: "Get the current weather in a given location",
  run: (input) => {
    return `The weather in ${input.location} is foggy and 60°F`;
  }
});

const finalMessage = await anthropic.beta.messages.toolRunner({
  model: "claude-opus-5-5",
  max_tokens: 1000,
  messages: [{ role: "user", content: "What is the weather in San Francisco?" }],
  tools: [weatherTool]
});

console.log(finalMessage.content);
```

### 工具错误

要将工具中的错误报告回模型，请从 `run` 函数中抛出 `ToolError`。与普通的 `Error` 不同，`ToolError` 接受内容块，允许您在错误响应中包含图像或其他结构化内容：

```typescript
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { ToolError } from "@anthropic-ai/sdk/resources/beta/messages";
import { z } from "zod";

const screenshotTool = betaZodTool({
  name: "take_screenshot",
  inputSchema: z.object({ url: z.string() }),
  description: "Take a screenshot of a webpage",
  run: async (input) => {
    if (!isValidUrl(input.url)) {
      throw new ToolError(`Invalid URL: ${input.url}`);
    }
    const result = await takeScreenshot(input.url);
    if (result.error) {
      // 附上错误截图，以便模型能看到出了什么问题
      throw new ToolError([
        { type: "text", text: `Failed to load page: ${result.error}` },
        {
          type: "image",
          source: { type: "base64", data: result.screenshot, media_type: "image/png" }
        }
      ]);
    }
    return [
      {
        type: "image",
        source: { type: "base64", data: result.screenshot, media_type: "image/png" }
      }
    ];
  }
});
```

如果抛出的是普通 `Error`，其消息将被转换为文本内容块。

## 工具使用

此 SDK 支持工具使用，也称为 “function calling”（函数调用）。有关更多详细信息，请参阅[使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)。

## MCP 辅助工具

此 SDK 提供了用于与 [“Model Context Protocol”，即 MCP](https://modelcontextprotocol.io/) 服务器集成的辅助工具。这些辅助工具将 MCP 类型转换为 Claude API 类型，从而在使用 MCP 工具、提示和资源时减少样板代码。

<Tip>
  Claude API 还支持 [`mcp_servers` 参数](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)，让 Claude 可以直接连接到远程 MCP 服务器。当您拥有可通过 URL 访问的远程服务器且只需要工具支持时，请使用 `mcp_servers`。当您需要本地 MCP 服务器、提示、资源或对 MCP 连接进行更多控制时，请使用 MCP 辅助工具。
</Tip>

```typescript
import {
  mcpTools,
  mcpMessages,
  mcpResourceToContent,
  mcpResourceToFile
} from "@anthropic-ai/sdk/helpers/beta/mcp";
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

const anthropic = new Anthropic();

// 连接到 MCP 服务器
const transport = new StdioClientTransport({ command: "mcp-server", args: [] });
const mcpClient = new Client({ name: "my-client", version: "1.0.0" });
await mcpClient.connect(transport);

// 使用 MCP 提示
const { messages } = await mcpClient.getPrompt({ name: "my-prompt" });
const response = await anthropic.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  messages: mcpMessages(messages)
});
console.log(response.content);

// 通过 toolRunner 使用 MCP 工具
const { tools } = await mcpClient.listTools();
const finalMessage = await anthropic.beta.messages.toolRunner({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Use the available tools" }],
  tools: mcpTools(tools, mcpClient)
});
console.log(finalMessage.content);

// 将 MCP 资源用作内容
const resource = await mcpClient.readResource({ uri: "file:///path/to/doc.txt" });
await anthropic.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        mcpResourceToContent(resource),
        { type: "text", text: "Summarize this document" }
      ]
    }
  ]
});

// 将 MCP 资源作为文件上传
const fileResource = await mcpClient.readResource({ uri: "file:///path/to/data.json" });
await anthropic.files.upload({ file: mcpResourceToFile(fileResource) });
```

### MCP 错误处理

如果某个 MCP 值不受 Claude API 支持（例如，不支持的内容类型、不支持的 MIME 类型、非 HTTP/HTTPS 资源链接），转换函数会抛出 `UnsupportedMCPValueError`。

## 消息批处理

此 SDK 在 `client.messages.batches` 命名空间下支持[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)。

### 创建批处理

Message Batches 接受一个请求数组，其中每个对象都有一个 `custom_id` 标识符，以及与标准 Messages API 完全相同的请求 `params`：

```typescript
const batch = await client.messages.batches.create({
  requests: [
    {
      custom_id: "my-first-request",
      params: {
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages: [{ role: "user", content: "Hello, world" }]
      }
    },
    {
      custom_id: "my-second-request",
      params: {
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages: [{ role: "user", content: "Hi again, friend" }]
      }
    }
  ]
});
```

### 获取批处理结果

当消息批次处理完成（由 `.processing_status === 'ended'` 表示）后，您可以使用 `.batches.results()` 访问结果：

```typescript
const results = await client.messages.batches.results(batch.id);
for await (const entry of results) {
  if (entry.result.type === "succeeded") {
    console.log(entry.result.message.content);
  }
}
```

## 文件上传

与文件上传对应的请求参数可以以多种不同形式传递：

* `File`（或具有相同结构的对象）
* `fetch` `Response`（或具有相同结构的对象）
* `fs.ReadStream`
* `toFile` 辅助函数的返回值

请显式设置内容类型，因为 Files API 不会为您推断它：

```typescript
import fs from "node:fs";
import Anthropic, { toFile } from "@anthropic-ai/sdk";

const client = new Anthropic();

// 如果您可以使用 Node `fs`，请使用 `fs.createReadStream()`：
await client.files.upload({
  file: await toFile(fs.createReadStream("/path/to/file"), undefined, {
    type: "application/json"
  })
});

// 或者，如果您可以使用 Web `File` API，则可以传入 `File` 实例：
await client.files.upload({
  file: new File(["my bytes"], "file.txt", { type: "text/plain" })
});
// 您也可以传入 `fetch` `Response`：
await client.files.upload({
  file: await fetch("https://somesite/file")
});

// 或者传入 `Buffer` / `Uint8Array`
await client.files.upload({
  file: await toFile(Buffer.from("my bytes"), "file", { type: "text/plain" })
});
await client.files.upload({
  file: await toFile(new Uint8Array([0, 1, 2]), "file", { type: "text/plain" })
});
```

## 处理错误

当库无法连接到 API， 或者 API 返回非成功状态码（即 4xx 或 5xx 响应）时， 会抛出 `APIError` 的子类：

```typescript
const message = await client.messages
  .create({
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    model: "claude-opus-5-5"
  })
  .catch(async (err) => {
    if (err instanceof Anthropic.APIError) {
      console.log(err.status); // 400
      console.log(err.name); // BadRequestError
      console.log(err.headers); // {server: 'nginx', ...}
    } else {
      throw err;
    }
  });
```

错误代码如下：

| 状态码   | 错误类型                       |
| ----- | -------------------------- |
| 400   | `BadRequestError`          |
| 401   | `AuthenticationError`      |
| 403   | `PermissionDeniedError`    |
| 404   | `NotFoundError`            |
| 409   | `ConflictError`            |
| 422   | `UnprocessableEntityError` |
| 429   | `RateLimitError`           |
| >=500 | `InternalServerError`      |
| 不适用   | `APIConnectionError`       |

## 请求 ID

> 有关调试请求的更多信息，请参阅[请求 ID](https://platform.claude.com/docs/zh-CN/api/errors#request-id)。

SDK 中的所有对象响应都提供一个 `_request_id` 属性，该属性取自 `request-id` 响应头，以便您记录失败的请求并将其报告给 Anthropic。

```typescript
const message = await client.messages.create({
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  model: "claude-opus-5-5"
});
console.log(message._request_id); // req_018EeWyXxfu5pfWkrYcMdjWG
```

## 重试

某些错误默认会自动重试 2 次，并采用短暂的指数退避。 连接错误（例如，由于网络连接问题）、408 Request Timeout、409 Conflict、 429 Rate Limit 以及 >=500 Internal 错误默认都会重试。

您可以使用 `maxRetries` 选项来配置或禁用此功能：

```typescript
// 为所有请求配置默认值：
const client = new Anthropic({
  maxRetries: 0 // default is 2
});

// 或者，按请求单独配置：
await client.messages.create(
  {
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    model: "claude-opus-5-5"
  },
  { maxRetries: 5 }
);
```

## 超时

默认情况下，请求在 10 分钟后超时。但是，如果您指定了较大的 `max_tokens` 值并且 *没有*使用流式传输，默认超时将使用以下公式动态计算：

```typescript
const minimum = 10 * 60;
const calculated = (60 * 60 * maxTokens) / 128_000;
return calculated < minimum ? minimum * 1000 : calculated * 1000;
```

这将产生最长 60 分钟的超时，并根据 `max_tokens` 参数进行缩放，除非在请求或客户端级别进行了覆盖。

您可以使用 `timeout` 选项进行配置：

```typescript
// 为所有请求配置默认值：
const client = new Anthropic({
  timeout: 20 * 1000 // 20 seconds (default is 10 minutes)
});

// 按请求覆盖：
await client.messages.create(
  {
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    model: "claude-opus-5-5"
  },
  { timeout: 5 * 1000 }
);
```

超时时会抛出 `APIConnectionTimeoutError`。

请注意，超时的请求[默认会重试两次](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#retries)。

## 长时间请求

<Warning>
  对于运行时间较长的请求，请考虑使用[流式 Messages API](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#streaming-responses)。
</Warning>

避免在不使用流式传输的情况下设置较大的 `max_tokens` 值。 某些网络可能会在一段时间后断开空闲连接，这 可能导致请求失败或在未收到 Anthropic 响应的情况下[超时](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#timeouts)。

如果预计非流式请求的时长超过大约 10 分钟，此 SDK 也会抛出错误。 传递 `stream: true` 或在客户端或请求级别[覆盖](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#timeouts) `timeout` 选项可以禁用此错误。

如果非流式请求的预期 “latency”（延迟）超过[超时时间](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#timeouts)， 客户端将终止连接并在未收到响应的情况下重试。

在 `fetch` 实现支持的情况下，SDK 会设置 [TCP 套接字保活](https://tldp.org/HOWTO/TCP-Keepalive-HOWTO/overview.html)选项， 以减少某些网络上空闲连接超时的影响。 可以通过配置自定义代理来[覆盖](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#configuring-proxies)此设置。

## 自动分页

Claude API 中的列表方法是分页的。 您可以使用 `for await ... of` 语法遍历所有页面中的项目：

```typescript
async function fetchAllMessageBatches() {
  const allMessageBatches = [];
  // 根据需要自动获取更多页面。
  for await (const messageBatch of client.messages.batches.list({ limit: 20 })) {
    allMessageBatches.push(messageBatch);
  }
  return allMessageBatches;
}
```

或者，您可以一次请求一个页面：

```typescript
let page = await client.messages.batches.list({ limit: 20 });
for (const messageBatch of page.data) {
  console.log(messageBatch);
}

// 提供了便捷方法用于手动分页：
while (page.hasNextPage()) {
  page = await page.getNextPage();
  // ...
}
```

## 默认请求头

SDK 会自动发送 `anthropic-version` 请求头，其值设置为此 SDK 主版本对应的 API 版本（请参阅[语义化版本控制](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#semantic-versioning)）。

如有需要，您可以通过按请求设置默认请求头来覆盖它。

请注意，这样做可能会导致 SDK 中出现不正确的类型以及其他意外或未定义的行为。

```typescript
const client = new Anthropic();

const message = await client.messages.create(
  {
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    model: "claude-opus-5-5"
  },
  { headers: { "anthropic-version": "My-Custom-Value" } }
);
```

## 高级用法

### 访问原始 Response 数据（例如，请求头）

可以通过所有方法返回的 `APIPromise` 类型上的 `.asResponse()` 方法访问 `fetch()` 返回的"原始" `Response`。 此方法在收到成功响应的请求头后立即返回，并且不会消费响应体，因此您可以自由编写自定义解析或流式传输逻辑。

您还可以使用 `.withResponse()` 方法获取原始 `Response` 以及解析后的数据。 与 `.asResponse()` 不同，此方法会消费响应体，并在解析完成后返回。

```typescript
const client = new Anthropic();

const response = await client.messages
  .create({
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    model: "claude-opus-5-5"
  })
  .asResponse();
console.log(response.headers.get("X-My-Header"));
console.log(response.statusText); // access the underlying Response object

const { data: message, response: raw } = await client.messages
  .create({
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    model: "claude-opus-5-5"
  })
  .withResponse();
console.log(raw.headers.get("X-My-Header"));
console.log(message.content);
```

### 日志记录

<Warning>
  所有日志消息仅用于调试目的。日志消息的格式和内容 可能会在不同版本之间发生变化。
</Warning>

#### 日志级别

您可以通过两种方式配置日志级别：

1. 通过 `ANTHROPIC_LOG` 环境变量
2. 通过 `logLevel` 客户端选项（如果设置，将覆盖环境变量）

```typescript
const client = new Anthropic({
  logLevel: "debug" // Show all log messages
});
```

可用的日志级别，从最详细到最简略：

* `'debug'` - 显示调试消息、信息、警告和错误
* `'info'` - 显示信息消息、警告和错误
* `'warn'` - 显示警告和错误（默认）
* `'error'` - 仅显示错误
* `'off'` - 禁用所有日志记录

在 `'debug'` 级别，所有 HTTP 请求和响应都会被记录，包括请求头和请求体。 某些与身份验证相关的请求头会被隐去，但请求体和响应体中的敏感数据 仍可能可见。

#### 自定义日志记录器

默认情况下，此库将日志输出到 `globalThis.console`。您也可以提供自定义日志记录器。 大多数日志库都受支持，包括 [pino](https://www.npmjs.com/package/pino)、[winston](https://www.npmjs.com/package/winston)、[bunyan](https://www.npmjs.com/package/bunyan)、[consola](https://www.npmjs.com/package/consola)、[signale](https://www.npmjs.com/package/signale) 和 [@std/log](https://jsr.io/@std/log)。如果您的日志记录器无法正常工作，请提交 issue。

提供自定义日志记录器时，`logLevel` 选项仍然控制发出哪些消息；低于 配置级别的消息不会发送到您的日志记录器。

```typescript
import pino from "pino";

const logger = pino();

const client = new Anthropic({
  logger: logger.child({ name: "Anthropic" }),
  logLevel: "debug" // Send all messages to pino, allowing it to filter
});
```

### OpenTelemetry 追踪

当您的应用程序注册 OpenTelemetry tracer provider 时，SDK 会为其发送的每个请求发出一个 span。要将这些 span 发送到您的可观测性平台，请参阅[在 Anthropic SDK 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

### 发出自定义/未记录的请求

此库具有类型定义，便于访问已记录的 API。如果您需要访问未记录的 端点、参数或响应属性，仍然可以使用此库。

#### 未记录的端点

要向未记录的端点发出请求，您可以使用 `client.get`、`client.post` 和其他 HTTP 动词。 发出这些请求时，客户端上的选项（例如重试）会被遵循。

```typescript
await client.post("/some/path", {
  body: { some_prop: "foo" },
  query: { some_query_arg: "bar" }
});
```

#### 未记录的请求参数

要使用未记录的参数发起请求，您可以在未记录的参数上使用 `// @ts-expect-error`。 此库不会在运行时验证请求是否与类型匹配，因此您发送的任何额外值 都会按原样发送。

```typescript
client.messages.create({
  // ...
  // @ts-expect-error baz is not yet public
  baz: "undocumented option"
});
```

对于使用 `GET` 动词的请求，任何额外参数都会放在查询字符串中；所有其他请求会将 额外参数放在请求体中发送。

如果您想显式发送额外参数，可以使用 `query`、`body` 和 `headers` 请求 选项来实现。

#### 未记录的响应属性

要访问未记录的响应属性，您可以在响应对象上使用 `// @ts-expect-error` 来访问， 或者将响应对象转换为所需的类型。与请求参数一样，SDK 不会 验证或剥离 API 响应中的额外属性。

### 自定义 fetch 客户端

默认情况下，此库要求定义全局 `fetch` 函数。

如果您想使用不同的 `fetch` 函数，可以对全局对象进行 polyfill：

```typescript
import fetch from "my-fetch";

globalThis.fetch = fetch;
```

或者将其传递给客户端：

```typescript
import fetch from "my-fetch";

const client = new Anthropic({ fetch });
```

### Fetch 选项

如果您想在不覆盖 `fetch` 函数的情况下设置自定义 `fetch` 选项，可以在创建客户端或发出请求时提供 `fetchOptions` 对象。（特定于请求的选项会覆盖客户端选项。）

```typescript
const client = new Anthropic({
  fetchOptions: {
    // `RequestInit` 选项
  }
});
```

### 配置代理

要修改代理行为，您可以提供自定义 `fetchOptions`，为请求添加特定于运行时的代理 选项：

<Tabs>
  <Tab title="Node.js">
    ```typescript
    import * as undici from "undici";

    const proxyAgent = new undici.ProxyAgent("http://localhost:8888");
    const client = new Anthropic({
      fetchOptions: {
        dispatcher: proxyAgent
      }
    });
    ```
  </Tab>

  <Tab title="Bun">
    ```typescript
    const client = new Anthropic({
      fetchOptions: {
        proxy: "http://localhost:8888"
      }
    });
    ```
  </Tab>

  <Tab title="Deno">
    ```typescript
    import Anthropic from "npm:@anthropic-ai/sdk";

    const httpClient = Deno.createHttpClient({ proxy: { url: "http://localhost:8888" } });
    const client = new Anthropic({
      fetchOptions: {
        client: httpClient
      }
    });
    ```
  </Tab>
</Tabs>

## Beta 功能

Beta 功能在正式发布之前提供，以便获取早期反馈并测试新功能。您可以在[功能概览](https://platform.claude.com/docs/zh-CN/build-with-claude/overview)中查看 Claude 所有功能和工具的可用性。

您可以通过客户端的 `beta` 属性访问大多数 beta API 功能。要启用特定的 beta 功能，您需要在创建消息时将相应的 [beta 请求头](https://platform.claude.com/docs/zh-CN/api/beta-headers)添加到 `betas` 字段中。

例如，要启用[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)：

```typescript
const client = new Anthropic();
const response = await client.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  betas: ["context-management-2025-06-27"]
});
```

## 运行时支持

<Accordion title="浏览器使用">
  启用 `dangerouslyAllowBrowser` 选项可能存在危险，因为它会在客户端代码中暴露您的秘密 API 凭据。Web 浏览器本质上不如服务器环境安全。任何能够访问浏览器的用户都可能检查、提取和滥用这些凭据。这可能导致他人使用您的凭据进行未经授权的访问，并可能危及敏感数据或功能。

  **什么情况下这可能不危险？**

  在某些场景中，启用浏览器支持可能不会带来重大风险：

  * **内部工具：** 如果应用程序仅在受控的内部环境中使用，且用户是可信的，则可以降低凭据暴露的风险。
  * **开发或调试目的：** 临时启用此功能可能是可以接受的，前提是凭据是短期有效的、不会同时用于生产环境，或者会频繁轮换。

  即使在这些情况下，只有当您的组织可以直接从浏览器调用 Claude API 时，浏览器支持才能正常工作。启用了"zero data retention"（零数据保留），即 ZDR，或启用了 Compliance API 的组织无法这样做，其浏览器请求会因 401 错误而失败。请参阅[基于浏览器的应用程序](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#browser-based-applications)。
</Accordion>

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

TypeScript SDK 支持以下平台：

* **Claude Models on Google Cloud (Google managed)：** `npm install @anthropic-ai/vertex-sdk`：提供 `AnthropicVertex` 客户端
* **Bedrock：** `npm install @anthropic-ai/bedrock-sdk`：提供 `AnthropicBedrockMantle` 客户端，以及用于 `bedrock-runtime` 路径的 `AnthropicBedrock`
* **Claude Platform on AWS：** `npm install @anthropic-ai/aws-sdk`：提供 `AnthropicAws` 客户端。将 `workspaceId` 传递给构造函数或设置 `ANTHROPIC_AWS_WORKSPACE_ID` 环境变量。以 beta 形式提供。
* **Foundry：** `npm install @anthropic-ai/foundry-sdk`：提供 `AnthropicFoundry` 客户端

请将 `AnthropicBedrockMantle` 与 `bedrock-mantle` 端点配合使用，将 `AnthropicBedrock` 与 `bedrock-runtime` 端点上的 `InvokeModel` API 配合使用。

TypeScript SDK 还通过 `AnthropicGoogleCloud` 客户端支持 Claude Models on Google Cloud (Anthropic managed)。该客户端目前处于 beta 阶段。要安装它，请运行 `npm install @anthropic-ai/google-cloud-sdk`。请将 `workspaceId` 传递给构造函数，或设置 `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID` 环境变量。有关客户端如何解析工作区 ID，请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。

## 语义化版本控制

此软件包通常遵循 [SemVer](https://semver.org/spec/v2.0.0.html) 约定，但某些向后不兼容的更改可能会作为次要版本发布：

1. 仅影响静态类型而不破坏运行时行为的更改。
2. 对库内部实现的更改，这些内部实现在技术上是公开的，但并非为外部使用而设计或记录。
3. 预计在实践中不会影响绝大多数用户的更改。

我们非常重视向后兼容性，以确保您可以获得顺畅的升级体验。

对 Claude API 的每个请求都使用一个 API 版本。使用此 SDK 时，您的请求会使用 SDK 主版本发布时的当前 API 版本，因此您需要通过升级到新的主版本来迁移到新的 API 版本。版本 1.0.0 是首个使用 `2026-10-21` 的版本。有关如何升级，请参阅[迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。

## 常见问题

请参阅 [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-typescript)获取常见问题解答、issue 和社区支持。

## 其他资源

* [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-typescript)
* [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)
* [流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
* [使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)
