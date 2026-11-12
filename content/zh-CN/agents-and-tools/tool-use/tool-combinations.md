---
title: 工具组合
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-combinations
description: 适用于研究智能体、编码智能体和长时间运行智能体的常见 Anthropic 工具搭配。
---

Anthropic 提供的工具旨在协同工作。常见的智能体模式会将覆盖工作流互补阶段的工具搭配使用：一个工具负责收集或发现，另一个工具负责处理或执行操作。以下组合只是起点，而非硬性规定。您可以根据任务需要自由混合搭配。

每个代码片段仅展示 `tools` 数组。完整的请求结构请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。

## 研究智能体：web\_search + code\_execution

搜索负责查找来源；代码执行负责分析和综合。Claude 先搜索数据，然后编写 Python 代码对其进行处理、制表或可视化。这种搭配非常适合既需要最新信息、又需要对这些信息进行非平凡计算的问题，例如"比较本季度前五大云服务提供商的收益"。

```json
{
  "tools": [
    { "type": "web_search_20260209", "name": "web_search" },
    { "type": "code_execution_20260521", "name": "code_execution" }
  ]
}
```

典型流程是先搜索，然后运行代码，如果第一轮发现了信息缺口，还可以选择再次搜索。代码执行在服务器端运行，因此无需管理客户端沙箱。

## 编码智能体：text\_editor + bash

文本编辑器负责读取和修改文件；bash 负责运行测试和构建命令。这是经典的软件开发循环：检查代码、进行编辑、运行测试、重复。这两个工具都在客户端执行，因此由您的应用程序控制可访问哪些文件和命令。

```json
{
  "tools": [
    { "type": "text_editor_20250728", "name": "str_replace_based_edit_tool" },
    { "type": "bash_20250124", "name": "bash" }
  ]
}
```

如果智能体操作的是不受信任的代码，请将其与受限的工作目录和命令允许列表搭配使用。有关执行约定，请参阅[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)和 [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)。

## 先引用后抓取：web\_search + web\_fetch

搜索负责找出候选 URL；抓取负责获取相关 URL 的完整页面内容。这样可以避免预先抓取所有内容。Claude 先运行搜索，检查摘要片段，挑选出两三个看起来真正相关的结果，然后只抓取这些结果。

```json
{
  "tools": [
    { "type": "web_search_20260209", "name": "web_search" },
    { "type": "web_fetch_20260209", "name": "web_fetch" }
  ]
}
```

当答案存在于搜索摘要无法完整呈现的长篇内容（文档页面、文章、规范）中时，这种搭配非常有用。抓取会拉取完整页面，以便 Claude 能够引用具体段落。

## 长时间运行智能体：memory + 任意其他工具

记忆负责跨对话持久化状态；其他工具负责完成实际工作。对于任何需要记住先前会话的智能体，都可以添加记忆功能，例如能够回忆客户早先问题的支持智能体，或跟踪上周所做决策的项目助手。

```json
{
  "tools": [{ "type": "memory_20250818", "name": "memory" }]
}
```

将您的其他工具与 `memory` 一起添加到同一数组中。

记忆与您的其他工具是正交的。它不会改变其他工具的行为；它为 Claude 提供了一个记录并在之后检索事实的地方，否则这些事实会在 “context window”（上下文窗口）重置时丢失。有关存储模型，请参阅[记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)。

## 一体化：computer\_use

计算机使用工具通过操作完整的桌面环境，涵盖了大多数其他工具的功能。Claude 接收屏幕截图并发出鼠标和键盘操作，这意味着它可以操作人类能够操作的任何应用程序。当任务需要更具体的工具无法实现的任意 GUI 交互时，请使用此工具：例如没有 API 的旧版软件、视觉验证步骤，或跨越多个桌面应用程序的工作流。

```json
{
  "tools": [{ "type": "computer_toolset_20260801" }]
}
```

该工具集条目不接受 `name` 或显示尺寸参数：坐标以您返回的屏幕截图的像素空间表示，您可以通过条目的 `configs` 字段关闭单个操作。

计算机使用是最通用的选项，同时也是最慢的，因为 Claude 通常在每批操作之后都需要一张新的屏幕截图。当更专用的工具能够覆盖您的用例时，请优先使用它们；当其他工具都不适用时，再使用计算机使用。如果任务始终在网页浏览器内进行，请使用下一节中的浏览器智能体模式。有关沙箱设置，请参阅[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)。

## 浏览器智能体：browser\_use

当整个任务都在网页内进行时（填写表单、读取页面内容、跨标签页工作），浏览器使用工具比计算机使用更合适。您的应用程序驱动一个由其控制的浏览器，并返回屏幕截图或页面状态；Claude 除了点击和输入之外，还会调用 `read_page`、`find`、`form_input` 和 `get_page_text` 等感知页面的成员工具，因此它除了像素坐标之外，还可以基于元素引用执行操作。

```json
{
  "tools": [{ "type": "browser_toolset_20260801" }]
}
```

与计算机使用工具集一样，该条目不接受 `name`，您可以通过其 `configs` 字段关闭单个成员工具。有关执行约定，请参阅[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="工具参考" icon="book" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    Anthropic 提供的工具的完整目录，包含类型字符串和参数。
  </Card>

  <Card title="工具使用概述" icon="map" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview">
    工具使用的工作原理，以及何时使用 Anthropic 工具、何时自行定义工具。
  </Card>
</CardGroup>
