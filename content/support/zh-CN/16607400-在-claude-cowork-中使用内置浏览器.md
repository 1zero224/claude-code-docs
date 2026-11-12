# 在 Claude Cowork 中使用内置浏览器

Claude Cowork 在 Claude Desktop 应用中内置了浏览器。本文介绍了内置浏览器的工作原理、它与 Chrome 中的 Claude 的区别，以及如何选择 Claude 使用哪一个。

## 可用性

内置浏览器在 Claude Desktop 的 Cowork 中可用，支持 macOS、Windows 和 Linux（测试版），适用于 Pro、Max 和 Ultra 计划，以及 Team 和 Enterprise 计划（除非所有者已将其关闭）。当桌面应用在线时，它也可在网页或移动设备上的 Cowork 中使用。

新的 Claude 体验将聊天和 Cowork 结合在一起，正在向 Pro、Max 和 Ultra 计划推出。在 Team 计划（测试版）和 Enterprise 计划中，它在启用**聊天和 Cowork 统一**的地方可用。如果您的消息框不再显示"聊天"和

## 内置浏览器的工作原理

当 Cowork 任务涉及网站时，浏览器会在任务旁的侧面板中打开。Claude 会打开网站、阅读页面、点击、输入和填写表单，而您可以观看整个过程，无需切换窗口。任务记录中的链接会在同一面板中打开。

内置浏览器具有与 Chrome 中的 Claude 相同的浏览功能。它不依赖您自己的浏览器，无论您通常使用哪个浏览器都能正常工作。

内置浏览器位于桌面应用中，因此即使您的 Cowork 会话在云中运行，Claude Desktop 也需要打开并在线才能让 Claude 使用它。如果您在桌面上启动任务，只要桌面应用保持打开，您可以继续从网页上的 Claude 或 Claude Mobile 引导它。

## 登录网站

内置浏览器首次打开时，您会看到"通过从浏览器导入 Cookie 来保持登录状态"的选项。点击"导入 Cookie"按钮，一步即可从浏览器导入已保存的登录信息。导入是逐个网站进行的，因此您可以选择要导入哪些登录信息。银行、电子邮件和单点登录网站默认保持未选中状态。

在 macOS 上可从 Chrome、Edge 和 Firefox 导入已保存的登录信息，在 Windows 和 Linux（测试版）上可从 Firefox 导入。Safari 不支持导入。

您也可以在使用过程中登录网站，Claude 会在 Cowork 会话中记住您的登录信息，因此您无需再次登录。

## Claude 可以看到什么

内置浏览器与您自己的浏览器是分开的。除非您选择导入，否则 Claude 看不到您保存的登录信息。

**注意：**您在内置浏览器中登录的任何内容都可供 Claude 在该计算机上的未来 Cowork 会话中使用。请谨慎选择要登录的网站，特别是处理金钱或个人信息的网站。

## 内置浏览器或 Chrome 中的 Claude

Cowork 可以通过两种方式使用网络：

- **内置浏览器。**Claude 在侧面板中的自己的浏览器中工作。无需安装任何内容，也不会影响您的标签页或登录信息。当您想要交接任务的网络部分并继续工作时，请使用它。

- **Chrome 中的 Claude。**Claude 通过 Claude in Chrome 扩展程序在您自己的 Chrome 浏览器中工作，在您已经打开的页面上，使用您已经登录的账户。当工作在您面前的页面上时，请使用它。详细了解**[开始使用 Chrome 中的 Claude](https://support.claude.com/en/articles/12012173)**。

如果您已经使用 Chrome 中的 Claude，它将保持为 Cowork 中网络任务的默认选项。如果您没有该扩展程序，或者您是 Cowork 中浏览器使用的新手，Claude 将使用内置浏览器。

## 更改 Claude 使用的浏览器

您可以随时切换默认设置：

1. 打开 Claude Desktop 应用并转到**设置 > Cowork**。

2. 在**首选浏览器**下，选择"内置浏览器"或"Chrome（Chrome 中的 Claude）"。

Claude 将为网络任务使用您首选的浏览器，除非您要求它使用另一个。

如果您选择内置浏览器作为首选浏览器，只要应用打开并在线，在网页或移动设备上启动的任务将使用桌面应用中的浏览器。如果您首选的浏览器是 Chrome 中的 Claude，网页或移动设备上的任务将直接使用该扩展程序；您的会话需要连接到桌面，但应用不必打开。

如果您首选的浏览器不可用，Claude 会告诉您并继续使用另一个。如果您按名称要求使用特定浏览器而该浏览器不可用，Claude 会告诉您并在使用另一个之前询问。

---

## 安全性

内置浏览器运行与 Chrome 中的 Claude 相同的安全措施：

- Claude 在首次对网站采取行动前会请求您的许可。

- 高风险网站被阻止。

- 每项操作都会通过安全检查，将 Claude 正在执行的操作与您要求的操作进行比较。

任何在浏览器中执行操作的 AI 代理都可能成为提示注入的目标，其中网页中隐藏的指令试图重定向 Claude。这些安全措施可以降低该风险，但无法完全消除它。从您信任的网站开始，密切关注具有真实后果的任务，如果发现异常，请停止任务。

详细了解**[安全使用 Chrome 中的 Claude](https://support.claude.com/en/articles/12902428)**和**[安全使用 Claude Cowork](https://support.claude.com/en/articles/13364135)**。

**重要提示：**我们强烈建议不要使用内置浏览器或 Chrome 中的 Claude 来管理或执行涉及敏感信息的操作，例如财务账户、医疗信息或他人的个人数据。

---

## 在 Team 和 Enterprise 计划上

您组织的所有者控制内置浏览器和 Chrome 中的 Claude 是否可用。如果您看不到内置浏览器，或**首选浏览器**设置缺少某个选项，请联系您的管理员。有关管理员文档，请参阅**[为 Team 和 Enterprise 计划在 Claude Cowork 中设置浏览器使用](https://support.claude.com/en/articles/16635803)**。