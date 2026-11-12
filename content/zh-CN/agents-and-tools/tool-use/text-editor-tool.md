---
title: 文本编辑器工具
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool
description: 为 Claude 提供 Anthropic 定义的文本编辑器工具，以查看、创建和编辑文件，并处理其 view、str_replace、create 和 insert 命令。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). 在您的环境中运行的客户端工具。
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
---

Claude 可以使用 Anthropic 架构的文本编辑器工具来查看和修改文本文件，帮助您调试、修复和改进代码或其他文本文档。这使 Claude 能够直接与您的文件交互，提供实际的帮助，而不仅仅是建议更改。

有关模型支持，请参阅[工具参考](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference)。

## 何时使用文本编辑器工具

以下是一些使用文本编辑器工具的示例场景：

* **代码调试：** 让 Claude 识别并修复代码中的错误，从语法错误到逻辑问题。
* **代码重构：** 让 Claude 通过有针对性的编辑来改进代码结构、可读性和性能。
* **文档生成：** 让 Claude 为您的代码库添加文档字符串、注释或 README 文件。
* **测试创建：** 让 Claude 根据其对实现的分析为您的代码创建单元测试。

## 使用文本编辑器工具

通过 Messages API 向 Claude 提供文本编辑器工具（名为 `str_replace_based_edit_tool`）。

您可以选择指定 `max_characters` 参数，以在查看大文件时控制截断。

<Note>
  `max_characters` 仅与 `text_editor_20250728` 及更高版本的文本编辑器工具兼容。
