---
title: 工具使用的工作原理
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/how-tool-use-works
description: 了解工具使用循环、工具在何处运行，以及何时应使用工具而非纯文本。
---

本页解释工具使用背后的概念：工具在何处运行、代理循环如何工作，以及何时工具使用是正确的方法。如需实操指导，请从[构建使用工具的代理](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/build-a-tool-using-agent)教程或[定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)指南开始。

## 工具使用契约

工具使用是您的应用程序与模型之间的一种约定。您指定哪些操作可用，以及它们的输入和输出采用什么结构；Claude 决定何时以及如何调用它们。模型从不自行运行任何东西。它会发出一个结构化请求，由您的代码（或 Anthropic 的服务器）运行该操作，然后结果会流回对话中。

这个契约使模型的行为不像文本生成器，而更像您调用的一个函数。具有传统API经验的工程师可以像集成任何其他类型化接口一样集成工具使用：定义模式、处理回调、返回结果。不同之处在于，另一端的调用者是一个语言模型，它根据对话选择要调用哪个函数。

## 工具在何处运行

工具之间最主要的区别在于代码在何处运行。每个工具都属于以下三类之一，而类别决定了您的应用程序需要负责什么。

### 用户定义的工具（客户端执行）

您编写 schema、运行代码并返回结果。这是最常见的情况：绝大多数工具使用流量都是调用应用程序特定逻辑的[用户定义的工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)。

当 Claude 调用您的某个工具时，API 响应中会包含一个 `tool_use` 块，其中包括工具名称和一个 JSON 参数对象。您的应用程序提取这些参数，运行该操作（数据库查询、HTTP 调用、文件写入，或该工具所做的任何事情），并在下一个请求中通过 `tool_result` 块将输出发送回去。Claude 永远不会收到您的实现；它只会收到您提供的 schema 和您返回的结果。

### Anthropic模式工具（客户端执行）

