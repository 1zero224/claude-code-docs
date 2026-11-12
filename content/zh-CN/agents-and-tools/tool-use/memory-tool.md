---
title: 记忆工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool
description: 通过在您的应用程序中实现记忆工具的文件操作，让 Claude 跨对话存储和检索信息。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 客户端记忆存储，数据保留由您控制。
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
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

记忆工具（memory tool）让 Claude 能够在一个记忆文件目录中跨对话存储和检索信息。Claude 可以创建、读取、更新和删除在会话之间持久保存的文件，从而随着时间推移积累知识，而无需将所有内容都保留在 “context window”（上下文窗口）中。

记忆支持 “just-in-time context retrieval”（即时上下文检索）。智能体不是预先加载所有相关信息，而是将其学到的内容记录在记忆文件中，并按需读回。这使活动上下文专注于当前任务，这对于长时间运行的会话非常重要，否则这些会话可能会超出上下文窗口的容量。有关更广泛的模式，请参阅[有效的上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)。

记忆工具在客户端运行：Claude 请求文件操作，由您的应用程序执行这些操作。您可以通过自己的基础设施控制数据的存储位置和方式。

<Note>
  请使用[反馈表单](https://forms.gle/YXC2EKGMhjN1c4L88)分享您对此功能的反馈。
</Note>

## 使用场景

* 在多个智能体会话之间维护项目上下文
* 将过去交互、决策和反馈中的经验应用于新任务
* 随着时间推移构建知识库

## 工作原理

启用记忆工具后，Claude 在开始任务之前会自动检查其记忆目录。在工作过程中，Claude 会将学到的内容存储在 `/memories` 下的文件中，并在之后的对话中读回这些内容以继续先前的工作。

由于记忆工具是客户端工具，Claude 只会请求记忆操作。您的应用程序针对您控制的存储执行每个请求，并在 `tool_result` 块中返回结果（请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)）。`/memories` 路径是一个前缀，由您的处理程序将其映射到实际存储，例如每个用户的目录或数据库中的键。记忆完全存在于您的应用程序中。当之后的对话发送相同的 `tools` 条目且您的处理程序提供相同的存储时，该对话就会从相同的记忆继续。出于安全考虑，请将所有记忆操作限制在 `/memories` 目录内（请参阅[路径遍历防护](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool#path-traversal-protection)）。

### 示例：记忆工具调用的工作方式

典型的交互如下所示：

**1. 用户请求：**

```text wrap
"Help me respond to this customer service ticket."
```

**2. Claude 检查记忆目录：**

```text wrap
"I'll help you respond to the customer service ticket. Let me check my memory for any previous context."
```

Claude 调用记忆工具：

```json
{
  "type": "tool_use",
  "id": "toolu_01C4D5E6F7G8H9I0J1K2L3M4",
  "name": "memory",
  "input": {
    "command": "view",
    "path": "/memories"
  }
}
```

**3. 您的应用程序返回目录内容：**

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01C4D5E6F7G8H9I0J1K2L3M4",
  "content": "Here're the files and directories up to 2 levels deep in /memories, excluding hidden items and node_modules:\n4.0K\t/memories\n1.5K\t/memories/customer_service_guidelines.xml\n2.0K\t/memories/refund_policies.xml"
}
```

**4. Claude 读取相关文件：**

```json
{
  "type": "tool_use",
  "id": "toolu_01D5E6F7G8H9I0J1K2L3M4N5",
  "name": "memory",
  "input": {
    "command": "view",
    "path": "/memories/customer_service_guidelines.xml"
  }
}
```

**5. 您的应用程序返回文件内容：**

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01D5E6F7G8H9I0J1K2L3M4N5",
  "content": "Here's the content of /memories/customer_service_guidelines.xml with line numbers:\n     1\t<guidelines>\n     2\t<addressing_customers>\n     3\t- Always address customers by their first name\n     4\t- Use empathetic language\n..."
}
```

**6. Claude 利用记忆提供帮助：**

```text wrap
"Based on your customer service guidelines, I can help you craft a response. Please share the ticket details..."
```

记忆工具适用于所有 Claude 4 及更高版本的模型。有关 Anthropic 提供的工具的完整列表，请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)。

## 入门

使用记忆工具需要两个步骤：

