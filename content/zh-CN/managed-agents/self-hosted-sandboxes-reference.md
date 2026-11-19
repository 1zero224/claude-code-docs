---
title: 自托管 worker 参考
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference
description: 自托管沙箱 worker 的参考文档：ant CLI 标志、环境变量、主机要求、文件系统路径以及 SDK 辅助工具选项。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  status: beta
  betaHeader: managed-agents-2026-04-01
  zdr:
    eligibility: not-eligible
    note: 不具备 ZDR 资格，自托管 Managed Agents 除外。会话是有状态的资源；对话记录会一直保留，直到您将其删除。
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

本页记录了为 `self_hosted` 环境提供服务的预构建 worker。如需面向任务的指南，请从[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)和[部署自托管 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers) 开始。

## CLI 命令和标志

| 命令                     | 描述                                                         |
| ---------------------- | ---------------------------------------------------------- |
| `ant beta:worker poll` | 从环境的队列中认领工作项，并在进程内运行每个会话。使用 `--on-work` 时，改为针对每个工作项调用您的脚本。 |
| `ant beta:worker run`  | 处理一个已认领的会话后退出。将其用作每会话沙箱的入口点。                               |

| 标志                  | 描述                                                                                                                                                                                                                                                                                                                                                               |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--environment-id`  | 要轮询工作的环境。也会从 `ANTHROPIC_ENVIRONMENT_ID` 读取。                                                                                                                                                                                                                                                                                                                      |
| `--environment-key` | 使用此环境对 worker 进行身份验证。也会从 `ANTHROPIC_ENVIRONMENT_KEY` 读取。                                                                                                                                                                                                                                                                                                         |
| `--workdir`         | worker 下载技能以及工具读写文件的目录。默认为 `.`（当前目录）。该目录必须在 worker 启动前已存在。                                                                                                                                                                                                                                                                                                       |
| `--on-work`         | 针对每个已认领工作项调用的脚本，用于替代在进程内运行工具。该脚本通过环境变量接收会话详细信息，并通过标准输入以 JSON 形式接收工作项。                                                                                                                                                                                                                                                                                            |
| `--max-idle`        | 会话处于空闲状态时，worker 在 `ant beta:worker run` 退出或 `ant beta:worker poll` 转到下一个工作项之前等待的时长。因 [`requires_action` 停止原因](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#respond-when-the-session-goes-idle)而处于空闲状态的时间不计入。默认为 `60s`。将其设置为 `0` 可关闭超时。使用 `--on-work` 脚本时，`ant beta:worker poll` 会忽略此标志。请改为将其传递给您的脚本所启动的 `ant beta:worker run` 命令。 |
| `--log-format`      | 日志输出格式。使用 `json` 以便进行结构化日志采集。默认为 `text`。                                                                                                                                                                                                                                                                                                                         |

## 环境变量

| 变量                              | 描述                 | 设置方                                                                                                                                              |
| ------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ANTHROPIC_ENVIRONMENT_ID`      | worker 所服务队列对应的环境。 | 您，在 worker 主机上设置。轮询器会将其传递给 `--on-work` 脚本。                                                                                                       |
| `ANTHROPIC_ENVIRONMENT_KEY`     | 向其队列验证 worker 的身份。 | 您，在 worker 主机上设置。轮询器会将其传递给 `--on-work` 脚本。                                                                                                       |
| `ANTHROPIC_SESSION_ID`          | 已认领工作项所代表的会话。      | 轮询器，为 `--on-work` 脚本设置。                                                                                                                          |
| `ANTHROPIC_WORK_ID`             | 已认领的工作项。           | 轮询器，为 `--on-work` 脚本设置。                                                                                                                          |
| `ANTHROPIC_WORK_SECRET`         | 工作项针对其会话的机密。       | 您。轮询器不会设置它。请参阅[转发工作项的机密](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#forward-the-work-items-secret)。 |
| `ANTHROPIC_BASE_URL`            | 覆盖默认的 API 端点。可选。   | 您，在 worker 主机上设置。                                                                                                                                |
| `ANTHROPIC_WEBHOOK_SIGNING_KEY` | 验证传入的 webhook 负载。  | 您，在 webhook 处理程序主机上设置。                                                                                                                           |

## 主机要求

| Worker          | 要求                                                                                                         |
| --------------- | ---------------------------------------------------------------------------------------------------------- |
| 所有 worker       | 一台 Linux 或 macOS 主机，且 `/bin/bash` 位于该确切路径。worker 的 `bash` 工具会直接运行它，而不查询 `PATH`。在 macOS 上，即为系统自带的 Bash 3.2。 |
| TypeScript SDK  | `PATH` 上有 `unzip` 和 `tar`，以及 Node.js 22 或更高版本。                                                             |
| Python 和 Go SDK | 无需额外的二进制文件。这些 SDK 使用其标准库进行归档解压。                                                                            |

[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#requirements)有其自身的额外要求。

在 macOS 上，根卷是只读的，因此 `mkdir` 无法创建 `/workspace`。请将 worker 的工作目录设置为您可以写入的目录，例如您主目录中的 `workspace` 目录。要在 macOS 上使用记忆存储，请先[准备 macOS 主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-a-macos-host)。

## 沙箱文件系统

| 路径                         | 内容                                                                                                                                                                                        |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/workspace`               | 自托管沙箱相关页面的示例中使用的工作目录。它不是 worker 的默认值：除非您设置 `--workdir` 或 `workdir` (go: `Workdir`)，否则 worker 使用当前目录。Anthropic 的云沙箱将技能和代码仓库保存在 `/workspace` 中。如果您使用不同的工作目录，请更新智能体的系统提示，以便 Claude 能够找到技能文件。 |
| `<workdir>/skills/<name>/` | 智能体已下载的技能。                                                                                                                                                                                |
| `/mnt/memory/<store>/`     | 每个附加的记忆存储的本地副本，每个存储对应一个目录。API 会在会话的记忆存储资源的 `mount_path` (java, php: `mountPath`; csharp, go: `MountPath`) 字段中返回每个目录的完整路径（例如 `/mnt/memory/user-preferences/`）。                             |

worker 在认领会话时创建记忆存储目录。当它处理完该会话，或者当您[优雅地停止 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#stop-workers-gracefully) 时，它会删除这些目录。这只会删除本地副本，记忆仍保留在存储中。

技能可以包含智能体可直接运行的可执行文件。CLI 和 SDK worker 在解压技能包时会保留其中记录的可执行权限。如果您自行下载技能，请自行设置其可执行权限。

### worker 何时结束对会话的处理

在以下任一情况下，worker 会结束对会话的处理：

* 会话结束。
* 您[停止会话的工作项](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#stop-a-session-gracefully)。
* 会话保持空闲的时间达到 worker 的空闲限制。会话本身仍保持打开状态。当您随后向其发送新事件（例如用户消息）时，同一个 worker 或另一个 worker 会认领它。该 worker 会再次下载这些存储。

空闲限制默认为 60 秒。要更改它，对于 `ant` CLI worker，请使用 [`--max-idle`](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#cli-commands-and-flags)；对于 `EnvironmentWorker`，请使用 [`max_idle` (typescript: `maxIdleMs`; go: `MaxIdle`)](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#environment-worker)。[`cma-sandboxes` Helm chart](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#run-sandboxes-on-kubernetes) 则以 30 分钟的限制启动其 worker。要更改该限制，请更改该 chart 的 `sandbox.idleTimeout` 设置。

### 最终交付物

自托管沙箱没有 `/mnt/session/outputs` 目录。在 Anthropic 的云沙箱上，会话的系统提示会告诉智能体将最终交付物复制到该目录。在自托管环境中，系统提示会省略该指令。最终交付物会保留在智能体写入的任何位置，通常位于工作目录下，并且 worker 不会上传它们。

要将最终交付物收集到一个位置，请在智能体的系统提示中提供某个目录的完整路径。请选择 worker 的文件工具可以写入的目录：

* **工作目录内：** 例如，当工作目录为 `/workspace` 时选择 `/workspace/outputs`。
* **其他任何位置，仅限 SDK worker：** 将该目录添加到 `allowed_roots` (typescript: `allowedRoots`; go: `AllowedRoots`)。请参阅 [AgentToolContext 和智能体工具集](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#agent-tool-context-and-the-agent-toolset)。

## SDK 辅助工具

Python、TypeScript 和 Go SDK 提供了三个不同控制级别的辅助工具：

| 辅助工具                                                                                                                                                                                                                                                                 | 功能                              | 适用场景                             |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | -------------------------------- |
| [`EnvironmentWorker`](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#environment-worker)                                                                                                                                      | 端到端处理轮询、设置和工具运行。                | 大多数情况。                           |
| [`work.poller()` (typescript: `new WorkPoller()`; go: `environments.NewWorkPoller()`)](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#work-poller)                                                                            | 轮询工作队列，并将每个已认领的会话交给您。           | 您需要决定如何处理每个会话，例如启动沙箱而不是在进程内运行工具。 |
| [`client.beta.sessions.events.tool_runner()` (typescript: `client.beta.sessions.events.toolRunner()`; go: `client.Beta.Sessions.Events.NewToolRunner()`)](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#session-tool-runner) | 在给定会话 ID 和工具列表的情况下，为单个会话运行工具调用。 | 您已经认领了工作，只需要运行工具。                |

### EnvironmentWorker

| 方法                                                               | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `run()` (go: `Run()`)                                            | 无限期运行，在会话到达时接收它们。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`) | 处理一个已由轮询器认领的工作项。轮询器可以是 `ant beta:worker poll --on-work`，也可以是您自己的代码。当 worker 处理完该工作项的会话时，该方法返回。您可以传入 `work_id` (typescript: `workId`; go: `WorkID`)、`session_id` (typescript: `sessionId`; go: `SessionID`)、`environment_id` (typescript: `environmentId`; go: `EnvironmentID`) 和 `work_secret` (typescript: `workSecret`; go: `WorkSecret`)。对于您省略的每一项，该方法会读取对应的 [`ANTHROPIC_*` 变量](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#environment-variables)。 |

| 选项                                                                                     | 描述                                                                                                                                                                                                                                                                                                                                                       |
| -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tools` (go: `ToolsFunc`)                                                              | 一个工厂函数，接收会话的 `AgentToolContext` 并返回工具列表。默认为标准智能体工具集。                                                                                                                                                                                                                                                                                                     |
| `max_idle` (typescript: `maxIdleMs`; go: `MaxIdle`)                                    | 会话处于空闲状态时，worker 在结束对该会话的处理之前等待的时长。因 [`requires_action` 停止原因](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#respond-when-the-session-goes-idle)而处于空闲状态的时间不计入。默认为 60 秒。要等待 5 分钟，请将其设置为 `300` (typescript: `300_000`; go: `anthropic.Ptr(5 * time.Minute)`)。要关闭该限制，请将其设置为 `None` (typescript: `0`; go: `new(time.Duration)`)。 |
| `memory_sync_interval` (typescript: `memorySyncIntervalMs`; go: `MemorySyncInterval`)  | 会话运行期间，附加的记忆存储两次同步之间的最短时间。worker 会在每次调用在您沙箱中运行的工具之后检查该间隔，而不是依据计时器。默认为 15 秒。请参阅[同步间隔](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#sync-interval)。                                                                                                                                                                  |
| `memory_sync_deletions` (typescript: `memorySyncDeletions`; go: `MemorySyncDeletions`) | 智能体在本地删除的文件是否也从存储中删除。请参阅[删除](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#deletions)。                                                                                                                                                                                                                              |

`EnvironmentWorker` 会自动管理 `AgentToolContext` 和工具集。传入 `tools` (go: `ToolsFunc`) 工厂函数即可自定义工具列表：

<CodeGroup exclude="shell, csharp, java, php, ruby">
  <CodeGroupItem language="cURL">
    worker 是一个长时间运行的进程，而不是单个 HTTP 请求。要自定义工具列表，请使用 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    `ant` CLI worker 始终使用标准智能体工具集。要自定义工具列表，请使用 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
  </CodeGroupItem>

  ```python Python
  import asyncio
  import contextlib
  import os
  import signal
  from anthropic import AsyncAnthropic, beta_async_tool
  from anthropic.lib.environments import EnvironmentWorker
  from anthropic.lib.tools.agent_toolset import beta_bash_tool


  @beta_async_tool
  async def get_order_status(order_id: str) -> str:
      """Look up an order in the internal fulfillment system by order ID."""
      return f"Order {order_id}: shipped"


  async def main() -> None:
      environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
      environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
      async with AsyncAnthropic(auth_token=environment_key) as client:
          worker = EnvironmentWorker(
              client,
              environment_id=environment_id,
              environment_key=environment_key,
              # 列出智能体可以调用的所有工具。worker 不会响应其他任何工具。
              tools=lambda env: [beta_bash_tool(env), get_order_status],
          )
          task = asyncio.create_task(worker.run())
          loop = asyncio.get_running_loop()
          for signum in (signal.SIGINT, signal.SIGTERM):
              loop.add_signal_handler(signum, task.cancel)
          with contextlib.suppress(asyncio.CancelledError):
              await task


  asyncio.run(main())
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";
  import { EnvironmentWorker } from "@anthropic-ai/sdk/helpers/beta/environments";
  import { betaTool } from "@anthropic-ai/sdk/helpers/beta/json-schema";
  import { betaBashTool } from "@anthropic-ai/sdk/tools/agent-toolset/node";

  const getOrderStatus = betaTool({
    name: "get_order_status",
    description: "Look up an order in the internal fulfillment system by order ID.",
    inputSchema: {
      type: "object",
      properties: { order_id: { type: "string", description: "The order ID" } },
      required: ["order_id"]
    },
    run: async ({ order_id }) => `Order ${order_id}: shipped`
  });

  const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
  const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
  const client = new Anthropic({ authToken: environmentKey });
  const controller = new AbortController();
  process.once("SIGINT", () => controller.abort());
  process.once("SIGTERM", () => controller.abort());

  await new EnvironmentWorker({
    client,
    environmentId,
    environmentKey,
    signal: controller.signal,
    // 列出智能体可以调用的所有工具。worker 不会响应任何其他工具。
    tools: (ctx) => [betaBashTool(ctx), getOrderStatus]
  }).run();
  ```

  <CodeGroupItem language="C#">
    C# SDK 没有 `EnvironmentWorker`。要从您自己的代码中响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所示。
  </CodeGroupItem>

  ```go Go
  package main

  import (
  	"context"
  	"log"
  	"os"
  	"os/signal"
  	"syscall"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"github.com/anthropics/anthropic-sdk-go/v2/lib/environments"
  	"github.com/anthropics/anthropic-sdk-go/v2/option"
  	"github.com/anthropics/anthropic-sdk-go/v2/toolrunner"
  	"github.com/anthropics/anthropic-sdk-go/v2/tools/agenttoolset"
  )

  type orderStatusInput struct {
  	OrderID string `json:"order_id"`
  }

  func main() {
  	environmentKey := os.Getenv("ANTHROPIC_ENVIRONMENT_KEY")
  	environmentID := os.Getenv("ANTHROPIC_ENVIRONMENT_ID")

  	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
  	defer stop()

  	getOrderStatus := toolrunner.NewBetaTool(
  		"get_order_status",
  		"Look up an order in the internal fulfillment system by order ID.",
  		anthropic.BetaToolInputSchemaParam{
  			Properties: map[string]any{
  				"order_id": map[string]any{"type": "string", "description": "The order ID"},
  			},
  			Required: []string{"order_id"},
  		},
  		func(ctx context.Context, input orderStatusInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
  			return anthropic.BetaToolResultBlockParamContentUnion{
  				OfText: &anthropic.BetaTextBlockParam{Text: "Order " + input.OrderID + ": shipped"},
  			}, nil
  		},
  	)

  	client := anthropic.NewClient(option.WithAuthToken(environmentKey))

  	worker := environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{
  		EnvironmentID:  environmentID,
  		EnvironmentKey: environmentKey,
  		// 列出智能体可调用的所有工具。worker 不会响应其他任何工具。
  		ToolsFunc: func(env *agenttoolset.AgentToolContext) []anthropic.BetaTool {
  			return []anthropic.BetaTool{agenttoolset.BetaBashTool(env), getOrderStatus}
  		},
  	})
  	if err := worker.Run(ctx); err != nil {
  		log.Fatalf("worker: %v", err)
  	}
  }
  ```

  <CodeGroupItem language="Java">
    Java SDK 没有 `EnvironmentWorker`。要从您自己的代码中响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所示。
  </CodeGroupItem>

  <CodeGroupItem language="PHP">
    PHP SDK 没有 `EnvironmentWorker`。要从您自己的代码中响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所示。
  </CodeGroupItem>

  <CodeGroupItem language="Ruby">
    Ruby SDK 没有 `EnvironmentWorker`。要从您自己的代码中响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所示。
  </CodeGroupItem>
</CodeGroup>

### 工作轮询器

| 选项                                                                                   | 描述                                                                                                                                   |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| `drain` (go: `Drain`)                                                                | 队列为空时是否停止轮询，而不是等待新工作。                                                                                                                |
| `block_ms` (typescript: `blockMs`; go: `BlockMs`)                                    | 每次轮询在返回前等待工作到达的时长，以毫秒为单位。必须介于 1 到 999 之间。辅助工具会自动再次轮询。传入 `null` (python: `None`; go: `param.Null[int64]()`) 可进行非阻塞检查。默认为 999ms 长轮询。   |
| `reclaim_older_than_ms` (typescript: `reclaimOlderThanMs`; go: `ReclaimOlderThanMs`) | 重新认领那些已被认领但在此毫秒数内从未被确认的工作项。                                                                                                          |
| `auto_stop` (typescript: `autoStop`; go: `AutoStop`)                                 | 在您的循环体处理完每个工作项后，轮询器是否为其发送停止信号。默认为 `true` (python: `True`)。在此表之后列出的两种情况下，请将其设置为 `false` (python: `False`; go: `param.NewOpt(false)`)。 |

在以下两种情况下，请将 `auto_stop` (typescript: `autoStop`; go: `AutoStop`) 设置为 `false` (python: `False`; go: `param.NewOpt(false)`)：

1. 您的循环调用 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`)，该方法会自行发送停止信号。
2. 您的循环为该工作项启动沙箱后继续处理下一项。该沙箱中的 worker 必须在会话的工作结束时发送停止信号，就像 `ant beta:worker run` 和 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`) 所做的那样。

如需完整示例，请参阅[从 SDK 轮询器启动沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#launch-sandboxes-from-the-sdk-poller)。

### 会话工具运行器

`client.beta.sessions.events.tool_runner()` (typescript: `client.beta.sessions.events.toolRunner()`; go: `client.Beta.Sessions.Events.NewToolRunner()`) 以 `tools` (go: `Tools`) 的形式接收工具列表。要构建该列表，请自行设置 `AgentToolContext` 并调用 `beta_agent_toolset_20260401(env)` (typescript: `betaAgentToolset20260401(ctx)`; go: `agenttoolset.BetaAgentToolset20260401(env)`)：

<CodeGroup exclude="shell, csharp, java, php, ruby">
  <CodeGroupItem language="cURL">
    `AgentToolContext` 是 Python、TypeScript 和 Go SDK 中的辅助工具。cURL 没有等效功能。请使用这些 SDK 之一。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    `AgentToolContext` 是 Python、TypeScript 和 Go SDK 中的辅助工具。`ant` CLI 没有等效功能。请使用这些 SDK 之一。
  </CodeGroupItem>

  ```python Python
  from anthropic.lib.tools.agent_toolset import (
      AgentToolContext,
      beta_agent_toolset_20260401,
  )

  async with AgentToolContext(
      workdir="/workspace", client=client, session_id=work.data.id
  ) as env:
      # 技能下载到 /workspace/skills/<name>/
      tools = beta_agent_toolset_20260401(env)
  ```

  ```typescript TypeScript
  import {
    setupSkills,
    betaAgentToolset20260401
  } from "@anthropic-ai/sdk/tools/agent-toolset/node";

  const ctx = { workdir: "/workspace", client, sessionId: work.data.id };
  await setupSkills(ctx);
  const tools = betaAgentToolset20260401(ctx);
  ```

  <CodeGroupItem language="C#">
    C# SDK 没有 `AgentToolContext`。
  </CodeGroupItem>

  ```go Go
  env := &agenttoolset.AgentToolContext{Workdir: "/workspace"}
  if err := env.SetupSkills(ctx, client, work.Data.ID); err != nil {
  	panic(err)
  }
  // 技能下载到 /workspace/skills/<name>/
  tools := agenttoolset.BetaAgentToolset20260401(env)
  ```

  <CodeGroupItem language="Java">
    Java SDK 没有 `AgentToolContext`。
  </CodeGroupItem>

  <CodeGroupItem language="PHP">
    PHP SDK 没有 `AgentToolContext`。
  </CodeGroupItem>

  <CodeGroupItem language="Ruby">
    Ruby SDK 没有 `AgentToolContext`。
  </CodeGroupItem>
</CodeGroup>

### AgentToolContext 和智能体工具集

`AgentToolContext` 是工具调用的执行上下文。它定义了工作目录和路径策略，并且可以下载会话的技能。

| 选项                                                                   | 描述                                                        |
| -------------------------------------------------------------------- | --------------------------------------------------------- |
| `allowed_roots` (typescript: `allowedRoots`; go: `AllowedRoots`)     | 除工作目录之外，文件工具（`read`、`write`、`edit`、`glob`、`grep`）可以访问的目录。 |
| `read_only_roots` (typescript: `readOnlyRoots`; go: `ReadOnlyRoots`) | `write` 和 `edit` 会拒绝其下路径的目录。                              |

`EnvironmentWorker` 会自行将会话的记忆存储目录添加到 `allowed_roots` (typescript: `allowedRoots`; go: `AllowedRoots`)，并将以 `access: "read_only"` 附加的存储目录添加到 `read_only_roots` (typescript: `readOnlyRoots`; go: `ReadOnlyRoots`)。

这种限制仅是针对文件工具的防护措施，而不是沙箱。它不会约束 `bash`。

`beta_agent_toolset_20260401(env)` (typescript: `betaAgentToolset20260401(ctx)`; go: `agenttoolset.BetaAgentToolset20260401(env)`) 接收一个 `AgentToolContext`，并返回标准工具实现（`bash`、`read`、`write`、`edit`、`glob`、`grep`）。
