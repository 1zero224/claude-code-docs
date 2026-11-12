# Microsoft Entra ID SSO 设置

本指南将引导您使用 Microsoft Entra ID（原 Azure Active Directory）作为身份提供商为 Claude 配置单点登录 (SSO)。适用于 Team 计划、Enterprise 计划和 Console 组织。

## 前置条件

- Claude Team 计划、Enterprise 计划或具有父组织的 Console 组织（请参阅**[启用 SSO 前的重要注意事项](https://support.claude.com/en/articles/10276682)**了解 Console 父组织要求）

- Owner 或 Primary Owner 角色（Team 和 Enterprise）或 Admin 角色（Console）

- Microsoft Entra ID P1 或 P2 许可证（SCIM 预配所需）

- Entra 中的全局管理员或应用程序管理员角色

- 您的域已在 Claude 的身份和访问设置中验证 — 请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的设置路径，包括域验证

## 查找配置值的位置

下面引用的实体 ID、回复 URL (ACS URL) 和 SCIM 凭证在您的身份和访问设置中的 WorkOS 设置流程中提供 — 不是通过联系支持获得。

- **Team 和 Enterprise 计划：**转到**[claude.ai/admin-settings/identity](https://claude.ai/admin-settings/identity)**

- **Console 组织：**转到**[platform.claude.com/settings/identity](https://platform.claude.com/settings/identity)**

在那里启动 SSO 设置流程，并在您完成以下步骤时将其与 Entra 管理中心并排打开。

---

## 步骤 1 — 在 Entra 中添加 Claude 作为企业应用程序

**注意：**下面的"企业应用程序"是指 Microsoft Entra 管理中心中的部分 — 这是 Microsoft 对您集成的任何应用程序的术语，与您的 Claude 计划类型无关。

1. 在**Entra 管理中心**中，转到**企业应用程序 → 新应用程序**。

2. 在库中搜索"Claude"。如果可用，选择它；否则选择**创建您自己的应用程序**并将其命名为"Claude"。

3. 选择**集成您在库中找不到的任何其他应用程序**并单击"创建"。

## 步骤 2 — 配置 SAML SSO

1. 在您的新应用程序中，单击**单点登录 → SAML**。

2. 在**基本 SAML 配置**中，输入：来自 WorkOS 设置流程的**标识符（实体 ID）**、来自 WorkOS 设置流程的**回复 URL (ACS URL)** 和**登录 URL：****<https://claude.ai/login>**。

3. 在**属性和声明**中，确保电子邮件声明设置为发送 user.mail（或您将用于 SCIM 的相同属性）。

4. 下载**联合元数据 XML** 并在提示时在 WorkOS 设置流程中上传它。

## 步骤 3 — 配置 SCIM 预配

**注意：**SCIM 预配仅在 Enterprise 计划和符合条件的 Console 组织上可用。如果您使用的是 Team 计划，请跳过此步骤 — 您可以改用 JIT 预配。请参阅**[设置 JIT 或 SCIM 预配](https://support.claude.com/en/articles/13133195)**。

1. 在应用程序中，转到**预配 → 开始**。

2. 将**预配模式**设置为**自动**。

3. 输入来自 WorkOS 设置流程的**租户 URL** 和**密钥令牌**。

4. 单击**测试连接**进行验证。

5. 在**映射**下，确保电子邮件属性映射指向与您的 SSO 电子邮件声明相同的字段（通常为 user.mail）。

6. 将**预配状态**设置为**开启**并保存。

**关键：**用于 SCIM 电子邮件和 SSO 电子邮件的属性必须相同。不匹配是登录失败的最常见原因。如需故障排除，请参阅**[Microsoft Entra ID SSO/SCIM 电子邮件不匹配](https://support.claude.com/en/articles/13917829)**。

## 步骤 4 — 分配人员和组

1. 在**用户和组**中，分配应该有权访问 Claude 的人员或组。

2. 只有分配的人员和组才会被预配并允许进行 SSO。

## 步骤 5 — 验证

1. 如果您启用了 SCIM，触发预配周期并确认人员出现在 Claude 的管理设置中。

2. 让测试用户完成 SSO 登录并验证他们进入您的组织工作区。

---

## 需要帮助？

请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的端到端流程，包括域验证和选择预配方法。如果您遇到问题，请使用您的 Entra 租户 ID 和 SAML 配置的屏幕截图联系**[我们的支持团队](https://support.claude.com/en/articles/9015913)**。