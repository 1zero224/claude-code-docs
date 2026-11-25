---
title: 测试并发布您的应用
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish
description: 添加测试用户，发布您的 Linked billing 应用以便任何符合条件的客户都可以关联它，并在客户依赖该应用后对其进行更改。
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

新的 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing) 应用从测试阶段开始，在此阶段只有您指定的[测试用户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#manage-test-users)才能将其组织关联到该应用。要向您的客户开放该应用，您需要证明您的域名并[发布应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#publish-the-app)，然后由 Anthropic 对其进行审核。本页还介绍了[测试期间支出上限的工作方式](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#caps-while-testing)，以及如何更改或删除应用。

Claude Console 中的 [OAuth apps](https://platform.claude.com/settings/oauth-apps) 页面会显示每个应用的状态：

| 状态 | 含义 |
| ----- | ----- |
| **Testing** | 您尚未发布该应用 |
| **In review** | Anthropic 正在审核该应用或对其的更改 |
| **Not approved** | Anthropic 已审核该应用或更改，但未予批准 |
| **Active** | 应用已上线：任何[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#eligible-customers)都可以关联它 |

如果 Anthropic 以发票方式向您的组织收取 API 使用费用，您的应用将跳过审核：请参阅[如果您的组织采用发票计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#with-invoiced-billing)。

## 开始之前

您需要在组织中拥有 Admin 角色，并拥有一个已[为 Linked billing 完成设置](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app)的应用。

## 管理测试用户

在您的应用上线之前，其 **Who can link** 设置显示为 **Test users only**，且无法更改。只有其 **Test users** 列表中的人员才能将其组织关联到该应用，每个应用都有自己的列表。批准页面会拒绝其他任何人；如果他们返回您的应用，您的应用会收到 [`access_denied`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#access-denied)。在应用未上线期间进行关联的每个人都必须设置不超过 50 美元的每月支出上限：请参阅[测试期间的上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#caps-while-testing)。

### 添加测试用户

测试使用真实资金：与任何客户一样，Anthropic 会就您计费给每个测试用户组织的请求向该组织收费。测试用户需[关联一个组织](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer)，且其在该组织中拥有 Admin 或 Billing 角色。该组织不能是拥有该应用的组织。要亲自测试应用，请添加您自己的邮箱地址，并关联您名下的第二个 Claude 组织。

1. 在 Console 的 [OAuth apps](https://platform.claude.com/settings/oauth-apps) 页面中打开该应用。
2. 在 **Settings** 选项卡上，找到 **Who can link** 下的 **Test users** 列表。
3. 输入测试用户 Claude 账户的电子邮件地址，然后点击 **Add**。对每个测试用户重复此操作。
4. 点击 **Save changes**。

该列表遵循以下规则：

* 最多可容纳 10 人。
* 地址必须与账户地址完全匹配，但不区分大小写。`name+test@example.com` 与 `name@example.com` 是不同的地址。

### 移除测试用户

<Warning>
  移除测试用户会终止其批准的所有与您应用的连接。
</Warning>

在已上线应用的更改审核期间，这可能包括与客户组织的连接。向这些组织计费的新请求通常会在约 10 秒内停止，并以 [`user_profile_billing_not_authorized`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-billing-not-authorized) 被拒绝。使用已终止连接的访问令牌发出的请求则可能收到 [`401 authentication_error`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#authentication-error)。要移除某人：

1. 如果 **Settings** 选项卡有未保存的更改，请点击 **Save changes** 或 **Discard**。在您完成此操作之前，Console 不允许您移除已保存的测试用户。
2. 从 **Test users** 列表中移除该人员的地址。
3. 当 Console 询问时，点击 **Remove**。您无需点击 **Save changes**。

## 发布应用

发布时可能会要求您添加一条 DNS 记录，以证明您控制应用的域名。DNS 更改最多可能需要 48 小时才能生效，因此请在需要客户关联之前尽早发布。发布之前，请检查以下事项：

* 已设置[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#set-up-client-authentication)。在设置完成之前，您无法发布。
* 应用拥有一个主页（在其 **Appearance** 选项卡上设置），且位于您自己的域名上。托管服务的共享域名不符合要求。
* 您可以更改主页所在域名的 DNS 记录。
* 每个重定向 URI 都位于主页的主机名（不含前导 `www.`）或其子域名上。有关例外情况，请参阅[域名规则](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#domain-rules)。

要发布应用：

1. 保存应用各选项卡上所有未保存的更改，因为在您完成此操作之前，Console 不允许您发布。然后，在应用页面上点击 **Publish app**。
2. 如果对话框显示 **Prove your domain**，请在您管理域名 DNS 的位置添加对话框提供的 DNS TXT 记录。然后，点击 **Check**。如果域名未显示 **Proven**，请参阅[如果您无法证明域名](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#if-you-cant-prove-the-domain)。当其显示 **Proven** 时，点击 **Continue**。
3. 如果对话框显示接受 Linked Billing Terms 的复选框，请勾选它。这些条款即 [Linked Billing Beta Terms](https://www.anthropic.com/legal/linked-billing-terms)。
4. 在对话框中点击 **Publish app**。

此后应用的状态为 **In review**。在审核期间以及出现 **Not approved** 结果之后，您的测试用户会保留其连接，您仍然可以添加和移除测试用户。审核结束后，应用页面会显示结果，Anthropic 通常会通过电子邮件将结果发送给发布该应用的人：

* **Approved：** 应用已上线，其在 OAuth apps 页面上的状态显示为 **Active**。**Who can link** 显示为 **Any Claude user**，Console 会隐藏 **Test users** 列表，测试用户的连接保持不变。
* **Not approved：** 您无法自行再次发布该应用：请参阅[如果应用未获批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#if-the-app-isnt-approved)。

采用发票计费时，Anthropic 不会审核您的应用：请参阅[如果您的组织采用发票计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#with-invoiced-billing)。

## 更改已上线的应用

<Warning>
  更改已上线应用的名称、徽标、主页或重定向 URI 会将新连接限制为仅限测试用户，直到 Anthropic 批准该更改。
</Warning>

在 Anthropic 批准更改之前：

* 已关联的客户保持关联，您计费给他们的请求继续正常工作。
* 取消关联的客户只能通过 **Test users** 列表中的人员重新关联。
* 对上限的更改遵循[测试期间的上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#caps-while-testing)。

如果 Anthropic 拒绝该更改，限制将保持不变：请参阅[如果应用未获批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#if-the-app-isnt-approved)。有关其余规则，请参阅[更改处于审核中或未获批准期间](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#while-a-change-is-in-review)。

更改在 Console 保存后立即生效，因此客户在审核结束前就会看到更改。对应用权限或客户端身份验证的更改不会启动审核；采用[发票计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#with-invoiced-billing)时，任何更改都不会启动审核。

要更改名称、主页或重定向 URI：

1. 在应用的 **Appearance** 选项卡（名称和主页）或 **Settings** 选项卡（重定向 URI）上更改应用。
2. 点击 **Save changes**。在保存任何内容之前会打开一个对话框。
3. 如果对话框显示 **Prove your domain**，请逐一证明其列出的每个域名。对于每个域名，添加其 DNS TXT 记录，点击 **Check**，当其显示 **Proven** 时，点击 **Next domain** 或 **Continue**。有关会列出哪些域名，请参阅[域名规则](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#domain-rules)。
4. 点击 **Save and send**，这会保存更改并将其发送审核。

要更改徽标，请在 **Appearance** 选项卡上上传或移除徽标，然后在打开的对话框中点击 **Send to review**。如果您取消，徽标不会更改。

在 Anthropic 审核更改期间，应用的状态显示为 **In review**。审核结束后，Anthropic 通常会通过电子邮件将结果发送给保存该更改的人。

## 更改处于测试或审核中的应用

当应用状态为 **Testing**、**In review** 或 **Not approved** 时，请按照本节操作。这包括更改正在审核中的已上线应用。在应用的 **Appearance** 选项卡（名称、徽标和主页）或 **Settings** 选项卡（重定向 URI）上更改应用，然后点击 **Save changes**。徽标更改会在您上传或移除徽标时保存。

不会有对话框要求您确认，状态也不会改变。在审核期间，该更改会用您编辑后的应用替换 Anthropic 正在审核的内容。

对于处于测试阶段的应用，Console 仅在您发布时检查已证明的域名。当状态为 **In review** 时，主页和每个非本地重定向 URI 都必须已位于您组织已证明的域名上。否则，Console 会拒绝该更改，且不会要求您证明域名。要迁移到尚未证明的域名，请等待审核结束。

## 删除应用

<Warning>
  删除应用会取消关联所有客户，且无法撤销。
</Warning>

向客户计费的请求将被拒绝，所有 OAuth 令牌都会被撤销，客户端 ID 也将停止工作。Anthropic 不会通过电子邮件通知您的客户，因此请先告知他们。要删除应用：

1. 在应用的 **Settings** 选项卡上，保存或放弃所有未保存的更改。
2. 在 **Danger zone** 下，点击 **Delete app**，并通过 **Delete app** 进行确认。[OAuth apps](https://platform.claude.com/settings/oauth-apps) 页面上该应用所在行的菜单中也有 **Delete**。

要再次提供 Linked billing，请创建一个新应用，并让每个客户关联它。新应用以新的客户端 ID 从测试阶段开始，客户的支出上限不会沿用。

## 规则和特殊情况

本页其余部分介绍发布和更改应用背后的规则，还涵盖诸如无法证明域名之类的情况。

### 域名规则

域名是不含前导 `www.` 的主机名，因此位于 `https://www.example.com` 的主页的域名为 `example.com`。您的域名即您主页的域名。已证明的域名涵盖其自身及其子域名，并保持已证明状态。

* **本地重定向 URI：** 位于 `localhost`、`127.0.0.1` 或 `[::1]` 上的重定向 URI 无需位于主页的域名上。
* **同级子域名：** 位于 `app.example.com` 上的主页不涵盖位于 `auth.example.com` 上的重定向 URI。将主页放在 `example.com` 上即可同时涵盖两者。
* **您的组织已证明的域名：** 如果这些域名涵盖主页和每个非本地重定向 URI，发布时将跳过 **Prove your domain**。此时重定向 URI 无需位于主页的域名上。

更改已上线的应用时，只有在您更改了主页或某个重定向 URI 的情况下，Console 才会要求您证明域名。随后它会检查应用的每个域名（无论是否更改），并列出您组织的已证明域名未涵盖的每一个域名。进行此类更改后，如果您的组织证明了每个域名，重定向 URI 可以位于主页域名以外的域名上。Console 不检查本地重定向 URI，但更改本地重定向 URI 仍会将应用发送审核。

### 如果您无法证明域名

* **域名仍显示 Not proven：** DNS 更改尚未生效。关闭对话框，稍后再回来。如果您之前是在发布，请再次点击 **Publish app**。如果您之前是在更改已上线的应用，请再次进行相同的编辑：在记录过期之前，Console 会为您提供相同的记录。
* **Check 提示记录已过期：** 记录自 Console 首次显示之日起 30 天内有效。点击 **Get a new record** 并添加新记录。
* **对话框显示一条消息而非记录：** 例如，该消息可能提示您的组织已拥有 20 个域名。在该问题解决之前，您无法继续。如需帮助，请联系 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)。

### 如果应用未获批准

您无法自行再次发布该应用，更改也不会启动新的审核。要申请再次审核，请联系 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)。对已上线应用的更改被拒绝后，该更改仍然有效，新连接仍仅限于您的测试用户。

### 更改处于审核中或未获批准期间

从您将已上线应用的更改发送审核起，直到 Anthropic 批准该更改为止：

* **新连接：** **Who can link** 恢复为 **Test users only**。Console 会再次显示 **Test users** 列表，其中包含之前在列表中的人员。
* **测试用户：** 您可以添加和移除人员。
* **已设置的上限：** 已关联的组织保留其上限设置，即使高于 50 美元或未设上限。

审核结束时，会出现以下情况之一：

* **Approved：** 应用再次上线。任何符合条件的客户都可以关联，Console 会隐藏 **Test users** 列表，测试期间的上限规则不再适用。
* **Not approved：** 状态变为 **Not approved**，且这些规则仍然适用。

### 测试期间的上限

客户为您的应用设置的支出上限会限制您每月可向其计费的金额。批准页面将其称为每月支出限额。这些规则适用于进行关联的人员，以及客户在 Console 中的 Admin 和 Billing 成员：

| 上限设置 | 应用处于测试、审核中或未获批准时 | 应用上线时 |
| ----- | ----- | ----- |
| 批准页面上的支出上限 | 必填，不超过 50 美元 | 可选 |
| 批准页面上的金额字段 | 初始值为 50 美元 | 初始为空 |
| **Continue without a limit** | 不提供 | 提供 |
| 在 Console 中移除上限 | 不可行 | 可行 |
| 可设置的最大上限 | 50 美元 | 无实际上限 |

应用上线后，客户在测试期间保存的上限保持不变。客户随后可以提高或移除它。有关上限的其他规则，请参阅[“您的客户会看到什么”中的支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#spend-caps)。

### 如果您的组织采用发票计费

采用发票计费时，Anthropic 以发票方式向您的组织收取 API 使用费用，您的组织无需购买预付额度。您的应用仍从测试阶段开始，测试用户和上限的规则仍然适用。此后，有三点不同：

* 要发布，请点击 **Publish app**，然后在对话框中点击 **Publish app**。Anthropic 不会审核该应用，因此您一发布它就会上线。
* 您无需证明域名，应用发布时也无需主页。重定向 URI 的规则不适用。但仍必须先设置客户端身份验证。
* 对已上线应用的更改会在您点击 **Save changes** 或上传、移除徽标时保存。之后不会进行域名检查、确认或审核。

## 后续步骤

<CardGroup cols={2}>
  <Card title="管理已关联的客户" icon="list" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers">
    查看用量，检查客户是否已关联，以及取消关联或重新关联客户
  </Card>

  <Card title="为每个客户使用单独的凭据" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer">
    将凭据限制为仅用于一个客户，适用于单租户产品和沙盒
  </Card>
</CardGroup>
