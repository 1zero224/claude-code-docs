---
title: 保护您的 Managed Agents 部署
url: https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment
description: 限制您的智能体可以访问和执行哪些操作的设置，按重要性从高到低排列。
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

本指南列出了限制 Claude Managed Agents 上的智能体可以访问和执行哪些操作的设置，按重要性从高到低排列。每个部分都链接到完整介绍该设置的页面。

| 部分                                                                                                                                                                | 设置或决策                                          | 推荐                                                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| [选择智能体的运行位置](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#choose-where-the-agent-runs)                                                 | 云端或自托管沙箱                                       | 云端，除非工具必须在您控制的基础设施上运行                                                                                                           |
| [限制网络](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#limit-the-network)                                                                 | 环境的 `networking.type`                          | `limited`，仅包含智能体需要的主机                                                                                                           |
| [将凭据保存在保管库中](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#keep-credentials-in-vaults)                                                  | 每个环境变量凭据上的 `injection_location` 和 `networking` | 仅限请求头，并使用 `limited` 网络，仅包含该密钥所对应的主机                                                                                             |
| [将凭据保存在保管库中](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#keep-credentials-in-vaults)                                                  | 您的应用程序如何向 Claude API 进行身份验证                    | 在可行的情况下，使用仅在一个工作区中启用的工作负载身份联合规则，否则使用限定于一个工作区的 API 密钥                                                                            |
| [限制网页搜索和网页抓取](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#restrict-web-search-and-web-fetch)                                          | 每个 Web 工具上的 `allowed_domains`                  | 在每个工具上设置允许列表。在 `web_fetch` 上，仅列出智能体需要的站点。在 `web_search` 上，列出您希望从中获取结果的站点                                                        |
| [审批有风险的操作](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#approve-risky-actions)                                                         | 每个工具上的 `permission_policy`                     | 在 `bash`、`write`、`edit`、`web_fetch` 以及仅读取数据的 MCP 工具上使用 `auto`。如果您无法撤销某个工具在沙箱之外所做的更改，或者仅凭转发的最终用户文本不应使其运行，则改为在该工具上使用 `always_ask` |
| [为智能体提供最小的工具集](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#give-the-agent-the-smallest-tool-set)                                      | 智能体工具集上的 `default_config.enabled`              | `false`，然后启用智能体需要的工具                                                                                                            |
| [让不受信任的输入远离强大的工具](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#keep-untrusted-input-away-from-powerful-tools)                          | 智能体在读取不受信任的输入时持有哪些工具和凭据                        | 任何读取不受信任输入的智能体都不应持有设置为 `always_allow` 且会写入文件、运行命令或更改记录的工具                                                                       |
| [设置限制和停止方式](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#set-limits-and-a-way-to-stop)                                                 | 每个会话和部署上的 `budget`                             | 根据您预期的单次运行成本设定上限                                                                                                                |
| [可选：让离线审查器阅读每份会话记录](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring#optionally-have-an-offline-reviewer-read-every-transcript) | 由您运行的、读取每个会话事件的离线审查作业                          | 可选。按计划审查每个已进入空闲或已终止的会话                                                                                                          |
| [监控会话并查看保留策略](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring#monitor-sessions-and-review-retention)                           | Webhook 和会话删除                                  | 在会话进入空闲、达到预算或终止时发出告警，并检查其进入空闲的原因。将您想保留的事件存储在您自己的日志中，并删除不再需要的会话                                                                  |

[安全风险与沙箱边界](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-risks)解释了这些设置为何重要以及它们无法防范哪些情况。要审查和监控会话，请参阅[审查和监控智能体会话](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring)。

Claude Code 附带的 [Claude API 技能](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/claude-api-skill)提供了一个 `managed-agents-secure` 命令，可根据本指南检查智能体、其环境及其保管库：

让 Claude Code 根据本指南检查您的智能体

```text
/claude-api managed-agents-secure
```

该命令不会更改任何内容，其报告会列出每个需要更改的设置及其推荐值，以及只有您才能回答的问题。运行之前，请更新 Claude Code，并按照 [CLI 快速入门](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart)所示为智能体的工作区设置 `ant` CLI。

## 选择智能体的运行位置

智能体工具的运行位置决定了由谁来保护沙箱：使用云环境时由 Anthropic 负责，使用自托管沙箱时由您负责。要在两者之间做出选择，并了解每种方式下哪些内容会到达 Anthropic，请参阅[选择智能体环境](https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment)。

如果您使用[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)，请牢记以下四点：

* **网络：** [限制网络](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#limit-the-network)适用于云环境。对于自托管沙箱，请在您自己的防火墙中对其流量设置同等的限制。您的防火墙不会限制智能体声明的 MCP 服务器，因此请审查这些服务器，以及您的应用程序为会话设置的任何服务器。
* **Web 工具：** Web 工具在 Anthropic 的服务器上运行，因此您的防火墙不会限制它们。请使用它们的域名列表来限制它们。请参阅[限制网页搜索和网页抓取](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#restrict-web-search-and-web-fetch)。
* **凭据：** 自托管沙箱尚不支持环境变量凭据。
* **隔离和加固：** 这两项都由您负责。请参阅[安全模型](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-security)。

这两种选项都不会改变 Managed Agents 的合规覆盖范围，也不会改变其保留会话历史的时长。请参阅[监控会话并查看保留策略](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring#monitor-sessions-and-review-retention)。

## 限制网络

使用 `limited` 网络，并在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中仅列出智能体需要的主机。省略 `networking` (csharp, go: `Networking`) 的环境创建请求可能会得到 `unrestricted`，因此请在每个创建请求中设置 `networking` (csharp, go: `Networking`)。请参阅[不受限网络的风险](https://platform.claude.com/docs/zh-CN/managed-agents/environments#risks-of-unrestricted-networking)。

无论 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 如何设置，您附加的文件、记忆存储和 GitHub 仓库始终可供智能体使用。

本页的示例使用一个客服智能体。它读取客户的工单，查询订单，阅读您的帮助中心，并起草回复。订单查询由您的应用程序运行，不需要任何主机，因此该环境只为 `web_fetch` 允许一个主机，即帮助中心：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/environments \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "support-agent",
      "config": {
        "type": "cloud",
        "networking": {
          "type": "limited",
          "allowed_hosts": ["help.example.com"]
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
      name: support-agent
      config:
        type: cloud
        networking:
          type: limited
          allowed_hosts:
            - help.example.com
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  environment = client.beta.environments.create(
      name="support-agent",
      config={
          "type": "cloud",
          "networking": {
              "type": "limited",
              "allowed_hosts": ["help.example.com"],
          },
      },
  )
  ```

  ```typescript TypeScript
  const environment = await client.beta.environments.create({
    name: "support-agent",
    config: {
      type: "cloud",
      networking: {
        type: "limited",
        allowed_hosts: ["help.example.com"]
      }
    }
  });
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Environments;

  var environment = await client.Beta.Environments.Create(new()
  {
      Name = "support-agent",
      Config = new BetaCloudConfigParams
      {
          Networking = new BetaLimitedNetworkParams
          {
              AllowedHosts = ["help.example.com"],
          },
      },
  });
  ```

  ```go Go
  environment, err := client.Beta.Environments.New(ctx, anthropic.BetaEnvironmentNewParams{
  	Name: "support-agent",
  	Config: anthropic.BetaEnvironmentNewParamsConfigUnion{
  		OfCloud: &anthropic.BetaCloudConfigParams{
  			Networking: anthropic.BetaCloudConfigParamsNetworkingUnion{
  				OfLimited: &anthropic.BetaLimitedNetworkParams{
  					AllowedHosts: []string{"help.example.com"},
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
      .name("support-agent")
      .config(BetaCloudConfigParams.builder()
          .networking(BetaLimitedNetworkParams.builder()
              .allowedHosts(List.of("help.example.com"))
              .build())
          .build())
      .build());
  ```

  ```php PHP
  $environment = $client->beta->environments->create(
      name: 'support-agent',
      config: [
          'type' => 'cloud',
          'networking' => [
              'type' => 'limited',
              'allowed_hosts' => ['help.example.com'],
          ],
      ],
  );
  ```

  ```ruby Ruby
  environment = client.beta.environments.create(
    name: "support-agent",
    config: {
      type: "cloud",
      networking: {
        type: "limited",
        allowed_hosts: ["help.example.com"]
      }
    }
  )
  ```
</CodeGroup>

`limited` 环境有三个网络设置：

| 设置                                                                                                                      | 向沙箱中的代码开放的内容                                                                                                                                                                                     | 推荐                                                                                                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`)                                                 | 您列出的主机，可用于任何请求                                                                                                                                                                                   | 仅限智能体需要的主机，包括您运营或信任的每个 MCP 服务器的主机                                                                                                                                                    |
| `allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`)                        | [包管理器主机](https://platform.claude.com/docs/zh-CN/managed-agents/environments#package-manager-hosts)。其中包括 `github.com`、`gitlab.com` 和 `bitbucket.org` 上的每个仓库和路径，任何人都可以在这些地方发布内容，沙箱中的代码也可以向这些地方上传文件 | `false`，除非环境的 `packages` (csharp, go: `Packages`) 设置列出了软件包，这种情况需要设为 `true`。如果智能体在运行时安装软件包，请改为在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中仅列出它需要的注册表主机 |
| `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) | 不开放任何主机。Managed Agents 从 Anthropic 的服务器连接到 MCP 服务器                                                                                                                                               | `false`，并在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中列出您运营或信任的每个 MCP 服务器的主机                                                                           |

`allow_package_managers` (java, php: `allowPackageManagers`; csharp, go: `AllowPackageManagers`) 和 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 设置不会为网页搜索和网页抓取添加任何主机。

保持 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 关闭（这是默认设置），并在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中列出您运营或信任的每个 MCP 服务器的主机。您列出的每个主机也会向沙箱中的代码开放。在 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 关闭的情况下，如果某个 MCP 服务器的主机未被列出，会话创建时的检查会拒绝该请求。请参阅[在会话创建时提供身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector#provide-authentication-at-session-creation)。当会话更新更改 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 时，也会运行相同的检查。

<Warning>
  网络访问是按主机授予的，而不是按操作授予的。沙箱可以向允许的主机发送任何请求，包括 `git push` 和发布软件包等上传操作。它还可以使用命令提供的任何凭据，即使是有人隐藏在智能体所读取文本中的凭据。
</Warning>

为 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 选择主机时，请遵循以下两条规则：

* 仅列出您运营或信任的主机。
* 避免使用 `*.` 通配符以及任何人都可以发布内容的共享托管域名。

如果智能体保留 `bash`，请按照[审批有风险的操作](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#approve-risky-actions)中的说明设置 `bash` 工具的[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)。

<Note>
  收紧环境设置后，请启动新会话并停止已在运行的会话。已在运行的会话可能会保留旧的限制，直到其结束。请参阅[设置限制和停止方式](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#set-limits-and-a-way-to-stop)。
</Note>

有关所有网络设置，请参阅[云环境设置中的网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)。

## 将凭据保存在保管库中

将每个密钥都存储在保管库中。对于每个环境变量凭据，将注入位置设置为仅限请求头，并仅列出该密钥所针对的主机。沙箱中的代码看到的是占位符而非密钥。Managed Agents 仅在发往该凭据所允许主机的出站 HTTPS 请求中替换为真实密钥。因此，密钥不会被模型直接获取，除非某个服务将其发回，例如会回显请求头的端点。如果凭据的 `networking` (csharp, go: `Networking`) 为 `unrestricted`，Managed Agents 会在发往环境允许沙箱访问的任何主机的请求中替换为真实密钥。要使用保管库，请在创建会话或部署时在 `vault_ids` (java: `vaultIds`; php: `vaultIDs`; csharp: `VaultIds`; go: `VaultIDs`) 中传入其 ID。

| 设置                                                                                     | 设置为                                                                                            | 原因                                                                           | 它不能做什么                                           |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------ |
| `injection_location` (java, php: `injectionLocation`; csharp, go: `InjectionLocation`) | `{"header": true, "body": false}`，在您确认服务从请求头读取密钥之后                                             | 请求体通常由智能体读取的内容构建，因此请求体中的密钥暴露风险更高。请同时指定两个字段：省略该对象的创建请求会同时启用两个位置，而更新只会更改您发送的字段 | 代码仍然可以在凭据允许的每个主机上使用该密钥。期望在请求体中接收密钥的服务会收到占位符并拒绝请求 |
| 凭据上的 `networking` (csharp, go: `Networking`)                                           | `limited`，并列出该密钥所对应的确切主机                                                                       | 响应会未经脱敏地到达沙箱，因此请避免使用通配符、共享托管域名以及回显请求头的主机                                     | 它不会开放该主机。环境也必须允许该主机                              |
| 密钥的权限，在签发该密钥的服务处设置                                                                     | 仅限智能体需要的权限                                                                                     | 智能体可以执行该密钥允许的任何操作                                                            | 不适用                                              |
| 已挂载仓库的 GitHub 令牌                                                                       | 具有最低权限的[细粒度令牌](https://platform.claude.com/docs/zh-CN/managed-agents/github#token-permissions) | 沙箱中的代码可以在整个会话期间使用它从已挂载的仓库拉取和推送。仅当智能体必须推送时才授予写入权限                             | 不适用                                              |

客服智能体不需要保管库凭据，因为您的应用程序持有订单 API 的 API 密钥。如果您的智能体确实持有保管库凭据，请参阅[添加凭证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults#add-a-credential)，了解各种语言的请求示例。

不要将密钥放在系统提示、智能体定义和消息中。不要向智能体提供 Claude API 密钥或 Claude API 访问令牌。即使它存放在保管库中，沙箱中的代码也可以用它直接调用 Claude API。它可以执行该凭据允许的任何操作，例如启动或删除会话，或者使用网页搜索和网页抓取。智能体的工具设置、审批和预算不适用于这些调用。在 `limited` 环境中，只有当 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 允许沙箱访问 `api.anthropic.com` 时，代码才能进行这些调用。

请自行获取第三方令牌并将其存储在保管库中。沙箱中的代码使用已存储的密钥获取的令牌会以未脱敏的形式返回。

将不同信任级别工作负载的保管库放在不同的[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)中，因为任何能够在某个工作区中启动会话的人都可以引用该工作区的保管库。将每个工作负载限制在一个工作区内：在可行的情况下，使用仅在该工作区中启用的[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)规则，否则使用限定于该工作区的 API 密钥。工作负载身份联合为您的应用程序提供短期有效的 Claude API 访问令牌，因此不存在可能泄露的长期 Claude API 密钥。

有关 MCP 凭据和轮换，请参阅[使用 vault 进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)。

## 限制网页搜索和网页抓取

将 `web_fetch` 上的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 设置为智能体需要的站点。智能体获取的任何 URL 都可能将数据携带到其指向的站点，因此此列表是一项安全控制措施。在 `web_search` 上，将 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 设置为您希望从中获取结果的站点，以便结果仅来自您信任其准确性的站点。该列表不会阻止搜索查询到达搜索提供商。搜索结果包含来自页面的文本，因此该列表也会限制哪些站点的文本可以到达智能体。对于每种类型的环境，都请在这两个工具上设置 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`)。

`blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`) 列表会阻止您列出的主机及其子域名，而不会阻止其他主机。相同的内容仍可能通过网页存档、缓存、镜像或您未列出的相关域名到达智能体。不要依赖 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`) 来保障安全。请使用 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 并指定您信任的主机。

管理员可以在 Claude Console 的 [**功能**](https://platform.claude.com/settings/capabilities) 页面上关闭网页搜索或网页抓取，或限制它们的域名。这些设置也适用于会话。当某个工具被关闭时，其调用会向智能体返回错误结果，会话会继续进行。当设置了域名限制时，搜索结果或获取的 URL 必须同时通过组织的限制和该工具自身的域名列表。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

在 `limited` 环境中，[限制网络](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#limit-the-network)中的 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 列表也会限制 Web 工具。`unrestricted` 环境或自托管沙箱不会限制 Web 工具。

| 在 `limited` 环境中                                            | 结果                                                                                             |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| 已启用的 Web 工具上的某个 `allowed_domains` 条目不在 `allowed_hosts` 范围内 | 创建会话会失败并返回 400 错误，添加此类条目的会话更新也会失败。由部署按计划或通过手动运行启动的会话仍会被创建。之后该工具不会从 `allowed_hosts` 之外的主机返回任何内容 |
| 某个 Web 工具没有自己的列表                                           | 该工具仅返回来自 `allowed_hosts` 中主机的页面和结果。也请在该工具上设置 `allowed_domains`                                 |
| `allowed_hosts` 没有任何条目                                     | 两个 Web 工具都不会返回任何页面或搜索结果                                                                        |
| 您为某个 Web 工具向 `allowed_hosts` 添加了一个主机                       | 该主机也会向沙箱中的代码开放                                                                                 |

这两个列表匹配主机的方式不同：

| 列表                        | 条目              | 匹配                    | 不匹配               |
| ------------------------- | --------------- | --------------------- | ----------------- |
| `allowed_hosts`（环境）       | `example.com`   | `example.com`         | `www.example.com` |
| `allowed_hosts`（环境）       | `*.example.com` | 每个子域名                 | `example.com`     |
| `allowed_domains`（Web 工具） | `example.com`   | `example.com` 及其每个子域名 | 其他域名              |

如果智能体需要某个站点及其所有子域名，请在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中同时列出 `example.com` 和 `*.example.com`，否则请列出确切的主机。如果某个工具列出了 `example.com`，而 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 仅列出了 `*.example.com`，会话仍会被创建。会话创建时的检查会将 `example.com` 视为在 `*.example.com` 范围内，但该工具不会从 `example.com` 本身返回任何内容。有关这两个列表如何匹配，请参阅[在智能体上设置域名列表](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#set-domain-lists-on-an-agent)。

`web_fetch` 工具也只会打开已在会话中出现过的 URL：

| URL 出现的位置                                | `web_fetch` 能否打开            |
| ---------------------------------------- | --------------------------- |
| 用户消息的文本，包括您转发的最终用户文本                     | 能                           |
| `user.define_outcome` 事件的描述              | 能                           |
| 网页搜索结果的地址                                | 能                           |
| `web_fetch` 之前返回的页面文本（非 PDF），其中显示了完整 URL | 能                           |
| 自定义工具结果的文本，当 Claude 直接调用该工具时             | 能，除非结果被标记为 `is_error: true` |
| 系统提示                                     | 不能                          |
| 智能体的配置，包括 `allowed_domains` 条目           | 不能                          |
| Claude 自己的输出                             | 不能                          |
| `bash` 和其他沙箱工具的输出                        | 不能                          |
| MCP 工具结果                                 | 不能                          |
| 附加到用户消息的文件                               | 不能                          |

在多智能体会话中，只有当来自另一个智能体的 URL 是通过标记为"能"的来源到达该智能体时，`web_fetch` 才会打开它。直接调用的自定义工具结果算作标记为"能"的来源。即使其文本是由 Claude 编写的（例如当工具重复其输入时），这一点也成立。即使 URL 在书写时没有 `https://`（例如 `example.com/docs`），也可能被计入。`web_fetch` 工具并不总能打开显示文字而非 URL 的链接，例如"联系支持"。如果智能体必须获取尚未在会话中出现过的 URL，请在用户消息中发送该 URL。

此规则降低了注入文本构造出携带私密数据的 URL 的可能性。最终用户文本、搜索结果、获取的页面和自定义工具结果仍然可以添加 URL，因此请保持 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 的设置。Messages API 采用了类似的规则，但来源列表有所不同。请参阅 [URL 验证](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#url-validation)。

`web_fetch` 上的域名列表不会限制 `bash` 工具。请同时[限制网络](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#limit-the-network)。请参阅[域名列表规则](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#domain-list-rules)。

## 审批有风险的操作

在 `bash`、`write`、`edit` 和 `web_fetch` 上，以及仅读取数据的 MCP 工具上设置 `auto`。如果您无法撤销某个工具在沙箱之外所做的更改，请改为在该工具上使用 `always_ask`。对于不应仅因您转发的最终用户文本要求就运行的工具，也请使用它。[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)决定工具调用是直接运行、等待您的审批，还是由服务器进行评估。

| 工具                    | 推荐策略                                                                                      | 原因                                                                                                     |
| --------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `read`、`glob`、`grep`  | `always_allow`                                                                            | 它们读取沙箱中的文件，不做任何更改                                                                                      |
| `bash`、`write`、`edit` | `auto`。如果您无法撤销某个工具在沙箱之外所做的更改，请在该工具上使用 `always_ask`。如果您转发最终用户文本，请在 `bash` 上使用 `always_ask` | 智能体工具集的默认值是 `always_allow`，它会在未经审批的情况下运行每个调用                                                           |
| `web_fetch`           | `auto`                                                                                    | 智能体获取的 URL 可能将数据携带到其指向的站点                                                                              |
| `web_search`          | 如果智能体处理您不希望在搜索查询中发送的数据，则使用 `always_ask`。否则使用 `always_allow`                               | `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 限制的是结果来自哪些站点，而不是查询包含什么内容 |
| 仅读取数据的 MCP 工具         | `auto`，在工具集的 `configs` (csharp, go: `Configs`) 中为每个工具单独设置                                 | 工具集默认值保持为 `always_ask`，因此 MCP 服务器之后添加的工具仍会等待审批                                                         |
| 更改数据的 MCP 工具          | `always_ask`                                                                              | 这是 MCP 工具集的默认值。请保持不变                                                                                   |
| 自定义工具                 | 由您的代码决定每个调用是否运行                                                                           | 权限策略不适用                                                                                                |

[为智能体提供最小的工具集](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#give-the-agent-the-smallest-tool-set)中的工具集应用了这些策略。

[`auto` 策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#let-the-server-evaluate-each-call-with-auto)允许服务器运行调用、拒绝调用或暂停以等待审批。在长时间运行中，人工无法审批每个调用，因此改由服务器检查每个调用。请将 `auto` 与受限网络、权限范围窄的密钥、预算以及空闲会话告警搭配使用。`auto` 不是人工检查点，因此如果必须由人工在工具调用运行前进行审查，请在该工具上配置 `always_ask`。

`auto` 会将 `user.message` 事件中的文本以及 `user.define_outcome` 事件中的描述视为您的指令，包括您转发的最终用户文本。

暂停的会话会无限期等待，因此无人值守的会话需要一个能够响应暂停调用或中断会话的客户端。

在允许每个暂停的调用之前，请先对其进行审查。一个不向人工展示就允许每个调用的客户端，等同于 `always_allow`。请参阅[响应确认请求](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#respond-to-confirmation-requests)和[查看每次调用的评估方式](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#see-how-each-call-was-evaluated)。

对于能够转移资金的工具（例如用于付款、提现、转账或支付款项的工具），请让人工审批对其的每个调用。如果该工具接受权限策略，请将其设置为 `always_ask`，而不是 `always_allow` 或 `auto`。如果它是自定义工具，请在人工审批之前不要运行调用。有关可添加到智能体系统提示中的文本，请参阅[付款、账号和财务记录](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices#payments-account-numbers-and-financial-records)。

## 为智能体提供最小的工具集

将 `default_config.enabled` (java, php: `defaultConfig.enabled`; csharp, go: `DefaultConfig.Enabled`) 设置为 `false`，并[仅启用智能体需要的工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#enabling-only-specific-tools)。智能体工具集默认启用全部八个内置工具。

| 工具                       | 作用                              |
| ------------------------ | ------------------------------- |
| `bash`                   | 运行任何命令，包括向允许的主机发出请求             |
| `write`、`edit`           | 更改沙箱中的文件                        |
| `read`、`glob`、`grep`     | 读取沙箱中的文件                        |
| `web_fetch`、`web_search` | 通过 Anthropic 的服务器返回来自网络的页面和搜索结果 |

客服智能体需要三个内置工具：用于起草回复的 `read` 和 `write`，以及用于访问帮助中心的 `web_fetch`。它通过由您的应用程序运行的[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#custom-tools) `lookup_order` 查询订单。您的应用程序持有订单 API 的 API 密钥，因此智能体既不需要 `bash`，也不需要凭据。

以下示例使用 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 将 `web_fetch` 限制为 `help.example.com`。它按照[审批有风险的操作](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#approve-risky-actions)的建议，在 `write` 和 `web_fetch` 上设置了 `auto`。如果客户的工单导致智能体使用这些工具，智能体只能更改沙箱中的文件或打开帮助中心页面。这就是这两个工具都不需要 `always_ask` 的原因。该示例还在 `default_config` (java, php: `defaultConfig`; csharp, go: `DefaultConfig`) 中设置了 `always_ask`，因此您以后启用的、没有自身策略的工具会使用 `always_ask`：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Support Agent",
      "model": "claude-opus-5-5",
      "tools": [
        {
          "type": "agent_toolset_20260401",
          "default_config": {
            "enabled": false,
            "permission_policy": {"type": "always_ask"}
          },
          "configs": [
            {"name": "read", "enabled": true, "permission_policy": {"type": "always_allow"}},
            {"name": "write", "enabled": true, "permission_policy": {"type": "auto"}},
            {
              "name": "web_fetch",
              "enabled": true,
              "allowed_domains": ["help.example.com"],
              "permission_policy": {"type": "auto"}
            }
          ]
        },
        {
          "type": "custom",
          "name": "lookup_order",
          "description": "Look up one order by its ID. Returns the status, the items, and the delivery date.",
          "input_schema": {
            "type": "object",
            "properties": {
              "order_id": {"type": "string", "description": "The order ID from the ticket"}
            },
            "required": ["order_id"]
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
      name: Support Agent
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
          default_config:
            enabled: false
            permission_policy:
              type: always_ask
          configs:
            - name: read
              enabled: true
              permission_policy:
                type: always_allow
            - name: write
              enabled: true
              permission_policy:
                type: auto
            - name: web_fetch
              enabled: true
              allowed_domains: [help.example.com]
              permission_policy:
                type: auto
        - type: custom
          name: lookup_order
          description: Look up one order by its ID. Returns the status, the items, and the delivery date.
          input_schema:
            type: object
            properties:
              order_id:
                type: string
                description: The order ID from the ticket
            required:
              - order_id
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  tools = [
      {
          "type": "agent_toolset_20260401",
          "default_config": {
              "enabled": False,
              "permission_policy": {"type": "always_ask"},
          },
          "configs": [
              {
                  "name": "read",
                  "enabled": True,
                  "permission_policy": {"type": "always_allow"},
              },
              {
                  "name": "write",
                  "enabled": True,
                  "permission_policy": {"type": "auto"},
              },
              {
                  "name": "web_fetch",
                  "enabled": True,
                  "allowed_domains": ["help.example.com"],
                  "permission_policy": {"type": "auto"},
              },
          ],
      },
      {
          "type": "custom",
          "name": "lookup_order",
          "description": "Look up one order by its ID. Returns the status, the items, and the delivery date.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "order_id": {
                      "type": "string",
                      "description": "The order ID from the ticket",
                  },
              },
              "required": ["order_id"],
          },
      },
  ]
  ```

  ```typescript TypeScript
  const tools = [
    {
      type: "agent_toolset_20260401",
      default_config: {
        enabled: false,
        permission_policy: { type: "always_ask" }
      },
      configs: [
        {
          name: "read",
          enabled: true,
          permission_policy: { type: "always_allow" }
        },
        {
          name: "write",
          enabled: true,
          permission_policy: { type: "auto" }
        },
        {
          name: "web_fetch",
          enabled: true,
          allowed_domains: ["help.example.com"],
          permission_policy: { type: "auto" }
        }
      ]
    },
    {
      type: "custom",
      name: "lookup_order",
      description:
        "Look up one order by its ID. Returns the status, the items, and the delivery date.",
      input_schema: {
        type: "object",
        properties: {
          order_id: { type: "string", description: "The order ID from the ticket" }
        },
        required: ["order_id"]
      }
    }
  ] satisfies Anthropic.Beta.AgentCreateParams["tools"];
  ```

  ```csharp C#
  using System.Text.Json;
  using Anthropic.Models.Beta.Agents;
  using Tool = Anthropic.Models.Beta.Agents.Tool;

  Tool[] tools =
  [
      new BetaManagedAgentsAgentToolset20260401Params
      {
          DefaultConfig = new()
          {
              Enabled = false,
              PermissionPolicy = new BetaManagedAgentsAlwaysAskPolicy(),
          },
          Configs =
          [
              new BetaManagedAgentsReadToolConfigParams
              {
                  Enabled = true,
                  PermissionPolicy = new BetaManagedAgentsAlwaysAllowPolicy(),
              },
              new BetaManagedAgentsWriteToolConfigParams
              {
                  Enabled = true,
                  PermissionPolicy = new BetaManagedAgentsAutoPolicy(),
              },
              new BetaManagedAgentsWebFetchToolConfigParams
              {
                  Enabled = true,
                  AllowedDomains = ["help.example.com"],
                  PermissionPolicy = new BetaManagedAgentsAutoPolicy(),
              },
          ],
      },
      new BetaManagedAgentsCustomToolParams
      {
          Name = "lookup_order",
          Description = "Look up one order by its ID. Returns the status, the items, and the delivery date.",
          InputSchema = new()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["order_id"] = JsonSerializer.SerializeToElement(
                      new { type = "string", description = "The order ID from the ticket" }
                  ),
              },
              Required = ["order_id"],
          },
      },
  ];
  ```

  ```go Go
  tools := []anthropic.BetaAgentNewParamsToolUnion{{
  	OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
  		DefaultConfig: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParams{
  			Enabled: anthropic.Bool(false),
  			PermissionPolicy: anthropic.BetaManagedAgentsAgentToolsetDefaultConfigParamsPermissionPolicyUnion{
  				OfAlwaysAsk: &anthropic.BetaManagedAgentsAlwaysAskPolicyParam{},
  			},
  		},
  		Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{
  			{OfRead: &anthropic.BetaManagedAgentsReadToolConfigParams{
  				Enabled: anthropic.Bool(true),
  				PermissionPolicy: anthropic.BetaManagedAgentsReadToolConfigParamsPermissionPolicyUnion{
  					OfAlwaysAllow: &anthropic.BetaManagedAgentsAlwaysAllowPolicyParam{},
  				},
  			}},
  			{OfWrite: &anthropic.BetaManagedAgentsWriteToolConfigParams{
  				Enabled: anthropic.Bool(true),
  				PermissionPolicy: anthropic.BetaManagedAgentsWriteToolConfigParamsPermissionPolicyUnion{
  					OfAuto: &anthropic.BetaManagedAgentsAutoPolicyParam{},
  				},
  			}},
  			{OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{
  				AllowedDomains: []string{"help.example.com"},
  				Enabled:        anthropic.Bool(true),
  				PermissionPolicy: anthropic.BetaManagedAgentsWebFetchToolConfigParamsPermissionPolicyUnion{
  					OfAuto: &anthropic.BetaManagedAgentsAutoPolicyParam{},
  				},
  			}},
  		},
  	},
  }, {
  	OfCustom: &anthropic.BetaManagedAgentsCustomToolParams{
  		Name:        "lookup_order",
  		Description: "Look up one order by its ID. Returns the status, the items, and the delivery date.",
  		InputSchema: anthropic.BetaManagedAgentsCustomToolInputSchemaParam{
  			Properties: map[string]any{
  				"order_id": map[string]any{
  					"type":        "string",
  					"description": "The order ID from the ticket",
  				},
  			},
  			Required: []string{"order_id"},
  		},
  	},
  }}
  ```

  ```java Java
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.beta.agents.*;
  import java.util.List;
  import java.util.Map;

  var tools = List.of(
      AgentCreateParams.Tool.ofAgentToolset20260401(
          BetaManagedAgentsAgentToolset20260401Params.builder()
              .defaultConfig(
                  BetaManagedAgentsAgentToolsetDefaultConfigParams.builder()
                      .enabled(false)
                      .permissionPolicy(BetaManagedAgentsAlwaysAskPolicy.builder().build())
                      .build()
              )
              .addConfig(
                  BetaManagedAgentsReadToolConfigParams.builder()
                      .enabled(true)
                      .permissionPolicy(BetaManagedAgentsAlwaysAllowPolicy.builder().build())
                      .build()
              )
              .addConfig(
                  BetaManagedAgentsWriteToolConfigParams.builder()
                      .enabled(true)
                      .permissionPolicy(BetaManagedAgentsAutoPolicy.builder().build())
                      .build()
              )
              .addConfig(
                  BetaManagedAgentsWebFetchToolConfigParams.builder()
                      .enabled(true)
                      .allowedDomains(List.of("help.example.com"))
                      .permissionPolicy(BetaManagedAgentsAutoPolicy.builder().build())
                      .build()
              )
              .build()
      ),
      AgentCreateParams.Tool.ofCustom(
          BetaManagedAgentsCustomToolParams.builder()
              .name("lookup_order")
              .description("Look up one order by its ID. Returns the status, the items, and the delivery date.")
              .inputSchema(
                  BetaManagedAgentsCustomToolInputSchema.builder()
                      .properties(
                          BetaManagedAgentsCustomToolInputSchema.Properties.builder()
                              .putAdditionalProperty("order_id", JsonValue.from(Map.of(
                                  "type", "string",
                                  "description", "The order ID from the ticket")))
                              .build()
                      )
                      .addRequired("order_id")
                      .build()
              )
              .build()
      )
  );
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolsetDefaultConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsAlwaysAllowPolicy;
  use Anthropic\Beta\Agents\BetaManagedAgentsAlwaysAskPolicy;
  use Anthropic\Beta\Agents\BetaManagedAgentsAutoPolicy;
  use Anthropic\Beta\Agents\BetaManagedAgentsCustomToolInputSchema;
  use Anthropic\Beta\Agents\BetaManagedAgentsCustomToolParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsReadToolConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebFetchToolConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsWriteToolConfigParams;

  $tools = [
      BetaManagedAgentsAgentToolset20260401Params::with(
          defaultConfig: BetaManagedAgentsAgentToolsetDefaultConfigParams::with(
              enabled: false,
              permissionPolicy: BetaManagedAgentsAlwaysAskPolicy::with(),
          ),
          configs: [
              BetaManagedAgentsReadToolConfigParams::with(
                  enabled: true,
                  permissionPolicy: BetaManagedAgentsAlwaysAllowPolicy::with(),
              ),
              BetaManagedAgentsWriteToolConfigParams::with(
                  enabled: true,
                  permissionPolicy: BetaManagedAgentsAutoPolicy::with(),
              ),
              BetaManagedAgentsWebFetchToolConfigParams::with(
                  allowedDomains: ['help.example.com'],
                  enabled: true,
                  permissionPolicy: BetaManagedAgentsAutoPolicy::with(),
              ),
          ],
      ),
      BetaManagedAgentsCustomToolParams::with(
          name: 'lookup_order',
          description: 'Look up one order by its ID. Returns the status, the items, and the delivery date.',
          inputSchema: BetaManagedAgentsCustomToolInputSchema::with(
              properties: ['order_id' => ['type' => 'string', 'description' => 'The order ID from the ticket']],
              required: ['order_id'],
          ),
      ),
  ];
  ```

  ```ruby Ruby
  tools = [
    {
      type: "agent_toolset_20260401",
      default_config: {
        enabled: false,
        permission_policy: {type: "always_ask"}
      },
      configs: [
        {
          name: "read",
          enabled: true,
          permission_policy: {type: "always_allow"}
        },
        {
          name: "write",
          enabled: true,
          permission_policy: {type: "auto"}
        },
        {
          name: "web_fetch",
          enabled: true,
          allowed_domains: ["help.example.com"],
          permission_policy: {type: "auto"}
        }
      ]
    },
    {
      type: "custom",
      name: "lookup_order",
      description: "Look up one order by its ID. Returns the status, the items, and the delivery date.",
      input_schema: {
        type: "object",
        properties: {order_id: {type: "string", description: "The order ID from the ticket"}},
        required: ["order_id"]
      }
    }
  ]
  ```
</CodeGroup>

[智能体更新](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-semantics)会替换整个 `tools` (csharp, go: `Tools`) 数组，因此请发送您想保留的每个条目。

智能体更新只会作用于新会话，因此还请检查部署、列出此智能体的其他智能体以及现有会话：

* 部署会固定一个智能体版本，因此请将每个部署指向新版本。
* 在 [`subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#list-the-subagents)或 `workflows` (csharp, go: `Workflows`) 的该列表中列出此智能体的智能体，会保留其固定的版本。请同时更新该智能体。
* 要收紧已存在的会话，请[更新该会话的工具](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#updating-the-agent-configuration)。会话必须处于 `idle` 状态，且更新会替换整个 `tools` (csharp, go: `Tools`) 数组。在多智能体会话中，会话所运行智能体的 Web 工具上新的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 和 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`) 列表也会应用于其他所有智能体。它们从每个智能体的下一轮开始生效。这些智能体已启用的工具和权限策略不会改变。要更改这些内容，请更新每个智能体，然后启动新会话。

同样，请将 MCP 服务器和自定义工具限制在智能体需要的范围内：

* **MCP 服务器：** MCP 工具集默认启用服务器公开的每个工具。将 `default_config.enabled` (java, php: `defaultConfig.enabled`; csharp, go: `DefaultConfig.Enabled`) 设置为 `false` 并启用指定的工具，这样服务器以后添加的工具会保持关闭状态。移除智能体不使用的服务器。请参阅 [MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector#configure-which-mcp-tools-are-available)。
* **自定义工具：** 由您的应用程序运行，权限策略不适用。请在您的应用程序中验证每个调用的输入，并限制该工具可以触及的范围。

## 让不受信任的输入远离强大的工具

每当智能体读取不受信任的输入（即您团队之外的人可以编写的文本）时，请检查它持有哪些工具和凭据。本节使用两个术语：

* **不受信任的输入（untrusted input）：** 网页、搜索结果、支持工单、电子邮件、代码仓库文件、上传内容、MCP 服务器结果，以及您转发的最终用户消息
* **强大的工具（powerful tools）：** `bash`、`write`、`edit`、`web_fetch`、`web_search`、会写入或发送数据的 MCP 工具和自定义工具，以及任何使用凭据的工具

不受信任输入中的文本可以指示智能体使用其持有的强大工具。客服智能体会读取工单，因此它只能通过自定义工具 `lookup_order` 访问您的订单 API。您的应用程序会检查该订单是否属于工单的发送者。

| 智能体读取的输入       | 它同时拥有的能力                                                                                                  | 应对措施                                                                                                                                                                                          |
| -------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 工单、电子邮件或上传内容   | 带有凭据的 `bash`                                                                                              | 用只做一件事的自定义工具替换 `bash`。如果智能体必须保留 `bash`，请使用 `always_ask` 审批每个调用                                                                                                                                |
| 网页或搜索结果        | 写入文件或更改记录                                                                                                 | 关闭 Web 工具，或将其限制为您运营或信任的站点                                                                                                                                                                     |
| MCP 结果或仓库文件    | 会写入的 MCP 工具                                                                                               | 将 MCP 工具集保持为 `always_ask`，并仅启用智能体需要的工具                                                                                                                                                        |
| 已挂载的 GitHub 仓库 | 任何工具                                                                                                      | 仅挂载您信任的仓库。拥有 `read` 的智能体会将仓库 `.claude/skills` 目录中的技能作为智能体指令加载，且没有审查步骤。请参阅[从 GitHub 仓库加载技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills#load-skills-from-a-github-repository) |
| 任何不受信任的输入      | 写入[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory#attach-a-memory-store-to-a-session) | 除非智能体必须写入，否则以 `read_only` 方式附加存储。默认值为 `read_write`，之后的会话会将已保存的文本作为受信任的记忆读取                                                                                                                    |

在云环境中，使用 `read` 读取文件还可能将 `CLAUDE.md` 和 `AGENTS.md` 文件作为指令加载。它们从该文件所在目录及其父目录加载，直到沙箱的工作目录为止。它们通过 `@` 加路径引用的文件（例如 `@docs/setup.md`）也会被加载，最多可嵌套五层引用。引用可以指向沙箱中的任何位置，例如 `@/etc/hosts`。除了禁用 `read` 之外，没有任何设置可以关闭此行为，因此请将您团队之外的人可以写入的仓库、软件包或归档文件视为不受信任的输入。

当智能体必须读取不受信任的输入并同时执行操作时，本指南推荐以下三种设计之一：

* **将读取与执行分离。** 在一个多智能体会话中，由一个[主智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#list-the-subagents)将任务分配给负责读取网页的获取者，以及负责更改文件或记录的写入者。
* **为智能体提供一个范围窄的自定义工具。** 由您的应用程序运行该工具、检查其输入并持有凭据。客服智能体使用的是这种设计，而不是分离设计。请参阅[回答使会话暂停的工具调用](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#answer-tool-calls-that-pause-the-session)。
* **让人工参与其中。** 在强大的工具上使用 `always_ask`，以便由人工审批每个调用。

分离设计使用三个智能体：

| 智能体             | 读取                | 持有                                                                                                                                                | 传递                  |
| --------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| 主智能体（会话所运行的智能体） | 您的消息，以及其他智能体返回的内容 | 无内置工具。其 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中指定了获取者和写入者 | 给每个智能体的任务           |
| 获取者             | 网页和搜索结果           | 仅 `web_fetch` 和 `web_search`，每个都带有 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`)                                  | 简短的结果，格式由您在其系统提示中指定 |
| 写入者             | 来自主智能体的任务         | 写入文件、运行命令或更改记录的工具，设置为 `auto` 或 `always_ask`。没有 Web 工具                                                                                             | 其结果                 |

获取者的结果是基于不受信任的页面构建的，主智能体可能会将其文本传递给写入者。请在写入者的工具上保持 `auto` 或 `always_ask`。获取者仍然可以将其读取的部分内容发送到其 `web_fetch` 列表中的站点，并通过查询发送给搜索提供商。

同一多智能体会话中的智能体[共享沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#how-it-works)、其文件以及保管库凭据，因此这种分离只隔离了它们的工具和对话。要实现更强的隔离，请使用两个具有独立环境和保管库的会话。让您的应用程序在两个会话之间仅传递固定字段，例如订单 ID。

## 设置限制和停止方式

在创建每个会话时设置预算，并提前决定如何停止智能体。

| 控制措施                                                                                                                  | 作用                                                                                                                                                                                                                                                       |
| --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 会话预算                                                                                                                  | 按公开标价计算的支出上限。达到上限时，会话及每个[线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)都会暂停。正在进行的请求可以完成，其工具调用仍会运行，因此支出可能超过上限。请参阅[当会话达到其预算时](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#when-a-session-reaches-its-budget) |
| 部署预算                                                                                                                  | 相同的上限，应用于部署启动的每个会话                                                                                                                                                                                                                                       |
| [`user.interrupt` 事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#interrupt-the-agent) | 立即停止模型响应。正在运行的工具调用可以先完成。会话进入 `idle` 状态（除非某个工作流运行使其保持 `running`），并在您发送下一条消息时恢复。中断不会结束任何工作流运行                                                                                                                                                              |
| 归档或删除会话                                                                                                               | 归档会保留历史记录并阻止新事件。删除是永久性的，会删除记录、事件和沙箱                                                                                                                                                                                                                      |
| 暂停或归档部署                                                                                                               | 暂停会停止新的计划运行，但仍允许手动运行。归档会永久结束计划。已在运行的会话会继续运行                                                                                                                                                                                                              |
| 归档或删除保管库或凭据                                                                                                           | 无需重启即可从正在运行的会话中移除密钥。请预留一定的延迟时间。会话会继续运行，因此也请中断它们。智能体已使用该密钥获取的令牌会保留在沙箱中                                                                                                                                                                                    |

要停止正在运行的智能体，请按照以下步骤操作：

1. 单独发送 `user.interrupt`。
2. 对于计划运行的智能体，还要暂停部署，以免启动新的计划运行。要永久结束计划，请归档该部署。
3. 如果某个密钥可能已泄露，请在签发该密钥的服务处轮换它。
4. 如果智能体有打开的工作流运行，还要发送一条 `user.message` 要求其停止这些运行，因为中断不会结束任何运行。有关需要先取消的工具调用，请参阅[在有打开的运行时中断会话](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#interrupt-a-session-with-runs-open)。
5. 等待会话进入 `idle` 状态。正在运行的会话无法被归档或删除。
6. 归档会话以保留其历史记录。当您不再需要历史记录时，将其删除。

中断不会撤销智能体已经执行或发送的内容。归档智能体不会停止其正在运行的会话。

预算的 `max_list_cost.amount` (java, php: `maxListCost.amount`; csharp, go: `MaxListCost.Amount`) 是以字符串形式书写的整数美分数，因此 `"125"` 表示 1.25 美元。请参阅[在创建会话时设置预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#set-a-budget-at-session-creation)。

您无法为创建时未设置预算的会话添加预算。要更改或移除上限，请参阅[恢复达到预算的会话](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#resume-a-session-at-its-budget)。移除会话的上限后，您将无法再设置新的上限。

预算会限制支出，但不会限制智能体在达到上限之前执行的操作。要将不同信任级别的工作负载隔离开来，请使用不同的[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)。将每个工作负载限制在一个工作区内：在可行的情况下，使用仅在该工作区中启用的[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)规则，否则使用限定于该工作区的 API 密钥。对多个工作区的访问权限会让工作负载能够在其中任何一个工作区中执行操作。

请参阅[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)、[会话操作](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#archiving-a-session)和[计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments#managing-deployment-lifecycle)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="云环境设置" icon="cloud" href="https://platform.claude.com/docs/zh-CN/managed-agents/environments">
    网络模式、允许的主机和包管理器。
  </Card>

  <Card title="权限策略" icon="check" href="https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies">
    审批策略和确认请求。
  </Card>

  <Card title="使用 vault 进行身份验证" icon="lock" href="https://platform.claude.com/docs/zh-CN/managed-agents/vaults">
    凭据类型、注入位置和轮换。
  </Card>

  <Card title="会话预算" icon="coins" href="https://platform.claude.com/docs/zh-CN/managed-agents/budgets">
    上限、在达到预算时恢复，以及部署上的预算。
  </Card>
</CardGroup>
