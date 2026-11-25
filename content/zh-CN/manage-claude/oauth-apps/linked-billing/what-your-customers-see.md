---
title: 您的客户会看到什么
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see
description: 向您的客户解释 Linked billing：批准页面、他们为您的应用设置的支出上限、连接如何结束，以及他们会收到的电子邮件。
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

借助 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing)，您的客户在 Anthropic 的页面上（而不是在您的产品中）批准您的应用、为其设置任何[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#spend-caps)，以及取消关联。Anthropic 还会向他们发送电子邮件。您的产品和支持团队可以使用本页面向客户解释 Linked billing。您的工程师可以使用本页面为您的应用无法看到的内容做好规划。

## 工作原理

例如，Acme 的一位 Admin 在 Claude 的批准页面上批准您的应用，并为其设置 200 美元的每月支出上限。之后，Acme 可以在其 [Linked platforms](https://platform.claude.com/linked-platforms) 页面上更改上限或取消关联您的应用。在建立连接时以及 Acme 取消关联您的应用时，Anthropic 会向 Acme 的 Admin 和 Billing 成员发送电子邮件。当支出达到上限的 80% 时，也会向他们发送电子邮件。

### 批准您的应用

批准页面位于 `auth.claude.com`。它使用浏览器中已登录的 Claude 账户，并且只提供该账户的 Claude 组织。进行批准的人需要在其关联的组织中拥有 Admin 或 Billing 角色。每个界面都会显示您的应用名称，并有一个 **Cancel** 按钮：

| 界面 | 用户的操作 |
| ----- | ----- |
| **Select an organization** | 选择要关联的 Claude 组织（仅在有多个选项时出现） |
| **How billing works** | 阅读说明：该组织按自己的费率为此用量付费，然后点击 **Continue** |
| **Set a monthly spend limit** | 以整数美元输入您的应用的支出上限，或点击 **Continue without a limit** 并确认 |
| 最后一个界面 | 查看您的应用可以执行的操作（包括上限），然后点击 **Link and authorize** |

在 **Set a monthly spend limit** 界面上，金额字段初始为空。**Continue without a limit** 不会保存新的上限。客户之前为您的应用保存的上限仍然适用。如果没有，则适用其[默认上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#spend-caps)。在您的应用上线之前，适用其他规则：请参阅[测试期间的上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#caps-while-testing)。

点击 **Link and authorize** 后，浏览器会返回您的应用，不显示确认界面。

在以下情况下，批准流程会有所不同：

* **登录了错误的账户：** 例如，用户可能使用个人账户而不是工作账户登录。页面会显示已登录的账户，并允许用户切换。您的应用无法选择账户：OpenID Connect 参数 `prompt=select_account` 和 `login_hint` 不会改变它。页面也可能要求已登录的用户重新登录。
* **该账户不属于任何 Claude Platform 组织：** 页面会提供 **Set up Claude Platform**，它会打开 Claude Console 以创建组织，然后返回批准流程。在该组织设置好计费之前，向其计费的请求会收到 [`user_profile_not_billable`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-not-billable)。
* **用户没有可以关联的组织：** 他们在任何[有资格关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#eligible-customers)的 Claude 组织中都没有 Admin 或 Billing 角色，因此页面没有可提供的组织。如果他们返回您的应用，您的应用会收到 `access_denied` 错误。有关该错误的其他原因，请参阅 [Linked billing 错误中的 `access_denied`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#access-denied)。
* **用户点击 Cancel：** 不会建立连接。浏览器返回后，您的应用会收到 `access_denied`。

### 关联之后

连接属于客户的整个 Claude 组织，而不属于批准它的人，因此即使该人离开，连接也会保留。客户在其 Claude Console 中的 [Linked platforms](https://platform.claude.com/linked-platforms) 页面上管理连接。在该页面上，拥有 Admin 或 Billing 角色的成员可以查看您的应用本月产生的费用。他们还可以为其设置支出上限。要取消关联，他们需点击您的应用，然后点击 **Unlink**，并在对话框中确认。

连接会以以下方式结束，其中一些方式会向客户的 Admin 和 Billing 成员发送电子邮件：

| 连接如何结束 | 是否向客户发送电子邮件 |
| ----- | ----- |
| 客户取消关联您的应用 | 是，但不会指明是谁取消了关联 |
| 您的后端通过 API 结束连接 | 是 |
| 您组织的 Admin 在 Console 中取消关联该客户 | 否 |
| 您的后端在刷新令牌首次使用超过 30 秒后[重复使用该刷新令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails) | 否 |
| 您从应用的 **Test users** 列表中移除了批准该连接的测试用户 | 否 |
| 客户使用其他作用域再次批准，这会用新连接替换原连接，因此客户保持关联状态 | 否 |
| 您[删除您的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#delete-the-app)，这会一次性结束所有连接 | 否 |

如果您的应用仍然存在，且没有新连接替换已结束的连接，向该客户计费的请求通常会在约 10 秒内被拒绝。错误为 [`user_profile_billing_not_authorized`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-billing-not-authorized)。使用已结束连接的访问令牌发出的请求则可能收到 [`401 authentication_error`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#authentication-error)。在您的 User profiles 页面上，他们的配置文件会显示为 **Unlinked**，您可以[使用同一配置文件再次关联他们](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

### 支出上限

您的应用的 “spend cap”（支出上限）是一个以美元计的金额，用于限制您的应用每月可以向客户计费的金额。批准页面将其标记为每月支出限额。另外两项设置很容易与之混淆：

* **组织的 Monthly spend limit：** 它限制该组织的所有用量，在 [Spend limits](https://platform.claude.com/settings/billing#spend-limits) 下设置。您的应用的上限可以高于它。
* **客户的默认上限：** 它完整地适用于每个没有自己上限的已关联应用。

您的应用的上限或组织的限额都可能拒绝您向该客户计费的请求，并返回 [`linked_organization_spend_limit_reached`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#linked-organization-spend-limit-reached) 错误。缺少[可用于支付的额度](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#which-credits-can-pay)也会如此。该错误不会说明具体是哪一种原因。您的应用的上限按以下方式运作：

* 拒绝会持续到下个月第一天的 00:00 UTC，或直到客户提高或移除上限。
* 上限按标价计算令牌费用和网络搜索费用，不计任何协商折扣。因此，客户的账单可能与上限所计算的金额不同，享有折扣的客户可能会在账单达到该金额之前就达到上限。
* 跨越上限的那个请求仍会被处理，已经在运行的请求也会完成。因此，总额最终可能会超出上限，同时运行的请求越多，超出越多。
* 对上限的更改大约需要一分钟才能生效。最小上限为 0 美元，这会拒绝所有请求。Console 接受整数美元，并且在您的应用上线后没有实际的最大值。
* 在达到上限的 80% 时以及达到上限时，Anthropic 会向客户的 Admin 和 Billing 成员发送电子邮件，0 美元上限除外。

在 Linked platforms 页面上，拥有 Admin 或 Billing 角色的成员可以管理上限：

* **设置或更改您的应用的上限：** 点击您的应用，然后点击 **Set spend cap** 或 **Edit spend cap**。
* **设置默认上限：** 在 **Default spend cap** 下，点击 **Set default** 或 **Edit**。
* **移除您的应用的上限：** 点击您的应用，点击 **Edit spend cap**，然后点击 **Remove cap**。之后您的应用将使用默认上限或没有上限。

### 电子邮件

Anthropic 会在以下时间向客户发送电子邮件：

* 关联完成时
* 连接以某种[会发送电子邮件的方式](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#after-linking)结束时
* [达到上限的 80% 时以及达到上限时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#spend-caps)

关于关联和取消关联的电子邮件会链接到客户的 Linked platforms 页面。有些组织完全不会收到电子邮件。关联是否发送电子邮件取决于授权码交换：

| 事件 | 是否向客户发送电子邮件 |
| ----- | ----- |
| 客户批准后，您的后端交换了授权码 | 是 |
| 客户批准了，但您的后端从未交换授权码 | 否 |
| 客户在仍处于关联状态时再次批准 | 否 |

## 决定如何让客户了解情况

您的应用无法看到客户所看到的部分内容，因此请规划您的产品要告诉客户什么：

* **取消关联：** 客户取消关联时，Anthropic 不会通知您：您不会收到 webhook 事件，也不会收到电子邮件。请[决定检查频率](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#decide-how-often-to-check-link-status)，以确认客户是否仍处于关联状态。
* **上限：** 您无法看到客户的上限，也无法看到他们已使用了多少。当请求因 `linked_organization_spend_limit_reached` 被拒绝时，请让客户检查他们为您的应用设置的上限、其组织的 **Monthly spend limit** 以及他们的额度。
* **电子邮件：** Anthropic 不保证电子邮件一定会被发送或送达。每当您取消关联客户时，请自行告知客户，并且不要依赖电子邮件来告诉他们已完成关联。

## 后续步骤

<CardGroup cols={2}>
  <Card title="Link your Claude organization 按钮" icon="code" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button">
    该按钮的 HTML、CSS、颜色和徽标
  </Card>

  <Card title="关联客户并向其计费" icon="link" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer">
    关联客户并将请求计费给他们，从测试客户开始
  </Card>
</CardGroup>
