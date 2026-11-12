---
title: 编程式工具调用
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling
description: 让 Claude 在代码执行容器中通过代码调用您的工具，减少多工具工作流中的模型往返次数和令牌使用量。
featureMetadata:
  status: stable
  zdr:
    eligibility: not-eligible
    note: 基于代码执行容器构建；数据最多保留 30 天。
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
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API: stable
    Claude Platform on AWS: stable
    Microsoft Foundry (hosted on Anthropic): stable
  details:
    - 程序化工具调用需要使用 `code_execution_20260120` 或更高[工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#tool-versions)的代码执行工具。
---

“Programmatic tool calling”（程序化工具调用）允许 Claude 编写代码，在[代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)容器内以编程方式调用您的工具，而无需每次工具调用都经过模型往返。这降低了多工具工作流的"latency"（延迟），并通过允许 Claude 在数据到达模型的"context window"（上下文窗口）之前对其进行过滤或处理来减少令牌消耗。在 [BrowseComp](https://arxiv.org/abs/2504.12516) 和 [DeepSearchQA](https://github.com/google-deepmind/deepsearchqa) 等测试多步骤网络研究和复杂信息检索的智能体搜索基准测试中，在基本搜索工具之上添加程序化工具调用后，性能平均提升了 11%，同时输入令牌减少了 24%（请参阅[通过动态过滤改进网络搜索](https://claude.com/blog/improved-web-search-with-dynamic-filtering)）。

假设要检查 20 名员工的预算合规情况：传统方法需要 20 次独立的模型往返，并在此过程中将数千条费用明细拉入上下文。使用程序化工具调用，单个脚本即可运行全部 20 次查询、过滤结果，并仅返回超出限额的员工，从而将 Claude 需要推理的内容从数百 KB 缩减到几行。

<Tip>
  如需更深入地了解编程式工具调用所解决的推理和上下文成本问题，请参阅[高级工具使用](https://www.anthropic.com/engineering/advanced-tool-use)。
</Tip>

程序化工具调用需要使用工具版本为 `code_execution_20260120` 或更高版本的[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)。Claude Haiku 4.5 接受 `code_execution_20260120` 及更高的工具版本，但不支持程序化工具调用。要在发送请求之前检查某个模型是否支持程序化工具调用，请从 Models API 读取其 `capabilities.code_execution.supported` 值。[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api) 介绍了该字段。

## 快速开始

以下示例中，Claude 以编程方式多次查询数据库并汇总结果。在工具定义中添加 `allowed_callers: ["code_execution_20260120"]` 即可使该工具能够在代码执行中被调用（请参阅 [`allowed_callers` 字段](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#the-allowed-callers-field)）：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue"
        }
      ],
      "tools": [
        {
          "type": "code_execution_20260120",
          "name": "code_execution"
        },
        {
          "name": "query_database",
          "description": "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
          "input_schema": {
            "type": "object",
            "properties": {
              "sql": {
                "type": "string",
                "description": "SQL query to execute"
              }
            },
            "required": ["sql"]
          },
          "allowed_callers": ["code_execution_20260120"]
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: >-
        Query sales data for the West, East, and Central regions, then
        tell me which region had the highest revenue
  tools:
    - type: code_execution_20260120
      name: code_execution
    - name: query_database
      description: >-
        Execute a SQL query against the sales database. Returns a list
        of rows as JSON objects.
      input_schema:
        type: object
        properties:
          sql:
            type: string
            description: SQL query to execute
        required:
          - sql
      allowed_callers:
        - code_execution_20260120
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue",
          }
      ],
      tools=[
          {"type": "code_execution_20260120", "name": "code_execution"},
          {
              "name": "query_database",
              "description": "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "sql": {"type": "string", "description": "SQL query to execute"}
                  },
                  "required": ["sql"],
              },
              "allowed_callers": ["code_execution_20260120"],
          },
      ],
  )

  print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content:
          "Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue"
      }
    ],
    tools: [
      {
        type: "code_execution_20260120",
        name: "code_execution"
      },
      {
        name: "query_database",
        description:
          "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
        input_schema: {
          type: "object" as const,
          properties: {
            sql: {
              type: "string",
              description: "SQL query to execute"
            }
          },
          required: ["sql"]
        },
        allowed_callers: ["code_execution_20260120"]
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
      MaxTokens = 4096,
      Messages = [
          new UserMessageParam
          {
              Content = "Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue"
          }
      ],
      Tools = [
          new CodeExecutionTool20260120(),
          new ToolUnion(new Tool()
          {
              Name = "query_database",
              Description = "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
              InputSchema = new InputSchema()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["sql"] = JsonSerializer.SerializeToElement(new { type = "string", description = "SQL query to execute" }),
                  },
                  Required = ["sql"],
              },
              AllowedCallers = ["code_execution_20260120"]
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
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfCodeExecutionTool20260120: &anthropic.CodeExecutionTool20260120Param{}},
  		{OfTool: &anthropic.ToolParam{
  			Name:        "query_database",
  			Description: anthropic.String("Execute a SQL query against the sales database. Returns a list of rows as JSON objects."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"sql": map[string]any{
  						"type":        "string",
  						"description": "SQL query to execute",
  					},
  				},
  				Required: []string{"sql"},
  			},
  			AllowedCallers: []string{"code_execution_20260120"},
  		}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.CodeExecutionTool20260120;
  // ...

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue")
          .addTool(CodeExecutionTool20260120.builder().build())
          .addTool(Tool.builder()
              .name("query_database")
              .description("Execute a SQL query against the sales database. Returns a list of rows as JSON objects.")
              .inputSchema(InputSchema.builder()
                  .properties(JsonValue.from(Map.of(
                      "sql", Map.of(
                          "type", "string",
                          "description", "SQL query to execute"
                      )
                  )))
                  .putAdditionalProperty("required", JsonValue.from(List.of("sql")))
                  .build())
              .allowedCallers(List.of(Tool.AllowedCaller.of("code_execution_20260120")))
              .build())
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          ['role' => 'user', 'content' => 'Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue'],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          [
              'type' => 'code_execution_20260120',
              'name' => 'code_execution',
          ],
          [
              'name' => 'query_database',
              'description' => 'Execute a SQL query against the sales database. Returns a list of rows as JSON objects.',
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'sql' => [
                          'type' => 'string',
                          'description' => 'SQL query to execute',
                      ],
                  ],
                  'required' => ['sql'],
              ],
              'allowed_callers' => ['code_execution_20260120'],
          ],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Query sales data for the West, East, and Central regions, then tell me which region had the highest revenue"
      }
    ],
    tools: [
      {
        type: "code_execution_20260120",
        name: "code_execution"
      },
      {
        name: "query_database",
        description: "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
        input_schema: {
          type: "object",
          properties: {
            sql: {
              type: "string",
              description: "SQL query to execute"
            }
          },
          required: ["sql"]
        },
        allowed_callers: ["code_execution_20260120"]
      }
    ]
  )

  puts message
  ```
</CodeGroup>

响应以 `stop_reason: "tool_use"` 停止，包含一个 `container` ID，以及一个针对 `query_database` 的 `tool_use` 块，其 `caller` 字段标识了调用它的代码执行运行。按照[示例工作流的步骤 3](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#step-3-provide-tool-result) 所示返回结果，以便代码能够完成执行。

## 编程式工具调用的工作原理

当您将某个工具配置为可从代码执行中调用，并且 Claude 判断需要该工具时：

1. Claude 编写将该工具作为函数调用的 Python 代码。代码可以包含多个工具调用，以及预处理和后处理逻辑。
2. Claude 通过代码执行在沙盒容器中运行这段代码。
3. 当调用工具函数时，代码执行暂停，API 返回一个 `tool_use` 块。
4. 您提供工具结果，代码执行继续（中间结果不会加载到 Claude 的上下文窗口中）。
5. 所有代码执行完成后，Claude 接收最终输出并继续处理任务。

这种方法特别适用于：

* **大规模数据处理：** 在工具结果到达 Claude 的上下文之前对其进行过滤或聚合。
* **多步骤工作流：** 通过串行或循环调用工具，无需在工具调用之间对 Claude 进行采样，从而节省令牌并降低延迟。
* **条件逻辑：** 根据中间工具结果做出决策。

<Note>
  允许代码执行调用方的工具会以异步 Python 函数的形式暴露给 Claude 的代码，因此 Claude 可以使用 `asyncio.gather` 并行运行它们。每个函数接受一个参数字典并返回一个字符串：即您发回的 `tool_result` 的文本。Claude 的代码使用顶层 `await` 等待这些函数，并将所需的结果解析为结构化数据，例如 `rows = json.loads(await query_database({"sql": "<sql>"}))`。
</Note>

## 核心概念

### `allowed_callers` 字段

`allowed_callers` 字段指定哪些上下文可以调用工具：

```json
{
  "name": "query_database",
  "description": "Execute a SQL query against the database",
  "input_schema": {
    // ...
  },
  "allowed_callers": ["code_execution_20260120"]
}
```

**可能的取值：**

* `["direct"]` - 引导 Claude 直接调用此工具（省略时的默认值）。
* `["code_execution_20260120"]` - 引导 Claude 仅在代码执行中调用此工具。
* `["direct", "code_execution_20260120"]` - Claude 可以直接调用此工具，也可以在代码执行中调用。

`"code_execution_20260120"` 和 `"code_execution_20260521"` 在 `allowed_callers` 中均被接受且可互换：使用任一代码执行工具版本的请求都能满足列出任一调用方的工具。无论请求声明的是哪个版本，响应块始终将调用方标记为 `code_execution_20260120`。

<Tip>
  请为每个工具选择 `["direct"]` 或 `["code_execution_20260120"]` 其中之一，而不是同时启用两者，因为这样能为 Claude 提供更清晰的指引，说明如何最好地使用该工具。
</Tip>

<Note>
  `allowed_callers` 控制工具如何呈现给 Claude，并会根据 `tool_choice` 进行验证，但它并不是在 API 层面对直接调用的硬性阻止。Claude 会被强烈引导遵守它，但您的客户端仍应准备好处理其定义的任何工具的直接 `tool_use`。请勿将 `allowed_callers` 作为安全边界。
</Note>

### 响应中的 `caller` 字段

每个工具使用块都包含一个 `caller` 字段，指示其调用方式：

**直接调用（传统工具使用）：**

```json
{
  "type": "tool_use",
  "id": "toolu_abc123",
  "name": "query_database",
  "input": { "sql": "<sql>" },
  "caller": { "type": "direct" }
}
```

**程序化调用：**

```json
{
  "type": "tool_use",
  "id": "toolu_xyz789",
  "name": "query_database",
  "input": { "sql": "<sql>" },
  "caller": {
    "type": "code_execution_20260120",
    "tool_id": "srvtoolu_abc123"
  }
}
```

`tool_id` 是发起调用的代码执行 `server_tool_use` 块的 `id`，因此您可以将每个编程式 `tool_use` 与产生它的代码执行运行相匹配。

### 容器生命周期

编程式工具调用使用与代码执行相同的容器：

* **容器创建：** 除非您重用现有容器，否则每个请求都会创建一个新容器。
* **容器 ID：** 在响应的 `container` 字段中返回，同时附带 `expires_at` 时间戳。
* **复用：** 在下一个请求中传回容器 ID 以保持状态。当编程式工具调用正在等待您的结果时，该请求中的容器 ID 是必需的而非可选的：没有它，API 会拒绝该请求。
* **过期：** 容器在创建 30 天后过期。闲置约 5 分钟后，容器会被保存检查点（checkpoint），在 30 天期限内使用其 ID 发送请求即可恢复该容器。`expires_at` 时间戳是一个较短的滚动值，并不反映 30 天的期限。

<Warning>
  当 Claude 的代码正在等待编程式工具结果时，待处理的调用会在大约 4 分钟后超时，并在代码内部抛出 `TimeoutError`。请在暂停响应的 `expires_at` 时间戳之前尽早返回每个工具结果。请参阅[工具调用期间容器过期](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#container-expiration-during-tool-call)。
</Warning>

## 示例工作流

以下是完整的编程式工具调用流程的工作方式：

### 第 1 步：初始请求

发送一个包含代码执行和允许编程式调用的工具的请求。要启用编程式调用，请在工具定义中添加 `allowed_callers` 字段。

<Note>
  请在工具描述中详细说明工具的输出格式。如果您指定工具返回 JSON，Claude 会尝试在代码中反序列化并处理结果。您提供的输出模式细节越多，Claude 就越能以编程方式妥善处理响应。
</Note>

请求结构与[快速开始](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#quick-start)示例完全相同：在工具列表中包含 `code_execution`，为您希望 Claude 从代码中调用的任何工具添加 `allowed_callers: ["code_execution_20260120"]`，然后发送您的用户消息。本工作流的其余步骤使用用户消息 `"Query customer purchase history from the last quarter and identify our top 5 customers by revenue"`。

### 第 2 步：包含工具调用的 API 响应

Claude 编写调用您工具的代码。API 暂停并返回：

```json Output
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "I'll query the purchase history and analyze the results."
    },
    {
      "type": "server_tool_use",
      "id": "srvtoolu_abc123",
      "name": "code_execution",
      "input": {
        "code": "import json\n\nrows = json.loads(await query_database({'sql': '<sql>'}))\ntop_customers = sorted(rows, key=lambda x: x['revenue'], reverse=True)[:5]\nprint(f'Top 5 customers: {top_customers}')"
      }
    },
    {
      "type": "tool_use",
      "id": "toolu_def456",
      "name": "query_database",
      "input": { "sql": "<sql>" },
      "caller": {
        "type": "code_execution_20260120",
        "tool_id": "srvtoolu_abc123"
      }
    }
  ],
  "container": {
    "id": "container_xyz789",
    "expires_at": "2026-01-20T14:30:00Z"
  },
  "stop_reason": "tool_use"
}
```

### 第 3 步：提供工具结果

发送完整的对话历史以及您的工具结果。此请求有三个细节需要注意：

* 承载您结果的用户消息只能包含 `tool_result` 块。请参阅[消息格式限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#message-formatting-restrictions)。
* 传入暂停响应中的 `container` ID。如果续接请求存在待处理的编程式工具调用但没有容器 ID，API 会拒绝该请求。
* 发送与原始请求相同的 `tools` 数组。代码执行工具必须仍然存在，暂停的代码才能恢复；并且您在此请求中发送的工具就是 Claude 和正在运行的代码在本轮剩余部分可以使用的定义。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "container": "container_xyz789",
      "messages": [
        {
          "role": "user",
          "content": "Query customer purchase history from the last quarter and identify our top 5 customers by revenue"
        },
        {
          "role": "assistant",
          "content": [
            {
              "type": "text",
              "text": "I will query the purchase history and analyze the results."
            },
            {
              "type": "server_tool_use",
              "id": "srvtoolu_abc123",
              "name": "code_execution",
              "input": {"code": "..."}
            },
            {
              "type": "tool_use",
              "id": "toolu_def456",
              "name": "query_database",
              "input": {"sql": "<sql>"},
              "caller": {
                "type": "code_execution_20260120",
                "tool_id": "srvtoolu_abc123"
              }
            }
          ]
        },
        {
          "role": "user",
          "content": [
            {
              "type": "tool_result",
              "tool_use_id": "toolu_def456",
              "content": "[{\"customer_id\": \"C1\", \"revenue\": 45000}, {\"customer_id\": \"C2\", \"revenue\": 38000}]"
            }
          ]
        }
      ],
      "tools": [
        {
          "type": "code_execution_20260120",
          "name": "code_execution"
        },
        {
          "name": "query_database",
          "description": "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
          "input_schema": {
            "type": "object",
            "properties": {
              "sql": {
                "type": "string",
                "description": "SQL query to execute"
              }
            },
            "required": ["sql"]
          },
          "allowed_callers": ["code_execution_20260120"]
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  container: container_xyz789
  messages:
    - role: user
      content: >-
        Query customer purchase history from the last quarter and identify our
        top 5 customers by revenue
    - role: assistant
      content:
        - type: text
          text: I'll query the purchase history and analyze the results.
        - type: server_tool_use
          id: srvtoolu_abc123
          name: code_execution
          input:
            code: "..."
        - type: tool_use
          id: toolu_def456
          name: query_database
          input:
            sql: "<sql>"
          caller:
            type: code_execution_20260120
            tool_id: srvtoolu_abc123
    - role: user
      content:
        - type: tool_result
          tool_use_id: toolu_def456
          content: >-
            [{"customer_id": "C1", "revenue": 45000}, {"customer_id": "C2",
            "revenue": 38000}, ...]
  # 与原始请求相同的 tools 数组
  tools:
    - type: code_execution_20260120
      name: code_execution
    - name: query_database
      description: >-
        Execute a SQL query against the sales database. Returns a list
        of rows as JSON objects.
      input_schema:
        type: object
        properties:
          sql:
            type: string
            description: SQL query to execute
        required:
          - sql
      allowed_callers:
        - code_execution_20260120
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      container="container_xyz789",  # Reuse the container
      messages=[
          {
              "role": "user",
              "content": "Query customer purchase history from the last quarter and identify our top 5 customers by revenue",
          },
          {
              "role": "assistant",
              "content": [
                  {
                      "type": "text",
                      "text": "I'll query the purchase history and analyze the results.",
                  },
                  {
                      "type": "server_tool_use",
                      "id": "srvtoolu_abc123",
                      "name": "code_execution",
                      "input": {"code": "..."},
                  },
                  {
                      "type": "tool_use",
                      "id": "toolu_def456",
                      "name": "query_database",
                      "input": {"sql": "<sql>"},
                      "caller": {
                          "type": "code_execution_20260120",
                          "tool_id": "srvtoolu_abc123",
                      },
                  },
              ],
          },
          {
              "role": "user",
              "content": [
                  {
                      "type": "tool_result",
                      "tool_use_id": "toolu_def456",
                      "content": '[{"customer_id": "C1", "revenue": 45000}, {"customer_id": "C2", "revenue": 38000}, ...]',
                  }
              ],
          },
      ],
      # 与原始请求相同的 tools 数组
      tools=[
          {"type": "code_execution_20260120", "name": "code_execution"},
          {
              "name": "query_database",
              "description": "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "sql": {"type": "string", "description": "SQL query to execute"}
                  },
                  "required": ["sql"],
              },
              "allowed_callers": ["code_execution_20260120"],
          },
      ],
  )

  print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    container: "container_xyz789", // Reuse the container
    messages: [
      {
        role: "user",
        content:
          "Query customer purchase history from the last quarter and identify our top 5 customers by revenue"
      },
      {
        role: "assistant",
        content: [
          { type: "text", text: "I'll query the purchase history and analyze the results." },
          {
            type: "server_tool_use",
            id: "srvtoolu_abc123",
            name: "code_execution",
            input: { code: "..." }
          },
          {
            type: "tool_use",
            id: "toolu_def456",
            name: "query_database",
            input: { sql: "<sql>" },
            caller: {
              type: "code_execution_20260120",
              tool_id: "srvtoolu_abc123"
            }
          }
        ]
      },
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: "toolu_def456",
            content:
              '[{"customer_id": "C1", "revenue": 45000}, {"customer_id": "C2", "revenue": 38000}, ...]'
          }
        ]
      }
    ],
    // 与原始请求相同的 tools 数组
    tools: [
      {
        type: "code_execution_20260120",
        name: "code_execution"
      },
      {
        name: "query_database",
        description:
          "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
        input_schema: {
          type: "object" as const,
          properties: {
            sql: {
              type: "string",
              description: "SQL query to execute"
            }
          },
          required: ["sql"]
        },
        allowed_callers: ["code_execution_20260120"]
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
      MaxTokens = 4096,
      Container = "container_xyz789",
      Messages =
      [
          new UserMessageParam
          {
              Content = "Query customer purchase history from the last quarter and identify our top 5 customers by revenue"
          },
          new AssistantMessageParam
          {
              Content = new List<ContentBlockParam>
              {
                  new TextBlockParam { Text = "I'll query the purchase history and analyze the results." },
                  new ServerToolUseBlockParam
                  {
                      ID = "srvtoolu_abc123",
                      Name = ServerToolUseBlockParamName.CodeExecution,
                      Input = new Dictionary<string, JsonElement>
                      {
                          ["code"] = JsonSerializer.SerializeToElement("..."),
                      }
                  },
                  new ToolUseBlockParam
                  {
                      ID = "toolu_def456",
                      Name = "query_database",
                      Input = new Dictionary<string, JsonElement>
                      {
                          ["sql"] = JsonSerializer.SerializeToElement("<sql>"),
                      },
                      Caller = new ServerToolCaller20260120 { ToolID = "srvtoolu_abc123" }
                  }
              }
          },
          new UserMessageParam
          {
              Content = new List<ContentBlockParam>
              {
                  new ToolResultBlockParam
                  {
                      ToolUseID = "toolu_def456",
                      Content = "[{\"customer_id\": \"C1\", \"revenue\": 45000}, {\"customer_id\": \"C2\", \"revenue\": 38000}, ...]"
                  }
              }
          }
      ],
      // 与原始请求相同的 tools 数组
      Tools = [
          new CodeExecutionTool20260120(),
          new ToolUnion(new Tool()
          {
              Name = "query_database",
              Description = "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
              InputSchema = new InputSchema()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["sql"] = JsonSerializer.SerializeToElement(new { type = "string", description = "SQL query to execute" }),
                  },
                  Required = ["sql"],
              },
              AllowedCallers = ["code_execution_20260120"]
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
  	MaxTokens: 4096,
  	Container: anthropic.MessageCreateParamsContainerUnion{
  		OfString: anthropic.String("container_xyz789"),
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Query customer purchase history from the last quarter and identify our top 5 customers by revenue")),
  		anthropic.NewAssistantMessage(
  			anthropic.NewTextBlock("I'll query the purchase history and analyze the results."),
  			anthropic.NewServerToolUseBlock("srvtoolu_abc123", map[string]any{"code": "..."}, anthropic.ServerToolUseBlockParamNameCodeExecution),
  			anthropic.ContentBlockParamUnion{OfToolUse: &anthropic.ToolUseBlockParam{
  				ID:    "toolu_def456",
  				Name:  "query_database",
  				Input: map[string]any{"sql": "<sql>"},
  				Caller: anthropic.ToolUseBlockParamCallerUnion{
  					OfCodeExecution20260120: &anthropic.ServerToolCaller20260120Param{
  						ToolID: "srvtoolu_abc123",
  					},
  				},
  			}},
  		),
  		anthropic.NewUserMessage(anthropic.ContentBlockParamUnion{OfToolResult: &anthropic.ToolResultBlockParam{
  			ToolUseID: "toolu_def456",
  			Content: []anthropic.ToolResultBlockParamContentUnion{
  				{OfText: &anthropic.TextBlockParam{
  					Text: `[{"customer_id": "C1", "revenue": 45000}, {"customer_id": "C2", "revenue": 38000}, ...]`,
  				}},
  			},
  		}}),
  	},
  	// 与原始请求相同的 tools 数组
  	Tools: []anthropic.ToolUnionParam{
  		{OfCodeExecutionTool20260120: &anthropic.CodeExecutionTool20260120Param{}},
  		{OfTool: &anthropic.ToolParam{
  			Name:        "query_database",
  			Description: anthropic.String("Execute a SQL query against the sales database. Returns a list of rows as JSON objects."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"sql": map[string]any{
  						"type":        "string",
  						"description": "SQL query to execute",
  					},
  				},
  				Required: []string{"sql"},
  			},
  			AllowedCallers: []string{"code_execution_20260120"},
  		}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.CodeExecutionTool20260120;
  // ...

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .container("container_xyz789")
          .addUserMessage("Query customer purchase history from the last quarter and identify our top 5 customers by revenue")
          .addAssistantMessageOfContentBlockParams(List.of(
              ContentBlockParam.ofText(
                  TextBlockParam.builder()
                      .text("I'll query the purchase history and analyze the results.")
                      .build()),
              ContentBlockParam.ofServerToolUse(
                  ServerToolUseBlockParam.builder()
                      .id("srvtoolu_abc123")
                      .name(ServerToolUseBlockParam.Name.CODE_EXECUTION)
                      .input(JsonValue.from(Map.of("code", "...")))
                      .build()),
              ContentBlockParam.ofToolUse(
                  ToolUseBlockParam.builder()
                      .id("toolu_def456")
                      .name("query_database")
                      .input(JsonValue.from(Map.of("sql", "<sql>")))
                      .codeExecution20260120Caller("srvtoolu_abc123")
                      .build())
          ))
          .addUserMessageOfContentBlockParams(List.of(
              ContentBlockParam.ofToolResult(
                  ToolResultBlockParam.builder()
                      .toolUseId("toolu_def456")
                      .content("[{\"customer_id\": \"C1\", \"revenue\": 45000}, {\"customer_id\": \"C2\", \"revenue\": 38000}, ...]")
                      .build())
          ))
          // 与原始请求相同的 tools 数组
          .addTool(CodeExecutionTool20260120.builder().build())
          .addTool(Tool.builder()
              .name("query_database")
              .description("Execute a SQL query against the sales database. Returns a list of rows as JSON objects.")
              .inputSchema(InputSchema.builder()
                  .properties(JsonValue.from(Map.of(
                      "sql", Map.of(
                          "type", "string",
                          "description", "SQL query to execute"
                      )
                  )))
                  .putAdditionalProperty("required", JsonValue.from(List.of("sql")))
                  .build())
              .allowedCallers(List.of(Tool.AllowedCaller.of("code_execution_20260120")))
              .build())
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => 'Query customer purchase history from the last quarter and identify our top 5 customers by revenue',
          ],
          [
              'role' => 'assistant',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => "I'll query the purchase history and analyze the results.",
                  ],
                  [
                      'type' => 'server_tool_use',
                      'id' => 'srvtoolu_abc123',
                      'name' => 'code_execution',
                      'input' => ['code' => '...'],
                  ],
                  [
                      'type' => 'tool_use',
                      'id' => 'toolu_def456',
                      'name' => 'query_database',
                      'input' => ['sql' => '<sql>'],
                      'caller' => [
                          'type' => 'code_execution_20260120',
                          'tool_id' => 'srvtoolu_abc123',
                      ],
                  ],
              ],
          ],
          [
              'role' => 'user',
              'content' => [
                  [
                      'type' => 'tool_result',
                      'tool_use_id' => 'toolu_def456',
                      'content' => '[{"customer_id": "C1", "revenue": 45000}, {"customer_id": "C2", "revenue": 38000}, ...]',
                  ],
              ],
          ],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      container: 'container_xyz789',
      // 与原始请求相同的 tools 数组
      tools: [
          [
              'type' => 'code_execution_20260120',
              'name' => 'code_execution',
          ],
          [
              'name' => 'query_database',
              'description' => 'Execute a SQL query against the sales database. Returns a list of rows as JSON objects.',
              'input_schema' => [
                  'type' => 'object',
                  'properties' => [
                      'sql' => [
                          'type' => 'string',
                          'description' => 'SQL query to execute',
                      ],
                  ],
                  'required' => ['sql'],
              ],
              'allowed_callers' => ['code_execution_20260120'],
          ],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    container: "container_xyz789",
    messages: [
      {
        role: "user",
        content: "Query customer purchase history from the last quarter and identify our top 5 customers by revenue"
      },
      {
        role: "assistant",
        content: [
          {
            type: "text",
            text: "I'll query the purchase history and analyze the results."
          },
          {
            type: "server_tool_use",
            id: "srvtoolu_abc123",
            name: "code_execution",
            input: { code: "..." }
          },
          {
            type: "tool_use",
            id: "toolu_def456",
            name: "query_database",
            input: { sql: "<sql>" },
            caller: {
              type: "code_execution_20260120",
              tool_id: "srvtoolu_abc123"
            }
          }
        ]
      },
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: "toolu_def456",
            content: '[{"customer_id": "C1", "revenue": 45000}, {"customer_id": "C2", "revenue": 38000}, ...]'
          }
        ]
      }
    ],
    # 与原始请求相同的 tools 数组
    tools: [
      {
        type: "code_execution_20260120",
        name: "code_execution"
      },
      {
        name: "query_database",
        description: "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
        input_schema: {
          type: "object",
          properties: {
            sql: {
              type: "string",
              description: "SQL query to execute"
            }
          },
          required: ["sql"]
        },
        allowed_callers: ["code_execution_20260120"]
      }
    ]
  )

  puts message
  ```
