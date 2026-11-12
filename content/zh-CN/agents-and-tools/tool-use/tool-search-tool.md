---
title: 工具搜索工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool
description: 让 Claude 搜索您的工具目录并仅加载所需的工具，从而扩展到数百或数千个工具。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 由 Anthropic 运行的服务器端工具；每次调用时在内存中搜索请求中的工具定义，响应后不存储任何内容。
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
    Amazon Bedrock (bedrock-runtime):
      availability: stable
      note: 在 Amazon Bedrock 上，无法通过 Converse API 使用。
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

工具搜索工具让 Claude 能够通过按需发现和加载工具来处理数百或数千个工具。Claude 不会预先将所有工具定义加载到 “context window”（上下文窗口）中，而是搜索您的工具目录（包括工具名称、描述、参数名称和参数描述），并仅加载所需的工具。

随着工具库的增长，预先加载每个工具定义会导致两个问题：

* **上下文膨胀：** 一个典型的多服务器设置（GitHub、Slack、Sentry、Grafana 和 Splunk）在 Claude 开始任何工作之前，仅定义就可能消耗约 55k 个令牌。工具搜索通常可将其减少 85% 以上，仅加载 Claude 针对特定请求所需的 3–5 个工具。
* **工具选择准确性：** 一旦可用工具超过 30–50 个，Claude 选择正确工具的能力就会下降。由于工具搜索仅按需加载一组聚焦的相关工具，即使在数千个工具中，选择准确性也能保持在较高水平。

