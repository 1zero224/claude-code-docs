---
title: 审查 Safety Case
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases
description: 您的团队如何接收、调查和响应 Anthropic 发送的 Safety Case，以及在何处获取帮助。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

自动化安全监控会检查您的提示和响应中是否存在严重滥用的迹象，例如试图开发进攻性网络或生物能力。Enterprise Frontier Safeguards 要求关闭 [Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。关闭后，自动化安全监控发现的可能违反 Anthropic 使用政策的行为可能会成为一个 Safety Case，交由您自己的团队审查。Anthropic 预计对于大多数组织而言，Safety Case 都很少见。

## Safety Case 审查的工作方式

1. **[Anthropic 向您发送 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#where-cases-arrive)。**
2. **[您审查它](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#read-a-case)**，查看它涵盖哪些消息以及报告的是哪类风险。
3. **[您进行调查](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#investigate)**，借助您自己的日志和对话记录。
4. **[您进行响应](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#respond-to-a-case)**，以确认您已完成调查。
5. **[Anthropic 关闭 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#after-you-respond)。**

## 获取 Safety Case

Safety Case 可以通过两种方式送达您：作为您自己存储中的文件，以及通过 API。

在 Claude Platform 上，请保持 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#set-up-the-compliance-api) 开启。在其关闭期间发送的请求在您的 [Case 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)中或通过 Safety Cases API 都没有对话记录。如果没有 Case 存储桶，在其关闭期间您也不会收到任何 Safety Case。

### 在您自己的存储中

Anthropic 会将可读的 Safety Case 和对话记录写入[您自己云账户中的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage#what-is-stored)。要选择存储桶：

1. 在选择存储桶之前，请阅读[您存储桶中的对话记录和 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#case-and-transcript-bucket)。并非每个 Amazon S3 存储桶都能接收 Safety Case。
2. 在 Claude Console 中为每个工作区在 **Safety Case and Transcript storage** 下选择该存储桶。

当 Anthropic 向您发送 Safety Case 时，它会将 [Safety Case 文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files)添加到您的存储中。

有两项限制：

* 只有拥有[安全数据存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)的工作区才能获取 Safety Case 文件（请参阅[设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)）。该存储桶存放您的团队无法读取的密封的安全数据。您可以在 **Sealed Safety Data storage** 下选择它，这是与 **Safety Case and Transcript storage** 分开的设置。
* 在 Claude Platform 上，您也只能针对工作区开始使用其安全数据存储桶之后发送的消息获取 Safety Case 文件。

### 通过 API

[Safety Cases API](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api) 通过 HTTP 提供相同的 Safety Case。该 API 可让您读取和响应 Safety Case。它仅在 Claude Platform 上可用。它处于测试阶段。

您是否还需要 Case 存储桶取决于您的平台：

* **Claude Platform：** 您还可以在 **Safety Case and Transcript storage** 下为每个工作区选择一个 Case 存储桶。如果没有 Case 存储桶，对于带有用户配置文件的消息，您将不会收到任何 Safety Case。
* **Claude Models on Google Cloud (Anthropic managed) 和 Claude Platform on AWS：** Safety Case 只能通过 Case 存储桶送达您的团队。请对该存储桶进行监控，因为 7 天的响应期从 Anthropic 发送 Safety Case 时开始计算。

### 如何得知 Safety Case 已创建

Anthropic 创建 Safety Case 时不会自动通知您。监控您的存储或[轮询 API](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#poll) 以获取 Safety Case，以便您能够快速响应。请参阅[新文件警报](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#alerts)，了解基于存储更新向您的 SIEM 或工单系统发送警报的方式。

## 审查 Safety Case

### Safety Case 包含的内容

每个 Safety Case 包含：

* 其 ID
* 政策类别，例如 `cyber` 或 `bio`
* 每条消息的 ID，以及在何处找到其对话记录
* 发送这些消息的工作区，有时还包括发送它们的用户配置文件

在您的存储桶中，Safety Case 是一个 JSON 文件。[Safety Case 文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#safety-case-file)描述了其字段。

通过 API，[列出您的 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#poll)，然后[读取每条消息](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#read-a-message)。API 还会返回 Anthropic 为消息记录的用户或 API 密钥。

### 调查 Safety Case

大多数 Safety Case 最终会归结为以下四种结果之一：

| 您的发现 | 应采取的措施 |
| ----- | ----- |
| **经授权的工作。** 例如，您的红队在 Cyber Verification Program 下进行了渗透测试。 | 与负责该工作的团队确认。以 `no_violation` 响应。 |
| **员工或用户擅自行动，违反政策。** 他们在未经您组织批准的情况下违反了您的政策或 Anthropic 的使用政策。 | 撤销其访问权限并遵循您自己的流程。以 `violation` 响应。 |
| **外部用户或被盗凭据。** | 封禁该用户或轮换密钥。以 `violation` 响应。 |
| **误报。** 自动化安全监控误读了意图或内容。 | 无需修复。以 `no_violation` 响应。 |

由谁参与（包括人力资源、法务或经理）取决于您自己的流程。

<Note>
  将这些对话记录输入 Claude 或其他 AI 模型时请务必小心。它们可能会导致另一个 Safety Case。
</Note>

## 响应 Safety Case

请在 Safety Case 的 `created_at`（即 Anthropic 发送它的时间）之后的 7 个日历日内响应。通过响应，您确认您已进行调查并查明原因，并且如果存在违规，已采取措施确保其不再发生。

您的 Safety Case Response 说明您是否发现了违规。您无需与 Anthropic 分享姓名、证据或对话记录。您在响应中选择与 Anthropic 分享的任何信息都会成为 Safety Case 的一部分。响应会关闭 Safety Case，因此您之后无法更改或补充该响应。

您可以通过以下两种方式之一进行响应。每个 Safety Case 请只使用其中一种，因为 Anthropic 以第一个响应为准。

### 选项 1：写入响应文件

将[响应文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#response-file)写入存放您 Safety Case 的存储桶。其路径取决于您的存储提供商：

* **Amazon S3：** `review-responses/<case_id>.json`
* **在 Claude Console 中连接的 Google Cloud Storage 存储桶：** `<organization ID>/review-responses/<case_id>.json`

请对照[有效文件的规则](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#response-file-rules)检查该文件，因为只有有效的文件才算数。

### 选项 2：通过 API 响应

发送一个请求来[响应 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#respond)。

### 响应之后

当 Anthropic 读取到有效响应时，它会关闭 Safety Case。关闭原因为 `responded` 表示 Anthropic 已收到您的响应。通过 API 响应时，Safety Case 会立即关闭。如果在您写入文件一天后 Safety Case 仍处于开启状态，请参阅[Safety Case 为何保持开启状态](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#still-open)。

如果 Anthropic 需要您就您的响应提供更多信息，它会发送 Safety Notice（请参阅[轮询 Safety Notice](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#poll-for-notices)），或者您的 Anthropic 客户团队会与您联系。

## 如果 Anthropic 采取措施

Enterprise Frontier Safeguards 不会改变 Anthropic 的[使用政策](https://www.anthropic.com/legal/aup)或其执行标准。Anthropic 仅针对明确的、故意的危害采取执行措施。

### 如果您不响应

及时响应有助于保持您账户的良好状态。如果 7 个日历日过去仍没有有效响应，Anthropic 会关闭 Safety Case。如果 Safety Case 一直未得到答复，Anthropic 可能会暂停访问。

### 如果 Anthropic 暂停访问

在极少数情况下，如果自动化安全监控发现明确的、故意的危害，Anthropic 可以在您调查期间暂停访问以限制进一步的危害。暂停范围可以小到单个用户或用户配置文件。如果您发送了用户配置文件，Safety Case 可以指明该用户配置文件。要申请恢复访问，请联系您的 Anthropic 客户团队。如果 Safety Case 仍处于开启状态，也请对其进行响应。在您的整个组织被暂停期间，Safety Cases API 会拒绝您的密钥。

## 最佳实践

* **记录消息 ID。** 为了将消息追溯到其发送者，Anthropic 建议将每个 Messages API 响应的 `id`（`msg_...`）与您自己的用户 ID 一起记录下来。通常应尽早开始这样做，因为您的日志只涵盖从那时起的请求。您还可以将 `metadata.user_id` 设置为一个不透明 ID，该 ID 会在对话记录中返回。
* **保存您需要的对话记录。** 对话记录会过期（请参阅[存储桶中的内容](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#whats-in-the-bucket)）。

## 寻求帮助

您的 Anthropic 客户团队是您处理 Safety Case 的联系人。请引用 Safety Case 的 ID，以便 Anthropic 找到正确的 Safety Case。您无需引用或提供对话记录。

在没有 Anthropic Human Review 的情况下，Anthropic 中处理 Safety Case 的人员只能看到有限的详细信息，例如其 ID、类别以及任何拟议的措施。他们无法读取您的对话记录。

* **您需要更多时间：** 请在 7 天结束之前告知您的 Anthropic 客户团队。Safety Case 仍会按时关闭。
* **您认为 Safety Case 有误：** 以 `no_violation` 响应。

## 问题与故障排除

<AccordionGroup>
  <Accordion title="您的团队是否需要 24/7 值守？">
    Anthropic 不要求为 Safety Case 提供 24/7 值守。Anthropic 预计 Safety Case 很少见，并且您有数天而非数小时的时间来响应。请将警报发送到每个工作日都有人查看的队列。
  </Accordion>

  <Accordion title="能否在真实的 Safety Case 到来之前测试流程？">
    请询问您的 Anthropic 客户团队，您的组织是否可以使用测试 Safety Case。
  </Accordion>

  <Accordion title="Anthropic 是否会为 Safety Case 发送 webhook？">
    不会。请改为[监控您的存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#set-up-alerts)或[轮询 Safety Cases API](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#poll)。
  </Accordion>

  <Accordion title="Anthropic Human Review 开启时由谁审查？">
    Anthropic 的一个小型专业团队可以审查自动化安全监控发现的内容。您只能为使用 Anthropic 管理的存储的工作区保持 Anthropic Human Review 开启。Enterprise Frontier Safeguards 要求将其关闭。
  </Accordion>
</AccordionGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="检查您的设置" icon="check" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#check-your-setup">
    在 Claude Platform 上，在开始生产使用之前，确认您的设置能正常工作。
  </Card>

  <Card title="设置客户自有存储" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage">
    将您的密封的安全数据保存在您自己云账户中的存储里。
  </Card>

  <Card title="Human Review 和访问透明度" icon="shield" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency">
    了解 Anthropic 中哪些人会审查您的数据，以及您会获得什么记录。
  </Card>
</CardGroup>
