---
title: 按消息设置 effort
url: https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message
description: 使用系统消息在对话中途更改 effort 级别，同时保留提示缓存。
featureMetadata:
  status: beta
  betaHeader: mid-conversation-output-config-2026-07-01
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-haiku-5-5
  supportedPlatforms:
    Claude API: beta
    Amazon Bedrock: beta
    Amazon Bedrock (bedrock-runtime):
      availability: beta
      note: 在请求正文的 `anthropic_beta` 数组中发送 beta 值。
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Anthropic managed): beta
    Claude Models on Google Cloud (Google managed): beta
    Microsoft Foundry: beta
---

按消息更改 effort 会为对话的后续轮次设置新的 effort 级别，而不会更改请求的顶层值，因此缓存的前缀仍然匹配。

## 更改 effort 级别

要在对话中途更改级别：

1. 在 `messages` 中添加一条 `role: "system"` 消息，将 `content` 留空，并在 `output_config.effort` 中设置新的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。
2. 发送 `mid-conversation-output-config-2026-07-01` [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers)。
3. 保持顶层值不变。

以下示例将顶层值保持为 `high`，并在一次常规的后续交互中降至 `low`：

<CodeGroup>
  ```bash cURL
  # 仅含 effort 的系统消息：新级别从下一个用户轮次开始生效。
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: mid-conversation-output-config-2026-07-01" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-fable-5-1",
      "max_tokens": 4096,
      "output_config": {"effort": "high"},
      "messages": [
        {"role": "user", "content": "Plan a migration from SQLite to PostgreSQL in three short steps."},
        {"role": "assistant", "content": "1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts."},
        {"role": "system", "content": [], "output_config": {"effort": "low"}},
        {"role": "user", "content": "Summarize the plan in one sentence."}
      ]
    }'
  ```

  ```bash CLI
  ant beta:messages create \
    --beta mid-conversation-output-config-2026-07-01 <<'YAML'
  model: claude-fable-5-1
  max_tokens: 4096
  output_config:
    effort: high
  messages:
    - role: user
      content: Plan a migration from SQLite to PostgreSQL in three short steps.
    - role: assistant
      content: "1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts."
    # 仅含 effort 的系统消息：新级别从下一个用户轮次起生效。
    - role: system
      content: []
      output_config:
        effort: low
    - role: user
      content: Summarize the plan in one sentence.
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.beta.messages.create(
      model="claude-fable-5-1",
      max_tokens=4096,
      output_config={"effort": "high"},
      messages=[
          {
              "role": "user",
              "content": "Plan a migration from SQLite to PostgreSQL in three short steps.",
          },
          {
              "role": "assistant",
              "content": "1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts.",
          },
          # 仅含 effort 的系统消息：新级别从下一个用户轮次开始生效。
          {"role": "system", "content": [], "output_config": {"effort": "low"}},
          {"role": "user", "content": "Summarize the plan in one sentence."},
      ],
      betas=["mid-conversation-output-config-2026-07-01"],
  )

  for block in response.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.beta.messages.create({
    model: "claude-fable-5-1",
    max_tokens: 4096,
    output_config: { effort: "high" },
    messages: [
      {
        role: "user",
        content: "Plan a migration from SQLite to PostgreSQL in three short steps."
      },
      {
        role: "assistant",
        content:
          "1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts."
      },
      // 仅含 effort 的系统消息：新级别从下一个用户轮次起生效。
      { role: "system", content: [], output_config: { effort: "low" } },
      { role: "user", content: "Summarize the plan in one sentence." }
    ],
    betas: ["mid-conversation-output-config-2026-07-01"]
  });

  for (const block of response.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta;
  using Anthropic.Models.Beta.Messages;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  var response = await client.Beta.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeFable5_1,
      MaxTokens = 4096,
      OutputConfig = new() { Effort = Effort.High },
      Messages =
      [
          new BetaUserMessageParam { Content = "Plan a migration from SQLite to PostgreSQL in three short steps." },
          new BetaAssistantMessageParam { Content = "1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts." },
          // 仅含 effort 的系统消息：新级别从下一个用户轮次起生效。
          new BetaSystemMessageParam
          {
              Content = new([]),
              OutputConfig = new() { Effort = BetaSystemMessageOutputConfigEffort.Low },
          },
          new BetaUserMessageParam { Content = "Summarize the plan in one sentence." },
      ],
      Betas = [AnthropicBeta.MidConversationOutputConfig2026_07_01],
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

  response, err := client.Beta.Messages.New(context.Background(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeFable5_1,
  	MaxTokens: 4096,
  	OutputConfig: anthropic.BetaOutputConfigParam{
  		Effort: anthropic.BetaOutputConfigEffortHigh,
  	},
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Plan a migration from SQLite to PostgreSQL in three short steps.")),
  		{OfAssistant: &anthropic.BetaAssistantMessageParam{
  			Content: []anthropic.BetaContentBlockParamUnion{anthropic.NewBetaTextBlock("1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts.")},
  		}},
  		// 仅含 effort 的系统消息：新级别从下一个用户轮次起生效。
  		anthropic.NewBetaSystemMessage(anthropic.BetaSystemMessageOutputConfigParam{
  			Effort: anthropic.BetaSystemMessageOutputConfigEffortLow,
  		}),
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Summarize the plan in one sentence.")),
  	},
  	Betas: []anthropic.AnthropicBeta{anthropic.AnthropicBetaMidConversationOutputConfig2026_07_01},
  })
  if err != nil {
  	log.Fatal(err)
  }

  for _, block := range response.Content {
  	if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.beta.AnthropicBeta;
  import com.anthropic.models.beta.messages.BetaMessage;
  import com.anthropic.models.beta.messages.BetaOutputConfig;
  import com.anthropic.models.beta.messages.BetaSystemMessageOutputConfig;
  import com.anthropic.models.beta.messages.BetaSystemMessageParam;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_FABLE_5_1)
          .maxTokens(4096L)
          .addBeta(AnthropicBeta.MID_CONVERSATION_OUTPUT_CONFIG_2026_07_01)
          .outputConfig(BetaOutputConfig.builder()
              .effort(BetaOutputConfig.Effort.HIGH)
              .build())
          .addUserMessage("Plan a migration from SQLite to PostgreSQL in three short steps.")
          .addAssistantMessage("1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts.")
          // 仅含 effort 的系统消息：新级别从下一个用户轮次起生效。
          .addMessage(BetaSystemMessageParam.builder()
              .contentOfBlocks(List.of())
              .outputConfig(BetaSystemMessageOutputConfig.builder()
                  .effort(BetaSystemMessageOutputConfig.Effort.LOW)
                  .build())
              .build())
          .addUserMessage("Summarize the plan in one sentence.")
          .build();

      BetaMessage response = client.beta().messages().create(params);
      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  use Anthropic\Beta\AnthropicBeta;
  use Anthropic\Beta\Messages\BetaMessageParam;
  use Anthropic\Beta\Messages\BetaOutputConfig;
  use Anthropic\Beta\Messages\BetaSystemMessageOutputConfig;
  use Anthropic\Beta\Messages\BetaTextBlock;
  use Anthropic\Client;
  use Anthropic\Messages\Model;

  $client = new Client();

  $response = $client->beta->messages->create(
      model: Model::CLAUDE_FABLE_5_1,
      maxTokens: 4096,
      outputConfig: BetaOutputConfig::with(effort: 'high'),
      messages: [
          BetaMessageParam::with(role: 'user', content: 'Plan a migration from SQLite to PostgreSQL in three short steps.'),
          BetaMessageParam::with(role: 'assistant', content: '1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts.'),
          // 仅含 effort 的系统消息：新级别从下一个用户轮次起生效。
          BetaMessageParam::with(
              role: 'system',
              content: [],
              outputConfig: BetaSystemMessageOutputConfig::with(effort: 'low'),
          ),
          BetaMessageParam::with(role: 'user', content: 'Summarize the plan in one sentence.'),
      ],
      betas: [AnthropicBeta::MID_CONVERSATION_OUTPUT_CONFIG_2026_07_01],
  );

  foreach ($response->content as $block) {
      if ($block instanceof BetaTextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_FABLE_5_1,
    max_tokens: 4096,
    output_config: {effort: :high},
    messages: [
      {role: "user", content: "Plan a migration from SQLite to PostgreSQL in three short steps."},
      {role: "assistant", content: "1. Export the SQLite data. 2. Create the PostgreSQL schema. 3. Import the data and verify row counts."},
      # 仅含 effort 的系统消息：新级别从下一个用户轮次起生效。
      {role: "system", content: [], output_config: {effort: :low}},
      {role: "user", content: "Summarize the plan in one sentence."}
    ],
    betas: [Anthropic::AnthropicBeta::MID_CONVERSATION_OUTPUT_CONFIG_2026_07_01]
  )

  response.content.each do |block|
    case block
    when Anthropic::Beta::BetaTextBlock
      puts block.text
    end
  end
  ```
