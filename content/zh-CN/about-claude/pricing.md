---
title: 定价
url: https://platform.claude.com/docs/zh-CN/about-claude/pricing
description: 了解 Anthropic 模型和功能的定价结构
---

本页面提供 Anthropic 模型和功能的详细定价信息。所有价格均以美元计价。

如需获取最新的定价信息，请参阅 [Claude 定价](https://claude.com/pricing)页面。

## 模型定价

下表显示了所有 Claude 模型的定价：

| Model                                                                                                                                                                     | Base input tokens     | 5m cache writes | 1h cache writes | Cache hits and refreshes | Output tokens          |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :-------------------- | :-------------- | :-------------- | :----------------------- | :--------------------- |
| Claude Fable 5.2                                                                                                                                                          | $8 / MTok             | $10 / MTok      | $16 / MTok      | $0.20 / MTok<sup>1</sup> | $40 / MTok             |
| Claude Mythos 5.2 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                               | $8 / MTok             | $10 / MTok      | $16 / MTok      | $0.20 / MTok<sup>1</sup> | $40 / MTok             |
| Claude Fable 5.1                                                                                                                                                          | $10 / MTok            | $12.50 / MTok   | $20 / MTok      | $0.25 / MTok<sup>1</sup> | $50 / MTok             |
| Claude Mythos 5.1 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                               | $10 / MTok            | $12.50 / MTok   | $20 / MTok      | $0.25 / MTok<sup>1</sup> | $50 / MTok             |
| Claude Fable 5                                                                                                                                                            | $10 / MTok            | $12.50 / MTok   | $20 / MTok      | $1 / MTok                | $50 / MTok             |
| Claude Mythos 5 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                                 | $10 / MTok            | $12.50 / MTok   | $20 / MTok      | $1 / MTok                | $50 / MTok             |
| Claude Opus 5.5                                                                                                                                                           | $4 / MTok             | $5 / MTok       | $8 / MTok       | $0.20 / MTok<sup>2</sup> | $20 / MTok             |
| Claude Opus 5                                                                                                                                                             | $5 / MTok             | $6.25 / MTok    | $10 / MTok      | $0.50 / MTok             | $25 / MTok             |
| Claude Opus 4.8                                                                                                                                                           | $5 / MTok             | $6.25 / MTok    | $10 / MTok      | $0.50 / MTok             | $25 / MTok             |
| Claude Opus 4.7                                                                                                                                                           | $5 / MTok             | $6.25 / MTok    | $10 / MTok      | $0.50 / MTok             | $25 / MTok             |
| Claude Opus 4.6                                                                                                                                                           | $5 / MTok             | $6.25 / MTok    | $10 / MTok      | $0.50 / MTok             | $25 / MTok             |
| Claude Opus 4.5                                                                                                                                                           | $5 / MTok             | $6.25 / MTok    | $10 / MTok      | $0.50 / MTok             | $25 / MTok             |
| Claude Opus 4.1 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | $15 / MTok            | $18.75 / MTok   | $30 / MTok      | $1.50 / MTok             | $75 / MTok             |
| Claude Opus 4 ([retired, except on Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))               | $15 / MTok            | $18.75 / MTok   | $30 / MTok      | $1.50 / MTok             | $75 / MTok             |
| Claude Sonnet 5.5                                                                                                                                                         | $2 / MTok             | $2.50 / MTok    | $4 / MTok       | $0.10 / MTok<sup>2</sup> | $10 / MTok             |
| Claude Sonnet 5                                                                                                                                                           | $2 / MTok<sup>3</sup> | $2.50 / MTok    | $4 / MTok       | $0.20 / MTok             | $10 / MTok<sup>3</sup> |
| Claude Sonnet 4.6                                                                                                                                                         | $3 / MTok             | $3.75 / MTok    | $6 / MTok       | $0.30 / MTok             | $15 / MTok             |
| Claude Sonnet 4.5                                                                                                                                                         | $3 / MTok             | $3.75 / MTok    | $6 / MTok       | $0.30 / MTok             | $15 / MTok             |
| Claude Sonnet 4 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | $3 / MTok             | $3.75 / MTok    | $6 / MTok       | $0.30 / MTok             | $15 / MTok             |
| Claude Haiku 5.5 (for prompts up to 100,000 tokens)                                                                                                                       | $0.10 / MTok          | $0.125 / MTok   | $0.20 / MTok    | $0.01 / MTok             | $0.50 / MTok           |
| Claude Haiku 5.5 (for prompts over 100,000 tokens)                                                                                                                        | $0.50 / MTok          | $0.625 / MTok   | $1 / MTok       | $0.05 / MTok             | $2.50 / MTok           |
| Claude Haiku 4.5                                                                                                                                                          | $1 / MTok             | $1.25 / MTok    | $2 / MTok       | $0.10 / MTok             | $5 / MTok              |

*<sup>1 Cache hits and refreshes on Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, and Claude Mythos 5.1 are priced at 0.025x the base input price.</sup>*

*<sup>2 Cache hits and refreshes on Claude Opus 5.5 and Claude Sonnet 5.5 are priced at 0.05x the base input price.</sup>*

*<sup>All other models use the standard 0.1x multiplier.</sup>*

*<sup>3 The $2/$10 per million input/output token pricing for Claude Sonnet 5, announced at launch as introductory pricing through August 31, 2026, is now the standard price. The previously scheduled increase to $3/$15 per million input/output tokens on September 1, 2026 will not occur.</sup>*

* **MTok:** Million tokens. $5 / MTok is $5 for every million tokens.
* **5m cache writes:** Writing a prompt prefix to the 5-minute prompt cache.
* **1h cache writes:** Writing a prompt prefix to the 1-hour prompt cache.
* **Cache hits and refreshes:** Reading a prompt prefix from the prompt cache, which also refreshes it.
* **Limited access:** Available only to organizations verified through Anthropic’s verification programs, such as the [Cyber Verification Program](https://support.claude.com/en/articles/14604842). To request access, apply to the program that covers your use case.
* **Retired:** May still be available on other cloud platforms. See [Model deprecations](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) for more.

Claude 4.7 and later models and Claude Mythos Preview use a newer tokenizer that contributes to their improved performance on a wide range of tasks. This tokenizer produces approximately 30% more tokens for the same text. The exact increase depends on the content and workload shape. Claude Sonnet 4.6 and earlier models use the previous tokenizer.

## 云平台定价

本节介绍由合作伙伴运营的云平台，这些平台由云服务提供商向您开具发票。对于通过市场计费、由 Anthropic 运营的云平台，请参阅 [Claude Platform on AWS 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#claude-platform-on-aws-pricing)、[Claude Models on Google Cloud (Anthropic managed) 计费](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#billing)和 [Claude in Microsoft Foundry 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#claude-in-microsoft-foundry-pricing)。

如需了解 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 上的官方定价，请访问：

* [Amazon Bedrock 定价](https://aws.amazon.com/bedrock/pricing/)
* [Google Cloud 定价](https://cloud.google.com/vertex-ai/generative-ai/pricing#claude-models)

<Note>
  **Claude 4.5 及后续模型的区域和多区域端点定价**

  从 Claude Sonnet 4.5、Haiku 4.5 和 Opus 4.5 开始：

  * **Bedrock** 提供两种端点类型：全局端点（动态路由以实现最大可用性）和区域端点（保证数据通过特定地理区域路由）。
  * **Claude Models on Google Cloud (Google managed)** 为这些模型提供全局端点和多区域端点（在某一地理区域内动态路由），并为 Claude Sonnet 4.6 及更早的模型提供区域端点。

  区域端点和多区域端点的价格比全局端点高出 10%。Claude API（第一方）默认为全局；有关第一方数据驻留选项和定价，请参阅[数据驻留定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#data-residency-pricing)。

  **适用范围：** 此定价结构适用于 Claude Sonnet 4.5（已弃用）、Haiku 4.5、Opus 4.5 及更高版本的模型。它不适用于较早的模型（Claude Opus 4.1 及之前的版本）。

  有关实现细节和代码示例：

  * Amazon Bedrock 全局端点与区域端点：根据您调用的是 `bedrock-mantle` 还是 `bedrock-runtime`，请参阅 [Amazon Bedrock 中的 Claude（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#regions) 或 [Amazon Bedrock 上的 Claude（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#global-vs-regional-endpoints)
  * [Claude Models on Google Cloud (Google managed) 全局、多区域和区域端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#global-multi-region-and-regional-endpoints)
</Note>

## Claude Platform on AWS 定价

[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 通过 AWS Marketplace 使用 Claude Consumption Units（CCU，Claude 消耗单位）进行计费。Anthropic 按照标准的各模型、各功能费率以美元计算您的令牌用量，应用任何协商折扣，然后按每 CCU 0.01 美元将结果转换为 CCU，并每小时向 AWS Marketplace 报告 CCU 数量。您的 AWS 账单显示单个 CCU 明细项目。

| 概念         | 详情                                                                                                                                              |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **计费单位**   | Claude Consumption Unit（CCU）                                                                                                                    |
| **CCU 价格** | 每 CCU 0.01 美元（固定；折扣在令牌到 CCU 的转换时应用，而不是应用于 CCU 价格）                                                                                               |
| **转换**     | 令牌用量按照标准的各模型、各功能费率以美元计价（与 [Claude API 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#model-pricing)相同），然后按每 CCU 0.01 美元转换为 CCU |
| **计费周期**   | 每小时向 AWS Marketplace 计量；按月开具发票                                                                                                                  |
| **付款模式**   | 仅后付费；不提供预付额度                                                                                                                                    |
| **折扣**     | 以减少计量的 CCU 数量的方式应用                                                                                                                              |
| **税费**     | 税前计量；由 AWS Marketplace 处理税费                                                                                                                     |
| **成本可见性**  | 在 Claude Console 中提供实时明细（通过 AWS Console 访问）；AWS Cost Explorer 显示汇总的 CCU                                                                         |

**CCU:** Claude Consumption Unit（Claude 消耗单位）。CCU 是仅用于 Marketplace 平台开具发票的计量单位。如果客户通过某些 Marketplace 平台（例如 Claude Platform on AWS）访问服务，则用量将以 Claude Consumption Units（"CCU"）而非按 MTok 开具发票。一百（100）CCU 代表服务应付费用 1.00 美元，该费用按照 [claude.com/pricing#api](https://claude.com/pricing#api) 上的适用价格计算，并已应用任何折扣。

### 推理地理位置

对于 Claude 4.6 及更高版本的模型，使用 `inference_geo: "us"` 会应用 1.1 倍的定价乘数。`inference_geo: "global"`（默认）使用标准定价。详情请参阅[数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)。

### 私有报价

当您在 AWS Console 的 **Claude Platform on AWS** 服务页面上注册时，AWS Console 会查找与您的账户关联的任何私有报价，并提示您在 AWS Marketplace 中接受该报价。有关私有报价条款，请联系您的 Anthropic 客户代表。

<Note>
  如果您已有 Amazon Bedrock 私有报价，请在开始使用 Claude Platform on AWS 之前联系您的 Anthropic 或 AWS 客户代表，以确保您的折扣得到正确应用。折扣无法追溯应用于在接受私有报价之前产生的用量。
</Note>

## Claude in Microsoft Foundry 定价

[Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 通过 Azure Marketplace 使用 Claude Consumption Units（CCU）进行计费。Anthropic 按照标准的各模型、各功能费率以美元计算您的令牌用量，应用任何协商折扣，然后按每 CCU 0.01 美元将结果转换为 CCU，并每小时向 Azure Marketplace 报告 CCU 数量。您的 Azure 账单显示单个 CCU 明细项目。

| 概念         | 详情                                                                                                                                              |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **计费单位**   | Claude Consumption Unit（CCU）                                                                                                                    |
| **CCU 价格** | 每 CCU 0.01 美元（固定；折扣在令牌到 CCU 的转换时应用，而不是应用于 CCU 价格）                                                                                               |
| **转换**     | 令牌用量按照标准的各模型、各功能费率以美元计价（与 [Claude API 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#model-pricing)相同），然后按每 CCU 0.01 美元转换为 CCU |
| **计费周期**   | 每小时向 Azure Marketplace 计量；按月开具发票                                                                                                                |
| **付款模式**   | 仅后付费；不提供预付额度                                                                                                                                    |
| **折扣**     | 以减少计量的 CCU 数量的方式应用                                                                                                                              |
| **税费**     | 税前计量；由 Azure Marketplace 处理税费                                                                                                                   |
| **成本可见性**  | Azure Cost Management 显示汇总的 CCU                                                                                                                 |

**CCU:** Claude Consumption Unit（Claude 消耗单位）。CCU 是仅用于 Marketplace 平台开具发票的计量单位。如果客户通过某些 Marketplace 平台（例如 Claude Platform on AWS、Claude in Microsoft Foundry）访问服务，则用量将以 Claude Consumption Units（"CCU"）而非按 MTok 开具发票。一百（100）CCU 代表服务应付费用 1.00 美元，该费用按照 [claude.com/pricing#api](https://claude.com/pricing#api) 上的适用价格计算，并已应用任何折扣。

### 推理地理位置

托管在 Azure 上的部署可以使用 US Data Zone Standard 部署类型，该类型将推理保持在美国境内。这相当于 Claude API 上的 `inference_geo: "us"`，并应用相同的 1.1 倍定价乘数。详情请参阅[数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)。

## 特定功能定价

### 提示缓存

“Prompt caching”（提示缓存）通过在 API 调用之间重用先前已处理的提示部分来降低成本和延迟。API 不必在每个请求中重新处理相同的大型系统提示、文档或对话历史记录，而是以标准输入价格的一小部分从缓存中读取。

有两种方式可以启用提示缓存：

* **自动缓存：** 在请求的顶层添加单个 `cache_control` 字段。系统会随着对话的增长自动管理缓存断点。这是大多数用例的推荐起点。
* **显式缓存断点：** 将 `cache_control` 直接放在各个内容块上，以精细控制具体缓存哪些内容。

提示缓存使用以下相对于基础输入令牌费率的定价乘数：

| 缓存操作     | 乘数                   | 持续时间         |
| -------- | -------------------- | ------------ |
| 5 分钟缓存写入 | 基础输入价格的 1.25 倍       | 缓存有效期为 5 分钟  |
| 1 小时缓存写入 | 基础输入价格的 2 倍          | 缓存有效期为 1 小时  |
| 缓存读取（命中） | 大多数模型上为基础输入价格的 0.1 倍 | 与之前的写入持续时间相同 |

缓存写入令牌在内容首次存储时收费。缓存读取令牌在后续请求检索缓存内容时收费。

少量缓存读取后，缓存即可节省成本：

* **5 分钟持续时间（1.25 倍写入）：** 一次缓存读取后
* **1 小时持续时间（2 倍写入）：** 两次缓存读取后

缓存命中的费用取决于模型：

| 模型                                   | 缓存命中乘数  | 每百万令牌的缓存命中价格                                                                         |
| ------------------------------------ | ------- | ------------------------------------------------------------------------------------ |
| Claude Fable 5.2 和 Claude Mythos 5.2 | 0.025 倍 | 0.20 美元                                                                              |
| Claude Fable 5.1 和 Claude Mythos 5.1 | 0.025 倍 | 0.25 美元                                                                              |
| Claude Opus 5.5                      | 0.05 倍  | 0.20 美元                                                                              |
| Claude Sonnet 5.5                    | 0.05 倍  | 0.10 美元                                                                              |
| 所有其他模型                               | 0.1 倍   | 请参阅[模型定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#model-pricing) |

这些乘数会与其他定价调整因素叠加，包括 Batch API 折扣和数据驻留。

有关实现细节、支持的模型和代码示例，请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)。

### 数据驻留定价

对于 Claude 4.6 及更高版本的模型，通过 `inference_geo` 参数指定仅限美国的推理会对所有令牌定价类别应用 1.1 倍乘数，包括输入令牌、输出令牌、缓存写入和缓存读取。在 Claude Haiku 5.5 上，该乘数也适用于超过 100,000 个令牌的提示的较高价格（请参阅[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)）。全局路由（默认）使用标准定价。

这适用于 Claude API（第一方）、Claude Platform on AWS 和 Claude Models on Google Cloud (Anthropic managed)。在 Claude in Microsoft Foundry 上，相同的 1.1 倍乘数适用于使用 US Data Zone Standard 部署类型的部署（请参阅[推理地理位置](https://platform.claude.com/docs/zh-CN/about-claude/pricing#foundry-inference-geography)）。由合作伙伴运营的平台，即 Claude Models on Google Cloud (Google managed) 和 Bedrock，有独立的区域定价。详情请参阅 [Amazon Bedrock 定价](https://aws.amazon.com/bedrock/pricing/)和 [Google Cloud 定价](https://cloud.google.com/vertex-ai/generative-ai/pricing#claude-models)。较早的模型不支持 `inference_geo` 参数，始终使用标准定价；在这些模型上包含该参数的请求会返回 400 错误。

有关更多信息，请参阅[数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)。

### 快速模式定价

[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)以溢价定价提供最高 2.5 倍的每秒输出令牌数。它适用于 Claude Fable 5.2、Claude Opus 5.5、Claude Opus 5 和 Claude Opus 4.8。Claude Opus 5 的快速模式处于研究预览阶段，Claude Opus 4.8 的快速模式已弃用。

快速模式定价适用于整个上下文窗口，包括超过 200k 输入令牌的请求。

快速模式可在 Claude API（第一方）上使用。对于 Claude Opus 5.5，还可在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#billing) 以及 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上托管在 Azure 的部署中使用。

Claude Fable 5.2 的快速模式还可在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#billing) 上使用。

Claude Fable 5.2 的快速模式还可在 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 上使用。该平台上的价格由 Google Cloud 设定。请参阅[云平台定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)。

Claude Fable 5.2 的快速模式还可在 Microsoft Foundry 中的 Claude 上托管在 Anthropic 的部署中使用。请参阅 [Claude in Microsoft Foundry 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#claude-in-microsoft-foundry-pricing)。

快速模式还可用于 Claude API 上的 Claude Mythos 5.2。[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出了 Claude Mythos 5.2 支持快速模式的所有平台。

| 模型                              | 输入           | 输出           |
| ------------------------------- | ------------ | ------------ |
| Claude Fable 5.2                | 16 美元 / MTok | 80 美元 / MTok |
| Claude Mythos 5.2               | 16 美元 / MTok | 80 美元 / MTok |
| Claude Opus 5.5                 | 8 美元 / MTok  | 40 美元 / MTok |
| Claude Opus 5 / Claude Opus 4.8 | 10 美元 / MTok | 50 美元 / MTok |

快速模式定价会与其他定价调整因素叠加：

* [提示缓存乘数](https://platform.claude.com/docs/zh-CN/about-claude/pricing#prompt-caching)在快速模式定价的基础上应用。
* [数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)乘数在快速模式定价的基础上应用。

快速模式不适用于 [Batch API](https://platform.claude.com/docs/zh-CN/about-claude/pricing#batch-processing)。

有关更多信息，请参阅[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。

### 批处理

Batch API 允许异步处理大量请求，输入和输出令牌均可享受 50% 的折扣。

| Model                                                                                                                                                                     | Batch input  | Batch output  |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------- | :------------ |
| Claude Fable 5.2                                                                                                                                                          | $4 / MTok    | $20 / MTok    |
| Claude Mythos 5.2 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                               | $4 / MTok    | $20 / MTok    |
| Claude Fable 5.1                                                                                                                                                          | $5 / MTok    | $25 / MTok    |
| Claude Mythos 5.1 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                               | $5 / MTok    | $25 / MTok    |
| Claude Fable 5                                                                                                                                                            | $5 / MTok    | $25 / MTok    |
| Claude Mythos 5 ([limited availability](https://support.claude.com/en/articles/14604842))                                                                                 | $5 / MTok    | $25 / MTok    |
| Claude Opus 5.5                                                                                                                                                           | $2 / MTok    | $10 / MTok    |
| Claude Opus 5                                                                                                                                                             | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.8                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.7                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.6                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.5                                                                                                                                                           | $2.50 / MTok | $12.50 / MTok |
| Claude Opus 4.1 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | $7.50 / MTok | $37.50 / MTok |
| Claude Opus 4 ([retired, except on Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))               | $7.50 / MTok | $37.50 / MTok |
| Claude Sonnet 5.5                                                                                                                                                         | $1 / MTok    | $5 / MTok     |
| Claude Sonnet 5                                                                                                                                                           | $1 / MTok    | $5 / MTok     |
| Claude Sonnet 4.6                                                                                                                                                         | $1.50 / MTok | $7.50 / MTok  |
| Claude Sonnet 4.5                                                                                                                                                         | $1.50 / MTok | $7.50 / MTok  |
| Claude Sonnet 4 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | $1.50 / MTok | $7.50 / MTok  |
| Claude Haiku 5.5 (for prompts up to 100,000 tokens)                                                                                                                       | $0.05 / MTok | $0.25 / MTok  |
| Claude Haiku 5.5 (for prompts over 100,000 tokens)                                                                                                                        | $0.25 / MTok | $1.25 / MTok  |
| Claude Haiku 4.5                                                                                                                                                          | $0.50 / MTok | $2.50 / MTok  |

* **MTok:** Million tokens. $5 / MTok is $5 for every million tokens.
* **Limited access:** Available only to organizations verified through Anthropic’s verification programs, such as the [Cyber Verification Program](https://support.claude.com/en/articles/14604842). To request access, apply to the program that covers your use case.
* **Retired:** May still be available on other cloud platforms. See [Model deprecations](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) for more.

有关批处理的更多信息，请参阅[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)。

### 长上下文定价

Claude 4.6 及更高版本的模型（Claude Haiku 5.5 除外）和 [Claude Mythos Preview](https://anthropic.com/glasswing) 以标准定价包含完整的 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)。（900k 令牌的请求与 9k 令牌的请求按相同的每令牌费率计费。）提示缓存和批处理折扣在整个上下文窗口中按标准费率应用。

Claude Haiku 5.5 按提示长度定价：提示超过 100,000 个令牌的请求需支付较高的价格。请求的提示长度计入其所有输入令牌，包括缓存读取和缓存写入。每个请求单独定价：超过阈值的请求即使其部分提示为缓存命中，也需支付较高的价格，而之前的请求保持其计费时的价格。[模型定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#model-pricing)和[批处理定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#batch-processing)部分列出了两组价格。

### 工具使用定价

工具使用请求的定价基于：

1. 发送给模型的输入令牌总数（包括 `tools` 参数中的令牌）。
2. 生成的输出令牌数量。
3. 对于服务器端工具，还有额外的基于用量的定价（例如，网页搜索按每次执行的搜索收费）。

客户端工具的定价与任何其他 Claude API 请求相同，但服务器端工具可能会根据其具体用量产生额外费用。

工具使用产生的额外令牌来自：

* API 请求中的 `tools` 参数（工具名称、描述和模式）
* API 请求和响应中的 `tool_use` 内容块
* API 请求中的 `tool_result` 内容块

当您使用 `tools` 时，API 还会自动为模型包含一个启用工具使用的特殊系统提示。下表列出了每个模型所需的工具使用令牌数量（不包括前面列出的额外令牌）。该表假设您至少提供了 1 个工具。如果您未提供 `tools`，则工具选择为 `none` 时使用 0 个额外的系统提示令牌。

| Model                                                                                                                                                                     | Tool use system prompt tokens: auto, none | Tool use system prompt tokens: any, tool |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------------- | :--------------------------------------- |
| Claude Opus 5.5                                                                                                                                                           | 286 tokens                                |                                          |
| Claude Opus 5                                                                                                                                                             | 286 tokens                                | 406 tokens                               |
| Claude Opus 4.8                                                                                                                                                           | 290 tokens                                | 410 tokens                               |
| Claude Opus 4.7                                                                                                                                                           | 675 tokens                                | 804 tokens                               |
| Claude Opus 4.6                                                                                                                                                           | 497 tokens                                | 589 tokens                               |
| Claude Opus 4.5                                                                                                                                                           | 496 tokens                                | 588 tokens                               |
| Claude Opus 4.1 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | 313 tokens                                | 315 tokens                               |
| Claude Opus 4 ([retired, except on Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))               | 313 tokens                                | 315 tokens                               |
| Claude Sonnet 5.5                                                                                                                                                         | 286 tokens                                |                                          |
| Claude Sonnet 5                                                                                                                                                           | 354 tokens                                | 474 tokens                               |
| Claude Sonnet 4.6                                                                                                                                                         | 497 tokens                                | 589 tokens                               |
| Claude Sonnet 4.5                                                                                                                                                         | 496 tokens                                | 588 tokens                               |
| Claude Sonnet 4 ([retired, except on Claude Models on Google Cloud (Google managed) and Bedrock](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | 313 tokens                                | 315 tokens                               |
| Claude Haiku 5.5                                                                                                                                                          | 286 tokens                                | 406 tokens                               |
| Claude Haiku 4.5                                                                                                                                                          | 496 tokens                                | 588 tokens                               |

* **auto, none:** The count when tool\_choice is auto or none.
* **any, tool:** The count when tool\_choice is any or tool.
* **Retired:** May still be available on other cloud platforms. See [Model deprecations](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations) for more.

这些令牌数会加到您正常的输入和输出令牌中，以计算请求的总费用。

有关当前各模型的价格，请参阅[模型定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#model-pricing)部分。

有关工具使用实现和最佳实践的更多信息，请参阅[使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)。

### 特定工具定价

#### Bash 工具

bash 工具定义会向您的请求中添加以下 input tokens（输入令牌）。这是在每个模型的[工具使用系统提示](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#pricing)之外额外添加的，只要存在任何工具，该系统提示就会生效。

| 模型                                        | 额外输入令牌  |
| ----------------------------------------- | ------- |
| Claude 4.7 及更高版本模型和 Claude Mythos Preview | 325 个令牌 |
| Claude 4.6 及更早版本模型                        | 244 个令牌 |

以下内容会消耗额外的令牌：

* 命令输出（stdout/stderr）
* 错误消息
* 大型文件内容

有关完整的定价详情，请参阅[工具使用定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#tool-use-pricing)。

#### 代码执行工具

**与 web search（网页搜索）或 web fetch（网页抓取）一起使用时，代码执行是免费的。** 当您的 API 请求中包含 `web_search_20260209`（或更高版本）或 `web_fetch_20260209`（或更高版本）时，除标准的输入和输出令牌费用外，代码执行工具调用不会产生额外费用。

在不与这些工具一起使用时，代码执行按执行时间计费，并与令牌用量分开跟踪：

* 执行时间最少按 5 分钟计算。
* 每个组织每月可获得 **1,550 小时免费**使用时长。
* 超出 1,550 小时的额外使用按**每个容器每小时 0.05 美元**计费。
* 如果请求中包含文件，即使未调用该工具，也会对执行时间计费，因为文件会预先加载到容器中。

代码执行用量会在响应中进行跟踪：

```json
{
  "usage": {
    "input_tokens": 105,
    "output_tokens": 239,
    "server_tool_use": {
      "code_execution_requests": 1
    }
  }
}
```

#### 文本编辑器工具

文本编辑器工具采用与 Claude 配合使用的其他工具相同的定价结构。它遵循基于您所使用的 Claude 模型的标准输入和输出令牌（token）定价。

除基础令牌外，文本编辑器工具还需要以下额外的输入令牌：

| 工具                                                                 | 额外输入令牌  |
| ------------------------------------------------------------------ | ------- |
| `text_editor_20250728`（Claude 4.7 及更高版本模型以及 Claude Mythos Preview） | 974 个令牌 |
| `text_editor_20250728`（Claude 4.6 及更早版本模型）                         | 745 个令牌 |

有关完整的定价详情，请参阅[工具使用定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#tool-use-pricing)。

#### 网络搜索工具

除令牌用量外，网页搜索的使用会额外计费：

```json
{
  "usage": {
    "input_tokens": 105,
    "output_tokens": 6039,
    "cache_read_input_tokens": 7123,
    "cache_creation_input_tokens": 7345,
    "server_tool_use": {
      "web_search_requests": 1
    }
  }
}
```

网络搜索在 Claude API 上的价格为**每 1,000 次搜索 10 美元**，另加搜索生成内容的标准令牌费用。在整个对话过程中检索到的网络搜索结果都会计为输入令牌，包括在单个轮次中运行的搜索迭代以及后续的对话轮次。

每次网页搜索计为一次使用，与返回的结果数量无关。因错误而失败的网页搜索不会计费。

#### 网页获取工具

Web fetch（网页抓取）的使用除标准令牌费用外**不收取任何额外费用**：

```json
{
  "usage": {
    "input_tokens": 25039,
    "output_tokens": 931,
    "cache_read_input_tokens": 0,
    "cache_creation_input_tokens": 0,
    "server_tool_use": {
      "web_fetch_requests": 1
    }
  }
}
```

Web fetch 工具在 Claude API 上**无需额外费用**即可使用。您只需为成为对话上下文一部分的抓取内容支付标准令牌费用。

为防止无意中抓取会消耗过多令牌的大型内容，请使用 `max_content_tokens` 参数，根据您的使用场景和预算考虑设置适当的限制。

典型内容的令牌使用量示例：

* 普通网页（10 kB）：约 2,500 个令牌
* 大型文档页面（100 kB）：约 25,000 个令牌
* 研究论文 PDF（500 kB）：约 125,000 个令牌

#### 计算机使用工具

计算机使用遵循标准的[工具使用定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#pricing)。使用计算机使用工具时：

**工具集定义开销：** 声明带有默认成员的 `computer_toolset_20260801` 会为请求增加约 4,500 个输入令牌（在 Claude Sonnet 5 上约为 4,590 个，在其他受支持的模型上约为 4,520 个），其中涵盖成员工具定义和工具使用系统提示。使用 `configs` 禁用 `zoom` 可减少其中约 410 个令牌。请求的确切令牌数会在响应的 `usage` 中报告，您也可以使用[令牌计数端点](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)提前进行估算。

**早期工具版本：** 以下数据适用于 `computer_20251124` 和 `computer_20250124` 工具版本，不适用于 `computer_toolset_20260801`：

* 系统提示开销：向系统提示中添加 466–499 个令牌
* 工具定义：每个工具定义约 735 个输入令牌（使用 `computer_20250124` 测量）

**额外令牌消耗：**

* 工具结果中返回的屏幕截图和缩放图像，按图像输入计费（请参阅[视觉定价](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)）
* 返回给 Claude 的工具执行结果

<Note>
  如果您在计算机使用的同时还使用 bash 或文本编辑器工具，这些工具有各自的令牌成本，详见其各自的文档页面。
</Note>

#### 浏览器使用工具

浏览器使用遵循标准的[工具使用定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#pricing)。使用浏览器使用工具时：

**工具集定义开销：** 声明带有默认成员的 `browser_toolset_20260801` 会为请求增加约 6,600 个输入令牌（在 Claude Sonnet 5 上约为 6,670 个，在其他受支持的模型上约为 6,610 个），其中涵盖成员工具定义和工具使用系统提示。启用全部四个可选成员会增加约 880 个令牌，而使用 `configs` 禁用成员则会减少令牌数。请求的确切令牌数会在响应的 `usage` 中报告，您也可以使用[令牌计数端点](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)提前进行估算。

**额外的令牌消耗：**

* 工具结果中返回的屏幕截图和缩放图像，按图像输入计费（请参阅[视觉定价](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)）
* 返回给 Claude 的文本工具结果，例如无障碍树、页面文本以及控制台或网络条目

<Note>
  如果您在浏览器使用之外还同时使用计算机使用工具、bash 工具、文本编辑器工具或您自己的工具，这些工具各自有其令牌成本，详见其各自的文档页面。
</Note>

## Claude Managed Agents 定价

[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 按两个维度计费：令牌和会话运行时间。

### 令牌

Claude Managed Agents 会话消耗的所有令牌均按[模型定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#model-pricing)中显示的费率计费。[提示缓存乘数](https://platform.claude.com/docs/zh-CN/about-claude/pricing#prompt-caching)同样适用。在会话中触发的网络搜索按标准的每 1,000 次搜索 10 美元收费。在 Claude Platform on AWS 上，会话令牌和运行时间费用按标准费率转换为 Claude Consumption Units（请参阅 [Claude Platform on AWS 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#claude-platform-on-aws-pricing)）。当代理的 `model.speed` 设置为 `"fast"` 时，适用[快速模式溢价定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)。

[数据驻留乘数](https://platform.claude.com/docs/zh-CN/about-claude/pricing#data-residency-pricing)同样适用：当代理的 `model.inference_geo` 固定为 `"us"` 时，运行该代理的会话所消耗的令牌按标准费率的 1.1 倍计费，这与 Messages API 上仅限美国推理所适用的乘数相同。

以下 Messages API 调整因素**不**适用于 Claude Managed Agents 会话：

| 调整因素                                                                                         | 不适用的原因               |
| -------------------------------------------------------------------------------------------- | -------------------- |
| [Batch API 折扣](https://platform.claude.com/docs/zh-CN/about-claude/pricing#batch-processing) | 会话是有状态且交互式的。没有批处理模式。 |
| [云平台定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)  | 在由合作伙伴运营的云平台上不可用。    |

### 会话运行时间

| SKU    | 费率            | 计量               |
| ------ | ------------- | ---------------- |
| 会话运行时间 | 每会话小时 0.08 美元 | `running` 状态持续时间 |

运行时间精确到毫秒计量，并且仅在会话状态为 `running` 时累计。处于 `idle`（等待您的下一条消息或工具确认）、`rescheduling` 或 `terminated` 状态的时间不计入运行时间。

<Note>
  使用 Claude Managed Agents 时，会话运行时间取代了[代码执行](https://platform.claude.com/docs/zh-CN/about-claude/pricing#code-execution-tool)的容器小时计费模式。除会话运行时间外，您不会另外为容器小时付费。
</Note>

### 计算示例

一个使用 Claude Opus 5.5 的 1 小时编码会话，消耗 50,000 个输入令牌和 15,000 个输出令牌：

| 明细项目   | 计算                       | 费用          |
| ------ | ------------------------ | ----------- |
| 输入令牌   | 50,000 × $4 / 1,000,000  | 0.20 美元     |
| 输出令牌   | 15,000 × $20 / 1,000,000 | 0.30 美元     |
| 会话运行时间 | 1.0 小时 × $0.08           | 0.08 美元     |
| **总计** |                          | **0.58 美元** |

如果启用了提示缓存，并且其中 40,000 个输入令牌为缓存读取：

| 明细项目     | 计算                             | 费用           |
| -------- | ------------------------------ | ------------ |
| 未缓存的输入令牌 | 10,000 × $4 / 1,000,000        | 0.04 美元      |
| 缓存读取令牌   | 40,000 × $4 × 0.05 / 1,000,000 | 0.008 美元     |
| 输出令牌     | 15,000 × $20 / 1,000,000       | 0.30 美元      |
| 会话运行时间   | 1.0 小时 × $0.08                 | 0.08 美元      |
| **总计**   |                                | **0.428 美元** |

## 其他定价注意事项

### 成本优化策略

使用 Claude 构建代理时：

1. **使用合适的模型：** 简单任务选择 Haiku，大多数生产工作负载选择 Sonnet，最复杂的推理选择 Opus。
2. **实施提示缓存：** 降低重复上下文的成本。
3. **批量操作：** 对非时间敏感的任务使用 Batch API。
4. **监控使用模式：** 跟踪令牌消耗以发现优化机会。

<Tip>
  对于大批量的代理应用程序，请联系[企业销售团队](https://claude.com/contact-sales)以获取定制定价安排。
</Tip>

### 速率限制

“Rate limit”（速率限制）因使用层级而异，并影响您可以发出的请求数量：

* **起始等级：** 用于入门的初级限制
* **Build 层级：** 为成长中的应用程序提供更高的限制
* **Scale 层级：** 为生产工作负载提供最高的标准限制

有关详细的速率限制信息，请参阅[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)。

如需超出 Scale 层级的限制或定制定价安排，请[联系销售团队](https://claude.com/contact-sales)。

### 批量折扣

大用量用户可能可以享受批量折扣。这些折扣根据具体情况协商确定。

* 标准使用层级使用[模型定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#model-pricing)中显示的定价。
* 企业客户可以[联系销售](mailto:sales@anthropic.com)获取定制定价。
* 可能提供学术和研究折扣。

### 企业定价

适用于有特定需求的企业客户：

* 定制速率限制
* 批量折扣
* 专属支持
* 定制条款

请通过 [sales@anthropic.com](mailto:sales@anthropic.com) 或 [Claude Console 中的速率限制](https://platform.claude.com/usage/limits)联系销售团队，讨论企业定价选项。

## 计费和付款

* 计费基于实际月度用量。
* 所有付款均以美元计价。
* 提供信用卡和发票付款选项。
* 可在 [Claude Console](https://platform.claude.com/) 中跟踪用量。

## 常见问题

### 令牌用量是如何计算的？

令牌是模型处理的文本片段。相同文本产生多少令牌取决于模型的分词器、语言和内容类型。Claude 4.7 及更高版本的模型以及 Claude Mythos Preview（`claude-mythos-preview`）使用较新的分词器，对于相同文本，其产生的令牌比早期模型多约 30%。要查看某个请求在特定模型上使用了多少令牌，请使用[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)和该模型的 ID 进行计数。

### 是否有免费层级或试用？

新用户会获得少量免费额度用于测试 API。有关企业评估的延长试用信息，请[联系销售](mailto:sales@anthropic.com)。

### 折扣如何叠加？

Batch API 和提示缓存折扣可以组合使用。例如，同时使用这两项功能与标准 API 调用相比可显著节省成本。有关乘数如何相互作用，请参阅[提示缓存定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#prompt-caching)。

### 接受哪些付款方式？

标准账户接受主要信用卡。要购买额度或管理您的付款方式，请前往 Claude Console 中的[账单](https://platform.claude.com/settings/billing)页面（`platform.claude.com/settings/billing`）。企业客户可以安排发票付款和其他付款方式。

如有其他定价问题，请联系 [support@anthropic.com](mailto:support@anthropic.com)。
