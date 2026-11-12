---
title: 迁移
url: https://platform.claude.com/docs/zh-CN/managed-agents/migration
description: 将基于 Messages API 或 Claude Agent SDK 构建的现有智能体迁移到 Claude Managed Agents。
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

Claude Managed Agents 用托管基础设施取代您手写的智能体循环。本页介绍从基于 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 构建的自定义循环或从 [Claude Agent SDK](https://code.claude.com/docs/zh-CN/agent-sdk/overview) 迁移时会发生哪些变化。

## 从 Messages API 智能体循环迁移

如果您通过在循环中调用 `client.messages.create()` (csharp: `client.Messages.Create()`; go: `client.Messages.New()`; java: `client.messages().create()`; php: `$client->messages->create()`; cli: `ant messages create`; curl: `POST /v1/messages`)、自行运行工具调用并将结果追加到对话历史中来构建智能体，那么其中大部分代码都可以删除。

### 您不再需要管理的内容

| 之前                                                   | 之后                                                     |
| ---------------------------------------------------- | ------------------------------------------------------ |
| 您维护对话历史数组，并在每一轮将其传回。                                 | 会话在服务器端存储历史。您发送事件并接收事件。                                |
| 您遍历 `tool_use` 内容块，运行每个工具，然后带着 `tool_result` 消息回到循环。 | 预构建工具在沙箱内自动运行。您只需通过 `agent.custom_tool_use` 事件处理自定义工具。 |
| 您自行配置沙箱来运行智能体生成的代码。                                  | 会话沙箱负责处理代码执行、文件操作和 bash。                               |
| 您决定循环何时结束。                                           | 当智能体没有更多事情要做时，会话会发出 `session.status_idle`。             |

### 代码对比

**之前**（Messages API 循环，简化版）：

<CodeGroup exclude="shell:cURL, shell:CLI">
  ```python Python
  messages = [{"role": "user", "content": task}]
  while True:
      response = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=messages,
          tools=tools,
      )
      messages.append({"role": "assistant", "content": response.content})
      if response.stop_reason == "end_turn":
          break
      for block in response.content:
          if block.type == "tool_use":
              result = execute_tool(block.name, block.input)
              messages.append(
                  {
                      "role": "user",
                      "content": [
                          {
                              "type": "tool_result",
                              "tool_use_id": block.id,
                              "content": result,
                          }
                      ],
                  }
              )
  ```

  ```typescript TypeScript
  const messages: Anthropic.MessageParam[] = [{ role: "user", content: task }];
  while (true) {
    const response = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages,
      tools
    });
    messages.push({ role: "assistant", content: response.content });
    if (response.stop_reason === "end_turn") {
      break;
    }
    for (const block of response.content) {
      if (block.type === "tool_use") {
        const result = executeTool(block.name, block.input);
        messages.push({
          role: "user",
          content: [
            {
              type: "tool_result",
              tool_use_id: block.id,
              content: result
            }
          ]
        });
      }
    }
  }
  ```

  ```csharp C#
  List<MessageParam> messages = [new UserMessageParam { Content = task }];
  while (true)
  {
      var response = await client.Messages.Create(new()
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = messages,
          Tools = tools,
      });
      messages.Add(new AssistantMessageParam
      {
          Content = new([.. response.Content.Select(block => new ContentBlockParam(block.Json))]),
      });
      if (response.StopReason == StopReason.EndTurn)
      {
          break;
      }
      foreach (var block in response.Content)
      {
          if (block.Value is ToolUseBlock toolUse)
          {
              var result = ExecuteTool(toolUse.Name, toolUse.Input);
              messages.Add(new UserMessageParam
              {
                  Content = new([new ToolResultBlockParam { ToolUseID = toolUse.ID, Content = result }]),
              });
          }
      }
  }
  ```

  ```go Go
  messages := []anthropic.MessageParam{
  	anthropic.NewUserMessage(anthropic.NewTextBlock(task)),
  }
  for {
  	response, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Messages:  messages,
  		Tools:     tools,
  	})
  	if err != nil {
  		log.Fatal(err)
  	}
  	messages = append(messages, response.ToParam())
  	if response.StopReason == anthropic.StopReasonEndTurn {
  		break
  	}
  	for _, block := range response.Content {
  		if toolUse, ok := block.AsAny().(anthropic.ToolUseBlock); ok {
  			result := executeTool(toolUse.Name, toolUse.Input)
  			messages = append(messages, anthropic.NewUserMessage(
  				anthropic.NewToolResultBlock(toolUse.ID, result, false),
  			))
  		}
  	}
  }
  ```

  ```java Java
  var messages = new ArrayList<MessageParam>();
  messages.add(MessageParam.ofUser(task));
  while (true) {
      var response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .messages(messages)
          .tools(tools)
          .build());
      messages.add(response.toParam());
      if (StopReason.END_TURN.equals(response.stopReason().orElse(null))) {
          break;
      }
      for (var block : response.content()) {
          block.toolUse().ifPresent(toolUse -> {
              var result = executeTool(toolUse.name(), toolUse._input());
              messages.add(MessageParam.ofUserOfContentBlockParams(List.of(
                  ContentBlockParam.ofToolResult(ToolResultBlockParam.builder()
                      .toolUseId(toolUse.id())
                      .content(result)
                      .build()))));
          });
      }
  }
  ```

  ```php PHP
  $messages = [['role' => 'user', 'content' => $task]];
  while (true) {
      $response = $client->messages->create(
          model: 'claude-opus-5-5',
          maxTokens: 1024,
          messages: $messages,
          tools: $tools,
      );
      $messages[] = ['role' => 'assistant', 'content' => $response->content];
      if ($response->stopReason === 'end_turn') {
          break;
      }
      foreach ($response->content as $block) {
          if ($block->type === 'tool_use') {
              $result = executeTool($block->name, $block->input);
              $messages[] = [
                  'role' => 'user',
                  'content' => [
                      [
                          'type' => 'tool_result',
                          'tool_use_id' => $block->id,
                          'content' => $result,
                      ],
                  ],
              ];
          }
      }
  }
  ```

  ```ruby Ruby
  messages = [{ role: "user", content: task }]
  loop do
    response = client.messages.create(
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: messages,
      tools: tools
    )
    messages << { role: "assistant", content: response.content }
    break if response.stop_reason == :end_turn
    response.content.each do |block|
      next unless block.type == :tool_use
      result = execute_tool(block.name, block.input)
      messages << {
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: block.id,
            content: result
          }
        ]
      }
    end
  end
  ```
</CodeGroup>

**之后**（Claude Managed Agents）：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Task Runner",
      model="claude-opus-5-5",
      tools=[{"type": "agent_toolset_20260401"}],
  )

  session = client.beta.sessions.create(
      agent={"type": "agent", "id": agent.id, "version": agent.version},
      environment_id=environment.id,
  )

  with client.beta.sessions.events.stream(session.id) as stream:
      client.beta.sessions.events.send(
          session.id,
          events=[{"type": "user.message", "content": [{"type": "text", "text": task}]}],
      )
      for event in stream:
          if event.type == "session.status_idle":
              break
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Task Runner",
    model: "claude-opus-5-5",
    tools: [{ type: "agent_toolset_20260401" }]
  });

  const session = await client.beta.sessions.create({
    agent: { type: "agent", id: agent.id, version: agent.version },
    environment_id: environment.id
  });

  const stream = await client.beta.sessions.events.stream(session.id);

  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.message",
        content: [{ type: "text", text: task }]
      }
    ]
  });

  for await (const event of stream) {
    if (event.type === "session.status_idle") {
      break;
    }
  }
  ```

  ```csharp C#
  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Task Runner",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      Tools = [new BetaManagedAgentsAgentToolset20260401Params()],
  });

  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = new BetaManagedAgentsAgentParams
      {
          ID = agent.ID,
          Version = agent.Version,
      },
      EnvironmentID = environment.ID,
  });

  var stream = client.Beta.Sessions.Events.StreamStreaming(session.ID);

  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content = [new BetaManagedAgentsTextBlock { Text = task }],
          },
      ],
  });

  await foreach (var streamEvent in stream)
  {
      if (streamEvent.Value is BetaManagedAgentsSessionStatusIdleEvent)
      {
          break;
      }
  }
  ```

  ```go Go
  	agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  		Name: "Task Runner",
  		Model: anthropic.BetaManagedAgentsModelConfigParams{
  			ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  		},
  		Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  			OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{},
  		}},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  		Agent: anthropic.BetaSessionNewParamsAgentUnion{
  			OfBetaManagedAgentsAgents: &anthropic.BetaManagedAgentsAgentParams{
  				ID:      agent.ID,
  				Version: anthropic.Int(agent.Version),
  			},
  		},
  		EnvironmentID: environment.ID,
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{})
  	defer stream.Close()

  	_, err = client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  		Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  					OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  						Text: task,
  					},
  				}},
  			},
  		}},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	for stream.Next() {
  		event := stream.Current()
  		if event.Type == "session.status_idle" {
  			break
  		}
  	}
  	if err := stream.Err(); err != nil {
  		log.Fatal(err)
  	}
  ```

  ```java Java
      var agent = client.beta().agents().create(
          AgentCreateParams.builder()
              .name("Task Runner")
              .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
              .addTool(BetaManagedAgentsAgentToolset20260401Params.builder().build())
              .build()
      );

      var session = client.beta().sessions().create(
          SessionCreateParams.builder()
              .agent(
                  BetaManagedAgentsAgentParams.builder()
                      .id(agent.id())
                      .version(agent.version())
                      .build()
              )
              .environmentId(environment.id())
              .build()
      );

      try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
          client.beta().sessions().events().send(
              session.id(),
              EventSendParams.builder()
                  .addEvent(
                      BetaManagedAgentsUserMessageEventParams.builder()
                          .addTextContent(task)
                          .build()
                  )
                  .build()
          );
          stream.stream()
              .takeWhile(event -> !event.isSessionStatusIdle())
              .forEach(_ -> {});
      }
  ```

  ```php PHP
  $agent = $client->beta->agents->create(
      name: 'Task Runner',
      model: 'claude-opus-5-5',
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(),
      ],
  );

  $session = $client->beta->sessions->create(
      agent: BetaManagedAgentsAgentParams::with(
          id: $agent->id,
          version: $agent->version,
      ),
      environmentID: $environment->id,
  );

  $stream = $client->beta->sessions->events->streamStream($session->id);

  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => $task]],
          ],
      ],
  );

  foreach ($stream as $event) {
      if ($event->type === 'session.status_idle') {
          break;
      }
  }
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Task Runner",
    model: "claude-opus-5-5",
    tools: [{type: "agent_toolset_20260401"}]
  )

  session = client.beta.sessions.create(
    agent: {type: "agent", id: agent.id, version: agent.version},
    environment_id: environment.id
  )

  stream = client.beta.sessions.events.stream_events(session.id)
  client.beta.sessions.events.send_(
    session.id,
    events: [{type: "user.message", content: [{type: "text", text: task}]}]
  )
  stream.each do
    break if it.type == :"session.status_idle"
  end
  ```
</CodeGroup>

### 您仍然可以控制的内容

* **系统提示和模型：** 字段相同，现在位于智能体定义上。
* **自定义工具：** 仍然使用 JSON Schema 声明。执行方式从内联处理改为响应 `agent.custom_tool_use` 事件。请参阅[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。
* **网页搜索和网页抓取设置：** 相同的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`)、`blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`)、`max_content_tokens` (java, php: `maxContentTokens`; csharp, go: `MaxContentTokens`) 和 `user_location` (java, php: `userLocation`; csharp, go: `UserLocation`) 字段，现在只需在智能体工具集的 `configs` (csharp, go: `Configs`) 数组中的 `web_search` 和 `web_fetch` 条目上设置一次，而无需在每个请求中设置。`max_uses` (java, php: `maxUses`; csharp, go: `MaxUses`)、`citations` (csharp, go: `Citations`) 和 `cache_control` (java, php: `cacheControl`; csharp, go: `CacheControl`) 字段不可用。请参阅[与 Messages API 工具的区别](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#differences-from-the-messages-api-tools)。
* **上下文：** 您仍然可以通过系统提示、[文件资源](https://platform.claude.com/docs/zh-CN/managed-agents/files)或[技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills)注入上下文。

## 从 Claude Agent SDK 迁移

如果您使用 [Claude Agent SDK](https://code.claude.com/docs/zh-CN/agent-sdk/overview) 构建，那么您已经在以概念的形式使用智能体、工具和会话。区别在于它们的运行位置：SDK 在您运维的进程中运行，而 Managed Agents 在 Anthropic 的基础设施中运行。迁移的主要工作是将 SDK 配置对象映射到其 API 端的等效项。

### 发生的变化

| Agent SDK                                                                                  | Managed Agents                                                                                                                                                                                                                                                                                                     |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 每次运行时构造 `ClaudeAgentOptions(...)` (typescript: `options`)                                  | 只需调用一次 `client.beta.agents.create(...)` (csharp: `client.Beta.Agents.Create(...)`; go: `client.Beta.Agents.New(...)`; java: `client.beta().agents().create(...)`; php: `$client->beta->agents->create(...)`)；Agent 会在服务器端持久化并进行版本管理。请参阅[智能体设置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)。 |
| `async with ClaudeSDKClient(...)` 或 `query(...)`                                           | 调用 `client.beta.sessions.create(...)` (csharp: `client.Beta.Sessions.Create(...)`; go: `client.Beta.Sessions.New(...)`; java: `client.beta().sessions().create(...)`; php: `$client->beta->sessions->create(...)`)，然后发送和接收[事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。        |
| 使用 `@tool` (typescript: `tool()`) 定义的函数，由 SDK 自动分派                                         | 在 Agent 上声明为 `{"type": "custom", ...}`；您的客户端处理 `agent.custom_tool_use` 事件并以 `user.custom_tool_result` 回复。请参阅[工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools)。                                                                                                                                     |
| 内置工具在您的进程中针对您的文件系统运行                                                                       | `{"type": "agent_toolset_20260401"}` 在会话沙箱内针对 `/workspace` 运行相同的工具。                                                                                                                                                                                                                                                |
| `cwd`、`add_dirs` (typescript: `additionalDirectories`) 指向本地路径                              | 将[文件](https://platform.claude.com/docs/zh-CN/managed-agents/files)作为会话资源上传或挂载。                                                                                                                                                                                                                                     |
| `system_prompt` (typescript: `systemPrompt`) 和 `CLAUDE.md` 层级结构                            | Agent 上的单个 `system` (csharp, go: `System`; ruby: `system_`) 字符串。每次更改智能体的更新都会在服务器端生成一个新版本；将会话固定到特定版本，即可在无需部署的情况下进行升级或回滚。请参阅[智能体设置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)。                                                                                                              |
| `mcp_servers` (typescript: `mcpServers`) 在同一处配置和认证                                         | 在 Agent 上声明服务器；通过 Session 上的 [Vault](https://platform.claude.com/docs/zh-CN/managed-agents/vaults) 提供凭据。                                                                                                                                                                                                           |
| `permission_mode` (typescript: `permissionMode`)、`can_use_tool` (typescript: `canUseTool`) | 按工具设置的 [`permission_policy` (java, php: `permissionPolicy`; csharp, go: `PermissionPolicy`)](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)（`always_allow`、`always_ask` 或 `auto`）；对于暂停等待您批准的调用，发送 `user.tool_confirmation` 事件。                                                         |

### 代码对比

**之前**（Agent SDK）：

<CodeGroup exclude="shell, csharp, go, java, php, ruby">
  ```python Python
  from claude_agent_sdk import (
      ClaudeAgentOptions,
      ClaudeSDKClient,
      create_sdk_mcp_server,
      tool,
  )


  @tool("get_weather", "Get the current weather for a city.", {"city": str})
  async def get_weather(args: dict) -> dict:
      return {"content": [{"type": "text", "text": f"{args['city']}: 18°C, clear"}]}


  options = ClaudeAgentOptions(
      model="claude-opus-5-5",
      system_prompt="You are a concise weather assistant.",
      mcp_servers={
          "weather": create_sdk_mcp_server("weather", "1.0", tools=[get_weather])
      },
  )

  async with ClaudeSDKClient(options=options) as agent:
      await agent.query("What's the weather in Tokyo?")
      async for msg in agent.receive_response():
          print(msg)
  ```

  ```typescript TypeScript
  import { createSdkMcpServer, query, tool } from "@anthropic-ai/claude-agent-sdk";
  import { z } from "zod";

  const getWeather = tool(
    "get_weather",
    "Get the current weather for a city.",
    { city: z.string() },
    async (args) => ({
      content: [{ type: "text", text: `${args.city}: 18°C, clear` }]
    })
  );

  for await (const message of query({
    prompt: "What's the weather in Tokyo?",
    options: {
      model: "claude-opus-5-5",
      systemPrompt: "You are a concise weather assistant.",
      mcpServers: {
        weather: createSdkMcpServer({ name: "weather", version: "1.0", tools: [getWeather] })
      }
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

**之后**（Managed Agents）：

<CodeGroup exclude="shell">
  ```python Python
  from anthropic import Anthropic

  client = Anthropic()

  agent = client.beta.agents.create(
      name="weather-agent",
      model="claude-opus-5-5",
      system="You are a concise weather assistant.",
      tools=[
          {
              "type": "custom",
              "name": "get_weather",
              "description": "Get the current weather for a city.",
              "input_schema": {
                  "type": "object",
                  "properties": {"city": {"type": "string"}},
                  "required": ["city"],
              },
          }
      ],
  )
  environment = client.beta.environments.create(
      name="weather-env",
      config={
          "type": "cloud",
          "networking": {"type": "limited", "allow_package_managers": True},
      },
  )

  session = client.beta.sessions.create(
      agent={"type": "agent", "id": agent.id, "version": agent.version},
      environment_id=environment.id,
  )


  def get_weather(city: str) -> str:
      return f"{city}: 18°C, clear"


  with client.beta.sessions.events.stream(session.id) as stream:
      client.beta.sessions.events.send(
          session.id,
          events=[
              {
                  "type": "user.message",
                  "content": [{"type": "text", "text": "What's the weather in Tokyo?"}],
              }
          ],
      )
      for event in stream:
          match event.type:
              case "agent.message":
                  print(
                      "".join(
                          block.text for block in event.content if block.type == "text"
                      )
                  )
              case "agent.custom_tool_use":
                  result = get_weather(**event.input)
                  client.beta.sessions.events.send(
                      session.id,
                      events=[
                          {
                              "type": "user.custom_tool_result",
                              "custom_tool_use_id": event.id,
                              "content": [{"type": "text", "text": result}],
                          }
                      ],
                  )
              case "session.status_idle":
                  if event.stop_reason and event.stop_reason.type == "end_turn":
                      break
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic();

  const agent = await client.beta.agents.create({
    name: "weather-agent",
    model: "claude-opus-5-5",
    system: "You are a concise weather assistant.",
    tools: [
      {
        type: "custom",
        name: "get_weather",
        description: "Get the current weather for a city.",
        input_schema: {
          type: "object",
          properties: { city: { type: "string" } },
          required: ["city"]
        }
      }
    ]
  });
  const environment = await client.beta.environments.create({
    name: "weather-env",
    config: {
      type: "cloud",
      networking: { type: "limited", allow_package_managers: true }
    }
  });

  const session = await client.beta.sessions.create({
    agent: { type: "agent", id: agent.id, version: agent.version },
    environment_id: environment.id
  });

  function getWeather({ city }: Record<string, unknown>): string {
    return `${city}: 18°C, clear`;
  }

  const stream = await client.beta.sessions.events.stream(session.id);

  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "What's the weather in Tokyo?" }]
      }
    ]
  });

  loop: for await (const event of stream) {
    switch (event.type) {
      case "agent.message":
        for (const block of event.content) {
          if (block.type === "text") {
            console.log(block.text);
          }
        }
        break;
      case "agent.custom_tool_use": {
        const result = getWeather(event.input);
        await client.beta.sessions.events.send(session.id, {
          events: [
            {
              type: "user.custom_tool_result",
              custom_tool_use_id: event.id,
              content: [{ type: "text", text: result }]
            }
          ]
        });
        break;
      }
      case "session.status_idle":
        if (event.stop_reason?.type === "end_turn") {
          break loop;
        }
        break;
    }
  }
  ```

  ```csharp C#
  using System.Text.Json;

  using Anthropic.Models.Beta.Agents;
  using Anthropic.Models.Beta.Environments;
  using Anthropic.Models.Beta.Sessions;
  using Anthropic.Models.Beta.Sessions.Events;

  AnthropicClient client = new();

  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "weather-agent",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      System = "You are a concise weather assistant.",
      Tools =
      [
          new BetaManagedAgentsCustomToolParams
          {
              Name = "get_weather",
              Description = "Get the current weather for a city.",
              InputSchema = new()
              {
                  Properties = new Dictionary<string, JsonElement>
                  {
                      ["city"] = JsonSerializer.SerializeToElement(new { type = "string" }),
                  },
                  Required = ["city"],
              },
          },
      ],
  });
  var environment = await client.Beta.Environments.Create(new()
  {
      Name = "weather-env",
      Config = new BetaCloudConfigParams
      {
          Networking = new BetaLimitedNetworkParams { AllowPackageManagers = true },
      },
  });

  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = new BetaManagedAgentsAgentParams
      {
          ID = agent.ID,
          Version = agent.Version,
      },
      EnvironmentID = environment.ID,
  });

  static string GetWeather(string city) => $"{city}: 18°C, clear";

  using var stream = await client.Beta.Sessions.Events.WithRawResponse.StreamStreaming(session.ID);

  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content = [new BetaManagedAgentsTextBlock { Text = "What's the weather in Tokyo?" }],
          },
      ],
  });

  await foreach (var streamEvent in stream.Enumerate())
  {
      if (streamEvent.Value is BetaManagedAgentsAgentMessageEvent message)
      {
          var text = string.Concat(message.Content.Select(block =>
              block.Value is BetaManagedAgentsTextBlock textBlock ? textBlock.Text : ""));
          Console.WriteLine(text);
      }
      else if (streamEvent.Value is BetaManagedAgentsAgentCustomToolUseEvent toolUse)
      {
          var result = GetWeather(toolUse.Input["city"].GetString()!);
          await client.Beta.Sessions.Events.Send(session.ID, new()
          {
              Events =
              [
                  new BetaManagedAgentsUserCustomToolResultEventParams
                  {
                      CustomToolUseID = toolUse.ID,
                      Content = [new BetaManagedAgentsTextBlock { Text = result }],
                  },
              ],
          });
      }
      else if (streamEvent.Value is BetaManagedAgentsSessionStatusIdleEvent idle
          && idle.StopReason?.Value is BetaManagedAgentsSessionEndTurn)
      {
          break;
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()
  ctx := context.Background()

  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "weather-agent",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	System: anthropic.String("You are a concise weather assistant."),
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfCustom: &anthropic.BetaManagedAgentsCustomToolParams{
  			Name:        "get_weather",
  			Description: "Get the current weather for a city.",
  			InputSchema: anthropic.BetaManagedAgentsCustomToolInputSchemaParam{
  				Properties: map[string]any{
  					"city": map[string]any{"type": "string"},
  				},
  				Required: []string{"city"},
  			},
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }
  environment, err := client.Beta.Environments.New(ctx, anthropic.BetaEnvironmentNewParams{
  	Name: "weather-env",
  	Config: anthropic.BetaEnvironmentNewParamsConfigUnion{
  		OfCloud: &anthropic.BetaCloudConfigParams{
  			Networking: anthropic.BetaCloudConfigParamsNetworkingUnion{
  				OfLimited: &anthropic.BetaLimitedNetworkParams{
  					AllowPackageManagers: anthropic.Bool(true),
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }

  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfBetaManagedAgentsAgents: &anthropic.BetaManagedAgentsAgentParams{
  			ID:      agent.ID,
  			Version: anthropic.Int(agent.Version),
  		},
  	},
  	EnvironmentID: environment.ID,
  })
  if err != nil {
  	panic(err)
  }

  getWeather := func(city string) string {
  	return fmt.Sprintf("%s: 18°C, clear", city)
  }

  stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{})
  defer stream.Close()

  _, err = client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  			Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  				OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  					Text: "What's the weather in Tokyo?",
  				},
  			}},
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }

  loop:
  for stream.Next() {
  	event := stream.Current()
  	switch event.Type {
  	case "agent.message":
  		for _, block := range event.AsAgentMessage().Content {
  			if block.Type == "text" {
  				fmt.Println(block.Text)
  			}
  		}
  	case "agent.custom_tool_use":
  		toolUse := event.AsAgentCustomToolUse()
  		result := getWeather(toolUse.Input["city"].(string))
  		if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  			Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  				OfUserCustomToolResult: &anthropic.BetaManagedAgentsUserCustomToolResultEventParams{
  					CustomToolUseID: toolUse.ID,
  					Content: []anthropic.BetaManagedAgentsUserCustomToolResultEventParamsContentUnion{{
  						OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  							Text: result,
  						},
  					}},
  				},
  			}},
  		}); err != nil {
  			panic(err)
  		}
  	case "session.status_idle":
  		idle := event.AsSessionStatusIdle()
  		if _, ok := idle.StopReason.AsAny().(anthropic.BetaManagedAgentsSessionEndTurn); ok {
  			break loop
  		}
  	}
  }
  if err := stream.Err(); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  import java.util.Map;
  import java.util.function.Function;

  import com.anthropic.models.beta.agents.AgentCreateParams;
  import com.anthropic.models.beta.agents.BetaManagedAgentsCustomToolInputSchema;
  import com.anthropic.models.beta.agents.BetaManagedAgentsCustomToolParams;
  import com.anthropic.models.beta.agents.BetaManagedAgentsModel;
  import com.anthropic.models.beta.environments.BetaCloudConfigParams;
  import com.anthropic.models.beta.environments.BetaLimitedNetworkParams;
  import com.anthropic.models.beta.environments.EnvironmentCreateParams;
  import com.anthropic.models.beta.sessions.BetaManagedAgentsAgentParams;
  import com.anthropic.models.beta.sessions.SessionCreateParams;
  import com.anthropic.models.beta.sessions.events.BetaManagedAgentsUserCustomToolResultEventParams;
  import com.anthropic.models.beta.sessions.events.BetaManagedAgentsUserMessageEventParams;
  import com.anthropic.models.beta.sessions.events.EventSendParams;

  var client = AnthropicOkHttpClient.fromEnv();

  var agent = client.beta().agents().create(AgentCreateParams.builder()
      .name("weather-agent")
      .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
      .system("You are a concise weather assistant.")
      .addTool(BetaManagedAgentsCustomToolParams.builder()
          .name("get_weather")
          .description("Get the current weather for a city.")
          .inputSchema(BetaManagedAgentsCustomToolInputSchema.builder()
              .properties(BetaManagedAgentsCustomToolInputSchema.Properties.builder()
                  .putAdditionalProperty("city", JsonValue.from(Map.of("type", "string")))
                  .build())
              .addRequired("city")
              .build())
          .build())
      .build());
  var environment = client.beta().environments().create(EnvironmentCreateParams.builder()
      .name("weather-env")
      .config(BetaCloudConfigParams.builder()
          .networking(BetaLimitedNetworkParams.builder()
              .allowPackageManagers(true)
              .build())
          .build())
      .build());

  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(BetaManagedAgentsAgentParams.builder()
          .id(agent.id())
          .version(agent.version())
          .build())
      .environmentId(environment.id())
      .build());

  Function<String, String> getWeather = city -> city + ": 18°C, clear";

  try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
      client.beta().sessions().events().send(
          session.id(),
          EventSendParams.builder()
              .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
                  .addTextContent("What's the weather in Tokyo?")
                  .build())
              .build());

      loop:
      for (var event : stream) {
          switch (event.type().value()) {
              case AGENT_MESSAGE -> {
                  for (var block : event.asAgentMessage().content()) {
                      block.text().ifPresent(textBlock -> IO.println(textBlock.text()));
                  }
              }
              case AGENT_CUSTOM_TOOL_USE -> {
                  var toolUse = event.asAgentCustomToolUse();
                  var city = toolUse.input()._additionalProperties().get("city").asStringOrThrow();
                  var result = getWeather.apply(city);
                  client.beta().sessions().events().send(
                      session.id(),
                      EventSendParams.builder()
                          .addEvent(BetaManagedAgentsUserCustomToolResultEventParams.builder()
                              .customToolUseId(toolUse.id())
                              .addTextContent(result)
                              .build())
                          .build());
              }
              case SESSION_STATUS_IDLE -> {
                  if (event.asSessionStatusIdle().stopReason().isEndTurn()) {
                      break loop;
                  }
              }
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Client;
  use Anthropic\Beta\Agents\BetaManagedAgentsCustomToolInputSchema;
  use Anthropic\Beta\Agents\BetaManagedAgentsCustomToolParams;
  use Anthropic\Beta\Sessions\BetaManagedAgentsAgentParams;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsAgentCustomToolUseEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionEndTurn;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionStatusIdleEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock;

  $client = new Client();

  $agent = $client->beta->agents->create(
      name: 'weather-agent',
      model: 'claude-opus-5-5',
      system: 'You are a concise weather assistant.',
      tools: [
          BetaManagedAgentsCustomToolParams::with(
              name: 'get_weather',
              description: 'Get the current weather for a city.',
              inputSchema: BetaManagedAgentsCustomToolInputSchema::with(
                  properties: ['city' => ['type' => 'string']],
                  required: ['city'],
              ),
          ),
      ],
  );
  $environment = $client->beta->environments->create(
      name: 'weather-env',
      config: [
          'type' => 'cloud',
          'networking' => ['type' => 'limited', 'allow_package_managers' => true],
      ],
  );

  $session = $client->beta->sessions->create(
      agent: BetaManagedAgentsAgentParams::with(
          id: $agent->id,
          version: $agent->version,
      ),
      environmentID: $environment->id,
  );

  function getWeather(string $city): string
  {
      return "{$city}: 18°C, clear";
  }

  $stream = $client->beta->sessions->events->streamStream($session->id);

  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => "What's the weather in Tokyo?"]],
          ],
      ],
  );

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof ManagedAgentsAgentMessageEvent:
              foreach ($event->content as $block) {
                  if ($block instanceof ManagedAgentsTextBlock) {
                      echo $block->text . "\n";
                  }
              }
              break;
          case $event instanceof ManagedAgentsAgentCustomToolUseEvent:
              $result = getWeather($event->input['city']);
              $client->beta->sessions->events->send(
                  $session->id,
                  events: [
                      [
                          'type' => 'user.custom_tool_result',
                          'custom_tool_use_id' => $event->id,
                          'content' => [['type' => 'text', 'text' => $result]],
                      ],
                  ],
              );
              break;
          case $event instanceof ManagedAgentsSessionStatusIdleEvent:
              if ($event->stopReason instanceof ManagedAgentsSessionEndTurn) {
                  break 2;
              }
              break;
      }
  }
  $stream->close();
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::Client.new

  agent = client.beta.agents.create(
    name: "weather-agent",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
    system_: "You are a concise weather assistant.",
    tools: [
      {
        type: "custom",
        name: "get_weather",
        description: "Get the current weather for a city.",
        input_schema: {
          type: "object",
          properties: {city: {type: "string"}},
          required: ["city"]
        }
      }
    ]
  )
  environment = client.beta.environments.create(
    name: "weather-env",
    config: {type: "cloud", networking: {type: "limited", allow_package_managers: true}}
  )

  session = client.beta.sessions.create(
    agent: {type: "agent", id: agent.id, version: agent.version},
    environment_id: environment.id
  )

  def get_weather(city)
    "#{city}: 18°C, clear"
  end

  stream = client.beta.sessions.events.stream_events(session.id)
  client.beta.sessions.events.send_(
    session.id,
    events: [{type: "user.message", content: [{type: "text", text: "What's the weather in Tokyo?"}]}]
  )

  stream.each do |event|
    case event
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
      event.content.each do |block|
        puts block.text if block.is_a?(Anthropic::Beta::Sessions::BetaManagedAgentsTextBlock)
      end
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentCustomToolUseEvent
      result = get_weather(event.input[:city])
      client.beta.sessions.events.send_(
        session.id,
        events: [
          {
            type: "user.custom_tool_result",
            custom_tool_use_id: event.id,
            content: [{type: "text", text: result}]
          }
        ]
      )
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusIdleEvent
      break if event.stop_reason.is_a?(Anthropic::Beta::Sessions::BetaManagedAgentsSessionEndTurn)
    end
  end
  ```
</CodeGroup>

智能体和环境只需创建一次，即可在多个会话中重复使用。工具函数仍然在您的进程中运行；区别在于您需要读取 `agent.custom_tool_use` 事件并显式发送结果，而不是由 SDK 为您分派。

### 转移到您客户端的功能

由 Anthropic 运行智能体循环的代价是，SDK 原本自动处理的一些事项变成了您客户端的职责。

| SDK 功能                               | Managed Agents 的处理方式                                                                                                                                                                                                                                                                        |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 计划模式                                 | 先运行一个仅用于规划的会话，然后运行第二个会话来执行该计划。                                                                                                                                                                                                                                                              |
| 输出样式、斜杠命令                            | 在发送 `user.message` 之前或接收 `agent.message` 之后，在您的客户端中应用。                                                                                                                                                                                                                                      |
| `PreToolUse` / `PostToolUse` 钩子      | 您的客户端在响应之前已经能看到每个 `agent.custom_tool_use` 事件；将逻辑放在那里即可。对于内置工具，使用 `permission_policy: always_ask` 来审查每次调用。[`auto`](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#let-the-server-evaluate-each-call-with-auto) 则让服务器来评估每次调用，但如果服务器评估某次调用是安全的，该调用会直接运行，而不会到达您的客户端。 |
| `max_turns` (typescript: `maxTurns`) | 在客户端计算轮次。                                                                                                                                                                                                                                                                                   |

## 迁移清单

1. [创建一个环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments)，包含您的智能体所需的网络和运行时。
2. 将您的系统提示和工具选择移植到[智能体定义](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)中。
3. 替换您的循环：使用 `client.beta.sessions.create()` (go: `client.Beta.Sessions.New()`; csharp: `client.Beta.Sessions.Create()`; java: `client.beta().sessions().create()`; php: `$client->beta->sessions->create()`; cli: `ant beta:sessions create`; curl: `POST /v1/sessions`) [创建会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)，并使用 `client.beta.sessions.events.stream()` (ruby: `client.beta.sessions.events.stream_events()`; go: `client.Beta.Sessions.Events.StreamEvents()`; csharp: `client.Beta.Sessions.Events.StreamStreaming()`; java: `client.beta().sessions().events().streamStreaming()`; php: `$client->beta->sessions->events->streamStream()`; cli: `ant beta:sessions:events stream`; curl: `GET /v1/sessions/{session_id}/events/stream`) [流式传输其事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。
4. 对于智能体读取的任何本地文件，通过 [Files API](https://platform.claude.com/docs/zh-CN/managed-agents/files) 上传，并将其挂载为 `resources` (csharp, go: `Resources`)。
5. 对于任何自定义工具处理程序，将执行移入您的事件循环中，作为对 `agent.custom_tool_use` 事件的响应。
6. 在将生产流量指向新流程之前，先用测试会话进行验证。

## 在模型版本之间迁移

当新的 Claude 模型发布时，迁移 Claude Managed Agents 集成通常只需更改一个字段：更新您[智能体定义](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)上的 `model` (csharp, go: `Model`)，更改即会在您创建的下一个会话中生效。

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -sS --fail-with-body "https://api.anthropic.com/v1/agents/$AGENT_ID?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "version": '"$AGENT_VERSION"',
      "model": "claude-opus-5-5"
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply agent.md
    ```

    <File filename="agent.md">
      ```markdown
      ---
      name: Task Runner
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
      ---

      You are a task automation agent. Complete the task you are given end to end.
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  client.beta.agents.update(
      agent.id,
      version=agent.version,
      model="claude-opus-5-5",
  )
  ```

  ```typescript TypeScript
  await client.beta.agents.update(agent.id, {
    version: agent.version,
    model: "claude-opus-5-5"
  });
  ```

  ```csharp C#
  await client.Beta.Agents.Update(agent.ID, new()
  {
      Version = agent.Version,
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
  });
  ```

  ```go Go
  _, err = client.Beta.Agents.Update(ctx, agent.ID, anthropic.BetaAgentUpdateParams{
  	Version: anthropic.Int(agent.Version),
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().agents().update(
      agent.id(),
      AgentUpdateParams.builder()
          .version(agent.version())
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .build()
  );
  ```

  ```php PHP
  $client->beta->agents->update(
      $agent->id,
      version: $agent->version,
      model: 'claude-opus-5-5',
  );
  ```

  ```ruby Ruby
  client.beta.agents.update(
    agent.id,
    version: agent.version,
    model: "claude-opus-5-5"
  )
  ```
</CodeGroup>

[Messages API 迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)中记录的大多数模型级行为变化不需要您采取任何行动：

* **请求参数变化**（`max_tokens` (java, php: `maxTokens`; csharp, go: `MaxTokens`) 默认值、`thinking` (csharp, go: `Thinking`) 配置）由 Claude Managed Agents 运行时处理。这些字段不会在智能体定义上公开。
* **助手消息预填充**在基于事件的会话模型中不存在，因此在较新模型上移除该功能不会产生任何影响。
* **工具参数 JSON 转义**在您收到 `agent.custom_tool_use` 事件之前已由运行时解析。您看到的是结构化数据，而不是原始字符串。

Messages API 指南中的行为描述（模型有哪些不同的表现）仍然适用。迁移步骤（如何更改您的请求代码）则不适用。
