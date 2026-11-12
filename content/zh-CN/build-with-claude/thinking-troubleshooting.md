---
title: 思考功能故障排查
url: https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting
description: 诊断并修复最常见的思考功能故障：配置导致的 400 错误、空的或缺失的思考块、max_tokens 停止以及缓存未命中。
---

<Note>
  要了解"zero data retention"（零数据保留），即 ZDR 如何适用于此功能，请参阅 [API 与数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)。
</Note>

本页涵盖配置思考功能或往返传递思考块（即在后续请求中将返回的思考块发送回去）时最常见的故障。第一部分列出了每个模型所支持的思考配置以及它会拒绝的配置；其后的各部分均从您观察到的症状出发，以便您可以将错误消息或意外响应直接对应到其原因和修复方法。要了解思考功能的工作原理，请参阅[思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)概述。

## 各模型的思考支持、默认值和被拒绝的配置

大多数思考配置错误都源于请求中的 `thinking.type` 值与模型支持的内容不匹配。在大多数模型上，思考以 `thinking: {type: "adaptive"}` 的形式运行，并且许多模型默认开启思考。一些较早的模型则使用 “extended thinking”（[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)），这是一种旧版手动模式，配置为 `thinking: {type: "enabled", budget_tokens: N}`。

“Extended thinking”（扩展思考）（`thinking.type: "enabled"` 搭配 `budget_tokens`）在 Claude 4.6 模型上已被弃用（使用它的请求仍会成功）。Claude 4.7 及更高版本的模型不支持它，并会拒绝使用它的请求，返回 400 错误。在支持思考的 Claude 4.5 及更早版本的模型上，扩展思考是唯一可用的思考模式。Claude Mythos Preview 同时支持两种模式。在两种模式均可用的情况下，请改用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。

下表列出了每个模型支持的内容、默认值，以及它会以 400 错误拒绝的 `thinking.type` 值；未列为被拒绝的值均可接受。只有 Claude Sonnet 5.5 接受 `"between_tools"`，并且它用该值代替 `"disabled"`。

| 模型                    | 思考类型                            | 默认值  | 以 400 拒绝                             |
| --------------------- | ------------------------------- | ---- | ------------------------------------ |
| Claude Fable 5.2      | 仅自适应                            | 始终开启 | `"enabled"`、`"disabled"`             |
| Claude Mythos 5.2     | 仅自适应                            | 始终开启 | `"enabled"`、`"disabled"`             |
| Claude Fable 5.1      | 仅自适应                            | 始终开启 | `"enabled"`、`"disabled"`             |
| Claude Mythos 5.1     | 仅自适应                            | 始终开启 | `"enabled"`、`"disabled"`             |
| Claude Fable 5        | 仅自适应                            | 始终开启 | `"enabled"`、`"disabled"`             |
| Claude Mythos 5       | 仅自适应                            | 始终开启 | `"enabled"`、`"disabled"`             |
| Claude Mythos Preview | 自适应、扩展                          | 始终开启 | `"disabled"`                         |
| Claude Opus 5.5       | 仅自适应                            | 始终开启 | `"enabled"`、`"disabled"`             |
| Claude Opus 5         | 仅自适应                            | 开启   | `"enabled"`、`"disabled"`<sup>2</sup> |
| Claude Opus 4.8       | 仅自适应                            | 关闭   | `"enabled"`                          |
| Claude Opus 4.7       | 仅自适应                            | 关闭   | `"enabled"`                          |
| Claude Sonnet 5.5     | 自适应、`between_tools`<sup>3</sup> | 开启   | `"enabled"`、`"disabled"`             |
| Claude Sonnet 5       | 仅自适应                            | 开启   | `"enabled"`                          |
| Claude Haiku 5.5      | 仅自适应                            | 开启   | `"enabled"`、`"disabled"`<sup>2</sup> |
| Claude Opus 4.6       | 自适应、扩展（已弃用）<sup>1</sup>         | 关闭   | 无                                    |
| Claude Sonnet 4.6     | 自适应、扩展（已弃用）<sup>1</sup>         | 关闭   | 无                                    |
| Claude Opus 4.5       | 仅扩展                             | 关闭   | `"adaptive"`                         |
| Claude Haiku 4.5      | 仅扩展                             | 关闭   | `"adaptive"`                         |
| Claude Sonnet 4.5     | 仅扩展                             | 关闭   | `"adaptive"`                         |

