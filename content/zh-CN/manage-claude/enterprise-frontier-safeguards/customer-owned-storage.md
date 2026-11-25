---
title: 客户自有存储
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage
description: 将数据保存在您自己云账户中的存储桶里。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

<Where cloud="aws" label="Claude Platform on AWS">

借助 Enterprise Frontier Safeguards（EFS），您可以自带存储并存储您自己的数据，并遵循您自己的访问策略和审计日志记录。在相关响应生成且提示和响应已写入您的存储桶后，Anthropic 会及时删除它们。您为以下内容提供存储：

</Where>

<Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">

客户自有存储会将某个工作区的数据保存在您自己云账户中的一个存储桶里，而不是 Anthropic 管理的存储中。对于密封的安全数据，Enterprise Frontier Safeguards（EFS）要求使用客户自有存储。对于对话记录和 Safety Case，它在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上是必需的。有三类数据可以进入您的存储桶：

</Where>

<Where cloud="claude" label="Claude Platform">

客户自有存储会将某个工作区的密封的安全数据和对话记录保存在您自己云账户中的一个存储桶里，而不是 Anthropic 管理的存储中。对于密封的安全数据，Enterprise Frontier Safeguards（EFS）要求使用客户自有存储。对于对话记录和 Safety Case，它在 Claude Platform 上是可选的。有三类数据可以进入您的存储桶：

</Where>

