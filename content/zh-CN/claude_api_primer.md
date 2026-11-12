---
title: Claude API 使用入门
url: https://platform.claude.com/docs/zh-CN/claude_api_primer
description: 本指南旨在为 Claude 提供使用 Claude API 的基础知识。它对模型 ID/基本 Messages API、工具使用、流式传输和思考进行了解释并给出示例，不涉及其他内容。
---

# Claude API 使用入门

> 本指南旨在为 Claude 提供使用 Claude API 的基础知识。它对模型 ID/基本 Messages API、"tool use"（工具使用）、"streaming"（流式传输）和 “thinking”（思考）进行了解释并给出示例，不涉及其他内容。

## 模型

```text wrap
Recommended default for most work, including complex agentic coding: Claude Opus 5.5: claude-opus-5-5
Step up for the hardest long-running agentic and research tasks, at 2x Claude Opus 5.5 pricing: Claude Fable 5.2: claude-fable-5-2
Previous Opus model: Claude Opus 5: claude-opus-5
Smart model: Claude Sonnet 5.5: claude-sonnet-5-5
Previous Sonnet model: Claude Sonnet 5: claude-sonnet-5
For fast, cost-effective tasks: Claude Haiku 5.5: claude-haiku-5-5
Previous Haiku model: Claude Haiku 4.5: claude-haiku-4-5-20251001
```

## 调用 API

### 基本请求和响应

<CodeGroup exclude="shell:cURL, typescript, csharp, go, java, php, ruby">
  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{"role": "user", "content": "Hello, Claude"}'
  ```

  ```python Python
  import anthropic

  message = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  print(message)
  ```
</CodeGroup>

```json Output
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello!"
    }
  ],
  "model": "claude-opus-5-5",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 12,
    "output_tokens": 6
  }
}
```

### 多轮对话

Messages API 是 “stateless”（无状态）的，这意味着您始终需要将完整的对话历史发送给 API。您可以使用这种模式随时间逐步构建对话。较早的对话轮次不一定真的来自 Claude。您可以使用合成的 `assistant` 消息。

借助处于 beta 阶段的 [message threads](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads)（消息线程），API 可以将对话保留下来，这样之后的每个请求只需发送新消息。线程会在其最后一次请求后 5 分钟过期，并在其最后一次使用后不超过 1 小时内被删除。

<CodeGroup exclude="shell:cURL, typescript, csharp, go, java, php, ruby">
  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  messages:
    - role: user
      content: Hello, Claude
    - role: assistant
      content: Hello!
    - role: user
      content: Can you describe LLMs to me?
  YAML
  ```

  ```python Python
  import anthropic

  message = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[
          {"role": "user", "content": "Hello, Claude"},
          {"role": "assistant", "content": "Hello!"},
          {"role": "user", "content": "Can you describe LLMs to me?"},
      ],
  )
  print(message)
  ```
</CodeGroup>

### 预填充 Claude 的响应

您可以在输入消息列表的最后一个位置预填充 Claude 响应的一部分。使用此技术来塑造 Claude 的响应。以下示例使用 `"max_tokens": 1` 从 Claude 获取单个多项选择答案。

<Note>
  Claude 4.6 及更高版本的模型以及 Claude Mythos Preview 不支持 assistant 消息预填充；发送给这些模型的请求必须以 user 消息结尾。以下示例使用的是支持预填充的模型。
</Note>

<CodeGroup exclude="shell:cURL, typescript, csharp, go, java, php, ruby">
  ```bash CLI
  ant messages create <<'YAML'
  model: claude-sonnet-4-5
  max_tokens: 1
  messages:
    - role: user
      content: "What is latin for Ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae"
    - role: assistant
      content: "The answer is ("
  YAML
  ```

  ```python Python
  import anthropic

  message = anthropic.Anthropic().messages.create(
      model="claude-sonnet-4-5",
      max_tokens=1,
      messages=[
          {
              "role": "user",
              "content": "What is latin for Ant? (A) Apoidea, (B) Rhopalocera, (C) Formicidae",
          },
          {"role": "assistant", "content": "The answer is ("},
      ],
  )
  print(message.content[0].text)
  ```
