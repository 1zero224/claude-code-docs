---
title: Max、Ultra 和 Team 计划的 API 额度
url: https://platform.claude.com/docs/zh-CN/about-claude/api-credits-for-subscribers
description: 领取 Claude Max、Ultra 和 Team 计划中包含的每月 Claude Platform 额度，并了解这些额度的适用范围。
---

Claude Max、Ultra 和 Team 计划包含每月的 Claude API 额度。要领取这些额度，您需要将一个 [Claude Console](https://platform.claude.com/) 组织关联到您的计划。之后，额度会在每个计费周期（年度计划则为每月）发放到该组织中，您可以使用它们来构建和运行您自己的应用程序和智能体。您无需在 Claude Platform 中添加付款方式。

<Note>
  Claude Platform on AWS、Amazon Bedrock、Google Cloud 和 Microsoft Foundry：这些额度仅适用于 Claude Console 中的 Claude API，不能在其他平台上使用。
</Note>

## 概要

| 主题                       | 详情                                                             |
| ------------------------ | -------------------------------------------------------------- |
| **符合条件的计划**              | Max、Ultra 和 Team，包括折扣版 Team 计划                                 |
| **每月金额**                 | 100 美元（Max）、200 美元（Ultra）、最高 500 美元（Team，合并计算\*）               |
| **适用范围**                 | Claude API、Claude Managed Agents、Claude Agent SDK 和 playground |
| **不适用范围**                | Claude Code、Claude 应用中的额外用量                                    |
| **刷新**                   | 每个计费周期（年度计划则为每月）                                               |
| **到期**                   | 每个计费周期结束时（年度计划则为每月），不可结转                                       |
| **Claude Platform 付款方式** | 无需                                                             |

## API 额度金额

| 计划                     | 每月额度                                                  |
| ---------------------- | ----------------------------------------------------- |
| Max                    | 100 美元                                                |
| Ultra                  | 200 美元                                                |
| Team，Standard 席位       | 每个席位 20 美元\*                                          |
| Team，Premium 席位        | 每个席位 100 美元\*                                         |
| 折扣版 Team 计划（非营利组织、科学家） | 与 Team 相同：每个 Standard 席位 20 美元，每个 Premium 席位 100 美元\* |

\*在 Team 计划中，所有席位的额度会合并为一个每月余额，上限为 500 美元。每月额度（包括第一个月）均根据该计费月开始时您计划中的席位数计算。例如，一个拥有三个 Standard 席位和两个 Premium 席位的团队每月可获得 260 美元。再增加一个 Premium 席位后，下个月的额度将提高到 360 美元。

## 资格条件

* 您的符合条件的计划必须处于有效状态且信誉良好。
* 新订阅用户在使用符合条件的计划满 7 天后即可领取额度。
* 通过 Claude for iOS 或 Claude for Android 购买的计划也符合条件。请在网页浏览器中通过 [claude.ai](https://claude.ai) 领取您的额度。
* Free、Pro 和 Enterprise 计划不符合条件。

## 领取您的额度

### 开始之前

您需要同时具备以下两个角色：

| 位置                    | 所需角色                                |
| --------------------- | ----------------------------------- |
| **Claude 计划**         | Max 或 Ultra：订阅者<br />Team：主要所有者、所有者 |
| **Claude Console 组织** | 所有者、管理员或账单                          |

如果您还没有 Claude Console 组织，请先在 [platform.claude.com](https://platform.claude.com/) 上的 Console 中创建一个，或在领取流程中创建。

### 步骤

<Steps>
  <Step title="打开账单设置">
    在 claude.ai 上，如果您使用的是 Max 或 Ultra 计划，请前往**设置 > 账单**；如果您使用的是 Team 计划，请前往**组织设置 > 账单**。
  </Step>

  <Step title="关联 Console 组织">
    在 **API 额度**部分，选择应接收额度的 Console 组织，然后通过关联操作接受[计划条款](https://www.anthropic.com/legal/credit-terms)。
  </Step>

  <Step title="开始构建">
    您的额度会显示在 Console 中[账单](https://platform.claude.com/settings/billing)页面（`platform.claude.com/settings/billing`）的**促销额度**部分。创建一个 API 密钥并发出您的第一个请求。请参阅[入门指南](https://platform.claude.com/docs/zh-CN/get-started)。
  </Step>
</Steps>

<Warning>
  每个计划只能关联一个 Console 组织，每个 Console 组织也只能接收来自一个计划的额度。您无法自行更改已关联的组织，因此请关联您打算用于构建的组织。如需日后更改，请[联系支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)。
</Warning>

不符合条件的订阅者在其设置页面中看不到 API 额度。如果您认为自己符合条件但看不到 API 额度，请[联系支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)。

## 额度的适用范围

这些 API 额度只能在 Claude Platform 上使用。下表列出了适用的产品：

| 产品                                                                                      | 是否适用 |
| --------------------------------------------------------------------------------------- | ---- |
| [Claude API](https://platform.claude.com/docs/zh-CN/api/overview)                       | 是    |
| [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) | 是    |
| [Claude Agent SDK](https://code.claude.com/docs/zh-CN/agent-sdk/overview)               | 是    |
| Claude Console 中的 [Playground](https://platform.claude.com/playground)                  | 是    |
| Claude Code                                                                             | 否    |
| Claude 应用中的额外用量                                                                         | 否    |
| Claude Platform on AWS、Bedrock、Google Cloud、Microsoft Foundry                           | 否    |

## 额度的使用方式

* **发放时间。** 在每个计费周期，额度会在您的计划付款处理完成后不久发放。对于年度计划，额度按月发放。
* **使用顺序。** 系统会先使用计划包含的额度，然后再使用您购买的额度。"Auto-reload"（自动充值）仅基于您购买的余额。当该余额达到您设定的阈值时就会充值，即使计划包含的额度仍有剩余。
* **到期。** 未使用的额度会在每个计费周期结束时（年度计划则为每月）到期，且不可结转。
* **共享。** 关联组织中的每个 API 密钥和工作区都从同一余额中扣除。要限制某个项目的支出，请为其单独创建一个工作区，并设置该工作区的 “spend limit”（[支出限制](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#setting-workspace-limits)）。
* **可见性。** 您的额度会显示在 Console 中[账单](https://platform.claude.com/settings/billing)页面（`platform.claude.com/settings/billing`）的**促销额度**部分，并附有金额和到期日期。
* **您的 Claude 计划。** 额度不会改变您在 Claude 或 Claude Code 中的使用限制。

## 支出限制和速率限制

额度不会改变 “rate limits and spend limits”（[速率限制和支出限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)）的运作方式：

* 使用额度支付的用量会计入您组织的 “monthly spend cap”（[每月支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#spend-limits)）。支出上限在每个日历月的第一天重置，而额度则按您的计费周期（年度计划则为每月）刷新。
* 关联到符合条件计划的组织至少会升级到 Start 层级。如果其每月额度超过 200 美元，则至少会升级到 Build 层级。
* 除此之外，领取额度不会改变您的 “usage tier”（[使用层级](https://platform.claude.com/docs/zh-CN/api/rate-limits#about-rate-limits)），这些额度也不会计入升级到更高层级的条件。

### 额度用完时

* **如果组织还有其他额度**，请求将继续使用购买的额度。如果已开启自动充值，系统会向购买的余额中添加额度。
* **如果组织已没有剩余额度**，API 请求将停止，直到您下一次每月 API 额度发放。用量绝不会计入您的 Claude 计划费用。如需继续构建，请在 Console 中的[账单](https://platform.claude.com/settings/billing)页面（`platform.claude.com/settings/billing`）购买额度或开启自动充值。
* **如果您的组织通过 Anthropic 销售团队开具发票**，超出 API 额度的用量将照常计费。

当余额耗尽时，请求会返回：

```text wrap
Your credit balance is too low to access the Anthropic API. Please go to Plans & Billing to upgrade or purchase credits.
```

如果某个组织仅持有这些额度，并发送了这些额度不适用的请求（例如 Claude Code 会话），Claude Code 会显示：

```text wrap
Credit balance too low · Add funds: https://platform.claude.com/settings/billing
```

## 如果您的计划发生变更

| 变更                     | 结果                                                                                     |
| ---------------------- | -------------------------------------------------------------------------------------- |
| 取消、降级到不符合条件的计划或退款      | 您的计划结束后将不再发放新额度。您已有的额度在到期前仍可使用。                                                        |
| 从 Max 升级到 Ultra        | 您会立即获得按比例计算的 API 额度，之后每个计费周期获得 200 美元。如果您通过 Google Play 订阅，200 美元的额度会稍后发放，最迟在您下次续订时到账。 |
| 从 Max 或 Ultra 升级到 Team | 与您的 Max 或 Ultra 计划的关联将终止。Team 计划生效满 7 天后，Team 所有者即可领取团队的额度。                            |
| 添加或移除 Team 席位          | 添加到计划中的席位从下一个计费月的额度开始计算。您移除的席位会保留在计划中直到续订，因此之后的额度可能会降低。额度池的上限仍为 500 美元。                |

## 高效使用您的额度

您的每月成本主要取决于您选择的模型以及应用程序的运行频率。要减少额度消耗：

* 为每项任务选择合适的模型。要了解如何选择，请参阅[选择合适的模型](https://platform.claude.com/docs/zh-CN/about-claude/models/choosing-a-model)。
* 对重复的上下文使用 “prompt caching”（[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)）。
* 对于不需要立即响应的工作，使用 [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)，其费用可降低 50%。
* 查看[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)，并在 Claude Console 的[费用](https://platform.claude.com/cost)页面跟踪支出。

## 相关内容

* [速率限制和支出限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)
* [工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)
* [定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)
* [帮助中心：Max、Ultra 和 Team 计划的 API 额度](https://support.claude.com/en/articles/17154008)
