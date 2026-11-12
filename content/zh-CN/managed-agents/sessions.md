---
title: 启动会话
url: https://platform.claude.com/docs/zh-CN/managed-agents/sessions
description: 创建会话以运行您的代理并开始执行任务。
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

会话（session）是环境中的一个智能体实例。每个会话都引用一个[智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)和一个[环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments)（两者分别单独创建），并在多次交互中维护对话历史。会话遵循两步生命周期：首先[创建会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#creating-a-session)，然后[发送用户事件](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#starting-the-session)以开始工作。您也可以使用 [`initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`)](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#seed-the-session-with-initial-events) 将这两步合并为一次调用。

## 创建会话

会话需要一个 `agent` ID 和一个 `environment` ID。智能体是带版本的资源；以字符串形式传入 `agent` (csharp, go: `Agent`) ID 时，会使用最新的智能体版本创建会话。

<CodeGroup>
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'"
    }'
  ```

  ```bash CLI
  ant beta:sessions create --agent "$AGENT_ID" --environment-id "$ENVIRONMENT_ID"
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
  )
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id
  });
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
  });
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String(agent.ID),
  	},
  	EnvironmentID: environment.ID,
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(agent.id())
      .environmentId(environment.id())
      .build());
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
  );
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id
  )
  ```
</CodeGroup>

要将会话固定到特定的智能体版本，请传入一个对象。这样您可以精确控制运行哪个版本，并独立地分阶段推出新版本。

<CodeGroup>
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": {"type": "agent", "id": "'"$AGENT_ID"'", "version": 1},
      "environment_id": "'"$ENVIRONMENT_ID"'"
    }'
  ```

  ```bash CLI
  ant beta:sessions create <<YAML
  agent:
    type: agent
    id: $AGENT_ID
    version: 1
  environment_id: $ENVIRONMENT_ID
  YAML
  ```

  ```python Python
  pinned_session = client.beta.sessions.create(
      agent={"type": "agent", "id": agent.id, "version": 1},
      environment_id=environment.id,
  )
  ```

  ```typescript TypeScript
  const pinnedSession = await client.beta.sessions.create({
    agent: { type: "agent", id: agent.id, version: 1 },
    environment_id: environment.id
  });
  ```

  ```csharp C#
  var pinnedSession = await client.Beta.Sessions.Create(new()
  {
      Agent = new BetaManagedAgentsAgentParams
      {
          ID = agent.ID,
          Version = 1,
      },
      EnvironmentID = environment.ID,
  });
  ```

  ```go Go
  pinnedSession, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfBetaManagedAgentsAgents: &anthropic.BetaManagedAgentsAgentParams{
  			ID:      agent.ID,
  			Version: anthropic.Int(1),
  		},
  	},
  	EnvironmentID: environment.ID,
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var pinnedSession = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(BetaManagedAgentsAgentParams.builder()
          .id(agent.id())
          .version(1)
          .build())
      .environmentId(environment.id())
      .build());
  ```

  ```php PHP
  $pinnedSession = $client->beta->sessions->create(
      agent: ['type' => 'agent', 'id' => $agent->id, 'version' => 1],
      environmentID: $environment->id,
  );
  ```

  ```ruby Ruby
  pinned_session = client.beta.sessions.create(
    agent: {type: :agent, id: agent.id, version: 1},
    environment_id: environment.id
  )
  ```
</CodeGroup>

### 使用初始事件预置会话

您可以在一次调用中创建会话并启动其工作。`initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 是一个可选数组，包含在创建时发送给会话的初始[事件](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types)，这些事件按顺序处理。它支持 `user.message` 和 [`user.define_outcome`](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes) 事件，最多接受 50 个事件。非空列表会在同一次调用中启动智能体循环：会话直接以 `running` 状态创建，无需进一步请求。

以下示例创建一个会话，其 `initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 中包含单个 `user.message`：

