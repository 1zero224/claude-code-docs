---
title: 严格工具使用
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use
description: 通过语法约束采样，强制 Claude 的工具输入符合 JSON Schema。
featureMetadata:
  status: stable
  zdr:
    eligibility: qualified
    note: "Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 您的提示和 Claude 的输出不会被存储。仅缓存 JSON schema，自上次使用起最长保留 24 小时。这也涵盖严格工具使用（工具上的 `strict: true`），它使用相同的语法管道。"
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

在工具定义上设置 `strict: true`，可以通过将模型的令牌采样约束为符合 schema 的输出（一种称为"grammar-constrained sampling"（语法约束采样）的技术），保证 Claude 的工具输入与您的 JSON Schema 匹配。本页介绍严格模式对智能体为何重要、如何启用它以及常见用例。有关受支持的 JSON Schema 子集，请参阅 [JSON Schema 限制](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#json-schema-limitations)。有关非严格 schema 的指导，请参阅[定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)。

严格工具使用会验证工具参数，确保 Claude 使用类型正确的参数调用您的函数。在以下情况下使用严格工具使用：

* 验证工具参数
* 构建智能体工作流
* 确保类型安全的函数调用
* 处理具有嵌套属性的复杂工具

## 为什么严格工具使用对智能体很重要

构建可靠的智能体系统需要保证 schema 一致性。如果没有严格模式，Claude 可能会返回不兼容的类型（`"2"` 而不是 `2`）或遗漏必填字段，从而破坏您的函数并导致运行时错误。

严格工具使用保证类型安全的参数：

* 每次都获得类型正确的函数参数
* 无需验证和重试工具调用
* 可在规模化场景下稳定运行的生产就绪智能体

例如，假设一个预订系统需要 `passengers: int`。如果没有严格模式，Claude 可能会提供 `passengers: "two"` 或 `passengers: "2"`。使用 `strict: true` 后，响应始终包含 `passengers: 2`。

## 快速开始

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "What is the weather in San Francisco?"}
      ],
      "tools": [{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "strict": true,
        "input_schema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g. San Francisco, CA"
            },
            "unit": {
              "type": "string",
              "enum": ["celsius", "fahrenheit"]
            }
          },
          "required": ["location"],
          "additionalProperties": false
        }
      }]
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
      strict: true
      input_schema:
        type: object
        properties:
          location:
            type: string
            description: The city and state, e.g. San Francisco, CA
          unit:
            type: string
            enum: [celsius, fahrenheit]
        required: [location]
        additionalProperties: false
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "What's the weather like in San Francisco?"}],
      tools=[
          {
              "name": "get_weather",
              "description": "Get the current weather in a given location",
              "strict": True,  # Enable strict mode
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
                          "description": "The unit of temperature, either 'celsius' or 'fahrenheit'",
                      },
                  },
                  "required": ["location"],
                  "additionalProperties": False,
              },
          }
      ],
  )
  print(response.content)
  ```

  ```typescript TypeScript
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: "What's the weather like in San Francisco?"
      }
    ],
    tools: [
      {
        name: "get_weather",
        description: "Get the current weather in a given location",
        strict: true, // Enable strict mode
        input_schema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "The city and state, e.g. San Francisco, CA"
            },
            unit: {
              type: "string",
              enum: ["celsius", "fahrenheit"]
            }
          },
          required: ["location"],
          additionalProperties: false
        }
      }
    ]
  });
  console.log(response.content);
  ```

  ```csharp C#
  using System.Text.Json;
  using Anthropic;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "What's the weather like in San Francisco?" }],
      Tools = [
          new ToolUnion(new Tool()
          {
              Name = "get_weather",
              Description = "Get the current weather in a given location",
              Strict = true,
              InputSchema = new InputSchema(new Dictionary<string, JsonElement>
              {
                  ["properties"] = JsonSerializer.SerializeToElement(new Dictionary<string, object>
                  {
                      ["location"] = new { type = "string", description = "The city and state, e.g. San Francisco, CA" },
                      ["unit"] = new { type = "string", @enum = new[] { "celsius", "fahrenheit" } },
                  }),
                  ["required"] = JsonSerializer.SerializeToElement(new[] { "location" }),
                  ["additionalProperties"] = JsonSerializer.SerializeToElement(false),
              }),
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
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather like in San Francisco?")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfTool: &anthropic.ToolParam{
  			Name:        "get_weather",
  			Description: anthropic.String("Get the current weather in a given location"),
  			Strict:      anthropic.Bool(true),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"location": map[string]any{
  						"type":        "string",
  						"description": "The city and state, e.g. San Francisco, CA",
  					},
  					"unit": map[string]any{
  						"type": "string",
  						"enum": []string{"celsius", "fahrenheit"},
  					},
  				},
  				Required: []string{"location"},
  				ExtraFields: map[string]any{
  					"additionalProperties": false,
  				},
  			}}},
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
                  ),
                  "unit", Map.of(
                      "type", "string",
                      "enum", List.of("celsius", "fahrenheit")
                  )
              )
          )
      )
      .putAdditionalProperty("required", JsonValue.from(List.of("location")))
      .putAdditionalProperty("additionalProperties", JsonValue.from(false))
      .build();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      .addUserMessage("What's the weather like in San Francisco?")
      .addTool(
          Tool.builder()
              .name("get_weather")
              .description("Get the current weather in a given location")
              .strict(true)
              .inputSchema(schema)
              .build()
      )
      .build();

  Message response = client.messages().create(params);
  IO.println(response.content());
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => "What's the weather like in San Francisco?"]
      ],
      model: 'claude-opus-5-5',
      tools: [
          [
              'name' => 'get_weather',
              'description' => 'Get the current weather in a given location',
              'strict' => true,
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'location' => [
                          'type' => 'string',
                          'description' => 'The city and state, e.g. San Francisco, CA'
                      ],
                      'unit' => [
                          'type' => 'string',
                          'enum' => ['celsius', 'fahrenheit']
                      ]
                  ],
                  'required' => ['location'],
                  'additionalProperties' => false
              ]
          ]
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [
      { role: "user", content: "What's the weather like in San Francisco?" }
    ],
    tools: [
      {
        name: "get_weather",
        description: "Get the current weather in a given location",
        strict: true,
        input_schema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "The city and state, e.g. San Francisco, CA"
            },
            unit: {
              type: "string",
              enum: ["celsius", "fahrenheit"]
            }
          },
          required: ["location"],
          additionalProperties: false
        }
      }
    ]
  )
  puts message.content
  ```
</CodeGroup>

**响应格式：** 工具使用块，其经过验证的输入位于 `response.content[x].input` 中

```json Output
{
  "type": "tool_use",
  "name": "get_weather",
  "input": {
    "location": "San Francisco, CA"
  }
}
```

**保证：**

* 工具 `input` 严格遵循 `input_schema`
* 工具 `name` 始终有效（来自所提供的工具或服务器工具）

## 工作原理

<Steps>
  <Step title="定义您的工具 schema">
    为工具的 `input_schema` 创建 JSON schema。该 schema 使用标准 JSON Schema 格式，但存在一些限制（请参阅 [JSON Schema 限制](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#json-schema-limitations)）。
  </Step>

  <Step title="添加 strict: true">
    在工具定义中将 `"strict": true` 设置为顶层属性，与 `name`、`description` 和 `input_schema` 并列。
  </Step>

  <Step title="处理工具调用">
    当 Claude 使用该工具时，`tool_use` 块中的 `input` 字段严格遵循您的 `input_schema`，并且 `name` 始终有效。
  </Step>
</Steps>

[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)工具集条目（`computer_toolset_20260801` 和 `browser_toolset_20260801`）不接受 `strict: true`；在任一条目上设置该属性的请求都会被拒绝。

## 常见用例

<AccordionGroup>
  <Accordion title="经过验证的工具输入">
    确保工具参数与您的 schema 完全匹配：

    <CodeGroup>
      ```bash cURL
      curl https://api.anthropic.com/v1/messages \
        -H "content-type: application/json" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -d '{
          "model": "claude-opus-5-5",
          "max_tokens": 1024,
          "messages": [
            {"role": "user", "content": "Search for flights to Tokyo departing June 1, 2026"}
          ],
          "tools": [{
            "name": "search_flights",
            "strict": true,
            "input_schema": {
              "type": "object",
              "properties": {
                "destination": {"type": "string"},
                "departure_date": {"type": "string", "format": "date"},
                "passengers": {"type": "integer", "enum": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]}
              },
              "required": ["destination", "departure_date"],
              "additionalProperties": false
            }
          }]
        }'
      ```

      ```bash CLI
      ant messages create <<'YAML'
      model: claude-opus-5-5
      max_tokens: 1024
      messages:
        - role: user
          content: Search for flights to Tokyo departing June 1, 2026
      tools:
        - name: search_flights
          strict: true
          input_schema:
            type: object
            properties:
              destination:
                type: string
              departure_date:
                type: string
                format: date
              passengers:
                type: integer
                enum: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
            required: [destination, departure_date]
            additionalProperties: false
      YAML
      ```

      ```python Python
      client = Anthropic()
      response = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[
              {
                  "role": "user",
                  "content": "Search for flights to Tokyo departing June 1, 2026",
              }
          ],
          tools=[
              {
                  "name": "search_flights",
                  "strict": True,
                  "input_schema": {
                      "type": "object",
                      "properties": {
                          "destination": {"type": "string"},
                          "departure_date": {"type": "string", "format": "date"},
                          "passengers": {
                              "type": "integer",
                              "enum": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
                          },
                      },
                      "required": ["destination", "departure_date"],
                      "additionalProperties": False,
                  },
              }
          ],
      )

      print(response)
      ```

      ```typescript TypeScript
      const client = new Anthropic();

      const searchFlightsTool: Anthropic.Tool = {
        name: "search_flights",
        strict: true,
        input_schema: {
          type: "object",
          properties: {
            destination: { type: "string" },
            departure_date: { type: "string", format: "date" },
            passengers: { type: "integer", enum: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] }
          },
          required: ["destination", "departure_date"],
          additionalProperties: false
        }
      };

      const response = await client.messages.create({
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages: [{ role: "user", content: "Search for flights to Tokyo departing June 1, 2026" }],
        tools: [searchFlightsTool]
      });

      console.log(response);
      ```

      ```csharp C#
      using System.Text.Json;
      using Anthropic;
      using Anthropic.Models.Messages;

      AnthropicClient client = new();

      var parameters = new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Search for flights to Tokyo departing June 1, 2026" }],
          Tools = [
              new ToolUnion(new Tool()
              {
                  Name = "search_flights",
                  Strict = true,
                  InputSchema = new InputSchema(new Dictionary<string, JsonElement>
                  {
                      ["properties"] = JsonSerializer.SerializeToElement(new Dictionary<string, object>
                      {
                          ["destination"] = new { type = "string" },
                          ["departure_date"] = new { type = "string", format = "date" },
                          ["passengers"] = new { type = "integer", @enum = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 } },
                      }),
                      ["required"] = JsonSerializer.SerializeToElement(new[] { "destination", "departure_date" }),
                      ["additionalProperties"] = JsonSerializer.SerializeToElement(false),
                  }),
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
      	MaxTokens: 1024,
      	Messages: []anthropic.MessageParam{
      		anthropic.NewUserMessage(anthropic.NewTextBlock("Search for flights to Tokyo departing June 1, 2026")),
      	},
      	Tools: []anthropic.ToolUnionParam{
      		{OfTool: &anthropic.ToolParam{
      			Name:   "search_flights",
      			Strict: anthropic.Bool(true),
      			InputSchema: anthropic.ToolInputSchemaParam{
      				Properties: map[string]any{
      					"destination": map[string]any{
      						"type": "string",
      					},
      					"departure_date": map[string]any{
      						"type":   "string",
      						"format": "date",
      					},
      					"passengers": map[string]any{
      						"type": "integer",
      						"enum": []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10},
      					},
      				},
      				Required: []string{"destination", "departure_date"},
      				ExtraFields: map[string]any{
      					"additionalProperties": false,
      				},
      			}}},
      	},
      })
      if err != nil {
      	log.Fatal(err)
      }
      fmt.Println(response.RawJSON())
      ```

      ```java Java
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      InputSchema schema = InputSchema.builder()
          .properties(
              JsonValue.from(
                  Map.of(
                      "destination", Map.of("type", "string"),
                      "departure_date", Map.of("type", "string", "format", "date"),
                      "passengers", Map.of(
                          "type", "integer",
                          "enum", List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
                      )
                  )
              )
          )
          .putAdditionalProperty("required", JsonValue.from(List.of("destination", "departure_date")))
          .putAdditionalProperty("additionalProperties", JsonValue.from(false))
          .build();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("Search for flights to Tokyo departing June 1, 2026")
          .addTool(
              Tool.builder()
                  .name("search_flights")
                  .strict(true)
                  .inputSchema(schema)
                  .build()
          )
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
      ```

      ```php PHP
      $client = new Client();

      $message = $client->messages->create(
          maxTokens: 1024,
          messages: [
              ['role' => 'user', 'content' => 'Search for flights to Tokyo departing June 1, 2026']
          ],
          model: 'claude-opus-5-5',
          tools: [
              [
                  'name' => 'search_flights',
                  'strict' => true,
                  'input_schema' => [
                      'type' => 'object',
                      'properties' => [
                          'destination' => ['type' => 'string'],
                          'departure_date' => ['type' => 'string', 'format' => 'date'],
                          'passengers' => [
                              'type' => 'integer',
                              'enum' => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
                          ]
                      ],
                      'required' => ['destination', 'departure_date'],
                      'additionalProperties' => false
                  ]
              ]
          ],
      );

      echo $message;
      ```

      ```ruby Ruby
      client = Anthropic::Client.new

      message = client.messages.create(
        model: Anthropic::Model::CLAUDE_OPUS_5_5,
        max_tokens: 1024,
        messages: [
          { role: "user", content: "Search for flights to Tokyo departing June 1, 2026" }
        ],
        tools: [
          {
            name: "search_flights",
            strict: true,
            input_schema: {
              type: "object",
              properties: {
                destination: { type: "string" },
                departure_date: { type: "string", format: "date" },
                passengers: {
                  type: "integer",
                  enum: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
                }
              },
              required: ["destination", "departure_date"],
              additionalProperties: false
            }
          }
        ]
      )
      puts message
      ```
    </CodeGroup>
  </Accordion>

  <Accordion title="使用多个经过验证的工具的智能体工作流">
    构建工具参数有保证的可靠多步骤智能体：

    <CodeGroup>
      ```bash cURL
      curl https://api.anthropic.com/v1/messages \
        -H "content-type: application/json" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -d '{
          "model": "claude-opus-5-5",
          "max_tokens": 1024,
          "messages": [
            {"role": "user", "content": "Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026"}
          ],
          "tools": [
            {
              "name": "search_flights",
              "strict": true,
              "input_schema": {
                "type": "object",
                "properties": {
                  "origin": {"type": "string"},
                  "destination": {"type": "string"},
                  "departure_date": {"type": "string", "format": "date"},
                  "travelers": {"type": "integer", "enum": [1, 2, 3, 4, 5, 6]}
                },
                "required": ["origin", "destination", "departure_date"],
                "additionalProperties": false
              }
            },
            {
              "name": "search_hotels",
              "strict": true,
              "input_schema": {
                "type": "object",
                "properties": {
                  "city": {"type": "string"},
                  "check_in": {"type": "string", "format": "date"},
                  "guests": {"type": "integer", "enum": [1, 2, 3, 4]}
                },
                "required": ["city", "check_in"],
                "additionalProperties": false
              }
            }
          ]
        }'
      ```

      ```bash CLI
      ant messages create <<'YAML'
      model: claude-opus-5-5
      max_tokens: 1024
      messages:
        - role: user
          content: >-
            Help me plan a trip from New York to Paris for 2 people,
            departing June 1, 2026
      tools:
        - name: search_flights
          strict: true
          input_schema:
            type: object
            properties:
              origin: {type: string}
              destination: {type: string}
              departure_date: {type: string, format: date}
              travelers: {type: integer, enum: [1, 2, 3, 4, 5, 6]}
            required: [origin, destination, departure_date]
            additionalProperties: false
        - name: search_hotels
          strict: true
          input_schema:
            type: object
            properties:
              city: {type: string}
              check_in: {type: string, format: date}
              guests: {type: integer, enum: [1, 2, 3, 4]}
            required: [city, check_in]
            additionalProperties: false
      YAML
      ```

      ```python Python
      client = Anthropic()
      response = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[
              {
                  "role": "user",
                  "content": "Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026",
              }
          ],
          tools=[
              {
                  "name": "search_flights",
                  "strict": True,
                  "input_schema": {
                      "type": "object",
                      "properties": {
                          "origin": {"type": "string"},
                          "destination": {"type": "string"},
                          "departure_date": {"type": "string", "format": "date"},
                          "travelers": {"type": "integer", "enum": [1, 2, 3, 4, 5, 6]},
                      },
                      "required": ["origin", "destination", "departure_date"],
                      "additionalProperties": False,
                  },
              },
              {
                  "name": "search_hotels",
                  "strict": True,
                  "input_schema": {
                      "type": "object",
                      "properties": {
                          "city": {"type": "string"},
                          "check_in": {"type": "string", "format": "date"},
                          "guests": {"type": "integer", "enum": [1, 2, 3, 4]},
                      },
                      "required": ["city", "check_in"],
                      "additionalProperties": False,
                  },
              },
          ],
      )

      print(response)
      ```

      ```typescript TypeScript
      const client = new Anthropic();

      const tools: Anthropic.Tool[] = [
        {
          name: "search_flights",
          strict: true,
          input_schema: {
            type: "object",
            properties: {
              origin: { type: "string" },
              destination: { type: "string" },
              departure_date: { type: "string", format: "date" },
              travelers: { type: "integer", enum: [1, 2, 3, 4, 5, 6] }
            },
            required: ["origin", "destination", "departure_date"],
            additionalProperties: false
          }
        },
        {
          name: "search_hotels",
          strict: true,
          input_schema: {
            type: "object",
            properties: {
              city: { type: "string" },
              check_in: { type: "string", format: "date" },
              guests: { type: "integer", enum: [1, 2, 3, 4] }
            },
            required: ["city", "check_in"],
            additionalProperties: false
          }
        }
      ];

      const response = await client.messages.create({
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages: [
          {
            role: "user",
            content:
              "Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026"
          }
        ],
        tools: tools
      });

      console.log(response);
      ```

      ```csharp C#
      using System.Text.Json;
      using Anthropic;
      using Anthropic.Models.Messages;

      AnthropicClient client = new();

      var parameters = new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026" }],
          Tools = [
              new ToolUnion(new Tool()
              {
                  Name = "search_flights",
                  Strict = true,
                  InputSchema = new InputSchema(new Dictionary<string, JsonElement>
                  {
                      ["properties"] = JsonSerializer.SerializeToElement(new Dictionary<string, object>
                      {
                          ["origin"] = new { type = "string" },
                          ["destination"] = new { type = "string" },
                          ["departure_date"] = new { type = "string", format = "date" },
                          ["travelers"] = new { type = "integer", @enum = new[] { 1, 2, 3, 4, 5, 6 } },
                      }),
                      ["required"] = JsonSerializer.SerializeToElement(new[] { "origin", "destination", "departure_date" }),
                      ["additionalProperties"] = JsonSerializer.SerializeToElement(false),
                  }),
              }),
              new ToolUnion(new Tool()
              {
                  Name = "search_hotels",
                  Strict = true,
                  InputSchema = new InputSchema(new Dictionary<string, JsonElement>
                  {
                      ["properties"] = JsonSerializer.SerializeToElement(new Dictionary<string, object>
                      {
                          ["city"] = new { type = "string" },
                          ["check_in"] = new { type = "string", format = "date" },
                          ["guests"] = new { type = "integer", @enum = new[] { 1, 2, 3, 4 } },
                      }),
                      ["required"] = JsonSerializer.SerializeToElement(new[] { "city", "check_in" }),
                      ["additionalProperties"] = JsonSerializer.SerializeToElement(false),
                  }),
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
      	MaxTokens: 1024,
      	Messages: []anthropic.MessageParam{
      		anthropic.NewUserMessage(anthropic.NewTextBlock("Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026")),
      	},
      	Tools: []anthropic.ToolUnionParam{
      		{OfTool: &anthropic.ToolParam{
      			Name:   "search_flights",
      			Strict: anthropic.Bool(true),
      			InputSchema: anthropic.ToolInputSchemaParam{
      				Properties: map[string]any{
      					"origin":         map[string]any{"type": "string"},
      					"destination":    map[string]any{"type": "string"},
      					"departure_date": map[string]any{"type": "string", "format": "date"},
      					"travelers":      map[string]any{"type": "integer", "enum": []int{1, 2, 3, 4, 5, 6}},
      				},
      				Required: []string{"origin", "destination", "departure_date"},
      				ExtraFields: map[string]any{
      					"additionalProperties": false,
      				},
      			}}},
      		{OfTool: &anthropic.ToolParam{
      			Name:   "search_hotels",
      			Strict: anthropic.Bool(true),
      			InputSchema: anthropic.ToolInputSchemaParam{
      				Properties: map[string]any{
      					"city":     map[string]any{"type": "string"},
      					"check_in": map[string]any{"type": "string", "format": "date"},
      					"guests":   map[string]any{"type": "integer", "enum": []int{1, 2, 3, 4}},
      				},
      				Required: []string{"city", "check_in"},
      				ExtraFields: map[string]any{
      					"additionalProperties": false,
      				},
      			}}},
      	},
      })
      if err != nil {
      	log.Fatal(err)
      }
      fmt.Println(response.RawJSON())
      ```

      ```java Java
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      InputSchema flightsSchema = InputSchema.builder()
          .properties(
              JsonValue.from(
                  Map.of(
                      "origin", Map.of("type", "string"),
                      "destination", Map.of("type", "string"),
                      "departure_date", Map.of("type", "string", "format", "date"),
                      "travelers", Map.of("type", "integer", "enum", List.of(1, 2, 3, 4, 5, 6))
                  )
              )
          )
          .putAdditionalProperty("required", JsonValue.from(List.of("origin", "destination", "departure_date")))
          .putAdditionalProperty("additionalProperties", JsonValue.from(false))
          .build();

      InputSchema hotelsSchema = InputSchema.builder()
          .properties(
              JsonValue.from(
                  Map.of(
                      "city", Map.of("type", "string"),
                      "check_in", Map.of("type", "string", "format", "date"),
                      "guests", Map.of("type", "integer", "enum", List.of(1, 2, 3, 4))
                  )
              )
          )
          .putAdditionalProperty("required", JsonValue.from(List.of("city", "check_in")))
          .putAdditionalProperty("additionalProperties", JsonValue.from(false))
          .build();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026")
          .addTool(
              Tool.builder()
                  .name("search_flights")
                  .strict(true)
                  .inputSchema(flightsSchema)
                  .build()
          )
          .addTool(
              Tool.builder()
                  .name("search_hotels")
                  .strict(true)
                  .inputSchema(hotelsSchema)
                  .build()
          )
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
      ```

      ```php PHP
      $client = new Client();

      $message = $client->messages->create(
          maxTokens: 1024,
          messages: [
              ['role' => 'user', 'content' => 'Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026']
          ],
          model: 'claude-opus-5-5',
          tools: [
              [
                  'name' => 'search_flights',
                  'strict' => true,
                  'input_schema' => [
                      'type' => 'object',
                      'properties' => [
                          'origin' => ['type' => 'string'],
                          'destination' => ['type' => 'string'],
                          'departure_date' => ['type' => 'string', 'format' => 'date'],
                          'travelers' => ['type' => 'integer', 'enum' => [1, 2, 3, 4, 5, 6]]
                      ],
                      'required' => ['origin', 'destination', 'departure_date'],
                      'additionalProperties' => false
                  ]
              ],
              [
                  'name' => 'search_hotels',
                  'strict' => true,
                  'input_schema' => [
                      'type' => 'object',
                      'properties' => [
                          'city' => ['type' => 'string'],
                          'check_in' => ['type' => 'string', 'format' => 'date'],
                          'guests' => ['type' => 'integer', 'enum' => [1, 2, 3, 4]]
                      ],
                      'required' => ['city', 'check_in'],
                      'additionalProperties' => false
                  ]
              ]
          ],
      );

      echo $message;
      ```

      ```ruby Ruby
      client = Anthropic::Client.new

      message = client.messages.create(
        model: Anthropic::Model::CLAUDE_OPUS_5_5,
        max_tokens: 1024,
        messages: [
          { role: "user", content: "Help me plan a trip from New York to Paris for 2 people, departing June 1, 2026" }
        ],
        tools: [
          {
            name: "search_flights",
            strict: true,
            input_schema: {
              type: "object",
              properties: {
                origin: { type: "string" },
                destination: { type: "string" },
                departure_date: { type: "string", format: "date" },
                travelers: { type: "integer", enum: [1, 2, 3, 4, 5, 6] }
              },
              required: ["origin", "destination", "departure_date"],
              additionalProperties: false
            }
          },
          {
            name: "search_hotels",
            strict: true,
            input_schema: {
              type: "object",
              properties: {
                city: { type: "string" },
                check_in: { type: "string", format: "date" },
                guests: { type: "integer", enum: [1, 2, 3, 4] }
              },
              required: ["city", "check_in"],
              additionalProperties: false
            }
          }
        ]
      )
      puts message
      ```
    </CodeGroup>
  </Accordion>
</AccordionGroup>

## 数据保留

严格工具使用会使用与[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)相同的管道，将工具 `input_schema` 定义编译为语法。工具 schema 会在最后一次使用后被临时缓存最多 24 小时。提示和响应不会在 API 响应之后被保留。

严格工具使用符合 HIPAA 资格，但**受保护的健康信息（PHI）不得包含在工具 schema 定义中**。API 将编译后的 schema 与消息内容分开缓存，这些缓存的 schema 不会获得与提示和响应相同的 PHI 保护。请勿在 `input_schema` 属性名称、`enum` 值、`const` 值或 `pattern` 正则表达式中包含 PHI。PHI 只应出现在消息内容（提示和响应）中，在那里它受到 HIPAA 保障措施的保护。

## 后续步骤

<CardGroup cols={2}>
  <Card title="网页获取工具" icon="link" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool">
    从特定 URL 获取并读取内容，将实时网页内容引入 Claude 的上下文。
  </Card>

  <Card title="结合提示缓存的工具使用" icon="database" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching">
    跨轮次缓存工具定义，以降低成本和延迟。
  </Card>

  <Card title="结构化输出" icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs">
    使用相同的语法约束采样获取经过验证的 JSON 响应。
  </Card>

  <Card title="定义工具" icon="hammer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools">
    指定工具 schema、编写有效的描述，并控制 Claude 何时调用您的工具。
  </Card>
</CardGroup>

## 可用性

严格工具使用在所有模型上可用，并具备HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | 模型                                                             |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 可用 | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                                                                                 | HIPAA                                                                                                                                             |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 状态   | 具备资格（有条件；不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)）                                                         | 具备资格                                                                                                                                              |
| 详细信息 | 您的提示和 Claude 的输出不会被存储。仅缓存 JSON schema，自上次使用起最长保留 24 小时。这也涵盖严格工具使用（工具上的 `strict: true`），它使用相同的语法管道。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#data-retention) | JSON schema 定义中不得包含 PHI；请参阅 PHI 处理指南。 [了解更多](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#phi-handling-guidelines) |

适用的端点：

* `/v1/messages`
