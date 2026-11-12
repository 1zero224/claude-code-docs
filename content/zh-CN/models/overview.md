---
title: 模型概览
url: https://platform.claude.com/docs/zh-CN/models/overview
description: Claude 是由 Anthropic 开发的一系列最先进的大型语言模型。本指南介绍可用的模型并比较它们的性能。
---

# 模型概览

Claude 是由 Anthropic 开发的一系列最先进的大型语言模型。比较当前的模型阵容，查找每个平台的模型 ID，并打开每个模型的页面以查看其完整规格和资源。

<HomeQuickChip icon="Signpost" href="https://platform.claude.com/docs/zh-CN/about-claude/models/choosing-a-model">
  选择模型
</HomeQuickChip>

<HomeQuickChip icon="DollarSign" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
  定价
</HomeQuickChip>

<HomeQuickChip icon="ArrowUpCircle" href="https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide">
  迁移指南
</HomeQuickChip>

## 最新模型

如果您不确定该使用哪个模型，对于大多数工作负载，建议从 [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview) 开始。对于要求较高的推理任务和长周期的 “agentic”（智能体）工作，或者当您在较高 effort 级别下对 Claude Opus 5.5 进行的 “evals”（评估）仍未达到要求时，请使用 [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview)。所有当前模型均支持文本和图像输入、文本输出、多语言能力、"vision"（视觉）以及 “tool use”（工具使用）。

