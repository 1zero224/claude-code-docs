---
title: 细粒度工具流式传输
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming
description: 为延迟敏感型应用流式传输工具输入，无需服务器端 JSON 缓冲。
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
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

“Fine-grained tool streaming”（细粒度工具流式传输）会在 Claude 生成工具输入的同时将其传送到您的客户端，而无需服务器端缓冲或 JSON 验证。跳过缓冲步骤可以缩短大型参数（例如文档或代码块）首个片段的到达时间，并且这些片段通过与标准工具使用相同的[流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)事件到达。

<Warning>
  由于 API 在流式传输工具输入之前不会对其进行缓冲或验证，您可能会收到不完整或无效的 JSON。以[停止原因](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons) `max_tokens` 结束的响应也可能在参数中途将其截断。请累积这些片段、对解析进行防护，并参阅[处理工具响应中的无效 JSON](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming#handling-invalid-json-in-tool-responses)，了解如何将无法解析的输入返回给 Claude。
</Warning>

## 如何使用细粒度工具流式传输

所有模型均在 Claude API、[Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上支持细粒度工具流式传输。要使用它，请在您希望启用细粒度流式传输的任何用户定义工具上将 `eager_input_streaming` 设置为 `true`，并在您的请求中启用流式传输。

`eager_input_streaming` 字段是可选的。将其设置为 `true` 会为该工具开启细粒度流式传输，而省略它则会使用标准的缓冲流式传输，即 API 在将每个参数值流式传回之前对其进行缓冲和验证。例外情况是发送 `fine-grained-tool-streaming-2025-05-14` beta 标头的请求，该标头会为未设置此字段的工具开启细粒度流式传输。按工具设置的字段取代了该标头，并且即使请求发送该标头，显式的 `false` 也会使该工具保持缓冲流式传输。该标头不能与[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)或[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)工具集条目组合使用：API 会拒绝同时发送两者的请求，因此请移除该标头，并在需要它的用户定义工具上设置 `eager_input_streaming`。有关字段定义，请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)。

以下示例为 `make_file` 工具开启细粒度流式传输，并要求 Claude 写一首长诗，这样工具输入就足够大，可以观察它流式传入：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 65536,
      "tools": [
        {
          "name": "make_file",
          "description": "Write text to a file",
          "eager_input_streaming": true,
          "input_schema": {
            "type": "object",
            "properties": {
              "filename": {
                "type": "string",
                "description": "The filename to write text to"
              },
              "lines_of_text": {
                "type": "array",
                "description": "An array of lines of text to write to the file"
              }
            },
            "required": ["filename", "lines_of_text"]
          }
        }
      ],
      "messages": [
        {
          "role": "user",
          "content": "Can you write a long poem and make a file called poem.txt?"
        }
      ],
      "stream": true
    }'
  ```

  ```bash CLI
  ant messages create --stream <<'YAML'
  model: claude-opus-5-5
  max_tokens: 65536
  tools:
    - name: make_file
      description: Write text to a file
      eager_input_streaming: true
      input_schema:
        type: object
        properties:
          filename:
            type: string
            description: The filename to write text to
          lines_of_text:
            type: array
            description: An array of lines of text to write to the file
        required:
          - filename
          - lines_of_text
  messages:
    - role: user
      content: Can you write a long poem and make a file called poem.txt?
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  with client.messages.stream(
      max_tokens=65536,
      model="claude-opus-5-5",
      tools=[
          {
              "name": "make_file",
              "description": "Write text to a file",
              "eager_input_streaming": True,
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "filename": {
                          "type": "string",
                          "description": "The filename to write text to",
                      },
                      "lines_of_text": {
                          "type": "array",
                          "description": "An array of lines of text to write to the file",
                      },
                  },
                  "required": ["filename", "lines_of_text"],
              },
          }
      ],
      messages=[
          {
              "role": "user",
              "content": "Can you write a long poem and make a file called poem.txt?",
          }
      ],
  ) as stream:
      for event in stream:
          if event.type == "input_json":
              print(event.partial_json, end="", flush=True)
      final_message = stream.get_final_message()

  print()
  for block in final_message.content:
      if block.type == "tool_use":
          print(f"Complete tool input: {block.input}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const stream = client.messages.stream({
    model: "claude-opus-5-5",
    max_tokens: 65536,
    tools: [
      {
        name: "make_file",
        description: "Write text to a file",
        eager_input_streaming: true,
        input_schema: {
          type: "object",
          properties: {
            filename: {
              type: "string",
              description: "The filename to write text to"
            },
            lines_of_text: {
              type: "array",
              description: "An array of lines of text to write to the file"
            }
          },
          required: ["filename", "lines_of_text"]
        }
      }
    ],
    messages: [
      {
        role: "user",
        content: "Can you write a long poem and make a file called poem.txt?"
      }
    ]
  });

  stream.on("inputJson", (partialJson) => {
    process.stdout.write(partialJson);
  });

  const message = await stream.finalMessage();
  console.log();
  for (const block of message.content) {
    if (block.type === "tool_use") {
      console.log("Complete tool input:", block.input);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  MessageCreateParams parameters = new()
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 65536,
      Tools =
      [
          new Tool
          {
              Name = "make_file",
              Description = "Write text to a file",
              EagerInputStreaming = true,
              InputSchema = new InputSchema
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["filename"] = JsonSerializer.SerializeToElement(
                          new { type = "string", description = "The filename to write text to" }
                      ),
                      ["lines_of_text"] = JsonSerializer.SerializeToElement(
                          new { type = "array", description = "An array of lines of text to write to the file" }
                      ),
                  },
                  Required = ["filename", "lines_of_text"],
              },
          },
      ],
      Messages =
      [
          new UserMessageParam
          {
              Content = "Can you write a long poem and make a file called poem.txt?",
          },
      ],
  };

  // C# 示例自行组装输入：内容块索引 -> 累积的 JSON
  var toolInputs = new Dictionary<long, StringBuilder>();

  await foreach (var streamEvent in client.Messages.CreateStreaming(parameters))
  {
      if (
          streamEvent.TryPickContentBlockStart(out var start)
          && start.ContentBlock.TryPickToolUse(out _)
      )
      {
          toolInputs[start.Index] = new StringBuilder();
      }
      else if (
          streamEvent.TryPickContentBlockDelta(out var delta)
          && delta.Delta.TryPickInputJson(out var inputJson)
      )
      {
          Console.Write(inputJson.PartialJson);
          toolInputs[delta.Index].Append(inputJson.PartialJson);
      }
  }

  Console.WriteLine();
  foreach (var accumulatedInput in toolInputs.Values)
  {
      Console.WriteLine($"Complete tool input: {accumulatedInput}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  makeFileTool := anthropic.ToolParam{
  	Name:                "make_file",
  	Description:         anthropic.String("Write text to a file"),
  	EagerInputStreaming: anthropic.Bool(true),
  	InputSchema: anthropic.ToolInputSchemaParam{
  		Properties: map[string]any{
  			"filename": map[string]any{
  				"type":        "string",
  				"description": "The filename to write text to",
  			},
  			"lines_of_text": map[string]any{
  				"type":        "array",
  				"description": "An array of lines of text to write to the file",
  			},
  		},
  		Required: []string{"filename", "lines_of_text"},
  	},
  }

  stream := client.Messages.NewStreaming(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 65536,
  	Tools:     []anthropic.ToolUnionParam{{OfTool: &makeFileTool}},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock(
  			"Can you write a long poem and make a file called poem.txt?",
  		)),
  	},
  })

  message := anthropic.Message{}
  for stream.Next() {
  	event := stream.Current()
  	if err := message.Accumulate(event); err != nil {
  		panic(err)
  	}
  	if delta, ok := event.AsAny().(anthropic.ContentBlockDeltaEvent); ok {
  		if inputJSON, ok := delta.Delta.AsAny().(anthropic.InputJSONDelta); ok {
  			fmt.Print(inputJSON.PartialJSON)
  		}
  	}
  }
  if err := stream.Err(); err != nil {
  	panic(err)
  }

  fmt.Println()
  for _, block := range message.Content {
  	if toolUse, ok := block.AsAny().(anthropic.ToolUseBlock); ok {
  		fmt.Printf("Complete tool input: %s\n", toolUse.Input)
  	}
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  Tool makeFileTool = Tool.builder()
      .name("make_file")
      .description("Write text to a file")
      .eagerInputStreaming(true)
      .inputSchema(Tool.InputSchema.builder()
          .properties(Tool.InputSchema.Properties.builder()
              .putAdditionalProperty("filename", JsonValue.from(Map.of(
                  "type", "string",
                  "description", "The filename to write text to")))
              .putAdditionalProperty("lines_of_text", JsonValue.from(Map.of(
                  "type", "array",
                  "description", "An array of lines of text to write to the file")))
              .build())
          .addRequired("filename")
          .addRequired("lines_of_text")
          .build())
      .build();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(65536L)
      .addTool(makeFileTool)
      .addUserMessage("Can you write a long poem and make a file called poem.txt?")
      .build();

  MessageAccumulator accumulator = MessageAccumulator.create();

  try (StreamResponse<RawMessageStreamEvent> streamResponse =
          client.messages().createStreaming(params)) {
      streamResponse.stream().forEach(event -> {
          accumulator.accumulate(event);
          if (event.isContentBlockDelta()) {
              var delta = event.asContentBlockDelta().delta();
              if (delta.isInputJson()) {
                  IO.print(delta.asInputJson().partialJson());
              }
          }
      });
  }

  IO.println("");
  accumulator.message().content().forEach(block ->
      block.toolUse().ifPresent(toolUse ->
          IO.println("Complete tool input: " + toolUse._input())));
  ```

  ```php PHP
  use Anthropic\Client;
  use Anthropic\Messages\InputJSONDelta;
  use Anthropic\Messages\Model;
  use Anthropic\Messages\RawContentBlockDeltaEvent;
  use Anthropic\Messages\RawContentBlockStartEvent;
  use Anthropic\Messages\ToolUseBlock;

  $client = new Client();

  $stream = $client->messages->createStream(
      maxTokens: 65536,
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          [
              'name' => 'make_file',
              'description' => 'Write text to a file',
              'eager_input_streaming' => true,
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'filename' => [
                          'type' => 'string',
                          'description' => 'The filename to write text to',
                      ],
                      'lines_of_text' => [
                          'type' => 'array',
                          'description' => 'An array of lines of text to write to the file',
                      ],
                  ],
                  'required' => ['filename', 'lines_of_text'],
              ],
          ],
      ],
      messages: [
          [
              'role' => 'user',
              'content' => 'Can you write a long poem and make a file called poem.txt?',
          ],
      ],
  );

  // PHP 示例自行组装输入：index => 累积的 JSON 字符串
  $toolInputs = [];

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof RawContentBlockStartEvent:
              if ($event->contentBlock instanceof ToolUseBlock) {
                  $toolInputs[$event->index] = '';
              }
              break;
          case $event instanceof RawContentBlockDeltaEvent:
              if ($event->delta instanceof InputJSONDelta) {
                  echo $event->delta->partialJSON;
                  $toolInputs[$event->index] .= $event->delta->partialJSON;
              }
              break;
      }
  }

  echo "\n";
  foreach ($toolInputs as $toolInput) {
      echo "Complete tool input: {$toolInput}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  stream = client.messages.stream(
    model: Anthropic::Models::Model::CLAUDE_OPUS_5_5,
    max_tokens: 65_536,
    tools: [
      {
        name: "make_file",
        description: "Write text to a file",
        eager_input_streaming: true,
        input_schema: {
          type: "object",
          properties: {
            filename: {
              type: "string",
              description: "The filename to write text to"
            },
            lines_of_text: {
              type: "array",
              description: "An array of lines of text to write to the file"
            }
          },
          required: ["filename", "lines_of_text"]
        }
      }
    ],
    messages: [
      {
        role: "user",
        content: "Can you write a long poem and make a file called poem.txt?"
      }
    ]
  )

  stream.each do |event|
    print event.partial_json if event.is_a?(Anthropic::Streaming::InputJsonEvent)
  end

  puts
  stream.accumulated_message.content.each do |block|
    puts "Complete tool input: #{block.input}" if block.type == :tool_use
  end
  ```
</CodeGroup>

每个输入片段在 Claude 写出它的那一刻就会到达，这些片段拼接成完整的工具输入，因此这首诗会一段一段地到达：

```text wrap
{"filename": "poem.txt", "lines_of_text": ["The Wanderer's Journey", "", "I.", "", "Beneath the vast and star-strewn sky,", "Where silver moonbeams softly lie,", ...
Complete tool input: {"filename": "poem.txt", "lines_of_text": ["The Wanderer's Journey", ...]}
```

如果不使用 `eager_input_streaming`，API 会在将每个参数值流式传回之前对其进行缓冲和验证，因此在 Claude 完成生成大型参数之前不会打印任何内容。使用它之后，片段会从 Claude 开始生成参数的那一刻起到达，并且这些片段通常更长，在单词中间断开的情况也更少。

## 累积工具输入增量

累积约定与标准工具使用流式传输相同，因此本节在使用和不使用 `eager_input_streaming` 的情况下均适用。有关事件格式，请参阅流式传输消息中的[输入 JSON 增量](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming#input-json-delta)。细粒度工具流式传输改变的是您可以对结果做出的假设：服务器在不验证片段的情况下流式传输它们，因此累积的字符串可能不是有效的 JSON。

当 `tool_use` 内容块进行流式传输时，初始的 `content_block_start` 事件包含 `input: {}`（一个空对象）。这是一个占位符。实际输入以一系列 `input_json_delta` 事件的形式到达，每个事件携带一个 `partial_json` 字符串片段。要组装完整的输入，请将这些片段连接起来，并在块关闭时解析结果。

如果您的 SDK 提供了累积器辅助工具（如上一个示例中的 Python、TypeScript、Go、Java 和 Ruby 标签页所示），它会为您处理这些工作。当您的 SDK 没有辅助工具，或者您希望完全控制输入的组装方式时，请使用手动模式。

累积约定：

1. 在 `type: "tool_use"` 的 `content_block_start` 上，初始化一个空字符串：`input_json = ""`。
2. 对于每个 `type: "input_json_delta"` 的 `content_block_delta`，追加：`input_json += event.delta.partial_json`。
3. 在 `content_block_stop` 上，解析累积的字符串。

请像以下 SDK 示例那样对解析操作进行保护。响应也可能在参数中途因 `max_tokens` 而停止。请检查 [stop reason](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)（停止原因），并决定是使用更高的 `max_tokens` 重试请求，还是修复不完整的输入。

初始 `input: {}`（对象）与 `partial_json`（字符串）之间的类型不匹配是有意设计的。空对象标记了内容数组中的位置。增量字符串构建实际的值。

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    累积每个 `tool_use` 块的输入片段所需的逻辑超出了单个 shell 命令的能力。请改用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。要在片段流式传输时查看原始片段，请运行[如何使用细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming#how-to-use-fine-grained-tool-streaming)中的请求。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    累积每个 `tool_use` 块的输入片段所需的逻辑超出了单个 shell 命令的能力。请改用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。要在片段流式传输时查看原始片段，请运行[如何使用细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming#how-to-use-fine-grained-tool-streaming)中的请求。
  </CodeGroupItem>

  ```python Python
  client = anthropic.Anthropic()

  tool_inputs: dict[int, str] = {}  # index -> accumulated JSON string

  with client.messages.stream(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[
          {
              "name": "get_weather",
              "description": "Get current weather for a city",
              "eager_input_streaming": True,
              "input_schema": {
                  "type": "object",
                  "properties": {"city": {"type": "string"}},
                  "required": ["city"],
              },
          }
      ],
      messages=[{"role": "user", "content": "Weather in Paris?"}],
  ) as stream:
      for event in stream:
          match event.type:
              case "content_block_start" if event.content_block.type == "tool_use":
                  tool_inputs[event.index] = ""
              case "content_block_delta" if event.delta.type == "input_json_delta":
                  tool_inputs[event.index] += event.delta.partial_json
              case "content_block_stop" if event.index in tool_inputs:
                  raw_input = tool_inputs[event.index]
                  try:
                      parsed = json.loads(raw_input)
                  except json.JSONDecodeError:
                      # 累积的字符串不保证是有效的 JSON。
                      # 请参阅本页的“处理工具响应中的无效 JSON”。
                      print(f"Invalid tool input: {raw_input}")
                  else:
                      print(f"Tool input: {parsed}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const toolInputs = new Map<number, string>();

  const stream = client.messages.stream({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [
      {
        name: "get_weather",
        description: "Get current weather for a city",
        eager_input_streaming: true,
        input_schema: {
          type: "object",
          properties: { city: { type: "string" } },
          required: ["city"]
        }
      }
    ],
    messages: [{ role: "user", content: "Weather in Paris?" }]
  });

  for await (const event of stream) {
    switch (event.type) {
      case "content_block_start":
        if (event.content_block.type === "tool_use") {
          toolInputs.set(event.index, "");
        }
        break;
      case "content_block_delta":
        if (event.delta.type === "input_json_delta") {
          toolInputs.set(
            event.index,
            (toolInputs.get(event.index) ?? "") + event.delta.partial_json
          );
        }
        break;
      case "content_block_stop":
        if (toolInputs.has(event.index)) {
          const rawInput = toolInputs.get(event.index)!;
          try {
            console.log("Tool input:", JSON.parse(rawInput));
          } catch {
            // 累积的字符串不保证是有效的 JSON。
            // 请参阅本页的“处理工具响应中的无效 JSON”。
            console.log("Invalid tool input:", rawInput);
          }
        }
        break;
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  MessageCreateParams parameters = new()
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools =
      [
          new Tool
          {
              Name = "get_weather",
              Description = "Get current weather for a city",
              EagerInputStreaming = true,
              InputSchema = new InputSchema
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["city"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                  },
                  Required = ["city"],
              },
          },
      ],
      Messages = [new UserMessageParam { Content = "Weather in Paris?" }],
  };

  // 块索引 -> 累积的 JSON 片段
  // 本示例手动累积增量（delta），以展示原始流；
  // SDK 的 MessageContentAggregator 也可以自动累积工具输入。
  var toolInputs = new Dictionary<long, StringBuilder>();

  await foreach (var streamEvent in client.Messages.CreateStreaming(parameters))
  {
      if (
          streamEvent.TryPickContentBlockStart(out var start)
          && start.ContentBlock.TryPickToolUse(out _)
      )
      {
          toolInputs[start.Index] = new StringBuilder();
      }
      else if (
          streamEvent.TryPickContentBlockDelta(out var delta)
          && delta.Delta.TryPickInputJson(out var inputJson)
      )
      {
          toolInputs[delta.Index].Append(inputJson.PartialJson);
      }
      else if (
          streamEvent.TryPickContentBlockStop(out var stop)
          && toolInputs.TryGetValue(stop.Index, out var accumulated)
      )
      {
          try
          {
              using var parsed = JsonDocument.Parse(accumulated.ToString());
              Console.WriteLine($"Tool input: {parsed.RootElement}");
          }
          catch (JsonException)
          {
              // 累积得到的字符串不保证是有效的 JSON。
              // 请参阅本页的“处理工具响应中的无效 JSON”。
              Console.WriteLine($"Invalid tool input: {accumulated}");
          }
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  toolInputs := map[int64]string{} // content block index -> accumulated JSON

  stream := client.Messages.NewStreaming(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{{
  		OfTool: &anthropic.ToolParam{
  			Name:                "get_weather",
  			Description:         anthropic.String("Get current weather for a city"),
  			EagerInputStreaming: anthropic.Bool(true),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"city": map[string]any{"type": "string"},
  				},
  				Required: []string{"city"},
  			},
  		},
  	}},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Weather in Paris?")),
  	},
  })

  for stream.Next() {
  	switch event := stream.Current().AsAny().(type) {
  	case anthropic.ContentBlockStartEvent:
  		if _, ok := event.ContentBlock.AsAny().(anthropic.ToolUseBlock); ok {
  			toolInputs[event.Index] = ""
  		}
  	case anthropic.ContentBlockDeltaEvent:
  		if delta, ok := event.Delta.AsAny().(anthropic.InputJSONDelta); ok {
  			toolInputs[event.Index] += delta.PartialJSON
  		}
  	case anthropic.ContentBlockStopEvent:
  		if accumulated, ok := toolInputs[event.Index]; ok {
  			var parsed map[string]any
  			if err := json.Unmarshal([]byte(accumulated), &parsed); err != nil {
  				// 累积的字符串不保证是有效的 JSON。
  				// 请参阅本页的“处理工具响应中的无效 JSON”。
  				fmt.Println("Invalid tool input:", accumulated)
  			} else {
  				fmt.Println("Tool input:", parsed)
  			}
  		}
  	}
  }
  if err := stream.Err(); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();
  ObjectMapper objectMapper = new ObjectMapper();

  Tool weatherTool = Tool.builder()
      .name("get_weather")
      .description("Get current weather for a city")
      .eagerInputStreaming(true)
      .inputSchema(Tool.InputSchema.builder()
          .properties(Tool.InputSchema.Properties.builder()
              .putAdditionalProperty("city", JsonValue.from(Map.of("type", "string")))
              .build())
          .addRequired("city")
          .build())
      .build();

  MessageCreateParams createParams = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .addTool(weatherTool)
      .addUserMessage("Weather in Paris?")
      .build();

  // 内容块索引 -> 累积的工具输入 JSON
  Map<Long, StringBuilder> toolInputs = new HashMap<>();

  try (StreamResponse<RawMessageStreamEvent> streamResponse = client.messages().createStreaming(createParams)) {
      for (RawMessageStreamEvent event : streamResponse) {
          switch (event.type().value()) {
              case CONTENT_BLOCK_START -> {
                  var blockStart = event.asContentBlockStart();
                  if (blockStart.contentBlock().isToolUse()) {
                      toolInputs.put(blockStart.index(), new StringBuilder());
                  }
              }
              case CONTENT_BLOCK_DELTA -> {
                  var blockDelta = event.asContentBlockDelta();
                  if (blockDelta.delta().isInputJson() && toolInputs.containsKey(blockDelta.index())) {
                      toolInputs.get(blockDelta.index()).append(blockDelta.delta().asInputJson().partialJson());
                  }
              }
              case CONTENT_BLOCK_STOP -> {
                  var blockStop = event.asContentBlockStop();
                  if (toolInputs.containsKey(blockStop.index())) {
                      String accumulated = toolInputs.get(blockStop.index()).toString();
                      try {
                          IO.println("Tool input: " + objectMapper.readTree(accumulated));
                      } catch (JsonProcessingException e) {
                          // 累积的字符串不保证是有效的 JSON。
                          // 请参阅本页的“处理工具响应中的无效 JSON”部分。
                          IO.println("Invalid tool input: " + accumulated);
                      }
                  }
              }
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Client;
  use Anthropic\Messages\InputJSONDelta;
  use Anthropic\Messages\Model;
  use Anthropic\Messages\RawContentBlockDeltaEvent;
  use Anthropic\Messages\RawContentBlockStartEvent;
  use Anthropic\Messages\RawContentBlockStopEvent;
  use Anthropic\Messages\ToolUseBlock;

  $client = new Client();

  // PHP SDK 未提供用于工具输入的流累加器；
  // 此处展示的手动模式是受支持的做法。
  $toolInputs = []; // index => accumulated JSON string

  $stream = $client->messages->createStream(
      maxTokens: 1024,
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          [
              'name' => 'get_weather',
              'description' => 'Get current weather for a city',
              'eager_input_streaming' => true,
              'input_schema' => [
                  'type' => 'object',
                  'properties' => ['city' => ['type' => 'string']],
                  'required' => ['city'],
              ],
          ],
      ],
      messages: [['role' => 'user', 'content' => 'Weather in Paris?']],
  );

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof RawContentBlockStartEvent:
              if ($event->contentBlock instanceof ToolUseBlock) {
                  $toolInputs[$event->index] = '';
              }
              break;
          case $event instanceof RawContentBlockDeltaEvent:
              if ($event->delta instanceof InputJSONDelta) {
                  $toolInputs[$event->index] .= $event->delta->partialJSON;
              }
              break;
          case $event instanceof RawContentBlockStopEvent:
              if (isset($toolInputs[$event->index])) {
                  $accumulated = $toolInputs[$event->index];
                  try {
                      $parsed = json_decode($accumulated, associative: true, flags: JSON_THROW_ON_ERROR);
                      echo 'Tool input: ' . json_encode($parsed) . "\n";
                  } catch (\JsonException $e) {
                      // 累积得到的字符串不保证是有效的 JSON。
                      // 请参阅本页的“处理工具响应中的无效 JSON”。
                      echo "Invalid tool input: {$accumulated}\n";
                  }
              }
              break;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  tool_inputs = {} # index -> accumulated JSON string

  stream = client.messages.stream_raw(
    model: Anthropic::Models::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: [
      {
        name: "get_weather",
        description: "Get current weather for a city",
        eager_input_streaming: true,
        input_schema: {
          type: "object",
          properties: {city: {type: "string"}},
          required: ["city"]
        }
      }
    ],
    messages: [{role: "user", content: "Weather in Paris?"}]
  )

  stream.each do |event|
    case event
    when Anthropic::Models::RawContentBlockStartEvent
      tool_inputs[event.index] = +"" if event.content_block.type == :tool_use
    when Anthropic::Models::RawContentBlockDeltaEvent
      if event.delta.is_a?(Anthropic::Models::InputJSONDelta)
        tool_inputs[event.index] << event.delta.partial_json
      end
    when Anthropic::Models::RawContentBlockStopEvent
      if tool_inputs.key?(event.index)
        accumulated = tool_inputs[event.index]
        begin
          parsed = JSON.parse(accumulated)
          puts "Tool input: #{parsed}"
        rescue JSON::ParserError
          # 累积的字符串不保证是有效的 JSON。
          # 请参阅本页的“处理工具响应中的无效 JSON”。
          puts "Invalid tool input: #{accumulated}"
        end
      end
    end
  end
  ```
</CodeGroup>

<Tip>
  对片段做出反应和组装片段是两个独立的关注点。第一个示例在每个片段到达时对其做出反应，并且在使用累积器辅助工具的标签页中仍然将组装工作交给 SDK。当您不使用累积器辅助工具，或者希望完全控制组装过程时，请使用手动模式。
</Tip>

## 处理工具响应中的无效 JSON

使用细粒度工具流式传输时，工具调用的累积输入可能是无效或不完整的 JSON。在这种情况下，您无法运行该工具，因此应将失败情况报告给 Claude。工具结果的 `content` 不必是 JSON，但将原始字符串包装在一个单键 JSON 对象中，可以让 Claude 明确知道您收到了无效的 JSON，并保留原始输入以便调试：

```json
{
  "INVALID_JSON": "<the unparseable input you received>"
}
```

将该包装对象序列化为字符串，作为[工具结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls#handling-errors-with-is-error)内容块的 `content` 返回，并将 `is_error` 设置为 `true`：

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
  "is_error": true,
  "content": "{\"INVALID_JSON\": \"<the unparseable input you received>\"}"
}
```

<Note>
  请使用您的 JSON 库构建包装对象，而不是通过拼接字符串，这样无效输入中的引号和其他特殊字符才能被正确转义。
</Note>

## 后续步骤

<CardGroup cols={2}>
  <Card title="上下文窗口" icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows">
    了解上下文窗口的工作原理、扩展思考和工具使用如何计入其中，以及如何随着对话增长管理上下文。
  </Card>

  <Card title="流式传输消息" icon="lightning" href="https://platform.claude.com/docs/zh-CN/build-with-claude/streaming">
    使用服务器发送事件增量流式传输 Messages API 响应，包括文本、工具使用和扩展思考增量。
  </Card>

  <Card title="处理工具调用" icon="arrows-left-right" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls">
    解析 `tool_use` 块，格式化 `tool_result` 响应，并使用 `is_error` 处理错误。
  </Card>

  <Card title="工具参考" icon="book" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    Anthropic 提供的工具目录以及可选工具定义属性的参考。
  </Card>
</CardGroup>

## 可用性

细粒度工具流式传输在所有模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目 | 零数据保留                                                                                                                                     | HIPAA |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态 | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格  |

适用的端点：

* `/v1/messages`
