---
title: 代码执行工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool
description: 在沙盒容器中运行 Python 和 Bash 代码，以分析数据、生成文件并迭代改进解决方案。
featureMetadata:
  status: stable
  zdr:
    eligibility: not-eligible
    note: 容器数据最多保留 30 天。
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
    Microsoft Foundry (hosted on Anthropic): stable
  details:
    - 每个受支持的模型都接受全部三个[工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#tool-versions)。在 Claude Haiku 4.5 上，程序化工具调用和 REPL 状态持久化不可用，因此较新的版本在该模型上的行为与 `code_execution_20250825` 相同。
---

Claude 可以直接在 API 对话中分析数据、创建可视化图表、执行复杂计算、运行系统命令、创建和编辑文件，以及处理上传的文件。代码执行工具允许 Claude 在安全的沙盒环境中运行 Bash 命令和操作文件，包括编写代码。

**与网络搜索或网页获取（`web_search_20260209`、`web_fetch_20260209` 或更高版本）一起使用时，代码执行免费。** 当您的请求中包含其中一个工具时，除标准令牌费用外，该请求中的代码执行不会产生额外费用。这既涵盖动态过滤背后的代码执行，也涵盖 Claude 直接运行的任何代码。未包含这些工具时，适用标准代码执行定价。

代码执行还为[网络搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)和[网页获取](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)工具中的动态过滤提供支持：Claude 会在结果进入上下文窗口之前，在代码执行环境中对其进行过滤。当动态过滤运行时，API 会自动为请求配置所需的代码执行，因此您无需为此在请求中添加代码执行工具。

<Note>
  请使用[反馈表单](https://forms.gle/LTAU6Xn2puCJMi1n6)分享您对此功能的反馈。
</Note>

## 工具版本

代码执行工具目前有三个版本，每个[受支持的模型](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#availability-feature-support)都接受全部三个版本。每个版本都在前一个版本的基础上构建：

* `code_execution_20250825` 支持 Bash 命令和文件操作。
* `code_execution_20260120` 增加了 REPL 状态持久化以及在沙盒内进行[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)的能力。Claude Haiku 4.5 接受 `code_execution_20260120` 和 `code_execution_20260521` 工具类型，但程序化工具调用以及依赖它的 REPL 状态持久化在该模型上不可用，因此较新的版本在该模型上的行为与 `code_execution_20250825` 相同。
* `code_execution_20260521` 与 `code_execution_20260120` 使用相同的运行时。区别在于其工具描述会告知 Claude 程序化工具调用中每个 Python 单元有 90 秒的挂钟时间限制，以便 Claude 为长时间运行的单元做好时间规划。超过该限制的单元会返回一个正常的代码执行结果，其中包含非零的 `return_code`，并在输出中包含 `detection_timeout` 状态消息。这与 `execution_time_exceeded` [错误代码](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#errors)不同，后者是在整个工具调用超过最大执行时间时由 API 返回的。

对于 [Claude Mythos Preview](https://anthropic.com/glasswing)，代码执行在 Claude API 和 Microsoft Foundry 上受支持。

这三个工具版本都不需要 `anthropic-beta` 标头。旧版代码执行 beta 标头仍然是有效的选择加入方式。

本页的示例使用 `code_execution_20250825`，它涵盖了示例中演示的 Bash 和文件操作，并且在每个受支持的模型上行为一致；当您需要程序化工具调用或 REPL 状态持久化时，请使用 `code_execution_20260120` 或更高版本。当前的[网络搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)和[网页获取](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)工具（`web_search_20260209`、`web_fetch_20260209` 及更高版本）要求其代码执行版本为 `code_execution_20260120` 或更高版本。

旧版工具不保证与较新的模型保持兼容。在采用新模型时，请查看[工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#tool-versions)和 [API 支持](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#availability-feature-support)，并优先使用您的集成所支持的最新工具版本。

要在发送请求之前检查某个模型是否接受代码执行工具，请从 Models API 中读取其 `capabilities.server_tools.code_execution.supported` 值。不要为此使用顶层的 `capabilities.code_execution` 值。该值表示在此工具中运行的代码是否可以调用您请求中的其他工具，如程序化工具调用中那样。[使用 Models API](https://platform.claude.com/docs/zh-CN/models/overview#using-the-models-api) 介绍了这两个字段。

<Note>
  如果您正在使用 `code_execution_20250522`（仅支持 Python），请参阅[升级到最新工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#upgrade-to-latest-tool-version)以从中迁移。
</Note>

## 快速入门

以下是一个要求 Claude 执行计算的示例：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [
        {
          "role": "user",
          "content": "Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"
        }
      ],
      "tools": [
        {
          "type": "code_execution_20250825",
          "name": "code_execution"
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --message '{
      role: user,
      content: "Use the code execution tool to calculate the mean and standard
        deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"
    }' \
    --tool '{type: code_execution_20250825, name: code_execution}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]",
          }
      ],
      tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
  )

  print(response.to_json())
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
          "Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"
      }
    ],
    tools: [{ type: "code_execution_20250825", name: "code_execution" }]
  });

  console.log(JSON.stringify(response));
  ```

  ```csharp C#
  AnthropicClient client = new();

  var message = await client.Messages.Create(new()
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [new UserMessageParam { Content = "Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]" }],
      Tools = [new CodeExecutionTool20250825()]
  });

  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfCodeExecutionTool20250825: &anthropic.CodeExecutionTool20250825Param{}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response.RawJSON())
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(4096L)
      .addUserMessage("Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]")
      .addTool(CodeExecutionTool20250825.builder().build())
      .build();

  Message response = client.messages().create(params);
  IO.println(ObjectMappers.jsonMapper().valueToTree(response));
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => 'Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]',
          ],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [new CodeExecutionTool20250825()],
  );

  echo json_encode($message, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Use the code execution tool to calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"
      }
    ],
    tools: [Anthropic::CodeExecutionTool20250825.new]
  )

  puts message.to_json
  ```
</CodeGroup>

响应中交替出现 `server_tool_use` 块（Claude 运行的命令）及其工具结果块，随后是 Claude 的文本。顶层还包含一个 `container` 对象，您可以[在多个请求之间复用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#container-reuse)其 `id`。有关块的结构，请参阅[响应格式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#response-format)。

## 代码执行的工作原理

当您将代码执行工具添加到 API 请求中时：

1. Claude 会评估代码执行是否有助于回答您的问题。

2. 该工具会自动为 Claude 提供以下能力：

   * **Bash 命令：** 运行 shell 命令以执行系统操作
   * **文件操作：** 直接创建、查看和编辑文件，包括编写代码

3. Claude 可以在单个请求中使用这些能力的任意组合。

4. 所有操作都在安全的沙盒容器中运行。容器没有互联网访问权限，因此 Claude 无法在运行时下载软件包：只能使用[预安装的库](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#pre-installed-libraries)。

5. API 在服务器端运行每条命令，并在同一请求中将结果返回给 Claude，因此您永远不需要自己运行代码或发回 `tool_result` 块。一个例外是当 Claude 在代码执行的同时调用您的某个客户端工具时：API 会返回不带结果的代码执行调用。结果会在您为客户端工具发回 `tool_result` 块之后，在后续响应中到达。

6. 除非您传回先前响应中的容器 ID，否则每个请求都会在新容器中运行（请参阅[容器复用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#container-reuse)）。

7. Claude 会提供结果以及生成的任何图表、计算或分析。

容器预装了 Python。Claude 使用文件操作子工具编写 Python，并通过 Bash 命令运行它。使用 `code_execution_20260120` 或更高版本以及[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)时，Python 解释器状态（例如变量绑定）也会在复用该容器的请求之间保持。

### Claude 何时运行代码

当请求能从计算或文件处理中受益时，Claude 会运行代码：

* 非简单的数学运算（大数、多步骤、对精度敏感的结果）
* 数据分析、文件解析或可视化
* 算法执行或模拟
* 明确要求"运行"、"计算"或"执行"的请求

对于以下情况，Claude 会直接回答而不运行代码：

* 简单算术和众所周知的数学事实
* 事实性、对话性或创意性请求
* 简单的单位换算或翻译

如果您希望 Claude 针对处于边界情况的请求运行代码，请明确提出要求（例如，"运行代码来验证这一点"）。

## 处理文件

### 上传并分析您自己的文件

要分析您自己的数据文件（例如 CSV、Excel 或图像），请通过 Files API 上传它们，并在请求中引用它们。

Python 环境可以处理通过 Files API 上传的各种文件类型，包括：

* CSV
* Excel（.xlsx、.xls）
* JSON
* XML
* 图像（JPEG、PNG、GIF、WebP）
* 文本文件（.txt、.md、.py 等）

#### 上传并分析文件

1. 使用 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) **上传您的文件**。
2. 在消息中使用 `container_upload` 内容块**引用该文件**。
3. 在 API 请求中**包含代码执行工具**。

<CodeGroup>
  ```bash cURL
  # 首先，上传文件
  curl --fail-with-body -sS https://api.anthropic.com/v1/files \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -F "file=@data.csv"

  # 然后将该文件用于代码执行：FILE_ID 是上传响应中返回的 ID
  curl --fail-with-body -sS https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [{
        "role": "user",
        "content": [
          {"type": "text", "text": "Analyze this CSV data"},
          {"type": "container_upload", "file_id": "'"$FILE_ID"'"}
        ]
      }],
      "tools": [{
        "type": "code_execution_20250825",
        "name": "code_execution"
      }]
    }'
  ```

  ```bash CLI
  # 首先，上传一个文件
  ant files upload --file ./data.csv

  # 然后在代码执行中使用该文件：FILE_ID 是上传响应中返回的 ID
  ant messages create <<YAML
  model: claude-opus-5-5
  max_tokens: 4096
  messages:
    - role: user
      content:
        - type: text
          text: Analyze this CSV data
        - type: container_upload
          file_id: $FILE_ID
  tools:
    - type: code_execution_20250825
      name: code_execution
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 上传文件
  file_object = client.files.upload(file=Path("data.csv"))

  # 在代码执行中使用 file_id
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": [
                  {"type": "text", "text": "Analyze this CSV data"},
                  {"type": "container_upload", "file_id": file_object.id},
              ],
          }
      ],
      tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
  )

  print(response.to_json())
  ```

  ```typescript TypeScript
  import { createReadStream } from "node:fs";
  // ...
  const client = new Anthropic();

  // 上传文件
  const fileObject = await client.files.upload({
    file: createReadStream("data.csv")
  });

  // 将 file_id 用于代码执行
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: [
          { type: "text", text: "Analyze this CSV data" },
          { type: "container_upload", file_id: fileObject.id }
        ]
      }
    ],
    tools: [
      {
        type: "code_execution_20250825",
        name: "code_execution"
      }
    ]
  });

  console.log(JSON.stringify(response));
  ```

  ```csharp C#
  AnthropicClient client = new();

  // 上传文件
  var fileObject = await client.Files.Upload(new FileUploadParams
  {
      File = File.OpenRead("data.csv")
  });

  // 将 file_id 用于代码执行
  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [
          new UserMessageParam
          {
              Content = new([
                  new TextBlockParam { Text = "Analyze this CSV data" },
                  new ContainerUploadBlockParam { FileID = fileObject.ID }
              ])
          }
      ],
      Tools = [new CodeExecutionTool20250825()]
  };

  var response = await client.Messages.Create(parameters);
  Console.WriteLine(response);
  ```

  ```go Go
  ctx := context.Background()
  client := anthropic.NewClient()

  // 上传文件
  file, err := os.Open("data.csv")
  if err != nil {
  	log.Fatal(err)
  }
  defer file.Close()

  fileObject, err := client.Files.Upload(ctx, anthropic.FileUploadParams{
  	File: file,
  })
  if err != nil {
  	log.Fatal(err)
  }

  // 将 file_id 用于代码执行
  response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(
  			anthropic.NewTextBlock("Analyze this CSV data"),
  			anthropic.NewContainerUploadBlock(fileObject.ID),
  		),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfCodeExecutionTool20250825: &anthropic.CodeExecutionTool20250825Param{}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(response.RawJSON())
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  // 上传文件
  FileMetadata fileObject = client.files().upload(
      FileUploadParams.builder()
          .file(Path.of("data.csv"))
          .build()
  );

  // 将 file_id 用于代码执行
  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessageOfContentBlockParams(List.of(
              ContentBlockParam.ofText(TextBlockParam.builder()
                  .text("Analyze this CSV data")
                  .build()),
              ContentBlockParam.ofContainerUpload(ContainerUploadBlockParam.builder()
                  .fileId(fileObject.id())
                  .build())
          ))
          .addTool(CodeExecutionTool20250825.builder().build())
          .build()
  );

  IO.println(ObjectMappers.jsonMapper().valueToTree(response));
  ```

  ```php PHP
  $client = new Client();

  // 上传文件
  $fileObject = $client->files->upload(
      file: FileParam::fromResource(fopen('data.csv', 'r')),
  );

  // 将 file_id 与代码执行配合使用
  $response = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => [
                  TextBlockParam::with(text: 'Analyze this CSV data'),
                  ContainerUploadBlockParam::with(fileID: $fileObject->id),
              ],
          ],
      ],
      tools: [new CodeExecutionTool20250825()],
  );

  echo json_encode($response), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 上传文件
  file_object = client.files.upload(
    file: Pathname("data.csv")
  )

  # 在代码执行中使用 file_id
  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: [
          { type: "text", text: "Analyze this CSV data" },
          { type: "container_upload", file_id: file_object.id }
        ]
      }
    ],
    tools: [
      Anthropic::CodeExecutionTool20250825.new
    ]
  )

  puts response.to_json
  ```
</CodeGroup>

### 检索生成的文件

当 Claude 在代码执行期间将文件保存到其输出目录时（请参阅[如何捕获生成的文件](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#how-generated-files-are-captured)），每个文件的 ID 会出现在代码执行工具结果中，您可以使用 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 下载它：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    下载每个生成的文件需要遍历工具结果中的文件 ID，这需要比单个 shell 命令更多的逻辑。请为此使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    下载每个生成的文件需要遍历工具结果中的文件 ID，这需要比单个 shell 命令更多的逻辑。请为此使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)。
  </CodeGroupItem>

  ```python Python
  client = Anthropic()

  # 请求执行会创建文件的代码
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Create a matplotlib visualization and save it as output.png",
          }
      ],
      tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
  )


  # 从响应中提取文件 ID
  def extract_file_ids(response: Message) -> list[str]:
      file_ids: list[str] = []
      for item in response.content:
          if item.type == "bash_code_execution_tool_result":
              content_item = item.content
              if content_item.type == "bash_code_execution_result":
                  for output_block in content_item.content:
                      file_ids.append(output_block.file_id)
      return file_ids


  # 下载所创建的文件
  for file_id in extract_file_ids(response):
      file_metadata = client.files.retrieve_metadata(file_id)
      file_content = client.files.download(file_id)
      file_content.write_to_file(file_metadata.filename)
      print(f"Downloaded: {file_metadata.filename}")
  ```

  ```typescript TypeScript
  import { writeFile } from "node:fs/promises";

  const client = new Anthropic();

  // 请求执行会创建文件的代码
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Create a matplotlib visualization and save it as output.png"
      }
    ],
    tools: [
      {
        type: "code_execution_20250825",
        name: "code_execution"
      }
    ]
  });

  // 从响应中提取文件 ID，并下载每个已创建的文件
  for (const block of response.content) {
    if (block.type === "bash_code_execution_tool_result") {
      const result = block.content;
      if (result.type === "bash_code_execution_result") {
        for (const outputBlock of result.content) {
          const [fileMetadata, fileResponse] = await Promise.all([
            client.files.retrieveMetadata(outputBlock.file_id),
            client.files.download(outputBlock.file_id)
          ]);
          await writeFile(fileMetadata.filename, await fileResponse.bytes());
          console.log(`Downloaded: ${fileMetadata.filename}`);
        }
      }
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [new UserMessageParam { Content = "Create a matplotlib visualization and save it as output.png" }],
      Tools = [new CodeExecutionTool20250825()]
  };

  var response = await client.Messages.Create(parameters);

  // 从工具结果中收集文件 ID
  List<string> fileIds = [];
  foreach (var block in response.Content)
  {
      if (!block.TryPickBashCodeExecutionToolResult(out var toolResult))
          continue;
      if (!toolResult.Content.TryPickBashCodeExecutionResultBlock(out var result))
          continue;
      foreach (var output in result.Content)
      {
          fileIds.Add(output.FileID);
      }
  }

  // 下载每个已创建的文件
  foreach (var fileId in fileIds)
  {
      var fileMetadata = await client.Files.RetrieveMetadata(fileId);
      using var download = await client.Files.Download(fileId);
      var downloadStream = await download.ReadAsStream();
      await using var target = File.Create(fileMetadata.Filename);
      await downloadStream.CopyToAsync(target);
      Console.WriteLine($"Downloaded: {fileMetadata.Filename}");
  }
  ```

  ```go Go
  	client := anthropic.NewClient()
  	ctx := context.Background()

  	response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 4096,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Create a matplotlib visualization and save it as output.png")),
  		},
  		Tools: []anthropic.ToolUnionParam{
  			{OfCodeExecutionTool20250825: &anthropic.CodeExecutionTool20250825Param{}},
  		},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	fileIDs := extractFileIDs(response)

  	for _, fileID := range fileIDs {
  		fileMetadata, err := client.Files.GetMetadata(ctx, fileID, anthropic.FileGetMetadataParams{})
  		if err != nil {
  			log.Fatal(err)
  		}

  		fileContent, err := client.Files.Download(ctx, fileID, anthropic.FileDownloadParams{})
  		if err != nil {
  			log.Fatal(err)
  		}

  		outFile, err := os.Create(fileMetadata.Filename)
  		if err != nil {
  			log.Fatal(err)
  		}

  		_, err = io.Copy(outFile, fileContent.Body)
  		if err != nil {
  			log.Fatal(err)
  		}
  		outFile.Close()
  		fileContent.Body.Close()

  		fmt.Printf("Downloaded: %s\n", fileMetadata.Filename)
  	}
  // ...

  func extractFileIDs(response *anthropic.Message) []string {
  	var fileIDs []string
  	for _, item := range response.Content {
  		switch variant := item.AsAny().(type) {
  		case anthropic.BashCodeExecutionToolResultBlock:
  			// 从工具结果中收集文件 ID
  			for _, file := range variant.Content.Content {
  				if file.FileID != "" {
  					fileIDs = append(fileIDs, file.FileID)
  				}
  			}
  		}
  	}
  	return fileIDs
  }
  ```

  ```java Java
  void main() throws Exception {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(4096L)
          .addUserMessage("Create a matplotlib visualization and save it as output.png")
          .addTool(CodeExecutionTool20250825.builder().build())
          .build();

      Message response = client.messages().create(params);

      List<String> fileIds = extractFileIds(response);

      for (String fileId : fileIds) {
          FileMetadata fileMetadata = client.files().retrieveMetadata(fileId);
          try (HttpResponse fileContent = client.files().download(fileId)) {
              Files.copy(
                  fileContent.body(),
                  Path.of(fileMetadata.filename()),
                  StandardCopyOption.REPLACE_EXISTING);
          }
          IO.println("Downloaded: " + fileMetadata.filename());
      }
  }

  List<String> extractFileIds(Message response) {
      List<String> fileIds = new ArrayList<>();
      // 从工具结果中收集文件 ID
      for (ContentBlock item : response.content()) {
          item.bashCodeExecutionToolResult().ifPresent(toolResult -> {
              if (toolResult.content().isBashCodeExecutionResultBlock()) {
                  BashCodeExecutionResultBlock result =
                      toolResult.content().asBashCodeExecutionResultBlock();
                  for (BashCodeExecutionOutputBlock output : result.content()) {
                      fileIds.add(output.fileId());
                  }
              }
          });
      }
      return fileIds;
  }
  ```

  ```php PHP
  $client = new Client();

  // 请求执行会创建文件的代码
  $response = $client->messages->create(
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => 'Create a matplotlib visualization and save it as output.png',
          ],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [new CodeExecutionTool20250825()],
  );

  /**
   * Extract file IDs from the response.
   *
   * @return list<string>
   */
  function extractFileIds(Message $response): array
  {
      $fileIds = [];
      foreach ($response->content as $block) {
          if ($block->type !== 'bash_code_execution_tool_result') {
              continue;
          }
          $resultBlock = $block->content;
          if ($resultBlock->type !== 'bash_code_execution_result') {
              continue;
          }
          foreach ($resultBlock->content as $outputBlock) {
              $fileIds[] = $outputBlock->fileID;
          }
      }
      return $fileIds;
  }

  // 下载已创建的文件
  foreach (extractFileIds($response) as $fileId) {
      $fileMetadata = $client->files->retrieveMetadata($fileId);
      $fileContent = $client->files->download($fileId);

      file_put_contents($fileMetadata->filename, $fileContent);
      echo "Downloaded: {$fileMetadata->filename}\n";
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
        content: "Create a matplotlib visualization and save it as output.png"
      }
    ],
    tools: [
      {
        type: "code_execution_20250825",
        name: "code_execution"
      }
    ]
  )

  def extract_file_ids(response)
    file_ids = []
    response.content.each do |item|
      if item.type == :bash_code_execution_tool_result
        # 从工具结果中收集文件 ID
        content_item = item.content
        if content_item[:type].to_s == "bash_code_execution_result"
          Array(content_item[:content]).each do |output_block|
            file_ids << output_block[:file_id]
          end
        end
      end
    end
    file_ids
  end

  extract_file_ids(response).each do |file_id|
    file_metadata = client.files.retrieve_metadata(file_id)
    file_content = client.files.download(file_id)

    File.open(file_metadata.filename, "wb") do |f|
      f.write(file_content.read)
    end

    puts "Downloaded: #{file_metadata.filename}"
  end
  ```
</CodeGroup>

#### 如何捕获生成的文件

每次 `bash_code_execution` 调用都会获得一个新的空目录，命令可以通过 `$OUTPUT_DIR` 访问该目录。命令完成后，该目录顶层的文件会被捕获，并作为结果 `content` 列表中的 `file_id` 条目返回。写入其他任何位置的文件会保留在容器中，不会被返回。

工具描述会告知 Claude 通过将文件复制到 `$OUTPUT_DIR` 来共享文件。如果您的应用程序依赖于接收某个文件，请提示 Claude 将其复制到 `$OUTPUT_DIR`，并在同一命令中列出该目录，以便 `ls` 的输出确认捕获成功（Claude 不会收到 `content` 列表）：

```bash
python /tmp/make_report.py && cp /tmp/report.pdf "$OUTPUT_DIR/" && ls "$OUTPUT_DIR"
```

Claude 写入其他位置的文件仍在容器中，因此您可以[复用该容器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#container-reuse)并要求 Claude 将其复制到 `$OUTPUT_DIR`。

### 生成文件上的内容凭证

在 Claude API 上，当您通过 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 下载 Claude 在代码执行沙盒中生成的受支持图像、视频和音频文件时，这些文件会带有 [C2PA](https://c2pa.org/) 内容凭证（Content Credentials）。[受支持的格式](https://opensource.contentauthenticity.org/docs/sdk-repos/c2pa-python/docs/supported-formats/)包括 PNG、JPEG、GIF、WebP、TIFF、HEIC、AVIF、SVG、MP4、MOV、MP3、WAV、FLAC 和 M4A。该凭证是嵌入在文件元数据中的经过加密签名的清单。它将 Anthropic 标识为签发方，带有时间戳，并记录操作描述"Claude provided this file at the request of a user and may have created or modified the file contents."

签名不需要对您的请求或响应处理进行任何更改，并且清单不会记录有关您、您的组织或您的请求的任何信息。文件的可见内容保持不变。清单会增加几千字节，因此下载文件的大小和校验和与容器内的文件不同。文本文件、PDF 和办公文档不会被签名，因为它们不是受支持的签名格式。您上传的文件会按原样存储，包括它们已携带的任何内容凭证。

要验证凭证，请使用任何兼容 C2PA 的工具检查文件，例如开源的 [c2patool 命令行实用程序](https://github.com/contentauth/c2pa-rs)。重新编码、格式转换、屏幕截图以及剥离元数据的工具会移除凭证，因此缺少凭证并不意味着文件不是由 Claude 生成的。有关凭证可能缺失的更多原因，请参阅 [Claude 如何标记 AI 生成的内容](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)。

## 工具定义

代码执行工具不需要额外的参数：

```json JSON
{
  "type": "code_execution_20250825",
  "name": "code_execution"
}
```

这两个字段都是固定的：`type` 用于选择工具版本，`name` 必须为 `code_execution`。

当您提供此工具时，Claude 会自动获得两个子工具的访问权限：

* `bash_code_execution`：运行 shell 命令
* `text_editor_code_execution`：查看、创建和编辑文件，包括编写代码

当 Claude 运行代码时，响应还会包含一个顶层 `container` 对象，其中包含容器的 `id` 和 `expires_at` 时间戳。在顶层 `container` 请求参数中传回该 ID，即可继续使用同一容器。请参阅[容器复用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#container-reuse)。

## 响应格式

代码执行工具可以根据操作返回两种类型的结果：

### Bash 命令响应

```json Output
{
  "type": "server_tool_use",
  "id": "srvtoolu_01B3C4D5E6F7G8H9I0J1K2L3",
  "name": "bash_code_execution",
  "input": {
    "command": "ls -la | head -5"
  }
},
{
  "type": "bash_code_execution_tool_result",
  "tool_use_id": "srvtoolu_01B3C4D5E6F7G8H9I0J1K2L3",
  "content": {
    "type": "bash_code_execution_result",
    "stdout": "total 24\ndrwxr-xr-x 2 user user 4096 Jan 1 12:00 .\ndrwxr-xr-x 3 user user 4096 Jan 1 11:00 ..\n-rw-r--r-- 1 user user  220 Jan 1 12:00 data.csv\n-rw-r--r-- 1 user user  180 Jan 1 12:00 config.json",
    "stderr": "",
    "return_code": 0,
    "content": []
  }
}
```

### 文件操作响应

**查看文件：**

```json Output
{
  "type": "server_tool_use",
  "id": "srvtoolu_01C4D5E6F7G8H9I0J1K2L3M4",
  "name": "text_editor_code_execution",
  "input": {
    "command": "view",
    "path": "config.json"
  }
},
{
  "type": "text_editor_code_execution_tool_result",
  "tool_use_id": "srvtoolu_01C4D5E6F7G8H9I0J1K2L3M4",
  "content": {
    "type": "text_editor_code_execution_view_result",
    "file_type": "text",
    "content": "{\n  \"setting\": \"value\",\n  \"debug\": true\n}",
    "num_lines": 4,
    "start_line": 1,
    "total_lines": 4
  }
}
```

**创建文件：**

```json Output
{
  "type": "server_tool_use",
  "id": "srvtoolu_01D5E6F7G8H9I0J1K2L3M4N5",
  "name": "text_editor_code_execution",
  "input": {
    "command": "create",
    "path": "new_file.txt",
    "file_text": "Hello, World!"
  }
},
{
  "type": "text_editor_code_execution_tool_result",
  "tool_use_id": "srvtoolu_01D5E6F7G8H9I0J1K2L3M4N5",
  "content": {
    "type": "text_editor_code_execution_create_result",
    "is_file_update": false
  }
}
```

**编辑文件（str\_replace）：**

```json Output
{
  "type": "server_tool_use",
  "id": "srvtoolu_01E6F7G8H9I0J1K2L3M4N5O6",
  "name": "text_editor_code_execution",
  "input": {
    "command": "str_replace",
    "path": "config.json",
    "old_str": "\"debug\": true",
    "new_str": "\"debug\": false"
  }
},
{
  "type": "text_editor_code_execution_tool_result",
  "tool_use_id": "srvtoolu_01E6F7G8H9I0J1K2L3M4N5O6",
  "content": {
    "type": "text_editor_code_execution_str_replace_result",
    "old_start": 3,
    "old_lines": 1,
    "new_start": 3,
    "new_lines": 1,
    "lines": ["-  \"debug\": true", "+  \"debug\": false"]
  }
}
```

### 结果

Bash 命令结果（`bash_code_execution_result`）包括：

* `stdout`：成功执行的输出
* `stderr`：执行失败时的错误消息
* `return_code`：成功为 0，失败为非零值
* `content`：一个列表，命令留在 `$OUTPUT_DIR` 中的每个文件对应一个条目（请参阅[如何捕获生成的文件](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#how-generated-files-are-captured)）。每个条目都带有 `file_id`，可用于通过 Files API [检索该文件](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#retrieve-generated-files)

文件操作结果有各自的字段：

* **查看**（`text_editor_code_execution_view_result`）：`file_type`、`content`、`num_lines`、`start_line`、`total_lines`
* **创建**（`text_editor_code_execution_create_result`）：`is_file_update`（文件是否已存在）
* **编辑**（`text_editor_code_execution_str_replace_result`）：`old_start`、`old_lines`、`new_start`、`new_lines`、`lines`（diff 格式）

### 错误

每种工具类型都可能返回特定的错误：

**常见错误（所有工具）：**

```json Output
{
  "type": "bash_code_execution_tool_result",
  "tool_use_id": "srvtoolu_01VfmxgZ46TiHbmXgy928hQR",
  "content": {
    "type": "bash_code_execution_tool_result_error",
    "error_code": "unavailable"
  }
}
```

**按工具类型划分的错误代码：**

| 工具           | 错误代码                      | 描述               |
| ------------ | ------------------------- | ---------------- |
| 所有工具         | `unavailable`             | 该工具暂时不可用         |
| 所有工具         | `execution_time_exceeded` | 工具调用超过了最大执行时间    |
| 所有工具         | `invalid_tool_input`      | 向工具提供了无效参数       |
| 所有工具         | `too_many_requests`       | 工具使用超出速率限制       |
| bash         | `output_file_too_large`   | 命令输出超过了最大大小      |
| text\_editor | `file_not_found`          | 文件不存在（针对查看/编辑操作） |

已过期的容器无法复用：引用它的请求会返回错误，而不会恢复它。请在不带 `container` 参数的情况下重新发送请求，以获取新容器。

### `pause_turn` 停止原因

响应可能包含 `pause_turn` 停止原因，这表示 API 暂停了一个长时间运行的轮次。您可以 在后续请求中按原样提供该响应，让 Claude 继续其轮次；如果您想中断对话，也可以 修改内容。

请在该请求中发回响应的容器 ID（请参阅[容器复用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#container-reuse)）。如果不发回，该请求将在新容器中运行，本轮早先产生的文件将会丢失。

## 容器

代码执行工具在专为代码执行设计的安全容器化环境中运行，更侧重于 Python。

### 运行时环境

* **Python 版本：** 3.11
* **操作系统：** 基于 Linux 的容器
* **架构：** x86\_64（AMD64）

### 资源限制

* **内存：** 容器有 5 GiB 的 RAM。
* **磁盘空间：** 容器有 5 GiB 的工作区存储。
* **CPU：** 容器有 1 个 CPU。
* **执行时间：** 运行超过最大执行时间的工具调用会返回 `execution_time_exceeded` [错误](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#errors)。使用[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)时，每个 REPL 单元还有 90 秒的挂钟时间限制。

### 网络与安全

* **互联网访问：** 出于安全考虑完全禁用
* **外部连接：** 不允许出站网络请求
* **沙盒隔离：** 与主机系统和其他容器完全隔离
* **文件访问：** 仅限于工作区目录
* **工作区范围：** 与 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 一样，限定在请求所属的工作区范围内
* **过期：** 创建后 30 天

### 预安装的库

沙盒化的 Python 环境包含以下常用库：

* **数据科学：** pandas、numpy、scipy、scikit-learn、statsmodels
* **可视化：** matplotlib、seaborn
* **文件处理：** pyarrow、openpyxl、xlsxwriter、xlrd、pillow、python-pptx、python-docx、pypdf、pdfplumber、pypdfium2、pdf2image、pdfkit、tabula-py、reportlab\[pycairo]、Img2pdf
* **数学与计算：** sympy、mpmath
* **实用工具：** tqdm、python-dateutil、pytz、joblib

容器还包含命令行工具，例如 unzip、unrar、7zip、bc、rg（ripgrep）、fd 和 sqlite。

容器没有互联网访问权限，因此 Claude 无法在运行时下载或安装额外的软件包：只能使用预安装的库。

## 容器复用

您可以通过提供先前响应中的容器 ID，在多个 API 请求之间复用现有容器。 这使您可以在请求之间保留已创建的文件。使用 `code_execution_20260120` 或更高版本以及[程序化工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)时，Python 解释器状态也会保持。

容器在创建后 30 天过期。在大约 5 分钟无活动后，容器会被设置检查点，在 30 天期限内发送带有其 ID 的请求即可恢复它。响应 `container` 对象中的 `expires_at` 时间戳是一个较短的滚动值，并不反映 30 天的期限。已过期的容器无法复用。请在不带 `container` 参数的情况下重新发送请求，以获取新容器。

### 示例

<CodeGroup>
  ```bash cURL
  # 第一个请求：创建一个包含随机数的文件
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [{
        "role": "user",
        "content": "Write a file with a random number and save it to \"/tmp/number.txt\""
      }],
      "tools": [{
        "type": "code_execution_20250825",
        "name": "code_execution"
      }]
    }'

  # 第二个请求：复用容器以读取该文件
  # CONTAINER_ID 是第一个响应中的 container.id
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "container": "'"$CONTAINER_ID"'",
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [{
        "role": "user",
        "content": "Read the number from \"/tmp/number.txt\" and calculate its square"
      }],
      "tools": [{
        "type": "code_execution_20250825",
        "name": "code_execution"
      }]
    }'
  ```

  ```bash CLI
  # 第一个请求：创建一个包含随机数的文件
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --message '{role: user, content: Write a file with a random number and save it to "/tmp/number.txt"}' \
    --tool '{type: code_execution_20250825, name: code_execution}'

  # 第二个请求：复用容器来读取该文件
  # CONTAINER_ID 是第一个响应中的 container.id
  ant messages create \
    --container "$CONTAINER_ID" \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --message '{role: user, content: Read the number from "/tmp/number.txt" and calculate its square}' \
    --tool '{type: code_execution_20250825, name: code_execution}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 第一个请求：在新容器中创建一个包含随机数的文件
  response1 = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Write a file with a random number and save it to '/tmp/number.txt'",
          }
      ],
      tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
  )

  # 第二个请求：传回容器 ID，让 Claude 复用同一个容器
  response2 = client.messages.create(
      container=response1.container.id,
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Read the number from '/tmp/number.txt' and calculate its square",
          }
      ],
      tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
  )

  print(response2.to_json())
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 第一个请求：Claude 在一个新的代码执行容器中创建文件
  const response1 = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Write a file with a random number and save it to '/tmp/number.txt'"
      }
    ],
    tools: [{ type: "code_execution_20250825", name: "code_execution" }]
  });

  // 代码执行工具运行后，响应中会包含该容器
  if (!response1.container) {
    throw new Error("Expected the first response to include a container");
  }

  // 第二个请求：传回容器 ID，以便复用同一个容器
  const response2 = await client.messages.create({
    container: response1.container.id,
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Read the number from /tmp/number.txt and calculate its square" }
    ],
    tools: [{ type: "code_execution_20250825", name: "code_execution" }]
  });

  console.log(JSON.stringify(response2));
  ```

  ```csharp C#
  AnthropicClient client = new();

  // 第一个请求：Claude 在一个全新的代码执行容器中创建文件
  var response1 = await client.Messages.Create(new()
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [new UserMessageParam { Content = "Write a file with a random number and save it to '/tmp/number.txt'" }],
      Tools = [new CodeExecutionTool20250825()]
  });

  // 第二个请求：传回容器 ID，使 Claude 复用同一个容器
  var response2 = await client.Messages.Create(new()
  {
      Container = response1.Container!.ID,
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [new UserMessageParam { Content = "Read the number from '/tmp/number.txt' and calculate its square" }],
      Tools = [new CodeExecutionTool20250825()]
  });

  Console.WriteLine(response2);
  ```

  ```go Go
  client := anthropic.NewClient()
  ctx := context.Background()

  codeExecution := []anthropic.ToolUnionParam{
  	{OfCodeExecutionTool20250825: &anthropic.CodeExecutionTool20250825Param{}},
  }

  // 第一个请求：在新容器中创建一个包含随机数的文件
  response1, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Write a file with a random number and save it to '/tmp/number.txt'")),
  	},
  	Tools: codeExecution,
  })
  if err != nil {
  	log.Fatal(err)
  }

  // 复用第一个请求的容器，因此该文件仍然存在。
  response2, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  	Container: anthropic.MessageCreateParamsContainerUnion{
  		OfString: anthropic.String(response1.Container.ID),
  	},
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Read the number from '/tmp/number.txt' and calculate its square")),
  	},
  	Tools: codeExecution,
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(response2.RawJSON())
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  // 第一个请求：在新容器中创建一个包含随机数的文件
  MessageCreateParams params1 = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(4096L)
      .addUserMessage("Write a file with a random number and save it to '/tmp/number.txt'")
      .addTool(CodeExecutionTool20250825.builder().build())
      .build();

  Message response1 = client.messages().create(params1);

  // 第二个请求：传回容器 ID，以便复用同一个容器
  MessageCreateParams params2 = MessageCreateParams.builder()
      .container(response1.container().orElseThrow().id())
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(4096L)
      .addUserMessage("Read the number from '/tmp/number.txt' and calculate its square")
      .addTool(CodeExecutionTool20250825.builder().build())
      .build();

  Message response2 = client.messages().create(params2);
  IO.println(ObjectMappers.jsonMapper().valueToTree(response2));
  ```

  ```php PHP
  $client = new Client();

  // 第一个请求：Claude 在全新的代码执行容器中写入该文件
  $response1 = $client->messages->create(
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => "Write a file with a random number and save it to '/tmp/number.txt'",
          ],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [new CodeExecutionTool20250825()],
  );

  // 第二个请求：复用该容器，因此 '/tmp/number.txt' 仍然存在
  $response2 = $client->messages->create(
      container: $response1->container->id,
      maxTokens: 4096,
      messages: [
          [
              'role' => 'user',
              'content' => "Read the number from '/tmp/number.txt' and calculate its square",
          ],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [new CodeExecutionTool20250825()],
  );

  echo json_encode($response2), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 第一个请求：Claude 在新的代码执行容器中创建文件
  response1 = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Write a file with a random number and save it to '/tmp/number.txt'"
      }
    ],
    tools: [Anthropic::CodeExecutionTool20250825.new]
  )

  # 第二个请求：传回容器 ID，以便 Claude 复用同一容器
  response2 = client.messages.create(
    container: response1.container.id,
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Read the number from '/tmp/number.txt' and calculate its square"
      }
    ],
    tools: [Anthropic::CodeExecutionTool20250825.new]
  )

  puts response2.to_json
  ```
</CodeGroup>

## 将代码执行与其他执行工具结合使用

当您在提供代码执行的同时还提供了其他也能运行代码的客户端工具（例如 [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)或自定义 REPL）时，Claude 就处于一个多计算机环境中。代码执行工具在 Anthropic 的沙盒容器中运行，而您的客户端工具则在您控制的独立环境中运行。Claude 有时可能会使用错误的工具，或表现得好像这些环境之间共享状态。

为避免这种情况，请在系统提示中添加说明以阐明两者的区别：

```text wrap
When multiple code execution environments are available, be aware that:
- Variables, files, and state do NOT persist between different execution environments
- Use the code_execution tool for general-purpose computation in Anthropic's sandboxed environment
- Use client-provided execution tools (e.g., bash) when you need access to the user's local system, files, or data
- If you need to pass results between environments, explicitly include outputs in subsequent tool calls rather than assuming shared state
```

在将代码执行与[网络搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)或[网页获取](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)结合使用时，这一点尤为重要，因为它们会自动启用代码执行。如果您的应用程序已经提供了客户端 shell 工具，自动代码执行会创建第二个执行环境，Claude 需要将其与第一个环境区分开来。

当 Claude 在代码执行的同时调用您的某个客户端工具时，API 会返回不带结果的代码执行调用。结果会在您为客户端工具发回 `tool_result` 块之后，在后续响应中到达。

## 流式传输

启用[流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)（`"stream": true`）后，您将在代码执行事件发生时接收它们。子工具输入以 `input_json_delta` 事件的形式流式传输，而每个结果块则在单个 `content_block_start` 事件中完整到达：

```sse
event: content_block_start
data: {"type": "content_block_start", "index": 1, "content_block": {"type": "server_tool_use", "id": "srvtoolu_xyz789", "name": "bash_code_execution"}}

