---
title: Claude Haiku 5.5 迁移指南
url: https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide
description: 借助本迁移指南，从早期 Haiku 模型切换到 Claude Haiku 5.5。启用 Claude Haiku 5.5 的指导内容包括新的模型 ID、会返回错误的设置、思考方面的变更，以及针对每个起始模型的检查清单。
---

<Note>
  本指南涵盖 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 代码的迁移。如果您使用 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)，则除了更新模型名称之外无需进行任何更改。
</Note>

让 Claude Code 为您迁移代码

```text
/claude-api migrate 将此项目迁移到 claude-haiku-5-5
```

本指南介绍如何将调用 Claude Haiku 4.5 的代码迁移到 Claude Haiku 5.5。对于调用 Claude Haiku 3.5 或 Claude Haiku 3 的代码，还需进行[从 Claude Haiku 3.5 及更早的 Haiku 模型迁移到 Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#migrating-from-haiku-35) 中的更改。如果您想改为升级到 Sonnet 或 Opus 模型，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。关于 Claude Haiku 4.5 的可用期限，请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。

## 按起始模型划分的迁移检查清单

请按顺序逐组处理，并在列出您当前模型的那一组之后停止。如果您使用的是 Claude Haiku 4.5，第一组就是完整清单。每一项都是您需要在代码中进行的一处更改。

### 所有起始模型

1. 将模型 ID 替换为您所用平台上的 Claude Haiku 5.5 ID。请参阅[使用 Claude Haiku 5.5 模型 ID](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#use-the-claude-haiku-5-5-model-id)。
2. 重新计算提示的令牌数，并重新审视 `max_tokens` 限制和成本估算，因为相同的文本会计为更多令牌，大尺寸图像也会计为更多视觉令牌。请参阅[重新计算令牌](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#recount-tokens)。
3. 如果您的请求发送 `thinking: {"type": "enabled", "budget_tokens": N}`，请将 `thinking` 改为 `{"type": "adaptive"}`。请参阅[配置思考](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#configure-thinking)。
4. 如果您的代码将第一个内容块读取为答案，请改为按 `type` 选择内容块。请参阅[配置思考](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#configure-thinking)。
5. 从您的请求中移除 `temperature`、`top_p` 和 `top_k`。请参阅[移除采样参数](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#remove-sampling-parameters)。
6. 如果您的请求以 assistant 轮次结束 `messages` 以便模型继续生成，请改为以 user 轮次结束。请参阅[替换 assistant 预填充](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#replace-assistant-prefill)。
7. 如果您使用计算机使用功能，请将 `computer_20250124` 替换为 `computer_toolset_20260801` 工具集。在 Amazon Bedrock 上，`computer_20251124` 也可以使用。请参阅[将计算机使用迁移到工具集](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#computer-use-toolset)。
8. 如果您通过不同的账户重放已存储的对话，请通过生成每个对话的账户来重放该对话。请参阅[通过生成思考块的账户重放思考块](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#replay-thinking-blocks-through-the-producing-account)。
9. 如果您的代码在同一对话的请求之间更改 `system`、`tools` 或较早的 `messages`，并将思考块发回，请保持对话仅追加。请参阅[保持较早的轮次不变](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#keep-earlier-turns-unchanged)。
10. 处理 `stop_reason: "refusal"`。Claude Haiku 5.5 运行的安全分类器可能会拒绝请求，并且它没有服务器端回退机制。请参阅[安全防护拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5#safeguard-refusals)。

如果您的组织在 Claude Haiku 4.5 上有 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers#supported-models) 承诺，请单独规划容量：Claude Haiku 5.5 不支持 Priority Tier。

### Claude Haiku 3.5 或更早版本

1. 将 Claude Haiku 3.5 或 Claude Haiku 3 的模型 ID 替换为您所用平台的 Claude Haiku 5.5 ID。请参阅[从 Claude Haiku 3.5 及更早的 Haiku 模型迁移到 Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#migrating-from-haiku-35)。
2. 如果您使用 `code_execution_20250522` 工具，请迁移到 `code_execution_20250825` 或更高版本。
3. 如果您使用文本编辑器工具，请迁移到 `text_editor_20250728`。
4. 处理 `refusal` 和 `model_context_window_exceeded` 停止原因。
5. 如果您的代码对工具调用的字符串参数进行精确匹配，请允许末尾换行符的存在。
6. 检查您的提示。

## 使用 Claude Haiku 5.5 模型 ID

将 Claude Haiku 4.5 模型 ID 替换为您所用平台的 Claude Haiku 5.5 ID。

| 平台                                                | Claude Haiku 4.5                                 | Claude Haiku 5.5             |
| ------------------------------------------------- | ------------------------------------------------ | ---------------------------- |
| Claude API                                        | `claude-haiku-4-5-20251001` 或 `claude-haiku-4-5` | `claude-haiku-5-5`           |
| Amazon Bedrock                                    | `anthropic.claude-haiku-4-5`                     | `anthropic.claude-haiku-5-5` |
| Claude Platform on AWS                            | `claude-haiku-4-5`                               | `claude-haiku-5-5`           |
| Claude Models on Google Cloud (Anthropic managed) | `claude-haiku-4-5-20251001`                      | `claude-haiku-5-5`           |
| Claude Models on Google Cloud (Google managed)    | `claude-haiku-4-5@20251001`                      | `claude-haiku-5-5`           |
| Microsoft Foundry                                 | `claude-haiku-4-5`                               | `claude-haiku-5-5`           |

`claude-haiku-5-5` 是一个固定的模型 ID，没有日期后缀，也没有单独的别名。

## 重新计算令牌

Claude Haiku 5.5 使用与 Claude 4.7 及更高版本模型相同的 tokenizer（分词器）。与所有使用此分词器的模型一样，相同的输入文本在 Claude Haiku 5.5 上产生的令牌数比在 Claude Haiku 4.5 上多约 30%。具体增幅取决于内容。请求、响应和 streaming（流式传输）事件的结构保持不变。发生变化的是您以令牌为单位衡量或预算的任何内容：

* 对于相同的文本，`usage` 字段和[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)结果会更高。
* 相同数量的令牌所能容纳的文本更少。
* 针对 Claude Haiku 4.5 调整的 `max_tokens` 限制可能会截断等量的输出。
* 基于 Claude Haiku 4.5 令牌计数得出的成本估算需要使用 Claude Haiku 5.5 的计数和价格重新计算，包括其针对长提示的更高价格。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)和[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)。

大图像也可能消耗更多令牌。Claude Haiku 5.5 使用高分辨率图像层级，会对长边超过 2,576 像素或超过 4,784 个视觉令牌的图像进行缩小。Claude Haiku 4.5 及更早的 Haiku 模型使用标准层级，会对长边超过 1,568 像素或超过 1,568 个视觉令牌的图像进行缩小。一张 2,000 × 1,500 像素的图像在 Claude Haiku 5.5 上消耗的视觉令牌约为 Claude Haiku 4.5 上的 2.5 倍。请参阅[分辨率和令牌成本](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)。

请在将 `model` 设置为 `claude-haiku-5-5` 的情况下计算提示的令牌数，而不是沿用在 Claude Haiku 4.5 上测得的计数。

## 配置思考

Claude Haiku 5.5 配置思考的方式与 Claude Haiku 4.5 不同。`thinking` 值为 `{"type": "enabled", "budget_tokens": N}` 时会返回 400 错误，因此发送该值的请求需要使用新的 `thinking` 值。

之前，发送到 Claude Haiku 4.5 的请求将 `thinking` 设置为 `enabled` 并指定令牌预算：

```json
{
  "model": "claude-haiku-4-5",
  "max_tokens": 16000,
  "thinking": { "type": "enabled", "budget_tokens": 8000 },
  "messages": [{ "role": "user", "content": "..." }]
}
```

之后，发送到 Claude Haiku 5.5 的相同请求使用 adaptive thinking（自适应思考）。`thinking` 值发生变化，并由 `output_config.effort` 设置模型的思考量：

```json
{
  "model": "claude-haiku-5-5",
  "max_tokens": 16000,
  "thinking": { "type": "adaptive" },
  "output_config": { "effort": "medium" },
  "messages": [{ "role": "user", "content": "..." }]
}
```

自适应思考默认开启，因此即使请求未设置 `thinking`，响应也可能以一个或多个 `thinking` 块开头。请保持 `thinking` 未设置或将其设置为 `{"type": "adaptive"}`，并使用 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 作为控制手段：在 Claude Haiku 4.5 不使用思考或使用较小预算以节省令牌的场景中，请选择较低的 effort 级别。在较低级别下，模型思考更少，并且对于较简单的请求可以完全跳过思考。有关提示指导，请参阅[使用 effort 控制思考](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5#use-effort-to-control-thinking)。请按内容块的 `type` 字段而非位置来选择内容块，并将 `thinking` 块原样连同工具结果一起传回。

思考令牌计入 `max_tokens`，因此 `max_tokens` 较小的请求可能会在一个 `thinking` 块之后、任何文本之前以 `stop_reason: "max_tokens"` 停止。如果您为 Claude Haiku 4.5 设置了较小的 `max_tokens`，请提高该值以为思考留出空间，或选择较低的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。

来自较早助手轮次的思考块会保留在上下文中并计为输入令牌，而 Claude Haiku 4.5 只保留最新轮次的思考块。因此，多轮对话携带的输入令牌会比仅由分词器变更所能解释的更多。要移除较旧的思考块，请使用[思考块清除](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#thinking-block-clearing)。请参阅[各模型的思考块保留](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)。

默认情况下，Claude Haiku 5.5 返回的每个 `thinking` 块的 `thinking` 字段为空，仅包含 `signature`，而 Claude Haiku 4.5 返回摘要形式的思考。要接收摘要形式的思考，请设置 `thinking: {"type": "adaptive", "display": "summarized"}`。

Claude Haiku 5.5 接受强制的 `tool_choice`（`any` 或指定名称的工具），但响应会以工具调用开头，且没有 `thinking` 块。要让模型在调用工具之前进行思考，请使用 `tool_choice: {"type": "auto"}`，并在提示中说明何时使用该工具。

从 Claude Haiku 4.5 切换到 Claude Haiku 5.5 的对话会保留其推理过程。对于其他切换，则取决于所涉及的两个模型：

| 切换                                                                                            | 是否保留推理过程                                                          |
| --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 切换到 Claude Haiku 5.5，来源为 Claude Haiku 4.5 或更早版本、Claude Sonnet 5 或更早版本，或 Claude Opus 4.8 或更早版本 | 是                                                                 |
| 切换到 Claude Haiku 5.5，来源为任何其他模型                                                                | 否。API 会丢弃该模型的思考块而不报错。                                             |
| 从 Claude Haiku 5.5 切换到 Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.2 或 Claude Mythos 5.2 | 是，但仅限 Claude API 和 Claude Models on Google Cloud (Google managed) |
| 从 Claude Haiku 5.5 切换到任何其他模型                                                                  | 否                                                                 |

请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。对于不涉及 Claude Haiku 5.5 的切换，请参阅[在 Claude API 上哪些模型切换会保留推理](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#which-model-switches-keep-reasoning)。

## 移除采样参数

在 Claude Haiku 5.5 上，请省略 `temperature`、`top_p` 和 `top_k`，改用提示来引导模型的行为。

## 替换助手预填充

预填充（prefill）是 `messages` 中供模型续写的最后一个助手轮次。Claude Haiku 4.5 在思考关闭时接受预填充。Claude Haiku 5.5 即使在思考关闭时也会以 400 错误拒绝预填充。请以用户轮次结束 `messages`，并根据每个预填充的用途进行替换：

* **输出格式：** 使用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)，或在分类场景中使用带有枚举字段的工具。
* **开场白：** 在系统提示中要求直接给出答案。
* **续写：** 将其移到用户消息中，例如"您之前的回复被中断了，结尾是 `[previous_response]`。请从中断处继续。"
* **上下文提醒：** 将其放入用户轮次中。

## 将计算机使用迁移到工具集

Claude Haiku 4.5 通过 `computer_20250124` 工具并配合 `computer-use-2025-01-24` beta 请求头支持 “computer use”（[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)）。Claude Haiku 5.5 通过 `computer_toolset_20260801` 工具集支持计算机使用。向 Claude Haiku 5.5 发送声明了 `computer_20250124` 的请求会返回 400 错误。

要将集成迁移到工具集，请去掉 `computer-use-2025-01-24` beta 请求头，并将 `tools` 条目替换为 `{"type": "computer_toolset_20260801"}`。然后按照[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)中的说明，对请求和智能体循环进行其他更改。根据每个成员 `tool_use` 块的 `name` 和 `toolset_name` 进行分派，而不是根据 `input.action`。处理一个轮次中的每个此类块，并在结果中回传 `toolset_name`。工具集中的缩放功能默认开启；如果您的环境未实现该功能，请添加 `"configs": {"zoom": {"enabled": false}}`。如果您发送了 `fine-grained-tool-streaming-2025-05-14` beta 请求头，请将其移除。与工具集条目一起使用时，它会返回 400 错误。

在 Amazon Bedrock 上，Claude Haiku 5.5 还接受配合 `computer-use-2025-11-24` beta 请求头使用的 `computer_20251124` 工具。

Claude Haiku 5.5 支持用于网页内任务的[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)（`browser_toolset_20260801`）。Claude Haiku 4.5 不支持该工具。

## 通过生成思考块的账户重放思考块

来自 Claude Haiku 5.5 的思考块只能在生成它们的账户或与其关联的账户中使用。当另一个账户发送此类思考块时，API 会在模型接收之前丢弃该块，请求仍会成功，但不包含该推理内容。这会影响存储对话并通过不同账户重放对话的代码，例如从同一个对话存储为多个客户提供服务的服务。请通过生成对话的账户重放每个对话。请参阅[思考块始终归属于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。

## 保持较早的轮次不变

Claude Haiku 5.5 的思考块仅在其之前发送的所有内容保持不变时才有效：包括 `system` 提示、`tools` 以及之前的每条消息。Claude Haiku 4.5 不执行前缀检查。请保持对话为仅追加模式。如果在发生此类更改后仍将思考块发回，且未设置 `thinking.block_binding.prefix_mismatch_behavior`，则该请求会收到 400 错误。这适用于在 2026 年 8 月 31 日 00:00 UTC 或之后创建的账户。对于在该日期之前创建的账户，该请求可能会也可能不会收到该错误。有关会触发该错误的更改以及应采取的替代做法，请参阅[前缀检查会影响哪些用户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#who-is-affected)。

## 从 Claude Haiku 3.5 及更早的 Haiku 模型迁移到 Claude Haiku 5.5

Claude Haiku 3.5 已在 Claude API、Amazon Bedrock 和 Claude Models on Google Cloud (Google managed) 上停用。Claude Haiku 3 已在 Claude API 和 Claude Models on Google Cloud (Google managed) 上停用。请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。

无论从哪个模型迁移，都请先应用前面的所有部分，然后进行以下更改：

* **模型 ID：** 将 `claude-3-5-haiku-20241022`、其别名 `claude-3-5-haiku-latest` 或 `claude-3-haiku-20240307` 替换为 `claude-haiku-5-5`。在 Claude Models on Google Cloud (Google managed) 上，将 `claude-3-5-haiku@20241022` 替换为 `claude-haiku-5-5`。在 Amazon Bedrock 上，请使用[使用 Claude Haiku 5.5 模型 ID](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#use-the-claude-haiku-5-5-model-id) 中的 Claude Haiku 5.5 ID。
* **代码执行：** Claude Haiku 5.5 接受 `code_execution_20250825` 及更高版本。如果您使用仅支持 Python 的 `code_execution_20250522`，请迁移到其中一个版本。请参阅[升级到最新工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#upgrade-to-latest-tool-version)。
* **文本编辑器：** 如果您使用文本编辑器工具，请迁移到 `text_editor_20250728`（工具名称为 `str_replace_based_edit_tool`），该版本没有 `undo_edit` 命令。请参阅[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)。
* **停止原因：** 处理 `refusal` 和 `model_context_window_exceeded`。请参阅[停止原因和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)。
* **末尾换行符：** Claude 4.5 及更高版本模型会在工具调用的字符串参数中保留末尾换行符。如果您的代码对这些字符串进行精确匹配，请允许末尾换行符的存在。
* **提示：** Claude 4 及更高版本模型的沟通风格更简洁、直接，需要明确的指示。请对照[为 Claude Haiku 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5)和[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)检查您的提示。