<CodeGroup>
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'",
      "initial_events": [
        {
          "type": "user.message",
          "content": [{"type": "text", "text": "List the files in the working directory."}]
        }
      ]
    }'
  ```

  ```bash CLI
  ant beta:sessions create <<YAML
  agent: $AGENT_ID
  environment_id: $ENVIRONMENT_ID
  initial_events:
    - type: user.message
      content:
        - type: text
          text: List the files in the working directory.
  YAML
  ```

  ```python Python
  seeded_session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
      initial_events=[
          {
              "type": "user.message",
              "content": [
                  {"type": "text", "text": "List the files in the working directory."}
              ],
          },
      ],
  )
  # initial_events 不会在创建响应中回显；需从
  # 会话的事件列表中读回它们。
  for event in client.beta.sessions.events.list(seeded_session.id):
      if event.type == "user.message":
          for block in event.content:
              if block.type == "text":
                  print(f"Seeded event: {block.text}")
  ```

  ```typescript TypeScript
  const seededSession = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id,
    initial_events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "List the files in the working directory." }]
      }
    ]
  });

  // initial_events 不会在创建响应中回显；需列出会话的
  // 事件才能读回预置的消息。
  for await (const event of client.beta.sessions.events.list(seededSession.id)) {
    if (event.type === "user.message") {
      for (const block of event.content) {
        if (block.type === "text") {
          console.log(`Seeded event: ${block.text}`);
        }
      }
    }
  }
  ```

  ```csharp C#
  var seededSession = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
      InitialEvents =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content =
              [
                  new BetaManagedAgentsTextBlock
                  {
                      Text = "List the files in the working directory.",
                  },
              ],
          },
      ],
  });
  // initial_events 不会在创建响应中回显；请从
  // 会话的事件列表中读取它们。
  var seededEvents = await client.Beta.Sessions.Events.List(seededSession.ID);
  await foreach (var sessionEvent in seededEvents.Paginate())
  {
      if (sessionEvent.TryPickUserMessage(out var userMessage))
      {
          foreach (var contentBlock in userMessage.Content)
          {
              if (contentBlock.TryPickBetaManagedAgentsTextBlock(out var textBlock))
              {
                  Console.WriteLine($"Seeded event: {textBlock.Text}");
              }
          }
      }
  }
  ```

  ```go Go
  seededSession, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String(agent.ID),
  	},
  	EnvironmentID: environment.ID,
  	InitialEvents: []anthropic.BetaSessionNewParamsInitialEventUnion{{
  		OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  			Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  				OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  					Text: "List the files in the working directory.",
  				},
  			}},
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }
  // initial_events 不会在创建响应中回显，因此需列出
  // 会话的事件以读回预置的 user.message。
  seededEvents, err := client.Beta.Sessions.Events.List(ctx, seededSession.ID, anthropic.BetaSessionEventListParams{})
  if err != nil {
  	panic(err)
  }
  for _, event := range seededEvents.Data {
  	if event.Type != "user.message" {
  		continue
  	}
  	for _, contentBlock := range event.AsUserMessage().Content {
  		if contentBlock.Type == "text" {
  			fmt.Printf("Seeded event: %s\n", contentBlock.AsText().Text)
  		}
  	}
  }
  ```

  ```java Java
  var seededSession = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(agent.id())
      .environmentId(environment.id())
      .addInitialEvent(BetaManagedAgentsUserMessageEventParams.builder()
          .addTextContent("List the files in the working directory.")
          .build())
      .build());
  // initial_events 不会在创建响应中回显；列出
  // 会话的事件以读回预置的 user.message。
  for (var event : client.beta().sessions().events().list(seededSession.id()).autoPager()) {
      if (event.isUserMessage()) {
          for (var contentBlock : event.asUserMessage().content()) {
              if (contentBlock.isText()) {
                  IO.println("Seeded event: " + contentBlock.asText().text());
              }
          }
      }
  }
  ```

  ```php PHP
  $seededSession = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
      initialEvents: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => 'List the files in the working directory.']],
          ],
      ],
  );

  // initial_events 不会在创建响应中回显；需从会话的
  // 事件列表中读取它们。
  $seededEvents = $client->beta->sessions->events->list($seededSession->id);
  foreach ($seededEvents->getItems() as $event) {
      if ($event->type === 'user.message') {
          echo "Seeded event: {$event->content[0]->text}\n";
      }
  }
  ```

  ```ruby Ruby
  seeded_session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id,
    initial_events: [
      {
        type: :"user.message",
        content: [{type: :text, text: "List the files in the working directory."}]
      }
    ]
  )

  # initial_events 不会在创建响应中回显；需要从
  # 会话的事件列表中读回它们。
  client.beta.sessions.events.list(seeded_session.id).auto_paging_each do |event|
    next unless event.type == :"user.message"
    event.content.each do |block|
      puts "Seeded event: #{block.text}" if block.type == :text
    end
  end
  ```
</CodeGroup>

`initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 不会在创建响应中回显。要查看预置的消息，请列出会话的事件。

