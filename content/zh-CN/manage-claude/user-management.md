---
title: 用户管理
url: https://platform.claude.com/docs/zh-CN/manage-claude/user-management
description: 使用 Admin API 管理您 Claude Enterprise 组织中的人员：列出成员并更改角色、发送和撤回邀请、管理群组以及读取自定义角色。
---

本页介绍如何使用 [Admin API](https://platform.claude.com/docs/zh-CN/api/beta/organization) 以编程方式管理您的 **Claude Enterprise**（claude.ai）组织中的人员：

* 列出成员并按电子邮件地址查找成员
* 更改成员的角色
* 移除成员，或在不移除成员的情况下将其从所有位置登出（beta）
* 发送和撤回邀请
* 管理您企业的群组及其成员
* 读取您组织的自定义角色

对于 Claude Console（Claude Platform）组织，请参阅 [Claude Console 的 Admin API 指南](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)。

## 您的组织可以使用哪些端点？

Admin API 是位于 `https://api.anthropic.com/v1/organizations/` 下的一组统一的端点。Claude Console 和 Claude Enterprise 组织使用不同的 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)进行身份验证，并且各自可以访问不同的端点子集：

| 端点                                                                                                                                                                                                                                                                                                                           | Claude Console（Claude Platform）                                                                    | Claude Enterprise（claude.ai）                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [成员](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#members)和[邀请](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#invites)                                                                                                                                                        | 可用；请参阅 [Admin API 指南](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)              | 可用（本页）                                                                                                                                                                                                                                               |
| [群组](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#groups)                                                                                                                                                                                                                                            | 不可用                                                                                                | 可用（本页）                                                                                                                                                                                                                                               |
| [自定义角色](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#custom-roles)                                                                                                                                                                                                                                   | 不可用                                                                                                | 可用，只读（本页）                                                                                                                                                                                                                                            |
| [插件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api)和 [Claude Tag](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api)                                                                                                                                                                    | 不可用                                                                                                | 以 beta 形式提供                                                                                                                                                                                                                                          |
| [支出限额](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)                                                                                                                                                                                                                                                | 不可用                                                                                                | 可用                                                                                                                                                                                                                                                   |
| 用量和成本报告                                                                                                                                                                                                                                                                                                                      | 可用；请参阅[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) | 通过[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)以及 [Claude Enterprise 分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api#get-access-to-the-claude-enterprise-analytics-api) 提供 |
| [工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)、[API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#api-keys)、[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api)以及 [Admin API 指南](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)中的其他端点 | 可用                                                                                                 | 不可用                                                                                                                                                                                                                                                  |

成员和邀请对两种组织类型使用相同的端点；本页记录它们在 Claude Enterprise 中的行为，包括 Claude Enterprise 的[组织角色](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#organization-roles)。群组和自定义角色端点仅适用于 Claude Enterprise。

<Check>
  **需要带作用域的 Admin API 密钥**

  这些端点需要具有与请求相匹配的作用域的 Admin API 密钥。成员和邀请的 `GET` 端点以及所有自定义角色端点需要 `read:members`（没有单独的角色作用域）。成员和邀请的 `POST` 和 `DELETE` 端点需要 `write:members`。群组 `GET` 端点需要 `read:rbac_groups`，群组 `POST` 和 `DELETE` 端点需要 `write:rbac_groups`。分配 `managed` 角色或在邀请中分配群组时，除 `write:members` 外还需要 `write:rbac_groups`。带有 `read:org_audit` 作用域（用于安全审计集成的只读作用域）的密钥也可以调用本页上的所有 `GET` 端点以及 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的读取端点。请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization)，了解在何处创建密钥以及应选择哪些作用域。在每个请求中通过 `x-api-key` 标头传递密钥，并同时传递 [`anthropic-version`](https://platform.claude.com/docs/zh-CN/api/versioning) 标头。
</Check>

## 概述

本页涵盖五种资源：

| 资源        | 端点                                                                                                                                                                                                                                                           | 用途                                         |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------ |
| **成员**    | `GET /v1/organizations/users`<br />`GET /v1/organizations/users/{user_id}`<br />`POST /v1/organizations/users/{user_id}`<br />`DELETE /v1/organizations/users/{user_id}`<br />`POST /v1/organizations/users/{user_id}/sign_out`（beta）                        | 列出组织的成员或按电子邮件查找成员；更改成员的角色；移除成员；将成员从所有位置登出。 |
| **邀请**    | `POST /v1/organizations/invites`<br />`GET /v1/organizations/invites`<br />`GET /v1/organizations/invites/{invite_id}`<br />`DELETE /v1/organizations/invites/{invite_id}`                                                                                   | 邀请某人加入组织、跟踪邀请状态，并在邀请被接受前撤回。                |
| **群组**    | `GET /v1/organizations/rbac_groups`<br />`GET /v1/organizations/rbac_groups/{rbac_group_id}`<br />`POST /v1/organizations/rbac_groups`<br />`POST /v1/organizations/rbac_groups/{rbac_group_id}`<br />`DELETE /v1/organizations/rbac_groups/{rbac_group_id}` | 读取企业的群组以及附加到每个群组的自定义角色；创建、重命名和删除群组。        |
| **群组成员**  | `GET /v1/organizations/rbac_groups/{rbac_group_id}/members`<br />`POST /v1/organizations/rbac_groups/{rbac_group_id}/members`<br />`DELETE /v1/organizations/rbac_groups/{rbac_group_id}/members/{user_id}`                                                  | 读取群组的成员；添加和移除成员。                           |
| **自定义角色** | `GET /v1/organizations/rbac_roles`<br />`GET /v1/organizations/rbac_roles/{rbac_role_id}`<br />`GET /v1/organizations/rbac_roles/{rbac_role_id}/permissions`                                                                                                 | 读取组织的自定义角色以及每个角色授予的权限。                     |

自定义角色及其群组附加关系在 [claude.ai 组织设置](https://claude.ai/admin-settings)中管理；API 可以读取它们，但无法更改它们。

## 快速入门

将 `ANTHROPIC_API_KEY` 环境变量设置为您的 Admin API 密钥。本页上的示例会从该变量读取密钥。SDK 和 `ant` CLI 会为您发送 `Authorization` 和 `anthropic-version` 标头。

列出组织的成员，最新的排在最前：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/users?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:users list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  users = client.organization.users.list(limit=20)

  for user in users:
      print(f"{user.id}: {user.email} ({user.role})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const users = await client.organization.users.list({ limit: 20 });

  for await (const user of users) {
    console.log(`${user.id}: ${user.email} (${user.role})`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Users.List(new() { Limit = 20 });

  await foreach (var user in page.Paginate())
  {
      Console.WriteLine($"{user.ID}: {user.Email} ({user.Role.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  users := client.Organization.Users.ListAutoPaging(context.Background(), anthropic.OrganizationUserListParams{
  	Limit: anthropic.Int(20),
  })

  for users.Next() {
  	user := users.Current()
  	fmt.Printf("%s: %s (%s)\n", user.ID, user.Email, user.Role)
  }
  if err := users.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.users.UserListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserListParams.builder()
          .limit(20)
          .build();
      var users = client.organization().users().list(params);

      for (var user : users.autoPager()) {
          IO.println(user.id() + ": " + user.email() + " (" + user.role().asString() + ")");
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $users = $client->organization->users->list(limit: 20);

  foreach ($users->pagingEachItem() as $user) {
      echo "{$user->id}: {$user->email} ({$user->role})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  users = client.organization.users.list(limit: 20)

  users.auto_paging_each do |user|
    puts "#{user.id}: #{user.email} (#{user.role})"
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "type": "user",
      "id": "user_01AbCdEfGhIjKlMnOpQrSt",
      "email": "jane@example.com",
      "name": "Jane Smith",
      "role": "user",
      "added_at": "2026-06-12T09:14:03Z"
    }
  ],
  "has_more": false,
  "first_id": "user_01AbCdEfGhIjKlMnOpQrSt",
  "last_id": "user_01AbCdEfGhIjKlMnOpQrSt"
}
```

## 关键概念

### 组织角色

每个成员恰好拥有一个组织角色。读取操作会将成员的角色返回为以下五个值之一：

| 角色                 | 含义                        |
| ------------------ | ------------------------- |
| `user`             | 标准成员。                     |
| `managed`          | 其权限通过附加到其所在群组的自定义角色授予的成员。 |
| `owner`            | 组织所有者。                    |
| `membership_admin` | 可以管理组织成员的成员。              |
| `primary_owner`    | 组织的主所有者。有且仅有一个。           |

API 只能在创建邀请和更新角色时分配 `user` 和 `managed` 角色。分配 `managed` 时，除 `write:members` 外还需要 `write:rbac_groups` 作用域，因为受管成员的权限来自附加到其所在群组的自定义角色。管理角色（`owner`、`membership_admin` 和 `primary_owner`）在 claude.ai 组织设置中分配，持有这些角色的成员无法通过此 API 修改或移除。

### 成员和邀请

一个人通过接受邀请（或在已配置的情况下通过组织的单点登录）成为成员。创建邀请会发送一封邀请电子邮件；之后邀请的状态显示为 `pending`，直到收件人接受（`accepted`）或超过服务器分配的 `expires_at`（`expired`）。只有 `pending` 状态的邀请可以撤回。要更改待处理邀请的电子邮件地址或角色，请撤回该邀请并创建一个新邀请。

如果您组织的套餐从有限的已购席位池中分配成员，则待处理的邀请会占用一个席位。创建邀请端点不接受席位或层级参数：席位会自动从有可用席位的最低层级中分配。在没有空闲席位时创建邀请会失败并返回 400 错误，而不会购买席位。撤回邀请、让邀请过期或之后移除该成员都会将席位归还到池中。

### 群组和角色

群组将成员与自定义角色关联起来（即 “role-based access control”（基于角色的访问控制），也就是端点路径和作用域名称中的 `rbac`）。群组归您的父组织及其下的每个组织所有，而不是归某个单一组织所有。因此，群组作用域（`read:rbac_groups` 和 `write:rbac_groups`）需要适用于所有关联组织的密钥。每个群组都带有一个 `source_type`：在 claude.ai 中创建的群组为 `direct`，由您的身份提供商预配的群组为 `scim`。群组的 `role_ids` 字段列出了附加到该群组的自定义角色的 ID。请使用[自定义角色端点](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#custom-roles)将其解析为名称和权限。每个组织都有自己的角色目录，但群组在您的父组织范围内共享。因此，获取属于您父组织下其他组织的角色时，您的密钥会收到 404。当角色数据暂时不可用时，`role_ids` 字段为 `null`（而不是 `[]`），因此请重试，以区分降级读取与没有角色的群组。

## 版本控制

在每个请求中发送 `anthropic-version` 标头；有关可用版本，请参阅 [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)。

## 速率限制

Admin API 端点共享每个组织**每分钟 100 个请求**的限制；邀请创建则有其单独的限制，为**每小时 1,200 个请求**。超出限制的请求会返回 **429 Too Many Requests**。

## 分页

成员和邀请列表使用基于 ID 的分页：传入 `limit`（默认 20，最大 1,000），以及 `before_id` 或 `after_id` 中的至多一个，并使用每个响应的 `first_id` 和 `last_id` 字段进行翻页，直到 `has_more` 为 `false`。群组和自定义角色列表则使用**不透明游标**（opaque cursor）：在下一个请求中将响应的 `next_page` 值原样作为 `page` 参数传入，直到 `next_page` 为 `null`。

当您遍历结果时，SDK 会为您获取更多页面，因此 `limit` 设置的是页面大小，而不是总数。在 CLI 中，对于成员和邀请列表，`--limit` 设置结果总数。对于群组和角色列表，它设置页面大小。

## 错误响应

错误响应遵循 [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)中记录的标准格式。

## 成员

### 列出成员

`GET /v1/organizations/users` 返回组织的成员，最近添加的排在最前。按 `email` 筛选可查找特定成员；匹配不区分大小写，并且可以容忍同一地址的常见变体（例如，`jane+hiring@example.com` 匹配 `jane@example.com`）。需要 `read:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[列出用户](https://platform.claude.com/docs/zh-CN/api/organization/users/list)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/users?email=jane@example.com" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:users list --email jane@example.com
  ```

  ```python Python
  client = anthropic.Anthropic()

  users = client.organization.users.list(email="jane@example.com")

  for user in users:
      print(f"{user.id}: {user.email} ({user.role})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const users = await client.organization.users.list({ email: "jane@example.com" });

  for await (const user of users) {
    console.log(`${user.id}: ${user.email} (${user.role})`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Users.List(new() { Email = "jane@example.com" });

  await foreach (var user in page.Paginate())
  {
      Console.WriteLine($"{user.ID}: {user.Email} ({user.Role.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  users := client.Organization.Users.ListAutoPaging(context.Background(), anthropic.OrganizationUserListParams{
  	Email: anthropic.String("jane@example.com"),
  })

  for users.Next() {
  	user := users.Current()
  	fmt.Printf("%s: %s (%s)\n", user.ID, user.Email, user.Role)
  }
  if err := users.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.users.UserListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserListParams.builder()
          .email("jane@example.com")
          .build();
      var users = client.organization().users().list(params);

      for (var user : users.autoPager()) {
          IO.println(user.id() + ": " + user.email() + " (" + user.role().asString() + ")");
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $users = $client->organization->users->list(email: 'jane@example.com');

  foreach ($users->pagingEachItem() as $user) {
      echo "{$user->id}: {$user->email} ({$user->role})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  users = client.organization.users.list(email: "jane@example.com")

  users.auto_paging_each do |user|
    puts "#{user.id}: #{user.email} (#{user.role})"
  end
  ```
</CodeGroup>

### 获取成员

`GET /v1/organizations/users/{user_id}` 按 ID 返回一个成员。需要 `read:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[获取用户](https://platform.claude.com/docs/zh-CN/api/organization/users/retrieve)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/users/user_01AbCdEfGhIjKlMnOpQrSt" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:users retrieve --user-id user_01AbCdEfGhIjKlMnOpQrSt
  ```

  ```python Python
  client = anthropic.Anthropic()

  user = client.organization.users.retrieve("user_01AbCdEfGhIjKlMnOpQrSt")

  print(f"id: {user.id}")
  print(f"email: {user.email}")
  print(f"role: {user.role}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const user = await client.organization.users.retrieve("user_01AbCdEfGhIjKlMnOpQrSt");

  console.log(`id: ${user.id}`);
  console.log(`email: ${user.email}`);
  console.log(`role: ${user.role}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var user = await client.Organization.Users.Retrieve("user_01AbCdEfGhIjKlMnOpQrSt");

  Console.WriteLine($"id: {user.ID}");
  Console.WriteLine($"email: {user.Email}");
  Console.WriteLine($"role: {user.Role.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  user, err := client.Organization.Users.Get(context.Background(), "user_01AbCdEfGhIjKlMnOpQrSt")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", user.ID)
  fmt.Printf("email: %s\n", user.Email)
  fmt.Printf("role: %s\n", user.Role)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var user = client.organization().users()
      .retrieve("user_01AbCdEfGhIjKlMnOpQrSt");

  IO.println("id: " + user.id());
  IO.println("email: " + user.email());
  IO.println("role: " + user.role().asString());
  ```

  ```php PHP
  $client = new Client();

  $user = $client->organization->users->retrieve(
      userID: 'user_01AbCdEfGhIjKlMnOpQrSt',
  );

  echo "id: {$user->id}\n";
  echo "email: {$user->email}\n";
  echo "role: {$user->role}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user = client.organization.users.retrieve("user_01AbCdEfGhIjKlMnOpQrSt")

  puts "id: #{user.id}"
  puts "email: #{user.email}"
  puts "role: #{user.role}"
  ```
</CodeGroup>

### 更改成员的角色

`POST /v1/organizations/users/{user_id}` 将成员的角色设置为 `user` 或 `managed`。持有管理角色（`owner`、`membership_admin` 或 `primary_owner`）的成员无法通过此端点更改，也无法分配管理角色；这两种情况都会返回 400，并且需要在 claude.ai 组织设置中管理。如果您组织的身份提供商管理角色（高级 SSO 或高级 SCIM 预配），角色更新会返回 400。需要 `write:members` 作用域，将角色更改为 `managed` 时还需要 `write:rbac_groups`。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[更新用户](https://platform.claude.com/docs/zh-CN/api/organization/users/update)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/users/user_01AbCdEfGhIjKlMnOpQrSt" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{"role": "managed"}'
  ```

  ```bash CLI
  ant organization:users update \
    --user-id user_01AbCdEfGhIjKlMnOpQrSt \
    --role managed
  ```

  ```python Python
  client = anthropic.Anthropic()

  user = client.organization.users.update("user_01AbCdEfGhIjKlMnOpQrSt", role="managed")

  print(f"id: {user.id}")
  print(f"role: {user.role}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const user = await client.organization.users.update("user_01AbCdEfGhIjKlMnOpQrSt", {
    role: "managed"
  });

  console.log(`id: ${user.id}`);
  console.log(`role: ${user.role}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.Users;
  // ...

  AnthropicClient client = new();

  var user = await client.Organization.Users.Update(
      "user_01AbCdEfGhIjKlMnOpQrSt",
      new() { Role = Role.Managed }
  );

  Console.WriteLine($"id: {user.ID}");
  Console.WriteLine($"role: {user.Role.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  user, err := client.Organization.Users.Update(
  	context.Background(),
  	"user_01AbCdEfGhIjKlMnOpQrSt",
  	anthropic.OrganizationUserUpdateParams{
  		Role: anthropic.OrganizationUserUpdateParamsRoleManaged,
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", user.ID)
  fmt.Printf("role: %s\n", user.Role)
  ```

  ```java Java
  import com.anthropic.models.organization.users.UserUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserUpdateParams.builder()
          .role(UserUpdateParams.Role.MANAGED)
          .build();
      var user = client.organization().users()
          .update("user_01AbCdEfGhIjKlMnOpQrSt", params);

      IO.println("id: " + user.id());
      IO.println("role: " + user.role().asString());
  }
  ```

  ```php PHP
  use Anthropic\Organization\Users\UserUpdateParams\Role;
  // ...

  $client = new Client();

  $user = $client->organization->users->update(
      userID: 'user_01AbCdEfGhIjKlMnOpQrSt',
      role: Role::MANAGED,
  );

  echo "id: {$user->id}\n";
  echo "role: {$user->role}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user_id = "user_01AbCdEfGhIjKlMnOpQrSt"
  user = client.organization.users.update(user_id, role: :managed)

  puts "id: #{user.id}"
  puts "role: #{user.role}"
  ```
</CodeGroup>

### 移除成员

`DELETE /v1/organizations/users/{user_id}` 将成员从组织中移除，并将其占用的任何已购席位归还到组织的池中。持有管理角色的成员无法通过此端点移除；如果您的身份提供商管理成员资格（SCIM），移除操作会返回 400。需要 `write:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[移除用户](https://platform.claude.com/docs/zh-CN/api/organization/users/remove)。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/users/user_01AbCdEfGhIjKlMnOpQrSt" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:users remove --user-id user_01AbCdEfGhIjKlMnOpQrSt
  ```

  ```python Python
  client = anthropic.Anthropic()

  removed_user = client.organization.users.remove("user_01AbCdEfGhIjKlMnOpQrSt")

  print(f"id: {removed_user.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const removedUser = await client.organization.users.remove("user_01AbCdEfGhIjKlMnOpQrSt");

  console.log(`id: ${removedUser.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var removedUser = await client.Organization.Users.Remove("user_01AbCdEfGhIjKlMnOpQrSt");

  Console.WriteLine($"id: {removedUser.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  removedUser, err := client.Organization.Users.Remove(context.Background(), "user_01AbCdEfGhIjKlMnOpQrSt")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", removedUser.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var removedUser = client.organization().users()
      .remove("user_01AbCdEfGhIjKlMnOpQrSt");

  IO.println("id: " + removedUser.id());
  ```

  ```php PHP
  $client = new Client();

  $removedUser = $client->organization->users->remove(
      userID: 'user_01AbCdEfGhIjKlMnOpQrSt',
  );

  echo "id: {$removedUser->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user_id = "user_01AbCdEfGhIjKlMnOpQrSt"
  removed_user = client.organization.users.remove(user_id)

  puts "id: #{removed_user.id}"
  ```
</CodeGroup>

```json
{
  "type": "user_deleted",
  "id": "user_01AbCdEfGhIjKlMnOpQrSt"
}
```

### 在所有位置登出成员

`POST /v1/organizations/users/{user_id}/sign_out` 会在每个浏览器和设备上将成员登出 Claude，而不会将其从组织中移除。当成员的设备丢失或被盗、发生安全事件时，或者在您决定是否移除该成员之前调查其账户期间，请使用此端点。需要 `write:members` 作用域。

<Note>
  此端点目前面向 Claude Enterprise 组织提供 **beta** 版本。

  每个请求都必须包含 [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers) `anthropic-beta: admin-user-sign-out-2026-10-08`。缺少该标头的请求会返回 400，并附带一条指明该值的消息。SDK 和 CLI 会为您发送该标头。

  如果您使用的 SDK 版本或 `ant` CLI 不包含此端点，请使用此标头通过 HTTP 发送请求。
</Note>

登出会作用于该用户的整个 Claude 账户，包括其在其他组织中以及个人计划中的会话。该成员会保留其席位和角色，并且可以立即重新登录。若要阻止某人访问，请[移除该成员](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#remove-a-member)，或者如果您的组织要求单点登录，请在您的身份提供商中禁用其账户。禁用账户会阻止新的登录，但该用户仍会保留在您的成员列表中。如果您移除了某人但其身份提供商账户仍处于活动状态，自动预配可能会在其下次登录时将其重新添加。如果您的身份提供商管理成员资格，则只能在身份提供商中移除该成员。

请在需要之前提前创建密钥。请参阅[为 Claude Enterprise 组织创建密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization)。`write:members` 密钥仅在创建它的组织中有效，因此每个关联的组织都需要各自的密钥。该密钥由该组织的主要所有者创建。若要使用同一密钥按电子邮件地址查找成员，还需选择 `read:members` 作用域。

该请求没有请求体。请传入成员的 `id`（由[列出成员](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#list-members)返回）：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/users/user_01AbCdEfGhIjKlMnOpQrSt/sign_out" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: admin-user-sign-out-2026-10-08"
  ```

  ```bash CLI
  ant beta:organization:users sign-out --user-id user_01AbCdEfGhIjKlMnOpQrSt
  ```

  ```python Python
  client = anthropic.Anthropic()

  user = client.beta.organization.users.sign_out("user_01AbCdEfGhIjKlMnOpQrSt")

  print(f"id: {user.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const user = await client.beta.organization.users.signOut("user_01AbCdEfGhIjKlMnOpQrSt");

  console.log(`id: ${user.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var user = await client.Beta.Organization.Users.SignOut("user_01AbCdEfGhIjKlMnOpQrSt");

  Console.WriteLine($"id: {user.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  user, err := client.Beta.Organization.Users.SignOut(context.Background(), "user_01AbCdEfGhIjKlMnOpQrSt", anthropic.BetaOrganizationUserSignOutParams{})
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", user.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var user = client.beta().organization().users()
      .signOut("user_01AbCdEfGhIjKlMnOpQrSt");

  IO.println("id: " + user.id());
  ```

  ```php PHP
  $client = new Client();

  $user = $client->beta->organization->users->signOut(
      userID: 'user_01AbCdEfGhIjKlMnOpQrSt',
  );

  echo "id: {$user->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user = client.beta.organization.users.sign_out("user_01AbCdEfGhIjKlMnOpQrSt")

  puts "id: #{user.id}"
  ```
</CodeGroup>

成功的请求会返回 200 以及该成员信息，其角色和其他字段保持不变：

```json
{
  "type": "user",
  "id": "user_01AbCdEfGhIjKlMnOpQrSt",
  "email": "jane@example.com",
  "name": "Jane Smith",
  "role": "user",
  "added_at": "2026-06-12T09:14:03Z"
}
```

如果您还没有密钥，所有者可以改为在 [claude.ai 组织设置](https://claude.ai/admin-settings)的"成员"页面上使用**在所有设备上退出登录**操作。它会结束相同的会话、受信任设备和 OAuth 令牌，并移除相同的推送通知注册。所有者不能对自己的账户使用此操作：他们需要在 claude.ai 的账户设置中登出自己的会话。

#### 请求会结束的内容

该请求会为该成员结束以下所有内容：

* **登录会话**：该用户账户在每个浏览器和设备上的所有登录会话。
* **受信任设备**：该用户为敏感操作注册为受信任的每个浏览器、计算机或手机。该用户必须重新注册每台设备。
* **为此组织签发的 OAuth 令牌**：无论由哪个应用持有，例如 Claude Code 或第三方应用。这些应用必须重新登录。令牌几乎总是会立即失效，最长不超过一小时。

该请求还会清除该用户选择跳过 Claude 两步验证的浏览器记录，以便在这些浏览器上下次登录时再次要求第二步验证。它还会移除 Claude 为该用户的手机、桌面应用和浏览器保存的推送通知注册，使 Claude 停止向这些设备发送通知。此移除操作是异步执行的：通常在一分钟内完成，但在极少数情况下可能需要更长时间。返回 200 并不能确认这两个步骤已完成。

只有 OAuth 令牌仅限于此组织。会话、受信任设备和推送通知注册属于该用户的账户，因此登出也会作用于其所属的其他组织以及任何个人计划。

该请求不会删除已存储在设备上的任何内容。

#### 请求不会结束的内容

该请求不会结束以下任何内容：

* **API 密钥**：它们会继续有效，包括该用户创建的 Admin API 密钥。若要停用 Admin API 密钥，请在 [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access) 的**密钥**部分将其删除。
* **其他组织的 OAuth 令牌**：若要结束这些令牌，请在每个组织中使用该组织的密钥发送请求。该用户在每个组织中的 `id` 都相同。签发给该用户账户而非某个组织的 OAuth 令牌也会继续有效，且此请求无法结束它们。
* **Claude Code 云端会话和例程**：在 Anthropic 基础设施上运行的会话（例如网页版 Claude Code），或在您的某个[自托管环境](https://code.claude.com/docs/en/self-hosted-environments)中运行的会话会继续运行。在您发送请求时正在进行的工作，或在请求后不久开始的工作，可能会失败。随后该会话会在用户无需重新登录的情况下获得新凭据并继续运行。该用户按计划或根据 GitHub 事件运行的[例程](https://code.claude.com/docs/en/routines)会继续启动新会话。**在所有设备上退出登录**操作同样不会停止这些内容。
* **与 Cowork 关联的计算机**：它们会保持关联。
* **正在进行的登录**：已经在进行中的登录可能会在请求期间或请求后最多 5 分钟内完成。如果[审计日志导出](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api#export-audit-logs)或[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)显示该用户在请求之后登录，请再次发送请求。登录 Claude Code 等应用的操作不会记录在这些位置，因此如果该用户当时正在登录此类应用，请在 5 分钟后再次发送请求。

#### 无法登出的对象

该请求无法登出组织的主要所有者：它会返回 400，错误代码为 `cannot_sign_out_primary_owner`。您组织中的任何人都无法登出主要所有者，在"成员"页面上也不行。主要所有者需要在 claude.ai 的账户设置中登出自己的会话。

具有 `write:members` 作用域的密钥可以登出其他所有人。这包括所有者、其自定义角色包含管理所有者权限的成员，以及具有 `membership_admin` 角色的成员。所有者和主要所有者可以通过"成员"页面上的**在所有设备上退出登录**操作执行相同的操作。只有该成员在此组织中的角色才会被考虑，因此您可以登出在其他组织中担任主要所有者的人。

在创建密钥的人更改角色或离开组织后，该密钥仍会继续有效。请参阅[创建 Admin API 密钥的人离开后，该密钥会怎样？](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#what-happens-to-an-admin-api-key-when-the-person-who-created-it-leaves)。

#### 响应

| 状态                                      | 含义                                                                                                                                                |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200                                     | 登出已完成。可以安全地重试：没有剩余可结束内容的请求也会返回 200，第二次请求会结束该用户自第一次请求以来获得的任何会话、受信任设备或 OAuth 令牌。                                                                    |
| 400 `cannot_sign_out_primary_owner`     | 该成员是主要所有者，无法被登出。请参阅[无法登出的对象](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#who-you-cant-sign-out)。                         |
| 400 `organization_not_enrolled_in_beta` | 此端点对您的组织不可用。                                                                                                                                      |
| 400                                     | 请求缺少 `anthropic-beta` 值（消息中会指明该值），或者路径中的 `id` 不以 `user_` 开头。                                                                                      |
| 401                                     | 密钥缺失、无效或已删除，或者该密钥是为父组织创建的，未绑定到某个组织。                                                                                                               |
| 403                                     | 密钥缺少 `write:members` 作用域。                                                                                                                         |
| 404                                     | 您的组织中不存在具有该 ID 的成员。                                                                                                                               |
| 429                                     | 该组织超出了 Admin API 端点共享的[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#rate-limits)。`retry-after` 标头会指明何时重试。登出没有单独的限制。 |
| 503                                     | 请求所需的某项服务暂时不可用。请求已结束的内容会保持结束状态。请再次发送请求以结束其余内容。                                                                                                    |

错误响应遵循 [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)中记录的标准格式，错误代码位于 `error.details.error_code` 中：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "...",
    "details": {
      "error_code": "cannot_sign_out_primary_owner"
    }
  },
  "request_id": "req_018EeWyXxfu5pfWkrYcMdjWG"
}
```

#### 登出的记录位置

每次完成的登出都会记录为 `org_user_trusted_devices_revoked`，与**在所有设备上退出登录**操作记录的事件相同。它会出现在所有者从 claude.ai 下载的[审计日志导出](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api#export-audit-logs)中。如果您的组织拥有 Compliance API，它还会出现在[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)中，并附带该请求结束的受信任设备和会话的数量。中途停止的请求也会出现在活动源中，其 `completed` 设置为 `false`，但不会出现在审计日志中。

## 邀请

### 创建邀请

`POST /v1/organizations/invites` 发送一封邀请电子邮件，并返回带有服务器分配的 `expires_at` 的邀请。`role` 必须为 `user` 或 `managed`。如果该电子邮件地址已存在待处理的邀请，或该地址已属于某个成员，请求会返回 400 并指明现有资源。身份提供商自动预配用户（JIT 或 SCIM）的组织无法通过 API 创建邀请。需要 `write:members` 作用域，当 `role` 为 `managed` 时还需要 `write:rbac_groups`。

在从有限席位池中分配成员的套餐上，邀请会自动从有可用席位的最低层级中占用一个席位。API 不接受层级参数。如果没有空闲席位，请求会失败并返回 400 错误，而不会购买席位。请通过组织的套餐管理添加席位后重试。

可选的 `rbac_group_ids` 字段列出在成员接受邀请时要分配给该成员的群组（使用带 `rbac_group_` 前缀的 ID）。传递非空的 `rbac_group_ids` 还要求密钥带有 `write:rbac_groups` 作用域，因为群组分配可能会授予附加到群组角色的权限。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[创建邀请](https://platform.claude.com/docs/zh-CN/api/organization/invites/create)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/invites" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "email": "newhire@example.com",
      "role": "managed",
      "rbac_group_ids": ["rbac_group_01UvWxYzAbCdEfGhIjKlMn"]
    }'
  ```

  ```bash CLI
  ant organization:invites create \
    --email newhire@example.com \
    --role managed \
    --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn
  ```

  ```python Python
  client = anthropic.Anthropic()

  invite = client.organization.invites.create(
      email="newhire@example.com",
      role="managed",
      rbac_group_ids=["rbac_group_01UvWxYzAbCdEfGhIjKlMn"],
  )

  print(f"id: {invite.id}")
  print(f"email: {invite.email}")
  print(f"status: {invite.status}")
  print(f"expires_at: {invite.expires_at}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const invite = await client.organization.invites.create({
    email: "newhire@example.com",
    role: "managed",
    rbac_group_ids: ["rbac_group_01UvWxYzAbCdEfGhIjKlMn"]
  });

  console.log(`id: ${invite.id}`);
  console.log(`email: ${invite.email}`);
  console.log(`status: ${invite.status}`);
  console.log(`expires_at: ${invite.expires_at}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.Invites;
  // ...

  AnthropicClient client = new();

  var invite = await client.Organization.Invites.Create(new()
  {
      Email = "newhire@example.com",
      Role = Role.Managed,
      RbacGroupIds = ["rbac_group_01UvWxYzAbCdEfGhIjKlMn"]
  });

  Console.WriteLine($"id: {invite.ID}");
  Console.WriteLine($"email: {invite.Email}");
  Console.WriteLine($"status: {invite.Status.Raw()}");
  Console.WriteLine($"expires_at: {invite.ExpiresAt:O}");
  ```

  ```go Go
  client := anthropic.NewClient()

  invite, err := client.Organization.Invites.New(context.Background(), anthropic.OrganizationInviteNewParams{
  	Email:        "newhire@example.com",
  	Role:         anthropic.OrganizationInviteNewParamsRoleManaged,
  	RBACGroupIDs: []string{"rbac_group_01UvWxYzAbCdEfGhIjKlMn"},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", invite.ID)
  fmt.Printf("email: %s\n", invite.Email)
  fmt.Printf("status: %s\n", invite.Status)
  fmt.Printf("expires_at: %s\n", invite.ExpiresAt.Format(time.RFC3339))
  ```

  ```java Java
  import com.anthropic.models.organization.invites.InviteCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = InviteCreateParams.builder()
          .email("newhire@example.com")
          .role(InviteCreateParams.Role.MANAGED)
          .addRbacGroupId("rbac_group_01UvWxYzAbCdEfGhIjKlMn")
          .build();
      var invite = client.organization().invites().create(params);

      IO.println("id: " + invite.id());
      IO.println("email: " + invite.email());
      IO.println("status: " + invite.status().asString());
      IO.println("expires_at: " + invite.expiresAt());
  }
  ```

  ```php PHP
  use Anthropic\Organization\Invites\InviteCreateParams\Role;
  // ...

  $client = new Client();

  $invite = $client->organization->invites->create(
      email: 'newhire@example.com',
      role: Role::MANAGED,
      rbacGroupIDs: ['rbac_group_01UvWxYzAbCdEfGhIjKlMn'],
  );

  echo "id: {$invite->id}\n";
  echo "email: {$invite->email}\n";
  echo "status: {$invite->status}\n";
  echo "expires_at: {$invite->expiresAt->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  invite = client.organization.invites.create(
    email: "newhire@example.com",
    role: :managed,
    rbac_group_ids: ["rbac_group_01UvWxYzAbCdEfGhIjKlMn"]
  )

  puts "id: #{invite.id}"
  puts "email: #{invite.email}"
  puts "status: #{invite.status}"
  puts "expires_at: #{invite.expires_at.iso8601}"
  ```
</CodeGroup>

```json
{
  "type": "invite",
  "id": "invite_01QrStUvWxYzAbCdEfGhIj",
  "email": "newhire@example.com",
  "role": "managed",
  "invited_at": "2026-07-06T16:20:11Z",
  "expires_at": "2026-07-27T16:20:11Z",
  "accepted_at": null,
  "status": "pending",
  "rbac_group_ids": ["rbac_group_01UvWxYzAbCdEfGhIjKlMn"]
}
```

### 列出邀请

`GET /v1/organizations/invites` 返回组织的邀请，最新的排在最前，涵盖 `pending`、`accepted` 和 `expired` 状态。若只想列出处于特定状态的邀请，请为每个状态传递一次 `statuses[]`，例如 `statuses[]=pending`。需要 `read:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[列出邀请](https://platform.claude.com/docs/zh-CN/api/organization/invites/list)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/invites?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:invites list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  invites = client.organization.invites.list(limit=20)

  for invite in invites:
      print(f"{invite.id}: {invite.email} ({invite.status})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const invites = await client.organization.invites.list({ limit: 20 });

  for await (const invite of invites) {
    console.log(`${invite.id}: ${invite.email} (${invite.status})`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Invites.List(new() { Limit = 20 });

  await foreach (var invite in page.Paginate())
  {
      Console.WriteLine($"{invite.ID}: {invite.Email} ({invite.Status.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  invites := client.Organization.Invites.ListAutoPaging(context.Background(), anthropic.OrganizationInviteListParams{
  	Limit: anthropic.Int(20),
  })

  for invites.Next() {
  	invite := invites.Current()
  	fmt.Printf("%s: %s (%s)\n", invite.ID, invite.Email, invite.Status)
  }
  if err := invites.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.invites.InviteListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = InviteListParams.builder()
          .limit(20)
          .build();
      var invites = client.organization().invites().list(params);

      for (var invite : invites.autoPager()) {
          IO.println(invite.id() + ": " + invite.email() + " (" + invite.status().asString() + ")");
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $invites = $client->organization->invites->list(limit: 20);

  foreach ($invites->pagingEachItem() as $invite) {
      echo "{$invite->id}: {$invite->email} ({$invite->status})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  invites = client.organization.invites.list(limit: 20)

  invites.auto_paging_each do |invite|
    puts "#{invite.id}: #{invite.email} (#{invite.status})"
  end
  ```
</CodeGroup>

### 获取邀请

`GET /v1/organizations/invites/{invite_id}` 按 ID 返回一个邀请。需要 `read:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[获取邀请](https://platform.claude.com/docs/zh-CN/api/organization/invites/retrieve)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/invites/invite_01QrStUvWxYzAbCdEfGhIj" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:invites retrieve --invite-id invite_01QrStUvWxYzAbCdEfGhIj
  ```

  ```python Python
  client = anthropic.Anthropic()

  invite = client.organization.invites.retrieve("invite_01QrStUvWxYzAbCdEfGhIj")

  print(f"id: {invite.id}")
  print(f"email: {invite.email}")
  print(f"status: {invite.status}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const invite = await client.organization.invites.retrieve("invite_01QrStUvWxYzAbCdEfGhIj");

  console.log(`id: ${invite.id}`);
  console.log(`email: ${invite.email}`);
  console.log(`status: ${invite.status}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var invite = await client.Organization.Invites.Retrieve("invite_01QrStUvWxYzAbCdEfGhIj");

  Console.WriteLine($"id: {invite.ID}");
  Console.WriteLine($"email: {invite.Email}");
  Console.WriteLine($"status: {invite.Status.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  invite, err := client.Organization.Invites.Get(context.Background(), "invite_01QrStUvWxYzAbCdEfGhIj")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", invite.ID)
  fmt.Printf("email: %s\n", invite.Email)
  fmt.Printf("status: %s\n", invite.Status)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var invite = client.organization().invites()
      .retrieve("invite_01QrStUvWxYzAbCdEfGhIj");

  IO.println("id: " + invite.id());
  IO.println("email: " + invite.email());
  IO.println("status: " + invite.status().asString());
  ```

  ```php PHP
  $client = new Client();

  $invite = $client->organization->invites->retrieve(
      inviteID: 'invite_01QrStUvWxYzAbCdEfGhIj',
  );

  echo "id: {$invite->id}\n";
  echo "email: {$invite->email}\n";
  echo "status: {$invite->status}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  invite = client.organization.invites.retrieve("invite_01QrStUvWxYzAbCdEfGhIj")

  puts "id: #{invite.id}"
  puts "email: #{invite.email}"
  puts "status: #{invite.status}"
  ```
</CodeGroup>

### 撤回邀请

`DELETE /v1/organizations/invites/{invite_id}` 撤回一个 `pending` 状态的邀请，使邀请电子邮件中的链接失效。撤回 `accepted` 状态的邀请会返回 400（请改为移除该成员）；撤回 `expired` 状态的邀请会返回 400。需要 `write:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[删除邀请](https://platform.claude.com/docs/zh-CN/api/organization/invites/delete)。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/invites/invite_01QrStUvWxYzAbCdEfGhIj" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:invites delete --invite-id invite_01QrStUvWxYzAbCdEfGhIj
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted_invite = client.organization.invites.delete("invite_01QrStUvWxYzAbCdEfGhIj")

  print(f"id: {deleted_invite.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deletedInvite = await client.organization.invites.delete(
    "invite_01QrStUvWxYzAbCdEfGhIj"
  );

  console.log(`id: ${deletedInvite.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deletedInvite = await client.Organization.Invites.Delete("invite_01QrStUvWxYzAbCdEfGhIj");

  Console.WriteLine($"id: {deletedInvite.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  deletedInvite, err := client.Organization.Invites.Delete(context.Background(), "invite_01QrStUvWxYzAbCdEfGhIj")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", deletedInvite.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var deletedInvite = client.organization().invites()
      .delete("invite_01QrStUvWxYzAbCdEfGhIj");

  IO.println("id: " + deletedInvite.id());
  ```

  ```php PHP
  $client = new Client();

  $deletedInvite = $client->organization->invites->delete(
      inviteID: 'invite_01QrStUvWxYzAbCdEfGhIj',
  );

  echo "id: {$deletedInvite->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  invite_id = "invite_01QrStUvWxYzAbCdEfGhIj"
  deleted_invite = client.organization.invites.delete(invite_id)

  puts "id: #{deleted_invite.id}"
  ```
</CodeGroup>

## 群组

您的企业直接创建的群组（在 [claude.ai 组织设置](https://claude.ai/admin-settings)中或通过此 API 创建，`source_type: "direct"`）支持本节中的所有端点。由您的身份提供商预配的群组（`source_type: "scim"`）可以读取但不能修改：重命名或删除 SCIM 群组，或更改其成员资格，都会返回 400，因为该群组归您的身份提供商所有。

### 列出群组

`GET /v1/organizations/rbac_groups` 返回您企业的群组，包括由身份提供商管理的（`scim`）群组。需要 `read:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[列出群组](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/list)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/rbac_groups?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-groups list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  groups = client.beta.organization.rbac_groups.list(limit=20)

  for group in groups:
      print(f"{group.id}: {group.name} ({group.source_type})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const groups = await client.beta.organization.rbacGroups.list({ limit: 20 });

  for await (const group of groups) {
    console.log(`${group.id}: ${group.name} (${group.source_type})`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.RbacGroups.List(new() { Limit = 20 });

  await foreach (var group in page.Paginate())
  {
      Console.WriteLine($"{group.ID}: {group.Name} ({group.SourceType.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  groups := client.Beta.Organization.RBACGroups.ListAutoPaging(context.Background(), anthropic.BetaOrganizationRBACGroupListParams{
  	Limit: anthropic.Int(20),
  })

  for groups.Next() {
  	group := groups.Current()
  	fmt.Printf("%s: %s (%s)\n", group.ID, group.Name, group.SourceType)
  }
  if err := groups.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacgroups.RbacGroupListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RbacGroupListParams.builder()
          .limit(20)
          .build();
      var groups = client.beta().organization().rbacGroups().list(params);

      for (var group : groups.autoPager()) {
          IO.println(group.id() + ": " + group.name() + " (" + group.sourceType().asString() + ")");
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $groups = $client->beta->organization->rbacGroups->list(limit: 20);

  foreach ($groups->pagingEachItem() as $group) {
      echo "{$group->id}: {$group->name} ({$group->sourceType})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  groups = client.beta.organization.rbac_groups.list(limit: 20)

  groups.auto_paging_each do |group|
    puts "#{group.id}: #{group.name} (#{group.source_type})"
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "type": "rbac_group",
      "id": "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
      "name": "Engineering",
      "source_type": "direct",
      "role_ids": ["rbac_role_01CdEfGhIjKlMnOpQrStUv"],
      "created_at": "2026-03-18T10:01:42Z",
      "updated_at": "2026-05-02T08:55:09Z"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

### 获取群组

`GET /v1/organizations/rbac_groups/{rbac_group_id}` 按 ID 返回一个群组。需要 `read:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[获取群组](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/retrieve)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/rbac_groups/rbac_group_01UvWxYzAbCdEfGhIjKlMn" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-groups retrieve \
    --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn
  ```

  ```python Python
  client = anthropic.Anthropic()

  group = client.beta.organization.rbac_groups.retrieve(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  )

  print(f"id: {group.id}")
  print(f"name: {group.name}")
  print(f"source_type: {group.source_type}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const group = await client.beta.organization.rbacGroups.retrieve(
    "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  );

  console.log(`id: ${group.id}`);
  console.log(`name: ${group.name}`);
  console.log(`source_type: ${group.source_type}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var group = await client.Beta.Organization.RbacGroups.Retrieve("rbac_group_01UvWxYzAbCdEfGhIjKlMn");

  Console.WriteLine($"id: {group.ID}");
  Console.WriteLine($"name: {group.Name}");
  Console.WriteLine($"source_type: {group.SourceType.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  group, err := client.Beta.Organization.RBACGroups.Get(context.Background(), "rbac_group_01UvWxYzAbCdEfGhIjKlMn")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", group.ID)
  fmt.Printf("name: %s\n", group.Name)
  fmt.Printf("source_type: %s\n", group.SourceType)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var group = client.beta().organization().rbacGroups()
      .retrieve("rbac_group_01UvWxYzAbCdEfGhIjKlMn");

  IO.println("id: " + group.id());
  IO.println("name: " + group.name());
  IO.println("source_type: " + group.sourceType().asString());
  ```

  ```php PHP
  $client = new Client();

  $group = $client->beta->organization->rbacGroups->retrieve(
      rbacGroupID: 'rbac_group_01UvWxYzAbCdEfGhIjKlMn',
  );

  echo "id: {$group->id}\n";
  echo "name: {$group->name}\n";
  echo "source_type: {$group->sourceType}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_group_id = "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  group = client.beta.organization.rbac_groups.retrieve(rbac_group_id)

  puts "id: #{group.id}"
  puts "name: #{group.name}"
  puts "source_type: #{group.source_type}"
  ```
</CodeGroup>

### 创建群组

`POST /v1/organizations/rbac_groups` 使用给定的 `name`（1–255 个字符）创建一个没有角色或成员的群组。需要 `write:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[创建群组](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/create)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/rbac_groups" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{"name": "Engineering"}'
  ```

  ```bash CLI
  ant beta:organization:rbac-groups create --name Engineering
  ```

  ```python Python
  client = anthropic.Anthropic()

  group = client.beta.organization.rbac_groups.create(name="Engineering")

  print(f"id: {group.id}")
  print(f"name: {group.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const group = await client.beta.organization.rbacGroups.create({ name: "Engineering" });

  console.log(`id: ${group.id}`);
  console.log(`name: ${group.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var group = await client.Beta.Organization.RbacGroups.Create(new()
  {
      Name = "Engineering"
  });

  Console.WriteLine($"id: {group.ID}");
  Console.WriteLine($"name: {group.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  group, err := client.Beta.Organization.RBACGroups.New(context.Background(), anthropic.BetaOrganizationRBACGroupNewParams{
  	Name: "Engineering",
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", group.ID)
  fmt.Printf("name: %s\n", group.Name)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacgroups.RbacGroupCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RbacGroupCreateParams.builder()
          .name("Engineering")
          .build();
      var group = client.beta().organization().rbacGroups().create(params);

      IO.println("id: " + group.id());
      IO.println("name: " + group.name());
  }
  ```

  ```php PHP
  $client = new Client();

  $group = $client->beta->organization->rbacGroups->create(
      name: 'Engineering',
  );

  echo "id: {$group->id}\n";
  echo "name: {$group->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  group = client.beta.organization.rbac_groups.create(name: "Engineering")

  puts "id: #{group.id}"
  puts "name: #{group.name}"
  ```
</CodeGroup>

```json
{
  "type": "rbac_group",
  "id": "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  "name": "Engineering",
  "source_type": "direct",
  "role_ids": [],
  "created_at": "2026-07-09T18:00:00Z",
  "updated_at": "2026-07-09T18:00:00Z"
}
```

### 重命名群组

`POST /v1/organizations/rbac_groups/{rbac_group_id}` 更新群组。`name` 是此端点唯一可以更改的字段。需要 `write:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[更新群组](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/update)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/rbac_groups/rbac_group_01UvWxYzAbCdEfGhIjKlMn" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{"name": "Platform Engineering"}'
  ```

  ```bash CLI
  ant beta:organization:rbac-groups update \
    --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn \
    --name "Platform Engineering"
  ```

  ```python Python
  client = anthropic.Anthropic()

  group = client.beta.organization.rbac_groups.update(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn", name="Platform Engineering"
  )

  print(f"id: {group.id}")
  print(f"name: {group.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const group = await client.beta.organization.rbacGroups.update(
    "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
    { name: "Platform Engineering" }
  );

  console.log(`id: ${group.id}`);
  console.log(`name: ${group.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var group = await client.Beta.Organization.RbacGroups.Update(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
      new() { Name = "Platform Engineering" }
  );

  Console.WriteLine($"id: {group.ID}");
  Console.WriteLine($"name: {group.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  group, err := client.Beta.Organization.RBACGroups.Update(
  	context.Background(),
  	"rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  	anthropic.BetaOrganizationRBACGroupUpdateParams{
  		Name: anthropic.String("Platform Engineering"),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", group.ID)
  fmt.Printf("name: %s\n", group.Name)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacgroups.RbacGroupUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RbacGroupUpdateParams.builder()
          .name("Platform Engineering")
          .build();
      var group = client.beta().organization().rbacGroups()
          .update("rbac_group_01UvWxYzAbCdEfGhIjKlMn", params);

      IO.println("id: " + group.id());
      IO.println("name: " + group.name());
  }
  ```

  ```php PHP
  $client = new Client();

  $group = $client->beta->organization->rbacGroups->update(
      rbacGroupID: 'rbac_group_01UvWxYzAbCdEfGhIjKlMn',
      name: 'Platform Engineering',
  );

  echo "id: {$group->id}\n";
  echo "name: {$group->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_group_id = "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  group = client.beta.organization.rbac_groups.update(rbac_group_id, name: "Platform Engineering")

  puts "id: #{group.id}"
  puts "name: #{group.name}"
  ```
</CodeGroup>

### 删除群组

`DELETE /v1/organizations/rbac_groups/{rbac_group_id}` 删除群组。其成员仍然是各自组织的成员，但会失去该群组所附加角色的权限，并且群组[支出限额](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)（如果存在）将不再适用于他们。需要 `write:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[删除群组](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/delete)。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/rbac_groups/rbac_group_01UvWxYzAbCdEfGhIjKlMn" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-groups delete \
    --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted_group = client.beta.organization.rbac_groups.delete(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  )

  print(f"id: {deleted_group.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deletedGroup = await client.beta.organization.rbacGroups.delete(
    "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  );

  console.log(`id: ${deletedGroup.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deletedGroup = await client.Beta.Organization.RbacGroups.Delete(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  );

  Console.WriteLine($"id: {deletedGroup.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  deletedGroup, err := client.Beta.Organization.RBACGroups.Delete(context.Background(), "rbac_group_01UvWxYzAbCdEfGhIjKlMn")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", deletedGroup.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var deletedGroup = client.beta().organization().rbacGroups()
      .delete("rbac_group_01UvWxYzAbCdEfGhIjKlMn");

  IO.println("id: " + deletedGroup.id());
  ```

  ```php PHP
  $client = new Client();

  $deletedGroup = $client->beta->organization->rbacGroups->delete(
      rbacGroupID: 'rbac_group_01UvWxYzAbCdEfGhIjKlMn',
  );

  echo "id: {$deletedGroup->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_group_id = "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  deleted_group = client.beta.organization.rbac_groups.delete(rbac_group_id)

  puts "id: #{deleted_group.id}"
  ```
</CodeGroup>

```json
{
  "id": "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  "type": "rbac_group_deleted"
}
```

### 列出群组的成员

`GET /v1/organizations/rbac_groups/{rbac_group_id}/members` 返回群组的成员（每个成员都带有其 `user_id` 和电子邮件），最早的排在最前。只会返回您企业各组织的当前成员，因此在 `has_more` 为 `true` 时，一个页面包含的条目可能少于 `limit`。需要 `read:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[列出群组成员](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/members/list)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/rbac_groups/rbac_group_01UvWxYzAbCdEfGhIjKlMn/members?limit=100" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-groups:members list \
    --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn \
    --limit 100
  ```

  ```python Python
  client = anthropic.Anthropic()

  members = client.beta.organization.rbac_groups.members.list(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn", limit=100
  )

  for member in members:
      print(f"{member.user_id}: {member.email}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const members = await client.beta.organization.rbacGroups.members.list(
    "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
    { limit: 100 }
  );

  for await (const member of members) {
    console.log(`${member.user_id}: ${member.email}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.RbacGroups.Members.List(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
      new() { Limit = 100 }
  );

  await foreach (var member in page.Paginate())
  {
      Console.WriteLine($"{member.UserID}: {member.Email}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  members := client.Beta.Organization.RBACGroups.Members.ListAutoPaging(
  	context.Background(),
  	"rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  	anthropic.BetaOrganizationRBACGroupMemberListParams{
  		Limit: anthropic.Int(100),
  	},
  )

  for members.Next() {
  	member := members.Current()
  	fmt.Printf("%s: %s\n", member.UserID, member.Email)
  }
  if err := members.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacgroups.members.MemberListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MemberListParams.builder()
          .limit(100)
          .build();
      var members = client.beta().organization().rbacGroups().members()
          .list("rbac_group_01UvWxYzAbCdEfGhIjKlMn", params);

      for (var member : members.autoPager()) {
          IO.println(member.userId() + ": " + member.email());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $members = $client->beta->organization->rbacGroups->members->list(
      rbacGroupID: 'rbac_group_01UvWxYzAbCdEfGhIjKlMn',
      limit: 100,
  );

  foreach ($members->pagingEachItem() as $member) {
      echo "{$member->userID}: {$member->email}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_group_id = "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  members = client.beta.organization.rbac_groups.members.list(rbac_group_id, limit: 100)

  members.auto_paging_each do |member|
    puts "#{member.user_id}: #{member.email}"
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "type": "rbac_group_member",
      "rbac_group_id": "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
      "user_id": "user_01AbCdEfGhIjKlMnOpQrSt",
      "email": "jane@example.com",
      "created_at": "2026-04-07T12:30:00Z"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

### 向群组添加成员

`POST /v1/organizations/rbac_groups/{rbac_group_id}/members` 按 `user_id` 将组织成员添加到群组。该用户必须已经是您企业某个组织的成员（否则请求返回 404），添加已在群组中的人员会返回 400。对于 `scim` 群组，成员资格在您的身份提供商中管理，此请求会返回 400。要为尚未加入的人员分配群组，请改为在[创建邀请](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#create-an-invite)时使用 `rbac_group_ids`。需要 `write:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[添加群组成员](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/members/add)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/rbac_groups/rbac_group_01UvWxYzAbCdEfGhIjKlMn/members" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{"user_id": "user_01AbCdEfGhIjKlMnOpQrSt"}'
  ```

  ```bash CLI
  ant beta:organization:rbac-groups:members add \
    --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn \
    --user-id user_01AbCdEfGhIjKlMnOpQrSt
  ```

  ```python Python
  client = anthropic.Anthropic()

  member = client.beta.organization.rbac_groups.members.add(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn", user_id="user_01AbCdEfGhIjKlMnOpQrSt"
  )

  print(f"user_id: {member.user_id}")
  print(f"email: {member.email}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const member = await client.beta.organization.rbacGroups.members.add(
    "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
    { user_id: "user_01AbCdEfGhIjKlMnOpQrSt" }
  );

  console.log(`user_id: ${member.user_id}`);
  console.log(`email: ${member.email}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var member = await client.Beta.Organization.RbacGroups.Members.Add(
      "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
      new() { UserID = "user_01AbCdEfGhIjKlMnOpQrSt" }
  );

  Console.WriteLine($"user_id: {member.UserID}");
  Console.WriteLine($"email: {member.Email}");
  ```

  ```go Go
  client := anthropic.NewClient()

  member, err := client.Beta.Organization.RBACGroups.Members.Add(
  	context.Background(),
  	"rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  	anthropic.BetaOrganizationRBACGroupMemberAddParams{
  		UserID: "user_01AbCdEfGhIjKlMnOpQrSt",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("user_id: %s\n", member.UserID)
  fmt.Printf("email: %s\n", member.Email)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacgroups.members.MemberAddParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MemberAddParams.builder()
          .userId("user_01AbCdEfGhIjKlMnOpQrSt")
          .build();
      var member = client.beta().organization().rbacGroups().members()
          .add("rbac_group_01UvWxYzAbCdEfGhIjKlMn", params);

      IO.println("user_id: " + member.userId());
      IO.println("email: " + member.email());
  }
  ```

  ```php PHP
  $client = new Client();

  $member = $client->beta->organization->rbacGroups->members->add(
      rbacGroupID: 'rbac_group_01UvWxYzAbCdEfGhIjKlMn',
      userID: 'user_01AbCdEfGhIjKlMnOpQrSt',
  );

  echo "user_id: {$member->userID}\n";
  echo "email: {$member->email}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_group_id = "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  member = client.beta.organization.rbac_groups.members.add(
    rbac_group_id,
    user_id: "user_01AbCdEfGhIjKlMnOpQrSt"
  )

  puts "user_id: #{member.user_id}"
  puts "email: #{member.email}"
  ```
</CodeGroup>

```json
{
  "type": "rbac_group_member",
  "rbac_group_id": "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  "user_id": "user_01AbCdEfGhIjKlMnOpQrSt",
  "email": "jane@example.com",
  "created_at": "2026-07-09T18:00:00Z"
}
```

### 从群组中移除成员

`DELETE /v1/organizations/rbac_groups/{rbac_group_id}/members/{user_id}` 将成员从群组中移除；该成员仍然是其组织的成员。如果该用户不是群组成员，请求返回 404；对于 `scim` 群组（其成员资格在您的身份提供商中管理），请求返回 400。需要 `write:rbac_groups` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[移除群组成员](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/members/remove)。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/rbac_groups/rbac_group_01UvWxYzAbCdEfGhIjKlMn/members/user_01AbCdEfGhIjKlMnOpQrSt" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-groups:members remove \
    --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn \
    --user-id user_01AbCdEfGhIjKlMnOpQrSt
  ```

  ```python Python
  client = anthropic.Anthropic()

  removed_member = client.beta.organization.rbac_groups.members.remove(
      "user_01AbCdEfGhIjKlMnOpQrSt", rbac_group_id="rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  )

  print(f"user_id: {removed_member.user_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const removedMember = await client.beta.organization.rbacGroups.members.remove(
    "user_01AbCdEfGhIjKlMnOpQrSt",
    { rbac_group_id: "rbac_group_01UvWxYzAbCdEfGhIjKlMn" }
  );

  console.log(`user_id: ${removedMember.user_id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var removedMember = await client.Beta.Organization.RbacGroups.Members.Remove(
      "user_01AbCdEfGhIjKlMnOpQrSt",
      new() { RbacGroupID = "rbac_group_01UvWxYzAbCdEfGhIjKlMn" }
  );

  Console.WriteLine($"user_id: {removedMember.UserID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  removedMember, err := client.Beta.Organization.RBACGroups.Members.Remove(
  	context.Background(),
  	"user_01AbCdEfGhIjKlMnOpQrSt",
  	anthropic.BetaOrganizationRBACGroupMemberRemoveParams{
  		RBACGroupID: "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("user_id: %s\n", removedMember.UserID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacgroups.members.MemberRemoveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MemberRemoveParams.builder()
          .rbacGroupId("rbac_group_01UvWxYzAbCdEfGhIjKlMn")
          .build();
      var removedMember = client.beta().organization().rbacGroups().members()
          .remove("user_01AbCdEfGhIjKlMnOpQrSt", params);

      IO.println("user_id: " + removedMember.userId());
  }
  ```

  ```php PHP
  $client = new Client();

  $removedMember = $client->beta->organization->rbacGroups->members->remove(
      userID: 'user_01AbCdEfGhIjKlMnOpQrSt',
      rbacGroupID: 'rbac_group_01UvWxYzAbCdEfGhIjKlMn',
  );

  echo "user_id: {$removedMember->userID}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_group_id = "rbac_group_01UvWxYzAbCdEfGhIjKlMn"
  removed_member = client.beta.organization.rbac_groups.members.remove(
    "user_01AbCdEfGhIjKlMnOpQrSt",
    rbac_group_id: rbac_group_id
  )

  puts "user_id: #{removed_member.user_id}"
  ```
</CodeGroup>

```json
{
  "rbac_group_id": "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
  "user_id": "user_01AbCdEfGhIjKlMnOpQrSt",
  "type": "rbac_group_member_deleted"
}
```

## 自定义角色

自定义角色通过 API 是只读的：这些端点列出您组织的自定义角色（在 [claude.ai 组织设置](https://claude.ai/admin-settings)中定义或由 Anthropic 预配）以及每个角色授予的权限。自定义角色读取使用 `read:members` 作用域（没有单独的角色作用域），可以使用组织级密钥，并且返回的目录是您组织自己的目录。

从角色的 `display_name` 字段读取其名称。对于由 Anthropic 预配的角色，`display_name` 可能与 claude.ai 显示的标签不同，并且 Anthropic 可能会更改该名称。要持久引用某个角色，请存储其 `id`。

### 列出角色

`GET /v1/organizations/rbac_roles` 返回您组织的自定义角色。需要 `read:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[列出角色](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_roles/list)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/rbac_roles?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-roles list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  roles = client.beta.organization.rbac_roles.list(limit=20)

  for role in roles:
      print(f"{role.id}: {role.display_name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const roles = await client.beta.organization.rbacRoles.list({ limit: 20 });

  for await (const role of roles) {
    console.log(`${role.id}: ${role.display_name}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.RbacRoles.List(new() { Limit = 20 });

  await foreach (var role in page.Paginate())
  {
      Console.WriteLine($"{role.ID}: {role.DisplayName}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  roles := client.Beta.Organization.RBACRoles.ListAutoPaging(context.Background(), anthropic.BetaOrganizationRBACRoleListParams{
  	Limit: anthropic.Int(20),
  })

  for roles.Next() {
  	role := roles.Current()
  	fmt.Printf("%s: %s\n", role.ID, role.DisplayName)
  }
  if err := roles.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacroles.RbacRoleListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RbacRoleListParams.builder()
          .limit(20)
          .build();
      var roles = client.beta().organization().rbacRoles().list(params);

      for (var role : roles.autoPager()) {
          IO.println(role.id() + ": " + role.displayName());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $roles = $client->beta->organization->rbacRoles->list(limit: 20);

  foreach ($roles->pagingEachItem() as $role) {
      echo "{$role->id}: {$role->displayName}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  roles = client.beta.organization.rbac_roles.list(limit: 20)

  roles.auto_paging_each do |role|
    puts "#{role.id}: #{role.display_name}"
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "type": "rbac_role",
      "id": "rbac_role_01CdEfGhIjKlMnOpQrStUv",
      "display_name": "Engineering base",
      "created_at": "2026-03-18T10:01:42Z",
      "updated_at": "2026-05-02T08:55:09Z"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

### 获取角色

`GET /v1/organizations/rbac_roles/{rbac_role_id}` 按 ID 返回一个角色。需要 `read:members` 作用域。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[获取角色](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_roles/retrieve)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/rbac_roles/rbac_role_01CdEfGhIjKlMnOpQrStUv" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-roles retrieve \
    --rbac-role-id rbac_role_01CdEfGhIjKlMnOpQrStUv
  ```

  ```python Python
  client = anthropic.Anthropic()

  role = client.beta.organization.rbac_roles.retrieve("rbac_role_01CdEfGhIjKlMnOpQrStUv")

  print(f"id: {role.id}")
  print(f"display_name: {role.display_name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const role = await client.beta.organization.rbacRoles.retrieve(
    "rbac_role_01CdEfGhIjKlMnOpQrStUv"
  );

  console.log(`id: ${role.id}`);
  console.log(`display_name: ${role.display_name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var role = await client.Beta.Organization.RbacRoles.Retrieve("rbac_role_01CdEfGhIjKlMnOpQrStUv");

  Console.WriteLine($"id: {role.ID}");
  Console.WriteLine($"display_name: {role.DisplayName}");
  ```

  ```go Go
  client := anthropic.NewClient()

  role, err := client.Beta.Organization.RBACRoles.Get(context.Background(), "rbac_role_01CdEfGhIjKlMnOpQrStUv")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", role.ID)
  fmt.Printf("display_name: %s\n", role.DisplayName)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var role = client.beta().organization().rbacRoles()
      .retrieve("rbac_role_01CdEfGhIjKlMnOpQrStUv");

  IO.println("id: " + role.id());
  IO.println("display_name: " + role.displayName());
  ```

  ```php PHP
  $client = new Client();

  $role = $client->beta->organization->rbacRoles->retrieve(
      rbacRoleID: 'rbac_role_01CdEfGhIjKlMnOpQrStUv',
  );

  echo "id: {$role->id}\n";
  echo "display_name: {$role->displayName}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_role_id = "rbac_role_01CdEfGhIjKlMnOpQrStUv"
  role = client.beta.organization.rbac_roles.retrieve(rbac_role_id)

  puts "id: #{role.id}"
  puts "display_name: #{role.display_name}"
  ```
</CodeGroup>

### 列出角色的权限

`GET /v1/organizations/rbac_roles/{rbac_role_id}/permissions` 返回角色的权限。每个权限将一个 `resource`（其适用对象：组织的产品功能、某个连接器工具、某个连接器 OAuth 作用域、单个连接器或所有连接器）与一个 `action`（其在该资源上授予的内容）配对。未为您的组织启用的功能所对应的行会被省略，因此在 `has_more` 为 `true` 时，一个页面包含的行可能少于 `limit`。需要 `read:members` 作用域。

有两个 `action` 值需要特别注意。action 为 `capability_access_all`（所有产品功能）或 `capability_access_all_ga`（所有稳定产品功能，即所有未标记为 beta 或研究预览的功能）的 `organization` 权限是一种全面授权。它以单独一行列出，而不会展开。全面授权既不涵盖模型访问，也不涵盖带 `permission_` 前缀的管理面板权限，也不涵盖 `inference_hooks_inspection`，角色只能通过明确指定它的行来持有该权限。在统计某个角色授予的内容时，请将全面授权行视为涵盖其变体所描述的所有内容，而不仅仅是其他行中列出的功能。

有关完整的参数详细信息和响应架构，请参阅 API 参考中的[列出角色权限](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_roles/permissions/list)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/rbac_roles/rbac_role_01CdEfGhIjKlMnOpQrStUv/permissions?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:rbac-roles:permissions list \
    --rbac-role-id rbac_role_01CdEfGhIjKlMnOpQrStUv \
    --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  permissions = client.beta.organization.rbac_roles.permissions.list(
      "rbac_role_01CdEfGhIjKlMnOpQrStUv", limit=20
  )

  for permission in permissions:
      print(permission.action)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const permissions = await client.beta.organization.rbacRoles.permissions.list(
    "rbac_role_01CdEfGhIjKlMnOpQrStUv",
    { limit: 20 }
  );

  for await (const permission of permissions) {
    console.log(permission.action);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.RbacRoles.Permissions.List(
      "rbac_role_01CdEfGhIjKlMnOpQrStUv",
      new() { Limit = 20 }
  );

  await foreach (var permission in page.Paginate())
  {
      Console.WriteLine(permission.Action);
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  permissions := client.Beta.Organization.RBACRoles.Permissions.ListAutoPaging(
  	context.Background(),
  	"rbac_role_01CdEfGhIjKlMnOpQrStUv",
  	anthropic.BetaOrganizationRBACRolePermissionListParams{
  		Limit: anthropic.Int(20),
  	},
  )

  for permissions.Next() {
  	permission := permissions.Current()
  	fmt.Println(permission.Action)
  }
  if err := permissions.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.rbacroles.permissions.PermissionListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = PermissionListParams.builder()
          .limit(20)
          .build();
      var permissions = client.beta().organization().rbacRoles().permissions()
          .list("rbac_role_01CdEfGhIjKlMnOpQrStUv", params);

      for (var permission : permissions.autoPager()) {
          IO.println(permission.action());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $permissions = $client->beta->organization->rbacRoles->permissions->list(
      rbacRoleID: 'rbac_role_01CdEfGhIjKlMnOpQrStUv',
      limit: 20,
  );

  foreach ($permissions->pagingEachItem() as $permission) {
      echo "{$permission->action}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rbac_role_id = "rbac_role_01CdEfGhIjKlMnOpQrStUv"
  permissions = client.beta.organization.rbac_roles.permissions.list(rbac_role_id, limit: 20)

  permissions.auto_paging_each do |permission|
    puts permission.action
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "type": "rbac_role_permission",
      "resource": {
        "type": "organization",
        "organization_id": "12345678-1234-5678-1234-567812345678"
      },
      "action": "capability_access_all_ga"
    },
    {
      "type": "rbac_role_permission",
      "resource": {
        "type": "connector_tool",
        "connector_id": "mcpsrv_01WxYzAbCdEfGhIjKlMnOp",
        "tool_name": "search_tickets"
      },
      "action": "use"
    }
  ],
  "has_more": false,
  "next_page": null
}
```

## 示例工作流

### 为离职员工办理离职

1. 按电子邮件查找成员：

   <CodeGroup>
     ```bash cURL
     curl "https://api.anthropic.com/v1/organizations/users?email=departing@example.com" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21"
     ```

     ```bash CLI
     ant organization:users list --email departing@example.com
     ```

     ```python Python
     client = anthropic.Anthropic()

     users = client.organization.users.list(email="departing@example.com")

     for user in users:
         print(f"{user.id}: {user.email} ({user.role})")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const users = await client.organization.users.list({ email: "departing@example.com" });

     for await (const user of users) {
       console.log(`${user.id}: ${user.email} (${user.role})`);
     }
     ```

     ```csharp C#
     AnthropicClient client = new();

     var page = await client.Organization.Users.List(new() { Email = "departing@example.com" });

     await foreach (var user in page.Paginate())
     {
         Console.WriteLine($"{user.ID}: {user.Email} ({user.Role.Raw()})");
     }
     ```

     ```go Go
     client := anthropic.NewClient()

     users := client.Organization.Users.ListAutoPaging(context.Background(), anthropic.OrganizationUserListParams{
     	Email: anthropic.String("departing@example.com"),
     })

     for users.Next() {
     	user := users.Current()
     	fmt.Printf("%s: %s (%s)\n", user.ID, user.Email, user.Role)
     }
     if err := users.Err(); err != nil {
     	log.Fatal(err)
     }
     ```

     ```java Java
     import com.anthropic.models.organization.users.UserListParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = UserListParams.builder()
             .email("departing@example.com")
             .build();
         var users = client.organization().users().list(params);

         for (var user : users.autoPager()) {
             IO.println(user.id() + ": " + user.email() + " (" + user.role().asString() + ")");
         }
     }
     ```

     ```php PHP
     $client = new Client();

     $users = $client->organization->users->list(email: 'departing@example.com');

     foreach ($users->pagingEachItem() as $user) {
         echo "{$user->id}: {$user->email} ({$user->role})\n";
     }
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     users = client.organization.users.list(email: "departing@example.com")

     users.auto_paging_each do |user|
       puts "#{user.id}: #{user.email} (#{user.role})"
     end
     ```
   </CodeGroup>

2. 使用响应中的 `id`，通过 `DELETE /v1/organizations/users/{user_id}` 移除该成员。其席位（如果有）会归还到池中。

3. 如果此人尚未加入，查找将不会返回任何成员；请改为列出邀请并撤回其 `pending` 状态的邀请。

### 应对设备丢失或被盗

在需要此工作流之前，请为[登出端点](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#sign-a-member-out-everywhere)创建一个具有 `write:members` 和 `read:members` 作用域的密钥。如果您还没有密钥，所有者可以在 [claude.ai 组织设置](https://claude.ai/admin-settings)的"成员"页面上使用**在所有设备上退出登录**操作来代替第 2 至 4 步。所有者不能对自己的账户使用此操作：他们需要在 claude.ai 的账户设置中登出自己的会话。

1. 首先，将该用户从您的身份提供商中登出，或在其中禁用其账户。否则，持有丢失设备的人可以通过该设备的身份提供商会话或该用户的邮箱再次登录 Claude。

2. 按电子邮件地址查找该成员，方法与[为离职员工办理离职](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#offboard-a-departing-employee)的第 1 步相同。电子邮件匹配可以容忍同一地址的变体，因此请检查响应是否为正确的人。

3. 使用响应中的 `id`，通过 `POST /v1/organizations/users/{user_id}/sign_out` 将其登出。[在所有位置登出成员](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#sign-a-member-out-everywhere)列出了该请求所需的标头以及它会结束的内容。

4. 如果请求返回 503，请再次发送，直到返回 200。如果返回 400 并带有 `cannot_sign_out_primary_owner`，请参阅[无法登出的对象](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#who-you-cant-sign-out)。

5. 检查[审计日志导出](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api#export-audit-logs)或[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)，查看第一次请求之后是否有登录。如果发现有登录，或者该用户当时正在登录 Claude Code 等应用，请在 5 分钟后再次发送请求。如果您无法发送请求，请改为再次使用**在所有设备上退出登录**操作。这也会将该成员从其此后启动的任何会话中登出，因此请事先告知他们。

6. 在[登出的记录位置](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#where-the-sign-out-is-recorded)确认登出，并查看[请求不会结束的内容](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#what-the-request-doesnt-end)。

### 审计群组成员资格

1. 列出群组，并记录每个群组的 `id`、`name` 和 `role_ids`。

2. 对于每个带有敏感角色的群组，翻页浏览 `GET /v1/organizations/rbac_groups/{rbac_group_id}/members`，并将成员电子邮件与您身份提供商的名册进行比较。

3. 使用 `DELETE /v1/organizations/rbac_groups/{rbac_group_id}/members/{user_id}` 移除不应再留在群组中的成员。对于 `scim` 群组，请改为在您的身份提供商中进行更改。

有关将群组成员资格与临时提高支出限额相结合的工作流，请参阅支出限额 API 页面上的[在事件期间临时提高成员的支出限额](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api#temporarily-raise-a-members-spend-limit-during-an-incident)。

## 常见问题

### 这是与 Admin API 不同的 API 吗？

不是。成员和邀请端点与 Claude Console 组织使用的 `/v1/organizations/` 端点相同；本页记录它们在 Claude Enterprise 中的行为。群组和自定义角色端点属于同一个 API，并且仅适用于 Claude Enterprise 组织。[可用性表](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#which-endpoints-can-your-organization-use)显示了每种组织类型可以调用哪些端点。

### API 可以分配所有者或成员管理员角色吗？

不可以。API 仅在创建邀请和更新角色时分配 `user` 和 `managed`。管理角色在 claude.ai 组织设置中分配，持有这些角色的成员无法通过 API 修改或移除。

### API 可以创建或修改群组吗？

可以，使用 `write:rbac_groups` 作用域即可：创建、重命名和删除群组，以及添加或移除其成员。有两类内容无法通过 API 更改：由您的身份提供商预配的群组（`source_type: "scim"`），其名称和成员归身份提供商所有；以及[自定义角色](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#custom-roles)，它们在 claude.ai 组织设置中管理（API 只读取它们）。

### 未接受的邀请会占用席位吗？

在具有有限席位池的套餐上，会：`pending` 状态的邀请会占用一个席位。撤回邀请或让其过期会释放该席位。在没有席位池的套餐上，邀请不会占用任何资源。

### 当您的组织使用单点登录时，哪些操作可用？

如果您的身份提供商自动预配用户（JIT 或 SCIM），创建邀请会返回 400。如果它管理角色（高级 SSO 或高级 SCIM 预配），角色更新会返回 400。如果它管理成员资格（SCIM 预配），移除成员会返回 400。读取操作在任何情况下都可用。[将成员从所有位置登出](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#sign-a-member-out-everywhere)也是如此，无论您的组织使用哪种单点登录模式。

### 登出成员能阻止其访问吗？

不能。[登出成员](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#sign-a-member-out-everywhere)会结束其登录会话、受信任设备以及您组织的 OAuth 令牌，但他们可以立即重新登录。若要阻止某人访问，请[移除该成员](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#remove-a-member)，或者如果您的组织要求单点登录，请在您的身份提供商中禁用其账户。仅禁用账户可能无法结束已经打开的会话，因此请先将该用户从您的身份提供商中登出，然后将其登出 Claude，再移除该成员或禁用其账户。

### 当创建 Admin API 密钥的人离职时，该密钥会怎样？

该密钥会继续有效。Admin API 密钥的作用范围是组织，而不是个人用户，并且在 claude.ai 中创建的密钥不会过期。将创建者从组织中移除或通过您的身份提供商取消其预配，会终止其本人的访问权限，但不会影响其创建的密钥。降低其角色也不会改变这些密钥：每个密钥都会保持有效并保留其原始作用域。当您为创建过 Admin API 密钥的人员办理离职时，请在 [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access) 的**密钥**部分删除这些密钥并创建替代密钥。

## 另请参阅

<CardGroup cols={2}>
  <Card title="创建 Admin API 密钥" href="https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys">
    在何处创建带作用域的密钥以及应选择哪些作用域。
  </Card>

  <Card title="Compliance API" href="https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api">
    审计活动，并检索或删除整个组织中的用户内容。
  </Card>

  <Card title="用量和成本分析 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api">
    跟踪您组织的令牌用量和成本。
  </Card>

  <Card title="支出限额 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api">
    设置每个成员的支出限额并审核提额请求。
  </Card>
</CardGroup>
