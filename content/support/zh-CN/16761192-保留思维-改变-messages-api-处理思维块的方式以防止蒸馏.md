# 保留思维：改变 Messages API 处理思维块的方式以防止蒸馏

我们正在改变 Messages API 处理思维块的方式以防止蒸馏。思维块是 Claude 在处理响应时可能产生的推理记录。在 Claude Fable 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 上，新的 API 账户在多轮对话中不再能编辑思维块周围的上下文，例如消息、工具或系统提示。在 Claude Fable 5.2 和 Claude Mythos 5.2 上，保留思维适用于所有 API 账户。

修改此前的上下文有合法的应用，我们继续通过下面列出的调整来支持。但是，这样的修改也是工业规模非法蒸馏的常见且**[公开记录的技术](https://arxiv.org/abs/2608.09867)**，这被我们的**[使用政策](https://www.anthropic.com/legal/aup)**和服务条款禁止。

API 现在验证思维块是否与产生它的相同系统提示、工具和消息一起发送回来，如果不匹配将返回错误。为了使修改的请求成功，开发者可以选择让思维块从这些请求中被*移除*；模型将在不看到思维块的情况下响应。

在本文中，我们分享了我们这样做的原因以及您可以进行的调整以最小化中断。

## 什么是思维块？

Claude 在提供最终答案之前会产生推理步骤。在 API 上，这些以加密的

## 有什么变化？

对于使用下面列出的模型的受影响账户，如果系统提示、工具或先前思维块之前的消息已被修改，API 将返回错误。

为了避免错误消息，您可以选择删除受影响的思维块（请参阅下面的文档了解标头和设置）。请求随后将通过，但模型不会看到这些块。这允许您继续对话或任务，尽管先前轮次的思维不会显示给模型。发生这种情况时，API 响应将告诉您删除了哪些块。删除块可能会显著增加输出令牌，因此在更新集成时将其用作临时解决方案，而不是长期设置。

## 我们为什么要做出这个改变？

改变对话的早期轮次是**[非法蒸馏活动中使用的常见技术](https://arxiv.org/abs/2608.09867)**，这些活动旨在提取高级模型的能力——尤其是思维——来训练另一个模型，而无需授权。蒸馏通常在工业规模上进行，使用数千个虚假账户。我们加密 Claude 的思维块以防止这种情况，但通过编辑思维块之前的对话，用户可以让 Claude 解密并打印其推理。以这种方式训练的系统可以继承它们本来不会有的能力，*而不会*继承我们为防止广泛滥用（如网络攻击和武器开发）而构建的保障措施。

这一改变旨在使蒸馏活动更难执行。它建立在现有的反蒸馏措施基础上，例如**[增强的蒸馏分类器](https://www.anthropic.com/research/next-generation-constitutional-classifiers)**和限制将会话或推理从更高级模型转移到具有较弱保障的较低能力模型的限制。

## 这对 API 集成意味着什么？

某些集成——特别是那些涉及在对话中途重写早期轮次的集成，如上下文压缩、注入的系统提醒或在会话中途更改工具——可能需要调整。

以下是帮助您完成此更新的资源：

- **[保留思维文档](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking)**涵盖了更改适用的位置、如何检查您的集成是否受影响，以及**[如何进行常见编辑](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking#replace-prefix-edits)**而不破坏保留思维。

- **[压缩](https://platform.claude.com/docs/en/build-with-claude/compaction)**（测试版）可以总结较早的轮次，同时逐字保留最近的轮次，并可以在您的代理继续工作时在后台构建摘要。它替代了客户端压缩，这可能是集成需要更改的最常见原因。

- **[对话中途工具更改](https://platform.claude.com/docs/en/build-with-claude/mid-conversation-system-messages#mid-conversation-tool-changes)**（测试版）允许您在对话期间添加或删除工具，而无需编辑早期轮次。

- 我们的**[迁移指南](https://platform.claude.com/docs/en/about-claude/models/migration-guide)**有一个完整的检查清单，用于迁移到每个模型。

检查每个页面以了解 Amazon Bedrock、Google Cloud Vertex AI 和 Microsoft Foundry 上的可用性。

保持思维块一致还有额外的好处：这意味着 API 可以更频繁地重用缓存的提示，从而降低成本和响应时间。

## 这会影响谁？

保留思维适用于这些模型和账户：

| **模型**            | **2026 年 8 月 31 日或之后创建的 API 账户（00:00 UTC）** | **在此之前创建的账户** |
| ----------------- | ------------------------------------------- | ------------- |
| Claude Mythos 5.2 | 适用                                          | 适用            |
| Claude Fable 5.2  | 适用                                          | 适用            |
| Claude Fable 5.1  | 适用                                          | 不适用           |
| Claude Opus 5.5   | 适用                                          | 不适用           |
| Claude Sonnet 5.5 | 适用                                          | 不适用           |
| Claude Haiku 5.5  | 适用                                          | 不适用           |

这涵盖了 Claude Platform 组织、Amazon Bedrock 账户、Google Cloud Vertex AI 项目和 Microsoft Foundry 项目。

我们采取了分阶段的执行方法，从新账户开始，我们在这些账户中看到蒸馏相关滥用的最高集中度。在 Claude Fable 5.2 和 Claude Mythos 5.2 上，保留思维适用于所有 API 账户。

如果您使用 Claude Code、Claude Cowork 或 Claude.ai，您无需进行任何更改；这些产品会为您处理思维块。

## 思维只能由创建它的账户读取

在 Claude Mythos 5.2、Claude Fable 5.2、Claude Sonnet 5.5 和 Claude Haiku 5.5 上，思维块只能由创建它的账户或与其关联的账户读取。

思维块可能包含私人信息。因为它们是加密的，当您检查记录时该信息不可见。此检查有助于防止共享或泄露的记录暴露其中的推理。它还使蒸馏者在我们禁止产生它们的账户后更难将收获的记录转移到新账户。

如果请求包含来自未关联账户的思维，API 会删除该思维，请求继续。它不返回错误。下一个响应可能会更慢并使用更多令牌，类似于压缩。

同一 Claude Platform 父组织、AWS 组织或 Google Cloud 组织中的账户会自动关联。

在 Claude Code 中，在会话中途切换账户具有相同的效果：下一个响应更慢并使用更多令牌。