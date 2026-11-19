---
title: 应用测试与审核
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review
description: 新的 Linked billing 应用如何结束测试阶段并向您的客户开放，以及更改已上线的应用时会发生什么。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：它是根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 beta 期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

借助 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections)，客户会将其 Claude 组织关联到您的 OAuth 应用。新的 Linked billing 应用从测试阶段开始，在此阶段只有您指定的 “test users”（测试用户）才能将其组织关联到该应用。要向您的客户开放应用，您需要发布它，然后由 Anthropic 进行审核。

如果您的组织使用发票计费，Anthropic 不会审核您的应用。请参阅[发票计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#invoiced-billing)。

当 Anthropic 批准了应用，或者您在使用发票计费的情况下发布了应用时，应用即为已上线状态。任何[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#eligible-customers-and-endpoints)都可以将其组织关联到已上线的应用，并且其测试用户建立的关联会保留。在 Claude Console 的 OAuth 应用页面上，已上线应用的状态显示为 **Active**。

在应用上线之前，Console 会为其显示以下状态之一：

| 状态               | 含义                                  |
| ---------------- | ----------------------------------- |
| **Testing**      | 您尚未发布该应用。                           |
| **In review**    | Anthropic 正在审核该应用，或审核您在应用上线期间所做的更改。 |
| **Not approved** | Anthropic 已审核该应用或更改，但未予批准。          |

在更改后重新进入审核的应用，在 Anthropic 批准该更改之前不会再次上线。每个 Linked billing 应用都有自己的状态和自己的测试用户。请参阅 Linked billing 指南中[步骤 1：创建应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#step-1-create-the-app)里的说明。

## 开始之前

完成 Linked billing 指南中的[设置流程](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#setup-process)。

## 管理测试用户

### 添加测试用户

1. 在 Console 的 [OAuth 应用页面](https://platform.claude.com/settings/oauth-apps)中打开应用的页面。
2. 在 **Settings** 标签页上，前往 **Who can link** 下的 **Test users** 列表。
3. 输入测试用户的 Claude 账户电子邮件地址，然后点击 **Add**。对每个测试用户重复此操作。
4. 点击 **Save changes**。

保存后，列表中的人员即可将其组织关联到您的应用。在您的应用上线之前，Claude 批准页面（用户在此批准关联）会拒绝其他任何人。关于您的应用随后会收到什么，请参阅 Linked billing 指南中的[您的重定向 URI 收到 `error=access_denied`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#access-denied)。

只要应用尚未上线，**Who can link** 就会保持为 **Test users only**，并且您无法更改此设置。该列表最多可容纳 10 个测试用户。每个地址都必须与账户的地址完全一致，但不区分字母大小写。带有加号标签的地址（例如 `name+test@example.com`）被视为与 `name@example.com` 不同的地址。

### 关联测试用户的组织

测试用户从您的应用关联其组织，并在客户使用的同一页面上批准关联。他们需要在该组织中拥有管理员或账单角色。具体步骤是 Linked billing 指南中的步骤 4 到步骤 8。该指南中的[以客户身份测试](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#test-as-a-customer)会引导您完成这些步骤，并说明如何检查关联是否已建立。

要亲自测试应用，请将您自己的电子邮件地址添加到列表中。拥有该应用的组织无法关联到该应用，因此请使用您的另一个 Claude 组织。

测试用户关联其组织后，[发送计费到该组织的请求](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#step-3-update-calls-to-claude)来试用您的应用。测试使用真实资金。Anthropic 会像对任何客户一样，就您计费到每个测试用户组织的请求向该组织收费。

在您的应用上线之前，每个新关联都需要不超过 50 美元的每月 “spend cap”（支出上限）。关于该上限的规则，请参阅指南中的[应用处于测试阶段时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#while-your-app-is-in-testing)。

### 移除测试用户

移除测试用户会立即生效。它会终止该人员已批准的每个到您应用的关联。关联终止后，Claude API 会拒绝您计费到该组织的请求。请参阅[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#expected-errors)。

1. 如果 **Settings** 标签页有未保存的更改，请点击 **Save changes** 或 **Discard**。在此之前，Console 不允许您移除已保存的测试用户。
2. 从 **Test users** 列表中删除该测试用户的地址。
3. 当 Console 询问时，点击 **Remove** 进行确认。您无需点击 **Save changes**。

## 发布应用

发布时可能会要求您通过添加 DNS 记录来证明您控制应用的域名。随后 Anthropic 会审核该应用。DNS 更改需要时间，因此请在您需要客户关联之前就开始。

在发布之前，请检查以下事项：

* 已设置好客户端身份验证。在设置好之前，您无法发布应用。
* 您的应用有一个主页，您可以在 **Appearance** 标签页上设置它。
* 您可以更改主页所在域名的 DNS 记录。
* 每个重定向 URI 都位于主页的域名或其某个子域名上。

关于什么算作主页的域名以及例外情况，请参阅[域名规则](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#domain-rules)。

按照以下步骤发布应用：

1. 在 Console 中的应用页面上，点击 **Publish app**。随即会打开一个对话框。
2. 如果对话框显示 **Prove your domain**，请在管理域名 DNS 的位置添加对话框提供给您的 DNS TXT 记录。否则，请转到第 5 步。
3. 点击 **Check**。如果域名仍显示 **Not proven yet**，请参阅[如果您无法证明域名](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#if-you-cant-prove-the-domain)。
4. 当域名显示 **Proven** 时，点击 **Continue**。
5. 如果对话框显示一个用于接受 Linked Billing Terms 的复选框，请勾选它。然后在对话框中点击 **Publish app** 进行确认。

在 Anthropic 审核应用期间，其状态为 **In review**。关于审核结果，请参阅[审核](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#the-review)。

### 如果您无法证明域名

* **域名仍显示 Not proven yet：** DNS 更改最多可能需要 48 小时才能生效。您可以关闭对话框，稍后再回来。如果您之前在发布，请再次点击 **Publish app**。如果您之前在更改已上线的应用，请再次进行相同的编辑，在记录过期之前，Console 会为您提供同一条记录。
* **Check 提示该记录已过期：** DNS 记录自 Console 首次显示它起 30 天内有效。之后，请点击 **Get a new record** 并添加新记录。显示 **Proven** 的域名会保持已证明状态。
* **对话框显示一条消息而不是记录：** 例如，消息可能提示您的组织已有 20 个域名。在该问题解决之前，您无法继续。如需帮助，请[联系支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)。

### 域名规则

域名是不带前导 `www.` 的主机名，您的域名即您主页的域名。例如，位于 `https://www.example.com` 的主页的域名是 `example.com`。已证明的域名涵盖其自身及其子域名。

* **共享域名：** 托管服务的共享域名不符合条件，因此请使用您自己的域名。
* **本地重定向 URI：** 位于 `localhost`、`127.0.0.1` 或 `[::1]` 上的重定向 URI 不必位于主页的域名上。
* **同级子域名：** 如果您的主页位于 `app.example.com`，您的域名就是该子域名，它不涵盖位于 `auth.example.com` 的重定向 URI。同时使用两者的一种方法是将主页放在 `example.com` 上，该域名涵盖这两者。
* **您的组织已经证明的域名：** 您的组织可能之前已经证明过某个域名，例如为另一个应用证明的。如果该域名涵盖您的主页以及每个非本地重定向 URI，对话框会跳过 **Prove your domain**。此时重定向 URI 不必位于主页的域名上。例如，在已证明 `example.com` 的情况下，主页和重定向 URI 可以位于其不同的子域名上。

## 审核

审核结束后，Console 中的应用页面会显示结果：

* **Anthropic 批准该应用：** 应用上线。随后 **Who can link** 会显示 **Any Claude user**，无需您执行任何操作，并且 Console 会隐藏 **Test users** 列表。
* **Anthropic 未批准该应用：** 应用的状态会变为 **Not approved**。

在审核期间以及出现 **Not approved** 结果之后，您的测试用户会保留其关联，您仍然可以添加和移除测试用户。

### 如果应用未获批准

您无法自行再次发布该应用。如需询问该决定，请[联系支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)。您仍然可以[更改应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#change-an-app-that-isnt-live)，但更改不会触发新的审核。

## 更改已上线的应用

更改已上线应用的名称、徽标、主页或重定向 URI 会使应用重新进入审核。在 Anthropic 批准更改之前，只有您的测试用户可以将其组织关联到该应用。请参阅[在 Anthropic 批准更改之前](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#until-anthropic-approves-the-change)。其他更改（例如权限或客户端身份验证）不会使应用重新进入审核。

按照以下步骤更改名称、主页或重定向 URI：

1. 在应用的 **Appearance** 标签页（名称和主页）或 **Settings** 标签页（重定向 URI）上更改应用。

2. 点击 **Save changes**。在保存任何内容之前会打开一个对话框。

3. 如果对话框显示 **Prove your domain**，请证明其列出的每个域名：

   1. 为该域名添加 DNS TXT 记录。然后点击 **Check**。如果域名仍显示 **Not proven yet**，请参阅[如果您无法证明域名](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#if-you-cant-prove-the-domain)。
   2. 当域名显示 **Proven** 时，如果还剩其他域名，请点击 **Next domain**。否则，请点击 **Continue**。

4. 点击 **Save and send to review**。

随后 Console 会保存更改，更改无需等待审核即可生效。在 Anthropic 审核更改期间，应用的状态为 **In review**。

要更改徽标，请在 **Appearance** 标签页上选择或移除徽标。对话框会立即打开。点击 **Send to review**。如果您取消，徽标不会更改。

要在应用状态为 **In review** 或 **Not approved** 时再次更改应用，请参阅[更改尚未上线的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#change-an-app-that-isnt-live)。

### Console 何时要求您证明域名

只有在您更改了主页或某个重定向 URI 时，Console 才会要求。它还会检查您未更改的主页和重定向 URI，因此对话框也可能列出它们的域名。它只会列出您组织的任何已证明域名都未涵盖的域名。它不会检查本地重定向 URI，但对本地重定向 URI 的更改仍会使应用重新进入审核。

更改后，只要您的组织证明了每个域名，重定向 URI 就可以位于主页域名以外的域名上。

### 在 Anthropic 批准更改之前

从您将更改送审时起，以下规则适用：

* **已经关联的客户：** 他们的组织保持关联状态，您计费到他们的请求会继续正常工作。取消关联的客户无法重新关联，除非进行关联的人员在 **Test users** 列表中。
* **新关联：** **Who can link** 设置会恢复为 **Test users only**。Console 会重新显示 **Test users** 列表，其中包含之前在列表中的人员。
* **测试用户：** 您可以添加和移除列表中的人员。移除某人也会终止其在应用上线期间与某个客户组织建立的关联。请参阅[移除测试用户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#remove-a-test-user)。
* **已关联组织的支出上限：** 已关联的组织会保留其当前的[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#spend-caps)设置，即使其上限高于 50 美元或根本没有上限。它所更改的设置必须遵守[应用处于测试阶段时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#while-your-app-is-in-testing)中的规则。

### 更改的审核结束时

审核会以以下两种方式之一结束：

* **Anthropic 批准更改：** 应用再次上线。任何符合条件的客户都可以关联，Console 会隐藏 **Test users** 列表，并且针对尚未上线应用的上限规则不再适用。
* **Anthropic 未批准更改：** 应用的状态会变为 **Not approved**。更改仍然有效，[在 Anthropic 批准更改之前](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#until-anthropic-approves-the-change)中的规则仍然适用。请参阅[如果应用未获批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#if-the-app-isnt-approved)。

## 更改尚未上线的应用

当应用的状态为 **Testing**、**In review** 或 **Not approved** 时，请按照以下步骤操作：

1. 在应用的 **Appearance** 标签页（名称、徽标和主页）或 **Settings** 标签页（重定向 URI）上更改应用。
2. 点击 **Save changes**。徽标更改会在您选择或移除徽标时保存。

不会有对话框要求您确认，更改也不会改变应用的状态。

当您保存对主页或重定向 URI 的更改时，Console 仅在状态为 **In review** 时检查域名。处于测试阶段的应用会在您发布时接受域名检查。在审核期间，更改后应用的主页和每个非本地重定向 URI 都必须位于您组织已证明的域名上。否则，Console 会拒绝该更改，并且不会要求您证明域名。如需迁移到尚未证明的域名，请等待审核结束。

## 发票计费

使用 “invoiced billing”（发票计费）时，Anthropic 会就拥有您应用的组织的 API 用量向其开具发票，该组织不购买预付额度。您的应用仍然从测试阶段开始，测试用户和支出上限的规则仍然适用。在此之后，有三点不同：

* **[发布应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#publish-the-app)：** 在应用页面上，点击 **Publish app**。然后在对话框中点击 **Publish app** 进行确认。Anthropic 不会审核该应用，因此您发布后应用会立即上线。
* **主页和域名：** 您无需证明任何域名，因此对话框会跳过 **Prove your domain**。您的应用无需主页即可发布，并且[发布应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#publish-the-app)下的重定向 URI 规则不适用。仍然必须先设置好客户端身份验证。
* **[更改已上线的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/app-review#change-a-live-app)：** 当您点击 **Save changes** 或选择或移除徽标时，Console 会保存更改。它不会要求您证明域名或进行确认，之后也不会有审核。
