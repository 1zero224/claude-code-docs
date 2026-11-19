# 在网络、桌面和移动设备上使用 Claude Cowork

Claude Cowork 可在桌面、网络、移动设备和 Chrome 侧边栏中的 Claude 上使用。您的会话和文件与您的 Claude 账户关联，可在任何设备上随处使用。本文介绍如何启动 Cowork 会话以及各个平台上的可用功能。

## 可用性

Claude Cowork 可在付费计划上使用（Pro、Max、Ultra、Team 和 Enterprise）：

- **Claude Desktop for macOS、Windows 和 Linux（测试版）：**所有付费计划。在 **claude.com/download** 下载或更新。

- **网络和 Claude Mobile：**Pro、Max、Ultra 和 Team 计划（Team 上处于测试版），以及 Enterprise 计划（当所有者启用时）。

- **Chrome 侧边栏中的 Claude：**所有付费计划。在 Enterprise 计划上，所有者需要在云中启用 Claude in Chrome 和 Cowork。请参阅 **[Claude in Chrome 管理员控制](https://support.claude.com/en/articles/13065128-claude-in-chrome-admin-controls#h_bdb63199e1)**。

新的 Claude 体验将聊天和 Cowork 结合在一起，正在向 Pro、Max 和 Ultra 计划推出。在 Team 和 Enterprise 计划上，当启用**聊天和 Cowork 统一**时可用。如果您的消息框不再显示 “聊天”和 “Cowork” 选项，说明您已拥有新体验，本文中的某些步骤可能看起来不同。详细了解 **[Claude Cowork 和聊天合为一体](https://support.claude.com/en/articles/16761823)**。

---

## 启动 Cowork 会话

在桌面、网络和移动设备上，聊天和 Cowork 共享一个主页，因此您可以从同一位置启动两者。找到消息框，在左下角选择 “Cowork”，然后描述您的任务。要返回常规对话，选择 “聊天”。

如果您拥有新的 Claude 体验，则没有 “Cowork” 选项可选择。在任何对话中描述您的任务，Claude 会处理其余部分。

Chrome 侧边栏的工作方式不同。打开侧边栏会直接启动 Cowork 会话，无需在聊天和 Cowork 之间切换的选择器。

- **网络：**转到 **[claude.ai](https://claude.ai)** 并找到 “主页” 标签。

- **移动设备：**打开最新版本的 Claude for iOS 或 Claude for Android。如果您在消息框中看不到 Cowork 选项，请更新应用。

- **桌面：**打开最新版本的 Claude Desktop 应用。如果您在消息框中看不到 Cowork 选项，请更新应用。桌面是完整的 Cowork 体验，Claude 还可以使用您的本地文件和浏览器。

- **Chrome：**点击 Chrome 工具栏中的 Claude 图标打开侧边栏。侧边栏会启动 Cowork 会话，因此无需先选择 “Cowork”。

**注意：**要在不使用或添加到您的记忆的情况下运行任务，请在启动任务前在 “+” 菜单中关闭 “记忆”。

---

## 云中的 Cowork 如何工作

使用云中的 Cowork 时，Claude 的工作在 Anthropic 的服务器上运行，而不是在您的计算机上，您的会话和文件保存到您的 Claude 账户。这改变了 Cowork 的功能：

- 工作在后台继续。关闭您的笔记本电脑，Claude 继续运行。

- 计划任务在没有设备在线的情况下运行。

- 相同的会话和文件在桌面、网络和移动设备上可用。

- 在 Pro、Max 和 Ultra 计划上，新会话在每个平台的云中运行。您在 10 月 6 日之前在计算机上启动的任务保留在那里，以便您可以完成它们。

在 Pro、Max 和 Ultra 计划上，没有设置可选择任务运行的位置。

有关云和本地会话如何隔离以及每个会话可以访问的内容的详细信息，请参阅 **[Claude Cowork 架构概述](https://support.claude.com/en/articles/14479288)**。有关安全指导，请参阅 **[安全使用 Claude Cowork](https://support.claude.com/en/articles/13364135)**。

---

## Pro、Max 和 Ultra 计划在 10 月 6 日的变化

2026 年 10 月 6 日，Pro、Max 和 Ultra 计划上的新 Cowork 任务在云中运行。**[设置 > 常规](https://claude.ai/settings/general)**中的**仅在您的计算机上**选项已被移除。如果您仍设置为仅在计算机上运行 Cowork，下次打开桌面应用时您将被移至云，您会看到一条通知。无需设置任何内容，您无法切换回去。

您已在计算机上启动的任务保留在那里，您可以继续处理它们直到完成。每个任务都显示一个横幅，说明它仅在此计算机上运行。

您的文件夹保留在您的计算机上。Claude 仅通过桌面应用访问您已连接的文件夹，且仅在应用打开时访问。当任务需要云中的文件时，Claude 仅获取该文件的副本。删除会话时，Claude 获取的副本也会被删除，符合我们的**[数据保留实践](https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data)**。您的对话是否用于改进 Claude 遵循**[设置 > 隐私](https://claude.ai/settings/data-privacy-controls)**中的**帮助改进我们的 AI 模型**设置。

您在计算机上创建的计划任务会自动移至云。新的计划任务在云中运行。请参阅**[在 Claude Cowork 中安排定期任务](https://support.claude.com/en/articles/13854387)**。

### 如果您希望任务在计算机上运行

对于仅在您的计算机上运行的新任务，请在桌面应用中使用 Claude Code。它在您的计算机上运行，并将您的文件夹和历史记录保留在那里。**[获取 Claude Code 桌面](https://claude.com/download)**。要随身携带 Cowork 任务：

1. 打开任务并选择 “下载任务数据”。

2. 解压下载文件并在 Claude Code 中打开文件夹。

3. 运行 /port-task-data 以继续您的上下文。

要保留所有内容的副本，请在桌面主页的卡片上选择 “导出您的 Cowork 数据”。

如果您在工作和家中使用 Claude，并希望它们分开，请为每个使用单独的账户。不支持共享登录。

---

## 各个平台上的可用功能

以下是您可以在各个平台上使用的功能：

| **功能**                    | **桌面** | **网络** | **移动设备** |
| ------------------------- | ------ | ------ | -------- |
| 启动、指导和审查任务                | ✅      | ✅      | ✅        |
| 恢复在另一个平台上启动的会话            | ✅      | ✅      | ✅        |
| 连接器                       | ✅      | ✅      | ✅        |
| 技能和插件                     | ✅      | ✅      | ✅        |
| 预览 Claude 创建的文件           | ✅      | ✅      | ✅        |
| 定时任务                      | ✅      | ✅      | ✅        |
| 项目                        | ✅      | ✅      | ✅        |
| 实时工件（2026 年 8 月 19 日之前创建） | ✅      |        |          |
| 工件（2026 年 8 月 19 日或之后创建）  | ✅      | ✅      |          |
| 本地文件访问                    | ✅      | ✅\*    | ✅\*      |
| 浏览器使用                     | ✅      | ✅\*    | ✅\*      |
| 计算机使用                     | ✅      | ✅\*    | ✅\*      |

关于该表的几点说明：

- *本地文件访问、本地连接器、浏览器使用和计算机使用在网页和移动设备上通过 Claude Desktop 应用运行。云会话只能在桌面应用打开且会话在桌面上启动或为定时任务时，读取和写入您在计算机上连接的文件夹中的文件。如果应用关闭，会话会继续运行，但无法访问您的本地文件。

- 项目在所有平台上都可用。您可以从项目启动聊天或协作会话，Claude 会使用项目的知识作为上下文。与本地文件夹关联的项目仅在桌面上支持协作会话，协作不会更改项目内容，因此请自行将要保留的任何内容添加到项目中。有关更多信息，请参阅 **[使用 Claude Cowork 中的项目组织您的任务](https://support.claude.com/en/articles/14116274)**和 **[在 Claude Cowork 中使用实时工件](https://support.claude.com/en/articles/14729249)**。

- 2026 年 8 月 19 日或之后创建的工件在桌面应用和网页上可用。在该日期之前创建的实时工件仅在桌面应用上可用。有关更多信息，请参阅 **[在 Claude Cowork 中使用工件](https://support.claude.com/en/articles/14729249)**。

- 包含本地 MCP 服务器的本地连接器和插件仅通过桌面应用运行。

- 定时任务在云中运行，因此不再需要您的计算机保持唤醒状态。有关更多信息，请参阅 **[在 Claude Cowork 中安排定期任务](https://support.claude.com/en/articles/13854387)**。

- 计算机使用适用于 Pro、Max 和 Ultra 计划。有关更多信息，请参阅 **[让 Claude 在 Cowork 中使用您的计算机](https://support.claude.com/en/articles/14128542)**。

## 需要桌面应用的功能

某些功能需要访问您计算机上的内容，因此即使您的会话在云中运行，也需要在您的计算机上打开 Claude Desktop 应用：

- **本地文件访问。**Claude 读取和写入您在计算机上连接的文件夹中的文件。云中的会话仅在桌面应用打开、仅针对您连接的文件夹以及您已设置的权限下才能访问这些文件。

- **本地连接器。**这包括使用这些连接器的插件。

- **浏览器使用。**在桌面上，Claude 默认使用 Claude Desktop 应用中内置的浏览器，或通过 Claude in Chrome 使用您自己的 Chrome 浏览器（如果这是您首选的浏览器）。当桌面应用在线时，内置浏览器也可在网页或移动设备上的 Cowork 中使用。在桌面上启动的会话可以从网页或移动设备进行控制，同时桌面应用保持打开。在 Chrome 侧边栏中，Claude 可以读取您所在的标签页，无需桌面应用。Claude 作为任务的一部分驱动浏览器仍需要桌面应用打开。在 **[在 Claude Cowork 中使用内置浏览器](https://support.claude.com/en/articles/16607400)**中了解更多信息。

- **计算机使用。**Claude 可以直接点击、输入和导航您的屏幕。

## 在不同设备间切换

使用云端 Cowork 时，会话会跟随您的账户，因此您可以在任务中途切换设备：

1. 在任何设备上开始任务。

2. 从另一设备打开相同会话以查看进度、回答 Claude 的问题或重新指导工作。

3. 在任何地方获取完成的输出。

例如，在查看仪表板时在 Chrome 侧边栏中开始任务，然后在桌面上继续处理已下载的文件。

当 Claude 完成任务或需要您的输入时，您会在手机上收到通知。要开始使用，请参阅 **[Claude Cowork 入门指南](https://support.claude.com/en/articles/13345190)**。