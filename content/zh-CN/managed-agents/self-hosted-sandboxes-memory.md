---
title: 自托管沙箱中的记忆存储
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory
description: 将记忆存储附加到在自托管沙箱中运行的 Claude Managed Agents 会话：准备主机、配置同步，以及处理只读存储和冲突。
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
---

自托管环境上的会话附加[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)的方式与云环境上的会话完全相同。在创建会话时，将它们列在 `resources` (csharp, go: `Resources`) 中，如[将记忆存储附加到会话](https://platform.claude.com/docs/zh-CN/managed-agents/memory#attach-a-memory-store-to-a-session)中所示。一个会话最多可接受 8 个记忆存储。

区别在于由谁来实体化存储。在自托管环境上，由您的 worker（而不是 Anthropic 的基础设施）将每个存储下载到沙箱中，并将智能体的更改同步回去。

## 要求

* **能够挂载记忆存储的 worker：** 使用 `ant` CLI 1.33.0 或更高版本，或者 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
* **POSIX 文件系统：** 不支持 Windows 主机，因为 worker 在打开记忆文件时需要 `O_NOFOLLOW`。Anthropic 建议使用区分大小写的文件系统，以免仅大小写不同的记忆路径发生冲突。macOS 卷默认不区分大小写，因此此类路径在 macOS 上会发生冲突。
* **可写的 `/mnt/memory` 目录：** 请参阅[准备主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-the-host)。在 macOS 上，请参阅[准备 macOS 主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-a-macos-host)。
* **工作项的机密：** 如果由您自己的代码启动 worker，请[转发工作项的机密](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session#forward-the-work-items-secret)给它。

## 准备主机

在 macOS 上，请跳过下一条命令，改为按照[准备 macOS 主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-a-macos-host)操作。

在启动 worker 之前，请创建父目录，并使其对运行 worker 的用户可写：

```bash
sudo mkdir -p /mnt/memory && sudo chown "$USER" /mnt/memory
```

不要自己创建任何存储的目录。worker 会在认领会话时创建每个存储的 `mount_path` (java, php: `mountPath`; csharp, go: `MountPath`) 目录（例如 `/mnt/memory/user-preferences`），并在处理完该会话后将其删除。如果该路径上已存在任何内容（即使是空目录），worker 也会使工作项失败。

在[每个会话一个沙箱模式](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session)中，沙箱镜像需要一个可写的 `/mnt/memory`。您无需将记忆目录绑定挂载到主机，因为存储保存着每条记忆的持久副本。

### 准备 macOS 主机

在 macOS 上，根卷是只读的，因此 `mkdir` 无法创建 `/mnt`。请改为在可写的数据卷上创建 `/System/Volumes/Data/mnt/memory`。然后向 `/etc/synthetic.conf` 添加一行，使 macOS 将 `/mnt` 创建为指向 `/System/Volumes/Data/mnt` 的符号链接（symbolic link）。这些步骤需要管理员权限。

如果没有此设置，worker 将无法创建存储的目录，因此会使工作项失败。随后会话会保持空闲状态，且不会产生错误事件。请参阅[记忆存储挂载或同步失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#memory-stores-fail-to-mount)。

<Steps>
  <Step title="在数据卷上创建目录">
    创建 `memory` 目录，并将 worker 的用户设为其所有者。`$USER` 是您自己的用户名。如果 worker 以其他用户身份运行，请将 `"$USER"` 替换为该用户的名称。

    ```bash
    sudo mkdir -p /System/Volumes/Data/mnt/memory && sudo chown "$USER" /System/Volumes/Data/mnt/memory
    ```

    此位置只是一种选择。如果您使用其他位置，请相应修改 `synthetic.conf` 行中的路径。Anthropic 仅测试过 `/System/Volumes/Data/mnt`。
  </Step>

  <Step title="向 synthetic.conf 添加一行">
    运行 `cat -et /etc/synthetic.conf`，它会将每个制表符显示为 `^I`，将每个行尾显示为 `$`。

    * 如果某一行为 `mnt^ISystem/Volumes/Data/mnt$`，则此步骤已完成。请转到下一步。
    * 如果另有一行以 `mnt` 开头，请停止。这台 Mac 上的其他内容已经定义了 `/mnt`。
    * 如果最后一行不以 `$` 结尾，请运行 `echo | sudo tee -a /etc/synthetic.conf` 来结束该行。然后添加该行。
    * 如果没有输出任何内容或报告 `No such file or directory`，请添加该行。如果文件不存在，以下命令会创建它。
    * 否则，请添加该行。

    要添加该行，请运行此命令。此命令只需运行一次，因为每次运行都会追加一行。

    ```bash
    printf 'mnt\tSystem/Volumes/Data/mnt\n' | sudo tee -a /etc/synthetic.conf
    ```

    该命令会打印它添加的行。该行将名称 `mnt` 映射到包含 `memory` 的目录。文件中两列之间需要一个制表符，由 `\t` 写入。
  </Step>

  <Step title="创建链接">
    使用 `-t` 选项运行 `apfs.util`。它会根据您添加的行创建链接，无需重启。

    ```bash
    sudo /System/Library/Filesystems/apfs.fs/Contents/Resources/apfs.util -t
    ```
  </Step>

  <Step title="检查结果">
    列出 `/mnt` 和 `/mnt/memory`。

    ```bash
    ls -ld /mnt /mnt/memory
    ```

    第一行以 `/mnt -> System/Volumes/Data/mnt` 结尾。第二行显示 worker 的用户为 `/mnt/memory` 的所有者。现在您可以启动 worker 了。
  </Step>
</Steps>

Anthropic 已在 macOS 26.7.1 上使用 `ant` CLI worker 测试过此设置。

### 隔离共享同一存储的会话

两个会话无法在同一文件系统上同时挂载同一个存储，因为两者需要相同的路径。要并行运行共享同一存储的会话，请为每个会话提供独立的文件系统。如果您为每个会话使用一个沙箱，每个会话都会自动获得独立的文件系统。请参阅[每个会话运行一个沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-per-session)。一台主机可以运行许多这样的沙箱。

## worker 如何处理记忆

当 worker 认领一个其会话附加了记忆存储的工作项时，它会：

1. **将每个存储下载到其 `mount_path` (java, php: `mountPath`; csharp, go: `MountPath`)。** 这与云会话使用的 `/mnt/memory/` 下的目录相同，会话的系统提示会向智能体描述该目录。例如，名为"User Preferences"的存储会位于 `/mnt/memory/user-preferences/`。
2. **向文件工具开放这些目录。** 智能体使用与在工作目录中相同的文件工具来处理记忆。
3. **在工具调用后进行同步。** 如果自上次同步以来已超过[同步间隔](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#sync-interval)（默认为 15 秒），worker 会在工具调用后进行同步。每次同步都会将存储中已更改的记忆写入磁盘，并上传智能体更改的文件。
4. **在会话结束或空闲时间达到 worker 的空闲上限时执行最终同步。** 最终同步最多可能需要 60 秒。届时仍未上传的更改将会丢失。然后 worker 会删除它创建的目录。

空闲上限默认为 60 秒。[worker 何时结束对会话的处理](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#when-the-worker-finishes-with-a-session)说明了该上限涵盖的内容以及如何更改它。

智能体的 `write` 和 `edit` 工具只会更改本地副本，worker 会在下一次同步时上传这些更改。`/mnt/memory/` 下存储目录之外的路径不是临时空间。worker 的文件工具会拒绝写入这些路径，而 shell 命令写入其中的任何内容都永远不会同步到存储。

[记忆版本](https://platform.claude.com/docs/zh-CN/managed-agents/memory#audit-memory-changes)、脱敏（redact）以及在 Claude Console 中查看或编辑记忆的工作方式与云会话相同。智能体对记忆的读写会作为普通工具事件出现在[事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)中。

每个存储目录都包含一个名为 `.anthropic-memory-store` 的标记文件，用于将该目录与其存储关联起来。请保留该文件。如果标记文件缺失或被修改，且该目录中仍有文件，worker 将不会同步该目录，也不会在处理完该会话后删除该目录。

<Warning>
  如果您在会话期间优雅地停止 worker 或[停止会话的工作项](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#stop-a-session-gracefully)，worker 仍会上传新增和已更改的文件，并删除存储目录。它不会发送任何删除操作，因此智能体删除的记忆会保留在存储中，除非之前的同步已将其从存储中删除。被强制终止（而非停止）的 worker 不会执行清理，因此未同步的编辑会丢失，存储目录也会残留。请参阅[优雅地停止 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#stop-workers-gracefully)。
</Warning>

## 配置同步

有两个 `EnvironmentWorker` 选项控制记忆行为。在构造 worker 的任何位置设置它们，包括在 webhook 处理程序中。`ant` CLI worker 始终使用默认值。

### 同步间隔

`memory_sync_interval` (typescript: `memorySyncIntervalMs`; go: `MemorySyncInterval`) 设置会话运行期间两次同步之间的最短时间。

| 设置       | 值                                                   |
| -------- | --------------------------------------------------- |
| 默认值      | 15 秒                                                |
| 最小值      | 5 秒                                                 |
| 示例（10 秒） | `10` (typescript: `10_000`; go: `10 * time.Second`) |
| 禁用记忆支持   | `None` (typescript: `null`; go: `-1`)               |

worker 不按定时器进行同步。它会在每次调用在您的沙箱中运行的工具（例如 `bash`、`read` 或自定义工具）后检查间隔。该调用不必涉及记忆。如果没有工具调用，在[最终同步](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#how-the-worker-handles-memory)之前不会进行任何同步。一个会话中的更改只有在两个会话都完成同步后才会到达另一个正在运行的会话，因此等待时间没有固定长度。

较短的间隔可以缩小另一个会话看到过时记忆的时间窗口，但代价是更多的记忆存储请求。

仅在其会话不附加任何记忆存储的 worker 上禁用记忆支持。已禁用的 worker 既不下载也不同步存储，因此附加了存储的会话将在没有这些存储的情况下运行，即使其系统提示仍然描述了它们。

启用记忆支持时，对于附加了存储的会话，如果工作项到达时没有 `secret` (csharp, go: `Secret`)，该工作项将失败，而不是在没有记忆的情况下运行。请参阅[记忆存储挂载或同步失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#memory-stores-fail-to-mount)。

### 删除

`memory_sync_deletions` (typescript: `memorySyncDeletions`; go: `MemorySyncDeletions`) 设置智能体在本地删除的文件是否也从存储中删除。上传和下载不受影响。

| 值                                                               | 行为                                                       |
| --------------------------------------------------------------- | -------------------------------------------------------- |
| `"enabled"` (go: `environments.MemorySyncDeletionsEnabled`)（默认） | 在后续同步确认文件仍不存在后，从存储中删除该记忆。                                |
| `"log_only"` (go: `environments.MemorySyncDeletionsLogOnly`)    | 执行相同的检查，但仅记录本应删除的内容。在信任启用模式之前，可使用此模式观察您的 worker 会删除哪些内容。 |
| `"disabled"` (go: `environments.MemorySyncDeletionsDisabled`)   | 从不从存储中删除。                                                |

例如，要最多每 10 秒同步一次，并且仅记录 worker 本应执行的删除操作，请使用以下示例。该示例使用 `/workspace` 作为工作目录，该目录必须在 worker 启动之前存在。[运行常驻 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#run-an-always-on-worker) 介绍了如何创建该目录以及在 macOS 上应使用什么。

<CodeGroup exclude="shell, csharp, java, php, ruby">
  <CodeGroupItem language="cURL">
    worker 是一个长时间运行的进程，而不是单个 HTTP 请求。要更改同步设置，请使用 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    `ant` CLI worker 始终使用默认同步设置。要更改这些设置，请使用 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
  </CodeGroupItem>

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
              memory_sync_interval=10,  # seconds
              memory_sync_deletions="log_only",
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

  const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
  const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
  const client = new Anthropic({ authToken: environmentKey });
  const controller = new AbortController();
  process.once("SIGINT", () => controller.abort());
  process.once("SIGTERM", () => controller.abort());

  const worker = new EnvironmentWorker({
    client,
    environmentId,
    environmentKey,
    workdir: "/workspace",
    signal: controller.signal,
    memorySyncIntervalMs: 10_000,
    memorySyncDeletions: "log_only"
  });
  await worker.run();
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
  	"time"

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
  		EnvironmentID:       environmentID,
  		EnvironmentKey:      environmentKey,
  		Workdir:             "/workspace",
  		MemorySyncInterval:  10 * time.Second,
  		MemorySyncDeletions: environments.MemorySyncDeletionsLogOnly,
  	})
  	if err := worker.Run(ctx); err != nil {
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

## 只读存储和冲突

### 只读存储

对于以 `access: "read_only"` 附加的存储，`write` 和 `edit` 工具会拒绝更改其目录中的文件。worker 从不从中上传任何内容。

通过 `bash`，或通过您从沙箱提供的自定义工具或 MCP 服务器所做的更改，不会在本地被阻止。这些更改永远不会到达存储。已更改的文件会保持更改状态，直到发生以下情况之一：

* **该记忆在存储中有了新版本。** worker 的下一次同步会用该版本覆盖该文件。
* **worker 处理完该会话。** 它会删除该目录。

要同时使本地副本保持不变，请为该智能体禁用 `bash` 工具。此外，不要为其提供任何会写入沙箱文件系统的自定义工具或 MCP 服务器。不要在存储的路径上挂载只读卷。挂载会在该路径上放置一个目录，而如果该路径上已存在任何内容，worker 会使工作项失败。

### 冲突

冲突是指自上次同步以来在存储和沙箱中都发生了更改的记忆。删除也算作更改。对于以 `access: "read_write"` 附加的存储，worker 会在下一次同步时解决冲突：

| 在存储中 | 在沙箱中 | 结果                   |
| ---- | ---- | -------------------- |
| 已更改  | 已更改  | 存储的版本覆盖该文件。          |
| 已更改  | 已删除  | worker 将存储的版本写回沙箱。   |
| 已删除  | 已更改  | worker 根据该文件重新创建该记忆。 |

在前两行中，智能体的更改或删除会丢失，即使工具调用已成功。不会有任何内容通知智能体，丢失的唯一迹象是 worker 日志中的一条警告。在最后一行中，存储中的删除会被撤销。

## 故障排除

有关 worker 的日志消息及其修复方法，请参阅[记忆存储挂载或同步失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#memory-stores-fail-to-mount)。

## 可用性

自托管沙箱中的记忆存储在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台        |
| ---- | ---------- |
| Beta | Claude API |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                                                         | HIPAA                                  |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| 状态   | 不具备资格                                                                                                                                                                         | 不具备资格                                  |
| 详细信息 | 不具备 ZDR 资格，自托管 Managed Agents 除外。会话是有状态的资源；对话记录会一直保留，直到您将其删除。 [了解更多](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#how-long-data-is-kept) | 不具备 HIPAA 资格。适用于所有 Managed Agents 子功能。 |

适用的端点：

* `/v1/agents`
* `/v1/sessions`
* `/v1/environments`
