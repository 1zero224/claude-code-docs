---
title: 工作负载身份联合
url: https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation
description: 使用来自您自己的身份提供商的短期身份令牌（而非长期静态 API 密钥）向 Claude API 验证工作负载的身份。
---

“Workload Identity Federation”（工作负载身份联合），即 WIF，让您的工作负载可以使用短期的 “OpenID Connect”（开放身份连接），即 OIDC 令牌向 Claude API 进行身份验证，而无需使用长期的 `sk-ant-...` API 密钥。这些令牌来自您已经在运营的 “identity provider”（身份提供商），即 IdP：AWS IAM、Google Cloud，或任何符合标准的 OIDC 颁发者，例如 GitHub Actions、Kubernetes、SPIFFE、Microsoft Entra ID 或 Okta。

您的工作负载出示一个由您的身份提供商签名的 JWT。Anthropic 根据您在 Claude Console 中配置的信任规则对其进行验证，并返回一个短期的 Anthropic 访问令牌，该令牌绑定到您组织中的某个服务账户。无需生成、存储在 CI 中、轮换或担心泄露任何静态密钥。

工作负载身份联合通过将静态 API 密钥替换为几分钟内即过期（而非永不过期）的令牌，来增强您的安全态势。但它本身并不是一个完整的安全解决方案。联合身份验证的强度仅取决于签署 JWT 的上游身份提供商。请将工作负载身份联合与您的 IdP 已支持的控制措施（工作负载身份绑定、条件访问、审计日志）结合使用，以实现纵深防御。

## 概念

在任何工作负载能够进行联合之前，您需要在 Claude Console 中配置三种资源。它们共同表达"由颁发者 X 签名、声明形如 Y 的令牌，可以作为服务账户 Z 行事"。

### 服务账户

“**service account**”（服务账户）（`svac_...`）是您的 Anthropic 组织内一个具名的非人类身份。它是[服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)或联合令牌所代表的主体。服务账户存在于组织级别，当您将其添加为某个工作区的成员时，它便在该工作区中生效。在交换时，Anthropic 会检查联合规则的工作区是否与该服务账户的某个工作区成员资格相匹配。随后生成的令牌遵循该工作区的速率限制和用量归属，与 API 密钥相同。与人类用户不同，服务账户没有电子邮件、没有密码，也无法登录 Console。每个服务账户都隐式地是您组织默认工作区的成员。对于它应当在其中行事的任何其他工作区，请添加显式成员资格。若要让一个全工作区范围的服务账户密钥在某个工作区中行事，请将该服务账户添加到该工作区。

工作区 API 密钥*本身就是*凭据，而服务账户*拥有*凭据。由于每个工作负载都以一个具名服务账户的身份进行身份验证，您可以更轻松地审计哪些工作负载以哪个服务账户的身份执行了操作。

### 联合颁发者

“**federation issuer**”（联合颁发者）（`fdis_...`）将一个 OIDC 身份提供商注册到您的组织。注册颁发者即告知 Anthropic："由该提供商签名的 JWT 可以为我的组织断言工作负载身份。"

一个颁发者有两项配置：

* **颁发者 URL：** 出现在该提供商 JWT 中的确切 `iss` 声明值，例如 `https://token.actions.githubusercontent.com` 或 `https://oidc.eks.us-west-2.amazonaws.com/id/EXAMPLE`。
* **JWKS 来源：** Anthropic 获取用于验证 JWT 签名的公钥的方式。对于任何在其颁发者 URL 下提供 `/.well-known/openid-configuration` 的提供商，请使用 `discovery`（默认值）。使用 `explicit_url` 可直接指向某个 JWKS 端点。使用 `inline` 可为无法从公共互联网访问的颁发者（例如私有 Kubernetes 集群）上传密钥集。

颁发者 URL 和 JWKS URL 必须为 `https`、使用 443 端口，并使用可解析为公共 IP 地址的公共 DNS 主机名。Anthropic 不接受 IP 字面量。这些约束仅适用于 Anthropic 需要获取的 URL。在 `explicit_url` 和 `inline` 模式下，Anthropic 将 `issuer_url` 作为字符串进行比较，该 URL 可以引用内部主机名。

