---
title: 为 Linked billing 设置您的应用
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app
description: 在 Claude Console 中将您的产品注册为 OAuth 应用，并为您的后端提供一个可以通过 Linked billing 向客户计费的身份。
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

借助 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing)，您的客户通过他们自己的 Claude 组织为您产品中的 Claude 用量付费。在任何客户进行关联之前，您需要在 Claude Console 中一次性设置好您的应用。

完成后，您的应用将拥有一个客户端 ID、[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#set-up-client-authentication)、一个[操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account)和一个[默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#default-workspace)。您还将选择[是否获取刷新令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#choose-whether-to-get-refresh-tokens)。本页末尾介绍了对客户已在使用的应用进行更改的情况。

## 开始之前

* **Admin 角色：** 本页上的每个步骤都需要您在 Claude 组织中拥有 Admin 角色。
* **适用于您后端的身份提供商：** 您的后端通过工作负载身份联合，使用来自您自己身份提供商的短期令牌来证明其身份。
* **由 Anthropic 直接计费的组织：** 您的组织必须通过预付额度或发票计费的方式，直接向 Anthropic 支付其 API 用量费用。通过云市场计费的组织无法以这种方式向客户计费。

如果您的身份提供商不在[身份提供商](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#identity-providers)列表中，您可以将任何 OpenID Connect 提供商注册为[联合颁发者](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#federation-issuers)。

## 创建应用

应用是您的产品在 Anthropic 的注册信息。它将您的重定向 URI（即您产品中在客户批准后接收浏览器跳转的地址）与一个客户端 ID 绑定在一起。

1. 在 Console 的 [OAuth 应用](https://platform.claude.com/settings/oauth-apps)页面上，点击 **Create**。

2. 输入您应用的名称和重定向 URI。

   * **名称：** 客户在批准时会看到它，因此请以您的产品名称开头，例如 “Example Analytics for Claude”。名称不能以 “Claude” 或 “Anthropic” 或与之相似的词开头，也不能在它们旁边加上 “official”。
   * **重定向 URI：** 该地址必须遵循[重定向 URI 规则](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#redirect-uri-rules)。

3. 在 **What the app can do** 下，确保已选中 **Bill and link to organizations**。这会为应用授予 `billing:attribution` 作用域，使其能够向您的客户计费。

4. 选中复选框 **I accept Linked Billing Terms on behalf of** 您的组织，然后点击 **Continue**。这些条款即 [Linked Billing Beta Terms](https://www.anthropic.com/legal/linked-billing-terms)。

5. 当对话框要求设置客户端身份验证时，点击 **Skip for now**。Console 会创建应用，您将在下一节中设置客户端身份验证。在此之前，应用页面会警告它无法进行身份验证。

6. 复制应用页面顶部的 **Client ID**。它以 `clid_` 开头。发送到 Anthropic 授权服务器 `auth.claude.com` 的每个请求都会携带它，包括将客户引导至批准页面的重定向。客户端 ID 是公开的。

7. 在 **Appearance** 选项卡上，输入您的主页并点击 **Save changes**。

8. 上传您的徽标，格式为方形 PNG 或 JPEG 文件，每边 256 到 2,048 像素，最大 2 MB。它会立即保存。如果没有徽标，客户将看到您应用名称的首字母。

新应用以测试状态开始。在上线之前，只有其 **Test users** 列表中的人员才能关联它：请参阅[发布应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#publish-the-app)。

## 设置客户端身份验证

您的后端通过客户端断言（client assertion）证明向 `auth.claude.com` 发出的请求来自您的应用：客户端断言是由您的身份提供商颁发给您后端的短期 JWT。Anthropic 不会为 OAuth 应用颁发客户端密钥。要设置客户端身份验证：

1. 在应用的 **Client authentication** 选项卡上，选择您的 **Identity provider**，即为您后端的令牌签名的颁发者，例如 AWS、Google Cloud 或 Kubernetes。如需添加，请点击 **Register a new issuer**，填写相关信息，然后点击 **Register**，或参阅[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)。
2. 设置一个 **Subject pattern**（主体模式），您后端令牌的 `sub` 声明须与之匹配，或在 **Advanced match options** 下设置声明条件。`auth.claude.com` 会接受来自您身份提供商且与这些条件匹配的任何令牌，因此请在后端允许的范围内尽可能收窄这些条件。有关模式的语法，请参阅[规则匹配语义](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#rule-matching-semantics)。
3. 点击 **Save**。该选项卡随后会显示 **Expected audience**。
4. 让您的身份提供商为您后端颁发客户端断言，其 `aud` 声明须与该受众完全一致。请保持较短的有效期，不超过您颁发者的[最长有效期](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#jwt-verification)（默认为 1 小时）：任何截获客户端断言的人都可以在其过期前使用它。

有关客户端断言必须包含的所有声明，请参阅[客户端断言声明](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-assertion-claims)。

## 为您的后端提供身份

您的后端以服务账户的身份调用 Claude API，服务账户是您 Claude 组织中的非人类账户。您选择作为应用操作员的那个账户是唯一能够向您已关联客户计费的账户。它通过以下两种方式之一计费：

* 使用其自身的凭据，并指定客户的 [用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)
* 通过客户[自己的 OAuth 访问令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer)，该令牌视为该账户，且只能向该客户计费

### 选择操作员服务账户

服务账户使用其自身的联合规则进行身份验证，这与客户端身份验证是分开的。客户端身份验证证明发往 `auth.claude.com` 的请求来自您的应用。联合规则则让您的后端能够以服务账户的身份调用 Claude API。

1. [创建服务账户及其联合规则](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#set-up-federation)。

   * **专属账户：** 使用一个不是其他应用操作员的服务账户，因为每个应用都需要自己的服务账户。
   * **规则的作用域：** 保留默认作用域 `workspace:developer`，您的后端需要它来创建用户配置文件、检查客户是否已关联以及取消关联。不要添加 `org:admin`，它涵盖整个 Admin API。
   * **规则的受众：** 如果您将其留空，它通常与您应用的 **Expected audience** 不同，因此您需要两个 JWT：请参阅[两个受众不同时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-the-audiences-differ)。

2. 在应用的 **Client authentication** 选项卡上，在 **Operator service account** 下选择该服务账户，然后点击 **Save**。

<Warning>
  一旦有客户已关联，更改或归档操作员服务账户可能会导致以您应用身份进行的部分或全部计费停止。
</Warning>

更改会使客户的 OAuth 令牌停止计费，而归档会使以您应用身份进行的所有计费停止。要从这两种情况中恢复，请参阅[如果您更改服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-change-the-service-account)。

### 选择默认工作区

使用客户自己访问令牌的请求会在您组织的一个工作区中运行，即应用的默认工作区。每个新应用都需要一个默认工作区，即使您从不使用这些令牌。在关联的最后一步，您的后端会用一次性代码换取与客户组织的连接。如果没有默认工作区，该交换会失败并返回 `unauthorized_client`。这种拒绝不会消耗该代码，因此您可以设置工作区，并在代码的 5 分钟有效期内再次发送同一代码。

保存操作员服务账户后，该设置才会出现：

1. 在应用的 **Client authentication** 选项卡上，在操作员服务账户下方找到 **Default workspace for platform auth mode OAuth tokens**（应用的 `connection_token_workspace_id` 设置），然后点击 **Choose workspace**。
2. 选择一个未归档且该服务账户是其成员的工作区，然后点击 **Save**。每个服务账户都是您组织内置的名为 **Default** 的工作区的成员，因此您始终可以选择该工作区。

<Warning>
  一旦有客户已关联，请勿归档应用的默认工作区：归档会撤销在其中运行的每个 OAuth 令牌，且无法撤消。
</Warning>

要使用其他工作区，请按照[如果您迁移到其他工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-move-to-another-workspace)操作。

## 选择是否获取刷新令牌

每个连接都会为您提供一个针对其客户的访问令牌，该令牌只能向该客户计费，并在 1 小时内过期。刷新令牌可以在不再次询问客户的情况下获取新的访问令牌。决定因素是您的后端如何计费：

* **仅使用操作员服务账户自身的凭据：** 保持 **Offline access** 关闭。
* **使用客户自己的访问令牌：** 开启 **Offline access**。如果您无法完全控制的代码代表客户调用 Claude（例如沙盒中的代理），您就会以这种方式计费。如果您希望每个计费请求都使用客户自己的访问令牌，也会以这种方式计费。

要获取刷新令牌，请转到应用的 **Permissions** 选项卡并开启 **Offline access**，这样您就可以请求 `offline_access` 作用域。然后，在您[引导每位客户进行批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)时请求 `billing:attribution offline_access`。请在客户关联之前开启它。如果您稍后才开启，每位已关联的客户都必须[再次批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#implementation-details)才能为您提供刷新令牌。您无需先取消他们的关联。

如果 **Permissions** 选项卡还列出了已弃用的作用域 `org:billing_attribution`，则移除它将无法撤消：请参阅[如果您仍在使用已弃用的作用域](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-still-use-the-deprecated-scope)。

## 检查结果

在 Console 中的应用页面上，检查以下内容：

* 页面未显示应用无法进行身份验证的警告。
* **Client authentication** 选项卡显示您的身份提供商及其 **Expected audience**。
* 同一选项卡显示您的操作员服务账户和应用的默认工作区。
* **Permissions** 选项卡显示 **Billing** 下的 `billing:attribution` 已开启。
* 如果您选择了刷新令牌，**Permissions** 选项卡还会显示 **Offline access** 已开启。

已保存的更改可能需要几分钟才能同步到 `auth.claude.com`。在最后一次更改后请等待几分钟，再关联客户。

## 决定是否使用单独的测试应用

新应用以测试状态开始，因此您可以使用生产应用及其测试用户来尝试关联。您也可以注册一个单独的应用用于测试。每个应用都有自己的连接、测试用户、状态和操作员服务账户。

<Warning>
  只要您提供 Linked billing，就请将其保持在同一个生产应用上：客户的连接仅适用于他们所关联的那个应用。
</Warning>

## 规则和特殊情况

当您更改客户已在使用的应用，或者您的应用仍在使用已弃用的作用域时，会遇到这些情况。有两个术语会反复出现：

* **再次批准：** 让客户完成与首次关联时相同的批准流程。使用其现有的用户配置文件，并且不要先取消其关联。
* **再次关联：** 按照[重新关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)操作，其中涵盖了每种情况。

### 如果您更改作用域

要为已关联的客户提供刷新令牌：

1. 在 **Permissions** 选项卡上开启 **Offline access**。
2. 让客户再次批准，并请求作用域 `billing:attribution offline_access`。
3. 用授权码交换返回的新连接 ID 替换已存储的连接 ID。

如果推送授权请求请求了应用不具备的作用域，`auth.claude.com` 会以 `invalid_scope` 拒绝该请求。在您更改应用作用域后的几分钟内、更改尚未同步之前发送的请求也会被拒绝：请等待后再次发送。

### 如果您更改服务账户

在您更改或归档操作员服务账户后，`auth.claude.com` 会以 `unauthorized_client` 拒绝对有效令牌的每次刷新。已被撤销或已过期的令牌在更改后会收到 `invalid_grant`，在归档后则可能收到这两种错误中的任意一种。

**如果您更改了它：**

| 请求 | 结果 |
| ----- | ----- |
| 使用您持有的客户访问令牌，或以先前的服务账户身份 | 在更改后不久停止向客户计费 |
| 以新的操作员服务账户身份 | 仍可向您已关联的客户计费，除非您[关闭了仅操作员身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#turn-off-operator-only-authentication)，这样只有客户自己的访问令牌才能计费 |

要恢复，请确保您的后端以新的操作员服务账户身份进行身份验证。该账户必须是应用默认工作区的成员。然后，再次关联您使用其令牌的每位客户。对于仅以操作员服务账户身份计费的客户，无需进行其他操作。

**如果您归档了它：** 在应用拥有新的操作员服务账户之前，任何请求都无法以您应用的身份计费，并且 `auth.claude.com` 会拒绝每次授权码交换。Console 会提示该服务账户已不存在并显示 **Delete app** 按钮，但您无需删除应用。如果应用的默认工作区也已归档，Console 将无法选择新的服务账户：请联系 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)。

每位客户处拥有 Admin 或 Billing 角色的人员都必须再次关联，因此请先与他们约定时间。然后：

1. [在 Console 中取消每位已关联客户的关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#unlink-a-customer)。取消关联端点无法完成此操作，因为它需要已归档的账户。
2. 创建[另一个操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account)及其联合规则。它必须是应用默认工作区的成员。
3. 在应用的 **Client authentication** 选项卡上，点击 **Operator service account** 下的 **Change**。选择新的服务账户并点击 **Change service account**。
4. 确保您的后端以新的服务账户身份进行身份验证，然后等待几分钟。
5. 让每位客户使用其现有的用户配置文件再次关联，并存储新的连接 ID 和 OAuth 令牌以替换先前的值。

### 如果您迁移到其他工作区

使用客户自己访问令牌的请求会在应用的默认工作区中运行。要迁移到其他工作区，请按顺序执行以下步骤。在第 6 步之前不要归档先前的工作区：归档会撤销其客户的 OAuth 令牌。

1. 确保操作员服务账户是新工作区的成员。
2. 在应用的 **Client authentication** 选项卡上，点击 **Change workspace**，选择新工作区，然后点击 **Save**。
3. 等待几分钟，让更改同步到 `auth.claude.com`。在等待期间进行的刷新可能会返回仍在先前工作区中运行的令牌。
4. 刷新每位拥有刷新令牌的客户的 OAuth 令牌。
5. 让每位没有刷新令牌的客户以相同的作用域再次批准。
6. 如果您不再需要先前的工作区，请将其归档。

如果您过早归档了先前的工作区，请立即停止刷新其客户的令牌。接下来会发生的情况：

| 时间 | 拥有刷新令牌的客户 | 没有刷新令牌的客户 |
| ----- | ----- | ----- |
| 在您选择新的默认工作区之前 | 刷新会收到 `invalid_grant` | 访问令牌会在几分钟内失效 |
| 在您选择之后 | 刷新会终止连接 | 保持关联 |

选择新的默认工作区后，请再次关联每位曾拥有刷新令牌的客户，并让每位没有刷新令牌的客户再次批准。

### 如果您仍在使用已弃用的作用域

在 `billing:attribution` 出现之前创建的应用可能拥有已弃用的作用域 `org:billing_attribution`。仅批准了该作用域的客户不会获得自己的令牌，因此您只能以操作员服务账户身份向其计费：

* 授权码交换会返回一个 `access_token`，但它不是客户自己的访问令牌。请勿将其作为客户自己的访问令牌使用。
* 当客户的连接终止，或您[关闭仅操作员身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#turn-off-operator-only-authentication)时，对其的计费将停止。

已弃用的作用域适用两条规则：

* **单独发送：** 如果与 `billing:attribution` 或 `offline_access` 一起发送，推送授权请求会收到 `invalid_scope`。
* **移除是永久性的：** 如果您在 **Permissions** 选项卡上移除 `org:billing_attribution`，将无法再将其添加回来。

要为此类客户提供他们自己的令牌：

1. 在应用的 **Permissions** 选项卡上，开启 **Billing** 下的 `billing:attribution`，并开启 **Offline access**。
2. 确保应用拥有[默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#default-workspace)。
3. 让客户再次批准，并请求 `billing:attribution offline_access`。

## 后续步骤

<CardGroup cols={1}>
  <Card title="Link your Claude organization 按钮" icon="link" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button">
    将客户从您的产品引导至批准您应用的按钮
  </Card>
</CardGroup>