</CodeGroup>

### 第 4 步：下一次工具调用或完成

代码从暂停处继续执行并处理您的结果。每个续接响应要么带着更多程序化 `tool_use` 块再次暂停，要么完成代码执行并让 Claude 继续本轮（步骤 5）。检查 `stop_reason` 以及每个 `tool_use` 块的 `caller` 来区分这两种情况。等待您处理的暂停响应具有 `stop_reason: "tool_use"`，以及一个 `caller` 指明代码执行版本的 `tool_use` 块。在这种情况下，请重复步骤 3，在一条用户消息中为每个待处理的程序化调用提供 `tool_result`。

### 第 5 步：最终响应

代码执行完成后，Claude 提供最终响应：

```json Output
{
  "content": [
    {
      "type": "code_execution_tool_result",
      "tool_use_id": "srvtoolu_abc123",
      "content": {
        "type": "code_execution_result",
        "stdout": "Top 5 customers: [{'customer_id': 'C1', 'revenue': 45000}, {'customer_id': 'C2', 'revenue': 38000}, {'customer_id': 'C5', 'revenue': 32000}, {'customer_id': 'C8', 'revenue': 28500}, {'customer_id': 'C3', 'revenue': 24000}]",
        "stderr": "",
        "return_code": 0,
        "content": []
      }
    },
    {
      "type": "text",
      "text": "I've analyzed the purchase history from last quarter. Your top 5 customers generated $167,500 in total revenue, with Customer C1 leading at $45,000."
    }
  ],
  "stop_reason": "end_turn"
}
```