通常您会为每个环境注册一个颁发者：您的生产 EKS 集群、预发布集群和 GitHub Actions 是三个独立的颁发者。

### 联合规则

“**federation rule**”（联合规则）（`fdrl_...`）将颁发者绑定到服务账户："当来自颁发者 X 的 JWT 具有形如 Y 的声明时，为服务账户 Z 生成一个作用域为 S 的令牌。"

一条规则定义匹配条件、目标，以及规则匹配时适用的授权作用域和令牌生命周期：

* **匹配：** 传入的 JWT 必须满足的条件。您可以按 `subject_prefix`、精确的 `audience`、精确声明值的映射、用于复杂逻辑的 [“Common Expression Language”（通用表达式语言），即 CEL](https://cel.dev/) `condition` 表达式，或它们的任意组合进行匹配。`subject_prefix` 可以是精确匹配，例如 `system:serviceaccount:prod:worker`，也可以是以结尾的 `*` 进行的前缀匹配。请至少设置 `subject_prefix`、`claims` 或 `condition` 中的一项。所有已配置的匹配器都必须通过，Anthropic 才会接受该 JWT。
* **目标：** 匹配的 JWT 所映射到的服务账户。
* **授权：** 在生成的令牌上授予的 OAuth `scope`。默认值为 `workspace:developer`，它授予与工作区 API 密钥相同的访问权限。某些产品在您从其流程中创建规则时会锁定作用域。例如，[MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)的创建隧道对话框会创建作用域为 `workspace:manage_tunnels` 的规则。请参阅 [OAuth 范围](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#oauth-scopes)。规则还会设置 `token_lifetime_seconds`（60 到 86,400，默认 3,600）。

单个颁发者可以有多条规则：每个团队、命名空间或权限级别各一条。Anthropic 按 ID 评估规则。客户端在交换请求中指定要使用的规则，Anthropic 验证 JWT 是否满足该规则的匹配条件。不存在隐式的规则搜索。

## 工作原理

1. **您的 IdP 向工作负载颁发 JWT。** 在大多数平台上，工作负载已可通过 Kubernetes 投射的服务账户令牌、Google Cloud 元数据服务器、"Azure Instance Metadata Service"（Azure 实例元数据服务），即 IMDS，或 GitHub Actions OIDC 端点获取 JWT。JWT 的 `iss` 声明标识提供商，其 `sub` 及其他声明标识具体的工作负载。
2. **SDK 将 JWT 交换为 Anthropic 访问令牌。** SDK 使用 [RFC 7523](https://www.rfc-editor.org/rfc/rfc7523) `jwt-bearer` 授权类型将 JWT 发送到 `POST /v1/oauth/token`。Anthropic 根据颁发者的 JWKS 和联合规则的匹配条件验证 JWT。然后它返回一个短期的 `sk-ant-oat01-...` 令牌，该令牌代表规则的目标服务账户行事。
3. **SDK 在每个请求中发送该令牌，并在其过期前刷新。** 您的应用程序代码在不提供 API 密钥的情况下构造客户端，并照常调用 API。SDK 会在令牌过期前重新执行交换。

## 设置联合

您需要在 Anthropic 组织中拥有管理员、所有者或主要所有者角色。您还需要一个支持 OIDC 且具有可访问 JWKS 端点的身份提供商，或者一份可为物理隔离集群粘贴的 JWKS 文档。您的工作负载必须能够从该提供商获取身份令牌。

**连接工作负载**向导在一个引导式流程中创建全部三种资源（颁发者、服务账户和联合规则），然后端到端地验证连接。

<Steps>
  <Step title="打开“连接工作负载”">
    在 Claude Console 中，前往[工作负载身份](https://platform.claude.com/settings/workload-identity-federation)页面（`platform.claude.com/settings/workload-identity-federation`）并选择**连接工作负载**。
  </Step>

  <Step title="选择您的提供商">
    选择您的身份提供商对应的磁贴：GitHub Actions、AWS、Google Cloud、Microsoft Entra ID 或 Kubernetes。每个磁贴都会预填颁发者 URL 模式以及该提供商 JWT 所支持的匹配字段。对于任何其他符合标准的提供商（例如 SPIFFE 或 Okta），请选择 **Custom OIDC**。
  </Step>

  <Step title="填写引导字段">
    向导会引导您填写提供商特定的字段：颁发者配置、传入 JWT 的匹配条件，以及它所创建的服务账户和联合规则的名称。向导会预填 `oauth_scope=workspace:developer` 和 `token_lifetime_seconds=600`。如果您的工作负载需要不同的作用域或生命周期，请进行调整。当请求省略 `token_lifetime_seconds` 时，API 的默认值为 3,600 秒。
  </Step>

  <Step title="验证颁发者">
    可选择 **Verify issuer（验证颁发者）**，在向导创建任何内容之前对颁发者配置进行试运行。验证会确认 Anthropic 能够从您输入的 URL 获取并解析 JWKS，从而及早发现可达性和配置错误。
  </Step>

  <Step title="测试连接">
    向导会创建颁发者、服务账户和联合规则，然后在 15 分钟内监听一次成功的令牌交换。请在该时间窗口内从您的工作负载触发一次交换（参见[从您的工作负载进行身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#authenticate-from-your-workload)）以确认设置有效。如果时间窗口已过，资源仍会保留。您可以从联合规则的详情页重新运行测试。请记下向导创建的规则 ID（`fdrl_...`）和服务账户 ID（`svac_...`）。您的工作负载在每个令牌交换请求中都会传递这两者，以及您的组织 ID，当规则覆盖多个工作区时还需传递您的工作区 ID。
  </Step>
</Steps>

要以编程方式管理这些资源，请参阅[使用 Admin API 管理 WIF](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api)。有关完整的参数详情和响应模式，请参阅 [服务账户 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/service_accounts)、[联合颁发者 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/federation/issuers) 和 [联合规则 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/federation/rules)。

## 从您的工作负载进行身份验证

配置好联合后，您的工作负载会在运行时将其 IdP 颁发的 JWT 交换为 Anthropic 令牌。SDK 会为您处理交换和刷新循环。如需调试，或使用没有 SDK 支持的语言，[令牌交换请求](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#token-exchange-request)列出了底层 HTTP 请求的各个字段。

### 构造 SDK 客户端

您可以使用显式凭据或不带任何参数来构造客户端。不带参数时，SDK 会从环境变量或活动配置文件中解析凭据，如[凭据优先级](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#credential-precedence)中所述。零参数形式是生产工作负载的推荐模式。在所有地方部署相同的容器镜像，并为每个环境注入 `ANTHROPIC_FEDERATION_RULE_ID`、`ANTHROPIC_ORGANIZATION_ID`、`ANTHROPIC_SERVICE_ACCOUNT_ID`、`ANTHROPIC_WORKSPACE_ID` 和 `ANTHROPIC_IDENTITY_TOKEN_FILE`。

<CodeGroup>
  ```bash cURL
  # 1. 获取您的 IdP 签发的 JWT（因平台而异；请参阅各提供商的指南）。
  JWT=$(cat /var/run/secrets/anthropic.com/token)

  # 2. 用它换取短期有效的 Anthropic 访问令牌。
  ACCESS_TOKEN=$(curl -sS https://api.anthropic.com/v1/oauth/token \
    -H "content-type: application/json" \
    -d @- <<JSON | jq -r .access_token
  {
    "grant_type": "urn:ietf:params:oauth:grant-type:jwt-bearer",
    "assertion": "$JWT",
    "federation_rule_id": "fdrl_...",
    "organization_id": "00000000-0000-0000-0000-000000000000",
    "service_account_id": "svac_...",
    "workspace_id": "wrkspc_..."
  }
  JSON
  )
  # 响应中的 expires_in 是令牌的有效期（以秒为单位）；请在其过期前重新交换。

  # 3. 在 Authorization: Bearer 标头中携带访问令牌调用 API。
  curl -sS https://api.anthropic.com/v1/messages \
    -H "authorization: Bearer $ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}]
    }'
  ```

  ```bash CLI
  # CLI 会自行将身份令牌交换为访问令牌。
  # 对于为所有工作区或多个工作区启用的规则，请将 ANTHROPIC_PROFILE 设置为
  # 一个设置了 workspace_id 的联合配置文件。
  export ANTHROPIC_IDENTITY_TOKEN_FILE=/var/run/secrets/anthropic.com/token
  export ANTHROPIC_FEDERATION_RULE_ID=fdrl_...
  export ANTHROPIC_ORGANIZATION_ID=00000000-0000-0000-0000-000000000000
  export ANTHROPIC_SERVICE_ACCOUNT_ID=svac_...

  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  from anthropic import Anthropic, WorkloadIdentityCredentials, IdentityTokenFile

  client = Anthropic(
      credentials=WorkloadIdentityCredentials(
          identity_token_provider=IdentityTokenFile(
              "/var/run/secrets/anthropic.com/token"
          ),
          federation_rule_id="fdrl_...",
          organization_id="00000000-0000-0000-0000-000000000000",
          service_account_id="svac_...",
          workspace_id="wrkspc_...",
      ),
  )

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  print(next(block.text for block in message.content if block.type == "text"))
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";
  import { oidcFederationProvider } from "@anthropic-ai/sdk/lib/credentials/oidc-federation";
  import { identityTokenFromFile } from "@anthropic-ai/sdk/lib/credentials/identity-token";

  const client = new Anthropic({
    credentials: oidcFederationProvider({
      identityTokenProvider: identityTokenFromFile("/var/run/secrets/anthropic.com/token"),
      federationRuleId: "fdrl_...",
      organizationId: "00000000-0000-0000-0000-000000000000",
      serviceAccountId: "svac_...",
      workspaceId: "wrkspc_...",
      baseURL: "https://api.anthropic.com",
      fetch
    })
  });

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  });
  for (const block of message.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Credentials;
  // ...

  var credentials = new WorkloadIdentityCredentials(new WorkloadIdentityOptions
  {
      FederationRuleId = "fdrl_...",
      OrganizationId = "00000000-0000-0000-0000-000000000000",
      ServiceAccountId = "svac_...",
      WorkspaceId = "wrkspc_...",
      IdentityTokenProvider = new FileIdentityTokenProvider("/var/run/secrets/anthropic.com/token"),
  });
  using var client = new AnthropicClient(new ClientOptions { Credentials = credentials });

  var message = await client.Messages.Create(new()
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }],
  });
  foreach (var block in message.Content)
  {
      if (block.Value is TextBlock textBlock)
      {
          Console.WriteLine(textBlock.Text);
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient(
  	option.WithFederationTokenProvider(
  		option.IdentityTokenFile("/var/run/secrets/anthropic.com/token"),
  		option.FederationOptions{
  			FederationRuleID: "fdrl_...",
  			OrganizationID:   "00000000-0000-0000-0000-000000000000",
  			ServiceAccountID: "svac_...",
  			WorkspaceID:      "wrkspc_...",
  		},
  	),
  )

  message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  for _, block := range message.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		fmt.Println(textBlock.Text)
  		break
  	}
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.config.AuthenticationConfig;
  import com.anthropic.config.AuthenticationType;
  import com.anthropic.config.IdentityTokenConfig;
  import com.anthropic.config.InMemoryProfileConfigProvider;
  import com.anthropic.config.ProfileConfig;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.builder()
          .fromEnv()
          .configurationProvider(InMemoryProfileConfigProvider.of(ProfileConfig.builder()
              .organizationId("00000000-0000-0000-0000-000000000000")
              .workspaceId("wrkspc_...")
              .authentication(AuthenticationConfig.builder()
                  .type(AuthenticationType.OIDC_FEDERATION)
                  .federationRuleId("fdrl_...")
                  .serviceAccountId("svac_...")
                  .identityToken(IdentityTokenConfig.builder()
                      .source("file")
                      .path("/var/run/secrets/anthropic.com/token")
                      .build())
                  .build())
              .build()))
          .build();

      var message = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .build());

      IO.println(message.content());
  }
  ```

  ```php PHP
  use Anthropic\Client;
  use Anthropic\Lib\Credentials\CredentialResult;
  use Anthropic\Lib\Credentials\IdentityTokenFile;
  use Anthropic\Lib\Credentials\TokenCache;
  use Anthropic\Lib\Credentials\WorkloadIdentityCredentials;

  $client = new Client(credentials: new CredentialResult(
      provider: new TokenCache(
          new WorkloadIdentityCredentials(
              identityProvider: new IdentityTokenFile('/var/run/secrets/anthropic.com/token'),
              federationRuleId: 'fdrl_...',
              organizationId: '00000000-0000-0000-0000-000000000000',
              serviceAccountId: 'svac_...',
              workspaceId: 'wrkspc_...',
          ),
      ),
  ));

  $message = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
  );

  $textBlock = array_find($message->content, static fn ($block): bool => $block->type === 'text');
  echo $textBlock->text . PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new(
    credentials: Anthropic::Credentials::WorkloadIdentity.new(
      identity_token_provider: Anthropic::Credentials::IdentityTokenFile.new(
        "/var/run/secrets/anthropic.com/token"
      ),
      federation_rule_id: "fdrl_...",
      organization_id: "00000000-0000-0000-0000-000000000000",
      service_account_id: "svac_...",
      workspace_id: "wrkspc_..."
    )
  )

  message = client.messages.create(
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{role: "user", content: "Hello, Claude"}]
  )

  puts message.content.find { it.type == :text }.text
  ```
</CodeGroup>

令牌交换响应遵循 [RFC 6749 §5.1](https://www.rfc-editor.org/rfc/rfc6749#section-5.1)。字段参考请参阅[令牌交换响应](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#token-exchange-response)。

## 凭据优先级

SDK 按五层顺序解析凭据：构造函数参数，然后是 `ANTHROPIC_API_KEY` / `ANTHROPIC_AUTH_TOKEN`，然后是显式的 `ANTHROPIC_PROFILE`，然后是联合环境变量，最后是隐式的活动配置文件。第一个产生凭据的来源胜出。

<Warning>
  `ANTHROPIC_API_KEY` 位于联合层级之上，因此环境中遗留的密钥会悄无声息地覆盖联合。 将工作负载从 API 密钥迁移到工作负载身份联合时，请确认在该工作负载运行的所有位置 （容器环境、CI 密钥、shell 配置文件）都已取消设置 `ANTHROPIC_API_KEY`。CLI 的 [`ant auth status`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication#check-authentication-status) 命令会报告哪个来源胜出。
</Warning>

有关完整的优先级表、每层的含义以及配置文件的文件模式，请参阅 [WIF 参考中的凭据优先级](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#credential-precedence)。

## 从 API 密钥迁移

若要在不停机的情况下将现有工作负载从静态 API 密钥切换到联合：

1. **并行配置联合。** 完成[设置联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#set-up-federation)中的步骤并确认联合规则与您工作负载的令牌匹配。暂时保留现有的 `ANTHROPIC_API_KEY`。
2. **检查哪个凭据胜出。** 在工作负载内部运行 `ant auth status`（或检查 SDK 调试日志）。由于 `ANTHROPIC_API_KEY` 在优先级链中位于联合层级之上，此阶段 API 密钥仍然胜出。
3. **在所有注入 `ANTHROPIC_API_KEY` 的位置取消设置它。** 将其从 CI 密钥、容器环境和 shell 配置文件中移除（请参阅前面的警告）。重新运行 `ant auth status`，确认现在联合来源胜出。
4. **删除 API 密钥。** 一旦工作负载使用联合令牌运行，请在 Claude Console 的 [API 密钥](https://platform.claude.com/settings/keys)页面（`platform.claude.com/settings/keys`）上删除该密钥。

## 令牌生命周期与刷新

生成的 Anthropic 令牌的生命周期取以下两者中的较小值：(a) 规则的 `token_lifetime_seconds`（默认 3,600 秒），以及 (b) 您出示的 IdP JWT 剩余生命周期的两倍。结果永远不会少于 60 秒。第二个上限可防止 Anthropic 令牌的存活时间超出其所派生的上游身份太多。

SDK 会缓存令牌，并按照两级计划进行刷新：

* **建议性刷新**：在过期前 120 秒。SDK 尝试一次新的交换。如果令牌端点不可达，SDK 会继续使用缓存的令牌，该令牌大约还有 90 秒有效。
* **强制性刷新**：在过期前 30 秒。此时交换失败会导致错误。缓存的令牌距离过期太近，已不安全。

由于 SDK 在每次交换时都会重新读取 `ANTHROPIC_IDENTITY_TOKEN_FILE`，它可以自动获取已轮换的投射令牌（例如，Kubernetes 服务账户令牌会在其 `exp` 之前很久就进行轮换）。

默认情况下，带有 `jti` 声明的身份令牌是一次性的。每次交换都必须提交一个此前未被交换过的 JWT。再次提交同一 JWT 会失败，并在 Claude Console 的[身份验证历史页面](https://platform.claude.com/settings/workload-identity-federation?tab=history)上显示原因 `jti_reused`。如果您的工作负载从您的身份提供商自行获取令牌，请为每次交换生成一个新的 JWT，而不是重复使用缓存的 JWT。重试循环是重复使用的常见原因。SDK 在每次交换时都会重新读取 `ANTHROPIC_IDENTITY_TOKEN_FILE`，因此在每次刷新之前，该文件必须包含一个新令牌。重新读取未轮换令牌的刷新会以同样的方式失败，重新提交已交换过的令牌的重启进程也是如此。在生成令牌的有效期内尽早轮换令牌，可使该文件领先于刷新计划。如果您的令牌来源无法如此频繁地轮换，作为最后手段，您可以为该颁发者禁用 `check_jti`。这会移除该颁发者上所有规则的重放保护。有关详细信息，请参阅 [JWT 验证](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#jwt-verification)。

## 身份提供商

每份指南都涵盖该平台上 JWT 的来源、其声明的形式，以及需要注册的颁发者和规则配置。

<CardGroup cols={3}>
  <Card title="AWS" icon="cloud" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/aws">
    STS Web 身份令牌，或 EKS 投射服务账户令牌。
  </Card>

  <Card title="Google Cloud" icon="cloud" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/gcp">
    来自元数据服务器的 Google 签名身份令牌。
  </Card>

  <Card title="Microsoft Entra ID" icon="cloud" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/azure">
    托管身份（IMDS）和 AKS 上的 Entra Workload ID。
  </Card>

  <Card title="GitHub Actions" icon="github-logo" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/github-actions">
    使用 Actions OIDC 令牌进行无密钥 CI 身份验证。
  </Card>

  <Card title="Kubernetes" icon="cube" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/kubernetes">
    使用投射服务账户令牌的自管理和本地部署集群。
  </Card>

  <Card title="SPIFFE" icon="fingerprint" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/spiffe">
    使用来自 SPIRE 或其他符合规范的颁发者的 SPIFFE JWT-SVID 的工作负载。
  </Card>

  <Card title="Okta" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/okta">
    使用客户端凭据流程的 Okta 服务应用。
  </Card>
</CardGroup>

## 另请参阅

* [使用 Admin API 管理 WIF](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api)：通过基础设施即代码创建颁发者、服务账户和规则
* [WIF 参考](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference)：环境变量、配置文件的文件模式、验证规则和错误代码
* [身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)：SDK 的所有身份验证选项
* [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization)：每个 Admin API 端点的生成请求和响应模式