1. 将记忆工具添加到您的请求中。`tools` 条目 `{"type": "memory_20250818", "name": "memory"}` 就是全部配置：`name` 必须为 `memory`，并且您无需为 Anthropic 提供的工具定义输入架构。
2. 为每个记忆命令实现客户端处理程序。您的处理程序必须拒绝 `/memories` 之外的路径，因此在编写之前请阅读[路径遍历防护](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool#path-traversal-protection)。

## 基本用法

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 2048,
      "messages": [
        {
          "role": "user",
          "content": "Help me respond to this customer service ticket."
        }
      ],
      "tools": [{
        "type": "memory_20250818",
        "name": "memory"
      }]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 2048
  tools:
    - type: memory_20250818
      name: memory
  messages:
    - role: user
      content: Help me respond to this customer service ticket.
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=2048,
      messages=[
          {
              "role": "user",
              "content": "Help me respond to this customer service ticket.",
          }
      ],
      tools=[{"type": "memory_20250818", "name": "memory"}],
  )

  print(message)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  const message = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 2048,
    messages: [
      {
        role: "user",
        content: "Help me respond to this customer service ticket."
      }
    ],
    tools: [{ type: "memory_20250818", name: "memory" }]
  });

  console.log(message);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var message = await client.Messages.Create(
      new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 2048,
          Messages =
          [
              new UserMessageParam { Content = "Help me respond to this customer service ticket." },
          ],
          Tools = [new MemoryTool20250818()],
      }
  );

  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 2048,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Help me respond to this customer service ticket.")),
  	},
  	Tools: []anthropic.ToolUnionParam{
  		{OfMemoryTool20250818: &anthropic.MemoryTool20250818Param{}},
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(message)
  ```

  ```java Java
  import com.anthropic.models.messages.MemoryTool20250818;
  // ...
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(2048L)
          .addTool(MemoryTool20250818.builder().build())
          .addUserMessage("Help me respond to this customer service ticket.")
          .build();

      Message message = client.messages().create(params);
      IO.println(message);
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 2048,
      messages: [
          [
              'role' => 'user',
              'content' => 'Help me respond to this customer service ticket.',
          ],
      ],
      tools: [new MemoryTool20250818()],
  );

  echo $message;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 2048,
    messages: [
      {
        role: "user",
        content: "Help me respond to this customer service ticket."
      }
    ],
    tools: [
      {
        type: "memory_20250818",
        name: "memory"
      }
    ]
  )
  puts message
  ```
</CodeGroup>

## 实现记忆处理程序

Claude 对类似上述请求的回复会以一个请求记忆操作（例如 `view /memories`）的 `tool_use` 块结尾。您的应用程序执行该操作并在 `tool_result` 块中返回结果，然后将对话发送回去，以便 Claude 继续：这就是标准的[工具使用循环](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。

有四个 SDK 提供了记忆工具辅助程序，用于处理工具接口和循环。继承 `BetaAbstractMemoryTool`（Python 和 C#）、使用 `betaMemoryTool`（TypeScript）或实现 `BetaMemoryToolHandler`（Java），即可使用您自己的存储来支持记忆，例如磁盘上的文件、数据库、云存储或加密文件。Python 和 TypeScript 还提供了一个现成的本地文件系统实现 `BetaLocalFilesystemMemoryTool`。尽管记忆工具本身不需要 beta 标头，但辅助程序和工具运行器接口位于 SDK 的 beta 命名空间中。Go 和 Ruby SDK 没有记忆辅助程序，因此这些示例自行运行工具使用循环，而 PHP 则将您的处理程序闭包包装在其通用的 `BetaRunnableTool` 中。这三者都使用内存存储，您可以将其替换为自己的存储。

<CodeGroup exclude="shell">
  ```python Python
  import anthropic
  from anthropic.tools import BetaLocalFilesystemMemoryTool

  client = anthropic.Anthropic()
  memory = BetaLocalFilesystemMemoryTool(base_path="./memory")

  runner = client.beta.messages.tool_runner(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[
          {
              "role": "user",
              "content": "Remember that customer Acme Corp prefers email follow-ups.",
          }
      ],
      tools=[memory],
  )

  final_message = runner.until_done()
  print(final_message.content)
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";
  import { betaMemoryTool } from "@anthropic-ai/sdk/helpers/beta/memory";
  import { BetaLocalFilesystemMemoryTool } from "@anthropic-ai/sdk/tools/memory/node";

  const client = new Anthropic();

  const backend = await BetaLocalFilesystemMemoryTool.init("./memory");
  const memory = betaMemoryTool(backend); // or pass your own handlers object

  const runner = client.beta.messages.toolRunner({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: "Remember that customer Acme Corp prefers email follow-ups."
      }
    ],
    tools: [memory],
    max_iterations: 10
  });

  const finalMessage = await runner;
  console.log(finalMessage.content);
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Helpers.Beta;
  using Anthropic.Models.Beta.Messages;
  using Model = Anthropic.Models.Messages.Model;

  var client = new AnthropicClient();

  // 您的 BetaAbstractMemoryTool 子类
  var memory = new FilesystemMemoryTool("./memories");

  var runner = client.Beta.Messages.ToolRunner(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages =
          [
              new BetaUserMessageParam
              {
                  Content = "Remember that customer Acme Corp prefers email follow-ups.",
              },
          ],
      },
      [memory],
      maxIterations: 10
  );

  var finalMessage = await runner.RunUntilDoneAsync();
  Console.WriteLine(finalMessage);
  ```

  ```go Go
  package main

  import (
  	"context"
  	"encoding/json"
  	"fmt"
  	"log"
  	"slices"
  	"sort"
  	"strings"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  // 一个内存存储，将记忆文件路径映射到其内容。
  // 在生产环境中请使用您自己的存储。
  var store = map[string]string{}

  type memoryCommand struct {
  	Command    string `json:"command"`
  	Path       string `json:"path"`
  	FileText   string `json:"file_text"`
  	OldStr     string `json:"old_str"`
  	NewStr     string `json:"new_str"`
  	InsertLine int    `json:"insert_line"`
  	InsertText string `json:"insert_text"`
  	OldPath    string `json:"old_path"`
  	NewPath    string `json:"new_path"`
  }

  func executeMemory(raw json.RawMessage) string {
  	var cmd memoryCommand
  	if err := json.Unmarshal(raw, &cmd); err != nil {
  		return "Error: invalid memory command"
  	}
  	switch cmd.Command {
  	case "view":
  		if content, ok := store[cmd.Path]; ok {
  			lines := strings.Split(strings.TrimSuffix(content, "\n"), "\n")
  			for i, line := range lines {
  				lines[i] = fmt.Sprintf("%6d\t%s", i+1, line)
  			}
  			return fmt.Sprintf("Here's the content of %s with line numbers:\n%s", cmd.Path, strings.Join(lines, "\n"))
  		}
  		if cmd.Path == "/memories" {
  			listing := []string{"1.0K\t/memories"}
  			for path := range store {
  				listing = append(listing, "1.0K\t"+path)
  			}
  			sort.Strings(listing[1:])
  			return fmt.Sprintf("Here're the files and directories up to 2 levels deep in %s, excluding hidden items and node_modules:\n%s", cmd.Path, strings.Join(listing, "\n"))
  		}
  		return fmt.Sprintf("The path %s does not exist. Please provide a valid path.", cmd.Path)
  	case "create":
  		store[cmd.Path] = cmd.FileText
  		return "File created successfully at: " + cmd.Path
  	case "str_replace":
  		content, ok := store[cmd.Path]
  		if !ok || !strings.Contains(content, cmd.OldStr) {
  			return fmt.Sprintf("No replacement was performed, old_str `%s` did not appear verbatim in %s.", cmd.OldStr, cmd.Path)
  		}
  		store[cmd.Path] = strings.Replace(content, cmd.OldStr, cmd.NewStr, 1)
  		return "The memory file has been edited."
  	case "insert":
  		content, ok := store[cmd.Path]
  		if !ok {
  			return fmt.Sprintf("Error: The path %s does not exist", cmd.Path)
  		}
  		lines := strings.Split(content, "\n")
  		if cmd.InsertLine < 0 || cmd.InsertLine > len(lines) {
  			return fmt.Sprintf("Error: Invalid `insert_line` parameter: %d. It should be within the range of lines of the file: [0, %d]", cmd.InsertLine, len(lines))
  		}
  		lines = slices.Insert(lines, cmd.InsertLine, strings.TrimSuffix(cmd.InsertText, "\n"))
  		store[cmd.Path] = strings.Join(lines, "\n")
  		return fmt.Sprintf("The file %s has been edited.", cmd.Path)
  	case "delete":
  		if _, ok := store[cmd.Path]; !ok {
  			return fmt.Sprintf("Error: The path %s does not exist", cmd.Path)
  		}
  		delete(store, cmd.Path)
  		return "Successfully deleted " + cmd.Path
  	case "rename":
  		if _, ok := store[cmd.OldPath]; !ok {
  			return fmt.Sprintf("Error: The path %s does not exist", cmd.OldPath)
  		}
  		if _, ok := store[cmd.NewPath]; ok {
  			return fmt.Sprintf("Error: The destination %s already exists", cmd.NewPath)
  		}
  		store[cmd.NewPath] = store[cmd.OldPath]
  		delete(store, cmd.OldPath)
  		return fmt.Sprintf("Successfully renamed %s to %s", cmd.OldPath, cmd.NewPath)
  	default:
  		return "Error: unknown command " + cmd.Command
  	}
  }

  func main() {
  	client := anthropic.NewClient()
  	tools := []anthropic.ToolUnionParam{{OfMemoryTool20250818: &anthropic.MemoryTool20250818Param{}}}
  	messages := []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Remember that customer Acme Corp prefers email follow-ups.")),
  	}

  	for {
  		message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 1024,
  			Messages:  messages,
  			Tools:     tools,
  		})
  		if err != nil {
  			log.Fatal(err)
  		}
  		if message.StopReason != anthropic.StopReasonToolUse {
  			for _, block := range message.Content {
  				if block.Type == "text" {
  					fmt.Println(block.Text)
  				}
  			}
  			break
  		}
  		results := []anthropic.ContentBlockParamUnion{}
  		for _, block := range message.Content {
  			if block.Type == "tool_use" {
  				results = append(results, anthropic.NewToolResultBlock(block.ID, executeMemory(block.Input), false))
  			}
  		}
  		messages = append(messages, message.ToParam(), anthropic.NewUserMessage(results...))
  	}
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.helpers.BetaMemoryToolHandler;
  import com.anthropic.helpers.BetaToolRunner;
  import com.anthropic.models.beta.messages.BetaMemoryTool20250818;
  import com.anthropic.models.beta.messages.BetaMessage;
  import com.anthropic.models.beta.messages.MessageCreateParams; // beta package, not models.messages
  import com.anthropic.models.beta.messages.ToolRunnerCreateParams;
  import com.anthropic.models.messages.Model;
  import java.nio.file.Path;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      // 您对六个记忆命令的 BetaMemoryToolHandler 实现
      BetaMemoryToolHandler handler = new FileSystemMemoryToolHandler(Path.of("memories"));

      MessageCreateParams createParams = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(BetaMemoryTool20250818.builder().build())
          .addUserMessage("Remember that customer Acme Corp prefers email follow-ups.")
          .build();

      ToolRunnerCreateParams runnerParams = ToolRunnerCreateParams.builder()
          .betaMemoryToolHandler(handler)
          .initialMessageParams(createParams)
          .maxIterations(10)
          .build();

      BetaToolRunner runner = client.beta().messages().toolRunner(runnerParams);
      for (BetaMessage message : runner) {
          IO.println(message);
      }
  }
  ```

  ```php PHP
  <?php

  use Anthropic\Beta\Messages\BetaMemoryTool20250818;
  use Anthropic\Client;
  use Anthropic\Lib\Tools\BetaRunnableTool;
  use Anthropic\Messages\Model;

  $client = new Client();

  // 一个内存存储，将记忆文件路径映射到其内容。
  // 在生产环境中请使用您自己的存储。
  $store = [];

  $memory = new BetaRunnableTool(
      definition: new BetaMemoryTool20250818(),
      run: function (array $input) use (&$store): string {
          $path = $input['path'] ?? '';
          switch ($input['command']) {
              case 'view':
                  if (isset($store[$path])) {
                      $numbered = [];
                      foreach (explode("\n", preg_replace('/\n\z/', '', $store[$path])) as $i => $line) {
                          $numbered[] = sprintf("%6d\t%s", $i + 1, $line);
                      }
                      return "Here's the content of {$path} with line numbers:\n" . implode("\n", $numbered);
                  }
                  if ($path === '/memories') {
                      $listing = ["1.0K\t/memories"];
                      foreach (array_keys($store) as $stored) {
                          $listing[] = "1.0K\t{$stored}";
                      }
                      return "Here're the files and directories up to 2 levels deep in {$path}, excluding hidden items and node_modules:\n" . implode("\n", $listing);
                  }
                  return "The path {$path} does not exist. Please provide a valid path.";
              case 'create':
                  $store[$path] = $input['file_text'];
                  return "File created successfully at: {$path}";
              case 'str_replace':
                  $position = strpos($store[$path] ?? '', $input['old_str']);
                  if ($position === false) {
                      return "No replacement was performed, old_str `{$input['old_str']}` did not appear verbatim in {$path}.";
                  }
                  $store[$path] = substr_replace($store[$path], $input['new_str'] ?? '', $position, strlen($input['old_str']));
                  return 'The memory file has been edited.';
              case 'insert':
                  if (!isset($store[$path])) {
                      return "Error: The path {$path} does not exist";
                  }
                  $lines = explode("\n", $store[$path]);
                  if ($input['insert_line'] < 0 || $input['insert_line'] > count($lines)) {
                      return "Error: Invalid `insert_line` parameter: {$input['insert_line']}. It should be within the range of lines of the file: [0, " . count($lines) . ']';
                  }
                  array_splice($lines, $input['insert_line'], 0, [preg_replace('/\n\z/', '', $input['insert_text'])]);
                  $store[$path] = implode("\n", $lines);
                  return "The file {$path} has been edited.";
              case 'delete':
                  if (!isset($store[$path])) {
                      return "Error: The path {$path} does not exist";
                  }
                  unset($store[$path]);
                  return "Successfully deleted {$path}";
              case 'rename':
                  if (!isset($store[$input['old_path']])) {
                      return "Error: The path {$input['old_path']} does not exist";
                  }
                  if (isset($store[$input['new_path']])) {
                      return "Error: The destination {$input['new_path']} already exists";
                  }
                  $store[$input['new_path']] = $store[$input['old_path']];
                  unset($store[$input['old_path']]);
                  return "Successfully renamed {$input['old_path']} to {$input['new_path']}";
              default:
                  return "Error: unknown command {$input['command']}";
          }
      },
  );

  $runner = $client->beta->messages->toolRunner(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Remember that customer Acme Corp prefers email follow-ups.']],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [$memory],
      maxIterations: 10,
  );

  $finalMessage = $runner->runUntilDone();
  print_r($finalMessage->content);
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::Client.new
  TOOLS = [{type: "memory_20250818", name: "memory"}].freeze

  # 一个内存存储，用于将记忆文件路径映射到其内容。
  # 在生产环境中请使用您自己的存储。
  STORE = {}

  def execute_memory(input)
    path = input[:path]
    case input[:command]
    when "view"
      if STORE.key?(path)
        lines = STORE[path].chomp.split("\n", -1)
        lines = [""] if lines.empty?
        numbered = lines.each_with_index.map { |line, i| format("%6d\t%s", i + 1, line) }
        "Here's the content of #{path} with line numbers:\n#{numbered.join("\n")}"
      elsif path == "/memories"
        listing = ["1.0K\t/memories"] + STORE.keys.map { |stored| "1.0K\t#{stored}" }
        "Here're the files and directories up to 2 levels deep in #{path}, excluding hidden items and node_modules:\n#{listing.join("\n")}"
      else
        "The path #{path} does not exist. Please provide a valid path."
      end
    when "create"
      STORE[path] = input[:file_text]
      "File created successfully at: #{path}"
    when "str_replace"
      unless STORE.key?(path) && STORE[path].include?(input[:old_str])
        return "No replacement was performed, old_str `#{input[:old_str]}` did not appear verbatim in #{path}."
      end
      STORE[path] = STORE[path].sub(input[:old_str]) { input[:new_str].to_s }
      "The memory file has been edited."
    when "insert"
      return "Error: The path #{path} does not exist" unless STORE.key?(path)
      lines = STORE[path].split("\n", -1)
      lines = [""] if lines.empty?
      if input[:insert_line] < 0 || input[:insert_line] > lines.length
        return "Error: Invalid `insert_line` parameter: #{input[:insert_line]}. It should be within the range of lines of the file: [0, #{lines.length}]"
      end
      lines.insert(input[:insert_line], input[:insert_text].chomp)
      STORE[path] = lines.join("\n")
      "The file #{path} has been edited."
    when "delete"
      return "Error: The path #{path} does not exist" unless STORE.key?(path)
      STORE.delete(path)
      "Successfully deleted #{path}"
    when "rename"
      return "Error: The path #{input[:old_path]} does not exist" unless STORE.key?(input[:old_path])
      return "Error: The destination #{input[:new_path]} already exists" if STORE.key?(input[:new_path])
      STORE[input[:new_path]] = STORE.delete(input[:old_path])
      "Successfully renamed #{input[:old_path]} to #{input[:new_path]}"
    else
      "Error: unknown command #{input[:command]}"
    end
  end

  messages = [{role: "user", content: "Remember that customer Acme Corp prefers email follow-ups."}]
  loop do
    message = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      messages: messages,
      tools: TOOLS
    )
    unless message.stop_reason == :tool_use
      puts message.content
      break
    end
    tool_results = message.content.filter_map do |block|
      next unless block.type == :tool_use
      {type: "tool_result", tool_use_id: block.id, content: execute_memory(block.input)}
    end
    messages << {role: "assistant", content: message.content} << {role: "user", content: tool_results}
  end
  ```
</CodeGroup>

Go、PHP 和 Ruby 示例中的内存存储使这些示例保持自包含：每个存储都根据 `tool_use` 块的 `input` 中的 `command` 字段进行分派，并返回[工具命令](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool#tool-commands)中描述的字符串。生产环境中的处理程序还需要这些演示存储所省略的[路径验证](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool#path-traversal-protection)。有关 SDK 自身的完整示例，请参阅：

* Python：[examples/memory/basic.py](https://github.com/anthropics/anthropic-sdk-python/blob/main/examples/memory/basic.py)
* TypeScript：[examples/tools-helpers-memory.ts](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/examples/tools-helpers-memory.ts)
* C#：[MemoryToolExample](https://github.com/anthropics/anthropic-sdk-csharp/tree/main/examples/MemoryToolExample)
* Java：[BetaMemoryToolExample.java](https://github.com/anthropics/anthropic-sdk-java/blob/main/anthropic-java-example/src/main/java/com/anthropic/example/BetaMemoryToolExample.java)

## 工具命令

您的客户端实现必须处理以下命令。这些规范描述了推荐的行为和返回字符串：Claude 会读取您的工具结果中包含的任何文本，因此如果您的应用程序需要，您可以返回不同的字符串。

### view

显示目录内容或文件内容，可选择指定行范围：

```json
{
  "command": "view",
  "path": "/memories/notes.txt",
  "view_range": [1, 10]
}
```

`view_range` 是可选的，适用于文本文件视图：`[start_line, end_line]` 返回这些行，`[start_line, -1]` 返回从 `start_line` 到文件末尾的所有内容。

#### 返回值

**对于目录：** 返回一个列表，显示文件和目录及其大小：

```text
Here're the files and directories up to 2 levels deep in {path}, excluding hidden items and node_modules:
{size}\t{path}
{size}\t{path}/{filename1}
{size}\t{path}/{filename2}
```

* 列出最多 2 层深度的文件
* 显示人类可读的大小（例如 `5.5K`、`1.2M`）
* 排除隐藏项（以 `.` 开头的文件）和 `node_modules`
* 在大小和路径之间使用制表符

在空存储上首次对 `/memories` 执行 `view` 不是错误。如果您的 SDK 提供了本地文件系统记忆工具 `BetaLocalFilesystemMemoryTool`，它会在 Claude 首次调用之前创建记忆根目录，并返回列表标题，后跟空目录本身的一行大小和路径。

**对于文件：** 返回带有标题和行号的文件内容：

```text wrap
Here's the content of {path} with line numbers:
{line_numbers}{tab}{content}
```

行号格式：

* **宽度：** 6 个字符，右对齐，用空格填充
* **分隔符：** 行号和内容之间使用制表符
* **索引：** 从 1 开始（第一行是第 1 行）
* **行数限制：** 超过 999,999 行的文件应返回错误：`"File {path} exceeds maximum line limit of 999,999 lines."`

**示例输出：**

```text
Here's the content of /memories/notes.txt with line numbers:
     1	Hello World
     2	This is line two
    10	Line ten
   100	Line one hundred
