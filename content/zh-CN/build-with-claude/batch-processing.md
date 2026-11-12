---
title: 批处理
url: https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing
description: 使用 Message Batches API 异步处理大量 Messages 请求，将成本降低 50% 并提高吞吐量。
featureMetadata:
  status: stable
  zdr:
    eligibility: not-eligible
    note: 保留 29 天；需要异步存储。
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
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Anthropic managed): stable
---

“Batch processing”（批处理）是一种高效处理大量请求的强大方法。批处理不是逐个处理请求并立即返回响应，而是允许您将多个请求一起提交以进行异步处理。这种模式在以下情况下特别有用：

* 您需要处理大量数据
* 不需要立即响应
* 您希望优化成本效率
* 您正在运行大规模评估或分析

Message Batches API 是 Anthropic 对这一模式的首个实现。

# Message Batches API

Message Batches API 是一种强大且经济高效的方式，用于异步处理大量 [Messages](https://platform.claude.com/docs/zh-CN/api/messages/create) 请求。这种方法非常适合不需要立即响应的任务，大多数批次在 1 小时内完成，同时将成本降低 50% 并提高吞吐量。

除本指南外，您还可以[直接浏览 API 参考](https://platform.claude.com/docs/zh-CN/api/messages/batches/create)。

## Message Batches API 的工作原理

当您向 Message Batches API 发送请求时：

1. 系统使用提供的 Messages 请求创建一个新的消息批次。
2. 然后该批次被异步处理，每个请求独立处理。
3. 您可以轮询批次的状态，并在所有请求处理结束后检索结果。

这对于不需要立即获得结果的批量操作特别有用，例如：

* 大规模评估：高效处理数千个测试用例。
* 内容审核：异步分析大量用户生成的内容。
* 数据分析：为大型数据集生成洞察或摘要。
* 批量内容生成：为各种用途创建大量文本（例如产品描述、文章摘要）。

### 批处理限制

* 一个消息批次最多包含 100,000 个 Message 请求或 256 MB 大小，以先达到者为准。
* 系统会尽可能快地处理每个批次，大多数批次在 1 小时内完成。您可以在所有消息完成后或 24 小时后访问批次结果，以先到者为准。如果处理未在 24 小时内完成，批次将过期。
* 批次结果在创建后 29 天内可用。之后，您仍然可以查看该批次，但其结果将无法再下载。
* 批次的作用域限定在 Claude Console 中的某个[工作区](https://platform.claude.com/settings/workspaces)。您可以查看在您的请求所运行的工作区中创建的所有批次（及其结果）。
* “Rate limit”（速率限制）同时适用于 Batches API HTTP 请求和批次中等待处理的请求数量。请参阅 [Message Batches API 速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#message-batches-api)。此外，处理速度可能会根据当前需求和您的请求量而减慢。在这种情况下，您可能会看到更多请求在 24 小时后过期。
* 由于高吞吐量和并发处理，批次可能会略微超出您在 Claude Console 中为工作区配置的[支出限制](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#setting-workspace-limits)。
* 每个批处理请求的 [`max_tokens`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.max_tokens) 必须至少为 `1`。批次中不支持 `max_tokens: 0`（[缓存预热](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#pre-warming-the-cache)），因为在批处理期间写入的临时缓存条目很可能会在后续请求运行之前过期。

### 支持的模型

所有[活跃模型](https://platform.claude.com/docs/zh-CN/models/overview)都支持 Message Batches API。

### 可以批处理的内容

几乎所有可以向 Messages API 发出的请求都可以包含在批次中。这包括：

* 视觉
* “Tool use”（工具使用），包括所有[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)（网络搜索、网页获取、代码执行、MCP 连接器、顾问和工具搜索）
* 系统消息
* 多轮对话
* “Extended thinking”（扩展思考）
* 大多数 beta 功能

由于批次中的每个请求都是独立处理的，您可以在单个批次中混合不同类型的请求。

少数 Messages API 参数在批处理请求中**不**受支持。包含其中任何一个都会返回验证错误：

| 参数                                                                                                                         | 原因                                                                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `stream: true`                                                                                                             | 批次结果以单个文件形式返回，而不是流。                                                                                                                                            |
| `speed`（[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)）                                        | 快速模式针对同步延迟进行调优，不适用于异步批处理。                                                                                                                                      |
| `fallbacks`（[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)） | 服务器端回退不适用于批次。要重试被拒绝的请求，请参阅 [Message Batches 中的拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusals-in-message-batches)。 |
| `max_tokens: 0`                                                                                                            | 请参阅[批处理限制](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#batch-limitations)。                                                       |

<Tip>
  由于批次处理可能需要超过 5 分钟，在处理具有共享上下文的批次时，请考虑将 [1 小时缓存时长](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#1-hour-cache-duration)与"prompt caching"（提示缓存）结合使用，以获得更好的缓存命中率。
</Tip>

## 定价

Batches API 可显著节省成本。所有用量均按标准 API 价格的 50% 收费。

| Model                                                                                                                                                                     | Batch input  | Batch output  |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------- | :------------ |
| Claude Fable 5.2                                                                                                                                                          | $4 / MTok    | $20 / MTok    |
| Claude Mythos 5.2 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                               | $4 / MTok    | $20 / MTok    |
| Claude Fable 5.1                                                                                                                                                          | $5 / MTok    | $25 / MTok    |
| Claude Mythos 5.1 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                               | $5 / MTok    | $25 / MTok    |
| Claude Fable 5                                                                                                                                                            | $5 / MTok    | $25 / MTok    |
| Claude Mythos 5 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                                 | $5 / MTok    | $25 / MTok    |
| Claude Opus 5.5                                                                                                                                                           | $2 / MTok    | $10 / MTok    |
| Claude Opus 5                                                                                                                                                             | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.8                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.7                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.6                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.5                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.1 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | $7.50 / MTok | $37.50 / MTok |
| Claude Opus 4 ([retired, except on Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))               | $7.50 / MTok | $37.50 / MTok |
| Claude Sonnet 5.5                                                                                                                                                         | $1 / MTok    | $5 / MTok     |
| Claude Sonnet 5                                                                                                                                                           | $1 / MTok    | $5 / MTok     |
| Claude Sonnet 4.6                                                                                                                                                         | $1.50 / MTok | $7.50 / MTok  |
| Claude Sonnet 4.5                                                                                                                                                         | $1.50 / MTok | $7.50 / MTok  |
| Claude Sonnet 4 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | $1.50 / MTok | $7.50 / MTok  |
| Claude Haiku 5.5 (for prompts up to 100,000 tokens)                                                                                                                       | $0.05 / MTok | $0.25 / MTok  |
| Claude Haiku 5.5 (for prompts over 100,000 tokens)                                                                                                                        | $0.25 / MTok | $1.25 / MTok  |
| Claude Haiku 4.5                                                                                                                                                          | $0.50 / MTok | $2.50 / MTok  |

* **MTok:** Million tokens. $5 / MTok is $5 for every million tokens.
* **Limited access:** Available only to organizations verified through Anthropic’s verification programs, such as the [Cyber Verification Program](https://support.claude.com/en/articles/14604842). To request access, apply to the program that covers your use case.
* **Retired:** May still be available on other cloud platforms. See [Model deprecations](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) for more.

## 如何使用 Message Batches API

### 准备和创建批次

消息批次由一系列创建 Message 的请求组成。单个请求的结构包括：

* 一个唯一的 [`custom_id`](https://platform.claude.com/docs/zh-CN/api/messages/batches/create#create.requests.custom_id)，用于标识 Messages 请求。必须为 1 到 64 个字符，且只能包含字母数字字符、连字符和下划线（匹配 `^[a-zA-Z0-9_-]{1,64}$`）。
* 一个包含标准 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 参数的 [`params`](https://platform.claude.com/docs/zh-CN/api/messages/batches/create#create.requests.params) 对象

您可以通过将此列表传入 `requests` 参数来[创建批次](https://platform.claude.com/docs/zh-CN/api/messages/batches/create)：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages/batches \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "requests": [
        {
          "custom_id": "my-first-request",
          "params": {
            "model": "claude-opus-5-5",
            "max_tokens": 1024,
            "messages": [
              {"role": "user", "content": "Hello, world"}
            ]
          }
        },
        {
          "custom_id": "my-second-request",
          "params": {
            "model": "claude-opus-5-5",
            "max_tokens": 1024,
            "messages": [
              {"role": "user", "content": "Hi again, friend"}
            ]
          }
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages:batches create <<'YAML'
  requests:
    - custom_id: my-first-request
      params:
        model: claude-opus-5-5
        max_tokens: 1024
        messages:
          - role: user
            content: Hello, world
    - custom_id: my-second-request
      params:
        model: claude-opus-5-5
        max_tokens: 1024
        messages:
          - role: user
            content: Hi again, friend
  YAML
  ```

  ```python Python
  from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
  from anthropic.types.messages.batch_create_params import Request

  client = anthropic.Anthropic()

  message_batch = client.messages.batches.create(
      requests=[
          Request(
              custom_id="my-first-request",
              params=MessageCreateParamsNonStreaming(
                  model="claude-opus-5-5",
                  max_tokens=1024,
                  messages=[
                      {
                          "role": "user",
                          "content": "Hello, world",
                      }
                  ],
              ),
          ),
          Request(
              custom_id="my-second-request",
              params=MessageCreateParamsNonStreaming(
                  model="claude-opus-5-5",
                  max_tokens=1024,
                  messages=[
                      {
                          "role": "user",
                          "content": "Hi again, friend",
                      }
                  ],
              ),
          ),
      ]
  )

  print(message_batch)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const messageBatch = await client.messages.batches.create({
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

  console.log(messageBatch);
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Models.Messages;
  using Anthropic.Models.Messages.Batches;

  AnthropicClient client = new();

  var batch = await client.Messages.Batches.Create(new BatchCreateParams
  {
      Requests =
      [
          new()
          {
              CustomID = "my-first-request",
              Params = new()
              {
                  Model = Model.ClaudeOpus5_5,
                  MaxTokens = 1024,
                  Messages = [new UserMessageParam { Content = "Hello, world" }]
              }
          },
          new()
          {
              CustomID = "my-second-request",
              Params = new()
              {
                  Model = Model.ClaudeOpus5_5,
                  MaxTokens = 1024,
                  Messages = [new UserMessageParam { Content = "Hi again, friend" }]
              }
          }
      ]
  });

  Console.WriteLine(batch);
  ```

  ```go Go
  client := anthropic.NewClient()

  batch, _ := client.Messages.Batches.New(context.Background(),
  	anthropic.MessageBatchNewParams{
  		Requests: []anthropic.MessageBatchNewParamsRequest{
  			{
  				CustomID: "my-first-request",
  				Params: anthropic.MessageBatchNewParamsRequestParams{
  					Model:     anthropic.ModelClaudeOpus5_5,
  					MaxTokens: 1024,
  					Messages: []anthropic.MessageParam{
  						anthropic.NewUserMessage(
  							anthropic.NewTextBlock("Hello, world"),
  						),
  					},
  				},
  			},
  			{
  				CustomID: "my-second-request",
  				Params: anthropic.MessageBatchNewParamsRequestParams{
  					Model:     anthropic.ModelClaudeOpus5_5,
  					MaxTokens: 1024,
  					Messages: []anthropic.MessageParam{
  						anthropic.NewUserMessage(
  							anthropic.NewTextBlock("Hi again, friend"),
  						),
  					},
  				},
  			},
  		},
  	})

  fmt.Println(batch.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  BatchCreateParams params = BatchCreateParams.builder()
      .addRequest(
          BatchCreateParams.Request.builder()
              .customId("my-first-request")
              .params(
                  BatchCreateParams.Request.Params.builder()
                      .model(Model.CLAUDE_OPUS_5_5)
                      .maxTokens(1024)
                      .addUserMessage("Hello, world")
                      .build()
              )
              .build()
      )
      .addRequest(
          BatchCreateParams.Request.builder()
              .customId("my-second-request")
              .params(
                  BatchCreateParams.Request.Params.builder()
                      .model(Model.CLAUDE_OPUS_5_5)
                      .maxTokens(1024)
                      .addUserMessage("Hi again, friend")
                      .build()
              )
              .build()
      )
      .build();

  MessageBatch messageBatch = client.messages().batches().create(params);

  System.out.println(messageBatch);
  ```

  ```php PHP
  $client = new Client();

  $batch = $client->messages->batches->create(
      requests: [
          [
              'custom_id' => 'my-first-request',
              'params' => [
                  'model' => 'claude-opus-5-5',
                  'max_tokens' => 1024,
                  'messages' => [
                      ['role' => 'user', 'content' => 'Hello, world']
                  ]
              ]
          ],
          [
              'custom_id' => 'my-second-request',
              'params' => [
                  'model' => 'claude-opus-5-5',
                  'max_tokens' => 1024,
                  'messages' => [
                      ['role' => 'user', 'content' => 'Hi again, friend']
                  ]
              ]
          ]
      ],
  );

  echo $batch->id;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  batch = client.messages.batches.create(
    requests: [
      {
        custom_id: "my-first-request",
        params: {
          model: Anthropic::Model::CLAUDE_OPUS_5_5,
          max_tokens: 1024,
          messages: [
            { role: "user", content: "Hello, world" }
          ]
        }
      },
      {
        custom_id: "my-second-request",
        params: {
          model: Anthropic::Model::CLAUDE_OPUS_5_5,
          max_tokens: 1024,
          messages: [
            { role: "user", content: "Hi again, friend" }
          ]
        }
      }
    ]
  )

  puts batch
  ```
</CodeGroup>

在此示例中，两个独立的请求被一起批处理以进行异步处理。每个请求都有一个唯一的 `custom_id`，并包含您在 Messages API 调用中使用的标准参数。

<Tip>
  **使用 Messages API 测试您的批处理请求**

  每个消息请求的 `params` 对象的验证是异步执行的，验证错误会在整个批次处理结束后返回。您可以先使用 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 验证您的请求结构，以确保正确构建输入。
</Tip>

批次首次创建时，响应的处理状态为 `in_progress`。

```json Output
{
  "id": "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d",
  "type": "message_batch",
  "processing_status": "in_progress",
  "request_counts": {
    "processing": 2,
    "succeeded": 0,
    "errored": 0,
    "canceled": 0,
    "expired": 0
  },
  "ended_at": null,
  "created_at": "2024-09-24T18:37:24.100435Z",
  "expires_at": "2024-09-25T18:37:24.100435Z",
  "cancel_initiated_at": null,
  "results_url": null
}
```

### 跟踪您的批次

消息批次的 [`processing_status`](https://platform.claude.com/docs/zh-CN/api/messages/batches/retrieve#message_batch.processing_status) 字段指示批次所处的处理阶段。它最初为 `in_progress`，在批次中所有请求处理完毕且结果就绪后更新为 `ended`。您可以通过访问 [Console](https://platform.claude.com/settings/workspaces/default/batches) 或使用检索端点来监控批次的状态。

(See the `messages/batches/retrieve` method in [the API reference](https://platform.claude.com/docs/zh-CN/api/messages/batches/retrieve).)

#### 轮询消息批次完成情况

要轮询消息批次，您需要它的 `id`，该 `id` 在创建批次时的响应中提供，也可以通过列出批次获得。您可以实现一个轮询循环，定期检查批次状态，直到处理结束：

<CodeGroup>
  ```bash cURL
  # 检查状态；重复直到 processing_status 为 "ended"
  curl "https://api.anthropic.com/v1/messages/batches/$MESSAGE_BATCH_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 检查状态；重复执行，直到 processing_status 为 "ended"
  ant messages:batches retrieve --message-batch-id "$MESSAGE_BATCH_ID"
  ```

  ```python Python
  import time

  client = anthropic.Anthropic()

  MESSAGE_BATCH_ID = "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d"

  message_batch = None
  while True:
      message_batch = client.messages.batches.retrieve(MESSAGE_BATCH_ID)
      if message_batch.processing_status == "ended":
          break

      print(f"Batch {MESSAGE_BATCH_ID} is still processing...")
      time.sleep(60)
  print(message_batch)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const messageBatchId = "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d";

  let messageBatch;
  while (true) {
    messageBatch = await client.messages.batches.retrieve(messageBatchId);
    if (messageBatch.processing_status === "ended") {
      break;
    }

    console.log(`Batch ${messageBatchId} is still processing... waiting`);
    await new Promise((resolve) => setTimeout(resolve, 60_000));
  }
  console.log(messageBatch);
  ```

  ```csharp C#
  AnthropicClient client = new();
  string messageBatchId = Environment.GetEnvironmentVariable("MESSAGE_BATCH_ID");

  MessageBatch messageBatch = null;
  while (true)
  {
      messageBatch = await client.Messages.Batches.Retrieve(messageBatchId);
      if (messageBatch.ProcessingStatus == "ended")
      {
          break;
      }

      Console.WriteLine($"Batch {messageBatchId} is still processing...");
      await Task.Delay(60000);
  }
  Console.WriteLine(messageBatch);
  ```

  ```go Go
  client := anthropic.NewClient()
  messageBatchID := os.Getenv("MESSAGE_BATCH_ID")

  var messageBatch *anthropic.MessageBatch
  for {
  	var err error
  	messageBatch, err = client.Messages.Batches.Get(context.TODO(), messageBatchID, anthropic.MessageBatchGetParams{})
  	if err != nil {
  		log.Fatal(err)
  	}
  	if messageBatch.ProcessingStatus == "ended" {
  		break
  	}

  	fmt.Printf("Batch %s is still processing...\n", messageBatchID)
  	time.Sleep(60 * time.Second)
  }
  fmt.Println(messageBatch)
  ```

  ```java Java
  import com.anthropic.models.messages.batches.MessageBatch;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();
          String messageBatchId = "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d";

          MessageBatch messageBatch = null;
          while (true) {
              messageBatch = client.messages().batches().retrieve(messageBatchId);
              if (messageBatch.processingStatus().equals(MessageBatch.ProcessingStatus.ENDED)) {
                  break;
              }

              System.out.println("Batch " + messageBatchId + " is still processing...");
              Thread.sleep(60000);
          }
          System.out.println(messageBatch);
  ```

  ```php PHP
  $client = new Client();
  $messageBatchId = getenv('MESSAGE_BATCH_ID');

  $messageBatch = null;
  while (true) {
      $messageBatch = $client->messages->batches->retrieve(
          messageBatchID: $messageBatchId,
      );
      if ($messageBatch->processingStatus === 'ended') {
          break;
      }

      echo "Batch {$messageBatchId} is still processing...\n";
      sleep(60);
  }
  echo json_encode($messageBatch, JSON_PRETTY_PRINT);
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message_batch_id = ENV["MESSAGE_BATCH_ID"]
  message_batch = nil
  loop do
    message_batch = client.messages.batches.retrieve(message_batch_id)
    break if message_batch.processing_status == :ended

    puts "Batch #{message_batch_id} is still processing..."
    sleep 60
  end
  puts message_batch
  ```
</CodeGroup>

### 列出所有消息批次

您可以使用[列表端点](https://platform.claude.com/docs/zh-CN/api/messages/batches/list)列出工作区中的所有消息批次。该 API 支持分页，会根据需要自动获取更多页面：

<CodeGroup>
  ```bash cURL
  # 获取一页结果。当响应中的 has_more 为 true 时，将其
  # last_id 作为 after_id 传入以获取下一页。（SDK 和 CLI
  # 会自动执行分页。）
  curl -s "https://api.anthropic.com/v1/messages/batches?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 根据需要自动获取更多页面
  ant messages:batches list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 根据需要自动获取更多页面。
  for message_batch in client.messages.batches.list(limit=20):
      print(message_batch)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 根据需要自动获取更多页面。
  for await (const messageBatch of client.messages.batches.list({
    limit: 20
  })) {
    console.log(messageBatch);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new BatchListParams
  {
      Limit = 20
  };

  // 根据需要自动获取更多页面
  var page = await client.Messages.Batches.List(parameters);
  await foreach (var messageBatch in page.Paginate())
  {
      Console.WriteLine(messageBatch);
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  // 根据需要自动获取更多页面
  iter := client.Messages.Batches.ListAutoPaging(context.TODO(), anthropic.MessageBatchListParams{
  	Limit: anthropic.Int(20),
  })

  for iter.Next() {
  	messageBatch := iter.Current()
  	fmt.Println(messageBatch)
  }

  if err := iter.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  // 根据需要自动获取更多页面
  for (MessageBatch messageBatch : client
      .messages()
      .batches()
      .list(BatchListParams.builder()
          .limit(20)
          .build())
      .autoPager()) {
      System.out.println(messageBatch);
  }
  ```

  ```php PHP
  $client = new Client();

  // 根据需要自动获取更多页面
  foreach ($client->messages->batches->list(limit: 20)->pagingEachItem() as $messageBatch) {
      echo $messageBatch->id . "\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 根据需要自动获取更多页面
  client.messages.batches.list(limit: 20).auto_paging_each do |message_batch|
    puts message_batch
  end
  ```
</CodeGroup>

### 检索批次结果

批次处理结束后，批次中的每个 Messages 请求都会有一个结果。共有四种结果类型：

| 结果类型        | 描述                                           |
| ----------- | -------------------------------------------- |
| `succeeded` | 请求成功。包含消息结果。                                 |
| `errored`   | 请求遇到错误，未创建消息。可能的错误包括无效请求和内部服务器错误。这些请求不会向您收费。 |
| `canceled`  | 用户在此请求发送到模型之前取消了批次。这些请求不会向您收费。               |
| `expired`   | 批次在此请求发送到模型之前达到了 24 小时过期时限。这些请求不会向您收费。       |

批次的 `request_counts` 对象统计仍在处理中的请求以及每种结果类型的请求数量。

(See `message_batch.request_counts` on `messages/batches/retrieve` in [the API reference](https://platform.claude.com/docs/zh-CN/api/messages/batches/retrieve#message_batch.request_counts).)

批次结果可通过消息批次上的 [`results_url`](https://platform.claude.com/docs/zh-CN/api/messages/batches/retrieve#message_batch.results_url) 属性下载，如果组织权限允许，也可以在 Console 中下载。由于结果可能很大，建议[以流式方式获取结果](https://platform.claude.com/docs/zh-CN/api/messages/batches/results)，而不是一次性全部下载。

<CodeGroup>
  ```bash cURL
  # 以流式传输方式读取批处理的 .jsonl 结果，每行一个结果。
  # 如需逐条处理结果（重试、验证错误），请使用 Anthropic SDK。
  curl "https://api.anthropic.com/v1/messages/batches/msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d/results" \
    -H "anthropic-version: 2026-10-21" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  # 每行打印一个结果，例如 `{"custom_id":"test-1","result":{"type":"succeeded",…}}`。
  # 如需逐条处理结果（重试、验证错误），请使用其他选项卡中的 Anthropic SDK
  # 示例。
  ant messages:batches results \
    --message-batch-id msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d \
    --format jsonl
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 以节省内存的分块方式流式传输结果文件，每次处理一个
  for result in client.messages.batches.results(
      "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d",
  ):
      outcome = result.result
      match outcome.type:
          case "succeeded":
              print(f"Success! {result.custom_id}")
          case "errored":
              if outcome.error.error.type == "invalid_request_error":
                  # 重新发送请求前必须先修正请求体
                  print(f"Validation error {result.custom_id}")
              else:
                  # 可以直接重试请求
                  print(f"Server error {result.custom_id}")
          case "expired":
              print(f"Request expired {result.custom_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 以节省内存的分块方式流式读取结果文件，每次处理一个
  for await (const result of await client.messages.batches.results(
    "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d"
  )) {
    switch (result.result.type) {
      case "succeeded":
        console.log(`Success! ${result.custom_id}`);
        break;
      case "errored":
        if (result.result.error.error.type === "invalid_request_error") {
          // 重新发送请求前必须先修正请求体
          console.log(`Validation error: ${result.custom_id}`);
        } else {
          // 可以直接重试请求
          console.log(`Server error: ${result.custom_id}`);
        }
        break;
      case "expired":
        console.log(`Request expired: ${result.custom_id}`);
        break;
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  await foreach (var result in client.Messages.Batches.ResultsStreaming("msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d"))
  {
      switch (result.Result.Type.Raw())
      {
          case "succeeded":
              Console.WriteLine($"Success! {result.CustomID}");
              break;
          case "errored":
              if (result.Result.TryPickErrored(out var errored)
                  && errored.Error.Error.Type.Raw() == "invalid_request_error")
              {
                  Console.WriteLine($"Validation error: {result.CustomID}");
              }
              else
              {
                  Console.WriteLine($"Server error: {result.CustomID}");
              }
              break;
          case "expired":
              Console.WriteLine($"Request expired: {result.CustomID}");
              break;
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  stream := client.Messages.Batches.ResultsStreaming(context.TODO(), "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d", anthropic.MessageBatchResultsParams{})

  for stream.Next() {
  	result := stream.Current()

  	switch variant := result.Result.AsAny().(type) {
  	case anthropic.MessageBatchSucceededResult:
  		fmt.Printf("Success! %s\n", result.CustomID)
  	case anthropic.MessageBatchErroredResult:
  		fmt.Printf("Error: %s - %s\n", result.CustomID, variant.Error.Error.Message)
  	case anthropic.MessageBatchExpiredResult:
  		fmt.Printf("Request expired: %s\n", result.CustomID)
  	}
  }

  if err := stream.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.core.http.StreamResponse;
  import com.anthropic.models.messages.batches.BatchResultsParams;
  import com.anthropic.models.messages.batches.MessageBatchIndividualResponse;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          // 以节省内存的分块方式流式传输结果文件，每次处理一个
          try (
              StreamResponse<MessageBatchIndividualResponse> streamResponse = client
                  .messages()
                  .batches()
                  .resultsStreaming(
                      BatchResultsParams.builder()
                          .messageBatchId("msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d")
                          .build()
                  )
          ) {
              streamResponse
                  .stream()
                  .forEach(result -> {
                      switch (result.result().type().value()) {
                          case SUCCEEDED -> System.out.println("Success! " + result.customId());
                          case ERRORED -> {
                              if (result.result().asErrored().error().error().isInvalidRequestError()) {
                                  // 重新发送请求前必须先修正请求体
                                  System.out.println("Validation error: " + result.customId());
                              } else {
                                  // 可以直接重试该请求
                                  System.out.println("Server error: " + result.customId());
                              }
                          }
                          case EXPIRED -> System.out.println("Request expired: " + result.customId());
                      }
                  });
          }
  ```

  ```php PHP
  use Anthropic\Messages\Batches\MessageBatchErroredResult;
  use Anthropic\Messages\Batches\MessageBatchExpiredResult;
  use Anthropic\Messages\Batches\MessageBatchSucceededResult;

  $client = new Client();

  foreach ($client->messages->batches->resultsStream(messageBatchID: 'msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d') as $result) {
      switch (true) {
          case $result->result instanceof MessageBatchSucceededResult:
              echo "Success! {$result->customID}\n";
              break;
          case $result->result instanceof MessageBatchErroredResult:
              if ($result->result->error->error->type === 'invalid_request_error') {
                  echo "Validation error: {$result->customID}\n";
              } else {
                  echo "Server error: {$result->customID}\n";
              }
              break;
          case $result->result instanceof MessageBatchExpiredResult:
              echo "Request expired: {$result->customID}\n";
              break;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  client.messages.batches.results_streaming("msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d").each do |result|
    outcome = result.result
    case outcome
    when Anthropic::Models::Messages::MessageBatchSucceededResult
      puts "Success! #{result.custom_id}"
    when Anthropic::Models::Messages::MessageBatchErroredResult
      if outcome.error.error.type == :invalid_request_error
        puts "Validation error: #{result.custom_id}"
      else
        puts "Server error: #{result.custom_id}"
      end
    when Anthropic::Models::Messages::MessageBatchExpiredResult
      puts "Request expired: #{result.custom_id}"
    end
  end
  ```
</CodeGroup>

结果采用 `.jsonl` 格式，其中每一行都是一个有效的 JSON 对象，表示消息批次中单个请求的结果。对于每个流式传输的结果，您可以根据其 `custom_id` 和结果类型执行不同的操作。以下是一组示例结果：

```jsonl .jsonl file
{"custom_id":"my-second-request","result":{"type":"succeeded","message":{"id":"msg_014VwiXbi91y3JMjcpyGBHX5","type":"message","role":"assistant","model":"claude-opus-5-5","content":[{"type":"text","text":"Hello again! It's nice to see you. How can I assist you today? Is there anything specific you'd like to chat about or any questions you have?"}],"stop_reason":"end_turn","stop_sequence":null,"usage":{"input_tokens":11,"output_tokens":36}}}}
{"custom_id":"my-first-request","result":{"type":"succeeded","message":{"id":"msg_01FqfsLoHwgeFbguDgpz48m7","type":"message","role":"assistant","model":"claude-opus-5-5","content":[{"type":"text","text":"Hello! How can I assist you today? Feel free to ask me any questions or let me know if there's anything you'd like to chat about."}],"stop_reason":"end_turn","stop_sequence":null,"usage":{"input_tokens":10,"output_tokens":34}}}}
```

如果您的结果包含错误，其 `result.error` 将被设置为标准的[错误结构](https://platform.claude.com/docs/zh-CN/api/errors#error-shapes)。

批次结果使用创建该批次的请求所用的 API 版本。

<Tip>
  **批次结果可能与输入顺序不一致**

  批次结果可以以任意顺序返回，可能与创建批次时请求的顺序不一致。在前面的示例中，第二个批次请求的结果先于第一个返回。要将结果与其对应的请求正确匹配，请始终使用 `custom_id` 字段。
</Tip>

### 取消消息批次

您可以使用[取消端点](https://platform.claude.com/docs/zh-CN/api/messages/batches/cancel)取消当前正在处理的消息批次。取消后，批次的 `processing_status` 会立即变为 `canceling`。您可以使用前面描述的相同轮询技术来等待取消完成。已取消的批次最终状态为 `ended`，并且可能包含在取消之前已处理的请求的部分结果。

<CodeGroup>
  ```bash cURL
  curl -X POST https://api.anthropic.com/v1/messages/batches/$MESSAGE_BATCH_ID/cancel \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant messages:batches cancel --message-batch-id "$MESSAGE_BATCH_ID"
  ```

  ```python Python
  client = anthropic.Anthropic()

  MESSAGE_BATCH_ID = "msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d"

  message_batch = client.messages.batches.cancel(
      MESSAGE_BATCH_ID,
  )
  print(message_batch)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const messageBatch = await client.messages.batches.cancel(MESSAGE_BATCH_ID);
  console.log(messageBatch);
  ```

  ```csharp C#
  AnthropicClient client = new();
  string messageBatchId = Environment.GetEnvironmentVariable("MESSAGE_BATCH_ID");

  var messageBatch = await client.Messages.Batches.Cancel(messageBatchId);
  Console.WriteLine(messageBatch);
  ```

  ```go Go
  client := anthropic.NewClient()
  messageBatchID := os.Getenv("MESSAGE_BATCH_ID")

  messageBatch, err := client.Messages.Batches.Cancel(context.TODO(), messageBatchID, anthropic.MessageBatchCancelParams{})
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(messageBatch)
  ```

  ```java Java
  import com.anthropic.models.messages.batches.*;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageBatch messageBatch = client
              .messages()
              .batches()
              .cancel("msgbatch_01HkcTjaV5uDC8jWR4ZsDV8d");
          System.out.println(messageBatch);
  ```

  ```php PHP
  $client = new Client();

  $messageBatch = $client->messages->batches->cancel(
      messageBatchID: 'msgbatch_example_id',
  );
  echo $messageBatch;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message_batch_id = ENV.fetch("MESSAGE_BATCH_ID")
  message_batch = client.messages.batches.cancel(message_batch_id)
  puts message_batch
  ```
</CodeGroup>

响应显示批次处于 `canceling` 状态：

```json Output
{
  "id": "msgbatch_013Zva2CMHLNnXjNJJKqJ2EF",
  "type": "message_batch",
  "processing_status": "canceling",
  "request_counts": {
    "processing": 2,
    "succeeded": 0,
    "errored": 0,
    "canceled": 0,
    "expired": 0
  },
  "ended_at": null,
  "created_at": "2024-09-24T18:37:24.100435Z",
  "expires_at": "2024-09-25T18:37:24.100435Z",
  "cancel_initiated_at": "2024-09-24T18:39:03.114875Z",
  "results_url": null
}
```

### 在 Message Batches 中使用提示缓存

Message Batches API 支持提示缓存，可能帮助您降低批处理请求的成本和处理时间。提示缓存和 Message Batches 的价格折扣可以叠加，在同时使用这两项功能时可节省更多成本。但是，由于批处理请求是异步并发处理的，缓存命中是尽力而为的。用户通常会获得 30% 到 98% 的缓存命中率，具体取决于其流量模式。

要最大化批处理请求中缓存命中的可能性：

1. 在批次中的每个 Message 请求中包含相同的 `cache_control` 块。
2. 保持稳定的请求流，以防止缓存条目在 5 分钟生命周期后过期。
3. 构建您的请求，使其尽可能多地共享缓存内容。

在批次中实现提示缓存的示例：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages/batches \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "requests": [
        {
          "custom_id": "my-first-request",
          "params": {
            "model": "claude-opus-5-5",
            "max_tokens": 1024,
            "system": [
              {
                "type": "text",
                "text": "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
              },
              {
                "type": "text",
                "text": "<the entire contents of Pride and Prejudice>",
                "cache_control": {"type": "ephemeral"}
              }
            ],
            "messages": [
              {"role": "user", "content": "Analyze the major themes in Pride and Prejudice."}
            ]
          }
        },
        {
          "custom_id": "my-second-request",
          "params": {
            "model": "claude-opus-5-5",
            "max_tokens": 1024,
            "system": [
              {
                "type": "text",
                "text": "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
              },
              {
                "type": "text",
                "text": "<the entire contents of Pride and Prejudice>",
                "cache_control": {"type": "ephemeral"}
              }
            ],
            "messages": [
              {"role": "user", "content": "Write a summary of Pride and Prejudice."}
            ]
          }
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages:batches create <<'YAML'
  requests:
    - custom_id: my-first-request
      params:
        model: claude-opus-5-5
        max_tokens: 1024
        system:
          - type: text
            text: >
              You are an AI assistant tasked with analyzing literary works. Your
              goal is to provide insightful commentary on themes, characters, and
              writing style.
          - type: text
            text: "<the entire contents of Pride and Prejudice>"
            cache_control:
              type: ephemeral
        messages:
          - role: user
            content: Analyze the major themes in Pride and Prejudice.
    - custom_id: my-second-request
      params:
        model: claude-opus-5-5
        max_tokens: 1024
        system:
          - type: text
            text: >
              You are an AI assistant tasked with analyzing literary works. Your
              goal is to provide insightful commentary on themes, characters, and
              writing style.
          - type: text
            text: "<the entire contents of Pride and Prejudice>"
            cache_control:
              type: ephemeral
        messages:
          - role: user
            content: Write a summary of Pride and Prejudice.
  YAML
  ```

  ```python Python
  from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
  from anthropic.types.messages.batch_create_params import Request

  client = anthropic.Anthropic()

  message_batch = client.messages.batches.create(
      requests=[
          Request(
              custom_id="my-first-request",
              params=MessageCreateParamsNonStreaming(
                  model="claude-opus-5-5",
                  max_tokens=1024,
                  system=[
                      {
                          "type": "text",
                          "text": "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n",
                      },
                      {
                          "type": "text",
                          "text": "<the entire contents of Pride and Prejudice>",
                          "cache_control": {"type": "ephemeral"},
                      },
                  ],
                  messages=[
                      {
                          "role": "user",
                          "content": "Analyze the major themes in Pride and Prejudice.",
                      }
                  ],
              ),
          ),
          Request(
              custom_id="my-second-request",
              params=MessageCreateParamsNonStreaming(
                  model="claude-opus-5-5",
                  max_tokens=1024,
                  system=[
                      {
                          "type": "text",
                          "text": "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n",
                      },
                      {
                          "type": "text",
                          "text": "<the entire contents of Pride and Prejudice>",
                          "cache_control": {"type": "ephemeral"},
                      },
                  ],
                  messages=[
                      {
                          "role": "user",
                          "content": "Write a summary of Pride and Prejudice.",
                      }
                  ],
              ),
          ),
      ]
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const messageBatch = await client.messages.batches.create({
    requests: [
      {
        custom_id: "my-first-request",
        params: {
          model: "claude-opus-5-5",
          max_tokens: 1024,
          system: [
            {
              type: "text",
              text: "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
            },
            {
              type: "text",
              text: "<the entire contents of Pride and Prejudice>",
              cache_control: { type: "ephemeral" }
            }
          ],
          messages: [
            { role: "user", content: "Analyze the major themes in Pride and Prejudice." }
          ]
        }
      },
      {
        custom_id: "my-second-request",
        params: {
          model: "claude-opus-5-5",
          max_tokens: 1024,
          system: [
            {
              type: "text",
              text: "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
            },
            {
              type: "text",
              text: "<the entire contents of Pride and Prejudice>",
              cache_control: { type: "ephemeral" }
            }
          ],
          messages: [{ role: "user", content: "Write a summary of Pride and Prejudice." }]
        }
      }
    ]
  });
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Models.Messages;
  using Anthropic.Models.Messages.Batches;

  AnthropicClient client = new()
  {
      ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY")
  };

  var messageBatch = await client.Messages.Batches.Create(new BatchCreateParams
  {
      Requests =
      [
          new()
          {
              CustomID = "my-first-request",
              Params = new()
              {
                  Model = Model.ClaudeOpus5_5,
                  MaxTokens = 1024,
                  System = new List<TextBlockParam>
                  {
                      new()
                      {
                          Text = "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
                      },
                      new()
                      {
                          Text = "<the entire contents of Pride and Prejudice>",
                          CacheControl = new()
                      }
                  },
                  Messages =
                  [
                      new UserMessageParam { Content = "Analyze the major themes in Pride and Prejudice." }
                  ]
              }
          },
          new()
          {
              CustomID = "my-second-request",
              Params = new()
              {
                  Model = Model.ClaudeOpus5_5,
                  MaxTokens = 1024,
                  System = new List<TextBlockParam>
                  {
                      new()
                      {
                          Text = "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
                      },
                      new()
                      {
                          Text = "<the entire contents of Pride and Prejudice>",
                          CacheControl = new()
                      }
                  },
                  Messages =
                  [
                      new UserMessageParam { Content = "Write a summary of Pride and Prejudice." }
                  ]
              }
          }
      ]
  });
  ```

  ```go Go
  client := anthropic.NewClient()

  messageBatch, err := client.Messages.Batches.New(context.TODO(), anthropic.MessageBatchNewParams{
  	Requests: []anthropic.MessageBatchNewParamsRequest{
  		{
  			CustomID: "my-first-request",
  			Params: anthropic.MessageBatchNewParamsRequestParams{
  				Model:     anthropic.ModelClaudeOpus5_5,
  				MaxTokens: 1024,
  				System: []anthropic.TextBlockParam{
  					{
  						Text: "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n",
  					},
  					{
  						Text:         "<the entire contents of Pride and Prejudice>",
  						CacheControl: anthropic.NewCacheControlEphemeralParam(),
  					},
  				},
  				Messages: []anthropic.MessageParam{
  					anthropic.NewUserMessage(anthropic.NewTextBlock("Analyze the major themes in Pride and Prejudice.")),
  				},
  			},
  		},
  		{
  			CustomID: "my-second-request",
  			Params: anthropic.MessageBatchNewParamsRequestParams{
  				Model:     anthropic.ModelClaudeOpus5_5,
  				MaxTokens: 1024,
  				System: []anthropic.TextBlockParam{
  					{
  						Text: "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n",
  					},
  					{
  						Text:         "<the entire contents of Pride and Prejudice>",
  						CacheControl: anthropic.NewCacheControlEphemeralParam(),
  					},
  				},
  				Messages: []anthropic.MessageParam{
  					anthropic.NewUserMessage(anthropic.NewTextBlock("Write a summary of Pride and Prejudice.")),
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(messageBatch)
  ```

  ```java Java
  import com.anthropic.models.messages.CacheControlEphemeral;
  // ...
  import com.anthropic.models.messages.batches.*;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          BatchCreateParams createParams = BatchCreateParams.builder()
              .addRequest(
                  BatchCreateParams.Request.builder()
                      .customId("my-first-request")
                      .params(
                          BatchCreateParams.Request.Params.builder()
                              .model(Model.CLAUDE_OPUS_5_5)
                              .maxTokens(1024)
                              .systemOfTextBlockParams(
                                  List.of(
                                      TextBlockParam.builder()
                                          .text(
                                              "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
                                          )
                                          .build(),
                                      TextBlockParam.builder()
                                          .text("<the entire contents of Pride and Prejudice>")
                                          .cacheControl(CacheControlEphemeral.builder().build())
                                          .build()
                                  )
                              )
                              .addUserMessage("Analyze the major themes in Pride and Prejudice.")
                              .build()
                      )
                      .build()
              )
              .addRequest(
                  BatchCreateParams.Request.builder()
                      .customId("my-second-request")
                      .params(
                          BatchCreateParams.Request.Params.builder()
                              .model(Model.CLAUDE_OPUS_5_5)
                              .maxTokens(1024)
                              .systemOfTextBlockParams(
                                  List.of(
                                      TextBlockParam.builder()
                                          .text(
                                              "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
                                          )
                                          .build(),
                                      TextBlockParam.builder()
                                          .text("<the entire contents of Pride and Prejudice>")
                                          .cacheControl(CacheControlEphemeral.builder().build())
                                          .build()
                                  )
                              )
                              .addUserMessage("Write a summary of Pride and Prejudice.")
                              .build()
                      )
                      .build()
              )
              .build();

          MessageBatch messageBatch = client.messages().batches().create(createParams);
  ```

  ```php PHP
  $client = new Client();

  $messageBatch = $client->messages->batches->create(
      requests: [
          [
              'custom_id' => 'my-first-request',
              'params' => [
                  'model' => 'claude-opus-5-5',
                  'max_tokens' => 1024,
                  'system' => [
                      [
                          'type' => 'text',
                          'text' => 'You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n'
                      ],
                      [
                          'type' => 'text',
                          'text' => '<the entire contents of Pride and Prejudice>',
                          'cache_control' => ['type' => 'ephemeral']
                      ]
                  ],
                  'messages' => [
                      ['role' => 'user', 'content' => 'Analyze the major themes in Pride and Prejudice.']
                  ]
              ]
          ],
          [
              'custom_id' => 'my-second-request',
              'params' => [
                  'model' => 'claude-opus-5-5',
                  'max_tokens' => 1024,
                  'system' => [
                      [
                          'type' => 'text',
                          'text' => 'You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n'
                      ],
                      [
                          'type' => 'text',
                          'text' => '<the entire contents of Pride and Prejudice>',
                          'cache_control' => ['type' => 'ephemeral']
                      ]
                  ],
                  'messages' => [
                      ['role' => 'user', 'content' => 'Write a summary of Pride and Prejudice.']
                  ]
              ]
          ]
      ],
  );
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message_batch = client.messages.batches.create(
    requests: [
      {
        custom_id: "my-first-request",
        params: {
          model: Anthropic::Model::CLAUDE_OPUS_5_5,
          max_tokens: 1024,
          system: [
            {
              type: "text",
              text: "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
            },
            {
              type: "text",
              text: "<the entire contents of Pride and Prejudice>",
              cache_control: { type: "ephemeral" }
            }
          ],
          messages: [
            { role: "user", content: "Analyze the major themes in Pride and Prejudice." }
          ]
        }
      },
      {
        custom_id: "my-second-request",
        params: {
          model: Anthropic::Model::CLAUDE_OPUS_5_5,
          max_tokens: 1024,
          system: [
            {
              type: "text",
              text: "You are an AI assistant tasked with analyzing literary works. Your goal is to provide insightful commentary on themes, characters, and writing style.\n"
            },
            {
              type: "text",
              text: "<the entire contents of Pride and Prejudice>",
              cache_control: { type: "ephemeral" }
            }
          ],
          messages: [
            { role: "user", content: "Write a summary of Pride and Prejudice." }
          ]
        }
      }
    ]
  )
  ```
</CodeGroup>

在此示例中，批次中的两个请求都包含相同的系统消息以及用 `cache_control` 标记的《傲慢与偏见》全文，以提高缓存命中的可能性。

### 服务器工具与智能体循环

所有[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)（网络搜索、网页获取、代码执行、MCP 连接器、顾问和工具搜索）都可以在批处理请求中使用。批处理工作进程运行与同步 Messages API 相同的服务器端智能体循环。

由于无需维持开放连接，批处理循环在返回 `stop_reason: "pause_turn"` 之前，**单轮中可以运行比同步请求更多的迭代**。如果批次结果返回 `pause_turn`，则表示该轮未完成；您可以在后续请求（批处理或同步）中提交已暂停的助手内容来继续该轮，具体方式与 [pause\_turn 续接模式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#the-server-side-loop-and-pause-turn)中所示完全相同。

批处理工作进程还会针对每个组织对 `web_search` 进行限流，以避免高并发的批处理耗尽您组织的网络搜索速率限制。批处理会自动重试被限流的请求；您无需自行处理，但非常大的网络搜索批次可能需要更长时间才能完成。

### 扩展输出（beta）

`output-300k-2026-03-24` beta 标头将使用 Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 5.5、Claude Sonnet 5、Claude Sonnet 4.6 或 Claude Haiku 5.5 的批处理请求的 `max_tokens` 上限提高到 300,000。包含此标头即可在单轮中生成远超标准 128k `max_tokens` 限制的输出。

<Note>
  扩展输出仅在 Message Batches API 上可用，同步 Messages API 不支持。它在 Claude API 和 Claude Platform on AWS 上受支持，目前在 Amazon Bedrock、Claude Models on Google Cloud (Google managed) 或 Microsoft Foundry 上不可用。
</Note>

扩展输出适用于长篇内容生成，例如书籍篇幅的草稿和技术文档、详尽的结构化数据提取、大型代码生成脚手架以及长推理链。

单次 300k 令牌的生成可能需要超过一小时才能完成，因此请在规划批次提交时考虑 24 小时的处理窗口。适用标准批处理定价（标准 API 价格的 50%）。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages/batches \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: output-300k-2026-03-24" \
    -H "content-type: application/json" \
    -d '{
      "requests": [
        {
          "custom_id": "long-form-request",
          "params": {
            "model": "claude-opus-5-5",
            "max_tokens": 300000,
            "messages": [
              {"role": "user", "content": "Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices."}
            ]
          }
        }
      ]
    }'
  ```

  ```bash CLI
  ant beta:messages:batches create --beta output-300k-2026-03-24 <<'YAML'
  requests:
    - custom_id: long-form-request
      params:
        model: claude-opus-5-5
        max_tokens: 300000
        messages:
          - role: user
            content: >-
              Write a comprehensive technical guide to building distributed
              systems, covering architecture patterns, consistency models,
              fault tolerance, and operational best practices.
  YAML
  ```

  ```python Python
  from anthropic.types.beta.message_create_params import MessageCreateParamsNonStreaming
  from anthropic.types.beta.messages.batch_create_params import Request

  client = anthropic.Anthropic()

  message_batch = client.beta.messages.batches.create(
      betas=["output-300k-2026-03-24"],
      requests=[
          Request(
              custom_id="long-form-request",
              params=MessageCreateParamsNonStreaming(
                  model="claude-opus-5-5",
                  max_tokens=300_000,
                  messages=[
                      {
                          "role": "user",
                          "content": "Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices.",
                      }
                  ],
              ),
          ),
      ],
  )

  print(message_batch)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const messageBatch = await client.beta.messages.batches.create({
    betas: ["output-300k-2026-03-24"],
    requests: [
      {
        custom_id: "long-form-request",
        params: {
          model: "claude-opus-5-5",
          max_tokens: 300000,
          messages: [
            {
              role: "user",
              content:
                "Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices."
            }
          ]
        }
      }
    ]
  });

  console.log(messageBatch);
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Models.Beta.Messages;
  using Anthropic.Models.Beta.Messages.Batches;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  var batch = await client.Beta.Messages.Batches.Create(new BatchCreateParams
  {
      Betas = ["output-300k-2026-03-24"],
      Requests =
      [
          new()
          {
              CustomID = "long-form-request",
              Params = new()
              {
                  Model = Model.ClaudeOpus5_5,
                  MaxTokens = 300_000,
                  Messages =
                  [
                      new BetaUserMessageParam { Content = "Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices." }
                  ]
              }
          }
      ]
  });

  Console.WriteLine(batch);
  ```

  ```go Go
  client := anthropic.NewClient()

  batch, err := client.Beta.Messages.Batches.New(context.Background(),
  	anthropic.BetaMessageBatchNewParams{
  		Betas: []anthropic.AnthropicBeta{"output-300k-2026-03-24"},
  		Requests: []anthropic.BetaMessageBatchNewParamsRequest{
  			{
  				CustomID: "long-form-request",
  				Params: anthropic.BetaMessageBatchNewParamsRequestParams{
  					Model:     anthropic.ModelClaudeOpus5_5,
  					MaxTokens: 300_000,
  					Messages: []anthropic.BetaMessageParam{
  						anthropic.NewBetaUserMessage(
  							anthropic.NewBetaTextBlock("Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices."),
  						),
  					},
  				},
  			},
  		},
  	})
  if err != nil {
  	panic(err)
  }

  fmt.Println(batch.ID)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.batches.*;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      BatchCreateParams params = BatchCreateParams.builder()
          .addBeta("output-300k-2026-03-24")
          .addRequest(
              BatchCreateParams.Request.builder()
                  .customId("long-form-request")
                  .params(
                      BatchCreateParams.Request.Params.builder()
                          .model(Model.CLAUDE_OPUS_5_5)
                          .maxTokens(300_000L)
                          .addUserMessage("Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices.")
                          .build()
                  )
                  .build()
          )
          .build();

      BetaMessageBatch messageBatch = client.beta().messages().batches().create(params);

      IO.println(messageBatch);
  }
  ```

  ```php PHP
  $client = new Client();

  $batch = $client->beta->messages->batches->create(
      betas: ['output-300k-2026-03-24'],
      requests: [
          [
              'custom_id' => 'long-form-request',
              'params' => [
                  'model' => 'claude-opus-5-5',
                  'max_tokens' => 300_000,
                  'messages' => [
                      ['role' => 'user', 'content' => 'Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices.']
                  ]
              ]
          ]
      ],
  );

  echo $batch->id;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  batch = client.beta.messages.batches.create(
    betas: ["output-300k-2026-03-24"],
    requests: [
      {
        custom_id: "long-form-request",
        params: {
          model: Anthropic::Model::CLAUDE_OPUS_5_5,
          max_tokens: 300_000,
          messages: [
            { role: "user", content: "Write a comprehensive technical guide to building distributed systems, covering architecture patterns, consistency models, fault tolerance, and operational best practices." }
          ]
        }
      }
    ]
  )

  puts batch
  ```
</CodeGroup>

### 高效批处理的最佳实践

要有效使用 Batches API：

* 定期监控批次处理状态，并为失败的请求实现适当的重试逻辑。
* 使用有意义的 `custom_id` 值，以便轻松将结果与请求匹配，因为顺序无法保证。
* 考虑将非常大的数据集拆分为多个批次，以便更好地管理。
* 先使用 Messages API 试运行单个请求结构，以避免验证错误。

### 常见问题排查

如果遇到意外行为：

* 确认批处理请求的总大小不超过 256 MB。如果请求过大，您可能会收到 413 `request_too_large` 错误。
* 检查批次中的所有请求是否都使用了[支持的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#supported-models)。
* 确保批次中的每个请求都有唯一的 `custom_id`。
* 确保距离批次 `created_at`（而非处理 `ended_at`）时间不超过 29 天。如果已超过 29 天，结果将无法再查看。
* 确认批次未被取消。

请注意，批次中一个请求的失败不会影响其他请求的处理。

## 批次存储和隐私

* **工作区隔离：** 批次在创建它们的工作区内隔离。只有同一工作区中的 API 请求或在 Console 中有权查看工作区批次的用户才能访问它们。

* **结果可用性：** 批次结果在批次创建后 29 天内可用，为检索和处理留出充足的时间。

## 常见问题

<AccordionGroup>
  <Accordion title="批次处理需要多长时间？">
    批次处理最多可能需要 24 小时，但许多批次会更快完成。实际处理时间取决于批次的大小、当前需求和您的请求量。批次有可能过期且未在 24 小时内完成。
  </Accordion>

  <Accordion title="Batches API 是否适用于所有模型？">
    有关支持的模型列表，请参阅[支持的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#supported-models)。
  </Accordion>

  <Accordion title="我可以将 Message Batches API 与其他 API 功能一起使用吗？">
    可以，Message Batches API 支持 Messages API 中几乎所有可用的功能，包括大多数 beta 功能。少数参数（`stream`、`speed`、`fallbacks` 和 `max_tokens: 0`）不受支持。完整列表请参阅[可以批处理的内容](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#what-can-be-batched)。
  </Accordion>

  <Accordion title="Message Batches API 如何影响定价？">
    与标准 API 价格相比，Message Batches API 对所有用量提供 50% 的折扣。这适用于输入令牌、输出令牌和任何特殊令牌。有关定价的更多信息，请访问[定价](https://claude.com/pricing#anthropic-api)。
  </Accordion>

  <Accordion title="批次提交后可以更新吗？">
    不可以，批次一旦提交就无法修改。如果需要进行更改，请取消当前批次并提交新批次。请注意，取消可能不会立即生效。
  </Accordion>

  <Accordion title="Message Batches API 是否有速率限制？它们与 Messages API 速率限制是否相互影响？">
    除了对待处理请求数量的限制外，Message Batches API 还有基于 HTTP 请求的速率限制。请参阅 [Message Batches API 速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#message-batches-api)。使用 Batches API 不会影响 Messages API 中的速率限制。
  </Accordion>

  <Accordion title="如何处理批处理请求中的错误？">
    检索结果时，每个请求都有一个 `result` 字段，指示其是 `succeeded`、`errored`、`canceled` 还是 `expired`。对于 `errored` 结果，会提供额外的错误信息。请在 [API 参考](https://platform.claude.com/docs/zh-CN/api/messages/batches/create)中查看错误响应对象。
  </Accordion>

  <Accordion title="Message Batches API 如何处理隐私和数据隔离？">
    Message Batches API 在设计上具有强大的隐私和数据隔离措施：

    1. 批次及其结果在创建它们的工作区内隔离。这意味着只有同一工作区中的 API 请求才能访问它们。
    2. 批次中的每个请求都是独立处理的，请求之间不会发生数据泄露。
    3. 结果仅在有限时间内（29 天）可用，并遵循 Anthropic 的[数据保留政策](https://support.claude.com/en/articles/7996866-how-long-do-you-store-personal-data)。
    4. 可以在组织级别或按工作区禁用在 Console 中下载批次结果。
  </Accordion>

  <Accordion title="我可以在 Message Batches API 中使用提示缓存吗？">
    可以，可以在 Message Batches API 中使用提示缓存。但是，由于异步批处理请求可能会以任意顺序并发处理，缓存命中是尽力而为的。
  </Accordion>
</AccordionGroup>

## 可用性

批处理在所有模型上可用。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台                                                                                                                                                                                                                                                                                  |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                            | HIPAA |
| ---- | ---------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 不具备资格                                                                                                            | 不具备资格 |
| 详细信息 | 保留 29 天；需要异步存储。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#data-retention) | —     |

适用的端点：

* `/v1/messages/batches`

批处理会在批次创建后最多 29 天内存储请求和响应数据。您可以在处理完成后随时使用 [`DELETE /v1/messages/batches/{batch_id}`](https://platform.claude.com/docs/zh-CN/api/messages/batches/delete) 端点删除消息批次。要删除正在进行中的批次，请先取消它。异步处理需要在服务器端存储输入和输出，直到批次完成并检索结果为止。

## 后续步骤

<CardGroup cols={2}>
  <Card title="搜索结果" icon="magnifying-glass" href="https://platform.claude.com/docs/zh-CN/build-with-claude/search-results">
    通过提供带有来源归属的搜索结果，为 RAG 应用获取自然的引用。
  </Card>

  <Card title="提示缓存" icon="database" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching">
    通过缓存批次中各请求共享的提示前缀来降低成本和延迟。
  </Card>
</CardGroup>
