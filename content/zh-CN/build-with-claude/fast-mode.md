---
title: 快速模式
url: https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode
description: 在受支持的 Claude 模型上获得最高 2.5 倍的每秒输出令牌数。
featureMetadata:
  betaHeader: fast-mode-2026-02-01
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 使用相同的 Messages API 端点，推理速度更快。无论速度设置如何，ZDR 均适用。
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-opus-5-5
    - claude-opus-5
    - claude-opus-4-8
  supportedPlatforms:
    Claude API:
      availability: stable
      note: '移除日期：2026 年 11 月 30 日。移除后，向 `claude-opus-4-8` 发送的带有 `speed: "fast"` 的请求将返回错误。如需继续使用快速模式，请迁移到 Claude Opus 5.5。请参阅[从 Claude Opus 4.8 迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8)。'
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry:
      availability: stable
      note: 在 Microsoft Foundry 上，快速模式有自己的模型 ID，以 `-fast` 结尾。请将请求发送到快速模式模型的部署，并省略 `speed`。
  details:
    - 快速模式在 Claude Opus 5 上处于研究预览阶段，在 Claude Opus 4.8 上已弃用，且在这两个模型上均需要使用 `fast-mode-2026-02-01` beta 标头。
---

**API 支持**：用法因产品而异。

* **Messages API**：在请求中设置 `speed: "fast"`。
* **Managed Agents**：在智能体的模型对象上设置 `speed`，例如 `{"id": "claude-opus-5-5", "speed": "fast"}`。

“Fast mode”（快速模式）以溢价定价提供最高 2.5 倍的每秒输出令牌数。在请求中设置 `speed: "fast"` 即可选择启用。快速模式可在 Claude Fable 5.2、Claude Mythos 5.2 和 Claude Opus 5.5 上使用。快速模式在 Claude Opus 5 上处于研究预览阶段，在 Claude Opus 4.8 上已弃用。

## 快速模式的工作原理

快速模式使用更快的推理配置运行同一模型。智能水平和能力不会发生任何变化。

* 与标准速度相比，每秒输出令牌数最高可提升 2.5 倍
* 速度优势集中在 “output tokens per second”（每秒输出令牌数），即 OTPS，而非 “time to first token”（首个令牌生成时间），即 TTFT
* 与 “[streaming](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)”（流式传输）兼容，在流式传输中 OTPS 的提升最为明显

## 基本用法