</CodeGroup>

### 视觉

Claude 可以读取请求中的文本和图像。图像支持 `base64` 和 `url` 两种来源类型，以及 `image/jpeg`、`image/png`、`image/gif` 和 `image/webp` 媒体类型。

<CodeGroup exclude="shell:cURL, typescript, csharp, go, java, php, ruby">
  ```bash CLI
  # 选项 1：base64 编码的图像（@ 前缀会自动将二进制文件编码为 base64）
  curl -sSo vision-example.jpg \
    https://platform.claude.com/docs/images/vision-example.jpg

  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  messages:
    - role: user
      content:
        - type: image
          source:
            type: base64
            media_type: image/jpeg
            data: "@./vision-example.jpg"
        - type: text
          text: What is in the above image?
  YAML

  # 选项 2：通过 URL 引用的图像
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  messages:
    - role: user
      content:
        - type: image
          source:
            type: url
            url: https://platform.claude.com/docs/images/vision-example.jpg
        - type: text
          text: What is in the above image?
  YAML
  ```

  ```python Python
  import anthropic
  import base64
  import httpx2

  # 选项 1：Base64 编码的图像
  image_url = "https://platform.claude.com/docs/images/vision-example.jpg"
  image_media_type = "image/jpeg"
  image_data = base64.standard_b64encode(httpx2.get(image_url).content).decode("utf-8")

  message = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[
          {
              "role": "user",
              "content": [
                  {
                      "type": "image",
                      "source": {
                          "type": "base64",
                          "media_type": image_media_type,
                          "data": image_data,
                      },
                  },
                  {"type": "text", "text": "What is in the above image?"},
              ],
          }
      ],
  )
  print(next(block.text for block in message.content if block.type == "text"))

  # 选项 2：通过 URL 引用的图像
  message_from_url = anthropic.Anthropic().messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[
          {
              "role": "user",
              "content": [
                  {
                      "type": "image",
                      "source": {
                          "type": "url",
                          "url": "https://platform.claude.com/docs/images/vision-example.jpg",
                      },
                  },
                  {"type": "text", "text": "What is in the above image?"},
              ],
          }
      ],
  )
  print(next(block.text for block in message_from_url.content if block.type == "text"))
  ```
</CodeGroup>

## 思考