// Tool input streamed as partial JSON
event: content_block_delta
data: {"type": "content_block_delta", "index": 1, "delta": {"type": "input_json_delta", "partial_json": "{\"command\": \"python analyze.py\"}"}}

// Pause while the command runs

// Execution result delivered as a complete block
event: content_block_start
data: {"type": "content_block_start", "index": 2, "content_block": {"type": "bash_code_execution_tool_result", "tool_use_id": "srvtoolu_xyz789", "content": {"type": "bash_code_execution_result", "stdout": "   A  B  C\n0  1  2  3\n1  4  5  6", "stderr": "", "return_code": 0, "content": []}}}
```

## 批量请求

您可以在 [Messages Batches API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 中包含代码执行工具。通过 Messages Batches API 进行的代码执行工具调用与常规 Messages API 请求中的调用定价相同。

## 用量与定价

**与 web search（网页搜索）或 web fetch（网页抓取）一起使用时，代码执行是免费的。** 当您的 API 请求中包含 `web_search_20260209`（或更高版本）或 `web_fetch_20260209`（或更高版本）时，除标准的输入和输出令牌费用外，代码执行工具调用不会产生额外费用。

在不与这些工具一起使用时，代码执行按执行时间计费，并与令牌用量分开跟踪：

* 执行时间最少按 5 分钟计算。
* 每个组织每月可获得 **1,550 小时免费**使用时长。
* 超出 1,550 小时的额外使用按**每个容器每小时 0.05 美元**计费。
* 如果请求中包含文件，即使未调用该工具，也会对执行时间计费，因为文件会预先加载到容器中。

代码执行用量会在响应中进行跟踪：

```json
{
  "usage": {
    "input_tokens": 105,
    "output_tokens": 239,
    "server_tool_use": {
      "code_execution_requests": 1
    }
  }
}
```

## 升级到最新工具版本

最新的工具版本是 `code_execution_20260521`。要在三个当前版本之间切换，请更新请求中的 `type` 字符串：这三个版本都会返回[响应格式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#response-format)中记录的响应块。有关每个版本新增的功能，请参阅[工具版本](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#tool-versions)；有关支持它们的模型，请参阅 [API 支持](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#availability-feature-support)。

本节其余部分介绍如何从旧版仅支持 Python 的 `code_execution_20250522` 迁移到当前工具版本。

### 变更内容

| 组件      | 旧版                          | 当前                                                                 |
| ------- | --------------------------- | ------------------------------------------------------------------ |
| Beta 标头 | `code-execution-2025-05-22` | 无需                                                                 |
| 工具类型    | `code_execution_20250522`   | `code_execution_20250825` 或更高版本                                    |
| 能力      | 仅 Python                    | Bash 命令、文件操作                                                       |
| 响应类型    | `code_execution_result`     | `bash_code_execution_result`、`text_editor_code_execution_*_result` |

### 向后兼容性

* 所有现有的 Python 代码执行将继续完全按原样工作
* 现有的仅 Python 工作流无需任何更改

### 升级步骤

要升级，请更新 API 请求中的工具类型：

```diff
- "type": "code_execution_20250522"
+ "type": "code_execution_20250825"
```

**检查响应处理**（如果以编程方式解析响应）：

* API 不再为 Python 执行响应发送以前的块。
* 取而代之的是，API 会为 Bash 和文件操作发送新的响应类型（请参阅[响应格式](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#response-format)）。

## 数据保留

代码执行在服务器端沙盒容器中运行。容器数据（包括执行产物、上传的文件和输出）最多保留 30 天。此保留期适用于在容器环境中处理的所有数据。代码执行在 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 中创建的文件（可通过 `client.files.download()` (csharp, go: `client.Files.Download()`; java: `client.files().download()`; php: `$client->files->download()`) 检索）会一直保留，直到被明确删除。

## 后续步骤

<CardGroup cols={2}>
  <Card title="Advisor 工具" icon="compass" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool">
    将速度更快的执行模型与智能程度更高的顾问模型配对，由顾问模型在生成过程中提供战略性 指导。
  </Card>

  <Card title="程序化工具调用" icon="code" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling">
    从代码执行容器内运行的代码中调用您自己的工具。
  </Card>

  <Card title="Files API" icon="file" href="https://platform.claude.com/docs/zh-CN/build-with-claude/files">
    上传文件以供分析，并下载代码执行创建的文件。
  </Card>

  <Card title="通过 API 使用 Agent Skills" icon="book" href="https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide">
    了解如何使用 Agent Skills 通过 API 扩展 Claude 的能力。
  </Card>
</CardGroup>

## 可用性

代码执行工具在所有模型上可用。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Microsoft Foundry (hosted on Anthropic)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                      | HIPAA |
| ---- | -------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 不具备资格                                                                                                                      | 不具备资格 |
| 详细信息 | 容器数据最多保留 30 天。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#data-retention) | —     |

适用的端点：

* `/v1/messages` 使用 `code_execution` 工具
