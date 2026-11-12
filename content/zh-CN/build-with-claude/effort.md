---
title: Effort
url: https://platform.claude.com/docs/zh-CN/build-with-claude/effort
description: 使用 effort 参数控制 Claude 在响应时使用的令牌数量，在响应的全面性与令牌效率之间进行权衡。
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
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
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

* **Messages API**：在请求中设置 `output_config.effort`。未设置值时，采用模型的默认级别。
* **Managed Agents**：在智能体的模型对象上或会话的模型覆盖中设置 `effort`。未设置值时，Managed Agents 会填入模型的默认值。

effort 参数让您可以控制 Claude 在响应请求时花费的令牌数量。您可以在单个模型上权衡响应的全面性与令牌效率。

<Tip>
  要了解 effort 如何与思考相互作用以及应该使用哪种控制方式，请参阅[思考与努力程度](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-effort)。在可使用自适应思考的情况下，effort 是控制思考深度的推荐方式。
</Tip>

## 设置 effort 级别

在请求中设置 `output_config.effort`。以下示例以 `medium` 努力程度运行一个请求。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
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
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --output-config '{effort: medium}' \
    --message '{role: user, content: "Analyze the trade-offs between microservices and monolithic architectures"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Analyze the trade-offs between microservices and monolithic architectures",
          }
      ],
      output_config={"effort": "medium"},
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

  const textBlock = response.content.find((block) => block.type === "text");
  console.log(textBlock?.text);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
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
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
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
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Analyze the trade-offs between microservices and monolithic architectures")
          .outputConfig(OutputConfig.builder()
              .effort(OutputConfig.Effort.MEDIUM)
              .build())
          .build();

      Message response = client.messages().create(params);
      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  use Anthropic\Client;
  use Anthropic\Messages\Model;
  use Anthropic\Messages\TextBlock;

  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          ['role' => 'user', 'content' => 'Analyze the trade-offs between microservices and monolithic architectures']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      outputConfig: ['effort' => 'medium'],
  );

  foreach ($message->content as $block) {
      if ($block instanceof TextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Analyze the trade-offs between microservices and monolithic architectures" }
    ],
    output_config: {
      effort: "medium"
    }
  )

  message.content.each do |block|
    case block
    when Anthropic::Models::TextBlock
      puts block.text
    end
  end
  ```
</CodeGroup>

## effort 的工作原理

大多数 Claude 模型默认使用高努力程度，会根据需要花费尽可能多的令牌以获得出色的结果；Claude Opus 5.5 和 Claude Haiku 5.5 默认使用 medium。您可以将努力程度提高到 `max` 以获得绝对最高的能力，也可以降低努力程度以更节省令牌用量，在接受一定能力下降的同时优化速度和成本。

<Tip>
  将 `effort` 设置为模型的默认值（Claude Opus 5.5 和 Claude Haiku 5.5 上为 `"medium"`，其他模型上为 `"high"`）与完全省略 `effort` 参数的行为完全相同。
</Tip>

effort 参数影响响应中的**所有令牌**，包括：

* 文本响应和解释
* 工具调用和函数参数
* 思考（启用时）

由于 effort 适用于每个输出令牌，因此无论是否启用思考它都能发挥作用。较低的努力程度也意味着更少、更简洁的工具调用。

### effort 级别

| 级别       | 描述                                                                                                                                                                                                                                                                                                      | 典型用例                              |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `max`    | 绝对最高能力，对令牌花费没有任何限制。适用于 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Mythos Preview、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 5.5、Claude Sonnet 5、Claude Sonnet 4.6 和 Claude Haiku 5.5。 | 需要最深入推理和最全面分析的任务                  |
| `xhigh`  | 面向长周期工作的扩展能力。适用于 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Opus 4.7、Claude Sonnet 5.5、Claude Sonnet 5 和 Claude Haiku 5.5。                                                               | 令牌预算达数百万的长时间运行（超过 30 分钟）的智能体和编码任务 |
| `high`   | 根据任务需要花费尽可能多的令牌以获得出色的结果。是除 Claude Opus 5.5 和 Claude Haiku 5.5 之外所有支持 effort 的模型的默认值。                                                                                                                                                                                                                    | 复杂推理、困难的编码问题、智能体任务                |
| `medium` | 均衡的方法，可适度节省令牌。是 Claude Opus 5.5 和 Claude Haiku 5.5 的默认值。                                                                                                                                                                                                                                                | 需要在速度、成本和性能之间取得平衡的智能体任务           |
| `low`    | 效率最高。大幅节省令牌，但能力有所下降。                                                                                                                                                                                                                                                                                    | 需要最佳速度和最低成本的较简单任务，例如子智能体          |

并非所有支持 `max` 的模型都支持 `xhigh`。

<Note>
  effort 是一种行为信号，而不是严格的令牌预算。在较低的努力程度下，Claude 在面对足够困难的问题时仍会思考，但对于同一问题，思考量会少于较高努力程度时。
</Note>

以下针对各模型的建议在与此表不同时以建议为准。

## 推荐的 effort 级别

**[Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview) & [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview)** — *For demanding reasoning and long-horizon agentic work*

Claude Fable 5.2 支持全部五个 effort 级别。API 默认值为 `high`。自适应思考始终开启，因此 effort 是您控制思考深度、延迟和成本的主要手段。从 `high` 开始。大部分能力提升在 `high` 级别就已具备，高于该级别后的提升并不均匀，且取决于具体任务。对于最困难的长时间运行的编码和智能体工作，在您已测得有提升的情况下，可提升至 `xhigh`。在您的评估表明质量能够保持的情况下，可降低至 `medium` 或 `low`。请将 `max` 保留用于衡量模型的能力上限，以及时间和令牌都无关紧要的少见任务。在 `max` 下，模型会不计令牌成本地优化能力。其运行速度比 `xhigh` 更慢、成本更高，而提升往往很小。请在您自己的评估上进行 effort 扫描，而不是沿用 Claude Fable 5.1 的设置。在 `high` 及以上级别时，请设置较大的 `max_tokens`。它是总输出（思考加响应文本）的硬性上限。Effort 几乎不会改变模型回复的长度，因此请通过提示来改变回复的长度和语气。相同的建议同样适用于 Claude Mythos 5.2。

在您的平台支持的情况下，Claude Fable 5.2 和 Claude Mythos 5.2 还支持通过每条消息的 `output_config` [在对话中途更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)，这会保留提示缓存。

See also: [为 Claude Fable 5.2 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#calibrate-effort)

**[Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)** — *For long-running agentic coding and knowledge work*

Claude Opus 5.5 支持全部五个 effort 级别，默认值为 `medium`（Claude Opus 5 及更早的 Opus 模型默认为 `high`，因此省略 `effort` 的请求运行的级别比在 Claude Opus 5 上低一级）。自适应思考始终开启且无法关闭，因此 effort 是控制模型推理量和请求成本的主要手段。请在您自己的评估上进行 effort 扫描，而不是沿用早期模型的设置；并在较高级别时设置较大的 `max_tokens`：它是总输出（思考加响应文本）的硬性上限。设置 `thinking: {"type": "disabled"}` 的请求在任何 effort 级别都会返回 400 错误。Claude Opus 5.5 还支持通过每条消息的 `output_config` [在对话中途更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)，这会保留提示缓存。

See also: [为 Claude Opus 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5)

**[Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview)** — *The best combination of speed and intelligence*

Claude Sonnet 5.5 支持全部五个 effort 级别，在 Claude API 上默认值为 `high`。其级别经过重新校准，因此某一级别产生的思考量与 Claude Sonnet 5 上同一级别并不相同。请在评估上重新进行 effort 扫描，而不是沿用您在 Claude Sonnet 5 上使用的设置。除非您的工作负载是智能体类或对延迟敏感的，否则请从 `high` 开始。对于智能体编码和多步骤工具使用，定义明确的任务从 `medium` 开始，较难或较长的任务则改用 `high`。对于聊天和其他对延迟敏感的工作，从 `medium` 或 `low` 开始。只有在您的评估显示质量有所提升时，才使用 `xhigh` 或 `max`。设置 `max_tokens` 时，请为思考和回复预留空间。即使不返回思考内容，思考也会计入 `max_tokens`。对于智能体编码，请将 `max_tokens` 设置为模型的最大值 128,000，并对响应进行[流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)。

要关闭前置思考，请发送 `thinking: {"type": "between_tools"}` 而不是 `"disabled"`。这是 Claude Sonnet 5.5 上最低的思考设置，适用于 `low`、`medium` 和 `high` effort。在 `xhigh` 或 `max` 下，使用该设置的请求会返回 400 错误，因此在这些级别请使用自适应思考：省略 `thinking` 字段或发送 `thinking: {"type": "adaptive"}`。请参阅[在没有前置思考的情况下运行](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#running-without-up-front-thinking)。

在您的平台支持的情况下，Claude Sonnet 5.5 还支持通过每条消息的 `output_config` [在对话中途更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)，这会保留提示缓存。使用 `between_tools` 时，effort 无法在对话中途更改：与当前生效级别不同的每条消息 `output_config.effort` 会返回 400 错误。要为每个轮次调整 effort，请使用自适应思考。

See also: [为 Claude Sonnet 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#calibrate-effort)

**[Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)** — *For high-volume, latency-sensitive tasks such as classification, extraction, and routing*

Claude Haiku 5.5 支持全部五个 effort 级别，默认值为 `medium`。Effort 是控制模型思考量的主要手段，进而影响质量、延迟和成本。对于大多数工作（包括智能体编码），**从 `medium` 开始**。对于聊天、简短的工具任务以及简单的高吞吐量请求，使用最便宜、最快的级别 `low`。在较长的智能体提示中，模型在 `low` 下更可能跳过搜索、提前停止或跳过检查。对于知识工作、较长的智能体任务和严格的指令遵循，使用 `high`。只有在您的评估显示质量有所提升时，才使用 `xhigh` 或 `max`，并在性能、成本和速度方面将其与 Claude Sonnet 5.5 进行比较。思考默认开启并计入 `max_tokens`，因此请为其预留空间。

要减少思考，请降低 effort 级别。您也可以在 `high` 及以下 effort 时发送 `thinking: {"type": "disabled"}`。在 `xhigh` 或 `max` 下，该设置会返回 400 错误，因此在这些级别请使用自适应思考：省略 `thinking` 字段或发送 `thinking: {"type": "adaptive"}`。

在您的平台支持的情况下，Claude Haiku 5.5 还允许您通过每条消息的 `output_config` [在对话中途更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)，这会保留提示缓存。使用 `thinking: {"type": "disabled"}` 时，effort 无法在对话中途更改：与当前生效级别不同的每条消息 `output_config.effort` 会返回 400 错误。要为每个轮次调整 effort，请使用自适应思考。

See also: [为 Claude Haiku 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5#use-effort-to-control-thinking)

**[Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview) & [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview)**

Claude Fable 5.1 支持全部五个 effort（努力程度）级别。**从默认值 `high` 开始。** 对于对能力最敏感的智能体和编码工作，可提升至 `xhigh` 或 `max`；当您的评估表明质量能够保持时，对于常规或对延迟敏感的工作，可降低至 `medium` 或 `low`。在 `high` 及以上级别时，请设置较大的 `max_tokens`。它是总输出（思考加响应文本）的硬性上限。

Claude Fable 5.1 还支持通过每条消息的 `output_config` [在对话中途更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)，这会保留提示缓存。

See also: [为 Claude Fable 5.1 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#consider-all-effort-levels)

**[Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview) & [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview)**

Effort 是在 Claude Fable 5 上权衡智能、延迟和成本的主要控制手段。**对于大多数任务，从默认值 `high` 开始**；对于对能力最敏感的工作负载，使用 `xhigh`；对于常规工作，则降低至 `medium` 或 `low`。Claude Fable 5 上较低的 effort 设置仍然表现良好，并且通常超过先前模型在 `xhigh` 下的表现。在 `high` 和 `xhigh` 级别时，请设置较大的 `max_tokens`。它是总输出（思考加响应文本）的硬性上限。请参阅[成本控制](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#cost-control)。

如果任务能够完成但耗时超出必要，或者您希望获得更快、更具交互性的工作方式，请降低 effort。

See also: [为 Claude Fable 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5)

**[Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview)**

Claude Opus 5 支持全部五个 effort 级别。**从默认值 `high` 开始**，并根据您的评估进行调整：对于要求较高的编码和智能体工作，提升至 `xhigh`；当任务值得不受限制地消耗令牌时，提升至 `max`。在评估表明质量能够保持的情况下，请大胆使用 `low` 和 `medium`，作为控制令牌成本和响应时间的主要手段。如果您沿用了早期模型的 effort 设置，请在评估上重新进行 effort 扫描，而不是直接复用。

Effort 控制的是思考量，而不是可见响应的长度：在 Claude Opus 5 上，更改 effort 并不能可靠地缩短响应，因此请改为[通过提示控制长度](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5#response-length-and-verbosity)。

API 默认值为 `high`。要使用其他级别，请显式设置 `effort`。您传入的值会覆盖默认值。

在 Claude Opus 5 上，在 `xhigh` 或 `max` effort 下无法禁用思考：在这些级别设置 `thinking: {"type": "disabled"}` 的请求会返回 400 错误。请参阅[结合思考使用 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-with-thinking)。

在 `xhigh` 或 `max` effort 下运行 Claude Opus 5 时，请设置较大的 `max_tokens`，以便模型有空间在子智能体和工具调用之间进行思考和操作。从 64k 令牌开始并在此基础上调整是一个合理的默认做法。

在您的平台支持的情况下，Claude Opus 5 还支持通过每条消息的 `output_config` [在对话中途更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)，这会保留提示缓存。

See also: [为 Claude Opus 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5)

**[Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)**

适用于 Claude Opus 4.7 的指导同样适用于 Claude Opus 4.8。**对于编码和智能体用例，从 `xhigh` 开始**；对于大多数其他对智能敏感的工作负载，使用 `high`；只有在您已测得较低级别能在评估中保持质量时，才降低至 `medium` 或 `low`。

API 默认值为 `high`。要使用其他级别，请显式设置 `effort`。您传入的值会覆盖默认值。

在 `xhigh` 或 `max` effort 下运行 Claude Opus 4.8 时，请设置较大的 `max_tokens`，以便模型有空间在子智能体和工具调用之间进行思考和操作。从 64k 令牌开始并在此基础上调整是一个合理的默认做法。

See also: [为 Claude Opus 4.8 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-4-8)

**[Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview)**

**对于编码和智能体用例，从 `xhigh` 开始**，并将 `high` 作为大多数对智能敏感的工作负载的最低级别。对于对成本敏感的工作负载，降低至 `medium`；只有当您的评估表明在 `xhigh` 下仍有可衡量的提升空间时，才提升至 `max`。

API 默认值为 `high`。要使用 `xhigh`，请显式设置 `effort`。您传入的值会覆盖默认值。

* **Low effort：** 高效，但最适合简短、范围明确的任务。如果您的任务包含多个部分，请将 `low` 与明确的检查清单搭配使用。
* **Medium effort：** 适用于一般工作流程，在降低成本的同时获得良好结果。
* **High effort：** 适用于仍需在智能和令牌消耗之间取得平衡的高级用例。这通常是质量与令牌效率之间的最佳平衡点。
* **Xhigh effort：** 编码和智能体工作的推荐起点，也适用于探索性任务，例如重复的工具调用、详细的网络搜索和知识库搜索。预计令牌使用量会明显高于 `high`。
* **Max effort：** 仅用于前沿问题。在大多数工作负载上，`max` 会以显著的成本换取相对较小的质量提升；在某些结构化输出或对智能不太敏感的任务上，它可能导致过度思考。

与 Claude Opus 4.6 相比，Claude Opus 4.7 还会更严格地遵循 effort 级别，尤其是在 `low` 和 `medium` 下。在较低的 effort 级别下，模型会将其工作范围限定在所要求的内容上，而不会做超出要求的事情。如果您在 Claude Opus 4.7 上处理复杂问题时发现推理较浅，请提高 effort，而不是通过提示来绕开。如果出于延迟考虑必须保持较低的 effort，请添加有针对性的指导，例如"This task involves multistep reasoning. Think carefully before responding."

在 `xhigh` 或 `max` effort 下运行 Claude Opus 4.7 时，请设置较大的 `max_tokens`，以便模型有空间在子智能体和工具调用之间进行思考和操作。从 64k 令牌开始并在此基础上调整是一个合理的默认做法。

See also: [提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)

**[Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview)**

Claude Sonnet 5 在 Claude API 和 Claude Code 上默认使用 `high` effort。

* **High effort（默认）：** 适用于质量比速度或成本更重要的复杂推理、编码和智能体任务。
* **Xhigh effort：** 适用于最困难的编码和智能体任务。
* **Medium effort：** 从默认值降级以节省成本。相当于 high effort 下的 Claude Sonnet 4.6。
* **Low effort：** 适用于高吞吐量或对延迟敏感的工作负载。适合优先考虑更快响应的聊天和非编码用例。
* **Max effort：** 适用于需要绝对最高能力且对令牌消耗没有限制的任务。

See also: [为 Claude Sonnet 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5#calibrating-effort-and-thinking-depth)

**[Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview)**

Sonnet 4.6 默认使用 `high` effort。使用 Sonnet 4.6 时请显式设置 effort，以避免意外的延迟：

* **Medium effort（推荐默认值）：** 对于大多数应用，在速度、成本和性能之间取得最佳平衡。适用于智能体编码、大量使用工具的工作流程和代码生成。
* **Low effort：** 适用于高吞吐量或对延迟敏感的工作负载。适合优先考虑更快响应的聊天和非编码用例。
* **High effort：** 适用于质量比速度或成本更重要的复杂推理和任务。
* **Max effort：** 适用于需要绝对最高能力且对令牌消耗没有限制的任务。

See also: [提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)

## effort 与工具使用

使用工具时，effort 参数既影响围绕工具调用的解释，也影响工具调用本身。较低的努力程度往往会：

* 将多个操作合并为更少的工具调用
* 进行更少的工具调用
* 不加铺垫直接采取行动
* 完成后使用简洁的确认消息

较高的努力程度可能会：

* 进行更多的工具调用
* 在采取行动前解释计划
* 提供详细的更改摘要
* 包含更全面的代码注释

## effort 与思考

`thinking` 参数控制 Claude 在回答之前是否在[思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)中进行思考；`effort` 参数控制 Claude 在整个响应中投入多少工作量，在自适应模式下，这包括它思考的频率和深度。不要将 `adaptive` 作为 `effort` 的值传递：`adaptive` 是一种思考模式，而不是努力程度级别。

在较高的努力程度下，Claude 会更主动、更长时间地思考。在工具使用循环中，仅处理工具结果的后续请求在任何级别下仍可能跳过思考。在较低的级别下，Claude 对于较简单的问题可能会完全跳过思考。有关这两种控制如何协同工作的完整指导，请参阅[思考与努力程度](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-effort)。

在 Claude Opus 4.5（唯一支持 effort 的仅扩展思考模型）上，它与 [`budget_tokens`](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking) 配合使用：为您的任务设置努力程度，然后根据任务所需的推理深度设置思考令牌预算。

有关各模型的思考可用性，请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#supported-models)。无论是否启用思考，effort 都能发挥作用。请参阅[effort 的工作原理](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#how-effort-works)。

## 在对话中途更改 effort

您可以通过两种方式以不同的努力程度运行对话的后续轮次：按消息更改努力程度，这样可以保留提示缓存；或在下一个请求中设置新的顶层值，这会重新开始缓存。

### 按消息设置 effort

[按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)通过 `messages` 中的一条 `system` 消息设置新级别。

### 在下一个请求中设置顶层 effort

顶层 `output_config.effort` 适用于整个请求。要以不同的级别运行对话的后续部分，请在下一个请求中设置新值。由于顶层努力程度会影响渲染后的提示，因此在请求之间更改它不会保留早期轮次的缓存前缀。如果您在长会话中依赖[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)，并且您的模型不支持按消息设置努力程度，请在开始时选择一个努力程度并保持不变。

## 最佳实践

1. **显式设置努力程度：**&#x41;PI 默认为 `high`（Claude Opus 5.5 和 Claude Haiku 5.5 上为 `medium`），但合适的起点取决于您的模型和工作负载。
2. **对于对速度敏感或简单的任务使用 low：**&#x5F53;延迟很重要或任务较为简单时，低努力程度可以显著减少响应时间和成本。
3. **测试您的用例：**&#x52AA;力程度的影响因任务类型而异。在部署之前，请评估其在您特定用例上的表现。
4. **考虑动态努力程度：**&#x6839;据任务复杂度调整努力程度。简单查询可能适合低努力程度，而智能体编码和复杂推理则受益于高努力程度。在单个对话中改变努力程度之前，请先参阅下一条。
5. **在缓存的对话中保持顶层努力程度不变：**&#x5728;请求之间更改顶层努力程度值会使[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)失效，因此请在不同工作负载之间改变它，而不是在依赖缓存命中的对话中改变。在支持的模型上，请改用[按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) 更改，这样可以保留缓存。请参阅[思考与提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-prompt-caching)。

## 后续步骤

<CardGroup>
  <Card title="任务预算" icon="gauge" href="https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets">
    为 Claude 提供覆盖完整智能体循环的建议性令牌预算，帮助模型在长时间的智能体任务中进行自我调节。
  </Card>

  <Card title="引导思考" icon="compass" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost">
    了解自适应思考（由 Claude 决定何时思考以及思考多少），并通过努力程度和提示对其进行引导。
  </Card>

  <Card title="思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    了解思考的工作原理、Claude 默认何时思考，以及思考如何与努力程度相互作用。
  </Card>
</CardGroup>

## 可用性

Effort在部分模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：始终开启
* 用法：在请求中设置 `output_config.effort`。未设置值时，采用模型的默认级别。

| 状态 | 云平台     | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| -- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | 所有支持的平台 | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview), [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启
* 用法：在智能体的模型对象上或会话的模型覆盖中设置 `effort`。未设置值时，Managed Agents 会填入模型的默认值。

| 状态 | 云平台                                                                                                                   | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| -- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview), [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目 | 零数据保留                                                                                                                                     | HIPAA |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态 | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格  |

适用的端点：

* `/v1/messages` 使用 `effort`
