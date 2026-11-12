---
title: 停止原因与回退
url: https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons
description: 了解每个 stop_reason 值的含义，以及如何在您的应用程序中处理截断、工具使用、暂停的轮次和拒绝。
---

每个 Messages API 响应都包含一个 [`stop_reason`](https://platform.claude.com/docs/zh-CN/api/messages/create#message.stop_reason) 字段，用于告诉您 Claude 停止生成的原因。检查此字段以决定是按原样使用响应、继续对话、重试，还是回退到另一个模型。

有关完整的响应模式，请参阅 [Messages API 参考](https://platform.claude.com/docs/zh-CN/api/messages/create)。

## 快速参考

| 值                                                                                                                                               | 何时出现                             | 应对措施                                                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| [`end_turn`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#end-turn)                                           | Claude 自然地完成了其响应。                | 使用该响应。                                                                                                                                 |
| [`max_tokens`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#max-tokens)                                       | 响应达到了您的 `max_tokens` 限制。         | 提高 `max_tokens` 或[继续生成响应](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#ensuring-complete-responses)。 |
| [`stop_sequence`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#stop-sequence)                                 | Claude 输出了您的某个 `stop_sequences`。 | 读取 `stop_sequence` 以查看触发的是哪一个。                                                                                                         |
| [`tool_use`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#tool-use)                                           | Claude 正在调用工具。                   | 运行该工具并返回结果。仍缺少结果块的服务器工具调用会在后续响应中完成。                                                                                                    |
| [`pause_turn`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#pause-turn)                                       | 服务器工具循环达到了其迭代上限。                 | 将助手内容发回以继续。                                                                                                                            |
| [`refusal`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#refusal)                                             | Claude 拒绝响应。                     | 读取 `stop_details` 并[在回退模型上重试](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)。                         |
| [`model_context_window_exceeded`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#model-context-window-exceeded) | 响应填满了模型的上下文窗口。                   | 将响应视为已截断。                                                                                                                              |

## stop\_reason 字段

`stop_reason` 字段是每个成功的 Messages API 响应的一部分。与表示请求处理失败的错误不同，`stop_reason` 告诉您 Claude 为何完成了响应生成。

```json Example response
{
  "id": "msg_01234",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Here's the answer to your question..."
    }
  ],
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "stop_details": null,
  "usage": {
    "input_tokens": 100,
    "output_tokens": 50
  }
}
```

## 停止原因值

### end\_turn

最常见的停止原因。表示 Claude 自然地完成了其响应。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello!"}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello!"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello!"}],
  )
  if response.stop_reason == "end_turn":
      # 处理完整的响应
      for block in response.content:
          if block.type == "text":
              print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello!" }]
  });

  if (response.stop_reason === "end_turn") {
    // 处理完整的响应
    const textBlock = response.content.find((block) => block.type === "text");
    console.log(textBlock?.text);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello!" }]
  });

  if (response.StopReason == "end_turn")
  {
      // 处理完整的响应
      foreach (var block in response.Content)
      {
          if (block.TryPickText(out var textBlock))
          {
              Console.WriteLine(textBlock.Text);
          }
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello!")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "end_turn" {
  	// 处理完整的响应
  	for _, block := range response.Content {
  		if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  			fmt.Println(textBlock.Text)
  		}
  	}
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("Hello!")
          .build()
  );

  if (response.stopReason().map(StopReason.END_TURN::equals).orElse(false)) {
      // 处理完整的响应
      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello!']],
      model: 'claude-opus-5-5',
  );

  if ($response->stopReason === 'end_turn') {
      // 处理完整的响应
      foreach ($response->content as $block) {
          if ($block->type === 'text') {
              echo $block->text, PHP_EOL;
          }
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello!" }]
  )

  if response.stop_reason == :end_turn
    # 处理完整的响应
    response.content.each do |block|
      puts block.text if block.type == :text
    end
  end
  ```
</CodeGroup>

<Accordion title="带有 end_turn 的空响应">
  有时 Claude 会返回一个空响应（恰好 2–3 个令牌且没有内容），并带有 `stop_reason: "end_turn"`。这通常发生在 Claude 判断助手轮次已完成时，尤其是在工具结果之后。

  **常见原因：**

  * 在工具结果之后紧接着添加文本块（这样历史记录中每个工具结果之后都会出现用户文本，因此 Claude 会结束其轮次以遵循该模式）
  * 将 Claude 已完成的响应原样发回而不添加任何内容（Claude 已经判定自己完成了，因此它会保持完成状态）

  **如何防止空响应：**

  <CodeGroup exclude="shell">
    ```python Python
    # 错误：在 tool_result 之后立即添加文本
    messages = [
        {"role": "user", "content": "Calculate the sum of 1234 and 5678"},
        {
            "role": "assistant",
            "content": [
                {
                    "type": "tool_use",
                    "id": "toolu_123",
                    "name": "calculator",
                    "input": {"operation": "add", "a": 1234, "b": 5678},
                }
            ],
        },
        {
            "role": "user",
            "content": [
                {"type": "tool_result", "tool_use_id": "toolu_123", "content": "6912"},
                {
                    "type": "text",
                    "text": "Here's the result",  # Don't add text after tool_result
                },
            ],
        },
    ]

    # 正确：直接发送工具结果，不附加额外文本
    messages = [
        {"role": "user", "content": "Calculate the sum of 1234 and 5678"},
        {
            "role": "assistant",
            "content": [
                {
                    "type": "tool_use",
                    "id": "toolu_123",
                    "name": "calculator",
                    "input": {"operation": "add", "a": 1234, "b": 5678},
                }
            ],
        },
        {
            "role": "user",
            "content": [
                {"type": "tool_result", "tool_use_id": "toolu_123", "content": "6912"}
            ],
        },  # Only the tool_result, no additional text
    ]
    ```

    ```typescript TypeScript
    // 错误：在 tool_result 之后立即添加文本
    let messages: Anthropic.MessageParam[] = [
      { role: "user", content: "Calculate the sum of 1234 and 5678" },
      {
        role: "assistant",
        content: [
          {
            type: "tool_use",
            id: "toolu_123",
            name: "calculator",
            input: { operation: "add", a: 1234, b: 5678 }
          }
        ]
      },
      {
        role: "user",
        content: [
          { type: "tool_result", tool_use_id: "toolu_123", content: "6912" },
          { type: "text", text: "Here's the result" } // Don't add text after tool_result
        ]
      }
    ];

    // 正确：直接发送工具结果，不附加额外文本
    messages = [
      { role: "user", content: "Calculate the sum of 1234 and 5678" },
      {
        role: "assistant",
        content: [
          {
            type: "tool_use",
            id: "toolu_123",
            name: "calculator",
            input: { operation: "add", a: 1234, b: 5678 }
          }
        ]
      },
      {
        role: "user",
        // 仅包含 tool_result，不附加额外文本
        content: [{ type: "tool_result", tool_use_id: "toolu_123", content: "6912" }]
      }
    ];
    ```

    ```csharp C#
    using System.Text.Json;
    using Anthropic.Models.Messages;

    var input = JsonSerializer.Deserialize<Dictionary<string, JsonElement>>(
        """{"operation":"add","a":1234,"b":5678}"""
    )!;

    // 错误：在 tool_result 之后立即添加文本
    List<MessageParam> messages =
    [
        new UserMessageParam { Content = "Calculate the sum of 1234 and 5678" },
        new AssistantMessageParam
        {
            Content = new List<ContentBlockParam>
            {
                new ToolUseBlockParam { ID = "toolu_123", Name = "calculator", Input = input }
            }
        },
        new UserMessageParam
        {
            Content = new List<ContentBlockParam>
            {
                new ToolResultBlockParam { ToolUseID = "toolu_123", Content = "6912" },
                new TextBlockParam { Text = "Here's the result" } // Don't add text after tool_result
            }
        }
    ];

    // 正确：直接发送工具结果，不附加额外文本
    messages =
    [
        new UserMessageParam { Content = "Calculate the sum of 1234 and 5678" },
        new AssistantMessageParam
        {
            Content = new List<ContentBlockParam>
            {
                new ToolUseBlockParam { ID = "toolu_123", Name = "calculator", Input = input }
            }
        },
        new UserMessageParam
        {
            // 仅包含 tool_result，不附加额外文本
            Content = new List<ContentBlockParam>
            {
                new ToolResultBlockParam { ToolUseID = "toolu_123", Content = "6912" }
            }
        }
    ];
    ```

    ```go Go
    input := map[string]any{"operation": "add", "a": 1234, "b": 5678}

    // 错误：在 tool_result 之后立即添加文本
    messages := []anthropic.MessageParam{
    	anthropic.NewUserMessage(anthropic.NewTextBlock("Calculate the sum of 1234 and 5678")),
    	anthropic.NewAssistantMessage(
    		anthropic.NewToolUseBlock("toolu_123", input, "calculator"),
    	),
    	anthropic.NewUserMessage(
    		anthropic.NewToolResultBlock("toolu_123", "6912", false),
    		anthropic.NewTextBlock("Here's the result"), // Don't add text after tool_result
    	),
    }

    // 正确：直接发送工具结果，不附加额外文本
    messages = []anthropic.MessageParam{
    	anthropic.NewUserMessage(anthropic.NewTextBlock("Calculate the sum of 1234 and 5678")),
    	anthropic.NewAssistantMessage(
    		anthropic.NewToolUseBlock("toolu_123", input, "calculator"),
    	),
    	// 仅包含 tool_result，不附加额外文本
    	anthropic.NewUserMessage(
    		anthropic.NewToolResultBlock("toolu_123", "6912", false),
    	),
    }
    ```

    ```java Java
    ToolUseBlockParam toolUse = ToolUseBlockParam.builder()
        .id("toolu_123")
        .name("calculator")
        .input(ToolUseBlockParam.Input.builder()
            .putAdditionalProperty("operation", JsonValue.from("add"))
            .putAdditionalProperty("a", JsonValue.from(1234))
            .putAdditionalProperty("b", JsonValue.from(5678))
            .build())
        .build();

    // 错误：在 tool_result 之后紧接着添加文本
    List<MessageParam> messages = List.of(
        MessageParam.ofUser("Calculate the sum of 1234 and 5678"),
        MessageParam.ofAssistantOfContentBlockParams(List.of(ContentBlockParam.ofToolUse(toolUse))),
        MessageParam.ofUserOfContentBlockParams(List.of(
            ContentBlockParam.ofToolResult(
                ToolResultBlockParam.builder()
                    .toolUseId("toolu_123")
                    .content("6912")
                    .build()),
            // 不要在 tool_result 之后添加文本
            ContentBlockParam.ofText(TextBlockParam.builder()
                .text("Here's the result")
                .build())
        ))
    );

    // 正确：直接发送工具结果，不附加额外文本
    messages = List.of(
        MessageParam.ofUser("Calculate the sum of 1234 and 5678"),
        MessageParam.ofAssistantOfContentBlockParams(List.of(ContentBlockParam.ofToolUse(toolUse))),
        // 仅包含 tool_result，不附加任何文本
        MessageParam.ofUserOfContentBlockParams(List.of(
            ContentBlockParam.ofToolResult(
                ToolResultBlockParam.builder()
                    .toolUseId("toolu_123")
                    .content("6912")
                    .build())
        ))
    );
    ```

    ```php PHP
    // 错误：在 tool_result 之后立即添加文本
    $messages = [
        ['role' => 'user', 'content' => 'Calculate the sum of 1234 and 5678'],
        [
            'role' => 'assistant',
            'content' => [
                [
                    'type' => 'tool_use',
                    'id' => 'toolu_123',
                    'name' => 'calculator',
                    'input' => ['operation' => 'add', 'a' => 1234, 'b' => 5678],
                ],
            ],
        ],
        [
            'role' => 'user',
            'content' => [
                ['type' => 'tool_result', 'tool_use_id' => 'toolu_123', 'content' => '6912'],
                // 不要在 tool_result 之后添加文本
                ['type' => 'text', 'text' => "Here's the result"],
            ],
        ],
    ];

    // 正确：直接发送工具结果，不附加额外文本
    $messages = [
        ['role' => 'user', 'content' => 'Calculate the sum of 1234 and 5678'],
        [
            'role' => 'assistant',
            'content' => [
                [
                    'type' => 'tool_use',
                    'id' => 'toolu_123',
                    'name' => 'calculator',
                    'input' => ['operation' => 'add', 'a' => 1234, 'b' => 5678],
                ],
            ],
        ],
        [
            'role' => 'user',
            // 仅包含 tool_result，不附加额外文本
            'content' => [
                ['type' => 'tool_result', 'tool_use_id' => 'toolu_123', 'content' => '6912'],
            ],
        ],
    ];
    ```

    ```ruby Ruby
    # 错误：在 tool_result 之后紧接着添加文本
    messages = [
      { role: "user", content: "Calculate the sum of 1234 and 5678" },
      {
        role: "assistant",
        content: [
          {
            type: "tool_use",
            id: "toolu_123",
            name: "calculator",
            input: { operation: "add", a: 1234, b: 5678 }
          }
        ]
      },
      {
        role: "user",
        content: [
          { type: "tool_result", tool_use_id: "toolu_123", content: "6912" },
          # 不要在 tool_result 之后添加文本
          { type: "text", text: "Here's the result" }
        ]
      }
    ]

    # 正确：直接发送工具结果，不附加额外文本
    messages = [
      { role: "user", content: "Calculate the sum of 1234 and 5678" },
      {
        role: "assistant",
        content: [
          {
            type: "tool_use",
            id: "toolu_123",
            name: "calculator",
            input: { operation: "add", a: 1234, b: 5678 }
          }
        ]
      },
      {
        role: "user",
        # 仅包含 tool_result，不附加额外文本
        content: [
          { type: "tool_result", tool_use_id: "toolu_123", content: "6912" }
        ]
      }
    ]
    ```
  </CodeGroup>

  如果在修正消息结构后仍然得到空响应，请在新的用户消息中添加一个继续提示，而不是用空响应重试：

  <CodeGroup exclude="shell">
    ```python Python
    def handle_empty_response(client, messages):
        response = client.messages.create(
            model="claude-opus-5-5", max_tokens=1024, messages=messages
        )

        # 检查响应是否为空
        if response.stop_reason == "end_turn" and not response.content:
            # 错误做法：不要直接用空响应重试
            # 这样行不通，因为 Claude 已经判定自己完成了

            # 正确做法：在一条新的用户消息中添加继续提示
            messages.append({"role": "user", "content": "Please continue"})

            response = client.messages.create(
                model="claude-opus-5-5", max_tokens=1024, messages=messages
            )

        return response
    ```

    ```typescript TypeScript
    async function handleEmptyResponse(
      client: Anthropic,
      messages: Anthropic.MessageParam[]
    ): Promise<Anthropic.Message> {
      let response = await client.messages.create({
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages
      });

      // 检查响应是否为空
      if (response.stop_reason === "end_turn" && response.content.length === 0) {
        // 错误做法：不要直接用空响应重试
        // 这样行不通，因为 Claude 已经判定自己完成了

        // 正确做法：在一条新的用户消息中添加继续提示
        messages.push({ role: "user", content: "Please continue" });

        response = await client.messages.create({
          model: "claude-opus-5-5",
          max_tokens: 1024,
          messages
        });
      }

      return response;
    }
    ```

    ```csharp C#
    static async Task<Message> HandleEmptyResponse(AnthropicClient client, List<MessageParam> messages)
    {
        var response = await client.Messages.Create(new MessageCreateParams
        {
            Model = Model.ClaudeOpus5_5,
            MaxTokens = 1024,
            Messages = messages
        });

        // 检查响应是否为空
        if (response.StopReason == "end_turn" && response.Content.Count == 0)
        {
            // 正确做法：在一条新的用户消息中添加继续提示
            messages.Add(new UserMessageParam { Content = "Please continue" });

            response = await client.Messages.Create(new MessageCreateParams
            {
                Model = Model.ClaudeOpus5_5,
                MaxTokens = 1024,
                Messages = messages
            });
        }

        return response;
    }
    ```

    ```go Go
    func handleEmptyResponse(client anthropic.Client, messages []anthropic.MessageParam) (*anthropic.Message, error) {
    	response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    		Model:     anthropic.ModelClaudeOpus5_5,
    		MaxTokens: 1024,
    		Messages:  messages,
    	})
    	if err != nil {
    		return nil, err
    	}

    	// 检查响应是否为空
    	if response.StopReason == "end_turn" && len(response.Content) == 0 {
    		// 正确做法：在一条新的 user 消息中添加继续提示
    		messages = append(messages, anthropic.NewUserMessage(anthropic.NewTextBlock("Please continue")))

    		response, err = client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    			Model:     anthropic.ModelClaudeOpus5_5,
    			MaxTokens: 1024,
    			Messages:  messages,
    		})
    		if err != nil {
    			return nil, err
    		}
    	}

    	return response, nil
    }
    ```

    ```java Java
    static Message handleEmptyResponse(AnthropicClient client, List<MessageParam> messages) {
        Message response = client.messages().create(
            MessageCreateParams.builder()
                .model(Model.CLAUDE_OPUS_5_5)
                .maxTokens(1024L)
                .messages(messages)
                .build()
        );

        // 检查响应是否为空
        boolean isEndTurn = response.stopReason().map(StopReason.END_TURN::equals).orElse(false);
        if (isEndTurn && response.content().isEmpty()) {
            // 正确做法：在一条新的用户消息中添加继续提示
            List<MessageParam> extended = new ArrayList<>(messages);
            extended.add(MessageParam.ofUser("Please continue"));

            response = client.messages().create(
                MessageCreateParams.builder()
                    .model(Model.CLAUDE_OPUS_5_5)
                    .maxTokens(1024L)
                    .messages(extended)
                    .build()
            );
        }

        return response;
    }
    ```

    ```php PHP
    function handle_empty_response(Client $client, array $messages)
    {
        $response = $client->messages->create(
            maxTokens: 1024,
            messages: $messages,
            model: 'claude-opus-5-5',
        );

        // 检查响应是否为空
        if ($response->stopReason === 'end_turn' && count($response->content) === 0) {
            // 正确做法：在一条新的用户消息中添加继续提示
            $messages[] = ['role' => 'user', 'content' => 'Please continue'];

            $response = $client->messages->create(
                maxTokens: 1024,
                messages: $messages,
                model: 'claude-opus-5-5',
            );
        }

        return $response;
    }
    ```

    ```ruby Ruby
    def handle_empty_response(client, messages)
      response = client.messages.create(
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages: messages
      )

      # 检查响应是否为空
      if response.stop_reason == :end_turn && response.content.empty?
        # 正确做法：在一条新的用户消息中添加继续提示
        messages << { role: "user", content: "Please continue" }

        response = client.messages.create(
          model: "claude-opus-5-5",
          max_tokens: 1024,
          messages: messages
        )
      end

      response
    end
    ```
  </CodeGroup>

  **最佳实践：**

  1. **切勿在工具结果之后立即添加文本块：** 这会形成每次工具使用后都有用户输入的模式，而 Claude 会通过结束其轮次来遵循该模式。
  2. **不要不加修改地重试空响应：** 将空响应发回不会有帮助。
  3. **将继续提示作为最后手段：** 仅在上述修正无法解决问题时使用。
</Accordion>

### max\_tokens

Claude 因达到您请求中指定的 [`max_tokens`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.max_tokens) 限制而停止。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 10,
      "messages": [{"role": "user", "content": "Explain quantum physics"}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 10 \
    --message '{role: user, content: "Explain quantum physics"}'
  ```

  ```python Python
  client = anthropic.Anthropic()
  # 限制令牌数的请求
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=10,
      messages=[{"role": "user", "content": "Explain quantum physics"}],
  )

  if response.stop_reason == "max_tokens":
      # 响应已被截断
      print("Response was cut off at token limit")
      # 可考虑再发送一次请求以继续
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 限制令牌数量的请求
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 10,
    messages: [{ role: "user", content: "Explain quantum physics" }]
  });

  if (response.stop_reason === "max_tokens") {
    // 响应已被截断
    console.log("Response was cut off at token limit");
    // 可考虑再发起一次请求以继续
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  // 限制令牌数量的请求
  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 10,
      Messages = [new UserMessageParam { Content = "Explain quantum physics" }]
  });

  if (response.StopReason == "max_tokens")
  {
      // 响应已被截断
      Console.WriteLine("Response was cut off at token limit");
      // 可考虑再发起一次请求以继续
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  // 限制令牌数量的请求
  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 10,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Explain quantum physics")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "max_tokens" {
  	// 响应已被截断
  	fmt.Println("Response was cut off at token limit")
  	// 可考虑再发起一次请求以继续
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  // 限制令牌数量的请求
  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(10L)
          .addUserMessage("Explain quantum physics")
          .build()
  );

  if (response.stopReason().map(StopReason.MAX_TOKENS::equals).orElse(false)) {
      // 响应已被截断
      IO.println("Response was cut off at token limit");
      // 可考虑再发起一次请求以继续生成
  }
  ```

  ```php PHP
  $client = new Client();

  // 限制令牌数量的请求
  $response = $client->messages->create(
      maxTokens: 10,
      messages: [['role' => 'user', 'content' => 'Explain quantum physics']],
      model: Model::CLAUDE_OPUS_5_5,
  );

  if ($response->stopReason === 'max_tokens') {
      // 响应已被截断
      echo 'Response was cut off at token limit', PHP_EOL;
      // 可考虑再发起一次请求以继续
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 限制令牌数量的请求
  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 10,
    messages: [{ role: "user", content: "Explain quantum physics" }]
  )

  if response.stop_reason == :max_tokens
    # 响应已被截断
    puts "Response was cut off at token limit"
    # 可考虑再发送一次请求以继续
  end
  ```
</CodeGroup>

<Accordion title="不完整的工具使用块">
  如果 Claude 的响应因达到 `max_tokens` 限制而被截断，并且截断的响应包含一个不完整的工具使用块，您需要使用更高的 `max_tokens` 值重试请求，以获取完整的工具使用。

  <CodeGroup exclude="shell:cURL, shell:CLI">
    <CodeGroupItem language="cURL">
      此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
    </CodeGroupItem>

    <CodeGroupItem language="CLI">
      此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
    </CodeGroupItem>

    ```python Python
    # 检查响应是否在 tool use（工具使用）过程中被截断
    if response.stop_reason == "max_tokens":
        # 检查最后一个内容块是否为不完整的 tool_use
        last_block = response.content[-1]
        if last_block.type == "tool_use":
            # 使用更高的 max_tokens 发送请求
            response = client.messages.create(
                model="claude-opus-5-5",
                max_tokens=4096,  # Increased limit
                messages=messages,
                tools=tools,
            )
    ```

    ```typescript TypeScript
    // 检查响应是否在工具使用过程中被截断
    if (response.stop_reason === "max_tokens") {
      // 检查最后一个内容块是否为不完整的 tool_use
      const lastBlock = response.content[response.content.length - 1];
      if (lastBlock.type === "tool_use") {
        // 使用更高的 max_tokens 发送请求
        response = await client.messages.create({
          model: "claude-opus-5-5",
          max_tokens: 4096, // Increased limit
          messages: messages,
          tools: tools
        });
      }
    }
    ```

    ```csharp C#
    using System.Linq;
    using Anthropic;
    using Anthropic.Models.Messages;

    AnthropicClient client = new();

    var parameters = new MessageCreateParams
    {
        Model = Model.ClaudeOpus5_5,
        MaxTokens = 1024,
        Messages = messages,
        Tools = tools
    };

    var response = await client.Messages.Create(parameters);

    if (response.StopReason == "max_tokens")
    {
        var lastBlock = response.Content.Last();
        if (lastBlock.TryPickToolUse(out _))
        {
            response = await client.Messages.Create(parameters with { MaxTokens = 4096 });
        }
    }
    ```

    ```go Go
    response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    	Model:     anthropic.ModelClaudeOpus5_5,
    	MaxTokens: 1024,
    	Messages:  messages,
    	Tools:     tools,
    })
    if err != nil {
    	log.Fatal(err)
    }

    if response.StopReason == "max_tokens" {
    	lastBlock := response.Content[len(response.Content)-1]
    	switch lastBlock.AsAny().(type) {
    	case anthropic.ToolUseBlock:
    		response, err = client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    			Model:     anthropic.ModelClaudeOpus5_5,
    			MaxTokens: 4096,
    			Messages:  messages,
    			Tools:     tools,
    		})
    		if err != nil {
    			log.Fatal(err)
    		}
    	}
    }
    ```

    ```java Java
    // 检查响应是否在工具使用过程中被截断
    if (response.stopReason().isPresent() && response.stopReason().get().equals(StopReason.MAX_TOKENS)) {
        ContentBlock lastBlock = response.content().get(response.content().size() - 1);
        if (lastBlock.toolUse().isPresent()) {
            // 使用更高的 max_tokens 重新发送请求
            response = client.messages().create(
                MessageCreateParams.builder()
                    .model(Model.CLAUDE_OPUS_5_5)
                    .maxTokens(4096L) // Increased limit
                    .messages(messages)
                    .tools(tools)
                    .build()
            );
        }
    }
    ```

    ```php PHP
    $response = $client->messages->create(
        maxTokens: 1024,
        messages: $messages,
        model: 'claude-opus-5-5',
        tools: $tools,
    );

    if ($response->stopReason === 'max_tokens') {
        $lastBlock = end($response->content);
        if ($lastBlock->type === 'tool_use') {
            $response = $client->messages->create(
                maxTokens: 4096,
                messages: $messages,
                model: 'claude-opus-5-5',
                tools: $tools,
            );
        }
    }
    ```

    ```ruby Ruby
    response = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      messages: messages,
      tools: tools
    )

    if response.stop_reason == :max_tokens
      last_block = response.content.last
      if last_block.type == :tool_use
        response = client.messages.create(
          model: Anthropic::Model::CLAUDE_OPUS_5_5,
          max_tokens: 4096,
          messages: messages,
          tools: tools
        )
      end
    end
    ```
  </CodeGroup>
</Accordion>

### stop\_sequence

Claude 遇到了您的某个自定义[停止序列](https://platform.claude.com/docs/zh-CN/api/messages/create#create.stop_sequences)。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "stop_sequences": ["END", "STOP"],
      "messages": [{"role": "user", "content": "Generate text until you say END"}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --stop-sequence END --stop-sequence STOP \
    --message '{role: user, content: "Generate text until you say END"}'
  ```

  ```python Python
  client = anthropic.Anthropic()
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      stop_sequences=["END", "STOP"],
      messages=[{"role": "user", "content": "Generate text until you say END"}],
  )

  if response.stop_reason == "stop_sequence":
      print(f"Stopped at sequence: {response.stop_sequence}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    stop_sequences: ["END", "STOP"],
    messages: [{ role: "user", content: "Generate text until you say END" }]
  });

  if (response.stop_reason === "stop_sequence") {
    console.log(`Stopped at sequence: ${response.stop_sequence}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      StopSequences = ["END", "STOP"],
      Messages = [new UserMessageParam { Content = "Generate text until you say END" }]
  });

  if (response.StopReason == "stop_sequence")
  {
      Console.WriteLine($"Stopped at sequence: {response.StopSequence}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:         anthropic.ModelClaudeOpus5_5,
  	MaxTokens:     1024,
  	StopSequences: []string{"END", "STOP"},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Generate text until you say END")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "stop_sequence" {
  	fmt.Printf("Stopped at sequence: %s\n", response.StopSequence)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addStopSequence("END")
          .addStopSequence("STOP")
          .addUserMessage("Generate text until you say END")
          .build()
  );

  if (response.stopReason().map(StopReason.STOP_SEQUENCE::equals).orElse(false)) {
      IO.println("Stopped at sequence: " + response.stopSequence().orElse(""));
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Generate text until you say END']],
      model: Model::CLAUDE_OPUS_5_5,
      stopSequences: ['END', 'STOP'],
  );

  if ($response->stopReason === 'stop_sequence') {
      echo "Stopped at sequence: {$response->stopSequence}", PHP_EOL;
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    stop_sequences: ["END", "STOP"],
    messages: [{ role: "user", content: "Generate text until you say END" }]
  )

  if response.stop_reason == :stop_sequence
    puts "Stopped at sequence: #{response.stop_sequence}"
  end
  ```
</CodeGroup>

### tool\_use

Claude 正在调用一个需要您运行的工具。

<Note>
  对于大多数工具使用实现，请使用[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)，它会自动处理工具执行、结果格式化和对话管理。
</Note>

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "tools": [{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
          "type": "object",
          "properties": {"location": {"type": "string", "description": "City and state"}},
          "required": ["location"]
        }
      }],
      "messages": [{"role": "user", "content": "What is the weather in San Francisco?"}]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  messages:
    - role: user
      content: What is the weather in San Francisco?
  tools:
    - name: get_weather
      description: Get the current weather in a given location
      input_schema:
        type: object
        properties:
          location: {type: string, description: City and state}
        required: [location]
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()
  weather_tool = {
      "name": "get_weather",
      "description": "Get the current weather in a given location",
      "input_schema": {
          "type": "object",
          "properties": {
              "location": {"type": "string", "description": "City and state"},
          },
          "required": ["location"],
      },
  }


  def execute_tool(name, tool_input):
      """Run a tool and return the result."""
      return f"Weather in {tool_input.get('location', 'unknown')}: 72°F"


  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[weather_tool],
      messages=[{"role": "user", "content": "What is the weather in San Francisco?"}],
  )

  if response.stop_reason == "tool_use":
      # 提取并运行工具
      for block in response.content:
          if block.type == "tool_use":
              result = execute_tool(block.name, block.input)
              # 将结果返回给 Claude 以生成最终响应
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const weatherTool: Anthropic.Tool = {
    name: "get_weather",
    description: "Get the current weather in a given location",
    input_schema: {
      type: "object",
      properties: {
        location: { type: "string", description: "City and state" }
      },
      required: ["location"]
    }
  };

  function executeTool(name: string, input: Record<string, string>): string {
    return `Weather in ${input.location ?? "unknown"}: 72°F`;
  }

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [weatherTool],
    messages: [{ role: "user", content: "What is the weather in San Francisco?" }]
  });

  if (response.stop_reason === "tool_use") {
    // 提取并运行工具
    for (const block of response.content) {
      if (block.type === "tool_use") {
        const result = executeTool(block.name, block.input as Record<string, string>);
        // 将结果返回给 Claude 以生成最终响应
      }
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var weatherTool = new Tool
  {
      Name = "get_weather",
      Description = "Get the current weather in a given location",
      InputSchema = new InputSchema
      {
          Properties = new Dictionary<string, JsonElement>
          {
              ["location"] = JsonSerializer.SerializeToElement(
                  new { type = "string", description = "City and state" }
              ),
          },
          Required = ["location"]
      }
  };

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = [weatherTool],
      Messages = [new UserMessageParam { Content = "What is the weather in San Francisco?" }]
  });

  if (response.StopReason == "tool_use")
  {
      // 提取并运行工具
      foreach (var block in response.Content)
      {
          if (block.TryPickToolUse(out var toolUse))
          {
              // 使用 toolUse.Input 运行 toolUse.Name，并将结果返回给 Claude
          }
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  weatherTool := anthropic.ToolParam{
  	Name:        "get_weather",
  	Description: anthropic.String("Get the current weather in a given location"),
  	InputSchema: anthropic.ToolInputSchemaParam{
  		Properties: map[string]any{
  			"location": map[string]string{"type": "string", "description": "City and state"},
  		},
  		Required: []string{"location"},
  	},
  }

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools:     []anthropic.ToolUnionParam{{OfTool: &weatherTool}},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is the weather in San Francisco?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "tool_use" {
  	// 提取并运行工具
  	for _, block := range response.Content {
  		if toolUse, ok := block.AsAny().(anthropic.ToolUseBlock); ok {
  			fmt.Println(toolUse.Name, toolUse.Input)
  			// 将结果返回给 Claude 以生成最终响应
  		}
  	}
  }
  ```

  ```java Java
  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Tool weatherTool = Tool.builder()
          .name("get_weather")
          .description("Get the current weather in a given location")
          .inputSchema(Tool.InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "location", Map.of("type", "string", "description", "City and state")
              )))
              .putAdditionalProperty("required", JsonValue.from(List.of("location")))
              .build())
          .build();

      Message response = client.messages().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024L)
              .addTool(weatherTool)
              .addUserMessage("What is the weather in San Francisco?")
              .build()
      );

      if (response.stopReason().map(StopReason.TOOL_USE::equals).orElse(false)) {
          // 提取并运行工具
          for (ContentBlock block : response.content()) {
              block.toolUse().ifPresent(toolUse -> {
                  // 使用 toolUse.input() 运行 toolUse.name()，并将结果返回给 Claude
              });
          }
      }
  ```

  ```php PHP
  $client = new Client();

  $weatherTool = [
      'name' => 'get_weather',
      'description' => 'Get the current weather in a given location',
      'input_schema' => [
          'type' => 'object',
          'properties' => [
              'location' => ['type' => 'string', 'description' => 'City and state'],
          ],
          'required' => ['location'],
      ],
  ];

  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'What is the weather in San Francisco?']],
      model: 'claude-opus-5-5',
      tools: [$weatherTool],
  );

  if ($response->stopReason === 'tool_use') {
      // 提取并运行工具
      foreach ($response->content as $block) {
          if ($block->type === 'tool_use') {
              // 使用 $block->input 运行 $block->name，并将结果返回给 Claude
          }
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  weather_tool = {
    name: "get_weather",
    description: "Get the current weather in a given location",
    input_schema: {
      type: "object",
      properties: {
        location: { type: "string", description: "City and state" }
      },
      required: ["location"]
    }
  }

  response = client.messages.create(
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [weather_tool],
    messages: [{ role: "user", content: "What is the weather in San Francisco?" }]
  )

  if response.stop_reason == :tool_use
    # 提取并运行工具
    response.content.each do |block|
      next unless block.type == :tool_use
      # 使用 block.input 运行 block.name，并将结果返回给 Claude
    end
  end
  ```
</CodeGroup>

`tool_use` 响应还可能包含一个 `server_tool_use` 块，其 `id` 没有匹配的结果块。该服务器工具调用尚未完成，并且此响应不包含其结果。常见情况是，Claude 在同一组并行工具调用中调用了一个[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)和您的某个客户端工具：API 在不运行服务器工具的情况下返回，以便您可以先运行客户端工具。此状态没有其他标记；可以通过检查每个 `server_tool_use` 或 `mcp_tool_use` 块的 `id` 是否有匹配的结果块来检测。

<Note>
  使用 [programmatic tool calling](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)（程序化工具调用）时，相同的响应形态含义不同。客户端 `tool_use` 块来自在 `code_execution` 工具中运行的代码，而不是直接来自 Claude，其 `caller` 字段指明调用它的 `code_execution` 块。该代码已经开始运行：它处于暂停状态，等待您的 `tool_result` 块，发送这些块会恢复执行，而不是启动一个延迟的工具。`code_execution` 块自身的结果块会在代码完成后到达，这可能需要不止一轮工具结果。两种情况下的后续用户消息本身是相同的；使用程序化工具调用时，还需按该页面所示传回响应的 `container` 字段中的 `id`。
</Note>

```json A mixed tool_use response
{
  "stop_reason": "tool_use",
  "content": [
    {
      "type": "server_tool_use",
      "id": "srvtoolu_01HxbWnMRmbWyMfUtJKC45rA",
      "name": "web_search",
      "input": { "query": "example article" }
    },
    {
      "type": "tool_use",
      "id": "toolu_01PjgRJLbXrXEMZwDNYLnBqk",
      "name": "run_command",
      "input": { "command": "uname -a" }
    }
  ]
}
```

继续的方式是发送一条由 `tool_result` 块组成的用户消息，响应中的每个 `tool_use` 块对应一个（参见[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)），并附加两条规则：该消息除 `tool_result` 块外不得包含任何其他内容，且请求必须保持相同的 `tools` 数组。如果恢复请求不再定义正在等待的服务器工具，则会以 400 失败，其消息以 ``but no `web_search` tool was provided`` 结尾。API 会将您的结果附加到仍处于打开状态的助手轮次，运行被延迟的服务器工具（对于暂停的代码执行，则恢复它），并继续该轮次。对于 Claude 直接调用的服务器工具，下一个响应的 `content` 以回应上一个响应中 `server_tool_use` `id` 的结果块开头。

```json The follow-up user message
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01PjgRJLbXrXEMZwDNYLnBqk",
      "content": "Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux"
    }
  ]
}
```

在该用户消息的 `tool_result` 块之后添加任何内容（例如文本）都会结束助手轮次；对于 Claude 直接调用的服务器工具，请求随后会以 400 `invalid_request_error` 失败，并指明未解决的服务器工具：

```text wrap
`web_search` tool use with id `srvtoolu_01HxbWnMRmbWyMfUtJKC45rA` was found without a corresponding `web_search_tool_result` block
```

遗漏某个 `tool_result`，或将其放在其他内容之后，则会更早地以标准的 `tool_use ids were found without tool_result blocks immediately after` 错误失败。若要向 Claude 提供更多输入，请在轮次完成后将其作为单独的用户消息发送。

### pause\_turn

当服务器端采样循环在运行[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)（例如网络搜索）时达到其迭代限制时返回。默认限制为每个请求 10 次迭代。

发生这种情况时，响应可能包含一个没有对应结果块的 `server_tool_use` 块。要让 Claude 完成处理，请将响应原样发送回去以继续对话。留有等待您处理的客户端 `tool_use` 块的响应，其 `stop_reason` 永远不会是 `pause_turn`：当 Claude 停下来调用您的工具时，`stop_reason` 是 [`tool_use`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#tool-use)，您需要通过发送客户端 `tool_result` 块（而不是响应本身）来继续。

<CodeGroup>
  ```bash cURL
  # SDK 会直接处理续写。使用 cURL 时，请检查响应中的 stop_reason，
  # 并将 assistant 内容追加到消息后重新发送 POST 请求。
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "tools": [{"type": "web_search_20250305", "name": "web_search"}],
      "messages": [{"role": "user", "content": "Search for latest AI news"}]
    }'
  ```

  ```bash CLI
  # 检查 stop_reason；如果其值为 pause_turn，则将助手的
  # 响应追加到 --message 后重新运行。
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  tools:
    - {type: web_search_20250305, name: web_search}
  messages:
    - {role: user, content: "Search for latest AI news"}
  YAML
  ```

  ```python Python
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      tools=[{"type": "web_search_20250305", "name": "web_search"}],
      messages=[{"role": "user", "content": "Search for latest AI news"}],
  )

  if response.stop_reason == "pause_turn":
      # 将响应发送回去以继续对话
      messages = [
          {"role": "user", "content": "Search for latest AI news"},
          {"role": "assistant", "content": response.content},
      ]
      continuation = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=4096,
          messages=messages,
          tools=[{"type": "web_search_20250305", "name": "web_search"}],
      )
  ```

  ```typescript TypeScript
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    tools: [{ type: "web_search_20250305", name: "web_search" }],
    messages: [{ role: "user", content: "Search for latest AI news" }]
  });

  if (response.stop_reason === "pause_turn") {
    // 将响应发送回去以继续对话
    const continuation = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 4096,
      tools: [{ type: "web_search_20250305", name: "web_search" }],
      messages: [
        { role: "user", content: "Search for latest AI news" },
        { role: "assistant", content: response.content }
      ]
    });
  }
  ```

  ```csharp C#
  List<ToolUnion> tools = [new ToolUnion(new WebSearchTool20250305())];
  MessageParam userMessage = new UserMessageParam { Content = "Search for latest AI news" };

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Tools = tools,
      Messages = [userMessage]
  });

  if (response.StopReason == "pause_turn")
  {
      // 将响应发送回去以继续对话
      var continuation = await client.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 4096,
          Tools = tools,
          Messages =
          [
              userMessage,
              new AssistantMessageParam
              {
                  Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList()
              }
          ]
      });
  }
  ```

  ```go Go
  tools := []anthropic.ToolUnionParam{
  	{OfWebSearchTool20250305: &anthropic.WebSearchTool20250305Param{}},
  }
  userMessage := anthropic.NewUserMessage(anthropic.NewTextBlock("Search for latest AI news"))

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Tools:     tools,
  	Messages:  []anthropic.MessageParam{userMessage},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "pause_turn" {
  	// 将响应发送回去以继续对话
  	var contentParams []anthropic.ContentBlockParamUnion
  	for _, block := range response.Content {
  		contentParams = append(contentParams, block.ToParam())
  	}
  	continuation, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 4096,
  		Tools:     tools,
  		Messages:  []anthropic.MessageParam{userMessage, anthropic.NewAssistantMessage(contentParams...)},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}
  	_ = continuation
  }
  ```

  ```java Java
  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addTool(WebSearchTool20250305.builder().build())
          .addUserMessage("Search for latest AI news")
          .build()
  );

  if (response.stopReason().map(StopReason.PAUSE_TURN::equals).orElse(false)) {
      // 将响应发送回去以继续对话
      Message continuation = client.messages().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addTool(WebSearchTool20250305.builder().build())
              .addUserMessage("Search for latest AI news")
              .addMessage(response)
              .build()
      );
  }
  ```

  ```php PHP
  $tools = [['type' => 'web_search_20250305', 'name' => 'web_search']];
  $userMessage = ['role' => 'user', 'content' => 'Search for latest AI news'];

  $response = $client->messages->create(
      maxTokens: 4096,
      messages: [$userMessage],
      model: 'claude-opus-5-5',
      tools: $tools,
  );

  if ($response->stopReason === 'pause_turn') {
      // 将响应发回以继续对话
      $continuation = $client->messages->create(
          maxTokens: 4096,
          messages: [
              $userMessage,
              ['role' => 'assistant', 'content' => $response->content],
          ],
          model: 'claude-opus-5-5',
          tools: $tools,
      );
  }
  ```

  ```ruby Ruby
  tools = [{ type: "web_search_20250305", name: "web_search" }]
  user_message = { role: "user", content: "Search for latest AI news" }

  response = client.messages.create(
    model: "claude-opus-5-5",
    max_tokens: 4096,
    tools: tools,
    messages: [user_message]
  )

  if response.stop_reason == :pause_turn
    # 将响应发送回去以继续对话
    continuation = client.messages.create(
      model: "claude-opus-5-5",
      max_tokens: 4096,
      tools: tools,
      messages: [user_message, { role: "assistant", content: response.content }]
    )
  end
  ```
</CodeGroup>

<Note>
  您的应用程序应在任何使用服务器工具的智能体循环中处理 `pause_turn`。将助手的响应添加到您的消息数组中，并发起另一个 API 请求以让 Claude 继续。
</Note>

### refusal

Claude 拒绝生成响应。安全分类器以正常的 HTTP 200 响应而非错误的形式返回此停止原因。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "[Unsafe request]"}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "[Unsafe request]"}'
  ```

  ```python Python
  client = anthropic.Anthropic()
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "[Unsafe request]"}],
  )

  if response.stop_reason == "refusal":
      # Claude 拒绝了回复
      print("Claude was unable to process this request")
      # 请考虑重新表述或修改请求
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "[Unsafe request]" }]
  });

  if (response.stop_reason === "refusal") {
    // Claude 拒绝了响应
    console.log("Claude was unable to process this request");
    // 请考虑重新表述或修改请求
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "[Unsafe request]" }]
  });

  if (response.StopReason == "refusal")
  {
      // Claude 拒绝了回复
      Console.WriteLine("Claude was unable to process this request");
      // 请考虑重新表述或修改请求
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("[Unsafe request]")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == "refusal" {
  	// Claude 拒绝了回复
  	fmt.Println("Claude was unable to process this request")
  	// 请考虑重新表述或修改请求
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("[Unsafe request]")
          .build()
  );

  if (response.stopReason().map(StopReason.REFUSAL::equals).orElse(false)) {
      // Claude 拒绝了回复
      IO.println("Claude was unable to process this request");
      // 请考虑重新表述或修改请求
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => '[Unsafe request]']],
      model: Model::CLAUDE_OPUS_5_5,
  );

  if ($response->stopReason === 'refusal') {
      // Claude 拒绝了响应
      echo 'Claude was unable to process this request', PHP_EOL;
      // 请考虑重新表述或修改请求
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "[Unsafe request]" }]
  )

  if response.stop_reason == :refusal
    # Claude 拒绝了响应
    puts "Claude was unable to process this request"
    # 请考虑重新表述或修改请求
  end
  ```
