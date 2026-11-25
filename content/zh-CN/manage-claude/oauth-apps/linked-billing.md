---
title: Linked billing
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing
description: 了解由谁付费、哪些客户符合条件以及适用谁的条款，以便您判断 Linked billing 是否适合您的业务。
featureMetadata:
  status: beta
  betaHeader: user-profiles-billing-2026-10-01
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-fable-5
    - claude-mythos-5
    - claude-mythos-preview
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-sonnet-5
    - claude-haiku-5-5
    - claude-haiku-4-5-20251001
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API:
      availability: beta
      note: 只有发送 `billing_policy` 的请求才需要 beta 标头。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 测试期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

Linked billing 让您的客户通过他们自己的 Claude 组织为您产品中的 Claude 使用量付费。它是 [Claude Platform OAuth](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/overview) 的一项功能，因此它通过您为产品注册的 [OAuth 应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/overview#your-oauth-app)运作。客户只需将其组织关联到您的应用一次。之后，您可以在每个 Claude API 请求上选择由您还是由客户付费。

## 工作原理

假设 Acme 使用您的产品，并且已经为其自身的 Claude 使用量向 Anthropic 付费。您为 Acme 创建一个 “user profile”（[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)）：即您组织中代表该客户的一条记录。Acme 的某位人员在您的产品中点击 [Link your Claude organization 按钮](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button)。在 Claude 的批准页面上，他们批准您的应用。批准需要在 Acme 的 Claude 组织中拥有 Admin 或 Billing 角色。他们还可以为其设置每月 “spend cap”（[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#spend-caps)）。

从那时起，您的后端以您应用的 [operator service account](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account)（操作员服务账户）的身份发送 Acme 的 Claude API 请求，这是您 Claude 组织中的一个非人类账户。只有该账户可以向已关联的客户计费。客户自己的 OAuth 访问令牌也算作该账户，因此使用该令牌的请求只能向该客户计费，而不能向其他任何客户计费。您的后端会标记由 Acme 付费的请求，Anthropic 会按照 Acme 自己的费率将这些请求计费给 Acme 的组织，而不是您的组织。[Linked Billing Beta Terms](https://www.anthropic.com/legal/linked-billing-terms) 规定了如何向客户收费。

### 连接如何开始和结束

关联会创建一个连接，将客户的 Claude 组织与其用户配置文件绑定在一起。该连接属于客户的组织，而不属于批准它的人，因此即使该人离开，连接也会保留。以下是连接结束的常见方式：

* 客户取消关联您的应用。
* 您组织的 Admin 在 Claude Console 中取消关联该客户。
* 您的后端通过 API 结束该连接。

还有一些[不太常见的连接结束方式](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#after-linking)。客户取消关联时，Anthropic 不会通知您，因此请[检查客户是否仍处于关联状态](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#check-a-links-status)。

### 在每个请求上选择由谁付费

您的操作员服务账户为已关联客户发送的每个请求，都会在 `anthropic-user-profile-id` 标头中指明其用户配置文件。请求正文中的内容决定由谁付费：

* 当正文的 [`billing_policy`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#bill-a-request) 字段将 `bill_to` 设置为 `user_profile` 时，**由客户付费**。
* 当 `bill_to` 为 `self` 或正文中没有 `billing_policy` 时，**由您付费**。

因此，您可以将客户的部分使用量计费给他们，其余部分由您自己付费。而使用客户自己的访问令牌发送的请求只会向该客户计费：请参阅[为每个客户使用单独的凭据](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer)。

### 请求被拒绝时

如果 Anthropic 无法就某个请求向客户计费，Claude API 会拒绝该请求。它绝不会转而向您计费。以下是常见原因：

* 客户达到了他们为您的应用设置的支出上限，或其组织达到了每月支出限额。
* 客户账户上没有可用于支付该请求的额度。
* 连接已结束。

您可以将被拒绝的请求作为由您付费的请求再次发送。有关每种拒绝情况及其解决方法，请参阅 [Linked billing 错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors)。

## 何时使用

Linked billing 适用于用户主要是拥有自己 Claude 组织的 Claude Platform 客户的产品。要提供此功能，您自己的组织必须通过预付额度或发票计费的方式直接向 Anthropic 付款。

有些请求无法计费给客户。要处理这些请求，请将它们作为由您付费的请求发送：

* 针对[无法以这种方式计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#eligible-customers)的组织的客户的请求
* 发往[无法向客户计费的端点](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#which-endpoints-can-bill-a-customer)的请求

### 符合条件的客户

只有当客户的 Claude 组织通过预付额度或发票计费的方式直接向 Anthropic 支付其 API 使用费用时，才能以这种方式向其计费。以下组织无法以这种方式计费：

* 通过云市场（AWS、Google Cloud 或 Azure）计费的组织
* 仅拥有 Claude 订阅、自身没有 Claude API 访问权限的组织

仅拥有 Claude 订阅的组织无法关联您的应用。通过云市场计费的组织会在尝试关联时被拒绝，或者在您向其计费的每个请求上被拒绝。因此，关联完成并不表示此类组织可以被计费。

### 哪些额度可以支付

并非所有额度都可以支付应用向客户计费的请求。可用于支付的额度取决于被计费的组织：

* **当您的应用向客户计费时：**&#x5BA2;户的组织
* **当其他公司的应用向您计费时：**&#x60A8;自己的组织

在 Claude Console 中为该组织购买的额度可以支付这些请求。如果该组织采用发票计费，则会为这些请求开具发票。

Anthropic 的部分促销额度无法支付这些请求。其中包括 Claude Max、Ultra 和 Team 计划附带的每月 API 额度。来自 Anthropic 合作伙伴、风投和初创企业计划的额度不符合支付这些请求的条件。

当该组织账户上没有任何额度可以支付某个请求时，Claude API 可能会以 402 拒绝该请求。响应不会说明拒绝是由于额度不足还是支出限额造成的。请参阅 [`linked_organization_spend_limit_reached`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#linked-organization-spend-limit-reached)。

### 哪些端点可以向客户计费

| 端点 | 可以向客户计费 |
| ----- | ----- |
| Messages API（`/v1/messages`） | 是，[计费的代码执行](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#if-you-use-code-execution)除外 |
| OpenAI SDK 兼容性（`/v1/chat/completions`） | 是 |
| [Message Batches](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#if-you-use-message-batches) | 否 |
| Claude Managed Agents | 否 |

如需申请支持其他端点，请联系 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)。

## 适用谁的条款和限制

<Warning>
  如果客户的此类使用需要 HIPAA、FERPA 或客户管理的加密密钥保障，请不要向其提供 Linked billing。
</Warning>

您向客户计费的请求仍在您的组织中运行：

| 设置 | 是否适用于这些请求 |
| ----- | ----- |
| 您的速率限制、使用层级、模型访问权限和[服务层级](https://platform.claude.com/docs/zh-CN/api/service-tiers) | 是 |
| 您的使用层级的[每月支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#spend-limits) | 是 |
| 您的数据保留和加密密钥 | 是 |
| 客户的[零数据保留（ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#zero-data-retention-zdr-scope)、业务伙伴协议（BAA）、客户管理的加密密钥（CMEK）以及 IP 允许列表 | 否 |
| 客户为您的应用设置的支出上限及其组织的每月支出限额 | 是 |
| 您为您的组织及其工作区设置的支出限额 | 否 |

一旦您的组织达到您的使用层级的每月支出上限，Claude API 就会拒绝其所有请求，包括向客户计费的请求：请参阅 [`enforced_spend_limit_reached`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#enforced-spend-limit-reached)。您向客户计费的使用量不计入该上限。

## 您可以计费的内容

仅就客户指示您产生的使用量向其计费。即客户及其用户在您的产品中产生的 Claude 使用量。不要就您自己的使用量向他们计费，例如测试、评估或为其他客户进行的工作。Claude API 无法为您核查这一点，因此这是您的责任。您在创建应用时即接受 [Linked Billing Beta Terms](https://www.anthropic.com/legal/linked-billing-terms)。

## 由谁为客户提供支持

您需要为客户提供有关您应用的一切支持，包括您向他们计费的 Claude 使用量。Anthropic 仅在客户的 Claude Console 中为他们提供[批准页面和 Linked platforms 页面](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see)。有关您应用的所有其他问题都会找到您，因此请告诉您的客户如何联系您。

## 规则和特殊情况

有些功能无法向客户计费，有些功能会以各自的方式处理 `billing_policy`。

### 如果您使用 Message Batches

将 [message batch](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)（消息批处理）中的请求作为由您付费的请求发送：将 `bill_to` 设置为 `self` 或不包含 `billing_policy`。它们会照常计费给您。批处理中标记为由客户付费的请求会被拒绝。其结果为 `errored`，并带有 `invalid_request_error`，且不会向任何人计费。批处理中的其他请求照常运行。

### 如果您使用代码执行

向客户计费的请求不能使用[计费的代码执行功能](https://platform.claude.com/docs/zh-CN/about-claude/pricing#code-execution-tool)：API 会以 400 拒绝该请求。请将此类请求作为由您付费的请求发送。

### 如果您计算令牌数

当请求带有 `user-profiles-billing-2026-10-01` beta 标头时，令牌计数端点（`/v1/messages/count_tokens`）会接受 `billing_policy` 字段，并忽略该字段。您无需在计算令牌数之前将其删除。

## 可用性

Linked billing在所有模型上处于 beta 阶段。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台 |
| ----- | ----- |
| Beta<sup>1</sup> | Claude API |

1. 只有发送 `billing_policy` 的请求才需要 beta 标头。

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `user-profiles-billing-2026-10-01`
