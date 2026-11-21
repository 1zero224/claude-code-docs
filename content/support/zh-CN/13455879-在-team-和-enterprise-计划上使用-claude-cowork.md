# 在 Team 和 Enterprise 计划上使用 Claude Cowork

本文介绍了使用 Claude Cowork 的 Team 和 Enterprise 组织的重要限制和注意事项。

## 可用性

Claude Cowork 在付费计划上可用（Pro、Max、Ultra、Team 和 Enterprise）：

- **Claude Desktop for macOS、Windows 和 Linux（测试版）：**所有付费计划。在 **claude.com/download** 下载或更新。

- **Web 和 Claude Mobile：**Pro、Max、Ultra 和 Team 计划（Team 上处于测试版），以及当所有者启用时的 Enterprise 计划。

- **Claude in Chrome 侧边栏：**所有付费计划。在 Enterprise 计划上，所有者需要在云中启用 Claude in Chrome 和 Cowork。请参阅 **[Chrome 中的 Claude 入门](https://support.claude.com/en/articles/12012173)**。

结合聊天和 Cowork 的新 Claude 体验正在向 Pro、Max 和 Ultra 计划推出。在 Team 和 Enterprise 计划上，当**聊天和 Cowork 统一**打开时可用。尚未打开的组织将聊天和 Cowork 保持为单独的选项，因此本文仍然适用。所有者可以在适用于您的计划的文章中了解更多信息：

- **[为您的 Team 组织设置和管理新的统一 Claude 体验](https://support.claude.com/en/articles/18342806)**

- **[为您的 Enterprise 组织设置和管理新的统一 Claude 体验](https://support.claude.com/en/articles/17444462)**

---

## Cowork 运行位置

在 Team 和 Enterprise 计划上，Cowork 会话可以在两个位置运行：

- **云中的会话：**在云中运行 Cowork 让成员可以在 Anthropic 的基础设施上而不是他们自己的机器上运行任务。这意味着工作可以跨桌面、Web 和移动设备继续进行，计划的任务在笔记本电脑关闭且没有设备在线时运行。

- **本地会话：**Claude 的工作在用户的计算机上运行，代码在隔离的虚拟机中。

## 管理员控制

Claude Cowork 默认启用，但组织所有者可以手动禁用它。在已启用 HIPAA 的组织中，Cowork 默认关闭。所有者可以在**[组织设置 > Cowork](https://claude.ai/admin-settings/cowork)** 中打开 Cowork。

### 启用或禁用 Cowork

1. 以所有者或主要所有者身份登录您的 Team 或 Enterprise 组织。

2. 导航到**[组织设置 > Cowork](https://claude.ai/admin-settings/cowork)**。

3. 在 **Cowork** 下找到**为您的组织启用**切换开关。

4. 切换关闭以为组织中的所有用户禁用 Cowork。

此切换控制 Cowork 是否可用。会话是否可以在云中运行是一个单独的控制。

**注意：**这是一个组织范围的设置。在 Enterprise 计划上，您可以使用组和自定义角色为特定团队启用 Cowork。请参阅下面的**[访问控制](#h_8465b1b558)**。

### 启用或禁用云中的会话

对于 Team 和 Enterprise 计划，在**[组织设置 > Cowork](https://claude.ai/admin-settings/cowork)** 中的“在云中运行 Cowork”下有一个单独的组织范围切换开关。

- **Team 计划：**默认启用。所有者可以随时从“在云中运行 Cowork”切换开关关闭它。

- **Enterprise 计划：**默认关闭。所有者打开“在云中运行 Cowork”，然后使用自定义角色向组授予 Cowork in the cloud 功能。请参阅**[在 Enterprise 计划上管理自定义角色](https://support.claude.com/en/articles/13930452-manage-custom-roles-on-enterprise-plans)**。应用于 Claude Code（本地模式）和 Cowork（本地模式）的 HIPAA 配置后，云中的 Cowork 不可用，没有设置可以打开它。

### 启用或禁用内置浏览器

Claude 可以通过两种方式在 Cowork 中使用网络：Claude Desktop 应用中内置的浏览器，或通过 Claude in Chrome 扩展程序使用用户自己的 Chrome 浏览器。您可以启用一个、两个或都不启用。

- **内置浏览器：**从**[组织设置 > 功能](https://claude.ai/admin-settings/capabilities)**控制。在 Team 计划上，默认启用。Team 所有者可以随时关闭它。在 Enterprise 计划上，默认启用，除非您已关闭它。在已启用 HIPAA 的组织中，内置浏览器保持关闭状态，直到所有者在**[组织设置 > 功能](https://claude.ai/admin-settings/capabilities)**中打开它。应用**[HIPAA 配置到 Claude Code（本地模式）和 Cowork（本地模式）](https://support.claude.com/en/articles/17318731)**后，内置浏览器不可用，没有设置可以打开它。关闭时，用户无法打开内置浏览器，Claude 也无法使用它。

- **Claude in Chrome：**从**[组织设置 > Claude in Chrome](https://claude.ai/admin-settings/browser-extension)** 控制，用户的浏览器仍需要部署或安装扩展程序。请参阅**[Claude in Chrome 管理员控制](https://support.claude.com/en/articles/13065128)**。

两者都运行相同的安全层：高风险网站的黑名单和每个操作的安全检查。这些自动检查可以看到最新的屏幕截图，Anthropic 可以在其检查记录中保留该屏幕截图的副本。内置浏览器需要 Claude Desktop 应用打开并在线；Claude in Chrome 需要在用户的浏览器中安装扩展程序。浏览器流量来自运行桌面应用的用户机器。对于网站运营商来说，它看起来像来自该设备的流量，即使会话是从 Web 或移动设备控制的。

在**[在 Claude Cowork 中使用内置浏览器](https://support.claude.com/en/articles/16607400)**中了解更多。

### 计算机使用（测试版）

在 Team 和 Enterprise 计划上，计算机使用在使用“Cowork”选项启动的任务中不起作用。一旦所有者打开它，它就可以在新的 Claude 体验中工作。这正在逐步向组织推出。在**[为 Team 和 Enterprise 计划设置计算机使用](https://support.claude.com/en/articles/18483812-set-up-computer-use-for-team-and-enterprise-plans)**中了解更多。

### 自动模式可用性

**[组织设置 > 功能](https://claude.ai/admin-settings/capabilities)**中的两个组织设置控制 Cowork 中的“自动批准”模式：

- **允许“自动批准”模式**控制成员是否可以使用该模式。此设置默认启用，因此除非您关闭它，否则该模式对您的成员可用。关闭时，“自动批准”不会出现在您的成员的模式选择器中。

- **在“自动批准”模式下启动新任务**控制新聊天和任务是否可以在该模式下启动。除非您关闭它或**允许“自动批准”模式**关闭，否则它处于启用状态。当“自动批准”作为 Team 计划的默认值推出时，它会生效。在 Enterprise 计划上，Chrome 中 Cowork 侧边栏外的新任务仍在“手动批准”中启动。在 Chrome 中的 Cowork 侧边栏中，新任务已经在“自动批准”中启动，此设置现在适用于此处。关闭它以保持“手动批准”作为默认值。已经选择模式的用户保持该模式，计划的任务不受影响。

成员可以在任何会话中切换模式。

在**[Claude Cowork 入门](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork#h_e1353133dd)**中了解有关模式如何不同的更多信息。

### 连接器工具批准

**[组织设置 > Cowork](https://claude.ai/admin-settings/cowork)**（在**权限**下）中的组织设置**允许连接器工具“始终允许”**控制成员是否可以跳过 Cowork 中写入功能连接器工具的每任务批准。此设置默认关闭。

设置关闭时：

- “允许所有任务”选项在 Cowork 的批准对话框中显示为灰显，即使您的组织范围工具策略允许这些工具。

- 之前保存的写入工具始终允许首选项不被遵守。成员在每个任务中批准这些工具，直到设置打开。

只有当连接器将只读工具注释为只读时，只读工具才被豁免。大多数自定义连接器不注释其工具，因此这些连接器上的每个工具都受到限制。

在 Enterprise 计划上，此设置与自定义角色授予一起工作，最严格的层获胜。角色授予无法覆盖它。有关完整的分层模型，请参阅**[在 Enterprise 计划上管理自定义角色](https://support.claude.com/en/articles/13930452-manage-custom-roles-on-enterprise-plans#h_979e558d00)**。

### 插件

插件包含在 Cowork 中，由相同的管理员切换控制——没有单独的设置来管理 Cowork 中的插件访问。

有关成员可以对插件执行的操作的详细信息，请参阅**[在 Claude 中使用插件](https://support.claude.com/en/articles/13837440-use-plugins-in-cowork)**。

---

## 项目

Cowork 中的项目让用户将任务组织到专用工作区中，具有自己的文件、链接、说明和记忆。项目对所有 Cowork 用户可用。没有针对项目的单独管理员控制，因此所有者目前无法在组织级别限制项目创建。

项目在成员使用 Cowork 的任何地方都可用。与本地文件夹关联的项目仅支持桌面上的 Cowork 会话。对于本地会话，项目数据存储在用户的计算机上；对于云中的会话，项目与成员的 Claude 账户一起保存。有关完整详细信息，请参阅**[在 Cowork 中使用项目组织您的任务](https://support.claude.com/en/articles/14116274-organize-your-tasks-with-projects-in-cowork)**。

---

## 为您的组织管理插件

所有者可以创建插件市场来在其组织中分发精选插件。这让您可以控制用户在聊天、Cowork 和 Claude Code 中看到和使用哪些插件（对于使用其 Claude 账户登录 Claude Code 的用户）。

- **默认安装**——自动为组织中的每个人添加。成员可以选择关闭它。

- **可供安装**——出现在**发现**选项卡上，供成员自行添加。

- **必需**——自动为所有成员安装。成员无法关闭或删除它。

- **不可用**——对成员隐藏。对于暂存或弃用插件很有用。

在 Enterprise 计划上，管理员还可以为特定组覆盖这些首选项——例如，为一个团队自动安装插件，同时对其他所有人隐藏它。有关详细信息，请参阅**[为您的组织管理插件](https://support.claude.com/en/articles/13837433-manage-cowork-plugins-for-your-organization)**。

---

## 公司品牌

Cowork 现在展示您的组织品牌，包括为您的团队定制的重新设计的主屏幕。Team 和 Enterprise 所有者可以在**组织设置**中配置品牌。

---

## 安全、合规和监控

**注意：**有关最新和最全面的指南，请参阅**[Cowork 安全最佳实践](https://trust.anthropic.com/resources?s=uukz8hyx7jmdmo80lys36s&name=claude-cowork-security-best-practices)**。

**重要：**在 HIPAA 就绪的 Enterprise 计划上，在主要所有者将 HIPAA 配置应用于 Claude Code（本地模式）和 Cowork（本地模式）后，您的业务关联协议 (BAA) 涵盖 Claude Desktop 中的 Cowork 会话。没有该配置，Cowork 可用但不在您的 BAA 下。应用该配置后，云中的 Cowork 不可用。请参阅**[在 HIPAA 就绪的 Enterprise 计划上使用 Claude Code（本地模式）和 Cowork（本地模式）](https://support.claude.com/en/articles/17318731)**。

### 监控

**合规 API**

通过 Claude、Claude Desktop 和 Claude Mobile 的 Cowork 会话在合规 API 中被捕获。了解有关**[在合规 API 中检索远程会话](https://platform.claude.com/docs/en/manage-claude/compliance-content-data)**的更多信息。

**本地对话存储**

对于本地会话，Cowork 在用户的计算机上存储对话历史。此数据不受 Anthropic 的标准**[数据保留政策](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data)**约束，管理员无法集中管理或删除它。Claude Enterprise 管理员可以通过合规 API 检索此内容。本地会话的删除端点尚不可用。在已启用 HIPAA 并打开合规 API 的组织中，Anthropic 捕获在成员计算机上运行的 Cowork 会话，以便您的组织可以通过合规 API 检索它们。合规 API 返回最近 30 天的会话。如果您的组织的保留期短于 30 天，合规 API 会返回该期间的会话。

对于云中的会话，会话和文件保存到成员的 Claude 账户。

**OpenTelemetry**

Team 和 Enterprise 所有者可以通过 OpenTelemetry 将 Cowork 事件流式传输到您的 SIEM 和可观测性工具。这为安全团队提供了对工具调用、文件访问、人工批准决定等的可见性。它不能替代合规目的的审计日志。

有关设置、支持的事件和安全考虑，请参阅**[使用 OpenTelemetry 监控 Cowork 活动](https://support.claude.com/en/articles/14477985-monitor-cowork-activity-with-opentelemetry)**。您也可以参考我们 Claude 文档中的**[监控](https://claude.com/docs/cowork/monitoring)**。

**端点检测和响应 (EDR)**

对于本地会话，管理员可以在用户计算机上的虚拟机内运行其组织的 EDR 传感器，以便虚拟机中的活动报告到您的安全团队已在使用的控制台。这在 macOS 和 Windows 上处于测试版，不涵盖云中的会话。了解有关**[在 Claude Cowork 中运行您的 EDR 传感器](https://support.claude.com/en/articles/17828685-run-your-endpoint-detection-and-response-edr-sensor-in-claude-cowork)**的更多信息。

### 提示注入风险

由于 Cowork 的代理性质和互联网访问，它具有独特的风险。我们已实施安全措施，包括模型训练和内容分类器，但提示注入攻击的风险不为零。

用户应该：

- 避免授予对包含敏感信息的文件的访问权限

- 监控 Claude 的可疑操作

- 限制浏览器和网络访问到受信任的来源

- 立即报告可疑行为

### 网络访问

Cowork 遵守您的组织的当前网络出口权限。在启用 Cowork 之前，请在**[组织设置 > 功能](https://claude.ai/admin-settings/capabilities)**中的**代码执行**下查看您的网络访问设置。

网络设置在创建新的 Cowork 会话时应用。如果您在活跃对话期间更改网络访问模式或向允许列表添加域，这些更改不会在该会话中生效。启动新对话以应用更新的设置。

网络出口权限不适用于网络获取或**[网络搜索](https://support.claude.com/en/articles/10684626-enabling-and-using-web-search)**工具，或 MCP，包括 Claude in Chrome。网络获取在服务器端运行，仅限于搜索结果和您共享的 URL。Team 和 Enterprise 所有者可以在**[组织设置 > 功能](https://claude.ai/admin-settings/capabilities)**中关闭 Cowork 和 Chat 的网络搜索，在**[组织设置 > Claude in Chrome](https://claude.ai/admin-settings/browser-extension)** 中关闭 Claude in Chrome。