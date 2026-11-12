---
title: 结合提示缓存的工具使用
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching
description: 跨轮次缓存工具定义，并了解哪些操作会使您的缓存失效。
---

本页介绍针对工具定义的 “prompt caching”（提示缓存）：在何处放置 `cache_control` 断点、`defer_loading` 如何保留您的缓存，以及哪些操作会使缓存失效。有关通用的提示缓存，请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)。

## 工具定义上的 cache\_control

将 `cache_control: {"type": "ephemeral"}` 放置在 `tools` 数组中的最后一个工具上。这会缓存整个工具定义前缀，从第一个工具一直到标记的断点：

```json
{
  "tools": [
    {
      "name": "get_weather",
      "description": "Get the current weather in a given location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": { "type": "string" }
        },
        "required": ["location"]
      }
    },
    {
      "name": "get_time",
      "description": "Get the current time in a given time zone",
      "input_schema": {
        "type": "object",
        "properties": {
          "timezone": { "type": "string" }
        },
        "required": ["timezone"]
      },
      "cache_control": { "type": "ephemeral" }
    }
  ]
}
```

对于 `mcp_toolset`，`cache_control` 断点会落在该工具集中的最后一个工具上。您无法控制 MCP 工具集内部的工具顺序，因此请将断点放置在 `mcp_toolset` 条目本身上，API 会将其应用到最终展开的工具上。

[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)工具集条目遵循相同的规则：将 `cache_control` 放在工具集条目本身上，断点会落在该工具集的定义之后。它不能放在成员的 `configs` 条目中，因为工具集的成员是作为一个定义加载的。在[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)中，该轮次任何成员 `tool_use` 或 `tool_result` 块上的 `cache_control` 标记都会被接受，并在该批次结束时生效，因此同一批次中的多个标记会作为单个断点起作用。每个标记仍会计入请求的[四个断点](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#when-to-use-multiple-breakpoints)限制，因此每个轮次最多使用一个。

## defer\_loading 与缓存保留

延迟加载的工具不会包含在系统提示前缀中。当模型通过[工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)发现某个延迟加载的工具时，其定义会作为 `tool_reference` 块内联追加到对话历史中。前缀保持不变，因此提示缓存得以保留。

这意味着通过工具搜索动态添加工具不会破坏您的缓存。您可以用一小组始终加载的工具（已缓存）开始对话，让模型按需发现其他工具，并在每一轮中保持相同的缓存命中。

`defer_loading` 的作用也独立于[严格模式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)的语法构建。无论哪些工具被延迟加载，语法都会基于完整的工具列表构建，因此在工具动态加载时，提示缓存和语法缓存都会得到保留。

## 哪些操作会使您的缓存失效

缓存遵循前缀层级结构（`tools` → `system` → `messages`），因此某一层级的更改会使该层级及其之后的所有内容失效：

| 更改                             | 失效范围                                                                                                                                                                              |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 修改工具定义                         | 整个缓存（tools、system、messages）                                                                                                                                                       |
| 开启或关闭网页搜索或引用                   | system 和 messages 缓存                                                                                                                                                              |
| 更改 `tool_choice`               | messages 缓存                                                                                                                                                                       |
| 更改 `disable_parallel_tool_use` | messages 缓存                                                                                                                                                                       |
| 切换图像的有/无                       | messages 缓存                                                                                                                                                                       |
| 更改思考参数                         | messages 缓存始终失效；在将思考配置渲染在 tools 和 system 之前的模型上，tools 和 system 缓存也会失效（参见[思考与提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-prompt-caching)） |
| 更改 `output_config.effort`      | 与思考参数相同；显式设置模型的默认值等同于省略该参数                                                                                                                                                        |

<Note>
  如果您需要在对话中途更改 `tool_choice`，请考虑将缓存断点放置在变化点之前。
</Note>

## 服务器工具结果会自动缓存

当您的请求启用了提示缓存，且 Claude 使用了[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)（例如网页搜索、网页抓取或代码执行）时，API 会在运行智能体循环的下一次迭代之前，自动在服务器工具结果上放置一个缓存断点。这使得同一请求中的后续迭代可以从缓存中读取不断增长的前缀，而无需重新处理。

此自动断点始终使用默认的 5 分钟 TTL，与您在自己的 `cache_control` 标记上设置的任何 TTL 无关。在响应的 `usage` 中，这些写入显示在 `cache_creation.ephemeral_5m_input_tokens` 下，因此即使您设置的每个 `cache_control` 都使用 1 小时 TTL，您也可能会看到 5 分钟的缓存写入。

此行为仅在您的请求已包含至少一个 `cache_control` 标记时适用。未启用提示缓存的请求不会获得自动断点。

## 各工具交互表

| 工具                                                                                           | 缓存注意事项                                                                                                                                                                                                     |
| -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [网页搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)     | 启用或禁用会使 system 和 messages 缓存失效                                                                                                                                                                             |
| [网页抓取](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)      | 启用或禁用会使 system 和 messages 缓存失效                                                                                                                                                                             |
| [代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool) | 容器状态独立于提示缓存                                                                                                                                                                                                |
| [工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)    | 发现的工具以 `tool_reference` 块加载，保留前缀缓存                                                                                                                                                                         |
| [计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)  | 截图的有无会影响 messages 缓存；`cache_control` 应放在工具集条目上（参见[工具定义上的 cache\_control](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching#cache-control-on-tool-definitions)） |
| [浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)   | 截图的有无会影响 messages 缓存；`cache_control` 应放在工具集条目上（参见[工具定义上的 cache\_control](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching#cache-control-on-tool-definitions)） |
| [文本编辑器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)   | 标准客户端工具，无特殊缓存交互                                                                                                                                                                                            |
| [Bash](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)           | 标准客户端工具，无特殊缓存交互                                                                                                                                                                                            |
| [记忆](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)           | 标准客户端工具，无特殊缓存交互                                                                                                                                                                                            |

## 后续步骤

<CardGroup cols={3}>
  <Card title="提示缓存" icon="database" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching">
    了解完整的提示缓存模型，包括 TTL 和定价。
  </Card>

  <Card title="工具搜索工具" icon="magnifying-glass" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool">
    按需加载工具而不破坏您的缓存。
  </Card>

  <Card title="工具参考" icon="book" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    浏览所有可用工具及其参数。
  </Card>
</CardGroup>
