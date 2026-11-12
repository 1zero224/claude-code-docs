---
title: Claude Mythos 5.1
url: https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview
description: Claude Mythos 5.1 参考：与 Claude Fable 5.1 是同一模型，仅向通过 Anthropic 验证计划验证的组织提供。包括模型 ID、规格、定价以及迁移资源。Claude Mythos 5.2 是当前的 Mythos 模型。
---

**Verification required.** Released September 1, 2026.

Claude Fable 5.1 for verified organizations

Model ID: `claude-mythos-5-1`

Context window: 1M tokens · Max output: 128K tokens · Input pricing: $10 / MTok · Output pricing: $50 / MTok

[Announcement](https://www.anthropic.com/claude-fable-and-mythos-5-1) · [Migration guide](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-mythos-5-1-to-claude-mythos-5-2)

Claude Mythos 5.1 is available only to organizations verified through Anthropic’s verification programs, such as the [Cyber Verification Program](https://support.claude.com/en/articles/14604842). It has the same specifications and pricing as Claude Fable 5.1. To request access, apply to the program that covers your use case. [See Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview) · [How refusals and fallback work](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)

## 对比情况

| Model                                                                                  | Context | Max output | Price / MTok       | Thinking             | Default effort | Knowledge cutoff |
| :------------------------------------------------------------------------------------- | :------ | :--------- | :----------------- | :------------------- | :------------- | :--------------- |
| [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview)   | 1M      | 128K       | $8 / $40           | Adaptive (always on) | `high`         | Jun 2026         |
| **Claude Mythos 5.1** (this model)                                                     | 1M      | 128K       | $10 / $50          | Adaptive (always on) | `high`         | Jun 2026         |
| [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview)     | 1M      | 128K       | $4 / $20           | Adaptive (always on) | `medium`       | Jun 2026         |
| [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) | 1M      | 128K       | $2 / $10           | Adaptive             | `high`         | Jun 2026         |
| [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)   | 1M      | 128K       | From $0.10 / $0.50 | Adaptive             | `medium`       | Jun 2026         |

* **Context:** 1M tokens is roughly 555k words or 2.5M Unicode characters on the current tokenizer (introduced with Claude Opus 4.7); models before it fit about 750k words in 1M tokens. 200k tokens is roughly 150k words.
* **Max output:** Synchronous Messages API limit. On the Message Batches API, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Sonnet 5, Claude Haiku 5.5, Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, and Claude Sonnet 4.6 support up to 300k output tokens with the output-300k-2026-03-24 beta header.
* **Price / MTok:** Input / output, base price per million tokens. Batch API requests are 50% off; prompt caching reads cost 10% of the base input price (2.5% on Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, and Claude Mythos 5.1, 5% on Claude Opus 5.5 and Claude Sonnet 5.5). See Pricing for the full list.
* **Thinking:** Adaptive thinking lets the model decide how much to think, steered by effort. Extended thinking is the manual budget\_tokens mode on earlier models.
* **Default effort:** The effort parameter’s default on the Claude API. Models without a value don’t support the parameter.
* **Knowledge cutoff:** Reliable knowledge cutoff: the date through which the model’s knowledge is most extensive and reliable.

## 规格

### Model IDs

| Platform                                                                                                                                                | Model ID                      |
| :------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------- |
| Claude API                                                                                                                                              | `claude-mythos-5-1`           |
| [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                     | `anthropic.claude-mythos-5-1` |
| [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) | `claude-mythos-5-1`           |
| [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                               | `claude-mythos-5-1`           |

### Pricing

| Feature                                                                                   | Value                            |
| :---------------------------------------------------------------------------------------- | :------------------------------- |
| Input                                                                                     | $10 / MTok                       |
| Output                                                                                    | $50 / MTok                       |
| [5m cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $12.50 / MTok                    |
| [1h cache write](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching) | $20 / MTok                       |
| [Cache read](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)     | $0.25 / MTok                     |
| [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)    | 50% discount on input and output |

[Full price list](https://platform.claude.com/docs/zh-CN/about-claude/pricing)

### Capabilities

| Feature                                                                                    | Value                  |
| :----------------------------------------------------------------------------------------- | :--------------------- |
| [Context window](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows) | 1M tokens              |
| Max output                                                                                 | 128K tokens            |
| [Thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)              | Adaptive (always on)   |
| [Default effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)          | `high`                 |
| Input → output                                                                             | Text and images → text |
| Reliable knowledge cutoff                                                                  | Jun 2026               |
| Training data cutoff                                                                       | Jun 2026               |

### Availability

| Feature                                                                          | Value                                                                                                                                                                                                                                                                                                                                                                               |
| :------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Status](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) | Active (verification required)                                                                                                                                                                                                                                                                                                                                                      |
| Released                                                                         | September 1, 2026                                                                                                                                                                                                                                                                                                                                                                   |
| Retirement                                                                       | Not sooner than September 1, 2027                                                                                                                                                                                                                                                                                                                                                   |
| Platforms                                                                        | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

## 资源

<CardGroup cols={3}>
  <Card title="迁移到 Claude Mythos 5.2" icon="arrows-left-right" href="https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-mythos-5-1-to-claude-mythos-5-2">
    从 Claude Mythos 5.1 迁移到 Claude Mythos 5.2 时会发生哪些变化。
  </Card>

  <Card title="Claude Mythos 5.2" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview">
    当前的 Mythos 模型：概述、规格和资源。
  </Card>
</CardGroup>

## 参考资料

<CardGroup cols={3}>
  <Card title="系统卡" icon="file" href="https://www.anthropic.com/claude-fable-5-1-mythos-5-1-system-card">
    Claude Fable 5.1 和 Claude Mythos 5.1 的安全评估与部署决策。
  </Card>

  <Card title="定价" icon="coins" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
    完整价格列表，包括批量折扣和提示缓存费率。
  </Card>

  <Card title="模型 ID 与版本管理" icon="fingerprint" href="https://platform.claude.com/docs/zh-CN/about-claude/models/model-ids-and-versions">
    模型 ID、别名和固定快照的工作原理。
  </Card>

  <Card title="模型弃用" icon="clock" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    每个 Claude 模型的生命周期状态和退役承诺。
  </Card>
</CardGroup>
