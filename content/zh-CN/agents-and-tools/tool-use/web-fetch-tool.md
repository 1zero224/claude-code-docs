---
title: Web fetch 工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool
description: 从特定 URL 获取并读取内容，以实时网页内容增强 Claude 的上下文。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 获取的网页内容在 API 响应中返回。Web fetch 的动态过滤不具备 ZDR 或 HIPAA 资格。网站发布者可能会根据其自身政策保留请求数据（例如获取的 URL 和请求元数据）。
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
    Microsoft Foundry:
      availability: stable
      note: 托管在 Azure 上的部署仅支持 `web_fetch_20250910`：不支持动态过滤、响应包含或缓存绕过。
---

**API 支持**：用法因产品而异。

* **Messages API**：在请求的 `tools` 中添加 `web_fetch` 工具。
* **Managed Agents**：在智能体工具集中默认开启，不支持 `max_uses`、`citations` 或 `cache_control`。

网页获取工具允许 Claude 从指定的网页和 PDF 文档中检索完整内容。

最新的网页获取工具版本（`web_fetch_20260318`）支持 **dynamic filtering**（动态过滤）：Claude 可以编写并执行代码，在获取的内容进入 “context window”（上下文窗口）之前对其进行过滤，仅保留相关信息并丢弃其余部分。这在保持响应质量的同时减少了令牌消耗。动态过滤适用于 Claude 4.6 及更高版本的模型以及 [Claude Mythos Preview](https://anthropic.com/glasswing)。`web_fetch_20260318` 还为智能体工作流添加了[响应包含](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#response-inclusion)控制。其他版本（支持动态过滤和[缓存绕过](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#cache-bypass)的 `web_fetch_20260309`、仅支持动态过滤的 `web_fetch_20260209`、用于基础获取的 `web_fetch_20250910`）均可用。

网页获取（无论是否启用动态过滤）可在 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 和 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。在 Microsoft Foundry 上，[托管在 Azure 上](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry#additional-features-not-supported-when-hosted-on-azure)的部署仅支持基础网页获取工具（`web_fetch_20250910`，不含动态过滤）。托管在 Anthropic 上的部署支持所有版本。

<Note>
  对于 [Claude Mythos Preview](https://anthropic.com/glasswing)，网页获取可在 Claude API 和 Microsoft Foundry 上使用。
</Note>

<Note>
  请使用[反馈表单](https://forms.gle/NhWcgmkcvPCMmPE86)就模型响应的质量、API 本身或文档质量提供反馈。
</Note>

有关零数据保留资格和 `allowed_callers` 变通方法，请参阅[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#zdr-and-allowed-callers)。

<Warning>
  在 Claude 同时处理不受信任的输入和敏感数据的环境中启用网页获取工具会带来数据泄露风险。请仅在受信任的环境中或处理非敏感数据时使用此工具。

  为了将泄露风险降至最低，Claude 无法获取仅出现在其自身输出中的 URL。Claude 只能获取之前在对话中出现过的 URL：用户消息中的 URL、客户端工具结果中的 URL，以及来自之前网页搜索或网页获取结果的 URL（请参阅 [URL 验证](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#url-validation)）。Claude 也无法获取看起来包含凭据（例如 API 密钥或密码）的 URL，除非该凭据出现在系统提示中或用户消息的文本中。

  但是，在使用此工具时，仍存在您应仔细考虑的残留风险。

  如果您担心数据泄露，请考虑：

  * 完全禁用网页获取工具
  * 使用 [`max_uses`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#max-uses) 参数限制单个轮次中的获取次数
  * 使用 `allowed_domains` 参数限制为已知的安全域名
  * 使用 `url_sources` 参数阻止客户端工具结果被计为来源（请参阅 [URL 来源](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#url-sources)）
</Warning>

有关模型支持，请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)。

## 网页获取的工作原理

网页获取是一种[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)：API 在请求期间获取内容并将结果插入对话中。您无需运行任何内容，也无需返回 `tool_result`。例外情况是，当 Claude 在同一组并行工具调用中同时调用网页获取和您的某个客户端工具时：API 会在该获取运行之前返回带有 `stop_reason: "tool_use"` 的响应，然后在您发回客户端 `tool_result` 块时运行该获取。请参阅[在一个轮次中混合使用服务器工具和客户端工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#mixing-server-tools-and-client-tools-in-one-turn)。

当您将网页获取工具添加到 API 请求中时：

1. Claude 根据提示和可用的 URL 决定何时获取内容。
2. API 从指定的 URL 检索完整的文本内容。
3. 对于 PDF，API 以 base64 编码数据的形式返回内容，并像处理直接附加的 PDF 文档一样处理它。
4. Claude 分析获取的内容，并提供带有可选引用的响应。

<Note>
  网页获取工具目前不支持使用 JavaScript 动态渲染的网站。对于需要真实浏览器的页面（JavaScript 渲染、点击或填写表单），请考虑使用[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)，这是一种客户端工具，由您的应用程序驱动浏览器，并将页面文本或屏幕截图作为工具结果返回给 Claude。
</Note>

### Claude 何时获取

当请求指向特定页面或文档时，Claude 会执行获取：

* 对话（或之前的工具结果）中提供了 URL
* 用户在没有 URL 的情况下指明了特定资源（某篇特定文章、README、定价页面或文档章节），并且同时启用了[网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)，使 Claude 能够先定位该资源（请参阅[组合搜索和获取](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#combined-search-and-fetch)）

对于不引用特定页面的常识性或开放式问题，Claude **不会**执行获取。"总结这篇文章：`<url>`"会触发获取。"REST API 设计的最佳实践是什么？"则会被直接回答。

### 动态过滤

获取完整的网页和 PDF 会迅速消耗令牌，尤其是当只需要从大型文档中获取特定信息时。使用 `web_fetch_20260209` 或更高版本时，Claude 可以编写并运行代码，在将获取的内容加载到上下文之前对其进行过滤。

这种动态过滤特别适用于：

* 从长文档中提取特定章节
* 处理来自网页的结构化数据
* 从 PDF 中过滤相关信息
* 在处理大型文档时降低令牌成本

<Note>
  动态过滤在[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)上运行，API 会自动为该请求启用此工具。您无需将代码执行工具添加到 `tools` 数组中。
</Note>

要启用动态过滤，请使用 `web_fetch_20260209` 或任何更高版本。以下示例使用 `web_fetch_20260318`：

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
          "content": "Fetch the content at https://example.com/research-paper and extract the key findings."
        }
      ],
      "tools": [{
        "type": "web_fetch_20260318",
        "name": "web_fetch"
      }]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --message '{role: user, content: "Fetch the content at https://example.com/research-paper and extract the key findings."}' \
    --tool '{type: web_fetch_20260318, name: web_fetch}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Fetch the content at https://example.com/research-paper and extract the key findings.",
          }
      ],
      tools=[{"type": "web_fetch_20260318", "name": "web_fetch"}],
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
          "Fetch the content at https://example.com/research-paper and extract the key findings."
      }
    ],
    tools: [{ type: "web_fetch_20260318", name: "web_fetch" }]
  });

  console.log(response);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [new UserMessageParam { Content = "Fetch the content at https://example.com/research-paper and extract the key findings." }],
      Tools = [new ToolUnion(new WebFetchTool20260318())]
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
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Fetch the content at https://example.com/research-paper and extract the key findings.")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfWebFetchTool20260318: &anthropic.WebFetchTool20260318Param{}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.WebFetchTool20260318;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Fetch the content at https://example.com/research-paper and extract the key findings.")
          .addTool(WebFetchTool20260318.builder().build())
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
          ['role' => 'user', 'content' => 'Fetch the content at https://example.com/research-paper and extract the key findings.']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [[
          'type' => 'web_fetch_20260318',
          'name' => 'web_fetch',
      ]],
  );
  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Fetch the content at https://example.com/research-paper and extract the key findings." }
    ],
    tools: [{
      type: "web_fetch_20260318",
      name: "web_fetch"
    }]
  )
  puts message
  ```
</CodeGroup>

## 如何使用网页获取

<Note>
  组织管理员可以在 Claude Console 的[**功能**](https://platform.claude.com/settings/capabilities)页面上为您的组织关闭网页获取。关闭网页获取后，包含该工具的 Messages API 请求会失败，并返回 400 `invalid_request_error`，提示网页获取未启用。开启网页获取后，管理员还可以在同一页面上为网页获取设置允许或阻止的域名。有关这些列表如何应用于请求，请参阅[域名过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#domain-filtering)。
</Note>

有关您组织的网页获取设置如何在 Claude Managed Agents 会话中生效，请参阅[限制网页搜索和网页获取域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

在您的 API 请求中提供网页获取工具：

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
          "content": "Please analyze the content at https://example.com/article"
        }
      ],
      "tools": [{
        "type": "web_fetch_20250910",
        "name": "web_fetch",
        "max_uses": 5
      }]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --message '{role: user, content: "Please analyze the content at https://example.com/article"}' \
    --tool '{type: web_fetch_20250910, name: web_fetch, max_uses: 5}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Please analyze the content at https://example.com/article",
          }
      ],
      tools=[{"type": "web_fetch_20250910", "name": "web_fetch", "max_uses": 5}],
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
        content: "Please analyze the content at https://example.com/article"
      }
    ],
    tools: [
      {
        type: "web_fetch_20250910",
        name: "web_fetch",
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
      Messages = [new UserMessageParam { Content = "Please analyze the content at https://example.com/article" }],
      Tools = [new ToolUnion(new WebFetchTool20250910() { MaxUses = 5 })]
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
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Please analyze the content at https://example.com/article")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfWebFetchTool20250910: &anthropic.WebFetchTool20250910Param{
  			MaxUses: anthropic.Int(5),
  		}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.WebFetchTool20250910;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Please analyze the content at https://example.com/article")
          .addTool(WebFetchTool20250910.builder()
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
          ['role' => 'user', 'content' => 'Please analyze the content at https://example.com/article']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [[
          'type' => 'web_fetch_20250910',
          'name' => 'web_fetch',
          'max_uses' => 5,
      ]],
  );
  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Please analyze the content at https://example.com/article" }
    ],
    tools: [{
      type: "web_fetch_20250910",
      name: "web_fetch",
      max_uses: 5
    }]
  )
  puts message
  ```