快速模式仅适用于受支持的模型。有关详细信息（包括哪些模型需要 beta 标头），请参阅[可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "speed": "fast",
      "messages": [{
        "role": "user",
        "content": "Refactor this module to use dependency injection"
      }]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  speed: fast
  messages:
    - role: user
      content: Refactor this module to use dependency injection
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      speed="fast",
      messages=[
          {"role": "user", "content": "Refactor this module to use dependency injection"}
      ],
  )

  for block in response.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    speed: "fast",
    messages: [
      {
        role: "user",
        content: "Refactor this module to use dependency injection"
      }
    ]
  });

  const textBlock = response.content.find((block) => block.type === "text");
  console.log(textBlock?.text);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Speed = Speed.Fast,
      Messages = [
          new UserMessageParam { Content = "Refactor this module to use dependency injection" }
      ],
  });

  foreach (var block in response.Content)
  {
      if (block.TryPickText(out var textBlock))
      {
          Console.WriteLine(textBlock.Text);
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Speed:     anthropic.MessageNewParamsSpeedFast,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Refactor this module to use dependency injection")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  for _, block := range response.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .speed(MessageCreateParams.Speed.FAST)
          .addUserMessage("Refactor this module to use dependency injection")
          .build());

  response.content().stream()
      .flatMap(block -> block.text().stream())
      .forEach(textBlock -> IO.println(textBlock.text()));
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 4096,
      speed: 'fast',
      messages: [
          ['role' => 'user', 'content' => 'Refactor this module to use dependency injection'],
      ],
  );

  foreach ($response->content as $block) {
      if ($block instanceof \Anthropic\Messages\TextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    speed: "fast",
    messages: [{ role: "user", content: "Refactor this module to use dependency injection" }]
  )

  response.content.each do |block|
    puts block.text if block.is_a?(Anthropic::Models::TextBlock)
  end
  ```
</CodeGroup>

在 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上，快速模式有自己的模型 ID，速度由部署决定。请将快速模式请求发送到快速模式模型的部署，并省略 `speed`：

| 模型                | 快速模式模型 ID                |
| ----------------- | ------------------------ |
| Claude Fable 5.2  | `claude-fable-5-2-fast`  |
| Claude Mythos 5.2 | `claude-mythos-5-2-fast` |
| Claude Opus 5.5   | `claude-opus-5-5-fast`   |

在标准模型（例如 `claude-opus-5-5`）的部署上设置 `speed` 的请求会返回错误。

## 速率限制

快速模式具有专用的 “rate limit”（速率限制），与标准速率限制相互独立。当超出快速模式速率限制时，API 会返回 `429` 错误，并附带 `retry-after` 标头，指示何时会有可用容量。在 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#rate-limits) 上，速率限制和配额由 Google Cloud 管理。

Claude Fable 5.2、Claude Mythos 5.2 和 Claude Opus 5.5 具有默认的快速模式速率限制。该默认限制适用于 Start、Build 和 Scale [使用层级](https://platform.claude.com/docs/zh-CN/api/rate-limits)。您无需为这些模型申请快速模式访问权限。处于评估层级的组织在升级到 Start、Build 或 Scale 层级后，即可在这些模型上使用快速模式。

响应中包含指示快速模式速率限制状态的标头：

| 标头                                       | 描述               |
| ---------------------------------------- | ---------------- |
| `anthropic-fast-input-tokens-limit`      | 每分钟快速模式输入令牌的最大数量 |
| `anthropic-fast-input-tokens-remaining`  | 剩余的快速模式输入令牌数     |
| `anthropic-fast-input-tokens-reset`      | 快速模式输入令牌限制的重置时间  |
| `anthropic-fast-output-tokens-limit`     | 每分钟快速模式输出令牌的最大数量 |
| `anthropic-fast-output-tokens-remaining` | 剩余的快速模式输出令牌数     |
| `anthropic-fast-output-tokens-reset`     | 快速模式输出令牌限制的重置时间  |

要查看您的快速模式速率限制，请前往 Claude Console 中的[速率限制](https://platform.claude.com/usage/limits)页面。

## 检查所使用的速度

响应的 `usage` 对象包含一个 `speed` 字段，用于指示所使用的速度，值为 `"fast"` 或 `"standard"`。在[不支持快速模式的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)上请求 `speed: "fast"` 会返回错误，超出快速模式的速率限制或容量时同样会返回错误（`429` 或 `529`）。当带有 `speed: "fast"` 的请求成功时，`usage.speed` 为 `"fast"`。Claude Opus 4.6 是一个例外：带有 `speed: "fast"` 和 `fast-mode-2026-02-01` beta 标头的请求不会返回错误，而是会成功，但以标准速度运行。在这种情况下，`usage.speed` 显示为 `"standard"`。在 Claude Models on Google Cloud (Anthropic managed) 上，即使带有该标头，带有 `speed: "fast"` 的 Claude Opus 4.6 请求也会返回 `400` 错误。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "speed": "fast",
      "messages": [{"role": "user", "content": "Hello"}]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  speed: fast
  messages:
    - role: user
      content: Hello
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      speed="fast",
      messages=[{"role": "user", "content": "Hello"}],
  )

  print(response.usage.speed)  # "fast" or "standard"
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    speed: "fast",
    messages: [{ role: "user", content: "Hello" }]
  });

  console.log(response.usage.speed); // "fast" or "standard"
  ```

  ```csharp C#
  AnthropicClient client = new();

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Speed = Speed.Fast,
      Messages = [new UserMessageParam { Content = "Hello" }],
  });

  Console.WriteLine(response.Usage.Speed);  // "fast" or "standard"
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Speed:     anthropic.MessageNewParamsSpeedFast,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.Usage.Speed) // "fast" or "standard"
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .speed(MessageCreateParams.Speed.FAST)
      .addUserMessage("Hello")
      .build();

  Message response = client.messages().create(params);
  IO.println(response.usage().speed().orElseThrow());  // "fast" or "standard"
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      speed: 'fast',
      messages: [['role' => 'user', 'content' => 'Hello']],
  );

  echo $response->usage->speed;  // "fast" or "standard"
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    speed: "fast",
    messages: [{ role: "user", content: "Hello" }]
  )

  puts(response.usage.speed)  # "fast" or "standard"
  ```
</CodeGroup>

```json Output
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
// ...
  "usage": {
    "input_tokens": 8,
    "output_tokens": 12,
    "speed": "fast"
  }
}
```

要跟踪整个组织的快速模式使用情况和成本，请参阅[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)。

## 重试和回退

### 自动重试

默认情况下，SDK 会自动重试快速模式的 `429` 错误最多 2 次（可通过 `max_retries` (typescript, php: `maxRetries`; java: `.maxRetries()`; csharp: `MaxRetries`; go: `option.WithMaxRetries()`) 进行配置），每次重试前都会等待 `retry-after` 标头中指定的延迟时间。由于快速模式采用持续的令牌补充机制，`retry-after` 延迟通常很短，一旦有可用容量，请求即可成功。

### 回退到标准速度

如果您更希望回退到标准速度而不是等待快速模式容量，请捕获速率限制错误，并在不带 `speed: "fast"` 的情况下重试。在 Microsoft Foundry 中的 Claude 上，速度由部署决定，因此请改为在标准模型的部署上重试。在初始快速请求上将 `max_retries` (typescript, php: `maxRetries`; java: `.maxRetries()`; csharp: `MaxRetries`; go: `option.WithMaxRetries()`) 设置为 `0`，即可跳过自动重试，并在遇到速率限制错误时立即失败。

<Note>
  从快速速度回退到标准速度会导致[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)未命中。不同速度的请求不共享缓存前缀。
</Note>

由于将 `max_retries` (typescript, php: `maxRetries`; java: `.maxRetries()`; csharp: `MaxRetries`; go: `option.WithMaxRetries()`) 设置为 `0` 也会禁用其他瞬态错误（过载、内部服务器错误）的重试，因此以下示例会在遇到这些错误后重新发送请求。

<CodeGroup exclude="shell:cURL, shell:CLI">
  ```python Python
  client = anthropic.Anthropic()


  def create_message_with_fast_fallback(max_retries=0, max_attempts=3, **params):
      try:
          return client.with_options(max_retries=max_retries).messages.create(**params)
      except anthropic.RateLimitError:
          if params.get("speed") == "fast":
              del params["speed"]
              return create_message_with_fast_fallback(max_retries=max_retries, **params)
          raise
      except (
          anthropic.APIStatusError,
          anthropic.APIConnectionError,
      ) as error:
          if isinstance(error, anthropic.APIStatusError) and error.status_code < 500:
              raise
          if max_attempts > 1:
              return create_message_with_fast_fallback(
                  max_retries=max_retries, max_attempts=max_attempts - 1, **params
              )
          raise


  message = create_message_with_fast_fallback(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello"}],
      speed="fast",
      max_retries=0,
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  async function createMessageWithFastFallback(
    params: Anthropic.MessageCreateParamsNonStreaming,
    requestOptions?: Anthropic.RequestOptions,
    maxAttempts: number = 3
  ): Promise<Anthropic.Message> {
    try {
      return await client.messages.create(params, requestOptions);
    } catch (e) {
      if (e instanceof Anthropic.RateLimitError && params.speed === "fast") {
        const { speed, ...rest } = params;
        return createMessageWithFastFallback(rest);
      }
      if (
        e instanceof Anthropic.InternalServerError ||
        e instanceof Anthropic.APIConnectionError
      ) {
        if (maxAttempts > 1) {
          return createMessageWithFastFallback(params, undefined, maxAttempts - 1);
        }
      }
      throw e;
    }
  }

  const message = await createMessageWithFastFallback(
    {
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello" }],
      speed: "fast"
    },
    { maxRetries: 0 }
  );
  ```

  ```csharp C#
  AnthropicClient client = new();

  async Task<Message> CreateMessageWithFastFallback(
      MessageCreateParams parameters,
      int? maxRetries = null,
      int maxAttempts = 3)
  {
      try
      {
          var requestClient = maxRetries is int retries
              ? client.WithOptions(options => options with { MaxRetries = retries })
              : client;
          return await requestClient.Messages.Create(parameters);
      }
      catch (AnthropicRateLimitException)
      {
          if (parameters.Speed is not null)
          {
              return await CreateMessageWithFastFallback(
                  parameters with { Speed = null });
          }
          throw;
      }
      catch (Anthropic5xxException)
      {
          if (maxAttempts > 1)
          {
              return await CreateMessageWithFastFallback(
                  parameters, maxAttempts: maxAttempts - 1);
          }
          throw;
      }
  }

  var message = await CreateMessageWithFastFallback(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Hello" }],
          Speed = Speed.Fast,
      },
      maxRetries: 0);
  ```

  ```go Go
  func createMessageWithFastFallback(
  	ctx context.Context,
  	client *anthropic.Client,
  	params anthropic.MessageNewParams,
  	maxAttempts int,
  	opts ...option.RequestOption,
  ) (*anthropic.Message, error) {
  	message, err := client.Messages.New(ctx, params, opts...)
  	if err != nil {
  		var apierr *anthropic.Error
  		if errors.As(err, &apierr) && apierr.StatusCode == 429 && params.Speed != "" {
  			params.Speed = ""
  			return createMessageWithFastFallback(ctx, client, params, maxAttempts)
  		}
  		if (errors.As(err, &apierr) && apierr.StatusCode >= 500) || !errors.As(err, &apierr) {
  			if maxAttempts > 1 {
  				return createMessageWithFastFallback(ctx, client, params, maxAttempts-1)
  			}
  		}
  		return nil, err
  	}
  	return message, nil
  }

  func main() {
  	client := anthropic.NewClient()
  	message, err := createMessageWithFastFallback(
  		context.TODO(),
  		&client,
  		anthropic.MessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 1024,
  			Messages: []anthropic.MessageParam{
  				anthropic.NewUserMessage(anthropic.NewTextBlock("Hello")),
  			},
  			Speed: anthropic.MessageNewParamsSpeedFast,
  		},
  		3,
  		option.WithMaxRetries(0),
  	)
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println(message)
  }
  ```

  ```java Java
  import com.anthropic.errors.InternalServerException;
  import com.anthropic.errors.RateLimitException;
  // ...
  // 禁用 SDK 自动重试，由后续的回退逻辑处理
  AnthropicClient client =
      AnthropicOkHttpClient.builder()
          .fromEnv()
          .maxRetries(0)
          .build();

  Message createMessageWithFastFallback(
          MessageCreateParams params, int maxAttempts) {
      try {
          return client.messages().create(params);
      } catch (RateLimitException e) {
          if (params.speed().isPresent()) {
              MessageCreateParams retryParams = params.toBuilder()
                  .speed(Optional.empty())
                  .build();
              return createMessageWithFastFallback(retryParams, maxAttempts);
          }
          throw e;
      } catch (InternalServerException e) {
          if (maxAttempts > 1) {
              return createMessageWithFastFallback(params, maxAttempts - 1);
          }
          throw e;
      }
  }

  void main() {
      Message message = createMessageWithFastFallback(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024L)
              .addUserMessage("Hello")
              .speed(MessageCreateParams.Speed.FAST)
              .build(),
          3);
      message.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  use Anthropic\Core\Exceptions\APIConnectionException;
  use Anthropic\Core\Exceptions\InternalServerException;
  use Anthropic\Core\Exceptions\RateLimitException;
  use Anthropic\Messages\Model;
  use Anthropic\RequestOptions;

  $client = new Client();

  function createMessageWithFastFallback(
      Client $client,
      array $params,
      ?RequestOptions $requestOptions = null,
      int $maxAttempts = 3,
  ) {
      try {
          return $client->messages->create(
              ...$params,
              requestOptions: $requestOptions,
          );
      } catch (RateLimitException $e) {
          if (isset($params['speed'])) {
              unset($params['speed']);
              return createMessageWithFastFallback($client, $params);
          }
          throw $e;
      } catch (InternalServerException | APIConnectionException $e) {
          if ($maxAttempts > 1) {
              return createMessageWithFastFallback(
                  $client, $params, maxAttempts: $maxAttempts - 1
              );
          }
          throw $e;
      }
  }

  $message = createMessageWithFastFallback(
      $client,
      [
          'model' => Model::CLAUDE_OPUS_5_5,
          'maxTokens' => 1024,
          'messages' => [['role' => 'user', 'content' => 'Hello']],
          'speed' => 'fast',
      ],
      RequestOptions::with(maxRetries: 0),
  );
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  def create_message_with_fast_fallback(client, request_options: {}, max_attempts: 3, **params)
    client.messages.create(**params, request_options: request_options)
  rescue Anthropic::Errors::RateLimitError
    raise unless params[:speed] == "fast"
    params.delete(:speed)
    create_message_with_fast_fallback(client, **params)
  rescue Anthropic::Errors::InternalServerError, Anthropic::Errors::APIConnectionError
    raise unless max_attempts > 1
    create_message_with_fast_fallback(client, max_attempts: max_attempts - 1, **params)
  end

  message = create_message_with_fast_fallback(
    client,
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello" }],
    speed: "fast",
    request_options: { max_retries: 0 }
  )
  ```
</CodeGroup>

## 注意事项

* **Batch API：** 快速模式不适用于 [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)。
* **Priority Tier：** 快速模式不适用于 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers) 承诺。