不接受其他任何事件类型。响应智能体轮次的事件（`user.tool_confirmation`、`user.tool_result` 和 `user.custom_tool_result`）不被接受，因为此时尚不存在智能体轮次；`user.interrupt` 也不被接受，因为没有可停止的轮次。与计划部署上的 `initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 不同，会话的 `initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 不接受 `system.message`。

`initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 中的每个事件都会在创建响应返回之前按列表顺序进行验证和持久化，并获得服务器分配的 ID，效果与您在创建后立即将其发布到[发送事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#send-events)端点完全相同。每个事件的内容规则也与该端点相同。空列表等同于省略该字段。验证是全有或全无的：如果任何事件验证失败，整个请求都会被拒绝，且不会创建会话。

在以下情况下，创建请求会被拒绝：

| 条件                                                                                                                   | 状态  |
| -------------------------------------------------------------------------------------------------------------------- | --- |
| 多于一个 `user.define_outcome` 事件                                                                                        | 400 |
| `user.define_outcome` 事件缺少 `rubric`                                                                                  | 400 |
| 整个列表中来源于文件的 [`document` 内容块](https://platform.claude.com/docs/zh-CN/build-with-claude/files#document-blocks)超过 100 个 | 400 |
| 请求体超过 32 MB                                                                                                          | 413 |

`initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 中的 `user.define_outcome` 事件的接受条件与向现有会话发送该事件时相同；请参阅[定义结果](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)。

### 为会话覆盖智能体配置

您可以通过三种形式传入 `agent` (csharp, go: `Agent`)：智能体 ID 字符串、固定版本对象（`type: "agent"`）或覆盖对象（`type: "agent_with_overrides"`）。覆盖形式会为单个会话更改智能体配置的部分内容。使用它可以在一个会话中尝试不同的模型或授予额外的工具，而无需为智能体创建新版本。对于覆盖形式，请传入智能体的 `id` (csharp, go: `ID`)，并可选地传入 `version` (csharp, go: `Version`)（省略 `version` (csharp, go: `Version`) 则使用智能体的最新版本）。然后包含 `model` (csharp, go: `Model`)、`system` (csharp, go: `System`; ruby: `system_`)、`tools` (csharp, go: `Tools`)、`mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 或 `skills` (csharp, go: `Skills`) 中的任意字段，并设置会话应使用的值。

每个可覆盖字段都遵循相同的三条规则：

* **省略该字段：** 会话继承其所引用的智能体版本中的值。

* **将该字段设置为 `null`，或对于列表字段设置为空数组：** 会话在该字段被清空的情况下运行。此规则完全适用于 `system` (csharp, go: `System`; ruby: `system_`) 和 `skills` (csharp, go: `Skills`)。有三个例外：

  * `model` (csharp, go: `Model`) 永远不能清空。会话始终需要一个模型，因此将 `model` (csharp, go: `Model`) 设为 `null` 会返回 400 `agent_model_required` 错误。
  * 当会话的有效 `skills` (csharp, go: `Skills`) 非空时，清空 `tools` (csharp, go: `Tools`) 会返回 400 错误，因为技能需要 `read` 工具。否则，将 `tools` (csharp, go: `Tools`) 设为 `null` 或空数组会清空该字段。
  * 当会话的有效 `tools` (csharp, go: `Tools`) 仍包含引用智能体某个服务器的 `mcp_toolset` 时，清空 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 会返回 400 错误。请在同一请求中覆盖 `tools` (csharp, go: `Tools`) 以移除这些 `mcp_toolset` 条目，然后再清空 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`)。

