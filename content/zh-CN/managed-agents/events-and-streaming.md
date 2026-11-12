---
title: 会话事件流
url: https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming
description: 发送事件、流式传输响应，并在会话运行时中断或重定向会话。
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

与 Claude Managed Agents 的通信基于事件。您打开一个流来跟踪会话，发送事件来指挥它，并在它暂停等待输入时作出回应。

## 事件类型

事件在两个方向上流动：您向智能体发送事件，会话向您发回事件。

* **您发送的事件：** `user.*` 事件用于启动会话，并在会话推进过程中引导它。[`system.message`](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#send-system-messages) 事件用于追加系统级上下文。
* **您接收的事件：** 会话事件、span 事件和智能体事件报告会话的状态和智能体的进度。

这些事件类型字符串遵循 `{domain}.{action}` 命名约定。完整目录请参阅参考文档中的[事件类型](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types)。

[Webhook 事件类型](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#supported-event-types)是独立的，其中一些名称与流中的名称不同。例如，webhook 使用 `session.status_idled` 而不是 `session.status_idle`。

## 流式传输事件

从会话流式传输事件，以便在智能体工作时接收实时更新。流只会传递其打开之后发出的事件，因此请在发送事件之前打开流，以避免竞态条件。

<CodeGroup>
  ```bash cURL
  # 在发送第一个事件之前打开流：只有之后的事件才会被传递。
  curl --fail-with-body -sS -N \
    "https://api.anthropic.com/v1/sessions/$SESSION_ID/events/stream?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "accept: text/event-stream"
  ```

  ```bash CLI
  # 在发送第一个事件之前打开流：只有之后的事件才会被传递。
  ant beta:sessions:events stream --session-id "$SESSION_ID"
  ```

  ```python Python
  # 先打开流，再发送用户消息
  with client.beta.sessions.events.stream(session.id) as stream:
      client.beta.sessions.events.send(
          session.id,
          events=[
              {
                  "type": "user.message",
                  "content": [{"type": "text", "text": "Summarize the repo README"}],
              },
          ],
      )

      for event in stream:
          match event.type:
              case "agent.message":
                  for block in event.content:
                      if block.type == "text":
                          print(block.text, end="")
              case "session.status_idle":
                  break
              case "session.error":
                  error_message = event.error.message if event.error else "unknown"
                  print(f"\n[Error: {error_message}]")
                  break
  ```

  ```typescript TypeScript
  // 先打开流，再发送用户消息
  const stream = await client.beta.sessions.events.stream(session.id);
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "Summarize the repo README" }]
      }
    ]
  });

  events: for await (const event of stream) {
    switch (event.type) {
      case "agent.message":
        for (const block of event.content) {
          if (block.type === "text") {
            process.stdout.write(block.text);
          }
        }
        break;
      case "session.status_idle":
        break events;
      case "session.error":
        console.log(`\n[Error: ${event.error?.message ?? "unknown"}]`);
        break events;
    }
  }
  ```

  ```csharp C#
  // 先打开流，再发送用户消息
  using var stream = await client.Beta.Sessions.Events.WithRawResponse.StreamStreaming(session.ID);
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content = [new BetaManagedAgentsTextBlock { Text = "Summarize the repo README" }],
          },
      ],
  });

  await foreach (var streamEvent in stream.Enumerate())
  {
      if (streamEvent.Value is BetaManagedAgentsAgentMessageEvent message)
      {
          foreach (var block in message.Content)
          {
              if (block.Value is BetaManagedAgentsTextBlock textBlock)
              {
                  Console.Write(textBlock.Text);
              }
          }
      }
      else if (streamEvent.Value is BetaManagedAgentsSessionStatusIdleEvent)
      {
          break;
      }
      else if (streamEvent.Value is BetaManagedAgentsSessionErrorEvent error)
      {
          Console.WriteLine($"\n[Error: {error.Error?.Message ?? "unknown"}]");
          break;
      }
  }
  ```

  ```go Go
  	// 先打开流，再发送用户消息
  	stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{})
  	defer stream.Close()

  	if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  		Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  					OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  						Text: "Summarize the repo README",
  					},
  				}},
  			},
  		}},
  	}); err != nil {
  		panic(err)
  	}

  events:
  	for stream.Next() {
  		switch event := stream.Current().AsAny().(type) {
  		case anthropic.BetaManagedAgentsAgentMessageEvent:
  			// 具体类型列表：BetaManagedAgentsTextBlock
  			for _, block := range event.Content {
  				fmt.Print(block.Text)
  			}
  		case anthropic.BetaManagedAgentsSessionStatusIdleEvent:
  			break events
  		case anthropic.BetaManagedAgentsSessionErrorEvent:
  			fmt.Printf("\n[Error: %s]\n", cmp.Or(event.Error.Message, "unknown"))
  			break events
  		}
  	}
  	if err := stream.Err(); err != nil {
  		panic(err)
  	}
  ```

  ```java Java
  // 先打开流，再发送用户消息
  try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
      client.beta().sessions().events().send(
          session.id(),
          EventSendParams.builder()
              .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
                  .addTextContent("Summarize the repo README")
                  .build())
              .build()
      );

      events:
      for (var event : stream) {
          switch (event.type().value()) {
              case AGENT_MESSAGE -> event.asAgentMessage().content().forEach(block -> block.text().ifPresent(textBlock -> IO.print(textBlock.text())));
              case SESSION_STATUS_IDLE -> {
                  break events;
              }
              case SESSION_ERROR -> {
                  IO.println("\n[Error: " + event.asSessionError().error().message() + "]");
                  break events;
              }
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionErrorEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock;

  // 先打开流，再发送用户消息
  $stream = $client->beta->sessions->events->streamStream($session->id);
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => 'Summarize the repo README']],
          ],
      ],
  );

  foreach ($stream as $event) {
      match (true) {
          $event instanceof ManagedAgentsAgentMessageEvent => array_walk(
              $event->content,
              static fn ($block) => $block instanceof ManagedAgentsTextBlock ? print($block->text) : null,
          ),
          $event instanceof ManagedAgentsSessionErrorEvent => printf("\n[Error: %s]", $event->error?->message ?? 'unknown'),
          default => null,
      };
      if ($event->type === 'session.status_idle' || $event->type === 'session.error') {
          break;
      }
  }
  $stream->close();
  ```

  ```ruby Ruby
  # 先打开流，再发送用户消息
  stream = client.beta.sessions.events.stream_events(session.id)

  client.beta.sessions.events.send_(
    session.id,
    events: [{
      type: "user.message",
      content: [{type: "text", text: "Summarize the repo README"}]
    }]
  )

  stream.each do |event|
    case event
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
      event.content.each { print it.text }
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusIdleEvent
      break
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionErrorEvent
      puts "\n[Error: #{event.error&.message || "unknown"}]"
      break
    else
      # 忽略其他事件类型
    end
  end
  ```
</CodeGroup>

会话通过 `session.error` 事件报告错误。有关其字段，请参阅[事件类型](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types)。

智能体的响应文本以缓冲的 `agent.message` 事件形式到达，每个事件仅在产生它的模型请求完成后才会发出。要在模型仍在生成文本时渲染文本，请参阅[使用事件增量预览响应](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas)。

### 重新连接而不遗漏事件

要重新连接到现有会话而不遗漏事件，请将新流与事件历史结合使用：

1. 打开一个新流。
2. [列出完整的事件历史](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#list-past-events)，以初始化一组已见事件 ID。
3. 跟踪实时流，跳过历史列表中已返回的任何事件。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  with client.beta.sessions.events.stream(session.id) as stream:
      # 流已打开并在缓冲。先列出历史记录，再跟踪实时事件。
      history = client.beta.sessions.events.list(session.id)
      seen_event_ids = {past_event.id for past_event in history}

      # 跟踪实时事件，跳过已见过的事件
      for event in stream:
          if event.type == "event_start" or event.type == "event_delta":
              # 此连接未启用增量预览。
              continue
          if event.id in seen_event_ids:
              continue
          seen_event_ids.add(event.id)
          match event.type:
              case "agent.message":
                  for block in event.content:
                      if block.type == "text":
                          print(block.text, end="")
              case "session.status_idle":
                  break
  ```

  ```typescript TypeScript
  const seenEventIds = new Set<string>();
  const stream = await client.beta.sessions.events.stream(session.id);

  // 流已打开并在缓冲。跟踪实时事件前先列出历史记录。
  for await (const event of client.beta.sessions.events.list(session.id)) {
    seenEventIds.add(event.id);
  }

  // 跟踪实时事件，跳过已见过的事件
  tail: for await (const event of stream) {
    // 预览事件（event_start/event_delta）不带顶层 `id`
    if (event.type === "event_start" || event.type === "event_delta") {
      continue;
    }
    if (seenEventIds.has(event.id)) {
      continue;
    }
    seenEventIds.add(event.id);
    switch (event.type) {
      case "agent.message":
        for (const block of event.content) {
          if (block.type === "text") {
            process.stdout.write(block.text);
          }
        }
        break;
      case "session.status_idle":
        break tail;
    }
  }
  ```

  ```csharp C#
  using var stream = await client.Beta.Sessions.Events.WithRawResponse.StreamStreaming(session.ID);

  // 流已打开并在缓冲。先列出历史记录，再跟踪实时事件。
  HashSet<string> seenEventIds = [];
  var history = await client.Beta.Sessions.Events.List(session.ID);
  await foreach (var pastEvent in history.Paginate())
  {
      seenEventIds.Add(pastEvent.ID);
  }

  // 跟踪实时事件，跳过已见过的事件
  await foreach (var streamEvent in stream.Enumerate())
  {
      if (!seenEventIds.Add(streamEvent.ID))
      {
          continue;
      }
      if (streamEvent.Value is BetaManagedAgentsAgentMessageEvent message)
      {
          foreach (var block in message.Content)
          {
              if (block.Value is BetaManagedAgentsTextBlock textBlock)
              {
                  Console.Write(textBlock.Text);
              }
          }
      }
      else if (streamEvent.Value is BetaManagedAgentsSessionStatusIdleEvent)
      {
          break;
      }
  }
  ```

  ```go Go
  	stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{})
  	defer stream.Close()

  	// 流已打开并在缓冲。跟踪实时事件前先列出历史记录。
  	seenEventIDs := map[string]struct{}{}
  	history := client.Beta.Sessions.Events.ListAutoPaging(ctx, session.ID, anthropic.BetaSessionEventListParams{})
  	for history.Next() {
  		seenEventIDs[history.Current().ID] = struct{}{}
  	}
  	if err := history.Err(); err != nil {
  		panic(err)
  	}

  	// 跟踪实时事件，跳过已见过的内容
  tail:
  	for stream.Next() {
  		event := stream.Current()
  		if _, seen := seenEventIDs[event.ID]; seen {
  			continue
  		}
  		seenEventIDs[event.ID] = struct{}{}
  		switch event := event.AsAny().(type) {
  		case anthropic.BetaManagedAgentsAgentMessageEvent:
  			// 具体类型列表：BetaManagedAgentsTextBlock
  			for _, block := range event.Content {
  				fmt.Print(block.Text)
  			}
  		case anthropic.BetaManagedAgentsSessionStatusIdleEvent:
  			break tail
  		}
  	}
  	if err := stream.Err(); err != nil {
  		panic(err)
  	}
  ```

  ```java Java
  try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
      // 流已打开并在缓冲。先列出历史记录，再跟踪实时事件。
      var seenEventIds = new HashSet<String>();
      for (var pastEvent : client.beta().sessions().events().list(session.id()).autoPager()) {
          seenEventIds.add(pastEvent.id());
      }

      // 跟踪实时事件；对已见过的 ID，Set.add 返回 false，从而跳过重放。
      stream.stream()
          .filter(event -> event.id().map(seenEventIds::add).orElse(true))
          .takeWhile(event -> !event.isSessionStatusIdle())
          .filter(BetaManagedAgentsStreamSessionEvents::isAgentMessage)
          .forEach(event -> event.asAgentMessage().content()
              .forEach(block -> block.text().ifPresent(textBlock -> IO.print(textBlock.text()))));
  }
  ```

  ```php PHP
  use Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock;

  $stream = $client->beta->sessions->events->streamStream($session->id);

  // 流已打开并在缓冲。先列出历史记录，再跟踪实时事件。
  $seenEventIds = [];
  foreach ($client->beta->sessions->events->list($session->id)->pagingEachItem() as $event) {
      $seenEventIds[$event->id] = true;
  }

  // 跟踪实时事件，跳过已见过的事件
  foreach ($stream as $event) {
      if (isset($seenEventIds[$event->id])) {
          continue;
      }
      $seenEventIds[$event->id] = true;
      match (true) {
          $event instanceof ManagedAgentsAgentMessageEvent => array_walk(
              $event->content,
              static fn ($block) => $block instanceof ManagedAgentsTextBlock ? print($block->text) : null,
          ),
          default => null,
      };
      if ($event->type === 'session.status_idle') {
          break;
      }
  }
  $stream->close();
  ```

  ```ruby Ruby
  stream = client.beta.sessions.events.stream_events(session.id)

  # 流已打开并在缓冲。先列出历史记录，再跟踪实时事件。
  seen_event_ids = Set.new
  client.beta.sessions.events.list(session.id).auto_paging_each { seen_event_ids << it.id }

  # 跟踪实时事件，跳过已见过的事件（Set#add? 对重复项返回 nil）
  stream.each do |event|
    next unless seen_event_ids.add?(event.id)
    case event
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
      event.content.each { print it.text }
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusIdleEvent
      break
    else
      # 忽略其他事件类型
    end
  end
  ```
</CodeGroup>

## 发送事件

发送 `user.message` 事件以启动或继续智能体的工作：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {
          "type": "user.message",
          "content": [
            {"type": "text", "text": "Analyze the performance of the sort function in utils.py"}
          ]
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
          text: Analyze the performance of the sort function in utils.py
  YAML
  ```

  ```python Python
  client.beta.sessions.events.send(
      session.id,
      events=[
          {
              "type": "user.message",
              "content": [
                  {
                      "type": "text",
                      "text": "Analyze the performance of the sort function in utils.py",
                  },
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
        content: [
          {
            type: "text",
            text: "Analyze the performance of the sort function in utils.py"
          }
        ]
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
                      Text = "Analyze the performance of the sort function in utils.py",
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
  					Text: "Analyze the performance of the sort function in utils.py",
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
              .addTextContent("Analyze the performance of the sort function in utils.py")
              .build())
          .build());
  ```

  ```php PHP
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => 'Analyze the performance of the sort function in utils.py',
                  ],
              ],
          ],
      ],
  );
  ```

  ```ruby Ruby
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {
        type: "user.message",
        content: [
          {
            type: "text",
            text: "Analyze the performance of the sort function in utils.py"
          }
        ]
      }
    ]
  )
  ```
</CodeGroup>

会话历史中的每个事件都包含一个 `processed_at` (java, php: `processedAt`; csharp, go: `ProcessedAt`) 时间戳，在事件处理完成时设置。对于您发送的事件，当事件仍排在较早事件之后等待处理时，`processed_at` (java, php: `processedAt`; csharp, go: `ProcessedAt`) 为 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `time.Time{}`)。例外情况是 `user.define_outcome`、`user.custom_tool_result` 和 `user.tool_result`，它们在收到时即被处理，回显时 `processed_at` (java, php: `processedAt`; csharp, go: `ProcessedAt`) 已被填充。

