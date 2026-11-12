---
title: 在令牌阈值处进行压缩
url: https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold
description: 当对话达到您设置的令牌阈值时，让 API 在普通请求中自动总结较早的上下文。
featureMetadata:
  status: beta
  betaHeader: compact-2026-01-12
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 服务器端压缩结果通过 API 响应以无状态方式返回和往返传递。
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
  supportedPlatforms:
    Claude API: beta
    Amazon Bedrock: beta
    Amazon Bedrock (bedrock-runtime): beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Google managed): beta
    Microsoft Foundry: beta
---

**API 支持**：用法因产品而异。

* **Messages API**：通过 `compact_20260112` 编辑启用；默认在输入令牌达到 150,000 时开始压缩。
* **Managed Agents**：始终开启。它会压缩会话的对话历史，使其适合上下文窗口。

“Threshold compaction”（阈值压缩）是一种自动压缩方式：您在普通请求上设置一个令牌阈值，一旦达到该阈值，API 就会在请求进行过程中总结较早的上下文。它与按需压缩并存，按需压缩由您决定何时生成摘要（请参阅[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)）。要在两者之间做出选择，请参阅[选择压缩方式](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction#choose-how-to-compact)。

压缩通过在接近 “context window”（上下文窗口）限制时自动总结较早的上下文，为长时间运行的对话和任务扩展有效上下文长度。它还能使活动上下文保持精简：随着对话增长，响应质量会下降，因此压缩会用简洁的摘要替换较早的内容。

<Tip>
  要深入了解长上下文为何会导致质量下降以及压缩如何提供帮助，请参阅 [有效的上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)。
</Tip>

这非常适用于：

* 基于聊天的多轮对话，您希望用户能够长时间使用同一个聊天
* 需要大量后续工作（通常是 “tool use”（工具使用））且可能超出上下文窗口的面向任务的提示

## 压缩的工作原理

启用压缩后，当对话达到配置的令牌阈值时，Claude 会自动总结您的对话。API 会：

1. 检测输入令牌何时达到您指定的触发阈值。
2. 生成当前对话的摘要。
3. 创建一个包含该摘要的 `compaction` 块。
4. 使用压缩后的上下文继续生成响应。

在后续请求中，将响应追加到您的消息中。API 会自动丢弃 `compaction` 块之前的所有内容块，从摘要处继续对话。

<Frame>
  ![Compaction flow（压缩流程）：当输入令牌达到触发阈值时，Claude 将摘要写入压缩块并继续](https://platform.claude.com/docs/zh-CN/images/compaction-flow.svg)
</Frame>

## 基本用法

通过在 Messages API 请求的 `context_management.edits` 中添加 `compact_20260112` 策略来启用压缩。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: compact-2026-01-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Help me build a website"
        }
      ],
      "context_management": {
        "edits": [
          {
            "type": "compact_20260112"
          }
        ]
      }
    }'
  ```

  ```bash CLI
  ant beta:messages create --beta compact-2026-01-12 <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: Help me build a website
  context_management:
    edits:
      - type: compact_20260112
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  messages = [{"role": "user", "content": "Help me build a website"}]

  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={"edits": [{"type": "compact_20260112"}]},
  )

  # 追加响应（包括任何压缩块）以继续对话
  messages.append({"role": "assistant", "content": response.content})
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Help me build a website" }
  ];

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112"
        }
      ]
    }
  });

  // 追加响应（包括任何压缩块）以继续对话
  messages.push({
    role: "assistant",
    content: response.content
  });
  ```

  ```csharp C#
  AnthropicClient client = new();

  var messages = new List<BetaMessageParam>
  {
      new BetaUserMessageParam { Content = "Help me build a website" }
  };

  var parameters = new MessageCreateParams
  {
      Betas = ["compact-2026-01-12"],
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit()]
      }
  };

  var response = await client.Beta.Messages.Create(parameters);

  // 追加响应（包括任何压缩块）以继续对话
  messages.Add(new BetaAssistantMessageParam
  {
      Content = response.Content.Select(block => new BetaContentBlockParam(block.Json)).ToList()
  });

  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  messages := []anthropic.BetaMessageParam{
  	anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Help me build a website")),
  }

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages:  messages,
  	ContextManagement: anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
  		},
  	},
  	Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }

  // 追加响应（包括任何压缩块）以继续对话
  messages = append(messages, response.ToParam())

  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageCreateParams params = MessageCreateParams.builder()
              .addBeta("compact-2026-01-12")
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addUserMessage("Help me build a website")
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder().build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);

          // 将响应（包括任何压缩块）追加到对话中以继续对话
          // 方法是将其包含在下一个请求的 messages 中
          System.out.println(response);
  ```

  ```php PHP
  $client = new Client();

  $messages = [
      ['role' => 'user', 'content' => 'Help me build a website']
  ];

  $response = $client->beta->messages->create(
      maxTokens: 4096,
      messages: $messages,
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [
              ['type' => 'compact_20260112']
          ]
      ]
  );

  // 追加响应（包括任何压缩块）以继续对话
  $messages[] = ['role' => 'assistant', 'content' => $response->content];

  echo json_encode($response, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  messages = [
    { role: "user", content: "Help me build a website" }
  ]

  response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  )

  # 将响应（包括任何压缩块）追加到消息中，以继续对话
  messages << { role: "assistant", content: response.content }

  puts response
  ```
</CodeGroup>

## 参数

| 参数                       | 类型      | 默认值                                         | 描述                                                         |
| ------------------------ | ------- | ------------------------------------------- | ---------------------------------------------------------- |
| `type`                   | string  | 必需                                          | 必须为 `"compact_20260112"`                                   |
| `trigger`                | object  | `{"type": "input_tokens", "value": 150000}` | 何时触发压缩。`input_tokens` 是唯一支持的触发类型。`value` 必须至少为 50,000 个令牌。 |
| `pause_after_compaction` | boolean | `false`                                     | 生成压缩摘要后是否暂停                                                |
| `instructions`           | string  | `null`                                      | 自定义摘要提示。提供时会完全替换默认提示。                                      |

### 触发配置

使用 `trigger` 参数配置压缩的触发时机：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: compact-2026-01-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Hello, Claude"
        }
      ],
      "context_management": {
        "edits": [
          {
            "type": "compact_20260112",
            "trigger": {
              "type": "input_tokens",
              "value": 150000
            }
          }
        ]
      }
    }'
  ```

  ```bash CLI
  ant beta:messages create --beta compact-2026-01-12 <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: Hello, Claude
  context_management:
    edits:
      - type: compact_20260112
        trigger:
          type: input_tokens
          value: 150000
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]
  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={
          "edits": [
              {
                  "type": "compact_20260112",
                  "trigger": {"type": "input_tokens", "value": 150000},
              }
          ]
      },
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Hello, Claude" }
  ];

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          trigger: {
            type: "input_tokens",
            value: 150000
          }
        }
      ]
    }
  });
  ```

  ```csharp C#
  AnthropicClient client = new();
  List<BetaMessageParam> messages = [new BetaUserMessageParam { Content = "Hello" }];

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Betas = ["compact-2026-01-12"],
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit
          {
              Trigger = new BetaInputTokensTrigger(150000)
          }]
      }
  };

  var message = await client.Beta.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()
  messages := []anthropic.BetaMessageParam{anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude"))}

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages:  messages,
  	ContextManagement: anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{
  				Trigger: anthropic.BetaInputTokensTriggerParam{Value: 150000},
  			}},
  		},
  	},
  	Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  import com.anthropic.models.beta.messages.BetaInputTokensTrigger;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageCreateParams params = MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addBeta("compact-2026-01-12")
              .addUserMessage("Hello, Claude")
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder()
                      .trigger(BetaInputTokensTrigger.builder()
                          .value(150000L)
                          .build())
                      .build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);
          System.out.println(response);
  ```

  ```php PHP
  $client = new Client();
  $messages = [['role' => 'user', 'content' => 'Hello, Claude']];

  $message = $client->beta->messages->create(
      maxTokens: 4096,
      messages: $messages,
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [
              [
                  'type' => 'compact_20260112',
                  'trigger' => [
                      'type' => 'input_tokens',
                      'value' => 150000
                  ]
              ]
          ]
      ]
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = [{ role: "user", content: "Hello, Claude" }]

  response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          trigger: {
            type: "input_tokens",
            value: 150000
          }
        }
      ]
    }
  )
  puts response
  ```
</CodeGroup>

### 自定义摘要指令

默认摘要提示因模型而异。每个默认提示都会指示 Claude 在 `<summary></summary>` 标签内编写摘要，其中包含在未来的上下文窗口中继续任务所需的信息。例如，某些模型使用以下提示：

```text wrap
You have written a partial transcript for the initial task above. Please write a summary of the transcript. The purpose of this summary is to provide continuity so you can continue to make progress towards solving the task in a future context, where the raw history above may not be accessible and will be replaced with this summary. Write down anything that would be helpful, including the state, next steps, learnings etc. You must wrap your summary in a <summary></summary> block.
```

