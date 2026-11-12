---
title: 身份验证
url: https://platform.claude.com/docs/zh-CN/manage-claude/authentication
description: 使用 API 密钥、工作负载身份联合或 App Attest 向 Claude API 进行身份验证。
---

Claude API 支持三种对请求进行身份验证的方式：

| 方法                                                                                                                                    | 凭据                                                        | 最适合                                                                          |
| ------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------------------- |
| [API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#api-keys)                                                | 作为 bearer 令牌在 `Authorization` 标头中发送的静态 `sk-ant-api...` 密钥 | 本地开发、原型设计、脚本以及您控制密钥存储的服务器                                                    |
| [Workload Identity Federation（WIF）](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#workload-identity-federation) | 从您的身份提供商的身份令牌交换而来的短期 bearer 令牌                            | 云平台（AWS、Google Cloud 和 Azure）上的生产工作负载、CI/CD 流水线以及 Kubernetes，在这些场景中您希望消除静态密钥 |
| [App Attest](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#app-attest)                                          | 颁发给您已注册的 iOS 或 macOS 应用的真实、经过证明的安装实例的短期访问令牌               | 分发给最终用户的 iOS 和 macOS 应用，其中应用直接调用 Claude API，没有后端或代理                          |

API 密钥和工作负载身份联合授予对 Claude API 端点的相同访问权限。选择 API 密钥可以快速上手：用于您自己开发的个人密钥，或用于任何共享用途的服务账户密钥。当您的工作负载已经拥有可以联合的平台颁发身份时，请转向工作负载身份联合。对于您分发给最终用户的 iOS 和 macOS 应用，请使用 App Attest。

## API 密钥

API 密钥是您在 Claude Console 中生成的静态密钥，并在每个请求中作为 bearer 令牌在 `Authorization` 标头中发送。

### 密钥类型

当您创建密钥时，您需要选择其类型，这决定了密钥可以做什么、在哪里有效以及何时停止工作：

| 密钥类型          | 代表身份                                                                                                         | 适用范围                                                     | 何时失效                                                                                        |
| ------------- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **个人密钥**      | 您本人（用户），具有您的角色和权限                                                                                            | 单个工作区，或您的角色允许使用 API 的所有工作区，在创建密钥时选择                      | 您失去对组织的访问权限时。单工作区密钥在您失去对该工作区的访问权限或该工作区被归档时也会失效。当您被移出组织时，个人密钥会被归档。如果您被重新邀请，请创建新密钥；已归档的密钥不会恢复 |
| **服务账户密钥**    | 一个[服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#service-accounts) | 单个工作区，或该服务账户有权访问的任何内容，在创建密钥时选择。服务账户可以访问默认工作区以及它已被添加到的工作区 | 服务账户被归档时。单工作区密钥在服务账户被移出该工作区或该工作区被归档时也会失效                                                    |
| **工作区密钥**（旧版） | 不代表任何人：它属于创建它的工作区                                                                                            | 该工作区                                                     | 密钥过期、被禁用或删除，或其所属工作区被归档时，无论其创建者是否离开组织                                                        |

个人密钥和服务账户密钥是基于身份的：每个密钥都属于您的组织已在管理的某个用户或服务账户，每个请求都以该身份执行。当该身份被移出组织时，密钥即失效。这意味着密钥不会意外地比拥有它们的人员或工作负载存续得更久。对于新的集成，请优先使用它们而不是工作区密钥。

将个人密钥用于您自己的开发和脚本。共享的个人密钥充当一个人，当该人离开时就会失效。对于共享或自动化的工作负载（CI、生产服务），请让组织管理员创建一个服务账户，以便工作负载拥有自己的身份。

工作区 API 密钥有效，但属于旧版。请优先使用基于身份的密钥或[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)。要进行迁移，请参阅[替换工作区 API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#replacing-workspace-api-keys)。

### 创建和使用密钥

* **创建密钥：**&#x5728; Claude Console 中，前往 [API 密钥](https://platform.claude.com/settings/keys)页面（`platform.claude.com/settings/keys`）并点击**创建密钥**。为密钥命名并选择[过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)。将**已关联账户**设置为您自己以创建个人密钥，或设置为某个服务账户以创建供多个用户共享的密钥。您还可以将密钥限定到特定工作区，这样在以后的请求中就无需手动设置工作区 ID。
* **使用密钥：**&#x5728;直接 HTTP 请求中以 `Authorization: Bearer <key>` 的形式发送，或设置 `ANTHROPIC_API_KEY` 环境变量，[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 会自动读取它。

```http
POST /v1/messages
Authorization: Bearer YOUR_API_KEY
anthropic-version: 2026-10-21
content-type: application/json
```

您也可以在旧版的 `x-api-key: YOUR_API_KEY` 标头中发送密钥，以替代 `Authorization`。

请将 API 密钥存储在密钥管理器中，定期轮换，并禁用或删除任何您怀疑已泄露的密钥。在 Claude Console 的 [API 密钥](https://platform.claude.com/settings/keys)页面上，**停用**是可逆的（Admin API 会将密钥的 `status` 报告为 `"inactive"`，而**重新启用**会将其恢复为 `"active"`），而**删除**是永久性的：密钥会被归档，并仍会以 `status: "archived"` 出现在[列出 API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/list)中。已过期的密钥只能被删除。您还可以在创建密钥时设置[过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)，以限制泄露的凭据可被使用的时长。若要限制泄露的密钥可在何处使用，请启用 [IP 允许列表](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist)。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
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
  # 有关 zsh、bash 和 Windows 的变体写法，请参阅 /docs/en/cli-sdks-libraries/cli/authentication#api-key
  export ANTHROPIC_API_KEY=sk-ant-api03-...
  ```

  ```python Python
  client = Anthropic(api_key="my-anthropic-api-key")
  # 或者，在环境中设置 ANTHROPIC_API_KEY 后：
  client = Anthropic()
  ```

  ```typescript TypeScript
  const client = new Anthropic({ apiKey: "my-anthropic-api-key" });
  // 或者，在环境中设置 ANTHROPIC_API_KEY 后：
  // const client = new Anthropic();
  ```

  ```csharp C#
  AnthropicClient client = new() { ApiKey = "my-anthropic-api-key" };
  // 或者，在环境中设置了 ANTHROPIC_API_KEY 的情况下：
  // AnthropicClient client = new();
  ```

  ```go Go
  client := anthropic.NewClient(
  	option.WithAPIKey("my-anthropic-api-key"), // defaults to os.LookupEnv("ANTHROPIC_API_KEY")
  )
  ```

  ```java Java
  // 显式指定
  AnthropicClient client = AnthropicOkHttpClient.builder()
      .apiKey("my-anthropic-api-key")
      .build();

  // 从 ANTHROPIC_API_KEY（或 anthropic.apiKey 系统属性）读取
  AnthropicClient clientFromEnv = AnthropicOkHttpClient.fromEnv();
  ```

  ```php PHP
  // 从环境变量中读取 ANTHROPIC_API_KEY
  $client = new Client();
  // 或显式传入密钥：
  $client = new Client(apiKey: 'my-anthropic-api-key');
  ```

  ```ruby Ruby
  anthropic = Anthropic::Client.new(api_key: "my-anthropic-api-key")
  # 或者，在环境中设置 ANTHROPIC_API_KEY 后：
  anthropic = Anthropic::Client.new
  ```
</CodeGroup>

### 选择工作区

为特定工作区创建的 API 密钥仅在该工作区中有效，使用这些密钥的 API 请求可以省略工作区 ID。

如果您的 API 密钥未限定到某个工作区，则必须在每个请求的 `anthropic-workspace-id` 标头中指定工作区 ID。请参阅以下示例，了解如何在请求中或 SDK 中设置此标头。

[Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 仅在密钥未限定到特定工作区时才接受个人密钥或服务账户密钥。

您可以在 Claude Console 的[工作区](https://platform.claude.com/settings/workspaces)页面（`platform.claude.com/settings/workspaces`）的 **ID** 列中找到工作区的 ID，也可以调用[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)端点获取。仅当您传入 `include_default=true` 时，列出工作区端点才会包含默认工作区；其 ID 也会出现在任何在该工作区中运行的请求的 `anthropic-workspace-id` [响应标头](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#identify-the-workspace-behind-an-api-response)中。

<CodeGroup>
  ```bash cURL
  # 对于未限定到某个工作区的密钥，每个请求都必须包含此项。
  # 对于限定到单个工作区的密钥，请省略 anthropic-workspace-id 标头。
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-workspace-id: wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}]
    }'
  ```

  ```bash CLI
  # 对于未限定到工作区的密钥，每条命令都必须提供此参数。
  # 对于限定到单个工作区的密钥，请省略 --workspace-id。
  ant messages create \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  client = Anthropic()  # reads ANTHROPIC_API_KEY

  # 对于未限定到某个工作区的密钥，每个请求都必须提供此项。
  # 对于限定到单个工作区的密钥，请省略 extra_headers。
  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
      extra_headers={"anthropic-workspace-id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"},
  )
  print(message.content)

  # 或者为此客户端发出的所有请求统一设置一次：
  workspace_client = Anthropic(
      default_headers={"anthropic-workspace-id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"},
  )
  ```

  ```typescript TypeScript
  const client = new Anthropic(); // reads ANTHROPIC_API_KEY

  // 对于未限定到某个工作区的密钥，每个请求都必须提供此项。
  // 对于限定到单个工作区的密钥，请省略第二个参数。
  const message = await client.messages.create(
    {
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    },
    { headers: { "anthropic-workspace-id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ" } }
  );
  console.log(message.content);

  // 或者为此客户端的所有请求统一设置一次：
  const workspaceClient = new Anthropic({
    defaultHeaders: { "anthropic-workspace-id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ" }
  });
  ```

  ```csharp C#
  AnthropicClient client = new(); // reads ANTHROPIC_API_KEY

  MessageCreateParams parameters = new()
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }],
  };

  // 对于未限定到某个工作区的密钥，每个请求都必须提供此项。
  // 对于限定到单个工作区的密钥，直接调用 client.Messages.Create(parameters) 即可。
  var message = await client
      .WithOptions(options =>
          options with
          {
              ExtraHeaders = new Dictionary<string, string>
              {
                  ["anthropic-workspace-id"] = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
              },
          }
      )
      .Messages.Create(parameters);
  Console.WriteLine(message);

  // 或者为此客户端的所有请求统一设置一次：
  AnthropicClient workspaceClient = new(new ClientOptions
  {
      ExtraHeaders = new Dictionary<string, string>
      {
          ["anthropic-workspace-id"] = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      },
  });
  ```

  ```go Go
  client := anthropic.NewClient() // reads ANTHROPIC_API_KEY

  // 对于未限定到某个工作区的密钥，每个请求都必须提供此项。
  // 如果密钥仅限定到一个工作区，请省略此选项。
  message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  }, option.WithHeader("anthropic-workspace-id", "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"))
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(message.Content)

  // 或者为此客户端的所有请求统一设置一次：
  workspaceClient := anthropic.NewClient(
  	option.WithHeader("anthropic-workspace-id", "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"),
  )
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv(); // reads ANTHROPIC_API_KEY

  // 对于未限定到某个工作区的密钥，每个请求都必须提供此项。
  // 对于限定到单个工作区的密钥，请省略 putAdditionalHeader。
  Message message = client.messages().create(MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .addUserMessage("Hello, Claude")
      .putAdditionalHeader("anthropic-workspace-id", "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
      .build());

  IO.println(message.content());

  // 或者为此客户端的所有请求统一设置一次：
  AnthropicClient workspaceClient = AnthropicOkHttpClient.builder()
      .fromEnv()
      .putHeader("anthropic-workspace-id", "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
      .build();
  ```

  ```php PHP
  $client = new Client(); // reads ANTHROPIC_API_KEY

  // 对于未限定到某个工作区的密钥，每个请求都必须提供此项。
  // 对于限定到单个工作区的密钥，请省略 requestOptions。
  $message = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      requestOptions: [
          'extraHeaders' => ['anthropic-workspace-id' => 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ'],
      ],
  );

  echo json_encode($message->content), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new # reads ANTHROPIC_API_KEY

  # 对于未限定到某个工作区的密钥，每个请求都必须提供此项。
  # 对于限定到单个工作区的密钥，请省略 request_options。
  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    request_options: { extra_headers: { "anthropic-workspace-id" => "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ" } }
  )

  puts message.content
  ```
</CodeGroup>

如果使用未限定到工作区的密钥发出的请求省略了该标头，API 将返回 400 `invalid_request_error`：

```json JSON
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "This API key is not scoped to a workspace, so this request must include the anthropic-workspace-id header with the ID of the workspace to use. Add the header, or use an API key that is scoped to a workspace."
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

如果标头值不是有效的工作区 ID，将返回 400 `invalid_request_error`，消息为 `anthropic-workspace-id header must be a valid workspace ID.`。如果工作区不存在，或者密钥的用户或服务账户无权访问它，API 将返回 404 `not_found_error`，消息为 ``Workspace `<id>` not found.``，这与任何未知工作区的响应相同。

工作负载身份联合则会在令牌交换时选择工作区；详情请参阅 [WIF 参考](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference)。

### 密钥过期

当您在 Claude Console 的 [API 密钥](https://platform.claude.com/settings/keys)页面上创建 API 密钥时，需要选择一个过期时间：预设值（3 小时、1 天、7 天或 30 天）、自定义时长，或者对于存储在密钥管理器中并由您自行轮换的密钥选择**从不**。如果您的组织设有最长过期时间策略，Console 会将预设值和自定义时长限制在策略允许的最大值以内，并且**从不**不可用。过期时间在创建时设置，之后无法更改。在 Claude Console 中[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)时，同样适用此过期时间选项。

当过期时间临近时，Anthropic 会向密钥的创建者发送电子邮件：对于生命周期至少为 14 天的密钥，在过期前 7 天发送；对于生命周期至少为 7 天的密钥，在过期前 1 天发送。生命周期较短的密钥会在没有警告邮件的情况下过期。

密钥过期后，使用它发出的请求将返回 `401 authentication_error`。创建新密钥以恢复访问；过期的密钥无法重新激活。

Console 的 API 密钥表格会显示每个密钥的过期时间，Admin API 也会在[列出 API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/list)和[检索 API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/retrieve)端点上报告每个密钥的 `expires_at` 时间戳，以便您在密钥过期前进行审计和轮换。对于没有过期时间的密钥，该字段为 `null`。

过期时间限制了泄露凭据的生命周期，但它不能替代密钥卫生管理。无论是否过期，都应将密钥存储在密钥管理器中，并禁用或删除任何您怀疑已泄露的密钥。

### 替换工作区 API 密钥

如果您有工作区密钥，您可能希望用[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)或个人或服务账户密钥来替换它。这提供了更好的安全性和可观测性。

有关配置 Workload Identity Federation（工作负载身份联合）的详细信息，请参阅 [WIF 参考](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference)，相比长期有效的密钥，我们更推荐使用这种方式。

要用个人或服务账户密钥替换工作区密钥：

1. **确定密钥类型。**&#x60A8;自己的工具应使用个人密钥。共享或无人值守的工作负载应使用服务账户密钥。
2. **创建服务账户**（如有必要）。您可能需要请组织管理员在 Claude Console 的[服务账户](https://platform.claude.com/settings/service-accounts)页面（`platform.claude.com/settings/service-accounts`）上创建一个服务账户，并将其添加到相关工作区。
3. **创建新密钥。**&#x9664;非集成需要访问多个工作区，否则请专门为该集成所在的工作区创建密钥。
4. **部署新密钥。**&#x5728;集成读取密钥的所有位置替换旧密钥，通常是 `ANTHROPIC_API_KEY` 环境变量或密钥管理器中的条目。对于未限定到工作区的密钥，还需按照[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)中所示发送 `anthropic-workspace-id` 标头。
5. **删除旧密钥。**&#x786E;认请求成功后，在 Claude Console 的 [API 密钥](https://platform.claude.com/settings/keys)页面上删除该工作区密钥。

### 基于浏览器的应用程序

请从您自己的后端调用 Claude API，而不是从在用户浏览器中运行的代码调用。任何使用网页的人都可以读取该页面代码中的 API 密钥。通常，您的 Web 前端会调用您的后端，由后端持有密钥并调用 Claude API。

直接从浏览器发送的请求是跨源（CORS）请求。只有当这些请求包含 `anthropic-dangerous-direct-browser-access: true` 标头时，API 才会接受它们；当您将 `dangerouslyAllowBrowser` 设置为 `true` 时，TypeScript SDK 会发送该标头。对于以这种方式暴露密钥可以接受的情况，请参阅 TypeScript SDK 指南中的[浏览器使用](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#runtime-support)。

启用了[零数据保留（ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#zero-data-retention-zdr-scope)或 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#set-up-the-compliance-api) 的组织无法直接从浏览器调用 API，即使带有该标头也不行。其他组织设置也可能产生相同的效果，并且没有任何设置可以为这些组织允许浏览器请求。即使 API 密钥有效，API 也会返回 401 `authentication_error`，其消息以 `CORS requests are not allowed for this Organization` 开头。关联到 Claude Enterprise 父组织的 Claude Console 组织会从父组织继承 Compliance API 设置。因此，即使您的 Claude Console 设置中没有显示 Compliance API 部分，该限制也可能适用。

该限制仅适用于包含 `Origin` 标头的请求，浏览器会添加此标头，而服务器端 HTTP 客户端通常不会。来自您自己后端的请求不受影响。如果您的后端转发从浏览器发起的请求，请让后端创建一个新的请求发送到 Claude API。不要直接透传浏览器的请求。该请求仍然包含浏览器的 `Origin` 标头，API 会以同样的方式拒绝它。

## 工作负载身份联合

“Workload Identity Federation”（工作负载身份联合），即 WIF，允许工作负载使用由您已信任的"identity provider"（身份提供商），即 IdP，签发的短期身份令牌进行身份验证。此类 IdP 包括 AWS IAM、Google Cloud 以及任何符合标准的 OIDC 签发方（例如 GitHub Actions、Kubernetes 服务账户、SPIFFE、Microsoft Entra ID 或 Okta）。工作负载在 `POST /v1/oauth/token` 处用其 IdP 签发的 JWT 换取短期的 Claude API 访问令牌，SDK 会在该令牌过期前自动刷新。无需生成、分发或轮换任何 `sk-ant-api...` 字符串。

联合身份验证会从您的环境中移除长期有效的 Claude API 密钥，从而限制凭据泄露可能造成的损害，并让您能够使用已用于云资源的相同 IdP 控制措施来管理访问权限。但它本身并不能保证端到端的安全性：信任链的强度仅取决于您的身份提供商的配置，而信任链中更早一环的长期密钥（例如，能够生成 IdP 令牌的静态云凭据）仍可能削弱其安全性。请将联合身份验证与您提供商的控制措施结合使用，例如 IP 允许列表、MFA 和审计日志。

要配置联合身份验证，您需要在 Claude Console 中创建三个资源（一个服务账户、一个联合颁发者和一个联合规则），然后将您的 SDK 指向该规则。有关完整的设置演练，请参阅[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)。

## App Attest

App Attest 用于对直接从设备调用 Claude API 的 iOS 和 macOS 应用进行身份验证。每个安装实例都会借助 Apple 的 App Attest 服务，证明自己是您在 Claude Console 中注册的应用的真实、未经修改的构建版本。随后，Anthropic 会向该设备签发一个短期访问令牌，并将用量计费到您的工作区。令牌限定于您的工作区，1 小时后过期，并且仅授权 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 调用。

要注册您的应用并获取客户端 ID，请参阅 [适用于 iOS 和 macOS 应用的 App Attest](https://platform.claude.com/docs/zh-CN/manage-claude/app-attest)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="设置工作负载身份联合" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation">
    配置颁发者、规则和服务账户，然后交换令牌
  </Card>

  <Card title="身份提供商指南" icon="cloud" href="https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#identity-providers">
    AWS、Google Cloud、Azure、GitHub Actions、Kubernetes、SPIFFE 和 Okta 的分步指南
  </Card>

  <Card title="WIF 参考" icon="book" href="https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference">
    环境变量、验证规则、配置文件配置和错误参考
  </Card>

  <Card title="适用于 iOS 和 macOS 应用的 App Attest" icon="fingerprint" href="https://platform.claude.com/docs/zh-CN/manage-claude/app-attest">
    让您应用的真实安装实例无需附带 API 密钥即可调用 Claude API
  </Card>

  <Card title="客户端 SDK" icon="code" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview">
    Python、TypeScript、C#、Go、Java、PHP、Ruby 和 CLI
  </Card>
</CardGroup>