## 高级模式

### 使用循环进行批处理

Claude 可以编写高效处理多个项目的代码：

```python
regions = ["West", "East", "Central", "North", "South"]
results = {}
for region in regions:
    rows = json.loads(await query_database({"sql": f"<sql for {region}>"}))
    results[region] = sum(row["revenue"] for row in rows)

# 以编程方式处理结果
top_region = max(results.items(), key=lambda item: item[1])
print(f"Top region: {top_region[0]} with ${top_region[1]:,} in revenue")
```

此模式：

* 将模型往返次数从 N 次（每个区域一次）减少到 1 次
* 在返回给 Claude 之前以编程方式处理大型结果集
* 通过仅返回汇总结论而非原始数据来节省令牌

### 提前终止

一旦满足成功条件，Claude 即可停止处理：

```python
endpoints = ["us-east", "eu-west", "apac"]
for endpoint in endpoints:
    status = await check_health({"endpoint": endpoint})
    if status == "healthy":
        print(f"Found healthy endpoint: {endpoint}")
        break  # Stop early, don't check remaining
```

### 条件式工具选择

```python
path = "/tmp/example.txt"
file_info = json.loads(await get_file_info({"path": path}))
if file_info["size"] < 10000:
    content = await read_full_file({"path": path})
else:
    content = await read_file_summary({"path": path})
print(content)
```

