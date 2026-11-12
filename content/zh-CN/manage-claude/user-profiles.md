---
title: 用于终端客户归属的用户配置文件
url: https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles
description: 将 API 请求归属到您的平台所服务的终端客户，使安全执行措施可以仅作用于单个配置文件，而不是您的整个组织。
---

用户配置文件（简称"配置文件"）用于告知 Anthropic 每个 API 请求是为您的哪位终端客户发出的。这样，安全执行措施（例如限制或封禁）就可以仅作用于某一位终端客户，而不是您的整个组织。在这种情况下，您的其余流量不会受到影响。

如果您运营平台或网关，或转售 API 访问权限，那么您为终端客户发送的请求都会归入您的组织。Anthropic 会对 API 流量运行自动化安全分类器。如果请求上没有配置文件，单个终端客户就可能触发作用于您整个组织的执行措施。

您只需使用 `POST /v1/user_profiles` 为每个终端客户注册一次，并存储返回的 `uprof_` ID。然后，在您代表该终端客户发出的每个请求中，通过 `anthropic-user-profile-id` 标头发送该 ID。

就其本身而言，配置文件不会改变计费或定价。

## 确定为谁创建配置文件

配置文件代表什么取决于您销售 Claude 的方式：是使用您产品的个人，还是您向其转售 API 访问权限的公司。这两种情况涵盖了大多数平台。配置文件的 `access_type` 字段表明适用哪一种。