</CodeGroup>

<Tip>
  如果您在使用 Claude Sonnet 4.5（[已弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)）或 Claude Opus 4.1（后者[已停用，Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 除外](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)）时频繁遇到 `refusal` 停止原因，可以尝试将 API 调用更新为使用 Claude Haiku 4.5（`claude-haiku-4-5-20251001`），它具有不同的使用限制。详细了解[理解 Sonnet 4.5 的 API 安全过滤器](https://support.claude.com/en/articles/12449294-understanding-sonnet-4-5-s-api-safety-filters)。
</Tip>

发生拒绝时，[`stop_details`](https://platform.claude.com/docs/zh-CN/api/messages/create#message.stop_details) 对象会标识触发拒绝的策略类别。这些类别以及完整的拒绝响应形态在[拒绝是什么样子的](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)中有介绍。对于 `refusal` 以外的所有停止原因，`stop_details` 均为 `null`。

被拒绝的请求通常可以通过在另一个 Claude 模型上重试来获得响应。这适用于 Claude Fable 5.2、Claude Fable 5.1、Claude Fable 5、Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5 和 Claude Haiku 5.5。[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)介绍了如何在服务器端或您的客户端中设置该重试。Claude Haiku 5.5 没有服务器端回退，因此请通过[客户端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)设置其重试。如果您自行构建从上述任一模型（Claude Haiku 5.5 除外）发起的重试，[回退抵扣](https://platform.claude.com/docs/zh-CN/build-with-claude/fallback-credit)介绍了如何避免两次支付提示缓存费用。

### model\_context\_window\_exceeded

Claude 因达到模型的上下文窗口限制而停止。这使您可以在不知道确切输入大小的情况下请求尽可能多的令牌。

<Note>
  此停止原因目前仅在 SDK 的 `beta` (csharp, go: `Beta`; java: `beta()`) 命名空间中有类型定义，因此以下示例调用 `client.beta.messages` (csharp, go: `client.Beta.Messages`; java: `client.beta().messages()`; php: `$client->beta->messages`) 并使用带 `Beta` 前缀的类型。在 Sonnet 4.5（已弃用）及更新的模型上，API 无需 beta 标头即可返回此值。对于更早的模型，请添加 `model-context-window-exceeded-2025-08-26` beta 标头以启用它。
</Note>

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 20000,
      "messages": [{"role": "user", "content": "Large input that uses most of context window..."}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 20000 \
    --message '{role: user, content: "Large input that uses most of context window..."}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 请求时设置最大令牌数，以尽可能获取更多内容
  response = client.beta.messages.create(
      model="claude-opus-5-5",
      max_tokens=20000,  # Python SDK requires streaming for max_tokens above ~21k
      messages=[
          {"role": "user", "content": "Large input that uses most of context window..."}
      ],
  )

  if response.stop_reason == "model_context_window_exceeded":
      # 响应在达到 max_tokens 之前已触及 context window（上下文窗口）限制
      print("Response reached model's context window limit")
      # 响应仍然有效，但受到了上下文窗口的限制
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 使用最大令牌数发起请求，以尽可能多地获取内容
  const response = await client.beta.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 20000,
    messages: [{ role: "user", content: "Large input that uses most of context window..." }]
  });

  if (response.stop_reason === "model_context_window_exceeded") {
    // 响应在达到 max_tokens 之前已触及 context window（上下文窗口）限制
    console.log("Response reached model's context window limit");
    // 响应仍然有效，但受到了上下文窗口的限制
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  // 使用最大令牌数发起请求，以获取尽可能多的内容
  var response = await client.Beta.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 20000,
      Messages = [new BetaUserMessageParam { Content = "Large input that uses most of context window..." }]
  });

  if (response.StopReason?.Value() == BetaStopReason.ModelContextWindowExceeded)
  {
      // 响应在达到 max_tokens 之前已触及 context window（上下文窗口）限制
      Console.WriteLine("Response reached model's context window limit");
      // 响应仍然有效，但受到了上下文窗口的限制
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  // 使用最大令牌数发起请求，以尽可能多地获取内容
  response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 20000,
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Large input that uses most of context window...")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == anthropic.BetaStopReasonModelContextWindowExceeded {
  	// 响应在达到 max_tokens 之前已触及 context window（上下文窗口）限制
  	fmt.Println("Response reached model's context window limit")
  	// 响应仍然有效，但受到了上下文窗口的限制
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  // 使用最大令牌数发起请求，以获取尽可能多的内容
  BetaMessage response = client.beta().messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(20000L)
          .addUserMessage("Large input that uses most of context window...")
          .build()
  );

  if (response.stopReason().map(BetaStopReason.MODEL_CONTEXT_WINDOW_EXCEEDED::equals).orElse(false)) {
      // 响应在达到 max_tokens 之前已触及 context window（上下文窗口）限制
      IO.println("Response reached model's context window limit");
      // 响应仍然有效，但受到了上下文窗口的限制
  }
  ```

  ```php PHP
  $client = new Client();

  // 使用最大令牌数发起请求，以尽可能多地获取输出
  $response = $client->beta->messages->create(
      maxTokens: 20000,
      messages: [['role' => 'user', 'content' => 'Large input that uses most of context window...']],
      model: 'claude-opus-5-5',
  );

  if ($response->stopReason === 'model_context_window_exceeded') {
      // 响应在达到 max_tokens 之前已触及上下文窗口限制
      echo 'Response reached model\'s context window limit', PHP_EOL;
      // 响应仍然有效，但受到了上下文窗口的限制
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 请求时设置最大令牌数，以尽可能多地获取输出
  response = client.beta.messages.create(
    model: "claude-opus-5-5",
    max_tokens: 20000,
    messages: [{ role: "user", content: "Large input that uses most of context window..." }]
  )

  if response.stop_reason == :model_context_window_exceeded
    # 响应在达到 max_tokens 之前已触及上下文窗口限制
    puts "Response reached model's context window limit"
    # 响应仍然有效，但受到了上下文窗口的限制
  end
  ```
</CodeGroup>

## 处理停止原因的最佳实践

### 始终检查 stop\_reason

养成在响应处理逻辑中检查 `stop_reason` 的习惯：

<CodeGroup exclude="shell">
  ```python Python
  def handle_response(response):
      match response.stop_reason:
          case "tool_use":
              return handle_tool_use(response)
          case "max_tokens":
              return handle_truncation(response)
          case "model_context_window_exceeded":
              return handle_context_limit(response)
          case "pause_turn":
              return handle_pause(response)
          case "refusal":
              return handle_refusal(response)
          case _:
              # 处理 end_turn 及其他情况
              return next(
                  (block.text for block in response.content if block.type == "text"),
                  "",
              )
  ```

  ```typescript TypeScript
  function handleResponse(response: Anthropic.Beta.BetaMessage): string {
    switch (response.stop_reason) {
      case "tool_use":
        return handleToolUse(response);
      case "max_tokens":
        return handleTruncation(response);
      case "model_context_window_exceeded":
        return handleContextLimit(response);
      case "pause_turn":
        return handlePause(response);
      case "refusal":
        return handleRefusal(response);
      default: {
        // 处理 end_turn 及其他情况
        const textBlock = response.content.find((block) => block.type === "text");
        return textBlock?.text ?? "";
      }
    }
  }
  ```

  ```csharp C#
  static string HandleResponse(BetaMessage response)
  {
      return response.StopReason?.Value() switch
      {
          BetaStopReason.ToolUse => HandleToolUse(response),
          BetaStopReason.MaxTokens => HandleTruncation(response),
          BetaStopReason.ModelContextWindowExceeded => HandleContextLimit(response),
          BetaStopReason.PauseTurn => HandlePause(response),
          BetaStopReason.Refusal => HandleRefusal(response),
          // 处理 end_turn 及其他情况
          _ => response.Content.Select(b => b.Value).OfType<BetaTextBlock>().FirstOrDefault()?.Text ?? "",
      };
  }
  ```

  ```go Go
  func handleResponse(response *anthropic.BetaMessage) string {
  	switch response.StopReason {
  	case anthropic.BetaStopReasonToolUse:
  		return handleToolUse(response)
  	case anthropic.BetaStopReasonMaxTokens:
  		return handleTruncation(response)
  	case anthropic.BetaStopReasonModelContextWindowExceeded:
  		return handleContextLimit(response)
  	case anthropic.BetaStopReasonPauseTurn:
  		return handlePause(response)
  	case anthropic.BetaStopReasonRefusal:
  		return handleRefusal(response)
  	default:
  		// 处理 end_turn 及其他情况
  		for _, block := range response.Content {
  			if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  				return textBlock.Text
  			}
  		}
  		return ""
  	}
  }
  ```

  ```java Java
  static String handleResponse(BetaMessage response) {
      BetaStopReason reason = response.stopReason().orElse(BetaStopReason.END_TURN);
      if (reason.equals(BetaStopReason.TOOL_USE)) {
          return handleToolUse(response);
      } else if (reason.equals(BetaStopReason.MAX_TOKENS)) {
          return handleTruncation(response);
      } else if (reason.equals(BetaStopReason.MODEL_CONTEXT_WINDOW_EXCEEDED)) {
          return handleContextLimit(response);
      } else if (reason.equals(BetaStopReason.PAUSE_TURN)) {
          return handlePause(response);
      } else if (reason.equals(BetaStopReason.REFUSAL)) {
          return handleRefusal(response);
      }
      // 处理 end_turn 及其他情况
      return response.content().stream()
          .filter(BetaContentBlock::isText)
          .findFirst()
          .map(block -> block.asText().text())
          .orElse("");
  }
  ```

  ```php PHP
  function handle_response($response): string
  {
      return match ($response->stopReason) {
          'tool_use' => handle_tool_use($response),
          'max_tokens' => handle_truncation($response),
          'model_context_window_exceeded' => handle_context_limit($response),
          'pause_turn' => handle_pause($response),
          'refusal' => handle_refusal($response),
          // 处理 end_turn 及其他情况
          default => array_find($response->content, static fn ($block): bool => $block->type === 'text')?->text ?? '',
      };
  }
  ```

  ```ruby Ruby
  def handle_response(response)
    case response.stop_reason
    when :tool_use then handle_tool_use(response)
    when :max_tokens then handle_truncation(response)
    when :model_context_window_exceeded then handle_context_limit(response)
    when :pause_turn then handle_pause(response)
    when :refusal then handle_refusal(response)
    else
      # 处理 end_turn 及其他情况
      response.content.find { it.type == :text }&.text
    end
  end
  ```
</CodeGroup>

### 妥善处理被截断的响应

当响应因令牌限制或上下文窗口而被截断时，请附加一条提示，让读者知道输出不完整。若要改为从响应中断处继续生成，请参阅[确保响应完整](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#ensuring-complete-responses)。

<CodeGroup exclude="shell">
  ```python Python
  def handle_truncated_response(response):
      text = next((block.text for block in response.content if block.type == "text"), "")
      if response.stop_reason in ["max_tokens", "model_context_window_exceeded"]:
          if response.stop_reason == "max_tokens":
              note = "[Response truncated due to max_tokens limit]"
          else:
              note = "[Response truncated due to context window limit]"
          return f"{text}\n\n{note}"
      return text
  ```

  ```typescript TypeScript
  function handleTruncatedResponse(response: Anthropic.Beta.BetaMessage): string {
    const textBlock = response.content.find((block) => block.type === "text");
    const text = textBlock?.text ?? "";

    if (
      response.stop_reason === "max_tokens" ||
      response.stop_reason === "model_context_window_exceeded"
    ) {
      const note =
        response.stop_reason === "max_tokens"
          ? "[Response truncated due to max_tokens limit]"
          : "[Response truncated due to context window limit]";
      return `${text}\n\n${note}`;
    }
    return text;
  }
  ```

  ```csharp C#
  static string HandleTruncatedResponse(BetaMessage response)
  {
      var text = response.Content.Select(b => b.Value).OfType<BetaTextBlock>().FirstOrDefault()?.Text ?? "";
      var reason = response.StopReason?.Value();

      if (reason is BetaStopReason.MaxTokens or BetaStopReason.ModelContextWindowExceeded)
      {
          var note = reason == BetaStopReason.MaxTokens
              ? "[Response truncated due to max_tokens limit]"
              : "[Response truncated due to context window limit]";
          return $"{text}\n\n{note}";
      }
      return text;
  }
  ```

  ```go Go
  func handleTruncatedResponse(response *anthropic.BetaMessage) string {
  	text := ""
  	for _, block := range response.Content {
  		if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  			text = textBlock.Text
  			break
  		}
  	}

  	if response.StopReason == anthropic.BetaStopReasonMaxTokens ||
  		response.StopReason == anthropic.BetaStopReasonModelContextWindowExceeded {
  		note := "[Response truncated due to context window limit]"
  		if response.StopReason == anthropic.BetaStopReasonMaxTokens {
  			note = "[Response truncated due to max_tokens limit]"
  		}
  		return text + "\n\n" + note
  	}
  	return text
  }
  ```

  ```java Java
  static String handleTruncatedResponse(BetaMessage response) {
      String text = response.content().stream()
          .filter(BetaContentBlock::isText)
          .findFirst()
          .map(block -> block.asText().text())
          .orElse("");
      BetaStopReason reason = response.stopReason().orElse(BetaStopReason.END_TURN);

      if (reason.equals(BetaStopReason.MAX_TOKENS)
              || reason.equals(BetaStopReason.MODEL_CONTEXT_WINDOW_EXCEEDED)) {
          String note = reason.equals(BetaStopReason.MAX_TOKENS)
              ? "[Response truncated due to max_tokens limit]"
              : "[Response truncated due to context window limit]";
          return text + "\n\n" + note;
      }
      return text;
  }
  ```

  ```php PHP
  function handle_truncated_response($response): string
  {
      $text = array_find($response->content, static fn ($block): bool => $block->type === 'text')?->text ?? '';

      if (in_array($response->stopReason, ['max_tokens', 'model_context_window_exceeded'], true)) {
          $note = $response->stopReason === 'max_tokens'
              ? '[Response truncated due to max_tokens limit]'
              : '[Response truncated due to context window limit]';
          return "{$text}\n\n{$note}";
      }
      return $text;
  }
  ```

  ```ruby Ruby
  def handle_truncated_response(response)
    text = response.content.find { it.type == :text }&.text

    if [:max_tokens, :model_context_window_exceeded].include?(response.stop_reason)
      note = if response.stop_reason == :max_tokens
        "[Response truncated due to max_tokens limit]"
      else
        "[Response truncated due to context window limit]"
      end
      return "#{text}\n\n#{note}"
    end
    text
  end
  ```
</CodeGroup>

### 为 pause\_turn 实现重试逻辑

当您使用[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)时，如果服务器端采样循环达到其迭代限制（默认为 10），API 可能会返回 `pause_turn`。处理方法是将暂停的响应原样添加到您的消息中，然后再次发送请求。如果响应包含 `container`，也请[将其 ID 发回](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#container-reuse)。否则，下一个请求将启动一个新的容器，该轮次中较早产生的文件将会丢失。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  client = anthropic.Anthropic()


  def handle_server_tool_conversation(client, user_query, tools, max_requests=5):
      messages = [{"role": "user", "content": user_query}]
      container_id = None

      for _ in range(max_requests):
          response = client.messages.create(
              model="claude-opus-5-5",
              max_tokens=4096,
              messages=messages,
              tools=tools,
              container=container_id,
          )

          if response.stop_reason != "pause_turn":
              # 未暂停：返回响应
              return response

          # pause_turn：将暂停的响应作为 assistant 消息添加，然后继续
          messages.append({"role": "assistant", "content": response.content})

          # 如果工具在容器中运行，请回传其 ID 以保留其文件
          if response.container:
              container_id = response.container.id

      # 请求 max_requests 次后仍暂停：返回最后一个响应
      return response


  response = handle_server_tool_conversation(
      client,
      "Use the code execution tool to find the 1,000th prime number.",
      [{"type": "code_execution_20250825", "name": "code_execution"}],
  )
  print(response.stop_reason)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  async function handleServerToolConversation(
    client: Anthropic,
    userQuery: string,
    tools: Anthropic.ToolUnion[],
    maxRequests = 5
  ): Promise<Anthropic.Message> {
    const messages: Anthropic.MessageParam[] = [{ role: "user", content: userQuery }];
    let containerId: string | null = null;
    let response: Anthropic.Message;
    let requestCount = 0;

    do {
      response = await client.messages.create({
        model: "claude-opus-5-5",
        max_tokens: 4096,
        messages,
        tools,
        container: containerId
      });

      if (response.stop_reason !== "pause_turn") {
        // 未暂停：返回响应
        return response;
      }

      // pause_turn：将暂停的响应作为 assistant 消息添加，然后继续
      messages.push({ role: "assistant", content: response.content });

      // 如果工具在容器中运行，请回传其 ID 以保留其文件
      if (response.container) {
        containerId = response.container.id;
      }
    } while (++requestCount < maxRequests);

    // 请求 maxRequests 次后仍暂停：返回最后一个响应
    return response;
  }

  const response = await handleServerToolConversation(
    client,
    "Use the code execution tool to find the 1,000th prime number.",
    [{ type: "code_execution_20250825", name: "code_execution" }]
  );
  console.log(response.stop_reason);
  ```

  ```csharp C#
  // 服务器工具的一个轮次可能运行数分钟，超过 SDK 的默认超时时间，
  // 该超时由 MaxTokens 决定（4096 时约为 2 分钟）。
  AnthropicClient client = new() { Timeout = TimeSpan.FromMinutes(10) };

  static async Task<Message> HandleServerToolConversation(
      AnthropicClient client,
      string userQuery,
      List<ToolUnion> tools,
      int maxRequests = 5)
  {
      List<MessageParam> messages = [new UserMessageParam { Content = userQuery }];
      MessageCreateParamsContainer? container = null;
      Message response;
      var requestCount = 0;

      do
      {
          response = await client.Messages.Create(new MessageCreateParams
          {
              Model = Model.ClaudeOpus5_5,
              MaxTokens = 4096,
              Messages = messages,
              Tools = tools,
              Container = container
          });

          if (response.StopReason != StopReason.PauseTurn)
          {
              // 未暂停：返回响应
              return response;
          }

          // pause_turn：将暂停的响应作为 assistant 消息添加，然后继续
          // 请求块和响应块是不同的类型：原样复制每个块的 JSON
          messages.Add(new AssistantMessageParam
          {
              Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList()
          });

          // 如果工具在容器中运行，请回传其 ID 以保留其文件
          if (response.Container is not null)
          {
              container = response.Container.ID;
          }
      } while (++requestCount < maxRequests);

      // 发出 maxRequests 次请求后仍处于暂停状态：返回最后一个响应
      return response;
  }

  var response = await HandleServerToolConversation(
      client,
      "Use the code execution tool to find the 1,000th prime number.",
      [new CodeExecutionTool20250825()]);
  Console.WriteLine(response.StopReason?.Raw());
  ```

  ```go Go
  func handleServerToolConversation(
  	client anthropic.Client,
  	userQuery string,
  	tools []anthropic.ToolUnionParam,
  	maxRequests int,
  ) (*anthropic.Message, error) {
  	messages := []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock(userQuery)),
  	}
  	var container anthropic.MessageCreateParamsContainerUnion
  	var response *anthropic.Message
  	var err error

  	for range maxRequests {
  		response, err = client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 4096,
  			Messages:  messages,
  			Tools:     tools,
  			Container: container,
  		})
  		if err != nil {
  			return nil, err
  		}

  		if response.StopReason != anthropic.StopReasonPauseTurn {
  			// 未暂停：返回响应
  			return response, nil
  		}

  		// pause_turn：将暂停的响应作为 assistant 消息添加，然后继续
  		messages = append(messages, response.ToParam())

  		// 如果工具在容器中运行，请回传其 ID 以保留其文件
  		if response.Container.ID != "" {
  			container.OfString = anthropic.String(response.Container.ID)
  		}
  	}

  	// 在 maxRequests 次请求后仍暂停：返回最后一个响应
  	return response, nil
  }

  func main() {
  	client := anthropic.NewClient()

  	response, err := handleServerToolConversation(
  		client,
  		"Use the code execution tool to find the 1,000th prime number.",
  		[]anthropic.ToolUnionParam{
  			{OfCodeExecutionTool20250825: &anthropic.CodeExecutionTool20250825Param{}},
  		},
  		5,
  	)
  	if err != nil {
  		log.Fatal(err)
  	}
  	fmt.Println(response.StopReason)
  }
  ```

  ```java Java
  Message handleServerToolConversation(
      AnthropicClient client,
      String userQuery,
      List<ToolUnion> tools,
      int maxRequests
  ) {
      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .tools(tools)
          .addUserMessage(userQuery);
      Message response = null;

      for (int request = 0; request < maxRequests; request++) {
          response = client.messages().create(params.build());

          boolean paused = response.stopReason().filter(StopReason.PAUSE_TURN::equals).isPresent();
          if (!paused) {
              // 未暂停：返回响应
              return response;
          }

          // pause_turn：将暂停的响应作为 assistant 消息添加，然后继续
          params.addMessage(response);

          // 如果工具在容器中运行，请传回其 ID 以保留其文件
          response.container().ifPresent(container -> params.container(container.id()));
      }

      // 在 maxRequests 次请求后仍处于暂停状态：返回最后一个响应
      return response;
  }

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Message response = handleServerToolConversation(
          client,
          "Use the code execution tool to find the 1,000th prime number.",
          List.of(ToolUnion.ofCodeExecutionTool20250825(CodeExecutionTool20250825.builder().build())),
          5
      );
      response.stopReason().ifPresent(IO::println);
  }
  ```

  ```php PHP
  $client = new Client();

  function handle_server_tool_conversation(
      Client $client,
      string $userQuery,
      array $tools,
      int $maxRequests = 5,
  ): Message {
      $messages = [['role' => 'user', 'content' => $userQuery]];
      $containerId = null;
      $requestCount = 0;

      do {
          $response = $client->messages->create(
              maxTokens: 4096,
              messages: $messages,
              model: Model::CLAUDE_OPUS_5_5,
              tools: $tools,
              container: $containerId,
          );

          if ($response->stopReason !== StopReason::PAUSE_TURN->value) {
              // 未暂停：返回响应
              return $response;
          }

          // pause_turn：将暂停的响应作为 assistant 消息添加，然后继续
          $messages[] = ['role' => 'assistant', 'content' => $response->content];

          // 如果工具在容器中运行，请传回其 ID，以便保留其文件
          $containerId = $response->container->id ?? $containerId;
      } while (++$requestCount < $maxRequests);

      // 请求 $maxRequests 次后仍处于暂停状态：返回最后一个响应
      return $response;
  }

  $response = handle_server_tool_conversation(
      $client,
      'Use the code execution tool to find the 1,000th prime number.',
      [new CodeExecutionTool20250825()],
  );
  echo $response->stopReason, PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  def handle_server_tool_conversation(client, user_query, tools, max_requests: 5)
    messages = [{ role: "user", content: user_query }]
    container_id = nil
    response = nil

    max_requests.times do
      response = client.messages.create(
        model: Anthropic::Model::CLAUDE_OPUS_5_5,
        max_tokens: 4096,
        messages:,
        tools:,
        container: container_id
      )

      # 未暂停：返回响应
      return response unless response.stop_reason == :pause_turn

      # pause_turn：将暂停的响应作为 assistant 消息添加，然后继续
      messages << { role: "assistant", content: response.content }

      # 如果工具在容器中运行，请发回其 ID 以保留其文件
      container_id = response.container&.id || container_id
    end

    # 请求 max_requests 次后仍处于暂停状态：返回最后一个响应
    response
  end

  response = handle_server_tool_conversation(
    client,
    "Use the code execution tool to find the 1,000th prime number.",
    [Anthropic::CodeExecutionTool20250825.new]
  )
  puts response.stop_reason
  ```
