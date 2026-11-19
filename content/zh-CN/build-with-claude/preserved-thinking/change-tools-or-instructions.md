---
title: 更改工具或指令而不会导致前缀检查失败
url: https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions
description: 发送指令、工具、提醒或摘要，恢复已保存的对话，或转发其他人的请求，同时让 Claude 保留其先前的思考。
featureMetadata:
  status: stable
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-opus-5-5
    - claude-sonnet-5-5
    - claude-haiku-5-5
  supportedPlatforms:
    Claude API: stable
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

要更新工具、指令或其他上下文，请让您已发送的所有内容保持原样，并把更新添加到末尾，且一直留在那里。在具有 “preserved thinking”（保留思考）的模型上，更改您已发送的内容可能会使其后的思考失效，并且[请求可能失败](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)。

具有"preserved thinking"（保留思考）的模型是 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5。

## 原样发回助手轮次

按顺序发回每个响应的完整 `content` 数组，包括文本为空的思考块。文本为空的块是完整的，因为它的 `signature` 以加密形式保存了推理内容。包含 [`fallback` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#continuing-the-conversation)的轮次有其单独的规则。

[序列化器、过滤器和流解析器](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#append-assistant-turns-exactly-as-returned-details)可能会破坏这一点。

## 更改指令

在每个请求中保持 `system` 不变。将您自己的指令放在末尾的 `role: "system"` 消息中，并且仅在有内容发生变化时才发送。Claude 会将其视为来自您（即运营方）的指令。在对话的剩余部分中保留该消息，因为之后删除或改写它会使其后的思考失效。[您的用户的话语](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#user-words)有其单独的规则。

必须随每个轮次一起发送的提醒则不同：[发送一个会自行清除的提醒](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#per-turn-reminders)。

系统消息放在最新的用户消息之后，即使该用户消息包含[工具结果](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#new-instructions-details)。包含文本或工具块的系统消息如果紧跟在 Claude 的轮次之后或紧挨在用户消息之前，会收到 400 错误。因此，当 Claude 的轮次位于最后时，请等到下一条用户消息就位之后再发送。

系统消息并不能保证 Claude 遵循某条指令，因此请在运行您的工具的代码中实施诸如只读模式之类的限制。部分模型[不接受系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)。

## 其他人可以编辑的文本

请思考您的用户是否会期望 Claude 信任某个文件（例如其他人可以编辑的 `AGENTS.md` 或 `CLAUDE.md`），或已存储的记忆。如果不会，则仅在 `tool_result` 块中发送其文本，第一个版本也不例外。原因请参阅[间接提示注入](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks#indirect-prompt-injection)。

不要将此类文本放在系统消息中或用户消息的文本中；在新对话中，也不要将其放在 `system` 中。

当您加载此类文件或文件发生变化时，请告诉 Claude 使用工具读取它，而不要包含其文本。或者，在下一条用户消息之后，自行追加工具调用及其结果：一条包含 `tool_use` 块的 `assistant` 消息，然后是一条其 `tool_result` 块中包含该文本的 `user` 消息。具体步骤请参阅[自行编写工具调用](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#write-the-tool-call)。

您的用户期望 Claude 遵循的文件可以放在用户消息中。只有您控制的指令文件可以放在系统消息中：加载一次，然后追加每个更新后的版本。

## 更新日期、文件或其他上下文

只构建一次 `system` 和第一条消息，并在每个请求中原样发送它们。

将更新后的值放在您即将发送的用户消息中它自己的文本块里，[位于所有工具结果之后](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#user-words)：

```json
{
  "role": "user",
  "content": [
    { "type": "text", "text": "Environment update: the current branch is now release-2." },
    { "type": "text", "text": "Run the tests again." }
  ]
}
```

仅在值发生变化时才说明它，并将其留在那里。

将您的代码添加到某条消息中的任何上下文与该消息一起存储。否则，下一个请求会在同一段思考之前发送不同的内容。

## 添加、移除或关闭工具

在每个请求中保持 `tools` 不变。在其中添加、更改或移除工具（[延迟加载的工具](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#deferred-tools)除外）会使对话中已有的思考失效，因为 `tools` 位于所有思考之前。描述中改写一句话就足以造成这种情况。

要添加或移除工具，请追加一条包含 `tool_addition` 或 `tool_removal` 块的系统消息，并使用[适用于您情况的 beta 标头](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#tool-changes-details)。当 MCP 服务器或插件连接时，也请这样做。

要为单个请求关闭工具，请设置 `tool_choice: {"type": "none"}`，并保持 `tools` 原样。

## 裁剪或总结较早的轮次

要保留最近的轮次及其思考，请在[保留的思考须满足的条件](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-thinking-blocks#conditions-for-kept-thinking-to-stay-valid)下，让 API 通过[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand#request-a-summary)来编写摘要。以它返回的压缩块作为 `messages` 的开头，并移除它所总结的消息。

如果由您自己的代码编写摘要，或者在没有摘要的情况下截去较早的轮次，请[移除保留的轮次的思考，或发送 `"drop_block"`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#server-side-trimming-details)。每次新的截断都会再次使保留的轮次的思考失效，因此请少截断，并且每次大幅截断。如果要改为清除旧的工具结果，请[使用上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#clear-old-content)。

## 保存、恢复或分叉对话

存储您发送的内容和您收到的内容，并原样发送它们。恢复的对话就是一个普通的下一个请求：`system`、`tools` 和较早的 `messages` 必须保持它们原有的值。对于分叉，请将该分支的指令放在其第一条新的用户消息中，或放在其后的系统消息中。思考无法转移到具有不同历史记录的对话中：请改为从摘要开始该对话。

更改您的系统提示或工具的版本发布，对于已在运行中的对话来说就是一种更改。使您的代码变为 append-only（仅追加）的修复本身也可能是一种更改。请继续向每个运行中的对话发送其最后一次成功请求的 `system` 和 `tools`，因为其最新的思考是在该版本之后产生的。回退到更早的版本会使该思考失效。

在新对话中使用更新后的 `system` 和 `tools`。如果该版本出于安全原因移除了某个工具或更改了某条指令，也请将其应用于运行中的对话。追加[一个 `tool_removal` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#tool-changes)或[一条系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#new-instructions)。

## 移除机密、个人数据或注入的文本

安全和隐私优先于保持思考有效。如果某条消息包含您必须删除的机密或个人数据，请移除该文本。同时，也要从每条重复该文本的消息和摘要中将其移除。

对于包含注入指令的工具结果，请将其内容替换为一条简短说明，例如 “Removed.”。保留 `tool_result` 块本身，因为其对应的 `tool_use` 需要一个。如果某个[压缩块](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)包含该文本，请删除该块，并从您自己的摘要开始。

然后，删除该文本首次出现位置之后的每个 `thinking` 和 `redacted_thinking` 块。如果该文本位于 `system` 或 `tools` 中，那就是全部此类块。每个块都可能重复该文本，而您无法读取或编辑其中的内容。在这里仅发送 `"drop_block"` 是不够的，因为它会把这些块留在您存储的历史记录中。对话会在没有那部分推理的情况下继续进行。

在注入的结果之后，后续轮次可能已经根据它采取了行动。请检查此后进行的工具调用，并撤销您需要撤销的操作。

## 技术细节

### 代码更改助手轮次的方式

* 序列化器丢弃它不认识的块类型，或对块进行重新排序、合并或拆分。这会改变每个后续块之前的内容。
* 代码跳过看起来为空的思考块。如果跳过全部此类块，Claude 会失去那部分推理。如果只跳过其中一部分，由此产生的空缺会使其后的思考失效。
* 您自己编写的流解析器丢弃了没有收到文本的思考块。SDK 的流辅助工具会保留它。

### 省略思考

Claude 会失去您省略的推理。可以省略最早的块、最新的块或全部块，但不能省略中间的块。每个块都记录了它是在哪个块之后产生的，因此空缺会使其后的块失效。在一个轮次中彼此之间没有其他块的思考块会一起接受前缀检查，因此要么全部省略，要么都不省略。一旦某个块被省略，就要一直省略它。

当工具调用尚未完成时，请[发回该轮次的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)，以便 Claude 带着其推理完成这一回合。

### 工具循环中您的指令

消息按以下顺序排列：

```json
[
  {
    "role": "assistant",
    "content": [
      { "type": "thinking", "thinking": "", "signature": "..." },
      {
        "type": "tool_use",
        "id": "toolu_01",
        "name": "run_tests",
        "input": { "path": "tests/" }
      }
    ]
  },
  {
    "role": "user",
    "content": [
      { "type": "tool_result", "tool_use_id": "toolu_01", "content": "2 failed, 48 passed" }
    ]
  },
  {
    "role": "system",
    "content": "Plan mode is off. You can edit files."
  }
]
```

### 自行编写工具调用

在第一个请求中声明一个读取文件的工具，或[稍后在系统消息中添加它](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#tool-changes)，并让您的代码响应 Claude 对它的调用。在下一条用户消息之后，追加一条包含该工具的 `tool_use` 块的 `assistant` 消息。紧接着，追加一条 `user` 消息，其 `tool_result` 块具有匹配的 `tool_use_id` 并包含文件的文本。Claude 随后会作出回答：

```json
[
  { "role": "user", "content": "Run the tests again." },
  {
    "role": "assistant",
    "content": [
      {
        "type": "tool_use",
        "id": "toolu_app_agents_md_1",
        "name": "read_file",
        "input": { "path": "AGENTS.md" }
      }
    ]
  },
  {
    "role": "user",
    "content": [
      {
        "type": "tool_result",
        "tool_use_id": "toolu_app_agents_md_1",
        "content": "The file's text"
      }
    ]
  }
]
```

将两者都保留在原位，并为每个新版本追加一对新的消息，使用新的 `id`。

### 工具循环中您的用户的话语和更新后的值

如果您转发用户在 Claude 工作期间输入的内容，请在发送携带 `tool_result` 块的用户消息之前添加它。将其作为最后一个 `tool_result` 之后的文本块。

来自您的值也可以改为放在系统消息中。在工具结果之后，Claude Sonnet 5.5 和 Claude Haiku 5.5 可能会将系统消息中用户的话语当作提示注入而忽略。如果您自己的指令和提醒位于每个工具结果之后，Claude Sonnet 5.5 也可能对它们这样做。

当该轮次还调用了尚无结果的[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)，或使用了程序化工具调用时，包含工具结果的用户消息必须只包含 `tool_result` 块。这时请将文本添加到下一条用户消息中。

### 不接受系统消息的模型

[对话中途系统消息的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#feature-availability)列出了接受系统消息的平台和模型。在这些平台上，该列表包含每个具有保留思考的模型。如果您切换到不在该列表中的模型，请参阅[当模型不接受系统消息时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#unchosen-switches-details)。

### 为单个轮次发送提醒

将提醒作为[轮次范围的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#turn-scoped-system-messages)发送：设置 `clear_at: "next_user_message"`，并发送 beta 标头 `mid-conversation-system-clear-at-2026-08-21`。它位于携带 `tool_result` 块的用户消息之后：

```json
{
  "role": "system",
  "clear_at": "next_user_message",
  "content": "Run the linter before you report that a task is done."
}
```

在下一条用户消息处（即使该消息只包含 `tool_result` 块），API 会清除它，因此 Claude 不再读取它。在 Claude Sonnet 5.5 上，[不要在每个工具结果之后都发送一个](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#user-words)。将每条提醒都保留在您的 `messages` 中，无论是否已被清除，除非它[包含您必须移除的内容](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#remove-sensitive-content)。移除或改写提醒会使其后的思考失效。

如果不使用该标头，请发送一条普通的系统消息并将其保留在原位。它永远不会被清除，因此提醒会不断累积。在不接受系统消息的模型上，请在用户消息中使用文本块。

### 工具更改需要哪种块和 beta 标头

关于接受各标头的平台，请参阅[对话中途的系统消息与工具变更](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)。

* **如果您在开始时就知道所有工具：** 在第一个请求中声明所有工具，并对您稍后才提供的工具设置 `defer_loading: true`。然后在系统消息的 `tool_addition` 或 `tool_removal` 块中按名称指定该工具。Beta 标头：`inline-tools-2026-09-15` 或 `mid-conversation-tool-changes-2026-07-01`。
* **如果您稍后才得知某个工具，或者某个工具的 schema 会发生变化：** 在系统消息的 `tool_addition` 块中定义该工具。Beta 标头：`inline-tools-2026-09-15`。
* **如果您稍后才得知某个工具，且所在平台不支持 `inline-tools-2026-09-15` 标头：** 将该工具添加到 `tools` 的末尾并设置 `defer_loading: true`（这不算更改），然后按名称提供它。Beta 标头：`mid-conversation-tool-changes-2026-07-01`。
* **如果您通过 MCP 连接器较晚连接 MCP 服务器：** 在 `tool_addition` 块中定义其 `mcp_toolset`。Beta 标头：`inline-tools-2026-09-15` 和 `mcp-client-2026-09-15` 两者都需要。要检查您的平台是否接受第二个标头，请参阅[MCP 连接器的可用性](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#feature-availability)。

如果 MCP 服务器的工具列表可能会变化，请参阅[固定 MCP 服务器的工具列表](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#pin-mcp-tool-list)。

以下系统消息按名称提供一个已声明的工具，并定义另一个工具，这需要 `inline-tools-2026-09-15`：

```json
{
  "role": "system",
  "content": [
    {
      "type": "tool_addition",
      "tool": { "type": "tool_reference", "name": "search_tickets" }
    },
    {
      "type": "tool_addition",
      "tool": {
        "type": "tool_definition",
        "definition": {
          "name": "get_weather",
          "description": "Get the current weather in a city.",
          "input_schema": {
            "type": "object",
            "properties": { "city": { "type": "string" } },
            "required": ["city"]
          }
        }
      }
    }
  ]
}
```

### 带有 `defer_loading` 的工具，以及指明它的 400 错误

设置了 `defer_loading: true` 的工具不会进入 Claude 的上下文，也不参与前缀检查，直到某条消息或工具搜索结果引用它为止。在此之前，添加、更新或移除它都不算更改。在此之后则算。开启或关闭 `defer_loading` 也算作更改 `tools`。

请至少声明一个非延迟加载的工具，否则请求可能会收到指明 `defer_loading` 的 400 错误。

### 缩短或清除旧的结果和图像

在首次发送工具结果或图像之前对其进行缩短或降低分辨率。如果您的代码清除、缩短或重新编码已经发送过的内容，其后的思考就会失效。要在之后清除旧的工具结果（包括其中的任何图像），请使用上下文编辑。发送带有 `clear_tool_uses_20250919` 类型编辑的 `context_management`，以及 beta 标头 `context-management-2025-06-27`。有关完整请求及其选项，请参阅[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#tool-result-clearing)。

### 通过 URL 发送的文件

对于通过 URL 提供的文件，API 会比较它获取到的字节。因此，当签名 URL 过期时，为同一文件使用新的 URL 是没有问题的。如果 URL 背后的内容可能会变化，请从第一个请求起就发送 `file_id` 或 base64。

### 如果您自己的代码总结或截去较早的轮次

在您自己进行总结或截断之后，保留的轮次的思考会无法通过[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)，因为它之前的消息已经不在了。请移除该思考，或[发送 `"drop_block"`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#preserved-thinking-controls)。Claude 仍会在没有该思考的情况下读取该轮次。

### 库、代理和网关

如果其他人的请求经过您的代码，您的改写会更改他们的历史记录，而他们可能无法看到或修复这些更改。不要隐藏 400 错误。如果您为调用方进行了恢复，请告知对方，因为他们的历史记录中仍然包含该更改。

* 按原样传递 `anthropic-beta` 值和 `thinking.block_binding`。如果您的网关只允许已知字段，请添加 `thinking.block_binding` 和标头值 `thinking-binding-controls-2026-08-01`，否则您的用户无法选择 `"drop_block"`。
* 不要在移除 beta 标头的同时转发该设置。这样的请求会收到 400 错误。
* 将文本为空的思考块以及 `input_transformations` 数组返回给调用方。
* 将 `role: "system"` 消息保留在原位。将其移入 `system` 会使对话中已有的思考失效。
* 在将 [`prefix_mismatch_behavior`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#preserved-thinking-controls) 设置为 `"error"` 的情况下进行测试，这样在每个具有保留思考的模型上都会返回该错误。

## 后续步骤

<CardGroup cols={2}>
  <Card title="保留思考" icon="book" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking">
    什么是保留思考、思考块与什么绑定，以及切换模型或账户。
  </Card>

  <Card title="修复绑定到其他对话的思考块" icon="wrench" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation">
    恢复失败的对话，并了解丢弃思考的代价。
  </Card>

  <Card title="对话中途的系统消息与工具变更" icon="message" href="https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages">
    系统消息和工具变更应放在哪里，以及哪些模型和平台接受它们。
  </Card>
</CardGroup>