## 在会话进入空闲状态时作出响应

`session.status_idle` 事件表示智能体已停止并正在等待输入。其 `stop_reason.type` 说明了原因：

| `stop_reason.type` | 会话停止的原因                                                                                                             | 应采取的操作                                                                                                                                |
| ------------------ | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `end_turn`         | 智能体完成了其轮次，或[您中断了它](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#interrupt-the-agent)。 | 当您有更多工作交给智能体时，[发送 `user.message`](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#resume-an-idle-session)。 |
| `requires_action`  | 一个或多个工具调用需要您的回答，例如自定义工具调用或确认请求。                                                                                     | [回答每个阻塞的工具调用](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#answer-tool-calls-that-pause-the-session)。   |
| `budget_reached`   | 会话跟踪的标价成本达到了其[预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)。                                   | [更改或移除预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#resume-a-session-at-its-budget)。                              |

您发送的任何事件都无法恢复因达到预算而暂停的会话。当您将预算更改为高于已消耗标价成本的值或移除预算时，被预算暂停的工作会自动恢复。工作恢复时，会话会为每个[被预算暂停的工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#budgets-and-limits)发出一个 `workflow_run.status_running` 事件。被中断暂停的运行会保持暂停状态；请参阅[在达到预算时恢复会话](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#resume-a-session-at-its-budget)。有关标记暂停的事件以及会话仍接受的事件，请参阅[当会话达到其预算时](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#when-a-session-reaches-its-budget)。

## 回答使会话暂停的工具调用

当智能体调用[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#custom-tools)时，以及当某个工具调用在[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies)下需要您确认时，会话会暂停。这两种暂停都遵循相同的顺序：

1. 会话以 `agent.custom_tool_use`、`agent.tool_use` 或 `agent.mcp_tool_use` 事件的形式发出工具调用。
2. 会话以一个 `stop_reason.type` 为 `requires_action` 的 `session.status_idle` 事件暂停。阻塞事件的 ID 位于 `stop_reason.event_ids` 数组中。
3. 针对每个阻塞事件 ID，发送一个 [`user.custom_tool_result`](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#return-a-custom-tool-result) 或 [`user.tool_confirmation`](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#confirm-a-tool-call) 事件。自定义工具结果不必等待第 2 步。
4. 所有阻塞事件都解决后，会话会转换回 `running`。

在多智能体会话中，子智能体的阻塞事件会被交叉发布到主线程。请参阅[工具权限和自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads#tool-permissions-and-custom-tools)。

### 返回自定义工具结果

`agent.custom_tool_use` 事件包含工具名称和输入。在您的系统中运行该工具。然后发送 `user.custom_tool_result` 事件，在 `custom_tool_use_id` (java: `customToolUseId`; php: `customToolUseID`; csharp, go: `CustomToolUseID`) 参数中传入事件 ID，并附上结果内容。

您可以在 `agent.custom_tool_use` 事件到达后立即发送结果，无需等待 `session.status_idle`。会话仍会针对该调用发出带有 `requires_action` 停止原因的 `session.status_idle`，您的客户端可以忽略它。针对同一调用的第二个结果会被接受，但不会产生任何效果。

当您[重新连接](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#reconnect-without-missing-events)到已暂停的会话时，最新 `session.status_idle` 事件上的 `stop_reason.event_ids` 会列出需要回答的调用。

以下示例在每个调用到达时回答它：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  with client.beta.sessions.events.stream(session.id) as stream:
      for event in stream:
          match event.type:
              case "agent.custom_tool_use":
                  result = call_tool(event.name, event.input)

                  client.beta.sessions.events.send(
                      session.id,
                      events=[
                          {
                              "type": "user.custom_tool_result",
                              "custom_tool_use_id": event.id,
                              "content": [{"type": "text", "text": result}],
                          },
                      ],
                  )
              case "session.status_idle":
                  if event.stop_reason and event.stop_reason.type == "end_turn":
                      break
  ```

  ```typescript TypeScript
  const stream = await client.beta.sessions.events.stream(session.id);

  loop: for await (const event of stream) {
    switch (event.type) {
      case "agent.custom_tool_use": {
        const result = await callTool(event.name, event.input);

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
        if (event.stop_reason.type === "end_turn") {
          break loop;
        }
        break;
    }
  }
  ```

  ```csharp C#
  await foreach (var streamEvent in client.Beta.Sessions.Events.StreamStreaming(session.ID))
  {
      if (streamEvent.Value is BetaManagedAgentsAgentCustomToolUseEvent toolUse)
      {
          var result = await CallTool(toolUse.Name, toolUse.Input);

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
  	stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{})
  	defer stream.Close()

  loop:
  	for stream.Next() {
  		switch event := stream.Current().AsAny().(type) {
  		case anthropic.BetaManagedAgentsAgentCustomToolUseEvent:
  			result := callTool(event.Name, event.Input)
  			if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  				Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  					OfUserCustomToolResult: &anthropic.BetaManagedAgentsUserCustomToolResultEventParams{
  						CustomToolUseID: event.ID,
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
  		case anthropic.BetaManagedAgentsSessionStatusIdleEvent:
  			if _, ok := event.StopReason.AsAny().(anthropic.BetaManagedAgentsSessionEndTurn); ok {
  				break loop
  			}
  		}
  	}
  	if err := stream.Err(); err != nil {
  		panic(err)
  	}
  ```

  ```java Java
  try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
      loop:
      for (var event : stream) {
          switch (event.type().value()) {
              case AGENT_CUSTOM_TOOL_USE -> {
                  var toolUse = event.asAgentCustomToolUse();
                  var result = callTool(toolUse.name(), toolUse.input());

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
  $stream = $client->beta->sessions->events->streamStream($session->id);

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsAgentCustomToolUseEvent:
              $result = callTool($event->name, $event->input);

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
          case $event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsSessionStatusIdleEvent:
              if ($event->stopReason instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsSessionEndTurn) {
                  break 2;
              }
              break;
      }
  }
  ```

  ```ruby Ruby
  client.beta.sessions.events.stream_events(session.id).each do |event|
    case event
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentCustomToolUseEvent
      result = call_tool.call(event.name, event.input)
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

如果智能体有打开的[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)，自定义工具调用可能会在会话保持 `running` 时到达。只有当[会话的所有线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)都不在工作时，`requires_action` 空闲事件才会到达。不要等待它：在每个调用的 `agent.custom_tool_use` 事件到达时就回答它。[跟踪运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#follow-a-run)中的示例展示了具体做法。

### 确认工具调用

发送 `user.tool_confirmation` 事件，在 `tool_use_id` (java: `toolUseId`; php: `toolUseID`; csharp, go: `ToolUseID`) 参数中传入事件 ID。将 `result` (csharp, go: `Result`) 设置为 `"allow"` 或 `"deny"`。有关哪些调用需要等待确认以及如何解释拒绝，请参阅[响应确认请求](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#respond-to-confirmation-requests)。

以下示例批准每个待处理的调用：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  with client.beta.sessions.events.stream(session.id) as stream:
      for event in stream:
          if event.type == "session.status_idle" and (stop_reason := event.stop_reason):
              match stop_reason.type:
                  case "requires_action":
                      for event_id in stop_reason.event_ids:
                          client.beta.sessions.events.send(
                              session.id,
                              events=[
                                  {
                                      "type": "user.tool_confirmation",
                                      "tool_use_id": event_id,
                                      "result": "allow",
                                  },
                              ],
                          )
                  case "end_turn":
                      break
  ```

  ```typescript TypeScript
  const stream = await client.beta.sessions.events.stream(session.id);

  for await (const event of stream) {
    if (event.type !== "session.status_idle") {
      continue;
    }
    if (event.stop_reason.type === "end_turn") {
      break;
    }
    if (event.stop_reason.type !== "requires_action") {
      continue;
    }

    for (const eventId of event.stop_reason.event_ids) {
      await client.beta.sessions.events.send(session.id, {
        events: [
          {
            type: "user.tool_confirmation",
            tool_use_id: eventId,
            result: "allow"
          }
        ]
      });
    }
  }
  ```

  ```csharp C#
  await foreach (var streamEvent in client.Beta.Sessions.Events.StreamStreaming(session.ID))
  {
      if (streamEvent.Value is not BetaManagedAgentsSessionStatusIdleEvent idle) continue;

      if (idle.StopReason?.Value is BetaManagedAgentsSessionRequiresAction requiresAction)
      {
          foreach (var eventId in requiresAction.EventIds)
          {
              await client.Beta.Sessions.Events.Send(session.ID, new()
              {
                  Events =
                  [
                      new BetaManagedAgentsUserToolConfirmationEventParams
                      {
                          ToolUseID = eventId,
                          Result = BetaManagedAgentsUserToolConfirmationEventParamsResult.Allow,
                      },
                  ],
              });
          }
      }
      else if (idle.StopReason?.Value is BetaManagedAgentsSessionEndTurn)
      {
          break;
      }
  }
  ```

  ```go Go
  	stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{})
  	defer stream.Close()

  loop:
  	for stream.Next() {
  		event, ok := stream.Current().AsAny().(anthropic.BetaManagedAgentsSessionStatusIdleEvent)
  		if !ok {
  			continue
  		}
  		switch stopReason := event.StopReason.AsAny().(type) {
  		case anthropic.BetaManagedAgentsSessionRequiresAction:
  			for _, eventID := range stopReason.EventIDs {
  				if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  					Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  						OfUserToolConfirmation: &anthropic.BetaManagedAgentsUserToolConfirmationEventParams{
  							ToolUseID: eventID,
  							Result:    anthropic.BetaManagedAgentsUserToolConfirmationEventParamsResultAllow,
  						},
  					}},
  				}); err != nil {
  					panic(err)
  				}
  			}
  		case anthropic.BetaManagedAgentsSessionEndTurn:
  			break loop
  		}
  	}
  	if err := stream.Err(); err != nil {
  		panic(err)
  	}
  ```

  ```java Java
  try (var stream = client.beta().sessions().events().streamStreaming(session.id())) {
      stream.stream()
          .filter(BetaManagedAgentsStreamSessionEvents::isSessionStatusIdle)
          .map(idleEvent -> idleEvent.asSessionStatusIdle().stopReason())
          .takeWhile(stopReason -> !stopReason.isEndTurn())
          .filter(stopReason -> stopReason.isRequiresAction())
          .flatMap(stopReason -> stopReason.asRequiresAction().eventIds().stream())
          .forEach(toolUseId -> client.beta().sessions().events().send(
              session.id(),
              EventSendParams.builder()
                  .addEvent(BetaManagedAgentsUserToolConfirmationEventParams.builder()
                      .toolUseId(toolUseId)
                      .result(BetaManagedAgentsUserToolConfirmationEventParams.Result.ALLOW)
                      .build())
                  .build()));
  }
  ```

  ```php PHP
  $stream = $client->beta->sessions->events->streamStream($session->id);

  foreach ($stream as $event) {
      if ($event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsSessionStatusIdleEvent && $event->stopReason) {
          switch (true) {
              case $event->stopReason instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsSessionRequiresAction:
                  foreach ($event->stopReason->eventIDs as $eventId) {
                      $client->beta->sessions->events->send(
                          $session->id,
                          events: [
                              [
                                  'type' => 'user.tool_confirmation',
                                  'tool_use_id' => $eventId,
                                  'result' => 'allow',
                              ],
                          ],
                      );
                  }
                  break;
              case $event->stopReason instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsSessionEndTurn:
                  break 2;
          }
      }
  }
  ```

  ```ruby Ruby
  client.beta.sessions.events.stream_events(session.id).each do |event|
    case event
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusIdleEvent
      stop_reason = event.stop_reason
      case stop_reason
      when Anthropic::Beta::Sessions::BetaManagedAgentsSessionRequiresAction
        stop_reason.event_ids.each do |event_id|
          client.beta.sessions.events.send_(
            session.id,
            events: [
              {type: "user.tool_confirmation", tool_use_id: event_id, result: "allow"}
            ]
          )
        end
      when Anthropic::Beta::Sessions::BetaManagedAgentsSessionEndTurn
        break
      end
    end
  end
  ```
</CodeGroup>

上面的示例在会话进入空闲状态后批准每个调用。如果智能体有打开的[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)，工具调用可能会在会话保持 `running` 时等待您的确认。不要等待空闲事件：当 `agent.tool_use` 或 `agent.mcp_tool_use` 事件到达且其 [`evaluated_permission` (java, php: `evaluatedPermission`; csharp, go: `EvaluatedPermission`)](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#see-how-each-call-was-evaluated) 为 `ask` 时，就回答它。[跟踪运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#follow-a-run)中的示例以这种方式回答自定义工具调用，其简介部分说明了如何添加确认。

## 恢复空闲会话

会话在多次交互之间持久存在。要恢复会话，像往常一样向其发送 `user.message` 事件即可：

<CodeGroup>
  ```bash cURL
  # 在生产环境中，请传入您想要恢复的会话的已存储 ID。
  curl --fail-with-body -sS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {
          "type": "user.message",
          "content": [
            {"type": "text", "text": "Now run the tests against the changes you made earlier."}
          ]
        }
      ]
    }'
  ```

  ```bash CLI
  # 在生产环境中，传入您想要恢复的会话的已存储 ID。
  ant beta:sessions:events send --session-id "$SESSION_ID" <<'YAML'
  events:
    - type: user.message
      content:
        - type: text
          text: Now run the tests against the changes you made earlier.
  YAML
  ```

  ```python Python
  # 通过发送新的 user.message 事件来恢复先前创建的会话。
  # 在生产环境中，传入您想要恢复的会话的已存储 ID。
  client.beta.sessions.events.send(
      session.id,
      events=[
          {
              "type": "user.message",
              "content": [
                  {
                      "type": "text",
                      "text": "Now run the tests against the changes you made earlier.",
                  },
              ],
          },
      ],
  )
  ```

  ```typescript TypeScript
  // 通过向先前创建的会话发送新的用户事件来恢复该会话。
  // 在生产环境中，请传入您想要恢复的会话的已存储 ID。
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.message",
        content: [
          {
            type: "text",
            text: "Now run the tests against the changes you made earlier."
          }
        ]
      }
    ]
  });
  ```

  ```csharp C#
  // 通过 ID 恢复先前创建的会话。在生产环境中，请传入
  // 您在创建会话时存储的会话 ID。
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
                      Text = "Now run the tests against the changes you made earlier.",
                  },
              ],
          },
      ],
  });
  ```

  ```go Go
  // 恢复先前创建的会话：向其发送新的 user.message
  // 事件。在生产环境中，请传入要恢复的会话的已存储 ID。
  if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  			Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  				OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  					Text: "Now run the tests against the changes you made earlier.",
  				},
  			}},
  		},
  	}},
  }); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  // 通过 ID 恢复先前创建的会话。在生产环境中，请传入
  // 您在创建会话时存储的会话 ID。
  client.beta().sessions().events().send(
      session.id(),
      EventSendParams.builder()
          .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
              .addTextContent("Now run the tests against the changes you made earlier.")
              .build())
          .build());
  ```

  ```php PHP
  // 通过发送新的 user.message 事件来恢复先前创建的会话。
  // 在生产环境中，请传入您在创建会话时保存的会话 ID。
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => 'Now run the tests against the changes you made earlier.',
                  ],
              ],
          ],
      ],
  );
  ```

  ```ruby Ruby
  # 要恢复会话，请向其发送下一个事件。在生产环境中，
  # 请传入创建会话时存储的会话 ID。
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {
        type: "user.message",
        content: [
          {type: "text", text: "Now run the tests against the changes you made earlier."}
        ]
      }
    ]
  )
  ```
</CodeGroup>

除非会话被显式删除，否则对话历史会被保留。当会话进入空闲状态时，其沙箱会被创建检查点。检查点保留完整的沙箱状态，包括文件系统、已安装的软件包以及智能体创建的任何文件。

<Note>
  沙箱状态仅在沙箱创建后保留 30 天，且活动不会延长此期限。30 天后，沙箱状态（文件、已安装的工具等）将无法恢复，恢复的会话将从全新的沙箱开始。如果您的工作流依赖于沙箱内容，请让智能体在期限结束前将重要产物写入[输出](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes#retrieving-deliverables)。
</Note>

## 中断智能体

发送 `user.interrupt` 事件以在智能体运行时停止它，然后发送 `user.message` 事件来重定向它：

<CodeGroup>
  ```bash cURL
  # 智能体当前正在分析文件...
  # 用新的指令中断：
  curl --fail-with-body -sS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {"type": "user.interrupt"},
        {
          "type": "user.message",
          "content": [
            {"type": "text", "text": "Instead, focus on fixing the bug in line 42."}
          ]
        }
      ]
    }'
  ```

  ```bash CLI
  # 代理当前正在分析一个文件……
  # 用新的指令中断：
  ant beta:sessions:events send --session-id "$SESSION_ID" <<'YAML'
  events:
    - type: user.interrupt
    - type: user.message
      content:
        - type: text
          text: Instead, focus on fixing the bug in line 42.
  YAML
  ```

  ```python Python
  # 智能体当前正在分析文件...
  # 用新的指令中断：
  client.beta.sessions.events.send(
      session.id,
      events=[
          {"type": "user.interrupt"},
          {
              "type": "user.message",
              "content": [
                  {
                      "type": "text",
                      "text": "Instead, focus on fixing the bug in line 42.",
                  },
              ],
          },
      ],
  )
  ```

  ```typescript TypeScript
  // 智能体当前正在分析文件……
  // 用新的指令中断：
  await client.beta.sessions.events.send(session.id, {
    events: [
      { type: "user.interrupt" },
      {
        type: "user.message",
        content: [
          {
            type: "text",
            text: "Instead, focus on fixing the bug in line 42."
          }
        ]
      }
    ]
  });
  ```

  ```csharp C#
  // 智能体当前正在分析文件...
  // 用新的指令进行中断：
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserInterruptEventParams(),
          new BetaManagedAgentsUserMessageEventParams
          {
              Content =
              [
                  new BetaManagedAgentsTextBlock
                  {
                      Text = "Instead, focus on fixing the bug in line 42.",
                  },
              ],
          },
      ],
  });
  ```

  ```go Go
  // 智能体当前正在分析文件...
  // 以新的指示中断：
  if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{
  		{
  			OfUserInterrupt: &anthropic.BetaManagedAgentsUserInterruptEventParams{},
  		},
  		{
  			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  					OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  						Text: "Instead, focus on fixing the bug in line 42.",
  					},
  				}},
  			},
  		},
  	},
  }); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  // 智能体当前正在分析文件……
  // 以新的指示进行中断：
  client.beta().sessions().events().send(
      session.id(),
      EventSendParams.builder()
          .addEvent(BetaManagedAgentsUserInterruptEventParams.builder().build())
          .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
              .addTextContent("Instead, focus on fixing the bug in line 42.")
              .build())
          .build());
  ```

  ```php PHP
  // 智能体当前正在分析一个文件...
  // 以新的方向进行中断：
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          ['type' => 'user.interrupt'],
          [
              'type' => 'user.message',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => 'Instead, focus on fixing the bug in line 42.',
                  ],
              ],
          ],
      ],
  );
  ```

  ```ruby Ruby
  # 智能体当前正在分析一个文件……
  # 用新的指令中断：
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {type: "user.interrupt"},
      {
        type: "user.message",
        content: [
          {type: "text", text: "Instead, focus on fixing the bug in line 42."}
        ]
      }
    ]
  )
  ```
</CodeGroup>

事件一旦进入队列，调用就会返回。随后中断按以下顺序生效：

1. 智能体应用中断。在此之前，中断的 `processed_at` (java, php: `processedAt`; csharp, go: `ProcessedAt`) 保持为 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `time.Time{}`)，会话保持 `running`。正在进行的模型响应会立即停止，但当工具调用正在运行时，中断可能需要更长时间才能生效。
2. `user.interrupt` 事件出现在流中，被中断的轮次以 `session.status_idle` 事件结束。
3. 智能体以您在中断后发送的 `user.message` 开始下一个轮次。

空闲事件的 `stop_reason.type` 为 `end_turn`，与自行完成的轮次的值相同。如果有打开的工作流运行，中断不会结束其中任何一个。正在运行的运行可能使会话保持 `running` 状态，因此步骤 2 中的 `session.status_idle` 事件可能不会到达。请参阅[在有打开的运行时中断会话](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#interrupt-a-session-with-runs-open)。

## 列出过去的事件

检索会话的完整事件历史：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json"
  ```

  ```bash CLI
  ant beta:sessions:events list --session-id "$SESSION_ID" --format jsonl
  ```

  ```python Python
  events = client.beta.sessions.events.list(session.id)
  for event in events.data:
      print(f"[{event.type}] {event.processed_at}")
  ```

  ```typescript TypeScript
  const events = await client.beta.sessions.events.list(session.id);
  for (const event of events.data) {
    console.log(`[${event.type}] ${event.processed_at}`);
  }
  ```

  ```csharp C#
  var events = await client.Beta.Sessions.Events.List(session.ID);
  foreach (var sessionEvent in events.Items)
  {
      Console.WriteLine($"[{sessionEvent.Type.Raw()}] {sessionEvent.ProcessedAt}");
  }
  ```

  ```go Go
  events, err := client.Beta.Sessions.Events.List(ctx, session.ID, anthropic.BetaSessionEventListParams{})
  if err != nil {
  	panic(err)
  }
  for _, event := range events.Data {
  	fmt.Printf("[%s] %s\n", event.Type, event.ProcessedAt)
  }
  ```

  ```java Java
  var events = client.beta().sessions().events().list(session.id());
  for (var event : events.data()) {
      IO.println("[" + event.type().asString() + "] " + event.processedAt().orElse(null));
  }
  ```

  ```php PHP
  $events = $client->beta->sessions->events->list($session->id);
  foreach ($events->data as $event) {
      $processedAt = ($event->processedAt ?? null)?->format(DATE_RFC3339) ?? 'null';
      echo "[{$event->type}] {$processedAt}\n";
  }
  ```

  ```ruby Ruby
  events = client.beta.sessions.events.list(session.id)
  events.data.each { puts "[#{it.type}] #{it.processed_at}" }
  ```
