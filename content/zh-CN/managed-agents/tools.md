---
title: 工具
url: https://platform.claude.com/docs/zh-CN/managed-agents/tools
description: 配置您的智能体可用的工具。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  status: beta
  betaHeader: managed-agents-2026-04-01
  zdr:
    eligibility: not-eligible
    note: 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。
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
    Claude API: beta
    Claude Platform on AWS: beta
---

Claude Managed Agents 提供了一组内置工具，Claude 可以在[会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)中自主使用这些工具。您可以通过在智能体配置中指定工具来控制哪些工具可用。

Claude Managed Agents 还支持自定义的用户定义工具。您的应用程序会单独运行这些工具，并将结果返回给 Claude，Claude 使用这些结果继续完成任务。如需为智能体提供来自 MCP 服务器的工具，请改用 [MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)。

## 可用工具

智能体工具集包含以下工具。当您在智能体配置中包含该工具集时，所有工具默认启用。

| 工具    | 名称           | 描述                    |
| ----- | ------------ | --------------------- |
| Bash  | `bash`       | 在 shell 会话中运行 bash 命令 |
| Read  | `read`       | 从沙箱文件系统读取文件           |
| Write | `write`      | 向沙箱文件系统写入文件           |
| Edit  | `edit`       | 在文件中执行字符串替换           |
| Glob  | `glob`       | 使用 glob 模式执行快速文件模式匹配  |
| Grep  | `grep`       | 使用正则表达式模式搜索文本         |
| 网页抓取  | `web_fetch`  | 从 URL 抓取内容            |
| 网页搜索  | `web_search` | 在网络上搜索信息              |

当工具输出超过 100,000 个字符（约 25,000 个令牌）时，它会自动写入[沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/environments)中的文件。模型会收到一个包含文件路径的截断预览，并可以从该路径读取完整内容。

## 配置工具集

