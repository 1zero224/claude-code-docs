---
title: 自托管 Claude Managed Agents 中的数据与访问
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access
description: 自托管 Claude Managed Agents 会向您的网络外部发送哪些内容、其日志、跟踪和指标的去向、数据的保留时长、谁可以访问服务器，以及如何将租户相互隔离。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

使用[自托管 Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted)时，您负责运维该服务器、其数据库，以及工作器在其中运行智能体 shell 和文件工具的沙箱。本页面解答安全审查中会提出的问题。服务器的推理提供商是负责响应其模型调用的服务。只有当该提供商是 Claude API 时，服务器才会连接到 Anthropic。Anthropic 无法远程访问服务器或其日志。

## 哪些内容会离开您的网络

### 服务器建立的所有连接

服务器仅连接到以下目标：

* 您的推理提供商
* 服务器的数据库
* 为模型调用签发服务器凭据的服务（如果它从某项服务获取该凭据）
* 云提供商用于列出模型的服务（如果您的推理提供商在单独服务上列出其模型）
* 您允许的 MCP 服务器
* 您允许的 Git 主机，用于会话所使用的代码仓库
* OpenTelemetry 收集器（如果您配置了）
* 您为单点登录列出的身份提供商（如果您配置了）
* 密钥管理器或密钥管理服务（如果您配置了）
* 为服务器签发用于访问该密钥管理器或密钥管理服务的凭据的服务
* 您配置服务器调用的任何其他服务

默认推理提供商是 Claude API。如果您配置了其他推理提供商，模型调用将改为发送到该提供商。只有当 Claude API 是其推理提供商时，服务器才会连接到 Anthropic。

如果推理提供商提供网络搜索和网页获取工具，则由推理提供商运行这些工具。服务器不会为此自行建立任何连接。在您于服务器设置中启用这些工具之前，它们处于关闭状态。