| Feature                                                                                                                                                          | Claude Fable 5.2                                                                     | Claude Opus 5.5                                                                    | Claude Sonnet 5.5                                                                      | Claude Haiku 5.5                                                                         |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- |
| Description                                                                                                                                                      | For demanding reasoning and long-horizon agentic work                                | For long-running agentic coding and knowledge work                                 | The best combination of speed and intelligence                                         | For high-volume, latency-sensitive tasks such as classification, extraction, and routing |
| Model page                                                                                                                                                       | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview) | [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview) | [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) | [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)     |
| Comparative latency                                                                                                                                              | Moderate                                                                             | Moderate                                                                           | Fast                                                                                   | Fastest                                                                                  |
| [Pricing](https://platform.claude.com/docs/zh-CN/about-claude/pricing)                                                                                           | $8 / input MTok, $40 / output MTok                                                   | $4 / input MTok, $20 / output MTok                                                 | $2 / input MTok, $10 / output MTok                                                     | From $0.10 / input MTok, From $0.50 / output MTok                                        |
| Claude API ID                                                                                                                                                    | `claude-fable-5-2`                                                                   | `claude-opus-5-5`                                                                  | `claude-sonnet-5-5`                                                                    | `claude-haiku-5-5`                                                                       |
| [Thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)                                                                                    | Adaptive (always on)                                                                 | Adaptive (always on)                                                               | Adaptive                                                                               | Adaptive                                                                                 |
| [Default effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)                                                                                | `high`                                                                               | `medium`                                                                           | `high`                                                                                 | `medium`                                                                                 |
| [Context window](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)                                                                       | 1M tokens                                                                            | 1M tokens                                                                          | 1M tokens                                                                              | 1M tokens                                                                                |
| Max output                                                                                                                                                       | 128K tokens                                                                          | 128K tokens                                                                        | 128K tokens                                                                            | 128K tokens                                                                              |
| Reliable knowledge cutoff                                                                                                                                        | Jun 2026                                                                             | Jun 2026                                                                           | Jun 2026                                                                               | Jun 2026                                                                                 |
| Training data cutoff                                                                                                                                             | Jun 2026                                                                             | Jun 2026                                                                           | Jun 2026                                                                               | Jun 2026                                                                                 |
| [Retirement](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)                                                                             | Not sooner than November 9, 2027                                                     | Not sooner than September 22, 2027                                                 | Not sooner than September 28, 2027                                                     | Not sooner than October 7, 2027                                                          |
| Claude API alias                                                                                                                                                 | `claude-fable-5-2`                                                                   | `claude-opus-5-5`                                                                  | `claude-sonnet-5-5`                                                                    | `claude-haiku-5-5`                                                                       |
| [Amazon Bedrock ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                           | `global.anthropic.claude-fable-5-2`                                                  | `anthropic.claude-opus-5-5`                                                        | `anthropic.claude-sonnet-5-5`                                                          | `anthropic.claude-haiku-5-5`                                                             |
| [Claude Models on Google Cloud (Google managed) ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)       | `claude-fable-5-2`                                                                   | `claude-opus-5-5`                                                                  | `claude-sonnet-5-5`                                                                    | `claude-haiku-5-5`                                                                       |
| [Microsoft Foundry ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                     | `claude-fable-5-2`                                                                   | `claude-opus-5-5`                                                                  | `claude-sonnet-5-5`                                                                    | `claude-haiku-5-5`                                                                       |
| [Claude Platform on AWS ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)                                                     | `claude-fable-5-2`                                                                   | `claude-opus-5-5`                                                                  | `claude-sonnet-5-5`                                                                    | `claude-haiku-5-5`                                                                       |
| [Claude Models on Google Cloud (Anthropic managed) ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | `claude-fable-5-2`                                                                   | `claude-opus-5-5`                                                                  | `claude-sonnet-5-5`                                                                    | `claude-haiku-5-5`                                                                       |

* **Comparative latency:** Relative to the current lineup. Actual latency depends on prompt length, output length, and thinking effort.
* **Pricing:** Base price per million tokens. Batch API requests are 50% off; prompt cache reads cost 10% of the base input price (2.5% on Claude Fable 5.2 and Claude Fable 5.1, 5% on Claude Opus 5.5 and Claude Sonnet 5.5). See Pricing for cache writes, long-context, and per-platform pricing.
* **Claude API ID:** Every Claude model ID is a pinned snapshot, including the dateless IDs used from the 4.6 generation on.
* **Thinking:** Adaptive thinking lets the model decide how much to think, steered by effort. Extended thinking is the manual thinking.type “enabled” + budget\_tokens mode on earlier models; it is deprecated on Claude Opus 4.6 and Claude Sonnet 4.6 and not accepted on later models.
* **Default effort:** The effort parameter’s default on the Claude API. Set effort explicitly to use a different level.
* **Context window:** 1M tokens is roughly 555k words or 2.5M Unicode characters on the current tokenizer (introduced with Claude Opus 4.7); models before it fit about 750k words in 1M tokens. 200k tokens is roughly 150k words.
* **Max output:** Synchronous Messages API limit. On the Message Batches API, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Sonnet 5, Claude Haiku 5.5, Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, and Claude Sonnet 4.6 support up to 300k output tokens with the output-300k-2026-03-24 beta header.
* **Reliable knowledge cutoff:** The date through which the model’s knowledge is most extensive and reliable. Training data cutoff (under Additional details) is the broader range of data used. See Anthropic’s Transparency Hub for details.
* **Retirement:** Anthropic’s commitment for the platforms that Anthropic operates. Partner-operated platforms set their own dates.
* **Claude API alias:** For models before the 4.6 generation, the alias is a convenience pointer that resolves to the dated ID. Dateless IDs are their own pinned snapshot; the alias row repeats them.
* **Amazon Bedrock ID:** The ID on Bedrock’s bedrock-mantle endpoint; a model that Bedrock serves only on its bedrock-runtime endpoint shows its inference profile ID instead. Bedrock offers global endpoints (dynamic routing) and regional endpoints (guaranteed data routing) for Claude Sonnet 4.5 and later, and sets its own lifecycle dates.
* **Claude Models on Google Cloud (Google managed) ID:** Claude Models on Google Cloud (Google managed) offers global, multi-region, and regional endpoints, and sets its own lifecycle dates.
* **Microsoft Foundry ID:** Foundry deployments default to the Claude API model ID (the alias, where one exists); the deployment name is what you send. Foundry follows the Claude API lifecycle schedule.
* **Claude Platform on AWS ID:** Claude Platform on AWS uses the Claude API model IDs (the dateless form where the Claude API has an alias), not Bedrock-style IDs, and follows Anthropic’s first-party model lifecycle.

See [Model IDs and versioning](https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions) and [Pricing](https://platform.claude.com/docs/zh-CN/about-claude/pricing).

选定模型后，[了解如何进行您的第一次 API 调用](https://platform.claude.com/docs/zh-CN/get-started)。要了解模型 ID、别名和快照的工作方式，请参阅[模型 ID 和版本控制](https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions)。有关每个模型背后的可靠知识截止日期和训练数据截止日期，请参阅 Anthropic 的 [Transparency Hub](https://www.anthropic.com/transparency)。

## 所有模型

较旧的模型可能仍然可用，但在可能的情况下，您应默认使用[最新模型](https://platform.claude.com/docs/zh-CN/models/overview#latest-models-comparison)。要迁移到较新的模型，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。有关退役日期，请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。

| Model                                                                                                                                                                     | Capabilities: Context window | Capabilities: Max output | Base tokens: Input | Base tokens: Output |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------------------- | :----------------------- | :----------------- | :------------------ |
| [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview)                                                                                      | 1M tokens                    | 128K tokens              | $8 / MTok          | $40 / MTok          |
| [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)                                                                                        | 1M tokens                    | 128K tokens              | $4 / MTok          | $20 / MTok          |
| [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview)                                                                                    | 1M tokens                    | 128K tokens              | $2 / MTok          | $10 / MTok          |
| [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)                                                                                      | 1M tokens                    | 128K tokens              | From $0.10 / MTok  | From $0.50 / MTok   |
| [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) ([limited availability](https://support.claude.com/en/articles/14604842))          | 1M tokens                    | 128K tokens              | $8 / MTok          | $40 / MTok          |
| [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview)                                                                                      | 1M tokens                    | 128K tokens              | $10 / MTok         | $50 / MTok          |
| [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview) ([limited availability](https://support.claude.com/en/articles/14604842))          | 1M tokens                    | 128K tokens              | $10 / MTok         | $50 / MTok          |
| [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview)                                                                                          | 1M tokens                    | 128K tokens              | $10 / MTok         | $50 / MTok          |
| [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview) ([limited availability](https://support.claude.com/en/articles/14604842))              | 1M tokens                    | 128K tokens              | $10 / MTok         | $50 / MTok          |
| [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview)                                                                                            | 1M tokens                    | 128K tokens              | $5 / MTok          | $25 / MTok          |
| [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                                                                                        | 1M tokens                    | 128K tokens              | $5 / MTok          | $25 / MTok          |
| [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview)                                                                                        | 1M tokens                    | 128K tokens              | $5 / MTok          | $25 / MTok          |
| [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview)                                                                                        | 1M tokens                    | 128K tokens              | $5 / MTok          | $25 / MTok          |
| [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview)                                                                                        | 200K tokens                  | 64K tokens               | $5 / MTok          | $25 / MTok          |
| Claude Opus 4.1 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | 200K tokens                  | 32K tokens               | $15 / MTok         | $75 / MTok          |
| Claude Opus 4 ([retired, except on Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))               | 200K tokens                  | 32K tokens               | $15 / MTok         | $75 / MTok          |
| [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview)                                                                                        | 1M tokens                    | 128K tokens              | $2 / MTok          | $10 / MTok          |
| [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview)                                                                                    | 1M tokens                    | 128K tokens              | $3 / MTok          | $15 / MTok          |
| [Claude Sonnet 4.5](https://platform.claude.com/docs/zh-CN/models/sonnet-4-5/overview)                                                                                    | 200K tokens                  | 64K tokens               | $3 / MTok          | $15 / MTok          |
| Claude Sonnet 4 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | 200K tokens                  | 64K tokens               | $3 / MTok          | $15 / MTok          |
| [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/haiku-4-5/overview)                                                                                      | 200K tokens                  | 64K tokens               | $1 / MTok          | $5 / MTok           |

* **MTok:** Million tokens. $5 / MTok is $5 for every million tokens.
* **Limited access:** Available only to organizations verified through Anthropic’s verification programs, such as the [Cyber Verification Program](https://support.claude.com/en/articles/14604842). To request access, apply to the program that covers your use case.
* **Retired:** May still be available on other cloud platforms. See [Model deprecations](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) for more.

## 使用 Models API

您可以通过 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 以编程方式查询模型能力和令牌限制。响应中包含每个可用模型的 [`max_input_tokens`](https://platform.claude.com/docs/zh-CN/api/models#model_info.max_input_tokens)、[`max_tokens`](https://platform.claude.com/docs/zh-CN/api/models#model_info.max_tokens) 以及一个 [`capabilities`](https://platform.claude.com/docs/zh-CN/api/models#model_info.capabilities) 对象。

响应中的每个模型还有一个 [`line`](https://platform.claude.com/docs/zh-CN/api/models#model_info.line) 字段，用于指明该模型所属的模型系列。Claude Opus 4.5 和 Claude Opus 4.6 都报告为 `opus`。使用 `line` 对模型进行分组，例如在模型选择器中。当模型不属于任何系列时，`line` 为 `null`。请读取 `line`，而不要从模型的 `id` 推断它。Anthropic 可能会添加更多系列，因此不要将这组值视为固定不变。

每个模型的 `capabilities` 对象都包含 [`thinking.types.disabled`](https://platform.claude.com/docs/zh-CN/api/models#thinking_types.disabled)，它报告该模型是否接受 `thinking: {type: "disabled"}`，即[关闭思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#turning-thinking-off)的设置。当模型以 400 错误拒绝 `"disabled"` 时，`supported` 为 `false`；对于不支持思考的模型，`supported` 为 `true`。即使 `supported` 为 `true`，API 仍可能因其他原因拒绝 `"disabled"` 请求。其中一个原因是模型在关闭思考时不允许的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。

每个模型的 `capabilities` 对象还包含 [`thinking.types.between_tools`](https://platform.claude.com/docs/zh-CN/api/models#thinking_types.between_tools)，它报告该模型是否接受 `thinking: {type: "between_tools"}`，即[关闭前置思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#turning-thinking-off)的设置。当模型以 400 错误拒绝 `"between_tools"` 时，`supported` 为 `false`。即使 `supported` 为 `true`，API 仍可能因其他原因拒绝 `"between_tools"` 请求，例如模型在使用 `"between_tools"` 时不允许的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。例如，对于 Claude Sonnet 5.5，Models API 报告 `thinking.types.between_tools.supported` 为 `true`，而 `thinking.types.disabled.supported` 为 `false`。

每个模型的 `capabilities` 对象还包含 [`server_tools`](https://platform.claude.com/docs/zh-CN/api/models#model_capabilities.server_tools)，它报告该模型是否接受[网页搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)和[代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)工具。它不涵盖其他服务器工具，例如网页获取。当模型接受某个工具的至少一个版本（不一定是每个版本）时，`server_tools.web_search.supported` 和 `server_tools.code_execution.supported` 为 `true`。当模型接受这两个工具中的至少一个时，`server_tools.supported` 为 `true`。即使某个工具受支持，您组织的设置仍可能导致使用该工具的请求失败。例如，管理员可以[禁用网页搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#how-to-use-web-search)。

顶层的 [`code_execution`](https://platform.claude.com/docs/zh-CN/api/models#model_capabilities.code_execution) 能力是另一项不同的检查。它报告 Claude 在代码执行工具中运行的代码是否可以调用请求中的其他工具，如[编程式工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)中所述。例如，对于 Claude Haiku 4.5，Models API 报告 `server_tools.code_execution.supported` 为 `true`，而 `code_execution.supported` 为 `false`。

API 参考列出了"列出模型"端点的参数和完整响应。

(See the `models/list` method in [the API reference](https://platform.claude.com/docs/zh-CN/api/models/list).)

## 提示与输出性能

当前的 Claude 模型在以下方面表现出色：

* **性能：** 在推理、编码、多语言任务、长上下文处理、诚实性和图像处理方面均达到顶级水平。有关通用及特定模型的提示指导，请参阅[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)。
* **引人入胜的响应：** Claude 模型非常适合需要丰富、类人交互的应用程序。如果您更喜欢简洁的响应，可以调整提示，引导模型生成您想要的输出长度。详情请参阅[提示工程指南](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering)。
* **输出质量：** 从上一代模型迁移时，您可能会注意到整体性能有更大的提升。如果您使用的是 Claude Opus 5 或更早版本，请参阅 [Claude Opus 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide)。

## 开始使用 Claude

无论您是希望将 Claude 集成到应用程序中的开发者，还是希望亲身体验 AI 强大功能的用户，以下资源都能为您提供帮助。

<CardGroup cols={3}>
  <Card title="Claude 简介" icon="check" href="https://platform.claude.com/docs/zh-CN/intro">
    探索 Claude 的能力和开发流程。
  </Card>

  <Card title="快速入门" icon="lightning" href="https://platform.claude.com/docs/zh-CN/get-started">
    了解如何在几分钟内进行您的第一次 API 调用。
  </Card>

  <Card title="选择模型" icon="compass" href="https://platform.claude.com/docs/zh-CN/about-claude/models/choosing-a-model">
    建立标准并为您的用例选择合适的模型。
  </Card>

  <Card title="定价" icon="coins" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
    完整定价，包括批量折扣和 “prompt caching”（提示缓存）费率。
  </Card>

  <Card title="模型弃用" icon="clock" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    每个模型的生命周期状态和退役承诺。
  </Card>

  <Card title="Claude Console" icon="code" href="https://platform.claude.com/">
    直接在浏览器中编写和测试提示。
  </Card>
</CardGroup>

想与 Claude 聊天？请访问 [claude.ai](https://claude.ai)。如有疑问，请联系[支持团队](https://support.claude.com/)或 [Discord 社区](https://www.anthropic.com/discord)。
