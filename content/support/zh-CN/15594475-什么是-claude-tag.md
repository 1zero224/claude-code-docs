# 什么是 Claude Tag？

**[Slack 中的 Claude](https://claude.com/product/tag)** 已于 2026 年 8 月 3 日切换到新的 Claude Tag 体验。要集成 Claude 和 Slack，请改用 Claude Tag。了解如何**[设置 Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview)** 或**[从早期的 Slack 中的 Claude 迁移](https://claude.com/docs/claude-tag/admins/migrate-from-earlier)**。

Claude Tag 是与 Claude 协作的新方式：在对话中标记 @Claude，它会承担真实工作，使用您组织的工具和周围的共享上下文。Claude 以自己的身份工作，通过记住所在频道的相关信息来建立上下文，并可以自主跟进。

Claude Tag 在 Team 和 Enterprise 计划中以测试版形式提供。Claude Tag 目前在 Slack 中可用。

这是我们将 Claude 的功能直接引入 Slack 的方式，将 AI 助手引入您团队的工作区。此集成允许您在不离开 Slack 的情况下与 Claude 协作，提供三个便捷的界面：

- **频道标记：**在任何频道中标记 @Claude 以分配任务，并在线程中跟进其工作进度。

- **与 Claude 的直接消息**：与 @Claude 开始私密对话。

- **AI 助手面板**：点击 Slack AI 助手标题中的 Claude 图标，在 Slack 窗口右侧打开一个面板，让您可以从 Slack 应用中的任何位置访问 Claude。

在频道中标记 @Claude 时，Claude 会处理任务，整个交互过程对频道中的所有人都可见。频道中的每个人都与同一个 Claude 协作，因此任何人都可以引导它或从它停止的地方继续。Claude 也可以自主检查，例如在工作完成时发布或在线程停滞时标记您。要了解更多信息，请参阅 **[Claude Tag 概览](https://claude.com/docs/claude-tag/overview)**。

在直接消息和助手面板中，Claude 具有您在自己的 Claude 账户中启用的功能，例如网络搜索和您连接的工具。频道标记的工作方式不同：Claude 以您组织的身份行动，使用管理员为该频道设置的工具和访问权限，工作费用由您的组织承担，而不是由您承担。

---

## 设置 Claude Tag

安装 Claude 应用后，主所有者或所有者设置 Claude Tag：配置 Claude 的身份、连接您组织的工具和存储库，并选择 Claude Tag 可以在哪些频道中工作。团队成员在频道准备就绪后无需单独设置任何内容。有关完整演练，请参阅 **[Claude Tag 设置指南](https://claude.com/docs/claude-tag/admins/setup-overview)**。

**重要提示：**只有主所有者或所有者可以设置 Claude Tag 的访问权限和频道。管理员角色无法设置。

---

## 控制谁可以使用 Claude Tag

在**组织设置 > Claude Tag** 中，成员访问有三种模式：对 Slack 工作区中的任何人开放、对您的 Claude 组织的任何成员开放，或仅对其角色允许的成员开放。第三个选项是基于角色的访问权限，在 Claude Enterprise 计划中可用。要按角色限制，请将成员访问设置为 “仅限其角色允许的成员”，并向自定义角色授予 “Slack 中的 Claude Tag” 功能。此设置适用于频道提及和直接消息。要设置成员访问权限，请参阅 **[限制 Claude Tag 的运行位置](https://claude.com/docs/claude-tag/admins/restrict-access)**。

[https://www.youtube.com/embed/JhipXUs1Y98?rel=0](https://www.youtube.com/embed/JhipXUs1Y98?rel=0)

## 管理 Claude Tag 的支出限额

Claude Tag 是基于消费的，因此支出基于使用情况而不是人数。作为主所有者或所有者，您可以从管理控制台的使用设置中控制它。

- **组织范围的限额：**对所有频道的 Claude Tag 总支出的硬性上限。支出不能超过此限额。

- **按频道限额：**在组织范围的上限之上，为任何单个频道设置限额。新频道继承默认限额。

- **阈值警报：**在任何限额的 75% 和 95% 处通知管理员。

- **使用分析：**按频道支出明细显示在同一页面上。

**注意：**超过限额的工作会被拒绝，永远不会被无声地截断。被阻止的用户可以在不离开 Slack 的情况下向其管理员请求更多配额，警报会说明是限额还是可用余额导致了阻止。

在频道中标记 Claude 的费用由您的组织承担。直接消息则由您自己的 Claude 账户承担。

要设置限额，请参阅 **[为 Claude Tag 设置支出限额](https://claude.com/docs/claude-tag/admins/restrict-access#set-spend-limits)**。

## 为 Claude Tag 设置访问权限和权限

您可以通过在三个级别设置凭证和存储库访问权限来决定 Claude Tag 可以访问的内容。每个级别都继承上一级别的权限和内存。

- **组织范围：**适用于安装 Claude Tag 的任何地方的凭证和存储库。

- **工作区：**适用于 Slack 工作区内每个公共频道的访问权限。继承组织范围的权限和内存。

- **私密频道：**在工作区已授予的基础上额外的凭证或存储库。使用私密频道将敏感连接限制在较小的群体。例如，为法律工作设置的频道将其工具和内存与工程频道分开。

要配置访问权限，请参阅 **[Claude Tag 身份和访问权限](https://claude.com/docs/claude-tag/concepts/agent-identity)**。

### 包含来宾的频道

包含 Slack 来宾的频道有一个单独的**允许 Claude 回复来宾**设置，用于控制 Claude 是否在那里回复，以及 Claude 回复时拥有的访问权限。选择**限制**（默认值，完全阻止 Claude 访问这些频道）、**仅频道**（Claude 回复，但当来宾在场时，它以仅频道访问权限运行）或**允许**（Claude 以为其配置的完整访问权限回复，如同任何其他频道）。有关完整指南，请参阅[**限制来宾频道**](https://claude.com/docs/claude-tag/admins/restrict-access#restrict-guest-channels)。

## 查看 Claude Tag 的内存和活动

Claude Tag 按频道和工作区保持上下文。所有者可以查看、编辑和删除该内存。

在**组织设置 > Claude Tag > 活动**中的活动页面列出了您组织中所有计划任务和一次性任务，以及使用代理身份进行的所有网络调用。只有所有者可以打开此页面。每个操作也可以在发生的工具中追踪：帖子来自 Slack 中的 Claude 应用，提交和拉取请求显示 Claude GitHub 应用作为作者，并链接回启动它们的 Slack 线程。在任何频道中，您可以询问 “@Claude 你在这里设置了哪些触发器？”来查看和关闭常规工作。

---

## 隐私和数据

### 数据存储

您与 Claude 的 Slack 对话与您的 Claude 历史记录保持分离，使工作在各平台间保持有序。

### 数据可见性

- 在 Slack 中启动的对话在**[您的 Claude 聊天历史记录](https://claude.ai/recents)**中不可见。

- 在 Claude 网络应用中启动的对话在 Slack 中无法访问。

- 每个平台维护单独的对话历史记录。

### 数据删除

- 如果您断开集成或卸载应用，对话会在 30 天内自动从 Claude 中删除。

- 您在 Slack 中的对话遵循您组织的 Slack 保留政策。

### Claude Tag 内存

Claude Tag 记住上下文以完成其工作，因此频道和工作区内存被保留而不是在每个任务后丢弃。内存和活动尊重频道边界，管理员可以查看或删除 Claude 记住的内容。频道工作归属于您组织的 Claude 身份，而在直接消息中完成的工作在您自己的账户上运行。

---

## 常见问题

### Claude Tag 与我已在使用的 Slack 中的 Claude 有何不同？

Claude Tag 是该体验的下一代，在同一位置。熟悉的工作方式仍然适用，Claude 现在可以做更多事情：它记住跨越数天的上下文、安排自己的跟进、主动检查并以自己的身份行动。组织的主所有者或所有者选择让您的组织迁移。

### 谁可以设置 Claude Tag，谁为其付费？

只有组织的主所有者或所有者可以设置 Claude Tag 的访问权限和频道。在频道中标记 Claude 的费用由您的组织承担。直接消息则由该人自己的 Claude 账户承担。