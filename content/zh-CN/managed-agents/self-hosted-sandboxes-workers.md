---
title: 部署自托管 worker
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers
description: 选择自托管沙箱 worker 如何认领工作以及会话在何处运行，然后运行常驻 worker 或通过 webhook 触发 worker。
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

[自托管沙箱快速入门](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#quickstart)运行一个 `ant` CLI 工作进程，该进程持续轮询，并在一个进程中运行每个会话。本页帮助您选择部署模式，然后介绍常驻 worker 和由 webhook 触发的 worker。

## 选择部署模式

部署 worker 时，您需要做出两个选择：worker 如何认领工作，以及每个会话在何处运行。

**worker 如何认领工作：**

* **常驻运行：** 一个长时间运行的进程持续轮询队列。该进程只发出出站 HTTPS 请求，因此您无需向 Anthropic 公开端点。
* **Webhook 触发：** 处理程序在 `session.status_run_started` 时被唤醒并开始轮询。这避免了空闲的轮询器，但需要一个 Anthropic 可以访问的 [webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks) 端点。

**每个会话在何处运行：**

* **进程内：** 认领会话的 worker 同时在一个共享工作目录中运行其工具调用。
* **每个会话一个沙箱：** 轮询器为每个认领的会话启动一个全新的沙箱。如需更强的隔离，请选择此方式：全新的文件系统、资源限制或按会话的网络控制。请参阅[每个会话运行一个沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session)，其中还介绍了适用于 Kubernetes 的 Helm chart。

CLI 和 SDK worker 支持不同的组合：

| 功能 | `ant` CLI | SDK（Python、TypeScript、Go） |
| ----- | ----- | ----- |
| 常驻轮询 | 是 | 是 |
| Webhook 触发 | 否 | 是 |
| [每个会话一个沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session) | 是 | 是 |
| [记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory) | 是，使用默认同步设置 | 是，同步可配置 |
| [自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools) | 否 | 是 |

有关每个 CLI 标志和 SDK 选项，请参阅[自托管 worker 参考](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference)。

如需更多控制，请直接调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)并实现您自己的 worker。这样，您的 worker 需要完成 SDK worker 所做的工作。除其他事项外，它需要确认每个工作项、发送心跳，并下载智能体的技能。它只运行会话的[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#see-how-each-call-was-evaluated)所允许的工具调用。它将每个结果作为 `user.tool_result` 或 `user.custom_tool_result` 事件返回。最后，它停止该工作项。您可以参考 [Python SDK](https://github.com/anthropics/anthropic-sdk-python/tree/main/src/anthropic/lib/environments)、[TypeScript SDK](https://github.com/anthropics/anthropic-sdk-typescript/tree/main/src/lib/environments) 或 [Go SDK](https://github.com/anthropics/anthropic-sdk-go/tree/main/lib/environments) 中的 worker 源代码。

## 运行常驻 worker

这些示例使用 `/workspace` 作为 worker 的工作目录。该目录必须在 worker 启动前存在。在新主机上，请创建该目录并为 worker 的用户授予写入权限，如[运行您的第一个会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)中的“启动 worker”步骤所示。

在 macOS 上，`mkdir` 无法创建 `/workspace`，因为根卷是只读的。请改为创建 `~/workspace`（如该步骤所示）。请使用 `echo ~/workspace` 输出的完整路径代替 `/workspace`，因为 SDK 不会展开 `~`。

启动一个持续轮询队列的 worker。它使用快速入门中的[环境密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)进行身份验证：

<CodeGroup exclude="shell:cURL, csharp, java, php, ruby">
  <CodeGroupItem language="cURL">
    worker 是一个长时间运行的进程，而不是单个 HTTP 请求。请改为运行 `ant` CLI worker：`ant beta:worker poll --workdir /workspace`。
  </CodeGroupItem>

  ```bash CLI
  ant beta:worker poll --workdir /workspace
  ```

  ```python Python
  import asyncio
  import contextlib
  import os
  import signal
  from anthropic import AsyncAnthropic
  from anthropic.lib.environments import EnvironmentWorker


  async def main() -> None:
      environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
      environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
      async with AsyncAnthropic(auth_token=environment_key) as client:
          worker = EnvironmentWorker(
              client,
              environment_id=environment_id,
              environment_key=environment_key,
              workdir="/workspace",
          )
          task = asyncio.create_task(worker.run())
          # 取消任务而非终止进程，可让 worker 停止其
          # 正在处理的工作项，并在退出前上传已更改的记忆文件。
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

  const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
  const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
  const client = new Anthropic({ authToken: environmentKey });
  const controller = new AbortController();
  // 在任一信号上中止可让 worker 上传已更改的内存文件并移除其
  // 存储目录，然后进程退出。
  process.once("SIGINT", () => controller.abort());
  process.once("SIGTERM", () => controller.abort());

  await new EnvironmentWorker({
    client,
    environmentId,
    environmentKey,
    workdir: "/workspace",
    signal: controller.signal
  }).run();
  ```

  <CodeGroupItem language="C#">
    C# SDK 没有 `EnvironmentWorker`。请改为运行 `ant` CLI worker：`ant beta:worker poll --workdir /workspace`。
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
  )

  func main() {
  	environmentKey := os.Getenv("ANTHROPIC_ENVIRONMENT_KEY")
  	environmentID := os.Getenv("ANTHROPIC_ENVIRONMENT_ID")

  	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
  	defer stop()

  	client := anthropic.NewClient(option.WithAuthToken(environmentKey))

  	worker := environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{
  		EnvironmentID:  environmentID,
  		EnvironmentKey: environmentKey,
  		Workdir:        "/workspace",
  	})
  	if err := worker.Run(ctx); err != nil {
  		log.Fatalf("worker: %v", err)
  	}
  }
  ```

  <CodeGroupItem language="Java">
    Java SDK 没有 `EnvironmentWorker`。请改为运行 `ant` CLI worker：`ant beta:worker poll --workdir /workspace`。
  </CodeGroupItem>

  <CodeGroupItem language="PHP">
    PHP SDK 没有 `EnvironmentWorker`。请改为运行 `ant` CLI worker：`ant beta:worker poll --workdir /workspace`。
  </CodeGroupItem>

  <CodeGroupItem language="Ruby">
    Ruby SDK 没有 `EnvironmentWorker`。请改为运行 `ant` CLI worker：`ant beta:worker poll --workdir /workspace`。
  </CodeGroupItem>
</CodeGroup>

## 通过 webhook 触发 worker

<Steps>
  <Step title="订阅会话 webhook">
    在 [Claude Console](https://platform.claude.com/settings/workspaces/default/webhooks) 中，定义一个监听 `session.status_run_started` 事件的 webhook 端点。详情请参阅[订阅 webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks)。
  </Step>

  <Step title="导出 webhook 签名密钥">
    除了[运行您的第一个会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)中的环境 ID 和密钥外，还要在处理程序主机上导出 webhook 签名密钥。处理程序使用它来验证传入的负载。

    ```bash
    export ANTHROPIC_WEBHOOK_SIGNING_KEY="whsec_..."
    ```
  </Step>

  <Step title="实现 webhook 处理程序">
    在 `session.status_run_started` 触发时启动工作进程。处理程序会清空队列，并将每个认领的工作项交给 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`)（在 Python、TypeScript 和 Go SDK 中）。该方法会下载技能、运行工具调用、回传结果，然后返回。

    这些示例使用 `/workspace` 作为工作目录。在启动处理程序之前，请在处理程序主机上创建该目录。确保处理程序的用户可以写入该目录，如[运行您的第一个会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)中的“启动 worker”步骤所示。

    <CodeGroup exclude="shell, csharp, java, php, ruby">
      <CodeGroupItem language="cURL">
        此处理程序所需的逻辑超出了单个 shell 命令的能力。请使用 Python、TypeScript 或 Go SDK 来运行它。
      </CodeGroupItem>

      <CodeGroupItem language="CLI">
        此处理程序所需的逻辑超出了单个 shell 命令的能力。请使用 Python、TypeScript 或 Go SDK 来运行它。
      </CodeGroupItem>

      <CodeGroupItem>
        要验证 webhook 签名，请安装 webhooks 附加组件：`pip install "anthropic[webhooks]"`。

        ```python Python
        import asyncio
        import os
        import anthropic
        import standardwebhooks  # installed by the anthropic[webhooks] extra

        environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
        environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
        client = anthropic.AsyncAnthropic(
            auth_token=environment_key,
        )
        # 由 shutdown() 取消，以便进行中的工作项能在进程退出前上传已更改的记忆文件并
        # 删除其存储目录。
        inflight: set[asyncio.Task[None]] = set()


        # 请在宿主的关闭钩子中 await 此方法，例如 ASGI lifespan 关闭阶段（FastAPI lifespan 中
        # `yield` 之后的代码），uvicorn 会在收到 SIGTERM 时运行它。uvicorn 会先让未完成的请求
        # 结束后再运行该钩子，因此请设置 --timeout-graceful-shutdown 以限制等待时间。
        async def shutdown() -> None:
            for task in inflight:
                task.cancel()
            await asyncio.gather(*inflight, return_exceptions=True)


        async def handle(raw: bytes, headers: dict[str, str]) -> tuple[dict[str, str], int]:
            try:
                event = client.beta.webhooks.unwrap(raw.decode(), headers=headers)
            except standardwebhooks.WebhookVerificationError:
                return {"error": "signature verification failed"}, 401
            if event.data.type != "session.status_run_started":
                return {"status": "ignored"}, 200
            task = asyncio.create_task(run_queued_work())
            inflight.add(task)
            task.add_done_callback(inflight.discard)
            try:
                # 已屏蔽取消：投递中断或超时不得取消该工作项；只有 shutdown() 会取消。
                await asyncio.shield(task)
            except asyncio.CancelledError:
                return {"status": "shutting down"}, 503
            return {"status": "ok"}, 200


        async def run_queued_work() -> None:
            async for work in client.beta.environments.work.poller(
                environment_id=environment_id,
                environment_key=environment_key,
                block_ms=None,
                reclaim_older_than_ms=2000,
                drain=True,
                auto_stop=False,
            ):
                await client.beta.environments.work.worker(workdir="/workspace").handle_item(
                    work_id=work.id,
                    environment_id=environment_id,
                    session_id=work.data.id,
                    environment_key=environment_key,
                    # worker 凭借每个会话的密钥来挂载该会话的记忆存储。
                    work_secret=work.secret,
                )
        ```
      </CodeGroupItem>

      ```typescript TypeScript
      import Anthropic from "@anthropic-ai/sdk";

      const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
      const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
      const client = new Anthropic({
        authToken: environmentKey
      });
      // 在宿主的 SIGTERM/SIGINT 处理程序中调用 shutdown.abort()，同时关闭服务器，
      // 然后在退出前等待进行中的 handle() 调用完成：abort 可让正在运行的工作项
      // 先上传已更改的内存文件并移除其存储目录。
      export const shutdown = new AbortController();

      export async function handle(req: Request): Promise<Response> {
        // 切勿确认其工作不会在此处运行的投递；返回 503 会让发送方重试。
        if (shutdown.signal.aborted) {
          return Response.json({ status: "shutting down" }, { status: 503 });
        }
        const body = await req.text();
        let event;
        try {
          event = client.beta.webhooks.unwrap(body, { headers: Object.fromEntries(req.headers) });
        } catch {
          return new Response("signature verification failed", { status: 401 });
        }
        if (event.data.type !== "session.status_run_started") {
          return Response.json({ status: "ignored" });
        }

        for await (const work of client.beta.environments.work.poller({
          environmentId,
          environmentKey,
          blockMs: null,
          reclaimOlderThanMs: 2000,
          drain: true,
          autoStop: false,
          signal: shutdown.signal
        })) {
          await client.beta.environments.work.worker({ workdir: "/workspace" }).handleItem({
            workId: work.id,
            environmentId,
            sessionId: work.data.id,
            environmentKey,
            // 每会话 secret 正是让 worker 能够挂载该会话内存存储的凭据。
            workSecret: work.secret ?? undefined,
            signal: shutdown.signal
          });
        }
        // 轮询器和 handleItem 在 abort 时会静默返回，因此被中断的排空操作会落到这里。
        if (shutdown.signal.aborted) {
          return Response.json({ status: "shutting down" }, { status: 503 });
        }
        return Response.json({ status: "ok" });
      }
      ```

      <CodeGroupItem language="C#">
        C# SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请按照[选择部署模式](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#choose-a-deployment-pattern)中的说明实现您自己的 worker。
      </CodeGroupItem>

      ```go Go
      package main

      import (
      	"context"
      	"encoding/json"
      	"errors"
      	"io"
      	"log/slog"
      	"net/http"
      	"os"
      	"os/signal"
      	"syscall"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	"github.com/anthropics/anthropic-sdk-go/v2/lib/environments"
      	"github.com/anthropics/anthropic-sdk-go/v2/option"
      	"github.com/anthropics/anthropic-sdk-go/v2/packages/param"
      )

      var (
      	environmentKey = os.Getenv("ANTHROPIC_ENVIRONMENT_KEY")
      	environmentID  = os.Getenv("ANTHROPIC_ENVIRONMENT_ID")
      	client         = anthropic.NewClient(
      		option.WithAuthToken(environmentKey),
      		option.WithWebhookKey(os.Getenv("ANTHROPIC_WEBHOOK_SIGNING_KEY")),
      	)
      	worker = environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{
      		Workdir: "/workspace",
      	})
      	// 在收到 SIGINT 或 SIGTERM 时取消（在 main 中设置），以便进行中的工作项
      	// 能在退出前上传已更改的记忆文件并删除其存储目录。
      	shutdown context.Context
      )

      func handle(w http.ResponseWriter, r *http.Request) {
      	body, err := io.ReadAll(r.Body)
      	if err != nil {
      		http.Error(w, "bad request", http.StatusBadRequest)
      		return
      	}
      	event, err := client.Beta.Webhooks.Unwrap(body, r.Header)
      	if err != nil {
      		http.Error(w, "signature verification failed", http.StatusUnauthorized)
      		return
      	}
      	if event.Data.Type != "session.status_run_started" {
      		json.NewEncoder(w).Encode(map[string]string{"status": "ignored"})
      		return
      	}

      	// Go SDK 未提供 RunOne 便捷方法：请使用 WorkPoller 取出待处理项，
      	// 并使用 HandleItem 逐个运行。
      	// 与 r.Context() 分离：会话的持续时间可能超过 webhook 投递超时。
      	// 进程级的关闭上下文仍会在 SIGTERM 时干净地结束该工作项。
      	ctx := shutdown
      	poller := environments.NewWorkPoller(ctx, client, environments.WorkPollerOptions{
      		EnvironmentID:      environmentID,
      		EnvironmentKey:     environmentKey,
      		BlockMs:            param.Null[int64](),
      		ReclaimOlderThanMs: param.NewOpt[int64](2000),
      		Drain:              true,
      		AutoStop:           param.NewOpt(false),
      	})
      	defer poller.Close()
      	for poller.Next() {
      		item := poller.Current()
      		if err := worker.HandleItem(ctx, environments.HandleItemOptions{
      			WorkID:         item.ID,
      			EnvironmentID:  item.EnvironmentID,
      			SessionID:      item.Data.ID,
      			EnvironmentKey: environmentKey,
      			// 每个会话的密钥使 worker 能够挂载该会话的记忆存储。
      			WorkSecret: item.Secret,
      		}); err != nil {
      			slog.Error("handle work item", "work_id", item.ID, "err", err)
      			http.Error(w, "internal error", http.StatusInternalServerError)
      			return
      		}
      	}
      	if err := poller.Err(); err != nil {
      		slog.Error("poll work queue", "err", err)
      		http.Error(w, "internal error", http.StatusInternalServerError)
      		return
      	}
      	json.NewEncoder(w).Encode(map[string]string{"status": "ok"})
      }

      func main() {
      	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
      	defer stop()
      	shutdown = ctx

      	server := &http.Server{Addr: ":8080"}
      	http.HandleFunc("POST /webhook", handle)
      	go func() {
      		if err := server.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
      			slog.Error("http server", "err", err)
      			os.Exit(1)
      		}
      	}()
      	// 收到信号时，停止接受投递，并仅在进行中的
      	// 处理程序（及其工作项的记忆清理）完成后才返回。
      	<-ctx.Done()
      	if err := server.Shutdown(context.Background()); err != nil {
      		slog.Error("http shutdown", "err", err)
      	}
      }
      ```

      <CodeGroupItem language="Java">
        Java SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请按照[选择部署模式](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#choose-a-deployment-pattern)中的说明实现您自己的 worker。
      </CodeGroupItem>

      <CodeGroupItem language="PHP">
        PHP SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请按照[选择部署模式](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#choose-a-deployment-pattern)中的说明实现您自己的 worker。
      </CodeGroupItem>

      <CodeGroupItem language="Ruby">
        Ruby SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请按照[选择部署模式](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#choose-a-deployment-pattern)中的说明实现您自己的 worker。
      </CodeGroupItem>
    </CodeGroup>

    在 Python、TypeScript 和 Go SDK 中，`handle_item()` (typescript: `handleItem()`; go: `HandleItem()`) 需要工作项的机密来挂载记忆存储。使用 `run()` (go: `Run()`) 轮询的 worker 已经持有该机密。此处理程序自行认领每个工作项，因此它通过 `work_secret` (typescript: `workSecret`; go: `WorkSecret`) 参数传递该机密。请参阅[转发工作项的机密](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#forward-the-work-items-secret)。
  </Step>
</Steps>

此处理程序在一个进程中运行每个认领的工作项，并且会同时运行来自不同 webhook 投递的工作项。这些会话共享同一个文件系统，包括 `/workspace` 和 `/mnt/memory`。如果您的会话附加了同一个记忆存储，请参阅[隔离共享同一存储的会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#isolate-sessions-that-share-a-store)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="每个会话运行一个沙箱" icon="cube" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session">
    使用 Kubernetes 上的 Helm chart、启动脚本或 SDK 轮询器启动沙箱。
  </Card>

  <Card title="监控自托管 worker 并排查故障" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations">
    读取队列深度，干净地停止会话和 worker，并修复常见故障。
  </Card>

  <Card title="安全模型" icon="lock" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-security">
    自托管沙箱环境的责任共担模型。
  </Card>
</CardGroup>

## 可用性

预构建的自托管 worker在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态 | 云平台 |
| ----- | ----- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目 | 零数据保留 | HIPAA |
| ----- | ----- | ----- |
| 状态 | 不具备资格 | 不具备资格 |
| 详细信息 | 不具备 ZDR 资格，自托管 Managed Agents 除外。会话是有状态的资源；对话记录会一直保留，直到您将其删除。 [了解更多](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#how-long-data-is-kept) | 不具备 HIPAA 资格。适用于所有 Managed Agents 子功能。 |

适用的端点：

* `/v1/agents`
* `/v1/sessions`
* `/v1/environments`
