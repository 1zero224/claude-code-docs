---
title: 服务器工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools
description: 使用由 Anthropic 运行的工具：server_tool_use 块、pause_turn 续接、混合服务器工具和客户端工具的轮次，以及域名过滤。
---

由服务器执行的工具共享以下机制：`server_tool_use` 块、`pause_turn` 续接、混合服务器工具和客户端工具的轮次、"Zero Data Retention"（零数据保留），即 ZDR 资格，以及域名过滤。有关各个工具的信息，请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)。

## server\_tool\_use 块

当由服务器执行的工具运行时，`server_tool_use` 块会出现在 Claude 的响应中。其 `id` 字段使用 `srvtoolu_` 前缀，以便与客户端工具调用区分开来：

```json
{
  "type": "server_tool_use",
  "id": "srvtoolu_01A2B3C4D5E6F7G8H9",
  "name": "web_search",
  "input": { "query": "latest quantum computing breakthroughs" }
}
```

API 在内部运行该工具。您可以在响应中看到调用及其结果，但无需处理执行过程。与客户端 `tool_use` 块不同，您不需要用 `tool_result` 进行响应。工具的结果块（例如网络搜索的 `web_search_tool_result`）在同一助手轮次中跟随在 `server_tool_use` 块之后，并通过 `tool_use_id` 配对。如果 Claude 同时调用了您的某个客户端工具，则 `server_tool_use` 块会在没有结果的情况下出现，并且响应以 `stop_reason: "tool_use"` 结束。当您在下一个请求中返回客户端 `tool_result` 块时，API 才会运行该工具。

## 服务器端循环与 pause\_turn

在使用网络搜索等服务器工具时，API 会在服务器端的 “agentic loop”（智能体循环）中执行工具调用。在长时间运行的轮次中，API 可能会暂停该循环并返回 `pause_turn` 停止原因。

以下是处理 `pause_turn` 停止原因的方法：

