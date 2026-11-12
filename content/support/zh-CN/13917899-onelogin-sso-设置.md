# OneLogin SSO 设置

本指南介绍如何使用 OneLogin 作为身份提供商为 Claude 配置 SSO 和 SCIM。适用于 Team 计划、Enterprise 计划和 Console 组织。

## 前置条件

- Claude Team 计划、Enterprise 计划或具有父组织的 Console 组织（请参阅**[启用 SSO 前的重要注意事项](https://support.claude.com/en/articles/10276682)**了解 Console 父组织的要求）

- Owner 或 Primary Owner 角色（Team 和 Enterprise）或 Admin 角色（Console）

- OneLogin 超级用户访问权限

- 在 Claude 的身份和访问设置中验证的域名——请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的设置路径，包括域名验证

## 查找配置值的位置

下面引用的 Audience（Entity ID）、ACS URL 和 SCIM 凭证在身份和访问设置中的 WorkOS 设置流程中提供，不需要联系支持团队。

- **Team 和 Enterprise 计划：**转到**[claude.ai/admin-settings/identity](https://claude.ai/admin-settings/identity)**

- **Console 组织：**转到**[platform.claude.com/settings/identity](https://platform.claude.com/settings/identity)**

在那里启动 SSO 设置流程，并在完成以下步骤时将其与 OneLogin 管理门户并排打开。

---

## 步骤 1 — 在 OneLogin 中创建新应用程序

1. 在 OneLogin 管理门户中，转到**应用程序 → 应用程序 → 添加应用程序**。

2. 搜索"SAML 自定义连接器"并选择"SAML 自定义连接器（高级）"。

3. 将其命名为"Claude"并点击"保存"。

## 步骤 2 — 配置 SAML 设置

1. 在**配置**选项卡中，输入 WorkOS 设置流程中的值：**Audience（EntityID）**、**ACS（Consumer）URL** 和 **ACS URL 验证器**（ACS URL 的正则表达式或精确匹配）。

2. 将**SAML 发起者**设置为**服务提供商**。

3. 将**SAML nameID 格式**设置为**电子邮件**。

4. 在**SSO**选项卡中，下载**颁发者 URL**元数据，并在系统提示时在 WorkOS 设置流程中上传。

## 步骤 3 — 映射属性

1. 在**参数**选项卡中，添加一个字段：**字段名称：**email，**值：**Email（该人员的 OneLogin 电子邮件字段）。

2. 确保在下一步中为 SCIM 配置使用相同的电子邮件字段。

## 步骤 4 — 启用 SCIM 配置

**注意：**SCIM 配置仅在 Enterprise 计划和符合条件的 Console 组织上可用。如果您使用的是 Team 计划，请跳过此步骤——您可以改用 JIT 配置。请参阅**[设置 JIT 或 SCIM 配置](https://support.claude.com/en/articles/13133195)**。

1. 在**配置**选项卡中，启用**启用配置**。

2. 输入 WorkOS 设置流程中的 SCIM 基础 URL 和持有者令牌。

3. 启用**创建用户**、**删除用户**和**更新用户**。

4. 在**权利**中，将电子邮件属性映射到与 SAML 相同的 OneLogin 字段。

5. 点击"保存"。

**重要：**SAML 和 SCIM 必须发送相同的电子邮件值。如果人员在配置后遇到登录失败，请参阅**[OneLogin SSO/SCIM 电子邮件不匹配](https://support.claude.com/en/articles/13917861)**。

## 步骤 5 — 分配人员

在**用户**选项卡中，分配个人或使用**规则**根据角色或组自动分配。

## 步骤 6 — 验证

1. 如果启用了 SCIM，请检查 OneLogin 配置活动日志以确认人员已在 Claude 中创建。

2. 让测试用户完成 SSO 登录，并验证他们进入您的组织工作区。

---

## 需要帮助？

请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的端到端流程，包括域名验证和选择配置方法。如果遇到问题，请联系**[我们的支持团队](https://support.claude.com/en/articles/9015913)**，并提供您的 OneLogin 域名和 SAML 配置的屏幕截图。