| 流量的服务对象                  | 配置文件                                                                                                                                          | `access_type`                             |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| 您产品的已注册用户                | 每个用户一个，在用户注册时创建。                                                                                                                              | `application`（默认值，省略 `access_type` 时即为此值） |
| 您向其转售 API 访问权限的公司，例如作为网关 | 每个公司一个，而不是公司内的每个人一个。                                                                                                                          | `passthrough`（非默认值，因此您必须发送它）              |
| 您自己的组织                   | 无。Anthropic 会将不带配置文件 ID 的请求计为您自己的使用量。请将这部分使用量放在[单独的组织](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#before-you-begin)中。 | 不适用                                       |
| 您组织中的工作区                 | 无需单独的配置文件。配置文件属于您的组织，其 ID 可用于通过该组织中任何工作区发出的请求。                                                                                                | 不适用                                       |

## 配置文件包含的内容以及 Anthropic 如何使用它

配置文件仅包含您发送的有关终端客户的信息，以及 Anthropic 设置的任何[计划状态](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles-verification-portal#after-the-end-customer-links-the-profile)或执行状态。它不包含对话内容。Anthropic 使用您发送的详细信息来归属请求、监控滥用行为、执行[使用政策](https://www.anthropic.com/legal/aup)，以及支持申请和访问受信任访问计划。Anthropic 不会验证这些详细信息。

Anthropic 根据[隐私政策](https://www.anthropic.com/legal/privacy)处理配置文件。

## 开始之前

您可以在以下任一位置创建和管理配置文件：

* Claude API 上的组织。
* [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上的组织。调用主体需要具有[用户配置文件 IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#user-profiles)权限。
* [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上的组织，您可以在其中创建、检索、列出和更新配置文件。请使用 [Claude Models on Google Cloud (Anthropic managed) 的 SDK 客户端](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#install-an-sdk)的最新版本，或直接调用[组织级基础 URL](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#making-requests)，并使用 Google Cloud 凭据而不是 API 密钥。

Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 没有 `/v1/user_profiles` 端点。如果您的推理在这些平台上运行，请在 Claude API 或 Claude Platform on AWS 上的组织中创建配置文件。在 Amazon Bedrock 上，配置文件仅适用于 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)。请参阅[在 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 上归属请求](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#attribute-requests-on-amazon-bedrock-and-google-cloud)。

如果您的推理运行在[托管在 Anthropic 上](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry#hosting-options)的 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 部署中，请在 Claude API 上的组织中创建配置文件。Anthropic 必须先将您的 Foundry 资源关联到该组织，您的 Foundry 请求才能使用这些配置文件 ID。如需申请关联，请联系您的 Anthropic 客户团队。`/v1/user_profiles` 端点在 Microsoft Foundry 上不可用。

请将您为终端客户发送的流量放在一个组织中，将您自己的内部使用放在另一个组织中。配置文件属于组织而不是工作区，因此在同一组织中使用不同的工作区是不够的。只有用于终端客户流量的组织需要用户配置文件。要在 Claude API 上创建另一个组织，请参阅 Claude 帮助中心中[在 Claude Console 中创建和管理工作区](https://support.claude.com/en/articles/9796807)一文的"How to create a new organization"（如何创建新组织）部分。

示例使用 API 密钥调用 Claude API。您不需要 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。在 Claude Platform on AWS 上，请使用其基础 URL 和身份验证方式。

## 为每个终端客户创建配置文件

为您向 Claude 发送其流量的每个终端客户创建一个配置文件。请在您的客户入驻流程中创建一次。`POST /v1/user_profiles` 不是幂等的，因此每次调用都会创建一个新的配置文件，即使 `reference_id` 与现有配置文件相同也是如此。如果您已有尚未创建配置文件的终端客户，请参阅本节后面关于注册现有客户群的指导。

以下示例注册了一家通过网关购买 API 访问权限的公司，因此将 `access_type` 设置为 `passthrough`。对于 `application` 配置文件，请省略 `access_type`，并在 `name` 和 `external_user_details` 中描述单个终端用户，而不是公司。[确定为谁创建配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#decide-who-gets-a-profile)比较了这两种访问类型。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/user_profiles \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "name": "Acme Analytics, Inc.",
      "access_type": "passthrough",
      "external_user_details": {
        "reference_id": "cust-8f2a1c",
        "country": "US"
      }
    }'
  ```

  ```bash CLI
  ant user-profiles create \
    --name "Acme Analytics, Inc." \
    --access-type passthrough \
    --external-user-details '{reference_id: cust-8f2a1c, country: US}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  profile = client.user_profiles.create(
      name="Acme Analytics, Inc.",
      access_type="passthrough",
      external_user_details={"reference_id": "cust-8f2a1c", "country": "US"},
  )

  print(profile.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const profile = await client.userProfiles.create({
    name: "Acme Analytics, Inc.",
    access_type: "passthrough",
    external_user_details: { reference_id: "cust-8f2a1c", country: "US" }
  });

  console.log(profile.id);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var profile = await client.UserProfiles.Create(new()
  {
      Name = "Acme Analytics, Inc.",
      AccessType = UserProfileAccessType.Passthrough,
      ExternalUserDetails = new()
      {
          ReferenceID = "cust-8f2a1c",
          Country = "US"
      }
  });

  Console.WriteLine(profile.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  profile, err := client.UserProfiles.New(context.Background(), anthropic.UserProfileNewParams{
  	Name:       anthropic.String("Acme Analytics, Inc."),
  	AccessType: anthropic.UserProfileAccessTypePassthrough,
  	ExternalUserDetails: anthropic.UserProfileExternalUserDetailsParams{
  		ReferenceID: anthropic.String("cust-8f2a1c"),
  		Country:     anthropic.String("US"),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(profile.ID)
  ```

  ```java Java
  import com.anthropic.models.userprofiles.UserProfileAccessType;
  import com.anthropic.models.userprofiles.UserProfileCreateParams;
  import com.anthropic.models.userprofiles.UserProfileExternalUserDetailsParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserProfileCreateParams.builder()
          .name("Acme Analytics, Inc.")
          .accessType(UserProfileAccessType.PASSTHROUGH)
          .externalUserDetails(UserProfileExternalUserDetailsParams.builder()
              .referenceId("cust-8f2a1c")
              .country("US")
              .build())
          .build();
      var profile = client.userProfiles().create(params);

      IO.println(profile.id());
  // ...
  }
  ```

  ```php PHP
  $client = new Client();

  $profile = $client->userProfiles->create(
      name: 'Acme Analytics, Inc.',
      accessType: 'passthrough',
      externalUserDetails: ['referenceID' => 'cust-8f2a1c', 'country' => 'US'],
  );

  echo $profile->id, PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  profile = client.user_profiles.create(
    name: "Acme Analytics, Inc.",
    access_type: :passthrough,
    external_user_details: { reference_id: "cust-8f2a1c", country: "US" }
  )

  puts profile.id
  ```
</CodeGroup>

响应即为新的配置文件：

```json
{
  "type": "user_profile",
  "id": "uprof_011CZkZCu8hGbp5mYRQgUmz9",
  "name": "Acme Analytics, Inc.",
  "access_type": "passthrough",
  "external_user_details": {
    "reference_id": "cust-8f2a1c",
    "onboarded_at": null,
    "account_status": null,
    "entity_type": null,
    "country": "US",
    "name_hash": null,
    "email_hash": null
  },
  "metadata": {},
  "programs": {
    "cyber": null,
    "efs": null
  },
  "created_at": "2026-03-15T10:00:00Z",
  "updated_at": "2026-03-15T10:00:00Z"
}
```

请将 `id` 存储在您的客户记录中，并让您的服务实例从那里读取它，而不是各自创建配置文件。您可以通过该 `id` 检索、更新和删除配置文件。`programs` 字段是只读的，归属不会使用它。它保存终端客户在 Anthropic 运行的每个计划中的状态。有关其取值的含义，请参阅[哪些计划适用于配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles-verification-portal#after-the-end-customer-links-the-profile)。[检查 Anthropic 是否已限制或封禁配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#check-whether-anthropic-has-restricted-or-banned-a-profile)解释了 `enforcement_status` 和 `enforcement_actions` 字段。

一个组织中的配置文件数量没有上限，但配置文件的创建受"rate limit"（速率限制）约束。在批量注册现有客户群之前，请联系您的 Anthropic 客户团队。

### 向 Anthropic 提供哪些终端客户信息

所有字段都是可选的。您可以发送 `name`、`metadata`，以及 `external_user_details` 中的以下字段：`entity_type`、`country`、`reference_id`、`onboarded_at`、`account_status`、`name_hash` 和 `email_hash`。[创建配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/create)参考文档描述了每个字段及其类型、格式和长度限制。

设置字段时请遵循以下规则：

* **未知值：** 如果您不知道某个值，请省略它，而不是发送占位值。您之后可以替换 `external_user_details` 中的值，但无法清除它。
* **公司名称：** 对于 `passthrough` 配置文件，如果您知道公司的法定名称，请在 `name` 中发送。
* **国家/地区：** 在 `country` 中发送终端客户所在的国家/地区，而不是您所在的国家/地区。
* **您自己的 ID：** 在 `reference_id` 中发送您为该终端客户使用的稳定标识符。它不必是唯一的。
* **无法共享的姓名和电子邮件地址：** 省略 `name`，改为发送 `external_user_details.name_hash`。对于电子邮件地址，请发送 `external_user_details.email_hash`。配置文件没有用于存放地址本身的字段。
* **哈希值：** 请使用您选择的函数自行计算哈希值。它们可以向 Anthropic 表明您的哪些配置文件共享同一姓名或电子邮件地址。请对每个配置文件使用相同的函数以及相同的大小写和空白规范化方式，因为只有以相同方式计算的值才能匹配。

## 归属每个请求

在您代表终端客户发出的每个请求中，都请在 `anthropic-user-profile-id` 标头中发送配置文件 ID。SDK 和 CLI 提供了一个可以为您发送该标头的参数：`user_profile_id` (java: `userProfileId`; php: `userProfileID`; csharp, go: `UserProfileID`; cli: `--user-profile-id`)。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-user-profile-id: $PROFILE_ID" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "Hello, Claude"}
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}' \
    --user-profile-id "$PROFILE_ID"
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
      user_profile_id=profile_id,
  )

  print(message)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    user_profile_id: profileId
  });

  console.log(message);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }],
      UserProfileID = profileId
  };
  var message = await client.Messages.Create(parameters);

  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  	UserProfileID: anthropic.String(profileID),
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(message)
  ```

  ```java Java
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("Hello, Claude")
          .userProfileId(profileId)
          .build();
      var message = client.messages().create(params);

      IO.println(message);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
      userProfileID: $profileId,
  );

  echo json_encode($message, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    user_profile_id: profile_id
  )

  puts message
  ```
</CodeGroup>

该标头在[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)、[Message Batches](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 和 [OpenAI SDK 兼容性](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk)请求中的工作方式相同。在批处理中，它适用于批处理中的每个请求。

Claude API、Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed) 和托管在 Anthropic 上的 Microsoft Foundry 部署都会对每个请求检查该 ID。如果 ID 格式错误或不属于发送请求的组织，API 会返回 `400` `invalid_request_error`。默认情况下，这些平台仅在来自创建该配置文件的组织的请求中接受配置文件 ID。这意味着，如果您在多个组织下发送流量，例如一个在 Claude API 上、一个在 Claude Platform on AWS 上，那么来自未创建该配置文件的组织的请求也会收到同样的 `400`。对于托管在 Anthropic 上的 Microsoft Foundry 部署，所有配置文件 ID 都会收到此 `400`，直到 Anthropic 将[您的 Foundry 资源关联](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#before-you-begin)到创建该配置文件的组织为止。

Message Batches 是个例外：创建批处理时只检查 ID 的格式。如果格式正确的 ID 不属于发送批处理的组织，批处理仍会被创建，但其中的每个请求随后都会失败，返回包含相同 `invalid_request_error` 的 `errored` 结果。

### 在 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 上归属请求

任一访问类型的配置文件 ID 也适用于 Claude Models on Google Cloud (Google managed) 和 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)。它不适用于您通过 `InvokeModel` 或 `Converse` 调用的 [Amazon Bedrock (`bedrock-runtime`)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)。您仅使用 Anthropic 组织的凭据来创建和管理配置文件。推理仍使用您现有的 AWS 或 Google Cloud 凭据。

| 平台                                                                                                                                                            | 创建配置文件                                                                                                                                       | 配置文件 ID 的发送位置                  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| Claude API                                                                                                                                                    | 是                                                                                                                                            | `anthropic-user-profile-id` 标头 |
| [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)                                                     | 是                                                                                                                                            | `anthropic-user-profile-id` 标头 |
| [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)（托管在 Anthropic 上的部署）                                 | 否。请在 Claude API 上创建配置文件，并[请 Anthropic 关联您的 Foundry 资源](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#before-you-begin)。 | `anthropic-user-profile-id` 标头 |
| [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | 是                                                                                                                                            | `anthropic-user-profile-id` 标头 |
| [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)       | 否。请在 Claude API 或 Claude Platform on AWS 上创建。                                                                                                | `anthropic-user-profile-id` 标头 |
| [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                 | 否。请在 Claude API 或 Claude Platform on AWS 上创建。                                                                                                | 请求体中的 `user_profile_id` 字段     |

在 Claude in Amazon Bedrock 上，请将 ID 放在请求体的顶层：

```json
{
  "model": "anthropic.claude-opus-5-5",
  "max_tokens": 1024,
  "user_profile_id": "uprof_011CZkZCu8hGbp5mYRQgUmz9",
  "messages": [{ "role": "user", "content": "Hello, Claude" }]
}
```

请将每个向 Claude in Amazon Bedrock 发送请求的 AWS 账户，以及每个向 Claude Models on Google Cloud (Google managed) 发送请求的 Google Cloud 项目，关联到创建您配置文件的 Anthropic 组织：

1. 该组织的管理员或所有者在 `portal.anthropic.com` 的 Verification Portal 中的 **Linked accounts（已关联账户）** 下发起关联。
2. 能够管理该 AWS 账户或 Google Cloud 项目中 IAM 的人员运行门户提供的命令。
3. 管理员或所有者在门户中完成验证。

适用于某个配置文件的计划，只有在账户完成关联且门户将其显示为 **Linked（已关联）** 之后，才会应用于该账户的请求。这些计划包括：终端客户[将配置文件关联到其组织](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles-verification-portal)后，该终端客户的组织获批的计划；对于 Cyber Verification Program 访问级别，还需要终端客户在门户中为该配置文件启用这些级别。

Claude Models on Google Cloud (Google managed) 和 Claude in Amazon Bedrock 会检查 ID 的格式，但不会查找配置文件。它们会以 `400` 拒绝格式错误的 ID。它们接受任何格式正确的 ID，因此成功的响应并不能确认该 ID 与某个配置文件匹配。不带 ID 的请求会在未归属的情况下通过，如果它触发了执行措施，该措施将作用于您的整个组织。

### 识别公司内的个人用户

`passthrough` 配置文件标识的是公司，而不是发出请求的个人。如果请求来自一个可识别的个人，请同时在 Messages 请求的 [`metadata.user_id`](https://platform.claude.com/docs/zh-CN/api/messages/create) 字段中发送该个人的标识符。这是请求自身的 `metadata` 对象，而不是配置文件的 `metadata` 字段。该标识符是可选的，它有助于 Anthropic 检测个人的滥用行为。

## 检查 Anthropic 是否已限制或封禁某个配置文件

配置文件的 `enforcement_status` 和 `enforcement_actions` 字段会报告 Anthropic 针对终端客户开启的执行措施。执行措施是指限制或封禁。每个返回配置文件的响应都包含这两个字段：创建、检索、更新，以及列表中的每个配置文件。发送这些请求时，请勿在 `anthropic-beta` 标头中包含任何 `user-profiles-` 值，因为 `user-profiles-2026-09-04` 及更早的值会省略这些字段。以下示例展示了检索到的配置文件上的这些字段，其他字段已省略：

```json
{
  "type": "user_profile",
  "id": "uprof_011CZkZCu8hGbp5mYRQgUmz9",
  "enforcement_status": "restricted",
  "enforcement_actions": [
    {
      "type": "restriction",
      "id": "enac_011CZkZEq4TnWb7xLfRgPm2v",
      "reason": "fraud_or_abuse",
      "action": null,
      "opened_at": "2026-10-02T19:30:00Z",
      "expires_at": null
    }
  ]
}
```

`enforcement_status` 用一个值概括所有开启的措施：

* `unrestricted`：没有开启的措施。
* `restricted`：至少有一项措施处于开启状态，且其中没有封禁。
* `banned`：有一项封禁处于开启状态。

当某项措施处于开启状态时，Anthropic 会拒绝部分或全部携带该配置文件 ID 的推理请求。

`enforcement_actions` 列出所有开启的措施，没有措施时为空。措施结束后会从列表中移除。每项措施包含以下字段：

* `type`：`restriction` 或 `ban`。
* `id`：措施的 ID，以 `enac_` 开头。
* `reason`：Anthropic 开启该措施的原因。取值为 `unsupported_region`、`fraud_or_abuse`、`high_harm`、`usage_policy_violation`、`account_compromise` 或 `unspecified` 之一。
* `action`：终端客户可以采取哪些操作来解除该措施；如果无法采取任何操作，则为 `null`。
* `opened_at`：Anthropic 开启该措施的时间。
* `expires_at`：该措施自动结束的时间；如果没有结束日期，则为 `null`。

要检查某个配置文件，请使用 [`GET /v1/user_profiles/{id}`](https://platform.claude.com/docs/zh-CN/api/user_profiles/retrieve) 检索它。要查找存在开启措施的配置文件，请使用 `enforcement_status[]=restricted` 和 `enforcement_status[]=banned` [列出您的配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/list)。要在措施开启或结束时收到事件，请参阅[组织 webhook](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks)。如果不使用事件，请将这两个字段与您上次看到的值进行比较。字段变化并不一定会更新 `updated_at`。

这两个字段均由 Anthropic 设置。如果创建或更新请求中包含其中任一字段，API 将返回 `400`。`enforcement_status` 与 `external_user_details.account_status` 相互独立，后者记录的是您自己对该账户的决定。

Anthropic 可能会添加新的取值。请将无法识别的 `enforcement_status` 视为 `restricted`，将无法识别的 `type` 视为 `restriction`，将无法识别的 `reason` 视为 `unspecified`，将无法识别的 `action` 视为 `null`。

在 Claude Platform on AWS 上，这两个字段不会报告执行措施：`enforcement_status` 始终为 `unrestricted`，`enforcement_actions` 始终为空。Anthropic 不会因为开启的措施而拒绝该平台上的请求。

## 保持配置文件最新

当终端客户的详细信息或账户状态发生变化时，请更新配置文件；当他们关闭账户或要求您删除其数据时，请删除配置文件。

### 更新配置文件

[`POST /v1/user_profiles/{id}`](https://platform.claude.com/docs/zh-CN/api/user_profiles/update) 仅更改您发送的字段。例如，当公司更名或您暂停某个账户时，可以使用它。

* `name` 和 `access_type` 会替换存储的值。
* 您在 `external_user_details` 中发送的每个字段都会替换存储的值。您无法清除值。API 会以 `400` 拒绝 `null`。
* `metadata` 按键合并。您发送的键会覆盖存储的值，空字符串会删除该键，您未发送的键保持不变。

以下示例将账户标记为已暂停，并删除 `trial_cohort` 元数据键。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/user_profiles/$PROFILE_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "external_user_details": {"account_status": "suspended"},
      "metadata": {"trial_cohort": ""}
    }'
  ```

  ```bash CLI
  ant user-profiles update \
    --user-profile-id "$PROFILE_ID" \
    --external-user-details '{account_status: suspended}' \
    --metadata '{trial_cohort: ""}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  profile = client.user_profiles.update(
      profile_id,
      external_user_details={"account_status": "suspended"},
      metadata={"trial_cohort": ""},
  )

  print(profile)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const profile = await client.userProfiles.update(profileId, {
    external_user_details: { account_status: "suspended" },
    metadata: { trial_cohort: "" }
  });

  console.log(profile);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var profile = await client.UserProfiles.Update(profileId, new()
  {
      ExternalUserDetails = new()
      {
          AccountStatus = UserProfileExternalUserAccountStatus.Suspended
      },
      Metadata = new Dictionary<string, string> { ["trial_cohort"] = "" }
  });

  Console.WriteLine(profile);
  ```

  ```go Go
  client := anthropic.NewClient()

  profile, err := client.UserProfiles.Update(
  	context.Background(),
  	profileID,
  	anthropic.UserProfileUpdateParams{
  		ExternalUserDetails: anthropic.UserProfileExternalUserDetailsParams{
  			AccountStatus: anthropic.UserProfileExternalUserAccountStatusSuspended,
  		},
  		Metadata: map[string]string{"trial_cohort": ""},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(profile)
  ```

  ```java Java
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.userprofiles.UserProfileExternalUserAccountStatus;
  import com.anthropic.models.userprofiles.UserProfileExternalUserDetailsParams;
  import com.anthropic.models.userprofiles.UserProfileUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserProfileUpdateParams.builder()
          .userProfileId(profileId)
          .externalUserDetails(UserProfileExternalUserDetailsParams.builder()
              .accountStatus(UserProfileExternalUserAccountStatus.SUSPENDED)
              .build())
          .metadata(UserProfileUpdateParams.Metadata.builder()
              .putAdditionalProperty("trial_cohort", JsonValue.from(""))
              .build())
          .build();
      var profile = client.userProfiles().update(params);

      IO.println(profile);
  }
  ```

  ```php PHP
  $client = new Client();

  $profile = $client->userProfiles->update(
      $profileId,
      externalUserDetails: ['accountStatus' => 'suspended'],
      metadata: ['trial_cohort' => ''],
  );

  echo json_encode($profile, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  profile = client.user_profiles.update(
    profile_id,
    external_user_details: { account_status: :suspended },
    metadata: { trial_cohort: "" }
  )

  puts profile
  ```
</CodeGroup>

### 删除配置文件

[`DELETE /v1/user_profiles/{id}`](https://platform.claude.com/docs/zh-CN/api/user_profiles/delete) 会永久删除一个配置文件以及您为其提供的所有内容，包括 `name`、`external_user_details` 和 `metadata`。它不会删除 Anthropic 针对该配置文件 ID 保留的信任与安全记录。

当终端客户关闭账户或要求您删除其数据时，请删除配置文件。在删除配置文件之前，请停止在该终端客户的请求中发送其 ID。如果终端客户在提出删除请求后仍继续使用您的服务，请创建一个新的配置文件并切换到其 ID，然后删除旧的配置文件。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/user_profiles/$PROFILE_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant user-profiles delete --user-profile-id "$PROFILE_ID"
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted = client.user_profiles.delete(profile_id)

  print(deleted.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deleted = await client.userProfiles.delete(profileId);

  console.log(deleted.id);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deleted = await client.UserProfiles.Delete(profileId);

  Console.WriteLine(deleted.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  deleted, err := client.UserProfiles.Delete(
  	context.Background(),
  	profileID,
  	anthropic.UserProfileDeleteParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(deleted.ID)
  ```

  ```java Java
  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var deleted = client.userProfiles().delete(profileId);

      IO.println(deleted.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $deleted = $client->userProfiles->delete($profileId);

  echo $deleted->id, PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  deleted = client.user_profiles.delete(profile_id)

  puts deleted.id
  ```
</CodeGroup>

响应会重复该 ID，并带有 `"type": "user_profile_deleted"`。如果删除操作返回服务器错误，请重试。重试时返回 `404` 表示第一次删除已成功。

删除后，配置文件端点会对该 ID 返回 `404`。在最多约 10 分钟内，携带已删除 ID 的推理请求仍可能成功。此后，在每个请求中检查 ID 的平台会以 `400` 拒绝该请求。Claude Models on Google Cloud (Google managed) 和 Claude in Amazon Bedrock 不会查找配置文件，因此它们仍会接受该请求。

## 后续步骤

API 参考中列出了每个端点的字段、类型和限制：[创建配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/create)、[检索配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/retrieve)、[更新配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/update)、[删除配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/delete)以及[列出配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/list)。

如果终端客户通过您的平台使用 Claude 时应适用其计划审批，请参阅[向终端客户发送 Verification Portal 链接](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles-verification-portal)。

要在 Anthropic 限制或封禁您的某个终端客户时，或在该限制或封禁结束时收到事件，请参阅[组织 webhook](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks)。

<CardGroup cols={2}>
  <Card title="身份验证" href="https://platform.claude.com/docs/zh-CN/manage-claude/authentication">
    选择您的服务向 Claude API 进行身份验证的方式。
  </Card>

  <Card title="用量和成本分析 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api">
    跟踪您组织的令牌用量和成本。
  </Card>

  <Card title="数据驻留" href="https://platform.claude.com/docs/zh-CN/manage-claude/data-residency">
    管理模型推理的运行位置和数据的存储位置。
  </Card>

  <Card title="Compliance API" href="https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api">
    审计您组织中的活动。
  </Card>
</CardGroup>
