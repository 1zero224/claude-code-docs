# Ping Identity SSO 设置

本指南介绍如何使用 PingOne 或 PingFederate 作为身份提供商来配置 Claude。适用于 Team 计划、Enterprise 计划和 Console 组织。

## 前置条件

- Claude Team 计划、Enterprise 计划或具有父组织的 Console 组织（请参阅**[启用 SSO 前的重要注意事项](https://support.claude.com/en/articles/10276682)**了解 Console 父组织的要求）

- Owner 或 Primary Owner 角色（Team 和 Enterprise）或 Admin 角色（Console）

- PingOne Environment Admin 或 PingFederate Admin 访问权限

- 在 Claude 的身份和访问设置中验证您的域——请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的设置路径，包括域验证

## 查找配置值的位置

下面引用的 ACS URL、Entity ID 和 SCIM 凭证在您的身份和访问设置中的 WorkOS 设置流程中提供——不是通过联系支持团队获得。

- **Team 和 Enterprise 计划：**转到**[claude.ai/admin-settings/identity](https://claude.ai/admin-settings/identity)**

- **Console 组织：**转到**[platform.claude.com/settings/identity](https://platform.claude.com/settings/identity)**

在那里启动 SSO 设置流程，并在执行以下步骤时将其与您的 Ping 管理控制台并排打开。

---

## PingOne 设置

### 步骤 1 — 创建应用程序连接

1. 在 PingOne 管理控制台中，转到**连接 → 应用程序 → + 添加应用程序**。

2. 选择“SAML 应用程序”。

3. 将其命名为“Claude”，然后单击“配置”。

### 步骤 2 — 配置 SAML

1. 选择“手动输入”，并从 WorkOS 设置流程中提供 SP 详情：**ACS URL** 和 **Entity ID**。

2. 下载 PingOne **IdP 元数据**，并在系统提示时在 WorkOS 设置流程中上传。

3. 在**属性映射**中，将 `email` 映射到 PingOne **电子邮件地址**属性。

### 步骤 3 — 启用 SCIM 预配

**注意：**SCIM 预配仅在 Enterprise 计划和符合条件的 Console 组织上可用。如果您使用的是 Team 计划，请跳过此步骤——您可以改用 JIT 预配。请参阅**[设置 JIT 或 SCIM 预配](https://support.claude.com/en/articles/13133195)**。

1. 在应用程序设置中，转到**预配**选项卡。

2. 启用**出站预配**，并输入来自 WorkOS 设置流程的 SCIM 端点 URL 和访问令牌。

3. 将 `emails[primary].value` 映射到 PingOne **电子邮件地址**属性——与 SAML 中使用的属性相同。

**关键：**确保 SAML 和 SCIM 使用相同的属性源。请参阅**[Ping Identity SSO/SCIM 电子邮件不匹配](https://support.claude.com/en/articles/13917875)**进行故障排除。

### 步骤 4 — 分配人口

1. 在**人口**下，分配应该访问 Claude 的用户人口。

2. 启用应用程序，然后单击“保存”。

---

## PingFederate 设置

PingFederate 配置因版本和部署而异。一般方法是：

1. 使用来自 WorkOS 设置流程的 SP 元数据创建新的 **SP 连接**。

2. 配置**属性契约**以包含 `email`。

3. 在**适配器映射**中将电子邮件属性映射到该人的主要电子邮件字段。

4. 对于 SCIM（仅限 Enterprise 计划和符合条件的 Console 组织），配置针对来自 WorkOS 设置流程的 SCIM 端点的出站预配通道。

请联系**[我们的支持团队](https://support.claude.com/en/articles/9015913)**获取 PingFederate 特定的指导。

---

## 需要帮助？

请参阅**[设置单点登录](https://support.claude.com/en/articles/13132885)**了解完整的端到端流程，包括域验证和选择预配方法。如果遇到问题，请联系**[我们的支持团队](https://support.claude.com/en/articles/9015913)**并提供您的 Ping 环境详情。