### 数据过滤

```python
server_id = "srv-01"
log_text = await fetch_logs({"server_id": server_id})
errors = [line for line in log_text.splitlines() if "ERROR" in line]
print(f"Found {len(errors)} errors")
for error in errors[-10:]:  # Only return last 10 errors
    print(error)
```

## 响应格式

### 编程式工具调用

当代码执行调用工具时：

```json
{
  "type": "tool_use",
  "id": "toolu_abc123",
  "name": "query_database",
  "input": { "sql": "<sql>" },
  "caller": {
    "type": "code_execution_20260120",
    "tool_id": "srvtoolu_xyz789"
  }
}
```

### 工具结果处理

您的工具结果会被传回正在运行的代码：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_abc123",
      "content": "[{\"customer_id\": \"C1\", \"revenue\": 45000, \"orders\": 23}, {\"customer_id\": \"C2\", \"revenue\": 38000, \"orders\": 18}, ...]"
    }
  ]
}
```

### 代码执行完成

当所有工具调用均已满足且代码完成时：

```json
{
  "type": "code_execution_tool_result",
  "tool_use_id": "srvtoolu_xyz789",
  "content": {
    "type": "code_execution_result",
    "stdout": "Analysis complete. Top 5 customers identified from 847 total records.",
    "stderr": "",
    "return_code": 0,
    "content": []
  }
}
```

## 错误处理

### 常见错误

| 错误                                        | 出现位置                                                | 描述                                                        | 解决方案                                                                                                       |
| ----------------------------------------- | --------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `invalid_tool_input`                      | 响应中 `code_execution_tool_result` 错误块上的 `error_code` | 向代码执行工具传递了无效参数                                            | 请参阅[代码执行工具错误](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#errors) |
| `invalid_request_error`（针对 `tool_choice`） | HTTP 400 错误响应                                       | `tool_choice` 指定了一个其 `allowed_callers` 不包含 `"direct"` 的工具 | 将 `"direct"` 添加到该工具的 `allowed_callers` 中，或者从 `tool_choice` 中移除该工具，让 Claude 从代码中调用它                         |

### 工具调用期间容器过期

如果您的工具结果未在大约 4 分钟内到达，待处理的调用会在 Claude 正在运行的代码中引发 `TimeoutError`。Claude 会在 `stderr` 中收到该错误，并通常会重试调用：

```json
{
  "type": "code_execution_tool_result",
  "tool_use_id": "srvtoolu_abc123",
  "content": {
    "type": "code_execution_result",
    "stdout": "",
    "stderr": "TimeoutError: Calling tool ['query_database'] timed out (no response after 270s).",
    "return_code": 0,
    "content": []
  }
}
```

为防止超时：

* 监控暂停响应中的 `expires_at` 字段。
* 为您的工具执行实现超时机制。
* 考虑将长时间运行的操作拆分为较小的块。

### 工具执行错误

如果您的工具返回错误：

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_abc123",
  "content": "Error: Query timeout - table lock exceeded 30 seconds"
}
```

