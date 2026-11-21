---
title: 与托管服务之间的差异
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences
description: 了解哪些功能仅自托管 Claude Managed Agents 具备、哪些托管功能自托管 Claude Managed Agents 不具备、哪些功能的行为不同，以及哪些功能需要先进行服务器设置。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

自托管 Claude Managed Agents 服务器具有与 Anthropic 托管的 [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 服务相同的 API，但本页面所述的差异除外。下表说明了针对每种差异应如何处理。在将应用程序迁移到您的服务器之前，请查阅这些表。不会从托管服务复制任何内容，包括会话历史记录。请在您的服务器上重新创建您的智能体、技能、vault 凭据和记忆存储。

## 仅您的服务器具备的功能

| 功能 | 作用 | 如何设置 |
| ----- | ----- | ----- |
| 可选择推理提供商 | 服务器可以将模型调用发送到云提供商或您自己的模型网关，而不是 Claude API。 | [推理提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider) |
| 单点登录 | 用户使用来自您的 OpenID Connect 身份提供商的令牌调用服务器。服务器根据令牌中的组为每个调用方分配角色。服务器 API 密钥仍然有效。 | [单点登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#single-sign-on) |
| 您自己的密钥管理器 | 服务器可以将每个新 vault 凭据的密钥存储在您的密钥管理器中，而不是存储在其自身的数据库中。 | [密钥管理器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#secret-manager) |
| 智能体身份 | 服务器可以对一个短期令牌进行签名，并在智能体调用 MCP 服务器时随调用一起发送。MCP 服务器或网关会根据您发布的公钥校验该令牌。 | [智能体身份](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#agent-identity) |

## 不可用的功能

| 功能 | 会发生什么 | 替代做法 |
| ----- | ----- | ----- |
| [云环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments) | 仅存在 `self_hosted` 环境。请求云环境会返回 `400` 错误。 | 为每个环境[部署自托管 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers)。 |
| Claude API 的其余部分，例如 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 和 Models API | 服务器不提供这些 API。SDK 对 `/v1/messages` 和 `/v1/models` 的调用会返回 `404` 错误。 | 从不指向您服务器的客户端将这些请求发送到 Claude API 或您的提供商。 |
| [文件会话资源](https://platform.claude.com/docs/zh-CN/managed-agents/files)以及 Files API | 服务器会拒绝 `file` 资源，并且 `/v1/files` 不存在。`memory_store` 资源可以正常使用。 | 将文件放入 worker 的工作目录中。请参阅[在会话中传入和传出数据](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#move-data-in-and-out-of-a-session)。要向模型展示图像或 PDF，请在消息中以 `base64` 形式发送。 |
| [`mcp_oauth` vault 凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults) | 服务器会在您创建这些凭据时拒绝它们，因为它没有用于刷新令牌的 OAuth 流程。 | 将令牌存储为 `static_bearer` 凭据。在旧令牌过期之前，使用新令牌更新该凭据。 |
| [预构建的 `docx`、`pdf`、`pptx` 和 `xlsx` 技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills) | 服务器会拒绝声明这些技能的智能体。只有您上传的技能才存在。 | 从[技能仓库](https://github.com/anthropics/skills)下载它们的源代码，并将其作为自定义技能上传。 |
| [计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)和[梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams) | 服务器没有这些功能的路由。 | 从您自己的调度程序（例如 cron 作业）启动会话。Dreams 没有替代方案。 |
| [Webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks) | 服务器不发送 webhook，因此依赖 webhook 启动的 worker 永远不会启动。 | 通过会话的[事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)跟踪会话，或列出其事件。保持 worker 运行，以便它们可以轮询。 |

## 行为不同的功能

| 功能 | 有何不同 | 处理方法 |
| ----- | ----- | ----- |
| Claude Console | 位于 platform.claude.com 的 Claude Console 不会显示您服务器上的内容。 | 使用[您服务器上的控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console)。 |
| 工具调用 | 即使没有 worker 轮询会话所在的环境，会话的模型轮次仍会运行。随后第一个工具调用会一直等待（不报错），直到有 worker 执行它。 | 如果会话停在某个工具调用处，请检查是否有 worker 正在轮询其环境。 |
| [记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory) | API 是完整的。由 worker（而不是服务器）将每个附加的存储复制到沙箱中并写回更改。请参阅[自托管沙箱中的记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)。 | 在[最新版本的 `ant`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart#installation) 上运行您的 worker，并提供可写的 `/mnt/memory` 目录。 |
| [自定义技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills#create-a-custom-skill)上传 | 服务器不会解压 zip 归档。上传 zip 归档会返回 `400` 错误，内容为 `SKILL.md not found in uploaded files`。 | `ant apply` 和 Python SDK 的 `files_from_dir` 辅助函数可以直接使用。使用其他方式时，请将技能的文件作为单独的部分上传，并让每个部分的文件名以技能文件夹开头，例如 `my-skill/SKILL.md`。 |
| [多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration) | 智能体只能委派给您列出的子智能体。动态工作流、advisor 以及智能体自行定义的子智能体均不可用。仅当 `multiagent` (csharp, go: `Multiagent`) 块的 `type` 为 `coordinator` 时，服务器才会接受该块。多智能体编排中的示例将 `type` 设置为 `multiagent_20261001`，这会返回 `400` 错误。带有 `advisor` (csharp, go: `Advisor`) 设置的块也会返回 `400` 错误。 | 将 `multiagent` (csharp, go: `Multiagent`) 设置为 `{"type": "coordinator", "agents": [...]}`。在 `agents` (csharp, go: `Agents`) 中列出 1 到 20 个智能体。每个条目采用[预定义智能体的形式](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)之一。 |
| [GitHub 仓库](https://platform.claude.com/docs/zh-CN/managed-agents/github)会话资源 | 在您于服务器的设置中列出其 Git 主机之前，服务器会拒绝 `github_repository` 资源。服务器不会发现仓库 `.claude/skills` 目录中的技能，因此请将技能附加到智能体。 | 请参阅[Git 仓库需要添加到允许列表](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#git-repositories-need-an-allowlist-entry)。 |
| [MCP 服务器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector) | 服务器只连接您在其设置中列出的 MCP 主机。 | 请参阅[MCP 服务器需要添加到允许列表](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#mcp-servers-need-an-allowlist-entry)。 |
| [Web 工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions) | 在您于服务器的设置中启用之前，`web_search` 和 `web_fetch` 工具处于关闭状态。 | 请参阅[Web 工具需要服务器设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#web-tools-need-a-server-setting)。 |
| [`environment_variable` vault 凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults) | 在您启用出口网关（位于您的沙箱和互联网之间的一个代理）之前，服务器会拒绝它们。 | 请参阅[环境变量凭据](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#environment-variable-credentials)。如果没有网关，请通过您自己的工具为沙箱提供所需的 CLI 和 API 凭据。 |
| 自定义工具结果 | 服务器会将自定义工具结果的完整文本发送给模型。如果结果超出模型 “context window”（上下文窗口），服务器仍会以 `200` 响应，随后会话以 `session.error` 结束。 | 在发送之前，先在客户端中缩短较长的结果。 |
| MCP 工具结果 | 服务器会截断大于其 `mcp.max_result_bytes` 设置（默认为 100 KiB）的结果。它会告知模型该结果已被截断，其余部分会丢失。托管服务也会截断较长的结果，但对于云沙箱中的会话，可以[将完整结果保存到文件](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector#mcp-tool-output-handling)。 | 要更改该限制，请在[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中设置 `mcp.max_result_bytes`。 |
| 通过 URL 发送的图像和文档 | 服务器接受带有 `url` 来源的 `image` 或 `document` 块，但模型永远看不到它，且不会报错。带有 `file` 来源的 `image` 块也是如此。 | 以 `base64` 形式发送数据，对于文档也可以以 `text` 形式发送。 |
| `rescheduling` 会话状态 | 与托管服务一样，服务器会重试因临时原因失败的模型调用，并在事件流上报告每次尝试。在两次尝试之间，会话状态保持为 `running`，因此 `statuses[]=rescheduling` 筛选条件不会匹配任何会话。 | 从事件流中读取重试信息。 |
| 会话元数据（使用模型网关时） | 如果服务器的 `inference.credentials.assertion.forward_metadata` 设置列出了会话元数据键，则添加、删除或更改其中任一键的更新会返回 `400` 错误。 | 请在创建会话时设置这些键。有关该设置，请参阅[短期网关令牌](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#short-lived-gateway-tokens)。 |

## 需要服务器设置的功能

在您更改服务器的[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)之前，以下每项功能都处于关闭状态，或会拒绝所有主机。

### MCP 服务器需要添加到允许列表

服务器只连接主机已列在其[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中 `mcp.allowed_hostnames` 下的 MCP 服务器：

```yaml
mcp:
  allowed_hostnames:
    - mcp.example.com
```

主机需精确匹配，不支持通配符和子域名，并且该列表默认为空。在 MCP 服务器的主机加入该列表之前，您的服务器会拒绝保存引用它的智能体。

位于您私有网络内的 MCP 服务器不需要 [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)。按照 [MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)中的说明，将其 URL 放入智能体定义中，并将其主机添加到允许列表。URL 需遵守两条规则：

* 必须使用 `https`，除非 MCP 服务器位于环回地址上。
* 如果其主机是只解析为私有或保留地址的名称，您的服务器默认会拒绝连接。请选择适合您情况的解决方法：

| 您的情况 | 解决方法 |
| ----- | ----- |
| 地址位于 `10.0.0.0/8`、`172.16.0.0/12` 或 `192.168.0.0/16` 中，并且您能控制该名称的 DNS 记录 | 保留该名称，同时在设置文件中的 `mcp.private_hostnames` 下列出它。 |
| 其他任何情况 | 在 URL 和允许列表中使用 IP 地址代替名称。MCP 服务器的证书必须包含该地址。 |

### Git 仓库需要添加到允许列表

在托管服务上，使用[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#session-resources)的会话不能拥有 `github_repository` 资源。在您的服务器上，会话可以拥有。在您于服务器的[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中列出其 Git 主机之前，服务器会拒绝这些资源。智能体只能推送到您列出的分支：

```yaml
git:
  allowed_hostnames:
    - github.com
  allowed_push_refs:
    - refs/heads/agent/
```

请注意以下几点：

* **URL：** 主机不必是 `github.com`，但 URL 必须使用 HTTPS。
* **令牌：** 您的客户端发送的令牌永远不会到达沙箱，因为 Git 流量会经过您的服务器，由服务器添加该令牌。
* **分支：** 智能体不会被告知分支列表，因此请在提示中指明分支名称。
* **强制推送：** 服务器不会拒绝强制推送，因此智能体可以覆盖您列出的任何分支。要拒绝强制推送，请在您的 Git 主机上保护该分支。

[访问 GitHub](https://platform.claude.com/docs/zh-CN/managed-agents/github)展示了该请求以及如何轮换令牌。如果克隆失败，会话会收到一个指明原因的 `session.error` 事件。要检查推送，请在您的 Git 主机上查找该分支。即使推送被您的主机拒绝，服务器日志也会显示 `ok`。

### Web 工具需要服务器设置

`web_search` 和 `web_fetch` 工具默认处于关闭状态。要启用它们，请将以下内容添加到服务器的[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中：

```yaml
web_tools:
  enabled: true
```

仅使用此设置时，每个智能体自己的域名列表和内容限制都会生效，与托管服务相同。

要为所有智能体限制这些工具，请设置 `web_tools.allowed_domains`、`web_tools.blocked_domains` 或 `web_tools.max_content_tokens`。这些设置中的每一项也会启用这些工具。服务器会拒绝保存请求超出这些设置允许范围的智能体。例如，它会拒绝允许 `web_tools.allowed_domains` 以外域名的智能体，或设置了更大内容限制的智能体。如果此类智能体已经存在，则在您更新它之前，它无法启动会话。

在这些工具处于关闭状态时，智能体会发生什么取决于其工具集中的 `web_search` 和 `web_fetch` 条目：

| 工具集包含 | 会发生什么 |
| ----- | ----- |
| 没有任一工具的条目 | 智能体会在没有这些工具的情况下运行，并且不会有任何错误提示。 |
| 仅关闭其工具的条目，例如 `{"name": "web_fetch", "enabled": false}` | 同上。 |
| 保持其工具开启的条目，或设置了其域名列表、内容限制或位置的条目 | 服务器会拒绝保存该智能体或为其启动会话。 |

可用的 Web 工具还[取决于您的推理提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#differences-that-depend-on-your-inference-provider)。

## 取决于推理提供商的差异

某些功能仅在服务器的 `inference.provider` 设置为特定值时才能使用。下表显示了在其他值下会发生什么。其中某些情况不会返回任何错误。

`web_search` 和 `web_fetch` 行在您[启用 Web 工具](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#web-tools-need-a-server-setting)之后才适用。在此之前，这两种工具在任何提供商下都无法使用。

| 功能 | 可用的 `inference.provider` 值 | 在其他值下 | 处理方法 |
| ----- | ----- | ----- | ----- |
| `web_search` 工具 | `anthropic`、`vertex`、`foundry` | 服务器永远不会向模型提供该工具。智能体仍可保存和运行，并且仍会列出该工具，但模型无法调用它。 | 为智能体提供可执行搜索的自定义工具或 MCP 服务器。 |
| `web_fetch` 工具 | `anthropic`、`foundry` | 与 `web_search` 相同。 | 为智能体提供可获取页面的自定义工具或 MCP 服务器。 |
| `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) | `anthropic` | 服务器会拒绝保存设置了该项的智能体。 | 从智能体中移除 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`)。 |
| 快速模式（`speed: "fast"`） | 取决于提供商以及智能体的模型 | 设置了该项的智能体必须通过两项检查。<br />**保存智能体时：** 服务器的模型目录必须将该智能体的模型列为接受快速模式的模型。否则，服务器会拒绝保存该智能体。<br />**每个轮次：** 如果提供商不为该模型或部署提供快速模式，该轮次会以错误结束。 | 从智能体中移除 `speed` (csharp, go: `Speed`)。请参阅[接受快速模式的模型](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#models-that-accept-fast-mode)。 |

在 `anthropic` 下，服务器默认会请求 Claude API 的[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)。当安全分类器拒绝某个请求时，另一个 Claude 模型（可能是能力较弱的模型）随后可以回答该请求。由 Anthropic 选择模型，并且不会有事件报告这一变化，但读取会话时会显示该模型的 ID。要关闭回退，请设置 `inference.model_fallback: false`。在其他提供商下，此类请求会以拒绝结束当前轮次。

您的提供商文档会说明哪些模型和区域支持每项功能。自定义工具、MCP 工具以及由 worker 运行的工具在所有提供商下都相同。

### 接受快速模式的模型

默认情况下，服务器的模型目录将四个模型列为接受快速模式的模型：Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8 和 Claude Opus 4.6。对于 Claude Opus 4.6，Claude API 会接受该请求，但[以标准速度运行它](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#checking-which-speed-was-used)。以下 `inference.provider` 值属于例外情况：

* **`foundry`：** 该目录仅列出 Claude Opus 5.5。
* **使用 `format: vertex-raw-predict` 的 `gateway`：** 该目录不列出任何模型。
* **`vertex`：** 该目录不列出任何模型，且任何设置都无法改变这一点。

要更改目录所列的内容，请参阅[模型条目](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#model-entries)。

## 如果您已在运行自托管沙箱

worker 软件是相同的，因此您的 worker 可以连接到您的服务器。[运行您自己的 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#point-a-worker-at-your-server)列出了执行轮询的进程所需的三个值，以及需要首先更改的内容。依赖 webhook 启动的 worker 在此处无法工作。
