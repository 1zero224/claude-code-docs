---
title: 选择智能体环境
url: https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment
description: 决定 Claude Managed Agents 的工具和代码是在 Anthropic 托管的云环境中运行，还是在自托管沙箱中运行，并了解切换后会发生哪些变化。
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

环境决定了您的智能体的工具和代码在何处运行。它们在 “sandbox”（沙箱）中运行。沙箱是一个隔离的工作空间，例如容器，智能体在其中运行命令并处理文件。您有两个选项：

* **云环境：** Anthropic 为您运行沙箱。每个会话都会在 Anthropic 托管的基础设施上获得一个全新的独立 Linux 容器，其中已预装常用的语言、数据库和实用工具。您可以选择其网络规则以及要安装的任何额外软件包。
* **自托管沙箱：** 您在自己的基础设施上运行沙箱。Claude 仍在 Anthropic 一侧运行，并决定接下来调用哪个工具。一个由您运行的程序，称为 “worker”（工作器），在您的机器上执行智能体的 bash 和文件工具调用，并将每个结果发回。它还可以提供自定义工具。

无论您选择哪种方式，智能体定义都是相同的。您在智能体上设置模型、提示和工具，而不是在环境上设置，如[定义您的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)中所述。

## 从云环境开始

如果您不确定该选择哪种方式，或者您是 Claude Managed Agents 的新用户，请像[快速入门](https://platform.claude.com/docs/zh-CN/managed-agents/quickstart)那样，从[云环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments)开始。Anthropic 会为每个会话运行一个隔离的容器，其中已预装常用的语言和工具，因此您无需托管任何内容。仅当工具必须在您控制的基础设施上运行时，才选择自托管沙箱。

## 比较选项

下表比较了这两个选项。

|                   | [云环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments)（默认）                                                                                                                             | [自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes) |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| 工具和代码的运行位置        | Anthropic 托管的沙箱                                                                                                                                                                                           | 您的 worker，位于您的基础设施上                                                                  |
| 哪些内容会到达 Anthropic | [对话历史](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#list-past-events)、[工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/tools#available-tools)、沙箱的文件和状态、技能以及记忆存储 | 对话历史、工具结果、技能以及记忆存储。沙箱的文件和进程保留在您的 worker 上。                                           |
| 您需要运维的内容          | 无                                                                                                                                                                                                         | worker 主机、其安全加固、其网络限制，以及您的 worker 用于身份验证的环境密钥                                        |
| 网络访问范围            | 您在环境上设置的网络规则                                                                                                                                                                                              | 您自己的网络策略                                                                             |
| 文件和 GitHub 仓库挂载   | 由 Anthropic 管理                                                                                                                                                                                            | 由您管理                                                                                 |
| 记忆存储              | 由 Anthropic 挂载到 `/mnt/memory/`                                                                                                                                                                            | 下载到 `/mnt/memory/` 并由 worker 同步                                                      |
| 沙箱生命周期            | 由 Anthropic 管理                                                                                                                                                                                            | 由您管理                                                                                 |
| 何时选择              | 默认选项。您不需要工具在您自己的基础设施上运行。                                                                                                                                                                                  | 工具必须在您控制的基础设施上运行。                                                                    |

## 何时选择自托管沙箱

当智能体需要执行以下任一操作时，自托管沙箱是合适的选择：

* **处理不能离开您网络的数据：** 智能体的文件和进程保留在您的 worker 上。工具返回给模型的内容仍会到达 Managed Agents 服务。
* **访问无法公开路由的内部服务：** 智能体的 bash 和文件工具，以及您的 worker 提供的任何自定义工具，都在您的网络内运行。连接智能体 MCP 服务器的是 Anthropic，而不是您的 worker。私有 MCP 服务器的背后可能是您的内部工单系统或公司 wiki。要访问它，您可以在任一选项下使用 [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)。MCP 隧道目前处于研究预览阶段，需要申请访问权限。请参阅[自托管沙箱与 MCP 隧道](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#self-hosted-sandboxes-and-mcp-tunnels)。
* **在您组织自己的合规和审计控制下运行：** 工具在您配置、记录日志和审计的主机上运行。

## 自托管沙箱不会改变的内容

无论选择哪个选项，对话历史和工具结果都会发送到由 Anthropic 运营的 Managed Agents 服务。模型正是通过这种方式接收每次工具调用的结果。网页搜索和网页抓取工具也在 Anthropic 一侧运行，而不是在沙箱中运行。请参阅[限制网页搜索和网页抓取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。两个选项都不会改变 Managed Agents 的合规覆盖范围，也不会改变其保留对话历史的时长。请参阅 [API 与数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)。

## 使用自托管沙箱时您需要承担和放弃的内容

自托管沙箱使您成为 worker 主机的运维者。您需要加固沙箱镜像、控制网络出站流量、存储和轮换环境密钥，并将各个会话相互隔离。[安全模型](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-security)页面提供了详细信息。

您还将放弃以下云环境功能：

* [`file` 和 `github_repository` 会话资源](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#session-resources)。您需要[自行暂存输入文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#stage-files-for-a-session)，并且还需要负责挂载您自己的 GitHub 仓库。
* [仓库技能发现](https://platform.claude.com/docs/zh-CN/managed-agents/skills#load-skills-from-a-github-repository)。
* 保管库中的[环境变量凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults#add-a-credential)。
* 云沙箱的[预装运行时和软件包](https://platform.claude.com/docs/zh-CN/managed-agents/cloud-sandboxes-reference)。自托管沙箱中包含的是您的 worker 所提供的内容。

## Claude Platform on AWS 上的差异

如果您通过 Claude Platform on AWS 使用 Claude Managed Agents，服务仍由 Anthropic 运营。在该平台上，自托管沙箱无法使用记忆存储，worker 不使用环境密钥，并且 MCP 隧道不可用。请参阅 [Claude Platform on AWS 上的 Claude Managed Agents](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#claude-managed-agents) 和[不支持的功能](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)。

## Claude Models on Google Cloud (Anthropic managed) 上的差异

如果您通过 Claude Models on Google Cloud (Anthropic managed) 使用 Claude Managed Agents，服务仍由 Anthropic 运营。在该平台上，自托管沙箱中记忆存储不可用，并且 MCP 隧道不可用。worker 使用 Google Cloud 凭据而非环境密钥进行身份验证。您需要自行编写 worker，因为 `ant` CLI 以及 Python、TypeScript 和 Go SDK 中的预构建 worker 在该平台上无法使用。请参阅 Claude Models on Google Cloud (Anthropic managed) 指南中的[功能支持](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#feature-support)。

## 从云环境切换到自托管沙箱

要切换到自托管沙箱，您需要保留智能体，创建一个 `self_hosted` 环境，并在新的会话和部署中引用其 ID。

* **可沿用的内容：** 智能体定义、您的应用程序使用的凭据（API 密钥、AWS 凭据或 Google Cloud 凭据）、您的技能、您的 MCP 服务器，以及您的[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)。
* **新增的内容：** 一个 `self_hosted` 环境、一个您在 Claude Console 中生成的环境密钥，以及一个由您运行的 worker。[自托管沙箱快速入门](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#quickstart)涵盖了这三项内容。
* **停止工作的内容：** [使用自托管沙箱时您需要承担和放弃的内容](https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment#what-you-take-on-and-give-up-with-a-self-hosted-sandbox)中列出的功能。

在 Claude Platform on AWS 上，记忆存储不会延续到自托管沙箱，并且 worker 不使用环境密钥。请参阅 [Claude Platform on AWS 上的差异](https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment#differences-on-claude-platform-on-aws)。

在 Claude Models on Google Cloud (Anthropic managed) 上，自托管沙箱中记忆存储不可用，您需要自行编写 worker，并且该 worker 不使用环境密钥。请参阅 [Claude Models on Google Cloud (Anthropic managed) 上的差异](https://platform.claude.com/docs/zh-CN/managed-agents/choosing-an-environment#differences-on-claude-models-on-google-cloud-anthropic-managed)。