</CodeGroup>

该函数返回最后一个响应，其中仅包含该轮次的最后一部分。若要显示整个轮次或继续对话，请同时保留那些暂停的响应。

## 停止原因与错误

区分 `stop_reason` 值和实际错误非常重要：

### 停止原因（成功的响应）

* 是响应正文的一部分
* 表示生成为何正常停止
* 响应包含有效内容

### 错误（失败的请求）

* HTTP 状态码为 4xx 或 5xx
* 表示请求处理失败
* 响应包含错误详情

<CodeGroup>
  ```bash cURL
  # 使用 --fail-with-body 时，cURL 遇到 HTTP 错误会以非零状态退出；请检查
  # $? 以判断错误，成功响应则检查 stop_reason。
  curl --fail-with-body https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello!"}]
    }'
  ```

  ```bash CLI
  # API 出错时 CLI 以非零状态码退出；成功时会返回 stop_reason。
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello!"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  try:
      response = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[{"role": "user", "content": "Hello!"}],
      )

      # 处理带有 stop_reason 的成功响应
      if response.stop_reason == "max_tokens":
          print("Response was truncated")

  except anthropic.APIStatusError as e:
      # 处理实际错误
      match e.status_code:
          case 429:
              print("Rate limit exceeded")
          case 500:
              print("Server error")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  try {
    const response = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    });

    // 处理带有 stop_reason 的成功响应
    if (response.stop_reason === "max_tokens") {
      console.log("Response was truncated");
    }
  } catch (err) {
    // 处理实际错误
    if (err instanceof Anthropic.APIError) {
      switch (err.status) {
        case 429:
          console.log("Rate limit exceeded");
          break;
        case 500:
          console.log("Server error");
          break;
      }
    } else {
      throw err;
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  try
  {
      var response = await client.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Hello!" }]
      });

      // 处理带有 stop_reason 的成功响应
      if (response.StopReason == "max_tokens")
      {
          Console.WriteLine("Response was truncated");
      }
  }
  catch (AnthropicRateLimitException)
  {
      // 处理实际错误
      Console.WriteLine("Rate limit exceeded");
  }
  catch (Anthropic5xxException)
  {
      Console.WriteLine("Server error");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello!")),
  	},
  })
  if err != nil {
  	// 处理实际错误
  	var apiErr *anthropic.Error
  	if errors.As(err, &apiErr) {
  		switch apiErr.StatusCode {
  		case 429:
  			fmt.Println("Rate limit exceeded")
  		case 500:
  			fmt.Println("Server error")
  		}
  	}
  	log.Fatal(err)
  }

  // 处理带有 stop_reason 的成功响应
  if response.StopReason == "max_tokens" {
  	fmt.Println("Response was truncated")
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  try {
      Message response = client.messages().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024L)
              .addUserMessage("Hello!")
              .build()
      );

      // 处理带有 stop_reason 的成功响应
      if (response.stopReason().map(StopReason.MAX_TOKENS::equals).orElse(false)) {
          IO.println("Response was truncated");
      }
  } catch (RateLimitException e) {
      // 处理实际错误
      IO.println("Rate limit exceeded");
  } catch (AnthropicServiceException e) {
      if (e.statusCode() == 500) {
          IO.println("Server error");
      }
  }
  ```

  ```php PHP
  $client = new Client();

  try {
      $response = $client->messages->create(
          maxTokens: 1024,
          messages: [['role' => 'user', 'content' => 'Hello!']],
          model: Model::CLAUDE_OPUS_5_5,
      );

      // 处理带有 stop_reason 的成功响应
      if ($response->stopReason === 'max_tokens') {
          echo 'Response was truncated', PHP_EOL;
      }
  } catch (RateLimitException $e) {
      // 处理实际错误
      echo 'Rate limit exceeded', PHP_EOL;
  } catch (InternalServerException $e) {
      echo 'Server error', PHP_EOL;
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  begin
    response = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    )

    # 处理带有 stop_reason 的成功响应
    if response.stop_reason == :max_tokens
      puts "Response was truncated"
    end
  rescue Anthropic::Errors::RateLimitError
    # 处理实际错误
    puts "Rate limit exceeded"
  rescue Anthropic::Errors::APIStatusError => e
    puts "Server error" if e.status == 500
  end
  ```
