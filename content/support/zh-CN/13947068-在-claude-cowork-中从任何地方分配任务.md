# 在 Claude Cowork 中从任何地方分配任务

Claude Cowork 为您提供一个持续的 Claude 对话，您可以从手机或桌面访问。通过 Dispatch，您可以从手机向 Claude 发送消息，让它在桌面计算机上工作，使用您的本地文件、连接器、插件和应用程序，然后返回完成的工作。

Dispatch 在您的桌面上运行任务，因此当 Claude 工作时，您的计算机需要处于唤醒状态且 Claude Desktop 应用程序必须打开。这与云会话不同，云会话在 Anthropic 的服务器上运行，即使您的计算机关闭也会继续工作。有关 Cowork 在各个平台上的运行位置，请参阅 **[在网络、桌面和移动设备上使用 Claude Cowork](https://support.claude.com/en/articles/15520349)**。

## 可用性

Dispatch 在某些 Pro、Max 和 Ultra 计划中处于有限测试版，不适用于新用户。如果您已经使用 Dispatch，您现在可以继续使用它，本文仍然适用。它需要 Claude Desktop 应用程序和 Claude 移动应用程序。如果您在 Cowork 侧面板中看不到 Dispatch，请在云中使用 Cowork，而不是此处描述的配对流程。

新的 Claude 体验（结合了聊天和 Cowork）正在向 Pro、Max 和 Ultra 计划推出。在团队计划（测试版）和企业计划上，它在 **Chat and Cowork unified** 打开的地方可用。如果您的消息框不再显示"Chat"和

---

## 要求

要使用此功能，您需要：

- 在您的计算机（macOS、Windows x64 或 Linux）上安装并运行最新版本的 **Claude Desktop 应用程序**。当 Claude 处理任务时，您的计算机必须处于唤醒状态且应用程序必须打开。在 **[claude.com/download](https://claude.com/download)** 下载或更新。

- 在您的手机上安装最新版本的 **Claude 移动应用程序**。现有移动应用程序用户需要在使用此功能之前更新到最新版本。

- **Pro、Max 或 Ultra 计划**。

- **两个设备上的活跃互联网连接**。

---

## 工作原理

您不是为每个任务启动新会话，而是与 Claude 有一个单一的持久线程。此线程不会重置。Claude 保留来自以前任务的上下文，因此您可以从中断的地方继续。

在上班途中从手机向 Claude 发送消息，然后在坐下时从桌面跟进。这是相同的对话、相同的上下文，无论您从哪里访问它。

当您分配任务时，Claude 会确定需要什么样的工作并启动正确的会话。开发任务在 Claude Code 中运行；知识工作在 Cowork 中运行。这些会话出现在各自的侧栏中。您可以点击任何会话查看详细信息，或在线程中等待结果。

Claude 向您发送结果消息（电子表格、备忘录、比较表、拉取请求），而不是向您显示过程的每一步。当任务完成或 Claude 需要您的批准时，您会在手机上收到推送通知。

---

## 开始使用

按照以下步骤开始使用：

1. 下载或更新 Claude Desktop。

2. 下载或更新 Claude for iOS 或 Android。

3. 在您的手机或桌面上打开 Cowork。

4. 点击左侧面板上的"Dispatch"。

5. 您将进入描述功能的页面。点击"开始使用"：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2169954086/419674f781edb2977b93cce062b4/93b1893c-d79a-4eb6-b2f1-2fe3e043bd90?expires=1794465900&amp;signature=3755c997cf92b6d9985b0fa181d162d6eedf9f55c6d635164e7778e154bab8d5&amp;req=diEhH8B7mYFXX%2FMW1HO4zSZP05aPEwv2B32drIe5EDkAMRr1uh2MOfov7PiI%0AOxwR%0A)

6. 在下一个屏幕上，您可以通过切换来授予 Claude 访问您的文件的权限并保持计算机唤醒：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2169955082/de4053ee0eab8fcb9263584bb171/d39b77da-1a69-4682-9fdb-7ed488f236b0?expires=1794465900&amp;signature=b938da39f7f62368cf58f17852fbe1e3386b50ad574c7a66fa950541fd016e2f&amp;req=diEhH8B7mIFXW%2FMW1HO4zaZWst6cWwQXepuGRb1rD3K%2FrCYCZZFgIrNST2p%2F%0AxuEL%0A)

7. 点击"完成设置"。

