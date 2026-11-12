---
title: 计算机使用工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool
description: 通过计算机使用工具（即 computer_toolset_20260801 客户端工具集），让 Claude 获得桌面环境的截图、鼠标和键盘控制能力。
featureMetadata:
  betaHeader: computer-use-2025-11-24, computer-use-2025-01-24
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 客户端工具，截图和文件在您的环境中捕获和存储，而非由 Anthropic 存储。
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
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
  details:
    - Claude Fable 5.2、Claude Mythos 5.2、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 通过 `computer_toolset_20260801` 工具集支持计算机使用。对于它们不接受的任何计算机使用工具版本（例如 Claude API 上的 `computer_20251124`），它们会返回 400 错误。要将 `computer_20251124` 集成迁移到该工具集，请参阅[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)。要从 `computer_20250124` 迁移，请参阅 Claude Haiku 5.5 迁移指南中的[将计算机使用迁移到工具集](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#computer-use-toolset)。
    - Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 4.6 和 Claude Opus 4.5 仅通过 `computer_20251124` 工具版本支持计算机使用，该版本需要 beta 标头；请参阅[较早的工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#earlier-tool-versions)。
---

<Note>
  **从 SDK 工具集开始：** 对于 `computer_toolset_20260801`，Python 和 TypeScript SDK 提供了一个计算机工具集类（beta 版）。您可以从[合作伙伴的集成](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#partner-integrations)开始，或参阅 SDK 工具集页面的[计算机工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#computer-toolset)部分。请将本页用作工具参考，或在您自行处理工具调用时参考本页。
</Note>

Claude 可以通过计算机使用工具与计算机环境交互，该工具提供截图功能以及鼠标/键盘控制，以实现自主的桌面交互。

计算机使用工具是一个由 Anthropic 定义的[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)：在 `tools` 中加入一个 `{"type": "computer_toolset_20260801"}` 条目，即可为 Claude 提供 17 个成员工具，例如 `screenshot`、`left_click`、`type` 和 `zoom`，而每次调用都由您的应用程序在您控制的环境中运行。该工具目前在 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/tools) 中不可用。Claude 的调用是 `tool_use` 块，其 `name` 为成员名称，并带有 `"toolset_name": "computer"`，通常在一个轮次中包含多个（即一个[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)）。

对于停留在网页内的任务，[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)更为合适：其成员工具直接读取并操作页面本身，且不需要完整的桌面环境。

有关支持该工具集的模型和平台，请参阅 [API 支持](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#availability-feature-support)。[较早的工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#earlier-tool-versions)列出了每个较早版本的 beta 标头以及可搭配使用的模型。

## 安全注意事项

计算机使用具有不同于标准 API 功能的独特风险。在与互联网交互时，这些风险会进一步加剧。

<Warning>
  为了将风险降至最低，请考虑采取以下预防措施：

  1. 使用权限最小化的专用虚拟机或容器，以防止直接的系统攻击或意外事故。
  2. 避免让模型访问敏感数据（例如账户登录信息），以防止信息被盗。
  3. 将互联网访问限制在域名允许列表内，以减少接触恶意内容的机会。
  4. 对于可能产生重大现实后果的决策以及任何需要明确同意的任务（例如接受 cookie、完成金融交易或同意服务条款），请让人工进行确认。
</Warning>

在某些情况下，Claude 会遵循内容中的命令，即使这些命令与您的指令相冲突。例如，网页上的指令或图像中包含的指令可能会覆盖您的指令或导致 Claude 出错。请采取预防措施，将 Claude 与敏感数据和操作隔离，以避免与 “prompt injection”（提示注入）相关的风险。

Anthropic 已训练模型抵御这些提示注入，并增加了一层额外的防御。如果您使用计算机使用工具，分类器会自动扫描工具返回的内容（例如截图），以标记潜在的提示注入。当这些分类器识别出潜在的提示注入时，它们会引导模型在执行指令之前先确认该指令是否确实来自您。

这种额外保护并不适用于所有用例（例如没有人工参与的用例），因此如果您希望选择退出并将其关闭，请[联系支持团队](https://support.claude.com/en/)。即使有这些分类器，前面列出的预防措施仍然很重要。

在您自己的产品中启用计算机使用之前，请告知最终用户相关风险并征得他们的同意。

## 快速入门

将计算机使用工具集以 `{"type": "computer_toolset_20260801"}` 的形式添加到 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 请求的 `tools` 数组中。该请求不需要 beta 标头。此示例还声明了[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)和 [bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)，Claude 通常会将它们与计算机使用一起使用：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "tools": [
        {
          "type": "computer_toolset_20260801"
        },
        {
          "type": "text_editor_20250728",
          "name": "str_replace_based_edit_tool"
        },
        {
          "type": "bash_20250124",
          "name": "bash"
        }
      ],
      "messages": [
        {
          "role": "user",
          "content": "Save a picture of a cat to my desktop."
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  tools:
    - type: computer_toolset_20260801
    - type: text_editor_20250728
      name: str_replace_based_edit_tool
    - type: bash_20250124
      name: bash
  messages:
    - role: user
      content: Save a picture of a cat to my desktop.
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[
          {"type": "computer_toolset_20260801"},
          {"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"},
          {"type": "bash_20250124", "name": "bash"},
      ],
      messages=[{"role": "user", "content": "Save a picture of a cat to my desktop."}],
  )
  print(response)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [
      {
        type: "computer_toolset_20260801"
      },
      {
        type: "text_editor_20250728",
        name: "str_replace_based_edit_tool"
      },
      {
        type: "bash_20250124",
        name: "bash"
      }
    ],
    messages: [{ role: "user", content: "Save a picture of a cat to my desktop." }]
  });

  console.log(response);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools =
      [
          new ComputerToolset20260801(),
          new ToolTextEditor20250728(),
          new ToolBash20250124(),
      ],
      Messages = [new UserMessageParam { Content = "Save a picture of a cat to my desktop." }],
  };

  var response = await client.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{
  		{OfComputerToolset20260801: &anthropic.ComputerToolset20260801Param{}},
  		{OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{}},
  		{OfBashTool20250124: &anthropic.ToolBash20250124Param{}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Save a picture of a cat to my desktop.")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  import com.anthropic.models.messages.ComputerToolset20260801;
  // ...
  import com.anthropic.models.messages.ToolBash20250124;
  import com.anthropic.models.messages.ToolTextEditor20250728;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(ComputerToolset20260801.builder().build())
          .addTool(ToolTextEditor20250728.builder().build())
          .addTool(ToolBash20250124.builder().build())
          .addUserMessage("Save a picture of a cat to my desktop.")
          .build();

      Message response = client.messages().create(params);
      IO.println(response);
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => 'Save a picture of a cat to my desktop.'],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [
          ['type' => 'computer_toolset_20260801'],
          [
              'type' => 'text_editor_20250728',
              'name' => 'str_replace_based_edit_tool',
          ],
          [
              'type' => 'bash_20250124',
              'name' => 'bash',
          ],
      ],
  );

  echo $response;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: [
      { type: "computer_toolset_20260801" },
      {
        type: "text_editor_20250728",
        name: "str_replace_based_edit_tool"
      },
      {
        type: "bash_20250124",
        name: "bash"
      }
    ],
    messages: [
      { role: "user", content: "Save a picture of a cat to my desktop." }
    ]
  )

  puts response
  ```
</CodeGroup>

当 Claude 在桌面上执行操作时，响应的 `stop_reason` 为 `tool_use`，并包含一个或多个成员 `tool_use` 块，每个块指定一个成员工具并带有 `"toolset_name": "computer"`。在此任务进行到一半、Claude 已收到桌面截图之后，响应可能如下所示：

```json Output
{
  "id": "msg_01UZ3bXcQH8mTqNhVfL9eK2p",
  "type": "message",
  "role": "assistant",
  "model": "claude-opus-5-5",
  "content": [
    {
      "type": "text",
      "text": "I'll open the web browser to find a picture of a cat."
    },
    {
      "type": "tool_use",
      "id": "toolu_01WkoTUvSHDzTBu2xnGk8Ep8",
      "name": "left_click",
      "toolset_name": "computer",
      "input": { "coordinate": [512, 742] }
    },
    {
      "type": "tool_use",
      "id": "toolu_017nJn3RgSCkTMwuZDb4uUov",
      "name": "screenshot",
      "toolset_name": "computer",
      "input": {}
    }
  ],
  "stop_reason": "tool_use",
  "stop_sequence": null
}
```

您的应用程序在自己的环境中按顺序运行每个调用，为每个 `tool_use` 块返回一个 `tool_result` 块，然后再次调用 API；[计算机使用的工作原理](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#how-computer-use-works)描述了这个循环，本页其余部分将展示如何实现它。

***

## 计算机使用的工作原理

<Steps>
  <Step title="为 Claude 提供计算机使用工具和用户提示">
    * 将计算机使用工具集（以及可选的其他工具）添加到 API 请求的 `tools` 数组中。
    * 包含一个需要桌面交互的用户提示，例如"将一张猫的图片保存到我的桌面。"
  </Step>

  <Step title="Claude 以成员工具调用进行响应">
    * Claude 评估在桌面上执行操作是否有助于处理用户的请求。
    * 如果是，Claude 会以一个或多个成员 `tool_use` 块进行响应，例如 `screenshot`、`left_click` 或 `type`，每个块都带有 `"toolset_name": "computer"`。包含多个此类块的响应即为一个[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)。
    * API 响应的 `stop_reason` 为 `tool_use`，表示这是一个工具使用请求。
  </Step>

  <Step title="按顺序运行调用并返回结果">
    * 按顺序遍历响应中的每个 `tool_use` 块。对于每个块，根据成员 `name` 和 `toolset_name` 进行分派，并使用该块的 `input` 在您的容器或虚拟机上执行该操作。
    * 使用一条新的 `user` 消息继续对话，该消息为每个 `tool_use` 块包含一个 `tool_result` 块，通过 `tool_use_id` 进行匹配，并且每个块都回显 `"toolset_name": "computer"`。对于 `screenshot` 和 `zoom` 返回图像；对于其他操作，返回 `OK` 之类的简短文本即可。
    * 如果某个操作失败，请为该块返回 `is_error: true`，并按照[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)中的说明回复批次中的其余部分。
  </Step>

  <Step title="Claude 持续执行直到任务完成">
    * Claude 分析工具结果，以确定是否需要更多操作或任务是否已完成。
    * 如果 Claude 判断需要更多操作，它会再次以 `tool_use` `stop_reason` 进行响应；返回第 3 步。
    * 否则，它会向用户返回文本响应。
  </Step>
</Steps>

在没有用户输入的情况下重复第 3 步和第 4 步，即构成 “agent loop”（智能体循环）：Claude 以工具使用请求进行响应，而您的应用程序将执行该请求的结果返回给 Claude。

### 批量操作

Claude 可以规划一个简短的操作序列，例如点击、输入，然后截图，并在一个响应中一起返回。这称为 “batch action”（批量操作）；它使用与[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)相同的响应结构，但有一个区别：您需要按顺序而不是并发地运行这些块。

包含三个操作的批次的响应如下所示：

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01HqCF3nJ4Vzr8sTkPZ2wxYA",
      "name": "left_click",
      "toolset_name": "computer",
      "input": { "coordinate": [640, 60] }
    },
    {
      "type": "tool_use",
      "id": "toolu_01Ppr3sZ3TnE9m6VUu4RyH2K",
      "name": "type",
      "toolset_name": "computer",
      "input": { "text": "pictures of cats" }
    },
    {
      "type": "tool_use",
      "id": "toolu_01Xf5W1sD8Q9aBcJ7kLmN2pQ",
      "name": "screenshot",
      "toolset_name": "computer",
      "input": {}
    }
  ]
}
```

为每个 `tool_use` 块返回一个 `tool_result` 块，通过 `tool_use_id` 进行匹配，并全部放在下一条 `user` 消息中。成员工具的每个结果都必须带有 `"toolset_name": "computer"`；省略它的结果，或者指定了与其 `tool_use` 块不同工具集的结果，都会被拒绝。只有 `screenshot` 和 `zoom` 的结果需要图像；对于其他成员，`OK` 之类的简短文本确认即可（`cursor_position` 以文本形式返回坐标）：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01HqCF3nJ4Vzr8sTkPZ2wxYA",
      "toolset_name": "computer",
      "content": [{ "type": "text", "text": "OK" }]
    },
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01Ppr3sZ3TnE9m6VUu4RyH2K",
      "toolset_name": "computer",
      "content": [{ "type": "text", "text": "OK" }]
    },
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01Xf5W1sD8Q9aBcJ7kLmN2pQ",
      "toolset_name": "computer",
      "content": [
        {
          "type": "image",
          "source": {
            "type": "base64",
            "media_type": "image/png",
            "data": "iVBORw0KGgo..."
          }
        }
      ]
    }
  ]
}
```

**按顺序运行块，并在第一次失败时停止。** 批次中较后的操作通常依赖于较早的操作：此示例中的 `type` 会将文本输入到前一次点击所聚焦的位置。请按照块在 `content` 中出现的顺序依次运行，如果其中一个失败，则不要运行其余的块。每个 `tool_use` 块仍然需要一个 `tool_result`，因此请按如下方式回复该批次：

* 对于每个成功的操作，返回其正常结果。
* 对于失败的操作，返回 `is_error: true`，并附上说明出错原因的文本描述。
* 对于批次中之后的每个操作，返回 `is_error: true`，并使用与下面完全相同的文本（浏览器使用工具使用其自己的[中止文本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)）：

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01Xf5W1sD8Q9aBcJ7kLmN2pQ",
  "toolset_name": "computer",
  "is_error": true,
  "content": "Not executed: an earlier computer action in this turn failed."
}
```

