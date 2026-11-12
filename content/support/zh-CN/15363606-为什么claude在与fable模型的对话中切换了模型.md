# 为什么Claude在与Fable模型的对话中切换了模型

本文解释了为什么Claude Fable模型上的请求可能被阻止、对话切换到不同Claude模型时会发生什么、哪些阻止不会切换模型，以及如何管理自动切换。

## 为什么某些请求被阻止

Claude Fable 5.2、Fable 5.1和Fable 5是Mythos级模型，这意味着它们在软件工程、知识工作、视觉以及网络安全和生物学等领域表现出色。

发布具有这些功能的模型存在风险。如果没有强有力的保障措施，Fable在网络安全和生物学等领域的先进功能可能被用户滥用，用于发动大规模网络攻击或制造生物武器，可能造成灾难性损害。

认识到这些风险，为了让普通用户能够访问Fable的绝大多数功能，我们推出了这些模型，并配备了保障措施，可以在某些特定领域重定向或阻止其响应，符合我们的**[服务条款](https://www.anthropic.com/legal/commercial-terms)**和**[使用政策](https://www.anthropic.com/legal/aup)**。自我们首次推出Claude Fable 5以来，我们也一直在**[迭代改进](https://www.anthropic.com/news/redeploying-fable-5)**保障措施。

我们正在努力使这些保障措施更加精确，以便仅阻止真正有风险的请求，减少当今存在的误报。Fable模型上被这些保障措施阻止的大多数用户查询可能会改为从功能较弱的模型（即

## 哪些请求可能会回退

**Claude Fable 5.2、Fable 5.1和Fable 5对每个用户请求运行自动安全检查或分类器。这些检查旨在在用户提交以下请求时从Fable模型明显回退到Opus模型：**

- 攻击性网络安全技术，如构建漏洞利用、恶意软件或攻击工具。Claude Fable 5.2、Fable 5.1和Fable 5可以协助处理常规网络安全任务，但用户应该预期较高的回退率。保障措施旨在阻止访问Mythos级功能。

- 我们认为的大部分生物学双用途查询，如病毒学、毒理学、药物设计和分子设计——因此目前不建议在专业生物学研究和药物开发中使用Fable模型。（**[分类器已更新](https://www.anthropic.com/news/improving-fable-5-s-biology-safeguards)**：2026年8月6日在Claude、Claude应用和Claude平台上，Amazon Bedrock、AWS上的Claude平台、Google Cloud Vertex AI和Microsoft Foundry将随后更新。）

- 一组狭义的前沿大语言模型开发任务，如分布式训练基础设施、机器学习加速器设计和某些非标准芯片的内核开发。

这些阻止保障措施故意设置得很宽泛，我们持续改进保障措施以减少对用户体验的影响。当请求被阻止时，它们可能会回退到非Mythos模型，目前生物学、化学和生命科学请求以及前沿大语言模型开发请求回退到Opus 5，攻击性网络安全技术请求回退到Opus 4.8。

尝试提取模型推理的请求，如尝试暴露其**[总结思考](https://platform.claude.com/docs/en/build-with-claude/extended-thinking#summarized-thinking)**的请求，会被直接阻止。它们不会回退到另一个模型。

检查还会审查模型读取的所有内容，不仅仅是您的最新消息——包括记忆、来自连接器的内容、网络搜索结果和文件，因此阻止可能由您未输入的内容触发。

## 回退后会发生什么

自动模型切换在Claude应用中默认处于活跃状态，必须在API中配置。当您的请求回退时，Claude会在同一对话中的Opus模型上重新运行您被阻止的Fable请求。您会看到一条通知，说明模型已切换，响应将标记为回答的模型。

切换后，模型选择器在对话的其余部分保持在Opus上。您可以随时从模型选择器切换回Fable模型。

**注意：**如果在自动模型切换后切换回Fable模型，如果您的原始请求仍然是对话的一部分，相同的Fable保障措施可能会导致Claude再次回退。在重试前编辑您之前的消息通常会有所帮助。

## 如果回退请求也被阻止

Opus有自己的安全系统。要了解Claude Opus 5中的回退，请参阅**[为什么Claude在与Opus 5的对话中切换了模型](https://support.claude.com/en/articles/16049681)**。

如果您的请求在功能较弱的模型上也被阻止，您可以编辑您的消息并重试。对于网络安全，如果您的用例具有合法的防御目的并受到这些保障措施的影响，您可以申请Opus的网络验证计划（CVP）。了解更多关于**[网络验证计划](https://support.claude.com/en/articles/14604842)**的信息。

## 管理自动模型切换

首次选择Claude Fable模型时，自动切换默认启用。它默认保持启用，每个类别的行为**[如上所述](#h_74dc898410)**，您可以随时关闭它：

1. 转到**[设置 > 功能](https://claude.ai/settings/capabilities)**（或Claude Code中的**配置 > 模型和输出**）。

2. 关闭**消息被标记时切换模型**。

关闭自动模型切换后，被阻止的请求会暂停对话而不是切换模型。然后您可以：

- 编辑您的消息并在  同一Fable模型上重试

- 手动将相同消息发送到功能较弱的模型
​

## 使用情况和计费

被阻止的请求根据阻止发生的时间以不同方式计费：

- **Claude响应前被阻止：**为了破坏对我们保障措施的协调攻击，在任何输出之前到达的拒绝在因生物学、蒸馏或前沿大语言模型开发安全分类器而停止或回退时计费。这些是截至2026年9月我们误报率最低的类别。在其他类别中在任何输出前被阻止的请求不收费。

- **Claude开始响应后被阻止：**如果请求在中途被阻止，输入令牌和在阻止前流式传输的令牌按生成它们的模型费率计费。

- **回退请求：**如果您选择了自动模型切换，并且对话在阻止后切换到Opus，Opus响应将按各自模型的费率单独计费。我们在请求时提供信用以补偿回退请求的缓存未命中。

了解更多关于**[拒绝和回退](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#how-refusals-are-billed)**的信息。

## 提供反馈

如果您被阻止的请求似乎与上面列出的分类器无关，或者您在这些领域的合法工作不断被阻止，请告诉我们。使用

## 敬请期待更新

展望未来，我们计划考虑为双用途网络防御和生物学研究开放配额的方式。随着我们安全系统的成熟，我们的目标是支持合法的生物学和防御性网络安全工作，同时保持强有力的防止滥用保护。

我们将在项目详情（包括资格和申请方式）可用时分享更多信息。关注此帮助中心以获取更新，或**[在此注册通知](https://claude.com/form/mythos-access-interest)**。

## 自动模型切换适用的地方

自动模型切换在您可以使用Claude Fable 5.2、Fable 5.1和Fable 5的任何地方都以相同方式工作：

- 网页版Claude

- Claude移动应用

- Claude桌面版

- Claude协作

- Claude代码

- Claude设计

- Claude for Microsoft 365

- Claude标签

**重要：**如果您使用Claude API，模型切换的工作方式不同。自动切换默认不活跃，API客户必须选择加入并配置回退。在配置回退之前，模型将在API上返回带有停止原因的200响应。有关详情，请参阅**[开发者文档](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)**。

有关Fable模型的更多信息，请参阅此页面：**[Claude Fable](https://www.anthropic.com/claude/fable)**。

我们的保障措施是为了匹配模型的功能而构建的。有关保障措施如何在Opus模型上工作，请参阅**[为什么Claude在与Opus 5或Opus 5.5的对话中切换了模型](https://support.claude.com/en/articles/16049681)**。有关Sonnet 5.5上保障措施的更多信息，请参阅**[为什么Claude在与Sonnet 5.5的对话中切换了模型](https://support.claude.com/en/articles/17161993)**。