* **将该字段设置为某个值：** 该值会完全替换智能体的值。覆盖永远不会与智能体的配置合并，因此 `tools` (csharp, go: `Tools`) 覆盖必须列出会话应拥有的每个工具。同样，`model` (csharp, go: `Model`) 覆盖会完全替换智能体的 `model` (csharp, go: `Model`) 对象，因此智能体自身的 `effort` (csharp, go: `Effort`) 不会被沿用。要以特定的 effort 级别运行会话，请在覆盖的 `model` (csharp, go: `Model`) 对象中设置 `effort` (csharp, go: `Effort`)。模型不支持的级别会返回 400 错误，而不含 `effort` (csharp, go: `Effort`) 的 `model` (csharp, go: `Model`) 覆盖会以该模型的默认 effort 级别运行。

覆盖仅适用于您创建的会话。它们不会修改智能体资源，也不会创建新的智能体版本，因此引用同一智能体的其他会话不受影响。

在响应中，`agent` (csharp, go: `Agent`) 对象反映应用覆盖后会话运行所使用的配置。其 `id` (csharp, go: `ID`) 和 `version` (csharp, go: `Version`) 仍然标识应用覆盖的智能体和版本。这使您可以将会话追溯到其基础智能体。

以下示例启动一个会话，该会话覆盖了模型并清空了系统提示：