</CodeGroup>

## 流式传输注意事项

使用 streaming（流式传输）时，`stop_reason`：

* 在初始的 `message_start` 事件中为 `null`
* 在 `message_delta` 事件中提供
* 不在任何其他事件中提供

<CodeGroup>
  ```bash cURL
  # SSE 流中的 message_delta 事件携带 stop_reason。
  curl -N https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "stream": true,
      "messages": [{"role": "user", "content": "Hello!"}]
    }'
  ```

  ```bash CLI
  # stop_reason 出现在 message_delta 事件中。
  ant messages create --stream \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello!"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  with client.messages.stream(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello!"}],
  ) as stream:
      for event in stream:
          if event.type == "message_delta":
              stop_reason = event.delta.stop_reason
              if stop_reason:
                  print(f"Stream ended with: {stop_reason}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const stream = client.messages.stream({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello!" }]
  });

  for await (const event of stream) {
    if (event.type === "message_delta" && event.delta.stop_reason) {
      console.log(`Stream ended with: ${event.delta.stop_reason}`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello!" }]
  };

  await foreach (var streamEvent in client.Messages.CreateStreaming(parameters))
  {
      switch (streamEvent.Value)
      {
          case RawMessageDeltaEvent deltaEvent when deltaEvent.Delta.StopReason is not null:
              Console.WriteLine($"Stream ended with: {deltaEvent.Delta.StopReason}");
              break;
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello!")),
  	},
  })

  // 将事件累积为最终的 Message，其中包含 stop_reason。
  message := anthropic.Message{}
  for stream.Next() {
  	if err := message.Accumulate(stream.Current()); err != nil {
  		log.Fatal(err)
  	}
  }
  if err := stream.Err(); err != nil {
  	log.Fatal(err)
  }

  if message.StopReason != "" {
  	fmt.Printf("Stream ended with: %s\n", message.StopReason)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .addUserMessage("Hello!")
      .build();

  // 将事件累积为最终的 Message，其中包含 stop_reason。
  MessageAccumulator accumulator = MessageAccumulator.create();
  try (StreamResponse<RawMessageStreamEvent> streamResponse =
          client.messages().createStreaming(params)) {
      streamResponse.stream().forEach(accumulator::accumulate);
  }

  accumulator.message().stopReason().ifPresent(stopReason ->
      IO.println("Stream ended with: " + stopReason)
  );
  ```

  ```php PHP
  $client = new Client();

  $stream = $client->messages->createStream(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello!']],
      model: Model::CLAUDE_OPUS_5_5,
  );

  foreach ($stream as $event) {
      if ($event instanceof \Anthropic\Messages\RawMessageDeltaEvent && $event->delta->stopReason !== null) {
          echo "Stream ended with: {$event->delta->stopReason}", PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  stream = client.messages.stream(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello!" }]
  )

  stream.each do |event|
    next unless event.type == :message_delta
    stop_reason = event.delta.stop_reason
    puts "Stream ended with: #{stop_reason}" if stop_reason
  end
  ```
