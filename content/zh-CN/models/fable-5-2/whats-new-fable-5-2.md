---
title: Claude Fable 5.2 的新功能
url: https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2
description: 与 Claude Fable 5.1 和 Claude Mythos 5.1 相比，Claude Fable 5.2 和 Claude Mythos 5.2 的破坏性变更、功能支持、定价和可用性。
---

Claude Fable 5.2 专为高要求的推理和长周期智能体工作而打造。对于大多数工作负载，请从最新的 Claude Opus 模型开始（请参阅[选择合适的模型](https://platform.claude.com/docs/zh-CN/about-claude/models/choosing-a-model)）；当您在更高 effort 下对 Claude Opus 模型进行的评估仍无法满足要求时，再转向 Claude Fable 5.2。

相比 Claude Fable 5.1，有四项变更可能会破坏您的代码：

* [前缀检查适用于所有账户](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#editing-earlier-turns-invalidates-thinking-blocks)。
* [只有 Claude Fable 5.2 和 Claude Mythos 5.2 能读取 Claude Fable 5.2 的思考块](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#thinking-blocks-are-tied-to-the-model-that-produced-them)。
* 早期的 [`computer_20251124` 计算机使用工具会返回错误](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#computer-20251124-is-not-supported)（Amazon Bedrock 以外的所有平台）。
* 使用 Advisor 工具时，Claude Fable 5.2 执行器[需要 Claude Fable 5.2 或 Claude Mythos 5.2 作为顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#advisor-tool-pairs)。

## 模型

| 模型                | Claude API ID     | 描述                      | 可用性                                                                                                      |
| ----------------- | ----------------- | ----------------------- | -------------------------------------------------------------------------------------------------------- |
| Claude Fable 5.2  | claude-fable-5-2  | 适用于高要求的推理和长周期智能体工作      | Claude API 上的所有客户（合作伙伴平台请参阅各平台页面）                                                                        |
| Claude Mythos 5.2 | claude-mythos-5-2 | 与 Claude Fable 5.2 能力相同 | 通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织 |

Claude Fable 5.2 和 Claude Mythos 5.2 共享以下规格：

* **上下文窗口和输出：** [1M 令牌的 “context window”（上下文窗口）](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)（默认值和最大值），以及 128k 最大输出令牌。
* **思考：** [“adaptive thinking”（自适应思考）](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)始终开启。使用 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)控制思考深度。
* **分词器：** Claude Opus 4.7 引入的分词器。请参阅[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting#token-counts-on-claude-fable-5)。

有关所有当前模型，请参阅[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)。

## 破坏性变更

### 编辑较早的轮次会使所有账户上的思考块失效

API 的前缀检查（请参阅[保持前缀不变](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)）会检测 Claude Fable 5.2 或 Claude Mythos 5.2 思考块之前的任何内容自模型生成该块以来是否发生了变化：包括 `system` 提示、`tools` 或较早的消息。在 Claude Fable 5.2 和 Claude Mythos 5.2 上，发回无效思考块且未设置 `prefix_mismatch_behavior` 的请求会收到 400 错误，与账户创建时间无关（请参阅[未设置 `prefix_mismatch_behavior` 时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)）。在 Claude Fable 5.1 上，此类请求会在 2026 年 8 月 31 日或之后创建的账户上收到该错误。在 Claude API 上，Claude Mythos 5.1 已在所有账户上返回该错误。因此，在 Claude Fable 5.1 上编辑历史记录的代码，在较旧的账户上切换之前可以无错误运行，切换之后则会失败。

检查失败时，请求会返回 400 错误，其消息为 `The block is bound to a different conversation`。若要改为丢弃受影响的块并继续，请发送 `thinking-binding-controls-2026-08-01` beta 标头，并将 `thinking.block_binding.prefix_mismatch_behavior` 设置为 `"drop_block"`。响应随后会在顶层的 `input_transformations` 数组中列出每个被丢弃的块。请将 `"drop_block"` 视为修复编辑期间的临时措施，因为[丢弃块可能会增加输出令牌](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#drop-block-cost)。

如果您的请求由 Claude Code、claude.ai、[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 或 [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) 构建，则对您没有任何变化。

如果您的请求由您自己的代码构建，只要以下所有条件都成立，对您也没有任何变化：

* 您的代码在整个对话期间保持 `system` 和 `tools` 不变。
* 您的代码只向 `messages` 追加内容，从不更改、重新排序或删除较早的消息。
* 您的代码[完全按照 API 返回的原样发回每个助手轮次](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#append-assistant-turns-exactly-as-returned)：包含每个块，且顺序相同。
* 您的代码不执行[在您的集成中需要检查的内容](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#what-to-check)中列出的任何操作。

否则，请在切换前测试您的代码：

* **Claude Fable 5.1 上的代码：** 按照迁移指南的[前缀检查部分](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-preserved-thinking)操作。
* **Claude Mythos 5.1 上的代码：** 按照[从 Claude Mythos 5.1 迁移到 Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-mythos-5-1-to-claude-mythos-5-2)操作。

[保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)解释了哪些操作算作编辑，还展示了如何在不编辑的情况下完成每项更改。

### 只有 Claude Fable 5.2 和 Claude Mythos 5.2 能读取 Claude Fable 5.2 的思考块

每个思考块都会记录生成它的模型，而一个模型只能读取它自己的块以及部分其他模型的块。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 和 Claude Mythos 5.2 可以读取彼此的思考块，以及来自 Claude Opus 5.5、Claude Sonnet 5.5、Claude Haiku 5.5、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5、Claude Fable 5 和 Claude Mythos 5，以及更早的 Opus、Sonnet 和 Haiku 模型的思考块。没有任何其他模型能读取它们的块。

* **从上述模型之一切换到 Claude Fable 5.2 或 Claude Mythos 5.2** 时，对话的推理会保留在 Claude API 和 Claude Models on Google Cloud (Google managed) 上。在 Claude Fable 5.2 和 Claude Mythos 5.2 之间切换也是如此。
* **从 Claude Fable 5.2 或 Claude Mythos 5.2 切换到任何其他模型**，会在该模型运行的轮次中丢失它们的推理内容。这包括 Claude Opus 5.5、Claude Fable 5.1、Claude Mythos 5.1，以及 Claude Fable 5.2 的回退模型（Claude Opus 4.8 和 Claude Opus 5）。

当请求携带当前模型无法读取的块时，API 会在模型接收之前丢弃该块。请求会成功，被丢弃的块不计费。使用 `thinking-binding-controls-2026-08-01` beta 标头时，响应会在 `input_transformations` 中列出该丢弃。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。

### 使用计算机使用工具集代替 `computer_20251124`

Claude Fable 5.1 既接受以 `computer_toolset_20260801` 工具集形式提供的[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)，也接受在使用 `computer-use-2025-11-24` beta 标头时以早期 `computer_20251124` 工具形式提供的计算机使用。Claude Fable 5.2 和 Claude Mythos 5.2 通过工具集支持计算机使用。向这两个模型中任一模型发送声明了 `computer_20251124` 工具的请求都会返回 400 `invalid_request_error`（Amazon Bedrock 除外）。在 Amazon Bedrock 上，两个模型都接受早期工具：使用 InvokeModel API 时，请在请求正文的 `anthropic_beta` 数组中发送 `computer-use-2025-11-24`。

要迁移现有集成，请按照[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)操作。

### Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 作为顾问

使用处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，顾问的能力必须至少与执行器相当。Claude Fable 5.2 或 Claude Mythos 5.2 执行器只接受 Claude Fable 5.2 或 Claude Mythos 5.2 作为其顾问。任何其他顾问（包括 Claude Fable 5.1 和 Claude Mythos 5.1）都会返回 400 `invalid_request_error`。

如果您使用 Advisor 工具并将执行器迁移到这些模型之一，请在同一次变更中将 Advisor 工具定义中的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。更改顾问的 `model` 属于对 `tools` 的编辑，因此请在新对话中进行此更改。变更后发回较早思考块的请求将无法通过[前缀检查](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#editing-earlier-turns-invalidates-thinking-blocks)并返回 400 错误，除非您将 `thinking.block_binding.prefix_mismatch_behavior` 设置为 `"drop_block"`（beta，`thinking-binding-controls-2026-08-01` 标头）。若要保留当前执行器而只更改顾问，请将顾问的 `model` 设置为 `claude-fable-5-2`：所有执行器模型都接受它。Claude Fable 5.2 顾问会以加密形式返回建议，因此如果您的代码需要读取建议文本，请参阅[结果变体](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#result-variants)。另请参阅[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#model-compatibility)。

## 思考和工具选择

有两条 Messages API 规则适用于每个 Claude Fable 5.2 请求。如果您来自 Claude Fable 5.1、Claude Mythos 5.1 或 Claude Opus 5.5，两者都不算变更。如果来自自适应思考已始终开启的 Claude Fable 5 或 Claude Mythos 5，则只有强制工具使用规则是破坏性变更。如果来自 Claude Opus 5 或更早的 Claude Opus 模型，两者都是破坏性变更。[Claude Fable 5.2 和 Claude Mythos 5.2 迁移指南](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide)涵盖了每种迁移路径。

* **自适应思考始终开启。** 带 `budget_tokens` 的 `thinking: {"type": "enabled"}` 和 `thinking: {"type": "disabled"}` 都会返回 400 错误。请省略 `thinking` 或发送 `{"type": "adaptive"}`，并使用 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 控制思考深度。
* **不支持强制工具使用。** 将 `tool_choice` 设置为 `{"type": "any"}` 或 `{"type": "tool", "name": "..."}` 会返回 400 `invalid_request_error`。若要获得符合 schema 的 JSON，请保持 `tool_choice: {"type": "auto"}` 并通过[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)设置 `strict: true`，或使用 [JSON 输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#usage)。在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)不适用于 Claude Fable 5.2 或 Claude Mythos 5.2。这包括严格工具使用和 JSON 输出。在这种情况下，请省略 `strict` 并在代码中验证工具输入。

## 功能支持

Claude Fable 5.2 支持以下 API 功能（不限于此）：

* **Effort：** 全部五个 [effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-levels)：`low`、`medium`、`high`、`xhigh` 和 `max`。API 默认值为 `high`。请从默认值开始，因为大部分能力提升在 `high` 时就已具备。对于最困难的长时间运行的编码和智能体工作，在您已测量到收益的情况下，可升级到 `xhigh`。`max` 仅用于测量模型的能力上限，以及时间和令牌都无关紧要的少数任务。请参阅[Claude Fable 5.2 的推荐 effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)。
* **按需压缩（beta）：** 使用 `compact-2026-09-04` beta 标头，您可以选择何时用摘要替换较早的轮次。请参阅[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)。
* **对话中途的工具变更（beta）：** 您可以在 `role: "system"` 消息中添加或移除工具，而无需编辑 `tools`。在 Claude API、Claude Platform on AWS 和 Claude Models on Google Cloud (Anthropic managed) 上，请发送 `inline-tools-2026-09-15` beta 标头。`mid-conversation-tool-changes-2026-07-01` 标头适用于按引用命名工具的变更，适用平台见[对话中途的系统消息和工具变更](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)。请参阅[对话中途的工具变更](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#mid-conversation-tool-changes)。
* **计算机使用：** 通过 `computer_toolset_20260801` 工具集。早期的 `computer_20251124` 工具会返回 400 错误，但 Amazon Bedrock 除外。在该平台上，Claude Fable 5.2 和 Claude Mythos 5.2 也接受早期工具。请参阅[计算机使用破坏性变更](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#computer-20251124-is-not-supported)和[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)。

按需压缩和对话中途的工具变更可帮助您保持历史记录仅追加，这正是[前缀检查](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#editing-earlier-turns-invalidates-thinking-blocks)所要求的。有关模型和平台可用性，请参阅各功能的页面。

[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)可用于 Claude API 上 Messages API 请求中的 Claude Fable 5.2。

快速模式还可用于 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上的 Claude Fable 5.2。

快速模式还可用于 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 上的 Claude Fable 5.2。

快速模式还可用于 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上托管于 Anthropic 的部署中的 Claude Fable 5.2，作为单独的模型 `claude-fable-5-2-fast` 提供。

快速模式还可用于 Claude API 上 Messages API 请求中的 Claude Mythos 5.2。[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出了 Claude Mythos 5.2 支持快速模式的所有平台。

### 思考块仅限于生成它们的账户

Claude Fable 5.2 和 Claude Mythos 5.2 生成的思考块仅在生成它们的账户或与该账户关联的账户中有效。当其他账户发送此类块时，API 会在模型接收之前丢弃该块，并且请求会成功。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，使用 `thinking-binding-controls-2026-08-01` beta 标头时，响应会在 `input_transformations` 中列出每个被丢弃的块，并附带 `reason: "organization_binding_mismatch"`。请参阅"保留思考"中的[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。

## 行为差异

Claude Fable 5.2 的一些行为无需任何代码更改就会显现。迁移提示时请检查这些行为。[为 Claude Fable 5.2 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2)针对每一项都提供了指导：

* **要求减少思考的提示语句无法阻止模型在 `medium` 及以上 effort 时进行思考。** Claude Fable 5.1 通常会遵循"直接回答，不要深思"之类的语句。在 Anthropic 的测试中，此类语句并未阻止 Claude Fable 5.2 在 `medium` 及以上时进行思考。在 `high` 及以上时，模型几乎在每次回复前都会思考。如果简单请求耗时过长或成本过高，请为这些轮次使用较低的 effort 级别。请参阅[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#calibrate-effort)。
* **轮次可能在工作完成之前结束。** 在长时间的多步骤任务中，模型有时会以剩余工作列表、继续执行的提议，或在遇到普通障碍时停止来结束一个轮次。这种情况在 `low` 和 `medium` effort 下更常见。请参阅[完成整个任务](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#finish-the-whole-task)。
* **提示中关于代码所有权的说明很重要。** 在 Anthropic 针对小型编码任务的测试中，当提示未说明代码归谁所有时，模型倾向于在现有结构上添加内容，并报告附近的问题而不是修复它们。请参阅[说明代码归谁所有](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#say-who-owns-the-code)。
* **长串工具调用可能几乎不产生面向用户的文本。** 模型也可能在未说明将要做什么的情况下进行第一次工具调用。请参阅[在长轮次中让用户了解进展](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#keep-users-informed-during-long-turns)。
* **模型向子智能体委派的程度取决于您的框架和任务。** 在提示鼓励委派的智能体编码产品中，模型会积极委派。子智能体可能占会话成本的很大一部分。请参阅[为子代理设定预期](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#set-expectations-for-subagents)。
* **模型会严格遵循风格规则。** 对句子长度的硬性限制可能会使模型的文字显得生硬断续。请参阅[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)。

## 拒绝、回退和计费

Claude Fable 5.2 运行安全分类器，[拒绝和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)中的所有内容均适用。

* **拒绝：** 被拒绝的请求会返回 HTTP 200，带有 `stop_reason: "refusal"` 以及一个指明所触发策略领域的 [`stop_details`](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response) 对象。
* **回退：** 您可以使用[服务端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)、[SDK 中间件](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)或您自己的重试逻辑，在其他模型上重试被拒绝的请求。服务端回退在 Claude API 和 Claude Models on Google Cloud (Anthropic managed) 上处于 beta 阶段。使用服务端回退时，`fallbacks: "default"` 会在 Anthropic 为该类别推荐的模型上重试被拒绝的请求。对于没有推荐回退模型的类别（例如 `reasoning_extraction`），拒绝结果维持不变。Claude Fable 5.2 允许的回退目标是 Claude Opus 4.8 和 Claude Opus 5。
* **计费：** 在产生任何输出之前到达的拒绝是否计费取决于其拒绝类别，并且无论如何都会计入您的速率限制。请参阅[拒绝如何计费](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。[回退抵扣](https://platform.claude.com/docs/zh-CN/build-with-claude/fallback-credit)会退还切换模型产生的提示缓存成本。

## 定价

Claude Fable 5.2 和 Claude Mythos 5.2 的价格相同（以美元计）：

| 基础输入      | 5 分钟缓存写入   | 1 小时缓存写入   | 缓存读取         | 输出         |
| --------- | ---------- | ---------- | ------------ | ---------- |
| $8 / MTok | $10 / MTok | $16 / MTok | $0.20 / MTok | $40 / MTok |

输入和输出的成本低于 Claude Fable 5.1 和 Claude Mythos 5.1。Claude Fable 5.1 和 Claude Mythos 5.1 的价格为每百万输入令牌 10 美元、每百万输出令牌 50 美元。缓存读取（命中和刷新）的成本为基础输入价格的 0.025 倍，与这些模型的倍数相同。[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)为每百万输入令牌 4 美元、每百万输出令牌 20 美元。Claude Fable 5.2 和 Claude Mythos 5.2 的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)为每百万输入令牌 16 美元、每百万输出令牌 80 美元。请参阅[快速模式定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)。有关数据驻留和工具定价，请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。

## 可用性

Claude Fable 5.2 可在以下平台使用：

* **Claude API：** 所有客户，ID 为 `claude-fable-5-2`
* **AWS：** [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)（ID 为 `claude-fable-5-2`）
* **Amazon Bedrock：** 仅在 [`bedrock-runtime` 端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#api-model-ids)上为 `global.anthropic.claude-fable-5-2` 或 `us.anthropic.claude-fable-5-2`
* **Google Cloud：** [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)（ID 为 `claude-fable-5-2`）和 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)（ID 为 `claude-fable-5-2`）
* **Microsoft Foundry：** [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)（ID 为 `claude-fable-5-2`），运行在 Anthropic 基础设施上（请参阅[托管选项](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry#hosting-options)）

Claude Mythos 5.2 仅向通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织提供。它可在以下平台使用：

* **Claude API：** ID 为 `claude-mythos-5-2`
* **Amazon Bedrock：** 仅在 [`bedrock-runtime` 端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#api-model-ids)上为 `global.anthropic.claude-mythos-5-2` 或 `us.anthropic.claude-mythos-5-2`
* **Google Cloud：** [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)（ID 为 `claude-mythos-5-2`）和 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)（ID 为 `claude-mythos-5-2`）
* **Microsoft Foundry：** [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)（ID 为 `claude-mythos-5-2`），运行在 Anthropic 基础设施上

如需申请访问权限，请申请适用于您用例的计划，或联系您的客户团队。

Claude Fable 5.2 和 Claude Mythos 5.2 要求 30 天数据保留，除非获得 Anthropic 明确授权，否则不支持零数据保留。两者均为[受覆盖模型](https://support.claude.com/en/articles/15425695)。请参阅[特定模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。

## 从 Claude Fable 5.1 迁移

要从 Claude Fable 5.1 迁移，请更新您的模型 ID：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    将请求正文中的 `model` 字段设置为 `claude-fable-5-2`。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    将 `--model` 标志设置为 `claude-fable-5-2`。
  </CodeGroupItem>

  ```python Python
  model = "claude-fable-5-1"  # Before
  model = "claude-fable-5-2"  # After
  ```

  ```typescript TypeScript
  let model = "claude-fable-5-1"; // Before
  model = "claude-fable-5-2"; // After
  ```

  ```csharp C#
  var model = Model.ClaudeFable5_1; // Before
  model = Model.ClaudeFable5_2; // After
  ```

  ```go Go
  model := anthropic.ModelClaudeFable5_1 // Before
  model = anthropic.ModelClaudeFable5_2  // After
  ```

  ```java Java
  Model model = Model.CLAUDE_FABLE_5_1; // Before
  model = Model.CLAUDE_FABLE_5_2; // After
  ```

  ```php PHP
  $model = Model::CLAUDE_FABLE_5_1; // Before
  $model = Model::CLAUDE_FABLE_5_2; // After
  ```

  ```ruby Ruby
  model = Anthropic::Model::CLAUDE_FABLE_5_1 # Before
  model = Anthropic::Model::CLAUDE_FABLE_5_2 # After
  ```
</CodeGroup>

然后检查以下事项：

1. **检查您的代码是否编辑历史记录。** 如果您的代码自行构建 `messages` 数组，请运行迁移指南[前缀检查部分](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-preserved-thinking)中的测试，即使 Claude Fable 5.1 从未对您的账户返回过该错误。
2. **检查模型切换。** 如果路由器或回退机制会在模型之间移动对话，那么除 Claude Mythos 5.2 之外的所有模型（包括 Claude Fable 5.1 和 Claude Opus 5.5）都会丢弃 Claude Fable 5.2 的思考块。请参阅[只有 Claude Fable 5.2 和 Claude Mythos 5.2 能读取 Claude Fable 5.2 的思考块](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#thinking-blocks-are-tied-to-the-model-that-produced-them)。
3. **将计算机使用迁移到工具集。** 如果您在 Amazon Bedrock 以外的任何平台上使用 `computer_20251124` 工具，Claude Fable 5.2 会对该工具返回 400 错误。请改为声明 `computer_toolset_20260801`。有关具体更改，请参阅[使用计算机使用工具集代替 `computer_20251124`](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#computer-20251124-is-not-supported)。
4. **更新顾问。** 如果您在以 Claude Fable 5.2 为执行器的情况下使用 Advisor 工具，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。请参阅[Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 作为顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#advisor-tool-pairs)。
5. **重新运行您的评估。** 包括一次从默认值 `high` 开始的 effort 扫描。请参阅[Claude Fable 5.2 的推荐 effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)。

有关分步说明（包括从 Claude Mythos 5.1 和 Claude Opus 模型迁移），请参阅[Claude Fable 5.2 和 Claude Mythos 5.2 迁移指南](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide)。有关特定于模型的提示模式，请参阅[为 Claude Fable 5.2 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2)。

## 后续步骤

<CardGroup cols={3}>
  <Card title="模型概览" icon="arrow-right" href="https://platform.claude.com/docs/zh-CN/models/overview">
    所有当前 Claude 模型的规格和定价。
  </Card>

  <Card title="迁移指南" icon="code" href="https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide">
    将代码从 Claude Fable 5.1、Claude Mythos 5.1 和 Claude Opus 模型迁移到 Claude Fable 5.2 或 Claude Mythos 5.2。
  </Card>

  <Card title="Claude Fable 5.2 提示指南" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2">
    特定于 Claude Fable 5.2 的提示模式。
  </Card>
</CardGroup>