```

Claude 的工具描述还说明，`view` 会显示图像文件（`.jpg`、`.jpeg` 和 `.png`），并会截断超过 16,000 个字符的文件的文本视图。因此请预期会有针对图像路径的 `view` 调用，以及针对长文件的后续指定范围的查看。

#### 错误处理

* **文件或目录不存在：** `"The path {path} does not exist. Please provide a valid path."`

### create

创建新文件：

```json
{
  "command": "create",
  "path": "/memories/notes.txt",
  "file_text": "Meeting notes:\n- Discussed project timeline\n- Next steps defined\n"
}
```

#### 返回值

* **成功：** `"File created successfully at: {path}"`

#### 错误处理

* **文件已存在：** `"Error: File {path} already exists"`

Claude 的工具描述称 `create` 会"创建或覆盖"文件，因此请预期会有针对已存在路径的 `create` 调用。返回错误是参考行为，而改为覆盖也是一种有效的实现选择。

### str\_replace

替换文件中的文本：

```json
{
  "command": "str_replace",
  "path": "/memories/preferences.txt",
  "old_str": "Favorite color: blue",
  "new_str": "Favorite color: green"
}
```

对于 `str_replace`，`new_str` 是可选的：省略时，`old_str` 会被删除而不进行替换。

#### 返回值

* **成功：** `"The memory file has been edited."`，后跟带有行号的已编辑文件片段

#### 错误处理

* **文件不存在：** `"Error: The path {path} does not exist. Please provide a valid path."`
* **未找到文本：** ``"No replacement was performed, old_str `\{old_str}` did not appear verbatim in {path}."``
* **文本重复：** 当 `old_str` 出现多次时，返回：``"No replacement was performed. Multiple occurrences of old_str `\{old_str}` in lines: {line_numbers}. Please ensure it is unique"``

#### 目录处理

如果路径是目录，则返回"文件不存在"错误。

### insert

在指定行插入文本：

```json
{
  "command": "insert",
  "path": "/memories/todo.txt",
  "insert_line": 2,
  "insert_text": "- Review memory tool documentation\n"
}
```

`insert_text` 会插入到第 `insert_line` 行之后，`0` 表示插入到文件开头。

#### 返回值

* **成功：** `"The file {path} has been edited."`

#### 错误处理

* **文件不存在：** `"Error: The path {path} does not exist"`
* **行号无效：** ``"Error: Invalid `insert_line` parameter: {insert_line}. It should be within the range of lines of the file: [0, {n_lines}]"``

#### 目录处理

如果路径是目录，则返回"文件不存在"错误。

### delete

删除文件或目录：

```json
{
  "command": "delete",
  "path": "/memories/old_file.txt"
}
```

#### 返回值

* **成功：** `"Successfully deleted {path}"`

#### 错误处理

* **文件或目录不存在：** `"Error: The path {path} does not exist"`

#### 目录处理

递归删除目录及其所有内容。工具描述告知 Claude 它不能删除 `/memories` 目录本身，因此请拒绝路径为记忆根目录的 `delete`。

### rename

重命名或移动文件或目录：

```json
{
  "command": "rename",
  "old_path": "/memories/draft.txt",
  "new_path": "/memories/final.txt"
}
```

#### 返回值

* **成功：** `"Successfully renamed {old_path} to {new_path}"`

#### 错误处理

* **源不存在：** `"Error: The path {old_path} does not exist"`
* **目标已存在：** 返回错误（不要覆盖）：`"Error: The destination {new_path} already exists"`

#### 目录处理

重命名目录。工具描述告知 Claude 它不能重命名 `/memories` 目录本身，因此请拒绝 `old_path` 为记忆根目录的 `rename`。

## 提示指导

当记忆工具出现在您请求的 `tools` 中时，API 会自动将以下指令添加到 “system prompt”（系统提示）中。您无需自己发送：

```text wrap
IMPORTANT: ALWAYS VIEW YOUR MEMORY DIRECTORY BEFORE DOING ANYTHING ELSE.
MEMORY PROTOCOL:
1. Use the `view` command of your `memory` tool to check for earlier progress.
2. ... (work on the task) ...
   - As you make progress, record status / progress / thoughts etc in your memory.
