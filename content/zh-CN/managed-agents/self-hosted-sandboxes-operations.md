---
title: 监控自托管 worker 并排查故障
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations
description: 读取队列深度，在不丢失工作的情况下停止会话和 worker，并修复常见的自托管沙箱故障。
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

本页中的队列统计和停止调用从您的监控或运维工具中运行，并使用您的组织 API 密钥进行身份验证。worker 辅助工具会处理认领和保活循环，因此您无需直接调用这些端点。

<Warning>
  队列统计和停止端点既接受您的组织 API 密钥，也接受环境密钥。请在 worker 主机之外使用您的组织 API 密钥调用它们。在 worker 主机上设置 `ANTHROPIC_API_KEY` 会将组织范围的凭据暴露给智能体的工具调用。
</Warning>

## 读取队列深度

`GET /v1/environments/{environment_id}/work/stats` (python, typescript, ruby: `client.beta.environments.work.stats()`; go, csharp: `client.Beta.Environments.Work.Stats()`; java: `client.beta().environments().work().stats()`; php: `$client->beta->environments->work->stats()`; cli: `ant beta:environments:work stats`) 返回某个环境的队列状态：

| 字段                                                                             | 含义                                                                                                                            | 用途                                 |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| `depth` (csharp, go: `Depth`)                                                  | 等待被认领的项目。                                                                                                                     | 扩缩您的 worker 集群，或针对积压发出告警。          |
| `pending` (csharp, go: `Pending`)                                              | 已被 worker 认领但尚未确认的项目。worker 辅助工具会在处理每个项目之前先确认它，因此在正常运行时该值保持在接近零的水平。                                                           | 检测在认领和确认之间停滞的 worker：针对持续非零的值发出告警。 |
| `oldest_queued_at` (java, php: `oldestQueuedAt`; csharp, go: `OldestQueuedAt`) | 队列中仍存在的最旧项目的时间戳，该项目要么正在等待被认领，要么已被认领但尚未确认。当不存在此类项目时为 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `""`)。 | 查看最旧的项目已等待了多长时间。                   |
| `workers_polling` (java, php: `workersPolling`; csharp, go: `WorkersPolling`)  | 在过去 30 秒内进行过轮询的 worker。                                                                                                       | 针对存活状态发出告警。                        |

<CodeGroup>
  ```bash cURL
  curl -sS "https://api.anthropic.com/v1/environments/$ANTHROPIC_ENVIRONMENT_ID/work/stats" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:environments:work stats --environment-id "$ANTHROPIC_ENVIRONMENT_ID"
  ```

  ```python Python
  import os

  import anthropic

  client = anthropic.Anthropic()

  stats = client.beta.environments.work.stats(os.environ["ANTHROPIC_ENVIRONMENT_ID"])
  print(f"depth={stats.depth} pending={stats.pending}")
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const stats = await client.beta.environments.work.stats(process.env.ANTHROPIC_ENVIRONMENT_ID!);

  console.log(`depth=${stats.depth} pending=${stats.pending}`);
  ```

  ```csharp C#
  using Anthropic;

  var client = new AnthropicClient();

  var environmentId = Environment.GetEnvironmentVariable("ANTHROPIC_ENVIRONMENT_ID")!;

  var stats = await client.Beta.Environments.Work.Stats(environmentId);

  Console.WriteLine($"depth={stats.Depth} pending={stats.Pending}");
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
  	environmentID := os.Getenv("ANTHROPIC_ENVIRONMENT_ID")

  	stats, err := client.Beta.Environments.Work.Stats(
  		context.Background(),
  		environmentID,
  		anthropic.BetaEnvironmentWorkStatsParams{},
  	)
  	if err != nil {
  		panic(err)
  	}

  	fmt.Printf("depth=%d pending=%d\n", stats.Depth, stats.Pending)
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.beta.environments.work.BetaSelfHostedWorkQueueStats;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      BetaSelfHostedWorkQueueStats stats = client.beta()
          .environments()
          .work()
          .stats(System.getenv("ANTHROPIC_ENVIRONMENT_ID"));

      IO.println("depth=" + stats.depth() + " pending=" + stats.pending());
  }
  ```

  ```php PHP
  <?php

  use Anthropic\Client;

  $client = new Client();

  $stats = $client->beta->environments->work->stats(getenv('ANTHROPIC_ENVIRONMENT_ID'));

  printf("depth=%d pending=%d\n", $stats->depth, $stats->pending);
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::Client.new

  stats = client.beta.environments.work.stats(ENV.fetch("ANTHROPIC_ENVIRONMENT_ID"))

  puts "depth=#{stats.depth} pending=#{stats.pending}"
  ```
</CodeGroup>

```json
{
  "type": "work_queue_stats",
  "depth": 0,
  "pending": 0,
  "oldest_queued_at": null,
  "workers_polling": 0
}
```

## 优雅地停止会话