对于少数常见操作（管理暂存记忆、运行shell命令、编辑文件、控制桌面或浏览器），Anthropic发布工具模式，而您的应用程序处理执行。此类别中的工具有[`memory`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)、[`bash`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)、[`text_editor`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)、[`computer`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和[`browser`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)。

执行模型与用户定义的工具相同：响应包含一个`tool_use`块，您的代码运行操作，然后您发回一个`tool_result`。使用Anthropic模式工具而不是定义您自己的等效工具的原因在于，这些模式是经过训练内置的。Claude已在数千条使用这些确切工具签名的成功轨迹上进行了优化，因此它调用它们比调用执行相同功能的自定义工具更可靠，并且能更优雅地从错误中恢复。该模式是模型已经预期的接口。

### 服务器执行的工具

对于 [`web_search`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)、[`web_fetch`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)、[`code_execution`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool) 和 [`tool_search`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)，由 Anthropic 运行代码。您在请求中启用该工具，服务器处理其余所有事情。您永远不需要为这些工具构造 `tool_result` 块。当某个轮次只调用[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)时，服务器端循环会运行该操作，并在响应到达您之前将输出反馈给模型，除非循环在完成前停止，最常见的原因是循环暂停。

您收到的响应包含`server_tool_use`块，显示运行了什么以及返回了什么。在常见情况下，当您看到它们时执行已经完成，您的应用程序的工作是启用工具并读取最终答案，而不是参与执行循环；主要的例外是暂停的循环（[`pause_turn`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/how-tool-use-works#the-server-side-loop)）和同时调用客户端工具的回合。

## 代理循环（客户端工具）

客户端执行的工具（包括用户定义的和Anthropic模式的）要求您的应用程序驱动一个循环。模型无法运行您的代码，因此每次工具调用都是一次往返：模型询问，您执行，您报告回来，模型继续。

典型的形式是一个以`stop_reason`为键的`while`循环：

1. 发送一个包含您的`tools`数组和用户消息的请求。
2. Claude以`stop_reason: "tool_use"`和一个或多个`tool_use`块响应。
3. 运行每个工具。将输出格式化为 `tool_result` 块。
4. 发送一个新请求，包含原始消息、助手的响应，以及一个带有`tool_result`块的用户消息。
5. 当`stop_reason`为`"tool_use"`时，从步骤2重复。

实际上，这可以理解为：当 `stop_reason == "tool_use"` 时，运行工具并继续对话。循环在遇到任何其他停止原因（`"end_turn"`、`"max_tokens"`、`"stop_sequence"` 或 `"refusal"`）时退出，这意味着 Claude 已经给出了最终答案，或者因其他原因停止，需要您的应用程序处理。

有关构建请求、处理并行工具调用和格式化结果的机制，请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。

## 服务器端循环

服务器执行的工具在Anthropic的基础设施内运行它们自己的循环。来自您应用程序的单个请求可能会在响应返回之前触发多次网络搜索或代码执行。模型搜索、读取结果、确定是否再次搜索，并迭代直到它获得所需内容，所有这些都无需您的应用程序参与。

这个内部循环有迭代次数限制。如果模型在达到上限时仍在迭代，响应将以 `stop_reason: "pause_turn"` 而不是 `"end_turn"` 返回。暂停的轮次意味着工作尚未完成；请重新发送对话（包括暂停的响应），让模型从中断处继续。有关继续执行的模式，请参阅[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)。

如果Claude在同一组并行工具调用中调用该服务器工具和一个客户端工具，循环也会在服务器工具运行之前将控制权交还给您。然后响应以`stop_reason: "tool_use"`和一个尚无结果块的`server_tool_use`块返回；API在您返回客户端工具结果后运行它。有关确切的契约，请参阅[停止原因和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#tool-use)。

## 何时使用工具（以及何时不使用）

当任务需要模型仅凭文本无法完成的事情时，工具使用是合适的：

* **具有副作用的操作。** 发送电子邮件、写入文件、更新记录。模型可以描述这些操作，但只有工具才能执行它们。
* **新鲜或外部数据。** 当前价格、今天的天气、数据库的内容。任何训练数据之外或特定于您系统的内容都需要工具来获取。
* **结构化、形式有保证的输出。** 当您需要一个具有特定字段的JSON对象，而不是恰好包含该信息的纯文本时，工具模式会强制执行该形式。
* **调用现有系统。** 数据库、内部API、文件系统。工具使用是自然语言请求与满足它们的系统之间的桥梁。

一个明确的迹象表明您应该使用工具：如果您正在编写正则表达式以从模型输出中提取决策，那么该决策本应是一次工具调用。解析自由格式文本以恢复结构化意图，表明该结构属于模式。

工具使用不适合的情况：

* 模型仅凭训练就能回答。摘要、翻译和常识问题不需要工具往返。
* 交互是没有副作用的一次性问答。如果没有需要运行的东西，工具就无事可做。
* 工具调用延迟会主导一个微不足道的响应。每次工具调用至少是一次额外的往返；对于轻量级任务，开销可能超过工作本身。

## 在方法之间进行选择

| 方法               | 何时使用                       | 预期效果                             | 了解更多                                                                                    |
| ---------------- | -------------------------- | -------------------------------- | --------------------------------------------------------------------------------------- |
| 用户定义的客户端工具       | 自定义业务逻辑、内部API、专有数据         | 您处理执行和代理循环                       | [定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)   |
| Anthropic模式客户端工具 | 标准开发操作（bash、文件编辑、桌面和浏览器控制） | 您处理执行；由于模式是经过训练内置的，Claude可靠地调用工具 | [工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference) |
| 服务器执行的工具         | 网络搜索、代码沙箱、网络获取             | Anthropic处理执行；您读取结果而不是产生它们       | [服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)  |

## 后续步骤

<CardGroup>
  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/build-a-tool-using-agent" title="教程：构建使用工具的代理">
    从单次工具调用到生产环境，逐步构建一个代理。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools" title="定义工具">
    模式规范、描述和`tool_choice`。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference" title="工具参考">
    Anthropic提供的工具目录。
  </Card>
</CardGroup>
