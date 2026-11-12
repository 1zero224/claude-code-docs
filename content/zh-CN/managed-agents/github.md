---
title: 访问 GitHub
url: https://platform.claude.com/docs/zh-CN/managed-agents/github
description: 将您的智能体连接到 GitHub 仓库，以进行克隆、读取和创建拉取请求。
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

您可以将 GitHub 仓库挂载到会话沙箱中，并连接到 GitHub MCP 以创建拉取请求（pull request）。

GitHub 仓库会被缓存，因此后续使用同一仓库的会话启动速度更快。

## GitHub MCP 与会话资源

首先，创建一个声明 GitHub MCP 服务器的智能体。智能体定义中包含服务器 URL，但不包含身份验证令牌：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Code Reviewer",
      "model": "claude-opus-5-5",
      "system": "You are a code review assistant with access to GitHub.",
      "mcp_servers": [
        {
          "type": "url",
          "name": "github",
          "url": "https://api.githubcopilot.com/mcp/"
        }
      ],
      "tools": [
        {
          "type": "agent_toolset_20260401",
          "configs": [
            {"name": "web_search", "enabled": false},
            {"name": "web_fetch", "enabled": false}
          ]
        },
        {
          "type": "mcp_toolset",
          "mcp_server_name": "github"
        }
      ]
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply code-reviewer.md
    ```

    <File filename="code-reviewer.md">
      ```markdown
      ---
      name: Code Reviewer
      model: claude-opus-5-5
      mcp_servers:
        - type: url
          name: github
          url: https://api.githubcopilot.com/mcp/
      tools:
        - type: agent_toolset_20260401
          configs:
            - name: web_search
              enabled: false
            - name: web_fetch
              enabled: false
        - type: mcp_toolset
          mcp_server_name: github
      ---

      You are a code review assistant with access to GitHub.
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Code Reviewer",
      model="claude-opus-5-5",
      system="You are a code review assistant with access to GitHub.",
      mcp_servers=[
          {
              "type": "url",
              "name": "github",
              "url": "https://api.githubcopilot.com/mcp/",
          },
      ],
      tools=[
          {
              "type": "agent_toolset_20260401",
              "configs": [
                  {"name": "web_search", "enabled": False},
                  {"name": "web_fetch", "enabled": False},
              ],
          },
          {
              "type": "mcp_toolset",
              "mcp_server_name": "github",
          },
      ],
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Code Reviewer",
    model: "claude-opus-5-5",
    system: "You are a code review assistant with access to GitHub.",
    mcp_servers: [
      {
        type: "url",
        name: "github",
        url: "https://api.githubcopilot.com/mcp/"
      }
    ],
    tools: [
      {
        type: "agent_toolset_20260401",
        configs: [
          { name: "web_search", enabled: false },
          { name: "web_fetch", enabled: false }
        ]
      },
      {
        type: "mcp_toolset",
        mcp_server_name: "github"
      }
    ]
  });
  ```

  ```csharp C#
  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Code Reviewer",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      System = "You are a code review assistant with access to GitHub.",
      McpServers =
      [
          new() { Name = "github", Url = "https://api.githubcopilot.com/mcp/" },
      ],
      Tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params
          {
              Configs =
              [
                  new BetaManagedAgentsWebSearchToolConfigParams { Enabled = false },
                  new BetaManagedAgentsWebFetchToolConfigParams { Enabled = false },
              ],
          },
          new BetaManagedAgentsMcpToolsetParams { McpServerName = "github" },
      ],
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Code Reviewer",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	System: anthropic.String("You are a code review assistant with access to GitHub."),
  	MCPServers: []anthropic.BetaManagedAgentsURLMCPServerParams{
  		{
  			Name: "github",
  			URL:  "https://api.githubcopilot.com/mcp/",
  		},
  	},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{
  		{
  			OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
  				Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{
  					{OfWebSearch: &anthropic.BetaManagedAgentsWebSearchToolConfigParams{
  						Enabled: anthropic.Bool(false),
  					}},
  					{OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{
  						Enabled: anthropic.Bool(false),
  					}},
  				},
  			},
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
  var agent = client.beta().agents().create(AgentCreateParams.builder()
      .name("Code Reviewer")
      .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
      .system("You are a code review assistant with access to GitHub.")
      .addMcpServer(BetaManagedAgentsUrlMcpServerParams.builder()
          .name("github")
          .url("https://api.githubcopilot.com/mcp/")
          .build())
      .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
          .addConfig(BetaManagedAgentsWebSearchToolConfigParams.builder()
              .enabled(false)
              .build())
          .addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()
              .enabled(false)
              .build())
          .build())
      .addTool(BetaManagedAgentsMcpToolsetParams.builder()
          .mcpServerName("github")
          .build())
      .build());
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsModel;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebFetchToolConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebSearchToolConfigParams;

  $agent = $client->beta->agents->create(
      name: 'Code Reviewer',
      model: BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
      system: 'You are a code review assistant with access to GitHub.',
      mcpServers: [
          [
              'type' => 'url',
              'name' => 'github',
              'url' => 'https://api.githubcopilot.com/mcp/',
          ],
      ],
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(
              configs: [
                  BetaManagedAgentsWebSearchToolConfigParams::with(enabled: false),
                  BetaManagedAgentsWebFetchToolConfigParams::with(enabled: false),
              ],
          ),
          [
              'type' => 'mcp_toolset',
              'mcpServerName' => 'github',
          ],
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Code Reviewer",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
    system_: "You are a code review assistant with access to GitHub.",
    mcp_servers: [
      {
        type: "url",
        name: "github",
        url: "https://api.githubcopilot.com/mcp/"
      }
    ],
    tools: [
      {
        type: "agent_toolset_20260401",
        configs: [
          {name: "web_search", enabled: false},
          {name: "web_fetch", enabled: false}
        ]
      },
      {
        type: "mcp_toolset",
        mcp_server_name: "github"
      }
    ]
  )
  ```
</CodeGroup>

然后创建一个挂载该 GitHub 仓库的会话。在 `limited` [环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)中，如果智能体声明的 MCP 服务器的主机不在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中，会话创建将失败并返回 400 错误。在环境的网络设置中将 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 设置为 `true` 可关闭此检查。

<CodeGroup>
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d @- <<JSON
  {
    "agent": "$agent_id",
    "environment_id": "$environment_id",
    "resources": [
      {
        "type": "github_repository",
        "url": "https://github.com/org/repo",
        "mount_path": "/workspace/repo",
        "authorization_token": "ghp_your_github_token"
      }
    ]
  }
  JSON
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent "$AGENT_ID" \
    --environment-id "$ENVIRONMENT_ID" <<'EOF'
  resources:
    - type: github_repository
      url: https://github.com/org/repo
      mount_path: /workspace/repo
      authorization_token: ghp_your_github_token
  EOF
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
      resources=[
          {
              "type": "github_repository",
              "url": "https://github.com/org/repo",
              "mount_path": "/workspace/repo",
              "authorization_token": "ghp_your_github_token",
          },
      ],
  )
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id,
    resources: [
      {
        type: "github_repository",
        url: "https://github.com/org/repo",
        mount_path: "/workspace/repo",
        authorization_token: "ghp_your_github_token"
      }
    ]
  });
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
      Resources =
      [
          new BetaManagedAgentsGitHubRepositoryResourceParams
          {
              Url = "https://github.com/org/repo",
              MountPath = "/workspace/repo",
              AuthorizationToken = "ghp_your_github_token",
          },
      ],
  });
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent:         anthropic.BetaSessionNewParamsAgentUnion{OfString: anthropic.String(agent.ID)},
  	EnvironmentID: environment.ID,
  	Resources: []anthropic.BetaSessionNewParamsResourceUnion{
  		{
  			OfGitHubRepository: &anthropic.BetaManagedAgentsGitHubRepositoryResourceParams{
  				URL:                "https://github.com/org/repo",
  				MountPath:          anthropic.String("/workspace/repo"),
  				AuthorizationToken: "ghp_your_github_token",
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(agent.id())
      .environmentId(environment.id())
      .addResource(BetaManagedAgentsGitHubRepositoryResourceParams.builder()
          .url("https://github.com/org/repo")
          .mountPath("/workspace/repo")
          .authorizationToken("ghp_your_github_token")
          .build())
      .build());
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
      resources: [
          [
              'type' => 'github_repository',
              'url' => 'https://github.com/org/repo',
              'mountPath' => '/workspace/repo',
              'authorizationToken' => 'ghp_your_github_token',
          ],
      ],
  );
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id,
    resources: [
      {
        type: "github_repository",
        url: "https://github.com/org/repo",
        mount_path: "/workspace/repo",
        authorization_token: "ghp_your_github_token"
      }
    ]
  )
  ```
