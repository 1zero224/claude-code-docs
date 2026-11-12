---
title: Claude Haiku 5.5 的新功能
url: https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5
description: 概述 Claude Haiku 5.5 中的新功能、破坏性变更和行为变更，并附有每项功能指南的链接以及代码变更迁移指南的链接。
---

Claude Haiku 5.5 专为高吞吐量、对延迟敏感的工作而构建，例如分类、路由、提取和子代理任务。它支持带有 effort 参数的自适应思考、1M 令牌的上下文窗口，以及最多 128k 输出令牌。它使用与 Claude 4.7 及更高版本模型相同的较新分词器，因此相同的文本计为的令牌数比 Claude Haiku 4.5 多约 30%。其思考块仅在生成它们的账户或与该账户关联的账户中有效。

有关代码变更，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide)。有关模型 ID、定价和限制，请参阅 [Claude Haiku 5.5 概述](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview)。有关提示指导，请参阅[为 Claude Haiku 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5)。

## 相对于 Claude Haiku 4.5 的变更摘要

每一行列出一项变更、该变更属于新增、变更还是破坏性变更，以及您的代码需要执行的操作。

| 变更                                                                                                                                   | 类型    | 所需操作                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------ | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [自适应思考和 effort](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5#adaptive-thinking-and-effort)            | 新增    | 可选：设置 `effort`，在响应质量与速度和成本之间进行权衡。                                                                                                              |
| [更大的上下文窗口和输出](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5#larger-context-window-and-output)          | 新增    | 无。现有的 `max_tokens` 值仍然有效，但思考令牌会计入其中。长提示按更高的价格计费；请参阅[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)。 |
| [浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)                                         | 新增    | 无。                                                                                                                                             |
| [安全分类器可能拒绝请求](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)                       | 新增    | 在您的客户端中处理 `stop_reason: "refusal"`。服务器端回退不可用。                                                                                                  |
| [手动扩展思考会返回错误](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#configure-thinking)                            | 破坏性变更 | 将 `budget_tokens` 替换为自适应思考。                                                                                                                    |
| [采样参数可能返回错误](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#remove-sampling-parameters)                     | 破坏性变更 | 省略 `temperature`、`top_p` 和 `top_k`。                                                                                                            |
| [助手消息预填充会返回错误](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#replace-assistant-prefill)                    | 破坏性变更 | 以用户轮次结束 `messages`。                                                                                                                            |
| [`computer_20250124` 会返回错误](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#computer-use-toolset)            | 破坏性变更 | 将其替换为 `computer_toolset_20260801`。在 Amazon Bedrock 上，`computer_20251124` 也可使用。                                                                 |
| [更改较早的轮次会使思考块失效](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#keep-earlier-turns-unchanged)               | 破坏性变更 | 如果您将思考块发送回去，请保持对话只追加不修改。                                                                                                                       |
| [响应可能以思考块开头](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5#responses-can-begin-with-thinking-blocks)   | 已更改   | 按 `type` 而非位置选择内容块。                                                                                                                            |
| [默认省略思考文本](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#configure-thinking)                               | 已更改   | 要接收摘要形式的思考内容，请将 `thinking.display` 设置为 `"summarized"`。                                                                                         |
| [相同文本计为更多令牌](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5#same-text-counts-as-more-tokens)            | 已更改   | 重新计算提示令牌数，并重新审视 `max_tokens` 和成本估算。                                                                                                            |
| [跨账户重放思考块](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5#replaying-thinking-blocks-across-accounts)    | 已更改   | 如果您通过不同的账户重放已存储的对话，请通过生成该对话的账户重放每个对话。                                                                                                          |
| [较早的思考块保留在上下文中](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5#earlier-thinking-blocks-stay-in-context) | 已更改   | 在多轮对话中预计会有更多输入令牌。要移除较旧的块，请使用[思考块清除](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#thinking-block-clearing)。         |
| [更短的提示可以被缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#cache-limitations)                              | 已更改   | 无。最小可缓存提示长度为 512 个令牌，低于 Claude Haiku 4.5 上的 4,096 个令牌，因此长度介于两者之间且设置了 `cache_control` 的提示会在 Claude Haiku 5.5 上被缓存。                              |
| [不再注入上下文感知标签](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows#context-awareness)                            | 已更改   | 要为模型提供智能体循环的明确预算，请使用[任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)（beta）。                                       |

## 新功能

### 自适应思考和 effort

借助 [自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)，Claude Haiku 5.5 会自行决定何时思考以及思考多少。自适应思考默认开启。虽然在 `high` effort 或更低级别下，您仍然可以使用 `thinking: {"type": "disabled"}` 关闭思考，但在响应质量与速度和成本之间进行权衡的更好方式是使用 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)。

### 更大的上下文窗口和输出

Claude Haiku 5.5 拥有 1M 令牌的[上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)，最多返回 128k 输出令牌，而 Claude Haiku 4.5 上分别为 200k 和 64k。现有的 `max_tokens` 值仍然有效，但思考令牌会计入 `max_tokens`，因此较小的限制可能会在 `thinking` 块之后、任何文本之前停止。请参阅[配置思考](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#configure-thinking)。长提示按更高价格计费；请参阅[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)。

## 行为变更

### 响应可能以思考块开头

自适应思考默认开启，因此即使请求中未提及思考，响应也可能以一个或多个 `thinking` 块开头。将第一个内容块读取为答案的代码需要按 `type` 字段来选择块。请参阅迁移指南中的[配置思考](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#configure-thinking)。

### 较早的思考块保留在上下文中

Claude Haiku 5.5 会在上下文中保留所有较早助手轮次的思考块，并且它们计为输入令牌。Claude Haiku 4.5 仅保留最新轮次的块。因此，多轮对话携带的输入令牌比仅由分词器变更所能解释的更多。要移除较旧的块，请使用[思考块清除](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#thinking-block-clearing)。请参阅[各模型的思考块保留](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-block-preservation-by-model)。

### 相同文本计为更多令牌

Claude Haiku 5.5 使用与 Claude 4.7 及更高版本模型相同的新版分词器。与所有使用此分词器的模型一样，相同的输入文本在 Claude Haiku 5.5 上产生的令牌比在 Claude Haiku 4.5 上多约 30%。确切的增幅取决于内容。请求和响应的结构不依赖于分词器，但您以令牌为单位衡量或预算的任何内容都会发生变化。请参阅迁移指南中的[重新计算令牌](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#recount-tokens)。

### 跨账户重放思考块

来自 Claude Haiku 5.5 的思考块仅在生成它们的账户或与该账户关联的账户中有效。只有当您存储对话并通过不同的账户重放它们时，这一点才有影响。请参阅[通过生成思考块的账户重放思考块](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#replay-thinking-blocks-through-the-producing-account)。