Claude 的代码会收到此错误并可以妥善处理。

## 约束与限制

### 功能不兼容

* **结构化输出：** 程序化调用不支持带有 `strict: true` 的工具。
* **工具选择：** 您无法通过 `tool_choice` 强制以程序化方式调用特定工具。
* **并行工具使用：** 程序化调用不支持 `disable_parallel_tool_use: true`。

### 输入模式限制

`input_schema` 中包含递归 `$ref`（引用循环，例如引用自身的模式）的自定义工具无法启用编程式调用。在此类工具的 `allowed_callers` 中包含代码执行工具版本会导致请求失败，返回 `400 invalid_request_error`，其消息包含 `Circular $ref detected`。相同的模式在直接工具调用中是被接受的。

要解决此问题，请执行以下操作之一：

* 通过省略 `allowed_callers`（或将其设置为 `["direct"]`）使该工具仅支持直接调用。同一请求中的其他工具仍可使用编程式调用。
* 从模式中移除循环。例如，将递归展开到固定深度，并在最内层的 `description` 中描述任何更深的嵌套；或者将递归属性替换为一个普通的 `{"type": "object"}`，并在其 `description` 中说明预期的结构。

### 工具限制

以下工具无法以编程方式调用：

* 由 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)提供的工具
* [计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)工具集（`computer_toolset_20260801` 和 `browser_toolset_20260801`），其 `allowed_callers` 字段仅接受 `"direct"`