</CodeGroup>

`github_repository` 资源接受以下字段：

| 字段                                                                                        | 描述                                                                                                                                                                                                       |
| ----------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type` (csharp, go: `Type`)                                                               | 必填。必须为 `"github_repository"`。                                                                                                                                                                            |
| `url` (csharp: `Url`; go: `URL`)                                                          | 必填。仓库的 HTTPS URL，格式为 `https://github.com/<owner>/<repo>`，不带 `.git` 后缀。其他格式（包括 SSH URL）将被拒绝并返回 `invalid_request_error`。                                                                                   |
| `authorization_token` (java, php: `authorizationToken`; csharp, go: `AuthorizationToken`) | 可选。用于克隆仓库和向其推送的 GitHub 令牌。如果没有令牌，私有仓库将在会话创建后克隆失败。公共仓库无需令牌即可克隆，智能体也可以在其克隆中提交，但 `git push` 会失败。该令牌不会在 API 响应中回显。请参阅[令牌权限](https://platform.claude.com/docs/zh-CN/managed-agents/github#token-permissions)。 |
| `mount_path` (java, php: `mountPath`; csharp, go: `MountPath`)                            | 可选。`/workspace` 下用于克隆仓库的目录。默认为 `/workspace/<repo-name>`。                                                                                                                                                 |
| `checkout` (csharp, go: `Checkout`)                                                       | 可选。要检出的分支（`{"type": "branch", "name": "main"}`）或提交（`{"type": "commit", "sha": "..."}`）。默认为仓库的默认分支。                                                                                                       |

挂载仓库还会加载存储在其根目录 `.claude/skills` 目录中的所有技能。每个会话只发现一次技能，基于会话开始时检出的仓库状态。请参阅[从 GitHub 仓库加载技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills#load-skills-from-a-github-repository)。

## 令牌权限

提供 GitHub 令牌时，请使用所需的最小权限：

| 操作       | 所需范围                      |
| -------- | ------------------------- |
| 克隆私有仓库   | `repo`                    |
| 创建 PR    | `repo`                    |
| 读取 issue | `repo`（私有）或 `public_repo` |
| 创建 issue | `repo`（私有）或 `public_repo` |

<Warning>
  请使用具有所需最小权限的细粒度个人访问令牌（fine-grained personal access token）。避免使用对您的 GitHub 账户拥有广泛访问权限的令牌。
</Warning>

## 多个仓库

通过向 `resources` (csharp, go: `Resources`) 数组添加条目来挂载多个仓库：

<CodeGroup>
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d @- <<JSON
  {
    "agent": "$agent_id",
    "environment_id": "$environment_id",
    "resources": [
      {
        "type": "github_repository",
        "url": "https://github.com/org/frontend",
        "mount_path": "/workspace/frontend",
        "authorization_token": "ghp_your_github_token"
      },
      {
        "type": "github_repository",
        "url": "https://github.com/org/backend",
        "mount_path": "/workspace/backend",
        "authorization_token": "ghp_your_github_token"
      }
    ]
  }
  JSON
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent "$AGENT_ID" \
    --environment-id "$ENVIRONMENT_ID" <<'EOF'
  resources:
    - type: github_repository
      url: https://github.com/org/frontend
      mount_path: /workspace/frontend
      authorization_token: ghp_your_github_token
    - type: github_repository
      url: https://github.com/org/backend
      mount_path: /workspace/backend
      authorization_token: ghp_your_github_token
  EOF
  ```

  ```python Python
  resources = [
      {
          "type": "github_repository",
          "url": "https://github.com/org/frontend",
          "mount_path": "/workspace/frontend",
          "authorization_token": "ghp_your_github_token",
      },
      {
          "type": "github_repository",
          "url": "https://github.com/org/backend",
          "mount_path": "/workspace/backend",
          "authorization_token": "ghp_your_github_token",
      },
  ]
  ```

  ```typescript TypeScript
  const resources = [
    {
      type: "github_repository",
      url: "https://github.com/org/frontend",
      mount_path: "/workspace/frontend",
      authorization_token: "ghp_your_github_token"
    },
    {
      type: "github_repository",
      url: "https://github.com/org/backend",
      mount_path: "/workspace/backend",
      authorization_token: "ghp_your_github_token"
    }
  ];
  ```

  ```csharp C#
  BetaManagedAgentsGitHubRepositoryResourceParams[] resources =
  [
      new()
      {
          Url = "https://github.com/org/frontend",
          MountPath = "/workspace/frontend",
          AuthorizationToken = "ghp_your_github_token",
      },
      new()
      {
          Url = "https://github.com/org/backend",
          MountPath = "/workspace/backend",
          AuthorizationToken = "ghp_your_github_token",
      },
  ];
  ```

  ```go Go
  resources := []anthropic.BetaSessionNewParamsResourceUnion{
  	{
  		OfGitHubRepository: &anthropic.BetaManagedAgentsGitHubRepositoryResourceParams{
  			URL:                "https://github.com/org/frontend",
  			MountPath:          anthropic.String("/workspace/frontend"),
  			AuthorizationToken: "ghp_your_github_token",
  		},
  	},
  	{
  		OfGitHubRepository: &anthropic.BetaManagedAgentsGitHubRepositoryResourceParams{
  			URL:                "https://github.com/org/backend",
  			MountPath:          anthropic.String("/workspace/backend"),
  			AuthorizationToken: "ghp_your_github_token",
  		},
  	},
  }
  ```

  ```java Java
  var resources = List.of(
      BetaManagedAgentsGitHubRepositoryResourceParams.builder()
          .url("https://github.com/org/frontend")
          .mountPath("/workspace/frontend")
          .authorizationToken("ghp_your_github_token")
          .build(),
      BetaManagedAgentsGitHubRepositoryResourceParams.builder()
          .url("https://github.com/org/backend")
          .mountPath("/workspace/backend")
          .authorizationToken("ghp_your_github_token")
          .build());
  ```

  ```php PHP
  $resources = [
      [
          'type' => 'github_repository',
          'url' => 'https://github.com/org/frontend',
          'mountPath' => '/workspace/frontend',
          'authorizationToken' => 'ghp_your_github_token',
      ],
      [
          'type' => 'github_repository',
          'url' => 'https://github.com/org/backend',
          'mountPath' => '/workspace/backend',
          'authorizationToken' => 'ghp_your_github_token',
      ],
  ];
  ```

  ```ruby Ruby
  resources = [
    {
      type: "github_repository",
      url: "https://github.com/org/frontend",
      mount_path: "/workspace/frontend",
      authorization_token: "ghp_your_github_token"
    },
    {
      type: "github_repository",
      url: "https://github.com/org/backend",
      mount_path: "/workspace/backend",
      authorization_token: "ghp_your_github_token"
    }
  ]
  ```
</CodeGroup>

## 在运行中的会话上管理仓库

会话创建后，您可以列出其仓库资源并轮换其授权令牌。每个资源都有一个在会话创建时返回的 `id` (csharp, go: `ID`)（也可通过 `GET /v1/sessions/{session_id}/resources` (python, typescript, ruby: `client.beta.sessions.resources.list()`; go, csharp: `client.Beta.Sessions.Resources.List()`; java: `client.beta().sessions().resources().list()`; php: `$client->beta->sessions->resources->list()`; cli: `ant beta:sessions:resources list`) 获取），您可以使用它进行更新。仓库在会话的整个生命周期内保持挂载；如需更改挂载的仓库，请创建新会话。

<CodeGroup>
  ```bash cURL
  # 列出会话上的资源
  curl -fsS "https://api.anthropic.com/v1/sessions/$session_id/resources" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json"

  # 轮换授权令牌
  curl -fsS \
    "https://api.anthropic.com/v1/sessions/$session_id/resources/$repo_resource_id" \
  # ...
    -d @- <<'JSON'
  {
    "authorization_token": "ghp_your_new_github_token"
  }
  JSON
  ```

  ```bash CLI
  # 列出会话上的资源
  ant beta:sessions:resources list --session-id "$SESSION_ID"

  # 轮换特定资源上的授权令牌
  ant beta:sessions:resources update \
    --session-id "$SESSION_ID" \
    --resource-id "$RESOURCE_ID" \
    --authorization-token "ghp_your_new_github_token"
  ```

  ```python Python
  # 列出会话上的资源
  listed = client.beta.sessions.resources.list(session.id)
  repo_resource_id = listed.data[0].id
  print(repo_resource_id)  # "sesrsc_01ABC..."

  # 轮换授权令牌
  client.beta.sessions.resources.update(
      repo_resource_id,
      session_id=session.id,
      authorization_token="ghp_your_new_github_token",
  )
  ```

  ```typescript TypeScript
  // 列出会话上的资源
  const listed = await client.beta.sessions.resources.list(session.id);
  const repoResource = listed.data.find((entry) => entry.type === "github_repository");
  if (!repoResource) {
    throw new Error("No GitHub repository resource on the session");
  }
  const repoResourceId = repoResource.id;
  console.log(repoResourceId); // "sesrsc_01ABC..."

  // 轮换授权令牌
  await client.beta.sessions.resources.update(repoResourceId, {
    session_id: session.id,
    authorization_token: "ghp_your_new_github_token"
  });
  ```

  ```csharp C#
  // 列出会话上的资源
  var listed = await client.Beta.Sessions.Resources.List(session.ID);
  var repoResourceId = (await listed.Paginate().FirstAsync()).ID;
  Console.WriteLine(repoResourceId); // "sesrsc_01ABC..."

  // 轮换授权令牌
  await client.Beta.Sessions.Resources.Update(repoResourceId, new()
  {
      SessionID = session.ID,
      AuthorizationToken = "ghp_your_new_github_token",
  });
  ```

  ```go Go
  // 列出会话上的资源
  listed, err := client.Beta.Sessions.Resources.List(ctx, session.ID, anthropic.BetaSessionResourceListParams{})
  if err != nil {
  	panic(err)
  }
  repoResourceID := listed.Data[0].ID
  fmt.Println(repoResourceID) // "sesrsc_01ABC..."

  // 轮换授权令牌
  _, err = client.Beta.Sessions.Resources.Update(ctx, repoResourceID, anthropic.BetaSessionResourceUpdateParams{
  	SessionID:          session.ID,
  	AuthorizationToken: "ghp_your_new_github_token",
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  // 列出会话上的资源
  var listed = client.beta().sessions().resources().list(session.id());
  var repoResourceId = listed.data().getFirst().asGitHubRepository().id();
  IO.println(repoResourceId);  // "sesrsc_01ABC..."

  // 轮换授权令牌
  client.beta().sessions().resources().update(repoResourceId, ResourceUpdateParams.builder()
      .sessionId(session.id())
      .authorizationToken("ghp_your_new_github_token")
      .build());
  ```

  ```php PHP
  // 列出会话上的资源
  $listed = $client->beta->sessions->resources->list($session->id);
  $repoResourceId = $listed->data[0]->id;
  echo $repoResourceId, PHP_EOL; // "sesrsc_01ABC..."

  // 轮换授权令牌
  $client->beta->sessions->resources->update(
      $repoResourceId,
      sessionID: $session->id,
      authorizationToken: 'ghp_your_new_github_token',
  );
  ```

  ```ruby Ruby
  # 列出会话上的资源
  listed = client.beta.sessions.resources.list(session.id)
  repo_resource_id = listed.data.first.id
  puts repo_resource_id # "sesrsc_01ABC..."

  # 轮换授权令牌
  client.beta.sessions.resources.update(
    repo_resource_id,
    session_id: session.id,
    authorization_token: "ghp_your_new_github_token"
  )
  ```
</CodeGroup>

## 创建拉取请求

借助 GitHub MCP 服务器，智能体可以创建分支、提交更改并推送它们：

<CodeGroup>
  ```bash cURL
  curl -fsS "https://api.anthropic.com/v1/sessions/$session_id/events" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {
          "type": "user.message",
          "content": [
            {
              "type": "text",
              "text": "Fix the type error in src/utils.ts, commit it to a new branch, and push it."
            }
          ]
        }
      ]
    }'
  ```

  ```bash CLI
  ant beta:sessions:events send --session-id "$SESSION_ID" <<'EOF'
  events:
    - type: user.message
      content:
        - type: text
          text: Fix the type error in src/utils.ts, commit it to a new branch, and push it.
  EOF
  ```

  ```python Python
  client.beta.sessions.events.send(
      session.id,
      events=[
          {
              "type": "user.message",
              "content": [
                  {
                      "type": "text",
                      "text": "Fix the type error in src/utils.ts, commit it to a new branch, and push it.",
                  },
              ],
          },
      ],
  )
  ```

  ```typescript TypeScript
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.message",
        content: [
          {
            type: "text",
            text: "Fix the type error in src/utils.ts, commit it to a new branch, and push it."
          }
        ]
      }
    ]
  });
  ```

  ```csharp C#
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content =
              [
                  new BetaManagedAgentsTextBlock
                  {
                      Text = "Fix the type error in src/utils.ts, commit it to a new branch, and push it.",
                  },
              ],
          },
      ],
  });
  ```

  ```go Go
  _, err = client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{
  		{
  			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{
  					{
  						OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  							Text: "Fix the type error in src/utils.ts, commit it to a new branch, and push it.",
  						},
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
  client.beta().sessions().events().send(session.id(), EventSendParams.builder()
      .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
          .addContent(BetaManagedAgentsTextBlock.builder()
              .text("Fix the type error in src/utils.ts, commit it to a new branch, and push it.")
              .build())
          .build())
      .build());
  ```

  ```php PHP
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => 'Fix the type error in src/utils.ts, commit it to a new branch, and push it.',
                  ],
              ],
          ],
      ],
  );
  ```

  ```ruby Ruby
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {
        type: "user.message",
        content: [
          {
            type: "text",
            text: "Fix the type error in src/utils.ts, commit it to a new branch, and push it."
          }
        ]
      }
    ]
  )
  ```
</CodeGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    在智能体创建拉取请求时流式传输事件并引导智能体
  </Card>

  <Card title="MCP 连接器" icon="link" href="https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector">
    连接更多 MCP 服务器，为智能体提供额外的工具
  </Card>

  <Card title="添加文件" icon="file" href="https://platform.claude.com/docs/zh-CN/managed-agents/files">
    在沙箱中与您的仓库一起挂载文件
  </Card>
</CardGroup>

## 可用性

GitHub 访问在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

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
