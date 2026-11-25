---
title: Human Review 和访问透明度
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency
description: 关闭 Anthropic Human Review，并检查 Anthropic 访问您数据的记录。
---

<Warning>
  Enterprise Data Controls 处于测试阶段，适用于 Claude Platform。它依据您与 Anthropic 签订的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

Anthropic 的自动化安全监控会检查您的提示和响应，Anthropic 的一个小型专业团队可以审查其发现。这称为 Anthropic Human Review，对于使用 Anthropic 管理的存储的新工作区，它默认处于开启状态。

在 Claude Platform 上，Enterprise Data Controls 允许您在 Anthropic 管理的存储上[关闭 Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。[Enterprise Frontier Safeguards（EFS）](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)允许您[使用自己的存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage)并[自行审查](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。Claude Platform 上符合条件的企业组织可以通过在 Claude Console 的[计划](https://platform.claude.com/settings/programs)页面上接受条款来同时获得这两者。在接受之前，请阅读[获取访问权限会带来哪些变化](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)。

每当 Anthropic 中有人查看您的数据时，[Access Transparency（访问透明度）](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)都会为您提供一条记录。访问透明度并不涵盖您组织中的每一项数据。请参阅[访问透明度不涵盖的内容](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-does-not-cover)。

## 自动化安全监控何时会访问您的数据？

下表显示了自动化安全监控和 Anthropic Human Review 可能访问您数据的方式和时间：

| 谁 | 何时 | 您会获得什么记录 |
| ----- | ----- | ----- |
| 自动化安全监控系统 | 在每个使用 Anthropic 管理的存储或 Enterprise Frontier Safeguards 的工作区上。不适用于 **Legacy ZDR** 上的工作区。这些系统会检查您的提示和响应是否存在滥用。此检查完全自动化，因此在此步骤中没有任何人会看到您的内容。 | 没有 [`anthropic_access` 事件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)。 |
| Anthropic 的一个由经批准的审查人员组成的小型专业团队 | 仅在 Anthropic Human Review 开启期间。只有该团队会审查自动化安全监控的发现。关于 Anthropic 访问内容的原因类型，请参阅[原因代码](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#reason-codes)。 | 每次有人查看您的数据时，都会产生一个 `anthropic_access` 事件，前提是该访问属于[访问透明度涵盖的范围](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)。 |

使用[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage)时，Anthropic 的系统会将您的密封的安全数据写入您的存储桶，并将其读回以运行自动化安全监控。对于 Amazon S3 存储桶，请参阅 [Anthropic 在 AWS CloudTrail 中的调用](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#cloudtrail)。

## 关闭 Anthropic Human Review

如果您已为某个工作区配置 Enterprise Frontier Safeguards，则由您自己的团队而非 Anthropic 的团队收到并审查 [Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。在关闭 Anthropic Human Review 之前，请确保您已阅读[审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#where-cases-arrive)。

Anthropic 的 Claude Managed Agents 服务不会在关闭了 Anthropic Human Review 的工作区中运行会话或[梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)。该工作区的现有会话将停止接受新消息。[Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 也会拒绝该工作区中的新批处理。要运行会话、梦境或批处理，请在开启了 Anthropic Human Review 的工作区中创建它们。

要为某个工作区配置 Anthropic Human Review，请以组织管理员身份前往[数据保留](https://platform.claude.com/settings/privacy)页面。更改可能需要最多 2 小时才能生效。使用 Enterprise Frontier Safeguards 的工作区始终关闭 Anthropic Human Review。在 Claude Models on Google Cloud (Anthropic managed) 和 Claude Platform on AWS 上，使用 Enterprise Frontier Safeguards 是关闭它的唯一方式。

您可以在组织的[数据保留](https://platform.claude.com/settings/privacy)页面上[设置工作区策略](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#set-organization-defaults)，从而在您的新工作区中强制执行 Anthropic Human Review 设置。

有关更多信息，请参阅[在 Claude Platform 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console)、[在 Claude Platform on AWS 上开启 EFS](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#turn-on-efs-for-a-workspace)或[在 Claude Models on Google Cloud (Anthropic managed) 上开启 EFS](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#turn-on-efs-for-a-workspace)。

## 检查访问透明度

每当 Anthropic 中有人查看您的数据时，访问透明度都会为您提供一条记录。这些记录以 `anthropic_access` 事件的形式出现在您的 Compliance API 活动源中。关于记录内容的边界，请参阅[访问透明度涵盖的内容](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)。

对于使用 Enterprise Frontier Safeguards 或 Enterprise Data Controls 的 Claude Platform 客户，访问透明度会在您首次开始连接存储桶、附加客户管理密钥或关闭 Anthropic Human Review 时为您的整个组织开启。开启后，它会在 2 小时内开始记录。

您必须开启 Compliance API 才能读取这些事件。

如果您关闭 Compliance API，Anthropic 会继续记录访问透明度事件，并且在您重新开启它后即可读取这些事件。在它关闭期间，您组织的其他活动不会被记录，之后也无法恢复。

要读取您的访问透明度事件，您需要一个可以读取活动源的密钥。如果您使用 Claude Platform on AWS，请直接跳至[在 Claude Platform on AWS 上](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency-on-aws)。

<Steps>
  <Step title="创建密钥">
    [设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)，并创建一个可以读取活动源的 Compliance Access Key 或 Admin API 密钥。
  </Step>

  <Step title="读取 anthropic_access 事件">
    从您的[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)中读取 `anthropic_access` 事件。
  </Step>
</Steps>

在 Claude Platform 上，空结果无法说明客户自有存储中数据的任何情况，因为目前该数据没有访问透明度记录。关于您 AWS 账户中的 AWS CloudTrail 对您存储桶的记录内容，请参阅 [Anthropic 在 AWS CloudTrail 中的调用](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#cloudtrail)。

以下是解读结果的方法：

* **没有 `anthropic_access` 事件：** 您的活动源显示 Anthropic 人员没有访问您的数据，这适用于[访问透明度涵盖的范围](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)内的所有内容。事件可能会在其所记录的访问发生后最多 2 个工作日才到达。
* **一个 `anthropic_access` 事件：** Anthropic 中有人查看了您的数据。在[原因代码](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#reason-codes)中查找其原因。

要验证事件本身，请参阅[使用透明度日志验证访问透明度事件](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log)。

### 在 Claude Platform on AWS 上

在 Claude Platform on AWS 上，您需要[使用您的 AWS 凭证为每个请求签名](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#sig-v4-authentication)，而不是使用 Compliance API 密钥。要读取您的活动源，请通过 HTTPS 发送包含以下部分的 `GET` 请求：

* **主机：** `aws-external-anthropic.{region}.api.aws`，其中 `{region}` 是您某个工作区所在的区域。
* **路径：** `/v1/compliance/activities`。
* **标头：** `anthropic-version: 2023-06-01`。
* **筛选条件（可选）：** 在查询字符串中加入 `activity_types%5B%5D=anthropic_access`，以仅获取访问透明度事件。请保持方括号为编码形式，否则签名将不匹配。

您的 IAM 主体需要 `aws-external-anthropic:ListComplianceActivities` 操作权限。

以下是解读响应的方法：

* **一个列表，即使是空列表：** Compliance API 已开启。事件可能需要最多 2 个工作日才会出现。
* **一个 `anthropic_access` 事件：** Anthropic 中有人查看了您的数据。[原因代码](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#reason-codes)说明了原因。
* **一个 400 错误 `Compliance API is not enabled for this organization`：** Compliance API 已关闭。请使用 **Compliance API** 开关将其开启，等待约 1 分钟，然后再次发送活动源请求。

要开启 Compliance API，请使用 Claude Console 中[合规](https://platform.claude.com/settings/security)页面上的 **Compliance API** 开关。您的 IAM 主体需要针对 `Resource: "*"` 的 `aws-external-anthropic:UpdateComplianceSettings` 操作权限。`AnthropicFullAccess` 策略包含此操作。

## 后续步骤

<CardGroup cols={2}>
  <Card title="审查 Safety Case" icon="file" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases">
    与您自己的团队一起审查自动化安全监控的发现。
  </Card>

  <Card title="检查您的设置" icon="check" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#check-your-setup">
    在 Claude Platform 上，在开始生产使用之前，检查密封的安全数据是否能到达您的存储桶。
  </Card>
</CardGroup>