*<sup>1</sup> `enabled` 和 `budget_tokens` 在这些模型上仍然有效，但已弃用；请改用自适应思考。*\
*<sup>2</sup> Claude Opus 5 和 Claude Haiku 5.5 在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 为 `high` 或更低时接受 `"disabled"`；将其与 effort `xhigh` 或 `max` 组合使用会返回 400 错误。在 Claude Opus 5 上，此限制针对每个请求强制执行。在 Claude Haiku 5.5 上，与当前生效级别不同的按消息设置的 effort 也会返回 400 错误。*\
*<sup>3</sup> Claude Sonnet 5.5 在 effort 为 `high` 或更低时接受 `"between_tools"`。将其与 effort `xhigh` 或 `max` 组合使用会返回 400 错误，与当前生效级别不同的按消息设置的 effort 也会返回 400 错误。*

标记为`始终开启`的模型无法关闭思考。标记为`开启`的模型默认进行思考。Claude Opus 5、Claude Sonnet 5 和 Claude Haiku 5.5 接受 `thinking: {type: "disabled"}`。在 Claude Sonnet 5.5 上，发送 `thinking: {type: "between_tools"}` 以关闭前置思考。

较早的 Claude 4 模型（Claude Opus 4.1、Claude Sonnet 4 和 Claude Opus 4）仅支持扩展思考。有关其可用性，请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。除非获得 Anthropic 的明确授权，否则 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5 和 Claude Mythos 5 在[零数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)下不可用。

## 400 错误提示不支持 `"thinking.type.enabled"`

请求失败并返回 400 错误，其消息内容为：

```text wrap
"thinking.type.enabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

出现此错误是因为您请求的模型已移除扩展思考（请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#rejected-configurations)）。

将请求切换为 `thinking: {type: "adaptive"}`，并使用 `effort` 而非 `budget_tokens` 来控制思考深度。[迁移到自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#migrating-to-adaptive-thinking)详细介绍了转换过程。

## 发送 `thinking: {type: "disabled"}` 后出现 400 错误

请求失败并返回 400 错误。在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5 和 Claude Opus 5.5 上，消息为：

```text wrap
"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

在 Claude Mythos Preview（这些模型中唯一接受扩展思考的模型）上，消息为：

```text wrap
"thinking.type.disabled" is not supported for this model. Thinking defaults to adaptive mode when not specified; use "thinking.type.enabled" with "budget_tokens" for extended thinking.
```

出现此错误是因为所有这些模型的思考始终开启（请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#rejected-configurations)）。

省略 `thinking` 参数；这些模型无需任何配置即可进行思考。如果您的目标是让响应中不包含思考文本，请使用 `display: "omitted"` 而不是禁用思考；请参阅[控制思考显示](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)。

`"disabled"` 导致的 400 错误也可能出现在 Claude Opus 5 和 Claude Haiku 5.5 上，它们仅在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 为 `high` 或更低时接受 `thinking: {type: "disabled"}`：将其与 effort `xhigh` 或 `max` 组合使用会被拒绝。请降低 effort 级别，或将 `thinking` 设置为 `{"type": "adaptive"}`。

在 Claude Sonnet 5.5 上，`thinking: {type: "disabled"}` 在任何 effort 级别下都会返回 400 错误。消息为：

```text wrap
To turn thinking off on this model, send "thinking": {"type": "between_tools"} instead of {"type": "disabled"}. The model does not think before responding. The short updates it writes between tool calls come back as thinking blocks.
```

要在 Claude Sonnet 5.5 上关闭前置思考，请改为发送 `thinking: {type: "between_tools"}`，且 effort 为 `high` 或更低。

## 400 错误提示不支持 `"thinking.type.between_tools"`