</CodeGroup>

## 常见模式

### 处理工具使用工作流

<Tip>
  **使用工具运行器更简单：** 以下示例展示了手动工具处理。对于大多数用例，[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)能以少得多的代码自动处理工具执行。
</Tip>

<CodeGroup exclude="shell">
  ```python Python
  def complete_tool_workflow(client, user_query, tools):
      messages = [{"role": "user", "content": user_query}]

      while True:
          response = client.messages.create(
              model="claude-opus-5-5",
              max_tokens=1024,
              messages=messages,
              tools=tools,
          )

          if response.stop_reason == "tool_use":
              # 运行工具并继续
              tool_results = execute_tools(response.content)
              messages.append({"role": "assistant", "content": response.content})
              messages.append({"role": "user", "content": tool_results})
          else:
              # 最终响应
              return response
  ```

  ```typescript TypeScript
  async function completeToolWorkflow(
    client: Anthropic,
    userQuery: string,
    tools: Anthropic.ToolUnion[]
  ): Promise<Anthropic.Message> {
    const messages: Anthropic.MessageParam[] = [{ role: "user", content: userQuery }];

    while (true) {
      const response = await client.messages.create({
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages,
        tools
      });

      if (response.stop_reason === "tool_use") {
        // 运行工具并继续
        const toolResults = executeTools(response.content);
        messages.push({ role: "assistant", content: response.content });
        messages.push({ role: "user", content: toolResults });
      } else {
        // 最终响应
        return response;
      }
    }
  }
  ```

  ```csharp C#
  static async Task<Message> CompleteToolWorkflow(
      AnthropicClient client,
      string userQuery,
      List<ToolUnion> tools)
  {
      List<MessageParam> messages = [new UserMessageParam { Content = userQuery }];

      while (true)
      {
          var response = await client.Messages.Create(new MessageCreateParams
          {
              Model = Model.ClaudeOpus5_5,
              MaxTokens = 1024,
              Messages = messages,
              Tools = tools
          });

          if (response.StopReason == "tool_use")
          {
              // 运行工具并继续
              var toolResults = ExecuteTools(response.Content);
              messages.Add(new AssistantMessageParam
              {
                  Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList()
              });
              messages.Add(new UserMessageParam { Content = toolResults });
          }
          else
          {
              // 最终响应
              return response;
          }
      }
  }
  ```

  ```go Go
  func completeToolWorkflow(
  	client anthropic.Client,
  	userQuery string,
  	tools []anthropic.ToolUnionParam,
  ) (*anthropic.Message, error) {
  	messages := []anthropic.MessageParam{anthropic.NewUserMessage(anthropic.NewTextBlock(userQuery))}

  	for {
  		response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 1024,
  			Messages:  messages,
  			Tools:     tools,
  		})
  		if err != nil {
  			return nil, err
  		}

  		if response.StopReason != "tool_use" {
  			// 最终响应
  			return response, nil
  		}

  		// 运行工具并继续
  		toolResults := executeTools(response.Content)
  		var contentParams []anthropic.ContentBlockParamUnion
  		for _, block := range response.Content {
  			contentParams = append(contentParams, block.ToParam())
  		}
  		messages = append(messages, anthropic.NewAssistantMessage(contentParams...))
  		messages = append(messages, anthropic.NewUserMessage(toolResults...))
  	}
  }
  ```

  ```java Java
  static Message completeToolWorkflow(
      AnthropicClient client,
      String userQuery,
      List<Tool> tools
  ) {
      List<MessageParam> messages = new ArrayList<>();
      messages.add(MessageParam.ofUser(userQuery));

      while (true) {
          MessageCreateParams.Builder params = MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024L)
              .messages(messages);
          tools.forEach(params::addTool);

          Message response = client.messages().create(params.build());

          if (!response.stopReason().map(StopReason.TOOL_USE::equals).orElse(false)) {
              // 最终响应
              return response;
          }

          // 运行工具并继续
          List<ToolResultBlockParam> toolResults = executeTools(response.content());
          messages.add(response.toParam());
          messages.add(MessageParam.ofUserOfContentBlockParams(
              toolResults.stream().map(ContentBlockParam::ofToolResult).toList()));
      }
  }
  ```

  ```php PHP
  function complete_tool_workflow(Client $client, string $userQuery, array $tools)
  {
      $messages = [['role' => 'user', 'content' => $userQuery]];

      while (true) {
          $response = $client->messages->create(
              maxTokens: 1024,
              messages: $messages,
              model: 'claude-opus-5-5',
              tools: $tools,
          );

          if ($response->stopReason !== 'tool_use') {
              // 最终响应
              return $response;
          }

          // 运行工具并继续
          $toolResults = execute_tools($response->content);
          $messages[] = ['role' => 'assistant', 'content' => $response->content];
          $messages[] = ['role' => 'user', 'content' => $toolResults];
      }
  }
  ```

  ```ruby Ruby
  def complete_tool_workflow(client, user_query, tools)
    messages = [{ role: "user", content: user_query }]

    loop do
      response = client.messages.create(
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages: messages,
        tools: tools
      )

      # 最终响应
      return response unless response.stop_reason == :tool_use

      # 运行工具并继续
      tool_results = execute_tools(response.content)
      messages << { role: "assistant", content: response.content }
      messages << { role: "user", content: tool_results }
    end
  end
  ```
