---
title: MCP 连接器
url: https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector
description: 将 MCP 服务器连接到您的智能体，以访问外部工具和数据源。
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

Claude Managed Agents 支持将 [Model Context Protocol (MCP)](https://modelcontextprotocol.io) 服务器连接到您的智能体。这使智能体能够通过标准化协议访问外部工具、数据源和服务。

MCP 配置分为两个步骤：

1. **智能体创建**通过名称和 URL 声明智能体要连接的 MCP 服务器。
2. **会话创建**通过引用预先注册的 vault（保管库）为这些服务器提供身份验证（请参阅[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)）。

这种分离使密钥不会出现在可复用的智能体定义中，同时允许每个会话使用自己的凭据进行身份验证。

## 在智能体上声明 MCP 服务器

创建智能体时，在 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 数组中指定 MCP 服务器。每个服务器都有一个值为 `url` 的 `type` (csharp, go: `Type`)、一个唯一的 `name` (csharp, go: `Name`) 以及一个 `url` (csharp: `Url`; go: `URL`)。此阶段不提供任何身份验证令牌。

每个声明的服务器还需要在 `tools` (csharp, go: `Tools`) 数组中有一个对应的 `mcp_toolset` 条目。该工具集的 `mcp_server_name` (java, php: `mcpServerName`; csharp: `McpServerName`; go: `MCPServerName`) 必须与服务器的 `name` (csharp, go: `Name`) 相匹配。

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -sS --fail-with-body https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "GitHub Assistant",
      "model": "claude-opus-5-5",
      "mcp_servers": [
        {
          "type": "url",
          "name": "github",
          "url": "https://api.githubcopilot.com/mcp/"
        }
      ],
      "tools": [
        {"type": "agent_toolset_20260401"},
        {"type": "mcp_toolset", "mcp_server_name": "github"}
      ]
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply github-assistant.md
    ```

    <File filename="github-assistant.md">
      ```markdown
      ---
      name: GitHub Assistant
      model: claude-opus-5-5
      mcp_servers:
        - type: url
          name: github
          url: https://api.githubcopilot.com/mcp/
      tools:
        - type: agent_toolset_20260401
        - type: mcp_toolset
          mcp_server_name: github
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="GitHub Assistant",
      model="claude-opus-5-5",
      mcp_servers=[
          {
              "type": "url",
              "name": "github",
              "url": "https://api.githubcopilot.com/mcp/",
          },
      ],
      tools=[
          {"type": "agent_toolset_20260401"},
          {"type": "mcp_toolset", "mcp_server_name": "github"},
      ],
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "GitHub Assistant",
    model: "claude-opus-5-5",
    mcp_servers: [
      {
        type: "url",
        name: "github",
        url: "https://api.githubcopilot.com/mcp/"
      }
    ],
    tools: [
      { type: "agent_toolset_20260401" },
      { type: "mcp_toolset", mcp_server_name: "github" }
    ]
  });
  ```

  ```csharp C#
  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "GitHub Assistant",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      McpServers =
      [
          new() { Name = "github", Url = "https://api.githubcopilot.com/mcp/" },
      ],
      Tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params(),
          new BetaManagedAgentsMcpToolsetParams { McpServerName = "github" },
      ],
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "GitHub Assistant",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	MCPServers: []anthropic.BetaManagedAgentsURLMCPServerParams{{
  		Name: "github",
  		URL:  "https://api.githubcopilot.com/mcp/",
  	}},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{
  		{
  			OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{},
  		},
  		{
  			OfMCPToolset: &anthropic.BetaManagedAgentsMCPToolsetParams{
  				MCPServerName: "github",
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("GitHub Assistant")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .addMcpServer(
              BetaManagedAgentsUrlMcpServerParams.builder()
                  .name("github")
                  .url("https://api.githubcopilot.com/mcp/")
                  .build()
          )
          .addTool(BetaManagedAgentsAgentToolset20260401Params.builder().build())
          .addTool(
              BetaManagedAgentsMcpToolsetParams.builder()
                  .mcpServerName("github")
                  .build()
          )
          .build()
  );
  ```

  ```php PHP
  $agent = $client->beta->agents->create(
      name: 'GitHub Assistant',
      model: BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
      mcpServers: [
          BetaManagedAgentsURLMCPServerParams::with(
              name: 'github',
              url: 'https://api.githubcopilot.com/mcp/',
          ),
      ],
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(),
          BetaManagedAgentsMCPToolsetParams::with(
              mcpServerName: 'github',
          ),
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "GitHub Assistant",
    model: "claude-opus-5-5",
    mcp_servers: [
      {
        type: "url",
        name: "github",
        url: "https://api.githubcopilot.com/mcp/"
      }
    ],
    tools: [
      {type: "agent_toolset_20260401"},
      {type: "mcp_toolset", mcp_server_name: "github"}
    ]
  )
  ```
</CodeGroup>

<Tip>
  MCP 工具集的默认权限策略为 `always_ask`，即每次工具调用前都需要用户批准。请参阅[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)以配置此行为。
</Tip>

### `mcp_servers` 字段参考

`mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 数组中的每个条目定义一个连接。

| 字段                               | 描述                                                                                                                                                                                                                                                                   |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type` (csharp, go: `Type`)      | 必填。必须为 `"url"`。                                                                                                                                                                                                                                                      |
| `name` (csharp, go: `Name`)      | 必填。该服务器在智能体内的唯一名称（1–255 个字符）。用作 `tools` (csharp, go: `Tools`) 数组中的 `mcp_server_name` (java, php: `mcpServerName`; csharp: `McpServerName`; go: `MCPServerName`)，并显示在[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)中的 MCP 工具事件上。 |
| `url` (csharp: `Url`; go: `URL`) | 必填。远程 MCP 服务器的端点（最多 2,048 个字符）。有关传输要求，请参阅[支持的 MCP 服务器类型](https://platform.claude.com/docs/zh-CN/managed-agents/reference#supported-mcp-server-types)。                                                                                                                |

约束：

* 一个智能体最多可以声明 20 个 MCP 服务器。服务器名称在数组内必须唯一。
* 每个 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 条目都必须被 `tools` (csharp, go: `Tools`) 数组中的某个 `mcp_toolset` 引用，并且每个 `mcp_toolset` 都必须引用一个已声明的服务器。API 会拒绝包含未被引用的服务器或悬空工具集的智能体定义。

## 配置可用的 MCP 工具

`mcp_toolset` 条目支持一个 `default_config` (java, php: `defaultConfig`; csharp, go: `DefaultConfig`) 对象和一个 `configs` (csharp, go: `Configs`) 数组，它们应用于 MCP 服务器公开的工具。每个 `configs` (csharp, go: `Configs`) 条目仅接受 `name` (csharp, go: `Name`)、`enabled` (csharp, go: `Enabled`) 和 `permission_policy` (java, php: `permissionPolicy`; csharp, go: `PermissionPolicy`)。与内置智能体工具集中的条目不同，MCP 工具条目不接受 `type` (csharp, go: `Type`) 字段，并且 `web_search` 和 `web_fetch` 上可用的 [Web 设置](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#settings)不适用于 MCP 工具。每个 `configs` (csharp, go: `Configs`) 条目中的 `name` (csharp, go: `Name`) 是服务器报告的原始工具名称。

默认情况下，MCP 服务器公开的所有工具均已启用。若要仅启用特定工具，请将 `default_config.enabled` 设置为 `false`，并显式启用您需要的工具：

```json
{
  "type": "mcp_toolset",
  "mcp_server_name": "github",
  "default_config": { "enabled": false },
  "configs": [
    { "name": "get_issue", "enabled": true },
    { "name": "list_issues", "enabled": true },
    { "name": "add_issue_comment", "enabled": true }
  ]
}
```

当服务器公开了许多工具但智能体只需要其中几个时，或者当您希望服务器运营方新增的工具在您审核之前保持关闭状态时，此模式非常有用。

若要禁用特定工具而保持其余工具启用，请省略 `default_config`，并在各个条目上设置 `enabled: false`：

```json
{
  "type": "mcp_toolset",
  "mcp_server_name": "github",
  "configs": [{ "name": "delete_repository", "enabled": false }]
}
```

有关通用的 `default_config` (java, php: `defaultConfig`; csharp, go: `DefaultConfig`) / `configs` (csharp, go: `Configs`) 模式，请参阅[配置工具集](https://platform.claude.com/docs/zh-CN/managed-agents/tools#configuring-the-toolset)；有关在 MCP 工具上设置 `permission_policy` (java, php: `permissionPolicy`; csharp, go: `PermissionPolicy`) 以及处理确认请求，请参阅 [MCP 工具集权限](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#mcp-toolset-permissions)。

### MCP 工具输出处理

当 MCP 工具输出超过 100,000 个字符（约 25,000 个令牌）时，它会被自动写入沙箱中的文件。模型会收到一个包含文件路径的截断预览，并可以从该文件读取完整内容。

## 在会话创建时提供身份验证

启动会话时，传入 `vault_ids` (java: `vaultIds`; php: `vaultIDs`; csharp: `VaultIds`; go: `VaultIDs`) 以为您的 MCP 服务器提供凭据。保管库是凭据的集合，您只需注册一次，之后即可通过 ID 引用。有关如何创建保管库和管理凭据，请参阅[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)。

使用 `limited` [环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)时，如果智能体声明的 MCP 服务器的主机不在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中，会话创建将失败并返回 400 错误。在环境的网络设置中将 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 设置为 `true` 可关闭此检查。

<CodeGroup>
  ```bash cURL
  curl -sS --fail-with-body https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$agent_id"'",
      "environment_id": "'"$environment_id"'",
      "vault_ids": ["'"$vault_id"'"]
    }'
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent "$AGENT_ID" \
    --environment-id "$ENVIRONMENT_ID" \
    --vault-id "$VAULT_ID"
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
      vault_ids=[vault.id],
  )
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id,
    vault_ids: [vault.id]
  });
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
      VaultIds = [vault.ID],
  });
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent:         anthropic.BetaSessionNewParamsAgentUnion{OfString: anthropic.String(agent.ID)},
  	EnvironmentID: environment.ID,
  	VaultIDs:      []string{vault.ID},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var session = client.beta().sessions().create(
      SessionCreateParams.builder()
          .agent(agent.id())
          .environmentId(environment.id())
          .addVaultId(vault.id())
          .build()
  );
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
      vaultIDs: [$vault->id],
  );
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id,
    vault_ids: [vault.id]
  )
  ```
</CodeGroup>

凭据按 URL 进行匹配，因此保管库中必须包含一个凭据，其 `mcp_server_url` (java: `mcpServerUrl`; php: `mcpServerURL`; csharp: `McpServerUrl`; go: `MCPServerURL`) 与 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 中声明的 `url` (csharp: `Url`; go: `URL`) 指向同一服务器。两个 URL 在匹配前都会被规范化（协议和主机名转为小写，去除默认端口和末尾斜杠），因此主机名大小写、默认端口或末尾斜杠的差异不会妨碍匹配；但不同的路径、子域名或非默认端口则会导致不匹配。如果没有匹配的凭据，将尝试以未经身份验证的方式建立连接。有关 `static_bearer` 和 `mcp_oauth` 凭据类型，请参阅[添加凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults#add-a-credential)。

### 处理连接和身份验证失败

会话创建不会验证 MCP 连接性或凭据。但它会根据环境的网络设置检查每个已声明服务器的主机：使用 `limited` 环境时，如果某个主机不被允许，会话创建将失败并返回 400 错误，如[在会话创建时提供身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector#provide-authentication-at-session-creation)中所述。如果 MCP 服务器无法访问或拒绝所提供的凭据，会话仍会启动，并且仍可进行交互。系统会发出一个 [`session.error`](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types) 事件，其中包含受影响服务器的 `mcp_server_name` (java, php: `mcpServerName`; csharp: `McpServerName`; go: `MCPServerName`) 以及一个 `retry_status` (java, php: `retryStatus`; csharp, go: `RetryStatus`)：

| 错误类型                              | 含义                                                                    |
| --------------------------------- | --------------------------------------------------------------------- |
| `mcp_connection_failed_error`     | 无法访问 MCP 服务器（网络错误、超时或非身份验证类的 HTTP 失败）。                                |
| `mcp_authentication_failed_error` | 与 MCP 服务器的身份验证失败：服务器拒绝了来自所附保管库的凭据，或服务器要求身份验证但未配置匹配的凭据，或 OAuth 令牌刷新失败。 |

您可以决定是否因此错误阻止进一步交互、触发凭据轮换，或让会话在没有受影响服务器工具的情况下继续运行。连接会在下一次从 `session.status_idle` 到 `session.status_running` 的状态转换时重试。

## 后续步骤

<CardGroup cols={2}>
  <Card title="权限策略" icon="check" href="https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies">
    控制智能体工具和 MCP 工具何时运行。
  </Card>

  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    发送事件、流式传输响应，并在会话运行时中断或重定向会话。
  </Card>

  <Card title="支持的 MCP 服务器类型" icon="book" href="https://platform.claude.com/docs/zh-CN/managed-agents/reference#supported-mcp-server-types">
    远程 MCP 服务器的传输要求。
  </Card>
</CardGroup>
