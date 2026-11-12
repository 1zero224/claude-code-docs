# Anthropic 是否从网络爬取数据，网站所有者如何阻止爬虫？

本页面介绍 Anthropic 的机器人（爬虫和获取器）。有关 Anthropic 托管的浏览器（Claude 用于为用户完成任务，包括在 Claude Duet (`Claude-Agent`) 中）的信息，请参阅 **[Anthropic 托管的浏览器代表用户浏览网络时如何识别自己？](https://support.claude.com/en/articles/18234133)**

## 阻止或限制 Anthropic 的机器人

根据行业标准，Anthropic 使用多种机器人从公共网络收集数据用于模型开发、网络搜索以及按用户指示检索网络内容。Anthropic 使用不同的机器人来提高网站所有者的透明度和选择权。以下是 Anthropic 使用的三个机器人的信息，以及如何设置网站偏好以允许您想要的机器人访问您的内容并限制您不想要的机器人。

| **机器人**            | **用途**                                                                       | **禁用时会发生什么**                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `ClaudeBot`        | `ClaudeBot` 通过收集可能有助于训练我们生成式 AI 模型的网络内容，帮助增强我们生成式 AI 模型的实用性和安全性。             | 当网站限制 `ClaudeBot` 访问时，这表示该网站的未来材料应被排除在我们的 AI 模型训练数据集之外。                                                         |
| `Claude-User`      | `Claude-User` 支持 Claude AI 用户。当个人向 Claude 提出问题时，它可能会使用 `Claude-User` 代理访问网站。 | `Claude-User` 允许网站所有者控制哪些网站可以通过这些用户发起的请求访问。在您的网站上禁用 `Claude-User` 可防止我们的系统根据用户查询检索您的内容，这可能会降低您网站在用户定向网络搜索中的可见性。 |
| `Claude-SearchBot` | `Claude-SearchBot` 在网络上导航以改进用户的搜索结果质量。它专门分析在线内容以增强搜索响应的相关性和准确性。              | 在您的网站上禁用 `Claude-SearchBot` 可防止我们的系统为搜索优化索引您的内容，这可能会降低您网站在用户搜索结果中的可见性和准确性。                                      |

作为我们构建安全可靠的前沿系统和推进负责任 AI 开发领域的使命的一部分，我们分享了我们收集数据的原则以及如何选择退出我们未来爬取的说明：

- 我们的数据收集应该是 *透明的*。Anthropic 使用上述机器人访问网络内容。

- 我们的爬取应该 *不* *具有* *侵入性或破坏性*。我们通过谨慎考虑爬取同一域名的速度并在适当时尊重爬取延迟来实现最小干扰。

- Anthropic 的机器人 *尊重"不爬取"* 信号，通过遵守 robots.txt 中的行业标准指令。

- Anthropic 的机器人 *尊重反规避技术*（例如，我们不会尝试绕过我们爬取的网站的验证码。）

为了限制爬取活动，我们支持 robots.txt 的非标准爬取延迟扩展。一个例子可能是：

```
User-agent: ClaudeBot
Crawl-delay: 1
```

要从您的整个网站阻止机器人，请将其添加到顶级目录中的 robots.txt 文件。请对您希望选择退出的每个子域执行此操作。一个例子是：

```
User-agent: ClaudeBot
Disallow: /
```

选择退出被 Anthropic 机器人爬取需要按上述方式修改 robots.txt 文件。其他方法，如阻止 Anthropic 机器人运行的 IP 地址，可能无法正确工作或持久保证选择退出，因为这样做会阻碍我们读取您的 robots.txt 文件的能力。如果爬虫的源 IP 地址在 **[此列表](https://claude.com/crawling/bots.json)** 上，则表示爬虫来自 Anthropic。

## Anthropic 如何尊重网站对 AI 训练的偏好

自 2026 年 10 月 30 日起，Anthropic 已开始识别一个机器可读信号，让网站所有者表达其内容是否可用于训练 AI 模型的偏好。

此公告是我们继续承诺尊重 robots.txt（其中明确命名 ClaudeBot）的承诺。我们现在确认我们将尊重一个新的偏好信号，让网站所有者说"不进行 AI 训练"。

我们正在随着时间的推移在全球范围内在我们的所有服务中推出这一功能，并将提供我们进展的更新。

该信号遵循 **[基于 IETF 工作的草案](https://datatracker.ietf.org/doc/draft-kelley-aipref-training-preference-signal)**，IETF 是制定许多互联网技术标准的组织，在其 aipref 工作组中。由于该领域的标准仍在不断发展，我们读取和应用这些设置的方式也可能会改变。

### ClaudeBot 规则如何优先

ClaudeBot 是我们的训练爬虫，我们将直接命名它的规则视为比通用信号更具体。实际上，对于给定的路径：

- **如果您的 robots.txt 文件命名了 ClaudeBot**，我们遵循该规则，即使您的通用设置说不同的内容。

- **如果您的 robots.txt 文件未提及 ClaudeBot**，我们遵循上述通用设置。

您可以在我们的 **[隐私和法律](https://support.claude.com/en/collections/4078534-privacy-and-legal)** 集合中了解更多关于我们的数据处理实践的信息。如果您有疑问或认为我们的爬虫工作不如所述，请从包含您询问的域名的电子邮件中联系 **<claudebot@anthropic.com>**。

您可以通过点击此处并填写表单来获得有关本文章重大更改的通知：

[订阅更新](https://docs.google.com/forms/d/e/1FAIpQLScQk_jt1NrI0AZU3RfBCDkCe4RBS2lAzMVILBICWdi4xwBaRw/viewform)