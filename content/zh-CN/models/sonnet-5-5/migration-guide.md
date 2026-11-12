---
title: Claude Sonnet 5.5 迁移指南
url: https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide
description: 借助本迁移指南，从较早的 Sonnet 模型或 Claude Haiku 4.5 切换到 Claude Sonnet 5.5。启用 Claude Sonnet 5.5 的指导内容包括会返回错误的设置、思考方面的变化，以及针对每种起始模型的检查清单。
---

本指南列出了从 Claude Sonnet 5、Claude Sonnet 4.6、Claude Sonnet 4.5、Claude Sonnet 4、Claude 3.7 Sonnet 或 Claude Haiku 4.5 迁移到 Claude Sonnet 5.5 所需的代码更改。请先阅读前两节，然后向下阅读到与您当前模型对应的章节。[迁移检查清单](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migration-checklist)按起始模型列出了每一项更改。

<Note>
  本指南涵盖 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 代码的迁移。如果您使用 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)，则除了更新模型名称之外无需进行任何更改。
</Note>

让 Claude Code 为您迁移代码

```text
/claude-api migrate 将此项目迁移到 claude-sonnet-5-5
```

Claude Sonnet 5.5 的价格与 Claude Sonnet 5 相同，但提示缓存读取除外，其费用为每百万令牌 0.10 美元，是 Claude Sonnet 5 费率的一半。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。有关其上下文窗口和输出限制，请参阅 [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) 模型页面。有关功能和提示编写，请参阅 [Claude Sonnet 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#feature-support)和[为 Claude Sonnet 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5)。

## 向 Claude Sonnet 5.5 发送请求

此请求可以按原样在 Claude Sonnet 5.5 上运行。它设置了 effort 级别，并省略了五个会返回 400 错误的设置：[思考预算](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#sonnet-46-breaking-changes)、[采样参数](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#sonnet-46-breaking-changes)、[助手预填充](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migrating-from-sonnet-45)、[强制工具选择](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#forced-tool-use)以及 [`thinking: {"type": "disabled"}`](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#turn-off-up-front-thinking)。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 4096,
      "messages": [{
        "role": "user",
        "content": "Analyze the trade-offs between microservices and monolithic architectures"
      }],
      "output_config": {
        "effort": "medium"
      }
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-sonnet-5-5 \
    --max-tokens 4096 \
    --output-config '{effort: medium}' \
    --message '{role: user, content: "Analyze the trade-offs between microservices and monolithic architectures"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-sonnet-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Analyze the trade-offs between microservices and monolithic architectures",
          }
      ],
      output_config={"effort": "medium"},
  )

  print(f"Stop reason: {response.stop_reason}")
  for block in response.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Analyze the trade-offs between microservices and monolithic architectures"
      }
    ],
    output_config: {
      effort: "medium"
    }
  });

  console.log(`Stop reason: ${response.stop_reason}`);
  const textBlock = response.content.find((block) => block.type === "text");
  console.log(textBlock?.text);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 4096,
      Messages = [
          new UserMessageParam
          {
              Content = "Analyze the trade-offs between microservices and monolithic architectures"
          }
      ],
      OutputConfig = new OutputConfig
      {
          Effort = Effort.Medium
      }
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine($"Stop reason: {message.StopReason?.Raw()}");
  foreach (var block in message.Content)
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
  	Model:     anthropic.ModelClaudeSonnet5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Analyze the trade-offs between microservices and monolithic architectures")),
  	},
  	OutputConfig: anthropic.OutputConfigParam{
  		Effort: anthropic.OutputConfigEffortMedium,
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println("Stop reason:", response.StopReason)
  for _, block := range response.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.messages.OutputConfig;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_SONNET_5_5)
          .maxTokens(4096L)
          .addUserMessage("Analyze the trade-offs between microservices and monolithic architectures")
          .outputConfig(OutputConfig.builder()
              .effort(OutputConfig.Effort.MEDIUM)
              .build())
          .build();

      Message response = client.messages().create(params);
      response.stopReason().ifPresent(reason -> IO.println("Stop reason: " + reason));
      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          ['role' => 'user', 'content' => 'Analyze the trade-offs between microservices and monolithic architectures']
      ],
      model: Model::CLAUDE_SONNET_5_5,
      outputConfig: ['effort' => 'medium'],
  );

  echo "Stop reason: {$message->stopReason}", PHP_EOL;
  foreach ($message->content as $block) {
      if ($block instanceof \Anthropic\Messages\TextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Analyze the trade-offs between microservices and monolithic architectures" }
    ],
    output_config: {
      effort: "medium"
    }
  )

  puts "Stop reason: #{message.stop_reason}"
  message.content.each do |block|
    puts block.text if block.is_a?(Anthropic::Models::TextBlock)
  end
  ```
</CodeGroup>

## 默认启用思考

在 Claude Sonnet 5.5 上，不含 `thinking` 字段的请求会以 [adaptive thinking（自适应思考）](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)运行，`thinking: {"type": "adaptive"}` 也是如此。在 Claude Sonnet 4.6 及更早的模型以及 Claude Haiku 4.5 上，此类请求不进行思考。若要继续在不进行前置思考的情况下运行，请参阅[关闭前置思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#turn-off-up-front-thinking)。

| 模型                                   | 不含 `thinking` 字段时的思考 | 接受的 `thinking.type` 值                      | 默认 `display`   |
| ------------------------------------ | -------------------- | ------------------------------------------ | -------------- |
| Claude Sonnet 5.5                    | 开启                   | `"adaptive"`、`"between_tools"`             | `"omitted"`    |
| Claude Sonnet 5                      | 开启                   | `"adaptive"`、`"disabled"`                  | `"omitted"`    |
| Claude Sonnet 4.6                    | 关闭                   | `"adaptive"`、`"disabled"`、`"enabled"`（已弃用） | `"summarized"` |
| Claude Sonnet 4.5 和 Claude Haiku 4.5 | 关闭                   | `"disabled"`、`"enabled"`                   | `"summarized"` |

### 处理响应中的思考

原本不进行思考的代码需要完成以下全部三项。来自 Claude Sonnet 5 的代码可能已经具备前两项。

* **按 `type` 读取内容块。** 响应可能以 `thinking` 块开头，因此读取 `content[0].text` 的代码会出错。
* **在工具使用循环中原样传回 `thinking` 块**，包括空块。请参阅[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)。
* **重新审视 `max_tokens`。** 它涵盖思考和文本，且思考令牌按输出令牌计费。请参阅[成本控制](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#cost-control)。

思考文本默认被省略。`thinking` 块到达时 `thinking` 字段为空，并带有一个 `signature`。若要获取可读的摘要，请设置 `display: "summarized"`，这是 Claude Sonnet 4.6 及更早模型以及 Claude Haiku 4.5 上的默认值。请参阅[控制思考显示](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)。

### 关闭前置思考

若要在 Claude Sonnet 5.5 上关闭前置思考，请发送 `thinking: {"type": "between_tools"}`。这是最低的思考设置。它在工具调用之间的进度更新仍会以带有摘要文本的 `thinking` 块形式返回。没有工具时，响应仅包含文本。Claude Sonnet 5 则使用 `thinking: {"type": "disabled"}` 关闭思考，而更早的模型默认不进行思考。在 Claude Sonnet 5.5 上，`disabled` 会返回 400 `invalid_request_error`：

```text wrap
To turn thinking off on this model, send "thinking": {"type": "between_tools"} instead of {"type": "disabled"}. The model does not think before responding. The short updates it writes between tool calls come back as thinking blocks.
```

`between_tools` 适用于所有提供 Claude Sonnet 5.5 的平台。它在 `low`、`medium` 和 `high` effort 下均被接受。在 `xhigh` 或 `max` 下，它会返回 400 错误。若要以这些级别运行，请使用自适应思考：省略 `thinking` 字段或发送 `thinking: {"type": "adaptive"}`。`between_tools` 不接受其他字段：与其一同发送 `display`、`budget_tokens` 或 `block_binding` 会返回 400 错误。使用[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)时，回退到 Claude Sonnet 5 的 `between_tools` 请求会在该模型上以 `thinking: {"type": "disabled"}` 运行。

使用 `between_tools` 时，effort 不能在对话中途更改：与当前生效级别不同的逐消息 `output_config.effort` 会返回 400 错误。若要在每一轮中改变 effort，请使用自适应思考。有关提示编写指导，请参阅[在不进行前置思考的情况下运行](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#running-without-up-front-thinking)。

之前（Claude Sonnet 5）：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5",
      "max_tokens": 16000,
      "thinking": {"type": "disabled"},
      "output_config": {"effort": "xhigh"},
      "messages": [{"role": "user", "content": "..."}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-sonnet-5 \
    --max-tokens 16000 \
    --thinking '{type: disabled}' \
    --output-config '{effort: xhigh}' \
    --message '{role: user, content: "..."}'
  ```

  ```python Python
  client.messages.create(
      model="claude-sonnet-5",
      max_tokens=16000,
      thinking={"type": "disabled"},
      output_config={"effort": "xhigh"},
      messages=[{"role": "user", "content": "..."}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-sonnet-5",
    max_tokens: 16000,
    thinking: { type: "disabled" },
    output_config: { effort: "xhigh" },
    messages: [{ role: "user", content: "..." }]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigDisabled(),
      OutputConfig = new() { Effort = Effort.Xhigh },
      Messages = [new UserMessageParam { Content = "..." }],
  });
  ```

  ```go Go
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfDisabled: &anthropic.ThinkingConfigDisabledParam{},
  	},
  	OutputConfig: anthropic.OutputConfigParam{
  		Effort: anthropic.OutputConfigEffortXhigh,
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_SONNET_5)
      .maxTokens(16000L)
      .thinking(ThinkingConfigDisabled.builder().build())
      .outputConfig(OutputConfig.builder()
          .effort(OutputConfig.Effort.XHIGH)
          .build())
      .addUserMessage("...")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_SONNET_5,
      maxTokens: 16000,
      thinking: ThinkingConfigDisabled::with(),
      outputConfig: OutputConfig::with(effort: Effort::XHIGH),
      messages: [['role' => 'user', 'content' => '...']],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5,
    max_tokens: 16000,
    thinking: Anthropic::ThinkingConfigDisabled.new,
    output_config: { effort: Anthropic::OutputConfig::Effort::XHIGH },
    messages: [{ role: "user", content: "..." }]
  )
  ```
