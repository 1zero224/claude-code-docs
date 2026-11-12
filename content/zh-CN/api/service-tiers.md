---
title: 服务层级
url: https://platform.claude.com/docs/zh-CN/api/service-tiers
description: 不同的服务层级让您能够根据应用程序的需求，在可用性、性能和可预测的成本之间取得平衡。
featureMetadata:
  status: stable
  supportedModels:
    - claude-fable-5
    - claude-haiku-4-5-20251001
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API: stable
---

<Warning>
  服务层级属于旧版功能。
</Warning>

<Warning>
  Priority Tier（优先层级）容量承诺已不再提供购买。已有承诺的组织可以在合同到期日之前继续使用 Priority Tier，本页面将继续作为参考保留供其使用。如果您需要有保障的容量，请[联系销售](https://claude.com/contact-sales)。
</Warning>

Anthropic 提供三种服务层级：

* **Priority Tier（优先层级）：** 仅适用于已有容量承诺的组织
* **Standard（标准层级）：** 默认层级，适用于日常用例的试点和规模化扩展
* **Batch（批处理层级）：** 最适合可以等待、或能从脱离您的常规容量中获益的异步工作流

## 标准层级

标准层级是所有 API 请求的默认服务层级。API 会以尽力而为的可用性，将这些请求与所有其他请求一同进行优先级处理。

## Priority Tier

API 会将此层级中的请求优先于所有其他请求进行处理。这种优先处理有助于最大限度地减少["服务器过载"错误](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)，即使在高峰时段也是如此。

有关更多信息，请参阅[现有 Priority Tier 承诺](https://platform.claude.com/docs/zh-CN/api/service-tiers#existing-priority-tier-commitments)。

## 请求如何被分配层级

在处理请求时，Anthropic 会在以下情况下决定将请求分配到 Priority Tier：

* 您的组织拥有足够的 Priority Tier 容量每分钟**输入**令牌
* 您的组织拥有足够的 Priority Tier 容量每分钟**输出**令牌

Anthropic 按如下方式将用量计入 Priority Tier 容量：

**输入令牌**

* 缓存读取：从缓存中读取的每个令牌计为 0.1 个令牌
* 缓存写入：每写入 5 分钟 TTL 缓存 1 个令牌计为 1.25 个令牌
* 缓存写入：每写入 1 小时 TTL 缓存 1 个令牌计为 2.00 个令牌
* 对于在 Claude 4.6 及更高版本模型上的[仅限美国推理](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)（`inference_geo: "us"`）请求，每个输入令牌计为 1.1 个令牌
* 所有其他输入令牌均为每个令牌计为 1 个令牌

**输出令牌**

* 对于在 Claude 4.6 及更高版本模型上的[仅限美国推理](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)（`inference_geo: "us"`）请求，每个输出令牌计为 1.1 个令牌
* 所有其他输出令牌均为每个令牌计为 1 个令牌

否则，请求将按标准层级处理。

<Note>
  这些消耗率反映了每种令牌类型的相对定价。例如，仅限美国推理在 Claude 4.6 及更高版本模型上的定价为 1.1 倍，因此使用 `inference_geo: "us"` 消耗的每个令牌会从您的 Priority Tier 容量中扣除 1.1 个令牌。
</Note>

<Note>
  分配到 Priority Tier 的请求会同时占用 Priority Tier 容量和常规速率限制。 如果处理该请求会超出速率限制，则该请求将被拒绝。
</Note>

## 使用服务层级

您可以通过设置 `service_tier` 参数来控制请求可以使用哪些服务层级：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-4-8",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude!"}],
      "service_tier": "auto"
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-4-8
  max_tokens: 1024
  messages:
    - role: user
      content: Hello, Claude!
  service_tier: auto  # Automatically use Priority Tier when available, fall back to standard
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.messages.create(
      model="claude-opus-4-8",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude!"}],
      service_tier="auto",  # Automatically use Priority Tier when available, fall back to standard
  )
  print(message.usage.service_tier)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-opus-4-8",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude!" }],
    service_tier: "auto" // Automatically use Priority Tier when available, fall back to standard
  });
  console.log(message.usage.service_tier);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var message = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus4_8,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude!" }],
      ServiceTier = ServiceTier.Auto, // Automatically use Priority Tier when available, fall back to standard
  });
  Console.WriteLine(message.Usage.ServiceTier);
  ```

  ```go Go
  client := anthropic.NewClient()

  message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus4_8,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude!")),
  	},
  	// 在可用时自动使用 Priority Tier（优先级层级），否则回退到标准层级
  	ServiceTier: anthropic.MessageNewParamsServiceTierAuto,
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(message.Usage.ServiceTier)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_4_8)
      .maxTokens(1024L)
      .addUserMessage("Hello, Claude!")
      // 在可用时自动使用 Priority Tier，否则回退到标准层级
      .serviceTier(MessageCreateParams.ServiceTier.AUTO)
      .build();

  Message message = client.messages().create(params);
  IO.println(message.usage().serviceTier().orElseThrow());
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      model: Model::CLAUDE_OPUS_4_8,
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude!']],
      serviceTier: 'auto', // Automatically use Priority Tier when available, fall back to standard
  );
  echo $message->usage->serviceTier;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_4_8,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude!" }],
    service_tier: :auto # Automatically use Priority Tier when available, fall back to standard
  )
  puts(message.usage.service_tier)
  ```
</CodeGroup>

`service_tier` 参数接受以下值：

* `"auto"`（默认）- 如果可用则使用 Priority Tier 容量，否则回退到您的其他容量
* `"standard_only"` - 仅使用标准层级容量，适用于您不想使用 Priority Tier 容量的情况

响应的 `usage` 对象还包含分配给该请求的服务层级：

```json
{
  "usage": {
    "input_tokens": 410,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0,
    "output_tokens": 585,
    "service_tier": "priority"
  }
}
```

这使您能够确定请求被分配到了哪个服务层级。

当对具有 Priority Tier 承诺的模型请求 `service_tier="auto"` 时，以下响应头会提供相关信息：

```text wrap
anthropic-priority-input-tokens-limit: 10000
anthropic-priority-input-tokens-remaining: 9618
anthropic-priority-input-tokens-reset: 2025-01-12T23:11:59Z
anthropic-priority-output-tokens-limit: 10000
anthropic-priority-output-tokens-remaining: 6000
anthropic-priority-output-tokens-reset: 2025-01-12T23:12:21Z
```

您可以通过这些响应头是否存在来检测您的请求是否符合 Priority Tier 的条件，即使它已超出限制。

## 现有 Priority Tier 承诺

Priority Tier 承诺包括：

* 每分钟输入令牌数量
* 每分钟输出令牌数量
* 承诺期限（1、3、6 或 12 个月）
* 特定的模型版本

Priority Tier 的目标是借助优先分配的计算资源实现 99.5% 的正常运行时间。超出您承诺容量的请求会自动回退到标准层级。

### 支持的模型

除 Claude Fable 5.2、Claude Fable 5.1、Claude Haiku 5.5、Claude Mythos 5.2、Claude Mythos 5.1、Claude Mythos 5、[Claude Mythos Preview](https://anthropic.com/glasswing)、Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5 和 Claude Sonnet 5 之外，所有可用的 Claude 模型均支持 Priority Tier。

有关可用模型的更多详细信息，请查看[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)。

## 可用性

服务层级在部分模型上可用。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：需要 Priority Tier 承诺，该承诺已停止销售。拥有承诺时，请求默认使用 Priority Tier；将 `service_tier` 设置为 `"standard_only"` 可仅使用标准容量。

| 状态 | 云平台        | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 可用 | Claude API | [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/haiku-4-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview), [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview), [Claude Sonnet 4.5](https://platform.claude.com/docs/zh-CN/models/sonnet-4-5/overview) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用
