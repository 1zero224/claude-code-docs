---
title: Claude Haiku 5.5
url: https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview
description: Claude Haiku 5.5 概览：其用途、各平台上的模型 ID、上下文窗口、输出限制、定价、可用性，以及使用它进行构建的指南和资源。
---

**Latest.** Released October 7, 2026.

For high-volume, latency-sensitive tasks such as classification, extraction, and routing

Model ID: `claude-haiku-5-5`

Context window: 1M tokens · Max output: 128K tokens · Input pricing: From $0.10 / MTok · Output pricing: From $0.50 / MTok

[Announcement](https://www.anthropic.com/claude-haiku-5-5) · [What’s new](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5) · [Migration guide](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide)

## 概述

Claude Haiku 5.5 专为高吞吐量、对延迟敏感的工作而构建，例如分类、路由、提取和子代理任务。它支持带有 effort 参数的自适应思考、1M 令牌的上下文窗口，以及最多 128k 输出令牌。它使用与 Claude 4.7 及更高版本模型相同的较新分词器，因此相同的文本计为的令牌数比 Claude Haiku 4.5 多约 30%。其思考块仅在生成它们的账户或与该账户关联的账户中有效。

有关代码变更，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide)。有关模型 ID、定价和限制，请参阅 [Claude Haiku 5.5 概述](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)。有关提示指导，请参阅[为 Claude Haiku 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5)。

[Claude Haiku 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5)

## 对比

| Model                                                                                  | Context | Max output | Price / MTok       | Latency  | Thinking             | Default effort | Knowledge cutoff |
| :------------------------------------------------------------------------------------- | :------ | :--------- | :----------------- | :------- | :------------------- | :------------- | :--------------- |
| [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview)   | 1M      | 128K       | $8 / $40           | Moderate | Adaptive (always on) | `high`         | Jun 2026         |
| [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)     | 1M      | 128K       | $4 / $20           | Moderate | Adaptive (always on) | `medium`       | Jun 2026         |
| [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) | 1M      | 128K       | $2 / $10           | Fast     | Adaptive             | `high`         | Jun 2026         |
| **Claude Haiku 5.5** (this model)                                                      | 1M      | 128K       | From $0.10 / $0.50 | Fastest  | Adaptive             | `medium`       | Jun 2026         |

* **Context:** 1M tokens is roughly 555k words or 2.5M Unicode characters on the current tokenizer (introduced with Claude Opus 4.7); models before it fit about 750k words in 1M tokens. 200k tokens is roughly 150k words.
* **Max output:** Synchronous Messages API limit. On the Message Batches API, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Sonnet 5, Claude Haiku 5.5, Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, and Claude Sonnet 4.6 support up to 300k output tokens with the output-300k-2026-03-24 beta header.
* **Price / MTok:** Input / output, base price per million tokens. Batch API requests are 50% off; prompt caching reads cost 10% of the base input price (2.5% on Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, and Claude Mythos 5.1, 5% on Claude Opus 5.5 and Claude Sonnet 5.5). See Pricing for the full list.
* **Latency:** Comparative latency, relative to the current lineup, as published in the models overview. Actual latency depends on prompt length, output length, and thinking effort.
* **Thinking:** Adaptive thinking lets the model decide how much to think, steered by effort. Extended thinking is the manual budget\_tokens mode on earlier models.
* **Default effort:** The effort parameter’s default on the Claude API. Models without a value don’t support the parameter.
* **Knowledge cutoff:** Reliable knowledge cutoff: the date through which the model’s knowledge is most extensive and reliable.

## 规格

### Model IDs

