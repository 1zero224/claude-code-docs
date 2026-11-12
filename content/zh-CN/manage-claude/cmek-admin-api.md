---
title: 使用 Admin API 添加并附加加密密钥
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-admin-api
description: 通过三次 Admin API 调用创建、验证并附加客户管理的加密密钥。
---

这些调用适用于 Claude Platform。在 Claude Enterprise 或 Claude Platform on AWS 上，请按照[您的设置页面](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#set-it-up)操作。

创建外部密钥即 Claude Console 中**添加密钥**所执行的操作，验证外部密钥即**验证**所执行的操作。

## 开始之前

* 已为您的组织[启用](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#availability) “customer-managed encryption keys”（客户管理的加密密钥），即 CMEK。
* 您拥有一个 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。示例从 `ANTHROPIC_API_KEY` 读取该密钥。
* 您已在 [AWS KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#encryption-key-setup)、[Google Cloud KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms#encryption-key-setup) 或 [Azure Key Vault](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-azure-key-vault#encryption-key-setup) 中创建了您的密钥，并保存了设置步骤中要求您复制的值。
* 您有一个[保留数据](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#data-retention)的工作区，最好是新工作区，因为[现有数据仍受 Anthropic 的密钥保护](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)。

## 创建、验证并附加您的密钥

<Steps>
  <Step title="创建外部密钥">
    <Tabs>
      <Tab title="AWS">
        将 `<friendly-name>` 替换为显示名称，将 `<key-arn-from-create-key-step>` 替换为密钥 ARN。

        <CodeGroup>
          ```bash cURL
          curl -sS "https://api.anthropic.com/v1/organizations/external_keys" \
            -H "x-api-key: $ANTHROPIC_API_KEY" \
            -H "anthropic-version: 2026-10-21" \
            -H "content-type: application/json" \
            -d '{
              "display_name": "<friendly-name>",
              "geo": "us",
              "provider_config": {
                "type": "aws",
                "kms_arn": "<key-arn-from-create-key-step>"
              }
            }'
          ```

          ```bash CLI
          ant organization:external-keys create <<'YAML'
          display_name: "<friendly-name>"
          geo: us
          provider_config:
            type: aws
            kms_arn: "<key-arn-from-create-key-step>"
          YAML
          ```

          ```python Python
          client = anthropic.Anthropic()

          external_key = client.organization.external_keys.create(
              display_name="<friendly-name>",
              geo="us",
              provider_config={"type": "aws", "kms_arn": "<key-arn-from-create-key-step>"},
          )

          print(f"id: {external_key.id}")
          print(f"display_name: {external_key.display_name}")
          ```

          ```typescript TypeScript
          const client = new Anthropic();

          const externalKey = await client.organization.externalKeys.create({
            display_name: "<friendly-name>",
            geo: "us",
            provider_config: {
              type: "aws",
              kms_arn: "<key-arn-from-create-key-step>"
            }
          });

          console.log(`id: ${externalKey.id}`);
          console.log(`display_name: ${externalKey.display_name}`);
          ```

          ```csharp C#
          using Anthropic.Models.Organization.ExternalKeys;

          AnthropicClient client = new();

          var externalKey = await client.Organization.ExternalKeys.Create(new()
          {
              DisplayName = "<friendly-name>",
              Geo = Geo.Us,
              ProviderConfig = new AwsExternalKeyConfig
              {
                  KmsArn = "<key-arn-from-create-key-step>"
              }
          });

          Console.WriteLine($"id: {externalKey.ID}");
          Console.WriteLine($"display_name: {externalKey.DisplayName}");
          ```

          ```go Go
          client := anthropic.NewClient()

          externalKey, err := client.Organization.ExternalKeys.New(context.Background(), anthropic.OrganizationExternalKeyNewParams{
          	DisplayName: anthropic.String("<friendly-name>"),
          	Geo:         anthropic.OrganizationExternalKeyNewParamsGeoUs,
          	ProviderConfig: anthropic.OrganizationExternalKeyNewParamsProviderConfigUnion{
          		OfAWS: &anthropic.AWSExternalKeyConfigParam{
          			KMSARN: "<key-arn-from-create-key-step>",
          		},
          	},
          })
          if err != nil {
          	log.Fatal(err)
          }

          fmt.Printf("id: %s\n", externalKey.ID)
          fmt.Printf("display_name: %s\n", externalKey.DisplayName)
          ```

          ```java Java
          import com.anthropic.models.organization.externalkeys.AwsExternalKeyConfig;
          import com.anthropic.models.organization.externalkeys.ExternalKeyCreateParams;

          void main() {
              AnthropicClient client = AnthropicOkHttpClient.fromEnv();

              var params = ExternalKeyCreateParams.builder()
                  .displayName("<friendly-name>")
                  .geo(ExternalKeyCreateParams.Geo.US)
                  .providerConfig(AwsExternalKeyConfig.builder()
                      .kmsArn("<key-arn-from-create-key-step>")
                      .build())
                  .build();
              var externalKey = client.organization().externalKeys().create(params);

              IO.println("id: " + externalKey.id());
              IO.println("display_name: " + externalKey.displayName().orElseThrow());
          }
          ```

          ```php PHP
          use Anthropic\Organization\ExternalKeys\ExternalKeyCreateParams\Geo;
          // ...

          $client = new Client();

          $externalKey = $client->organization->externalKeys->create(
              displayName: '<friendly-name>',
              geo: Geo::US,
              providerConfig: [
                  'type' => 'aws',
                  'kmsARN' => '<key-arn-from-create-key-step>',
              ],
          );

          echo "id: {$externalKey->id}\n";
          echo "display_name: {$externalKey->displayName}\n";
          ```

          ```ruby Ruby
          client = Anthropic::Client.new

          external_key = client.organization.external_keys.create(
            display_name: "<friendly-name>",
            geo: :us,
            provider_config: {
              type: :aws,
              kms_arn: "<key-arn-from-create-key-step>"
            }
          )

          puts "id: #{external_key.id}"
          puts "display_name: #{external_key.display_name}"
          ```
        </CodeGroup>
      </Tab>

      <Tab title="Google Cloud">
        将 `<friendly-name>` 替换为显示名称。将以 `projects/` 开头的值替换为完整的密钥资源名称。

        <CodeGroup>
          ```bash cURL
          curl -sS "https://api.anthropic.com/v1/organizations/external_keys" \
            -H "x-api-key: $ANTHROPIC_API_KEY" \
            -H "anthropic-version: 2026-10-21" \
            -H "content-type: application/json" \
            -d '{
              "display_name": "<friendly-name>",
              "geo": "us",
              "provider_config": {
                "type": "gcp",
                "key_name": "projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>"
              }
            }'
          ```

          ```bash CLI
          ant organization:external-keys create <<'YAML'
          display_name: "<friendly-name>"
          geo: us
          provider_config:
            type: gcp
            key_name: "projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>"
          YAML
          ```

          ```python Python
          client = anthropic.Anthropic()

          external_key = client.organization.external_keys.create(
              display_name="<friendly-name>",
              geo="us",
              provider_config={
                  "type": "gcp",
                  "key_name": "projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>",
              },
          )

          print(f"id: {external_key.id}")
          print(f"display_name: {external_key.display_name}")
          ```

          ```typescript TypeScript
          const client = new Anthropic();

          const externalKey = await client.organization.externalKeys.create({
            display_name: "<friendly-name>",
            geo: "us",
            provider_config: {
              type: "gcp",
              key_name:
                "projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>"
            }
          });

          console.log(`id: ${externalKey.id}`);
          console.log(`display_name: ${externalKey.display_name}`);
          ```

          ```csharp C#
          using Anthropic.Models.Organization.ExternalKeys;

          AnthropicClient client = new();

          var externalKey = await client.Organization.ExternalKeys.Create(new()
          {
              DisplayName = "<friendly-name>",
              Geo = Geo.Us,
              ProviderConfig = new GcpExternalKeyConfig
              {
                  KeyName = "projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>"
              }
          });

          Console.WriteLine($"id: {externalKey.ID}");
          Console.WriteLine($"display_name: {externalKey.DisplayName}");
          ```

          ```go Go
          client := anthropic.NewClient()

          externalKey, err := client.Organization.ExternalKeys.New(context.Background(), anthropic.OrganizationExternalKeyNewParams{
          	DisplayName: anthropic.String("<friendly-name>"),
          	Geo:         anthropic.OrganizationExternalKeyNewParamsGeoUs,
          	ProviderConfig: anthropic.OrganizationExternalKeyNewParamsProviderConfigUnion{
          		OfGCP: &anthropic.GCPExternalKeyConfigParam{
          			KeyName: "projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>",
          		},
          	},
          })
          if err != nil {
          	log.Fatal(err)
          }

          fmt.Printf("id: %s\n", externalKey.ID)
          fmt.Printf("display_name: %s\n", externalKey.DisplayName)
          ```

          ```java Java
          import com.anthropic.models.organization.externalkeys.ExternalKeyCreateParams;

          void main() {
              AnthropicClient client = AnthropicOkHttpClient.fromEnv();

              var params = ExternalKeyCreateParams.builder()
                  .displayName("<friendly-name>")
                  .geo(ExternalKeyCreateParams.Geo.US)
                  .gcpProviderConfig("projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>")
                  .build();
              var externalKey = client.organization().externalKeys().create(params);

              IO.println("id: " + externalKey.id());
              IO.println("display_name: " + externalKey.displayName().orElseThrow());
          }
          ```

          ```php PHP
          use Anthropic\Organization\ExternalKeys\ExternalKeyCreateParams\Geo;
          // ...

          $client = new Client();

          $externalKey = $client->organization->externalKeys->create(
              displayName: '<friendly-name>',
              geo: Geo::US,
              providerConfig: [
                  'type' => 'gcp',
                  'keyName' => 'projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>',
              ],
          );

          echo "id: {$externalKey->id}\n";
          echo "display_name: {$externalKey->displayName}\n";
          ```

          ```ruby Ruby
          client = Anthropic::Client.new

          external_key = client.organization.external_keys.create(
            display_name: "<friendly-name>",
            geo: :us,
            provider_config: {
              type: :gcp,
              key_name: "projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>"
            }
          )

          puts "id: #{external_key.id}"
          puts "display_name: #{external_key.display_name}"
          ```
        </CodeGroup>
      </Tab>

      <Tab title="Azure">
        替换以下占位符：

        * `<friendly-name>`：显示名称
        * `https://<your-vault-name>.vault.azure.net/`：保管库 URI
        * `<your-key-name>`：密钥名称
        * `<your-tenant-id>`：保管库的租户 ID

        <CodeGroup>
          ```bash cURL
          curl -sS "https://api.anthropic.com/v1/organizations/external_keys" \
            -H "x-api-key: $ANTHROPIC_API_KEY" \
            -H "anthropic-version: 2026-10-21" \
            -H "content-type: application/json" \
            -d '{
              "display_name": "<friendly-name>",
              "geo": "us",
              "provider_config": {
                "type": "azure",
                "vault_uri": "https://<your-vault-name>.vault.azure.net/",
                "key_name": "<your-key-name>",
                "tenant_id": "<your-tenant-id>"
              }
            }'
          ```

          ```bash CLI
          ant organization:external-keys create <<'YAML'
          display_name: "<friendly-name>"
          geo: us
          provider_config:
            type: azure
            vault_uri: "https://<your-vault-name>.vault.azure.net/"
            key_name: "<your-key-name>"
            tenant_id: "<your-tenant-id>"
          YAML
          ```

          ```python Python
          client = anthropic.Anthropic()

          external_key = client.organization.external_keys.create(
              display_name="<friendly-name>",
              geo="us",
              provider_config={
                  "type": "azure",
                  "vault_uri": "https://<your-vault-name>.vault.azure.net/",
                  "key_name": "<your-key-name>",
                  "tenant_id": "<your-tenant-id>",
              },
          )

          print(f"id: {external_key.id}")
          print(f"display_name: {external_key.display_name}")
          ```

          ```typescript TypeScript
          const client = new Anthropic();

          const externalKey = await client.organization.externalKeys.create({
            display_name: "<friendly-name>",
            geo: "us",
            provider_config: {
              type: "azure",
              vault_uri: "https://<your-vault-name>.vault.azure.net/",
              key_name: "<your-key-name>",
              tenant_id: "<your-tenant-id>"
            }
          });

          console.log(`id: ${externalKey.id}`);
          console.log(`display_name: ${externalKey.display_name}`);
          ```

          ```csharp C#
          using Anthropic.Models.Organization.ExternalKeys;

          AnthropicClient client = new();

          var externalKey = await client.Organization.ExternalKeys.Create(new()
          {
              DisplayName = "<friendly-name>",
              Geo = Geo.Us,
              ProviderConfig = new AzureExternalKeyConfigParam
              {
                  VaultUri = "https://<your-vault-name>.vault.azure.net/",
                  KeyName = "<your-key-name>",
                  TenantID = "<your-tenant-id>"
              }
          });

          Console.WriteLine($"id: {externalKey.ID}");
          Console.WriteLine($"display_name: {externalKey.DisplayName}");
          ```

          ```go Go
          client := anthropic.NewClient()

          externalKey, err := client.Organization.ExternalKeys.New(context.Background(), anthropic.OrganizationExternalKeyNewParams{
          	DisplayName: anthropic.String("<friendly-name>"),
          	Geo:         anthropic.OrganizationExternalKeyNewParamsGeoUs,
          	ProviderConfig: anthropic.OrganizationExternalKeyNewParamsProviderConfigUnion{
          		OfAzure: &anthropic.AzureExternalKeyConfigParam{
          			VaultURI: "https://<your-vault-name>.vault.azure.net/",
          			KeyName:  "<your-key-name>",
          			TenantID: "<your-tenant-id>",
          		},
          	},
          })
          if err != nil {
          	log.Fatal(err)
          }

          fmt.Printf("id: %s\n", externalKey.ID)
          fmt.Printf("display_name: %s\n", externalKey.DisplayName)
          ```

          ```java Java
          import com.anthropic.models.organization.externalkeys.AzureExternalKeyConfigParam;
          import com.anthropic.models.organization.externalkeys.ExternalKeyCreateParams;

          void main() {
              AnthropicClient client = AnthropicOkHttpClient.fromEnv();

              var params = ExternalKeyCreateParams.builder()
                  .displayName("<friendly-name>")
                  .geo(ExternalKeyCreateParams.Geo.US)
                  .providerConfig(AzureExternalKeyConfigParam.builder()
                      .vaultUri("https://<your-vault-name>.vault.azure.net/")
                      .keyName("<your-key-name>")
                      .tenantId("<your-tenant-id>")
                      .build())
                  .build();
              var externalKey = client.organization().externalKeys().create(params);

              IO.println("id: " + externalKey.id());
              IO.println("display_name: " + externalKey.displayName().orElseThrow());
          }
          ```

          ```php PHP
          use Anthropic\Organization\ExternalKeys\ExternalKeyCreateParams\Geo;
          // ...

          $client = new Client();

          $externalKey = $client->organization->externalKeys->create(
              displayName: '<friendly-name>',
              geo: Geo::US,
              providerConfig: [
                  'type' => 'azure',
                  'vaultURI' => 'https://<your-vault-name>.vault.azure.net/',
                  'keyName' => '<your-key-name>',
                  'tenantID' => '<your-tenant-id>',
              ],
          );

          echo "id: {$externalKey->id}\n";
          echo "display_name: {$externalKey->displayName}\n";
          ```

          ```ruby Ruby
          client = Anthropic::Client.new

          external_key = client.organization.external_keys.create(
            display_name: "<friendly-name>",
            geo: :us,
            provider_config: {
              type: :azure,
              vault_uri: "https://<your-vault-name>.vault.azure.net/",
              key_name: "<your-key-name>",
              tenant_id: "<your-tenant-id>"
            }
          )

          puts "id: #{external_key.id}"
          puts "display_name: #{external_key.display_name}"
          ```
        </CodeGroup>
      </Tab>
    </Tabs>

    响应中的 `id` 即外部密钥 ID。请复制它，以用于接下来的两个步骤。

    ```json
    {
      "type": "external_key",
      "id": "ekey_<id>",
      "display_name": "<friendly-name>"
    }
    ```
  </Step>

  <Step title="验证密钥">
    将 `ekey_<id>` 替换为外部密钥 ID。

    <CodeGroup>
      ```bash cURL
      curl -sS -X POST "https://api.anthropic.com/v1/organizations/external_keys/ekey_<id>/validate" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21"
      ```

      ```bash CLI
      ant organization:external-keys validate --external-key-id "ekey_<id>"
      ```

      ```python Python
      client = anthropic.Anthropic()

      validation = client.organization.external_keys.validate("ekey_<id>")

      print(f"status: {validation.status}")
      print(f"error: {validation.error}")
      ```

      ```typescript TypeScript
      const client = new Anthropic();

      const validation = await client.organization.externalKeys.validate("ekey_<id>");

      console.log(`status: ${validation.status}`);
      console.log(`error: ${validation.error}`);
      ```

      ```csharp C#
      AnthropicClient client = new();

      var validation = await client.Organization.ExternalKeys.Validate("ekey_<id>");

      Console.WriteLine($"status: {validation.Status.Raw()}");
      Console.WriteLine($"error: {validation.Error}");
      ```

      ```go Go
      client := anthropic.NewClient()

      validation, err := client.Organization.ExternalKeys.Validate(context.Background(), "ekey_<id>")
      if err != nil {
      	log.Fatal(err)
      }

      fmt.Printf("status: %s\n", validation.Status)
      fmt.Printf("error: %s\n", validation.Error)
      ```

      ```java Java
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var validation = client.organization().externalKeys().validate("ekey_<id>");

      IO.println("status: " + validation.status().asString());
      IO.println("error: " + validation.error().orElse(""));
      ```

      ```php PHP
      $client = new Client();

      $validation = $client->organization->externalKeys->validate(
          externalKeyID: 'ekey_<id>',
      );

      echo "status: {$validation->status}\n";
      echo "error: {$validation->error}\n";
      ```

      ```ruby Ruby
      client = Anthropic::Client.new

      external_key_id = "ekey_<id>"
      validation = client.organization.external_keys.validate(external_key_id)

      puts "status: #{validation.status}"
      puts "error: #{validation.error}"
      ```
    </CodeGroup>

    成功的响应如下所示：

    ```json
    { "type": "external_key_validation", "status": "success", "error": null }
    ```

    如果 `status` 为 `failure`，`error` 字段会说明原因。请参阅 [CMEK 故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting)。
  </Step>

  <Step title="将密钥附加到工作区">
    <Warning>
      附加密钥[无法撤消](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)。如果密钥被删除，Anthropic 将无法恢复它所保护的数据。
    </Warning>

    将 `<workspace-id>` 替换为工作区 ID（以 `wrkspc_` 开头），并将 `ekey_<id>` 替换为外部密钥 ID。

    <CodeGroup>
      ```bash cURL
      curl -sS -X POST "https://api.anthropic.com/v1/organizations/workspaces/<workspace-id>" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "content-type: application/json" \
        -d '{
          "external_key_id": "ekey_<id>"
        }'
      ```

      ```bash CLI
      ant organization:workspaces update \
        --workspace-id "<workspace-id>" \
        --external-key-id "ekey_<id>"
      ```

      ```python Python
      client = anthropic.Anthropic()

      workspace = client.organization.workspaces.update(
          "<workspace-id>", external_key_id="ekey_<id>"
      )

      print(f"id: {workspace.id}")
      print(f"external_key_id: {workspace.external_key_id}")
      ```

      ```typescript TypeScript
      const client = new Anthropic();

      const workspace = await client.organization.workspaces.update("<workspace-id>", {
        external_key_id: "ekey_<id>"
      });

      console.log(`id: ${workspace.id}`);
      console.log(`external_key_id: ${workspace.external_key_id}`);
      ```

      ```csharp C#
      AnthropicClient client = new();

      var workspace = await client.Organization.Workspaces.Update("<workspace-id>", new()
      {
          ExternalKeyID = "ekey_<id>"
      });

      Console.WriteLine($"id: {workspace.ID}");
      Console.WriteLine($"external_key_id: {workspace.ExternalKeyID}");
      ```

      ```go Go
      client := anthropic.NewClient()

      workspace, err := client.Organization.Workspaces.Update(
      	context.Background(),
      	"<workspace-id>",
      	anthropic.OrganizationWorkspaceUpdateParams{
      		ExternalKeyID: anthropic.String("ekey_<id>"),
      	},
      )
      if err != nil {
      	log.Fatal(err)
      }

      fmt.Printf("id: %s\n", workspace.ID)
      fmt.Printf("external_key_id: %s\n", workspace.ExternalKeyID)
      ```

      ```java Java
      import com.anthropic.models.organization.workspaces.WorkspaceUpdateParams;

      void main() {
          AnthropicClient client = AnthropicOkHttpClient.fromEnv();

          var params = WorkspaceUpdateParams.builder()
              .externalKeyId("ekey_<id>")
              .build();
          var workspace = client.organization().workspaces().update("<workspace-id>", params);

          IO.println("id: " + workspace.id());
          IO.println("external_key_id: " + workspace.externalKeyId().orElseThrow());
      }
      ```

      ```php PHP
      $client = new Client();

      $workspace = $client->organization->workspaces->update(
          workspaceID: '<workspace-id>',
          externalKeyID: 'ekey_<id>',
      );

      echo "id: {$workspace->id}\n";
      echo "external_key_id: {$workspace->externalKeyID}\n";
      ```

      ```ruby Ruby
      client = Anthropic::Client.new

      workspace_id = "<workspace-id>"
      workspace = client.organization.workspaces.update(
        workspace_id,
        external_key_id: "ekey_<id>"
      )

      puts "id: #{workspace.id}"
      puts "external_key_id: #{workspace.external_key_id}"
      ```
    </CodeGroup>

    响应会在 `external_key_id` 中显示已附加密钥的 ID。如果请求被拒绝，请[查找该错误](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#refused)。
  </Step>
</Steps>
