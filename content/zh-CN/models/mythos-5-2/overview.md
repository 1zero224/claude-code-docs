---
title: Claude Mythos 5.2
url: https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview
description: Claude Mythos 5.2 与 Claude Fable 5.2 是同一模型，仅向通过 Anthropic 验证计划验证的组织提供。包括模型 ID、规格、定价以及如何申请访问权限。
---

**Verification required.** Released November 9, 2026.

Claude Fable 5.2 for verified organizations

Model ID: `claude-mythos-5-2`

Context window: 1M tokens · Max output: 128K tokens · Input pricing: $8 / MTok · Output pricing: $40 / MTok

[Announcement](https://www.anthropic.com/claude-fable-and-mythos-5-2) · [What’s new](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2) · [Migration guide](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-mythos-5-1-to-claude-mythos-5-2)

Claude Mythos 5.2 is available only to organizations verified through Anthropic’s verification programs, such as the [Cyber Verification Program](https://support.claude.com/en/articles/14604842). It has the same specifications and pricing as Claude Fable 5.2. To request access, apply to the program that covers your use case. [See Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview) · [How refusals and fallback work](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)

## 对比情况

| Model                                                                                  | Context | Max output | Price / MTok       | Latency  | Thinking             | Default effort | Knowledge cutoff |
| :------------------------------------------------------------------------------------- | :------ | :--------- | :----------------- | :------- | :------------------- | :------------- | :--------------- |
| [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview)   | 1M      | 128K       | $8 / $40           | Moderate | Adaptive (always on) | `high`         | Jun 2026         |
| **Claude Mythos 5.2** (this model)                                                     | 1M      | 128K       | $8 / $40           | Moderate | Adaptive (always on) | `high`         | Jun 2026         |
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

| Platform                                                                                                                                                      | Model ID                             |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------------------------------- |
| Claude API                                                                                                                                                    | `claude-mythos-5-2`                  |
| [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)                                  | `global.anthropic.claude-mythos-5-2` |
| [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)       | `claude-mythos-5-2`                  |
| [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                     | `claude-mythos-5-2`                  |
| [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | `claude-mythos-5-2`                  |

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

| Feature                                                                          | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| :------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Status](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) | Active (verification required)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Released                                                                         | November 9, 2026                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Retirement                                                                       | Not sooner than November 9, 2027                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Platforms                                                                        | Claude API, [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) |

## 参考

<CardGroup cols={3}>
  <Card title="从 Claude Mythos 5.1 迁移" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-mythos-5-1-to-claude-mythos-5-2">
    从 Claude Mythos 5.1 迁移时会发生哪些变化。
  </Card>

  <Card title="系统卡" icon="file" href="https://www.anthropic.com/document/claude-fable-5-2-mythos-5-2-system-card">
    Claude Fable 5.2 和 Claude Mythos 5.2 的安全评估和部署决策。
  </Card>

  <Card title="定价" icon="coins" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
    完整价格表，包括批处理折扣和提示缓存费率。
  </Card>

  <Card title="模型 ID 和版本控制" icon="fingerprint" href="https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions">
    模型 ID、别名和固定快照的工作原理。
  </Card>

  <Card title="模型弃用" icon="clock" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    每个 Claude 模型的生命周期状态和停用承诺。
  </Card>

  <Card title="快速模式" icon="lightning" href="https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode">
    在 Messages API 请求中以溢价定价获得 Claude Mythos 5.2 更快的输出。
  </Card>
</CardGroup>
