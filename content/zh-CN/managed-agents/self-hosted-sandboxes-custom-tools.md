---
title: 自托管沙箱中的自定义工具
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-custom-tools
description: 从自托管沙箱 worker 提供自定义工具，并将您网络内的 MCP 服务器封装为自定义工具，无需运行隧道。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  status: beta
  betaHeader: managed-agents-2026-04-01
  zdr:
    eligibility: not-eligible
    note: 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。
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
    Claude API: beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Anthropic managed): beta
---

[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#custom-tools)是由您自己的代码运行的工具：智能体发出 `agent.custom_tool_use` 事件，并等待匹配的 `user.custom_tool_result`。您的 worker 可以就是这段代码。由于它在您的沙箱内运行，该工具可以访问您为沙箱配置的内部服务、凭据和网络出口，仅此而已。

环境密钥授权发布自定义工具结果，因此您的 Claude API 密钥无需放在 worker 主机上。

<Note>
  提供自定义工具需要使用 SDK worker。`ant` CLI worker 无法注册自定义工具实现。在每个会话一个沙箱（sandbox-per-session）的模式中，请[在沙箱内运行 SDK worker](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers#run-the-sdk-worker-inside-the-sandbox)。
</Note>

## 提供自定义工具

<Steps>
  <Step title="在智能体上声明工具">
    在智能体的 `tools` 中添加一个 `custom` 条目，其 `name` 与您的 worker 注册的工具相匹配。有关完整的声明结构，请参阅[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#custom-tools)。

    ```json
    {
      "type": "custom",
      "name": "get_order_status",
      "description": "Look up an order in the internal fulfillment system by order ID.",
      "input_schema": {
        "type": "object",
        "properties": {
          "order_id": { "type": "string", "description": "The order ID" }
        },
        "required": ["order_id"]
      }
    }
    ```
  </Step>

  <Step title="向 worker 注册实现">
    通过 worker 的 `tools` (go: `ToolsFunc`) 工厂（参阅 [`EnvironmentWorker`](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-reference#environment-worker)）传入该工具，并与内置工具集一起使用：

    <CodeGroup exclude="shell, csharp, java, php, ruby">
      ```python Python
      import asyncio
      import os
      from anthropic import AsyncAnthropic, beta_async_tool
      from anthropic.lib.environments import EnvironmentWorker
      from anthropic.lib.tools.agent_toolset import beta_agent_toolset_20260401


      @beta_async_tool
      async def get_order_status(order_id: str) -> str:
          """Look up an order in the internal fulfillment system by order ID."""
          # 在 worker 主机上运行：可调用沙箱能够访问的任何内容。
          return f"Order {order_id}: shipped"


      async def main() -> None:
          environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
          environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
          async with AsyncAnthropic(auth_token=environment_key) as client:
              await EnvironmentWorker(
                  client,
                  environment_id=environment_id,
                  environment_key=environment_key,
                  workdir="/workspace",
                  tools=lambda env: [*beta_agent_toolset_20260401(env), get_order_status],
              ).run()


      asyncio.run(main())
      ```

      ```typescript TypeScript
      import Anthropic from "@anthropic-ai/sdk";
      import { EnvironmentWorker } from "@anthropic-ai/sdk/helpers/beta/environments";
      import { betaTool } from "@anthropic-ai/sdk/helpers/beta/json-schema";
      import { betaAgentToolset20260401 } from "@anthropic-ai/sdk/tools/agent-toolset/node";

      const getOrderStatus = betaTool({
        name: "get_order_status",
        description: "Look up an order in the internal fulfillment system by order ID.",
        inputSchema: {
          type: "object",
          properties: { order_id: { type: "string", description: "The order ID" } },
          required: ["order_id"]
        },
        // 在 worker 主机上运行：可调用沙箱能访问的任何内容。
        run: async ({ order_id }) => `Order ${order_id}: shipped`
      });

      const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
      const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
      const client = new Anthropic({ authToken: environmentKey });
      const controller = new AbortController();
      process.once("SIGTERM", () => controller.abort());

      await new EnvironmentWorker({
        client,
        environmentId,
        environmentKey,
        workdir: "/workspace",
        signal: controller.signal,
        tools: (ctx) => [...betaAgentToolset20260401(ctx), getOrderStatus]
      }).run();
      ```

      <CodeGroupItem language="C#">
        C# SDK 没有 `EnvironmentWorker`。要从您自己的代码响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所述。
      </CodeGroupItem>

      ```go Go
      package main

      import (
      	"context"
      	"log"
      	"os"
      	"os/signal"
      	"syscall"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	"github.com/anthropics/anthropic-sdk-go/v2/lib/environments"
      	"github.com/anthropics/anthropic-sdk-go/v2/option"
      	"github.com/anthropics/anthropic-sdk-go/v2/toolrunner"
      	"github.com/anthropics/anthropic-sdk-go/v2/tools/agenttoolset"
      )

      type orderStatusInput struct {
      	OrderID string `json:"order_id"`
      }

      func main() {
      	environmentKey := os.Getenv("ANTHROPIC_ENVIRONMENT_KEY")
      	environmentID := os.Getenv("ANTHROPIC_ENVIRONMENT_ID")

      	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
      	defer stop()

      	getOrderStatus := toolrunner.NewBetaTool(
      		"get_order_status",
      		"Look up an order in the internal fulfillment system by order ID.",
      		anthropic.BetaToolInputSchemaParam{
      			Properties: map[string]any{
      				"order_id": map[string]any{"type": "string", "description": "The order ID"},
      			},
      			Required: []string{"order_id"},
      		},
      		// 在工作进程主机上运行：可调用沙箱能够访问的任何内容。
      		func(ctx context.Context, input orderStatusInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
      			return anthropic.BetaToolResultBlockParamContentUnion{
      				OfText: &anthropic.BetaTextBlockParam{Text: "Order " + input.OrderID + ": shipped"},
      			}, nil
      		},
      	)

      	client := anthropic.NewClient(option.WithAuthToken(environmentKey))

      	worker := environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{
      		EnvironmentID:  environmentID,
      		EnvironmentKey: environmentKey,
      		Workdir:        "/workspace",
      		ToolsFunc: func(env *agenttoolset.AgentToolContext) []anthropic.BetaTool {
      			return append(agenttoolset.BetaAgentToolset20260401(env), getOrderStatus)
      		},
      	})
      	if err := worker.Run(ctx); err != nil {
      		log.Fatalf("worker: %v", err)
      	}
      }
      ```

      <CodeGroupItem language="Java">
        Java SDK 没有 `EnvironmentWorker`。要从您自己的代码响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所述。
      </CodeGroupItem>

      <CodeGroupItem language="PHP">
        PHP SDK 没有 `EnvironmentWorker`。要从您自己的代码响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所述。
      </CodeGroupItem>

      <CodeGroupItem language="Ruby">
        Ruby SDK 没有 `EnvironmentWorker`。要从您自己的代码响应自定义工具调用，请在会话的事件流上发送每个结果，如[返回自定义工具结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result)中所述。
      </CodeGroupItem>
    </CodeGroup>
  </Step>
</Steps>

worker 只响应向其注册的工具。如果某个工具已在智能体上声明，但没有任何 worker 或客户端提供它，会话将以 `requires_action` 停止原因暂停，并一直保持暂停，直到有程序发布结果。有关事件流程，请参阅[答复使会话暂停的工具调用](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#answer-tool-calls-that-pause-the-session)。

## 将 MCP 服务器封装为自定义工具

[MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)从 Anthropic 一侧连接到 MCP 服务器。因此，服务器必须公开一个 Anthropic 可以访问的 HTTP 端点，可以是直接访问，也可以通过 [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)访问。

要使用只有您的网络才能访问的服务器，请改为让 worker 充当 MCP 客户端，并将服务器的工具声明为自定义工具。MCP 服务器不需要来自您网络外部的入站连接。Anthropic 会接收您在智能体上声明的工具定义、每次调用的输入，以及您的 worker 回传的结果。

在运行时，模型会像调用任何其他自定义工具一样调用封装的工具：

1. 智能体发出 `agent.custom_tool_use` 事件。
2. 位于您沙箱内的 worker 通过其已打开的 MCP 会话，将调用转发到您网络上的服务器。
3. worker 将服务器的响应作为 `user.custom_tool_result` 发布。

### 安装 MCP SDK

SDK 的[客户端 MCP 辅助函数](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#client-side-mcp-helpers)会将服务器的工具转换为 worker 可接受的可运行工具。请在 Anthropic SDK 之外安装一个 MCP SDK：`pip install "anthropic[mcp]" "mcp>=1.24"` (typescript: `npm install @modelcontextprotocol/sdk`; go: `go get github.com/modelcontextprotocol/go-sdk`; csharp: `dotnet add package ModelContextProtocol`; java: `com.anthropic:anthropic-java-mcp`; php: `composer require mcp/sdk`; ruby: `bundle add mcp faraday event_stream_parser`)。

示例在连接时不使用身份验证。要发送凭据，请配置您传给 MCP 传输层的 `http_client` (typescript: `requestInit`; go: `HTTPClient`; csharp: `AdditionalHeaders`; java: `.customizeRequest()`; php, ruby: `headers`)。

### 声明并提供工具

<Steps>
  <Step title="在智能体上声明服务器的工具">
    列出 MCP 服务器的工具，并将每个工具声明为 `custom` 工具。MCP 的 `name`、`description` 和 `inputSchema` 与自定义工具的字段一一对应。如果服务器对其工具列表进行分页，请声明每一页；worker 必须列出相同的页面。

    <CodeGroup exclude="shell">
      ```python Python
      import asyncio
      from typing import Any, cast
      from anthropic import AsyncAnthropic
      from anthropic.types.beta import BetaManagedAgentsCustomToolParams
      from mcp import ClientSession, types

      # 需要 mcp >= 1.24，该版本将 streamablehttp_client 重命名为 streamable_http_client。
      from mcp.client.streamable_http import streamable_http_client

      MCP_SERVER_URL = "http://mcp.internal.example.com:8000/mcp"


      def to_custom_tool(tool: types.Tool) -> BetaManagedAgentsCustomToolParams:
          # MCP 字段与自定义工具声明一一对应。cast
          # 将 schema 字典原样传递给 SDK 的类型化参数。
          return {
              "type": "custom",
              "name": tool.name,
              "description": tool.description or tool.name,
              "input_schema": cast(Any, tool.inputSchema),
          }


      async def main() -> None:
          # 请在您创建代理的位置运行此代码，而不是在工作主机上：
          # 它使用您的 Claude API 密钥（ANTHROPIC_API_KEY）进行身份验证。
          async with (
              streamable_http_client(MCP_SERVER_URL) as (read, write, _),
              ClientSession(read, write) as mcp_session,
              AsyncAnthropic() as client,
          ):
              await mcp_session.initialize()
              listed = await mcp_session.list_tools()
              agent = await client.beta.agents.create(
                  name="Internal tools agent",
                  model="claude-opus-5-5",
                  tools=[
                      {
                          "type": "agent_toolset_20260401",
                          "configs": [
                              {"name": "web_search", "enabled": False},
                              {"name": "web_fetch", "enabled": False},
                          ],
                      },
                      *[to_custom_tool(tool) for tool in listed.tools],
                  ],
              )
              print(agent.id)


      asyncio.run(main())
      ```

      ```typescript TypeScript
      import Anthropic from "@anthropic-ai/sdk";
      import { Client } from "@modelcontextprotocol/sdk/client/index.js";
      import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

      const MCP_SERVER_URL = "http://mcp.internal.example.com:8000/mcp";

      // 请在您创建代理的位置运行此代码，而不是在工作主机上：它
      // 使用您的 Claude API 密钥（ANTHROPIC_API_KEY）进行身份验证。
      const client = new Anthropic();

      const mcpClient = new Client({ name: "declare-agent-tools", version: "1.0.0" });
      await mcpClient.connect(new StreamableHTTPClientTransport(new URL(MCP_SERVER_URL)));
      const { tools } = await mcpClient.listTools();

      const agent = await client.beta.agents.create({
        name: "Internal tools agent",
        model: "claude-opus-5-5",
        tools: [
          {
            type: "agent_toolset_20260401",
            configs: [
              { name: "web_search", enabled: false },
              { name: "web_fetch", enabled: false }
            ]
          },
          // MCP 字段与自定义工具声明一一对应。
          ...tools.map((tool) => ({
            type: "custom" as const,
            name: tool.name,
            description: tool.description || tool.name,
            input_schema: tool.inputSchema
          }))
        ]
      });
      console.log(agent.id);

      await mcpClient.close();
      ```

      ```csharp C#
      using System.Text.Json;
      using Anthropic;
      using Anthropic.Models.Beta.Agents;
      using ModelContextProtocol.Client;

      const string McpServerUrl = "http://mcp.internal.example.com:8000/mcp";

      // 请在您创建 agent 的地方运行此代码，而不是在 worker 主机上：它
      // 使用您的 Claude API 密钥（ANTHROPIC_API_KEY）进行身份验证。
      var client = new AnthropicClient();

      await using var mcpClient = await McpClient.CreateAsync(
          new HttpClientTransport(new HttpClientTransportOptions { Endpoint = new Uri(McpServerUrl) })
      );
      var mcpTools = await mcpClient.ListToolsAsync();

      List<Tool> tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params
          {
              Configs =
              [
                  new BetaManagedAgentsWebSearchToolConfigParams { Enabled = false },
                  new BetaManagedAgentsWebFetchToolConfigParams { Enabled = false },
              ],
          },
      ];
      foreach (var tool in mcpTools)
      {
          var description = tool.Description;
          if (string.IsNullOrEmpty(description))
          {
              description = tool.Name;
          }

          // MCP 字段与自定义工具声明一一对应。
          // schema 的关键字会原样传递给 SDK。
          tools.Add(
              new BetaManagedAgentsCustomToolParams
              {
                  Name = tool.Name,
                  Description = description,
                  InputSchema = new(
                      tool.ProtocolTool.InputSchema.Deserialize<Dictionary<string, JsonElement>>()!
                  ),
              }
          );
      }

      var agent = await client.Beta.Agents.Create(new()
      {
          Name = "Internal tools agent",
          Model = BetaManagedAgentsModel.ClaudeOpus5_5,
          Tools = tools,
      });
      Console.WriteLine(agent.ID);
      ```

      ```go Go
      package main

      import (
      	"context"
      	"encoding/json"
      	"fmt"
      	"log"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	mcpsdk "github.com/modelcontextprotocol/go-sdk/mcp"
      )

      const mcpServerURL = "http://mcp.internal.example.com:8000/mcp"

      // toCustomTool 将一个 MCP 工具定义映射为一个自定义工具声明。
      // 字段一一对应：类型化参数承载 `properties` 和
      // `required`，服务器输出的其他所有 JSON Schema 关键字都放入
      // ExtraFields，以便声明的 schema 与服务器的 schema 保持一致。
      func toCustomTool(tool *mcpsdk.Tool) (anthropic.BetaAgentNewParamsToolUnion, error) {
      	raw, err := json.Marshal(tool.InputSchema)
      	if err != nil {
      		return anthropic.BetaAgentNewParamsToolUnion{}, err
      	}
      	var schema map[string]any
      	if err := json.Unmarshal(raw, &schema); err != nil {
      		return anthropic.BetaAgentNewParamsToolUnion{}, err
      	}

      	inputSchema := anthropic.BetaManagedAgentsCustomToolInputSchemaParam{ExtraFields: map[string]any{}}
      	for keyword, value := range schema {
      		switch keyword {
      		case "type":
      			// 该参数类型始终序列化为 "type": "object"。
      		case "properties":
      			properties, _ := value.(map[string]any)
      			inputSchema.Properties = properties
      		case "required":
      			entries, _ := value.([]any)
      			for _, entry := range entries {
      				if name, isString := entry.(string); isString {
      					inputSchema.Required = append(inputSchema.Required, name)
      				}
      			}
      		default:
      			inputSchema.ExtraFields[keyword] = value
      		}
      	}

      	description := tool.Description
      	if description == "" {
      		description = tool.Name
      	}
      	return anthropic.BetaAgentNewParamsToolUnion{
      		OfCustom: &anthropic.BetaManagedAgentsCustomToolParams{
      			Name:        tool.Name,
      			Description: description,
      			InputSchema: inputSchema,
      		},
      	}, nil
      }

      func main() {
      	ctx := context.Background()

      	// 请在您创建智能体的位置运行此程序，而非在工作节点主机上：
      	// 它使用您的 Claude API 密钥（ANTHROPIC_API_KEY）进行身份验证。
      	client := anthropic.NewClient()

      	mcpClient := mcpsdk.NewClient(&mcpsdk.Implementation{Name: "declare-agent-tools", Version: "1.0.0"}, nil)
      	session, err := mcpClient.Connect(ctx, &mcpsdk.StreamableClientTransport{Endpoint: mcpServerURL}, nil)
      	if err != nil {
      		log.Fatalf("connect to MCP server: %v", err)
      	}
      	defer session.Close()

      	listed, err := session.ListTools(ctx, nil)
      	if err != nil {
      		log.Fatalf("list MCP tools: %v", err)
      	}

      	tools := []anthropic.BetaAgentNewParamsToolUnion{
      		{OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
      			Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{
      				{OfWebSearch: &anthropic.BetaManagedAgentsWebSearchToolConfigParams{
      					Enabled: anthropic.Bool(false),
      				}},
      				{OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{
      					Enabled: anthropic.Bool(false),
      				}},
      			},
      		}},
      	}
      	for _, tool := range listed.Tools {
      		custom, err := toCustomTool(tool)
      		if err != nil {
      			log.Fatalf("convert MCP tool %s: %v", tool.Name, err)
      		}
      		tools = append(tools, custom)
      	}

      	agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
      		Name:  "Internal tools agent",
      		Model: anthropic.BetaManagedAgentsModelConfigParams{ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5},
      		Tools: tools,
      	})
      	if err != nil {
      		log.Fatalf("create agent: %v", err)
      	}
      	fmt.Println(agent.ID)
      }
      ```

      ```java Java
      import com.anthropic.client.okhttp.AnthropicOkHttpClient;
      import com.anthropic.core.JsonValue;
      import com.anthropic.models.beta.agents.AgentCreateParams;
      import com.anthropic.models.beta.agents.BetaManagedAgentsAgentToolset20260401Params;
      import com.anthropic.models.beta.agents.BetaManagedAgentsCustomToolInputSchema;
      import com.anthropic.models.beta.agents.BetaManagedAgentsCustomToolParams;
      import com.anthropic.models.beta.agents.BetaManagedAgentsModel;
      import com.anthropic.models.beta.agents.BetaManagedAgentsWebFetchToolConfigParams;
      import com.anthropic.models.beta.agents.BetaManagedAgentsWebSearchToolConfigParams;
      import io.modelcontextprotocol.client.McpClient;
      import io.modelcontextprotocol.client.transport.HttpClientStreamableHttpTransport;
      import io.modelcontextprotocol.spec.McpSchema;

      // MCP 字段与自定义工具声明一一对应。MCP Java
      // SDK 仅保留工具输入 schema 的以下顶层关键字：type、
      // properties、required、additionalProperties、$defs 和 definitions。
      BetaManagedAgentsCustomToolParams toCustomTool(McpSchema.Tool tool) {
          var description = tool.description();
          if (description == null || description.isEmpty()) {
              description = tool.name();
          }

          return BetaManagedAgentsCustomToolParams.builder()
              .name(tool.name())
              .description(description)
              .inputSchema(JsonValue.from(tool.inputSchema())
                  .convert(BetaManagedAgentsCustomToolInputSchema.class))
              .build();
      }

      void main() {
          // 请在您创建 agent 的地方运行此代码，而非在 worker 主机上：它
          // 使用您的 Claude API 密钥（ANTHROPIC_API_KEY）进行身份验证。
          var client = AnthropicOkHttpClient.fromEnv();

          // MCP 服务器的地址，然后是其 MCP 端点的路径。
          var transport = HttpClientStreamableHttpTransport.builder("http://mcp.internal.example.com:8000")
              .endpoint("/mcp")
              .build();
          try (var mcpClient = McpClient.sync(transport)
                  .clientInfo(new McpSchema.Implementation("declare-agent-tools", "1.0.0"))
                  .build()) {
              mcpClient.initialize();

              var params = AgentCreateParams.builder()
                  .name("Internal tools agent")
                  .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
                  .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
                      .addConfig(BetaManagedAgentsWebSearchToolConfigParams.builder()
                          .enabled(false)
                          .build())
                      .addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()
                          .enabled(false)
                          .build())
                      .build());
              for (var tool : mcpClient.listTools().tools()) {
                  params.addTool(toCustomTool(tool));
              }

              var agent = client.beta().agents().create(params.build());
              IO.println(agent.id());
          }
      }
      ```

      ```php PHP
      use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
      use Anthropic\Beta\Agents\BetaManagedAgentsModel;
      use Anthropic\Beta\Agents\BetaManagedAgentsWebFetchToolConfigParams;
      use Anthropic\Beta\Agents\BetaManagedAgentsWebSearchToolConfigParams;
      use Anthropic\Client as Anthropic;
      use Mcp\Client as McpClient;
      use Mcp\Client\Transport\HttpTransport;

      $mcpServerUrl = 'http://mcp.internal.example.com:8000/mcp';

      // 请在您创建 agent 的地方运行此代码，而非在 worker 主机上：它
      // 使用您的 Claude API 密钥（ANTHROPIC_API_KEY）进行身份验证。
      $client = new Anthropic();

      $mcpClient = McpClient::builder()->build();
      $mcpClient->connect(new HttpTransport($mcpServerUrl));

      $tools = [
          BetaManagedAgentsAgentToolset20260401Params::with(
              configs: [
                  BetaManagedAgentsWebSearchToolConfigParams::with(enabled: false),
                  BetaManagedAgentsWebFetchToolConfigParams::with(enabled: false),
              ],
          ),
      ];
      foreach ($mcpClient->listTools()->tools as $tool) {
          // MCP 字段与自定义工具声明一一对应。
          // schema 的关键字会原样传递给 SDK。
          $tools[] = [
              'type' => 'custom',
              'name' => $tool->name,
              'description' => $tool->description ?: $tool->name,
              'inputSchema' => $tool->inputSchema,
          ];
      }

      $agent = $client->beta->agents->create(
          name: 'Internal tools agent',
          model: BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
          tools: $tools,
      );
      echo $agent->id, "\n";

      $mcpClient->disconnect();
      ```

      ```ruby Ruby
      require "anthropic"
      require "mcp"

      MCP_SERVER_URL = "http://mcp.internal.example.com:8000/mcp"

      # 请在您创建 agent 的地方运行此代码，而非在 worker 主机上：它
      # 使用您的 Claude API 密钥（ANTHROPIC_API_KEY）进行身份验证。
      client = Anthropic::Client.new

      transport = MCP::Client::HTTP.new(url: MCP_SERVER_URL)
      mcp_client = MCP::Client.new(transport: transport)
      mcp_client.connect

      custom_tools = mcp_client.tools.map do |tool|
        description = tool.description
        description = tool.name if description.nil? || description.empty?

        # MCP 字段与自定义工具声明一一对应。
        # schema 的关键字会原样传递给 SDK。
        {
          type: :custom,
          name: tool.name,
          description: description,
          input_schema: tool.input_schema
        }
      end

      agent = client.beta.agents.create(
        name: "Internal tools agent",
        model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
        tools: [
          {
            type: :agent_toolset_20260401,
            configs: [
              {name: :web_search, enabled: false},
              {name: :web_fetch, enabled: false}
            ]
          },
          *custom_tools
        ]
      )
      puts agent.id

      transport.close
      ```
    </CodeGroup>
  </Step>

  <Step title="从 worker 提供工具">
    在启动时连接到同一个 MCP 服务器，使用 `async_mcp_tool()` (typescript: `mcpTools()`; go: `mcp.NewBetaTools()`) 转换其工具，并将它们与 `beta_agent_toolset_20260401()` (typescript: `betaAgentToolset20260401()`; go: `agenttoolset.BetaAgentToolset20260401()`) 一起注册。在 worker 的整个生命周期内保持一个 MCP 会话处于打开状态。

    <CodeGroup exclude="shell, csharp, java, php, ruby">
      ```python Python
      import asyncio
      import os
      from datetime import timedelta
      from anthropic import AsyncAnthropic
      from anthropic.lib.environments import EnvironmentWorker
      from anthropic.lib.tools.agent_toolset import beta_agent_toolset_20260401
      from anthropic.lib.tools.mcp import async_mcp_tool
      from mcp import ClientSession

      # 需要 mcp >= 1.24，该版本将 streamablehttp_client 重命名为 streamable_http_client。
      from mcp.client.streamable_http import streamable_http_client

      MCP_SERVER_URL = "http://mcp.internal.example.com:8000/mcp"


      async def main() -> None:
          environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
          environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
          # 在启动时连接一次 MCP 服务器，并在工作进程的整个生命周期内
          # 保持会话打开。超时设置会将挂起的工具调用转为错误
          # 结果，而不是让调用一直停滞。
          async with (
              streamable_http_client(MCP_SERVER_URL) as (read, write, _),
              ClientSession(
                  read, write, read_timeout_seconds=timedelta(seconds=60)
              ) as mcp_session,
              AsyncAnthropic(auth_token=environment_key) as client,
          ):
              await mcp_session.initialize()
              listed = await mcp_session.list_tools()
              mcp_tools = [async_mcp_tool(tool, mcp_session) for tool in listed.tools]
              await EnvironmentWorker(
                  client,
                  environment_id=environment_id,
                  environment_key=environment_key,
                  workdir="/workspace",
                  tools=lambda env: [*beta_agent_toolset_20260401(env), *mcp_tools],
              ).run()


      asyncio.run(main())
      ```

      ```typescript TypeScript
      import Anthropic from "@anthropic-ai/sdk";
      import { EnvironmentWorker } from "@anthropic-ai/sdk/helpers/beta/environments";
      import {
        mcpTools,
        type MCPCallToolResultLike,
        type MCPClientLike
      } from "@anthropic-ai/sdk/helpers/beta/mcp";
      import { betaAgentToolset20260401 } from "@anthropic-ai/sdk/tools/agent-toolset/node";
      import { Client } from "@modelcontextprotocol/sdk/client/index.js";
      import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

      const MCP_SERVER_URL = "http://mcp.internal.example.com:8000/mcp";

      const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
      const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
      const client = new Anthropic({ authToken: environmentKey });
      const controller = new AbortController();
      process.once("SIGTERM", () => controller.abort());

      // 在启动时连接一次 MCP 服务器，并在工作进程的整个生命周期内
      // 保持连接打开。
      const mcpClient = new Client({ name: "sandbox-worker", version: "1.0.0" });
      await mcpClient.connect(new StreamableHTTPClientTransport(new URL(MCP_SERVER_URL)));
      const { tools } = await mcpClient.listTools();

      // MCP SDK 的 callTool 返回类型仍包含 mcpTools 不接受的旧版结果形状；
      // 需要收窄类型。待 MCPClientLike 放宽后删除此处理。
      const mcpClientForTools: MCPClientLike = {
        callTool: (params) => mcpClient.callTool(params) as Promise<MCPCallToolResultLike>
      };

      await new EnvironmentWorker({
        client,
        environmentId,
        environmentKey,
        workdir: "/workspace",
        signal: controller.signal,
        tools: (ctx) => [...betaAgentToolset20260401(ctx), ...mcpTools(tools, mcpClientForTools)]
      }).run();
      ```

      <CodeGroupItem language="C#">
        C# SDK 没有 `EnvironmentWorker`。
      </CodeGroupItem>

      ```go Go
      package main

      import (
      	"context"
      	"log"
      	"os"
      	"os/signal"
      	"syscall"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	"github.com/anthropics/anthropic-sdk-go/v2/lib/environments"
      	"github.com/anthropics/anthropic-sdk-go/v2/mcp"
      	"github.com/anthropics/anthropic-sdk-go/v2/option"
      	"github.com/anthropics/anthropic-sdk-go/v2/tools/agenttoolset"
      	mcpsdk "github.com/modelcontextprotocol/go-sdk/mcp"
      )

      const mcpServerURL = "http://mcp.internal.example.com:8000/mcp"

      func main() {
      	environmentKey := os.Getenv("ANTHROPIC_ENVIRONMENT_KEY")
      	environmentID := os.Getenv("ANTHROPIC_ENVIRONMENT_ID")

      	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
      	defer stop()

      	client := anthropic.NewClient(option.WithAuthToken(environmentKey))

      	// 在启动时连接一次 MCP 服务器，并在工作进程的整个生命周期内
      	// 保持会话打开。
      	mcpClient := mcpsdk.NewClient(&mcpsdk.Implementation{Name: "sandbox-worker", Version: "1.0.0"}, nil)
      	session, err := mcpClient.Connect(ctx, &mcpsdk.StreamableClientTransport{Endpoint: mcpServerURL}, nil)
      	if err != nil {
      		log.Fatalf("connect to MCP server: %v", err)
      	}
      	defer session.Close()

      	listed, err := session.ListTools(ctx, nil)
      	if err != nil {
      		log.Fatalf("list MCP tools: %v", err)
      	}
      	mcpTools, err := mcp.NewBetaTools(listed.Tools, session)
      	if err != nil {
      		log.Fatalf("convert MCP tools: %v", err)
      	}

      	worker := environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{
      		EnvironmentID:  environmentID,
      		EnvironmentKey: environmentKey,
      		Workdir:        "/workspace",
      		ToolsFunc: func(env *agenttoolset.AgentToolContext) []anthropic.BetaTool {
      			return append(agenttoolset.BetaAgentToolset20260401(env), mcpTools...)
      		},
      	})
      	if err := worker.Run(ctx); err != nil {
      		log.Fatalf("worker: %v", err)
      	}
      }
      ```

      <CodeGroupItem language="Java">
        Java SDK 没有 `EnvironmentWorker`。
      </CodeGroupItem>

      <CodeGroupItem language="PHP">
        PHP SDK 没有 `EnvironmentWorker`。
      </CodeGroupItem>

      <CodeGroupItem language="Ruby">
        Ruby SDK 没有 `EnvironmentWorker`。
      </CodeGroupItem>
    </CodeGroup>
  </Step>
</Steps>

## 限制与行为

### 工具需要声明，而非在运行时发现

worker 在启动时列出一次 MCP 服务器的工具，并且无法向正在运行的会话添加工具。当服务器的工具发生变化时：

1. 重新声明这些工具，可以在智能体上声明，也可以通过[更新智能体配置](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#updating-the-agent-configuration)在空闲会话上声明。
2. 重启 worker。

### 声明必须符合 Managed Agents API 的要求

MCP 辅助工具会保留服务器的名称和描述，大多数 schema 会原样传递。如果某个声明违反了以下规则之一，请进行重命名、精简或内联：

| 字段              | 规则                                                                                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`          | 在每个智能体中唯一。由字母、数字、下划线和连字符组成，长度为 1–128 个字符。不能与内置智能体工具（如 `bash` 或 `read`）同名，也不能使用保留的 `mcp__` 前缀或 `ant__` 前缀。                                                     |
| `description`   | 必填且不能为空。                                                                                                                                                      |
| `input_schema`  | 接受 MCP 服务器常用的 JSON Schema 关键字，例如 `additionalProperties` 和 `title`。拒绝任何位置的引用关键字（如 `$ref`），以及顶层的 `oneOf`、`anyOf` 和 `allOf`。属性名称由字母、数字、下划线、点和连字符组成，长度为 1–64 个字符。 |
| 智能体的 `tools` 数组 | 最多 256 个条目。每个封装的工具占一个条目，内置工具集再占一个。                                                                                                                            |

有两种情况需要额外处理：

* **两个服务器公开了相同的工具名称：** 自行以带前缀的名称定义封装器，并让它调用服务器的原始工具名称。
* **Pydantic 等生成器将 schema 提取到 `$defs` 中：** 在声明工具之前，先将这些 schema 内联。

### 工具失败以错误工具结果的形式呈现

当 MCP 服务器报告工具错误时，worker 会发布一个模型可以做出响应的错误工具结果。没有对应工具结果形式的 MCP 内容（例如音频块和资源链接）也会以错误形式呈现。

请在 MCP 客户端上设置超时，以获得更快、更清晰的失败反馈，就像 Python worker 示例中使用 `read_timeout_seconds` 所做的那样。有关未设置超时时会发生的情况，请参阅[封装的 MCP 工具调用挂起](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#a-wrapped-mcp-tool-call-hangs)。

### 仅封装您运营或信任的服务器

封装工具的名称、描述和结果会像任何其他工具一样进入模型的上下文。它们属于不受信任的输入，可能会影响智能体如何使用其他工具，包括 worker 主机上的 `bash`。请仅声明您希望智能体使用的工具。

### 权限策略不适用

[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#custom-tools)管控的是内置工具集和 MCP 工具集。worker 会运行模型发起的每一个封装工具调用，因此请将任何审批步骤放在您自己的工具代码中。