您可以通过 `instructions` 参数提供自定义指令。自定义指令不会补充默认提示，而是完全替换它：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: compact-2026-01-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Hello, Claude"
        }
      ],
      "context_management": {
        "edits": [
          {
            "type": "compact_20260112",
            "instructions": "Focus on preserving code snippets, variable names, and technical decisions."
          }
        ]
      }
    }'
  ```

  ```bash CLI
  ant beta:messages create --beta compact-2026-01-12 <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: Hello, Claude
  context_management:
    edits:
      - type: compact_20260112
        instructions: >-
          Focus on preserving code snippets, variable names, and
          technical decisions.
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]
  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={
          "edits": [
              {
                  "type": "compact_20260112",
                  "instructions": "Focus on preserving code snippets, variable names, and technical decisions.",
              }
          ]
      },
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Hello, Claude" }
  ];

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          instructions:
            "Focus on preserving code snippets, variable names, and technical decisions."
        }
      ]
    }
  });
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Betas = ["compact-2026-01-12"],
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages =
      [
          new BetaUserMessageParam { Content = "Help me build a Python web scraper" },
          new BetaAssistantMessageParam { Content = "I'll help you build a web scraper..." },
          new BetaUserMessageParam { Content = "Add support for JavaScript-rendered pages" }
      ],
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit
          {
              Instructions = "Focus on preserving code snippets, variable names, and technical decisions."
          }]
      }
  };

  var message = await client.Beta.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Help me build a Python web scraper")),
  		{OfAssistant: &anthropic.BetaAssistantMessageParam{Content: []anthropic.BetaContentBlockParamUnion{anthropic.NewBetaTextBlock("I'll help you build a web scraper...")}}},
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Add support for JavaScript-rendered pages")),
  	},
  	ContextManagement: anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{
  				Instructions: anthropic.String("Focus on preserving code snippets, variable names, and technical decisions."),
  			}},
  		},
  	},
  	Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageCreateParams params = MessageCreateParams.builder()
              .addBeta("compact-2026-01-12")
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addUserMessage("Help me build a Python web scraper")
              .addAssistantMessage("I'll help you build a web scraper...")
              .addUserMessage("Add support for JavaScript-rendered pages")
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder()
                      .instructions("Focus on preserving code snippets, variable names, and technical decisions.")
                      .build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);
          System.out.println(response);
  ```

  ```php PHP
  $client = new Client();

  $response = $client->beta->messages->create(
      maxTokens: 4096,
      messages: [
          ['role' => 'user', 'content' => 'Help me build a Python web scraper'],
          ['role' => 'assistant', 'content' => "I'll help you build a web scraper..."],
          ['role' => 'user', 'content' => 'Add support for JavaScript-rendered pages']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [
              [
                  'type' => 'compact_20260112',
                  'instructions' => 'Focus on preserving code snippets, variable names, and technical decisions.'
              ]
          ]
      ]
  );

  echo json_encode($response, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Help me build a Python web scraper" },
      { role: "assistant", content: "I'll help you build a web scraper..." },
      { role: "user", content: "Add support for JavaScript-rendered pages" }
    ],
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          instructions:
            "Focus on preserving code snippets, variable names, and technical decisions."
        }
      ]
    }
  )

  puts response
  ```
</CodeGroup>

在某些模型上，带有自定义 `instructions` 的请求仅根据可见对话进行总结，不包含较早的思考块。这些模型是 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5。

### 压缩后暂停

使用 `pause_after_compaction` 可在生成压缩摘要后暂停 API。这样您就可以在 API 继续生成响应之前添加额外的内容块（例如保留最近的消息或特定的指令类消息）。

启用后，API 在生成压缩块后会返回一条停止原因为 `compaction` 的消息：

<CodeGroup>
  ```bash cURL
  # pause_after_compaction 会在生成压缩（compaction）摘要后立即停止响应，
  # 以便您在继续之前调整消息。继续这一步
  # 不便用一次性 shell 命令实现；完整的暂停并继续流程
  # 请参阅 SDK 选项卡。单个暂停请求：
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: compact-2026-01-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Hello, Claude"
        }
      ],
      "context_management": {
        "edits": [
          {
            "type": "compact_20260112",
            "pause_after_compaction": true
          }
        ]
      }
    }'
  ```

  ```bash CLI
  # pause_after_compaction 会在生成 compaction（压缩）摘要后立即停止响应，
  # 以便您在继续之前调整消息。“继续”
  # 步骤不便用一次性 CLI 命令实现；请参阅 SDK 选项卡了解
  # 完整的暂停并继续流程。单个暂停请求：
  ant beta:messages create --beta compact-2026-01-12 --format jsonl <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: Hello, Claude
  context_management:
    edits:
      - type: compact_20260112
        pause_after_compaction: true
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]
  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={
          "edits": [{"type": "compact_20260112", "pause_after_compaction": True}]
      },
  )

  # 检查 compaction（压缩）是否触发了暂停
  if response.stop_reason == "compaction":
      # 响应仅包含压缩块
      messages.append({"role": "assistant", "content": response.content})

      # 继续该请求
      response = client.beta.messages.create(
          betas=["compact-2026-01-12"],
          model="claude-opus-5-5",
          max_tokens=4096,
          messages=messages,
          context_management={"edits": [{"type": "compact_20260112"}]},
      )
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Hello, Claude" }
  ];

  let response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          pause_after_compaction: true
        }
      ]
    }
  });

  // 检查 compaction（压缩）是否触发了暂停
  if (response.stop_reason === "compaction") {
    // 响应仅包含压缩块
    messages.push({
      role: "assistant",
      content: response.content
    });

    // 继续该请求
    response = await client.beta.messages.create({
      betas: ["compact-2026-01-12"],
      model: "claude-opus-5-5",
      max_tokens: 4096,
      messages,
      context_management: {
        edits: [{ type: "compact_20260112" }]
      }
    });
  }
  ```

  ```csharp C#
  var client = new AnthropicClient();
  var messages = new List<BetaMessageParam>
  {
      new BetaUserMessageParam { Content = "Hello, Claude" }
  };

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Betas = ["compact-2026-01-12"],
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit
          {
              PauseAfterCompaction = true
          }]
      }
  };

  var response = await client.Beta.Messages.Create(parameters);

  if (response.StopReason == BetaStopReason.Compaction)
  {
      messages.Add(new BetaAssistantMessageParam
      {
          Content = response.Content.Select(block => new BetaContentBlockParam(block.Json)).ToList()
      });

      parameters = new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 4096,
          Betas = ["compact-2026-01-12"],
          Messages = messages,
          ContextManagement = new BetaContextManagementConfig
          {
              Edits = [new BetaCompact20260112Edit()]
          }
      };

      response = await client.Beta.Messages.Create(parameters);
  }

  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()
  messages := []anthropic.BetaMessageParam{anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude"))}

  compactEdit := anthropic.BetaContextManagementConfigParam{
  	Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  		{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{
  			PauseAfterCompaction: anthropic.Bool(true),
  		}},
  	},
  }

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:             anthropic.ModelClaudeOpus5_5,
  	MaxTokens:         4096,
  	Messages:          messages,
  	ContextManagement: compactEdit,
  	Betas:             []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "compaction" {
  	messages = append(messages, response.ToParam())

  	response, err = client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 4096,
  		Messages:  messages,
  		ContextManagement: anthropic.BetaContextManagementConfigParam{
  			Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  				{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
  			},
  		},
  		Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}
  }

  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  import com.anthropic.models.beta.messages.BetaStopReason;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageCreateParams params = MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addBeta("compact-2026-01-12")
              .addUserMessage("Help me build a website")
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder()
                      .pauseAfterCompaction(true)
                      .build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);

          // 检查 compaction（压缩）是否触发了暂停
          if (response.stopReason().isPresent()
                  && response.stopReason().get().equals(BetaStopReason.COMPACTION)) {
              // 追加压缩块并继续请求
              // 方法是使用压缩后的上下文构建新请求
              MessageCreateParams continueParams = MessageCreateParams.builder()
                  .model(Model.CLAUDE_OPUS_5_5)
                  .maxTokens(4096L)
                  .addBeta("compact-2026-01-12")
                  .addUserMessage("Help me build a website")
                  .addMessage(response)
                  .contextManagement(BetaContextManagementConfig.builder()
                      .addEdit(BetaCompact20260112Edit.builder().build())
                      .build())
                  .build();

              response = client.beta().messages().create(continueParams);
          }

          System.out.println(response);
  ```

  ```php PHP
  $client = new Client();
  $messages = [['role' => 'user', 'content' => 'Hello, Claude']];

  $response = $client->beta->messages->create(
      maxTokens: 4096,
      messages: $messages,
      model: 'claude-opus-5-5',
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [
              [
                  'type' => 'compact_20260112',
                  'pause_after_compaction' => true
              ]
          ]
      ]
  );

  if ($response->stopReason === 'compaction') {
      $messages[] = [
          'role' => 'assistant',
          'content' => $response->content
      ];

      $response = $client->beta->messages->create(
          maxTokens: 4096,
          messages: $messages,
          model: 'claude-opus-5-5',
          betas: ['compact-2026-01-12'],
          contextManagement: [
              'edits' => [
                  ['type' => 'compact_20260112']
              ]
          ]
      );
  }

  echo $response;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = [{ role: "user", content: "Hello, Claude" }]

  response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          pause_after_compaction: true
        }
      ]
    }
  )

  if response.stop_reason == :compaction
    messages << { role: "assistant", content: response.content }

    response = client.beta.messages.create(
      betas: ["compact-2026-01-12"],
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 4096,
      messages: messages,
      context_management: {
        edits: [{ type: "compact_20260112" }]
      }
    )
  end

  puts response
  ```
</CodeGroup>

即使未生成摘要，API 也会暂停。此时 `compaction` 块的 `content` 为 `null`，因此在继续时请保留原始消息。如果安全保障措施拦截了摘要，`content` 同样为 `null`，且停止原因为 `refusal` 而不是 `compaction`。如果存在策略类别，[`stop_details`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#refusal) 会给出该策略类别。两种情况下的继续方式相同：查找 `compaction` 块，而不是检查 `stop_reason`，正如[保留最近消息的示例](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold#examples)所做的那样。

#### 强制执行总令牌预算

当模型处理包含大量工具使用迭代的长任务时，总令牌消耗可能会显著增长。您可以将 `pause_after_compaction` 与压缩计数器结合使用，以估算累计用量，并在达到预算后优雅地结束任务。

此示例仅以 SDK 语言提供：其价值在于请求周围的预算跟踪逻辑。原始请求将[触发配置](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold#trigger-configuration)中的 `trigger` 与[压缩后暂停](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold#pausing-after-compaction)中的 `pause_after_compaction` 结合使用。

<CodeGroup exclude="shell">
  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]
  TRIGGER_THRESHOLD = 100_000
  TOTAL_TOKEN_BUDGET = 3_000_000
  n_compactions = 0

  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={
          "edits": [
              {
                  "type": "compact_20260112",
                  "trigger": {"type": "input_tokens", "value": TRIGGER_THRESHOLD},
                  "pause_after_compaction": True,
              }
          ]
      },
  )

  if response.stop_reason == "compaction":
      n_compactions += 1
      messages.append({"role": "assistant", "content": response.content})

      # 估算已消耗的令牌总数；若超出预算，则提示收尾
      if n_compactions * TRIGGER_THRESHOLD >= TOTAL_TOKEN_BUDGET:
          messages.append(
              {
                  "role": "user",
                  "content": "Please wrap up your current work and summarize the final state.",
              }
          )
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Hello, Claude" }
  ];
  const TRIGGER_THRESHOLD = 100_000;
  const TOTAL_TOKEN_BUDGET = 3_000_000;
  let compactionCount = 0;

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          trigger: { type: "input_tokens", value: TRIGGER_THRESHOLD },
          pause_after_compaction: true
        }
      ]
    }
  });

  if (response.stop_reason === "compaction") {
    compactionCount += 1;
    messages.push({ role: "assistant", content: response.content });

    // 估算已消耗的令牌总数；若超出预算，则提示收尾
    if (compactionCount * TRIGGER_THRESHOLD >= TOTAL_TOKEN_BUDGET) {
      messages.push({
        role: "user",
        content: "Please wrap up your current work and summarize the final state."
      });
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();
  List<BetaMessageParam> messages = [new BetaUserMessageParam { Content = "Hello, Claude" }];

  const int TriggerThreshold = 100_000;
  const int TotalTokenBudget = 3_000_000;
  int compactionCount = 0;

  var response = await client.Beta.Messages.Create(new()
  {
      Betas = ["compact-2026-01-12"],
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit
          {
              Trigger = new BetaInputTokensTrigger(TriggerThreshold),
              PauseAfterCompaction = true
          }]
      }
  });

  if (response.StopReason == BetaStopReason.Compaction)
  {
      compactionCount += 1;
      messages.Add(new BetaAssistantMessageParam
      {
          Content = response.Content.Select(block => new BetaContentBlockParam(block.Json)).ToList()
      });

      // 估算已消耗的令牌总数；若超出预算，则提示收尾
      if (compactionCount * TriggerThreshold >= TotalTokenBudget)
      {
          messages.Add(new BetaUserMessageParam
          {
              Content = "Please wrap up your current work and summarize the final state."
          });
      }
  }

  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()
  messages := []anthropic.BetaMessageParam{anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude"))}

  const triggerThreshold = 100_000
  const totalTokenBudget = 3_000_000
  compactionCount := 0

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages:  messages,
  	ContextManagement: anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{
  				Trigger:              anthropic.BetaInputTokensTriggerParam{Value: triggerThreshold},
  				PauseAfterCompaction: anthropic.Bool(true),
  			}},
  		},
  	},
  	Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "compaction" {
  	compactionCount++
  	messages = append(messages, response.ToParam())

  	// 估算已消耗的令牌总数；若超出预算，则提示收尾
  	if compactionCount*triggerThreshold >= totalTokenBudget {
  		messages = append(messages, anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Please wrap up your current work and summarize the final state.")))
  	}
  }

  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  import com.anthropic.models.beta.messages.BetaInputTokensTrigger;
  import com.anthropic.models.beta.messages.BetaStopReason;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          long triggerThreshold = 100_000;
          long totalTokenBudget = 3_000_000;
          int compactionCount = 0;

          List<BetaMessageParam> messages = new ArrayList<>();
          messages.add(BetaMessageParam.ofUser("Hello, Claude"));

          MessageCreateParams params = MessageCreateParams.builder()
              .addBeta("compact-2026-01-12")
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .messages(messages)
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder()
                      .trigger(BetaInputTokensTrigger.builder()
                          .value(triggerThreshold)
                          .build())
                      .pauseAfterCompaction(true)
                      .build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);

          if (response.stopReason().isPresent()
                  && response.stopReason().get().equals(BetaStopReason.COMPACTION)) {
              compactionCount += 1;
              messages.add(response.toParam());

              // 估算已消耗的令牌总数；若超出预算，则提示收尾
              if (compactionCount * triggerThreshold >= totalTokenBudget) {
                  messages.add(BetaMessageParam.ofUser(
                      "Please wrap up your current work and summarize the final state."));
              }
          }

          System.out.println(response);
  ```

  ```php PHP
  $client = new Client();

  $triggerThreshold = 100_000;
  $totalTokenBudget = 3_000_000;
  $compactionCount = 0;

  $messages = [['role' => 'user', 'content' => 'Hello, Claude']];

  $response = $client->beta->messages->create(
      maxTokens: 4096,
      messages: $messages,
      model: 'claude-opus-5-5',
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [
              [
                  'type' => 'compact_20260112',
                  'trigger' => ['type' => 'input_tokens', 'value' => $triggerThreshold],
                  'pause_after_compaction' => true
              ]
          ]
      ]
  );

  if ($response->stopReason === 'compaction') {
      $compactionCount += 1;
      $messages[] = ['role' => 'assistant', 'content' => $response->content];

      // 估算已消耗的令牌总数；若超出预算，则提示收尾
      if ($compactionCount * $triggerThreshold >= $totalTokenBudget) {
          $messages[] = [
              'role' => 'user',
              'content' => 'Please wrap up your current work and summarize the final state.'
          ];
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = [{ role: "user", content: "Hello, Claude" }]
  TRIGGER_THRESHOLD = 100_000
  TOTAL_TOKEN_BUDGET = 3_000_000
  compaction_count = 0

  response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: messages,
    context_management: {
      edits: [
        {
          type: "compact_20260112",
          trigger: { type: "input_tokens", value: TRIGGER_THRESHOLD },
          pause_after_compaction: true
        }
      ]
    }
  )

  if response.stop_reason == :compaction
    compaction_count += 1
    messages << { role: "assistant", content: response.content }

    # 估算已消耗的令牌总数；若超出预算则提示收尾
    if compaction_count * TRIGGER_THRESHOLD >= TOTAL_TOKEN_BUDGET
      messages << {
        role: "user",
        content: "Please wrap up your current work and summarize the final state."
      }
    end
  end
  ```
