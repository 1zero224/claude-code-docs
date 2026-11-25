---
title: 设置客户自有存储
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage
description: 在您自己的云账户中创建存储桶，在 Claude Console 中连接它，并确认您的密封的安全数据已送达其中。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

使用 Enterprise Frontier Safeguards（EFS）时，[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage)会将工作区的密封的安全数据保存在您自己云账户中的存储桶里。

<Where cloud="aws" label="Claude Platform on AWS">

请从[在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws) 开始，并使用本页面设置存储桶本身。

</Where>

<Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">

请从[在 Claude Models on Google Cloud (Anthropic managed) 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed) 开始。

</Where>

<Where bucket="s3" label="Amazon S3">

## 在 Amazon S3 上设置客户自有存储

<Where method="console" label="Claude Console">

要在 Amazon S3 中设置您的存储，请在您自己的云账户中创建资源，然后在 Claude Console 中连接它们：

1. [更新您的云组织的策略](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#if-an-aws-organization-policy-blocks-anthropic)，以允许来自选定外部账户的访问。
2. [创建您的存储桶及其加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket)。
3. [创建角色](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-roles)，以授予 Anthropic 对您存储桶的访问权限。
4. [在 Claude Console 中连接您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#connect-the-bucket)，并测试设置。

测试通过后，您可以[在工作区中使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#use-your-bucket-in-a-workspace)。

**Connect storage** 对话框会为您提供设置文件，并测试您的设置。

</Where>

<Where method="api" label="Admin API">

您可以使用 Admin API 为[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage)注册、检查并附加您的 Amazon S3 存储桶。如果您想通过脚本完成设置，或者您管理着大量工作区，Admin API 会很有用。虽然大多数步骤都可以通过 API 完成，但您仍需要从 Claude Console 获取设置文件，具体见[获取您的设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#get-the-setup-file)中的说明。

在 API 中，External Storage 是 Anthropic 对您存储桶的记录，而不是您的存储桶本身。

</Where>

## 开始之前

请确保您具备以下各项：

* **Claude Console 中的组织管理员**，负责获取设置文件并连接您的存储桶。
* **[Enterprise Frontier Safeguards 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console)。** 如果您的组织拥有该权限，Claude Console 中的[Programs](https://platform.claude.com/settings/programs)页面会显示它以及一个 **Configure** 按钮。
* **用于存储桶的云账户。** Anthropic 建议使用一个专用账户：其中只存放此存储桶，且没有任何其他 Anthropic 组织将其用于客户自有存储。您也可以[使用存放其他存储桶的账户](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)。
* **没有会删除您的存储桶、其密钥或角色的自动化机制。** 请与管理该云账户的团队确认，因为某些测试账户设有清理作业。如果有任何机制删除了其中之一，[您的存储桶就会停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)。

<Warning title="使用存储桶会将其与一个 Anthropic 组织绑定">
  当您首次使用存储桶时，您会将该存储桶、其名称、其密钥和其角色与此组织绑定在一起。首次使用是指您为工作区选择该存储桶并保存，或使用它创建工作区（在 API 中，是指第一个成功的附加请求）。请将这种绑定视为永久性的。不要计划让任何其他 Anthropic 组织将该存储桶、其名称、其密钥或其角色用于客户自有存储。
</Warning>

<Where method="api" label="Admin API">

### 对您的请求进行身份验证

<Tabs>
  <Tab title="cURL">
    本页面上的每个请求都会发送 `anthropic-version: 2026-10-21` 和 `anthropic-beta: efs-2026-10-01`。每个请求都要发送以下凭证之一：

    * **具有 `org:admin` 范围的 OAuth bearer 令牌：** 本页面上的每个请求都接受此令牌，放在 `authorization: Bearer` 标头中。要获取一个，请参阅 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。示例从 `$ANTHROPIC_AUTH_TOKEN` 中读取它。该令牌的有效期很短：如果请求返回 401 错误，请获取一个新令牌。
    * **个人密钥或服务账户密钥：** 本页面上的每个请求都接受在 `x-api-key` 中发送的此类密钥。请使用未限定于某个工作区的密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。Admin API 会拒绝限定于某个工作区的密钥，即使是针对该工作区的请求也是如此。
    * **Admin API 密钥：** 它无法附加或分离存储：这些请求会返回权限错误。External Storage 请求接受在 `x-api-key` 中发送的此类密钥。要获取一个，请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。

    对于令牌或者个人密钥或服务账户密钥，其账户必须具有管理员角色。要使用密钥，请将每个示例中的 `authorization` 标头替换为 `x-api-key: $ANTHROPIC_API_KEY`。
  </Tab>

  <Tab title="CLI">
    本页面上的每个请求都使用同一个凭证。请使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为令牌，或将 `ANTHROPIC_API_KEY` 设置为密钥。只设置其中一个：如果两者都已设置，`ant` CLI 会发送密钥。令牌的有效期很短：如果请求返回 401 错误，请获取新令牌。请勿设置 `ANTHROPIC_WORKSPACE_ID`：如果设置了它，密钥在这些请求上会收到 400 错误。

    CLI 会替您发送凭证和 `anthropic-version` 标头。在 External Storage 请求上，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置 `inference_data_privacy` 的工作区请求上，它不会发送该标头，因此请自行使用 `--beta` 传入 `efs-2026-10-01`，如附加和分离示例所示。

    当 API 拒绝请求时，CLI 会打印完整的响应体。请在其中查找 `error.details.error_code`。
  </Tab>

  <Tab title="Python">
    本页面上的每个请求都使用同一个凭证。使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论使用哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌，或将 `ANTHROPIC_API_KEY` 设置为该密钥。只设置其中一个：如果两者都已设置，SDK 会发送密钥。该令牌的有效期很短：如果请求返回 401 错误，请获取新的令牌。

    SDK 会为您发送凭证和 `anthropic-version` 标头。在 External Storage 请求中，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置了 `inference_data_privacy` 的工作区请求中它不会发送，因此请如附加和分离示例所示，自行将 `efs-2026-10-01` 传给 SDK。

    当 API 拒绝某个请求时，SDK 调用会失败并返回错误，该错误的文本包含完整的响应正文。请在其中查找 `error.details.error_code`。
  </Tab>

  <Tab title="TypeScript">
    本页面上的每个请求都使用同一个凭证。使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论使用哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌，或将 `ANTHROPIC_API_KEY` 设置为该密钥。只设置其中一个：如果两者都已设置，SDK 会发送密钥。该令牌的有效期很短：如果请求返回 401 错误，请获取新的令牌。

    SDK 会为您发送凭证和 `anthropic-version` 标头。在 External Storage 请求中，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置了 `inference_data_privacy` 的工作区请求中它不会发送，因此请如附加和分离示例所示，自行将 `efs-2026-10-01` 传给 SDK。

    当 API 拒绝某个请求时，SDK 调用会失败并返回错误，该错误的文本包含完整的响应正文。请在其中查找 `error.details.error_code`。
  </Tab>

  <Tab title="C#">
    本页面上的每个请求都使用同一个凭证。使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论使用哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌，或将 `ANTHROPIC_API_KEY` 设置为该密钥。只设置其中一个：如果两者都已设置，SDK 会发送密钥。该令牌的有效期很短：如果请求返回 401 错误，请获取新的令牌。

    SDK 会为您发送凭证和 `anthropic-version` 标头。在 External Storage 请求中，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置了 `inference_data_privacy` 的工作区请求中它不会发送，因此请如附加和分离示例所示，自行将 `efs-2026-10-01` 传给 SDK。

    当 API 拒绝某个请求时，SDK 调用会失败并返回错误，该错误的文本包含完整的响应正文。请在其中查找 `error.details.error_code`。
  </Tab>

  <Tab title="Go">
    本页面上的每个请求都使用同一个凭证。使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论使用哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌，或将 `ANTHROPIC_API_KEY` 设置为该密钥。只设置其中一个：如果两者都已设置，SDK 会发送密钥。该令牌的有效期很短：如果请求返回 401 错误，请获取新的令牌。

    SDK 会为您发送凭证和 `anthropic-version` 标头。在 External Storage 请求中，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置了 `inference_data_privacy` 的工作区请求中它不会发送，因此请如附加和分离示例所示，自行将 `efs-2026-10-01` 传给 SDK。

    当 API 拒绝某个请求时，SDK 调用会失败并返回错误，该错误的文本包含完整的响应正文。请在其中查找 `error.details.error_code`。
  </Tab>

  <Tab title="Java">
    本页面上的每个请求都使用同一个凭证。使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论使用哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌，或将 `ANTHROPIC_API_KEY` 设置为该密钥。只设置其中一个：如果两者都已设置，SDK 会发送密钥。该令牌的有效期很短：如果请求返回 401 错误，请获取新的令牌。

    SDK 会为您发送凭证和 `anthropic-version` 标头。在 External Storage 请求中，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置了 `inference_data_privacy` 的工作区请求中它不会发送，因此请如附加和分离示例所示，自行将 `efs-2026-10-01` 传给 SDK。

    当 API 拒绝某个请求时，SDK 调用会失败并返回错误，该错误的文本包含完整的响应正文。请在其中查找 `error.details.error_code`。
  </Tab>

  <Tab title="PHP">
    本页面上的每个请求都使用同一个凭证。使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论使用哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌，或将 `ANTHROPIC_API_KEY` 设置为该密钥。只设置其中一个：如果两者都已设置，SDK 会发送密钥。该令牌的有效期很短：如果请求返回 401 错误，请获取新的令牌。

    SDK 会为您发送凭证和 `anthropic-version` 标头。在 External Storage 请求中，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置了 `inference_data_privacy` 的工作区请求中它不会发送，因此请如附加和分离示例所示，自行将 `efs-2026-10-01` 传给 SDK。

    当 API 拒绝某个请求时，SDK 调用会失败并返回错误，该错误的文本包含完整的响应正文。请在其中查找 `error.details.error_code`。
  </Tab>

  <Tab title="Ruby">
    本页面上的每个请求都使用同一个凭证。使用具有 `org:admin` 范围的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。或者使用未限定于某个工作区的个人密钥或服务账户密钥（请参阅[密钥类型](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)）。无论使用哪种方式，该账户都必须具有管理员角色。请勿使用 Admin API 密钥：它无法附加或分离存储。

    将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌，或将 `ANTHROPIC_API_KEY` 设置为该密钥。只设置其中一个：如果两者都已设置，SDK 会发送密钥。该令牌的有效期很短：如果请求返回 401 错误，请获取新的令牌。

    SDK 会为您发送凭证和 `anthropic-version` 标头。在 External Storage 请求中，它还会发送 `anthropic-beta: efs-2026-10-01`。在设置了 `inference_data_privacy` 的工作区请求中它不会发送，因此请如附加和分离示例所示，自行将 `efs-2026-10-01` 传给 SDK。

    当 API 拒绝某个请求时，SDK 调用会失败并返回错误，该错误的文本包含完整的响应正文。请在其中查找 `error.details.error_code`。
  </Tab>
</Tabs>

## 注册您的存储桶

您可以在创建存储桶之前注册它。Claude Console 会使用您注册的详细信息来填写[您的设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#get-the-setup-file)。

告知 Anthropic 您计划使用哪个存储桶：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/organizations/external_storage \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: efs-2026-10-01" \
    -H "content-type: application/json" \
    -d '{
      "display_name": "Claude retention",
      "provider_config": {
        "type": "aws",
        "account_id": "111122223333",
        "bucket_name": "acme-claude-retention",
        "region": "us-east-2"
      }
    }'
  ```

  ```bash CLI
  ant beta:organization:external-storage create \
    --display-name "Claude retention" \
    --provider-config '{type: aws, account_id: "111122223333", bucket_name: acme-claude-retention, region: us-east-2}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  storage = client.beta.organization.external_storage.create(
      display_name="Claude retention",
      provider_config={
          "type": "aws",
          "account_id": "111122223333",
          "bucket_name": "acme-claude-retention",
          "region": "us-east-2",
      },
  )

  print(f"id: {storage.id}")
  print(f"status: {storage.status.type}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const storage = await client.beta.organization.externalStorage.create({
    display_name: "Claude retention",
    provider_config: {
      type: "aws",
      account_id: "111122223333",
      bucket_name: "acme-claude-retention",
      region: "us-east-2"
    }
  });

  console.log(`id: ${storage.id}`);
  console.log(`status: ${storage.status.type}`);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.ExternalStorage;

  AnthropicClient client = new();

  var storage = await client.Beta.Organization.ExternalStorage.Create(new()
  {
      DisplayName = "Claude retention",
      ProviderConfig = new BetaAwsExternalStorageConfigParam
      {
          AccountID = "111122223333",
          BucketName = "acme-claude-retention",
          Region = "us-east-2",
      },
  });

  Console.WriteLine($"id: {storage.ID}");
  Console.WriteLine($"status: {storage.Status.Type.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  storage, err := client.Beta.Organization.ExternalStorage.New(
  	context.Background(),
  	anthropic.BetaOrganizationExternalStorageNewParams{
  		DisplayName: anthropic.String("Claude retention"),
  		ProviderConfig: anthropic.BetaOrganizationExternalStorageNewParamsProviderConfigUnion{
  			OfAWS: &anthropic.BetaAWSExternalStorageConfigParam{
  				AccountID:  "111122223333",
  				BucketName: "acme-claude-retention",
  				Region:     "us-east-2",
  			},
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", storage.ID)
  fmt.Printf("status: %s\n", storage.Status.Type)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.externalstorage.BetaAwsExternalStorageConfigParam;
  import com.anthropic.models.beta.organization.externalstorage.ExternalStorageCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = ExternalStorageCreateParams.builder()
          .displayName("Claude retention")
          .providerConfig(BetaAwsExternalStorageConfigParam.builder()
              .accountId("111122223333")
              .bucketName("acme-claude-retention")
              .region("us-east-2")
              .build())
          .build();
      var storage = client.beta().organization().externalStorage().create(params);

      IO.println("id: " + storage.id());
      IO.println("status: " + storage.status().type());
  }
  ```

  ```php PHP
  $client = new Client();

  $storage = $client->beta->organization->externalStorage->create(
      displayName: 'Claude retention',
      providerConfig: [
          'type' => 'aws',
          'accountID' => '111122223333',
          'bucketName' => 'acme-claude-retention',
          'region' => 'us-east-2',
      ],
  );

  echo "id: {$storage->id}\n";
  echo "status: {$storage->status->type}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  storage = client.beta.organization.external_storage.create(
    display_name: "Claude retention",
    provider_config: {
      type: :aws,
      account_id: "111122223333",
      bucket_name: "acme-claude-retention",
      region: "us-east-2"
    }
  )

  puts "id: #{storage.id}"
  puts "status: #{storage.status.type}"
  ```
</CodeGroup>

`provider_config` 标识您的存储桶。在发送请求之前，请根据各字段的约束逐一检查：

* `type`：`aws`。
* `account_id`：拥有您存储桶的 AWS 账户的 12 位 ID。[创建并连接您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)概述了一个账户在什么情况下可以包含多个存储桶。
* `bucket_name`：全局唯一，由 3 到 63 个小写字母、数字和连字符组成，以字母或数字开头和结尾，不含点号。如果由设置文件创建您的存储桶，请勿重复使用您已拥有的存储桶的名称。
* `region`：您的存储桶所在的 AWS 区域，须为 [Anthropic 支持的区域](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket)之一。如果存储桶不在您所给的区域中，[激活您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-activate)会失败。

响应如下所示：

```json
{
  "type": "external_storage",
  "id": "estrg_01SDCCSbTxrXDpWc1phhtcfK",
  "display_name": "Claude retention",
  "provider_config": {
    "type": "aws",
    "account_id": "111122223333",
    "bucket_name": "acme-claude-retention",
    "region": "us-east-2"
  },
  "status": {
    "type": "pending_configuration"
  },
  "setup_template_version": null,
  "created_at": "2026-10-08T17:00:00Z",
  "updated_at": "2026-10-08T17:00:00Z"
}
```

请保存您存储桶的 `id`（以 `estrg_` 开头），以便在之后的步骤中使用。

该请求可能因以下原因失败：

* **您的组织未获批准：** 如果您的组织可以发出这些请求，但 Anthropic 尚未批准其使用客户自有存储，响应会告知您联系您的 Anthropic 客户团队。
* **您的组织不符合条件：** 如果 `error.details.error_code` 为 `organization_not_eligible`，则您的组织无法使用客户自有存储。
* **该存储桶已注册：** Anthropic 会拒绝对同一个存储桶的第二次注册。要查找已有的记录，请使用 `GET /v1/organizations/external_storage` 列出您的 External Storage。

</Where>

## 创建并连接您的存储桶

要设置客户自有存储，您需要在 AWS 账户中创建三样东西：您的存储桶、KMS 密钥和 IAM 角色。使用 CloudFormation 或 Terraform 时，一个文件会创建所有这些资源。使用您自己的工具时，您需要自己逐一创建。

您可以在同一个 AWS 账户中放置多个存储桶，即使这些存储桶属于不同的 Anthropic 组织。如果您这样做，或者有其他团队在该账户中工作，请记住以下几点：

* **独立的资源：** 每个组织都需要自己的存储桶和 KMS 密钥，每个存储桶都需要自己的堆栈，或者在使用 Terraform 时需要自己的文件夹。
* **还有谁在该账户中工作：** 该 AWS 账户中任何可以创建 IAM 角色、或拥有广泛的 Amazon S3 和 AWS KMS 权限的人，都可以读取您的对话记录和 Safety Case 文件、删除文件并切断 Anthropic 的访问。

### 1. 更新您的 AWS 组织的策略

在此步骤中，您需要在 AWS 组织的策略中为 Anthropic 的 AWS 账户添加例外，以便 Anthropic 能够访问您的存储桶。只有当您的 AWS 组织有一个拒绝组织外部主体执行 AWS STS 操作的 “resource control policy”（资源控制策略），即 RCP 时，才需要执行此步骤。如果在此类策略下跳过此更改，**Test setup** 会在 IAM 角色处失败。此更改通常由其他团队完成，因此最好尽早提出请求。

Anthropic 通过您在后续步骤中创建的 IAM 角色访问您的存储桶。它从其 AWS 账户 `804820989669` 代入这些角色，并以这些角色的身份在 Amazon S3 和 AWS KMS 中执行操作。在此类 RCP 下，无论角色的信任策略如何规定，AWS 都会拒绝 Anthropic 代入这些角色的请求。

#### 确认此步骤是否适用

如果您通过 **Sign up for AWS (new)** 创建了 AWS 账户，则此步骤适用。该账户属于一个由 AWS 管理其策略的 AWS 组织，这些策略包括此类 RCP。该策略是 `ManagedAccountResourceControlPolicy`，语句是 `DenyAnyoneOutsideMyOrgAndAWS`。在尝试更改该策略之前，请阅读 AWS 文档中的[您组织的托管策略](https://docs.aws.amazon.com/accounts/latest/reference/scps-and-rcps-for-projects.html)。

对于其他任何账户，请询问管理您 AWS 组织策略的团队，是否有 RCP 拒绝组织外部的主体执行 AWS STS 操作。如果没有，请跳过此更改并前往[步骤 2](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket)。

#### 将 Anthropic 的 AWS 账户添加到策略中

请让管理您 AWS 组织策略的团队将 Anthropic 的 AWS 账户添加到拒绝外部主体的语句中。在此示例中，添加的行是组织 ID 旁边的 `aws:PrincipalAccount`：

```json
{
  "Sid": "EnforceOrgIdentities",
  "Effect": "Deny",
  "Principal": "*",
  "Action": ["sts:*", "s3:*", "kms:*"],
  "Resource": "*",
  "Condition": {
    "StringNotEqualsIfExists": {
      "aws:PrincipalOrgID": "o-exampleorgid",
      "aws:PrincipalAccount": "804820989669"
    },
    "BoolIfExists": { "aws:PrincipalIsAWSService": "false" }
  }
}
```

如果您自己的语句已经在 `aws:PrincipalAccount` 下列出了账户，请将 `804820989669` 添加到该列表中，而不是再添加一行。

如果 Claude Platform 上的某个工作区还使用 AWS KMS 中的[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys)，Anthropic 会从账户 `915198916910` 调用该密钥。如果您的语句拒绝 AWS KMS 操作，请同时添加该账户。

示例中添加的行会豁免 Anthropic 的 AWS 账户中的每个主体，适用于该语句涵盖的每个操作和每个资源。更窄的例外应该就足够了，但 Anthropic 只看到过在账户范围例外的情况下 **Test setup** 通过。有关更窄的例外需要允许哪些内容，请参阅[模板参考](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#template-reference)。

<Where method="console" label="Claude Console">

#### 检查是否生效

要检查更改，请运行步骤 4 中的 **Test setup**。如果有策略阻止了 Anthropic，测试会显示[多个检查失败](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)。

</Where>

### 2. 创建您的存储桶及其 KMS 密钥

在此步骤中，您将创建存放您数据的存储桶及其 KMS 密钥。KMS 密钥是存储桶的默认加密密钥，因此它会加密 Anthropic 写入其中的内容。此密钥与您附加到工作区的[客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key)是分开的。

请在以下 AWS 区域之一中创建您的存储桶及其 KMS 密钥：`us-east-2`、`us-east-1`、`us-west-2`、`eu-central-1`、`eu-west-1`、`ap-northeast-1` 或 `ap-southeast-2`。之后您无法更改存储桶的区域。存储桶的区域决定了您的云提供商在何处存储该存储桶的对象。它不会改变[模型推理的运行位置](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#inference-geo)，也不会改变 Anthropic 处理您数据的位置。Anthropic 建议使用 `us-east-2`，因为 Anthropic 从 `us-east-2` 读取和写入您的存储桶。

设置文件会使用以下设置创建密钥和存储桶：

| 资源 | 设置 |
| ----- | ----- |
| KMS 密钥 | 与存储桶位于同一 AWS 账户和区域中的单区域密钥，已开启轮换。由您的账户管理。其策略既未指定 Anthropic，也未指定这些角色。 |
| 存储桶 | 开启版本控制。默认使用 KMS 密钥加密（通过其密钥 ARN 指定），并开启 S3 Bucket Key。阻止公共访问。关闭 ACL（`BucketOwnerEnforced`）。关闭 S3 Object Lock。 |
| 生命周期规则 | 下一个表格列出了它们的作用。 |
| S3 Inventory | 针对 `slab/` 和 `media/` 的每日报告。Amazon S3 会将它们提交到同一存储桶的 `inventory/` 下。另外两份报告涵盖 `safeguards-data/` 下的相同前缀，并存放在 `safeguards-data/inventory/` 下。 |
| 存储桶策略 | 它拒绝永久删除 `slab/` 下的对象版本，拒绝不使用 TLS 的请求，并允许 Amazon S3 提交清单报告。它还会拒绝名称以 `anthropic-efs-` 开头的所有 IAM 角色，此设置的角色除外。 |

请不要在您自己角色的名称和路径中使用 `anthropic-efs-`，包括部署该文件的角色，因为存储桶策略会拒绝这些角色。

生命周期规则执行以下操作：

| 生命周期规则作用的对象 | 执行的操作 |
| ----- | ----- |
| `slab/` 和 `media/` 下的旧版本 | 3 天后删除 |
| 未完成的分段上传 | 7 天后停止 |
| 清单报告 | 14 天后过期 |
| 残留的删除标记 | 移除 |
| 对话记录和 Safety Case 摘要 | 28 天后过期，并在此后再过 3 天删除其早期版本 |
| 其他 Safety Case 文件，包括您团队的回复 | 2,555 天（约 7 年）后过期，并将每个早期版本在其不再是当前版本后保留相同的天数 |

#### 获取您的设置文件

<Where method="console" label="Claude Console">

从 **Connect storage** 对话框的 **Create in AWS** 步骤获取设置文件及其命令。如果对话框尚未打开，请按照[步骤 4](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#connect-the-bucket) 的说明打开它，输入存储桶的详细信息，并在 **Create in AWS** 处停下。请保存文件和命令，因为测试通过后 Claude Console 不会再显示它们。如果由其他人在 AWS 中运行该文件，请将文件和命令发送给他们，并在他们完成后点击 **Test setup**。

</Where>

<Where method="api" label="Admin API">

从 Claude Console 获取设置文件及其命令，它会填入您的值。以组织管理员身份打开[数据保留](https://platform.claude.com/settings/privacy)页面，并在顶部选择**组织**。点击您注册的存储桶旁边的 **Finish setup**，并在 **Create in AWS** 处停下。按照[创建您的存储桶及其 KMS 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket)中的说明运行该文件。您也可以通过其他方式创建相同的资源，只要它们与该文件一致即可。

</Where>

请在您于对话框中输入的区域创建所有内容，即使文件的标头指定了 `us-east-2`。对于新存储桶，请只运行对话框提供给您的文件，该文件会创建七个角色。如果您已经运行过会创建固定名称角色的较早文件，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)中的"Test setup 显示 Anthropic 无法代入 verify 角色"。

<Warning title="请勿在使用固定角色名称的设置上运行七角色文件">
  如果您有一个已通过 **Test setup** 且其角色使用固定名称（例如 `anthropic-efs-ingest`）的设置，请保持该设置及其存储目标原样。请勿在该设置的 Terraform 文件夹中应用会创建七个角色的文件，因为应用会删除这些固定名称的角色并切断 Anthropic 的访问。
</Warning>

<Tabs>
  <Tab title="CloudFormation">
    #### 部署 CloudFormation 堆栈

    1. **检查 AWS 账户。** 将文件保存在一个空文件夹中。从该文件夹运行 `aws sts get-caller-identity` 并检查它显示的是否为用于您存储桶的 AWS 账户。
    2. **部署堆栈。** 在同一个终端中按显示的内容运行对话框中的部署命令，其中已填好您的值。请勿更改堆栈名称 `anthropic-storage-<RoleDigest>`，因为模板会拒绝任何其他名称。`RoleDigest` 是一个已包含在命令中的 20 个字符的值。

    **检查是否生效。** 命令会输出 `Successfully created/updated stack` 和您的堆栈名称。如果部署失败，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)。
  </Tab>

  <Tab title="Terraform">
    #### 应用 Terraform 文件

    Terraform 文件需要 Terraform 1.5 或更高版本，以及 AWS provider 的 5 或 6 版本。

    1. **检查 AWS 账户。** 将文件保存在一个空文件夹中，因为 Terraform 会加载文件夹中的每个 `.tf` 文件。运行 `aws sts get-caller-identity` 并检查它显示的是否为用于您存储桶的 AWS 账户。
    2. **应用文件。** 从该文件夹运行对话框中的命令，其中已填好您的值。请勿编辑该文件。在输入 `yes` 之前请阅读计划，本次应用及之后的每次应用都是如此；如果第一个计划显示有任何要销毁的内容，请停止。

    **检查是否生效。** 应用以 `Apply complete!` 结束，并列出其输出，包括存储桶的名称、其区域和密钥的 ARN。如果应用失败，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)。

    请保留整个文件夹，包括 `terraform.tfstate`、`.terraform/` 和 `.terraform.lock.hcl`。该文件没有 backend 块，因此 Terraform 将其状态保存在那里。如果您丢失了该状态，下一次应用会在存储桶处失败，直到您恢复状态或对这些资源运行 `terraform import`。
  </Tab>

  <Tab title="Your own tooling">
    #### 自行创建密钥和存储桶

    使用您自己的工具时，您需要基于 CloudFormation 模板进行构建。[打开 **Connect storage** 对话框的 **Create in AWS** 步骤](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#get-the-setup-file)，选择 **CloudFormation** 下载模板，然后选择 **Your own tooling** 并记下外部 ID 和角色 ARN。请从该模板复制每项设置，因为 **Test setup** 会根据该模板检查您设置的部分内容。将模板中的 `${…}` 占位符替换为您自己的值。分区请使用 `aws`。

    **创建密钥：**

    * 在您存储桶所在的 AWS 账户中、您创建存储桶的区域内创建单区域 KMS 密钥。请与模板中的密钥保持一致，该密钥是对称密钥，并已开启轮换。
    * 复制模板中的密钥策略，该策略允许您的账户管理该密钥。它还允许 Amazon S3 为此账户和此存储桶调用 `kms:GenerateDataKey`，以便 S3 Inventory 能够提交其每日报告。

    使用此步骤表格中的设置**创建存储桶**。复制模板中的生命周期规则、S3 Inventory 配置和存储桶策略，不要添加其他生命周期规则。

    在存储桶的默认加密中，请通过密钥 ARN 指定密钥。如果您通过别名、别名 ARN 或单纯的密钥 ID 指定密钥，**Test setup** 会失败。如果您的存储桶缺少模板中任一必需的生命周期规则，或有其他使对象过期的规则，它也会失败。

    **Test setup** 不会检查您设置中的以下部分，因此请自行将它们与模板进行比较：

    * 存储桶策略的其余部分。**Test setup** 只检查拒绝其他设置的角色的两条语句。
    * S3 Inventory 配置。
    * KMS 密钥的策略及其设置，例如轮换。**Test setup** 只确认 Anthropic 可以通过 Amazon S3 使用该密钥。
  </Tab>
</Tabs>

### 3. 创建 IAM 角色

此步骤介绍授予 Anthropic 对您存储桶访问权限的七个 IAM 角色。每个角色允许一类工作。每个角色的名称为 `anthropic-efs-<RoleDigest>-<ending>`，其中 `RoleDigest` 是一个特定于此设置的 20 个字符的值。

| 角色名称结尾 | Anthropic 通过它执行的操作 | 它可以对您存储桶的 KMS 密钥执行的操作 |
| ----- | ----- | ----- |
| `ingest` | 在 `slab/` 和 `media/` 下写入您的密封的安全数据。 | `kms:GenerateDataKey` 和 `kms:Decrypt` |
| `read` | 读取 `slab/`、`media/` 和 `slab-manifest/` 下的内容，并列出这三个前缀。 | `kms:Decrypt` |
| `maintenance` | 读取和写入 `slab/`、`media/` 和 `slab-manifest/` 下的对象，并为其添加删除标记。读取 `inventory/` 下的清单报告以及存储桶的配置。列出 `slab/`、`media/`、`slab-manifest/` 和 `inventory/`。可以列出存储桶中任意位置未完成的分段上传。它从不删除对象版本。 | `kms:GenerateDataKey` 和 `kms:Decrypt` |
| `media-erasure` | 列出 `media/` 并永久删除其下的对象版本，除此之外不做任何操作。 | 无 |
| `verify` | 在 `verify/` 下写入、读取和删除测试对象及其版本，并列出该处的对象版本。读取存储桶的配置。可以读取 `inventory/` 下的清单报告 manifest，并列出 `inventory/`。 | `kms:GenerateDataKey`、`kms:Decrypt` 和 `kms:DescribeKey` |
| `transcripts` | 在 `transcripts/` 下写入对话记录。列出并永久删除其版本。它无法读取对话记录。 | `kms:GenerateDataKey` 和 `kms:Decrypt` |
| `review` | 在 `review-cases/` 下写入、读取和列出 Safety Case。在 `review-summaries/` 和 `review-notices/` 下写入摘要和 Safety Notices。读取并列出 `review-responses/` 下您团队的回复。可以永久删除案例、摘要和 Safety Notices。它无法读取对话记录，也无法写入、更改或删除回复。 | `kms:GenerateDataKey` 和 `kms:Decrypt` |

每个角色都只信任 Anthropic 的 AWS 账户 `804820989669` 中名称以 `anthropic-storage-broker-` 开头的角色（对于 review 角色，则为 `anthropic-review-broker-`）。调用方必须属于 Anthropic 的 AWS 组织。每个角色还要求您的外部 ID，以及来自该角色允许集合中的源身份。您的设置文件包含每个角色的完整策略。

对于表格中的前五个角色，其权限同样适用于 `safeguards-data/` 下的相同前缀，例如 `safeguards-data/slab/`。生命周期规则和存储桶策略也涵盖这些前缀。[哪些机制保护您存储桶中的对象版本](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#what-protects-object-versions)说明了 Anthropic 的角色无法删除哪些内容。

<Tabs>
  <Tab title="CloudFormation">
    #### 使用 CloudFormation 堆栈中的角色

    您在[步骤 2](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket) 中部署的堆栈已创建所有角色，因此此处无需运行任何内容。
  </Tab>

  <Tab title="Terraform">
    #### 使用 Terraform 应用中的角色

    [步骤 2](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket) 中的应用已创建所有角色，因此此处无需运行任何内容。
  </Tab>

  <Tab title="Your own tooling">
    #### 自行创建角色

    **Create in AWS** 步骤中的 **Your own tooling** 选项会列出角色 ARN。为每个 ARN 创建一个角色，并使用该 ARN 中包含的确切名称。

    从您的 CloudFormation 模板复制每个角色的完整信任策略及其权限。`transcripts` 和 `review` 角色的权限位于 `TranscriptsRolePolicy` 和 `ReviewRolePolicy` 中。然后，调整每个副本：

    * **信任策略：** 将外部 ID 设置为 **Your own tooling** 显示的值，不要更改其他任何内容。
    * **权限：** 将模板中的引用替换为您自己的值，包括您的区域和 KMS 密钥的 ARN。

    请自行将每个角色的权限和信任策略与模板进行比较。**Test setup** 不会读取任何角色的权限或信任策略。
  </Tab>
</Tabs>

<Warning title="保持存储桶和角色原样">
  创建存储桶和角色之后，请勿更改存储桶、其存储桶策略、其 KMS 密钥的策略或角色。您唯一可以进行的更改是设置 [`ReviewExpiryDays`](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#template-reference)。如果 Anthropic 的检查发现任何其他更改，检查就会失败。
</Warning>

<Where method="console" label="Claude Console">

### 4. 在 Claude Console 中连接您的存储桶

请在其工作区将使用您存储桶的 Anthropic 组织中登录 Claude Console，因为这些角色只接受该组织的 ID。

在[数据保留](https://platform.claude.com/settings/privacy)页面顶部选择**组织**。在 **Storage destinations** 下，点击 **Connect storage**；如果您已经开始了该对话框，则点击 **Finish setup**。存储目标是 Claude Console 对您存储桶的记录。

#### 选择您的提供商

选择 **Amazon S3**，然后点击 **Continue**。

#### 输入存储桶详细信息

输入您将在其中创建存储桶的 AWS 账户 ID 和 AWS 区域。在 **New bucket name** 下，输入您将为新存储桶指定的名称。请勿重复使用您已拥有的存储桶的名称。

#### Create in AWS

如果您已经运行了此步骤的文件，请前往 **Test setup**。如果尚未运行，请选择 **CloudFormation**、**Terraform** 或 **Your own tooling**，然后按照[步骤 1 到 3](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#if-an-aws-organization-policy-blocks-anthropic) 操作。

#### 测试设置

点击 **Test setup**。Anthropic 会检查您的存储桶、其密钥以及用于密封的安全数据的角色，并列出需要修复的内容。如果测试失败，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)。

</Where>

<Where method="api" label="Admin API">

### 4. 激活您的存储桶

请 Anthropic 检查您的设置。该请求没有请求体：

<CodeGroup>
  ```bash cURL
  curl -X POST https://api.anthropic.com/v1/organizations/external_storage/estrg_01SDCCSbTxrXDpWc1phhtcfK/activate \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: efs-2026-10-01"
  ```

  ```bash CLI
  ant beta:organization:external-storage activate \
    --external-storage-id estrg_01SDCCSbTxrXDpWc1phhtcfK
  ```

  ```python Python
  client = anthropic.Anthropic()

  storage = client.beta.organization.external_storage.activate(
      "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  )

  print(f"status: {storage.status.type}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const storage = await client.beta.organization.externalStorage.activate(
    "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  );

  console.log(`status: ${storage.status.type}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var storage = await client.Beta.Organization.ExternalStorage.Activate(
      "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  );

  Console.WriteLine($"status: {storage.Status.Type.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  storage, err := client.Beta.Organization.ExternalStorage.Activate(
  	context.Background(),
  	"estrg_01SDCCSbTxrXDpWc1phhtcfK",
  	anthropic.BetaOrganizationExternalStorageActivateParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("status: %s\n", storage.Status.Type)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var storage = client.beta().organization().externalStorage()
      .activate("estrg_01SDCCSbTxrXDpWc1phhtcfK");

  IO.println("status: " + storage.status().type());
  ```

  ```php PHP
  $client = new Client();

  $storage = $client->beta->organization->externalStorage->activate(
      externalStorageID: 'estrg_01SDCCSbTxrXDpWc1phhtcfK',
  );

  echo "status: {$storage->status->type}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  storage = client.beta.organization.external_storage.activate("estrg_01SDCCSbTxrXDpWc1phhtcfK")

  puts "status: #{storage.status.type}"
  ```
</CodeGroup>

Anthropic 会在请求运行期间检查您的存储桶、其 KMS 密钥以及用于密封的安全数据的角色。如果全部通过，状态会变为 `available`，您的存储桶即可附加。如果某项检查失败，请参阅[管理客户自有存储中的故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#api-troubleshooting)。

</Where>

## 在工作区中使用您的存储目标

<Where method="console" label="Claude Console">

存储桶通过测试后，请为每个应使用它的工作区选择它。请在工作区承载需要存储到您存储桶中的流量之前执行此操作，因为[更改不会立即生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)。如果您的组织有 Legacy Zero Data Retention (ZDR) 安排，请在创建工作区时选择 **Enterprise Frontier Safeguards** 卡片和您的存储桶。<Where cloud="aws" label="Claude Platform on AWS">请按照[在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws)中的 Legacy ZDR 步骤操作。</Where>

</Where>

对于使用客户自有存储的工作区，Anthropic 强烈建议使用客户管理密钥，但不强制要求。在做出决定之前，请阅读除了您存储桶自身的 KMS 密钥之外，[客户管理密钥还能带来什么](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#if-you-use-your-own-bucket)。

<Where method="console" label="Claude Console">

<Warning>
  为工作区选择存储桶会[将该存储桶与此组织绑定](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#before-you-begin)。请将这种绑定视为永久性的。在选择存储桶之前，请确认您位于其工作区将使用此存储桶的 Anthropic 组织中。
</Warning>

在工作区的[数据保留](https://platform.claude.com/settings/privacy)页面上，选择 **Enterprise Frontier Safeguards** 卡片，然后在 **Sealed Safety Data storage** 下选择您的存储桶。<Where cloud="aws" label="Claude Platform on AWS">还需在 **Safety Case and Transcript storage** 下选择您的存储桶。</Where> <Where cloud="claude" label="Claude Platform">**Safety Case and Transcript storage** 是可选的。如果其下没有选择存储桶，则只有当您的组织拥有[Safety Cases API 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#get-access)时，您的团队才会收到 Safety Case。如果您的存储桶[可以接收对话记录和 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#case-and-transcript-bucket)，您可以在其下选择您的存储桶。</Where>

然后保存工作区。如果保存时 Anthropic 拒绝了您的存储桶，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)。

保存操作会关闭 Anthropic Human Review，这[最多可能需要 2 小时](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)。要检查保存是否生效，请确认在[工作区](https://platform.claude.com/settings/workspaces)页面上该工作区的侧边面板中，**Case review by** 显示为 **My organization**。如果 **Case review by** 未显示为 **My organization**，请在[数据保留](https://platform.claude.com/settings/privacy)页面上打开该工作区并再次点击 **Save**，即使您没有做任何更改。如果您无法这样做，或者 **Case review by** 仍未显示为 **My organization**，请在发送真实流量之前联系您的 Anthropic 客户团队。

</Where>

<Where method="api" label="Admin API">

### 将您的存储桶附加到工作区

将您的存储桶附加到每个应使用它的工作区。附加操作会告诉 Anthropic 将该工作区的密封的安全数据存储在您的存储桶中。

在具有 Legacy Zero Data Retention (ZDR) 安排的组织中，请勿将您的存储桶附加到数据保留处于关闭状态的工作区。请改为[创建已附加您存储桶的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-create-workspace-attached)。

<Warning>
  在发送此请求之前，请确认您的密钥或令牌属于将使用此存储桶的各个工作区所在的组织。第一个成功的附加请求会[将您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#before-you-begin)、其名称、其 KMS 密钥和其 IAM 角色与该组织关联。请将此视为永久性的。
</Warning>

要附加您的存储桶，请使用 `POST /v1/organizations/workspaces/{workspace_id}` 更新工作区，并将 `inference_data_privacy.safeguards_storage` 设置为 `external` 类型，同时指定您存储桶的 ID。此字段对应 Claude Console 中的 **Sealed Safety Data storage**。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: efs-2026-10-01" \
    -H "content-type: application/json" \
    -d '{
      "inference_data_privacy": {
        "safeguards_storage": {
          "type": "external",
          "external_storage_id": "estrg_01SDCCSbTxrXDpWc1phhtcfK"
        },
        "human_review": {
          "type": "disabled"
        }
      }
    }'
  ```

  ```bash CLI
  ant beta:organization:workspaces update \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --beta efs-2026-10-01 <<YAML
  inference_data_privacy:
    safeguards_storage:
      type: external
      external_storage_id: estrg_01SDCCSbTxrXDpWc1phhtcfK
    human_review:
      type: disabled
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspace = client.beta.organization.workspaces.update(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      inference_data_privacy={
          "safeguards_storage": {
              "type": "external",
              "external_storage_id": "estrg_01SDCCSbTxrXDpWc1phhtcfK",
          },
          "human_review": {"type": "disabled"},
      },
      betas=["efs-2026-10-01"],
  )

  privacy = workspace.inference_data_privacy
  if privacy and privacy.safeguards_storage:
      print(f"safeguards_storage: {privacy.safeguards_storage.type}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const workspace = await client.beta.organization.workspaces.update(
    "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    {
      inference_data_privacy: {
        safeguards_storage: {
          type: "external",
          external_storage_id: "estrg_01SDCCSbTxrXDpWc1phhtcfK"
        },
        human_review: { type: "disabled" }
      },
      betas: ["efs-2026-10-01"]
    }
  );

  const type = workspace.inference_data_privacy?.safeguards_storage?.type;
  console.log(`safeguards_storage: ${type}`);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Workspaces;

  AnthropicClient client = new();

  var workspace = await client.Beta.Organization.Workspaces.Update(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      new()
      {
          InferenceDataPrivacy = new()
          {
              SafeguardsStorage = new BetaExternalStorageMemberParam
              {
                  ExternalStorageID = "estrg_01SDCCSbTxrXDpWc1phhtcfK",
              },
              HumanReview = new() { Type = BetaAnthropicHumanReviewParamType.Disabled },
          },
          Betas = ["efs-2026-10-01"],
      }
  );

  var type = workspace.InferenceDataPrivacy?.SafeguardsStorage?.Type.Raw();
  Console.WriteLine($"safeguards_storage: {type}");
  ```

  ```go Go
  client := anthropic.NewClient()

  workspace, err := client.Beta.Organization.Workspaces.Update(
  	context.Background(),
  	"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  	anthropic.BetaOrganizationWorkspaceUpdateParams{
  		InferenceDataPrivacy: anthropic.BetaInferenceDataPrivacyUpdateParam{
  			SafeguardsStorage: anthropic.BetaStorageMemberParamOfExternal("estrg_01SDCCSbTxrXDpWc1phhtcfK"),
  			HumanReview: anthropic.BetaAnthropicHumanReviewParam{
  				Type: anthropic.BetaAnthropicHumanReviewParamTypeDisabled,
  			},
  		},
  		Betas: []anthropic.AnthropicBeta{"efs-2026-10-01"},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("safeguards_storage: %s\n", workspace.InferenceDataPrivacy.SafeguardsStorage.Type)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.workspaces.BetaAnthropicHumanReviewParam;
  import com.anthropic.models.beta.organization.workspaces.BetaInferenceDataPrivacyUpdateParam;
  import com.anthropic.models.beta.organization.workspaces.WorkspaceUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = WorkspaceUpdateParams.builder()
          .inferenceDataPrivacy(BetaInferenceDataPrivacyUpdateParam.builder()
              .externalSafeguardsStorage("estrg_01SDCCSbTxrXDpWc1phhtcfK")
              .humanReview(BetaAnthropicHumanReviewParam.builder()
                  .type(BetaAnthropicHumanReviewParam.Type.DISABLED)
                  .build())
              .build())
          .addBeta("efs-2026-10-01")
          .build();
      var workspace = client.beta().organization().workspaces()
          .update("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ", params);

      IO.println("safeguards_storage: " + workspace.inferenceDataPrivacy()
          .flatMap(privacy -> privacy.safeguardsStorage())
          .map(storage -> storage.type().asString())
          .orElse(null));
  }
  ```

  ```php PHP
  $client = new Client();

  $workspace = $client->beta->organization->workspaces->update(
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
      inferenceDataPrivacy: [
          'safeguardsStorage' => [
              'type' => 'external',
              'externalStorageID' => 'estrg_01SDCCSbTxrXDpWc1phhtcfK',
          ],
          'humanReview' => ['type' => 'disabled'],
      ],
      betas: ['efs-2026-10-01'],
  );

  echo "safeguards_storage: {$workspace->inferenceDataPrivacy?->safeguardsStorage?->type}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace = client.beta.organization.workspaces.update(
    "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    inference_data_privacy: {
      safeguards_storage: {
        type: :external,
        external_storage_id: "estrg_01SDCCSbTxrXDpWc1phhtcfK"
      },
      human_review: {
        type: :disabled
      }
    },
    betas: ["efs-2026-10-01"]
  )

  puts "safeguards_storage: #{workspace.inference_data_privacy&.safeguards_storage&.type}"
  ```
</CodeGroup>

请求成功后，您存储桶的状态会变为 `active`。示例请求还会将 `inference_data_privacy.human_review` 设置为 `disabled` 类型，因为在 Admin API 中，仅靠附加操作不会关闭 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。使用 Enterprise Frontier Safeguards (EFS) 时，Anthropic Human Review 始终处于关闭状态，由您自己的团队审查该工作区的 [Safety Cases](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。

附加操作[不会立即生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)。在生效之前，如果工作区原先没有[安全数据存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)，其密封的安全数据可能会进入 Anthropic 管理的存储。在第一个新对象到达满 10 分钟之前，只发送不含敏感数据的测试请求，因为 Anthropic 的服务器不会同时获取该更改。要确认附加已经生效，请留意您存储桶中 `slab/` 或 `safeguards-data/slab/` 下是否出现新对象。无论是 `active` 状态还是重新读取工作区，都无法表明附加已经生效。如果已有另一个工作区向此存储桶发送数据，新对象也无法表明这一点，因此在发送真实流量之前，请咨询您的 Anthropic 客户团队。

要将工作区移至另一个存储桶，请按照[更换您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-your-bucket)操作。在第 1 步中，[注册](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-register)并[激活](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-activate)您的新存储桶。在第 2 步中，请改为使用新存储桶的 ID 发送附加请求。如果 `transcripts_storage` 指定的是您的旧存储桶，请在同一个请求中将其设置为新存储桶的 ID。请勿先[分离](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#api-stop)，否则新的密封的安全数据可能会进入 Anthropic 管理的存储。

附加请求可能返回如下错误：

* `human_review_on_with_external_storage`：如果在 Anthropic Human Review 处于开启状态时省略 `human_review`，Anthropic 可能会返回此错误。
* `approval_required`：您的组织未获得 Anthropic 关于关闭 Anthropic Human Review 的批准。该请求不会做任何更改。
* `workspace_policy_violation`：您组织的策略拒绝了该请求，例如因为该策略要求使用 Anthropic Human Review。
* `transcripts_storage_required`：不会做任何更改。请在同一个请求中[同时设置 `transcripts_storage`](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-attach-transcripts)。

有关附加错误的更多信息，请参阅[管理客户自有存储中的故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#api-troubleshooting)。

#### 将对话记录和 Safety Cases 发送到存储桶

要将对话记录和 Safety Cases 发送到您的存储桶，请将 `inference_data_privacy.transcripts_storage` 设置为 `external` 类型，并使用与 `safeguards_storage` 相同的 ID。此字段对应 Claude Console 中的 **Safety Case and Transcript storage**。您可以在同一个请求中设置这两个存储字段。如果其中任一字段的检查失败，Anthropic 会终止整个请求。[您存储桶中的对话记录和 Safety Cases](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#case-and-transcript-bucket)说明了哪些存储桶可以接收它们。

#### 创建已附加您存储桶的工作区

要创建一个已附加您存储桶的工作区，请将相同的 `inference_data_privacy` 设置发送到 `POST /v1/organizations/workspaces`。附加操作仍然[不会立即生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)。在具有 Legacy ZDR 安排的组织中，新工作区的数据保留可能处于关闭状态。如果数据保留已关闭，在您将其开启之前，不会有任何内容发送到您的存储桶。要开启数据保留，请在 Claude Console 中打开该工作区的[数据保留](https://platform.claude.com/settings/privacy)页面，点击 **Finish switching**，然后点击 **Accept**。工作区从点击 **Accept** 时开始保留密封的安全数据，因此请先留出时间让附加生效。在第一个新对象到达满 10 分钟之前，只发送测试请求。

如果该创建请求返回 `approval_required`、`grant_required`、`authorization_check_unavailable`、`human_review_on_with_external_storage` 或 `transcripts_storage_required`，则不会创建该工作区。

针对选定工作区的批准无法涵盖尚不存在的工作区。如果在具有 Legacy ZDR 安排的组织中创建请求返回 `grant_required`，请勿在不附加您存储桶的情况下创建工作区。请联系您的 Anthropic 客户团队。在其他任何组织中，请在不带 `inference_data_privacy` 的情况下创建工作区，请您的 Anthropic 客户团队为该工作区办理批准，然后按照[将您的存储桶附加到工作区](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-attach)中的说明附加您的存储桶。

</Where>

<Where cloud="claude" label="Claude Platform">

有关完整步骤（包括工作区的其他设置），请参阅[在 Claude Platform 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#turn-on-efs-for-a-workspace)。

</Where>

<Where cloud="aws" label="Claude Platform on AWS">

有关完整步骤（包括工作区的其他设置），请参阅[在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#turn-on-efs-for-a-workspace)。

</Where>

您可以使用[针对新工作区的策略](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#set-organization-defaults)，要求每个保留密封的安全数据的新工作区都使用客户自有存储。

### 确认更改已生效

在工作区承载需要存储到您存储桶中的流量之前，请发送不含敏感数据的测试请求，直到有新对象到达 `slab/` 或 `safeguards-data/slab/` 下。然后，再等待 10 分钟，因为 Anthropic 的服务器不会同时获取该更改。在等待结束之前，请只发送测试请求，因为您的密封的安全数据可能会进入 Anthropic 管理的存储。如果已有另一个工作区向此存储桶发送数据，新对象无法表明此更改已生效，因此在发送真实流量之前，请咨询您的 Anthropic 客户团队。如果进行更改 4 小时后 `slab/` 或 `safeguards-data/slab/` 下仍未出现新对象，请联系您的 Anthropic 客户团队。

更改不会立即生效。在等待期间，即使设置成功，以下情况也属正常：

* **等待。** 当新工作区或使用 Anthropic 管理的存储的工作区开始使用您的存储桶时，更改通常在 10 分钟内生效，但也可能需要更长时间。
* **没有新对象。** 在更改生效之前，来自该工作区请求的密封的安全数据可能会进入 Anthropic 管理的存储，`slab/` 或 `safeguards-data/slab/` 下可能不会出现新对象。
* **显示 Active 状态但没有对象。** 在更改生效之前，您的存储桶可能已在 **Storage destinations** 中显示为 **Active**，因此 **Active** 并不意味着对象正在到达。
* **Anthropic Human Review。** 如果更改同时关闭了 Anthropic Human Review，这最多可能需要 2 小时。在此之前，Anthropic 的审查团队可能仍会审查该工作区的数据。

如果您之后将工作区移至其他存储桶或停止使用 EFS，该迁移同样不会立即生效。在一次保存或请求中于两个 Amazon S3 存储桶之间迁移后，新的密封的安全数据在一段时间内仍可能进入您的旧存储桶。该迁移本身不会将新数据发送到 Anthropic 管理的存储。请保留旧存储桶以及 Anthropic 对它的访问权限，如[更改或停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket)中所述。

## 您存储桶中的对话记录和 Safety Cases

要在您的存储桶中接收工作区的对话记录和 Safety Cases，请检查该存储桶是否可以接收它们，然后为该工作区选择它。对话记录和 Safety Cases 会进入同一个存储桶，您不能只选择其中之一。

<Where cloud="aws" label="Claude Platform on AWS">

[在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws)介绍了设置 **Safety Case and Transcript storage** 的步骤。

</Where>

<Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">

[在 Claude Models on Google Cloud (Anthropic managed) 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#turn-on-efs-for-a-workspace)介绍了设置 **Safety Case and Transcript storage** 的步骤。

</Where>

<Where cloud="claude" label="Claude Platform">

要确定存储桶能否接收对话记录和 Safety Cases，请查看其角色的名称：

* **以 `anthropic-efs-<RoleDigest>-` 开头的名称：** 存储桶可以接收对话记录和 Safety Cases。
* **固定名称，例如 `anthropic-efs-ingest`：** 存储桶无法接收它们。要接收它们，请[更换为新的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-your-bucket)，并在两项存储设置下都选择它。

如果存储桶可以接收它们，请打开工作区的[数据保留](https://platform.claude.com/settings/privacy)页面。如果该页面显示 **Safety Case and Transcript storage**，请在该设置和 **Sealed Safety Data storage** 下选择同一个存储桶。如果您选择了不同的存储桶，即使保存成功，它也可能不会收到对话记录和 Safety Cases。保存时，Anthropic 会检查该存储桶是否已针对对话记录和 Safety Cases 完成设置。如果 Anthropic 拒绝该存储桶，整个保存操作都会停止。要解决此问题，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)。如果在选中 **Enterprise Frontier Safeguards** 卡片后页面未显示 **Safety Case and Transcript storage**，请联系您的 Anthropic 客户团队。

每类文件都有自己的前缀：

| 前缀 | 存放内容 | 写入者 |
| ----- | ----- | ----- |
| `transcripts/` | 对话记录 | Anthropic |
| `review-cases/`、`review-summaries/` 和 `review-notices/` | Safety Case 文件 | Anthropic |
| `review-responses/` | 您团队的回复 | 您的团队。任何 Anthropic 角色都无法写入、更改或删除它们。 |

设置文件的生命周期规则会在[为每类内容设置的天数](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket)后，使所有这些文件过期。

</Where>

### 向您的团队授予存储桶访问权限

要向您的团队授予访问权限，请将以下权限添加到您自己用户的 IAM 策略中。设置文件创建的任何内容都无需更改。

* **要读取对话记录和 Safety Cases：** 允许对 `transcripts/` 和审查前缀（`review-cases/`、`review-responses/`、`review-summaries/` 和 `review-notices/`）下的内容执行 `s3:GetObject`，对存储桶执行 `s3:ListBucket` 并附带将其限制在这些前缀内的 `s3:prefix` 条件，以及对密钥执行 `kms:Decrypt`。请勿授予对任何其他前缀的 S3 访问权限。
* **要读取文件的早期版本：** 还需允许 `s3:GetObjectVersion` 和 `s3:ListBucketVersions`，并附带相同的限制。
* **要在 `review-responses/` 下写入回复：** 还需允许对 `review-responses/*` 执行 `s3:PutObject`，以及对密钥执行 `kms:GenerateDataKey`。请使用存储桶的默认加密写入回复，因为 Anthropic 无法读取使用其他 KMS 密钥加密的回复。

有关审查流程，请参阅[审查 Safety Cases](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。

## 故障排除

<AccordionGroup>
  <Accordion title="CloudFormation 部署失败">
    如果部署立即停止并出现关于 `Fn::Select` 的错误，说明堆栈名称与 `RoleDigest` 不匹配。没有任何内容被更改，因此请完全按照显示的内容运行对话框中的命令。如果您当时正在更新另一个名称的堆栈，请保持其原样并联系您的 Anthropic 客户团队。

    要查找其他任何失败的原因，请运行 **Create in AWS** 步骤在部署命令下方显示的两个只读命令，或者在 AWS CloudFormation 控制台中打开该堆栈的 **Events** 选项卡。

    如果存储桶名称已被占用，请删除失败的堆栈和存储目标，然后使用另一个存储桶名称连接一个新的存储目标。如果原因仍不清楚，请将全部输出发送给您的 Anthropic 客户团队。请勿向任何命令添加 `--debug`，因为调试输出可能包含您的 AWS 会话令牌。
  </Accordion>

  <Accordion title="Terraform 应用失败">
    如果第一次应用中途失败，请再次运行相同的应用以创建缺失的部分。如果应用因为另一个 AWS 账户已拥有您所输入名称的存储桶而失败，Terraform 不会创建任何内容，再次运行应用也无济于事。请改为删除该存储目标，使用另一个存储桶名称连接一个新的存储目标，并使用对话框为新存储目标提供的命令进行应用。如果您自己的 AWS 账户中已有之前应用创建的存储桶，请保留该名称，并恢复该文件夹的状态或运行 `terraform import`。
  </Accordion>

  <Accordion title="Test setup 显示 Anthropic 无法运行测试">
    请在几分钟后再次运行测试。如果测试持续显示此消息，请告知您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="Test setup 显示 Anthropic 无法代入 verify 角色">
    出现此问题时，**Test setup** 可能会显示多个检查失败。第一个失败检查的消息以 “Anthropic could not assume the verify role” 开头，其他消息则显示某个角色 “must have the setup template’s trust policy”。即使您完全按照说明创建了存储桶和角色，也可能发生这种情况。

    首先，查看您的 IAM 角色的名称。如果此存储桶从未通过 **Test setup**，且其角色使用固定名称（例如 `anthropic-efs-ingest`），则说明您的设置来自较早的设置文件。Anthropic 不接受角色使用固定名称的新存储桶，更改这些角色也无法使测试通过。请改为执行以下操作：

    1. 删除存储目标。
    2. 使用新的存储桶名称[从 **Connect storage** 重新开始](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)，并使用对话框提供给您的文件或值。使用 Terraform 时，请使用新的文件夹，且不要在早期文件夹中应用。
    3. 之后，如果没有其他存储桶使用这些早期角色，您可以按照[停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#stop-using-your-bucket)中 **Remove Anthropic’s access** 步骤的说明移除它们。然后，您可以删除未使用的存储桶。

    如果您的角色不使用固定名称，或者此存储桶之前已通过 **Test setup**，请逐一完成以下检查：

    1. 检查您在 AWS 中的设置是否已使用对话框中的文件或值完成。检查您输入的 AWS 账户 ID 和区域是否为您部署所在的账户和区域。要更改其中任一项，请删除存储目标并连接一个新的存储目标。从其他任何地方获取的该文件副本可能会创建名称不同的角色，而测试无法找到这些角色。如果您使用了此类副本，请删除存储目标，使用新的存储桶名称连接一个新的存储目标，并使用对话框提供给您的文件。

    2. 检查您是否完全按照显示的内容运行了对话框中的命令，以确保外部 ID 和角色名称正确。具体操作取决于您更改了什么：

       * **组织 ID：** 再次运行对话框中的命令。如果 Terraform 随后因组织 ID 已更改而停止，其消息会说明需要向命令添加什么。使用 CloudFormation 时，如果您的堆栈名称与对话框命令中的名称不同，请改为按照下一项操作。

       * **`RoleDigest`（使用 CloudFormation 时）：** 在 AWS CloudFormation 控制台中打开部署成功的堆栈。检查其 **Stack info** 选项卡上的 **Created time** 以及其 **Outputs** 选项卡上的 `BucketName`。具体操作取决于它们显示的内容：

         * **Created time 是您首次为此存储桶部署的时间：** 删除堆栈，这会保留其存储桶和 KMS 密钥。删除该未使用的存储桶，以及任何使用对话框命令中名称的失败堆栈。然后使用对话框中的文件运行对话框中的命令。
         * **堆栈早于该时间，且显示另一个存储桶：** 请勿改动它，并为此存储桶运行对话框中的命令。
         * **堆栈早于该时间，且显示此存储桶：** 您的部署更改了另一个存储桶的堆栈。请勿再运行任何内容，并立即联系您的 Anthropic 客户团队。

       * **存储桶名称：** 请告知您的 Anthropic 客户团队。

    3. 如果前两项检查未发现任何问题，请检查是否有 [AWS 组织策略阻止了 Anthropic](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#if-an-aws-organization-policy-blocks-anthropic)。单个拒绝 Anthropic 的策略就可能导致所有检查失败。如果原因在于策略，设置文件无需更改。

    修复后，点击 **Test again**。如果您已关闭对话框，请在 **Storage destinations** 中点击 **Finish setup**，然后点击 **Test setup**。如果测试仍然失败，请告知您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="Test setup 提到另一个组织的 External Storage">
    当 **Test setup** 失败，且有一项检查的消息以 “Another organization’s External Storage” 开头时，本节适用。当另一个 Anthropic 组织已使用了此设置的一部分，或者该存储桶名称已在另一个 AWS 账户中使用时，该检查会失败。该检查还可能将已删除组织的存储目标计算在内。

    如果另一个组织已使用过此 AWS 账户，您可以继续使用该账户。请先检查您部署的是否为对话框为此存储桶提供给您的文件，然后再次测试。

    如果测试仍然失败，或者没有其他组织使用过此 AWS 账户，请使用新的存储桶名称，并为其配备专属的密钥和角色。请从 **Connect storage** 重新开始，因为 **Finish setup** 会保留您输入的 AWS 账户和存储桶名称。如果新的存储桶以同样的方式失败，请告知您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="保存工作区时 Anthropic 拒绝了您的存储桶">
    您首次为工作区选择存储桶并保存时，Anthropic 会重复 **Test setup** 中的检查。如果 Anthropic 拒绝了您的存储桶，可能是其中某项检查失败了，原因是以下两种之一：

    * 自测试通过以来发生了更改：存储桶设置、存储桶的 KMS 密钥、IAM 角色或 AWS 组织策略。
    * 另一个 Anthropic 组织已使用了此设置的一部分，或者（如果您的角色使用固定名称）此 AWS 账户。

    Claude Console 会在对话框中列出失败的检查。如果您在 **Safety Case and Transcript storage** 下选择了存储桶、未列出任何检查，且错误未提及 Human Review，请前往下一个问题"在 Safety Case and Transcript storage 下 Anthropic 拒绝了您的存储桶"。

    如果唯一失败的检查以 “Another organization’s External Storage” 开头，具体操作取决于错误位于哪项设置下：

    * **Safety Case and Transcript storage：** 请联系您的 Anthropic 客户团队。
    * **Sealed Safety Data storage：** 请使用新的存储桶名称，并为其配备专属的密钥和角色。然后[从 **Connect storage** 重新开始](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)。

    对于其他任何失败的检查，请修复原因，然后再次保存工作区。如果您无法判断原因，请联系您的 Anthropic 客户团队。如果错误位于 **Safety Case and Transcript storage** 下，请按照下一个问题"在 Safety Case and Transcript storage 下 Anthropic 拒绝了您的存储桶"操作。

    如果错误显示 Enterprise Frontier Safeguards 需要关闭 Anthropic Human Review，则该次保存没有更改任何内容。如果错误显示 Anthropic 不对您自有存储中的数据提供 Human Review，情况也是如此。您的组织需要获得 Anthropic 的批准才能关闭 Anthropic Human Review，并且您组织的策略不得要求 Anthropic Human Review。如果您的组织缺少该批准，请让组织管理员[接受相关条款](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)。如果未提供这些条款，请咨询您的 Anthropic 客户团队。如果该策略要求 Anthropic Human Review，请让管理员更改该策略。在该策略下，**Enterprise Frontier Safeguards** 卡片可能会显示 **Ineligible**。如果两者都不适用，请重新加载页面并再次保存。如果错误再次出现，请联系您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="在 Safety Case and Transcript storage 下 Anthropic 拒绝了您的存储桶">
    如果您存储桶的角色使用固定名称（例如 `anthropic-efs-ingest`），该存储桶无法接收对话记录和 Safety Cases，重试也无济于事。<Where cloud="aws" label="Claude Platform on AWS">请先咨询您的 Anthropic 客户团队，然后再执行任何其他操作。</Where>请勿使用[设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#get-the-setup-file)更新该存储桶的设置。要接收对话记录和 Safety Cases，请[更换为新的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-your-bucket)，并在保存之前在 **Sealed Safety Data storage** 和 **Safety Case and Transcript storage** 下都选择它。如果对话框为您提供使用固定角色名称的文件，请勿运行它，并联系您的 Anthropic 客户团队。

    如果您存储桶的角色不使用固定名称，请修复失败的检查所列出的问题，然后再次保存工作区。如果唯一失败的检查以 “Another organization’s External Storage” 开头，请联系您的 Anthropic 客户团队。如果消息显示该存储桶无法存储 Safety Case 和对话记录，或者暂时无法为对话记录附加存储，也请联系您的 Anthropic 客户团队。

    如果错误显示 Anthropic 无法检查该存储桶的设置，或仅显示工作区设置更新失败，请检查您的存储桶是否仍在两项存储设置下被选中，然后再次保存。如果错误再次出现，请联系您的 Anthropic 客户团队。

    <Where cloud="claude" label="Claude Platform">

    要仅保存 **Sealed Safety Data storage**，如果页面提供该选项，请在 **Safety Case and Transcript storage** 下选择 **Not copied to your storage**。如果未提供，请联系您的 Anthropic 客户团队。

    </Where>
  </Accordion>
</AccordionGroup>

## 模板参考

在运行 **Create in AWS** 步骤为您提供的 CloudFormation 模板或 Terraform 文件之前，请先查看它。该文件包含每个角色的完整策略，以及其中的每个操作和条件。

要更改 Safety Case 文件和您团队的回复的过期时间，请设置 `ReviewExpiryDays`（在 Terraform 中为 `review_expiry_days`），这是您可以自行设置的唯一一个值。它是这些文件在多少天后过期，最多为 9,999。默认值为 2,555，约 7 年。以下规则适用，因此请在更改之前阅读它们：

* 如果 28 天更早，摘要会在 28 天后过期。
* 即使文件的审查尚未完成，文件也会过期，因此在设置较小的数字之前，请咨询您的 Anthropic 客户团队。
* 新的数字也适用于存储桶中已有的文件，因此较小的数字可能会使较旧的文件过期并移除早期版本。
* 过期、被替换或被删除的文件会作为早期版本再保留同样的天数，对于摘要则最多保留 3 天。要立即移除文件，请删除其所有版本。
* 使用 Terraform 时，请在每次应用时传入 `review_expiry_days`。未传入该值的应用会恢复为默认值。

<AccordionGroup>
  <Accordion title="角色可以对您存储桶的 KMS 密钥执行的操作">
    [角色表格](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-roles)中的每项 KMS 权限都只适用于设置文件为您的存储桶创建的 KMS 密钥。`kms:GenerateDataKey` 和 `kms:Decrypt` 还带有针对您存储桶所在区域中 Amazon S3 的 `kms:ViaService` 条件，因此角色只能通过 Amazon S3 使用该密钥进行加密和解密，而不能自行调用 AWS KMS。
  </Accordion>

  <Accordion title="更窄的 AWS 组织策略例外需要允许的内容">
    更窄的例外需要允许两个调用：`sts:AssumeRole` 和 `sts:SetSourceIdentity`。这些调用来自 Anthropic 的 AWS 账户中名称以 `anthropic-storage-broker-` 或 `anthropic-review-broker-` 开头的 IAM 角色。这些调用只针对存放您存储桶的 AWS 账户中此设置的角色，设置文件不允许 Anthropic 的调用方执行其他任何操作。之后，Anthropic 在 Amazon S3 和 AWS KMS 中的操作以此设置的角色身份运行，而这些角色是您自己 AWS 账户中的主体。一个允许这两个调用的例外应该就足够了，但 Anthropic 只看到过在账户范围例外的情况下 **Test setup** 通过。**Test setup** 从不测试 `review` 角色，因此测试通过并不能说明您的例外涵盖了 `anthropic-review-broker-`。
  </Accordion>
</AccordionGroup>

## 后续步骤

<CardGroup cols={3}>
  <Card title="管理客户自有存储" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage">
    检查存储桶的状态，修复停止工作的存储桶，以及更改或停止使用它。
  </Card>

  <Where cloud="claude" label="Claude Platform">

  <Card title="在 Claude Platform 上设置 Enterprise Frontier Safeguards" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console">
    为工作区选择存储目标，然后检查您的设置。
  </Card>

  </Where>

  <Where cloud="aws" label="Claude Platform on AWS">

  <Card title="在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws">
    为工作区选择存储目标，然后检查您的设置。
  </Card>

  </Where>

  <Card title="使用客户管理密钥" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys">
    使用您在自己的密钥管理服务中持有的密钥加密您的数据。
  </Card>
</CardGroup>

</Where>

<Where bucket="gcs" label="Google Cloud Storage">

## 在 Google Cloud Storage 上设置客户自有存储

您自己云账户中的 Google Cloud Storage 存储桶会保存工作区的[密封的安全数据](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#what-is-stored)。要设置该存储桶，请使用 Terraform 创建它，并在 Claude Console 中连接它。您的团队无法读取密封的安全数据。开始之前，请阅读[在工作区中使用您的 Cloud Storage 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)。

设置分为三个部分：

1. [检查您 Google Cloud 组织的政策](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#check-your-policies)。
2. [创建并测试存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-create-the-bucket)。
3. [在工作区中使用您的 Cloud Storage 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)。

## 开始之前

请确保您具备以下各项：

* **[Enterprise Frontier Safeguards 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console)。**

  <Where cloud="claude" label="Claude Platform">如果您的组织拥有该权限，Claude Console 中的 [Programs](https://platform.claude.com/settings/programs) 页面会显示它，并带有 **Configure** 按钮。</Where>
* **一种连接您存储桶的方式。**&#x5728; Claude Platform 上，请检查您能否在 **Connect storage** 下选择 **Google Cloud Storage**。**Connect storage** 位于 Claude Console 的[数据保留](https://platform.claude.com/settings/privacy)页面上，需在页面顶部选择**组织**。如果您无法选择 **Google Cloud Storage**，请在开始第 1 部分之前联系您的 Anthropic 客户团队。
* **一名 Organization Admin**（位于 Claude Console 中），负责获取设置文件并连接您的存储桶。<Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">在 Claude Models on Google Cloud (Anthropic managed) 上，这是指在其组织将使用此存储桶的 Google Cloud 项目上拥有 [`roles/claude.admin`](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console) 角色的人员。</Where>
* **一个 Google Cloud 项目**，其中没有自动化程序会删除自定义角色、授权或存储桶。
* **工具：**[Terraform 1.9 或更高版本](https://developer.hashicorp.com/terraform/install)以及 [Google Cloud CLI](https://docs.cloud.google.com/sdk/docs/install-sdk)。
* **项目上的角色：**&#x9879;目 Owner，或以下所有角色：`roles/storage.admin`、`roles/cloudkms.admin`、`roles/iam.roleAdmin` 和 `roles/serviceusage.serviceUsageAdmin`。
* **用于更改政策的角色：**&#x47;oogle Cloud 组织上的 `roles/orgpolicy.policyAdmin`。项目 Owner 不包含该角色。

## 1. 检查您 Google Cloud 组织的政策

在此步骤中，您需要确保您 Google Cloud 组织的政策允许设置文件创建其资源，并授予 Anthropic 对您存储桶的访问权限。

### 允许这些账号

为了让设置文件向 Anthropic 授予访问权限，如果开启了 “domain-restricted sharing”（网域限制共享），请允许以下账号。它是 2024 年 5 月 3 日或之后创建的 Google Cloud 组织的默认设置，只允许向您组织的成员授予 IAM 权限。如果您不允许这些账号，apply 会停止并显示类似 “One or more users named in the policy do not belong to a permitted customer.” 的错误。

最后一个账号是您项目自己的 Cloud Storage 服务代理，它会使用存储桶的密钥：

* `anthropic-efs-ingest@anthropic-storage-us.iam.gserviceaccount.com`
* `anthropic-efs-read@anthropic-storage-us.iam.gserviceaccount.com`
* `anthropic-efs-maintenance@anthropic-storage-us.iam.gserviceaccount.com`
* `anthropic-efs-media-erasure@anthropic-storage-us.iam.gserviceaccount.com`
* `anthropic-efs-verify@anthropic-storage-us.iam.gserviceaccount.com`
* `anthropic-efs-transcripts@anthropic-storage-us.iam.gserviceaccount.com`
* `anthropic-efs-review@anthropic-storage-us.iam.gserviceaccount.com`
* `service-123456789012@gs-project-accounts.iam.gserviceaccount.com`，其中使用您项目的编号

在政策中，请在每个地址前写上 `serviceAccount:`。请更改适用于此项目的每个约束：

* **`constraints/iam.managed.allowedPolicyMembers`** 可以列出单个账号。请添加所有这些账号。

* **`constraints/iam.allowedPolicyMemberDomains`** 只能列出整个组织。请选择以下选项之一：

  * 在此项目上，以您自己的组织和这些账号强制执行 `constraints/iam.managed.allowedPolicyMembers`，然后覆盖 `constraints/iam.allowedPolicyMemberDomains`。
  * 从此项目上移除该约束，执行 apply，然后再恢复该约束（[强制账号访问](https://docs.cloud.google.com/organization-policy/restrict-domains#forcing_access)）。对之后任何添加或替换授权的 apply 重复此操作。

对政策的更改最多可能需要 15 分钟才能生效。

### 检查您的其他政策

请检查您组织的其他政策是否满足设置所需的条件：

| 政策 | 设置所需的条件 |
| ----- | ----- |
| `constraints/cloudkms.allowedProtectionLevels` | 允许 `HSM` |
| `constraints/gcp.resourceLocations` | 允许您的存储桶所在的[区域](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#supported-regions) |
| `constraints/storage.softDeletePolicySeconds` | 允许 0 |
| `constraints/gcp.restrictCmekCryptoKeyProjects` | 允许此项目 |
| `constraints/storage.retentionPolicySeconds` | 不适用于此项目 |
| VPC Service Controls 服务边界 | 有一条入站规则允许 Anthropic 的账号从任何来源访问 `storage.googleapis.com` |
| IP 过滤或 IAM 拒绝政策 | 不作用于 Anthropic 的账号 |

## 2. 创建并测试存储桶

在此步骤中，您将从 Claude Console 下载一个 Terraform 文件，对其执行 apply，并测试结果。该文件会创建存储桶、一个加密该存储桶的 `HSM` Cloud KMS 密钥、自定义角色，以及授予 Anthropic 账号的授权。它还会准备存储桶以存放对话记录和 Safety Cases。

存储桶自身的生命周期规则会在每个密封的安全数据对象存在满 28 天后将其删除，并在至少一天后将其永久移除。

<Warning>
  在该项目上拥有 Viewer 角色的任何人都可以下载其所有存储桶中的对象，包括任何对话记录和 Safety Case 文件。这些文件未经密封，因此任何下载它们的人都可以读取它们。拥有 Editor 或 Owner 角色的任何人还可以更改或删除这些对象。仅当您接受这种访问权限时，才将一个项目用于多个 Anthropic 组织。
</Warning>

1. 在 Claude Console 的[数据保留](https://platform.claude.com/settings/privacy)页面上，在页面顶部选择**组织**并点击 **Connect storage**。请确保您位于其工作区将使用此存储桶的 Anthropic 组织中。

   * **在 Claude Models on Google Cloud (Anthropic managed) 上：**&#x5BF9;话框不会要求选择提供商，并且会填入其组织将使用此存储桶的项目的编号。如果情况并非如此，则说明您位于错误的组织中。请从该项目打开 Claude Console。
   * **在 Claude Platform 上：**&#x9009;择 **Google Cloud Storage**。
   * 如果 **Connect storage** 缺失或不可用，或者 **Google Cloud Storage** 显示为 **Not available yet**，请联系您的 Anthropic 客户团队。

2. 输入您项目的编号；如果对话框已填入该编号，请核对它。然后，输入一个不太可能已被使用的新存储桶名称。在 **Google Cloud region** 下，保留 `us-east5` 或选择[其他区域](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#supported-regions)。点击 **Continue** 后，只能通过[删除存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#delete-a-storage-destination)并重新连接来更改这三个值。

3. 选择 **Terraform**，输入您项目的 ID，然后将 `customer-bucket-setup-gcs.tf` 下载到一个空文件夹中。

4. 在该文件夹中运行以下命令。对话框会显示包含您自己的值的两条 `terraform` 命令。

```bash
gcloud auth login
gcloud auth application-default login
gcloud services enable storage.googleapis.com --project acme-claude-storage
gcloud services enable cloudkms.googleapis.com --project acme-claude-storage
gcloud services enable iam.googleapis.com --project acme-claude-storage

terraform init
terraform apply \
  -var "project_id=acme-claude-storage" \
  -var "project_number=123456789012" \
  -var "organization_id=8a3f2c1e-5b7d-4e9a-a1c6-2f4e8d0b7c35" \
  -var "bucket_name=acme-claude-retention" \
  -var "region=us-east5"
```

首次 apply 时，在出现 `Enter a value:` 提示时，仅当计划显示 `Plan: 42 to add, 0 to change, 0 to destroy.` 时才回答 `yes`。如果不是，请回答 `no`，重新下载该文件并覆盖旧文件，然后再次运行两条 `terraform` 命令。如果计划仍然不同，请联系您的 Anthropic 客户团队。

保留整个文件夹和 apply 命令。测试通过后，Claude Console 不会再显示该命令。

apply 完成后，请等待 10 分钟让授权生效，然后点击 **Test setup** 并等待显示 **Ready**。

<Warning title="保持文件创建的内容不变">
  如果您更改了文件创建的内容，您可能会丢失数据，或终止 Anthropic 对您存储桶的访问。
</Warning>

特别是，请勿：

* 添加会删除对象的生命周期规则。
* 关闭版本控制。
* 更改或移除文件中的规则、角色、授权或标签。
* 开启软删除、保留政策、对象保留、默认保全或 Requester Pays。
* 更改存储桶的默认密钥。
* 停用或销毁密钥或其版本，或移除密钥对 Cloud Storage 的授权。否则对象将无法读取。
* 更改、移动、删除或添加以您的 Anthropic 组织 ID 命名的文件夹中的对象。例外情况是您团队的[响应文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#response-file)，它们位于该文件夹中的 `review-responses/` 下。
* 从文件中移除 `prevent_destroy`。
* 对授权运行 `terraform taint` 或 `terraform apply -replace=...`。

### 支持的区域

您可以使用对话框 **Google Cloud region** 列表中的任何区域。Anthropic 建议使用 `us-east5`，因为 Anthropic 从 `us-east5` 读取和写入您的存储桶。位于其他区域的存储桶可能会影响您的网络传输费用。您存储桶的区域决定了您的云提供商在何处存储该存储桶的对象。它不会改变[模型推理的运行位置](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#inference-geo)，也不会改变 Anthropic 处理您数据的位置。

### 如果 apply 或 Test setup 失败

要修复失败，请在此表中找到相应情况并按说明操作。对于其他任何失败，请在同一文件夹中使用相同的值再次 apply。

| 您看到的情况 | 应采取的操作 |
| ----- | ----- |
| 在新项目中，Google Cloud 拒绝某项授权或存储桶。 | 等待 10 分钟，然后再次 apply。 |
| Google Cloud 拒绝对 Anthropic 某个账号或对 Cloud Storage 的授权。 | [允许这些账号](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#allow-the-accounts)，然后再次 apply。 |
| 组织政策拒绝密钥或存储桶。 | [检查您的其他政策](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#check-your-other-policies)，然后再次 apply。 |
| “Instance cannot be destroyed.” 或 “organization\_id differs from the value that Terraform’s state keeps” | 传入首次 apply 时的值。如果其 `organization_id` 有误，请不要做任何更改，并联系您的 Anthropic 客户团队。 |
| 某项授权 “is tainted, so must be replaced.” | 不要应用该计划。如果该授权存在于 Google Cloud 中，请使用其地址运行 `terraform untaint`。如果不存在，请使用其地址运行 `terraform state rm`，然后再次 apply。 |
| 该存储桶的名称已被占用。 | [删除存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#delete-a-storage-destination)，然后使用新名称和一个新的空文件夹，从[创建并测试存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-create-the-bucket)重新开始。首次 apply 创建的密钥和角色会保留在您的项目中。 |
| **Test setup** 提示另一个组织已使用过此存储桶。 | [删除存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#delete-a-storage-destination)，然后使用新的存储桶名称和一个新的空文件夹，从[创建并测试存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-create-the-bucket)重新开始。首次 apply 创建的存储桶、密钥和角色会保留在原处。 |

### 更新用早期设置文件创建的存储桶

如果 **Test setup** 提示该存储桶来自早期的设置模板，请使用当前文件更新该存储桶：

1. 在您列出 Anthropic 账号的每个位置，添加[允许这些账号](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#allow-the-accounts)中新增的任何账号。
2. 点击 **Finish setup**，输入您项目的 ID，然后下载该文件并覆盖您文件夹中的旧文件。
3. 运行 `terraform init`，然后使用相同的值运行您的 apply 命令。如果计划失败或者会销毁或替换任何内容，请回答 `no` 并联系您的 Anthropic 客户团队。
4. 等待 10 分钟，然后点击 **Test setup**。

### 删除存储目标

要删除显示为 **Needs setup** 或 **Ready** 的存储目标，请在 **Storage destinations** 下其行菜单中选择 **Delete**。存储桶和 Anthropic 的授权会保留在您的项目中。

## 3. 在工作区中使用您的 Cloud Storage 存储桶

存储桶通过测试后，请为每个应使用它的工作区选择该存储桶。如果您的组织有 Legacy Zero Data Retention (ZDR) 安排，请在创建工作区时选择 **Enterprise Frontier Safeguards** 卡片和您的存储桶。

使用 EFS 时，Anthropic Human Review 始终处于关闭状态，因此审查 [Safety Cases](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases) 是您自己团队的职责。

在选择存储桶之前，请检查您位于其工作区将使用它的 Anthropic 组织中。一旦您在选中该存储桶的情况下保存或创建工作区，您就无法删除该存储目标。此后，其他任何 Anthropic 组织都不能将该存储桶用于客户自有存储。

<Warning>
  在以下情况下，请在选择存储桶之前先与您的 Anthropic 客户团队沟通：

  * **您的组织使用 Legacy Zero Data Retention。**&#x5BC6;封的安全数据[一开始可能会进入 Anthropic 管理的存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#confirm-that-objects-arrive)。
  * **您想使用多个存储桶或大量工作区。**&#x41;nthropic 对两者的数量都有限制。一旦超出限制，数据不会到达您的存储桶，Claude Console 也不会显示错误。
</Warning>

为工作区附加[客户管理的密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key)是永久性的。请使用与存储桶自身密钥不同的密钥。

在该工作区的[数据保留](https://platform.claude.com/settings/privacy)页面上，选择 **Enterprise Frontier Safeguards** 卡片，然后在 **Sealed Safety Data storage** 下选择您的存储桶。在 **Safety Case and Transcript storage** 下，您为以您的团队可读取的形式存放的对话记录和 Safety Cases 选择存储桶。如果页面显示该设置，您可以选择同一个存储桶。<Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">如果您未在该设置下选择存储桶，保存会失败。如果页面未显示该设置，请在保存之前联系您的 Anthropic 客户团队。</Where>然后保存工作区。如果 Anthropic 在您保存时拒绝您的存储桶，请在连接另一个存储桶之前联系您的 Anthropic 客户团队。<Where cloud="claude" label="Claude Platform">有关完整步骤，请参阅[在 Claude Platform 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#turn-on-efs-for-a-workspace)。</Where> <Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">有关完整步骤，请参阅[在 Claude Models on Google Cloud (Anthropic managed) 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#turn-on-efs-for-a-workspace)。</Where>

保存会关闭 Anthropic Human Review，这最多可能需要 2 小时。要检查保存是否成功，请确保在[工作区](https://platform.claude.com/settings/workspaces)页面上该工作区的侧面板中，**Case review by** 显示为 **My organization**。如果未显示 **My organization**，请在[数据保留](https://platform.claude.com/settings/privacy)页面上打开该工作区并再次点击 **Save**，即使您没有做任何更改。如果您无法这样做，或者 **Case review by** 仍未显示 **My organization**，请在发送真实流量之前联系您的 Anthropic 客户团队。

在您的存储桶中，[Safety Case 文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files)中所述的文件会存放在以您的 Anthropic 组织 ID 命名的文件夹中，例如 `<organization ID>/review-cases/<id>.json` 和 `<organization ID>/review-responses/<case_id>.json`。

要审查您存储桶中的 Safety Cases，您的团队需要列出并读取您组织文件夹中 `transcripts/`、`review-cases/`、`review-summaries/`、`review-notices/` 和 `review-responses/` 下的文件。要进行回复，您的团队还需要在 `review-responses/` 下创建文件。设置文件不会向您的团队授予此访问权限，因此请您自行在 Google Cloud IAM 中授予。请将读取权限限制在这些文件夹内，并将创建权限限制在 `review-responses/` 内。

### 确认对象已到达

在该工作区承载需要存储在您存储桶中的流量之前，请确认新对象已到达。然后再等待 10 分钟，因为 Anthropic 的服务器不会同时获取该更改。在此之前，请只发送测试请求，因为该工作区的密封的安全数据可能会进入 Anthropic 管理的存储。如果另一个工作区已在向此存储桶发送数据，新对象并不能表明此更改已生效，因此请在发送真实流量之前咨询您的 Anthropic 客户团队。

向该工作区发送不含敏感数据的测试请求。<Where cloud="claude" label="Claude Platform">请使用该工作区的凭据。</Where>然后，使用 apply 命令中的 `bucket_name` 和 `organization_id` 列出 `slab/`：

```bash
gcloud storage ls gs://acme-claude-retention/8a3f2c1e-5b7d-4e9a-a1c6-2f4e8d0b7c35/safeguards-data/slab/
```

**Active** 状态可能会在对象到达之前显示，因此请勿依赖它。如果该命令仍以 “One or more URLs matched no objects.” 停止，请联系您的 Anthropic 客户团队。`slab/` 下的对象表明密封的安全数据已到达。它们并不能表明对话记录或 Safety Cases 已到达。

### 如果您的存储桶停止工作

为避免丢失数据，请尽快恢复停止工作的存储桶。如果对您存储桶的写入失败，Anthropic 会重试。请在请求发出后约 46 小时内恢复您的存储桶，并让每个工作区继续使用它，否则您可能会丢失这些数据。Anthropic 最迟会在请求发出约 72 小时后停止重试，然后删除这些数据。任何人都无法恢复这些数据，这些数据也不会进入 Anthropic 管理的存储。

要发现故障，请留意 `slab/` 下的新对象。请勿依赖电子邮件或存储桶的状态来提醒您。如果您发现故障，请尽快告知您的 Anthropic 客户团队。

要恢复您的存储桶，请根据更改情况执行相应操作：

* **授权或设置：**&#x4F7F;用相同的值再次 apply。计划不应销毁任何内容。
* **已删除的自定义角色：**&#x5728; 7 天内运行 `gcloud iam roles undelete`。apply 不会恢复它。
* **已停用的密钥版本：**&#x8FD0;行 `gcloud kms keys versions enable`；如果该版本已计划销毁，请先执行 `restore`。
* **其他任何情况：**&#x8BF7;联系您的 Anthropic 客户团队。

## 停止使用您的存储桶

要停止使用您的存储桶，请按顺序执行以下步骤，否则您可能会丢失数据。<Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">请在取消订阅或删除项目之前执行这些步骤。这两项操作都不会将工作区从存储桶上移走。</Where>

在更改或移除工作区的存储桶之前，请先[回复其未结的 Safety Cases](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#respond-to-a-case)，并[等待它们关闭](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#after-you-respond)。如果您先更改或移除存储桶，Anthropic 可能不会读取您的响应文件。

<Warning title="保留设置文件及其状态">
  如果没有该文件，Terraform 会计划销毁状态中仍存在的所有内容，且 `prevent_destroy` 将不再阻止该计划。
</Warning>

1. 将每个工作区从存储桶上移走。在每个工作区的[数据保留](https://platform.claude.com/settings/privacy)页面上，在 **Sealed Safety Data storage** 下选择另一个存储桶。或者，选择 **Anthropic managed** 卡片，或（如果列出）**Legacy ZDR** 卡片。请在选择另一个存储桶之前与您的 Anthropic 客户团队沟通。如果 **Safety Case and Transcript storage** 仍显示此存储桶，请同时更改该设置。然后保存。

   * **如果您无法选择 Anthropic managed 卡片，**&#x8BF7;先更改您组织的[新工作区政策](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#set-organization-defaults)。
   * 任一卡片都会使工作区退出 EFS，并且 **Anthropic managed** 卡片可能会[重新开启 Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。要在选择 **Legacy ZDR** 之后再次使用 EFS，请在 **Enterprise Frontier Safeguards** 卡片上创建一个新工作区。

2. 从每个工作区发送不含敏感数据的测试请求，并添加 `--recursive --long` 运行[列出命令](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#confirm-that-objects-arrive)，直到不再有新对象到达。**Draining** 状态可能会提前显示且没有结束日期，因此请勿依赖它。

3. 阻止任何流水线应用该文件，因为 `terraform apply` 会重新创建 Anthropic 的授权。然后，与您的 Anthropic 客户团队沟通，并等待他们的答复。Anthropic 在停止写入后仍需要其访问权限，以便读取已存储的内容并删除已过期的内容。团队确认 Anthropic 已停止使用该存储桶后，仅移除 Anthropic 的授权，使用与上次 apply 时相同的值：

```bash
terraform destroy \
  -target=google_storage_bucket_iam_member.object \
  -target=google_storage_bucket_iam_member.list \
  -target=google_storage_bucket_iam_member.bucket \
  -target=google_storage_bucket_iam_member.review_object \
  -target=google_storage_bucket_iam_member.review_bucket \
  -var "project_id=acme-claude-storage" \
  -var "project_number=123456789012" \
  -var "organization_id=8a3f2c1e-5b7d-4e9a-a1c6-2f4e8d0b7c35" \
  -var "bucket_name=acme-claude-retention" \
  -var "region=us-east5"
```

计划必须不添加、不更改任何内容，并且仅销毁 `google_storage_bucket_iam_member` 资源。停用密钥并不会终止 Anthropic 的访问权限，因为这些账号仍然可以列出和删除。

</Where>