### 消息格式限制

在响应编程式工具调用时，有严格的格式要求：

**仅包含工具结果的响应：** 如果存在等待结果的待处理编程式工具调用，您的响应消息必须**仅**包含 `tool_result` 块。您不能包含任何文本内容，即使是在工具结果之后。

无效（响应程序化工具调用时不能包含文本）：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01",
      "content": "[{\"customer_id\": \"C1\", \"revenue\": 45000}]"
    },
    { "type": "text", "text": "What should I do next?" }
  ]
}
```

有效（响应程序化工具调用时仅包含工具结果）：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01",
      "content": "[{\"customer_id\": \"C1\", \"revenue\": 45000}]"
    }
  ]
}
```

此限制仅适用于响应编程式（代码执行）工具调用的情况。对于常规的客户端工具调用，您可以在工具结果之后包含文本内容。

**仅限文本的工具结果内容：** 回应编程式调用的每个 `tool_result` 的 `content` 必须是字符串或 `text` 块。图像、文档和其他内容块类型会被拒绝。

### 速率限制

程序化工具调用与常规工具调用受相同的速率限制约束。来自代码执行的每个工具调用都会单独计入这些限制。

### 使用前验证工具结果

在实现 Claude 以编程方式调用的用户自定义工具时：

* **工具结果以字符串形式返回：** 它们可以包含任何内容，包括可能被执行环境处理的代码片段或可执行命令。
* **验证外部工具结果：** 如果您的工具返回来自外部来源的数据或接受用户输入，并且输出会被解释或作为代码运行，请注意代码注入风险。