</CodeGroup>

之后（Claude Sonnet 5.5）：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 16000,
      "thinking": {"type": "between_tools"},
      "output_config": {"effort": "high"},
      "messages": [{"role": "user", "content": "..."}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-sonnet-5-5 \
    --max-tokens 16000 \
    --thinking '{type: between_tools}' \
    --output-config '{effort: high}' \
    --message '{role: user, content: "..."}'
  ```

  ```python Python
  client.messages.create(
      model="claude-sonnet-5-5",
      max_tokens=16000,
      thinking={"type": "between_tools"},
      output_config={"effort": "high"},
      messages=[{"role": "user", "content": "..."}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 16000,
    thinking: { type: "between_tools" },
    output_config: { effort: "high" },
    messages: [{ role: "user", content: "..." }]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigBetweenTools(),
      OutputConfig = new() { Effort = Effort.High },
      Messages = [new UserMessageParam { Content = "..." }],
  });
  ```

  ```go Go
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5_5,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfBetweenTools: &anthropic.ThinkingConfigBetweenToolsParam{},
  	},
  	OutputConfig: anthropic.OutputConfigParam{
  		Effort: anthropic.OutputConfigEffortHigh,
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_SONNET_5_5)
      .maxTokens(16000L)
      .thinking(ThinkingConfigBetweenTools.builder().build())
      .outputConfig(OutputConfig.builder()
          .effort(OutputConfig.Effort.HIGH)
          .build())
      .addUserMessage("...")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_SONNET_5_5,
      maxTokens: 16000,
      thinking: ThinkingConfigBetweenTools::with(),
      outputConfig: OutputConfig::with(effort: Effort::HIGH),
      messages: [['role' => 'user', 'content' => '...']],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 16000,
    thinking: Anthropic::ThinkingConfigBetweenTools.new,
    output_config: { effort: Anthropic::OutputConfig::Effort::HIGH },
    messages: [{ role: "user", content: "..." }]
  )
  ```
</CodeGroup>

## 按起始模型划分的迁移检查清单

从上往下依次处理各组，在涉及您的模型的那一组之后停止。对于 Claude Haiku 4.5，请应用除"Claude Sonnet 4 或更早版本"之外的所有组，最后以"仅限 Claude Haiku 4.5"结束。

### 所有起始模型

* 将模型 ID 更改为 `claude-sonnet-5-5`。
* [按 `type` 读取内容块](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#thinking-in-responses)，并原样传回 `thinking` 块。
* 如需在不进行预先思考的情况下继续运行，请在 `high` 或更低的 effort 下发送[最低思考设置](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#turn-off-up-front-thinking)。
* 将[强制工具使用](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#forced-tool-use)替换为 `auto` 和严格工具。
* 保持对话[仅追加](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#thinking-blocks)。
* 将计算机使用迁移到[工具集](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#computer-use-toolset)，且不使用 `fine-grained-tool-streaming-2025-05-14` beta 标头。在 Amazon Bedrock 上，`computer_20251124` 也可以使用。
* [将顾问工具](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#advisor-tool)与受支持的顾问配对使用，并预期会收到加密的建议。
* 从 `thinking` 块中读取[工具调用之间的文本](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#text-between-tool-calls)。
* [处理拒绝](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#safety-classifiers-and-fallback)，并配置回退。
* [重新运行您的 effort 扫描](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#recommended-changes)，并重新确定成本基线。

### Claude Sonnet 4.6 或更早版本

* 预期[不含 `thinking` 字段的请求也会进行思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#thinking-on-by-default)，并重新审视 `max_tokens`。
* 用 effort 级别替换[思考预算](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#sonnet-46-breaking-changes)。
* 移除 `temperature`、`top_p` 和 `top_k`。
* 如果您[显示思考文本](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#thinking-in-responses)，请设置 `display: "summarized"`。
* [重新计算令牌](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#other-changes-from-claude-sonnet-4-6)，并重新规划图像令牌预算。

### Claude Sonnet 4.5 或更早版本

* 替换[助手预填充](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migrating-from-sonnet-45)。
* 使用标准 JSON 解析器解析工具调用输入。
* 显式设置 `output_config.effort`。
* 移除所有上下文窗口相关的 beta 标头。
* 移除 `interleaved-thinking-2025-05-14`，并将 `fine-grained-tool-streaming-2025-05-14` 替换为 `eager_input_streaming`。
* 将 `output_format` 移至 `output_config.format`。

### Claude Sonnet 4 或更早版本

* 将[工具版本](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migrating-from-claude-sonnet-4-or-earlier)更新为 `text_editor_20250728` 和 `code_execution_20260521`。
* 处理 `refusal` 和 `model_context_window_exceeded` 停止原因。
* 检查工具字符串参数中的末尾换行符。
* 移除 `token-efficient-tools-2025-02-19` 和 `output-128k-2025-02-19`。
* 审查您的提示。

### 仅限 Claude Haiku 4.5

* 替换 [`claude-haiku-4-5-20251001`](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migrating-from-claude-haiku-4-5) 或其别名。
* 按更高的每令牌价格重新确定成本基线。
* 审查那些在 Claude Haiku 4.5 上因过短而无法缓存的提示。

## 从 Claude Sonnet 5 迁移到 Claude Sonnet 5.5

所有起始模型都需要完成本节中的更改。将您的模型 ID 替换为 `claude-sonnet-5-5`，它没有日期后缀。在其他平台上，请使用[可用性](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#availability)下列出的 ID。

### 不支持强制工具使用

本页上所有较早的模型都接受类型为 `any` 或 `tool` 的 `tool_choice`。Claude Sonnet 5.5 会以 400 错误拒绝这两者，包括在[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)端点上：

```text wrap
tool_choice: type "tool" and "any" are not supported for this model.
```

请发送 `tool_choice: {"type": "auto"}`，并将该工具标记为 `strict: true`，以使其输入与 schema 匹配。此时模型可以不调用该工具而直接作答，因此请在提示中说明何时使用它。[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)支持 JSON Schema 的一个子集，并要求每个对象都设置 `additionalProperties: false`。请参阅 [JSON Schema 限制](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#json-schema-limitations)。

之前（Claude Sonnet 5）：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5",
      "max_tokens": 1024,
      "tools": [{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g. San Francisco, CA"
            }
          },
          "required": ["location"],
          "additionalProperties": false
        }
      }],
      "tool_choice": {"type": "tool", "name": "get_weather"},
      "messages": [{"role": "user", "content": "What is the weather in Paris?"}]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-sonnet-5
  max_tokens: 1024
  tools:
    - name: get_weather
      description: Get the current weather in a given location
      input_schema:
        type: object
        properties:
          location:
            type: string
            description: The city and state, e.g. San Francisco, CA
        required: [location]
        additionalProperties: false
  tool_choice:
    type: tool
    name: get_weather
  messages:
    - role: user
      content: What's the weather in Paris?
  YAML
  ```

  ```python Python
  client.messages.create(
      model="claude-sonnet-5",
      max_tokens=1024,
      tools=tools,
      tool_choice={"type": "tool", "name": "get_weather"},
      messages=[{"role": "user", "content": "What's the weather in Paris?"}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-sonnet-5",
    max_tokens: 1024,
    tools,
    tool_choice: { type: "tool", name: "get_weather" },
    messages: [{ role: "user", content: "What's the weather in Paris?" }]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5,
      MaxTokens = 1024,
      Tools = [.. tools],
      ToolChoice = new ToolChoiceTool { Name = "get_weather" },
      Messages = [new UserMessageParam { Content = "What's the weather in Paris?" }],
  });
  ```

  ```go Go
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:      anthropic.ModelClaudeSonnet5,
  	MaxTokens:  1024,
  	Tools:      tools,
  	ToolChoice: anthropic.ToolChoiceParamOfTool("get_weather"),
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather in Paris?")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_SONNET_5)
      .maxTokens(1024L)
      .tools(tools)
      .toolChoice(ToolChoiceTool.of("get_weather"))
      .addUserMessage("What's the weather in Paris?")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_SONNET_5,
      maxTokens: 1024,
      tools: $tools,
      toolChoice: ToolChoiceTool::with(name: 'get_weather'),
      messages: [['role' => 'user', 'content' => "What's the weather in Paris?"]],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5,
    max_tokens: 1024,
    tools: tools,
    tool_choice: Anthropic::ToolChoiceTool.new(name: "get_weather"),
    messages: [{ role: "user", content: "What's the weather in Paris?" }]
  )
  ```
