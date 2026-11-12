# 在 Claude Cowork 中为团队和企业计划设置浏览器使用

Claude 可以通过两种方式在 Claude Cowork 中使用网络：Claude Desktop 应用中内置的浏览器，或通过 Claude in Chrome 扩展程序使用用户自己的 Chrome 浏览器。本文介绍了两者的区别、如何为您的组织启用每一种方式，以及当两者都启用时用户会看到什么。

## 可用性

Cowork 的浏览器控制在团队和企业计划中可用。内置浏览器在 macOS、Windows 和 Linux（测试版）上的 Claude Desktop 应用中运行。当桌面应用在线时，它也可在 Cowork 网页版或移动版中使用。

新的 Claude 体验结合了聊天和 Cowork，正在向 Pro、Max 和 Ultra 计划推出。在团队计划（测试版）和企业计划中，当启用**聊天和 Cowork 统一**时可用。尚未启用此功能的组织将聊天和 Cowork 保持为单独的选项，因此本文仍然适用。企业所有者可以在**[为您的企业组织设置和管理新的统一 Claude 体验](https://support.claude.com/en/articles/17444462)**中了解更多信息。

## Claude 使用网络的两种方式

- **内置浏览器。**Claude 在自己的浏览器中运行，该浏览器在 Cowork 桌面应用内的 Claude 侧面板中打开。无需安装任何内容。它与用户自己的浏览器分离，因此 Claude 看不到他们的登录信息，除非他们选择导入。它需要桌面应用处于打开和在线状态。

- **Claude in Chrome。**Claude 通过 Claude in Chrome 扩展程序在用户自己的 Chrome 浏览器中运行，在他们已经访问的页面上，使用他们已经登录的账户。您的用户浏览器需要部署或安装该扩展程序。网页版和移动版上的 Cowork 会话在 Claude in Chrome 是用户首选浏览器时也可以使用它。

**两者都运行相同的安全层：**在 Claude 访问新网站前的每个网站权限提示、高风险网站的阻止列表，以及对每个操作的安全检查。在**[在 Claude Cowork 中使用内置浏览器](https://support.claude.com/en/articles/16607400)**和**[安全使用 Claude in Chrome](https://support.claude.com/en/articles/12902428)**中了解更多信息。

您可以启用一个、两个或都不启用。

## 启用或禁用内置浏览器

- **团队计划：**默认启用。

- **企业计划：**默认启用，除非您已将其关闭。

  - 在已启用 HIPAA 的组织中，内置浏览器保持关闭状态，直到所有者在**组织设置 > Cowork** 中将其打开。使用**[应用于 Claude Code（本地模式）和 Cowork（本地模式）的 HIPAA 配置](https://support.claude.com/en/articles/17318731)**，内置浏览器不可用，没有任何设置可以将其打开。

要为您的组织打开或关闭内置浏览器：

1. 以所有者或主要所有者身份登录 Claude。

2. 导航到**[组织设置 > Cowork](https://claude.ai/admin-settings/cowork)**。

3. 找到**内置浏览器**并将其打开或关闭。

当内置浏览器关闭时，用户无法打开它，Claude 也无法使用它。此设置不影响 Claude in Chrome 或 Claude Code 中的浏览器。

**注意：**在企业计划中，当您打开内置浏览器时，用户不会自动收到通知。您可能需要通过内部渠道传达可用性。

## 启用或禁用 Claude in Chrome

Claude in Chrome 在**[组织设置 > Claude in Chrome](https://claude.ai/admin-settings/browser-extension)** 中单独管理。在团队计划中默认启用。在企业计划中，除非您已禁用，否则默认启用。在已启用 HIPAA 的组织中，Claude in Chrome 扩展程序保持关闭状态，直到所有者在**组织设置 > Claude in Chrome** 中将其打开。您在那里配置的网站允许列表和阻止列表适用于扩展程序和内置浏览器。同一列表管理两者，因此无需维护单独的列表。有关设置、部署和试点指导，请参阅**[Claude in Chrome 管理员控制](https://support.claude.com/en/articles/13065128)**。

## 当两者都启用时

如果您的组织同时启用了内置浏览器和 Claude in Chrome，每个用户可以通过**[设置 > Cowork](https://claude.ai/settings/cowork)** 中的**首选浏览器**切换来选择 Claude 使用哪一个。已经使用 Claude in Chrome 的用户会将其保留为首选浏览器。没有该扩展程序的用户会获得内置浏览器。

Claude 使用首选浏览器执行网络任务。如果任务需要浏览器时首选浏览器不可用：

- 如果用户要求 Claude 通常使用浏览器，Claude 会告诉他们首选浏览器离线，并继续使用另一个。

- 如果用户按名称要求特定浏览器，Claude 会告诉他们该浏览器不可用，并在使用另一个前询问。

首选浏览器设置也适用于网页版和移动版上的 Cowork 会话。在网页版或移动版上启动的会话在内置浏览器是用户首选且桌面应用处于打开和在线状态时使用它。如果 Claude in Chrome 是首选，会话使用该扩展程序。对于网页版和移动版会话使用 Claude in Chrome，它们必须连接到桌面，但应用无需打开。