思考有时可以帮助 Claude 完成非常困难的任务。当前的机制是 [adaptive thinking（自适应思考）](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（`thinking: {"type": "adaptive"}`）：Claude 自行决定何时思考以及思考多少，您可以通过 [`effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 参数而非令牌预算来引导思考深度。Claude 4.6 及更高版本的模型以及 Claude Mythos Preview 支持自适应思考。在 Claude 5 及更高版本的模型以及 Claude Mythos Preview 上，省略 `thinking` 参数时思考默认开启。

以下模型支持思考：

* Claude Fable 5.2（`claude-fable-5-2`，仅支持自适应思考，始终开启）
* Claude Mythos 5.2（`claude-mythos-5-2`，仅支持自适应思考，始终开启）
* Claude Opus 5.5（`claude-opus-5-5`，仅支持自适应思考，始终开启）
* Claude Sonnet 5.5（`claude-sonnet-5-5`，仅支持自适应思考，默认开启；`thinking: {"type": "disabled"}` 会返回 400 错误，因此请发送 `thinking: {"type": "between_tools"}` 来关闭前置思考）
* Claude Haiku 5.5（`claude-haiku-5-5`，仅支持自适应思考，默认开启）
* Claude Fable 5.1（`claude-fable-5-1`，仅支持自适应思考，始终开启）
* Claude Mythos 5.1（`claude-mythos-5-1`，仅支持自适应思考，始终开启）
* Claude Fable 5（`claude-fable-5`，仅支持自适应思考，始终开启）
* Claude Mythos 5（`claude-mythos-5`，仅支持自适应思考，始终开启）
* Claude Opus 5（claude-opus-5，仅支持自适应思考，默认开启）
* Claude Sonnet 5（`claude-sonnet-5`，仅支持自适应思考，默认开启）
* Claude Opus 4.8（claude-opus-4-8，仅支持自适应思考）
* Claude Opus 4.7（`claude-opus-4-7`，仅支持自适应思考）
* Claude Opus 4.6（`claude-opus-4-6`，支持自适应思考或旧版手动思考）
* Claude Sonnet 4.6（`claude-sonnet-4-6`，支持自适应思考或旧版手动思考）
* Claude Opus 4.5（`claude-opus-4-5-20251101`，仅支持旧版手动思考）
* Claude Sonnet 4.5（`claude-sonnet-4-5-20250929`，[已弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)，仅支持旧版手动思考）
* Claude Haiku 4.5（`claude-haiku-4-5-20251001`，仅支持旧版手动思考）

<Note>
  在 Claude 4.7 及更高版本的模型上，不支持手动 “extended thinking”（扩展思考）（带有 `budget_tokens` 值的 `type: enabled`），使用时会返回 400 错误。请改用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（`type: adaptive`）。
</Note>

### 思考的工作原理

开启思考后，Claude 会创建 `thinking` 内容块，并在其中输出其内部推理。API 响应包含 `thinking` 内容块，随后是 `text` 内容块。

<CodeGroup exclude="shell:cURL, typescript, csharp, go, java, php, ruby">
  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 16000
  thinking:
    type: adaptive
    display: summarized
  messages:
    - role: user
      content: Are there an infinite number of prime numbers such that n mod 4 == 3?
  YAML
  ```

  ```python Python
  import anthropic

  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=16000,
      thinking={"type": "adaptive", "display": "summarized"},
      messages=[
          {
              "role": "user",
              "content": "Are there an infinite number of prime numbers such that n mod 4 == 3?",
          }
      ],
  )

  # 响应包含摘要形式的思考块和文本块
  for block in response.content:
      match block.type:
          case "thinking":
              print(f"\nThinking summary: {block.thinking}")
          case "text":
              print(f"\nResponse: {block.text}")
  ```
</CodeGroup>

手动扩展思考（`thinking: {"type": "enabled", "budget_tokens": N}`）是旧版机制。它仅适用于支持思考的 Claude 4 至 4.6 模型；Claude 4.7 及更高版本的模型会以 400 错误拒绝 `type: enabled`，并改用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。使用手动扩展思考时，`budget_tokens` 设置 Claude 在其内部推理过程中允许使用的最大令牌数；该限制适用于完整的思考令牌，而非摘要后的输出。除非您使用的是[交错思考](https://platform.claude.com/docs/zh-CN/claude_api_primer#interleaved-thinking)，否则 `budget_tokens` 必须小于 `max_tokens`，以便 Claude 在思考完成后有空间编写其响应。

## 结合工具使用的思考

思考可以与工具使用一起使用，使 Claude 能够对工具选择和结果处理进行推理。

重要限制：

1. **工具选择限制：** 使用手动扩展思考（`thinking: {"type": "enabled"}`）时，仅支持 `tool_choice: {"type": "auto"}`（默认）或 `tool_choice: {"type": "none"}`。自适应思考接受强制工具使用，但[强制工具使用](https://platform.claude.com/docs/zh-CN/claude_api_primer#forcing-tool-use)下列出的模型除外。在接受强制工具使用的情况下，强制的工具调用会跳过思考：响应以工具调用开始，且没有 `thinking` 块。
2. **保留思考块：** 在工具使用期间，您必须将最后一条 assistant 消息的 `thinking` 块传回 API。

### 保留思考块

<CodeGroup exclude="shell:cURL, shell:CLI, typescript, csharp, go, java, php, ruby">
  ```python Python
  import anthropic

  client = anthropic.Anthropic()

  weather_tool = {
      "name": "get_weather",
      "description": "Get the current weather for a location.",
      "input_schema": {
          "type": "object",
          "properties": {"location": {"type": "string", "description": "The city name."}},
          "required": ["location"],
      },
  }

  weather_data = {"temperature": 72}

  # 第一个请求 - Claude 返回思考内容和工具请求
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=16000,
      thinking={"type": "adaptive", "display": "summarized"},
      tools=[weather_tool],
      messages=[{"role": "user", "content": "What's the weather in Paris?"}],
  )

  # 提取思考块和工具使用块
  thinking_block = next(
      (block for block in response.content if block.type == "thinking"), None
  )
  tool_use_block = next(
      (block for block in response.content if block.type == "tool_use"), None
  )

  # 第二个请求 - 包含思考块和工具结果
  continuation = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=16000,
      thinking={"type": "adaptive", "display": "summarized"},
      tools=[weather_tool],
      messages=[
          {"role": "user", "content": "What's the weather in Paris?"},
          # 注意，thinking_block 与 tool_use_block 一同传入
          {"role": "assistant", "content": [thinking_block, tool_use_block]},
          {
              "role": "user",
              "content": [
                  {
                      "type": "tool_result",
                      "tool_use_id": tool_use_block.id,
                      "content": f"Current temperature: {weather_data['temperature']}°F",
                  }
              ],
          },
      ],
  )

  for block in continuation.content:
      if block.type == "text":
          print(block.text)
  ```
</CodeGroup>

### 交错思考

“Interleaved thinking”（交错思考）使 Claude 能够在工具调用之间进行思考，在确定下一步之前对工具结果进行推理。

<Info>
  在支持[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（`thinking: {type: "adaptive"}`）的模型上，交错思考会自动启用，无需 beta 标头。Sonnet 4.6 既支持配合手动扩展思考使用的 `interleaved-thinking-2025-05-14` beta 标头，也支持自适应思考。
</Info>

在使用手动扩展思考的较旧模型（Claude 4、4.5 和 Sonnet 4.6 模型）上，通过在 API 请求中添加 beta 标头 `interleaved-thinking-2025-05-14` 来启用交错思考：

<CodeGroup exclude="shell:cURL, typescript, csharp, go, java, php, ruby">
  ```bash CLI
  ant beta:messages create --beta interleaved-thinking-2025-05-14 <<'YAML'
  model: claude-sonnet-4-6
  max_tokens: 16000
  thinking:
    type: enabled
    budget_tokens: 10000
  tools:
    - name: calculator
      description: Perform arithmetic calculations.
      input_schema:
        type: object
        properties:
          expression:
            type: string
            description: The math expression to evaluate.
        required:
          - expression
    - name: database_query
      description: Query the product database.
      input_schema:
        type: object
        properties:
          query:
            type: string
            description: The database query.
        required:
          - query
  messages:
    - role: user
      content: "What's the total revenue if we sold 150 units of product A at $50 each?"
  YAML
  ```

  ```python Python
  import anthropic

  client = anthropic.Anthropic()

  calculator_tool = {
      "name": "calculator",
      "description": "Perform arithmetic calculations.",
      "input_schema": {
          "type": "object",
          "properties": {
              "expression": {
                  "type": "string",
                  "description": "The math expression to evaluate.",
              }
          },
          "required": ["expression"],
      },
  }

  database_tool = {
      "name": "database_query",
      "description": "Query the product database.",
      "input_schema": {
          "type": "object",
          "properties": {
              "query": {"type": "string", "description": "The database query."}
          },
          "required": ["query"],
      },
  }

  response = client.beta.messages.create(
      model="claude-sonnet-4-6",
      max_tokens=16000,
      thinking={"type": "enabled", "budget_tokens": 10000},
      tools=[calculator_tool, database_tool],
      messages=[
          {
              "role": "user",
              "content": "What's the total revenue if we sold 150 units of product A at $50 each?",
          }
      ],
      betas=["interleaved-thinking-2025-05-14"],
  )

  for block in response.content:
      match block.type:
          case "thinking":
              print(f"Thinking: {block.thinking}")
          case "tool_use":
              print(f"Tool call: {block.name}({block.input})")
          case "text":
              print(f"Response: {block.text}")
  ```
</CodeGroup>

在使用交错思考时，并且仅在使用交错思考时（而非常规的手动扩展思考），`budget_tokens` 可以超过 `max_tokens` 参数，因为在这种情况下 `budget_tokens` 表示一个 assistant 轮次内所有思考块的总预算。

## 工具使用

### 指定客户端工具

客户端工具在 API 请求的顶层参数 `tools` 中指定。每个工具定义包括：

| 参数             | 描述                                                    |
| -------------- | ----------------------------------------------------- |
| `name`         | 工具的名称。必须匹配正则表达式 `^[a-zA-Z0-9_-]{1,128}$`。             |
| `description`  | 对工具功能、使用时机及其行为方式的详细纯文本描述。                             |
| `input_schema` | 定义工具预期参数的 [JSON Schema](https://json-schema.org/) 对象。 |

```json
{
  "name": "get_weather",
  "description": "Get the current weather in a given location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "The city and state, e.g. San Francisco, CA"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "The unit of temperature, either 'celsius' or 'fahrenheit'"
      }
    },
    "required": ["location"]
  }
}
```

### 工具定义的最佳实践

**提供极其详细的描述。** 这是迄今为止影响工具性能的最重要因素。您的描述应解释有关该工具的每一个细节，包括：

* 工具的功能
* 何时应该使用它（以及何时不应该使用）
* 每个参数的含义以及它如何影响工具的行为
* 任何重要的注意事项或限制

**对于复杂工具，考虑使用 `input_examples`。** 对于具有嵌套对象、可选参数或对格式敏感的输入的工具，您可以使用 `input_examples` 字段（beta）提供具体示例。这会向 Claude 展示预期的输入模式。详情请参阅[提供工具使用示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#providing-tool-use-examples)。

良好工具描述的示例：

```json
{
  "name": "get_stock_price",
  "description": "Retrieves the current stock price for a given ticker symbol. The ticker symbol must be a valid symbol for a publicly traded company on a major US stock exchange like NYSE or NASDAQ. The tool will return the latest trade price in USD. It should be used when the user asks about the current or most recent price of a specific stock. It will not provide any other information about the stock or company.",
  "input_schema": {
    "type": "object",
    "properties": {
      "ticker": {
        "type": "string",
        "description": "The stock ticker symbol, e.g. AAPL for Apple Inc."
      }
    },
    "required": ["ticker"]
  }
}
```

## 控制 Claude 的输出

### 强制工具使用

您可以通过在 `tool_choice` 字段中指定工具来强制 Claude 使用特定工具：

```python
tool_choice = {"type": "tool", "name": "get_weather"}
```

使用 `tool_choice` 参数时，有四个可能的选项：

* `auto` 允许 Claude 自行决定是否调用任何提供的工具（默认）。
* `any` 告诉 Claude 必须使用所提供工具中的一个。
* `tool` 强制 Claude 始终使用某个特定工具。
* `none` 阻止 Claude 使用任何工具。

在 Claude Fable 5.2、Claude Mythos 5.2、Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.1 和 Claude Mythos 5.1 上，`any` 和 `tool` 会返回 400 错误。请将 `tool_choice` 保持为 `auto`，并在工具定义上设置 `"strict": true`，以保证 Claude 发出的任何调用都与该工具的 `input_schema` 匹配。请参阅[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)。

### JSON 输出

工具不一定需要是客户端函数。每当您希望模型返回遵循所提供 schema 的 JSON 输出时，都可以使用工具。

### 并行工具使用

默认情况下，Claude 可能会使用多个工具来回答用户查询。您可以通过设置 `disable_parallel_tool_use=true` 来禁用此行为。

## 处理工具使用和工具结果内容块

### 处理客户端工具的结果

响应的 `stop_reason` 为 `tool_use`，并包含一个或多个 `tool_use` 内容块，其中包括：

* `id`：此特定工具使用块的唯一标识符。
* `name`：正在使用的工具的名称。
* `input`：包含传递给工具的输入的对象。

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01A09q90qw90lq917835lq9",
      "name": "get_weather",
      "input": { "location": "San Francisco, CA" }
    }
  ]
}
```

当您收到工具使用响应时，您应该：

1. 从 `tool_use` 块中提取 `name`、`id` 和 `input`。
2. 在您的代码库中运行与该工具名称对应的实际工具。
3. 通过发送包含 `tool_result` 的新消息来继续对话：

```json
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

### 处理 `max_tokens` 停止原因

如果 Claude 的响应在工具使用期间因达到 `max_tokens` 限制而被截断，请使用更高的 `max_tokens` 值重试请求。

### 处理 `pause_turn` 停止原因

使用网络搜索等服务器工具时，API 可能会返回 `pause_turn` 停止原因。请在后续请求中将暂停的响应原样传回，以继续对话。

## 错误排查

### 工具执行错误

如果工具本身在执行期间抛出错误，请返回带有 `"is_error": true` 的错误消息：

```json
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

### 无效的工具名称

如果 Claude 尝试的工具使用无效（例如，缺少必需参数），请在工具定义中使用更详细的 `description` 值后重试请求。

## 流式传输消息

创建 Message 时，您可以设置 `"stream": true`，以使用 “server-sent events”（服务器发送事件），即 SSE，增量地流式传输响应。

### 使用 SDK 进行流式传输

<CodeGroup exclude="shell:cURL, typescript, csharp, go, java, php, ruby">
  ```bash CLI
  ant messages create --stream \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello"}'
  ```

  ```python Python
  import anthropic

  client = anthropic.Anthropic()

  with client.messages.stream(
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello"}],
      model="claude-opus-5-5",
  ) as stream:
      for text in stream.text_stream:
          print(text, end="", flush=True)
  ```
</CodeGroup>

### 事件类型

每个服务器发送事件都包含一个命名的事件类型和相关的 JSON 数据。每个流使用以下事件流程：

1. `message_start`：包含一个 `content` 为空的 `Message` 对象。
2. 一系列内容块，每个内容块都有 `content_block_start`、一个或多个 `content_block_delta` 事件以及 `content_block_stop`。
3. 一个或多个 `message_delta` 事件，表示对最终 `Message` 对象的顶层更改。
4. 最后一个 `message_stop` 事件。

**警告：** `message_delta` 事件的 `usage` 字段中显示的令牌计数是*累计的*。

### 内容块增量类型

#### 文本增量

```json
{
  "type": "content_block_delta",
  "index": 0,
  "delta": { "type": "text_delta", "text": "Hello frien" }
}
```

#### 输入 JSON 增量

对于 `tool_use` 内容块，增量是*部分 JSON 字符串*：

```json
{"type": "content_block_delta","index": 1,"delta": {"type": "input_json_delta","partial_json": "{\"location\": \"San Fra"}}}
```

#### 思考增量

在流式传输中使用思考时：

```json
{
  "type": "content_block_delta",
  "index": 0,
  "delta": {
    "type": "thinking_delta",
    "thinking": "Let me solve this step by step..."
  }
}
```

### 基本流式传输请求示例

```sse
event: message_start
data: {"type": "message_start", "message": {"id": "msg_1nZdL29xx5MUA1yADyHTEsnR8uuvGzszyY", "type": "message", "role": "assistant", "content": [], "model": "claude-opus-5-5", "stop_reason": null, "stop_sequence": null, "usage": {"input_tokens": 25, "output_tokens": 1}}}

event: content_block_start
data: {"type": "content_block_start", "index": 0, "content_block": {"type": "text", "text": ""}}

event: content_block_delta
data: {"type": "content_block_delta", "index": 0, "delta": {"type": "text_delta", "text": "Hello"}}

event: content_block_delta
data: {"type": "content_block_delta", "index": 0, "delta": {"type": "text_delta", "text": "!"}}

event: content_block_stop
data: {"type": "content_block_stop", "index": 0}

event: message_delta
data: {"type": "message_delta", "delta": {"stop_reason": "end_turn", "stop_sequence":null}, "usage": {"output_tokens": 15}}

event: message_stop
data: {"type": "message_stop"}
```