</CodeGroup>

## 使用压缩块

触发压缩时，API 会在助手响应的开头返回一个 `compaction` 块。

长时间运行的对话可能会产生多次压缩。最后一个压缩块反映提示的最终状态，并用生成的摘要替换其之前的内容。

```json Output
{
  "content": [
    {
      "type": "compaction",
      "content": "Summary of the conversation: The user requested help building a web scraper..."
    },
    {
      "type": "text",
      "text": "Based on our conversation so far..."
    }
  ]
}
```

### 回传压缩块

您必须在后续请求中将 `compaction` 块回传给 API，才能使用缩短后的提示继续对话。最简单的方法是将整个响应内容追加到您的消息中：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]
  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={"edits": [{"type": "compact_20260112"}]},
  )
  # 收到包含压缩块的响应后
  messages.append({"role": "assistant", "content": response.content})

  # 继续对话
  messages.append({"role": "user", "content": "Now add error handling"})

  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={"edits": [{"type": "compact_20260112"}]},
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Hello, Claude" }
  ];

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  });

  // 收到包含 compaction（压缩）块的响应后
  messages.push({
    role: "assistant",
    content: response.content
  });

  // 继续对话
  messages.push({ role: "user", content: "Now add error handling" });

  const nextResponse = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  });
  ```

  ```csharp C#
  AnthropicClient client = new();

  var messages = new List<BetaMessageParam>
  {
      new BetaUserMessageParam { Content = "Help me build a web scraper" }
  };

  var response = await client.Beta.Messages.Create(new()
  {
      Betas = ["compact-2026-01-12"],
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit()]
      }
  });

  messages.Add(new BetaAssistantMessageParam
  {
      Content = response.Content.Select(block => new BetaContentBlockParam(block.Json)).ToList()
  });

  messages.Add(new BetaUserMessageParam { Content = "Now add error handling" });

  var nextResponse = await client.Beta.Messages.Create(new()
  {
      Betas = ["compact-2026-01-12"],
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit()]
      }
  });

  Console.WriteLine(nextResponse);
  ```

  ```go Go
  client := anthropic.NewClient()

  messages := []anthropic.BetaMessageParam{
  	anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Help me build a web scraper")),
  }

  compactEdit := anthropic.BetaContextManagementConfigParam{
  	Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  		{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
  	},
  }

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:             anthropic.ModelClaudeOpus5_5,
  	MaxTokens:         4096,
  	Messages:          messages,
  	ContextManagement: compactEdit,
  	Betas:             []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }

  messages = append(messages, response.ToParam())

  messages = append(messages, anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Now add error handling")))

  nextResponse, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:             anthropic.ModelClaudeOpus5_5,
  	MaxTokens:         4096,
  	Messages:          messages,
  	ContextManagement: compactEdit,
  	Betas:             []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(nextResponse)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          // 第一个请求
          BetaMessage response = client.beta().messages().create(
              MessageCreateParams.builder()
                  .addBeta("compact-2026-01-12")
                  .model(Model.CLAUDE_OPUS_5_5)
                  .maxTokens(4096L)
                  .addUserMessage("Help me build a web scraper")
                  .contextManagement(BetaContextManagementConfig.builder()
                      .addEdit(BetaCompact20260112Edit.builder().build())
                      .build())
                  .build());

          // 收到包含压缩块的响应后，追加完整的
          // 内容（包括压缩块）并继续对话
          BetaMessage nextResponse = client.beta().messages().create(
              MessageCreateParams.builder()
                  .addBeta("compact-2026-01-12")
                  .model(Model.CLAUDE_OPUS_5_5)
                  .maxTokens(4096L)
                  .addUserMessage("Help me build a web scraper")
                  .addMessage(response)
                  .addUserMessage("Now add error handling")
                  .contextManagement(BetaContextManagementConfig.builder()
                      .addEdit(BetaCompact20260112Edit.builder().build())
                      .build())
                  .build());

          System.out.println(nextResponse);
  ```

  ```php PHP
  $client = new Client();

  $messages = [
      ['role' => 'user', 'content' => 'Help me build a web scraper']
  ];

  $response = $client->beta->messages->create(
      maxTokens: 4096,
      messages: $messages,
      model: 'claude-opus-5-5',
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [['type' => 'compact_20260112']]
      ]
  );

  $messages[] = ['role' => 'assistant', 'content' => $response->content];

  $messages[] = ['role' => 'user', 'content' => 'Now add error handling'];

  $nextResponse = $client->beta->messages->create(
      maxTokens: 4096,
      messages: $messages,
      model: 'claude-opus-5-5',
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [['type' => 'compact_20260112']]
      ]
  );

  echo json_encode($nextResponse, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  messages = [
    { role: "user", content: "Help me build a web scraper" }
  ]

  response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  )

  messages << { role: "assistant", content: response.content }

  messages << { role: "user", content: "Now add error handling" }

  next_response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  )

  puts next_response.content
  ```
</CodeGroup>

在 Python 中，请像本页示例那样使用 `client.beta.messages`。如果您调用 `client.messages` 并自行序列化块，普通的 `model_dump()` 会向 `compaction` 块添加 `text: null` 和 `citations: null`。API 随后会以 400 错误（`Extra inputs are not permitted`）拒绝该请求。请改用 `to_dict()` 或 `model_dump(exclude_none=True)`。[从摘要继续](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand#continue-from-the-summary)针对按需压缩给出了相同的建议。

当 API 收到 `compaction` 块时，其之前的所有内容块都会被忽略。您可以：

* 将原始消息保留在列表中，让 API 负责移除已压缩的内容
* 手动丢弃已压缩的消息，只包含从压缩块开始的内容

如果未生成摘要，该块的 `content` 为 `null`，API 会忽略该块。不会压缩任何内容，因此请保留原始消息。

`compaction` 块之前的思考块不会被延续，因此摘要是模型对早期工作所掌握的全部内容。如果您编写自己的 `instructions`，请告诉模型摘要必须保留哪些内容；请参阅[告诉模型在压缩摘要中保留什么](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#tell-the-model-what-to-preserve-in-compaction-summaries)。

### 流式传输

压缩块的 “streaming”（流式传输）方式与文本块不同。您会收到一个 `content_block_start` 事件，随后是一个包含完整摘要内容的 `content_block_delta`（没有中间流式传输），然后是一个 `content_block_stop` 事件。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: compact-2026-01-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "stream": true,
      "messages": [
        {
          "role": "user",
          "content": "Hello, Claude"
        }
      ],
      "context_management": {
        "edits": [
          {
            "type": "compact_20260112"
          }
        ]
      }
    }'
  ```

  ```bash CLI
  ant beta:messages create \
    --stream \
    --beta compact-2026-01-12 \
    --format jsonl <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: Hello, Claude
  context_management:
    edits:
      - type: compact_20260112
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]

  with client.beta.messages.stream(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=messages,
      context_management={"edits": [{"type": "compact_20260112"}]},
  ) as stream:
      for event in stream:
          match event.type:
              case "content_block_start":
                  block = event.content_block
                  match block.type:
                      case "compaction":
                          print("Compaction started...")
                      case "text":
                          print("Text response started...")

              case "content_block_delta":
                  delta = event.delta
                  match delta.type:
                      case "compaction_delta":
                          print(f"Compaction complete: {len(delta.content or '')} chars")
                      case "text_delta":
                          print(delta.text, end="", flush=True)

      # 获取最终累积的消息
      message = stream.get_final_message()
      messages.append({"role": "assistant", "content": message.content})
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Hello, Claude" }
  ];

  const stream = await client.beta.messages.stream({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  });

  for await (const event of stream) {
    switch (event.type) {
      case "content_block_start":
        switch (event.content_block.type) {
          case "compaction":
            console.log("Compaction started...");
            break;
          case "text":
            console.log("Text response started...");
            break;
        }
        break;
      case "content_block_delta":
        switch (event.delta.type) {
          case "compaction_delta":
            console.log(`Compaction complete: ${event.delta.content?.length ?? 0} chars`);
            break;
          case "text_delta":
            process.stdout.write(event.delta.text);
            break;
        }
        break;
    }
  }

  // 获取最终累积的消息
  const message = await stream.finalMessage();
  messages.push({
    role: "assistant",
    content: message.content
  });
  ```

  ```csharp C#
  var client = new AnthropicClient();
  List<BetaMessageParam> messages = [new BetaUserMessageParam { Content = "Hello" }];

  var parameters = new MessageCreateParams
  {
      Betas = ["compact-2026-01-12"],
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit()]
      }
  };

  await foreach (var streamEvent in client.Beta.Messages.CreateStreaming(parameters))
  {
      if (streamEvent.TryPickContentBlockStart(out var startEvent))
      {
          if (startEvent.ContentBlock.TryPickBetaCompaction(out _))
          {
              Console.WriteLine("Compaction started...");
          }
          else if (startEvent.ContentBlock.TryPickBetaText(out _))
          {
              Console.WriteLine("Text response started...");
          }
      }
      else if (streamEvent.TryPickContentBlockDelta(out var deltaEvent))
      {
          if (deltaEvent.Delta.TryPickCompaction(out var compactionDelta))
          {
              Console.WriteLine($"Compaction complete: {compactionDelta.Content?.Length ?? 0} chars");
          }
          else if (deltaEvent.Delta.TryPickText(out var textDelta))
          {
              Console.Write(textDelta.Text);
          }
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()
  messages := []anthropic.BetaMessageParam{anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude"))}

  stream := client.Beta.Messages.NewStreaming(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages:  messages,
  	ContextManagement: anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
  		},
  	},
  	Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })

  for stream.Next() {
  	event := stream.Current()
  	switch eventVariant := event.AsAny().(type) {
  	case anthropic.BetaRawContentBlockStartEvent:
  		switch eventVariant.ContentBlock.AsAny().(type) {
  		case anthropic.BetaCompactionBlock:
  			fmt.Println("Compaction started...")
  		case anthropic.BetaTextBlock:
  			fmt.Println("Text response started...")
  		}
  	case anthropic.BetaRawContentBlockDeltaEvent:
  		switch deltaVariant := eventVariant.Delta.AsAny().(type) {
  		case anthropic.BetaCompactionContentBlockDelta:
  			fmt.Printf("Compaction complete: %d chars\n", len(deltaVariant.Content))
  		case anthropic.BetaTextDelta:
  			fmt.Print(deltaVariant.Text)
  		}
  	}
  }
  if err := stream.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageCreateParams params = MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addBeta("compact-2026-01-12")
              .addUserMessage("Hello, Claude")
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder().build())
                  .build())
              .build();

          try (var streamResponse = client.beta().messages().createStreaming(params)) {
              streamResponse.stream().forEach(event -> {
                  event.contentBlockStart().ifPresent(startEvent -> {
                      startEvent.contentBlock().compaction().ifPresent(compactionBlock ->
                          System.out.println("Compaction started...")
                      );
                      startEvent.contentBlock().text().ifPresent(textBlock ->
                          System.out.println("Text response started...")
                      );
                  });

                  event.contentBlockDelta().ifPresent(deltaEvent -> {
                      deltaEvent.delta().compaction().ifPresent(compactionDelta ->
                          System.out.println("Compaction complete: " + compactionDelta.content().map(String::length).orElse(0) + " chars")
                      );
                      deltaEvent.delta().text().ifPresent(textDelta ->
                          System.out.print(textDelta.text())
                      );
                  });
              });
          }
  ```

  ```php PHP
  use Anthropic\Beta\Messages\BetaCompactionBlock;
  use Anthropic\Beta\Messages\BetaCompactionContentBlockDelta;
  use Anthropic\Beta\Messages\BetaRawContentBlockDeltaEvent;
  use Anthropic\Beta\Messages\BetaRawContentBlockStartEvent;
  use Anthropic\Beta\Messages\BetaTextBlock;
  use Anthropic\Beta\Messages\BetaTextDelta;
  use Anthropic\Messages\Model;

  $client = new Client();
  $messages = [['role' => 'user', 'content' => 'Hello, Claude']];

  $stream = $client->beta->messages->createStream(
      maxTokens: 4096,
      messages: $messages,
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [
              ['type' => 'compact_20260112']
          ]
      ]
  );

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof BetaRawContentBlockStartEvent:
              switch (true) {
                  case $event->contentBlock instanceof BetaCompactionBlock:
                      echo "Compaction started...\n";
                      break;
                  case $event->contentBlock instanceof BetaTextBlock:
                      echo "Text response started...\n";
                      break;
              }
              break;
          case $event instanceof BetaRawContentBlockDeltaEvent:
              switch (true) {
                  case $event->delta instanceof BetaCompactionContentBlockDelta:
                      echo 'Compaction complete: ' . strlen($event->delta->content ?? '') . " chars\n";
                      break;
                  case $event->delta instanceof BetaTextDelta:
                      echo $event->delta->text;
                      break;
              }
              break;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = [{ role: "user", content: "Hello, Claude" }]

  stream = client.beta.messages.stream(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  )

  stream.each do |event|
    case event
    when Anthropic::Models::BetaRawContentBlockStartEvent
      case event.content_block
      when Anthropic::Models::BetaCompactionBlock
        puts "Compaction started..."
      when Anthropic::Models::BetaTextBlock
        puts "Text response started..."
      end
    when Anthropic::Models::BetaRawContentBlockDeltaEvent
      delta = event.delta
      case delta
      when Anthropic::Models::BetaCompactionContentBlockDelta
        puts "Compaction complete: #{(delta.content || "").length} chars"
      when Anthropic::Models::BetaTextDelta
        print delta.text
      end
    end
  end
  ```
