---
title: 教程：构建一个使用工具的智能体
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/build-a-tool-using-agent
description: 从单次工具调用到生产就绪的智能体循环的引导式演练。
---

本教程通过五个同心环（ring）构建一个日历管理智能体。每一环都是一个完整、可运行的程序，并且只在前一环的基础上增加一个概念。完成后，您将先手动编写"agentic loop"（智能体循环），然后用"tool runner"（工具运行器）这一 SDK 抽象来替代它。

示例工具是 `create_calendar_event`。它的 schema 使用了嵌套对象、数组和可选字段，因此您将看到 Claude 如何处理真实的输入结构，而不仅仅是单个扁平字符串。

<Note>
  每一环都可以独立运行。将任意一环复制到一个新文件中，无需前面各环的代码即可运行。
</Note>

## 第 1 环：单个工具，单轮对话

最小的工具使用程序：一个工具、一条用户消息、一次工具调用、一个结果。代码带有大量注释，便于您将每一行对应到[工具使用生命周期](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/how-tool-use-works)。

请求在用户消息之外还发送一个 `tools` 数组。当 Claude 判断需要调用工具时，响应会返回 `stop_reason: "tool_use"` 以及一个 `tool_use` 内容块，其中包含工具名称、唯一的 `id` 和结构化的 `input`。您的代码运行该工具，然后在一个 `tool_result` 块中将结果发回，该块的 `tool_use_id` 与调用中的 `id` 相匹配。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  # 第 1 环：单个工具，单轮对话。

  import json

  import anthropic

  # 创建客户端。它会从环境变量中读取 ANTHROPIC_API_KEY。
  client = anthropic.Anthropic()

  # 定义一个工具。input_schema 是一个 JSON Schema 对象，用于描述
  # Claude 调用此工具时应传递的参数。此 schema
  # 包含嵌套对象（recurrence）、数组（attendees）和可选
  # 字段，这比扁平的字符串参数更接近真实世界的工具。
  tools = [
      {
          "name": "create_calendar_event",
          "description": "Create a calendar event with attendees and optional recurrence.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "title": {"type": "string"},
                  "start": {"type": "string", "format": "date-time"},
                  "end": {"type": "string", "format": "date-time"},
                  "attendees": {
                      "type": "array",
                      "items": {"type": "string", "format": "email"},
                  },
                  "recurrence": {
                      "type": "object",
                      "properties": {
                          "frequency": {"enum": ["daily", "weekly", "monthly"]},
                          "count": {"type": "integer", "minimum": 1},
                      },
                  },
              },
              "required": ["title", "start", "end"],
          },
      }
  ]

  # 将用户的请求连同工具定义一起发送。Claude 会根据
  # 请求和工具描述来决定是否调用该工具。
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=tools,
      tool_choice={"type": "auto", "disable_parallel_tool_use": True},
      messages=[
          {
              "role": "user",
              "content": "Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am.",
          }
      ],
  )

  # 当 Claude 调用工具时，响应的 stop_reason 为 "tool_use"，
  # 并且 content 数组中会包含一个 tool_use 块以及可能的文本。
  print(f"stop_reason: {response.stop_reason}")

  # 查找 tool_use 块。响应可能在 tool_use 块之前包含文本块，
  # 因此应扫描 content 数组，而不是假定其位置。
  tool_use = next(block for block in response.content if block.type == "tool_use")
  print(f"Tool: {tool_use.name}")
  print(f"Input: {tool_use.input}")

  # 运行工具。在真实系统中，这里会调用您的日历 API。
  # 此处将结果硬编码，以使示例保持独立完整。
  result = {"event_id": "evt_123", "status": "created"}

  # 将结果发回。tool_result 块放在 user 消息中，
  # 其 tool_use_id 必须与之前 tool_use 块的 ID 匹配。
  # 同时包含 assistant 之前的响应，以便 Claude 获得完整历史。
  followup = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=tools,
      tool_choice={"type": "auto", "disable_parallel_tool_use": True},
      messages=[
          {
              "role": "user",
              "content": "Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am.",
          },
          {"role": "assistant", "content": response.content},
          {
              "role": "user",
              "content": [
                  {
                      "type": "tool_result",
                      "tool_use_id": tool_use.id,
                      "content": json.dumps(result),
                  }
              ],
          },
      ],
  )

  # 获得工具结果后，Claude 会生成最终的自然语言
  # 回答，且 stop_reason 变为 "end_turn"。
  print(f"stop_reason: {followup.stop_reason}")
  final_text = next(block for block in followup.content if block.type == "text")
  print(final_text.text)
  ```

  ```typescript TypeScript
  // 第 1 环：单个工具，单轮对话。

  import Anthropic from "@anthropic-ai/sdk";

  // 创建客户端。它会从环境变量中读取 ANTHROPIC_API_KEY。
  const client = new Anthropic();

  // 定义一个工具。input_schema 是一个 JSON Schema 对象，用于描述
  // Claude 调用此工具时应传递的参数。此 schema
  // 包含嵌套对象（recurrence）、数组（attendees）和可选
  // 字段，这比扁平的字符串参数更接近真实世界的工具。
  const tools: Anthropic.Tool[] = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: { type: "string" },
          start: { type: "string", format: "date-time" },
          end: { type: "string", format: "date-time" },
          attendees: {
            type: "array",
            items: { type: "string", format: "email" }
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: { enum: ["daily", "weekly", "monthly"] },
              count: { type: "integer", minimum: 1 }
            }
          }
        },
        required: ["title", "start", "end"]
      }
    }
  ];

  // 将用户请求与工具定义一起发送。Claude 会根据
  // 请求和工具描述来决定是否调用该工具。
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools,
    tool_choice: { type: "auto", disable_parallel_tool_use: true },
    messages: [
      {
        role: "user",
        content:
          "Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am."
      }
    ]
  });

  // 当 Claude 调用工具时，响应的 stop_reason 为 "tool_use"，
  // 并且 content 数组中会包含一个 tool_use 块以及可能的文本。
  console.log(`stop_reason: ${response.stop_reason}`);

  // 查找 tool_use 块。响应中可能在 tool_use 块之前包含文本块，
  // 因此应扫描 content 数组，而不是假定其位置。
  const toolUse = response.content.find((block) => block.type === "tool_use")!;
  console.log(`Tool: ${toolUse.name}`);
  console.log(`Input: ${JSON.stringify(toolUse.input)}`);

  // 运行工具。在真实系统中，这里会调用您的日历 API。
  // 此处将结果硬编码，以使示例保持独立完整。
  const result = { event_id: "evt_123", status: "created" };

  // 将结果发回。tool_result 块放在 user 消息中，
  // 其 tool_use_id 必须与之前 tool_use 块的 ID 匹配。
  // 同时包含助手之前的响应，以便 Claude 获得完整历史。
  const followup = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools,
    tool_choice: { type: "auto", disable_parallel_tool_use: true },
    messages: [
      {
        role: "user",
        content:
          "Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am."
      },
      { role: "assistant", content: response.content },
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: toolUse.id,
            content: JSON.stringify(result)
          }
        ]
      }
    ]
  });

  // 获得工具结果后，Claude 会生成最终的自然语言
  // 回答，stop_reason 变为 "end_turn"。
  console.log(`stop_reason: ${followup.stop_reason}`);
  for (const block of followup.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  // 第 1 环：单个工具，单轮对话。

  using System;
  using System.Collections.Generic;
  using System.Linq;
  using System.Text.Json;
  using System.Threading.Tasks;
  using Anthropic;
  using Anthropic.Models.Messages;

  // 创建客户端。它会从环境变量中读取 ANTHROPIC_API_KEY。
  AnthropicClient client = new();

  // 定义一个工具。输入模式是一个 JSON Schema 对象，用于描述
  // Claude 调用此工具时应传递的参数。此模式
  // 包含嵌套对象（recurrence）、数组（attendees）和可选
  // 字段，这比扁平的字符串参数更接近真实世界的工具。
  List<ToolUnion> tools =
  [
      new ToolUnion(new Tool()
      {
          Name = "create_calendar_event",
          Description = "Create a calendar event with attendees and optional recurrence.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["title"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                  ["start"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["end"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["attendees"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "array",
                      items = new { type = "string", format = "email" },
                  }),
                  ["recurrence"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "object",
                      properties = new
                      {
                          frequency = new { @enum = new[] { "daily", "weekly", "monthly" } },
                          count = new { type = "integer", minimum = 1 },
                      },
                  }),
              },
              Required = ["title", "start", "end"],
          },
      }),
  ];

  // 要求每轮最多调用一次工具，以便后续的单轮流程
  // 保持可预测。
  var toolChoice = new ToolChoice(new ToolChoiceAuto { DisableParallelToolUse = true });

  const string userPrompt =
      "Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am.";

  // 将用户请求与工具定义一起发送。Claude 会根据
  // 请求和工具描述来决定是否调用该工具。
  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = tools,
      ToolChoice = toolChoice,
      Messages = [new UserMessageParam { Content = userPrompt }],
  });

  // 当 Claude 调用工具时，响应的 stop_reason 为 "tool_use"，
  // 并且 content 数组中会包含一个 tool_use 块以及任何文本。
  Console.WriteLine($"stop_reason: {response.StopReason?.Raw()}");

  // 查找 tool_use 块。响应中可能在 tool_use 块之前包含文本块，
  // 因此应扫描 content 数组，而不是假定其位置。
  ToolUseBlock? toolUse = null;
  foreach (var block in response.Content)
  {
      if (block.TryPickToolUse(out var picked))
      {
          toolUse = picked;
          break;
      }
  }
  Console.WriteLine($"Tool: {toolUse!.Name}");
  Console.WriteLine($"Input: {JsonSerializer.Serialize(toolUse.Input)}");

  // 运行工具。在真实系统中，这里会调用您的日历 API。
  // 此处将结果硬编码，以使示例保持独立完整。
  var result = """{"event_id": "evt_123", "status": "created"}""";

  // 将结果发回。tool_result 块放在 user 消息中，
  // 其 tool_use_id 必须与之前 tool_use 块的 ID 匹配。
  // 同时包含助手之前的响应，以便 Claude 获得完整历史。
  List<ContentBlockParam> toolResults =
  [
      new ContentBlockParam(new ToolResultBlockParam()
      {
          ToolUseID = toolUse.ID,
          Content = result,
      }),
  ];

  var followup = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = tools,
      ToolChoice = toolChoice,
      Messages =
      [
          new UserMessageParam { Content = userPrompt },
          new AssistantMessageParam { Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList() },
          new UserMessageParam { Content = toolResults },
      ],
  });

  // 获得工具结果后，Claude 会生成最终的自然语言
  // 回答，stop_reason 变为 "end_turn"。
  Console.WriteLine($"stop_reason: {followup.StopReason?.Raw()}");
  foreach (var block in followup.Content)
  {
      if (block.TryPickText(out var text))
      {
          Console.WriteLine(text.Text);
      }
  }
  ```

  ```go Go
  // 第 1 环：单个工具，单轮对话。

  package main

  import (
  	"context"
  	"fmt"
  	"log"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  func main() {
  	// 创建客户端。它会从环境变量中读取 ANTHROPIC_API_KEY。
  	client := anthropic.NewClient()
  	ctx := context.Background()

  	// 定义一个工具。输入模式是一个 JSON Schema 对象，用于描述
  	// Claude 调用此工具时应传递的参数。此模式
  	// 包含嵌套对象（recurrence）、数组（attendees）和可选
  	// 字段，这比扁平的字符串参数更接近真实世界的工具。
  	tools := []anthropic.ToolUnionParam{
  		{OfTool: &anthropic.ToolParam{
  			Name:        "create_calendar_event",
  			Description: anthropic.String("Create a calendar event with attendees and optional recurrence."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"title": map[string]any{"type": "string"},
  					"start": map[string]any{"type": "string", "format": "date-time"},
  					"end":   map[string]any{"type": "string", "format": "date-time"},
  					"attendees": map[string]any{
  						"type":  "array",
  						"items": map[string]any{"type": "string", "format": "email"},
  					},
  					"recurrence": map[string]any{
  						"type": "object",
  						"properties": map[string]any{
  							"frequency": map[string]any{"enum": []string{"daily", "weekly", "monthly"}},
  							"count":     map[string]any{"type": "integer", "minimum": 1},
  						},
  					},
  				},
  				Required: []string{"title", "start", "end"},
  			},
  		}},
  	}

  	// 要求每轮最多调用一次工具，以便后续的单轮流程
  	// 保持可预测。
  	toolChoice := anthropic.ToolChoiceUnionParam{
  		OfAuto: &anthropic.ToolChoiceAutoParam{DisableParallelToolUse: anthropic.Bool(true)},
  	}

  	userMessage := anthropic.NewUserMessage(anthropic.NewTextBlock(
  		"Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am.",
  	))

  	// 将用户请求与工具定义一起发送。Claude 会根据
  	// 请求和工具描述来决定是否调用该工具。
  	response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:      anthropic.ModelClaudeOpus5_5,
  		MaxTokens:  1024,
  		Tools:      tools,
  		ToolChoice: toolChoice,
  		Messages:   []anthropic.MessageParam{userMessage},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	// 当 Claude 调用工具时，响应的 stop_reason 为 "tool_use"，
  	// 并且 content 数组中会包含一个 tool_use 块以及任何文本。
  	fmt.Printf("stop_reason: %s\n", response.StopReason)

  	// 查找 tool_use 块。响应中可能在 tool_use 块之前包含文本块，
  	// 因此应扫描 content 数组，而不是假定其位置。
  	var toolUse anthropic.ContentBlockUnion
  	for _, block := range response.Content {
  		if block.Type == "tool_use" {
  			toolUse = block
  			break
  		}
  	}
  	fmt.Printf("Tool: %s\n", toolUse.Name)
  	fmt.Printf("Input: %s\n", string(toolUse.Input))

  	// 运行工具。在真实系统中，这里会调用您的日历 API。
  	// 此处将结果硬编码，以使示例保持独立完整。
  	result := `{"event_id": "evt_123", "status": "created"}`

  	// 将结果发回。tool_result 块放在 user 消息中，
  	// 其 tool_use_id 必须与之前 tool_use 块的 ID 匹配。
  	// 同时包含助手之前的响应，以便 Claude 获得完整历史记录。
  	var assistantContent []anthropic.ContentBlockParamUnion
  	for _, block := range response.Content {
  		assistantContent = append(assistantContent, block.ToParam())
  	}

  	followup, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:      anthropic.ModelClaudeOpus5_5,
  		MaxTokens:  1024,
  		Tools:      tools,
  		ToolChoice: toolChoice,
  		Messages: []anthropic.MessageParam{
  			userMessage,
  			anthropic.NewAssistantMessage(assistantContent...),
  			anthropic.NewUserMessage(anthropic.NewToolResultBlock(toolUse.ID, result, false)),
  		},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	// 获得工具结果后，Claude 会生成最终的自然语言
  	// 回答，stop_reason 变为 "end_turn"。
  	fmt.Printf("stop_reason: %s\n", followup.StopReason)
  	for _, block := range followup.Content {
  		if block.Type == "text" {
  			fmt.Println(block.Text)
  		}
  	}
  }
  ```

  ```java Java
  // 第 1 环：单个工具，单轮对话。

  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.messages.ContentBlockParam;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;
  import com.anthropic.models.messages.Tool;
  import com.anthropic.models.messages.Tool.InputSchema;
  import com.anthropic.models.messages.ToolChoiceAuto;
  import com.anthropic.models.messages.ToolResultBlockParam;
  import com.anthropic.models.messages.ToolUseBlock;
  import java.util.List;
  import java.util.Map;

  void main() {
      // 创建客户端。它会从环境变量中读取 ANTHROPIC_API_KEY。
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      // 定义一个工具。输入模式是一个 JSON Schema 对象，用于描述
      // Claude 调用此工具时应传递的参数。此模式
      // 包含嵌套对象（recurrence）、数组（attendees）和可选
      // 字段，这比扁平的字符串参数更接近真实世界的工具。
      Tool calendarTool = Tool.builder()
          .name("create_calendar_event")
          .description("Create a calendar event with attendees and optional recurrence.")
          .inputSchema(InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "title", Map.of("type", "string"),
                  "start", Map.of("type", "string", "format", "date-time"),
                  "end", Map.of("type", "string", "format", "date-time"),
                  "attendees", Map.of(
                      "type", "array",
                      "items", Map.of("type", "string", "format", "email")
                  ),
                  "recurrence", Map.of(
                      "type", "object",
                      "properties", Map.of(
                          "frequency", Map.of("enum", List.of("daily", "weekly", "monthly")),
                          "count", Map.of("type", "integer", "minimum", 1)
                      )
                  )
              )))
              .required(List.of("title", "start", "end"))
              .build())
          .build();

      // 要求每轮最多调用一次工具，以便后续的单轮流程
      // 保持可预测。
      ToolChoiceAuto toolChoice = ToolChoiceAuto.builder()
          .disableParallelToolUse(true)
          .build();

      String userPrompt =
          "Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am.";

      // 将用户请求与工具定义一起发送。Claude 会根据
      // 请求和工具描述来决定是否调用该工具。
      Message response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(calendarTool)
          .toolChoice(toolChoice)
          .addUserMessage(userPrompt)
          .build());

      // 当 Claude 调用工具时，响应的 stop_reason 为 "tool_use"，
      // 并且 content 数组中会包含一个 tool_use 块以及任何文本。
      IO.println("stop_reason: " + response.stopReason().orElse(null));

      // 查找 tool_use 块。响应可能在 tool_use 块之前包含文本块，
      // 因此应扫描 content 数组，而不是假定其位置。
      ToolUseBlock toolUse = response.content().stream()
          .flatMap(block -> block.toolUse().stream())
          .findFirst()
          .orElseThrow();
      IO.println("Tool: " + toolUse.name());
      IO.println("Input: " + toolUse._input());

      // 运行工具。在实际系统中，这里会调用您的日历 API。
      // 此处将结果硬编码，以使示例保持独立完整。
      String result = "{\"event_id\": \"evt_123\", \"status\": \"created\"}";

      // 将结果发回。tool_result 块放在 user 消息中，
      // 其 tool_use_id 必须与之前 tool_use 块中的 ID 匹配。
      // 同时包含助手之前的响应，以便 Claude 获得完整历史记录。
      Message followup = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(calendarTool)
          .toolChoice(toolChoice)
          .addUserMessage(userPrompt)
          .addMessage(response)
          .addUserMessageOfContentBlockParams(List.of(ContentBlockParam.ofToolResult(
              ToolResultBlockParam.builder()
                  .toolUseId(toolUse.id())
                  .content(result)
                  .build())))
          .build());

      // 获得工具结果后，Claude 会生成最终的自然语言
      // 回答，stop_reason 变为 "end_turn"。
      IO.println("stop_reason: " + followup.stopReason().orElse(null));
      followup.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  <?php

  // 第 1 环：单个工具，单轮对话。

  use Anthropic\Client;
  use Anthropic\Messages\ToolChoiceAuto;

  // 创建客户端。它会从环境变量中读取 ANTHROPIC_API_KEY。
  $client = new Client();

  // 定义一个工具。input_schema 是一个 JSON Schema 对象，用于描述
  // Claude 调用此工具时应传递的参数。此 schema
  // 包含嵌套对象（recurrence）、数组（attendees）和可选
  // 字段，相比扁平的字符串参数更接近真实世界的工具。
  $tools = [
      [
          'name' => 'create_calendar_event',
          'description' => 'Create a calendar event with attendees and optional recurrence.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'title' => ['type' => 'string'],
                  'start' => ['type' => 'string', 'format' => 'date-time'],
                  'end' => ['type' => 'string', 'format' => 'date-time'],
                  'attendees' => [
                      'type' => 'array',
                      'items' => ['type' => 'string', 'format' => 'email'],
                  ],
                  'recurrence' => [
                      'type' => 'object',
                      'properties' => [
                          'frequency' => ['enum' => ['daily', 'weekly', 'monthly']],
                          'count' => ['type' => 'integer', 'minimum' => 1],
                      ],
                  ],
              ],
              'required' => ['title', 'start', 'end'],
          ],
      ],
  ];

  $userMessage = [
      'role' => 'user',
      'content' => 'Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am.',
  ];

  // 要求每轮最多调用一次工具，以便后续的单轮流程
  // 保持可预测。
  $toolChoice = ToolChoiceAuto::with(disableParallelToolUse: true);

  // 将用户请求与工具定义一起发送。Claude 会根据
  // 请求和工具描述来决定是否调用该工具。
  $response = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      tools: $tools,
      toolChoice: $toolChoice,
      messages: [$userMessage],
  );

  // 当 Claude 调用工具时，响应的 stop_reason 为 "tool_use"，
  // 且 content 数组中会包含一个 tool_use 块以及可能的文本。
  printf("stop_reason: %s\n", $response->stopReason);

  // 查找 tool_use 块。响应中可能在 tool_use 块之前包含文本块，
  // 因此应扫描 content 数组，而不是假定其位置。
  $toolUse = null;
  foreach ($response->content as $block) {
      if ($block->type === 'tool_use') {
          $toolUse = $block;
          break;
      }
  }
  printf("Tool: %s\n", $toolUse->name);
  printf("Input: %s\n", json_encode($toolUse->input));

  // 运行工具。在真实系统中，这里会调用您的日历 API。
  // 此处将结果硬编码，以使示例保持独立完整。
  $result = ['event_id' => 'evt_123', 'status' => 'created'];

  // 将结果发回。tool_result 块放在 user 消息中，
  // 其 tool_use_id 必须与之前 tool_use 块的 ID 匹配。
  // 同时包含助手之前的响应，以便 Claude 获得完整历史。
  $followup = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      tools: $tools,
      toolChoice: $toolChoice,
      messages: [
          $userMessage,
          ['role' => 'assistant', 'content' => $response->content],
          [
              'role' => 'user',
              'content' => [
                  [
                      'type' => 'tool_result',
                      'tool_use_id' => $toolUse->id,
                      'content' => json_encode($result),
                  ],
              ],
          ],
      ],
  );

  // 获得工具结果后，Claude 会生成最终的自然语言
  // 回答，stop_reason 变为 "end_turn"。
  printf("stop_reason: %s\n", $followup->stopReason);
  foreach ($followup->content as $block) {
      if ($block->type === 'text') {
          echo $block->text, "\n";
      }
  }
  ```

  ```ruby Ruby
  # 第 1 环：单个工具，单轮对话。

  require "anthropic"

  # 创建客户端。它会从环境变量中读取 ANTHROPIC_API_KEY。
  client = Anthropic::Client.new

  # 定义一个工具。input_schema 是一个 JSON Schema 对象，用于描述
  # Claude 调用此工具时应传递的参数。此 schema
  # 包含嵌套对象（recurrence）、数组（attendees）和可选
  # 字段，这比扁平的字符串参数更接近真实世界的工具。
  tools = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: {type: "string"},
          start: {type: "string", format: "date-time"},
          end: {type: "string", format: "date-time"},
          attendees: {
            type: "array",
            items: {type: "string", format: "email"}
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: {enum: ["daily", "weekly", "monthly"]},
              count: {type: "integer", minimum: 1}
            }
          }
        },
        required: ["title", "start", "end"]
      }
    }
  ]

  user_message = {
    role: "user",
    content: "Schedule a 30-minute sync with alice@example.com and bob@example.com on Monday, March 30, 2026 at 10am."
  }

  # 要求每轮最多调用一次工具，以便后续的单轮流程
  # 保持可预测。
  tool_choice = {type: "auto", disable_parallel_tool_use: true}

  # 将用户的请求连同工具定义一起发送。Claude 会根据
  # 请求和工具描述来决定是否调用该工具。
  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: tools,
    tool_choice: tool_choice,
    messages: [user_message]
  )

  # 当 Claude 调用工具时，响应的 stop_reason 为 "tool_use"，
  # 并且 content 数组中会包含一个 tool_use 块以及可能的文本。
  puts "stop_reason: #{response.stop_reason}"

  # 查找 tool_use 块。响应可能在 tool_use 块之前包含文本块，
  # 因此应扫描 content 数组，而不是假定其位置。
  tool_use = response.content.find { |block| block.type == :tool_use }
  puts "Tool: #{tool_use.name}"
  puts "Input: #{tool_use.input}"

  # 运行工具。在真实系统中，这里会调用您的日历 API。
  # 此处将结果硬编码，以使示例保持独立完整。
  result = {event_id: "evt_123", status: "created"}

  # 将结果发回。tool_result 块放在 user 消息中，
  # 其 tool_use_id 必须与之前 tool_use 块的 ID 匹配。
  # 同时包含 assistant 之前的响应，以便 Claude 获得完整历史。
  followup = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: tools,
    tool_choice: tool_choice,
    messages: [
      user_message,
      {role: "assistant", content: response.content},
      {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: tool_use.id,
            content: JSON.generate(result)
          }
        ]
      }
    ]
  )

  # 获得工具结果后，Claude 会生成最终的自然语言
  # 回答，且 stop_reason 变为 "end_turn"。
  puts "stop_reason: #{followup.stop_reason}"
  followup.content.each do |block|
    puts block.text if block.type == :text
  end
  ```
</CodeGroup>

**预期结果**

```text Output wrap
stop_reason: tool_use
Tool: create_calendar_event
Input: {'title': 'Sync', 'start': '2026-03-30T10:00:00', 'end': '2026-03-30T10:30:00', 'attendees': ['alice@example.com', 'bob@example.com']}
stop_reason: end_turn
I've scheduled your 30-minute sync with Alice and Bob for Monday, March 30 at 10am.
```

第一个 `stop_reason` 是 `tool_use`，因为 Claude 正在等待日历结果。在您发送结果之后，第二个 `stop_reason` 是 `end_turn`，其内容是面向用户的自然语言。

## 第 2 环：智能体循环

第 1 环假设 Claude 恰好只调用一次工具。实际任务通常需要多次调用：Claude 可能会创建一个事件，读取确认信息，然后再创建另一个。解决办法是使用一个 `while` 循环，持续运行工具并将结果反馈回去，直到 `stop_reason` 不再是 `"tool_use"`。

另一个变化是对话历史。不要在每次请求时重新构建 `messages` 数组，而是维护一个持续更新的列表并向其追加内容。这样每次请求都会包含完整的先前上下文。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  # 第 2 环：agentic loop（智能体循环）。

  import json

  import anthropic

  client = anthropic.Anthropic()

  tools = [
      {
          "name": "create_calendar_event",
          "description": "Create a calendar event with attendees and optional recurrence.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "title": {"type": "string"},
                  "start": {"type": "string", "format": "date-time"},
                  "end": {"type": "string", "format": "date-time"},
                  "attendees": {
                      "type": "array",
                      "items": {"type": "string", "format": "email"},
                  },
                  "recurrence": {
                      "type": "object",
                      "properties": {
                          "frequency": {"enum": ["daily", "weekly", "monthly"]},
                          "count": {"type": "integer", "minimum": 1},
                      },
                  },
              },
              "required": ["title", "start", "end"],
          },
      }
  ]


  def run_tool(name, tool_input):
      if name == "create_calendar_event":
          return {
              "event_id": "evt_123",
              "status": "created",
              "title": tool_input["title"],
          }
      return {"error": f"Unknown tool: {name}"}


  # 将完整的对话历史保存在列表中，使每一轮都能看到之前的上下文。
  messages = [
      {
          "role": "user",
          "content": "Schedule a weekly team standup every Monday at 9am for the next 4 weeks, starting Monday, March 30, 2026. Invite the whole team: alice@example.com, bob@example.com, carol@example.com.",
      }
  ]

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=tools,
      tool_choice={"type": "auto", "disable_parallel_tool_use": True},
      messages=messages,
  )

  # 循环直到 Claude 不再请求工具。每次迭代都会运行所请求的
  # 工具，将结果追加到历史记录中，并请求 Claude 继续。
  while response.stop_reason == "tool_use":
      tool_use = next(block for block in response.content if block.type == "tool_use")
      result = run_tool(tool_use.name, tool_use.input)

      messages.append({"role": "assistant", "content": response.content})
      messages.append(
          {
              "role": "user",
              "content": [
                  {
                      "type": "tool_result",
                      "tool_use_id": tool_use.id,
                      "content": json.dumps(result),
                  }
              ],
          }
      )

      response = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          tools=tools,
          tool_choice={"type": "auto", "disable_parallel_tool_use": True},
          messages=messages,
      )

  final_text = next(block for block in response.content if block.type == "text")
  print(final_text.text)
  ```

  ```typescript TypeScript
  // 第 2 环：智能体循环（agentic loop）。

  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const tools: Anthropic.Tool[] = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: { type: "string" },
          start: { type: "string", format: "date-time" },
          end: { type: "string", format: "date-time" },
          attendees: {
            type: "array",
            items: { type: "string", format: "email" }
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: { enum: ["daily", "weekly", "monthly"] },
              count: { type: "integer", minimum: 1 }
            }
          }
        },
        required: ["title", "start", "end"]
      }
    }
  ];

  function runTool(name: string, input: Record<string, unknown>) {
    if (name === "create_calendar_event") {
      return { event_id: "evt_123", status: "created", title: input.title };
    }
    return { error: `Unknown tool: ${name}` };
  }

  // 保留完整的对话历史，使每一轮都能看到之前的上下文。
  const messages: Anthropic.MessageParam[] = [
    {
      role: "user",
      content:
        "Schedule a weekly team standup every Monday at 9am for the next 4 weeks, starting Monday, March 30, 2026. Invite the whole team: alice@example.com, bob@example.com, carol@example.com."
    }
  ];

  let response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools,
    tool_choice: { type: "auto", disable_parallel_tool_use: true },
    messages
  });

  // 循环直到 Claude 不再请求工具。每次迭代都会运行所请求的
  // 工具，将结果追加到历史记录中，并请 Claude 继续。
  while (response.stop_reason === "tool_use") {
    const toolUse = response.content.find((block) => block.type === "tool_use")!;
    const result = runTool(toolUse.name, toolUse.input as Record<string, unknown>);

    messages.push({ role: "assistant", content: response.content });
    messages.push({
      role: "user",
      content: [
        {
          type: "tool_result",
          tool_use_id: toolUse.id,
          content: JSON.stringify(result)
        }
      ]
    });

    response = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      tools,
      tool_choice: { type: "auto", disable_parallel_tool_use: true },
      messages
    });
  }

  for (const block of response.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  // 第 2 环：agentic loop（智能体循环）。

  using System;
  using System.Collections.Generic;
  using System.Linq;
  using System.Text.Json;
  using System.Threading.Tasks;
  using Anthropic;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  List<ToolUnion> tools =
  [
      new ToolUnion(new Tool()
      {
          Name = "create_calendar_event",
          Description = "Create a calendar event with attendees and optional recurrence.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["title"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                  ["start"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["end"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["attendees"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "array",
                      items = new { type = "string", format = "email" },
                  }),
                  ["recurrence"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "object",
                      properties = new
                      {
                          frequency = new { @enum = new[] { "daily", "weekly", "monthly" } },
                          count = new { type = "integer", minimum = 1 },
                      },
                  }),
              },
              Required = ["title", "start", "end"],
          },
      }),
  ];

  // 运行所请求的工具，并以字符串形式返回其结果。
  string RunTool(ToolUseBlock toolUse)
  {
      if (toolUse.Name == "create_calendar_event")
      {
          var title = toolUse.Input.TryGetValue("title", out var t) ? t.GetString() : "";
          return JsonSerializer.Serialize(new { event_id = "evt_123", status = "created", title });
      }
      return JsonSerializer.Serialize(new { error = $"Unknown tool: {toolUse.Name}" });
  }

  var toolChoice = new ToolChoice(new ToolChoiceAuto { DisableParallelToolUse = true });

  // 将完整的对话历史保存在列表中，使每一轮都能看到先前的上下文。
  List<MessageParam> messages =
  [
      new UserMessageParam
      {
          Content = "Schedule a weekly team standup every Monday at 9am for the next 4 weeks, starting Monday, March 30, 2026. Invite the whole team: alice@example.com, bob@example.com, carol@example.com.",
      },
  ];

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = tools,
      ToolChoice = toolChoice,
      Messages = messages,
  });

  // 循环执行，直到 Claude 不再请求工具。每次迭代都会运行所请求的
  // 工具，将结果追加到历史记录中，并请求 Claude 继续。
  while (response.StopReason == StopReason.ToolUse)
  {
      ToolUseBlock? toolUse = null;
      foreach (var block in response.Content)
      {
          if (block.TryPickToolUse(out var picked))
          {
              toolUse = picked;
              break;
          }
      }
      var result = RunTool(toolUse!);

      messages.Add(new AssistantMessageParam
      {
          Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList(),
      });
      messages.Add(new UserMessageParam
      {
          Content = new MessageContentParam(
          [
              new ContentBlockParam(new ToolResultBlockParam() { ToolUseID = toolUse!.ID, Content = result }),
          ]),
      });

      response = await client.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Tools = tools,
          ToolChoice = toolChoice,
          Messages = messages,
      });
  }

  foreach (var block in response.Content)
  {
      if (block.TryPickText(out var text))
      {
          Console.WriteLine(text.Text);
      }
  }
  ```

  ```go Go
  // 第 2 环：智能体循环（agentic loop）。

  package main

  import (
  	"context"
  	"encoding/json"
  	"fmt"
  	"log"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  func runTool(name string, input map[string]any) string {
  	if name == "create_calendar_event" {
  		title, _ := input["title"].(string)
  		return fmt.Sprintf(`{"event_id": "evt_123", "status": "created", "title": %q}`, title)
  	}
  	return fmt.Sprintf(`{"error": "Unknown tool: %s"}`, name)
  }

  func main() {
  	client := anthropic.NewClient()
  	ctx := context.Background()

  	tools := []anthropic.ToolUnionParam{
  		{OfTool: &anthropic.ToolParam{
  			Name:        "create_calendar_event",
  			Description: anthropic.String("Create a calendar event with attendees and optional recurrence."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"title": map[string]any{"type": "string"},
  					"start": map[string]any{"type": "string", "format": "date-time"},
  					"end":   map[string]any{"type": "string", "format": "date-time"},
  					"attendees": map[string]any{
  						"type":  "array",
  						"items": map[string]any{"type": "string", "format": "email"},
  					},
  					"recurrence": map[string]any{
  						"type": "object",
  						"properties": map[string]any{
  							"frequency": map[string]any{"enum": []string{"daily", "weekly", "monthly"}},
  							"count":     map[string]any{"type": "integer", "minimum": 1},
  						},
  					},
  				},
  				Required: []string{"title", "start", "end"},
  			},
  		}},
  	}

  	toolChoice := anthropic.ToolChoiceUnionParam{
  		OfAuto: &anthropic.ToolChoiceAutoParam{DisableParallelToolUse: anthropic.Bool(true)},
  	}

  	// 将完整的对话历史保存在一个切片中，使每一轮都能看到先前的上下文。
  	messages := []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock(
  			"Schedule a weekly team standup every Monday at 9am for the next 4 weeks, starting Monday, March 30, 2026. Invite the whole team: alice@example.com, bob@example.com, carol@example.com.",
  		)),
  	}

  	response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:      anthropic.ModelClaudeOpus5_5,
  		MaxTokens:  1024,
  		Tools:      tools,
  		ToolChoice: toolChoice,
  		Messages:   messages,
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	// 循环直到 Claude 不再请求工具。每次迭代都会运行所请求的
  	// 工具，将结果追加到历史记录中，并请求 Claude 继续。
  	for response.StopReason == "tool_use" {
  		var toolUse anthropic.ContentBlockUnion
  		for _, block := range response.Content {
  			if block.Type == "tool_use" {
  				toolUse = block
  				break
  			}
  		}

  		var input map[string]any
  		if err := json.Unmarshal(toolUse.Input, &input); err != nil {
  			log.Fatal(err)
  		}
  		result := runTool(toolUse.Name, input)

  		var assistantContent []anthropic.ContentBlockParamUnion
  		for _, block := range response.Content {
  			assistantContent = append(assistantContent, block.ToParam())
  		}
  		messages = append(messages, anthropic.NewAssistantMessage(assistantContent...))
  		messages = append(messages, anthropic.NewUserMessage(
  			anthropic.NewToolResultBlock(toolUse.ID, result, false),
  		))

  		response, err = client.Messages.New(ctx, anthropic.MessageNewParams{
  			Model:      anthropic.ModelClaudeOpus5_5,
  			MaxTokens:  1024,
  			Tools:      tools,
  			ToolChoice: toolChoice,
  			Messages:   messages,
  		})
  		if err != nil {
  			log.Fatal(err)
  		}
  	}

  	for _, block := range response.Content {
  		if block.Type == "text" {
  			fmt.Println(block.Text)
  		}
  	}
  }
  ```

  ```java Java
  // 第 2 环：agentic loop（智能体循环）。

  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.messages.ContentBlockParam;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.MessageParam;
  import com.anthropic.models.messages.Model;
  import com.anthropic.models.messages.StopReason;
  import com.anthropic.models.messages.Tool;
  import com.anthropic.models.messages.Tool.InputSchema;
  import com.anthropic.models.messages.ToolChoiceAuto;
  import com.anthropic.models.messages.ToolResultBlockParam;
  import com.anthropic.models.messages.ToolUseBlock;
  import java.util.ArrayList;
  import java.util.List;
  import java.util.Map;

  String runTool(ToolUseBlock toolUse) {
      // 原始工具输入是一个 JSON 对象；请将其作为 map 读取其中的字段。
      Map<String, JsonValue> input = (Map<String, JsonValue>) toolUse._input().asObject().get();
      if (toolUse.name().equals("create_calendar_event")) {
          String title = input.containsKey("title") ? input.get("title").asStringOrThrow() : "";
          return "{\"event_id\": \"evt_123\", \"status\": \"created\", \"title\": \"" + title + "\"}";
      }
      return "{\"error\": \"Unknown tool: " + toolUse.name() + "\"}";
  }

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Tool calendarTool = Tool.builder()
          .name("create_calendar_event")
          .description("Create a calendar event with attendees and optional recurrence.")
          .inputSchema(InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "title", Map.of("type", "string"),
                  "start", Map.of("type", "string", "format", "date-time"),
                  "end", Map.of("type", "string", "format", "date-time"),
                  "attendees", Map.of(
                      "type", "array",
                      "items", Map.of("type", "string", "format", "email")
                  ),
                  "recurrence", Map.of(
                      "type", "object",
                      "properties", Map.of(
                          "frequency", Map.of("enum", List.of("daily", "weekly", "monthly")),
                          "count", Map.of("type", "integer", "minimum", 1)
                      )
                  )
              )))
              .required(List.of("title", "start", "end"))
              .build())
          .build();

      ToolChoiceAuto toolChoice = ToolChoiceAuto.builder()
          .disableParallelToolUse(true)
          .build();

      // 将完整的对话历史保存在列表中，使每一轮都能看到先前的上下文。
      List<MessageParam> messages = new ArrayList<>();
      messages.add(MessageParam.ofUser(
          "Schedule a weekly team standup every Monday at 9am for the next 4 weeks, starting Monday, March 30, 2026. Invite the whole team: alice@example.com, bob@example.com, carol@example.com."));

      Message response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(calendarTool)
          .toolChoice(toolChoice)
          .messages(messages)
          .build());

      // 循环直到 Claude 不再请求工具。每次迭代都会运行所请求的
      // 工具，将结果追加到历史记录中，并请求 Claude 继续。
      while (response.stopReason().isPresent()
              && response.stopReason().get().equals(StopReason.TOOL_USE)) {
          ToolUseBlock toolUse = response.content().stream()
              .flatMap(block -> block.toolUse().stream())
              .findFirst()
              .orElseThrow();
          String result = runTool(toolUse);

          messages.add(response.toParam());
          messages.add(MessageParam.ofUserOfContentBlockParams(List.of(ContentBlockParam.ofToolResult(
              ToolResultBlockParam.builder()
                  .toolUseId(toolUse.id())
                  .content(result)
                  .build()))));

          response = client.messages().create(MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024L)
              .addTool(calendarTool)
              .toolChoice(toolChoice)
              .messages(messages)
              .build());
      }

      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  <?php

  // 第 2 环：agentic loop（智能体循环）。

  use Anthropic\Client;
  use Anthropic\Messages\ToolChoiceAuto;

  $client = new Client();

  $tools = [
      [
          'name' => 'create_calendar_event',
          'description' => 'Create a calendar event with attendees and optional recurrence.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'title' => ['type' => 'string'],
                  'start' => ['type' => 'string', 'format' => 'date-time'],
                  'end' => ['type' => 'string', 'format' => 'date-time'],
                  'attendees' => [
                      'type' => 'array',
                      'items' => ['type' => 'string', 'format' => 'email'],
                  ],
                  'recurrence' => [
                      'type' => 'object',
                      'properties' => [
                          'frequency' => ['enum' => ['daily', 'weekly', 'monthly']],
                          'count' => ['type' => 'integer', 'minimum' => 1],
                      ],
                  ],
              ],
              'required' => ['title', 'start', 'end'],
          ],
      ],
  ];

  function runTool(string $name, array $input): string
  {
      if ($name === 'create_calendar_event') {
          return json_encode([
              'event_id' => 'evt_123',
              'status' => 'created',
              'title' => $input['title'],
          ]);
      }

      return json_encode(['error' => "Unknown tool: {$name}"]);
  }

  $toolChoice = ToolChoiceAuto::with(disableParallelToolUse: true);

  // 将完整的对话历史保存在数组中，使每一轮都能看到先前的上下文。
  $messages = [
      [
          'role' => 'user',
          'content' => 'Schedule a weekly team standup every Monday at 9am for the next 4 weeks, starting Monday, March 30, 2026. Invite the whole team: alice@example.com, bob@example.com, carol@example.com.',
      ],
  ];

  $response = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      tools: $tools,
      toolChoice: $toolChoice,
      messages: $messages,
  );

  // 循环直到 Claude 不再请求工具。每次迭代都会运行所请求的
  // 工具，将结果追加到历史记录中，并请求 Claude 继续。
  while ($response->stopReason === 'tool_use') {
      $toolUse = null;
      foreach ($response->content as $block) {
          if ($block->type === 'tool_use') {
              $toolUse = $block;
              break;
          }
      }

      $result = runTool($toolUse->name, $toolUse->input);

      $messages[] = ['role' => 'assistant', 'content' => $response->content];
      $messages[] = [
          'role' => 'user',
          'content' => [
              [
                  'type' => 'tool_result',
                  'tool_use_id' => $toolUse->id,
                  'content' => $result,
              ],
          ],
      ];

      $response = $client->messages->create(
          model: 'claude-opus-5-5',
          maxTokens: 1024,
          tools: $tools,
          toolChoice: $toolChoice,
          messages: $messages,
      );
  }

  foreach ($response->content as $block) {
      if ($block->type === 'text') {
          echo $block->text, "\n";
      }
  }
  ```

  ```ruby Ruby
  # 第 2 环：agentic loop（智能体循环）。

  require "anthropic"

  client = Anthropic::Client.new

  tools = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: {type: "string"},
          start: {type: "string", format: "date-time"},
          end: {type: "string", format: "date-time"},
          attendees: {
            type: "array",
            items: {type: "string", format: "email"}
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: {enum: ["daily", "weekly", "monthly"]},
              count: {type: "integer", minimum: 1}
            }
          }
        },
        required: ["title", "start", "end"]
      }
    }
  ]

  def run_tool(name, input)
    case name
    when "create_calendar_event"
      JSON.generate({event_id: "evt_123", status: "created", title: input[:title]})
    else
      JSON.generate({error: "Unknown tool: #{name}"})
    end
  end

  tool_choice = {type: "auto", disable_parallel_tool_use: true}

  # 将完整的对话历史保存在数组中，使每一轮都能看到先前的上下文。
  messages = [
    {
      role: "user",
      content: "Schedule a weekly team standup every Monday at 9am for the next 4 weeks, starting Monday, March 30, 2026. Invite the whole team: alice@example.com, bob@example.com, carol@example.com."
    }
  ]

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: tools,
    tool_choice: tool_choice,
    messages: messages
  )

  # 循环直到 Claude 不再请求工具。每次迭代都会运行所请求的
  # 工具，将结果追加到历史记录中，并请求 Claude 继续。
  while response.stop_reason == :tool_use
    tool_use = response.content.find { |block| block.type == :tool_use }
    result = run_tool(tool_use.name, tool_use.input)

    messages << {role: "assistant", content: response.content}
    messages << {
      role: "user",
      content: [
        {
          type: "tool_result",
          tool_use_id: tool_use.id,
          content: result
        }
      ]
    }

    response = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      tools: tools,
      tool_choice: tool_choice,
      messages: messages
    )
  end

  response.content.each do |block|
    puts block.text if block.type == :text
  end
  ```
