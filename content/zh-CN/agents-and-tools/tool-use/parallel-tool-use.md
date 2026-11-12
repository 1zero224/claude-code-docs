---
title: 并行工具使用
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use
description: 启用、格式化和禁用并行工具调用，并提供消息历史指导和故障排除。
featureMetadata:
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
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

默认情况下，Claude 可能会在单个响应中调用多个工具。本页介绍如何运行这些调用、如何格式化消息历史以保持并行性正常工作，以及在需要时如何禁用"parallel tool use"（并行工具使用）。有关单次调用流程，请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。

## 执行语义

当 Claude 调用工具时，响应的 `stop_reason` 为 `tool_use`，并且可以在单个助手轮次中包含多个 `tool_use` 块。如何运行这些调用由您决定。API 不规定执行顺序：您可以并发运行这些调用（`Promise.all`、`asyncio.gather`），按它们出现的顺序依次运行，或者采用任何适合您工具的组合方式。

请根据您的工具的功能选择策略。独立的只读操作通常可以安全地并行运行，以降低"latency"（延迟）。具有副作用、共享状态或顺序要求的工具可能更适合依次运行。

无论您使用哪种策略，都要为每个 `tool_use` 块返回一个 `tool_result`，并将它们全部放在下一条用户消息中。使用 `tool_use_id` 将每个结果与其调用匹配，并将每个 `tool_result` 块放在该消息中任何文本内容之前。有关完整的格式规则，请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。如果您选择不运行某个特定调用（例如，因为您依次运行了该批次而较早的调用失败了），仍然要为其返回一个带有 `is_error: true` 和简短说明的 `tool_result`。

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_02",
  "is_error": true,
  "content": "Not executed: the preceding write_file call failed."
}
```

[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)和[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)的要求更为严格。当 Claude 在一个轮次中返回多个其成员工具调用（批量操作）时，请按它们出现的顺序依次运行，并在第一次失败时停止；每个工具都定义了针对您跳过的调用应返回的确切文本。

## 测试并行工具调用

<Note>
  **大多数应用程序请使用工具运行器：** SDK 的[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)会处理包含多个工具调用的响应并为您格式化结果，因此您无需自己编写这些处理逻辑。当您需要直接控制调用的运行方式（例如自定义批处理、排序或错误处理）时，请使用本页的手动模式。
</Note>

以下脚本发送一个应触发并行工具调用的请求，验证响应中包含这些调用，并格式化工具结果以保持并行性正常工作。请在环境中设置 `ANTHROPIC_API_KEY` 后运行它：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    此端到端测试需要的逻辑超出了单个 shell 命令的能力。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 来运行它。它发送的请求是一个定义了两个工具的标准工具使用请求。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此端到端测试需要的逻辑超出了单个 shell 命令的能力。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 来运行它。
  </CodeGroupItem>

  ```python Python
  client = Anthropic()

  # 定义工具
  tools = [
      {
          "name": "get_weather",
          "description": "Get the current weather in a given location",
          "input_schema": {
              "type": "object",
              "properties": {
                  "location": {
                      "type": "string",
                      "description": "The city and state, e.g. San Francisco, CA",
                  }
              },
              "required": ["location"],
          },
      },
      {
          "name": "get_time",
          "description": "Get the current time in a given timezone",
          "input_schema": {
              "type": "object",
              "properties": {
                  "timezone": {
                      "type": "string",
                      "description": "The timezone, e.g. America/New_York",
                  }
              },
              "required": ["timezone"],
          },
      },
  ]

  # 测试包含并行工具调用的对话
  messages = [
      {
          "role": "user",
          "content": "What's the weather in SF and NYC, and what time is it there?",
      }
  ]

  # 发起初始请求
  print("Requesting parallel tool calls...")
  response = client.messages.create(
      model="claude-opus-5-5", max_tokens=1024, messages=messages, tools=tools
  )

  # 检查是否存在并行工具调用
  tool_uses = [block for block in response.content if block.type == "tool_use"]
  print(f"\n✓ Claude made {len(tool_uses)} tool calls")

  if len(tool_uses) > 1:
      print("✓ Parallel tool calls detected!")
      for tool in tool_uses:
          print(f"  - {tool.name}: {tool.input}")
  else:
      print("✗ No parallel tool calls detected")

  # 模拟工具执行并正确格式化结果
  tool_results = []
  for tool_use in tool_uses:
      if tool_use.name == "get_weather":
          if "San Francisco" in str(tool_use.input):
              result = "San Francisco: 68°F, partly cloudy"
          else:
              result = "New York: 45°F, clear skies"
      else:  # get_time
          if "Los_Angeles" in str(tool_use.input):
              result = "2:30 PM PST"
          else:
              result = "5:30 PM EST"

      tool_results.append(
          {"type": "tool_result", "tool_use_id": tool_use.id, "content": result}
      )

  # 使用工具结果继续对话
  messages.extend(
      [
          {"role": "assistant", "content": response.content},
          {"role": "user", "content": tool_results},  # All results in one message!
      ]
  )

  # 获取最终响应
  print("\nGetting final response...")
  final_response = client.messages.create(
      model="claude-opus-5-5", max_tokens=1024, messages=messages, tools=tools
  )

  final_text = next(
      block.text for block in final_response.content if block.type == "text"
  )
  print(f"\nClaude's response:\n{final_text}")

  # 验证格式
  print("\n--- Verification ---")
  print(f"✓ Tool results sent in single user message: {len(tool_results)} results")
  print("✓ No text before tool results in content array")
  print("✓ Conversation formatted correctly for future parallel tool use")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 定义工具
  const tools: Anthropic.Tool[] = [
    {
      name: "get_weather",
      description: "Get the current weather in a given location",
      input_schema: {
        type: "object" as const,
        properties: {
          location: {
            type: "string",
            description: "The city and state, e.g. San Francisco, CA"
          }
        },
        required: ["location"]
      }
    },
    {
      name: "get_time",
      description: "Get the current time in a given timezone",
      input_schema: {
        type: "object" as const,
        properties: {
          timezone: {
            type: "string",
            description: "The timezone, e.g. America/New_York"
          }
        },
        required: ["timezone"]
      }
    }
  ];

  // 发起初始请求
  console.log("Requesting parallel tool calls...");
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: "What's the weather in SF and NYC, and what time is it there?"
      }
    ],
    tools: tools
  });

  // 检查是否存在并行工具调用
  const toolUses = response.content.filter((block) => block.type === "tool_use");
  console.log(`\n✓ Claude made ${toolUses.length} tool calls`);

  if (toolUses.length > 1) {
    console.log("✓ Parallel tool calls detected!");
    for (const tool of toolUses) {
      if (tool.type === "tool_use") {
        console.log(`  - ${tool.name}: ${JSON.stringify(tool.input)}`);
      }
    }
  } else {
    console.log("✗ No parallel tool calls detected");
  }

  // 模拟工具执行并正确格式化结果
  const toolResults: Anthropic.ToolResultBlockParam[] = toolUses
    .filter((block) => block.type === "tool_use")
    .map((toolUse) => {
      const input = toolUse.input as Record<string, string>;
      let result: string;
      if (toolUse.name === "get_weather") {
        result = input.location?.includes("San Francisco")
          ? "San Francisco: 68F, partly cloudy"
          : "New York: 45F, clear skies";
      } else {
        result = input.timezone?.includes("Los_Angeles") ? "2:30 PM PST" : "5:30 PM EST";
      }

      return {
        type: "tool_result" as const,
        tool_use_id: toolUse.id,
        content: result
      };
    });

  // 获取格式正确的最终响应
  console.log("\nGetting final response...");
  const finalResponse = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: "What's the weather in SF and NYC, and what time is it there?"
      },
      { role: "assistant", content: response.content },
      { role: "user", content: toolResults }
    ],
    tools: tools
  });

  for (const block of finalResponse.content) {
    if (block.type === "text") {
      console.log(`\nClaude's response:\n${block.text}`);
    }
  }

  // 验证格式
  console.log("\n--- Verification ---");
  console.log(`✓ Tool results sent in single user message: ${toolResults.length} results`);
  console.log("✓ No text before tool results in content array");
  console.log("✓ Conversation formatted correctly for future parallel tool use");
  ```

  ```csharp C#
  AnthropicClient client = new();

  var tools = new List<ToolUnion>
  {
      new ToolUnion(new Tool()
      {
          Name = "get_weather",
          Description = "Get the current weather in a given location",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["location"] = JsonSerializer.SerializeToElement(new { type = "string", description = "The city and state, e.g. San Francisco, CA" }),
              },
              Required = ["location"],
          },
      }),
      new ToolUnion(new Tool()
      {
          Name = "get_time",
          Description = "Get the current time in a given timezone",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["timezone"] = JsonSerializer.SerializeToElement(new { type = "string", description = "The timezone, e.g. America/New_York" }),
              },
              Required = ["timezone"],
          },
      }),
  };

  Console.WriteLine("Requesting parallel tool calls...");
  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "What's the weather in SF and NYC, and what time is it there?" }],
      Tools = tools
  };

  var response = await client.Messages.Create(parameters);

  var toolUses = new List<ToolUseBlock>();
  foreach (var block in response.Content)
  {
      if (block.TryPickToolUse(out var toolUse))
      {
          toolUses.Add(toolUse);
      }
  }
  Console.WriteLine($"\n✓ Claude made {toolUses.Count} tool calls");

  if (toolUses.Count > 1)
  {
      Console.WriteLine("✓ Parallel tool calls detected!");
      foreach (var tool in toolUses)
      {
          Console.WriteLine($"  - {tool.Name}: {JsonSerializer.Serialize(tool.Input)}");
      }
  }
  else
  {
      Console.WriteLine("✗ No parallel tool calls detected");
  }

  var toolResults = new List<ContentBlockParam>();
  foreach (var toolUse in toolUses)
  {
      string result;
      if (toolUse.Name == "get_weather")
      {
          result = JsonSerializer.Serialize(toolUse.Input).Contains("San Francisco")
              ? "San Francisco: 68°F, partly cloudy"
              : "New York: 45°F, clear skies";
      }
      else
      {
          result = JsonSerializer.Serialize(toolUse.Input).Contains("Los_Angeles")
              ? "2:30 PM PST"
              : "5:30 PM EST";
      }

      toolResults.Add(new ContentBlockParam(new ToolResultBlockParam()
      {
          ToolUseID = toolUse.ID,
          Content = result,
      }));
  }

  Console.WriteLine("\nGetting final response...");
  var finalParameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [
          new UserMessageParam { Content = "What's the weather in SF and NYC, and what time is it there?" },
          new AssistantMessageParam { Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList() },
          new UserMessageParam { Content = toolResults }
      ],
      Tools = tools
  };

  var finalResponse = await client.Messages.Create(finalParameters);
  var text = finalResponse.Content.Select(block => block.Value).OfType<TextBlock>().FirstOrDefault();
  Console.WriteLine($"\nClaude's response:\n{text?.Text}");

  Console.WriteLine("\n--- Verification ---");
  Console.WriteLine($"✓ Tool results sent in single user message: {toolResults.Count} results");
  Console.WriteLine("✓ No text before tool results in content array");
  Console.WriteLine("✓ Conversation formatted correctly for future parallel tool use");
  ```

  ```go Go
  client := anthropic.NewClient()

  tools := []anthropic.ToolUnionParam{
  	{OfTool: &anthropic.ToolParam{
  		Name:        "get_weather",
  		Description: anthropic.String("Get the current weather in a given location"),
  		InputSchema: anthropic.ToolInputSchemaParam{
  			Properties: map[string]any{
  				"location": map[string]any{
  					"type":        "string",
  					"description": "The city and state, e.g. San Francisco, CA",
  				},
  			},
  			Required: []string{"location"},
  		},
  	}},
  	{OfTool: &anthropic.ToolParam{
  		Name:        "get_time",
  		Description: anthropic.String("Get the current time in a given timezone"),
  		InputSchema: anthropic.ToolInputSchemaParam{
  			Properties: map[string]any{
  				"timezone": map[string]any{
  					"type":        "string",
  					"description": "The timezone, e.g. America/New_York",
  				},
  			},
  			Required: []string{"timezone"},
  		},
  	}},
  }

  fmt.Println("Requesting parallel tool calls...")
  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather in SF and NYC, and what time is it there?")),
  	},
  	Tools: tools,
  })
  if err != nil {
  	log.Fatal(err)
  }

  // 使用类型 switch 查找工具使用块
  type toolUseInfo struct {
  	ID    string
  	Name  string
  	Input json.RawMessage
  }
  var toolUses []toolUseInfo
  for _, block := range response.Content {
  	switch variant := block.AsAny().(type) {
  	case anthropic.ToolUseBlock:
  		toolUses = append(toolUses, toolUseInfo{
  			ID:    variant.ID,
  			Name:  variant.Name,
  			Input: variant.Input,
  		})
  	}
  }

  fmt.Printf("\n✓ Claude made %d tool calls\n", len(toolUses))

  if len(toolUses) > 1 {
  	fmt.Println("✓ Parallel tool calls detected!")
  	for _, tool := range toolUses {
  		fmt.Printf("  - %s: %s\n", tool.Name, string(tool.Input))
  	}
  } else {
  	fmt.Println("✗ No parallel tool calls detected")
  }

  // 构建工具结果
  var toolResults []anthropic.ContentBlockParamUnion
  for _, toolUse := range toolUses {
  	var result string
  	inputStr := string(toolUse.Input)

  	if toolUse.Name == "get_weather" {
  		if strings.Contains(inputStr, "San Francisco") {
  			result = "San Francisco: 68°F, partly cloudy"
  		} else {
  			result = "New York: 45°F, clear skies"
  		}
  	} else {
  		if strings.Contains(inputStr, "Los_Angeles") {
  			result = "2:30 PM PST"
  		} else {
  			result = "5:30 PM EST"
  		}
  	}

  	toolResults = append(toolResults, anthropic.NewToolResultBlock(toolUse.ID, result, false))
  }

  fmt.Println("\nGetting final response...")
  finalResponse, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather in SF and NYC, and what time is it there?")),
  		response.ToParam(),
  		anthropic.NewUserMessage(toolResults...),
  	},
  	Tools: tools,
  })
  if err != nil {
  	log.Fatal(err)
  }

  var finalText string
  for _, block := range finalResponse.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		finalText = textBlock.Text
  		break
  	}
  }

  fmt.Printf("\nClaude's response:\n%s\n", finalText)

  fmt.Println("\n--- Verification ---")
  fmt.Printf("✓ Tool results sent in single user message: %d results\n", len(toolResults))
  fmt.Println("✓ No text before tool results in content array")
  fmt.Println("✓ Conversation formatted correctly for future parallel tool use")
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  Tool weatherTool = Tool.builder()
      .name("get_weather")
      .description("Get the current weather in a given location")
      .inputSchema(InputSchema.builder()
          .properties(JsonValue.from(Map.of(
              "location", Map.of(
                  "type", "string",
                  "description", "The city and state, e.g. San Francisco, CA"
              )
          )))
          .putAdditionalProperty("required", JsonValue.from(List.of("location")))
          .build())
      .build();

  Tool timeTool = Tool.builder()
      .name("get_time")
      .description("Get the current time in a given timezone")
      .inputSchema(InputSchema.builder()
          .properties(JsonValue.from(Map.of(
              "timezone", Map.of(
                  "type", "string",
                  "description", "The timezone, e.g. America/New_York"
              )
          )))
          .putAdditionalProperty("required", JsonValue.from(List.of("timezone")))
          .build())
      .build();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .addTool(weatherTool)
      .addTool(timeTool)
      .addUserMessage("What's the weather in SF and NYC, and what time is it there?")
      .build();

  IO.println("Requesting parallel tool calls...");
  Message response = client.messages().create(params);

  List<ToolUseBlock> toolUses = new ArrayList<>();
  for (ContentBlock block : response.content()) {
      if (block.toolUse().isPresent()) {
          toolUses.add(block.toolUse().get());
      }
  }

  IO.println("\n✓ Claude made " + toolUses.size() + " tool calls");

  if (toolUses.size() > 1) {
      IO.println("✓ Parallel tool calls detected!");
      for (ToolUseBlock tool : toolUses) {
          IO.println("  - " + tool.name() + ": " + tool._input());
      }
  } else {
      IO.println("✗ No parallel tool calls detected");
  }

  List<ContentBlockParam> toolResults = new ArrayList<>();
  for (ToolUseBlock toolUse : toolUses) {
      String result;
      if (toolUse.name().equals("get_weather")) {
          String location = toolUse._input().toString();
          result = location.contains("San Francisco")
              ? "San Francisco: 68°F, partly cloudy"
              : "New York: 45°F, clear skies";
      } else {
          String timezone = toolUse._input().toString();
          result = timezone.contains("Los_Angeles")
              ? "2:30 PM PST"
              : "5:30 PM EST";
      }
      toolResults.add(ContentBlockParam.ofToolResult(
          ToolResultBlockParam.builder()
              .toolUseId(toolUse.id())
              .content(result)
              .build()
      ));
  }

  IO.println("\nGetting final response...");
  MessageCreateParams finalParams = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .addTool(weatherTool)
      .addTool(timeTool)
      .addUserMessage("What's the weather in SF and NYC, and what time is it there?")
      .addMessage(response)
      .addUserMessageOfContentBlockParams(toolResults)
      .build();

  Message finalResponse = client.messages().create(finalParams);
  finalResponse.content().stream()
      .flatMap(block -> block.text().stream())
      .forEach(textBlock -> IO.println("\nClaude's response:\n" + textBlock.text()));

  IO.println("\n--- Verification ---");
  IO.println("✓ Tool results sent in single user message: " + toolResults.size() + " results");
  IO.println("✓ No text before tool results in content array");
  IO.println("✓ Conversation formatted correctly for future parallel tool use");
  ```

  ```php PHP
  $client = new Client();

  $tools = [
      [
          'name' => 'get_weather',
          'description' => 'Get the current weather in a given location',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'location' => [
                      'type' => 'string',
                      'description' => 'The city and state, e.g. San Francisco, CA'
                  ]
              ],
              'required' => ['location']
          ]
      ],
      [
          'name' => 'get_time',
          'description' => 'Get the current time in a given timezone',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'timezone' => [
                      'type' => 'string',
                      'description' => 'The timezone, e.g. America/New_York'
                  ]
              ],
              'required' => ['timezone']
          ]
      ]
  ];

  echo "Requesting parallel tool calls...\n";
  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => "What's the weather in SF and NYC, and what time is it there?"]
      ],
      model: 'claude-opus-5-5',
      tools: $tools,
  );

  $toolUses = array_filter($response->content, fn($block) => $block->type === 'tool_use');
  echo "\n✓ Claude made " . count($toolUses) . " tool calls\n";

  if (count($toolUses) > 1) {
      echo "✓ Parallel tool calls detected!\n";
      foreach ($toolUses as $tool) {
          echo "  - {$tool->name}: " . json_encode($tool->input) . "\n";
      }
  } else {
      echo "✗ No parallel tool calls detected\n";
  }

  $toolResults = [];
  foreach ($toolUses as $toolUse) {
      if ($toolUse->name === 'get_weather') {
          $result = str_contains(json_encode($toolUse->input), 'San Francisco')
              ? 'San Francisco: 68°F, partly cloudy'
              : 'New York: 45°F, clear skies';
      } else {
          $result = str_contains(json_encode($toolUse->input), 'Los_Angeles')
              ? '2:30 PM PST'
              : '5:30 PM EST';
      }

      $toolResults[] = [
          'type' => 'tool_result',
          'tool_use_id' => $toolUse->id,
          'content' => $result
      ];
  }

  echo "\nGetting final response...\n";
  $finalResponse = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => "What's the weather in SF and NYC, and what time is it there?"],
          ['role' => 'assistant', 'content' => $response->content],
          ['role' => 'user', 'content' => $toolResults]
      ],
      model: 'claude-opus-5-5',
      tools: $tools,
  );

  $textBlock = array_find($finalResponse->content, static fn ($block): bool => $block->type === 'text');
  echo "\nClaude's response:\n{$textBlock->text}\n";

  echo "\n--- Verification ---\n";
  echo '✓ Tool results sent in single user message: ' . count($toolResults) . " results\n";
  echo "✓ No text before tool results in content array\n";
  echo "✓ Conversation formatted correctly for future parallel tool use\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  tools = [
    {
      name: "get_weather",
      description: "Get the current weather in a given location",
      input_schema: {
        type: "object",
        properties: {
          location: {
            type: "string",
            description: "The city and state, e.g. San Francisco, CA"
          }
        },
        required: ["location"]
      }
    },
    {
      name: "get_time",
      description: "Get the current time in a given timezone",
      input_schema: {
        type: "object",
        properties: {
          timezone: {
            type: "string",
            description: "The timezone, e.g. America/New_York"
          }
        },
        required: ["timezone"]
      }
    }
  ]

  puts "Requesting parallel tool calls..."
  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [
      { role: "user", content: "What's the weather in SF and NYC, and what time is it there?" }
    ],
    tools: tools
  )

  tool_uses = response.content.select { |block| block.type == :tool_use }
  puts "\n✓ Claude made #{tool_uses.length} tool calls"

  if tool_uses.length > 1
    puts "✓ Parallel tool calls detected!"
    tool_uses.each do |tool|
      puts "  - #{tool.name}: #{tool.input}"
    end
  else
    puts "✗ No parallel tool calls detected"
  end

  tool_results = tool_uses.map do |tool_use|
    result = if tool_use.name == "get_weather"
      location = tool_use.input[:location].to_s
      location.include?("San Francisco") ? "San Francisco: 68°F, partly cloudy" : "New York: 45°F, clear skies"
    else
      timezone = tool_use.input[:timezone].to_s
      timezone.include?("Los_Angeles") ? "2:30 PM PST" : "5:30 PM EST"
    end

    {
      type: "tool_result",
      tool_use_id: tool_use.id,
      content: result
    }
  end

  puts "\nGetting final response..."
  final_response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [
      { role: "user", content: "What's the weather in SF and NYC, and what time is it there?" },
      { role: "assistant", content: response.content },
      { role: "user", content: tool_results }
    ],
    tools: tools
  )

  final_text = final_response.content.find { |block| block.type == :text }
  puts "\nClaude's response:\n#{final_text.text}"

  puts "\n--- Verification ---"
  puts "✓ Tool results sent in single user message: #{tool_results.length} results"
  puts "✓ No text before tool results in content array"
  puts "✓ Conversation formatted correctly for future parallel tool use"
  ```
</CodeGroup>

末尾的摘要行重申了保持并行性正常工作的两条格式规则：所有工具结果都在单条用户消息中返回，并且该消息中工具结果之前不出现任何文本内容。

## 最大化并行工具使用

当请求能从多个工具中受益时，Claude 4 及更高版本的模型默认会进行并行工具调用。对于所有模型，您都可以通过有针对性的提示来提高并行工具调用的可能性：

<AccordionGroup>
  <Accordion title="用于并行工具使用的系统提示">
    对于 Claude 4 及更高版本的模型，请将以下内容添加到您的系统提示中：

    ```text wrap
    For maximum efficiency, whenever you need to perform multiple independent operations, invoke all relevant tools simultaneously rather than sequentially.
    ```

    如需更强的并行工具使用（如果默认设置不够，建议使用），请使用：

    ```text wrap
    <use_parallel_tool_calls>
    For maximum efficiency, whenever you perform multiple independent operations, invoke all relevant tools simultaneously rather than sequentially. Prioritize calling tools in parallel whenever possible. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. When running multiple read-only commands like `ls` or `list_dir`, always run all of the commands in parallel. Err on the side of maximizing parallel tool calls rather than running too many tools sequentially.
    </use_parallel_tool_calls>
    ```
  </Accordion>

  <Accordion title="用户消息提示">
    您也可以在特定的用户消息中鼓励并行工具使用：

    ```text wrap
    Instead of:
    "What's the weather in Paris? Also check London."

    Use:
    "Check the weather in Paris and London simultaneously."

    Or be explicit:
    "Please use parallel tool calls to get the weather for Paris, London, and Tokyo at the same time."
    ```
  </Accordion>
</AccordionGroup>

<Note>
  **长智能体循环中的 Claude Fable 5.1**

  Claude Fable 5.1 发出的并行工具调用可能比早期模型少，这在下一步读取操作仅为隐含的长智能体循环中最为明显（自定义编码智能体、bash 和文本编辑器框架、计算机使用）。标准函数调用不受影响。有关要添加的批处理指令及其放置位置，请参阅[在智能体循环中批量处理独立的工具调用](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#batch-independent-tool-calls-in-agent-loops)。
</Note>

## 禁用并行工具使用

并行工具使用默认开启。要关闭它，请在 [`tool_choice`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#forcing-tool-use) 对象内设置 `disable_parallel_tool_use: true`。它不是顶层请求参数。其效果取决于 `tool_choice` 的类型。

### 最多一次工具调用

当 `tool_choice` 类型为 `auto`（默认值）时，设置 `disable_parallel_tool_use: true` 意味着 Claude 在每个响应中最多调用一个工具。Claude 仍然可以不调用任何工具而以纯文本作答。高亮显示的行是与标准工具使用请求相比唯一的改动：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
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
          "required": ["location"]
        }
      }],
      "tool_choice": {"type": "auto", "disable_parallel_tool_use": true},
      "messages": [
        {"role": "user", "content": "What is the weather in San Francisco and New York?"}
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
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
  tool_choice:
    type: auto
    disable_parallel_tool_use: true
  messages:
    - role: user
      content: What is the weather in San Francisco and New York?
  YAML
  ```

  ```python Python
  client = Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[
          {
              "name": "get_weather",
              "description": "Get the current weather in a given location",
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "location": {
                          "type": "string",
                          "description": "The city and state, e.g. San Francisco, CA",
                      }
                  },
                  "required": ["location"],
              },
          }
      ],
      tool_choice={"type": "auto", "disable_parallel_tool_use": True},
      messages=[
          {
              "role": "user",
              "content": "What is the weather in San Francisco and New York?",
          }
      ],
  )
  print(response.content)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [
      {
        name: "get_weather",
        description: "Get the current weather in a given location",
        input_schema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "The city and state, e.g. San Francisco, CA"
            }
          },
          required: ["location"]
        }
      }
    ],
    tool_choice: { type: "auto", disable_parallel_tool_use: true },
    messages: [{ role: "user", content: "What is the weather in San Francisco and New York?" }]
  });
  console.log(response.content);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = [
          new ToolUnion(new Tool()
          {
              Name = "get_weather",
              Description = "Get the current weather in a given location",
              InputSchema = new InputSchema()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["location"] = JsonSerializer.SerializeToElement(new { type = "string", description = "The city and state, e.g. San Francisco, CA" }),
                  },
                  Required = ["location"],
              },
          }),
      ],
      ToolChoice = new ToolChoiceAuto { DisableParallelToolUse = true },
      Messages = [new UserMessageParam { Content = "What is the weather in San Francisco and New York?" }]
  };

  var response = await client.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{
  		{OfTool: &anthropic.ToolParam{
  			Name:        "get_weather",
  			Description: anthropic.String("Get the current weather in a given location"),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"location": map[string]any{
  						"type":        "string",
  						"description": "The city and state, e.g. San Francisco, CA",
  					},
  				},
  				Required: []string{"location"},
  			},
  		}},
  	},
  	ToolChoice: anthropic.ToolChoiceUnionParam{
  		OfAuto: &anthropic.ToolChoiceAutoParam{
  			DisableParallelToolUse: anthropic.Bool(true),
  		},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is the weather in San Francisco and New York?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.Content)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  InputSchema schema = InputSchema.builder()
      .properties(
          JsonValue.from(
              Map.of(
                  "location", Map.of(
                      "type", "string",
                      "description", "The city and state, e.g. San Francisco, CA"
                  )
              )
          )
      )
      .putAdditionalProperty("required", JsonValue.from(List.of("location")))
      .build();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .addTool(
          Tool.builder()
              .name("get_weather")
              .description("Get the current weather in a given location")
              .inputSchema(schema)
              .build()
      )
      .toolChoice(ToolChoiceAuto.builder()
          .disableParallelToolUse(true)
          .build())
      .addUserMessage("What is the weather in San Francisco and New York?")
      .build();

  Message response = client.messages().create(params);
  IO.println(response.content());
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => 'What is the weather in San Francisco and New York?']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      toolChoice: ['type' => 'auto', 'disableParallelToolUse' => true],
      tools: [
          [
              'name' => 'get_weather',
              'description' => 'Get the current weather in a given location',
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'location' => [
                          'type' => 'string',
                          'description' => 'The city and state, e.g. San Francisco, CA'
                      ]
                  ],
                  'required' => ['location']
              ]
          ]
      ],
  );

  echo $response;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: [
      {
        name: "get_weather",
        description: "Get the current weather in a given location",
        input_schema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "The city and state, e.g. San Francisco, CA"
            }
          },
          required: ["location"]
        }
      }
    ],
    tool_choice: { type: "auto", disable_parallel_tool_use: true },
    messages: [
      { role: "user", content: "What is the weather in San Francisco and New York?" }
    ]
  )
  puts response.content
  ```
