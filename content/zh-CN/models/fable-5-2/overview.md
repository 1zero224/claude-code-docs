---
title: Claude Fable 5.2
url: https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview
description: Claude Fable 5.2 概览：用途、各平台上的模型 ID、上下文窗口、输出限制、定价、可用性以及使用它进行构建的资源。
---

**Latest.** Released November 9, 2026.

For demanding reasoning and long-horizon agentic work

Model ID: `claude-fable-5-2`

Context window: 1M tokens · Max output: 128K tokens · Input pricing: $8 / MTok · Output pricing: $40 / MTok

[Announcement](https://www.anthropic.com/claude-fable-and-mythos-5-2) · [What’s new](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2) · [Migration guide](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide)

## 概述

Claude Fable 5.2 专为高要求的推理和长周期的智能体工作而打造。典型用途包括持续数小时的智能体会话、多步骤深度研究，以及一直推进到生成完整文档、电子表格或演示文稿的分析工作。

## Claude Fable 5.2 与 Claude Mythos 5.2

[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) 提供相同的能力，但仅面向通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织。它与 Claude Fable 5.2 具有相同的规格和定价。如需申请访问权限，请申请适用于您用例的计划，或联系您的客户团队。

## 对比情况

| Model                                                                                  | Context | Max output | Price / MTok       | Latency  | Thinking             | Default effort | Knowledge cutoff |
| :------------------------------------------------------------------------------------- | :------ | :--------- | :----------------- | :------- | :------------------- | :------------- | :--------------- |
| **Claude Fable 5.2** (this model)                                                      | 1M      | 128K       | $8 / $40           | Moderate | Adaptive (always on) | `high`         | Jun 2026         |
| [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)     | 1M      | 128K       | $4 / $20           | Moderate | Adaptive (always on) | `medium`       | Jun 2026         |
| [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) | 1M      | 128K       | $2 / $10           | Fast     | Adaptive             | `high`         | Jun 2026         |
| [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)   | 1M      | 128K       | From $0.10 / $0.50 | Fastest  | Adaptive             | `medium`       | Jun 2026         |

* **Context:** 1M tokens is roughly 555k words or 2.5M Unicode characters on the current tokenizer (introduced with Claude Opus 4.7); models before it fit about 750k words in 1M tokens. 200k tokens is roughly 150k words.
* **Max output:** Synchronous Messages API limit. On the Message Batches API, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Sonnet 5, Claude Haiku 5.5, Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, and Claude Sonnet 4.6 support up to 300k output tokens with the output-300k-2026-03-24 beta header.
* **Price / MTok:** Input / output, base price per million tokens. Batch API requests are 50% off; prompt caching reads cost 10% of the base input price (2.5% on Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, and Claude Mythos 5.1, 5% on Claude Opus 5.5 and Claude Sonnet 5.5). See Pricing for the full list.
* **Latency:** Comparative latency, relative to the current lineup, as published in the models overview. Actual latency depends on prompt length, output length, and thinking effort.
* **Thinking:** Adaptive thinking lets the model decide how much to think, steered by effort. Extended thinking is the manual budget\_tokens mode on earlier models.
* **Default effort:** The effort parameter’s default on the Claude API. Models without a value don’t support the parameter.
* **Knowledge cutoff:** Reliable knowledge cutoff: the date through which the model’s knowledge is most extensive and reliable.

## 规格

### Model IDs

| Platform                                                                                                                                                      | Model ID                            |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------- |
| Claude API                                                                                                                                                    | `claude-fable-5-2`                  |
| [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)                                  | `global.anthropic.claude-fable-5-2` |
| [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)       | `claude-fable-5-2`                  |
| [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                     | `claude-fable-5-2`                  |
| [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)                                                     | `claude-fable-5-2`                  |
| [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | `claude-fable-5-2`                  |

### Pricing

| Feature                                                                                   | Value                            |
| :---------------------------------------------------------------------------------------- | :------------------------------- |
| Input                                                                                     | $8 / MTok                        |
| Output                                                                                    | $40 / MTok                       |
| [5m cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $10 / MTok                       |
| [1h cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $16 / MTok                       |
| [Cache read](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)     | $0.20 / MTok                     |
| [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)    | 50% discount on input and output |

[Full price list](https://platform.claude.com/docs/zh-CN/about-claude/pricing)

### Capabilities

| Feature                                                                                    | Value                  |
| :----------------------------------------------------------------------------------------- | :--------------------- |
| [Context window](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows) | 1M tokens              |
| Max output                                                                                 | 128K tokens            |
| [Thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)              | Adaptive (always on)   |
| [Default effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)          | `high`                 |
| Comparative latency                                                                        | Moderate               |
| Input → output                                                                             | Text and images → text |
| Reliable knowledge cutoff                                                                  | Jun 2026               |
| Training data cutoff                                                                       | Jun 2026               |

### Availability

| Feature                                                                          | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Status](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) | Active (latest)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Released                                                                         | November 9, 2026                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Retirement                                                                       | Not sooner than November 9, 2027                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Platforms                                                                        | Claude API, [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) |

## 须知

* “Adaptive thinking”（自适应思考）始终开启，无法关闭。请使用 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)控制思考深度。
* 强制工具使用（`tool_choice` 类型 `any` 和 `tool`）会返回 400 错误。请参阅[思考与工具选择](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#thinking-and-tool-choice)。
* 请省略 `temperature`、`top_p` 和 `top_k`。有关会返回 400 错误的值，请参阅[采样参数](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#sampling-parameters)。
* Claude Fable 5.2 要求 30 天数据保留，除非获得 Anthropic 明确授权，否则无法在 “zero data retention”（零数据保留）下使用。请参阅[特定模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。
* 使用 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 以编程方式查询限制和能力。

## 资源

<CardGroup cols={3}>
  <Card title="为 Claude Fable 5.2 编写提示" icon="lightbulb" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2">
    Claude Fable 5.2 特有的行为差异和提示模式。
  </Card>

  <Card title="Claude Fable 5.2 迁移指南" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide">
    从 Claude Fable 5.1 或 Claude Opus 模型迁移时会发生哪些变化。
  </Card>

  <Card title="保留思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking">
    对于所有账户，编辑较早的轮次都会使思考块失效。
  </Card>

  <Card title="自适应思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    自适应思考的工作原理。它是 Claude Fable 5.2 上唯一的思考模式。
  </Card>

  <Card title="Effort" icon="sliders" href="https://platform.claude.com/docs/zh-CN/build-with-claude/effort">
    控制思考深度、延迟和成本的设置。为每种工作负载选择合适的级别。
  </Card>

  <Card title="拒绝与回退" icon="shield" href="https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback">
    处理分类器拒绝，并在另一个 Claude 模型上重试。
  </Card>

  <Card title="快速模式" icon="lightning" href="https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode">
    在 Messages API 请求中以溢价定价获得 Claude Fable 5.2 更快的输出。
  </Card>
</CardGroup>

## 参考

<CardGroup cols={3}>
  <Card title="系统提示" icon="text" href="https://platform.claude.com/docs/zh-CN/release-notes/system-prompts/claude-fable-5-2">
    Claude Fable 5.2 在 claude.ai 和 Claude 应用中使用的系统提示。
  </Card>

  <Card title="系统卡" icon="file" href="https://www.anthropic.com/document/claude-fable-5-2-mythos-5-2-system-card">
    Claude Fable 5.2 和 Claude Mythos 5.2 的安全评估与部署决策。
  </Card>

  <Card title="定价" icon="coins" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
    完整价目表，包括批处理折扣和提示缓存费率。
  </Card>

  <Card title="模型 ID 与版本控制" icon="fingerprint" href="https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions">
    模型 ID、别名和固定快照的工作原理。
  </Card>

  <Card title="模型弃用" icon="clock" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    每个 Claude 模型的生命周期状态和停用承诺。
  </Card>
</CardGroup>
