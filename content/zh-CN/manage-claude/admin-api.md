---
title: Admin API
url: https://platform.claude.com/docs/zh-CN/manage-claude/admin-api
description: 使用 Admin API 以编程方式管理组织成员、工作区、邀请和 API 密钥，可使用 Admin API 密钥、`org:admin` OAuth 令牌，或个人密钥或服务账户密钥进行身份验证。
---

[Admin API](https://platform.claude.com/docs/zh-CN/api/organization) 让您能够以编程方式管理组织的成员、工作区、邀请和 API 密钥，而无需在 [Claude Console](https://platform.claude.com/) 中手动操作。

## 可用性

Admin API 适用于 Claude Console 和 Claude Enterprise 组织，不适用于 claude.ai Team 计划下的组织或个人 Console 账户。个人 Console 账户必须先在 Claude Console 的[组织](https://platform.claude.com/settings/organization)页面（`platform.claude.com/settings/organization`）中设置组织。有关每种组织类型在何处创建其密钥，请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。

Claude Enterprise（claude.ai）组织使用在 claude.ai 中创建的限定作用域 API 密钥调用 Admin API。在本页中，只有成员和邀请端点适用于它们。它们还可以调用[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)，该 API 目前处于 beta 阶段，需要使用具有 `read:analytics` 作用域的密钥。它们还可使用 Enterprise 专属端点：群组和自定义角色读取，以及[支出限额](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api)。同样为 Enterprise 专属的[插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api) 目前处于 beta 阶段。[Claude Tag API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api) 也是如此。请参阅[用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)。

在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上，仅以下 Admin API 端点可用：

* 工作区端点：`/v1/organizations/workspaces` 上的创建、获取、列出、更新和归档。
* 用于 [CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws) 的外部密钥端点：`/v1/organizations/external_keys` 上的注册、获取、列出、更新和删除。没有验证端点，因为密钥在附加到工作区时会进行验证。
* [遥测目标端点](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#telemetry-destinations)。
* [用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 端点：`/v1/organizations/analytics/usage` 和 `/v1/organizations/analytics/cost`。

组织成员、工作区成员、邀请、API 密钥、旧版[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)以及速率限制报告在 Claude Platform on AWS 上均不可用。

## 身份验证

Admin API 接受以下三种凭证中的任意一种：

* **Admin API 密钥**（以 `sk-ant-admin...` 开头），通过 `x-api-key` 标头发送。只有具有 admin 角色的组织成员才能配置此类密钥。请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。
* 具有 `org:admin` 作用域的 **OAuth bearer 令牌**，通过 `authorization: Bearer` 标头发送。只有具有管理员、所有者或主要所有者角色的成员才能获取此类令牌。请参阅[获取 OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。
* 未限定于特定工作区的**个人密钥**或**服务账户密钥**，通过 `x-api-key` 标头发送。该密钥拥有与所关联账户相同的权限。请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)。

Admin API 密钥涵盖大多数端点。服务账户、联合身份颁发者和联合规则端点仅接受 `org:admin` OAuth 令牌。个人密钥或服务账户密钥与 Admin API 密钥一样，通过 `x-api-key` 标头发送。以下示例分别使用 OAuth 令牌和 Admin API 密钥调用[组织信息端点](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#accessing-organization-info)。

SDK 在 `client.organization` (csharp, go: `client.Organization`; java: `client.organization()`; php: `$client->organization`) 下公开本页中的端点，`ant` CLI 则在 `ant organization` 下公开这些端点。本页中的示例使用默认客户端，该客户端从 `ANTHROPIC_API_KEY` 读取 Admin API 密钥，或从 `ANTHROPIC_AUTH_TOKEN` 读取 OAuth bearer 令牌。每个列表示例中的注释会说明它是获取更多页面，还是仅读取第一页。有关每个端点的参数和响应，请参阅 [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/organization)。

### OAuth bearer 令牌

使用 [`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart) 在具有 `org:admin` 作用域的专用配置文件下登录（请参阅[管理员访问](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication#admin-access)），然后导出 bearer 令牌。`--profile admin` 会将 `org:admin` 凭证存储在其自己的配置文件下，并将其设为 CLI 的活动配置文件。导出的变量适用于该 shell 中的每个 SDK 和 CLI 调用。请使用专门用于管理的 shell，完成后取消设置该变量，并使用 `ant profile activate default` 将 CLI 切换回去：

```bash CLI
ant auth login --profile admin --scope "org:admin"
export ANTHROPIC_AUTH_TOKEN=$(ant auth print-credentials --profile admin --access-token)
```

交互式令牌的有效期很短。如果请求开始返回 401，请重新运行 `export` 命令以刷新令牌。

SDK 和 `ant` CLI 会自动读取 `ANTHROPIC_AUTH_TOKEN`。请在同一 shell 中保持 `ANTHROPIC_API_KEY` 未设置，以便它们发送 bearer 令牌。自动化工作负载无需登录：它们通过工作负载身份联合进行身份验证，SDK 和 CLI 会根据联合环境变量执行令牌交换。请参阅[引导工作负载以管理 WIF](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#bootstrap-a-workload-to-manage-wif)。

使用导出的令牌调用 Admin API：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/me" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization retrieve
  ```

  ```python Python
  client = anthropic.Anthropic()

  organization = client.organization.retrieve()

  print(f"id: {organization.id}")
  print(f"name: {organization.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const organization = await client.organization.retrieve();

  console.log(`id: ${organization.id}`);
  console.log(`name: ${organization.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var organization = await client.Organization.Retrieve();

  Console.WriteLine($"id: {organization.ID}");
  Console.WriteLine($"name: {organization.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  organization, err := client.Organization.Get(context.Background())
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", organization.ID)
  fmt.Printf("name: %s\n", organization.Name)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var organization = client.organization().retrieve();

  IO.println("id: " + organization.id());
  IO.println("name: " + organization.name());
  ```

  ```php PHP
  $client = new Client();

  $organization = $client->organization->retrieve();

  echo "id: {$organization->id}\n";
  echo "name: {$organization->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  organization = client.organization.retrieve

  puts "id: #{organization.id}"
  puts "name: #{organization.name}"
  ```
</CodeGroup>

`org:admin` 令牌授予对整个组织的访问权限，无论底层配置文件或[联合规则](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#federation-rules)绑定到哪个工作区。

对于 CI 和其他非交互式工作负载，请使用工作负载身份联合（Workload Identity Federation）签发令牌，而不是交互式登录。请参阅[使用 Admin API 管理 WIF](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#workload-ci-and-automation)。

### Admin API 密钥

要为您的组织类型创建 Admin API 密钥，请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/me" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization retrieve
  ```

  ```python Python
  client = anthropic.Anthropic()

  organization = client.organization.retrieve()

  print(f"id: {organization.id}")
  print(f"name: {organization.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const organization = await client.organization.retrieve();

  console.log(`id: ${organization.id}`);
  console.log(`name: ${organization.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var organization = await client.Organization.Retrieve();

  Console.WriteLine($"id: {organization.ID}");
  Console.WriteLine($"name: {organization.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  organization, err := client.Organization.Get(context.Background())
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", organization.ID)
  fmt.Printf("name: %s\n", organization.Name)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var organization = client.organization().retrieve();

  IO.println("id: " + organization.id());
  IO.println("name: " + organization.name());
  ```

  ```php PHP
  $client = new Client();

  $organization = $client->organization->retrieve();

  echo "id: {$organization->id}\n";
  echo "name: {$organization->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  organization = client.organization.retrieve

  puts "id: #{organization.id}"
  puts "name: #{organization.name}"
  ```
</CodeGroup>

## Admin API 的工作原理

使用[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)中的任意凭证进行身份验证，然后管理以下资源：

* 组织成员及其角色
* 组织邀请
* 工作区及其成员
* API 密钥
* 服务账户、联合身份颁发者和联合规则（仅限 `org:admin` OAuth 令牌）

常见用途包括自动化入职和离职流程、管理工作区访问权限以及审计 API 密钥。

## 组织角色和权限

共有五种组织级角色。详情请参阅 [API Console 角色和权限](https://support.claude.com/en/articles/10186004-api-console-roles-and-permissions)。

| 角色                 | 权限                                                                           |
| ------------------ | ---------------------------------------------------------------------------- |
| user               | 可以使用 playground                                                              |
| claude\_code\_user | 可以使用 playground 和 [Claude Code](https://code.claude.com/docs/zh-CN/overview) |
| developer          | 可以使用 playground 并管理 API 密钥                                                   |
| billing            | 可以使用 playground 并管理账单详情                                                      |
| admin              | 可以执行上述所有操作，并可管理用户                                                            |

组织所有者和主要所有者拥有所有管理员权限，并且还可以对管理员进行管理。本页中所有对管理员角色的引用同样适用于所有者和主要所有者。

## 关键概念

### 组织成员

列出[组织成员](https://platform.claude.com/docs/zh-CN/api/organization/users/retrieve)、更新其角色以及移除成员。

列出您组织的成员：

<CodeGroup>
  ```bash cURL
  # 返回一页。
  curl "https://api.anthropic.com/v1/organizations/users?limit=10" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 在终端中，视图会随您滚动而获取更多页面。通过管道输出或使用 `--format json` 时，`--limit` 限制结果数量。
  ant organization:users list --limit 10
  ```

  ```python Python
  client = anthropic.Anthropic()

  users = client.organization.users.list(limit=10)

  # 根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  for user in users:
      print(f"{user.id}: {user.email} ({user.role})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const users = await client.organization.users.list({ limit: 10 });

  // 会根据需要自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
  for await (const user of users) {
    console.log(`${user.id}: ${user.email} (${user.role})`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Users.List(new() { Limit = 10 });

  // 会根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  await foreach (var user in page.Paginate())
  {
      Console.WriteLine($"{user.ID}: {user.Email} ({user.Role.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  users := client.Organization.Users.ListAutoPaging(context.Background(), anthropic.OrganizationUserListParams{
  	Limit: anthropic.Int(10),
  })

  // 会按需自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
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
          .limit(10)
          .build();
      var users = client.organization().users().list(params);

      // 根据需要自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
      for (var user : users.autoPager()) {
          IO.println(user.id() + ": " + user.email() + " (" + user.role().asString() + ")");
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $users = $client->organization->users->list(limit: 10);

  // 仅读取第一页。
  foreach ($users->getItems() as $user) {
      echo "{$user->id}: {$user->email} ({$user->role})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  users = client.organization.users.list(limit: 10)

  # 仅读取第一页。
  users.data.each do |user|
    puts "#{user.id}: #{user.email} (#{user.role})"
  end
  ```
</CodeGroup>

更新成员的角色：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/users/user_01WxYzAbCdEfGhJkMnPqRsTu" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{"role": "developer"}'
  ```

  ```bash CLI
  ant organization:users update \
    --user-id user_01WxYzAbCdEfGhJkMnPqRsTu \
    --role developer
  ```

  ```python Python
  client = anthropic.Anthropic()

  user = client.organization.users.update(
      "user_01WxYzAbCdEfGhJkMnPqRsTu", role="developer"
  )

  print(f"id: {user.id}")
  print(f"role: {user.role}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const user = await client.organization.users.update("user_01WxYzAbCdEfGhJkMnPqRsTu", {
    role: "developer"
  });

  console.log(`id: ${user.id}`);
  console.log(`role: ${user.role}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.Users;
  // ...

  AnthropicClient client = new();

  var user = await client.Organization.Users.Update(
      "user_01WxYzAbCdEfGhJkMnPqRsTu",
      new() { Role = Role.Developer }
  );

  Console.WriteLine($"id: {user.ID}");
  Console.WriteLine($"role: {user.Role.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  user, err := client.Organization.Users.Update(
  	context.Background(),
  	"user_01WxYzAbCdEfGhJkMnPqRsTu",
  	anthropic.OrganizationUserUpdateParams{
  		Role: anthropic.OrganizationUserUpdateParamsRoleDeveloper,
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
          .role(UserUpdateParams.Role.DEVELOPER)
          .build();
      var user = client.organization().users()
          .update("user_01WxYzAbCdEfGhJkMnPqRsTu", params);

      IO.println("id: " + user.id());
      IO.println("role: " + user.role().asString());
  }
  ```

  ```php PHP
  use Anthropic\Organization\Users\UserUpdateParams\Role;
  // ...

  $client = new Client();

  $user = $client->organization->users->update(
      userID: 'user_01WxYzAbCdEfGhJkMnPqRsTu',
      role: Role::DEVELOPER,
  );

  echo "id: {$user->id}\n";
  echo "role: {$user->role}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user_id = "user_01WxYzAbCdEfGhJkMnPqRsTu"
  user = client.organization.users.update(user_id, role: :developer)

  puts "id: #{user.id}"
  puts "role: #{user.role}"
  ```
</CodeGroup>

从组织中移除成员：

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/users/user_01WxYzAbCdEfGhJkMnPqRsTu" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:users remove --user-id user_01WxYzAbCdEfGhJkMnPqRsTu
  ```

  ```python Python
  client = anthropic.Anthropic()

  removed_user = client.organization.users.remove("user_01WxYzAbCdEfGhJkMnPqRsTu")

  print(f"id: {removed_user.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const removedUser = await client.organization.users.remove("user_01WxYzAbCdEfGhJkMnPqRsTu");

  console.log(`id: ${removedUser.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var removedUser = await client.Organization.Users.Remove("user_01WxYzAbCdEfGhJkMnPqRsTu");

  Console.WriteLine($"id: {removedUser.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  removedUser, err := client.Organization.Users.Remove(context.Background(), "user_01WxYzAbCdEfGhJkMnPqRsTu")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", removedUser.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var removedUser = client.organization().users()
      .remove("user_01WxYzAbCdEfGhJkMnPqRsTu");

  IO.println("id: " + removedUser.id());
  ```

  ```php PHP
  $client = new Client();

  $removedUser = $client->organization->users->remove(
      userID: 'user_01WxYzAbCdEfGhJkMnPqRsTu',
  );

  echo "id: {$removedUser->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user_id = "user_01WxYzAbCdEfGhJkMnPqRsTu"
  removed_user = client.organization.users.remove(user_id)

  puts "id: #{removed_user.id}"
  ```
</CodeGroup>

### 组织邀请

邀请用户加入您的组织并管理待处理的[邀请](https://platform.claude.com/docs/zh-CN/api/organization/invites/retrieve)。

邀请用户加入您的组织：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/invites" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "email": "user@example.com",
      "role": "developer"
    }'
  ```

  ```bash CLI
  ant organization:invites create --email user@example.com --role developer
  ```

  ```python Python
  client = anthropic.Anthropic()

  invite = client.organization.invites.create(email="user@example.com", role="developer")

  print(f"id: {invite.id}")
  print(f"email: {invite.email}")
  print(f"status: {invite.status}")
  print(f"expires_at: {invite.expires_at}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const invite = await client.organization.invites.create({
    email: "user@example.com",
    role: "developer"
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
      Email = "user@example.com",
      Role = Role.Developer
  });

  Console.WriteLine($"id: {invite.ID}");
  Console.WriteLine($"email: {invite.Email}");
  Console.WriteLine($"status: {invite.Status.Raw()}");
  Console.WriteLine($"expires_at: {invite.ExpiresAt:O}");
  ```

  ```go Go
  client := anthropic.NewClient()

  invite, err := client.Organization.Invites.New(context.Background(), anthropic.OrganizationInviteNewParams{
  	Email: "user@example.com",
  	Role:  anthropic.OrganizationInviteNewParamsRoleDeveloper,
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
          .email("user@example.com")
          .role(InviteCreateParams.Role.DEVELOPER)
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
      email: 'user@example.com',
      role: Role::DEVELOPER,
  );

  echo "id: {$invite->id}\n";
  echo "email: {$invite->email}\n";
  echo "status: {$invite->status}\n";
  echo "expires_at: {$invite->expiresAt->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  invite = client.organization.invites.create(email: "user@example.com", role: :developer)

  puts "id: #{invite.id}"
  puts "email: #{invite.email}"
  puts "status: #{invite.status}"
  puts "expires_at: #{invite.expires_at.iso8601}"
  ```
</CodeGroup>

列出待处理、已接受和已过期的邀请：

<CodeGroup>
  ```bash cURL
  # 返回一页。
  curl "https://api.anthropic.com/v1/organizations/invites?limit=10" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 在终端中，视图会在您滚动时加载更多页面。通过管道输出或使用 `--format json` 时，由 `--limit` 限制结果数量。
  ant organization:invites list --limit 10
  ```

  ```python Python
  client = anthropic.Anthropic()

  invites = client.organization.invites.list(limit=10)

  # 根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  for invite in invites:
      print(f"{invite.id}: {invite.email} ({invite.status})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const invites = await client.organization.invites.list({ limit: 10 });

  // 会根据需要自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
  for await (const invite of invites) {
    console.log(`${invite.id}: ${invite.email} (${invite.status})`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Invites.List(new() { Limit = 10 });

  // 会根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  await foreach (var invite in page.Paginate())
  {
      Console.WriteLine($"{invite.ID}: {invite.Email} ({invite.Status.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  invites := client.Organization.Invites.ListAutoPaging(context.Background(), anthropic.OrganizationInviteListParams{
  	Limit: anthropic.Int(10),
  })

  // 会根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
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
          .limit(10)
          .build();
      var invites = client.organization().invites().list(params);

      // 会按需自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
      for (var invite : invites.autoPager()) {
          IO.println(invite.id() + ": " + invite.email() + " (" + invite.status().asString() + ")");
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $invites = $client->organization->invites->list(limit: 10);

  // 仅读取第一页。
  foreach ($invites->getItems() as $invite) {
      echo "{$invite->id}: {$invite->email} ({$invite->status})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  invites = client.organization.invites.list(limit: 10)

  # 仅读取第一页。
  invites.data.each do |invite|
    puts "#{invite.id}: #{invite.email} (#{invite.status})"
  end
  ```
</CodeGroup>

删除邀请：

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/invites/invite_015gWxHNr6h6TdRPZTmuCGnn" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:invites delete --invite-id invite_015gWxHNr6h6TdRPZTmuCGnn
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted_invite = client.organization.invites.delete("invite_015gWxHNr6h6TdRPZTmuCGnn")

  print(f"id: {deleted_invite.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deletedInvite = await client.organization.invites.delete(
    "invite_015gWxHNr6h6TdRPZTmuCGnn"
  );

  console.log(`id: ${deletedInvite.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deletedInvite = await client.Organization.Invites.Delete(
      "invite_015gWxHNr6h6TdRPZTmuCGnn"
  );

  Console.WriteLine($"id: {deletedInvite.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  deletedInvite, err := client.Organization.Invites.Delete(context.Background(), "invite_015gWxHNr6h6TdRPZTmuCGnn")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", deletedInvite.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var deletedInvite = client.organization().invites()
      .delete("invite_015gWxHNr6h6TdRPZTmuCGnn");

  IO.println("id: " + deletedInvite.id());
  ```

  ```php PHP
  $client = new Client();

  $deletedInvite = $client->organization->invites->delete(
      inviteID: 'invite_015gWxHNr6h6TdRPZTmuCGnn',
  );

  echo "id: {$deletedInvite->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  invite_id = "invite_015gWxHNr6h6TdRPZTmuCGnn"
  deleted_invite = client.organization.invites.delete(invite_id)

  puts "id: #{deleted_invite.id}"
  ```
</CodeGroup>

### 工作区

有关 Console 和 API 示例，请参阅[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)。

### 工作区成员

管理[用户对特定工作区的访问权限](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/members/retrieve)。

向工作区添加成员：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/members" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "user_id": "user_01WxYzAbCdEfGhJkMnPqRsTu",
      "workspace_role": "workspace_developer"
    }'
  ```

  ```bash CLI
  ant organization:workspaces:members add \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --user-id user_01WxYzAbCdEfGhJkMnPqRsTu \
    --workspace-role workspace_developer
  ```

  ```python Python
  client = anthropic.Anthropic()

  member = client.organization.workspaces.members.add(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      user_id="user_01WxYzAbCdEfGhJkMnPqRsTu",
      workspace_role="workspace_developer",
  )

  print(f"user_id: {member.user_id}")
  print(f"workspace_role: {member.workspace_role}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const member = await client.organization.workspaces.members.add(
    "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    {
      user_id: "user_01WxYzAbCdEfGhJkMnPqRsTu",
      workspace_role: "workspace_developer"
    }
  );

  console.log(`user_id: ${member.user_id}`);
  console.log(`workspace_role: ${member.workspace_role}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.Workspaces;

  AnthropicClient client = new();

  var member = await client.Organization.Workspaces.Members.Add(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      new()
      {
          UserID = "user_01WxYzAbCdEfGhJkMnPqRsTu",
          WorkspaceRole = NoBillingWorkspaceRole.WorkspaceDeveloper
      }
  );

  Console.WriteLine($"user_id: {member.UserID}");
  Console.WriteLine($"workspace_role: {member.WorkspaceRole.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  member, err := client.Organization.Workspaces.Members.Add(
  	context.Background(),
  	"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  	anthropic.OrganizationWorkspaceMemberAddParams{
  		UserID:        "user_01WxYzAbCdEfGhJkMnPqRsTu",
  		WorkspaceRole: anthropic.NoBillingWorkspaceRoleWorkspaceDeveloper,
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("user_id: %s\n", member.UserID)
  fmt.Printf("workspace_role: %s\n", member.WorkspaceRole)
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.NoBillingWorkspaceRole;
  import com.anthropic.models.organization.workspaces.members.MemberAddParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MemberAddParams.builder()
          .userId("user_01WxYzAbCdEfGhJkMnPqRsTu")
          .workspaceRole(NoBillingWorkspaceRole.WORKSPACE_DEVELOPER)
          .build();
      var member = client.organization().workspaces().members()
          .add("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ", params);

      IO.println("user_id: " + member.userId());
      IO.println("workspace_role: " + member.workspaceRole().asString());
  }
  ```

  ```php PHP
  use Anthropic\Organization\Workspaces\NoBillingWorkspaceRole;
  // ...

  $client = new Client();

  $member = $client->organization->workspaces->members->add(
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
      userID: 'user_01WxYzAbCdEfGhJkMnPqRsTu',
      workspaceRole: NoBillingWorkspaceRole::WORKSPACE_DEVELOPER,
  );

  echo "user_id: {$member->userID}\n";
  echo "workspace_role: {$member->workspaceRole}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace_id = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  member = client.organization.workspaces.members.add(
    workspace_id,
    user_id: "user_01WxYzAbCdEfGhJkMnPqRsTu",
    workspace_role: :workspace_developer
  )

  puts "user_id: #{member.user_id}"
  puts "workspace_role: #{member.workspace_role}"
  ```
</CodeGroup>

列出工作区的成员：

<CodeGroup>
  ```bash cURL
  # 返回一页结果。
  curl "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/members?limit=10" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 在终端中，视图会随您滚动加载更多页面。通过管道输出或使用 `--format json` 时，`--limit` 用于限制结果数量。
  ant organization:workspaces:members list \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --limit 10
  ```

  ```python Python
  client = anthropic.Anthropic()

  members = client.organization.workspaces.members.list(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ", limit=10
  )

  # 根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  for member in members:
      print(f"{member.user_id}: {member.workspace_role}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const members = await client.organization.workspaces.members.list(
    "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    { limit: 10 }
  );

  // 会根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  for await (const member of members) {
    console.log(`${member.user_id}: ${member.workspace_role}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Workspaces.Members.List(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      new() { Limit = 10 }
  );

  // 会按需自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
  await foreach (var member in page.Paginate())
  {
      Console.WriteLine($"{member.UserID}: {member.WorkspaceRole.Raw()}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  members := client.Organization.Workspaces.Members.ListAutoPaging(
  	context.Background(),
  	"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  	anthropic.OrganizationWorkspaceMemberListParams{
  		Limit: anthropic.Int(10),
  	},
  )

  // 会根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  for members.Next() {
  	member := members.Current()
  	fmt.Printf("%s: %s\n", member.UserID, member.WorkspaceRole)
  }
  if err := members.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.members.MemberListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MemberListParams.builder()
          .limit(10)
          .build();
      var members = client.organization().workspaces().members()
          .list("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ", params);

      // 会根据需要自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
      for (var member : members.autoPager()) {
          IO.println(member.userId() + ": " + member.workspaceRole().asString());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $members = $client->organization->workspaces->members->list(
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
      limit: 10,
  );

  // 仅读取第一页。
  foreach ($members->getItems() as $member) {
      echo "{$member->userID}: {$member->workspaceRole}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace_id = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  members = client.organization.workspaces.members.list(workspace_id, limit: 10)

  # 仅读取第一页。
  members.data.each do |member|
    puts "#{member.user_id}: #{member.workspace_role}"
  end
  ```
</CodeGroup>

更新工作区成员的角色：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/members/user_01WxYzAbCdEfGhJkMnPqRsTu" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{"workspace_role": "workspace_admin"}'
  ```

  ```bash CLI
  ant organization:workspaces:members update \
    --user-id user_01WxYzAbCdEfGhJkMnPqRsTu \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --workspace-role workspace_admin
  ```

  ```python Python
  client = anthropic.Anthropic()

  member = client.organization.workspaces.members.update(
      "user_01WxYzAbCdEfGhJkMnPqRsTu",
      workspace_id="wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      workspace_role="workspace_admin",
  )

  print(f"user_id: {member.user_id}")
  print(f"workspace_role: {member.workspace_role}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const member = await client.organization.workspaces.members.update(
    "user_01WxYzAbCdEfGhJkMnPqRsTu",
    {
      workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      workspace_role: "workspace_admin"
    }
  );

  console.log(`user_id: ${member.user_id}`);
  console.log(`workspace_role: ${member.workspace_role}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.Workspaces;

  AnthropicClient client = new();

  var member = await client.Organization.Workspaces.Members.Update(
      "user_01WxYzAbCdEfGhJkMnPqRsTu",
      new()
      {
          WorkspaceID = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
          WorkspaceRole = WorkspaceRole.WorkspaceAdmin
      }
  );

  Console.WriteLine($"user_id: {member.UserID}");
  Console.WriteLine($"workspace_role: {member.WorkspaceRole.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  member, err := client.Organization.Workspaces.Members.Update(
  	context.Background(),
  	"user_01WxYzAbCdEfGhJkMnPqRsTu",
  	anthropic.OrganizationWorkspaceMemberUpdateParams{
  		WorkspaceID:   "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  		WorkspaceRole: anthropic.WorkspaceRoleWorkspaceAdmin,
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("user_id: %s\n", member.UserID)
  fmt.Printf("workspace_role: %s\n", member.WorkspaceRole)
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.WorkspaceRole;
  import com.anthropic.models.organization.workspaces.members.MemberUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MemberUpdateParams.builder()
          .workspaceId("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
          .workspaceRole(WorkspaceRole.WORKSPACE_ADMIN)
          .build();
      var member = client.organization().workspaces().members()
          .update("user_01WxYzAbCdEfGhJkMnPqRsTu", params);

      IO.println("user_id: " + member.userId());
      IO.println("workspace_role: " + member.workspaceRole().asString());
  }
  ```

  ```php PHP
  use Anthropic\Organization\Workspaces\WorkspaceRole;
  // ...

  $client = new Client();

  $member = $client->organization->workspaces->members->update(
      userID: 'user_01WxYzAbCdEfGhJkMnPqRsTu',
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
      workspaceRole: WorkspaceRole::WORKSPACE_ADMIN,
  );

  echo "user_id: {$member->userID}\n";
  echo "workspace_role: {$member->workspaceRole}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user_id = "user_01WxYzAbCdEfGhJkMnPqRsTu"
  member = client.organization.workspaces.members.update(
    user_id,
    workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    workspace_role: :workspace_admin
  )

  puts "user_id: #{member.user_id}"
  puts "workspace_role: #{member.workspace_role}"
  ```
</CodeGroup>

从工作区中移除成员：

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/members/user_01WxYzAbCdEfGhJkMnPqRsTu" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:workspaces:members remove \
    --user-id user_01WxYzAbCdEfGhJkMnPqRsTu \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ
  ```

  ```python Python
  client = anthropic.Anthropic()

  removed_member = client.organization.workspaces.members.remove(
      "user_01WxYzAbCdEfGhJkMnPqRsTu", workspace_id="wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  )

  print(f"user_id: {removed_member.user_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const removedMember = await client.organization.workspaces.members.remove(
    "user_01WxYzAbCdEfGhJkMnPqRsTu",
    { workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ" }
  );

  console.log(`user_id: ${removedMember.user_id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var removedMember = await client.Organization.Workspaces.Members.Remove(
      "user_01WxYzAbCdEfGhJkMnPqRsTu",
      new() { WorkspaceID = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ" }
  );

  Console.WriteLine($"user_id: {removedMember.UserID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  removedMember, err := client.Organization.Workspaces.Members.Remove(
  	context.Background(),
  	"user_01WxYzAbCdEfGhJkMnPqRsTu",
  	anthropic.OrganizationWorkspaceMemberRemoveParams{
  		WorkspaceID: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("user_id: %s\n", removedMember.UserID)
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.members.MemberRemoveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MemberRemoveParams.builder()
          .workspaceId("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
          .build();
      var removedMember = client.organization().workspaces().members()
          .remove("user_01WxYzAbCdEfGhJkMnPqRsTu", params);

      IO.println("user_id: " + removedMember.userId());
  }
  ```

  ```php PHP
  $client = new Client();

  $removedMember = $client->organization->workspaces->members->remove(
      userID: 'user_01WxYzAbCdEfGhJkMnPqRsTu',
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
  );

  echo "user_id: {$removedMember->userID}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user_id = "user_01WxYzAbCdEfGhJkMnPqRsTu"
  removed_member = client.organization.workspaces.members.remove(
    user_id,
    workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  )

  puts "user_id: #{removed_member.user_id}"
  ```
</CodeGroup>

### API 密钥

监控和管理 [API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/list)。响应中的每个密钥都包含其 `expires_at` 时间戳（对于没有[过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)的密钥为 `null`）以及 `principal`，即该密钥所代表的身份（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。对于个人密钥，`principal` 为 `{"type": "user_actor", "user_id": "user_..."}`。对于服务账户密钥，它为 `{"type": "service_account_actor", "service_account_id": "svac_..."}`。对于工作区密钥，它为 `null`。每个密钥还有一个 `scope` 对象：对于绑定到单个工作区的密钥，为 `{"type": "workspace", "workspace_id": "wrkspc_..."}`；对于可在账户有权访问的任何工作区中使用的密钥，为 `{"type": "organization"}`。使用默认工作区的 ID 按 `workspace_id` 筛选列表时，仅返回绑定到默认工作区的密钥；没有工作区范围的密钥不会在任何 `workspace_id` 筛选条件下返回。

列出工作区中的活动 API 密钥：

<CodeGroup>
  ```bash cURL
  # 返回一页结果。
  curl "https://api.anthropic.com/v1/organizations/api_keys?limit=10&status=active&workspace_id=wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 在终端中，视图会随您滚动获取更多页面。通过管道传输或使用 `--format json` 时，`--limit` 限制结果数量。
  ant organization:api-keys list \
    --limit 10 \
    --status active \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ
  ```

  ```python Python
  client = anthropic.Anthropic()

  api_keys = client.organization.api_keys.list(
      limit=10, status="active", workspace_id="wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  )

  # 根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  for api_key in api_keys:
      print(f"{api_key.id}: {api_key.name} ({api_key.status})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const apiKeys = await client.organization.apiKeys.list({
    limit: 10,
    status: "active",
    workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  });

  // 会根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  for await (const apiKey of apiKeys) {
    console.log(`${apiKey.id}: ${apiKey.name} (${apiKey.status})`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Organization.ApiKeys;

  AnthropicClient client = new();

  var page = await client.Organization.ApiKeys.List(new()
  {
      Limit = 10,
      Status = ApiKeyListParamsStatus.Active,
      WorkspaceID = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  });

  // 会根据需要自动获取更多页面，因此 `limit` 设置的是每页大小，而非总数。
  await foreach (var apiKey in page.Paginate())
  {
      Console.WriteLine($"{apiKey.ID}: {apiKey.Name} ({apiKey.Status.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  apiKeys := client.Organization.APIKeys.ListAutoPaging(context.Background(), anthropic.OrganizationAPIKeyListParams{
  	Limit:       anthropic.Int(10),
  	Status:      anthropic.OrganizationAPIKeyListParamsStatusActive,
  	WorkspaceID: anthropic.String("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"),
  })

  // 会根据需要自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
  for apiKeys.Next() {
  	apiKey := apiKeys.Current()
  	fmt.Printf("%s: %s (%s)\n", apiKey.ID, apiKey.Name, apiKey.Status)
  }
  if err := apiKeys.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.apikeys.ApiKeyListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = ApiKeyListParams.builder()
          .limit(10)
          .status(ApiKeyListParams.Status.ACTIVE)
          .workspaceId("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
          .build();
      var apiKeys = client.organization().apiKeys().list(params);

      // 会根据需要自动获取更多页，因此 `limit` 设置的是每页大小，而非总数。
      for (var apiKey : apiKeys.autoPager()) {
          IO.println(apiKey.id() + ": " + apiKey.name() + " (" + apiKey.status().asString() + ")");
      }
  }
  ```

  ```php PHP
  use Anthropic\Organization\APIKeys\APIKeyListParams\Status;
  // ...

  $client = new Client();

  $apiKeys = $client->organization->apiKeys->list(
      limit: 10,
      status: Status::ACTIVE,
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
  );

  // 仅读取第一页。
  foreach ($apiKeys->getItems() as $apiKey) {
      echo "{$apiKey->id}: {$apiKey->name} ({$apiKey->status})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  api_keys = client.organization.api_keys.list(
    limit: 10,
    status: :active,
    workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  )

  # 仅读取第一页。
  api_keys.data.each do |api_key|
    puts "#{api_key.id}: #{api_key.name} (#{api_key.status})"
  end
  ```
</CodeGroup>

重命名或停用 API 密钥：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/api_keys/apikey_01Rj2N8SVvo6BePZj99NhmiT" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "status": "inactive",
      "name": "New Key Name"
    }'
  ```

  ```bash CLI
  ant organization:api-keys update \
    --api-key-id apikey_01Rj2N8SVvo6BePZj99NhmiT \
    --status inactive \
    --name "New Key Name"
  ```

  ```python Python
  client = anthropic.Anthropic()

  api_key = client.organization.api_keys.update(
      "apikey_01Rj2N8SVvo6BePZj99NhmiT", status="inactive", name="New Key Name"
  )

  print(f"id: {api_key.id}")
  print(f"name: {api_key.name}")
  print(f"status: {api_key.status}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const apiKey = await client.organization.apiKeys.update("apikey_01Rj2N8SVvo6BePZj99NhmiT", {
    status: "inactive",
    name: "New Key Name"
  });

  console.log(`id: ${apiKey.id}`);
  console.log(`name: ${apiKey.name}`);
  console.log(`status: ${apiKey.status}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.ApiKeys;

  AnthropicClient client = new();

  var apiKey = await client.Organization.ApiKeys.Update(
      "apikey_01Rj2N8SVvo6BePZj99NhmiT",
      new()
      {
          Status = Status.Inactive,
          Name = "New Key Name"
      }
  );

  Console.WriteLine($"id: {apiKey.ID}");
  Console.WriteLine($"name: {apiKey.Name}");
  Console.WriteLine($"status: {apiKey.Status.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  apiKey, err := client.Organization.APIKeys.Update(
  	context.Background(),
  	"apikey_01Rj2N8SVvo6BePZj99NhmiT",
  	anthropic.OrganizationAPIKeyUpdateParams{
  		Status: anthropic.OrganizationAPIKeyUpdateParamsStatusInactive,
  		Name:   anthropic.String("New Key Name"),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", apiKey.ID)
  fmt.Printf("name: %s\n", apiKey.Name)
  fmt.Printf("status: %s\n", apiKey.Status)
  ```

  ```java Java
  import com.anthropic.models.organization.apikeys.ApiKeyUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = ApiKeyUpdateParams.builder()
          .status(ApiKeyUpdateParams.Status.INACTIVE)
          .name("New Key Name")
          .build();
      var apiKey = client.organization().apiKeys()
          .update("apikey_01Rj2N8SVvo6BePZj99NhmiT", params);

      IO.println("id: " + apiKey.id());
      IO.println("name: " + apiKey.name());
      IO.println("status: " + apiKey.status().asString());
  }
  ```

  ```php PHP
  use Anthropic\Organization\APIKeys\APIKeyUpdateParams\Status;
  // ...

  $client = new Client();

  $apiKey = $client->organization->apiKeys->update(
      apiKeyID: 'apikey_01Rj2N8SVvo6BePZj99NhmiT',
      status: Status::INACTIVE,
      name: 'New Key Name',
  );

  echo "id: {$apiKey->id}\n";
  echo "name: {$apiKey->name}\n";
  echo "status: {$apiKey->status}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  api_key_id = "apikey_01Rj2N8SVvo6BePZj99NhmiT"
  api_key = client.organization.api_keys.update(
    api_key_id,
    status: :inactive,
    name: "New Key Name"
  )

  puts "id: #{api_key.id}"
  puts "name: #{api_key.name}"
  puts "status: #{api_key.status}"
  ```
</CodeGroup>

### 服务账户

创建和管理服务账户（`svac_...`），即[服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)和[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)令牌所代表的非人类身份。与联合颁发者和联合规则端点一样，这些端点需要 `org:admin` OAuth 令牌。请参阅[使用 Admin API 管理 WIF 中的服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#service-accounts)。

### 联合身份颁发者

注册 OIDC 身份提供商（`fdis_...`），其令牌可以为您的组织声明工作负载身份。请参阅[使用 Admin API 管理 WIF 中的联合颁发者](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#federation-issuers)。

### 联合规则

管理将颁发者令牌映射到服务账户和作用域的规则（`fdrl_...`）。请参阅[使用 Admin API 管理 WIF 中的联合规则](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#federation-rules)。

## 访问组织信息

`/v1/organizations/me` 端点返回您的凭证所属的组织：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/me" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization retrieve
  ```

  ```python Python
  client = anthropic.Anthropic()

  organization = client.organization.retrieve()

  print(f"id: {organization.id}")
  print(f"name: {organization.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const organization = await client.organization.retrieve();

  console.log(`id: ${organization.id}`);
  console.log(`name: ${organization.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var organization = await client.Organization.Retrieve();

  Console.WriteLine($"id: {organization.ID}");
  Console.WriteLine($"name: {organization.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  organization, err := client.Organization.Get(context.Background())
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", organization.ID)
  fmt.Printf("name: %s\n", organization.Name)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var organization = client.organization().retrieve();

  IO.println("id: " + organization.id());
  IO.println("name: " + organization.name());
  ```

  ```php PHP
  $client = new Client();

  $organization = $client->organization->retrieve();

  echo "id: {$organization->id}\n";
  echo "name: {$organization->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  organization = client.organization.retrieve

  puts "id: #{organization.id}"
  puts "name: #{organization.name}"
  ```
</CodeGroup>

```json
{
  "id": "12345678-1234-5678-1234-567812345678",
  "type": "organization",
  "name": "Organization Name"
}
```

有关参数详情和响应架构，请参阅[组织信息 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/retrieve)。

## 用量和成本报告

要跟踪您组织的用量和成本，请使用 [用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)。[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api) 是此类数据的旧版 API。

## Claude Code 分析

使用 [Claude Code 分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-code-analytics-api) 监控开发者生产力和 Claude Code 采用情况。

## 速率限制

使用 [Rate Limits API](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api)（速率限制 API）读取为您的组织及其工作区配置的速率限制，并设置工作区速率限制。

## 合规 API

使用[合规 API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 检索您组织的审计和活动数据。Admin API 密钥只能读取活动源（Activity Feed）。如需完整访问权限，请参阅[设置合规 API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。

## 组织 webhook

使用[组织 webhook](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks) 注册 “webhook endpoint”（webhook 端点），以接收有关针对您组织的用户配置文件所采取的执行操作的事件。

## 最佳实践

* 为工作区和 API 密钥使用有意义的名称和描述
* 处理失败操作产生的错误
* 定期审计成员角色和权限
* 归档未使用的工作区，并删除不再需要的待处理邀请
* 监控 API 密钥使用情况，审计每个密钥的 [`expires_at`](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)，并定期轮换密钥

## 常见问题

<AccordionGroup>
  <Accordion title="使用 Admin API 需要哪些权限？">
    Admin API 接受 Admin API 密钥（以 `sk-ant-admin` 开头）、具有 `org:admin` 作用域的 OAuth bearer 令牌，或未限定于特定工作区的个人密钥或服务账户密钥。只有具有管理员角色的组织成员才能配置 Admin API 密钥，只有具有管理员、所有者或主要所有者角色的成员才能获取 `org:admin` 令牌。个人密钥或服务账户密钥拥有与所关联账户相同的权限。请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)。
  </Accordion>

  <Accordion title="Admin API 可以创建 API 密钥吗？">
    不可以。您需要在 Claude Console 中创建 API 密钥。Admin API 只能读取、重命名和更改现有密钥的状态。
  </Accordion>

  <Accordion title="移除用户时 API 密钥会怎样？">
    具体行为取决于[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)。

    个人密钥在其用户被从组织中移除时停止工作。服务账户密钥在其服务账户被归档时停止工作，但即使创建它们的用户被移除，它们仍会继续工作。工作区 API 密钥会继续工作。在 [Claude Code 工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#claude-code-workspace)中，每个密钥都绑定到创建它的成员，并在该成员被移除时停止工作。
  </Accordion>

  <Accordion title="可以通过 API 移除组织管理员吗？">
    不可以。API 无法移除具有 admin 角色的成员。
  </Accordion>

  <Accordion title="组织邀请的有效期是多久？">
    邀请在 21 天后过期。过期期限不可配置。
  </Accordion>
</AccordionGroup>

有关工作区的具体问题，请参阅[工作区常见问题](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#faq)。
