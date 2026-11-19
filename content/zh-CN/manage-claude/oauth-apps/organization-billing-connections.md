---
title: Linked billing
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections
description: 让您的客户通过 OAuth 将其 Claude Platform 组织关联到您的应用，并将 Claude 用量计费到客户的组织。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：它是根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 beta 期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

Linked billing 让您的客户可以通过他们自己的 Claude 组织为您应用中的 Claude 用量付费。客户只需将其 Claude 组织关联到您的应用一次。API 将这种关联称为"connection"（连接）。客户完成关联后，您可以在每个 Claude API 请求上选择由客户还是由您付费。客户可以随时取消与您应用的关联。如果您标记为由客户付费的请求无法计费给客户，Claude API 会拒绝该请求，而不是向您计费。

本指南面向其应用用户主要为 Claude Platform 客户的开发者。您需要在应用中添加一个"Link your Claude organization"按钮。您的用户点击该按钮，登录 Claude，并批准您的应用就客户指示您发出的 Claude API 请求向其 Claude 组织计费。之后，您的应用会在由客户付费的请求中添加一个 [`billing_policy`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#step-3-update-calls-to-claude) 字段。

## 符合条件的客户和端点

**符合条件的客户：** 只有当客户的 Claude Platform 组织直接向 Anthropic 支付其 API 用量费用时，才能以这种方式向其计费。预付额度和发票计费均符合条件。通过云市场（AWS、Google Cloud 或 Azure）计费的组织无法通过此集成进行计费。仅拥有 Claude 订阅方案的组织也是如此。针对这些组织的请求会被拒绝，并返回 403 `user_profile_not_billable`。

请求在您的组织中运行，因此适用您的速率限制、模型访问权限、服务层级、数据保留和加密密钥。您客户自身的数据安排不适用于这些请求，包括["zero data retention"（零数据保留），即 ZDR](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#zero-data-retention-zdr-scope)、"business associate agreement"（业务伙伴协议），即 BAA，以及"customer-managed encryption keys"（客户管理的加密密钥），即 CMEK。对于此类用量需要 HIPAA、FERPA 或 CMEK 保障的客户，请勿进行关联。

**支持的端点：** `billing_policy` 在以下端点上受支持：

* `/v1/messages`
  * 已知限制：向您的客户计费的请求（`bill_to: "user_profile"`）无法使用[计费的代码执行功能](https://platform.claude.com/docs/zh-CN/about-claude/pricing#code-execution-tool)（API 会返回 400），因此请使用 `bill_to: "self"` 或不带 `billing_policy` 发送这些请求。
* `/v1/chat/completions`
* `/v1/messages/count_tokens` 接受 `billing_policy` 字段但会忽略它，因此您可以对即将发送的完全相同的请求体进行令牌计数。

[Message Batches](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)（`/v1/messages/batches`）目前尚不支持向您的客户计费。请在批次中使用 `bill_to: "self"` 或不带 `billing_policy` 发送请求，这些请求将照常向您计费。批次中设置了 `bill_to: "user_profile"` 的请求会被拒绝：其结果为 `errored`，并附带 `invalid_request_error`，您和您的客户都不会为此请求付费，且该批次中的其他请求将照常运行。

如需请求在其他端点上提供支持，请联系您的 Anthropic 客户团队。

### 哪些额度可以付费

并非所有额度都能支付平台计费给客户的请求。可以支付的项目取决于被计费的组织：如果您向客户计费，则为该客户；如果平台向您计费，则为您的组织。

在 Claude Console 中为该组织购买的额度可以支付这些请求。如果该组织采用发票计费，则会就这些请求收到发票。

Anthropic 提供的某些促销额度无法支付这些请求。其中包括 Claude Max、Ultra 和 Team 方案附带的每月 API 额度。来自 Anthropic 合作伙伴、风险投资（VC）和初创企业计划的额度无资格支付这些请求。

当该组织账户上没有任何项目可以支付某个请求时，Claude API 可能会以 402 拒绝该请求。响应不会说明拒绝是由额度不足还是支出限额导致的。请参阅[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#expected-errors)。

## 设置流程

以下三个步骤只需执行一次。它们会向 Anthropic 注册您的应用。执行这三个步骤都需要您在组织中拥有管理员角色。之后，[要关联哪个用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#which-user-profile-to-link)介绍了您在创建用户配置文件之前需要了解的一条规则。

### 步骤 1：创建应用

在 Claude Console 中创建 OAuth 应用。它是向 Anthropic 标识您产品的注册信息：它将您的重定向 URI 与客户端 ID 关联起来，其名称就是您的用户在批准页面上看到的名称。

您组织中具有管理员角色的成员在 Console 中打开 [OAuth 应用页面](https://platform.claude.com/settings/oauth-apps)，然后点击**创建**。输入您应用的名称和重定向 URI（批准后浏览器返回到的您应用中的地址）。在 **What the app can do** 下，选择 **Bill and link to organizations**。

新应用的页面随即打开。复制 Console 为该应用显示的客户端 ID。您的应用会在步骤 5 请求批准页面时、在步骤 6 重定向浏览器时以及在步骤 8 交换代码时发送该 ID，以便 Anthropic 知道是哪个应用在发出请求。客户端 ID 是公开的。在**外观**选项卡上，设置您应用的名称、主页和徽标 URL。

在**设置**选项卡上，对于新应用，**Who can link** 显示为 **Test users only**，新应用初始处于测试状态。在您[发布应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#publish-your-app)且其上线后，该设置显示为 **Any Claude user**。

<Note>
  一个组织可以注册多个 OAuth 应用，例如一个用于生产环境，一个用于测试。每个应用都有自己的连接：关联了您测试应用的客户并不会关联到您的生产应用。在集成的整个生命周期内请将 Linked billing 保留在同一个生产应用上。
</Note>

### 步骤 2：设置客户端身份验证

设置您的后端如何向 `auth.claude.com` 证明其请求来自您的应用。这取代了客户端密钥：Anthropic 不会为 OAuth 应用颁发客户端密钥。

在应用的 **Client authentication** 选项卡上，选择您的**身份提供商**（为您后端的令牌签名的颁发者）。如需添加一个，请点击 **Register a new issuer**。然后设置这些令牌必须匹配的**主题模式**，或在**高级匹配选项**下设置声明条件。保存后，该选项卡会显示预期的受众。您的客户端断言的 `aud` 声明必须与该值完全一致。

<Note>
  您的应用使用 `private_key_jwt` 方法（RFC 7523）的一种变体向 PAR、令牌和撤销端点进行身份验证：每次请求时，您的后端都会发送一个"client assertion"（客户端断言），即由您的身份提供商颁发的 JWT。`auth.claude.com` 会使用为该颁发者注册的密钥检查令牌的签名，并根据您在此处设置的主体或声明条件对其进行检查。有关客户端断言必须包含的声明，请参阅[客户端断言声明](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#client-assertion-claims)。
</Note>

### 步骤 3：设置您后端的身份

您的后端需要拥有自己的身份才能调用 Claude API。请创建以下两项：

* **“Service account”（服务账户）。** 这是您 Claude 组织中的一个非人类账户。您的后端在调用 Claude API 时以该账户身份进行身份验证。
* **第二条联合规则。** 与步骤 2 中用于应用客户端身份验证的规则类似，它让您的后端能够使用来自您自己身份提供商的令牌进行身份验证。

如需在 Claude Console 中进行设置，请按照[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)指南操作。

然后，在应用的 **Client authentication** 选项卡上，选择该服务账户作为应用的操作员服务账户。只有该服务账户有权发送推理请求并将计费归属到已关联的用户配置文件。一个服务账户只能作为一个 OAuth 应用的操作员。

默认情况下，以该服务账户身份进行身份验证的任何请求都可以通过在 Messages API 中设置用户配置文件标头和 `bill_to` 向每个已关联的客户计费。如需获取仅限于特定连接的身份验证令牌，请参阅[每个连接的 OAuth 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#oauth-tokens-for-each-connection)。在关联客户之前，请为您的应用[设置工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#settings-for-a-connections-oauth-tokens)。步骤 8 会为每个客户返回这样一个令牌，并要求应用在您的组织中指定一个用于运行推理的默认工作区。如果您需要每个连接的刷新令牌，还需为您的应用开启 **Offline access**。然后，在关联客户之前，在步骤 5 的授权流程中请求 `offline_access`。

### 要关联哪个用户配置文件

“User profile”（用户配置文件）是您自己的 Claude 组织中代表您某个客户的一条记录。每个向客户计费的请求都包含该配置文件的 ID，以便 Anthropic 知道要向哪个客户计费。有关配置文件的其他用途，请参阅[用于终端客户归属的用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)。

在关联第一个客户之前，请确定每个客户由哪个用户配置文件代表。在一个应用内，连接是一对一的：

* 一个客户的 Claude 组织只能关联到您的一个用户配置文件。
* 一个用户配置文件只能关联到一个 Claude 组织。

客户的 Claude 组织是指用户在批准您的应用时选择的那个组织。如需了解客户的各个团队如何使用 Claude 组织，请询问客户。如需将一对一规则应用于您的客户记录：

* **一个客户的多个团队共用一个 Claude 组织：** 为所有这些团队使用一个用户配置文件。
* **每个团队都有自己的 Claude 组织：** 为每个组织创建一个用户配置文件。

只要符合一对一规则，您组织中的任何活跃用户配置文件都可以被关联。关联不会更改该配置文件、其 ID 或其用量历史记录。如果您将第二个用户配置文件关联到一个已被关联的 Claude 组织，批准会被拒绝：请参阅 [`user_profile_already_linked`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#user-profile-already-linked)。

#### 为每个人创建用户配置文件

您可能已经为使用您产品的每个人都创建了用户配置文件。其中有几人可能为同一个客户工作。在这种情况下，Anthropic 建议您再为该客户的 Claude 组织创建一个用户配置文件，并关联该配置文件。

要发送的配置文件 ID 取决于由谁为请求付费：

* **您计费给客户的请求：** 发送该客户配置文件的 ID。如需同时在 `/v1/messages` 请求中标识具体的人员，请将 [`metadata.user_id`](https://platform.claude.com/docs/zh-CN/api/messages/create) 设置为该人员的标识符。该标识符是可选的，它有助于 Anthropic 检测单个人员的滥用行为。请使用不透明的值，而不是姓名、电子邮件地址或电话号码。
* **由您付费的请求：** 您可以继续发送该人员自己的配置文件 ID。

## 关联每个客户

对于每个关联其组织的客户，您需要执行步骤 4 到 8。以下 Python 代码片段需要 Anthropic SDK 和一些标准导入；请参阅[代码片段初始化](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#code-snippet-initialization)。其他语言的示例和 CLI 从环境中获取凭据。请确保未设置 `ANTHROPIC_API_KEY`，因为您的应用需要使用工作负载身份联合进行身份验证。

在您产品中客户管理计费或集成的位置添加一个"Link your Claude organization"按钮（有关按钮的 HTML、颜色和徽标，请参阅[品牌示例](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/branding-samples#buttons)）。告知客户，要进行关联，他们必须在其 Claude 组织中具有管理员或账单角色。

如需先自己走一遍这些步骤，请参阅[以客户身份测试](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#test-as-a-customer)。

<Frame>
  ![步骤 5 到 8 的序列图：批准请求、用户的批准以及兑换连接 ID](https://platform.claude.com/docs/zh-CN/images/oauth-apps-billing-connection-flow.svg)
</Frame>

### 步骤 4：为客户创建用户配置文件

为每个客户的 Claude 组织创建一个用户配置文件，并将该配置文件的 ID 存储在您产品中该客户的记录上。

如果您已有该客户 Claude 组织的用户配置文件，请跳过此步骤并使用该配置文件的 ID。如需确定是哪个配置文件，请参阅[要关联哪个用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#which-user-profile-to-link)。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/user_profiles \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "name": "Example Customer Inc.",
      "external_user_details": {"reference_id": "your-customer-id-123"}
    }'
  ```

  ```bash CLI
  ant user-profiles create \
    --name "Example Customer Inc." \
    --external-user-details.reference-id your-customer-id-123
  ```

  ```python Python
  def create_user_profile(name: str, customer_id: str) -> str:
      profile = client.user_profiles.create(
          name=name,
          external_user_details={"reference_id": customer_id},
      )
      return profile.id  # "uprof_..."


  user_profile_id = create_user_profile("Example Customer Inc.", "your-customer-id-123")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const profile = await client.userProfiles.create({
    name: "Example Customer Inc.",
    external_user_details: { reference_id: "your-customer-id-123" }
  });

  console.log(profile.id); // "uprof_..."
  ```

  ```csharp C#
  AnthropicClient client = new();

  var profile = await client.UserProfiles.Create(
      new()
      {
          Name = "Example Customer Inc.",
          ExternalUserDetails = new() { ReferenceID = "your-customer-id-123" },
      }
  );

  Console.WriteLine(profile.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  profile, err := client.UserProfiles.New(context.Background(), anthropic.UserProfileNewParams{
  	Name: anthropic.String("Example Customer Inc."),
  	ExternalUserDetails: anthropic.UserProfileExternalUserDetailsParams{
  		ReferenceID: anthropic.String("your-customer-id-123"),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(profile.ID)
  ```

  ```java Java
  import com.anthropic.models.userprofiles.UserProfileCreateParams;
  import com.anthropic.models.userprofiles.UserProfileExternalUserDetailsParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserProfileCreateParams.builder()
          .name("Example Customer Inc.")
          .externalUserDetails(UserProfileExternalUserDetailsParams.builder()
              .referenceId("your-customer-id-123")
              .build())
          .build();
      var profile = client.userProfiles().create(params);

      IO.println(profile.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $profile = $client->userProfiles->create(
      name: 'Example Customer Inc.',
      externalUserDetails: ['referenceID' => 'your-customer-id-123'],
  );

  echo $profile->id, PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  profile = client.user_profiles.create(
    name: "Example Customer Inc.",
    external_user_details: { reference_id: "your-customer-id-123" }
  )

  puts(profile.id)
  ```
</CodeGroup>

如果您的组织转售 API 访问权限，请在创建此配置文件时同时发送 `"access_type": "passthrough"`。这些示例省略了 `access_type`，这会创建一个 `application` 配置文件。有关这两种类型，请参阅[决定谁获得配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#decide-who-gets-a-profile)。

响应包含该配置文件的 `id`，例如 `uprof_01HV9Z`。

<Note>
  每个创建请求都会生成一个新的配置文件，即使 `reference_id`（您为该客户设定的 ID）相同也是如此，因此请复用已存储的 ID，而不是在每次连接尝试时都创建配置文件。
</Note>

### 步骤 5：创建推送授权请求 URL

创建一个"pushed authorization request"（推送授权请求），引用步骤 4 中客户的用户配置文件。响应会为您提供一个 `request_uri`。在下一步中，您将使用它把用户重定向到批准页面。

在发送请求之前，您的后端会创建两个随机值，并将它们存储在浏览器会话和客户记录上。它会在步骤 8 中再次使用这两个值：

* 一个 `state` 值，让您可以检查返回到您应用的浏览器就是发起此请求的浏览器。
* 一个 PKCE 代码验证器，这是只有您的后端知道的密钥。请求中携带它的哈希值（代码质询），而步骤 8 会发送验证器本身，从而只有您的后端才能使用返回的代码。

<CodeGroup exclude="shell:CLI, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/par \
    -H "content-type: application/json" \
    -d '{
      "response_type": "code",
      "client_id": "YOUR_CLIENT_ID",
      "redirect_uri": "https://app.example.com/claude/billing/callback",
      "scope": "billing:attribution",
      "connection_type": "organization_shared",
      "connection_auth_target": "platform",
      "user_profile_id": "uprof_01HV9Z",
      "state": "RANDOM_STATE",
      "code_challenge": "BASE64URL_SHA256_OF_VERIFIER",
      "code_challenge_method": "S256",
      "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      "client_assertion": "YOUR_CLIENT_ASSERTION_JWT"
    }'
  ```

  ```python Python
  def push_authorization_request(user_profile_id: str) -> dict:
      state = secrets.token_urlsafe(32)
      code_verifier = secrets.token_urlsafe(64)
      code_challenge = create_s256_code_challenge(code_verifier)

      response = httpx.post(
          f"{AUTH}/v1/oauth/par",
          json={
              "response_type": "code",
              "client_id": CLIENT_ID,
              "redirect_uri": REDIRECT_URI,
              "scope": "billing:attribution",
              "connection_type": "organization_shared",
              "connection_auth_target": "platform",
              "user_profile_id": user_profile_id,
              "state": state,
              "code_challenge": code_challenge,
              "code_challenge_method": "S256",
              "client_assertion_type": JWT_BEARER,
              "client_assertion": client_assertion(),
          },
      )
      response.raise_for_status()  # on an error, do not redirect the browser

      return {
          "state": state,
          "code_verifier": code_verifier,
          "request_uri": response.json()["request_uri"],  # valid for 600 seconds
          "user_profile_id": user_profile_id,
      }


  par = push_authorization_request(user_profile_id)
  ```
</CodeGroup>

`auth.claude.com` 会以 HTTP 201 响应，并返回一个 `request_uri` 以及值为 600 秒的 `expires_in`。`request_uri` 标识此请求对应的批准页面。`scope` 必须正好是 `billing:attribution` 或 `billing:attribution offline_access`。有关详细信息，请参阅[计费连接的作用域](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#scopes-for-a-billing-connection)。示例发送的是 `billing:attribution`。如果您的应用没有此作用域，请在应用的 **Permissions** 选项卡上的 **Billing** 下添加它。

`connection_auth_target` 必须为 `platform`：推理请求在您自己的组织中运行，以操作员服务账户身份进行身份验证，但可以向客户计费。

`user_profile_id` 必须是您组织中某个活跃用户配置文件的 ID。如果推送授权请求失败，`auth.claude.com` 会向您的后端返回 OAuth 错误响应。在这种情况下，请勿重定向浏览器。

### 步骤 6：将用户引导至 Claude 批准页面

首先，向您的用户显示他们正在关联的是您产品中的哪个账户，并告知他们检查批准页面所显示的 Claude 账户。该页面会显示当前登录的是哪个 Claude 账户，并允许用户切换到其他账户。批准页面只提供浏览器中当前登录的 Claude 账户的组织，因此错误的账户可能会关联错误的组织。由用户关联的组织付费。

<Note>
  进行批准的人必须是客户 Claude 组织的管理员：即具有管理员或账单角色的成员。
</Note>

然后，将您的用户重定向到授权端点，该端点会显示一个批准页面。仅包含您的 `client_id` 和步骤 5 中的 `request_uri`。

```python
def authorize_url(par: dict) -> str:
    query = urlencode({"client_id": CLIENT_ID, "request_uri": par["request_uri"]})
    return f"{AUTH}/oauth/authorize?{query}"


redirect_to = authorize_url(par)  # answer the click with a 302 to this URL
```

### 步骤 7：用户批准

在此步骤中，您的应用无需执行任何操作。用户在 `auth.claude.com` 上完成以下步骤：

1. 用户登录 Claude，除非浏览器已处于登录状态。

2. 批准页面会显示以下界面，每个界面都会显示您应用的名称。有关用户在每个界面上执行的操作，请参阅[关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connecting)。

   1. **Select an organization**，仅当用户有多个组织可供选择时才会显示。
   2. **How billing works**。
   3. **Set a monthly spend limit**。
   4. 带有 **Link and authorize** 按钮的最后一个界面。

3. 当用户点击 **Link and authorize** 时，`auth.claude.com` 会将浏览器送回您的应用。批准页面不会显示确认界面。

如果用户取消或无法批准，则不会建立连接：请参阅[如果关联未完成](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#if-linking-doesnt-complete)。

<Note>
  连接属于整个 Claude 组织，而不属于批准它的用户。如果该用户离开组织，连接仍保持有效。客户组织或您组织的任何管理员都可以随时终止连接，您的后端也可以。
</Note>

### 步骤 8：处理回调并保存连接

批准后，`auth.claude.com` 会将浏览器送回您的重定向 URI，并附带一次性代码和您的 `state` 值。这个发往您重定向 URI 的请求称为"callback"（回调）。您的后端用该代码兑换一个连接 ID，并将其存储在客户记录上。至此，连接完成。回调可能携带 `error` 参数而不含代码。这种情况下不会建立连接：请参阅[如果关联未完成](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#if-linking-doesnt-complete)。

检查 `state` 是否与您在步骤 5 中存储的值匹配，然后立即从您的后端将代码与 PKCE 代码验证器一起进行兑换。该代码只能使用一次，并在 5 分钟后过期。

<CodeGroup exclude="shell:CLI, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/token \
    -H "content-type: application/json" \
    -d '{
      "grant_type": "authorization_code",
      "code": "AUTH_CODE",
      "code_verifier": "ORIGINAL_CODE_VERIFIER",
      "redirect_uri": "https://app.example.com/claude/billing/callback",
      "client_id": "YOUR_CLIENT_ID",
      "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      "client_assertion": "YOUR_CLIENT_ASSERTION_JWT"
    }'
  ```

  ```python Python
  def handle_callback(par: dict, params: dict) -> str:
      if not secrets.compare_digest(
          params.get("state", "").encode(), par["state"].encode()
      ):
          raise ValueError("state does not match")
      if "error" in params:  # for example access_denied
          raise PermissionError(params["error"])
      code = params["code"]  # single-use, expires after 5 minutes
      response = httpx.post(
          f"{AUTH}/v1/oauth/token",
          json={
              "grant_type": "authorization_code",
              "code": code,
              "code_verifier": par["code_verifier"],
              "redirect_uri": REDIRECT_URI,
              "client_id": CLIENT_ID,
              "client_assertion_type": JWT_BEARER,
              "client_assertion": client_assertion(),
          },
          timeout=10,  # a response that times out loses the tokens in it
      )
      response.raise_for_status()
      tokens = response.json()
      # 要使用该连接的令牌，请存储 tokens["access_token"]、其过期
      # 时间（来自 tokens["expires_in"]），以及（使用 offline_access 时）
      # tokens["refresh_token"]，作为机密保存在此：该 code 无法再次交换。
      # "ocon_..."；将其存储在客户记录中
      return tokens["connection_id"]
  ```
</CodeGroup>

响应中包含连接 ID 和[该连接的访问令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#oauth-tokens-for-each-connection)，该令牌可以进行身份验证并向批准的客户计费。如果客户批准了 `offline_access`，响应中还会包含一个刷新令牌。请立即存储 `connection_id` 和这些令牌：兑换只会返回一次令牌。[存储令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#keep-the-tokens-from-step-8)介绍了该响应。

本页面中调用 Claude API 的示例以步骤 3 中的服务账户身份进行身份验证，而不是使用此响应中的令牌。[使用连接的访问令牌调用 Claude](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#call-claude-with-a-connections-access-token) 介绍了如何改用该令牌。

响应不会标识用户关联的是哪个 Claude 组织。

连接完成并不保证 Anthropic 能够向客户计费。例如，客户的组织可能不属于[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#eligible-customers-and-endpoints)。有关 Claude API 如何检查这一点，请参阅[更新对 Claude 的调用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#step-3-update-calls-to-claude)和[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#expected-errors)。由于存在该检查，请将您的成功消息表述为"已关联"，而不是"计费已生效"。

如果兑换因 `unauthorized_client` 被拒绝，请参阅[代码兑换返回 `unauthorized_client`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#code-exchange-unauthorized-client)。

<Note>
  用户可以针对已关联的用户配置文件再次进行批准。结果取决于作用域：

  * 针对同一用户配置文件以相同的作用域再次批准：连接保持不变，令牌响应返回相同的 `connection_id` 和一个新的访问令牌。例外情况是具有 `billing:attribution offline_access` 的连接：代码兑换会返回 `invalid_grant`，连接保持原样，[再次关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connect-a-customer-again)介绍了应如何处理。
  * 针对同一用户配置文件以其他作用域再次批准，例如在 `billing:attribution` 之后以 `billing:attribution offline_access` 批准：您兑换新代码时会创建一个具有新 `connection_id` 的新连接，并取代之前的连接。之前 `billing:attribution` 连接的令牌会被撤销。请存储新的 `connection_id` 以替换之前的 ID。如果兑换失败并返回 5xx 响应，请再次执行步骤 5 到 8。[计费连接的作用域](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#scopes-for-a-billing-connection)介绍了何时需要这样做。
  * 连接被撤销后，您可以再次关联同一个用户配置文件。
</Note>

### 以客户身份测试

如需检查关联是否正常工作，请关联您自己的一个组织。您首先需要：

* 您的第二个 Claude 组织，用于充当客户。您无法将应用关联到拥有该应用的组织。批准页面不会提供该组织。第二个组织必须属于[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#eligible-customers-and-endpoints)。
* 该组织中一位具有管理员或账单角色的人员，由其批准关联。
* 在您的应用上线之前，该人员必须位于您应用的 **Test users** 列表中。该列表位于应用的**设置**选项卡上的 **Who can link** 下。通过该人员 Claude 账户的电子邮件地址添加其本人，然后点击 **Save changes**。[应用处于测试阶段时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#while-your-app-is-in-testing)介绍了适用的支出上限。从列表中移除某人会终止其批准的关联。

然后，执行步骤 4 到 8，并让该人员在步骤 7 中进行批准。关联建立后，该用户配置文件会在 Console 中您的 [User profiles](https://platform.claude.com/user-profiles) 页面上显示为 **Linked**。

### 如果关联未完成

当用户未完成批准，或 `auth.claude.com` 拒绝批准时，不会建立连接。以下条目涵盖 Linked billing 特有的结果。有关其他所有错误，请参阅 OAuth 参考中的[错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#errors)。

#### 您的重定向 URI 收到 `error=access_denied`

该请求还会携带您的 `state` 值。您的重定向 URI 会在以下等情况下收到此错误：

1. 用户在任一批准界面上点击 **Cancel**。

2. 批准页面没有可提供给该用户的组织，且用户点击了返回您应用的按钮。例如：

   * 该用户在任何 Claude Platform 组织中都不具有管理员或账单角色。
   * 该用户唯一的组织就是拥有您应用的组织。请参阅[以客户身份测试](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#test-as-a-customer)。
   * 该用户配置文件已关联到另一个 Claude 组织，且该用户在该组织中不具有管理员或账单角色。
   * 在您的应用上线之前，该用户不在其 **Test users** 列表中。

在此列表中的每种情况下，`error_description` 都显示为 `the user declined the request`，因此您无法据此区分这些情况。

另一个原因在于您应用的设置。如果您的应用已上线，且 **Who can link** 显示为仅限您的组织，则您组织以外的任何人都无法关联该应用。请选择 **Any Claude user** 并点击 **Save changes**。

请告知用户关联未完成，不要假定他们拒绝了批准。请他们检查登录所用的 Claude 账户，以及他们在要关联的组织中的角色。用户取消或返回后，`request_uri` 无法再次使用，因此如需让用户重试，请从步骤 5 重新开始。

#### 您的应用收到 `user_profile_already_linked`

客户的 Claude 组织已通过您的另一个用户配置文件关联到您的应用。`auth.claude.com` 会要求用户联系您。如果用户返回您的应用，您的重定向 URI 将收到 `error=user_profile_already_linked` 和您的 `state` 值。当您在步骤 8 中兑换代码时，令牌端点也可能返回此错误。例如，当该组织在此期间已通过另一个用户配置文件关联时，就会出现这种情况。

Anthropic 不会告知您客户关联的是哪个 Claude 组织，因此您无法事先发现这一点。步骤 5 中的推送授权请求会成功，`auth.claude.com` 仅在用户批准时才会拒绝该连接。

该错误不会指明持有该连接的用户配置文件。最可能的持有者是该客户的另一个已关联配置文件。Anthropic 不会显示某个配置文件关联到哪个组织，因此请在采取行动之前与客户确认。如果该配置文件持有该连接，请执行以下操作之一：

* 在您计费给该客户的请求中，发送该用户配置文件的 ID。
* 按照[取消关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#disconnecting-a-customer)中的说明终止该连接，然后使用被拒绝的用户配置文件再次执行步骤 5 到 8。

如果该客户的所有配置文件都未关联，则用户可能选择了属于您另一个客户的组织。请从步骤 5 重新开始，并让用户检查其选择的组织。

#### 您的重定向 URI 未收到任何内容

您的应用无法检测以下两种情况：

* 用户关闭了批准页面。
* `request_uri` 已过期或已被使用。此时批准页面会显示错误，且不会将浏览器送回您的应用。

如果用户在没有回调的情况下返回您的产品，请从步骤 5 重新开始。未使用的 `request_uri` 会在您创建它 10 分钟后过期。

#### 代码兑换返回 `unauthorized_client`

您的应用未针对此请求进行设置。对于工作区问题，`error_description` 会指明原因。对于缺少操作员服务账户的情况，它仅显示 `the client is not registered for this grant`。该拒绝不会消耗代码，因此在代码过期之前，您可以再次兑换同一代码。首先，请在 Claude Console 中检查以下各项：

| 检查项          | 位置                                                                                                                                                 | 修复方法                                                |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| 应用有操作员服务账户   | 应用 **Client authentication** 选项卡上的 **Operator service account**                                                                                    | 选择一个并点击 **Save**。步骤 3 介绍了如何设置。                      |
| 应用的工作区存在且未归档 | [设置您的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#settings-for-a-connections-oauth-tokens) | 选择一个工作区。在应用没有工作区时，兑换同样会因 `unauthorized_client` 被拒绝。 |

保存的服务账户可能需要一两分钟才能同步到 `auth.claude.com`，因此如果兑换再次被拒绝，请等待这么长时间后重试。如果届时代码已过期，请让用户再次批准（步骤 5 到 8）。有关 `error_description` 针对连接令牌所指明的原因，请参阅[存储令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#keep-the-tokens-from-step-8)和[刷新令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#refresh-the-connection)。

在其他任何情况下，请联系您的 Anthropic 客户团队。如果 Console 显示该服务账户已不存在，也请这样做。

### 发布您的应用

在您的应用上线之前，只有其 **Test users** 列表中的人员才能关联它，因此您的客户无法关联。如需发布应用，请在 Claude Console 中该应用的页面上点击 **Publish app**。随即会打开一个对话框。

* 如果您的组织采用发票计费，则在您于该对话框中确认后，应用会立即上线。
* 否则，您需要先在该对话框中证明您控制该应用的域名，然后 Anthropic 会审核该应用。

**域名证明：** 该域名是您应用主页的域名。每个重定向 URI 都必须位于该域名或其子域名上，但位于 `localhost`、`127.0.0.1` 或 `[::1]` 上的重定向 URI 除外。证明方式是一条 DNS TXT 记录。DNS 更改可能需要长达 48 小时，因此请在需要客户关联之前就开始。如果您的组织已经证明过涵盖该主页和这些重定向 URI 的域名，对话框会跳过此证明。已证明的域名涵盖其子域名。

您在对话框的最后一步进行确认。如果该步骤显示用于接受 Linked Billing Terms 的复选框，请选中它。然后，在对话框中点击 **Publish app**。

**审核：** 审核期间，应用在 Console 中的状态显示为 **In review**。Anthropic 批准后，应用即上线。如果 Anthropic 未批准，状态显示为 **Not approved**，且您无法自行再次发布该应用。如需询问该决定，请联系支持团队。

除非您的组织采用发票计费，否则更改已上线应用的名称、徽标、主页或重定向 URI 会使该应用重新进入审核。已关联的客户会保持关联，并保留其已保存的上限。在 Anthropic 批准该更改之前，只有 **Test users** 列表中的人员才能关联该应用。

## 更新对 Claude 的调用

对于已关联的客户，在每次调用 `/v1/messages` 和 `/v1/chat/completions` 时发送 `anthropic-user-profile-id` 标头。在您计费给客户的请求中，该标头的值是持有该客户连接的用户配置文件的 ID。设置了 `billing_policy` 的调用还必须发送 `user-profiles-billing-2026-10-01` beta 标头。

针对已关联客户的每个请求要么计费给客户，要么计费给您，您可以在每个请求上选择。以下示例展示了这两种情况。

[每个连接的 OAuth 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#oauth-tokens-for-each-connection)介绍了另一种对客户请求进行身份验证的方式：一个只能向该客户计费的访问令牌。在 `/v1/messages` 和 `/v1/chat/completions` 上，使用此类令牌的请求无法计费给您。在 `billing_policy` 和 `anthropic-user-profile-id` 标头为可选的情况下，即使不带它们，该请求也会计费给该客户：请参阅[使用连接的访问令牌调用 Claude](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#call-claude-with-a-connections-access-token) 中的说明。如果该请求指定了另一个用户配置文件或设置了 `bill_to: "self"`，则会被拒绝。切勿将此类令牌用于由您付费的请求。

### 由您的客户付费

在请求体中添加带有 `bill_to: "user_profile"` 的 `billing_policy`，并发送计费 beta 标头和配置文件 ID 标头。

<Warning>
  请检查该标头指定的是该请求所针对的客户的配置文件。默认情况下，您操作员服务账户自己的凭据可以向通过此应用关联的每个客户计费。如果使用该凭据的请求指定了通过此应用关联的另一个配置文件，Anthropic 会向该配置文件的组织计费，且不会有任何错误提示您。[连接的访问令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#oauth-tokens-for-each-connection)只能向其自己的客户计费。
</Warning>

仅在客户指示您发出的请求上设置 `bill_to: "user_profile"`：即客户及其用户在您的产品中产生的 Claude 用量。不要将您自己的用量（例如测试、评估或为其他客户所做的工作）计费给客户。Claude API 无法为您检查这一点，因此这是您的责任。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-billing-2026-10-01" \
    -H "anthropic-user-profile-id: uprof_01HV9Z" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}],
      "billing_policy": {"type": "billing_policy", "bill_to": "user_profile"}
    }'
  ```

  ```bash CLI
  ant beta:messages create \
    --beta user-profiles-billing-2026-10-01 \
    --user-profile-id uprof_01HV9Z \
    --model claude-sonnet-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}' <<'YAML'
  billing_policy:
    type: billing_policy
    bill_to: user_profile
  YAML
  ```

  ```python Python
  def message_billed_to_user(user_profile_id: str, prompt: str) -> BetaMessage:
      return client.beta.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[{"role": "user", "content": prompt}],
          betas=[BILLING_BETA],
          extra_headers={"anthropic-user-profile-id": user_profile_id},
          extra_body={
              "billing_policy": {"type": "billing_policy", "bill_to": "user_profile"}
          },
      )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.beta.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    billing_policy: { type: "billing_policy", bill_to: "user_profile" }
  });

  console.log(message.content);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Messages;
  using System.Text.Json;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 1024,
      Messages = [new BetaUserMessageParam { Content = "Hello, Claude" }],
      Betas = ["user-profiles-billing-2026-10-01"],
      UserProfileID = "uprof_01HV9Z",
  };
  var body = new Dictionary<string, JsonElement>(parameters.RawBodyData)
  {
      ["billing_policy"] = JsonSerializer.SerializeToElement(
          new { type = "billing_policy", bill_to = "user_profile" }
      ),
  };

  var message = await client.Beta.Messages.Create(
      MessageCreateParams.FromRawUnchecked(parameters.RawHeaderData, parameters.RawQueryData, body)
  );

  Console.WriteLine(string.Join("\n", message.Content));
  ```

  ```go Go
  client := anthropic.NewClient()

  params := anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude")),
  	},
  	Betas:         []anthropic.AnthropicBeta{"user-profiles-billing-2026-10-01"},
  	UserProfileID: anthropic.String("uprof_01HV9Z"),
  }
  params.SetExtraFields(map[string]any{
  	"billing_policy": map[string]string{
  		"type":    "billing_policy",
  		"bill_to": "user_profile",
  	},
  })

  message, err := client.Beta.Messages.New(context.Background(), params)
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("%+v\n", message.Content)
  ```

  ```java Java
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;
  import java.util.Map;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_SONNET_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .addBeta("user-profiles-billing-2026-10-01")
          .userProfileId("uprof_01HV9Z")
          .putAdditionalBodyProperty("billing_policy", JsonValue.from(Map.of(
              "type", "billing_policy",
              "bill_to", "user_profile")))
          .build();
      var message = client.beta().messages().create(params);

      IO.println(message.content());
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->beta->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_SONNET_5_5,
      betas: ['user-profiles-billing-2026-10-01'],
      userProfileID: 'uprof_01HV9Z',
      requestOptions: [
          'extraBodyParams' => [
              'billing_policy' => ['type' => 'billing_policy', 'bill_to' => 'user_profile'],
          ],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    request_options: {
      extra_body: { billing_policy: { type: "billing_policy", bill_to: "user_profile" } }
    }
  )

  puts(message.content)
  ```
</CodeGroup>

如果请求被拒绝，请参阅[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#expected-errors)。

### 由您付费

如需由您自己付费，可以完全省略 `billing_policy`（在这种情况下，计费 beta 标头是可选的）：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-user-profile-id: uprof_01HV9Z" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --user-profile-id uprof_01HV9Z \
    --model claude-sonnet-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  def message_billed_to_you(user_profile_id: str, prompt: str) -> Message:
      return client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[{"role": "user", "content": prompt}],
          user_profile_id=user_profile_id,
      )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    user_profile_id: "uprof_01HV9Z"
  });

  console.log(message.content);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var message = await client.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeSonnet5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Hello, Claude" }],
          UserProfileID = "uprof_01HV9Z",
      }
  );

  Console.WriteLine(string.Join("\n", message.Content));
  ```

  ```go Go
  client := anthropic.NewClient()

  message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  	UserProfileID: anthropic.String("uprof_01HV9Z"),
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("%+v\n", message.Content)
  ```

  ```java Java
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_SONNET_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .userProfileId("uprof_01HV9Z")
          .build();
      var message = client.messages().create(params);

      IO.println(message.content());
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_SONNET_5_5,
      userProfileID: 'uprof_01HV9Z',
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    user_profile_id: "uprof_01HV9Z"
  )

  puts(message.content)
  ```
</CodeGroup>

或者使用 `bill_to: "self"`：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-billing-2026-10-01" \
    -H "anthropic-user-profile-id: uprof_01HV9Z" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}],
      "billing_policy": {"type": "billing_policy", "bill_to": "self"}
    }'
  ```

  ```bash CLI
  ant beta:messages create \
    --beta user-profiles-billing-2026-10-01 \
    --user-profile-id uprof_01HV9Z \
    --model claude-sonnet-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}' <<'YAML'
  billing_policy:
    type: billing_policy
    bill_to: self
  YAML
  ```

  ```python Python
  def message_billed_to_self(user_profile_id: str, prompt: str) -> BetaMessage:
      return client.beta.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[{"role": "user", "content": prompt}],
          betas=[BILLING_BETA],
          extra_headers={"anthropic-user-profile-id": user_profile_id},
          extra_body={"billing_policy": {"type": "billing_policy", "bill_to": "self"}},
      )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.beta.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    billing_policy: { type: "billing_policy", bill_to: "self" }
  });

  console.log(message.content);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Messages;
  using System.Text.Json;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 1024,
      Messages = [new BetaUserMessageParam { Content = "Hello, Claude" }],
      Betas = ["user-profiles-billing-2026-10-01"],
      UserProfileID = "uprof_01HV9Z",
  };
  var body = new Dictionary<string, JsonElement>(parameters.RawBodyData)
  {
      ["billing_policy"] = JsonSerializer.SerializeToElement(
          new { type = "billing_policy", bill_to = "self" }
      ),
  };

  var message = await client.Beta.Messages.Create(
      MessageCreateParams.FromRawUnchecked(parameters.RawHeaderData, parameters.RawQueryData, body)
  );

  Console.WriteLine(string.Join("\n", message.Content));
  ```

  ```go Go
  client := anthropic.NewClient()

  params := anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude")),
  	},
  	Betas:         []anthropic.AnthropicBeta{"user-profiles-billing-2026-10-01"},
  	UserProfileID: anthropic.String("uprof_01HV9Z"),
  }
  params.SetExtraFields(map[string]any{
  	"billing_policy": map[string]string{
  		"type":    "billing_policy",
  		"bill_to": "self",
  	},
  })

  message, err := client.Beta.Messages.New(context.Background(), params)
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("%+v\n", message.Content)
  ```

  ```java Java
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;
  import java.util.Map;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_SONNET_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .addBeta("user-profiles-billing-2026-10-01")
          .userProfileId("uprof_01HV9Z")
          .putAdditionalBodyProperty("billing_policy", JsonValue.from(Map.of(
              "type", "billing_policy",
              "bill_to", "self")))
          .build();
      var message = client.beta().messages().create(params);

      IO.println(message.content());
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->beta->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_SONNET_5_5,
      betas: ['user-profiles-billing-2026-10-01'],
      userProfileID: 'uprof_01HV9Z',
      requestOptions: [
          'extraBodyParams' => [
              'billing_policy' => ['type' => 'billing_policy', 'bill_to' => 'self'],
          ],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    request_options: {
      extra_body: { billing_policy: { type: "billing_policy", bill_to: "self" } }
    }
  )

  puts(message.content)
  ```
</CodeGroup>

## 每个连接的 OAuth 令牌

您的后端以您应用的操作员服务账户（步骤 3）身份调用 Claude API。默认情况下，以该服务账户身份进行身份验证的任何请求都可以通过传递用户配置文件 ID 标头并在 Messages API 请求中包含 `bill_to` 向每个已关联的客户计费。此默认行为称为"operator-only tokenless authentication"（仅限操作员的无令牌身份验证）。

为了支持您产品的单租户部署或由客户控制的沙箱环境，您可以获取绑定到连接的 OAuth 访问令牌和刷新令牌，而不是使用仅限操作员的无令牌身份验证。

对于每个连接，步骤 8 中的代码兑换会返回该连接自己的访问令牌。该令牌以您的操作员服务账户身份进行身份验证，但只能向该连接的客户计费。它会在 1 小时内过期。

如需分别对每个连接进行身份验证：

* **使用离线访问。** 为您的应用开启 [**Offline access**](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#settings-for-a-connections-oauth-tokens)，并在步骤 5 中请求 `offline_access`。这样，每个连接还会获得一个刷新令牌，您的后端可使用它请求新的访问令牌。如果没有刷新令牌，客户必须再次批准才能为您提供新的访问令牌：请使用相同的 `scope` 再次执行步骤 5 到 8。
* **设置默认工作区。** 在您的组织中[选择一个工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#settings-for-a-connections-oauth-tokens)，使用这些令牌的请求将在其中运行。
* **（可选）关闭仅限操作员的无令牌身份验证。** 默认情况下，即使客户的访问令牌已过期，您仍可以以您的操作员服务账户身份（通过传递用户配置文件标头和 `bill_to`）向已关联的客户计费。在您[关闭仅操作员身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#turn-off-operator-only-authentication)之后，每个向客户计费的请求都必须使用该客户的访问令牌进行身份验证。
* **使用 `billing:attribution`。** 只有此作用域会为连接提供其自己的令牌。您仍可以像使用已弃用的 `org:billing_attribution` 时一样，以您的操作员服务账户身份向已关联的客户计费。请参阅[计费连接的作用域](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#scopes-for-a-billing-connection)。

### 设置您的应用

使用连接访问令牌的请求在您组织的一个工作区中运行。每个应用都需要一个工作区，即使您不使用这些令牌也是如此：当您的应用没有工作区时，步骤 8 中的代码兑换会被拒绝，并返回 `unauthorized_client`。

您组织中具有管理员角色的成员在 Console 中打开应用的 **Client authentication** 选项卡。在操作员服务账户下方，找到 **Default workspace for platform auth mode OAuth tokens**（应用的 `connection_token_workspace_id` 设置），然后点击 **Choose workspace**。选择操作员服务账户（步骤 3）所属的一个工作区。该工作区不得处于已归档状态。

如需获取刷新令牌，请在应用的 **Permissions** 选项卡上开启 **Offline access**。对应用作用域的更改需要几分钟才能同步到 `auth.claude.com`。

<Note>
  归档默认工作区会撤销在其中运行的所有令牌，且无法撤消。如需迁移到另一个工作区，请设置新工作区并等待几分钟。在更改同步到 `auth.claude.com` 之前进行的刷新可能会返回仍在之前工作区中运行的令牌。然后，刷新每个拥有刷新令牌的连接，之后再归档之前的工作区。如果您先归档了某个工作区，请在设置另一个工作区之前停止刷新其连接：之后的刷新会终止该连接。

  当有客户已关联时，请勿更改或归档应用的操作员服务账户。在您更改或归档它之后，`auth.claude.com` 会拒绝每次刷新，您持有的令牌很快将无法再向客户计费。之后，您使用其令牌的每个客户都必须重新关联：如需帮助，请[联系支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)。
</Note>

### 计费连接的作用域

在步骤 5 中，将 `scope` 设置为 `billing:attribution` 发送。如需同时获取"refresh token"（刷新令牌），请发送 `billing:attribution offline_access`。

如需更改已关联客户的作用域（例如添加 `offline_access`），请使用该客户现有的用户配置文件和新的 `scope` 再次执行步骤 5 到 8。您无需先取消关联。代码兑换会返回一个新的 `connection_id`，新连接将替代之前的连接。请参阅步骤 8 中的说明。

一个应用可以同时拥有 `billing:attribution` 和已弃用的 `org:billing_attribution`。您可以在应用的 **Permissions** 选项卡上的 **Billing** 下添加 `billing:attribution`。如果您在该处移除 `org:billing_attribution`，则无法再次添加它。客户使用 `org:billing_attribution` 批准的连接会继续通过您的操作员服务账户计费，直到该连接终止或您[关闭仅操作员身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#turn-off-operator-only-authentication)。

### 存储令牌

代码兑换仅返回一次令牌，因此请在步骤 8 中存储它们。响应包含 `access_token`、`expires_in`（最长 1 小时，以秒为单位）和 `connection_id`。如果客户批准了 `offline_access`，响应还会包含 `refresh_token` 和 `refresh_token_expires_in`。刷新令牌最长有效 1 年，且刷新不会延长该期限。到期后，请[再次关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connect-a-customer-again)。

请将令牌及其过期时间存储在只有负责刷新的服务才能读取的密钥存储中。

如果兑换因 `unauthorized_client` 被拒绝，`error_description` 会给出原因，例如工作区缺失或已归档。该代码不会被消耗：请更正原因，并在代码过期之前再次发送同一代码。

### 使用连接的访问令牌调用 Claude

<Note>
  在 beta 期间，请向您的 Anthropic 客户团队确认您的组织是否已启用使用连接访问令牌的 Claude API 请求。在未启用之前，所有此类请求都会失败，无论其发送什么内容。同时请确认 `anthropic-user-profile-id` 标头和 `billing_policy` 是否为可选。在它们不是可选之前，请按照[由您的客户付费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#your-customer-pays)所示，在每个使用连接访问令牌的 `/v1/messages` 和 `/v1/chat/completions` 请求中发送该标头、`bill_to: "user_profile"` 和计费 beta 标头。
</Note>

在 `Authorization` 标头中将连接的访问令牌作为 bearer 令牌发送，以代替您操作员服务账户自己的凭据。不要发送 `x-api-key` 标头。在 Anthropic SDK 中，将令牌作为 `auth_token` (typescript, php: `authToken`; java: `.authToken()`; csharp: `AuthToken`; go: `option.WithAuthToken()`) 传入，或设置 `ANTHROPIC_AUTH_TOKEN` 环境变量。

在 `/v1/messages` 和 `/v1/chat/completions` 上，请求会计费给该连接对应的客户。`anthropic-user-profile-id` 标头和 `billing_policy` 是可选的。仅在使用 `billing_policy` 时才需要计费 beta 标头。

在这两个端点上，您无法使用此令牌为请求付费。请仅在这两个端点、`/v1/messages/count_tokens` 和 Models 端点上使用该令牌。所有其他请求以及所有由您付费的请求，都应以您的操作员服务账户身份从您自己的后端发送。这包括使用[计费的代码执行功能](https://platform.claude.com/docs/zh-CN/about-claude/pricing#code-execution-tool)的请求，此类请求无法向客户计费。

Claude API 会拒绝以下请求：

* `400 invalid_request_error`，不含错误代码：请求指定了其他用户配置文件或设置了 `bill_to: "self"`。
* `403 permission_error`，不含错误代码：请求发送到了 [Message Batches](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 端点。
* `401 authentication_error`：令牌已过期、已被刷新替换，或其连接已终止。连接终止后，响应也可能是 `403 user_profile_not_billable`。

### 刷新令牌

如需续订连接的访问令牌，请使用您应用的[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#client-authentication)将其刷新令牌发送到令牌端点 `https://auth.claude.com/v1/oauth/token`。不要发送 `scope` 参数。

<CodeGroup exclude="shell:CLI, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/token \
    -H "content-type: application/json" \
    -d @- <<JSON
  {
    "grant_type": "refresh_token",
    "refresh_token": "$CONNECTION_REFRESH_TOKEN",
    "client_id": "YOUR_CLIENT_ID",
    "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
    "client_assertion": "$CLIENT_ASSERTION"
  }
  JSON
  ```

  ```python Python
  connection_locks: defaultdict[str, threading.Lock] = defaultdict(threading.Lock)


  def error_body(response: httpx.Response) -> dict:
      try:
          body = response.json()
      except ValueError:
          return {}
      return body if isinstance(body, dict) else {}


  def is_lost(response: httpx.Response) -> bool:
      """True for a 5xx response that can follow a refresh that took effect."""
      if response.status_code < 500:
          return False
      # Claude API 格式且带 retry-after 标头的 503 属于拒绝，
      # 不会消耗刷新令牌。其他所有 5xx 响应均视为丢失。
      paused = (
          response.status_code == 503
          and error_body(response).get("type") == "error"
          and "retry-after" in response.headers
      )
      return not paused


  def hold_error(connection_id: str, reason: str) -> RefreshOnHold:
      return RefreshOnHold(
          f"Connection {connection_id} is on hold. {reason} Don't send its refresh token"
          ' again: that can end the connection. See "Link a customer again" in the'
          " Linked billing guide."
      )


  def refresh_connection(connection_id: str, *, force: bool = False) -> tuple[str, int]:
      """Refreshes the connection if its access token is close to its expiry.

      Returns the current access token and its expiry time in Unix seconds. Call
      this from your schedule, and pass force=True to refresh a token that isn't
      close to its expiry. Raises RefreshOnHold when the refresh token can't be sent
      again, now and on every later call, and httpx.HTTPStatusError for a refusal
      that keeps the refresh token. After RefreshOnHold for a lost refresh, pause
      your schedule. Resume with one connection, and resume the others only after
      that refresh succeeds or gets a 4xx.
      """
      # 此锁仅在单个进程内生效。如果服务有多个副本，请使用
      # 所有副本共享的锁。
      with connection_locks[connection_id]:
          tokens = load_tokens(connection_id)
          if tokens.on_hold:
              raise hold_error(
                  connection_id, "An earlier refresh might have used its refresh token."
              )
          if not force and tokens.expires_at - time.time() > REFRESH_MARGIN_SECONDS:
              return tokens.access_token, tokens.expires_at  # not close to expiry
          request = {
              "grant_type": "refresh_token",
              "refresh_token": tokens.refresh_token,
              "client_id": CLIENT_ID,
              "client_assertion_type": JWT_BEARER,
              "client_assertion": client_assertion(),
          }
          url = f"{AUTH}/v1/oauth/token"
          # 在刷新令牌离开此进程之前，将连接置于暂停状态。
          # 如果进程在新令牌保存前停止，暂停状态将保留。
          save_tokens(connection_id, replace(tokens, on_hold=True))
          issued_at = int(time.time())
          started = time.monotonic()
          # httpx 本身不会重试。不要添加重试：延迟的重试会终止
          # 连接。
          try:
              response = httpx.post(url, json=request, timeout=REQUEST_TIMEOUT)
              lost = is_lost(response)
          except (httpx.ConnectError, httpx.ConnectTimeout):
              # 未建立连接，因此刷新令牌从未离开此进程。
              save_tokens(connection_id, tokens)
              raise
          except httpx.TransportError:
              lost = True
          if lost:
              # 第一个请求可能已使用了刷新令牌。第二个请求
              # 仅在上述时限内才安全，因此要么立即发送一次，要么
              # 不发送。
              if time.monotonic() - started > SECOND_REQUEST_DEADLINE_SECONDS:
                  raise hold_error(connection_id, "A refresh got no usable answer.")
              try:
                  response = httpx.post(url, json=request, timeout=REQUEST_TIMEOUT)
              except httpx.TransportError:
                  raise hold_error(
                      connection_id, "Two refreshes got no answer."
                  ) from None
              if response.status_code != 200:
                  # 收到除新令牌以外的任何响应后，您都无法判断
                  # 第一个请求是否生效，之后的请求也不安全。
                  raise hold_error(connection_id, "The second refresh failed.")
          if error_body(response).get("error") == "invalid_grant":
              # 已过期、已被使用或连接已终止。再次发送
              # 可能会终止连接。
              raise hold_error(connection_id, "The refresh returned invalid_grant.")
          if response.status_code != 200:
              # 保留刷新令牌的拒绝，因此解除暂停状态。
              save_tokens(connection_id, tokens)
              response.raise_for_status()
          body = response.json()
          tokens = ConnectionTokens(
              refresh_token=body["refresh_token"],
              access_token=body["access_token"],
              expires_at=issued_at + body["expires_in"],
              refresh_expires_at=issued_at + body["refresh_token_expires_in"],
          )
          # 如果写入失败，请重试写入。不要再次刷新。
          for attempt in range(5):
              try:
                  save_tokens(connection_id, tokens)
                  return tokens.access_token, tokens.expires_at
              except OSError:  # your secret store's error type
                  if attempt < 4:
                      time.sleep(2**attempt)
          # 新令牌仅存在于此进程中，且已存储的刷新令牌
          # 已被用掉。已存储的记录仍处于暂停状态。错误中携带新
          # 令牌，以便调用方在密钥存储恢复后保存它们。
          error = hold_error(
              connection_id,
              "The refresh succeeded, but the new tokens could not be saved.",
          )
          error.new_tokens = tokens
          raise error
  ```
</CodeGroup>

响应包含一个新的 `access_token` 和一个新的 `refresh_token`。您发送的刷新令牌已被替换：请勿再次发送它，除非下面的失败情况列表另有说明。之前的访问令牌会在几分钟内失效。[OAuth 令牌代码片段初始化](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#oauth-token-snippet-initialization)定义了示例所调用的函数。

* 在使用新的访问令牌之前，先存储新的刷新令牌。
* 从一个服务刷新每个连接的令牌，每次只进行一次刷新。如果该服务有多个副本，请使用其所有副本共享的锁。已使用的刷新令牌如果在首次使用 30 秒之后再次到达，会终止该连接。所有令牌都会被撤销，客户显示为 **Unlinked**，并且在客户再次关联之前，任何凭据都无法向其计费。
* 在您的 HTTP 客户端和任何代理中关闭刷新请求的自动重试。

如果使用某个刷新令牌的首次请求失败：

* 请求无法建立连接：刷新令牌未被发送。稍后再试。

* 连接中断，或出现除此列表中的 `503` 以外的任何 5xx 响应：您无法判断该请求是否已生效，因此请再发送一次同一刷新令牌。

  1. 立即发送，并且仅在距您发送首次请求不到 30 秒时发送。
  2. 如果它返回了新令牌，请存储它们并仅使用这些令牌。如果首次请求确实已生效，其令牌（从未送达您）会被撤销。
  3. 如果它返回任何其他内容或没有响应，或者 30 秒已过，请不要再次发送该刷新令牌。停止刷新该连接，并[再次关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connect-a-customer-again)。

* 超时：请求可能仍在运行。不要再次发送该刷新令牌。停止刷新该连接，并[再次关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connect-a-customer-again)。

* `429`，或带有 `retry-after` 标头且响应体顶层为 `"type": "error"` 的 `503`：没有任何内容被撤销。至少等待该标头指示的时长，然后再次发送同一刷新令牌。

* `unauthorized_client` 或 `invalid_client`：没有任何内容被撤销。`error_description` 会给出原因，例如工作区缺失、您的应用不再注册 `offline_access`，或客户端断言失败。更正后，再次发送同一刷新令牌。

* `invalid_scope`：请求发送了 `scope` 参数，或您的应用不再注册 `billing:attribution`。没有任何内容被撤销。更正原因后，再次发送同一刷新令牌。

* `invalid_grant`：刷新令牌已过期或已被使用、连接已终止，或客户的组织已被暂停。停止刷新该连接，并按照[再次关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connect-a-customer-again)操作，该流程首先会检查组织是否被暂停。

如果连接中断、5xx 响应或超时导致您停止刷新某个连接，请同时暂停其他连接的刷新。恢复时，先刷新一个连接，仅在该刷新成功或收到 4xx 响应后再恢复其他连接。

### 在沙箱中运行智能体

只向沙箱提供访问令牌，绝不提供刷新令牌。由您的后端进行刷新，并将每个新的访问令牌发送到沙箱。后端根据沙箱自身的身份（例如其 pod 身份）来确定沙箱服务于哪个客户，而绝不依据沙箱发送的值。后端只向沙箱发送该客户的令牌。沙箱不得能够获取与您的任何联合规则或您应用的客户端身份验证相匹配的 JWT。不要为其提供已挂载的身份令牌，也不要允许其访问云元数据服务器。

Claude Code、[Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) 和 Anthropic SDK 可以从配置文件中的凭据文件读取令牌（请参阅[配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#profile-configuration-file)）。在沙箱中，使用此配置文件创建一个配置目录。不要在其中添加 `client_id`：如果添加了，SDK 会尝试自行刷新令牌。

```json configs/customer.json
{
  "version": "1.0",
  "authentication": {
    "type": "user_oauth"
  }
}
```

在智能体启动之前以及每次刷新之后，将访问令牌写入同一目录下的 `credentials/customer.json`。`expires_at` 是以 Unix 秒表示的过期时间。以 `0600` 模式写入一个新文件，然后将其重命名以覆盖旧文件：

```json credentials/customer.json
{
  "type": "oauth_token",
  "access_token": "sk-ant-oat01-...",
  "expires_at": 1793635200
}
```

将 `ANTHROPIC_CONFIG_DIR` 设置为该配置目录，将 `ANTHROPIC_PROFILE` 设置为 `customer`。取消设置沙箱中的所有其他凭据，例如 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`CLAUDE_CODE_OAUTH_TOKEN` 以及 Claude Code 的 `apiKeyHelper` 设置，因为它们中的每一个都可能优先于该配置文件。智能体会在 `expires_at` 之前的最后 2 分钟内以及收到 401 响应后再次读取该文件。

```python
def write_credentials_file(
    config_dir: Path, profile: str, token: str, expires_at: int
) -> None:
    credentials_dir = config_dir / "credentials"
    credentials_dir.mkdir(mode=0o700, parents=True, exist_ok=True)
    body = {"type": "oauth_token", "access_token": token, "expires_at": expires_at}
    # 将写完的文件重命名到目标位置，使 agent 不会读到不完整的文件。
    # mkstemp 设置权限 0600 且所有者为当前用户，因此请以
    # agent 的用户身份运行。
    handle, temporary_path = tempfile.mkstemp(dir=credentials_dir)
    with os.fdopen(handle, "w") as temporary_file:
        json.dump(body, temporary_file)
    os.replace(temporary_path, credentials_dir / f"{profile}.json")
```

### 再次关联客户

在以下情况下再次关联客户：

* 刷新返回了 `invalid_grant`。
* 您丢失了刷新令牌，或刷新令牌已泄露。
* 您丢失了步骤 8 中针对 `billing:attribution offline_access` 的响应，或者步骤 8 针对此类连接返回了 `invalid_grant`。
* 刷新令牌即将过期。

请按以下步骤操作：

1. 在您的 [User profiles](https://platform.claude.com/user-profiles) 页面上，检查该客户是否显示为 **Linked**。收到 `invalid_grant` 后，请先向您的 Anthropic 客户团队确认该客户的组织是否已被暂停：暂停结束后，您持有的刷新令牌会重新生效。如果所有连接同时返回 `invalid_grant`，请在取消关联任何客户之前先排查您这一侧的原因，例如工作区已归档。
2. 如果该客户显示为 **Linked**，请按照[取消关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#disconnecting-a-customer)中的说明取消其关联。取消关联会撤销该连接的令牌。请在客户组织的管理员准备好批准时再执行此操作，因为在其批准之前，通过该连接的计费会停止。
3. 使用该客户现有的用户配置文件再次执行步骤 5 到 8。存储新的连接 ID 和新令牌，以替换之前的值。

### 关闭仅操作员身份验证

<Note>
  在 beta 期间，请向您的 Anthropic 客户团队确认您的组织是否可以使用此设置。在其可用之前，您无法开启它。
</Note>

如需阻止仅以您的操作员服务账户身份进行身份验证的请求向客户计费，请开启 **Disable operator-only tokenless authentication**（应用的 `disable_operator_only_auth` 设置）。该设置位于应用的 **Client authentication** 选项卡上的操作员服务账户下方。您的应用必须拥有 `billing:attribution` 和 `offline_access` 作用域以及一个[用于其 OAuth 令牌的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#settings-for-a-connections-oauth-tokens)。在开启之前，请确保每个已关联的客户都已批准 `billing:attribution offline_access`，并且您使用其访问令牌发出的请求能够正常工作。开启后，只有 `billing:attribution` 连接的访问令牌才能向该连接的客户计费。仅以服务账户身份进行身份验证并设置了 `bill_to: "user_profile"` 的请求会被拒绝，并返回 `403 user_profile_not_billable`。客户仍显示为 **Linked**，因此请在日志中留意此错误。服务账户仍可以管理用户配置文件并发送由您付费的请求。您可以再次关闭此设置。

### OAuth 令牌代码片段初始化

本节中的示例共用以下设置。它定义了：

* 导入，其中包括 `httpx` 包
* 您应用的客户端 ID 和客户端断言类型
* `ConnectionTokens`，用于保存连接的令牌、其过期时间以及 `on_hold` 标记
* `RefreshOnHold`，即当刷新令牌不得再次发送时 `refresh_connection()` 引发的错误：该令牌可能已被使用过，或者令牌端点返回了 `invalid_grant`。在您清除已存储令牌中的 `on_hold` 之前（例如在暂停结束之后），`refresh_connection()` 之后的每次调用也都会引发该错误。当刷新已成功、仅您的密钥存储拒绝了写入时，其 `new_tokens` 保存着需要存储的令牌。
* `client_assertion()`，它从 `ANTHROPIC_IDENTITY_TOKEN_FILE` 指定的文件中读取您身份提供商的 JWT。该 JWT 的 `aud` 声明必须是您应用客户端身份验证设置中的受众。
* `load_tokens()` 和 `save_tokens()`，您需要针对自己的密钥存储编写它们

```python
import json
import os
import tempfile
import threading
import time
from collections import defaultdict
from dataclasses import dataclass, field, replace
from pathlib import Path

import httpx

AUTH = "https://auth.claude.com"
# 注册 OAuth 应用后从 platform.claude.com 复制。
CLIENT_ID = "YOUR_CLIENT_ID"
JWT_BEARER = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer"
REFRESH_MARGIN_SECONDS = 600
# 在此超时的刷新会丢失新令牌，因此请为响应预留 10 秒。
# 无法建立的连接会在 2 秒内失败。
REQUEST_TIMEOUT = httpx.Timeout(10, connect=2)
# 第二个请求必须在第一个请求后这么短时间内发起，否则不得发起。
# 页面给出 30 秒，但请保持小于 REQUEST_TIMEOUT 的读取超时：
# 已超时的请求不得再次发送。
SECOND_REQUEST_DEADLINE_SECONDS = 4


@dataclass
class ConnectionTokens:
    refresh_token: str = field(repr=False)
    access_token: str = field(repr=False)
    expires_at: int  # the access token's expiry time, in Unix seconds
    refresh_expires_at: int  # the refresh token's expiry time, in Unix seconds
    # True：刷新令牌可能已被使用。不要再次发送。
    on_hold: bool = False


class RefreshOnHold(Exception):
    """The refresh token might have been used, so a person decides what to do next.

    Don't catch this to try again. If new_tokens is set, the refresh succeeded and
    only the write failed: store those tokens.
    """

    new_tokens: ConnectionTokens | None = None


def client_assertion() -> str:
    """Reads your identity provider's JWT on every call, because its file rotates."""
    return Path(os.environ["ANTHROPIC_IDENTITY_TOKEN_FILE"]).read_text().strip()


def load_tokens(connection_id: str) -> ConnectionTokens:
    """Reads the connection's newest tokens from your secret store."""
    raise NotImplementedError("Read the tokens from your secret store.")


def save_tokens(connection_id: str, tokens: ConnectionTokens) -> None:
    """Writes the connection's newest tokens to your secret store."""
    raise NotImplementedError("Write the tokens to your secret store.")
```

## 管理您的连接

### 在 Console 中查看客户

除了 OAuth 应用页面外，您还可以在 Console 的 [User profiles](https://platform.claude.com/user-profiles) 页面上查看您的所有用户配置文件。在该列表以及每个用户配置文件的详情页面上，**Your spend** 表示您自己为该用户配置文件的请求所承担的费用。这是 Anthropic 应用任何额度之前的费用。这些页面将连接称为"link"：连接处于活跃状态的用户配置文件显示为 **Linked**，连接已终止的用户配置文件显示为 **Unlinked**。

点击单个用户配置文件，可查看其过去 7 天、28 天或 90 天用量的更详细视图。

### 监控每个客户的用量

Claude Console 和[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 都会报告每个用户配置文件的用量。两者都会统计由您付费的请求和由您的客户付费的请求。两者都不会显示客户支付了多少费用，也不会显示客户的费率。

在 Console 中，[User profiles](https://platform.claude.com/user-profiles) 页面会列出每个用户配置文件在过去 28 天内的 **Requests**、**Tokens** 和 **Your spend**。由您的客户付费的请求会计入该配置文件的 **Requests** 和 **Tokens**，但不会计入 **Your spend**。

请使用 [Admin API 凭据](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)调用该 API，例如具有 `org:admin` 作用域的 OAuth 访问令牌或 Admin API 密钥。不要使用您的操作员服务账户（步骤 3）。其联合规则默认授予 `workspace:developer`，而不是 `org:admin`。`org:admin` 作用域涵盖整个 Admin API，因此请勿将其添加到操作员服务账户。

如需通过该 API 按用户配置文件生成报告，请调用其[用量端点](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#usage-endpoint)或[成本端点](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#cost-endpoint)。如需为每个用户配置文件获取一行数据，请仅按 `user_profile_id` 分组，并省略 `time_bucket_width`。只有在您请求的时间范围内有用量的用户配置文件才会有对应的行。另外还有一行的 `user_profile_id` 为 null。它包含未指定任何配置文件的用量，以及在请求之外计量的用量，例如代码执行容器时间。

在成本端点上，由您的客户付费的请求在 `amount` 中计为 0，并在 `list_amount` 中计入其标价。`amount` 字段与 **Your spend** 是相同的度量。该 API 没有用于表示由谁付费的字段、筛选条件或 `group_by[]` 值。`amount` 为 0 且 `list_amount` 大于 0 的行通常表示该行中的每个请求都由客户付费。

请勿使用这些数据在您选定的金额处停止客户的用量。Console 和该 API 都不会实时报告。Console 中的数据大约每小时刷新一次，而一个请求可能需要几个小时才会显示出来。有关该 API 的延迟，请参阅[数据新鲜度](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#data-freshness)。[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#spend-caps)介绍了客户自己设置的限额的作用。

### 读取客户的关联状态

如需从您的后端读取客户的关联状态，请在 `oauth_app_id` 中传入您应用的客户端 ID 来请求该客户的用户配置文件，并发送 `user-profiles-search-2026-08-30` beta 标头。以您的操作员服务账户（步骤 3）身份进行身份验证，使用其访问令牌或绑定到该账户的 API 密钥。访问令牌必须来自具有 `workspace:developer` 作用域的联合规则，这是默认设置。

<CodeGroup exclude="shell:CLI, python, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  curl "https://api.anthropic.com/v1/user_profiles/uprof_01HV9Z?oauth_app_id=YOUR_CLIENT_ID" \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-search-2026-08-30"
  ```
</CodeGroup>

在响应中，`connection_status` 描述该用户配置文件与您应用的关联：

| `connection_status.type` | 含义                                                                                                                                                            |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `connected`              | 该用户配置文件与您的应用之间至少有一个连接处于活跃状态。                                                                                                                                  |
| `disconnected`           | 该用户配置文件曾被关联，且所有连接均已终止。`last_revoked_at` 表示最后一个连接终止的时间。`last_revoked_reason.type` 表示终止原因，例如 `revoked_by_admin`（客户组织的管理员）或 `revoked_by_app_owner`（您的组织，包括您的后端）。 |
| `not_connected`          | 该用户配置文件从未关联到您的应用。                                                                                                                                             |

请将值为 `null` 的 `connection_status` 或您无法识别的 `type` 视为未知。将您无法识别的原因视为 `other`。`oauth_connections.data` 列出了连接本身。

`connected` 并不保证请求会成功。它仅表示有一个连接处于活跃状态。Claude API 仍会检查每个设置了 `bill_to: "user_profile"` 的请求。它可能会拒绝该请求，例如因为客户的组织不属于[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#eligible-customers-and-endpoints)或已达到支出限制。因此，请在每个请求上处理[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#expected-errors)，即使用户配置文件显示为 `connected` 也是如此。

400 表示缺少 beta 标头或 `oauth_app_id` 不符合客户端 ID 的格式。403 表示调用方不是 `oauth_app_id` 中应用的操作员服务账户。404 表示该用户配置文件不存在。如果收到 503，请使用退避策略重试。

### 取消关联客户

如需在 Console 中取消关联客户，请使用 [User profiles 页面](https://platform.claude.com/user-profiles)上的列表，或用户配置文件详情页面右上角的 **Unlink** 按钮。

如需从您的后端取消关联客户，请使用与代码兑换相同的客户端身份验证，将您在步骤 8 中存储的连接 ID 发送到[撤销端点](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#revocation)。如需改为通过用户配置文件指定连接，请发送 `user_profile_id` 以代替 `connection_id`。

<CodeGroup exclude="shell:CLI">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/revoke \
    -H "content-type: application/json" \
    -d @- <<'JSON'
  {
    "connection_id": "ocon_01HV9Z",
    "client_id": "YOUR_CLIENT_ID",
    "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
    "client_assertion": "YOUR_CLIENT_ASSERTION_JWT"
  }
  JSON
  ```

  ```python Python
  def revoke_connection(connection_id: str) -> list[str]:
      response = httpx.post(
          f"{AUTH}/v1/oauth/revoke",
          json={
              "connection_id": connection_id,
              "client_id": CLIENT_ID,
              "client_assertion_type": JWT_BEARER,
              "client_assertion": client_assertion(),
          },
      )
      response.raise_for_status()
      # 此请求结束的连接 ID。如果连接已结束
      # 或 ID 未知，则列表为空。
      return response.json()["revoked_connection_ids"]
  ```

  ```typescript TypeScript
  // 没有 SDK 支持 auth.claude.com，因此本示例直接调用它。
  import { readFile } from "node:fs/promises";

  // 您在步骤 8 中存储的连接 ID。
  const connectionId = "ocon_01HV9Z";

  // 客户端断言：身份令牌文件中的 JWT。由于该文件会轮换，请在每次请求时重新读取。
  // 如果您应用的客户端身份验证所需的 audience 与该 JWT 的不同，请使用第二个令牌文件。
  const clientAssertion = (
    await readFile(process.env.ANTHROPIC_IDENTITY_TOKEN_FILE!, "utf8")
  ).trim();

  const response = await fetch("https://auth.claude.com/v1/oauth/revoke", {
    method: "POST",
    redirect: "error",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({
      connection_id: connectionId,
      client_id: "YOUR_CLIENT_ID",
      client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      client_assertion: clientAssertion
    })
  });
  if (!response.ok) {
    throw new Error(`revoke failed: ${response.status}`);
  }

  const { revoked_connection_ids: revokedConnectionIds } = (await response.json()) as {
    revoked_connection_ids: string[];
  };
  console.log(revokedConnectionIds); // IDs of the connections this request ended. The list is empty if the connection had already ended or the ID is unknown.
  ```

  ```csharp C#
  // 没有 SDK 支持 auth.claude.com，因此本示例直接调用它。
  using System.Net.Http.Json;
  using System.Text.Json;

  // 您在第 8 步中存储的连接 ID。
  var connectionId = "ocon_01HV9Z";

  // 客户端断言：即身份令牌文件中的 JWT。由于该文件会轮换，请在每次请求时重新读取。
  // 如果您应用的客户端身份验证所需的 audience 与该 JWT 中的不同，请使用第二个令牌文件。
  var tokenFile = Environment.GetEnvironmentVariable("ANTHROPIC_IDENTITY_TOKEN_FILE")!;
  var clientAssertion = (await File.ReadAllTextAsync(tokenFile)).Trim();

  using var http = new HttpClient();
  using var response = await http.PostAsJsonAsync(
      "https://auth.claude.com/v1/oauth/revoke",
      new Dictionary<string, string>
      {
          ["connection_id"] = connectionId,
          ["client_id"] = "YOUR_CLIENT_ID",
          ["client_assertion_type"] = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
          ["client_assertion"] = clientAssertion,
      }
  );
  response.EnsureSuccessStatusCode();

  var result = await response.Content.ReadFromJsonAsync<JsonElement>();
  Console.WriteLine(result.GetProperty("revoked_connection_ids")); // IDs of the connections this request ended. The list is empty if the connection had already ended or the ID is unknown.
  ```

  ```go Go
  // 没有 SDK 支持 auth.claude.com，因此本示例直接调用它。

  import (
  	"bytes"
  	"encoding/json"
  	"fmt"
  	"net/http"
  	"os"
  	"strings"
  )

  func main() {
  	// 您在第 8 步中存储的连接 ID。
  	connectionID := "ocon_01HV9Z"

  	// 客户端断言：身份令牌文件中的 JWT。由于该文件会轮换，请在每次请求时读取。
  	// 如果您应用的客户端身份验证所需的 audience 与该 JWT 的不同，请使用第二个令牌文件。
  	token, err := os.ReadFile(os.Getenv("ANTHROPIC_IDENTITY_TOKEN_FILE"))
  	if err != nil {
  		panic(err)
  	}

  	body, err := json.Marshal(map[string]string{
  		"connection_id":         connectionID,
  		"client_id":             "YOUR_CLIENT_ID",
  		"client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
  		"client_assertion":      strings.TrimSpace(string(token)),
  	})
  	if err != nil {
  		panic(err)
  	}

  	response, err := http.Post("https://auth.claude.com/v1/oauth/revoke", "application/json", bytes.NewReader(body))
  	if err != nil {
  		panic(err)
  	}
  	defer response.Body.Close()
  	if response.StatusCode < 200 || response.StatusCode > 299 {
  		panic(fmt.Sprintf("revoke failed: %d", response.StatusCode))
  	}

  	var result struct {
  		RevokedConnectionIDs []string `json:"revoked_connection_ids"`
  	}
  	if err := json.NewDecoder(response.Body).Decode(&result); err != nil {
  		panic(err)
  	}
  	fmt.Println(result.RevokedConnectionIDs) // IDs of the connections this request ended. The list is empty if the connection had already ended or the ID is unknown.
  }
  ```

  ```java Java
  // 没有 SDK 覆盖 auth.claude.com，因此本示例直接调用它。Java SDK 会引入 Jackson。
  import com.fasterxml.jackson.databind.ObjectMapper;
  import java.net.URI;
  import java.net.http.HttpClient;
  import java.net.http.HttpRequest;
  import java.net.http.HttpResponse;
  import java.nio.file.Files;
  import java.nio.file.Path;

  void main() throws Exception {
      // 您在第 8 步中存储的连接 ID。
      String connectionId = "ocon_01HV9Z";

      // 客户端断言：即身份令牌文件中的 JWT。每次请求时都要重新读取，因为该文件会轮换。
      // 如果您应用的客户端身份验证所需的 audience 与该 JWT 的不同，请使用第二个令牌文件。
      String clientAssertion = Files.readString(
          Path.of(System.getenv("ANTHROPIC_IDENTITY_TOKEN_FILE"))).strip();

      var mapper = new ObjectMapper();
      String body = mapper.createObjectNode()
          .put("connection_id", connectionId)
          .put("client_id", "YOUR_CLIENT_ID")
          .put("client_assertion_type", "urn:ietf:params:oauth:client-assertion-type:jwt-bearer")
          .put("client_assertion", clientAssertion)
          .toString();

      var request = HttpRequest.newBuilder(URI.create("https://auth.claude.com/v1/oauth/revoke"))
          .header("content-type", "application/json")
          .POST(HttpRequest.BodyPublishers.ofString(body))
          .build();
      var response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
      if (response.statusCode() / 100 != 2) {
          throw new IllegalStateException("revoke failed: " + response.statusCode());
      }

      // 此请求终止的连接的 ID。如果连接已终止或 ID 未知，则列表为空。
      IO.println(mapper.readTree(response.body()).get("revoked_connection_ids"));
  }
  ```

  ```php PHP
  // 没有 SDK 支持 auth.claude.com，因此本示例直接调用它。

  // 您在步骤 8 中存储的连接 ID。
  $connectionId = 'ocon_01HV9Z';

  // 客户端断言：身份令牌文件中的 JWT。每次请求时都要读取，因为该文件会轮换。
  // 如果您应用的客户端身份验证所需的 audience 与该 JWT 的不同，请使用第二个令牌文件。
  $clientAssertion = trim(
      file_get_contents((string) getenv('ANTHROPIC_IDENTITY_TOKEN_FILE'))
          ?: throw new \RuntimeException('the file named by ANTHROPIC_IDENTITY_TOKEN_FILE cannot be read or is empty')
  );

  $body = json_encode([
      'connection_id' => $connectionId,
      'client_id' => 'YOUR_CLIENT_ID',
      'client_assertion_type' => 'urn:ietf:params:oauth:client-assertion-type:jwt-bearer',
      'client_assertion' => $clientAssertion,
  ], JSON_THROW_ON_ERROR);

  $context = stream_context_create(['http' => [
      'method' => 'POST',
      'header' => 'content-type: application/json',
      'content' => $body,
  ]]);
  $response = file_get_contents('https://auth.claude.com/v1/oauth/revoke', context: $context);
  if ($response === false) {
      throw new \RuntimeException('revoke failed');
  }

  /** @var array{revoked_connection_ids: list<string>} $result */
  $result = json_decode($response, associative: true, flags: JSON_THROW_ON_ERROR);
  // 此请求终止的连接的 ID。如果连接已终止或 ID 未知，则列表为空。
  echo implode(', ', $result['revoked_connection_ids']), PHP_EOL;
  ```

  ```ruby Ruby
  # 没有 SDK 支持 auth.claude.com，因此本示例直接调用它。
  require "json"
  require "net/http"

  # 您在第 8 步中存储的连接 ID。
  connection_id = "ocon_01HV9Z"

  # 客户端断言：身份令牌文件中的 JWT。由于该文件会轮换，每次请求时都需重新读取。
  # 如果您应用的客户端身份验证所需的 audience 与该 JWT 的不同，请使用第二个令牌文件。
  client_assertion = File.read(ENV.fetch("ANTHROPIC_IDENTITY_TOKEN_FILE")).strip

  response = Net::HTTP.post(
    URI("https://auth.claude.com/v1/oauth/revoke"),
    {
      connection_id: connection_id,
      client_id: "YOUR_CLIENT_ID",
      client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      client_assertion: client_assertion
    }.to_json,
    "content-type" => "application/json"
  )
  response.value # raises on an error

  # 此请求结束的连接的 ID。如果连接已结束或 ID 未知，则列表为空。
  puts(JSON.parse(response.body).fetch("revoked_connection_ids"))
  ```
</CodeGroup>

`auth.claude.com` 会以 HTTP 200 响应，并返回 `revoked_connection_ids`，即此请求终止的连接的 ID。如果连接已终止，该列表为空，因此您可以安全地重复发送带有 `connection_id` 的请求。如果 ID 未知，该列表也为空，因此空列表并不能证明该连接曾经存在。有关其他参数，请参阅[撤销](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#revocation)，另请参阅[撤销端点的错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#errors-from-the-revocation-endpoint)。

如需改为通过 Claude API 取消关联客户，请在 `oauth_app_id` 中将您应用的客户端 ID 发送到 `POST /v1/user_profiles/{id}/unlink`。这会终止与使用 `user_profile_id` 调用撤销端点时相同的连接。这两种请求的区别如下：

|             | 撤销端点                      | 取消关联                                                                                    |
| ----------- | ------------------------- | --------------------------------------------------------------------------------------- |
| **主机**      | `auth.claude.com`         | `api.anthropic.com`                                                                     |
| **身份验证**    | 您应用的客户端身份验证（步骤 2）         | 您的操作员服务账户（步骤 3）：其访问令牌，或绑定到该账户的 API 密钥。访问令牌必须来自具有 `workspace:developer` 作用域的联合规则，这是默认设置。 |
| **Beta 标头** | 无                         | `user-profiles-search-2026-08-30`                                                       |
| **终止的内容**   | 一个连接，或您的应用在某个用户配置文件上的所有连接 | 您的应用在某个用户配置文件上的所有连接                                                                     |

<CodeGroup exclude="shell:CLI, python, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  curl https://api.anthropic.com/v1/user_profiles/uprof_01HV9Z/unlink \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-search-2026-08-30" \
    -H "content-type: application/json" \
    -d '{"oauth_app_id": "YOUR_CLIENT_ID"}'
  ```
</CodeGroup>

响应包含 `revoked_connection_ids`，即此请求终止的连接的 ID。如果没有处于活跃状态的连接，该列表为空。400 表示缺少 `oauth_app_id` 或其不符合客户端 ID 的格式。403 表示调用方不是 `oauth_app_id` 中应用的操作员服务账户。404 表示该用户配置文件不存在或缺少 beta 标头。

连接终止后，该用户配置文件显示为 **Unlinked**，为该客户设置了 `bill_to: "user_profile"` 的请求会收到 `user_profile_not_billable`。您仍可以使用 `bill_to: "self"` 以您自己的成本为该客户提供服务。[关联之后](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#after-connecting)介绍了您的客户会看到什么，[关于关联和取消关联的电子邮件](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#emails-to-your-customer)介绍了哪种取消关联方式会向客户发送电子邮件。

如果取消关联请求返回 503，它可能只终止了部分连接。在首次请求后的一天内，请使用退避策略重复该请求，直到收到 200。一天之后，请改为将您在首次请求之前存储的每个 `connection_id` 发送到撤销端点。即使 `revoked_connection_ids` 为空，这也会完成剩余的工作。在那么晚的时候重复请求，可能会返回带有空列表的 200，并跳过那些已终止但仍留有令牌或计费访问权限的连接。如果客户自首次请求以来可能已再次关联，也请以这种方式发送 `connection_id`，因为重复请求也会终止新连接。

### 将客户关联到其他 Claude 组织

如需将客户的用户配置文件关联到其他 Claude 组织，请先取消关联该客户，然后再次关联。

取消关联后，您计费给该客户的请求会被拒绝。这种情况会持续到用户为另一个组织进行批准且您在步骤 8 中兑换代码为止。当该用户准备好进行批准时，请按以下步骤操作：

1. 如果您的客户已在其 [Linked platforms](https://platform.claude.com/linked-platforms) 页面上取消了与您应用的关联，请跳过此步骤。否则，请按照[取消关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#disconnecting-a-customer)中的说明取消关联该客户。之后，该用户配置文件会在您的 [User profiles](https://platform.claude.com/user-profiles) 页面上显示为 **Unlinked**。
2. 使用同一个用户配置文件再次执行[步骤 5 到 8](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#runtime-step-2)。对于[步骤 7](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#the-administrator-approves)，用户需要一个在另一个组织中具有管理员或账单角色的账户。让该用户为该组织进行批准。
3. 存储新的连接 ID 以替换之前的 ID。如果您使用连接的令牌，请同样用新令牌替换旧令牌。

在该客户被取消关联之前，批准页面只允许用户为已关联的组织进行批准。

支出上限属于某个客户组织与您的应用，因此第一个组织的上限不会沿用。[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#spend-caps)介绍了相关规则。

## 预期错误

当请求设置了 `bill_to: "user_profile"` 时，Anthropic 要么向您的客户计费，要么拒绝该请求。它绝不会静默地改为向您计费。在 `/v1/messages` 上，以下列表中的代码位于 `error.details.error_code` 中。在 `/v1/chat/completions` 上，您会收到相同的 HTTP 状态和 OpenAI 格式的消息，但不含该代码：

* `403 permission_error, user_profile_not_billable`：Anthropic 无法就此请求向该客户计费，且响应中不会说明原因。不可重试，但新连接建立后的前 10 秒内除外。在要求客户重新关联之前，请参阅[`user_profile_not_billable` 的原因](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#user-profile-not-billable)：重新关联只能解决其中部分原因。您仍可以通过使用 `bill_to: "self"` 重新发送请求，以您自己的成本为该客户提供服务。在 Anthropic SDK 中，请通过 403 状态对应的[错误类](https://platform.claude.com/docs/zh-CN/api/errors#sdk-error-types)处理 `user_profile_not_billable`，即 `PermissionDeniedError` (java, php: `PermissionDeniedException`; csharp: `AnthropicForbiddenException`; go: `apierr.StatusCode == http.StatusForbidden`)。它也会匹配其他 403 错误，因此还需检查错误代码。

* `402 billing_error, linked_organization_billing_unavailable`：Anthropic 当前无法向客户的组织计费（例如，该组织已被暂停，或在 Claude 开始回复之后其额度用尽）。响应中不会说明原因。在请求到达时没有额度的客户会改为收到 `linked_organization_spend_limit_reached`。不可重试。请告知客户检查其 Claude Platform 账户。您仍可以通过使用 `bill_to: "self"` 重新发送请求，以您自己的成本为其提供服务。

* `402 billing_error, linked_organization_spend_limit_reached`：客户的组织已达到其设置的支出限制。该限制可能是适用于您的应用的[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#spend-caps)，也可能是整个组织的每月支出限制。当该组织没有可支付该请求的额度时，Claude API 也会发送相同的代码。响应中不会说明是哪一种。该请求在 Claude 开始回复之前即被拒绝。不可重试。您仍可以通过使用 `bill_to: "self"` 重新发送请求，以您自己的成本为该客户提供服务。请让客户在 Claude Console 中检查以下各项：

  * **适用于您应用的支出上限：** 在 [Linked platforms](https://platform.claude.com/linked-platforms) 下您的应用页面中，客户可以设置或提高支出上限。在此处设置的上限将替代其为所有已关联平台设置的任何默认上限。
  * **组织的每月支出限额：** 在[支出限额](https://platform.claude.com/settings/billing#spend-limits)下，客户可以提高该限额。
  * **额度余额：** 在其[账单](https://platform.claude.com/settings/billing)页面上，客户可以查看其额度余额并购买额度。购买可能需要一些时间才能生效，因此购买后不久发送的请求仍可能收到此 402。如果出现这种情况，请稍后再试。某些额度无法支付这些请求：请参阅[哪些额度可以付费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#which-credits-can-pay)。

* `503 overloaded_error`：Anthropic 目前无法确认是否可以向客户计费。请使用退避策略重试，并遵循 `retry-after` 标头。该请求未被处理，也未被计费。

* `400 invalid_request_error, user_profile_required`：您设置了 `bill_to: "user_profile"`，但未提供 `anthropic-user-profile-id` 标头。此类请求需要该标头，除非它使用连接的访问令牌且该标头为可选。如需了解该标头是否为可选，请参阅[使用连接的访问令牌调用 Claude](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#call-claude-with-a-connections-access-token) 中的说明。

* `400 invalid_request_error`，不含错误代码：`anthropic-user-profile-id` 标头中的 ID 格式错误，或不是您组织的用户配置文件。

* `400 invalid_request_error`，不含错误代码：请求使用连接的访问令牌进行身份验证，并且设置了 `bill_to: "self"`，或指定了该令牌所限定的用户配置文件以外的其他用户配置文件。请参阅[使用连接的访问令牌调用 Claude](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#call-claude-with-a-connections-access-token)。

在 `/v1/messages` 和 `/v1/chat/completions` 上，连接的访问令牌无法发送由您付费的请求。如需使用 `bill_to: "self"` 重新发送被拒绝的请求，请从您自己的后端使用您操作员服务账户自己的凭据进行身份验证。[使用连接的访问令牌调用 Claude](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#call-claude-with-a-connections-access-token) 列出了令牌本身会遭到的拒绝情况。

成功的响应不会说明向谁计费，因此请自行记录您使用 `bill_to: "user_profile"` 发送了哪些请求。

### `user_profile_not_billable` 的原因

请按以下顺序检查：

* **您的请求。** 请求必须以您 OAuth 应用的操作员服务账户（步骤 3）身份进行身份验证。[连接的访问令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#oauth-tokens-for-each-connection)也以该服务账户身份进行身份验证。以任何其他方式进行身份验证的请求都会被拒绝，例如以其他服务账户身份进行身份验证，或使用未绑定到该服务账户的 API 密钥。如果 `anthropic-user-profile-id` 标头指定的是您某个未关联的配置文件，该请求会被拒绝。如果凭据或标头有误，请更正请求。新建连接无济于事。
* **连接。** 客户尚未通过此应用进行关联，或者连接已终止：其组织或您组织的管理员终止了该连接，或者您的后端终止了该连接。已终止的连接会在您的 [User profiles](https://platform.claude.com/user-profiles) 页面上显示为 **Unlinked**。仅通过您的测试应用关联的配置文件并未关联到您的生产应用。在客户完成关联之前，请停止向该客户计费。连接的访问令牌仅限于该连接，因此在客户重新关联后，请使用新连接的访问令牌。如果您的应用已[关闭仅操作员身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#turn-off-operator-only-authentication)，则您只能使用客户连接的访问令牌向其计费。此时，对于您未持有任何有效 `billing:attribution` 连接令牌的客户，虽然其显示为 **Linked**，但仍必须按照[再次关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connect-a-customer-again)中的说明重新批准。
* **客户的组织。** 它必须是[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#eligible-customers-and-endpoints)。您无法从您这一侧看到该组织的状态。客户的新组织可以在设置好计费之前完成关联。在此之前，向其计费的请求会收到此 403，而不是额度用尽时的 402。请让客户在 Claude Console 中的[账单](https://platform.claude.com/settings/billing)页面上购买额度，然后再次发送请求。购买可能需要一些时间才能生效，因此如果您仍收到此 403 或那个额度用尽时的 402（`linked_organization_spend_limit_reached`），请稍后再试。如果这样仍无济于事，请咨询您的 Anthropic 客户团队。

新连接通常会立即生效，但在步骤 8 的兑换之后可能需要大约 10 秒。如果请求在这 10 秒内因 `user_profile_not_billable` 被拒绝，请再次发送。此后，在原因得到解决之前，等待或重试都不会改变结果。请记录每个被拒绝请求的完整 `error.message` 和 `request-id` 响应标头。如果连接显示为 **Linked** 的客户仍被拒绝，且您的请求正确无误，请将被拒绝请求的 `request-id` 发送给您的 Anthropic 客户团队，以便 Anthropic 查明其被拒绝的原因。

## 客户视角

### 关联

在步骤 6 中您的应用重定向用户之后，您的用户在 `auth.claude.com` 上进行批准。[步骤 7](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#the-administrator-approves) 列出了批准页面的各个界面。每个界面都会显示您应用的名称，并带有一个 **Cancel** 按钮。用户在每个界面上的操作如下：

* **Select an organization：** 用户选择要关联其哪个 Claude 组织。仅当用户有多个组织可供选择时，页面才会显示此界面。
* **How billing works：** 用户阅读该界面并点击 **Continue**。
* **Set a monthly spend limit：** 用户以整数美元为您的应用输入一个限额并点击 **Continue**。金额字段初始为空，因此页面不会替用户选择限额。**Continue without a limit** 不会保存新的限额，客户之前保存的上限或默认值仍然适用。用户输入的限额即[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#spend-caps)中所述的支出上限。当您的应用处于测试阶段时，此界面有所不同：请参阅[应用处于测试阶段时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#while-your-app-is-in-testing)。
* **最后一个界面：** 此界面列出您的应用将能够执行的操作，包括用户所选的限额。用户点击 **Link and authorize**。

批准页面使用浏览器中当前登录的 Claude 账户，并且只提供该账户的 Claude 组织。该账户可能不是您的用户预期的账户，例如是个人账户而不是工作账户。您的应用无法选择账户。OpenID Connect 参数 `prompt=select_account` 和 `login_hint` 不会改变该页面使用哪个账户。

### 关联之后

您的客户在 Claude Console 的 [Linked platforms](https://platform.claude.com/linked-platforms) 页面上管理其连接。他们可以随时取消与您应用的关联。客户取消关联时，Anthropic 不会通知您：您不会收到任何 webhook 事件或电子邮件。如需检查客户的状态，请参阅[读取客户的关联状态](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#reading-link-status)。

客户取消关联后，您计费给该客户的请求会被拒绝，并返回 403 `user_profile_not_billable`（请参阅[`user_profile_not_billable` 的原因](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#user-profile-not-billable)）。该客户的用户配置文件会在 Console 中您的 [User profiles](https://platform.claude.com/user-profiles) 页面上显示为 **Unlinked**。如需重新关联，请使用该客户现有的用户配置文件再次执行[步骤 5 到 8](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#runtime-step-2)。

### 关于关联和取消关联的电子邮件

Anthropic 关于关联和取消关联的电子邮件会发送给您客户组织中具有管理员或账单角色的成员。每封电子邮件都会链接到客户的 Linked platforms 页面。Anthropic 是否发送电子邮件取决于具体事件：

* **客户关联您的应用时：** 当您在步骤 8 中兑换代码时，Anthropic 会发送电子邮件。对于您从未兑换其代码的批准，它不会发送任何邮件。当客户仍处于关联状态而用户再次批准时，它也不会发送任何邮件。
* **客户取消与您应用的关联时：** Anthropic 会发送电子邮件。该邮件不会指明是谁取消了与您应用的关联。
* **您从后端取消关联客户时：** Anthropic 会发送电子邮件。
* **您在 Console 中取消关联客户时：** Anthropic 不会发送电子邮件。

Anthropic 不保证电子邮件一定会发送或送达。取消关联客户时，请您亲自告知客户。请勿依赖这些电子邮件来告知客户其已完成关联。

### 支出上限

您的客户可以为您的应用设置以美元（USD）计的每月支出上限。批准页面将此上限称为每月支出限额，进行关联的用户可以在该页面上设置它（请参阅[关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#connecting)）。它独立于组织的**每月支出限额**，后者适用于整个组织。您无法读取客户的上限，也无法查看其已使用了多少。

当客户通过您的应用产生的支出达到上限时，Claude API 会以 402 拒绝您计费给该客户的请求。请参阅[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#expected-errors)。拒绝会持续到下个月第一天的 00:00 UTC，或直到客户提高或移除该上限。对上限的更改可能需要大约一分钟才能生效。

关联之后，客户组织中具有管理员或账单角色的成员可以在 Console 的 [Linked platforms](https://platform.claude.com/linked-platforms) 页面上管理上限：

* **设置或更改上限：** 点击您的应用，然后点击 **Set spend cap** 或 **Edit spend cap**。
* **设置默认值：** 在 **Default spend cap** 下，点击 **Set default** 或 **Edit**。每个没有自身上限的已关联应用每月最多可以支出该默认值。这些应用不共享该额度。
* **移除上限：** 点击您的应用，点击 **Edit spend cap**，然后点击 **Remove cap**。之后您的应用将使用默认值。如果没有默认值，您的应用就没有上限。

最小上限为 $0 USD，这会拒绝您计费给该客户的每个请求。Console 接受整美元金额，实际上没有最大值。上限按标价计算令牌费用和网络搜索费用，不计入任何协商折扣。客户的账单金额可能与上限所计算的金额不同。

上限可以高于组织的**每月支出限额**，客户在[支出限额](https://platform.claude.com/settings/billing#spend-limits)下设置该限额。当达到其中任何一项时，Claude API 都会拒绝您计费给该客户的请求。

#### 应用处于测试阶段时

当您的应用处于测试中、审核中或未获批准状态时，只要有人关联您的应用或更改其上限，就会适用以下限制：

|                                     | 测试中、审核中或未获批准   | 应用上线之后   |
| ----------------------------------- | -------------- | -------- |
| 批准页面上的每月支出限额                        | 必填，不超过 $50 USD | 可选       |
| 批准页面上的金额字段                          | 初始为 $50 USD    | 初始为空     |
| 批准页面上的 **Continue without a limit** | 不提供            | 提供       |
| 在 Console 中移除上限                     | 不可行            | 可行       |
| 客户可设置的最大上限                          | $50 USD        | 实际上没有最大值 |

您的应用上线后，客户在此之前保存的上限会保持不变。之后客户可以提高或移除该上限。

### 为您的客户提供支持

您需要就应用的所有方面为客户提供支持，包括您计费给他们的 Claude 用量。Anthropic 为您的客户提供批准页面以及 Claude Console 中的 Linked platforms 页面。在 Linked platforms 页面上，具有管理员或账单角色的成员可以查看您的应用本月为该组织产生的费用。他们还可以为您的应用设置每月[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#spend-caps)，以及取消与您应用的关联。关于您应用的所有其他问题都应由您来处理，因此请告知您的客户如何联系您。

## 代码片段初始化

本页面上的 Python 示例共用以下设置，您只需编写一次。它定义了：

* 导入
* 您应用的客户端 ID 和重定向 URI
* 计费 beta 标头值
* 以您的服务账户身份进行身份验证的 Claude API 客户端
* `client_assertion`，它返回您的后端发送给 `auth.claude.com` 的客户端断言

```python
# Python 示例假定已完成以下初始化和变量设置
import secrets
from urllib.parse import urlencode

import anthropic
import httpx
from anthropic.lib.credentials import IdentityTokenFile
from anthropic.types import Message
from anthropic.types.beta import BetaMessage
from authlib.oauth2.rfc7636 import create_s256_code_challenge

AUTH = "https://auth.claude.com"
# 注册 OAuth 应用后从 platform.claude.com 复制。
CLIENT_ID = "YOUR_CLIENT_ID"
REDIRECT_URI = "https://app.example.com/claude/billing/callback"
JWT_BEARER = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer"
BILLING_BETA = "user-profiles-billing-2026-10-01"

# API 客户端。SDK 会将您工作负载的身份 JWT 交换为访问
# 令牌并自动刷新，因此您无需自行处理 "Bearer <access token>"。
client = anthropic.Anthropic()

# auth.claude.com 的客户端断言：即身份 JWT 本身，每次调用时重新读取，
# 因为该文件会轮换。如果您应用的客户端身份验证
# 设置指定的受众与联合规则不同，请将此处指向
# 另一个令牌文件：IdentityTokenFile("/path/to/other/token")。
client_assertion = IdentityTokenFile()
```
