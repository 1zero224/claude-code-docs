---
title: Linked billing 错误
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors
description: 了解关联或计费客户时可能返回的每个错误的处理方法，以及按检查顺序排列的原因。
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

以下是 `auth.claude.com` 在关联过程中返回的错误，以及 Claude API 在计费过程中返回的错误，[刷新失败](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails)的情况除外。

## 关联过程中

这些错误会出现在您的重定向 URI 上，或来自授权码交换。连接（connection）是将客户的 Claude 组织与您的某个[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)绑定在一起的记录。在每种情况下，都不会建立连接。有关标准 OAuth 错误，请参阅 [OAuth 端点和参数](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#errors)。

### `access_denied`

首先，检查您的应用：原因 3 和 4 需要由您来修复。

否则，请告知此人关联未完成，不要假设他们拒绝了。请他们检查自己登录时使用的是哪个 Claude 账户，以及他们在想要关联的组织中的角色。如需让他们重试，请使用新的推送授权请求[引导他们进行批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)。

对于以下原因，您的重定向 URI 会收到 `error=access_denied` 以及您的 `state` 值：

1. 此人在批准界面上点击了 **Cancel**。

2. 批准页面没有可提供的组织，此人点击了返回您应用的按钮。以下情况会发生这种情况：

   * 此人在任何 Claude 组织中都没有 Admin 或 Billing 角色
   * 此人唯一的组织就是拥有您应用的那个组织
   * 您发送的用户配置文件已关联到另一个组织，而此人在该组织中没有 Admin 或 Billing 角色
   * 此人所属的组织全部属于无法关联的类型：请参阅[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#eligible-customers)

3. 您应用的 **Who can link** 设置在您组织名称后显示"only"。没有人能够关联这样的应用。在其 **Settings** 选项卡上，选择 **Any Claude user** 并点击 **Save changes**。

4. 您的应用尚未上线，此人不在其 **Test users** 列表中，并且他们点击了返回您应用的按钮。[将他们添加为测试用户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#add-test-users)，或[发布应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#publish-the-app)。

对于这些原因，`error_description` 显示为 `the user declined the request`，因此无法据此判断发生的是哪一种情况。如果它显示的是 `the request cannot be approved for this organization`，则说明此人选择的组织无法批准您的应用。请让他们选择另一个组织。

### `user_profile_already_linked`

每个客户的组织同一时间只能与您的应用建立一个连接，您的每个配置文件也是如此。`error_description` 会说明适用以下两种原因中的哪一种：

1. **该组织已有连接**，通过您的另一个配置文件建立。这是常见原因，`error_description` 以 `This organization already has an active connection` 开头。在您的 [User profiles 页面](https://platform.claude.com/user-profiles)上，查找该客户显示为 **Linked** 的另一个配置文件。Anthropic 不会显示配置文件关联到哪个组织，因此请与客户确认该组织是他们的。

   * **您找到了它：**&#x901A;过该配置文件向客户计费，或[取消关联该配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#unlink-a-customer)，然后使用被拒绝的配置文件再次引导客户进行批准。
   * **该客户的配置文件都未关联：**&#x6B64;人可能选择了属于您另一位客户的组织。再次引导他们进行批准，并请他们检查所选择的组织。

2. **您发送的配置文件已有连接**，连接到另一个组织。其 `error_description` 以 `This user profile already has an active connection` 开头。请先结束该连接：请参阅[将客户迁移到另一个组织](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#move-a-customer-to-another-organization)。

您无法提前捕获此错误：推送授权请求会成功。

该错误会出现在以下两个位置之一：

* **在您的重定向 URI 上：**&#x5982;果此人返回您的应用，重定向会携带 `error=user_profile_already_linked` 以及您的 `state` 值。`auth.claude.com` 已经要求他们凭此代码联系您。
* **来自令牌端点：**&#x5982;果连接是在批准之后建立的，授权码交换会返回此错误。

该错误不会指明持有该连接的配置文件。

### 无重定向

如果此人在没有回调的情况下返回您的产品，请使用新的推送授权请求再次引导他们进行批准。您不会收到回调，也无法区分以下情况：

* 此人关闭了批准页面。
* `request_uri` 已过期。它在您创建后 10 分钟过期。
* `request_uri` 已被使用。每个只能使用一次。

在后两种情况下，批准页面会显示错误，并且不会将浏览器送回您的应用。

### 来自授权码交换的 `unauthorized_client`

在 Claude Console 中检查您应用的设置，修复缺失的部分，然后再次交换同一个授权码。此次拒绝不会消耗该授权码，授权码有效期为 5 分钟。

| 检查项 | `error_description` | 修复方法 |
| ----- | ----- | ----- |
| 应用具有 操作员服务账户 | `the client is not registered for this grant` | [选择操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account) |
| 应用具有未归档的默认工作区 | 指明工作区问题 | [选择默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#default-workspace) |
| 操作员服务账户是该工作区的成员 | 指明工作区问题 | [选择该账户所属的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#default-workspace)，例如名为 **Default** 的内置工作区 |

已保存的更改可能需要几分钟才能同步到 `auth.claude.com`，因此如果交换再次被拒绝，请等待后重试。如果授权码先过期，请再次引导客户进行批准。

如果 Console 显示该服务账户已不存在，请参阅[如果您更改服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-change-the-service-account)。对于任何其他原因，请联系 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)，或联系您的 Anthropic 客户团队（如果有）。

## 计费过程中

Claude API 在 Claude 开始回复之前拒绝的请求不会向任何人计费。错误详情的位置取决于端点：

| 端点 | 错误代码位置 | 请求 ID 标头 |
| ----- | ----- | ----- |
| `/v1/messages` | `error.details.error_code` | `request-id` |
| `/v1/chat/completions` | 不发送 | `x-request-id` |

在 `/v1/chat/completions` 上，错误采用另一种形式：

* `error.type` 为 `invalid_request_error`。
* `error.code` 包含错误类型，例如 `billing_error`，而不是本页标题中的某个错误代码。

请根据 HTTP 状态和 `error.code` 进行匹配。在那里，两个 `402` 错误看起来相同，`403` 错误也是如此，因此请按照所有符合条件的条目处理。针对访问令牌的 `401` 在两个端点上都采用 `/v1/messages` 形式并带有 `request-id` 标头。

`/v1/messages` 上的拒绝采用以下形式：

```json
{
  "type": "error",
  "error": {
    "type": "billing_error",
    "message": "...",
    "details": {
      "error_code": "linked_organization_spend_limit_reached"
    }
  }
}
```

本页后面的 `401` 和 `503` 错误没有错误代码。在 `/v1/messages` 上，请根据状态和 `error.type` 进行匹配。

在 Anthropic SDK 中，捕获对应 HTTP 状态的错误类，然后从其 `error_code` (typescript: `errorCode`; csharp: `ErrorCode`; go: `ErrorCode()`; java: `errorCode()`; php: `getErrorCode()`) 读取错误代码。`402` 对应的类是 `PaymentRequiredError` (ruby: `Anthropic::Errors::PaymentRequiredError`; java, php: `PaymentRequiredException`; csharp: `AnthropicPaymentRequiredException`; go: `*anthropic.Error`)，`403` 对应的类是 `PermissionDeniedError` (ruby: `Anthropic::Errors::PermissionDeniedError`; java, php: `PermissionDeniedException`; csharp: `AnthropicForbiddenException`; go: `*anthropic.Error`)。

除 Go 外，每个类涵盖其状态下的所有错误。Go 对所有状态只有一个类型 `*anthropic.Error`：请检查其 `StatusCode`。其中一个代码也可能出现在流式回复中：请参阅 [`linked_organization_billing_unavailable`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#linked-organization-billing-unavailable)。

记录每个被拒绝请求的完整 `error.message` 和请求 ID。如需自费为被拒绝的客户提供服务，请将请求[作为由您付费的请求](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#you-pay)重新发送，并在 `billing_policy` 中设置 `bill_to: "self"`。请以您的[操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account)身份发送，而不是使用客户的访问令牌。

对于 `402` 或 `403`，在原因持续存在期间，重新发送相同的请求不会有帮助。新连接的最初几秒是个例外：请参阅 [`user_profile_billing_not_authorized`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-billing-not-authorized)。

同时符合两个 `403` 代码原因的请求可能会收到其中任一代码，例如，为一个同样未关联的客户使用了错误的凭据。请将这两个代码放在一起处理，如果某个条目的原因无法解释拒绝，请检查另一个条目的原因。

### 403 `user_profile_billing_not_authorized`

没有任何连接允许您的请求向其指定的用户配置文件计费。响应不会说明原因。要区分原因 2 至 4，请[检查该客户是否已通过您预期的应用关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#check-a-links-status)。

请按顺序检查以下原因：

1. **您的请求。**&#x8BF7;求必须指定该客户关联的用户配置文件。它还必须以客户所关联应用的操作员服务账户身份进行身份验证，或使用该连接的访问令牌。如果客户未通过您的另一个应用进行关联，那么使用该应用的服务账户会收到此错误。请更正请求：新的连接无济于事。

2. **客户尚未通过此应用进行关联。**[连接状态](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#check-a-links-status)显示为 `not_connected`。仅通过您另一个应用（例如[单独的测试应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-run-several-apps)）关联的配置文件并未关联到此应用。在客户[通过此应用进行关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)之前，请求只能通过 `bill_to: "self"` 向您计费。

3. **连接已结束。**&#x8FDE;接状态显示为 `disconnected`，并且该配置文件在您的 [User profiles 页面](https://platform.claude.com/user-profiles)上显示为 **Unlinked**。状态中的 `last_revoked_reason.type` 会说明原因。在客户[再次关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)之前，请求只能通过 `bill_to: "self"` 向您计费。

4. **连接状态显示为 `connected`，但请求仍被拒绝。**

   * **新连接：**&#x65B0;连接通常会立即生效，但在授权码交换后可能需要约 10 秒。如果连接是刚刚建立的，请等到交换后约 10 秒，然后再发送一次请求。
   * **先前连接的访问令牌：**&#x5BA2;户的访问令牌仅对签发它的连接有效。该连接结束后，使用其令牌的请求可能会收到此错误或 [`401 authentication_error`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#authentication-error)，即使客户已再次关联。请使用新连接的访问令牌。
   * **都不是：**&#x5C06;被拒绝请求的 ID 发送给 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)，或发送给您的 Anthropic 客户团队（如果有）。

### 403 `user_profile_not_billable`

在要求客户再次关联之前，请先找出原因，因为再次关联同一组织只能解决原因 2。

请按顺序检查以下原因：

1. **您请求的凭据。**&#x8BF7;求必须以您应用的[操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account)身份进行身份验证，或使用客户的访问令牌。任何其他凭据都会被拒绝，例如未绑定到服务账户的 API 密钥。不担任任何 OAuth 应用操作员的服务账户也会被拒绝，例如[您已替换的操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-change-the-service-account)。请更正请求。

2. **仅操作员身份验证。**&#x5982;果您[关闭了仅操作员身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#turn-off-operator-only-authentication)，则只有客户自己的访问令牌才能向其计费。仅来自您服务账户的请求会被拒绝，即使客户显示为 **Linked**。

   * **您持有客户的访问令牌：**&#x4F7F;用该令牌发送请求。如果它已过期，请先[刷新 OAuth 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#refresh-the-tokens)。
   * **您没有刷新令牌，或刷新返回 `invalid_grant`：**[再次关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

3. **客户的组织。**&#x60A8;无法从您这边看到其状态，因此请询问客户其组织如何支付 Claude API 用量费用。

   * **它不符合条件：**&#x5B83;必须[直接向 Anthropic 付费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#eligible-customers)。一些通过云市场计费的组织可以进行关联，但随后您向其计费的每个请求都会收到此错误。
   * **它未设置计费：**&#x65B0;组织可以在设置计费之前进行关联。在此之前，它会收到此错误，而不是余额不足的 `402`。请客户在其 [Billing 页面](https://platform.claude.com/settings/billing)上购买额度，然后再次发送请求。购买可能需要一些时间才能生效，因此如果您仍然收到此错误或 [`linked_organization_spend_limit_reached`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#linked-organization-spend-limit-reached)，请稍后重试。

4. **只有 Anthropic 才能看到的问题。**&#x5BA2;户显示为 **Linked**，且您的请求正确。将被拒绝请求的 ID 发送给 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)，或发送给您的 Anthropic 客户团队（如果有）。

### 402 `linked_organization_spend_limit_reached`

请客户方具有 Admin 或 Billing 角色的人员在 Claude Console 中检查以下各项。响应不会说明适用的是哪一项：

* **他们为您的应用设置的支出上限：**&#x5728; [Linked platforms](https://platform.claude.com/linked-platforms) 下您应用的页面上，他们可以设置或提高该上限。在此处设置的上限会替代他们为所有关联平台设置的默认上限。
* **他们组织的每月支出限额：**&#x5728; [Spend limits](https://platform.claude.com/settings/billing#spend-limits) 下，他们可以提高该限额。
* **他们的额度余额：**&#x5728;其 [Billing 页面](https://platform.claude.com/settings/billing)上，他们可以购买额度。购买可能需要一些时间才能生效，因此在购买后不久发出的请求仍可能收到此错误。如果是这样，请稍后重试。某些额度无法用于支付这些请求：请参阅[哪些额度可以支付](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#which-credits-can-pay)。因此，余额高于 0 美元 时仍可能收到此错误。

在您的应用尚未上线期间，上限不能超过 50 美元。届时在 50 美元处的拒绝会持续到下个月开始（UTC）：请参阅[测试期间的上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#caps-while-testing)。

已设置计费但没有额度的组织会收到此代码。尚未设置计费的组织会收到 [`user_profile_not_billable`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-not-billable)。

### 402 `linked_organization_billing_unavailable`

处理方法取决于错误出现的时间：

* **回复开始之前：**&#x41;nthropic 无法向客户的组织计费，例如因为该组织已被暂停。响应不会说明原因。请客户检查其 Claude 账户以及其在 Anthropic 的计费状态。
* **Claude 开始回复之后：**&#x901A;常是组织的额度已用完，不过暂停也可能中断回复。请客户在其 [Billing 页面](https://platform.claude.com/settings/billing)上购买额度或检查其账户。

非流式请求在这两种情况下都会收到 HTTP `402`，因此请两者都检查。在请求到达时没有额度的客户会改为收到 [`linked_organization_spend_limit_reached`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#linked-organization-spend-limit-reached)。

对于流式回复，回复被中断时 HTTP 状态已经是 `200`，错误会出现在流中：

* **在 `/v1/messages` 上：**&#x4F5C;为一个 `error` 事件，其 `error.type` 为 `billing_error`，`error.details.error_code` 为此代码。SDK 中 `402` 对应的类不会捕获它。
* **在 `/v1/chat/completions` 上：**&#x4F5C;为最后一行 `data:`，包含一个 OpenAI 错误对象，`error.code` 中为 `billing_error`，其后没有 `[DONE]`。

被中断的回复不会向任何人收费。如果请求已经完成了某个步骤，例如 Claude 在网络搜索之前的轮次，则客户需要为该步骤付费。

### 400 `user_profile_required`

在 `anthropic-user-profile-id` 标头中发送客户的用户配置文件 ID。该请求设置了 `bill_to: "user_profile"` 但没有该标头。使用[客户访问令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#call-claude-with-a-token)的请求不需要该标头。

### 401 `authentication_error`

使用客户最新的访问令牌重新发送请求，或[刷新 OAuth 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#refresh-the-tokens)。仅当客户的配置文件显示为 **Unlinked**、您没有其刷新令牌，或刷新返回 `invalid_grant` 时，才需要[再次关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

此条目涵盖使用客户访问令牌但该令牌已失效的请求：令牌已过期、已被刷新替换或已被撤销。刷新也会撤销先前的令牌，因此 `error.message` 中的"revoked"本身并不意味着连接已结束。连接结束后，使用其令牌的每个请求都可能收到此错误或 [`403 user_profile_billing_not_authorized`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-billing-not-authorized)，顺序不固定。

### 429 `enforced_spend_limit_reached`

[为您的组织申请更高的每月支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#requesting-higher-limits)，或等待下个月开始（UTC）。再次发送请求无济于事，即使作为由您付费的请求发送也是如此。

您的组织已达到您所在使用层级的每月支出上限。原因并非客户为您的应用设置的支出上限，因此客户无法修复此问题。请参阅[达到支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#reaching-your-spend-cap)。

### 503 `overloaded_error`

使用退避策略重试。如果响应带有 `retry-after` 标头，请至少等待该时长。Claude API 无法确认是否可以向客户计费，因此它未处理该请求，也未计费。

### 其他无代码的错误

| 状态和类型 | 原因 | 修复方法 |
| ----- | ----- | ----- |
| `400 invalid_request_error` | `anthropic-user-profile-id` 标头中的 ID 格式错误，或不是您组织的用户配置文件之一 | 发送客户的用户配置文件 ID |
| `400 invalid_request_error` | 请求发送了 `billing_policy`，但没有 `user-profiles-billing-2026-10-01` beta 标头 | 添加 beta 标头 |
| `400 invalid_request_error` | `billing_policy` 中有缺失或无效的字段 | 更正 `billing_policy` |
| `400 invalid_request_error` | 请求向客户计费并使用了[计费的代码执行功能](https://platform.claude.com/docs/zh-CN/about-claude/pricing#code-execution-tool) | 以您的操作员服务账户身份发送，并向您计费 |
| `errored` 批处理结果中的 `invalid_request_error` | 批处理中的某个请求向客户计费：请参阅[如果您使用 Message Batches](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#if-you-use-message-batches) | 发送时向您计费 |
| `400 invalid_request_error` | 请求使用客户的访问令牌并设置了 `bill_to: "self"` | 以您的操作员服务账户身份发送由您付费的请求 |
| `400 invalid_request_error` | 请求使用客户的访问令牌并指定了另一个用户配置文件 | 仅指定该客户的配置文件 |
| `403 permission_error` | 请求在不接受客户访问令牌的端点上使用了该令牌，例如 [Message Batches](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) | 以您的操作员服务账户身份发送 |
| `403 permission_error` | Anthropic 不直接向您的组织计费，因此无法代表您向客户计费 | 从直接向 Anthropic 付费的组织向客户计费 |
