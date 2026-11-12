---
title: 开始使用 Claude Managed Agents
url: https://platform.claude.com/docs/zh-CN/managed-agents/quickstart
description: 创建您的第一个自主智能体。
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

本指南将引导您完成创建智能体、设置环境、启动会话以及流式传输智能体响应的过程。

让 Claude Code 为您设置智能体

```text
/claude-api managed-agents-onboard
```

## 核心概念

| 概念                  | 描述                                           |
| ------------------- | -------------------------------------------- |
| **Agent（智能体）**      | 模型、系统提示、工具、MCP 服务器和技能                        |
| **Environment（环境）** | 会话运行位置的配置：Anthropic 托管的云沙箱，或在您自己的基础设施上自托管的沙箱 |
| **Session（会话）**     | 在环境中运行的智能体实例，执行特定任务并生成输出                     |
| **Events（事件）**      | 您的应用程序与智能体之间交换的消息（用户轮次、工具结果、状态更新）            |

## 开始之前

<Steps continues>
  <Step title="创建 Claude Console 账户">
    [注册](https://platform.claude.com/) Claude Console 账户，或登录您已有的账户。
  </Step>

  <Step title="选择身份验证方式">
    对于本地开发，建议使用 CLI 登录。

    Follow the steps for one of the following options.
  </Step>
</Steps>

**CLI**

<Steps continued>
  <Step title="安装 CLI" stepNumber={3}>
    <Tabs noDeepLinks>
      <Tab title="Homebrew (macOS)">
        ```bash
        brew install anthropics/tap/ant
        ```
      </Tab>

      <Tab title="curl (Linux/WSL)">
        对于 Linux 环境，请直接下载发布版二进制文件。

        ```bash
        VERSION=2.9.0
        OS=$(uname -s | tr '[:upper:]' '[:lower:]')
        case $(uname -m) in
          x86_64) ARCH=amd64 ;;
          aarch64) ARCH=arm64 ;;
        esac
        curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_${OS}_${ARCH}.tar.gz" \
          | sudo tar -xz -C /usr/local/bin ant
        ```

        您可以在 [GitHub 发布页面](https://github.com/anthropics/anthropic-cli/releases)上找到所有发布版本。
      </Tab>

      <Tab title="go install">
        您也可以使用 `go install` 从源代码安装 CLI，这需要 Go 1.26.10 或更高版本。

        ```bash
        go install github.com/anthropics/anthropic-cli/v2/cmd/ant@latest
        ```

        `go install` 会将二进制文件放置在 `$(go env GOPATH)/bin` 中。如果该目录尚未添加到您的 `PATH` 中，请将其添加进去：

        ```bash
        export PATH="$PATH:$(go env GOPATH)/bin"
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="登录" stepNumber={4}>
    ```bash
    ant auth login
    ```

    登录完成后，SDK 和 CLI 将自动获取您的凭据。
  </Step>
</Steps>

**API 密钥**

<Steps continued>
  <Step title="创建 API 密钥" stepNumber={3}>
    在 Claude Console 中创建 [API 密钥](https://platform.claude.com/settings/keys)。
  </Step>

  <Step title="设置环境变量" stepNumber={4}>
    SDK 和 CLI 从 `ANTHROPIC_API_KEY` 读取您的密钥：

    ```bash
    export ANTHROPIC_API_KEY="your-api-key-here"
    ```
  </Step>
</Steps>

**API 密钥**

要运行 CLI 示例，请安装 CLI：

<Tabs noDeepLinks>
  <Tab title="Homebrew (macOS)">
    ```bash
    brew install anthropics/tap/ant
    ```
  </Tab>

  <Tab title="curl (Linux/WSL)">
    对于 Linux 环境，请直接下载发布版二进制文件。

    ```bash
    VERSION=2.9.0
    OS=$(uname -s | tr '[:upper:]' '[:lower:]')
    case $(uname -m) in
      x86_64) ARCH=amd64 ;;
      aarch64) ARCH=arm64 ;;
    esac
    curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_${OS}_${ARCH}.tar.gz" \
      | sudo tar -xz -C /usr/local/bin ant
    ```

    您可以在 [GitHub 发布页面](https://github.com/anthropics/anthropic-cli/releases)上找到所有发布版本。
  </Tab>

  <Tab title="go install">
    您也可以使用 `go install` 从源代码安装 CLI，这需要 Go 1.26.10 或更高版本。

    ```bash
    go install github.com/anthropics/anthropic-cli/v2/cmd/ant@latest
    ```

    `go install` 会将二进制文件放置在 `$(go env GOPATH)/bin` 中。如果该目录尚未添加到您的 `PATH` 中，请将其添加进去：

    ```bash
    export PATH="$PATH:$(go env GOPATH)/bin"
    ```
  </Tab>
</Tabs>

## 安装 SDK

<Tabs>
  <Tab title="Python">
    ```bash
    pip install anthropic
    ```
  </Tab>

  <Tab title="TypeScript">
    ```bash
    npm install @anthropic-ai/sdk
    ```
  </Tab>

  <Tab title="Java">
    <Tabs>
      <Tab title="Gradle">
        ```kotlin
        implementation("com.anthropic:anthropic-java:3.8.0")
        ```
      </Tab>

      <Tab title="Maven">
        ```xml
        <dependency>
            <groupId>com.anthropic</groupId>
            <artifactId>anthropic-java</artifactId>
            <version>3.8.0</version>
        </dependency>
        ```
      </Tab>
    </Tabs>
  </Tab>

  <Tab title="Go">
    ```bash
    go get github.com/anthropics/anthropic-sdk-go/v2
    ```
  </Tab>

  <Tab title="C#">
    ```bash
    dotnet add package Anthropic
    ```
  </Tab>

  <Tab title="Ruby">
    ```bash
    bundle add anthropic
    ```
  </Tab>

  <Tab title="PHP">
    ```bash
    composer require "anthropic-ai/sdk" "guzzlehttp/guzzle:^7"
    ```
  </Tab>
</Tabs>

## 创建您的第一个会话

<Steps>
  <Step title="创建智能体">
    创建一个智能体，用于定义模型、系统提示和可用工具。

    <CodeGroup defaultLanguage="CLI">
      **CLI 登录**

      <CodeGroupItem>
        ```bash cURL
        curl -sS --fail-with-body https://api.anthropic.com/v1/agents \
          -H "Authorization: Bearer $(ant auth print-credentials --access-token)" \
          -H "anthropic-version: 2026-10-21" \
          -H "anthropic-beta: managed-agents-2026-04-01" \
          -H "content-type: application/json" \
          -d '{
            "name": "Coding Assistant",
            "model": "claude-opus-5-5",
            "system": "You are a helpful coding assistant. Write clean, well-documented code.",
            "tools": [
              {
                "type": "agent_toolset_20260401",
                "configs": [
                  {"name": "web_search", "enabled": false},
                  {"name": "web_fetch", "enabled": false}
                ]
              }
            ]
          }'
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      **API 密钥**

      <CodeGroupItem>
        ```bash cURL
        curl -sS --fail-with-body https://api.anthropic.com/v1/agents \
          -H "x-api-key: $ANTHROPIC_API_KEY" \
          -H "anthropic-version: 2026-10-21" \
          -H "anthropic-beta: managed-agents-2026-04-01" \
          -H "content-type: application/json" \
          -d '{
            "name": "Coding Assistant",
            "model": "claude-opus-5-5",
            "system": "You are a helpful coding assistant. Write clean, well-documented code.",
            "tools": [
              {
                "type": "agent_toolset_20260401",
                "configs": [
                  {"name": "web_search", "enabled": false},
                  {"name": "web_fetch", "enabled": false}
                ]
              }
            ]
          }'
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```bash CLI
        ant apply coding-assistant.md
        ```

        <File filename="coding-assistant.md">
          ```markdown
          ---
          name: Coding Assistant
          model: claude-opus-5-5
          tools:
            - type: agent_toolset_20260401
              configs:
                - name: web_search
                  enabled: false
                - name: web_fetch
                  enabled: false
          ---

          You are a helpful coding assistant. Write clean, well-documented code.
          ```
        </File>

        [`ant apply`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/apply) 会打印智能体的 ID 并将其记录在 `claude-lock.json` 中。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```python Python
        from anthropic import Anthropic

        client = Anthropic()

        agent = client.beta.agents.create(
            name="Coding Assistant",
            model="claude-opus-5-5",
            system="You are a helpful coding assistant. Write clean, well-documented code.",
            tools=[
                {
                    "type": "agent_toolset_20260401",
                    "configs": [
                        {"name": "web_search", "enabled": False},
                        {"name": "web_fetch", "enabled": False},
                    ],
                },
            ],
        )

        print(f"Agent ID: {agent.id}, version: {agent.version}")
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```typescript TypeScript
        import Anthropic from "@anthropic-ai/sdk";

        const client = new Anthropic();

        const agent = await client.beta.agents.create({
          name: "Coding Assistant",
          model: "claude-opus-5-5",
          system: "You are a helpful coding assistant. Write clean, well-documented code.",
          tools: [
            {
              type: "agent_toolset_20260401",
              configs: [
                { name: "web_search", enabled: false },
                { name: "web_fetch", enabled: false }
              ]
            }
          ]
        });

        console.log(`Agent ID: ${agent.id}, version: ${agent.version}`);
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```csharp C#
        using Anthropic;
        using Anthropic.Models.Beta.Agents;
        using Anthropic.Models.Beta.Environments;
        using Anthropic.Models.Beta.Sessions;
        using Anthropic.Models.Beta.Sessions.Events;

        var client = new AnthropicClient();

        var agent = await client.Beta.Agents.Create(new()
        {
            Name = "Coding Assistant",
            Model = BetaManagedAgentsModel.ClaudeOpus5_5,
            System = "You are a helpful coding assistant. Write clean, well-documented code.",
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
            ],
        });

        Console.WriteLine($"Agent ID: {agent.ID}, version: {agent.Version}");
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```go Go
        package main

        import (
        	"context"
        	"fmt"

        	"github.com/anthropics/anthropic-sdk-go/v2"
        )

        func main() {
        	client := anthropic.NewClient()
        	ctx := context.Background()

        	agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
        		Name: "Coding Assistant",
        		Model: anthropic.BetaManagedAgentsModelConfigParams{
        			ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
        		},
        		System: anthropic.String("You are a helpful coding assistant. Write clean, well-documented code."),
        		Tools: []anthropic.BetaAgentNewParamsToolUnion{{
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
        		}},
        	})
        	if err != nil {
        		panic(err)
        	}

        	fmt.Printf("Agent ID: %s, version: %d\n", agent.ID, agent.Version)
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```java Java
        import com.anthropic.client.okhttp.AnthropicOkHttpClient;
        import com.anthropic.models.beta.agents.AgentCreateParams;
        import com.anthropic.models.beta.agents.BetaManagedAgentsAgentToolset20260401Params;
        import com.anthropic.models.beta.agents.BetaManagedAgentsModel;
        import com.anthropic.models.beta.agents.BetaManagedAgentsWebFetchToolConfigParams;
        import com.anthropic.models.beta.agents.BetaManagedAgentsWebSearchToolConfigParams;
        import com.anthropic.models.beta.environments.BetaCloudConfigParams;
        import com.anthropic.models.beta.environments.BetaLimitedNetworkParams;
        import com.anthropic.models.beta.environments.EnvironmentCreateParams;
        import com.anthropic.models.beta.sessions.SessionCreateParams;
        import com.anthropic.models.beta.sessions.events.BetaManagedAgentsUserMessageEventParams;
        import com.anthropic.models.beta.sessions.events.EventSendParams;

        void main() {
            var client = AnthropicOkHttpClient.fromEnv();

            var agent = client.beta().agents().create(AgentCreateParams.builder()
                .name("Coding Assistant")
                .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
                .system("You are a helpful coding assistant. Write clean, well-documented code.")
                .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
                    .addConfig(BetaManagedAgentsWebSearchToolConfigParams.builder()
                        .enabled(false)
                        .build())
                    .addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()
                        .enabled(false)
                        .build())
                    .build())
                .build());

            IO.println("Agent ID: " + agent.id() + ", version: " + agent.version());
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```php PHP
        use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
        use Anthropic\Beta\Agents\BetaManagedAgentsModel;
        use Anthropic\Beta\Agents\BetaManagedAgentsWebFetchToolConfigParams;
        use Anthropic\Beta\Agents\BetaManagedAgentsWebSearchToolConfigParams;
        use Anthropic\Client;

        $client = new Client();

        $agent = $client->beta->agents->create(
            name: 'Coding Assistant',
            model: BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
            system: 'You are a helpful coding assistant. Write clean, well-documented code.',
            tools: [
                BetaManagedAgentsAgentToolset20260401Params::with(
                    configs: [
                        BetaManagedAgentsWebSearchToolConfigParams::with(enabled: false),
                        BetaManagedAgentsWebFetchToolConfigParams::with(enabled: false),
                    ],
                ),
            ],
        );

        echo "Agent ID: {$agent->id}, version: {$agent->version}\n";
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>

      <CodeGroupItem>
        ```ruby Ruby
        require "anthropic"

        client = Anthropic::Client.new

        agent = client.beta.agents.create(
          name: "Coding Assistant",
          model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
          system_: "You are a helpful coding assistant. Write clean, well-documented code.",
          tools: [
            {
              type: "agent_toolset_20260401",
              configs: [
                {name: "web_search", enabled: false},
                {name: "web_fetch", enabled: false}
              ]
            }
          ]
        )

        puts "Agent ID: #{agent.id}, version: #{agent.version}"
        ```

        保存返回的 `agent.id`。您创建的每个会话都会引用它。
      </CodeGroupItem>
    </CodeGroup>

    `agent_toolset_20260401` 工具类型会启用预构建的智能体工具（bash、文件操作、网络搜索等）。此示例禁用了网络搜索和网页获取，因为本指南中的任务（编写一个简短的 Python 脚本）不会用到网页。请只为智能体提供[它所需的工具](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#give-the-agent-the-smallest-tool-set)。有关完整列表和各工具的配置选项，请参阅[工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools)。
  </Step>

  <Step title="创建环境">
    环境定义了智能体运行所在的沙箱。此环境使用 `limited` [网络](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)并允许包管理器，因此在沙箱中运行的代码可以访问公共[包注册表和代码托管平台](https://platform.claude.com/docs/zh-CN/managed-agents/environments#package-manager-hosts)。在 `limited` 网络下，[`allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 列表](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)同样适用于网络搜索和网页获取，尽管它们在沙箱外运行。如果未列出任何主机（此环境正是如此），这些工具不会返回任何页面或搜索结果。如果您自己的智能体使用这些工具，请将它所需的主机添加到该列表中。

    <CodeGroup defaultLanguage="CLI">
      **CLI 登录**

      <CodeGroupItem>
        ```bash cURL
        curl -sS --fail-with-body https://api.anthropic.com/v1/environments \
          -H "Authorization: Bearer $(ant auth print-credentials --access-token)" \
          -H "anthropic-version: 2026-10-21" \
          -H "anthropic-beta: managed-agents-2026-04-01" \
          -H "content-type: application/json" \
          -d '{
            "name": "quickstart-env",
            "config": {
              "type": "cloud",
              "networking": {"type": "limited", "allow_package_managers": true}
            }
          }'
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      **API 密钥**

      <CodeGroupItem>
        ```bash cURL
        curl -sS --fail-with-body https://api.anthropic.com/v1/environments \
          -H "x-api-key: $ANTHROPIC_API_KEY" \
          -H "anthropic-version: 2026-10-21" \
          -H "anthropic-beta: managed-agents-2026-04-01" \
          -H "content-type: application/json" \
          -d '{
            "name": "quickstart-env",
            "config": {
              "type": "cloud",
              "networking": {"type": "limited", "allow_package_managers": true}
            }
          }'
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```bash CLI
        ant apply environment.yaml
        ```

        <File filename="environment.yaml">
          ```yaml
          # yaml-language-server: $schema=https://platform.claude.com/schemas/ant/beta/environment.json
          name: quickstart-env
          config:
            type: cloud
            networking:
              type: limited
              allow_package_managers: true
          ```
        </File>

        [`ant apply`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/apply) 同样会将环境的 ID 记录在 `claude-lock.json` 中。要用一条命令同时创建智能体和环境，请传入两个文件：`ant apply coding-assistant.md environment.yaml`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```python Python
        environment = client.beta.environments.create(
            name="quickstart-env",
            config={
                "type": "cloud",
                "networking": {"type": "limited", "allow_package_managers": True},
            },
        )

        print(f"Environment ID: {environment.id}")
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```typescript TypeScript
        const environment = await client.beta.environments.create({
          name: "quickstart-env",
          config: {
            type: "cloud",
            networking: { type: "limited", allow_package_managers: true }
          }
        });

        console.log(`Environment ID: ${environment.id}`);
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```csharp C#
        var environment = await client.Beta.Environments.Create(new()
        {
            Name = "quickstart-env",
            Config = new BetaCloudConfigParams
            {
                Networking = new BetaLimitedNetworkParams { AllowPackageManagers = true },
            },
        });

        Console.WriteLine($"Environment ID: {environment.ID}");
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```go Go
        environment, err := client.Beta.Environments.New(ctx, anthropic.BetaEnvironmentNewParams{
        	Name: "quickstart-env",
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

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```java Java
        var environment = client.beta().environments().create(EnvironmentCreateParams.builder()
            .name("quickstart-env")
            .config(BetaCloudConfigParams.builder()
                .networking(BetaLimitedNetworkParams.builder()
                    .allowPackageManagers(true)
                    .build())
                .build())
            .build());

        IO.println("Environment ID: " + environment.id());
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```php PHP
        $environment = $client->beta->environments->create(
            name: 'quickstart-env',
            config: [
                'type' => 'cloud',
                'networking' => ['type' => 'limited', 'allow_package_managers' => true],
            ],
        );

        echo "Environment ID: {$environment->id}\n";
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>

      <CodeGroupItem>
        ```ruby Ruby
        environment = client.beta.environments.create(
          name: "quickstart-env",
          config: {type: "cloud", networking: {type: "limited", allow_package_managers: true}}
        )

        puts "Environment ID: #{environment.id}"
        ```

        同样保存返回的 `environment.id`。
      </CodeGroupItem>
    </CodeGroup>

    <Tip>
      本快速入门使用云环境。要决定是否改为通过

      [自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)

      在您自己的基础设施上运行工具，请参阅

      [选择智能体环境](https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment)

      。
    </Tip>
  </Step>

  <Step title="启动会话">
    创建一个引用您的智能体和环境的会话。

    <CodeGroup>
      **CLI 登录**

      ```bash cURL
      curl -sS --fail-with-body https://api.anthropic.com/v1/sessions \
        -H "Authorization: Bearer $(ant auth print-credentials --access-token)" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01" \
        -H "content-type: application/json" \
        -d '{
          "agent": "'"$AGENT_ID"'",
          "environment_id": "'"$ENVIRONMENT_ID"'",
          "title": "Quickstart session"
        }'
      ```

      **API 密钥**

      ```bash cURL
      curl -sS --fail-with-body https://api.anthropic.com/v1/sessions \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01" \
        -H "content-type: application/json" \
        -d '{
          "agent": "'"$AGENT_ID"'",
          "environment_id": "'"$ENVIRONMENT_ID"'",
          "title": "Quickstart session"
        }'
      ```

      ```bash CLI
      ant beta:sessions create \
        --agent "$AGENT_ID" \
        --environment-id "$ENVIRONMENT_ID" \
        --title "Quickstart session"
      ```

      ```python Python
      session = client.beta.sessions.create(
          agent=agent.id,
          environment_id=environment.id,
          title="Quickstart session",
      )

      print(f"Session ID: {session.id}")
      ```

      ```typescript TypeScript
      const session = await client.beta.sessions.create({
        agent: agent.id,
        environment_id: environment.id,
        title: "Quickstart session"
      });

      console.log(`Session ID: ${session.id}`);
      ```

      ```csharp C#
      var session = await client.Beta.Sessions.Create(new()
      {
          Agent = agent.ID,
          EnvironmentID = environment.ID,
          Title = "Quickstart session",
      });

      Console.WriteLine($"Session ID: {session.ID}");
      ```

      ```go Go
      session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
      	Agent:         anthropic.BetaSessionNewParamsAgentUnion{OfString: anthropic.String(agent.ID)},
      	EnvironmentID: environment.ID,
      	Title:         anthropic.String("Quickstart session"),
      })
      if err != nil {
      	panic(err)
      }

      fmt.Printf("Session ID: %s\n", session.ID)
      ```

      ```java Java
      var session = client.beta().sessions().create(SessionCreateParams.builder()
          .agent(agent.id())
          .environmentId(environment.id())
          .title("Quickstart session")
          .build());

      IO.println("Session ID: " + session.id());
      ```

      ```php PHP
      $session = $client->beta->sessions->create(
          agent: $agent->id,
          environmentID: $environment->id,
          title: 'Quickstart session',
      );

      echo "Session ID: {$session->id}\n";
      ```

      ```ruby Ruby
      session = client.beta.sessions.create(
        agent: agent.id,
        environment_id: environment.id,
        title: "Quickstart session"
      )

      puts "Session ID: #{session.id}"
      ```
    </CodeGroup>
  </Step>

  <Step title="发送消息并流式传输响应">
    打开一个流，发送用户事件，然后在事件到达时进行处理：

    <CodeGroup exclude="shell">
      <CodeGroupItem language="cURL">
        此工作流需要的逻辑超出了单条 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 来运行它。
      </CodeGroupItem>

      <CodeGroupItem language="CLI">
        此工作流需要的逻辑超出了单条 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 来运行它。
      </CodeGroupItem>

      ```python Python
      with client.beta.sessions.events.stream(session.id) as stream:
          # 在流打开后发送用户消息
          client.beta.sessions.events.send(
              session.id,
              events=[
                  {
                      "type": "user.message",
                      "content": [
                          {
                              "type": "text",
                              "text": "Create a Python script that generates the first 20 Fibonacci numbers and saves them to fibonacci.txt",
                          },
                      ],
                  },
              ],
          )

          # 处理流式传输事件
          for event in stream:
              match event.type:
                  case "agent.message":
                      for block in event.content:
                          if block.type == "text":
                              print(block.text, end="")
                  case "agent.tool_use":
                      print(f"\n[Using tool: {event.name}]")
                  case "session.status_idle":
                      print("\n\nAgent finished.")
                      break
      ```

      ```typescript TypeScript
      const stream = await client.beta.sessions.events.stream(session.id);

      // 在流打开后发送用户消息
      await client.beta.sessions.events.send(session.id, {
        events: [
          {
            type: "user.message",
            content: [
              {
                type: "text",
                text: "Create a Python script that generates the first 20 Fibonacci numbers and saves them to fibonacci.txt"
              }
            ]
          }
        ]
      });

      // 处理流式传输事件
      loop: for await (const event of stream) {
        switch (event.type) {
          case "agent.message":
            for (const block of event.content) {
              if (block.type === "text") {
                process.stdout.write(block.text);
              }
            }
            break;
          case "agent.tool_use":
            console.log(`\n[Using tool: ${event.name}]`);
            break;
          case "session.status_idle":
            console.log("\n\nAgent finished.");
            break loop;
        }
      }
      ```

      ```csharp C#
      var stream = client.Beta.Sessions.Events.StreamStreaming(session.ID);

      // 在流打开后发送用户消息
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
                          Text = "Create a Python script that generates the first 20 Fibonacci numbers and saves them to fibonacci.txt",
                      },
                  ],
              },
          ],
      });

      // 处理流式传输事件
      await foreach (var streamEvent in stream)
      {
          if (streamEvent.Value is BetaManagedAgentsAgentMessageEvent message)
          {
              foreach (var block in message.Content)
              {
                  if (block.Value is BetaManagedAgentsTextBlock textBlock)
                  {
                      Console.Write(textBlock.Text);
                  }
              }
          }
          else if (streamEvent.Value is BetaManagedAgentsAgentToolUseEvent toolUse)
          {
              Console.WriteLine($"\n[Using tool: {toolUse.Name}]");
          }
          else if (streamEvent.Value is BetaManagedAgentsSessionStatusIdleEvent)
          {
              Console.WriteLine("\n\nAgent finished.");
              break;
          }
      }
      ```

      ```go Go
      	stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{})
      	defer stream.Close()

      	// 在流打开后发送用户消息
      	_, err = client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
      		Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
      			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
      				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
      					OfText: &anthropic.BetaManagedAgentsTextBlockParam{
      						Text: "Create a Python script that generates the first 20 Fibonacci numbers and saves them to fibonacci.txt",
      					},
      				}},
      			},
      		}},
      	})
      	if err != nil {
      		panic(err)
      	}

      	// 处理流式传输事件
      loop:
      	for stream.Next() {
      		switch event := stream.Current().AsAny().(type) {
      		case anthropic.BetaManagedAgentsAgentMessageEvent:
      			for _, block := range event.Content {
      				if block.Type == "text" {
      					fmt.Print(block.Text)
      				}
      			}
      		case anthropic.BetaManagedAgentsAgentToolUseEvent:
      			fmt.Printf("\n[Using tool: %s]\n", event.Name)
      		case anthropic.BetaManagedAgentsSessionStatusIdleEvent:
      			fmt.Print("\n\nAgent finished.\n")
      			break loop
      		}
      	}
      	if err := stream.Err(); err != nil {
      		panic(err)
      	}
      ```

      ```java Java
      try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
          // 在流打开后发送用户消息
          client.beta().sessions().events().send(session.id(), EventSendParams.builder()
              .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
                  .addTextContent("Create a Python script that generates the first 20 Fibonacci numbers and saves them to fibonacci.txt")
                  .build())
              .build());

          // 处理流式传输事件
          loop:
          for (var event : stream) {
              switch (event.type().value()) {
                  case AGENT_MESSAGE -> event.asAgentMessage().content().forEach(block -> block.text().ifPresent(textBlock -> IO.print(textBlock.text())));
                  case AGENT_TOOL_USE -> IO.println("\n[Using tool: " + event.asAgentToolUse().name() + "]");
                  case SESSION_STATUS_IDLE -> {
                      IO.println("\n\nAgent finished.");
                      break loop;
                  }
              }
          }
      }
      ```

      ```php PHP
      $stream = $client->beta->sessions->events->streamStream($session->id);

      // 在流打开后发送用户消息
      $client->beta->sessions->events->send(
          $session->id,
          events: [
              [
                  'type' => 'user.message',
                  'content' => [
                      ['type' => 'text', 'text' => 'Create a Python script that generates the first 20 Fibonacci numbers and saves them to fibonacci.txt'],
                  ],
              ],
          ],
      );

      // 处理流式传输事件
      foreach ($stream as $event) {
          match (true) {
              $event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent => array_walk(
                  $event->content,
                  static fn ($block) => $block instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock ? print($block->text) : null,
              ),
              $event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsAgentToolUseEvent => print("\n[Using tool: {$event->name}]\n"),
              $event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsSessionStatusIdleEvent => print("\n\nAgent finished.\n"),
              default => null,
          };
          if ($event->type === 'session.status_idle') {
              break;
          }
      }
      ```

      ```ruby Ruby
      stream = client.beta.sessions.events.stream_events(session.id)

      # 在流打开后发送用户消息
      client.beta.sessions.events.send_(
        session.id,
        events: [{
          type: "user.message",
          content: [{type: "text", text: "Create a Python script that generates the first 20 Fibonacci numbers and saves them to fibonacci.txt"}]
        }]
      )

      # 处理流式传输事件
      stream.each do |event|
        case event
        when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
          event.content.each { print it.text if it.is_a?(Anthropic::Beta::Sessions::BetaManagedAgentsTextBlock) }
        when Anthropic::Beta::Sessions::BetaManagedAgentsAgentToolUseEvent
          puts "\n[Using tool: #{event.name}]"
        when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusIdleEvent
          puts "\n\nAgent finished."
          break
        else
          # 忽略其他事件类型
        end
      end
      ```
    </CodeGroup>

    智能体会编写一个 Python 脚本，在沙箱中运行它，并验证输出文件已创建。您的输出类似如下：

    ```text wrap
    I'll create a Python script that generates the first 20 Fibonacci numbers and saves them to a file.
    [Using tool: write]
    [Using tool: bash]
    The script ran successfully. Let me verify the output file.
    [Using tool: bash]
    fibonacci.txt contains the first 20 Fibonacci numbers (0 through 4181).

    Agent finished.
    ```
  </Step>
</Steps>

## 运行过程

当您发送用户事件时，Claude Managed Agents 会：

1. **配置沙箱：**&#x60A8;的环境配置决定沙箱的构建方式。
2. **运行智能体循环：**&#x43;laude 根据您的消息决定使用哪些工具。
3. **运行工具：**&#x6587;件写入、bash 命令和其他工具调用都在沙箱内运行。
4. **流式传输事件：**&#x5728;智能体工作时，您会收到实时更新。
5. **进入空闲状态：**&#x5F53;智能体没有更多工作要做时，会发出 `session.status_idle` 事件。

## 构建完整应用

以下每个快速入门都将 Claude Managed Agents 与一个流行的聊天框架结合，构建出完整、可运行的应用程序。在每个示例中，框架负责渲染聊天界面，而托管会话在服务器端运行智能体循环：会话保存对话记录、在沙箱中运行工具，并流式传输由前端渲染的事件。

<CardGroup cols={3}>
  <Card title="Chat SDK" icon="github-logo" href="https://github.com/anthropics/claude-quickstarts/tree/main/managed-agents/chat-sdk">
    一个使用 Vercel 的 Chat SDK 构建的浏览器聊天研究分析师。每个对话都是一个持久会话，在流式传输回复的同时，实时信息流会显示工具调用。更换 Chat SDK 适配器即可将同一处理程序迁移到 Slack、Teams、Discord 或 WhatsApp。
  </Card>

  <Card title="assistant-ui" icon="github-logo" href="https://github.com/anthropics/claude-quickstarts/tree/main/managed-agents/assistant-ui">
    一个使用 assistant-ui 基础组件构建的聊天式电子表格分析师。会话即线程列表，一个 reducer 将会话事件日志转换为消息和工具卡片，每条 bash 命令在运行前都会渲染一个内联的允许/拒绝关卡。
  </Card>

  <Card title="CopilotKit (AG-UI)" icon="github-logo" href="https://github.com/anthropics/claude-quickstarts/tree/main/managed-agents/copilot-kit-ag-ui">
    一个 CopilotKit 聊天中的个人理财助手。适用于 Claude Managed Agents 的 AG-UI 适配器将每个聊天线程映射到一个托管会话，并逐令牌流式传输回复，自定义工具会在对话中内联渲染交互式图表。
  </Card>
</CardGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="定义您的智能体" icon="brain" href="https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup">
    创建可复用、带版本的智能体配置
  </Card>

  <Card title="配置环境" icon="settings" href="https://platform.claude.com/docs/zh-CN/managed-agents/environments">
    自定义网络和沙箱设置
  </Card>

  <Card title="智能体工具" icon="tool" href="https://platform.claude.com/docs/zh-CN/managed-agents/tools">
    为您的智能体启用特定工具
  </Card>

  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    处理事件并在智能体运行时对其进行引导
  </Card>

  <Card title="定时部署" icon="arrows-clockwise" href="https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments">
    按周期性 cron 计划运行您的智能体
  </Card>

  <Card title="知识 wiki 快速入门" icon="github-logo" href="https://github.com/anthropics/claude-quickstarts/tree/main/managed-agents/knowledge-wiki">
    将文档语料库一次性提炼为知识 wiki，然后以极低的成本基于它回答重复的问题
  </Card>
</CardGroup>