<CodeGroup>
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": {
        "type": "agent_with_overrides",
        "id": "'"$AGENT_ID"'",
        "model": {"id": "claude-sonnet-5-5"},
        "system": null
      },
      "environment_id": "'"$ENVIRONMENT_ID"'"
    }'
  ```

  ```bash CLI
  # 响应中的 `agent` 是解析后的快照：每个覆盖项仅针对此会话替换
  # 该字段，agent 资源保留其 ID 和版本。
  ant beta:sessions create <<YAML
  agent:
    type: agent_with_overrides
    id: $AGENT_ID
    model:
      id: claude-sonnet-5-5
    system: null
  environment_id: $ENVIRONMENT_ID
  YAML
  ```

  ```python Python
  override_session = client.beta.sessions.create(
      agent={
          "type": "agent_with_overrides",
          "id": agent.id,
          "model": {"id": "claude-sonnet-5-5"},
          "system": None,  # clear the agent's system prompt for this session
      },
      environment_id=environment.id,
  )
  # 响应中的 agent 是应用了覆盖后的已解析快照。
  print(f"Model: {override_session.agent.model.id}")
  print(f"System: {override_session.agent.system}")
  ```

  ```typescript TypeScript
  const overrideSession = await client.beta.sessions.create({
    agent: {
      type: "agent_with_overrides",
      id: agent.id,
      model: { id: "claude-sonnet-5-5" },
      system: null // clear the agent's system prompt for this session
    },
    environment_id: environment.id
  });
  // 响应中的 agent 是应用了覆盖项后解析出的快照。
  console.log(`Model: ${overrideSession.agent.model.id}`);
  console.log(`System: ${overrideSession.agent.system}`);
  ```

  ```csharp C#
  var overrideSession = await client.Beta.Sessions.Create(new()
  {
      Agent = new BetaManagedAgentsAgentWithOverridesParams
      {
          ID = agent.ID,
          Model = new BetaManagedAgentsModelConfigParams
          {
              ID = BetaManagedAgentsModel.ClaudeSonnet5_5,
          },
          System = null, // clear the agent's system prompt for this session
      },
      EnvironmentID = environment.ID,
  });
  // 响应中的 agent 是应用了覆盖项后解析出的快照。
  Console.WriteLine($"Model: {overrideSession.Agent.Model.ID.Raw()}");
  Console.WriteLine($"System: {overrideSession.Agent.System ?? "null"}");
  ```

  ```go Go
  overrideSession, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfBetaManagedAgentsAgentWithOverridess: &anthropic.BetaManagedAgentsAgentWithOverridesParams{
  			ID: agent.ID,
  			Model: anthropic.BetaManagedAgentsModelConfigParams{
  				ID: anthropic.BetaManagedAgentsModelClaudeSonnet5_5,
  			},
  			// 清除此会话的代理系统提示。
  			System: param.Null[string](),
  		},
  	},
  	EnvironmentID: environment.ID,
  })
  if err != nil {
  	panic(err)
  }
  // 响应中的 agent 是应用了覆盖项后解析出的快照。
  fmt.Printf("Model: %s\n", overrideSession.Agent.Model.ID)
  fmt.Printf("System: %q\n", overrideSession.Agent.System)
  ```

  ```java Java
  var overrideSession = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(BetaManagedAgentsAgentWithOverridesParams.builder()
          .id(agent.id())
          .model(BetaManagedAgentsModelConfigParams.builder()
              .id(BetaManagedAgentsModel.CLAUDE_SONNET_5_5)
              .build())
          .system((String) null) // clear the agent's system prompt for this session
          .build())
      .environmentId(environment.id())
      .build());
  // 响应中的 agent 是应用了覆盖项后解析出的快照。
  IO.println("Model: " + overrideSession.agent().model().id());
  IO.println("System: " + overrideSession.agent().system().orElse("null"));
  ```

  ```php PHP
  $overrides = BetaManagedAgentsAgentWithOverridesParams::with(
      id: $agent->id,
      model: ['id' => 'claude-sonnet-5-5'],
  );
  // 清除此 session 的系统提示。此处必须使用数组访问：
  // create() 会从原始数组中去除 null，而 ::with() 将 null 参数视为省略。
  $overrides['system'] = null;

  $overrideSession = $client->beta->sessions->create(
      agent: $overrides,
      environmentID: $environment->id,
  );
  // 响应中的 agent 是应用了覆盖后的已解析快照。
  echo "Model: {$overrideSession->agent->model->id}\n";
  echo 'System: ' . ($overrideSession->agent->system ?? 'null') . "\n";
  ```

  ```ruby Ruby
  # 系统提示覆盖项是 `system_`（带尾部下划线），因为普通的
  # `system` 是 Ruby 的 Kernel#system。将其设为 nil 会清除该提示。
  override_session = client.beta.sessions.create(
    agent: Anthropic::Beta::BetaManagedAgentsAgentWithOverridesParams.new(
      type: :agent_with_overrides,
      id: agent.id,
      model: {id: "claude-sonnet-5-5"},
      system_: nil
    ),
    environment_id: environment.id
  )
  # 响应中的代理是应用了覆盖项后解析出的快照。
  puts "Model: #{override_session.agent.model.id}"
  puts "System: #{override_session.agent.system_.inspect}"
  ```
</CodeGroup>

#### 为会话固定推理地理位置

由于 `model` (csharp, go: `Model`) 覆盖会完全替换代理的 `model` (csharp, go: `Model`) 对象，它也会为会话设置或清除模型的 “inference geo”（[推理地理位置](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)）固定设置。包含 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 的覆盖会固定为会话模型请求提供服务的地理位置。省略它的覆盖会清除代理的固定设置，因此会话将遵循工作区的 `default_inference_geo`。创建会话时，覆盖的值会根据工作区的 `allowed_inference_geos` 进行验证。

以下示例从一个模型未固定地理位置的智能体启动会话，并通过在 `model` (csharp, go: `Model`) 覆盖中包含 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`)，将会话的模型请求固定到美国推理。响应会在 `agent.model` (csharp, go: `Agent.Model`) 中回显该值：

