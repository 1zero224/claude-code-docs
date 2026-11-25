---
title: 在 Claude Platform 上设置 Enterprise Frontier Safeguards
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console
description: 为 Claude Platform 工作区开启 Enterprise Frontier Safeguards，并选择其存储和密钥。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

您需要在 Claude Console 中设置 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS）。EFS 适用于您开启它的[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)中发往每个模型的每个请求。要决定哪些工作区应使用它，请参阅[规划您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#plan-your-setup)。

设置分为五个部分：

1. 获取访问权限。
2. 添加云资源：您的存储桶和加密密钥。
3. 为工作区开启 EFS。
4. （可选）为新工作区设置策略。
5. 检查您的设置。

## 规划您的设置

请先规划您的设置，因为有些选择无法撤消。

首先，决定哪些[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)使用 Enterprise Frontier Safeguards。在 Claude Platform 上，EFS 涵盖[工作区中对每个模型的每个请求](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview#workspace-scope)。如需仅对部分流量使用 EFS，请使用单独的工作区。Default Workspace（默认工作区）无法使用 EFS。

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
    * **Google Cloud Storage：** 请参阅[存储桶及其项目的要求](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-before-you-begin)和[受支持的区域](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#supported-regions)。

    如果您的 AWS 组织使用资源控制策略（RCP）阻止外部账户，请尽早申请[为 Anthropic 的 AWS 账户设置例外](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#if-an-aws-organization-policy-blocks-anthropic)。

    Anthropic 不对 EFS 收费，但您的云提供商会向您收取存储、读取、写入和密钥操作费用，以及任何适用的网络传输费用。Anthropic 从其自身的区域读写您的存储桶，这可能会影响您的网络传输费用。
  </Accordion>

  <Accordion title="我的团队中需要哪些人员参与？">
    要设置 EFS 并在之后审查 Safety Case，您需要以下人员：

    | 人员 | 职责 |
    | ----- | ----- |
    | 云管理员 | 针对 [Amazon S3](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles) 或 [Google Cloud Storage](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-create-the-bucket)，运行来自 Claude Console 的设置文件，以创建存储桶和密钥，并向 Anthropic 授予访问权限。 |
    | 组织管理员 | 在 Claude Console 中连接存储桶、为每个工作区开启 EFS，并附加任何密钥。 |
    | 您的审查人员 | [审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。 |
  </Accordion>
</AccordionGroup>

## 开始之前

要设置 Enterprise Frontier Safeguards，您需要满足以下条件：

* 在 Claude Console 中拥有组织管理员角色。

## 1. 获取访问权限

要在 Claude Platform 上获得访问权限，需由组织管理员为您的组织接受条款。接受条款还使组织管理员能够关闭 Anthropic Human Review。如果您的 Anthropic 客户团队已为您的组织提供了访问权限，请跳过本节。

<Warning>
  接受条款会为您的组织以及关联到您 [Verification Portal](https://portal.anthropic.com) 配置文件的任何其他符合资格的组织开启 Compliance API。
</Warning>

### 获取访问权限会带来哪些变化

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

要接受条款，请前往 Claude Console 中的[计划](https://platform.claude.com/settings/programs)页面，并在 **Enterprise Frontier Safeguards** 卡片上点击 **Accept terms**。如果您在那里看不到 **Accept terms**，请登录 [Verification Portal](https://portal.anthropic.com)，选择您的组织，然后点击 **Accept terms**。

如果您无法登录 Verification Portal，或在那里也看不到 **Accept terms**，请向您的 Anthropic 客户团队申请访问权限。如果您没有客户团队，请填写 [Enterprise Frontier Safeguards 申请表](https://claude.com/form/enterprise-frontier-safeguards)。

如果您的组织已获得访问权限，[Programs](https://platform.claude.com/settings/programs) 页面会显示 **Enterprise Frontier Safeguards** 以及 **Configure** 按钮。

## 2. 添加云资源

要使用 Enterprise Frontier Safeguards，您需要自己的存储桶。您自己的加密密钥是可选的。如果没有，工作区将使用 Anthropic 管理的密钥。要添加您的存储桶、密钥，请在 Claude Console 中前往[数据保留](https://platform.claude.com/settings/privacy)页面，然后在页面顶部选择**组织**。

要使用您自己的密钥，请在您自己的密钥管理服务中创建[客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys)，然后在**加密密钥**下向 Anthropic 注册该密钥。

要为[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)设置您的存储桶，请按照适用于您的云提供商的步骤操作：

* **Amazon S3：** 按照[在 Amazon S3 上设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#amazon-s3)操作，直至完成[创建并连接您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)。
* **Google Cloud Storage：** 按照[在 Google Cloud Storage 上设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#google-cloud-storage)操作，直至完成[创建并测试存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-create-the-bucket)。

**Test setup** 通过后，工作区即可使用您的存储桶。

## 3. 为工作区开启 EFS

要为工作区开启 Enterprise Frontier Safeguards，请在 Claude Console 中前往[数据保留](https://platform.claude.com/settings/privacy)页面。首先为该工作区选择 **Enterprise Frontier Safeguards**，然后选择您的存储，再选择您自己的密钥（如果需要），然后保存。

使用 EFS 时，Anthropic Human Review 始终处于关闭状态，因此您的组织将负责审查 Anthropic 发送给您的所有 [Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。保存时，Claude Console 会为该工作区关闭 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。

请确定您团队中由谁来审查 Safety Case。要了解 Safety Case 如何到达您的团队，请参阅[获取 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#prepare)。

<Steps>
  <Step title="选择 Enterprise Frontier Safeguards">
    如果您的组织有 Legacy Zero Data Retention（ZDR）安排，您需要创建一个新工作区才能使用 EFS。Claude Console 会将处于 **Legacy ZDR** 状态的现有工作区的 **Enterprise Frontier Safeguards** 卡片标记为 **Ineligible**。要使用 EFS，请在创建新工作区时选择该卡片。在您完成[检查您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#check-your-setup)之前，请将您的流量保留在旧工作区上。

    要在创建工作区时选择该卡片，请先阅读本部分的其余内容。然后，在[工作区](https://platform.claude.com/settings/workspaces)页面上创建该工作区，并选择 **Enterprise Frontier Safeguards**、您的存储以及您的密钥（如果使用）。然后，前往[检查您的设置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#check-your-setup)。

    如果您在创建工作区时无法选择您的存储桶，请检查它是否已通过 **Test setup**。如果这无济于事，或者 Claude Console 未能完成工作区的创建，请联系您的 Anthropic 客户团队。

    要为已有的工作区选择该卡片，请在[数据保留](https://platform.claude.com/settings/privacy)页面顶部从**组织**切换到该工作区。您无法为 Default Workspace（默认工作区）选择 **Enterprise Frontier Safeguards**，因为它始终遵循您组织的数据保留设置。

    然后，选择 **Enterprise Frontier Safeguards** 卡片。
  </Step>

  <Step title="选择您的存储">
    <Warning>
      在[您选择的存储桶生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)之前，即使来自新工作区，并且即使您的组织有 Legacy ZDR 安排，密封的安全数据也可能会进入 Anthropic 管理的存储。在第一个新对象出现在您存储桶中的 `slab/` 或 `safeguards-data/slab/` 下之后满 10 分钟之前，请仅向此工作区发送不含敏感数据的测试请求。
    </Warning>

    在 **Sealed Safety Data storage** 下选择您的存储桶。没有存储桶，Claude Console 就无法创建使用 **Enterprise Frontier Safeguards** 的工作区。您也可以选择现在或稍后在 **Safety Case and Transcript storage** 下选择该存储桶。工作区有两项存储设置：

    * **Sealed Safety Data storage：** 设置 Anthropic 发送[密封的安全数据](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#what-is-stored)（即以 Anthropic 格式加密的提示、响应和模型推理）的位置。即使在您自己的存储桶中，您的团队也无法读取密封的安全数据。
    * **Safety Case and Transcript storage：** 设置 Anthropic 发送对话记录和 Safety Case 以供您的团队读取的位置。要在您的存储桶中接收它们，请参阅[您存储桶中的对话记录和 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#case-and-transcript-bucket)。

    如果您将 **Safety Case and Transcript storage** 保留为 **Not copied to your storage**，只有在您的组织拥有 [Safety Cases API 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#get-access)时，您的团队才能获取 Safety Case。

    一旦有工作区使用您的存储桶，该存储桶就会与此组织关联。请将这种关联视为永久性的。对于 Google Cloud Storage 存储桶，请在保存之前同时阅读[在工作区中使用您的 Cloud Storage 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)。

    <Warning>
      在发送请求之前，请确认 Compliance API 已开启，并保持其开启。在其关闭期间发送的请求没有对话记录，因此您的团队之后无法审查这些请求。要找到该设置，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#set-up-the-compliance-api)。
    </Warning>
  </Step>

  <Step title="附加客户管理密钥">
    要使用您自己的密钥加密数据，请在**加密密钥**下选择该密钥。在您[创建并注册密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key)后，即可选择它。附加密钥是永久性的，因此请在保存之前阅读[客户管理密钥的限制](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#what-you-commit-to)。

    对于已在处理流量的工作区，密钥在您保存后最多可能需要一天才能生效。

    密钥还会关闭某些功能。请参阅[使用密钥后会发生哪些变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#turned-off-or-changed)。
  </Step>
</Steps>

## 4. 为新工作区设置策略

您可以（可选）设置一项组织范围的策略，使新工作区使用 EFS 控制措施。通过策略，您可以为新工作区选定设置。您可以要求 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review) 处于开启或关闭状态（在 **Who reviews flagged cases?** 下），要求使用客户自有存储或客户管理密钥，或指定特定的存储桶或密钥。

请先[注册密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#add-cloud-resources)或[添加存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)，再设置相应要求，否则任何人都无法创建保留密封的安全数据的工作区。

要设置或更改该策略，请在 Claude Console 中[数据保留](https://platform.claude.com/settings/privacy)页面的顶部选择**组织**，然后找到 **Policy for new workspaces**。

该策略适用于新工作区，不会更改已存在的工作区。要更改现有工作区，请参阅[为工作区开启 EFS](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#turn-on-efs-for-a-workspace)。如果创建或更改工作区的请求违反该策略，Anthropic 会拒绝该请求。

## 5. 检查您的设置

向该工作区发送不含敏感数据的测试请求。在[您选择的存储桶生效](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take)后，新对象会出现在您在 **Sealed Safety Data storage** 下选择的存储桶中的 `slab/` 或 `safeguards-data/slab/` 下。在第一个新对象出现后，请等待 10 分钟再发送真实流量。如果已有其他工作区向此存储桶发送数据，新对象并不能表明此更改已生效，因此请在发送真实流量之前询问您的 Anthropic 客户团队。

## 检查谁在访问您的存储桶和密钥

要查看谁在访问您的 Amazon S3 存储桶和您的客户管理密钥，请检查您自己的日志。如果您发现其他调用方，请联系您的 Anthropic 客户团队。

* **Amazon S3 存储桶：** 在 `us-east-2` 中打开 AWS CloudTrail 的 **Event history**，即使您的存储桶位于其他区域也是如此。每个针对您的设置所创建的 IAM 角色的 `AssumeRole` 调用都应来自您的设置文件中指定的 Anthropic AWS 账户。Claude Console 的设置文件指定的账户为 `804820989669`。要同时记录对象的读取、写入和删除操作，请参阅[Anthropic 在 AWS CloudTrail 中的调用](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#cloudtrail)。
* **客户管理密钥：** 在[您的密钥生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)后，您密钥的审计日志会显示来自 Anthropic 的调用，但并非每个请求都有一条。您团队之外的每个调用都应来自[您密钥所用服务的设置页面](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#set-it-up)中指定的身份。在 AWS CloudTrail 中，该身份显示 `userIdentity.accountId` `915198916910`，以及以 `:anthropic-cmek` 结尾的 `userIdentity.principalId`。

## 后续步骤

<CardGroup cols={2}>
  <Card title="审查 Safety Case" icon="file" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases">
    接收、调查并响应 Anthropic 发送的 Safety Case。
  </Card>

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