ASSUME INTERRUPTION: Your context window might be reset at any moment, so you risk losing any progress that is not recorded in your memory directory.
```

Claude 的工具描述已经告知它要保持记忆目录井然有序，因此您无需重复该指令。如果 Claude 仍然创建了杂乱的记忆文件，您可以在提示中加以强调：

```text wrap
Note: when editing your memory folder, always try to keep its content up-to-date, coherent and organized. You can rename or delete files that are no longer relevant. Do not create new files unless necessary.
```

您还可以指导 Claude 向记忆中写入的内容。例如："Only write down information relevant to \<topic> in your memory system."

## 安全注意事项

您的应用程序会执行 Claude 请求的每个文件操作，因此以下防护措施由您负责：

### 敏感信息

Claude 通常会拒绝将敏感信息写入记忆文件。为了获得更强的保证，请添加验证，在处理程序写入文件之前剥离敏感数据。

### 文件存储大小

跟踪记忆文件的大小，并限制文件可以增长到的大小。考虑限制 `view` 命令返回的字符数，并让 Claude 使用 `view_range` 分页浏览其余内容。

### 记忆过期

定期删除长时间未被访问的记忆文件。

### 路径遍历防护

<Warning>
  诸如 `/memories/../../secrets.env` 之类的恶意路径可以访问 `/memories` 目录之外的文件。您的实现必须验证每个命令中的每个路径，以防止目录遍历攻击。
</Warning>

请考虑以下防护措施：

* 验证所有路径都以 `/memories` 开头
* 将路径解析为规范形式，并验证它们仍位于记忆目录内
* 拒绝包含 `../`、`..\\` 等序列或其他遍历模式的路径
* 注意 URL 编码的遍历序列（`%2e%2e%2f`）
* 使用您所用语言的内置路径安全工具（例如 Python 的 `pathlib.Path.resolve()` 和 `relative_to()`）

## 错误处理

记忆工具使用与[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool#handle-errors)类似的错误处理模式。每个命令的错误消息都列在[工具命令](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool#tool-commands)下。要向 Claude 返回错误，请在工具结果上将 `is_error` 设置为 `true`，并将消息放入 `content` 中：

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01C4D5E6F7G8H9I0J1K2L3M4",
  "content": "Error: The path /memories/notes.txt does not exist",
  "is_error": true
}
```

