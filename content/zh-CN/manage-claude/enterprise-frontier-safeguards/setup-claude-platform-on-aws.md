---
title: 在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws
description: 为 Claude Platform on AWS 工作区开启 Enterprise Frontier Safeguards，并选择其存储和密钥。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

您需要在 Claude Console 中设置 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS），而 Claude Console 需从 AWS Console 打开。EFS 适用于您开启了它的工作区中发往每个模型的每个请求。要决定哪些工作区应使用它，请参阅[规划您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#plan-your-setup)。

设置分为三个部分：

1. 添加云资源：您的存储桶和加密密钥。
2. 为工作区开启 EFS。
3. 检查您的设置。

## 规划您的设置

请先规划您的设置，因为有些选择无法撤消。

首先，决定哪些[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)使用 Enterprise Frontier Safeguards。如需仅对部分流量使用 EFS，请使用单独的工作区。Default Workspace（默认工作区）无法使用 EFS。

如果您的组织有 Legacy Zero Data Retention（ZDR）安排，请为 EFS 使用一个新工作区，并在创建该工作区时选择 **Enterprise Frontier Safeguards** 卡片。在 Claude Console 中，对于使用 **Legacy ZDR** 的现有工作区，您无法选择该卡片。

然后，查看哪些步骤可以撤消，以及每个步骤还会改变什么。只有[客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys)是可选的。

| 步骤 | 能否撤消？ | 还会改变什么 |
| ----- | ----- | ----- |
| 为组织获取访问权限 | 无法自行撤消。[访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)一旦开启，您将无法关闭它。如需移除您组织的访问权限，请联系您的 Anthropic 客户团队。 | 为您的组织开启 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)（请参阅[接受条款](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)）。在 Claude Platform 上，在您配置工作区后，[访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)也会开启。获取访问权限不会为任何工作区开启 EFS。 |
| [附加客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key) | 否。这是永久性的。 | 会关闭部分功能。请参阅[附加密钥后的变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#turned-off-or-changed)。 |
| 使用[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage) | 部分可以。您可以[更改或停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket)，但停止使用会使该工作区退出 EFS。 | 需要关闭 Anthropic 的 [Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)，因此必须由您组织中的人员审查 [Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#where-cases-arrive)。请规划仅将该存储桶用于[一个 Anthropic 组织](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#before-you-begin)。 |

最后，请回答以下问题：

<AccordionGroup>
  <Accordion title="我的存储桶应该放在哪里？费用是多少？">
    在开始之前，请选择存储桶所在的云账户和区域：

    * **Amazon S3：** 请参阅[创建并连接您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)和[受支持的 AWS 区域](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket)。Anthropic 建议使用 `us-east-2`。

    如果您的 AWS 组织使用资源控制策略（RCP）阻止外部账户，请尽早申请[为 Anthropic 的 AWS 账户设置例外](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#if-an-aws-organization-policy-blocks-anthropic)。

    Anthropic 不对 EFS 收费，但您的云提供商会向您收取存储、读取、写入和密钥操作费用，以及任何适用的网络传输费用。Anthropic 从其自身的区域读写您的存储桶，这可能会影响您的网络传输费用。
  </Accordion>

  <Accordion title="我的团队中需要哪些人员参与？">
    要设置 EFS 并在之后审查 Safety Case，您需要以下人员：

    | 人员 | 职责 |
    | ----- | ----- |
    | 云管理员 | 针对 [Amazon S3](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)，运行来自 Claude Console 的设置文件，以创建存储桶和密钥，并向 Anthropic 授予访问权限。 |
    | 组织管理员 | 在 Claude Console 中连接存储桶、为每个工作区开启 EFS，并附加任何密钥。 |
    | 您的审查人员 | [审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。 |
  </Accordion>
</AccordionGroup>

## 开始之前

要设置 Enterprise Frontier Safeguards，您需要具备以下条件：

* Claude Console 中的 Admin 角色。要获得该角色，请前往 AWS Console 中的 **Claude Platform on AWS** 服务页面，选择 **Admin**，然后点击 **Sign in**。在您通过本页面的链接前往 Claude Console 之前，请先以这种方式登录。每个链接都会在您的浏览器当前登录的 Anthropic 组织中打开。
* 为您的组织开通的 [EFS 访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#what-getting-access-changes)。
* 您的 IAM 主体所需的 [IAM 权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#iam-permissions)。

即使您已经拥有工作区，也请为 EFS 使用一个新工作区。Anthropic 建议您将其 **AWS region** 设置为 `us-east-2`，以避免网络传输费用，但这不是必需的。区域之后无法更改。

如果您的组织有 Legacy 零数据保留（ZDR）安排，请按照以下步骤操作：

1. [添加您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#add-cloud-resources)，确认其通过 **Test setup**，然后返回此列表。
2. 阅读[为工作区开启 EFS](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#turn-on-efs-for-a-workspace)。
3. 在[工作区](https://platform.claude.com/settings/workspaces)页面上创建工作区。创建时，选择 **Enterprise Frontier Safeguards** 卡片，并在两项存储设置下都选择您的存储桶。如果您无法选择该卡片或您的存储桶，或者 Claude Console 未能完成工作区的创建，请联系您的 Anthropic 客户团队。
4. 要使用您自己的密钥，请[创建并注册该密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#add-cloud-resources)，然后在该工作区的[数据保留](https://platform.claude.com/settings/privacy)页面上附加它。
5. 前往[检查您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#check-your-setup)。

否则，请在开始之前，在[工作区](https://platform.claude.com/settings/workspaces)页面上创建工作区，并且不要选择 **Enterprise Frontier Safeguards** 卡片。该卡片需要您的存储桶，而您需要在创建工作区之后才添加存储桶。在您[为其开启 EFS](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#turn-on-efs-for-a-workspace) 之前，请勿向该工作区发送任何请求。

## 获取访问权限会带来哪些变化

<Warning>
  接受条款会为您的组织以及关联到您 [Verification Portal](https://portal.anthropic.com) 配置文件的任何其他符合资格的组织开启 Compliance API。
</Warning>

接受条款还会使组织管理员能够关闭 Anthropic Human Review。如果您的 Anthropic 客户团队已为您的组织提供了访问权限，请跳过本节。

只有符合资格的组织才能获得 Enterprise Frontier Safeguards。如需申请访问权限，请联系您的 Anthropic 客户团队。如果您没有客户团队，请填写 [Enterprise Frontier Safeguards 申请表](https://claude.com/form/enterprise-frontier-safeguards)。EFS 目前仅供您的组织内部使用。如果您的组织开启了 [HIPAA 就绪](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#hipaa-readiness)，则无法获得 EFS。EFS 无法与某些 Anthropic 访问计划结合使用，因此请告知您的 Anthropic 客户团队您的组织使用了哪些计划。

接受条款所改变的不仅仅是您计划使用的工作区。请在接受之前阅读以下主要变化：

<AccordionGroup>
  <Accordion title="访问透明度一旦开启，您将无法关闭它">
    [Access Transparency（访问透明度）](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)会记录 Anthropic 人员对[其所涵盖数据](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)的访问。它何时开启取决于您在何处使用 Claude：

    * 在 Claude Platform 上，它会在您首次开始连接存储桶、附加客户管理密钥或关闭 Anthropic Human Review 时开启。如果某个工作区已有密钥、存储桶或已关闭 Anthropic Human Review，访问透明度会在您获取访问权限时开启。
    * 在 Claude Models on Google Cloud (Anthropic managed) 或 Claude Platform on AWS 上，它会在您获取访问权限时开启。

    一旦开启，它就对该组织中的每个工作区都处于开启状态。如果同一[父组织](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#which-key-do-you-need)下的其他 Claude Console 组织现在或之后开启了访问透明度，您在该组织中同样无法关闭它。现在或之后关联到您的 Verification Portal 配置文件的组织也可能会开启访问透明度。要了解您的哪些组织开启了访问透明度，请询问您的 Anthropic 客户团队。
  </Accordion>

  <Accordion title="Compliance API 将开启并开始记录活动">
    如果其中任何组织有[父组织](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#which-key-do-you-need)，这会发生在该父组织下的每个组织中，即使之前有人已将 Compliance API 关闭。如果有人使用 Claude 订阅计划登录 Claude Code 和 Cowork 等应用，Compliance API 还会存储会话对话记录。
  </Accordion>

  <Accordion title="Anthropic 可能会将数据复制到您的存储中">
    获得访问权限还意味着，配置为保留数据的工作区（无论是保留在 Anthropic 管理的存储中还是您自己的存储桶中）可能会保留您的 API 请求和响应的副本。只有当您连接 **Safety Case and Transcript storage**，或者当 Anthropic [将某份副本标记供您审查](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)时，您的团队才能读取该副本。之后重新配置工作区或组织设置不会删除已保留的副本。
  </Accordion>
</AccordionGroup>

### 接受条款

要接受条款，请前往 Claude Console 中的[计划](https://platform.claude.com/settings/programs)页面，并在 **Enterprise Frontier Safeguards** 卡片上点击 **Accept terms**。如果您在那里看不到 **Accept terms**，请在同一页面上点击 **Open Verification Portal**，然后在 Verification Portal 中点击 **Accept terms**。

如果您无法打开 Verification Portal，或在那里也看不到 **Accept terms**，请向您的 Anthropic 客户团队申请访问权限。如果您没有客户团队，请填写 [Enterprise Frontier Safeguards 申请表](https://claude.com/form/enterprise-frontier-safeguards)。

## 1. 添加云资源

要使用 Enterprise Frontier Safeguards，您需要自己的存储桶。您自己的加密密钥是可选的。如果没有您自己的密钥，工作区将使用 Anthropic 管理的密钥。要添加您的存储桶、密钥，请在 Claude Console 中前往[数据保留](https://platform.claude.com/settings/privacy)页面，然后在页面顶部选择 **组织**。

要使用您自己的密钥，请在 AWS KMS 中、工作区所在区域内创建[客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws)，然后在 **加密密钥** 下向 Anthropic 注册该密钥。链接指南中的示例密钥策略需要工作区的区间 ID，因此请先创建工作区。

要为[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=aws)设置您的存储桶，请阅读[存储桶及其 AWS 账户需要满足的条件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=aws#before-you-begin)，然后[创建并连接您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=aws#create-the-bucket-and-roles)。在 Claude Console 中，**Connect storage** 对话框会在 **AWS account ID** 中填入您注册时所用 AWS 账户的 ID。如果要在另一个 AWS 账户中创建存储桶，请更改该 ID。**Test setup** 通过后，工作区即可使用您的存储桶。

## 2. 为工作区开启 EFS

要为工作区开启 Enterprise Frontier Safeguards，请在 Claude Console 中前往[数据保留](https://platform.claude.com/settings/privacy)页面。为该工作区选择 **Enterprise Frontier Safeguards** 和您的存储，如有需要再添加您自己的密钥，然后保存一次。

<Steps>
  <Step title="选择 Enterprise Frontier Safeguards">
    如果您的组织有 Legacy 零数据保留（ZDR）安排，您需要创建一个新工作区才能使用 EFS。Claude Console 会将使用 **Legacy ZDR** 的现有工作区的 **Enterprise Frontier Safeguards** 卡片标记为 **Ineligible**。要使用 EFS，请按照[开始之前](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#before-you-begin)所述，在创建工作区时选择该卡片。在完成[检查您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws#check-your-setup)之前，请让您的流量继续使用现有工作区。

    否则，请在页面顶部，从 **组织** 切换到您为 EFS 创建的工作区。然后，选择 **Enterprise Frontier Safeguards** 卡片。

    如果 Claude Console 将该卡片标记为 **Ineligible**，该标记的提示信息会说明原因以及您该怎么做。如果 Claude Console 隐藏了该卡片，请联系您的 Anthropic 客户团队。

    剩余步骤请停留在此页面上，并且在最后一步之前不要保存。
  </Step>

  <Step title="选择您的存储">
    <Warning>
      在[您选择的存储桶生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=aws#how-long-changes-take)之前，密封的安全数据可能会进入 Anthropic 管理的存储，即使来自新工作区、即使您的组织有 Legacy ZDR 安排也是如此。在第一个新对象出现在您存储桶的 `slab/` 或 `safeguards-data/slab/` 下满 10 分钟之前，请仅向此工作区发送不含敏感数据的测试请求。
    </Warning>

    请在该工作区的两项存储设置下都选择您的存储桶：

    * **Sealed Safety Data storage：** 设置 Anthropic 发送[密封的安全数据](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage?cloud=aws#what-is-stored)（即以 Anthropic 的格式加密的提示、响应和模型推理）的位置。即使在您自己的存储桶中，您的团队也无法读取密封的安全数据。
    * **Safety Case and Transcript storage：** 设置 Anthropic 发送对话记录和 [Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases) 以供您的团队读取的位置。

    请在 **Safety Case and Transcript storage** 下选择您的存储桶，即使 Claude Console 将该设置标记为可选。EFS 要求在此处选择存储桶。如果将该设置留空，您的保存可能会失败，或者 Anthropic Human Review 可能会保持开启。

    选择存储桶时，请牢记以下几点：

    * 如果 Claude Console 未显示 **Safety Case and Transcript storage**，请在保存之前联系您的 Anthropic 客户团队。如果 Claude Console 拒绝您的存储桶，请在连接其他存储桶之前联系您的 Anthropic 客户团队。
    * 在 **Safety Case and Transcript storage** 下保存存储桶后，只要该工作区使用 EFS，您就无法清除该设置。要[迁移到另一个存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-your-bucket)，请先咨询您的 Anthropic 客户团队。
    * 如果您的存储桶停止工作，请在发现后立即告知您的 Anthropic 客户团队。在故障期间，Anthropic 无法将数据写入您的存储桶。在这些数据发送后约 1 天，Anthropic 会删除所有未被恢复的数据。请参阅[如果您的存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)。

    一旦有工作区使用您的存储桶，该存储桶就会与此组织关联，并且您无法删除其存储目标。请将此关联视为永久性的。

    <Warning>
      在发送请求之前，请确认 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 已开启，并保持其开启状态。在其关闭期间发送的请求可能不会有对话记录，即使您之后将其开启也是如此。在另一个标签页中，于 [Compliance](https://platform.claude.com/settings/security) 页面上找到 **Compliance API** 开关。
    </Warning>
  </Step>

  <Step title="附加客户管理密钥">
    要使用您自己的密钥加密数据，请在 **加密密钥** 下选择该密钥。在您[创建并注册密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws)后，即可选择它。附加密钥是永久性的，因此在保存之前，请阅读[客户管理密钥的限制](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#what-you-commit-to)。

    密钥还会关闭某些功能。请参阅[使用密钥后会发生哪些变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#turned-off-or-changed)。
  </Step>

  <Step title="保存">
    保存时，Claude Console 会为该工作区关闭 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。使用 EFS 时，Anthropic Human Review 始终处于关闭状态。该更改生效后，[Anthropic 的一个小型专业团队](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#who-at-anthropic-can-access-your-data)将不再审查该工作区的新提示和响应。从那时起，审查 [Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases) 将由您的团队负责。
  </Step>
</Steps>

## 3. 检查您的设置

向该工作区发送不含敏感数据的测试请求。在[您选择的存储桶生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=aws#how-long-changes-take)后，新对象会出现在您在 **Sealed Safety Data storage** 下选择的存储桶的 `slab/` 或 `safeguards-data/slab/` 下。在出现第一个新对象后，请等待 10 分钟再发送真实流量。如果已有其他工作区向此存储桶发送数据，新对象并不能表明此更改已生效，因此请在发送真实流量之前咨询您的 Anthropic 客户团队。

在[工作区](https://platform.claude.com/settings/workspaces)页面上，选择该工作区，并检查 **Data retention** 下的 **Cases, transcripts** 是否显示您在 **Safety Case and Transcript storage** 下选择的存储桶。如果它缺失或显示的是其他存储桶，请在发送真实流量之前联系您的 Anthropic 客户团队。如果它显示 **Not copied to your storage**，请在该工作区的[数据保留](https://platform.claude.com/settings/privacy)页面上，于 **Safety Case and Transcript storage** 下选择您的存储桶，然后点击 **Save**。

## 检查谁在访问您的存储桶和密钥

要查看谁在访问您的 Amazon S3 存储桶和您的客户管理密钥，请检查您自己的日志。如果您发现其他调用方，请联系您的 Anthropic 客户团队。

* **Amazon S3 存储桶：** 在 `us-east-2` 中打开 AWS CloudTrail 的 **Event history**，即使您的存储桶位于其他区域也是如此。每个针对您的设置所创建的 IAM 角色的 `AssumeRole` 调用都应来自您的设置文件中指定的 Anthropic AWS 账户。Claude Console 的设置文件指定的账户为 `804820989669`。要同时记录对象的读取、写入和删除操作，请参阅[Anthropic 在 AWS CloudTrail 中的调用](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage?cloud=aws#cloudtrail)。
* **客户管理密钥：** 在[您的密钥生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)后，CloudTrail 会显示对您密钥的 `Encrypt` 或 `Decrypt` 调用，但并非每个请求都对应一次调用。来自您团队之外的每个调用都应显示 `userIdentity.invokedBy` 为 `aws-external-anthropic.amazonaws.com`，即您的密钥策略所允许的 AWS 服务主体。

## IAM 权限

您用于登录的 IAM 主体需要拥有设置存储的权限。`AnthropicFullAccess` 策略包含该权限。在您自己的策略中，请对 `Resource: "*"` 允许以下 `aws-external-anthropic` 操作：

* `RegisterAccountExternalStorage`：连接存储目标。
* `ActivateAccountExternalStorage`：运行 **Test setup**。
* `GetAccountExternalStorage`：查看存储目标。
* `ListAccountExternalStorages`：列出您的存储目标。
* `UpdateAccountExternalStorage`：更改存储目标的显示名称。
* `DeregisterAccountExternalStorage`：删除存储目标。这不会删除您的 S3 存储桶。
* `GetAccountPrograms`：查看 Anthropic 已为您的组织批准了哪些内容。
* `GetAccountWorkspacePolicy`：查看您的组织针对新工作区的策略。
* `UpdateAccountWorkspacePolicy`：更改您的组织针对新工作区的策略。
* `UpdateComplianceSettings`：更改 Compliance API 设置。

即使 [Claude Platform on AWS 的 IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions)未列出这些操作，也请允许它们。该页面列出了您在第 1 部分和第 2 部分中需要的其他操作：

* `AssumeConsole`
* `CreateWorkspace`
* `ListWorkspaces`
* `GetWorkspace`
* `UpdateWorkspace`
* `RegisterKey`
* `ListKeys`
* `GetKey`

您的 IAM 主体还需要两个 AWS STS 操作：`sts:GetWebIdentityToken` 和 `sts:TagGetWebIdentityToken`。`AnthropicFullAccess` 策略包含这两个操作。如果您使用自己的策略，请按照 [IAM 策略](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#iam-policies)中所示授予这两个操作。

要附加密钥，您还需要[该密钥的 KMS 权限](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws#prerequisites)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="如果您的密钥停止工作" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#if-your-key-stops-working">
    请求会发生什么，以及您该怎么做。
  </Card>

  <Card title="如果您的存储桶停止工作" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working">
    您的数据会发生什么，以及如何恢复正常运行。
  </Card>

  <Card title="更改或停止使用您的存储桶" icon="sliders" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket">
    迁移到新的存储桶，或让工作区停用 EFS。
  </Card>
</CardGroup>