</CodeGroup>

### 恰好一次工具调用

当 `tool_choice` 类型为 `any` 或 `tool` 时，设置 `disable_parallel_tool_use: true` 意味着 Claude 恰好调用一个工具。Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5 和 Claude Sonnet 5.5 不支持这些 `tool_choice` 类型（请参阅[强制工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#forcing-tool-use)）。以下示例使用 `any`。同一字段也适用于 `tool`：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5",
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
          "required": ["location"]
        }
      }],
      "tool_choice": {"type": "any", "disable_parallel_tool_use": true},
      "messages": [
        {"role": "user", "content": "What is the weather in San Francisco and New York?"}
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5
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
  tool_choice:
    type: any
    disable_parallel_tool_use: true
  messages:
    - role: user
      content: What is the weather in San Francisco and New York?
  YAML
  ```

  ```python Python
  client = Anthropic()

  response = client.messages.create(
      model="claude-opus-5",
      max_tokens=1024,
      tools=[
          {
              "name": "get_weather",
              "description": "Get the current weather in a given location",
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "location": {
                          "type": "string",
                          "description": "The city and state, e.g. San Francisco, CA",
                      }
                  },
                  "required": ["location"],
              },
          }
      ],
      tool_choice={"type": "any", "disable_parallel_tool_use": True},
      messages=[
          {
              "role": "user",
              "content": "What is the weather in San Francisco and New York?",
          }
      ],
  )
  print(response.content)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5",
    max_tokens: 1024,
    tools: [
      {
        name: "get_weather",
        description: "Get the current weather in a given location",
        input_schema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "The city and state, e.g. San Francisco, CA"
            }
          },
          required: ["location"]
        }
      }
    ],
    tool_choice: { type: "any", disable_parallel_tool_use: true },
    messages: [{ role: "user", content: "What is the weather in San Francisco and New York?" }]
  });
  console.log(response.content);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5,
      MaxTokens = 1024,
      Tools = [
          new ToolUnion(new Tool()
          {
              Name = "get_weather",
              Description = "Get the current weather in a given location",
              InputSchema = new InputSchema()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["location"] = JsonSerializer.SerializeToElement(new { type = "string", description = "The city and state, e.g. San Francisco, CA" }),
                  },
                  Required = ["location"],
              },
          }),
      ],
      ToolChoice = new ToolChoiceAny { DisableParallelToolUse = true },
      Messages = [new UserMessageParam { Content = "What is the weather in San Francisco and New York?" }]
  };

  var response = await client.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{
  		{OfTool: &anthropic.ToolParam{
  			Name:        "get_weather",
  			Description: anthropic.String("Get the current weather in a given location"),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"location": map[string]any{
  						"type":        "string",
  						"description": "The city and state, e.g. San Francisco, CA",
  					},
  				},
  				Required: []string{"location"},
  			},
  		}},
  	},
  	ToolChoice: anthropic.ToolChoiceUnionParam{
  		OfAny: &anthropic.ToolChoiceAnyParam{
  			DisableParallelToolUse: anthropic.Bool(true),
  		},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is the weather in San Francisco and New York?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.Content)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  InputSchema schema = InputSchema.builder()
      .properties(
          JsonValue.from(
              Map.of(
                  "location", Map.of(
                      "type", "string",
                      "description", "The city and state, e.g. San Francisco, CA"
                  )
              )
          )
      )
      .putAdditionalProperty("required", JsonValue.from(List.of("location")))
      .build();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5)
      .maxTokens(1024L)
      .addTool(
          Tool.builder()
              .name("get_weather")
              .description("Get the current weather in a given location")
              .inputSchema(schema)
              .build()
      )
      .toolChoice(ToolChoiceAny.builder()
          .disableParallelToolUse(true)
          .build())
      .addUserMessage("What is the weather in San Francisco and New York?")
      .build();

  Message response = client.messages().create(params);
  IO.println(response.content());
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => 'What is the weather in San Francisco and New York?']
      ],
      model: Model::CLAUDE_OPUS_5,
      toolChoice: ['type' => 'any', 'disableParallelToolUse' => true],
      tools: [
          [
              'name' => 'get_weather',
              'description' => 'Get the current weather in a given location',
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'location' => [
                          'type' => 'string',
                          'description' => 'The city and state, e.g. San Francisco, CA'
                      ]
                  ],
                  'required' => ['location']
              ]
          ]
      ],
  );

  echo $response;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5,
    max_tokens: 1024,
    tools: [
      {
        name: "get_weather",
        description: "Get the current weather in a given location",
        input_schema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "The city and state, e.g. San Francisco, CA"
            }
          },
          required: ["location"]
        }
      }
    ],
    tool_choice: { type: "any", disable_parallel_tool_use: true },
    messages: [
      { role: "user", content: "What is the weather in San Francisco and New York?" }
    ]
  )
  puts response.content
  ```
</CodeGroup>

## 故障排除

如果 Claude 没有按预期进行并行工具调用，请检查以下常见问题：

**1. 工具结果格式不正确**

最常见的问题是在对话历史中错误地格式化工具结果。这会"教会" Claude 避免并行调用。

具体针对并行工具使用：

* **错误：** 为每个工具结果使用单独的用户消息
* **正确：** 所有工具结果一起放在单条用户消息中

```json
// Wrong: separate user messages reduce parallel tool use
[
  {"role": "assistant", "content": [tool_use_1, tool_use_2]},
  {"role": "user", "content": [tool_result_1]},
  {"role": "user", "content": [tool_result_2]}  // Separate message
]

