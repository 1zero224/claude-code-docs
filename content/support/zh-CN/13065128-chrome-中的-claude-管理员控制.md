# Chrome 中的 Claude 管理员控制

本文介绍了团队和企业所有者如何为其组织管理 Chrome 中的 Claude。Chrome 中的 Claude 是一个浏览器扩展程序，允许 Claude 代表您的用户读取、点击和浏览网站。作为所有者，您可以控制用户是否可以安装该扩展程序以及他们可以访问哪些网站。

## 可用性

Chrome 中的 Claude 管理员控制适用于团队和企业计划。侧面板在 Pro、Max、Ultra 和团队计划上作为 Claude Cowork 会话运行。在企业计划上，启用后会作为 Cowork 会话为您的组织运行。请参阅下面的**[启用 Cowork 侧面板](#h_bdb63199e1)**。如果未启用 Cowork 侧面板，用户将使用经典侧面板，您现有的组织设置将继续适用于它。

## 访问 Chrome 中的 Claude 设置

要为您的组织管理 Chrome 中的 Claude 设置：

1. 使用您的所有者或主要所有者账户登录 Claude。

2. 导航到**[组织设置 > Chrome 中的 Claude](https://claude.ai/admin-settings/browser-extension)**。

## 启用或禁用扩展程序

**重要提示：**在为您的组织启用 Chrome 中的 Claude 之前，请查看**[安全使用 Chrome 中的 Claude](https://support.claude.com/en/articles/12902428)**以了解基于浏览器的 AI 的风险，包括提示注入分类器、现有的保护措施和剩余风险。

使用**为您的团队启用**切换开关为整个组织启用或禁用 Chrome 中的 Claude。

- **团队计划：**扩展程序默认启用。如果您希望用户无法访问，请禁用它。

- **企业计划：**扩展程序默认启用，除非您已禁用它。在已启用 HIPAA 的组织中，Chrome 中的 Claude 扩展程序保持关闭状态，直到所有者在**[组织设置 > Chrome 中的 Claude](https://claude.ai/admin-settings/browser-extension)**中将其打开。

Chrome 中的 Claude 和 Claude Cowork 是分开管理的。为您的组织启用 Chrome 中的 Claude 可让用户使用该扩展程序。Claude 是否可以在 Cowork 中使用它是一个单独的功能设置，用户的浏览器仍需要部署或安装该扩展程序。Cowork 还在 Claude Desktop 应用中内置了一个浏览器，不需要该扩展程序；在企业计划上默认启用，并从**[组织设置 > Cowork](https://claude.ai/admin-settings/cowork)**进行管理。您可以启用 Chrome 中的 Claude、内置浏览器、两者或都不启用。有关 Cowork 管理员设置，请参阅**[在团队和企业计划上使用 Claude Cowork](https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans)**。

**注意：**为企业组织启用扩展程序时，用户不会自动收到通知。您可能希望通过内部渠道传达可用性。

## 配置网站访问

使用允许列表和阻止列表来控制 Claude 在 Chrome 中的 Claude 或 Cowork 内置浏览器中工作时可以访问哪些网站。同一列表管理两者，因此无需为内置浏览器维护单独的列表。

**允许列表：**通过将网站添加到允许列表来指定 Claude 允许访问的网站。

**阻止列表：**通过将网站添加到阻止列表来指定 Claude 不应访问的网站，无论其他设置如何。这在**[Claude 的默认阻止类别](https://support.claude.com/en/articles/12902428-using-claude-for-chrome-safely#h_34f8d5ca87)**之外提供了额外的保护层。

**建议：**从更严格的允许列表开始以保护您组织的数据安全，然后随着您对扩展程序行为的熟悉程度增加而逐步扩展访问权限。

**注意：**允许列表和阻止列表管理 Claude 可以导航到和操作的 URL。如果用户手动在内置浏览器中输入被阻止的 URL，页面仍会加载，并显示"此网站被您组织的政策阻止"横幅，Claude 的工具在该页面上被禁用。

## 启用 Cowork 侧面板

在企业计划上，启用后，Chrome 中的 Claude 侧面板可以作为 Claude Cowork 会话运行。侧面板会话保存到用户的历史记录中，跨表面移动，并支持他们的技能、插件和连接器。在启用之前，用户会看到经典侧面板。

应用于 Claude Code（本地模式）和 Cowork（本地模式）的 HIPAA 配置中，云中的 Cowork 不可用，因此用户保留经典侧面板。

**要为您的组织打开 Cowork 侧面板：**

1. 在**[组织设置 > Cowork](https://claude.ai/admin-settings/cowork)**中为您的组织启用云中的 Cowork。请参阅**[在团队和企业计划上使用 Claude Cowork](https://support.claude.com/en/articles/13455879)**。

2. 导航到**[组织设置 > Chrome 中的 Claude](https://claude.ai/admin-settings/browser-extension)**并打开**为您的团队启用**。

3. 通过您的 Chrome 管理工具部署扩展程序，或让用户从**Chrome 网上应用店**安装它。

如果您的组织已在使用 Chrome 中的 Claude，一旦您启用云中的 Cowork，用户将自动看到 Cowork 侧面板。用户无需重新安装任何内容。

## 控制密码管理器访问

Claude 的 1Password 让 macOS 用户完成需要登录的任务，1Password 直接在页面上填充凭据，因此 Claude 永远看不到密码或一次性代码。该集成默认对您的组织关闭。

启用后，符合条件的用户将在 Claude Desktop 中看到该集成。用户还需要在 Mac 上安装 1Password 桌面应用、1Password 浏览器扩展、Claude Desktop 和 Chrome 中的 Claude。有关设置详情和要求，请参阅**[Claude 的 1Password 入门](https://support.claude.com/en/articles/15936181)**。

## 在 Claude Desktop 上管理用户访问

同时安装了 Chrome 中的 Claude 和 Claude Desktop 的用户现在可以选择在桌面应用上启动任务，让它处理浏览器中的工作，而无需切换窗口。

如果您想为组织成员禁用此功能，可以完全关闭扩展程序，或编辑您的企业配置。

**在组织设置中禁用 Chrome 扩展程序：**

1. 使用您的所有者账户登录 Claude。

2. 导航到**[组织设置 > Chrome 中的 Claude](https://claude.ai/admin-settings/browser-extension)**。

3. 关闭扩展程序。

或者，在**[您的企业配置](https://support.claude.com/en/articles/12622667-enterprise-configuration)**中禁用 `isLocalDevMcpEnabled`。

## 部署选项

启用后，用户可以通过两种方式访问 Chrome 中的 Claude：

- **自助服务：**用户从**[Chrome 网上应用店](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn)**自行安装扩展程序。

- **托管部署：**使用您现有的 Chrome 管理工具（Google Workspace 管理员控制台或 MDM）将扩展程序部署到特定用户或组。

大多数企业组织已经有 Chrome 扩展程序管理。您可以使用这些现有控制在试点阶段限制哪些员工可以安装扩展程序。

## 运行试点

要在更广泛推出之前使用用户子集测试 Chrome 中的 Claude：

1. 在组织级别启用扩展程序。

2. 配置限制性允许列表，将 Claude 限制为特定的受信任网站。

3. 使用您的 IT 控制限制哪些员工可以安装扩展程序。

4. 与试点用户分享**[安全使用 Chrome 中的 Claude](https://support.claude.com/en/articles/12902428)**。

5. 收集反馈并随时间推移扩展访问权限。

## Chrome 中的 Claude 如何适应您现有的控制

- **基于角色的权限：**Chrome 中的 Claude 有自己的权限，与 Claude Cowork 分开。两个设置控制 Chrome 中的 Claude：组织级别切换和管理员可以授予或拒绝的每个角色功能。该每个角色功能适用于使用自定义角色的企业组织。Chrome 中的 Claude 不继承用户的 Cowork 访问权限。

- **网络控制：**Chrome 中的 Claude 通过您现有的 Claude 端点（`claude.ai`、`api.anthropic.com`、`platform.claude.com`）发送其聊天流量，因此您在这些端点上设置的任何控制也适用于此处。它还连接到 Claude Desktop 使用的相同桥接端点（`wss://bridge.claudeusercontent.com`）和标准遥测服务。在限制性网络环境中，允许这些连接。要限制扩展程序可以使用的组织，请部署 `forceLoginOrgUUID` Chrome 企业政策。

- **零数据保留 (ZDR)：**Chrome 中的 Claude 不支持，与 Cowork 相同。

## 教育您的用户

我们建议在用户开始使用 Chrome 中的 Claude 之前与他们分享这些资源：

- **[Chrome 中的 Claude 入门](https://support.claude.com/en/articles/12012173-getting-started-with-claude-for-chrome)**：安装和核心功能

- **[安全使用 Chrome 中的 Claude](https://support.claude.com/en/articles/12902428)**：风险和最佳实践

- **[Chrome 中的 Claude 权限指南](https://support.claude.com/en/articles/12902446-claude-for-chrome-permissions-guide)**：用户如何控制 Claude 可以访问的内容