</CodeGroup>

**预期结果**

```text Output wrap
I've set up your weekly team standup for the next 4 Mondays at 9am with Alice, Bob, and Carol invited.
```

循环可能运行一次或多次，取决于 Claude 如何拆解任务。您的代码不再需要事先知道。

## 第 3 环：多个工具，并行调用

智能体很少只有一种能力。添加第二个工具 `list_calendar_events`，这样 Claude 就可以在创建新事件之前检查现有日程。

当 Claude 需要进行多个相互独立的工具调用时，它可能会在单个响应中返回多个 `tool_use` 块。您的循环需要处理所有这些块，并在一条用户消息中将所有结果一起发回。请遍历 `response.content` 中的每一个 `tool_use` 块，而不仅仅是第一个。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  # 第 3 环：多个工具，并行调用。

  import json

  import anthropic

  client = anthropic.Anthropic()

  tools = [
      {
          "name": "create_calendar_event",
          "description": "Create a calendar event with attendees and optional recurrence.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "title": {"type": "string"},
                  "start": {"type": "string", "format": "date-time"},
                  "end": {"type": "string", "format": "date-time"},
                  "attendees": {
                      "type": "array",
                      "items": {"type": "string", "format": "email"},
                  },
                  "recurrence": {
                      "type": "object",
                      "properties": {
                          "frequency": {"enum": ["daily", "weekly", "monthly"]},
                          "count": {"type": "integer", "minimum": 1},
                      },
                  },
              },
              "required": ["title", "start", "end"],
          },
      },
      {
          "name": "list_calendar_events",
          "description": "List all calendar events on a given date.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "date": {"type": "string", "format": "date"},
              },
              "required": ["date"],
          },
      },
  ]


  def run_tool(name, tool_input):
      if name == "create_calendar_event":
          return {
              "event_id": "evt_123",
              "status": "created",
              "title": tool_input["title"],
          }
      if name == "list_calendar_events":
          return {
              "events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]
          }
      return {"error": f"Unknown tool: {name}"}


  messages = [
      {
          "role": "user",
          "content": "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.",
      }
  ]

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=tools,
      messages=messages,
  )

  while response.stop_reason == "tool_use":
      # 单个响应可以包含多个 tool_use 块。请处理所有
      # 这些块，并在一条用户消息中一并返回所有结果。
      tool_results = []
      for block in response.content:
          if block.type == "tool_use":
              result = run_tool(block.name, block.input)
              tool_results.append(
                  {
                      "type": "tool_result",
                      "tool_use_id": block.id,
                      "content": json.dumps(result),
                  }
              )

      messages.append({"role": "assistant", "content": response.content})
      messages.append({"role": "user", "content": tool_results})

      response = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          tools=tools,
          messages=messages,
      )

  final_text = next(block for block in response.content if block.type == "text")
  print(final_text.text)
  ```

  ```typescript TypeScript
  // 第 3 环：多个工具，并行调用。

  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const tools: Anthropic.Tool[] = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: { type: "string" },
          start: { type: "string", format: "date-time" },
          end: { type: "string", format: "date-time" },
          attendees: {
            type: "array",
            items: { type: "string", format: "email" }
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: { enum: ["daily", "weekly", "monthly"] },
              count: { type: "integer", minimum: 1 }
            }
          }
        },
        required: ["title", "start", "end"]
      }
    },
    {
      name: "list_calendar_events",
      description: "List all calendar events on a given date.",
      input_schema: {
        type: "object",
        properties: {
          date: { type: "string", format: "date" }
        },
        required: ["date"]
      }
    }
  ];

  function runTool(name: string, input: Record<string, unknown>) {
    if (name === "create_calendar_event") {
      return { event_id: "evt_123", status: "created", title: input.title };
    }
    if (name === "list_calendar_events") {
      return {
        events: [{ title: "Existing meeting", start: "14:00", end: "15:00" }]
      };
    }
    return { error: `Unknown tool: ${name}` };
  }

  const messages: Anthropic.MessageParam[] = [
    {
      role: "user",
      content:
        "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts."
    }
  ];

  let response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools,
    messages
  });

  while (response.stop_reason === "tool_use") {
    // 单个响应可以包含多个 tool_use 块。请处理所有
    // 这些块，并在一条用户消息中一并返回所有结果。
    const toolResults: Anthropic.ToolResultBlockParam[] = [];
    for (const block of response.content) {
      if (block.type === "tool_use") {
        const result = runTool(block.name, block.input as Record<string, unknown>);
        toolResults.push({
          type: "tool_result",
          tool_use_id: block.id,
          content: JSON.stringify(result)
        });
      }
    }

    messages.push({ role: "assistant", content: response.content });
    messages.push({ role: "user", content: toolResults });

    response = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      tools,
      messages
    });
  }

  for (const block of response.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  // 第 3 环：多个工具，并行调用。

  using System;
  using System.Collections.Generic;
  using System.Linq;
  using System.Text.Json;
  using System.Threading.Tasks;
  using Anthropic;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  List<ToolUnion> tools =
  [
      new ToolUnion(new Tool()
      {
          Name = "create_calendar_event",
          Description = "Create a calendar event with attendees and optional recurrence.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["title"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                  ["start"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["end"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["attendees"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "array",
                      items = new { type = "string", format = "email" },
                  }),
                  ["recurrence"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "object",
                      properties = new
                      {
                          frequency = new { @enum = new[] { "daily", "weekly", "monthly" } },
                          count = new { type = "integer", minimum = 1 },
                      },
                  }),
              },
              Required = ["title", "start", "end"],
          },
      }),
      new ToolUnion(new Tool()
      {
          Name = "list_calendar_events",
          Description = "List all calendar events on a given date.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["date"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date" }),
              },
              Required = ["date"],
          },
      }),
  ];

  string RunTool(ToolUseBlock toolUse)
  {
      if (toolUse.Name == "create_calendar_event")
      {
          var title = toolUse.Input.TryGetValue("title", out var t) ? t.GetString() : "";
          return JsonSerializer.Serialize(new { event_id = "evt_123", status = "created", title });
      }
      if (toolUse.Name == "list_calendar_events")
      {
          return """{"events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]}""";
      }
      return JsonSerializer.Serialize(new { error = $"Unknown tool: {toolUse.Name}" });
  }

  List<MessageParam> messages =
  [
      new UserMessageParam
      {
          Content = "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.",
      },
  ];

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = tools,
      Messages = messages,
  });

  while (response.StopReason == StopReason.ToolUse)
  {
      // 单个响应可以包含多个 tool_use 块。请处理所有
      // 这些块，并在一条用户消息中一并返回所有结果。
      List<ContentBlockParam> toolResults = [];
      foreach (var block in response.Content)
      {
          if (block.TryPickToolUse(out var toolUse))
          {
              toolResults.Add(new ContentBlockParam(new ToolResultBlockParam()
              {
                  ToolUseID = toolUse.ID,
                  Content = RunTool(toolUse),
              }));
          }
      }

      messages.Add(new AssistantMessageParam
      {
          Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList(),
      });
      messages.Add(new UserMessageParam { Content = toolResults });

      response = await client.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Tools = tools,
          Messages = messages,
      });
  }

  foreach (var block in response.Content)
  {
      if (block.TryPickText(out var text))
      {
          Console.WriteLine(text.Text);
      }
  }
  ```

  ```go Go
  // 第 3 环：多个工具，并行调用。

  package main

  import (
  	"context"
  	"encoding/json"
  	"fmt"
  	"log"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  func runTool(name string, input map[string]any) string {
  	if name == "create_calendar_event" {
  		title, _ := input["title"].(string)
  		return fmt.Sprintf(`{"event_id": "evt_123", "status": "created", "title": %q}`, title)
  	}
  	if name == "list_calendar_events" {
  		return `{"events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]}`
  	}
  	return fmt.Sprintf(`{"error": "Unknown tool: %s"}`, name)
  }

  func main() {
  	client := anthropic.NewClient()
  	ctx := context.Background()

  	tools := []anthropic.ToolUnionParam{
  		{OfTool: &anthropic.ToolParam{
  			Name:        "create_calendar_event",
  			Description: anthropic.String("Create a calendar event with attendees and optional recurrence."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"title": map[string]any{"type": "string"},
  					"start": map[string]any{"type": "string", "format": "date-time"},
  					"end":   map[string]any{"type": "string", "format": "date-time"},
  					"attendees": map[string]any{
  						"type":  "array",
  						"items": map[string]any{"type": "string", "format": "email"},
  					},
  					"recurrence": map[string]any{
  						"type": "object",
  						"properties": map[string]any{
  							"frequency": map[string]any{"enum": []string{"daily", "weekly", "monthly"}},
  							"count":     map[string]any{"type": "integer", "minimum": 1},
  						},
  					},
  				},
  				Required: []string{"title", "start", "end"},
  			},
  		}},
  		{OfTool: &anthropic.ToolParam{
  			Name:        "list_calendar_events",
  			Description: anthropic.String("List all calendar events on a given date."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"date": map[string]any{"type": "string", "format": "date"},
  				},
  				Required: []string{"date"},
  			},
  		}},
  	}

  	messages := []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock(
  			"Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.",
  		)),
  	}

  	response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Tools:     tools,
  		Messages:  messages,
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	for response.StopReason == "tool_use" {
  		// 单个响应可以包含多个 tool_use 块。请处理所有
  		// 这些块，并在一条用户消息中一并返回所有结果。
  		var toolResults []anthropic.ContentBlockParamUnion
  		for _, block := range response.Content {
  			if block.Type == "tool_use" {
  				var input map[string]any
  				if err := json.Unmarshal(block.Input, &input); err != nil {
  					log.Fatal(err)
  				}
  				result := runTool(block.Name, input)
  				toolResults = append(toolResults, anthropic.NewToolResultBlock(block.ID, result, false))
  			}
  		}

  		var assistantContent []anthropic.ContentBlockParamUnion
  		for _, block := range response.Content {
  			assistantContent = append(assistantContent, block.ToParam())
  		}
  		messages = append(messages, anthropic.NewAssistantMessage(assistantContent...))
  		messages = append(messages, anthropic.NewUserMessage(toolResults...))

  		response, err = client.Messages.New(ctx, anthropic.MessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 1024,
  			Tools:     tools,
  			Messages:  messages,
  		})
  		if err != nil {
  			log.Fatal(err)
  		}
  	}

  	for _, block := range response.Content {
  		if block.Type == "text" {
  			fmt.Println(block.Text)
  		}
  	}
  }
  ```

  ```java Java
  // 第 3 环：多个工具，并行调用。

  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.messages.ContentBlock;
  import com.anthropic.models.messages.ContentBlockParam;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.MessageParam;
  import com.anthropic.models.messages.Model;
  import com.anthropic.models.messages.StopReason;
  import com.anthropic.models.messages.Tool;
  import com.anthropic.models.messages.Tool.InputSchema;
  import com.anthropic.models.messages.ToolResultBlockParam;
  import com.anthropic.models.messages.ToolUseBlock;
  import java.util.ArrayList;
  import java.util.List;
  import java.util.Map;

  String runTool(ToolUseBlock toolUse) {
      // 原始工具输入是一个 JSON 对象；以 map 形式从中读取字段。
      Map<String, JsonValue> input = (Map<String, JsonValue>) toolUse._input().asObject().get();
      if (toolUse.name().equals("create_calendar_event")) {
          String title = input.containsKey("title") ? input.get("title").asStringOrThrow() : "";
          return "{\"event_id\": \"evt_123\", \"status\": \"created\", \"title\": \"" + title + "\"}";
      }
      if (toolUse.name().equals("list_calendar_events")) {
          return "{\"events\": [{\"title\": \"Existing meeting\", \"start\": \"14:00\", \"end\": \"15:00\"}]}";
      }
      return "{\"error\": \"Unknown tool: " + toolUse.name() + "\"}";
  }

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Tool calendarTool = Tool.builder()
          .name("create_calendar_event")
          .description("Create a calendar event with attendees and optional recurrence.")
          .inputSchema(InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "title", Map.of("type", "string"),
                  "start", Map.of("type", "string", "format", "date-time"),
                  "end", Map.of("type", "string", "format", "date-time"),
                  "attendees", Map.of(
                      "type", "array",
                      "items", Map.of("type", "string", "format", "email")
                  ),
                  "recurrence", Map.of(
                      "type", "object",
                      "properties", Map.of(
                          "frequency", Map.of("enum", List.of("daily", "weekly", "monthly")),
                          "count", Map.of("type", "integer", "minimum", 1)
                      )
                  )
              )))
              .required(List.of("title", "start", "end"))
              .build())
          .build();

      Tool listTool = Tool.builder()
          .name("list_calendar_events")
          .description("List all calendar events on a given date.")
          .inputSchema(InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "date", Map.of("type", "string", "format", "date")
              )))
              .required(List.of("date"))
              .build())
          .build();

      List<MessageParam> messages = new ArrayList<>();
      messages.add(MessageParam.ofUser(
          "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts."));

      Message response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(calendarTool)
          .addTool(listTool)
          .messages(messages)
          .build());

      while (response.stopReason().isPresent()
              && response.stopReason().get().equals(StopReason.TOOL_USE)) {
          // 单个响应可能包含多个 tool_use 块。请处理所有
          // 这些块，并在一条 user 消息中一并返回所有结果。
          List<ContentBlockParam> toolResults = new ArrayList<>();
          for (ContentBlock block : response.content()) {
              if (block.toolUse().isPresent()) {
                  ToolUseBlock toolUse = block.toolUse().get();
                  toolResults.add(ContentBlockParam.ofToolResult(
                      ToolResultBlockParam.builder()
                          .toolUseId(toolUse.id())
                          .content(runTool(toolUse))
                          .build()));
              }
          }

          messages.add(response.toParam());
          messages.add(MessageParam.ofUserOfContentBlockParams(toolResults));

          response = client.messages().create(MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024L)
              .addTool(calendarTool)
              .addTool(listTool)
              .messages(messages)
              .build());
      }

      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  <?php

  // 第 3 环：多个工具，并行调用。

  use Anthropic\Client;

  $client = new Client();

  $tools = [
      [
          'name' => 'create_calendar_event',
          'description' => 'Create a calendar event with attendees and optional recurrence.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'title' => ['type' => 'string'],
                  'start' => ['type' => 'string', 'format' => 'date-time'],
                  'end' => ['type' => 'string', 'format' => 'date-time'],
                  'attendees' => [
                      'type' => 'array',
                      'items' => ['type' => 'string', 'format' => 'email'],
                  ],
                  'recurrence' => [
                      'type' => 'object',
                      'properties' => [
                          'frequency' => ['enum' => ['daily', 'weekly', 'monthly']],
                          'count' => ['type' => 'integer', 'minimum' => 1],
                      ],
                  ],
              ],
              'required' => ['title', 'start', 'end'],
          ],
      ],
      [
          'name' => 'list_calendar_events',
          'description' => 'List all calendar events on a given date.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'date' => ['type' => 'string', 'format' => 'date'],
              ],
              'required' => ['date'],
          ],
      ],
  ];

  function runTool(string $name, array $input): string
  {
      if ($name === 'create_calendar_event') {
          return json_encode([
              'event_id' => 'evt_123',
              'status' => 'created',
              'title' => $input['title'],
          ]);
      }
      if ($name === 'list_calendar_events') {
          return json_encode([
              'events' => [['title' => 'Existing meeting', 'start' => '14:00', 'end' => '15:00']],
          ]);
      }

      return json_encode(['error' => "Unknown tool: {$name}"]);
  }

  $messages = [
      [
          'role' => 'user',
          'content' => 'Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.',
      ],
  ];

  $response = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      tools: $tools,
      messages: $messages,
  );

  while ($response->stopReason === 'tool_use') {
      // 单个响应可以包含多个 tool_use 块。请处理所有
      // 这些块，并在一条用户消息中一并返回所有结果。
      $toolResults = [];
      foreach ($response->content as $block) {
          if ($block->type === 'tool_use') {
              $toolResults[] = [
                  'type' => 'tool_result',
                  'tool_use_id' => $block->id,
                  'content' => runTool($block->name, $block->input),
              ];
          }
      }

      $messages[] = ['role' => 'assistant', 'content' => $response->content];
      $messages[] = ['role' => 'user', 'content' => $toolResults];

      $response = $client->messages->create(
          model: 'claude-opus-5-5',
          maxTokens: 1024,
          tools: $tools,
          messages: $messages,
      );
  }

  foreach ($response->content as $block) {
      if ($block->type === 'text') {
          echo $block->text, "\n";
      }
  }
  ```

  ```ruby Ruby
  # 第 3 环：多个工具，并行调用。

  require "anthropic"

  client = Anthropic::Client.new

  tools = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: {type: "string"},
          start: {type: "string", format: "date-time"},
          end: {type: "string", format: "date-time"},
          attendees: {
            type: "array",
            items: {type: "string", format: "email"}
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: {enum: ["daily", "weekly", "monthly"]},
              count: {type: "integer", minimum: 1}
            }
          }
        },
        required: ["title", "start", "end"]
      }
    },
    {
      name: "list_calendar_events",
      description: "List all calendar events on a given date.",
      input_schema: {
        type: "object",
        properties: {
          date: {type: "string", format: "date"}
        },
        required: ["date"]
      }
    }
  ]

  def run_tool(name, input)
    case name
    when "create_calendar_event"
      JSON.generate({event_id: "evt_123", status: "created", title: input[:title]})
    when "list_calendar_events"
      JSON.generate({events: [{title: "Existing meeting", start: "14:00", end: "15:00"}]})
    else
      JSON.generate({error: "Unknown tool: #{name}"})
    end
  end

  messages = [
    {
      role: "user",
      content: "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts."
    }
  ]

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: tools,
    messages: messages
  )

  while response.stop_reason == :tool_use
    # 单个响应可以包含多个 tool_use 块。请处理所有
    # 这些块，并在一条用户消息中一并返回所有结果。
    tool_results = response.content.select { |block| block.type == :tool_use }.map do |tool_use|
      {
        type: "tool_result",
        tool_use_id: tool_use.id,
        content: run_tool(tool_use.name, tool_use.input)
      }
    end

    messages << {role: "assistant", content: response.content}
    messages << {role: "user", content: tool_results}

    response = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      tools: tools,
      messages: messages
    )
  end

  response.content.each do |block|
    puts block.text if block.type == :text
  end
  ```
</CodeGroup>

**预期结果**

```text Output wrap
I checked your calendar for Monday, March 30 and found an existing meeting from 2pm to 3pm. I've scheduled the planning session for 10am to 11am to avoid the conflict.
```

有关并发执行和顺序保证的更多信息，请参阅[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)。

## 第 4 环：错误处理

工具会失败。日历 API 可能会拒绝参与者过多的事件，或者日期格式可能有误。当工具抛出错误时，请使用 `is_error: true` 将错误消息发回，而不是让程序崩溃。Claude 会读取错误信息，并可以使用修正后的输入重试、向用户请求澄清，或解释该限制。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  # 第 4 环：错误处理。

  import json

  import anthropic

  client = anthropic.Anthropic()

  tools = [
      {
          "name": "create_calendar_event",
          "description": "Create a calendar event with attendees and optional recurrence.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "title": {"type": "string"},
                  "start": {"type": "string", "format": "date-time"},
                  "end": {"type": "string", "format": "date-time"},
                  "attendees": {
                      "type": "array",
                      "items": {"type": "string", "format": "email"},
                  },
                  "recurrence": {
                      "type": "object",
                      "properties": {
                          "frequency": {"enum": ["daily", "weekly", "monthly"]},
                          "count": {"type": "integer", "minimum": 1},
                      },
                  },
              },
              "required": ["title", "start", "end"],
          },
      },
      {
          "name": "list_calendar_events",
          "description": "List all calendar events on a given date.",
          "input_schema": {
              "type": "object",
              "properties": {
                  "date": {"type": "string", "format": "date"},
              },
              "required": ["date"],
          },
      },
  ]


  def run_tool(name, tool_input):
      if name == "create_calendar_event":
          if "attendees" in tool_input and len(tool_input["attendees"]) > 10:
              raise ValueError("Too many attendees (max 10)")
          return {
              "event_id": "evt_123",
              "status": "created",
              "title": tool_input["title"],
          }
      if name == "list_calendar_events":
          return {
              "events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]
          }
      raise ValueError(f"Unknown tool: {name}")


  messages = [
      {
          "role": "user",
          "content": "Schedule a one-hour all-hands on Monday, March 30, 2026 at 10am with everyone: "
          + ", ".join(f"user{i}@example.com" for i in range(15)),
      }
  ]

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=tools,
      messages=messages,
  )

  while response.stop_reason == "tool_use":
      tool_results = []
      for block in response.content:
          if block.type == "tool_use":
              try:
                  result = run_tool(block.name, block.input)
                  tool_results.append(
                      {
                          "type": "tool_result",
                          "tool_use_id": block.id,
                          "content": json.dumps(result),
                      }
                  )
              except Exception as exc:
                  # 发出失败信号，以便 Claude 重试或请求澄清。
                  tool_results.append(
                      {
                          "type": "tool_result",
                          "tool_use_id": block.id,
                          "content": str(exc),
                          "is_error": True,
                      }
                  )

      messages.append({"role": "assistant", "content": response.content})
      messages.append({"role": "user", "content": tool_results})

      response = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          tools=tools,
          messages=messages,
      )

  final_text = next(block for block in response.content if block.type == "text")
  print(final_text.text)
  ```

  ```typescript TypeScript
  // 第 4 环：错误处理。

  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const tools: Anthropic.Tool[] = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: { type: "string" },
          start: { type: "string", format: "date-time" },
          end: { type: "string", format: "date-time" },
          attendees: {
            type: "array",
            items: { type: "string", format: "email" }
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: { enum: ["daily", "weekly", "monthly"] },
              count: { type: "integer", minimum: 1 }
            }
          }
        },
        required: ["title", "start", "end"]
      }
    },
    {
      name: "list_calendar_events",
      description: "List all calendar events on a given date.",
      input_schema: {
        type: "object",
        properties: {
          date: { type: "string", format: "date" }
        },
        required: ["date"]
      }
    }
  ];

  function runTool(name: string, input: Record<string, unknown>) {
    if (name === "create_calendar_event") {
      const attendees = input.attendees as string[] | undefined;
      if (attendees && attendees.length > 10) {
        throw new Error("Too many attendees (max 10)");
      }
      return { event_id: "evt_123", status: "created", title: input.title };
    }
    if (name === "list_calendar_events") {
      return {
        events: [{ title: "Existing meeting", start: "14:00", end: "15:00" }]
      };
    }
    throw new Error(`Unknown tool: ${name}`);
  }

  const emails = Array.from({ length: 15 }, (_, i) => `user${i}@example.com`).join(", ");
  const messages: Anthropic.MessageParam[] = [
    {
      role: "user",
      content: `Schedule a one-hour all-hands on Monday, March 30, 2026 at 10am with everyone: ${emails}`
    }
  ];

  let response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools,
    messages
  });

  while (response.stop_reason === "tool_use") {
    const toolResults: Anthropic.ToolResultBlockParam[] = [];
    for (const block of response.content) {
      if (block.type === "tool_use") {
        try {
          const result = runTool(block.name, block.input as Record<string, unknown>);
          toolResults.push({
            type: "tool_result",
            tool_use_id: block.id,
            content: JSON.stringify(result)
          });
        } catch (err) {
          // 发出失败信号，以便 Claude 重试或请求澄清。
          toolResults.push({
            type: "tool_result",
            tool_use_id: block.id,
            content: String(err),
            is_error: true
          });
        }
      }
    }

    messages.push({ role: "assistant", content: response.content });
    messages.push({ role: "user", content: toolResults });

    response = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      tools,
      messages
    });
  }

  for (const block of response.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  // 第 4 环：错误处理。

  using System;
  using System.Collections.Generic;
  using System.Linq;
  using System.Text.Json;
  using System.Threading.Tasks;
  using Anthropic;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  List<ToolUnion> tools =
  [
      new ToolUnion(new Tool()
      {
          Name = "create_calendar_event",
          Description = "Create a calendar event with attendees and optional recurrence.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["title"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                  ["start"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["end"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date-time" }),
                  ["attendees"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "array",
                      items = new { type = "string", format = "email" },
                  }),
                  ["recurrence"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "object",
                      properties = new
                      {
                          frequency = new { @enum = new[] { "daily", "weekly", "monthly" } },
                          count = new { type = "integer", minimum = 1 },
                      },
                  }),
              },
              Required = ["title", "start", "end"],
          },
      }),
      new ToolUnion(new Tool()
      {
          Name = "list_calendar_events",
          Description = "List all calendar events on a given date.",
          InputSchema = new InputSchema()
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["date"] = JsonSerializer.SerializeToElement(new { type = "string", format = "date" }),
              },
              Required = ["date"],
          },
      }),
  ];

  string RunTool(ToolUseBlock toolUse)
  {
      if (toolUse.Name == "create_calendar_event")
      {
          if (toolUse.Input.TryGetValue("attendees", out var attendees) && attendees.GetArrayLength() > 10)
          {
              throw new InvalidOperationException("Too many attendees (max 10)");
          }
          var title = toolUse.Input.TryGetValue("title", out var t) ? t.GetString() : "";
          return JsonSerializer.Serialize(new { event_id = "evt_123", status = "created", title });
      }
      if (toolUse.Name == "list_calendar_events")
      {
          return """{"events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]}""";
      }
      throw new InvalidOperationException($"Unknown tool: {toolUse.Name}");
  }

  // 构造一个超出该工具参会人数上限的请求，以触发错误处理路径。
  var emails = string.Join(", ", Enumerable.Range(0, 15).Select(i => $"user{i}@example.com"));

  List<MessageParam> messages =
  [
      new UserMessageParam { Content = $"Schedule a one-hour all-hands on Monday, March 30, 2026 at 10am with everyone: {emails}" },
  ];

  var response = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Tools = tools,
      Messages = messages,
  });

  while (response.StopReason == StopReason.ToolUse)
  {
      List<ContentBlockParam> toolResults = [];
      foreach (var block in response.Content)
      {
          if (block.TryPickToolUse(out var toolUse))
          {
              ToolResultBlockParam toolResult;
              try
              {
                  toolResult = new ToolResultBlockParam() { ToolUseID = toolUse.ID, Content = RunTool(toolUse) };
              }
              catch (Exception e)
              {
                  // 发出失败信号，以便 Claude 重试或请求澄清。
                  toolResult = new ToolResultBlockParam()
                  {
                      ToolUseID = toolUse.ID,
                      Content = e.Message,
                      IsError = true,
                  };
              }
              toolResults.Add(new ContentBlockParam(toolResult));
          }
      }

      messages.Add(new AssistantMessageParam
      {
          Content = response.Content.Select(block => new ContentBlockParam(block.Json)).ToList(),
      });
      messages.Add(new UserMessageParam { Content = toolResults });

      response = await client.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Tools = tools,
          Messages = messages,
      });
  }

  foreach (var block in response.Content)
  {
      if (block.TryPickText(out var text))
      {
          Console.WriteLine(text.Text);
      }
  }
  ```

  ```go Go
  // 第 4 环：错误处理。

  package main

  import (
  	"context"
  	"encoding/json"
  	"fmt"
  	"log"
  	"strings"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  func runTool(name string, input map[string]any) (string, error) {
  	if name == "create_calendar_event" {
  		if attendees, ok := input["attendees"].([]any); ok && len(attendees) > 10 {
  			return "", fmt.Errorf("too many attendees (max 10)")
  		}
  		title, _ := input["title"].(string)
  		return fmt.Sprintf(`{"event_id": "evt_123", "status": "created", "title": %q}`, title), nil
  	}
  	if name == "list_calendar_events" {
  		return `{"events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]}`, nil
  	}
  	return "", fmt.Errorf("unknown tool: %s", name)
  }

  func main() {
  	client := anthropic.NewClient()
  	ctx := context.Background()

  	tools := []anthropic.ToolUnionParam{
  		{OfTool: &anthropic.ToolParam{
  			Name:        "create_calendar_event",
  			Description: anthropic.String("Create a calendar event with attendees and optional recurrence."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"title": map[string]any{"type": "string"},
  					"start": map[string]any{"type": "string", "format": "date-time"},
  					"end":   map[string]any{"type": "string", "format": "date-time"},
  					"attendees": map[string]any{
  						"type":  "array",
  						"items": map[string]any{"type": "string", "format": "email"},
  					},
  					"recurrence": map[string]any{
  						"type": "object",
  						"properties": map[string]any{
  							"frequency": map[string]any{"enum": []string{"daily", "weekly", "monthly"}},
  							"count":     map[string]any{"type": "integer", "minimum": 1},
  						},
  					},
  				},
  				Required: []string{"title", "start", "end"},
  			},
  		}},
  		{OfTool: &anthropic.ToolParam{
  			Name:        "list_calendar_events",
  			Description: anthropic.String("List all calendar events on a given date."),
  			InputSchema: anthropic.ToolInputSchemaParam{
  				Properties: map[string]any{
  					"date": map[string]any{"type": "string", "format": "date"},
  				},
  				Required: []string{"date"},
  			},
  		}},
  	}

  	// 构造一个超出该工具参会人数上限的请求，以触发错误处理路径。
  	emails := make([]string, 15)
  	for i := range emails {
  		emails[i] = fmt.Sprintf("user%d@example.com", i)
  	}
  	messages := []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock(
  			"Schedule a one-hour all-hands on Monday, March 30, 2026 at 10am with everyone: " + strings.Join(emails, ", "),
  		)),
  	}

  	response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Tools:     tools,
  		Messages:  messages,
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	for response.StopReason == "tool_use" {
  		var toolResults []anthropic.ContentBlockParamUnion
  		for _, block := range response.Content {
  			if block.Type == "tool_use" {
  				var input map[string]any
  				if err := json.Unmarshal(block.Input, &input); err != nil {
  					log.Fatal(err)
  				}
  				result, toolErr := runTool(block.Name, input)
  				if toolErr != nil {
  					// 发出失败信号，以便 Claude 重试或请求澄清。
  					toolResults = append(toolResults, anthropic.NewToolResultBlock(block.ID, toolErr.Error(), true))
  				} else {
  					toolResults = append(toolResults, anthropic.NewToolResultBlock(block.ID, result, false))
  				}
  			}
  		}

  		var assistantContent []anthropic.ContentBlockParamUnion
  		for _, block := range response.Content {
  			assistantContent = append(assistantContent, block.ToParam())
  		}
  		messages = append(messages, anthropic.NewAssistantMessage(assistantContent...))
  		messages = append(messages, anthropic.NewUserMessage(toolResults...))

  		response, err = client.Messages.New(ctx, anthropic.MessageNewParams{
  			Model:     anthropic.ModelClaudeOpus5_5,
  			MaxTokens: 1024,
  			Tools:     tools,
  			Messages:  messages,
  		})
  		if err != nil {
  			log.Fatal(err)
  		}
  	}

  	for _, block := range response.Content {
  		if block.Type == "text" {
  			fmt.Println(block.Text)
  		}
  	}
  }
  ```

  ```java Java
  // 第 4 环：错误处理。

  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.messages.ContentBlock;
  import com.anthropic.models.messages.ContentBlockParam;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.MessageParam;
  import com.anthropic.models.messages.Model;
  import com.anthropic.models.messages.StopReason;
  import com.anthropic.models.messages.Tool;
  import com.anthropic.models.messages.Tool.InputSchema;
  import com.anthropic.models.messages.ToolResultBlockParam;
  import com.anthropic.models.messages.ToolUseBlock;
  import java.util.ArrayList;
  import java.util.List;
  import java.util.Map;
  import java.util.stream.Collectors;
  import java.util.stream.IntStream;

  String runTool(ToolUseBlock toolUse) {
      // 原始工具输入是一个 JSON 对象；请将其作为 map 读取字段。
      Map<String, JsonValue> input = (Map<String, JsonValue>) toolUse._input().asObject().get();
      if (toolUse.name().equals("create_calendar_event")) {
          int attendeeCount = input.containsKey("attendees")
              ? ((List<?>) input.get("attendees").asArray().get()).size()
              : 0;
          if (attendeeCount > 10) {
              throw new IllegalArgumentException("Too many attendees (max 10)");
          }
          String title = input.containsKey("title") ? input.get("title").asStringOrThrow() : "";
          return "{\"event_id\": \"evt_123\", \"status\": \"created\", \"title\": \"" + title + "\"}";
      }
      if (toolUse.name().equals("list_calendar_events")) {
          return "{\"events\": [{\"title\": \"Existing meeting\", \"start\": \"14:00\", \"end\": \"15:00\"}]}";
      }
      throw new IllegalArgumentException("Unknown tool: " + toolUse.name());
  }

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Tool calendarTool = Tool.builder()
          .name("create_calendar_event")
          .description("Create a calendar event with attendees and optional recurrence.")
          .inputSchema(InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "title", Map.of("type", "string"),
                  "start", Map.of("type", "string", "format", "date-time"),
                  "end", Map.of("type", "string", "format", "date-time"),
                  "attendees", Map.of(
                      "type", "array",
                      "items", Map.of("type", "string", "format", "email")
                  ),
                  "recurrence", Map.of(
                      "type", "object",
                      "properties", Map.of(
                          "frequency", Map.of("enum", List.of("daily", "weekly", "monthly")),
                          "count", Map.of("type", "integer", "minimum", 1)
                      )
                  )
              )))
              .required(List.of("title", "start", "end"))
              .build())
          .build();

      Tool listTool = Tool.builder()
          .name("list_calendar_events")
          .description("List all calendar events on a given date.")
          .inputSchema(InputSchema.builder()
              .properties(JsonValue.from(Map.of(
                  "date", Map.of("type", "string", "format", "date")
              )))
              .required(List.of("date"))
              .build())
          .build();

      // 构造一个超出该工具参会者上限的请求，以触发错误路径。
      String emails = IntStream.range(0, 15)
          .mapToObj(i -> "user" + i + "@example.com")
          .collect(Collectors.joining(", "));

      List<MessageParam> messages = new ArrayList<>();
      messages.add(MessageParam.ofUser(
          "Schedule a one-hour all-hands on Monday, March 30, 2026 at 10am with everyone: " + emails));

      Message response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addTool(calendarTool)
          .addTool(listTool)
          .messages(messages)
          .build());

      while (response.stopReason().isPresent()
              && response.stopReason().get().equals(StopReason.TOOL_USE)) {
          List<ContentBlockParam> toolResults = new ArrayList<>();
          for (ContentBlock block : response.content()) {
              if (block.toolUse().isPresent()) {
                  ToolUseBlock toolUse = block.toolUse().get();
                  ToolResultBlockParam.Builder resultBuilder = ToolResultBlockParam.builder()
                      .toolUseId(toolUse.id());
                  try {
                      resultBuilder.content(runTool(toolUse));
                  } catch (Exception e) {
                      // 发出失败信号，以便 Claude 重试或请求澄清。
                      resultBuilder.content(e.getMessage()).isError(true);
                  }
                  toolResults.add(ContentBlockParam.ofToolResult(resultBuilder.build()));
              }
          }

          messages.add(response.toParam());
          messages.add(MessageParam.ofUserOfContentBlockParams(toolResults));

          response = client.messages().create(MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024L)
              .addTool(calendarTool)
              .addTool(listTool)
              .messages(messages)
              .build());
      }

      response.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  <?php

  // 第 4 环：错误处理。

  use Anthropic\Client;

  $client = new Client();

  $tools = [
      [
          'name' => 'create_calendar_event',
          'description' => 'Create a calendar event with attendees and optional recurrence.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'title' => ['type' => 'string'],
                  'start' => ['type' => 'string', 'format' => 'date-time'],
                  'end' => ['type' => 'string', 'format' => 'date-time'],
                  'attendees' => [
                      'type' => 'array',
                      'items' => ['type' => 'string', 'format' => 'email'],
                  ],
                  'recurrence' => [
                      'type' => 'object',
                      'properties' => [
                          'frequency' => ['enum' => ['daily', 'weekly', 'monthly']],
                          'count' => ['type' => 'integer', 'minimum' => 1],
                      ],
                  ],
              ],
              'required' => ['title', 'start', 'end'],
          ],
      ],
      [
          'name' => 'list_calendar_events',
          'description' => 'List all calendar events on a given date.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'date' => ['type' => 'string', 'format' => 'date'],
              ],
              'required' => ['date'],
          ],
      ],
  ];

  function runTool(string $name, array $input): string
  {
      if ($name === 'create_calendar_event') {
          if (count($input['attendees'] ?? []) > 10) {
              throw new \InvalidArgumentException('Too many attendees (max 10)');
          }

          return json_encode([
              'event_id' => 'evt_123',
              'status' => 'created',
              'title' => $input['title'],
          ]);
      }
      if ($name === 'list_calendar_events') {
          return json_encode([
              'events' => [['title' => 'Existing meeting', 'start' => '14:00', 'end' => '15:00']],
          ]);
      }

      throw new \InvalidArgumentException("Unknown tool: {$name}");
  }

  // 构造一个超出该工具参会人数上限的请求，以触发错误处理路径。
  $emails = array_map(fn (int $i): string => "user{$i}@example.com", range(0, 14));
  $messages = [
      [
          'role' => 'user',
          'content' => 'Schedule a one-hour all-hands on Monday, March 30, 2026 at 10am with everyone: ' . implode(', ', $emails),
      ],
  ];

  $response = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      tools: $tools,
      messages: $messages,
  );

  while ($response->stopReason === 'tool_use') {
      $toolResults = [];
      foreach ($response->content as $block) {
          if ($block->type === 'tool_use') {
              try {
                  $toolResults[] = [
                      'type' => 'tool_result',
                      'tool_use_id' => $block->id,
                      'content' => runTool($block->name, $block->input),
                  ];
              } catch (\Exception $e) {
                  // 发出失败信号，以便 Claude 重试或请求澄清。
                  $toolResults[] = [
                      'type' => 'tool_result',
                      'tool_use_id' => $block->id,
                      'content' => $e->getMessage(),
                      'is_error' => true,
                  ];
              }
          }
      }

      $messages[] = ['role' => 'assistant', 'content' => $response->content];
      $messages[] = ['role' => 'user', 'content' => $toolResults];

      $response = $client->messages->create(
          model: 'claude-opus-5-5',
          maxTokens: 1024,
          tools: $tools,
          messages: $messages,
      );
  }

  foreach ($response->content as $block) {
      if ($block->type === 'text') {
          echo $block->text, "\n";
      }
  }
  ```

  ```ruby Ruby
  # 第 4 环：错误处理。

  require "anthropic"

  client = Anthropic::Client.new

  tools = [
    {
      name: "create_calendar_event",
      description: "Create a calendar event with attendees and optional recurrence.",
      input_schema: {
        type: "object",
        properties: {
          title: {type: "string"},
          start: {type: "string", format: "date-time"},
          end: {type: "string", format: "date-time"},
          attendees: {
            type: "array",
            items: {type: "string", format: "email"}
          },
          recurrence: {
            type: "object",
            properties: {
              frequency: {enum: ["daily", "weekly", "monthly"]},
              count: {type: "integer", minimum: 1}
            }
          }
        },
        required: ["title", "start", "end"]
      }
    },
    {
      name: "list_calendar_events",
      description: "List all calendar events on a given date.",
      input_schema: {
        type: "object",
        properties: {
          date: {type: "string", format: "date"}
        },
        required: ["date"]
      }
    }
  ]

  def run_tool(name, input)
    case name
    when "create_calendar_event"
      attendees = input[:attendees]
      raise ArgumentError, "Too many attendees (max 10)" if attendees && attendees.length > 10
      JSON.generate({event_id: "evt_123", status: "created", title: input[:title]})
    when "list_calendar_events"
      JSON.generate({events: [{title: "Existing meeting", start: "14:00", end: "15:00"}]})
    else
      raise ArgumentError, "Unknown tool: #{name}"
    end
  end

  # 构造一个超出该工具参会人数上限的请求，以触发错误路径。
  emails = (0...15).map { |i| "user#{i}@example.com" }
  messages = [
    {
      role: "user",
      content: "Schedule a one-hour all-hands on Monday, March 30, 2026 at 10am with everyone: #{emails.join(", ")}"
    }
  ]

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: tools,
    messages: messages
  )

  while response.stop_reason == :tool_use
    tool_results = response.content.select { |block| block.type == :tool_use }.map do |tool_use|
      begin
        {
          type: "tool_result",
          tool_use_id: tool_use.id,
          content: run_tool(tool_use.name, tool_use.input)
        }
      rescue => e
        # 发出失败信号，以便 Claude 重试或请求澄清。
        {
          type: "tool_result",
          tool_use_id: tool_use.id,
          content: e.message,
          is_error: true
        }
      end
    end

    messages << {role: "assistant", content: response.content}
    messages << {role: "user", content: tool_results}

    response = client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      tools: tools,
      messages: messages
    )
  end

  response.content.each do |block|
    puts block.text if block.type == :text
  end
  ```
</CodeGroup>

**预期结果**

```text Output wrap
I tried to schedule the all-hands but the calendar only allows 10 attendees per event. I can split this into two sessions, or you can let me know which 10 people to prioritize.
```

`is_error` 标志是与成功结果之间的唯一区别。Claude 会接收该标志和错误文本，并作出相应的回应。有关完整的错误处理参考，请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)。

## 第 5 环：工具运行器 SDK 抽象

第 2 环到第 4 环都手动编写了相同的循环：调用 API、检查 `stop_reason`、运行工具、追加结果、重复。工具运行器会替您完成这些工作。将每个工具定义为一个函数，把列表传给 `client.beta.messages.tool_runner()` (typescript: `client.beta.messages.toolRunner()`; java: `client.beta().messages().toolRunner()`; php: `$client->beta->messages->toolRunner()`; csharp: `client.Beta.Messages.ToolRunner()`; go: `client.Beta.Messages.NewToolRunner()`)，并在循环完成后获取最终消息。错误包装、结果格式化和对话管理都在内部处理。

SDK 提供了一个辅助工具，可以将普通函数转换为可运行的工具，并根据函数签名推导出输入 schema；以下示例展示了惯用写法。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    工具运行器是 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 中的一个辅助工具。原始 HTTP 请求没有对应的等效功能。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    工具运行器是 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 中的一个辅助工具。`ant` CLI 没有对应的等效功能。
  </CodeGroupItem>

  ```python Python
  # 第 5 环：工具运行器 SDK 抽象。

  import json

  import anthropic
  from anthropic import beta_tool

  client = anthropic.Anthropic()


  @beta_tool
  def create_calendar_event(
      title: str,
      start: str,
      end: str,
      attendees: list[str] | None = None,
      recurrence: dict | None = None,
  ) -> str:
      """Create a calendar event with attendees and optional recurrence.

      Args:
          title: Event title.
          start: Start time in ISO 8601 format.
          end: End time in ISO 8601 format.
          attendees: Email addresses to invite.
          recurrence: Dict with 'frequency' (daily, weekly, monthly) and 'count'.
      """
      if attendees and len(attendees) > 10:
          raise ValueError("Too many attendees (max 10)")
      return json.dumps({"event_id": "evt_123", "status": "created", "title": title})


  @beta_tool
  def list_calendar_events(date: str) -> str:
      """List all calendar events on a given date.

      Args:
          date: Date in YYYY-MM-DD format.
      """
      return json.dumps(
          {"events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]}
      )


  final_message = client.beta.messages.tool_runner(
      model="claude-opus-5-5",
      max_tokens=1024,
      tools=[create_calendar_event, list_calendar_events],
      messages=[
          {
              "role": "user",
              "content": "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.",
          }
      ],
  ).until_done()

  for block in final_message.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  // 第 5 环：工具运行器 SDK 抽象。

  import Anthropic from "@anthropic-ai/sdk";
  import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
  import { z } from "zod";

  const client = new Anthropic();

  const createCalendarEvent = betaZodTool({
    name: "create_calendar_event",
    description: "Create a calendar event with attendees and optional recurrence.",
    inputSchema: z.object({
      title: z.string(),
      start: z.string().datetime(),
      end: z.string().datetime(),
      attendees: z.array(z.string().email()).optional(),
      recurrence: z
        .object({
          frequency: z.enum(["daily", "weekly", "monthly"]),
          count: z.number().int().min(1)
        })
        .optional()
    }),
    run: async (input) => {
      if (input.attendees && input.attendees.length > 10) {
        throw new Error("Too many attendees (max 10)");
      }
      return JSON.stringify({
        event_id: "evt_123",
        status: "created",
        title: input.title
      });
    }
  });

  const listCalendarEvents = betaZodTool({
    name: "list_calendar_events",
    description: "List all calendar events on a given date.",
    inputSchema: z.object({
      date: z.string().date()
    }),
    run: async () => {
      return JSON.stringify({
        events: [{ title: "Existing meeting", start: "14:00", end: "15:00" }]
      });
    }
  });

  const finalMessage = await client.beta.messages.toolRunner({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [createCalendarEvent, listCalendarEvents],
    messages: [
      {
        role: "user",
        content:
          "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts."
      }
    ]
  });

  for (const block of finalMessage.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  // 第 5 环：工具运行器 SDK 抽象。

  using System;
  using System.Collections.Generic;
  using System.Text.Json;
  using System.Threading.Tasks;
  using Anthropic;
  using Anthropic.Helpers.Beta;
  using Anthropic.Models.Beta.Messages;
  using MessageCreateParams = Anthropic.Models.Beta.Messages.MessageCreateParams;
  using InputSchema = Anthropic.Models.Beta.Messages.InputSchema;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  // 将每个工具定义为可运行工具：定义中包含 JSON Schema，
  // 而 Run 回调包含具体实现。抛出异常时，
  // 该消息会作为设置了 is_error 的工具结果发回给 Claude。
  var createCalendarEvent = new BetaRunnableTool
  {
      Name = "create_calendar_event",
      Definition = new BetaTool
      {
          Name = "create_calendar_event",
          Description = "Create a calendar event with attendees and optional recurrence.",
          InputSchema = new InputSchema
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["title"] = JsonSerializer.SerializeToElement(new { type = "string", description = "Event title" }),
                  ["start"] = JsonSerializer.SerializeToElement(new { type = "string", description = "Start time in ISO 8601 format" }),
                  ["end"] = JsonSerializer.SerializeToElement(new { type = "string", description = "End time in ISO 8601 format" }),
                  ["attendees"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "array",
                      items = new { type = "string" },
                      description = "Email addresses to invite",
                  }),
                  ["recurrence"] = JsonSerializer.SerializeToElement(new
                  {
                      type = "object",
                      properties = new
                      {
                          frequency = new { @enum = new[] { "daily", "weekly", "monthly" } },
                          count = new { type = "integer", minimum = 1 },
                      },
                  }),
              },
              Required = ["title", "start", "end"],
          },
      },
      Run = (toolUse, _) =>
      {
          if (toolUse.Input.TryGetValue("attendees", out var attendees) && attendees.GetArrayLength() > 10)
          {
              throw new InvalidOperationException("Too many attendees (max 10)");
          }
          var title = toolUse.Input.TryGetValue("title", out var t) ? t.GetString() : "";
          return Task.FromResult<BetaToolResultBlockParamContent>(
              JsonSerializer.Serialize(new { event_id = "evt_123", status = "created", title })
          );
      },
  };

  var listCalendarEvents = new BetaRunnableTool
  {
      Name = "list_calendar_events",
      Definition = new BetaTool
      {
          Name = "list_calendar_events",
          Description = "List all calendar events on a given date.",
          InputSchema = new InputSchema
          {
              Properties = new Dictionary<string, JsonElement>
              {
                  ["date"] = JsonSerializer.SerializeToElement(new { type = "string", description = "Date in YYYY-MM-DD format" }),
              },
              Required = ["date"],
          },
      },
      Run = (toolUse, _) => Task.FromResult<BetaToolResultBlockParamContent>(
          """{"events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]}"""
      ),
  };

  // runner 会调用 API、运行所请求的工具并回传结果，
  // 直到 Claude 给出最终答案。
  var runner = client.Beta.Messages.ToolRunner(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages =
          [
              new BetaUserMessageParam
              {
                  Content = "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.",
              },
          ],
      },
      [createCalendarEvent, listCalendarEvents]
  );

  BetaMessage? finalMessage = null;
  await foreach (var message in runner)
  {
      finalMessage = message;
  }

  foreach (var block in finalMessage!.Content)
  {
      if (block.TryPickText(out var text))
      {
          Console.WriteLine(text.Text);
      }
  }
  ```

  ```go Go
  // 第 5 环：工具运行器 SDK 抽象。

  package main

  import (
  	"context"
  	"fmt"
  	"log"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"github.com/anthropics/anthropic-sdk-go/v2/toolrunner"
  )

  // 输入结构体定义了每个工具的 schema。tool runner 会根据
  // 结构体字段及其 jsonschema 标签生成 JSON Schema。
  type RecurrenceInput struct {
  	Frequency string `json:"frequency,omitempty" jsonschema:"enum=daily,enum=weekly,enum=monthly,description=How often the event repeats"`
  	Count     int    `json:"count,omitempty" jsonschema:"description=Number of occurrences"`
  }

  type CreateCalendarEventInput struct {
  	Title      string           `json:"title" jsonschema:"required,description=Event title"`
  	Start      string           `json:"start" jsonschema:"required,description=Start time in ISO 8601 format"`
  	End        string           `json:"end" jsonschema:"required,description=End time in ISO 8601 format"`
  	Attendees  []string         `json:"attendees,omitempty" jsonschema:"description=Email addresses to invite"`
  	Recurrence *RecurrenceInput `json:"recurrence,omitempty"`
  }

  type ListCalendarEventsInput struct {
  	Date string `json:"date" jsonschema:"required,description=Date in YYYY-MM-DD format"`
  }

  func main() {
  	client := anthropic.NewClient()
  	ctx := context.Background()

  	// 将每个工具定义为一个处理函数。返回错误时，会将
  	// 该消息作为设置了 is_error 的工具结果发回给 Claude。
  	createCalendarEvent, err := toolrunner.NewBetaToolFromJSONSchema(
  		"create_calendar_event",
  		"Create a calendar event with attendees and optional recurrence.",
  		func(ctx context.Context, input CreateCalendarEventInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
  			if len(input.Attendees) > 10 {
  				return anthropic.BetaToolResultBlockParamContentUnion{}, fmt.Errorf("too many attendees (max 10)")
  			}
  			return anthropic.BetaToolResultBlockParamContentUnion{
  				OfText: &anthropic.BetaTextBlockParam{
  					Text: fmt.Sprintf(`{"event_id": "evt_123", "status": "created", "title": %q}`, input.Title),
  				},
  			}, nil
  		},
  	)
  	if err != nil {
  		log.Fatal(err)
  	}

  	listCalendarEvents, err := toolrunner.NewBetaToolFromJSONSchema(
  		"list_calendar_events",
  		"List all calendar events on a given date.",
  		func(ctx context.Context, input ListCalendarEventsInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
  			return anthropic.BetaToolResultBlockParamContentUnion{
  				OfText: &anthropic.BetaTextBlockParam{
  					Text: `{"events": [{"title": "Existing meeting", "start": "14:00", "end": "15:00"}]}`,
  				},
  			}, nil
  		},
  	)
  	if err != nil {
  		log.Fatal(err)
  	}

  	// runner 会调用 API、运行所请求的工具并将结果回传，
  	// 直到 Claude 给出最终答案。
  	runner := client.Beta.Messages.NewToolRunner(
  		[]anthropic.BetaTool{createCalendarEvent, listCalendarEvents},
  		anthropic.BetaToolRunnerParams{
  			BetaMessageNewParams: anthropic.BetaMessageNewParams{
  				Model:     anthropic.ModelClaudeOpus5_5,
  				MaxTokens: 1024,
  				Messages: []anthropic.BetaMessageParam{
  					anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock(
  						"Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.",
  					)),
  				},
  			},
  		},
  	)

  	var finalMessage *anthropic.BetaMessage
  	for message, err := range runner.All(ctx) {
  		if err != nil {
  			log.Fatal(err)
  		}
  		finalMessage = message
  	}

  	for _, block := range finalMessage.Content {
  		if block.Type == "text" {
  			fmt.Println(block.Text)
  		}
  	}
  }
  ```

  ```java Java
  // 第 5 环：工具运行器 SDK 抽象。

  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.helpers.BetaToolRunner;
  import com.anthropic.models.beta.messages.BetaMessage;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;
  import com.fasterxml.jackson.annotation.JsonClassDescription;
  import com.fasterxml.jackson.annotation.JsonPropertyDescription;
  import java.util.List;
  import java.util.function.Supplier;

  // 将每个工具定义为一个类：字段描述输入 schema，
  // get() 方法包含具体实现。抛出异常会将
  // 消息作为设置了 is_error 的工具结果发回给 Claude。
  @JsonClassDescription("Create a calendar event with attendees.")
  static class CreateCalendarEvent implements Supplier<String> {
      @JsonPropertyDescription("Event title")
      public String title;

      @JsonPropertyDescription("Start time in ISO 8601 format")
      public String start;

      @JsonPropertyDescription("End time in ISO 8601 format")
      public String end;

      @JsonPropertyDescription("Email addresses to invite")
      public List<String> attendees;

      @Override
      public String get() {
          if (attendees != null && attendees.size() > 10) {
              throw new IllegalArgumentException("Too many attendees (max 10)");
          }
          return "{\"event_id\": \"evt_123\", \"status\": \"created\", \"title\": \"" + title + "\"}";
      }
  }

  @JsonClassDescription("List all calendar events on a given date.")
  static class ListCalendarEvents implements Supplier<String> {
      @JsonPropertyDescription("Date in YYYY-MM-DD format")
      public String date;

      @Override
      public String get() {
          return "{\"events\": [{\"title\": \"Existing meeting\", \"start\": \"14:00\", \"end\": \"15:00\"}]}";
      }
  }

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      // runner 会调用 API、运行所请求的工具并回传结果，
      // 直到 Claude 给出最终答案。
      BetaToolRunner runner = client.beta()
          .messages()
          .toolRunner(MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024)
              .addBeta("structured-outputs-2025-11-13")
              .addUserMessage("Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.")
              .addTool(CreateCalendarEvent.class)
              .addTool(ListCalendarEvents.class)
              .build());

      BetaMessage finalMessage = null;
      for (BetaMessage message : runner) {
          finalMessage = message;
      }

      finalMessage.content().stream()
          .flatMap(block -> block.text().stream())
          .forEach(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  <?php

  // 第 5 环：工具运行器 SDK 抽象。

  use Anthropic\Client;
  use Anthropic\Lib\Tools\BetaRunnableTool;
  use Anthropic\Messages\Model;

  $client = new Client();

  // 将每个工具定义为可运行工具：定义中包含 JSON Schema，
  // run 闭包则包含具体实现。抛出异常时，会将该
  // 消息作为设置了 is_error 的工具结果发回给 Claude。
  $createCalendarEvent = new BetaRunnableTool(
      definition: [
          'name' => 'create_calendar_event',
          'description' => 'Create a calendar event with attendees and optional recurrence.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'title' => ['type' => 'string', 'description' => 'Event title'],
                  'start' => ['type' => 'string', 'description' => 'Start time in ISO 8601 format'],
                  'end' => ['type' => 'string', 'description' => 'End time in ISO 8601 format'],
                  'attendees' => [
                      'type' => 'array',
                      'items' => ['type' => 'string'],
                      'description' => 'Email addresses to invite',
                  ],
                  'recurrence' => [
                      'type' => 'object',
                      'properties' => [
                          'frequency' => ['enum' => ['daily', 'weekly', 'monthly']],
                          'count' => ['type' => 'integer', 'minimum' => 1],
                      ],
                  ],
              ],
              'required' => ['title', 'start', 'end'],
          ],
      ],
      run: function (array $input): string {
          if (count($input['attendees'] ?? []) > 10) {
              throw new \InvalidArgumentException('Too many attendees (max 10)');
          }

          return json_encode([
              'event_id' => 'evt_123',
              'status' => 'created',
              'title' => $input['title'],
          ]);
      },
  );

  $listCalendarEvents = new BetaRunnableTool(
      definition: [
          'name' => 'list_calendar_events',
          'description' => 'List all calendar events on a given date.',
          'input_schema' => [
              'type' => 'object',
              'properties' => [
                  'date' => ['type' => 'string', 'description' => 'Date in YYYY-MM-DD format'],
              ],
              'required' => ['date'],
          ],
      ],
      run: fn (array $input): string => json_encode([
          'events' => [['title' => 'Existing meeting', 'start' => '14:00', 'end' => '15:00']],
      ]),
  );

  // runner 会调用 API、运行所请求的工具并回传结果，
  // 直到 Claude 给出最终答案。
  $runner = $client->beta->messages->toolRunner(
      maxTokens: 1024,
      messages: [
          [
              'role' => 'user',
              'content' => 'Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts.',
          ],
      ],
      model: Model::CLAUDE_OPUS_5_5,
      tools: [$createCalendarEvent, $listCalendarEvents],
  );

  $finalMessage = null;
  foreach ($runner as $message) {
      $finalMessage = $message;
  }

  foreach ($finalMessage->content as $block) {
      if ($block->type === 'text') {
          echo $block->text, "\n";
      }
  }
  ```

  ```ruby Ruby
  # 第 5 环：工具运行器 SDK 抽象。

  require "anthropic"

  client = Anthropic::Client.new

  # 将每个工具定义为一个类：类型化的输入模型描述其 schema，
  # call 方法包含具体实现。抛出错误时，错误消息会
  # 作为设置了 is_error 的工具结果发回给 Claude。
  class RecurrenceInput < Anthropic::BaseModel
    optional :frequency, Anthropic::InputSchema::EnumOf["daily", "weekly", "monthly"],
             doc: "How often the event repeats"
    optional :count, Integer, doc: "Number of occurrences"
  end

  class CreateCalendarEventInput < Anthropic::BaseModel
    required :title, String, doc: "Event title"
    required :start, String, doc: "Start time in ISO 8601 format"
    required :end, String, doc: "End time in ISO 8601 format"
    optional :attendees, Anthropic::InputSchema::ArrayOf[String], doc: "Email addresses to invite"
    optional :recurrence, RecurrenceInput, doc: "Optional recurrence rule"
  end

  class CreateCalendarEvent < Anthropic::BaseTool
    doc "Create a calendar event with attendees and optional recurrence."
    input_schema CreateCalendarEventInput

    def call(input)
      raise ArgumentError, "Too many attendees (max 10)" if input.attendees && input.attendees.length > 10
      JSON.generate({event_id: "evt_123", status: "created", title: input.title})
    end
  end

  class ListCalendarEventsInput < Anthropic::BaseModel
    required :date, String, doc: "Date in YYYY-MM-DD format"
  end

  class ListCalendarEvents < Anthropic::BaseTool
    doc "List all calendar events on a given date."
    input_schema ListCalendarEventsInput

    def call(input)
      JSON.generate({events: [{title: "Existing meeting", start: "14:00", end: "15:00"}]})
    end
  end

  # runner 会调用 API、运行所请求的工具并将结果回传，
  # 直到 Claude 给出最终答案。
  runner = client.beta.messages.tool_runner(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    tools: [CreateCalendarEvent.new, ListCalendarEvents.new],
    messages: [
      {
        role: "user",
        content: "Check what I have on Monday, March 30, 2026, then schedule a one-hour planning session that day that avoids any conflicts."
      }
    ]
  )

  final_message = nil
  runner.each_message { |message| final_message = message }

  final_message.content.each do |block|
    puts block.text if block.type == :text
  end
  ```
</CodeGroup>

**预期结果**

```text Output wrap
I checked your calendar for Monday, March 30 and found an existing meeting from 2pm to 3pm. I've scheduled the planning session for 10am to 11am to avoid the conflict.
```

输出与第 3 环完全相同。区别在于代码：行数大约减少一半，没有手动循环，并且 schema 与实现放在一起。

## 您构建了什么

您从一个硬编码的单次工具调用开始，最终构建出一个具备生产形态的智能体，它能够处理多个工具、并行调用和错误，然后又将所有这些简化为工具运行器。在此过程中，您了解了工具使用协议的每个组成部分：`tool_use` 块、`tool_result` 块、`tool_use_id` 匹配、`stop_reason` 检查以及 `is_error` 信号。

## 后续步骤

<CardGroup>
  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools" title="定义工具">
    Schema 规范与最佳实践。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner" title="工具运行器">
    完整的 SDK 抽象参考。
  </Card>

  <Card href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/troubleshooting-tool-use" title="故障排除">
    修复常见的工具使用错误。
  </Card>
</CardGroup>