## 与上下文编辑集成

记忆工具可与 “context editing”（上下文编辑）配合使用来管理长时间运行的对话。有关详细信息，请参阅[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)。

## 与压缩一起使用

记忆工具还可以与 [compaction（压缩）](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)配合使用，压缩会在服务器端总结较早的对话上下文。上下文编辑在客户端清除特定的工具结果。压缩则会在对话接近上下文窗口限制时，在服务器上自动总结整个对话。

对于长时间运行的智能体，请考虑同时使用两者：压缩无需客户端记录即可使活动上下文保持较小，而记忆则保留必须在总结后依然存在的信息。

## 多会话软件开发模式

对于跨越多个智能体会话的软件项目，请有意识地设置记忆文件，而不是在工作进展过程中临时编写。以下模式将记忆转变为一种恢复机制：每个新会话都从上一个会话记录的状态继续。

### 该模式的工作方式

1. **初始化会话：** 第一个会话在任何实质性工作开始之前设置记忆文件。这包括进度日志（跟踪已完成的工作和接下来的工作）、功能清单（定义工作范围），以及对项目所需的任何启动或初始化脚本的引用。

2. **后续会话：** 每个新会话都以读取这些记忆文件开始。这样可以恢复项目状态，而无需重新探索代码库或回溯先前的决策。