<CodeGroup>
  ```bash cURL
  # 替换智能体的整个 model 对象：重新声明 id，并添加 inference_geo 以固定。
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": {
        "type": "agent_with_overrides",
        "id": "'"$AGENT_ID"'",
        "model": {"id": "claude-opus-5-5", "inference_geo": "us"}
      },
      "environment_id": "'"$ENVIRONMENT_ID"'"
    }'
  ```

  ```bash CLI
  # 替换智能体的整个 model 对象：重新声明 id，并添加 inference_geo 以固定。
  ant beta:sessions create <<YAML
  agent:
    type: agent_with_overrides
    id: $AGENT_ID
    model:
      id: claude-opus-5-5
      inference_geo: us
  environment_id: $ENVIRONMENT_ID
  YAML
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent={
          "type": "agent_with_overrides",
          "id": agent.id,
          # 完整替换智能体的 `model`：重新声明 `id`，添加 `inference_geo` 以固定。
          "model": {"id": "claude-opus-5-5", "inference_geo": "us"},
      },
      environment_id=environment.id,
  )
  print(f"Inference geo: {session.agent.model.inference_geo}")
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: {
      type: "agent_with_overrides",
      id: agent.id,
      // 完整替换智能体的 `model`：重新声明 `id`，添加 `inference_geo` 以固定。
      model: { id: "claude-opus-5-5", inference_geo: "us" }
    },
    environment_id: environment.id
  });
  console.log(`Inference geo: ${session.agent.model.inference_geo}`);
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = new BetaManagedAgentsAgentWithOverridesParams
      {
          ID = agent.ID,
          // 完整替换智能体的 `model`：重新声明 `id`，添加 `inference_geo` 以固定。
          Model = new BetaManagedAgentsModelConfigParams
          {
              ID = BetaManagedAgentsModel.ClaudeOpus5_5,
              InferenceGeo = "us",
          },
      },
      EnvironmentID = environment.ID,
  });
  Console.WriteLine($"Inference geo: {session.Agent.Model.InferenceGeo}");
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfBetaManagedAgentsAgentWithOverridess: &anthropic.BetaManagedAgentsAgentWithOverridesParams{
  			ID: agent.ID,
  			// 完整替换智能体的 `model`：重新声明 `id`，添加 `inference_geo` 以固定。
  			Model: anthropic.BetaManagedAgentsModelConfigParams{
  				ID:           anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  				InferenceGeo: anthropic.String("us"),
  			},
  		},
  	},
  	EnvironmentID: environment.ID,
  })
  if err != nil {
  	panic(err)
  }
  fmt.Printf("Inference geo: %s\n", session.Agent.Model.InferenceGeo)
  ```

  ```java Java
  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(BetaManagedAgentsAgentWithOverridesParams.builder()
          .id(agent.id())
          // 完整替换智能体的 `model`：重新声明 `id`，并添加 `inference_geo` 以固定。
          .model(BetaManagedAgentsModelConfigParams.builder()
              .id(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
              .inferenceGeo("us")
              .build())
          .build())
      .environmentId(environment.id())
      .build());
  IO.println("Inference geo: " + session.agent().model().inferenceGeo().orElseThrow());
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: BetaManagedAgentsAgentWithOverridesParams::with(
          id: $agent->id,
          // 完整替换智能体的 `model`：重新声明 `id`，并添加 `inference_geo` 以固定。
          model: BetaManagedAgentsModelConfigParams::with(
              id: 'claude-opus-5-5',
              inferenceGeo: 'us',
          ),
      ),
      environmentID: $environment->id,
  );
  echo "Inference geo: {$session->agent->model->inferenceGeo}\n";
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: {
      type: :agent_with_overrides,
      id: agent.id,
      # 完整替换智能体的 `model`：重新声明 `id`，并添加 `inference_geo` 以固定。
      model: {id: "claude-opus-5-5", inference_geo: "us"}
    },
    environment_id: environment.id
  )
  puts "Inference geo: #{session.agent.model.inference_geo}"
  ```
</CodeGroup>

<Tip>
  智能体定义了 Claude 在会话中的行为方式，包括模型、系统提示、工具和 MCP 服务器。详情请参阅[定义您的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)。
</Tip>

### 设置会话预算

