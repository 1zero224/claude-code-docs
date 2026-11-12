---
title: 思考
url: https://platform.claude.com/docs/zh-CN/build-with-claude/thinking
description: 了解 Claude 的思考机制：如何开启思考、读取思考输出、通过 effort 控制思考深度，以及将思考与工具、缓存和流式传输结合使用。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements).
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

**API 支持**：用法因产品而异。

* **Messages API**：默认开启。在接受该值的模型上，`thinking: {"type": "disabled"}` 会将其关闭。在默认关闭的模型上，设置 `thinking` 即可将其开启。
* **Managed Agents**：始终开启。由 Managed Agents 设置思考配置；智能体定义中没有 `thinking` 字段。

一次性作答的模型必须第一次就把所有事情做对：没有草稿，没有检查，也无法中途改变方法。对于证明、棘手的 bug 或漫长的智能体任务，第一种方法往往不是最好的。

“Thinking”（思考）消除了这一限制。当思考处于活动状态时，Claude 会在回答之前用自己的语言梳理问题：它会重述所问的内容、尝试不同方法、检查中间结果，并放弃站不住脚的路径。这种预先思考以 `thinking` 内容块的形式出现在响应之前，Claude 会借助它生成最终答案。这就是为什么思考能提升复杂任务的表现，例如数学、编程、分析和长时间运行的智能体工作——在这些任务中，答案的质量取决于中间过程，否则这些过程会被压缩进响应本身或被跳过。

思考是有成本的：Claude 用于推理的令牌按输出令牌计费，即使思考文本没有返回给您也是如此，并且它们与响应文本一起计入 `max_tokens`。本页介绍思考在整个 API 中的行为方式：如何开启思考、读取其输出，以及管理它与工具、流式传输、缓存和上下文窗口之间的交互。

## 思考的工作原理

