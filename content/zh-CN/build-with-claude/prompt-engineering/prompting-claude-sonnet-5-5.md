---
title: 为 Claude Sonnet 5.5 编写提示
url: https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5
description: Claude Sonnet 5.5 特有的提示模式：effort、主动性与范围、无预先思考运行、JSON 输出、进度更新、工具使用、轮次中途消息、编码验证、工具调用、视觉输入以及拒绝。
---

本指南介绍 Claude Sonnet 5.5 特有的提示模式。有关该模型的 API 变更，请参阅 [Claude Sonnet 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5)。有关适用于所有当前 Claude 模型的技巧，请参阅[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)。

现有的 Claude Sonnet 5 提示无需修改即可表现良好，[为 Claude Sonnet 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5)中的模式仍然是一个合理的起点。对于最困难的长周期工作，Opus 模型是更好的选择。请从与您观察到的情况相符的部分开始：

* 不确定应使用哪个 effort 级别，或者轮次运行时间比在 Claude Sonnet 5 上更长或更短：[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#calibrate-effort)
* 模型在编码任务完成之前停下来与您确认，或者做了超出您要求的事情：[引导主动性与范围](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#steer-initiative-and-scope)
* 您的集成目前在关闭思考的情况下运行：[无预先思考运行](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#running-without-up-front-thinking)
* 对需要几步推算的任务，JSON 答案错误或无法解析：[带 JSON 输出的推理任务](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#reasoning-tasks-with-json-output)
* 长时间的智能体轮次看起来没有任何输出：[面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#user-facing-progress-updates)
* 模型根据其训练知识作答，而搜索本可以发现已发生变化的细节：[聊天和知识工作中的工具使用](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#tool-use-in-chat-and-knowledge-work)
* 用户在任务中途发送的消息被忽略或被视为注入的文本：[轮次中途的用户消息](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#mid-turn-user-messages-and-task-budgets)
* 代码更改在未运行测试或构建的情况下被报告为已完成：[编码任务中的验证](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#verification-on-coding-tasks)
* 模型调用工具时大小写错误，或以略有不同的名称传递参数：[容错的工具调用处理](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#tolerant-tool-call-handling)
* 关于密集图表或技术图纸的回答遗漏细节：[用于复杂视觉输入的工具](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#tools-for-complex-visual-inputs)
* 请求返回 `stop_reason: "refusal"`：[安全防护拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#safeguard-refusals)

<Note>
  有关从 Claude Sonnet 5 迁移时的五项破坏性 API 变更，请参阅 [Claude Sonnet 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#migrating-from-claude-sonnet-5)。
</Note>

## 校准 effort

[Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)（努力程度）是控制 Claude Sonnet 5.5 思考量的主要手段，并由此影响质量、"latency"（延迟）和成本。其级别已重新校准：某个级别产生的思考量与 Claude Sonnet 5 上相同级别并不相同。请针对您自己的评估重新进行一轮测试，而不是沿用您在 Claude Sonnet 5 上使用的设置。除非您的工作负载是智能体型或对延迟敏感的，否则请从 `high`（Claude API 上的默认值）开始。对于智能体编码和多步骤工具使用，明确定义的任务从 `medium` 开始，更难或更长的任务改用 `high`。对于聊天和其他对延迟敏感的工作，从 `medium` 或 `low` 开始，因为更高的 effort 意味着回复开始前需要等待更长时间。如果质量需要，再提高 effort。

较低的 effort 还会改变模型完成智能体工作的方式。在 `low` 级别下，它会保持思考简短，并且可能跳过对更改的验证。请参阅[编码任务中的验证](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#verification-on-coding-tasks)。在 `low` 和 `medium` 级别下，对于长时间的智能体任务，它更有可能在完成工作之前停下来与用户确认。请参阅[引导主动性与范围](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#steer-initiative-and-scope)。

以下三项调整会有所帮助：

* 设置 `max_tokens` 时，为思考和您预期的回复留出空间。即使思考内容不会返回给您，思考也会计入 `max_tokens`。按无思考请求设定的上限可能会截断回复。对于智能体编码，请将 `max_tokens` 设置为 128,000（模型的最大值），并对响应进行[流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)。
* 仅在您已测得质量提升的工作中使用 `xhigh` 和 `max`，因为在这些级别下思考和回复会变得长得多。在这些级别下不接受 `between_tools`，因此无法关闭预先思考。
* 要减少思考，请降低 effort 级别。在系统提示中要求模型少思考并不能可靠地减少其思考量。

在请求之间更改顶层 `effort` 值会使提示缓存失效。要在对话中途更改级别，请改用[按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) 更改，这样可以保留缓存。按消息设置 effort 目前处于 beta 阶段。一次更改会对之后的每个轮次生效，直到被另一次更改替换。例如，在 `low` 级别的交互式会话中，在用户提出难题之前添加一次改为 `high` 的更改，并在 Claude 回答之后再添加一次改回 `low` 的更改。按消息更改 effort 需要自适应思考。与 `between_tools` 一起使用时，它们会返回 400 错误，如[无预先思考运行](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#running-without-up-front-thinking)中所述。

## 引导主动性与范围

Claude Sonnet 5.5 自主推进的程度取决于 effort 级别和请求本身。在较低的 effort 下，它有时会在编码任务完成之前与您确认。在较高的 effort 下，或面对开放式请求时，它可能会做超出您要求的事情。请通过 effort 级别和系统提示中的指令来引导它。

**将工作进行到底。** 在 `low` 和 `medium` effort 下的智能体编码任务中，模型有时会在工作完成之前与您确认。它可能会停下来确认计划、提出一个它本可以自己回答的问题，或者在完成多部分任务中的一部分后停下来询问是否继续。请先尝试更高的 effort 级别。若要在不更改 effort 的情况下让模型持续工作，请将以下内容添加到您的系统提示中：

```text wrap
Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step.

When the work the user asked for is done and checked, stop and report. Don't add features, tests, files, docs, or refactors that weren't asked for. If you think one would help, mention it at the end instead of doing it.
```

使用此提示后，模型在 `low` 和 `medium` effort 下会完成更多工作，因此这些级别的会话运行时间更长、成本更高。该提示不能替代您自己关于高风险或不可逆操作的规则。请将这些规则保留在您的系统提示中。

**编码时未经请求的添加。** 模型倾向于添加符合您代码仓库约定的测试、文档和小型辅助文件，即使您并未要求。它在每个 effort 级别下都会这样做，在更高的 effort 下更甚。所请求的更改本身会与要求保持接近。大多数团队会欢迎这一点。如果您希望更改仅限于明确请求的内容，请只添加该提示的第二段，即以"When the work the user asked for is done"开头的那一段。在 `xhigh` 和 `max` effort 下，该段会减少这些添加，并使更改整体上更小。

**`xhigh` 和 `max` effort 下的彻底性。** 在这些级别下，模型尤其彻底。完成任务后，它可能会自行开始多轮审查和验证，如果您的框架提供子智能体，有时还会使用子智能体。它还可能顺带修复过程中注意到的相关问题。这会花费更多时间和令牌，因此请在 `high` 或更低级别下运行常规工作，在这些级别下这种情况很少发生。如果您确实希望获得这些 effort 级别带来的额外彻底性，但希望将其引导到任务本身，请将以下内容添加到您的系统提示中：

```text wrap
When the work the user asked for is done and its checks pass, stop and report. Don't start extra rounds of review or hardening on your own, and don't launch reviewer subagents unless the user asked for a review. If you think a deeper review is worth doing, say so at the end.
```

在 `max` effort 下的编码任务测试中，这阻止了模型启动审查子代理，并将会话成本降低了约三分之一，而质量没有变化。它会减少主代理自行发起审查轮次的频率，但不会完全消除。

**开放式请求。** 当请求是开放式的，例如"向我展示你能用这个做什么"时，模型可能会开始构建演示文稿、报告或视频，而您只是想要一些想法。如果您希望先获得想法或计划，请在请求中说明，或将以下内容添加到您的系统提示中：

```text wrap
When the user asks for ideas, options, or a plan, give them that and stop. Don't start building or changing anything until they say to go ahead.
```

## 无预先思考运行

要在无预先思考的情况下运行 Claude Sonnet 5.5，请发送 `thinking: {"type": "between_tools"}`。这是该模型上最低的思考设置，在 `high` effort 或更低级别下可用。如果您的集成目前在关闭思考的情况下运行，请将其切换为 `between_tools` 并检查以下几点：

* **在 `high` effort 或更低级别下发送 `between_tools`。** 在 `xhigh` 或 `max` effort 下，带有 `between_tools` 的请求会返回 400 错误。使用 `between_tools` 时，effort 也无法在对话中途更改：与当前生效级别不同的按消息 `output_config.effort` 会返回 400 错误。要为每个轮次设置不同的 effort，请使用自适应思考。使用 `between_tools` 时，请删除任何告诉模型不要思考的指令。此类指令会使模型更有可能在其可见输出中写出内部 XML 标签。
* **按块类型读取响应。** 使用自适应思考时，响应可能以 `thinking` 块开头，在默认的 `display: "omitted"` 下，其 `thinking` 字段为空。使用 `between_tools` 时，响应可能以进度更新 `thinking` 块开头。不要假设第一个内容块是文本。
* **原样传回 `thinking` 块。** 使用 `between_tools` 时，模型在工具调用之间写下的说明如果超过一两句话，仍会以 `thinking` 块的形式返回。每个块都带有该说明的摘要。请将它们与助手轮次的其余部分一起原样传回。您传回的块会向模型提供它所写的完整说明，而不是摘要。
* **对于不使用工具的推理任务，请使用自适应思考。** 在不含工具的请求中，`between_tools` 意味着模型不先思考就直接作答。对于需要几步推算的任务，请改用自适应思考。请参阅[带 JSON 输出的推理任务](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#reasoning-tasks-with-json-output)。

## 带 JSON 输出的推理任务

本节适用于您要求 Claude Sonnet 5.5 对需要几步推算的任务给出 JSON 答案的情况。示例包括汇总文档中的数字、应用某条规则或对项目进行排序。在此类任务中，模型经常不经思考直接作答，尤其是在 `low` 和 `medium` effort 下。有效的做法取决于您请求 JSON 的方式。在可用的情况下，请使用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。这样响应文本就是符合您 schema 的 JSON，无需解析。

使用结构化输出时，响应文本只包含 JSON，因此模型只能在思考中推算问题。当它跳过思考时，在这些任务上的准确性可能会降低。以下更改有助于保持高准确性。

**要求模型先思考。** 使用自适应思考时，将以下这行添加到系统提示的末尾：

```text wrap
Think the problem through before you answer.
```

加上这行后，模型会更频繁地在作答前思考。在 `high` effort 下，这行能使准确性接近模型在 `xhigh` 下达到的水平，而输出令牌仅适度增加。在 `low` 和 `medium` effort 下，它能提高准确性，但达不到模型在 `high` 下的水平，且输出令牌的增加幅度更大。

**或者使用 `xhigh` effort。** 使用自适应思考时，即使不加这行，`xhigh` 也能在这些任务上提供最高的准确性。它比 `high` 使用更多的输出令牌。

**使用自适应思考而非 `between_tools`。** 在不含工具的请求中，模型在 `between_tools` 下不会在作答前思考。这行在那里不起作用，这些任务的准确性也更低。请对这些请求使用自适应思考，并配合本节中的步骤。在测试中，将请求拆分为两个（一个请求获取答案，一个请求获取 JSON）可以获得很高的答案准确性和 JSON 合规性，但成本和延迟非常高。

在 `low` 和 `medium` effort 下使用结构化输出时，模型偶尔会一直思考直到达到 `max_tokens`。在 `high` effort 及以上，这种情况几乎不会发生。请将任何 `stop_reason` 为 `"max_tokens"` 的响应视为失败，即使其文本包含有效的 JSON，并进行重试。按照[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#calibrate-effort) 中的说明，将 `max_tokens` 设置得足够高以容纳思考和 JSON，但不要高于您愿意为单次尝试花费的量。

如果您无法使用结构化输出，请改为在提示中要求 JSON。此时模型通常会在响应文本中推算问题，并在末尾写出 JSON。JSON 通常包含正确答案，但期望整个响应都是 JSON 的解析器会失败。以下两点会有所帮助：

* **解析响应中的最后一个 JSON 值。** 只读取 `text` 块，并将 `stop_reason` 为 `"max_tokens"` 的响应视为失败。从每个 `{` 或 `[` 开始，尝试解析一个 JSON 值。当某个值解析成功时，从该值的末尾继续，这样嵌套在其中的值就不会被单独计算。保留找到的最后一个值。不要截取从第一个 `{` 到最后一个 `}` 之间的全部内容。模型偶尔会在最终 JSON 之前写一个草稿，而该范围会同时包含两者。如果您的答案是连续的多个 JSON 值，例如每行一条记录，请保留最后一段仅由空格、逗号或换行符分隔的值。检查结果是否包含您期望的字段，如果不包含，则重试一次。在测试中，这使几乎每个响应都可用，且不改变其准确性。
* **也可以考虑使用自适应思考配合 `xhigh` effort。** 此时模型会在思考中推算问题，并且几乎总是只返回 JSON。总输出令牌与 `high` 下大致相同，因为推算过程从响应文本转移到了思考中。

## 面向用户的进度更新

在工具调用之间，Claude Sonnet 5.5 会写下面向用户的说明，介绍它刚刚发现了什么以及接下来要做什么。超过一两句话的说明会以[进度更新 `thinking` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)的形式返回。较短的评论仍为 `text`。在默认的 `thinking.display` 下，进度更新块的文本为空，因此只渲染 `text` 块的客户端在长时间的智能体轮次中可能看起来没有任何输出。这在聊天界面以及用户实时跟踪模型工作的其他产品中最为重要。

要显示这些说明，请设置 `display: "updates"`（beta，`thinking-display-updates-2026-08-18` 标头）。使用 `between_tools` 时，这些说明会连同其摘要文本一起返回，因此不需要 `display` 字段。`between_tools` 不接受任何其他字段：与其一起发送的 `display`、`budget_tokens` 或 `block_binding` 会返回 400 错误。迁移指南中关于[工具调用之间的文本](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#text-between-tool-calls)的部分展示了如何渲染这些说明。有时模型需要在长轮次中途向用户展示确切的文本，例如代码片段或它需要得到回答的问题。针对这种情况，请为它提供一个用于向用户发送消息的简单工具。告诉模型仅将该工具用于此类内容。请在会话的第一个请求中声明该工具，这样 `tools` 列表之后就不会发生变化。

接下来，删除诸如"将所有发现保留到最终响应中"之类的旧指令。如果您之后希望在可预测的节点获得更新，例如在第一次工具调用之前用一行说明模型将要做什么，并在结束时给出简短回顾，请在系统提示中说明。模型会遵循此类指令。在固定节点进行更新在人机协作（human-in-the-loop）工作中最有帮助。

如果长时间的工具调用轮次仍然比您期望的沉默得更久，您的框架可以提示模型进行更新。让框架统计连续未向用户发送任何文本或进度更新的工具调用步骤。在连续若干步（例如五步）之后，在最新的工具结果之后附加一条单轮提醒。将其作为[轮次范围的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#turn-scoped-system-messages)（beta）发送，文本类似如下：

```text wrap
The user hasn't heard from you in a while: say in a few words what you're doing, then continue.
```

如果该轮次仍然保持沉默，请在第二次或第三次提醒后停止发送提醒。在工具结果之后频繁出现框架文本可能导致模型将该文本视为可能的提示注入，如[轮次中途的用户消息](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#mid-turn-user-messages-and-task-budgets)中所述。在之后的请求中，请将每条提醒保留在 `messages` 中。由于提醒是追加的，而不是插入后再删除，提示缓存和[保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)都能保持完整。在 `high` effort 下，当有用于向用户发送消息的工具可用时，提醒会使模型更频繁地向用户更新，并缩短其最长的沉默时段，而任务质量没有可测量的变化。

## 聊天和知识工作中的工具使用

在聊天和知识工作任务中，Claude Sonnet 5.5 有时会根据其训练知识作答，而网络搜索本可以发现已发生变化的细节。示例包括哪些内容是允许的、必需的或收费的。

首先，检查您的提示中是否有不鼓励工具使用的措辞，例如"仅在绝对必要时使用工具"或"尽量减少工具调用"，并将其删除。然后，如果您的产品为模型提供了搜索工具，请将以下内容添加到您的系统提示中：

```text wrap
Use the search tool to check specifics that may have changed since your training, such as what is allowed, required, or charged, even when you feel confident. For researched work such as a report or a comparison, gather current sources rather than writing from your training knowledge.
```

这对研究和支持类产品最为重要，因为这些产品的答案依赖于最新的细节。

## 轮次中途的用户消息

Claude Sonnet 5.5 经过训练，能够抵御间接提示注入，即通过工具结果以及它在任务期间读取的其他内容传入的恶意指令。有时它会将真实的用户消息视为可能的注入。假设用户在任务中途输入的消息以[对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)的形式紧跟在工具结果之后到达模型，或者位于 `tool_result` 块内部。此时模型可能会告诉用户，工具结果中包含冒充用户消息的文本，并忽略该消息或要求用户确认。

您的框架在每个工具结果之后添加的令牌倒计时可能会导致这种情况。允许用户在模型执行多步骤轮次的中途发送消息，或让您的框架在每一步的工具结果之后添加指令或上下文，也会如此。在每种情况下，文本都紧跟在工具结果之后到达。使用倒计时或每步指令时，这可能在每次工具调用时发生。偶尔的单轮提醒，例如[面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#user-facing-progress-updates)中的那种，出现的频率要低得多。如果您发现模型对您自己的提醒出现了这种反应，请降低发送提醒的频率。为避免误读：

* 切勿将用户文本放在 `tool_result` 块内。模型最常误读这种放置方式。
* 将轮次中途的用户输入作为用户轮次传递。将用户的话作为文本块追加到携带 `tool_result` 块的用户消息中，放在最后一个 `tool_result` 之后。
* 将运行框架通知（例如提醒）放在用户的话之后的单独的对话中途系统消息中。切勿将通知和用户的话放在同一个块中。
* 在用户可以在轮次中途输入的交互式会话中，不要在工具结果之后添加您自己的令牌或预算倒计时。[任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)（beta）会添加类似的倒计时，但尚未发现它们会导致这种误读。如果在设置了任务预算的情况下出现误读，请尝试在不设置任务预算的情况下运行会话。

## 编码任务中的验证

在智能体编码任务中，Claude Sonnet 5.5 通常会在报告更改完成之前检查其工作。不过，在 `low` effort 下，它有时会在未运行能够实际检验该更改的检查的情况下就报告更改已完成。例如，它可能因为项目的依赖项未安装而跳过项目的测试。

如果您发现更改被报告为已完成，但记录中没有测试或构建输出，请将以下段落或类似段落添加到系统提示中。在 `low` effort 下，它能使跳过检查或敷衍检查的情况变得罕见，任务质量没有可测量的变化，每个任务的成本仅略有增加：

```text wrap
When you change code that can be run, built, or type-checked, run a real check that exercises the change before reporting it done: the project's tests, type-checker, or build, or the changed command itself. A syntax-only check, or a check command that failed to start, does not count; if all that is missing is the project's declared dependencies, install them with its own package manager and lockfile (for example, npm install or pip install -r requirements.txt), never through sudo or the system package manager, unless told not to. Only if no real check can run here, say which one you did not run and why instead of reporting the change as done.
```

## 容错的工具调用处理

Claude Sonnet 5.5 偶尔会以仅大小写不同的名称调用已声明的工具，例如用 `bash` 调用 `Bash`。它也可能以略有不同的名称传递已知参数。与其将此类调用视为致命错误，不如让您的运行框架以以下两种方式之一进行处理：

* 当匹配明确无歧义时接受该调用，即使大小写有误。
* 返回一个带有 `is_error: true` 的 `tool_result`，并说明确切的预期名称。模型通常会在下一轮中更正调用。请参阅[使用 `is_error` 处理错误](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls#handling-errors-with-is-error)。

## 用于复杂视觉输入的工具

对于密集的图表和技术图纸，请为 Claude Sonnet 5.5 提供裁剪、缩放图像或对图像运行代码的方法。有了这些工具，模型读取这些输入的准确性会显著提高。对于图表，这些工具在每个 effort 级别都有帮助。对于技术图纸，它们仅在 `high` effort 及以上才有帮助，在 `xhigh` 和 `max` 下帮助最大。对于图表，添加工具比提高 effort 更有帮助：在测试中，在 `high` effort 下使用工具时，模型读取图表的准确性高于在 `max` effort 下不使用工具时，而成本仅为其一小部分。[裁剪工具示例](https://platform.claude.com/cookbook/multimodal-crop-tool)提供了一个可用的工具定义。

## 安全防护拒绝

Claude Sonnet 5.5 运行的安全分类器可能会拒绝请求。拒绝以正常响应的形式返回，带有 `stop_reason: "refusal"`，并由 `stop_details.category` 指明[拒绝类别](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)：

* `cyber`：该请求可能助长网络危害，例如恶意软件或漏洞利用开发。允许在源代码中查找漏洞。不允许高风险的两用网络安全工作。
* `bio`：该请求可能助长生物危害，例如危险的实验室方法。日常健康和教育类问题不受影响。
* `frontier_llm`：该请求可能有助于开发竞争性的 AI 模型。
* `reasoning_extraction`：该请求要求模型在响应文本中复现其内部推理。
* `general_harms`：该请求属于其他使用政策领域。良性工作也可能触发此类别。

如果 `cyber` 分类器阻止了您组织的合法安全工作，您可以申请加入 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)。如果 `bio` 分类器阻止了您组织的生命科学工作，您可以申请加入[生命科学验证计划](https://www.anthropic.com/news/life-sciences-verification-program)。

如果您开启了[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)（beta），它会在 Claude Sonnet 5 上重试 `cyber` 和 `frontier_llm` 拒绝。它不会重试 `bio`、`reasoning_extraction` 或 `general_harms` 拒绝。请参阅[拒绝、回退和计费](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#refusals-fallback-and-billing)。

如果您的提示要求模型在响应中包含其推理，请删除这些指令，因为它们会招致 `reasoning_extraction` 拒绝。使用自适应思考时，请改为从[摘要思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#summarized-thinking)块中读取推理（`display: "summarized"`）。您仍然可以要求对答案进行简短解释或对所采取的操作进行总结；请参阅[将推理保留在思考块中](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#keep-reasoning-in-thinking-blocks)。
