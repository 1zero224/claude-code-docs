---
title: 功能概览
url: https://platform.claude.com/docs/zh-CN/build-with-claude/overview
description: 探索 Claude 的高级功能和能力。
---

有关每项功能在"zero data retention"（零数据保留），即 ZDR 下的适用性以及 HIPAA 就绪情况，请参阅 [API 和数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)。

有关管理和治理，请参阅 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)、[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 和 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)。

## 平台功能

Claude Platform 中可跨其各个 API 使用的功能。每项功能的页面都会说明您可以在哪里使用它。

<Tip>
  您可以通过编程方式了解某个模型支持哪些能力。[Models API](https://platform.claude.com/docs/zh-CN/api/models/list) 会为每个可用模型返回 `max_input_tokens`、`max_tokens` 以及一个 `capabilities` 对象。
</Tip>

| 功能                                                                                                                                     | 描述                                                                                                 |
| :------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)                                            | 让速度更快的执行模型在生成过程中咨询能力更强的顾问模型，以获取计划或纠正方向。                                                            |
| [Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview)                                          | 由指令、元数据和可选脚本组成的可复用包，当它们与任务相关时，Claude 会自动使用。                                                        |
| ↳ [自定义技能](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide#managing-custom-skills)                                | 您编写并上传到工作区的技能。                                                                                     |
| ↳ [预构建 Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview#pre-built-agent-skills)             | Anthropic 为 PowerPoint、Excel、Word 和 PDF 文件预构建的技能。                                                  |
| [批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)                                                       | 以标准价格的一半，在一个批次中异步处理大量 Messages API 请求。                                                             |
| [浏览器和计算机使用 SDK 工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk)                                  | Python 和 TypeScript SDK 中的类，用于将浏览器使用或计算机使用工具连接到您提供的浏览器或桌面。                                         |
| [浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)                                           | 让 Claude 在您自己的浏览器环境中导航、读取网页并与之交互。                                                                  |
| [引用](https://platform.claude.com/docs/zh-CN/build-with-claude/citations)                                                               | Claude 关于您文档的回答会引用支持每个论点的确切段落。                                                                     |
| [分类器拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)                                                | 某些模型上的安全分类器可能会拒绝请求；拒绝以正常响应返回，而不是错误。                                                                |
| ↳ [客户端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)                         | Anthropic SDK 中的一个中间件，使用您列出的回退模型重试被拒绝的请求。                                                          |
| ↳ [回退抵扣](https://platform.claude.com/docs/zh-CN/build-with-claude/fallback-credit)                                                     | 对在另一个模型上重试被拒绝请求的计费方式，视同对话从一开始就在该模型上进行，因此您无需为提示缓存写入支付两次费用。                                          |
| ↳ [服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)                        | API 在同一次调用中使用回退模型重试被拒绝的请求，并返回一个响应。                                                                 |
| [计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)                                          | 让 Claude 通过截图、鼠标和键盘控制桌面环境。                                                                         |
| [自定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)                                                 | 您使用名称、描述和输入 schema 定义的工具。Claude 请求调用它们；您的代码运行它们并返回结果。                                              |
| [数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)                                                            | 用于控制模型推理的运行位置（推理地域）以及静态数据的存储位置（工作区地域）的设置。                                                          |
| [Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)                                                              | 一项设置，用于控制 Claude 在响应上花费多少令牌，在周全程度与令牌效率之间进行权衡。                                                      |
| ↳ [按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)                                          | 通过按消息设置的 `output_config` 更改对话后续轮次的 effort 级别，同时保留提示缓存。                                             |
| [快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)                                                             | 受支持模型的每秒输出令牌数最多可达 2.5 倍，按高级定价计费。                                                                   |
| [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)                                                            | 上传文件一次后，即可在后续请求中通过 `file_id` 引用，还可下载技能或代码执行工具创建的文件。                                                |
| [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)                                                       | 将 Claude 连接到远程 MCP 服务器，使其能够调用这些服务器的工具，而无需运行您自己的 MCP 客户端。                                           |
| ↳ [客户端 MCP 辅助工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#client-side-mcp-helpers)                        | 在 MCP 类型与 Claude API 类型之间进行转换的 SDK 辅助工具，适用于您运行自己的 MCP 客户端的情况，例如用于本地服务器、提示或资源。                      |
| [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)                                                 | 通过仅出站连接将 Claude 连接到您私有网络内的 MCP 服务器，无需开放任何入站端口。                                                     |
| [对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)                                 | 在对话中途添加的系统指令，不会更改顶层系统提示或其之前的缓存前缀。                                                                  |
| ↳ [对话中途的工具变更](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#mid-conversation-tool-changes) | 在对话中途提供或撤回工具，而无需更改 tools 数组，从而使缓存前缀保持有效。                                                           |
| ↳ [轮次范围的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#turn-scoped-system-messages)   | 一种对话中途的系统消息，仅在下一条用户消息之前生效，之后仍保留在历史中，但不呈现任何内容，也不消耗输入令牌。                                             |
| [多语言支持](https://platform.claude.com/docs/zh-CN/build-with-claude/multilingual-support)                                                 | Claude 可使用多种语言工作，相对于英语表现出色。                                                                        |
| [OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry)                                                    | 使用来自 Anthropic SDK 和 Claude API 的 OpenTelemetry span 追踪您的 Messages API 请求、工具使用和 Managed Agents 会话。 |
| ↳ [API 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api)                                                     | Claude API 针对其运行的每个模型请求和工具使用，发送到您的可观测性平台的 OpenTelemetry span。                                      |
| ↳ [SDK 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)                                                     | Anthropic SDK 在您的应用中针对其发送的每个请求以及其工具运行器运行的每个工具发出的 OpenTelemetry span。                               |
| [并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)                                           | Claude 可以在一次响应中调用多个工具。                                                                             |
| [PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)                                                         | 将 PDF 发送给 Claude，使其能够读取其中的文本、图片、图表和表格。                                                             |
| [编程式工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)                                  | 让 Claude 在代码执行容器中通过代码调用您的工具，而无需每次调用都进行一次模型往返。                                                      |
| [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)                                                        | 重用近期请求中已缓存的提示前缀，以减少处理时间和成本。                                                                        |
| ↳ [1 小时有效期](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#1-hour-cache-duration)                             | 1 小时的提示缓存有效期，缓存写入价格高于默认的 5 分钟有效期。                                                                  |
| ↳ [5 分钟有效期（默认）](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#how-prompt-caching-works)                      | 默认的提示缓存有效期。每次使用缓存内容都会免费刷新其有效期。                                                                     |
| ↳ [自动缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)                                    | 使用单个顶层 `cache_control` 字段的提示缓存：缓存断点位于最后一个可缓存的块上，并随对话增长向前移动。                                        |
| ↳ [缓存诊断](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)                                                   | 将请求与您的上一个请求进行比较，并报告提示前缀发生变化的位置，以解释缓存未命中。                                                           |
| ↳ [显式缓存断点](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#explicit-cache-breakpoints)                         | 在各个内容块上使用 `cache_control` 的提示缓存，可精确选择要缓存的内容。                                                       |
| [SDK 中间件](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/middleware)                                                        | Anthropic SDK 中的一个钩子，在每个请求发送之前以及收到其响应之后运行您的代码。                                                     |
| [搜索结果](https://platform.claude.com/docs/zh-CN/build-with-claude/search-results)                                                        | 包含您自己的搜索结果（附带来源和标题）的内容块，使 Claude 能够像引用网页搜索结果一样引用它们。                                                |
| [服务层级](https://platform.claude.com/docs/zh-CN/api/service-tiers)                                                                       | 让请求使用您组织的 Priority Tier 容量。                                                                        |
| [会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)                                                                  | 对单个会话设置的以美元计的硬性支出上限，按公开标价计算。                                                                       |
| [流式拒绝](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/handle-streaming-refusals)                       | 当流式分类器因可能的政策违规而中止流式响应时，该响应以 `stop_reason` `"refusal"` 结束。                                          |
| [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)                                                   | 将 Claude 的响应约束为您提供的 JSON schema，使输出有效且可解析。                                                         |
| [任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)                                                          | 告知 Claude 完整智能体循环可用的令牌数，使其能够规划工作并在预算用完之前完成。                                                        |
| [思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)                                                                | Claude 在写出答案之前，先在思考块中逐步解决问题。                                                                       |
| ↳ [自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost)                                         | 一种思考模式，由 Claude 针对每个请求决定是否思考以及思考多少。                                                                |
| ↳ [扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)                                                   | 一种手动思考模式，您在每个请求上设置固定的思考令牌预算（`budget_tokens`）。                                                      |
| ↳ [保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)                                                  | 仅当当前模型能够读取先前轮次中的思考块，且它们之前的内容（系统提示、工具、先前的消息）均未更改时，Claude 才会使用这些思考块。                                 |
| [阈值压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)                                                  | 自动压缩：当对话达到令牌阈值时，API 在请求内对较早的上下文进行摘要。                                                               |
| [令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)                                                        | 在将消息发送给 Claude 之前计算其中的令牌数。                                                                         |
| [工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)                                                  | 一个 SDK 辅助工具，为您运行工具使用循环：在 Claude 调用您的工具时运行它们，并将结果发回。                                                |
| [工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)                                            | 让 Claude 搜索您的工具目录，并仅加载其所需的工具定义。                                                                    |
| [视觉（图像）](https://platform.claude.com/docs/zh-CN/build-with-claude/vision)                                                              | Claude 可以读取和分析您发送给它的图像。                                                                            |
| [网页域名限制](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)                                                 | 允许或阻止的域名列表，用于控制网页搜索和 Web fetch 工具可以访问哪些网站。                                                         |
| [Web fetch 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)                                        | 一种服务器工具，让 Claude 可以获取指定 URL 处网页和 PDF 文档的完整内容。                                                      |
| [网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)                                             | 一种服务器工具，让 Claude 可以在网络上搜索最新信息。                                                                     |

## Messages API 功能

| 功能                                                                                                                      | 描述                                                          |
| :---------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------- |
| [Apple Foundation Models](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/apple-foundation-models)  | 一个 Swift 包，使 Claude 可作为 Apple Foundation Models 框架中的语言模型使用。 |
| [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)                                   | 让 Claude 请求执行 shell 命令，这些命令在一个于命令之间保持状态的 bash 会话中运行。        |
| [代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)                          | 让 Claude 在沙箱容器中运行 Python 和 Bash 代码。                         |
| [上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)                                       | 随着对话历史增长，从中清除选定的内容，例如旧的工具结果或旧的思考块。                          |
| [强制工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#forcing-tool-use)                | 让 Claude 调用某个工具或某个特定工具，而不是直接回答。                             |
| [记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)                                    | 让 Claude 在记忆文件目录中跨对话存储和读取信息。                                |
| ↳ [记忆工具辅助工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool#implement-the-memory-handler) | 处理记忆工具接口和工具使用循环的 SDK 辅助工具，您的代码只需提供存储。                       |
| [消息流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)                                            | 在生成 Messages API 响应的同时，以服务器发送事件的形式增量接收该响应。                  |
| ↳ [细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming)             | 在 Claude 编写工具输入时将其流式传输到您的客户端，服务器不会先缓冲并检查 JSON。              |
| [消息线程](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads)                                        | 每个请求仅发送新消息，使长时间运行的智能体更快获得首个令牌，在长对话中尤为明显。                    |
| [按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)                                   | 在您的应用请求时，用 Claude 编写的摘要替换对话中较早的消息。                          |
| ↳ [后台压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-background)                                | 在对话基于完整历史继续进行的同时运行按需压缩请求，并在摘要返回后将其替换进来。                     |
| ↳ [压缩与保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-thinking-blocks)                        | 在支持保留思考的模型上，按需压缩后保留的轮次中的思考块在规定条件下可以保持有效。                    |
| ↳ [保留尾部压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-keep-recent-turns)                       | 一种按需压缩，对较早的轮次进行摘要，并在摘要之后逐字保留最后几轮。                           |
| ↳ [工具运行器压缩辅助方法](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand#compact-in-a-loop)        | SDK 工具运行器上的一个方法，在当前轮次及其工具调用完成后为您发送压缩请求。                     |
| [OpenAI SDK 兼容性](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk)                        | 一个兼容层，让您可以使用 OpenAI SDK 调用 Claude API，用于测试和比较模型。            |
| [严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)                              | 使 Claude 的工具输入始终符合该工具的 JSON Schema。                         |
| [文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)                            | 让 Claude 使用 Anthropic 定义的工具查看、创建和编辑文本文件。                    |
| [类型化结构化输出辅助工具](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#usage)                       | SDK 辅助工具，根据您代码中的类型构建 JSON schema，并将 Claude 的响应解析为该类型。       |

## Managed Agents 功能

| 功能                                                                                                  | 描述                                                                 |
| :-------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------- |
| [智能体配置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)                          | 可复用、带版本的配置，包含会话运行的模型、系统提示、工具、MCP 服务器和技能。                           |
| [智能体工具集](https://platform.claude.com/docs/zh-CN/managed-agents/tools)                               | 智能体在其沙箱中使用的一组内置工具（bash、read、write、edit、glob、grep、Web fetch 和网页搜索）。 |
| [云沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/environments)                           | 位于 Anthropic 管理的基础设施上的隔离 Linux 容器，会话的工具和代码在其中运行。                   |
| [Console 智能体构建器](https://platform.claude.com/docs/zh-CN/managed-agents/onboarding)                  | Claude Console 中用于创建、配置和测试智能体的可视化界面。                               |
| [梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)                                  | 一个作业，读取记忆存储和过去的会话记录，并写入一个经过重新组织的新记忆存储。                             |
| [事件增量](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas)                          | 在模型仍在生成时，于会话事件流上实时预览智能体的响应文本。                                      |
| [GitHub 访问](https://platform.claude.com/docs/zh-CN/managed-agents/github)                           | 从会话访问 GitHub：挂载在沙箱中的代码仓库，以及用于拉取请求的 GitHub MCP 服务器。                 |
| [记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)                                | 智能体读取和写入的文本文档集合，使信息能够跨会话保留。                                        |
| [多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)            | 让会话的智能体将工作交给其他智能体（作为子智能体或在工作流中），并咨询顾问模型。                           |
| ↳ [动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)                      | 由智能体编写的程序，用于在后台运行多个智能体并合并它们的结果。                                    |
| ↳ [会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)                     | 多智能体会话中每个智能体工作所在的线程，您可以列出、流式传输、中断和归档这些线程。                          |
| [结果](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)                         | 会话的目标结果，附带一份评分标准，由独立的评分器据此检查智能体的工作，直到达成目标。                         |
| [权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)                   | 用于控制智能体的内置工具和 MCP 工具是自动运行、等待您的批准，还是在每次调用时由服务器检查的设置。                |
| [计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)                 | 按 cron 计划运行智能体，使智能体自行启动会话。                                         |
| [自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)                | 位于您自己基础设施上的沙箱，用于运行智能体的工具调用，而 Claude 仍在 Anthropic 一侧运行。             |
| ↳ [自托管 worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers) | 您在自己的基础设施上运行的进程，用于从自托管环境的队列中认领会话并运行其工具调用。                          |
| [会话查看器](https://platform.claude.com/docs/zh-CN/managed-agents/session-observability)                | Claude Console 中的会话查看器以及每个会话的用量汇总，用于查看智能体做了什么以及用了多少。               |
| [保管库](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)                                 | 存放各用户第三方服务凭据的存储，您只需注册一次，并在创建会话时通过 ID 引用。                           |
| [Webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks)                           | 当您的 Managed Agents 资源中发生事件（例如会话状态变化）时，向您的 HTTPS 端点发送的通知。           |