8. 在"Dispatch"部分中开始与 Claude 发送消息。

完成这些步骤后，您与 Claude 的持续对话会自动在两个平台上同步。

---

## 您可以做什么

从您的手机，您可以将使用桌面上所有内容的任务交给 Claude，包括您无法在手机上打开的内容。例如：

- 要求 Claude 从本地电子表格中提取数据并编译摘要报告。

- 让 Claude 搜索您的 Slack 消息和电子邮件，然后起草简报文档。

- 请求从 Google Drive 中的文件构建的格式化演示文稿。

- 告诉 Claude 组织或处理计算机上特定文件夹中的文件。

Claude 使用您已在 Cowork 中配置的相同连接器、插件和文件访问。您不需要为移动设备单独设置任何内容。

### 检索文件和输出

当 Claude 完成生成文件的任务时，您可以直接从移动设备访问它，或在 Claude 指定的位置在桌面上找到它。

### 记忆

Claude 记住您所做的工作并学习您的工作方式。上下文在会话之间传递，因此您不必重新解释您的偏好、您的项目或您喜欢的做事方式。

您可以控制 Claude 记住的内容。您可以随时查看、编辑和删除您的记忆。

### 计划任务

您可以设置 Claude 按计划自动运行的任务。告诉 Claude 一次每天早上检查您的电子邮件、每周提取您的指标或编译周五报告，它会从那里处理，无需再次询问。

有关设置和管理计划任务的更多信息，请参阅 **[在 Cowork 中安排定期任务](https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-cowork)**。

### 计算机使用

Claude 可以使用您计算机上的应用程序来完成您通过 Dispatch 分配的任务。如果您要求 Claude 在 Excel 中更新电子表格、导航内部仪表板或运行您的开发工具，Claude 可以直接在您的桌面上使用这些应用程序。

有关计算机使用的工作原理、权限和安全指导的详细信息，请参阅 **[让 Claude 在 Cowork 中使用您的计算机](https://support.claude.com/en/articles/14128542-computer-use-safety)**。

**注意：**计算机使用在 Linux 测试版中不可用。在 Linux 桌面上，Dispatch 仍然可以使用您的文件、连接器和插件，但它无法通过计算机使用驱动桌面应用程序。

---

## 安全考虑

从您的手机，您现在可以通过 Claude 访问桌面上的所有内容——包括文件、连接器、您安装的任何插件以及通过计算机使用的应用程序。

给移动 AI 代理远程控制桌面 AI 代理会创建一个链，其中来自您手机的指令可以触发计算机上的真实操作：读取、移动或删除本地文件、与连接的服务交互、控制您的浏览器以及使用您的桌面应用程序。这很强大，但这也意味着错误（或模型在途中遇到的恶意内容）可能会产生真实的后果。被篡改的指令、意外命令或在浏览器中打开的网络钓鱼链接可能会级联为难以或不可能撤销的操作。

在启用此功能之前，请确保您：

- 信任链中的每个应用程序和服务

- 了解哪些文件和帐户可访问

- 知道如何快速断开连接或撤销访问权限

仅在您对他们*可能*做的事情感到满意时才连接这些代理，而不仅仅是您打算让他们做的事情。

有关其他安全指导，请参阅 **[安全使用 Cowork](https://support.claude.com/en/articles/13364135-use-cowork-safely)**。

---

## 当前限制

以下限制适用：

- **您的桌面必须处于活跃状态。**Dispatch 使用桌面计算机上的本地文件和应用程序，因此当 Claude 工作时，您的计算机需要处于唤醒状态且 Claude Desktop 应用程序必须打开。如果您希望 Claude 在计算机关闭时继续工作，请改为启动云会话。请参阅 **[在网络、桌面和移动设备上使用 Claude Cowork](https://support.claude.com/en/articles/15520349)**。

- **计算机使用具有与其他 Cowork 工具不同的安全属性。**Claude 直接点击、输入和导航您的屏幕，而不是通过连接器或权限门控文件访问。有关详细信息，请参阅 **[让 Claude 在 Cowork 中使用您的计算机](https://support.claude.com/en/articles/14128542-computer-use-safety)**。

- **一个持续的线程。**没有办法启动新线程或管理多个线程。所有消息都存在于单个对话中。

- **在 Linux 上，依赖计算机使用的任务不可用**，因为计算机使用不是 Linux 测试版的一部分。文件、连接器和插件任务正常工作。