</CodeGroup>

### 提示缓存

压缩与 [“prompt caching”（提示缓存）](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)配合良好。您可以在压缩块上添加 `cache_control` 断点来缓存摘要内容。

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "compaction",
      "content": "[summary text]",
      "cache_control": { "type": "ephemeral" }
    },
    {
      "type": "text",
      "text": "Based on our conversation..."
    }
  ]
}
```

#### 使用系统提示最大化缓存命中

发生压缩时，摘要会成为需要写入缓存的新内容。如果没有额外的缓存断点，这也会使任何已缓存的 “system prompt”（系统提示）失效，需要将其与压缩摘要一起重新缓存。

为了最大化缓存命中率，请在系统提示的末尾添加一个 `cache_control` 断点。这样可以使系统提示与对话分开缓存，因此在发生压缩时：

* 系统提示缓存保持有效，并从缓存中读取
* 只有压缩摘要需要作为新的缓存条目写入

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: compact-2026-01-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "system": [
        {
          "type": "text",
          "text": "You are a helpful coding assistant...",
          "cache_control": {
            "type": "ephemeral"
          }
        }
      ],
      "messages": [
        {
          "role": "user",
          "content": "Hello, Claude"
        }
      ],
      "context_management": {
        "edits": [
          {
            "type": "compact_20260112"
          }
        ]
      }
    }'
  ```

  ```bash CLI
  ant beta:messages create --beta compact-2026-01-12 <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  system:
    - type: text
      text: You are a helpful coding assistant...
      cache_control:
        type: ephemeral
  messages:
    - role: user
      content: Hello, Claude
  context_management:
    edits:
      - type: compact_20260112
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]
  response = client.beta.messages.create(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      max_tokens=4096,
      system=[
          {
              "type": "text",
              "text": "You are a helpful coding assistant...",
              "cache_control": {
                  "type": "ephemeral"
              },  # Cache the system prompt separately
          }
      ],
      messages=messages,
      context_management={"edits": [{"type": "compact_20260112"}]},
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Hello, Claude" }
  ];

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    max_tokens: 4096,
    system: [
      {
        type: "text",
        text: "You are a helpful coding assistant...",
        cache_control: { type: "ephemeral" } // Cache the system prompt separately
      }
    ],
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  });
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var parameters = new MessageCreateParams
  {
      Betas = ["compact-2026-01-12"],
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      System = new List<BetaTextBlockParam>
      {
          new()
          {
              Text = "You are a helpful coding assistant...",
              CacheControl = new BetaCacheControlEphemeral()
          }
      },
      Messages = [new BetaUserMessageParam { Content = "Hello, Claude" }],
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit()]
      }
  };

  var response = await client.Beta.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	System: []anthropic.BetaTextBlockParam{
  		{
  			Text:         "You are a helpful coding assistant...",
  			CacheControl: anthropic.NewBetaCacheControlEphemeralParam(),
  		},
  	},
  	Messages: []anthropic.BetaMessageParam{anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude"))},
  	ContextManagement: anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
  		},
  	},
  	Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  import com.anthropic.models.beta.messages.BetaCacheControlEphemeral;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageCreateParams params = MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addBeta("compact-2026-01-12")
              .systemOfBetaTextBlockParams(List.of(
                  BetaTextBlockParam.builder()
                      .text("You are a helpful coding assistant...")
                      .cacheControl(BetaCacheControlEphemeral.builder().build())
                      .build()
              ))
              .addUserMessage("Hello, Claude")
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder().build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);
          System.out.println(response);
  ```

  ```php PHP
  $client = new Client();

  $response = $client->beta->messages->create(
      maxTokens: 4096,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['compact-2026-01-12'],
      system: [
          [
              'type' => 'text',
              'text' => 'You are a helpful coding assistant...',
              'cache_control' => [
                  'type' => 'ephemeral'
              ]
          ]
      ],
      contextManagement: [
          'edits' => [
              ['type' => 'compact_20260112']
          ]
      ]
  );

  echo json_encode($response, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.beta.messages.create(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    system: [
      {
        type: "text",
        text: "You are a helpful coding assistant...",
        cache_control: {
          type: "ephemeral"
        }
      }
    ],
    messages: [{ role: "user", content: "Hello, Claude" }],
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  )
  puts response
  ```
