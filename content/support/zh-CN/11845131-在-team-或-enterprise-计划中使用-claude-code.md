# 在 Team 或 Enterprise 计划中使用 Claude Code

本文适用于使用 Team 或 Enterprise 计划订阅来访问 Claude Code 的组织成员。如果您是使用 Pro、Max 或 Ultra 计划订阅的个人消费者，请参阅 **[在 Pro、Max 或 Ultra 计划中使用 Claude Code](https://support.claude.com/en/articles/11145838)**。

## 什么是 Claude Code？

Claude Code 是一个编码工具，让您可以将复杂的编码任务委托给 Claude，同时保持透明度和控制权。它包含在 Claude Desktop 中，您也可以在终端或支持的 IDE 中运行它。

Claude Code 包含在每个 Team 计划席位中。高级席位为工作量较大的团队成员提供更多使用量。对于 Enterprise 计划，Claude Code 包含在新的和自助 Enterprise 席位中。在较旧的 Enterprise 计划中，Claude Code 可用于 Chat + Claude Code 席位（基于使用量的计费）和高级席位（基于席位的计费）。

使用 Team 或 Enterprise 计划，您可以通过一个订阅访问网页、桌面和移动应用上的 Claude，以及 Claude Code。

### 为什么使用 Claude 和 Claude Code？

在一个统一的订阅中结合两个强大的 AI 产品：

- 使用 Claude 进行跨团队的写作、研究、分析和协作。

- 在 Claude Desktop、终端或 IDE 中使用 Claude Code 进行编码和开发任务。

---

## 将 Claude Code 连接到您的 Team 或 Enterprise 计划

### 步骤 1：确认 Claude Code 访问权限（仅限 Enterprise 计划）

如果您的组织使用新的或自助 Enterprise 计划，Claude Code 已包含在每个 Enterprise 席位中——无需额外购买。请进行步骤 2。

**注意：**在 HIPAA 就绪的 Enterprise 计划中，在主所有者将 HIPAA 配置应用于 Claude Code（本地模式）和 Cowork（本地模式）后，您的业务伙伴协议 (BAA) 涵盖终端中的 Claude Code 和 Claude Desktop 的代码选项卡。如果没有该配置或 Claude Code 的零数据保留 (ZDR)，Claude Code 包含在您的席位中，可用但不在您的 BAA 保护范围内。请勿将其用于受保护的健康信息 (PHI)。请参阅 **[在 HIPAA 就绪的 Enterprise 计划中使用 Claude Code（本地模式）和 Cowork（本地模式）](https://support.claude.com/en/articles/17318731)**。

如果您的组织使用较旧的 Enterprise 计划，具有 Chat 和 Chat + Claude Code 席位，或标准和高级席位，您需要确保拥有包含 Claude Code 的席位类型。所有者可以在 **[组织设置 > 组织](https://claude.ai/admin-settings/organization)** 中购买或重新分配 **Chat + Claude Code / 高级席位**。有关详细信息，请参阅 **[在 Enterprise 计划中购买和管理席位](https://support.claude.com/en/articles/13393991-purchase-and-manage-seats-on-enterprise-plans)**。

### 步骤 2：打开 Claude Code

**推荐：**下载 **Claude Desktop**，使用您的 Team 或 Enterprise 账户登录，然后打开"代码"选项卡。Claude Desktop 包含 Claude Code，因此无需安装其他内容，它使用您登录的账户。在 Claude Code 文档中按照 **桌面快速入门** 进行首次会话。您可以跳过步骤 3。

要改用终端，请按照我们 **[Claude Code 文档](https://code.claude.com/docs/en/quickstart#step-1-install-claude-code)** 中针对您所使用环境的安装说明进行操作。

**注意：**如果您已在计算机上安装了 Claude Code，请进行步骤 3。

### 步骤 3：在终端中进行身份验证

1. 在终端中键入 `claude` 以启动 Claude Code 会话。

2. 在设置或首次使用时出现提示时，选择登录方法。

  1. 如果您已通过其他账户登录到 Claude Code，请运行 /login 以选择不同的登录方法。

3. 选择"带订阅的 Claude 账户"以被路由到 OAuth 提示。

4. 选择您的 Team 或 Enterprise 计划，然后单击"授权"。

5. 您的高级席位订阅将链接到 Claude Code。

登录后，来自您的 Claude 账户的技能和插件（包括您的组织提供的技能和插件）会自动加载到 Claude Code 中。这需要 Claude Code v2.1.273 或更高版本。在 Claude Code 文档中了解更多关于 **[同步技能的行为](https://code.claude.com/docs/en/skills#how-synced-skills-behave)**。

### 在使用 Team 或 Enterprise 账户访问 Claude Code 时遇到问题？

如果您在终端中看不到使用首选账户进行身份验证的选项，请按照以下步骤更新 Claude Code：

1. 使用 `/logout` 命令完全注销您的活跃会话。

2. 运行 `claude update`。

3. 完全重启您的终端以使更改生效。

4. 运行 `claude` 并选择正确的账户以使用 Claude Code。

## 在 IDE 中使用 Claude Code

您的席位也涵盖支持的 IDE 中的 Claude Code，包括 VS Code、Cursor 和其他 VS Code 分支，以及 JetBrains IDE（如 IntelliJ 和 PyCharm）。使用与在 Claude Desktop 或终端中相同的 Team 或 Enterprise 账户登录。IDE 使用受限，计费方式与您计划中的终端使用相同。

要为您的 IDE 安装和设置扩展，请参阅我们 Claude Code 文档中的 **[平台和集成](https://code.claude.com/docs/en/platforms)**。

**注意：**在 HIPAA 就绪的 Enterprise 计划中，VS Code 和 JetBrains 的 Claude Code 扩展可用但不在您的 BAA 保护范围内，包括将 HIPAA 配置应用于 Claude Code（本地模式）和 Cowork（本地模式）的情况。请勿使用它们处理受保护的健康信息 (PHI)。

---

## 达到使用限制时会发生什么

如果您的组织使用 **基于使用量的 Enterprise 计划**（包括自助 Enterprise），则没有每席位的使用限制——使用量基于消费并按 API 费率计费。有关使用量计费如何工作的详细信息，请参阅 **[我的 Enterprise 计划如何计费？](https://support.claude.com/en/articles/11526368-how-am-i-billed-for-my-enterprise-plan)**。

如果您的组织使用 Team 计划或基于席位的 Enterprise 计划，您可以启用使用额度，以允许团队成员在达到其包含的使用限制后继续使用 Claude、Cowork 和 Claude Code。有关更多信息，请参阅 **[为 Team 和基于席位的 Enterprise 计划管理使用额度](https://support.claude.com/en/articles/12005970-)**。

Team 计划附带的每月 API 额度不能在 Claude Code 或 Claude 应用中使用，包括额外使用。了解更多关于 **[Max、Ultra 和 Team 计划的每月 API 额度](https://support.claude.com/en/articles/17154008)**。