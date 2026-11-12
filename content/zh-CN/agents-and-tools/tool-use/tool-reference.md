---
title: 工具参考
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference
description: Anthropic 提供的服务器工具、客户端工具和客户端工具集目录，以及可选工具定义属性的参考。
---

本页面是 Anthropic 提供的工具以及您可以在任何工具定义上设置的可选属性的参考。有关工具使用的概念性介绍，请参阅 [Claude 的工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)。有关在应用程序中实现工具使用的指导，请参阅[定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)。

## Anthropic 提供的工具

Anthropic 提供两类工具：在 Anthropic 基础设施上运行的 **“server tools”（服务器工具）**，以及由 Anthropic 定义 schema、但由您的应用程序负责执行的 **“client tools”（客户端工具）**。这两类工具都与任何用户定义的工具一起出现在请求的 `tools` 数组中。

| 工具                                                                                              | `type`                                                                                             | 执行方 | [Beta 头](https://platform.claude.com/docs/zh-CN/api/beta-headers) |
| ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | --- | ----------------------------------------------------------------- |
| [网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)      | `web_search_20260318`<br />`web_search_20260209`<br />`web_search_20250305`                        | 服务器 | 无                                                                 |
| [Web fetch 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool) | `web_fetch_20260318`<br />`web_fetch_20260309`<br />`web_fetch_20260209`<br />`web_fetch_20250910` | 服务器 | 无                                                                 |
| [代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)  | `code_execution_20260521`<br />`code_execution_20260120`<br />`code_execution_20250825`            | 服务器 | 无                                                                 |
| [顾问工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)           | `advisor_20260301`                                                                                 | 服务器 | `advisor-tool-2026-03-01`                                         |
| [工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)     | `tool_search_tool_regex_20251119`<br />`tool_search_tool_bm25_20251119`                            | 服务器 | 无                                                                 |
| [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)                | `mcp_toolset`                                                                                      | 服务器 | `mcp-client-2025-11-20`                                           |
| [记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)            | `memory_20250818`                                                                                  | 客户端 | 无                                                                 |
| [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)           | `bash_20250124`                                                                                    | 客户端 | 无                                                                 |
| [文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)    | `text_editor_20250728`<br />`text_editor_20250124`                                                 | 客户端 | 无                                                                 |
| [计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)   | `computer_toolset_20260801`<br />`computer_20251124`<br />`computer_20250124`                      | 客户端 | 无<br />`computer-use-2025-11-24`<br />`computer-use-2025-01-24`   |
| [浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)    | `browser_toolset_20260801`                                                                         | 客户端 | 无                                                                 |

有关模型兼容性，请参阅各工具的页面。支持的模型因工具和工具版本而异。要在代码中检查某个模型是否接受网页搜索或代码执行，请参阅[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api)。

<Note>
  工具搜索的 `type` 值也接受不带日期的别名： `tool_search_tool_regex` 和 `tool_search_tool_bm25`。它们会解析为 最新的带日期版本。
</Note>

### 工具版本控制

大多数 Anthropic 提供的工具在 `type` 字符串中带有 `_YYYYMMDD` 后缀。当工具的行为、schema 或模型支持发生变化时，会发布新版本。旧版本仍然可用，以便现有集成继续正常工作。

当一个工具有多个活跃版本时，它们之间的关系各不相同：

* **按功能区分：** `web_search_20260209` 和 `web_fetch_20260209` 相比其前代版本增加了动态内容过滤；`web_fetch_20260309` 增加了绕过缓存的选项；`web_search_20260318` 和 `web_fetch_20260318` 增加了响应包含控制。`code_execution_20260120` 增加了从沙箱内部进行[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)的功能；`code_execution_20260521` 在工具描述中披露了每个单元的时间限制。在每种情况下，新版本和旧版本都是当前版本；使用哪一个取决于您是否需要新功能。
* **按模型区分：** `text_editor_20250728` 适用于 Claude 4 及更高版本的模型，`text_editor_20250124` 适用于更早的模型。您使用的版本取决于您的目标模型。
* **变体，而非版本：** `tool_search_tool_regex_20251119` 和 `tool_search_tool_bm25_20251119` 是同时发布的两种搜索算法。两者互不取代。
* **旧版：** `code_execution_20250522` 仅支持 Python。`code_execution_20250825` 增加了 Bash 和文件操作。
* **后继版本：** `computer_toolset_20260801` 是 beta 版 `computer_20251124` 和 `computer_20250124` 的稳定后继版本，后两者在[较早的工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#earlier-tool-versions)中为其列出的模型上可用。`browser_toolset_20260801` 是浏览器使用工具的第一个版本。两者都是[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)。

`mcp_toolset` 类型不按日期进行版本控制；其版本信息改由 `anthropic-beta` 头承载。

### 客户端工具集

[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)是 Anthropic 定义的 “client toolsets”（客户端工具集）：`tools` 中的一个条目声明一组固定的成员工具，其名称、描述和输入 schema 由 Anthropic 定义，而每次调用都由您的应用程序执行。该条目不接受 `name`，因为带日期的 `type` 已固定了成员名称。`configs`、`cache_control` 和 `allowed_callers`（仅接受 `["direct"]`）是可选的。

客户端工具集是 Messages API 工具。它们目前不能作为 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/tools) 中的智能体工具使用，后者提供自己的内置智能体工具集、MCP 工具集和自定义工具。

```json
{
  "type": "browser_toolset_20260801",
  "configs": {
    "javascript_exec": { "enabled": true }
  },
  "cache_control": { "type": "ephemeral" }
}
```

`configs` 用于调整单个成员：

* 键是成员名称，每个值仅接受 `enabled` 和 `defer_loading`。
* 您省略的成员保持其默认值。缺失的值、`{}` 和重述的默认值是等效的。
* 未知的成员名称或成员值中的任何其他字段都会被拒绝，禁用所有成员的 `configs` 也会被拒绝（请改为省略该条目）。
* 被禁用的成员会从 Claude 接收的工具中移除。如果 Claude 仍然指名调用它，请返回一个错误 `tool_result`。

请为每个成员设置 `defer_loading`，切勿在条目上设置，并为每个启用的成员赋予相同的值：在[工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#deferred-tool-loading)下，工具集作为一个定义进行加载和展开。当所有启用的成员都延迟加载时，只有本身未被延迟的[工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)才能呈现该工具集，因此请在同一请求中声明一个。不要在成员延迟加载的工具集条目上设置 `cache_control`；请改为在非延迟工具上设置断点，因为延迟加载的定义不属于缓存前缀的一部分。

`cache_control` 只能放在条目上；要了解断点落在何处（包括批量操作内部的标记），请参阅[工具使用与提示缓存](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching#cache-control-on-tool-definitions)。

**处理成员工具调用。** Claude 通过 `tool_use` 块调用成员，该块的 `name` 为成员名称，`toolset_name` 为 `computer` 或 `browser`；`input` 包含该成员的参数，且没有 `action` 字段。请根据 `toolset_name` 和 `name` 的组合进行分派，因为自定义工具可能与某个成员同名，并且两个工具集共享诸如 `screenshot` 之类的名称。只有成员结果会回显 `toolset_name`。一轮中的多个成员调用构成一个批量操作，您需要按顺序运行（[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)、[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)）。新成员只会随新的带日期 `type` 一起推出。

**工具集条目不支持的内容。** API 会以 `invalid_request_error` 拒绝以下每一项：

* `strict: true` 或 `input_examples`。
* 条目上的 `defer_loading`，或启用的成员的 `defer_loading` 值不同（请在 `configs` 中为每个成员设置，且全部设为相同的值）。
* `allowed_callers` 中的代码执行调用方（不支持[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)）。
* `fine-grained-tool-streaming-2025-05-14` beta 头。当您进行流式传输时，每个成员的 `input` 以一个完整的 `input_json_delta` 到达。
* 指定工具集或某个成员的 `tool` 类型 `tool_choice`（请使用 `auto`、`any` 或 `none`）。
* 同一工具集的两个条目，或带有该工具集名称的其他工具：与 `computer_toolset_20260801` 并存的名为 `computer` 的工具，或与 `browser_toolset_20260801` 并存的名为 `browser` 的工具。这两个工具集可以一起声明。

## 工具定义属性

`tools` 数组中的每个工具（包括用户定义的工具）都接受可选属性，用于控制工具的加载方式、谁可以调用它以及如何验证其输入。您可以组合使用这些属性，但有两个例外：设置了 `defer_loading: true` 的工具不能同时携带 `cache_control`，设置了 `strict: true` 的用户定义工具不能在 `allowed_callers` 中列出代码执行调用方。

| 属性                      | 用途                                             | 适用范围                                                                                                                                                                                                                                                                                           | 详细指南                                                                                                                            |
| ----------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `cache_control`         | 在此工具定义处设置提示缓存断点                                | 所有工具（对于 `computer_toolset_20260801` 和 `browser_toolset_20260801`，请在工具集条目本身上设置，而不是在成员 `configs` 中设置）                                                                                                                                                                                            | [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)                                                 |
| `strict`                | 保证对工具名称和输入进行 schema 验证                         | 除 `mcp_toolset`、`computer_toolset_20260801` 和 `browser_toolset_20260801` 之外的所有工具                                                                                                                                                                                                               | [严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)                                      |
| `defer_loading`         | 将工具从初始系统提示中排除；当工具搜索为其返回 `tool_reference` 时按需加载 | 所有工具（对于 `mcp_toolset`，请参阅 [MCP 工具集配置](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#mcp-toolset-configuration)）。在计算机使用和浏览器使用工具集上，请在 `configs` 中为每个成员单独设置；请参阅[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)。 | [工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)                                     |
| `allowed_callers`       | 限制哪些调用方可以调用该工具                                 | 除 `mcp_toolset` 之外的所有工具（在 `computer_toolset_20260801` 和 `browser_toolset_20260801` 上仅接受 `["direct"]`；请参阅[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)）                                                                             | [程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#the-allowed-callers-field) |
| `input_examples`        | 提供示例输入对象，向 Claude 展示如何调用该工具                    | 用户定义的工具和 Anthropic schema 客户端工具，`computer_toolset_20260801` 和 `browser_toolset_20260801` 除外。不适用于服务器工具。                                                                                                                                                                                         | [定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#providing-tool-use-examples)               |
| `eager_input_streaming` | 为此工具启用细粒度输入流式传输（`true`）或保持标准缓冲流式传输（`false`）    | 仅限用户定义的工具                                                                                                                                                                                                                                                                                      | [细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming)                       |

### `allowed_callers` 值

`allowed_callers` 是一个数组，接受以下值的任意组合：

| 值                           | 含义                                                       |
| --------------------------- | -------------------------------------------------------- |
| `"direct"`                  | 模型可以在 `tool_use` 块中直接调用此工具。如果省略 `allowed_callers`，这是默认值。 |
| `"code_execution_20260120"` | 在 `code_execution_20260120` 或更高版本沙箱内运行的代码可以调用此工具。        |

`"code_execution_20260120"` 和 `"code_execution_20260521"` 在 `allowed_callers` 中均被接受且可互换：使用任一代码执行工具版本的请求都能满足列出任一调用方的工具。无论请求声明的是哪个版本，响应块始终将调用方标记为 `code_execution_20260120`。

从数组中省略 `"direct"`（例如 `"allowed_callers": ["code_execution_20260120"]`）会引导 Claude 仅从代码执行内部调用该工具。响应的 `tool_use` 块包含一个 `caller` 字段，用于标识是哪个调用方调用了该工具。有关完整说明（包括 `caller` 响应结构和错误行为），请参阅[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#the-allowed-callers-field)。

### `defer_loading` 与提示缓存

`defer_loading: true` 会保留您的提示缓存：工具搜索可以在对话期间加载延迟加载的工具，而不会使现有缓存条目失效。请将 `cache_control` 断点设置在非延迟工具上。要了解延迟加载的工具如何使缓存保持有效，请参阅[`defer_loading` 与缓存保留](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching#defer-loading-and-cache-preservation)。