</CodeGroup>

这样可以在整个对话的多次压缩事件中保持长系统提示的缓存。

## 了解用量

压缩需要一个额外的采样步骤，这会计入 “rate limit”（速率限制）和计费。API 会在响应中返回详细的用量信息：

```json Output
{
  "usage": {
    "input_tokens": 23000,
    "output_tokens": 1000,
    "iterations": [
      {
        "type": "compaction",
        "input_tokens": 180000,
        "output_tokens": 3500
      },
      {
        "type": "message",
        "input_tokens": 23000,
        "output_tokens": 1000
      }
    ]
  }
}
```

`iterations` 数组显示每次采样迭代的用量。发生压缩时，您会看到一个 `compaction` 迭代，随后是主 `message` 迭代。在此示例中，顶层的 `input_tokens` 和 `output_tokens` 与 `message` 迭代完全一致，因为只有一个非压缩迭代。最后一次迭代的令牌计数反映了压缩后的有效上下文大小。

<Note>
  顶层的 `input_tokens` 和 `output_tokens` 不包括压缩迭代的用量。它们反映的是所有非压缩迭代的总和。要计算一个请求消耗和计费的总令牌数，请对 `usage.iterations` 数组中的所有条目求和。

  如果您之前依赖 `usage.input_tokens` 和 `usage.output_tokens` 进行成本跟踪或审计，则在启用压缩时，需要更新跟踪逻辑以汇总 `usage.iterations` 中的数据。启用压缩 beta 后，每个响应都会包含 `usage.iterations`，即使没有发生压缩也是如此。只有在请求期间触发了新的压缩时，才会出现 `compaction` 条目。重新应用之前的 `compaction` 块不会产生额外的压缩成本，在这种情况下顶层用量字段仍然准确。
</Note>

## 与其他功能结合使用

### 服务器工具

使用服务器工具（例如网页搜索）时，会在每次采样迭代开始时检查压缩触发条件。根据您的触发阈值和生成的输出量，单个请求中可能会发生多次压缩。

### 令牌计数

