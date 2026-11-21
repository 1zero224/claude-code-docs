---
title: 思考块的前缀检查
url: https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks
description: 前缀检查比较什么、哪些模型会运行它、请求未通过前缀检查时会发生什么，以及如何处理 400 错误和不返回错误的情况。
featureMetadata:
  status: stable
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-opus-5-5
    - claude-sonnet-5-5
    - claude-haiku-5-5
  supportedPlatforms:
    Claude API: stable
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

每个将思考块发送回[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)的请求都会经过一次 “prefix check”（前缀检查）。如果在 Claude 生成某个块之后，`system`、`tools` 或更早的消息发生了更改，前缀检查就会失败。如果位于其他思考块之间的某个思考块被省略或移动，前缀检查也会失败。

未通过前缀检查可能会也可能不会导致 400 错误，[这取决于模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)，因此请[处理两种情况](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#handle-either-outcome)。

## 前缀检查会影响谁

如果以下任一情况成立，您无需做任何更改：

* 由 Claude Code、claude.ai 或 Claude Managed Agents 管理您的对话历史。
* 由 Claude Agent SDK 管理历史，且您提供给 SDK 的系统提示保持不变。
* 您的代码遵循[仅追加集成的三个习惯](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)，并且在传输途中没有任何东西改写其请求。
* 您没有调用任何[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)。

如果您的请求会经过某个库、代理或网关，请[将它发送的内容与您发送的内容进行比较](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#find-the-change-details)。如果您运行着这样的组件，请参阅[库、代理和网关](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#libraries-proxies-gateways)，了解应当原样传递哪些内容。

## 哪些内容必须保持不变，以及原因

请将您的集成设计为 “append-only”（仅追加）。以下三个习惯可以做到这一点：

1. 将每个助手轮次完全按照 API 返回的原样发送回去，包括文本为空的思考块。
2. 为每个对话只构建一次 `system` 和 `tools`，并且每次都发送这同一份副本。
3. 将每次更新放在末尾的一条新消息中，并保持其不变。

每个思考块的推理都恰好针对其之前的内容：`system`、`tools` 以及每一条更早的消息。这些内容就是该块的*前缀*，也正是[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check-details)所比较的内容。

前缀检查与 “prompt caching”（提示缓存）类似，不同之处在于：更改会使其后的思考失效，而不仅仅是未命中缓存。

以一个在系统提示中写入当天日期的应用为例。到了午夜，日期发生变化，对话中已有思考的前缀也随之改变。这些思考会失效，下一个请求可能会失败。仅追加的应用会保持系统提示原样不变，并在[末尾的一条消息](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#changing-context)中说明新的日期。

<Frame>
  ![在末尾添加更新（update）可使所有思考保持有效；更改第 2 轮或 system 会使其后的思考失效](https://platform.claude.com/docs/zh-CN/images/preserved-thinking-add-or-change.svg)
</Frame>

一次更改只会使已经位于其后的思考失效。位于其之前的思考，以及 Claude 在此之后生成的思考，都保持有效。

## 如果您的代码更改了系统提示、工具或历史

| 如果您的代码…… | 应当怎么做 |
| ----- | ----- |
| 使用日期、模式或指令文件重新构建系统提示 | 停止更改 `system`。[将您自己的指令作为系统消息追加](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#new-instructions)（即 `messages` 中的一个 `role: "system"` 条目），并[在最新的轮次中说明新的值](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#changing-context)。对于其他人可以编辑的文本，请改为参阅[其他人可以编辑的文本](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#untrusted-text)。 |
| 添加、移除或改写某个工具，或在 MCP 服务器连接时获得新工具 | 停止更改 `tools`，并[在系统消息中添加或移除工具](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#tool-changes)。 |
| 裁剪、折叠或总结较早的轮次，并保持最新的轮次不变 | [让 API 编写摘要，或移除保留的轮次中的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#server-side-trimming)。 |

其他原因，例如跳过文本为空的思考块或重建已保存的对话，各有其修复方法：请参阅[更改工具或指令而不会导致前缀检查失败](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions)。要找出您的原因，请[让智能体检查您的代码](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#hand-to-an-agent)。

## 未通过前缀检查时各模型的行为

在 Claude API 和[部分云平台](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#feature-availability)上，在未设置 [`prefix_mismatch_behavior`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#preserved-thinking-controls) 的情况下，未通过前缀检查的结果取决于模型：

| 具有保留思考的模型 | 结果 |
| ----- | ----- |
| Claude Fable 5.2 | 400 错误 |
| Claude Mythos 5.2 | 400 错误 |
| Claude Mythos 5.1 | 400 错误 |
| Claude Fable 5.1 | [可能出现 400 错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#possible-400-error) |
| Claude Opus 5.5 | [可能出现 400 错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#possible-400-error) |
| Claude Sonnet 5.5 | [可能出现 400 错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#possible-400-error) |
| Claude Haiku 5.5 | [可能出现 400 错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#possible-400-error) |

在表中显示"可能出现 400 错误"的地方，存在两种情况：未通过前缀检查的请求可能收到 400 错误，也可能不返回错误。当不返回错误时，未通过前缀检查的块会到达模型。Claude 会像读取其他任何输入一样读取它们，您需要为它们付费，而 Claude 会基于针对已发生更改的内容所做的推理来工作。

在这些模型上，您可以[检查请求会遇到哪种情况](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#check-which-case)，但仍应[处理两种情况](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#handle-either-outcome)。

<Tip>
  遵循[仅追加集成的三个习惯](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)的代码不会导致请求未通过前缀检查。
</Tip>

未设置 `prefix_mismatch_behavior` 的 [Message Batches 条目](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#batches)不会收到错误，并且 API 可能会丢弃未通过前缀检查的块。

## 处理两种情况

要让每个请求都得到同一种结果，请将您的集成设计为仅追加，或设置 `prefix_mismatch_behavior`。否则，您的代码必须处理两种行为。

* **仅追加：**&#x9075;循[这三个习惯](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)，您的代码就不会导致请求未通过前缀检查。
* **该设置：**&#x5728;每个请求中使用 `"error"`，每次失败都是 400 错误，您可以[在代码中捕获它](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#handle-the-error-in-code)。
* **两种行为：**&#x5982;果您不设置该设置，请捕获 400 错误，并[留意到达 Claude 的未通过前缀检查的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#watch-production-traffic)。

出现 400 错误后，在该对话后续的请求中发送 `"drop_block"` 即可恢复。这是有代价的：请参阅[丢弃思考的代价](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#drop-block-cost)。不要在您的所有对话中都发送它来掩盖错误。

## 技术细节

### 前缀检查的工作原理

具有保留思考的模型会在每个思考块的 `signature` 中记录其前缀的指纹。API 参考将前缀检查称为对话检查，而 400 错误会指出未通过前缀检查的块 `is bound to a different conversation`。

### 哪些模型会运行前缀检查

前缀检查在 Claude API 和[部分云平台](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#feature-availability)上运行。它会在发往这些模型之一的请求中，检查由[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)生成的思考块。在 Claude API 上，其他模型（例如 Claude Opus 5）不会运行前缀检查，前缀检查也不涵盖它们的块。

### 什么算作更改

* **每个块都会与其自身的记录进行比较。**&#x41;PI 会将本次请求中某个块之前的内容，与 Claude 生成该块时其签名所记录的内容进行比较。您的上一个请求不起任何作用。前缀检查只关注内容，而不关注会话、ID 或用户。相同的内容出现在新请求或分叉中也能通过前缀检查。
* **前缀检查比较的是您发送的内容，而不是 Claude 读取的内容。**[在令牌阈值处进行压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)和上下文编辑只会改变 Claude 读取的内容，因此它们从不算作更改。在阈值压缩块之后，API 只比较从该块开始的消息，以及 `system` 和 `tools`。按需压缩有其自身的条件：请参阅[保留的思考保持有效的条件](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-thinking-blocks#conditions-for-kept-thinking-to-stay-valid)。对于通过 URL 发送的文件，以及在 Claude API 上的 [MCP 工具列表](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#pin-mcp-tool-list)，API 比较的是它获取到的内容。
* **您发送的思考必须是一段连续不间断的序列。**&#x601D;考块不会作为内容进行比较。但除第一个之外，您发送的每个思考块都必须与 Claude 生成它时拥有相同的上一个思考块。因此，您可以[省略最早的、最新的或全部思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#leave-out-thinking)，但不能省略中间的某一个。

一次更改可能小到只是工具描述中的一个词。更改、插入、移动或删除一条更早的消息都算作更改，无论其角色是什么。在思考块所在的消息中更改其之前的文本也算作更改。

### 什么不算作更改

| 以下不算作更改 | 原因 |
| ----- | ----- |
| `model`、`effort`、`max_tokens`、`tool_choice`、`metadata`、`thinking.display` 及其他参数 | 只比较 `system`、`tools` 和消息 |
| JSON 格式或键的顺序 | API 比较的是解析后的 JSON，而不是其文本 |
| `cache_control` 标记 | 它们说明的是缓存位置，而不是 Claude 读取的内容 |
| 设置了 `defer_loading: true` 且尚未被任何内容引用的工具 | 在被引用之前，它不在 Claude 的上下文中 |

### 设置 `prefix_mismatch_behavior`

[beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers#beta-features-on-other-platforms) `thinking-binding-controls-2026-08-01` 会添加设置 `thinking.block_binding.prefix_mismatch_behavior` 以及响应中的 `input_transformations` 数组。

| 值 | 如果某个块未通过前缀检查 |
| ----- | ----- |
| `"error"` | API 以 400 错误拒绝该请求，错误中会指出第一个无效的块 |
| `"drop_block"` | API 丢弃无效的块，请求成功。您无需为这些块付费，但[丢弃它们可能会花费更多](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#drop-block-cost) |
| 未设置，或为 `null` | [取决于模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes) |

只要更改仍然存在，未通过前缀检查的块在之后每个发送它们的请求中都保持无效。该设置只作用于携带它的那一个请求。在关闭思考（`"disabled"` 或 `"between_tools"`）的情况下，发送该设置在所有模型上都会导致 400 错误。

以下请求将 `prefix_mismatch_behavior` 设置为 `"drop_block"`。在第一个轮次中没有需要发送回去的思考，因此返回的 `input_transformations` 为空：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: thinking-binding-controls-2026-08-01" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 16000,
      "thinking": {
        "type": "adaptive",
        "block_binding": {
          "prefix_mismatch_behavior": "drop_block"
        }
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
  ant beta:messages create --beta thinking-binding-controls-2026-08-01 <<'YAML'
  model: claude-opus-5-5
  max_tokens: 16000
  thinking:
    type: adaptive
    block_binding:
      prefix_mismatch_behavior: drop_block
  messages:
    - role: user
      content: What is the greatest common divisor of 1071 and 462?
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.beta.messages.create(
      model="claude-opus-5-5",
      max_tokens=16000,
      thinking={
          "type": "adaptive",
          "block_binding": {"prefix_mismatch_behavior": "drop_block"},
      },
      messages=[
          {
              "role": "user",
              "content": "What is the greatest common divisor of 1071 and 462?",
          }
      ],
      betas=["thinking-binding-controls-2026-08-01"],
  )

  for block in response.content:
      if block.type == "text":
          print(block.text)

  print(f"Input transformations: {len(response.input_transformations or [])}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.beta.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 16000,
    thinking: {
      type: "adaptive",
      block_binding: { prefix_mismatch_behavior: "drop_block" }
    },
    messages: [
      { role: "user", content: "What is the greatest common divisor of 1071 and 462?" }
    ],
    betas: ["thinking-binding-controls-2026-08-01"]
  });

  for (const block of response.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  console.log(`Input transformations: ${response.input_transformations?.length ?? 0}`);
  ```

  ```csharp C#
  using Anthropic.Models.Beta;
  using Anthropic.Models.Beta.Messages;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  var response = await client.Beta.Messages.Create(
      new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 16000,
          Thinking = new BetaThinkingConfigAdaptive
          {
              BlockBinding = new()
              {
                  PrefixMismatchBehavior = BetaThinkingPrefixMismatchBehavior.DropBlock,
              },
          },
          Messages =
          [
              new BetaUserMessageParam
              {
                  Content = "What is the greatest common divisor of 1071 and 462?",
              },
          ],
          Betas = [AnthropicBeta.ThinkingBindingControls2026_08_01],
      }
  );

  foreach (var block in response.Content)
  {
      if (block.TryPickText(out var textBlock))
      {
          Console.WriteLine(textBlock.Text);
      }
  }

  Console.WriteLine($"Input transformations: {response.InputTransformations?.Count ?? 0}");
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 16000,
  	Thinking: anthropic.BetaThinkingConfigParamUnion{
  		OfAdaptive: &anthropic.BetaThinkingConfigAdaptiveParam{
  			BlockBinding: anthropic.BetaThinkingBlockBindingParam{
  				PrefixMismatchBehavior: anthropic.BetaThinkingPrefixMismatchBehaviorDropBlock,
  			},
  		},
  	},
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("What is the greatest common divisor of 1071 and 462?")),
  	},
  	Betas: []anthropic.AnthropicBeta{anthropic.AnthropicBetaThinkingBindingControls2026_08_01},
  })
  if err != nil {
  	log.Fatal(err)
  }

  for _, block := range response.Content {
  	if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  fmt.Printf("Input transformations: %d\n", len(response.InputTransformations))
  ```

  ```java Java
  import com.anthropic.models.beta.AnthropicBeta;
  import com.anthropic.models.beta.messages.BetaMessage;
  import com.anthropic.models.beta.messages.BetaThinkingBlockBinding;
  import com.anthropic.models.beta.messages.BetaThinkingConfigAdaptive;
  import com.anthropic.models.beta.messages.BetaThinkingPrefixMismatchBehavior;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(16000L)
          .addBeta(AnthropicBeta.THINKING_BINDING_CONTROLS_2026_08_01)
          .thinking(BetaThinkingConfigAdaptive.builder()
              .blockBinding(BetaThinkingBlockBinding.builder()
                  .prefixMismatchBehavior(BetaThinkingPrefixMismatchBehavior.DROP_BLOCK)
                  .build())
              .build())
          .addUserMessage("What is the greatest common divisor of 1071 and 462?")
          .build();

      BetaMessage response = client.beta().messages().create(params);

      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
      IO.println("Input transformations: "
          + response.inputTransformations().map(List::size).orElse(0));
  }
  ```

  ```php PHP
  use Anthropic\Beta\AnthropicBeta;
  use Anthropic\Beta\Messages\BetaTextBlock;
  use Anthropic\Beta\Messages\BetaThinkingBlockBinding;
  use Anthropic\Beta\Messages\BetaThinkingConfigAdaptive;
  use Anthropic\Beta\Messages\BetaThinkingPrefixMismatchBehavior;
  use Anthropic\Client;
  use Anthropic\Messages\Model;

  $client = new Client();

  $response = $client->beta->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 16000,
      thinking: BetaThinkingConfigAdaptive::with(
          blockBinding: BetaThinkingBlockBinding::with(
              prefixMismatchBehavior: BetaThinkingPrefixMismatchBehavior::DROP_BLOCK,
          ),
      ),
      messages: [
          ['role' => 'user', 'content' => 'What is the greatest common divisor of 1071 and 462?'],
      ],
      betas: [AnthropicBeta::THINKING_BINDING_CONTROLS_2026_08_01],
  );

  foreach ($response->content as $block) {
      if ($block instanceof BetaTextBlock) {
          echo $block->text, PHP_EOL;
      }
  }

  echo 'Input transformations: ', count($response->inputTransformations ?? []), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 16_000,
    thinking: {
      type: "adaptive",
      block_binding: { prefix_mismatch_behavior: "drop_block" }
    },
    messages: [
      { role: "user", content: "What is the greatest common divisor of 1071 and 462?" }
    ],
    betas: [Anthropic::AnthropicBeta::THINKING_BINDING_CONTROLS_2026_08_01]
  )

  response.content.each do |block|
    puts block.text if block.is_a?(Anthropic::Beta::BetaTextBlock)
  end

  puts "Input transformations: #{response.input_transformations&.length || 0}"
  ```
</CodeGroup>

### 监控生产流量

在未设置 `prefix_mismatch_behavior` 的情况下，在[部分模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)上，未通过前缀检查的请求可能不会收到错误。这时未通过前缀检查的思考块会到达模型。要在生产流量中找出这些块，请发送 `thinking-binding-controls-2026-08-01` beta 标头，不设置该设置，并记录每个响应的 `input_transformations` 数组。该标头会添加这个数组，并不会改变模型接收到的内容。

每个这样的块都会在该数组中拥有各自的 `thinking_mismatch_allowed` 条目。一个条目如下所示：

```json
{
  "input_transformations": [
    {
      "type": "thinking_mismatch_allowed",
      "path": "messages.1.content.0",
      "reason": "prefix_binding_mismatch"
    }
  ]
}
```

要找出更改，请参阅[找出您的代码更改了什么](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#find-the-change)。

### 检查您遇到哪种情况

在[部分模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)上，未通过前缀检查的请求可能收到 400 错误，也可能不返回错误。以下测试可以显示某个请求遇到的是哪一种：

1. 选取一个包含这些模型之一所生成思考块的对话。更改或添加系统提示（`system`）。该块随后会未通过前缀检查，因为系统提示在 Claude 生成该块之后发生了更改。
2. 将其发送给同一模型，附带 `thinking-binding-controls-2026-08-01` beta 标头，并且不设置 `prefix_mismatch_behavior`。

响应为以下之一：

* **一个显示 `bound to a different conversation` 的 [400 错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#prefix-mismatch-error)：**&#x8BE5;请求收到了 400 错误。
* **一个 `input_transformations` 数组中包含 `thinking_mismatch_allowed` 条目的 200 响应：**&#x8BE5;请求没有收到错误。每个条目都指出一个未通过前缀检查并到达了 Claude 的思考块。
* **一个 `input_transformations` 数组为空的 200 响应：**&#x6CA1;有块未通过前缀检查，因此该测试无法说明任何情况。请确认该对话包含来自该模型的思考块。

该结果只描述这一个请求，并不保证之后的请求会怎样，因此请编写代码来[处理两种情况](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#handle-either-outcome)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="更改工具或指令而不会导致前缀检查失败" icon="edit" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions">
    针对您的代码对对话所做的每种更改，应当改为发送什么。
  </Card>

  <Card title="修复绑定到其他对话的思考块" icon="wrench" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation">
    恢复失败的对话，找出您的代码更改了什么，并测试问题是否已消除。
  </Card>

  <Card title="保留思考" icon="book" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking">
    什么是保留思考、思考块与什么绑定，以及切换模型或账户。
  </Card>
</CardGroup>
