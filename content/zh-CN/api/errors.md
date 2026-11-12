---
title: Claude API 错误
url: https://platform.claude.com/docs/zh-CN/api/errors
description: 了解 Claude API 返回的 HTTP 状态码、错误响应结构和请求 ID，并使用 SDK 的类型化异常处理错误。
---

## HTTP 错误

API 遵循可预测的 HTTP 错误代码格式：

* 400 - `invalid_request_error`：您的请求格式或内容存在问题。此错误类型也可能用于本节未列出的其他 4XX 状态码。当用量达到您为组织或工作区[设置的支出限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#setting-your-own-spend-limit)时，API 也会返回 400，但 [Claude Code 工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#claude-code-workspace)上的限制除外，该限制可能改为返回 429。

* 401 - `authentication_error`：您的 [API 密钥](https://platform.claude.com/docs/zh-CN/get-api-key)存在问题（例如，格式错误、已被撤销或已过期；请参阅[密钥过期](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)）。在 Claude Platform on AWS 上，这也可能表示您的 AWS 凭证或 SigV4 签名存在问题。

  如果 401 错误的消息以 `CORS requests are not allowed for this Organization` 开头，则问题并不在于您的 API 密钥。该请求包含了 `Origin` 标头（浏览器会添加此标头），而您的组织无法直接从浏览器调用 Claude API。该消息后续为 `because of custom retention settings` 或 `because of its settings`。前者仅在数据保留安排（例如 “zero data retention”（零数据保留），即 ZDR）下出现；后者是一般情况，其中包括 Compliance API。对浏览器请求做任何更改都无法解决此问题，因此请从您自己的后端发送请求；请参阅[基于浏览器的应用程序](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#browser-based-applications)。

* 402 - `billing_error`：您的账单或付款信息存在问题。请在 [Claude Console](https://platform.claude.com) 中检查您的付款详情；如果您使用的是 Claude Platform on AWS，请在 AWS Marketplace 中检查。

* 403 - `permission_error`：您的 API 密钥没有使用指定资源的权限。请在 [Claude Console](https://platform.claude.com) 中检查您组织的访问权限和工作区设置。

* 404 - `not_found_error`：未找到所请求的资源。请检查端点路径以及请求 URL 中的所有资源 ID。

* 409 - `conflict_error`：请求与资源的当前状态冲突。例如，资源被并发修改，或者某个必须唯一的值已被使用。请解决冲突，然后重试请求。

* 413 - `request_too_large`：请求超过了允许的最大字节数。有关各端点的最大值，请参阅[请求大小限制](https://platform.claude.com/docs/zh-CN/api/errors#request-size-limits)。

* 429 - `rate_limit_error`：您的组织已达到[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)、达到其使用层级的每月支出上限，或达到 Claude Code 工作区的支出限制。层级支出上限导致的 429 没有 `retry-after` 标头，并且在访问恢复之前会持续失败；有关如何识别它，请参阅[达到支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#reaching-your-spend-cap)。

* 500 - `api_error`：Anthropic 系统内部发生了意外错误。请使用 “exponential backoff”（指数退避）重试请求；如果错误持续存在，请携带[请求 ID](https://platform.claude.com/docs/zh-CN/api/errors#request-id) 联系支持团队。如果错误类型为 `timeout_error`，请参阅 504 条目。

* 504 - `timeout_error`：请求在处理过程中超时。API 也会以其他状态码返回 `timeout_error`，例如当请求在响应开始之前达到服务器的时间限制时返回 500。对于长时间运行的 Messages API 请求，请使用[流式传输 Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)。有关更多选项，请参阅[长请求](https://platform.claude.com/docs/zh-CN/api/errors#long-requests)。

* 529 - `overloaded_error`：API 暂时过载。

  <Warning>
    当 API 在所有用户范围内遇到高流量时，可能会出现 529 错误。

    在极少数情况下，如果您的组织用量急剧增加，您可能会因 API 的加速限制而看到 429 错误。为避免触及加速限制，请逐步增加流量并保持一致的使用模式。
  </Warning>

官方 SDK 会使用指数退避自动重试暂时性故障（例如连接错误、速率限制和 5xx 服务器错误），默认重试两次，并在存在 `retry-after` 标头时遵循该标头。SDK 客户端接受 `max_retries` (typescript, php: `maxRetries`; java: `.maxRetries()`; csharp: `MaxRetries`; go: `option.WithMaxRetries()`) 来配置或禁用此行为。

当通过 “server-sent events”（服务器发送事件），即 SSE 接收[流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)响应时，错误可能在 API 返回 200 响应之后发生。在这种情况下，错误处理不遵循这些标准机制。有关流中错误的结构，请参阅[错误事件](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming#error-events)。

## 请求大小限制

第一方 Claude API 强制执行以下请求大小限制。有关其他平台，请参阅 [API 概述](https://platform.claude.com/docs/zh-CN/api/overview#request-size-limits)。

| 端点类型                                                                                   | 最大请求大小 |
| -------------------------------------------------------------------------------------- | ------ |
| Messages API                                                                           | 128 MB |
| Token Counting API                                                                     | 128 MB |
| [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) | 256 MB |
| [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)            | 500 MB |

如果您超过这些限制，您将收到 413 `request_too_large` 错误。在第一方 Claude API 上，Cloudflare 会在请求到达 API 服务器之前返回此错误。

## 错误结构

API 始终以 JSON 形式返回错误，其中包含一个顶级 `error` 对象，该对象始终包含 `type` 和 `message` 值。响应还包括一个 `request_id` 字段，以便于跟踪和调试。例如：

```json JSON
{
  "type": "error",
  "error": {
    "type": "not_found_error",
    "message": "The requested resource could not be found."
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

根据[版本控制](https://platform.claude.com/docs/zh-CN/api/versioning)策略，这些对象中的值可能会扩展，并且 `type` 值可能会随着时间的推移而增加。

## SDK 错误类型

官方 SDK 会针对这些错误抛出类型化异常，而不是返回原始 JSON。例如，404 会表现为 `anthropic.NotFoundError` (typescript: `Anthropic.NotFoundError`; ruby: `Anthropic::Errors::NotFoundError`; java: `com.anthropic.errors.NotFoundException`; csharp: `AnthropicNotFoundException`; php: `Anthropic\Core\Exceptions\NotFoundException`; go: `*anthropic.Error`)。Go SDK 对所有状态码只有一种错误类型 `*anthropic.Error`：请根据 `StatusCode` 进行分支处理。请捕获 SDK 的类型化类，而不是对错误消息进行字符串匹配，并优先处理最具体的类。您所用 SDK 的页面记录了完整的异常层次结构：

* [Python](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/python#handling-errors) · [TypeScript](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#handling-errors) · [C#](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/csharp#error-handling) · [Go](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/go#error-handling) · [Java](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#error-handling) · [PHP](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/php#error-handling) · [Ruby](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/ruby#handling-errors)

## 请求 ID

每个 API 响应都包含一个唯一的 `request-id` 标头。此标头包含类似 `req_018EeWyXxfu5pfWkrYcMdjWG` 的值。相同的标识符在[错误响应正文](https://platform.claude.com/docs/zh-CN/api/errors#error-shapes)中显示为 `request_id` 字段。当就特定请求联系支持团队时，请包含此 ID 以帮助快速解决您的问题。

在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上，响应包含两个请求 ID：AWS 请求 ID（`x-amzn-requestid`，主要，在 CloudTrail 中建立索引）和 Anthropic 请求 ID（`request-id`，次要）。使用 AWS 请求 ID 进行 CloudTrail 查找，使用 Anthropic 请求 ID 处理 Anthropic 支持工单。

Python 和 TypeScript SDK 在顶层响应对象上以 `_request_id` 属性公开请求 ID。C#、Go、Java 和 PHP SDK 通过其原始响应访问器公开请求 ID，Ruby SDK 则通过[中间件](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/middleware)公开。在除 Ruby 之外的所有 SDK 中，使用 `with_raw_response` (typescript: `.withResponse()`; java: `.withRawResponse()`; csharp: `WithRawResponse`; go: `option.WithResponseInto()`; php: `->raw`) 读取任何其他[响应标头](https://platform.claude.com/docs/zh-CN/api/overview#response-headers)，例如 `anthropic-organization-id` 和 [`anthropic-workspace-id`](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#identify-the-workspace-behind-an-api-response)。在 Ruby 中，请使用相同的中间件。在 Claude Platform on AWS 上，还可以使用原始响应访问器读取 AWS 请求 ID（`x-amzn-requestid`）：

<CodeGroup>
  ```bash cURL
  # -i 会在响应体之前打印响应头，包括 request-id
  curl -sS -i https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}]
    }'
  ```

  ```bash CLI
  # 使用 --debug 时，request-id 标头会打印到 stderr：
  ant --debug messages create \
    --model claude-sonnet-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.messages.create(
      model="claude-sonnet-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  print(f"Request ID: {message._request_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-sonnet-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  });
  console.log("Request ID:", message._request_id);
  ```

  ```csharp C#
  AnthropicClient client = new();

  using var response = await client.WithRawResponse.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }]
  });
  Console.WriteLine($"Request ID: {response.RequestID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  var response *http.Response
  _, err := client.Messages.New(
  	context.Background(),
  	anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeSonnet5,
  		MaxTokens: 1024,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  		},
  	},
  	option.WithResponseInto(&response),
  )
  if err != nil {
  	panic(err)
  }

  fmt.Println("Request ID:", response.Header.Get("request-id"))
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.http.HttpResponseFor;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      HttpResponseFor<Message> response = client.messages().withRawResponse().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_SONNET_5)
              .maxTokens(1024)
              .addUserMessage("Hello, Claude")
              .build()
      );

      IO.println("Request ID: " + response.requestId().orElse(null));
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->raw->create([
      'model' => 'claude-sonnet-5',
      'maxTokens' => 1024,
      'messages' => [['role' => 'user', 'content' => 'Hello, Claude']],
  ]);
  echo 'Request ID: ' . $response->getHeaderLine('request-id') . "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 在每个请求的中间件中读取响应头，该中间件会接收
  # SDK 解析之前的原始 HTTP 响应
  request_id = nil
  read_request_id = lambda do |request, call_next|
    response = call_next.call(request)
    # response.headers 中的键为小写
    request_id = response.headers["request-id"]
    response
  end

  client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    request_options: { middleware: [read_request_id] }
  )
  puts "Request ID: #{request_id}"
  ```

  ```python Python (Claude Platform on AWS)
  from anthropic import AnthropicAWS

  client = AnthropicAWS(aws_region="us-west-2")

  response = client.messages.with_raw_response.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  print(f"AWS request ID: {response.headers.get('x-amzn-requestid')}")
  message = response.parse()
  print(f"Anthropic request ID: {message._request_id}")
  ```

  ```typescript TypeScript (Claude Platform on AWS)
  import AnthropicAws from "@anthropic-ai/aws-sdk";

  const client = new AnthropicAws({ awsRegion: "us-west-2" });

  const { response: raw, request_id } = await client.messages
    .create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    })
    .withResponse();
  console.log("AWS request ID:", raw.headers.get("x-amzn-requestid"));
  console.log("Anthropic request ID:", request_id);
  ```
</CodeGroup>

有关其他语言的 Claude Platform on AWS 请求 ID 示例，请参阅[请求 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#request-ids)。

## 长请求

<Warning>
  对于长时间运行的请求，尤其是超过 10 分钟的请求，请考虑使用[流式传输 Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming) 或 [Message Batches API](https://platform.claude.com/docs/zh-CN/api/messages/batches/create)。
</Warning>

避免在不使用[流式传输 Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming) 或 [Message Batches API](https://platform.claude.com/docs/zh-CN/api/messages/batches/create) 的情况下设置较大的 `max_tokens` 值：

* 某些网络可能会在不定的时间段后断开空闲连接，这 可能导致请求失败或超时，而无法收到来自 Anthropic 的响应。
* 不同网络的可靠性各不相同。[Message Batches API](https://platform.claude.com/docs/zh-CN/api/messages/batches/create) 允许您轮询结果，而不需要不间断的网络连接，从而帮助您 管理网络问题带来的风险。

如果您正在构建直接的 API 集成，设置 [TCP socket keep-alive](https://tldp.org/HOWTO/TCP-Keepalive-HOWTO/programming.html)（TCP 套接字保活）可以减少某些网络上空闲连接超时的影响。

[Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 会验证您的非流式 Messages API 请求预计不会超过 10 分钟的超时时间。它还会为 TCP keep-alive 设置套接字选项。

如果您不需要增量处理事件，SDK 可以为您消费流并返回完整的 `Message` 对象，与非流式调用返回的内容完全相同：

<CodeGroup exclude="shell:cURL">
  <CodeGroupItem language="cURL">
    curl 请求会打印原始流事件，无法将它们合并为最终消息。要从流中获取完整的 `Message` 对象，请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。
  </CodeGroupItem>

  ```bash CLI
  # CLI 以流式传输方式输出事件；--format jsonl 每行输出一个事件
  ant messages create --stream --format jsonl <<'YAML'
  model: claude-sonnet-5
  max_tokens: 128000
  messages:
    - role: user
      content: Write a detailed analysis...
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  with client.messages.stream(
      max_tokens=128000,
      messages=[{"role": "user", "content": "Write a detailed analysis..."}],
      model="claude-sonnet-5",
  ) as stream:
      message = stream.get_final_message()

  print(next(block.text for block in message.content if block.type == "text"))
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const stream = client.messages.stream({
    max_tokens: 128000,
    messages: [{ role: "user", content: "Write a detailed analysis..." }],
    model: "claude-sonnet-5"
  });

  const message = await stream.finalMessage();
  const textBlock = message.content.find((block) => block.type === "text");
  if (textBlock && textBlock.type === "text") {
    console.log(textBlock.text);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5,
      MaxTokens = 128000,
      Messages = [new UserMessageParam { Content = "Write a detailed analysis..." }]
  };

  var message = await client.Messages.CreateStreaming(parameters).Aggregate();
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5,
  	MaxTokens: 128000,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Write a detailed analysis...")),
  	},
  })

  message := anthropic.Message{}
  for stream.Next() {
  	event := stream.Current()
  	if err := message.Accumulate(event); err != nil {
  		log.Fatal(err)
  	}
  }
  if err := stream.Err(); err != nil {
  	log.Fatal(err)
  }

  for _, block := range message.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		fmt.Println(textBlock.Text)
  		break
  	}
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.helpers.MessageAccumulator;
  import com.anthropic.models.messages.ContentBlock;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_SONNET_5)
          .maxTokens(128000L)
          .addUserMessage("Write a detailed analysis...")
          .build();

      MessageAccumulator accumulator = MessageAccumulator.create();
      try (var streamResponse = client.messages().createStreaming(params)) {
          streamResponse.stream().forEach(accumulator::accumulate);
      }

      Message message = accumulator.message();
      message.content().stream()
          .filter(ContentBlock::isText)
          .findFirst()
          .flatMap(ContentBlock::text)
          .ifPresent(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  use Anthropic\Lib\Streaming\MessageAccumulator;

  $client = new Client();

  $stream = $client->messages->createStream(
      model: 'claude-sonnet-5',
      maxTokens: 128000,
      messages: [['role' => 'user', 'content' => 'Write a detailed analysis...']],
  );

  $accumulator = MessageAccumulator::forMessages();
  foreach ($stream as $event) {
      $accumulator->accumulate($event);
  }

  echo array_find($accumulator->message()->content, static fn ($block): bool => $block->type === 'text')->text;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.stream(
    model: Anthropic::Model::CLAUDE_SONNET_5,
    max_tokens: 128000,
    messages: [{ role: "user", content: "Write a detailed analysis..." }]
  ).accumulated_message

  puts message.content.find { it.type == :text }.text
  ```
</CodeGroup>

有关更多详细信息，请参阅[无需处理事件即可获取最终消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming#get-the-final-message-without-handling-events)。

## 常见验证错误

### 不支持预填充

Claude 4.6 及更高版本的模型以及 [Claude Mythos Preview](https://anthropic.com/glasswing) 不支持 “prefilling”（预填充）助手消息。向这些模型中的任何一个发送最后一条助手消息为预填充内容的请求，都会返回 400 `invalid_request_error`：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "This model does not support assistant message prefill. The conversation must end with a user message."
  }
}
```

请改用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)（在支持它的模型上）、系统提示指令或 [`output_config.format`](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#usage)。

### 思考块无法修改

如果最近的助手消息包含在发送回 API 之前被编辑、重新排序、过滤掉或重建的 `thinking` 或 `redacted_thinking` 块，则请求会返回 400 `invalid_request_error`。错误消息以有问题的块的位置开头（例如 `messages.1.content.0`），并包含：

```text wrap
`thinking` or `redacted_thinking` blocks in the latest assistant message cannot be modified. These blocks must remain as they were in the original response.
```

在工具使用中，请将助手轮次中的每个 `thinking` 和 `redacted_thinking` 块按收到时的原样传回，包括 `thinking` 字段为空的块。如果您的应用程序在重新发送之前按类型过滤内容块，请同时包含 `thinking` 和 `redacted_thinking`。请参阅思考故障排除中的[已修改思考块的修复方法](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-blocks-modified)，以及[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)和[保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-thinking)。

### 不支持扩展思考

Claude 4.7 及更高版本的模型不支持扩展思考。向这些模型中的任何一个发送 `thinking: {"type": "enabled"}` 会返回 400 `invalid_request_error`：

```text wrap
"thinking.type.enabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

请改用 [adaptive thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（自适应思考）。[迁移到自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#migrating-to-adaptive-thinking)展示了参数映射，思考故障排除则介绍了[扩展思考错误的按症状修复方法](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-type-enabled)。

### 不支持自适应思考

仅支持扩展思考的模型（Claude 4.5 及更早版本的模型）会以 400 `invalid_request_error` 拒绝 `thinking: {"type": "adaptive"}`：

```text wrap
adaptive thinking is not supported on this model
```

在这些模型上，请使用 `thinking: {"type": "enabled", "budget_tokens": N}`；有关配置，请参阅[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)，另请参阅思考故障排除中的[自适应思考错误的按症状修复方法](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-type-adaptive)。

### 无法禁用思考

在 Claude Fable 5.2、[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview)、Claude Fable 5.1、[Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview)、Claude Fable 5、[Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview)、Claude Opus 5.5 和 [Claude Mythos Preview](https://anthropic.com/glasswing) 上，思考始终处于开启状态。向这些模型中的任何一个发送 `thinking: {"type": "disabled"}` 都会返回 400 `invalid_request_error`。在除 Claude Mythos Preview 之外的所有这些模型上，消息内容为：

```text wrap
"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

在 Claude Mythos Preview（这些模型中唯一接受扩展思考的模型）上，消息内容为：

```text wrap
"thinking.type.disabled" is not supported for this model. Thinking defaults to adaptive mode when not specified; use "thinking.type.enabled" with "budget_tokens" for extended thinking.
```

在 Claude Sonnet 5.5 上，思考不能设置为 `disabled`。请使用 `thinking: {"type": "between_tools"}` 作为最低的思考设置，它会关闭前置思考。发送 `thinking: {"type": "disabled"}` 会返回 400 `invalid_request_error`，消息如下：

```text wrap
To turn thinking off on this model, send "thinking": {"type": "between_tools"} instead of {"type": "disabled"}. The model does not think before responding. The short updates it writes between tool calls come back as thinking blocks.
```

在 `xhigh` 或 `max` [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)（努力程度）下，使用 `between_tools` 的请求也会返回 400 `invalid_request_error`。消息表示思考已被禁用，因为 `between_tools` 没有前置思考：

```text wrap
output_config.effort 'xhigh' is not supported when thinking is disabled on this model. Use effort 'high' or below, or enable thinking.
```

使用 `between_tools` 时，effort 不能在对话中途更改：如果某条消息的 `output_config.effort` 与当前生效的级别不同，将返回 400 错误。错误会指出设置新级别的消息的位置：

```text wrap
messages.N: output_config.effort 'low' differs from the 'high' in effect before it; effort cannot change when thinking is disabled on this model. Use effort 'high', or enable thinking.
```

Claude Haiku 5.5 接受 `thinking: {"type": "disabled"}`，并对其施加与 `between_tools` 相同的两项 effort 限制：在 `xhigh` 或 `max` effort 下，或者某条消息的 `output_config.effort` 与当前生效的级别不同时，请求会返回 400 `invalid_request_error`，并附带前面所示的相应消息。

在这两条消息中，"enable thinking"（启用思考）指的是自适应思考：省略 `thinking` 字段或发送 `thinking: {"type": "adaptive"}`。Claude Sonnet 5.5 和 Claude Haiku 5.5 也会以 400 错误拒绝 `"enabled"`。要在每个轮次中改变努力程度，请使用自适应思考。

向 Claude Sonnet 5.5 以外的任何模型发送 `thinking: {"type": "between_tools"}` 都会返回 400 `invalid_request_error`：

```text wrap
"thinking.type.between_tools" is not supported for this model.
```

有关修复方法，请参阅思考故障排除中的 [`between_tools` 和努力程度错误](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-type-between-tools)。

省略 `thinking` 参数，请求将以自适应思考运行。要在不关闭思考的情况下让响应中不包含思考内容，请在思考配置中设置 `display: "omitted"`。请参阅思考故障排除中的[禁用思考错误的修复方法](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-type-disabled)。

### 不支持强制工具使用

Claude Fable 5.2、[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview)、Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.1 和 [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview) 不支持强制工具使用。向这些模型中的任何一个发送 `tool_choice: {"type": "any"}` 或 `tool_choice: {"type": "tool", "name": "..."}`（包括在[令牌计数端点](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)上）都会返回 400 `invalid_request_error`：

```text wrap
tool_choice: type "tool" and "any" are not supported for this model.
```

`tool_choice: {"type": "auto"}`（默认值）和 `{"type": "none"}` 是被接受的。请将 `auto` 与[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)结合使用，以保持工具输入符合 schema；当您需要响应本身采用固定的 JSON 结构时，请使用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。请参阅[强制工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#forcing-tool-use)。

### 不支持的计算机使用工具版本

对于不接受的任何[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)工具版本，Claude Fable 5.2、Claude Mythos 5.2、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 都会返回 400 错误。在 Claude API 上，这包括带有其 beta 标头的 `computer_20251124`。它们在任何平台上都不接受 `computer_20250124`。Claude API 的 `invalid_request_error` 消息会指出被拒绝的类型，并在 `Did you mean one of` 之后列出模型接受的工具类型。对于 Claude Opus 5.5，消息开头为：

```text wrap
'claude-opus-5-5' does not support tool types: computer_20251124.
```

对于所请求模型不支持的任何 Anthropic 定义的工具类型，Claude API 都会返回相同的消息。Claude Fable 5.2、Claude Mythos 5.2、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 通过 `computer_toolset_20260801` 工具集支持计算机使用。请在不带 beta 标头的情况下声明 `{"type": "computer_toolset_20260801"}`，并按照[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)中的说明更新您的智能体循环。要从 `computer_20250124` 迁移，请参阅 Claude Haiku 5.5 迁移指南中的[将计算机使用迁移到工具集](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#computer-use-toolset)。

在 Amazon Bedrock 上，Claude Fable 5.2、Claude Mythos 5.2、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 也接受带有 `computer-use-2025-11-24` beta 标头的 `computer_20251124` 工具。使用 Amazon Bedrock 的 InvokeModel API 时，请在请求正文的 `anthropic_beta` 数组中发送 `computer-use-2025-11-24`，而不是在标头中发送。有关该 API，请参阅 [Amazon Bedrock 上的 Claude（bedrock-runtime）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)。[早期工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#earlier-tool-versions)列出了每个早期版本应搭配使用的模型。

### 绑定到不同对话的思考块

此 400 `invalid_request_error` 通常意味着您发回的某个思考块未通过前缀检查：该块之前的某些内容在该块生成后发生了更改，例如 `system` 提示、`tools` 或更早的消息。Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 会执行此检查。对于未设置 `prefix_mismatch_behavior` 的请求，[未设置 `prefix_mismatch_behavior` 时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)说明了哪些请求会收到此错误。该消息以第一个检查失败的块的位置开头：

```text wrap
messages.{i}.content.{j}: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block".
```

如果没有 `thinking-binding-controls-2026-08-01` beta 标头，消息中还会指出该标头。重新发送相同的历史记录会以同样的方式失败，即使您已修复进行了该编辑的代码也是如此。要继续对话，请在重试以及之后的每个请求中发送该 beta 标头并附带 `prefix_mismatch_behavior: "drop_block"`。或者，删除错误中指出的块以及之后的每个 `thinking` 和 `redacted_thinking` 块，然后重试一次。在 Claude Sonnet 5.5 和 Claude Haiku 5.5 上，`block_binding` 仅适用于 `thinking: {"type": "adaptive"}`，因此在任何其他思考设置下，请删除这些块。为防止新对话失败，请保持历史记录仅追加（append-only）。来自目标模型无法读取的模型的块会被丢弃，而不是被拒绝。请参阅[保持前缀不变](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)以及思考故障排除中的[无效思考块签名的修复方法](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-block-signature)。

在不附带 `thinking-binding-controls-2026-08-01` [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers)的情况下发送 `thinking.block_binding`，会返回 400 `invalid_request_error`，其消息结尾为：

```text wrap
block_binding: Extra inputs are not permitted
```

请添加该标头，或移除该字段。

### 出站 Web 身份联合已禁用（Claude Platform on AWS）

如果对 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 的每个请求都返回 `"Outbound web identity federation is disabled for your account"`，请为每个 AWS 账户运行一次 `aws iam enable-outbound-web-identity-federation`。有关详细信息，请参阅[启用出站 Web 身份联合](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#enable-outbound-web-identity-federation)。

## 后续步骤

<CardGroup cols={3}>
  <Card title="思考故障排除" icon="wrench" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting">
    针对思考配置 400 错误、空思考块和 `max_tokens` 停止的以症状为先的修复方法。
  </Card>

  <Card title="速率限制" icon="gauge" href="https://platform.claude.com/docs/zh-CN/api/rate-limits">
    为了减少滥用并管理 API 上的容量，对组织可以使用 Claude API 的量设置了限制。
  </Card>

  <Card title="流式消息" icon="lightning" href="https://platform.claude.com/docs/zh-CN/build-with-claude/streaming">
    使用服务器发送事件增量流式传输 Messages API 响应，包括文本、工具使用和扩展思考增量。
  </Card>
</CardGroup>