* **[密封的安全数据](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#what-is-stored)：** 提示、响应和模型推理，以 Anthropic 的格式加密。
* **对话记录：** 提示和响应的可读副本，供您的团队阅读。
* **Safety Correspondence：** Safety Cases、Safety Notices 以及您的 Safety Case Responses。绝不包含提示和响应。您的团队按照[审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases) 中的说明审查 Safety Case。

<Where bucket="s3" label="Amazon S3">您在自己的云账户中创建该 Amazon S3 存储桶，以及一个加密密钥和若干角色。密钥用于加密存储桶，角色则授予 Anthropic 对它的访问权限。</Where> <Where bucket="gcs" label="Google Cloud Storage">您在自己的云账户中创建该 Google Cloud Storage 存储桶、一个加密密钥、若干自定义角色，以及对 Anthropic 账户的授权。密钥用于加密存储桶，授权则授予 Anthropic 对它的访问权限。</Where><Where bucket="s3" label="Amazon S3">您账户的日志会记录 Anthropic 每次代入您某个角色的情况（请参阅 [Anthropic 在 AWS CloudTrail 中的调用](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#cloudtrail)）。</Where>您可以移除 Anthropic 的访问权限。对于密封的安全数据，受影响工作区中的请求随后可能会被拒绝，直到访问权限恢复。<Where bucket="s3" label="Amazon S3">要移除该权限，请按顺序执行[停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#stop-using-your-bucket)中的步骤。</Where> <Where bucket="gcs" label="Google Cloud Storage">要移除该权限，请按顺序执行[停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-stop-using-your-bucket)中的步骤。</Where>

Anthropic 不对客户自有存储收费，但您的云服务提供商会就存储、读取、写入和密钥操作，以及任何适用的网络传输向您收费。

<CardGroup cols={3}>
  <Where cloud="aws" label="Claude Platform on AWS">

  <Card title="在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws">
    为工作区开启 Enterprise Frontier Safeguards，并选择其存储和密钥。
  </Card>

  </Where>

  <Where cloud="claude" label="Claude Platform">

  <Where bucket="s3" label="Amazon S3">

  <Card title="设置客户自有存储" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage">
    创建您的存储桶、其密钥和角色，然后在 Claude Console 中连接它们。
  </Card>

  </Where>

  <Where bucket="gcs" label="Google Cloud Storage">

  <Card title="在 Google Cloud Storage 上设置客户自有存储" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#google-cloud-storage">
    创建您的存储桶、其密钥和角色，然后在 Claude Console 中连接它们。
  </Card>

  </Where>

  </Where>

  <Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">

  <Card title="在 Claude Models on Google Cloud (Anthropic managed) 上设置 Enterprise Frontier Safeguards" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed">
    为工作区开启 Enterprise Frontier Safeguards，并选择其存储和密钥。
  </Card>

  </Where>

  <Where bucket="s3" label="Amazon S3">

  <Card title="管理客户自有存储" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage">
    检查您存储桶的状态，修复停止工作的存储桶，以及更换或停止使用它。
  </Card>

  </Where>

  <Where bucket="gcs" label="Google Cloud Storage">

  <Card title="如果您的存储桶停止工作" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-if-your-bucket-stops-working">
    修复停止工作的存储桶，或停止使用它。
  </Card>

  </Where>

  <Where bucket="s3" label="Amazon S3">

  <Where cloud="claude" label="Claude Platform">

  <Card title="使用 Admin API 设置客户自有存储" icon="code" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?method=api">
    使用 Admin API 通过脚本完成 Claude Console 中的部分，在 API 中，External Storage 是 Anthropic 对您存储桶的记录。
  </Card>

  </Where>

  </Where>
</CardGroup>

## 存储的内容及位置

这三类数据的区别在于谁可以读取它们，以及当 Anthropic 无法写入您的存储桶时会发生什么。<Where cloud="claude" label="Claude Platform"><Where bucket="s3" label="Amazon S3">[您存储桶中的对话记录和 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#case-and-transcript-bucket)说明了哪些存储桶可以接收对话记录和 Safety Case。</Where> <Where bucket="gcs" label="Google Cloud Storage">[在工作区中使用您的 Cloud Storage 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)说明了如何为对话记录和 Safety Case 选择您的存储桶。</Where></Where> <Where cloud="aws" label="Claude Platform on AWS">EFS 也要求将您的存储桶用于对话记录和 Safety Case。</Where> <Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">EFS 也要求将您的存储桶用于对话记录和 Safety Case。</Where>

| 数据类别 | 谁可以读取 | 如果 Anthropic 无法写入您的存储桶 |
| ----- | ----- | ----- |
| 密封的安全数据：提示、响应和模型推理，以 Anthropic 的格式加密。 | 您的团队无法读取，即使是在您拥有的存储中。它供自动化安全监控使用。有关详细信息，请参阅[自动化安全监控何时会访问您的数据？](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#who-at-anthropic-can-access-your-data) | <Where bucket="s3" label="Amazon S3">对该工作区的请求在[最长 24 小时的宽限期](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)内继续正常运行。此后，Anthropic 可能会拒绝这些请求。Anthropic 会在有限时间内重试写入，并且不会改为将该数据发送到 Anthropic 管理的存储。一旦发现故障，请立即告知您的 Anthropic 客户团队，因为 Anthropic 会删除尚未到达您存储桶的数据。在 Claude Platform 上，Anthropic 会在请求发出约 72 小时后删除该数据。在任何其他平台上，最早可能在请求发出约一天后删除。</Where> <Where bucket="gcs" label="Google Cloud Storage">对该工作区的请求在最长 24 小时的宽限期内继续正常运行。此后，Anthropic 可能会拒绝这些请求。Anthropic 会重试失败的写入。请在请求发出后约 46 小时内恢复您的存储桶，并让每个工作区继续使用它，否则您可能会丢失该数据。Anthropic 最迟会在请求发出约 72 小时后停止重试，然后删除该数据。任何人都无法恢复该数据，并且它不会进入 Anthropic 管理的存储。一旦发现故障，请立即告知您的 Anthropic 客户团队。请参阅[如果您的存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-if-your-bucket-stops-working)。</Where> |
| 您的提示和响应的对话记录。这些对话记录可能包含从未导致 Safety Case 的请求，因此请规划好谁可以读取您的存储桶。 | 您的团队。Anthropic 以您的团队可以读取的形式写入它们。 | <Where cloud="claude aws" label="Claude Platform or Claude Platform on AWS">仅此存储发生故障不会导致请求被阻止。</Where>在 Anthropic 无法写入该存储期间，您不会在那里收到对话记录或 Safety Case。 |
| Safety Correspondence：Safety Cases、Safety Notices 以及您的 Safety Case Responses。 | 您的团队，以及 Anthropic（见此表格后的段落）。 | 与对话记录相同。 |

Safety Cases、Safety Notices 以及您的 Safety Case Responses（统称 Safety Correspondence）绝不包含您的提示或响应。Safety Case 包含标识符和详细信息，例如状态、时间戳、策略类别、已采取的任何执行措施，以及对相关对话记录的引用。<Where bucket="s3" label="Amazon S3">Anthropic 可以通过[审查角色](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-roles)读取您存储桶中的 Safety Case 和您的 Safety Case Responses，但其身份无法读取您的对话记录，也无法写入或更改您的 Safety Case Responses。</Where> <Where bucket="gcs" label="Google Cloud Storage">Anthropic 可以读取您存储桶中的 Safety Case 和您的 Safety Case Responses，但其身份无法读取您的对话记录，也无法写入或更改您的 Safety Case Responses。</Where>无论您的 Safety Case 被投递到何处，Anthropic 还会保留一份 Safety Correspondence 记录，经授权的 Anthropic 人员可以读取该记录。该记录不包含提示或响应。

### 两个存储设置

**Enterprise Frontier Safeguards** 卡片上的工作区有两个存储设置：

* **Sealed Safety Data storage** 设置 Anthropic 将密封的安全数据发送到何处。该设置是必需的，并且只提供您自己的存储。
* **Safety Case and Transcript storage** 设置 Anthropic 将对话记录和 Safety Case 发送到何处。<Where cloud="claude" label="Claude Platform">该设置是可选的。如果该设置下没有存储桶，则只有当您的组织[有权访问 Safety Cases API](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#get-access) 时，您的团队才能收到 Safety Case。 **Not copied to your storage** 表示 Anthropic 不会将它们复制到您的存储。<Where bucket="s3" label="Amazon S3">如果 **Sealed Safety Data storage** 下的存储桶[可以接收它们](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#case-and-transcript-bucket)，您也可以在首次保存时或之后选择它。</Where> <Where bucket="gcs" label="Google Cloud Storage">如果 Data retention 页面显示了该设置，您可以在首次保存时或之后选择同一个存储桶。请参阅[在工作区中使用您的 Cloud Storage 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)。</Where></Where> <Where cloud="aws" label="Claude Platform on AWS">EFS 要求设置该项。请参阅[在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws)。**Not copied to your storage** 表示 Anthropic 不会将它们复制到您的存储。</Where> <Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">EFS 要求设置该项。请参阅[在 Claude Models on Google Cloud (Anthropic managed) 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed)。**Not copied to your storage** 表示 Anthropic 不会将它们复制到您的存储。</Where>

对话记录和 Safety Case 会进入同一个存储桶，您不能只选择其中之一。

### 密封的安全数据的保留时长

<Where cloud="claude" label="Claude Platform">

密封的安全数据会在您发送请求后在您的存储桶中保留有限的天数。这段时间就是保留期，最长为 28 天。

</Where>

<Where cloud="gcp" label="Claude Models on Google Cloud (Anthropic managed)">

密封的安全数据会在您发送请求后在您的存储桶中保留有限的天数。这段时间就是保留期，最长为 28 天。

</Where>

<Where cloud="aws" label="Claude Platform on AWS">

密封的安全数据的保留期最长为您发送请求后 28 天。

</Where>

<Where bucket="gcs" label="Google Cloud Storage">

您存储桶自身的生命周期规则会在每个密封的安全数据对象存在满 28 天后将其删除，并至少在一天后将其永久移除。

</Where>

<Where bucket="s3" label="Amazon S3">

<Where cloud="claude" label="Claude Platform">

一旦您存储桶中某个对象的所有内容都超过保留期，Anthropic 就会删除该对象。您无需设置任何清理机制。[旧版本和某些对象会保留更长时间](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#what-removes-data)。

</Where>

</Where>
