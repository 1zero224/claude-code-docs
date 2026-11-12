---
title: 为 Claude Fable 5.2 编写提示
url: https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2
description: Claude Fable 5.2 和 Claude Mythos 5.2 特有的提示模式：effort、过早停止、审批、代码归属、进度更新、子代理、不断累积的谨慎、为早期模型设计的提示和 harness、最终报告以及拒绝。
---

本指南介绍 Claude Fable 5.2 特有的提示模式。有关该模型的能力和 API 变更，请参阅 [Claude Fable 5.2 的新功能](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2)和 [Claude Fable 5.2 和 Claude Mythos 5.2 迁移指南](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide)。有关适用于所有当前 Claude 模型的技巧，请参阅[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)。

Claude Fable 5.2 与早期模型相比有几处值得了解的行为差异。请先查看与您观察到的情况相符的部分：

* 不确定应使用哪个 effort 级别，或简单请求启动缓慢：[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#calibrate-effort)
* 一个轮次在工作完成之前就结束了：[完成整个任务](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#finish-the-whole-task)
* 编码代理复制现有代码，或者报告问题却置之不理：[说明代码归谁所有](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#say-who-owns-the-code)
* 长时间的代理轮次看起来没有任何输出：[在长轮次中让用户了解进展](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#keep-users-informed-during-long-turns)
* 代理会话启动的子代理多于任务所需：[为子代理设定预期](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#set-expectations-for-subagents)
* 代理请求不需要的审批，或搁置常规工作：[防止谨慎不断累积](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#keep-caution-from-piling-up)
* 您的系统提示、技能或 harness 是为 Claude Fable 5.1 或 Claude Opus 5.5 编写的：[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)
* 您的用户会在长任务结束后阅读最终报告或交接内容：[让报告易于理解](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#make-reports-easy-to-follow)
* 请求返回 `stop_reason: "refusal"`：[安全防护拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#safeguard-refusals)

<Note>
  有关迁移时应用本指南的顺序，请参阅[从哪里开始](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#where-to-start)。

  本页的几个部分建议向您的系统提示中添加文本、从系统提示或代理携带的笔记和规则中删除某些行，或者修改工具描述。请仅在新对话中进行这些更改。在 Claude Fable 5.2 上，API 会对所有账户强制执行[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)。如果在 `system`、`tools` 或较早的消息发生变化后仍回传思考块，请求可能会返回 400 错误，因此使用新文本恢复已存储的对话可能会失败。要在对话中途更改指令，请[将其放在最新的轮次中](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#changing-context)。在支持[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)的平台上，您可以改为[向对话追加一条系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#new-instructions)。
</Note>

## 校准 effort

在 Claude Fable 5.2 上，[effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)（投入程度）级别决定模型思考的多少、进行的工具调用次数、一个轮次耗费的时间以及使用的令牌数量。请将 effort 级别与提示一同调整。

从 `high` 开始，这是 Claude Fable 5.2 在 Claude API 上的默认值。根据用例，您可以上调或下调 effort，并用您自己的评估进行测试。各 Claude 模型的 effort 级别排序相同，但每个级别的思考量可能不同。因此，请重新进行 effort 扫描测试，而不要沿用您在 Claude Fable 5.1 或 Claude Opus 5.5 上调好的设置。

* `low` 适合快速的来回交互、定义明确的任务以及高吞吐量工作负载（例如智能且快速的子代理）。在编码评估中，Claude Fable 5.2 在 `low` effort 下的表现优于以往的模型。在 Anthropic 的测试中，它的得分接近 Claude Fable 5.1 在更高 effort 下的得分。
* `medium` 适合交互式编码、聊天和日常知识工作。
* `high` 适合困难的编码、研究、代理工作以及长时间无人值守的运行。轮次完成所需的时间明显长于 `medium`。
* `xhigh` 用于您最困难的长时间编码和代理工作，前提是您已测得收益。在 Anthropic 的内部编码评估中，它略微提高了得分，但使用的令牌多得多。
* `max` 用于测量模型的上限，或用于时间和令牌都无关紧要的少数任务。对于开放式问题（例如科学发现和数学），使用 `max` 最为合理。轮次很慢，使用的令牌也比 `xhigh` 多得多。

使用 `high` effort 时，您就能获得大部分能力提升。从 `medium` 到 `high` 的提升提高了 Anthropic 跟踪的大多数编码和知识工作评估的得分。这一提升也会使用更多令牌和时间，因此凡是您的评估结果保持不变的地方，请降到 `medium`。在 `high` 之上，收益参差不齐，并取决于任务。

对于在 `low` 或 `medium` 下进行的代理工作，请添加[完成整个任务](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#finish-the-whole-task)中的文本。在较低的 effort 级别下，模型更可能过早停止。

在 `high` 及以上级别，模型几乎在每次回复前都会思考，包括回复问候语。如果您发现基本请求耗时较长或成本较高，请对这些轮次使用较低的 effort。在 Anthropic 的测试中，在 `medium` 及以上级别，要求直接回答而不进行斟酌的指令并不能阻止模型思考。

effort 对助手轮次回复的长度影响很小。要改变回复的长度和语气，请使用提示。

您也可以在一个工作流中混合使用不同级别。例如，您可以在 `medium` 下与模型一起规划和迭代，然后为长时间无人值守的部分提高 effort。要在对话中更改级别并保留提示缓存，请使用[按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)（beta，`mid-conversation-output-config-2026-07-01` 标头）。该页面列出了支持此功能的平台。一次更改会一直有效，直到另一次更改取代它。在不支持按消息设置 effort 的地方，请在下一个请求中设置新的顶层 `effort` 值，这会使缓存重新开始。

## 完成整个任务

在长时间、多步骤的任务中，Claude Fable 5.2 有时会在工作完成之前结束一个轮次。它可能会以列出剩余待办事项、提议继续，或者在超时等普通障碍处停止的方式结束轮次。这种情况在 `low` 和 `medium` effort 下更常见。

如果您在代理中看到这类停止，请向系统提示中添加如下文本：

```text wrap
# Finishing work
Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed.

While status notes are welcome, as is clarifying with the user on open decisions, carry on with work that does not depend on the user's answer without unnecessary stops.

If you notice yourself inviting the user to redirect you or offering to wait, instead proceed on the next part of the task. You may meet ordinary obstacles like errors, timeouts, locked files, empty results or failing tools. Diagnose them first, and when they are not real blockers, work through them with the access you have (wait and retry, fix the request, use another tool or source) rather than stopping unnecessarily.

If someone clearly decided something is a hard blocker, such as a file marked not to be touched, access that was intentionally withheld, or a safety guardrail, leave it alone, say plainly what you found, and look for another safe way to finish the task. Stopping before the task is complete is very rarely merited, as in when the task can't move forward without user input, or where the blocker is deliberate and should not be worked past. This does not override the need for confirmation on risky or destructive actions.
```

这段提示告诉 Claude，对于普通障碍，例如错误、超时和失败的工具，应当诊断并解决。对于有意设置的阻碍，例如标记为不得修改的文件，则应不予触碰并进行报告。请将其作为起点，并根据您的用例进一步调整这段提示。

在测试中，Anthropic 在 `medium` 下重放了一款代理编码产品中录制的会话。使用这段文本的一个版本后，以本可继续完成的未完成工作而结束的会话数量减少了大约一半。它消除了大部分请求许可或提议继续的结尾，但没有消除列出剩余事项后就停止的结尾。输出令牌增加了约 30%。

由于使用这段文本后模型提问的频率降低，请为有风险或不可逆的操作保留您自己的确认步骤。

### 审批

当您让模型继续时，请准确说明您想要什么，而不是只发送一个简单的"继续"。例如，在您可能会发送"go"或点赞表情的地方，回复"是的，在预发布数据库上运行迁移"。

如果您还在系统提示中写入了审批规则，请具体说明您希望复核的操作。在 Anthropic 针对无人审批的模拟任务进行的测试中，一条将"向本对话之外的人发送消息"列为需要审批的步骤之一的规则，导致模型搁置了大约一半本应发给同事和其他团队的常规请求。

## 说明代码归谁所有

Claude Fable 5.2 会密切关注其所处理的代码归谁所有。在 Anthropic 针对小型编码任务的测试中，当提示未提及归属，或将代码库描述为他人所有时，模型倾向于在现有结构上添加内容，即使这会重复现有代码。当提示说明代码库归模型自己所有时，它通常会合并重复代码，有时还会在未被要求的情况下进行相关修复。

如果您希望模型在工作时改进结构，请说明代码归它自己所有：

```text wrap
You are the owner and long-term maintainer of this codebase, and its design is up to you.
```

这句归属声明会使模型将重复代码合并为一个共享函数的可能性大幅提高。在 Anthropic 针对小型编码任务的测试中，不使用这句话时，模型在不到 10% 的运行中合并了重复代码；使用时，这一比例约为 75%。

使用这段提示时，输出令牌增加了约 70% 到 80%，并且模型做了一些无人要求的更改，其中许多是新的测试和文档。在某些运行中，它还更改了现有功能的行为。Claude Fable 5.2 的结尾 PR 总结常常遗漏小的更改，因此请同时审查所报告的 diff 和总结。

如果代码归用户所有，且用户希望尽可能少地改动代码，请不要使用这句归属声明。限制范围的规则并不能抵消它。在 Anthropic 的测试中，即使同一提示前面出现了"No drive-by refactors, renames or cleanups"（不要顺手重构、重命名或清理），模型仍在超过一半的运行中合并了重复代码。

## 在长轮次中让用户了解进展

在长时间连续调用工具的过程中，Claude Fable 5.2 有时很少写出面向用户的文本。此外，模型也可能在进行第一次工具调用时不说明它将要做什么。

首先，请检查您的客户端是否能收到模型的更新。在 Claude Fable 5.2 上，模型在工具调用之间写下的更新会以[进度更新 `thinking` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)的形式返回。这些更新说明模型发现了什么以及接下来要做什么。在 `thinking.display` 的默认值 `"omitted"` 下，其文本为空。因此，只渲染 `text` 块的客户端在长时间的代理轮次中可能看起来毫无输出。设置 `display: "updates"`（beta，`thinking-display-updates-2026-08-18` 标头）可接收每条更新的简短摘要，设置 `"summarized"` 则可同时接收更新和推理摘要。然后在工具调用之间渲染非空的 `thinking` 块。

频繁的更新在交互式场景中最为重要。如果这是您的用例，请尝试以下做法：

### 修改您的 harness

鼓励模型沟通的一个有效做法是统计连续多少个工具调用步骤没有给用户任何可读内容：既没有 `text` 块，也没有带文本的 `thinking` 块。统计时请将 `display` 设置为 `"updates"` 或 `"summarized"`，因为在 `"omitted"` 下每个 `thinking` 块都是空的。这一做法适用于支持[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)的平台。当连续出现若干个这样的步骤时，您可以在最新的工具结果之后，以[轮次范围的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#turn-scoped-system-messages)（`clear_at: "next_user_message"`，beta，`mid-conversation-system-clear-at-2026-08-21` 标头）的形式追加以下提醒：

```text wrap
The user hasn't heard from you in a while: say in a few words what you're doing, then continue.
```

在 Anthropic 针对多轮编码任务的测试中，该提醒将最长的无输出时段大致缩短了一半，且令牌和时间没有明显增加。

在后续请求中，请将每条提醒保留在 `messages` 中。删除或改写其中一条会编辑较早的轮次，从而使其之后的思考块失效。请参阅[将每轮提醒作为轮次作用域系统消息发送](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#per-turn-reminders)。

### 表明意图

如果希望在第一次工具调用之前得到一句开场说明、简短的更新以及结尾的回顾，您还可以向系统提示中添加以下文本：

```text wrap
Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next), so a reader who only sees the last message has the full picture.
```

使用这段文本时，在编码任务的每次运行中，模型都会在第一次工具调用之前写一条消息。请注意，这也使每个任务的输出令牌增加了约 20%。

请在有人直接参与其中的场景下使用这段文本。要增加交互式更新，请将这段文本与前面的提醒一起使用。

### 更新系统提示指令

如果您无法在 harness 中添加该提醒，或者您的平台不支持对话中途系统消息，请改为向系统提示中添加这句话：

```text wrap
Every few tool calls, give the user a one-line update on where things stand.
```

在 Anthropic 的测试中，这句话消除了工作流自动化任务中大部分长时间的无输出时段。即使同时使用提醒以及开场和回顾文本，更新之间仍可能出现连续多次的工具调用。Claude Fable 5.2 在长时间运行的异步任务上表现良好，因此 Anthropic 推荐将该模型用于无人需要频繁更新的工作。

## 为子代理设定预期

Claude Fable 5.2 的委派程度取决于您的 harness 和任务。在 Anthropic 针对短任务、使用一个最简子代理工具进行的测试中，它很少启动子代理。在一款提示中鼓励委派的代理编码产品中，它则很乐于委派。

在较高的 effort 级别（例如 `xhigh` 和 `max`）下，Anthropic 观察到 Claude Fable 5.2 在一款代理编码产品中使用子代理来审查自己的工作，而 Claude Fable 5.1 在相同任务上不会这样做。

早期测试者将 Claude Fable 5.2 用作其他模型的规划者和编排者，取得了良好效果。对于适合并行运行的工作，委派可以加快速度。不过，子代理可能占据会话成本的很大一部分，因此请检查您的子代理默认使用哪个模型。

只有当 Claude Fable 5.2 启动的子代理多于任务所需时，才修改子代理工具的描述。当 harness 已经鼓励委派，且子代理占您令牌用量的很大一部分时，新的描述最为有用。如果您的子代理工具描述很简单，且您的提示中没有其他内容要求委派，您可能不需要新的描述。如果您的系统提示、技能或代理文件要求模型进行委派，请首先审查这些内容。

在新的工具描述中，说明子代理会给会话增加多少成本，说明何时值得委派、何时不值得，并告诉模型如何向它启动的子代理交代任务。

如果模型委派过多，请更多地说明子代理在您的 harness 中的成本，并测试这一更改。在 Anthropic 于 `high` effort 下进行的测试中，一个侧重于此的较长描述与较短版本相比，使子代理数量大致减少了一半。

请根据您的需求调整描述，并调整其中告诉模型何时进行委派的内容。

## 防止谨慎不断累积

谨慎性的指令可能会通过记忆、笔记和指令文件从过去的会话中不断累积。当 Claude Fable 5.2 严格遵循这些指令并请求不必要的审批时，这可能会适得其反。在早期测试中，Anthropic 主要在多个代理协同工作时看到这种情况。

如果您看到这种情况，请从代理的上下文中删除过时的指令：

* **删除不再需要的笔记。** 记忆中可能包含诸如"永远不要重试"之类的旧指令，模型会一直遵循，因此一旦这些指令不再相关，请将其删除。
* **列出不需要审批的步骤。** 告诉模型更自由地行动过于笼统，因此请列出它可以自行执行的常规步骤。
* **检查您的验证步骤。** 诸如"在第一个错误处停止并审查"之类的条件可能会阻碍自主工作，因此请删除或收窄您不需要的停止条件。
* **明确说明例外。** 模型会将包含"never"的指令视为绝对的。例如，如果您希望模型对某个操作保留责任，请写"删除文件前先请求审批"，而不是"永远不要删除文件"。请参阅[审批](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#approvals)。

请为有风险或不可逆的操作保留您自己的确认步骤。

## 审查为早期模型设计的提示和 harness

迁移到 Claude Fable 5.2 时，请审查您的提示和 harness。在 Anthropic 的测试中，一些为早期模型设计的指令和干预措施提高了成本、造成了副作用，或者不再起作用。以下是需要重新考虑的做法：

* **长期有效的委派指令：** 要求模型进行委派的内容可能存在于系统提示、技能和代理文件中。迁移前请阅读这些文件，如果模型在测试中委派过多，请删除不再需要的内容。请参阅[为子代理设定预期](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#set-expectations-for-subagents)。
* **向模型展示的令牌预算：** 有些 harness 会向模型展示任务的令牌预算以及剩余多少。这个预算的大小会影响 Claude Fable 5.2 的花费。在 Anthropic 针对编码任务、在 `high` 和 `xhigh` effort 下进行的测试中，去掉一个较大的预算降低了成本，且性能相当。请在不带预算的情况下测试您的任务。这一发现针对的是提示中的文本，而不是 API 的[任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)，后者是一项 beta 功能，不在该测试范围内。
* **编辑较早轮次的 harness 代码：** Claude Fable 5.2 使用[保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)。如果您的 harness 在请求之间更改系统提示、工具或较早的消息，那么历史记录中更改点之后的思考块就会失效。默认情况下，Messages API 此时会拒绝该请求，而 Message Batches API 会丢弃这些块。要在不出现此错误的情况下缩短历史记录，请使用[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)，或者在摘要之后不保留任何较早轮次的[压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)。要让 Messages API 也丢弃这些块，请将 `thinking.block_binding.prefix_mismatch_behavior` 设置为 `"drop_block"`（beta，`thinking-binding-controls-2026-08-01` 标头）。在 Anthropic 的测试中，当每个轮次都丢弃较早的思考时，模型会重新思考并使用更多输出令牌。这是最坏的情况，增加的幅度取决于您的 harness 编辑较早轮次的频率。除压缩和上下文编辑之外，请保持历史记录只追加不修改。这样可以保持较高的缓存命中率，并且不会丢弃任何思考。在依赖 `"drop_block"` 之前，请在您自己的任务上测量输出令牌。请参阅[API 如何处理无效块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#mismatch-behavior)和[在不编辑前缀的情况下进行更改](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#replace-prefix-edits)。
* **"将该回答视为已完成"类语句：** [为 Claude Opus 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#thinking-instructions-in-chat-system-prompts)为聊天系统提示提供了两句话。它们告诉模型将较早的回答视为已完成，并将思考集中在新消息上。在 Anthropic 于 `high` effort 下进行的测试中，这些指令减少了 Claude Fable 5.2 在后续问题上的思考。模型指出自己早先回答中错误的可能性也大大降低。如果您希望模型持续重新审视其早先的回答，请不要使用这些语句。在其他场景中，采用这些语句之前，请测试模型是否仍会指出自己的错误。
* **要求少思考的指令：** Claude Fable 5.1 通常会遵循诸如"直接回答，不要斟酌"之类的语句。在 Claude Fable 5.2 上，在 `medium` 及以上级别，添加这类语句并不能阻止模型思考。请删除这类语句并降低 effort 级别，如[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#calibrate-effort)中所述。
* **写作风格规则：** Claude Fable 5.2 会严格遵循这些规则，因此诸如"每句一条指令，不超过 20 个词"之类的硬性限制可能会使其文字显得生硬断续。在添加[让报告易于理解](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#make-reports-easy-to-follow)中的文本之前，请删除不再需要的规则。
* **不断累积的规则、笔记和记忆：** 记忆文件、笔记和指令文件会随着时间积累各种谨慎提示，其中一些可能是模型自己写的。迁移前，请删除理由已不再成立的谨慎提示，并改写绝对化的规则，写明其适用范围和理由。如果某个系统已经陷入停滞，请考虑在精简其记忆和笔记后重新启动。请参阅[防止谨慎不断累积](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#keep-caution-from-piling-up)。

## 让报告易于理解

早期测试者认为 Claude Fable 5.2 的写作优于 Claude Fable 5.1，清晰度与 Claude Opus 5.5 相近。不过，在长时间的代理工作之后，阅读 Claude Fable 5.2 最终报告的人通常没有看到其背后的工具调用或文件，因此报告需要能够独立成文、自明其意。

如果您要求 Claude Fable 5.2 撰写最终报告或交接内容，请向系统提示中添加以下文本，并根据您的读者和产品进行调整：

```text wrap
# Communicating clearly
Write for an intelligent but low-context reader. The person reading has not seen your tool calls, your reasoning or the files you opened, and may be switching in from other work. Messages should generally be self-contained: the reader should understand what you mean without having to dig for other context. Put the key points in the first few sentences. Practice progressive disclosure in your sentence order, and do not use undefined references and ideas in line 1 that won't make sense until the reader reaches line 12.

Use plain words, and write in clear, understandable full sentences. Do not use a term or abbreviation that you made up during the work. Say what the thing is in a way any reader would understand. Write mostly in the style of ASD-STE100 Simplified Technical English: short sentences, one idea per sentence, active voice, and consistently using the same word for the same thing. Use a paragraph of flowing prose where ideas connect, and keep bullets for lists of separate items. Leave out unnecessary details unless the reader asks and practice concision, but do ensure that your messages have the key context. Aim to make your user-facing text as human-readable and clear as possible.
```

在 Anthropic 针对代理任务的测试中，这段文本使最终报告不那么密集，并减少了分号的数量。在工作流自动化任务中，它还使句子变得更短。根据您的用例，您可能还需要添加有关长度的指令，或要求模型包含特定细节。

## 安全防护拒绝

Claude Fable 5.2 运行的安全分类器可能会拒绝请求。被拒绝的请求会返回 `stop_reason: "refusal"`，并由 `stop_details.category` 指明[拒绝类别](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)：

* `cyber`：该请求可能促成网络危害，例如恶意软件或漏洞利用开发。
* `bio`：该请求可能促成生物危害，例如危险的实验室方法。
* `frontier_llm`：该请求可能有助于开发与之竞争的 AI 模型。
* `reasoning_extraction`：该请求要求模型在回复文本中重现其内部推理。
* `general_harms`：该请求属于其他使用政策领域。
* `null`：该拒绝不对应任何已命名的类别。

良性的网络安全和生命科学工作也可能触发 `cyber` 和 `bio` 分类器。如果这阻碍了您组织的正当工作，请参阅 Anthropic 的 [Cyber Verification Program](https://support.claude.com/en/articles/14604842) 和[生命科学验证计划](https://www.anthropic.com/news/life-sciences-verification-program)。[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) 提供与 Claude Fable 5.2 相同的能力，但仅面向通过这些计划验证的组织。

如果您的提示、技能或工具描述要求模型在回复中写出其推理，请删除这些指令。要查看模型的推理过程，请设置 `display: "summarized"` 并阅读思考块。请参阅[将推理保留在思考块中](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#keep-reasoning-in-thinking-blocks)。

要在另一个模型上重试被拒绝的请求，请参阅[拒绝、回退和计费](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#refusals-fallback-and-billing)。

## 从哪里开始

1. 将 `effort` 显式设置为 `high`，然后也在 `medium` 和 `xhigh` 下运行您的评估。
2. 阅读您沿用的系统提示、技能、代理文件、记忆文件、笔记和 harness 文本，并删除或测试[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)中描述的内容。
3. 运行您最长的代理任务，并记录模型在哪些地方过早停止、请求不需要的审批、不再输出、委派超出任务所需，或者在您希望重构的地方只是在现有代码上添加内容。
4. 针对您观察到的每种行为，逐一添加相应文本或遵循相应建议，并在您自己的任务上测量其效果和令牌成本。
