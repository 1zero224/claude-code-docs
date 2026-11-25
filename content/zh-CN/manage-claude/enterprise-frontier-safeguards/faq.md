---
title: Enterprise Frontier Safeguards 常见问题
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq
description: 关于 Enterprise Frontier Safeguards 常见问题的简短解答，并附有详细信息的链接。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

要设置 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS），请参阅[设置指南](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console)。

## 数据与保留

<AccordionGroup>
  <Accordion title="什么是 Enterprise Frontier Safeguards（EFS）？">
    [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS）是面向使用 [Claude 的能力最强的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)的符合条件组织的零数据保留（Zero Data Retention，ZDR）。Anthropic 不会保留您的提示和响应。密封的安全数据会写入您自有的存储。EFS 既包括[客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)，也包括由您组织中的人员[审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。使用 [Anthropic 管理的存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)的工作区不属于 EFS，因为 Anthropic 会存储该工作区提示和响应的加密副本。
  </Accordion>

  <Accordion title="如何获得 EFS 的访问权限？">
    1. 如果 Claude Console 中的[计划](https://platform.claude.com/settings/programs)页面已显示 **Enterprise Frontier Safeguards**，请直接跳至第 2 步。否则，请联系您的 Anthropic 客户团队申请访问权限。如果您没有客户团队，请填写 [Enterprise Frontier Safeguards 申请表](https://claude.com/form/enterprise-frontier-safeguards)。
    2. Anthropic 确认您的组织符合资格后，由组织管理员在 Claude Console 的[计划](https://platform.claude.com/settings/programs)页面接受相关条款。
    3. 要设置 EFS，请按照[适用于您平台的 EFS 设置指南](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console)进行操作。

    在 Amazon Bedrock 上，您通过 AWS 获取访问权限。请参阅 [Amazon Bedrock 上的 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-amazon-bedrock)。
  </Accordion>

  <Accordion title="Enterprise Data Controls 和 EFS 有什么区别？">
    主要区别在于您的密封的安全数据存储在何处。使用 EFS 时，这些数据会写入您自有的存储，因此 EFS 属于零数据保留（ZDR）。使用 [Enterprise Data Controls](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview#enterprise-data-controls) 时，工作区仍使用 Anthropic 管理的存储。在 Claude Platform 上，您仍然可以[使用客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys)并[关闭 Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)，以进一步控制您的数据。
  </Accordion>

  <Accordion title="EFS 是否仅适用于需要数据保留的模型？">
    不是。在 Claude Platform 上，EFS 适用于您开启它的工作区中对每个模型的每个请求，其中包括不需要数据保留的模型。同一个工作区不能对某些模型使用 EFS，而对其他模型不使用。要仅对部分流量使用 EFS，请[使用不同的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview#workspace-scope)，并通过 API 密钥或工作区 ID 选择工作区。
  </Accordion>

  <Accordion title="我是否必须为整个组织开启 EFS？">
    不需要。在 Claude Platform 上，获得 EFS 访问权限不会为任何工作区开启 EFS。为某个工作区选择 EFS 也不会改变您的其他工作区。访问透明度是一个例外：一旦开启，它将对您的整个组织生效，并且您无法将其关闭。请参阅[检查访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)。如果您的组织有 Legacy ZDR 安排，使用 **Legacy ZDR** 的工作区会保持不变。要使用 EFS，请创建一个新工作区，并用它代替您的 Legacy ZDR 工作区。

    从使用 **Legacy ZDR** 的工作区向[需要 30 天数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)的模型发送的请求会失败，并返回 400 错误，除非 Anthropic 已授权您的组织在 Legacy ZDR 下使用该模型。请改从使用 EFS 或 Anthropic 管理的存储的工作区发送这些请求。
  </Accordion>

  <Accordion title="EFS 涵盖哪些数据？">
    EFS 涵盖在为 EFS 配置的工作区中对 Messages API 发出的请求，无论使用哪个模型。它不涵盖 Claude Platform 功能为其自身用途而存储的数据，即使这些数据处于某个 Messages API 请求之中也是如此，例如代码执行容器、Files API 上传的文件、Batch API 结果以及 Claude Managed Agents 会话。关于每项功能，请参阅 [API 与数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)。

    EFS 从配置生效之时起适用。在此之前写入的数据会保留在其原写入位置，直至过期。
  </Accordion>

  <Accordion title="EFS 是否支持 Batch API？">
    不支持，因为 [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 不会保留供您的团队审查的对话记录。请改为通过 Messages API 单独发送每个请求，或者从不属于 EFS 且开启了 Anthropic Human Review 的工作区发送批处理。
  </Accordion>

  <Accordion title="EFS 是否支持 Anthropic 的 Claude Managed Agents 服务？">
    不支持。每个使用 EFS 的工作区都关闭了 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。Anthropic 的 Claude Managed Agents 服务不会在关闭了 Anthropic Human Review 的工作区中运行会话或[梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)。此类工作区中已存在的会话将停止接受新消息。要运行会话或梦境，请在开启了 Anthropic Human Review 的工作区中创建它们。
  </Accordion>

  <Accordion title="EFS 是否适用于设置之前的流量？">
    不适用。EFS 在您的设置生效后开始适用。请参阅[更改需要多长时间](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#how-long-changes-take)。
  </Accordion>

  <Accordion title="Anthropic 会使用我的数据进行训练吗？">
    Anthropic 不会在未获得明确许可的情况下使用企业数据训练 Claude 模型。
  </Accordion>
</AccordionGroup>

## 访问与审查

<AccordionGroup>
  <Accordion title="Anthropic 的哪些人会审查我的数据？">
    自动化安全监控会检查您的流量，在这一步中没有任何人会看到您的内容。使用 EFS 时，Anthropic Human Review 始终处于关闭状态。在 Anthropic 管理的存储上，当 Anthropic Human Review 开启时，Anthropic 的一个小型专业团队可以审查自动化安全监控发现的内容。请参阅[自动化安全监控何时访问您的数据](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#who-at-anthropic-can-access-your-data)。
  </Accordion>

  <Accordion title="我如何知道 Anthropic 人员是否访问了我的数据？">
    对于[访问透明度涵盖的所有内容](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)，Access Transparency（访问透明度）会记录 Anthropic 人员对您数据的访问。在 Claude Platform 上，它没有关于客户自有存储中数据的记录。请参阅[检查访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)。
  </Accordion>

  <Accordion title="我可以添加自己的分类器吗？">
    不可以。由 Anthropic 的自动化安全监控检查您的流量。
  </Accordion>
</AccordionGroup>

## 存储与密钥

<AccordionGroup>
  <Accordion title="客户自有存储支持哪些云提供商？">
    您的存储桶的提供商取决于您在何处使用 Claude：

    * **Claude Platform：** Amazon S3 或 Google Cloud Storage。
    * **[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)：** Amazon S3。
    * **[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)：** Google Cloud Storage。

    请参阅[设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)或[在 Google Cloud Storage 上设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#google-cloud-storage)。
  </Accordion>

  <Accordion title="我可以更改存储桶中的内容吗？">
    仅可用于写入您团队的[回复文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#response-file)，前提是您的存储桶接收 [Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)。请勿向您的存储桶写入任何其他内容，并保持其其余内容不变。对于 Amazon S3 存储桶，请参阅[向您的团队授予存储桶访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#let-your-team-read-the-bucket)和[哪些操作会从您的存储桶中移除数据](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#what-removes-data)。
  </Accordion>

  <Accordion title="多个 Anthropic 组织可以共享一个 AWS 账户吗？">
    可以。请参阅[创建并连接您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)。

    每个组织都需要自己的存储桶，并配有[自己的 KMS 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#if-you-use-your-own-bucket)，并且使用某个存储桶会[将其与一个组织绑定](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#before-you-begin)。
  </Accordion>

  <Accordion title="我可以在同一个 AWS 账户中连接多个存储桶吗？">
    可以。每个存储桶都需要自己的 CloudFormation 堆栈或自己的 Terraform 文件夹。请参阅[创建并连接您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#create-the-bucket-and-roles)。
  </Accordion>

  <Accordion title="每个工作区可以使用自己的存储桶吗？">
    在 Claude Platform 和 Claude Platform on AWS 上，可以。请在 Claude Console 中为每个工作区选择其存储桶。对于 Google Cloud Storage，请先与您的 Anthropic 客户团队沟通，因为 [Anthropic 会限制存储桶和工作区的数量](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)。
  </Accordion>

  <Accordion title="我可以将对话记录和 Safety Case 保存在与密封的安全数据不同的存储桶中吗？">
    请计划为两者使用同一个存储桶。如果一个工作区同时使用 [**Sealed Safety Data storage** 和 **Safety Case and Transcript storage**](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#the-two-storage-settings)，请为两者选择同一个存储桶。如果您选择了不同的存储桶，即使保存成功，该存储桶也可能收不到对话记录和 Safety Case。请参阅[您存储桶中的对话记录和 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#case-and-transcript-bucket)。
  </Accordion>

  <Accordion title="如果我停止使用客户自有存储，我的数据会怎样？">
    EFS 需要客户自有存储，因此停止使用它的工作区将退出 EFS。关于您的数据会怎样以及步骤的顺序，请参阅[更改或停止使用您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket)。

    在您[停止使用某个存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#change-or-stop-using-your-bucket)之后，如果可以，请[删除其存储目标](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#delete-a-storage-destination)。如果无法删除，请在移除 Anthropic 对该存储桶的访问权限之前，先与您的 Anthropic 客户团队沟通。即使其中的所有内容都已超过保留期，这一点也同样适用。
  </Accordion>

  <Accordion title="附加客户管理密钥后会有哪些变化？">
    [附加密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key)会关闭部分功能，并且是永久性的。首先，请查看[关闭了哪些功能](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#turned-off-or-changed)，并阅读[附加密钥之前](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#before-you-attach-a-key)。
  </Accordion>
</AccordionGroup>

## Claude Platform on AWS 上的 EFS

要在 Claude Platform on AWS 上设置 EFS，请按照[在 Claude Platform on AWS 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-platform-on-aws) 进行操作。客户管理的加密密钥已通过 AWS KMS 在该平台上可用。

## Claude Models on Google Cloud (Anthropic managed) 上的 EFS

要在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上设置 EFS，请按照[在 Claude Models on Google Cloud (Anthropic managed) 上设置 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-claude-models-on-google-cloud-anthropic-managed) 进行操作。您的存储桶位于 Google Cloud Storage 中，您的客户管理的加密密钥位于 Google Cloud KMS 中。关于密钥，请参阅[使用 Google Cloud KMS 设置 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms)。

## 如果您的密钥或存储桶停止工作

在 Claude Platform 上，如果您的密钥在[生效之后](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)被撤销或禁用，请求可能会在最多一小时内继续成功。此后，对该工作区的请求将会失败。关于如何恢复正常运行，请参阅[如果您的密钥停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#if-your-key-stops-working)。

如果 Anthropic 无法写入您的存储桶，请参阅[如果您的存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#if-your-bucket-stops-working)。对于 Google Cloud Storage 存储桶，请参阅[如果您的 Cloud Storage 存储桶停止工作](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-if-your-bucket-stops-working)。

## 更改需要多长时间

请根据以下时间安排您的检查和向生产环境的迁移：

| 更改 | 所需时间 |
| ----- | ----- |
| 新工作区或使用 Anthropic 管理的存储的工作区[开始使用客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#how-long-changes-take) | 通常在 10 分钟内，但也可能需要更长时间。第一个新对象到达您的存储桶后，请再等待 10 分钟。 |
| [访问透明度开始记录](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)（在其开启之后） | 2 小时内 |
| 访问透明度事件到达您的活动源 | 在其所记录的访问发生后 2 个工作日内 |
| [客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys#attach-a-customer-managed-key)在已处理流量的工作区上生效 | 最多一天 |
| 对 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review) 的更改生效 | 最多 2 小时 |

## 术语表

| 术语 | 含义 |
| ----- | ----- |
| Claude Platform | 在 Claude Console 中设置的 Claude API。 |
| 自动化安全监控 | 在大量请求中查找滥用模式的自动化系统。使用 EFS 时，这些系统会分析存储在您云账户中的数据，并由您的团队审查任何发现。 |
| 密封的安全数据 | 以 Anthropic 格式加密的提示、响应和模型推理。即使在您自己的存储桶中，您的团队也无法读取这些数据。请参阅[存储的内容及位置](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#what-is-stored)。 |
| 对话记录 | 提示和响应的可读副本，供您的团队阅读。 |
| Safety Correspondence | Safety Cases、Safety Notices 以及您的 Safety Case Responses。绝不包含提示和响应。 |
| 安全数据存储桶 | 位于您自己云账户中、工作区用于存放密封的安全数据的存储桶。您在该工作区的[数据保留](https://platform.claude.com/settings/privacy)页面上的 **Sealed Safety Data storage** 下选择它。EFS 需要一个这样的存储桶。 |
| Case 存储桶 | Anthropic 以您的团队可以读取的形式发送工作区的对话记录和 Safety Case 的存储桶。您在 [**Safety Case and Transcript storage**](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#the-two-storage-settings) 下选择它。在 Claude Models on Google Cloud (Anthropic managed) 和 Claude Platform on AWS 上，EFS 需要一个这样的存储桶。同一个存储桶可以同时作为安全数据存储桶和 Case 存储桶，因此请计划为两者使用同一个存储桶。 |
| Legacy ZDR | 具有实时安全防护但没有自动化安全监控的 [ZDR](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)。在返回 API 响应之后，Anthropic 不会存储提示或响应，除非为遵守法律或打击滥用行为而需要。 |
| Anthropic 管理的存储 | 由 Anthropic 运行的存储，供使用 **Anthropic managed** 卡片的工作区使用。 |
| Anthropic managed | Claude Console 中适用于使用 Anthropic 管理的存储的工作区的一张卡片。它是[需要 30 天数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)的 Claude 模型的默认设置。 |
| Enterprise Data Controls | 一组功能，适用于符合 EFS 条件、但使用 Anthropic 管理的存储而非 EFS 的工作区。您可以使用客户管理密钥（CMEK），并在 Claude Platform 上关闭 Anthropic Human Review。Anthropic 仍会存储您的密封的安全数据，因此这种设置不属于 EFS，也不属于 ZDR。请参阅 [Enterprise Data Controls](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview#enterprise-data-controls)。 |
| 客户自有存储 | 将您的密封的安全数据存储在您自己云账户中的存储桶里。请参阅[设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)或[在 Google Cloud Storage 上设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#google-cloud-storage)。 |
| 客户管理密钥 | 位于您自己的密钥管理服务中、用于加密您数据的加密密钥，也称为 CMEK。请参阅[使用客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys)。 |
| Human Review | 也称为 Anthropic Human Review。在使用 Anthropic 管理的存储的 Claude Platform 上，如果 Claude Console 显示问题 **Who reviews flagged cases?**，选择 **Anthropic** 会开启它，选择 **My organization** 会关闭它。开启时，Anthropic 的一个小型专业团队可以审查自动化安全监控发现的内容。使用 EFS 时，它始终处于关闭状态。请参阅[可以在哪里关闭 Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。 |
| 访问透明度 | 通过 Compliance API 提供的 Anthropic 人员访问您数据的记录，涵盖[访问透明度涵盖的所有内容](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)。 |
| Safety Case | 当自动化安全监控在未开启 Anthropic Human Review 的工作区中发现可能违反 Anthropic 使用政策的情况时，Anthropic 可能会开立的一条记录。它按标识符列出相关的对话记录。[审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases) 说明了哪些工作区会收到 Safety Case。 |
| 存储目标 | 您在 Claude Console 的[数据保留](https://platform.claude.com/settings/privacy)页面上 **Storage destinations** 下连接的存储桶。您可以在 **Sealed Safety Data storage** 或 **Safety Case and Transcript storage** 下为工作区选择一个存储目标。 |
| Anthropic 管理的密钥 | 当您未附加自己的密钥时，Anthropic 用于加密您数据的密钥。 |

## 参考

关于 API 详细信息，请参阅 [Admin API 中的客户管理密钥](https://platform.claude.com/docs/zh-CN/api/organization/external_keys)、[Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 和[访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency)。
