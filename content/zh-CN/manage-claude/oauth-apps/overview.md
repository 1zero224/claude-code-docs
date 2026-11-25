---
title: Claude Platform OAuth
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/overview
description: 从何处开始使用 Claude Platform OAuth 和 Linked billing，以及客户的批准如何传达到您的后端。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 测试期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

Claude Platform OAuth 是客户批准您的产品使用其 Claude 组织的方式。它为 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing) 提供支持，让您的客户可以通过其自己的 Claude 组织支付在您产品中产生的 Claude 用量费用。如果要使用您组织的凭据为您自己的产品调用 Claude，您不需要 OAuth 应用：请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。

要提供 Linked billing，您需要一个 OAuth 应用，以及一个向您的后端颁发令牌的[身份提供商](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/overview#no-client-secret)。您的客户需要一个直接向 Anthropic 付费的 Claude 组织：请参阅[符合条件的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#eligible-customers)。

## 工作原理

客户只需批准您的应用一次。这会创建一个连接，将其 Claude 组织与您的应用关联起来，直到他们或您终止该连接。

您客户公司中的某位用户在您的产品中点击 [Link your Claude organization 按钮](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button)。您的后端将其引导至 `auth.claude.com` 上 Claude 的批准页面。如果该用户在客户的组织中具有Admin 或 Billing 角色，他们就可以批准您的应用，浏览器随后会带着一次性代码返回。您的后端用该代码交换连接的 ID 和 OAuth 令牌。之后，Anthropic 就可以就您的后端代表客户发出的请求向客户的组织计费。

<Frame>
  ![时序图：用户在 auth.claude.com 批准您的应用，然后您的后端交换一次性代码](https://platform.claude.com/docs/zh-CN/images/linked-billing-round-trip.svg)
</Frame>

不具备这两种角色之一的人无法批准：请参阅[您的客户会看到什么](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#approving-the-link)。如果您了解 OAuth，这就是带 PKCE 的授权码流程。

### 您的 OAuth 应用

您的 OAuth 应用是您的产品在 Anthropic 的注册信息。您 Claude 组织的管理员在 Claude Console 中创建该应用，它会为您的产品提供一个客户端 ID。在批准页面上，客户会看到它的名称、徽标以及它所请求的内容。

### 使用身份提供商而非客户端密钥

您的后端发送到 `auth.claude.com` 的每个请求都通过[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)携带来自您身份提供商的令牌。Anthropic 不会为 OAuth 应用颁发客户端密钥。任何 OpenID Connect 颁发者都可以使用，并且我们提供了[常见身份提供商的指南](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#identity-providers)。

## 选择您的路径

从您当前的任务开始。要构建集成，请[设置您的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app)，[关联测试客户并向其计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer)，然后[测试并发布您的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish)。在您的应用上线之前，只有其测试用户可以关联它，因此请先[添加您测试客户的批准人](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#add-test-users)。

<CardGroup cols={2}>
  <Card title="Linked billing" icon="coins" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing">
    判断 Linked billing 是否适合您的产品和客户
  </Card>

  <Card title="为 Linked billing 设置您的应用" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app">
    注册您的应用，然后继续关联客户并向其计费
  </Card>

  <Card title="管理已关联的客户" icon="list" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers">
    为已关联的客户提供支持、检查他们是否仍处于关联状态，或重新关联他们
  </Card>

  <Card title="Linked billing 错误" icon="triangle-alert" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors">
    修复关联或计费返回的错误
  </Card>

  <Card title="您的客户会看到什么" icon="browser" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see">
    面向产品和支持团队：您的客户看到的批准页面、支出上限和电子邮件
  </Card>

  <Card title="OAuth 端点和参数" icon="book" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters">
    查询确切的端点、参数、作用域、OAuth 令牌字段或错误
  </Card>
</CardGroup>