</CodeGroup>

### 确保响应完整

<CodeGroup exclude="shell">
  ```python Python
  def get_complete_response(client, prompt, max_attempts=3):
      messages = [{"role": "user", "content": prompt}]
      full_response = ""

      for _ in range(max_attempts):
          response = client.messages.create(
              model="claude-opus-5-5", messages=messages, max_tokens=4096
          )

          full_response += next(
              (block.text for block in response.content if block.type == "text"), ""
          )

          if response.stop_reason != "max_tokens":
              break

          # 从中断处继续
          messages = [
              {"role": "user", "content": prompt},
              {"role": "assistant", "content": full_response},
              {"role": "user", "content": "Please continue from where you left off."},
          ]

      return full_response
  ```

  ```typescript TypeScript
  async function getCompleteResponse(
    client: Anthropic,
    prompt: string,
    maxAttempts = 3
  ): Promise<string> {
    let messages: Anthropic.MessageParam[] = [{ role: "user", content: prompt }];
    let fullResponse = "";

    for (let i = 0; i < maxAttempts; i++) {
      const response = await client.messages.create({
        model: "claude-opus-5-5",
        max_tokens: 4096,
        messages
      });

      const textBlock = response.content.find((block) => block.type === "text");
      fullResponse += textBlock?.text ?? "";

      if (response.stop_reason !== "max_tokens") {
        break;
      }

      // 从上次中断处继续
      messages = [
        { role: "user", content: prompt },
        { role: "assistant", content: fullResponse },
        { role: "user", content: "Please continue from where you left off." }
      ];
    }

    return fullResponse;
  }
  ```

  ```csharp C#
  static async Task<string> GetCompleteResponse(AnthropicClient client, string prompt, int maxAttempts = 3)
  {
      List<MessageParam> messages = [new UserMessageParam { Content = prompt }];
      var fullResponse = "";

      for (var i = 0; i < maxAttempts; i++)
      {
          var response = await client.Messages.Create(new MessageCreateParams
          {
              Model = Model.ClaudeOpus5_5,
              MaxTokens = 4096,
              Messages = messages
          });

          foreach (var block in response.Content)
          {
              if (block.TryPickText(out var textBlock))
              {
                  fullResponse += textBlock.Text;
                  break;
              }
          }

          if (response.StopReason != "max_tokens")
          {
              break;
          }

          // 从上次中断处继续
          messages =
          [
              new UserMessageParam { Content = prompt },
              new AssistantMessageParam { Content = fullResponse },
              new UserMessageParam { Content = "Please continue from where you left off." }
          ];
      }

      return fullResponse;
  }
  ```

  ```go Go
  func getCompleteResponse(client anthropic.Client, prompt string, maxAttempts int) (string, error) {
  	messages := []anthropic.MessageParam{anthropic.NewUserMessage(anthropic.NewTextBlock(prompt))}
  	fullResponse := ""

  	for range maxAttempts {
  		response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 4096,
  			Messages:  messages,
  		})
  		if err != nil {
  			return "", err
  		}

  		for _, block := range response.Content {
  			if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  				fullResponse += textBlock.Text
  				break
  			}
  		}

  		if response.StopReason != "max_tokens" {
  			break
  		}

  		// 从中断处继续
  		messages = []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock(prompt)),
  			anthropic.NewAssistantMessage(anthropic.NewTextBlock(fullResponse)),
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Please continue from where you left off.")),
  		}
  	}

  	return fullResponse, nil
  }
  ```

  ```java Java
  static String getCompleteResponse(AnthropicClient client, String prompt, int maxAttempts) {
      List<MessageParam> messages = List.of(MessageParam.ofUser(prompt));
      StringBuilder fullResponse = new StringBuilder();

      for (int i = 0; i < maxAttempts; i++) {
          Message response = client.messages().create(
              MessageCreateParams.builder()
                  .model(Model.CLAUDE_OPUS_5_5)
                  .maxTokens(4096L)
                  .messages(messages)
                  .build()
          );

          response.content().stream()
              .filter(ContentBlock::isText)
              .findFirst()
              .ifPresent(block -> fullResponse.append(block.asText().text()));

          if (!response.stopReason().map(StopReason.MAX_TOKENS::equals).orElse(false)) {
              break;
          }

          // 从上次中断处继续
          messages = List.of(
              MessageParam.ofUser(prompt),
              MessageParam.ofAssistant(fullResponse.toString()),
              MessageParam.ofUser("Please continue from where you left off.")
          );
      }

      return fullResponse.toString();
  }
  ```

  ```php PHP
  function get_complete_response(Client $client, string $prompt, int $maxAttempts = 3): string
  {
      $messages = [['role' => 'user', 'content' => $prompt]];
      $fullResponse = '';

      for ($i = 0; $i < $maxAttempts; $i++) {
          $response = $client->messages->create(
              maxTokens: 4096,
              messages: $messages,
              model: 'claude-opus-5-5',
          );

          $fullResponse .= array_find($response->content, static fn ($block): bool => $block->type === 'text')?->text ?? '';

          if ($response->stopReason !== 'max_tokens') {
              break;
          }

          // 从中断处继续
          $messages = [
              ['role' => 'user', 'content' => $prompt],
              ['role' => 'assistant', 'content' => $fullResponse],
              ['role' => 'user', 'content' => 'Please continue from where you left off.'],
          ];
      }

      return $fullResponse;
  }
  ```

  ```ruby Ruby
  def get_complete_response(client, prompt, max_attempts: 3)
    messages = [{ role: "user", content: prompt }]
    full_response = +""

    max_attempts.times do
      response = client.messages.create(
        model: "claude-opus-5-5",
        max_tokens: 4096,
        messages: messages
      )

      full_response << response.content.find { it.type == :text }&.text.to_s

      break unless response.stop_reason == :max_tokens

      # 从中断处继续
      messages = [
        { role: "user", content: prompt },
        { role: "assistant", content: full_response },
        { role: "user", content: "Please continue from where you left off." }
      ]
    end

    full_response
  end
  ```