</CodeGroup>

该消息的位置决定新级别何时开始生效：

* 紧跟在包含新输入的 `user` 轮次之后：Claude 对该轮次的回复会使用新的级别。
* 在其他任何位置，例如前面示例中那样位于一个 `assistant` 轮次与下一个 `user` 轮次之间：新的级别从下一个包含新输入的 `user` 轮次开始生效。

仅包含 `tool_result` 块的 `user` 轮次不算新输入。

新的级别会一直有效，直到后续消息更改它为止。

请将该消息放在您已发送的所有内容之后，并在后续请求中将其保留在原位。不要将其插入到您已发送的轮次之前，之后也不要移动或删除它。在支持 [preserved thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)（保留思考）的模型上，任何此类更改都会使该消息之后 `assistant` 轮次中的思考块失效。[API 如何处理无效块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#mismatch-behavior)说明了接下来会发生什么。

如果它与一条有内容的 `system` 消息相邻，则两条消息都必须遵循[对话中途系统消息的放置规则](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#limitations)。

使用 Amazon Bedrock InvokeModel API 时，请在请求体的 `anthropic_beta` 数组中发送 beta 值，而不是通过标头发送。有关在该平台上支持按消息设置 effort 的模型，请参阅 [Bedrock 上的对话中系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#mid-conversation-system-messages-on-bedrock)。

## 错误

* 如果没有 beta 标头，按消息设置的 `output_config` 会返回 400 错误：`messages.N.output_config: Extra inputs are not permitted`，其中 `N` 是该 `system` 消息在 `messages` 中的索引。
* 如果带有 beta 标头，不支持按消息设置 effort 的模型（例如 Claude Fable 5）会返回 400 错误：`output_config.effort requires a model that supports per-turn effort; this model does not`。在 Amazon Bedrock 上，这些模型以及 Claude Opus 5 则会返回 `Extra inputs are not permitted`。
* 在使用 `thinking: {"type": "between_tools"}` 的 Claude Sonnet 5.5 以及使用 `thinking: {"type": "disabled"}` 的 Claude Haiku 5.5 上，effort 级别无法在对话中途更改。在这些情况下，带有新级别的按消息设置的 `output_config.effort` 会返回 400 错误。要为每个轮次设置不同的 effort，请使用 adaptive thinking（自适应思考）。

## 按消息设置还是顶层设置 effort

相比在请求之间[更改顶层值](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#top-level-effort-on-the-next-request)，建议优先使用按消息设置 effort。按消息更改不会影响其之前的所有内容，因此缓存的前缀仍然匹配。更改顶层值会使缓存重新开始。在 Claude Fable 5.1 上，更改顶层值对模型的引导效果也不太可靠。模型之前的回复是在先前级别下生成的，它往往会与这些回复保持一致。

## 可用性

按消息设置 effort在部分模型上处于 beta 阶段。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态               | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 模型                                                                                                                                                                                                                                                                                                                                                     |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Beta             | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)                                                                                                                                                                               |
| Beta<sup>1</sup> | [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview) |
| Beta             | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                      | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)     |
| Beta             | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry (hosted on Anthropic)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                                                                                                                                     | [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview)                                                                                                                                                                                                                                                                 |
| Beta             | Claude API, [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry (hosted on Anthropic)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                                                                           | [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview)                                                                                                                                                                                                                                                                 |

1. 在请求正文的 `anthropic_beta` 数组中发送 beta 值。

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用
* 用法：改为通过覆盖智能体的模型设置来为会话设置 effort。

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `mid-conversation-output-config-2026-07-01`
