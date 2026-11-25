---
title: 每个会话运行一个沙箱
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session
description: 通过 Kubernetes 上的 Helm chart、启动脚本或 SDK 轮询器，为每个会话提供其专属的自托管沙箱。
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

在"sandbox-per-session"（每个会话一个沙箱）模式中，每个会话都在一个全新的沙箱中运行，该沙箱可以拥有自己的文件系统、资源限制和网络控制。要将其与其他模式进行比较，请参阅[选择部署模式](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#choose-a-deployment-pattern)。

您可以通过三种方式进行设置：

* **[Kubernetes 上的 Helm chart](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#run-sandboxes-on-kubernetes)：** Anthropic 的调度器（dispatcher）认领工作，并在您的集群上为每个会话启动一个 pod。
* **[启动脚本](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#launch-sandboxes-with-a-spawn-script)：** `ant` CLI 认领工作并调用您的脚本，由脚本在任意平台上启动沙箱。
* **[SDK 轮询器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#launch-sandboxes-from-the-sdk-poller)：** 由您自己的代码认领工作并启动沙箱。

[在 Kubernetes 上运行沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#run-sandboxes-on-kubernetes)之后的各节适用于启动脚本和 SDK 轮询器。

## 在 Kubernetes 上运行沙箱

Anthropic 发布了一个 Helm chart `cma-sandboxes` 及其镜像。该 chart 在您自己的 Kubernetes 集群上为每个会话运行一个沙箱。它的调度器轮询环境的工作队列，并为每个会话分配一个专属的 pod：

* **隔离：** 每个 pod 都在 [gVisor](https://gvisor.dev/) 下运行，并受 chart 的网络策略保护。它拥有自己的文件系统和资源限制。只有调度器持有环境密钥。
* **会话亲和性：** 会话中的每条消息都会发送到该会话的沙箱。会话空闲 30 分钟后（chart 的 `sandbox.idleTimeout` 设置的默认值），其 pod 会停止。其 `/workspace` 卷会一直保留，直到会话结束。下一条消息会在同一个卷上启动一个 pod。
* **预热池：** 除 Google Kubernetes Engine (GKE) 外，chart 会保留两个就绪的 pod 供新会话使用（chart 的 `sandbox.warmPoolSize` 设置的默认值）。

沙箱 pod 运行 `ant` CLI worker。它们支持使用默认同步设置的[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)，但不支持[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools)。

### 安装 chart

该 chart 可在 GKE、Amazon EKS 及其他集群上运行。安装之前，请先完成以下设置：

* **在集群中：** [Agent Sandbox](https://github.com/kubernetes-sigs/agent-sandbox) 控制器（在 GKE 上为 Agent Sandbox 插件）、运行 gVisor 的节点，以及强制执行 NetworkPolicy 的网络插件。
* **在您自己的命名空间中：** Restricted Pod Security Standard 标签、包含环境 ID 和[环境密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#run-your-first-session)的 Secret，以及调度器的 TLS 证书。

[GitHub 上的安装指南](https://github.com/anthropics/anthropic-cli/blob/main/sandboxes/kubernetes/README.md)提供了每种集群的设置步骤，还涵盖了升级和密钥轮换。

将您的设置保存为 `cma-values.yaml`。如果此文件中缺少这三项设置，安装将失败：

<File filename="cma-values.yaml">
  ```yaml
  platform: gke             # or eks, or generic for any other cluster
  egress:
    # The chart already blocks the private ranges. List your other addresses
    # that sandboxes must not reach.
    clusterCIDRs:
      - 203.0.113.10/32     # your cluster's Kubernetes API endpoint
    cmaServer:
      public: true          # the Claude API has a public address
  ```
</File>

安装指南列出了取决于您集群的设置，例如 GKE Autopilot 上的 `gke.dns`。它还说明了何时用 `egress.clusterIsPrivate: true` 代替 `egress.clusterCIDRs`。

然后，安装 chart：

```bash
helm install cma \
  oci://us-docker.pkg.dev/anthropic-public-registry/sandboxes/charts/cma-sandboxes \
  --version 0.1.0 \
  --namespace cma \
  -f cma-values.yaml \
  --wait
```

安装完成后，运行安装指南中描述的预检检查。

<Warning>
  不要仅使用 `helm uninstall` 移除 chart。该命令会删除网络策略，使会话的沙箱在没有网络策略的情况下继续运行。请先停止调度器并删除沙箱，顺序按照[安装指南的 Removal 部分](https://github.com/anthropics/anthropic-cli/blob/main/sandboxes/kubernetes/README.md#removal)给出的进行。
</Warning>

### 向沙箱镜像添加工具

发布的沙箱镜像是包含 `bash`、`curl`、`jq` 和 `ant` CLI 的 Alpine Linux。chart 以只读根文件系统和非 root 用户身份运行它，因此智能体无法在会话期间安装软件包。

要为您的智能体提供更多工具（例如 Python 或 Git），请基于该版本标签从仓库构建这两个镜像。[安装指南的 Install 部分](https://github.com/anthropics/anthropic-cli/blob/main/sandboxes/kubernetes/README.md#install)提供了相关命令。将镜像以 `cma-dispatcher` 和 `cma-sandbox` 为名推送到您的镜像仓库，并使用 chart 的版本作为标签。然后，将 `image.registry` 设置为该镜像仓库。如果您固定了镜像摘要，请使用您自己镜像的摘要。

### 限制网络出站流量

默认情况下，chart 的网络策略允许沙箱访问公共 IPv4 互联网。它们会阻止私有地址范围、云元数据服务器以及 `egress.clusterCIDRs` 中的地址。遭受"prompt injection"（提示注入）的智能体可以将其读取的内容发送到其沙箱能够访问的任何地址。安装指南中的 [Network egress](https://github.com/anthropics/anthropic-cli/blob/main/sandboxes/kubernetes/README.md#network-egress) 提供了详细信息。

* **启用 Dataplane V2 的 GKE：** 设置 `egress.mode: fqdnGke`，将连接限制为 `egress.fqdnGke.hosts` 中主机的 443 端口。请在该列表中保留 `api.anthropic.com`，并添加您的智能体命令所需的每个主机。与默认模式不同，此模式没有阻止范围列表。它允许所列主机名称解析到的任何地址，因此只列出您信任其 DNS 的主机。沙箱仍然可以通过 DNS 查询向外发送数据。
* **其他集群和未启用 Dataplane V2 的 GKE：** 默认模式是唯一的模式。
* **所有集群：** 网络策略不涵盖集群自身节点监听的端口。请参阅安装指南中的 [Node isolation you own](https://github.com/anthropics/anthropic-cli/blob/main/sandboxes/kubernetes/README.md#node-isolation-you-own)。

## 使用启动脚本启动沙箱

主机上的轮询器认领工作，并为每个工作项调用一次您的脚本。该脚本为这一个会话启动一个沙箱。

<Steps>
  <Step title="构建沙箱镜像">
    安装 `ant` 并将 `ant beta:worker run` 设置为入口点。沙箱启动时，它会从环境变量中读取会话详细信息，处理该会话，然后退出。基础镜像必须提供 `/bin/bash`；构建过程仅使用 `curl` 下载 `ant`。

    <File filename="Dockerfile">
      ```dockerfile
      FROM your-base-image
      ARG ANT_VERSION=2.14.0
      ARG TARGETARCH
      RUN ARCH=$([ "$TARGETARCH" = "arm64" ] && echo arm64 || echo amd64) && \
          curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${ANT_VERSION}/ant_${ANT_VERSION}_linux_${ARCH}.tar.gz" \
            | tar -xz -C /usr/local/bin ant
      WORKDIR /workspace
      VOLUME /workspace
      ENTRYPOINT ["ant", "beta:worker", "run"]
      ```
    </File>
  </Step>

  <Step title="编写启动脚本">
    该脚本将会话详细信息转发到一个全新的沙箱中。它要求轮询器主机上安装 `jq`。

    <File filename="spawn.sh">
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
    </File>

    轮询器会设置 `ANTHROPIC_WORK_ID`、`ANTHROPIC_ENVIRONMENT_ID`、`ANTHROPIC_SESSION_ID` 和 `ANTHROPIC_ENVIRONMENT_KEY`。只有当您在轮询器主机上设置了 `ANTHROPIC_BASE_URL` 时，它才会传递给脚本。脚本从标准输入上工作项的 JSON 中读取 `ANTHROPIC_WORK_SECRET`。请参阅[环境变量](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#environment-variables)。

    `/host/outputs` 是您选择的主机目录。将其挂载到 `/workspace` 后，您可以在沙箱退出后取回会话的交付成果。该挂载还会包含下载的 `skills/` 目录树以及所有中间文件。
  </Step>

  <Step title="启动轮询器">
    ```bash
    ant beta:worker poll --on-work ./spawn.sh
    ```
  </Step>
</Steps>

## 转发工作项的机密

每个被认领的工作项都可以携带一个 Anthropic 为该会话签发的 `secret` (csharp, go: `Secret`)。运行该会话的 worker 需要它来挂载[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)。

在同一进程中认领并运行会话的 worker 会自行传递该机密。使用 `run()` (go: `Run()`) 的 `EnvironmentWorker` 就是这样的 worker，不带 `--on-work` 的 `ant beta:worker poll` 也是。当您自己的代码位于认领和 worker 之间时，需要由您转发该机密：

| 您认领工作的方式 | 机密的到达形式 | 传递给 worker 的方式 |
| ----- | ----- | ----- |
| `ant beta:worker poll --on-work` | 脚本标准输入上工作项 JSON 的 `secret` 字段 | 沙箱环境中的 `ANTHROPIC_WORK_SECRET` |
| Python、TypeScript 或 Go SDK 的 `work.poller()` (typescript: `new WorkPoller()`; go: `environments.NewWorkPoller()`) | 每个被认领工作项的 `secret` (go: `Secret`) 字段 | 沙箱环境中的 `ANTHROPIC_WORK_SECRET`，或 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`) 的 `work_secret` (typescript: `workSecret`; go: `WorkSecret`) 参数 |

只将机密传入为该会话服务的沙箱，并且切勿记录它。有关它与环境密钥的关系，请参阅[安全模型](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-security)。

## 从 SDK 轮询器启动沙箱

要从您自己的代码而不是 `ant beta:worker poll --on-work` 认领工作，请使用 Python、TypeScript 或 Go SDK 中的轮询器：`work.poller()` (typescript: `new WorkPoller()`; go: `environments.NewWorkPoller()`)。它轮询队列并将每个被认领的会话交给您，由您启动沙箱：

<CodeGroup exclude="shell, csharp, java, php, ruby">
  <CodeGroupItem language="cURL">
    在循环中认领工作并为每个会话启动沙箱所需的逻辑超出了单个 shell 命令的范围。在 shell 中，请结合启动脚本运行 `ant beta:worker poll --on-work`，如[使用启动脚本启动沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#launch-sandboxes-with-a-spawn-script)中所示。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    要从 shell 认领工作并为每个会话启动沙箱，请结合启动脚本运行 `ant beta:worker poll --on-work`。请参阅[使用启动脚本启动沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#launch-sandboxes-with-a-spawn-script)。
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
      # 密钥（切勿转发您的 API 密钥）以及工作项的每会话机密：沙箱内的 worker
      # 需要该机密来挂载会话的记忆存储。
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
    // 密钥（切勿转发您的 API 密钥）以及工作项的每会话机密：沙箱内的 worker
    // 需要该机密来挂载会话的记忆存储。
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
    C# SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用 [Environments Work 端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)来轮询工作项并确认它。
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
    Java SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用 [Environments Work 端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)来轮询工作项并确认它。
  </CodeGroupItem>

  <CodeGroupItem language="PHP">
    PHP SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用 [Environments Work 端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)来轮询工作项并确认它。
  </CodeGroupItem>

  <CodeGroupItem language="Ruby">
    Ruby SDK 没有轮询工作的辅助工具。要从您自己的代码认领工作，请调用 [Environments Work 端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)来轮询工作项并确认它。
  </CodeGroupItem>
</CodeGroup>

## 在沙箱内运行 SDK worker

当沙箱必须[提供自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools)或使用[非默认的记忆同步设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#configure-sync)时，请将 `ant beta:worker run` 入口点替换为 SDK 入口点。该入口点构造 `EnvironmentWorker` 并调用 `handle_item()` (typescript: `handleItem()`; go: `HandleItem()`)，后者读取启动脚本所转发的相同 `ANTHROPIC_*` 变量。

<CodeGroup exclude="shell, csharp, java, php, ruby">
  <CodeGroupItem language="cURL">
    worker 是一个长期运行的进程，而不是单个 HTTP 请求。要提供自定义工具或更改记忆同步设置，请使用 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    `ant` CLI worker（`ant beta:worker run`）无法提供自定义工具，也无法更改记忆同步设置。要实现其中任一项，请使用 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
  </CodeGroupItem>

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
          # 不带参数时，handle_item() 会读取启动脚本转发的
          # ANTHROPIC_* 变量，包括 ANTHROPIC_WORK_SECRET。
          task = asyncio.create_task(worker.handle_item())
          # 在容器停止时取消任务，可让 worker 在退出前上传
          # 已更改的记忆文件并删除存储目录。
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
  // 在容器停止时中止，可让 worker 在退出前上传已更改的记忆
  // 文件并删除存储目录。
  process.once("SIGTERM", () => controller.abort());
  process.once("SIGINT", () => controller.abort());

  // 不带参数时，handleItem() 会读取启动
  // 脚本转发的 ANTHROPIC_* 变量，包括 ANTHROPIC_WORK_SECRET。
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
  	// 在容器停止时取消上下文，可让工作进程在退出前上传
  	// 已更改的记忆文件并删除存储目录。
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

Anthropic 不会将文件或 GitHub 仓库挂载到自托管沙箱中。要提供会话特定的文件：

1. 在会话的 `metadata` (csharp, go: `Metadata`) 字段中传递文件引用，例如 S3 路径或提交 SHA。
2. 在您的启动脚本或 `--on-work` 处理程序中，获取会话（`GET /v1/sessions/{session_id}`）并读取 `metadata` (csharp, go: `Metadata`)。被认领的工作项携带会话 ID，但不携带元数据。
3. 在任何工具运行之前，将文件准备到工作目录中。

cURL 示例从 `ANTHROPIC_AUTH_TOKEN` 读取访问令牌。要获取访问令牌，请按照[设置联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#set-up-federation)中的说明，使用默认的 `workspace:developer` 范围设置 Workload Identity Federation。然后，按照[从您的工作负载进行身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#authenticate-from-your-workload)中所示，将您工作负载的身份令牌交换为访问令牌。仅在运行 cURL 示例的 shell 中设置 `ANTHROPIC_AUTH_TOKEN`，因为它的优先级高于您的联合设置。[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)列出了 API 接受的其他凭据，例如快速入门中的 API 密钥。

<CodeGroup>
  ```bash cURL
  curl -sS --fail-with-body https://api.anthropic.com/v1/sessions \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "agent_011CYm1BLqPXpQRk5khsSXrs",
      "environment_id": "env_01595EKxaaTTGwwY3kyXdtbs",
      "metadata": {"input_file": "s3://my-bucket/data.csv"}
    }'
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent agent_011CYm1BLqPXpQRk5khsSXrs \
    --environment-id env_01595EKxaaTTGwwY3kyXdtbs \
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
  <Card title="监控自托管 worker 并排查故障" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations">
    读取队列深度，干净地停止会话和 worker，并修复常见故障。
  </Card>
</CardGroup>