</CodeGroup>

传入 `types` (curl: `types[]`; cli: `--type`; csharp, go: `Types`) 过滤器以仅返回特定的事件类型：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true&types[]=agent.tool_use&types[]=agent.tool_result" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:sessions:events list --session-id "$SESSION_ID" \
    --type agent.tool_use --type agent.tool_result \
    --format jsonl
  ```

  ```python Python
  events = client.beta.sessions.events.list(
      session.id,
      types=["agent.tool_use", "agent.tool_result"],
  )
  for event in events.data:
      print(f"[{event.type}] {event.processed_at}")
  ```

  ```typescript TypeScript
  const events = await client.beta.sessions.events.list(session.id, {
    types: ["agent.tool_use", "agent.tool_result"]
  });
  for (const event of events.data) {
    console.log(`[${event.type}] ${event.processed_at}`);
  }
  ```

  ```csharp C#
  var events = await client.Beta.Sessions.Events.List(session.ID, new()
  {
      Types = ["agent.tool_use", "agent.tool_result"],
  });
  foreach (var sessionEvent in events.Items)
  {
      Console.WriteLine($"[{sessionEvent.Type.Raw()}] {sessionEvent.ProcessedAt}");
  }
  ```

  ```go Go
  events, err := client.Beta.Sessions.Events.List(ctx, session.ID, anthropic.BetaSessionEventListParams{
  	Types: []anthropic.BetaManagedAgentsSessionEventType{
  		anthropic.BetaManagedAgentsSessionEventTypeAgentToolUse,
  		anthropic.BetaManagedAgentsSessionEventTypeAgentToolResult,
  	},
  })
  if err != nil {
  	panic(err)
  }
  for _, event := range events.Data {
  	fmt.Printf("[%s] %s\n", event.Type, event.ProcessedAt)
  }
  ```

  ```java Java
  var events = client.beta().sessions().events().list(
      session.id(),
      EventListParams.builder()
          .addType(BetaManagedAgentsSessionEventType.AGENT_TOOL_USE)
          .addType(BetaManagedAgentsSessionEventType.AGENT_TOOL_RESULT)
          .build());
  for (var event : events.data()) {
      event.agentToolUse().ifPresent(toolUse ->
          IO.println("[" + toolUse._type() + "] " + toolUse.processedAt()));
      event.agentToolResult().ifPresent(toolResult ->
          IO.println("[" + toolResult._type() + "] " + toolResult.processedAt()));
  }
  ```

  ```php PHP
  $events = $client->beta->sessions->events->list(
      $session->id,
      types: ['agent.tool_use', 'agent.tool_result'],
  );
  foreach ($events->data as $event) {
      $processedAt = ($event->processedAt ?? null)?->format(DATE_RFC3339) ?? 'null';
      echo "[{$event->type}] {$processedAt}\n";
  }
  ```

  ```ruby Ruby
  events = client.beta.sessions.events.list(
    session.id,
    types: ["agent.tool_use", "agent.tool_result"]
  )
  events.data.each { puts "[#{it.type}] #{it.processed_at}" }
  ```
</CodeGroup>

## 发送系统消息

在[受支持的模型](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#supported-models)上，发送 `system.message` 事件可为智能体提供特权的系统级上下文。该上下文适用于随附的轮次以及所有后续轮次。当智能体在会话中途需要更新的系统级指导时使用它，例如不同的角色设定、修订后的约束条件或在运行时获取的上下文。

该事件的 `content` (csharp, go: `Content`) 接受 1–1,000 个文本项。内容作为 `role: "system"` 轮次追加到会话的系统上下文中。它不会替换顶层系统提示，顶层系统提示由智能体定义上的 `system` (csharp, go: `System`; ruby: `system_`) 字段设置。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {
          "type": "system.message",
          "content": [
            {"type": "text", "text": "The current timezone of the user is America/New_York."}
          ]
        }
      ]
    }'
  ```

  ```bash CLI
  ant beta:sessions:events send --session-id "$SESSION_ID" <<'YAML'
  events:
    - type: system.message
      content:
        - type: text
          text: "The user's current timezone is America/New_York."
  YAML
  ```

  ```python Python
  client.beta.sessions.events.send(
      session.id,
      events=[
          {
              "type": "system.message",
              "content": [
                  {
                      "type": "text",
                      "text": "The user's current timezone is America/New_York.",
                  },
              ],
          },
      ],
  )
  ```

  ```typescript TypeScript
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "system.message",
        content: [
          {
            type: "text",
            text: "The user's current timezone is America/New_York."
          }
        ]
      }
    ]
  });
  ```

  ```csharp C#
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsSystemMessageEventParams
          {
              Content =
              [
                  new BetaManagedAgentsSystemContentBlock
                  {
                      Text = "The user's current timezone is America/New_York.",
                  },
              ],
          },
      ],
  });
  ```

  ```go Go
  if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfSystemMessage: &anthropic.BetaManagedAgentsSystemMessageEventParams{
  			Content: []anthropic.BetaManagedAgentsSystemContentBlockParam{{
  				Text: "The user's current timezone is America/New_York.",
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
          .addEvent(BetaManagedAgentsSystemMessageEventParams.builder()
              .addTextContent("The user's current timezone is America/New_York.")
              .build())
          .build());
  ```

  ```php PHP
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'system.message',
              'content' => [
                  [
                      'type' => 'text',
                      'text' => "The user's current timezone is America/New_York.",
                  ],
              ],
          ],
      ],
  );
  ```

  ```ruby Ruby
  client.beta.sessions.events.send_(
    session.id,
    events: [
      {
        type: "system.message",
        content: [
          {type: "text", text: "The user's current timezone is America/New_York."}
        ]
      }
    ]
  )
  ```
