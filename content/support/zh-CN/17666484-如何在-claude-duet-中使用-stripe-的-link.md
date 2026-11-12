# 如何在 Claude Duet 中使用 Stripe 的 Link

连接 Stripe 的数字钱包 Link 到 Claude Duet，这样它就可以为您批准的购买付款。

Link 在 Claude Duet 中可用，这是美国 Pro、Max 和 Ultra 计划的研究预览版，需要美国支付方式。目前在 Claude、Cowork 或 Claude Code 中不可用。

Link 与 Stripe 连接器分开，后者允许 Claude 在您的 Stripe 账户中工作。您可以单独使用其中任何一个。

## 您可以做什么

- **批准每笔购买。**Claude Duet 的设计目的是在每次购买前询问您，并且在您批准最高金额之前不会提交 Link 付款。当您批准时，Link 可能会在您选择的支付方式上放置临时授权。只有在商家完成购买时，才会完成最终扣款。

- **使用一次性卡付款。**每次购买都使用仅适用于该购买的一次性卡。

## 如何连接

Link 未在连接器目录中列出。您在 Claude Duet 中连接它。

1. 在 Claude Duet 中选择您的头像。

2. 选择连接付款的选项，然后选择 Link。

3. 登录您的 Link 账户并批准访问权限。

当您要求需要付款的内容时，Claude Duet 也会建议连接 Link。

Claude Duet 可以看到您保存的支付方式的最后四位数字和过期日期，以及您的账单地址。它无法访问您的完整卡号或银行账户号码。

## 如何使用 Link 付款

1. 向 Claude Duet 要求您需要的内容，例如"为办公室打印机重新订购墨水。"

2. Claude Duet 找到该商品并要求您批准购买。

3. 使用 Link 移动应用或 app.link.com 创建或登录您的 Link 账户，然后批准您的支出请求。

4. Claude Duet 使用限定于商家和金额的一次性卡付款。

## 当前限制

- Link 仅在美国的 Claude Duet 中的 Pro、Max 和 Ultra 计划中可用。

- 需要美国支付方式。

- 在您在 Link 应用中批准金额之前，Claude Duet 无法使用 Link 付款。

- 在您批准支出请求后，企业可能会拒绝或阻止结账。

- 失败的购买可能会在选定的资金来源上暂时显示为待处理。

- Claude Duet 无法使用 Link 进行定期付款。

- 某些网站可能会阻止 Claude Duet 付款。如果发生这种情况，您可能需要自己完成购买。

---

## 常见问题

### Claude Duet 可以在不询问我的情况下购买东西吗？

Claude Duet 的设计目的是在每次购买前询问，在您在 Link 应用中批准金额之前，它无法使用 Link 付款。

### 为什么我在连接器目录中找不到 Link？

Link 内置于 Claude Duet 中，因此您在那里连接它。

### Link 与 Stripe 连接器相同吗？

不是。Stripe 连接器允许 Claude 在您的 Stripe 账户中工作，例如检查未支付的发票。Link 用于购买付款。您需要分别连接它们。

### 关于费用、退款或购买问题，我应该联系谁？

- 如有订单状态、交付、取消、退货、损坏或不正确的产品以及产品退款问题，请联系商店。

- 如有 Link 账户问题、批准问题、支付方式问题、重复或不正确的付款金额、无法识别的 Link 付款或失败购买后仍待处理的付款授权，请联系 **[Link 支持](https://support.link.com/topics/link-agent-wallet)**。

- 如果 Claude Duet 选择了与您要求不同的产品、商店、数量或选项，或者 Claude Duet 本身无法正常工作，请联系 **[我们的支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)**。您可以直接在聊天中告诉 Claude Duet 发生了什么，它会将您的报告转发给我们的团队。

- 如果您不认识某笔购买，请拒绝任何待处理的批准请求，从 Claude Duet 断开 Link 连接，并立即联系 Link 支持。

### 我如何断开 Link 连接？

1. 在 Claude Duet 中，选择您的 Claude Duet 头像。

2. 选择付款，然后选择 Link。

3. 选择"断开连接"。