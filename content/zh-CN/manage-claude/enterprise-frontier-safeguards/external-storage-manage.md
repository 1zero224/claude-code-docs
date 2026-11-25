---
title: 管理客户自有存储
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage
description: 检查您存储桶的状态，在存储桶停止工作时修复它，以及更改或停止使用它。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

请在工作区将[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage)与 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview) (EFS) 一起使用后使用本页面。本页面介绍如何检查您存储桶的状态、在存储桶停止工作时修复它，以及更改或停止使用它。

## 存储桶状态

在 Claude Console 的[数据保留](https://platform.claude.com/settings/privacy)页面上，在顶部选择**组织**，然后在 **Storage destinations** 下查找每个存储桶的状态：

| 状态 | 含义 |
| ----- | ----- |
| **Needs setup** | <Where bucket="s3" label="Amazon S3">[**Test setup**](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#test-the-setup)</Where> <Where bucket="gcs" label="Google Cloud Storage">[**Test setup**](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-create-the-bucket)</Where> 尚未通过。 |
| **Ready** | 您的存储桶已通过 **Test setup**，且没有工作区使用它。 |
| **Active** | 有工作区设置为使用它。 |
| **Failed last check** | 最近一次检查未通过。请参阅[如果您的存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)。 |
| **Draining** | 现在已没有工作区使用它，但之前曾有工作区使用过。<Where bucket="s3" label="Amazon S3">[请保留您的存储桶以及 Anthropic 对它的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket)。</Where> <Where bucket="gcs" label="Google Cloud Storage">[请保留您的存储桶以及 Anthropic 对它的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-stop-using-your-bucket)。</Where> |
| **Deleting** | <Where bucket="s3" label="Amazon S3">Anthropic 正在[完成对此存储目标的删除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#delete-a-storage-destination)。</Where> <Where bucket="gcs" label="Google Cloud Storage">Anthropic 正在完成对此存储目标的删除。</Where>您无法使用它。 |

Anthropic 会在工作区首次使用存储桶之前对其进行检查。Anthropic 还会每天检查一次工作区所使用的或显示为 **Draining** 的每个存储桶。即使您存储桶用于[对话记录和 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage) 的 IAM 角色失败，每日检查也可能通过。

如果每日检查无法得出结论，状态和上次检查时间都不会改变，即使您的存储桶已停止工作也是如此。即使 **Draining** 存储桶未通过检查，它也会保持 **Draining** 状态。

如果检查失败，即使是 **Draining** 存储桶，Anthropic 通常也会向您组织中有权管理加密密钥的成员发送电子邮件。在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account) 和 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#set-up-your-account) 上，Anthropic 通常会改为向完成您组织设置的人员发送电子邮件。

<Where method="api" label="Admin API">

<Where bucket="s3" label="Amazon S3">

## 检查您的存储桶

这些 Admin API 请求使用“设置客户自有存储”页面上[对您的请求进行身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-authenticate)中所述的凭据。

获取单个存储桶以查看其状态：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/organizations/external_storage/estrg_01SDCCSbTxrXDpWc1phhtcfK \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: efs-2026-10-01"
  ```

  ```bash CLI
  ant beta:organization:external-storage retrieve \
    --external-storage-id estrg_01SDCCSbTxrXDpWc1phhtcfK
  ```

  ```python Python
  client = anthropic.Anthropic()

  storage = client.beta.organization.external_storage.retrieve(
      "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  )

  print(f"status: {storage.status.type}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const storage = await client.beta.organization.externalStorage.retrieve(
    "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  );

  console.log(`status: ${storage.status.type}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var storage = await client.Beta.Organization.ExternalStorage.Retrieve(
      "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  );

  Console.WriteLine($"status: {storage.Status.Type.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  storage, err := client.Beta.Organization.ExternalStorage.Get(
  	context.Background(),
  	"estrg_01SDCCSbTxrXDpWc1phhtcfK",
  	anthropic.BetaOrganizationExternalStorageGetParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("status: %s\n", storage.Status.Type)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var storage = client.beta().organization().externalStorage()
      .retrieve("estrg_01SDCCSbTxrXDpWc1phhtcfK");

  IO.println("status: " + storage.status().type());
  ```

  ```php PHP
  $client = new Client();

  $storage = $client->beta->organization->externalStorage->retrieve(
      externalStorageID: 'estrg_01SDCCSbTxrXDpWc1phhtcfK',
  );

  echo "status: {$storage->status->type}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  storage = client.beta.organization.external_storage.retrieve("estrg_01SDCCSbTxrXDpWc1phhtcfK")

  puts "status: #{storage.status.type}"
  ```
</CodeGroup>

要列出您的所有存储桶，请调用 `GET /v1/organizations/external_storage`。结果按从新到旧排列，每页 20 条；如果传入 `?limit=`，每页最多 1,000 条。要获取下一页，请将响应中的 `next_page` 作为 `?page=` 传入。

`status.type` 会告诉您存储桶处于其生命周期的哪个阶段。每个值都对应 Claude Console 在[存储桶状态](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#bucket-health)下显示的一种状态：

| API 状态 | Console 状态 | 含义 |
| ----- | ----- | ----- |
| `pending_configuration` | Needs setup | 已注册，尚未激活。 |
| `available` | Ready | 已激活，且没有工作区使用它。 |
| `active` | Active | 至少有一个工作区设置为使用它。 |
| `draining` | Draining | 现在已没有工作区使用它，但之前至少有一个工作区使用过。请按照[更改或停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket)中的说明，保留您的存储桶以及 Anthropic 对它的访问权限。 |

对于 `available`、`active` 和 `draining`，`status` 还包含两个字段。`healthy` 表示您的存储桶是否通过了最近一次检查，`last_checked_at` 表示该检查的运行时间。对于 `draining`，它还包含 `removal_started`，删除一旦开始，该字段即为 `true`。此时 Claude Console 会显示 **Deleting**。对于 `available` 和 `active`，`healthy` 值为 `false` 时，会在 Claude Console 中显示为 **Failed last check**。请参阅[如果您的存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)。

在您的存储桶被激活时，以及在其作为 `safeguards_storage` 附加前通过 Anthropic 的检查时，会设置 `healthy` 和 `last_checked_at`。在 Claude Platform 上，每日检查在得出结果时也会设置它们（请参阅[存储桶状态](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#bucket-health)）。如果某天的检查未得出结果，它们会保持原样，因此即使您的存储桶停止工作，`healthy` 也可能保持为 `true`。请勿仅依赖这些字段。要发现故障，请在发送请求时留意 `slab/` 或 `safeguards-data/slab/` 下是否出现新对象。

</Where>

</Where>

## 如果您的存储桶停止工作

<Where bucket="s3" label="Amazon S3">

要修复停止工作的存储桶，请检查[您的存储桶状态](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#bucket-health)，然后按照本节所述恢复您的存储桶或其 IAM 角色。

如果您在过去几小时内为工作区选择了存储目标，而 `slab/` 或 `safeguards-data/slab/` 下尚未收到任何内容，[更改可能尚未生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)。

当 Anthropic 无法写入存放[密封的安全数据](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#what-is-stored)的存储桶时，会发生以下情况：

* **请求：** 请求在最长 24 小时的宽限期内继续正常运行。此后，Anthropic 可能会拒绝对该工作区的请求。
* **您的密封的安全数据：** Anthropic 会在有限的时间内重试写入。它不会改为将这些数据发送到 Anthropic 管理的存储。

为避免丢失数据，请牢记以下几点：

* 要发现故障，请在发送请求时留意 `slab/` 或 `safeguards-data/slab/` 下是否有新对象。不要等待有关故障的电子邮件，也不要只依赖[您的存储桶状态](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#bucket-health)。
* 请尽快恢复您的存储桶。如果 Anthropic 无法写入您的存储桶超过约一小时，请立即告知您的 Anthropic 客户团队，并在您的存储桶恢复正常时再次告知他们。在您恢复存储桶后，Anthropic 需要运行一次恢复，而恢复不保证能够成功。
* 在 Claude Platform 上，密封的安全数据发送约 72 小时后，Anthropic 会删除所有尚未恢复的内容。在 Claude Platform on AWS 上，这一时间可能更短，因此请在发现故障后立即告知您的 Anthropic 客户团队。

如何恢复您的存储桶取决于发生了哪些更改：

* **IAM 角色：** 如果有任何机制移除了 Anthropic 对您存储桶的访问权限，请将角色完全恢复为原来的样子。

  * **CloudFormation：** 请勿删除或更新您的堆栈，也不要为此存储桶创建第二个堆栈。请改为立即联系您的 Anthropic 客户团队。
  * **Terraform：** 在与之前相同的文件夹中，使用相同的区域和值，再次运行您存储桶 Terraform 文件头部中的 apply 命令。如果您已不再拥有该文件夹，请立即联系您的 Anthropic 客户团队。
  * **您自己的工具：** 使用相同的名称，以及 **Connect storage** 对话框所提供文件中的信任策略和权限，重新创建这些角色。如果您没有保留该文件，请联系您的 Anthropic 客户团队。

* **您存储桶的 KMS 密钥（如果有人禁用了它）：** 重新启用它。

* **您存储桶的 KMS 密钥（如果有人计划删除它）：** 在所选的 7 到 30 天等待期结束之前取消删除。然后启用该密钥，因为 AWS 会使其保持禁用状态。等待期结束后，AWS KMS 会删除该密钥，任何人都无法读取该密钥加密的内容。

* **AWS 组织策略：** 如果某个策略现在阻止了 Anthropic，请[将 Anthropic 的 AWS 账户添加到该策略中](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#add-anthropic-to-the-rcp)。

* **您的客户管理密钥（如果该工作区有）：** 请按照[如果您的密钥停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#if-your-key-stops-working)进行操作。

如果您的角色具有固定名称（例如 `anthropic-efs-ingest`），请按照此列表所述恢复相同的设置。请勿将其迁移到其他设置文件。如需参考，请使用您部署的文件。

</Where>

<Where bucket="gcs" label="Google Cloud Storage">

请参阅[如果您的 Cloud Storage 存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-if-your-bucket-stops-working)。

</Where>

## 更改或停止使用您的存储桶

<Where bucket="s3" label="Amazon S3">

要迁移到新存储桶或停止使用您的存储桶，请按照本节中的步骤操作。无论哪种情况，都请保留旧存储桶以及 Anthropic 对它的访问权限，直到您存储桶中的所有内容都超过[保留期](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#retention-period)，且您的存储桶不再列于 [**Storage destinations**](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#bucket-health) 下。此后，[某些对象仍会保留在您的存储桶中](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#what-removes-data)。

在更改或移除工作区的存储桶之前，请先[回复其未结的 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#respond-to-a-case)，并[等待它们关闭](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#after-you-respond)。如果您先更改或移除了存储桶，Anthropic 可能无法读取您的[回复文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#response-file)。

当您的存储桶显示为 **Draining** 时，请保留该存储桶、其版本控制、其生命周期规则、其 KMS 密钥和其 IAM 角色。**Draining** 状态不会自行结束。当您[删除存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#delete-a-storage-destination)或再次为某个工作区选择该存储桶时，该状态才会结束。如果您无法删除它，请在移除 Anthropic 的访问权限之前先与您的 Anthropic 客户团队沟通。在 Claude Platform on AWS 上，一旦某个工作区使用过某个存储目标，您就无法删除它，因此它会一直显示为 **Draining**。请做好保留您的存储桶以及 Anthropic 对它的访问权限的准备。

只要您保留存储桶，就请勿禁用其 KMS 密钥或计划删除该密钥。否则，对您存储桶的写入和读取会开始失败，因此请[恢复该密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)。

如果您删除了某个工作区曾使用过的存储桶，在其存储目标仍被列出期间，请勿以相同名称再次创建它。请改为使用新名称[设置新存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-your-bucket)。

### 更换您的存储桶

要将您的工作区迁移到新存储桶，请按照以下步骤操作。如果要改为迁移到 Google Cloud Storage 存储桶，请与您的 Anthropic 客户团队沟通，因为 [Anthropic 会限制存储桶的数量](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)。在 Claude Platform on AWS 上，请在更改您的存储桶之前先咨询您的 Anthropic 客户团队。

<Warning>
  迁移不会立即生效，如[确认更改已生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)中所述。即使在对象已到达您的新存储桶之后，新的密封的安全数据在一段时间内仍可能进入您的旧存储桶。请保留旧存储桶以及 Anthropic 对它的访问权限，如[更改或停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket)中所述。如果您过早移除其中任何一项，可能会丢失新数据。
</Warning>

1. [创建并连接您的新存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)。请使用单独的堆栈创建它；如果使用 Terraform，则在单独的文件夹中创建。请勿更新现有堆栈使其指向新存储桶，否则 Anthropic 会在旧存储桶仍存放您的数据时失去对它的访问权限。
2. 在每个使用旧存储桶的工作区中，在 Claude Console 中的 **Sealed Safety Data storage** 下[选择您的新存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#turn-on-efs-for-a-workspace)。请勿先从该工作区移除旧存储桶，否则新的密封的安全数据可能会进入 Anthropic 管理的存储。如果该工作区还在 **Safety Case and Transcript storage** 下使用旧存储桶，也请在该设置下选择您的新存储桶。然后保存。

<Where method="api" label="Admin API">

在第 1 步中，[注册](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-register)并[激活](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-activate)您的新存储桶。在第 2 步中，改为使用新存储桶的 ID [发送附加请求](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-attach)。如果 `transcripts_storage` 指定的是您的旧存储桶，请在同一请求中将其设置为新存储桶的 ID。

</Where>

<Where method="console" label="Claude Console">

### 停止使用您的存储桶

要停止使用您的存储桶，请先从每个工作区移除它，最后再移除 Anthropic 对您存储桶的访问权限。停止向您的存储桶发送密封的安全数据的工作区会退出 EFS。

<Steps>
  <Step title="从每个工作区移除您的存储桶">
    在该工作区的[数据保留](https://platform.claude.com/settings/privacy)页面上，选择 **Enterprise Frontier Safeguards** 以外的卡片，然后保存。默认情况下，**Anthropic managed** 卡片会重新开启 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。[术语表](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)介绍了每种卡片。

    如果只想停止对话记录和 Safety Case，请改为在 **Safety Case and Transcript storage** 下选择 **Not copied to your storage**。如果该存储桶仍在接收密封的安全数据，请不要执行其余步骤。
  </Step>

  <Step title="在存储桶清空期间保留它">
    一旦您从每个工作区移除了您的存储桶，它就会在 **Storage destinations** 中显示为 **Draining**。只要状态为 **Draining**，就请保留您的存储桶以及 Anthropic 对它的访问权限。要结束 **Draining** 状态，请[删除存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#delete-a-storage-destination)。在 Claude Platform on AWS 上，您的存储桶会一直处于 **Draining** 状态，因此请做好保留它以及 Anthropic 对它的访问权限的准备。在删除完成或您的 Anthropic 客户团队告知您之前，请勿执行下一步。已在您存储桶中的对话记录和 Safety Case 会一直保留在那里，直到您存储桶的生命周期规则使其过期。请参阅[存储桶中的内容](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#whats-in-the-bucket)。
  </Step>

  <Step title="移除 Anthropic 的访问权限">
    要移除 Anthropic 的访问权限，请使用您创建存储桶 IAM 角色时所用的工具移除这些角色。移除 Anthropic 的访问权限后，密封的安全数据会一直保留在您的存储桶中，直到您将其删除。

    <Tabs>
      <Tab title="CloudFormation">
        删除您存储桶的堆栈。堆栈的名称在您的部署命令中：`anthropic-storage-<RoleDigest>`，对于使用固定角色名称的设置，则为 `anthropic-storage-<organization ID>`。删除堆栈会保留您的存储桶及其密钥。
      </Tab>

      <Tab title="Terraform">
        请只移除您存储桶的角色及其策略，以便保留您的数据。在存放您存储桶 Terraform 文件的文件夹中，使用您在设置时使用的区域和值，运行该 Terraform 文件头部中的移除命令。

        您会看到关于 `-target` 的警告。检查计划列出的每个资源是否都是 `aws_iam_role` 或 `aws_iam_role_policy`，然后输入 `yes`。之后再次应用会重新创建这些角色。
      </Tab>

      <Tab title="Your own tooling">
        删除该存储桶的 IAM 角色。
      </Tab>
    </Tabs>

    <Warning>
      角色被删除后，在它们恢复之前，请勿再次为工作区选择您的存储桶。在 Claude Platform on AWS 上，或者如果您的角色具有固定名称，Anthropic 不会在保存前检查处于 **Draining** 状态的存储桶，因此即使 Anthropic 无法写入您的存储桶，保存也可能会成功。如果您使用的是 CloudFormation，再次部署模板并不能恢复 Anthropic 的访问权限，因此请在选择您的存储桶之前联系您的 Anthropic 客户团队。
    </Warning>
  </Step>
</Steps>

</Where>

<Where method="api" label="Admin API">

### 停止使用您的存储桶

这些 Admin API 请求使用“设置客户自有存储”页面上[对您的请求进行身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-authenticate)中所述的凭据。

要停止使用您的存储桶，请按顺序执行以下步骤：

1. 将您的存储桶从各个工作区分离。
2. 在存储桶清空期间保留它。
3. 删除其 External Storage。
4. [移除 Anthropic 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#api-remove-access)。

要将您的存储桶从某个工作区分离，请将每个指定了您存储桶的存储字段设置为 `internal` 类型。这些字段是 `inference_data_privacy.safeguards_storage` 和 `inference_data_privacy.transcripts_storage`，您可以在同一个请求中设置两者。如果将 `safeguards_storage` 设置为 `internal` 类型，该工作区会切换到 Anthropic 管理的存储，从而不再使用 EFS。Anthropic 可能会拒绝使 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review) 保持关闭的分离操作，因此以下请求还会将 `inference_data_privacy.human_review` 设置为 `enabled` 类型，从而将 Anthropic Human Review 重新开启：

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
          "type": "internal"
        },
        "human_review": {
          "type": "enabled"
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
      type: internal
    human_review:
      type: enabled
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspace = client.beta.organization.workspaces.update(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      inference_data_privacy={
          "safeguards_storage": {"type": "internal"},
          "human_review": {"type": "enabled"},
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
        safeguards_storage: { type: "internal" },
        human_review: { type: "enabled" }
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
              SafeguardsStorage = new BetaInternalStorageParam(),
              HumanReview = new() { Type = BetaAnthropicHumanReviewParamType.Enabled },
          },
          Betas = ["efs-2026-10-01"],
      }
  );

  var type = workspace.InferenceDataPrivacy?.SafeguardsStorage?.Type.Raw();
  Console.WriteLine($"safeguards_storage: {type}");
  ```

  ```go Go
  client := anthropic.NewClient()

  internal := anthropic.NewBetaInternalStorageParam()
  workspace, err := client.Beta.Organization.Workspaces.Update(
  	context.Background(),
  	"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  	anthropic.BetaOrganizationWorkspaceUpdateParams{
  		InferenceDataPrivacy: anthropic.BetaInferenceDataPrivacyUpdateParam{
  			SafeguardsStorage: anthropic.BetaStorageMemberParamUnion{OfInternal: &internal},
  			HumanReview: anthropic.BetaAnthropicHumanReviewParam{
  				Type: anthropic.BetaAnthropicHumanReviewParamTypeEnabled,
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
  import com.anthropic.models.beta.organization.workspaces.BetaInternalStorageParam;
  import com.anthropic.models.beta.organization.workspaces.WorkspaceUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = WorkspaceUpdateParams.builder()
          .inferenceDataPrivacy(BetaInferenceDataPrivacyUpdateParam.builder()
              .safeguardsStorage(BetaInternalStorageParam.builder().build())
              .humanReview(BetaAnthropicHumanReviewParam.builder()
                  .type(BetaAnthropicHumanReviewParam.Type.ENABLED)
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
          'safeguardsStorage' => ['type' => 'internal'],
          'humanReview' => ['type' => 'enabled'],
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
        type: :internal
      },
      human_review: {
        type: :enabled
      }
    },
    betas: ["efs-2026-10-01"]
  )

  puts "safeguards_storage: #{workspace.inference_data_privacy&.safeguards_storage&.type}"
  ```
</CodeGroup>

如果[您组织的策略](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#set-organization-defaults)要求使用客户自有存储，或要求关闭 Anthropic Human Review，那么对于保留密封的安全数据的工作区，Anthropic 会以 `workspace_policy_violation` 拒绝此请求。请先更改该策略，然后再发送请求。

与附加一样，分离操作也[不会立即生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)。

在最后一次分离之后，您存储桶的状态会变为 `draining`。该状态不会自行脱离 `draining`。当您再次附加该存储桶或删除其 External Storage 时，该状态才会结束。在删除完成之前，请保留您的存储桶以及 Anthropic 对它的访问权限。如果您无法删除该 External Storage，请在移除 Anthropic 的访问权限之前与您的 Anthropic 客户团队沟通。[删除存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#delete-a-storage-destination)说明了谁可以删除它以及需要先做什么。

然后删除您存储桶的 External Storage。External Storage 是 Anthropic 对您存储桶的记录，而不是您的存储桶本身。具体结果取决于您存储桶的状态：

* `pending_configuration` 或 `available`：Anthropic 会立即删除该 External Storage。
* `active`：Anthropic 会拒绝删除。
* `draining`：第一次调用会返回 400 错误，并带有 `external_storage_in_use`。[获取该 External Storage](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#api-check)。如果 `status.removal_started` 为 `false`，错误消息会说明该怎么做。如果为 `true`，则无法撤消移除。请每隔几分钟再次发送删除请求，直到成功为止，这可能需要约 30 分钟。如果一小时后仍被拒绝，请联系您的 Anthropic 客户团队。503 错误表示没有任何更改，请稍后重试。

如果[您组织的策略](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#set-organization-defaults)列出了此存储桶，Anthropic 会拒绝删除，直到您将该存储桶从策略中移除。一旦某个工作区使用过您的存储桶，请将您的存储桶名称、KMS 密钥和 IAM 角色视为[与您的组织关联](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#before-you-begin)。

<CodeGroup>
  ```bash cURL
  curl -X DELETE https://api.anthropic.com/v1/organizations/external_storage/estrg_01SDCCSbTxrXDpWc1phhtcfK \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: efs-2026-10-01"
  ```

  ```bash CLI
  ant beta:organization:external-storage delete \
    --external-storage-id estrg_01SDCCSbTxrXDpWc1phhtcfK
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted = client.beta.organization.external_storage.delete(
      "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  )

  print(f"id: {deleted.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deleted = await client.beta.organization.externalStorage.delete(
    "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  );

  console.log(`id: ${deleted.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deleted = await client.Beta.Organization.ExternalStorage.Delete(
      "estrg_01SDCCSbTxrXDpWc1phhtcfK"
  );

  Console.WriteLine($"id: {deleted.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  deleted, err := client.Beta.Organization.ExternalStorage.Delete(
  	context.Background(),
  	"estrg_01SDCCSbTxrXDpWc1phhtcfK",
  	anthropic.BetaOrganizationExternalStorageDeleteParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", deleted.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var deleted = client.beta().organization().externalStorage()
      .delete("estrg_01SDCCSbTxrXDpWc1phhtcfK");

  IO.println("id: " + deleted.id());
  ```

  ```php PHP
  $client = new Client();

  $deleted = $client->beta->organization->externalStorage->delete(
      externalStorageID: 'estrg_01SDCCSbTxrXDpWc1phhtcfK',
  );

  echo "id: {$deleted->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  deleted = client.beta.organization.external_storage.delete("estrg_01SDCCSbTxrXDpWc1phhtcfK")

  puts "id: #{deleted.id}"
  ```
</CodeGroup>

#### 移除 Anthropic 的访问权限

要移除 Anthropic 的访问权限，请使用您创建存储桶 IAM 角色时所用的工具移除这些角色。移除 Anthropic 的访问权限后，密封的安全数据会一直保留在您的存储桶中，直到您将其删除。在存放对话记录和 Safety Case 的存储桶中，生命周期规则会持续使这些文件过期，直到您更改这些规则。

<Tabs>
  <Tab title="CloudFormation">
    删除您存储桶的堆栈。堆栈的名称在您的部署命令中：`anthropic-storage-<RoleDigest>`，对于使用固定角色名称的设置，则为 `anthropic-storage-<organization ID>`。删除堆栈会保留您的存储桶及其密钥。
  </Tab>

  <Tab title="Terraform">
    请只移除您存储桶的角色及其策略，以便保留您的数据。在存放您存储桶 Terraform 文件的文件夹中，使用您在设置时使用的区域和值，运行该 Terraform 文件头部中的移除命令。

    您会看到关于 `-target` 的警告。检查计划列出的每个资源是否都是 `aws_iam_role` 或 `aws_iam_role_policy`，然后输入 `yes`。之后再次应用会重新创建这些角色。

    不带 `-target` 的 `terraform destroy` 会失败且不会销毁任何内容，因为密钥、存储桶、其设置及其存储桶策略都标记了 `prevent_destroy`。
  </Tab>

  <Tab title="Your own tooling">
    删除该存储桶的 IAM 角色。
  </Tab>
</Tabs>

<Warning>
  角色被删除后，在它们恢复之前，请勿再次为工作区选择您的存储桶。在 Claude Platform on AWS 上，或者如果您的角色具有固定名称，Anthropic 不会在保存前检查处于 **Draining** 状态的存储桶，因此即使 Anthropic 无法写入您的存储桶，保存也可能会成功。如果您使用的是 CloudFormation，再次部署模板并不能恢复 Anthropic 的访问权限，因此请在选择您的存储桶之前联系您的 Anthropic 客户团队。
</Warning>

</Where>

</Where>

<Where bucket="gcs" label="Google Cloud Storage">

在迁移到新存储桶之前，请先与您的 Anthropic 客户团队沟通，因为 [Anthropic 限制存储桶的数量](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)。

请参阅[如何停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-stop-using-your-bucket)。

</Where>

<Where bucket="s3" label="Amazon S3">

## 删除存储目标

要结束 Anthropic 对显示为 **Draining** 的存储桶的使用，请删除其存储目标。在 Claude Platform on AWS 上，一旦某个工作区使用过某个存储目标，您就无法删除它。在 Claude Platform 上，仅当 Anthropic 的记录显示您的组织仅将 Claude 用于内部工具时，您才能删除 Amazon S3 存储桶的存储目标。如果您无法删除它，请在移除 Anthropic 的访问权限之前与您的 Anthropic 客户团队沟通。删除操作无法撤消。

<Where cloud="claude" label="Claude Platform">

在删除之前，请执行以下操作：

* **等待 `slab/` 和 `media/` 清空**，包括存储桶顶层和 `safeguards-data/` 下的这两个前缀。其中的大部分内容会在发送到那里的最后一批数据的[保留期](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#retention-period)结束几天后自行清除。请勿在此之前删除任何内容。否则您会丢失数据，而 Anthropic 不会向您发出警告。如果此后仍有[对象保留](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#what-removes-data)，请在不指定版本 ID 的情况下删除它们。旧版本和删除标记也计算在内，因此请等待生命周期规则清除它们，即每次删除后至少 3 天。
* **保留 Anthropic 的访问权限**，直到删除完成，因为 Anthropic 会先检查该存储桶。

<Warning>
  删除存储目标不会从您的存储桶中移除任何内容。对话记录、Safety Case 和您团队的回复可能仍在其中，任何有权访问该存储桶及其密钥的人都可以读取它们。在任何人再次将该存储桶用于客户自有存储之前，请清空该存储桶，包括较早的文件版本。
</Warning>

<Where method="console" label="Claude Console">

在[数据保留](https://platform.claude.com/settings/privacy)页面的 **Storage destinations** 下，在该行的菜单中选择 **Delete**。如果 Claude Console 拒绝删除，其消息会说明原因。如果几分钟后该行仍被列出，请在其菜单中选择 **Finish deleting**。如果一小时后仍被列出，请联系您的 Anthropic 客户团队。该行消失后，请按照[停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#stop-using-your-bucket)的最后一步移除 Anthropic 的访问权限。然后您就可以删除您的存储桶。

</Where>

<Where method="api" label="Admin API">

按照[停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#api-stop)中的说明发送删除请求。这需要多次调用。成功后，请[移除 Anthropic 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#api-remove-access)。然后您就可以删除您的存储桶。

</Where>

</Where>

</Where>

## 哪些操作会从您的存储桶中移除数据

<Where bucket="s3" label="Amazon S3">

<Where cloud="claude" label="Claude Platform">

Anthropic 通过 [maintenance 角色](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-roles)删除 `slab/` 和 `media/` 下的对象。只有当 `slab/` 下某个对象中的所有内容都超过保留期后，Anthropic 才会删除该对象。

</Where>

如果您的存储桶使用 `safeguards-data/`，本节和[哪些机制保护您存储桶中的对象版本](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#what-protects-object-versions)适用于 `safeguards-data/` 下的相同前缀。

由于您的存储桶保留版本，maintenance 角色执行的删除操作只会添加一个删除标记，旧版本仍会保留。生命周期规则会在至少 3 天后永久删除每个旧版本，然后移除删除标记。

有些对象会保留在您的存储桶中，因为设置文件中没有任何生命周期规则会删除它们：

* **`slab/` 下：** Anthropic 会在每天的对象中查找要删除的对象，但只会在有限的天数内进行查找。到那时仍未被它删除的对象会保留在您的存储桶中。
* **`slab-manifest/` 下：** Anthropic 的软件从不删除此前缀下的对象，因此它们会不断累积。
* **`media/` 下：** 中断或重复的写入可能会留下 Anthropic 不再跟踪的对象。它们仍保持加密状态。每月一次的清理会查找这些对象，但可能会将它们保留在原处。

请保持 Anthropic 写入的对象不变；在工作区使用您的存储桶期间，这些残留对象也请保持不变。如果您无法保留它们，请联系您的 Anthropic 客户团队，除非您正在[删除存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#delete-a-storage-destination)。

请仅通过设置 [`ReviewExpiryDays`](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#template-reference) 来更改您存储桶的生命周期规则。不要开启 S3 Object Lock，也不要添加使对象过期或归档对象的生命周期规则。否则，您的存储桶将无法通过 [**Test setup**](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#test-the-setup) 和 [Anthropic 的每日检查](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#bucket-health)。

### 哪些机制保护您存储桶中的对象版本

只有生命周期规则可以永久删除 `slab/` 下的对象版本。只要版本控制处于开启状态，且存储桶策略保留其 `NobodyHardDeletesSlabVersions` 语句，情况就是如此。该语句拒绝所有人员和角色（包括 Anthropic 的角色）在 `slab/` 下执行 `s3:DeleteObjectVersion`。

Anthropic 的角色无法更改存储桶策略、版本控制或生命周期规则。

存储桶策略中没有针对其他前缀的类似 `NobodyHardDeletesSlabVersions` 的语句。media-erasure 角色可以永久删除 `media/` 下的对象版本。transcripts 和 review 角色可以永久删除对话记录、Safety Case、摘要和 Safety Notice，但不能删除您团队的回复。

</Where>

<Where bucket="gcs" label="Google Cloud Storage">

您存储桶自身的生命周期规则会在每个密封的安全数据对象存在满 28 天后将其删除。请勿更改这些规则。

</Where>

<Where bucket="s3" label="Amazon S3">

## Anthropic 在 AWS CloudTrail 中的调用

每次 Anthropic 代入您的某个角色时，AWS CloudTrail 都会在您的账户中记录该调用。您可以在 CloudTrail **Event history** 中查找这些记录：

* **在哪里查找：** 请在 `us-east-2` 中查找。**Event history** 一次只显示一个 AWS 区域，并且无需任何设置即可将这些记录保留 90 天。
* **记录显示的内容：** 每条记录都带有一个表明工作类型的源身份，该会话中之后的每个调用也都带有它。
* **Event history 不包含的内容：** **Event history** 不会显示对象的读取、写入和删除，这些属于 Amazon S3 数据事件。要让 CloudTrail 记录它们，请为您的存储桶开启数据事件日志记录。AWS 会对该日志记录收费。

### 来自 Test setup 和每日检查的调用

**Test setup** 和[每日检查](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#bucket-health)会发送缺少外部 ID 或源身份、或者其值错误的请求。您的角色必须拒绝这些请求，因此这些拒绝属于预期情况。如果某个角色拒绝了带有您的外部 ID 和该角色所接受的源身份的请求，则该拒绝不属于预期情况。这可能意味着[某个 AWS 组织策略阻止了 Anthropic](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#if-an-aws-organization-policy-blocks-anthropic)。

</Where>

<Where method="api" label="Admin API">

## 故障排除

<AccordionGroup>
  <Accordion title="每个设置请求都返回错误，或提示 inference_data_privacy 不是有效字段">
    在您的组织拥有 [Enterprise Frontier Safeguards 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)之前，设置 External Storage 的请求会返回错误。如果请求未发送 beta 标头，也会返回错误。使用 OAuth bearer 令牌时，如果该令牌缺少 `org:admin` 范围，也会返回错误。在没有该标头的情况下，附加和分离请求会返回另一种错误，指出 `inference_data_privacy` 不是有效字段。如果在已具备访问权限且已添加标头的情况下仍出现这种情况，请联系您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="激活请求返回失败的检查">
    如果某项检查失败，请求会返回 400 错误，其 `error.details.error_code` 设置为 `storage_verification_failed`。`error.details.failed_checks` 会将每个问题列为一个包含 `check`、`role` 和 `message` 的对象。修复 `failed_checks` 中列出的每个问题后，再次调用激活。`already_registered` 检查有其专门的修复方法，见本条目后文。

    如果第一个 `message` 以 “Anthropic could not assume the verify role” 开头，请按顺序检查以下内容：

    1. 查看您的 IAM 角色的名称。角色使用固定名称（例如 `anthropic-efs-ingest`）的新存储桶无法通过此检查。对于此类存储桶，请删除此 External Storage，[注册一个新的存储桶名称](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-register)，并使用[获取您的设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?method=api#get-the-setup-file)中的文件。
    2. 否则，请检查您是否使用[获取您的设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?method=api#get-the-setup-file)中的文件创建了设置。
    3. 检查是否存在[阻止 Anthropic 的 AWS 组织策略](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#if-an-aws-organization-policy-blocks-anthropic)。

    如果 `failed_checks` 中只有一个 `check` 设置为 `already_registered` 的对象，则说明另一个 Anthropic 组织已使用过此设置的一部分，或者该存储桶名称已在其他 AWS 账户中使用。已删除组织的 External Storage 仍可能算作其中一种使用。

    如果另一个组织已使用过此 AWS 账户，您可以继续使用该账户。请先检查您是否为此存储桶部署了[获取您的设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?method=api#get-the-setup-file)中的文件，然后再次调用激活。

    如果检查仍然失败，或者没有其他组织使用过此 AWS 账户，请使用新的存储桶名称，并为其配备独立的密钥和角色，然后[注册新存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-register)。如果新存储桶以同样的方式失败，请告知您的 Anthropic 客户团队。

    [附加请求](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-attach)也可能因 `already_registered` 检查而失败。如果该检查对 `safeguards_storage` 失败，修复方法相同。如果该检查对 `transcripts_storage` 失败，请联系您的 Anthropic 客户团队。如果您的角色使用固定名称，附加时的 `already_registered` 失败也可能意味着另一个组织已使用过此 AWS 账户。

    如果激活请求或[附加请求](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#api-attach)返回 503 错误且没有 `error.details.error_code`，则说明 Anthropic 无法运行其检查，请在几分钟后重试。如果问题持续出现，请告知您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="附加请求返回与 Anthropic 批准相关的错误">
    Anthropic 可以为您的整个组织或为选定的工作区批准客户自有存储。此请求可能返回两种与该批准相关的错误，在这两种情况下都不会做任何更改。注册您的存储桶和激活它也可能返回相同的两个代码。这些步骤不涉及工作区，因此消息针对的是您组织的批准，而不是某个工作区的批准。

    | 您收到的内容 | 含义 | 应采取的措施 |
    | ----- | ----- | ----- |
    | 400 错误，其 `error.details.error_code` 设置为 `grant_required` | Anthropic 的批准不涵盖此工作区。消息会说明该工作区未获批准使用 External Storage，并请您联系 Anthropic 客户团队。 | 请您的 Anthropic 客户团队为该工作区办理批准，然后再次发送请求。 |
    | 503 错误，其 `error.details.error_code` 设置为 `authorization_check_unavailable`，并带有 `Retry-After` 标头 | 当时无法检查 Anthropic 的批准。 | 等待 `Retry-After` 标头中给出的秒数，然后再次发送相同的请求。 |

    在联系您的 Anthropic 客户团队之前，请检查组织管理员是否已[接受 EFS 和 Enterprise Data Controls 的条款](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)。新的批准可能需要一小段时间才能同步到 API，因此在您获得批准后立即发送的请求，仍可能返回 `grant_required`。请再次发送，如果问题持续出现，请联系您的 Anthropic 客户团队。

    附加请求还可能返回 `approval_required`。要获得 Anthropic 关于关闭 Anthropic Human Review 的批准，请参阅[在 Claude Platform 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)。
  </Accordion>

  <Accordion title="附加请求拒绝在 transcripts_storage 中使用您的存储桶">
    请求会返回 400 错误。检查失败不会做任何更改，因此同一请求中的 `safeguards_storage` 也不会被附加。应采取的措施取决于错误：

    * 如果 `error.details.error_code` 为 `storage_verification_failed`，且 `error.details.failed_checks` 中只有一个 `check` 为 `transcripts_support` 的对象，则再次发送请求无济于事。该对象的 `message` 会说明原因。[客户自有存储故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#troubleshooting)说明了应如何处理。
    * 如果 `check` 为 `already_registered`，或者错误表明暂时无法为对话记录附加存储，请联系您的 Anthropic 客户团队。
    * 对于其他任何检查，请按照其 `message` 所述进行修复，然后再次发送请求。

    在上述每种情况下，您都可以发送不带 `transcripts_storage` 的相同请求。如果 Anthropic 接受该请求，则只会附加 `safeguards_storage`。如果不带 `transcripts_storage` 的请求返回 `transcripts_storage_required`，请使用能够接收对话记录和 Safety Case 的存储桶，或联系您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="您存储桶的默认加密密钥已更改">
    如果在您激活存储桶之后其默认加密密钥发生了变化，且尚无工作区使用该存储桶，Anthropic 会拒绝附加，因为它会在此时再次检查该存储桶。要在使用您自己的工具创建的 IAM 角色下使用另一个密钥，请按以下步骤操作：

    1. 删除此 External Storage。
    2. 允许除 `media-erasure` 之外的每个角色使用新密钥，包括 `transcripts` 和 `review` 角色的写入条件中指定该密钥的位置。
    3. 在新密钥的策略中，允许 Amazon S3 调用 `kms:GenerateDataKey`（与您设置文件中的密钥策略相同），以便 S3 Inventory 能够提交其每日报告。
    4. 将您存储桶的默认加密设置为新密钥，并通过其密钥 ARN 指定。该密钥必须满足[创建您的存储桶及其 KMS 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket)中 **Your own tooling** 选项卡对存储桶密钥的要求。
    5. 重新注册您的存储桶并激活它。

    Anthropic 的 CloudFormation 模板和 Terraform 文件只为一个密钥设置角色，不提供受支持的方式来使用另一个密钥。对于使用固定角色名称（例如 `anthropic-efs-ingest`）的设置，请勿再次注册同一个存储桶。请使用[获取您的设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?method=api#get-the-setup-file)中的文件创建一个新的存储桶。
  </Accordion>

  <Accordion title="工作区请求中只有部分更改被保存">
    如果响应表明请求中的部分更改可能已保存而其他更改未保存，请再次发送相同的请求。已保存的更改不会重复执行。

    请勿重新发送返回 `workspace_created_incomplete` 的创建请求，因为该工作区已经存在，第二次创建会生成第二个工作区。请改为按照响应中的消息操作，该消息通常会要求您更新 `error.details.workspace_id` 所指定的工作区。在具有 Legacy Zero Data Retention (ZDR) 安排的组织中，如果消息要求您更新 `inference_data_privacy`，请归档该工作区并重新创建它。
  </Accordion>
</AccordionGroup>

</Where>