</CodeGroup>

之后（Claude Sonnet 5.5）：

<CodeGroup>
  ```bash cURL
  # 严格工具使用：每次调用都符合该工具的 input_schema
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 1024,
      "tools": [{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g. San Francisco, CA"
            }
          },
          "required": ["location"],
          "additionalProperties": false
        },
        "strict": true
      }],
      "tool_choice": {"type": "auto"},
      "messages": [{
        "role": "user",
        "content": "What is the weather in Paris? Use the get_weather tool."
      }]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-sonnet-5-5
  max_tokens: 1024
  tools:
    - name: get_weather
      description: Get the current weather in a given location
      input_schema:
        type: object
        properties:
          location:
            type: string
            description: The city and state, e.g. San Francisco, CA
        required: [location]
        additionalProperties: false
      # 严格工具使用：每次调用都符合该工具的 input_schema
      strict: true
  tool_choice:
    type: auto
  messages:
    - role: user
      content: What's the weather in Paris? Use the get_weather tool.
  YAML
  ```

  ```python Python
  client.messages.create(
      model="claude-sonnet-5-5",
      max_tokens=1024,
      # 严格工具使用：每次调用都符合工具的 input_schema
      tools=[{**tool, "strict": True} for tool in tools],
      tool_choice={"type": "auto"},
      messages=[
          {
              "role": "user",
              "content": "What's the weather in Paris? Use the get_weather tool.",
          }
      ],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 1024,
    // 严格工具使用：每次调用都符合该工具的 input_schema
    tools: tools.map((tool) => ({ ...tool, strict: true })),
    tool_choice: { type: "auto" },
    messages: [
      {
        role: "user",
        content: "What's the weather in Paris? Use the get_weather tool."
      }
    ]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 1024,
      // 严格工具使用：每次调用都符合该工具的 input_schema
      Tools = [.. tools.Select(tool => tool with { Strict = true })],
      ToolChoice = new ToolChoiceAuto(),
      Messages =
      [
          new UserMessageParam
          {
              Content = "What's the weather in Paris? Use the get_weather tool.",
          },
      ],
  });
  ```

  ```go Go
  // 严格工具使用：每次调用都与工具的 input_schema 匹配
  var strictTools []anthropic.ToolUnionParam
  for _, tool := range tools {
  	strictTool := *tool.OfTool
  	strictTool.Strict = anthropic.Bool(true)
  	strictTools = append(strictTools, anthropic.ToolUnionParam{OfTool: &strictTool})
  }
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:      anthropic.ModelClaudeSonnet5_5,
  	MaxTokens:  1024,
  	Tools:      strictTools,
  	ToolChoice: anthropic.ToolChoiceUnionParam{OfAuto: &anthropic.ToolChoiceAutoParam{}},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(
  			anthropic.NewTextBlock("What's the weather in Paris? Use the get_weather tool."),
  		),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_SONNET_5_5)
      .maxTokens(1024L)
      // 严格工具使用：每次调用都与该工具的 input_schema 匹配
      .tools(tools.stream()
          .map(tool -> tool.tool()
              .map(customTool -> customTool.toBuilder()
                  .strict(true)
                  .build())
              .map(ToolUnion::ofTool)
              .orElse(tool))
          .toList())
      .toolChoice(ToolChoiceAuto.builder().build())
      .addUserMessage("What's the weather in Paris? Use the get_weather tool.")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_SONNET_5_5,
      maxTokens: 1024,
      // 严格工具使用：每次调用都符合工具的 input_schema
      tools: array_map(fn (Tool $tool) => $tool->withStrict(true), $tools),
      toolChoice: ToolChoiceAuto::with(),
      messages: [
          [
              'role' => 'user',
              'content' => "What's the weather in Paris? Use the get_weather tool.",
          ],
      ],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 1024,
    # 严格工具使用：每次调用都符合该工具的 input_schema
    tools: tools.map { |tool| tool.merge(strict: true) },
    tool_choice: Anthropic::ToolChoiceAuto.new,
    messages: [
      { role: "user", content: "What's the weather in Paris? Use the get_weather tool." }
    ]
  )
  ```