请求失败并返回 400 错误，消息内容如下：

```text wrap
"thinking.type.between_tools" is not supported for this model.
```

出现此错误是因为只有 Claude Sonnet 5.5 接受 `thinking: {type: "between_tools"}`（请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#rejected-configurations)）。

请仅向 Claude Sonnet 5.5 发送 `between_tools`。在其他模型上，请省略 `thinking`，或使用表中未列为被拒绝的 `thinking.type` 值。

## 400 错误提示禁用思考时不支持某个 effort 级别

在 Claude Sonnet 5.5 上，effort 为 `xhigh` 或 `max` 且带有 `thinking: {type: "between_tools"}` 的请求会失败并返回 400 错误，其消息为：

```text wrap
output_config.effort 'xhigh' is not supported when thinking is disabled on this model. Use effort 'high' or below, or enable thinking.
```

出现此错误是因为 Claude Sonnet 5.5 仅在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 为 `high` 或更低时接受 `between_tools`。消息中说思考已禁用，是因为 `between_tools` 没有前置思考，即使请求并未发送 `"disabled"`。消息会指出请求发送的级别。

将 effort 降低到 `high` 或更低。要以 `xhigh` 或 `max` 运行，请使用自适应思考：省略 `thinking` 字段或发送 `thinking: {"type": "adaptive"}`。这就是消息中"enable thinking"的含义。Claude Sonnet 5.5 会以 400 错误拒绝 `"enabled"`。

当请求在 effort 为 `xhigh` 或 `max` 时发送 `thinking: {type: "disabled"}`，Claude Haiku 5.5 会返回相同的消息。它仅在 `high` 或更低时接受 `"disabled"`。请降低 effort，或将 `thinking` 设置为 `{"type": "adaptive"}`。

## 400 错误提示禁用思考时 effort 不能更改

在 Claude Sonnet 5.5 上，带有 `thinking: {type: "between_tools"}` 且其[按消息设置的 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) 更改了级别的请求会失败并返回 400 错误，其消息为：

```text wrap
messages.N: output_config.effort 'low' differs from the 'high' in effect before it; effort cannot change when thinking is disabled on this model. Use effort 'high', or enable thinking.
```

消息称思考已禁用，是因为 `between_tools` 没有前置思考。使用 `between_tools` 时，effort 不能在对话中途更改：与当前生效级别不同的按消息设置的 `output_config.effort` 会返回 400 错误。`messages.N` 是设置新级别的消息所在的位置。

移除该每条消息 effort，或将其设置为当前生效的级别。要在每个轮次中改变 effort，请使用自适应思考：省略 `thinking` 字段或发送 `thinking: {"type": "adaptive"}`。这就是消息中"enable thinking"的含义。

当使用 `thinking: {type: "disabled"}` 的请求设置了与当前生效级别不同的每条消息 effort 时，Claude Haiku 5.5 会返回相同的消息。请移除该每条消息 effort，或将 `thinking` 设置为 `{"type": "adaptive"}` 以在每个轮次中改变 effort。

## 400 错误提示不支持自适应思考

请求失败并返回 400 错误，其消息内容为：

```text wrap
adaptive thinking is not supported on this model
```

出现此错误是因为该模型仅支持扩展思考（请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#rejected-configurations)）。

请改用 `thinking: {type: "enabled", budget_tokens: N}`；有关配置，请参阅[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)。

## 400 错误提示思考块不能被修改

返回工具结果的请求失败，并返回 400 `invalid_request_error`，其消息包含：

```text wrap
`thinking` or `redacted_thinking` blocks in the latest assistant message cannot be modified
```

在多轮对话和工具使用对话中，您会将之前的助手消息（包括其 `thinking` 和 `redacted_thinking` 块）发送回 API，API 会验证它们是否未经修改。当您发送回的助手消息与 API 返回的不同时，就会出现此错误，最常见的原因是您的代码按类型过滤内容块并丢弃了 `redacted_thinking` 块，或者重新构建了助手消息而不是原样回传。

