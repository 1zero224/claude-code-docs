---
title: 管理工具上下文
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/manage-tool-context
description: 在工具搜索、程序化工具调用、提示缓存和上下文编辑之间进行选择，以管理上下文膨胀。
---

工具定义和累积的 `tool_result` 块会占用您的"context window"（上下文窗口）。拥有大量工具或多轮对话的长时间运行智能体可能会在任务完成之前耗尽可用上下文。有四种方法可以在流程的不同环节解决这一问题。

## 四种方法

每种方法针对不同的上下文压力来源。请选择与您的令牌消耗去向相匹配的方法。

| 方法      | 减少的内容                   | 适用场景                        | 了解更多                                                                                                         |
| ------- | ----------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 工具搜索    | 预先加载的工具定义               | 大量工具（10 个以上），其中大多数工具并非每轮都需要 | [工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)                  |
| 程序化工具调用 | `tool_result` 往返        | 可以作为单个脚本运行的工具调用链            | [程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)        |
| 提示缓存    | 重复工具定义的令牌成本             | 在大量请求中保持稳定的工具定义             | [结合提示缓存的工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching) |
| 上下文编辑   | 历史记录中的旧 `tool_result` 块 | 早期结果已不再相关的长对话               | [上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)                            |

### 工具搜索

“Tool search”（工具搜索）会将工具定义保留在上下文窗口之外，直到 Claude 请求它们为止。您无需预先将 50 个工具模式加载到上下文中，而是使用 `defer_loading: true` 标记它们，添加一个工具搜索工具，并让 Claude 按需发现它们。这以少量的 “latency”（延迟）（查找工具需要额外一轮）为代价，换取基线上下文使用量的大幅降低。

### 程序化工具调用

“Programmatic tool calling”（程序化工具调用）将一系列工具调用折叠为单个代码块，由 Claude 编写并由 Anthropic 的代码执行沙箱运行。Claude 无需进行五次 `tool_use` 和 `tool_result` 的往返，而是输出一个脚本，在沙箱内调用全部五个函数。中间结果永远不会进入对话历史。

### 提示缓存

“Prompt caching”（提示缓存）不会减少上下文中的令牌数量，但会降低您在后续请求中为这些令牌支付的费用。如果您的工具定义是稳定的，只需缓存一次，即可在数千个请求中重复使用缓存的前缀。当您的工具列表庞大但固定时，这是正确的选择。

### 上下文编辑

“Context editing”（上下文编辑）会在旧的 `tool_result` 块完成其用途后将其从对话历史中移除。一个长时间运行的智能体循环可能会产生数百个中间结果，这些结果在当时有用，但之后已不再需要。上下文编辑让您无需重新开始对话即可修剪它们。

## 组合使用多种方法

您可以组合使用这些方法，因为每种方法解决问题的不同部分。一个长时间运行的智能体可以使用工具搜索来减少加载的工具定义，使用提示缓存来分摊剩余定义的成本，并使用上下文编辑在对话增长时裁剪过时的结果。

对于高流量智能体，一个合理的起点是：

1. 从一开始就在工具定义上启用提示缓存。缓存写入比基础输入价格高出 25%，在第二次命中缓存的请求时即可收回成本。
2. 当您拥有 10 个或更多工具，或基线上下文使用量变得明显时，添加工具搜索。
3. 当单个对话开始变得足够长，以至于早期结果变得不再相关时，添加上下文编辑。
4. 如果您注意到存在可以作为单个批次运行的重复性小型工具调用链，请考虑使用程序化工具调用。

## 后续步骤

<CardGroup cols={2}>
  <Card title="工具搜索工具" icon="magnifying-glass" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool">
    按需加载工具定义，而非预先加载。
  </Card>

  <Card title="程序化工具调用" icon="code" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling">
    将工具调用链折叠为单个可执行脚本。
  </Card>

  <Card title="结合提示缓存的工具使用" icon="database" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching">
    跨请求缓存工具定义以降低令牌成本。
  </Card>

  <Card title="上下文编辑" icon="scissors" href="https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing">
    从长时间运行的对话中裁剪过时的工具结果。
  </Card>
</CardGroup>