要检查服务器的流量，请将您的出口代理或防火墙日志与此列表进行比对。服务器发往 Claude API 的请求会携带 `User-Agent` 标头。请参阅[标明服务器软件的字段](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#fields-that-name-the-server-software)。

### 工作器可以访问的内容

目标列表仅涵盖服务器。如果您运行[出口网关](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#environment-variable-credentials)，它会连接到您为其列出的主机，并向这些主机发送您为其存储的凭据值。工作器是独立的。工作器拥有您为其运行所在的容器、虚拟机或其他沙箱所允许的任何网络访问权限。智能体通过该工作器运行的每条命令都拥有相同的网络访问权限。

### 服务器向其推理提供商发送的内容

每个模型调用都携带截至当前的对话内容，包括提示、工具输入和工具结果。

当 Claude API 是推理提供商时，每个模型调用还会携带最多三个标头。这些标头包含由您的服务器生成的 ID：

| 标头                                        | 包含的内容         |
| ----------------------------------------- | ------------- |
| `anthropic-managed-agents-session-id`     | 该调用所服务的会话的 ID |
| `anthropic-managed-agents-agent-id`       | 该会话的智能体的 ID   |
| `anthropic-managed-agents-environment-id` | 该会话的环境的 ID    |

ID 不包含任何名称或内容，但确实编码了您的服务器创建它的时间。借助这些标头，Anthropic 可以将用量归属到某个会话、智能体或环境。没有任何设置可以关闭它们。如果 `ANTHROPIC_BASE_URL` 指向代理或网关，这些标头会发送到那里。在任何其他推理提供商下，服务器从不发送这些标头。

当 Claude API 是推理提供商时，服务器还会向 Claude API 核实其用于模型调用的凭据是否有效。它会在启动时进行检查，之后每 60 秒检查一次，除非您更改 `health.check_interval`。这些检查不会运行任何推理，也不携带这三个标头中的任何一个。

### 标明服务器软件的字段

对 Claude API 的每个请求都带有一个 `User-Agent` 标头，用于标明服务器软件及其版本。该标头本身不会标识您的部署。与每个 Claude API 请求一样，API 密钥会标识您的组织。Anthropic 可以使用该标头统计自托管 Managed Agents 的用量。

模型调用还可能在请求正文中以 `metadata.user_id` 的形式携带标签 `cma-server`。该标签在每台服务器上都相同，因此标签本身不会标识您的部署、您的用户或您的会话。当调用到达 Anthropic 时，即使代理移除了 `User-Agent` 标头，Anthropic 也可以使用该标签统计自托管 Managed Agents 的用量。

调用是否携带该标签取决于其去向：

* **服务器会在以下调用中发送该标签：**&#x53D1;送到 Claude API、Microsoft Foundry 以及接受与 Claude API 相同请求的模型网关（model gateway）的调用。
* **服务器从不在以下调用中发送 `metadata.user_id`：**&#x53D1;送到 Amazon Bedrock、Claude Models on Google Cloud (Google managed) 以及采用其他请求格式的模型网关的调用。

如果模型网关或代理将调用转发给其他提供商，`metadata.user_id` 可能会随之一同发送。对于携带 `metadata.user_id` 的调用，您自己的代理或网关必须在请求正文中接受该字段。如果它拒绝该字段，所有模型调用都会失败。

## 日志、跟踪和指标的去向

服务器将日志写入标准错误。当您在 Docker、Kubernetes 或服务管理器下运行服务器时，由该系统从标准错误收集日志。

在您使用标准 OpenTelemetry 环境变量启用追踪和指标之前，服务器不会记录任何追踪或指标。服务器只会将它们发送到这些变量所指向的位置。Prometheus 端点也可以公开指标。请参阅[服务器的监听位置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#where-the-server-listens)。

## 数据的保留时长

您的日志管道、收集器和追踪后端决定日志、追踪和指标的保留时长。服务器不保留任何日志文件，也不存储任何追踪或指标。

服务器将其他所有内容（例如智能体、会话和会话历史记录）存储在您的数据库中。如果您未设置数据库（这是默认情况），服务器会将这些数据保存在内存中，重启后数据将被清除。如果您配置了密钥管理器，服务器会将每个新的[保管库凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)的密钥保存在该密钥管理器中，而不是数据库中。服务器之前存储的密钥仍保留在数据库中。

会话及其历史记录会一直保留在您的数据库中，直到您将其删除。服务器不会因会话的存在时长而删除会话，无论该会话是否已归档。通过 API 删除会话会从您的数据库中移除该会话及其历史记录，但服务器会保留该会话的 ID。会话正在运行时，服务器可以拒绝删除操作。删除操作不会影响其他位置的副本，例如数据库备份、导出的日志和追踪，或智能体写入[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)的内容。在 Kubernetes 上，删除操作还会保留该会话的 `/workspace` 卷，如[释放会话的卷](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#free-a-sessions-volume)中所述。

服务器发送给模型的内容以及模型返回的内容，受您的组织与提供该模型的服务商之间的协议约束。该协议的数据保留条款适用。当由 Claude API 提供模型时，适用您的组织与 Anthropic 之间的数据保留协议和设置。如果您的组织有[零数据保留（ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#zero-data-retention-zdr-scope)安排，该安排以与其他 Messages API 调用相同的方式涵盖这些模型调用。

《API 与数据保留》页面将 Claude Managed Agents 列为[ZDR 不涵盖的范围](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#what-zdr-does-not-cover)。这是因为当由 Anthropic 运行该服务时，Anthropic 会保留会话记录，直到您将其删除。由谁存储会话记录取决于由谁运行该服务：

* **由 Anthropic 运行：**&#x7531; Anthropic 存储会话记录。使用[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)时，您仅托管沙箱，其余部分由 Anthropic 运行。
* **由您通过自托管 Managed Agents 运行：**&#x7531;您自己存储会话记录。

## 谁可以访问服务器

Anthropic 无法远程访问服务器或其日志。您的应用程序和工作器连接到服务器。服务器从不连接到工作器。

### 服务器接受的凭据

服务器接受的凭据均不由 Anthropic 签发，Anthropic 也不持有其中任何凭据。服务器接受的每个凭据都属于以下类型之一：

* 由您设置或创建的凭据
* 由您的身份提供商签发的凭据
* 由服务器自身生成的凭据，例如服务器随每项工作交给工作器的令牌

### 服务器的监听位置

服务器在一个地址和端口上接收 API 请求。服务器还会打开另外两个监听器，每个监听器仅在您启用时才会打开。

| 监听器                                                                                                                                 | 默认启用                                                                                                                   | 默认地址                                                     | 要求提供的内容                                                                                                                               |
| ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| API 请求                                                                                                                              | 是                                                                                                                      | `127.0.0.1:8400`，只有同一台机器可以访问。要更改地址和端口，请设置 `server.addr`。 | 请参阅[服务器接受的凭据](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#credentials-that-the-server-accepts)。 |
| [您服务器上的控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console)，一个网页                                         | 是。要停止提供该网页，请参阅[关闭控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console#turn-off-the-console)。 | 与 API 请求相同的地址和端口                                         | 任何能够访问该页面的人都无需提供任何内容即可加载该页面。要在控制台中执行任何操作，用户需要通过您的身份提供商登录。                                                                             |
| Prometheus 指标端点                                                                                                                     | 否                                                                                                                      | `localhost:9464`，只有同一台机器可以访问                             | 无需凭据                                                                                                                                  |
| 服务器为[出口网关](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#environment-variable-credentials)提供的监听器 | 否                                                                                                                      | 所有接口上的端口 8410                                            | 网关的密钥                                                                                                                                 |

关于最后两个监听器的更多详细信息：

* **Prometheus 指标端点：**&#x5B83;不要求任何凭据，因此谁可以访问它取决于它的监听位置。如果您将 `OTEL_METRICS_EXPORTER` 设置为 `prometheus` 且未设置地址，它将在默认地址上监听。如果您通过服务器 Helm chart 启用它，它将在每个服务器 pod 的端口 9464 上监听。此时，任何能够路由到 pod 地址的对象都可以访问它，除非 NetworkPolicy 阻止了该流量。
* **出口网关的监听器：**&#x51FA;口网关是位于您的沙箱与互联网之间的可选代理。服务器在单独的端口上监听网关。该监听器使用纯 HTTP 通信，其响应携带已存储的凭据值。Compose 栈和服务器 Helm chart 仅允许网关访问它。如果您以其他任何方式运行服务器，也请这样做。

Docker Compose 栈在 Docker 主机的环回地址 `127.0.0.1` 上发布端口 8400。请运行 Docker Engine 28.3.3 或更高版本。在某些较早的版本下，主机所在本地网络中的机器可以访问发布在环回地址上的端口。要在其他地址上发布该端口，请在 `.env` 中设置 `CMA_PUBLISH_ADDRESS`。

## 多租户

每个部署以单一信任级别为一个租户（tenant）提供服务。同一部署中的环境无法将信任级别相互隔离。服务器 API 密钥可以访问每个环境。每个拥有角色的调用方都可以在每个环境中运行会话，并可访问该环境的工作器所能访问的任何网络和文件。

| 要隔离的对象      | 操作方法                                                                                                                                                                                                                                                                                   |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 租户          | 为每个租户提供其自己的部署，包括其自己的数据库、服务器 API 密钥和工作器。                                                                                                                                                                                                                                                |
| 不同信任级别的工作负载 | 为每个信任级别提供其自己的部署，并按照租户的方式进行设置。例如可信代码与不可信代码，或者访问仅部分调用方可以使用的系统的工作器。                                                                                                                                                                                                                       |
| 同一部署中的调用方   | 启用[单点登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#single-sign-on)，将组映射到 `developer` 或 `user` 角色，并将服务器 API 密钥仅留给管理员使用。[各角色可执行的操作](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#what-each-role-can-do)说明了这些调用方可以看到的内容。 |

### 密钥管理器、单点登录和模型网关

如果您的部署使用其中某项服务，也请在该服务中将各部署相互隔离：

* **密钥管理器：**&#x6309;照[密钥管理器限制](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#limits)中的说明，将各部署的密钥相互隔离。
* **单点登录：**&#x4E3A;每个部署设置其他部署均未列出的 `audiences` 条目，使其拒绝其他部署的令牌。在每个信任级别的部署中，仅向该级别所允许的调用方授予角色，且不设置 `default_role`。
* **[模型网关](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#model-gateway)：**&#x8981;区分各部署，请在 `inference.headers` 下为每个部署的某个标头（例如 `x-tenant-id`）设置各自的值。

### 角色不限制的内容

角色仅限制调用方使用其自己的令牌发出的请求。智能体运行的代码可以使用其沙箱持有的任何凭据调用服务器。[环境密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted#the-three-credentials)可以读取其环境中每个会话的事件并发布工具结果。因此，请在各自独立的沙箱中运行每个会话，并确保该沙箱中不包含环境密钥。[自托管 Claude Managed Agents 入门](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart)和[在 Kubernetes 上部署](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes)中的栈就是这样做的。
