---
title: 部署自托管 worker
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers
description: 选择自托管沙箱 worker 如何认领工作以及会话在何处运行：常驻运行或由 webhook 触发，在单个进程中运行或每个会话一个沙箱。
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
---

[快速入门](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#quickstart)运行一个 `ant` CLI 工作进程，该进程持续轮询，并在一个进程中运行每个会话。本页介绍运行工作进程的其他方式以及如何在它们之间进行选择。

## 选择部署模式

部署 worker 时，您需要做出两个选择：worker 如何认领工作，以及每个会话在何处运行。

**worker 如何认领工作：**

* **常驻运行：** 一个长时间运行的进程持续轮询队列，只需要出站 HTTPS。这是最简单的设置。
* **Webhook 触发：** 处理程序在 `session.status_run_started` 时被唤醒并开始轮询。这避免了空闲的轮询器，但需要一个 Anthropic 可以访问的 [webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks) 端点。

**每个会话在何处运行：**

* **进程内：** 认领会话的 worker 同时在一个共享工作目录中运行其工具调用。
* **每个会话一个沙箱：** 轮询器为每个认领的会话启动一个全新的沙箱。如需更强的隔离，请选择此方式：全新的文件系统、资源限制或按会话的网络控制。

CLI 和 SDK worker 支持不同的组合：

| 功能                                                                                                | `ant` CLI  | SDK（Python、TypeScript、Go） |
| ------------------------------------------------------------------------------------------------- | ---------- | ------------------------- |
| 常驻轮询                                                                                              | 是          | 是                         |
| Webhook 触发                                                                                        | 否          | 是                         |
| 每个会话一个沙箱                                                                                          | 是          | 是                         |
| [记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)        | 是，使用默认同步设置 | 是，同步可配置                   |
| [自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools) | 否          | 是                         |

有关每个 CLI 标志和 SDK 选项，请参阅[自托管 worker 参考](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference)。如需更多控制，请直接调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)并实现您自己的 worker。

## 运行常驻 worker

启动一个持续轮询队列的 worker。它使用快速入门中的[环境密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)进行身份验证：

