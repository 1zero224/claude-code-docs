---
title: Claude Fable 5.2 和 Claude Mythos 5.2 迁移指南
url: https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide
description: 借助本迁移指南，从 Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5 或更早的 Claude Fable、Claude Mythos 或 Claude Opus 模型切换到 Claude Fable 5.2 和 Claude Mythos 5.2。启用 Claude Fable 5.2 和 Claude Mythos 5.2 的指南包括模型 ID、破坏性变更和迁移清单。
---

<Note>
  本指南涵盖 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 代码的迁移。如果您使用 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)，则除了更新模型名称之外无需进行任何更改。
</Note>

让 Claude Code 为您迁移代码

```text
/claude-api migrate 将此项目迁移到 claude-fable-5-2
```

如果您的 Claude Managed Agents 智能体配有顾问，那么顾问的 `model` 是另一个需要更新的模型名称：在更改智能体的模型时，将其设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。请参阅 [Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-advisor-pairs)。如果智能体的 `model` 对象设置了 `speed: "fast"`，也请将该设置移除。

以下各节介绍如何从 Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5 或更早的 Claude Fable、Claude Mythos 或 Claude Opus 模型迁移到 Claude Fable 5.2 或 Claude Mythos 5.2。有关变更本身，请参阅 [Claude Fable 5.2 的新功能](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2)。有关行为差异和特定于模型的提示模式，请参阅[为 Claude Fable 5.2 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2)。

Claude Fable 5.2 可在 Claude API、[Amazon Bedrock（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)、[Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 和 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) 具有相同的能力，仅供通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织使用。

`claude-fable-5-2` 和 `claude-mythos-5-2` 共有的基线设置：