创建智能体时，使用 `agent_toolset_20260401` 启用完整工具集。使用 `configs` (csharp, go: `Configs`) 数组来禁用特定工具或覆盖其设置。每个条目通过其 `name` (csharp, go: `Name`) 进行标识，其值取自[可用工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#available-tools)中的"名称"列。条目还接受一个可选的 `type` (csharp, go: `Type`) 字段，其值与名称相同。

每个配置条目还可以设置 `permission_policy` (java, php: `permissionPolicy`; csharp, go: `PermissionPolicy`)。该策略控制工具调用是无需确认即可运行、需要确认，还是由服务器逐一评估。有关可用的策略类型，请参阅[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)。

以下示例启用工具集并禁用 `web_fetch`：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Coding Assistant",
      "model": "claude-opus-5-5",
      "tools": [
        {
          "type": "agent_toolset_20260401",
          "configs": [
            {"name": "web_fetch", "enabled": false}
          ]
        }
      ]
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply agent.md
    ```

    <File filename="agent.md">
      ```markdown
      ---
      name: Coding Assistant
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
          configs:
            - name: web_fetch
              enabled: false
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Coding Assistant",
      model="claude-opus-5-5",
      tools=[
          {
              "type": "agent_toolset_20260401",
              "configs": [
                  {"name": "web_fetch", "enabled": False},
              ],
          },
      ],
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Coding Assistant",
    model: "claude-opus-5-5",
    tools: [
      {
        type: "agent_toolset_20260401",
        configs: [{ name: "web_fetch", enabled: false }]
      }
    ]
  });
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Agents;

  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Coding Assistant",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      Tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params
          {
              Configs =
              [
                  new BetaManagedAgentsWebFetchToolConfigParams { Enabled = false },
              ],
          },
      ],
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Coding Assistant",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
  			Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{{
  				OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{
  					Enabled: anthropic.Bool(false),
  				},
  			}},
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.agents.*;

  var agent = client.beta().agents().create(AgentCreateParams.builder()
      .name("Coding Assistant")
      .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
      .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
          .addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()
              .enabled(false)
              .build())
          .build())
      .build());
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsModel;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebFetchToolConfigParams;

  $agent = $client->beta->agents->create(
      name: 'Coding Assistant',
      model: BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(
              configs: [
                  BetaManagedAgentsWebFetchToolConfigParams::with(enabled: false),
              ],
          ),
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Coding Assistant",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
    tools: [
      {
        type: :agent_toolset_20260401,
        configs: [
          {name: :web_fetch, enabled: false}
        ]
      }
    ]
  )
  ```
</CodeGroup>

### 禁用特定工具

要禁用某个工具，请在其 `configs` 条目中设置 `enabled: false`：

```json
{
  "type": "agent_toolset_20260401",
  "configs": [
    { "name": "web_fetch", "enabled": false },
    { "name": "web_search", "enabled": false }
  ]
}
```

### 仅启用特定工具

`default_config` 对象为工具集中的每个工具设置基线，而每个工具的 `configs` 条目会覆盖它。若要从全部关闭开始并仅启用您需要的工具，请将 `default_config.enabled` 设置为 `false`：

```json
{
  "type": "agent_toolset_20260401",
  "default_config": { "enabled": false },
  "configs": [
    { "name": "bash", "enabled": true },
    { "name": "read", "enabled": true },
    { "name": "write", "enabled": true }
  ]
}
```

### 限制网页搜索和网页抓取

`web_search` 和 `web_fetch` 条目还接受域名列表、抓取内容的上限以及搜索位置。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

### SDK 中的配置条目类型

SDK 按工具为每个 `configs` (csharp, go: `Configs`) 条目指定类型。该类型是一个联合类型，每个内置工具对应一个成员，并通过 `type` (csharp, go: `Type`) 进行区分。

在 Go、Java、C# 和 PHP 中，您需要使用类型化的值而不是普通的字典或哈希来构造条目。在这些 SDK 中，`configs` (csharp, go: `Configs`) 的元素类型就是该联合类型本身，因此请使用每个工具对应的成员类型来构建各个条目。

构造条目时，您可以省略 `type` (csharp, go: `Type`)，因为服务器会根据 `name` (csharp, go: `Name`) 推断它。响应中始终会包含该字段。这种类型定义不会改变条目序列化后的 JSON。如果请求中的条目仅设置了 `name`、`enabled` 和 `permission_policy`，则无论是否包含 `type`，该请求都是有效的。

## 自定义工具

自定义工具类似于 Messages API 中的[用户定义的客户端工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/how-tool-use-works#user-defined-tools-client-executed)。您指定可用的操作及其返回内容，由 Claude 决定何时以及如何调用它们。

Claude 不会自行运行自定义工具。它会发出一个结构化请求，由您的代码执行该操作，然后您将结果返回给会话。

以下示例创建了一个包含一个自定义工具 `get_weather` 且不含任何内置工具的智能体：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Weather Agent",
      "model": "claude-opus-5-5",
      "tools": [
        {
          "type": "custom",
          "name": "get_weather",
          "description": "Get current weather for a location",
          "input_schema": {
            "type": "object",
            "properties": {
              "location": {"type": "string", "description": "City name"}
            },
            "required": ["location"]
          }
        }
      ]
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply agent.md
    ```

    <File filename="agent.md">
      ```markdown
      ---
      name: Weather Agent
      model: claude-opus-5-5
      tools:
        - type: custom
          name: get_weather
          description: Get current weather for a location
          input_schema:
            type: object
            properties:
              location:
                type: string
                description: City name
            required:
              - location
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Weather Agent",
      model="claude-opus-5-5",
      tools=[
          {
              "type": "custom",
              "name": "get_weather",
              "description": "Get current weather for a location",
              "input_schema": {
                  "type": "object",
                  "properties": {
                      "location": {"type": "string", "description": "City name"},
                  },
                  "required": ["location"],
              },
          },
      ],
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Weather Agent",
    model: "claude-opus-5-5",
    tools: [
      {
        type: "custom",
        name: "get_weather",
        description: "Get current weather for a location",
        input_schema: {
          type: "object",
          properties: { location: { type: "string", description: "City name" } },
          required: ["location"]
        }
      }
    ]
  });
  ```

  ```csharp C#
  using System.Text.Json;
  using Anthropic.Models.Beta.Agents;

  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Weather Agent",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      Tools =
      [
          new BetaManagedAgentsCustomToolParams
          {
              Name = "get_weather",
              Description = "Get current weather for a location",
              InputSchema = new()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["location"] = JsonSerializer.SerializeToElement(
                          new { type = "string", description = "City name" }
                      ),
                  },
                  Required = ["location"],
              },
          },
      ],
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Weather Agent",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfCustom: &anthropic.BetaManagedAgentsCustomToolParams{
  			Name:        "get_weather",
  			Description: "Get current weather for a location",
  			InputSchema: anthropic.BetaManagedAgentsCustomToolInputSchemaParam{
  				Properties: map[string]any{
  					"location": map[string]any{
  						"type":        "string",
  						"description": "City name",
  					},
  				},
  				Required: []string{"location"},
  			},
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.agents.*;
  import java.util.Map;

  var agent = client.beta().agents().create(AgentCreateParams.builder()
      .name("Weather Agent")
      .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
      .addTool(BetaManagedAgentsCustomToolParams.builder()
          .name("get_weather")
          .description("Get current weather for a location")
          .inputSchema(BetaManagedAgentsCustomToolInputSchema.builder()
              .properties(BetaManagedAgentsCustomToolInputSchema.Properties.builder()
                  .putAdditionalProperty("location", JsonValue.from(Map.of(
                      "type", "string",
                      "description", "City name")))
                  .build())
              .addRequired("location")
              .build())
          .build())
      .build());
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsCustomToolInputSchema;
  use Anthropic\Beta\Agents\BetaManagedAgentsCustomToolParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsModel;

  $agent = $client->beta->agents->create(
      name: 'Weather Agent',
      model: BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
      tools: [
          BetaManagedAgentsCustomToolParams::with(
              name: 'get_weather',
              description: 'Get current weather for a location',
              inputSchema: BetaManagedAgentsCustomToolInputSchema::with(
                  properties: ['location' => ['type' => 'string', 'description' => 'City name']],
                  required: ['location'],
              ),
          ),
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Weather Agent",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
    tools: [
      {
        type: :custom,
        name: "get_weather",
        description: "Get current weather for a location",
        input_schema: {
          type: :object,
          properties: {location: {type: "string", description: "City name"}},
          required: ["location"]
        }
      }
    ]
  )
  ```