令牌计数端点（`/v1/messages/count_tokens`）会应用提示中已有的 `compaction` 块，但不会触发新的压缩。使用它来检查之前压缩后的有效令牌数：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages/count_tokens \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: compact-2026-01-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "messages": [
        {
          "role": "user",
          "content": "Hello, Claude"
        }
      ],
      "context_management": {
        "edits": [
          {
            "type": "compact_20260112"
          }
        ]
      }
    }'
  ```

  ```bash CLI
  ant beta:messages count-tokens --beta compact-2026-01-12 <<'YAML'
  model: claude-opus-5-5
  messages:
    - role: user
      content: Hello, Claude
  context_management:
    edits:
      - type: compact_20260112
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()
  messages = [{"role": "user", "content": "Hello, Claude"}]
  count_response = client.beta.messages.count_tokens(
      betas=["compact-2026-01-12"],
      model="claude-opus-5-5",
      messages=messages,
      context_management={"edits": [{"type": "compact_20260112"}]},
  )

  print(f"Current tokens: {count_response.input_tokens}")
  print(f"Original tokens: {count_response.context_management.original_input_tokens}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [
    { role: "user", content: "Summarize the key points of our conversation so far." }
  ];

  const countResponse = await client.beta.messages.countTokens({
    betas: ["compact-2026-01-12"],
    model: "claude-opus-5-5",
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  });

  console.log(`Current tokens: ${countResponse.input_tokens}`);
  console.log(`Original tokens: ${countResponse.context_management!.original_input_tokens}`);
  ```

  ```csharp C#
  AnthropicClient client = new();
  List<BetaMessageParam> messages = [new BetaUserMessageParam { Content = "Hello" }];

  var countParams = new MessageCountTokensParams
  {
      Model = Model.ClaudeOpus5_5,
      Messages = messages,
      ContextManagement = new BetaContextManagementConfig
      {
          Edits = [new BetaCompact20260112Edit()]
      },
      Betas = ["compact-2026-01-12"]
  };

  var countResponse = await client.Beta.Messages.CountTokens(countParams);
  Console.WriteLine($"Current tokens: {countResponse.InputTokens}");
  Console.WriteLine($"Original tokens: {countResponse.ContextManagement?.OriginalInputTokens}");
  ```

  ```go Go
  client := anthropic.NewClient()
  messages := []anthropic.BetaMessageParam{anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude"))}

  countResponse, err := client.Beta.Messages.CountTokens(context.TODO(), anthropic.BetaMessageCountTokensParams{
  	Model:    anthropic.ModelClaudeOpus5_5,
  	Messages: messages,
  	ContextManagement: anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
  		},
  	},
  	Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("Current tokens: %d\n", countResponse.InputTokens)
  fmt.Printf("Original tokens: %d\n", countResponse.ContextManagement.OriginalInputTokens)
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaMessageTokensCount;
  import com.anthropic.models.beta.messages.MessageCountTokensParams;
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  // ...
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          MessageCountTokensParams params = MessageCountTokensParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .addUserMessage("Hello, Claude")
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder().build())
                  .build())
              .addBeta("compact-2026-01-12")
              .build();

          BetaMessageTokensCount countResponse = client.beta().messages().countTokens(params);
          System.out.println("Current tokens: " + countResponse.inputTokens());
          System.out.println("Original tokens: " + countResponse.contextManagement().get().originalInputTokens());
  ```

  ```php PHP
  $client = new Client();
  $messages = [['role' => 'user', 'content' => 'Hello, Claude']];

  $countResponse = $client->beta->messages->countTokens(
      messages: $messages,
      model: 'claude-opus-5-5',
      betas: ['compact-2026-01-12'],
      contextManagement: [
          'edits' => [
              ['type' => 'compact_20260112']
          ]
      ]
  );

  echo 'Current tokens: ' . $countResponse->inputTokens . "\n";
  echo 'Original tokens: ' . $countResponse->contextManagement->originalInputTokens . "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = [{ role: "user", content: "Hello, Claude" }]

  count_response = client.beta.messages.count_tokens(
    betas: ["compact-2026-01-12"],
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    messages: messages,
    context_management: {
      edits: [{ type: "compact_20260112" }]
    }
  )

  puts "Current tokens: #{count_response.input_tokens}"
  puts "Original tokens: #{count_response.context_management.original_input_tokens}"
  ```
</CodeGroup>

## 示例

以下是一个使用压缩的长时间运行对话的完整示例：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  client = anthropic.Anthropic()

  messages: list[dict] = []


  def chat(user_message: str) -> str:
      messages.append({"role": "user", "content": user_message})

      response = client.beta.messages.create(
          betas=["compact-2026-01-12"],
          model="claude-opus-5-5",
          max_tokens=4096,
          messages=messages,
          context_management={
              "edits": [
                  {
                      "type": "compact_20260112",
                      "trigger": {"type": "input_tokens", "value": 100000},
                  }
              ]
          },
      )

      # 追加响应（压缩块会自动包含在内）
      messages.append({"role": "assistant", "content": response.content})

      # 返回文本内容
      return next((block.text for block in response.content if block.type == "text"), "")


  # 运行一段长对话
  print(chat("Briefly, help me build a Python web scraper"))
  print(chat("Briefly, add support for JavaScript-rendered pages"))
  print(chat("Briefly, add rate limiting and error handling"))
  # 根据对话需要持续调用 chat()
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const messages: Anthropic.Beta.Messages.BetaMessageParam[] = [];

  async function chat(userMessage: string): Promise<string> {
    messages.push({ role: "user", content: userMessage });

    const response = await client.beta.messages.create({
      betas: ["compact-2026-01-12"],
      model: "claude-opus-5-5",
      max_tokens: 4096,
      messages,
      context_management: {
        edits: [
          {
            type: "compact_20260112",
            trigger: { type: "input_tokens", value: 100000 }
          }
        ]
      }
    });

    // 追加响应（压缩块会自动包含在内）
    messages.push({ role: "assistant", content: response.content });

    // 返回文本内容
    const textBlock = response.content.find((block) => block.type === "text");
    return textBlock?.text ?? "";
  }

  // 运行一段长对话
  console.log(await chat("Briefly, help me build a Python web scraper"));
  console.log(await chat("Briefly, add support for JavaScript-rendered pages"));
  console.log(await chat("Briefly, add rate limiting and error handling"));
  // 根据对话需要持续调用 chat()
  ```

  ```csharp C#
  AnthropicClient client = new();
  List<BetaMessageParam> messages = new();

  Console.WriteLine(await Chat(client, messages, "Briefly, help me build a Python web scraper"));
  Console.WriteLine(await Chat(client, messages, "Briefly, add support for JavaScript-rendered pages"));
  Console.WriteLine(await Chat(client, messages, "Briefly, add rate limiting and error handling"));

  static async Task<string> Chat(AnthropicClient client, List<BetaMessageParam> messages, string userMessage)
  {
      messages.Add(new BetaUserMessageParam { Content = userMessage });

      var parameters = new MessageCreateParams
      {
          Betas = ["compact-2026-01-12"],
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 4096,
          Messages = messages,
          ContextManagement = new BetaContextManagementConfig
          {
              Edits = [new BetaCompact20260112Edit
              {
                  Trigger = new BetaInputTokensTrigger(100000)
              }]
          }
      };

      var response = await client.Beta.Messages.Create(parameters);

      messages.Add(new BetaAssistantMessageParam
      {
          Content = response.Content.Select(block => new BetaContentBlockParam(block.Json)).ToList()
      });

      return response.Content
          .Select(block => block.Value)
          .OfType<BetaTextBlock>()
          .Select(textBlock => textBlock.Text)
          .FirstOrDefault() ?? "";
  }
  ```

  ```go Go
  package main

  import (
  	"context"
  	"fmt"
  	"log"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  var (
  	client   = anthropic.NewClient()
  	messages []anthropic.BetaMessageParam
  )

  func chat(userMessage string) string {
  	messages = append(messages, anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock(userMessage)))

  	response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 4096,
  		Messages:  messages,
  		ContextManagement: anthropic.BetaContextManagementConfigParam{
  			Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  				{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{
  					Trigger: anthropic.BetaInputTokensTriggerParam{Value: 100000},
  				}},
  			},
  		},
  		Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	messages = append(messages, response.ToParam())

  	for _, block := range response.Content {
  		if variant, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  			return variant.Text
  		}
  	}
  	return ""
  }

  func main() {
  	fmt.Println(chat("Briefly, help me build a Python web scraper"))
  	fmt.Println(chat("Briefly, add support for JavaScript-rendered pages"))
  	fmt.Println(chat("Briefly, add rate limiting and error handling"))
  }
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  import com.anthropic.models.beta.messages.BetaInputTokensTrigger;
  // ...
      private static final AnthropicClient client = AnthropicOkHttpClient.fromEnv();
      private static final List<BetaMessageParam> messages = new ArrayList<>();

      public static void main(String[] args) {
          System.out.println(chat("Briefly, help me build a Python web scraper"));
          System.out.println(chat("Briefly, add support for JavaScript-rendered pages"));
          System.out.println(chat("Briefly, add rate limiting and error handling"));
      }

      private static String chat(String userMessage) {
          messages.add(BetaMessageParam.ofUser(userMessage));

          MessageCreateParams params = MessageCreateParams.builder()
              .addBeta("compact-2026-01-12")
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .messages(messages)
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder()
                      .trigger(BetaInputTokensTrigger.builder()
                          .value(100000L)
                          .build())
                      .build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);

          // 追加响应（compaction（压缩）块会自动包含在内）
          messages.add(response.toParam());

          return response.content().stream()
              .filter(block -> block.text().isPresent())
              .map(block -> block.text().get().text())
              .findFirst()
              .orElse("");
      }
  ```

  ```php PHP
  $client = new Client();
  $messages = [];

  function chat($client, &$messages, $userMessage)
  {
      $messages[] = ['role' => 'user', 'content' => $userMessage];

      $response = $client->beta->messages->create(
          maxTokens: 4096,
          messages: $messages,
          model: 'claude-opus-5-5',
          betas: ['compact-2026-01-12'],
          contextManagement: [
              'edits' => [
                  [
                      'type' => 'compact_20260112',
                      'trigger' => ['type' => 'input_tokens', 'value' => 100000]
                  ]
              ]
          ]
      );

      $messages[] = ['role' => 'assistant', 'content' => $response->content];

      foreach ($response->content as $block) {
          if ($block->type === 'text') {
              return $block->text;
          }
      }
      return '';
  }

  echo chat($client, $messages, 'Briefly, help me build a Python web scraper') . "\n";
  echo chat($client, $messages, 'Briefly, add support for JavaScript-rendered pages') . "\n";
  echo chat($client, $messages, 'Briefly, add rate limiting and error handling') . "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = []

  def chat(client, messages, user_message)
    messages << { role: "user", content: user_message }

    response = client.beta.messages.create(
      betas: ["compact-2026-01-12"],
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 4096,
      messages: messages,
      context_management: {
        edits: [
          {
            type: "compact_20260112",
            trigger: { type: "input_tokens", value: 100000 }
          }
        ]
      }
    )

    messages << { role: "assistant", content: response.content }

    response.content.find { |block| block.type == :text }&.text || ""
  end

  puts chat(client, messages, "Briefly, help me build a Python web scraper")
  puts chat(client, messages, "Briefly, add support for JavaScript-rendered pages")
  puts chat(client, messages, "Briefly, add rate limiting and error handling")
  ```
</CodeGroup>

当您在压缩块之后重新插入较早的助手轮次时，在某些模型上，该轮次中的思考块无法通过[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-in-conversation)。这些模型是 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5。这些块之所以无法通过，是因为它们是在完整历史记录存在时生成的。

要避免此问题，请执行以下操作之一：

* **移除这些块：** 从您重新插入的轮次中移除 `thinking` 和 `redacted_thinking` 块。保留的文本块和工具块可以保持原样。
* **让 API 丢弃这些块：** 发送 `thinking.block_binding.prefix_mismatch_behavior: "drop_block"`。这需要附带 `thinking-binding-controls-2026-08-01` beta 标头（请参阅 [Beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers)）。
* **不要重新插入较早的轮次：** 让 API 总结所有内容。

在 Claude Sonnet 5.5 和 Claude Haiku 5.5 上，`block_binding` 仅适用于 `thinking: {"type": "adaptive"}`。在这两个模型上使用任何其他思考设置时，请移除这些块。

