---
title: Claude Sonnet 5.5 的新变化
url: https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5
description: 从 Claude Sonnet 5 迁移到 Claude Sonnet 5.5 时会发生哪些变化：破坏性变更、功能支持、行为差异、定价和可用性。
---

Claude Sonnet 5.5 提供了速度与智能的最佳组合。有五项破坏性变更可能会影响已在 Claude Sonnet 5 上运行的代码：

* [使用 `between_tools` 关闭前置思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#turn-off-up-front-thinking)。
* [强制工具使用会返回错误](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#forced-tool-use-is-not-supported)。
* [思考块与模型和对话绑定](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#thinking-blocks-are-tied-to-the-model-that-produced-them)。
* [部分计算机使用工具版本不受支持](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#computer-20251124-is-not-supported)。
* [advisor 工具拒绝将四个模型用作顾问](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#advisor-tool-pairings)：Claude Opus 4.8、Claude Opus 4.7、Claude Sonnet 5 和 Claude Haiku 5.5。

另有一项变更会改变响应结构，但不会导致任何请求失败：[工具调用之间的文本以 `thinking` 块的形式返回](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#text-between-tool-calls)。会将这些文本流式传输给用户的应用程序在工具调用之间会保持静默，直到它设置一个会返回该文本的 `display` 值，或使用 `between_tools` 关闭前置思考。

## 新模型

| 模型                | Claude API ID     | 描述         |
| ----------------- | ----------------- | ---------- |
| Claude Sonnet 5.5 | claude-sonnet-5-5 | 速度与智能的最佳组合 |

“[Adaptive thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)”（自适应思考）默认开启，"[effort parameter](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)"（努力程度参数）用于控制思考深度。该参数在 Claude API 上的默认值为 `high`。"tokenizer"（分词器）与 Claude Sonnet 5 相同，因此相同的文本会产生相同的令牌数。有关 “context window”（上下文窗口）、输出限制、知识截止日期和价格，请参阅 [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) 模型页面。

有关所有当前模型，请参阅[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)。

## 破坏性变更

### 使用 `between_tools` 关闭前置思考

要在 Claude Sonnet 5.5 上关闭前置思考，请发送 `thinking: {"type": "between_tools"}`，而不是 `"disabled"`。这是该模型上最低的思考设置，在所有提供 Claude Sonnet 5.5 的平台上均可用。模型在工具调用之间写下的简短[进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)仍会以 `thinking` 块的形式返回，并附带摘要文本。请将这些块与助手轮次的其余部分一起原样传回。您传回的进度更新块会向模型提供它所写的完整笔记，而不是摘要。如果您的请求不使用工具，响应将只包含文本，与在 Claude Sonnet 5 上使用 `disabled` 时相同。

在 Claude Sonnet 5.5 上，发送 `thinking: {"type": "disabled"}` 的请求会返回 400 `invalid_request_error`，其消息会提示您改用 `between_tools`。

`between_tools` 可在 `low`、`medium` 和 `high` effort 下使用。在 `xhigh` 或 `max` effort 下，带有 `between_tools` 的请求会返回 400 错误。要以 `xhigh` 或 `max` 运行，请使用自适应思考：省略 `thinking` 字段或发送 `thinking: {"type": "adaptive"}`，两者等效。使用 `between_tools` 时，effort 无法在对话中途更改：与当前生效级别不同的[逐消息 `output_config.effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) 会返回 400 错误。要为每个轮次设置不同的 effort，请使用自适应思考。

`between_tools` 不接受其他字段：与其一起发送 `display`、`budget_tokens` 或 `block_binding` 会返回 400 错误。手动思考预算（`thinking: {"type": "enabled", "budget_tokens": N}`）会返回 400 错误。请参阅[思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)以及迁移指南中的[关闭前置思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#turn-off-up-front-thinking)。

### 不支持强制工具使用

Claude Sonnet 5.5 不支持 “forced tool use”（强制工具使用）。将 `tool_choice` 设置为 `{"type": "any"}` 或 `{"type": "tool", "name": "..."}` 会返回 400 `invalid_request_error`：

```text wrap
tool_choice: type "tool" and "any" are not supported for this model.
```

支持 `tool_choice: {"type": "auto"}`（默认值）和 `{"type": "none"}`。同样的检查也适用于[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)端点。要获得符合 schema 的工具输入，请保持 `tool_choice: {"type": "auto"}`，并通过[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)设置 `strict: true`，或将 schema 移至[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。要让模型调用工具而不是以文本回复，请在提示中说明何时适用该工具。迁移指南中的[不支持强制工具使用](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#forced-tool-use)展示了修改前后的对比。

### 思考块与模型和对话绑定

每个思考块都会记录生成它的模型。切换模型的对话只有在切换后的目标模型能够读取先前模型的思考块时，才会保留其先前的推理内容：

| 切换                                                                                          | 是否保留推理内容                                                          |
| ------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 从 Claude Sonnet 5 或更早模型、Claude Opus 4.8 或更早模型，或 Claude Haiku 4.5 或更早模型切换到 Claude Sonnet 5.5 | 是                                                                 |
| 从 Claude Haiku 5.5 切换到 Claude Sonnet 5.5                                                    | 是，但仅限 Claude API 和 Claude Models on Google Cloud (Google managed) |
| 从任何其他模型切换到 Claude Sonnet 5.5                                                                | 否                                                                 |
| 从 Claude Sonnet 5.5 切换到 Claude Opus 5.5、Claude Fable 5.2 或 Claude Mythos 5.2                | 是，但仅限 Claude API 和 Claude Models on Google Cloud (Google managed) |
| 从 Claude Sonnet 5.5 切换到任何其他模型                                                               | 否                                                                 |

当请求携带目标模型无法读取的块时，API 会在模型看到它之前将其丢弃。请求会成功，且被丢弃的块不计费。使用 `thinking-binding-controls-2026-08-01` beta 请求头时，丢弃情况会在顶层 `input_transformations` 数组中报告。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。对于不涉及 Claude Sonnet 5.5 的切换，请参阅[在 Claude API 上哪些模型切换会保留推理](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#which-model-switches-keep-reasoning)。

API 还会检查 Claude Sonnet 5.5 思考块之前的任何内容自该块生成以来是否发生了更改：`system` 提示、`tools` 或更早的消息。请保持对话仅追加：使用[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)来更改指令或工具，而不是进行编辑。

如果请求在此类更改后重放某个块且未设置 `thinking.block_binding.prefix_mismatch_behavior`，则会收到 400 错误。这适用于在 2026 年 8 月 31 日 00:00 UTC 或之后创建的账户。[保留思考的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#feature-availability)列出了适用此规则的平台。对于在该日期之前创建的账户，请求可能会也可能不会收到该错误。如果要改为丢弃受影响的块，请发送 `thinking-binding-controls-2026-08-01` beta 标头并将该字段设置为 `"drop_block"`。

`block_binding` 仅适用于 `thinking: {"type": "adaptive"}`。使用任何其他思考设置时，您无法发送 `block_binding`。请改为删除错误中指明的块以及其后的每个 `thinking` 和 `redacted_thinking` 块。在之后的请求中不要包含它们，然后重试。

请参阅[保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)以及迁移指南中的[关于思考块的说明](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#thinking-blocks)。

### 部分计算机使用工具版本不受支持

Claude Sonnet 5.5 通过 `computer_toolset_20260801` 工具集支持 [computer use](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)（计算机使用）。在 Amazon Bedrock 上，Claude Sonnet 5.5 还接受带有 `computer-use-2025-11-24` beta 标头的 `computer_20251124` 工具。对于任何不接受的 computer use 工具版本（例如 Claude API 上的 `computer_20251124`），Claude Sonnet 5.5 会返回 400 错误。Claude API 的 `invalid_request_error` 消息会指明被拒绝的类型，并在 `Did you mean one of` 之后列出该模型接受的工具类型。该消息开头如下：

```text wrap
'claude-sonnet-5-5' does not support tool types: computer_20251124.
```

使用该工具集的集成无需任何更改。要将 `computer_20251124` 集成迁移到该工具集，请按照[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)操作，其中展示了迁移前后的请求。删除 beta 标头，并将 `tools` 条目替换为 `{"type": "computer_toolset_20260801"}`。然后更新您的智能体循环，以处理成员 `tool_use` 块、批量操作以及结果中的 `toolset_name`。

### 部分 Advisor 工具组合不受支持

[Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)处于 beta 阶段。Claude Sonnet 5.5 执行者仅接受以下顾问：

* Claude Mythos 5.2 和 Claude Fable 5.2
* Claude Mythos 5.1 和 Claude Fable 5.1
* Claude Mythos 5 和 Claude Fable 5
* Claude Opus 5.5 和 Claude Opus 5
* Claude Sonnet 5.5 本身

Claude Opus 4.8、Claude Opus 4.7、Claude Sonnet 5 和 Claude Haiku 5.5 顾问可与 Claude Sonnet 5 执行者配合使用。与 Claude Sonnet 5.5 执行者配合时，它们会返回 400 `invalid_request_error`。Claude Sonnet 5.5 接受的每个顾问都会以加密形式返回其建议，即 `advisor_redacted_result` 块。因此您的客户端无法读取建议文本。请参阅 Advisor 工具的[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#model-compatibility)和[结果变体](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#result-variants)。

## 功能支持

Claude Sonnet 5.5 支持以下功能：

* [按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)，处于 beta 阶段
* [对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)
* [对话中途的工具变更](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#mid-conversation-tool-changes)，处于 beta 阶段
* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)，最小可缓存提示为 512 个令牌
* [批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)
* [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)
* [PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)
* [视觉](https://platform.claude.com/docs/zh-CN/build-with-claude/vision)
* 服务器端和客户端工具（请参阅[使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)）

Claude Sonnet 5 不支持按消息设置 effort、对话中途的系统消息或对话中途的工具变更。在 Claude Sonnet 5 上，最小可缓存提示为 1,024 个令牌。有关模型可用性，请参阅各功能的页面。

### 按需压缩（beta）

使用 `compact-2026-09-04` beta 标头时，发送顶层 `compaction` 参数的请求会返回一个经过签名的 `compaction` 块，其中包含该请求中消息的摘要。在之后的请求中，该块放在 `messages` 的最前面，以替代被摘要的消息。何时压缩以及在该块之后保留哪些轮次由您决定。在 Claude Sonnet 5.5 上，您保留的轮次中的思考块在压缩后仍可保持有效。有关条件，请参阅[压缩与保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-thinking-blocks#conditions-for-kept-thinking-to-stay-valid)。有关平台可用性和完整的请求流程，请参阅[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)。

### 在消息中定义工具（beta）

使用 `inline-tools-2026-09-15` beta 标头时，对话中途系统消息中的 `tool_addition` 块可以携带完整的工具定义，而不仅仅是引用。这样，您可以在对话中途添加工具、更改其 schema，或将服务器工具升级到更新版本，而无需编辑 `tools`，也不会使提示缓存失效。请参阅[在消息中定义工具](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#define-tools-in-a-message-beta)。

### 思考块仅限于生成它们的账户

Claude Sonnet 5.5 生成的思考块仅在生成它们的账户或与其关联的账户中有效。当其他账户发送这些块之一时，API 会在模型接收之前丢弃该块，且请求会成功。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，您可以查看哪些块被丢弃。发送 `thinking-binding-controls-2026-08-01` beta 请求头后，响应会在 `input_transformations` 中列出每个被丢弃的块，并附带 `reason: "organization_binding_mismatch"`。来自早期模型的块不受影响。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。

## 行为差异

即使不更改任何代码，Claude Sonnet 5.5 在以下几个方面也会表现出与 Claude Sonnet 5 的不同。[Claude Sonnet 5.5 提示指南](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5)针对每一项都提供了指导：

* **努力程度已重新校准。** 同一[努力程度](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)级别产生的思考量与在 Claude Sonnet 5 上不同。请重新测试各个努力程度，而不是沿用原有设置。除非您的工作负载属于智能体类型或对延迟敏感，否则请从 `high` 开始。对于智能体编码和多步骤工具使用，明确定义的任务可从 `medium` 开始，更难或更长的任务再调整为 `high`。对于聊天和其他对延迟敏感的工作，请从 `medium` 或 `low` 开始。
* **工具调用之间的文本以思考块返回。** 在工具调用之间，长于一两句话的笔记会以[进度更新 `thinking` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)返回。较短的评论仍为 `text`。在默认的 `display: "omitted"` 下，进度更新块的文本为空，因此将这些笔记流式传输给用户的应用程序在工具调用之间会保持静默，且不会报错。如果您[使用 `between_tools` 关闭前置思考](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#turn-off-up-front-thinking)，文本会返回。要接收这些文本，请参阅迁移指南中的[工具调用之间的文本以思考块返回](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#text-between-tool-calls)。
* **安全防护类别。** 模型的安全防护措施可能会以五种 [`stop_details` 类别](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)拒绝请求。`"cyber"` 表示该请求可能导致网络危害。`"bio"` 表示它可能导致生物危害。`"frontier_llm"` 表示它可能有助于开发竞争性 AI 模型。`"reasoning_extraction"` 表示它要求模型在响应文本中复现其内部推理。`"general_harms"` 表示它属于其他使用政策领域。请参阅[拒绝、回退和计费](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#refusals-fallback-and-billing)。

## 拒绝、回退和计费

[拒绝和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)中的所有内容均适用于 Claude Sonnet 5.5。被拒绝的请求会返回 HTTP 200，带有 `stop_reason: "refusal"` 以及一个指明政策领域的 [`stop_details`](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response) 对象。

请在代码中处理拒绝，并配置回退。使用[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)（`fallbacks: "default"`）时，以 `"cyber"` 或 `"frontier_llm"` 类别被拒绝的请求会在 Claude Sonnet 5 上重新运行。以 `"bio"`、`"reasoning_extraction"` 或 `"general_harms"` 类别被拒绝的请求不会重试。服务器端回退在 Claude API 和 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上处于 beta 阶段。您也可以使用 [SDK 中间件](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)或您自己的重试逻辑。

在产生任何输出之前到达的拒绝，在某些拒绝类别中会计费，在其他类别中则不计费。无论哪种情况，它都会计入您的"rate limit"（速率限制）。请参阅[拒绝的计费方式](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。

## 定价

Claude Sonnet 5.5 的价格与 Claude Sonnet 5 相同，但提示缓存读取除外，其费用为每百万令牌 0.10 美元，是 Claude Sonnet 5 费率的一半。有关完整价目表、数据驻留和工具定价，请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。

## 可用性

Claude Sonnet 5.5 可在以下平台使用：

* **Claude API：** 所有客户，模型名为 `claude-sonnet-5-5`
* **AWS：** [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)，模型名为 `anthropic.claude-sonnet-5-5`；以及 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)，模型名为 `claude-sonnet-5-5`
* **Google Cloud：** [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)，模型名为 `claude-sonnet-5-5`；以及 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)，模型名为 `claude-sonnet-5-5`
* **Microsoft Foundry：** [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)，模型名为 `claude-sonnet-5-5`

## 从 Claude Sonnet 5 迁移

更新您的模型 ID：

<CodeGroup exclude="shell">
  ```python Python
  model = "claude-sonnet-5"  # Before
  model = "claude-sonnet-5-5"  # After
  ```

  ```typescript TypeScript
  let model = "claude-sonnet-5"; // Before
  model = "claude-sonnet-5-5"; // After
  ```

  ```csharp C#
  var model = Model.ClaudeSonnet5; // Before
  model = Model.ClaudeSonnet5_5; // After
  ```

  ```go Go
  model := anthropic.ModelClaudeSonnet5  // Before
  model = anthropic.ModelClaudeSonnet5_5 // After
  ```

  ```java Java
  Model model = Model.CLAUDE_SONNET_5; // Before
  model = Model.CLAUDE_SONNET_5_5; // After
  ```

  ```php PHP
  $model = Model::CLAUDE_SONNET_5; // Before
  $model = Model::CLAUDE_SONNET_5_5; // After
  ```

  ```ruby Ruby
  model = Anthropic::Model::CLAUDE_SONNET_5 # Before
  model = Anthropic::Model::CLAUDE_SONNET_5_5 # After
  ```
</CodeGroup>

然后检查以下六项：

1. 如果您的代码使用 `disabled` 关闭思考，请改为在 `high` 或更低的 effort 下发送 `between_tools`。
2. 将 `tool_choice` 类型 `any` 和 `tool` 替换为 `auto` 加上[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)。
3. 保持对话只追加。如果在编辑较早的历史记录后重放 Claude Sonnet 5.5 的思考块，请求可能会返回 400 错误。请参阅[思考块与模型和对话绑定](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#thinking-blocks-are-tied-to-the-model-that-produced-them)。
4. 如果您通过 `computer_20251124` 使用计算机使用功能，请[迁移到该工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)。在 Amazon Bedrock 上，`computer_20251124` 也可以使用。
5. 如果您将 advisor 工具与 Claude Opus 4.8、Claude Opus 4.7、Claude Sonnet 5 或 Claude Haiku 5.5 顾问一起使用，请[切换到 Claude Sonnet 5.5 接受的顾问](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5#advisor-tool-pairings)。
6. 如果您的界面会显示工具调用之间的文本，请在使用自适应思考时设置 `thinking.display`。使用 `between_tools` 时，无需设置该参数也会返回这些文本。请参阅[工具调用之间的文本在思考块中返回](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide#text-between-tool-calls)。

[Claude Sonnet 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide)提供了从 Claude Sonnet 5 及更早模型迁移的分步说明以及完整的检查清单。

## 后续步骤

<CardGroup cols={3}>
  <Card title="模型概览" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/models/overview">
    所有当前 Claude 模型的完整规格和定价。
  </Card>

  <Card title="迁移指南" icon="code" href="https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide">
    将代码从 Claude Sonnet 5 及更早模型迁移到 Claude Sonnet 5.5。
  </Card>

  <Card title="Claude Sonnet 5.5 提示指南" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5">
    Claude Sonnet 5.5 特有的行为差异和提示模式。
  </Card>

  <Card title="Effort" icon="gauge" href="https://platform.claude.com/docs/zh-CN/build-with-claude/effort">
    控制 Claude 在响应时使用的令牌数量，从 low 到 max。
  </Card>

  <Card title="思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    自适应思考的工作原理以及思考块的保留方式。
  </Card>

  <Card title="拒绝与回退" icon="shield" href="https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback">
    处理 `stop_reason: "refusal"` 并在其他模型上重试。
  </Card>
</CardGroup>