## 定价

快速模式在整个 “context window”（上下文窗口）范围内按标准费率的倍数定价，包括输入令牌超过 200k 的请求。在 Claude Models on Google Cloud (Google managed) 上，快速模式价格由 Google Cloud 设定。请参阅[云平台定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)。

| 模型                              | 输入             | 输出             |
| ------------------------------- | -------------- | -------------- |
| Claude Fable 5.2                | $16 USD / MTok | $80 USD / MTok |
| Claude Mythos 5.2               | $16 USD / MTok | $80 USD / MTok |
| Claude Opus 5.5                 | $8 USD / MTok  | $40 USD / MTok |
| Claude Opus 5 / Claude Opus 4.8 | $10 USD / MTok | $50 USD / MTok |

快速模式定价可与其他定价调整因素叠加：

* [提示缓存倍数](https://platform.claude.com/docs/zh-CN/about-claude/pricing#prompt-caching)在快速模式定价的基础上叠加应用。
* [数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)倍数在快速模式定价的基础上叠加应用。

有关完整的定价详情，请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)页面。

## 可用性

快速模式在部分模型上可用或处于研究预览阶段或已弃用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

快速模式仅在以下模型上可用。如果您之前在较早的模型上使用快速模式，请[迁移到支持快速模式的模型](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide)。