| Platform                                                                                                                                                      | Model ID                     |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------------------- |
| Claude API                                                                                                                                                    | `claude-haiku-5-5`           |
| [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                           | `anthropic.claude-haiku-5-5` |
| [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)       | `claude-haiku-5-5`           |
| [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                     | `claude-haiku-5-5`           |
| [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)                                                     | `claude-haiku-5-5`           |
| [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | `claude-haiku-5-5`           |

### Pricing

| Feature                                                                                   | Value                                                                                         |
| :---------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| Input                                                                                     | $0.10 / MTok for prompts up to 100,000 tokens; $0.50 / MTok for prompts over 100,000 tokens   |
| Output                                                                                    | $0.50 / MTok for prompts up to 100,000 tokens; $2.50 / MTok for prompts over 100,000 tokens   |
| [5m cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $0.125 / MTok for prompts up to 100,000 tokens; $0.625 / MTok for prompts over 100,000 tokens |
| [1h cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $0.20 / MTok for prompts up to 100,000 tokens; $1 / MTok for prompts over 100,000 tokens      |
| [Cache read](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)     | $0.01 / MTok for prompts up to 100,000 tokens; $0.05 / MTok for prompts over 100,000 tokens   |
| [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)    | 50% discount on input and output                                                              |

[Full price list](https://platform.claude.com/docs/zh-CN/about-claude/pricing)

### Capabilities

| Feature                                                                                                                        | Value                  |
| :----------------------------------------------------------------------------------------------------------------------------- | :--------------------- |
| [Context window](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)                                     | 1M tokens              |
| Max output                                                                                                                     | 128K tokens            |
| [Max output (Batch API, beta)](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#extended-output-beta) | 300K tokens            |
| [Thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)                                                  | Adaptive               |
| [Default effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)                                              | `medium`               |
| Comparative latency                                                                                                            | Fastest                |
| Input → output                                                                                                                 | Text and images → text |
| Reliable knowledge cutoff                                                                                                      | Jun 2026               |
| Training data cutoff                                                                                                           | Jun 2026               |

### Availability

| Feature                                                                          | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| :------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Status](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) | Active (latest)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Released                                                                         | October 7, 2026                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Retirement                                                                       | Not sooner than October 7, 2027                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Platforms                                                                        | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) |

## 须知

* “Adaptive thinking”（自适应思考）默认开启。使用 [“effort parameter”（effort 参数）](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)控制思考深度。
* 请省略 `temperature`、`top_p` 和 `top_k`。
* 在 [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#extended-output-beta) 上，Claude Haiku 5.5 通过 `output-300k-2026-03-24` beta 标头支持最多 300k 输出令牌。
* 最小可缓存提示长度为 512 个令牌。请参阅 [“Prompt caching”（提示缓存）](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#cache-limitations)。
* 使用 [Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 以编程方式查询限制和功能。

## 资源

<CardGroup cols={3}>
  <Card title="Claude Haiku 5.5 提示指南" icon="lightbulb" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5">
    Claude Haiku 5.5 特有的行为差异和提示模式。
  </Card>

  <Card title="降低延迟" icon="lightning" href="https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/reduce-latency">
    选择模型和 effort 级别、调整提示并流式传输输出，以获得更快的响应。
  </Card>

  <Card title="自适应思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    Claude Haiku 5.5 会自行决定何时思考以及思考多少。使用 `effort` 引导思考深度。
  </Card>

  <Card title="上下文窗口" icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows">
    “Context window”（上下文窗口）如何计数和管理。
  </Card>
</CardGroup>

## 参考

<CardGroup cols={3}>
  <Card title="系统提示" icon="text" href="https://platform.claude.com/docs/zh-CN/release-notes/system-prompts/claude-haiku-5-5">
    Claude Haiku 5.5 在 claude.ai 和 Claude 应用中使用的 “system prompt”（系统提示）。
  </Card>

  <Card title="系统卡" icon="file" href="https://www.anthropic.com/document/claude-haiku-5-5-system-card">
    Claude Haiku 5.5 的安全评估和部署决策。
  </Card>

  <Card title="定价" icon="coins" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
    完整价格表，包括批处理折扣和提示缓存费率。
  </Card>

  <Card title="模型 ID 和版本控制" icon="fingerprint" href="https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions">
    模型 ID、别名和固定快照的工作方式。
  </Card>

  <Card title="模型弃用" icon="clock" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    每个 Claude 模型的生命周期状态和停用承诺。
  </Card>
</CardGroup>
