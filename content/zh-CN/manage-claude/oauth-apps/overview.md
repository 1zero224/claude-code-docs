---
title: Claude Platform OAuth
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/overview
description: 让您的客户从其自己的 Claude 组织支付在您产品中产生的 Claude 用量费用。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 测试期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

Claude Platform OAuth 让您的用户可以将其在您应用内产生的用量计费到他们的 Claude 组织。

## 您可以构建什么

### 让客户为自己的 Claude 用量付费

当您的产品调用 Claude 时，用量可以计费到您客户自己的 Claude 组织。您的用户点击 **Link your Claude organization** 按钮，登录 Claude，并批准您的应用。对于 Linked billing，用户必须在其 Claude 组织中具有管理员或账单角色。

之后：

* 您可以在每个 Claude API 请求上选择由客户付费还是由您付费。
* 如果您标记为由客户付费的请求无法向客户计费，Claude API 会拒绝该请求，而不是向您计费。
* 客户可以随时取消关联您的应用。
* 您只能就客户指示您发出的请求向其计费，绝不能就您自己的用量向其计费。

您负责为您的客户提供支持：Anthropic 提供批准页面以及客户用于设置支出上限或取消关联您的应用的 Claude Console 页面，有关您的应用的所有其他问题都由您来处理。

Linked billing 适用于其 Claude 组织直接向 Anthropic 支付 API 用量费用的客户。它不适用于通过云市场计费的组织。请参阅 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections)。

## 工作原理

Linked billing 遵循一个简短的往返流程。用户离开您的产品，前往 Claude 的批准页面一次，在那里批准您的应用，然后返回：

<Frame>
  ![Sequence diagram（时序图）：用户在 auth.claude.com 批准您的应用，然后您的后端交换一次性代码](https://platform.claude.com/docs/zh-CN/images/oauth-apps-billing-round-trip.svg)
</Frame>

批准页面会显示您应用的名称以及您的应用所请求的内容，让用户清楚自己正在批准什么。

<Note>
  如果您以前使用过 OAuth：Claude Platform OAuth 使用带 PKCE 的 OAuth 2.0 授权码流程，由 Anthropic 作为授权服务器。
</Note>

## 一次性 OAuth 应用设置

Linked billing 从 Claude Console 中的以下一次性设置步骤开始。[Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#setup-process) 指南会引导您完成这些步骤。

这些步骤需要您在 Claude 组织中具有管理员角色。

1. **创建应用。** 在 Claude Console 中创建一个 OAuth 应用并复制其客户端 ID。该应用包含用户在批准页面上看到的名称，以及他们返回到的您产品中的地址。
2. **设置客户端身份验证。** 您的后端不使用客户端密钥，而是使用来自您自己的身份提供商（例如 AWS、Google Cloud、Microsoft Entra ID 或 Kubernetes）的短期令牌来证明请求来自您的应用。您可以在应用的 **Client authentication** 选项卡上选择身份提供商以及要接受其哪些令牌。
3. **设置您后端的身份。** 使用[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)创建一个服务账户和第二条联合规则，并将该服务账户选为应用的操作员服务账户。

## 后续步骤

Linked billing 指南会引导您完成设置，以及您的第一个已关联客户。

<CardGroup cols={3}>
  <Card title="Linked billing" icon="coins" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections">
    让您的客户从其自己的 Claude 组织支付在您产品中产生的 Claude 用量费用
  </Card>

  <Card title="OAuth 参考" icon="book" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference">
    端点、参数、令牌响应、有效期和错误
  </Card>
</CardGroup>