要限制会话的支出，请在创建会话时传入可选的 `budget` (csharp, go: `Budget`) 对象。预算是会话标价成本（list cost）的硬性上限：平台按公开标价对会话消耗的所有内容计价，一旦累计总额达到 `max_list_cost` (java, php: `maxListCost`; csharp, go: `MaxListCost`)，会话便停止发出新的模型请求。预算的 `type` (csharp, go: `Type`) 始终为 `limit`。请为 `max_list_cost` (java, php: `maxListCost`; csharp, go: `MaxListCost`) 提供 `amount` (csharp, go: `Amount`) 和 `currency` (csharp, go: `Currency`)。`amount` (csharp, go: `Amount`) 是以字符串形式书写的美分整数，例如 `"125"` 表示 $1.25 USD。API 接受字符串而非数字，因此永远不会进行浮点舍入。`USD` 是目前唯一支持的货币。当会话达到上限时，它会暂停并进入空闲状态，停止原因为 `budget_reached`。上限在模型请求之间执行，因此越过上限的那个请求会先完成，会话的最终标价成本可能[略微超出上限](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#when-a-session-reaches-its-budget)。预算只能在创建时附加：您之后可以[更改或移除预算](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#updating-the-session-budget)，但无法为创建时未设置预算的会话添加预算。

以下示例创建一个预算为 $1.25 USD 的会话：

<CodeGroup>
  ```bash cURL
  curl -sS --fail-with-body https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'",
      "budget": {
        "type": "limit",
        "max_list_cost": {"amount": "125", "currency": "USD"}
      }
    }'
  ```

  ```bash CLI
  # 请保持金额带引号，以便作为字符串而非数字发送。
  ant beta:sessions create \
    --agent "$AGENT_ID" \
    --environment-id "$ENVIRONMENT_ID" \
    --budget '{type: limit, max_list_cost: {amount: "125", currency: USD}}'
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
      budget={
          "type": "limit",
          "max_list_cost": {"amount": "125", "currency": "USD"},
      },
  )
  print(session.id, session.budget.max_list_cost.amount)  # sesn_01... 125
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id,
    budget: {
      type: "limit",
      max_list_cost: { amount: "125", currency: "USD" }
    }
  });
  console.log(session.id, session.budget?.max_list_cost.amount); // sesn_01... 125
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
      Budget = new()
      {
          MaxListCost = new() { Amount = "125", Currency = BetaCurrency.Usd },
      },
  });
  Console.WriteLine($"{session.ID} {session.Budget?.MaxListCost.Amount}");  // sesn_01... 125
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String(agent.ID),
  	},
  	EnvironmentID: environment.ID,
  	Budget: anthropic.BetaManagedAgentsBudgetLimitParam{
  		MaxListCost: anthropic.BetaMonetaryAmountParam{
  			Amount:   "125",
  			Currency: anthropic.BetaCurrencyUsd,
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  fmt.Println(session.ID, session.Budget.MaxListCost.Amount) // sesn_01... 125
  ```

  ```java Java
  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(agent.id())
      .environmentId(environment.id())
      .budget(BetaManagedAgentsBudgetLimit.builder()
          .maxListCost(BetaMonetaryAmount.builder()
              .amount("125")
              .currency(BetaCurrency.USD)
              .build())
          .build())
      .build());
  IO.println(session.id() + " " + session.budget().orElseThrow().maxListCost().amount());  // sesn_01... 125
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
      budget: [
          'type' => 'limit',
          'max_list_cost' => ['amount' => '125', 'currency' => 'USD'],
      ],
  );
  echo "{$session->id} {$session->budget->maxListCost->amount}\n"; // sesn_01... 125
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id,
    budget: {
      type: "limit",
      max_list_cost: {amount: "125", currency: "USD"}
    }
  )
  puts "#{session.id} #{session.budget.max_list_cost.amount}" # sesn_01... 125
  ```
</CodeGroup>

请参阅[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)，了解预算的执行方式、哪些内容计入标价成本，以及预算在多智能体会话中的行为。

## 通过保管库进行 MCP 身份验证

如果您的智能体使用需要身份验证的 MCP 工具，请在创建会话时传入 `vault_ids` (java: `vaultIds`; php: `vaultIDs`; csharp: `VaultIds`; go: `VaultIDs`)，以引用包含已存储 OAuth 凭据的保管库（vault）。Anthropic 会代您管理令牌刷新。有关如何创建保管库和注册凭据，请参阅[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)。

<CodeGroup>
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'",
      "vault_ids": ["'"$VAULT_ID"'"]
    }'
  ```

  ```bash CLI
  ant beta:sessions create <<YAML
  agent: $AGENT_ID
  environment_id: $ENVIRONMENT_ID
  vault_ids:
    - $VAULT_ID
  YAML
  ```

  ```python Python
  vault_session = client.beta.sessions.create(
      agent=agent.id,
      environment_id=environment.id,
      vault_ids=[vault.id],
  )
  ```

  ```typescript TypeScript
  const vaultSession = await client.beta.sessions.create({
    agent: agent.id,
    environment_id: environment.id,
    vault_ids: [vault.id]
  });
  ```

  ```csharp C#
  var vaultSession = await client.Beta.Sessions.Create(new()
  {
      Agent = agent.ID,
      EnvironmentID = environment.ID,
      VaultIds = [vault.ID],
  });
  ```

  ```go Go
  vaultSession, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String(agent.ID),
  	},
  	EnvironmentID: environment.ID,
  	VaultIDs:      []string{vault.ID},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var vaultSession = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(agent.id())
      .environmentId(environment.id())
      .addVaultId(vault.id())
      .build());
  ```

  ```php PHP
  $vaultSession = $client->beta->sessions->create(
      agent: $agent->id,
      environmentID: $environment->id,
      vaultIDs: [$vault->id],
  );
  ```

  ```ruby Ruby
  vault_session = client.beta.sessions.create(
    agent: agent.id,
    environment_id: environment.id,
    vault_ids: [vault.id]
  )
  ```
</CodeGroup>

## 启动会话

不带 `initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 创建会话只会注册该会话，而不会启动任何工作；环境的沙箱在会话创建后立即开始预配，因此第一次工具调用无需等待。要委派任务，请使用[用户事件](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types)向会话发送事件。若要改为在创建请求中提供第一个事件，请参阅[使用初始事件为会话预置内容](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#seed-the-session-with-initial-events)。会话充当跟踪进度的状态机，而事件驱动实际的执行。

<CodeGroup>
  ```bash cURL
  curl -fsSL "https://api.anthropic.com/v1/sessions/$SESSION_ID/events" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {
          "type": "user.message",
          "content": [{"type": "text", "text": "List the files in the working directory."}]
        }
      ]
    }'
  ```

  ```bash CLI
  ant beta:sessions:events send --session-id "$SESSION_ID" <<'YAML'
  events:
    - type: user.message
      content:
        - type: text
          text: List the files in the working directory.
  YAML
  ```

  ```python Python
  client.beta.sessions.events.send(
      session.id,
      events=[
          {
              "type": "user.message",
              "content": [
                  {"type": "text", "text": "List the files in the working directory."}
              ],
          },
      ],
  )
  ```

  ```typescript TypeScript
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "List the files in the working directory." }]
      }
    ]
  });
  ```

  ```csharp C#
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content =
              [
                  new BetaManagedAgentsTextBlock
                  {
                      Text = "List the files in the working directory.",
                  },
              ],
          },
      ],
  });
  ```

  ```go Go
  if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  			Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  				OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  					Text: "List the files in the working directory.",
  				},
  			}},
  		},
  	}},
  }); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().sessions().events().send(
      session.id(),
      EventSendParams.builder()
          .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
              .addTextContent("List the files in the working directory.")
              .build())
          .build());
  ```

  ```php PHP
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => 'List the files in the working directory.']],
          ],
      ],
  );
  ```

  ```ruby Ruby
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {
        type: :"user.message",
        content: [{type: :text, text: "List the files in the working directory."}]
      }
    ]
  )
  ```
</CodeGroup>

有关如何流式传输智能体的响应以及处理工具确认，请参阅[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。

有关会话所经历的各种状态，请参阅[会话状态](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#session-statuses)。

## 后续步骤

<CardGroup cols={3}>
  <Card title="会话操作" icon="settings" href="https://platform.claude.com/docs/zh-CN/managed-agents/session-operations">
    检索、列出、更新、归档和删除 Claude Managed Agents 会话。
  </Card>

  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    发送事件、流式传输响应，并在会话运行时中断或重定向会话。
  </Card>

  <Card title="计划部署" icon="arrows-clockwise" href="https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments">
    使用 Claude API 创建和管理部署：按周期性 cron 计划运行智能体并查看其运行历史。
  </Card>
</CardGroup>