</CodeGroup>

### 受支持的模型

以下模型支持 `system.message`：

* Claude Fable 5.2
* Claude Mythos 5.2
* Claude Fable 5.1
* Claude Mythos 5.1
* Claude Fable 5
* Claude Mythos 5
* Claude Opus 5.5
* Claude Opus 5
* Claude Opus 4.8
* Claude Sonnet 5.5
* Claude Haiku 5.5

如果智能体的主模型不支持在对话中途注入系统内容，该事件将被拒绝，并返回 `model_does_not_support_mid_conversation_system` 验证错误。子智能体模型不会被检查，因为 `system.message` 仅落在主线程上。

### 工具调用待处理时的系统消息

当会话以 `requires_action` 停止原因处于空闲状态时，`system.message` 必须在同一请求中跟在工具结果事件之后。如果单独发送或与 `user.message` 一起发送，在待处理的工具事件解决之前，它都会被拒绝。

## 后续步骤

<CardGroup cols={2}>
  <Card title="使用事件增量预览响应" icon="text" href="https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas">
    在模型仍在生成时渲染智能体的响应文本。
  </Card>

  <Card title="检查会话并跟踪用量" icon="magnifying-glass" href="https://platform.claude.com/docs/zh-CN/managed-agents/session-observability">
    在 Claude Console 中检查会话，并查看其令牌用量和标价成本。
  </Card>

  <Card title="订阅 webhook" icon="link" href="https://platform.claude.com/docs/zh-CN/managed-agents/webhooks">
    在发生重大事件时获得通知，无需轮询。
  </Card>

  <Card title="事件类型" icon="book" href="https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types">
    查阅会话可以发送或接收的每种事件类型。
  </Card>
</CardGroup>