<Frame>
  ![图表：Claude 决定是否预先思考，可能在工具调用之间思考，并在文本之前返回思考块](https://platform.claude.com/docs/zh-CN/images/how-thinking-works.svg)
</Frame>

Claude 是否会针对某个请求进行思考，以及思考的深度，取决于您的思考配置和请求的复杂程度。

以下是预先思考在响应中的样子：一个或多个 `thinking` 内容块出现在 `text` 块之前。思考块与其后的 `text` 块一样，仍然是生成的内容，但它与正式响应是分开的。每个思考块还带有一个 `signature` 字段，这是完整推理的加密副本，您需要在多轮对话和工具使用对话中原样传回（请参阅[思考加密](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-encryption)）：

```json
{
  "content": [
    {
      "type": "thinking",
      "thinking": "Let me break this down. The question has two parts, so I'll start with the simpler one and use its result to constrain the second...",
      "signature": "WaUjzkypQ2mUEVM36O2Txu...."
    },
    {
      "type": "text",
      "text": "Based on my analysis..."
    }
  ]
}
```

您并不总能看到这段文本，而且您看到的也从来不是原始的思维链：思考块中的文本是 [Claude 推理的摘要](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#summarized-thinking)。思考配置中的 `display` 字段控制是否返回该摘要：`"summarized"` 会返回摘要，而 `"omitted"`（许多模型上的默认值）返回的思考块中 `thinking` 字段为空。无论哪种方式，该块的计费方式相同，在多轮对话中的传回方式也相同。有关各模型的默认值和详细信息，请参阅[控制思考显示](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)。

如果 Claude 使用工具，思考也可能出现在工具调用之间。请参阅[思考与工具使用](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-with-tool-use)。有关完整的响应格式，请参阅 [Messages API 参考](https://platform.claude.com/docs/zh-CN/api/messages/create)。

## 配置思考

在大多数模型上，思考默认开启，或者只需一个参数即可开启。下表展示了每个模型对请求可发送的每种 `thinking` 值的处理方式。400 错误表示 API 拒绝该请求，[思考故障排除](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting)列出了每种错误及其修复方法。

| 模型                    | 无 `thinking` 字段 | `"adaptive"` | `"enabled"` 配合 `budget_tokens` | `"between_tools"`          | `"disabled"`             |
| --------------------- | --------------- | ------------ | ------------------------------ | -------------------------- | ------------------------ |
| Claude Opus 5.5       | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 400 错误                   |
| Claude Sonnet 5.5     | 自适应思考           | 自适应思考        | 400 错误                         | 在 `high` effort 或以下时关闭预先思考 | 400 错误                   |
| Claude Haiku 5.5      | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 在 `high` effort 或以下时关闭思考 |
| Claude Fable 5.2      | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 400 错误                   |
| Claude Mythos 5.2     | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 400 错误                   |
| Claude Fable 5.1      | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 400 错误                   |
| Claude Mythos 5.1     | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 400 错误                   |
| Claude Fable 5        | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 400 错误                   |
| Claude Mythos 5       | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 400 错误                   |
| Claude Opus 5         | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 在 `high` effort 或以下时关闭思考 |
| Claude Sonnet 5       | 自适应思考           | 自适应思考        | 400 错误                         | 400 错误                     | 关闭思考                     |
| Claude Opus 4.8       | 关闭思考            | 自适应思考        | 400 错误                         | 400 错误                     | 关闭思考                     |
| Claude Opus 4.7       | 关闭思考            | 自适应思考        | 400 错误                         | 400 错误                     | 关闭思考                     |
| Claude Mythos Preview | 自适应思考           | 自适应思考        | 扩展思考                           | 400 错误                     | 400 错误                   |
| Claude Opus 4.6       | 关闭思考            | 自适应思考        | 扩展思考（已弃用）                      | 400 错误                     | 关闭思考                     |
| Claude Sonnet 4.6     | 关闭思考            | 自适应思考        | 扩展思考（已弃用）                      | 400 错误                     | 关闭思考                     |
| Claude Opus 4.5       | 关闭思考            | 400 错误       | 扩展思考                           | 400 错误                     | 关闭思考                     |
| Claude Sonnet 4.5     | 关闭思考            | 400 错误       | 扩展思考                           | 400 错误                     | 关闭思考                     |
| Claude Haiku 4.5      | 关闭思考            | 400 错误       | 扩展思考                           | 400 错误                     | 关闭思考                     |

表中，"在 `high` effort 或以下"表示该请求在 `low`、`medium` 和 `high` effort 下有效，而在 `xhigh` 或 `max` 下返回 400 错误。

在 Claude 5 及更高版本的模型和 Claude Mythos Preview 上，思考已经开启，无需任何配置。在这些模型上，`display` 默认为 `"omitted"`，因此思考文本会被隐藏，除非您选择启用。使用 `thinking: {"type": "adaptive", "display": "summarized"}` 启用，如以下示例所示。

在 Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6 和 Claude Sonnet 4.6 上，思考默认关闭，直到您设置 `thinking: {type: "adaptive"}`，这会让 Claude 根据请求决定何时思考以及思考的深度。同样的请求，只需替换[模型字符串](https://platform.claude.com/docs/zh-CN/models/overview)，即可在这些模型上开启思考。以下示例设置了 `display: "summarized"` 以使思考文本可见，并使用了较大的 `max_tokens`：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 16000,
      "thinking": {
        "type": "adaptive",
        "display": "summarized"
      },
      "messages": [
        {
          "role": "user",
          "content": "What is the greatest common divisor of 1071 and 462?"
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 16000 \
    --thinking '{type: adaptive, display: summarized}' \
    --message '{role: user, content: "What is the greatest common divisor of 1071 and 462?"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=16000,
      thinking={"type": "adaptive", "display": "summarized"},
      messages=[
          {
              "role": "user",
              "content": "What is the greatest common divisor of 1071 and 462?",
          }
      ],
  )

  for block in response.content:
      match block.type:
          case "thinking":
              print(f"\nThinking: {block.thinking}")
          case "text":
              print(f"\nResponse: {block.text}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 16000,
    thinking: {
      type: "adaptive",
      display: "summarized"
    },
    messages: [
      {
        role: "user",
        content: "What is the greatest common divisor of 1071 and 462?"
      }
    ]
  });

  for (const block of response.content) {
    switch (block.type) {
      case "thinking":
        console.log(`\nThinking: ${block.thinking}`);
        break;
      case "text":
        console.log(`\nResponse: ${block.text}`);
        break;
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigAdaptive { Display = Display.Summarized },
      Messages = [
          new UserMessageParam { Content = "What is the greatest common divisor of 1071 and 462?" }
      ]
  };

  var message = await client.Messages.Create(parameters);

  foreach (var block in message.Content)
  {
      if (block.TryPickThinking(out ThinkingBlock? thinking))
      {
          Console.WriteLine($"\nThinking: {thinking.Thinking}");
      }
      else if (block.TryPickText(out TextBlock? text))
      {
          Console.WriteLine($"\nResponse: {text.Text}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{
  			Display: anthropic.ThinkingConfigAdaptiveDisplaySummarized,
  		},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is the greatest common divisor of 1071 and 462?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  for _, block := range response.Content {
  	switch v := block.AsAny().(type) {
  	case anthropic.ThinkingBlock:
  		fmt.Printf("\nThinking: %s", v.Thinking)
  	case anthropic.TextBlock:
  		fmt.Printf("\nResponse: %s", v.Text)
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.messages.ThinkingConfigAdaptive;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(16000L)
          .thinking(ThinkingConfigAdaptive.builder()
              .display(ThinkingConfigAdaptive.Display.SUMMARIZED)
              .build())
          .addUserMessage("What is the greatest common divisor of 1071 and 462?")
          .build();

      Message response = client.messages().create(params);

      response.content().forEach(block -> {
          block.thinking().ifPresent(thinkingBlock ->
              IO.println("\nThinking: " + thinkingBlock.thinking())
          );
          block.text().ifPresent(textBlock ->
              IO.println("\nResponse: " + textBlock.text())
          );
      });
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;
  use Anthropic\Messages\TextBlock;
  use Anthropic\Messages\ThinkingBlock;

  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 16000,
      messages: [
          [
              'role' => 'user',
              'content' => 'What is the greatest common divisor of 1071 and 462?'
          ]
      ],
      model: Model::CLAUDE_OPUS_5_5,
      thinking: ['type' => 'adaptive', 'display' => 'summarized'],
  );

  foreach ($message->content as $block) {
      switch (true) {
          case $block instanceof ThinkingBlock:
              echo "\nThinking: " . $block->thinking;
              break;
          case $block instanceof TextBlock:
              echo "\nResponse: " . $block->text;
              break;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 16000,
    thinking: {
      type: "adaptive",
      # 像这样的普通哈希使用 display:。带类型的 ThinkingConfigAdaptive 类
      # 将其写作 display_（带尾随下划线），以避免遮蔽 Ruby 的 Kernel#display。
      # 请求中发送的仍然是 display。
      display: "summarized"
    },
    messages: [
      {
        role: "user",
        content: "What is the greatest common divisor of 1071 and 462?"
      }
    ]
  )

  message.content.each do |block|
    case block
    when Anthropic::Models::ThinkingBlock
      puts "\nThinking: #{block.thinking}"
    when Anthropic::Models::TextBlock
      puts "\nResponse: #{block.text}"
    end
  end
  ```
</CodeGroup>

响应先包含摘要形式的思考，然后是答案：

```text Output wrap
Thinking: Use Euclidean algorithm.
1071 = 2*462 + 147
462 = 3*147 + 21
147 = 7*21 + 0
GCD = 21

Response: ## Finding GCD of 1071 and 462

I'll use the **Euclidean algorithm**, repeatedly dividing and taking remainders...
```

思考令牌计入 `max_tokens`，因此请将其设置得足够高，为思考和响应文本都留出空间。请参阅引导页面上的[成本控制](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#cost-control)以及[思考与上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-the-context-window)。

### 关闭思考

在思考默认开启的 Claude Sonnet 5 上，您可以将其关闭：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5",
      "max_tokens": 4096,
      "thinking": {"type": "disabled"},
      "messages": [
        {
          "role": "user",
          "content": "Summarize this article in one sentence."
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-sonnet-5 \
    --max-tokens 4096 \
    --thinking '{type: disabled}' \
    --message '{role: user, content: "Summarize this article in one sentence."}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-sonnet-5",
      max_tokens=4096,
      thinking={"type": "disabled"},
      messages=[{"role": "user", "content": "Summarize this article in one sentence."}],
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-sonnet-5",
    max_tokens: 4096,
    thinking: { type: "disabled" },
    messages: [{ role: "user", content: "Summarize this article in one sentence." }]
  });
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5,
      MaxTokens = 4096,
      Thinking = new ThinkingConfigDisabled(),
      Messages = [new UserMessageParam { Content = "Summarize this article in one sentence." }]
  };

  var message = await client.Messages.Create(parameters);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5,
  	MaxTokens: 4096,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfDisabled: &anthropic.ThinkingConfigDisabledParam{},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Summarize this article in one sentence.")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.messages.ThinkingConfigDisabled;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_SONNET_5)
          .maxTokens(4096L)
          .thinking(ThinkingConfigDisabled.builder().build())
          .addUserMessage("Summarize this article in one sentence.")
          .build();

      Message response = client.messages().create(params);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => 'Summarize this article in one sentence.'
          ]
      ],
      model: Model::CLAUDE_SONNET_5,
      thinking: ['type' => 'disabled'],
  );
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5,
    max_tokens: 4096,
    thinking: { type: "disabled" },
    messages: [
      {
        role: "user",
        content: "Summarize this article in one sentence."
      }
    ]
  )
  ```
</CodeGroup>

Claude Opus 5 同样默认开启思考，并在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)（努力程度）为 `high` 或以下时接受 `thinking: {type: "disabled"}`。在 `xhigh` 或 `max` effort 下，无法关闭思考：将 `thinking: {type: "disabled"}` 与这些 effort 级别组合的请求会返回 400 错误。此限制针对每个请求强制执行。在关闭思考的情况下，Claude Opus 5 偶尔可能会以纯文本形式输出工具调用，或在其可见输出中包含内部 XML 标签。有关提示层面的缓解措施，请参阅[在关闭思考的情况下运行](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5#running-with-thinking-disabled)。

Claude Sonnet 5.5 同样默认开启思考，并且会以 400 错误拒绝 `thinking: {type: "disabled"}`。要关闭预先思考，请改为发送 `thinking: {type: "between_tools"}`。这是 Claude Sonnet 5.5 上最低的思考设置，在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 为 `high` 或以下时被接受。模型仍会返回其[工具调用之间的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)。在没有工具的情况下，响应只包含文本，就像 Claude Sonnet 5 上的 `disabled` 一样。有关提示指导，请参阅[在不进行预先思考的情况下运行](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#running-without-up-front-thinking)。

Claude Haiku 5.5 同样默认开启思考，并在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 为 `high` 或以下时接受 `thinking: {type: "disabled"}`。在 `xhigh` 或 `max` effort 下，该组合会返回 400 错误。若要减少思考，请先降低 effort 级别。在关闭思考的情况下，当您同时请求 JSON 输出时，模型可能会跳过它需要的工具调用。在思考关闭时，与当前生效级别不同的[按消息设置的 `output_config.effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) 也会返回 400 错误。请参阅[使用 effort 控制思考](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5#use-effort-to-control-thinking)和[将自适应思考与 JSON 输出及您自己的工具一起使用](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5#json-output-with-your-own-tools)。

Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Opus 5.5 和 Claude Mythos Preview 会拒绝 `thinking: {type: "disabled"}`。在这些模型上无法关闭思考。

要在发送请求之前检查模型是否接受 `"disabled"` 或 `"between_tools"`，请从 Models API 读取其 `capabilities.thinking.types.disabled.supported` 或 `capabilities.thinking.types.between_tools.supported` 值。[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api) 介绍了这两个字段。

如果您的模型仅支持扩展思考（请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#supported-models)），请改用 `type: "enabled"` 和 `budget_tokens` 值进行配置。[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)页面介绍了该配置。如果任何思考配置返回 400 错误，[思考故障排除](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting)会将每条错误消息与其修复方法对应起来。

## 读取思考输出

### 控制思考显示

思考配置中的 `display` 字段控制思考内容在 API 响应中的返回方式。`display` 在两种模式下都有效：将其与 `type: "adaptive"` 或 `type: "enabled"` 一起设置。它接受以下值：

* `"summarized"`：思考块包含[摘要形式的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#summarized-thinking)文本，即 Claude 推理的可读摘要。这是 Claude Opus 4.6、Claude Sonnet 4.6 及更早模型上的默认值。
* `"omitted"`：返回的思考块中 `thinking` 字段为空。`signature` 字段仍携带加密的完整思考，以保持多轮对话的连续性（请参阅[思考加密](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-encryption)）。这是 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5、Claude Sonnet 5、Claude Opus 4.8、Claude Opus 4.7、[Claude Mythos Preview](https://anthropic.com/glasswing) 和 Claude Haiku 5.5 上的默认值。
* `"updates"`（beta）：返回的推理块中 `thinking` 字段为空，与 `"omitted"` 相同，而某些模型在工具调用之间编写的简短[进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)则以可读文本形式返回。需要 beta 标头 `thinking-display-updates-2026-08-18`。

当您的应用程序不向用户展示思考内容时，请设置 `display: "omitted"`。其主要好处是在流式传输时更快地获得首个文本令牌：服务器完全跳过流式传输思考令牌，只传送签名，因此最终的文本响应会更早开始流式传输。

使用 `display: "omitted"` 时，响应包含 `thinking` 字段为空的 `thinking` 块：

```json Output
{
  "content": [
    {
      "type": "thinking",
      "thinking": "",
      "signature": "EosnCkYICxIMMb3LzNrMu..."
    },
    {
      "type": "text",
      "text": "The answer is 12,231."
    }
  ]
}
```

使用省略的思考时，请注意以下几点：

* 您仍需为完整的思考令牌付费。省略降低的是延迟，而不是成本。
* 如果您在多轮对话中传回思考块，请原样传回。服务器会解密 `signature` 以重建原始思考用于构建提示（请参阅[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)）。您在往返传递的省略块的 `thinking` 字段中放置的任何文本都会被忽略。
* `display` 与 `thinking.type: "disabled"`（没有可显示的内容）以及 `thinking.type: "between_tools"`（仅限 Claude Sonnet 5.5，不接受任何其他字段）一起使用时无效。使用 `between_tools` 时，进度更新仍会附带摘要文本返回，就像在 `display: "updates"` 下一样（请参阅[进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)）。
* 当您使用 `thinking.type: "adaptive"` 且模型针对简单请求跳过思考时，无论 `display` 如何设置，都不会生成思考块。
* 在使用 `display: "omitted"` 进行流式传输时，不会流式传输任何思考文本。每个思考块会流式传输一个 `thinking` 字符串为空的 `thinking_delta`，然后是其 `signature_delta`。使用 `display: "updates"` 时，只有[进度更新块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)会流式传输携带文本的 `thinking_delta` 事件。有关事件顺序，请参阅[流式传输思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#streaming-thinking)。

<Note>
  无论您设置哪个 `display` 值，`signature` 字段都是相同的。支持在对话的不同轮次之间切换 `display` 值。
</Note>

### 摘要形式的思考

当 `display` 为 `"summarized"` 时，您收到的思考文本是 Claude 完整思考过程的摘要，而不是原始思维链。摘要形式的思考在防止滥用的同时，提供了思考带来的全部智能优势。没有任何 `display` 设置会返回原始思维链。

使用摘要形式的思考时，请注意以下几点：

* 您需要为原始请求生成的完整思考令牌付费，而不是为摘要令牌付费。计费的输出令牌数与您在响应中看到的令牌数并不一致。
* 在 Claude Opus 4.6、Claude Sonnet 4.6 及更早的模型上，思考输出的前几行会更加详细，提供对提示工程特别有帮助的详细推理。[Claude Mythos Preview](https://anthropic.com/glasswing) 从第一个令牌就开始进行摘要，因此其思考块不会显示这段详细的开头。
* 摘要在保留 Claude 思考过程关键思路的同时，只增加极少的延迟，因此摘要可以在生成时即时流式传输。
* 摘要由与您在请求中指定的模型不同的另一个模型处理。思考模型不会收到摘要输出。

<Note>
  在极少数需要访问完整思考输出的情况下，请[联系 Anthropic 销售团队](mailto:sales@anthropic.com)。
</Note>

要查看模型的推理，请读取 `thinking` 块，而不是在响应文本中通过提示索取推理。在 Claude Fable 5.2、Claude Fable 5.1、Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5 和 Claude Fable 5 上，试图将模型的内部推理作为响应文本的一部分引出的请求可能会被拒绝，并返回 `stop_details.category: "reasoning_extraction"`。有关字段参考，请参阅[拒绝的样子](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)；有关应改为请求的内容，请参阅[将推理保留在思考块中](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#keep-reasoning-in-thinking-blocks)。

### 流式传输思考

思考支持[流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)。思考块以 `content_block_delta` 事件中的 `thinking_delta` 事件形式流式传输，并在该块的 `content_block_stop` 之前紧跟一个 `signature_delta` 事件。文本块随后照常流式传输。

<Frame>
  ![思考流：块打开，仅当 display 返回文本时增量才携带文本，一个签名增量关闭该块，然后是文本](https://platform.claude.com/docs/zh-CN/images/how-thinking-streams.svg)
</Frame>

以下示例使用自适应思考流式传输响应，并在思考和文本增量到达时将其打印出来：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 16000,
      "stream": true,
      "thinking": {
        "type": "adaptive",
        "display": "summarized"
      },
      "messages": [
        {
          "role": "user",
          "content": "What is the greatest common divisor of 1071 and 462?"
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 16000 \
    --thinking '{type: adaptive, display: summarized}' \
    --message '{role: user, content: "What is the greatest common divisor of 1071 and 462?"}' \
    --stream \
    --format jsonl
  ```

  ```python Python
  client = anthropic.Anthropic()

  with client.messages.stream(
      model="claude-opus-5-5",
      max_tokens=16000,
      thinking={"type": "adaptive", "display": "summarized"},
      messages=[
          {
              "role": "user",
              "content": "What is the greatest common divisor of 1071 and 462?",
          }
      ],
  ) as stream:
      for event in stream:
          match event.type:
              case "content_block_start":
                  print(f"\nStarting {event.content_block.type} block...")
              case "content_block_delta":
                  delta = event.delta
                  match delta.type:
                      case "thinking_delta":
                          print(delta.thinking, end="", flush=True)
                      case "text_delta":
                          print(delta.text, end="", flush=True)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const stream = client.messages.stream({
    model: "claude-opus-5-5",
    max_tokens: 16000,
    thinking: { type: "adaptive", display: "summarized" },
    messages: [{ role: "user", content: "What is the greatest common divisor of 1071 and 462?" }]
  });

  for await (const event of stream) {
    switch (event.type) {
      case "content_block_start":
        console.log(`\nStarting ${event.content_block.type} block...`);
        break;
      case "content_block_delta":
        switch (event.delta.type) {
          case "thinking_delta":
            process.stdout.write(event.delta.thinking);
            break;
          case "text_delta":
            process.stdout.write(event.delta.text);
            break;
        }
        break;
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigAdaptive { Display = Display.Summarized },
      Messages = [new UserMessageParam { Content = "What is the greatest common divisor of 1071 and 462?" }]
  };

  await foreach (var rawEvent in client.Messages.CreateStreaming(parameters))
  {
      if (rawEvent.TryPickContentBlockStart(out var start))
      {
          Console.WriteLine($"\nStarting {start.ContentBlock.Type.Raw()} block...");
      }
      else if (rawEvent.TryPickContentBlockDelta(out var delta))
      {
          if (delta.Delta.TryPickThinking(out var thinkingDelta))
          {
              Console.Write(thinkingDelta.Thinking);
          }
          else if (delta.Delta.TryPickText(out var textDelta))
          {
              Console.Write(textDelta.Text);
          }
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{
  			Display: anthropic.ThinkingConfigAdaptiveDisplaySummarized,
  		},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is the greatest common divisor of 1071 and 462?")),
  	},
  })

  for stream.Next() {
  	event := stream.Current()
  	switch eventVariant := event.AsAny().(type) {
  	case anthropic.ContentBlockStartEvent:
  		fmt.Printf("\nStarting %s block...\n", eventVariant.ContentBlock.Type)
  	case anthropic.ContentBlockDeltaEvent:
  		switch deltaVariant := eventVariant.Delta.AsAny().(type) {
  		case anthropic.ThinkingDelta:
  			fmt.Print(deltaVariant.Thinking)
  		case anthropic.TextDelta:
  			fmt.Print(deltaVariant.Text)
  		}
  	}
  }
  if err := stream.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.messages.ThinkingConfigAdaptive;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(16000L)
          .thinking(ThinkingConfigAdaptive.builder()
              .display(ThinkingConfigAdaptive.Display.SUMMARIZED)
              .build())
          .addUserMessage("What is the greatest common divisor of 1071 and 462?")
          .build();

      try (var streamResponse = client.messages().createStreaming(params)) {
          streamResponse.stream().forEach(event -> {
              switch (event.type().value()) {
                  case CONTENT_BLOCK_START -> {
                      var startEvent = event.asContentBlockStart();
                      var block = startEvent.contentBlock();
                      switch (block.type().value()) {
                          case THINKING -> IO.println("\nStarting thinking block...");
                          case TEXT -> IO.println("\nStarting text block...");
                      }
                  }
                  case CONTENT_BLOCK_DELTA -> {
                      var deltaEvent = event.asContentBlockDelta();
                      deltaEvent.delta().thinking().ifPresent(thinkingDelta ->
                          IO.print(thinkingDelta.thinking())
                      );
                      deltaEvent.delta().text().ifPresent(textDelta ->
                          IO.print(textDelta.text())
                      );
                  }
              }
          });
      }
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;
  use Anthropic\Messages\RawContentBlockDeltaEvent;
  use Anthropic\Messages\RawContentBlockStartEvent;
  use Anthropic\Messages\TextDelta;
  use Anthropic\Messages\ThinkingDelta;

  $client = new Client();

  $stream = $client->messages->createStream(
      maxTokens: 16000,
      messages: [
          ['role' => 'user', 'content' => 'What is the greatest common divisor of 1071 and 462?']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      thinking: ['type' => 'adaptive', 'display' => 'summarized'],
  );

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof RawContentBlockStartEvent:
              echo "\nStarting {$event->contentBlock->type} block...\n";
              break;
          case $event instanceof RawContentBlockDeltaEvent:
              switch (true) {
                  case $event->delta instanceof ThinkingDelta:
                      echo $event->delta->thinking;
                      break;
                  case $event->delta instanceof TextDelta:
                      echo $event->delta->text;
                      break;
              }
              break;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  stream = client.messages.stream(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 16000,
    # 像这样的普通哈希使用 display:。类型化的 ThinkingConfigAdaptive 类
    # 将其写作 display_（末尾带下划线），以避免遮蔽 Ruby 的 Kernel#display。
    # 请求中发送的仍然是 display。
    thinking: { type: "adaptive", display: "summarized" },
    messages: [
      { role: "user", content: "What is the greatest common divisor of 1071 and 462?" }
    ]
  )

  stream.each do |event|
    case event
    when Anthropic::Streaming::ThinkingEvent
      print event.thinking
    when Anthropic::Streaming::TextEvent
      print event.text
    end
  end
  ```
</CodeGroup>

要在流式传输后重新组装带有签名的完整思考块，请使用 SDK 的消息累积辅助方法 `stream.get_final_message()` (typescript: `stream.finalMessage()`; ruby: `stream.accumulated_message`; csharp: `.Aggregate()`; go: `message.Accumulate(event)`; java, php: `MessageAccumulator`)，而不是自己拼接增量。

<Accordion title="完整的流式事件跟踪">
  ```sse Output
  event: message_start
  data: {"type": "message_start", "message": {"id": "msg_01...", "type": "message", "role": "assistant", "content": [], "model": "claude-opus-5-5", "stop_reason": null, "stop_sequence": null}}

  event: content_block_start
  data: {"type": "content_block_start", "index": 0, "content_block": {"type": "thinking", "thinking": "", "signature": ""}}

  event: content_block_delta
  data: {"type": "content_block_delta", "index": 0, "delta": {"type": "thinking_delta", "thinking": "I need to find the GCD of 1071 and 462 using the Euclidean algorithm.\n\n1071 = 2 × 462 + 147"}}

  event: content_block_delta
  data: {"type": "content_block_delta", "index": 0, "delta": {"type": "thinking_delta", "thinking": "\n462 = 3 × 147 + 21\n147 = 7 × 21 + 0\n\nSo GCD(1071, 462) = 21"}}

  // Additional thinking deltas...

  event: content_block_delta
  data: {"type": "content_block_delta", "index": 0, "delta": {"type": "signature_delta", "signature": "EqQBCgIYAhIM1gbcDa9GJwZA2b..."}}

  event: content_block_stop
  data: {"type": "content_block_stop", "index": 0}

  event: content_block_start
  data: {"type": "content_block_start", "index": 1, "content_block": {"type": "text", "text": ""}}

  event: content_block_delta
  data: {"type": "content_block_delta", "index": 1, "delta": {"type": "text_delta", "text": "The greatest common divisor of 1071 and 462 is **21**."}}

  // Additional text deltas...

  event: content_block_stop
  data: {"type": "content_block_stop", "index": 1}

  event: message_delta
  data: {"type": "message_delta", "delta": {"stop_reason": "end_turn", "stop_sequence": null}}

  event: message_stop
  data: {"type": "message_stop"}
  ```
</Accordion>

设置 `display: "omitted"` 时，思考块打开，一个 `thinking` 字符串为空的 `thinking_delta` 到达，随后是单个 `signature_delta`，然后块关闭。文本流式传输随即开始：

```sse Output
event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"thinking","thinking":"","signature":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"thinking_delta","thinking":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"signature_delta","signature":"EosnCkYICxIMMb3LzNrMu..."}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: content_block_start
data: {"type":"content_block_start","index":1,"content_block":{"type":"text","text":""}}
```

使用 `display: "updates"`（处于 beta 阶段）时，推理块的流式传输方式与 `"omitted"` 下相同。每个[进度更新块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)会在它所引出的 `tool_use` 块之前，以 `thinking_delta` 事件的形式流式传输其文本。在进度更新块打开之前出现数秒的停顿是正常的：

```sse Output
event: content_block_start
data: {"type":"content_block_start","index":1,"content_block":{"type":"thinking","thinking":"","signature":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":1,"delta":{"type":"thinking_delta","thinking":"Confirmed the retry path never refreshes the expired token. Editing auth.py to add the refresh call."}}

event: content_block_delta
data: {"type":"content_block_delta","index":1,"delta":{"type":"signature_delta","signature":"Es8CCkYICxIM..."}}

event: content_block_stop
data: {"type":"content_block_stop","index":1}

event: content_block_start
data: {"type":"content_block_start","index":2,"content_block":{"type":"tool_use","id":"toolu_01D7FLrfh4GYq7yT1ULFeyMV","name":"edit_file","input":{}}}
```

在 `"updates"` 下，一旦某个块的 `thinking_delta` 事件携带了非空文本，就将该块视为进度更新。

<Note>
  在启用思考的情况下使用流式传输时，您可能会注意到文本有时以较大的块到达，并与较小的逐令牌传送交替出现。这是预期行为，尤其是对于思考内容。

  流式传输系统以批次处理内容，这可能会延迟流式事件并将其分组，从而形成这种"块状"传送模式。
</Note>

有关流式传输的一般机制，请参阅[流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)。

## 思考与 effort

`thinking` 参数控制 Claude 在回答之前是否在[思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)中进行思考；`effort` 参数控制 Claude 在整个响应中投入多少工作量，在自适应模式下，这包括它思考的频率和深度。不要将 `adaptive` 作为 `effort` 的值传递：`adaptive` 是一种思考模式，而不是努力程度级别。

要了解每个 effort 级别对思考行为的影响，请参阅[引导思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost)页面上的[各级别思考行为表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#effort-levels)。[Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 页面记录了该参数本身，包括每个模型支持哪些级别。在 Claude Opus 4.5（唯一支持 effort 的仅扩展思考模型）上，effort 与 `budget_tokens` 组合使用。请参阅[预算规则与调优](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#budget-rules-and-tuning)。

两种控制方式以这种方式分开后，请选择与您的目标相符的那一种：

* **降低启用思考的工作负载的成本或延迟：** 首先降低 `effort`。它会整体缩减响应，包括思考。
* **Claude 思考得太少或太浅：** 提高 `effort`，或参阅引导页面上的[引导 Claude 的思考频率](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#tuning-thinking-behavior)。
* **您需要完全关闭思考：** 在允许的模型上使用 `thinking: {type: "disabled"}`（请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#supported-models)）。Claude Sonnet 5.5 会拒绝 `"disabled"`。它的最低设置是 `thinking: {type: "between_tools"}`，该设置会关闭预先思考。
* **您需要对开销设置硬性上限：** 使用 `max_tokens`。Effort 是软性引导，`max_tokens` 是严格限制。

## 思考与工具使用

思考可以与[工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)配合，让 Claude 对工具选择进行推理并处理工具结果。有两个约束条件：

1. **工具选择限制（手动模式）：** 手动扩展思考（`thinking: {type: "enabled"}`）下的工具使用仅支持 `tool_choice: {"type": "auto"}`（默认值）或 `tool_choice: {"type": "none"}`。使用 `tool_choice: {"type": "any"}` 或 `tool_choice: {"type": "tool", "name": "..."}` 会导致错误，因为这些选项会强制使用工具，与手动扩展思考不兼容。自适应思考（包括在思考默认开启的模型上）支持强制工具使用，但 Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1 和 Claude Mythos 5.1 除外（请参阅[响应预填充与强制工具使用](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#response-prefill-and-forced-tool-use)）。在接受强制工具使用的情况下，响应以工具调用开始，且没有 `thinking` 块。
2. **保留思考块：** 返回工具结果时，您必须将助手消息中的思考块完整且未经修改地传回 API。请参阅[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)。

**工具使用循环是一个助手轮次。** 从模型的角度来看，只有当 Claude 完成其完整响应时，助手轮次才会结束，而完整响应可能包括多次工具调用和结果。以下整个序列是单个助手轮次：

```text wrap
User: "What's the weather in Paris?"
Assistant: [thinking] + [tool_use: get_weather]
User: [tool_result: "20°C, sunny"]
Assistant: [text: "The weather in Paris is 20°C and sunny"]
```

整个轮次在单一思考模式下运行：您无法在轮次中间切换思考，包括在工具使用循环期间。在扩展（手动）模式下，API 还会强制要求启用思考的请求的最后一个助手轮次以思考块开始。自适应模式放宽了这一要求：任何助手轮次都无需以思考块开始。

**轮次中间的冲突会平稳降级。** 如果您在轮次中间切换思考（例如，在发送工具调用和返回其结果之间），API 不会报错。相反，它会针对该请求静默禁用思考。为了保持模型质量，API 可能会剥离会造成无效轮次结构的思考块，或者在对话历史与启用思考不兼容时禁用思考。要确认思考是否处于活动状态，请检查响应中是否存在 `thinking` 块。

**在轮次之间切换，而不是在轮次内切换。** 在每个轮次开始时规划您的思考策略。完成助手轮次后，再为下一个轮次更改思考配置：

```text wrap
User: "What's the weather?"
Assistant: [tool_use] (thinking disabled)
User: [tool_result]
Assistant: [text: "It's sunny"]
User: "What about tomorrow?"
Assistant: [thinking] + [text: "..."] (thinking enabled - new turn)
```

切换思考模式还会使提示缓存失效。请参阅[思考与提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-prompt-caching)。

### 保留思考块

当 Claude 调用工具时，它会暂停构建响应以等待外部信息。当您返回工具结果时，Claude 会继续构建同一个响应，因此其先前的推理必须仍然存在。请将每个 `thinking` 块连同它所伴随的 `tool_use` 块一起，完整且未经修改地传回 API。这之所以重要，有两个原因：

1. **推理连续性：** 思考块记录了导致工具请求的逐步推理。包含它们可以让 Claude 从中断的地方继续推理。
2. **上下文维护：** 工具结果在 API 结构中以用户消息的形式出现，但它们属于同一个连续的推理流程。保留思考块可以在多次 API 调用之间维持该流程。

简而言之：

* **必需：** 在工具使用轮次内，传回思考块。
* **建议：** 跨轮次时，传回所有内容。
* **允许：** 在工具使用之外，省略之前轮次的思考。

您无需自行修剪旧的思考。在多轮对话中传回所有思考块，API 会自动过滤它们，保留维持模型推理所需的块，并且只对实际展示给 Claude 的块计算输入令牌费用。保留哪些之前轮次的块因模型而异。请参阅[按模型划分的思考块保留](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)。要覆盖默认行为，请使用 [`clear_thinking_20251015` 上下文编辑策略](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#thinking-block-clearing)。

在最新的助手消息中，连续 `thinking` 块的顺序必须与模型在原始请求中生成的内容一致：您不能重新排列、编辑或部分删除它们。这也包括 [`redacted_thinking` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#redacted-thinking-blocks)。

<Note>
  被修改的思考块会以 400 错误被拒绝。有关确切的错误消息、常见原因和修复方法，请参阅[400 错误提示思考块不能被修改](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-blocks-modified)。唯一的例外是：放在[省略](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)块的空 `thinking` 字段中的文本会被忽略，而不是被拒绝。
</Note>

有关包含每种 SDK 代码的完整示例，请参阅[演练两轮工具使用往返](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-tool-workflows#two-turn-tool-use-round-trip)。该示例定义了一个工具，接收思考加工具使用的响应，并将助手轮次与工具结果一起回传。

### 交错思考

“Interleaved thinking”（交错思考）让 Claude 可以在工具调用之间进行思考，在对每个工具结果采取行动之前先对其进行推理。借助交错思考，Claude 可以：

* 在决定下一步做什么之前，对工具调用的结果进行推理
* 将多个工具调用串联起来，并在其间插入推理步骤
* 基于中间结果做出更细致的决策

<Note>
  连续的工具调用并不需要交错思考。无论是否使用交错思考，Claude 都可以串联工具调用。交错改变的是思考块在工具调用之间出现的位置，而不是工具调用能否串联。
</Note>

使用自适应思考时，交错思考在所有支持自适应思考的模型上都会自动启用。在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Mythos Preview、Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5、Claude Opus 4.8 和 Claude Opus 4.7 上，工具调用之间的推理始终出现在思考块中。Claude Haiku 4.5 不支持交错思考。在使用手动扩展思考的模型上，交错需要 beta 标头，并且会改变思考预算的计算方式。[手动模式下的交错思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#interleaved-thinking)介绍了各模型的规则以及特定平台的标头行为。

使用交错思考时，思考额度可以跨越整个助手轮次，而不是单个响应。交错思考仅支持[通过 Messages API 使用的工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)。

有关展示交错思考在双工具工作流中带来哪些变化的完整对比，请参阅[交错思考如何改变流程](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-tool-workflows#how-interleaved-thinking-changes-the-flow)。

### 工具调用之间的进度更新

在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Fable 5 上，模型可以在工具调用之间编写 “progress update”（进度更新）。进度更新是关于模型刚刚发现了什么以及接下来要做什么的说明，它是为观察智能体的人编写的，而不是作为推理。每个进度更新都作为独立的 `thinking` 块返回，带有自己的 `signature`，与同一位置的任何推理块分开。它紧接在它所引出的 `tool_use` 或 `server_tool_use` 块之前。每次工具调用之前最多有一个进度更新，且模型可以跳过其中任何一个。进度更新不是[交错思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#interleaved-thinking)：无论工具调用之间是否出现推理块，进度更新都会出现，并且一个响应可以同时包含两者。

进度更新块包含的内容取决于 [`display`](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)：

| `display`              | 推理块             | 进度更新块           |
| ---------------------- | --------------- | --------------- |
| `"omitted"`（这些模型上的默认值） | `thinking` 字段为空 | `thinking` 字段为空 |
| `"updates"`（beta）      | `thinking` 字段为空 | 摘要文本            |
| `"summarized"`         | 摘要文本            | 摘要文本，无法与推理块区分   |

在 Claude Sonnet 5.5 上，最低的思考设置是 `thinking: {type: "between_tools"}`。它会关闭预先思考，并且每个进度更新都会附带摘要文本返回，就像在 `display: "updates"` 下一样。`between_tools` 仅在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 为 `high` 或以下时被接受。在 `xhigh` 或 `max` 下，使用该设置的请求会返回 400 错误。`between_tools` 不接受任何其他字段：与其一起发送 `display`、`budget_tokens` 或 `block_binding` 都会返回 400 错误。该设置适用于所有提供 Claude Sonnet 5.5 的平台。请原样传回这些块：您传回的进度更新块会为模型提供它所编写的完整说明，而不是摘要。

对于隐藏推理、并在每一步向用户显示状态行的智能体界面，请使用 `display: "updates"`。在该设置下，任何带有非空文本的 `thinking` 块都是进度更新，因此只渲染这些块即可。该功能处于 beta 阶段，需要 beta 标头 `thinking-display-updates-2026-08-18`（要在 Amazon Bedrock、Claude Models on Google Cloud (Google managed) 或 Microsoft Foundry 上发送该标头，请参阅[其他平台上的 beta 功能](https://platform.claude.com/docs/zh-CN/api/beta-headers#beta-features-on-other-platforms)）。如果没有该标头，该值会被拒绝，并返回与未知 `display` 值相同的 400 `invalid_request_error`。

```json
{
  "model": "claude-fable-5-1",
  "max_tokens": 16000,
  "thinking": { "type": "adaptive", "display": "updates" },
  "tools": [
    {
      "name": "edit_file",
      "description": "Replace the contents of a file in the repository.",
      "input_schema": {
        "type": "object",
        "properties": {
          "path": { "type": "string" },
          "content": { "type": "string" }
        },
        "required": ["path", "content"]
      }
    }
  ],
  "messages": [
    {
      "role": "user",
      "content": "The login test fails after an hour of uptime. Find out why and fix it."
    }
  ]
}
```

在 `"updates"` 下，`tool_result` 之后的响应开头如下所示。第一个块是推理，保持为空，与 `"omitted"` 下一样。第二个块携带文本，因此它是进度更新。在 `"summarized"` 下两个块都携带文本，而在 `"omitted"` 下两个块都为空。

```json Output
{
  "content": [
    {
      "type": "thinking",
      "thinking": "",
      "signature": "EqMBCkYICxIM..."
    },
    {
      "type": "thinking",
      "thinking": "Confirmed the retry path never refreshes the expired token. Editing auth.py to add the refresh call.",
      "signature": "Es8CCkYICxIM..."
    },
    {
      "type": "tool_use",
      "id": "toolu_01D7FLrfh4GYq7yT1ULFeyMV",
      "name": "edit_file",
      "input": { "path": "auth.py", "content": "..." }
    }
  ]
}
```

使用进度更新时，请注意以下几点：

* 像其他 `thinking` 块一样，将进度更新块与助手轮次的其余部分一起原样传回。
* 您收到的文本是进度更新的摘要，通常为一两句话。不要依赖其长度。进度更新按其完整长度计入 `usage.output_tokens`，而不是按摘要长度计算。
* 在任何 `display` 值下，进度更新块都可能以空 `thinking` 字段返回。对于空块，不要渲染任何内容。在 `"updates"` 下，它看起来与空推理块相同，无需单独处理。
* 当响应在工具调用或工具结果之后不久因 `max_tokens`、`model_context_window_exceeded` 或 `stop_sequence` 而停止时，其最后一个块可能是一个进度更新块，代表模型尚未完成的工作。在 `"updates"` 和 `"summarized"` 下，其文本恰好为 `This part of the response was interrupted before it finished.`，您可以像显示其他更新一样显示它。在 `"omitted"` 下它为空。要继续，请原样传回助手轮次，并追加一条新的 `user` 消息（针对该轮次中的每个 `tool_use` 块附带一个 `tool_result`）。
* 在流式传输时，预计进度更新块打开前会有数秒的停顿。请参阅[流式传输思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#streaming-thinking)中的 `"updates"` 跟踪。
* 在较高的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 下以及在较长的工具链中，这些模型编写的进度更新会更少。如果您的界面依赖于它们，请参阅[要求提供面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#ask-for-user-facing-progress-updates)。对于 Claude Opus 5.5 和 Claude Sonnet 5.5，请参阅[为 Claude Opus 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#user-facing-progress-updates) 或[为 Claude Sonnet 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#user-facing-progress-updates) 中的"面向用户的进度更新"部分。对于 Claude Fable 5.2，请参阅[在较长轮次中让用户了解进展](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#keep-users-informed-during-long-turns)。

### 按模型划分的思考块保留

之前助手轮次中的思考块默认是否保留在上下文中，取决于模型：

* **保留所有之前的轮次：** Claude Opus 4.5 及更高版本的 Opus 模型、Claude Sonnet 4.6 及更高版本的 Sonnet 模型、Claude Haiku 5.5、Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5 以及 Claude Mythos Preview。
* **仅保留进行中的轮次（仅保留最后一个轮次的模型）：** 较早的 Opus 和 Sonnet 模型，以及截至 Claude Haiku 4.5 的所有 Haiku 模型。当您传回较旧的思考块时，API 会自动将其剥离。您无需自行删除它们。

保留带来两个好处：

* **缓存优化：** 保留的思考块允许在工具使用期间命中缓存，因为它们会与工具结果一起传回，并在整个助手轮次中增量缓存，从而在多步骤工作流中节省令牌。
* **不影响智能水平：** 保留思考块不会对模型性能产生负面影响。

代价是上下文占用：在保留全部轮次的模型上，长对话会消耗更多上下文空间，因为保留的思考块与其他对话历史一样计为输入（请参阅[思考与上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-the-context-window)）。在这两种机制下，该行为都是自动的。请按照[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)中的说明，继续传回完整、未经修改的思考块。要在任一方向上覆盖默认行为，请使用[思考块清除](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#thinking-block-clearing)。

**在对话中途切换模型。** 切换模型时（例如在[分类器拒绝回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)之后），请继续原样传回思考块。思考块只能由生成它的模型和某些其他模型读取。API 会忽略或丢弃目标模型无法读取的块（请参阅[被丢弃的块如何计费和报告](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)）。下表针对其中的每个模型，列出了该模型可以读取哪些块，以及哪些模型可以读取它的块：

* **Claude Fable 5.2 和 Claude Mythos 5.2：** 在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，它们可以读取彼此的思考块。在这些平台上，它们还可以读取来自其他模型的块，包括 Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5。没有任何其他模型可以读取 Claude Fable 5.2 或 Claude Mythos 5.2 的块。因此，从它们中的任何一个切换到这对模型之外的模型都会丢弃之前轮次的推理。
* **Claude Fable 5.1 和 Claude Mythos 5.1：** 它们可以读取彼此的思考块以及较早模型的思考块，但 Claude Mythos Preview 的思考块除外。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 和 Claude Mythos 5.2 也可以读取它们的块。其他模型都不能。从它们可以读取其块的模型切换到 Claude Fable 5.1 或 Claude Mythos 5.1 会保留对话的推理。从它们中的任何一个切换到无法读取其块的模型则会丢弃推理。
* **Claude Opus 5.5：** 它可以读取来自 Claude Opus 5 以及较早的 Opus、Sonnet 和 Haiku 模型的思考块。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，它还可以读取来自 Claude Sonnet 5.5 和 Claude Haiku 5.5 的思考块。它不能读取来自 Claude Fable 和 Claude Mythos 模型的思考块。在 Claude API 上，Claude Fable 5.1 和 Claude Mythos 5.1 可以读取 Claude Opus 5.5 的块。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 和 Claude Mythos 5.2 也可以读取它们。在 Claude API 上从 Claude Opus 5.5 升级切换到 Claude Fable 5.1 会保留之前轮次的推理。从 Claude Fable 5.1 切换到 Claude Opus 5.5 则会丢弃推理。
* **Claude Sonnet 5.5：** 它可以读取来自 Claude Sonnet 5、Claude Opus 4.8、Claude Haiku 4.5 及更早模型的思考块。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，它还可以读取来自 Claude Haiku 5.5 的思考块。它不能读取来自 Claude Opus 5、Claude Opus 5.5 或任何 Claude Fable 或 Claude Mythos 模型的思考块。在这两个平台上，Claude Opus 5.5、Claude Fable 5.2 和 Claude Mythos 5.2 可以读取 Claude Sonnet 5.5 的块。其他模型都不能。在这些平台上从 Claude Sonnet 5.5 升级切换到这三个模型之一会保留之前轮次的推理。从 Claude Sonnet 5.5 进行的任何其他切换都会丢弃推理。
* **Claude Haiku 5.5：** 它可以读取来自 Claude Sonnet 5、Claude Opus 4.8、Claude Haiku 4.5 及更早模型的思考块。它不能读取来自 Claude Opus 5、Claude Opus 5.5、Claude Sonnet 5.5 或任何 Claude Fable 或 Claude Mythos 模型的思考块。有四个模型可以读取 Claude Haiku 5.5 的块：Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.2 和 Claude Mythos 5.2。它们在 Claude API 和 Claude Models on Google Cloud (Google managed) 上可以这样做。其他模型都不能。

仅在会忽略（而非丢弃）之前的 `thinking` 和 `redacted_thinking` 块的模型上，为节省输入令牌才应自行剥离这些块。在兑换[回退额度](https://platform.claude.com/docs/zh-CN/build-with-claude/fallback-credit)时切勿剥离它们，因为兑换要求请求体保持不变。

## 保留的思考

[保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)决定了模型能否使用您从较早轮次传回的思考块。API 会检查请求中每个 `thinking` 或 `redacted_thinking` 块的 `signature`，确认两件事：

* **生成它的模型。** 每个模型都可以读取自己的思考块以及一组固定的其他模型的思考块。Claude Fable 5.1 可以读取来自 Claude Opus 5 的块，并且在 Claude API 上还可以读取来自 Claude Opus 5.5 的块；Claude Opus 5 和 Claude Opus 5.5 都不能读取来自 Claude Fable 5.1 的块。API 会丢弃当前模型无法读取的块，不会报错，也不会对其计费。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。
* **在它之前发送的所有内容。** 只有当顶层 `system` 提示、`tools` 以及它之前的消息保持不变时，思考块才保持有效。如果其中任何一项发生变化，该思考块以及历史记录中已有的之后所有思考块都将无法通过前缀检查。请参阅[保持前缀不变](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)。

来自 Claude Sonnet 5.5、Claude Haiku 5.5、Claude Fable 5.2 和 Claude Mythos 5.2 的思考块还与生成它们的账户绑定。有关此规则的适用范围，请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。

前缀检查在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 上运行。请将您的集成设计为仅追加（append-only），以便您回传的思考块能够通过该检查。[API 如何处理无效思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#mismatch-behavior)介绍了思考块未通过检查时会发生什么。

要使思考保持有效，请将每个助手轮次完全按照接收时的样子传回，并且只在 `messages` 的末尾添加新消息。如果您的代码自行构建 `messages` 数组，"保留的思考"页面涵盖以下内容：

* [哪些操作算作编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#what-counts-as-an-edit)，以及[如何检查您的代码是否进行了编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#how-to-tell-whether-your-integration-is-impacted)。
* [替代每种常见编辑的 API 功能](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#replace-prefix-edits)：用于新指令和每轮提醒的对话中途系统消息、用于工具变更的 `tool_addition` 和 `tool_removal` 块、用于 effort 变更的逐消息 `output_config`，以及用于裁剪的服务器端压缩和上下文编辑。
* [客户端压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#custom-compaction-on-the-client)：哪些模式能使思考保持有效，哪些不能。
* [`thinking-binding-controls-2026-08-01` beta 标头](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#preserved-thinking-controls)。它会在思考配置中添加一个 `block_binding.prefix_mismatch_behavior` 字段（`"error"` 或 `"drop_block"`），并在每个响应中添加一个 `input_transformations` 数组。该数组列出了 API 丢弃的每个思考块，以及未通过前缀检查但仍到达模型的每个思考块。

## 思考与提示缓存

“Prompt caching”（提示缓存）（[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)）与思考之间存在几种特定的交互方式。以下规则适用于两种思考模式。

**配置变更会使缓存失效。** 思考配置和解析后的 [`effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别会被渲染到提示本身中，因此更改其中任何一项都会开始一个新的缓存前缀。在 `adaptive`、`enabled` 和 `disabled` 之间切换、更改 `budget_tokens` 以及更改 effort 值，都会使缓存断点失效：消息级断点总是会未命中，而工具和系统提示断点也可能未命中，具体取决于模型在何处渲染该配置。请将任何思考或顶层 effort 变更视为重新开始缓存。在支持[逐消息 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) 的模型上，通过 `messages` 中的 `role: "system"` 消息进行的 effort 变更会使缓存前缀保持不变。保持相同配置的连续请求会保留缓存，并且将参数显式设置为其默认值等同于省略该参数。在任何[保留思考条件](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-thinking)下被 API 丢弃的思考块，会从该块的位置开始改变缓存前缀。原样传回的块会使缓存保持完整。包含用量输出的实际演示请参阅[引导思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#prompt-caching)页面。

**思考块会与工具结果一起缓存。** 在工具使用循环中，当您发出包含工具结果的后续请求时，就会发生缓存。此时，之前的对话历史（包括其中的思考块）可以被缓存，并且这些缓存的思考块在从缓存读取时会在您的用量指标中计为输入令牌。即使没有显式的 `cache_control` 标记，这也会自动发生，并且对于常规思考和交错思考的行为相同。代价是：您在响应中再也看不到的思考块，在从缓存读取时仍然会计入输入令牌用量。

**之前的块是否在上下文中取决于模型。** 这由[保留默认行为](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)决定。在保留全部轮次的模型上，之前轮次的思考块会保留在缓存和上下文中。在仅保留最后一个轮次的模型上，一旦您发送了一条不是工具结果的用户消息，之前所有的思考块都会从上下文中剥离。在这些模型上，如下对话：

```text wrap
User: ["What's the weather in Paris?"],
Assistant: [thinking_block_1] + [tool_use block 1],
User: [tool_result_1, cache=True],
Assistant: [thinking_block_2] + [text block 2],
User: [Text response, cache=True]
```

会被当作思考块从未存在过来处理：

```text wrap
User: ["What's the weather in Paris?"],
Assistant: [tool_use block 1],
User: [tool_result_1, cache=True],
Assistant: [text block 2],
User: [Text response, cache=True]
```

在保留全部轮次的模型上，同一请求会将 `thinking_block_1` 和 `thinking_block_2` 保留在上下文和缓存中。

**降级会从可缓存的历史中剥离思考。** 如果思考在轮次中途被禁用，而您在当前工具使用轮次中传入了思考内容，则该思考内容会被剥离，并且该请求的思考保持禁用状态（请参阅[优雅降级](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-with-tool-use)）。[交错思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#interleaved-thinking)会放大缓存失效的影响，因为思考块可能出现在多个工具调用之间。

<Tip>
  思考密集型任务的完成时间通常超过默认的 5 分钟缓存有效期。请考虑使用 [1 小时缓存时长](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#1-hour-cache-duration)，以便在较长的思考会话和多步骤工作流中保持缓存命中。
</Tip>

## 思考与上下文窗口

`max_tokens`（包括 Claude 在当前轮次中生成的所有思考）作为严格限制强制执行。在 Claude 4.5 及更新的模型上，如果输入令牌加上 `max_tokens` 超过"context window"（上下文窗口）大小，API 会接受该请求。如果生成随后达到上下文窗口限制，它会以 `stop_reason: "model_context_window_exceeded"` 停止，而不是返回错误。在较早的模型上，API 会返回验证错误。请参阅[停止原因与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)。

思考如何计入窗口取决于其生成时间：

* **当前轮次的思考**始终计入 `max_tokens`，按输出令牌计费，并在生成它的轮次中占用上下文窗口空间。
* **之前轮次的思考**取决于[保留默认行为](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)。在[保留所有之前轮次的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)上，之前的思考块会保留在上下文中，计入窗口，并像其他对话历史一样按输入令牌计费。在仅保留最后一个轮次的模型上，当您传回较旧的思考块时，API 会自动将其剥离，因此它们不会占用窗口空间或输入令牌。

实际应用中：

* 在保留全部轮次的模型上，请将思考视为普通对话历史来规划您的上下文窗口预算，因为它确实如此。长时间的智能体会话会在上下文中累积思考。如果您需要回收空间，请使用[思考块清除](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#thinking-block-clearing)。
* 在仅保留最后一个轮次的模型上，思考只是每轮的成本：每轮的思考计入该轮的 `max_tokens`，然后从窗口中移除。

以下图表说明了仅保留最后一个轮次（剥离）的机制。第一张图展示了多轮对话：每轮的思考块在输出中生成，但不会带入后续轮次的输入中。

<Frame>
  ![在剥离之前思考的模型上进行思考：每轮的思考都会生成，但不会带入后续轮次的输入](https://platform.claude.com/docs/zh-CN/images/context-window-thinking.svg)
</Frame>

第二张图展示了同一机制下的工具使用：在助手轮次期间，思考与其工具结果一起保留在上下文中，然后在下一个用户轮次时移除。

<Frame>
  ![在剥离之前思考的模型上结合工具进行思考：思考与其工具结果一起保留，直到下一个用户轮次](https://platform.claude.com/docs/zh-CN/images/context-window-thinking-tools.svg)
</Frame>

使用[令牌计数 API](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting) 为您的特定用例获取准确的计数，尤其是包含思考的多轮对话。

## 思考加密

完整的思考内容经过加密，并在每个思考块的 `signature` 字段中返回。当您传回思考块时，API 使用该签名来验证思考块是否由 Claude 生成。

使用签名时，请注意以下几点：

* 仅在[结合思考使用工具](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-with-tool-use)时才严格需要传回思考块。否则，您可以省略之前轮次的思考块。如果您确实传回了它们，API 是保留还是剥离它们取决于模型（请参阅[按模型划分的思考块保留](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)）。使用[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)来配置此行为。
* 传回思考块时，请将所有内容完全按照接收时的样子传回，以保持一致性并避免潜在问题。
* 在[流式传输响应](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#streaming-thinking)时，签名会在 `content_block_stop` 事件之前，作为 `content_block_delta` 事件中的 `signature_delta` 到达。
* Claude 4 及更高版本模型中的 `signature` 值明显比之前的模型更长。
* `signature` 字段是不透明的：请勿解释或解析它。
* `signature` 值在各平台之间兼容，包括 Claude API、[Amazon Bedrock (`bedrock-mantle`)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 和 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)。在一个平台上生成的值可以在另一个平台上使用，但来自 Claude Sonnet 5.5、Claude Haiku 5.5、Claude Fable 5.2 和 Claude Mythos 5.2 的思考块仅在生成它们的账户或与之关联的账户中有效。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。

## 已编辑的思考块

除了常规的 `thinking` 块之外，当 Claude 的部分推理因安全原因被编辑时，API 可能会返回 `redacted_thinking` 块。`redacted_thinking` 块在 `data` 字段中包含加密的思考内容，没有可读文本：

```json
{
  "type": "redacted_thinking",
  "data": "..."
}
```

`data` 字段是不透明且经过加密的。与常规思考块上的 `signature` 字段一样，在结合[工具](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-with-tool-use)继续多轮对话时，请将 `redacted_thinking` 块原样传回 API。

<Tip>
  如果您的代码在结合工具使用往返传递响应时按类型过滤内容块（例如 `block.type == "thinking"`），请同时包含 `redacted_thinking` 块。仅按 `block.type == "thinking"` 过滤会静默丢弃 `redacted_thinking` 块，并破坏[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)中描述的多轮协议。
</Tip>

<Note>
  `redacted_thinking` 块是一种独立的内容块类型，在思考因安全原因被编辑时返回。这与 [`display: "omitted"`](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display) 选项不同，后者返回 `thinking` 字段为空的常规 `thinking` 块。
</Note>

## 限制与功能兼容性

### 采样参数

在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Mythos Preview、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Opus 4.7、Claude Sonnet 5.5、Claude Sonnet 5 和 Claude Haiku 5.5 上，无论是否使用思考，非默认的 `temperature`、`top_p` 或 `top_k` 值在每个请求上都会返回 400 错误。在这些模型上，`top_p` 的默认值为 `0.99`，因此 `top_p` 为 `1` 会返回 400 错误，同时包含 `temperature` 和 `top_p` 的请求也会返回 400 错误，即使它们都是默认值。请参阅[移除采样参数](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#remove-sampling-parameters)。从 [API 版本 `2026-10-21`](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sampling-parameters) 开始，包含任意值的 `temperature`、`top_p` 或 `top_k` 的请求在每个模型上都会返回 400 错误。在较旧的模型上（在 `2026-10-21` 之前的 API 版本中），该限制仅在启用思考时适用：`temperature` 和 `top_k` 与思考不兼容，而 `top_p` 允许取 0.95 到 1 之间的值。

### 响应预填充与强制工具使用

Claude 4.6 及更高版本的模型以及 Claude Mythos Preview 会以 400 错误拒绝预填充的最终助手轮次，无论是否启用思考（请参阅[不支持预填充](https://platform.claude.com/docs/zh-CN/api/errors#prefill-not-supported)）。在较早的模型上，启用思考时无法预填充助手响应。强制工具使用（`tool_choice: {"type": "any"}` 或 `{"type": "tool", ...}`）与手动扩展思考不兼容，但可与自适应思考配合使用。例外情况是 Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1 和 Claude Mythos 5.1，它们在每个请求上都会以 400 错误拒绝强制工具使用。在这些模型上，请改用 `tool_choice: {"type": "auto"}` 配合[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)或[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。在接受强制工具使用的情况下，响应以工具调用开头，并且没有 `thinking` 块。要让模型在调用工具之前进行思考，请使用 `tool_choice: {"type": "auto"}` 并在提示中说明何时使用该工具。请参阅[结合工具使用进行思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-with-tool-use)。

### 输出限制

每个模型接受的 `max_tokens` 上限如下所列。在 [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#extended-output-beta) 上，`output-300k-2026-03-24` [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers)会为列出了批处理上限的模型提高该上限。

| Model                 | Max output tokens | Batches beta ceiling |
| :-------------------- | :---------------- | :------------------- |
| Claude Fable 5.2      | 128K              | —                    |
| Claude Mythos 5.2     | 128K              | —                    |
| Claude Fable 5.1      | 128K              | —                    |
| Claude Mythos 5.1     | 128K              | —                    |
| Claude Fable 5        | 128K              | —                    |
| Claude Mythos 5       | 128K              | —                    |
| Claude Mythos Preview | 128K              | Not available        |
| Claude Opus 5.5       | 128K              | 300K                 |
| Claude Opus 5         | 128K              | 300K                 |
| Claude Opus 4.8       | 128K              | 300K                 |
| Claude Opus 4.7       | 128K              | 300K                 |
| Claude Opus 4.6       | 128K              | 300K                 |
| Claude Opus 4.5       | 64K               | Not available        |
| Claude Sonnet 5.5     | 128K              | 300K                 |
| Claude Sonnet 5       | 128K              | 300K                 |
| Claude Sonnet 4.6     | 128K              | 300K                 |
| Claude Sonnet 4.5     | 64K               | Not available        |
| Claude Haiku 5.5      | 128K              | 300K                 |
| Claude Haiku 4.5      | 64K               | Not available        |

如需以编程方式读取模型的限制，请参阅[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api)。

### 长请求

当 `max_tokens` 大于 21,333 时，SDK 要求使用流式传输，以避免长时间运行的请求出现 HTTP 超时。这是客户端验证，而非 API 限制。如果您不需要增量处理事件，请将 `.stream()` (java: `.createStreaming()`; csharp: `.CreateStreaming()`; go: `.NewStreaming()`; php: `->createStream()`) 与 `.get_final_message()` (typescript: `.finalMessage()`; ruby: `.accumulated_message`; csharp: `.Aggregate()`; go: `message.Accumulate(event)`; java, php: `MessageAccumulator`) 配合使用，以获取完整的 `Message` 对象，而无需自行从各个事件中组装。请参阅[无需处理事件即可获取最终消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming#get-the-final-message-without-handling-events)。启用思考时，响应时间会更长，因为生成思考块会增加处理时间。对于每个请求的思考量超过约 32k 令牌的工作负载，请使用[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)来避免网络问题：此类请求的运行时间可能长到触及系统超时和打开连接数限制。

## 后续步骤

<CardGroup cols={2}>
  <Card title="引导思考" icon="compass" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost">
    通过 effort 级别、系统提示指导和逐消息引导，控制 Claude 思考的频率和深度，并了解思考的成本和定价。
  </Card>

  <Card title="工具和多轮工作流中的思考" icon="wrench" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-tool-workflows">
    逐步完成一个正确保留思考块的完整两轮工具使用往返流程，并了解交错思考如何改变该流程。
  </Card>

  <Card title="保留的思考" icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking">
    了解您的 Messages API 集成是否会编辑对话历史，并用能使较早思考块保持有效的 API 功能替换每种编辑。
  </Card>

  <Card title="思考问题排查" icon="hammer" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting">
    诊断并修复最常见的思考故障：配置导致的 400 错误、思考块为空或缺失、`max_tokens` 停止以及缓存未命中。
  </Card>

  <Card title="Effort" icon="sliders" href="https://platform.claude.com/docs/zh-CN/build-with-claude/effort">
    使用 effort 参数控制 Claude 在响应时使用的令牌数量，在响应的全面性与令牌效率之间进行权衡。
  </Card>
</CardGroup>

## 可用性

思考在所有模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：默认开启
* 用法：在接受该值的模型上，`thinking: {"type": "disabled"}` 会将其关闭。在默认关闭的模型上，设置 `thinking` 即可将其开启。

| 状态 | 云平台     |
| -- | ------- |
| 可用 | 所有支持的平台 |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启
* 用法：由 Managed Agents 设置思考配置；智能体定义中没有 `thinking` 字段。

| 状态 | 云平台     |
| -- | ------- |
| 可用 | 所有支持的平台 |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目 | 零数据保留                                                                                                                                     | HIPAA |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态 | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格  |

适用的端点：

* `/v1/messages` 使用 `thinking`
