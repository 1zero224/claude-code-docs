---
title: 使用 Admin API 管理 WIF
url: https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api
description: 以编程方式创建和管理工作负载身份联合服务账户、颁发者和规则，适用于基础设施即代码和 CI 工作流。
---

Admin API 让您能够以编程方式创建和管理[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)（WIF）资源：服务账户、联合颁发者和联合规则。您可以使用它将联合配置保存在基础设施即代码中、从 CI 进行预配，并在多个组织之间复现配置，而无需在 Claude Console 中逐项点击操作。这些端点与 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 的其余部分共享 `/v1/organizations` 路径前缀。

## 前提条件

本页上的每个请求都使用携带 `org:admin` 范围的 OAuth bearer 令牌进行身份验证。Anthropic 仅将该范围授予具有管理员、所有者或主要所有者角色的组织成员。该范围授予对整个组织的访问权限，这些端点会忽略任何工作区绑定。获取令牌有两种方式，它们具有不同的权限。通过您自己登录获得的令牌以用户身份执行操作。联合令牌以服务账户身份执行操作，无法执行本页上的所有操作。

### 交互式（您的终端）

在专门保留用于管理的 shell 中，使用 [`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart) 在专用配置文件下登录，请求 `org:admin` 范围（请参阅[管理员访问](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication#admin-access)），然后导出 bearer 令牌：

```bash CLI
ant auth login --profile admin --scope "org:admin"
export ANTHROPIC_AUTH_TOKEN=$(ant auth print-credentials --profile admin --access-token)
```

使用 `--profile admin` 登录会将 `org:admin` 凭据存储在其自己的配置文件名下，同时也使其成为 CLI 的活动配置文件。导出的变量适用于该 shell 中的每个 SDK 和 CLI 调用。完成后，请取消设置该变量，并使用 `ant profile activate default` 将 CLI 切换回去。

交互式令牌的有效期很短。如果请求开始返回 401，请重新运行 `export` 命令以刷新令牌。

SDK 和 `ant` CLI 会自动读取 `ANTHROPIC_AUTH_TOKEN`。请在同一 shell 中保持 `ANTHROPIC_API_KEY` 未设置。当两者都设置时，SDK 和 CLI 会使用该密钥，而这些端点会拒绝 API 密钥。

### 工作负载（CI 和自动化）

创建一个带有 `oauth_scope: org:admin` 的联合规则，其目标是 `organization_role` 为 `admin` 的服务账户。您必须在 Claude Console 中创建该规则本身。授予工作负载组织管理员访问权限是一项需要人工有意执行的操作，而不是自动化可以自行引导完成的事情。下一节将逐步介绍这一设置，每个组织只需执行一次。

## 引导工作负载以管理 WIF

只需一条在 Console 中创建的规则，即可将其余联合配置纳入基础设施即代码管理。为单个受信任的工作负载授予 `org:admin` 范围，然后让该工作负载通过此 API 管理联合颁发者以及所有工作区范围的联合规则。

<Steps>
  <Step title="在 Console 中创建 org:admin 规则">
    在 Claude Console 中，前往[工作负载身份](https://platform.claude.com/settings/workload-identity-federation)页面（`platform.claude.com/settings/workload-identity-federation`），然后选择**连接工作负载**。为您的自动化工作负载创建一条联合规则，例如基础设施仓库中的 GitHub Actions 工作流。在**高级规则选项**下，将规则的 OAuth 范围设置为 `org:admin`。向导随后会创建具有 Admin 组织角色的新服务账户，或要求您选择一个现有的管理员服务账户作为目标。

    <Warning>
      请将规则匹配到一个确切的工作负载身份，而不是宽泛的模式。除非 `subject_prefix` 以 `*` 结尾，否则它是精确匹配。对于 GitHub Actions，请将主体固定到受保护的分支，例如 `repo:my-org/my-repo:ref:refs/heads/main`。诸如 `repo:my-org/my-repo:*` 之类的尾随通配符也会匹配 `pull_request` 运行，包括从 fork 触发的运行。使用该模式时，任何能够向该仓库提交拉取请求的人都可以生成 `org:admin` 令牌。请参阅[限制哪些工作流可以进行认证](https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/github-actions#restrict-which-workflows-can-authenticate)。
    </Warning>
  </Step>

  <Step title="交换工作负载的身份令牌">
    使用 SDK 或 `ant` CLI 的工作负载不会自行执行交换。客户端会在第一次请求时交换身份令牌。在生成的访问令牌过期之前，它会重新读取身份令牌并再次交换。在不带参数的情况下构造客户端，与[构造 SDK 客户端](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#construct-the-sdk-client)中用于推理的方式完全相同，并通过联合环境变量将客户端指向该规则：

    ```bash
    export ANTHROPIC_FEDERATION_RULE_ID=fdrl_...        # the org:admin rule from step 1
    export ANTHROPIC_ORGANIZATION_ID=00000000-0000-0000-0000-000000000000
    export ANTHROPIC_SERVICE_ACCOUNT_ID=svac_...       # the rule's target service account
    export ANTHROPIC_IDENTITY_TOKEN_FILE=/path/to/jwt  # or ANTHROPIC_IDENTITY_TOKEN
    # 仅当该规则对所有工作区或多个工作区启用时，才需要 ANTHROPIC_WORKSPACE_ID；
    # org:admin 端点会忽略该绑定。
    unset ANTHROPIC_API_KEY ANTHROPIC_AUTH_TOKEN       # both take precedence over federation
    ```

    SDK 也接受这些设置作为显式构造函数参数，如[构造 SDK 客户端](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#construct-the-sdk-client)中所示。`ant` CLI 也读取这些变量，或者接受 `--federation-rule`、`--organization-id`、`--service-account-id` 和 `--identity-token-file` 标志。变量的完整列表及其优先顺序请参阅[环境变量](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#environment-variables)和[凭据优先级](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#credential-precedence)。

    对于运行多个 `ant` 命令的工作负载，请使用[联合配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#profile-configuration-file)，而不是标志或环境变量。配置文件会在命令之间缓存访问令牌。使用标志或变量时，每个命令都会再次交换身份令牌。默认情况下，Anthropic 只接受一次带有 `jti` 声明的身份令牌，而 GitHub Actions 令牌带有此声明。Anthropic 会拒绝出示同一令牌的第二个命令。

    当规则对所有工作区或多个工作区启用时，请为 `ant` CLI 提供一个设置了 `workspace_id` 的联合配置文件，并将 `ANTHROPIC_PROFILE` 设置为其名称。与 SDK 不同，CLI 只通过联合配置文件将 `ANTHROPIC_WORKSPACE_ID` 传入交换过程，而 `--workspace-id` 永远不会进入交换过程。

    使用 curl 调用 API 的工作负载需要自行将 JWT 交换为短期有效的 `org:admin` bearer 令牌。它使用与任何其他联合工作负载相同的[令牌交换](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#authenticate-from-your-workload)，并在 `authorization: Bearer` 标头中发送该令牌。
  </Step>

  <Step title="通过 API 管理颁发者和工作区范围的规则">
    配置好客户端后（或者对于 curl，将生成的令牌放入 `ANTHROPIC_AUTH_TOKEN` 后），工作负载即可使用本页上的端点创建和管理您的联合配置。
  </Step>
</Steps>

有关联合令牌可以执行和不能执行的操作，请参阅[权限和约束](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#permissions-and-constraints)。如果您已经使用**连接工作负载**向导创建了颁发者、服务账户或规则，请使用以下端点列出它们。然后将其导入您的基础设施即代码状态中，而不是重新创建。

## 身份验证

所有端点都位于 `https://api.anthropic.com/v1/organizations/` 下。对联合和服务账户端点的每个请求都需要 API 版本标头和 bearer 令牌。

在 SDK 中，这些端点是 `client.organization.service_accounts` (typescript: `client.organization.serviceAccounts`; csharp, go: `client.Organization.ServiceAccounts`; java: `client.organization().serviceAccounts()`; php: `$client->organization->serviceAccounts`)、`client.organization.federation.issuers` (csharp, go: `client.Organization.Federation.Issuers`; java: `client.organization().federation().issuers()`; php: `$client->organization->federation->issuers`) 和 `client.organization.federation.rules` (csharp, go: `client.Organization.Federation.Rules`; java: `client.organization().federation().rules()`; php: `$client->organization->federation->rules`)（在 CLI 中为 `ant organization:service-accounts`、`ant organization:federation:issuers` 和 `ant organization:federation:rules`）。SDK 和 `ant` CLI 会发送来自 `ANTHROPIC_AUTH_TOKEN` 的 bearer 令牌。在自动化工作负载中，它们会按照[引导工作负载以管理 WIF](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#bootstrap-a-workload-to-manage-wif) 中的描述自行执行联合交换。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/service_accounts" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:service-accounts list
  ```

  ```python Python
  client = anthropic.Anthropic()

  service_accounts = client.organization.service_accounts.list()

  for service_account in service_accounts:
      print(f"{service_account.id}: {service_account.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  for await (const serviceAccount of client.organization.serviceAccounts.list()) {
    console.log(`${serviceAccount.id}: ${serviceAccount.name}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.ServiceAccounts.List();

  await foreach (var serviceAccount in page.Paginate())
  {
      Console.WriteLine($"{serviceAccount.ID}: {serviceAccount.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  serviceAccounts := client.Organization.ServiceAccounts.ListAutoPaging(context.Background(), anthropic.OrganizationServiceAccountListParams{})

  for serviceAccounts.Next() {
  	serviceAccount := serviceAccounts.Current()
  	fmt.Printf("%s: %s\n", serviceAccount.ID, serviceAccount.Name)
  }
  if err := serviceAccounts.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var serviceAccounts = client.organization().serviceAccounts().list();

  for (var serviceAccount : serviceAccounts.autoPager()) {
      IO.println(serviceAccount.id() + ": " + serviceAccount.name());
  }
  ```

  ```php PHP
  $client = new Client();

  $serviceAccounts = $client->organization->serviceAccounts->list();

  foreach ($serviceAccounts->getItems() as $serviceAccount) {
      echo "{$serviceAccount->id}: {$serviceAccount->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  service_accounts = client.organization.service_accounts.list

  service_accounts.data.each do |service_account|
    puts "#{service_account.id}: #{service_account.name}"
  end
  ```
</CodeGroup>

这些端点不接受 Admin API 密钥，因此 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 页面中的 `x-api-key` 示例不适用于此处。

## 服务账户

[服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#service-accounts)（`svac_...`）是联合令牌所代表的非人类身份。请将 `organization_role` 设置为 `developer`。

创建服务账户：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/service_accounts" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "name": "inference-worker",
      "organization_role": "developer"
    }'
  ```

  ```bash CLI
  ant organization:service-accounts create \
    --name inference-worker \
    --organization-role developer
  ```

  ```python Python
  client = anthropic.Anthropic()

  service_account = client.organization.service_accounts.create(
      name="inference-worker", organization_role="developer"
  )

  print(f"id: {service_account.id}")
  print(f"name: {service_account.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const serviceAccount = await client.organization.serviceAccounts.create({
    name: "inference-worker",
    organization_role: "developer"
  });

  console.log(`id: ${serviceAccount.id}`);
  console.log(`name: ${serviceAccount.name}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.ServiceAccounts;

  AnthropicClient client = new();

  var serviceAccount = await client.Organization.ServiceAccounts.Create(new()
  {
      Name = "inference-worker",
      OrganizationRole = OrganizationRole.Developer
  });

  Console.WriteLine($"id: {serviceAccount.ID}");
  Console.WriteLine($"name: {serviceAccount.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  serviceAccount, err := client.Organization.ServiceAccounts.New(context.Background(), anthropic.OrganizationServiceAccountNewParams{
  	Name:             "inference-worker",
  	OrganizationRole: anthropic.OrganizationServiceAccountNewParamsOrganizationRoleDeveloper,
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", serviceAccount.ID)
  fmt.Printf("name: %s\n", serviceAccount.Name)
  ```

  ```java Java
  import com.anthropic.models.organization.serviceaccounts.ServiceAccountCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = ServiceAccountCreateParams.builder()
          .name("inference-worker")
          .organizationRole(ServiceAccountCreateParams.OrganizationRole.DEVELOPER)
          .build();
      var serviceAccount = client.organization().serviceAccounts().create(params);

      IO.println("id: " + serviceAccount.id());
      IO.println("name: " + serviceAccount.name());
  }
  ```

  ```php PHP
  use Anthropic\Organization\ServiceAccounts\ServiceAccountCreateParams\OrganizationRole;
  // ...

  $client = new Client();

  $serviceAccount = $client->organization->serviceAccounts->create(
      name: 'inference-worker',
      organizationRole: OrganizationRole::DEVELOPER,
  );

  echo "id: {$serviceAccount->id}\n";
  echo "name: {$serviceAccount->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  service_account = client.organization.service_accounts.create(
    name: "inference-worker",
    organization_role: :developer
  )

  puts "id: #{service_account.id}"
  puts "name: #{service_account.name}"
  ```
</CodeGroup>

列出服务账户：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/service_accounts?limit=20" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:service-accounts list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  service_accounts = client.organization.service_accounts.list(limit=20)

  for service_account in service_accounts:
      print(f"{service_account.id}: {service_account.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  for await (const serviceAccount of client.organization.serviceAccounts.list({
    limit: 20
  })) {
    console.log(`${serviceAccount.id}: ${serviceAccount.name}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.ServiceAccounts.List(new() { Limit = 20 });

  await foreach (var serviceAccount in page.Paginate())
  {
      Console.WriteLine($"{serviceAccount.ID}: {serviceAccount.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  serviceAccounts := client.Organization.ServiceAccounts.ListAutoPaging(context.Background(), anthropic.OrganizationServiceAccountListParams{
  	Limit: anthropic.Int(20),
  })

  for serviceAccounts.Next() {
  	serviceAccount := serviceAccounts.Current()
  	fmt.Printf("%s: %s\n", serviceAccount.ID, serviceAccount.Name)
  }
  if err := serviceAccounts.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.serviceaccounts.ServiceAccountListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = ServiceAccountListParams.builder()
          .limit(20)
          .build();
      var serviceAccounts = client.organization().serviceAccounts().list(params);

      for (var serviceAccount : serviceAccounts.autoPager()) {
          IO.println(serviceAccount.id() + ": " + serviceAccount.name());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $serviceAccounts = $client->organization->serviceAccounts->list(limit: 20);

  foreach ($serviceAccounts->getItems() as $serviceAccount) {
      echo "{$serviceAccount->id}: {$serviceAccount->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  service_accounts = client.organization.service_accounts.list(limit: 20)

  service_accounts.data.each do |service_account|
    puts "#{service_account.id}: #{service_account.name}"
  end
  ```
</CodeGroup>

归档服务账户：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS -X POST "https://api.anthropic.com/v1/organizations/service_accounts/svac_01ABCDEFabcdef0123456789XY/archive" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:service-accounts archive svac_01ABCDEFabcdef0123456789XY
  ```

  ```python Python
  client = anthropic.Anthropic()

  service_account = client.organization.service_accounts.archive(
      "svac_01ABCDEFabcdef0123456789XY"
  )

  print(f"id: {service_account.id}")
  print(f"archived_at: {service_account.archived_at}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const serviceAccount = await client.organization.serviceAccounts.archive(
    "svac_01ABCDEFabcdef0123456789XY"
  );

  console.log(`id: ${serviceAccount.id}`);
  console.log(`archived_at: ${serviceAccount.archived_at}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var serviceAccount = await client.Organization.ServiceAccounts.Archive(
      "svac_01ABCDEFabcdef0123456789XY"
  );

  Console.WriteLine($"id: {serviceAccount.ID}");
  Console.WriteLine($"archived_at: {serviceAccount.ArchivedAt:O}");
  ```

  ```go Go
  client := anthropic.NewClient()

  serviceAccount, err := client.Organization.ServiceAccounts.Archive(
  	context.Background(),
  	"svac_01ABCDEFabcdef0123456789XY",
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", serviceAccount.ID)
  fmt.Printf("archived_at: %s\n", serviceAccount.ArchivedAt)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var serviceAccount = client.organization().serviceAccounts()
      .archive("svac_01ABCDEFabcdef0123456789XY");

  IO.println("id: " + serviceAccount.id());
  IO.println("archived_at: " + serviceAccount.archivedAt().orElseThrow());
  ```

  ```php PHP
  $client = new Client();

  $serviceAccount = $client->organization->serviceAccounts->archive(
      serviceAccountID: 'svac_01ABCDEFabcdef0123456789XY',
  );

  echo "id: {$serviceAccount->id}\n";
  echo "archived_at: {$serviceAccount->archivedAt?->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  service_account_id = "svac_01ABCDEFabcdef0123456789XY"
  service_account = client.organization.service_accounts.archive(service_account_id)

  puts "id: #{service_account.id}"
  puts "archived_at: #{service_account.archived_at}"
  ```
</CodeGroup>

创建端点会返回新的服务账户：

```json
{
  "id": "svac_...",
  "name": "inference-worker",
  "organization_role": "developer",
  "created_at": "...",
  "type": "service_account",
  "...": "..."
}
```

要读取或更新单个服务账户，请对 `/v1/organizations/service_accounts/{service_account_id}` 使用 `GET` 和 `POST`。服务账户必须是某个工作区的成员，联合令牌才能在该工作区中执行操作。每个服务账户都隐式属于您组织的默认工作区。要管理在其他工作区中的显式成员资格，请对 `/v1/organizations/service_accounts/{service_account_id}/workspaces` 使用 `GET`、`POST` 和 `DELETE`，其中 `DELETE` 的目标是 `.../workspaces/{workspace_id}`。

有关完整的参数详细信息和响应架构，请参阅[服务账户 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/service_accounts)。

## 联合颁发者

[联合颁发者](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#federation-issuers)（`fdis_...`）用于向您的组织注册 OIDC 身份提供商。`jwks` 字段是一个可辨识联合类型，用于控制 Anthropic 如何获取提供商的签名密钥：

| `jwks` 值                                 | 使用场景                                                 |
| ---------------------------------------- | ---------------------------------------------------- |
| `{"type": "discovery"}`                  | 提供商在颁发者 URL 处提供 `/.well-known/openid-configuration`。 |
| `{"type": "explicit_url", "url": "..."}` | 直接指向 JWKS 端点。                                        |
| `{"type": "inline", "keys": [...]}`      | 为无法从公共互联网访问的提供商上传密钥集。                                |

注册颁发者。此示例使用 JWKS 发现注册 GitHub Actions：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/federation_issuers" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "name": "github-actions",
      "issuer_url": "https://token.actions.githubusercontent.com",
      "jwks": {"type": "discovery"}
    }'
  ```

  ```bash CLI
  ant organization:federation:issuers create \
    --name github-actions \
    --issuer-url https://token.actions.githubusercontent.com \
    --jwks '{type: discovery}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  issuer = client.organization.federation.issuers.create(
      name="github-actions",
      issuer_url="https://token.actions.githubusercontent.com",
      jwks={"type": "discovery"},
  )

  print(f"id: {issuer.id}")
  print(f"name: {issuer.name}")
  print(f"issuer_url: {issuer.issuer_url}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const issuer = await client.organization.federation.issuers.create({
    name: "github-actions",
    issuer_url: "https://token.actions.githubusercontent.com",
    jwks: { type: "discovery" }
  });

  console.log(`id: ${issuer.id}`);
  console.log(`name: ${issuer.name}`);
  console.log(`issuer_url: ${issuer.issuer_url}`);
  ```

  ```csharp C#
  using Anthropic.Models.Organization.Federation.Issuers;

  AnthropicClient client = new();

  var issuer = await client.Organization.Federation.Issuers.Create(new()
  {
      Name = "github-actions",
      IssuerUrl = "https://token.actions.githubusercontent.com",
      Jwks = new JwksDiscovery()
  });

  Console.WriteLine($"id: {issuer.ID}");
  Console.WriteLine($"name: {issuer.Name}");
  Console.WriteLine($"issuer_url: {issuer.IssuerUrl}");
  ```

  ```go Go
  client := anthropic.NewClient()

  issuer, err := client.Organization.Federation.Issuers.New(context.Background(), anthropic.OrganizationFederationIssuerNewParams{
  	Name:      "github-actions",
  	IssuerURL: "https://token.actions.githubusercontent.com",
  	JWKS: anthropic.OrganizationFederationIssuerNewParamsJWKSUnion{
  		OfDiscovery: &anthropic.JWKSDiscoveryParam{},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", issuer.ID)
  fmt.Printf("name: %s\n", issuer.Name)
  fmt.Printf("issuer_url: %s\n", issuer.IssuerURL)
  ```

  ```java Java
  import com.anthropic.models.organization.federation.issuers.JwksDiscovery;
  import com.anthropic.models.organization.federation.issuers.IssuerCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = IssuerCreateParams.builder()
          .name("github-actions")
          .issuerUrl("https://token.actions.githubusercontent.com")
          .jwks(JwksDiscovery.builder().build())
          .build();
      var issuer = client.organization().federation().issuers().create(params);

      IO.println("id: " + issuer.id());
      IO.println("name: " + issuer.name());
      IO.println("issuer_url: " + issuer.issuerUrl());
  }
  ```

  ```php PHP
  $client = new Client();

  $issuer = $client->organization->federation->issuers->create(
      name: 'github-actions',
      issuerURL: 'https://token.actions.githubusercontent.com',
      jwks: ['type' => 'discovery'],
  );

  echo "id: {$issuer->id}\n";
  echo "name: {$issuer->name}\n";
  echo "issuer_url: {$issuer->issuerURL}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  issuer = client.organization.federation.issuers.create(
    name: "github-actions",
    issuer_url: "https://token.actions.githubusercontent.com",
    jwks: {type: :discovery}
  )

  puts "id: #{issuer.id}"
  puts "name: #{issuer.name}"
  puts "issuer_url: #{issuer.issuer_url}"
  ```
</CodeGroup>

列出颁发者：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/federation_issuers?limit=20" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:federation:issuers list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  issuers = client.organization.federation.issuers.list(limit=20)

  for issuer in issuers:
      print(f"{issuer.id}: {issuer.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  for await (const issuer of client.organization.federation.issuers.list({ limit: 20 })) {
    console.log(`${issuer.id}: ${issuer.name}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Federation.Issuers.List(new() { Limit = 20 });

  await foreach (var issuer in page.Paginate())
  {
      Console.WriteLine($"{issuer.ID}: {issuer.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  issuers := client.Organization.Federation.Issuers.ListAutoPaging(context.Background(), anthropic.OrganizationFederationIssuerListParams{
  	Limit: anthropic.Int(20),
  })

  for issuers.Next() {
  	issuer := issuers.Current()
  	fmt.Printf("%s: %s\n", issuer.ID, issuer.Name)
  }
  if err := issuers.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.federation.issuers.IssuerListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = IssuerListParams.builder()
          .limit(20)
          .build();
      var issuers = client.organization().federation().issuers().list(params);

      for (var issuer : issuers.autoPager()) {
          IO.println(issuer.id() + ": " + issuer.name());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $issuers = $client->organization->federation->issuers->list(limit: 20);

  foreach ($issuers->getItems() as $issuer) {
      echo "{$issuer->id}: {$issuer->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  issuers = client.organization.federation.issuers.list(limit: 20)

  issuers.data.each do |issuer|
    puts "#{issuer.id}: #{issuer.name}"
  end
  ```
</CodeGroup>

归档颁发者：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS -X POST "https://api.anthropic.com/v1/organizations/federation_issuers/fdis_01ABCDEFabcdef0123456789XY/archive" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:federation:issuers archive \
    --federation-issuer-id fdis_01ABCDEFabcdef0123456789XY
  ```

  ```python Python
  client = anthropic.Anthropic()

  issuer = client.organization.federation.issuers.archive(
      "fdis_01ABCDEFabcdef0123456789XY"
  )

  print(f"id: {issuer.id}")
  print(f"archived_at: {issuer.archived_at}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const issuer = await client.organization.federation.issuers.archive(
    "fdis_01ABCDEFabcdef0123456789XY"
  );

  console.log(`id: ${issuer.id}`);
  console.log(`archived_at: ${issuer.archived_at}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var issuer = await client.Organization.Federation.Issuers.Archive(
      "fdis_01ABCDEFabcdef0123456789XY"
  );

  Console.WriteLine($"id: {issuer.ID}");
  Console.WriteLine($"archived_at: {issuer.ArchivedAt:O}");
  ```

  ```go Go
  client := anthropic.NewClient()

  issuer, err := client.Organization.Federation.Issuers.Archive(
  	context.Background(),
  	"fdis_01ABCDEFabcdef0123456789XY",
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", issuer.ID)
  fmt.Printf("archived_at: %s\n", issuer.ArchivedAt)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var issuer = client.organization().federation().issuers()
      .archive("fdis_01ABCDEFabcdef0123456789XY");

  IO.println("id: " + issuer.id());
  IO.println("archived_at: " + issuer.archivedAt().orElseThrow());
  ```

  ```php PHP
  $client = new Client();

  $issuer = $client->organization->federation->issuers->archive(
      federationIssuerID: 'fdis_01ABCDEFabcdef0123456789XY',
  );

  echo "id: {$issuer->id}\n";
  echo "archived_at: {$issuer->archivedAt?->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  issuer_id = "fdis_01ABCDEFabcdef0123456789XY"
  issuer = client.organization.federation.issuers.archive(issuer_id)

  puts "id: #{issuer.id}"
  puts "archived_at: #{issuer.archived_at}"
  ```
</CodeGroup>

要读取或更新单个颁发者，请对 `/v1/organizations/federation_issuers/{issuer_id}` 使用 `GET` 和 `POST`。如果某个颁发者支撑着 `oauth_scope` 不是 `workspace:developer` 或 `workspace:inference` 的规则，OAuth 调用方将无法更新该颁发者。请参阅[权限和约束](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#permissions-and-constraints)。

有关完整的参数详细信息和响应架构，请参阅[联合颁发者 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/federation/issuers)。

## 联合规则

[联合规则](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#federation-rules)（`fdrl_...`）将颁发者绑定到服务账户。来自该颁发者且满足规则匹配条件的 JWT 可以生成以规则目标身份执行操作的令牌。创建请求中的 `workspace_id` 会在创建时在该工作区中启用规则。之后可以通过 `/federation_rules/{rule_id}/workspaces` 子资源添加更多工作区。创建请求必须提供 `workspace_id` 或 `applies_to_all_workspaces: true` 之一。

创建规则。此示例允许来自 main 分支的 GitHub Actions 部署以该服务账户身份执行操作：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/federation_rules" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "name": "gha-deploy",
      "issuer_id": "fdis_01ABCDEFabcdef0123456789XY",
      "match": {
        "subject_prefix": "repo:my-org/my-repo:ref:refs/heads/main",
        "claims": {"repository_owner": "my-org"}
      },
      "target": {
        "type": "service_account",
        "service_account_id": "svac_01ABCDEFabcdef0123456789XY"
      },
      "workspace_id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      "oauth_scope": "workspace:developer",
      "token_lifetime_seconds": 600
    }'
  ```

  ```bash CLI
  ant organization:federation:rules create <<'YAML'
  name: gha-deploy
  issuer_id: fdis_01ABCDEFabcdef0123456789XY
  match:
    subject_prefix: "repo:my-org/my-repo:ref:refs/heads/main"
    claims:
      repository_owner: my-org
  target:
    type: service_account
    service_account_id: svac_01ABCDEFabcdef0123456789XY
  workspace_id: wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ
  oauth_scope: "workspace:developer"
  token_lifetime_seconds: 600
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  rule = client.organization.federation.rules.create(
      name="gha-deploy",
      issuer_id="fdis_01ABCDEFabcdef0123456789XY",
      match={
          "subject_prefix": "repo:my-org/my-repo:ref:refs/heads/main",
          "claims": {"repository_owner": "my-org"},
      },
      target={
          "type": "service_account",
          "service_account_id": "svac_01ABCDEFabcdef0123456789XY",
      },
      workspace_id="wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      oauth_scope="workspace:developer",
      token_lifetime_seconds=600,
  )

  print(f"id: {rule.id}")
  print(f"name: {rule.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rule = await client.organization.federation.rules.create({
    name: "gha-deploy",
    issuer_id: "fdis_01ABCDEFabcdef0123456789XY",
    match: {
      subject_prefix: "repo:my-org/my-repo:ref:refs/heads/main",
      claims: { repository_owner: "my-org" }
    },
    target: {
      type: "service_account",
      service_account_id: "svac_01ABCDEFabcdef0123456789XY"
    },
    workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    oauth_scope: "workspace:developer",
    token_lifetime_seconds: 600
  });

  console.log(`id: ${rule.id}`);
  console.log(`name: ${rule.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var rule = await client.Organization.Federation.Rules.Create(new()
  {
      Name = "gha-deploy",
      IssuerID = "fdis_01ABCDEFabcdef0123456789XY",
      Match = new()
      {
          SubjectPrefix = "repo:my-org/my-repo:ref:refs/heads/main",
          Claims = new Dictionary<string, string> { ["repository_owner"] = "my-org" }
      },
      Target = new() { ServiceAccountID = "svac_01ABCDEFabcdef0123456789XY" },
      WorkspaceID = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      OAuthScope = "workspace:developer",
      TokenLifetimeSeconds = 600
  });

  Console.WriteLine($"id: {rule.ID}");
  Console.WriteLine($"name: {rule.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  rule, err := client.Organization.Federation.Rules.New(context.Background(), anthropic.OrganizationFederationRuleNewParams{
  	Name:     "gha-deploy",
  	IssuerID: "fdis_01ABCDEFabcdef0123456789XY",
  	Match: anthropic.FederationRuleMatchParam{
  		SubjectPrefix: anthropic.String("repo:my-org/my-repo:ref:refs/heads/main"),
  		Claims:        map[string]string{"repository_owner": "my-org"},
  	},
  	Target: anthropic.ServiceAccountTargetParam{
  		ServiceAccountID: "svac_01ABCDEFabcdef0123456789XY",
  	},
  	WorkspaceID:          anthropic.String("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"),
  	OAuthScope:           "workspace:developer",
  	TokenLifetimeSeconds: anthropic.Int(600),
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", rule.ID)
  fmt.Printf("name: %s\n", rule.Name)
  ```

  ```java Java
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.organization.federation.rules.FederationRuleMatch;
  import com.anthropic.models.organization.federation.rules.ServiceAccountTarget;
  import com.anthropic.models.organization.federation.rules.RuleCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var match = FederationRuleMatch.builder()
          .subjectPrefix("repo:my-org/my-repo:ref:refs/heads/main")
          .claims(FederationRuleMatch.Claims.builder()
              .putAdditionalProperty("repository_owner", JsonValue.from("my-org"))
              .build())
          .build();
      var params = RuleCreateParams.builder()
          .name("gha-deploy")
          .issuerId("fdis_01ABCDEFabcdef0123456789XY")
          .match(match)
          .target(ServiceAccountTarget.of("svac_01ABCDEFabcdef0123456789XY"))
          .workspaceId("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
          .oauthScope("workspace:developer")
          .tokenLifetimeSeconds(600)
          .build();
      var rule = client.organization().federation().rules().create(params);

      IO.println("id: " + rule.id());
      IO.println("name: " + rule.name());
  }
  ```

  ```php PHP
  $client = new Client();

  $rule = $client->organization->federation->rules->create(
      name: 'gha-deploy',
      issuerID: 'fdis_01ABCDEFabcdef0123456789XY',
      match: [
          'subjectPrefix' => 'repo:my-org/my-repo:ref:refs/heads/main',
          'claims' => ['repository_owner' => 'my-org'],
      ],
      target: [
          'type' => 'service_account',
          'serviceAccountID' => 'svac_01ABCDEFabcdef0123456789XY',
      ],
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
      oauthScope: 'workspace:developer',
      tokenLifetimeSeconds: 600,
  );

  echo "id: {$rule->id}\n";
  echo "name: {$rule->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rule = client.organization.federation.rules.create(
    name: "gha-deploy",
    issuer_id: "fdis_01ABCDEFabcdef0123456789XY",
    match: {
      subject_prefix: "repo:my-org/my-repo:ref:refs/heads/main",
      claims: {repository_owner: "my-org"}
    },
    target: {
      type: :service_account,
      service_account_id: "svac_01ABCDEFabcdef0123456789XY"
    },
    workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    oauth_scope: "workspace:developer",
    token_lifetime_seconds: 600
  )

  puts "id: #{rule.id}"
  puts "name: #{rule.name}"
  ```
</CodeGroup>

列出规则，可选择按颁发者筛选：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/federation_rules?issuer_id=fdis_01ABCDEFabcdef0123456789XY" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:federation:rules list \
    --issuer-id fdis_01ABCDEFabcdef0123456789XY
  ```

  ```python Python
  client = anthropic.Anthropic()

  rules = client.organization.federation.rules.list(
      issuer_id="fdis_01ABCDEFabcdef0123456789XY"
  )

  for rule in rules:
      print(f"{rule.id}: {rule.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  for await (const rule of client.organization.federation.rules.list({
    issuer_id: "fdis_01ABCDEFabcdef0123456789XY"
  })) {
    console.log(`${rule.id}: ${rule.name}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Organization.Federation.Rules.List(new()
  {
      IssuerID = "fdis_01ABCDEFabcdef0123456789XY"
  });

  await foreach (var rule in page.Paginate())
  {
      Console.WriteLine($"{rule.ID}: {rule.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rules := client.Organization.Federation.Rules.ListAutoPaging(context.Background(), anthropic.OrganizationFederationRuleListParams{
  	IssuerID: anthropic.String("fdis_01ABCDEFabcdef0123456789XY"),
  })

  for rules.Next() {
  	rule := rules.Current()
  	fmt.Printf("%s: %s\n", rule.ID, rule.Name)
  }
  if err := rules.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.federation.rules.RuleListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RuleListParams.builder()
          .issuerId("fdis_01ABCDEFabcdef0123456789XY")
          .build();
      var rules = client.organization().federation().rules().list(params);

      for (var rule : rules.autoPager()) {
          IO.println(rule.id() + ": " + rule.name());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rules = $client->organization->federation->rules->list(
      issuerID: 'fdis_01ABCDEFabcdef0123456789XY',
  );

  foreach ($rules->getItems() as $rule) {
      echo "{$rule->id}: {$rule->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rules = client.organization.federation.rules.list(
    issuer_id: "fdis_01ABCDEFabcdef0123456789XY"
  )

  rules.data.each do |rule|
    puts "#{rule.id}: #{rule.name}"
  end
  ```
</CodeGroup>

归档规则：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS -X POST "https://api.anthropic.com/v1/organizations/federation_rules/fdrl_01ABCDEFabcdef0123456789XY/archive" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant organization:federation:rules archive \
    --federation-rule-id fdrl_01ABCDEFabcdef0123456789XY
  ```

  ```python Python
  client = anthropic.Anthropic()

  rule = client.organization.federation.rules.archive("fdrl_01ABCDEFabcdef0123456789XY")

  print(f"id: {rule.id}")
  print(f"archived_at: {rule.archived_at}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rule = await client.organization.federation.rules.archive(
    "fdrl_01ABCDEFabcdef0123456789XY"
  );

  console.log(`id: ${rule.id}`);
  console.log(`archived_at: ${rule.archived_at}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var rule = await client.Organization.Federation.Rules.Archive(
      "fdrl_01ABCDEFabcdef0123456789XY"
  );

  Console.WriteLine($"id: {rule.ID}");
  Console.WriteLine($"archived_at: {rule.ArchivedAt:O}");
  ```

  ```go Go
  client := anthropic.NewClient()

  rule, err := client.Organization.Federation.Rules.Archive(
  	context.Background(),
  	"fdrl_01ABCDEFabcdef0123456789XY",
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", rule.ID)
  fmt.Printf("archived_at: %s\n", rule.ArchivedAt)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var rule = client.organization().federation().rules()
      .archive("fdrl_01ABCDEFabcdef0123456789XY");

  IO.println("id: " + rule.id());
  IO.println("archived_at: " + rule.archivedAt().orElseThrow());
  ```

  ```php PHP
  $client = new Client();

  $rule = $client->organization->federation->rules->archive(
      federationRuleID: 'fdrl_01ABCDEFabcdef0123456789XY',
  );

  echo "id: {$rule->id}\n";
  echo "archived_at: {$rule->archivedAt?->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rule_id = "fdrl_01ABCDEFabcdef0123456789XY"
  rule = client.organization.federation.rules.archive(rule_id)

  puts "id: #{rule.id}"
  puts "archived_at: #{rule.archived_at}"
  ```
</CodeGroup>

列表端点会返回一页规则以及下一页的游标：

```json
{
  "data": [{ "id": "fdrl_...", "name": "gha-deploy", "...": "..." }],
  "next_page": "..."
}
```

要读取或更新单个规则，请对 `/v1/organizations/federation_rules/{rule_id}` 使用 `GET` 和 `POST`。要管理规则可以在其中生成令牌的工作区，请对 `/v1/organizations/federation_rules/{rule_id}/workspaces` 使用 `GET` 和 `POST`，并对 `/v1/organizations/federation_rules/{rule_id}/workspaces/{workspace_id}` 使用 `DELETE`。

有关完整的参数详细信息和响应架构，请参阅[联合规则 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/federation/rules)。

## 权限和约束

<Note>
  * 通过 OAuth 身份验证的调用方只能创建或修改 `oauth_scope` 为 `workspace:developer` 或 `workspace:inference` 的规则。要创建或修改具有任何其他范围（例如 `org:admin` 或 `workspace:manage_tunnels`）的规则，请使用 Console。
  * 如果某个联合颁发者支撑着 `oauth_scope` 不是 `workspace:developer` 或 `workspace:inference`（例如 `org:admin` 或 `workspace:manage_tunnels`）的规则，OAuth 调用方将无法更新该颁发者。建议为引导规则注册一个专用颁发者，以便工作区范围规则背后的颁发者仍可通过 API 更新。
  * 这些端点不接受 Admin API 密钥，无论是读取还是写入。请使用 `org:admin` OAuth 令牌。
  * 联合令牌无法在新的或现有的服务账户上将 `organization_role` 设置为 `admin`，即使该服务账户已经是 `admin`。要设置该角色，请使用[交互式令牌](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#interactive-your-terminal)或 Console。
</Note>

`oauth_scope: org:admin` 的规则必须以 `organization_role` 为 `admin` 的服务账户为目标。资源名称必须匹配 `^[a-z0-9-]+$`，长度为 1 到 255 个字符，并且对于每种资源类型在组织内必须唯一。有关完整的字段级约束，请参阅[验证规则](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#validation-rules)。

## 分页和归档

服务账户、联合颁发者和联合规则的列表端点接受 `limit`（页面大小，1 到 100，默认为 20）以及取自上一个响应的 `page` 游标。在下一个请求中，将响应的 `next_page` 值作为 `page` 查询参数传递。当您使用 `for` (typescript: `for await`; csharp: `Paginate()`; go: `ListAutoPaging()`; java: `autoPager()`; php: `pagingEachItem()`; ruby: `auto_paging_each`) 进行迭代时，SDK 会按需获取后续页面，而 `ant` CLI 默认就会这样做。规则工作区子资源列表会返回完整集合，不进行分页。已归档的资源默认不会出现在列表中。传递 `include_archived=true` 可将其包含在内。

归档是一种软删除。它是幂等的，因此归档已归档的资源会成功。当有有效的联合规则引用某个颁发者或服务账户时，归档该颁发者或服务账户会返回 `400`。请先归档该规则。

## 另请参阅

* [工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)：概念以及 Console 设置演练
* [WIF 参考](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference)：环境变量、验证规则、OAuth 范围和错误代码
* [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)：组织管理功能的其余部分
* [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/organization)：服务账户和联合端点的生成式请求和响应架构
