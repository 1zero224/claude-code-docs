# 什么是 Claude Tag？

**[Slack 中的 Claude](https://claude.com/product/tag)** 已于 2026 年 8 月 3 日切换到新的 Claude Tag 体验。要集成 Claude 和 Slack，请改用 Claude Tag。了解如何**[设置 Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview)** 或**[从早期的 Slack 中的 Claude 迁移](https://claude.com/docs/claude-tag/admins/migrate-from-earlier)**。

Claude Tag 是与 Claude 合作的新方式：在对话中标记 @Claude，它会承担真实工作，使用您组织的工具和周围的共享上下文。Claude 以自己的身份工作，通过记住它所在频道的相关信息来建立上下文，并可以自行跟进。

Claude Tag 在 Team 和 Enterprise 计划中以测试版形式提供。Claude Tag 目前在 Slack 中可用。

这是我们将 Claude 的功能直接引入 Slack 的方式，为您的团队工作区带来 AI 协助。此集成允许您在不离开 Slack 的情况下与 Claude 合作，通过三个便捷的界面：

- **频道标记：**在任何频道中标记 @Claude 以分配任务，并在线程中跟进其工作。

- **与 Claude 的直接消息**：与 @Claude 开始私人对话。

- **AI 助手面板**：点击 Slack AI 助手标题中的 Claude 图标，在 Slack 窗口右侧打开一个面板，允许您从 Slack 应用中的任何位置访问 Claude。

当您在频道中标记 @Claude 时，Claude 会处理任务，整个交换过程对频道中的所有人都可见。频道中的每个人都与同一个 Claude 合作，因此任何人都可以引导它或从它停止的地方继续。Claude 也可以自行检查，例如在工作完成时发布或在线程停滞时标记您。要了解更多信息，请参阅**[Claude Tag 概述](https://claude.com/docs/claude-tag/overview)**。

在直接消息和助手面板中，Claude 具有您在自己的 Claude 账户中启用的功能，例如网络搜索和您连接的工具。频道标记的工作方式不同：Claude 以您组织的身份行动，使用管理员为该频道设置的工具和访问权限，工作费用由您的组织而非您个人承担。

---

## 设置 Claude Tag

安装 Claude 应用后，主所有者或所有者设置 Claude Tag：配置 Claude 的身份、连接您组织的工具和存储库，并选择 Claude Tag 可以在哪些频道中工作。一旦频道准备就绪，您团队中的人员无需单独设置任何内容。有关完整演练，请参阅**[Claude Tag 设置指南](https://claude.com/docs/claude-tag/admins/setup-overview)**。

**重要提示：**只有主所有者或所有者可以设置 Claude Tag 的访问权限和频道。管理员角色无法设置。

---

## 控制谁可以使用 Claude Tag

在**组织设置 > Claude Tag** 中，成员访问有三种模式：对 Slack 工作区中的任何人开放、对您的 Claude 组织的任何成员开放，或仅对其角色允许的成员开放。第三个选项是基于角色的访问，在 Claude Enterprise 计划中可用。要按角色限制，请将成员访问设置为

[https://www.youtube.com/embed/JhipXUs1Y98?rel=0](https://www.youtube.com/embed/JhipXUs1Y98?rel=0)

## Manage spend limits for Claude Tag

Claude Tag is consumption-based, so spend is based on usage rather than the number of people. As a Primary Owner or Owner, you control it from the usage settings in your admin console.

- **Organization-wide limit:** a hard cap on total Claude Tag spend across every channel. Spend can't exceed it.

- **Per-channel limits:** set a limit on any individual channel, on top of the organization-wide cap. New channels inherit a default limit.

- **Threshold alerts:** admins are notified at 75% and 95% of any limit.

- **Usage analytics:** a per-channel spend breakdown lives on the same page.

**Note:** Work that would go over a limit is declined, never silently cut short. A blocked user can request more from their admin without leaving Slack, and the alert says whether the limit or the available balance caused the block.

Tagging Claude in a channel is billed to your organization. Direct messages are billed to your own Claude account instead.

To set limits, see **[Set spend limits for Claude Tag](https://claude.com/docs/claude-tag/admins/restrict-access#set-spend-limits)**.

## Set access and permissions for Claude Tag

You decide what Claude Tag can reach by setting credentials and repository access at three levels. Each level inherits the permissions and memory of the one above it.

- **Organization-wide:** credentials and repositories that apply everywhere Claude Tag is installed.

- **Workspace:** access that applies to every public channel inside a Slack workspace. Inherits organization-wide permissions and memory.

- **Private channel:** extra credentials or repositories on top of what the workspace already grants. Use a private channel to keep sensitive connections to a smaller group. For example, a channel set up for legal work keeps its tools and memory separate from an engineering channel.

To configure access, see **[Claude Tag identity and access](https://claude.com/docs/claude-tag/concepts/agent-identity)**.

### Channels with guests

Channels that include Slack guests have a separate **Allow Claude to respond to guests** setting that controls whether Claude responds there, and what access Claude has when it does. Choose **Restrict** (the default, which blocks Claude from those channels entirely), **Channel only** (Claude replies, but while a guest is present it runs with channel-only access), or **Allow** (Claude replies with the full access configured for it, as in any other channel). For the full guide, see[**Restrict guest channels**](https://claude.com/docs/claude-tag/admins/restrict-access#restrict-guest-channels).

## Review memory and activity for Claude Tag

Claude Tag keeps context per channel and per workspace. Owners can view, edit, and delete that memory.

An Activity page in **Organization settings > Claude Tag > Activity** lists every scheduled and one-time task across your organization in addition to all network calls made using Agent Identity. Only Owners can open this page. Each action is also traceable in the tool where it happened: posts come from the Claude app in Slack, and commits and pull requests show the Claude GitHub App as the author with a link back to the Slack thread that started them. In any channel, you can ask "@Claude what triggers do you have set up here?" to see and turn off standing work.

---

## Privacy and data

### Data storage

Your Slack conversations with Claude remain separate from your Claude history, keeping work organized across platforms.

### Data visibility

- Conversations initiated in Slack are not visible in **[your Claude chat history](https://claude.ai/recents)**.

- Conversations initiated in the Claude web app are not accessible in Slack.

- Each platform maintains separate conversation histories.

### Data deletion

- Conversations are automatically deleted from Claude within 30 days if you disconnect the integration or uninstall the app.

- Your conversations in Slack follow your organization's Slack retention policies.

### Claude Tag memory

Claude Tag remembers context to do its work, so channel and workspace memory is retained rather than discarded after each task. Memory and activity respect channel boundaries, and admins can review or delete what Claude remembers. Channel work is attributed to your organization's Claude identity, while work done in a direct message runs on your own account.

---

## Frequently asked questions

### How is Claude Tag different from the Claude in Slack I already use?

Claude Tag is the next generation of that experience, in the same place. The familiar ways of working still apply, and Claude can now do more: it remembers context across days, schedules its own follow-ups, checks in proactively, and acts under its own identity. An organization’s Primary Owner or Owner opts your organization in to move over.

### Who can set up Claude Tag, and who pays for it?

Only an organization's Primary Owner or Owner can set up Claude Tag's access and channels. Tagging Claude in a channel is billed to your organization. Direct messages are billed to the person’s own Claude account instead.