## 令牌效率

编程式工具调用通过三种方式减少令牌消耗：

* **编程式调用的工具结果不会添加到 Claude 的上下文中：** 只有最终的代码输出会被添加。
* **中间处理在代码中进行：** 过滤、聚合和其他转换不消耗模型令牌。
* **一次代码执行中进行多次工具调用：** 与分开的多个模型轮次相比，这减少了开销。

例如，直接调用 10 个工具所使用的令牌约为以编程方式调用它们并返回摘要的 10 倍。

在 Anthropic 对生产环境 Claude 模型的内部评估中：

* 在一个包含 75 个工具的项目管理智能体基准测试中，启用程序化工具调用使计费输入令牌减少了约 38%，且任务准确率没有变化。
* 在 [τ²-bench](https://arxiv.org/abs/2506.07982)（航空、零售和电信领域）上，每轮进行一到两次顺序工具调用，程序化工具调用使得分保持不变，成本增加约 8%。顺序单次调用的工作流无法从中受益。
* 在生产 API 流量中，`tools` 数组包含 10 到 49 个工具定义的请求，在启用程序化工具调用后通常可节省 20% 到 40% 的令牌。

实际节省量因工作负载形态而异。请参阅[何时使用编程式调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#when-to-use-programmatic-calling)。

## 用量与定价

编程式工具调用采用与代码执行相同的定价。详情请参阅[代码执行定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#usage-and-pricing)。

<Note>
  程序化工具调用的令牌计数：程序化调用的工具结果不计入您的输入/输出令牌用量。只有最终的代码执行结果和 Claude 的响应才会计入。
</Note>

## 最佳实践

### 工具设计

* **提供详细的输出描述：** 由于 Claude 在代码中反序列化工具结果，请记录格式（JSON 结构和字段类型）。
* **返回结构化数据：** JSON 或其他机器可读格式最适合程序化处理。
* **保持响应简洁：** 仅返回必要的数据，以最大限度地减少处理开销。

### 何时使用编程式调用

程序化工具调用以少量固定开销（容器启动、脚本生成）换取在工具结果令牌和模型往返上的大量节省。这种权衡是否值得取决于工作负载形态。

**非常适合：**

* 跨多个项目的扇出或并行操作（例如，检查 50 个端点或查找 20 条记录）
* 可在进入 Claude 上下文之前进行过滤、汇总或摘要的大型工具结果
* 智能体搜索和检索，其中迭代查询和结果过滤主导整个工作流

**不太适合：**

* 严格顺序的工作流，其中每次调用都依赖于 Claude 对前一结果的推理，因为在这种情况下脚本无法跳过模型往返
* 少量工具调用且响应较小，尤其是在对话的第一轮，此时容器和脚本开销可能超过节省量
* 需要在调用之间立即获得用户反馈的工具

如果您不确定，请在广泛启用之前，在具有代表性的流量样本上分别测量使用和不使用 `allowed_callers` 时的计费输入令牌。

### 性能优化

* 在发出多个相关请求时**重用容器**以保持状态。
* 尽可能在单次代码执行中**批量处理类似操作**。

## 故障排除

### 常见问题

**设置 `tool_choice` 时出现 `invalid_request_error`**

* `tool_choice` 不能指定其 `allowed_callers` 省略了 `"direct"` 的工具。将 `"direct"` 添加到该工具的 `allowed_callers` 中，或者从 `tool_choice` 中移除该工具，让 Claude 从代码中调用它。

**容器过期**

* 请在暂停响应的 `expires_at` 时间戳之前尽早响应每个编程式工具调用。Claude 的代码在大约 4 分钟后停止等待结果。
* 考虑实现更快的工具执行。

**工具结果未被正确解析**

* 确保您的工具返回 Claude 可以反序列化的字符串数据。
* 在工具描述中提供清晰的输出格式文档。

### 调试技巧

1. **记录所有工具调用和结果**以跟踪流程。
2. **检查 `caller` 字段**以确认是程序化调用。
3. **监控容器 ID** 以确保正确重用。
4. 在启用程序化调用之前**独立测试工具**。

## 编程式工具调用为何有效

Claude 在大量代码上进行过训练，因此将工具呈现为可调用的 Python 函数可以让它发挥这一优势：

* **工具组合：** 链式调用、循环和条件语句是普通的 Python 控制流，而不是一系列模型往返。
* **结果处理：** Claude 的代码会过滤和聚合大型工具输出，或将其写入文件，只有最终输出进入上下文窗口。
* **延迟：** 在一次代码执行内的工具调用之间不会重新对模型进行采样。

## 替代实现方案

编程式工具调用是一种可泛化的模式，也可以在您自己的基础设施上实现。以下是各种方法的比较：

### 客户端直接执行

为 Claude 提供代码执行工具，并描述该环境中可用的函数。当 Claude 使用代码调用该工具时，您的应用程序会在定义了这些函数的本地环境中运行它。

**优点：**

* 对应用程序的重构最少
* 完全控制环境和指令

**缺点：**

* 在沙盒之外运行不受信任的代码
* 工具调用可能成为代码注入的载体

**适用场景：** 您的应用程序可以安全地运行任意代码，您希望实现最简单，并且 Anthropic 的托管方案不符合您的需求。

### 自行管理的沙盒执行

从 Claude 的角度看方法相同，但代码在具有安全限制（例如，无网络出口）的沙盒容器中运行。如果您的工具需要外部资源，您需要一种在沙盒外运行工具调用的协议。

**优点：**

* 在您自己的基础设施上安全地进行编程式工具调用
* 完全控制执行环境

**缺点：**

* 构建和维护复杂
* 需要同时管理基础设施和进程间通信

**适用场景：** 安全性至关重要，且 Anthropic 的托管解决方案不符合您的要求。

### Anthropic 托管执行

Anthropic 的程序化工具调用是沙盒执行的托管版本，提供针对 Claude 调优的特定 Python 环境。Anthropic 负责容器管理、代码执行和安全的工具调用通信。

**优点：**

* 默认安全可靠
* 通过工具定义即可启用，无需运行任何基础设施
* 环境和指令针对 Claude 进行了优化

如果您正在使用 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 或 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)，请考虑使用 Anthropic 的托管解决方案。在 Microsoft Foundry 上，编程式工具调用需要[托管在 Anthropic 的部署](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry#additional-features-not-supported-when-hosted-on-azure)。

## 数据保留

编程式工具调用构建在代码执行基础设施之上，并使用相同的沙盒容器。容器数据（包括执行产物和输出）最多保留 30 天。

## 后续步骤

<CardGroup cols={2}>
  <Card title="细粒度工具流式传输" icon="bolt" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming">
    为延迟敏感型应用流式传输工具输入，无需服务器端 JSON 缓冲。
  </Card>

  <Card title="代码执行工具" icon="code" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool">
    在沙盒容器中运行 Python 和 Bash 代码，以分析数据、生成文件并迭代解决方案。
  </Card>

  <Card title="使用 Claude 进行工具使用" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview">
    将 Claude 连接到外部工具和 API。了解工具在何处运行、Claude 何时调用它们，以及哪种工具适合您的任务。
  </Card>

  <Card title="定义工具" icon="hammer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools">
    指定工具模式、编写有效的描述，并控制 Claude 何时调用您的工具。
  </Card>
</CardGroup>

## 可用性

编程式工具调用在部分模型上可用。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台                                                                                                                                                                                                                                                    | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Microsoft Foundry (hosted on Anthropic)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview), [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview), [Claude Sonnet 4.5](https://platform.claude.com/docs/zh-CN/models/sonnet-4-5/overview) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 不具备资格                                                                                                                                     | 不具备资格 |
| 详细信息 | 基于代码执行容器构建；数据最多保留 30 天。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#data-retention) | —     |

适用的端点：

* `/v1/messages` 使用 `code_execution` 工具