要在 Claude Opus 5 上使用快速模式，请联系您的客户经理或[加入候补名单](https://claude.com/fast-mode)。

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：在请求中设置 `speed: "fast"`。

| 状态              | 云平台                                                                                                                                                                                                                                                                                                                                | 模型                                                                                                                                                                           |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用              | Claude API, [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) |
| 可用<sup>1</sup>  | [Microsoft Foundry (hosted on Anthropic)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                                                                                    | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) |
| 可用              | Claude API, [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)                                                                                                                                                          | [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)                                                                                           |
| 可用<sup>1</sup>  | [Microsoft Foundry (hosted on Azure)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                                                                                        | [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)                                                                                           |
| 研究预览            | Claude API                                                                                                                                                                                                                                                                                                                         | [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview)                                                                                               |
| 已弃用<sup>2</sup> | Claude API                                                                                                                                                                                                                                                                                                                         | [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                                                                                           |

1. 在 Microsoft Foundry 上，快速模式有自己的模型 ID，以 `-fast` 结尾。请将请求发送到快速模式模型的部署，并省略 `speed`。
2. 移除日期：2026 年 11 月 30 日。移除后，向 `claude-opus-4-8` 发送的带有 `speed: "fast"` 的请求将返回错误。如需继续使用快速模式，请迁移到 Claude Opus 5.5。请参阅[从 Claude Opus 4.8 迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8)。

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：在智能体的模型对象上设置 `speed`，例如 `{"id": "claude-opus-5-5", "speed": "fast"}`。

| 状态              | 云平台        | 模型                                                                                 |
| --------------- | ---------- | ---------------------------------------------------------------------------------- |
| 可用              | Claude API | [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview) |
| 研究预览            | Claude API | [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview)     |
| 已弃用<sup>1</sup> | Claude API | [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview) |

1. 移除日期：2026 年 11 月 30 日。移除后，向 `claude-opus-4-8` 发送的带有 `speed: "fast"` 的请求将返回错误。如需继续使用快速模式，请迁移到 Claude Opus 5.5。请参阅[从 Claude Opus 4.8 迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8)。

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

| 请求头                    | 模型                                                                                                                                                                 |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `fast-mode-2026-02-01` | [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格  |
| 详细信息 | 使用相同的 Messages API 端点，推理速度更快。无论速度设置如何，ZDR 均适用。                                                                                            | —     |

适用的端点：

* `/v1/messages` 使用 `speed: "fast"`

## 后续步骤

<CardGroup cols={2}>
  <Card title="结构化输出" icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs">
    从智能体工作流中获取经过验证的 JSON 结果。
  </Card>

  <Card title="定价" icon="calculator" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing">
    了解 Anthropic 针对模型和功能的定价结构。
  </Card>

  <Card title="Effort" icon="gauge" href="https://platform.claude.com/docs/zh-CN/build-with-claude/effort">
    使用 effort 参数控制 Claude 在响应时使用的令牌数量，在响应的详尽程度与令牌效率之间进行权衡。
  </Card>

  <Card title="流式传输消息" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/build-with-claude/streaming">
    使用服务器发送事件以增量方式流式传输 Messages API 响应，包括文本、工具使用和扩展思考的增量内容。
  </Card>
</CardGroup>
