---
title: 使用客户管理密钥
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys
description: 使用您在自己的密钥管理服务中持有的密钥加密您的数据。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

使用 “customer-managed key”（客户管理密钥，即 CMEK）时，Anthropic 会使用您在自己的 “key management service”（密钥管理服务）中持有的密钥加密您的数据：AWS KMS、Google Cloud KMS 或 Azure Key Vault。在您[禁用密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#revoke-your-key)后约一小时内，Anthropic 将无法再解密您的密钥所保护的数据。

您需要在 Claude Console 中将密钥附加到每个使用它的[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)，并通过 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS）或 [Enterprise Data Controls](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview#enterprise-data-controls) 获得访问权限。

## 附加密钥之前

您需要具有组织管理员角色才能附加密钥。

您的组织还需要获得客户管理密钥的访问权限。如需通过 EFS 获得访问权限，请按照[在 Claude Platform 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)中的"获取访问权限"部分进行操作。如需在不使用 EFS 的情况下获得访问权限，请按照[谁可以使用 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#availability) 中的说明联系您的 Anthropic 客户团队。

附加您的第一个密钥会为您的整个组织开启 [“Access Transparency”（访问透明度）](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#access-transparency-with-customer-managed-keys)，除非它已经开启。您无法将其关闭。开启 Access Transparency 后，您的组织将无法[直接从浏览器调用 Claude API](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#browser-based-applications)。

密钥会关闭部分功能，并且[附加密钥是永久性的](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key)，因此 Anthropic 建议您先阅读客户管理的加密密钥指南中的以下部分：

* [使用密钥会带来哪些变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#turned-off-or-changed)：密钥会关闭的功能。
* [密钥涵盖哪些内容](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#encrypted-with-your-key)：您的密钥加密哪些内容。
* [撤销密钥意味着什么](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)：撤销或禁用您的密钥会产生什么影响。删除密钥无法撤消。

### 如果您使用客户自有存储

同时使用客户自有存储和客户管理密钥的工作区有两个相互独立的密钥：

| 密钥 | 创建位置 | 作用 |
| ----- | ----- | ----- |
| 您的存储桶密钥 | 随您的存储桶一起创建，即在您[创建存储桶和角色](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)时 | 加密您的存储桶。 |
| 您的客户管理密钥 | 在您的密钥管理服务中，按照本页后面的指南创建 | 加密您的数据。这是您附加到工作区的密钥。 |

请勿对两者使用同一个密钥。哪些密钥加密您存储桶中的内容取决于数据类型：

* **密封的安全数据：** Anthropic 在将内容写入您的存储桶之前，会使用您的客户管理密钥对其进行加密，然后您的存储服务会使用您的存储桶密钥再次加密所存储的对象。一旦您的客户管理密钥停止工作，无论您的存储桶密钥情况如何，Anthropic 都将无法解密该存储内容。
* **对话记录和 Safety Case：** 仅使用您的存储桶密钥加密。

## 附加客户管理密钥

<Warning>
  附加密钥是永久性的。您无法分离该密钥，也无法将其换成其他密钥。支持在同一密钥内轮换密钥材料。在 Claude Platform 上，该工作区无法切换到 **Legacy ZDR**，如需使用其他密钥，您需要创建一个新的工作区。
</Warning>

<Steps>
  <Step title="创建并注册您的密钥">
    按照适用于您的密钥管理服务的指南创建密钥并向 Anthropic 注册，然后回到此处附加密钥。

    * [AWS KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms)
    * [Google Cloud KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms)
    * [Azure Key Vault](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-azure-key-vault)
  </Step>

  <Step title="附加您的密钥">
    请在工作区开始处理流量之前附加密钥，因为对于已在处理流量的工作区，密钥最多可能需要一天才能生效。

    如果工作区将使用客户自有存储，请先在 **Storage destinations** 下将[存储目标连接](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)到您的组织。如果工作区使用 EFS，请按照[在 Claude Platform 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console)中的步骤进行操作。

    要附加密钥，请对每个将使用该密钥的工作区执行以下步骤：

    1. 在 Claude Console 中前往[数据保留](https://platform.claude.com/settings/privacy)页面，并在页面顶部选择该工作区。
    2. 在**加密密钥**下选择您的密钥并保存。

    如果工作区使用 Anthropic 管理的存储，则无需其他操作。

    要在 Claude Platform on AWS 上设置密钥，请按照[在 Claude Platform on AWS 上设置 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws)中的步骤进行操作。

    要在 Claude Models on Google Cloud (Anthropic managed) 上设置密钥，请从 Google Cloud 控制台[打开 Claude Console](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#before-you-begin)。然后，按照[添加云资源](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#add-cloud-resources)中针对您自己密钥的步骤进行操作。您需要的是 `roles/claude.admin`，而不是组织管理员角色。使用密钥不需要存储桶。
  </Step>
</Steps>

## 撤销您的密钥

在 Claude Platform 上，一旦您的密钥[已生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)，在您撤销或禁用密钥后的最多一小时内，对该工作区的请求将开始失败。在您禁用密钥后约一小时内，Anthropic 将无法再解密您的密钥所保护的数据。有一小部分记录[不受您的密钥保护](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#limited-preservation-outside-your-key)。撤销或禁用您的密钥不会删除数据，也不会撤消已经发生的读取。

在撤销密钥之前，请阅读[撤销意味着什么以及哪些操作无法撤消](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)。有关您的请求会发生什么以及会通知谁，请参阅[如果您的密钥停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#if-your-key-stops-working)。

## 如果您的密钥停止工作

下表说明了在 Claude Platform 上，当工作区无法使用您的密钥时会发生什么，以及应如何处理。

| 事件 | 请求 | 您的数据 | 通知对象 | 处理方法 |
| ----- | ----- | ----- | ----- | ----- |
| 您的密钥被撤销或禁用 | 如果您的密钥已生效，请求在最多一小时内仍可能继续成功。此后，对该工作区的请求将失败。 | 一旦您的密钥停止工作，在您重新启用密钥或恢复 Anthropic 对密钥的访问权限之前，Anthropic 无法解密您的密钥所保护的内容。删除密钥无法撤消。 | Anthropic 通常会向您组织中有权管理加密密钥的成员发送电子邮件。在 Claude Models on Google Cloud (Anthropic managed) 和 Claude Platform on AWS 上，Anthropic 通常会改为向完成您组织设置的人员发送电子邮件。 | 重新启用您的密钥，或恢复 Anthropic 对它的访问权限。在此期间失败的请求不会被重试，并且保留期会继续计算。请参阅[重新启用密钥后会恢复哪些内容](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#restore-access)。 |
| 无法访问您的密钥管理服务 | 最多一小时内仍可能继续成功。此后，对该工作区的请求将失败。在 Messages API 上，这些请求会返回[代码为 `cmek_key_unreachable` 的 503 错误](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#unreachable-key)，您可以重试。Message Batches API 会为您重试这些请求，直到您的密钥可用或批处理过期。 | 已存储的内容保持不变。一旦您的密钥停止工作，在您的密钥管理服务恢复可访问之前，Anthropic 无法解密您的密钥所保护的内容。在中断期间 Anthropic 无法加密的记录不会被保留。 | 不通知任何人。 | 恢复对您的密钥管理服务的访问。 |

如果该工作区还使用客户自有存储，请参阅[如果您的存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)。

## 轮换或更换您的密钥

您可以在同一密钥内轮换密钥材料，无需在 Claude Console 中进行任何更改。

在 Claude Platform 上，如需使用其他密钥，请使用新密钥创建一个新工作区并将流量迁移过去，因为[附加密钥是永久性的](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="在 Claude Platform 上设置 Enterprise Frontier Safeguards" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console">
    对于使用 EFS 的工作区，请附加您的密钥，然后检查您的设置。
  </Card>

  <Card title="设置客户自有存储" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage">
    将您的密封的安全数据保存在您自己云账户的存储中。
  </Card>

  <Card title="客户管理的加密密钥" icon="book" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek">
    密钥涵盖哪些内容、会关闭哪些功能，以及撤销密钥意味着什么。
  </Card>
</CardGroup>