<CodeGroup exclude="shell:cURL, csharp, java, php, ruby">
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
          # 取消任务（而非终止进程）可让 worker 停止其
          # 正在处理的工作项，并在退出前上传已更改的内存文件。
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
    在 [Claude Console](https://platform.claude.com/settings/workspaces/default/webhooks) 中，定义一个监听 `session.status_run_started` 事件的 webhook 端点。详情请参阅 [Webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks)。
  </Step>

  <Step title="导出 webhook 签名密钥">
    除了[运行您的第一个会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)中的环境 ID 和密钥外，还要在处理程序主机上导出 webhook 签名密钥。处理程序使用它来验证传入的负载。

    ```bash
    export ANTHROPIC_WEBHOOK_SIGNING_KEY="whsec_..."
    ```
  </Step>

  <Step title="实现 webhook 处理程序">
    在 `session.status_run_started` 触发时启动工作进程。处理程序会清空队列，并将每个认领的工作项交给 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`)，后者会下载技能、运行工具调用、回传结果，然后返回。

    <CodeGroup exclude="shell, csharp, java, php, ruby">
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
        # 由 shutdown() 取消，以便正在进行的工作项能够上传已更改的内存文件，
        # 并在进程退出前移除其存储目录。
        inflight: set[asyncio.Task[None]] = set()


        # 请在宿主的关闭钩子中 await 此方法，例如 ASGI lifespan 关闭阶段（FastAPI lifespan 中
        # `yield` 之后的代码），uvicorn 会在收到 SIGTERM 时运行它。uvicorn 会让未完成的请求
        # 在该钩子运行前结束，因此请设置 --timeout-graceful-shutdown 以限制等待时间。
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
                # 已屏蔽（shield）：投递被丢弃或超时不得取消该工作项；由 shutdown() 负责取消。
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
                    # 每会话密钥使 worker 能够挂载该会话的内存存储。
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
        C# SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
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
      	// 在 SIGINT 或 SIGTERM 时取消（在 main 中设置），以便进行中的工作项能够
      	// 在退出前上传已更改的内存文件并移除其存储目录。
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

      	// Go SDK 不提供 RunOne 便捷方法：使用 WorkPoller 排空待处理项，
      	// 并使用 HandleItem 逐个运行。
      	// 与 r.Context() 分离：会话的存活时间可能超过 webhook 投递超时。
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
      			// 每会话密钥使 worker 能够挂载该会话的内存存储。
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
      	// 收到信号时，停止接受投递，并且仅在进行中的
      	// 处理程序（及其工作项的内存清理）完成后才返回。
      	<-ctx.Done()
      	if err := server.Shutdown(context.Background()); err != nil {
      		slog.Error("http shutdown", "err", err)
      	}
      }
      ```

      <CodeGroupItem language="Java">
        Java SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
      </CodeGroupItem>

      <CodeGroupItem language="PHP">
        PHP SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
      </CodeGroupItem>

      <CodeGroupItem language="Ruby">
        Ruby SDK 没有 `EnvironmentWorker`。要从您自己的代码处理工作项，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
      </CodeGroupItem>
    </CodeGroup>

    由于处理程序自行认领工作，它必须[转发工作项的机密](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#forward-the-work-items-secret)，如此处的 `work_secret` (typescript: `workSecret`; go: `WorkSecret`) 参数所示。
  </Step>
</Steps>

此处理程序在一台主机上的一个进程中运行每个认领的工作项。如果您的会话附加了同一个记忆存储，请参阅[隔离共享同一存储的会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#isolate-sessions-that-share-a-store)。

## 每个会话运行一个沙箱

主机上的轮询器认领工作，并为每个工作项调用一次您的脚本。该脚本为这一个会话启动一个沙箱。

<Steps>
  <Step title="构建沙箱镜像">
    安装 `ant` 并将 `ant beta:worker run` 设置为入口点。沙箱启动时，它会从环境变量中读取会话详情，处理该会话，然后退出。基础镜像必须提供 `/bin/bash`；`curl` 仅在构建时使用。

    ```dockerfile
    FROM your-base-image
    ARG ANT_VERSION=2.9.0
    ARG TARGETARCH
    RUN ARCH=$([ "$TARGETARCH" = "arm64" ] && echo arm64 || echo amd64) && \
        curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${ANT_VERSION}/ant_${ANT_VERSION}_linux_${ARCH}.tar.gz" \
          | tar -xz -C /usr/local/bin ant
    WORKDIR /workspace
    VOLUME /workspace
    ENTRYPOINT ["ant", "beta:worker", "run"]
    ```
  </Step>

  <Step title="编写启动脚本">
    该脚本将会话详情转发到一个全新的沙箱中。它要求轮询器主机上安装 `jq`。

    ```bash
    #!/bin/bash
    # spawn.sh：每个已认领的工作项调用一次
    # 已认领的工作项以 JSON 形式通过 stdin 传入。
    ANTHROPIC_WORK_SECRET="$(jq -r '.secret // empty')"
    export ANTHROPIC_WORK_SECRET
    mkdir -p "/host/outputs/$ANTHROPIC_SESSION_ID"
    exec docker run --rm \
      -e ANTHROPIC_SESSION_ID -e ANTHROPIC_ENVIRONMENT_KEY \
      -e ANTHROPIC_WORK_ID -e ANTHROPIC_ENVIRONMENT_ID -e ANTHROPIC_BASE_URL \
      -e ANTHROPIC_WORK_SECRET \
      -v "/host/outputs/$ANTHROPIC_SESSION_ID":/workspace \
      your-image
    ```

    轮询器会设置脚本所转发的 `ANTHROPIC_*` 变量，但机密除外。请参阅[环境变量](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#environment-variables)。

    `/host/outputs` 是您选择的主机目录。将其挂载到 `/workspace` 后，您可以在沙箱退出后取回会话的交付成果。该挂载还会包含下载的 `skills/` 目录树以及所有中间文件。
  </Step>

  <Step title="启动轮询器">
    ```bash
    ant beta:worker poll --on-work ./spawn.sh
    ```
  </Step>
</Steps>

### 转发工作项的机密

每个认领的工作项都可以携带一个由 Anthropic 签发的按会话 `secret` (csharp, go: `Secret`)。运行该会话的 worker 需要它来挂载[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)。

在一个进程中认领并运行会话的 worker（不带 `--on-work` 的 `ant beta:worker poll`，或使用 `run()` (go: `Run()`) 的 `EnvironmentWorker`）会自行传递该机密。当您自己的代码位于认领和 worker 之间时，需要由您来转发它：

| 您认领工作的方式                                                   | 机密的到达形式                              | 传递给 worker 的方式                                                                                                                                                    |
| ---------------------------------------------------------- | ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ant beta:worker poll --on-work`                           | 脚本标准输入中工作项 JSON 的 `secret` 字段        | 沙箱环境中的 `ANTHROPIC_WORK_SECRET`                                                                                                                                    |
| SDK 的 `work.poller()` (go: `environments.NewWorkPoller()`) | 每个认领的工作项的 `secret` (go: `Secret`) 字段 | 沙箱环境中的 `ANTHROPIC_WORK_SECRET`，或 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`) 的 `work_secret` (typescript: `workSecret`; go: `WorkSecret`) 参数 |

仅将机密传入为该会话服务的沙箱，并且切勿记录它。有关它与环境密钥的关系，请参阅[安全模型](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-security)。

### 从 SDK 轮询器启动沙箱

要从您自己的代码而不是 `ant beta:worker poll --on-work` 认领工作，请使用 `work.poller()` (typescript: `new WorkPoller()`; go: `environments.NewWorkPoller()`)。它会轮询队列并将每个认领的会话交给您，由您来启动沙箱：

<CodeGroup exclude="shell, csharp, java, php, ruby">
  <CodeGroupItem language="cURL">
    在循环中认领工作并为每个会话启动沙箱所需的逻辑超出了单个 shell 命令的能力。在 shell 中，请配合启动脚本运行 `ant beta:worker poll --on-work`，如[每个会话运行一个沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#run-one-sandbox-per-session)中所示。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    要从 shell 认领工作并为每个会话启动沙箱，请配合启动脚本运行 `ant beta:worker poll --on-work`，如[每个会话运行一个沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#run-one-sandbox-per-session)中所示。
  </CodeGroupItem>

  ```python Python
  import asyncio
  import os

  from anthropic import AsyncAnthropic
  from anthropic.types.beta.environments import BetaSelfHostedWork

  SANDBOX_ENV = (
      "ANTHROPIC_ENVIRONMENT_ID",
      "ANTHROPIC_ENVIRONMENT_KEY",
      "ANTHROPIC_WORK_ID",
      "ANTHROPIC_SESSION_ID",
      "ANTHROPIC_WORK_SECRET",
      "ANTHROPIC_BASE_URL",  # forwarded only when set on this host
  )


  async def launch_container(work: BetaSelfHostedWork) -> None:
      print(f"claimed session {work.data.id}")
      # 将 `docker run` 替换为您自己的沙箱启动器。转发环境
      # 密钥（切勿转发您的 API 密钥）以及工作项的每会话密钥：其中的 worker
      # 需要该密钥来挂载会话的记忆存储。
      env = os.environ | {
          "ANTHROPIC_WORK_ID": work.id,
          "ANTHROPIC_SESSION_ID": work.data.id,
          "ANTHROPIC_WORK_SECRET": work.secret or "",
      }
      forward = [arg for name in SANDBOX_ENV for arg in ("-e", name)]
      launcher = await asyncio.create_subprocess_exec(
          "docker", "run", "--rm", "--detach", *forward, "your-image", env=env
      )
      await launcher.wait()


  async def main() -> None:
      environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
      environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
      async with AsyncAnthropic(auth_token=environment_key) as client:
          async for work in client.beta.environments.work.poller(
              environment_id=environment_id,
              environment_key=environment_key,
              auto_stop=False,  # the launched sandbox owns the stop call
          ):
              await launch_container(work)


  asyncio.run(main())
  ```

  ```typescript TypeScript
  import { spawn } from "node:child_process";
  import { once } from "node:events";
  import Anthropic from "@anthropic-ai/sdk";
  import { WorkPoller } from "@anthropic-ai/sdk/helpers/beta/environments";
  import type { BetaSelfHostedWork } from "@anthropic-ai/sdk/resources/beta/environments";

  const SANDBOX_ENV = [
    "ANTHROPIC_ENVIRONMENT_ID",
    "ANTHROPIC_ENVIRONMENT_KEY",
    "ANTHROPIC_WORK_ID",
    "ANTHROPIC_SESSION_ID",
    "ANTHROPIC_WORK_SECRET",
    "ANTHROPIC_BASE_URL" // forwarded only when set on this host
  ];

  const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
  const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
  const client = new Anthropic({ authToken: environmentKey });

  async function launchContainer(work: BetaSelfHostedWork): Promise<void> {
    console.log(`claimed session ${work.data.id}`);
    // 将 `docker run` 替换为您自己的沙箱启动器。转发环境
    // 密钥（切勿转发您的 API 密钥）以及工作项的每会话密钥：其中的 worker
    // 需要该密钥来挂载会话的记忆存储。
    const env = {
      ...process.env,
      ANTHROPIC_WORK_ID: work.id,
      ANTHROPIC_SESSION_ID: work.data.id,
      ANTHROPIC_WORK_SECRET: work.secret ?? ""
    };
    const forward = SANDBOX_ENV.flatMap((name) => ["-e", name]);
    const launcher = spawn("docker", ["run", "--rm", "--detach", ...forward, "your-image"], {
      env,
      stdio: "inherit"
    });
    await once(launcher, "close");
  }

  const poller = new WorkPoller({
    client,
    environmentId,
    environmentKey,
    autoStop: false // the launched sandbox owns the stop call
  });

  for await (const work of poller) {
    await launchContainer(work);
  }
  ```

  <CodeGroupItem language="C#">
    C# SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
  </CodeGroupItem>

  ```go Go
  package main

  import (
  	"context"
  	"fmt"
  	"log"
  	"os"
  	"os/exec"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"github.com/anthropics/anthropic-sdk-go/v2/lib/environments"
  	"github.com/anthropics/anthropic-sdk-go/v2/option"
  	"github.com/anthropics/anthropic-sdk-go/v2/packages/param"
  )

  var sandboxEnv = []string{
  	"ANTHROPIC_ENVIRONMENT_ID",
  	"ANTHROPIC_ENVIRONMENT_KEY",
  	"ANTHROPIC_WORK_ID",
  	"ANTHROPIC_SESSION_ID",
  	"ANTHROPIC_WORK_SECRET",
  	"ANTHROPIC_BASE_URL", // forwarded only when set on this host
  }

  func launchContainer(ctx context.Context, work *anthropic.BetaSelfHostedWork) error {
  	fmt.Printf("claimed session %s\n", work.Data.ID)
  	// 将 `docker run` 替换为您自己的沙箱启动器。转发环境
  	// 密钥（切勿转发您的 API 密钥）以及工作项的每会话机密：沙箱内的 worker
  	// 需要该机密来挂载会话的记忆存储。
  	args := []string{"run", "--rm", "--detach"}
  	for _, name := range sandboxEnv {
  		args = append(args, "-e", name)
  	}
  	launcher := exec.CommandContext(ctx, "docker", append(args, "your-image")...)
  	launcher.Env = append(os.Environ(),
  		"ANTHROPIC_WORK_ID="+work.ID,
  		"ANTHROPIC_SESSION_ID="+work.Data.ID,
  		"ANTHROPIC_WORK_SECRET="+work.Secret,
  	)
  	launcher.Stdout, launcher.Stderr = os.Stdout, os.Stderr
  	return launcher.Run()
  }

  func main() {
  	environmentID := os.Getenv("ANTHROPIC_ENVIRONMENT_ID")
  	environmentKey := os.Getenv("ANTHROPIC_ENVIRONMENT_KEY")

  	client := anthropic.NewClient(option.WithAuthToken(environmentKey))

  	ctx := context.Background()

  	poller := environments.NewWorkPoller(ctx, client, environments.WorkPollerOptions{
  		EnvironmentID:  environmentID,
  		EnvironmentKey: environmentKey,
  		AutoStop:       param.NewOpt(false), // the launched sandbox owns the stop call
  	})
  	defer poller.Close()

  	for work, err := range poller.All() {
  		if err != nil {
  			log.Fatal(err)
  		}
  		if err := launchContainer(ctx, work); err != nil {
  			log.Fatal(err)
  		}
  	}
  }
  ```

  <CodeGroupItem language="Java">
    Java SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
  </CodeGroupItem>

  <CodeGroupItem language="PHP">
    PHP SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
  </CodeGroupItem>

  <CodeGroupItem language="Ruby">
    Ruby SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。
  </CodeGroupItem>
</CodeGroup>

### 在沙箱内运行 SDK worker

当沙箱必须[提供自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools)或使用[非默认的记忆同步设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#configure-sync)时，请将 `ant beta:worker run` 入口点替换为 SDK 入口点。该入口点构造 `EnvironmentWorker` 并调用 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`)，后者读取启动脚本所转发的相同 `ANTHROPIC_*` 变量。

<CodeGroup exclude="shell, csharp, java, php, ruby">
  ```python Python
  import asyncio
  import contextlib
  import os
  import signal
  from anthropic import AsyncAnthropic
  from anthropic.lib.environments import EnvironmentWorker


  async def main() -> None:
      async with AsyncAnthropic(
          auth_token=os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
      ) as client:
          worker = EnvironmentWorker(client, workdir="/workspace")
          # 不带参数时，handle_item() 会读取启动脚本转发的 ANTHROPIC_* 变量，
          # 包括 ANTHROPIC_WORK_SECRET。
          task = asyncio.create_task(worker.handle_item())
          # 在容器停止时取消任务，可让工作进程在退出前上传
          # 已更改的内存文件并移除存储目录。
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

  const client = new Anthropic({ authToken: process.env.ANTHROPIC_ENVIRONMENT_KEY });
  const controller = new AbortController();
  // 在容器停止时中止，可让 worker 在退出前上传已更改的内存
  // 文件并移除存储目录。
  process.once("SIGTERM", () => controller.abort());
  process.once("SIGINT", () => controller.abort());

  // 不带参数时，handleItem() 会读取启动脚本转发的 ANTHROPIC_* 变量，
  // 包括 ANTHROPIC_WORK_SECRET。
  await new EnvironmentWorker({
    client,
    workdir: "/workspace",
    signal: controller.signal
  }).handleItem();
  ```

  <CodeGroupItem language="C#">
    C# SDK 没有 `EnvironmentWorker`。
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
  	// 在容器停止时取消 context，可让 worker 在退出前上传
  	// 已更改的内存文件并移除存储目录。
  	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
  	defer stop()

  	client := anthropic.NewClient(option.WithAuthToken(os.Getenv("ANTHROPIC_ENVIRONMENT_KEY")))
  	worker := environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{
  		Workdir: "/workspace",
  	})
  	// 使用零值选项时，HandleItem 会读取启动脚本转发的 ANTHROPIC_* 变量，
  	// 包括 ANTHROPIC_WORK_SECRET。
  	if err := worker.HandleItem(ctx, environments.HandleItemOptions{}); err != nil {
  		log.Fatalf("worker: %v", err)
  	}
  }
  ```

  <CodeGroupItem language="Java">
    Java SDK 没有 `EnvironmentWorker`。
  </CodeGroupItem>

  <CodeGroupItem language="PHP">
    PHP SDK 没有 `EnvironmentWorker`。
  </CodeGroupItem>

  <CodeGroupItem language="Ruby">
    Ruby SDK 没有 `EnvironmentWorker`。
  </CodeGroupItem>
</CodeGroup>

## 为会话准备文件

Anthropic 不会将文件或 GitHub 仓库挂载到自托管沙箱中。要使会话特定的文件可用：

1. 在会话的 `metadata` 字段中传递文件引用，例如 S3 路径或提交 SHA。
2. 在您的启动脚本或 `--on-work` 处理程序中，获取会话（`GET /v1/sessions/{session_id}`）并读取 `metadata`。认领的工作项携带会话 ID，但不携带元数据。
3. 在任何工具运行之前，将文件准备到工作目录中。

<CodeGroup>
  ```bash cURL
  curl -sS --fail-with-body https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ANTHROPIC_ENVIRONMENT_ID"'",
      "metadata": {"input_file": "s3://my-bucket/data.csv"}
    }'
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent "$AGENT_ID" \
    --environment-id "$ANTHROPIC_ENVIRONMENT_ID" \
    --metadata '{"input_file": "s3://my-bucket/data.csv"}'
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
      metadata={"input_file": "s3://my-bucket/data.csv"},
  )
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id,
    metadata: { input_file: "s3://my-bucket/data.csv" }
  });
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
      Metadata = new Dictionary<string, string> { ["input_file"] = "s3://my-bucket/data.csv" },
  });
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent:         anthropic.BetaSessionNewParamsAgentUnion{OfString: anthropic.String(agent.ID)},
  	EnvironmentID: environment.ID,
  	Metadata: map[string]string{
  		"input_file": "s3://my-bucket/data.csv",
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
      .metadata(SessionCreateParams.Metadata.builder()
          .putAdditionalProperty("input_file", JsonValue.from("s3://my-bucket/data.csv"))
          .build())
      .build());
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
      metadata: ['input_file' => 's3://my-bucket/data.csv'],
  );
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id,
    metadata: {input_file: "s3://my-bucket/data.csv"}
  )
  ```
</CodeGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="监控与故障排除" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations">
    读取队列深度，干净地停止会话和 worker，并修复常见故障。
  </Card>

  <Card title="安全模型" icon="lock" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-security">
    自托管沙箱环境的责任共担模型。
  </Card>
</CardGroup>

## 可用性

自托管 worker在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

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
