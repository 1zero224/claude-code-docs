---
title: Beta 标头
url: https://platform.claude.com/docs/zh-CN/api/beta-headers
description: 通过 `anthropic-beta` 标头或 SDK 的 `betas` 参数，在实验性功能成为标准 API 的一部分之前访问它们。
---

Beta 标头（beta headers）允许您在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

<Info>
  [客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 提供了一个 `client.beta` (csharp, go: `client.Beta`; java: `client.beta()`; php: `$client->beta`) 命名空间，用于在启用 beta 功能的情况下调用 API。
</Info>

## 如何使用 beta 标头

要访问 beta 功能，请在您的 API 请求中包含 `anthropic-beta` 标头：

```http
POST /v1/messages
Authorization: Bearer YOUR_API_KEY
anthropic-version: 2026-10-21
anthropic-beta: BETA_FEATURE_NAME
content-type: application/json
```

每个功能的文档都会说明需要发送的确切 beta 名称。[API 概述](https://platform.claude.com/docs/zh-CN/api/overview)列出了处于 beta 阶段的 API。

以下示例发送了一个使用 [上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing) beta 的请求。SDK 通过 `betas` (csharp, go: `Betas`; java: `.addBeta()`) 接收 beta 名称，并为您发送 `anthropic-beta` 标头：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: context-management-2025-06-27" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "Hello, Claude"}
      ]
    }'
  ```

  ```bash CLI
  ant beta:messages create \
    --beta context-management-2025-06-27 \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  client = Anthropic()

  response = client.beta.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
      betas=["context-management-2025-06-27"],
  )

  print(response.content)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const msg = await client.beta.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["context-management-2025-06-27"]
  });

  console.log(msg.content);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var message = await client.Beta.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new BetaUserMessageParam { Content = "Hello, Claude" }],
          Betas = ["context-management-2025-06-27"],
      }
  );

  Console.WriteLine(string.Join("\n", message.Content));
  ```

  ```go Go
  client := anthropic.NewClient()

  message, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude")),
  	},
  	Betas: []anthropic.AnthropicBeta{anthropic.AnthropicBetaContextManagement2025_06_27},
  })
  if err != nil {
  	panic(err)
  }

  fmt.Printf("%+v\n", message.Content)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .addUserMessage("Hello, Claude")
      .addBeta(AnthropicBeta.CONTEXT_MANAGEMENT_2025_06_27)
      .build();

  BetaMessage message = client.beta().messages().create(params);
  System.out.println(message.content());
  ```

  ```php PHP
  $client = new Client();

  $message = $client->beta->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['context-management-2025-06-27'],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{role: "user", content: "Hello, Claude"}],
    betas: ["context-management-2025-06-27"]
  )

  puts(message.content)
  ```
</CodeGroup>

<Warning>
  Beta 功能是实验性的，可能会：

  * 在通知后发生破坏性变更
  * 被弃用或移除
  * 具有不同的速率限制或定价
  * 并非在所有地区都可用
</Warning>

### 多个 beta 功能

要在单个请求中使用多个 beta 功能，请在标头中包含所有功能名称，并以逗号分隔：

```http
anthropic-beta: feature1,feature2,feature3
```

您也可以在同一请求中多次发送 `anthropic-beta` 标头。Claude API 会读取每个 `anthropic-beta` 标头，因此以下示例与上一个示例等效：

```http
anthropic-beta: feature1
anthropic-beta: feature2
anthropic-beta: feature3
```

使用 SDK 时，请列出每个功能（例如 `betas=["feature1", "feature2"]` (typescript, ruby: `betas: ["feature1", "feature2"]`; php: `betas: ['feature1', 'feature2']`; csharp: `Betas = ["feature1", "feature2"]`; go: `Betas: []anthropic.AnthropicBeta{"feature1", "feature2"}`; java: `.addBeta("feature1").addBeta("feature2")`)）。使用 CLI 时，请传递单个 `--beta` 标志，并用逗号分隔功能名称（例如 `--beta feature1,feature2`）。您也可以重复使用该标志（例如 `--beta feature1 --beta feature2`）。

### 其他平台上的 beta 功能

在所有接受 beta 名称的平台上，beta 名称都是相同的，但并非每个平台都接受所有 beta。[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)、[Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 和 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 与 Claude API 一样，通过 `anthropic-beta` 标头接收 beta 名称。要发送多个 beta，请将它们的名称放在一个标头中，并用逗号分隔：Claude Models on Google Cloud (Google managed) 只读取一个 `anthropic-beta` 标头并忽略其余标头，因此其他标头中的 beta 不会生效。

在 Amazon Bedrock 上，beta 名称的放置位置取决于您调用的 API：

* [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)（`bedrock-mantle` 端点）通过 `anthropic-beta` 标头接收它们。
* [InvokeModel API](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy) 从请求体而非标头中读取它们。请在请求体的 `anthropic_beta` 数组中列出它们，每个元素一个名称，例如 `"anthropic_beta": ["feature1", "feature2"]`。

### 特定端点的标头

某些 beta API 的作用范围限定于特定端点，并且要求在每个请求中都包含特定功能的 beta 标头：

| 端点                                             | Beta 标头                     |
| ---------------------------------------------- | --------------------------- |
| `/v1/agents`、`/v1/sessions`、`/v1/environments` | `managed-agents-2026-04-01` |
| `/v1/tunnels`                                  | `mcp-tunnels-2026-06-22`    |
| `/v1/memory_stores` 及其子资源                      | `agent-memory-2026-07-22`   |

SDK 的 `client.beta` (csharp, go: `client.Beta`; java: `client.beta()`; php: `$client->beta`) 命名空间会自动添加这些标头。仅在发出原始 HTTP 请求时才需要您自行添加。有关详细信息，请参阅 [Managed Agents 概述](https://platform.claude.com/docs/zh-CN/managed-agents/overview)、[使用智能体记忆](https://platform.claude.com/docs/zh-CN/managed-agents/memory)和 [MCP 隧道参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/reference#tunnels-api)。

适用于同一端点的特定端点标头并不总是可以组合使用。在记忆存储端点上，`agent-memory-2026-07-22` 会取代 `managed-agents-2026-04-01`：在同一请求中同时发送两者会返回 `400` 错误。SDK 会自动为每个端点发送正确的标头。

### 版本命名约定

Beta 功能名称通常遵循 `feature-name-YYYY-MM-DD` 的模式，其中日期表示该 beta 的发布时间。请始终使用文档中记载的确切 beta 功能名称。

## 错误处理

如果您使用了无效的 beta 名称，或者使用了您的组织无权访问的 beta，您将收到 `400` 错误响应：

```json Output
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Unexpected value(s) `invalid-beta-name` for the `anthropic-beta` header. Please consult our documentation at platform.claude.com/docs or try again without the header."
  },
  "request_id": "req_011CcnGfC9fELffo2EALu4Wd"
}
```

## 获取帮助

有关 beta 功能的更新，请参阅[发布说明](https://platform.claude.com/docs/zh-CN/release-notes/overview)。如需生产问题方面的帮助，请联系[支持团队](https://support.claude.com/)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="错误" icon="info" href="https://platform.claude.com/docs/zh-CN/api/errors">
    了解 Claude API 返回的 HTTP 状态码、错误响应结构和请求 ID，并使用 SDK 的类型化异常处理错误。
  </Card>

  <Card title="API 概述" icon="compass" href="https://platform.claude.com/docs/zh-CN/api/overview">
    探索 Claude API 的功能，包括处于 beta 阶段的 API。
  </Card>
</CardGroup>
