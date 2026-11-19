---
title: 文本水印
url: https://platform.claude.com/docs/zh-CN/build-with-claude/text-watermark
description: Anthropic 的文本水印不会改变哪些内容、在哪里可以找到其涵盖的模型和平台，以及如果您依赖水印，应省略哪些采样参数。
---

受支持的 Claude 模型生成的文本带有 Anthropic 的 “text watermark”（文本水印）。[Claude 如何标记 AI 生成的内容](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)列出了其涵盖的模型和平台，[Claude 的文本水印如何工作](https://www.anthropic.com/news/claude-text-watermark)介绍了相关背景。

水印不会改变输出的含义、质量或可读性。它不会添加任何令牌或隐藏字符，也不携带任何关于您或您所在组织的信息。您无需为此更改您的请求或响应处理方式。如果您依赖水印并设置了 `temperature`、`top_p` 或 `top_k`，请参阅[采样设置](https://platform.claude.com/docs/zh-CN/build-with-claude/text-watermark#sampling-settings)。

## 采样设置

**本节适用于：** Claude Opus 4.6、Claude Sonnet 4.6、Claude Opus 4.5、Claude Sonnet 4.5、Claude Haiku 4.5、Claude Opus 3（[可申请访问模型](https://www.anthropic.com/research/deprecation-updates-opus-3)）

如果您需要输出带有水印，请在请求中省略 `temperature`、`top_p` 和 `top_k`。如果请求将其中任何一个设置为非默认值，则输出可能不带有水印。

您的框架或客户端库可能会为您设置这些参数。如果您需要水印，请检查该框架或客户端库的设置以及它发送的请求体，并移除这三个参数中由它设置的每一个。