<CodeGroup>
  ```bash cURL
  # 初始请求。如果响应中的 "stop_reason" 为 "pause_turn"，请继续
  # 该轮次：将 assistant 内容追加到 messages 后重新发送请求。
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Search for comprehensive information about quantum computing breakthroughs in 2025"
        }
      ],
      "tools": [{"type": "web_search_20250305", "name": "web_search", "max_uses": 10}]
    }'
  ```

  ```bash CLI
  # 初始请求。如果输出中的 "stop_reason" 为 "pause_turn"，请重新运行，并将
  # 助手内容追加到 messages 中。
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  tools:
    - {type: web_search_20250305, name: web_search, max_uses: 10}
  messages:
    - {role: user, content: "Search for comprehensive information about quantum computing breakthroughs in 2025"}
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 带有网络搜索的初始请求
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Search for comprehensive information about quantum computing breakthroughs in 2025",
          }
      ],
      tools=[{"type": "web_search_20250305", "name": "web_search", "max_uses": 10}],
  )

  # 检查响应的停止原因是否为 pause_turn
  if response.stop_reason == "pause_turn":
      # 使用已暂停的内容继续对话
      messages = [
          {
              "role": "user",
              "content": "Search for comprehensive information about quantum computing breakthroughs in 2025",
          },
          {"role": "assistant", "content": response.content},
      ]

      # 发送继续对话的请求
      continuation = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=4096,
          messages=messages,
          tools=[{"type": "web_search_20250305", "name": "web_search", "max_uses": 10}],
      )

      print(continuation)
  else:
      print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 带有网络搜索的初始请求
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content:
          "Search for comprehensive information about quantum computing breakthroughs in 2025"
      }
    ],
    tools: [
      {
        type: "web_search_20250305",
        name: "web_search",
        max_uses: 10
      }
    ]
  });

  // 检查响应的停止原因是否为 pause_turn
  if (response.stop_reason === "pause_turn") {
    // 使用已暂停的内容继续对话
    const messages: Anthropic.MessageParam[] = [
      {
        role: "user",
        content:
          "Search for comprehensive information about quantum computing breakthroughs in 2025"
      },
      { role: "assistant", content: response.content }
    ];

    // 发送继续请求
    const continuation = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 4096,
      messages,
      tools: [
        {
          type: "web_search_20250305",
          name: "web_search",
          max_uses: 10
        }
      ]
    });

    console.log(continuation);
  } else {
    console.log(response);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [
          new UserMessageParam
          {
              Content = "Search for comprehensive information about quantum computing breakthroughs in 2025"
          }
      ],
      Tools = [new ToolUnion(new WebSearchTool20250305 { MaxUses = 10 })]
  };

  var response = await client.Messages.Create(parameters);

  if (response.StopReason?.Value() == StopReason.PauseTurn)
  {
      // 使用已暂停的内容继续对话
      var continuationParams = new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 4096,
          Messages = [
              new UserMessageParam
              {
                  Content = "Search for comprehensive information about quantum computing breakthroughs in 2025"
              },
              new AssistantMessageParam
              {
                  Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList()
              }
          ],
          Tools = [new ToolUnion(new WebSearchTool20250305 { MaxUses = 10 })]
      };

      var continuation = await client.Messages.Create(continuationParams);
      Console.WriteLine(continuation);
  }
  else
  {
      Console.WriteLine(response);
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  webSearchTool := []anthropic.ToolUnionParam{
  	{OfWebSearchTool20250305: &anthropic.WebSearchTool20250305Param{
  		MaxUses: anthropic.Int(10),
  	}},
  }

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Search for comprehensive information about quantum computing breakthroughs in 2025")),
  	},
  	Tools: webSearchTool,
  })
  if err != nil {
  	log.Fatal(err)
  }

  if response.StopReason == anthropic.StopReasonPauseTurn {
  	// 将暂停的响应原样传回，以便 Claude 继续该轮次
  	continuation, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 4096,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Search for comprehensive information about quantum computing breakthroughs in 2025")),
  			response.ToParam(),
  		},
  		Tools: webSearchTool,
  	})
  	if err != nil {
  		log.Fatal(err)
  	}
  	fmt.Println(continuation)
  } else {
  	fmt.Println(response)
  }
  ```

  ```java Java
  import com.anthropic.models.messages.StopReason;
  import com.anthropic.models.messages.WebSearchTool20250305;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Search for comprehensive information about quantum computing breakthroughs in 2025")
          .addTool(WebSearchTool20250305.builder()
              .maxUses(10L)
              .build())
          .build();

      Message response = client.messages().create(params);

      if (response.stopReason().isPresent()
              && response.stopReason().get().equals(StopReason.PAUSE_TURN)) {
          MessageCreateParams continuationParams = MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addUserMessage("Search for comprehensive information about quantum computing breakthroughs in 2025")
              .addMessage(response)
              .addTool(WebSearchTool20250305.builder()
                  .maxUses(10L)
                  .build())
              .build();

          Message continuation = client.messages().create(continuationParams);
          IO.println(continuation);
      } else {
          IO.println(response);
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => 'Search for comprehensive information about quantum computing breakthroughs in 2025'
          ]
      ],
      model: 'claude-opus-5-5',
      tools: [
          [
              'type' => 'web_search_20250305',
              'name' => 'web_search',
              'max_uses' => 10
          ]
      ],
  );

  if ($response->stopReason === 'pause_turn') {
      $messages = [
          [
              'role' => 'user',
              'content' => 'Search for comprehensive information about quantum computing breakthroughs in 2025'
          ],
          [
              'role' => 'assistant',
              'content' => $response->content
          ]
      ];

      $continuation = $client->messages->create(
          maxTokens: 4096,
          messages: $messages,
          model: 'claude-opus-5-5',
          tools: [
              [
                  'type' => 'web_search_20250305',
                  'name' => 'web_search',
                  'max_uses' => 10
              ]
          ],
      );

      echo $continuation;
  } else {
      echo $response;
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content:
          "Search for comprehensive information about quantum computing breakthroughs in 2025"
      }
    ],
    tools: [
      {
        type: "web_search_20250305",
        name: "web_search",
        max_uses: 10
      }
    ]
  )

  if response.stop_reason == :pause_turn
    messages = [
      {
        role: "user",
        content: "Search for comprehensive information about quantum computing breakthroughs in 2025"
      },
      {
        role: "assistant",
        content: response.content
      }
    ]

    continuation = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 4096,
      messages: messages,
      tools: [
        {
          type: "web_search_20250305",
          name: "web_search",
          max_uses: 10
        }
      ]
    )

    puts continuation
  else
    puts response
  end
  ```
</CodeGroup>

处理 `pause_turn` 时：