使用 `POST /v1/environments/{environment_id}/work/{work_id}/stop` (python, typescript, ruby: `client.beta.environments.work.stop()`; go, csharp: `client.Beta.Environments.Work.Stop()`; java: `client.beta().environments().work().stop()`; php: `$client->beta->environments->work->stop()`; cli: `ant beta:environments:work stop`) 请求处理特定会话的 worker 关闭该会话。

默认情况下，工作项会进入 `stopping` 状态。worker 会在下一次租约心跳（lease heartbeat）时检测到这一点，取消该会话正在进行的工具调用，并确认关闭。随后工作项变为 `stopped`。而 Python SDK worker 会先让正在进行的工具调用完成，这最多可能需要 2.5 分钟。

传入 `force: true` (python: `force=True`; csharp: `Force = true`; go: `BetaSelfHostedWorkStopRequest: anthropic.BetaSelfHostedWorkStopRequestParam{Force: anthropic.Bool(true)}`; java: `.force(true)`; cli: `--force`) 可立即将工作项标记为 `stopped`，而无需等待 worker 的确认。强制停止并不会让正在进行的工具调用更早结束，因为 worker 仍然是在下一次心跳时才察觉到停止请求。

由于这些调用是从您的运维工具而非 worker 主机运行的，因此不会自动设置 `ANTHROPIC_WORK_ID`。在运行以下示例之前，请将其设置为目标工作项的 ID。要查找工作项的 ID，请使用环境密钥调用[列出工作项](https://platform.claude.com/docs/zh-CN/api/beta/environments/work/list)，并以 `Authorization: Bearer $ANTHROPIC_ENVIRONMENT_KEY` 的形式发送该密钥。该端点会拒绝您的组织 API 密钥。

<CodeGroup>
  ```bash cURL
  curl -sS "https://api.anthropic.com/v1/environments/$ANTHROPIC_ENVIRONMENT_ID/work/$ANTHROPIC_WORK_ID/stop" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{}'
  ```

  ```bash CLI
  ant beta:environments:work stop \
    --environment-id "$ANTHROPIC_ENVIRONMENT_ID" \
    --work-id "$ANTHROPIC_WORK_ID"
  ```

  ```python Python
  import os

  import anthropic

  client = anthropic.Anthropic()

  work = client.beta.environments.work.stop(
      os.environ["ANTHROPIC_WORK_ID"],
      environment_id=os.environ["ANTHROPIC_ENVIRONMENT_ID"],
  )
  print(work.state)
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const work = await client.beta.environments.work.stop(process.env.ANTHROPIC_WORK_ID!, {
    environment_id: process.env.ANTHROPIC_ENVIRONMENT_ID!
  });

  console.log(work.state);
  ```

  ```csharp C#
  using Anthropic;

  var client = new AnthropicClient();

  var work = await client.Beta.Environments.Work.Stop(
      Environment.GetEnvironmentVariable("ANTHROPIC_WORK_ID")!,
      new()
      {
          EnvironmentID = Environment.GetEnvironmentVariable("ANTHROPIC_ENVIRONMENT_ID")!
      }
  );

  Console.WriteLine(work.State);
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

  	work, err := client.Beta.Environments.Work.Stop(
  		context.Background(),
  		os.Getenv("ANTHROPIC_WORK_ID"),
  		anthropic.BetaEnvironmentWorkStopParams{
  			EnvironmentID: os.Getenv("ANTHROPIC_ENVIRONMENT_ID"),
  		},
  	)
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println(work.State)
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.beta.environments.work.BetaSelfHostedWork;
  import com.anthropic.models.beta.environments.work.BetaSelfHostedWorkStopRequest;
  import com.anthropic.models.beta.environments.work.WorkStopParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      BetaSelfHostedWork work = client.beta().environments().work().stop(
          WorkStopParams.builder()
              .environmentId(System.getenv("ANTHROPIC_ENVIRONMENT_ID"))
              .workId(System.getenv("ANTHROPIC_WORK_ID"))
              .betaSelfHostedWorkStopRequest(BetaSelfHostedWorkStopRequest.builder().build())
              .build()
      );

      IO.println(work.state());
  }
  ```

  ```php PHP
  <?php

  use Anthropic\Client;

  $client = new Client();

  $work = $client->beta->environments->work->stop(
      getenv('ANTHROPIC_WORK_ID'),
      environmentID: getenv('ANTHROPIC_ENVIRONMENT_ID'),
  );

  echo $work->state . "\n";
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::Client.new

  work = client.beta.environments.work.stop(
    ENV.fetch("ANTHROPIC_WORK_ID"),
    environment_id: ENV.fetch("ANTHROPIC_ENVIRONMENT_ID")
  )

  puts work.state
  ```
</CodeGroup>

## 优雅地停止 worker

在会话运行期间被取消的 worker 会在退出前关闭该会话，就像您[停止会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#stop-a-session-gracefully)时一样。如果该会话附加了[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)，worker 会跳过最终同步，但仍会上传已更改的文件并删除存储目录。

被强制终止的进程不会执行任何清理。要干净地停止 worker：

1. **确保 SIGTERM 和 SIGINT 会取消 worker。** 具体做法取决于 worker：

   | Worker                      | 操作                                                                   |
   | --------------------------- | -------------------------------------------------------------------- |
   | `ant` CLI                   | 无需任何操作。CLI 会自行处理这两个信号：它会取消任何正在进行的工具调用，发布其错误结果，并将工作项标记为 `stopped`。    |
   | 作为独立进程的 SDK worker          | `EnvironmentWorker` 不会安装任何信号处理程序。请像独立 worker 示例那样，从信号处理程序中取消 worker。 |
   | 位于 webhook 服务器内的 SDK worker | 请像 webhook 示例那样，从服务器自身的关闭钩子中取消 worker。worker 不得接管服务器的信号。             |

2. **使用 SIGTERM 停止 worker，并在任何强制终止之前至少留出 30 秒（Python SDK worker 为 3 分钟）。** 将已更改的文件上传到记忆存储最多可能需要 30 秒，而 Python SDK worker 会先让正在进行的工具调用完成。默认情况下，Docker 会在发送停止信号 10 秒后发送 SIGKILL。可以通过 `docker run` 上的 `--stop-timeout`，或通过您的编排器的终止宽限期来提高该限制。

如果 worker 在执行清理之前被终止，任何尚未同步的记忆编辑都会丢失。在长期运行的主机上，还需要在下一个附加该存储的会话之前，删除 `/mnt/memory/` 下残留的存储目录。仅服务一个会话随后即被丢弃的沙箱无需清理。

## 故障排除

### worker 无法连接

如果 `workers_polling` (java, php: `workersPolling`; csharp, go: `WorkersPolling`) 一直为 0，说明 worker 未能访问到队列。请确认 worker 主机上已设置 `ANTHROPIC_ENVIRONMENT_KEY` 和 `ANTHROPIC_ENVIRONMENT_ID`。

### 会话一直处于排队状态

没有 worker 在认领工作。排队中的会话会一直等待，而不会失败。请在[读取队列深度](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#read-queue-depth)中检查 `workers_polling` (java, php: `workersPolling`; csharp, go: `WorkersPolling`) 和 `depth` (csharp, go: `Depth`)。

### 记忆存储挂载失败

worker 会将挂载失败和后台同步失败记录到日志中，而不是报告给会话。只有只读拒绝会以工具错误的形式传达给智能体（请参阅[只读存储和冲突](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#read-only-stores-and-conflicts)）。

如果 worker 在认领会话时无法挂载记忆存储，它会使该工作项失败。会话不会发出错误事件，并保持空闲状态。

| 症状                                                                                                             | 原因                                                                                     | 解决方法                                                                                                                                                                                                                                                                                      |
| -------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| worker 日志中包含 `the work item carried no sessions token`（在 Go 中为 `ErrSessionMemoryNoToken` 错误），且工作项失败。           | 工作项的每会话 `secret` (csharp, go: `Secret`) 未到达 worker。要么是您的代码没有转发它，要么是您的组织未启用自托管沙箱上的记忆存储。 | [转发工作项的机密](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#forward-the-work-items-secret)。如果 worker 在同一进程中轮询并运行会话，但仍记录此日志，请联系支持团队。                                                                                                                    |
| worker 日志中包含 `something already exists at the memory store's path`。                                            | 之前会话残留的目录，通常是其 worker 在执行清理之前就被终止的会话。                                                  | 删除日志行中指明的残留目录。其中尚未同步的编辑将会丢失。                                                                                                                                                                                                                                                              |
| worker 日志中包含 `cannot create the memory store's folder` 和 `the worker host must make this mount path writable`。 | 运行 worker 的用户无法在 `/mnt/memory` 下创建目录。                                                  | 创建 `/mnt/memory` 并将其 `chown` 给该用户。请参阅[准备主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-the-host)。在 macOS 上，请改为按照[准备 macOS 主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-a-macos-host)操作。 |
| 在 worker 认领会话后不久，会话处于 `idle` 状态，停止原因为 `requires_action`，且没有错误事件。                                               | 由于上述原因之一，worker 无法挂载记忆存储，因此使该工作项失败。                                                    | 在主机上修复原因，然后发送 [`user.interrupt`](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#interrupt-the-agent) 事件。会话的工作会重新排队，下一个认领它的 worker 会重试挂载。                                                                                                                      |

### 自定义工具调用始终没有返回

如果会话以 `requires_action` 停止原因处于暂停状态，说明没有 worker 或客户端为该工具提供服务。请参阅[提供自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools#serve-a-custom-tool)。

### 封装的 MCP 工具调用挂起

如果 MCP 客户端没有设置超时，对[封装的 MCP 服务器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools#wrap-an-mcp-server-as-custom-tools)的挂起调用只有在兜底机制触发时才会变成错误工具结果：

| SDK        | 兜底机制                   | 触发时间     |
| ---------- | ---------------------- | -------- |
| Python     | worker 自身的工具调用限制       | 约 2.5 分钟 |
| TypeScript | MCP SDK 的默认请求超时        | 约一分钟     |
| Go         | worker 会取消超出其默认限制的工具调用 | 120 秒    |
