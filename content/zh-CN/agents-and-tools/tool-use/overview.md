---
title: 使用 Claude 进行工具使用
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview
description: 将 Claude 连接到外部工具和 API。了解工具在何处运行、Claude 何时调用它们，以及哪种工具适合您的任务。
---

“Tool use”（工具使用，也称为 function calling（函数调用））允许 Claude 调用您定义的函数或 Anthropic 提供的函数。Claude 根据用户的请求和工具的描述来决定何时调用工具。然后，它会返回一个结构化调用，由您的应用程序运行（客户端工具）或由 Anthropic 运行（服务器工具）。

以下是一个使用服务器工具的最小示例，即[网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)，它由 Anthropic 为您运行：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "tools": [{"type": "web_search_20260209", "name": "web_search"}],
      "messages": [{"role": "user", "content": "What is the latest on the Mars rover?"}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --tool '{type: web_search_20260209, name: web_search}' \
    --message '{role: user, content: "What is the latest on the Mars rover?"}'
  ```

  ```python Python
  client = anthropic.Anthropic()
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      tools=[{"type": "web_search_20260209", "name": "web_search"}],
      messages=[{"role": "user", "content": "What is the latest on the Mars rover?"}],
  )
  print(response.content)
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    tools: [{ type: "web_search_20260209", name: "web_search" }],
    messages: [{ role: "user", content: "What is the latest on the Mars rover?" }]
  });
  console.log(response.content);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Tools = [new ToolUnion(new WebSearchTool20260209())],
      Messages = [new UserMessageParam { Content = "What is the latest on the Mars rover?" }]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message.Content);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Tools: []anthropic.ToolUnionParam{
  		{OfWebSearchTool20260209: &anthropic.WebSearchTool20260209Param{}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is the latest on the Mars rover?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.Content)
  ```

  ```java Java
  import com.anthropic.models.messages.WebSearchTool20260209;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addTool(WebSearchTool20260209.builder().build())
          .addUserMessage("What is the latest on the Mars rover?")
          .build();

      Message response = client.messages().create(params);
      IO.println(response.content());
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 4096,
      tools: [
          ['type' => 'web_search_20260209', 'name' => 'web_search'],
      ],
      messages: [
          ['role' => 'user', 'content' => 'What is the latest on the Mars rover?'],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    tools: [{ type: "web_search_20260209", name: "web_search" }],
    messages: [{ role: "user", content: "What is the latest on the Mars rover?" }]
  )
  puts message.content
  ```
</CodeGroup>

Claude 在 Anthropic 的基础设施上运行搜索，并在同一响应中返回带有引用的结果。要让 Claude 调用您定义的函数，请传入一个带有 `input_schema` 的工具，然后在 Claude 返回 `tool_use` 块时运行该调用。[客户端工具示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#how-tool-use-works)完整展示了这一往返过程。详细了解[定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)和[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。

## 工具使用的工作原理

工具的主要区别在于代码运行的位置。**客户端工具**（包括用户定义的工具和具有 Anthropic 定义架构的工具，例如 `bash` 和 `text_editor`）在您的应用程序中运行。Claude 会以 `stop_reason: "tool_use"` 和一个或多个 `tool_use` 块进行响应。您的代码执行该操作并发回 `tool_result`。**服务器工具**（例如 `web_search`、`web_fetch`、`code_execution` 和 `tool_search`）在 Anthropic 的基础设施上运行：您可以直接看到结果，无需处理执行，除非 Claude 在与您的某个客户端工具相同的一组并行工具调用中调用了该工具（请参阅[停止原因和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#tool-use)）。

下面是客户端工具的完整往返过程。第一个请求定义了一个 `get_weather` 工具，Claude 通过调用它来回答问题：响应中携带一个 `tool_use` 块，您的代码运行查询，然后第二个请求在 `tool_result` 块中将结果发回，以便 Claude 能够给出答案。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  client = anthropic.Anthropic()

  tools = [
      {
          "name": "get_weather",
          "description": "Get the current weather for a given location.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "location": {
                      "type": "string",
                      "description": "City and state, e.g. San Francisco, CA",
                  }
              },
              "required": ["location"],
          },
      }
  ]
  messages = [{"role": "user", "content": "What's the weather in San Francisco?"}]

  # Claude 会返回一个 tool_use 块，其中包含工具名称及其参数。
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=tools,
      # 每轮最多请求一次工具调用。
      tool_choice={"type": "auto", "disable_parallel_tool_use": True},
      messages=messages,
  )
  tool_use = next(block for block in response.content if block.type == "tool_use")
  print(f"Claude called {tool_use.name} with {json.dumps(tool_use.input)}")

  # 运行该工具，然后通过 tool_result 块将结果发回。
  weather = "15 degrees Celsius, partly cloudy"  # your weather lookup goes here
  messages += [
      {"role": "assistant", "content": response.content},
      {
          "role": "user",
          "content": [
              {"type": "tool_result", "tool_use_id": tool_use.id, "content": weather}
          ],
      },
  ]
  followup = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=tools,
      tool_choice={"type": "auto", "disable_parallel_tool_use": True},
      messages=messages,
  )

  # Claude 会利用该结果回答最初的问题。
  final_text = next(block for block in followup.content if block.type == "text")
  print(final_text.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const tools: Anthropic.Tool[] = [
    {
      name: "get_weather",
      description: "Get the current weather for a given location.",
      input_schema: {
        type: "object",
        properties: {
          location: { type: "string", description: "City and state, e.g. San Francisco, CA" }
        },
        required: ["location"]
      }
    }
  ];
  const messages: Anthropic.MessageParam[] = [
    { role: "user", content: "What's the weather in San Francisco?" }
  ];

  // Claude 会返回一个 tool_use 块，其中包含工具名称及其参数。
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools,
    // 每轮最多请求一次工具调用。
    tool_choice: { type: "auto", disable_parallel_tool_use: true },
    messages
  });
  const toolUse = response.content.find((block) => block.type === "tool_use")!;
  console.log(`Claude called ${toolUse.name} with ${JSON.stringify(toolUse.input)}`);

  // 运行该工具，然后通过 tool_result 块将结果发回。
  const weather = "15 degrees Celsius, partly cloudy"; // your weather lookup goes here
  messages.push(
    { role: "assistant", content: response.content },
    {
      role: "user",
      content: [{ type: "tool_result", tool_use_id: toolUse.id, content: weather }]
    }
  );
  const followup = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools,
    tool_choice: { type: "auto", disable_parallel_tool_use: true },
    messages
  });

  // Claude 会利用该结果回答最初的问题。
  const finalText = followup.content.find((block) => block.type === "text")!;
  console.log(finalText.text);
  ```

  ```csharp C#
  AnthropicClient client = new();

  List<ToolUnion> tools =
  [
      new ToolUnion(new Tool()
      {
          Name = "get_weather",
          Description = "Get the current weather for a given location.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["location"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "string",
                      description = "City and state, e.g. San Francisco, CA",
                  }),
              },
              Required = ["location"],
          },
      }),
  ];

  // 每轮最多请求一次工具调用。
  var toolChoice = new ToolChoice(new ToolChoiceAuto { DisableParallelToolUse = true });

  const string userPrompt = "What's the weather in San Francisco?";

  // Claude 会返回一个 tool_use 块，其中包含工具名称及其参数。
  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = tools,
      ToolChoice = toolChoice,
      Messages = [new UserMessageParam { Content = userPrompt }],
  });
  ToolUseBlock? toolUse = null;
  foreach (var block in response.Content)
  {
      if (block.TryPickToolUse(out var picked))
      {
          toolUse = picked;
          break;
      }
  }
  Console.WriteLine($"Claude called {toolUse!.Name} with {JsonSerializer.Serialize(toolUse.Input)}");

  // 运行该工具，然后在 tool_result 块中将结果发回。
  var weather = "15 degrees Celsius, partly cloudy";
  List<ContentBlockParam> toolResults =
  [
      new ContentBlockParam(new ToolResultBlockParam()
      {
          ToolUseID = toolUse.ID,
          Content = weather,
      }),
  ];
  var followup = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = tools,
      ToolChoice = toolChoice,
      Messages =
      [
          new UserMessageParam { Content = userPrompt },
          new AssistantMessageParam { Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList() },
          new UserMessageParam { Content = toolResults },
      ],
  });

  // Claude 会利用该结果回答最初的问题。
  foreach (var block in followup.Content)
  {
      if (block.TryPickText(out var text))
      {
          Console.WriteLine(text.Text);
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()
  ctx := context.Background()

  tools := []anthropic.ToolUnionParam{
  	{OfTool: &anthropic.ToolParam{
  		Name:        "get_weather",
  		Description: anthropic.String("Get the current weather for a given location."),
  		InputSchema: anthropic.ToolInputSchemaParam{
  			Properties: map[string]any{
  				"location": map[string]any{
  					"type":        "string",
  					"description": "City and state, e.g. San Francisco, CA",
  				},
  			},
  			Required: []string{"location"},
  		},
  	}},
  }
  // 每轮最多请求一次工具调用。
  toolChoice := anthropic.ToolChoiceUnionParam{
  	OfAuto: &anthropic.ToolChoiceAutoParam{DisableParallelToolUse: anthropic.Bool(true)},
  }
  messages := []anthropic.MessageParam{
  	anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather in San Francisco?")),
  }

  // Claude 会返回一个 tool_use 块，其中包含工具名称及其参数。
  response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:      anthropic.ModelClaudeOpus5_5,
  	MaxTokens:  1024,
  	Tools:      tools,
  	ToolChoice: toolChoice,
  	Messages:   messages,
  })
  if err != nil {
  	log.Fatal(err)
  }
  var toolUse anthropic.ContentBlockUnion
  for _, block := range response.Content {
  	if block.Type == "tool_use" {
  		toolUse = block
  		break
  	}
  }
  fmt.Printf("Claude called %s with %s\n", toolUse.Name, string(toolUse.Input))

  // 运行该工具，然后在 tool_result 块中将结果发回。
  weather := "15 degrees Celsius, partly cloudy"
  var assistantContent []anthropic.ContentBlockParamUnion
  for _, block := range response.Content {
  	assistantContent = append(assistantContent, block.ToParam())
  }
  messages = append(messages,
  	anthropic.NewAssistantMessage(assistantContent...),
  	anthropic.NewUserMessage(anthropic.NewToolResultBlock(toolUse.ID, weather, false)),
  )
  followup, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:      anthropic.ModelClaudeOpus5_5,
  	MaxTokens:  1024,
  	Tools:      tools,
  	ToolChoice: toolChoice,
  	Messages:   messages,
  })
  if err != nil {
  	log.Fatal(err)
  }

  // Claude 会利用该结果回答最初的问题。
  for _, block := range followup.Content {
  	if block.Type == "text" {
  		fmt.Println(block.Text)
  	}
  }
  ```

  ```java Java
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.messages.ContentBlockParam;
  // ...
  import com.anthropic.models.messages.Tool;
  import com.anthropic.models.messages.Tool.InputSchema;
  import com.anthropic.models.messages.ToolChoiceAuto;
  import com.anthropic.models.messages.ToolResultBlockParam;
  import com.anthropic.models.messages.ToolUseBlock;
  // ...

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Tool weatherTool = Tool.builder()
          .name("get_weather")
          .description("Get the current weather for a given location.")
          .inputSchema(InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "location", Map.of(
                      "type", "string",
                      "description", "City and state, e.g. San Francisco, CA"
                  )
              )))
              .required(List.of("location"))
              .build())
          .build();

      // 每个轮次最多请求一次工具调用。
      ToolChoiceAuto toolChoice = ToolChoiceAuto.builder()
          .disableParallelToolUse(true)
          .build();

      String userPrompt = "What's the weather in San Francisco?";

      // Claude 会返回一个 tool_use 块，其中包含工具名称及其参数。
      Message response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(weatherTool)
          .toolChoice(toolChoice)
          .addUserMessage(userPrompt)
          .build());
      ToolUseBlock toolUse = response.content().stream()
          .flatMap(block -> block.toolUse().stream())
          .findFirst()
          .orElseThrow();
      IO.println("Claude called " + toolUse.name() + " with " + toolUse._input());

      // 运行该工具，然后通过 tool_result 块将结果发回。
      String weather = "15 degrees Celsius, partly cloudy";
      Message followup = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(weatherTool)
          .toolChoice(toolChoice)
          .addUserMessage(userPrompt)
          .addMessage(response)
          .addUserMessageOfContentBlockParams(List.of(ContentBlockParam.ofToolResult(
              ToolResultBlockParam.builder()
                  .toolUseId(toolUse.id())
                  .content(weather)
                  .build())))
          .build());

      // Claude 会利用该结果回答最初的问题。
      followup.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  use Anthropic\Messages\ToolChoiceAuto;

  $client = new Client();

  $tools = [
      [
          'name' => 'get_weather',
          'description' => 'Get the current weather for a given location.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'location' => [
                      'type' => 'string',
                      'description' => 'City and state, e.g. San Francisco, CA',
                  ],
              ],
              'required' => ['location'],
          ],
      ],
  ];
  $userMessage = ['role' => 'user', 'content' => "What's the weather in San Francisco?"];

  // 每轮最多请求一次工具调用。
  $toolChoice = ToolChoiceAuto::with(disableParallelToolUse: true);

  // Claude 会返回一个 tool_use 块，其中包含工具名称及其参数。
  $response = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      tools: $tools,
      toolChoice: $toolChoice,
      messages: [$userMessage],
  );
  $toolUse = null;
  foreach ($response->content as $block) {
      if ($block->type === 'tool_use') {
          $toolUse = $block;
          break;
      }
  }
  printf("Claude called %s with %s\n", $toolUse->name, json_encode($toolUse->input));

  // 运行该工具，然后通过 tool_result 块将结果发回。
  $weather = '15 degrees Celsius, partly cloudy';
  $followup = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      tools: $tools,
      toolChoice: $toolChoice,
      messages: [
          $userMessage,
          ['role' => 'assistant', 'content' => $response->content],
          [
              'role' => 'user',
              'content' => [
                  [
                      'type' => 'tool_result',
                      'tool_use_id' => $toolUse->id,
                      'content' => $weather,
                  ],
              ],
          ],
      ],
  );

  // Claude 会利用该结果回答最初的问题。
  foreach ($followup->content as $block) {
      if ($block->type === 'text') {
          echo $block->text, "\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  tools = [
    {
      name: "get_weather",
      description: "Get the current weather for a given location.",
      input_schema: {
        type: "object",
        properties: {
          location: {type: "string", description: "City and state, e.g. San Francisco, CA"}
        },
        required: ["location"]
      }
    }
  ]
  messages = [{role: "user", content: "What's the weather in San Francisco?"}]

  # Claude 会返回一个 tool_use 块，其中包含工具名称及其参数。
  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: tools,
    # 每轮最多请求一次工具调用。
    tool_choice: {type: "auto", disable_parallel_tool_use: true},
    messages: messages
  )
  tool_use = response.content.find { |block| block.type == :tool_use }
  puts "Claude called #{tool_use.name} with #{JSON.generate(tool_use.input)}"

  # 运行该工具，然后通过 tool_result 块将结果发回。
  weather = "15 degrees Celsius, partly cloudy"
  messages += [
    {role: "assistant", content: response.content},
    {
      role: "user",
      content: [
        {type: "tool_result", tool_use_id: tool_use.id, content: weather}
      ]
    }
  ]
  followup = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: tools,
    tool_choice: {type: "auto", disable_parallel_tool_use: true},
    messages: messages
  )

  # Claude 会利用该结果回答最初的问题。
  final_text = followup.content.find { |block| block.type == :text }
  puts final_text.text
  ```
</CodeGroup>

```text Output wrap
Claude called get_weather with {"location": "San Francisco, CA"}
The current weather in San Francisco is 15 degrees Celsius with partly cloudy skies.
```

[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)详细介绍了每个步骤，包括结果格式化和错误信号；[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)介绍了同时调用多个工具的响应。如果不想自己编写这一往返过程，请使用 [tool runner](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)（工具运行器）：SDK 会运行您的工具并自动发回结果。

有关包括智能体循环在内的完整概念模型以及何时选择每种方法，请参阅[工具使用的工作原理](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/how-tool-use-works)。

要连接到 “Model Context Protocol”，即 MCP 服务器，请参阅 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)。要构建您自己的 MCP 客户端，请参阅 Model Context Protocol 指南中的[构建 MCP 客户端](https://modelcontextprotocol.io/docs/develop/build-client)。

## Claude 何时使用工具

在默认的 `tool_choice` 为 `{"type": "auto"}` 的情况下，Claude 在每一轮中决定是调用工具还是直接响应。当请求与某个工具所描述的能力相对应，且答案尚不在上下文中时，它会调用该工具。对于稳定的知识、创意任务和对话性轮次，它会直接响应。

这一边界可以通过您的 “system prompt”（系统提示）进行引导。如果 Claude 没有在您期望的时候调用工具，一条轻量的指令，例如 `"Use the tools to investigate before responding."`，可以增加工具使用。更强的形式，例如 `"Always call a tool first before responding."`，会进一步推动。相反，`"Use your judgment about whether to call a tool or respond directly."` 会使触发行为保持保守。

如果要强制要求工具调用而不是依赖提示，请设置 [`tool_choice`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#forcing-tool-use)。

<Tip>
  **通过严格工具使用保证 schema 一致性**

  在您的自定义工具定义中添加 `strict: true`，以确保 Claude 的工具调用始终与您的 schema 完全匹配。请参阅[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)。
</Tip>

每个服务器工具的页面都更详细地描述了其自身的触发边界。

<Accordion title="当缺少必需参数时">
  如果用户的提示没有包含足够的信息来填充工具的所有必需参数，Claude Opus 识别出缺少某个参数并询问该参数的可能性要大得多。Claude Sonnet 可能会询问，尤其是在被提示在输出工具请求之前先进行思考时。但它也可能推断出一个合理的值。

  例如，给定一个需要 `location` 参数的 `get_weather` 工具，如果您在未指定位置的情况下问 Claude “What’s the weather?”，Claude（尤其是 Claude Sonnet）可能会猜测您未提供的值：

  ```json JSON
  {
    "type": "tool_use",
    "id": "toolu_01A09q90qw90lq917835lq9",
    "name": "get_weather",
    "input": { "location": "New York, NY", "unit": "fahrenheit" }
  }
  ```

  这种行为并不能保证，尤其是对于更模糊的提示和能力较弱的模型。
</Accordion>

## 选择工具

有关 `type` 字符串、版本和 beta 头，请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)。

### 您自己的工具

对于您定义的工具，由您编写架构，并由您的应用程序运行每次调用。

<CardGroup cols={2}>
  <Card title="定义工具" icon="hammer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools">
    指定工具 schema、编写描述，并控制 Claude 何时调用您的工具。
  </Card>

  <Card title="处理工具调用" icon="arrows-left-right" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls">
    解析 `tool_use` 块、格式化 `tool_result` 响应并处理错误。
  </Card>
</CardGroup>

### Anthropic schema 客户端工具

Anthropic 发布架构并基于它训练 Claude。您的应用程序仍需运行每次调用并返回 `tool_result`。

<CardGroup cols={2}>
  <Card title="记忆工具" icon="brain" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool">
    在您控制的文件中跨对话存储和检索信息。
  </Card>

  <Card title="Bash 工具" icon="terminal" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool">
    在保持状态的持久会话中运行 shell 命令。
  </Card>

  <Card title="文本编辑器工具" icon="edit" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool">
    查看和修改文本文件，以调试、修复和改进代码。
  </Card>

  <Card title="计算机使用工具" icon="computer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool">
    在桌面环境中截取屏幕截图并控制鼠标和键盘。
  </Card>

  <Card title="浏览器使用工具" icon="browser" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool">
    在您自己的浏览器环境中导航、阅读网页并与之交互。
  </Card>
</CardGroup>

### 服务器工具

服务器工具在 Anthropic 的基础设施上运行，您的应用程序中无需处理程序代码。有关它们共有的机制，请参阅[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)。

<CardGroup cols={2}>
  <Card title="网页搜索工具" icon="magnifying-glass" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool">
    在网络上搜索知识截止日期之后的信息，并附带引用来源。
  </Card>

  <Card title="网页获取工具" icon="download" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool">
    检索指定网页和 PDF 文档的完整内容。
  </Card>

  <Card title="代码执行工具" icon="code" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool">
    在沙盒容器中运行 Python 和 Bash 代码，以分析数据和生成文件。
  </Card>

  <Card title="顾问工具" icon="lightbulb" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool">
    让速度更快的执行模型在生成过程中咨询智能程度更高的顾问模型。
  </Card>

  <Card title="工具搜索工具" icon="library" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool">
    通过按需发现和加载工具来使用数千种工具。
  </Card>

  <Card title="MCP 连接器" icon="link" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector">
    无需单独的 MCP 客户端，即可从 Messages API 连接到远程 MCP 服务器。
  </Card>
</CardGroup>

<Note>
  [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 提供了一套内置工具集，Claude 可在会话中自主使用。有关该工具集以及 Managed Agents 添加自定义工具的方式，请参阅其[工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools)页面。
</Note>

## 定价

工具使用请求的定价基于：

1. 发送给模型的输入令牌总数（包括 `tools` 参数中的令牌）。
2. 生成的输出令牌数量。
3. 对于服务器端工具，还有额外的基于用量的定价（例如，网页搜索按每次执行的搜索收费）。

客户端工具的定价与任何其他 Claude API 请求相同，但服务器端工具可能会根据其具体用量产生额外费用。

工具使用产生的额外令牌来自：

* API 请求中的 `tools` 参数（工具名称、描述和模式）
* API 请求和响应中的 `tool_use` 内容块
* API 请求中的 `tool_result` 内容块

当您使用 `tools` 时，API 还会自动为模型包含一个启用工具使用的特殊系统提示。下表列出了每个模型所需的工具使用令牌数量（不包括前面列出的额外令牌）。该表假设您至少提供了 1 个工具。如果您未提供 `tools`，则工具选择为 `none` 时使用 0 个额外的系统提示令牌。

| Model                                                                                                                                                                     | Tool use system prompt tokens: auto, none | Tool use system prompt tokens: any, tool |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------------- | :--------------------------------------- |
| Claude Opus 5.5                                                                                                                                                           | 286 tokens                                |                                          |
| Claude Opus 5                                                                                                                                                             | 286 tokens                                | 406 tokens                               |
| Claude Opus 4.8                                                                                                                                                           | 290 tokens                                | 410 tokens                               |
| Claude Opus 4.7                                                                                                                                                           | 675 tokens                                | 804 tokens                               |
| Claude Opus 4.6                                                                                                                                                           | 497 tokens                                | 589 tokens                               |
| Claude Opus 4.5                                                                                                                                                           | 496 tokens                                | 588 tokens                               |
| Claude Opus 4.1 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | 313 tokens                                | 315 tokens                               |
| Claude Opus 4 ([retired, except on Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))               | 313 tokens                                | 315 tokens                               |
| Claude Sonnet 5.5                                                                                                                                                         | 286 tokens                                |                                          |
| Claude Sonnet 5                                                                                                                                                           | 354 tokens                                | 474 tokens                               |
| Claude Sonnet 4.6                                                                                                                                                         | 497 tokens                                | 589 tokens                               |
| Claude Sonnet 4.5                                                                                                                                                         | 496 tokens                                | 588 tokens                               |
| Claude Sonnet 4 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | 313 tokens                                | 315 tokens                               |
| Claude Haiku 5.5                                                                                                                                                          | 286 tokens                                | 406 tokens                               |
| Claude Haiku 4.5                                                                                                                                                          | 496 tokens                                | 588 tokens                               |

* **auto, none:** The count when tool\_choice is auto or none.
* **any, tool:** The count when tool\_choice is any or tool.
* **Retired:** May still be available on other cloud platforms. See [Model deprecations](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) for more.

这些令牌数会加到您正常的输入和输出令牌中，以计算请求的总费用。

有关当前各模型的价格，请参阅[所有模型](https://platform.claude.com/docs/zh-CN/models/overview#all-models)表格。

当您发送工具使用提示时，与任何其他 API 请求一样，响应会在报告的 `usage` 指标中同时包含输入和输出令牌计数。

某些服务器工具会在令牌之外增加基于用量的费用：有关其费率，请参阅[网页搜索工具用量和定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#usage-and-pricing)和[代码执行工具用量和定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#usage-and-pricing)。

## 后续步骤

<CardGroup cols={3}>
  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/how-tool-use-works" title="工具使用的工作原理" icon="compass">
    了解工具使用循环、工具在何处运行，以及何时应使用工具而非纯文本。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/build-a-tool-using-agent" title="教程：构建一个使用工具的智能体" icon="graduation-cap">
    从单次工具调用到可用于生产的智能体循环的引导式演练。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference" title="工具参考" icon="book">
    Anthropic 提供的工具目录以及可选工具定义属性的参考。
  </Card>
</CardGroup>
