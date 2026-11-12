---
title: 速率限制
url: https://platform.claude.com/docs/zh-CN/api/rate-limits
description: 为了减少滥用并管理 API 的容量，我们对组织可以使用 Claude API 的程度设置了限制。
---

<Note>
  **[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)：** 本页面上的速率限制适用于 Claude Platform on AWS，但计费和限制管理方式有所不同。计费通过 AWS Marketplace 进行（而非购买 Anthropic 额度）。Claude Platform on AWS 上的组织会被置于 Start 层级，并可随着已支付的 AWS Marketplace 发票历史的积累自动升级到更高层级。如需申请更高的限制，请联系您的 Anthropic 客户代表或 [Anthropic 支持](https://support.claude.com)；**申请提升等级**流程不可用。Claude Platform on AWS 不支持按工作区配置速率限制，也不支持 [fast mode](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)（快速模式）。有关详细信息，请参阅 [Claude Platform on AWS 上的速率限制和配额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#rate-limits-and-quotas)。
</Note>

限制分为两种类型：

1. **支出限制**（spend limits）设定了组织每月因 API 使用而产生的最高费用。
2. **速率限制**（rate limits）设定了组织在规定时间段内可以发出的 API 请求的最大数量。

API 在组织级别强制执行服务配置的限制，但您也可以为组织的工作区设置用户可配置的限制。

## 关于速率限制

* 限制旨在防止 API 滥用，同时尽量减少对常见客户使用模式的影响。
* 限制由**使用层级**（usage tier）定义。组织会根据使用历史和账户状况自动被置于某一层级，并可随着使用 API 的时间推移升至更高层级。
* 新组织和使用历史有限的组织可能会从 Evaluation（评估）层级开始，在建立账户历史期间，其限制低于本页面所示的标准限制。这些起始限制是 Anthropic 防止欺诈和滥用的措施之一，并会随着您的组织积累使用历史而自动提高。
* 限制在组织级别设置。您可以在 [Claude Console](https://platform.claude.com/) 的[速率限制](https://platform.claude.com/usage/limits)页面（`platform.claude.com/usage/limits`）上查看组织的层级和当前限制。
* 您可能会在更短的时间间隔内触及速率限制。例如，每分钟 60 个请求（RPM）的速率可能会按每秒 1 个请求来执行。短时间的请求突发可能会超出限制并触发速率限制错误。
* 以下限制是每个层级的标准限制。如果您需要更高的限制，请参阅[申请更高的限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#requesting-higher-limits)。
* API 使用[令牌桶算法](https://en.wikipedia.org/wiki/Token_bucket)（token bucket algorithm）进行速率限制。这意味着您的容量会持续补充直至最大限制，而不是在固定间隔重置。
* 此处描述的所有限制均代表允许的最大使用量，而非保证的最低值。这些限制旨在减少无意的超支，并确保资源在用户之间公平分配。

## 支出限制

<Note>
  **[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)：** 同样的每月支出上限适用，请求在达到上限时也会以同样的方式停止。计费和层级提升的运作方式不同；请参阅 [Claude Platform on AWS 上的支出限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#spend-limits)。
</Note>

Start、Build 和 Scale 层级各自设有每月支出上限。该上限即您的组织每个日历月在 API 上可支出的最高金额。您可以在 [Claude Console 中的支出限制](https://platform.claude.com/settings/spend-limits)（`platform.claude.com/settings/spend-limits`）下查看组织的每月支出上限并设置您自己的限制。

| 使用层级  | 每月支出上限       |
| ----- | ------------ |
| Start | $500 USD     |
| Build | $1,000 USD   |
| Scale | $200,000 USD |

Custom（自定义）层级的组织没有每月支出上限；其限制由客户团队协商安排。

### 达到支出上限

一旦达到您所在层级的支出上限，API 使用将暂停，直到下个月第一天的 UTC 时间 00:00，除非您提前申请更高的限制。在使用暂停期间，API 请求会返回 HTTP 429：

```json
{
  "type": "error",
  "error": {
    "type": "rate_limit_error",
    "message": "You have reached your API usage limits: your organization has crossed its monthly API usage threshold, set based on your organization's API tier. You will regain access on 2026-09-01 at 00:00 UTC.",
    "details": { "error_code": "enforced_spend_limit_reached" }
  },
  "request_id": "req_018EeWyXxfu5pfWkrYcMdjWG"
}
```

* 错误类型为 `rate_limit_error`，与速率限制相同，但响应中没有 `retry-after` 标头。在访问恢复之前，重试（包括 SDK 的自动重试）都会失败。
* 在 Messages API 上，`error.details.error_code` 为 `enforced_spend_limit_reached`。可以用它将此响应与速率限制区分开来。
* 升级到更高层级可恢复访问；请参阅[申请更高的限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#requesting-higher-limits)。

### 设置您自己的支出限制

您还可以设置低于所在层级上限的自定义支出限制以控制成本：

<Steps>
  <Step title="前往您的支出限制">
    前往 [Claude Console 中的支出限制](https://platform.claude.com/settings/spend-limits)（`platform.claude.com/settings/spend-limits`）。
  </Step>

  <Step title="打开支出限制编辑器">
    在您组织的**每月支出限额**旁边，点击**调整限额**或**设置限额**。
  </Step>

  <Step title="调整您的支出限制">
    输入新值。您的支出限制不能超过当前层级的上限。
  </Step>
</Steps>

当使用量达到您设置的支出限制时，请求返回 HTTP 400，错误类型为 `invalid_request_error`。消息以 `You have reached your specified API usage limits` 开头（对于工作区限制则为 `You have reached your specified workspace API usage limits`），并说明何时恢复访问。提高或移除该限制可更早恢复访问。

[Claude Code 工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#claude-code-workspace)上的限制会单独检查：超出该工作区限制的 Claude Code 请求则可能会收到带有 `retry-after` 标头的 429。

## 速率限制

Messages API 的速率限制按每个模型类别的每分钟请求数（RPM）、每分钟输入令牌数（ITPM）和每分钟输出令牌数（OTPM）来衡量。 如果您超出任何速率限制，您将收到一个 [429 错误](https://platform.claude.com/docs/zh-CN/api/errors)，说明超出了哪个速率限制。当响应包含 `retry-after` 标头时，请等待相应的秒数后再重试。

<Note>
  如果您的组织使用量急剧增加，您也可能因 API 的加速限制而遇到 429 错误。为避免触及加速限制，请逐步增加流量并保持一致的使用模式。
</Note>

### 缓存感知的 ITPM

许多 API 提供商使用组合的"每分钟令牌数"（TPM）限制，可能包括所有令牌，无论是否缓存、输入还是输出。**Claude 的默认 ITPM 速率限制只计算未缓存的输入令牌。**&#x8FD9;使得速率限制实际上比最初看起来更高。

ITPM 速率限制在每个请求开始时进行估算，并在请求过程中调整估算值，以反映实际使用的输入令牌数。

以下是计入 ITPM 的内容：

* `input_tokens`（最后一个缓存断点之后的令牌）✓ **计入 ITPM**
* `cache_creation_input_tokens`（正在写入缓存的令牌）✓ **计入 ITPM**
* `cache_read_input_tokens`（从缓存读取的令牌）✗ **不计入 ITPM**

<Note>
  `input_tokens` 字段仅表示**最后一个缓存断点之后**出现的令牌，而不是请求中的所有输入令牌。要计算总输入令牌数：

  ```text wrap
  total_input_tokens = cache_read_input_tokens + cache_creation_input_tokens + input_tokens
  ```

  这意味着当您有缓存内容时，`input_tokens` 通常会远小于您的总输入。例如，对于一个 200k 令牌的缓存文档和一个 50 令牌的用户问题，即使总输入为 200,050 个令牌，您看到的也会是 `input_tokens: 50`。

  就速率限制而言，只有 `input_tokens` + `cache_creation_input_tokens` 计入您的 ITPM 限制，这使得[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)成为提高吞吐量的有效方法。
</Note>

**示例：** 在 2,000,000 ITPM 限制和 80% 缓存命中率的情况下，您实际上每分钟可以处理 10,000,000 个总输入令牌（2M 未缓存 + 8M 缓存），因为缓存令牌不计入您的速率限制。这些缓存令牌按[缓存读取费率](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#pricing)计费，该费率仅为基础输入价格的一小部分。

要在速率限制内获得最高吞吐量，请缓存重复的内容，例如系统指令和提示、大型上下文文档、工具定义以及对话历史；有关指导，请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)。通过有效的缓存，您可以在不提高速率限制的情况下大幅提高实际吞吐量。请在 Claude Console 的[用量](https://platform.claude.com/usage)页面上监控您的缓存命中率，以调整缓存策略。

OTPM 速率限制在生成输出令牌时实时评估，仅计算实际生成的令牌。`max_tokens` 参数不参与 OTPM 速率限制的计算，因此设置较高的 `max_tokens` 值不会对速率限制产生不利影响。

下表中的每一行都有各自的 RPM、ITPM 和 OTPM 限制。有些行涵盖共享同一限制的多个模型；请参阅表格后面的说明。 您可以在 Claude Console 的[速率限制](https://platform.claude.com/usage/limits)页面上查看您当前的 rate limit（速率限制）及其行为，也可以使用[速率限制 API](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api) 读取已配置的限制并以编程方式设置工作区限制。

<Note>
  速率限制在所有 `inference_geo` 值之间共享。使用 `inference_geo: "us"` 和 `inference_geo: "global"` 的请求从同一个速率限制池中消耗额度。
</Note>

<Tabs>
  <Tab title="Start 层级">
    | 模型                                            | 每分钟最大请求数（RPM） | 每分钟最大输入令牌数（ITPM） | 每分钟最大输出令牌数（OTPM） |
    | --------------------------------------------- | ------------- | ---------------- | ---------------- |
    | Claude Fable 5.2                              | 4,000         | 500,000          | 100,000          |
    | Claude Fable 5.1 和 Claude Fable 5<sup>1</sup> | 4,000         | 500,000          | 100,000          |
    | Claude Opus 5.5                               | 4,000         | 2,000,000        | 400,000          |
    | Claude Opus 5                                 | 1,000         | 2,000,000        | 400,000          |
    | Claude Opus 4.x<sup>2</sup>                   | 1,000         | 2,000,000        | 400,000          |
    | Claude Sonnet 5.5                             | 4,000         | 2,000,000        | 400,000          |
    | Claude Sonnet 5                               | 1,000         | 2,000,000        | 400,000          |
    | Claude Sonnet 4.x<sup>3</sup>                 | 1,000         | 2,000,000        | 400,000          |
    | Claude Haiku 5.5                              | 4,000         | 2,000,000        | 400,000          |
    | Claude Haiku 4.5                              | 1,000         | 2,000,000        | 400,000          |
  </Tab>

  <Tab title="Build 层级">
    | 模型                                            | 每分钟最大请求数（RPM） | 每分钟最大输入令牌数（ITPM） | 每分钟最大输出令牌数（OTPM） |
    | --------------------------------------------- | ------------- | ---------------- | ---------------- |
    | Claude Fable 5.2                              | 10,000        | 1,500,000        | 300,000          |
    | Claude Fable 5.1 和 Claude Fable 5<sup>1</sup> | 10,000        | 1,500,000        | 300,000          |
    | Claude Opus 5.5                               | 10,000        | 5,000,000        | 1,000,000        |
    | Claude Opus 5                                 | 5,000         | 5,000,000        | 1,000,000        |
    | Claude Opus 4.x<sup>2</sup>                   | 5,000         | 5,000,000        | 1,000,000        |
    | Claude Sonnet 5.5                             | 10,000        | 5,000,000        | 1,000,000        |
    | Claude Sonnet 5                               | 5,000         | 5,000,000        | 1,000,000        |
    | Claude Sonnet 4.x<sup>3</sup>                 | 5,000         | 5,000,000        | 1,000,000        |
    | Claude Haiku 5.5                              | 10,000        | 5,000,000        | 1,000,000        |
    | Claude Haiku 4.5                              | 5,000         | 5,000,000        | 1,000,000        |
  </Tab>

  <Tab title="Scale 层级">
    | 模型                                            | 每分钟最大请求数（RPM） | 每分钟最大输入令牌数（ITPM） | 每分钟最大输出令牌数（OTPM） |
    | --------------------------------------------- | ------------- | ---------------- | ---------------- |
    | Claude Fable 5.2                              | 13,000        | 4,000,000        | 800,000          |
    | Claude Fable 5.1 和 Claude Fable 5<sup>1</sup> | 13,000        | 4,000,000        | 800,000          |
    | Claude Opus 5.5                               | 13,000        | 10,000,000       | 2,000,000        |
    | Claude Opus 5                                 | 10,000        | 10,000,000       | 2,000,000        |
    | Claude Opus 4.x<sup>2</sup>                   | 10,000        | 10,000,000       | 2,000,000        |
    | Claude Sonnet 5.5                             | 13,000        | 10,000,000       | 2,000,000        |
    | Claude Sonnet 5                               | 10,000        | 10,000,000       | 2,000,000        |
    | Claude Sonnet 4.x<sup>3</sup>                 | 10,000        | 10,000,000       | 2,000,000        |
    | Claude Haiku 5.5                              | 13,000        | 10,000,000       | 2,000,000        |
    | Claude Haiku 4.5                              | 10,000        | 10,000,000       | 2,000,000        |
  </Tab>

  <Tab title="自定义层级">
    如果您需要高于 Scale 层级的限制，请通过 Claude Console 的[速率限制](https://platform.claude.com/usage/limits)页面联系销售团队。
  </Tab>
</Tabs>

*<sup>1 Fable 速率限制是一个总限制，适用于 Claude Fable 5.1 和 Claude Fable 5 的合并流量。Claude Fable 5.2 有单独的速率限制，不属于此合并额度。Claude Mythos 5.2、Claude Mythos 5.1 和 Claude Mythos 5 共享另一个单独的合并限制。</sup>*

*<sup>2 Opus 速率限制是一个总限制，适用于 Claude Opus 4.8、Opus 4.7、Opus 4.6 和 Opus 4.5 的合并流量。Claude Opus 5.5 和 Claude Opus 5 各自有单独的速率限制，不属于此合并额度。</sup>*

*<sup>3 Sonnet 4.x 速率限制是一个总限制，适用于 Sonnet 4.6 和 Sonnet 4.5（已弃用；请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)）的合并流量。Claude Sonnet 5.5 和 Claude Sonnet 5 各自有单独的速率限制，不属于此合并额度。</sup>*

### Message Batches API

Message Batches API 有自己的一组速率限制，在所有模型之间共享。这些限制包括对所有 API 端点的每分钟请求数（RPM）限制，以及对可同时处于处理队列中的批处理请求数量的限制。这里的"批处理请求"是指消息批次的一部分。您可以创建一个包含数千个批处理请求的消息批次，其中每个请求都计入此限制。批处理请求会一直留在处理队列中，直到模型成功处理它。

<Tabs>
  <Tab title="Start 层级">
    | 每分钟最大请求数（RPM） | 处理队列中的最大批处理请求数 | 每批最大批处理请求数 |
    | ------------- | -------------- | ---------- |
    | 1,000         | 200,000        | 100,000    |
  </Tab>

  <Tab title="Build 层级">
    | 每分钟最大请求数（RPM） | 处理队列中的最大批处理请求数 | 每批最大批处理请求数 |
    | ------------- | -------------- | ---------- |
    | 2,000         | 300,000        | 100,000    |
  </Tab>

  <Tab title="Scale 层级">
    | 每分钟最大请求数（RPM） | 处理队列中的最大批处理请求数 | 每批最大批处理请求数 |
    | ------------- | -------------- | ---------- |
    | 4,000         | 500,000        | 100,000    |
  </Tab>

  <Tab title="自定义层级">
    如果您需要高于 Scale 层级的限制，请通过 Claude Console 的[速率限制](https://platform.claude.com/usage/limits)页面联系销售团队。
  </Tab>
</Tabs>

### Managed Agents

[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 端点按组织进行速率限制。这些限制独立于本页面前面描述的 Messages API 速率限制。

| 操作                               | 限制            |
| -------------------------------- | ------------- |
| 创建类端点（例如智能体、会话和环境）               | 每分钟 300 个请求   |
| 读取类端点（例如 retrieve、list 和 stream） | 每分钟 1,200 个请求 |

### Files API

[Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 请求有自己的按组织限制，在上传、列出、检索、下载和删除操作之间共享，并与本页面前文所述的 Messages API 限制相互独立。当前数值请参阅 [Files API 速率限制](https://platform.claude.com/docs/zh-CN/build-with-claude/files#rate-limits)。

### 快速模式速率限制

通过 `speed: "fast"` 使用[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)时，将适用独立于标准速率限制的专用速率限制。这些限制适用于 Claude Fable 5.2、Claude Mythos 5.2、Claude Opus 5.5、Claude Opus 5 和 Claude Opus 4.8。快速模式在 Claude Opus 5 上处于研究预览阶段，在 Claude Opus 4.8 上已弃用。

Claude Fable 5.2、Claude Mythos 5.2 和 Claude Opus 5.5 有默认的快速模式速率限制。该默认限制适用于 Start、Build 和 Scale 层级。您无需为这些模型申请快速模式访问权限。处于评估层级的组织在升级到 Start、Build 或 Scale 层级后，即可在这些模型上使用快速模式。

要查看您的快速模式速率限制，请前往 Claude Console 中的[速率限制](https://platform.claude.com/usage/limits)页面。当超出快速模式速率限制时，API 会返回带有 `retry-after` 标头的 `429` 错误。

响应中包含 `anthropic-fast-*` 标头，用于指示您的快速模式速率限制状态。有关这些标头的详情，请参阅[快速模式速率限制](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#rate-limits)。

### 在 Console 中监控您的速率限制

您可以在 [Claude Console](https://platform.claude.com/) 的[用量](https://platform.claude.com/usage)页面监控您的速率限制使用情况。

除了提供令牌和请求图表外，**用量**页面还提供两个单独的速率限制图表。使用这些图表可以查看您还有多少增长空间、识别何时可能达到使用高峰、了解应申请哪些速率限制，以及学习如何提高缓存率。这些图表可视化了给定速率限制（例如每个模型）的多项指标：

* **Rate Limit - Input Tokens**（速率限制 - 输入令牌）图表包括：

  * 每小时的每分钟未缓存输入令牌数最大值
  * 您当前的每分钟输入令牌数速率限制
  * 您输入令牌的缓存率（即从缓存读取的输入令牌百分比）

* **Rate Limit - Output Tokens**（速率限制 - 输出令牌）图表包括：

  * 每小时的每分钟输出令牌数最大值
  * 您当前的每分钟输出令牌数速率限制

## 申请更高的限制

如需申请更高的速率限制或更高的每月支出上限，请使用 Claude Console 中[速率限制](https://platform.claude.com/usage/limits)页面上的**申请提升等级**。Anthropic 支持团队也可以提高限制；如有紧急需求，请联系 [Anthropic 支持](https://support.claude.com)。

<Note>
  **[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)：** **申请提升等级**流程不可用。请联系您的 Anthropic 客户代表或 [Anthropic 支持](https://support.claude.com)，并提供您需要提高限制的模型、每个模型的每分钟输入和输出令牌峰值，以及您的输入中大约有多少比例是已缓存或重复的上下文。请参阅 [Claude Platform on AWS 上的速率限制和配额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#rate-limits-and-quotas)。
</Note>

## 为工作区设置更低的限制

有关工作区的更多信息，请参阅[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)。

为了保护组织中的工作区免受潜在的过度使用，您可以为每个工作区设置自定义的支出和速率限制。

示例：如果您组织的限制为每分钟 40,000 个输入令牌和每分钟 8,000 个输出令牌，您可以将某个工作区限制为每分钟 30,000 个输入令牌。这可以保护其他工作区免受潜在的过度使用，并使资源在您的组织内更均衡地分配。其余每分钟 10,000 个输入令牌，加上该工作区未用完的 30,000 个中的任何部分，仍可供其他工作区使用。

注意：

* 您无法为默认工作区设置支出限制。
* 如果未设置，工作区限制与组织的限制一致。
* 工作区限制是按每种限制器类型设置的（例如每分钟请求数、每分钟输入令牌数或每分钟输出令牌数）。
* 组织范围的限制始终适用，即使各工作区限制之和更高。

要以编程方式读取您当前的组织和工作区速率限制或设置工作区速率限制，请使用[速率限制 API](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api)。

## 响应标头

API 响应中包含的标头会向您显示所执行的速率限制、当前使用情况以及限制何时重置。

返回以下标头：

| 标头                                            | 描述                                                                                                                                                       |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `retry-after`                                 | 在可以重试请求之前需要等待的秒数。提前重试将会失败。并非每个 429 都带有此标头；支出上限 429 从不带有此标头（请参阅[达到支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#reaching-your-spend-cap)）。 |
| `anthropic-ratelimit-requests-limit`          | 任意速率限制周期内允许的最大请求数。                                                                                                                                       |
| `anthropic-ratelimit-requests-remaining`      | 在被速率限制之前剩余的请求数。                                                                                                                                          |
| `anthropic-ratelimit-requests-reset`          | 请求速率限制完全补充的时间，以 RFC 3339 格式提供。                                                                                                                           |
| `anthropic-ratelimit-tokens-limit`            | 任意速率限制周期内允许的最大令牌数。                                                                                                                                       |
| `anthropic-ratelimit-tokens-remaining`        | 在被速率限制之前剩余的令牌数（四舍五入到最接近的千位）。                                                                                                                             |
| `anthropic-ratelimit-tokens-reset`            | 令牌速率限制完全补充的时间，以 RFC 3339 格式提供。                                                                                                                           |
| `anthropic-ratelimit-input-tokens-limit`      | 任意速率限制周期内允许的最大输入令牌数。                                                                                                                                     |
| `anthropic-ratelimit-input-tokens-remaining`  | 在被速率限制之前剩余的输入令牌数（四舍五入到最接近的千位）。                                                                                                                           |
| `anthropic-ratelimit-input-tokens-reset`      | 输入令牌速率限制完全补充的时间，以 RFC 3339 格式提供。                                                                                                                         |
| `anthropic-ratelimit-output-tokens-limit`     | 任意速率限制周期内允许的最大输出令牌数。                                                                                                                                     |
| `anthropic-ratelimit-output-tokens-remaining` | 在被速率限制之前剩余的输出令牌数（四舍五入到最接近的千位）。                                                                                                                           |
| `anthropic-ratelimit-output-tokens-reset`     | 输出令牌速率限制完全补充的时间，以 RFC 3339 格式提供。                                                                                                                         |
| `anthropic-priority-input-tokens-limit`       | 任意速率限制周期内允许的最大 Priority Tier 输入令牌数。（仅限 Priority Tier）                                                                                                    |
| `anthropic-priority-input-tokens-remaining`   | 在被速率限制之前剩余的 Priority Tier 输入令牌数（四舍五入到最接近的千位）。（仅限 Priority Tier）                                                                                          |
| `anthropic-priority-input-tokens-reset`       | Priority Tier 输入令牌速率限制完全补充的时间，以 RFC 3339 格式提供。（仅限 Priority Tier）                                                                                         |
| `anthropic-priority-output-tokens-limit`      | 任意速率限制周期内允许的最大 Priority Tier 输出令牌数。（仅限 Priority Tier）                                                                                                    |
| `anthropic-priority-output-tokens-remaining`  | 在被速率限制之前剩余的 Priority Tier 输出令牌数（四舍五入到最接近的千位）。（仅限 Priority Tier）                                                                                          |
| `anthropic-priority-output-tokens-reset`      | Priority Tier 输出令牌速率限制完全补充的时间，以 RFC 3339 格式提供。（仅限 Priority Tier）                                                                                         |

`anthropic-ratelimit-tokens-*` 标头显示当前生效的最严格限制的值。例如，如果您超出了工作区每分钟令牌限制，这些标头将包含工作区每分钟令牌速率限制的值。如果工作区限制不适用，这些标头将返回剩余的总令牌数，其中总数为输入令牌和输出令牌之和。要查看某个请求计入了哪个工作区，请读取 `anthropic-workspace-id` [响应标头](https://platform.claude.com/docs/zh-CN/api/overview#response-headers)，其中包含您的 API 密钥或访问令牌所解析到的工作区 ID。
