---
title: 定义工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools
description: 指定工具模式、编写有效的描述，并控制 Claude 何时调用您的工具。
featureMetadata:
  status: stable
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
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

**API 支持**：用法因产品而异。

* **Messages API**：在请求的 `tools` 中定义它们，并以 `tool_result` 块发送结果。
* **Managed Agents**：在智能体上以 `type` `"custom"` 声明它们；调用以 `agent.custom_tool_use` 事件送达。

## 前提条件

* 熟悉[工具使用概述](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)
* 一个 Claude API 密钥以及可用的 SDK 或 cURL 环境

<Tip>
  如果将 Claude 与工具使用和思考功能结合使用，请参阅[思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)了解更多信息。
</Tip>

## 指定客户端工具

“Client tools”（客户端工具）在 API 请求的 `tools` 顶层参数中指定。Anthropic 模式的客户端工具（例如 bash 和文本编辑器工具）通过带日期版本的 `type` 声明；请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)中链接的各工具页面，了解其接受的字段。计算机使用和浏览器使用工具是[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)：一个没有 `name` 的单一条目，声明一组固定的成员工具。用户定义的工具定义包括：

| 参数               | 描述                                                                                                                                                        |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | 工具的名称。必须匹配正则表达式 `^[a-zA-Z0-9_-]{1,128}$`。                                                                                                                 |
| `description`    | 详细的纯文本描述，说明工具的功能、何时应使用它以及它的行为方式。                                                                                                                          |
| `input_schema`   | 一个 [JSON Schema](https://json-schema.org/) 对象，用于定义工具的预期参数。                                                                                                |
| `input_examples` | （可选）示例输入对象数组，向 Claude 展示如何使用该工具。请参阅[提供工具使用示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#providing-tool-use-examples)。 |

有关任何单个工具定义上可用的完整可选属性集（包括 `cache_control`、`strict`、`defer_loading` 和 `allowed_callers`），请参阅[工具定义属性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#tool-definition-properties)。客户端工具集条目在条目上接受 `cache_control` 和 `allowed_callers`，并为每个成员设置 `defer_loading`；请参阅[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)。

<Accordion title="简单工具定义示例">
  ```json JSON
  {
    "name": "get_weather",
    "description": "Get the current weather in a given location",
    "input_schema": {
      "type": "object",
      "properties": {
        "location": {
          "type": "string",
          "description": "The city and state, e.g. San Francisco, CA"
        },
        "unit": {
          "type": "string",
          "enum": ["celsius", "fahrenheit"],
          "description": "The unit of temperature, either 'celsius' or 'fahrenheit'"
        }
      },
      "required": ["location"]
    }
  }
  ```

  这个名为 `get_weather` 的工具期望一个输入对象，其中包含必需的 `location` 字符串和可选的 `unit` 字符串，后者必须是 `celsius` 或 `fahrenheit`。
</Accordion>

### 工具使用系统提示

当您使用 `tools` 参数调用 Claude API 时，API 会根据工具定义、工具配置以及任何用户指定的系统提示构建一个特殊的 “system prompt”（系统提示）。构建的提示旨在指示模型使用指定的工具，并为工具正常运行提供必要的上下文：

```text wrap
In this environment you have access to a set of tools you can use to answer the user's question.
{{ FORMATTING INSTRUCTIONS }}
String and scalar parameters should be specified as is, while lists and objects should use JSON format. Note that spaces for string values are not stripped. The output is not expected to be valid XML and is parsed with regular expressions.
Here are the functions available in JSONSchema format:
{{ TOOL DEFINITIONS IN JSON SCHEMA }}
{{ USER SYSTEM PROMPT }}
{{ TOOL CONFIGURATION }}
```

### 工具定义的最佳实践

为了在使用工具时让 Claude 发挥最佳性能，请遵循以下准则：

* **提供极其详细的描述。** 这是迄今为止影响工具性能最重要的因素。您的描述应解释工具的每一个细节，包括：

  * 工具的功能
  * 何时应该使用它（以及何时不应该使用）
  * 每个参数的含义及其如何影响工具的行为
  * 任何重要的注意事项或限制，例如在工具名称不够清晰时，说明该工具不返回哪些信息。您为 Claude 提供的关于工具的上下文越多，它就越能判断何时以及如何使用这些工具。每个工具描述至少应有 3–4 句话，如果工具较复杂则应更多。

* **优先考虑描述，但对于复杂工具可考虑使用 `input_examples`。** 清晰的描述最为重要，但对于具有复杂输入、嵌套对象或格式敏感参数的工具，您可以使用 `input_examples` 字段提供经过模式验证的示例。详情请参阅[提供工具使用示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#providing-tool-use-examples)。

* **将相关操作合并为更少的工具。** 与其为每个操作创建单独的工具（`create_pr`、`review_pr`、`merge_pr`），不如将它们组合为一个带有 `action` 参数的工具。更少但功能更强的工具可以减少选择歧义，并使 Claude 更容易浏览您的工具集合。

* **在工具名称中使用有意义的命名空间。** 当您的工具跨越多个服务或资源时，请在名称前加上服务前缀（例如 `github_list_prs`、`slack_send_message`）。随着工具库的增长，这可以使工具选择不产生歧义，在使用[工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)时尤为重要。

* **设计工具响应以仅返回高信号信息。** 返回语义化、稳定的标识符（例如 slug 或 UUID），而不是不透明的内部引用，并且仅包含 Claude 推理下一步所需的字段。臃肿的响应会浪费上下文，并使 Claude 更难提取重要内容。

* **要求提供解释，而不是推理。** 如果某个参数询问 Claude 为何进行此调用，请要求提供简短的解释或支持性证据。要求提供模型思考或逐步推理的参数可能会导致 `reasoning_extraction` 拒绝。请参阅[将推理保留在思考块中](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#keep-reasoning-in-thinking-blocks)。

<AccordionGroup>
  <Accordion title="良好工具描述示例">
    ```json JSON
    {
      "name": "get_stock_price",
      "description": "Retrieves the current stock price for a given ticker symbol. The ticker symbol must be a valid symbol for a publicly traded company on a major US stock exchange like NYSE or NASDAQ. The tool will return the latest trade price in USD. It should be used when the user asks about the current or most recent price of a specific stock. It will not provide any other information about the stock or company.",
      "input_schema": {
        "type": "object",
        "properties": {
          "ticker": {
            "type": "string",
            "description": "The stock ticker symbol, e.g. AAPL for Apple Inc."
          }
        },
        "required": ["ticker"]
      }
    }
    ```
  </Accordion>

  <Accordion title="糟糕工具描述示例">
    ```json JSON
    {
      "name": "get_stock_price",
      "description": "Gets the stock price for a ticker.",
      "input_schema": {
        "type": "object",
        "properties": {
          "ticker": {
            "type": "string"
          }
        },
        "required": ["ticker"]
      }
    }
    ```
  </Accordion>
</AccordionGroup>

良好的描述清楚地解释了工具的功能、何时使用、返回什么数据以及 `ticker` 参数的含义。糟糕的描述过于简短，给 Claude 留下了许多关于工具行为和用法的未解问题。

<Tip>
  有关工具设计（合并、命名和响应塑造）的更深入指导，请参阅[为智能体编写工具](https://www.anthropic.com/engineering/writing-tools-for-agents)。
</Tip>

## 提供工具使用示例

您可以提供有效工具输入的具体示例，向 Claude 展示如何更有效地使用您的工具。这对于具有嵌套对象、可选参数或格式敏感输入的复杂工具特别有用。

### 基本用法

在工具定义中添加一个可选的 `input_examples` 字段，其中包含一组示例输入对象。每个示例都必须符合工具的 `input_schema`：

<CodeGroup>
  ```bash cURL
  curl -sS https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "tools": [
        {
          "name": "get_weather",
          "description": "Get the current weather in a given location",
          "input_schema": {
            "type": "object",
            "properties": {
              "location": {
                "type": "string",
                "description": "The city and state, e.g. San Francisco, CA"
              },
              "unit": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"],
                "description": "The unit of temperature"
              }
            },
            "required": ["location"]
          },
          "input_examples": [
            {"location": "San Francisco, CA", "unit": "fahrenheit"},
            {"location": "Tokyo, Japan", "unit": "celsius"},
            {"location": "New York, NY"}
          ]
        }
      ],
      "messages": [
        {"role": "user", "content": "What is the weather like in San Francisco?"}
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
          unit:
            type: string
            enum: [celsius, fahrenheit]
            description: The unit of temperature
        required: [location]
      input_examples:
        - location: San Francisco, CA
          unit: fahrenheit
        - location: Tokyo, Japan
          unit: celsius
        - location: New York, NY  # 'unit' is optional
  messages:
    - role: user
      content: What's the weather like in San Francisco?
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

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
                      },
                      "unit": {
                          "type": "string",
                          "enum": ["celsius", "fahrenheit"],
                          "description": "The unit of temperature",
                      },
                  },
                  "required": ["location"],
              },
              "input_examples": [
                  {"location": "San Francisco, CA", "unit": "fahrenheit"},
                  {"location": "Tokyo, Japan", "unit": "celsius"},
                  {
                      "location": "New York, NY"  # 'unit' is optional
                  },
              ],
          }
      ],
      messages=[{"role": "user", "content": "What's the weather like in San Francisco?"}],
  )

  print(response)
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
            },
            unit: {
              type: "string",
              enum: ["celsius", "fahrenheit"],
              description: "The unit of temperature"
            }
          },
          required: ["location"]
        },
        input_examples: [
          {
            location: "San Francisco, CA",
            unit: "fahrenheit"
          },
          {
            location: "Tokyo, Japan",
            unit: "celsius"
          },
          {
            location: "New York, NY"
            // 演示 'unit' 为可选项
          }
        ]
      }
    ],
    messages: [{ role: "user", content: "What's the weather like in San Francisco?" }]
  });

  console.log(response);
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
                      ["unit"] = JsonSerializer.SerializeToElement(new { type = "string", @enum = new[] { "celsius", "fahrenheit" }, description = "The unit of temperature" }),
                  },
                  Required = ["location"],
              },
              InputExamples =
              [
                  new Dictionary<string, JsonElement>()
                  {
                      { "location", JsonSerializer.SerializeToElement("San Francisco, CA") },
                      { "unit", JsonSerializer.SerializeToElement("fahrenheit") },
                  },
                  new Dictionary<string, JsonElement>()
                  {
                      { "location", JsonSerializer.SerializeToElement("Tokyo, Japan") },
                      { "unit", JsonSerializer.SerializeToElement("celsius") },
                  },
                  new Dictionary<string, JsonElement>()
                  {
                      { "location", JsonSerializer.SerializeToElement("New York, NY") },
                  },
              ],
          }),
      ],
      Messages = [new UserMessageParam { Content = "What's the weather like in San Francisco?" }]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
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
  					"unit": map[string]any{
  						"type":        "string",
  						"enum":        []string{"celsius", "fahrenheit"},
  						"description": "The unit of temperature",
  					},
  				},
  				Required: []string{"location"},
  			},
  			InputExamples: []map[string]any{
  				{
  					"location": "San Francisco, CA",
  					"unit":     "fahrenheit",
  				},
  				{
  					"location": "Tokyo, Japan",
  					"unit":     "celsius",
  				},
  				{
  					"location": "New York, NY",
  					// 演示 'unit' 是可选的
  				},
  			},
  		}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather like in San Francisco?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.Tool;
  import com.anthropic.models.messages.Tool.InputSchema;
  // ...
  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(Tool.builder()
              .name("get_weather")
              .description("Get the current weather in a given location")
              .inputSchema(InputSchema.builder()
                  .properties(JsonValue.from(Map.of(
                      "location", Map.of(
                          "type", "string",
                          "description", "The city and state, e.g. San Francisco, CA"
                      ),
                      "unit", Map.of(
                          "type", "string",
                          "enum", List.of("celsius", "fahrenheit"),
                          "description", "The unit of temperature"
                      )
                  )))
                  .required(List.of("location"))
                  .build())
              .putAdditionalProperty("input_examples", JsonValue.from(List.of(
                  Map.of(
                      "location", "San Francisco, CA",
                      "unit", "fahrenheit"
                  ),
                  Map.of(
                      "location", "Tokyo, Japan",
                      "unit", "celsius"
                  ),
                  Map.of(
                      "location", "New York, NY"
                  )
              )))
              .build())
          .addUserMessage("What's the weather like in San Francisco?")
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => "What's the weather like in San Francisco?"]
      ],
      model: Model::CLAUDE_OPUS_5_5,
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
                      ],
                      'unit' => [
                          'type' => 'string',
                          'enum' => ['celsius', 'fahrenheit'],
                          'description' => 'The unit of temperature'
                      ]
                  ],
                  'required' => ['location']
              ],
              'input_examples' => [
                  [
                      'location' => 'San Francisco, CA',
                      'unit' => 'fahrenheit'
                  ],
                  [
                      'location' => 'Tokyo, Japan',
                      'unit' => 'celsius'
                  ],
                  [
                      'location' => 'New York, NY'
                  ]
              ]
          ]
      ],
  );
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
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
            },
            unit: {
              type: "string",
              enum: ["celsius", "fahrenheit"],
              description: "The unit of temperature"
            }
          },
          required: ["location"]
        },
        input_examples: [
          {
            location: "San Francisco, CA",
            unit: "fahrenheit"
          },
          {
            location: "Tokyo, Japan",
            unit: "celsius"
          },
          {
            location: "New York, NY"
          }
        ]
      }
    ],
    messages: [
      { role: "user", content: "What's the weather like in San Francisco?" }
    ]
  )
  puts message
  ```
</CodeGroup>

示例会与您的工具模式一起包含在提示中，向 Claude 展示格式良好的工具调用的具体模式。这些模式表明何时包含可选参数、使用什么格式以及如何构建复杂输入。

### 要求和限制

* **模式验证** - 每个示例都必须符合工具的 `input_schema`。无效的示例会返回 400 错误。
* **不支持服务器端工具或客户端工具集** - 输入示例适用于用户定义的工具以及除[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)工具集之外的 Anthropic 模式客户端工具，但不适用于网络搜索或代码执行等服务器工具。
* **令牌成本** - 示例会增加提示令牌：简单示例约 20–50 个令牌，复杂嵌套对象约 100–200 个令牌。

## 控制 Claude 的输出

### 强制工具使用

在某些情况下，您可能希望 Claude 使用特定工具来回答用户的问题，即使 Claude 原本会在不调用工具的情况下直接回答。您可以通过在请求的 `tool_choice` 字段中指定工具来实现这一点。

并非所有模型和设置都支持强制工具使用。在不支持的情况下，`tool_choice: {"type": "any"}` 和 `tool_choice: {"type": "tool", "name": "..."}` 会失败，而 `tool_choice: {"type": "auto"}`（默认值）和 `tool_choice: {"type": "none"}` 仍然有效：

| 模型或设置                                                                                                                                                                                                                                               | 限制                                                                                                                        | 替代方案                                                                                                                                                                                                                                                          |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 手动[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)（`thinking: {type: "enabled"}`）                                                                                                                                 | 不支持 `any` 和 `tool`，使用会导致错误                                                                                                | `auto` 或 `none`。[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)本身不会阻止强制工具使用（默认开启思考的模型，例如 Claude Opus 5 和 Claude Haiku 5.5，接受强制工具使用，但强制调用会跳过思考：响应以工具调用开始，且没有 `thinking` 块）；下一行中的模型无论思考设置如何都会拒绝强制工具使用                                |
| Claude Fable 5.2、[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview)、Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.1 和 [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview) | `any` 和 `tool` 会返回 400 错误。请参阅[不支持强制工具使用](https://platform.claude.com/docs/zh-CN/api/errors#forced-tool-use-not-supported) | 使用 `auto` 并结合[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)来保证工具输入符合模式，或者在需要固定 JSON 结构的响应时使用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。提示仍会影响 `auto` 选择哪个工具。同样支持 `none` |

在支持该功能的模型上，高亮显示的行是与标准工具使用请求的唯一区别：

<CodeGroup>
  ```bash cURL
  curl -sS https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5",
      "max_tokens": 1024,
      "tools": [
        {
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
        }
      ],
      "tool_choice": {"type": "tool", "name": "get_weather"},
      "messages": [
        {"role": "user", "content": "What is the weather like in San Francisco?"}
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
    type: tool
    name: get_weather
  messages:
    - role: user
      content: What's the weather like in San Francisco?
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

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
      }
  ]

  response = client.messages.create(
      model="claude-opus-5",
      max_tokens=1024,
      tools=tools,
      tool_choice={"type": "tool", "name": "get_weather"},
      messages=[{"role": "user", "content": "What's the weather like in San Francisco?"}],
  )

  print(response)
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
    tool_choice: { type: "tool", name: "get_weather" },
    messages: [{ role: "user", content: "What's the weather like in San Francisco?" }]
  });

  console.log(response);
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
      ToolChoice = new ToolChoiceTool { Name = "get_weather" },
      Messages = [new UserMessageParam { Content = "What's the weather like in San Francisco?" }]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
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
  	ToolChoice: anthropic.ToolChoiceUnionParam{OfTool: &anthropic.ToolChoiceToolParam{Name: "get_weather"}},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather like in San Francisco?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.Tool;
  import com.anthropic.models.messages.Tool.InputSchema;
  import com.anthropic.models.messages.ToolChoice;
  import com.anthropic.models.messages.ToolChoiceTool;
  // ...
  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5)
          .maxTokens(1024L)
          .addTool(Tool.builder()
              .name("get_weather")
              .description("Get the current weather in a given location")
              .inputSchema(InputSchema.builder()
                  .properties(JsonValue.from(Map.of(
                      "location", Map.of(
                          "type", "string",
                          "description", "The city and state, e.g. San Francisco, CA"
                      )
                  )))
                  .required(List.of("location"))
                  .build())
              .build())
          .toolChoice(ToolChoice.ofTool(ToolChoiceTool.builder()
              .name("get_weather")
              .build()))
          .addUserMessage("What's the weather like in San Francisco?")
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => "What's the weather like in San Francisco?"]
      ],
      model: Model::CLAUDE_OPUS_5,
      toolChoice: ['type' => 'tool', 'name' => 'get_weather'],
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
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
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
    tool_choice: { type: "tool", name: "get_weather" },
    messages: [
      { role: "user", content: "What's the weather like in San Francisco?" }
    ]
  )
  puts message
  ```
</CodeGroup>

使用 `tool_choice` 参数时，有四个可能的选项：

* `auto` 允许 Claude 自行决定是否调用任何提供的工具。这是提供 `tools` 时的默认值。
* `any` 告诉 Claude 必须使用所提供工具中的一个，但不强制使用特定工具。
* `tool` 强制 Claude 始终使用特定工具。
* `none` 阻止 Claude 使用任何工具。这是未提供 `tools` 时的默认值。

<Note>
  使用[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#what-invalidates-the-cache)时，对 `tool_choice` 参数的更改会使已缓存的消息块失效。工具定义和系统提示仍保持缓存，但消息内容必须重新处理。
</Note>

下图说明了每个选项的工作方式：

<Frame>
  ![三种 tool\_choice 选项的示意图：auto（调用工具或返回纯文本响应）、any（调用其中一个工具）和 tool（调用指定名称的工具）](https://platform.claude.com/docs/zh-CN/images/tool_choice.svg)
</Frame>

请注意，当您将 `tool_choice` 设置为 `any` 或 `tool` 时，API 会预填充助手消息以强制使用工具。这意味着模型不会在 `tool_use` 内容块之前输出自然语言响应或解释，即使被明确要求这样做。

测试表明这不会降低性能。如果您希望模型在仍被要求使用特定工具的同时提供自然语言上下文或解释，可以将 `tool_choice` 设为 `{"type": "auto"}`（默认值），并在 `user` 消息中添加明确的指令。例如：`What's the weather like in London? Use the get_weather tool in your response.`

在所有接受强制工具使用的模型上，强制使用工具也会跳过思考：响应以 `tool_use` 块开始，且没有 `thinking` 块。若要让模型在调用工具之前进行思考，请以同样的方式使用 `auto`，并在提示中加入相应的指令。

<Tip>
  **使用严格工具保证工具调用**

  在支持强制工具使用的模型上，将 `tool_choice: {"type": "any"}` 与[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)结合使用，既可保证调用您的某个工具，又可保证工具输入严格遵循您的模式。在工具定义上设置 `strict: true` 以启用模式验证。
</Tip>

### 使用工具时的模型响应

使用工具时，Claude 通常会在调用工具之前对其正在做的事情进行说明，或自然地回应用户。

例如，对于提示"旧金山现在的天气怎么样，那里现在几点？"，Claude 可能会这样回应：

```json JSON
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "I'll help you check the current weather and time in San Francisco."
    },
    {
      "type": "tool_use",
      "id": "toolu_01A09q90qw90lq917835lq9",
      "name": "get_weather",
      "input": { "location": "San Francisco, CA" }
    }
  ]
}
```

这种自然的响应风格有助于用户理解 Claude 正在做什么，并创造更具对话性的交互。您可以通过系统提示以及在提示中提供 `<examples>` 来引导这些响应的风格和内容。

需要注意的是，Claude 在解释其操作时可能会使用各种措辞和方式。您的代码应像对待其他助手生成的文本一样对待这些响应，而不应依赖特定的格式约定。

## 后续步骤

<CardGroup>
  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls" title="处理工具调用">
    解析 `tool_use` 块并格式化 `tool_result` 响应。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner" title="工具运行器（SDK）">
    让 SDK 自动处理智能体循环。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference" title="工具参考">
    Anthropic 提供的工具和可选属性目录。
  </Card>
</CardGroup>

## 可用性

自定义工具在所有模型上可用。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：在请求的 `tools` 中定义它们。Claude 的响应包含 `tool_use` 块，您在下一个请求中以 `tool_result` 块发送结果。

| 状态 | 云平台     |
| -- | ------- |
| 可用 | 所有支持的平台 |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：在智能体上以 `type` `"custom"` 声明它们。调用以 `agent.custom_tool_use` 事件送达，您以 `user.custom_tool_result` 事件进行回复。

| 状态 | 云平台                                                                                                                   |
| -- | --------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |
