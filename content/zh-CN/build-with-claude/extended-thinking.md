---
title: 扩展思考
url: https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking
description: 在支持该功能的 Claude 模型上配置具有固定 budget_tokens 预算的手动扩展思考，并迁移到自适应思考。
featureMetadata:
  supportedModels:
    - claude-mythos-preview
    - claude-haiku-4-5-20251001
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

<Warning>
  扩展思考是一项旧版功能。

  扩展思考（`thinking.type: "enabled"` 搭配 `budget_tokens`）已弃用。Claude 4.7 及更高版本的模型不支持该功能，并会拒绝使用它的请求，返回 400 错误。

  * 推荐替代方案：[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost)
  * 迁移指南：[迁移到自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#migrating-to-adaptive-thinking)
</Warning>

手动模式下的 “extended thinking”（扩展思考）让您可以直接控制 Claude 的思考量。您在每个请求上通过 `thinking: {type: "enabled", budget_tokens: N}` 设置思考令牌预算，Claude 会在开始给出最终答案之前依据该预算进行思考。当您的工作负载需要可预测的延迟或对思考成本的精确控制时，手动模式仍然很有用。本页介绍如何设置和调整预算、手动模式如何与交错思考和 “prompt caching”（提示缓存）交互，以及如何迁移到自适应思考。

要了解思考本身的工作原理，包括思考块和响应结构、`display` 参数、"streaming"（流式传输）、结合 “tool use”（工具使用）的思考以及加密，请参阅[思考概述](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。

## 支持的模型

每个模型的扩展思考可用性（包括扩展思考是唯一模式的模型）列在[按模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#supported-models)中。

## 如何使用扩展思考

以下是在 Messages API 中使用扩展思考的示例：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-4-6",
      "max_tokens": 16000,
      "thinking": {
        "type": "enabled",
        "budget_tokens": 10000
      },
      "messages": [
        {
          "role": "user",
          "content": "Are there an infinite number of prime numbers such that n mod 4 == 3?"
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create --format yaml <<'YAML'
  model: claude-sonnet-4-6
  max_tokens: 16000
  thinking:
    type: enabled
    budget_tokens: 10000
  messages:
    - role: user
      content: Are there an infinite number of prime numbers such that n mod 4 == 3?
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-sonnet-4-6",
      max_tokens=16000,
      thinking={"type": "enabled", "budget_tokens": 10000},
      messages=[
          {
              "role": "user",
              "content": "Are there an infinite number of prime numbers such that n mod 4 == 3?",
          }
      ],
  )

  # 响应包含摘要化的思考块和文本块
  for block in response.content:
      match block.type:
          case "thinking":
              print(f"\nThinking summary: {block.thinking}")
          case "text":
              print(f"\nResponse: {block.text}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 16000,
    thinking: {
      type: "enabled",
      budget_tokens: 10000
    },
    messages: [
      {
        role: "user",
        content: "Are there an infinite number of prime numbers such that n mod 4 == 3?"
      }
    ]
  });

  // 响应包含摘要化的思考块和文本块
  for (const block of response.content) {
    switch (block.type) {
      case "thinking":
        console.log(`\nThinking summary: ${block.thinking}`);
        break;
      case "text":
        console.log(`\nResponse: ${block.text}`);
        break;
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var response = await client.Messages.Create(new()
  {
      Model = Model.ClaudeSonnet4_6,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigEnabled(budgetTokens: 10000),
      Messages =
      [
          new UserMessageParam
          {
              Content = "Are there an infinite number of prime numbers such that n mod 4 == 3?",
          },
      ],
  });

  // 响应包含摘要化的思考块和文本块
  foreach (var block in response.Content)
  {
      if (block.TryPickThinking(out var thinking))
      {
          Console.WriteLine($"\nThinking summary: {thinking.Thinking}");
      }
      else if (block.TryPickText(out var text))
      {
          Console.WriteLine($"\nResponse: {text.Text}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet4_6,
  	MaxTokens: 16000,
  	Thinking:  anthropic.ThinkingConfigParamOfEnabled(10000),
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Are there an infinite number of prime numbers such that n mod 4 == 3?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  // 响应包含摘要化的思考块和文本块
  for _, block := range response.Content {
  	switch block := block.AsAny().(type) {
  	case anthropic.ThinkingBlock:
  		fmt.Printf("\nThinking summary: %s", block.Thinking)
  	case anthropic.TextBlock:
  		fmt.Printf("\nResponse: %s", block.Text)
  	}
  }
  ```

  ```java Java
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      var client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_SONNET_4_6)
          .maxTokens(16_000)
          .enabledThinking(10_000)
          .addUserMessage("Are there an infinite number of prime numbers such that n mod 4 == 3?")
          .build();

      var response = client.messages().create(params);

      // 响应包含摘要化的思考块和文本块
      for (var block : response.content()) {
          block.thinking().ifPresent(thinkingBlock ->
              IO.println("\nThinking summary: " + thinkingBlock.thinking())
          );
          block.text().ifPresent(textBlock ->
              IO.println("\nResponse: " + textBlock.text())
          );
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: 'claude-sonnet-4-6',
      maxTokens: 16000,
      thinking: ['type' => 'enabled', 'budget_tokens' => 10000],
      messages: [
          [
              'role' => 'user',
              'content' => 'Are there an infinite number of prime numbers such that n mod 4 == 3?',
          ],
      ],
  );

  // 响应包含摘要化的思考块和文本块
  foreach ($response->content as $block) {
      echo match (true) {
          $block instanceof \Anthropic\Messages\ThinkingBlock => "\nThinking summary: {$block->thinking}",
          $block instanceof \Anthropic\Messages\TextBlock => "\nResponse: {$block->text}",
          default => '',
      };
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_4_6,
    max_tokens: 16_000,
    thinking: {
      type: :enabled,
      budget_tokens: 10_000
    },
    messages: [
      {
        role: :user,
        content: "Are there an infinite number of prime numbers such that n mod 4 == 3?"
      }
    ]
  )

  # 响应包含摘要化的思考块和文本块
  response.content.each do |block|
    case block
    when Anthropic::Models::ThinkingBlock
      puts "\nThinking summary: #{block.thinking}"
    when Anthropic::Models::TextBlock
      puts "\nResponse: #{block.text}"
    end
  end
  ```
</CodeGroup>

要开启手动扩展思考，请添加一个 `thinking` 对象，将 `type` 设置为 `enabled` 并提供 `budget_tokens` 值。

`budget_tokens` 参数设定了 Claude 可用于其内部推理过程的令牌数量目标。更大的预算可以让 Claude 对复杂问题进行更透彻的分析，从而提高响应质量。

## 预算规则与调优

`budget_tokens` 必须满足以下约束：

* **最小值为 1,024 个令牌。** API 会拒绝更小的值。
* **小于 `max_tokens`。** 思考令牌计入该轮次的 `max_tokens` 限制，因此预算必须为最终响应留出空间。唯一的例外是[手动模式下的交错思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#interleaved-thinking)，此时 `budget_tokens` 可以超过 `max_tokens`，因为预算涵盖一个助手轮次内的所有思考块。
* **不支持缓存预热。** 由于 `budget_tokens` 必须小于 `max_tokens`，扩展思考不能与 `max_tokens: 0`（[缓存预热](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#pre-warming-the-cache)）结合使用。

预算是一个目标，而不是严格的上限。实际令牌用量因任务而异，Claude 可能在预算耗尽之前很早就停止推理；`max_tokens` 仍然是总输出的硬性上限。

在 Claude Opus 4.5（唯一支持 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 的仅扩展思考模型）上，effort 决定整体响应，而 `budget_tokens` 设定思考深度；请同时设置两者。

要调整预算：

* 根据任务选择起点。对于简单任务，从接近 1,024 令牌的最小值开始，逐步增加，以找到适合您用例的最佳范围。对于复杂任务，从 16,000 令牌或更大的预算开始，并根据您对延迟和质量的需求进行调整。更高的预算可以实现更全面的推理，但收益递减的程度取决于任务，且会增加延迟。对于关键任务，请测试不同的设置以找到合适的平衡。
* 对于超过 32k 的思考预算，请使用[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)以避免网络问题。推动模型思考超过 32k 个令牌会产生长时间运行的请求，可能触及系统超时和开放连接限制。

要跟踪预算的实际成本，请监控响应中的 `usage.output_tokens_details.thinking_tokens` 字段，该字段报告计费输出令牌中有多少属于内部推理。在流式传输时，此明细仅出现在最终的 `message_delta` 事件中。

当您准备好不再使用手动预算时，请参阅[迁移到自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#migrating-to-adaptive-thinking)。

## 手动模式下的交错思考

交错思考让 Claude 能够在单个助手轮次内的工具调用之间进行思考，在决定下一步操作之前对每个工具结果进行推理。有关其概念、轮次结构以及在自适应思考模型上的行为，请参阅思考概述中的[交错思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#interleaved-thinking)。本节介绍在使用手动 `type: "enabled"` 思考时如何启用它。

在 Claude Opus 4.5、Claude Sonnet 4.5（已弃用）以及更早的 Claude 4 模型上，请在 API 请求中添加 `interleaved-thinking-2025-05-14` [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers)。

4.6 代模型在手动模式下有所分化：

* **Claude Sonnet 4.6**：beta 头与手动 `type: "enabled"` 配合使用仍然有效，但已弃用。建议使用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)，它无需头即可自动交错。
* **Claude Opus 4.6**：手动模式完全没有交错思考。只有其自适应模式会交错，因此如果您需要在此模型上于工具调用之间进行推理，请切换到 `thinking: {type: "adaptive"}`。

Claude Haiku 4.5 不支持交错思考。在 Claude API 上，beta 头会被接受但被忽略。

手动模式下交错思考的另外两个注意事项：

* 此处 `budget_tokens` 可以超过 `max_tokens`；[预算规则](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#budget-rules-and-tuning)解释了这一例外。
* 交错思考仅支持[通过 Messages API 使用的工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)。

不同平台对 beta 标头的处理方式有所不同。Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 和 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 在任何模型上都接受 `interleaved-thinking-2025-05-14`，并在不支持的情况下忽略它。接受并不等同于生效：在拒绝 `type: "enabled"` 的模型（4.7 及更高版本）或不支持手动模式交错的模型（Claude Opus 4.6）上，该标头不会产生手动模式效果；在这些模型上，adaptive thinking（自适应思考）会自动进行交错。

由合作伙伴运营的平台，即 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)，同样会在任何模型上接受该标头而不返回错误，并在不支持交错思考的模型上忽略该标头。

## 手动模式下的轮次结构

通用的轮次结构规则，包括单轮次工具使用循环、轮次中途冲突处理以及在轮次之间切换思考，请参阅[结合工具使用的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-with-tool-use)。

手动模式增加了一项要求：启用思考的请求中，最后一个助手轮次必须以思考块开头（[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)取消了这一要求）。在轮次之间更改思考配置也会使提示缓存失效；请参阅下一节。

## 手动模式下的提示缓存

在[思考与提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-prompt-caching)中描述的与模式无关的缓存行为之上，手动模式增加了一条规则：在请求之间更改 `budget_tokens` 会使缓存断点失效，就像切换思考模式一样，因为预算值会被渲染到提示中。预算更改后，消息级断点总是会未命中；工具和系统提示断点是否也会未命中取决于模型在何处渲染该配置。

在实践中，请选择一个预算并在缓存对话的整个生命周期内保持稳定。在 Claude Sonnet 4.6 上运行带有消息级缓存的多轮对话，并在第三个请求中将预算从 4,000 更改为 8,000 个令牌，可以直接展示失效情况：

```text Output wrap
First request - establishing cache
First response usage: { cache_creation_input_tokens: 1370, cache_read_input_tokens: 0, input_tokens: 17, output_tokens: 700 }

Second request - same thinking parameters (cache hit expected)
Second response usage: { cache_creation_input_tokens: 0, cache_read_input_tokens: 1370, input_tokens: 303, output_tokens: 874 }

Third request - different thinking budget (cache miss expected)
Third response usage: { cache_creation_input_tokens: 1370, cache_read_input_tokens: 0, input_tokens: 747, output_tokens: 619 }
```

第三个请求重新创建了缓存（`cache_creation_input_tokens=1370`，`cache_read_input_tokens=0`），因为预算在请求之间发生了变化。有关自适应模式下同一实验的可运行版本（其中 effort 级别扮演此处 `budget_tokens` 所扮演的缓存角色），请参阅引导页面上的[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#prompt-caching)。

## 共享机制

大多数思考行为与模式无关，并在[思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)页面上统一记录。那里的所有内容同样适用于手动模式：

* [控制思考显示](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)
* [流式传输思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#streaming-thinking)
* [结合工具使用的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-with-tool-use)，包括[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)
* [思考与提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-prompt-caching)
* [思考与上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-the-context-window)
* [思考加密](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-encryption)
* [定价](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#pricing)（位于[引导思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost)页面）

## 迁移到自适应思考

如果您的模型仅支持扩展思考（Claude Sonnet 4.5（已弃用）、Claude Opus 4.5、Claude Haiku 4.5 以及更早的 Claude 4 模型），目前无需采取任何操作：这些模型不支持自适应思考，`type: "adaptive"` [会返回 400 错误](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-type-adaptive)。请继续使用 `budget_tokens`，直到您迁移到支持自适应思考的模型，然后应用下面的映射。

在以下情况下，您需要从 `type: "enabled"` 迁移：

* 您使用的是 Claude Opus 4.6 或 Claude Sonnet 4.6，这些模型上的 `budget_tokens` 已弃用。
* 您使用版本号为 4.7 或更高的模型，在这些模型上 `type: "enabled"` 会返回 400 错误。例如 Claude Opus 5.5、Claude Sonnet 5、Claude Sonnet 5.5、Claude Fable 5.2、Claude Fable 5.1 和 Claude Haiku 5.5。

映射很简单：移除 `budget_tokens`，设置 `thinking: {type: "adaptive"}`，并使用 `output_config: {effort: ...}` 而非令牌预算来控制推理深度。

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 16000,
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000
  }
}
```

变为：

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 16000,
  "thinking": {
    "type": "adaptive"
  },
  "output_config": {
    "effort": "high"
  }
}
```

`effort: "high"` 是 Claude Sonnet 4.6 上的默认值，因此在此处省略它会产生相同的行为；这里写出它只是为了展示自适应模式下深度控制的位置。Claude Opus 5.5 和 Claude Haiku 5.5 默认为 `"medium"`，其他支持 effort 的模型默认为 `"high"`（请参阅 [Effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-levels)）。

请预期行为上的差异，而不仅仅是语法上的变化。使用固定预算时，Claude 会在每个请求上进行思考。使用自适应思考时，Claude 会决定每个请求是否思考以及思考多少，并且在较低的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 设置下，对于简单输入可能会完全跳过思考。迁移后，您还可以移除 `interleaved-thinking-2025-05-14` beta 标头：自适应思考会自动交错，且 Claude API 在这些模型上会忽略该标头。思考块保留方式也会变化：Claude Opus 4.5 以及版本号为 4.6 及更高的模型会在上下文中保留先前轮次的思考块并将其作为输入计费，而 Claude Sonnet 4.5（已弃用）、Claude Haiku 4.5 以及更早的模型会将其剥离；请参阅[按模型划分的思考块保留](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)。

切换模式属于思考配置更改，因此切换后的第一个请求会使缓存断点失效，如[手动模式下的提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#extended-thinking-with-prompt-caching)中所述。

有关完整指南，请参阅[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)、[effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 以及[模型迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。

## 后续步骤

<CardGroup cols={3}>
  <Card title="思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    了解思考的工作原理：思考块、显示、流式传输和工具使用。
  </Card>

  <Card title="引导思考" icon="compass" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost">
    让 Claude 在每个请求上决定何时思考以及思考多少。
  </Card>

  <Card title="工具和多轮工作流中的思考" icon="wrench" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-tool-workflows">
    保留思考块，并在工具调用和轮次之间管理思考。
  </Card>
</CardGroup>

## 可用性

扩展思考在部分模型上可用或已弃用。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态  | 模型                                                                                                                                                                                                                                                                                      |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用  | Claude Mythos Preview, [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/haiku-4-5/overview), [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview), [Claude Sonnet 4.5](https://platform.claude.com/docs/zh-CN/models/sonnet-4-5/overview) |
| 已弃用 | [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview)                                                                                                              |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用
* 用法：在 Claude 4.5 模型上，Managed Agents 会自行以您无法更改的固定预算开启手动思考。在更新的模型上，它使用自适应思考。 请改用[Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)。
