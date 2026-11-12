---
title: 浏览器使用工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool
description: 通过浏览器使用工具，让 Claude 在您自己的浏览器环境中导航、阅读网页并与之交互。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 客户端工具。除标准 API 处理外，Anthropic 不运行浏览器操作，也不保留页面内容。
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
    - claude-opus-4-8
  supportedPlatforms:
    Claude API: stable
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

<Note>
  **从 SDK 工具集开始：** Python 和 TypeScript SDK 提供了一个浏览器工具集类，目前处于 beta 阶段。您可以从[合作伙伴的集成](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#partner-integrations)开始，或参阅[使用 SDK 工具集进行浏览器和计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk)。请将本页用作工具参考，或在您自行处理工具调用时使用。
</Note>

浏览器使用工具让 Claude 能够在您的应用程序所运行的浏览器中导航、读取网页并与之交互。Claude 既可以通过页面结构（"accessibility tree"（无障碍树）、元素、表单和标签页）来处理页面，也可以通过屏幕截图和视口坐标来处理页面。

该工具是 Anthropic 定义的[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)：在 `tools` 中添加一个 `browser_toolset_20260801` 条目，默认即可为 Claude 提供 27 个成员工具，例如 `navigate`、`read_page`、`left_click` 和 `screenshot`；当您[启用可选成员](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#enable-optional-member-tools)时，还会再增加四个。您的应用程序会在自己的浏览器自动化环境中执行每一次调用；Anthropic 一侧不执行任何操作。该工具目前在 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/tools) 中不可用。

当任务局限于网页内部并需要对网页执行操作，或者页面使用 JavaScript 构建其内容时，请选择浏览器使用工具。当任务需要整个桌面时，请使用[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)，它仅通过屏幕截图和坐标工作。如果只是读取您可以指引 Claude 访问的页面，或在网络上查找来源，[网页获取工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)和[网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)更为轻量。它们是由 API 为您运行的[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)，无需操作浏览器。

使用浏览器使用时，Claude 会读取实时网页并对其执行操作，因此页面提供的一切都是不受信任的输入，而 Claude 采取的操作可能产生真实的影响。部署之前请参阅[安全注意事项](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#security-considerations)。

## 快速开始

要使用浏览器使用工具，请在 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 请求的 `tools` 数组中添加一个类型为 `browser_toolset_20260801` 的条目，且不带 `name`。有关支持的模型和平台，请参阅 [API 支持](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#availability-feature-support)。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 2048,
      "tools": [
        {
          "type": "browser_toolset_20260801"
        }
      ],
      "messages": [
        {
          "role": "user",
          "content": "Open example.com/docs and tell me how to get started."
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 2048
  tools:
    - type: browser_toolset_20260801
  messages:
    - role: user
      content: Open example.com/docs and tell me how to get started.
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=2048,
      tools=[{"type": "browser_toolset_20260801"}],
      messages=[
          {
              "role": "user",
              "content": "Open example.com/docs and tell me how to get started.",
          }
      ],
  )
  print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 2048,
    tools: [{ type: "browser_toolset_20260801" }],
    messages: [
      {
        role: "user",
        content: "Open example.com/docs and tell me how to get started."
      }
    ]
  });

  console.log(response);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 2048,
      Tools = [new BrowserToolset20260801()],
      Messages =
      [
          new UserMessageParam { Content = "Open example.com/docs and tell me how to get started." },
      ],
  };

  var response = await client.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 2048,
  	Tools: []anthropic.ToolUnionParam{
  		{OfBrowserToolset20260801: &anthropic.BrowserToolset20260801Param{}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Open example.com/docs and tell me how to get started.")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.BrowserToolset20260801;
  // ...

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(2048L)
          .addTool(BrowserToolset20260801.builder().build())
          .addUserMessage("Open example.com/docs and tell me how to get started.")
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 2048,
      messages: [
          ['role' => 'user', 'content' => 'Open example.com/docs and tell me how to get started.'],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          ['type' => 'browser_toolset_20260801'],
      ],
  );

  echo $response;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 2048,
    tools: [
      { type: "browser_toolset_20260801" }
    ],
    messages: [
      {
        role: "user",
        content: "Open example.com/docs and tell me how to get started."
      }
    ]
  )

  puts response
  ```
</CodeGroup>

Claude 的第一个响应以 `stop_reason: "tool_use"` 结束，并携带一个或多个成员 `tool_use` 块，每个块在 `name` 中指明一个成员工具，并携带 `"toolset_name": "browser"`：

```json Output
{
  "id": "msg_01HCDu4XSTLzTAcodEQ58vDo",
  "type": "message",
  "role": "assistant",
  "model": "claude-opus-5-5",
  "content": [
    {
      "type": "text",
      "text": "I'll open the documentation and read the page to find the getting-started instructions."
    },
    {
      "type": "tool_use",
      "id": "toolu_01NRLabsLyVHZPKxbKvkfSMn",
      "name": "navigate",
      "toolset_name": "browser",
      "input": { "url": "https://example.com/docs" }
    },
    {
      "type": "tool_use",
      "id": "toolu_01UvHU5cDyTZ2vXKf5wCkPqR",
      "name": "read_page",
      "toolset_name": "browser",
      "input": { "filter": "interactive" }
    }
  ],
  "stop_reason": "tool_use",
  "stop_sequence": null
}
```

您的 “executor”（执行器，即应用程序中驱动浏览器并生成工具结果的部分）先运行 `navigate`，然后运行 `read_page`。您的应用程序在下一个请求中为每个块返回一个 `tool_result`，并在每个结果上回传 `toolset_name`。`navigate` 结果在一个 `browser_state` 块中报告它所加载的标签页；`read_page` 结果是文本，其中每个元素都带有一个引用：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01NRLabsLyVHZPKxbKvkfSMn",
      "toolset_name": "browser",
      "content": [
        { "type": "text", "text": "Navigated to https://example.com/docs" },
        {
          "type": "browser_state",
          "tabs": [
            {
              "tab_id": "tab-1",
              "title": "Documentation",
              "url": "https://example.com/docs",
              "active": true
            }
          ]
        }
      ]
    },
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01UvHU5cDyTZ2vXKf5wCkPqR",
      "toolset_name": "browser",
      "content": [
        {
          "type": "text",
          "text": "link \"Documentation\" [ref_1]\nlink \"Getting started\" [ref_2]\ntextbox \"Search docs\" [ref_3]\nbutton \"Search\" [ref_4]\nlink \"Pricing\" [ref_5]"
        }
      ]
    }
  ]
}
```

Claude 现在持有可以据以执行操作的引用，因此它的下一轮可以点击 `ref_2` 来打开入门页面，而无需先在屏幕截图中定位该链接。

## 浏览器使用的工作原理

浏览器使用工具在您的应用程序中以 “agent loop”（智能体循环）的形式运行：Claude 返回成员工具调用，您的执行器在浏览器上运行这些调用，然后您返回结果，直到 Claude 以文本作答。

<Steps>
  <Step title="为 Claude 提供浏览器使用工具和用户提示">
    * 将 `browser_toolset_20260801` 条目（以及可选的其他工具）添加到您的 API 请求中。
    * 包含一个需要处理网页的用户提示，例如"打开 example.com/docs 并告诉我如何入门。"
  </Step>

  <Step title="Claude 以成员工具调用进行响应">
    * Claude 在单个助手轮次中返回一个或多个 `tool_use` 块；一个轮次中的多个块构成一个 “batch action”（批量操作），例如先 `left_click`，再 `type`，然后 `key`。
    * 每个块的 `name` 是成员名称，每个块都携带 `"toolset_name": "browser"`，`input` 仅包含该成员的参数，没有 `action` 字段。响应的 `stop_reason` 为 `tool_use`。
  </Step>

  <Step title="按顺序运行调用并返回结果">
    * 遍历 `response.content` 中的每个 `tool_use` 块（不要假设只有一个），并按它们出现的顺序依次运行，因为后面的调用通常依赖于前面的调用。
    * 在一条新的 `user` 消息中为每个块返回一个 `tool_result`，通过 `tool_use_id` 进行匹配，并在每个结果上回传 `"toolset_name": "browser"`。请回应每一个调用，否则下一个请求将被拒绝。
    * 如果某个调用失败，为该块返回 `is_error: true` 及文本描述，然后对该轮次中所有后续块应用[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)中的中止规则。
  </Step>

  <Step title="Claude 持续执行直到任务完成">
    * Claude 读取结果（页面文本、无障碍树、屏幕截图、标签页状态），如果需要更多信息，会返回更多成员调用，这会让您回到第 3 步。
    * 否则，它会向用户返回文本响应。
  </Step>
</Steps>

下面是该循环中工具调用步骤的骨架，分为两部分。首先，存根成员处理程序代替您的浏览器自动化。五个成员（`navigate`、`read_page`、`left_click`、`type` 和 `screenshot`）返回将成为结果内容的文本（对于 `screenshot` 则是图像块），而调度器会对它未实现的任何成员抛出错误。

<CodeGroup exclude="shell">
  ```python Python
  # 占位图像数据；真实的执行器会捕获视口并返回 PNG 字节
  PLACEHOLDER_PNG = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="


  def navigate(url):
      return f"navigated to {url}"


  def read_page():
      return 'link "Docs" [ref_1]\nbutton "Search" [ref_2]'


  def click(target):
      # 目标是来自 read_page 或 find 的元素引用，或视口坐标
      if target["type"] == "ref":
          return f"clicked {target['ref']}"
      return f"clicked at ({target['x']}, {target['y']})"


  def type_text(text):
      return f"typed: {text}"


  def capture_screenshot() -> list[ImageBlockParam]:
      # screenshot 以图像块而非文本作答：返回结果内容列表
      return [
          {
              "type": "image",
              "source": {
                  "type": "base64",
                  "media_type": "image/png",
                  "data": PLACEHOLDER_PNG,
              },
          }
      ]


  def handle_browser_action(name, tool_input):
      if name == "navigate":
          return navigate(tool_input["url"])
      elif name == "read_page":
          return read_page()
      elif name == "left_click":
          return click(tool_input["target"])
      elif name == "type":
          return type_text(tool_input["text"])
      elif name == "screenshot":
          return capture_screenshot()
      # 按需处理其他操作
      raise ValueError(f"Unknown or unimplemented member: {name}")
  ```

  ```typescript TypeScript
  // 占位图像数据；真实执行器会将视口捕获为 PNG 字节
  const PLACEHOLDER_PNG =
    "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==";

  function navigate(url: string): string {
    return `navigated to ${url}`;
  }

  function readPage(): string {
    return 'link "Docs" [ref_1]\nbutton "Search" [ref_2]';
  }

  function clickElement(ref: string): string {
    return `clicked ${ref}`;
  }

  function clickAt(x: number, y: number): string {
    return `clicked at (${x}, ${y})`;
  }

  function typeText(text: string): string {
    return `typed: ${text}`;
  }

  function captureScreenshot(): Anthropic.ImageBlockParam[] {
    // screenshot 以图像块而非文本作为响应
    return [
      {
        type: "image",
        source: {
          type: "base64",
          media_type: "image/png",
          data: PLACEHOLDER_PNG
        }
      }
    ];
  }

  function handleBrowserAction(
    action: string,
    input: unknown
  ): string | Anthropic.ImageBlockParam[] {
    const params: object = typeof input === "object" && input !== null ? input : {};
    if (action === "navigate" && "url" in params) {
      return navigate(String(params.url));
    } else if (action === "read_page") {
      return readPage();
    } else if (action === "left_click" && "target" in params) {
      // target 是来自 read_page 的元素引用或视口坐标
      const target: object =
        typeof params.target === "object" && params.target !== null ? params.target : {};
      if ("type" in target && target.type === "ref" && "ref" in target) {
        return clickElement(String(target.ref));
      } else if ("x" in target && "y" in target) {
        return clickAt(Number(target.x), Number(target.y));
      }
    } else if (action === "type" && "text" in params) {
      return typeText(String(params.text));
    } else if (action === "screenshot") {
      return captureScreenshot();
    }
    // 按需处理其他操作
    throw new Error(`Unknown or unimplemented member: ${action}`);
  }
  ```

  ```csharp C#
  // 占位图像数据；真实的执行器会捕获视口并返回 PNG 字节
  const string PlaceholderPng = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==";

  string Navigate(string url) => $"navigated to {url}";

  string ReadPage() =>
      """
      link "Docs" [ref_1]
      button "Search" [ref_2]
      """;

  string ClickRef(string elementRef) => $"clicked {elementRef}";

  string ClickAt(int x, int y) => $"clicked at ({x}, {y})";

  // target 为 {"type": "ref", "ref": "ref_1"} 或 {"type": "coordinate", "x": 640, "y": 380}
  string Click(JsonElement target) =>
      target.GetProperty("type").GetString() == "ref"
          ? ClickRef(target.GetProperty("ref").GetString()!)
          : ClickAt(target.GetProperty("x").GetInt32(), target.GetProperty("y").GetInt32());

  string TypeText(string text) => $"typed: {text}";

  // screenshot 以图像块而非文本作答：返回结果内容列表
  List<Block> CaptureScreenshot() =>
      [
          new ImageBlockParam(
              new Base64ImageSource { Data = PlaceholderPng, MediaType = MediaType.ImagePng }
          ),
      ];

  ToolResultBlockParamContent HandleBrowserAction(
      string action,
      IReadOnlyDictionary<string, JsonElement> input
  ) =>
      action switch
      {
          "navigate" => Navigate(input["url"].GetString()!),
          "read_page" => ReadPage(),
          "left_click" => Click(input["target"]),
          "type" => TypeText(input["text"].GetString()!),
          "screenshot" => CaptureScreenshot(),
          // 按需处理其他操作
          _ => throw new NotSupportedException($"Unknown or unimplemented member: {action}"),
      };
  ```

  ```go Go
  // placeholderPNG 用于替代真实截图：执行器会将
  // 视口以 base64 编码的 PNG 数据形式返回。
  const placeholderPNG = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="

  // textContent 将文本包装为 tool_result 内容。
  func textContent(text string) []anthropic.ToolResultBlockParamContentUnion {
  	return []anthropic.ToolResultBlockParamContentUnion{
  		{OfText: &anthropic.TextBlockParam{Text: text}},
  	}
  }

  func navigate(url string) string {
  	return fmt.Sprintf("navigated to %s", url)
  }

  func readPage() string {
  	return "link \"Docs\" [ref_1]\nbutton \"Search\" [ref_2]"
  }

  func clickRef(ref string) string {
  	return fmt.Sprintf("clicked %s", ref)
  }

  func clickAt(x, y int) string {
  	return fmt.Sprintf("clicked at (%d, %d)", x, y)
  }

  func typeText(text string) string {
  	return fmt.Sprintf("typed: %s", text)
  }

  // captureScreenshot 返回图像块而非文本。
  func captureScreenshot() []anthropic.ToolResultBlockParamContentUnion {
  	return []anthropic.ToolResultBlockParamContentUnion{{
  		OfImage: &anthropic.ImageBlockParam{
  			Source: anthropic.ImageBlockParamSourceUnion{
  				OfBase64: &anthropic.Base64ImageSourceParam{
  					MediaType: anthropic.Base64ImageSourceMediaTypeImagePNG,
  					Data:      placeholderPNG,
  				},
  			},
  		},
  	}}
  }

  func handleBrowserAction(action string, params map[string]any) ([]anthropic.ToolResultBlockParamContentUnion, error) {
  	switch action {
  	case "navigate":
  		if url, ok := params["url"].(string); ok {
  			return textContent(navigate(url)), nil
  		}
  	case "read_page":
  		return textContent(readPage()), nil
  	case "left_click":
  		// target 可以是来自 read_page 的元素引用，也可以是视口坐标
  		target, _ := params["target"].(map[string]any)
  		if ref, ok := target["ref"].(string); ok && target["type"] == "ref" {
  			return textContent(clickRef(ref)), nil
  		}
  		x, xok := target["x"].(float64)
  		y, yok := target["y"].(float64)
  		if xok && yok {
  			return textContent(clickAt(int(x), int(y))), nil
  		}
  	case "type":
  		if text, ok := params["text"].(string); ok {
  			return textContent(typeText(text)), nil
  		}
  	case "screenshot":
  		return captureScreenshot(), nil
  	// 按需处理其他操作
  	default:
  		return nil, fmt.Errorf("unknown or unimplemented member: %s", action)
  	}
  	// 当成员的输入缺少字段或字段类型错误时会到达此处
  	return nil, fmt.Errorf("invalid input for %s", action)
  }
  ```

  ```java Java
  /** Placeholder pixels; a real executor captures the viewport and base64-encodes the PNG. */
  static final String PLACEHOLDER_PNG = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==";

  ToolResultBlockParam.Content captureScreenshot() {
      ImageBlockParam image = ImageBlockParam.builder()
          .source(Base64ImageSource.builder()
              .mediaType(Base64ImageSource.MediaType.IMAGE_PNG)
              .data(PLACEHOLDER_PNG)
              .build())
          .build();
      return ToolResultBlockParam.Content.ofBlocks(
          List.of(ToolResultBlockParam.Content.Block.ofImage(image)));
  }

  String navigate(String url) {
      return "navigated to " + url;
  }

  String readPage() {
      return """
          link "Docs" [ref_1]
          button "Search" [ref_2]""";
  }

  String clickRef(String ref) {
      return "clicked " + ref;
  }

  String clickAt(long x, long y) {
      return "clicked at (" + x + ", " + y + ")";
  }

  String typeText(String text) {
      return "typed: " + text;
  }

  /** Runs one browser toolset member; {@code action} is the tool_use block's name. */
  ToolResultBlockParam.Content handleBrowserAction(String action, Map<String, JsonValue> input) {
      if (action.equals("screenshot")) {
          return captureScreenshot(); // the one member here that answers with an image block
      }
      String output = switch (action) {
          case "navigate" -> navigate(input.get("url").asStringOrThrow());
          case "read_page" -> readPage();
          case "left_click" -> {
              // target 为 {"type": "ref", "ref": "ref_1"} 或 {"type": "coordinate", "x": 640, "y": 380}
              Map<String, JsonValue> target =
                  (Map<String, JsonValue>) input.get("target").asObject().get();
              if (target.get("type").asStringOrThrow().equals("ref")) {
                  yield clickRef(target.get("ref").asStringOrThrow());
              }
              long x = ((Number) target.get("x").asNumber().get()).longValue();
              long y = ((Number) target.get("y").asNumber().get()).longValue();
              yield clickAt(x, y);
          }
          case "type" -> typeText(input.get("text").asStringOrThrow());
          // 根据需要处理其他操作
          default -> throw new UnsupportedOperationException("Unknown or unimplemented member: " + action);
      };
      return ToolResultBlockParam.Content.ofString(output);
  }
  ```

  ```php PHP
  // 真实 PNG 字节的占位符；真实执行器会捕获视口
  const PLACEHOLDER_PNG = 'iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==';

  function navigateTo(string $url): string
  {
      return "navigated to {$url}";
  }

  function readPage(): string
  {
      return <<<'TEXT'
          link "Docs" [ref_1]
          button "Search" [ref_2]
          TEXT;
  }

  function clickTarget(array $target): string
  {
      // 目标是来自 read_page 或 find 的元素引用，或视口像素坐标
      if ($target['type'] === 'ref') {
          return "clicked {$target['ref']}";
      }

      return "clicked at ({$target['x']}, {$target['y']})";
  }

  function typeText(string $text): string
  {
      return "typed: {$text}";
  }

  function captureScreenshot(): array
  {
      // screenshot 以图像块而非文本作答，因此返回结果内容列表
      $image = [
          'type' => 'image',
          'source' => ['type' => 'base64', 'media_type' => 'image/png', 'data' => PLACEHOLDER_PNG],
      ];

      return [$image];
  }

  function handleBrowserAction(string $name, array $input): string|array
  {
      return match ($name) {
          'navigate' => navigateTo($input['url']),
          'read_page' => readPage(),
          'left_click' => clickTarget($input['target']),
          'type' => typeText($input['text']),
          'screenshot' => captureScreenshot(),
          // 按需处理其他操作
          default => throw new \RuntimeException("Unknown or unimplemented member: {$name}"),
      };
  }
  ```

  ```ruby Ruby
  # 占位图像数据；真实的执行器会将视口截取为 PNG。
  PLACEHOLDER_PNG = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="

  def navigate(url)
    "navigated to #{url}"
  end

  def read_page
    <<~TREE
      link "Docs" [ref_1]
      button "Search" [ref_2]
    TREE
  end

  def click(target)
    return "clicked #{target[:ref]}" if target[:type] == "ref"

    "clicked at (#{target[:x]}, #{target[:y]})"
  end

  def type_text(text)
    "typed: #{text}"
  end

  def capture_screenshot
    [
      {
        type: "image",
        source: { type: "base64", media_type: "image/png", data: PLACEHOLDER_PNG }
      }
    ]
  end

  def handle_browser_action(name, input)
    case name
    when "navigate"
      navigate(input[:url])
    when "read_page"
      read_page
    when "left_click"
      # target 是元素引用（来自 read_page 或 find）或坐标
      click(input[:target])
    when "type"
      type_text(input[:text])
    when "screenshot"
      capture_screenshot
    # 按需处理其他操作
    else
      raise ArgumentError, "Unknown or unimplemented member: #{name}"
    end
  end
  ```
</CodeGroup>

第二部分按顺序运行一个批次，将每个块分派给这些处理程序，在每个结果上回显 `toolset_name`，并应用[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)中的中止规则，将处理程序错误转换为错误结果。调用它的采样循环就是[理解智能体循环](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#understanding-the-agentic-loop)中展示的那个循环，只是 `tools` 中放的是浏览器工具集。

<CodeGroup exclude="shell">
  ```python Python
  NOT_EXECUTED = "Not executed: an earlier action in this turn failed."


  def process_tool_calls(response: Message) -> list[ToolResultBlockParam]:
      """
      Run the browser actions in Claude's response in order and answer each
      one. After the first failure the rest are skipped, because Claude planned
      them assuming the earlier actions succeeded.
      """
      tool_results: list[ToolResultBlockParam] = []
      failed = False
      for block in response.content:
          # 仅声明了浏览器工具集；如添加其他工具，请在此处路由
          if block.type != "tool_use" or block.toolset_name != "browser":
              continue
          result: ToolResultBlockParam = {
              "type": "tool_result",
              "tool_use_id": block.id,
              "toolset_name": "browser",
          }
          if failed:
              result["content"] = NOT_EXECUTED
              result["is_error"] = True
          else:
              try:
                  # 字符串或内容块列表；真实的执行器还会向
                  # 导航和标签页管理结果中添加 browser_state 块
                  result["content"] = handle_browser_action(block.name, block.input)
              except Exception as err:
                  result["content"] = f"Error: {err}"
                  result["is_error"] = True
                  failed = True
          tool_results.append(result)
      return tool_results
  ```

  ```typescript TypeScript
  const HALT_TEXT = "Not executed: an earlier action in this turn failed.";

  function browserResult(
    toolUseId: string,
    content: string | Anthropic.ImageBlockParam[],
    isError?: boolean
  ): Anthropic.ToolResultBlockParam {
    return {
      type: "tool_result",
      tool_use_id: toolUseId,
      toolset_name: "browser",
      content,
      is_error: isError
    };
  }

  function processToolCalls(response: Anthropic.Message): Anthropic.ToolResultBlockParam[] {
    const toolResults: Anthropic.ToolResultBlockParam[] = [];
    let failed = false;
    for (const block of response.content) {
      if (block.type !== "tool_use") {
        continue;
      }
      if (block.toolset_name !== "browser") {
        // 此示例仅声明浏览器工具集；如果您添加了其他工具，
        // 请在此处进行路由。
        continue;
      }
      if (failed) {
        // 批处理在首次失败时停止；后续操作直接返回结果而不执行
        toolResults.push(browserResult(block.id, HALT_TEXT, true));
        continue;
      }
      try {
        // 字符串或图像块列表；真实执行器还会向导航和
        // 标签页管理结果中添加 browser_state 块
        const result = handleBrowserAction(block.name, block.input);
        toolResults.push(browserResult(block.id, result));
      } catch (error) {
        failed = true;
        const message = error instanceof Error ? error.message : String(error);
        toolResults.push(browserResult(block.id, `Error: ${message}`, true));
      }
    }
    return toolResults;
  }
  ```

  ```csharp C#
  const string HaltText = "Not executed: an earlier action in this turn failed.";

  List<ContentBlockParam> ProcessToolCalls(Message response)
  {
      List<ContentBlockParam> toolResults = [];
      var failed = false;
      foreach (var block in response.Content)
      {
          if (!block.TryPickToolUse(out var toolUse))
          {
              continue;
          }

          if (toolUse.ToolsetName != "browser")
          {
              // 本示例仅声明了浏览器工具集；如果您添加了其他工具，
              // 请在此处进行路由。
              continue;
          }

          if (failed)
          {
              // 批处理在首次失败时停止；对后续操作直接作答而不执行
              toolResults.Add(
                  new ToolResultBlockParam(toolUse.ID)
                  {
                      Content = HaltText,
                      IsError = true,
                      ToolsetName = "browser",
                  }
              );
              continue;
          }

          try
          {
              // 字符串或内容块列表；真实的执行器还会向导航和
              // 标签页管理的结果中添加 browser_state 块
              var result = HandleBrowserAction(toolUse.Name, toolUse.Input);
              toolResults.Add(
                  new ToolResultBlockParam(toolUse.ID) { Content = result, ToolsetName = "browser" }
              );
          }
          catch (Exception e)
          {
              failed = true;
              toolResults.Add(
                  new ToolResultBlockParam(toolUse.ID)
                  {
                      Content = $"Error: {e.Message}",
                      IsError = true,
                      ToolsetName = "browser",
                  }
              );
          }
      }
      return toolResults;
  }
  ```

  ```go Go
  const notExecuted = "Not executed: an earlier action in this turn failed."

  // browserToolResult 为单个浏览器操作构建结果。与
  // 普通工具结果不同，它必须回显工具集名称。真实的执行器还会
  // 在导航和标签页管理结果中添加 browser_state 块。
  func browserToolResult(toolUseID string, content []anthropic.ToolResultBlockParamContentUnion, isError bool) anthropic.ContentBlockParamUnion {
  	result := anthropic.ToolResultBlockParam{
  		ToolUseID:   toolUseID,
  		ToolsetName: anthropic.String("browser"),
  		Content:     content,
  	}
  	if isError {
  		result.IsError = anthropic.Bool(true)
  	}
  	return anthropic.ContentBlockParamUnion{OfToolResult: &result}
  }

  // processToolCalls 按顺序运行 Claude 响应中的浏览器操作，并
  // 为每个 tool_use 块构建一个 tool_result。首次失败后会跳过
  // 其余操作：Claude 是在假设先前操作成功的前提下规划它们的。
  func processToolCalls(response *anthropic.Message) []anthropic.ContentBlockParamUnion {
  	var toolResults []anthropic.ContentBlockParamUnion
  	failed := false
  	for _, block := range response.Content {
  		switch variant := block.AsAny().(type) {
  		case anthropic.ToolUseBlock:
  			// 本示例仅声明了浏览器工具集；如添加其他工具，请在此处路由。
  			if variant.ToolsetName != "browser" {
  				continue
  			}
  			if failed {
  				toolResults = append(toolResults, browserToolResult(variant.ID, textContent(notExecuted), true))
  				continue
  			}
  			var input map[string]any
  			var content []anthropic.ToolResultBlockParamContentUnion
  			err := json.Unmarshal(variant.Input, &input)
  			if err == nil {
  				content, err = handleBrowserAction(variant.Name, input)
  			}
  			if err != nil {
  				failed = true
  				content = textContent("Error: " + err.Error())
  			}
  			toolResults = append(toolResults, browserToolResult(variant.ID, content, err != nil))
  		}
  	}
  	return toolResults
  }
  ```

  ```java Java
  /** The exact text the toolset contract prescribes for member calls skipped after a failure. */
  static final String HALT_TEXT = "Not executed: an earlier action in this turn failed.";

  /** Every result answering a browser toolset member echoes toolset_name. */
  ToolResultBlockParam.Builder browserResult(ToolUseBlock toolUse) {
      return ToolResultBlockParam.builder()
          .toolUseId(toolUse.id())
          .toolsetName("browser");
  }

  /**
   * Run the browser actions in Claude's response in order and build one
   * tool_result for each tool_use block. After the first failure, skip the rest:
   * Claude planned them assuming the earlier actions succeeded.
   */
  List<ContentBlockParam> processToolCalls(Message response) {
      List<ContentBlockParam> toolResults = new ArrayList<>();
      boolean failed = false;
      for (ContentBlock block : response.content()) {
          // 此示例仅声明了浏览器工具集；如果您添加了其他工具，请在此处进行路由。
          if (!block.isToolUse() || !block.asToolUse().toolsetName().equals(Optional.of("browser"))) {
              continue;
          }
          ToolUseBlock toolUse = block.asToolUse();
          ToolResultBlockParam result;
          if (failed) {
              result = browserResult(toolUse)
                  .content(HALT_TEXT)
                  .isError(true)
                  .build();
          } else {
              try {
                  Map<String, JsonValue> input =
                      (Map<String, JsonValue>) toolUse._input().asObject().get();
                  ToolResultBlockParam.Content output = handleBrowserAction(toolUse.name(), input);
                  // 真实的执行器还会向导航和标签页管理结果中添加 browser_state 块；
                  // 请参阅本页的“使用 browser_state 跟踪标签页”。
                  result = browserResult(toolUse)
                      .content(output)
                      .build();
              } catch (RuntimeException e) {
                  failed = true;
                  result = browserResult(toolUse)
                      .content("Error: " + e.getMessage())
                      .isError(true)
                      .build();
              }
          }
          toolResults.add(ContentBlockParam.ofToolResult(result));
      }
      return toolResults;
  }
  ```

  ```php PHP
  const HALT_TEXT = 'Not executed: an earlier action in this turn failed.';

  function processToolCalls(Message $response): array
  {
      $toolResults = [];
      $failed = false;
      foreach ($response->content as $block) {
          // 此示例仅声明浏览器工具集；如添加其他工具，请在此处路由。
          if (!($block instanceof \Anthropic\Messages\ToolUseBlock) || $block->toolsetName !== 'browser') {
              continue;
          }
          $result = ['type' => 'tool_result', 'tool_use_id' => $block->id, 'toolset_name' => 'browser'];
          if ($failed) {
              // 批处理在首次失败时停止；其余操作不运行直接作答
              $toolResults[] = [...$result, 'content' => HALT_TEXT, 'is_error' => true];
              continue;
          }
          try {
              // 真实执行器在导航和标签页管理结果中还会返回 browser_state 块
              $toolResults[] = [...$result, 'content' => handleBrowserAction($block->name, $block->input)];
          } catch (\Throwable $e) {
              $failed = true;
              $toolResults[] = [...$result, 'content' => 'Error: ' . $e->getMessage(), 'is_error' => true];
          }
      }

      return $toolResults;
  }
  ```

  ```ruby Ruby
  NOT_EXECUTED = "Not executed: an earlier action in this turn failed."

  # 按顺序执行 Claude 响应中的浏览器操作，并为每个
  # tool_use 块构建一个 tool_result。首次失败后跳过其余操作：
  # Claude 在规划它们时假定之前的操作已成功。
  def process_tool_calls(response)
    tool_results = []
    failed = false
    response.content.each do |block|
      # 本示例仅声明了浏览器工具集；如果您添加了其他工具，
      # 请在此处进行路由。
      next unless block.type == :tool_use && block.toolset_name == "browser"

      result = { type: "tool_result", tool_use_id: block.id, toolset_name: "browser" }
      if failed
        result.update(content: NOT_EXECUTED, is_error: true)
      else
        begin
          # 一个 String，或截图的内容块。真实的执行器还会向
          # 导航和标签页管理结果中添加一个 browser_state 块。
          result[:content] = handle_browser_action(block.name, block.input)
        rescue => e
          result.update(content: "Error: #{e.message}", is_error: true)
          failed = true
        end
      end
      tool_results << result
    end
    tool_results
  end
  ```
</CodeGroup>

请根据（`toolset_name`、`name`）这一对值而不是仅根据 `name` 来分派每个块，因为同一请求中的自定义工具可能与某个成员同名；[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)描述了两个工具集共享的这部分约定。如果 Claude 指明了您的执行器未实现的成员，或者您已禁用的成员，请用[错误结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#return-errors-from-your-executor)应答该块，而不是丢弃它。

当您以流式传输方式接收响应时，每个成员的 `input` 会作为一个完整的 `input_json_delta` 到达，而不是分片到达，因此请等待该轮次结束后再运行批次。

### 批量操作

包含多个成员调用的轮次是一个批量操作：按调用出现的顺序运行它们，在第一次失败时停止，并用 `is_error: true` 和确切文本 `Not executed: an earlier action in this turn failed.` 应答之后的每个调用。批次使用与[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)相同的响应形状；区别在于您按顺序而非并发地运行这些块。在这里，Claude 在一个轮次中点击它之前找到的搜索框、输入查询并按下 Enter：

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01D7FLrfh4GYq7yT1ULFeyMV",
      "name": "left_click",
      "toolset_name": "browser",
      "input": { "target": { "type": "ref", "ref": "ref_3" } }
    },
    {
      "type": "tool_use",
      "id": "toolu_01Ez4kLb1nQ2vXo8sJ9pWm3c",
      "name": "type",
      "toolset_name": "browser",
      "input": { "text": "install" }
    },
    {
      "type": "tool_use",
      "id": "toolu_01FkP8rTz6uYh2mNq4LsXw7v",
      "name": "key",
      "toolset_name": "browser",
      "input": { "text": "Enter" }
    }
  ]
}
```

您的应用程序在一条 `user` 消息中返回三个 `tool_result` 块，每个块都携带 `toolset_name` 和一段简短的文本确认，例如 `Clicked element ref_3.`。按下 Enter 会加载结果页面，因此 `key` 的结果还携带一个包含该标签页更新后 URL 的 `browser_state` 块（[其他结果上的标签页上下文](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#tab-context-on-other-results)）。如果点击失败了，它的结果将携带您的错误文本，而另外两个结果将携带中止文本，如[从执行器返回错误](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#return-errors-from-your-executor)中所示。

您不需要在每次调用后都返回屏幕截图。Claude 通常以一个观察调用（`screenshot`、`read_page` 或 `get_page_text`）结束一个批次，您的应用程序也可以将自己的观察结果（例如一张新的屏幕截图或无障碍树）作为额外的内容块附加到批次中最后一个结果上，以节省一次往返。由于标签页管理结果必须恰好是一个 `browser_state` 块，请将其附加到最后一个不是标签页管理调用的结果上。

如果您的执行器每次往返只能运行一个调用，请在 `tool_choice` 中将 `disable_parallel_tool_use` 设置为 `true`，这样 Claude 每轮最多返回一个成员调用，代价是往返次数更多（[禁用并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use#disable-parallel-tool-use)）。[计算机使用工具的批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)中的其余约定同样适用，包括在下一条 `user` 消息中为每个 `tool_use` 返回一个 `tool_result`，但有两点除外：中止文本，以及成功结果的 `content` 所包含的内容。结果内容改为遵循本页的[成员工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#member-tools)：`new_tab`、`switch_tab`、`close_tab` 或 `list_tabs` 的结果恰好是一个 `browser_state` 块，不含文本或图像（[标签页管理结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#tab-management-results)），而任何其他成员的结果可以在其文本或图像之外添加一个 `browser_state` 块（[其他结果上的标签页上下文](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#tab-context-on-other-results)）。批次内的缓存断点在何处生效，请参见计算机使用工具的[工具参数](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#tool-parameters)中的 `cache_control` 行。

### 目标和坐标

对某个位置执行操作的成员工具接受一个 `target` 对象，它要么是视口像素坐标，要么是对 `read_page` 或 `find` 返回的元素的引用。[成员工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#member-tools)表格中用 `Target` 表示接受任一形状的参数。

| 形状                 | `target.type`  | 字段                       | 接受者                                                                                                                                                                          |
| ------------------ | -------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CoordinateTarget` | `"coordinate"` | `x`、`y`（整数，视口像素）         | `left_click`、`right_click`、`middle_click`、`double_click`、`triple_click`、`hover`、`left_click_drag`（`from` 和 `target`）、`left_mouse_down`、`left_mouse_up`、`mouse_move`、`scroll` |
| `RefTarget`        | `"ref"`        | `ref`（元素引用，例如 `"ref_2"`） | `left_click`、`right_click`、`middle_click`、`double_click`、`triple_click`、`hover`、`scroll_to`、`form_input`、`file_upload`                                                       |

**坐标是视口像素**，即完整视口 `screenshot` 的像素空间，原点位于渲染页面的左上角；不存在外围的桌面或窗口框架。该工具集不声明显示尺寸，Claude 会根据您返回的屏幕截图推断视口大小，因此请保持截图尺寸一致。`zoom` 不会改变坐标系，因此其 `region` 以及 Claude 在收到缩放图像后给出的任何坐标仍然是完整视口像素。

**屏幕截图必须符合图像限制。** API 不会缩小工具集图像：超过您模型的[图像尺寸限制](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)，或超过请求包含[超过 20 张图像](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#request-limits)时适用的更严格的单图限制的屏幕截图或缩放图像会被拒绝。请在返回前调整大小，并在分派 Claude 的坐标之前按您缩放因子的倒数将其放大回去（[调整屏幕截图大小以符合图像限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#handle-coordinate-scaling-for-higher-resolutions)）。

**元素引用来自 `read_page` 和 `find`。** 它们输出中的每个元素都带有一个标签，例如 `[ref_2]`，如快速开始的结果所示：

```text wrap
link "Documentation" [ref_1]
link "Getting started" [ref_2]
textbox "Search docs" [ref_3]
button "Search" [ref_4]
link "Pricing" [ref_5]
```

Claude 在之后的点击、`hover`、`scroll_to`、`form_input` 或 `file_upload` 调用中以 `{"type": "ref", "ref": "ref_2"}` 目标的形式传回引用，或者作为 `read_page` 的 `ref` 参数来读取子树。您的执行器负责分配引用，保存每个引用到底层节点（无障碍节点 ID、存储的选择器或等效物）的映射，并在引用传回时对该节点执行操作。

引用的作用域限于生成它们的标签页，并在该标签页导航或其 DOM 发生实质性变化之前保持有效。API 无法检测过期或未知的引用，因此当 Claude 传递一个您的执行器不再识别的引用时，请返回错误结果，例如 `Error: ref_3 is stale or not found on the current page. Re-read the page to get fresh references.`。Claude 随后会重新读取页面。在标签页导航之前，不要对您已经为该标签页分发的引用重新编号，因为那会悄无声息地使 Claude 仍持有的引用失效。

Claude 会使用两种定位方式，并根据页面暴露的内容在它们之间切换；您的提示和执行器返回的内容会引导这一选择：

* **在页面具有可用的无障碍树时，优先使用引用。** 引用能够经受住让像素坐标变得脆弱的布局偏移和重排，并让 Claude 能够操作难以用指针点击的控件。
* **对于树未描述的内容回退到坐标。** Canvas 渲染的界面、嵌入式视频或远程桌面表面、高度虚拟化的列表以及跨源 iframe 内的元素通常没有有用的节点，因此 Claude 依靠 `screenshot` 和 `zoom` 工作并按坐标点击；您的执行器负责解析坐标落在哪个框架中。
* **限定读取范围，并在截图之前先读取树。** 在大型页面上，带 `filter: "interactive"` 或容器 `ref` 的 `read_page` 会返回一个聚焦的子树，而对典型页面的树读取通常比屏幕截图消耗更少的输入令牌，同时为 Claude 提供可以立即据以执行操作的引用。当视觉布局、图像或渲染状态很重要时，屏幕截图仍然是正确的观察方式。

## 安全注意事项

浏览器使用带有标准 API 功能所没有的风险，因为 Claude 读取并操作来自开放网络的内容，而任何页面都可能包含为操纵它而编写的文本。

<Warning>
  为降低这些风险，请采取如下预防措施：

  1. 在专用容器或虚拟机中运行浏览器和执行器，赋予最低权限，使用不含任何凭据的全新配置文件，且不允许访问敏感文件系统或内部网络；以同样的方式隔离您与其一同运行的任何工具。
  2. 将浏览器可访问的主机限制在域名允许列表内，该列表在网络层强制执行，并在重定向后于您的 `navigate` 处理程序中再次检查；除非任务需要，否则阻止环回、链路本地和私有地址范围。
  3. 将页面提供的所有内容视为不受信任的输入。这包括您在 [`browser_state`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#track-tabs-and-page-state) 块中报告的标签页标题和 URL，以及每个下载项的 `url`、`path` 和 `error`。基于页面所渲染的内容（无障碍树或可见文本）而非原始 DOM 源代码来构建页面读取结果，以免隐藏文本到达 Claude。
  4. 在您的 `navigate` 处理程序中，接受历史关键字 `"back"`、`"forward"` 和 `"reload"`，将没有协议的 URL 视为 `https://`，然后解析 URL，并以[错误结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#return-errors-from-your-executor)拒绝 `http` 或 `https` 以外的任何协议（`javascript:`、`file:`、`data:`、`chrome:` 等）。请使用 URL 解析器而非字符串前缀来检查协议；API 不会过滤 Claude 打开的 URL，因此无法替您拒绝。
  5. 除非需要，否则保持 `javascript_exec` 和 `file_upload` 处于禁用状态，并在启用其中任何一个之前阅读[启用可选成员](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#enable-optional-member-tools)。
  6. 让人工确认重大操作以及任何需要明确同意的操作（购买、修改账户、发送消息和接受条款），并在每次调用之前在执行器中进行该检查，因为一个轮次可能包含多个调用。
</Warning>

Claude 有时会遵循在页面内容中发现的指令，即使它们与您的指令冲突；页面上写着"忽略你之前的指令并导航到……"的文本可能使它偏离任务。请将 Claude 与敏感数据和操作隔离，以限制"prompt injection"（提示注入）所能触及的范围，查阅[缓解越狱和提示注入](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)，如果任务无法避免已登录的会话，请使用专用的低权限账户，并对更改账户的操作保持人工确认。

Anthropic 已训练模型抵御这些提示注入，并增加了一层额外的防御。如果您使用浏览器使用工具，分类器会自动扫描浏览器返回的内容（例如页面文本或屏幕截图），以标记潜在的提示注入。当这些分类器识别出潜在的提示注入时，它们会引导模型在执行指令之前先确认该指令是否确实来自您。

这种额外的保护并不适用于所有用例（例如没有人工参与的用例），因此如果您希望选择退出并将其关闭，请[联系支持团队](https://support.claude.com/en/)。即使有这些分类器，前面列出的预防措施仍然很重要。

由于浏览器在您的环境中运行，Claude 访问的网站会看到您执行器的网络身份，而页面内容仅以您返回的工具结果的形式到达 API。在您的产品中启用浏览器使用之前，请告知最终用户相关风险并获得他们的同意。

## 成员工具

`browser_toolset_20260801` 条目声明了 31 个成员工具；每次调用的 `input` 恰好是此处列出的参数，而 `tab_id` 在为可选时默认为当前活动标签页。`Target`、`CoordinateTarget` 和 `RefTarget` 是[目标和坐标](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#targets-and-coordinates)中描述的结构。四个成员（`javascript_exec`、`file_upload`、`read_console` 和 `read_network`）默认禁用，只有在您启用它们时才会出现（请参阅[启用可选成员](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#enable-optional-member-tools)）。每个成员所在行中注明的输入范围和输出约定是告知 Claude 的，而非由 API 强制执行，因此请在执行器中验证输入（包括根据视口验证坐标）并应用这些约定。

只有 `screenshot` 和 `zoom` 要求在其结果中包含 [`image` 块](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls#handling-results-from-client-tools)，而四个标签页管理成员（`new_tab`、`list_tabs`、`switch_tab` 和 `close_tab`）恰好返回一个 `browser_state` 块（请参阅[标签页管理结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#tab-management-results)）。其他所有成员都返回一个 `text` 块：要么是诸如 `Clicked element ref_2.` 这样的简短确认，要么是该成员的输出。除标签页管理结果之外的任何结果还可以携带一个 `image` 块，通常是操作后拍摄的屏幕截图，这样 Claude 无需单独调用 `screenshot` 即可获得操作结果；[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)说明了在批次中应将其附加在何处。成员 `tool_result` 只能包含 `text`、`image` 和 `browser_state` 内容块。

### 导航和捕获

| 成员           | 输入                 | 描述                                                                                                                                                                |
| ------------ | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `navigate`   | `url`、`tab_id?`    | 加载 `http` 或 `https` URL，或使用 `"back"`、`"forward"` 或 `"reload"` 在历史记录中移动。将没有协议的 URL 视为 `https://`，并以错误结果拒绝任何其他协议。返回简短的确认，当标签页的 URL 或标题发生变化时再加上一个 `browser_state` 块。 |
| `screenshot` | `tab_id?`          | 捕获视口并返回一个 `image` 块。                                                                                                                                              |
| `zoom`       | `region`、`tab_id?` | 返回 `region`（以视口像素表示为 `[x0, y0, x1, y1]`）的裁剪并放大的 `image`，用于更仔细地检查小文本或控件。                                                                                           |

### 指针

| 成员                | 输入                                                                       | 描述                                                                                                      |
| ----------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| `left_click`      | `target: Target`、`modifiers?`、`tab_id?`                                  | 左键点击坐标或被引用的元素。`modifiers` 是点击期间按住的组合键，例如 `"shift"` 或 `"ctrl+shift"`。                                    |
| `right_click`     | `target: Target`、`modifiers?`、`tab_id?`                                  | 右键点击坐标或元素。                                                                                              |
| `middle_click`    | `target: Target`、`modifiers?`、`tab_id?`                                  | 中键点击坐标或元素。                                                                                              |
| `double_click`    | `target: Target`、`modifiers?`、`tab_id?`                                  | 左键双击坐标或元素。                                                                                              |
| `triple_click`    | `target: Target`、`modifiers?`、`tab_id?`                                  | 左键三击坐标或元素，通常会选中一行或一个段落。                                                                                 |
| `hover`           | `target: Target`、`tab_id?`                                               | 将指针移到坐标或元素上方而不点击。                                                                                       |
| `left_click_drag` | `from: CoordinateTarget`、`target: CoordinateTarget`、`tab_id?`            | 在 `from` 处按下，拖动到 `target`，然后释放。                                                                         |
| `left_mouse_down` | `target: CoordinateTarget`、`tab_id?`                                     | 在坐标处按下并按住左键；与 `left_mouse_up` 配对以实现自定义拖动。                                                               |
| `left_mouse_up`   | `target: CoordinateTarget`、`tab_id?`                                     | 在坐标处释放左键。                                                                                               |
| `mouse_move`      | `target: CoordinateTarget`、`tab_id?`                                     | 将指针移动到坐标。                                                                                               |
| `scroll`          | `target: CoordinateTarget`、`scroll_direction`、`scroll_amount?`、`tab_id?` | 在视口位置滚动。`scroll_direction` 为 `"up"`、`"down"`、`"left"` 或 `"right"`；`scroll_amount` 以滚轮刻度为单位，1 到 10，默认 3。 |
| `scroll_to`       | `target: RefTarget`、`tab_id?`                                            | 将被引用的元素滚动到视图中。                                                                                          |

### 键盘和计时

| 成员         | 输入                          | 描述                                                                                                                   |
| ---------- | --------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `type`     | `text`、`tab_id?`            | 在当前焦点处输入字面字符串。                                                                                                       |
| `key`      | `text`、`repeat?`、`tab_id?`  | 按下一个键或组合键。`text` 是单个键（`"Enter"`）、用 `+` 连接的组合键（`"ctrl+a"`）或以空格分隔的序列（`"Backspace Backspace"`）；`repeat` 为 1 到 100，默认 1。 |
| `hold_key` | `text`、`duration`、`tab_id?` | 按住一个键或组合键 `duration` 秒，0 到 30。                                                                                       |
| `wait`     | `duration`、`tab_id?`        | 暂停 `duration` 秒，0 到 30。                                                                                              |

### 页面读取

| 成员              | 输入                                  | 描述                                                                                                                                                                                                                                  |
| --------------- | ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `read_page`     | `filter?`、`depth?`、`ref?`、`tab_id?` | 以文本形式返回页面的无障碍树，每个元素都带有一个引用标签，例如 `[ref_2]`。省略 `filter` 时返回每个可见元素；为 `"interactive"` 时仅返回可见的交互元素；为 `"all"` 时还包括视口之外的元素。`depth` 限制树的深度（最小 1，默认 15），`ref` 将读取范围限定为该元素的子树。将输出限制在 50,000 个字符以内并在文本中说明；Claude 随后会用更小的 `depth` 或 `ref` 缩小范围。 |
| `find`          | `query`、`tab_id?`                   | 查找与自然语言描述（例如 `"search field"` 或 `"add to cart button"`）匹配的元素，并以与 `read_page` 相同的带标签格式返回最多 20 个匹配项。                                                                                                                                  |
| `get_page_text` | `tab_id?`                           | 以纯文本形式返回页面的可见文本，优先返回主要文章内容；适用于文章、文档和其他以文本为主的页面。                                                                                                                                                                                     |

### 表单和文件

| 成员                  | 输入                                                     | 描述                                                                                                                                                                              |
| ------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `form_input`        | `target: RefTarget`、`value`、`tab_id?`                  | 直接设置表单元素的值。`value` 是 `string`、`number` 或 `boolean`；对复选框使用 `boolean`，对下拉选择框使用选项的值或可见文本。                                                                                          |
| `file_upload`（默认禁用） | `target: RefTarget`、`paths?`、`document_ids?`、`tab_id?` | 从执行器文件系统上的 `paths`、您的应用程序已暂存的 `document_ids` 或两者设置文件输入元素上的文件；至少需要其中之一。参见[上传文件](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#upload-files)。 |

### 诊断和脚本

| 成员                      | 输入               | 描述                                                                                                                                                                             |
| ----------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `read_console`（默认禁用）    | `tab_id?`        | 返回自上次读取以来累积的标签页控制台条目（日志、警告和错误行），每个条目一行。参见[读取控制台和网络活动](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#read-console-and-network-activity)。    |
| `read_network`（默认禁用）    | `tab_id?`        | 返回自上次读取以来标签页的网络请求（方法、URL、状态、MIME 类型、计时），每个条目一行。                                                                                                                                |
| `javascript_exec`（默认禁用） | `text`、`tab_id?` | 在页面上下文中将 `text` 作为 JavaScript 运行，并以文本形式返回最后一个表达式的值。参见[启用可选成员](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#enable-optional-member-tools)。 |

### 标签页管理

| 成员           | 输入           | 描述                  |
| ------------ | ------------ | ------------------- |
| `new_tab`    | （无）          | 打开一个标签页并使其成为活动标签页。  |
| `list_tabs`  | （无）          | 报告标签页清单。            |
| `switch_tab` | `tab_id`（必需） | 使 `tab_id` 成为活动标签页。 |
| `close_tab`  | `tab_id`（必需） | 关闭 `tab_id`。        |

成功时，这些成员中的每一个都恰好返回一个 `browser_state` 块，不含文本或图像；参见[标签页管理结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#tab-management-results)。

## 配置工具集

除 `type` 之外，工具集条目还接受 `configs`、`cache_control` 和 `allowed_callers`；这些字段与计算机使用工具集共享的规则列在[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)下，本节介绍浏览器特有的默认值。`configs` 是一个以成员名称为键的对象，每个成员的值接受两个字段：

| 字段              | 默认值                                                                                                                                       | 含义                                                                                                                                                                                           |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enabled`       | `true`，但四个[可选成员](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#enable-optional-member-tools)为 `false` | 是否向 Claude 提供该成员。                                                                                                                                                                            |
| `defer_loading` | `false`                                                                                                                                   | 工具集的定义是否为工具搜索而延迟加载。必须在每个已启用的成员上解析为相同的值。在四个可选成员保持禁用的情况下，延迟加载工具集意味着在其他 27 个成员上设置它；参见[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)。 |

### 启用或禁用成员工具

在 `configs` 中只列出您想要更改的成员；您省略的每个成员都保持其默认值。例如，一个实现了控制台读取但未实现低级指针或按键保持控制的执行器会启用 `read_console` 并禁用三个成员：

```json
{
  "type": "browser_toolset_20260801",
  "configs": {
    "read_console": { "enabled": true },
    "left_mouse_down": { "enabled": false },
    "left_mouse_up": { "enabled": false },
    "hold_key": { "enabled": false }
  }
}
```

被禁用的成员会从 Claude 收到的定义中消失；但这并不能保证 Claude 永远不会指明它，因此您的执行器仍需用[错误结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#return-errors-from-your-executor)回应此类调用。

### 与其他工具组合

在同一个 `tools` 数组中将浏览器使用工具与您自己的工具以及其他 Anthropic 提供的工具一起声明。自定义工具可以与某个成员同名（例如您自己的 `navigate`），因为 `toolset_name` 可以区分 Claude 的调用，但其他任何条目都不能命名为 `browser`，并且一个请求只能包含一个浏览器工具集条目。

您也可以将它与[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)一起声明，无论是工具集还是更早的计算机使用工具版本。两者独立工作，各自使用自己的坐标系（这里是视口像素，那里是桌面屏幕截图像素），而 Claude 对同名成员（例如 `screenshot` 或 `key`）的调用通过 `toolset_name` 加以区分。

## 启用可选成员

四个成员工具默认禁用：`javascript_exec` 和 `file_upload` 是因为它们扩大了被操纵的页面可能让 Claude 做的事情，`read_console` 和 `read_network` 是因为并非每个浏览器自动化栈都能提供这些日志，而且它们扩大了到达 Claude 的页面控制内容的范围。仅当您的执行器实现了某个成员且任务需要它时，才使用 `configs` 启用它（例如 `"configs": {"file_upload": {"enabled": true}}`）。

### 上传文件

`file_upload` 直接设置 `<input type="file">` 元素上的文件，这比驱动原生文件选择器更可靠。它的 `target` 只能是引用，因为该调用需要元素的身份，并且它接受 `paths`、`document_ids` 或两者：

* `paths` 是执行器文件系统上的文件路径，适用于执行器可以直接读取您应用程序文件的部署（与您填充下载的 `path` 的条件相同）。
* `document_ids` 是您的应用程序为浏览器暂存的文件的标识符，适用于执行器无法直接读取的部署。您的应用程序定义这些标识符的含义；请像限定 `paths` 那样限定它们的解析范围，仅限于为此任务暂存的文件。

```json
{
  "type": "tool_use",
  "id": "toolu_01N7gVzFEfZjLjgsYwnrPgrF",
  "name": "file_upload",
  "toolset_name": "browser",
  "input": {
    "target": { "type": "ref", "ref": "ref_12" },
    "paths": ["/home/user/uploads/summary.pdf"],
    "tab_id": "tab-2"
  }
}
```

Claude 在读取不受信任的页面时写下这些路径，因此不受限制的实现会让恶意页面指示将执行器可读取的任何文件上传到该页面控制的网站。仅当您的执行器解析每个路径（跟随符号链接和 `..` 段）并且不接受专用的、列入允许列表的上传目录（其中仅包含用于该任务的文件）之外的任何内容时，才启用该成员。不要为此重用浏览器的下载目录；如果这样做，页面导致浏览器下载的每个文件都将变得可上传。

### 在页面中运行 JavaScript

`javascript_exec` 在页面的上下文中运行 Claude 编写的表达式，并以文本形式返回最后一个表达式的值；Claude 编写的是表达式，而不是 `return` 语句。代码以页面的完整权限运行，包括其 cookie、存储和同源请求。仅在不持有任何凭据的会话中启用该成员，保持[安全注意事项](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#security-considerations)中的域名允许列表有效，将返回值视为不受信任的输入，并记录 Claude 发出的代码。

### 读取控制台和网络活动

`read_console` 返回标签页的控制台条目，`read_network` 返回其网络请求，二者都以文本形式返回，自上次读取该标签页以来累积的每个条目占一行。控制台行包含一条日志、警告或错误条目；网络行包含方法、URL、状态、MIME 类型和耗时。条目仅从您的浏览器自动化附加到该标签页的那一刻起才存在，因此结果为空并不意味着一个早已打开的标签页没有任何流量。

这些成员让 Claude 无需反复截图即可诊断行为异常的页面（例如加载图标背后的失败请求、无响应按钮背后的脚本错误）。控制台和网络条目由页面控制，并且经常包含机密信息，例如请求 URL 中的令牌，因此在返回之前，请对您不希望进入 Claude 上下文的类凭据值进行脱敏，并截断过长的条目。

## 使用 `browser_state` 跟踪标签页

Claude 通过 `tab_id` 来指代标签页，您的应用程序是哪些标签页存在的唯一可信来源，您需要在 `browser_state` 内容块中报告该状态。Claude 永远不会直接接收该内容块：API 会根据它渲染出 Claude 读取的文本。

```json
{
  "type": "browser_state",
  "tabs": [
    {
      "tab_id": "tab-1",
      "title": "Documentation",
      "url": "https://example.com/docs",
      "active": true
    },
    { "tab_id": "tab-2", "title": "Pricing", "url": "https://example.com/pricing" }
  ]
}
```

* `tabs` 是调用结束后所有已打开标签页的完整清单，而不是增量。它可以为空；只要不为空，就必须恰好有一个条目带有 `"active": true`。
* `state_changes`（此处未展示）报告调用的副作用：对于该调用打开且在调用完成时仍处于打开状态的每个标签页，提供一个 `tab_opened` 条目，其 `tab_id` 也必须出现在 `tabs` 中；此外还包括[下载事件](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#report-downloads)。没有需要报告的内容时请省略该字段；空数组会被拒绝。
* 仅在响应浏览器成员调用的结果上发送该内容块，每个 `tool_result` 最多发送一次，并且永远不要在带有 `is_error: true` 的结果上发送。通过省略该内容块来表达"没有需要报告的标签页状态"。
* API 会将 `tabs` 以及 `state_changes` 中的所有下载条目渲染为供 Claude 阅读的文本。接下来的两个小节以及[报告下载](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#report-downloads)展示了这些文本。

**`tab_id` 的值由您分配。** 任何稳定的字符串都可以，例如您的自动化库的页面标识符或您自己的计数器，只要您不在具有该标识符的标签页仍在较早的结果中被列为打开状态时重复使用该 `tab_id` 即可。API 对该内容块强制执行以下限制：

* 每个 `tab_id`、`title` 和 `url` 最多 4,096 个字符，`tab_id` 必须非空，并且它们都不得包含控制字符（包括换行符）或 Unicode 行分隔符或段落分隔符。
* 一个内容块最多可列出 100 个标签页和 200 个状态变更。
* 同样的限制也适用于 Claude 传递给 `switch_tab` 和 `close_tab` 的 `tab_id`，因为 API 会将其渲染到结果文本中，因此对于 `tab_id` 违反这些限制的调用，请以错误结果而非 `browser_state` 内容块来回应。

<Warning>
  标签页标题和 URL 来自页面，并会被渲染为 Claude 读取的文本，因此它们是 “prompt injection”（提示注入）的攻击面。API 会原样渲染标签页 URL，因此请在填充 `tabs` 之前清理页面提供的 URL。API 在渲染标题时会转义其中的双引号和反斜杠，因此不要预先转义标题（预先转义的标题到达 Claude 时会被双重转义）；截断或丢弃可疑标题仍然是值得做的。下载条目的 `url`、`path` 和 `error` 也会被渲染为 Claude 读取的文本，因此也应将它们视为不可信内容；API 会像处理标题一样对它们加引号并转义。API 强制执行的长度和字符限制只是最低要求，而不是防御措施。
</Warning>

### 标签页管理结果

对于 `new_tab`、`switch_tab`、`close_tab` 和 `list_tabs`，成功结果的 `content` 恰好是一个 `browser_state` 内容块，不含文本或图像，由 API 编写 Claude 接收的文本。`new_tab` 结果的内容块还必须恰好携带一个 `tab_opened` 状态变更，其 `tab_id` 与标记为 `active: true` 的条目相匹配。

| 成员           | Claude 接收的文本                                                                                             |
| ------------ | -------------------------------------------------------------------------------------------------------- |
| `switch_tab` | `Switched to tab {tab_id}`，取自调用的 `input.tab_id`                                                          |
| `close_tab`  | `Closed tab {tab_id}`，取自调用的 `input.tab_id`                                                               |
| `new_tab`    | `Created new tab with tab_id: {tab_id}, URL: {url}. It is now the current tab.`，取自标记为 `active: true` 的条目 |
| `list_tabs`  | `Available tabs:` 后跟每个标签页一行，或者当 `tabs` 为空时为 `No tabs available`                                          |

一个 `list_tabs` 结果，其内容块列出两个标签页且第一个处于活动状态，渲染结果如下，每行缩进两个空格，且仅在活动标签页后追加 `(current)`：

```text wrap
Available tabs:
  • tab_id tab-1: "Documentation" (https://example.com/docs) (current)
  • tab_id tab-2: "Pricing" (https://example.com/pricing)
```

这些成员之一的错误结果则相反：`content` 中是普通的错误文本，`is_error: true`，并且没有 `browser_state` 内容块。

例如，当 Claude 调用 `new_tab`（其 `input` 为空）时，您的执行器打开该标签页，将其设为活动状态，并返回带有一个 `tab_opened` 条目的清单：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01WvHSbQVV9j5nWGvTmk4vNL",
      "toolset_name": "browser",
      "content": [
        {
          "type": "browser_state",
          "tabs": [
            { "tab_id": "tab-1", "title": "Documentation", "url": "https://example.com/docs" },
            { "tab_id": "tab-2", "title": "Pricing", "url": "https://example.com/pricing" },
            { "tab_id": "tab-3", "title": "", "url": "about:blank", "active": true }
          ],
          "state_changes": [{ "type": "tab_opened", "tab_id": "tab-3" }]
        }
      ]
    }
  ]
}
```

Claude 接收到 `Created new tab with tab_id: tab-3, URL: about:blank. It is now the current tab.`。请像此处一样报告打开标签页时的 URL，而不是之后重定向到的 URL；之后的结果会报告该标签页当时的 URL。

### 其他结果上的标签页上下文

在所有其他成员上，该内容块是可选的：当已打开标签页的集合、活动标签页、或某个标签页的标题或 URL 发生变化时，或者当有 `state_changes` 需要报告时发送它，并且始终包含完整的 `tabs` 清单。当一个结果同时携带文本和 `browser_state` 内容块时，API 会在该结果的文本后追加一个 `Tab Context` 页脚，与您的文本之间以一个空行分隔，这样 Claude 无需单独调用 `list_tabs` 即可获得新状态：

```text wrap
Tab Context:
- Executed on tab_id: tab-1
- Available tabs:
  • tab_id tab-1: "Documentation" (https://example.com/docs)
  • tab_id tab-2: "Pricing" (https://example.com/pricing)
```

`Executed on` 指明调用运行所在的标签页，即存在 `tab_id` 输入时为该输入，否则为活动标签页，并且页脚中的标签页行不带 `(current)` 标记。不要自行追加此文本；请发送结构化内容块并让 API 来渲染它。页脚会被去重，因此相同的标签页状态不会在后续结果上再次渲染，大量填充该内容块不会产生任何成本。

有三种情况即使存在该内容块也不会渲染页脚：

* 任何 `zoom` 结果。
* 没有 `text` 内容块的结果（例如仅包含图像的 `screenshot` 结果）。对于该结果，不会渲染或记住任何内容；标签页上下文会出现在下一个同时携带文本和 `browser_state` 内容块的结果上，因此如果您希望标签页变化在同一个结果中传达给 Claude，请在图像旁附上一个简短的文本块。例外情况是内容块报告了[下载事件](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#report-downloads)的结果。API 会将下载行作为一个文本块添加，页脚会像在任何带文本的结果上一样跟在它们之后。
* 调用未携带 `tab_id`，且结果的 `tabs` 列表为空，因为此时没有可指明的标签页。

例如，当 Claude 在本会话早些时候点击"Pricing"链接（`ref_5`）时，页面在一个 Claude 并未请求的新标签页中打开了它，如果没有报告，Claude 将不得不调用 `list_tabs` 才能发现它。请返回点击的确认信息，外加一个在 `state_changes` 中指明所打开标签页的内容块，并标记您的执行器保留为活动状态的那个标签页：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01EgTXj1FjE2FCTt2zNFWLao",
      "toolset_name": "browser",
      "content": [
        { "type": "text", "text": "Clicked element ref_5." },
        {
          "type": "browser_state",
          "tabs": [
            {
              "tab_id": "tab-1",
              "title": "Documentation",
              "url": "https://example.com/docs",
              "active": true
            },
            { "tab_id": "tab-2", "title": "Pricing", "url": "https://example.com/pricing" }
          ],
          "state_changes": [{ "type": "tab_opened", "tab_id": "tab-2" }]
        }
      ]
    }
  ]
}
```

Claude 接收到 `Clicked element ref_5.`，后跟前面展示的 Tab Context 页脚。在失败的调用期间打开的标签页不会获得 `tab_opened` 条目，因为错误结果不携带 `browser_state`；它会改为出现在下一个成功结果的 `tabs` 清单中。在批处理中，请将内容块附加到发生变化的那次调用的结果上，并且为每个成功的标签页管理结果提供各自的内容块，即使同一轮中较早的结果已经报告了相同的状态。

### 报告下载

当点击或导航启动文件下载时，请在发生下载的那次调用的结果中通过 `state_changes` 报告，并通过您分配的 `download_id` 在各结果之间关联。下载是异步运行的，可能跨越多个结果，因此有三种事件类型：

| `type`               | 字段                                        | 何时发送                                                                                                                                                                                          |
| -------------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `download_started`   | `download_id`、`url`                       | 在下载开始时所在调用的结果上发送。`url` 是经过重定向后提供该文件的最终 URL。                                                                                                                                                   |
| `download_completed` | `download_id`、`url`、`path?`、`size_bytes?` | 在下载完成时正在运行的后续调用的结果上发送。仅当同一环境中的其他工具（例如 [bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)或 `file_upload`）可以在该路径读取文件时才包含 `path`；否则 `download_id` 是该下载的唯一标识符。 |
| `download_failed`    | `download_id`、`url`、`error?`              | 当下载失败或被取消时，如果浏览器提供了原因，则在 `error` 中给出。                                                                                                                                                         |

API 会按条目出现的顺序，将每个条目渲染为一行供 Claude 阅读的文本。这些行位于结果文本之后（以一个空行隔开），并位于任何 Tab Context 页脚之前。所有类型的成员结果都会带上这些行，包括 `zoom` 和标签页管理结果。对于没有 `text` 块的结果，这些行会作为一个单独的文本块添加。每一行都会给出 `download_id` 和 `url`；如果您发送了相应字段，还会给出 `path` 和 `size_bytes`（针对 `download_completed`）或 `error`（针对 `download_failed`）。您无需在自己的文本中描述下载。API 会用双引号包裹 `url`、`path` 和 `error`，并转义其中的双引号和反斜杠，因此不要预先转义这些值。

例如，在 Pricing 标签页中点击 “Download price list (CSV)”（`ref_8`）会触发一次下载，因此该点击的结果带有一个 `download_started` 条目，其 `download_id` 为 `"dl-1"`，并附有文件的 URL。下载在后续一次 `screenshot` 调用运行期间完成，因此该结果的 `content` 包含图像、一个文本块（例如 `Screenshot captured.`），以及下面这个 `browser_state` 块，它使用同一个 `download_id` 报告下载完成：

```json
{
  "type": "browser_state",
  "tabs": [
    { "tab_id": "tab-1", "title": "Documentation", "url": "https://example.com/docs" },
    {
      "tab_id": "tab-2",
      "title": "Pricing",
      "url": "https://example.com/pricing",
      "active": true
    }
  ],
  "state_changes": [
    {
      "type": "download_completed",
      "download_id": "dl-1",
      "url": "https://example.com/pricing/price-list.csv",
      "path": "/home/user/downloads/price-list.csv",
      "size_bytes": 48213
    }
  ]
}
```

Claude 接收到 `Screenshot captured.`，后跟一个空行和如下一行：

```text wrap
Download completed with download_id: dl-1, URL: "https://example.com/pricing/price-list.csv". Saved to "/home/user/downloads/price-list.csv". Size: 48213 bytes.
```

下载报告遵循以下规则：

* 在单个内容块中，每个 `download_id` 最多发送一个条目，因此在同一次调用中开始并完成的下载只报告 `download_completed`。
* 绝不要在 `is_error: true` 的结果上发送 `state_changes`；在失败调用期间发生的下载事件请在下一个成功结果上报告。
* `state_changes` 不是进行中下载的清单；每个事件只报告一次。
* 每个条目仅携带其 `type` 所声明的字段。`size_bytes` 是非负整数，`download_id` 非空，并且 `download_id`、`url`、`path` 和 `error` 各自最多 4,096 个字符，不含控制字符或 Unicode 行分隔符或段落分隔符。`url` 来自远程服务器，在重定向后通常携带已签名的查询字符串凭据，因此请剥离您不希望出现在 Claude 上下文中的查询参数，并在报告它或将其用于文件系统路径之前对其进行清理。

## 处理错误

将失败的调用作为普通错误结果报告给 Claude：`is_error: true`、说明出错原因的文本内容、回显的 `toolset_name`，并且没有 `browser_state` 内容块。

### 从您的执行器返回错误

请使错误文本具体明确，因为 Claude 会阅读它并做出调整：`Error: Navigation to https://example.com/status timed out after 30 seconds. The page may be unavailable.` 为 Claude 提供了可据以行动的信息，而简单的 `Error: navigation failed` 则没有。其他常见情况：

<AccordionGroup>
  <Accordion title="被拒绝的导航协议">
    ```json
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01LeUTyqkhRxBFq1QTG3pkwN",
      "toolset_name": "browser",
      "is_error": true,
      "content": "Error: Navigation refused. Only http and https URLs are allowed."
    }
    ```
  </Accordion>

  <Accordion title="过期或未知的元素引用">
    ```json
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01D7FLrfh4GYq7yT1ULFeyMV",
      "toolset_name": "browser",
      "is_error": true,
      "content": "Error: ref_3 is stale or not found on the current page. Re-read the page to get fresh references."
    }
    ```
  </Accordion>

  <Accordion title="已禁用或未实现的成员">
    ```json
    {
      "type": "tool_result",
      "tool_use_id": "toolu_013h2Q55HcNwVyapSpy2s5ZG",
      "toolset_name": "browser",
      "is_error": true,
      "content": "Error: javascript_exec is not enabled in this environment."
    }
    ```
  </Accordion>

  <Accordion title="因本轮次中较早的失败而被跳过">
    当[批处理操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)中对 `ref_3` 的 `left_click` 因前面展示的过期引用错误而失败时，其后的 `type` 和 `key` 调用各自获得此结果：

    ```json
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01FkP8rTz6uYh2mNq4LsXw7v",
      "toolset_name": "browser",
      "is_error": true,
      "content": "Not executed: an earlier action in this turn failed."
    }
    ```
  </Accordion>
</AccordionGroup>

### 请求错误

API 会校验工具集条目以及对话中的每个成员 `tool_use` 和 `tool_result` 内容块。当其中之一格式错误时，API 会在 Claude 运行之前返回 `invalid_request_error`。在下表中，左列指明您发送的内容。

| 请求                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | 失败原因及处理方法                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 工具集条目不接受的选项或组合，例如：条目本身上的 `name`、`strict: true`、`input_examples`、`defer_loading`，不是成员名称的 `configs` 键，成员 `configs` 值中除 `enabled` 或 `defer_loading` 之外的字段（[配置工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#configure-the-toolset)），`defer_loading` 值不同的已启用成员（[配置工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#configure-the-toolset)），未启用任何成员的 `configs`，`allowed_callers` 中的代码执行调用方，请求中的 `fine-grained-tool-streaming-2025-05-14` beta 标头，指定 `browser` 或某个成员的 `tool` 类型 `tool_choice`，或者第二个浏览器工具集条目或另一个名为 `browser` 的工具 | 客户端工具集不支持这些。有关每条规则及其替代方案，请参阅[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)。                                             |
| 回应成员调用的 `tool_result` 缺少 `"toolset_name": "browser"` 或使用了不同的值，或者在其调用并非成员调用的结果上带有 `toolset_name`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | 在成员结果上原样回传 `toolset_name`，且仅在成员结果上回传。                                                                                                                                              |
| 来自较早轮次的成员 `tool_use` 没有匹配的 `tool_result`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 回应每一个成员调用，包括在失败后您未运行的调用。                                                                                                                                                           |
| 成员结果中包含 `text`、`image` 或 `browser_state` 之外的内容块                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | 成员结果仅接受这三种块类型。                                                                                                                                                                     |
| 违反[使用 `browser_state` 跟踪标签页](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#track-tabs-and-page-state)中规则的 `browser_state` 块，例如：位于 `is_error: true` 结果上或位于不回应浏览器成员调用的结果上的块，一个结果中有多个该块，非空的 `tabs` 中不是恰好有一个 `active: true` 条目，重复的 `tab_id`，空的 `state_changes` 数组，其 `tab_id` 不在 `tabs` 中的 `tab_opened`，针对同一个 `download_id` 的两个状态变更，或其 `type` 未声明的状态变更字段（[报告下载](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#report-downloads)），或者超出限制的字段                                                                                           | 修正该块。"没有需要报告的内容"应通过省略该块或 `state_changes` 字段来表达，绝不能使用空值。                                                                                                                            |
| 成功的 `new_tab`、`switch_tab`、`close_tab` 或 `list_tabs` 结果的 `content` 不是恰好一个 `browser_state` 块，或者 `new_tab` 结果中没有恰好一个与活动标签页匹配的 `tab_opened`                                                                                                                                                                                                                                                                                                                                                                                                                                                                | API 根据该块渲染这些结果，并要求其完全符合该结构；请参阅[标签页管理结果](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#tab-management-results)。                                 |
| 结果中的 `image` 超出您模型的[图像尺寸限制](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)，或者在请求包含[超过 20 张图像](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#request-limits)时超出适用的更严格的单图限制（计入较早结果中的截图和 `zoom` 图像）                                                                                                                                                                                                                                                                                                                                                | API 不会缩小工具集图像。请在返回截图之前调整其大小（[调整截图尺寸以符合图像限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#handle-coordinate-scaling-for-higher-resolutions)）。 |
| 不支持 `browser_toolset_20260801` 的 `model`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 有关支持的模型，请参阅 [API 支持](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#availability-feature-support)。                                              |

## 限制

* **仅支持完整输入流式传输：**&#x5F53;您使用 “streaming”（流式传输）时，每个成员的 `input` 会作为一个完整的 `input_json_delta` 到达（[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)）。
* **元素引用为尽力而为：**&#x9AD8;度动态的页面（虚拟化列表、canvas 渲染的界面、滚动时重新渲染的页面）可能不会暴露稳定的引用，在这些情况下 Claude 会回退到使用截图和坐标点击。
* **`read_console` 和 `read_network` 取决于您的浏览器自动化：**&#x5B83;们只报告浏览器自动化能够捕获的内容，并且只从其附加到标签页的那一刻开始。
* **通用智能体限制同样适用：**"latency"（延迟）、视觉准确性和 “prompt injection”（提示注入）风险都沿袭自计算机使用（请参阅计算机使用工具的[限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#understand-computer-use-limitations)），并且其在[通过提示优化模型性能](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#optimize-model-performance-with-prompting)、[管理截图历史](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#manage-screenshot-history)和[遵循实施最佳实践](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#follow-implementation-best-practices)（操作延迟、操作验证和日志记录）下的指导同样适用于浏览器执行器。

## 定价和数据保留

浏览器使用遵循标准的[工具使用定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#pricing)。使用浏览器使用工具时：

**工具集定义开销：** 声明带有默认成员的 `browser_toolset_20260801` 会为请求增加约 6,600 个输入令牌（在 Claude Sonnet 5 上约为 6,670 个，在其他受支持的模型上约为 6,610 个），其中涵盖成员工具定义和工具使用系统提示。启用全部四个可选成员会增加约 880 个令牌，而使用 `configs` 禁用成员则会减少令牌数。请求的确切令牌数会在响应的 `usage` 中报告，您也可以使用[令牌计数端点](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)提前进行估算。

**额外的令牌消耗：**

* 工具结果中返回的屏幕截图和缩放图像，按图像输入计费（请参阅[视觉定价](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)）
* 返回给 Claude 的文本工具结果，例如无障碍树、页面文本以及控制台或网络条目

<Note>
  如果您在浏览器使用之外还同时使用计算机使用工具、bash 工具、文本编辑器工具或您自己的工具，这些工具各自有其令牌成本，详见其各自的文档页面。
</Note>

浏览器会话、下载和上传的文件保留在您的环境中；您返回的截图、页面文本和标签页状态是您的 API 请求内容的一部分，遵循标准保留政策，或者如果您有 ZDR 安排则遵循该安排。浏览器使用工具符合 ZDR 资格；请参阅 [API 和数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)了解各功能的保留期限和资格。

## 后续步骤

<CardGroup cols={2}>
  <Card title="使用 SDK 工具集进行浏览器和计算机使用" icon="code" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk">
    从合作伙伴的驱动程序开始，或者用 Python 或 TypeScript 编写您自己的驱动程序。SDK 会运行循环以及您配置的检查。
  </Card>

  <Card title="浏览器驱动程序示例" icon="github-logo" href="https://github.com/anthropics/claude-quickstarts/tree/main/browser-toolset">
    阅读一个适用于 SDK 浏览器工具集的极简 Python 和 TypeScript 驱动程序。它通过 Chrome DevTools Protocol 控制 Chromium，并非生产代码。
  </Card>

  <Card title="计算机使用工具" icon="computer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool">
    当任务超出浏览器范围时，让 Claude 控制完整的桌面；其实现指导同样适用于浏览器执行器。
  </Card>

  <Card title="处理工具调用" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls">
    格式化 `tool_result` 内容块，返回图像和错误，并继续对话。
  </Card>

  <Card title="工具参考" icon="book" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    浏览客户端工具集以及所有其他 Anthropic 提供的工具，包括它们的版本和参数。
  </Card>
</CardGroup>

## 可用性

浏览器使用工具在部分模型上可用，并具备零数据保留资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台                                                                                                                                                                                                                  | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)                                                  | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview) |
| 可用 | [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                        |
| 可用 | [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                                                                                  | [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                                                                                                                                                                                                      |
| 可用 | [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)                                                                                         | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                                                                                                                                                                                                    |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                                   | HIPAA                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)）               | 不具备资格                             |
| 详细信息 | 客户端工具。除标准 API 处理外，Anthropic 不运行浏览器操作，也不保留页面内容。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#data-retention) | 不在 HIPAA 就绪范围内；包含浏览器使用工具的请求不会被阻止。 |

适用的端点：

* `/v1/messages` 使用 `browser` 工具集