// Correct: one user message with all results maintains parallel tool use
[
  {"role": "assistant", "content": [tool_use_1, tool_use_2]},
  {"role": "user", "content": [tool_result_1, tool_result_2]}  // Single message
]
```

有关其他格式规则，请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。

**2. 提示力度不足**

默认提示可能不够。请使用[最大化并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use#maximizing-parallel-tool-use)中更强的系统提示。

**3. 衡量并行工具使用情况**

要验证并行工具调用是否正常工作：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    要衡量并行工具使用情况，您的代码需要分析已收集的响应。这需要的逻辑超出了单个 shell 命令的能力。请改用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    要衡量并行工具使用情况，您的代码需要分析已收集的响应。这需要的逻辑超出了单个 shell 命令的能力。请改用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。
  </CodeGroupItem>

  ```python Python
  messages = []  # Message objects returned by client.messages.create across your run

  tool_call_messages = [
      msg for msg in messages if any(block.type == "tool_use" for block in msg.content)
  ]
  total_tool_calls = sum(
      len([block for block in msg.content if block.type == "tool_use"])
      for msg in tool_call_messages
  )
  avg_tools_per_message = (
      total_tool_calls / len(tool_call_messages) if tool_call_messages else 0.0
  )
  print(f"Average tools per message: {avg_tools_per_message}")
  # 如果并行调用正常工作，该值应 > 1.0
  ```

  ```typescript TypeScript
  const messages: Anthropic.Message[] = []; // Message objects returned by client.messages.create across your run

  const toolCallMessages = messages.filter((message) =>
    message.content.some((block) => block.type === "tool_use")
  );
  const totalToolCalls = toolCallMessages.reduce(
    (sum, message) => sum + message.content.filter((block) => block.type === "tool_use").length,
    0
  );
  const avgToolsPerMessage =
    toolCallMessages.length > 0 ? totalToolCalls / toolCallMessages.length : 0;
  console.log(`Average tools per message: ${avgToolsPerMessage}`);
  // 如果并行调用正常工作，该值应 > 1.0
  ```

  ```csharp C#
  List<Message> messages = []; // Message objects returned by client.Messages.Create across your run

  var toolCallMessages = messages
      .Where(message => message.Content.Any(block => block.TryPickToolUse(out _)))
      .ToList();
  var totalToolCalls = toolCallMessages
      .Sum(message => message.Content.Count(block => block.TryPickToolUse(out _)));
  var avgToolsPerMessage = toolCallMessages.Count > 0 ? (double)totalToolCalls / toolCallMessages.Count : 0.0;
  Console.WriteLine($"Average tools per message: {avgToolsPerMessage}");
  // 如果并行调用正常工作，该值应 > 1.0
  ```

  ```go Go
  var messages []anthropic.Message // Message values returned by client.Messages.New across your run

  toolCallMessageCount := 0
  totalToolCalls := 0
  for _, message := range messages {
  	callsInMessage := 0
  	for _, block := range message.Content {
  		if block.Type == "tool_use" {
  			callsInMessage++
  		}
  	}
  	if callsInMessage > 0 {
  		toolCallMessageCount++
  		totalToolCalls += callsInMessage
  	}
  }

  avgToolsPerMessage := 0.0
  if toolCallMessageCount > 0 {
  	avgToolsPerMessage = float64(totalToolCalls) / float64(toolCallMessageCount)
  }
  fmt.Println("Average tools per message:", avgToolsPerMessage)
  // 如果并行调用正常工作，该值应 > 1.0
  ```

  ```java Java
  List<Message> messages = List.of(); // Message objects returned by client.messages().create() across your run

  List<Message> toolCallMessages = messages.stream()
      .filter(message -> message.content().stream().anyMatch(ContentBlock::isToolUse))
      .toList();
  long totalToolCalls = toolCallMessages.stream()
      .mapToLong(message -> message.content().stream().filter(ContentBlock::isToolUse).count())
      .sum();
  double avgToolsPerMessage = toolCallMessages.isEmpty() ? 0.0 : (double) totalToolCalls / toolCallMessages.size();
  IO.println("Average tools per message: " + avgToolsPerMessage);
  // 如果并行调用正常工作，该值应 > 1.0
  ```

  ```php PHP
  // $messages：在整个运行过程中由 $client->messages->create() 返回的 Message 对象
  $messages = [];

  $toolCallMessages = array_values(array_filter(
      $messages,
      fn ($message) => count(array_filter($message->content, fn ($block) => $block->type === 'tool_use')) > 0
  ));
  $totalToolCalls = array_sum(array_map(
      fn ($message) => count(array_filter($message->content, fn ($block) => $block->type === 'tool_use')),
      $toolCallMessages
  ));
  $avgToolsPerMessage = count($toolCallMessages) > 0 ? $totalToolCalls / count($toolCallMessages) : 0.0;
  echo "Average tools per message: {$avgToolsPerMessage}\n";
  // 如果并行调用正常工作，该值应 > 1.0
  ```

  ```ruby Ruby
  messages = [] # Message objects returned by client.messages.create across your run

  tool_call_messages = messages.select { |message| message.content.any? { |block| block.type == :tool_use } }
  total_tool_calls = tool_call_messages.sum { |message| message.content.count { |block| block.type == :tool_use } }
  avg_tools_per_message = tool_call_messages.empty? ? 0.0 : total_tool_calls.to_f / tool_call_messages.size
  puts "Average tools per message: #{avg_tools_per_message}"
  # 如果并行调用正常工作，该值应 > 1.0
  ```
</CodeGroup>

**4. 批次中的调用似乎相互依赖**

执行顺序由您选择。如果您的工具存在顺序依赖，依次运行该批次并在第一次失败时停止是一种有效的策略（也是[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)和[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)工具所要求的策略）：对于您未运行的任何调用，返回 `is_error: true`。如果您并行运行，而某个调用因其前置条件尚未完成而失败，请返回 `is_error: true` 并附上自然的错误消息。Claude 将在下一轮次重新发出该调用。要减少相互依赖的调用一起出现的情况，请将以下内容添加到您的系统提示中："Only batch tool calls that are independent of each other."

## 后续步骤

<CardGroup cols={3}>
  <Card title="工具运行器（SDK）" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner">
    使用 SDK 的工具运行器抽象自动处理代理循环、错误包装和类型安全。
  </Card>

  <Card title="处理工具调用" icon="arrows-left-right" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls">
    解析 `tool_use` 块，格式化 `tool_result` 响应，并使用 `is_error` 处理错误。
  </Card>

  <Card title="定义工具" icon="hammer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools">
    指定工具模式、编写有效的描述，并控制 Claude 何时调用您的工具。
  </Card>
</CardGroup>

## 可用性

并行工具使用可用或处于 beta 阶段，具体取决于API。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：默认开启
* 用法：如需关闭，请在 `tool_choice` 中设置 `disable_parallel_tool_use: true`。

| 状态 | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |
