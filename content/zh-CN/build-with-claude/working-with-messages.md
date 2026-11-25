---
title: 使用 Messages API
url: https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages
description: 有效使用 Messages API 的实用模式和示例
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 用于生成 Claude 响应的标准 API 调用。
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
    Claude API: stable
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

Anthropic 提供两种使用 Claude 进行构建的方式，每种方式适用于不同的使用场景：

|  | Messages API | Claude Managed Agents |
| ----- | ----- | ----- |
| **它是什么** | 直接的模型提示访问 | 预构建、可配置的智能体框架，运行在托管基础设施中 |
| **最适合** | 自定义智能体循环和细粒度控制 | 长时间运行的任务和异步工作 |

本指南介绍使用 Messages API 的常见模式，包括基本请求、多轮对话、预填充技术和视觉功能。有关完整的 API 规范，请参阅 [Messages API 参考](https://platform.claude.com/docs/zh-CN/api/messages/create)。如需了解托管的智能体框架，请参阅 [Claude Managed Agents 概述](https://platform.claude.com/docs/zh-CN/managed-agents/overview)。

## 基本请求和响应

<Note>
  Claude 4.7 及更高版本的模型以及 Claude Mythos Preview，以及从 [API 版本 `2026-10-21`](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sampling-parameters) 开始的所有模型均不支持 `temperature`、`top_p` 和 `top_k` 采样参数。将它们设置为非默认值会返回 400 错误，而从 `2026-10-21` 开始，设置任何值都会返回该错误。请在请求负载中省略这些参数，改用提示来引导模型的行为。请参阅[迁移指南中的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#opus-46-breaking-changes)。
</Note>

<CodeGroup>
  ```bash cURL
  #!/bin/sh
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
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
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  message = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  print(message)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  const message = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  });
  console.log(message);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }]
  };
  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .addUserMessage("Hello, Claude")
      .build();

  Message response = client.messages().create(params);
  System.out.println(response);
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
  );
  echo json_encode($message, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [
      { role: "user", content: "Hello, Claude" }
    ]
  )
  puts message
  ```
</CodeGroup>

```json Output
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello!"
    }
  ],
  "model": "claude-opus-5-5",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 12,
    "output_tokens": 6
  }
}
```

在所有模型上，拒绝响应（`stop_reason: "refusal"`）还包含一个 `stop_details` 对象，用于标识触发拒绝的策略类别。有关字段参考和示例处理代码，请参阅[拒绝的样子](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)。

## 多轮对话

Messages API 是无状态的，这意味着您始终需要将完整的对话历史发送给 API。您可以使用这种模式随时间逐步构建对话。较早的对话轮次不一定实际来自 Claude。您可以使用合成的 `assistant` 消息。

借助目前处于 beta 阶段的[消息线程](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads)，API 可以将对话保留下来，这样之后的每个请求只需发送新消息。线程会在其最后一次请求后 5 分钟过期，并在其最后一次使用后不超过 1 小时内被删除。

<CodeGroup>
  ```bash cURL
  #!/bin/sh
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "Hello, Claude"},
        {"role": "assistant", "content": "Hello!"},
        {"role": "user", "content": "Can you describe LLMs to me?"}

      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}' \
    --message '{role: assistant, content: "Hello!"}' \
    --message '{role: user, content: "Can you describe LLMs to me?"}'
  ```

  ```python Python
  message = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[
          {"role": "user", "content": "Hello, Claude"},
          {"role": "assistant", "content": "Hello!"},
          {"role": "user", "content": "Can you describe LLMs to me?"},
      ],
  )
  print(message)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  const message = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      { role: "user", content: "Hello, Claude" },
      { role: "assistant", content: "Hello!" },
      { role: "user", content: "Can you describe LLMs to me?" }
    ]
  });
  console.log(message);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages =
      [
          new UserMessageParam { Content = "Hello, Claude" },
          new AssistantMessageParam { Content = "Hello!" },
          new UserMessageParam { Content = "Can you describe LLMs to me?" }
      ]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  		anthropic.NewAssistantMessage(anthropic.NewTextBlock("Hello!")),
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Can you describe LLMs to me?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .addUserMessage("Hello, Claude")
      .addAssistantMessage("Hello!")
      .addUserMessage("Can you describe LLMs to me?")
      .build();

  Message response = client.messages().create(params);
  System.out.println(response);
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => 'Hello, Claude'],
          ['role' => 'assistant', 'content' => 'Hello!'],
          ['role' => 'user', 'content' => 'Can you describe LLMs to me?'],
      ],
      model: Model::CLAUDE_OPUS_5_5,
  );

  echo json_encode($message, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [
      { role: "user", content: "Hello, Claude" },
      { role: "assistant", content: "Hello!" },
      { role: "user", content: "Can you describe LLMs to me?" }
    ]
  )
  puts message
  ```
</CodeGroup>

```json Output
{
  "id": "msg_018gCsTGsXkYJVqYPxTgDHBU",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Sure, I'd be happy to provide..."
    }
  ],
  "model": "claude-opus-5-5",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 30,
    "output_tokens": 309
  }
}
```

### 消息中的 system 角色

如需在对话中途添加新的系统指令，请在用户轮次之后紧接着包含一条 `"role": "system"` 的消息。该消息不能作为 `messages` 中的第一个条目。此功能适用于 Claude Fable 5.2、[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview)、Claude Fable 5.1、[Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview)、Claude Fable 5、[Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview)、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Sonnet 5.5 和 Claude Haiku 5.5。请参阅完整的[放置规则](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#limitations)。

对话中途的系统消息与顶层 `system` 字段具有相同的权威性，但由于它被追加到消息历史的末尾，因此不会使其之前的任何已缓存前缀失效。对于应从第一轮就开始适用的指令，请使用顶层 `system` 字段；对于仅在之后才变得相关的指令，请使用对话中途的系统消息。

有关完整指南，包括如何将其与[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)结合使用，请参阅[对话中途的系统消息与工具变更](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)。

## 预填充 Claude 的响应

“Prefill”（预填充）是您在 `assistant` 消息中编写并放在 `messages` 最后位置的文本。位于 `messages` 中其他任何位置的 `assistant` 消息都不是预填充。Claude 会接续预填充的文本，响应中只包含接续的内容。

使用预填充来设定响应的开头方式，例如让响应以固定格式开始，或跳过开场白。

API 是否接受预填充取决于模型，而不取决于 API 版本：

* **Claude 4.5 及更早版本的模型：** 除非该模型在您使用的平台上已退役，否则 API 接受预填充。对已退役模型的所有请求都会失败。要查看模型的状态，请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。
* **所有其他模型：** API 会返回 400 错误（请参阅 Claude API 错误页面的[不支持预填充](https://platform.claude.com/docs/zh-CN/api/errors#prefill-not-supported)部分）。请改用[预填充的替代方案](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages#alternatives-to-a-prefill)之一。

Claude Opus 4.5 已在 [Anthropic 运营的平台](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations#overview)上弃用。Claude Sonnet 4.5 已在相同平台上弃用。有关退役日期和推荐的替代模型，请参阅模型弃用页面上的[弃用历史](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations#deprecation-history)。[合作伙伴运营的平台](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations#overview)会自行制定退役时间表。

Claude Opus 4.5 和 Claude Sonnet 4.5 的推荐替代模型不接受预填充。

以下示例在 Claude Opus 4.5 上运行，并设置多项选择答案的格式。预填充 `The answer is (` 恰好在答案字母之前结束，这样 Claude 会首先写出该字母。由于该答案只有一个字母，此示例还将 `max_tokens` 设置为 `1`。对于较长的答案，请设置更高的 `max_tokens`。

<CodeGroup>
  ```bash cURL
  #!/bin/sh
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-4-5-20251101",
      "max_tokens": 1,
      "messages": [
        {"role": "user", "content": "What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae"},
        {"role": "assistant", "content": "The answer is ("}
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-4-5-20251101 \
    --max-tokens 1 \
    --message '{role: user, content: "What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae"}' \
    --message '{role: assistant, content: "The answer is ("}'
  ```

  ```python Python
  message = anthropic.Anthropic().messages.create(
      model="claude-opus-4-5-20251101",
      max_tokens=1,
      messages=[
          {
              "role": "user",
              "content": "What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae",
          },
          {"role": "assistant", "content": "The answer is ("},
      ],
  )
  print(message)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  const message = await anthropic.messages.create({
    model: "claude-opus-4-5-20251101",
    max_tokens: 1,
    messages: [
      {
        role: "user",
        content: "What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae"
      },
      { role: "assistant", content: "The answer is (" }
    ]
  });
  console.log(message);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus4_5_20251101,
      MaxTokens = 1,
      Messages = [
          new UserMessageParam { Content = "What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae" },
          new AssistantMessageParam { Content = "The answer is (" }
      ]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus4_5_20251101,
  	MaxTokens: 1,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae")),
  		anthropic.NewAssistantMessage(anthropic.NewTextBlock("The answer is (")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_4_5_20251101)
      .maxTokens(1L)
      .addUserMessage("What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae")
      .addAssistantMessage("The answer is (")
      .build();

  Message response = client.messages().create(params);
  System.out.println(response);
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1,
      messages: [
          ['role' => 'user', 'content' => 'What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae'],
          ['role' => 'assistant', 'content' => 'The answer is ('],
      ],
      model: Model::CLAUDE_OPUS_4_5_20251101,
  );
  echo json_encode($message, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_4_5_20251101,
    max_tokens: 1,
    messages: [
      {
        role: "user",
        content: "What is Latin for ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae"
      },
      { role: "assistant", content: "The answer is (" }
    ]
  )
  puts message
  ```
</CodeGroup>

```json Output
{
  "id": "msg_01Q8Faay6S7QPTvEUUQARt7h",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "C"
    }
  ],
  "model": "claude-opus-4-5-20251101",
  "stop_reason": "max_tokens",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 42,
    "output_tokens": 1
  }
}
```

请注意此响应中的两个字段：

* **`text`：** 值为 `C`。它只包含接续的内容，不包含预填充。
* **`stop_reason`：** 值为 `max_tokens`，因为此示例使响应在一个令牌后结束。

要获取完整文本 `The answer is (C`，请在您的代码中将预填充与 `text` 值拼接起来。

在 Claude 4.5 及更早版本的模型上，预填充受以下限制：

* **空白字符：** 如果预填充以空白字符结尾，API 会返回 400 错误。
* **结构化输出：** 如果请求同时设置了 `output_config.format`，API 会返回 400 错误（请参阅"结构化输出"页面的[功能兼容性](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#feature-compatibility)部分）。
* **[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)：** 如果请求同时将 `thinking.type` 设置为 `enabled`，预填充不会导致错误。API 会将思考视为关闭，因此响应中没有 `thinking` 块。

每个 400 错误中的 `message` 都会说明其原因。

### 预填充的替代方案

请以一条 `user` 消息结束 `messages`，以代替预填充。然后，根据您希望从响应中获得的内容选择一种方法：

* **遵循 schema 的 JSON：** 使用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)，它会约束响应遵循该 schema。
* **其他输出格式：** 在系统提示（即请求的 [`system` 字段](https://platform.claude.com/docs/zh-CN/api/messages/create#create.system)）中描述该格式。
* **无开场白：** 在系统提示中要求直接回答。

与预填充不同，系统提示指令并不能保证响应的开头方式，因此请在您的代码中检查响应的文本。

有关用什么来代替其他常见的预填充，请参阅"提示最佳实践"页面的[从预填充响应迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices#migrating-away-from-prefilled-responses)部分。

要将此多项选择示例迁移到不接受预填充的模型，请按照以下步骤操作：

1. 更改模型并移除 `assistant` 消息。
2. 在系统提示中要求只返回答案的字母。
3. 将 `max_tokens` 设置得足够高，以容纳整个响应。在[默认开启思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#configuring-thinking)上，思考令牌也会计入 `max_tokens`。
4. 从响应的 `text` 块中读取该字母。该块可能位于 `thinking` 块之后。不要将该字母与预填充拼接。

## 视觉

Claude 可以读取请求中的文本和图像。您可以使用 `base64`、`url` 或 `file` 源类型提供图像。`file` 源类型引用通过 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 上传的图像。支持的媒体类型为 `image/jpeg`、`image/png`、`image/gif` 和 `image/webp`。有关更多详细信息，请参阅[视觉](https://platform.claude.com/docs/zh-CN/build-with-claude/vision)指南。

<CodeGroup>
  ```bash cURL
  #!/bin/sh

  # 选项 1：base64 编码的图像
  IMAGE_BASE64=$(curl "https://platform.claude.com/docs/images/vision-example.jpg" | base64 | tr -d '\n')

  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d @- <<EOF
  {
    "model": "claude-opus-5-5",
    "max_tokens": 1024,
    "messages": [
      {"role": "user", "content": [
        {"type": "image", "source": {
          "type": "base64",
          "media_type": "image/jpeg",
          "data": "$IMAGE_BASE64"
        }},
        {"type": "text", "text": "What is in the above image?"}
      ]}
    ]
  }
  EOF

  # 选项 2：通过 URL 引用的图像
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": [
          {"type": "image", "source": {
            "type": "url",
            "url": "https://platform.claude.com/docs/images/vision-example.jpg"
          }},
          {"type": "text", "text": "What is in the above image?"}
        ]}
      ]
    }'
  ```

  ```bash CLI
  # 选项 1：base64 编码的图像（CLI 会自动编码二进制 @file 引用）
  curl -s "https://platform.claude.com/docs/images/vision-example.jpg" \
    -o ./vision-example.jpg

  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  messages:
    - role: user
      content:
        - type: image
          source:
            type: base64
            media_type: image/jpeg
            data: "@./vision-example.jpg"
        - type: text
          text: What is in the above image?
  YAML

  # 选项 2：通过 URL 引用的图像
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  messages:
    - role: user
      content:
        - type: image
          source:
            type: url
            url: https://platform.claude.com/docs/images/vision-example.jpg
        - type: text
          text: What is in the above image?
  YAML
  ```

  ```python Python
  import base64
  import httpx2

  # 选项 1：base64 编码的图像
  image_url = "https://platform.claude.com/docs/images/vision-example.jpg"
  image_media_type = "image/jpeg"
  image_data = base64.standard_b64encode(httpx2.get(image_url).content).decode("utf-8")

  message = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[
          {
              "role": "user",
              "content": [
                  {
                      "type": "image",
                      "source": {
                          "type": "base64",
                          "media_type": image_media_type,
                          "data": image_data,
                      },
                  },
                  {"type": "text", "text": "What is in the above image?"},
              ],
          }
      ],
  )
  print(message)

  # 选项 2：通过 URL 引用的图像
  message_from_url = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[
          {
              "role": "user",
              "content": [
                  {
                      "type": "image",
                      "source": {
                          "type": "url",
                          "url": "https://platform.claude.com/docs/images/vision-example.jpg",
                      },
                  },
                  {"type": "text", "text": "What is in the above image?"},
              ],
          }
      ],
  )
  print(message_from_url)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  // 选项 1：base64 编码的图像
  const imageUrl = "https://platform.claude.com/docs/images/vision-example.jpg";
  const imageMediaType = "image/jpeg";
  const imageArrayBuffer = await (await fetch(imageUrl)).arrayBuffer();
  const imageData = Buffer.from(imageArrayBuffer).toString("base64");

  const message = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "image",
            source: {
              type: "base64",
              media_type: imageMediaType,
              data: imageData
            }
          },
          {
            type: "text",
            text: "What is in the above image?"
          }
        ]
      }
    ]
  });
  console.log(message);

  // 选项 2：通过 URL 引用的图像
  const messageFromUrl = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "image",
            source: {
              type: "url",
              url: "https://platform.claude.com/docs/images/vision-example.jpg"
            }
          },
          {
            type: "text",
            text: "What is in the above image?"
          }
        ]
      }
    ]
  });
  console.log(messageFromUrl);
  ```

  ```csharp C#
  using System.Collections.Generic;
  using System.Net.Http;
  using Anthropic;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  // 选项 1：base64 编码的图像
  string imageUrl = "https://platform.claude.com/docs/images/vision-example.jpg";

  using HttpClient httpClient = new();
  byte[] imageBytes = await httpClient.GetByteArrayAsync(imageUrl);
  string imageData = Convert.ToBase64String(imageBytes);

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages =
      [
          new UserMessageParam
          {
              Content = new List<ContentBlockParam>
              {
                  new ContentBlockParam(new ImageBlockParam(
                      new ImageBlockParamSource(new Base64ImageSource()
                      {
                          Data = imageData,
                          MediaType = MediaType.ImageJpeg,
                      })
                  )),
                  new ContentBlockParam(new TextBlockParam("What is in the above image?")),
              },
          }
      ]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);

  // 选项 2：通过 URL 引用的图像
  var parametersFromUrl = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages =
      [
          new UserMessageParam
          {
              Content = new List<ContentBlockParam>
              {
                  new ContentBlockParam(new ImageBlockParam(
                      new ImageBlockParamSource(new UrlImageSource()
                      {
                          Url = "https://platform.claude.com/docs/images/vision-example.jpg",
                      })
                  )),
                  new ContentBlockParam(new TextBlockParam("What is in the above image?")),
              },
          }
      ]
  };

  var messageFromUrl = await client.Messages.Create(parametersFromUrl);
  Console.WriteLine(messageFromUrl);
  ```

  ```go Go
  client := anthropic.NewClient()

  // 选项 1：base64 编码的图像
  imageURL := "https://platform.claude.com/docs/images/vision-example.jpg"

  req, err := http.NewRequest("GET", imageURL, nil)
  if err != nil {
  	log.Fatal(err)
  }
  req.Header.Set("User-Agent", "AnthropicDocsBot/1.0")

  resp, err := http.DefaultClient.Do(req)
  if err != nil {
  	log.Fatal(err)
  }
  defer resp.Body.Close()

  imageBytes, err := io.ReadAll(resp.Body)
  if err != nil {
  	log.Fatal(err)
  }
  imageData := base64.StdEncoding.EncodeToString(imageBytes)

  message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(
  			anthropic.NewImageBlockBase64("image/jpeg", imageData),
  			anthropic.NewTextBlock("What is in the above image?"),
  		),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(message)

  // 选项 2：通过 URL 引用的图像
  messageFromURL, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(
  			anthropic.NewImageBlock(anthropic.URLImageSourceParam{
  				URL: "https://platform.claude.com/docs/images/vision-example.jpg",
  			}),
  			anthropic.NewTextBlock("What is in the above image?"),
  		),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(messageFromURL)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  // 选项 1：base64 编码的图像
  String imageUrl = "https://platform.claude.com/docs/images/vision-example.jpg";

  HttpClient httpClient = HttpClient.newHttpClient();
  HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create(imageUrl))
      .build();
  HttpResponse<byte[]> response = httpClient.send(request, HttpResponse.BodyHandlers.ofByteArray());
  String imageData = Base64.getEncoder().encodeToString(response.body());

  List<ContentBlockParam> base64Content = List.of(
      ContentBlockParam.ofImage(
          ImageBlockParam.builder()
              .source(Base64ImageSource.builder()
                  .data(imageData)
                  .mediaType(Base64ImageSource.MediaType.IMAGE_JPEG)
                  .build())
              .build()),
      ContentBlockParam.ofText(
          TextBlockParam.builder()
              .text("What is in the above image?")
              .build())
  );

  Message message = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessageOfContentBlockParams(base64Content)
          .build());
  System.out.println(message);

  // 选项 2：通过 URL 引用的图像
  List<ContentBlockParam> urlContent = List.of(
      ContentBlockParam.ofImage(
          ImageBlockParam.builder()
              .source(UrlImageSource.builder()
                  .url("https://platform.claude.com/docs/images/vision-example.jpg")
                  .build())
              .build()),
      ContentBlockParam.ofText(
          TextBlockParam.builder()
              .text("What is in the above image?")
              .build())
  );

  Message messageFromUrl = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessageOfContentBlockParams(urlContent)
          .build());
  System.out.println(messageFromUrl);
  ```

  ```php PHP
  $client = new Client();

  // 选项 1：base64 编码的图像
  $image_url = 'https://platform.claude.com/docs/images/vision-example.jpg';
  $image_media_type = 'image/jpeg';
  $image_data = base64_encode(file_get_contents($image_url));

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [
          [
              'role' => 'user',
              'content' => [
                  [
                      'type' => 'image',
                      'source' => [
                          'type' => 'base64',
                          'media_type' => $image_media_type,
                          'data' => $image_data,
                      ],
                  ],
                  [
                      'type' => 'text',
                      'text' => 'What is in the above image?',
                  ],
              ],
          ],
      ],
      model: 'claude-opus-5-5',
  );
  echo $message;

  // 选项 2：通过 URL 引用的图像
  $message_from_url = $client->messages->create(
      maxTokens: 1024,
      messages: [
          [
              'role' => 'user',
              'content' => [
                  [
                      'type' => 'image',
                      'source' => [
                          'type' => 'url',
                          'url' => 'https://platform.claude.com/docs/images/vision-example.jpg',
                      ],
                  ],
                  [
                      'type' => 'text',
                      'text' => 'What is in the above image?',
                  ],
              ],
          ],
      ],
      model: 'claude-opus-5-5',
  );
  echo $message_from_url;
  ```

  ```ruby Ruby
  require "base64"
  require "net/http"

  client = Anthropic::Client.new

  # 选项 1：base64 编码的图像
  image_url = "https://platform.claude.com/docs/images/vision-example.jpg"
  image_media_type = "image/jpeg"
  image_data = Base64.strict_encode64(Net::HTTP.get(URI(image_url)))

  message = client.messages.create(
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "image",
            source: {
              type: "base64",
              media_type: image_media_type,
              data: image_data
            }
          },
          {
            type: "text",
            text: "What is in the above image?"
          }
        ]
      }
    ]
  )
  puts message

  # 选项 2：通过 URL 引用的图像
  message_from_url = client.messages.create(
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "image",
            source: {
              type: "url",
              url: "https://platform.claude.com/docs/images/vision-example.jpg"
            }
          },
          {
            type: "text",
            text: "What is in the above image?"
          }
        ]
      }
    ]
  )
  puts message_from_url
  ```
</CodeGroup>

```json Output
{
  "id": "msg_011CdKmWtV3oFx1C5yUbf5CY",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "This image is a beautiful minimalist/flat-design illustration of a sunset landscape. Here's what it contains:\n\n**Sky & Sun:**\n- A warm gradient sky transitioning from golden-yellow at the top to deep orange toward the horizon\n- A large pale yellow sun positioned in the upper-right area\n\n**Birds:**\n- Three small silhouetted birds flying in the upper-left portion of the sky, depicted as simple \"M\" or \"v\" shapes\n\n**Mountains:**\n- Multiple layered mountain peaks in purple and maroon tones\n- The mountains overlap to create depth, with varying shades of dusty purple and deep burgundy\n\n**Water:**\n- A dark purple body of water at the bottom of the image\n- A reflection of the sun shown as horizontal cream/peach colored lines in the center-bottom area\n\nThe overall style is clean, geometric, and uses a warm sunset color palette (oranges, yellows, purples, and maroons), giving it a peaceful, serene aesthetic typical of modern vector/flat design artwork."
    }
  ],
  "model": "claude-opus-5-5",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 1030,
    "output_tokens": 350
  }
}
```

## 后续步骤

<CardGroup cols={2}>
  <Card title="停止原因和回退" icon="list" href="https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons">
    处理每个 `stop_reason` 值，并决定响应结束时该如何处理。
  </Card>

  <Card title="Claude 的工具使用" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview">
    为 Claude 提供工具，以便在 Messages API 中调用外部服务和 API。
  </Card>

  <Card title="计算机使用工具" icon="computer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool">
    使用 Messages API 控制桌面计算机环境。
  </Card>

  <Card title="浏览器使用工具" icon="browser" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool">
    让 Claude 在您运行的浏览器中导航、阅读网页并与之交互。
  </Card>

  <Card title="结构化输出" icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs">
    从 Claude 获取有保证的、经过 schema 验证的 JSON 输出。
  </Card>

  <Card title="任务预算" icon="gauge" href="https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets">
    使用 `output_config.task_budget` 为完整的智能体循环设置建议性的令牌预算。
  </Card>
</CardGroup>

## 可用性

Messages API在所有模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

可用

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目 | 零数据保留 | HIPAA |
| ----- | ----- | ----- |
| 状态 | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格 |
| 详细信息 | 用于生成 Claude 响应的标准 API 调用。 | — |

适用的端点：

* `/v1/messages`
