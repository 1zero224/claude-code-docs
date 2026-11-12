# Claude Cowork 架构概览

本文介绍了 Claude Cowork 的运行位置、每种执行模式的隔离方式，以及可用的管理员控制措施。

## 可用性

本文适用于企业管理员，但此处描述的架构在所有计划中都相同。末尾的设备级管理员控制适用于团队和企业计划。

Claude Cowork 在 Pro、Max 和 Ultra 计划中在云端运行，在团队计划中运行（测试版），以及在企业计划中当所有者启用时运行。

新的 Claude 体验结合了聊天和 Cowork，正在向 Pro、Max 和 Ultra 计划推出。在团队计划（测试版）和企业计划中，它在启用 **聊天和 Cowork 统一** 的地方可用。尚未启用的组织将聊天和 Cowork 保持为单独的选项，因此本文仍然适用。企业所有者可以在 **[为您的企业组织设置和管理新的统一 Claude 体验](https://support.claude.com/en/articles/17444462)** 中了解更多信息。

---

## Claude Cowork 的运行位置

Cowork 会话默认在云端运行：代理循环和代码执行在 Anthropic 的服务器上运行，会话和文件保存到成员的 Claude 账户。

本地执行仍可用于现有桌面部署：代理循环和代码执行在成员的设备上运行，如下所述。

### 云会话架构

在云中的会话中，代理循环和代码执行在 Anthropic 管理的基础设施上的隔离、临时沙箱中运行。每个会话都有自己的沙箱，在会话开始时创建，在会话结束时销毁，沙箱彼此之间或跨组织不共享状态。此基础设施与 Anthropic 的企业、研究和模型训练环境分开保存。

云中会话的关键属性：

- **默认无法访问您的网络。**沙箱无法到达私有、内部、链接本地或云元数据地址，也无法到达 Anthropic 内部系统，因此无法用于进入您的网络。

- **网络访问遵循您现有的策略。**云会话使用与管理本地 Cowork 和聊天的网络访问设置相同的设置。对于企业组织，默认不允许网络访问。

- **出站流量在沙箱外强制执行。**所有离开沙箱的流量都通过沙箱无法重新配置或绕过的强制代理，只有允许列表中的目标可以到达。

- **仅短期凭证。**沙箱仅保存在数小时内过期的会话范围令牌。连接器授权令牌永远不会进入沙箱；连接器调用在服务器端进行。

- **数据层的租户隔离。**每条存储的记录都限定在您的组织和账户范围内。

当云中的会话需要用户设备上的内容（如本地文件或浏览器）时，请求通过该设备上的 Claude Desktop 应用程序通过 Anthropic 代理连接进行。本地文件访问仅限于成员在桌面上连接的文件夹，每个本地工具调用在运行前都会根据成员的权限进行检查。如果桌面应用离线，云中的会话无法到达设备。

由于云中的会话在 Anthropic 的服务器上运行，代理的工作（包括它通过桌面应用打开的任何本地文件）在 Anthropic 的服务器上处理，而不是保留在设备上。对话数据按照与其他团队和企业数据相同的商业承诺处理，不用于训练 Claude。

### 本地会话架构

本地会话适用于现有桌面部署（在 Pro、Max 和 Ultra 计划中，仅限 **[在 2026 年 10 月 6 日之前在设备上启动的任务和计划任务](https://support.claude.com/en/articles/15520349-use-claude-cowork-on-web-desktop-and-mobile#h_f951c27c48)**），并在成员的设备上使用两个执行环境：

- **代理循环在设备上本地运行。**这包括 Claude 的对话处理、连接文件夹中的文件读写、网络获取和本地插件 MCP 服务器。访问由应用层权限系统控制，该系统强制执行成员的连接文件夹规则和您组织的网络出站设置。

- **代码执行在隔离的虚拟机 (VM) 中运行。**Shell 命令和 Claude 编写的任何代码在专用 Linux VM 内执行，由平台的虚拟机管理程序（macOS 上的 Apple Virtualization.framework，Windows 上的 Hyper-V）与主机操作系统隔离。VM 强制执行自己的网络出站过滤、系统调用限制和每会话用户隔离。

有关详细的技术概览，请参阅我们信任中心上的 **[Claude Cowork 桌面安全架构概览](https://trust.anthropic.com/resources?s=2a7bbzo1lyymvdt551q7kl&name=claude-cowork-desktop-security-architecture-overview)**。

---

## 托管设备的管理员控制

两个 MDM 密钥让您可以限制托管设备上 Cowork 的范围。两者都是通过您的 MDM 解决方案应用的设备级设置，而不是从组织设置应用。

- **禁用本地 MCP 服务器：**将 `isLocalDevMcpEnabled` 设置为 false 以禁用插件捆绑和本地配置的 MCP 服务器。

- **禁用桌面扩展：**将 `isDesktopExtensionEnabled` 设置为 false 以阻止 MCPB 和 DXT 扩展服务器运行。

两个控制都在 **[Claude Desktop 企业配置](https://support.claude.com/en/articles/12622667-enterprise-configuration-for-claude-desktop)** 中描述。

这些 MDM 密钥管理 Claude Desktop 应用，因此它们适用于本地会话和云中会话通过桌面应用到达的任何内容。本地 MCP 服务器不在云中的会话中运行。

**组织设置 > Cowork** 中的组织范围 Cowork 切换（**为您的组织启用**）控制 Cowork 是否可用。上面的设备级控制仅在启用 Cowork 时适用。

---

## 云中会话的组织控制

除了组织范围的 Cowork 切换外，云中的会话在组织设置中有自己的控制：

- 为组织打开或关闭云中的会话，同时保持本地桌面 Cowork 可用。

- 设置网络访问策略，确定云中的会话可以到达哪些目标。

- 通过关闭持久的"始终允许"来要求对每个权限门控工具调用进行新批准，并控制成员是否可以运行会话而无需每次调用批准提示。

- 要求云中会话的受信任设备注册和最近登录。启用后，这适用于组织中云中的每个会话。

上面的设备级 MDM 密钥管理 Claude Desktop 应用，因此它们也适用于云中的会话通过应用可以到达的内容。在托管设备上禁用本地 MCP 服务器后，只有文件夹限制的桌面文件工具可用于云中的会话。

---

## 常见问题

### 如果成员的设备无法启动 VM 会怎样？

这适用于本地会话。Cowork 在 VM 不可用时继续运行文件和网络工具。Shell 命令和代码执行报告"工作区不可用"，直到 VM 恢复。

### 云中的会话是否可以访问用户的设备或我们的网络？

默认情况下不能。云中的会话在 Anthropic 服务器上的隔离环境中运行，在您的网络之外，无法到达私有或内部地址。云中的会话仅通过该设备上的 Claude Desktop 应用到达成员的本地文件或浏览器，仅限于成员已连接的文件夹，且仅在应用在线时。

### Cowork 活动是否在合规性 API 或 OpenTelemetry 中捕获？

是的。通过 Claude、Claude Desktop 和 Claude Mobile 的 Cowork 在合规性 API 中捕获。了解更多关于 **[在合规性 API 中检索远程会话](https://platform.claude.com/docs/en/manage-claude/compliance-content-data)** 的信息。
​
如果您是团队或企业计划管理员，您可以 **[使用 OpenTelemetry (OTel) 监控您组织中的 Claude Cowork 活动](https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry)**。

### 端点检测 (EDR) 工具能否检查 VM 内的活动？

对于本地会话，可以，如果您进行设置。VM 在设计上与基于主机的安全工具隔离，因此主机上的 EDR 传感器无法看到其内部。管理员可以从批准的供应商在 VM 内运行传感器。这在 macOS 和 Windows 上处于测试版。了解更多关于 **[在 Claude Cowork 中运行您的端点检测和响应 (EDR) 传感器](https://support.claude.com/en/articles/17828685-run-your-endpoint-detection-and-response-edr-sensor-in-claude-cowork)** 的信息。

云中的会话完全在您的端点之外运行，因此 EDR 工具无法观察它们。如果您的合规性态势取决于端点可见性，请在推出 Cowork 之前考虑这一点。