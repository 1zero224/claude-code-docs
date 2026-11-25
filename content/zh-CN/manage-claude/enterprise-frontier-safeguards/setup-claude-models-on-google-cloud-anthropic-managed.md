---
title: 在 Claude Models on Google Cloud (Anthropic managed) 上设置 Enterprise Frontier Safeguards
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed
description: 在 Claude Models on Google Cloud (Anthropic managed) 上为工作区开启 Enterprise Frontier Safeguards，并选择其存储和密钥。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

要设置 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS），请从 Google Cloud 控制台打开 Claude Console。EFS 适用于已开启它的工作区中对每个模型的每个请求。要决定哪些工作区应使用它，请参阅[规划您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#plan-your-setup)。

设置包含三个部分：

1. 添加云资源：您的存储桶和加密密钥。
2. 为工作区开启 EFS。
3. 检查您的设置。

## 规划您的设置

请先规划您的设置，因为有些选择无法撤消。

首先，决定哪些[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)使用 Enterprise Frontier Safeguards。如需仅对部分流量使用 EFS，请使用单独的工作区。Default Workspace（默认工作区）无法使用 EFS。

如果您的组织有 Legacy Zero Data Retention（ZDR）安排，请为 EFS 使用一个新工作区，并在创建该工作区时选择 **Enterprise Frontier Safeguards** 卡片。在 Claude Console 中，对于使用 **Legacy ZDR** 的现有工作区，您无法选择该卡片。

最后，请回答以下问题：

<AccordionGroup>
  <Accordion title="我的存储桶应该放在哪里？费用是多少？">
    在开始之前，请选择存储桶所在的云账户和区域：

    * **Google Cloud Storage：** 请参阅[存储桶及其项目的要求](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-before-you-begin)和[受支持的区域](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#supported-regions)。

    Anthropic 不对 EFS 收费，但您的云提供商会向您收取存储、读取、写入和密钥操作费用，以及任何适用的网络传输费用。Anthropic 从其自身的区域读写您的存储桶，这可能会影响您的网络传输费用。
  </Accordion>

  <Accordion title="我的团队中需要哪些人员参与？">
    要设置 EFS 并在之后审查 Safety Case，您需要以下人员：

    | 人员 | 职责 |
    | ----- | ----- |
    | 云管理员 | 针对 [Google Cloud Storage](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-create-the-bucket)，运行来自 Claude Console 的设置文件，以创建存储桶和密钥，并向 Anthropic 授予访问权限。 |
    | 组织管理员 | 在 Claude Console 中连接存储桶、为每个工作区开启 EFS，并附加任何密钥。 |
    | 您的审查人员 | [审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。 |
  </Accordion>
</AccordionGroup>

## 开始之前

要设置 Enterprise Frontier Safeguards，您需要以下内容：

* 在其组织将使用 EFS 的 Google Cloud 项目上拥有 [`roles/claude.admin`](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console)。
* 您的组织已获得 [EFS 访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#what-getting-access-changes)。获得访问权限可能会更改一些您无法改回的设置。
* 要使用您自己的密钥：您的组织已[开启客户管理的密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#availability)。

要打开 Claude Console，请前往 Google Cloud 控制台中的 [Claude Platform](https://console.cloud.google.com/agent-platform/claude-platform) 并选择您的项目。然后，选择 **Anthropic managed** 并点击 **Open Claude Console**。请在点击本页面指向 Claude Console 的链接之前执行此操作。每个链接都会在您浏览器当前登录的 Anthropic 组织中打开。要检查您所在的组织，请前往 **设置** > **组织**。**Google Cloud project number** 必须显示您项目的编号，而不是其 ID。如果页面上没有这一行，您很可能位于其他组织中。

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

要使用 Enterprise Frontier Safeguards，您需要自己的存储桶。您自己的加密密钥是可选的。如果没有，工作区将使用 Anthropic 管理的密钥。要添加您的存储桶、密钥，请前往 Claude Console 中的 [数据保留](https://platform.claude.com/settings/privacy) 页面，并在页面顶部选择 **组织**。

要使用您自己的密钥，请在 Google Cloud KMS 中创建一个 [customer-managed key（客户管理的密钥）](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms)，然后在 **Encryption keys** 下向 Anthropic 注册该密钥。请按照 **Claude Platform** 选项卡操作，并在 **将密钥附加到工作区** 之前停止。请使用与您存储桶自身密钥不同的密钥，存储桶自身的密钥是您在创建存储桶时一并创建的。

要为客户自有存储设置您的存储桶，请使用一个仅供此组织使用的新存储桶。如果此组织已有存储桶，您可以使用该存储桶。按照[在 Google Cloud Storage 上设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=gcp#google-cloud-storage)的步骤操作，直至完成[创建并测试存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=gcp#gcs-create-the-bucket)的全部内容。

Cloud Audit Logs 不会记录您在数据保留页面上所做的某些更改。这些更改是指对存储目标、新工作区策略，以及工作区的存储、密钥或 Anthropic Human Review 所做的更改。

## 2. 为工作区开启 EFS

要开启 Enterprise Frontier Safeguards，请创建一个工作区。然后，选择 **Enterprise Frontier Safeguards** 卡片和您的存储，如果您使用自己的密钥则添加该密钥，并保存一次。

<Steps>
  <Step title="创建工作区">
    在 [工作区](https://platform.claude.com/settings/workspaces) 页面上为 EFS 创建一个新的工作区，即使您已有工作区也是如此。Default Workspace 无法使用 EFS。

    * **如果您可以选择 Enterprise Frontier Safeguards 卡片：** 您可以在创建工作区时开启 EFS。请先阅读本部分的其余内容。然后选择该卡片、在两个存储设置下选择您的存储桶，以及您的密钥（如果您使用密钥）。创建工作区，并前往[检查您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#check-your-setup)。
    * **否则，如果您的组织有 Legacy Zero Data Retention（ZDR）安排：** 请不要创建该工作区。检查您的存储桶是否已通过 **Test setup**。如果已通过，请联系您的 Anthropic 客户团队。
    * **否则：** 创建工作区，然后继续下一步。

    在到达[检查您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed#check-your-setup)之前，请不要向该工作区发送任何请求。
  </Step>

  <Step title="选择 Enterprise Frontier Safeguards">
    前往 Claude Console 中的 [数据保留](https://platform.claude.com/settings/privacy) 页面。在页面顶部，从 **组织** 切换到您为 EFS 创建的工作区。然后，选择 **Enterprise Frontier Safeguards** 卡片。在完成其余步骤期间请停留在此页面，并在最后一步之前不要保存。

    如果您无法选择该卡片，请检查您的存储桶是否已通过 **Test setup**。如果已通过，请联系您的 Anthropic 客户团队。
  </Step>

  <Step title="选择您的存储">
    在 **Sealed Safety Data storage** 下选择您的存储桶。然后，在 **Safety Case and Transcript storage** 下选择同一个存储桶。如果您未在 **Safety Case and Transcript storage** 下选择存储桶，您的保存将失败。

    <Warning>
      一旦某个工作区使用了您的存储桶，该存储桶就会绑定到此组织，并且您无法删除其存储目标。请将这种绑定视为永久性的。
    </Warning>

    在保存之前，请阅读[在工作区中使用您的 Cloud Storage 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=gcp#gcs-use-your-bucket-in-a-workspace)。

    如需迁移到其他存储桶，请先咨询您的 Anthropic 客户团队。

    如果 Claude Console 未显示 **Safety Case and Transcript storage**，请在保存之前联系您的 Anthropic 客户团队。如果它拒绝您的存储桶，请在连接其他存储桶之前联系他们。在您于 **Safety Case and Transcript storage** 下保存存储桶后，只要该工作区使用 EFS，您就无法清除这项设置。

    <Warning>
      在发送请求之前，请确认 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 已开启，并保持其开启状态。在其关闭期间发送的请求可能不会有对话记录，即使您之后将其开启也是如此。请在另一个标签页中，于 [Compliance](https://platform.claude.com/settings/security) 页面上找到 **Compliance API** 开关。
    </Warning>
  </Step>

  <Step title="附加客户管理的密钥">
    要使用您自己的密钥加密数据，请在 **Encryption key** 下选择该密钥。附加密钥是永久性的，因此请在保存或创建工作区之前阅读[客户管理的密钥的限制](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#what-you-commit-to)。
  </Step>

  <Step title="保存">
    当您保存时，或在您使用您的存储桶创建工作区时，Claude Console 会为该工作区关闭 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。使用 EFS 时，Anthropic Human Review 始终处于关闭状态。此更改生效后，[Anthropic 的一个小型专业团队](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#who-at-anthropic-can-access-your-data)将不再审查该工作区的新提示和响应。从那时起，审查 [Safety Cases](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases) 便是您团队的工作。请[监控您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#set-up-alerts)，因为 7 天的响应期限从 Anthropic 发送 Safety Case 时开始计算。
  </Step>
</Steps>

## 3. 检查您的设置

<Warning>
  在[您的存储桶选择生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=gcp#confirm-that-objects-arrive)之前，密封的安全数据可能会进入 Anthropic 管理的存储，即使您的组织有 Legacy Zero Data Retention（ZDR）安排也是如此。在此之前，请仅向此工作区发送不含敏感数据的测试请求。
</Warning>

向该工作区发送测试请求。在这些请求中，以及之后 EFS 应覆盖的每个请求中，都使用[新工作区的 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#find-your-workspace-id)。然后[确认对象已到达](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=gcp#confirm-that-objects-arrive)您的存储桶。这些对象是密封的安全数据，并不表明对话记录或 Safety Cases 已到达。

在第一个新对象到达存储桶中您组织文件夹的 `safeguards-data/slab/` 下满 10 分钟之前，请只发送测试请求。在此之前，请将您的流量保留在现有工作区上。如果另一个工作区已在向此存储桶发送数据，新对象并不能表明此更改已生效。在这种情况下，请在发送真实流量之前咨询您的 Anthropic 客户团队。

在 [工作区](https://platform.claude.com/settings/workspaces) 页面上选择该工作区，并检查 **Data retention** 下的 **Cases, transcripts** 是否显示您的存储桶。如果该行缺失或显示其他内容，请在发送真实流量之前联系您的 Anthropic 客户团队。

## 后续步骤

<CardGroup cols={2}>
  <Card title="如果您的密钥停止工作" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#if-your-key-stops-working">
    请求会发生什么，以及应该如何处理。
  </Card>

  <Card title="如果您的存储桶停止工作" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=gcp#gcs-if-your-bucket-stops-working">
    您的数据会发生什么，以及如何恢复您的存储桶。
  </Card>

  <Card title="停止使用您的存储桶" icon="sliders" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage?cloud=gcp#gcs-stop-using-your-bucket">
    在停止使用您的存储桶时保留您的数据。
  </Card>
</CardGroup>