</Note>

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
          "type": "text_editor_20250728",
          "name": "str_replace_based_edit_tool",
          "max_characters": 10000
        }
      ],
      "messages": [
        {
          "role": "user",
          "content": "There'\''s a syntax error in my primes.py file. Can you help me fix it?"
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --tool '{type: text_editor_20250728, name: str_replace_based_edit_tool, max_characters: 10000}' \
    --message '{role: user, content: There is a syntax error in my primes.py file. Can you help me fix it?}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[
          {
              "type": "text_editor_20250728",
              "name": "str_replace_based_edit_tool",
              "max_characters": 10000,
          }
      ],
      messages=[
          {
              "role": "user",
              "content": "There's a syntax error in my primes.py file. Can you help me fix it?",
          }
      ],
  )

  print(response)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  const response = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [
      {
        type: "text_editor_20250728",
        name: "str_replace_based_edit_tool",
        max_characters: 10000
      }
    ],
    messages: [
      {
        role: "user",
        content: "There's a syntax error in my primes.py file. Can you help me fix it?"
      }
    ]
  });

  console.log(response);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var response = await client.Messages.Create(
      new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Tools = [new ToolTextEditor20250728 { MaxCharacters = 10000 }],
          Messages =
          [
              new UserMessageParam
              {
                  Content = "There's a syntax error in my primes.py file. Can you help me fix it?",
              },
          ],
      }
  );

  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{
  		{OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{
  			MaxCharacters: anthropic.Int(10000),
  		}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("There's a syntax error in my primes.py file. Can you help me fix it?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.messages.ToolTextEditor20250728;
  // ...
  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      ToolTextEditor20250728 editorTool =
          ToolTextEditor20250728.builder()
              .maxCharacters(10000L)
              .build();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addTool(editorTool)
          .addUserMessage("There's a syntax error in my primes.py file. Can you help me fix it?")
          .build();

      Message message = client.messages().create(params);
      IO.println(message);
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      tools: [ToolTextEditor20250728::with(maxCharacters: 10000)],
      messages: [
          [
              'role' => 'user',
              'content' => "There's a syntax error in my primes.py file. Can you help me fix it?",
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
      {
        type: "text_editor_20250728",
        name: "str_replace_based_edit_tool",
        max_characters: 10000
      }
    ],
    messages: [
      {
        role: "user",
        content: "There's a syntax error in my primes.py file. Can you help me fix it?"
      }
    ]
  )

  puts response
  ```
</CodeGroup>

按以下方式使用文本编辑器工具：

<Steps>
  <Step title="向 Claude 提供文本编辑器工具和用户提示">
    * 在您的 API 请求中包含文本编辑器工具。
    * 提供一个可能需要检查或修改文件的用户提示，例如"你能修复我代码中的语法错误吗？"
  </Step>

  <Step title="Claude 使用该工具检查文件或目录">
    * Claude 评估需要查看的内容，并使用 `view` 命令检查文件内容或列出目录内容。
    * API 响应包含一个带有 `view` 命令的 `tool_use` 内容块。
  </Step>

  <Step title="执行 view 命令并返回结果">
    * 从 Claude 的工具使用请求中提取文件或目录路径。
    * 读取文件内容或列出目录内容。
    * 如果您在工具配置中指定了 `max_characters` 参数，请将文件内容截断到该长度。
    * 通过继续对话并发送一条包含 `tool_result` 内容块的新 `user` 消息，将结果返回给 Claude。
  </Step>

  <Step title="Claude 使用该工具修改文件">
    * 在检查文件或目录后，Claude 可能会使用 `str_replace` 等命令进行更改，或使用 `insert` 在特定行号处添加文本。
    * 如果 Claude 使用 `str_replace` 命令，Claude 会构造一个格式正确的工具使用请求，其中包含旧文本和用于替换的新文本。
  </Step>

  <Step title="执行编辑并返回结果">
    * 从 Claude 的工具使用请求中提取文件路径、旧文本和新文本。
    * 在文件中执行文本替换。
    * 将结果返回给 Claude。
  </Step>

  <Step title="Claude 提供分析和解释">
    * 在检查并可能编辑文件后，Claude 会完整解释它发现了什么以及做了哪些更改。
  </Step>
</Steps>

### 文本编辑器工具命令

文本编辑器工具支持多个用于查看和修改文件的命令：

#### view

`view` 命令允许 Claude 检查文件内容或列出目录内容。它可以读取整个文件或特定范围的行。

参数：

* `command`：必须为 `view`
* `path`：要查看的文件或目录的路径
* `view_range`（可选）：由两个整数组成的数组，指定要查看的起始和结束行号。行号从 1 开始，结束行为 -1 表示读取到文件末尾。此参数仅适用于查看文件，不适用于目录。

<Accordion title="view 命令示例">
  查看文件的示例：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_01A09q90qw90lq917835lq9",
    "name": "str_replace_based_edit_tool",
    "input": {
      "command": "view",
      "path": "primes.py"
    }
  }
  ```

  查看目录的示例：

  ```json
  {
    "type": "tool_use",
    "id": "toolu_02B19r91rw91mr917835mr9",
    "name": "str_replace_based_edit_tool",
    "input": {
      "command": "view",
      "path": "src/"
    }
  }
  ```
</Accordion>

#### str\_replace

`str_replace` 命令允许 Claude 将文件中的特定字符串替换为新字符串。这用于进行精确编辑。

参数：

* `command`：必须为 `str_replace`
* `path`：要修改的文件的路径
* `old_str`：要替换的文本（必须完全匹配，包括空白和缩进）
* `new_str`：要插入以替换旧文本的新文本

<Accordion title="str_replace 命令示例">
  ```json
  {
    "type": "tool_use",
    "id": "toolu_01A09q90qw90lq917835lq9",
    "name": "str_replace_based_edit_tool",
    "input": {
      "command": "str_replace",
      "path": "primes.py",
      "old_str": "for num in range(2, limit + 1)",
      "new_str": "for num in range(2, limit + 1):"
    }
  }
  ```
</Accordion>

#### create

`create` 命令允许 Claude 使用指定内容创建新文件。

参数：

* `command`：必须为 `create`
* `path`：应创建新文件的路径
* `file_text`：要写入新文件的内容

<Accordion title="create 命令示例">
  ```json
  {
    "type": "tool_use",
    "id": "toolu_01A09q90qw90lq917835lq9",
    "name": "str_replace_based_edit_tool",
    "input": {
      "command": "create",
      "path": "test_primes.py",
      "file_text": "import unittest\nimport primes\n\nclass TestPrimes(unittest.TestCase):\n    def test_is_prime(self):\n        self.assertTrue(primes.is_prime(2))\n        self.assertTrue(primes.is_prime(3))\n        self.assertFalse(primes.is_prime(4))\n\nif __name__ == '__main__':\n    unittest.main()"
    }
  }
  ```
</Accordion>

#### insert

`insert` 命令允许 Claude 在文件中的特定位置插入文本。

参数：

* `command`：必须为 `insert`
* `path`：要修改的文件的路径
* `insert_line`：在其后插入文本的行号（0 表示文件开头）
* `insert_text`：要插入的文本

<Accordion title="insert 命令示例">
  ```json
  {
    "type": "tool_use",
    "id": "toolu_01A09q90qw90lq917835lq9",
    "name": "str_replace_based_edit_tool",
    "input": {
      "command": "insert",
      "path": "primes.py",
      "insert_line": 0,
      "insert_text": "\"\"\"Module for working with prime numbers.\n\nThis module provides functions to check if a number is prime\nand to generate a list of prime numbers up to a given limit.\n\"\"\"\n"
    }
  }
  ```
</Accordion>

### 示例：使用文本编辑器工具修复语法错误

此示例演示 Claude 如何使用文本编辑器工具修复 Python 文件中的语法错误。

首先，您的应用程序向 Claude 提供文本编辑器工具和修复语法错误的提示：

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
          "type": "text_editor_20250728",
          "name": "str_replace_based_edit_tool"
        }
      ],
      "messages": [
        {
          "role": "user",
          "content": "There'\''s a syntax error in my primes.py file. Can you help me fix it?"
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --tool '{type: text_editor_20250728, name: str_replace_based_edit_tool}' \
    --message '{role: user, content: There is a syntax error in my primes.py file. Can you help me fix it?}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}],
      messages=[
          {
              "role": "user",
              "content": "There's a syntax error in my primes.py file. Can you help me fix it?",
          }
      ],
  )

  print(response)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  const response = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [
      {
        type: "text_editor_20250728",
        name: "str_replace_based_edit_tool"
      }
    ],
    messages: [
      {
        role: "user",
        content: "There's a syntax error in my primes.py file. Can you help me fix it?"
      }
    ]
  });

  console.log(response);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var response = await client.Messages.Create(
      new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Tools = [new ToolTextEditor20250728()],
          Messages =
          [
              new UserMessageParam
              {
                  Content = "There's a syntax error in my primes.py file. Can you help me fix it?",
              },
          ],
      }
  );

  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{
  		{OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("There's a syntax error in my primes.py file. Can you help me fix it?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  import com.anthropic.models.messages.ToolTextEditor20250728;
  // ...
  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      ToolTextEditor20250728 editorTool =
          ToolTextEditor20250728.builder().build();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addTool(editorTool)
          .addUserMessage("There's a syntax error in my primes.py file. Can you help me fix it?")
          .build();

      Message message = client.messages().create(params);
      IO.println(message);
  }
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      tools: [new ToolTextEditor20250728()],
      messages: [
          [
              'role' => 'user',
              'content' => "There's a syntax error in my primes.py file. Can you help me fix it?",
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
    tools: [{type: "text_editor_20250728", name: "str_replace_based_edit_tool"}],
    messages: [
      {
        role: "user",
        content: "There's a syntax error in my primes.py file. Can you help me fix it?"
      }
    ]
  )

  puts response
  ```
</CodeGroup>

Claude 首先使用文本编辑器工具查看文件：

```json Output
{
  "id": "msg_01XAbCDeFgHiJkLmNoPQrStU",
  "model": "claude-opus-5-5",
  "stop_reason": "tool_use",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue."
    },
    {
      "type": "tool_use",
      "id": "toolu_01AbCdEfGhIjKlMnOpQrStU",
      "name": "str_replace_based_edit_tool",
      "input": {
        "command": "view",
        "path": "primes.py"
      }
    }
  ]
}
```

然后，您的应用程序应读取该文件并将其内容返回给 Claude：

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
          "type": "text_editor_20250728",
          "name": "str_replace_based_edit_tool"
        }
      ],
      "messages": [
        {
          "role": "user",
          "content": "There'\''s a syntax error in my primes.py file. Can you help me fix it?"
        },
        {
          "role": "assistant",
          "content": [
            {
              "type": "text",
              "text": "I'\''ll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue."
            },
            {
              "type": "tool_use",
              "id": "toolu_01AbCdEfGhIjKlMnOpQrStU",
              "name": "str_replace_based_edit_tool",
              "input": {
                "command": "view",
                "path": "primes.py"
              }
            }
          ]
        },
        {
          "role": "user",
          "content": [
            {
              "type": "tool_result",
              "tool_use_id": "toolu_01AbCdEfGhIjKlMnOpQrStU",
              "content": "1: def is_prime(n):\n2:     \"\"\"Check if a number is prime.\"\"\"\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     \"\"\"Generate a list of prime numbers up to the given limit.\"\"\"\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     \"\"\"Main function to demonstrate prime number generation.\"\"\"\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f\"Prime numbers up to {limit}:\")\n29:     print(prime_list)\n30:     print(f\"Found {len(prime_list)} prime numbers.\")\n31: \n32: if __name__ == \"__main__\":\n33:     main()"
            }
          ]
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  tools:
    - type: text_editor_20250728
      name: str_replace_based_edit_tool
  messages:
    - role: user
      content: There's a syntax error in my primes.py file. Can you help me fix it?
    - role: assistant
      content:
        - type: text
          text: >-
            I'll help you fix the syntax error in your primes.py file. First,
            let me take a look at the file to identify the issue.
        - type: tool_use
          id: toolu_01AbCdEfGhIjKlMnOpQrStU
          name: str_replace_based_edit_tool
          input:
            command: view
            path: primes.py
    - role: user
      content:
        - type: tool_result
          tool_use_id: toolu_01AbCdEfGhIjKlMnOpQrStU
          content: |-
            1: def is_prime(n):
            2:     """Check if a number is prime."""
            3:     if n <= 1:
            4:         return False
            5:     if n <= 3:
            6:         return True
            7:     if n % 2 == 0 or n % 3 == 0:
            8:         return False
            9:     i = 5
            10:     while i * i <= n:
            11:         if n % i == 0 or n % (i + 2) == 0:
            12:             return False
            13:         i += 6
            14:     return True
            15:
            16: def get_primes(limit):
            17:     """Generate a list of prime numbers up to the given limit."""
            18:     primes = []
            19:     for num in range(2, limit + 1)
            20:         if is_prime(num):
            21:             primes.append(num)
            22:     return primes
            23:
            24: def main():
            25:     """Main function to demonstrate prime number generation."""
            26:     limit = 100
            27:     prime_list = get_primes(limit)
            28:     print(f"Prime numbers up to {limit}:")
            29:     print(prime_list)
            30:     print(f"Found {len(prime_list)} prime numbers.")
            31:
            32: if __name__ == "__main__":
            33:     main()
  YAML
  ```

  ```python Python
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}],
      messages=[
          {
              "role": "user",
              "content": "There's a syntax error in my primes.py file. Can you help me fix it?",
          },
          {
              "role": "assistant",
              "content": [
                  {
                      "type": "text",
                      "text": "I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue.",
                  },
                  {
                      "type": "tool_use",
                      "id": "toolu_01AbCdEfGhIjKlMnOpQrStU",
                      "name": "str_replace_based_edit_tool",
                      "input": {"command": "view", "path": "primes.py"},
                  },
              ],
          },
          {
              "role": "user",
              "content": [
                  {
                      "type": "tool_result",
                      "tool_use_id": "toolu_01AbCdEfGhIjKlMnOpQrStU",
                      "content": '1: def is_prime(n):\n2:     """Check if a number is prime."""\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     """Generate a list of prime numbers up to the given limit."""\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     """Main function to demonstrate prime number generation."""\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f"Prime numbers up to {limit}:")\n29:     print(prime_list)\n30:     print(f"Found {len(prime_list)} prime numbers.")\n31: \n32: if __name__ == "__main__":\n33:     main()',
                  }
              ],
          },
      ],
  )

  print(response)
  ```

  ```typescript TypeScript
  const anthropic = new Anthropic();

  const response = await anthropic.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [
      {
        type: "text_editor_20250728",
        name: "str_replace_based_edit_tool"
      }
    ],
    messages: [
      {
        role: "user",
        content: "There's a syntax error in my primes.py file. Can you help me fix it?"
      },
      {
        role: "assistant",
        content: [
          {
            type: "text",
            text: "I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue."
          },
          {
            type: "tool_use",
            id: "toolu_01AbCdEfGhIjKlMnOpQrStU",
            name: "str_replace_based_edit_tool",
            input: {
              command: "view",
              path: "primes.py"
            }
          }
        ]
      },
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: "toolu_01AbCdEfGhIjKlMnOpQrStU",
            content:
              '1: def is_prime(n):\n2:     """Check if a number is prime."""\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     """Generate a list of prime numbers up to the given limit."""\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     """Main function to demonstrate prime number generation."""\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f"Prime numbers up to {limit}:")\n29:     print(prime_list)\n30:     print(f"Found {len(prime_list)} prime numbers.")\n31: \n32: if __name__ == "__main__":\n33:     main()'
          }
        ]
      }
    ]
  });

  console.log(response);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var response = await client.Messages.Create(
      new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Tools = [new ToolTextEditor20250728()],
          Messages =
          [
              new UserMessageParam
              {
                  Content = "There's a syntax error in my primes.py file. Can you help me fix it?",
              },
              new AssistantMessageParam
              {
                  Content = new List<ContentBlockParam>
                  {
                      new ContentBlockParam(new TextBlockParam()
                      {
                          Text = "I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue.",
                      }),
                      new ContentBlockParam(new ToolUseBlockParam()
                      {
                          ID = "toolu_01AbCdEfGhIjKlMnOpQrStU",
                          Name = "str_replace_based_edit_tool",
                          Input = new Dictionary<string, JsonElement>
                          {
                              ["command"] = JsonSerializer.SerializeToElement("view"),
                              ["path"] = JsonSerializer.SerializeToElement("primes.py"),
                          },
                      }),
                  },
              },
              new UserMessageParam
              {
                  Content = new List<ContentBlockParam>
                  {
                      new ContentBlockParam(new ToolResultBlockParam()
                      {
                          ToolUseID = "toolu_01AbCdEfGhIjKlMnOpQrStU",
                          Content = "1: def is_prime(n):\n2:     \"\"\"Check if a number is prime.\"\"\"\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     \"\"\"Generate a list of prime numbers up to the given limit.\"\"\"\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     \"\"\"Main function to demonstrate prime number generation.\"\"\"\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f\"Prime numbers up to {limit}:\")\n29:     print(prime_list)\n30:     print(f\"Found {len(prime_list)} prime numbers.\")\n31: \n32: if __name__ == \"__main__\":\n33:     main()",
                      }),
                  },
              },
          ],
      }
  );

  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{
  		{OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("There's a syntax error in my primes.py file. Can you help me fix it?")),
  		anthropic.NewAssistantMessage(
  			anthropic.NewTextBlock("I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue."),
  			anthropic.NewToolUseBlock(
  				"toolu_01AbCdEfGhIjKlMnOpQrStU",
  				map[string]any{"command": "view", "path": "primes.py"},
  				"str_replace_based_edit_tool",
  			),
  		),
  		anthropic.NewUserMessage(
  			anthropic.NewToolResultBlock(
  				"toolu_01AbCdEfGhIjKlMnOpQrStU",
  				"1: def is_prime(n):\n2:     \"\"\"Check if a number is prime.\"\"\"\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     \"\"\"Generate a list of prime numbers up to the given limit.\"\"\"\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     \"\"\"Main function to demonstrate prime number generation.\"\"\"\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f\"Prime numbers up to {limit}:\")\n29:     print(prime_list)\n30:     print(f\"Found {len(prime_list)} prime numbers.\")\n31: \n32: if __name__ == \"__main__\":\n33:     main()",
  				false,
  			),
  		),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .addTool(ToolTextEditor20250728.builder().build())
      .addUserMessage("There's a syntax error in my primes.py file. Can you help me fix it?")
      .addAssistantMessageOfContentBlockParams(
          List.of(
              ContentBlockParam.ofText(
                  TextBlockParam.builder()
                      .text("I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue.")
                      .build()
              ),
              ContentBlockParam.ofToolUse(
                  ToolUseBlockParam.builder()
                      .id("toolu_01AbCdEfGhIjKlMnOpQrStU")
                      .name("str_replace_based_edit_tool")
                      .input(
                          ToolUseBlockParam.Input.builder()
                              .putAdditionalProperty("command", JsonValue.from("view"))
                              .putAdditionalProperty("path", JsonValue.from("primes.py"))
                              .build()
                      )
                      .build()
              )
          )
      )
      .addUserMessageOfContentBlockParams(
          List.of(
              ContentBlockParam.ofToolResult(
                  ToolResultBlockParam.builder()
                      .toolUseId("toolu_01AbCdEfGhIjKlMnOpQrStU")
                      .content("1: def is_prime(n):\n2:     \"\"\"Check if a number is prime.\"\"\"\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     \"\"\"Generate a list of prime numbers up to the given limit.\"\"\"\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     \"\"\"Main function to demonstrate prime number generation.\"\"\"\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f\"Prime numbers up to {limit}:\")\n29:     print(prime_list)\n30:     print(f\"Found {len(prime_list)} prime numbers.\")\n31: \n32: if __name__ == \"__main__\":\n33:     main()")
                      .build()
              )
          )
      )
      .build();

  Message message = client.messages().create(params);
  System.out.println(message);
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      tools: [new ToolTextEditor20250728()],
      messages: [
          [
              'role' => 'user',
              'content' => "There's a syntax error in my primes.py file. Can you help me fix it?",
          ],
          [
              'role' => 'assistant',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => "I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue.",
                  ],
                  [
                      'type' => 'tool_use',
                      'id' => 'toolu_01AbCdEfGhIjKlMnOpQrStU',
                      'name' => 'str_replace_based_edit_tool',
                      'input' => ['command' => 'view', 'path' => 'primes.py'],
                  ],
              ],
          ],
          [
              'role' => 'user',
              'content' => [
                  [
                      'type' => 'tool_result',
                      'tool_use_id' => 'toolu_01AbCdEfGhIjKlMnOpQrStU',
                      'content' => "1: def is_prime(n):\n2:     \"\"\"Check if a number is prime.\"\"\"\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     \"\"\"Generate a list of prime numbers up to the given limit.\"\"\"\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     \"\"\"Main function to demonstrate prime number generation.\"\"\"\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f\"Prime numbers up to {limit}:\")\n29:     print(prime_list)\n30:     print(f\"Found {len(prime_list)} prime numbers.\")\n31: \n32: if __name__ == \"__main__\":\n33:     main()",
                  ],
              ],
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
    tools: [{type: "text_editor_20250728", name: "str_replace_based_edit_tool"}],
    messages: [
      {
        role: "user",
        content: "There's a syntax error in my primes.py file. Can you help me fix it?"
      },
      {
        role: "assistant",
        content: [
          {
            type: "text",
            text: "I'll help you fix the syntax error in your primes.py file. First, let me take a look at the file to identify the issue."
          },
          {
            type: "tool_use",
            id: "toolu_01AbCdEfGhIjKlMnOpQrStU",
            name: "str_replace_based_edit_tool",
            input: {command: "view", path: "primes.py"}
          }
        ]
      },
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: "toolu_01AbCdEfGhIjKlMnOpQrStU",
            content: "1: def is_prime(n):\n2:     \"\"\"Check if a number is prime.\"\"\"\n3:     if n <= 1:\n4:         return False\n5:     if n <= 3:\n6:         return True\n7:     if n % 2 == 0 or n % 3 == 0:\n8:         return False\n9:     i = 5\n10:     while i * i <= n:\n11:         if n % i == 0 or n % (i + 2) == 0:\n12:             return False\n13:         i += 6\n14:     return True\n15: \n16: def get_primes(limit):\n17:     \"\"\"Generate a list of prime numbers up to the given limit.\"\"\"\n18:     primes = []\n19:     for num in range(2, limit + 1)\n20:         if is_prime(num):\n21:             primes.append(num)\n22:     return primes\n23: \n24: def main():\n25:     \"\"\"Main function to demonstrate prime number generation.\"\"\"\n26:     limit = 100\n27:     prime_list = get_primes(limit)\n28:     print(f\"Prime numbers up to {limit}:\")\n29:     print(prime_list)\n30:     print(f\"Found {len(prime_list)} prime numbers.\")\n31: \n32: if __name__ == \"__main__\":\n33:     main()"
          }
        ]
      }
    ]
  )

  puts response
  ```
</CodeGroup>

<Tip>
  **行号**

  在前面的示例中，`view` 工具结果包含的文件内容在每行前面都加上了行号（例如，"1: def is\_prime(n):"）。行号不是必需的，但对于成功使用 `view_range` 参数检查文件的特定部分以及使用 `insert_line` 参数在精确位置添加内容至关重要。
</Tip>

Claude 识别出语法错误，并使用 `str_replace` 命令进行修复：

```json Output
{
  "id": "msg_01VwXyZAbCdEfGhIjKlMnO",
  "model": "claude-opus-5-5",
  "stop_reason": "tool_use",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you."
    },
    {
      "type": "tool_use",
      "id": "toolu_01PqRsTuVwXyZAbCdEfGh",
      "name": "str_replace_based_edit_tool",
      "input": {
        "command": "str_replace",
        "path": "primes.py",
        "old_str": "    for num in range(2, limit + 1)",
        "new_str": "    for num in range(2, limit + 1):"
      }
    }
  ]
}
```

然后，您的应用程序应执行编辑并返回结果：

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
          "type": "text_editor_20250728",
          "name": "str_replace_based_edit_tool"
        }
      ],
      "messages": [
        {
          "role": "assistant",
          "content": [
            {
              "type": "text",
              "text": "I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you."
            },
            {
              "type": "tool_use",
              "id": "toolu_01PqRsTuVwXyZAbCdEfGh",
              "name": "str_replace_based_edit_tool",
              "input": {
                "command": "str_replace",
                "path": "primes.py",
                "old_str": "    for num in range(2, limit + 1)",
                "new_str": "    for num in range(2, limit + 1):"
              }
            }
          ]
        },
        {
          "role": "user",
          "content": [
            {
              "type": "tool_result",
              "tool_use_id": "toolu_01PqRsTuVwXyZAbCdEfGh",
              "content": "Successfully replaced text at exactly one location."
            }
          ]
        }
      ]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  tools:
    - type: text_editor_20250728
      name: str_replace_based_edit_tool
  messages:
    # 之前的消息...
    - role: assistant
      content:
        - type: text
          text: >-
            I found the syntax error in your primes.py file. In the `get_primes`
            function, there is a missing colon (:) at the end of the for loop
            line. Let me fix that for you.
        - type: tool_use
          id: toolu_01PqRsTuVwXyZAbCdEfGh
          name: str_replace_based_edit_tool
          input:
            command: str_replace
            path: primes.py
            old_str: "    for num in range(2, limit + 1)"
            new_str: "    for num in range(2, limit + 1):"
    - role: user
      content:
        - type: tool_result
          tool_use_id: toolu_01PqRsTuVwXyZAbCdEfGh
          content: Successfully replaced text at exactly one location.
  YAML
  ```

  ```python Python
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}],
      messages=[
          # 之前的消息...
          {
              "role": "assistant",
              "content": [
                  {
                      "type": "text",
                      "text": "I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you.",
                  },
                  {
                      "type": "tool_use",
                      "id": "toolu_01PqRsTuVwXyZAbCdEfGh",
                      "name": "str_replace_based_edit_tool",
                      "input": {
                          "command": "str_replace",
                          "path": "primes.py",
                          "old_str": "    for num in range(2, limit + 1)",
                          "new_str": "    for num in range(2, limit + 1):",
                      },
                  },
              ],
          },
          {
              "role": "user",
              "content": [
                  {
                      "type": "tool_result",
                      "tool_use_id": "toolu_01PqRsTuVwXyZAbCdEfGh",
                      "content": "Successfully replaced text at exactly one location.",
                  }
              ],
          },
      ],
  )

  print(response)
  ```

  ```typescript TypeScript
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [
      {
        type: "text_editor_20250728",
        name: "str_replace_based_edit_tool"
      }
    ],
    messages: [
      // 之前的消息……
      {
        role: "assistant",
        content: [
          {
            type: "text",
            text: "I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you."
          },
          {
            type: "tool_use",
            id: "toolu_01PqRsTuVwXyZAbCdEfGh",
            name: "str_replace_based_edit_tool",
            input: {
              command: "str_replace",
              path: "primes.py",
              old_str: "    for num in range(2, limit + 1)",
              new_str: "    for num in range(2, limit + 1):"
            }
          }
        ]
      },
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: "toolu_01PqRsTuVwXyZAbCdEfGh",
            content: "Successfully replaced text at exactly one location."
          }
        ]
      }
    ]
  });

  console.log(response);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var response = await client.Messages.Create(
      new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Tools = [new ToolTextEditor20250728()],
          Messages =
          [
              // 之前的消息...
              new AssistantMessageParam
              {
                  Content = new List<ContentBlockParam>
                  {
                      new ContentBlockParam(new TextBlockParam()
                      {
                          Text = "I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you.",
                      }),
                      new ContentBlockParam(new ToolUseBlockParam()
                      {
                          ID = "toolu_01PqRsTuVwXyZAbCdEfGh",
                          Name = "str_replace_based_edit_tool",
                          Input = new Dictionary<string, JsonElement>
                          {
                              ["command"] = JsonSerializer.SerializeToElement("str_replace"),
                              ["path"] = JsonSerializer.SerializeToElement("primes.py"),
                              ["old_str"] = JsonSerializer.SerializeToElement("    for num in range(2, limit + 1)"),
                              ["new_str"] = JsonSerializer.SerializeToElement("    for num in range(2, limit + 1):"),
                          },
                      }),
                  },
              },
              new UserMessageParam
              {
                  Content = new List<ContentBlockParam>
                  {
                      new ContentBlockParam(new ToolResultBlockParam()
                      {
                          ToolUseID = "toolu_01PqRsTuVwXyZAbCdEfGh",
                          Content = "Successfully replaced text at exactly one location.",
                      }),
                  },
              },
          ],
      }
  );

  Console.WriteLine(response);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Tools: []anthropic.ToolUnionParam{
  		{OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{}},
  	},
  	Messages: []anthropic.MessageParam{
  		// 之前的消息...
  		anthropic.NewAssistantMessage(
  			anthropic.NewTextBlock("I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you."),
  			anthropic.NewToolUseBlock(
  				"toolu_01PqRsTuVwXyZAbCdEfGh",
  				map[string]any{
  					"command": "str_replace",
  					"path":    "primes.py",
  					"old_str": "    for num in range(2, limit + 1)",
  					"new_str": "    for num in range(2, limit + 1):",
  				},
  				"str_replace_based_edit_tool",
  			),
  		),
  		anthropic.NewUserMessage(
  			anthropic.NewToolResultBlock(
  				"toolu_01PqRsTuVwXyZAbCdEfGh",
  				"Successfully replaced text at exactly one location.",
  				false,
  			),
  		),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(response)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .addTool(ToolTextEditor20250728.builder().build())
      // 之前的消息会放在这里
      .addAssistantMessageOfContentBlockParams(
          List.of(
              ContentBlockParam.ofText(
                  TextBlockParam.builder()
                      .text(
                          "I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you."
                      )
                      .build()
              ),
              ContentBlockParam.ofToolUse(
                  ToolUseBlockParam.builder()
                      .id("toolu_01PqRsTuVwXyZAbCdEfGh")
                      .name("str_replace_based_edit_tool")
                      .input(
                          ToolUseBlockParam.Input.builder()
                              .putAdditionalProperty("command", JsonValue.from("str_replace"))
                              .putAdditionalProperty("path", JsonValue.from("primes.py"))
                              .putAdditionalProperty(
                                  "old_str",
                                  JsonValue.from("    for num in range(2, limit + 1)")
                              )
                              .putAdditionalProperty(
                                  "new_str",
                                  JsonValue.from("    for num in range(2, limit + 1):")
                              )
                              .build()
                      )
                      .build()
              )
          )
      )
      .addUserMessageOfContentBlockParams(
          List.of(
              ContentBlockParam.ofToolResult(
                  ToolResultBlockParam.builder()
                      .toolUseId("toolu_01PqRsTuVwXyZAbCdEfGh")
                      .content("Successfully replaced text at exactly one location.")
                      .build()
              )
          )
      )
      .build();

  Message message = client.messages().create(params);
  System.out.println(message);
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      tools: [new ToolTextEditor20250728()],
      messages: [
          // 之前的消息……
          [
              'role' => 'assistant',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => 'I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you.',
                  ],
                  [
                      'type' => 'tool_use',
                      'id' => 'toolu_01PqRsTuVwXyZAbCdEfGh',
                      'name' => 'str_replace_based_edit_tool',
                      'input' => [
                          'command' => 'str_replace',
                          'path' => 'primes.py',
                          'old_str' => '    for num in range(2, limit + 1)',
                          'new_str' => '    for num in range(2, limit + 1):',
                      ],
                  ],
              ],
          ],
          [
              'role' => 'user',
              'content' => [
                  [
                      'type' => 'tool_result',
                      'tool_use_id' => 'toolu_01PqRsTuVwXyZAbCdEfGh',
                      'content' => 'Successfully replaced text at exactly one location.',
                  ],
              ],
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
    tools: [{type: "text_editor_20250728", name: "str_replace_based_edit_tool"}],
    messages: [
      # 之前的消息……
      {
        role: "assistant",
        content: [
          {
            type: "text",
            text: "I found the syntax error in your primes.py file. In the `get_primes` function, there is a missing colon (:) at the end of the for loop line. Let me fix that for you."
          },
          {
            type: "tool_use",
            id: "toolu_01PqRsTuVwXyZAbCdEfGh",
            name: "str_replace_based_edit_tool",
            input: {
              command: "str_replace",
              path: "primes.py",
              old_str: "    for num in range(2, limit + 1)",
              new_str: "    for num in range(2, limit + 1):"
            }
          }
        ]
      },
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: "toolu_01PqRsTuVwXyZAbCdEfGh",
            content: "Successfully replaced text at exactly one location."
          }
        ]
      }
    ]
  )

  puts response
  ```
</CodeGroup>

最后，Claude 会对修复进行完整的解释：

````json Output
{
  "id": "msg_01IjKlMnOpQrStUvWxYzAb",
  "model": "claude-opus-5-5",
  "stop_reason": "end_turn",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "I've fixed the syntax error in your primes.py file. The issue was in the `get_primes` function at line 19. There was a missing colon (:) at the end of the for loop line.\n\nHere's what I changed:\n\nFrom:\n```python\nfor num in range(2, limit + 1)\n```\n\nTo:\n```python\nfor num in range(2, limit + 1):\n```\n\nIn Python, a colon is required at the end of statements that start a new code block, such as for loops, if statements, function definitions, etc. Without the colon, Python raises a syntax error because it doesn't know where the code block begins.\n\nYour code should now run correctly. The function will properly generate a list of prime numbers up to the specified limit."
    }
  ]
}
````

## 实现文本编辑器工具

文本编辑器工具以无架构（schema-less）工具的形式实现。使用此工具时，您无需像其他工具那样提供输入架构；该架构内置于 Claude 的模型中，无法修改。

对于 Claude 4 及更高版本的模型，工具类型为 `type: "text_editor_20250728"`。

<Steps>
  <Step title="初始化您的编辑器实现">
    创建辅助函数来处理文件操作，例如读取、写入和修改文件。考虑实现备份功能以便从错误中恢复。
  </Step>

  <Step title="处理编辑器工具调用">
    创建一个函数，根据命令类型处理来自 Claude 的工具调用：

    <CodeGroup exclude="shell">
      ```python Python
      def handle_editor_tool(tool_call):
          input_params = tool_call.input
          command = input_params.get("command", "")
          file_path = input_params.get("path", "")

          match command:
              case "view":
                  # 读取并返回文件内容
                  pass
              case "str_replace":
                  # 替换文件中的文本
                  pass
              case "create":
                  # 创建新文件
                  pass
              case "insert":
                  # 在指定位置插入文本
                  pass
      ```

      ```typescript TypeScript
      function handleEditorTool(toolCall: { input: { command?: string; path?: string } }): void {
        const inputParams = toolCall.input;
        const command = inputParams.command ?? "";
        const filePath = inputParams.path ?? "";

        switch (command) {
          case "view":
            // 读取并返回文件内容
            break;
          case "str_replace":
            // 替换文件中的文本
            break;
          case "create":
            // 创建新文件
            break;
          case "insert":
            // 在指定位置插入文本
            break;
        }
      }
      ```

      ```csharp C#
      static string HandleEditorTool(IReadOnlyDictionary<string, JsonElement> input)
      {
          input.TryGetValue("command", out var commandEl);
          input.TryGetValue("path", out var pathEl);
          var command = commandEl.ValueKind == JsonValueKind.String ? commandEl.GetString() : null;
          var filePath = pathEl.ValueKind == JsonValueKind.String ? pathEl.GetString() : null;

          if (command == "view")
          {
              // 读取并返回文件内容
          }
          else if (command == "str_replace")
          {
              // 替换文件中的文本
          }
          else if (command == "create")
          {
              // 创建新文件
          }
          else if (command == "insert")
          {
              // 在指定位置插入文本
          }
          return "";
      }
      ```

      ```go Go
      func handleEditorTool(input map[string]any) string {
      	command, _ := input["command"].(string)
      	filePath, _ := input["path"].(string)
      // ...

      	switch command {
      	case "view":
      		// 读取并返回文件内容
      	case "str_replace":
      		// 替换文件中的文本
      	case "create":
      		// 创建新文件
      	case "insert":
      		// 在指定位置插入文本
      	}
      	return ""
      }
      ```

      ```java Java
      static void handleEditorTool(Map<String, Object> input) {
          var command = (String) input.getOrDefault("command", "");
          var filePath = (String) input.getOrDefault("path", "");

          if (command.equals("view")) {
              // 读取并返回文件内容
          } else if (command.equals("str_replace")) {
              // 替换文件中的文本
          } else if (command.equals("create")) {
              // 创建新文件
          } else if (command.equals("insert")) {
              // 在指定位置插入文本
          }
      }
      ```

      ```php PHP
      function handle_editor_tool(array $input): string
      {
          $command = $input['command'] ?? '';
          $filePath = $input['path'] ?? '';

          if ($command === 'view') {
              // 读取并返回文件内容
          } elseif ($command === 'str_replace') {
              // 替换文件中的文本
          } elseif ($command === 'create') {
              // 创建新文件
          } elseif ($command === 'insert') {
              // 在指定位置插入文本
          }
          return '';
      }
      ```

      ```ruby Ruby
      def handle_editor_tool(input)
        command = input[:command] || ""
        file_path = input[:path] || ""

        case command
        when "view"
          # 读取并返回文件内容
        when "str_replace"
          # 替换文件中的文本
        when "create"
          # 创建新文件
        when "insert"
          # 在指定位置插入文本
        end
      end
      ```
    </CodeGroup>
  </Step>

  <Step title="实施安全措施">
    添加验证和安全检查：

    * 验证文件路径以防止目录遍历
    * 在进行更改前创建备份
    * 妥善处理错误
    * 实施权限检查
  </Step>

  <Step title="处理 Claude 的响应">
    从 Claude 的响应中提取并处理工具调用：

    <CodeGroup exclude="shell">
      ```python Python
      # 处理 Claude 响应中的工具使用
      for content in response.content:
          if content.type == "tool_use":
              # 根据命令运行工具
              result = handle_editor_tool(content)

              # 将结果返回给 Claude
              tool_result = {
                  "type": "tool_result",
                  "tool_use_id": content.id,
                  "content": result,
              }
      ```

      ```typescript TypeScript
      // 处理 Claude 响应中的工具使用
      for (const block of response.content) {
        if (block.type === "tool_use") {
          // 根据命令运行工具
          const result = handleEditorTool(block);

          // 将结果返回给 Claude
          const toolResult = {
            type: "tool_result",
            tool_use_id: block.id,
            content: result
          };
        }
      }
      ```

      ```csharp C#
      // 处理 Claude 响应中的 tool use（工具使用）
      foreach (var block in response.Content)
      {
          if (block.TryPickToolUse(out var toolUse))
          {
              var result = HandleEditorTool(toolUse.Input);
              var toolResult = new ToolResultBlockParam
              {
                  ToolUseID = toolUse.ID,
                  Content = result,
              };
          }
      }
      ```

      ```go Go
      // 处理 Claude 响应中的工具使用
      for _, block := range response.Content {
      	if block.Type == "tool_use" {
      		var input map[string]any
      		if err := json.Unmarshal(block.Input, &input); err != nil {
      			log.Fatal(err)
      		}
      		result := handleEditorTool(input)

      		toolResult := anthropic.NewToolResultBlock(block.ID, result, false)
      // ...
      	}
      }
      ```

      ```java Java
      // 处理 Claude 响应中的工具使用
      for (var block : response.content()) {
          if (block.type().equals("tool_use")) {
              // 根据命令运行工具
              var result = handleEditorTool(block);

              // 将结果返回给 Claude
              var toolResult = Map.of(
                  "type", "tool_result",
                  "tool_use_id", block.id(),
                  "content", result
              );
          }
      }
      ```

      ```php PHP
      // 处理 Claude 响应中的工具使用
      foreach ($response->content as $block) {
          if ($block->type === 'tool_use') {
              // 根据命令运行工具
              $result = handle_editor_tool($block->input);

              // 将结果返回给 Claude
              $toolResult = [
                  'type' => 'tool_result',
                  'tool_use_id' => $block->id,
                  'content' => $result,
              ];
          }
      }
      ```

      ```ruby Ruby
      # 处理 Claude 响应中的工具使用
      tool_results = response.content.filter_map do |block|
        next unless block.type == :tool_use

        {type: "tool_result", tool_use_id: block.id, content: handle_editor_tool(block.input)}
      end
      ```
    </CodeGroup>
  </Step>
</Steps>

<Warning>
  在实现文本编辑器工具时，请牢记：

  1. **安全性：** 该工具可以访问您的本地文件系统，因此请实施适当的安全措施。
  2. **备份：** 在允许编辑重要文件之前，始终创建备份。
  3. **验证：** 验证所有输入以防止意外更改。
  4. **唯一匹配：** 确保替换仅匹配一个位置，以避免意外编辑。
</Warning>

### 处理错误

使用文本编辑器工具时，可能会发生各种错误。以下是有关如何处理这些错误的指导：

<AccordionGroup>
  <Accordion title="文件未找到">
    如果 Claude 尝试查看或修改不存在的文件，请在 `tool_result` 中返回适当的错误消息：

    ```json
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": "Error: File not found",
          "is_error": true
        }
      ]
    }
    ```
  </Accordion>

  <Accordion title="替换存在多个匹配">
    如果 Claude 的 `str_replace` 命令匹配到文件中的多个位置，请返回适当的错误消息：

    ```json
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": "Error: Found 3 matches for replacement text. Please provide more context to make a unique match.",
          "is_error": true
        }
      ]
    }
    ```
  </Accordion>

  <Accordion title="替换没有匹配">
    如果 Claude 的 `str_replace` 命令未匹配到文件中的任何文本，请返回适当的错误消息：

    ```json
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": "Error: No match found for replacement. Please check your text and try again.",
          "is_error": true
        }
      ]
    }
    ```
  </Accordion>

  <Accordion title="权限错误">
    如果在创建、读取或修改文件时存在权限问题，请返回适当的错误消息：

    ```json
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": "Error: Permission denied. Cannot write to file.",
          "is_error": true
        }
      ]
    }
    ```
  </Accordion>
</AccordionGroup>

### 遵循实现最佳实践

<AccordionGroup>
  <Accordion title="提供清晰的上下文">
    在要求 Claude 修复或修改代码时，请具体说明要检查哪些文件或要解决哪些问题。清晰的上下文有助于 Claude 识别正确的文件并进行适当的更改。

    **不太有用的提示：** "你能修复我的代码吗？"

    **更好的提示：** "我的 primes.py 文件中有一个语法错误，导致它无法运行。你能修复它吗？"
  </Accordion>

  <Accordion title="明确指定文件路径">
    在需要时清楚地指定文件路径，尤其是在处理多个文件或位于不同目录中的文件时。

    **不太有用的提示：** "审查我的辅助文件"

    **更好的提示：** "你能检查一下我的 utils/helpers.py 文件是否存在性能问题吗？"
  </Accordion>

  <Accordion title="编辑前创建备份">
    在您的应用程序中实现一个备份系统，在允许 Claude 编辑文件之前创建文件副本，尤其是对于重要代码或生产代码。

    <CodeGroup exclude="shell">
      ```python Python
      def backup_file(file_path):
          """Create a backup of a file before editing."""
          backup_path = f"{file_path}.backup"
          if os.path.exists(file_path):
              with open(file_path, "r") as src, open(backup_path, "w") as dst:
                  dst.write(src.read())
      ```

      ```typescript TypeScript
      async function backupFile(filePath: string): Promise<void> {
        const backupPath = `${filePath}.backup`;
        try {
          await access(filePath);
          await copyFile(filePath, backupPath);
        } catch {
          // 文件不存在；无需备份
        }
      }
      ```

      ```csharp C#
      static void BackupFile(string filePath)
      {
          var backupPath = $"{filePath}.backup";
          if (File.Exists(filePath))
          {
              File.Copy(filePath, backupPath, overwrite: true);
          }
      }
      ```

      ```go Go
      func backupFile(filePath string) error {
      	backupPath := filePath + ".backup"
      	data, err := os.ReadFile(filePath)
      	if err != nil {
      		if os.IsNotExist(err) {
      			return nil
      		}
      		return err
      	}
      	return os.WriteFile(backupPath, data, 0o644)
      }
      ```

      ```java Java
      static void backupFile(String filePath) throws IOException {
          Path source = Path.of(filePath);
          Path backupPath = Path.of(filePath + ".backup");
          if (Files.exists(source)) {
              Files.copy(source, backupPath, StandardCopyOption.REPLACE_EXISTING);
          }
      }
      ```

      ```php PHP
      function backup_file(string $filePath): void
      {
          $backupPath = $filePath . '.backup';
          if (file_exists($filePath)) {
              copy($filePath, $backupPath);
          }
      }
      ```

      ```ruby Ruby
      def backup_file(file_path)
        backup_path = "#{file_path}.backup"
        FileUtils.cp(file_path, backup_path) if File.exist?(file_path)
      end
      ```
    </CodeGroup>
  </Accordion>

  <Accordion title="谨慎处理唯一文本替换">
    `str_replace` 命令要求被替换的文本完全匹配。您的应用程序应确保旧文本恰好有一个匹配，否则应提供适当的错误消息。

    <CodeGroup exclude="shell">
      ```python Python
      def safe_replace(file_path, old_text, new_text):
          """Replace text only if there's exactly one match."""
          with open(file_path, "r") as f:
              content = f.read()

          count = content.count(old_text)
          if count == 0:
              return "Error: No match found"
          elif count > 1:
              return f"Error: Found {count} matches"
          else:
              new_content = content.replace(old_text, new_text)
              with open(file_path, "w") as f:
                  f.write(new_content)
              return "Successfully replaced text"
      ```

      ```typescript TypeScript
      async function safeReplace(
        filePath: string,
        oldText: string,
        newText: string
      ): Promise<string> {
        const content = await readFile(filePath, "utf8");

        const count = content.split(oldText).length - 1;
        if (count === 0) {
          return "Error: No match found";
        } else if (count > 1) {
          return `Error: Found ${count} matches`;
        } else {
          const newContent = content.replace(oldText, newText);
          await writeFile(filePath, newContent, "utf8");
          return "Successfully replaced text";
        }
      }
      ```

      ```csharp C#
      static string SafeReplace(string filePath, string oldText, string newText)
      {
          var content = File.ReadAllText(filePath);

          var count = content.Split(oldText).Length - 1;
          if (count == 0)
          {
              return "Error: No match found";
          }
          else if (count > 1)
          {
              return $"Error: Found {count} matches";
          }
          else
          {
              var newContent = content.Replace(oldText, newText);
              File.WriteAllText(filePath, newContent);
              return "Successfully replaced text";
          }
      }
      ```

      ```go Go
      func safeReplace(filePath, oldText, newText string) string {
      	data, err := os.ReadFile(filePath)
      	if err != nil {
      		return fmt.Sprintf("Error: %v", err)
      	}
      	content := string(data)

      	count := strings.Count(content, oldText)
      	if count == 0 {
      		return "Error: No match found"
      	} else if count > 1 {
      		return fmt.Sprintf("Error: Found %d matches", count)
      	}

      	newContent := strings.Replace(content, oldText, newText, 1)
      	if err := os.WriteFile(filePath, []byte(newContent), 0o644); err != nil {
      		return fmt.Sprintf("Error: %v", err)
      	}
      	return "Successfully replaced text"
      }
      ```

      ```java Java
      static String safeReplace(String filePath, String oldText, String newText) throws IOException {
          String content = Files.readString(Path.of(filePath));

          int count = content.split(Pattern.quote(oldText), -1).length - 1;
          if (count == 0) {
              return "Error: No match found";
          } else if (count > 1) {
              return "Error: Found " + count + " matches";
          } else {
              String newContent = content.replace(oldText, newText);
              Files.writeString(Path.of(filePath), newContent);
              return "Successfully replaced text";
          }
      }
      ```

      ```php PHP
      function safe_replace(string $filePath, string $oldText, string $newText): string
      {
          $content = file_get_contents($filePath);

          $count = substr_count($content, $oldText);
          if ($count === 0) {
              return 'Error: No match found';
          } elseif ($count > 1) {
              return "Error: Found {$count} matches";
          } else {
              $newContent = str_replace($oldText, $newText, $content);
              file_put_contents($filePath, $newContent);
              return 'Successfully replaced text';
          }
      }
      ```

      ```ruby Ruby
      def safe_replace(file_path, old_text, new_text)
        content = File.read(file_path)

        count = content.scan(old_text).length
        if count == 0
          "Error: No match found"
        elsif count > 1
          "Error: Found #{count} matches"
        else
          new_content = content.sub(old_text) { new_text }
          File.write(file_path, new_content)
          "Successfully replaced text"
        end
      end
      ```
    </CodeGroup>
  </Accordion>

  <Accordion title="验证更改">
    在 Claude 对文件进行更改后，通过运行测试或检查代码是否仍按预期工作来验证更改。

    <CodeGroup exclude="shell">
      ```python Python
      def verify_changes(file_path):
          """Run tests or checks after making changes."""
          try:
              # 对于 Python 文件，检查是否存在语法错误
              if file_path.endswith(".py"):
                  import ast

                  with open(file_path, "r") as f:
                      ast.parse(f.read())
                  return "Syntax check passed"
          except Exception as e:
              return f"Verification failed: {str(e)}"
      ```

      ```typescript TypeScript
      function verifyChanges(filePath: string): string {
        try {
          // 对于 Python 文件，检查是否存在语法错误
          if (filePath.endsWith(".py")) {
            execFileSync("python3", ["-m", "py_compile", filePath]);
            return "Syntax check passed";
          }
          return "No checks defined for this file type";
        } catch (err) {
          return `Verification failed: ${err}`;
        }
      }
      ```

      ```csharp C#
      static string VerifyChanges(string filePath)
      {
          try
          {
              // 对于 Python 文件，检查是否存在语法错误
              if (filePath.EndsWith(".py"))
              {
                  var psi = new ProcessStartInfo("python3")
                  {
                      RedirectStandardError = true,
                  };
                  psi.ArgumentList.Add("-m");
                  psi.ArgumentList.Add("py_compile");
                  psi.ArgumentList.Add(filePath);
                  using var proc = Process.Start(psi)!;
                  proc.WaitForExit();
                  if (proc.ExitCode != 0)
                  {
                      return $"Verification failed: {proc.StandardError.ReadToEnd()}";
                  }
                  return "Syntax check passed";
              }
              return "No checks defined for this file type";
          }
          catch (Exception e)
          {
              return $"Verification failed: {e.Message}";
          }
      }
      ```

      ```go Go
      func verifyChanges(filePath string) string {
      	// 对于 Python 文件，检查是否存在语法错误
      	if strings.HasSuffix(filePath, ".py") {
      		cmd := exec.Command("python3", "-m", "py_compile", filePath)
      		if out, err := cmd.CombinedOutput(); err != nil {
      			return fmt.Sprintf("Verification failed: %v: %s", err, out)
      		}
      		return "Syntax check passed"
      	}
      	return "No checks defined for this file type"
      }
      ```

      ```java Java
      static String verifyChanges(String filePath) {
          try {
              // 对于 Python 文件，检查语法错误
              if (filePath.endsWith(".py")) {
                  Process proc = new ProcessBuilder("python3", "-m", "py_compile", filePath)
                      .redirectErrorStream(true)
                      .start();
                  if (proc.waitFor() != 0) {
                      return "Verification failed: " + new String(proc.getInputStream().readAllBytes());
                  }
                  return "Syntax check passed";
              }
              return "No checks defined for this file type";
          } catch (IOException | InterruptedException e) {
              return "Verification failed: " + e.getMessage();
          }
      }
      ```

      ```php PHP
      function verify_changes(string $filePath): string
      {
          // 对于 Python 文件，检查是否存在语法错误
          if (str_ends_with($filePath, '.py')) {
              exec('python3 -m py_compile ' . escapeshellarg($filePath) . ' 2>&1', $output, $exitCode);
              if ($exitCode !== 0) {
                  return 'Verification failed: ' . implode("\n", $output);
              }
              return 'Syntax check passed';
          }
          return 'No checks defined for this file type';
      }
      ```

      ```ruby Ruby
      def verify_changes(file_path)
        # 对于 Python 文件，检查是否存在语法错误
        if file_path.end_with?(".py")
          if system("python3", "-m", "py_compile", file_path)
            "Syntax check passed"
          else
            "Verification failed: syntax error in #{file_path}"
          end
        else
          "No checks defined for this file type"
        end
      end
      ```
    </CodeGroup>
  </Accordion>
</AccordionGroup>

***

## 定价和令牌使用

文本编辑器工具采用与 Claude 配合使用的其他工具相同的定价结构。它遵循基于您所使用的 Claude 模型的标准输入和输出令牌（token）定价。

除基础令牌外，文本编辑器工具还需要以下额外的输入令牌：

| 工具                                                                 | 额外输入令牌  |
| ------------------------------------------------------------------ | ------- |
| `text_editor_20250728`（Claude 4.7 及更高版本模型以及 Claude Mythos Preview） | 974 个令牌 |
| `text_editor_20250728`（Claude 4.6 及更早版本模型）                         | 745 个令牌 |

有关工具定价的更多详细信息，请参阅[工具使用定价](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview#pricing)。

## 将文本编辑器工具与其他工具集成

您可以将文本编辑器工具与其他 Claude 工具一起使用。组合使用工具时，请确保：

* 使工具版本与您使用的模型相匹配
* 考虑请求中包含的所有工具所产生的额外令牌使用量

## 更新日志

| 日期               | 版本                     | 变更                                                                                                                                                                                               |
| ---------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 2025 年 7 月 28 日  | `text_editor_20250728` | 发布更新版文本编辑器工具，修复了一些问题并新增可选的 `max_characters` 参数。除此之外，它与 `text_editor_20250429` 完全相同。                                                                                                              |
| 2025 年 4 月 29 日  | `text_editor_20250429` | 发布适用于 Claude 4 的文本编辑器工具。此版本移除了 `undo_edit` 命令，但保留了所有其他功能。工具名称已更新，以反映其基于 `str_replace` 的架构。                                                                                                       |
| 2025 年 3 月 13 日  | `text_editor_20250124` | 推出独立的文本编辑器工具文档。此版本针对 Claude Sonnet 3.7 进行了优化，但功能与之前的版本相同。                                                                                                                                        |
| 2024 年 10 月 22 日 | `text_editor_20241022` | 随 Claude Sonnet 3.5（已停用；请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)）首次发布文本编辑器工具。通过 `view`、`create`、`str_replace`、`insert` 和 `undo_edit` 命令提供查看、创建和编辑文件的功能。 |

## 后续步骤

以下是一些以更便捷、更强大的方式使用文本编辑器工具的思路：

* **集成到您的开发工作流中：** 将文本编辑器工具构建到您的开发工具或 IDE 中。
* **创建代码审查系统：** 让 Claude 审查您的代码并进行改进。
* **构建调试助手：** 创建一个系统，让 Claude 帮助您诊断和修复代码中的问题。
* **实现文件格式转换：** 让 Claude 帮助您将文件从一种格式转换为另一种格式。
* **自动化文档编写：** 设置工作流，让 Claude 自动为您的代码编写文档。

文本编辑器工具使 Claude 能够直接处理您的代码库，支持从调试到自动化文档编写的各种工作流。

<CardGroup cols={3}>
  <Card title="工具使用概述" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview">
    了解如何实现与 Claude 配合使用的工具工作流。
  </Card>

  <Card title="Bash 工具" icon="terminal" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool">
    使用 Claude 运行 shell 命令。
  </Card>
</CardGroup>

## 可用性

文本编辑器工具在所有模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格  |
| 详细信息 | 在您的环境中运行的客户端工具。                                                                                                                           | —     |

适用的端点：

* `/v1/messages` 使用 `text_editor` 工具