</CodeGroup>

## 工具定义

网页获取工具支持以下参数：

```json JSON
{
  "type": "web_fetch_20250910",
  "name": "web_fetch",

  // Optional: Limit the number of fetches
  "max_uses": 10,

  // Optional: Only fetch from these domains
  "allowed_domains": ["example.com", "docs.example.com"],

  // Optional: Never fetch from these domains (cannot be combined with allowed_domains)
  "blocked_domains": ["private.example.com"],

  // Optional: Don't fetch URLs that appear only in client-side tool results
  "url_sources": {
    "client_tool_results": { "type": "none" }
  },

  // Optional: Enable citations for fetched content
  "citations": {
    "enabled": true
  },

  // Optional: Maximum content length in tokens
  "max_content_tokens": 100000
}
```

更高的工具版本新增了两个可选参数：`use_cache` 需要 `web_fetch_20260309` 或更高版本（请参阅[缓存绕过](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#cache-bypass)），`response_inclusion` 需要 `web_fetch_20260318` 或更高版本（请参阅[响应包含](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#response-inclusion)）。

### 最大使用次数

`max_uses` 参数限制单个轮次中执行的网页获取次数。失败的获取也计入限制。如果 Claude 直接调用网页获取的次数超过允许的次数，`web_fetch_tool_result` 将是一个带有 `max_uses_exceeded` 错误代码的错误。如果 Claude 从代码执行中调用网页获取的次数超过允许的次数，该代码执行运行将以错误结束。没有默认限制。

一个轮次可以跨越多个请求。当请求在 [`pause_turn`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#the-server-side-loop-and-pause-turn) 之后恢复，或者其最后一条消息仅包含 `tool_result` 块时，该请求即为某个轮次的延续。该轮次中较早请求的获取也计入限制，除非 Claude 是通过[动态过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#dynamic-filtering)从代码执行中运行它们的。

### 域名过滤

有关使用 `allowed_domains` 和 `blocked_domains` 进行域名过滤，请参阅[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools#domain-filtering)。

在 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 上，请在智能体工具集的 `web_fetch` 条目上设置这些字段；请参阅[限制网页搜索和网页获取域名](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions)。

### 内容限制

`max_content_tokens` 参数限制包含在上下文中的内容量。如果获取的内容超过此限制，工具会将其截断。这有助于在获取大型文档时控制令牌使用量。该限制是近似值：实际使用的输入令牌数可能会略有不同。该限制适用于文本内容，不适用于 PDF 等二进制内容。

如果获取的 PDF 会导致请求超出 [PDF 页数或大小限制](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support#check-pdf-requirements)，`web_fetch_tool_result` 将是一个带有 `content_too_large` 错误代码的错误。对话中已有的 PDF 和图像也会计入这些限制，因此移除它们可以使获取成功。

在 Claude Managed Agents 上，智能体工具集的 `web_fetch` 条目也接受 `max_content_tokens`；请参阅[网页工具设置](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#settings)。

### 缓存绕过

<Note>
  需要 `web_fetch_20260309` 或更高版本（包括 `web_fetch_20260318`）。
</Note>

`use_cache` 参数控制是否可以返回缓存的内容。设置 `"use_cache": false` 可绕过缓存并获取最新内容。默认值为 `true`。仅当用户明确请求最新内容或获取快速变化的来源时才禁用缓存，因为绕过缓存会增加 “latency”（延迟）。

```json
{
  "tools": [
    {
      "type": "web_fetch_20260309",
      "name": "web_fetch",
      "use_cache": false
    }
  ]
}
```

### 响应包含

<Note>
  需要 `web_fetch_20260318` 或更高版本。
</Note>

`response_inclusion` 参数控制当获取结果在同一响应中被已完成的[代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)调用消费时，获取结果块在 API 响应中的呈现方式。设置 `"response_inclusion": "excluded"` 可将这些嵌套的 `server_tool_use` 和结果块对从响应中完全移除，适用于不需要原始页面内容的智能体工作流。令牌用量不受影响，因为 Claude 读取的是代码的输出，而不是这些块。默认值为 `"full"`。对于来自直接调用的结果，或来自在完成前暂停的代码执行调用的结果，API 始终会完整返回，以便您在下一个请求中将其发回。

```json
{
  "tools": [
    {
      "type": "web_fetch_20260318",
      "name": "web_fetch",
      "response_inclusion": "excluded"
    }
  ]
}
```

### 引用

与始终启用引用的网页搜索不同，网页获取的引用是可选的，且默认禁用。设置 `"citations": {"enabled": true}` 可允许 Claude 引用所获取文档中的特定段落。

<Note>
  当直接向最终用户显示 API 输出时，请包含对原始来源的引用。如果您要修改 API 输出，包括在向最终用户显示之前对其进行再处理或与您自己的材料组合，请在咨询您的法律团队后酌情显示引用。
</Note>

## 响应

以下是一个示例响应结构：

```json Output
{
  "role": "assistant",
  "content": [
    // 1. Claude's decision to fetch
    {
      "type": "text",
      "text": "I'll fetch the content from the article to analyze it."
    },
    // 2. The fetch request
    {
      "type": "server_tool_use",
      "id": "srvtoolu_01234567890abcdef",
      "name": "web_fetch",
      "input": {
        "url": "https://example.com/article"
      }
    },
    // 3. Fetch results
    {
      "type": "web_fetch_tool_result",
      "tool_use_id": "srvtoolu_01234567890abcdef",
      "content": {
        "type": "web_fetch_result",
        "url": "https://example.com/article",
        "content": {
          "type": "document",
          "source": {
            "type": "text",
            "media_type": "text/plain",
            "data": "Full text content of the article..."
          },
          "title": "Article Title",
          "citations": { "enabled": true }
        },
        "retrieved_at": "2025-08-25T10:30:00Z"
      }
    },
    // 4. Claude's analysis with citations (if enabled)
    {
      "text": "Based on the article, ",
      "type": "text"
    },
    {
      "text": "the main argument presented is that artificial intelligence will transform healthcare",
      "type": "text",
      "citations": [
        {
          "type": "char_location",
          "document_index": 0,
          "document_title": "Article Title",
          "start_char_index": 1234,
          "end_char_index": 1456,
          "cited_text": "Artificial intelligence is poised to revolutionize healthcare delivery..."
        }
      ]
    }
  ],
  "id": "msg_a930390d3a",
  "usage": {
    "input_tokens": 25039,
    "output_tokens": 931,
    "server_tool_use": {
      "web_fetch_requests": 1
    }
  },
  "stop_reason": "end_turn"
}
```

### 获取结果

获取结果包括：

* `url`：被获取的 URL
* `content`：包含所获取内容的文档块
* `retrieved_at`：检索内容时的时间戳

<Note>
  网页获取工具会缓存结果，以提高性能并减少冗余请求。返回的内容可能并不总是反映该 URL 上可用的最新版本。缓存行为是自动管理的，并可能随时间变化，以针对不同的内容类型和使用模式进行优化。要获取最新内容，请设置 `"use_cache": false`（请参阅[缓存绕过](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#cache-bypass)）。
</Note>

对于 PDF 文档，内容以 base64 编码数据的形式返回：

```json Output
{
  "type": "web_fetch_tool_result",
  "tool_use_id": "srvtoolu_02",
  "content": {
    "type": "web_fetch_result",
    "url": "https://example.com/paper.pdf",
    "content": {
      "type": "document",
      "source": {
        "type": "base64",
        "media_type": "application/pdf",
        "data": "JVBERi0xLjQKJcOkw7zDtsOfCjIgMCBvYmo..."
      },
      "citations": { "enabled": true }
    },
    "retrieved_at": "2025-08-25T10:30:02Z"
  }
}
```

### 错误

当网页获取工具遇到错误时，Claude API 会返回 200（成功）响应，并在响应正文中表示该错误。Claude 会接收错误结果并继续该轮次。例如：

```json Output
{
  "type": "web_fetch_tool_result",
  "tool_use_id": "srvtoolu_a93jad",
  "content": {
    "type": "web_fetch_tool_result_error",
    "error_code": "url_not_accessible"
  }
}
```

可能的错误代码如下：

* `invalid_tool_input`：无效的工具输入，例如格式错误的 URL 或非 HTTP(S) 协议
* `url_too_long`：URL 超过最大长度（250 个字符）
* `url_not_allowed`：URL 被域名过滤规则（包括您组织的设置）或 Anthropic 端的限制阻止，例如私有地址、`robots.txt` 以及看起来包含您未提供的凭据的 URL
* `url_not_in_prior_context`：URL 之前未在对话中出现，或仅出现在 `url_sources` 排除的来源中（请参阅 [URL 验证](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#url-validation)）
* `url_not_accessible`：获取内容失败（HTTP 错误）
* `too_many_requests`：超过速率限制
* `unsupported_content_type`：不支持的内容类型（仅支持文本、HTML 和 PDF）
* `content_too_large`：获取的 PDF 会导致请求超出 PDF 页数或大小限制（请参阅[内容限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#content-limits)）
* `max_uses_exceeded`：超过网页获取工具的最大使用次数
* `unavailable`：发生内部错误

## URL 验证

出于安全原因，网页获取工具只能获取之前在对话上下文中出现过的 URL。这包括：

* 用户消息中的 URL
* 客户端工具结果中的 URL
* 来自之前网页搜索或网页获取结果的 URL

该工具无法获取仅出现在 Claude 自身输出中或仅出现在系统提示中的 URL。要使系统提示中的 URL 可被获取，请同时将其包含在用户消息中。其他服务器端工具的结果，例如[代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)、[MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)或[工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)，也不是允许的来源。默认情况下，客户端工具结果是允许的来源，即使它们回显了 Claude 生成的文本（例如，打印其输入的命令，或引用该文本的错误消息）。

该工具还会拒绝看起来包含凭据（例如 API 密钥或密码）的 URL，除非该凭据出现在系统提示中或用户消息的文本中。仅出现在工具结果中的凭据不算数。结果是 `url_not_allowed` 错误。要获取此类 URL，请将其包含在用户消息中。

### URL 来源

<Note>
  适用于 `web_fetch_20250910`、`web_fetch_20260209`、`web_fetch_20260309` 和 `web_fetch_20260318`。
</Note>

可选的 `url_sources` 参数允许您关闭三个允许来源中的任何一个，或将工具结果限制为特定工具。它为每个来源提供一个键，每个键的默认值均为 `{"type": "all"}`：

* `user_input`：用户消息中的 URL。其 `type` 为 `"all"` 或 `"none"`。
* `client_tool_results`：客户端工具结果中的 URL。其 `type` 为 `"all"`、`"none"`，或指定工具名称的 `"only"` 或 `"except"` 过滤器，如以下示例所示。
* `server_tool_results`：来自之前网页搜索或网页获取结果的 URL。它接受相同的值，其过滤器指定 `web_search` 或 `web_fetch`。

例如，一个运行 Claude 所编写代码的工具会返回该代码打印的任何内容，包括 Claude 放入其中的任何 URL。要阻止该工具的结果被计为来源，请在 `"except"` 过滤器中指定它：

```json
{
  "tools": [
    {
      "type": "web_fetch_20260318",
      "name": "web_fetch",
      "url_sources": {
        "client_tool_results": {
          "type": "except",
          "tools": [{ "type": "tool_reference", "name": "run_code" }]
        }
      }
    },
    {
      "name": "run_code",
      "description": "Run Python code and return its output",
      "input_schema": {
        "type": "object",
        "properties": { "code": { "type": "string" } },
        "required": ["code"]
      }
    }
  ]
}
```

过滤器以 `tool_reference` 对象的形式列出工具。每个名称都必须与请求的 `tools` 数组中的某个工具匹配：对于 `client_tool_results` 为客户端工具，对于 `server_tool_results` 为服务器工具。否则，请求将失败并返回错误。在 `"except"` 下，您之后添加到 `tools` 的工具会被计为来源。在 `"only"` 下则不会。

除 `run_code` 之外的客户端工具也可能返回 Claude 编写的文本，例如在引用该文本的错误消息中。要排除所有客户端工具结果，请将 `client_tool_results` 设置为 `{"type": "none"}`。

如果某个 URL 仅出现在 `url_sources` 排除的来源中，获取它将返回 `url_not_in_prior_context` 错误。

## 组合搜索和获取

当同时启用网页搜索和网页获取工具，且用户在未提供 URL 的情况下指明了特定页面或文档时（例如，"阅读 anthropics/anthropic-sdk-python 仓库中的 README"），Claude 会使用网页搜索来定位它，然后获取结果。以下示例在一个请求中同时要求搜索和分析：

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
          "content": "Find recent articles about quantum computing and analyze the most relevant one in detail"
        }
      ],
      "tools": [
        {
          "type": "web_search_20250305",
          "name": "web_search",
          "max_uses": 3
        },
        {
          "type": "web_fetch_20250910",
          "name": "web_fetch",
          "max_uses": 5,
          "citations": {"enabled": true}
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content: >-
        Find recent articles about quantum computing
        and analyze the most relevant one in detail
  tools:
    - type: web_search_20250305
      name: web_search
      max_uses: 3
    - type: web_fetch_20250910
      name: web_fetch
      max_uses: 5
      citations:
        enabled: true
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
              "content": "Find recent articles about quantum computing and analyze the most relevant one in detail",
          }
      ],
      tools=[
          {"type": "web_search_20250305", "name": "web_search", "max_uses": 3},
          {
              "type": "web_fetch_20250910",
              "name": "web_fetch",
              "max_uses": 5,
              "citations": {"enabled": True},
          },
      ],
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
          "Find recent articles about quantum computing and analyze the most relevant one in detail"
      }
    ],
    tools: [
      { type: "web_search_20250305", name: "web_search", max_uses: 3 },
      {
        type: "web_fetch_20250910",
        name: "web_fetch",
        max_uses: 5,
        citations: { enabled: true }
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
      Messages = [new UserMessageParam { Content = "Find recent articles about quantum computing and analyze the most relevant one in detail" }],
      Tools = [
          new ToolUnion(new WebSearchTool20250305() { MaxUses = 3 }),
          new ToolUnion(new WebFetchTool20250910() { MaxUses = 5, Citations = new() { Enabled = true } })
      ]
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
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Find recent articles about quantum computing and analyze the most relevant one in detail")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfWebSearchTool20250305: &anthropic.WebSearchTool20250305Param{
  			MaxUses: anthropic.Int(3),
  		}},
  		{OfWebFetchTool20250910: &anthropic.WebFetchTool20250910Param{
  			MaxUses:   anthropic.Int(5),
  			Citations: anthropic.CitationsConfigParam{Enabled: anthropic.Bool(true)},
  		}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.CitationsConfigParam;
  // ...
  import com.anthropic.models.messages.WebFetchTool20250910;
  import com.anthropic.models.messages.WebSearchTool20250305;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Find recent articles about quantum computing and analyze the most relevant one in detail")
          .addTool(WebSearchTool20250305.builder()
              .maxUses(3L)
              .build())
          .addTool(WebFetchTool20250910.builder()
              .maxUses(5L)
              .citations(CitationsConfigParam.builder()
                  .enabled(true)
                  .build())
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
          ['role' => 'user', 'content' => 'Find recent articles about quantum computing and analyze the most relevant one in detail']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          [
              'type' => 'web_search_20250305',
              'name' => 'web_search',
              'max_uses' => 3,
          ],
          [
              'type' => 'web_fetch_20250910',
              'name' => 'web_fetch',
              'max_uses' => 5,
              'citations' => ['enabled' => true],
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
      { role: "user", content: "Find recent articles about quantum computing and analyze the most relevant one in detail" }
    ],
    tools: [
      {
        type: "web_search_20250305",
        name: "web_search",
        max_uses: 3
      },
      {
        type: "web_fetch_20250910",
        name: "web_fetch",
        max_uses: 5,
        citations: { enabled: true }
      }
    ]
  )
  puts message
  ```
</CodeGroup>

在此工作流中，Claude：

1. 使用网页搜索查找相关文章。
2. 选择最有价值的结果。
3. 使用网页获取检索完整内容。
4. 提供带有引用的详细分析。

## 提示缓存

要跨轮次缓存工具定义，请参阅[工具使用与提示缓存](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching)。

## 流式传输

启用 “streaming”（流式传输）后，获取事件是流的一部分，并在内容检索期间出现暂停：

```sse Output
event: message_start
data: {"type": "message_start", "message": {"id": "msg_abc123", "type": "message"}}

event: content_block_start
data: {"type": "content_block_start", "index": 0, "content_block": {"type": "text", "text": ""}}

// Claude's decision to fetch

event: content_block_start
data: {"type": "content_block_start", "index": 1, "content_block": {"type": "server_tool_use", "id": "srvtoolu_xyz789", "name": "web_fetch"}}

// Fetch URL streamed
event: content_block_delta
data: {"type": "content_block_delta", "index": 1, "delta": {"type": "input_json_delta", "partial_json": "{\"url\":\"https://example.com/article\"}"}}

// Pause while fetch runs

// Fetch results streamed
event: content_block_start
data: {"type": "content_block_start", "index": 2, "content_block": {"type": "web_fetch_tool_result", "tool_use_id": "srvtoolu_xyz789", "content": {"type": "web_fetch_result", "url": "https://example.com/article", "content": {"type": "document", "source": {"type": "text", "media_type": "text/plain", "data": "Article content..."}}}}}

// Claude's response continues...
```

## 批量请求

您可以在 [Message Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 中包含网页获取工具。通过 Message Batches API 进行的网页获取工具调用与常规 Messages API 请求中的调用价格相同。

## 用量和定价

Web fetch（网页抓取）的使用除标准令牌费用外**不收取任何额外费用**：

```json
{
  "usage": {
    "input_tokens": 25039,
    "output_tokens": 931,
    "cache_read_input_tokens": 0,
    "cache_creation_input_tokens": 0,
    "server_tool_use": {
      "web_fetch_requests": 1
    }
  }
}
```

Web fetch 工具在 Claude API 上**无需额外费用**即可使用。您只需为成为对话上下文一部分的抓取内容支付标准令牌费用。

为防止无意中抓取会消耗过多令牌的大型内容，请使用 `max_content_tokens` 参数，根据您的使用场景和预算考虑设置适当的限制。

典型内容的令牌使用量示例：

* 普通网页（10 kB）：约 2,500 个令牌
* 大型文档页面（100 kB）：约 25,000 个令牌
* 研究论文 PDF（500 kB）：约 125,000 个令牌

## 后续步骤

<CardGroup>
  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool" title="代码执行工具" icon="code">
    在沙盒容器中运行 Python 和 Bash 代码，以分析数据、生成文件并迭代解决方案。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools" title="服务器工具" icon="cloud">
    使用由 Anthropic 运行的工具：`server_tool_use` 块、`pause_turn` 续接以及域名过滤。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference" title="工具参考" icon="book">
    Anthropic 提供的工具目录以及可选工具定义属性的参考。
  </Card>
</CardGroup>

## 可用性

Web fetch 工具在所有模型上可用，并具备零数据保留资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：在请求的 `tools` 中添加 `web_fetch` 工具。`max_uses`、域名列表、`citations` 和 `max_content_tokens` 等设置放在该工具上。

| 状态             | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用             | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Microsoft Foundry (hosted on Anthropic)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |
| 可用<sup>1</sup> | [Microsoft Foundry (hosted on Azure)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                                                                                                                                                                           |

1. 托管在 Azure 上的部署仅支持 `web_fetch_20250910`：不支持动态过滤、响应包含或缓存绕过。

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：属于智能体工具集，添加工具集后默认开启。域名列表（不含路径）和 `max_content_tokens` 放在工具集的 `web_fetch` 条目上；`max_uses`、`citations` 和 `cache_control` 不可用。

| 状态 | 云平台                                                                                                                   |
| -- | --------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 不具备资格 |
| 详细信息 | 获取的网页内容在 API 响应中返回。Web fetch 的动态过滤不具备 ZDR 或 HIPAA 资格。网站发布者可能会根据其自身政策保留请求数据（例如获取的 URL 和请求元数据）。                                             | —     |

适用的端点：

* `/v1/messages` 使用 `web_fetch` 工具