* **继续对话：** 在后续请求中将暂停的响应原样传回，让 Claude 继续其轮次。
* **保留工具状态：** 在续接请求中包含相同的工具。暂停的轮次可能以一个其工具尚未运行的 `server_tool_use` 块结束，如果续接请求中缺少该工具，API 会返回验证错误。
* **保留容器：** 如果响应中包含 `container`（例如使用[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)时），请在续接请求中传回其 ID。如果不传回该 ID，请求将在新容器中运行，该轮次中先前产生的文件都会丢失。
* **按需重复：** 续接的轮次可能会再次暂停。检查每个响应的 `stop_reason`，并持续续接直到获得不同的停止原因，同时像对待任何重试循环一样限制续接次数。请按顺序将每个暂停的响应保留在 `messages` 中。续接只包含新内容，因此如果请求遗漏了先前某个暂停的响应，就会丢失相应的工作成果，并可能返回 400 错误。

有关其他 `stop_reason` 值和通用处理模式，请参阅[停止原因与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)。

## 在一个轮次中混合使用服务器工具和客户端工具

Claude 可以在同一组并行工具调用中同时调用服务器工具和客户端工具，例如将 `web_fetch` 与用户定义的工具一起调用。客户端工具是指由您的代码运行并产生 `tool_use` 块的任何工具，无论它是用户定义的工具，还是 Anthropic 架构的客户端工具（例如 [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)）。发生这种情况时，API 不会运行服务器工具，而是立即返回，以便您先运行客户端工具：

* `stop_reason` 为 `"tool_use"`，而不是 `"pause_turn"`。
* `content` 包含 `server_tool_use` 块和客户端 `tool_use` 块，但没有服务器工具的结果块：该调用尚未完成。
* 没有其他标记。您可以通过查找响应中其 `id` 没有匹配结果块的 `server_tool_use` 块来检测这种状态。来自 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)的 `mcp_tool_use` 块的行为与此相同。在同一响应中已有结果块的服务器工具调用已经完成，无需您进行任何处理。

<Note>
  使用[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)时，相同的响应结构含义不同。客户端 `tool_use` 块来自在 `code_execution` 工具中运行的代码，而不是直接来自 Claude，其 `caller` 字段指明了调用它的 `code_execution` 块。该代码已经开始运行：它处于暂停状态，等待您的 `tool_result` 块，发送这些块会恢复执行，而不是启动一个被延迟的工具。`code_execution` 块自身的结果块会在代码完成后到达，这可能需要不止一轮工具结果。两种情况下的后续用户消息本身是相同的；使用程序化工具调用时，还需如该页面所示传回响应中 `container` 字段的 `id`。
</Note>

```json
{
  "stop_reason": "tool_use",
  "content": [
    {
      "type": "text",
      "text": "I'll fetch the article and check your system at the same time."
    },
    {
      "type": "server_tool_use",
      "id": "srvtoolu_01HxbWnMRmbWyMfUtJKC45rA",
      "name": "web_fetch",
      "input": { "url": "https://example.com/article" }
    },
    {
      "type": "tool_use",
      "id": "toolu_01PjgRJLbXrXEMZwDNYLnBqk",
      "name": "run_command",
      "input": { "command": "uname -a" }
    }
  ]
}
```

