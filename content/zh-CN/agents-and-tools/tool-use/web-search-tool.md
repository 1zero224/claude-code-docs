---
title: 网页搜索工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool
description: 让 Claude 访问最新的网络内容，并提供引用来源、可选的动态过滤和域名控制。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 实时网页搜索结果在 API 响应中返回。网页搜索的动态过滤不具备 ZDR 或 HIPAA 资格。
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-fable-5
    - claude-mythos-5
    - claude-mythos-preview
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-sonnet-5
    - claude-haiku-5-5
    - claude-haiku-4-5-20251001
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API: stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed):
      availability: stable
      note: 仅支持基础网页搜索工具（`web_search_20250305`），不支持动态过滤。
    Microsoft Foundry:
      availability: stable
      note: 仅支持基础网页搜索工具（`web_search_20250305`），不支持动态过滤。
---

**API 支持**：用法因产品而异。

* **Messages API**：在请求的 `tools` 中添加 `web_search` 工具。`max_uses`、域名列表和 `user_location` 等设置放在该工具上。
* **Managed Agents**：在智能体工具集中默认开启，不支持 `max_uses`、`citations` 或 `cache_control`。

网络搜索工具让 Claude 能够直接访问实时网络内容，使其能够利用超出其知识截止日期的最新信息来回答问题。响应中包含从搜索结果中提取的来源引用。

