---
title: 在自托管服务器上调用 API
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api
description: 将官方 SDK、cURL 或 ant CLI 指向您的自托管 Claude Managed Agents 服务器，启动会话并读取智能体的回复。然后，如果需要，连接您自己的 worker。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

自托管 Claude Managed Agents 服务器提供的 API 与 Anthropic 托管的 [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 服务相同。您的代码使用官方 SDK、cURL 或 `ant` CLI 调用它。有两处变化：基础 URL 是您服务器的地址，API 密钥是您的[服务器 API 密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted#the-three-credentials)，而不是 Claude API 密钥。

要在浏览器中创建智能体并运行会话，请参阅[使用您服务器上的控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console)。

## 先决条件

* **正在运行的服务器：** 使用[开始使用自托管 Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart)或[在 Kubernetes 上部署](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes)启动一个服务器。
* **服务器的访问权限：** 如果服务器由其他人运行，请向他们询问服务器的地址以及如何进行身份验证。如果他们使用单点登录，请参阅[让开发者从终端登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#let-developers-log-in-from-a-terminal)。
* **`ant` CLI：** 请按照 [CLI 快速入门](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart#installation)中的说明，安装最新版本。

## 将您的客户端指向您的服务器

SDK 和 `ant` CLI 会从 `ANTHROPIC_BASE_URL` 读取服务器地址，从 `ANTHROPIC_API_KEY` 读取 API 密钥。请在您为此服务器保留的终端中设置这两个变量。

对于[快速入门栈](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart)，请在包含您的 `.env` 文件的目录中运行以下命令：

```bash
export ANTHROPIC_BASE_URL=http://127.0.0.1:8400
export ANTHROPIC_API_KEY=$(grep '^CMA_SERVER__API_KEY=' .env | cut -d= -f2)
```

在 Kubernetes 上，请按照[运行会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#run-a-session)中的说明设置这两个变量，并保持其端口转发处于运行状态。

在该终端中，每个 `ant` 命令（例如 `ant beta:agents list`）都会发送到您的服务器。请不要在该终端中运行 Claude Code 或 `docker compose`，因为它们会读取相同的变量。

## 启动会话

会话需要一个智能体的 ID 和一个环境的 ID：

* **智能体：** 按照[创建智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#create-an-agent)中的说明创建一个，并将您的客户端指向您的服务器。如果您发送原始 HTTP 请求，请将 URL 中的 `https://api.anthropic.com` 替换为 `$ANTHROPIC_BASE_URL`。
* **环境：** 快速入门栈会创建一个名为 `compose-sandbox` 的环境，Helm chart 会创建一个名为 `default` 的环境。`ant beta:environments list` 会打印每个环境的名称和 ID。对于您自己的 worker，该 ID 是管理 CLI 在[运行您自己的 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#point-a-worker-at-your-server)中打印的 ID。

将您的两个 ID 放入您的 shell，替换这些示例 ID：

```bash
export AGENT_ID=agent_011CYm1BLqPXpQRk5khsSXrs
export ENVIRONMENT_ID=env_01595EKxaaTTGwwY3kyXdtbs
```

以下示例创建一个会话，并在同一请求中发送其第一条消息：

<CodeGroup>
  ```bash cURL
  curl -sS --fail-with-body "$ANTHROPIC_BASE_URL/v1/sessions" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'",
      "initial_events": [
        {
          "type": "user.message",
          "content": [{"type": "text", "text": "List the files in the working directory."}]
        }
      ]
    }'
  ```

  ```bash CLI
  ant beta:sessions create --agent "$AGENT_ID" --environment-id "$ENVIRONMENT_ID" \
    --initial-event '{type: user.message, content: [{type: text, text: "List the files in the working directory."}]}'
  ```

  ```python Python
  import os

  from anthropic import Anthropic

  client = Anthropic()

  session = client.beta.sessions.create(
      agent=os.environ["AGENT_ID"],
      environment_id=os.environ["ENVIRONMENT_ID"],
      initial_events=[
          {
              "type": "user.message",
              "content": [
                  {"type": "text", "text": "List the files in the working directory."}
              ],
          },
      ],
  )
  print(session.id)
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const session = await client.beta.sessions.create({
    agent: process.env["AGENT_ID"]!,
    environment_id: process.env["ENVIRONMENT_ID"]!,
    initial_events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "List the files in the working directory." }]
      }
    ]
  });
  console.log(session.id);
  ```

  ```csharp C#
  using System;
  using Anthropic;
  using Anthropic.Models.Beta.Agents;
  using Anthropic.Models.Beta.Sessions;
  using Anthropic.Models.Beta.Sessions.Events;

  AnthropicClient client = new();

  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = Environment.GetEnvironmentVariable("AGENT_ID")!,
      EnvironmentID = Environment.GetEnvironmentVariable("ENVIRONMENT_ID")!,
      InitialEvents =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content =
              [
                  new BetaManagedAgentsTextBlock
                  {
                      Text = "List the files in the working directory.",
                  },
              ],
          },
      ],
  });
  Console.WriteLine(session.ID);
  ```

  ```go Go
  package main

  import (
  	"context"
  	"fmt"
  	"os"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  func main() {
  	client := anthropic.NewClient()

  	session, err := client.Beta.Sessions.New(context.Background(), anthropic.BetaSessionNewParams{
  		Agent: anthropic.BetaSessionNewParamsAgentUnion{
  			OfString: anthropic.String(os.Getenv("AGENT_ID")),
  		},
  		EnvironmentID: os.Getenv("ENVIRONMENT_ID"),
  		InitialEvents: []anthropic.BetaSessionNewParamsInitialEventUnion{{
  			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  					OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  						Text: "List the files in the working directory.",
  					},
  				}},
  			},
  		}},
  	})
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println(session.ID)
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.beta.sessions.SessionCreateParams;
  import com.anthropic.models.beta.sessions.events.BetaManagedAgentsUserMessageEventParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var session = client.beta().sessions().create(SessionCreateParams.builder()
          .agent(System.getenv("AGENT_ID"))
          .environmentId(System.getenv("ENVIRONMENT_ID"))
          .addInitialEvent(BetaManagedAgentsUserMessageEventParams.builder()
              .addTextContent("List the files in the working directory.")
              .build())
          .build());
      IO.println(session.id());
  }
  ```

  ```php PHP
  <?php

  use Anthropic\Client;

  $client = new Client();

  $session = $client->beta->sessions->create(
      agent: (string) getenv('AGENT_ID'),
      environmentID: (string) getenv('ENVIRONMENT_ID'),
      initialEvents: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => 'List the files in the working directory.']],
          ],
      ],
  );
  echo $session->id, "\n";
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::Client.new

  session = client.beta.sessions.create(
    agent: ENV.fetch("AGENT_ID"),
    environment_id: ENV.fetch("ENVIRONMENT_ID"),
    initial_events: [
      {
        type: :"user.message",
        content: [{type: :text, text: "List the files in the working directory."}]
      }
    ]
  )
  puts session.id
  ```
</CodeGroup>

响应中包含会话的 ID，该 ID 以 `sesn_` 开头。智能体会立即开始工作。它的内置工具（例如 `bash`）在为该环境提供服务的 worker 上运行。

要观察会话，请在浏览器中打开[您服务器上的控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console)，选择 **Sessions**，然后选择该会话。像本页面这样使用服务器 API 密钥启动的会话仅对管理员可见。

如果要改为在终端中跟踪会话，请将其 ID 传给 [`ant beta:sessions connect`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/sessions-connect)：

```bash
ant beta:sessions connect sesn_011CZkZAtmR3yMPDzynEDxu7
```

有关会话还能做的其他事情（例如固定智能体版本），请参阅[启动会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)。

## 发送消息并读取回复

将您的会话 ID 放入您的 shell，替换此示例 ID：

```bash
export SESSION_ID=sesn_011CZkZAtmR3yMPDzynEDxu7
```

请等到会话处于空闲状态后再运行下一个示例。该示例会在一个轮次结束时停止读取。如果之前的轮次仍在运行，它的结束可能会在您消息的回复到达之前使示例停止。控制台和 `ant beta:sessions connect` 都会显示会话的状态。

以下示例发送一条消息，并打印智能体的回复。它会先打开会话的事件流，因为事件流只会传递其打开之后发生的事件：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单条 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 来运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单条 shell 命令所能实现的范围。要在终端中发送消息并读取回复，请使用 [`ant beta:sessions connect`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/sessions-connect)。
  </CodeGroupItem>

  ```python Python
  import os

  from anthropic import Anthropic

  client = Anthropic()
  session_id = os.environ["SESSION_ID"]

  with client.beta.sessions.events.stream(session_id) as stream:
      client.beta.sessions.events.send(
          session_id,
          events=[
              {
                  "type": "user.message",
                  "content": [
                      {"type": "text", "text": "Summarize what you did in one sentence."}
                  ],
              },
          ],
      )

      for event in stream:
          match event.type:
              case "agent.message":
                  for block in event.content:
                      if block.type == "text":
                          print(block.text, end="")
              case "session.error":
                  error_message = event.error.message if event.error else "unknown"
                  print(f"\n[Error: {error_message}]")
              case "session.status_idle":
                  if event.stop_reason.type != "requires_action":
                      break
              case "session.status_terminated":
                  break
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();
  const sessionId = process.env["SESSION_ID"]!;

  const stream = await client.beta.sessions.events.stream(sessionId);
  await client.beta.sessions.events.send(sessionId, {
    events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "Summarize what you did in one sentence." }]
      }
    ]
  });

  events: for await (const event of stream) {
    switch (event.type) {
      case "agent.message":
        for (const block of event.content) {
          if (block.type === "text") {
            process.stdout.write(block.text);
          }
        }
        break;
      case "session.error":
        console.log(`\n[Error: ${event.error?.message ?? "unknown"}]`);
        break;
      case "session.status_idle":
        if (event.stop_reason.type !== "requires_action") {
          break events;
        }
        break;
      case "session.status_terminated":
        break events;
    }
  }
  ```

  ```csharp C#
  using System;
  using Anthropic;
  using Anthropic.Models.Beta.Sessions.Events;

  AnthropicClient client = new();
  var sessionId = Environment.GetEnvironmentVariable("SESSION_ID")!;

  using var stream = await client.Beta.Sessions.Events.WithRawResponse.StreamStreaming(sessionId);
  await client.Beta.Sessions.Events.Send(sessionId, new()
  {
      Events =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content = [new BetaManagedAgentsTextBlock { Text = "Summarize what you did in one sentence." }],
          },
      ],
  });

  await foreach (var streamEvent in stream.Enumerate())
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
      else if (streamEvent.Value is BetaManagedAgentsSessionErrorEvent error)
      {
          Console.WriteLine($"\n[Error: {error.Error?.Message ?? "unknown"}]");
      }
      else if (streamEvent.Value is BetaManagedAgentsSessionStatusIdleEvent idle
          && idle.StopReason?.Value is not BetaManagedAgentsSessionRequiresAction)
      {
          break;
      }
      else if (streamEvent.Value is BetaManagedAgentsSessionStatusTerminatedEvent)
      {
          break;
      }
  }
  ```

  ```go Go
  package main

  import (
  	"cmp"
  	"context"
  	"fmt"
  	"os"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  func main() {
  	client := anthropic.NewClient()
  	ctx := context.Background()
  	sessionID := os.Getenv("SESSION_ID")

  	stream := client.Beta.Sessions.Events.StreamEvents(ctx, sessionID, anthropic.BetaSessionEventStreamParams{})
  	defer stream.Close()

  	if _, err := client.Beta.Sessions.Events.Send(ctx, sessionID, anthropic.BetaSessionEventSendParams{
  		Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  					OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  						Text: "Summarize what you did in one sentence.",
  					},
  				}},
  			},
  		}},
  	}); err != nil {
  		panic(err)
  	}

  events:
  	for stream.Next() {
  		switch event := stream.Current().AsAny().(type) {
  		case anthropic.BetaManagedAgentsAgentMessageEvent:
  			for _, block := range event.Content {
  				if block.Type == "text" {
  					fmt.Print(block.Text)
  				}
  			}
  		case anthropic.BetaManagedAgentsSessionErrorEvent:
  			fmt.Printf("\n[Error: %s]\n", cmp.Or(event.Error.Message, "unknown"))
  		case anthropic.BetaManagedAgentsSessionStatusIdleEvent:
  			if _, paused := event.StopReason.AsAny().(anthropic.BetaManagedAgentsSessionRequiresAction); !paused {
  				break events
  			}
  		case anthropic.BetaManagedAgentsSessionStatusTerminatedEvent:
  			break events
  		}
  	}
  	if err := stream.Err(); err != nil {
  		panic(err)
  	}
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.JsonObject;
  import com.anthropic.models.beta.sessions.events.BetaManagedAgentsUserMessageEventParams;
  import com.anthropic.models.beta.sessions.events.EventSendParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();
      var sessionId = System.getenv("SESSION_ID");

      try (var stream = client.beta().sessions().events().streamStreaming(sessionId)) {
          client.beta().sessions().events().send(
              sessionId,
              EventSendParams.builder()
                  .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
                      .addTextContent("Summarize what you did in one sentence.")
                      .build())
                  .build()
          );

          events:
          for (var event : stream) {
              switch (event.type().value()) {
                  case AGENT_MESSAGE -> {
                      for (var block : event.asAgentMessage().content()) {
                          block.text().ifPresent(textBlock -> IO.print(textBlock.text()));
                      }
                  }
                  case SESSION_ERROR -> {
                      var errorMessage =
                          event.asSessionError().error()._json().orElse(null) instanceof JsonObject json
                              ? json.values().get("message").asStringOrThrow()
                              : "unknown";
                      IO.println("\n[Error: " + errorMessage + "]");
                  }
                  case SESSION_STATUS_IDLE -> {
                      if (!event.asSessionStatusIdle().stopReason().isRequiresAction()) {
                          break events;
                      }
                  }
                  case SESSION_STATUS_TERMINATED -> {
                      break events;
                  }
                  default -> {}
              }
          }
      }
  }
  ```

  ```php PHP
  <?php

  use Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionErrorEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionRequiresAction;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionStatusIdleEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionStatusTerminatedEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock;
  use Anthropic\Client;

  $client = new Client();
  $sessionId = (string) getenv('SESSION_ID');

  $stream = $client->beta->sessions->events->streamStream($sessionId);
  $client->beta->sessions->events->send(
      $sessionId,
      events: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => 'Summarize what you did in one sentence.']],
          ],
      ],
  );

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof ManagedAgentsAgentMessageEvent:
              foreach ($event->content as $block) {
                  if ($block instanceof ManagedAgentsTextBlock) {
                      echo $block->text;
                  }
              }
              break;
          case $event instanceof ManagedAgentsSessionErrorEvent:
              printf("\n[Error: %s]\n", $event->error->message);
              break;
          case $event instanceof ManagedAgentsSessionStatusIdleEvent:
              if (!($event->stopReason instanceof ManagedAgentsSessionRequiresAction)) {
                  break 2; // Break foreach loop
              }
              break;
          case $event instanceof ManagedAgentsSessionStatusTerminatedEvent:
              break 2; // Break foreach loop
      }
  }
  $stream->close();
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::Client.new
  session_id = ENV.fetch("SESSION_ID")

  stream = client.beta.sessions.events.stream_events(session_id)

  client.beta.sessions.events.send_(
    session_id,
    events: [{
      type: "user.message",
      content: [{type: "text", text: "Summarize what you did in one sentence."}]
    }]
  )

  stream.each do |event|
    case event
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
      event.content.each do |block|
        print block.text if block.is_a?(Anthropic::Beta::Sessions::BetaManagedAgentsTextBlock)
      end
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionErrorEvent
      puts "\n[Error: #{event.error&.message || "unknown"}]"
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusIdleEvent
      break unless event.stop_reason.is_a?(Anthropic::Beta::Sessions::BetaManagedAgentsSessionRequiresAction)
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusTerminatedEvent
      break
    else
      # 忽略其他事件类型
    end
  end
  ```
</CodeGroup>

除了打印 `agent.message` 文本外，循环还会处理三种事件：

* **`session.status_idle`：** 循环结束，除非该事件的 `stop_reason.type` 是 `requires_action`。
* **`session.status_terminated`：** 循环结束。
* **`session.error`：** 循环会打印错误的消息并继续读取，因为服务器可以重试失败的模型调用并完成该轮次。

`requires_action` 表示会话已为工具调用而暂停。结束暂停的方式取决于该调用：

* **自定义工具调用，或确认工具调用的请求：** 只有您的代码可以回答它。如果您的智能体使用了其中任何一种，请参阅[回答使会话暂停的工具调用](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#answer-tool-calls-that-pause-the-session)。
* **对由 worker 运行的工具（例如 `bash`）的调用：** 该调用通常不会暂停会话。如果大约 3 分钟内没有 worker 接手该调用，会话会暂停，直到某个 worker 返回结果。此时您的代码无需回答任何内容。请保持事件流处于打开状态，当 worker 运行该调用时，该轮次会继续。

有关会话发送的其他事件，以及如何在不遗漏任何事件的情况下重新连接，请参阅[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。

## 在会话中传入和传出数据

数据通过 worker 的工作目录进入和离开会话。

在快速入门栈上，每个会话的容器启动时，其 `/workspace` 目录中除了会话仓库的克隆之外不包含任何内容。默认情况下，在一轮结束大约一分钟后，容器会退出，Docker 会将其连同该目录一起删除。会话必须保留的任何内容都必须在此之前通过工具调用（例如上传或 `git push`）传出。

容器在启动大约一小时后也会退出，即使正处于某一轮的中途。没有任何设置可以改变这一点。该轮次会在新容器中继续，通常在一两分钟之后，并且您的客户端不会收到关于此切换的任何事件。此切换有两项代价：

* **命令：** 在旧容器中运行的所有内容都会停止。新容器会从头重新运行被中断的工具调用，但不会重新启动智能体留在后台运行的命令。
* **文件：** 新容器的 `/workspace` 再次为空，只包含会话仓库的全新克隆。

在较长的轮次中，请让智能体边工作边上传或推送其成果，并避免使用运行两次会造成危害的命令。

在 Kubernetes 上，一个会话最多保留一个 pod 约一小时，并且该轮次会以相同的方式继续。有两点不同：新 pod 会在 `/workspace` 中找到会话的文件，并且它会在旧 pod 停止大约 6 分钟后启动。

对于您自己启动的 worker，请在发送需要输入文件的消息之前，将这些文件放入工作目录。之后再从中读取输出文件。

当您自己为每个会话启动容器时，请将包含会话输入文件的目录挂载为容器的工作目录。容器退出后，从中收集输出文件。请参阅[为会话准备文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#stage-files-for-a-session)。

## 在托管服务指南中查找更多示例

托管服务的指南介绍了 API 的[核心概念](https://platform.claude.com/docs/zh-CN/managed-agents/overview#core-concepts)，并为每项功能提供了示例。在设置了 `ANTHROPIC_BASE_URL` 和 `ANTHROPIC_API_KEY` 的终端中运行时，这些指南中的 SDK 和 CLI 示例会调用您的服务器。这些指南中的 cURL 示例使用的是 `https://api.anthropic.com`，因此在运行之前，请将该地址替换为 `$ANTHROPIC_BASE_URL`。

对于某些功能，您的服务器需要先进行设置，或者示例需要进行修改：

| 任务 | 托管服务指南 | 在您的服务器上 |
| ----- | ----- | ----- |
| 设置智能体的模型、系统提示和版本 | [定义您的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup) | 某些字段[取决于您的推理提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#differences-that-depend-on-your-inference-provider)。 |
| 选择内置工具，或添加自定义工具 | [工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools) | 要使用 Web 工具，请先按照 [Web 工具需要服务器设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#web-tools-need-a-server-setting)中的说明启用它们。具体有哪些工具取决于您的推理提供商。 |
| 连接 MCP 服务器并存储其令牌 | [MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)和[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults) | 将 MCP 服务器的主机添加到 [MCP 服务器允许列表](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#mcp-servers-need-an-allowlist-entry)中。将令牌存储为 `static_bearer` 凭据。 |
| 为智能体提供技能 | [技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills) | 将每个技能作为单独的文件上传，因为您的服务器会拒绝 zip 归档。该指南示例中的[预构建技能](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#features-that-arent-available)不可用。 |
| 跨会话保留记忆 | [使用智能体记忆](https://platform.claude.com/docs/zh-CN/managed-agents/memory) | 如果您运行自己的 worker，请为它们提供一个可写的 `/mnt/memory` 目录，详见[自托管沙箱中的记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)。 |
| 将工作交给其他智能体 | [多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration) | 将该指南示例中的 `multiagent_20261001` 块替换为 [`coordinator` 块](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#features-that-behave-differently)。 |
| 在 Git 仓库中工作 | [访问 GitHub](https://platform.claude.com/docs/zh-CN/managed-agents/github) | 按照 [Git 仓库需要允许列表条目](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#git-repositories-need-an-allowlist-entry)中的说明，在您服务器的设置中列出 Git 主机。该指南中的示例智能体还需要将 `api.githubcopilot.com` 加入 MCP 服务器允许列表。 |
| 让评分器根据评分标准检查工作 | [定义结果](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes) | 以文本形式发送评分标准。Files API 不可用，因此请从工作目录中[收集输出文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#move-data-in-and-out-of-a-session)。 |
| 跟踪会话并回答其工具调用 | [会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming) | 该指南的第一个示例会在任何暂停时关闭事件流，因此请改为从[发送消息并读取回复](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#send-a-message-and-read-the-reply)中的循环开始。 |

[与托管服务之间的差异](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences)列出了完整清单。在迁移现有应用程序之前，请先查看该页面。

## 运行您自己的 worker

[快速入门栈](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart)会创建一个环境并运行其 worker。[在 Kubernetes 上部署](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes)中的 Helm chart 也是如此。如果您只需要这些 worker，请跳过本节。

运行 worker 的方式与[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)相同：使用相同的 `ant beta:worker` 命令和 SDK worker，并采用[部署自托管 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers) 中所示的方式。例外的是由 webhook 触发的 worker，因为您的服务器不会发送任何 webhook。

### 创建环境及其密钥

请使用管理 CLI（而不是在 Claude Console 中）创建一个环境和一个环境密钥。该 CLI `cma-server-admin` 位于发布归档的顶层，因此请在该位置运行此命令。对于 `-base-url`，请传递您的 worker 访问服务器所用的地址：

* **在运行快速入门栈的机器上的 worker：** `http://127.0.0.1:8400`
* **另一台机器上的 worker：** [您放在服务器前面的 TLS 代理](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#put-tls-in-front-of-the-server)的 `https` 地址

该管理 CLI 会在同一地址调用服务器，因此请在能够访问该地址的机器上运行它。

```bash
echo "$ANTHROPIC_API_KEY" | ./cma-server-admin bootstrap \
  -base-url http://127.0.0.1:8400 -name my-env -key-file -
```

该命令会打印 worker 的设置、启动 worker 的命令，以及密钥的 ID 和过期时间：

```text wrap
export ANTHROPIC_BASE_URL='http://127.0.0.1:8400'
export ANTHROPIC_ENVIRONMENT_ID='env_...'
export ANTHROPIC_ENVIRONMENT_KEY='sk-ant-oat01-...'
# then: histchars= HISTFILE=/dev/null ant beta:worker poll --workdir /workspace
# key id: 'etok_...'  expires: '...'
```

该命令只显示一次密钥，因此请将密钥、其 ID 和过期时间存储在您的密钥管理器中。该密钥在 365 天后失效。在此之前，请按照[为您自己的 worker 替换环境密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#replace-an-environment-key-for-your-own-workers)中的说明操作。

不要将服务器 API 密钥放在运行 worker 的机器上。请在不包含任何需要保护内容的容器或虚拟机中运行 worker。有关您的其他安全职责，请参阅[您的安全职责](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted#your-security-responsibilities)。

### 运行本地实验

在您启动 worker 的 shell 中运行这三行 `export`。然后使用以下命令（而不是打印出的命令）启动 worker。第一条命令会创建 worker 的工作目录，因为 worker 不会创建它：

```bash
mkdir -p "$HOME/cma-workspace" "$HOME/.config/cma"
cat > "$HOME/.config/cma/worker.inputrc" <<'EOF'
$include /etc/inputrc
set disable-completion on
EOF
histchars= HISTFILE=/dev/null INPUTRC="$HOME/.config/cma/worker.inputrc" \
  ant beta:worker poll --workdir "$HOME/cma-workspace"
```

`histchars`、`INPUTRC` 和 `HISTFILE` 用于设置 worker 的 `bash` 工具背后的 shell，使其按照智能体编写的原样运行每条命令：

* **`histchars=`：** shell 不会将 `!` 视为对其命令历史的引用。
* **`INPUTRC`：** 该文件会关闭补全功能，否则补全会吞掉命令中的每个制表符。请将该文件放在工作目录之外，以免智能体的文件工具改写它。
* **`HISTFILE=/dev/null`：** shell 既不读取也不写入历史文件。

不要将此 worker 用于其他任何用途。它持有环境密钥，并在一个进程中运行每个会话的工具。它也不会克隆会话的仓库。会话会在没有这些仓库的情况下运行，并且您的客户端不会收到任何错误。

### 在生产环境中运行 worker

对于本地实验以外的任何情况，请为每个会话启动一个容器，并在其中运行 `ant beta:worker run`。[使用启动脚本启动沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#launch-sandboxes-with-a-spawn-script)展示了具体做法。请为该容器的镜像提供[构建您自己的沙箱镜像](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#build-your-own-sandbox-image)中列出的内容，并在该设置中更改三处：

* **环境密钥：** 不要将 `ANTHROPIC_ENVIRONMENT_KEY` 传入会话的容器。服务器会对该环境中的每个会话接受该密钥，而容器只需要访问一个会话。
* **工作项的机密：** 请传递此机密来代替该密钥，因为服务器仅针对这一个会话接受它。[转发工作项的机密](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#forward-the-work-items-secret)展示了您的代码接收它的位置。在该节设置 `ANTHROPIC_WORK_SECRET` 的地方，请将该机密放在一个只有容器用户可以读取的文件中。例如，将该机密通过管道传入容器的标准输入，并让容器在启动 worker 之前写入该文件。然后将 `ANTHROPIC_WORK_SECRET_FILE` 设置为该文件的路径。
* **服务器地址：** 将 `ANTHROPIC_BASE_URL` 设置为容器可以访问的地址。`http://127.0.0.1:8400` 不是这样的地址。

对于 SDK worker，还有两点适用：

* **版本：** 请按照 [SDK、CLI 和库](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)中的说明，基于最新版本的 SDK 构建 worker。该版本的 worker 会在运行需要批准的工具调用之前等待批准，并且绝不会运行被策略拒绝的调用。基于更早版本构建的 worker 会在每个工具调用到达后立即运行它。
* **示例：** [在沙箱内运行 SDK worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#run-the-sdk-worker-inside-the-sandbox) 中的示例会读取环境密钥，因此无法直接在此使用。

### 构建您自己的沙箱镜像

您自己的镜像需要以下软件包和设置：

* **软件包：** Git 2.38 或更高版本、`curl`、`jq` 和 ripgrep（worker 的 `grep` 工具使用它进行搜索）。

* **Shell 设置：** 与本地实验中相同的 `histchars`、`HISTFILE` 和 `INPUTRC`，以及 `TERMINFO`。请在 Dockerfile 中任何 `USER` 行之前加入以下几行：

  ```dockerfile
  ENV histchars=
  RUN printf '\nset disable-completion on\n' >> /etc/inputrc
  ENV INPUTRC=/etc/inputrc TERMINFO=/etc/terminfo HISTFILE=/dev/null
  RUN test -r /etc/terminfo/d/dumb
  ```

  `TERMINFO` 必须指定包含 `d/dumb`（worker 的 `bash` 读取的终端描述）的目录。在已安装 `bash` 的 Alpine 上，该目录是 `/etc/terminfo`，如上所示。在其他基础镜像（例如 Debian 或 Ubuntu）上，该目录是 `/lib/terminfo` 或 `/usr/share/terminfo`，以包含 `d/dumb` 的那个为准。请在 `TERMINFO` 设置和 `RUN test` 行中写入该目录；如果该文件缺失，`RUN test` 行会使构建停止。

该镜像还需要归档中的两个脚本，它们会在 worker 启动之前运行：

1. 将 `image-build/with-repositories.sh` 复制到 `/usr/local/bin/cma-with-repositories`。它会克隆一个会话的仓库。
2. 将 `image-build/with-egress.sh` 复制到 `/usr/local/bin/cma-with-egress`。如果您的服务器运行了["egress gateway"（出口网关）](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#environment-variable-credentials)，它会将会话的工具指向该网关。
3. 使这两个文件都可执行。
4. 将两者都放在 worker 命令的前面，例如 `cma-with-egress cma-with-repositories ant beta:worker run ...`。如果您的镜像运行归档中的 `kubernetes/sandbox/worker.sh`，请跳过此步骤，因为该脚本就是以这种方式启动 worker 的。

不使用 `cma-with-repositories` 启动的 worker 会在没有仓库的情况下运行会话。您的客户端不会收到任何错误。

### 从托管服务迁移 worker

如果您已在为 Anthropic 的托管服务运行 worker，这些 worker 可以改为为您的服务器提供服务。请完成以下三件事：

* 将管理 CLI 打印的三个 `export` 值提供给轮询工作的进程。
* 按照本地实验中的方式设置 `histchars`、`HISTFILE` 和 `INPUTRC` 后启动该进程。
* 进行[在生产环境中运行 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#run-workers-in-production)中所述的更改，并对照[构建您自己的沙箱镜像](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#build-your-own-sandbox-image)检查每个会话的镜像。

## 后续步骤

* [让开发者从终端登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#let-developers-log-in-from-a-terminal)：设置单点登录，使开发者能够在没有 `ANTHROPIC_API_KEY` 的终端中以自己的身份使用 `ant` CLI。
* [智能体身份和凭据](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity)：让外部系统信任您的智能体，并将 vault 凭据保存在您的密钥管理器中。
* [推理提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider)：将模型调用发送到云提供商或您自己的模型网关。
