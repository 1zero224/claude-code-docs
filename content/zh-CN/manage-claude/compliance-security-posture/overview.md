---
title: Compliance API 中的 Security posture
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview
description: 了解 Compliance API 公开了 claude.ai 中 Security posture 页面（Security Center）的哪些内容，以及从何处入手。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

claude.ai **组织设置**中的 **Security posture** 页面会将您组织的 Claude 设置与 Anthropic 推荐的安全控制项进行比较。通过 Compliance API，您的安全工具可以读取该页面记录的变更。它们还可以读取当前结果。Security posture 只报告您的设置，不会更改这些设置。侧边栏项目为 **Security posture**，其打开的页面标题为 **Security Center**。

## 选择您的路径

要将该页面上的变更导入您的安全信息和事件管理（security information and event management，SIEM）系统，请从[将 Security posture 事件发送到您的 SIEM](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events) 开始。如果您已经在摄取整个活动源，那么您已经拥有这些事件；请[检查两件事](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events#if-you-already-ingest-the-full-feed)。

* 要针对有风险的变更发出告警，请参阅[检测 Security posture 中的变更](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes)。
* 要规范化为开放网络安全架构框架（Open Cybersecurity Schema Framework，OCSF），请参阅[将 Security posture 事件映射到 OCSF](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/ocsf-mapping)。
* 要查明谁在何时接受了哪项风险以及接受到何时，请参阅[Security posture 事件参考](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference)；如需查看生效中的风险，请参阅[当前结果](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#current-results)。

要使用来自端点的当前结果，请参阅以下页面：

* 对于仪表板，请参阅[Security posture 结果参考](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference)。
* 要验证已存储的事件，请参阅[将已存储的 Security posture 事件与当前结果进行核对](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results)。
* 如果端点与页面不一致，请参阅[声明在响应中的显示方式](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#how-an-attestation-shows-in-the-response)。

要了解这些数据，请继续阅读。

## Security posture 评估的内容

Security posture 会评估一系列控制项，并为每个控制项给出一个结果。

### 控制项

控制项是一项推荐的设置或做法，例如单点登录。每个控制项都有一个显示名称和一个标识符。该标识符对所有组织都相同。控制项还可以有当前值（`current_value`）和推荐值（`desired_state`）。控制项分为三种：

| 类型      | 决定其结果的因素                                     | 示例                         |
| ------- | -------------------------------------------- | -------------------------- |
| 自动控制项   | 将当前值与推荐值进行比较的自动检查；但有生效中的声明时除外。               | `sso_enabled`              |
| 手动控制项   | 由人决定，因为 Anthropic 无法读取该设置，例如该设置位于其他供应商的控制台中。 | `oauth_conditional_access` |
| 仅供参考控制项 | 无：没有推荐值，因此该控制项只做报告，不予评定。                     | `allowed_invite_domains`   |

### 声明

声明是某人记录下来的、关于组织在某个控制项上所处状况的陈述；它不是检查的结果。大多数控制项都可以带有声明，包括自动控制项，无论其检查结果如何。声明具有以下三个值之一：

* “已完成”（`completed`）：该控制项已落实。
* “已接受风险”（`accepted_risk`）：该控制项未落实，且组织接受该风险。
* “未通过”（`control_failed`）：该控制项未落实。

有权进行声明的人员在页面上进行声明。API 可以读取声明，但无法创建声明。当有人进行声明时，页面可以提出一个问题，有时会要求作答，并将回答作为该声明的[备注](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#what-anthropic-stores-when-someone-attests)随声明存储，最多 2,000 个字符。活动源中的活动从不包含该备注，也不会显示存在备注。端点也不返回备注。

### 结果

每个控制项都具有以下四种结果之一。

| 结果                    | 含义                                      |
| --------------------- | --------------------------------------- |
| “通过”（`passed`）        | 满足推荐值，或者有生效中的“已完成”或“已接受风险”声明。           |
| “未通过”（`failed`）       | 不满足推荐值，或者被声明为“未通过”（`control_failed`）。   |
| “未声明”（`not_attested`） | 没有生效中声明的手动控制项。                          |
| “描述性”（`descriptive`）  | 未评定，即使已声明也是如此；涵盖仅供参考控制项，以及不适用或无法读取的控制项。 |

在自动控制项或手动控制项上，生效中的已接受风险计为“通过”，尽管该控制项未落实。在响应中，[在其检查未通过时被声明为“已完成”](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#how-an-attestation-shows-in-the-response)的自动控制项为 `failed`。

### 什么会终止声明

声明会一直生效，直到发生以下三种情况之一：

* 有人撤销或更改该声明。
* 到达其结束日期（`review_date`）。进行声明的人员可选择 90 天或不设结束日期。
* 在自动控制项或仅供参考控制项上，当前值发生变化；对于部分自动控制项上的已接受风险，则为自动检查通过。

没有生效中的声明时，自动控制项的结果取决于其检查，手动控制项则为“未声明”。

## 随时间推移的变更：活动源

活动源回答发生了什么变更、何时变更以及由谁变更。当有人对控制项进行声明、更改声明或撤销声明时，它会记录一条活动；有时在声明自动终止时也会记录。

这些页面的标题将这些活动称为事件。读取它们需要 `read:compliance_activities` 或 `read:org_audit` 作用域。[Security posture 事件参考](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference)列出了它们的字段。要收集这些事件，请参阅[将 Security posture 事件发送到您的 SIEM](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events)。

## 当前结果：端点

端点回答在请求时控制项的结果是什么。使用它可以对您的[父组织](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api#how-the-compliance-api-works)下的各个组织进行报告。每次调用读取一个组织，且不返回历史记录。对于其中列出的每个控制项，响应会给出其 `id`、`display_name`，以及 `rating` 中的结果，结果使用[结果表](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#results)中的标识符。

调用它需要 `read:compliance_org_data` 或 `read:org_audit` 作用域。

页面、端点和活动源显示的内容并不总是相同，因为响应中会略去部分控制项和部分声明。请参阅[声明在响应中的显示方式](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#how-an-attestation-shows-in-the-response)和[将已存储的 Security posture 事件与当前结果进行核对](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results)。

## 可用性

Security posture 处于 beta 阶段。

**如果您的组织使用 Claude Enterprise、Security posture 对其可用，并且其父组织已启用 Compliance API，** 活动源就会为其记录声明活动，端点也会返回您的组织的结果。

您还需要具有相应作用域的 Compliance Access Key。这些页面假定您已阅读[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)和[查询活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)。

**如果您的组织使用其他套餐，或者是 Claude Console 组织，** 活动源不记录该组织的 Security posture 活动，端点也会返回 404。

**如果针对您的组织，Security posture 页面已被关闭，** 端点就会对您的组织返回 404，直到该页面被开启。即使使用父组织的密钥，也会收到该 404。

对于某些组织，即使满足全部三个条件且该页面未被关闭，端点也会返回 404。

## 谁可以看到声明者

由某人引起的每条声明活动都会记录其用户 ID、其当时的电子邮件地址，以及请求的 IP 地址和用户代理。任何覆盖该组织并拥有 `read:compliance_activities` 或 `read:org_audit` 的密钥都可以读取这些信息，即使此人已离开该组织或其账户已被删除。删除该组织不会将这些活动从活动源中移除，活动源会将活动保留 6 年。该组织自身的密钥会在该组织被删除时停止工作。

任何同时拥有 `read:compliance_org_data` 和 `read:compliance_user_data`，或拥有 `read:org_audit` 的密钥，都可以在端点的 `attested_by` 中读取声明者的用户 ID（在端点返回该信息的情况下），即使是在您获得访问权限之前所做的声明也是如此。

这些值属于个人数据；您保存的副本在此人的成员资格结束后仍会存在。在收集这些数据之前，请告知进行声明的人员。

## 有人进行声明时 Anthropic 存储的内容

Anthropic 为每个控制项存储一条声明，包括其值、做出声明的时间、结束日期（如有）以及进行声明的人员的账户等内容。再次进行声明会替换该声明。

在声明到达其结束日期或[自动终止](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#what-ends-an-attestation)之后，Anthropic 会继续存储该声明，直到有人撤销该声明或 Anthropic 在删除该组织时一并删除该声明。如果进行声明的人员的账户被删除，Anthropic 会继续存储该声明，但该声明将不再注明由谁进行了声明。活动源中由此人引起的任何活动仍会注明此人。撤销或替换声明不会从活动源中移除任何内容（请参阅[谁可以看到声明者](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#who-can-see-who-attested)）。

[客户管理的加密密钥（CMEK）](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)、[Claude Code 的零数据保留（ZDR）](https://code.claude.com/docs/en/zero-data-retention)以及 [HIPAA 就绪的 Enterprise 计划](https://support.claude.com/en/articles/13296973-hipaa-ready-enterprise-plans)，都不会改变 Anthropic 就声明的控制项、值、日期和账户存储哪些内容。这三者中的任何一项都不会缩短 Anthropic 存储这四项内容的时长。CMEK 不会加密声明的这四项内容。

Anthropic 会存储声明的备注，直到有人撤销该声明或 Anthropic 在删除该组织时一并删除该备注。再次进行声明可能会替换或移除该备注。在 Security posture 对某个组织不可用期间，Anthropic 会继续存储该组织的声明和备注，并且任何人都无法在 Security posture 页面上查看或撤销它们。自定义对话保留期不会缩短 Anthropic 存储该声明或该备注的时长。活动源中的活动从不包含该备注（请参阅[活动源不记录的内容](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#what-the-feed-does-not-record)）。