有关支持工具搜索的模型，请参阅[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#model-compatibility)。

<Tip>
  有关工具搜索所解决的扩展挑战的背景信息，请参阅[高级工具使用](https://www.anthropic.com/engineering/advanced-tool-use)。工具搜索的按需加载也是[有效的上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)中所述的更广泛的即时检索原则的一个实例。
</Tip>

工具搜索作为服务器端工具运行，但您也可以实现自己的客户端工具搜索。详情请参阅[自定义工具搜索实现](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#custom-tool-search-implementation)。

<Note>
  请通过[反馈表单](https://forms.gle/MhcGFFwLxuwnWTkYA)分享您对此功能的反馈。
</Note>

<Warning>
  在 Amazon Bedrock 上，服务器端工具搜索仅通过 [InvokeModel API](https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-runtime_example_bedrock-runtime_InvokeModel_AnthropicClaude_section.html) 提供，不通过 Converse API 提供。
</Warning>

<Note>
  在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上，服务器端工具搜索的工作方式与 Claude API 完全相同。Claude Platform on AWS 直接使用 Anthropic Messages API，因此不存在 InvokeModel 或 Converse 的区别。
</Note>

## 模型兼容性

以下模型均支持两种工具搜索变体：

| 模型                                                                                                                                    | 工具版本                                                                |
| ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Claude Fable 5.2 (claude-fable-5-2)                                                                                                   | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Mythos 5.2 (claude-mythos-5-2)                                                                                                 | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Fable 5.1 (claude-fable-5-1)                                                                                                   | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Mythos 5.1 (claude-mythos-5-1)                                                                                                 | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Fable 5 (claude-fable-5)                                                                                                       | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Mythos 5 (claude-mythos-5)                                                                                                     | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Opus 5.5 (claude-opus-5-5)                                                                                                     | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Opus 5 (claude-opus-5)                                                                                                         | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Sonnet 5.5 (claude-sonnet-5-5)                                                                                                 | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Sonnet 5 (claude-sonnet-5)                                                                                                     | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Haiku 5.5 (claude-haiku-5-5)                                                                                                   | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Opus 4.8 (claude-opus-4-8)                                                                                                     | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Opus 4.7 (claude-opus-4-7)                                                                                                     | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Opus 4.6 (claude-opus-4-6)                                                                                                     | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Sonnet 4.6 (claude-sonnet-4-6)                                                                                                 | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Opus 4.5 (claude-opus-4-5-20251101)                                                                                            | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Sonnet 4.5 (claude-sonnet-4-5-20250929)（已弃用；请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)） | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |
| Claude Haiku 4.5 (claude-haiku-4-5-20251001)                                                                                          | `tool_search_tool_regex_20251119`, `tool_search_tool_bm25_20251119` |

Claude Opus 4.1 及更早的模型不支持工具搜索工具。

## 工具搜索的工作原理

工具搜索有两种变体：

* **Regex**（`tool_search_tool_regex_20251119`）：Claude 构造正则表达式模式来搜索工具。
* **BM25**（`tool_search_tool_bm25_20251119`）：Claude 使用自然语言查询来搜索工具。

启用工具搜索工具后：

1. 您在 `tools` 列表中包含一个工具搜索工具（例如 `tool_search_tool_regex_20251119` 或 `tool_search_tool_bm25_20251119`）。
2. 您在 `tools` 数组中提供每个工具定义，并在不应预先加载的工具上设置 `defer_loading: true`。至少有一个工具（通常是工具搜索工具本身）必须保持非延迟状态。
3. 最初，Claude 的上下文仅包含工具搜索工具和任何非延迟工具。
4. 当 Claude 需要其他工具时，它会使用工具搜索工具进行搜索。
5. API 执行搜索，并以 `tool_reference` 块的形式返回匹配的工具（默认最多 5 个，但 Claude 可以在其搜索输入中设置 `limit`）。
6. API 自动将这些引用展开为完整的工具定义。
7. Claude 从发现的工具中进行选择并调用它们。

## 快速入门

以下示例包含工具搜索工具和两个延迟加载的工具：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 2048,
      "messages": [
        {
          "role": "user",
          "content": "What is the weather in San Francisco?"
        }
      ],
      "tools": [
        {
          "type": "tool_search_tool_regex_20251119",
          "name": "tool_search_tool_regex"
        },
        {
          "name": "get_weather",
          "description": "Get the weather at a specific location",
          "input_schema": {
            "type": "object",
            "properties": {
              "location": {"type": "string"},
              "unit": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"]
              }
            },
            "required": ["location"]
          },
          "defer_loading": true
        },
        {
          "name": "search_files",
          "description": "Search through files in the workspace",
          "input_schema": {
            "type": "object",
            "properties": {
              "query": {"type": "string"},
              "file_types": {
                "type": "array",
                "items": {"type": "string"}
              }
            },
            "required": ["query"]
          },
          "defer_loading": true
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 2048
  messages:
    - role: user
      content: What is the weather in San Francisco?
  tools:
    - type: tool_search_tool_regex_20251119
      name: tool_search_tool_regex
    - name: get_weather
      description: Get the weather at a specific location
      input_schema:
        type: object
        properties:
          location:
            type: string
          unit:
            type: string
            enum: [celsius, fahrenheit]
        required: [location]
      defer_loading: true
    - name: search_files
      description: Search through files in the workspace
      input_schema:
        type: object
        properties:
          query:
            type: string
          file_types:
            type: array
            items:
              type: string
        required: [query]
      defer_loading: true
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=2048,
      messages=[{"role": "user", "content": "What is the weather in San Francisco?"}],
      tools=[
          {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},
          {
              "name": "get_weather",
              "description": "Get the weather at a specific location",
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "location": {"type": "string"},
                      "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                  },
                  "required": ["location"],
              },
              "defer_loading": True,
          },
          {
              "name": "search_files",
              "description": "Search through files in the workspace",
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "query": {"type": "string"},
                      "file_types": {"type": "array", "items": {"type": "string"}},
                  },
                  "required": ["query"],
              },
              "defer_loading": True,
          },
      ],
  )

  print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 2048,
    messages: [
      {
        role: "user",
        content: "What is the weather in San Francisco?"
      }
    ],
    tools: [
      {
        type: "tool_search_tool_regex_20251119",
        name: "tool_search_tool_regex"
      },
      {
        name: "get_weather",
        description: "Get the weather at a specific location",
        input_schema: {
          type: "object" as const,
          properties: {
            location: { type: "string" },
            unit: {
              type: "string",
              enum: ["celsius", "fahrenheit"]
            }
          },
          required: ["location"]
        },
        defer_loading: true
      },
      {
        name: "search_files",
        description: "Search through files in the workspace",
        input_schema: {
          type: "object" as const,
          properties: {
            query: { type: "string" },
            file_types: {
              type: "array",
              items: { type: "string" }
            }
          },
          required: ["query"]
        },
        defer_loading: true
      }
    ]
  });

  console.log(response);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 2048,
      Messages = [new UserMessageParam { Content = "What is the weather in San Francisco?" }],
      Tools = [
          new ToolUnion(new ToolSearchToolRegex20251119
          {
              Type = ToolSearchToolRegex20251119Type.ToolSearchToolRegex20251119
          }),
          new ToolUnion(new Tool()
          {
              Name = "get_weather",
              Description = "Get the weather at a specific location",
              InputSchema = new InputSchema()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["location"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                      ["unit"] = JsonSerializer.SerializeToElement(new { type = "string", @enum = new[] { "celsius", "fahrenheit" } }),
                  },
                  Required = ["location"],
              },
              DeferLoading = true,
          }),
          new ToolUnion(new Tool()
          {
              Name = "search_files",
              Description = "Search through files in the workspace",
              InputSchema = new InputSchema()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["query"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                      ["file_types"] = JsonSerializer.SerializeToElement(new { type = "array", items = new { type = "string" } }),
                  },
                  Required = ["query"],
              },
              DeferLoading = true,
          }),
      ]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 2048,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What is the weather in San Francisco?")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfToolSearchToolRegex20251119: &anthropic.ToolSearchToolRegex20251119Param{
  			Type: anthropic.ToolSearchToolRegex20251119TypeToolSearchToolRegex20251119,
  		}},
  		{OfTool: &anthropic.ToolParam{
  			Name:        "get_weather",
  			Description: anthropic.String("Get the weather at a specific location"),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"location": map[string]any{"type": "string"},
  					"unit": map[string]any{
  						"type": "string",
  						"enum": []string{"celsius", "fahrenheit"},
  					},
  				},
  				Required: []string{"location"},
  			},
  			DeferLoading: anthropic.Bool(true),
  		}},
  		{OfTool: &anthropic.ToolParam{
  			Name:        "search_files",
  			Description: anthropic.String("Search through files in the workspace"),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"query":      map[string]any{"type": "string"},
  					"file_types": map[string]any{"type": "array", "items": map[string]any{"type": "string"}},
  				},
  				Required: []string{"query"},
  			},
  			DeferLoading: anthropic.Bool(true),
  		}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.ToolSearchToolRegex20251119;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      InputSchema weatherSchema = InputSchema.builder()
          .properties(JsonValue.from(Map.of(
              "location", Map.of("type", "string"),
              "unit", Map.of(
                  "type", "string",
                  "enum", List.of("celsius", "fahrenheit")
              )
          )))
          .putAdditionalProperty("required", JsonValue.from(List.of("location")))
          .build();

      InputSchema searchSchema = InputSchema.builder()
          .properties(JsonValue.from(Map.of(
              "query", Map.of("type", "string"),
              "file_types", Map.of(
                  "type", "array",
                  "items", Map.of("type", "string")
              )
          )))
          .putAdditionalProperty("required", JsonValue.from(List.of("query")))
          .build();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(2048L)
          .addUserMessage("What is the weather in San Francisco?")
          .addTool(ToolSearchToolRegex20251119.builder()
              .type(ToolSearchToolRegex20251119.Type.TOOL_SEARCH_TOOL_REGEX_20251119)
              .build())
          .addTool(Tool.builder()
              .name("get_weather")
              .description("Get the weather at a specific location")
              .inputSchema(weatherSchema)
              .deferLoading(true)
              .build())
          .addTool(Tool.builder()
              .name("search_files")
              .description("Search through files in the workspace")
              .inputSchema(searchSchema)
              .deferLoading(true)
              .build())
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 2048,
      messages: [
          ['role' => 'user', 'content' => 'What is the weather in San Francisco?'],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          [
              'type' => 'tool_search_tool_regex_20251119',
              'name' => 'tool_search_tool_regex',
          ],
          [
              'name' => 'get_weather',
              'description' => 'Get the weather at a specific location',
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'location' => ['type' => 'string'],
                      'unit' => [
                          'type' => 'string',
                          'enum' => ['celsius', 'fahrenheit'],
                      ],
                  ],
                  'required' => ['location'],
              ],
              'defer_loading' => true,
          ],
          [
              'name' => 'search_files',
              'description' => 'Search through files in the workspace',
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'query' => ['type' => 'string'],
                      'file_types' => [
                          'type' => 'array',
                          'items' => ['type' => 'string'],
                      ],
                  ],
                  'required' => ['query'],
              ],
              'defer_loading' => true,
          ],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 2048,
    messages: [
      { role: "user", content: "What is the weather in San Francisco?" }
    ],
    tools: [
      {
        type: "tool_search_tool_regex_20251119",
        name: "tool_search_tool_regex"
      },
      {
        name: "get_weather",
        description: "Get the weather at a specific location",
        input_schema: {
          type: "object",
          properties: {
            location: { type: "string" },
            unit: {
              type: "string",
              enum: ["celsius", "fahrenheit"]
            }
          },
          required: ["location"]
        },
        defer_loading: true
      },
      {
        name: "search_files",
        description: "Search through files in the workspace",
        input_schema: {
          type: "object",
          properties: {
            query: { type: "string" },
            file_types: {
              type: "array",
              items: { type: "string" }
            }
          },
          required: ["query"]
        },
        defer_loading: true
      }
    ]
  )

  puts message
  ```
</CodeGroup>

Claude 搜索目录，发现 `get_weather` 并调用它。响应以 `stop_reason: "tool_use"` 结束。按照[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)中的方式运行发现的工具并返回 `tool_result`。[响应格式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#response-format)展示了您收到的块以及接下来要发送的内容。

## 工具定义

工具搜索工具有两种变体：

```json JSON
{
  "type": "tool_search_tool_regex_20251119",
  "name": "tool_search_tool_regex"
}
```

```json JSON
{
  "type": "tool_search_tool_bm25_20251119",
  "name": "tool_search_tool_bm25"
}
```

<Warning>
  **Regex 变体查询格式：Python 正则表达式，而非自然语言**

  使用 `tool_search_tool_regex_20251119` 时，Claude 编写的是 Python `re.search()` 模式，而非自然语言查询。匹配不区分大小写。常见模式包括：

  * `"weather"`：匹配名称和描述中包含 “weather” 的工具
  * `"get_.*_data"`：匹配 `get_user_data` 和 `get_weather_data` 等工具
  * `"database.*query|query.*database"`：匹配任一词序

  最大模式长度：200 个字符
</Warning>

<Note>
  **BM25 变体查询格式：自然语言**

  使用 `tool_search_tool_bm25_20251119` 时，Claude 使用自然语言查询进行搜索。最大查询长度：500 个字符。
</Note>

### 延迟工具加载

通过添加 `defer_loading: true` 将工具标记为按需加载：

```json JSON
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": { "type": "string" },
      "unit": { "type": "string", "enum": ["celsius", "fahrenheit"] }
    },
    "required": ["location"]
  },
  "defer_loading": true
}
```

`defer_loading` 控制的是进入上下文窗口的内容，而不是您在请求中发送的内容：

* 您仍需在每个请求的 `tools` 数组中发送每个工具的完整定义，包括延迟加载的工具。API 需要在服务器端使用它们来执行搜索并展开 `tool_reference` 块。
* 没有 `defer_loading` 的工具会立即加载到上下文中。
* 带有 `defer_loading: true` 的工具仅在 Claude 通过搜索发现它们时才会加载。
* 切勿在工具搜索工具本身上设置 `defer_loading: true`。
* 将您最常用的 3–5 个工具保持为非延迟状态，以便 Claude 无需先搜索即可调用它们。

计算机使用和浏览器使用工具集（`computer_toolset_20260801` 和 `browser_toolset_20260801`）在条目的 `configs` 对象中为每个成员工具设置 `defer_loading`，而不是在条目本身上设置；在条目级别设置它的请求会被拒绝。由于工具集作为一个整体进行延迟和展开，`defer_loading` 在每个已启用的成员上必须解析为相同的值，并且当 Claude 通过搜索发现该工具集时，所有已启用的成员会同时加载。有关 `configs` 格式，请参阅[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)。

两种工具搜索变体（`regex` 和 `bm25`）都会搜索工具名称、描述、参数名称和参数描述。

在内部，API 会将延迟工具排除在系统提示前缀之外。当 Claude 通过工具搜索发现一个延迟工具时，API 会在对话中内联追加一个 `tool_reference` 块，然后在将其传递给 Claude 之前将其展开为完整的工具定义。[严格模式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)的语法（约束工具调用输出以匹配您的模式的规则）基于完整的工具列表构建，因此将 `defer_loading` 与严格模式一起使用不会导致 API 重新编译语法。

## 响应格式

当 Claude 使用工具搜索工具时，响应包含以下块类型：

```json JSON
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "I'll search for tools to help with the weather information."
    },
    {
      "type": "server_tool_use",
      "id": "srvtoolu_01ABC123",
      "name": "tool_search_tool_regex",
      "input": {
        "pattern": "weather",
        "limit": 10
      }
    },
    {
      "type": "tool_search_tool_result",
      "tool_use_id": "srvtoolu_01ABC123",
      "content": {
        "type": "tool_search_tool_search_result",
        "tool_references": [{ "type": "tool_reference", "tool_name": "get_weather" }]
      }
    },
    {
      "type": "text",
      "text": "I found a weather tool. Let me get the weather for San Francisco."
    },
    {
      "type": "tool_use",
      "id": "toolu_01XYZ789",
      "name": "get_weather",
      "input": { "location": "San Francisco", "unit": "fahrenheit" }
    }
  ],
  "stop_reason": "tool_use"
}
```

### 理解响应

* **`server_tool_use`：** Claude 对工具搜索工具的调用。搜索在 Anthropic 的服务器上运行。切勿为其 `srvtoolu_...` ID 返回 `tool_result`。`input` 包含搜索内容（regex 变体为 `pattern`，BM25 为 `query`），并可能包含可选的 `limit`，这是一个 1 到 10,000 之间的整数，用于限制搜索返回的匹配工具数量（默认值：5）。
* **`tool_search_tool_result`：** 搜索结果，位于嵌套的 `tool_search_tool_search_result` 对象中。请将其原样保留在消息历史中。
* **`tool_references`：** 指向已发现工具的 `tool_reference` 对象数组。API 会为 Claude 展开这些引用。您永远不需要自己展开它们。
* **`tool_use`：** Claude 对已发现工具的调用。运行它并返回 `tool_result`，方式与标准 “tool use”（工具使用）完全相同。

API 会在将 `tool_reference` 块展示给 Claude 之前自动将其展开为完整的工具定义。只要您在 `tools` 参数中提供了所有匹配的工具定义，就无需自己处理此展开过程。

### 继续对话

在下一个请求中，原样传回助手的内容，包括 `server_tool_use` 和 `tool_search_tool_result` 块。在用户消息中添加已发现工具的 `tool_result`，并发送相同的 `tools` 数组：搜索工具加上每个延迟定义。不要为 `srvtoolu_...` ID 返回 `tool_result`：API 会拒绝该请求。API 会在整个对话历史中展开 `tool_reference` 块，因此 Claude 可以在后续轮次中重复使用已发现的工具，而无需重新搜索。没有匹配结果的搜索会返回一个带有空 `tool_references` 数组的 `tool_search_tool_search_result`，而不是错误。

## MCP 集成

如果您的工具通过 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)来自 MCP 服务器，则无需在各个工具定义上设置 `defer_loading`。相反，您可以在 `mcp_toolset` 条目的 `default_config` 中为整个服务器设置一次，或在其 `configs` 中按工具设置。请参阅 [MCP 工具集配置](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#mcp-toolset-configuration)。

## 自定义工具搜索实现

您可以通过从自定义工具返回 `tool_reference` 块来实现自己的工具搜索逻辑（例如，使用嵌入或语义搜索）。当 Claude 调用您的自定义搜索工具时，在内容数组中返回带有 `tool_reference` 块的标准 `tool_result`：

```json JSON
{
  "type": "tool_result",
  "tool_use_id": "toolu_your_tool_id",
  "content": [{ "type": "tool_reference", "tool_name": "discovered_tool_name" }]
}
```

每个被引用的工具都必须在顶层 `tools` 参数中有对应的工具定义，通常带有 `defer_loading: true`。这让您可以使用内置变体未提供的搜索方法，例如基于嵌入的检索，API 会以相同的方式展开返回的 `tool_reference` 块。

<Note>
  [响应格式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#response-format)部分所示的 `tool_search_tool_result` 格式是 Anthropic 内置工具搜索在内部使用的服务器端格式。对于自定义客户端实现，请始终使用带有 `tool_reference` 内容块的标准 `tool_result` 格式，如前面的示例所示。
</Note>

有关使用嵌入的完整示例，请参阅[使用嵌入的工具搜索](https://platform.claude.com/cookbook/tool-use-tool-search-with-embeddings)示例。

## 错误处理

<Note>
  [工具使用示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#providing-tool-use-examples) 可与工具搜索配合使用：当 Claude 发现一个延迟工具时，API 会将其 `input_examples` 与其定义一起展开。
</Note>

### HTTP 错误（400 状态）

这些错误会阻止 API 处理请求：

**所有工具均被延迟：**

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "At least one tool must have defer_loading=false. All tools cannot be deferred."
  }
}
```

