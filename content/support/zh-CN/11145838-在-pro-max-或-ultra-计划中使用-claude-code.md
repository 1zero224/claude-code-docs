# 在 Pro、Max 或 Ultra 计划中使用 Claude Code

本文适用于使用 Pro、Max 或 Ultra 计划订阅来访问 Claude Code 的个人消费者。如果您是团队或企业计划组织的成员，请参阅 **[在团队或企业计划中使用 Claude Code](https://support.claude.com/en/articles/11845131)**。

## 什么是 Claude Code？

Claude Code 是一个编码工具，让您可以直接在终端或支持的 IDE 中访问 Claude 模型，允许您委托复杂的编码任务，同时保持透明度和控制。通过 Pro、Max 和 Ultra 计划，您现在可以通过一个统一的订阅访问网页、桌面和移动应用中的 Claude，以及终端中的 Claude Code。

### 为什么使用 Claude 和 Claude Code？

在一个简单的订阅中使用两个强大的 AI 产品。

- 使用 Claude 进行写作、研究、分析等工作——在工作和家中都可以。

- 使用 Claude Code 进行基于终端的编码工作流。

---

## 如何将 Claude Code 连接到您的 Pro、Max 或 Ultra 计划

1. **确保您有有效的 Pro、Max 或 Ultra 计划订阅**

  - 如果您还未订阅，请在 **[claude.ai/upgrade](https://claude.ai/upgrade)** 升级

2. **安装 Claude Code**

  - 访问 **[Claude Docs 中的 Claude Code 页面](https://code.claude.com/docs/en/quickstart#step-1-install-claude-code)**下载并安装 Claude Code。

  - 按照您的操作系统的安装说明进行操作。

3. **使用您的 Claude 凭证进行身份验证**

  - 在设置或首次使用时出现提示时，使用您用于 Claude 的相同凭证登录。

  - 这将把您的 Pro、Max 或 Ultra 计划订阅连接到 Claude Code。

  - 如果您已通过 Claude Console PAYG 登录到 Claude Code，请在 Claude Code 中运行 /login 以切换到您的订阅计划。

### 使用您的 Pro、Max 或 Ultra 计划访问 Claude Code 时遇到问题？

如果您没有看到使用首选账户进行身份验证的选项，请按照以下步骤更新 Claude Code：

1. 使用 /logout 命令完全退出您的活跃会话。

2. 运行 claude update。

3. 完全重启您的终端以使更改生效。

4. 运行 claude 并选择正确的账户以使用 Claude Code。

**重要：**如果您的系统上设置了 ANTHROPIC_API_KEY 环境变量，Claude Code 将使用此 API 密钥进行身份验证，而不是您的 Claude 订阅（Pro、Max、Ultra、Team 或 Enterprise 计划），这将导致 API 使用费用而不是使用您的订阅包含的使用量。有关更多信息，请参阅本文：**[在 Claude Code 中管理 API 密钥环境变量](https://support.claude.com/en/articles/12304248-managing-api-key-environment-variables-in-claude-code)。**

## 在您的 IDE 中使用 Claude Code

您的 Pro、Max 或 Ultra 计划也涵盖支持的 IDE 中的 Claude Code，包括 VS Code、Cursor 和其他 VS Code 分支，以及 JetBrains IDE（如 IntelliJ 和 PyCharm）。使用您在终端中使用的相同 Claude 凭证登录。IDE 使用量计入在 Claude 和 Claude Code 之间共享的相同使用限制。

要安装和设置 IDE 的扩展，请参阅我们 Claude Code Docs 中的 **[平台和集成](https://code.claude.com/docs/en/platforms)**。

---

## 达到使用限制时会发生什么

Pro 和 Max 计划有五小时的会话限制和每周限制。Ultra 计划有每周限制，已验证的 Ultra 账户没有五小时的会话限制。了解更多。

Max 和 Ultra 计划还对 Fable 有单独的每周限制。这些限制在 Claude 和 Claude Code 之间共享，这意味着两个工具中的所有活动都计入相同的限制。为了帮助您监控使用情况，当您接近任何限制时，您会看到警告消息。

如果在 Claude Code 正在进行响应时达到五小时会话限制，它可能会继续工作一段时间，以便完成正在进行的步骤，而不是中途停止。在这种情况下，您会在 Claude Code 中看到"已达到使用限制·正在完成"。这个完成宽限期很小且有上限，使用量计入您的每周限制。它仅适用于已在进行的响应，不适用于达到限制后发送的新消息。

在 Pro 计划上，每个每周使用期最多应用一次。在 Max 计划上，每次达到五小时会话限制时都可以应用，在您的每周限制内。已验证的 Ultra 账户没有五小时限制，因此完成宽限期不适用。如果您的 Ultra 账户有临时限制，完成宽限期的工作方式与 Max 计划相同，直到限制被移除。

它正在逐步推出，因此可能还不适用于您的账户。了解更多关于 **[Claude Code 完成宽限期](https://support.claude.com/en/articles/17040437)**。

当您达到使用限制时，您可以根据需要从几个选项中选择：

### Pro 计划用户

- 如果您经常达到限制并需要更多容量来处理更大的存储库，请考虑升级到 Max 计划。

- **[启用使用额度](https://support.claude.com/en/articles/12429409-)**以在达到五小时会话限制或每周限制后继续使用 Pro 计划的 Claude。

- 切换到 Claude Console 账户并 **[购买 API 使用额度](https://support.claude.com/en/articles/8977456-how-do-i-pay-for-my-claude-api-usage)**进行密集编码冲刺。

- 等待您的使用限制重置。

### Max 计划用户

- 如果您经常达到限制，请考虑升级到 Ultra 计划。

- **[启用使用额度](https://support.claude.com/en/articles/12429409-)**以在达到五小时会话限制、一般每周限制或 Fable 的单独每周限制后继续使用 Max 计划的 Claude。

- 切换到 Claude Console 账户并 **[购买 API 使用额度](https://support.claude.com/en/articles/8977456-how-do-i-pay-for-my-claude-api-usage)**进行密集编码冲刺。

- Max 计划附带的每月 API 额度不能在 Claude Code 或 Claude 应用中使用，包括用于使用额度。了解更多关于 **[Max、Ultra 和 Team 计划的每月 API 额度](https://support.claude.com/en/articles/17154008)**。

- 等待您的使用限制重置。

### Ultra 计划用户

- 启用使用额度以在达到一般每周限制或 Fable 的单独每周限制后继续使用 Ultra 计划的 Claude。

- 切换到 Claude Console 账户并 **[购买 API 使用额度](https://support.claude.com/en/articles/8977456-how-do-i-pay-for-my-claude-api-usage)**进行密集编码冲刺。

- Ultra 计划附带的每月 API 额度不能在 Claude Code 或 Claude 应用中使用，包括用于使用额度。了解更多关于 **[Max、Ultra 和 Team 计划的每月 API 额度](https://support.claude.com/en/articles/17154008)**。

- 等待您的使用限制重置。

有关高效使用的更多详情，请参阅我们的 **[使用限制最佳实践](https://support.claude.com/en/articles/9797557-usage-limit-best-practices)**。

---

## Claude Code 计费

### 了解两个不同的系统

重要的是要认识到这些是独立的系统：

- **Claude Code：**提供通过 API 额度继续使用的选项。

- **Claude Console / Claude API：**包含可选的自动充值设置，用于 API 额度管理，其中您的终端位于。

### 选择使用 API 额度

如果您想通过 Claude Code 使用 API 额度：

- 使用将按 **[标准 API 费率](https://claude.com/pricing#api)**计费（不同于 Pro/Max/Ultra 计划定价）。

- 如果在您的 Console 账户中启用了自动充值，当余额不足时将自动添加额外额度。

### 保持在您的计划范围内

要严格保持使用在您的 Pro、Max 或 Ultra 计划分配范围内：

- 在出现提示时拒绝 API 额度选项。

- 等待您的使用期重置后再继续使用 Claude Code。

- 使用 /status 命令监控您的剩余分配。

### 选择不使用 Claude Code 的 API 额度

如果您希望完全防止 API 额度选项出现：

- 在您的终端中运行 claude logout。

- 运行 claude login 并仅使用您的 Pro、Max 或 Ultra 计划凭证进行身份验证。

- 避免在登录过程中添加任何 Claude Console 凭证。

这确保 Claude Code 将仅使用您的计划分配，当您达到限制时不会提示您使用 API 额度。

### 管理自动充值设置

自动充值功能在您的 Claude Console 账户中管理，而不是通过 Claude Code：

- 查看您的 **[Console 计费设置](https://platform.claude.com/settings/billing)**以检查自动充值状态。

- 如果您希望避免自动购买额度，请在 Console 中调整这些设置。

- 请记住，自动充值仅在您选择使用 API 额度时适用。

## 总结

- Claude Code 对计费决策保持严格的用户控制。

- 自动充值是一个独立的 Claude Console 功能。

- 要保持您的 Pro、Max 或 Ultra 计划预算，只需在提供时拒绝 API 额度选项。