* **思考：**[Adaptive thinking](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（自适应思考）始终开启，与 Claude Fable 5.1 相同。模型会决定何时思考以及思考多少，而 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 参数控制思考的深度。`thinking: {type: "disabled"}` 和手动扩展思考（`thinking: {type: "enabled", budget_tokens: N}`）都会返回 400 错误。
* **Effort：**&#x652F;持全部五个 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别，默认值为 `high`，与 Claude Fable 5.1 相同。请参阅 [Claude Fable 5.2 的推荐 effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)。
* **Prefill（预填充）：**&#x9884;填充 assistant 消息会返回 400 错误，与 Claude Fable 5.1 相同。请改用系统提示中的指令。
* **采样参数：**&#x7701;略 `temperature`、`top_p` 和 `top_k`，与 Claude Fable 5.1 相同。有关会返回 400 错误的取值，请参阅[采样参数](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#sampling-parameters)。
* **工具选择：**&#x652F;持 `{type: "auto"}`（默认值）和 `{type: "none"}`。使用 `{type: "any"}` 或 `{type: "tool", name: "..."}` 强制工具调用会返回 400 错误，与 Claude Fable 5.1 相同。
* **结构化输出：**&#x5728;使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)（CMEK）的组织中，[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)对这两个模型均不可用，与 Claude Fable 5.1 相同。这包括严格工具使用和 JSON 输出。如果您的请求在此类组织中设置了 `strict: true` 或 `output_config.format`，请将其移除，并在您的代码中验证输出。
* **跨模型保留思考：**&#x5728; Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 和 Claude Mythos 5.2 可以读取彼此的思考块，以及其他模型（包括 Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5）的思考块。没有任何其他模型能读取它们的思考块。完整列表请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。
* **绑定账户的思考：**&#x43;laude Fable 5.2 和 Claude Mythos 5.2 的思考块仅在生成它们的账户或与之关联的账户中有效。当其他账户发送这些思考块之一时，API 会在模型接收之前丢弃该块，请求仍会成功。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，如果使用 `thinking-binding-controls-2026-08-01` beta 标头，响应会在 `input_transformations` 中列出每个被丢弃的块，并附带 `reason: "organization_binding_mismatch"`。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* **上下文窗口和输出：**&#x9ED8;认提供 [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)，每个请求最多可输出 128k 令牌。
* **定价：**&#x6BCF;百万输入令牌 8 美元，每百万输出令牌 40 美元，而 Claude Fable 5.1 和 Claude Mythos 5.1 分别为 10 美元和 50 美元。提示缓存读取为每百万令牌 0.20 美元，而这些模型为 0.25 美元。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。
* **数据保留：**&#x8FD9;两个模型都要求 30 天数据保留，除非获得 Anthropic 明确授权，否则在 “zero data retention”（零数据保留），即 ZDR 安排下不可用，并且被指定为 Covered Models，与 Claude Fable 5.1 和 Claude Mythos 5.1 相同。在 Claude API 上，来自未启用 30 天保留的组织或工作区的请求会返回 400 `invalid_request_error`。有 ZDR 安排的组织应联系其 Anthropic 客户团队，或为单个工作区配置保留。有关各平台的详细信息，请参阅[特定模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。
* **Priority Tier：**&#x4E24;个模型均不支持 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers#supported-models)。
* **快速模式：**&#x4E24;个模型均在 Messages API 请求中支持[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出了每个模型支持的平台。

两个模型的不同之处：

* **可用性：**&#x5728; Claude API 上，Claude Fable 5.2 无需访问审批。Claude Mythos 5.2 仅供通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织使用。如需申请访问权限，请申请适用于您用例的计划，或联系您的 Anthropic 客户团队。
* **速率限制：**&#x43;laude Fable 5.2 有自己的速率限制，独立于 Claude Fable 5.1 和 Claude Fable 5 共享的合并限制。Claude Mythos 5.2 与 Claude Mythos 5.1 和 Claude Mythos 5 共享一个合并限制。请参阅[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)。

## 从 Claude Fable 5.1 迁移到 Claude Fable 5.2

迁移基本上可以直接替换。API 接口、始终开启的自适应思考、强制工具选择限制、拒绝处理、`stop_details` 类别以及数据保留要求都与 Claude Fable 5.1 一致。Claude Fable 5.2 成本更低：每百万输入令牌 8 美元，每百万输出令牌 40 美元，而 Claude Fable 5.1 分别为 10 美元和 50 美元。有四项变更可能会破坏您的代码。用于捕获对早期轮次编辑的 [“prefix check”（前缀检查）](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)适用于每个账户。Claude Fable 5.1 无法读取 Claude Fable 5.2 的思考块。早期的 computer use 工具 `computer_20251124` 在除 Amazon Bedrock 之外的所有平台上都会返回 400 错误。使用 Advisor 工具时，Claude Fable 5.2 “executor”（执行器）只接受 Claude Fable 5.2 或 Claude Mythos 5.2 作为其顾问。

### 更新您的模型名称

```python
model = "claude-fable-5-1"  # Before
model = "claude-fable-5-2"  # After
```

模型 ID 在所有平台上均为 `claude-fable-5-2`，但 Amazon Bedrock 除外。在 Amazon Bedrock 上，模型 ID 为 `global.anthropic.claude-fable-5-2` 或 `us.anthropic.claude-fable-5-2`，并且只有 `bedrock-runtime` 端点提供该模型。

### 破坏性变更

1. **前缀检查适用于每个账户：**&#x6765;自 Claude Fable 5.2 的每个 `thinking` 块仅对其之前的 `system` 提示、`tools` 和对话历史有效，与 Claude Fable 5.1 相同。Claude Fable 5.2 对每个账户都强制执行此[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)，无论账户创建时间长短。Claude Fable 5.1 默认对 2026 年 8 月 31 日或之后创建的账户强制执行。因此在较旧的账户上，在 Claude Fable 5.1 上能通过的请求可能会在 Claude Fable 5.2 上失败。

   如果由 Claude Code、claude.ai、[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 或 [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) 管理您的对话历史，则您无需做任何更改。如果您的代码自行构建 `messages` 数组，请将其在请求之间所做的操作与[前缀检查影响哪些用户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#who-is-affected)中的列表进行比较。在进行了其中某项编辑后仍回传思考块的请求会返回 400 错误：

   ```text wrap
   messages.5.content.0: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block". That setting requires the `thinking-binding-controls-2026-08-01` value in the `anthropic-beta` header.
   ```

   使用相同的请求体重试并不能消除该错误。要在切换生产流量之前找出这些编辑，请针对您当前使用的模型（Claude Fable 5.1 或 Claude Opus 5.5）运行[检查您的代码是否编辑了前缀](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#how-to-tell-whether-your-integration-is-impacted)中的检查。如果您使用 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)但没有[固定服务器的工具列表](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#pin-mcp-tool-list)，还请针对 `claude-fable-5-2` 运行该检查：在请求之间发生变化的工具列表对于 Claude Fable 5.2 的思考块而言属于编辑，但对于 Claude Fable 5.1 或 Claude Opus 5.5 的块则不是。该检查适用于任何账户，因为设置 `prefix_mismatch_behavior` 会让请求选择加入该检查。[保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)给出了每种模式的修复方法。

   在生产环境中，如果不匹配只可能意味着您代码中的错误，请将 `prefix_mismatch_behavior` 保留为默认值 `"error"`。如果要在修复编辑期间丢弃受影响的块并在没有该推理的情况下继续，请将其设置为 `"drop_block"`。请将此视为权宜之计，而非长期设置：[丢弃块可能会增加输出令牌](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#drop-block-cost)。

2. **Claude Fable 5.1 无法读取 Claude Fable 5.2 的思考块：**&#x5728; Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 可以读取 Claude Fable 5.1 的思考块，因此从 `claude-fable-5-1` 切换到 `claude-fable-5-2` 的对话会保留其早期推理。反方向则不行，Claude Fable 5.1 无法读取 Claude Fable 5.2 的思考块。如果路由器切换、客户端重试或[分类器拒绝回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)将对话从 Claude Fable 5.2 转移到其他模型（包括 Claude Fable 5.1），API 会丢弃该模型无法读取的块。请求会成功，并且您不会为被丢弃的输入令牌付费。随后模型会在没有该推理的情况下工作，这可能会增加切换后第一个轮次的成本和延迟。如果使用 `thinking-binding-controls-2026-08-01` beta 标头，`input_transformations` 会列出每个被丢弃的块，并附带 `reason: "model_binding_mismatch"`。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。

3. **使用 computer use 工具集代替 `computer_20251124`：**&#x7C7B;型为 `computer_20251124` 的 `tools` 条目在除 Amazon Bedrock 之外的所有平台上都会返回 400 错误。请改为声明 `computer_toolset_20260801` 工具集：移除 beta 标头，并发送不含 `name` 或显示尺寸的条目。然后按照[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)中的说明，为该工具集更新您的智能体循环。在 Amazon Bedrock 上，Claude Fable 5.2 和 Claude Mythos 5.2 接受早期工具：使用 InvokeModel API 时，请在请求体的 `anthropic_beta` 数组中发送 `computer-use-2025-11-24`。

4. **Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问：**&#x4F7F;用处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，顾问的能力必须至少与执行器相当。Claude Fable 5.2 或 Claude Mythos 5.2 执行器只接受 Claude Fable 5.2 或 Claude Mythos 5.2 作为其顾问，任何其他顾问（包括 Claude Fable 5.1 和 Claude Mythos 5.1）都会返回 400 `invalid_request_error`。更改执行器的模型 ID 时，请将 Advisor 工具定义中的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。更改顾问的 `model` 属于对 `tools` 的编辑，因此请在新对话中进行此更改。在更改后回传早期思考块的请求将无法通过前缀检查（请参阅[前缀检查适用于每个账户](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-preserved-thinking)）并返回 400 错误，除非您将 `prefix_mismatch_behavior` 设置为 `"drop_block"`。如果要保留当前执行器而只更改顾问，请将顾问的 `model` 设置为 `claude-fable-5-2`：每个执行器模型都接受它。Claude Fable 5.2 顾问以加密形式返回其建议，因此如果您的代码读取建议文本，请参阅[结果变体](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#result-variants)。相同的配对规则也适用于 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 智能体的顾问，因此在更改智能体的模型时，请将该顾问的 `model` 设置为这两个模型之一。请参阅[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#model-compatibility)和[为会话配置顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。

### 行为变更

1. **要求减少思考的提示语句无法阻止模型在 `medium` 及以上 effort 级别进行思考：**&#x43;laude Fable 5.1 通常会遵循诸如"直接回答，无需深思"之类的语句。在 Anthropic 的测试中，此类语句并未阻止 Claude Fable 5.2 在 `medium` 及以上级别进行思考。请移除这些语句并降低 effort 级别。请参阅[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#calibrate-effort)。
2. **在较高 effort 级别下，模型可能会启动子智能体来审查自己的工作：**&#x5728;较高 effort 级别（例如 `xhigh` 和 `max`）的智能体编码产品中，Anthropic 观察到 Claude Fable 5.2 使用子智能体来审查自己的工作。Claude Fable 5.1 在相同任务上不会这样做。子智能体可能占会话成本的很大一部分。请参阅[为子代理设定预期](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#set-expectations-for-subagents)。
3. **较小的 `max_tokens` 可能不会为文本留出空间：**&#x601D;考令牌计入 `max_tokens`，因此 `max_tokens` 较小的请求可能会在 `thinking` 块之后、任何文本之前以 `stop_reason: "max_tokens"` 停止。通常能让 Claude Fable 5.1 不进行思考的提示语句，并不能阻止 Claude Fable 5.2 在 `medium` 及以上 effort 级别进行思考。因此，在 Claude Fable 5.1 上依赖此类语句来适应较小 `max_tokens` 的请求，在 Claude Fable 5.2 上可能会在输出任何文本之前就耗尽令牌。请提高 `max_tokens` 以为思考留出空间，或选择较低的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。

有关迁移提示时需要检查的其他行为，请参阅[行为差异](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#behavior-differences)。

### 迁移清单

* 将模型名称从 `claude-fable-5-1` 更新为 `claude-fable-5-2`。
* 在迁移生产流量之前检查您的[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)。Claude Fable 5.2 有自己的速率限制，独立于 Claude Fable 5.1 和 Claude Fable 5 共享的限制。
* 如果您在 Amazon Bedrock 以外的任何平台上通过 `computer_20251124` 使用 computer use，Claude Fable 5.2 会针对该工具返回 400 错误。请改为声明 `computer_toolset_20260801`，并为该工具集更新您的智能体循环。
* 如果您在以 Claude Fable 5.2 作为执行器时使用 Advisor 工具，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。
* 如果您的代码自行构建 `messages` 数组，请使用 `thinking-binding-controls-2026-08-01` beta 标头和 `prefix_mismatch_behavior: "drop_block"` 运行一个会话，记录 `input_transformations`，并修复每个 `prefix_binding_mismatch` 条目。即使您的账户在 Claude Fable 5.1 上从未触发过该检查，也请执行此操作。
* 选择生产环境的 `prefix_mismatch_behavior`：`"error"`（默认值），或在修复编辑期间作为权宜之计使用 `"drop_block"`，因为[丢弃块可能会增加输出令牌](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#drop-block-cost)。监控 400 错误或 `input_transformations` 条目。
* 在每个轮次中继续原样回传 `thinking` 块，包括空块。如果路由器或回退可能将对话从 Claude Fable 5.2 转移到 Claude Fable 5.1，请预期 Claude Fable 5.1 将在没有 Claude Fable 5.2 思考块的情况下运行。
* 如果您通过不同的账户重放已存储的对话，请通过生成该对话的账户或与之关联的账户来重放每个对话。API 会丢弃由其他账户生成的 Claude Fable 5.2 思考块。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* 在您自己的评估上重新运行 [effort 扫描](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)，并重新建立成本和延迟基线。
* 对照[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)，检查您沿用的系统提示、技能和其他指令。

## 从 Claude Mythos 5.1 迁移到 Claude Mythos 5.2

[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) 是面向通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织的 Claude Fable 5.2 对应版本。在切换模型 ID 之前，请确认您的组织有权访问 Claude Mythos 5.2。

Claude Mythos 5.2 对每个账户都强制执行[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)，与 Claude Fable 5.2 相同。在 Claude API 上，Claude Mythos 5.1 已对每个账户返回该错误（请参阅[未设置 `prefix_mismatch_behavior` 时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)）。在 Amazon Bedrock 上，Claude Mythos 5.2 会针对已编辑的早期轮次返回 400 错误，而 Claude Mythos 5.1 在该平台上会接受此类请求。如果您的代码在请求之间更改 `system`、`tools` 或早期消息，请参阅[编辑早期轮次会使思考块失效](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-edited-history)了解需要更改的内容和测试方法，并针对 `claude-mythos-5-2` 运行测试。

在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Mythos 5.2 可以读取 Claude Mythos 5.1 的思考块，因此从 `claude-mythos-5-1` 切换到 `claude-mythos-5-2` 的对话会保留其早期推理。反方向则不行，Claude Mythos 5.1 无法读取 Claude Mythos 5.2 的思考块：API 会丢弃这些块，请求仍会成功。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。

使用处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，Claude Mythos 5.2 执行器只接受 Claude Mythos 5.2 或 Claude Fable 5.2 作为其顾问。Claude Mythos 5.1 或 Claude Fable 5.1 顾问会返回 400 错误。请参阅 [Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-advisor-pairs)。

Claude Mythos 5.2 通过 `computer_toolset_20260801` 工具集支持 computer use。类型为 `computer_20251124` 的 `tools` 条目在除 Amazon Bedrock（早期工具在该平台上可以使用）之外的所有平台上都会返回 400 错误。请参阅[使用 computer use 工具集代替 `computer_20251124`](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-computer-use)。

Claude Mythos 5.2 的价格为每百万输入令牌 8 美元、每百万输出令牌 40 美元，而 Claude Mythos 5.1 分别为 10 美元和 50 美元。提示缓存读取为每百万令牌 0.20 美元，而此前为 0.25 美元。

### 更新您的模型名称

```python
model = "claude-mythos-5-1"  # Before
model = "claude-mythos-5-2"  # After
```

### 迁移清单

* 将模型名称从 `claude-mythos-5-1` 更新为 `claude-mythos-5-2`。
* 如果您在 Amazon Bedrock 以外的任何平台上通过 `computer_20251124` 使用 computer use，Claude Mythos 5.2 会针对该工具返回 400 错误。请改为声明 `computer_toolset_20260801`，并为该工具集更新您的智能体循环。
* 如果您在以 Claude Mythos 5.2 作为执行器时使用 Advisor 工具，请将顾问的 `model` 设置为 `claude-mythos-5-2` 或 `claude-fable-5-2`。
* 在每个轮次中继续原样回传 `thinking` 块，包括空块。
* 如果您通过不同的账户重放已存储的对话，请通过生成该对话的账户或与之关联的账户来重放每个对话。API 会丢弃由其他账户生成的 Claude Mythos 5.2 思考块。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* 如果您的代码自行构建 `messages` 数组，请针对 `claude-mythos-5-2` 运行[编辑早期轮次会使思考块失效](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-edited-history)中的检查，修复每个 `prefix_binding_mismatch` 条目，并保持对话历史仅追加（请参阅[哪些算作编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#what-counts-as-an-edit)）。仅追加的历史还能保持提示缓存处于预热状态。
* 处理 `stop_reason: "refusal"`，并在读取响应内容之前读取 `stop_details.category`。请参阅[拒绝和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)。
* 重新运行您的 effort 扫描，并重新建立成本和延迟基线。
* 对照[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)，检查您沿用的系统提示、技能和其他指令。

## 从 Claude Opus 5.5 迁移到 Claude Fable 5.2

Claude Fable 5.2 使用与 [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview) 相同的 Messages API 和工具使用模式。两个模型都始终进行自适应思考，并对强制工具选择返回 400 错误，因此您请求中的这些部分可以直接沿用。当您将流量或回退路径迁移到 Claude Fable 5.2 时，以下差异适用。

### 更新您的模型名称

```python
model = "claude-opus-5-5"  # Before
model = "claude-fable-5-2"  # After
```

### 变更内容

1. **需要数据保留：**&#x43;laude Fable 5.2 要求 30 天数据保留，除非获得 Anthropic 明确授权，否则在零数据保留（ZDR）安排下不可用。如果您的组织或工作区在 ZDR 下运行 Claude Opus 5.5，请在切换之前为某个工作区启用 30 天保留，或联系您的 Anthropic 客户团队。否则，在 Claude API 上，对 Claude Fable 5.2 的每个请求都会返回 400 `invalid_request_error`。请参阅[特定模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。

2. **前缀检查适用于每个账户：**&#x43;laude Opus 5.5 默认对 2026 年 8 月 31 日或之后创建的账户强制执行前缀检查。Claude Fable 5.2 对每个账户都强制执行，无论账户创建时间长短。如果您的账户较旧且您的代码在请求之间更改 `system`、`tools` 或早期消息，那么在 Claude Opus 5.5 上正常工作的请求可能会在 Claude Fable 5.2 上失败。要在切换之前进行测试，请参阅[前缀检查适用于每个账户](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-preserved-thinking)。

3. **Claude Opus 5.5 无法读取 Claude Fable 5.2 的思考块：**&#x4ECE; Claude Fable 5.2 切换到 Claude Opus 5.5 的对话会保留其文本和工具调用，但切换后的轮次将在没有 Claude Fable 5.2 推理的情况下运行。API 会在模型接收之前丢弃这些块，请求会成功，被丢弃的块不会计费。预计切换后第一个轮次的成本和延迟会更高。反方向上，在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 可以读取 Claude Opus 5.5 的思考块，因此从 `claude-opus-5-5` 切换到 `claude-fable-5-2` 的对话会保留其早期推理。

4. **安全分类器和回退：**&#x43;laude Fable 5.2 运行的安全分类器涵盖与 Claude Fable 5.1 相同的 `stop_details` 类别。请处理[拒绝类别表](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)中的每个类别，而不仅仅是您在 Claude Opus 5.5 上见过的类别。要在另一个模型上重新运行被拒绝的请求，请设置 `fallbacks: "default"`，它会在 Anthropic 针对该类别推荐的模型上重试，或者构建您自己的重试逻辑。该参数在 Claude API 和 Claude Models on Google Cloud (Anthropic managed) 上处于 beta 阶段，并且需要 `server-side-fallback-2026-07-01` beta 标头。Claude Fable 5.2 允许的回退目标是 Claude Opus 4.8（`claude-opus-4-8`）和 Claude Opus 5（`claude-opus-5`）。服务器端回退不会重试因 `"reasoning_extraction"` 而被拒绝的请求。该拒绝会直接返回给您。回退模型不会收到 Claude Fable 5.2 的思考块。请参阅[拒绝和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)。

5. **定价：**&#x43;laude Fable 5.2 的价格为每百万输入令牌 8 美元、每百万输出令牌 40 美元，而 Claude Opus 5.5 分别为 4 美元和 20 美元。两个模型的提示缓存读取均为每百万令牌 0.20 美元。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。

6. **快速模式：**&#x4E0E; Claude Opus 5.5 一样，Claude Fable 5.2 在 Claude API 上的 Messages API 请求中支持[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)（`speed: "fast"`）。Claude Fable 5.2 的快速模式价格为每百万输入令牌 16 美元、每百万输出令牌 80 美元，而 Claude Opus 5.5 的快速模式分别为 8 美元和 40 美元。请参阅[快速模式定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)。

   Claude Fable 5.2 还在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上支持快速模式，使用相同的 `speed: "fast"` 设置。

   Claude Fable 5.2 还在 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 上支持快速模式，使用相同的 `speed: "fast"` 设置。该平台的价格由 Google Cloud 设定。请参阅[云平台定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)。

   Claude Fable 5.2 还在 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上针对托管在 Anthropic 上的部署支持快速模式。在该平台上，快速模式是一个单独的模型：请将请求发送到 `claude-fable-5-2-fast` 部署，并省略 `speed`。

   Claude Mythos 5.2 还在 Claude API 上的 Messages API 请求中支持快速模式，价格与 Claude Fable 5.2 的快速模式价格相同。[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出了 Claude Mythos 5.2 支持快速模式的所有平台。

   请在 Claude Managed Agents 中移除 `speed: "fast"`。此外，在[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)未列出您要迁移到的模型的任何平台或托管选项上，也请将其移除。

7. **CMEK 组织中的结构化输出：**&#x5728;使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，Claude Opus 5.5 支持[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)，而 Claude Fable 5.2 不支持。请从迁移的请求中移除 `strict: true` 和 `output_config.format`，并在您的代码中验证输出。

8. **默认 effort 为 `high`：**&#x43;laude Opus 5.5 默认为 `medium`，因此省略 `effort` 的请求在 Claude Fable 5.2 上会高一个级别运行。请求会成功，这一变化仅体现为成本、延迟和结果的不同。请显式设置 `effort`，然后在您自己的评估上重新运行 [effort 扫描](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)。

9. **Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问：**&#x4F7F;用处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，Claude Opus 5.5 执行器接受多个顾问，其中包括 Claude Opus 5.5 和 Claude Fable 5.1。Claude Fable 5.2 执行器只接受 Claude Fable 5.2 或 Claude Mythos 5.2，任何其他顾问都会返回 400 错误。更改执行器的模型时，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。请参阅 [Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-advisor-pairs)。

### 迁移清单

* 如果您的组织有 ZDR 安排，请首先为工作区启用 30 天保留，或与您的 Anthropic 客户团队确认资格。
* 将模型 ID 从 `claude-opus-5-5` 更新为 `claude-fable-5-2`。
* 在迁移生产流量之前检查您的[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)。Claude Fable 5.2 的速率限制独立于 Claude Opus 5.5，且每个层级的令牌限制更低。
* 如果您的代码自行构建 `messages` 数组，请运行[前缀检查适用于每个账户](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-preserved-thinking)中的测试，即使您的账户在 Claude Opus 5.5 上从未触发过该检查。
* 处理每个 `stop_details.category`。如果您设置了显式的 `fallbacks` 列表，请仅指定 Claude Fable 5.2 允许的目标。
* 如果路由器或回退将对话从 Claude Fable 5.2 转移到 Claude Opus 5.5，请预期切换后的轮次将在没有 Claude Fable 5.2 思考块的情况下运行。
* 如果您通过不同的账户重放已存储的对话，请通过生成该对话的账户或与之关联的账户来重放每个对话。API 会丢弃由其他账户生成的 Claude Fable 5.2 思考块。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* 如果您在 Messages API 请求中使用快速模式，请仅在[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出您要迁移到的模型的平台或托管选项上保留 `speed: "fast"`。在这些平台和托管选项上，请根据[快速模式价格](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)重新建立成本基线。在所有其他平台或托管选项上以及在 Claude Managed Agents 中，请移除 `speed: "fast"`。您不需要 `fast-mode-2026-02-01` beta 标头。在 Claude API 上，发送该标头的请求仍可正常工作。Microsoft Foundry 中的 Claude 是例外：在该平台上，请移除 `speed: "fast"`，并将请求发送到 `claude-fable-5-2-fast` 部署。
* 如果您在以 Claude Fable 5.2 作为执行器时使用 Advisor 工具，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。
* 在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，移除 `strict: true` 和 `output_config.format`，并在您的代码中验证输出。
* 显式设置 `effort`：Claude Fable 5.2 的默认值为 `high`，而 Claude Opus 5.5 的默认值为 `medium`，因此省略它的请求会高一个级别运行。然后在您自己的评估上重新运行 [effort 扫描](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)，并重新建立成本和延迟基线。
* 对照[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)，检查您沿用的系统提示、技能和其他指令。

## 从 Claude Fable 5 迁移到 Claude Fable 5.2

API 接口、上下文窗口和输出限制、分词器、始终开启的自适应思考以及拒绝处理都与 Claude Fable 5 一致。以下部分变更也适用于从 Claude Mythos 5 迁移到 Claude Mythos 5.2 的情况。[从 Claude Mythos 5 迁移到 Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-mythos-5-to-claude-mythos-5-2) 说明了具体是哪些变更。

### 更新您的模型名称

```python
model = "claude-fable-5"  # Before
model = "claude-fable-5-2"  # After
```

### 破坏性变更

1. **强制工具选择会返回 400 错误：** Claude Fable 5 接受 `tool_choice` 的 `auto`、`none`、`any` 和 `tool`。在 `claude-fable-5-2` 上，`{type: "any"}` 和 `{type: "tool", name: "..."}` 会返回 400 `invalid_request_error`，[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)端点也是如此。有关错误消息，请参阅[不支持强制工具使用](https://platform.claude.com/docs/zh-CN/api/errors#forced-tool-use-not-supported)。

   若要在不强制的情况下获得工具调用，请将 `tool_choice` 保持为 `auto`，并在您的指令中指明该工具。在工具上设置 `strict: true`，使调用符合您的 schema。请参阅[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)。如果您强制使用工具只是为了获得符合 schema 的 JSON，请改用 [JSON 输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#usage)。在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，"structured outputs"（[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)）不适用于 Claude Fable 5.2 或 Claude Mythos 5.2。这包括严格工具使用和 JSON 输出。在这种情况下，请保持 `auto` 和指令，省略 `strict`，并在您的代码中验证工具输入。

   如果您的应用程序要求在当前轮次进行特定的工具调用，请在最新的 `user` 轮次之后追加一条[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)。指明该工具，说明本轮次必须进行该调用，并告诉 Claude 以该调用开始其响应。较早的轮次保持不变，因此它们的提示缓存命中和思考块仍然有效。[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)列出了支持该功能的平台。对于不得调用工具的轮次，`tool_choice: {"type": "none"}` 仍然有效。请参阅[强制工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#forcing-tool-use)。

2. **编辑较早的轮次会使思考块失效，适用于所有账户：** Claude Fable 5 不会检查较早的轮次是否发生了变化。在 Claude Fable 5.2 上，每个 `thinking` 块仅相对于其之前的 `system` 提示、`tools` 和消息有效。Claude Fable 5.2 对所有账户强制执行这一"prefix check"（[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)）。如果在上述任一内容发生变化后，请求仍将思考块发回，则会返回 400 错误，且使用相同请求体重试也无法消除该错误。

   如果由 Claude Code、claude.ai、[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 或 [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) 管理您的对话历史，则您无需做任何更改。如果您的代码自行构建 `messages` 数组，请保持历史记录仅追加。在会话期间冻结 `system` 和 `tools`。通过[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)进行指令和工具的更改，并使用服务器端[压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)或[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)来裁剪上下文。[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)列出了支持该功能的平台。[在不编辑前缀的情况下进行更改](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#replace-prefix-edits)给出了每种常见编辑的替代方案。

   若要在切换生产流量之前找出这些编辑，请针对 `claude-fable-5-2` 运行[检查您的代码是否编辑了前缀](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#how-to-tell-whether-your-integration-is-impacted)中的检查。在生产环境中，如果不匹配只可能意味着您的代码存在错误，请将 `prefix_mismatch_behavior` 保持为默认值 `"error"`。若要在修复编辑期间丢弃受影响的块并在没有该推理的情况下继续，请配合 `thinking-binding-controls-2026-08-01` beta 标头将其设置为 `"drop_block"`。请将其视为权宜之计，而非长期设置：[丢弃块可能会增加输出令牌](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#drop-block-cost)。

3. **Claude Fable 5 无法读取 Claude Fable 5.2 的思考块：** 在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 可以读取 Claude Fable 5 的块，因此从 `claude-fable-5` 迁移到 `claude-fable-5-2` 的对话会保留其先前的推理。反过来，Claude Fable 5 无法读取 Claude Fable 5.2 的块。如果路由切换、客户端重试或[分类器拒绝回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)将对话从 Claude Fable 5.2 转移到其他模型（包括 Claude Fable 5），API 会丢弃该模型无法读取的块。请求会成功，并且不会对被丢弃的输入令牌向您计费。使用 `thinking-binding-controls-2026-08-01` beta 标头时，`input_transformations` 会列出每个被丢弃的块，并附带 `reason: "model_binding_mismatch"`。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。

4. **使用 computer use 工具集代替 `computer_20251124`：** 在 Claude Fable 5.2 上，类型为 `computer_20251124` 的 `tools` 条目在除 Amazon Bedrock 以外的所有平台上都会返回 400 错误。Claude Fable 5 在配合其 beta 标头时接受该工具。请改为声明 `computer_toolset_20260801` 工具集，如[使用 computer use 工具集代替 `computer_20251124`](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-computer-use) 所述。

5. **Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问：** 使用处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，Claude Fable 5.2 执行器仅接受 Claude Fable 5.2 或 Claude Mythos 5.2 作为其顾问。Claude Fable 5 执行器接受的其他顾问（例如 Claude Fable 5 和 Claude Opus 5.5）会返回 400 错误。在更改执行器的 `model` 时，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。请参阅 [Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-advisor-pairs)。

### 其他变更

* **定价：** 每百万输入令牌 8 美元，每百万输出令牌 40 美元，而 Claude Fable 5 分别为 10 美元和 50 美元。提示缓存读取为每百万令牌 0.20 美元，而此前为 1 美元。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。
* **Priority Tier：** Claude Fable 5.2 不支持 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers#supported-models)。Claude Fable 5 支持。如果您的组织有 Priority Tier 承诺，请单独规划容量。
* **按消息设置 effort（beta）：** 若要在对话中途更改 effort 而不丢失提示缓存命中，请发送一条[按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) 的 `role: "system"` 消息，而不是在请求之间更改 `output_config.effort`。按消息设置 effort 页面列出了支持该功能的平台。

### 迁移清单

* 将模型名称从 `claude-fable-5` 更新为 `claude-fable-5-2`。
* 在转移生产流量之前，请检查您的"rate limits"（[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)）。Claude Fable 5.2 有自己的速率限制，与 Claude Fable 5.1 和 Claude Fable 5 共享的速率限制相互独立。
* 将强制 `tool_choice`（`{type: "any"}` 或 `{type: "tool", ...}`）替换为 `{type: "auto"}` 加上明确的指令和 `strict: true` 工具，或替换为 JSON 输出。将指令放在 `user` 轮次中。当您的应用程序要求进行该调用时，请改为将其放在对话中途的 `role: "system"` 消息中（适用于[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)列出的平台）。在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，严格工具使用和 JSON 输出不适用于 Claude Fable 5.2。在这种情况下，请保持 `{type: "auto"}` 和指令，并在您的代码中验证工具输入。
* 如果您的代码自行构建 `messages` 数组，请使用 `thinking-binding-controls-2026-08-01` beta 标头和 `prefix_mismatch_behavior: "drop_block"` 运行一个会话，记录 `input_transformations`，并修复每个 `prefix_binding_mismatch` 条目。模型切换后出现的 `model_binding_mismatch` 条目属于预期情况。
* 保持历史记录仅追加：在会话开始时冻结 `system` 和 `tools`，通过 `role: "system"` 消息进行会话中途的更改，并在服务器上裁剪上下文。[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)列出了支持该功能的平台。
* 选择生产环境的 `prefix_mismatch_behavior`：`"error"`（默认值），或者在修复编辑期间将 `"drop_block"` 作为权宜之计，因为[丢弃块可能会增加输出令牌](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#drop-block-cost)。监控 400 错误或 `input_transformations` 条目。
* 在每个轮次中继续原样传回 `thinking` 块，包括空块。如果路由或回退可能将对话从 Claude Fable 5.2 转移到 Claude Fable 5，请预期 Claude Fable 5 将在没有 Claude Fable 5.2 的块的情况下运行。
* 如果您通过不同的账户重放存储的对话，请通过生成该对话的账户或与其关联的账户重放每个对话。API 会丢弃由其他账户生成的 Claude Fable 5.2 思考块。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* 如果您在 Amazon Bedrock 以外的任何平台上通过 `computer_20251124` 使用 computer use，Claude Fable 5.2 会对该工具返回 400 错误。请改为声明 `computer_toolset_20260801`，并针对该工具集更新您的代理循环。
* 如果您在将 Claude Fable 5.2 作为执行器的情况下使用 Advisor 工具，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。
* 如果您在请求之间更改 effort，请在您的平台支持的情况下，将该更改移至处于 beta 阶段的按消息设置 effort 的 `role: "system"` 消息。
* 如果您的组织有 Priority Tier 承诺，请单独规划容量。
* 处理 `stop_reason: "refusal"` 并读取 `stop_details.category`。考虑使用 `fallbacks: "default"`（在 Claude API 和 Claude Models on Google Cloud (Anthropic managed) 上处于 beta 阶段）。
* 在您自己的评估上重新运行 [effort 扫描](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)，并重新建立成本和延迟的基线。
* 对照[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)，检查您沿用的系统提示、skills 和其他指令。

## 从 Claude Mythos 5 迁移到 Claude Mythos 5.2

[Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) 是 Claude Fable 5.2 的对应模型，面向通过 Anthropic 验证计划（例如 [Cyber Verification Program](https://support.claude.com/en/articles/14604842)）验证的组织。在切换模型 ID 之前，请确认您的组织有权访问 Claude Mythos 5.2。

[从 Claude Fable 5 迁移到 Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-fable-5-to-claude-fable-5-2) 中的四项变更按原文适用：

* [强制工具选择会返回 400 错误](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-forced-tool-choice)。
* [编辑较早的轮次会使思考块失效](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-edited-history)（适用于所有账户）。
* [使用 computer use 工具集代替 `computer_20251124`](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-computer-use)。
* Claude Fable 5.2 或 Claude Mythos 5.2 执行器[需要这两个模型之一作为其顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-advisor-pairs)。

在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Mythos 5.2 可以读取 Claude Mythos 5 的思考块，因此从 `claude-mythos-5` 迁移到 `claude-mythos-5-2` 的对话会保留其先前的推理。反过来，Claude Mythos 5 无法读取 Claude Mythos 5.2 的块：API 会丢弃这些块，并且请求会成功。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。Claude Mythos 5.2 的价格为每百万输入令牌 8 美元、每百万输出令牌 40 美元，而 Claude Mythos 5 分别为 10 美元和 50 美元。提示缓存读取为每百万令牌 0.20 美元，而此前为 1 美元。

### 更新您的模型名称

```python
model = "claude-mythos-5"  # Before
model = "claude-mythos-5-2"  # After
```

### 迁移清单

* 将模型名称从 `claude-mythos-5` 更新为 `claude-mythos-5-2`。
* 将强制 `tool_choice`（`any` 或 `tool`）替换为 `auto` 加上明确的指令和 `strict: true` 工具，或替换为 JSON 输出。在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，严格工具使用和 JSON 输出不适用于 Claude Mythos 5.2。在这种情况下，请保持 `auto` 和指令，并在您的代码中验证工具输入。
* 如果您在 Amazon Bedrock 以外的任何平台上通过 `computer_20251124` 使用 computer use，Claude Mythos 5.2 会对该工具返回 400 错误。请改为声明 `computer_toolset_20260801`，并针对该工具集更新您的代理循环。
* 如果您在将 Claude Mythos 5.2 作为执行器的情况下使用 Advisor 工具，请将顾问的 `model` 设置为 `claude-mythos-5-2` 或 `claude-fable-5-2`。
* 在每个轮次中继续原样传回 `thinking` 块，包括空块。
* 如果您通过不同的账户重放存储的对话，请通过生成该对话的账户或与其关联的账户重放每个对话。API 会丢弃由其他账户生成的 Claude Mythos 5.2 思考块。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* 如果您的代码自行构建 `messages` 数组，请针对 `claude-mythos-5-2` 运行[编辑较早的轮次会使思考块失效](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-edited-history)中的检查，修复每个 `prefix_binding_mismatch` 条目，并保持对话历史仅追加（请参阅[哪些算作编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#what-counts-as-an-edit)）。仅追加的历史记录还能保持提示缓存处于预热状态。
* 处理 `stop_reason: "refusal"`，并在读取响应内容之前读取 `stop_details.category`。请参阅[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)。
* 重新运行您的 effort 扫描，并重新建立成本和延迟的基线。
* 对照[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)，检查您沿用的系统提示、skills 和其他指令。

## 从 Claude Opus 5 迁移到 Claude Fable 5.2

Claude Fable 5.2 使用与 [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview) 相同的 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 和"tool use"（[工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)）模式。1M 令牌的"context window"（上下文窗口）、128k 最大输出令牌、512 令牌的"prompt caching"（提示缓存）最小值以及[对话中途系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)均保持不变。对预填充和采样参数的限制也同样保持不变。当您将流量或回退路径转移到 Claude Fable 5.2 时，以下差异适用。

### 更新您的模型名称

```python
model = "claude-opus-5"  # Before
model = "claude-fable-5-2"  # After
```

### 变更内容

1. **思考无法禁用：** Claude Opus 5 在 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别为 `high` 或更低时接受 `thinking: {type: "disabled"}`。在 Claude Fable 5.2 上，[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)始终开启，`thinking: {type: "disabled"}` 在任何 effort 级别都会返回 400 错误。请删除该字段，并通过较低的 effort 级别来控制令牌开销。对于曾在禁用思考的情况下运行的工作负载，请重新审视 `max_tokens`，它涵盖思考和响应文本。如果该代码按位置读取内容（例如 `content[0].text`），请改为按 `type` 选择内容块，因为 `thinking` 块会排在最前面。

2. **强制工具选择会返回 400 错误：** Claude Opus 5 接受 `tool_choice` 的 `any` 和 `tool`。Claude Fable 5.2 会拒绝这两者。有关应改为发送的内容，请参阅[强制工具选择会返回 400 错误](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-forced-tool-choice)。

3. **编辑较早的轮次会使思考块失效，适用于所有账户：** Claude Opus 5 不会检查较早的轮次是否发生了变化。Claude Fable 5.2 会对所有账户进行检查。如果您的代码在请求之间编辑较早的消息、重建 `system` 或 `tools`，或在客户端进行压缩，Claude Fable 5.2 会拒绝请求或丢弃无效的思考块。有关需要更改的内容以及测试方法，请参阅[编辑较早的轮次会使思考块失效](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-edited-history)。

4. **Claude Opus 5 无法读取 Claude Fable 5.2 的思考块：** 在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 可以读取 Claude Opus 5 的块，因此从 `claude-opus-5` 迁移到 `claude-fable-5-2` 的对话会保留其先前的推理。反过来，Claude Opus 5 无法读取 Claude Fable 5.2 的块。从 Claude Fable 5.2 转移到 Claude Opus 5 的对话，在切换后的轮次中将在没有 Claude Fable 5.2 推理的情况下运行。API 会在模型接收这些块之前将其丢弃，请求会成功，并且被丢弃的块不会计费。请参阅[在对话中途切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。

5. **工具调用之间的文本以思考块形式返回：** 在 Claude Opus 5 上，模型在工具调用之间编写的文本以 `text` 块形式返回。在 Claude Fable 5.2 上，这些叙述以进度更新 `thinking` 块的形式返回，每次工具调用之前最多一个。如果您的界面会渲染这些叙述，请将 `thinking.display` 设置为 `"updates"`（该值处于 beta 阶段），以便以文本形式接收进度更新，同时推理保持隐藏；或设置为 `"summarized"` 以同时接收两者。然后渲染 `tool_use` 块之间的非空 `thinking` 块。请参阅[工具调用之间的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)。

6. **使用 computer use 工具集代替 `computer_20251124`：** Claude Opus 5 在配合其 beta 标头时接受较早的 `computer_20251124` 工具。Claude Fable 5.2 在除 Amazon Bedrock 以外的所有平台上都会对其返回 400 错误。请参阅[使用 computer use 工具集代替 `computer_20251124`](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-computer-use)。

7. **安全分类器与回退：** Claude Fable 5.2 的安全分类器涵盖的 `stop_details` 类别比 Claude Opus 5 的分类器更广泛。请预期会出现额外的 `stop_details.category` 值，例如 `"bio"`。请处理[拒绝类别表](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)中的每个类别。若要在其他模型上重新运行被拒绝的请求，请设置 `fallbacks: "default"`，或构建您自己的重试机制。该参数在 Claude API 和 Claude Models on Google Cloud (Anthropic managed) 上处于 beta 阶段，并且需要 `server-side-fallback-2026-07-01` beta 标头。[拒绝、回退与计费](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2#refusals-fallback-and-billing)列出了允许的回退目标。

8. **定价：** Claude Fable 5.2 的费用为每百万输入令牌 $8 USD、每百万输出令牌 $40 USD，而 Claude Opus 5 为 $5 USD 和 $25 USD。提示缓存读取为每百万令牌 $0.20 USD，相比之下为 $0.50 USD。请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)。

9. **需要数据保留：** Claude Fable 5.2 需要 30 天的数据保留，除非获得 Anthropic 的明确授权，否则不适用于"zero data retention"（零数据保留），即 ZDR 安排。Claude Opus 5 可在 ZDR 下使用。在 Claude API 上，来自没有 30 天保留的组织或工作区的请求会返回 400 `invalid_request_error`。请参阅[特定模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。

10. **快速模式：** Claude Opus 5 在研究预览阶段支持[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)（`speed: "fast"`），需要使用 `fast-mode-2026-02-01` beta 标头。Claude Fable 5.2 在 Claude API 上的 Messages API 请求中支持快速模式，无需该标头。发送该标头的请求可继续正常工作。Claude Fable 5.2 的快速模式费用为每百万输入令牌 $16 USD、每百万输出令牌 $80 USD，而 Claude Opus 5 的快速模式为 $10 USD 和 $50 USD。请参阅[快速模式定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)。

    Claude Fable 5.2 还在 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上支持快速模式，使用相同的 `speed: "fast"` 设置。

    Claude Fable 5.2 还在 [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed) 上支持快速模式，使用相同的 `speed: "fast"` 设置。该平台的价格由 Google Cloud 设定。请参阅[云平台定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)。

    Claude Fable 5.2 还在 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上针对托管在 Anthropic 上的部署支持快速模式。在该平台上，快速模式是一个单独的模型：请将请求发送到 `claude-fable-5-2-fast` 部署，并省略 `speed`。

    Claude Mythos 5.2 还在 Claude API 上的 Messages API 请求中支持快速模式，无需该标头，价格与 Claude Fable 5.2 的快速模式价格相同。[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出了 Claude Mythos 5.2 支持快速模式的所有平台。

    请在 Claude Managed Agents 中删除 `speed: "fast"`。如果[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)未列出您要转移到的模型，也请将其删除。

11. **CMEK 组织中的结构化输出：** 在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，Claude Opus 5 支持[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)，而 Claude Fable 5.2 不支持。请从您转移的请求中删除 `strict: true` 和 `output_config.format`，并在您的代码中验证输出。

12. **Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问：** 使用处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，Claude Opus 5 执行器接受多种顾问，其中包括 Claude Opus 5 和 Claude Fable 5.1。Claude Fable 5.2 执行器仅接受 Claude Fable 5.2 或 Claude Mythos 5.2，任何其他顾问都会返回 400 错误。在更改执行器的 `model` 时，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。请参阅 [Claude Fable 5.2 执行器需要 Claude Fable 5.2 或 Claude Mythos 5.2 顾问](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-advisor-pairs)。

### 迁移清单

* 如果您的组织有 ZDR 安排，请首先为工作区启用 30 天保留，或向您的 Anthropic 客户团队确认资格。
* 将模型 ID 从 `claude-opus-5` 更新为 `claude-fable-5-2`。
* 在转移生产流量之前，请检查您的[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)。Claude Fable 5.2 的速率限制与 Claude Opus 5 的相互独立，且在每个层级的令牌限制都更低。
* 删除任何 `thinking: {type: "disabled"}` 配置。通过较低的 effort 级别控制令牌开销，重新审视 `max_tokens`，并按 `type` 选择内容块。
* 在每个轮次中原样传回 `thinking` 块，包括空块。在工具使用循环中，将它们与工具结果一起发回。请参阅[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)。
* 将强制 `tool_choice`（`any` 或 `tool`）替换为 `auto` 加上明确的指令和 `strict: true` 工具，或替换为 JSON 输出。在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，严格工具使用和 JSON 输出不适用于 Claude Fable 5.2。在这种情况下，请保持 `auto` 和指令，并在您的代码中验证工具输入。
* 如果您的代码自行构建 `messages` 数组，请针对 `claude-fable-5-2` 运行[编辑较早的轮次会使思考块失效](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#fable-5-2-edited-history)中的检查，修复每个 `prefix_binding_mismatch` 条目，并保持历史记录仅追加。
* 如果您在 Amazon Bedrock 以外的任何平台上通过 `computer_20251124` 使用 computer use，Claude Fable 5.2 会对该工具返回 400 错误。请改为声明 `computer_toolset_20260801`，并针对该工具集更新您的代理循环。
* 如果您在将 Claude Fable 5.2 作为执行器的情况下使用 Advisor 工具，请将顾问的 `model` 设置为 `claude-fable-5-2` 或 `claude-mythos-5-2`。
* 如果路由或回退将对话从 Claude Fable 5.2 转移到 Claude Opus 5，请预期切换后的轮次将在没有 Claude Fable 5.2 思考块的情况下运行。
* 如果您通过不同的账户重放存储的对话，请通过生成该对话的账户或与其关联的账户重放每个对话。API 会丢弃由其他账户生成的 Claude Fable 5.2 思考块。请参阅[思考块仅限于生成它们的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。
* 如果您的界面会渲染工具调用之间的文本，请将 `thinking.display` 设置为 `"updates"`（该值处于 beta 阶段）或 `"summarized"`，并渲染非空的 `thinking` 块。
* 处理每个 `stop_details.category`。如果您设置了显式的 `fallbacks` 列表，请仅指定 Claude Fable 5.2 允许的目标。
* 如果您在 Claude API 上的 Messages API 请求中使用快速模式，请仅在[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出您要转移到的模型时保留 `speed: "fast"`。如果保留，请根据[快速模式价格](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)重新建立成本基线。在 Claude Managed Agents 中，请删除 `speed: "fast"`。您不需要 `fast-mode-2026-02-01` beta 标头，发送该标头的请求仍可正常工作。
* 在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，请删除 `strict: true` 和 `output_config.format`，并在您的代码中验证输出。
* 在您自己的评估上重新运行 [effort 扫描](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-fable-5-2)，并重新建立成本和延迟的基线。
* 对照[审查为早期模型设计的提示和 harness](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#review-instructions-written-for-earlier-models)，检查您沿用的系统提示、skills 和其他指令。

## 从 Claude Opus 4.8 或更早版本迁移到 Claude Fable 5.2

首先完成[从 Claude Opus 5 迁移到 Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migrating-from-claude-opus-5-to-claude-fable-5-2) 中的步骤，将 `claude-opus-4-8` 作为您要替换的模型 ID。该部分按原文适用于 Claude Opus 4.8 上的代码，因为 Claude Opus 4.8 与 Claude Opus 5 一样：

* 接受 `thinking: {type: "disabled"}`、强制工具选择、`computer_20251124` 工具和快速模式。
* 将工具调用之间的文本以 `text` 块形式返回。
* 可在零数据保留（ZDR）安排下使用。
* 价格为每百万输入令牌 5 美元、每百万输出令牌 25 美元，提示缓存读取为 0.50 美元。

本部分补充 Claude Opus 5 部分未涵盖的差异。

### 变更内容

1. **省略思考的请求也会运行思考：** 在 Claude Opus 4.8 上，除非您请求，否则思考处于关闭状态。在 Claude Fable 5.2 上，没有 `thinking` 字段的请求会以自适应思考运行。响应可能会在 `text` 块之前携带 `thinking` 块，因此请按 `type` 选择内容块，而不是读取 `content[0].text`。在工具使用循环中，将 `thinking` 块（包括空块）与工具结果一起原样发回。思考令牌按输出令牌计费，并且 `max_tokens` 涵盖思考和响应文本。请重新审视 `max_tokens`，并在您希望减少思考的地方降低 `effort`。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，Claude Fable 5.2 可以读取先前 Claude Opus 4.8 轮次中的思考块，因此迁移到 `claude-fable-5-2` 的对话会保留该推理。

2. **更低的提示缓存最小值：** Claude Fable 5.2 上的最小可缓存提示长度为 512 个令牌，低于 Claude Opus 4.8 上的 1,024 个令牌。以前因过短而无法缓存的提示现在可以创建缓存条目，无需更改代码。请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#cache-limitations)。

3. **不支持 Priority Tier：** Claude Opus 4.8 支持 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers#supported-models)。Claude Fable 5.2 不支持。如果您的组织有 Priority Tier 承诺，请单独规划容量。

4. **Advisor 工具的建议以加密形式返回：** 使用处于 beta 阶段的 [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)时，Claude Opus 4.8 执行器可以与以明文形式返回建议的顾问（例如 Claude Opus 4.8）配对。Claude Fable 5.2 执行器仅接受的顾问 Claude Fable 5.2 和 Claude Mythos 5.2 会以加密形式返回建议，即 `advisor_redacted_result` 块，因此您的客户端无法读取其文本。如果您的代码会读取建议文本，请移除该依赖。请参阅[结果变体](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#result-variants)。

如果您的代码使用的是 Claude Opus 4.7 或更早版本，还需应用 Claude Opus 5.5 指南迁移清单中的部分内容。从 [Claude Opus 4.7 或更早版本](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migration-checklist-from-claude-opus-47)这一组开始，依次应用其后的每一组，直到并包括提及您的模型的那一组为止。这些组涵盖了 Claude Opus 4.8 之前所做的请求变更，例如已移除的采样参数和 beta 标头。仅当[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出您要迁移到的模型时，才应用该清单中的快速模式一项。在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，结构化输出不适用于 Claude Fable 5.2，因此请勿在其中应用 `output_config.format` 一项。请改为从您的请求中删除 `output_format`，并在您的代码中验证 JSON。

### 迁移清单

* 完成 [Claude Opus 5 清单](https://platform.claude.com/docs/zh-CN/models/fable-5-2/migration-guide#migration-checklist-fable-5-2-from-opus-5)中的步骤，将 `claude-opus-4-8` 作为您要替换的模型 ID。
* 审查在没有 `thinking` 字段的情况下运行的工作负载。在 Claude Fable 5.2 上，它们会以思考方式运行，因此请重新审视 `max_tokens`，在您希望减少思考的地方降低 `effort`，并按 `type` 选择内容块。
* 检查任何解析 `thinking` 字段的代码是否仅将其视为显示文本。将 `thinking.display` 设置为 `"summarized"` 以接收可读的摘要。
* 审查接近缓存最小值的提示：512 个令牌或更多的提示可以创建缓存条目。
* 如果您的组织有 Priority Tier 承诺，请单独规划容量。
* 如果您使用 Advisor 工具且您的代码会读取建议文本，请移除该依赖：建议会以加密的 `advisor_redacted_result` 块形式到达。
* 在您自己的评估上运行全新的 effort 扫描，而不是沿用为 Claude Opus 4.8 调优的设置。
* 对于 Claude Opus 4.7 或更早版本，还需应用 Claude Opus 5.5 指南中适用于您的模型的清单组。除非[快速模式的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)列出您要迁移到的模型，否则请省略快速模式一项。在使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)的组织中，还需省略 `output_config.format` 一项，并从您的请求中删除 `output_format`。