</CodeGroup>

自定义工具的名称不能以 `mcp__` 或 `ant__` 开头，也不能与内置智能体工具的名称相同。违反此规则的请求会返回 400 错误。如果某个智能体已有名称以 `ant__` 开头的自定义工具，则只有当更新发送的 `tools` (csharp, go: `Tools`) 中不包含该名称时，才会接受该更新，因此请在下一次更新中重命名或移除该工具。如果此类工具存在于新会话的智能体上（在应用任何[覆盖](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)之后）、存在于 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中的某个智能体上，或存在于 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中的某个智能体上，则该新会话会被拒绝并返回 400 错误。已存在的会话不受影响。

智能体在会话期间调用其自定义工具。要接收调用并返回结果，请参阅[响应会暂停会话的工具调用](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#handling-custom-tool-calls)。

对于在自托管沙箱中运行的会话，环境工作进程可以[从沙箱提供自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools)。这些工具可以包括封装您网络内部 MCP 服务器的工具。

### 自定义工具定义的最佳实践

* **提供极其详细的描述。** 这是迄今为止影响工具性能的最重要因素。您为 Claude 提供的工具上下文越多，它就越能判断何时以及如何使用这些工具。每个工具描述应力求包含三到四句话，如果工具较为复杂则应更多。您的描述应涵盖：

  * 工具的功能
  * 何时使用它，以及何时不应使用它
  * 每个参数的含义及其如何影响工具的行为
  * 任何重要的注意事项或限制

* **将相关操作整合到更少的工具中。** 将 `create_pr`、`review_pr` 和 `merge_pr` 等操作归入一个带有 `action` 参数的单一工具中。更少但功能更强的工具可以减少选择上的歧义，并使 Claude 更容易驾驭您的工具集。

* **在工具名称中使用有意义的命名空间。** 当您的工具跨越多个服务或资源时，请以资源作为名称前缀（例如 `db_query` 或 `storage_read`）。随着您的工具库增长，这可以使工具选择毫无歧义。

* **设计工具响应时仅返回高信号信息。** 返回语义化、稳定的标识符（例如 slug 或 UUID），而不是不透明的内部引用。仅包含 Claude 确定下一步所需的字段。臃肿的响应会浪费上下文，并使 Claude 更难提取关键信息。

## 后续步骤

<CardGroup cols={2}>
  <Card title="限制网页搜索和网页抓取的域名" icon="shield" href="https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions">
    控制网页工具可以访问哪些站点、限制抓取的内容量，并对搜索结果进行本地化。
  </Card>

  <Card title="MCP 连接器" icon="link" href="https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector">
    将 MCP 服务器连接到您的智能体，以访问外部工具和数据源。
  </Card>

  <Card title="权限策略" icon="lock" href="https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies">
    控制智能体工具和 MCP 工具何时运行。
  </Card>

  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    在会话运行时发送事件、流式传输响应，以及中断或重定向会话。
  </Card>
</CardGroup>

## 可用性

智能体工具集在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                         | HIPAA                             |
| ---- | ------------------------------------------------------------- | --------------------------------- |
| 状态   | 不具备资格                                                         | 不具备资格                             |
| 详细信息 | 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。 | 适用于所有 Managed Agents 子功能，包括自托管沙箱。 |

适用的端点：

* `/v1/agents`
* `/v1/sessions`
* `/v1/environments`