以下示例使用 `pause_after_compaction` 来保留上一轮交流和当前用户消息（共三条消息），而不是对它们进行总结。它保持这些消息的文本块和工具块不变，从保留的助手轮次中移除 `thinking` 和 `redacted_thinking` 块，如果该轮次没有剩余内容则将其丢弃：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  from typing import Any

  client = anthropic.Anthropic()

  messages: list[dict[str, Any]] = []


  def chat(user_message: str) -> str:
      messages.append({"role": "user", "content": user_message})

      response = client.beta.messages.create(
          betas=["compact-2026-01-12"],
          model="claude-opus-5-5",
          max_tokens=4096,
          messages=messages,
          context_management={
              "edits": [
                  {
                      "type": "compact_20260112",
                      "trigger": {"type": "input_tokens", "value": 100000},
                      "pause_after_compaction": True,
                  }
              ]
          },
      )

      # 查找压缩块，而不是停止原因：
      # 如果摘要被拦截，stop_reason 为 "refusal"，而不是 "compaction"
      compaction_block = next(
          (block for block in response.content if block.type == "compaction"), None
      )

      # 仅在生成了摘要时才替换历史记录
      if compaction_block is not None and compaction_block.content is not None:
          # 保留之前的对话轮次 + 当前用户消息（3 条消息）
          # 方法是将它们放在压缩块之后，且不包含思考
          # 块；没有剩余内容的 assistant 轮次将被丢弃
          preserved_messages = [
              {
                  **message,
                  "content": [
                      block
                      for block in message["content"]
                      if block.type not in ("thinking", "redacted_thinking")
                  ],
              }
              if message["role"] == "assistant"
              else message
              for message in messages[-3:]
          ]
          preserved_messages = [m for m in preserved_messages if m["content"]]

          # 构建新的消息列表：压缩块 + 保留的消息
          new_assistant_content = [compaction_block]
          messages_after_compaction = [
              {"role": "assistant", "content": new_assistant_content}
          ] + preserved_messages

          # 更新消息列表以反映压缩结果
          messages.clear()
          messages.extend(messages_after_compaction)

      if compaction_block is not None:
          # 继续该请求，如果没有摘要则使用完整的历史记录
          response = client.beta.messages.create(
              betas=["compact-2026-01-12"],
              model="claude-opus-5-5",
              max_tokens=4096,
              messages=messages,
              context_management={"edits": [{"type": "compact_20260112"}]},
          )

      # 追加最终响应
      messages.append({"role": "assistant", "content": response.content})

      # 返回文本内容
      return next((block.text for block in response.content if block.type == "text"), "")


  # 运行一段长对话
  print(chat("Briefly, help me build a Python web scraper"))
  print(chat("Briefly, add support for JavaScript-rendered pages"))
  print(chat("Briefly, add rate limiting and error handling"))
  # 根据对话需要持续调用 chat()
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  let messages: Anthropic.Beta.Messages.BetaMessageParam[] = [];

  async function chat(userMessage: string): Promise<string> {
    messages.push({ role: "user", content: userMessage });

    let response = await client.beta.messages.create({
      betas: ["compact-2026-01-12"],
      model: "claude-opus-5-5",
      max_tokens: 4096,
      messages,
      context_management: {
        edits: [
          {
            type: "compact_20260112",
            trigger: { type: "input_tokens", value: 100000 },
            pause_after_compaction: true
          }
        ]
      }
    });

    // 查找压缩块，而不是停止原因：
    // 如果摘要被拦截，stop_reason 为 "refusal"，而不是 "compaction"
    const compactionBlock = response.content.find((block) => block.type === "compaction");

    // 仅在生成了摘要时才替换历史记录
    if (compactionBlock && compactionBlock.content !== null) {
      // 保留之前的对话交换和当前用户消息（3 条消息）
      // 方法是将它们放在压缩块之后，并去除思考
      // 块；没有剩余内容的助手轮次将被丢弃
      const preservedMessages = messages
        .slice(-3)
        .map((message) =>
          message.role === "assistant" && Array.isArray(message.content)
            ? {
                ...message,
                content: message.content.filter(
                  (block) => block.type !== "thinking" && block.type !== "redacted_thinking"
                )
              }
            : message
        )
        .filter((message) => message.role !== "assistant" || message.content.length > 0);

      // 构建新的消息列表：压缩块 + 保留的消息
      const messagesAfterCompaction: Anthropic.Beta.Messages.BetaMessageParam[] = [
        { role: "assistant", content: [compactionBlock] },
        ...preservedMessages
      ];

      // 更新消息列表以反映压缩结果
      messages = messagesAfterCompaction;
    }

    if (compactionBlock) {
      // 继续该请求，如果没有摘要则使用完整的历史记录
      response = await client.beta.messages.create({
        betas: ["compact-2026-01-12"],
        model: "claude-opus-5-5",
        max_tokens: 4096,
        messages,
        context_management: {
          edits: [{ type: "compact_20260112" }]
        }
      });
    }

    // 追加最终响应
    messages.push({ role: "assistant", content: response.content });

    // 返回文本内容
    const textBlock = response.content.find((block) => block.type === "text");
    return textBlock?.text ?? "";
  }

  // 运行一段长对话
  console.log(await chat("Briefly, help me build a Python web scraper"));
  console.log(await chat("Briefly, add support for JavaScript-rendered pages"));
  console.log(await chat("Briefly, add rate limiting and error handling"));
  // 根据对话需要持续调用 chat()
  ```

  ```csharp C#
  AnthropicClient client = new();
  List<BetaMessageParam> messages = new();

  Console.WriteLine(await Chat("Briefly, help me build a Python web scraper"));
  Console.WriteLine(await Chat("Briefly, add support for JavaScript-rendered pages"));
  Console.WriteLine(await Chat("Briefly, add rate limiting and error handling"));

  async Task<string> Chat(string userMessage)
  {
      messages.Add(new BetaUserMessageParam { Content = userMessage });

      var response = await client.Beta.Messages.Create(new()
      {
          Betas = ["compact-2026-01-12"],
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 4096,
          Messages = messages,
          ContextManagement = new BetaContextManagementConfig
          {
              Edits = [new BetaCompact20260112Edit
              {
                  Trigger = new BetaInputTokensTrigger(100000),
                  PauseAfterCompaction = true
              }]
          }
      });

      // 查找压缩块，而不是停止原因：
      // 如果摘要被拦截，StopReason 为 Refusal，而不是 Compaction
      var compactionBlock = response.Content.FirstOrDefault(block => block.Value is BetaCompactionBlock);

      // 仅在生成了摘要时才替换历史记录
      if (compactionBlock?.Value is BetaCompactionBlock { Content: not null })
      {
          // 保留先前的对话轮次及当前用户消息（共 3 条消息），
          // 不含思考块；内容为空的 assistant 轮次将被丢弃
          var preserved = new List<BetaMessageParam>();
          foreach (var message in messages.Skip(Math.Max(0, messages.Count - 3)))
          {
              if (!message.TryPickAssistant(out var assistant)
                  || !assistant.Content.TryPickBetaContentBlockParams(out var blocks))
              {
                  preserved.Add(message);
                  continue;
              }

              var kept = blocks
                  .Where(block => block.Type.Raw() is not ("thinking" or "redacted_thinking"))
                  .ToList();
              if (kept.Count > 0)
              {
                  preserved.Add(assistant with { Content = kept });
              }
          }

          var messagesAfterCompaction = new List<BetaMessageParam>
          {
              new BetaAssistantMessageParam
              {
                  Content = new List<BetaContentBlockParam> { new BetaContentBlockParam(compactionBlock.Json) }
              }
          };
          messagesAfterCompaction.AddRange(preserved);

          messages = messagesAfterCompaction;
      }

      if (compactionBlock is not null)
      {
          // 继续该请求，如果没有摘要则使用完整的历史记录
          response = await client.Beta.Messages.Create(new()
          {
              Betas = ["compact-2026-01-12"],
              Model = Model.ClaudeOpus5_5,
              MaxTokens = 4096,
              Messages = messages,
              ContextManagement = new BetaContextManagementConfig
              {
                  Edits = [new BetaCompact20260112Edit()]
              }
          });
      }

      messages.Add(new BetaAssistantMessageParam
      {
          Content = response.Content.Select(block => new BetaContentBlockParam(block.Json)).ToList()
      });

      return response.Content
          .Select(block => block.Value)
          .OfType<BetaTextBlock>()
          .Select(textBlock => textBlock.Text)
          .FirstOrDefault() ?? "";
  }
  ```

  ```go Go
  package main

  import (
  	"context"
  	"fmt"
  	"log"
  	"slices"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  var (
  	client   = anthropic.NewClient()
  	messages []anthropic.BetaMessageParam
  )

  func chat(userMessage string) string {
  	messages = append(messages, anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock(userMessage)))

  	compactEdit := anthropic.BetaContextManagementConfigParam{
  		Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  			{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{
  				Trigger:              anthropic.BetaInputTokensTriggerParam{Value: 100000},
  				PauseAfterCompaction: anthropic.Bool(true),
  			}},
  		},
  	}

  	response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  		Model:             anthropic.ModelClaudeOpus5_5,
  		MaxTokens:         4096,
  		Messages:          messages,
  		ContextManagement: compactEdit,
  		Betas:             []anthropic.AnthropicBeta{"compact-2026-01-12"},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	// 查找压缩块，而不是停止原因：
  	// 如果摘要被拦截，StopReason 为 "refusal"，而不是 "compaction"
  	compactionIndex := slices.IndexFunc(response.Content, func(block anthropic.BetaContentBlockUnion) bool {
  		return block.Type == "compaction"
  	})

  	// 仅在生成了摘要（content 不为 null）时才替换历史记录
  	if compactionIndex >= 0 && response.Content[compactionIndex].JSON.Content.Valid() {
  		compactionParam := response.Content[compactionIndex].ToParam()

  		// 保留之前的对话轮次及当前用户消息（共 3 条消息），
  		// 不含思考块；内容为空的 assistant 轮次将被丢弃
  		var preserved []anthropic.BetaMessageParam
  		for _, message := range messages[max(0, len(messages)-3):] {
  			if message.OfAssistant == nil {
  				preserved = append(preserved, message)
  				continue
  			}
  			var content []anthropic.BetaContentBlockParamUnion
  			for _, block := range message.OfAssistant.Content {
  				if block.OfThinking == nil && block.OfRedactedThinking == nil {
  					content = append(content, block)
  				}
  			}
  			if len(content) == 0 {
  				continue
  			}
  			preserved = append(preserved, anthropic.BetaMessageParam{OfAssistant: &anthropic.BetaAssistantMessageParam{Content: content}})
  		}

  		messagesAfterCompaction := []anthropic.BetaMessageParam{
  			{OfAssistant: &anthropic.BetaAssistantMessageParam{Content: []anthropic.BetaContentBlockParamUnion{compactionParam}}},
  		}
  		messagesAfterCompaction = append(messagesAfterCompaction, preserved...)

  		messages = messagesAfterCompaction
  	}

  	if compactionIndex >= 0 {
  		// 继续该请求，如果没有摘要则使用完整的历史记录
  		response, err = client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 4096,
  			Messages:  messages,
  			ContextManagement: anthropic.BetaContextManagementConfigParam{
  				Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
  					{OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
  				},
  			},
  			Betas: []anthropic.AnthropicBeta{"compact-2026-01-12"},
  		})
  		if err != nil {
  			log.Fatal(err)
  		}
  	}

  	messages = append(messages, response.ToParam())

  	for _, block := range response.Content {
  		if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  			return textBlock.Text
  		}
  	}
  	return ""
  }

  func main() {
  	fmt.Println(chat("Briefly, help me build a Python web scraper"))
  	fmt.Println(chat("Briefly, add support for JavaScript-rendered pages"))
  	fmt.Println(chat("Briefly, add rate limiting and error handling"))
  }
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContextManagementConfig;
  import com.anthropic.models.beta.messages.BetaCompact20260112Edit;
  import com.anthropic.models.beta.messages.BetaInputTokensTrigger;
  // ...
      private static final AnthropicClient client = AnthropicOkHttpClient.fromEnv();
      private static final List<BetaMessageParam> messages = new ArrayList<>();

      public static String chat(String userMessage) {
          messages.add(BetaMessageParam.ofUser(userMessage));

          MessageCreateParams params = MessageCreateParams.builder()
              .addBeta("compact-2026-01-12")
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .messages(messages)
              .contextManagement(BetaContextManagementConfig.builder()
                  .addEdit(BetaCompact20260112Edit.builder()
                      .trigger(BetaInputTokensTrigger.builder()
                          .value(100000L)
                          .build())
                      .pauseAfterCompaction(true)
                      .build())
                  .build())
              .build();

          BetaMessage response = client.beta().messages().create(params);

          // 查找压缩块，而不是停止原因：
          // 如果摘要被拦截，stopReason() 为 REFUSAL，而不是 COMPACTION
          var compactionBlock = response.content().stream()
              .flatMap(block -> block.compaction().stream())
              .findFirst();

          // 仅在生成了摘要时才替换历史记录
          if (compactionBlock.flatMap(block -> block.content()).isPresent()) {
              // 保留先前的交互及当前用户消息（共 3 条消息），
              // 不含思考块；内容为空的助手轮次将被丢弃
              List<BetaMessageParam> preservedMessages = new ArrayList<>();
              for (BetaMessageParam message : messages.subList(Math.max(0, messages.size() - 3), messages.size())) {
                  if (message.isAssistant() && message.asAssistant().content().isContentBlockParams()) {
                      var keptBlocks = message.asAssistant().content().asContentBlockParams().stream()
                          .filter(block -> !block.isThinking() && !block.isRedactedThinking())
                          .toList();
                      if (keptBlocks.isEmpty()) {
                          continue;
                      }
                      message = BetaMessageParam.ofAssistantOfContentBlockParams(keptBlocks);
                  }
                  preservedMessages.add(message);
              }

              // 构建新的消息列表：压缩内容 + 保留的消息
              List<BetaMessageParam> messagesAfterCompaction = new ArrayList<>();
              messagesAfterCompaction.add(response.toParam());
              messagesAfterCompaction.addAll(preservedMessages);

              // 更新消息列表以反映压缩结果
              messages.clear();
              messages.addAll(messagesAfterCompaction);
          }

          if (compactionBlock.isPresent()) {
              // 继续该请求，如果没有摘要则使用完整的历史记录
              MessageCreateParams continueParams = MessageCreateParams.builder()
                  .addBeta("compact-2026-01-12")
                  .model(Model.CLAUDE_OPUS_5_5)
                  .maxTokens(4096L)
                  .messages(messages)
                  .contextManagement(BetaContextManagementConfig.builder()
                      .addEdit(BetaCompact20260112Edit.builder().build())
                      .build())
                  .build();

              response = client.beta().messages().create(continueParams);
          }

          // 追加最终响应
          messages.add(response.toParam());

          return response.content().stream()
              .filter(block -> block.text().isPresent())
              .map(block -> block.text().get().text())
              .findFirst()
              .orElse("");
      }

      public static void main(String[] args) {
          System.out.println(chat("Briefly, help me build a Python web scraper"));
          System.out.println(chat("Briefly, add support for JavaScript-rendered pages"));
          System.out.println(chat("Briefly, add rate limiting and error handling"));
      }
  ```

  ```php PHP
  $client = new Client();
  $messages = [];

  function chat($client, &$messages, $userMessage)
  {
      $messages[] = ['role' => 'user', 'content' => $userMessage];

      $response = $client->beta->messages->create(
          maxTokens: 4096,
          messages: $messages,
          model: 'claude-opus-5-5',
          betas: ['compact-2026-01-12'],
          contextManagement: [
              'edits' => [
                  [
                      'type' => 'compact_20260112',
                      'trigger' => ['type' => 'input_tokens', 'value' => 100000],
                      'pause_after_compaction' => true
                  ]
              ]
          ]
      );

      // 查找压缩块，而不是停止原因：
      // 如果摘要被拦截，stopReason 为 'refusal'，而不是 'compaction'
      $compactionBlock = null;
      foreach ($response->content as $block) {
          if ($block instanceof \Anthropic\Beta\Messages\BetaCompactionBlock) {
              $compactionBlock = $block;
              break;
          }
      }

      // 仅在生成了摘要时才替换历史记录
      if ($compactionBlock?->content !== null) {
          // 保留先前的对话轮次及当前用户消息（共 3 条消息），
          // 不含思考块；内容为空的 assistant 轮次将被丢弃
          $preserved = array_values(array_filter(array_map(
              fn($message) => $message['role'] === 'assistant'
                  ? array_merge($message, ['content' => array_values(array_filter(
                      $message['content'],
                      fn($block) => !in_array($block->type, ['thinking', 'redacted_thinking'], true)
                  ))])
                  : $message,
              array_slice($messages, -3)
          ), fn($message) => $message['content'] !== []));

          $messagesAfterCompaction = array_merge(
              [['role' => 'assistant', 'content' => [$compactionBlock]]],
              $preserved
          );

          $messages = $messagesAfterCompaction;
      }

      if ($compactionBlock !== null) {
          // 继续该请求，如果没有摘要则使用完整的历史记录
          $response = $client->beta->messages->create(
              maxTokens: 4096,
              messages: $messages,
              model: 'claude-opus-5-5',
              betas: ['compact-2026-01-12'],
              contextManagement: [
                  'edits' => [['type' => 'compact_20260112']]
              ]
          );
      }

      $messages[] = ['role' => 'assistant', 'content' => $response->content];

      foreach ($response->content as $block) {
          if ($block->type === 'text') {
              return $block->text;
          }
      }
      return '';
  }

  echo chat($client, $messages, 'Briefly, help me build a Python web scraper') . "\n";
  echo chat($client, $messages, 'Briefly, add support for JavaScript-rendered pages') . "\n";
  echo chat($client, $messages, 'Briefly, add rate limiting and error handling') . "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = []

  def chat(client, messages, user_message)
    messages << { role: "user", content: user_message }

    response = client.beta.messages.create(
      betas: ["compact-2026-01-12"],
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 4096,
      messages: messages,
      context_management: {
        edits: [
          {
            type: "compact_20260112",
            trigger: { type: "input_tokens", value: 100000 },
            pause_after_compaction: true
          }
        ]
      }
    )

    # 查找压缩块，而不是停止原因：
    # 如果摘要被拦截，stop_reason 为 :refusal，而不是 :compaction
    compaction_block = response.content.find { |block| block.type == :compaction }

    # 仅在生成了摘要时才替换历史记录
    if compaction_block&.content
      # 保留之前的对话交换及当前用户消息（共 3 条消息），
      # 不含思考块；内容为空的 assistant 轮次将被丢弃
      preserved = messages.last(3).map do |message|
        next message unless message[:role] == "assistant"

        message.merge(content: message[:content].reject { |block| %i[thinking redacted_thinking].include?(block.type) })
      end.reject { |message| message[:content].empty? }

      messages_after_compaction = [
        { role: "assistant", content: [compaction_block] }
      ] + preserved

      messages.clear
      messages.concat(messages_after_compaction)
    end

    if compaction_block
      # 继续该请求，如果没有摘要则使用完整的历史记录
      response = client.beta.messages.create(
        betas: ["compact-2026-01-12"],
        model: Anthropic::Model::CLAUDE_OPUS_5_5,
        max_tokens: 4096,
        messages: messages,
        context_management: {
          edits: [{ type: "compact_20260112" }]
        }
      )
    end

    messages << { role: "assistant", content: response.content }

    response.content.find { |block| block.type == :text }&.text || ""
  end

  puts chat(client, messages, "Briefly, help me build a Python web scraper")
  puts chat(client, messages, "Briefly, add support for JavaScript-rendered pages")
  puts chat(client, messages, "Briefly, add rate limiting and error handling")
  ```
</CodeGroup>

## 当前限制

* **使用相同模型进行总结：** 请求中指定的模型将用于总结。目前无法选择使用其他（例如更便宜的）模型来生成摘要。

* **定义了工具时压缩可能失败：** 当您的请求包含 `tools` 时，模型偶尔会在内部总结步骤中调用工具，而不是编写摘要。发生这种情况时，响应会包含一个 `content: null` 的 `compaction` 块。为防止这种情况，请将 [`instructions`](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold#custom-summarization-instructions) 设置为明确告诉模型不要调用工具的提示，例如：

  ```text wrap
  Summarize the transcript inside <summary></summary> tags. Include relevant information in the summary for continuing the task in the next context window. Do not call any tools while writing this summary; respond with text only.
  ```

## 后续步骤

<CardGroup cols={3}>
  <Card title="上下文编辑" icon="edit" href="https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing">
    使用上下文编辑在对话增长时自动管理对话上下文。
  </Card>

  <Card title="上下文窗口" icon="arrows-left-right" href="https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows">
    了解上下文窗口大小和管理策略。
  </Card>

  <Card title="会话记忆压缩方案（Claude Cookbook）" icon="book" href="https://platform.claude.com/cookbook/misc-session-memory-compaction">
    探索一个实际实现，它使用后台线程和提示缓存，通过即时会话记忆压缩来管理长时间运行的对话。
  </Card>
</CardGroup>

## 可用性

阈值压缩在部分模型上处于 beta 阶段，并具备零数据保留资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：通过 `context_management.edits` 中的 `compact_20260112` 编辑启用。当输入令牌达到触发值时开始压缩，触发值默认为 150,000，最低为 50,000。

| 状态   | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview) |
| Beta | [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                                                                                                                                                                                                                                                                                                                                                                                     | [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview)                                                                                                                                                                                                                                                                                                                                                                                  |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启
* 用法：它会压缩会话的对话历史，使其适合上下文窗口，每次压缩都会发出一个 `agent.thread_context_compacted` 事件。

| 状态   | 云平台                                                                                                                   | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/haiku-4-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

| 请求头                  | API                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| `compact-2026-01-12` | [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 不具备资格 |
| 详细信息 | 服务器端压缩结果通过 API 响应以无状态方式返回和往返传递。                                                                                                           | —     |

适用的端点：

* `/v1/messages` 使用 `context_management`