**缺少工具定义：**

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Tool reference 'unknown_tool' not found in available tools"
  }
}
```

### 工具结果错误（200 状态）

当工具搜索操作在执行期间失败时，API 会返回 200 响应，并在正文中包含错误：

```json JSON
{
  "type": "tool_search_tool_result",
  "tool_use_id": "srvtoolu_01ABC123",
  "content": {
    "type": "tool_search_tool_result_error",
    "error_code": "invalid_tool_input",
    "error_message": "Invalid regular expression pattern: missing ) at position 1"
  }
}
```

API 定义了以下 `error_code` 值：

* `invalid_tool_input`：搜索输入无效，例如正则表达式模式格式错误或模式超过 200 个字符的限制
* `unavailable`：搜索无法运行，例如因为超时或服务不可用
* `too_many_requests`：超出工具搜索操作的 “rate limit”（速率限制）
* `execution_time_exceeded`：工具搜索不会返回此错误，它会将超时报告为 `unavailable`

### 常见错误

<Accordion title="400 错误：所有工具均被延迟">
  **原因：** 您在每个工具上都设置了 `defer_loading: true`，包括工具搜索工具。

  **修复：** 从工具搜索工具中移除 `defer_loading`：

  ```json
  {
    "type": "tool_search_tool_regex_20251119",
    "name": "tool_search_tool_regex"
  }
  ```
</Accordion>

<Accordion title="400 错误：缺少工具定义">
  **原因：** 某个 `tool_reference` 指向的工具不在您的 `tools` 数组中。

  **修复：** 确保每个可能被发现的工具都有完整的定义：

  ```json
  {
    "name": "my_tool",
    "description": "Full description here",
    "input_schema": {
      "type": "object"
    },
    "defer_loading": true
  }
  ```
</Accordion>

<Accordion title="Claude 找不到预期的工具">
  **原因：** 正则表达式模式与工具的名称、描述、参数名称或参数描述不匹配。

  **调试步骤：**

  1. 检查工具名称、描述、参数名称和参数描述。Claude 会搜索所有这些字段。
  2. 测试您的模式：`import re; re.search(r"your_pattern", "tool_name", re.IGNORECASE)`。
  3. 匹配不区分大小写，因此大小写差异不是问题所在。
  4. Claude 使用诸如 `".*weather.*"` 之类的宽泛模式，而不是精确匹配。

  **提示：** 在工具描述中添加常用关键词以提高可发现性。
</Accordion>

## 提示缓存

要了解 `defer_loading` 如何保留 “prompt caching”（提示缓存），请参阅[工具使用与提示缓存](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching)。

带有 `defer_loading: true` 的工具不能同时带有 `cache_control`：API 会返回 400。请将缓存断点放在非延迟工具上。

## 流式传输

启用 “streaming”（流式传输）后，您将在流中收到工具搜索事件：

```sse
event: content_block_start
data: {"type": "content_block_start", "index": 1, "content_block": {"type": "server_tool_use", "id": "srvtoolu_xyz789", "name": "tool_search_tool_regex"}}