</CodeGroup>

该示例将列表中的每个工具都标记为严格。一个请求最多可包含 20 个严格工具，且 MCP、计算机使用和浏览器使用工具集条目不接受 `strict`。在较长的工具列表中，只标记需要的工具。

### 思考块与模型和对话绑定

Claude Sonnet 5.5 可以读取来自 Claude Sonnet 5、Claude Opus 4.8、Claude Haiku 4.5 及更早模型的思考块，并且在 Claude API 和 Claude Models on Google Cloud (Google managed) 上还可以读取来自 Claude Haiku 5.5 的思考块。它不读取来自 Claude Opus 5、Claude Opus 5.5 或任何 Claude Fable 或 Claude Mythos 模型的块。API 会丢弃模型无法读取的块。请求仍返回 200，且被丢弃的块不计费。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。

每个 Claude Sonnet 5.5 思考块还会针对其之前的对话进行签名。请保持对话仅追加，并通过[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)更改指令或工具。请求可能会在早期历史记录被编辑后重放某个块，且未设置 `thinking.block_binding.prefix_mismatch_behavior`。对于在 2026 年 8 月 31 日 00:00 UTC 或之后创建的账户，该请求将收到 400 错误。[保留思考的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#feature-availability)列出了适用此规则的平台。Claude Sonnet 5.5 生成的思考块仅在生成它们的账户或与其关联的账户中有效。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。

### 部分计算机使用工具版本不受支持

Claude Sonnet 5.5 通过 `computer_toolset_20260801` 工具集支持计算机使用。在 Amazon Bedrock 上，Claude Sonnet 5.5 还接受带有 `computer-use-2025-11-24` beta 标头的 `computer_20251124` 工具。对于任何它不接受的计算机使用工具版本（例如 Claude API 上的 `computer_20251124`），Claude Sonnet 5.5 都会返回 400 错误。它在任何平台上都不接受 `computer_20250124`。

使用工具集的集成无需更改。要迁移到工具集，请将 `computer_20251124` 或 `computer_20250124` 替换为 `computer_toolset_20260801`，并移除该版本的 beta 标头。[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)列出了请求和智能体循环方面的更改。

如果您发送了 `fine-grained-tool-streaming-2025-05-14` beta 标头，请在迁移到工具集时将其移除。与工具集条目一起使用时，它会返回 400 错误。请改为在每个需要的工具上设置 `eager_input_streaming: true`。

### Advisor 工具接受的顾问更少

使用 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，Claude Sonnet 5.5 执行器需要以下顾问之一：Claude Opus 5、Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5、Claude Fable 5.1、Claude Fable 5.2、Claude Mythos 5、Claude Mythos 5.1 或 Claude Mythos 5.2。Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 5、Claude Sonnet 4.6 和 Claude Haiku 5.5 作为顾问会返回 400 错误。建议以 `advisor_redacted_result` 块的形式加密返回，因此其文本在响应中不可读。请参阅[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#model-compatibility)。

### 工具调用之间的文本在思考块中返回

在 Claude Sonnet 5.5 上，模型在工具调用之间写下的超过一两句话的说明会以[进度更新 `thinking` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)的形式返回，在默认 `display` 下为空。较短的评论仍为 `text`。在 Claude Sonnet 5 及更早的模型上，工具调用之间的所有文本都以 `text` 块返回。不会有请求失败，但显示这些说明的界面将不再显示它们。

使用自适应思考时，将 `display` 设置为 `"updates"`（beta，需 `thinking-display-updates-2026-08-18` 标头）可单独获取这些更新，设置为 `"summarized"` 则可获取与推理混合在一起的更新。请在其后紧跟的 `tool_use` 块之前渲染每个非空的 `thinking` 块。使用 [`between_tools`](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#turn-off-up-front-thinking) 时，无需 `display` 即可返回该文本。请参阅[面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#user-facing-progress-updates)。

### 安全分类器和回退

与 Claude Sonnet 5 相比，Claude Sonnet 5.5 会在更多类别中拒绝。拒绝会返回 `stop_reason: "refusal"`，其 `stop_details` 可能指明以下[类别](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)之一：

* **`"cyber"`：** 该请求可能助长网络危害，例如恶意软件或漏洞利用开发。
* **`"bio"`：** 该请求可能助长生物危害，例如危险的实验室方法。
* **`"frontier_llm"`：** 该请求可能有助于开发具有竞争关系的 AI 模型。
* **`"reasoning_extraction"`：** 该请求要求模型在响应文本中复现其内部推理。
* **`"general_harms"`：** 该请求属于另一个使用政策领域。良性的工作也可能触发此类别。

[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)（`fallbacks: "default"`）在 Claude API 和 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上处于 beta 阶段，它会在 Claude Sonnet 5 上重试 `"cyber"` 和 `"frontier_llm"` 拒绝。它不会重试 `"bio"`、`"reasoning_extraction"` 或 `"general_harms"` 拒绝。请参阅[拒绝和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)以及[拒绝如何计费](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。

对于来自 Claude Sonnet 4.6、Claude Sonnet 4.5 和 Claude Haiku 4.5 的代码，实时网络安全防护措施是新增的。如需开展合法的安全工作，请申请加入 [Cyber Verification Program](https://support.claude.com/en/articles/14604842-real-time-cyber-safeguards-on-claude-opus-and-sonnet)。

### 其他更改

* **提示缓存：** 最小可缓存提示为 512 个令牌，低于 Claude Sonnet 5、Claude Sonnet 4.6 和 Claude Sonnet 4.5 上的 1,024 个。请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#cache-limitations)。
* **新功能：** 有关对话中途的系统消息、对话中途的工具更改以及逐消息 effort，请参阅 [Claude Sonnet 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#feature-support)。使用 [`between_tools`](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#turn-off-up-front-thinking) 时，effort 不能在对话中途更改。

### 推荐的更改

重新运行您的 effort 扫描。Claude Sonnet 5.5 有五个 effort 级别：`low`、`medium`、`high`、`xhigh` 和 `max`。Claude API 上的默认值为 `high`。这些级别经过了重新校准，因此同一级别产生的思考量与 Claude Sonnet 5 上不同。除非您的工作负载是智能体型或对延迟敏感的，否则请从 `high` 开始。对于智能体编码和多步骤工具使用，规格明确的任务从 `medium` 开始，更难或更长的任务则改用 `high`。对于聊天和其他对延迟敏感的工作，从 `medium` 或 `low` 开始。在 `output_config.effort` 中设置级别。请参阅 [Claude Sonnet 5.5 的推荐 effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-sonnet-5-5)。然后对照[为 Claude Sonnet 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5)重新评估特定于模型的提示指令。

## 从 Claude Sonnet 4.6 及更早的 Sonnet 模型迁移到 Claude Sonnet 5.5

首先应用前面的每一节，替换 `claude-sonnet-4-6`。然后进行以下更改。如果使用的是 Claude Sonnet 4.5 或更早版本，请继续阅读后面的小节。

### 破坏性更改

**省略 thinking 的请求也会进行思考。** 请参阅[默认启用思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#thinking-on-by-default)和[关闭前置思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#turn-off-up-front-thinking)。

**思考预算会返回错误。** Claude Sonnet 4.6 将 `thinking: {"type": "enabled", "budget_tokens": N}` 作为已弃用的设置接受。Claude Sonnet 4.5 和 Claude Haiku 4.5 在所有思考中都使用它。Claude Sonnet 5.5 会返回 400 错误：

```text wrap
"thinking.type.enabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

移除预算并设置 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。预算与 effort 级别之间没有固定的对应关系，因此请在两到三个级别上运行您的评估。

之前（Claude Sonnet 4.6）：

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
          "content": "..."
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-sonnet-4-6
  max_tokens: 16000
  thinking:
    type: enabled
    budget_tokens: 10000
  messages:
    - role: user
      content: "..."
  YAML
  ```

  ```python Python
  client.messages.create(
      model="claude-sonnet-4-6",
      max_tokens=16000,
      thinking={"type": "enabled", "budget_tokens": 10000},
      messages=[{"role": "user", "content": "..."}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 16000,
    thinking: { type: "enabled", budget_tokens: 10000 },
    messages: [{ role: "user", content: "..." }]
  });
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeSonnet4_6,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigEnabled(budgetTokens: 10000),
      Messages = [new UserMessageParam { Content = "..." }]
  };

  var response = await client.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet4_6,
  	MaxTokens: 16000,
  	Thinking:  anthropic.ThinkingConfigParamOfEnabled(10000),
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
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
      .model(Model.CLAUDE_SONNET_4_6)
      .maxTokens(16000L)
      .enabledThinking(10000L)
      .addUserMessage("...")
      .build();

  Message response = client.messages().create(params);
  IO.println(response);
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 16000,
      messages: [['role' => 'user', 'content' => '...']],
      model: Model::CLAUDE_SONNET_4_6,
      thinking: ['type' => 'enabled', 'budgetTokens' => 10000],
  );
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_4_6,
    max_tokens: 16000,
    thinking: {
      type: "enabled",
      budget_tokens: 10000
    },
    messages: [
      { role: "user", content: "..." }
    ]
  )
  ```
</CodeGroup>

之后（Claude Sonnet 5.5）：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 16000,
      "thinking": {
        "type": "adaptive"
      },
      "output_config": {
        "effort": "high"
      },
      "messages": [
        {
          "role": "user",
          "content": "..."
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-sonnet-5-5
  max_tokens: 16000
  thinking:
    type: adaptive
  output_config:
    effort: high
  messages:
    - role: user
      content: "..."
  YAML
  ```

  ```python Python
  client.messages.create(
      model="claude-sonnet-5-5",
      max_tokens=16000,
      thinking={"type": "adaptive"},
      output_config={"effort": "high"},  # or "max", "xhigh", "medium", "low"
      messages=[{"role": "user", "content": "..."}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 16000,
    thinking: { type: "adaptive" },
    output_config: { effort: "high" }, // or "max", "xhigh", "medium", "low"
    messages: [{ role: "user", content: "..." }]
  });
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigAdaptive(),
      OutputConfig = new OutputConfig { Effort = Effort.High }, // or Max, Xhigh, Medium, Low
      Messages = [new UserMessageParam { Content = "..." }]
  };

  var response = await client.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5_5,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{},
  	},
  	OutputConfig: anthropic.OutputConfigParam{
  		Effort: anthropic.OutputConfigEffortHigh, // or Max, Xhigh, Medium, Low
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
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
      .model(Model.CLAUDE_SONNET_5_5)
      .maxTokens(16000L)
      .thinking(ThinkingConfigAdaptive.builder().build())
      .outputConfig(OutputConfig.builder()
          .effort(OutputConfig.Effort.HIGH) // or MAX, XHIGH, MEDIUM, LOW
          .build())
      .addUserMessage("...")
      .build();

  Message response = client.messages().create(params);
  IO.println(response);
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 16000,
      messages: [['role' => 'user', 'content' => '...']],
      model: Model::CLAUDE_SONNET_5_5,
      thinking: ['type' => 'adaptive'],
      outputConfig: ['effort' => 'high'], // or 'max', 'xhigh', 'medium', 'low'
  );
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 16000,
    thinking: {
      type: "adaptive"
    },
    output_config: {
      effort: "high" # or "max", "xhigh", "medium", "low"
    },
    messages: [
      { role: "user", content: "..." }
    ]
  )
  ```
</CodeGroup>

**“Sampling parameters”（采样参数）会返回错误。** 请移除 `temperature`、`top_p` 和 `top_k`。

**思考文本默认被省略。** 请参阅[处理响应中的思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#thinking-in-responses)。

### 其他更改

* **令牌增加约 30%：** Claude Sonnet 5.5 使用 Claude Sonnet 5 的分词器。与 Claude Sonnet 4.6、Claude Sonnet 4.5 和 Claude Haiku 4.5 相比，相同的文本会产生约多 30% 的令牌，具体取决于内容。请使用[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)重新计算，并重新审视 `max_tokens` 和成本。
* **Effort：** `xhigh` 是新增的，且各级别经过了重新校准。请参阅[推荐的更改](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#recommended-changes)。
* **图像：** Claude Sonnet 5.5 使用高分辨率图像层级，长边最多 2,576 像素，每张图像最多 4,784 个视觉令牌。Claude Sonnet 4.6、Claude Sonnet 4.5 和 Claude Haiku 4.5 的上限为 1,568 像素和 1,568 个令牌。一张 2000×1500 的图像在 Claude Sonnet 5.5 上消耗的令牌约为 2.5 倍。请参阅[分辨率和令牌成本](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)。

### 从 Claude Sonnet 4.5 或更早版本迁移

如果使用的是 Claude Sonnet 4.5、Claude Sonnet 4 或 Claude 3.7 Sonnet，请首先应用前面的每一节，然后进行以下更改。

**预填充会返回错误。** 与 Claude Sonnet 4.6 和 Claude Sonnet 5 一样，Claude Sonnet 5.5 会以 400 错误拒绝预填充的最后一个助手轮次。Claude Sonnet 4.5、Claude Haiku 4.5 及更早的模型接受预填充。错误内容如下：

```text wrap
This model does not support assistant message prefill. The conversation must end with a user message.
```

根据每个预填充的用途进行替换：

* **输出格式：** 使用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)，或在分类时使用带有枚举字段的工具。
* **开场白：** 在系统提示中要求直接回答。
* **不必要的拒绝：** 在用户消息中给出清晰的指令通常就足够了。
* **续写：** 将其移至用户消息，例如"您之前的回复被中断了，结尾是 `[previous_response]`。请从中断处继续。"
* **上下文提醒：** 将其放在用户轮次中。

**工具输入转义。** 工具调用参数中的转义可能有所不同。请使用标准 JSON 解析器解析 `input`。

**计算机使用。** Claude Sonnet 5.5 不接受 `computer_20250124`。请参阅[部分计算机使用工具版本不受支持](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#computer-use-toolset)。

**Effort。** Claude Sonnet 4.5 没有 effort 参数。请按照[推荐的更改](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#recommended-changes)中的说明显式设置 effort 级别。

**上下文和输出。** Claude Sonnet 5.5 拥有更大的上下文窗口和更高的输出限制。请参阅 [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) 模型页面。移除所有上下文窗口 beta 标头。

**Beta 标头。** 移除 `interleaved-thinking-2025-05-14`，因为自适应思考会自动交错进行。用每个需要的工具上的 `eager_input_streaming: true` 替换 `fine-grained-tool-streaming-2025-05-14`。该标头与计算机使用或浏览器使用工具集条目一起使用时会返回 400 错误。请参阅[细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming)。

**结构化输出。** `output_format` 参数已弃用。若仍要使用它，请添加 `structured-outputs-2025-11-13` beta 标头。否则，API 会返回 400 错误。请改用 `output_config.format`。

### 从 Claude Sonnet 4 或更早版本迁移

Claude Sonnet 4 已在 Claude API 上停用，但可在 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 上使用。Claude 3.7 Sonnet 已停用。从这两个模型中的任何一个迁移时，请首先应用前面的每一节，然后进行以下更改：

* **工具版本：** 使用 `text_editor_20250728`，工具名称为 `str_replace_based_edit_tool`，且没有 `undo_edit` 命令。使用 `code_execution_20260521`。请参阅[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)和[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#upgrade-to-latest-tool-version)。
* **停止原因：** 处理 `refusal`。Claude 4.5 及更高版本的模型在达到上下文窗口限制时还会以 `model_context_window_exceeded` 停止。请参阅[停止原因与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)。
* **末尾换行符：** Claude 4.5 及更高版本的模型会在工具调用字符串参数中保留末尾换行符。
* **Beta 标头：** 移除 `token-efficient-tools-2025-02-19` 和 `output-128k-2025-02-19`。
* **提示：** 对照[提示编写最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)审查您的提示。

## 从 Claude Haiku 4.5 迁移到 Claude Sonnet 5.5

首先应用直至并包括[从 Claude Sonnet 4.5 或更早版本迁移](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migrating-from-sonnet-45)在内的每一节，跳过 Claude Sonnet 4 小节。然后进行以下更改：

* **模型 ID：** 将 `claude-haiku-4-5-20251001` 或别名 `claude-haiku-4-5` 替换为 `claude-sonnet-5-5`。
* **成本：** 每令牌价格更高，且相同的文本会产生更多令牌。请重新计算令牌并重新确定成本基线。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。
* **提示缓存：** 最小可缓存提示从 4,096 个令牌降至 [Claude Sonnet 5.5 的最小值](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#other-changes-from-claude-sonnet-5)。
* **交错思考：** 自适应思考会自动在工具调用之间运行。
* **路由：** Claude Sonnet 5.5 可以读取 Claude Haiku 4.5 的思考块。向上迁移的对话会保留其推理。迁回 Claude Haiku 4.5 的对话会丢弃 Claude Sonnet 5.5 的块。