请将助手轮次原样回传，包括思考块。有关规则，请参阅[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)；有关各 SDK 中的正确代码，请参阅[工具和多轮工作流中的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-tool-workflows#two-turn-tool-use-round-trip)中的完整往返示例。

## 400 错误提示思考块签名无效

向 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 或 Claude Haiku 5.5 重放先前思考块（thinking blocks）的请求可能会失败，并返回 400 `invalid_request_error`，其消息内容为：

```text wrap
messages.{i}.content.{j}: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block".
```

如果请求未发送 `thinking-binding-controls-2026-08-01` beta 标头，消息会附加 ``That setting requires the `thinking-binding-controls-2026-08-01` value in the `anthropic-beta` header.``

消息通常以一句说明更改内容的句子结尾：`system` 提示、`tools` 列表、第一个不同的消息或块、缺失或新增的内容，或者缺失或顺序错乱的早期思考块。这句话是供人阅读和记录日志用的。其措辞可能会变化，因此不要在代码中对其进行匹配。

如果消息在 ``Invalid `signature` in `thinking` block`` 之后就结束了，说明签名本身未通过验证：它被截断、修改或以空值发送回来，此时 `prefix_mismatch_behavior` 不适用。编辑过的思考文本会返回不同的错误。请参阅[400 错误提示思考块不能被修改](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#error-thinking-blocks-modified)。

这些模型仅在其之前的 `system` 提示、`tools` 和消息保持不变时，才接受重放的思考块。请参阅[保持前缀不变](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)。该错误表示对话中较早的某些内容在两次请求之间发生了变化。常见原因包括以下几种：

* 某个轮次被编辑、重新排序或删除
* 某个按轮次注入的提醒后来被移除
* `system` 提示或 `tools` 数组被重新构建
* 客户端压缩原样保留了最近的轮次及其思考内容

服务器端[在令牌阈值处压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)和[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)永远不会触发此错误。使用[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)时，如果某个[保留思考的条件](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-thinking-blocks#conditions-for-kept-thinking-to-stay-valid)不成立，您在摘要之后保留的轮次中的思考内容就会触发此错误。对于未设置 `thinking.block_binding.prefix_mismatch_behavior` 的请求，请参阅[未设置 `prefix_mismatch_behavior` 时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)。

重试相同的请求体并不能清除该错误。修复您的代码也不能，因为存储的历史记录中保留了该编辑。要继续对话，请执行以下操作之一：

* 发送 `thinking-binding-controls-2026-08-01` beta 标头，并在重试请求及之后的每个请求上将 `thinking.block_binding.prefix_mismatch_behavior` 设置为 `"drop_block"`。这样请求就会成功，且不包含已失效的推理。在 Claude Sonnet 5.5 和 Claude Haiku 5.5 上，`block_binding` 仅适用于 `thinking: {"type": "adaptive"}`。在这些模型上使用任何其他思考设置时，您无法发送 `block_binding`，因此请使用下一个选项。
* 删除错误中指明的块以及之后的所有 `thinking` 和 `redacted_thinking` 块，保留每个轮次的其他块，然后重试一次。

然后修复根本原因，以免新的对话再遇到该错误。保持历史记录仅追加：按照发送和接收时的原样传回之前的轮次，使用[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)添加指令，而不是编辑 `system` 或 `tools`，并让服务器端的[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)或[压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)来完成任何裁剪。

来自目标模型无法读取的模型的块永远不会产生此错误：API 会丢弃它，并在使用 beta 标头时在 `input_transformations` 中报告。

## 响应中的 thinking 字段为空

响应包含 `thinking` 块，但其 `thinking` 字段为空字符串，只有 `signature` 字段有值。

出现这种情况是因为在较新的模型上 `display` 默认为 `"omitted"`，这会返回不含文本的思考块。

在思考配置中设置 `display: "summarized"` 以接收摘要后的思考文本。有关各模型的默认值，请参阅[控制思考显示](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)。如果您只想要某些模型在工具调用之间写出的简短状态行，而不需要推理内容，请改为设置 `display: "updates"`（beta）。请参阅[工具调用之间的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)。

`thinking` 字段为空的块仍然是完整的：`signature` 中保存着推理内容。请像处理其他块一样，将其随轮次一起发回。请参阅[按返回时的原样发回助手轮次](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#append-assistant-turns-exactly-as-returned)。

## 某些轮次没有出现思考块

即使已配置思考，某些响应也完全不包含 `thinking` 块。

这在自适应模式下是正常的：Claude 会对它判断为足够简单、可以直接回答的请求跳过思考。

如果您希望更频繁或更深入地进行思考，请提高 `effort` 或通过提示进行引导；请参阅[引导 Claude 的思考频率](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#tuning-thinking-behavior)。

强制的 `tool_choice`（`{"type": "any"}` 或指定的工具）也不会返回 `thinking` 块：响应以工具调用开始。要让模型在调用工具之前进行思考，请使用 `tool_choice: {"type": "auto"}`，并在提示中说明何时使用该工具。

## 文本输出中出现工具调用或 XML 标签

响应偶尔会将工具调用写入其文本中，而不是发出 `tool_use` 块，或者在其可见文本中包含 `<thinking>` 或其他内部 XML 标签。泄漏的工具调用永远不会运行，并且在智能体循环中，泄漏的文本会保留在对话历史中，因此后续轮次也会受到影响。

这种情况发生在禁用思考的 Claude Opus 5 上，最常见于搜索等大量使用工具的工作负载。指示模型不要思考或不要推理的系统提示规则会加剧标签泄露。

请重新启用思考（默认设置），并改用较低的 `effort` 级别来控制令牌成本。如果您的集成必须保持思考禁用，请应用[在禁用思考的情况下运行](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5#running-with-thinking-disabled)中的提示缓解措施。

## 响应以 `stop_reason: "max_tokens"` 停止

响应以 `stop_reason: "max_tokens"` 结束，通常伴随被截断或缺失的文本块。

出现这种情况是因为思考令牌计入 `max_tokens`，因此较长的思考过程可能会在文本响应完成之前耗尽预算。

请提高 `max_tokens` 以便为思考和文本都留出空间，或降低 `effort` 以使 Claude 在思考上花费更少；请参阅[成本控制](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#cost-control)和[思考与上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-the-context-window)。

## 更改思考设置后缓存命中率下降

在之前命中缓存的请求上，`cache_read_input_tokens` 降为零。

出现这种情况是因为思考配置和 effort 级别（或其默认值）是缓存提示前缀的一部分，因此更改其中任何一项都会开始一个新的前缀：切换思考模式、更改 effort 值以及更改 `budget_tokens` 都会使消息缓存断点失效，并且根据模型渲染配置的位置，还可能使工具和系统提示断点失效。

请在共享同一对话的请求之间保持思考配置和 effort 级别不变；将参数显式设置为其默认值等同于省略它，不会导致失效。请参阅[思考与提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-prompt-caching)。

## 设置 effort 不会改变思考

您更改了 `effort`，但思考频率或深度保持不变。

出现这种情况是因为 effort 仅在自适应模式下是主要的思考调节手段。在仅支持扩展思考的模型上，思考深度由 `budget_tokens` 设置。

请在这些模型上调整 `budget_tokens`，或检查您的模型运行在哪种模式下；请参阅[思考与 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-effort)。在 Claude Opus 4.5（唯一支持 effort 的仅扩展思考模型）上，effort 与预算共同作用；请参阅[预算规则与调优](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#budget-rules-and-tuning)。

## 后续步骤

<CardGroup cols={3}>
  <Card title="思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    概述：什么是思考、如何配置它，以及它如何与工具、缓存和流式传输交互。
  </Card>

  <Card title="错误" icon="book" href="https://platform.claude.com/docs/zh-CN/api/errors">
    完整的错误参考，包括思考配置相关的 400 错误及其确切的服务器消息。
  </Card>

  <Card title="迁移到自适应思考" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking#migrating-to-adaptive-thinking">
    将 `budget_tokens` 请求转换为使用 effort 的自适应思考。
  </Card>
</CardGroup>