使用 `web_search_20260209` 及更高版本时，Claude 可以编写并运行代码，在搜索结果到达 “context window”（上下文窗口）之前对其进行过滤（**dynamic filtering**，动态过滤），只保留相关信息。动态过滤适用于 Claude 4.6 及更高版本的模型以及 [Claude Mythos Preview](https://anthropic.com/glasswing)。

网络搜索工具有三个可用版本：

* `web_search_20250305`：基础网络搜索
* `web_search_20260209`：增加了[动态过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#dynamic-filtering)
* `web_search_20260318`：增加了面向智能体工作流的[响应包含](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#response-inclusion)控制

本页示例使用 `web_search_20250305` 进行基础搜索，使用 `web_search_20260318` 进行动态过滤。

<Note>
  对于 [Claude Mythos Preview](https://anthropic.com/glasswing)，网络搜索在 Claude API、Claude Models on Google Cloud (Google managed) 和 Microsoft Foundry 上受支持。
</Note>

有关网络搜索的零数据保留（Zero Data Retention）资格以及相关的 `allowed_callers` 配置，请参阅[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#zdr-and-allowed-callers)。

有关模型支持情况，请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)。

要在发送请求之前检查某个模型是否接受网络搜索，请从 Models API 读取其 `capabilities.server_tools.web_search.supported` 值。[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api) 介绍了该字段。

## 网络搜索的工作原理

当您将网络搜索工具添加到 API 请求中时：

1. Claude 根据提示决定何时进行搜索。
2. API 执行搜索并将结果提供给 Claude。此过程可能在单个请求中重复多次。
3. 在其轮次结束时，Claude 提供带有引用来源的最终响应。

### Claude 何时搜索

当请求依赖于最新的、不断变化的或超出其训练数据的信息时，Claude 会进行搜索：

* 近期事件、新闻或公告
* 当前价格、汇率、比分或统计数据
* 可能已发生变化的特定组织、人物或产品的信息
* 明确要求搜索或查找某些内容的请求

当请求依赖于稳定的知识时，Claude 会直接回答而不进行搜索：

* 既定事实、数学、科学基础知识或编程概念
* 创意写作或头脑风暴
* 对对话中已提供内容的分析
* 对话轮次和问候

可以通过您的系统提示来引导触发行为：您可以鼓励 Claude 更积极地搜索，或者倾向于直接回答。如需强制限制，请使用 [`max_uses`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#max-uses) 限制一个轮次中的搜索次数。

### 动态过滤

使用基础网络搜索时，每个搜索结果都会被加载到 Claude 的上下文窗口中，而其中许多内容可能与请求无关。使用 `web_search_20260209` 或更高版本时，Claude 会先编写并运行代码来过滤结果，因此只有相关内容才会进入上下文窗口。这可以减少搜索密集型请求的令牌使用量。

动态过滤从[代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)内部运行网络搜索：在 `web_search_20260209` 及更高版本上，该工具的 `allowed_callers` 字段默认为 `["code_execution_20260120"]`，并且在运行动态过滤时，API 会自动为请求提供所需的代码执行。您无需自行将代码执行工具添加到 `tools` 中。以这种方式进行的代码执行调用除标准令牌费用外不产生额外费用。

要直接调用网络搜索而不使用动态过滤，请设置 `allowed_callers: ["direct"]`。不支持程序化工具调用的模型需要此设置。如果没有此设置，API 会返回 400 错误，提示您进行设置。

<Note>
  网络搜索工具（带或不带动态过滤）可在 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 和 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。在 Microsoft Foundry 上，[托管在 Azure 上](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry#additional-features-not-supported-when-hosted-on-azure)的部署仅支持基础网络搜索工具（`web_search_20250305`，不带动态过滤）。托管在 Anthropic 上的部署支持所有版本。在 Claude Models on Google Cloud (Google managed) 上，仅提供基础网络搜索工具（不带动态过滤）。
</Note>

以下示例使用 `web_search_20260318`：

<CodeGroup>
  ```bash cURL
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
          "content": "Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio."
        }
      ],
      "tools": [{
        "type": "web_search_20260318",
        "name": "web_search"
      }]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: >-
        Search for the current prices of AAPL and GOOGL, then calculate
        which has a better P/E ratio.
  tools:
    - type: web_search_20260318
      name: web_search
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio.",
          }
      ],
      tools=[{"type": "web_search_20260318", "name": "web_search"}],
  )
  print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content:
          "Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio."
      }
    ],
    tools: [{ type: "web_search_20260318", name: "web_search" }]
  });

  console.log(response);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [new UserMessageParam { Content = "Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio." }],
      Tools = [new ToolUnion(new WebSearchTool20260318())]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio.")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfWebSearchTool20260318: &anthropic.WebSearchTool20260318Param{}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.messages.WebSearchTool20260318;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio.")
          .addTool(WebSearchTool20260318.builder().build())
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          ['role' => 'user', 'content' => 'Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio.'],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          [
              'type' => 'web_search_20260318',
              'name' => 'web_search',
          ],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Search for the current prices of AAPL and GOOGL, then calculate which has a better P/E ratio." }
    ],
    tools: [{
      type: "web_search_20260318",
      name: "web_search"
    }]
  )
  puts message
  ```
</CodeGroup>

## 如何使用网络搜索

<Note>
  网络搜索默认为您的组织启用，除非管理员已在 [Claude Console](https://platform.claude.com/settings/capabilities) 中将其禁用，管理员还可以在该处限制其搜索的域名。如果已禁用，包含该工具的 Messages API 请求将失败，并返回 400 `invalid_request_error`，提示网络搜索未启用，而不是在搜索结果中返回[错误代码](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#errors)。
</Note>

有关您组织的网络搜索设置如何在 Claude Managed Agents 会话中生效，请参阅[限制网络搜索和网络获取域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

在您的 API 请求中提供网络搜索工具：

<CodeGroup>
  ```bash cURL
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
          "content": "What is the weather in NYC?"
        }
      ],
      "tools": [{
        "type": "web_search_20250305",
        "name": "web_search",
        "max_uses": 5
      }]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --message '{role: user, content: What is the weather in NYC?}' \
    --tool '{type: web_search_20250305, name: web_search, max_uses: 5}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[{"role": "user", "content": "What's the weather in NYC?"}],
      tools=[{"type": "web_search_20250305", "name": "web_search", "max_uses": 5}],
  )
  print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "What's the weather in NYC?"
      }
    ],
    tools: [
      {
        type: "web_search_20250305",
        name: "web_search",
        max_uses: 5
      }
    ]
  });

  console.log(response);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [new UserMessageParam { Content = "What's the weather in NYC?" }],
      Tools = [new ToolUnion(new WebSearchTool20250305() { MaxUses = 5 })]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather in NYC?")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfWebSearchTool20250305: &anthropic.WebSearchTool20250305Param{
  			MaxUses: anthropic.Int(5),
  		}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.messages.WebSearchTool20250305;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("What's the weather in NYC?")
          .addTool(WebSearchTool20250305.builder()
              .maxUses(5L)
              .build())
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          ['role' => 'user', 'content' => "What's the weather in NYC?"],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          [
              'type' => 'web_search_20250305',
              'name' => 'web_search',
              'max_uses' => 5,
          ],
      ],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "What's the weather in NYC?" }
    ],
    tools: [{
      type: "web_search_20250305",
      name: "web_search",
      max_uses: 5
    }]
  )
  puts message
  ```
</CodeGroup>

## 工具定义

网络搜索工具支持以下参数：

```json JSON
{
  "type": "web_search_20250305",
  "name": "web_search",

  // Optional: Limit the number of searches
  "max_uses": 5,

  // Optional: Only include results from these domains.
  // Use allowed_domains or blocked_domains, not both.
  "allowed_domains": ["example.com", "trusteddomain.org"],

  // Optional: Never include results from these domains
  "blocked_domains": ["untrustedsource.com"],

  // Optional: Localize search results
  "user_location": {
    "type": "approximate",
    "city": "San Francisco",
    "region": "California",
    "country": "US",
    "timezone": "America/Los_Angeles"
  }
}
```

所有网络搜索工具版本都接受 `allowed_callers`，它控制 Claude 是直接调用网络搜索，还是通过[动态过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#dynamic-filtering)从代码执行中调用。在 `web_search_20260209` 及更高版本上，其默认值为 `["code_execution_20260120"]` 而不是 `["direct"]`。有关如何配置，请参阅 [ZDR 和 allowed\_callers](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#zdr-and-allowed-callers)。`web_search_20260318` 及更高版本还接受 [`response_inclusion`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#response-inclusion)。

### 最大使用次数

`max_uses` 参数限制单个轮次中的搜索次数。单一事实查询通常使用 1–3 次搜索；比较性或多实体研究可能使用 10 次或更多。

失败的搜索也计入限制。如果 Claude 直接调用网络搜索的次数超过允许值，`web_search_tool_result` 将是一个带有 `max_uses_exceeded` 错误代码的错误。如果 Claude 从代码执行中调用网络搜索的次数超过允许值，代码执行运行将以错误结束。

一个轮次可以跨越多个请求。当请求在 [`pause_turn`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#the-server-side-loop-and-pause-turn) 之后恢复，或其最后一条消息仅包含 `tool_result` 块时，该请求会延续一个轮次。该轮次中先前请求的搜索会计入限制，除非 Claude 是通过[动态过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#dynamic-filtering)从代码执行中运行这些搜索的。

### 域名过滤

提供 `allowed_domains` 或 `blocked_domains`，不能同时提供两者。如果请求同时包含两者，API 将返回 400 错误。条目为不带协议方案的裸域名，可附带可选路径，例如 `example.com` 或 `example.com/blog`。

有关完整的域名过滤规则，请参阅服务器工具指南中的[域名过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#domain-filtering)。

在 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 上，请在智能体工具集的 `web_search` 条目上设置这些字段；请参阅[限制网络搜索和网络获取域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

### 本地化

`user_location` 参数允许您根据用户的位置对搜索结果进行本地化。请至少提供 `city`、`region`、`country` 或 `timezone` 中的一项。

* `type`：位置类型（必须为 `approximate`）
* `city`：城市名称
* `region`：地区或州
* `country`：两个字母的 [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) 国家代码（API 会以 400 错误拒绝不受支持的代码）
* `timezone`：[IANA 时区 ID](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

在 Claude Managed Agents 上，智能体工具集的 `web_search` 条目也接受 `user_location`；请参阅[网络工具设置](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#settings)。

### 响应包含

<Note>
  需要 `web_search_20260318` 或更高版本。
</Note>

`response_inclusion` 参数控制当搜索结果被同一响应中已完成的[代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)调用使用时，搜索结果块在 API 响应中的呈现方式。设置 `"response_inclusion": "excluded"` 可将这些嵌套的 `server_tool_use` 和结果块对从响应中完全删除，适用于不需要原始搜索内容的智能体工作流。令牌用量不受影响，因为 Claude 读取的是代码的输出，而不是这些块。默认值为 `"full"`。来自直接调用的结果，或来自在完成前暂停的代码执行调用的结果，API 始终会完整返回，以便您可以在下一个请求中将其发送回去。

```json JSON
{
  "tools": [
    {
      "type": "web_search_20260318",
      "name": "web_search",
      "response_inclusion": "excluded"
    }
  ]
}
```

## 响应

以下是一个响应结构示例：

```json Output
{
  "role": "assistant",
  "content": [
    // 1. Claude's decision to search
    {
      "type": "text",
      "text": "I'll search for when Claude Shannon was born."
    },
    // 2. The search query used
    {
      "type": "server_tool_use",
      "id": "srvtoolu_01WYG3ziw53XMcoyKL4XcZmE",
      "name": "web_search",
      "input": {
        "query": "claude shannon birth date"
      }
    },
    // 3. Search results
    {
      "type": "web_search_tool_result",
      "tool_use_id": "srvtoolu_01WYG3ziw53XMcoyKL4XcZmE",
      "content": [
        {
          "type": "web_search_result",
          "url": "https://en.wikipedia.org/wiki/Claude_Shannon",
          "title": "Claude Shannon - Wikipedia",
          "encrypted_content": "EqgfCioIARgBIiQ3YTAwMjY1Mi1mZjM5LTQ1NGUtODgxNC1kNjNjNTk1ZWI3Y...",
          "page_age": "April 30, 2025"
        }
      ]
    },
    {
      "text": "Based on the search results, ",
      "type": "text"
    },
    // 4. Claude's response with citations
    {
      "text": "Claude Shannon was born on April 30, 1916, in Petoskey, Michigan",
      "type": "text",
      "citations": [
        {
          "type": "web_search_result_location",
          "url": "https://en.wikipedia.org/wiki/Claude_Shannon",
          "title": "Claude Shannon - Wikipedia",
          "encrypted_index": "Eo8BCioIAhgBIiQyYjQ0OWJmZi1lNm..",
          "cited_text": "Claude Elwood Shannon (April 30, 1916 – February 24, 2001) was an American mathematician, electrical engineer, computer scientist, cryptographer and i..."
        }
      ]
    }
  ],
  "id": "msg_a930390d3a",
  "usage": {
    "input_tokens": 6039,
    "output_tokens": 931,
    "server_tool_use": {
      "web_search_requests": 1
    }
  },
  "stop_reason": "end_turn"
}
```

此示例展示的是直接搜索。当搜索通过[动态过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool#dynamic-filtering)运行时，响应还会包含[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)的结果块，并且每个嵌套的 `server_tool_use` 和 `web_search_tool_result` 对都带有一个 `caller` 字段，用于标识发起它的代码执行调用。

### 搜索结果

搜索结果包括：

* `url`：来源页面的 URL
* `title`：来源页面的标题
* `page_age`：网站的最后更新时间
* `encrypted_content`：在多轮对话中必须回传的加密内容

要继续包含搜索结果的对话，请将助手的内容块按接收时的原样发送回去，包括每个结果的 `encrypted_content`。API 会在后续请求中解密该内容，以在 Claude 的上下文中恢复搜索结果。如果 `encrypted_content` 缺失或被修改，请求将失败并返回 400 验证错误。

### 引用

网络搜索始终启用引用，每个 `web_search_result_location` 包括：

* `url`：被引用来源的 URL
* `title`：被引用来源的标题
* `encrypted_index`：在多轮对话中必须回传的引用标识
* `cited_text`：最多 150 个字符的被引用内容

网络搜索引用字段 `cited_text`、`title` 和 `url` 不计入输入或输出令牌用量。

<Note>
  当直接向最终用户展示 API 输出时，必须包含指向原始来源的引用。如果您对 API 输出进行修改，包括在向最终用户展示之前对其进行重新处理或与您自己的材料合并，请在咨询您的法律团队后适当地展示引用。
</Note>

### 错误

当网络搜索工具遇到错误（例如达到速率限制）时，Claude API 仍会返回 200（成功）响应。错误在响应正文中使用以下结构表示：

```json Output
{
  "type": "web_search_tool_result",
  "tool_use_id": "srvtoolu_a93jad",
  "content": {
    "type": "web_search_tool_result_error",
    "error_code": "max_uses_exceeded"
  }
}
```

发生错误时，`content` 是单个错误对象，而不是结果块列表。搜索成功但没有匹配结果时，会返回空的 `content` 列表，而不是错误。

以下是可能的错误代码：

* `too_many_requests`：超出速率限制
* `invalid_tool_input`：搜索查询参数无效
* `max_uses_exceeded`：超出网络搜索工具的最大使用次数
* `query_too_long`：查询超出最大长度
* `request_too_large`：搜索请求过大，通常是因为域名过滤列表过长
* `unavailable`：发生内部错误

### `pause_turn` 停止原因

API 可能会暂停长时间运行的搜索轮次并返回 `stop_reason: "pause_turn"`。要继续，请在新请求中将暂停的助手消息原样发送回去。

如果 Claude 在同一组并行工具调用中同时调用了网络搜索和您的某个客户端工具，API 会改为返回 `stop_reason: "tool_use"`，并且暂不运行搜索。要继续，请返回客户端工具结果，API 会在下一个请求中运行搜索。请参阅[在一个轮次中混合使用服务器工具和客户端工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#mixing-server-tools-and-client-tools-in-one-turn)。

有关服务器端循环和 `pause_turn` 处理，请参阅服务器工具指南中的[服务器端循环和 pause\_turn](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#the-server-side-loop-and-pause-turn)。

## 提示缓存

要在多个轮次之间缓存工具定义，请参阅[结合提示缓存的工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching)。

## 流式传输

启用流式传输后，您将在流中收到搜索事件。搜索运行期间会有一段暂停：

```sse Output
event: message_start
data: {"type": "message_start", "message": {"id": "msg_abc123", "type": "message"}}

event: content_block_start
data: {"type": "content_block_start", "index": 0, "content_block": {"type": "text", "text": ""}}

// Claude's decision to search

event: content_block_start
data: {"type": "content_block_start", "index": 1, "content_block": {"type": "server_tool_use", "id": "srvtoolu_xyz789", "name": "web_search"}}

// Search query streamed
event: content_block_delta
data: {"type": "content_block_delta", "index": 1, "delta": {"type": "input_json_delta", "partial_json": "{\"query\":\"latest quantum computing breakthroughs 2025\"}"}}

// Pause while search runs

// Search results streamed
event: content_block_start
data: {"type": "content_block_start", "index": 2, "content_block": {"type": "web_search_tool_result", "tool_use_id": "srvtoolu_xyz789", "content": [{"type": "web_search_result", "title": "Quantum Computing Breakthroughs in 2025", "url": "https://example.com"}]}}

// Claude's response with citations (omitted in this example)
```

## 批量请求

您可以在 [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 中包含网络搜索工具。通过 Message Batches API 进行的网络搜索工具调用与常规 Messages API 请求中的定价相同。

为了保护共享容量，Batches API 会对每个组织的网络搜索请求进行限流，因此包含大量搜索的大型批次可能需要更长时间才能完成。您可以在 Claude Console 的[速率限制](https://platform.claude.com/usage/limits)页面上查看您组织的网络搜索速率限制。如需申请更高的限制，请从该页面联系销售团队。

## 用量和定价

除令牌用量外，网页搜索的使用会额外计费：

```json
{
  "usage": {
    "input_tokens": 105,
    "output_tokens": 6039,
    "cache_read_input_tokens": 7123,
    "cache_creation_input_tokens": 7345,
    "server_tool_use": {
      "web_search_requests": 1
    }
  }
}
```

网络搜索在 Claude API 上的价格为**每 1,000 次搜索 10 美元**，另加搜索生成内容的标准令牌费用。在整个对话过程中检索到的网络搜索结果都会计为输入令牌，包括在单个轮次中运行的搜索迭代以及后续的对话轮次。

每次网页搜索计为一次使用，与返回的结果数量无关。因错误而失败的网页搜索不会计费。

## 后续步骤

<CardGroup cols={3}>
  <Card title="网络获取工具" icon="link" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool">
    从特定 URL 获取并读取内容，用实时网络内容增强 Claude 的上下文。
  </Card>

  <Card title="服务器工具" icon="tool" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools">
    使用由 Anthropic 运行的工具：`server_tool_use` 块、`pause_turn` 续接和域名过滤。
  </Card>

  <Card title="工具参考" icon="book" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    Anthropic 提供的工具目录以及可选工具定义属性的参考。
  </Card>
</CardGroup>

## 可用性

网页搜索工具在所有模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：在请求的 `tools` 中添加 `web_search` 工具。`max_uses`、域名列表和 `user_location` 等设置放在该工具上。

| 状态             | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用             | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Microsoft Foundry (hosted on Anthropic)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |
| 可用<sup>1</sup> | [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry (hosted on Azure)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                  |

1. 仅支持基础网页搜索工具（`web_search_20250305`），不支持动态过滤。

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：属于智能体工具集，添加工具集后默认开启。域名列表和 `user_location` 放在工具集的 `web_search` 条目上；`max_uses`、`citations` 和 `cache_control` 不可用。

| 状态 | 云平台                                                                                                                   |
| -- | --------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA                        |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格                         |
| 详细信息 | 实时网页搜索结果在 API 响应中返回。网页搜索的动态过滤不具备 ZDR 或 HIPAA 资格。                                                                                          | 网页搜索的动态过滤不具备 ZDR 或 HIPAA 资格。 |

适用的端点：

* `/v1/messages` 使用 `web_search` 工具