3. **会话结束更新：** 在会话结束之前，它会用已完成的内容和剩余的内容更新进度日志。这确保下一个会话拥有准确的起点。

### 关键原则

一次只处理一个功能。只有在端到端验证确认功能正常工作后，才将其标记为完成，而不是在代码编写完成时。这可以使进度日志在各个会话之间保持准确。

<Tip>
  有关此模式实际应用的详细案例研究，包括初始化脚本、进度文件结构和基于 Git 的恢复，请参阅[适用于长时间运行智能体的有效框架](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)。
</Tip>

## 后续步骤

<CardGroup cols={2}>
  <Card title="Bash 工具" icon="terminal" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool">
    在持久的 bash 会话中运行 shell 命令。
  </Card>

  <Card title="上下文编辑" icon="edit" href="https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing">
    通过上下文编辑，在对话上下文增长时自动管理它。
  </Card>

  <Card title="压缩" icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/compaction">
    服务器端上下文压缩，用于管理接近上下文窗口限制的长对话。
  </Card>

  <Card title="工具参考" icon="book" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    Anthropic 提供的工具目录以及可选工具定义属性的参考。
  </Card>
</CardGroup>

## 可用性

记忆工具在所有模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 模型                                                             |
| -- | -------------------------------------------------------------- |
| 可用 | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格  |
| 详细信息 | 客户端记忆存储，数据保留由您控制。                                                                                                                         | —     |

适用的端点：

* `/v1/messages` 使用 `memory` 工具
