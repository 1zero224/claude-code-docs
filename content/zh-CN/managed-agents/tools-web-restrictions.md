---
title: 限制网页搜索和网页抓取的域名
url: https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions
description: 控制智能体的网页搜索和网页抓取工具可以访问哪些站点，限制获取的内容量，并对搜索结果进行本地化。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  betaHeader: managed-agents-2026-04-01
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
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Anthropic managed): beta
---

**API 支持**：用法因产品而异。

* **Messages API**：在请求中的 `web_search` 或 `web_fetch` 工具上设置 `allowed_domains` 或 `blocked_domains`。
* **Managed Agents**：在智能体工具集上设置相同的字段；每个列表最多包含 64 个域名。

要控制智能体的 Web 工具可以访问哪些站点，请在[智能体工具集](https://platform.claude.com/docs/zh-CN/managed-agents/tools#configuring-the-toolset)的 `web_search` 和 `web_fetch` 条目上设置域名列表。每个 `configs` (csharp, go: `Configs`) 条目接受以下两种列表之一：

* **`allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`)：** 工具只能访问这些主机。
* **`blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`)：** 工具永远无法访问这些主机。

每个工具都有自己的列表，因此 `web_search` 和 `web_fetch` 可以有不同的限制。

<Note>
  `web_search` 和 `web_fetch` 在 Anthropic 的服务器上运行，而不是在沙箱中运行。使用 `limited` [网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)的云环境也会将其 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 应用于这两个工具；请参阅[在智能体上设置域名列表](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#set-domain-lists-on-an-agent)。`unrestricted` 网络和自托管环境不会限制它们。每个工具的列表会进一步限制这些工具。
</Note>

您组织在 Claude Console 的 [**功能**](https://platform.claude.com/settings/capabilities) 页面上的网页搜索设置也适用于会话中的 `web_search`。如果管理员禁用了网页搜索，`web_search` 调用会向智能体返回错误结果，并且会话会继续进行。如果组织限制了网页搜索域名，智能体只会收到组织的域名限制和智能体的每个工具的列表都允许的搜索结果。

您组织在同一页面上的网页抓取设置也适用于会话中的 `web_fetch`。如果管理员关闭了网页抓取，`web_fetch` 调用会向智能体返回错误结果，并且会话会继续进行。如果组织为网页抓取设置了允许或屏蔽的域名，URL 必须同时通过组织的列表和智能体的每个工具的列表，重定向所指向的任何 URL 也必须如此。对其中任何一个列表所排除的 URL 发起的 `web_fetch` 调用同样会向智能体返回错误结果。

## 在智能体上设置域名列表

以下示例创建一个智能体，将 `web_search` 限制为两个站点，并为 `web_fetch` 屏蔽一个主机。它还设置了 `user_location` (java, php: `userLocation`; csharp, go: `UserLocation`) 和 `max_content_tokens` (java, php: `maxContentTokens`; csharp, go: `MaxContentTokens`)，[设置](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#settings)一节对此进行了说明。响应会回显 `configs` (csharp, go: `Configs`) 数组。

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Research Agent",
      "model": "claude-opus-5-5",
      "tools": [
        {
          "type": "agent_toolset_20260401",
          "configs": [
            {
              "type": "web_search",
              "name": "web_search",
              "allowed_domains": ["docs.example.com", "arxiv.org"],
              "user_location": {
                "type": "approximate",
                "country": "US",
                "timezone": "America/Los_Angeles"
              }
            },
            {
              "type": "web_fetch",
              "name": "web_fetch",
              "blocked_domains": ["ads.example.com"],
              "max_content_tokens": 50000
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
      name: Research Agent
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
          configs:
            - type: web_search
              name: web_search
              allowed_domains: [docs.example.com, arxiv.org]
              user_location:
                type: approximate
                country: US
                timezone: America/Los_Angeles
            - type: web_fetch
              name: web_fetch
              blocked_domains: [ads.example.com]
              max_content_tokens: 50000
      ---
      ```
    </File>

    [`ant apply`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/apply) 会创建智能体并打印其 ID，而不是 `configs` 数组。
  </CodeGroupItem>

  ```python Python
  client = Anthropic()

  agent = client.beta.agents.create(
      name="Research Agent",
      model="claude-opus-5-5",
      tools=[
          {
              "type": "agent_toolset_20260401",
              "configs": [
                  {
                      "name": "web_search",
                      "allowed_domains": ["docs.example.com", "arxiv.org"],
                      "user_location": {
                          "type": "approximate",
                          "country": "US",
                          "timezone": "America/Los_Angeles",
                      },
                  },
                  {
                      "name": "web_fetch",
                      "blocked_domains": ["ads.example.com"],
                      "max_content_tokens": 50_000,
                  },
              ],
          }
      ],
  )

  for tool in agent.tools:
      if tool.type == "agent_toolset_20260401":
          print(json.dumps([config.to_dict() for config in tool.configs], indent=2))
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const agent = await client.beta.agents.create({
    name: "Research Agent",
    model: "claude-opus-5-5",
    tools: [
      {
        type: "agent_toolset_20260401",
        configs: [
          {
            name: "web_search",
            allowed_domains: ["docs.example.com", "arxiv.org"],
            user_location: {
              type: "approximate",
              country: "US",
              timezone: "America/Los_Angeles"
            }
          },
          {
            name: "web_fetch",
            blocked_domains: ["ads.example.com"],
            max_content_tokens: 50_000
          }
        ]
      }
    ]
  });

  for (const tool of agent.tools) {
    if (tool.type === "agent_toolset_20260401") {
      console.log(JSON.stringify(tool.configs, null, 2));
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Agents;

  AnthropicClient client = new();

  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Research Agent",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      Tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params
          {
              Configs =
              [
                  new BetaManagedAgentsWebSearchToolConfigParams
                  {
                      AllowedDomains = ["docs.example.com", "arxiv.org"],
                      UserLocation = new()
                      {
                          Country = "US",
                          Timezone = "America/Los_Angeles",
                      },
                  },
                  new BetaManagedAgentsWebFetchToolConfigParams
                  {
                      BlockedDomains = ["ads.example.com"],
                      MaxContentTokens = 50_000,
                  },
              ],
          },
      ],
  });

  JsonSerializerOptions jsonOptions = new() { WriteIndented = true };
  foreach (var tool in agent.Tools)
  {
      if (tool.TryPickBetaManagedAgentsAgentToolset20260401(out var toolset))
      {
          Console.WriteLine(JsonSerializer.Serialize(toolset.Configs, jsonOptions));
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()
  ctx := context.Background()

  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Research Agent",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
  			Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{
  				{OfWebSearch: &anthropic.BetaManagedAgentsWebSearchToolConfigParams{
  					AllowedDomains: []string{"docs.example.com", "arxiv.org"},
  					UserLocation: anthropic.BetaManagedAgentsUserLocationParam{
  						Country:  anthropic.String("US"),
  						Timezone: anthropic.String("America/Los_Angeles"),
  					},
  				}},
  				{OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{
  					BlockedDomains:   []string{"ads.example.com"},
  					MaxContentTokens: anthropic.Int(50000),
  				}},
  			},
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }

  for _, tool := range agent.Tools {
  	switch toolset := tool.AsAny().(type) {
  	case anthropic.BetaManagedAgentsAgentToolset20260401:
  		configs := make([]json.RawMessage, len(toolset.Configs))
  		for i, config := range toolset.Configs {
  			configs[i] = json.RawMessage(config.RawJSON())
  		}
  		output, err := json.MarshalIndent(configs, "", "  ")
  		if err != nil {
  			panic(err)
  		}
  		fmt.Println(string(output))
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.beta.agents.AgentCreateParams;
  import com.anthropic.models.beta.agents.BetaManagedAgentsAgentToolset20260401Params;
  import com.anthropic.models.beta.agents.BetaManagedAgentsModel;
  import com.anthropic.models.beta.agents.BetaManagedAgentsUserLocation;
  import com.anthropic.models.beta.agents.BetaManagedAgentsWebFetchToolConfigParams;
  import com.anthropic.models.beta.agents.BetaManagedAgentsWebSearchToolConfigParams;

  void main() {
      var client = AnthropicOkHttpClient.fromEnv();

      var agent = client.beta().agents().create(AgentCreateParams.builder()
          .name("Research Agent")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
              .addConfig(BetaManagedAgentsWebSearchToolConfigParams.builder()
                  .allowedDomains(List.of("docs.example.com", "arxiv.org"))
                  .userLocation(BetaManagedAgentsUserLocation.builder()
                      .country("US")
                      .timezone("America/Los_Angeles")
                      .build())
                  .build())
              .addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()
                  .blockedDomains(List.of("ads.example.com"))
                  .maxContentTokens(50_000)
                  .build())
              .build())
          .build());

      for (var tool : agent.tools()) {
          if (tool.isAgentToolset20260401()) {
              var configs = tool.asAgentToolset20260401().configs();
              IO.println(ObjectMappers.jsonMapper().valueToTree(configs));
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401;
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsUserLocation;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebFetchToolConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebSearchToolConfigParams;
  // ...

  $client = new Client();

  $agent = $client->beta->agents->create(
      name: 'Research Agent',
      model: 'claude-opus-5-5',
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(
              configs: [
                  BetaManagedAgentsWebSearchToolConfigParams::with(
                      allowedDomains: ['docs.example.com', 'arxiv.org'],
                      userLocation: BetaManagedAgentsUserLocation::with(
                          country: 'US',
                          timezone: 'America/Los_Angeles',
                      ),
                  ),
                  BetaManagedAgentsWebFetchToolConfigParams::with(
                      blockedDomains: ['ads.example.com'],
                      maxContentTokens: 50_000,
                  ),
              ],
          ),
      ],
  );

  foreach ($agent->tools as $tool) {
      if ($tool instanceof BetaManagedAgentsAgentToolset20260401) {
          echo json_encode($tool->configs, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES), PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  agent = client.beta.agents.create(
    name: "Research Agent",
    model: "claude-opus-5-5",
    tools: [
      {
        type: :agent_toolset_20260401,
        configs: [
          {
            name: :web_search,
            allowed_domains: ["docs.example.com", "arxiv.org"],
            user_location: {type: :approximate, country: "US", timezone: "America/Los_Angeles"}
          },
          {
            name: :web_fetch,
            blocked_domains: ["ads.example.com"],
            max_content_tokens: 50_000
          }
        ]
      }
    ]
  )

  case agent.tools.first
  in Anthropic::Models::Beta::BetaManagedAgentsAgentToolset20260401 => toolset
    puts JSON.pretty_generate(toolset.configs.map(&:to_h))
  end
  ```
</CodeGroup>

在使用 `limited` [网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)的云环境中，环境的 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 也适用于 `web_search` 和 `web_fetch`。当已启用的 Web 工具的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 中有不在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 范围内的条目时，创建会话会失败并返回 400 错误。添加此类条目的会话更新也会失败。要解决此问题，请将该主机添加到 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`)，或从 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 中删除该条目。在运行时，如果 `web_fetch` 调用的 URL 所在主机与 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 不匹配，则会返回 `url_not_allowed` 错误结果。`web_search` 会省略来自此类主机的结果。这两个列表的匹配方式不同：工具的条目涵盖其子域名，但 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 条目只匹配一个确切的主机，除非它以 `*.` 开头。例如，工具条目 `docs.example.com` 不在 `["example.com"]` 的 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 范围内，但在 `["docs.example.com"]` 或 `["*.example.com"]` 范围内。

在 Claude Console 中，可以在智能体表单的**内置工具**卡片中的 `web_search` 和 `web_fetch` 行设置允许或屏蔽的域名。在智能体配置的**原始内容**视图中设置 `max_content_tokens` 和 `user_location`。

## 设置

除了 `enabled` (csharp, go: `Enabled`) 和 `permission_policy` (java, php: `permissionPolicy`; csharp, go: `PermissionPolicy`) 之外，Web 工具条目还接受以下设置：

| 设置                                                                                   | 适用于                      | 描述                                                                                                                                                         |
| ------------------------------------------------------------------------------------ | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`)        | `web_search`、`web_fetch` | 该工具唯一可以访问的主机。请参阅[域名列表规则](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#domain-list-rules)。                                  |
| `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`)        | `web_search`、`web_fetch` | 该工具无法访问的主机。请参阅[域名列表规则](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#domain-list-rules)。                                    |
| `max_content_tokens` (java, php: `maxContentTokens`; csharp, go: `MaxContentTokens`) | `web_fetch`              | 限制上下文中包含的已获取页面内容的数量。必须为正整数。请参阅[内容限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#content-limits)。                      |
| `user_location` (java, php: `userLocation`; csharp, go: `UserLocation`)              | `web_search`             | 对搜索结果进行本地化。一个对象，其字段与 Messages API 的 [`user_location`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#localization) 参数相同。 |

有关 SDK 如何为这些条目定义类型，请参阅 [SDK 中的配置条目类型](https://platform.claude.com/docs/zh-CN/managed-agents/tools#config-entry-types-in-the-sdks)。

## 当域名不被允许时

`web_search` 会省略其域名列表不允许的结果。对其域名列表不允许的 URL 发起的 `web_fetch` 调用会向智能体返回错误结果。`agent.tool_result` 事件带有 `is_error: true`，其内容会注明错误代码 `url_not_allowed`。

## 域名列表规则

这些规则同样适用于 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 和 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`)。违反任一规则的请求都会被拒绝，如[验证错误](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#validation-errors)所述。

* **每个条目最多一个列表：** 在一个条目上只能设置 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 或 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`) 之一，不能同时设置两者。
* **列表大小：** 每个列表包含 1 到 64 个域名，每个域名长度为 1 到 255 个字符。
* **不允许空列表：** 若不施加任何限制，请省略该字段或发送 `null`。
* **不允许重复：** 一个域名在列表中只能出现一次。`www.example.com` 和 `example.com` 被视为不同的域名。

### 列出的域名匹配什么

列出的域名会匹配该主机及其所有子域名。`example.com` 涵盖 `docs.example.com`，但 `docs.example.com` 不涵盖 `example.com` 或 `api.example.com`。

开头的 `www.` 与其他子域名一样，因此 `www.example.com` 不涵盖 `example.com`。列出裸域名即可同时涵盖两者。

主机名比较时不区分大小写。

### 域名格式

每个域名都是一个可注册域名或其子域名，以纯主机名形式书写。它可以包含 ASCII 字母、数字、连字符、下划线和点。末尾的单个 `/` 会被忽略。`web_search` 域名还可以带有路径，如[网页搜索域名上的路径后缀](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#path-suffixes-on-web-search-domains)中所述。

| 不接受                                     | 示例                     | 改用                      |
| --------------------------------------- | ---------------------- | ----------------------- |
| 协议方案                                    | `https://example.com`  | `example.com`           |
| 端口                                      | `example.com:443`      | `example.com`           |
| 通配符                                     | `*.example.com`        | `example.com`           |
| `web_fetch` 域名上的路径                      | `example.com/*`        | `example.com`           |
| 任何形式的 IP 地址，无论是 IPv4、IPv6、带方括号的还是数字简写形式 | `127.1`                | 该站点的域名                  |
| 裸顶级域名或注册后缀                              | `com`、`co.uk`、`gov.uk` | 完整域名，例如 `example.co.uk` |
| 单标签名称                                   | `intranet`             | 完整域名，例如 `example.co.uk` |
| 非 ASCII 字符，例如国际化域名中的字符                  |                        | `xn--`（Punycode）形式      |

如果域名包含凭据或空白字符，或者其某个标签以连字符开头或结尾，也会被拒绝。`localhost` 以及以 `.localhost`、`.local`、`.internal`、`.localdomain` 或 `.invalid` 结尾的主机同样会被拒绝。

### 网页搜索域名上的路径后缀

`web_search` 域名可以带有路径后缀，例如 `example.com/blog`。路径不能包含空格、`?`、`#` 或 `$ , | ^ !` 中的任何字符。

对于 `web_search`，也建议使用纯主机名。搜索提供商会将路径后缀作为 URL 模式进行匹配，而不是作为严格的主机规则。

## 验证错误

当您[创建智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#create-an-agent)或[更新智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-an-agent)时，API 会验证这些设置。当您创建或更新提供 `tools` (csharp, go: `Tools`) 的会话时，API 也会验证它们。

格式和限制违规会被拒绝，并返回 400 `invalid_request_error`：

| 违规             | 错误消息                                                                                                                    |
| -------------- | ----------------------------------------------------------------------------------------------------------------------- |
| 一个条目同时设置了两个列表。 | 包含 `Only one of allowed_domains or blocked_domains may be set.`                                                         |
| 列表为空。          | 包含 `allowed_domains: Empty list of domains is ambiguous. Provide at least one domain or null.`                          |
| 域名违反格式规则。      | 注明该域名所在的列表及其从零开始的位置。例如，`allowed_domains.0: IP addresses are not supported; provide a plain hostname like "example.com"` |

在相同的请求中，API 还会拒绝三种依赖于搜索和获取提供商的设置：

* `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 中 Anthropic 爬虫不被允许访问的域名。
* 搜索提供商不支持的 `user_location.country`。消息以 `user_location.country: not a country the search provider supports` 结尾。
* 不是有效 IANA 名称的 `user_location.timezone`。

在使用 `limited` [网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)的云环境中，会话创建和更新还会根据环境的 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 检查 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`)。请参阅[在智能体上设置域名列表](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#set-domain-lists-on-an-agent)中的规则。

### 当已接受的设置不再有效时

会话在首次初始化工具时会再次检查配置。如果之前已接受的设置此时不再有效，会话会发出 [`session.error`](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming) 事件，然后返回 `idle` 状态且不会重试。

要继续该会话：

1. 通过[更新会话的工具](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#updating-the-agent-configuration)修复该设置。
2. 同时更新智能体，以便新会话使用修正后的配置启动。
3. 发送新的 `user.message`。

## 多智能体会话和面向结果的会话

在[多智能体会话](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)中，适用于某个线程的所有域名列表会同时生效。列在 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中的智能体受三组每个工具的列表约束：

* 其自身的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 和 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`)
* 调用它的任何智能体的列表
* 会话所运行的智能体的当前列表

这些设置的组合方式如下：

| 设置                                   | 组合方式                                                       |
| ------------------------------------ | ---------------------------------------------------------- |
| `allowed_domains`                    | 只有当每个列表都涵盖某个主机时，该工具才能访问该主机。                                |
| `blocked_domains`                    | 各列表相加合并。                                                   |
| `max_content_tokens`、`user_location` | 不组合。如果线程自身的工具配置中设置了该值，则使用该值。否则使用调用它的智能体的值，再否则使用会话智能体的当前配置。 |

因此，列出的智能体可以缩小工具的访问范围，但永远无法扩大它：

* 设置了 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`) 的所列智能体会保留会话智能体的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`)，并在其中屏蔽这些主机。
* 设置了自己的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 的所列智能体只能访问其列表和会话智能体的列表都涵盖的主机。

`subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中的 `{"type": "self"}` 条目没有自己的 Web 设置，而是遵循会话智能体的当前设置。

如果组合后的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 列表没有共同的域名，该工具对该智能体仍然可用，但每次调用都会失败。每次调用都会返回 `url_not_allowed` 错误，其文本中包含 `no domain is permitted`。工具描述也会告知模型这一点。为避免这种情况，请确保每个所列智能体的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 都在会话智能体的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 范围内。

[面向结果的会话](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)中的评分器在运行时不使用 `web_search` 和 `web_fetch`，与这些设置无关。

## 在会话中途更改列表

您可以通过[更新空闲会话的工具](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#updating-the-agent-configuration)来更改其列表。新列表将应用于会话的剩余部分。

在多智能体会话中，每个线程从其下一轮开始应用新列表。对于列在 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中的智能体，此更新不会更改该智能体自身的列表。这些列表保持创建会话时智能体定义所设置的内容。

## 与 Messages API 工具的区别

这些设置使用与 Messages API 服务器工具上的[域名过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#domain-filtering)相同的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 和 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`) 字段。Managed Agents 在四个方面有所不同：

* 每个列表[最多包含 64 个域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#domain-list-rules)。
* 为 `web_fetch` 列出的域名[不能包含路径](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#domain-format)。
* 域名[必须为 ASCII](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#domain-format)。Messages API 接受 Unicode 条目，但不建议使用。
* `max_uses` (java, php: `maxUses`; csharp, go: `MaxUses`)、`citations` (csharp, go: `Citations`) 和 `cache_control` (java, php: `cacheControl`; csharp, go: `CacheControl`) 在工具集上不可用。

## 后续步骤

<CardGroup cols={2}>
  <Card title="工具" icon="tool" href="https://platform.claude.com/docs/zh-CN/managed-agents/tools">
    查看内置工具，启用或禁用它们，并定义自定义工具。
  </Card>

  <Card title="权限策略" icon="lock" href="https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies">
    控制智能体工具和 MCP 工具何时执行。
  </Card>

  <Card title="云环境设置" icon="settings" href="https://platform.claude.com/docs/zh-CN/managed-agents/environments">
    控制沙箱自身的出站网络访问。
  </Card>

  <Card title="多智能体编排" icon="sitemap" href="https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration">
    在单个会话中协调多个智能体。
  </Card>
</CardGroup>

## 可用性

网页域名限制可用或处于 beta 阶段，具体取决于API。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：在请求中的 `web_search` 或 `web_fetch` 工具上设置 `allowed_domains` 或 `blocked_domains`。

| 状态 | 云平台        |
| -- | ---------- |
| 可用 | Claude API |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：在智能体工具集的 `web_search` 和 `web_fetch` 条目上设置相同的字段。与 Messages API 不同，每个列表最多包含 64 个 ASCII 域名，且 `web_fetch` 域名不能包含路径。

| 状态   | 云平台     | 模型                                                             |
| ---- | ------- | -------------------------------------------------------------- |
| Beta | 所有支持的平台 | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

| 请求头                         | API                                                                              |
| --------------------------- | -------------------------------------------------------------------------------- |
| `managed-agents-2026-04-01` | [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) |
