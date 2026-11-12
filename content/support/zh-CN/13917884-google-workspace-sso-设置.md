# Google Workspace SSO 设置

本指南帮助您配置 Claude 以使用 Google Workspace 作为 SSO 和自动用户配置的身份提供商。适用于 Team 计划、Enterprise 计划和 Console 组织。

## 前置条件

- Claude Team 计划、Enterprise 计划或具有父组织的 Console 组织（请参阅**[启用 SSO 前的重要注意事项](https://support.claude.com/en/articles/10276682)**了解 Console 父组织要求）

- Owner 或 Primary Owner 角色（Team 和 Enterprise）或 Admin 角色（Console）

- Google Workspace 超级管理员访问权限

- 您的域名已在 Claude 的身份和访问设置中验证——请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的设置路径，包括域名验证

## 查找您的配置值

下面引用的 ACS URL、Entity ID 和 SCIM 凭证由您的身份和访问设置中的 WorkOS 设置流程提供，而不是通过联系支持部门获得。

- **Team 和 Enterprise 计划：**转到**[claude.ai/admin-settings/identity](https://claude.ai/admin-settings/identity)**

- **Console 组织：**转到**[platform.claude.com/settings/identity](https://platform.claude.com/settings/identity)**

在那里启动 SSO 设置流程，并在您完成以下步骤时将其与 Google Admin 控制台并排打开。

---

## 步骤 1 — 在 Google Admin 中添加自定义 SAML 应用

1. 在**Google Admin 控制台**中，转到**应用 → Web 和移动应用 → 添加应用 → 添加自定义 SAML 应用**。

2. 将其命名为"Claude"，然后点击"下一步"。

3. 下载**IdP 元数据** XML。当系统提示时，您将在 WorkOS 设置流程中上传此文件。

4. 点击"下一步"。

## 步骤 2 — 输入服务提供商详情

1. 输入 WorkOS 设置流程中的值：**ACS URL** 和**Entity ID**。

2. 将**Name ID 格式**设置为**EMAIL**，将**Name ID**设置为**基本信息 > 主要电子邮件**。

3. 点击"下一步"。

## 步骤 3 — 配置属性映射

1. 在**属性映射**步骤中，添加：Google Directory 属性：**主要电子邮件** → 应用属性：email。

2. 点击"完成"。

## 步骤 4 — 启用自动配置 (SCIM)

**注意：**SCIM 配置仅在 Enterprise 计划和符合条件的 Console 组织上可用。如果您使用的是 Team 计划，请跳过此步骤——您可以改用 JIT 配置。请参阅**[设置 JIT 或 SCIM 配置](https://support.claude.com/en/articles/13133195)**。

1. 在应用的设置中，转到**自动配置**部分，然后点击"配置自动配置"（可能需要在 Google Workspace 管理员设置中启用）。

2. 输入 WorkOS 设置流程中的**端点 URL** 和**访问令牌**。

3. 将 primaryEmail 属性映射到 Claude 电子邮件字段。

4. 启用配置并保存。

**关键：**SAML 和 SCIM 都必须使用 primaryEmail。如果某人有别名，请确保其主要 Google 电子邮件与 SCIM 发送的内容相匹配。如需故障排除，请参阅**[Google Workspace SSO/SCIM 电子邮件不匹配](https://support.claude.com/en/articles/13917817)**。

## 步骤 5 — 将应用分配给人员或组织单位

1. 在应用设置中，点击"用户访问"。

2. 为相关的组织单位或组启用该应用。

3. 点击"保存"。

## 步骤 6 — 验证

1. 如果您启用了 SCIM，请检查已配置的人员是否出现在您的组织的成员列表中。

2. 让测试用户通过 SSO 登录，并确认他们进入您的组织的工作区。

---

## 需要帮助？

请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的端到端流程，包括域名验证和选择配置方法。如果您遇到问题，请联系**[我们的支持团队](https://support.claude.com/en/articles/9015913)**，并提供您的组织域名和 SAML 配置的屏幕截图。