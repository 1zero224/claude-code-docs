---
title: 处理工具调用
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls
description: 解析 tool_use 块，格式化 tool_result 响应，并使用 is_error 处理错误。
---

本页介绍工具调用的生命周期：从 Claude 的响应中读取 `tool_use` 块，在您的回复中格式化 `tool_result` 块，以及发出错误信号。有关自动处理这些操作的 SDK 抽象，请参阅[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)。

<Note>
  **使用工具运行器更简单：** 本页描述的手动工具处理由[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)自动管理。当您需要对工具执行进行自定义控制时，请使用本页内容。
</Note>

Claude 的响应会根据它使用的是[客户端工具还是服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#how-tool-use-works)而有所不同。

## 处理客户端工具的结果

响应的 `stop_reason` 为 `tool_use`，并包含一个或多个 `tool_use` 内容块，其中包括：

* `id`：此特定工具使用块的唯一标识符。稍后将用它来匹配工具结果。
* `name`：正在使用的工具的名称。
* `input`：一个对象，包含传递给工具的输入，符合工具的 `input_schema`。

属于 [computer use](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)（计算机使用）或 [browser use](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)（浏览器使用）工具集成员的 `tool_use` 块还带有一个 `toolset_name` 字段（`"computer"` 或 `"browser"`）。其 `name` 是 Claude 正在调用的成员工具，例如 `screenshot` 或 `navigate`，因此请根据这两个字段来分派这些块。

<Accordion title="包含 `tool_use` 内容块的 API 响应示例">
  ```json JSON
  {
    "id": "msg_01Aq9w938a90dw8q",
    "model": "claude-opus-5-5",
    "stop_reason": "tool_use",
    "role": "assistant",
    "content": [
      {
        "type": "text",
        "text": "I'll check the current weather in San Francisco for you."
      },
      {
        "type": "tool_use",
        "id": "toolu_01A09q90qw90lq917835lq9",
        "name": "get_weather",
        "input": { "location": "San Francisco, CA", "unit": "celsius" }
      }
    ]
  }
  ```
</Accordion>

当您收到客户端工具的工具使用响应时，您应该：

1. 从 `tool_use` 块中提取 `name`、`id` 和 `input`。

2. 在您的代码库中运行与该工具名称对应的实际工具，并传入工具的 `input`。

3. 通过发送一条 `role` 为 `user` 的新消息来继续对话，其中的 `content` 块包含 `tool_result` 类型和以下信息：

   * `tool_use_id`：此结果所对应的工具使用请求的 `id`。
   * `content`（可选）：工具的结果，可以是字符串（例如 `"content": "15 degrees"`）、嵌套内容块列表（例如 `"content": [{"type": "text", "text": "15 degrees"}]`）或文档块列表（例如 `"content": [{"type": "document", "source": {"type": "text", "media_type": "text/plain", "data": "15 degrees"}}]`）。这些内容块可以使用 `text`、`image`、`document` 或 [`search_result`](https://platform.claude.com/docs/zh-CN/build-with-claude/search-results) 类型。
   * `is_error`（可选）：如果工具执行导致错误，则设置为 `true`。

例如，成功的工具结果如下所示：

```json JSON
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
      "content": "15 degrees"
    }
  ]
}
```

回应计算机使用或浏览器使用成员块的 `tool_result` 还必须回传与 `tool_use` 块相同的 `toolset_name` 值；省略该值的成员结果将被拒绝。其 `content` 的范围也更窄：成员结果只能包含 `text` 和 `image` 块，而浏览器使用结果可以额外添加一个 [`browser_state`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#track-tabs-and-page-state) 块（[标签页管理成员](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#tab-management-results)仅返回该块）。

<Note>
  **重要的格式要求：**

  * 工具结果块必须在消息历史中紧跟其对应的工具使用块。您不能在助手的工具使用消息和用户的工具结果消息之间包含任何消息。
  * 在包含工具结果的用户消息中，`tool_result` 块必须位于 content 数组的最前面。任何文本都必须放在所有工具结果之后。
  * 如果助手轮次还调用了尚无结果块的[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)，则用户消息必须仅包含 `tool_result` 块。结果之后的文本会提前结束该轮次；对于 Claude 直接调用的服务器工具，请求随后会失败并返回 400 错误，错误中会指明未解决的服务器工具。请参阅[停止原因与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#tool-use)。

  如果您收到诸如"tool\_use ids were found without tool\_result blocks immediately after"之类的错误，请检查您的工具结果格式是否正确。
</Note>

<Warning>
  工具结果通常包含来自您无法控制的来源的内容：网页、收到的电子邮件、用户上传的文件、第三方 API。请将此类内容视为不可信内容：能够影响这些内容的攻击者可能会嵌入试图改变 Claude 行为方向的指令（间接提示注入）。请将不可信内容保留在 `tool_result` 块中，而不是放在 `system` 提示或普通的用户 `text` 块中，并参阅[缓解越狱和提示注入](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks#indirect-prompt-injection)以进一步加固。
</Warning>

<AccordionGroup>
  <Accordion title="包含图像的工具结果示例">
    ```json JSON
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": [
            { "type": "text", "text": "15 degrees" },
            {
              "type": "image",
              "source": {
                "type": "base64",
                "media_type": "image/jpeg",
                "data": "/9j/4AAQSkZJRg..."
              }
            }
          ]
        }
      ]
    }
    ```
  </Accordion>

  <Accordion title="空工具结果示例">
    ```json JSON
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9"
        }
      ]
    }
    ```
  </Accordion>

  <Accordion title="包含文档的工具结果示例">
    ```json JSON
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": [
            { "type": "text", "text": "The weather is" },
            {
              "type": "document",
              "source": {
                "type": "text",
                "media_type": "text/plain",
                "data": "15 degrees"
              }
            }
          ]
        }
      ]
    }
    ```
  </Accordion>

  <Accordion title="工具结果前后的文本示例">
    例如，以下写法会导致 400 错误：

    ```json
    {
      "role": "user",
      "content": [
        { "type": "text", "text": "Here are the results:" }, // ❌ Text before tool_result
        { "type": "tool_result", "tool_use_id": "toolu_01" /* ... */ }
      ]
    }
    ```

    当助手轮次仅调用客户端工具时，以下内容是正确的：

    ```json
    {
      "role": "user",
      "content": [
        { "type": "tool_result", "tool_use_id": "toolu_01" /* ... */ },
        { "type": "text", "text": "What should I do next?" } // ✅ Text after tool_result
      ]
    }
    ```
  </Accordion>
</AccordionGroup>

收到工具结果后，Claude 将使用该信息继续生成对原始用户提示的响应。

## 处理服务器工具的结果

Claude 在内部运行工具，并将结果直接整合到其响应中，无需额外的用户交互。

<Note>
  响应可能同时包含客户端 `tool_use` 块和尚无结果块的 `server_tool_use` 块。该服务器工具调用尚未完成，其结果块会在后续响应中到达。请回复一条仅包含客户端工具 `tool_result` 块的用户消息，并保持相同的 `tools` 数组；对于 Claude 直接调用的服务器工具，API 会在该请求中运行它，下一个响应将以其结果块开头。请参阅[停止原因和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#tool-use)。
</Note>

<Tip>
  **与其他 API 的区别**

  与将工具使用分离或使用 `tool` 或 `function` 等特殊角色的 API 不同，Claude API 将工具直接集成到 `user` 和 `assistant` 消息结构中。

  消息包含由 `text`、`image`、`tool_use` 和 `tool_result` 块组成的数组。`user` 消息包含客户端内容和 `tool_result`，而 `assistant` 消息包含 AI 生成的内容和 `tool_use`。
</Tip>

## 使用 is\_error 处理错误

在 Claude 中使用工具时，可能会出现几种不同类型的错误：

<AccordionGroup>
  <Accordion title="工具执行错误">
    如果工具本身在执行过程中抛出错误（例如，获取天气数据时出现网络错误），您可以在 `content` 中返回错误消息，并附带 `"is_error": true`：

    ```json JSON
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": "ConnectionError: the weather service API is not available (HTTP 500)",
          "is_error": true
        }
      ]
    }
    ```

    然后 Claude 会将此错误整合到其对用户的响应中。例如："抱歉，由于天气服务 API 不可用，我无法获取当前天气。请稍后再试。"

    <Tip>
      编写具有指导性的错误消息。不要使用诸如 `"failed"` 之类的通用错误，而应说明出了什么问题以及 Claude 接下来应尝试什么（例如 `"Rate limit exceeded. Retry after 60 seconds."`）。这能为 Claude 提供恢复或调整所需的上下文，而无需猜测。
    </Tip>
  </Accordion>

  <Accordion title="无效的工具名称">
    如果 Claude 对工具的尝试使用无效（例如，缺少必需参数），通常意味着 Claude 没有足够的信息来正确使用该工具。在开发过程中，最佳做法是在工具定义中使用更详细的 `description` 值重新尝试请求。

    不过，您也可以通过一个指示错误的 `tool_result` 继续推进对话，Claude 将尝试补全缺失的信息后再次使用该工具：

    ```json JSON
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": "Error: Missing required 'location' parameter",
          "is_error": true
        }
      ]
    }
    ```

    如果工具请求无效或缺少参数，Claude 会在修正后重试 2–3 次，然后再向用户致歉。

    <Tip>
      要完全消除无效的工具调用，请在工具定义上使用带有 `strict: true` 的[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)。这可以保证工具输入始终与您的 schema 完全匹配，从而防止缺少参数和类型不匹配。
    </Tip>
  </Accordion>

  <Accordion title="服务器工具错误">
    当服务器工具遇到错误时（例如，网络搜索出现网络问题），Claude 会透明地处理这些错误，并尝试向用户提供替代响应或解释。与客户端工具不同，您无需为服务器工具处理 `is_error` 结果。

    具体就网页搜索而言，可能的错误代码包括：

    * `too_many_requests`：超出速率限制
    * `invalid_input`：无效的搜索查询参数
    * `max_uses_exceeded`：超出网页搜索工具的最大使用次数
    * `query_too_long`：查询超出最大长度
    * `unavailable`：发生内部错误
  </Accordion>
</AccordionGroup>

## 后续步骤

<CardGroup cols={3}>
  <Card title="并行工具使用" icon="grid" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use">
    处理 Claude 在单个轮次中调用多个工具的响应。
  </Card>

  <Card title="工具运行器（SDK）" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner">
    让 SDK 为您管理 `tool_use` 循环、结果格式化和重试。
  </Card>

  <Card title="定义工具" icon="hammer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools">
    编写能够引导 Claude 选择正确工具的 schema 和描述。
  </Card>
</CardGroup>