要继续该轮次，请运行客户端工具，并发送一条内容仅包含 `tool_result` 块的用户消息，为该响应中的每个 `tool_use` 块各提供一个。保持相同的 `tools` 数组：如果恢复请求不再定义正在等待的服务器工具，则会失败并返回 400 错误，其消息以 ``but no `web_fetch` tool was provided`` 结尾。

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01PjgRJLbXrXEMZwDNYLnBqk",
      "content": "Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux"
    }
  ]
}
```

API 会将您的结果附加到仍处于打开状态的助手轮次，运行被延迟的服务器工具（对于已暂停的代码执行，则恢复执行），然后让 Claude 继续。对于 Claude 直接调用的服务器工具，下一个响应以回应上一个响应中 `server_tool_use` `id` 的结果块开头，随后是新生成的内容和新的 `stop_reason`：

```json
{
  "stop_reason": "end_turn",
  "content": [
    {
      "type": "web_fetch_tool_result",
      "tool_use_id": "srvtoolu_01HxbWnMRmbWyMfUtJKC45rA",
      "content": {
        "type": "web_fetch_result",
        "url": "https://example.com/article",
        "content": {
          "type": "document",
          "source": {
            "type": "text",
            "media_type": "text/plain",
            "data": "Full text content of the article..."
          }
        }
      }
    },
    {
      "type": "text",
      "text": "The article argues that... and your machine is running Linux..."
    }
  ]
}
```

`server_tool_use` 块与其结果块通过 `tool_use_id` 配对，而不是通过位置配对：在此流程中，它们分别出现在两个不同的响应中，并且 `server_tool_use` 块不会在第二个响应中重复出现。在后续请求中，请按顺序将整个交互保留在 `messages` 数组中：第一个响应作为 `assistant` 消息，然后是 `tool_result` 用户消息，接着是下一个响应作为另一条 `assistant` 消息，与您累积任何其他工具使用交互的方式相同。

<Warning>
  后续用户消息除 `tool_result` 块外不得包含任何内容。在结果之后添加的块（例如文本）会告诉 API 助手轮次已经结束。对于 Claude 直接调用的服务器工具，这会使该轮次留下一个未解决的服务器工具调用，请求将失败并返回 400 `invalid_request_error`：

  ```text wrap
  `web_fetch` tool use with id `srvtoolu_01HxbWnMRmbWyMfUtJKC45rA` was found without a corresponding `web_fetch_tool_result` block
  ```

  如果后续消息在结果之前放置了内容、只回应了部分客户端 `tool_use` ID，或者根本不包含 `tool_result` 块，则会更早失败，并返回[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)中描述的客户端工具错误：

  ```text wrap
  `tool_use` ids were found without `tool_result` blocks immediately after: toolu_01PjgRJLbXrXEMZwDNYLnBqk. Each `tool_use` block must have a corresponding `tool_result` block in the next message.
  ```

  如需向 Claude 提供更多输入，请在该轮次完成后将其作为单独的用户消息发送。
</Warning>

**与 `pause_turn` 的区别：** [`pause_turn` 响应](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#the-server-side-loop-and-pause-turn)也可能以一个尚未运行的 `server_tool_use` 块结束，但它绝不会留下等待您处理的客户端 `tool_use` 块，因此您可以通过原样重新发送助手内容来续接。留下等待您处理的客户端 `tool_use` 块的响应，其 `stop_reason` 绝不会是 `pause_turn`：当 Claude 停下来调用您的工具时，`stop_reason` 为 `tool_use`，您需要将响应连同一条包含客户端 `tool_result` 块的新用户消息一起传回来续接。在这两种情况下，API 都会在下一个请求开始时运行待处理的服务器工具。

以下示例同时启用了网络获取和用户定义的 `run_command` 工具，并处理混合响应：

<CodeGroup>
  ```bash cURL
  # 如果 "stop_reason" 为 "tool_use"，且某个 server_tool_use 块没有匹配的
  # 结果块，则该调用尚未完成。请运行客户端工具，然后再次 POST，
  # 并附上 assistant 内容、另一条仅包含这些工具的
  # tool_result 块的 user 消息，以及相同的 tools 数组。
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Summarize https://example.com/article and run uname -a to tell me what system this is on."
        }
      ],
      "tools": [
        {"type": "web_fetch_20250910", "name": "web_fetch", "max_uses": 5},
        {
          "name": "run_command",
          "description": "Run a shell command on this computer and return its output.",
          "input_schema": {
            "type": "object",
            "properties": {"command": {"type": "string", "description": "The command to run"}},
            "required": ["command"]
          }
        }
      ]
    }'
  ```

  ```bash CLI
  # 如果 "stop_reason" 为 "tool_use"，且某个 server_tool_use 块没有匹配的
  # 结果块，请运行客户端工具，然后追加 assistant 内容和一条仅包含
  # 这些工具的 tool_result 块的用户消息并重新运行。
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: "Summarize https://example.com/article and run uname -a to tell me what system this is on."
  tools:
    - {type: web_fetch_20250910, name: web_fetch, max_uses: 5}
    - name: run_command
      description: Run a shell command on this computer and return its output.
      input_schema:
        type: object
        properties:
          command: {type: string, description: The command to run}
        required: [command]
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  tools = [
      {"type": "web_fetch_20250910", "name": "web_fetch", "max_uses": 5},
      {
          "name": "run_command",
          "description": "Run a shell command on this computer and return its output.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "command": {"type": "string", "description": "The command to run"}
              },
              "required": ["command"],
          },
      },
  ]
  messages = [
      {
          "role": "user",
          "content": "Summarize https://example.com/article and run uname -a to tell me what system this is on.",
      }
  ]

  response = client.messages.create(
      model="claude-opus-5-5", max_tokens=4096, tools=tools, messages=messages
  )

  tool_results = [
      {
          "type": "tool_result",
          "tool_use_id": block.id,
          # 在此处运行您的工具。此示例返回一个固定字符串。
          "content": "Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux",
      }
      for block in response.content
      if block.type == "tool_use"
  ]

  if response.stop_reason == "tool_use" and tool_results:
      # 若本次响应中的 server_tool_use 块没有对应的结果块，则表示其尚未完成；其结果将在后续响应中返回。
      # 仅回传客户端 tool_result 块，并使用相同的工具。
      continuation = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=4096,
          tools=tools,
          messages=[
              *messages,
              {"role": "assistant", "content": response.content},
              {"role": "user", "content": tool_results},
          ],
      )
      # 如果某个 web_fetch 被延迟执行，它将在本次请求中运行，其
      # web_fetch_tool_result 是 continuation.content 的第一个块。
      print(continuation)
  else:
      print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const webFetchTool = {
    type: "web_fetch_20250910",
    name: "web_fetch",
    max_uses: 5
  } as const;
  const runCommandTool: Anthropic.Tool = {
    name: "run_command",
    description: "Run a shell command on this computer and return its output.",
    input_schema: {
      type: "object" as const,
      properties: {
        command: { type: "string", description: "The command to run" }
      },
      required: ["command"]
    }
  };
  const messages: Anthropic.MessageParam[] = [
    {
      role: "user",
      content:
        "Summarize https://example.com/article and run uname -a to tell me what system this is on."
    }
  ];

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    tools: [webFetchTool, runCommandTool],
    messages
  });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const block of response.content) {
    if (block.type === "tool_use") {
      toolResults.push({
        type: "tool_result",
        tool_use_id: block.id,
        // 在此运行您的工具。本示例返回一个固定字符串。
        content: "Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux"
      });
    }
  }

  if (response.stop_reason === "tool_use" && toolResults.length > 0) {
    // 如果本次响应中的 server_tool_use 块没有对应的结果块，则表示它尚未完成；其结果将在后续响应中返回。
    // 仅发回客户端的 tool_result 块，并使用相同的工具。
    const continuation = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 4096,
      tools: [webFetchTool, runCommandTool],
      messages: [
        ...messages,
        { role: "assistant", content: response.content },
        { role: "user", content: toolResults }
      ]
    });
    // 如果某个 web_fetch 被延迟，它将在本次请求中运行，其
    // web_fetch_tool_result 是 continuation.content 的第一个块。
    console.log(continuation);
  } else {
    console.log(response);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  List<ToolUnion> tools =
  [
      new ToolUnion(new WebFetchTool20250910() { MaxUses = 5 }),
      new ToolUnion(new Tool()
      {
          Name = "run_command",
          Description = "Run a shell command on this computer and return its output.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["command"] = JsonSerializer.SerializeToElement(
                      new { type = "string", description = "The command to run" }
                  ),
              },
              Required = ["command"],
          },
      }),
  ];
  MessageParam userMessage = new UserMessageParam
  {
      Content = "Summarize https://example.com/article and run uname -a to tell me what system this is on."
  };

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Tools = tools,
      Messages = [userMessage]
  });

  var toolResults = new List<ContentBlockParam>();
  foreach (var block in response.Content)
  {
      if (block.TryPickToolUse(out var toolUse))
      {
          toolResults.Add(new ContentBlockParam(new ToolResultBlockParam()
          {
              ToolUseID = toolUse.ID,
              // 在此运行您的工具。本示例返回一个固定字符串。
              Content = "Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux",
          }));
      }
  }

  if (response.StopReason?.Value() == StopReason.ToolUse && toolResults.Count > 0)
  {
      // 若本次响应中的 server_tool_use 块没有对应的结果块，则表示其尚未完成；其结果将在后续响应中返回。
      // 仅发回客户端的 tool_result 块，并使用相同的工具。
      var continuation = await client.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 4096,
          Tools = tools,
          Messages =
          [
              userMessage,
              new AssistantMessageParam
              {
                  Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList()
              },
              new UserMessageParam { Content = toolResults }
          ]
      });
      // 如果某个 web_fetch 被推迟，它将在本次请求中运行，其
      // web_fetch_tool_result 是 continuation.Content 的第一个块。
      Console.WriteLine(continuation);
  }
  else
  {
      Console.WriteLine(response);
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  tools := []anthropic.ToolUnionParam{
  	{OfWebFetchTool20250910: &anthropic.WebFetchTool20250910Param{
  		MaxUses: anthropic.Int(5),
  	}},
  	{OfTool: &anthropic.ToolParam{
  		Name:        "run_command",
  		Description: anthropic.String("Run a shell command on this computer and return its output."),
  		InputSchema: anthropic.ToolInputSchemaParam{
  			Properties: map[string]any{
  				"command": map[string]any{
  					"type":        "string",
  					"description": "The command to run",
  				},
  			},
  			Required: []string{"command"},
  		},
  	}},
  }
  userMessage := anthropic.NewUserMessage(anthropic.NewTextBlock("Summarize https://example.com/article and run uname -a to tell me what system this is on."))

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Tools:     tools,
  	Messages:  []anthropic.MessageParam{userMessage},
  })
  if err != nil {
  	log.Fatal(err)
  }

  var toolResults []anthropic.ContentBlockParamUnion
  for _, block := range response.Content {
  	if toolUse, ok := block.AsAny().(anthropic.ToolUseBlock); ok {
  		// 在此运行您的工具。本示例返回一个固定字符串。
  		output := "Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux"
  		toolResults = append(toolResults, anthropic.NewToolResultBlock(toolUse.ID, output, false))
  	}
  }

  if response.StopReason == anthropic.StopReasonToolUse && len(toolResults) > 0 {
  	// 如果此响应中的 server_tool_use 块没有对应的结果块，则表示它尚未完成；其结果将在后续响应中返回。
  	// 仅发回客户端 tool_result 块，并使用相同的工具。
  	continuation, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 4096,
  		Tools:     tools,
  		Messages: []anthropic.MessageParam{
  			userMessage,
  			response.ToParam(),
  			anthropic.NewUserMessage(toolResults...),
  		},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}
  	// 如果某个 web_fetch 被延迟，它将在此请求中运行，其
  	// web_fetch_tool_result 是 continuation.Content 的第一个块。
  	fmt.Println(continuation)
  } else {
  	fmt.Println(response)
  }
  ```

  ```java Java
  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Tool runCommandTool = Tool.builder()
          .name("run_command")
          .description("Run a shell command on this computer and return its output.")
          .inputSchema(Tool.InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "command", Map.of("type", "string", "description", "The command to run")
              )))
              .putAdditionalProperty("required", JsonValue.from(List.of("command")))
              .build())
          .build();
      String prompt = "Summarize https://example.com/article and run uname -a to tell me what system this is on.";

      Message response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addTool(WebFetchTool20250910.builder()
              .maxUses(5L)
              .build())
          .addTool(runCommandTool)
          .addUserMessage(prompt)
          .build());

      List<ContentBlockParam> toolResults = new ArrayList<>();
      for (ContentBlock block : response.content()) {
          block.toolUse().ifPresent(toolUse -> toolResults.add(ContentBlockParam.ofToolResult(
              ToolResultBlockParam.builder()
                  .toolUseId(toolUse.id())
                  // 在此运行您的工具。本示例返回一个固定字符串。
                  .content("Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux")
                  .build()
          )));
      }

      boolean isToolUse = response.stopReason()
          .map(StopReason.TOOL_USE::equals)
          .orElse(false);
      if (isToolUse && !toolResults.isEmpty()) {
          // 若本次响应中的 server_tool_use 块没有对应的结果块，则表示其尚未完成；其结果将在后续响应中返回。
          // 仅回传客户端的 tool_result 块，并使用相同的工具。
          Message continuation = client.messages().create(MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(4096L)
              .addTool(WebFetchTool20250910.builder()
                  .maxUses(5L)
                  .build())
              .addTool(runCommandTool)
              .addUserMessage(prompt)
              .addMessage(response)
              .addUserMessageOfContentBlockParams(toolResults)
              .build());
          // 如果某个 web_fetch 被延迟，它会在本次请求中运行，其
          // web_fetch_tool_result 是 continuation.content() 的第一个块。
          IO.println(continuation);
      } else {
          IO.println(response);
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $tools = [
      ['type' => 'web_fetch_20250910', 'name' => 'web_fetch', 'max_uses' => 5],
      [
          'name' => 'run_command',
          'description' => 'Run a shell command on this computer and return its output.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'command' => ['type' => 'string', 'description' => 'The command to run']
              ],
              'required' => ['command']
          ]
      ]
  ];
  $userMessage = ['role' => 'user', 'content' => 'Summarize https://example.com/article and run uname -a to tell me what system this is on.'];

  $response = $client->messages->create(
      maxTokens: 4096,
      messages: [$userMessage],
      model: 'claude-opus-5-5',
      tools: $tools,
  );

  $toolResults = [];
  foreach ($response->content as $block) {
      if ($block->type === 'tool_use') {
          $toolResults[] = [
              'type' => 'tool_result',
              'tool_use_id' => $block->id,
              // 在此运行您的工具。本示例返回一个固定字符串。
              'content' => 'Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux'
          ];
      }
  }

  if ($response->stopReason === 'tool_use' && count($toolResults) > 0) {
      // 若本次响应中的 server_tool_use 块没有对应的结果块，则表示其尚未完成；其结果将在后续响应中返回。
      // 仅发回客户端 tool_result 块，并使用相同的工具。
      $continuation = $client->messages->create(
          maxTokens: 4096,
          messages: [
              $userMessage,
              ['role' => 'assistant', 'content' => $response->content],
              ['role' => 'user', 'content' => $toolResults],
          ],
          model: 'claude-opus-5-5',
          tools: $tools,
      );
      // 如果 web_fetch 被延迟，它将在本次请求中运行，其
      // web_fetch_tool_result 是 $continuation->content 的第一个块。
      echo $continuation;
  } else {
      echo $response;
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  tools = [
    { type: "web_fetch_20250910", name: "web_fetch", max_uses: 5 },
    {
      name: "run_command",
      description: "Run a shell command on this computer and return its output.",
      input_schema: {
        type: "object",
        properties: {
          command: { type: "string", description: "The command to run" }
        },
        required: ["command"]
      }
    }
  ]
  user_message = {
    role: "user",
    content: "Summarize https://example.com/article and run uname -a to tell me what system this is on."
  }

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    tools: tools,
    messages: [user_message]
  )

  tool_results = []
  response.content.each do |block|
    next unless block.type == :tool_use

    tool_results << {
      type: "tool_result",
      tool_use_id: block.id,
      # 在此处运行您的工具。此示例返回一个固定字符串。
      content: "Linux demo-host 6.8.0-52-generic x86_64 GNU/Linux"
    }
  end

  if response.stop_reason == :tool_use && !tool_results.empty?
    # 若本次响应中的 server_tool_use 块没有对应的结果块，则表示其尚未完成；其结果将在后续响应中返回。
    # 仅发回客户端的 tool_result 块，并使用相同的工具。
    continuation = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 4096,
      tools: tools,
      messages: [
        user_message,
        { role: "assistant", content: response.content },
        { role: "user", content: tool_results }
      ]
    )
    # 如果某个 web_fetch 被推迟，它将在本次请求中运行，其
    # web_fetch_tool_result 是 continuation.content 的第一个块。
    puts continuation
  else
    puts response
  end
  ```
</CodeGroup>

当 Claude 没有混合两种调用时，这段代码同样正确。只有客户端 `tool_use` 块的轮次采用相同的续接路径。只有服务器工具调用的轮次不需要您提供客户端 `tool_result` 块，因为其结果块通常已经存在。如果该轮次以挂起状态返回，例如作为 [`pause_turn` 响应](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#the-server-side-loop-and-pause-turn)，则应按原样重新发送。

## ZDR 与 allowed\_callers

网络搜索（`web_search_20250305`）和网络获取（`web_fetch_20250910`）的基础版本符合[零数据保留（ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)的条件。

带有 “dynamic filtering”（动态过滤）功能的 `_20260209` 及更高版本默认**不**符合 ZDR 条件，因为动态过滤在内部依赖代码执行。

要在 ZDR 下使用 `_20260209` 或更高版本的服务器工具，请在工具上设置 `"allowed_callers": ["direct"]` 以禁用动态过滤：

```json
{
  "type": "web_search_20260209",
  "name": "web_search",
  "allowed_callers": ["direct"]
}
```

这会将该工具限制为仅允许直接调用，从而绕过内部代码执行步骤。

`allowed_callers` 控制工具的调用方式：由 Claude 直接调用（`"direct"`）、从代码执行容器内部调用（例如 `"code_execution_20260120"`），或两者兼有。网络工具的 `_20260209` 版本默认仅允许代码执行调用方；更早的版本默认为 `["direct"]`。在不支持程序化工具调用的模型上，这些版本需要设置 `allowed_callers: ["direct"]`；如果未设置，API 会返回一个提示您进行设置的验证错误。

<Note>
  即使在符合 ZDR 条件的配置中使用网络获取，如果 Claude 从网站获取内容，网站发布者也可能会保留传递给 URL 的任何参数。
</Note>

## 域名过滤

访问网络的服务器工具接受 `allowed_domains` 和 `blocked_domains` 参数，用于控制 Claude 可以访问哪些域名。两者都是工具对象上的字段：

```json
{
  "type": "web_search_20250305",
  "name": "web_search",
  "allowed_domains": ["example.com", "docs.python.org"]
}
```

使用域名过滤器时：

* 域名不应包含 HTTP/HTTPS 协议前缀（使用 `example.com` 而不是 `https://example.com`）。
* 子域名会被自动包含（`example.com` 涵盖 `docs.example.com`）。
* 指定子域名会将结果限制为仅该子域名（`docs.example.com` 仅返回来自该子域名的结果，而不返回来自 `example.com` 或 `api.example.com` 的结果）。
* 网络搜索支持子路径，并匹配该路径之后的任何内容（`example.com/blog` 匹配 `example.com/blog/post-1`）。
* 网络获取仅按域名匹配：包含路径的条目永远不会匹配网络获取的 URL。
* 您可以使用 `allowed_domains` 或 `blocked_domains`，但不能在同一请求中同时使用两者。

