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

自托管环境上的会话附加[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)的方式与云环境上的会话完全相同。在创建会话时，将它们列在 `resources` (csharp, go: `Resources`) 中，如[将记忆存储附加到会话](https://platform.claude.com/docs/zh-CN/managed-agents/memory#attach-a-memory-store-to-a-session)中所示。一个会话最多可接受 8 个记忆存储。

区别在于由谁来实体化存储。在自托管环境上，由您的 worker（而不是 Anthropic 的基础设施）将每个存储下载到沙箱中，并将智能体的更改同步回去。

## 要求

* **能够挂载记忆存储的 worker：** 使用 `ant` CLI 1.33.0 或更高版本，或者 Python、TypeScript 或 Go SDK 中的 `EnvironmentWorker`。
* **POSIX 文件系统：** 不支持 Windows 主机，因为 worker 在打开记忆文件时需要 `O_NOFOLLOW`。建议使用区分大小写的文件系统，以免仅大小写不同的记忆路径发生冲突。macOS 卷默认不区分大小写，因此此类路径在 macOS 上会发生冲突。
* **可写的 `/mnt/memory` 目录：** 请参阅[准备主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-the-host)。在 macOS 上，请参阅[准备 macOS 主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-a-macos-host)。
* **工作项的机密：** 如果由您自己的代码启动 worker，请[转发工作项的机密](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#forward-the-work-items-secret)给它。

<Note>
  在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上，无法将记忆存储附加到自托管环境上的会话。

  在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上，自托管环境上的会话不支持记忆存储。
</Note>

## 准备主机

在 macOS 上，请跳过下一条命令，改为按照[准备 macOS 主机](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory#prepare-a-macos-host)操作。

在启动 worker 之前，请创建父目录，并使其对运行 worker 的用户可写：

```bash
sudo mkdir -p /mnt/memory && sudo chown "$USER" /mnt/memory
```

不要自己创建每个存储的目录。worker 会在会话开始时创建每个存储的 `mount_path` (java, php: `mountPath`; csharp, go: `MountPath`) 目录（例如 `/mnt/memory/user-preferences`），并在会话结束时将其删除。如果该路径上已存在内容，worker 将拒绝开始该会话的工作。

在[每个会话一个沙箱模式](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#run-one-sandbox-per-session)中，沙箱镜像需要一个可写的 `/mnt/memory`。您无需将记忆目录绑定挂载到主机，因为 worker 会在沙箱退出之前将其内容上传到存储。

### 准备 macOS 主机

在 macOS 上，根卷是只读的，因此 `mkdir` 无法创建 `/mnt`。请改为在可写的数据卷上创建 `/System/Volumes/Data/mnt/memory`。然后向 `/etc/synthetic.conf` 添加一行，使 macOS 将 `/mnt` 创建为指向 `/System/Volumes/Data/mnt` 的符号链接（symbolic link）。这些步骤需要管理员权限。

如果没有此设置，worker 将无法创建存储的目录，因此会使工作项失败。随后会话会保持空闲状态，且不会产生错误事件。请参阅[记忆存储挂载失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#memory-stores-fail-to-mount)。

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
    * 如果最后一行不以 `$` 结尾，请运行 `echo | sudo tee -a /etc/synthetic.conf` 来结束该行。
    * 如果没有输出任何内容或报告 `No such file or directory`，请继续。如果文件不存在，`printf` 命令会创建它。

    然后添加该行，除非属于前两种情况之一。此命令只需运行一次，因为每次运行都会追加一行。

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

两个会话无法在同一主机上同时挂载同一个存储，因为两者需要相同的路径。如果您的会话附加了同一个存储，请为每个文件系统只运行一个会话。为每个会话提供[独立的沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#run-one-sandbox-per-session)即可满足此规则。

## worker 如何处理记忆

当 worker 认领一个其会话附加了记忆存储的工作项时，它会：

1. **将每个存储下载到其 `mount_path` (java, php: `mountPath`; csharp, go: `MountPath`)。** 这与云会话使用的 `/mnt/memory/` 下的目录相同，会话的系统提示会向智能体描述该目录。例如，名为"User Preferences"的存储会位于 `/mnt/memory/user-preferences/`。
2. **向文件工具开放这些目录。** 智能体使用与在工作目录中相同的文件工具来处理记忆。
3. **在工具调用后协调更改，** 每个同步间隔最多一次（默认为 15 秒）。存储中发生更改的记忆会写入磁盘，智能体更改的文件会上传到存储。
4. **在会话结束时执行最终同步。** 它会在最多 30 秒内刷新所有仍待处理的上传，然后删除它创建的目录。

Anthropic 端的记忆存储仍然是唯一可信来源。[记忆版本](https://platform.claude.com/docs/zh-CN/managed-agents/memory#audit-memory-changes)、内容编辑（redaction）以及在 Claude Console 中查看或编辑记忆的工作方式与云会话相同。智能体对记忆的读写会作为普通工具事件出现在[事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)中。

由于每个 worker 按间隔进行同步，在一个会话中写入的更改只有在两个会话都完成同步后，才会对另一个正在运行的会话可见。在默认间隔下，这通常远少于一分钟。云沙箱上的会话几乎可以立即看到彼此的更改。

每个存储目录都包含一个名为 `.anthropic-memory-store` 的标记文件，用于将该目录与其存储关联起来。请保留该文件：worker 不会同步标记文件缺失或被修改的目录。

<Warning>
  被强制终止（而非正常停止）的 worker 不会执行清理，因此未同步的编辑会丢失，存储目录也会残留。请参阅[优雅地停止 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#stop-workers-gracefully)。
</Warning>

## 配置同步

有两个 `EnvironmentWorker` 选项控制记忆行为。在构造 worker 的任何位置设置它们，包括在 webhook 处理程序中。`ant` CLI worker 始终使用默认值。

### 同步间隔

`memory_sync_interval` (typescript: `memorySyncIntervalMs`; go: `MemorySyncInterval`) 设置会话运行期间已附加的存储与服务器协调的频率。

| 设置       | 值                                                   |
| -------- | --------------------------------------------------- |
| 默认值      | 15 秒                                                |
| 最小值      | 5 秒                                                 |
| 示例（10 秒） | `10` (typescript: `10_000`; go: `10 * time.Second`) |
| 禁用记忆支持   | `None` (typescript: `null`; go: `-1`)               |

较短的间隔可以缩小另一个会话看到过时记忆的时间窗口，但代价是更多的记忆存储请求。

仅在其会话不附加任何记忆存储的 worker 上禁用记忆支持。已禁用的 worker 既不下载也不同步存储，因此附加了存储的会话将在没有这些存储的情况下运行，即使其系统提示仍然描述了它们。

启用记忆支持时，对于附加了存储的会话，如果工作项到达时没有 `secret` (csharp, go: `Secret`)，该工作项将失败，而不是在没有记忆的情况下运行。请参阅[记忆存储挂载失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#memory-stores-fail-to-mount)。

### 删除

`memory_sync_deletions` (typescript: `memorySyncDeletions`; go: `MemorySyncDeletions`) 设置智能体在本地删除的文件是否也从存储中删除。上传和下载不受影响。

| 值                                                               | 行为                                                       |
| --------------------------------------------------------------- | -------------------------------------------------------- |
| `"enabled"` (go: `environments.MemorySyncDeletionsEnabled`)（默认） | 在后续同步确认文件仍不存在后，从存储中删除该记忆。                                |
| `"log_only"` (go: `environments.MemorySyncDeletionsLogOnly`)    | 执行相同的检查，但仅记录本应删除的内容。在信任启用模式之前，可使用此模式观察您的 worker 会删除哪些内容。 |
| `"disabled"` (go: `environments.MemorySyncDeletionsDisabled`)   | 从不从存储中删除。                                                |

例如，要每 10 秒同步一次，并且仅记录 worker 本应执行的删除操作：

<CodeGroup exclude="shell, csharp, java, php, ruby">
  ```python Python
  worker = EnvironmentWorker(
      client,
      environment_id=environment_id,
      environment_key=environment_key,
      workdir="/workspace",
      memory_sync_interval=10,  # seconds
      memory_sync_deletions="log_only",
  )
  ```

  ```typescript TypeScript
  const worker = new EnvironmentWorker({
    client,
    environmentId,
    environmentKey,
    workdir: "/workspace",
    memorySyncIntervalMs: 10_000,
    memorySyncDeletions: "log_only"
  });
  ```

  <CodeGroupItem language="C#">
    C# SDK 没有 `EnvironmentWorker`。
  </CodeGroupItem>

  ```go Go
  worker := environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{
  	EnvironmentID:       environmentID,
  	EnvironmentKey:      environmentKey,
  	Workdir:             "/workspace",
  	MemorySyncInterval:  10 * time.Second,
  	MemorySyncDeletions: environments.MemorySyncDeletionsLogOnly,
  })
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

对于以 `access: "read_only"` 附加的存储，`write` 和 `edit` 工具会拒绝更改其目录中的文件。worker 从不从中上传任何内容。

通过 `bash`，或通过您从沙箱提供的自定义工具或 MCP 服务器所做的更改，不会在本地被阻止。这些更改永远不会同步到存储，并且该记忆的下一次远程更改会覆盖它们。如果本地副本本身在会话期间必须保持不变：

* 为该智能体禁用 `bash` 工具，并且不要为其提供任何会写入沙箱文件系统的自定义工具。
* 不要以只读方式挂载存储路径。worker 本身必须创建该目录并将下载的记忆写入其中。

冲突以存储为准进行解决。假设智能体更改了一个记忆文件，而该文件自会话上次同步以来在存储中也发生了更改。在下一次同步时，worker 会保留存储的版本，用它覆盖本地文件，并记录一条警告。`write` 和 `edit` 工具本身会成功执行，不会有错误传递给智能体。如果智能体的更改仍然适用，它可以在同步后重新读取该文件并再次进行更改。

## 故障排除

有关 worker 的日志消息及其修复方法，请参阅[记忆存储挂载失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#memory-stores-fail-to-mount)。
