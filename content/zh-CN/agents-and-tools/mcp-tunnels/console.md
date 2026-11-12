---
title: 在 Claude Console 中管理隧道
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console
description: 在 Claude Console 中创建隧道、注册 CA 证书、获取隧道令牌，并将通过隧道连接的 MCP 服务器附加到智能体。
---

<Note>
  “MCP tunnels”（MCP 隧道）目前处于"research preview"（研究预览）阶段。[申请访问权限](https://claude.com/form/mcp-tunnels)即可试用。
</Note>

本页介绍 MCP 隧道部署中 Console 一侧的内容：创建隧道、注册您的 CA 证书、获取隧道令牌，以及将[上游 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)附加到智能体。[使用 Helm 部署 MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm)和[使用 Docker Compose 部署 MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-compose)介绍了如何在您的网络内部运行[隧道栈](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)。

## 前提条件

* **一个或多个 MCP 服务器**，运行在您的私有网络中。隧道将流量路由到这些服务器；它并不托管它们。请参阅[远程 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/remote-mcp-servers)了解您可以部署的示例。

* **具有"管理隧道"权限的 Console 角色**，以便您可以创建和归档隧道、轮换令牌以及管理证书。组织管理员和所有者默认拥有该权限；自定义角色和按账户授予的权限也可以包含它。没有该权限的角色对 **MCP 隧道**页面和隧道详情仅有只读访问权限。

* **一种让您的栈向 Tunnels API 进行身份验证的方式。**&#x8BF7;选择其一：

  * **[编程访问](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#credential-provisioning)（推荐）。**&#x5728;创建隧道期间设置 [Workload Identity Federation](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)（工作负载身份联合），使您的栈能够从您的身份提供商签发短期 API 令牌、获取隧道令牌，并自动生成和注册 CA 证书。需要管理联合规则的权限、一个已注册的 OIDC 颁发者，以及一条具有 `workspace:manage_tunnels` 作用域的联合规则。
  * **[手动](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#credential-provisioning)。**&#x8DF3;过编程访问。创建隧道后，[获取隧道令牌](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#get-the-connection-details)，自行生成并[注册 CA 证书](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#add-a-ca-certificate)，然后将令牌和您的服务器证书作为机密提供给您的隧道栈。

## 创建隧道

<Steps>
  <Step title="打开 MCP 隧道页面">
    在 Claude Console 中，前往 [MCP 隧道](https://platform.claude.com/settings/workspaces/default/mcp-tunnels)页面（`platform.claude.com/settings/workspaces/default/mcp-tunnels`）。隧道的作用域为工作区；新隧道属于 Console 中当前选定的工作区。请在侧边栏的工作区选择器中选择您需要的工作区。
  </Step>

  <Step title="为隧道命名">
    点击**新建隧道**，并在**创建隧道**对话框中输入名称。名称为必填项，用于在列表、详情页和智能体 MCP 服务器选择器中标识该隧道。系统会自动分配一个形如 `abcd1234.tunnel.anthropic.com` 的域名。
  </Step>

  <Step title="可选：设置编程访问">
    如果您的角色可以管理联合规则，则会出现一个**设置程序化访问**开关（默认关闭）。如果不能，Console 会在该位置显示一条通知，您的隧道栈将改用手动流程。无论哪种方式，创建流程的其余部分都相同。

    编程访问依赖于[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)；如果您不熟悉联合颁发者、规则和服务账户，请先阅读该页面。要打开该开关，您需要：

    1. **一个已注册的 OIDC 颁发者**，对应您的栈出示令牌所用的身份提供商（例如 Kubernetes 集群、AWS IAM、Google Cloud 或 GitHub Actions）。如果您的组织还没有，请在 Claude Console 的[工作负载身份](https://platform.claude.com/settings/workload-identity-federation)页面（`platform.claude.com/settings/workload-identity-federation`）的**颁发者**选项卡中注册一个。
    2. **一条具有 `workspace:manage_tunnels` 作用域的联合规则。** 打开开关后会显示**联合规则**选择器。选择一条具有该作用域的现有规则，或点击**创建联合规则**直接创建一条。
    3. **已将该规则的服务账户添加到此工作区。** Tunnels API 根据服务账户的工作区成员资格进行授权。如果您在组织默认工作区以外的工作区中创建隧道，请[将服务账户添加到该工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#role-inheritance)，并在部署时传入工作区 ID（Helm 使用 `api.wif.workspaceId`，Compose 使用 `ANTHROPIC_WORKSPACE_ID`）。

    完全支持跳过此步骤；两份部署指南都有一个**不使用编程访问**选项卡。
  </Step>

  <Step title="创建隧道">
    点击**创建隧道**。Console 会配置该隧道并打开详情页面。
  </Step>

  <Step title="记录部署标识符">
    两种部署路径都需要：

    * **隧道 ID**（`tnl_...`），显示在隧道详情页面上。
    * **隧道域名**（`abcd1234.tunnel.anthropic.com`），显示在隧道详情页面上。用作代理的 `tunnel_domain`，并用于服务器证书的 SAN 中。

    您还需要什么取决于[凭据配置模式](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#credential-provisioning)：

    | 使用编程访问                                                                                                                                                                                                                                          | 不使用编程访问                                                                                                                                                      |
    | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | 您所选规则的**联合规则 ID**（`fdrl_...`）。该规则属于组织级别，不存储在隧道上；可在 Claude Console 的[工作负载身份](https://platform.claude.com/settings/workload-identity-federation?tab=rules)页面（`platform.claude.com/settings/workload-identity-federation?tab=rules`）的**规则**选项卡中找到。 | **隧道令牌**，通过详情页面上**令牌**旁边的眼睛图标显示。请将其视为机密。请参阅[获取连接详情](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#get-the-connection-details)。 |
    | **组织 ID**（一个 UUID），显示在 Claude Console 的[组织](https://platform.claude.com/settings/organization)页面（`platform.claude.com/settings/organization`）上。                                                                                                 | 一个由您生成并[在隧道上注册](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#add-a-ca-certificate)的 **CA 证书**。                                |

    使用编程访问时，您的栈通过 Tunnels API 获取隧道令牌，在本地生成 CA 和服务器证书（私钥永远不会离开您的环境），并且仅向 Anthropic 注册 CA 的公共证书。您仍然负责保护私钥的安全，并在服务器证书过期之前对其进行续期。
  </Step>
</Steps>

您的组织最多可以拥有 10 个活跃隧道。创建隧道本身不会建立任何连接；只有当您的栈使用隧道令牌接入并且已注册 CA 证书后，才会建立连接。

## 获取连接详情

打开隧道。详情页面显示一个包含域名和令牌的**连接**部分，以及一个**证书**部分。

| 字段     | 描述                                                                       |
| ------ | ------------------------------------------------------------------------ |
| **域名** | 复制分配的 `abcd1234.tunnel.anthropic.com` 值。您代理的路由是此域名的子域名。                  |
| **令牌** | 点击眼睛图标（**显示令牌**）获取隧道令牌，然后使用复制图标将其复制到您隧道栈的机密存储中。点击**轮换令牌**可使当前令牌失效并签发新令牌。 |

<Note>
  每次显示和轮换都会记录在您组织的 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 活动日志中。轮换不会切断已建立的 cloudflared 连接，因此您可以先轮换，再使用新值重新部署，并让旧连接自然排空。
</Note>

## 添加 CA 证书

Anthropic 根据您在隧道上注册的 CA 证书来验证到您[代理](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)的[内层 TLS](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)。没有活动证书的隧道无法接受连接，并且在注册证书之前不会出现在智能体 MCP 服务器选择器中。

<Steps>
  <Step title="找到证书部分">
    在隧道的详情页面上，滚动到**证书**部分并点击**添加证书**。
  </Step>

  <Step title="提供证书">
    点击**选择文件**选择 `.pem`、`.crt` 或 `.cer` 文件，将文件拖到文本区域，或直接粘贴 PEM 块。模态窗口会拒绝私钥内容以及非 `-----BEGIN CERTIFICATE-----` 块的内容。文件大小不得超过 8 kB。
  </Step>

  <Step title="添加证书">
    点击**添加证书**。指纹和到期时间会出现在证书列表中，并且该部分标题上的槽位计数会递增。
  </Step>
</Steps>

一个隧道最多可持有两个活动证书，以便您可以在不停机的情况下进行轮换：在旧证书旁注册新证书，使用新密钥对重新部署您的代理，确认流量正常流动，然后点击旧证书所在行的**撤销**。已吊销的证书仍会在列表中显示，并带有**已撤销**标记。

## 部署隧道栈

隧道已存在于 Console 中，但在隧道栈于您的网络内部运行并使用隧道令牌拨入之前，不会有任何流量流动。请按照以下部署指南之一进行操作：

<CardGroup cols={2}>
  <Card title="使用 Docker Compose 部署" icon="cube" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-compose">
    在单台主机上运行隧道栈。涵盖编程访问和手动两种流程。
  </Card>

  <Card title="使用 Helm 部署" icon="stack" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm">
    在 Kubernetes 集群上运行隧道栈。涵盖编程访问和手动两种流程。
  </Card>
</CardGroup>

## 在智能体中使用隧道

一旦您的栈正在运行，即可在创建 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 会话时，将通过隧道连接的 MCP 服务器添加到智能体。如需改为从 Messages API 调用相同的服务器，请参阅[使用通过隧道连接的 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview#use-the-tunneled-mcp-servers)。

<Steps>
  <Step title="打开创建会话对话框">
    在 Claude Console 中，前往[会话](https://platform.claude.com/sessions)页面（`platform.claude.com/sessions`）。在侧边栏的工作区选择器中，选择包含该隧道的工作区。点击**创建会话**。
  </Step>

  <Step title="创建新智能体">
    从**智能体**下拉菜单中选择**新建智能体**，因为该对话框只能将 MCP 服务器添加到新智能体中。在**名称**字段中输入智能体的名称。
  </Step>

  <Step title="添加 MCP 服务器">
    在 **MCP 服务器**下，点击 **MCP 服务器**。在出现的卡片上，打开**选择服务器**下拉菜单，并从列表顶部选择您的隧道。只有所选工作区中拥有[活跃 CA 证书](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#add-a-ca-certificate)的隧道才会列出。
  </Step>

  <Step title="输入子域名和路径">
    在**子域名**字段中，输入您代理的 `routes` 映射发送到该 MCP 服务器的子域名，例如 `wiki`。在**路径**字段中，输入该 MCP 服务器所需的路径，例如 `mcp`。检查**解析为**行是否在隧道域名下显示了您的子域名和路径，例如 `https://wiki.abcd1234.tunnel.anthropic.com/mcp`。
  </Step>

  <Step title="如果 MCP 服务器需要凭据，请添加凭据">
    隧道负责将流量承载到上游 MCP 服务器，但不会向其进行身份验证。如果服务器需要 OAuth 或 bearer 令牌，请点击卡片上的**连接**并将凭据保存到 vault 中。如果卡片上显示的是 **Needs vault**，请前往**凭据保管库**并添加一个 vault，其中包含用于**解析为** URL 的凭据。如果服务器不需要身份验证，请忽略卡片上的**连接**或 **Needs vault**。要了解 vault 的工作原理，请参阅[使用 vault 进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)。
  </Step>

  <Step title="选择环境">
    在 **MCP 服务器**上方的**环境**字段中，选择一个环境。如果该字段下方没有出现警告，请继续下一步。如果警告提示该环境阻止 MCP 服务器，请点击**编辑环境**，这会在新的浏览器标签页中打开该环境。在**网络**下，将**解析为**行中的主机（例如 `wiki.abcd1234.tunnel.anthropic.com`）添加到**已允许的主机**。按照[限制网络](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#limit-the-network)部分的建议，保持**允许 MCP 服务器**复选框未选中。点击**保存**，然后返回包含该对话框的浏览器标签页。
  </Step>

  <Step title="创建会话">
    在**凭据保管库**下，移除会话不需要的所有 vault。如果仍有 vault 保留，请选中确认您有权使用它的复选框。在对话框中，点击**创建会话**。Console 会保存该智能体并打开会话。
  </Step>

  <Step title="测试连接">
    让智能体使用该 MCP 服务器的某个工具。您应该会看到一次工具调用及其结果。如果智能体无法访问该服务器，请参阅[MCP 隧道故障排查](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/troubleshooting)。
  </Step>
</Steps>

## 归档隧道

归档会立即使隧道停止接受连接，并且是永久性的。

在 Claude Console 中，前往 [MCP 隧道](https://platform.claude.com/settings/workspaces/default/mcp-tunnels)页面（`platform.claude.com/settings/workspaces/default/mcp-tunnels`）。在侧边栏的工作区选择器中，选择包含该隧道的工作区。打开该隧道的行菜单并选择**归档**。当您按**已归档**或**全部**筛选列表时，已归档的隧道仍然可见。

## 后续步骤

<CardGroup cols={2}>
  <Card title="使用 Helm 部署" icon="stack" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm">
    使用 Anthropic Helm chart 在 Kubernetes 集群上安装。
  </Card>

  <Card title="安全" icon="lock" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/security">
    加固指南、凭据轮换和入侵响应。
  </Card>
</CardGroup>
