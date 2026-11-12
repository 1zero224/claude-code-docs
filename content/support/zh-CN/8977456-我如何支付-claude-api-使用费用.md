# 我如何支付 Claude API 使用费用？

本文介绍了通过 Claude Console 账户使用 Claude API、playground 和 Claude Code 时的计费方式。大多数组织使用预付使用额度支付。与 Anthropic 签订发票协议的组织则按月计费。

## 预付使用额度

Claude API 和 playground 使用通过预付使用额度计费。在使用 API 前购买额度，额度将根据我们当前的**[定价](https://claude.com/pricing#api)**应用于您的使用。购买的额度涵盖 API 访问、playground 使用和 Claude Code。

您只需为成功的 API 调用和已完成的任务付费。失败的请求不收费。

**注意：**如果您的客户端在即将成功的请求中途断开连接或超时，该请求仍会被计费。

如果您的额度用尽，在添加更多额度之前，您将无法调用 API 或使用 playground。

## Max、Ultra 和 Team 计划的每月 API 额度

Claude Max、Ultra 和 Team 计划包括 Claude API 的每月额度。要领取您的额度，您需要将 Claude Console 组织关联到您的计划。您的每月额度在任何已购买的额度之前使用，未使用的额度在每个计费周期结束时过期。每月 API 额度不能用于 Claude Code 或 Claude 应用程序，包括额外使用。

如果您的组织通过 Anthropic 销售团队开具发票，超出额度的使用将按常规计费。详细了解**[Max、Ultra 和 Team 计划的每月 API 额度](https://support.claude.com/en/articles/17154008)**。

## 购买额度

1. 使用管理员或计费角色登录 Console。

2. 转到**计费**页面 (platform.claude.com/settings/billing)。

3. 点击"购买额度"。

4. 输入您要购买的额度金额并确认。

购买的额度立即可用。您的组织的可用额度余额和额度使用情况显示在同一**计费**页面上。

## 设置自动充值

自动充值在您的余额低于您设置的阈值时自动购买额外额度。

1. 转到**[计费页面](https://platform.claude.com/settings/billing)**。

2. 在**额度余额**部分，点击"设置自动充值"。如果自动充值已启用，按钮显示"管理自动充值"。

3. 在**当额度余额达到**中，输入触发购买的余额。

4. 在**将额度余额充值至**中，输入要充值到的余额。

5. 点击"启用"或"更新"（如果您正在更改现有设置）。

**注意：**如果您的余额已经处于或低于您设置的最小值，按钮显示"启用并购买"或"更新并购买"，您将立即被收费以将余额提升到目标金额，加上适用的税费。

要关闭自动充值，点击"管理自动充值"，然后"关闭"，并通过再次点击"关闭"进行确认。

## 额度过期和退款

购买的额度受我们的**[额度条款](https://www.anthropic.com/legal/credit-terms)**约束。额度自购买之日起一年后过期，过期日期无法延长。过期的额度显示在计费页面的**发票历史**中。所有额度购买均不可退款。

在**[Claude API 文档](https://platform.claude.com/docs/en/api/rate-limits)**中详细了解额度使用与速率限制的关系。

## 每月发票

某些组织按月后付而不是预先购买额度。这适用于通过我们销售团队设置了发票协议的 Console 组织。

在每月发票中，我们汇总您在 API 调用、Console 使用和与您账户关联的任何其他服务中的使用情况，并按我们的标准**[按需付费定价](https://claude.com/pricing#api)**计费。在每个日历月末，您将收到来自 Stripe 的发票。在 Stripe 中输入您的付款详情以支付。

您可以在 Claude Console 的**计费**页面 (platform.claude.com/settings/billing) 上查看当前计费周期的费用。

如果您需要自定义速率限制、每月发票或专业支持，请**[联系我们的销售团队](https://claude.com/contact-sales)**。

## 更新您的 Console 付款方式

1. 使用管理员或计费角色登录 Console。

2. 转到计费页面 (platform.claude.com/settings/billing)。

3. 点击您当前付款方式旁边的铅笔图标。

4. 在**更新付款方式**模态框中输入您的新卡详情，然后点击"更新"。