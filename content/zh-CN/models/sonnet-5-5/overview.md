---
title: Claude Sonnet 5.5
url: https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview
description: Claude Sonnet 5.5 概述：其用途、各平台上的模型 ID、上下文窗口、输出限制、定价、可用性，以及使用它进行构建的指南和资源。
---

**Latest.** Released September 28, 2026.

The best combination of speed and intelligence

Model ID: `claude-sonnet-5-5`

Context window: 1M tokens · Max output: 128K tokens · Input pricing: $2 / MTok · Output pricing: $10 / MTok

[Announcement](https://www.anthropic.com/claude-sonnet-5-5) · [What’s new](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5) · [Migration guide](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide)

## 概述

Claude Sonnet 5.5 提供了速度与智能的最佳组合。它专为日常编码、智能体和企业工作负载而构建，例如代码生成、数据分析、内容创作、视觉理解和智能体工具使用。

## 对比

| Model                                                                                | Context | Max output | Price / MTok       | Latency  | Thinking             | Default effort | Knowledge cutoff |
| :----------------------------------------------------------------------------------- | :------ | :--------- | :----------------- | :------- | :------------------- | :------------- | :--------------- |
| [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview) | 1M      | 128K       | $8 / $40           | Moderate | Adaptive (always on) | `high`         | Jun 2026         |
| [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)   | 1M      | 128K       | $4 / $20           | Moderate | Adaptive (always on) | `medium`       | Jun 2026         |
| **Claude Sonnet 5.5** (this model)                                                   | 1M      | 128K       | $2 / $10           | Fast     | Adaptive             | `high`         | Jun 2026         |
| [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview) | 1M      | 128K       | From $0.10 / $0.50 | Fastest  | Adaptive             | `medium`       | Jun 2026         |

* **Context:** 1M tokens is roughly 555k words or 2.5M Unicode characters on the current tokenizer (introduced with Claude Opus 4.7); models before it fit about 750k words in 1M tokens. 200k tokens is roughly 150k words.
* **Max output:** Synchronous Messages API limit. On the Message Batches API, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Sonnet 5, Claude Haiku 5.5, Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, and Claude Sonnet 4.6 support up to 300k output tokens with the output-300k-2026-03-24 beta header.
* **Price / MTok:** Input / output, base price per million tokens. Batch API requests are 50% off; prompt caching reads cost 10% of the base input price (2.5% on Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, and Claude Mythos 5.1, 5% on Claude Opus 5.5 and Claude Sonnet 5.5). See Pricing for the full list.
* **Latency:** Comparative latency, relative to the current lineup, as published in the models overview. Actual latency depends on prompt length, output length, and thinking effort.
* **Thinking:** Adaptive thinking lets the model decide how much to think, steered by effort. Extended thinking is the manual budget\_tokens mode on earlier models.
* **Default effort:** The effort parameter’s default on the Claude API. Models without a value don’t support the parameter.
* **Knowledge cutoff:** Reliable knowledge cutoff: the date through which the model’s knowledge is most extensive and reliable.

## 规格

### Model IDs

| Platform                                                                                                                                                      | Model ID                      |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------- |
| Claude API                                                                                                                                                    | `claude-sonnet-5-5`           |
| [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                           | `anthropic.claude-sonnet-5-5` |
| [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)       | `claude-sonnet-5-5`           |
| [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                     | `claude-sonnet-5-5`           |
| [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)                                                     | `claude-sonnet-5-5`           |
| [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | `claude-sonnet-5-5`           |

### Pricing

| Feature                                                                                   | Value                            |
| :---------------------------------------------------------------------------------------- | :------------------------------- |
| Input                                                                                     | $2 / MTok                        |
| Output                                                                                    | $10 / MTok                       |
| [5m cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $2.50 / MTok                     |
| [1h cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $4 / MTok                        |
| [Cache read](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)     | $0.10 / MTok                     |
| [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)    | 50% discount on input and output |

[Full price list](https://platform.claude.com/docs/zh-CN/about-claude/pricing)

### Capabilities

| Feature                                                                                                                        | Value                  |
| :----------------------------------------------------------------------------------------------------------------------------- | :--------------------- |
| [Context window](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)                                     | 1M tokens              |
| Max output                                                                                                                     | 128K tokens            |
| [Max output (Batch API, beta)](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#extended-output-beta) | 300K tokens            |
| [Thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)                                                  | Adaptive               |
| [Default effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)                                              | `high`                 |
| Comparative latency                                                                                                            | Fast                   |
| Input → output                                                                                                                 | Text and images → text |
| Reliable knowledge cutoff                                                                                                      | Jun 2026               |
| Training data cutoff                                                                                                           | Jun 2026               |

### Availability

| Feature                                                                          | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| :------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Status](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) | Active (latest)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Released                                                                         | September 28, 2026                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Retirement                                                                       | Not sooner than September 28, 2027                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Platforms                                                                        | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) |

## 须知

* “Adaptive thinking”（自适应思考）默认开启。最低的思考设置是 `between_tools`，它会关闭预先思考。该设置适用于 `high` “effort”（努力程度）或更低级别。请参阅 [Claude Sonnet 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#turn-off-up-front-thinking)。
* 设置 `temperature`、`top_p` 或 `top_k` 会返回 400 错误。
* 最小可缓存提示长度为 512 个令牌。请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#cache-limitations)。
* 在 [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#extended-output-beta) 上，Claude Sonnet 5.5 配合 `output-300k-2026-03-24` beta 标头最多支持 300k 个输出令牌。
* 使用 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 以编程方式查询限制和功能。

## 资源

<CardGroup cols={3}>
  <Card title="为 Claude Sonnet 5.5 编写提示" icon="lightbulb" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5">
    Claude Sonnet 5.5 特有的行为差异和提示模式。
  </Card>

  <Card title="努力程度" icon="sliders" href="https://platform.claude.com/docs/zh-CN/build-with-claude/effort">
    用于控制思考深度、延迟和成本的设置。为每个工作负载选择一个级别。
  </Card>

  <Card title="自适应思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    自适应思考的工作原理、每个模型接受哪些思考设置，以及思考块如何被保留。
  </Card>
</CardGroup>

## 参考

<CardGroup cols={3}>
  <Card title="系统提示" icon="text" href="https://platform.claude.com/docs/zh-CN/release-notes/system-prompts/claude-sonnet-5-5">
    Claude Sonnet 5.5 在 claude.ai 和 Claude 应用中使用的系统提示。
  </Card>

  <Card title="系统卡" icon="file" href="https://www.anthropic.com/document/claude-sonnet-5-5-system-card">
    Claude Sonnet 5.5 的安全评估和部署决策。
  </Card>

  <Card title="定价" icon="coins" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
    完整价目表，包括批处理折扣和提示缓存费率。
  </Card>

  <Card title="模型 ID 和版本控制" icon="fingerprint" href="https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions">
    模型 ID、别名和固定快照的工作方式。
  </Card>

  <Card title="模型弃用" icon="clock" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    每个 Claude 模型的生命周期状态和停用承诺。
  </Card>
</CardGroup>
