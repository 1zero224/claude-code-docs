---
title: 自托管沙箱
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes
description: 在自托管沙箱中运行 Claude Managed Agents 会话，将工具执行、文件和网络出口流量保留在您自己的基础设施中。
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

“Self-hosted sandboxes”（自托管沙箱）将智能体的工具执行从 Anthropic 托管的云沙箱迁移到您控制的基础设施中，而 Anthropic 继续运行智能体本身。要了解这意味着什么并判断它是否适合您的智能体，请参阅[选择智能体环境](https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment)。

## 工作原理

`self_hosted` 环境充当一个工作队列。您运行一个 **environment worker**（环境工作进程），这是在您自己的基础设施上为该队列提供服务的进程：

1. 您创建一个以该环境为目标的[会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)。Anthropic 将该会话作为工作项加入队列。
2. 您的工作进程认领该工作项，并下载智能体的[技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills)和会话的[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)。
3. Claude 在 Anthropic 一侧运行并请求工具调用。您的工作进程在本地运行每个调用，并将结果回传。

工具的输入和输出仍会流向 Anthropic 一侧的 Managed Agents，以便模型接收结果并决定下一步操作。技能和记忆存储由 Anthropic 存储。您的沙箱会为该会话保存一份副本。要了解会话的各个部分分别在哪里运行，请参阅[沙箱能做什么、不能做什么](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-risks#what-the-sandbox-can-and-cannot-do)。

自托管沙箱支持 Managed Agents 中可用的所有 Claude 模型。您在智能体上配置模型，而不是在环境上。

`ant` CLI 以及 Python、TypeScript 和 Go SDK 都附带预构建的工作进程。Cloudflare、Daytona 和 Modal 等沙箱提供商也发布了各自的[平台指南](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#platform-guides)。

## 快速入门

本快速入门使用 `ant` CLI 运行一个常驻工作进程，向其发送一个会话，并确认智能体的工具调用在您的主机上运行。

### 前提条件

* **一个智能体：** 如果您还没有，请先完成[开始使用 Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/quickstart)，并记下智能体 ID。
* **一台 Linux 或 macOS 主机：** 主机需要在该确切路径下有 `/bin/bash`。worker 只需要出站 HTTPS。
* **一个 Claude API 密钥：** 您在自己的机器上使用它来创建会话和读取队列统计信息。不要将它放在 worker 主机上，因为智能体的工具调用可能会读取到它。
* **您自己机器上的 `ant` CLI 或 SDK：** 本快速入门中的统计和会话步骤在您自己的机器上运行，而不是在 worker 主机上。请按照[运行您的第一个会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)中在 worker 主机上安装 CLI 的相同方式安装 CLI，或者按照 [SDK、CLI 和库](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)中的说明安装 SDK。

<Note>
  在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上，工作进程使用 AWS IAM（SigV4）或[在 AWS Console 中生成的 API 密钥](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#api-key-authentication)进行身份验证，而不是使用环境密钥。请将 [`AnthropicSelfHostedEnvironmentAccess`](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#managed-policies) 托管策略附加到运行工作进程的 IAM 主体上。在 Claude Console 中生成的环境密钥不适用于 Claude Platform on AWS 端点。

  在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上，工作进程使用 [Google Cloud 凭据](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#authentication)进行身份验证，而不是使用环境密钥。请在项目上为您的工作进程所使用的 Google Cloud 身份授予 `roles/claude.user` 或 `roles/claude.admin` 角色。这两个角色都包含 [`environments.processWork`](https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions#environments)（工作进程需要该权限来轮询和管理工作项）。它们还包含工作进程运行会话所需的会话和技能权限。在 Claude Console 中生成的环境密钥不适用于 Claude Models on Google Cloud (Anthropic managed) 端点。请不要使用预构建工作进程，而是实现您自己的工作进程，直接调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。请参阅[不支持的功能](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#features-not-supported)。
</Note>

<Note>
  在 macOS 上，即使您使用 Homebrew 安装了较新版本的 Bash，worker 的 bash 工具仍会运行位于 `/bin/bash` 的系统 Bash 3.2。要在 macOS 上使用记忆存储，请先[准备 macOS 主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-a-macos-host)。
</Note>

### 运行您的第一个会话

<Steps>
  <Step title="创建自托管环境">
    在 Claude Console 中，前往[环境](https://platform.claude.com/environments)页面（`platform.claude.com/environments`），点击**创建环境**，然后在**托管**下选择**自托管**。

    或者通过 API：

    <CodeGroup defaultLanguage="CLI">
      ```bash cURL
      curl -sS --fail-with-body https://api.anthropic.com/v1/environments \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01" \
        -H "content-type: application/json" \
        -d '{
          "name": "self-hosted",
          "config": {"type": "self_hosted"}
        }'
      ```

      <CodeGroupItem>
        ```bash CLI
        ant apply environment.yaml
        ```

        <File filename="environment.yaml">
          ```yaml
          # yaml-language-server: $schema=https://platform.claude.com/schemas/ant/beta/environment.json
          name: self-hosted
          config:
            type: self_hosted
          ```
        </File>
      </CodeGroupItem>

      ```python Python
      client = anthropic.Anthropic()

      environment = client.beta.environments.create(
          name="self-hosted", config={"type": "self_hosted"}
      )
      print(environment.id)
      ```

      ```typescript TypeScript
      const client = new Anthropic();

      const environment = await client.beta.environments.create({
        name: "self-hosted",
        config: { type: "self_hosted" }
      });
      console.log(environment.id);
      ```

      ```csharp C#
      using Anthropic.Models.Beta.Environments;

      var client = new AnthropicClient();

      var environment = await client.Beta.Environments.Create(
          new EnvironmentCreateParams
          {
              Name = "self-hosted",
              Config = new BetaSelfHostedConfigParams(),
          }
      );
      Console.WriteLine(environment.ID);
      ```

      ```go Go
      client := anthropic.NewClient()

      environment, err := client.Beta.Environments.New(context.Background(), anthropic.BetaEnvironmentNewParams{
      	Name: "self-hosted",
      	Config: anthropic.BetaEnvironmentNewParamsConfigUnion{
      		OfSelfHosted: &anthropic.BetaSelfHostedConfigParams{},
      	},
      })
      if err != nil {
      	panic(err)
      }
      fmt.Println(environment.ID)
      ```

      ```java Java
      import com.anthropic.models.beta.environments.BetaSelfHostedConfigParams;
      import com.anthropic.models.beta.environments.EnvironmentCreateParams;

      void main() {
          var client = AnthropicOkHttpClient.fromEnv();

          var environment = client.beta().environments().create(
              EnvironmentCreateParams.builder()
                  .name("self-hosted")
                  .config(BetaSelfHostedConfigParams.builder().build())
                  .build()
          );
          IO.println(environment.id());
      }
      ```

      ```php PHP
      $client = new Client();

      $environment = $client->beta->environments->create(
          name: 'self-hosted',
          config: ['type' => 'self_hosted'],
      );
      echo $environment->id, PHP_EOL;
      ```

      ```ruby Ruby
      client = Anthropic::Client.new

      environment = client.beta.environments.create(
        name: "self-hosted",
        config: {type: :self_hosted}
      )
      puts environment.id
      ```
    </CodeGroup>
  </Step>

  <Step title="生成环境密钥">
    在 Console 中，打开该环境并点击**生成环境密钥**。环境密钥用于向其队列验证 worker 的身份。即使是通过 API 创建的环境，也只能在 Console 中生成环境密钥。

    在 worker 主机上导出环境 ID 和密钥：

    ```bash
    export ANTHROPIC_ENVIRONMENT_KEY="sk-ant-oat01-..."
    export ANTHROPIC_ENVIRONMENT_ID="env_..."
    ```
  </Step>

  <Step title="安装 ant CLI">
    在 worker 主机上运行以下命令。

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

    检查安装情况：

    ```bash
    ant --version
    ```
  </Step>

  <Step title="启动 worker">
    创建工作目录，然后将 `--workdir` 设置为该目录来启动 worker。如果不使用该标志，worker 将使用当前目录。

    <Tabs>
      <Tab title="Linux">
        传入 `/workspace` 以与系统默认值保持一致。

        ```bash
        sudo mkdir -p /workspace && sudo chown "$USER" /workspace
        ant beta:worker poll --workdir /workspace
        ```
      </Tab>

      <Tab title="macOS">
        根卷是只读的，因此 `mkdir` 无法创建 `/workspace`。请改用您主目录中的某个目录。如果智能体使用 skills，请按照[沙箱文件系统](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#sandbox-filesystem)中的说明，将此目录的完整路径（`echo ~/workspace` 会打印该路径）添加到其系统提示中。

        ```bash
        mkdir -p ~/workspace
        ant beta:worker poll --workdir ~/workspace
        ```
      </Tab>
    </Tabs>

    worker 会读取您导出的两个变量，并持续轮询，直到您停止它。
  </Step>

  <Step title="验证 worker 已连接">
    在您自己的机器上，将 `ANTHROPIC_API_KEY` 设置为您的 Claude API 密钥（而不是环境密钥），并将 `ANTHROPIC_ENVIRONMENT_ID` (python, ruby: `environment_id`; typescript, java, csharp: `environmentId`; go: `environmentID`; php: `$environmentId`) 设置为环境 ID。确认 `workers_polling` (java, php: `workersPolling`; csharp, go: `WorkersPolling`) 至少为 1：

    <CodeGroup>
      ```bash cURL
      curl -sS --fail-with-body "https://api.anthropic.com/v1/environments/$ANTHROPIC_ENVIRONMENT_ID/work/stats" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01"
      ```

      ```bash CLI
      ant beta:environments:work stats --environment-id "$ANTHROPIC_ENVIRONMENT_ID"
      ```

      ```python Python
      client = anthropic.Anthropic()

      stats = client.beta.environments.work.stats(environment_id)
      print(f"workers_polling={stats.workers_polling}")
      ```

      ```typescript TypeScript
      const client = new Anthropic();

      const stats = await client.beta.environments.work.stats(environmentId);

      console.log(`workers_polling=${stats.workers_polling}`);
      ```

      ```csharp C#
      var client = new AnthropicClient();

      var stats = await client.Beta.Environments.Work.Stats(environmentId);

      Console.WriteLine($"workers_polling={stats.WorkersPolling}");
      ```

      ```go Go
      client := anthropic.NewClient()

      stats, err := client.Beta.Environments.Work.Stats(
      	context.Background(),
      	environmentID,
      	anthropic.BetaEnvironmentWorkStatsParams{},
      )
      if err != nil {
      	panic(err)
      }

      fmt.Printf("workers_polling=%d\n", stats.WorkersPolling)
      ```

      ```java Java
      var client = AnthropicOkHttpClient.fromEnv();

      var stats = client.beta().environments().work().stats(environmentId);

      IO.println("workers_polling=" + stats.workersPolling().orElse(null));
      ```

      ```php PHP
      $client = new Client();

      $stats = $client->beta->environments->work->stats($environmentId);

      printf("workers_polling=%d\n", $stats->workersPolling);
      ```

      ```ruby Ruby
      client = Anthropic::Client.new

      stats = client.beta.environments.work.stats(environment_id)

      puts "workers_polling=#{stats.workers_polling}"
      ```
    </CodeGroup>

    如果 `workers_polling` (java, php: `workersPolling`; csharp, go: `WorkersPolling`) 一直为 0，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#troubleshooting)。
  </Step>

  <Step title="启动会话">
    将 `AGENT_ID` (python, ruby: `agent_id`; typescript, java, csharp: `agentId`; go: `agentID`; php: `$agentId`) 设置为您的智能体 ID。创建一个以该环境为目标的会话，然后向其发送一个任务：

    <CodeGroup>
      ```bash cURL
      # 创建会话
      curl -sS --fail-with-body https://api.anthropic.com/v1/sessions \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01" \
        -H "content-type: application/json" \
        -d '{
          "agent": "'"$AGENT_ID"'",
          "environment_id": "'"$ANTHROPIC_ENVIRONMENT_ID"'"
        }'

      # 向其发送任务：SESSION_ID 是创建响应中返回的 ID
      curl -sS --fail-with-body "https://api.anthropic.com/v1/sessions/$SESSION_ID/events" \
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
                  "text": "Write the output of \"uname -a\" to hello.txt in your working directory."
                }
              ]
            }
          ]
        }'
      ```

      ```bash CLI
      # 创建会话
      ant beta:sessions create \
        --agent "$AGENT_ID" \
        --environment-id "$ANTHROPIC_ENVIRONMENT_ID"

      # 向其发送任务：SESSION_ID 是创建响应中返回的 ID
      ant beta:sessions:events send --session-id "$SESSION_ID" <<'YAML'
      events:
        - type: user.message
          content:
            - type: text
              text: Write the output of "uname -a" to hello.txt in your working directory.
      YAML
      ```

      ```python Python
      session = client.beta.sessions.create(
          agent=agent_id,
          environment_id=environment_id,
      )

      client.beta.sessions.events.send(
          session.id,
          events=[
              {
                  "type": "user.message",
                  "content": [
                      {
                          "type": "text",
                          "text": 'Write the output of "uname -a" to hello.txt in your working directory.',
                      },
                  ],
              },
          ],
      )
      ```

      ```typescript TypeScript
      const session = await client.beta.sessions.create({
        agent: agentId,
        environment_id: environmentId
      });

      await client.beta.sessions.events.send(session.id, {
        events: [
          {
            type: "user.message",
            content: [
              {
                type: "text",
                text: 'Write the output of "uname -a" to hello.txt in your working directory.'
              }
            ]
          }
        ]
      });
      ```

      ```csharp C#
      var session = await client.Beta.Sessions.Create(new()
      {
          Agent = agentId,
          EnvironmentID = environmentId,
      });

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
                          Text = "Write the output of \"uname -a\" to hello.txt in your working directory.",
                      },
                  ],
              },
          ],
      });
      ```

      ```go Go
      session, err := client.Beta.Sessions.New(context.Background(), anthropic.BetaSessionNewParams{
      	Agent:         anthropic.BetaSessionNewParamsAgentUnion{OfString: anthropic.String(agentID)},
      	EnvironmentID: environmentID,
      })
      if err != nil {
      	panic(err)
      }

      _, err = client.Beta.Sessions.Events.Send(context.Background(), session.ID, anthropic.BetaSessionEventSendParams{
      	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
      		OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
      			Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
      				OfText: &anthropic.BetaManagedAgentsTextBlockParam{
      					Text: "Write the output of \"uname -a\" to hello.txt in your working directory.",
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
      var session = client.beta().sessions().create(SessionCreateParams.builder()
          .agent(agentId)
          .environmentId(environmentId)
          .build());

      client.beta().sessions().events().send(
          session.id(),
          EventSendParams.builder()
              .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
                  .addTextContent("Write the output of \"uname -a\" to hello.txt in your working directory.")
                  .build())
              .build());
      ```

      ```php PHP
      $session = $client->beta->sessions->create(
          agent: $agentId,
          environmentID: $environmentId,
      );

      $client->beta->sessions->events->send(
          $session->id,
          events: [
              [
                  'type' => 'user.message',
                  'content' => [['type' => 'text', 'text' => 'Write the output of "uname -a" to hello.txt in your working directory.']],
              ],
          ],
      );
      ```

      ```ruby Ruby
      session = client.beta.sessions.create(
        agent: agent_id,
        environment_id: environment_id
      )

      client.beta.sessions.events.send_(
        session.id,
        events: [
          {
            type: :"user.message",
            content: [{type: :text, text: 'Write the output of "uname -a" to hello.txt in your working directory.'}]
          }
        ]
      )
      ```
    </CodeGroup>

    会话会在环境的队列中等待，直到有 worker 认领它。如果没有连接任何 worker，会话将保持排队状态，而不会失败。
  </Step>

  <Step title="确认工具在您的主机上运行">
    在 worker 主机上，读取智能体写入的文件：

    <Tabs>
      <Tab title="Linux">
        ```bash
        cat /workspace/hello.txt
        ```
      </Tab>

      <Tab title="macOS">
        ```bash
        cat ~/workspace/hello.txt
        ```
      </Tab>
    </Tabs>

    该文件描述的是您自己主机的内核，因此智能体的工具调用是在那里运行的。要实时跟踪智能体的工作，请参阅[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。
  </Step>
</Steps>

## 会话资源

自托管沙箱仅支持 `memory_store` 资源。在自托管环境上包含 `file` 或 `github_repository` 资源的会话会被拒绝，并返回 400 错误：

```text wrap
Environment env_... is a self-hosted environment. `resources` are not supported with self-hosted environments.
```

以自托管环境为目标的[部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)遵循相同的规则。要为会话提供其自己的输入文件，请参阅[为会话暂存文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#stage-files-for-a-session)。

## 自托管沙箱与 MCP 隧道

自托管控制的是*智能体代码在哪里运行*。[MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)控制的是 *Anthropic 如何访问您网络中的 MCP 服务器*。两者相互独立：

* 在 Anthropic 云沙箱中的会话可以通过隧道访问私有 MCP 服务器。例如，私有 MCP 服务器可能部署在您防火墙后的数据库或内部 API 前面。
* 自托管会话可以使用经隧道连接的 MCP 服务器或公共 MCP 服务器。

当您希望执行和工具访问都保留在您的边界内时，请同时使用两者。要跳过隧道，请[将 MCP 服务器封装为自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools#wrap-an-mcp-server-as-custom-tools)，由您的工作进程提供服务。

## 平台指南

自托管沙箱的各个页面介绍了如何在任何沙箱平台上构建工作进程。以下提供商也发布了各自平台的指南：

* [AWS Lambda MicroVMs](https://docs.aws.amazon.com/lambda/latest/dg/microvms-integrations-claude-managed-agents.html)
* [Blaxel](https://docs.blaxel.ai/Tutorials/Claude-Managed-Agents)
* [Cloudflare](https://developers.cloudflare.com/sandbox/claude-managed-agents/)
* [Daytona](https://www.daytona.io/docs/en/guides/claude/claude-managed-agents)
* [E2B](https://e2b.dev/docs/agents/claude-managed-agents)
* [Fly.io](https://docs.sprites.dev/integrations/claude-managed-agents/)
* [GKE Agent Sandbox](https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/tree/main/ai-ml/anthropic-agent-sandbox)
* [Modal](https://github.com/modal-labs/claude-managed-agents-modal-sandbox)
* [Namespace](https://namespace.so/docs/integrations/claude)
* [Superserve](https://docs.superserve.ai/integrations/managed-agents/claude-managed-agents)
* [Vercel](https://vercel.com/kb/guide/run-claude-managed-agent-tools-with-vercel-sandbox)

## 后续步骤

<CardGroup cols={2}>
  <Card title="部署工作进程" icon="play" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers">
    运行 SDK 工作进程、通过 webhook 触发工作进程，或为每个会话提供独立的沙箱。
  </Card>

  <Card title="自托管沙箱中的记忆存储" icon="brain" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory">
    准备主机、配置同步，并处理只读存储和冲突。
  </Card>

  <Card title="自定义工具和 MCP 服务器" icon="tool" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools">
    从工作进程提供您自己的工具，包括来自您网络内 MCP 服务器的工具。
  </Card>

  <Card title="监控和故障排除" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations">
    读取队列深度、干净地停止会话和工作进程，并修复常见故障。
  </Card>

  <Card title="工作进程参考" icon="book" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference">
    CLI 标志、环境变量、文件系统路径和 SDK 辅助选项。
  </Card>

  <Card title="安全模型" icon="lock" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-security">
    自托管沙箱环境的责任共担模型。
  </Card>
</CardGroup>

## 可用性

自托管沙箱在所有模型上处于 beta 阶段。

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