// Search pattern streamed
event: content_block_delta
data: {"type": "content_block_delta", "index": 1, "delta": {"type": "input_json_delta", "partial_json": "{\"pattern\":\"weather\"}"}}

// Pause while search runs

// Search results streamed
event: content_block_start
data: {"type": "content_block_start", "index": 2, "content_block": {"type": "tool_search_tool_result", "tool_use_id": "srvtoolu_xyz789", "content": {"type": "tool_search_tool_search_result", "tool_references": [{"type": "tool_reference", "tool_name": "get_weather"}]}}}

// Claude continues with discovered tools
```

## 批量请求

您可以在 [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 中包含工具搜索工具。

## 限制和最佳实践

### 限制

* **最大延迟工具数：** 每个请求最多 10,000 个带有 `defer_loading: true` 的工具
* **搜索结果：** 每次搜索默认最多返回 5 个匹配的工具；Claude 可以在其搜索输入中将 `limit` 设置为 1 到 10,000 之间的任意整数
* **模式和查询长度：** 正则表达式模式最多 200 个字符，BM25 查询最多 500 个字符
* **模型支持：** 请参阅[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#model-compatibility)

### 何时使用工具搜索

当以下任一情况适用时，请使用工具搜索：

* 您有 10 个或更多可用工具。
* 您的工具定义消耗超过 10k 个令牌。
* 随着您添加更多工具，工具选择准确性下降。
* 您聚合了多个 MCP 服务器（200 个以上工具）。
* 您的工具库随时间增长。

当您的工具少于 10 个、每个请求都使用每个工具，或者您的工具定义很小（总计少于 100 个令牌）时，不使用工具搜索的标准工具调用更为合适。

### 优化技巧

* 将您最常用的 3–5 个工具保持为非延迟状态。
* 编写清晰、具有描述性的工具名称和描述。
* 在工具名称中使用一致的命名空间：按服务或资源添加前缀（例如 `github_`、`slack_`），以便一次搜索即可匹配整个组。
* 在描述中使用与用户描述任务方式相匹配的关键词。
* 添加一个描述可用工具类别的 “system prompt”（系统提示）部分："You can search for tools to interact with Slack, GitHub, and Jira."
* 监控 Claude 发现了哪些工具，以完善您的描述。

## 用量

工具搜索不会作为单独的服务器工具计量。响应的 `usage.server_tool_use` 对象中没有工具搜索字段，搜索加载到上下文中的工具定义与任何其他工具定义一样计为输入令牌。

## 后续步骤

<CardGroup cols={2}>
  <Card title="记忆工具" icon="brain" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool">
    通过在您的应用程序中实现记忆工具的文件操作，让 Claude 能够跨对话存储和检索信息。
  </Card>

  <Card title="工具参考" icon="book" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    Anthropic 提供的工具目录以及可选工具定义属性的参考。
  </Card>

  <Card title="MCP 连接器" icon="link" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector">
    配置带有延迟加载的 MCP 工具集。
  </Card>

  <Card title="工具使用与提示缓存" icon="bolt" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching">
    跨轮次缓存工具定义，并了解哪些因素会使您的缓存失效。
  </Card>

  <Card title="定义工具" icon="hammer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools">
    指定工具模式，编写有效的描述，并控制 Claude 何时调用您的工具。
  </Card>
</CardGroup>

## 可用性

工具搜索工具在所有模型上可用，并具备零数据保留资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态             | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | 模型                                                             |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 可用             | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |
| 可用<sup>1</sup> | [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)                                                                                                                                                                                                                                                                                                                                                                   | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

1. 在 Amazon Bedrock 上，无法通过 Converse API 使用。

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 不具备资格 |
| 详细信息 | 由 Anthropic 运行的服务器端工具；每次调用时在内存中搜索请求中的工具定义，响应后不存储任何内容。                                                                                     | —     |

适用的端点：

* `/v1/messages` 使用 `tool_search` 工具
