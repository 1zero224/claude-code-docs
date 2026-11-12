---
title: 工作区
url: https://platform.claude.com/docs/zh-CN/manage-claude/workspaces
description: 使用工作区组织 API 密钥、管理团队访问权限并控制成本。
---

工作区提供了一种在组织内组织 API 使用情况的方式。使用工作区可以分隔不同的项目、环境或团队，同时保持集中的计费和管理。

## 工作区的工作原理

每个组织都有一个**默认工作区（Default Workspace）**，它无法被重命名、归档或删除。当您创建额外的工作区时，可以为每个工作区分配成员、服务账户、API 密钥和资源限制。

主要特征：

* **工作区标识符**使用 `wrkspc_` 前缀（例如 `wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ`）。
* 默认情况下，**每个组织最多 100 个工作区**（已归档的工作区不计入）。如果您需要更多，请联系您的客户团队。
* **默认工作区**与其他工作区一样拥有 `wrkspc_` ID（在 [`anthropic-workspace-id` 响应头](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#identify-the-workspace-behind-an-api-response)中返回，并被[获取工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/retrieve)接受），但只有当您传递 `include_default=true` 时，它才会出现在[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)的结果中。[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 会在该 ID 下报告其用量和成本，而旧版[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api) 的报告中其 `workspace_id` 显示为 `null`（对于绑定到默认工作区的密钥，API 密钥的 `scope` 字段包含真实 ID）。
* **API 密钥**可以限定到单个工作区。在这种情况下，它们只能访问该工作区内的资源。其他 API 密钥不限定到工作区。使用这类密钥时，请在每个请求中发送[工作区 ID 请求头](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)来选择工作区。该密钥的用户或服务账户必须有权访问该工作区。

### Claude Code 工作区

当您组织的成员首次使用其 Claude Console 账户登录 [Claude Code](https://code.claude.com/docs/en/overview) 时，Anthropic 会自动在组织中创建一个 **Claude Code** 工作区并将该成员添加到其中。之后每个登录 Claude Code 的成员都会以相同方式被添加。

Claude Code 工作区将 Claude Code 流量与您的其他 API 工作负载分隔开：

* Claude Code 在登录时在此工作区中铸造一个按用户的 API 密钥。您无法从 Console 手动在其中创建密钥。
* 与工作区密钥不同，如果 Claude Code 密钥的所有者被从工作区或组织中移除，该密钥将停止工作。
* Claude Code 的用量单独进行速率限制，管理员可以在 [Claude Console 中的速率限制](https://platform.claude.com/usage/limits)下限制其在组织限制中所占的份额。
* 它是唯一支持按用户每月支出限制的工作区。

<Warning>
  归档 Claude Code 工作区会为整个组织禁用通过 Console 计费的 Claude Code 登录。
</Warning>

## 工作区角色和权限

成员可以在每个工作区中拥有不同的角色，从而实现细粒度的访问控制。

| 角色       | 权限                                                                                                                                                                                                                                                                  |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 工作区用户    | 仅使用 playground                                                                                                                                                                                                                                                      |
| 工作区受限开发者 | 创建和管理 API 密钥，使用 API<br />对于 Claude Console 中的 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)，拥有此角色的成员无法打开或启动会话、打开记忆存储或管理部署。他们也无法在 Console 的[文件](https://platform.claude.com/files)页面上下载或删除文件。这些限制仅适用于 Console 界面，不适用于 API。 |
| 工作区开发者   | 创建和管理 API 密钥，使用 API                                                                                                                                                                                                                                                 |
| 工作区管理员   | 完全控制工作区设置和成员                                                                                                                                                                                                                                                        |
| 工作区账单    | 查看工作区计费信息（从组织计费角色继承）                                                                                                                                                                                                                                                |

### 角色继承

* **组织管理员**自动获得所有工作区的工作区管理员访问权限。
* **组织计费成员**自动获得所有工作区的工作区计费访问权限。
* **组织用户和开发者**必须被显式添加到每个工作区。
* **服务账户**在 Claude Console 的[服务账户](https://platform.claude.com/settings/service-accounts)页面（`platform.claude.com/settings/service-accounts`）上添加到工作区：在账户的菜单中或其自身页面上选择**添加到工作区**。要查看某个工作区中的账户，请按工作区筛选列表。

<Note>
  工作区计费角色无法手动分配。它是从拥有组织计费角色继承而来的。
</Note>

## 管理工作区

<Note>
  只有组织管理员可以创建工作区。组织用户和开发者必须由管理员添加到工作区。
</Note>

### 使用 Console

在 Claude Console 的[工作区](https://platform.claude.com/settings/workspaces)页面上创建和管理工作区。

#### 创建工作区

<Steps>
  <Step title="打开工作区设置">
    在 Claude Console 中，前往[工作区](https://platform.claude.com/settings/workspaces)页面（`platform.claude.com/settings/workspaces`）。
  </Step>

  <Step title="打开创建工作区页面">
    点击**创建工作区**，打开 Console 中的[创建工作区](https://platform.claude.com/settings/workspaces/new)页面（`platform.claude.com/settings/workspaces/new`）。
  </Step>

  <Step title="配置工作区">
    输入工作区名称并选择一种颜色以便于视觉识别。
  </Step>

  <Step title="创建该工作区">
    点击**创建工作区**完成创建。
  </Step>
</Steps>

<Tip>
  要在 Console 中切换工作区，请打开属于某个工作区的页面，例如[文件](https://platform.claude.com/files)（`platform.claude.com/files`），然后使用侧边栏中的工作区选择器。
</Tip>

#### 编辑工作区详情

要修改工作区的名称或颜色：

1. 从列表中选择工作区。
2. 打开**更多操作**菜单（三个点）并选择**编辑详情**。
3. 更新名称或颜色并保存您的更改。

<Note>
  默认工作区无法被重命名或删除。
</Note>

#### 向工作区添加成员

1. 在 Claude Console 中，前往[成员](https://platform.claude.com/settings/members)页面（`platform.claude.com/settings/members`）。
2. 点击**添加到工作区**。
3. 选择工作区、要添加的组织成员及其[工作区角色](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#workspace-roles-and-permissions)。
4. 点击**添加到工作区**确认。

#### 从工作区移除成员

1. 在 Claude Console 的[成员](https://platform.claude.com/settings/members)页面上，从**工作区**下拉菜单中选择工作区。如果显示为**全部**，则**移除成员**会改为将该成员从您的组织中移除。
2. 在该成员所在行的末尾，打开**更多操作**菜单（三个点）并选择**移除成员**。
3. 点击**移除**确认。

<Note>
  组织管理员和计费成员在持有这些组织角色期间无法从工作区中移除。
</Note>

#### 设置工作区限制

您需要在不同的位置设置工作区的速率限制和支出限制：

* **速率限制：** 在 [Claude Console 中的速率限制](https://platform.claude.com/usage/limits)下，从**工作区**下拉菜单中选择工作区，然后点击模型旁边的**编辑**，设置每分钟请求数、输入令牌或输出令牌的限制。
* **支出限制：**&#x5728; Claude Console 的[工作区](https://platform.claude.com/settings/workspaces)页面上，选择工作区。在打开的面板中，点击**支出限额**以限制每月支出，并配置支出达到特定阈值时的提醒。

#### 归档工作区

要归档工作区，请打开其**更多操作**菜单（三个点）并选择**归档工作区**。归档操作：

* 保留历史数据以供报告使用
* 停用工作区并归档为其创建的每个 API 密钥
* 无法撤销

<Warning>
  归档工作区会在几秒钟内归档为该工作区创建的每个 API 密钥（它们仍会在 Admin API 中以已归档状态列出），并且未限定到工作区的密钥也无法再在其中操作。此操作无法撤销。如果您归档 [Claude Code 工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#claude-code-workspace)，您组织的成员将无法再通过 Console 计费登录 Claude Code。
</Warning>

### 使用 Admin API

使用 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 以编程方式管理工作区。

<Note>
  Admin API 端点接受 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)、`org:admin` OAuth 令牌，或未限定到特定工作区的个人或服务账户密钥。工作区密钥在那里不起作用。请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)。
</Note>

以下 SDK 和 CLI 示例构造默认客户端，该客户端从 `ANTHROPIC_API_KEY` 环境变量读取 Admin API 密钥；SDK 在 `client.organization.workspaces` (csharp, go: `client.Organization.Workspaces`; java: `client.organization().workspaces()`; php: `$client->organization->workspaces`) 下公开这些端点。SDK 的列表方法会按需获取后续页面，因此 `limit` 设置的是页面大小；PHP、Ruby 和 curl 示例只返回一页。

创建工作区：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/workspaces" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{"name": "Production"}'
  ```

  ```bash CLI
  ant organization:workspaces create --name Production
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspace = client.organization.workspaces.create(name="Production")

  print(f"id: {workspace.id}")
  print(f"name: {workspace.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const workspace = await client.organization.workspaces.create({ name: "Production" });

  console.log(`id: ${workspace.id}`);
  console.log(`name: ${workspace.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var workspace = await client.Organization.Workspaces.Create(new()
  {
      Name = "Production"
  });

  Console.WriteLine($"id: {workspace.ID}");
  Console.WriteLine($"name: {workspace.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  workspace, err := client.Organization.Workspaces.New(context.Background(), anthropic.OrganizationWorkspaceNewParams{
  	Name: "Production",
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", workspace.ID)
  fmt.Printf("name: %s\n", workspace.Name)
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.WorkspaceCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = WorkspaceCreateParams.builder()
          .name("Production")
          .build();
      var workspace = client.organization().workspaces().create(params);

      IO.println("id: " + workspace.id());
      IO.println("name: " + workspace.name());
  }
  ```

  ```php PHP
  $client = new Client();

  $workspace = $client->organization->workspaces->create(
      name: 'Production',
  );

  echo "id: {$workspace->id}\n";
  echo "name: {$workspace->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace = client.organization.workspaces.create(name: "Production")

  puts "id: #{workspace.id}"
  puts "name: #{workspace.name}"
  ```
</CodeGroup>

列出工作区：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/workspaces?limit=10&include_archived=false" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:workspaces list --limit 10 --include-archived=false
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspaces = client.organization.workspaces.list(limit=10, include_archived=False)

  for workspace in workspaces:
      print(f"{workspace.id}: {workspace.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const workspaces = await client.organization.workspaces.list({
    limit: 10,
    include_archived: false
  });

  for await (const workspace of workspaces) {
    console.log(`${workspace.id}: ${workspace.name}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var workspaces = await client.Organization.Workspaces.List(new()
  {
      Limit = 10,
      IncludeArchived = false
  });

  await foreach (var workspace in workspaces.Paginate())
  {
      Console.WriteLine($"{workspace.ID}: {workspace.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  workspaces := client.Organization.Workspaces.ListAutoPaging(context.Background(), anthropic.OrganizationWorkspaceListParams{
  	Limit:           anthropic.Int(10),
  	IncludeArchived: anthropic.Bool(false),
  })

  for workspaces.Next() {
  	workspace := workspaces.Current()
  	fmt.Printf("%s: %s\n", workspace.ID, workspace.Name)
  }
  if err := workspaces.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.WorkspaceListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = WorkspaceListParams.builder()
          .limit(10)
          .includeArchived(false)
          .build();
      var workspaces = client.organization().workspaces().list(params);

      for (var workspace : workspaces.autoPager()) {
          IO.println(workspace.id() + ": " + workspace.name());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $workspaces = $client->organization->workspaces->list(
      limit: 10,
      includeArchived: false,
  );

  foreach ($workspaces->getItems() as $workspace) {
      echo "{$workspace->id}: {$workspace->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspaces = client.organization.workspaces.list(limit: 10, include_archived: false)

  workspaces.data.each do |workspace|
    puts "#{workspace.id}: #{workspace.name}"
  end
  ```
</CodeGroup>

归档工作区：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:workspaces archive \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspace = client.organization.workspaces.archive("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")

  print(f"id: {workspace.id}")
  print(f"archived_at: {workspace.archived_at}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const workspace = await client.organization.workspaces.archive(
    "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  );

  console.log(`id: ${workspace.id}`);
  console.log(`archived_at: ${workspace.archived_at}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var workspace = await client.Organization.Workspaces.Archive(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  );

  Console.WriteLine($"id: {workspace.ID}");
  Console.WriteLine($"archived_at: {workspace.ArchivedAt:O}");
  ```

  ```go Go
  client := anthropic.NewClient()

  workspace, err := client.Organization.Workspaces.Archive(context.Background(), "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", workspace.ID)
  fmt.Printf("archived_at: %s\n", workspace.ArchivedAt)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var workspace = client.organization().workspaces()
      .archive("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ");

  IO.println("id: " + workspace.id());
  IO.println("archived_at: " + workspace.archivedAt().orElseThrow());
  ```

  ```php PHP
  $client = new Client();

  $workspace = $client->organization->workspaces->archive(
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
  );

  echo "id: {$workspace->id}\n";
  echo "archived_at: {$workspace->archivedAt?->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace_id = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  workspace = client.organization.workspaces.archive(workspace_id)

  puts "id: #{workspace.id}"
  puts "archived_at: #{workspace.archived_at}"
  ```
</CodeGroup>

有关完整的参数详情和响应结构，请参阅[工作区 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/retrieve)。

### 管理工作区成员

向工作区添加成员：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/members" \
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

更新成员的角色：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/members/user_01WxYzAbCdEfGhJkMnPqRsTu" \
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

从工作区移除成员：

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
      "user_01WxYzAbCdEfGhJkMnPqRsTu",
      workspace_id="wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
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

有关完整的参数详情，请参阅[工作区成员 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/members/retrieve)。

## API 密钥和资源限定

每个请求都在恰好一个工作区中运行，并且只能访问该工作区内的资源。具体是哪个工作区取决于[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)：

* **工作区密钥**（没有所有者的旧式密钥）属于创建它的工作区，并始终在那里运行。
* **个人密钥**或**服务账户密钥**以其用户或服务账户的身份执行操作。单工作区密钥始终在创建时选定的工作区中运行。未限定到工作区的密钥在每个请求的 `anthropic-workspace-id` 请求头所指定的工作区中运行。账户必须有权访问该工作区才能使用它。

限定到工作区的资源包括：

* 通过 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 创建的**文件**
* 通过 [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 创建的**消息批次**
* 通过 [Skills API](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide) 创建的**技能**

某些资源的管理方式不同：

* **[MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)**&#x4F7F;用通过[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)获得的 `workspace:manage_tunnels` OAuth 令牌进行管理，而不是 API 密钥。隧道在工作区中创建，Console 的 **MCP 隧道**列表和托管代理服务器选择器仅显示当前工作区中的隧道；10 个活动隧道的上限适用于整个组织。隧道管理需要具有隧道管理权限的角色；组织开发者可以查看但无法更改它们。
* **工作区**本身和**组织成员**在组织级别通过 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 进行管理，使用 Admin API 密钥、`org:admin` OAuth 令牌，或未限定到特定工作区的个人或服务账户密钥。

要查找组织的工作区 ID，请调用[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)端点（传递 `include_default=true` 以包含默认工作区），或在 Claude Console 的[工作区](https://platform.claude.com/settings/workspaces)页面上查找。

<Note>
  在 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 和 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上，[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)也按工作区隔离。在 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 上，提示缓存按组织隔离。
</Note>

## 识别 API 响应背后的工作区

Claude API 响应在 `request-id` 和 `anthropic-organization-id` [响应头](https://platform.claude.com/docs/zh-CN/api/overview#response-headers)旁边包含一个 `anthropic-workspace-id` 头。其值是请求的 API 密钥或访问令牌解析到的工作区的 `wrkspc_` 前缀 ID，包括当该工作区是默认工作区时。例如，成功的响应包含如下头：

```http
HTTP/1.1 200 OK
request-id: req_018EeWyXxfu5pfWkrYcMdjWG
anthropic-organization-id: 0d0e7a3b-52f1-4c7e-9a51-3f6f2f7c1b9e
anthropic-workspace-id: wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ
```

当凭据未解析到工作区时（例如在 Admin API 请求上），或当请求在身份验证完成之前失败时（例如 401 错误），该头不存在。

以下示例发送一个 Messages API 请求并从响应头中读取工作区 ID：

<CodeGroup>
  ```bash cURL
  # -i 会在响应体之前打印响应头，其中包括 anthropic-workspace-id
  curl -sS -i https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}]
    }'
  ```

  ```bash CLI
  # --debug 将 HTTP 响应（包括 Anthropic-Workspace-Id
  # 标头）打印到 stderr
  ant --debug messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.with_raw_response.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  workspace_id = response.headers.get("anthropic-workspace-id")
  print(f"Workspace ID: {workspace_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const { response } = await client.messages
    .create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    })
    .withResponse();
  console.log("Workspace ID:", response.headers.get("anthropic-workspace-id"));
  ```

  ```csharp C#
  AnthropicClient client = new();

  using var response = await client.WithRawResponse.Messages.Create(new()
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }]
  });
  var workspaceId = response.GetHeaderValues("anthropic-workspace-id").First();
  Console.WriteLine($"Workspace ID: {workspaceId}");
  ```

  ```go Go
  client := anthropic.NewClient()

  var response *http.Response
  _, err := client.Messages.New(
  	context.Background(),
  	anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  		},
  	},
  	option.WithResponseInto(&response),
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println("Workspace ID:", response.Header.Get("anthropic-workspace-id"))
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.http.HttpResponseFor;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      HttpResponseFor<Message> response = client.messages().withRawResponse().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024)
              .addUserMessage("Hello, Claude")
              .build()
      );

      String workspaceId = response.headers().values("anthropic-workspace-id").getFirst();
      IO.println("Workspace ID: " + workspaceId);
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->raw->create([
      'model' => Model::CLAUDE_OPUS_5_5,
      'maxTokens' => 1024,
      'messages' => [['role' => 'user', 'content' => 'Hello, Claude']],
  ]);
  echo 'Workspace ID: ' . $response->getHeaderLine('anthropic-workspace-id') . "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 在每个请求的中间件中读取响应头，该中间件接收
  # SDK 解析之前的原始 HTTP 响应
  workspace_id = nil
  read_workspace_id = lambda do |request, call_next|
    response = call_next.call(request)
    # response.headers 中的键为小写
    workspace_id = response.headers["anthropic-workspace-id"]
    response
  end

  client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    request_options: { middleware: [read_workspace_id] }
  )
  puts "Workspace ID: #{workspace_id}"
  ```
</CodeGroup>

```text Output wrap
Workspace ID: wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ
```

相同的访问器也可以从其他 Claude API 端点读取该头，包括 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) API。例如，从[创建会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)的响应中读取 `anthropic-workspace-id`，以记录该会话属于哪个工作区。

有了来自响应的工作区 ID，您可以：

* 确认该请求计入了哪个工作区的使用情况、成本和[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)
* 将其与以下内容进行比对：[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)行中的 `workspace_id` 字段（对于默认工作区，这些行包含相同的 ID），以及 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 中 API 密钥的 `scope` 字段，该字段包含密钥所绑定工作区的真实 ID，包括默认工作区
* 使用 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)将其传递给[获取工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/retrieve)，以检查它是否为您的默认工作区的 ID：默认工作区会返回 `"name": "Default"`，尽管除非您传入 `include_default=true`，否则[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)会省略它
* 在 [Console](https://platform.claude.com/settings/workspaces) 中打开该工作区以查找请求的资源，例如会话、文件、消息批次和技能

## 工作区限制

您可以为每个工作区设置自定义支出和速率限制，以防止过度使用并确保公平的资源分配。

### 设置工作区限制

您可以将工作区限制设置为低于（但不高于）您组织的限制：

* **支出限制：**&#x9650;制工作区的每月支出。要进行设置，请前往 Claude Console 中的[工作区](https://platform.claude.com/settings/workspaces)页面（`platform.claude.com/settings/workspaces`）并选择工作区。在打开的面板中，点击**支出限额**。
* **速率限制：**&#x9650;制每分钟请求数、每分钟输入令牌数或每分钟输出令牌数。要进行设置，请前往 [Claude Console 中的速率限制](https://platform.claude.com/usage/limits)，从**工作区**下拉菜单中选择工作区，然后点击模型旁边的**编辑**。

<Note>
  - 您无法为默认工作区设置支出限制。
  - 如果未设置，工作区限制与组织限制相同。
  - 即使工作区限制的总和超过组织限制，组织范围的限制也始终适用。
</Note>

有关速率限制及其工作原理的详细信息，请参阅[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)。您还可以使用[速率限制 API](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api) 以编程方式读取当前组织和工作区的速率限制，并设置工作区速率限制。该 API 会存储高于组织限制的值，但仍会执行两者中较低的那个限制。

## 使用情况和成本跟踪

使用[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)按工作区跟踪使用量和成本：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/usage?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-09-08T00:00:00Z&\
  workspace_ids[]=wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ&\
  group_by[]=workspace_id&\
  time_bucket_width=1d" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:usage list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-09-08T00:00:00Z \
    --time-bucket-width 1d \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --group-by workspace_id
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.usage.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-09-08T00:00:00Z",
      time_bucket_width="1d",
      workspace_ids=["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
      group_by=["workspace_id"],
  )

  for row in rows:
      print(f"{row.starting_at} {row.group.workspace_id}: {row.usage.total_tokens}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.usage.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-09-08T00:00:00Z",
    time_bucket_width: "1d",
    workspace_ids: ["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
    group_by: ["workspace_id"]
  });

  for await (const row of rows) {
    console.log(`${row.starting_at} ${row.group.workspace_id}: ${row.usage.total_tokens}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics;
  using Anthropic.Models.Beta.Organization.Analytics.Usage;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Usage.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-09-08T00:00:00Z"),
          TimeBucketWidth = BetaAnalyticsTimeBucketWidth.Day,
          WorkspaceIds = ["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
          GroupBy = [BetaAnalyticsUsageGroupBy.WorkspaceID],
      }
  );

  await foreach (var row in page.Paginate())
  {
      Console.WriteLine($"{row.StartingAt} {row.Group.WorkspaceID}: {row.Usage.TotalTokens}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Usage.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsUsageListParams{
  	StartingAt:      time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:        anthropic.Time(time.Date(2026, time.September, 8, 0, 0, 0, 0, time.UTC)),
  	TimeBucketWidth: anthropic.BetaAnalyticsTimeBucketWidthDay,
  	WorkspaceIDs:    []string{"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"},
  	GroupBy:         []anthropic.BetaAnalyticsUsageGroupBy{anthropic.BetaAnalyticsUsageGroupByWorkspaceID},
  })

  for rows.Next() {
  	row := rows.Current()
  	fmt.Printf("%s %s: %d\n", row.StartingAt.Format(time.RFC3339), row.Group.WorkspaceID, row.Usage.TotalTokens)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.BetaAnalyticsTimeBucketWidth;
  import com.anthropic.models.beta.organization.analytics.usage.BetaAnalyticsUsageGroupBy;
  import com.anthropic.models.beta.organization.analytics.usage.UsageListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UsageListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-09-08T00:00:00Z"))
          .timeBucketWidth(BetaAnalyticsTimeBucketWidth.DAY)
          .addWorkspaceId("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
          .addGroupBy(BetaAnalyticsUsageGroupBy.WORKSPACE_ID)
          .build();
      var rows = client.beta().organization().analytics().usage().list(params);

      for (var row : rows.autoPager()) {
          IO.println(row.startingAt() + " " + row.group().workspaceId().orElse(null) + ": " + row.usage().totalTokens());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->usage->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-09-08T00:00:00Z'),
      timeBucketWidth: '1d',
      workspaceIDs: ['wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ'],
      groupBy: ['workspace_id'],
  );

  foreach ($rows->pagingEachItem() as $row) {
      $startingAt = $row->startingAt->format(\DateTimeInterface::ATOM);
      echo "{$startingAt} {$row->group->workspaceID}: {$row->usage->totalTokens}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.usage.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 9, 8),
    time_bucket_width: :"1d",
    workspace_ids: ["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
    group_by: [:workspace_id]
  )

  page.auto_paging_each do |row|
    puts "#{row.starting_at} #{row.group.workspace_id}: #{row.usage.total_tokens}"
  end
  ```
</CodeGroup>

默认工作区中的使用量和成本以默认工作区自身的 ID 进行报告。

## 常见用例

### 环境分隔

为开发、预发布和生产创建单独的工作区：

| 工作区 | 用途               |
| --- | ---------------- |
| 开发  | 使用较低速率限制进行测试和实验  |
| 预发布 | 使用类似生产的限制进行预生产测试 |
| 生产  | 具有完整速率限制和监控的实时流量 |

### 团队或部门隔离

将工作区分配给不同的团队以进行成本分配和访问控制：

* 具有开发者访问权限的**工程团队**
* 拥有自己 API 密钥的**数据科学团队**
* 对客户工具具有有限访问权限的**支持团队**

### 基于项目的组织

为特定项目或产品创建工作区，以分别跟踪使用情况和成本。

## 最佳实践

<Steps>
  <Step title="规划您的工作区结构">
    在创建工作区之前，考虑您将如何组织工作区。考虑计费、访问控制和使用情况跟踪需求。
  </Step>

  <Step title="使用有意义的名称">
    清晰地命名工作区以指示其用途（例如 “Production - Customer Chatbot” 或 “Dev - Internal Tools”）。
  </Step>

  <Step title="设置适当的限制">
    配置支出和速率限制以防止意外成本并确保公平的资源分配。
  </Step>

  <Step title="定期审计访问权限">
    定期审查工作区成员资格，以确保只有适当的用户才能访问。
  </Step>

  <Step title="监控使用情况">
    使用[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)跟踪工作区级别的消耗。
  </Step>
</Steps>

## 常见问题

<AccordionGroup>
  <Accordion title="什么是默认工作区？">
    每个组织都有一个"默认工作区"，它不能被重命名、归档或删除。与所有工作区一样，它拥有一个 `wrkspc_` ID：API 会在 [`anthropic-workspace-id` 响应头](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#identify-the-workspace-behind-an-api-response)中返回它，您可以将其传递给[获取工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/retrieve)和[更新工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/update)。它没有自己的成员列表，因为对它的访问权限取决于每个成员的组织角色。只有在您传入 `include_default=true` 时，它才会出现在[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)的结果中。[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)以其真实 ID 报告其使用量和成本。旧版[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)报告中的 `workspace_id` 显示为 `null`。对于属于它的 API 密钥，该密钥的 `scope` 字段包含该工作区的真实 ID。
  </Accordion>

  <Accordion title="什么是 Claude Code 工作区？">
    当您组织的成员首次使用其 Console 账户登录 Claude Code 时，Anthropic 会自动创建 Claude Code 工作区。它将 Claude Code 的 API 密钥、使用情况和速率限制与您的其他工作负载隔离开。有关详情，请参阅 [Claude Code 工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#claude-code-workspace)。
  </Accordion>

  <Accordion title="工作区有限制吗？">
    是的。默认情况下，每个组织最多可以有 100 个工作区，已归档的工作区不计入此限制。如果您需要更多，请联系您的客户团队。
  </Accordion>

  <Accordion title="组织角色如何影响工作区访问？">
    组织管理员自动在所有工作区中获得工作区管理员角色。组织计费成员自动获得工作区计费角色。组织用户和开发者必须手动添加到每个工作区。
  </Accordion>

  <Accordion title="可以在工作区中分配哪些角色？">
    组织用户和开发者可以被分配工作区管理员、工作区开发者、工作区受限开发者或工作区用户角色。工作区计费角色无法手动分配；它是从拥有组织 `billing` 角色继承而来的。
  </Accordion>

  <Accordion title="可以更改组织管理员或计费成员的工作区角色吗？">
    组织管理员和计费成员在持有这些组织角色期间无法更改其工作区角色或从工作区中移除（有一个例外：计费成员可以升级为工作区管理员角色）。对于受此约束的其他所有人，请先更改其组织角色以更改其工作区访问权限。
  </Accordion>

  <Accordion title="当组织角色更改时，工作区访问会发生什么？">
    如果组织管理员或计费成员被降级为用户或开发者，他们将失去对所有工作区的访问权限，除了那些他们被手动分配了角色的工作区。当用户被提升为管理员或计费角色时，他们将自动获得对所有工作区的访问权限。
  </Accordion>

  <Accordion title="当用户从工作区中移除时，API 密钥会发生什么？">
    行为取决于[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)。

    个人或服务账户密钥在其用户或服务账户从工作区中移除后不久就会停止在该工作区中工作。即使创建服务账户密钥的用户被移除，该密钥仍会继续工作。工作区 API 密钥继续工作。在 [Claude Code 工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#claude-code-workspace)中，每个密钥都绑定到创建它的成员，并在该成员被移除时停止工作。

    当个人密钥的用户从组织中移除时，个人密钥会被归档。如果该用户被重新邀请，他们需要创建新密钥；已归档的密钥不会被恢复。
  </Accordion>
</AccordionGroup>

## 另请参阅

* [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)
* [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/organization)
* [速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)
* [用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)和旧版[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)