然后，Claude 会读取哪些操作成功、哪个操作失败以及哪些操作被跳过，并在下一个轮次中重新规划。如果请求中批次里的任何 `tool_use` 块未得到回复，则该请求会被拒绝并返回 `invalid_request_error`，因此只读取第一个块的智能体循环会在下一次调用时失败。如果您的应用程序需要让人工确认重要操作，请在每个块运行之前进行该检查，因为一个批次可以在一个轮次内完成一个多步骤操作。

Claude 通常以 `screenshot` 结束一个批次，以便在决定下一步操作之前观察结果。当批次不以截图结束时，您的应用程序可以在批次中最后一个结果上附加一张截图作为额外的 `image` 块，以便 Claude 始终能收到屏幕的当前状态，与等待 Claude 请求截图相比，这样可以节省一次往返。您也可以提示 Claude 在每个批次结束时进行截图（请参阅[通过提示优化模型性能](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#optimize-model-performance-with-prompting)）。

### 计算环境

计算机使用需要一个沙盒化的计算环境，Claude 可以在其中安全地与应用程序和网络交互。该环境包括：

1. **虚拟显示器：** 一个虚拟 X11 显示服务器（使用 Xvfb），用于渲染桌面界面，Claude 以截图形式接收该界面，并通过鼠标/键盘操作对其进行控制。

2. **桌面环境：** 一个在 Linux 上运行的轻量级 UI，带有窗口管理器（Mutter）和面板（Tint2），为 Claude 提供一致的图形交互界面。

3. **应用程序：** 预装的 Linux 应用程序，例如 Firefox、LibreOffice、文本编辑器和文件管理器，Claude 可以使用它们来完成任务。

4. **工具实现：** 集成代码，将 Claude 的抽象工具请求（例如"移动鼠标"或"截图"）转换为虚拟环境中的实际操作。

5. **智能体循环：** 一个处理 Claude 与环境之间通信的程序，它将 Claude 的操作发送到环境，并将结果（截图、命令输出）返回给 Claude。

当您使用计算机使用时，Claude 并不会直接连接到该环境。相反，您的应用程序会：

1. 接收 Claude 的工具使用请求。
2. 将其转换为您计算环境中的操作。
3. 捕获结果（例如截图和命令输出）。
4. 将这些结果返回给 Claude。

出于安全和隔离考虑，参考实现将所有这些都运行在一个 Docker 容器内，并配置了适当的端口映射，用于查看环境并与之交互。

***

## 如何实现计算机使用

要升级现有的 `computer_20251124` 集成？请从[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)开始；本节其余内容同时适用于新集成和迁移后的集成。

<Tip>
  [计算机使用参考实现](https://github.com/anthropics/claude-quickstarts/tree/main/computer-use-demo)是一个完整的可运行示例：包括一个适用于计算机使用的[容器化环境](https://github.com/anthropics/claude-quickstarts/blob/main/computer-use-demo/Dockerfile)、[计算机使用工具](https://github.com/anthropics/claude-quickstarts/tree/main/computer-use-demo/computer_use_demo/tools)的实现、一个调用 Claude API 并运行工具的[智能体循环](https://github.com/anthropics/claude-quickstarts/blob/main/computer-use-demo/computer_use_demo/loop.py)，以及一个用于容器、循环和工具的 Web 界面。要使用 `computer_toolset_20260801`，请选择一个[支持它的模型](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#availability-feature-support)，然后在 Web 界面侧边栏的 **Tool Versions** 下选择该工具集。
</Tip>

### 理解智能体循环

计算机使用的核心是"智能体循环"：在这个循环中，Claude 请求工具操作，您的应用程序运行这些操作并将结果返回给 Claude。该循环使用您在[快速入门](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#quick-start)中创建的客户端、一个仅声明计算机使用工具集的 `tools` 数组，以及[实现计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#implement-the-computer-use-tool)中的工具调用处理辅助函数。如果您还声明了其他工具（例如快速入门中的 bash 和文本编辑器工具），请在同一轮处理中分派它们的 `tool_use` 块；该辅助函数只回复计算机使用的成员调用，而循环会将没有任何已回复调用的轮次视为已完成。以下是一个简化示例：

<CodeGroup exclude="shell">
  ```python Python
  def sampling_loop(model: str, messages: list[MessageParam], max_iterations: int = 10):
      """
      Run the computer-use agent loop until Claude stops requesting tools
      or the iteration limit is reached.
      """
      for _ in range(max_iterations):
          response = client.messages.create(
              model=model,
              max_tokens=4096,
              messages=messages,
              tools=TOOLS,
          )

          # Add Claude's response to the conversation history
          messages.append({"role": "assistant", "content": response.content})

          # Run the actions Claude requested, in order, and collect the results
          tool_results = process_tool_calls(response)
          if not tool_results:
              return messages  # No more tool use; task complete

          # Send every result back to Claude in a single user message
          messages.append({"role": "user", "content": tool_results})

      return messages
  ```

  ```typescript TypeScript
  async function samplingLoop(
    model: string,
    messages: Anthropic.MessageParam[],
    maxIterations = 10
  ): Promise<Anthropic.MessageParam[]> {
    // Run the computer-use agent loop until Claude stops requesting tools
    // or the iteration limit is reached.
    for (let iteration = 0; iteration < maxIterations; iteration++) {
      const response = await client.messages.create({
        model,
        max_tokens: 4096,
        messages,
        tools
      });

      // Add Claude's response to the conversation history
      messages.push({ role: "assistant", content: response.content });

      // Run any tools Claude requested and collect results
      const toolResults = processToolCalls(response);
      if (toolResults.length === 0) {
        return messages; // No more tool use; task complete
      }

      // Send tool results back to Claude for the next iteration
      messages.push({ role: "user", content: toolResults });
    }

    return messages;
  }
  ```

  ```csharp C#
  async Task<List<MessageParam>> SamplingLoop(
      Model model,
      List<MessageParam> messages,
      int maxIterations = 10
  )
  {
      // Run the computer-use agent loop until Claude stops requesting tools
      // or the iteration limit is reached.
      for (var iteration = 0; iteration < maxIterations; iteration++)
      {
          var response = await client.Messages.Create(
              new MessageCreateParams
              {
                  Model = model,
                  MaxTokens = 4096,
                  Messages = messages,
                  Tools = tools,
              }
          );

          // Add Claude's response to the conversation history
          messages.Add(
              new AssistantMessageParam
              {
                  Content = response
                      .Content.Select(block => new ContentBlockParam(block.Json))
                      .ToList(),
              }
          );

          // Run any tools Claude requested and collect results
          var toolResults = ProcessToolCalls(response);
          if (toolResults.Count == 0)
          {
              return messages; // No more tool use; task complete
          }

          // Send tool results back to Claude for the next iteration
          messages.Add(new UserMessageParam { Content = toolResults });
      }

      return messages;
  }
  ```

  ```go Go
  // samplingLoop runs the computer-use agent loop until Claude stops
  // requesting tools or the iteration limit is reached.
  func samplingLoop(ctx context.Context, model anthropic.Model, messages []anthropic.MessageParam, maxIterations int) ([]anthropic.MessageParam, error) {
  	for range maxIterations {
  		response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  			Model:     model,
  			MaxTokens: 4096,
  			Messages:  messages,
  			Tools:     tools,
  		})
  		if err != nil {
  			return nil, err
  		}

  		// Add Claude's response to the conversation history
  		messages = append(messages, response.ToParam())

  		// Run the actions Claude requested, in order, and collect the results
  		toolResults := processToolCalls(response)
  		if len(toolResults) == 0 {
  			return messages, nil // No more tool use; task complete
  		}

  		// Send every result back to Claude in a single user message
  		messages = append(messages, anthropic.NewUserMessage(toolResults...))
  	}
  	return messages, nil
  }
  ```

  ```java Java
  /**
   * Run the computer-use agent loop until Claude stops requesting tools
   * or the iteration limit is reached.
   */
  List<MessageParam> samplingLoop(Model model, List<MessageParam> messages, int maxIterations) {
      for (int iteration = 0; iteration < maxIterations; iteration++) {
          Message response = client.messages().create(MessageCreateParams.builder()
              .model(model)
              .maxTokens(4096)
              .messages(messages)
              .addTool(COMPUTER_TOOLSET)
              .build());

          // Add Claude's response to the conversation history
          messages.add(response.toParam());

          // Run any tools Claude requested and collect results
          List<ContentBlockParam> toolResults = processToolCalls(response);
          if (toolResults.isEmpty()) {
              return messages; // No more tool use; task complete
          }

          // Send tool results back to Claude for the next iteration
          messages.add(MessageParam.ofUserOfContentBlockParams(toolResults));
      }
      return messages;
  }
  ```

  ```php PHP
  /**
   * Run the computer-use agent loop until Claude stops requesting tools
   * or the iteration limit is reached.
   */
  function samplingLoop(Model|string $model, array $messages, int $maxIterations = 10): array
  {
      global $client, $tools;

      for ($iteration = 0; $iteration < $maxIterations; $iteration++) {
          $response = $client->messages->create(
              model: $model,
              maxTokens: 4096,
              messages: $messages,
              tools: $tools,
          );

          // Add Claude's response to the conversation history
          $messages[] = MessageParam::with(role: Role::ASSISTANT, content: $response->content);

          // Run any tools Claude requested and collect results
          $toolResults = processToolCalls($response);
          if ($toolResults === []) {
              return $messages; // No more tool use; task complete
          }

          // Send tool results back to Claude for the next iteration
          $messages[] = MessageParam::with(role: Role::USER, content: $toolResults);
      }

      return $messages;
  }
  ```

  ```ruby Ruby
  # Run the computer-use agent loop until Claude stops requesting tools
  # or the iteration limit is reached.
  def sampling_loop(model, messages, max_iterations: 10)
    max_iterations.times do
      response = CLIENT.messages.create(
        model: model,
        max_tokens: 4096,
        messages: messages,
        tools: TOOLS
      )

      # Add Claude's response to the conversation history
      messages << { role: "assistant", content: response.content }

      # Run the actions Claude requested, in order, and collect the results
      tool_results = process_tool_calls(response)
      return messages if tool_results.empty? # No more tool use; task complete

      # Send every result back to Claude in a single user message
      messages << { role: "user", content: tool_results }
    end

    messages
  end
  ```
</CodeGroup>

该循环会一直持续，直到 Claude 在不请求任何工具的情况下进行响应（任务完成），或达到最大迭代次数限制。这一保护措施可防止可能导致意外 API 成本的无限循环。

### 通过提示优化模型性能

1. 指定简单、定义明确的任务，并为每个步骤提供明确的指令。
2. Claude 有时会在没有明确检查结果的情况下假定其操作已成功。为防止这种情况，您可以使用以下提示：`After each step, take a screenshot and carefully evaluate if you have achieved the right outcome. State in one sentence what the screenshot shows and whether the step succeeded. If it didn't, try again. Only when you confirm a step was executed correctly should you move on to the next one.`
3. 某些 UI 元素（例如下拉菜单和滚动条）可能难以让 Claude 通过鼠标移动来操作。如果遇到这种情况，请尝试提示模型使用键盘快捷键。
4. 对于可重复的任务或 UI 交互，请在提示中包含成功结果的示例截图和工具调用。
5. 如果您需要模型登录，请在提示中将用户名和密码放在 XML 标签（例如 `<robot_credentials>`）内提供给模型。在需要登录的应用程序中使用计算机使用，会增加因提示注入而导致不良结果的风险。在向模型提供登录凭据之前，请查看[缓解越狱和提示注入](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)。
6. 构建用户轮次的 `content` 数组时，请将指令文本放在截图图像*之前*。在处理图像之前提供目标描述可以提高点击准确性。
7. 当被问及在截图默认分辨率下无法辨认的小字或特定 UI 元素（例如侧边栏中的文件名、标签页标题、状态栏文本、行号或按钮标签）时，Claude 会使用 `zoom` 操作以全分辨率检查某个区域。如果 Claude 没有在您预期的时候进行缩放，请询问某个特定区域或元素，而不是整个屏幕。
8. 如果您希望每个[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)都以截图结束，请在系统提示中说明，例如：`End each group of actions with a screenshot so you can verify the result before continuing.`

<Tip>
  如果您反复遇到一组明确的问题，或者事先知道 Claude 需要完成的任务，请使用系统提示为 Claude 提供 关于如何成功完成任务的明确提示或指令。
</Tip>

<Tip>
  对于跨越多个会话的智能体，请在每个会话开始时运行端到端验证， 而不仅仅是在实现之后。基于浏览器的检查可以 发现仅靠代码层面审查会遗漏的、来自先前会话的回归问题。详情请参阅 [适用于长时间运行智能体的有效框架](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)。
</Tip>

### 系统提示

当您在请求中包含计算机使用工具时，API 会生成一个特定于计算机使用的系统提示。它与[工具使用系统提示](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools#tool-use-system-prompt)类似，但以以下内容开头：

> You have access to a set of functions you can use to answer the user’s question. This includes access to a sandboxed computing environment. You do NOT currently have the ability to inspect files or interact with external resources, except by invoking the below functions.

与常规工具使用一样，用户提供的 `system` 参数仍会被遵循，并用于构建组合后的系统提示。

### 可用操作

每个操作都是计算机使用工具集的一个成员工具：Claude 在带有 `"toolset_name": "computer"` 的 `tool_use` 块中指定成员名称，而该块的 `input` 仅包含该成员的参数，没有 `action` 字段。该工具集有 17 个成员工具：

| 成员                                                         | 输入                                                                                                                              | 描述                                                                                      |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `screenshot`                                               | 无（`{}`）                                                                                                                         | 捕获整个显示屏并将其作为图像返回。                                                                       |
| `zoom`                                                     | `region`：`[x0, y0, x1, y1]`，要检查区域的左上角和右下角                                                                                       | 仅以全分辨率捕获显示屏的该区域并将其作为图像返回，在保持宽高比的情况下缩放以适应您通常的截图尺寸。这使 Claude 能够读取在缩小后的完整截图中无法辨认的小字或密集 UI。 |
| `left_click`                                               | `coordinate`（可选）：`[x, y]`；`text`（可选）：点击期间按住的修饰键：`shift`、`ctrl`、`alt`、`super`（Command 键或 Windows 键），或以 `+` 连接的组合，例如 `ctrl+shift` | 在 `coordinate` 处点击鼠标左键；省略 `coordinate` 时在当前光标位置点击。                                      |
| `right_click`、`middle_click`、`double_click`、`triple_click` | 与 `left_click` 相同                                                                                                               | 其他鼠标按键和多次点击。                                                                            |
| `left_click_drag`                                          | `start_coordinate`：`[x, y]`；`coordinate`：`[x, y]`；`text`（可选）：修饰键                                                                | 在 `start_coordinate` 处按下，拖动到 `coordinate`，然后释放。                                         |
| `mouse_move`                                               | `coordinate`：`[x, y]`                                                                                                           | 移动光标而不点击，例如用于悬停。                                                                        |
| `left_mouse_down`、`left_mouse_up`                          | 无（`{}`）                                                                                                                         | 在当前光标位置按下或释放鼠标左键，用于 `left_click_drag` 无法表达的拖动操作。请先使用 `mouse_move` 移动光标。                 |
| `cursor_position`                                          | 无（`{}`）                                                                                                                         | 以文本形式报告光标当前的 `[x, y]` 位置。                                                               |
| `scroll`                                                   | `scroll_direction`：`"up"`、`"down"`、`"left"` 或 `"right"`；`scroll_amount`：滚轮滚动格数；`coordinate`（可选）：`[x, y]`；`text`（可选）：修饰键         | 在 `coordinate` 处或当前光标位置滚动。                                                              |
| `type`                                                     | `text`：要输入的字符串                                                                                                                  | 在当前键盘焦点处输入字面文本。                                                                         |
| `key`                                                      | `text`：一个键或以 `+` 连接的组合，例如 `"Return"`、`"ctrl+s"` 或 `"alt+Tab"`；`repeat`（可选）：1 到 100，默认为 1                                        | 按下一个键或组合键，共 `repeat` 次。                                                                 |
| `hold_key`                                                 | `text`：一个键或组合；`duration`：秒数，最多 300                                                                                              | 在给定的持续时间内按住某个键。                                                                         |
| `wait`                                                     | `duration`：秒数，最多 300                                                                                                            | 在下一个操作之前暂停，例如在应用程序加载时。                                                                  |

实现这些成员时，请注意以下几点：

* **坐标以截图像素为单位。** 每个 `coordinate`、`start_coordinate` 和 `region` 值，以及 `cursor_position` 报告的位置，都位于您返回的完整显示屏截图的像素空间中，原点位于左上角。缩放图像不会改变这一点：在 `zoom` 之后，Claude 仍然以完整截图的空间表示坐标，而不会相对于缩放后的图像。如果您在返回截图之前将其缩小，请在将 Claude 的坐标应用到真实显示屏之前将其按比例放大（请参阅[调整截图尺寸以符合图像限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#handle-coordinate-scaling-for-higher-resolutions)）。
* **默认启用所有成员，包括 `zoom`。** 如果您的环境无法生成缩放图像，请使用 `configs` 禁用该成员（请参阅[工具参数](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#tool-parameters)），而不是保持启用并返回错误。如果 Claude 调用了您已禁用或未实现的成员，请为该块返回一个带有 `is_error: true` 的 `tool_result`。
* **根据（`toolset_name`、`name`）组合进行分派。** `toolset_name` 是将一个块标记为计算机操作的依据：同一请求中的自定义工具可能与某个成员同名，而较新的工具集版本可能会添加成员（请参阅[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)）。

<Accordion title="操作示例">
  每个示例都是一个完整的 `tool_use` 块，与其在 Claude 响应中的形式相同。

  在某个位置进行 Shift+点击，例如用于扩展选区。与 `hold_key` 不同，`text` 仅在该次点击或滚动期间按住修饰键：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_01Qg8m3XqC5aRy7tD2eS4jUg",
    "name": "left_click",
    "toolset_name": "computer",
    "input": { "coordinate": [500, 300], "text": "shift" }
  }
  ```

  从一个点拖动到另一个点：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_01Ed6j9VnA3yPw5rB8cQ2gSe",
    "name": "left_click_drag",
    "toolset_name": "computer",
    "input": {
      "start_coordinate": [200, 300],
      "coordinate": [600, 300]
    }
  }
  ```

  向下滚动三格滚轮：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_01Yc5h8UmZ2xNv4qA7bP9fRd",
    "name": "scroll",
    "toolset_name": "computer",
    "input": {
      "coordinate": [500, 400],
      "scroll_direction": "down",
      "scroll_amount": 3
    }
  }
  ```

  按四次 Tab 键：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_01Sb4g7TkY9wLu3pX6zM8eQc",
    "name": "key",
    "toolset_name": "computer",
    "input": { "text": "Tab", "repeat": 4 }
  }
  ```

  放大以全分辨率检查某个区域：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_01Kf7k2WpB4zQx6sC9dR3hTf",
    "name": "zoom",
    "toolset_name": "computer",
    "input": { "region": [100, 200, 400, 350] }
  }
  ```

  报告光标位置。请使用一个简短的文本结果回复此调用，给出以截图像素为单位的位置，例如 `X=512, Y=384`：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_01Ekh3vqB6yTs2mNc4Rw8pLd",
    "name": "cursor_position",
    "toolset_name": "computer",
    "input": {}
  }
  ```
</Accordion>

### 工具参数

`tools` 数组中的工具集条目接受四个参数；它们与浏览器使用工具集共享的规则列在[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)下。

| 参数                | 必需 | 描述                                                                                                                                                                                                                                                                                                   |
| ----------------- | -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`            | 是  | `computer_toolset_20260801`                                                                                                                                                                                                                                                                          |
| `configs`         | 否  | 以成员名称为键的每个成员的设置；每个成员接受 `enabled`（对全部 17 个成员默认为 `true`，包括 `zoom`）和 `defer_loading`（默认为 `false`，用于[工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#deferred-tool-loading)），省略的成员保持其默认值。                                                                        |
| `cache_control`   | 否  | 位于工具集定义处的[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)断点；仅限条目本身。批次中任何 `tool_use` 或 `tool_result` 块上的断点都会在该批次结束时生效；请参阅[工具使用与提示缓存](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching#cache-control-on-tool-definitions)。 |
| `allowed_callers` | 否  | 仅限 `["direct"]`。                                                                                                                                                                                                                                                                                     |

例如，以下条目为不实现 `zoom` 的环境禁用了该成员，并在工具集定义处设置了缓存断点：

```json
{
  "type": "computer_toolset_20260801",
  "configs": {
    "zoom": { "enabled": false }
  },
  "cache_control": { "type": "ephemeral" }
}
```

如果您的智能体循环每次往返只能运行一个操作，请在 `tool_choice` 中将 `disable_parallel_tool_use` 设置为 `true`；这样 Claude 每个轮次最多返回一个成员 `tool_use` 块（请参阅[禁用并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use#disable-parallel-tool-use)）。

该条目不接受以下来自较早工具版本的参数，包含其中任何一个参数的请求都会返回 `invalid_request_error`：

* `name`：成员名称由工具集版本固定。
* `display_width_px`、`display_height_px` 和 `display_number`：坐标始终位于您返回的截图的像素空间中。
* `enable_zoom`：缩放是一个您通过 `configs` 控制的成员工具。

该条目也不能与 `computer_20251124` 条目或另一个名为 `computer` 的工具在同一请求中声明。有关 `strict`、`input_examples`、`defer_loading` 的放置位置、`tool_choice`、流式传输以及调用方限制，请参阅[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)。

### 与思考结合使用

要将计算机使用与思考结合使用，请参阅[思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)。

<Tip>
  对于 `computer_20251124` 工具，在使用该工具的模型上进行的内部基准测试建议采用以下 `effort` 设置：

  * **Claude Opus 4.7：** 默认使用 `high`；对于高吞吐量或成本敏感的工作负载，使用 `low`。
  * **Claude Sonnet 4.6 和 Claude Opus 4.6：** 默认使用 `medium`（最佳的准确性与成本比）。避免使用 `max`，它会增加令牌成本，却不会提高 UI 任务的准确性。在这些模型上，`low` 使用的输出令牌比完全禁用思考*更少*（错误更少意味着重试更少），因此是成本敏感循环的一个有力选择。
</Tip>

### 使用其他工具增强计算机使用

要在计算机使用之外添加其他工具，请将它们包含在同一个 `tools` 数组中。[快速入门](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#quick-start)部分通过 [bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)和[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)展示了这种模式。您也可以用同样的方式添加您自己的[自定义工具定义](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)。

对于停留在网页内的任务，您还可以[在同一请求中声明浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#combine-with-other-tools)。这两个工具集独立工作，各自使用自己的坐标系。`toolset_name` 值用于区分对同名成员（例如 `screenshot` 或 `key`）的调用。

### 构建自定义计算机使用环境

[参考实现](https://github.com/anthropics/claude-quickstarts/tree/main/computer-use-demo)旨在帮助您开始使用计算机使用。它包含让 Claude 使用计算机所需的所有组件。不过，您也可以根据自己的需求构建自己的计算机使用环境。您需要：

* 一个适合与 Claude 一起进行计算机使用的虚拟化或容器化环境
* 计算机使用工具各项操作的实现
* 一个与 Claude API 交互并使用您的工具实现来运行 `tool_use` 结果的智能体循环
* 一个允许用户输入以启动智能体循环的 API 或 UI

### 实现计算机使用工具

计算机使用工具是作为无 schema 工具实现的。使用此工具时，您无需像使用其他工具那样提供输入 schema；该 schema 内置于 Claude 的模型中，无法修改。

<Steps>
  <Step title="设置您的计算环境">
    创建一个虚拟显示器，或连接到 Claude 与之交互的现有显示器。这通常涉及设置 Xvfb（X Virtual Framebuffer）或类似技术。
  </Step>

  <Step title="实现操作处理程序">
    创建函数来处理 Claude 可能请求的每种操作类型：

    <CodeGroup exclude="shell">
      ```python Python
      # Placeholder image data; a real executor captures the screen and returns the PNG bytes
      PLACEHOLDER_PNG = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="


      def capture_screenshot() -> list[ImageBlockParam]:
          # screenshot answers with an image block rather than text: return the result content list
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


      def click(coordinate=None):
          if coordinate is None:
              return "clicked at current cursor"
          x, y = coordinate
          return f"clicked at ({x}, {y})"


      def type_text(text):
          return f"typed: {text}"


      def handle_computer_action(name, tool_input):
          match name:
              case "screenshot":
                  return capture_screenshot()
              case "left_click":
                  # coordinate is optional; without it, click where the cursor already is
                  return click(tool_input.get("coordinate"))
              case "type":
                  return type_text(tool_input["text"])
          # Handle other actions as needed
          raise ValueError(f"Unknown or unimplemented member: {name}")
      ```

      ```typescript TypeScript
      // Placeholder image data; a real executor captures the screen as PNG bytes
      const PLACEHOLDER_PNG =
        "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==";

      function captureScreenshot(): Anthropic.ImageBlockParam[] {
        // screenshot answers with an image block rather than text
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

      function clickAt(x: number, y: number): string {
        return `clicked at (${x}, ${y})`;
      }

      function clickAtCursor(): string {
        return "clicked at the current cursor position";
      }

      function typeText(text: string): string {
        return `typed: ${text}`;
      }

      function handleComputerAction(
        action: string,
        input: unknown
      ): string | Anthropic.ImageBlockParam[] {
        const params: object = typeof input === "object" && input !== null ? input : {};
        switch (action) {
          case "screenshot":
            return captureScreenshot();
          case "left_click":
            // coordinate is optional on the toolset; without one, click at the cursor
            if ("coordinate" in params && Array.isArray(params.coordinate)) {
              const [x, y] = params.coordinate;
              return clickAt(x, y);
            }
            return clickAtCursor();
          case "type":
            if ("text" in params) {
              return typeText(String(params.text));
            }
            break;
        }
        // Handle other actions as needed
        throw new Error(`Unknown or unimplemented member: ${action}`);
      }
      ```

      ```csharp C#
      // Placeholder image data; a real executor captures the screen and returns the PNG bytes
      const string PlaceholderPng = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==";

      // screenshot answers with an image block rather than text: return the result content list
      List<Block> CaptureScreenshot() =>
          [
              new ImageBlockParam(
                  new Base64ImageSource { Data = PlaceholderPng, MediaType = MediaType.ImagePng }
              ),
          ];

      string ClickAt(int x, int y) => $"clicked at ({x}, {y})";

      string ClickAtCursor() => "clicked at the current cursor position";

      string TypeText(string text) => $"typed: {text}";

      ToolResultBlockParamContent HandleComputerAction(
          string action,
          IReadOnlyDictionary<string, JsonElement> input
      ) =>
          action switch
          {
              "screenshot" => CaptureScreenshot(),
              // coordinate is optional on click members; without it, click where the cursor is
              "left_click" when input.TryGetValue("coordinate", out var xy) => ClickAt(
                  xy[0].GetInt32(),
                  xy[1].GetInt32()
              ),
              "left_click" => ClickAtCursor(),
              "type" => TypeText(input["text"].GetString()!),
              // Handle other actions as needed
              _ => throw new NotSupportedException($"Unknown or unimplemented member: {action}"),
          };
      ```

      ```go Go
      // placeholderPNG stands in for a real capture: an executor returns the
      // screen as base64-encoded PNG data.
      const placeholderPNG = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="

      // captureScreenshot returns an image block rather than text.
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

      // textContent wraps text as tool_result content.
      func textContent(text string) []anthropic.ToolResultBlockParamContentUnion {
      	return []anthropic.ToolResultBlockParamContentUnion{
      		{OfText: &anthropic.TextBlockParam{Text: text}},
      	}
      }

      func clickAt(x, y int) string {
      	return fmt.Sprintf("clicked at (%d, %d)", x, y)
      }

      func clickAtCursor() string {
      	return "clicked at the current cursor position"
      }

      func typeText(text string) string {
      	return fmt.Sprintf("typed: %s", text)
      }

      func handleComputerAction(action string, params map[string]any) ([]anthropic.ToolResultBlockParamContentUnion, error) {
      	switch action {
      	case "screenshot":
      		return captureScreenshot(), nil
      	case "left_click":
      		// coordinate is optional; without it, click where the cursor already is
      		coord, ok := params["coordinate"].([]any)
      		if !ok {
      			return textContent(clickAtCursor()), nil
      		}
      		if len(coord) == 2 {
      			x, xok := coord[0].(float64)
      			y, yok := coord[1].(float64)
      			if xok && yok {
      				return textContent(clickAt(int(x), int(y))), nil
      			}
      		}
      	case "type":
      		if text, ok := params["text"].(string); ok {
      			return textContent(typeText(text)), nil
      		}
      	// Handle other actions as needed
      	default:
      		return nil, fmt.Errorf("unknown or unimplemented member: %s", action)
      	}
      	// Reached when a member's input is missing a field or a field has the wrong type
      	return nil, fmt.Errorf("invalid input for %s", action)
      }
      ```

      ```java Java
      /** Placeholder pixels; a real executor captures the screen and base64-encodes the PNG. */
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

      String clickAt(long x, long y) {
          return "clicked at (" + x + ", " + y + ")";
      }

      String clickAtCursor() {
          return "clicked at current cursor";
      }

      String typeText(String text) {
          return "typed: " + text;
      }

      /** Runs one computer toolset member; {@code action} is the tool_use block's name. */
      ToolResultBlockParam.Content handleComputerAction(String action, Map<String, JsonValue> input) {
          if (action.equals("screenshot")) {
              return captureScreenshot(); // the one member here that answers with an image block
          }
          String output = switch (action) {
              case "left_click" -> {
                  JsonValue coordinate = input.get("coordinate"); // optional on the toolset
                  if (coordinate == null) {
                      yield clickAtCursor();
                  }
                  List<JsonValue> point = (List<JsonValue>) coordinate.asArray().get();
                  long x = ((Number) point.get(0).asNumber().get()).longValue();
                  long y = ((Number) point.get(1).asNumber().get()).longValue();
                  yield clickAt(x, y);
              }
              case "type" -> typeText(input.get("text").asStringOrThrow());
              // Handle other actions as needed
              default -> throw new UnsupportedOperationException("Unknown or unimplemented member: " + action);
          };
          return ToolResultBlockParam.Content.ofString(output);
      }
      ```

      ```php PHP
      // Stand-in for real PNG bytes; a real executor captures the screen
      const PLACEHOLDER_PNG = 'iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==';

      function captureScreenshot(): array
      {
          // screenshot answers with an image block rather than text, so return the result content list
          $image = [
              'type' => 'image',
              'source' => ['type' => 'base64', 'media_type' => 'image/png', 'data' => PLACEHOLDER_PNG],
          ];

          return [$image];
      }

      function clickAt(?array $coordinate): string
      {
          // left_click may omit coordinate, in which case the click lands where the cursor already is
          if ($coordinate === null) {
              return 'clicked at current cursor';
          }
          [$x, $y] = $coordinate;

          return "clicked at ({$x}, {$y})";
      }

      function typeText(string $text): string
      {
          return "typed: {$text}";
      }

      function handleComputerAction(string $name, array $input): string|array
      {
          return match ($name) {
              'screenshot' => captureScreenshot(),
              'left_click' => clickAt($input['coordinate'] ?? null),
              'type' => typeText($input['text']),
              // Handle other actions as needed
              default => throw new \RuntimeException("Unknown or unimplemented member: {$name}"),
          };
      }
      ```

      ```ruby Ruby
      # Stand-in image data; a real executor captures the screen as a PNG.
      PLACEHOLDER_PNG = "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg=="

      # screenshot answers with an image block rather than text
      def capture_screenshot
        [
          {
            type: "image",
            source: { type: "base64", media_type: "image/png", data: PLACEHOLDER_PNG }
          }
        ]
      end

      def click(coordinate = nil)
        return "clicked at current cursor" if coordinate.nil?

        x, y = coordinate
        "clicked at (#{x}, #{y})"
      end

      def type_text(text)
        "typed: #{text}"
      end

      def handle_computer_action(name, input)
        case name
        when "screenshot"
          capture_screenshot
        when "left_click"
          # coordinate is optional; without it, click where the cursor already is
          click(input[:coordinate])
        when "type"
          type_text(input[:text])
        # Handle other actions as needed
        else
          raise ArgumentError, "Unknown or unimplemented member: #{name}"
        end
      end
      ```
    </CodeGroup>
  </Step>

  <Step title="处理 Claude 的工具调用">
    从 Claude 的响应中提取并运行工具调用：

    <CodeGroup exclude="shell">
      ```python Python
      NOT_EXECUTED = "Not executed: an earlier computer action in this turn failed."


      def process_tool_calls(response: Message) -> list[ToolResultBlockParam]:
          """
          Run the computer actions in Claude's response in order and answer each
          one. After the first failure the rest are skipped, because Claude planned
          them assuming the earlier actions succeeded.
          """
          tool_results: list[ToolResultBlockParam] = []
          failed = False
          for block in response.content:
              # Only the computer toolset is declared; route other tools here if you add them
              if block.type != "tool_use" or block.toolset_name != "computer":
                  continue
              result: ToolResultBlockParam = {
                  "type": "tool_result",
                  "tool_use_id": block.id,
                  "toolset_name": "computer",
              }
              if failed:
                  result["content"] = NOT_EXECUTED
                  result["is_error"] = True
              else:
                  try:
                      # A string, or a list of content blocks such as the screenshot image
                      result["content"] = handle_computer_action(block.name, block.input)
                  except Exception as err:
                      result["content"] = f"Error: {err}"
                      result["is_error"] = True
                      failed = True
              tool_results.append(result)
          return tool_results
      ```

      ```typescript TypeScript
      const HALT_TEXT = "Not executed: an earlier computer action in this turn failed.";

      function computerResult(
        toolUseId: string,
        content: string | Anthropic.ImageBlockParam[],
        isError?: boolean
      ): Anthropic.ToolResultBlockParam {
        return {
          type: "tool_result",
          tool_use_id: toolUseId,
          toolset_name: "computer",
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
          if (block.toolset_name !== "computer") {
            // This example declares only the computer toolset; route other tools
            // here if you add them.
            continue;
          }
          if (failed) {
            // A batch stops at its first failure; answer later actions without running them
            toolResults.push(computerResult(block.id, HALT_TEXT, true));
            continue;
          }
          try {
            // A string, or the image block list that screenshot returns
            const result = handleComputerAction(block.name, block.input);
            toolResults.push(computerResult(block.id, result));
          } catch (error) {
            failed = true;
            const message = error instanceof Error ? error.message : String(error);
            toolResults.push(computerResult(block.id, `Error: ${message}`, true));
          }
        }
        return toolResults;
      }
      ```

      ```csharp C#
      const string HaltText = "Not executed: an earlier computer action in this turn failed.";

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

              if (toolUse.ToolsetName != "computer")
              {
                  // This example declares only the computer toolset; route other tools
                  // here if you add them.
                  continue;
              }

              if (failed)
              {
                  // A batch stops at its first failure; answer later actions without running them
                  toolResults.Add(
                      new ToolResultBlockParam(toolUse.ID)
                      {
                          Content = HaltText,
                          IsError = true,
                          ToolsetName = "computer",
                      }
                  );
                  continue;
              }

              try
              {
                  // A string, or the image block list that screenshot returns
                  var result = HandleComputerAction(toolUse.Name, toolUse.Input);
                  toolResults.Add(
                      new ToolResultBlockParam(toolUse.ID) { Content = result, ToolsetName = "computer" }
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
                          ToolsetName = "computer",
                      }
                  );
              }
          }
          return toolResults;
      }
      ```

      ```go Go
      const notExecuted = "Not executed: an earlier computer action in this turn failed."

      // computerToolResult builds the result for one computer action. Unlike an
      // ordinary tool result, it must echo the toolset name.
      func computerToolResult(toolUseID string, content []anthropic.ToolResultBlockParamContentUnion, isError bool) anthropic.ContentBlockParamUnion {
      	result := anthropic.ToolResultBlockParam{
      		ToolUseID:   toolUseID,
      		ToolsetName: anthropic.String("computer"),
      		Content:     content,
      	}
      	if isError {
      		result.IsError = anthropic.Bool(true)
      	}
      	return anthropic.ContentBlockParamUnion{OfToolResult: &result}
      }

      // processToolCalls runs the computer actions in Claude's response in order and
      // builds one tool_result for each tool_use block. After the first failure it skips
      // the rest: Claude planned them assuming the earlier actions succeeded.
      func processToolCalls(response *anthropic.Message) []anthropic.ContentBlockParamUnion {
      	var toolResults []anthropic.ContentBlockParamUnion
      	failed := false
      	for _, block := range response.Content {
      		switch variant := block.AsAny().(type) {
      		case anthropic.ToolUseBlock:
      			// This example declares only the computer toolset; route other tools here if you add them.
      			if variant.ToolsetName != "computer" {
      				continue
      			}
      			if failed {
      				toolResults = append(toolResults, computerToolResult(variant.ID, textContent(notExecuted), true))
      				continue
      			}
      			var input map[string]any
      			var content []anthropic.ToolResultBlockParamContentUnion
      			err := json.Unmarshal(variant.Input, &input)
      			if err == nil {
      				// Text, or the image block that screenshot returns
      				content, err = handleComputerAction(variant.Name, input)
      			}
      			if err != nil {
      				failed = true
      				content = textContent("Error: " + err.Error())
      			}
      			toolResults = append(toolResults, computerToolResult(variant.ID, content, err != nil))
      		}
      	}
      	return toolResults
      }
      ```

      ```java Java
      /** The exact text the toolset contract prescribes for member calls skipped after a failure. */
      static final String HALT_TEXT = "Not executed: an earlier computer action in this turn failed.";

      /** Every result answering a computer toolset member echoes toolset_name. */
      ToolResultBlockParam.Builder computerResult(ToolUseBlock toolUse) {
          return ToolResultBlockParam.builder()
              .toolUseId(toolUse.id())
              .toolsetName("computer");
      }

      /**
       * Run the computer actions in Claude's response in order and build one
       * tool_result for each tool_use block. After the first failure, skip the rest:
       * Claude planned them assuming the earlier actions succeeded.
       */
      List<ContentBlockParam> processToolCalls(Message response) {
          List<ContentBlockParam> toolResults = new ArrayList<>();
          boolean failed = false;
          for (ContentBlock block : response.content()) {
              // This example declares only the computer toolset; route other tools here if you add them.
              if (!block.isToolUse() || !block.asToolUse().toolsetName().equals(Optional.of("computer"))) {
                  continue;
              }
              ToolUseBlock toolUse = block.asToolUse();
              ToolResultBlockParam result;
              if (failed) {
                  result = computerResult(toolUse)
                      .content(HALT_TEXT)
                      .isError(true)
                      .build();
              } else {
                  try {
                      Map<String, JsonValue> input =
                          (Map<String, JsonValue>) toolUse._input().asObject().get();
                      // A string, or the image block that screenshot returns
                      ToolResultBlockParam.Content output = handleComputerAction(toolUse.name(), input);
                      result = computerResult(toolUse)
                          .content(output)
                          .build();
                  } catch (RuntimeException e) {
                      failed = true;
                      result = computerResult(toolUse)
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
      const HALT_TEXT = 'Not executed: an earlier computer action in this turn failed.';

      function processToolCalls(Message $response): array
      {
          $toolResults = [];
          $failed = false;
          foreach ($response->content as $block) {
              // This example declares only the computer toolset; route other tools here if you add them.
              if (!($block instanceof \Anthropic\Messages\ToolUseBlock) || $block->toolsetName !== 'computer') {
                  continue;
              }
              $result = ['type' => 'tool_result', 'tool_use_id' => $block->id, 'toolset_name' => 'computer'];
              if ($failed) {
                  // A batch stops at its first failure; the remaining actions are answered without running
                  $toolResults[] = [...$result, 'content' => HALT_TEXT, 'is_error' => true];
                  continue;
              }
              try {
                  // A string, or the image block list that screenshot returns
                  $toolResults[] = [...$result, 'content' => handleComputerAction($block->name, $block->input)];
              } catch (\Throwable $e) {
                  $failed = true;
                  $toolResults[] = [...$result, 'content' => 'Error: ' . $e->getMessage(), 'is_error' => true];
              }
          }

          return $toolResults;
      }
      ```

      ```ruby Ruby
      NOT_EXECUTED = "Not executed: an earlier computer action in this turn failed."

      # Run the computer actions in Claude's response in order and build one
      # tool_result for each tool_use block. After the first failure, skip the rest:
      # Claude planned them assuming the earlier actions succeeded.
      def process_tool_calls(response)
        tool_results = []
        failed = false
        response.content.each do |block|
          # This example declares only the computer toolset; route other tools here
          # if you add them.
          next unless block.is_a?(Anthropic::Models::ToolUseBlock) && block.toolset_name == "computer"

          result = { type: "tool_result", tool_use_id: block.id, toolset_name: "computer" }
          if failed
            result.update(content: NOT_EXECUTED, is_error: true)
          else
            begin
              # A String, or the image content blocks that screenshot returns
              result[:content] = handle_computer_action(block.name, block.input)
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
  </Step>

  <Step title="实现智能体循环">
    将前两个步骤包装在一个循环中，该循环将结果发送回去并不断重复，直到 Claude 不再返回成员工具调用；[理解智能体循环](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#understanding-the-agentic-loop)展示了这个循环。
  </Step>
</Steps>

### 处理错误

将失败的操作以带有 `is_error: true` 和简短描述的 `tool_result` 报告给 Claude，并像其他任何成员结果一样包含 `"toolset_name": "computer"`。如果失败的操作是某个[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)的一部分，请使用该处所示的中止文本回复批次中剩余的块，而不是运行它们。

例如，当截图捕获失败时：

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
      "toolset_name": "computer",
      "content": "Error: Failed to capture screenshot. Display may be locked or unavailable.",
      "is_error": true
    }
  ]
}
```

对于超出显示屏边界的坐标以及无法运行的操作，请使用相同的结构，并附上说明出错原因的消息。

### 调整截图尺寸以符合图像限制

您返回给计算机使用工具集的截图和缩放图像必须已经符合模型的[图像尺寸限制](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)：该工具集不接受显示尺寸参数，API 也不会为您缩小图像，因此超出尺寸的 `tool_result` 图像会因验证错误而被拒绝。由于 Claude 返回的坐标位于其接收到的图像的像素空间中，请保留您所使用的缩放因子，以便将这些坐标映射回您的屏幕。

<Note>
  限制因模型而异。Claude Opus 4.7 及更高版本的模型（包括所有支持 `computer_toolset_20260801` 的模型）接受长边最多 2,576 像素、总计最多 4,784 个视觉令牌（`⌈width / 28⌉ × ⌈height / 28⌉`，约 3.75 百万像素）；较早的模型接受长边最多 1,568 像素、总计约 1.15 百万像素（各模型所属的层级请参阅[分辨率和令牌成本](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)）。以下示例使用较早模型的 1,568 px / 1.15 MP 限制。对于高分辨率层级的模型，请按视觉令牌限制而非像素总数来调整尺寸，例如使用[上传前调整图像尺寸](https://platform.claude.com/docs/zh-CN/build-with-claude/vision-coordinates#resize-your-image-before-uploading)中的尺寸调整辅助函数。
</Note>

如果您的屏幕大于该限制，请在返回每张截图之前调整其尺寸，并将 Claude 返回的坐标缩放回原始屏幕空间。由于该工具集不接受显示尺寸参数，您只需在应用程序代码中完成尺寸调整和坐标缩放即可：

<CodeGroup exclude="shell">
  ```python Python
  import math

  screen_width, screen_height = 1512, 982


  def get_scale_factor(width, height):
      """Calculate scale factor to meet API constraints."""
      long_edge = max(width, height)
      total_pixels = width * height

      long_edge_scale = 1568 / long_edge
      total_pixels_scale = math.sqrt(1_150_000 / total_pixels)

      return min(1.0, long_edge_scale, total_pixels_scale)


  # 截取屏幕截图时
  scale = get_scale_factor(screen_width, screen_height)
  scaled_width = int(screen_width * scale)
  scaled_height = int(screen_height * scale)

  # 在发送给 Claude 之前，将图像缩放到缩放后的尺寸
  screenshot = capture_and_resize(scaled_width, scaled_height)


  # 处理 Claude 返回的坐标时，将其按比例放大回原始尺寸
  def execute_click(x, y):
      screen_x = x / scale
      screen_y = y / scale
      perform_click(screen_x, screen_y)
  ```

  ```typescript TypeScript
  const screenWidth = 1512;
  const screenHeight = 982;
  const MAX_LONG_EDGE = 1568;
  const MAX_PIXELS = 1_150_000;

  function getScaleFactor(width: number, height: number): number {
    const longEdge = Math.max(width, height);
    const totalPixels = width * height;

    const longEdgeScale = MAX_LONG_EDGE / longEdge;
    const totalPixelsScale = Math.sqrt(MAX_PIXELS / totalPixels);

    return Math.min(1.0, longEdgeScale, totalPixelsScale);
  }

  // 截取屏幕截图时
  const scale = getScaleFactor(screenWidth, screenHeight);
  const scaledWidth = Math.floor(screenWidth * scale);
  const scaledHeight = Math.floor(screenHeight * scale);

  // 在发送给 Claude 之前，将图像调整为缩放后的尺寸
  const screenshot = captureAndResize(scaledWidth, scaledHeight);

  // 处理 Claude 返回的坐标时，将其按比例放大还原
  function executeClick(x: number, y: number): void {
    const screenX = x / scale;
    const screenY = y / scale;
    performClick(screenX, screenY);
  }
  ```

  ```csharp C#
  int screenWidth = 1512, screenHeight = 982;

  double GetScaleFactor(int width, int height)
  {
      // 计算缩放因子以满足 API 约束。
      int longEdge = Math.Max(width, height);
      int totalPixels = width * height;

      double longEdgeScale = 1568.0 / longEdge;
      double totalPixelsScale = Math.Sqrt(1_150_000.0 / totalPixels);

      return Math.Min(1.0, Math.Min(longEdgeScale, totalPixelsScale));
  }

  // 截取屏幕截图时
  double scale = GetScaleFactor(screenWidth, screenHeight);
  int scaledWidth = (int)(screenWidth * scale);
  int scaledHeight = (int)(screenHeight * scale);

  // 在发送给 Claude 之前，将图像调整为缩放后的尺寸
  var screenshot = CaptureAndResize(scaledWidth, scaledHeight);

  // 处理 Claude 返回的坐标时，将其按比例放大还原
  void ExecuteClick(int x, int y)
  {
      double screenX = x / scale;
      double screenY = y / scale;
      PerformClick(screenX, screenY);
  }
  ```

  ```go Go
  func getScaleFactor(width, height int) float64 {
  	longest := float64(max(width, height))
  	area := float64(width * height)
  	return min(1.0, 1568/longest, math.Sqrt(1_150_000/area))
  }

  // ...
  	screenWidth, screenHeight := 1512, 982

  	// 截取屏幕截图时
  	scale := getScaleFactor(screenWidth, screenHeight)
  	scaledWidth := int(float64(screenWidth) * scale)
  	scaledHeight := int(float64(screenHeight) * scale)

  	// 在发送给 Claude 之前将图像调整为缩放后的尺寸
  	screenshot := captureAndResize(scaledWidth, scaledHeight)

  	// 处理 Claude 返回的坐标时，将其按比例放大还原
  	executeClick := func(x, y int) {
  		performClick(float64(x)/scale, float64(y)/scale)
  	}
  ```

  ```java Java
  static double getScaleFactor(int width, int height) {
      return Math.min(
          1.0,
          Math.min(
              1568.0 / Math.max(width, height),
              Math.sqrt(1_150_000.0 / (width * height))
          )
      );
  }

  void main() {
      int screenWidth = 1512, screenHeight = 982;

      // 截取屏幕截图时
      double scale = getScaleFactor(screenWidth, screenHeight);
      int scaledWidth = (int)(screenWidth * scale);
      int scaledHeight = (int)(screenHeight * scale);

      // 在发送给 Claude 之前，将图像调整为缩放后的尺寸
      var screenshot = captureAndResize(scaledWidth, scaledHeight);

      // 处理 Claude 返回的坐标时，将其按比例放大还原
      BiConsumer<Integer, Integer> executeClick =
          (x, y) -> performClick(x / scale, y / scale);
  // ...
  }
  ```

  ```php PHP
  function getScaleFactor(int $width, int $height): float
  {
      return min(
          1.0,
          1568 / max($width, $height),
          sqrt(1_150_000 / ($width * $height)),
      );
  }

  $screenWidth = 1512;
  $screenHeight = 982;

  // 截取屏幕截图时
  $scale = getScaleFactor($screenWidth, $screenHeight);
  $scaledWidth = (int)($screenWidth * $scale);
  $scaledHeight = (int)($screenHeight * $scale);

  // 在发送给 Claude 之前，将图像调整为缩放后的尺寸
  $screenshot = captureAndResize($scaledWidth, $scaledHeight);

  // 处理 Claude 返回的坐标时，将其按比例放大回原始尺寸
  $executeClick = fn(int $x, int $y) => performClick($x / $scale, $y / $scale);
  ```

  ```ruby Ruby
  def get_scale_factor(width, height)
    [1.0, 1568.0 / [width, height].max, Math.sqrt(1_150_000.0 / (width * height))].min
  end

  screen_width, screen_height = 1512, 982

  # 截取屏幕截图时
  scale = get_scale_factor(screen_width, screen_height)
  scaled_width = (screen_width * scale).to_i
  scaled_height = (screen_height * scale).to_i

  # 在发送给 Claude 之前，将图像调整为缩放后的尺寸
  screenshot = capture_and_resize(scaled_width, scaled_height)

  # 处理 Claude 返回的坐标时，将其按比例放大回原始尺寸
  execute_click = ->(x, y) { perform_click(x / scale, y / scale) }
  ```
</CodeGroup>

<Note>
  **macOS Retina 显示屏**以 2 的设备像素比捕获截图，因此图像分辨率是逻辑屏幕坐标的两倍。您可以在发送前将截图缩小 2 倍，也可以在执行点击前将 Claude 返回的坐标减半。
</Note>

当您选择显示分辨率并返回截图时：

* 对于一般桌面任务，请使用 1024x768 或 1280x720；对于 Web 应用程序，请使用 1280x800 或 1366x768。
* 避免使用高于 1920x1080 的分辨率，以防止出现性能问题。
* 将截图编码为 base64 PNG 或 JPEG，并考虑压缩较大的截图以提高性能。
* 包含相关元数据，例如时间戳或显示状态。
* 如果您使用更高的分辨率，请确保坐标已准确缩放。

### 管理截图历史

长时间运行的智能体循环会迅速累积截图（每张大约 1,000–1,800 个输入令牌）。API 的[请求限制](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#request-limits)同样适用。一旦单个请求携带超过 20 张图像，其中的每张图像都将受到更严格的单边尺寸限制。保留截图历史的循环会在几十轮内达到该数量，因此请调整每张截图的尺寸以保持在该限制之内，或者修剪较早的截图，使请求中保留 20 张或更少。

要在限制上下文的同时保持"prompt caching"（[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)）有效：

* 在系统提示和工具定义之后放置一个 `cache_control` 断点，并在最近几轮中每一轮的最后一个 `tool_result` 块上最多再放置三个，每轮向前推进这些断点。在一次[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)中，多个块上的标记会作为单个断点生效，但每个标记仍计入四个断点的限制，因此任何一轮中最多使用一个。

* 请*分批*修剪旧截图，而不是每轮修剪一张。每轮丢弃一张截图会使前缀每轮都发生变化，从而使缓存失效。一个合理的默认做法是保留最后三张截图，并每 25 轮修剪一次，这样在两次修剪之间前缀保持字节级一致；如果您的截图超出了更严格的限制，请选择一个能使每个请求保持 20 张或更少图像的间隔。

* 在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1 和 Claude Mythos 5.1 上，请避免在客户端进行修剪。Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 也是如此。在这些模型上，从请求中移除较早的屏幕截图会[使该请求以及之后每个请求中所有后续的思考块失效](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-in-conversation)。

  * **替代做法：** 调整屏幕截图大小以保持在更严格的限制之内。使用服务器端的[工具结果清除](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#tool-result-clearing)功能从上下文中删除旧的屏幕截图。
  * **如果您必须进行修剪：** 请从此以后一直设置 [`prefix_mismatch_behavior: "drop_block"`](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-thinking-controls)。每次修剪后，API 会在该请求及之后的每个请求中丢弃已失效的思考块。因此，Claude 会在不包含其在被修剪的屏幕截图之后所产生的思考的情况下继续。
  * **如果您必须在 Claude Sonnet 5.5 或 Claude Haiku 5.5 上进行修剪：** `prefix_mismatch_behavior` 仅适用于 `thinking: {"type": "adaptive"}`。在这些模型上使用任何其他思考设置时，请保持历史记录仅追加，或者删除被修剪的屏幕截图之后的每个 `thinking` 和 `redacted_thinking` 块。

### 诊断点击问题

如果点击未命中目标，原因通常是以下之一：

| 症状                | 可能原因                                            | 尝试                                                                                                                                                                                                                     |
| ----------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 点击始终向某个方向偏移       | Claude 的坐标（位于您返回的截图的像素空间中）被应用到了尺寸不同的显示屏上，且未进行缩放 | 在点击之前，将每个坐标按屏幕尺寸与截图尺寸之比进行缩放（请参阅[调整截图尺寸以符合图像限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#handle-coordinate-scaling-for-higher-resolutions)）；在 macOS Retina 显示屏上，请考虑 2 倍的设备像素比 |
| 点击落在正确区域但未命中目标    | 目标非常小、从 4K 以上的源缩小时丢失了细节，或者宽高比发生了失真              | 保持 `zoom` 成员启用并实现它，以便 Claude 可以以全分辨率检查该区域；以较低 DPI 捕获或裁剪到相关区域；调整尺寸时保持宽高比                                                                                                                                                |
| Claude 点击了完全错误的元素 | 指令含糊，或附近存在视觉上相似的元素                              | 使用位置性提示（"右下角的蓝色 Submit 按钮"）；将交互拆分为更小的步骤                                                                                                                                                                                |
| 准确率始终很差           | 分辨率过低                                           | 尝试以 1280x720 作为基准                                                                                                                                                                                                      |

<Tip>
  **模型选择会影响点击精度。** 在使用 `computer_20251124` 工具的模型中，Claude Sonnet 4.6 的点击机械精度高于 Claude Opus 4.6，并且在截图需要大幅缩小时更加稳健。Claude Opus 4.7 缩小了这一差距：其点击精度与 Sonnet 4.6 大致相当，而且其更高的分辨率限制意味着需要的缩小更少。
</Tip>

### 遵循实现最佳实践

<AccordionGroup>
  <Accordion title="添加操作延迟">
    某些应用程序需要时间来响应操作：

    <CodeGroup exclude="shell">
      ```python Python
      def click_and_wait(x, y, wait_time=0.5):
          click_at(x, y)
          time.sleep(wait_time)  # Allow UI to update
      ```

      ```typescript TypeScript
      async function clickAndWait(x: number, y: number, waitMs = 500): Promise<void> {
        clickAt(x, y);
        await new Promise((resolve) => setTimeout(resolve, waitMs)); // Allow UI to update
      }
      ```

      ```csharp C#
      static void ClickAndWait(int x, int y, double waitSeconds = 0.5)
      {
          ClickAt(x, y);
          Thread.Sleep(TimeSpan.FromSeconds(waitSeconds));  // Allow UI to update
      }
      ```

      ```go Go
      func clickAndWaitFor(x, y int, wait time.Duration) {
      	clickAt(x, y)
      	time.Sleep(wait) // Allow UI to update
      }

      func clickAndWait(x, y int) {
      	clickAndWaitFor(x, y, 500*time.Millisecond)
      }
      ```

      ```java Java
      void clickAndWait(int x, int y) throws InterruptedException {
          clickAndWait(x, y, 500);
      }

      void clickAndWait(int x, int y, long waitTimeMillis) throws InterruptedException {
          clickAt(x, y);
          Thread.sleep(waitTimeMillis);  // Allow UI to update
      }
      ```

      ```php PHP
      function clickAndWait(int $x, int $y, float $waitSeconds = 0.5): void
      {
          clickAt($x, $y);
          usleep((int) ($waitSeconds * 1_000_000));  // Allow UI to update
      }
      ```

      ```ruby Ruby
      def click_and_wait(x, y, wait_time: 0.5)
        click_at(x, y)
        sleep(wait_time) # Allow UI to update
      end
      ```
    </CodeGroup>
  </Accordion>

  <Accordion title="在运行操作前进行验证">
    检查所请求的操作是否安全有效：

    <CodeGroup exclude="shell">
      ```python Python
      display_width, display_height = 1024, 768


      def validate_action(action_type, params):
          if action_type == "left_click" and "coordinate" in params:
              x, y = params["coordinate"]
              if not (0 <= x < display_width and 0 <= y < display_height):
                  return False, "Coordinates out of bounds"
          return True, None
      ```

      ```typescript TypeScript
      const displayWidth = 1024;
      const displayHeight = 768;

      interface ActionParams {
        coordinate?: [number, number];
      }

      function validateAction(actionType: string, params: ActionParams): [boolean, string | null] {
        if (actionType === "left_click" && params.coordinate) {
          const [x, y] = params.coordinate;
          if (!(x >= 0 && x < displayWidth && y >= 0 && y < displayHeight)) {
            return [false, "Coordinates out of bounds"];
          }
        }
        return [true, null];
      }
      ```

      ```csharp C#
      const int DisplayWidth = 1024;
      const int DisplayHeight = 768;
      // ...
      static (bool IsValid, string? Error) ValidateAction(string actionType, IReadOnlyDictionary<string, JsonElement> parameters)
      {
          if (actionType == "left_click" && parameters.TryGetValue("coordinate", out JsonElement coordinate))
          {
              int x = coordinate[0].GetInt32();
              int y = coordinate[1].GetInt32();
              if (x is < 0 or >= DisplayWidth || y is < 0 or >= DisplayHeight)
              {
                  return (false, "Coordinates out of bounds");
              }
          }
          return (true, null);
      }
      ```

      ```go Go
      const (
      	displayWidth  = 1024
      	displayHeight = 768
      )

      func validateAction(actionType string, params map[string]any) (bool, string) {
      	raw, hasCoordinate := params["coordinate"]
      	if actionType == "left_click" && hasCoordinate {
      		coord, ok := raw.([]any)
      		if !ok || len(coord) != 2 {
      			return false, "Invalid coordinate"
      		}
      		x, y := int(coord[0].(float64)), int(coord[1].(float64))
      		if !(0 <= x && x < displayWidth && 0 <= y && y < displayHeight) {
      			return false, "Coordinates out of bounds"
      		}
      	}
      	return true, ""
      }
      ```

      ```java Java
      static final int DISPLAY_WIDTH = 1024;
      static final int DISPLAY_HEIGHT = 768;

      record Validation(boolean valid, String error) {}

      Validation validateAction(String actionType, Map<String, JsonValue> params) {
          if (actionType.equals("left_click") && params.containsKey("coordinate")) {
              List<JsonValue> coord = (List<JsonValue>) params.get("coordinate").asArray().get();
              long x = ((Number) coord.get(0).asNumber().get()).longValue();
              long y = ((Number) coord.get(1).asNumber().get()).longValue();
              if (!(0 <= x && x < DISPLAY_WIDTH && 0 <= y && y < DISPLAY_HEIGHT)) {
                  return new Validation(false, "Coordinates out of bounds");
              }
          }
          return new Validation(true, null);
      }
      ```

      ```php PHP
      const DISPLAY_WIDTH = 1024;
      const DISPLAY_HEIGHT = 768;

      /** @return array{bool, ?string} */
      function validateAction(string $actionType, array $params): array
      {
          if ($actionType === 'left_click' && isset($params['coordinate'])) {
              [$x, $y] = $params['coordinate'];
              if (!(0 <= $x && $x < DISPLAY_WIDTH && 0 <= $y && $y < DISPLAY_HEIGHT)) {
                  return [false, 'Coordinates out of bounds'];
              }
          }
          return [true, null];
      }
      ```

      ```ruby Ruby
      DISPLAY_WIDTH = 1024
      DISPLAY_HEIGHT = 768

      def validate_action(action_type, params)
        if action_type == "left_click" && params.key?(:coordinate)
          x, y = params[:coordinate]
          unless (0...DISPLAY_WIDTH).cover?(x) && (0...DISPLAY_HEIGHT).cover?(y)
            return [false, "Coordinates out of bounds"]
          end
        end
        [true, nil]
      end
      ```
    </CodeGroup>
  </Accordion>

  <Accordion title="记录操作以便调试">
    记录所有操作的日志以便排查问题：

    <CodeGroup exclude="shell">
      ```python Python
      import logging


      def log_action(action_type, params, result):
          logging.info(f"Action: {action_type}, Params: {params}, Result: {result}")
      ```

      ```typescript TypeScript
      function logAction(actionType: string, params: unknown, result: unknown): void {
        console.error(
          `Action: ${actionType}, Params: ${JSON.stringify(params)}, Result: ${JSON.stringify(
            result
          )}`
        );
      }
      ```

      ```csharp C#
      static void LogAction(string actionType, object? parameters, object? result)
      {
          Console.Error.WriteLine($"Action: {actionType}, Params: {parameters}, Result: {result}");
      }
      ```

      ```go Go
      func logAction(actionType string, params map[string]any, result any) {
      	log.Printf("Action: %s, Params: %v, Result: %v", actionType, params, result)
      }
      ```

      ```java Java
      import static java.lang.System.Logger.Level.INFO;

      static final System.Logger LOGGER = System.getLogger("computer-use");

      void logAction(String actionType, Object params, Object result) {
          LOGGER.log(INFO, "Action: {0}, Params: {1}, Result: {2}", actionType, params, result);
      }
      ```

      ```php PHP
      function logAction(string $actionType, array $params, mixed $result): void
      {
          error_log(sprintf(
              'Action: %s, Params: %s, Result: %s',
              $actionType,
              json_encode($params),
              json_encode($result),
          ));
      }
      ```

      ```ruby Ruby
      require "logger"

      LOGGER = Logger.new($stderr)

      def log_action(action_type, params, result)
        LOGGER.info("Action: #{action_type}, Params: #{params}, Result: #{result}")
      end
      ```
    </CodeGroup>
  </Accordion>
</AccordionGroup>

***

## 从 `computer_20251124` 迁移

`computer_20251124` 集成无需任何更改即可在[较早的工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#earlier-tool-versions)中为该版本列出的模型上运行。在将该集成迁移到不接受 `computer_20251124` 的模型（例如 Claude API 上的 Claude 5.5 模型）之前，请先升级该集成。

要进行升级，请同时进行以下更改：

1. **删除 beta 标头。** 从您的请求中去掉 `anthropic-beta: computer-use-2025-11-24`。使用 SDK 时，删除 `betas` (csharp, go: `Betas`; java: `.addBeta()`)，并通过标准客户端而非 beta 命名空间调用 Messages API。
2. **更改 `tools` 条目。** 将 `type` 设置为 `computer_toolset_20260801`，并删除 `name`、`display_width_px`、`display_height_px`、`display_number` 和 `enable_zoom`。工具集会拒绝这些字段中的每一个。
3. **选择是否保持缩放启用。** 工具集默认启用缩放，而 `enable_zoom` 默认为 `false`。如果您的环境未实现缩放，请添加 `"configs": {"zoom": {"enabled": false}}` 以保持之前的行为；否则请实现它（请参阅[可用操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#available-actions)）。
4. **处理一轮中的每个块。** 更新您的智能体循环，使其遍历响应中的每个 `tool_use` 块，而不是只读取第一个，并根据块的 `name` 和 `toolset_name` 进行分派，而不是根据 `input.action`。成员输入不再包含 `action` 字段；其余字段保持不变。
5. **按顺序运行块并使用中止文本。** 按顺序运行这些块，在第一次失败时停止，并使用 `Not executed: an earlier computer action in this turn failed.` 响应其余的块，如[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)中所述。如果您的循环尚不能运行批量操作，[工具参数](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#tool-parameters)说明了如何将 Claude 限制为每轮一个操作。
6. **在结果中回传 `toolset_name`。** 在每个响应成员调用的 `tool_result` 中添加 `"toolset_name": "computer"`。结果只能包含 `text` 和 `image` 内容。
7. **在 `key` 上支持 `repeat`。** `key` 成员接受一个可选的 `repeat` 计数，范围为 1 到 100。忽略无法识别字段的处理程序只会按一次键，因此请让您的 `key` 处理程序遵循 `repeat`。
8. **自行调整截图尺寸。** 工具集会拒绝超出模型图像限制的截图或缩放图像，而不是将其缩小。请在返回图像之前调整其尺寸，并按照[调整截图尺寸以符合图像限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#handle-coordinate-scaling-for-higher-resolutions)中的说明继续缩放坐标。
9. **删除不支持的选项。** 将任何 `defer_loading` 从条目移到 `configs` 中，并在每个已启用的成员上使用相同的值。工具集条目不支持的其他选项列于[客户端工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference#client-toolsets)下。

这是更改前的 `tools` 条目，随 `anthropic-beta: computer-use-2025-11-24` 标头一起发送：

```json
{
  "type": "computer_20251124",
  "name": "computer",
  "display_width_px": 1024,
  "display_height_px": 768,
  "display_number": 1
}
```

这是更改后的 `tools` 条目，发送时不带 beta 标头。`configs` 对象保持缩放关闭，以与未设置 `enable_zoom` 的旧条目一致；若要接受默认值并让 Claude 进行缩放，请完全省略 `configs`：

```json
{
  "type": "computer_toolset_20260801",
  "configs": {
    "zoom": { "enabled": false }
  }
}
```

以下这对示例展示了更改前后的 `tool_use` 块。操作名称从 `input.action` 移到 `name`，并且块中新增了 `toolset_name`：

```json
{
  "type": "tool_use",
  "id": "toolu_01A9r5kQm2LxWc7vT3nZ4bJs",
  "name": "computer",
  "input": { "action": "left_click", "coordinate": [500, 300] }
}
```

```json
{
  "type": "tool_use",
  "id": "toolu_01A9r5kQm2LxWc7vT3nZ4bJs",
  "name": "left_click",
  "toolset_name": "computer",
  "input": { "coordinate": [500, 300] }
}
```

## 较早的工具版本

“Computer use”（计算机使用）工具有两个较早的版本以 beta 形式提供。每个版本都需要在每个请求中包含其 [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers)，其参数记录在 [beta Messages API 参考](https://platform.claude.com/docs/zh-CN/api/beta/messages/create)中。使用 SDK 时，请通过 `betas` (csharp, go: `Betas`; java: `.addBeta()`) 传递该标头并使用 beta 命名空间；只有计算机使用工具需要该标头，同一请求中的 bash 或文本编辑器工具则不需要。

| 工具版本                | Beta 标头                   | 搭配使用的模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | 参数                                                                        |
| ------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `computer_20251124` | `computer-use-2025-11-24` | Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Opus 5、Claude Sonnet 5、Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 4.6 和 Claude Opus 4.5                                                                                                                                                                                                                                                                                                                                                                                                      | [API 参考](https://platform.claude.com/docs/zh-CN/api/beta/messages/create) |
| `computer_20250124` | `computer-use-2025-01-24` | Claude Sonnet 4.5（[已弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)）、Claude Haiku 4.5、Claude Opus 4.1（[已退役，Claude Models on Google Cloud (Google managed) 和 Bedrock 除外](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)）、Claude Sonnet 4（[已退役，Claude Models on Google Cloud (Google managed) 和 Bedrock 除外](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)）以及 Claude Opus 4（[已退役，Claude Models on Google Cloud (Google managed) 除外](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)） | [API 参考](https://platform.claude.com/docs/zh-CN/api/beta/messages/create) |

在 Amazon Bedrock 上，Claude Fable 5.2、Claude Mythos 5.2、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 也接受搭配 `computer-use-2025-11-24` beta 标头的 `computer_20251124` 工具。使用 Amazon Bedrock 的 InvokeModel API 时，请在请求正文的 `anthropic_beta` 数组中发送 `computer-use-2025-11-24`，而不是在标头中发送。有关该 API，请参阅 [Amazon Bedrock 上的 Claude（bedrock-runtime）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)。

***

## 限制

1. **延迟：** 与常规的人工指导计算机操作相比，用于人机交互的计算机使用延迟可能过慢。请在受信任的环境中专注于速度不关键的用例（例如后台信息收集、自动化软件测试）。
2. **计算机视觉的准确性和可靠性：** Claude 在生成操作时输出特定坐标可能会出错或产生幻觉。Claude 的[摘要思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#summarized-thinking)输出可以帮助您理解模型的推理过程并识别潜在问题；请在思考配置中设置 `display: "summarized"`，因为支持工具集的模型默认会省略思考文本。
3. **工具选择的准确性和可靠性：** Claude 在生成操作时选择工具可能会出错或产生幻觉，或者采取意外的操作来解决问题。此外，在与小众应用程序交互或同时与多个应用程序交互时，可靠性可能会较低。在请求复杂任务时，请谨慎地向模型发出提示。
4. **滚动可靠性：** 滚动操作支持方向控制（上、下、左、右）和指定的滚动量。在滚动不生效的应用程序中，Page Down 等键盘替代方式会有所帮助。
5. **电子表格交互：** 使用细粒度的鼠标控制操作（`left_mouse_down`、`left_mouse_up`）和修饰键组合来选择单个单元格。复杂的电子表格操作可能仍需要多次尝试。
6. **在社交和通信平台上创建账户和生成内容：** 虽然 Claude 会访问网站，但其在社交媒体网站和平台上创建账户、生成和分享内容或以其他方式冒充人类的能力是有限的。
7. **漏洞：** 与任何前沿 AI 系统一样，越狱和提示注入可能会影响计算机使用，包括通过嵌入在网页或图像中的指令；请采取[安全注意事项](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#security-considerations)中的预防措施。
8. **不当或非法操作：** 根据 Anthropic 的服务条款，您不得利用计算机使用功能违反任何法律或可接受使用政策。

请始终仔细审查和验证 Claude 的计算机使用操作和日志。在没有人工监督的情况下，请勿将 Claude 用于需要完美精度或涉及敏感用户信息的任务。

## 数据保留

计算机使用是一种客户端工具。会话中涉及的所有截图、鼠标操作、键盘输入以及任何文件都在您的环境中捕获和存储，而不是由 Anthropic 存储。Anthropic 作为 API 调用的一部分实时处理截图图像和操作请求。这些 API 请求的保留受 [API 和数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)约束。

由于您的应用程序控制计算机使用数据的存储位置和方式，因此计算机使用符合 ZDR 资格。

## 定价

计算机使用遵循标准的[工具使用定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#pricing)。使用计算机使用工具时：

**工具集定义开销：** 声明带有默认成员的 `computer_toolset_20260801` 会为请求增加约 4,500 个输入令牌（在 Claude Sonnet 5 上约为 4,590 个，在其他受支持的模型上约为 4,520 个），其中涵盖成员工具定义和工具使用系统提示。使用 `configs` 禁用 `zoom` 可减少其中约 410 个令牌。请求的确切令牌数会在响应的 `usage` 中报告，您也可以使用[令牌计数端点](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)提前进行估算。

**早期工具版本：** 以下数据适用于 `computer_20251124` 和 `computer_20250124` 工具版本，不适用于 `computer_toolset_20260801`：

* 系统提示开销：向系统提示中添加 466–499 个令牌
* 工具定义：每个工具定义约 735 个输入令牌（使用 `computer_20250124` 测量）

**额外令牌消耗：**

* 工具结果中返回的屏幕截图和缩放图像，按图像输入计费（请参阅[视觉定价](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)）
* 返回给 Claude 的工具执行结果

<Note>
  如果您在计算机使用的同时还使用 bash 或文本编辑器工具，这些工具有各自的令牌成本，详见其各自的文档页面。
</Note>

## 后续步骤

<CardGroup cols={2}>
  <Card title="SDK 中的计算机工具集" icon="code" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#computer-toolset">
    从合作伙伴的驱动程序开始，或使用 Python 或 TypeScript 编写您自己的驱动程序。SDK 会运行循环以及您传入的审批回调。
  </Card>

  <Card title="工具使用故障排除" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/troubleshooting-tool-use">
    借助从症状到修复的诊断表，修复最常见的工具使用错误。
  </Card>

  <Card title="参考实现" icon="github-logo" href="https://github.com/anthropics/claude-quickstarts/tree/main/computer-use-demo">
    从完整的基于 Docker 的实现开始。
  </Card>

  <Card title="Claude 的工具使用" icon="tool" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview">
    将 Claude 连接到外部工具和 API。了解工具在何处运行、Claude 何时调用它们，以及哪种工具适合您的任务。
  </Card>

  <Card title="最佳实践详解" icon="book" href="https://claude.com/blog/best-practices-for-computer-and-browser-use-with-claude">
    关于分辨率、思考力度和上下文管理的基准测试建议。
  </Card>

  <Card title="浏览器使用工具" icon="browser" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool">
    让 Claude 在您自己的浏览器环境中导航、阅读网页并与之交互，适用于完全在浏览器内完成的任务。
  </Card>
</CardGroup>

## 可用性

计算机使用工具在部分模型上可用或处于 beta 阶段，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态   | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用   | Claude API, [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)                                                                                                                                                                                                                                                                                                                                                                                                                                                          | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview) |
| 可用   | [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)                                                                                                                                                                                                                                                                                                                                                                                                         | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                        |
| 可用   | [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                                                                                                                                                                                                      |
| 可用   | [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                                                                                                                                                                                                    |
| Beta | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/haiku-4-5/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview), [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview), [Claude Sonnet 4.5](https://platform.claude.com/docs/zh-CN/models/sonnet-4-5/overview)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

| 请求头                       | 模型                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `computer-use-2025-11-24` | [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview), [Claude Opus 4.5](https://platform.claude.com/docs/zh-CN/models/opus-4-5/overview) |
| `computer-use-2025-01-24` | [Claude Haiku 4.5](https://platform.claude.com/docs/zh-CN/models/haiku-4-5/overview), [Claude Sonnet 4.5](https://platform.claude.com/docs/zh-CN/models/sonnet-4-5/overview)                                                                                                                                                                       |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                              | HIPAA |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)）          | 具备资格  |
| 详细信息 | 客户端工具，截图和文件在您的环境中捕获和存储，而非由 Anthropic 存储。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#data-retention) | —     |

适用的端点：

* `/v1/messages` 使用 `computer` 工具集或工具
