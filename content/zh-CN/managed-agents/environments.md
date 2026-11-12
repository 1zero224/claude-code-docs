---
title: 云环境设置
url: https://platform.claude.com/docs/zh-CN/managed-agents/environments
description: 为您的会话自定义云沙箱。
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

环境（environment）定义了您的智能体运行所在的沙箱配置。您只需创建一次环境，然后在每次启动会话时引用其 ID。多个会话可以共享同一个环境，但每个会话都会获得自己独立隔离的沙箱（一个全新的 Linux 容器）。

本页介绍 `type: cloud` 环境。要在您自己的基础设施上运行沙箱，请参阅[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)。要在两者之间做出选择，请参阅[选择智能体环境](https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment)。

## 创建环境

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/environments \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "python-dev",
      "config": {
        "type": "cloud",
        "networking": {"type": "limited", "allow_package_managers": true}
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply environment.yaml
    ```

    <File filename="environment.yaml">
      ```yaml
      # yaml-language-server: $schema=https://platform.claude.com/schemas/ant/beta/environment.json
      name: python-dev
      config:
        type: cloud
        networking:
          type: limited
          allow_package_managers: true
      ```
    </File>

    [`ant apply`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/apply) 会根据 `environment.yaml` 创建环境，打印其 ID，并将其记录在 `claude-lock.json` 中。请提交 `claude-lock.json`，这样下一次 `ant apply` 会更新此环境，而不是尝试再次创建它。
  </CodeGroupItem>

  ```python Python
  environment = client.beta.environments.create(
      name="python-dev",
      config={
          "type": "cloud",
          "networking": {"type": "limited", "allow_package_managers": True},
      },
  )

  print(f"Environment ID: {environment.id}")
  ```

  ```typescript TypeScript
  const environment = await client.beta.environments.create({
    name: "python-dev",
    config: {
      type: "cloud",
      networking: { type: "limited", allow_package_managers: true }
    }
  });

  console.log(`Environment ID: ${environment.id}`);
  ```

  ```csharp C#
  var environment = await client.Beta.Environments.Create(new()
  {
      Name = "python-dev",
      Config = new BetaCloudConfigParams
      {
          Networking = new BetaLimitedNetworkParams
          {
              AllowPackageManagers = true,
          },
      },
  });

  Console.WriteLine($"Environment ID: {environment.ID}");
  ```

  ```go Go
  environment, err := client.Beta.Environments.New(ctx, anthropic.BetaEnvironmentNewParams{
  	Name: "python-dev",
  	Config: anthropic.BetaEnvironmentNewParamsConfigUnion{
  		OfCloud: &anthropic.BetaCloudConfigParams{
  			Networking: anthropic.BetaCloudConfigParamsNetworkingUnion{
  				OfLimited: &anthropic.BetaLimitedNetworkParams{
  					AllowPackageManagers: anthropic.Bool(true),
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }

  fmt.Printf("Environment ID: %s\n", environment.ID)
  ```

  ```java Java
  var environment = client.beta().environments().create(EnvironmentCreateParams.builder()
      .name("python-dev")
      .config(BetaCloudConfigParams.builder()
          .networking(BetaLimitedNetworkParams.builder()
              .allowPackageManagers(true)
              .build())
          .build())
      .build());
  IO.println("Environment ID: " + environment.id());
  ```

  ```php PHP
  $environment = $client->beta->environments->create(
      name: 'python-dev',
      config: [
          'type' => 'cloud',
          'networking' => ['type' => 'limited', 'allow_package_managers' => true],
      ],
  );
  echo "Environment ID: {$environment->id}\n";
  ```

  ```ruby Ruby
  environment = client.beta.environments.create(
    name: "python-dev",
    config: {
      type: "cloud",
      networking: {type: "limited", allow_package_managers: true}
    }
  )

  puts "Environment ID: #{environment.id}"
  ```
</CodeGroup>

请使用唯一且具有描述性的 `name` (csharp, go: `Name`)，以便区分不同的环境。此示例使用 `limited` [网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)并允许包管理器，因此沙箱可以访问包注册表和代码托管站点。要让它访问其他主机，请将这些主机添加到 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中。

## 在会话中使用环境

在[创建会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)时，以字符串形式传入环境 ID。

<CodeGroup>
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'"
    }'
  ```

  ```bash CLI
  ant beta:sessions create --agent "$AGENT_ID" --environment-id "$ENVIRONMENT_ID"
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
  )
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id
  });
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
  });
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String(agent.ID),
  	},
  	EnvironmentID: environment.ID,
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(agent.id())
      .environmentId(environment.id())
      .build());
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
  );
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id
  )
  ```
</CodeGroup>

## 配置选项

### 软件包

`packages` (csharp, go: `Packages`) 字段会在智能体启动之前将软件包预安装到沙箱中。软件包由各自的包管理器安装，并在共享同一环境的会话之间缓存。当指定了多个包管理器时，它们按字母顺序运行（apt、cargo、gem、go、npm、pip）。您可以选择固定特定版本。未固定版本的软件包会安装最新版本。如果环境使用 `limited` [网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)，还需将 `networking` (csharp, go: `Networking`) 中的 `allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`) 设置为 `true`；否则请求会被拒绝并返回 400 错误。

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/environments \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "data-analysis",
      "config": {
        "type": "cloud",
        "packages": {
          "pip": ["pandas", "numpy", "scikit-learn"],
          "npm": ["express"]
        },
        "networking": {"type": "limited", "allow_package_managers": true}
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply environment.yaml
    ```

    <File filename="environment.yaml">
      ```yaml
      # yaml-language-server: $schema=https://platform.claude.com/schemas/ant/beta/environment.json
      name: data-analysis
      config:
        type: cloud
        packages:
          pip:
            - pandas
            - numpy
            - scikit-learn
          npm:
            - express
        networking:
          type: limited
          allow_package_managers: true
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  environment = client.beta.environments.create(
      name="data-analysis",
      config={
          "type": "cloud",
          "packages": {
              "pip": ["pandas", "numpy", "scikit-learn"],
              "npm": ["express"],
          },
          "networking": {"type": "limited", "allow_package_managers": True},
      },
  )
  ```

  ```typescript TypeScript
  const environment = await client.beta.environments.create({
    name: "data-analysis",
    config: {
      type: "cloud",
      packages: {
        pip: ["pandas", "numpy", "scikit-learn"],
        npm: ["express"]
      },
      networking: { type: "limited", allow_package_managers: true }
    }
  });
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Environments;

  var environment = await client.Beta.Environments.Create(new()
  {
      Name = "data-analysis",
      Config = new BetaCloudConfigParams
      {
          Packages = new()
          {
              Pip = ["pandas", "numpy", "scikit-learn"],
              Npm = ["express"],
          },
          Networking = new BetaLimitedNetworkParams
          {
              AllowPackageManagers = true,
          },
      },
  });
  ```

  ```go Go
  environment, err := client.Beta.Environments.New(ctx, anthropic.BetaEnvironmentNewParams{
  	Name: "data-analysis",
  	Config: anthropic.BetaEnvironmentNewParamsConfigUnion{
  		OfCloud: &anthropic.BetaCloudConfigParams{
  			Packages: anthropic.BetaPackagesParams{
  				Pip: []string{"pandas", "numpy", "scikit-learn"},
  				Npm: []string{"express"},
  			},
  			Networking: anthropic.BetaCloudConfigParamsNetworkingUnion{
  				OfLimited: &anthropic.BetaLimitedNetworkParams{
  					AllowPackageManagers: anthropic.Bool(true),
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
  import com.anthropic.models.beta.environments.*;
  import java.util.List;

  var environment = client.beta().environments().create(EnvironmentCreateParams.builder()
      .name("data-analysis")
      .config(BetaCloudConfigParams.builder()
          .packages(BetaPackagesParams.builder()
              .pip(List.of("pandas", "numpy", "scikit-learn"))
              .npm(List.of("express"))
              .build())
          .networking(BetaLimitedNetworkParams.builder()
              .allowPackageManagers(true)
              .build())
          .build())
      .build());
  ```

  ```php PHP
  $environment = $client->beta->environments->create(
      name: 'data-analysis',
      config: [
          'type' => 'cloud',
          'packages' => [
              'pip' => ['pandas', 'numpy', 'scikit-learn'],
              'npm' => ['express'],
          ],
          'networking' => ['type' => 'limited', 'allow_package_managers' => true],
      ],
  );
  ```

  ```ruby Ruby
  environment = client.beta.environments.create(
    name: "data-analysis",
    config: {
      type: "cloud",
      packages: {
        pip: %w[pandas numpy scikit-learn],
        npm: %w[express]
      },
      networking: {type: "limited", allow_package_managers: true}
    }
  )
  ```
</CodeGroup>

支持的包管理器：

| 字段      | 包管理器           | 示例                                          |
| ------- | -------------- | ------------------------------------------- |
| `apt`   | 系统软件包（apt-get） | `"graphviz"`                                |
| `cargo` | Rust（cargo）    | `"hyperfine@1.18.0"`                        |
| `gem`   | Ruby（gem）      | `"rails:7.1.0"`                             |
| `go`    | Go 模块          | `"golang.org/x/tools/cmd/goimports@latest"` |
| `npm`   | Node.js（npm）   | `"express@4.18.0"`                          |
| `pip`   | Python（pip）    | `"sqlalchemy==2.0.30"`                      |

### 网络

`networking` (csharp, go: `Networking`) 字段控制沙箱的出站网络访问。

使用 `limited` 网络时，`allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 也适用于在 Anthropic 服务器上运行的 `web_search` 和 `web_fetch` 工具。如果 `web_fetch` 调用的 URL 所在主机与 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 不匹配，则会向智能体返回错误结果。`web_search` 会省略来自与 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 不匹配的主机的结果。`allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`) 和 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 不会为这些工具添加任何主机。当 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 未列出任何主机时，任何 `web_fetch` 或 `web_search` 调用都不会返回页面或搜索结果。您为这些工具添加到 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 的主机也会对沙箱开放。`unrestricted` 网络和自托管环境不会限制这些工具。要进一步限制它们，请在智能体工具集中该工具的条目上设置 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 或 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`)。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

| 模式             | 描述                                                                                                                                                                                                                                                                                                                                                                     |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `limited`      | 将沙箱网络访问限制为 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中的主机。将 `allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`) 和 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 设置为 `true` 可允许额外的访问。除非智能体必须访问您无法预先列出的站点，否则请使用此模式。 |
| `unrestricted` | 完全的出站网络访问，但通用安全阻止列表中的主机除外。使用前，请阅读[不受限网络的风险](https://platform.claude.com/docs/zh-CN/managed-agents/environments#risks-of-unrestricted-networking)。                                                                                                                                                                                                                      |

<Note>
  请在 API 请求中显式设置 `networking` (csharp, go: `Networking`)；省略该字段的创建请求会得到 `unrestricted`。Claude Console 中用于创建环境的表单默认选中**受限**，且不允许任何其他访问。
</Note>

以下示例创建一个使用 `limited` 网络模式的环境：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/environments \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "api-access",
      "config": {
        "type": "cloud",
        "networking": {
          "type": "limited",
          "allowed_hosts": ["api.example.com"],
          "allow_mcp_servers": true,
          "allow_package_managers": true
        }
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply environment.yaml
    ```

    <File filename="environment.yaml">
      ```yaml
      # yaml-language-server: $schema=https://platform.claude.com/schemas/ant/beta/environment.json
      name: api-access
      config:
        type: cloud
        networking:
          type: limited
          allowed_hosts:
            - api.example.com
          allow_mcp_servers: true
          allow_package_managers: true
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  environment = client.beta.environments.create(
      name="api-access",
      config={
          "type": "cloud",
          "networking": {
              "type": "limited",
              "allowed_hosts": ["api.example.com"],
              "allow_mcp_servers": True,
              "allow_package_managers": True,
          },
      },
  )
  ```

  ```typescript TypeScript
  const environment = await client.beta.environments.create({
    name: "api-access",
    config: {
      type: "cloud",
      networking: {
        type: "limited",
        allowed_hosts: ["api.example.com"],
        allow_mcp_servers: true,
        allow_package_managers: true
      }
    }
  });
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Environments;

  var environment = await client.Beta.Environments.Create(new()
  {
      Name = "api-access",
      Config = new BetaCloudConfigParams
      {
          Networking = new BetaLimitedNetworkParams
          {
              AllowedHosts = ["api.example.com"],
              AllowMcpServers = true,
              AllowPackageManagers = true,
          },
      },
  });
  ```

  ```go Go
  environment, err := client.Beta.Environments.New(ctx, anthropic.BetaEnvironmentNewParams{
  	Name: "api-access",
  	Config: anthropic.BetaEnvironmentNewParamsConfigUnion{
  		OfCloud: &anthropic.BetaCloudConfigParams{
  			Networking: anthropic.BetaCloudConfigParamsNetworkingUnion{
  				OfLimited: &anthropic.BetaLimitedNetworkParams{
  					AllowedHosts:         []string{"api.example.com"},
  					AllowMCPServers:      anthropic.Bool(true),
  					AllowPackageManagers: anthropic.Bool(true),
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
  import com.anthropic.models.beta.environments.*;
  import java.util.List;

  var environment = client.beta().environments().create(EnvironmentCreateParams.builder()
      .name("api-access")
      .config(BetaCloudConfigParams.builder()
          .networking(BetaLimitedNetworkParams.builder()
              .allowedHosts(List.of("api.example.com"))
              .allowMcpServers(true)
              .allowPackageManagers(true)
              .build())
          .build())
      .build());
  ```

  ```php PHP
  $environment = $client->beta->environments->create(
      name: 'api-access',
      config: [
          'type' => 'cloud',
          'networking' => [
              'type' => 'limited',
              'allowed_hosts' => ['api.example.com'],
              'allow_mcp_servers' => true,
              'allow_package_managers' => true,
          ],
      ],
  );
  ```

  ```ruby Ruby
  environment = client.beta.environments.create(
    name: "api-access",
    config: {
      type: "cloud",
      networking: {
        type: "limited",
        allowed_hosts: %w[api.example.com],
        allow_mcp_servers: true,
        allow_package_managers: true
      }
    }
  )
  ```
</CodeGroup>

<Info>
  请使用 `limited` 网络并配合显式的 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 列表。遵循最小权限原则，仅授予智能体所需的最低网络访问权限，并定期审核您允许的域名。
</Info>

使用 `limited` 网络且未设置其他字段时，不允许任何主机。您附加到会话的文件、记忆存储和 GitHub 仓库仍然可用。当沙箱在端口 80 或 443 上发出的请求因其主机不被允许而遭拒时，响应为 403，并会指明被阻止的主机。

使用 `limited` 网络时：

* `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 指定沙箱可以访问的域名。请指定裸主机名或通配符模式（例如 `*.example.com`）。不要包含 URL 协议、端口或路径。裸主机名仅匹配该确切主机：`example.com` 不匹配 `www.example.com`。`*.example.com` 匹配 `example.com` 的所有子域名，但不匹配 `example.com` 本身。
* `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 允许对智能体上配置的 MCP 服务器端点进行出站访问，这些端点不限于 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 数组中列出的主机。默认为 `false`。当其为 `false` 时，如果智能体声明的 MCP 服务器的主机不在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中，会话创建将失败并返回 400 错误。这同样适用于[它可以委派任务的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)。要解决此问题，请将该主机添加到 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`)，或将 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 设置为 `true`。
* `allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`) 允许对一组公共包注册表和代码托管站点进行出站访问，这些主机不限于 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 数组中列出的主机。有关列表，请参阅[包管理器主机](https://platform.claude.com/docs/zh-CN/managed-agents/environments#package-manager-hosts)。默认为 `false`。只要环境指定了 `packages` (csharp, go: `Packages`)，就请将其设置为 `true`；否则请求会被拒绝并返回 400 错误，即使注册表主机已列在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中也是如此。

#### 包管理器主机

当 `allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`) 为 `true` 时，除了 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中的主机外，沙箱还可以访问以下主机。此列表由 Anthropic 维护，并可能发生变化。

| 生态系统        | 主机                                                                                                                                                                                  |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 代码托管        | `github.com`、`api.github.com`、`codeload.github.com`、`raw.githubusercontent.com`、`objects.githubusercontent.com`、`release-assets.githubusercontent.com`、`gitlab.com`、`bitbucket.org` |
| Node.js     | `registry.npmjs.org`、`registry.yarnpkg.com`、`nodejs.org`                                                                                                                            |
| Python      | `pypi.org`、`files.pythonhosted.org`                                                                                                                                                 |
| Rust        | `crates.io`、`index.crates.io`、`static.crates.io`、`static.rust-lang.org`                                                                                                             |
| Go          | `proxy.golang.org`、`sum.golang.org`                                                                                                                                                 |
| Java        | `repo1.maven.org`、`repo.maven.apache.org`、`services.gradle.org`、`plugins.gradle.org`、`plugins-artifacts.gradle.org`                                                                 |
| Ruby        | `rubygems.org`、`index.rubygems.org`                                                                                                                                                 |
| PHP         | `packagist.org`、`repo.packagist.org`                                                                                                                                                |
| Ubuntu（apt） | `archive.ubuntu.com`、`security.ubuntu.com`、`ppa.launchpad.net`                                                                                                                      |
| 容器          | `registry-1.docker.io`、`auth.docker.io`、`production.cloudflare.docker.com`、`download.docker.com`、`ghcr.io`                                                                          |

<Warning>
  网络访问是按主机授予的，而不是按操作授予的。沙箱可以向允许的主机发送任何请求，包括 `git push` 和发布包等上传操作，并可使用命令提供的任何凭据。如果智能体处理不受信任的输入（仓库文件、获取的网页内容或第三方工具输出），一次成功的"prompt injection"（提示注入）就可能利用允许的主机将文件复制到沙箱之外。为降低此风险，请将 `bash` 工具的[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)设置为 `always_ask` 或 `auto`。如果环境未指定 `packages` (csharp, go: `Packages`)，您也可以将 `allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`) 保持为 `false`，并在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中仅列出智能体所需的主机。
</Warning>

#### 不受限网络的风险

使用 `unrestricted` 网络时，沙箱中的代码可以向互联网上的任何主机发送请求，但通用安全阻止列表中的主机除外。在选择此模式之前，请考虑智能体可以利用这种访问权限做什么：

* **智能体可以更改外部站点上的内容，而不仅仅是读取：** `bash` 工具可以发送任何请求。智能体可以发布数据、提交表单、调用 API，以及运行会更改外部站点数据的脚本。在某些站点上，即使只是获取 URL 的请求也可能更改数据。
* **默认情况下，没有任何机制会暂停这些请求：** 智能体工具集的默认[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)是 `always_allow`，因此 `bash` 命令无需批准即可运行。
* **沙箱中的任何内容都可能被传出：** 这包括文件、工具输出，以及您放入沙箱的任何凭据或密钥。
* **获取的内容可能操纵智能体：** 智能体读取的网页、API 响应和其他内容可能包含改变其下一步行为的指令（提示注入）。
* **智能体代表您行事：** 它的操作可能违反站点的服务条款，或在站点上创建账户和记录。
* **模型行为不是安全控制手段：** 智能体可能以您未要求的方式在外部站点上执行操作，包括在站点阻止请求后换一种方式重试。请使用网络设置和权限策略来限制它能做的事情。
* **安全阻止列表不是允许列表：** 它不会限制智能体访问哪些其他站点，也不会限制智能体在这些站点上做什么。

为降低这些风险，请使用带有显式主机列表的 `limited` 网络。以下 `networking` 值允许 `api.example.com`，并为安装软件包的智能体允许[包管理器主机](https://platform.claude.com/docs/zh-CN/managed-agents/environments#package-manager-hosts)：

```json
{
  "type": "limited",
  "allowed_hosts": ["api.example.com"],
  "allow_package_managers": true
}
```

如果您能列出智能体所需的站点，那么仅使用 `web_search` 和 `web_fetch` 工具的智能体不需要 `unrestricted` 网络。使用 `limited` 网络时，`allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 也适用于这些工具（请参阅[网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)），因此请在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中列出这些站点。同时在 `web_search` 的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 中列出它们，可使其在这些站点中搜索。您添加到 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 的主机也会对沙箱开放。要进一步限制这些工具，请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

仅当智能体必须访问您无法预先列出的站点时，才使用 `unrestricted`。在这种情况下，请勿将密钥和敏感文件放入沙箱，并仅向智能体提供任务所需的凭据。请考虑将 `bash` 工具的权限策略设置为 `always_ask` 或 `auto`，并[监视会话的事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。

## 环境生命周期

* 环境会一直保留，直到被显式归档或删除。
* 每个会话都会获得自己的沙箱实例，即使多个会话引用同一个环境也是如此。会话之间不共享文件系统状态。
* 环境不进行版本管理。如果您频繁更新环境，请自行记录变更，以便了解每个会话使用的是哪种配置。

## 管理环境

<CodeGroup>
  ```bash cURL
  # 列出环境
  curl -fsS https://api.anthropic.com/v1/environments \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"

  # 获取特定环境
  curl -fsS "https://api.anthropic.com/v1/environments/$ENVIRONMENT_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"

  # 归档环境（只读，不可创建新会话）
  curl -fsS -X POST "https://api.anthropic.com/v1/environments/$ENVIRONMENT_ID/archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"

  # 删除环境
  curl -fsS -X DELETE "https://api.anthropic.com/v1/environments/$ENVIRONMENT_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  # 列出环境
  ant beta:environments list

  # 获取指定环境
  ant beta:environments retrieve --environment-id "$ENVIRONMENT_ID"

  # 归档环境（只读，不可创建新会话）
  ant beta:environments archive --environment-id "$ENVIRONMENT_ID"

  # 删除环境
  ant beta:environments delete --environment-id "$ENVIRONMENT_ID"
  ```

  ```python Python
  # 列出环境
  environments = client.beta.environments.list()

  # 获取指定环境
  env = client.beta.environments.retrieve(environment.id)

  # 归档环境（只读，不可创建新会话）
  client.beta.environments.archive(environment.id)

  # 删除环境
  client.beta.environments.delete(environment.id)
  ```

  ```typescript TypeScript
  // 列出环境
  const environments = await client.beta.environments.list();

  // 检索特定环境
  const env = await client.beta.environments.retrieve(environment.id);

  // 归档环境（只读，不可创建新会话）
  await client.beta.environments.archive(environment.id);

  // 删除环境
  await client.beta.environments.delete(environment.id);
  ```

  ```csharp C#
  // 列出环境
  var environments = await client.Beta.Environments.List();

  // 检索特定环境
  var env = await client.Beta.Environments.Retrieve(environment.ID);

  // 归档环境（只读，不可创建新会话）
  await client.Beta.Environments.Archive(environment.ID);

  // 删除环境
  await client.Beta.Environments.Delete(environment.ID);
  ```

  ```go Go
  // 列出环境
  environments, err := client.Beta.Environments.List(ctx, anthropic.BetaEnvironmentListParams{})
  // ...

  // 检索特定环境
  env, err := client.Beta.Environments.Get(ctx, environment.ID, anthropic.BetaEnvironmentGetParams{})
  // ...

  // 归档环境（只读，不可创建新会话）
  _, err = client.Beta.Environments.Archive(ctx, environment.ID, anthropic.BetaEnvironmentArchiveParams{})
  // ...

  // 删除环境
  _, err = client.Beta.Environments.Delete(ctx, environment.ID, anthropic.BetaEnvironmentDeleteParams{})
  ```

  ```java Java
  // 列出环境
  var environments = client.beta().environments().list();
  // 获取指定环境
  var env = client.beta().environments().retrieve(environment.id());
  // 归档环境（只读，不可创建新会话）
  client.beta().environments().archive(environment.id());
  // 删除环境
  client.beta().environments().delete(environment.id());
  ```

  ```php PHP
  // 列出环境
  $environments = $client->beta->environments->list();
  // 获取特定环境
  $env = $client->beta->environments->retrieve($environment->id);
  // 归档环境（只读，不可创建新会话）
  $client->beta->environments->archive($environment->id);
  // 删除环境
  $client->beta->environments->delete($environment->id);
  ```

  ```ruby Ruby
  # 列出环境
  environments = client.beta.environments.list

  # 检索特定环境
  env = client.beta.environments.retrieve(environment.id)

  # 归档环境（只读，不可创建新会话）
  client.beta.environments.archive(environment.id)

  # 删除环境
  client.beta.environments.delete(environment.id)
  ```
</CodeGroup>

## 预安装的运行时

云沙箱开箱即包含常用的语言运行时、数据库和命令行工具。完整列表请参阅[云沙箱参考](https://platform.claude.com/docs/zh-CN/managed-agents/cloud-sandboxes-reference)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="云沙箱参考" icon="book" href="https://platform.claude.com/docs/zh-CN/managed-agents/cloud-sandboxes-reference">
    云沙箱中可用的预安装软件包、数据库和实用工具。
  </Card>

  <Card title="启动会话" icon="play" href="https://platform.claude.com/docs/zh-CN/managed-agents/sessions">
    创建会话以运行您的智能体并开始执行任务。
  </Card>
</CardGroup>

## 可用性

云沙箱在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                                                                                                                                                                                  | 模型                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

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
