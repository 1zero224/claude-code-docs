---
title: 权限策略
url: https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies
description: 控制智能体工具和 MCP 工具的运行时机。
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
    Claude Models on Google Cloud (Anthropic managed): beta
---

“Permission policy”（权限策略）控制服务器执行的工具（预构建的智能体工具集和 MCP 工具集）是自动运行、等待您的批准，还是由服务器评估每次调用。自定义工具由您的应用程序运行并由您控制，因此不受权限策略约束。

## 权限策略类型

| 策略             | 行为                                                                                                                                                                                  |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `always_allow` | 工具自动执行，无需确认。                                                                                                                                                                        |
| `always_ask`   | 会话暂停，并在执行前等待您的批准。有关事件流程，请参阅[响应确认请求](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#respond-to-confirmation-requests)。                                    |
| `auto`         | 服务器评估每次调用，然后运行该调用、拒绝该调用或暂停以等待您的批准。请参阅[使用 `auto` 让服务器评估每次调用](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#let-the-server-evaluate-each-call-with-auto)。 |

每种工具集类型都有自己的默认值：智能体工具集默认为 `always_allow`，MCP 工具集默认为 `always_ask`。

权限策略控制已启用的工具何时运行。若要将某个工具从智能体中完全移除，请改为禁用它。请参阅[禁用特定工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#disabling-specific-tools)。

## 为工具集设置策略

您在创建智能体时，在智能体的 `tools` (csharp, go: `Tools`) 配置中设置权限策略，之后可以通过[更新智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-an-agent)来更改它们。正在运行的会话会保留其创建时的工具集配置。更新将应用于之后创建的会话。

### 智能体工具集权限

创建智能体时，您可以使用 `default_config.permission_policy` (java, php: `defaultConfig.permissionPolicy`; csharp, go: `DefaultConfig.PermissionPolicy`) 为 `agent_toolset_20260401` 中的每个工具应用策略：

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
          "default_config": {
            "permission_policy": {"type": "always_ask"}
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
      name: Coding Assistant
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
          default_config:
            permission_policy:
              type: always_ask
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
              "default_config": {
                  "permission_policy": {"type": "always_ask"},
              },
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
        default_config: {
          permission_policy: { type: "always_ask" }
        }
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
              DefaultConfig = new()
              {
                  PermissionPolicy = new BetaManagedAgentsAlwaysAskPolicy(),
              },
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
  			DefaultConfig: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParams{
  				PermissionPolicy: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParamsPermissionPolicyUnion{
  					OfAlwaysAsk: &anthropic.BetaManagedAgentsAlwaysAskPolicyParam{},
  				},
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

  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Coding Assistant")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .addTool(
              BetaManagedAgentsAgentToolset20260401Params.builder()
                  .defaultConfig(
                      BetaManagedAgentsAgentToolsetDefaultConfigParams.builder()
                          .permissionPolicy(BetaManagedAgentsAlwaysAskPolicy.builder().build())
                          .build()
                  )
                  .build()
          )
          .build()
  );
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolsetDefaultConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsAlwaysAskPolicy;

  $agent = $client->beta->agents->create(
      name: 'Coding Assistant',
      model: 'claude-opus-5-5',
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(
              defaultConfig: BetaManagedAgentsAgentToolsetDefaultConfigParams::with(
                  permissionPolicy: BetaManagedAgentsAlwaysAskPolicy::with(),
              ),
          ),
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Coding Assistant",
    model: "claude-opus-5-5",
    tools: [
      {
        type: "agent_toolset_20260401",
        default_config: {
          permission_policy: {type: "always_ask"}
        }
      }
    ]
  )
  ```
</CodeGroup>

`default_config` (java, php: `defaultConfig`; csharp, go: `DefaultConfig`) 是可选的。如果省略它，智能体工具集将以默认权限策略 `always_allow` 启用。

### MCP 工具集权限

MCP 工具集默认为 `always_ask`。这可以确保添加到 MCP 服务器的新工具不会在未经批准的情况下在您的应用程序中运行。要自动批准来自受信任 MCP 服务器的工具，请在 `mcp_toolset` 条目上设置 `default_config.permission_policy` (java, php: `defaultConfig.permissionPolicy`; csharp, go: `DefaultConfig.PermissionPolicy`)。

`mcp_server_name` (java, php: `mcpServerName`; csharp: `McpServerName`; go: `MCPServerName`) 必须与 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 数组中某个服务器的 `name` (csharp, go: `Name`) 相匹配。

此示例连接一个 GitHub MCP 服务器，并允许其工具无需确认即可运行：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Dev Assistant",
      "model": "claude-opus-5-5",
      "mcp_servers": [
        {"type": "url", "name": "github", "url": "https://mcp.example.com/github"}
      ],
      "tools": [
        {"type": "agent_toolset_20260401"},
        {
          "type": "mcp_toolset",
          "mcp_server_name": "github",
          "default_config": {
            "permission_policy": {"type": "always_allow"}
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
      name: Dev Assistant
      model: claude-opus-5-5
      mcp_servers:
        - type: url
          name: github
          url: https://mcp.example.com/github
      tools:
        - type: agent_toolset_20260401
        - type: mcp_toolset
          mcp_server_name: github
          default_config:
            permission_policy:
              type: always_allow
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Dev Assistant",
      model="claude-opus-5-5",
      mcp_servers=[
          {"type": "url", "name": "github", "url": "https://mcp.example.com/github"},
      ],
      tools=[
          {"type": "agent_toolset_20260401"},
          {
              "type": "mcp_toolset",
              "mcp_server_name": "github",
              "default_config": {
                  "permission_policy": {"type": "always_allow"},
              },
          },
      ],
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Dev Assistant",
    model: "claude-opus-5-5",
    mcp_servers: [{ type: "url", name: "github", url: "https://mcp.example.com/github" }],
    tools: [
      { type: "agent_toolset_20260401" },
      {
        type: "mcp_toolset",
        mcp_server_name: "github",
        default_config: {
          permission_policy: { type: "always_allow" }
        }
      }
    ]
  });
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Agents;

  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Dev Assistant",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      McpServers =
      [
          new()
          {
              Name = "github",
              Url = "https://mcp.example.com/github",
          },
      ],
      Tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params(),
          new BetaManagedAgentsMcpToolsetParams
          {
              McpServerName = "github",
              DefaultConfig = new()
              {
                  PermissionPolicy = new BetaManagedAgentsAlwaysAllowPolicy(),
              },
          },
      ],
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Dev Assistant",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	MCPServers: []anthropic.BetaManagedAgentsURLMCPServerParams{{
  		Name: "github",
  		URL:  "https://mcp.example.com/github",
  	}},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{
  		{
  			OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{},
  		},
  		{
  			OfMCPToolset: &anthropic.BetaManagedAgentsMCPToolsetParams{
  				MCPServerName: "github",
  				DefaultConfig: anthropic.BetaManagedAgentsMCPToolsetDefaultConfigParams{
  					PermissionPolicy: anthropic.BetaManagedAgentsMCPToolsetDefaultConfigParamsPermissionPolicyUnion{
  						OfAlwaysAllow: &anthropic.BetaManagedAgentsAlwaysAllowPolicyParam{},
  					},
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.agents.*;

  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Dev Assistant")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .addMcpServer(
              BetaManagedAgentsUrlMcpServerParams.builder()
                  .name("github")
                  .url("https://mcp.example.com/github")
                  .build()
          )
          .addTool(BetaManagedAgentsAgentToolset20260401Params.builder().build())
          .addTool(
              BetaManagedAgentsMcpToolsetParams.builder()
                  .mcpServerName("github")
                  .defaultConfig(
                      BetaManagedAgentsMcpToolsetDefaultConfigParams.builder()
                          .permissionPolicy(BetaManagedAgentsAlwaysAllowPolicy.builder().build())
                          .build()
                  )
                  .build()
          )
          .build()
  );
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsAlwaysAllowPolicy;
  use Anthropic\Beta\Agents\BetaManagedAgentsMCPToolsetDefaultConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsMCPToolsetParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsURLMCPServerParams;

  $agent = $client->beta->agents->create(
      name: 'Dev Assistant',
      model: 'claude-opus-5-5',
      mcpServers: [
          BetaManagedAgentsURLMCPServerParams::with(
              name: 'github',
              url: 'https://mcp.example.com/github',
          ),
      ],
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(),
          BetaManagedAgentsMCPToolsetParams::with(
              mcpServerName: 'github',
              defaultConfig: BetaManagedAgentsMCPToolsetDefaultConfigParams::with(
                  permissionPolicy: BetaManagedAgentsAlwaysAllowPolicy::with(),
              ),
          ),
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Dev Assistant",
    model: "claude-opus-5-5",
    mcp_servers: [
      {type: "url", name: "github", url: "https://mcp.example.com/github"}
    ],
    tools: [
      {type: "agent_toolset_20260401"},
      {
        type: "mcp_toolset",
        mcp_server_name: "github",
        default_config: {
          permission_policy: {type: "always_allow"}
        }
      }
    ]
  )
  ```
</CodeGroup>

## 覆盖单个工具的策略

使用 `configs` (csharp, go: `Configs`) 数组覆盖单个工具的默认值。智能体工具集的 `name` (csharp, go: `Name`) 值列在[可用工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#available-tools)中。此示例默认允许整个智能体工具集，但要求在运行任何 bash 命令之前进行确认：

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
          "default_config": {
            "permission_policy": {"type": "always_allow"}
          },
          "configs": [
            {
              "name": "bash",
              "permission_policy": {"type": "always_ask"}
            }
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
          default_config:
            permission_policy:
              type: always_allow
          configs:
            - name: bash
              permission_policy:
                type: always_ask
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  tools = [
      {
          "type": "agent_toolset_20260401",
          "default_config": {
              "permission_policy": {"type": "always_allow"},
          },
          "configs": [
              {
                  "name": "bash",
                  "permission_policy": {"type": "always_ask"},
              },
          ],
      },
  ]
  ```

  ```typescript TypeScript
  const tools = [
    {
      type: "agent_toolset_20260401",
      default_config: {
        permission_policy: { type: "always_allow" }
      },
      configs: [
        {
          name: "bash",
          permission_policy: { type: "always_ask" }
        }
      ]
    }
  ] satisfies Anthropic.Beta.AgentCreateParams["tools"];
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Agents;
  using Tool = Anthropic.Models.Beta.Agents.Tool;

  Tool[] tools =
  [
      new BetaManagedAgentsAgentToolset20260401Params
      {
          DefaultConfig = new()
          {
              PermissionPolicy = new BetaManagedAgentsAlwaysAllowPolicy(),
          },
          Configs =
          [
              new BetaManagedAgentsBashToolConfigParams
              {
                  PermissionPolicy = new BetaManagedAgentsAlwaysAskPolicy(),
              },
          ],
      },
  ];
  ```

  ```go Go
  tools := []anthropic.BetaAgentNewParamsToolUnion{{
  	OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
  		DefaultConfig: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParams{
  			PermissionPolicy: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParamsPermissionPolicyUnion{
  				OfAlwaysAllow: &anthropic.BetaManagedAgentsAlwaysAllowPolicyParam{},
  			},
  		},
  		Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{{
  			OfBash: &anthropic.BetaManagedAgentsBashToolConfigParams{
  				PermissionPolicy: anthropic.BetaManagedAgentsBashToolConfigParamsPermissionPolicyUnion{
  					OfAlwaysAsk: &anthropic.BetaManagedAgentsAlwaysAskPolicyParam{},
  				},
  			},
  		}},
  	},
  }}
  ```

  ```java Java
  import com.anthropic.models.beta.agents.*;
  import java.util.List;

  var tools = List.of(
      AgentCreateParams.Tool.ofAgentToolset20260401(
          BetaManagedAgentsAgentToolset20260401Params.builder()
              .defaultConfig(
                  BetaManagedAgentsAgentToolsetDefaultConfigParams.builder()
                      .permissionPolicy(BetaManagedAgentsAlwaysAllowPolicy.builder().build())
                      .build()
              )
              .addConfig(
                  BetaManagedAgentsBashToolConfigParams.builder()
                      .permissionPolicy(BetaManagedAgentsAlwaysAskPolicy.builder().build())
                      .build()
              )
              .build()
      )
  );
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsBashToolConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolsetDefaultConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsAlwaysAllowPolicy;
  use Anthropic\Beta\Agents\BetaManagedAgentsAlwaysAskPolicy;

  $tools = [
      BetaManagedAgentsAgentToolset20260401Params::with(
          defaultConfig: BetaManagedAgentsAgentToolsetDefaultConfigParams::with(
              permissionPolicy: BetaManagedAgentsAlwaysAllowPolicy::with(),
          ),
          configs: [
              BetaManagedAgentsBashToolConfigParams::with(
                  permissionPolicy: BetaManagedAgentsAlwaysAskPolicy::with(),
              ),
          ],
      ),
  ];
  ```

  ```ruby Ruby
  tools = [
    {
      type: "agent_toolset_20260401",
      default_config: {
        permission_policy: {type: "always_allow"}
      },
      configs: [
        {
          name: "bash",
          permission_policy: {type: "always_ask"}
        }
      ]
    }
  ]
  ```
</CodeGroup>

在创建智能体的请求中传入此 `tools` (csharp, go: `Tools`) 配置（CLI 选项卡显示了完整命令）。MCP 工具集支持相同的按工具覆盖，其中 `name` (csharp, go: `Name`) 设置为 MCP 服务器报告的工具名称。请参阅[配置可用的 MCP 工具](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector#configure-which-mcp-tools-are-available)。

## 使用 `auto` 让服务器评估每次调用

使用 `auto` 权限策略时，服务器会在每次调用运行之前对其进行评估。由于评估会考虑工具、调用的输入以及截至该时刻的会话内容，服务器可能会以不同方式处理对同一工具的两次调用。每次调用有以下三种结果之一：

* **调用运行。** 当服务器确定调用是安全的时，工具会像在 `always_allow` 下一样运行。
* **调用被拒绝。** 当服务器将调用评估为高风险时，工具不会运行。智能体会收到一个错误工具结果，其内容为 `Permission to use {tool_name} has been denied.` 且 `is_error: true`。会话继续运行，您的客户端无法覆盖该拒绝。
* **调用暂停以等待您的批准。** 当服务器无法做出判定时，会话会像在 `always_ask` 下一样暂停。请参阅[响应确认请求](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#respond-to-confirmation-requests)。

要启用 `auto`，请将 `permission_policy` 设置为 `{"type": "auto"}`。它与其他策略放在相同的两个位置：工具集的 [`default_config` (java, php: `defaultConfig`; csharp, go: `DefaultConfig`)](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#set-a-policy-for-a-toolset)（适用于整个工具集），或 [`configs` (csharp, go: `Configs`) 条目](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#override-an-individual-tool-policy)（适用于单个工具）。智能体工具集和 MCP 工具集都接受它。没有任何工具集默认使用 `auto`。

以下示例将 `auto` 设置为智能体工具集和 `github` MCP 工具集的默认值，并将 `bash` 覆盖为 `always_ask`：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Ops Agent",
      "model": "claude-opus-5-5",
      "mcp_servers": [
        {"type": "url", "name": "github", "url": "https://mcp.example.com/github"}
      ],
      "tools": [
        {
          "type": "agent_toolset_20260401",
          "default_config": {
            "permission_policy": {"type": "auto"}
          },
          "configs": [
            {"name": "bash", "permission_policy": {"type": "always_ask"}}
          ]
        },
        {
          "type": "mcp_toolset",
          "mcp_server_name": "github",
          "default_config": {
            "permission_policy": {"type": "auto"}
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
      name: Ops Agent
      model: claude-opus-5-5
      mcp_servers:
        - type: url
          name: github
          url: https://mcp.example.com/github
      tools:
        - type: agent_toolset_20260401
          default_config:
            permission_policy:
              type: auto
          configs:
            - name: bash
              permission_policy:
                type: always_ask
        - type: mcp_toolset
          mcp_server_name: github
          default_config:
            permission_policy:
              type: auto
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Ops Agent",
      model="claude-opus-5-5",
      mcp_servers=[
          {"type": "url", "name": "github", "url": "https://mcp.example.com/github"},
      ],
      tools=[
          {
              "type": "agent_toolset_20260401",
              "default_config": {
                  "permission_policy": {"type": "auto"},
              },
              "configs": [
                  {"name": "bash", "permission_policy": {"type": "always_ask"}},
              ],
          },
          {
              "type": "mcp_toolset",
              "mcp_server_name": "github",
              "default_config": {
                  "permission_policy": {"type": "auto"},
              },
          },
      ],
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Ops Agent",
    model: "claude-opus-5-5",
    mcp_servers: [{ type: "url", name: "github", url: "https://mcp.example.com/github" }],
    tools: [
      {
        type: "agent_toolset_20260401",
        default_config: {
          permission_policy: { type: "auto" }
        },
        configs: [{ name: "bash", permission_policy: { type: "always_ask" } }]
      },
      {
        type: "mcp_toolset",
        mcp_server_name: "github",
        default_config: {
          permission_policy: { type: "auto" }
        }
      }
    ]
  });
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Agents;

  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Ops Agent",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      McpServers =
      [
          new()
          {
              Name = "github",
              Url = "https://mcp.example.com/github",
          },
      ],
      Tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params
          {
              DefaultConfig = new()
              {
                  PermissionPolicy = new BetaManagedAgentsAutoPolicy(),
              },
              Configs =
              [
                  new BetaManagedAgentsBashToolConfigParams
                  {
                      PermissionPolicy = new BetaManagedAgentsAlwaysAskPolicy(),
                  },
              ],
          },
          new BetaManagedAgentsMcpToolsetParams
          {
              McpServerName = "github",
              DefaultConfig = new()
              {
                  PermissionPolicy = new BetaManagedAgentsAutoPolicy(),
              },
          },
      ],
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Ops Agent",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	MCPServers: []anthropic.BetaManagedAgentsURLMCPServerParams{{
  		Name: "github",
  		URL:  "https://mcp.example.com/github",
  	}},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{
  		{
  			OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
  				DefaultConfig: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParams{
  					PermissionPolicy: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParamsPermissionPolicyUnion{
  						OfAuto: &anthropic.BetaManagedAgentsAutoPolicyParam{},
  					},
  				},
  				Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{{
  					OfBash: &anthropic.BetaManagedAgentsBashToolConfigParams{
  						PermissionPolicy: anthropic.BetaManagedAgentsBashToolConfigParamsPermissionPolicyUnion{
  							OfAlwaysAsk: &anthropic.BetaManagedAgentsAlwaysAskPolicyParam{},
  						},
  					},
  				}},
  			},
  		},
  		{
  			OfMCPToolset: &anthropic.BetaManagedAgentsMCPToolsetParams{
  				MCPServerName: "github",
  				DefaultConfig: anthropic.BetaManagedAgentsMCPToolsetDefaultConfigParams{
  					PermissionPolicy: anthropic.BetaManagedAgentsMCPToolsetDefaultConfigParamsPermissionPolicyUnion{
  						OfAuto: &anthropic.BetaManagedAgentsAutoPolicyParam{},
  					},
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.agents.*;

  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Ops Agent")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .addMcpServer(
              BetaManagedAgentsUrlMcpServerParams.builder()
                  .name("github")
                  .url("https://mcp.example.com/github")
                  .build()
          )
          .addTool(
              BetaManagedAgentsAgentToolset20260401Params.builder()
                  .defaultConfig(
                      BetaManagedAgentsAgentToolsetDefaultConfigParams.builder()
                          .permissionPolicy(BetaManagedAgentsAutoPolicy.builder().build())
                          .build()
                  )
                  .addConfig(
                      BetaManagedAgentsBashToolConfigParams.builder()
                          .permissionPolicy(BetaManagedAgentsAlwaysAskPolicy.builder().build())
                          .build()
                  )
                  .build()
          )
          .addTool(
              BetaManagedAgentsMcpToolsetParams.builder()
                  .mcpServerName("github")
                  .defaultConfig(
                      BetaManagedAgentsMcpToolsetDefaultConfigParams.builder()
                          .permissionPolicy(BetaManagedAgentsAutoPolicy.builder().build())
                          .build()
                  )
                  .build()
          )
          .build()
  );
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolsetDefaultConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsAlwaysAskPolicy;
  use Anthropic\Beta\Agents\BetaManagedAgentsAutoPolicy;
  use Anthropic\Beta\Agents\BetaManagedAgentsBashToolConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsMCPToolsetDefaultConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsMCPToolsetParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsURLMCPServerParams;

  $agent = $client->beta->agents->create(
      name: 'Ops Agent',
      model: 'claude-opus-5-5',
      mcpServers: [
          BetaManagedAgentsURLMCPServerParams::with(
              name: 'github',
              url: 'https://mcp.example.com/github',
          ),
      ],
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(
              defaultConfig: BetaManagedAgentsAgentToolsetDefaultConfigParams::with(
                  permissionPolicy: BetaManagedAgentsAutoPolicy::with(),
              ),
              configs: [
                  BetaManagedAgentsBashToolConfigParams::with(
                      permissionPolicy: BetaManagedAgentsAlwaysAskPolicy::with(),
                  ),
              ],
          ),
          BetaManagedAgentsMCPToolsetParams::with(
              mcpServerName: 'github',
              defaultConfig: BetaManagedAgentsMCPToolsetDefaultConfigParams::with(
                  permissionPolicy: BetaManagedAgentsAutoPolicy::with(),
              ),
          ),
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Ops Agent",
    model: "claude-opus-5-5",
    mcp_servers: [
      {type: "url", name: "github", url: "https://mcp.example.com/github"}
    ],
    tools: [
      {
        type: "agent_toolset_20260401",
        default_config: {
          permission_policy: {type: "auto"}
        },
        configs: [
          {name: "bash", permission_policy: {type: "always_ask"}}
        ]
      },
      {
        type: "mcp_toolset",
        mcp_server_name: "github",
        default_config: {
          permission_policy: {type: "auto"}
        }
      }
    ]
  )
  ```
</CodeGroup>

您在 `user.message` 事件中发布的内容被视为您的意图，它可能会使服务器允许原本会拒绝的调用。服务器不会从工具结果、获取的网页、MCP 服务器的响应或[会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads#tool-permissions-and-custom-tools)之间的消息中读取意图。它会评估这些内容，但不会从中接受指令。无论由谁发起请求，服务器都会将某些调用评估为高风险。如果您在 `user.message` 事件中转发不受信任的最终用户输入，服务器也会将该输入视为您的意图，并可能因此允许某个调用。对于您不希望该最终用户在未经审查的情况下运行的工具，请配置 `always_ask`。

<Warning>
  `auto` 不是人工检查点。如果服务器确定某个调用是安全的，该调用会在任何人看到之前运行，并且其影响可能无法撤销。如果某个工具的调用必须在运行前由人员审查，请在该工具上配置 `always_ask`。
</Warning>

## 查看每次调用的评估方式

在任何权限策略下，每个 `agent.tool_use` 和 `agent.mcp_tool_use` 事件都带有 `evaluated_permission`，即该调用权限检查的结果：`"allow"`、`"ask"` 或 `"deny"`。大多数事件还带有一个 `evaluation` 对象，其 `type` 指明产生该结果的策略。在 `auto` 下，该对象还会记录服务器的判定，并在结果为 `ask` 或 `deny` 时附带 `reason_code`。

例如，当 `bash` 处于 `auto` 下且服务器将某个调用评估为高风险时，被拒绝的调用在事件流中显示如下：

```json
{
  "type": "agent.tool_use",
  "id": "sevt_01pqr...",
  "name": "bash",
  "input": {
    "command": "rm -rf /workspace/reports"
  },
  "evaluated_permission": "deny",
  "evaluation": {
    "type": "auto",
    "evaluated_permission": {
      "type": "deny",
      "reason_code": "high_risk"
    }
  },
  "processed_at": "2026-03-25T14:05:12Z"
}
```

`evaluation` 对象采用下表中的某种形式。

| `evaluation`                                                                                | 顶层 `evaluated_permission` | 含义                                   |
| ------------------------------------------------------------------------------------------- | ------------------------- | ------------------------------------ |
| `{"type": "always_allow"}`                                                                  | `"allow"`                 | 解析后的策略为 `always_allow`，因此调用已运行。      |
| `{"type": "always_ask"}`                                                                    | `"ask"`                   | 解析后的策略为 `always_ask`，因此调用已暂停以等待您的批准。 |
| `{"type": "auto", "evaluated_permission": {"type": "allow"}}`                               | `"allow"`                 | 在 `auto` 下，服务器确定该调用是安全的，并已运行该调用。     |
| `{"type": "auto", "evaluated_permission": {"type": "ask", "reason_code": "indeterminate"}}` | `"ask"`                   | 在 `auto` 下，服务器未能做出判定，因此调用已暂停以等待您的批准。 |
| `{"type": "auto", "evaluated_permission": {"type": "deny", "reason_code": "high_risk"}}`    | `"deny"`                  | 在 `auto` 下，服务器将该调用评估为高风险并拒绝了它。       |

当 `evaluation.type` 为 `"auto"` 时，其嵌套的 `evaluated_permission.type` 与事件的顶层 `evaluated_permission` 相同，因此您可以从任一字段读取结果。`reason_code` 是供您的客户端进行分支判断并保存在审计记录中的值，而不是用于向最终用户显示的文本。

在两种情况下不存在 `evaluation`。当智能体指定了一个在会话中未启用的工具时，服务器会在不评估策略的情况下拒绝该调用：该事件带有 `evaluated_permission: "deny"`，但没有 `evaluation`。在引入 `evaluation` 之前记录的事件也不包含该字段：当 `evaluated_permission` 为 `"allow"` 时，请将其视为 `always_allow`；当其为 `"ask"` 时，请将其视为 `always_ask`。

请确保您的客户端能够容忍无法识别的 `evaluation.type` 或 `reason_code`。`agent.custom_tool_use` 事件不带这两个字段，因为权限策略不约束[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#custom-tools)。

## 响应确认请求

在 `always_ask` 策略下，或在 `auto` 下服务器未得出判定时，工具调用的评估结果为 `ask`。发生这种情况时：

1. 会话发出 `agent.tool_use` 或 `agent.mcp_tool_use` 事件。
2. 会话以一个 `session.status_idle` 事件暂停，其 `stop_reason.type` 为 `requires_action`。阻塞事件的 ID 位于 `stop_reason.event_ids` 数组中。会话会无限期等待响应。来自[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#while-a-run-is-open)线程的调用会通过主事件流到达您的客户端。只要会话中还有其他部分在工作，会话可能保持 `running` 状态且不发送 `session.status_idle`，因此请直接响应工具使用事件本身。该调用只会等待到运行结束对该线程的处理或运行结束为止。中断可能会使其保持等待状态，因此请自行允许或拒绝它。
3. 为每个阻塞事件发送一个 `user.tool_confirmation` 事件，并在 `tool_use_id` (java: `toolUseId`; php: `toolUseID`; csharp, go: `ToolUseID`) 参数中传入事件 ID。将 `result` (csharp, go: `Result`) 设置为 `"allow"` 或 `"deny"`。使用 `deny_message` (java, php: `denyMessage`; csharp, go: `DenyMessage`) 说明拒绝原因。您可以在单个 `events` 请求中发送多个确认。
4. 所有阻塞事件都处理完毕后，会话将转换回 `running` 状态。被允许的工具会运行。被拒绝的工具不会运行，智能体会收到一个说明调用已被拒绝的工具结果，其中包括您的 `deny_message` (java, php: `denyMessage`; csharp, go: `DenyMessage`)。

例如，在 `always_ask` 策略下暂停的 `bash` 调用在流中显示如下：

```json
{
  "type": "agent.tool_use",
  "id": "sevt_01def...",
  "name": "bash",
  "input": {
    "command": "pip install -r requirements.txt"
  },
  "evaluated_permission": "ask",
  "evaluation": {
    "type": "always_ask"
  },
  "processed_at": "2026-03-25T14:01:45Z"
}
```

如果您为 `evaluated_permission` (java, php: `evaluatedPermission`; csharp, go: `EvaluatedPermission`) 不是 `ask` 的事件发送 `user.tool_confirmation`，API 会以 400 错误拒绝它。这包括服务器在 `auto` 下拒绝的调用：您的客户端无法推翻这些拒绝。

如需以交互方式进行响应，请使用 `ant beta:sessions connect`，它会显示正在等待的调用，并在您允许或拒绝时发送此事件。请参阅[从终端连接到 Managed Agents 会话](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/sessions-connect#follow-and-steer-the-session)。

在以下示例中，工具使用事件 ID 来自 `session.status_idle` 事件的 `stop_reason.event_ids` 数组。在会话事件流指南中了解有关[接收事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#stream-events)的更多信息，或[订阅 webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks)，以便在会话暂停等待输入时收到通知。

<CodeGroup>
  ```bash cURL
  # 允许该工具运行
  curl -fsSL "https://api.anthropic.com/v1/sessions/$SESSION_ID/events" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {
          "type": "user.tool_confirmation",
          "tool_use_id": "'$AGENT_TOOL_USE_EVENT_ID'",
          "result": "allow"
        }
      ]
    }'

  # 或拒绝执行并附上说明
  curl -fsSL "https://api.anthropic.com/v1/sessions/$SESSION_ID/events" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {
          "type": "user.tool_confirmation",
          "tool_use_id": "'$MCP_TOOL_USE_EVENT_ID'",
          "result": "deny",
          "deny_message": "Don'\''t create issues in the production project. Use the staging project."
        }
      ]
    }'
  ```

  ```bash CLI
  # 允许该工具运行
  ant beta:sessions:events send \
    --session-id "$SESSION_ID" \
    --event "{type: user.tool_confirmation, tool_use_id: $AGENT_TOOL_USE_EVENT_ID, result: allow}"

  # 或拒绝执行并给出说明
  ant beta:sessions:events send \
    --session-id "$SESSION_ID" \
    --event "{type: user.tool_confirmation, tool_use_id: $MCP_TOOL_USE_EVENT_ID, result: deny,
      deny_message: Don't create issues in the production project. Use the staging project.}"
  ```

  ```python Python
  # 允许该工具运行
  client.beta.sessions.events.send(
      session.id,
      events=[
          {
              "type": "user.tool_confirmation",
              "tool_use_id": agent_tool_use_event.id,
              "result": "allow",
          },
      ],
  )

  # 或拒绝执行并给出说明
  client.beta.sessions.events.send(
      session.id,
      events=[
          {
              "type": "user.tool_confirmation",
              "tool_use_id": mcp_tool_use_event.id,
              "result": "deny",
              "deny_message": "Don't create issues in the production project. Use the staging project.",
          },
      ],
  )
  ```

  ```typescript TypeScript
  // 允许该工具运行
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.tool_confirmation",
        tool_use_id: agent_tool_use_event.id,
        result: "allow"
      }
    ]
  });

  // 或拒绝执行并给出说明
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.tool_confirmation",
        tool_use_id: mcp_tool_use_event.id,
        result: "deny",
        deny_message: "Don't create issues in the production project. Use the staging project."
      }
    ]
  });
  ```

  ```csharp C#
  // 允许该工具运行
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserToolConfirmationEventParams
          {
              ToolUseID = agentToolUseEvent.ID,
              Result = "allow",
          },
      ],
  });

  // 或拒绝执行并给出说明
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserToolConfirmationEventParams
          {
              ToolUseID = mcpToolUseEvent.ID,
              Result = "deny",
              DenyMessage = "Don't create issues in the production project. Use the staging project.",
          },
      ],
  });
  ```

  ```go Go
  // 允许该工具运行
  _, err = client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfUserToolConfirmation: &anthropic.BetaManagedAgentsUserToolConfirmationEventParams{
  			ToolUseID: agentToolUseEvent.ID,
  			Result:    anthropic.BetaManagedAgentsUserToolConfirmationEventParamsResultAllow,
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }

  // 或拒绝执行并给出说明
  _, err = client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfUserToolConfirmation: &anthropic.BetaManagedAgentsUserToolConfirmationEventParams{
  			ToolUseID:   mcpToolUseEvent.ID,
  			Result:      anthropic.BetaManagedAgentsUserToolConfirmationEventParamsResultDeny,
  			DenyMessage: anthropic.String("Don't create issues in the production project. Use the staging project."),
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  // 允许该工具运行
  client.beta().sessions().events().send(
      session.id(),
      EventSendParams.builder()
          .addEvent(
              BetaManagedAgentsUserToolConfirmationEventParams.builder()
                  .toolUseId(agentToolUseEvent.id())
                  .result(BetaManagedAgentsUserToolConfirmationEventParams.Result.ALLOW)
                  .build()
          )
          .build()
  );

  // 或拒绝执行并给出说明
  client.beta().sessions().events().send(
      session.id(),
      EventSendParams.builder()
          .addEvent(
              BetaManagedAgentsUserToolConfirmationEventParams.builder()
                  .toolUseId(mcpToolUseEvent.id())
                  .result(BetaManagedAgentsUserToolConfirmationEventParams.Result.DENY)
                  .denyMessage("Don't create issues in the production project. Use the staging project.")
                  .build()
          )
          .build()
  );
  ```

  ```php PHP
  use Anthropic\Beta\Sessions\Events\ManagedAgentsUserToolConfirmationEventParams;

  // 允许该工具运行
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          ManagedAgentsUserToolConfirmationEventParams::with(
              toolUseID: $agentToolUseEvent->id,
              result: 'allow',
          ),
      ],
  );

  // 或拒绝执行并给出说明
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          ManagedAgentsUserToolConfirmationEventParams::with(
              toolUseID: $mcpToolUseEvent->id,
              result: 'deny',
              denyMessage: "Don't create issues in the production project. Use the staging project.",
          ),
      ],
  );
  ```

  ```ruby Ruby
  # 允许该工具运行
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {
        type: "user.tool_confirmation",
        tool_use_id: agent_tool_use_event.id,
        result: "allow"
      }
    ]
  )

  # 或拒绝执行并给出说明
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {
        type: "user.tool_confirmation",
        tool_use_id: mcp_tool_use_event.id,
        result: "deny",
        deny_message: "Don't create issues in the production project. Use the staging project."
      }
    ]
  )
  ```
</CodeGroup>

## 自定义工具

权限策略不适用于自定义工具。当智能体调用自定义工具时，您的应用程序会收到一个 `agent.custom_tool_use` 事件，并负责决定是否运行该工具，然后再发回 `user.custom_tool_result`。有关完整流程，请参阅[响应使会话暂停的工具调用](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#answer-tool-calls-that-pause-the-session)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="技能" icon="books" href="https://platform.claude.com/docs/zh-CN/managed-agents/skills">
    为您的智能体附加可复用的、基于文件系统的专业知识，以支持特定领域的工作流程。
  </Card>

  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    发送事件、流式传输响应，并在会话运行时中断或重定向会话。
  </Card>
</CardGroup>

## 可用性

权限策略在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启

| 状态   | 云平台     |
| ---- | ------- |
| Beta | 所有支持的平台 |

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
