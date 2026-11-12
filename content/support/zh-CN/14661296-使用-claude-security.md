# 使用 Claude Security

## 概述

**[Claude Security](https://claude.com/product/claude-security)** 是内置于 Claude 中的一项功能，可持续监控您的代码库中的安全漏洞，跟踪每个发现直到修复，并建议针对性的补丁供人工审查。它帮助团队发现和修复传统方法经常遗漏的问题。

Claude Security 目前以测试版形式提供给企业计划的用户。

了解如何开始使用以及领先企业如何使用该工具，请访问：**[Claude Security 入门](https://academy.claude.com/tutorials/getting-started-with-claude-security)**。

Claude Security 允许您：

1. **全面扫描您的代码库** — Claude Security 理解您的代码如何组织在一起，跟踪跨文件和项目中存储库的数据流，并识别传统基于规则的扫描器无法检测的复杂多组件漏洞和错误。

2. **深度扫描** — 创建项目并连接来自 GitHub 或其他 Git 提供商的最多八个存储库。Claude 读取代码、编写威胁模型并运行深度基线扫描。

3. **持续扫描** — Claude Security 在新提交到达时运行高效扫描，并立即显示任何新发现以便快速修复。

4. **添加代码无法显示的内容** — 告诉 Claude 您的代码如何部署、谁可以访问它以及您已接受的风险。每个项目在威胁模型中保存这些信息，Claude 使用它来决定标记什么以及如何评级。

5. **验证发现** — 每个发现都经过多阶段验证，Claude 在显示发现之前会严格审查自己的发现。结果是：误报更少，噪音更少。

6. **审查和修补** — 无缝地从发现进入 Claude Code 会话以生成针对性的修复。快速解决漏洞，而不是积累待办事项。

7. **跟踪修复** — Claude 检查开放发现与已更改的代码，并在修复到达时将其标记为已修复。

8. **集成到您的工作流程中** — 通过 Claude Security API 或 Claude 和 Claude Code 中的 Claude Security 连接器 (MCP) 读取和解决发现，并在添加新发现时获得 webhook。

9. **控制访问** — 与具有完全、分类或查看角色的用户共享每个项目。管理员可以决定谁可以创建项目以及谁可以通过 RBAC 查看、分类或管理所有项目。

## 严重程度

严重程度是根据发现在您的代码库中的可利用程度分配的，而不是类别本身，因此相同的类别在不同的存储库中可能会处于不同的严重程度。Claude 使用项目的威胁模型来评估严重程度。

| **严重程度** | **标准**                                               | **典型示例**                  |
| -------- | ---------------------------------------------------- | ------------------------- |
| 严重       | 任何人都可以通过网络利用，无需登录、无需受害者操作且无特殊条件。它允许攻击者读取和更改数据。       | 面向互联网的服务中的未经身份验证的远程代码执行。  |
| 高        | 影响严重，但不属于严重级别：攻击者需要登录或受害者操作，或损害是部分的（例如，数据可以读取但不能更改）。 | 登录后的 SQL 注入，允许攻击者读取和更改数据。 |
| 中        | 影响有限（例如，少量非敏感数据），具有一个现实的前置条件，例如登录。                   | 登录后的 SQL 注入，仅暴露少数非敏感字段。   |
| 低        | 影响有限，需要登录或网络邻近性加上攻击者控制之外的特殊条件。                       | 需要网络邻近性和数千个请求的时序侧信道。      |

---

## 发现结构

每个发现包含以下字段：

- 标题 — 发现的简短描述性名称

- 严重程度 — 严重/高/中/低

- 类别 — 发现类型，适用时包含其 CWE ID

- 位置 — 文件路径和行范围，链接到源代码。一个发现可以引用多个文件和存储库。

- 状态 — 开放或已解决，包含发现日期。

- 受理人 — 分配给该发现的人员（如有）

- Claude 发现的内容 — 发现的描述及其重要性。它可以包括攻击路径图。

- 如何修复 — 关于如何解决它的指导

- 证据 — Claude 引用的文件和行范围，按存储库分组，包含扫描的分支和提交

- 攻击如何工作 — 攻击者如何到达和利用该问题

- Claude 为什么相信它 — Claude 的评估，涵盖利用场景、其前置条件和影响

- 历史 — 对发现的每次更改，最新的在前

- 仅在发现已解决时显示：

  - 解决方案 — 标记为已修复、通过合并的拉取请求修复或作为误报、已接受的风险、重复项或其他原因而被驳回

  - 分类说明 — 可选

---

## 常见问题

### 快速入门

**使用 Claude Security 之前我需要什么？**

需要有效的 Claude Enterprise 计划、为每个用户配置高级席位，以及为消费计费启用使用额度。所有者可在 **[组织设置 > Claude Security](https://claude.ai/admin-settings/claude-security)** 中启用 Claude Security。请参阅 **[快速入门指南](https://academy.claude.com/tutorials/getting-started-with-claude-security)** 了解设置步骤。

**如何授予 Claude Security 访问我的代码库的权限？**

对于 GitHub.com 和 GitHub Enterprise Server，所有者需安装 **[Claude GitHub App](https://github.com/apps/claude)** 并授予其访问您要扫描的代码库的权限。对于其他 Git 提供商（如 GitLab、Azure DevOps 和 Bitbucket），请输入代码库的 https 地址和可读取该代码库的身份验证令牌。地址应使用小写字母，不包含用户名、密码、查询参数或端口，例如：`https://gitlab.com/group/project`。

**我可以扫描不在 GitHub 上的代码库吗？**

可以。项目可以使用托管在 GitHub.com 和 GitHub Enterprise Server 上的代码库，或使用身份验证令牌托管在其他 Git 服务器上的代码库（包括可公开访问的自托管 Git 服务器）。

**我需要允许哪些 IP 地址？**

使用 **[出站 IP 地址](https://platform.claude.com/docs/en/api/ip-addresses#outbound-ip-addresses)** 列表进行 IP 地址白名单设置。如果您使用 GitHub Enterprise Server 或其他 Git 提供商，您的实例必须允许来自这些 IP 地址的入站流量，以便 Claude Security 可以访问您的代码库。

**Claude Security 是否支持零数据保留 (ZDR)？**

不支持。Anthropic 可能会在法律要求或处理使用政策违规时保留数据。

**最新更新后我现有的项目会发生什么？**

现有项目将继续按照现在的方式运行，仍可从 Claude Security 主页访问。新项目将使用新体验。

**我可以迁移现有项目吗？**

可以，如果您是项目所有者。打开项目并点击

### 持续监控

**首次扫描需要多长时间？**

根据代码库的大小，基线扫描可能需要几分钟到几小时。您可以在扫描运行时离开该页面。如果您为 Claude Security 启用了电子邮件通知，当项目扫描完成并发现新问题时，您将收到一封电子邮件。

**首次扫描后 Claude 多久扫描一次？**

Claude 根据风险和复杂性定期检查新提交并扫描更改。没有固定的时间表。

**后续扫描需要多长时间？**

扫描时间因代码库和代理的操作而异。

**后续扫描读取什么？**

Claude 检查更改、审查更改，并根据风险、复杂性和您的扫描工作量智能地确定扫描范围。根据您的设置、范围和时间间隔，Claude 可能会选择完整扫描或增量扫描。您可以在

**为什么同一代码的两次扫描会返回不同的结果？**

扫描在设计上是随机的。与传统的静态分析器不同，Claude Security 使用一个代理，该代理会根据每次运行进行调整，对代码上下文进行推理，而不是应用固定的模式匹配。这使得能够进行深度分析，以捕获逻辑级别的漏洞。

**我应该如何设置大型代码库或单一代码库？**

在代码库上设置文件夹范围，而不是扫描整个代码库，并排除您不需要覆盖的文件夹。Claude 仍然可以根据需要从代码库的其余部分提取上下文。

**一个项目可以有多少个代码库？**

最多八个。代码库可以在范围内（Claude 扫描它并报告发现），或仅用于上下文（Claude 在扫描期间读取它但不报告其中的发现）。两者都计入限制。在项目的

**我如何告诉 Claude 我的环境？**

打开项目的

**Claude Security 使用哪些模型？**

Claude Security 扫描由 Claude Mythos 模型提供支持。用户接收发现结果，无需直接访问模型。每个项目与 Claude 的聊天使用您的团队从组织允许的模型中选择的模型。Claude Security 的某些子组件在不需要 Mythos 级别推理时可能使用其他模型，例如枚举项目目录。

### 发现和修复

**严重性如何分配，我可以更改它吗？**

Claude 根据项目的威胁模型对每个发现进行评级，您的团队可以编辑和更正该模型。

**如果发现描述的攻击在我的环境中不可能发生怎么办？**

Claude 从您的代码编写威胁模型，因此可能会遗漏部署上下文和您公司已接受的模式。在项目的聊天中，告诉 Claude 为什么发现是错误的，并要求它更新威胁模型。Claude 使用威胁模型来决定要标记什么以及如何评级。

**我如何修复发现？**

点击发现上的

**修复跟踪如何工作？**

当代码库中的代码更改时，Claude 会重新检查发现。无论发现是手动还是自动标记为已修复，如果 Claude 后来发现问题未解决，它都会自动重新打开该发现。

**关闭发现时会发生什么？**

您可以将发现关闭为误报、已接受的风险、重复项或其他原因，并添加分类说明。已关闭的发现保持关闭状态。当后续扫描报告相同问题时，Claude 会将其与已关闭的发现匹配并保持关闭。您可以随时自己重新打开已关闭的发现。

### 集成和通知

**我如何将发现导出到我自己的工具？**

要将发现导出到您自己的跟踪和通知系统，请使用 Claude Security API、Claude Security 连接器或 webhook。

**我如何使用 Claude Security API？**

在 Claude Security 设置下创建个人访问令牌。令牌代表您：它只能查看和更改您可以查看和更改的内容。使用 API，您可以列出和读取项目和发现、解决和重新打开发现，以及创建和管理项目及其代码库。

**Claude Security 连接器 (MCP) 可以做什么?**

使用 Claude Security 连接器，Claude 和 Claude Code 可以列出并读取您的项目和发现结果，并在您要求时解决或重新打开发现结果。

**当有新发现时，我如何获得通知?**

在 Claude Security 设置中创建 webhook，将新发现事件推送到您自己的系统，例如 Slack 或 Jira。

### 访问和共享

**管理员如何控制谁可以使用 Claude Security?**

管理员可以通过在**[自定义角色](https://support.claude.com/en/articles/13930452-manage-custom-roles-on-enterprise-plans)**中配置功能，决定哪些 RBAC 角色可以创建项目，以及哪些可以查看、分类或管理所有项目。

**我如何共享项目，每个角色可以做什么?**

在项目中，点击

### Pricing

**How much does Claude Security cost?**

Claude Security is charged at direct token cost only. There is no additional platform fee. Usage covers scans and other work Claude Security does in a project: writing a threat model, the project chat, and checking fixes.

**How do I control and track what Claude Security costs?**

You can set a separate spend limit for Claude Security. Open Usage in a project’s settings to see its cost for the month to date and per day.

**Is there a launch offer for the updated Claude Security?**

Yes, eligible Claude Enterprise organizations receive free usage credits up to $25,000 a month, for two months, through December 31, 2026. This appears as a credit in **Organization settings > Usage**. Full **[launch offer terms](#h_07eed9f41c)** below.

**Does the launch offer also apply to the old experience?**

No, the launch offer only applies to projects created with the new experience.

### Claude Security plugin for Claude Code

**What is the Claude Security plugin for Claude Code?**

The Claude Security plugin for Claude Code is a separate feature, based on the same technology as Claude Security. It is available for users who wish to directly integrate the scanner into their workflows without relying on Anthropic’s hosted agent infrastructure, who have Zero Data Retention (ZDR) requirements, or who need model access through a third-party provider. Learn more here: **[Scan your codebase for vulnerabilities](https://code.claude.com/docs/en/claude-security)**.

**Does the Claude Security plugin for Claude Code use the same model?**

Not necessarily. The plugin uses whichever models are available in your Claude Code account.

**Scope of use:** You will only use Claude Security to scan code that you or your company owns and to which you or your company holds all necessary rights to scan. You will not use Claude Security to scan code owned by or licensed from third parties, including but not limited to open source projects or repositories other than those included in your company's codebase(s).

## Claude Security launch offer terms

**Who is eligible:** Claude Enterprise organizations, including Claude Enterprise purchased through AWS Marketplace.

**What you get:** Up to $25,000 in promotional credits for Claude Security usage in each of two consecutive months. Each month runs from your organization's start date to the same date the following month. Unused credits from a month don't carry over.

**How to get it:** No sign-up is needed. Credits apply automatically to usage in the new Claude Security. Usage in projects from the earlier version isn't covered until they're migrated.

**When it starts:** Organizations that already have Claude Enterprise start on October 21, 2026. Organizations that sign up for Claude Enterprise after that date start on the day they sign up. The two months run whether or not your organization uses Claude Security.

**When it ends:** The offer ends at the end of the day on December 31, 2026 (UTC), even if your two months haven't finished. Organizations that sign up after November 1, 2026 won't get two full months.

**How credits apply:** Credits are used before your organization's committed spend or usage credits, and are separate from any prepaid usage credits. After the credits are used or the offer ends, Claude Security usage is charged at direct token cost.

**Where to see it:** Organization admins can see the promotional balance in **[Organization Settings > Usage](https://claude.ai/admin-settings/usage)**, under **Services**, or in **Organization Settings > Claude Security**, under **Spend limit**.

The **[Claude Promotional Credit Offer Terms](https://www.anthropic.com/legal/promotion-credit-terms)** also apply, including when credits can be forfeited and when the offer can change. If they differ from this article on this offer, this article controls.