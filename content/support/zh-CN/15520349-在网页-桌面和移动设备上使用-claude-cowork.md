# 在网页、桌面和移动设备上使用 Claude Cowork

Claude Cowork 可在桌面、网页、移动设备和 Chrome 侧边栏中的 Claude 上使用。您的会话和文件与您的 Claude 账户相关联，可在任何设备上随处使用。本文介绍如何启动 Cowork 会话以及各个平台上的可用功能。

## 可用性

Claude Cowork 可在付费计划上使用（Pro、Max、Ultra、Team 和 Enterprise）：

- **Claude Desktop（macOS、Windows 和 Linux 测试版）：**所有付费计划。在 **claude.com/download** 下载或更新。

- **网页和 Claude Mobile：**Pro、Max、Ultra 和 Team 计划（Team 上处于测试版），以及 Enterprise 计划（当所有者启用时）。

- **Chrome 侧边栏中的 Claude：**所有付费计划。在 Enterprise 计划上，所有者需要启用 Chrome 中的 Claude 和云中的 Cowork。请参阅 **[Chrome 中的 Claude 管理员控制](https://support.claude.com/en/articles/13065128-claude-in-chrome-admin-controls#h_bdb63199e1)**。

新的 Claude 体验将聊天和 Cowork 结合在一起，正在向 Pro、Max 和 Ultra 计划推出。在 Team 计划（测试版）和 Enterprise 计划上，当 **聊天和 Cowork 统一**打开时可用。如果您的消息框不再显示"聊天"和

---

## 启动 Cowork 会话

在桌面、网页和移动设备上，聊天和 Cowork 共享一个主页，因此您可以从同一位置启动两者。找到消息框，在左下角选择"Cowork"，然后描述您的任务。要返回常规对话，选择"聊天"。

如果您拥有新的 Claude 体验，则没有"Cowork"选项可选择。在任何对话中描述您的任务，Claude 会处理其余部分。

Chrome 侧边栏的工作方式不同。打开侧边栏会直接启动 Cowork 会话，无需选择器在聊天和 Cowork 之间切换。

- **网页：**转到 **[claude.ai](https://claude.ai)** 并找到"主页"标签。

- **移动设备：**打开最新版本的 Claude for iOS 或 Claude for Android。如果您在消息框中看不到 Cowork 选项，请更新应用。

- **桌面：**打开最新版本的 Claude Desktop 应用。如果您在消息框中看不到 Cowork 选项，请更新应用。桌面是完整的 Cowork 体验，Claude 还可以使用您的本地文件和浏览器。

- **Chrome：**点击 Chrome 工具栏中的 Claude 图标以打开侧边栏。侧边栏会启动 Cowork 会话，因此无需先选择"Cowork"。

**注意：**要在不使用或添加到您的记忆的情况下运行任务，请在启动任务前在"+"菜单中关闭"记忆"。

---

## 云中的 Cowork 如何工作

使用云中的 Cowork 时，Claude 的工作在 Anthropic 的服务器上运行，而不是在您的计算机上，您的会话和文件保存到您的 Claude 账户。这改变了 Cowork 的功能：

- 工作在后台继续。关闭您的笔记本电脑，Claude 继续运行。

- 计划任务无需设备在线即可运行。

- 相同的会话和文件可在桌面、网页和移动设备上使用。

- 在 Pro、Max 和 Ultra 计划上，新会话在每个平台上都在云中运行。您在 10 月 6 日之前在计算机上启动的任务保留在那里，以便您可以完成它们。

在 Pro、Max 和 Ultra 计划上，没有设置可选择任务运行的位置。

有关云和本地会话如何隔离以及每个会话可以访问的内容的详细信息，请参阅 **[Claude Cowork 架构概览](https://support.claude.com/en/articles/14479288)**。有关安全指导，请参阅 **[安全使用 Claude Cowork](https://support.claude.com/en/articles/13364135)**。

---

## Pro、Max 和 Ultra 计划在 10 月 6 日的变化

2026 年 10 月 6 日，Pro、Max 和 Ultra 计划上的新 Cowork 任务在云中运行。**[设置 > 常规](https://claude.ai/settings/general)**中的**仅在您的计算机上**选项已被删除。如果您仍设置为仅在计算机上运行 Cowork，下次打开桌面应用时您将被移至云，您会看到一条通知。无需设置任何内容，您无法切换回去。

您已在计算机上启动的任务保留在那里，您可以继续处理它们直到完成。每个任务都显示一个横幅，说明它仅在此计算机上运行。

您的文件夹保留在您的计算机上。Claude 仅访问您已连接的文件夹（通过桌面应用），且仅在其打开时访问。当任务需要云中的文件时，Claude 会获取该文件的副本。删除会话时，Claude 获取的副本也会被删除，符合我们的**[数据保留政策](https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data)**。您的对话是否用于改进 Claude 遵循**[设置 > 隐私](https://claude.ai/settings/data-privacy-controls)**中的**帮助改进我们的 AI 模型**设置。

已在您的计算机上运行的计划任务继续在那里运行。新的计划任务在云中运行。请参阅**[在 Claude Cowork 中安排定期任务](https://support.claude.com/en/articles/13854387)**。

### 如果您希望任务在您的计算机上运行

对于仅在您的计算机上运行的新任务，请在桌面应用中使用 Claude Code。它在您的计算机上运行，并将您的文件夹和历史记录保留在那里。**[获取 Claude Code 桌面](https://claude.com/download)**。要随身携带 Cowork 任务：

1. 打开任务并选择"下载任务数据"。

2. 解压下载文件并在 Claude Code 中打开文件夹。

3. 运行 /port-task-data 以继续您的上下文。

要保留所有内容的副本，请在桌面主页的卡片上选择"导出您的 Cowork 数据"。

如果您在工作和家中使用 Claude，并希望它们分开，请为每个使用单独的账户。不支持共享登录。

---

## 各个平台上的可用功能

以下是您可以在各个平台上使用的功能：

| **功能**                    | **桌面** | **网页** | **移动设备** |
| ------------------------- | ------ | ------ | -------- |
| 启动、指导和审查任务                | ✅      | ✅      | ✅        |
| 恢复在另一个平台上启动的会话            | ✅      | ✅      | ✅        |
| 连接器                       | ✅      | ✅      | ✅        |
| 技能和插件                     | ✅      | ✅      | ✅        |
| 预览 Claude 创建的文件           | ✅      | ✅      | ✅        |
| 计划任务                      | ✅      | ✅      | ✅        |
| 项目                        | ✅      | ✅      | ✅        |
| 实时工件（2026 年 8 月 19 日之前创建） | ✅      |        |          |
| 工件（2026 年 8 月 19 日或之后创建）  | ✅      | ✅      |          |
| 本地文件访问                    | ✅      | ✅\*    | ✅\*      |
| 浏览器使用                     | ✅      | ✅\*    | ✅\*      |
| 计算机使用                     | ✅      | ✅\*    | ✅\*      |

关于该表的几点说明：

- *本地文件访问、本地连接器、浏览器使用和计算机使用可通过 Claude Desktop 应用在网页和移动设备上运行。云会话只有在桌面应用在该计算机上打开且会话在桌面上启动或是计划任务时，才能读取和写入您在计算机上连接的文件夹中的文件。如果应用关闭，会话会继续运行，但无法访问您的本地文件。

- 项目在所有平台上都可用。您可以从项目启动聊天或协作会话，Claude 会使用项目的知识作为上下文。与本地文件夹关联的项目仅在桌面上支持协作会话，协作不会更改项目的内容，因此请自行将要保留的任何内容添加到项目中。有关更多信息，请参阅**[在 Claude Cowork 中使用项目组织您的任务](https://support.claude.com/en/articles/14116274)**和**[在 Claude Cowork 中使用实时工件](https://support.claude.com/en/articles/14729249)**。

- 2026 年 8 月 19 日或之后创建的工件可在桌面应用和网页上使用。在该日期之前创建的实时工件仅在桌面应用上可用。有关更多信息，请参阅**[在 Claude Cowork 中使用工件](https://support.claude.com/en/articles/14729249)**。

- 本地连接器和包含本地 MCP 服务器的插件仅通过桌面应用运行。

- 计划任务在云中运行，因此不再需要您的计算机保持唤醒状态。有关更多信息，请参阅**[在 Claude Cowork 中安排定期任务](https://support.claude.com/en/articles/13854387)**。

- 计算机使用对 Pro、Max 和 Ultra 计划处于测试阶段。有关更多信息，请参阅**[让 Claude 在 Cowork 中使用您的计算机](https://support.claude.com/en/articles/14128542)**。

## 需要桌面应用的功能

某些功能需要访问您计算机上的内容，因此即使您的会话在云中运行，也需要在您的计算机上打开 Claude Desktop 应用：

- **本地文件访问。**Claude 读取和写入您在计算机上连接的文件夹中的文件。云中的会话仅在桌面应用打开时、仅对您连接的文件夹以及使用您已设置的权限访问这些文件。

- **本地连接器。**这包括使用这些连接器的插件。

- **浏览器使用。**在桌面上，Claude 默认使用 Claude Desktop 应用中内置的浏览器，或通过 Claude in Chrome 使用您自己的 Chrome 浏览器（如果这是您首选的浏览器）。当桌面应用在线时，内置浏览器也可在网页或移动设备上的 Cowork 中使用。在桌面上启动的会话可以从网页或移动设备进行控制，同时桌面应用保持打开。在 Chrome 侧边栏中，Claude 可以读取您所在的标签页，无需桌面应用。Claude 作为任务的一部分驱动浏览器仍需要桌面应用打开。在**[在 Claude Cowork 中使用内置浏览器](https://support.claude.com/en/articles/16607400)**中了解更多信息。

- **计算机使用。**Claude 直接点击、输入和导航您的屏幕。

## 在表面之间移动

在云中使用 Cowork 时，会话会跟随您的账户，因此您可以在任务中途切换表面：

1. 在任何表面上开始任务。

2. 从另一个表面打开相同的会话以检查进度、回答 Claude 的问题或重定向工作。

3. 无论您在哪里，都可以获取完成的输出。

例如，在查看仪表板时在 Chrome 侧面板中开始任务，然后在桌面上继续处理已下载的文件。

当 Claude 完成任务或需要您的输入时，您会在手机上收到通知。要开始使用，请参阅 **[Claude Cowork 入门](https://support.claude.com/en/articles/13345190)**。