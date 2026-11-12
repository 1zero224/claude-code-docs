---
title: 创建 Admin API 密钥
url: https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys
description: 为您的 Claude Console 或 Claude Enterprise 组织创建 Admin API 密钥。
---

Admin API 密钥可用于对本指南**管理**部分中的每个 API 进行身份验证：[Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)、[Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api)、[Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)、[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)、[Claude Tag API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api)、[支出限额 API](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)、[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)、[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api) 和[速率限制 API](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api)。您无需为每个 API 单独创建密钥。唯一的例外是 Admin API 的服务账户（service-account）、联合身份颁发者（federation-issuer）和联合身份规则（federation-rule）端点，它们仅接受具有 `org:admin` 作用域的 OAuth bearer 令牌。请参阅[获取 OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。

在何处创建密钥取决于您的组织使用的是哪款 Claude 产品。

## 您需要哪种密钥？

| 您的组织                                                      | 创建密钥的位置                                                                                                                | 密钥前缀                 | 谁可以创建                                                                                             | 适用于                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Claude Console**（Claude Platform，`platform.claude.com`） | Claude Console 中的[管理员密钥](https://platform.claude.com/settings/admin-keys)页面（`platform.claude.com/settings/admin-keys`） | `sk-ant-admin01-...` | 具有**管理员**角色的组织成员                                                                                  | [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)、[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)、[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)、[速率限制 API](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api)、[Claude Code 分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-code-analytics-api)，以及 Compliance API 的[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)                                                                                                                                                                                                                         |
| **Claude Enterprise**（`claude.ai`）                        | [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access)                                                  | `sk-ant-api01-...`   | 父组织的**主要所有者**（所有关联组织）。**组织所有者（organization owner）**&#x53EF;以创建仅携带 Compliance API 作用域、且仅限于其自身组织的密钥 | [用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)（Admin API 的成员、邀请和群组端点）、[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)、[Claude Tag API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api)、[Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)、[Claude Enterprise Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api)、[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 以及[支出限额 API](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)，具体取决于您选择的[作用域](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#choose-scopes-for-a-claude-enterprise-key) |

在一个组织中创建的密钥不能用于管理另一个组织。如果您的公司同时使用 Claude Console 和 Claude Enterprise，请在每个组织中各创建一个密钥。

## 为 Claude Console 组织创建密钥

<Steps>
  <Step title="以组织管理员身份登录">
    只有具有**管理员**角色的组织成员才能创建 Admin API 密钥。请参阅[组织角色和权限](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#organization-roles-and-permissions)。
  </Step>

  <Step title="打开管理员密钥设置">
    在 Claude Console 中，前往[管理员密钥](https://platform.claude.com/settings/admin-keys)页面（`platform.claude.com/settings/admin-keys`）。
  </Step>

  <Step title="创建密钥">
    点击**创建管理密钥**，为其命名，选择[密钥过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)，然后点击**添加**。Claude Console 密钥没有可选择的作用域；每个密钥都拥有对所有接受 Admin API 密钥的端点的完全访问权限（本页顶部提到的服务账户和联合身份端点不接受 Admin API 密钥）。
  </Step>

  <Step title="复制并存储密钥">
    复制显示的密钥（以 `sk-ant-admin01-` 开头）并将其存储在您的密钥管理器中。完整密钥仅显示一次。
  </Step>
</Steps>

## 为 Claude Enterprise 组织创建密钥

<Steps>
  <Step title="以主要所有者或组织所有者身份登录">
    Claude Enterprise 父组织的**主要所有者**可以创建能够访问每个关联组织的密钥，或仅限于单个组织的密钥。**组织所有者**可以创建仅具有 Compliance API 作用域、且仅限于其自身组织的密钥。
  </Step>

  <Step title="打开 API 设置">
    前往 [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access) 并找到**密钥**部分。
  </Step>

  <Step title="点击 + 创建密钥">
    为密钥命名，并从[作用域表](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#choose-scopes-for-a-claude-enterprise-key)中选择您需要的作用域。主要所有者可以在单个密钥上组合来自不同 API 的作用域（例如 `read:analytics` 和 `read:spend_limits`）。
  </Step>

  <Step title="复制并存储密钥">
    复制显示的密钥（以 `sk-ant-api01-` 开头）并将其存储在您的密钥管理器中。完整密钥仅显示一次。
  </Step>
</Steps>

## 为 Claude Enterprise 密钥选择作用域

创建 Claude Enterprise 密钥时，请选择您计划调用的 API 所需的每个作用域（scope）。作用域在创建时即固定；若要稍后添加作用域，请创建新密钥。

| 要调用……                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 选择这些作用域                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| Admin API [用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)：列出和查找成员及邀请；读取自定义角色及其权限                                                                                                                                                                                                                                                                                                                                                                                                                                           | `read:members`                |
| Admin API [用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)：更改成员角色、移除成员、[将成员从所有位置登出](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#sign-a-member-out-everywhere)（beta）、创建和撤回邀请（如需在角色变更或邀请中分配 `managed` 角色，还需选择 `write:rbac_groups`）                                                                                                                                                                                                                                                                   | `write:members`               |
| Admin API [用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)：读取群组及其成员                                                                                                                                                                                                                                                                                                                                                                                                                                                         | `read:rbac_groups`            |
| Admin API [用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)：创建、重命名和删除群组；添加和移除群组成员；与 `write:members` 一起使用时，可分配 `managed` 角色（在邀请或角色变更中）或在邀请中分配群组                                                                                                                                                                                                                                                                                                                                                                               | `write:rbac_groups`           |
| [插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)：读取插件、版本、安装设置、共享和市场；下载插件文件，包括成员个人插件的文件；在连接市场之前对其进行验证                                                                                                                                                                                                                                                                                                                                                                                                                          | `read:plugins`                |
| [插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)：上传插件和版本；选择向成员提供哪个版本；删除插件，包括成员的个人插件；设置和移除安装设置；设置市场的默认安装设置；在连接市场之前对其进行验证（如需读取插件，还需选择 `read:plugins`）                                                                                                                                                                                                                                                                                                                                                                           | `write:plugins`               |
| [Claude Tag API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api)：读取 Claude Tag 在 Slack 频道和 Slack 工作区上的设置；列出访问包及其附加位置（没有其他作用域可授予对 Claude Tag API 的访问权限）                                                                                                                                                                                                                                                                                                                                                                             | `read:claude_tag`             |
| [Claude Tag API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api)：设置和移除频道；更改设置；附加和分离访问包（如需同时读取，还需选择 `read:claude_tag`）                                                                                                                                                                                                                                                                                                                                                                                                              | `write:claude_tag`            |
| [支出限额 API](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)：读取成员的有效支出限额和提额请求                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `read:spend_limits`           |
| [支出限额 API](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)：设置或清除每用户支出限额；批准或拒绝提额请求                                                                                                                                                                                                                                                                                                                                                                                                                                                | `write:spend_limits`          |
| [Claude Enterprise Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api)：参与度、采用率、成本和使用情况报告；[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)：令牌用量和成本报告                                                                                                                                                                                                                                                                                                                          | `read:analytics`              |
| [Compliance API 活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)：组织范围的活动事件                                                                                                                                                                                                                                                                                                                                                                                                                                           | `read:compliance_activities`  |
| [Compliance API 聊天、文件和项目端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-content-data)以及 [Compliance API 会话端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions)：读取聊天、文件、项目、会话记录和[组织用户](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organization-users)；与 `read:compliance_org_data` 一起使用时，在端点返回该信息的情况下，还可读取谁作出了 [Security posture 声明](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#who-attested-and-comments)：用户 ID | `read:compliance_user_data`   |
| [Compliance API 聊天、文件和项目端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-content-data)：删除聊天、文件和项目                                                                                                                                                                                                                                                                                                                                                                                                                                    | `delete:compliance_user_data` |
| [Compliance API 组织端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data)：读取组织元数据和有效设置，以及组织的 [Security posture](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference)；[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)：每个 `GET` 端点，包括文件下载                                                                                                                                                                                                                 | `read:compliance_org_data`    |
| 使用单个只读作用域访问 Admin API [用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)读取端点、每个 Compliance API 读取端点，以及每个[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api) `GET` 端点（包括文件下载）（适用于安全审计集成；不包括支出限额或分析 API）                                                                                                                                                                                                                                                                                                 | `read:org_audit`              |

必须先为您的组织启用 Compliance 和 Analytics API，才能使用具有这些作用域的密钥。请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#set-up-the-compliance-api) 和[获取 Claude Enterprise Analytics API 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api#get-access-to-the-claude-enterprise-analytics-api)。

只有当您的组织可以使用插件 API 时，您才能创建和使用带有 `read:plugins` 或 `write:plugins` 的密钥。[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api) 指南说明了哪些组织可以使用它。在这些组织中，带有 `read:org_audit` 或 `read:compliance_org_data` 的密钥也可以调用插件 API 的读取端点（包括文件下载），但不能进行市场验证。`read:plugins` 和 `write:plugins` 作用域仅作用于创建该密钥的组织。请参阅[插件 API 作用域](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#scopes)和[读取同一父组织下的其他组织](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#reading-another-organization-under-the-same-parent)。

## 使用密钥

在每个请求的 `x-api-key` 标头中传递密钥。有关完整的请求示例，请参阅各 API 的文档。

超出密钥作用域的调用会返回 `403 Forbidden`，并附带一条消息，列出该密钥拥有的作用域以及该端点所需的作用域。

## 后续步骤

<CardGroup cols={2}>
  <Card title="Admin API" href="https://platform.claude.com/docs/zh-CN/manage-claude/admin-api">
    管理组织成员、工作区和 API 密钥。
  </Card>

  <Card title="支出限额 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api">
    为您的 Claude Enterprise 组织设置每成员支出限额并审核提额请求。
  </Card>

  <Card title="Analytics API" href="https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api">
    报告 Claude Code 生产力或 Claude Enterprise 参与度和采用率。
  </Card>

  <Card title="Compliance API" href="https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api">
    审计活动并检索或删除整个组织中的用户内容。
  </Card>
</CardGroup>