**通配符支持：**

* 通配符（`*`）不允许出现在域名本身中，只能出现在域名之后的路径中。
* 有效：`example.com/*`、`example.com/*/articles`
* 无效：`*.example.com`、`ex*.com`

无效的域名格式会在请求时被拒绝，并返回 400 `invalid_request_error`。

<Note>
  请求级别的域名限制与在 Claude Console 中配置的任何组织级别域名限制协同工作。请求级别的 `allowed_domains` 必须是组织级别允许列表的子集；超出该列表的条目会导致 API 返回验证错误。如果请求级别的允许列表包含您的组织所阻止的域名，请求将被拒绝并返回 `400` 错误，错误中会列出存在冲突的条目。
</Note>

<Warning>
  域名中的 Unicode 字符可能通过 “homograph attack”（同形异义字攻击）绕过域名过滤器：`аmazon.com`（包含西里尔字母 `а`）看起来与 `amazon.com` 完全相同，但却是不同的域名。请在允许列表和屏蔽列表中仅使用 ASCII 域名，并审查现有条目中是否存在非 ASCII 字符。
</Warning>

有关 Claude Managed Agents 会话中的域名列表和您组织的设置，请参阅[限制网络搜索和网络获取的域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

## 结合代码执行的动态过滤

网络搜索和网络获取的 `_20260209` 及更高版本在内部使用代码执行，对搜索结果应用动态过滤器。

<Note>
  对于这些版本，您无需添加 `code_execution` 工具：当动态过滤运行时，API 会自动为该请求配置代码执行，并且两个工具共享同一个执行容器。如果您确实包含了代码执行工具，请使用 `code_execution_20260120` 或更高版本；API 会拒绝与这些网络工具版本一起使用的旧版代码执行工具。
</Note>

## 流式传输服务器工具事件

服务器工具事件作为常规 “server-sent events”（服务器发送事件），即 SSE 流的一部分进行流式传输。Claude 直接调用的 `server_tool_use` 块的流式传输方式与客户端 `tool_use` 块相同：先是一个 `content_block_start` 事件，随后是 `input_json_delta` 事件。结果块会在单个 `content_block_start` 事件中完整到达，不包含增量。

有关完整的事件参考，请参阅[流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)。各个工具页面会记录与此不同的工具特定事件名称。

## 批量请求

所有服务器工具都支持批量处理。在批处理中，智能体循环的运行方式与同步请求相同，但每个轮次的迭代上限更高。如果循环达到该上限，响应将以 `stop_reason: "pause_turn"` 结束；您可以通过提交包含返回内容的后续请求来续接。有关详细信息，请参阅[服务器工具与智能体循环](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#server-tools-and-the-agentic-loop)。

常见的批处理工作负载包括：使用来自网络的信息丰富数据集、根据最新来源核查大量文档，以及对大量文件运行分析代码。

## 后续步骤

<CardGroup cols={2}>
  <Card title="工具使用故障排除" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/troubleshooting-tool-use">
    通过从症状到修复方法的诊断表，修复最常见的工具使用错误。
  </Card>

  <Card title="网络搜索工具" icon="magnifying-glass" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool">
    搜索网络并引用结果。
  </Card>

  <Card title="网络获取工具" icon="download" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool">
    从特定 URL 获取并读取内容，用实时网络内容增强 Claude 的上下文。
  </Card>

  <Card title="代码执行工具" icon="terminal" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool">
    在沙盒容器中运行 Python 和 Bash 代码，以分析数据、生成文件并迭代解决方案。
  </Card>

  <Card title="工具搜索工具" icon="compass" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool">
    按需发现和加载工具。
  </Card>
</CardGroup>
