---
title: Claude Opus 5.5 的新功能
url: https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5
description: Claude Opus 5.5 中的破坏性变更、功能支持和行为差异概述。
---

Claude Opus 5.5 专为长时间运行的智能体编码和知识工作而构建，定价为每百万输入/输出令牌 $4 / $20 美元。有四项破坏性变更可能会影响已在 Claude Opus 5 上运行的代码：

* [无法禁用思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#thinking-cant-be-disabled)。
* [强制工具使用会返回错误](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#forced-tool-use-is-not-supported)。
* [思考块与模型和对话绑定](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#thinking-blocks-are-tied-to-the-model-that-produced-them)。
* [不支持某些计算机使用工具版本](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#computer-20251124-is-not-supported)。

前三项同样适用于 Claude Fable 5.1。另有一项变更会改变响应结构，但不会导致任何请求失败：[工具调用之间的文本以 `thinking` 块返回](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#text-between-tool-calls)，并且在默认 `display` 设置下这些块的文本为空。如果应用程序将该文本作为进度更新流式传输给用户，那么在工具调用之间它将不再输出任何内容，直到设置一个会返回该文本的 `display` 值。

## 新模型

| 模型              | Claude API ID   | 描述                  |
| --------------- | --------------- | ------------------- |
| Claude Opus 5.5 | claude-opus-5-5 | 适用于长时间运行的智能体编码和知识工作 |

[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)始终开启，[effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)控制思考深度；该模型上的默认值为 `medium`。有关上下文窗口、输出限制、知识截止日期和价格，请参阅 [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview) 模型页面；有关所有当前模型，请参阅[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)。

## 破坏性变更

### 无法禁用思考

在 Claude Opus 5 上，思考默认开启，并且在 effort 为 `high` 或更低时接受 `thinking: {"type": "disabled"}`。在 Claude Opus 5.5 上，思考始终开启：设置了 `thinking: {"type": "disabled"}` 的请求，或使用 `thinking: {"type": "enabled", "budget_tokens": N}` 手动设置预算的请求，都会返回 400 `invalid_request_error`。请省略 `thinking` 字段，或发送与之等效的 `thinking: {"type": "adaptive"}`。

错误消息如下：

```text wrap
"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

```text wrap
"thinking.type.enabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

[effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)用于控制思考深度、延迟和成本：在您以前禁用思考的地方降低该参数；[针对成本和智能进行优化](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#tune-effort)提供了用于选择级别的实测结果。由于每个响应都可能以一个或多个 `thinking` 块开头（在默认 `display: "omitted"` 下返回时 `thinking` 字段为空），请按 `type` 字段而非位置来选择内容块，并在工具使用循环中原样回传 `thinking` 块。已在开启思考的 Claude Opus 5 上运行的代码无需更改。请参阅[思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)以及迁移指南中的[思考变更前后对比](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-cant-be-disabled)。

### 不支持强制工具使用

Claude Opus 5.5 不支持"forced tool use"（强制工具使用）。将 `tool_choice` 设置为 `{"type": "any"}` 或 `{"type": "tool", "name": "..."}` 会返回 400 `invalid_request_error`：

```text wrap
tool_choice: type "tool" and "any" are not supported for this model.
```

支持 `tool_choice: {"type": "auto"}`（默认值）和 `{"type": "none"}`，并且相同的验证也适用于[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)端点。要获得符合 schema 的 JSON，请保持 `tool_choice: {"type": "auto"}` 并通过[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)设置 `strict: true`，或将 schema 移至[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。要让模型调用工具而不是以文本回复，请在提示中说明何时适用该工具。迁移指南展示了[工具选择的变更前后对比](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#forced-tool-use)。

### 思考块与模型和对话绑定

每个 thinking block（思考块）都会记录生成它的模型。切换模型的对话只有在切换后的模型能够读取先前模型的思考块时，才会保留其先前的推理内容：

| 切换                                                                                      | 是否保留推理内容                                                         |
| --------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 从 Claude Opus 5 或更早版本、Claude Sonnet 5 或更早版本，或 Claude Haiku 4.5 或更早版本切换到 Claude Opus 5.5 | 是                                                                |
| 从 Claude Sonnet 5.5 或 Claude Haiku 5.5 切换到 Claude Opus 5.5                              | 是，仅限 Claude API 和 Claude Models on Google Cloud (Google managed) |
| 从 Claude Fable 或 Claude Mythos 模型切换到 Claude Opus 5.5                                    | 否                                                                |
| 从 Claude Opus 5.5 切换到 Claude Fable 5.2 或 Claude Mythos 5.2                              | 是，仅限 Claude API 和 Claude Models on Google Cloud (Google managed) |
| 从 Claude Opus 5.5 切换到 Claude Fable 5.1 或 Claude Mythos 5.1                              | 是，仅限 Claude API                                                  |
| 从 Claude Opus 5.5 切换到任何其他模型                                                             | 否                                                                |

当请求中携带目标模型无法读取的思考块时，API 会在模型看到之前将其丢弃。请求会成功，且被丢弃的思考块不会计费。使用 `thinking-binding-controls-2026-08-01` beta 标头时，丢弃情况会在顶层的 `input_transformations` 数组中报告。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。对于不涉及 Claude Opus 5.5 的切换，请参阅[在 Claude API 上哪些模型切换会保留推理](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#which-model-switches-keep-reasoning)。

API 还会检查 Claude Opus 5.5 思考块之前的任何内容（`system` 提示、`tools` 或更早的消息）自该思考块生成以来是否发生了变化。请保持对话仅追加：使用[对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)来更改指令或工具，而不是进行编辑。

在发生此类更改后重放思考块且未设置 `thinking.block_binding.prefix_mismatch_behavior` 的请求会收到 400 错误。这适用于在 2026 年 8 月 31 日 00:00 UTC 或之后创建的账户。[保留的思考的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#feature-availability)列出了适用此规则的平台。对于在该日期之前创建的账户，请求可能会也可能不会收到该错误。若要改为丢弃受影响的思考块，请发送 `thinking-binding-controls-2026-08-01` beta 标头并将该字段设置为 `"drop_block"`。请参阅[保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)以及迁移指南中的[关于思考块的说明](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-blocks)。

### 不支持某些计算机使用工具版本

Claude Opus 5.5 通过 `computer_toolset_20260801` 工具集支持[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)。在 Amazon Bedrock 上，Claude Opus 5.5 还接受带有 `computer-use-2025-11-24` beta 标头的 `computer_20251124` 工具。对于任何不接受的计算机使用工具版本（例如 Claude API 上的 `computer_20251124`），Claude Opus 5.5 都会返回 400 错误。Claude API 的 `invalid_request_error` 消息会指明被拒绝的类型，并在 `Did you mean one of` 之后列出模型接受的工具类型。该消息的开头如下：

```text wrap
'claude-opus-5-5' does not support tool types: computer_20251124.
```

使用该工具集的集成无需任何更改。要将 `computer_20251124` 集成迁移到工具集，请按照[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)进行操作。移除 beta 标头，并将 `tools` 条目替换为 `{"type": "computer_toolset_20260801"}`。然后更新您的智能体循环，以处理成员 `tool_use` 块、批量操作以及结果中的 `toolset_name`。迁移指南展示了[计算机使用的变更前后对比](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#computer-use-toolset)。

## 功能支持

Claude Opus 5.5 支持[按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)（处于 beta 阶段）、[对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)、[任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)、最小可缓存提示为 512 个令牌的[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)、[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)、[Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)、[PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)、[视觉](https://platform.claude.com/docs/zh-CN/build-with-claude/vision)，以及服务器端和客户端工具（请参阅[使用 Claude 进行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)）。有关模型可用性，请参阅各功能的页面。

### 快速模式

[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)可在 Claude API、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 以及托管在 Azure 上的部署的 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上用于 Claude Opus 5.5。在 Claude API 和 Claude Models on Google Cloud (Anthropic managed) 上，请在请求中设置 `speed: "fast"`。有关支持的模型、速率限制和定价，请参阅[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。

### 在消息中定义工具（beta）

使用 `inline-tools-2026-09-15` beta 标头时，对话中途系统消息中的 `tool_addition` 块可以携带完整的工具定义而非引用，因此您可以在对话中途添加工具、更改其 schema，或将服务器工具升级到更新版本，而无需编辑 `tools`，也不会丢失提示缓存。这适用于所有支持在对话中途更改工具的模型，包括 Claude Opus 5.5。请参阅[在消息中定义工具](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#define-tools-in-a-message-beta)。

### 按需压缩（beta）

使用 `compact-2026-09-04` beta 标头时，发送顶层 `compaction` 参数的请求会返回一个经过签名的 `compaction` 块，用于总结整个对话，随后您将其放在最前面发送，以替代被总结的消息。此功能适用于支持压缩的模型，包括 Claude Opus 5.5。您可以自行选择何时压缩，并且请求可以在后台运行。在[压缩与保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-thinking-blocks#conditions-for-kept-thinking-to-stay-valid)中所述的条件下，您保留的轮次中的思考块在替换后可以保持有效。这一点对 Claude Opus 5.5 很重要，因为它的[思考块与对话绑定](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#thinking-blocks-are-tied-to-the-model-that-produced-them)。有关平台可用性和完整的请求流程，请参阅[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)。

## 行为差异

Claude Opus 5.5 与 Claude Opus 5 在若干方面存在差异，这些差异无需任何代码更改即会显现。每项差异在[为 Claude Opus 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5)中都有相应指导：

* **默认 effort 为 `medium`。** 省略 `effort` 的请求以 `medium` 运行；在 Claude Opus 5 上则以 `high` 运行。请显式设置 `effort` 并重新运行您的参数扫描；请参阅[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#calibrate-effort)。
* **在给定 effort 级别下每轮思考更多。** 在相同的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 设置下，该模型每轮的思考往往比 Claude Opus 5 更多，在 `xhigh` 和 `max` 下尤为明显。请重新运行 effort 扫描，而不是沿用原有设置，并在 `max_tokens` 中为思考预留空间。请参阅[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#calibrate-effort)。
* **工具调用之间的文本以思考块形式返回。** 模型在工具调用之间写下的简短说明会以[进度更新 `thinking` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)而非 `text` 块的形式到达，因此在默认 `display: "omitted"` 下，将其流式传输给用户的应用程序在工具调用之间会保持静默，且不会出现错误。迁移指南说明了[如何接收工具调用之间的文本](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#text-between-tool-calls)，[面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#user-facing-progress-updates)介绍了如何请求更多此类更新。
* **更多安全防护类别。** 除网络安全分类器外，该模型还运行生物安全分类器。与 Claude Opus 5 一样，促使其在响应文本中复现内部推理的请求可能会以 `reasoning_extraction` 类别被拒绝。请参阅 [Claude Opus 5.5 中的拒绝和回退](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#refusals-and-fallback)以及[安全防护拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#safeguard-refusals)。
* **对图表、示意图和屏幕截图的解读更加精准。** 该模型无需工具即可更精确地从密集图表和依赖布局的视觉内容中读取数值，因此为早期模型构建的提示端视觉变通方法可能不再需要；对于最密集的输入，图像工具仍能提升准确性。请参阅[用于复杂视觉输入的工具](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#tools-for-complex-visual-inputs)。

如果您的 Claude Opus 5 集成在禁用思考的情况下运行，请结合破坏性变更[无法禁用思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#thinking-cant-be-disabled)参阅[为禁用思考编写的提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#prompts-written-for-thinking-disabled)。有关在智能体编码和代码审查、知识工作、沟通、视觉输入以及计算机使用方面的能力提升，请参阅[与提示相关的能力](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#capability-improvements)。

## 拒绝与回退

Claude Opus 5.5 附带安全分类器，[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)中的所有内容均适用。被拒绝的请求会返回 HTTP 200，其中包含 `stop_reason: "refusal"` 以及一个指明政策领域的 [`stop_details`](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response) 对象，因此请处理拒绝并配置回退：通过[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)（`fallbacks: "default"`，处于 beta 阶段，会在 Anthropic 针对该类别推荐的模型上重试）、[SDK 中间件](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)或您自己的重试逻辑在另一个模型上重试。在产生任何输出之前发生的拒绝是否计费取决于其拒绝类别，并且无论哪种情况都会计入您的速率限制；请参阅[拒绝如何计费](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。

## 定价

Claude Opus 5.5 的价格为每百万输入令牌 $4 美元、每百万输出令牌 $20 美元，低于 Claude Opus 5 的 $5 和 $25 美元；5 分钟缓存写入为每百万令牌 $5 美元，1 小时缓存写入为 $8 美元，缓存读取为 $0.20 美元（基础输入价格的 0.05 倍）。[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)为半价：$2 和 $10 美元。有关数据驻留和工具定价，请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。

## 可用性

Claude Opus 5.5 可在以下平台使用：

* **Claude API：** 所有客户，模型 ID 为 `claude-opus-5-5`
* **AWS：** [Amazon Bedrock 中的 Claude（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)，模型 ID 为 `anthropic.claude-opus-5-5`；[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)，模型 ID 为 `claude-opus-5-5`
* **Google Cloud：** [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)，模型 ID 为 `claude-opus-5-5`；[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)，模型 ID 为 `claude-opus-5-5`
* **Microsoft Foundry：** [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)，模型 ID 为 `claude-opus-5-5`

## 从 Claude Opus 5 迁移

更新您的模型 ID：

<CodeGroup exclude="shell">
  ```python Python
  model = "claude-opus-5"  # Before
  model = "claude-opus-5-5"  # After
  ```

  ```typescript TypeScript
  let model = "claude-opus-5"; // Before
  model = "claude-opus-5-5"; // After
  ```

  ```csharp C#
  var model = Model.ClaudeOpus5; // Before
  model = Model.ClaudeOpus5_5; // After
  ```

  ```go Go
  model := anthropic.ModelClaudeOpus5  // Before
  model = anthropic.ModelClaudeOpus5_5 // After
  ```

  ```java Java
  Model model = Model.CLAUDE_OPUS_5; // Before
  model = Model.CLAUDE_OPUS_5_5; // After
  ```

  ```php PHP
  $model = Model::CLAUDE_OPUS_5; // Before
  $model = Model::CLAUDE_OPUS_5_5; // After
  ```

  ```ruby Ruby
  model = Anthropic::Model::CLAUDE_OPUS_5 # Before
  model = Anthropic::Model::CLAUDE_OPUS_5_5 # After
  ```
</CodeGroup>

然后移除所有 `thinking: {"type": "disabled"}` 或 `thinking: {"type": "enabled", ...}` 设置，改为选择一个 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。将 `tool_choice` 类型 `any` 和 `tool` 替换为 `auto` 加上[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)。如果您通过 `computer_20251124` 使用计算机使用功能，请[迁移到工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)。在 Amazon Bedrock 上，`computer_20251124` 也可以使用。如果您的界面会显示工具调用之间的文本，还需设置 `thinking.display`；请参阅[工具调用之间的文本以思考块返回](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#text-between-tool-calls)。有关从 Claude Opus 5 及更早模型迁移的分步说明以及完整检查清单，请参阅[Claude Opus 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide)。

## 后续步骤

<CardGroup cols={3}>
  <Card title="模型概览" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/models/overview">
    所有当前 Claude 模型的完整规格和定价。
  </Card>

  <Card title="迁移指南" icon="code" href="https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide">
    将代码从 Claude Opus 5 及更早模型迁移到 Claude Opus 5.5。
  </Card>

  <Card title="为 Claude Opus 5.5 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5">
    Claude Opus 5.5 特有的行为差异和提示模式。
  </Card>

  <Card title="Effort" icon="gauge" href="https://platform.claude.com/docs/zh-CN/build-with-claude/effort">
    控制 Claude 在响应时使用的令牌数量，从 low 到 max。
  </Card>

  <Card title="思考" icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
    自适应思考的工作原理以及思考块的保留方式。
  </Card>

  <Card title="拒绝与回退" icon="shield" href="https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback">
    处理 `stop_reason: "refusal"` 并在另一个模型上重试。
  </Card>
</CardGroup>
