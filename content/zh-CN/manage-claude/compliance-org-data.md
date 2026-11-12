---
title: 列出组织、用户、角色、群组和设置
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data
description: 通过 Compliance API 枚举您的父组织下的各个组织（及其用户、角色和群组），并读取每个组织的有效设置。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

<Check>
  **所需作用域：** Compliance Access Key 上的 `read:compliance_org_data`。用户、用户偏好和群组成员端点则需要 `read:compliance_user_data`。

  唯一被接受的密钥类型是在 claude.ai 中创建的 Compliance Access Key（`sk-ant-api01-...`）。请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access) 以配置一个。使用 Admin API 密钥（`sk-ant-admin01-...`）进行身份验证的调用会返回 [403 Forbidden](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#403-forbidden)。
</Check>

本页上的端点公开了 Claude Enterprise 组织的目录侧信息：其关联组织、每个组织中的用户、每个组织上定义的角色，以及其"role-based access control"（基于角色的访问控制），即 RBAC 群组或"System for Cross-domain Identity Management"（跨域身份管理系统），即 SCIM 配置的群组及其成员。您可以使用它们为 eDiscovery 用户列表提供初始数据、构建报告仪表板，以及将群组成员资格与外部记录系统进行核对。覆盖父组织的 Compliance Access Key 会返回其下每个关联组织的数据，因此单个密钥即可访问整个组织树。[有效设置端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#get-effective-organization-settings)是对目录的补充：它返回某一组织实际生效的数据隐私、安全和功能设置。

## 列出组织

[列出组织](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/list)端点返回密钥所绑定的父组织下的每个组织。

以下调用列出您的父组织下的每个组织。响应是一个按 `created_at` 升序排序的组织记录 `data` 数组，外加用于分页的 `has_more` 和 `next_page`。当 `has_more` 为 `true` 时，请在下一次请求中将返回的 `next_page` 令牌原样作为 `page` 查询参数传回。有关 `limit` 和 `page` 参数的默认值和范围，请参阅 API 参考中的[列出组织](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/list)。

```bash cURL
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/organizations" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json Response
{
  "data": [
    {
      "uuid": "91012d09-e48b-438e-a489-1bebfd8fa6f9",
      "name": "Acme Engineering",
      "created_at": "2025-06-01T10:00:00Z"
    },
    {
      "uuid": "5a1b2c3d-4e5f-6789-abcd-ef0123456789",
      "name": "Acme Legal",
      "created_at": "2025-07-15T14:30:00Z"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

`uuid` 字段是下游查找的规范标识符。下表将其与 Compliance API 中的其他组织标识符进行对应：

| 字段                   | 位置                                                                                                                                                                                                                                                                                                                                          | 与 `uuid` 的关系                                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `{org_uuid}`         | 本页上各按组织划分的端点的路径参数                                                                                                                                                                                                                                                                                                                           | 相同的值                                                                     |
| `organization_uuid`  | 活动源、聊天、项目和会话记录                                                                                                                                                                                                                                                                                                                              | 相同的值；可直接在这两个字段上进行关联                                                      |
| `organization_id`    | 活动源、聊天和项目记录                                                                                                                                                                                                                                                                                                                                 | 同一组织，带 `org_` 前缀。在聊天和项目记录上已弃用；请改用 `organization_uuid`。                   |
| `organization_ids[]` | [查询活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)、[检索聊天和消息](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-content-data#retrieve-chats-and-messages)以及[远程会话列表](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-remote-sessions)上的筛选器（本地会话列表没有组织筛选器） | 接受 `uuid` 或带 `org_` 前缀的形式                                                |
| `organization_id`    | [有效组织设置](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#get-effective-organization-settings)响应                                                                                                                                                                                                                    | 相同的值，裸 UUID；此响应**不**使用 `organization_id` 在活动源、聊天和项目记录上所携带的带 `org_` 前缀的形式 |

大多数其他 Anthropic API 使用带 `org_` 前缀的形式。

要跟踪组织成员资格随时间的变化，请定期重新列出此端点，每次遍历时沿 `next_page` 令牌翻阅每一页。活动源也会通过 `org_deletion_requested`、`org_deleted_via_bulk`、`org_parent_join_proposal_created` 和 `org_join_proposal_decided` 活动类型呈现成员资格事件；请参阅[查询活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)。

## 列出组织用户

[列出组织用户](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/users/list)端点返回某一组织的分页用户记录列表。

此端点需要 `read:compliance_user_data`，而不是 `read:compliance_org_data`。如果您打算将 Compliance Access Key 用于目录枚举，请在创建时同时包含这两个作用域；否则调用会返回 [403 Forbidden](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#403-forbidden)。

有关 `limit` 和 `page` 查询参数的默认值和范围，请参阅 API 参考中的[列出组织用户](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/users/list)。

结果按加入组织的日期升序排序。与活动源的 `before_id`/`after_id` 游标（请参阅[分页结果](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed#paginate-results)）不同，目录端点使用 `next_page` 令牌进行分页：当 `has_more` 为 `true` 时，请在下一次请求中将 `next_page` 原样作为 `page` 查询参数传回。

```bash cURL
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/organizations/91012d09-e48b-438e-a489-1bebfd8fa6f9/users" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21" \
  --data-urlencode "limit=500"
```

```json Response
{
  "data": [
    {
      "id": "user_01WxYzAbCdEfGhJkMnPqRsTu",
      "full_name": "Priya Sharma",
      "email": "priya@example.com",
      "organization_role": "admin",
      "created_at": "2025-06-01T10:00:00Z"
    }
  ],
  "has_more": true,
  "next_page": "page_8aW5kZXgicG9zaXRpb25fdG9rZW5fOTE0"
}
```

此处返回的用户 ID 与 [查询活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed) 的 `actor_ids[]` 筛选器所接受的 `user_...` 标识符相同。[检索聊天和消息](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-content-data#retrieve-chats-and-messages)以及[远程会话列表](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-remote-sessions)上的 `user_ids[]` 筛选器也接受相同的标识符。[本地会话列表](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-local-sessions)没有用户筛选器，因此请通过每个会话对象上的 `user.id` 来归属本地会话。`organization_role` 字段包含用户在所列组织内的内置成员级别（`admin`、`billing`、`claude_code_user`、`developer`、`managed`、`membership_admin`、`owner`、`primary_owner` 或 `user` 之一），这一维度独立于 [列出角色](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-roles) 返回的任何自定义 RBAC 角色分配。典型的电子取证（eDiscovery）流程会列出一个或多个组织的用户，根据您自己的外部记录进行筛选，然后将得到的 ID 用于聊天和项目查询。

用户只有在作为组织的活跃成员期间才会出现在此处。被移除的用户会立即从列表中删除。他们的历史活动在整个保留窗口内仍可通过活动源查询，并以相同的 `user_...` ID 进行索引。

## 列出用户偏好

`GET /v1/compliance/organizations/{org_uuid}/user_preferences` 端点返回某个组织的当前成员在 claude.ai 中保存为 **Claude 指令**的文本（`conversation_preferences`），以及在 Cowork 中保存为**全局指令**的文本（`cowork_global_instructions`）。两者都是账户级设置，因此成员所属的每个组织下都会返回相同的文本。此端点处于 beta 阶段。

此端点需要 `read:compliance_user_data`，而不是 `read:compliance_org_data`；使用缺少 `read:compliance_user_data` 的密钥进行调用会返回 [403 Forbidden](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#403-forbidden)。

默认情况下，响应仅列出至少设置了这两个值之一的成员。传入 `include_unset=true` 可列出每个当前成员，未设置的值为 `null`。若只想返回特定成员，请在 `user_ids[]` 中传入最多 10 个用户 ID。结果使用 `limit`（默认 20，最大 100）和 `page` 进行分页。与[组织用户端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organization-users)一样，请将 `next_page` 原样作为 `page` 传回；同时在每一页请求中发送相同的 `include_unset` 和 `user_ids[]` 值，否则调用会返回 [400 Bad Request](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#invalid-pagination-cursor)。某一页包含的结果可能少于 `limit`，甚至为空，但仍不一定是最后一页，因此请持续请求页面，直到 `next_page` 为 `null`。

```bash cURL
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/organizations/91012d09-e48b-438e-a489-1bebfd8fa6f9/user_preferences" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json Response
{
  "data": [
    {
      "user": {
        "id": "user_01WxYzAbCdEfGhJkMnPqRsTu",
        "email_address": "priya@example.com"
      },
      "conversation_preferences": "Keep answers brief and cite sources.",
      "cowork_global_instructions": null
    }
  ],
  "has_more": false,
  "next_page": null
}
```

## 列出角色

[列出 Compliance 角色](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/roles/list)端点返回某一组织上定义的分页角色记录列表，[获取 Compliance 角色](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/roles/retrieve)则按 ID 返回单个角色。

两个角色端点都需要 `read:compliance_org_data`。列表端点接受与[组织用户端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organization-users)相同的 `limit` 和 `page` 参数。

```bash cURL
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/organizations/91012d09-e48b-438e-a489-1bebfd8fa6f9/roles" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json Response
{
  "data": [
    {
      "id": "rbac_role_01N2pQrS8tUvWxYz5AbCdEfGh",
      "name": "Compliance Reviewer",
      "description": "Read-only access to chat and project content for legal review.",
      "created_at": "2025-06-01T10:00:00Z",
      "updated_at": "2025-06-15T14:30:00Z"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

有关完整的角色记录结构，请参阅[列出 Compliance 角色](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/roles/list)响应模式。要列出当前授予某个角色的权限，请使用[列出 Compliance 角色权限](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/roles/permissions/list)。要审计历史角色分配和权限变更，请通过活动源查询 RBAC 活动类型（例如 `rbac_role_assigned` 和 `rbac_role_permission_added`）；请参阅[筛选活动](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed#filter-activities)。

## 列出群组和成员

[列出 Compliance 群组](https://platform.claude.com/docs/zh-CN/api/compliance/groups/list)端点返回 RBAC 和 SCIM 配置群组的分页列表，[获取 Compliance 群组](https://platform.claude.com/docs/zh-CN/api/compliance/groups/retrieve)则按 ID 返回单个群组。[列出 Compliance 群组成员](https://platform.claude.com/docs/zh-CN/api/compliance/groups/members/list)端点返回某一群组的成员。

群组列表和检索端点需要 `read:compliance_org_data`。成员端点需要 `read:compliance_user_data`。请在创建密钥时同时包含这两个作用域，以便端到端地遍历群组。两个列表端点都接受与[组织用户端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organization-users)相同的 `limit` 和 `page` 参数。

有关完整的群组记录结构，请参阅[列出 Compliance 群组](https://platform.claude.com/docs/zh-CN/api/compliance/groups/list)响应模式。`roles` 数组列出分配给该群组的角色 ID，与[列出角色](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-roles)中的 ID 相匹配。`source_type` 是区分通过 claude.ai 手动创建的群组（`direct`）与通过 SCIM 从外部身份提供商同步的群组（`scim`）的判别字段。

列出群组，然后为每个群组列出其成员：

```bash cURL
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/groups" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json Response
{
  "data": [
    {
      "id": "rbac_group_01P9qRsTuVwXyZa2BcDeFgHjK",
      "name": "Engineering",
      "description": "Engineering team members",
      "source_type": "scim",
      "roles": ["rbac_role_01N2pQrS8tUvWxYz5AbCdEfGh"],
      "created_at": "2025-06-01T10:00:00Z",
      "updated_at": "2025-06-15T14:30:00Z"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

对于每个群组 ID，列出其成员：

```bash cURL
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/groups/rbac_group_01P9qRsTuVwXyZa2BcDeFgHjK/members" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json Response
{
  "data": [
    {
      "user_id": "user_01WxYzAbCdEfGhJkMnPqRsTu",
      "email": "priya@example.com",
      "created_at": "2025-06-01T10:00:00Z",
      "updated_at": "2025-06-15T14:30:00Z"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

有关完整的成员记录结构，请参阅[列出 Compliance 群组成员](https://platform.claude.com/docs/zh-CN/api/compliance/groups/members/list)响应模式。`user_id` 字段与活动源、聊天列表和远程会话列表所接受的 `user_...` 标识符相同；它也与本地会话对象以及用户拥有的远程会话对象上的 `user.id` 相匹配（代理拥有的远程会话则在 `started_by_user.id` 中携带人类用户的 ID）。要获取成员的全名，请通过组织用户列表进行查找。

## 获取有效组织设置

[获取有效组织设置](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/settings/retrieve)端点返回您的父组织下某一组织的生效设置：即在应用监管限制（例如 HIPAA）、功能可用性规则、组织类型默认值以及功能间依赖关系之后的强制执行状态，这可能与管理员所配置的内容不同。您可以使用它来证明保留窗口、内容脱敏、单点登录强制执行、IP 允许列表和会话时长控制与您记录在案的基线相符，而无需管理员 Console 访问权限。

此端点需要 `read:compliance_org_data`；没有该作用域的密钥将返回 [403 Forbidden](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#403-forbidden)。目标必须是父组织的关联组织之一：父组织本身不是有效的目标。以下所有情况都会返回相同的 [404 Not Found](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#404-not-found)：

* 未知的组织
* 不是有效 UUID 的组织 ID
* 不在您父组织树中的组织
* 尚未获得此端点访问权限的父组织

因此，404 不会透露某个组织是否存在。设置端点是按父组织单独启用的，与 Compliance API 的其余部分分开；如果每个请求都返回 404，请联系您的 Anthropic 代表。

<Note>
  在 2026 年 6 月 30 日之前，此端点需要单独的 `read:compliance_org_settings` 作用域。该作用域已停用：创建密钥时无法再选择或授予它，并且仅携带该已停用作用域的密钥会返回 [403 Forbidden](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#403-forbidden)。请改为创建一个带有 `read:compliance_org_data` 的新 Compliance Access Key。
</Note>

```bash cURL
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/organizations/91012d09-e48b-438e-a489-1bebfd8fa6f9/settings" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

响应是一个带类型的设置行列表，出现哪些行因组织而异：组织管理员无法更改的设置（因为它由 Anthropic 策略控制或对该组织不可用）会从列表中省略。请将缺失的行视为"此组织的管理员无法控制"，而不是"关闭"。以下简略示例展示了响应可能包含的三行：

```json Response
{
  "type": "effective_organization_settings",
  "organization_id": "91012d09-e48b-438e-a489-1bebfd8fa6f9",
  "settings": [
    {
      "name": "data_retention_periods",
      "type": "data_retention",
      "value": {
        "chat": {
          "type": "fixed",
          "timescale": "day",
          "duration": 90
        }
      }
    },
    {
      "name": "content_redaction_enabled",
      "type": "boolean",
      "value": true
    },
    {
      "name": "ip_allowlist_ip_ranges",
      "type": "string_list",
      "value": ["10.0.0.0/8", "203.0.113.0/24"]
    }
  ],
  "api_keys": [
    {
      "type": "compliance_api_key",
      "id": "apikey_01Hx7k2mP9nQ4rS6tU8vW0xY",
      "name": "Compliance Export Key",
      "scopes": ["read:compliance_activities", "read:compliance_org_data"],
      "is_active": true,
      "created_at": "2026-03-14T09:30:00Z",
      "created_by_id": "user_01Jz3a4bC5dE6fG7hI8jK9lM",
      "expires_at": null
    }
  ]
}
```

每一行都携带 `name`、`type` 和 `value`；`type` 字段（`boolean`、`integer`、`string_list`、`provisioning_mode` 或 `data_retention`）告诉您 `value` 的结构。设置名称的完整列表以及每种类型的 `value` 模式，请参阅 API 参考中的[获取有效组织设置](https://platform.claude.com/docs/zh-CN/api/compliance/organizations/settings/retrieve)。

`api_keys` 数组列出为您的父组织配置的所有 Compliance Access Key，因此无论您查询哪个关联组织，返回的列表都相同。每个条目包含密钥的 `type`（`compliance_api_key`）、`id`、`name`、`scopes`、`is_active` 标志、`created_at` 和 `expires_at` 时间戳，以及 `created_by_id`（创建该密钥的用户 ID，或 `null`）。密钥的秘密值永远不会被返回。已停用的密钥也会包含在内，其值为 `is_active: false`，以便您审查以前拥有访问权限的密钥；仅带有已停用的 `read:compliance_org_settings` 作用域的密钥也仍保留在列表中，以便于审计和清理，尽管该作用域已不再授予访问权限。

顶层的 `organization_id` 是组织的裸 UUID：与组织列表中的 `uuid` 值相同，而不是 `organization_id` 在活动源、聊天和项目记录上所携带的带 `org_` 前缀的形式（请参阅[组织标识符表](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organizations)）。

各行反映的是实际生效的状态，而不是最后存储的配置，例如：

* `sso_provisioning_mode` 仅在启用目录同步时才报告已配置的 SCIM 模式。
* `ip_allowlist_enabled` 仅在允许列表处于开启状态且至少有一个有效范围时才为 `true`。
* 只要代码执行处于关闭状态，`code_execution_network_egress_enabled` 就为 `false`。

响应反映读取时的状态；不会进行任何快照。对这些设置中大多数的更改会作为事件呈现在[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)中；请使用此端点获取当前已解析的状态，并使用该信息流来审计谁在何时更改了什么。有关 Anthropic 推荐的安全控制项的当前结果，请参阅 [Compliance API 中的 Security posture](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview)。

## 读取插件和插件市场

您组织的插件和插件市场属于 Admin API 的[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)，而不是 Compliance API，但带有 `read:compliance_org_data` 的 Compliance Access Key 可以直接调用插件 API 的读取端点：

* 列出插件和检索插件
* 列出插件的版本、检索某个版本以及下载某个版本的文件
* 列出插件的安装设置及其共享
* 列出插件市场和检索某个市场

请在每个请求中发送 `anthropic-beta: ce-plugins-2026-09-01` 标头；如果没有该标头，这些端点会返回 404 Not Found（请参阅插件 API 的[错误响应](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#error-responses)）。每次调用读取一个组织。覆盖父组织的密钥会读取创建该密钥时所在的关联组织。要[读取其他组织](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#reading-another-organization-under-the-same-parent)，请传入 `organization_id` 查询参数（关联组织的 UUID 或其带 `org_` 前缀的 ID）。仅限于一个组织的密钥只能读取该组织。这些调用计入 Admin API 的[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#rate-limiting)，而不是 Compliance API 的速率限制。

创建或更改插件、其安装设置或市场设置需要带有 `write:plugins` 的 Admin API 密钥。在读取调用中，只有下载成员个人市场中插件的文件才会记录[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)事件（`claude_plugin_archive_accessed`）。有关每个端点、请求示例以及 [API 记录的事件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#activity-feed-events)，请参阅[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)指南。

## 后续步骤

<CardGroup cols={2}>
  <Card title="Compliance 组织 API 参考" href="https://platform.claude.com/docs/zh-CN/api/compliance/organizations">
    每个组织、用户、角色、群组和设置端点的完整请求和响应模式。
  </Card>

  <Card title="处理 Compliance API 错误" href="https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors">
    逐字的错误负载以及每种错误的修复方法。
  </Card>
</CardGroup>
