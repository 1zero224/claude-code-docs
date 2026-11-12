# 在 Claude 中为您的组织设置 Salesforce

Claude 中的 Salesforce 是由 Salesforce 构建的插件，可将您组织的 Salesforce 数据和工作流程引入 Claude。本文将指导管理员启用该插件并为您的组织连接 Salesforce。

有关如何在 Claude 中使用 Salesforce 的最终用户说明，请参阅 **[在 Claude 中使用 Salesforce](https://support.claude.com/en/articles/16952186)**。

Claude 中的 Salesforce 在测试版中可用于 Salesforce 通过其测试版注册批准的所有付费计划组织。它目前在聊天和 Claude Cowork（网页和桌面）中运行。

## 包含内容

Claude 中的 Salesforce 包括：

- 由 Salesforce 构建的 37 项销售技能，涵盖续约准备、QBR 演示、管道覆盖和会议跟进等工作流程。

- Salesforce 和 Slack 连接器，因此该插件无需分别连接每项服务即可工作。

- 一项设置技能，在用户首次打开插件时运行。它了解用户的角色和业务范围，并为他们个性化技能。

## 开始前

您的组织需要访问最新的 Sales Cloud 企业版才能符合测试版资格。

## 步骤 1：请求访问 Claude 中的 Salesforce 插件（Salesforce 管理员）

要安装 Claude 中的 Salesforce 插件，您 Salesforce 账户上的管理员可以通过 AgentExchange 请求访问权限：

1. 转到 **[AgentExchange](https://agentexchange.salesforce.com/sales-cloud-in-claude-beta-access)**。

2. 填写表单以请求访问 Claude 中的 Salesforce 插件。管理员将收到一封接受电子邮件，其中包含指向 Salesforce 帮助文章的链接，该文章提供设置说明。

3. 完成您的接受电子邮件中的设置说明。

## 步骤 2：为您的组织启用 Claude 中的 Salesforce（Claude 管理员）

完成步骤 1 后，选择您的安装偏好并连接 MCP 连接器。您需要具有 Claude 组织的主要所有者或所有者访问权限才能完成这些步骤。

要选择您的安装偏好：

1. 转到 **[组织设置 > 插件](https://claude.ai/admin-settings/plugins)**。

2. 找到 **Salesforce Marketplace**。

3. 为您想要使用它的组选择安装偏好："默认安装"、"可安装"或

要激活和配置 Salesforce MCP 连接器：

1. 您的 Salesforce 管理员按照 Salesforce 的设置指南打开 Salesforce MCP 服务器。

2. 管理员向您发送外部客户端应用的消费者密钥和消费者密钥。

3. 在 Claude 中，转到 **组织设置 > 连接器**并选择"Salesforce（测试版）"。在 **OAuth 客户端 ID** 字段中输入消费者密钥，在 **OAuth 客户端密钥** 字段中输入消费者密钥，然后保存。

之后，每个成员在首次使用该插件时使用自己的账户登录 Salesforce。

## 步骤 3：激活 Claude 中的 Salesforce 插件（您组织的成员）

完成步骤 1 和 2 后，您组织的成员可以打开并使用该插件。

如果您是组织的成员，想了解有关使用 Claude 中的 Salesforce 插件的更多信息，请参阅 **[在 Claude 中使用 Salesforce](https://support.claude.com/en/articles/16952186)**。

## Claude 在 Salesforce 中可以看到和做什么

Claude 以每个用户自己的 Salesforce 账户身份登录，只能看到该用户现有 Salesforce 权限已允许的内容。默认情况下，Claude 会要求您在写入任何建议的更改之前批准它。