</CodeGroup>

### 在不知道输入大小的情况下获取最大令牌数

借助 `model_context_window_exceeded` 停止原因，您可以在不计算输入大小的情况下请求尽可能多的令牌：

<CodeGroup exclude="shell">
  ```python Python
  def get_max_possible_tokens(client, prompt):
      """
      Get as many tokens as possible within the model's context window
      without needing to calculate input token count
      """
      response = client.beta.messages.create(
          model="claude-opus-5-5",
          messages=[{"role": "user", "content": prompt}],
          max_tokens=20000,  # Python SDK requires streaming for max_tokens above ~21k
      )

      match response.stop_reason:
          case "model_context_window_exceeded":
              # 已获得输入大小所允许的最大令牌数
              print(
                  f"Generated {response.usage.output_tokens} tokens (context limit reached)"
              )
          case "max_tokens":
              # 恰好获得了请求的令牌数
              print(
                  f"Generated {response.usage.output_tokens} tokens (max_tokens reached)"
              )
          case _:
              # 自然完成
              print(
                  f"Generated {response.usage.output_tokens} tokens (natural completion)"
              )

      return next((block.text for block in response.content if block.type == "text"), "")
  ```

  ```typescript TypeScript
  async function getMaxPossibleTokens(client: Anthropic, prompt: string): Promise<string> {
    const response = await client.beta.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 20000,
      messages: [{ role: "user", content: prompt }]
    });

    const tokens = response.usage.output_tokens;
    switch (response.stop_reason) {
      case "model_context_window_exceeded":
        // 已获得输入大小所允许的最大令牌数
        console.log(`Generated ${tokens} tokens (context limit reached)`);
        break;
      case "max_tokens":
        // 恰好获得了所请求的令牌数
        console.log(`Generated ${tokens} tokens (max_tokens reached)`);
        break;
      default:
        // 自然完成
        console.log(`Generated ${tokens} tokens (natural completion)`);
    }

    const textBlock = response.content.find((block) => block.type === "text");
    return textBlock?.text ?? "";
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Messages;
  using Model = Anthropic.Models.Messages.Model;

  static async Task<string> GetMaxPossibleTokens(AnthropicClient client, string prompt)
  {
      var response = await client.Beta.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 20000,
          Messages = [new BetaUserMessageParam { Content = prompt }]
      });

      var tokens = response.Usage.OutputTokens;
      var reason = response.StopReason?.Value();
      if (reason == BetaStopReason.ModelContextWindowExceeded)
      {
          // 已获得输入大小所允许的最大令牌数
          Console.WriteLine($"Generated {tokens} tokens (context limit reached)");
      }
      else if (reason == BetaStopReason.MaxTokens)
      {
          // 已获得恰好等于请求数量的令牌
          Console.WriteLine($"Generated {tokens} tokens (max_tokens reached)");
      }
      else
      {
          // 自然完成
          Console.WriteLine($"Generated {tokens} tokens (natural completion)");
      }

      return response.Content.Select(b => b.Value).OfType<BetaTextBlock>().FirstOrDefault()?.Text ?? "";
  }
  ```

  ```go Go
  func getMaxPossibleTokens(client anthropic.Client, prompt string) (string, error) {
  	response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 20000,
  		Messages: []anthropic.BetaMessageParam{
  			anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock(prompt)),
  		},
  	})
  	if err != nil {
  		return "", err
  	}

  	tokens := response.Usage.OutputTokens
  	switch response.StopReason {
  	case anthropic.BetaStopReasonModelContextWindowExceeded:
  		// 已获得输入大小所允许的最大令牌数
  		fmt.Printf("Generated %d tokens (context limit reached)\n", tokens)
  	case anthropic.BetaStopReasonMaxTokens:
  		// 恰好获得了所请求的令牌数
  		fmt.Printf("Generated %d tokens (max_tokens reached)\n", tokens)
  	default:
  		// 自然完成
  		fmt.Printf("Generated %d tokens (natural completion)\n", tokens)
  	}

  	for _, block := range response.Content {
  		if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  			return textBlock.Text, nil
  		}
  	}
  	return "", nil
  }
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaContentBlock;
  import com.anthropic.models.beta.messages.BetaMessage;
  import com.anthropic.models.beta.messages.BetaStopReason;
  import com.anthropic.models.beta.messages.MessageCreateParams;

  static String getMaxPossibleTokens(AnthropicClient client, String prompt) {
      BetaMessage response = client.beta().messages().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(20000L)
              .addUserMessage(prompt)
              .build()
      );

      long tokens = response.usage().outputTokens();
      BetaStopReason reason = response.stopReason().orElse(BetaStopReason.END_TURN);
      if (reason.equals(BetaStopReason.MODEL_CONTEXT_WINDOW_EXCEEDED)) {
          // 已获得输入大小所允许的最大令牌数
          IO.println("Generated " + tokens + " tokens (context limit reached)");
      } else if (reason.equals(BetaStopReason.MAX_TOKENS)) {
          // 已获得恰好等于请求数量的令牌
          IO.println("Generated " + tokens + " tokens (max_tokens reached)");
      } else {
          // 自然完成
          IO.println("Generated " + tokens + " tokens (natural completion)");
      }

      return response.content().stream()
          .filter(BetaContentBlock::isText)
          .findFirst()
          .map(block -> block.asText().text())
          .orElse("");
  }
  ```

  ```php PHP
  function get_max_possible_tokens(Client $client, string $prompt): string
  {
      $response = $client->beta->messages->create(
          maxTokens: 20000,
          messages: [['role' => 'user', 'content' => $prompt]],
          model: 'claude-opus-5-5',
      );

      $tokens = $response->usage->outputTokens;
      echo match ($response->stopReason) {
          // 已获得输入大小所允许的最大令牌数
          'model_context_window_exceeded' => "Generated {$tokens} tokens (context limit reached)",
          // 恰好获得了所请求的令牌数
          'max_tokens' => "Generated {$tokens} tokens (max_tokens reached)",
          // 自然完成
          default => "Generated {$tokens} tokens (natural completion)",
      }, PHP_EOL;

      return array_find($response->content, static fn ($block): bool => $block->type === 'text')?->text ?? '';
  }
  ```

  ```ruby Ruby
  def get_max_possible_tokens(client, prompt)
    response = client.beta.messages.create(
      model: "claude-opus-5-5",
      max_tokens: 20000,
      messages: [{ role: "user", content: prompt }]
    )

    tokens = response.usage.output_tokens
    case response.stop_reason
    when :model_context_window_exceeded
      # 已获得输入大小所允许的最大令牌数
      puts "Generated #{tokens} tokens (context limit reached)"
    when :max_tokens
      # 恰好获得了请求的令牌数
      puts "Generated #{tokens} tokens (max_tokens reached)"
    else
      # 自然完成
      puts "Generated #{tokens} tokens (natural completion)"
    end

    response.content.find { it.type == :text }.text
  end
  ```
</CodeGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="拒绝与回退" icon="arrows-clockwise" href="https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback">
    在服务器端或您的客户端中，在回退模型上重试被拒绝的请求。
  </Card>

  <Card title="工具运行器（SDK）" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner">
    让 SDK 为您管理 `tool_use` 循环、结果格式化和重试。
  </Card>

  <Card title="流式传输消息" icon="lightning" href="https://platform.claude.com/docs/zh-CN/build-with-claude/streaming">
    在流式传输时从 `message_delta` 事件中读取 `stop_reason`。
  </Card>

  <Card title="错误" icon="info" href="https://platform.claude.com/docs/zh-CN/api/errors">
    处理 4xx 和 5xx HTTP 错误，它们与停止原因不同。
  </Card>
</CardGroup>
