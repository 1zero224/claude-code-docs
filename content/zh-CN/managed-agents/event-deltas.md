---
title: 使用事件增量预览响应
url: https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas
description: 在模型仍在生成响应文本时，将智能体的响应文本渲染为实时预览。
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

默认情况下，智能体的响应文本以缓冲的 `agent.message` 事件形式到达[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。每个事件仅在生成它的模型请求完成后才会发出。"Event deltas"（事件增量）让您可以在模型仍在生成文本时，以实时预览的方式增量渲染该文本。

预览是一种尽力而为的显示辅助手段，缓冲的 `agent.message` 始终是权威记录。忽略预览的客户端仍会收到完整、正确的流。

## 选择启用预览

预览需要针对每个流连接单独选择启用。将 `event_deltas[]` 查询参数添加到您正在读取的流中，并为每个希望预览的事件类型重复一次该参数。可接受的值为 `agent.message` 和 `agent.thinking`。任何其他值都会返回 400 错误，包含超过 100 个值的请求也是如此。

两个流端点都接受该参数：

* **会话级流：** `GET /v1/sessions/{session_id}/events/stream`
* **会话线程流：** `GET /v1/sessions/{session_id}/threads/{thread_id}/stream`

子智能体的预览会出现在[该子智能体自己的线程流](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#preview-session-thread-events)上。

`[]` 是 shell 通配符模式，因此每当您在 shell 中构建请求时，请为 URL 加上引号。示例将方括号百分号编码为 `%5B%5D`，这种方式同样有效。

### 预览事件

当一个被预览的事件开始时，流会发出一个 `event_start`，其中携带即将到来的事件的类型和 `id`：

```json
{
  "type": "event_start",
  "event": {
    "type": "agent.message",
    "id": "sevt_01abc..."
  }
}
```

对于 `agent.message`，开始事件之后会跟随携带增量文本的 `event_delta` 事件。每个增量在 `event_id` 中指明它所扩展的事件，并在 `delta.index` 中指明它所扩展的内容块：

```json
{
  "type": "event_delta",
  "event_id": "sevt_01abc...",
  "delta": {
    "type": "content_delta",
    "index": 0,
    "content": {
      "type": "text",
      "text": "Here is the summary"
    }
  }
}
```

对于 `agent.thinking`，只会发出 `event_start`，作为思考块已开始的信号。之后不会跟随 `event_delta` 事件。结束该预览的缓冲 `agent.thinking` 事件是一个进度信号，不携带任何思考内容。

与持久化事件不同，`event_start` 和 `event_delta` 没有自己的 `id` 或 `processed_at`。它们携带的唯一标识符是其所预览事件的 `id`。它们的类型字符串也是持久化事件 `{domain}.{action}` 命名约定的例外。

<Note>
  事件增量使用的传输格式与[流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)不同，这种差异是有意为之的。被预览的 `agent.message` 只有一个 `event_start`，之后仅跟随 `event_delta` 事件。没有针对每个内容块的开始或停止事件，被预览的事件本身也没有停止事件。增量类型是 `content_delta`，而不是 `content_block_delta`。为 Messages API 编写的累加器代码无法原样沿用。
</Note>

## 累加与协调

每个支持事件增量的 SDK 都包含一个[累加器辅助工具](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#sdk-accumulator-helpers)，可为您处理 `index` (csharp, go: `Index`) 的记录工作。当您需要自定义记录逻辑时，本节中的手动模式适用于每种语言。请将其应用于生成的事件类型。

在手动模式中，将预览文本保存在一个以 `(event_id, index)` 为键的临时映射中，并将缓冲事件视为记录。针对每个模型请求协调这两者。

一个轮次以单个 `session.status_running` 事件开始。在正常完成的轮次中，每个模型请求随后会按顺序产生以下事件：

1. `span.model_request_start`
2. `event_start`
3. `event_delta` 事件
4. 缓冲的 `agent.message`
5. [`span.model_request_end`](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types)（位于 **Span 事件**选项卡中）

在传输层面，以下是该序列中被预览的部分，与该连接的其他缓冲事件交错出现：

```text wrap
event_start     {"event": {"type": "agent.message", "id": "sevt_01abc..."}}
event_delta     {"event_id": "sevt_01abc...", "delta": {"type": "content_delta", "index": 0, "content": {"type": "text", "text": "..."}}}
...
agent.message   {"id": "sevt_01abc...", "content": [...]}
```

`event_delta` 行会针对每个文本片段重复一次。在每个事件到达时对其进行处理：

1. 收到 `event_start` 时，记下所宣告的 `id`。这些标识符始终一致：`event_start.event.id`、每个 `event_delta.event_id` 以及缓冲 `agent.message` 的 `id` 是同一个值。
2. 收到每个 `event_delta` 时，将 `delta.content.text` 追加到 `(event_id, delta.index)` 处的条目，并渲染当前累积的文本。某个 `index` 的第一个增量会创建该条目。
3. 当缓冲的 `agent.message` 到达时，按 `id` 进行匹配，丢弃累积的预览，改为渲染该消息的内容。
4. 收到 `span.model_request_end` 时，关闭所有尚未被其缓冲事件协调的预览。不会再有针对它的增量到来。如果轮次出错或被中断，缓冲事件可能永远不会到达，但 `span.model_request_end` 仍会到达。

该模式依赖于两项保证：

* 按到达顺序、以 `(event_id, index)` 为键拼接某个预览的增量，得到的是缓冲事件中 `content[index].text` 的前缀。它不一定是完整文本，因为增量可能会[在负载下被丢弃](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#limitations)。
* 一个连接对每个 `event_id` 最多发出一个 `event_start`，并且缓冲事件是该连接针对该 `id` 传递的最后一项内容。

### SDK 累加器辅助工具

每个 SDK 的辅助工具都会处理 `index` (csharp, go: `Index`) 的记录工作。Go、Java、Ruby 和 C# 的辅助工具还会按事件的 `id` (csharp, go: `ID`) 为累积中的预览建立键。使用 Python、TypeScript 和 PHP 的辅助工具时，请自行维护该映射，并将每个增量合并到其 `id` (csharp, go: `ID`) 对应的条目中。

以下示例选择启用 `agent.message` 预览，并将其与缓冲事件进行协调：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  # 预览快照，以事件 ID 为键。accumulate_managed_agents_event 将每个
  # event_start / event_delta 折叠为 agent.message 快照；缓冲的
  # agent.message 会替换它。
  previews: dict[str, BetaManagedAgentsAgentMessageEvent] = {}

  # 在此连接上选择启用 agent.message 预览
  with client.beta.sessions.events.stream(
      session.id, event_deltas=["agent.message"]
  ) as stream:
      client.beta.sessions.events.send(
          session.id,
          events=[
              {
                  "type": "user.message",
                  "content": [
                      {"type": "text", "text": "Describe the repo in one sentence."}
                  ],
              },
          ],
      )

      for event in stream:
          match event.type:
              case "event_start":
                  snapshot = accumulate_managed_agents_event(None, event)
                  if snapshot is not None:
                      previews[event.event.id] = snapshot
                  print(f"event_start             {event.event.type} {event.event.id}")
              case "event_delta":
                  preview = accumulate_managed_agents_event(
                      previews.get(event.event_id), event
                  )
                  if preview is not None:
                      previews[event.event_id] = preview
                      text = "".join(block.text for block in preview.content)
                      print(f"event_delta             preview: {text!r}")
              case "agent.message":
                  # 缓冲的事件即为正式记录：它会替换并关闭预览
                  preview = accumulate_managed_agents_event(
                      previews.pop(event.id, None), event
                  )
                  text = "".join(block.text for block in preview.content)
                  print(f"agent.message           {event.id} {text!r}")
              case "span.model_request_end":
                  # 不会再有增量到达。关闭所有
                  # 缓冲事件始终未到达的预览。
                  for event_id in previews:
                      print(f"span.model_request_end  closing preview for {event_id}")
                  previews.clear()
              case "session.status_idle":
                  break
  ```

  ```typescript TypeScript
  // 预览快照，以事件 ID 为键。`accumulateManagedAgentsEvent`
  // 将 event_start / event_delta 预览合并为 agent.message 快照。
  const previews = new Map<string, BetaManagedAgentsAgentMessageEvent>();

  // 仅为此连接启用 agent.message 预览
  const stream = await client.beta.sessions.events.stream(session.id, {
    event_deltas: ["agent.message"]
  });
  await client.beta.sessions.events.send(session.id, {
    events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "Summarize the repo README" }]
      }
    ]
  });

  deltas: for await (const event of stream) {
    switch (event.type) {
      case "event_start": {
        // 1. 记下公布的 ID 并打开快照。增量和
        //    缓冲事件携带相同的 ID。
        const preview = accumulateManagedAgentsEvent(undefined, event);
        if (preview) {
          previews.set(event.event.id, preview);
        }
        console.log(`event_start             ${event.event.type} ${event.event.id}`);
        break;
      }
      case "event_delta": {
        // 2. 将片段合并到快照中并渲染
        const preview = accumulateManagedAgentsEvent(previews.get(event.event_id), event);
        if (preview) {
          previews.set(event.event_id, preview);
          const text = preview.content
            .map((block) => (block.type === "text" ? block.text : ""))
            .join("");
          console.log(`event_delta             preview: ${JSON.stringify(text)}`);
        }
        break;
      }
      case "agent.message": {
        // 3. 缓冲事件即为正式记录：它替换并关闭预览
        const message = accumulateManagedAgentsEvent(previews.get(event.id), event);
        previews.delete(event.id);
        const text = message.content
          .map((block) => (block.type === "text" ? block.text : ""))
          .join("");
        console.log(`agent.message           ${event.id} ${JSON.stringify(text)}`);
        break;
      }
      case "span.model_request_end":
        // 4. 不会再有增量。关闭所有未被协调的预览。
        for (const eventId of previews.keys()) {
          console.log(`span.model_request_end  closing preview for ${eventId}`);
        }
        previews.clear();
        break;
      case "session.status_idle":
        break deltas;
    }
  }
  stream.controller.abort();
  ```

  ```csharp C#
  // 启用事件增量：agent.message 事件在生成时即可预览。
  using var stream = await client.Beta.Sessions.Events.WithRawResponse.StreamStreaming(
      session.ID,
      new() { EventDeltas = [BetaManagedAgentsDeltaType.AgentMessage] }
  );
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content =
              [
                  new BetaManagedAgentsTextBlock { Text = "Write a haiku about event streams." },
              ],
          },
      ],
  });

  // 按（事件 ID，内容索引）累积预览片段。随后缓冲的
  // agent.message 携带完整内容，因此它会替换
  // 累积的预览，而不是追加到其后。
  Dictionary<string, SortedDictionary<long, string>> previews = [];

  await foreach (var streamEvent in stream.Enumerate())
  {
      if (streamEvent.TryPickStartEvent(out var start))
      {
          // 为具有此 ID 的事件打开的预览。此流仅启用
          // agent.message 增量；TryPick* 返回 false 而非抛出异常，
          // 因此会跳过其他预览类型（包括以后新增的类型）。
          if (start.Event.TryPickAgentMessage(out var preview))
          {
              Console.WriteLine($"event_start             {preview.Type.GetString()} {preview.ID}");
          }
      }
      else if (streamEvent.TryPickDeltaEvent(out var delta))
      {
          // 在新索引处插入，在已有索引处追加
          if (!previews.TryGetValue(delta.EventID, out var fragments))
          {
              previews[delta.EventID] = fragments = [];
          }
          var index = delta.Delta.Index ?? 0;
          fragments[index] = fragments.GetValueOrDefault(index, "") + delta.Delta.Content.Text;
          Console.WriteLine($"event_delta             preview: {fragments[index]}");
      }
      else if (streamEvent.TryPickAgentMessageEvent(out var message))
      {
          // 增量是尽力而为的：丢弃预览并使用缓冲的事件
          previews.Remove(message.ID);
          var text = string.Concat(message.Content.Select(block =>
              block.TryPickBetaManagedAgentsTextBlock(out var textBlock) ? textBlock.Text : ""));
          Console.WriteLine($"agent.message           {message.ID} {text}");
      }
      else if (streamEvent.TryPickSpanModelRequestEndEvent(out _))
      {
          // 不会再有增量；关闭所有从未被协调的预览。
          foreach (var eventId in previews.Keys)
          {
              Console.WriteLine($"span.model_request_end  closing preview for {eventId}");
          }
          previews.Clear();
      }
      else if (streamEvent.TryPickSessionStatusIdleEvent(out _))
      {
          break;
      }
  }
  ```

  ```go Go
  	// 选择启用 agent.message 事件的增量预览
  	stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, anthropic.BetaSessionEventStreamParams{
  		EventDeltas: []anthropic.BetaManagedAgentsDeltaType{
  			anthropic.BetaManagedAgentsDeltaTypeAgentMessage,
  		},
  	})

  	if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  		Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  			OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  				Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  					OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  						Text: "Write a haiku about the ocean.",
  					},
  				}},
  			},
  		}},
  	}); err != nil {
  		panic(err)
  	}

  	// 累加器将 event_start / event_delta 片段合并为
  	// agent.message 快照，每个事件 ID 一个。零值即可直接使用。
  	var previews anthropic.BetaManagedAgentsEventAccumulator

  deltas:
  	for stream.Next() {
  		event := stream.Current()
  		previews.Accumulate(event)

  		switch event := event.AsAny().(type) {
  		case anthropic.BetaManagedAgentsStartEvent:
  			fmt.Printf("event_start             %s %s\n", event.Event.Type, event.Event.ID)
  		case anthropic.BetaManagedAgentsDeltaEvent:
  			fmt.Printf("event_delta             preview: %q\n", previews.AgentMessageText(event.EventID))
  		case anthropic.BetaManagedAgentsAgentMessageEvent:
  			// 缓冲的事件包含完整内容：累加器
  			// 用它替换预览
  			fmt.Printf("agent.message           %s %q\n", event.ID, previews.AgentMessageText(event.ID))
  		case anthropic.BetaManagedAgentsSpanModelRequestEndEvent:
  			// 此请求不会再有增量。累加器
  			// 在此丢弃其快照，关闭所有从未被
  			// 缓冲的 agent.message 协调的预览。
  			fmt.Println("span.model_request_end  no more deltas for this request")
  		case anthropic.BetaManagedAgentsSessionStatusIdleEvent:
  			break deltas
  		}
  	}
  	if err := stream.Err(); err != nil {
  		panic(err)
  	}
  	stream.Close()
  ```

  ```java Java
  // 预览文本，先按事件 ID 再按内容索引作为键。缓冲的 agent.message 会替换它。
  Map<String, Map<Long, StringBuilder>> previews = new HashMap<>();

  // 在此连接上选择启用 agent.message 预览
  try (var stream = client.beta().sessions().events().streamStreaming(
          session.id(),
          EventStreamParams.builder()
              .addEventDelta(BetaManagedAgentsDeltaType.AGENT_MESSAGE)
              .build()
  )) {
      client.beta().sessions().events().send(
          session.id(),
          EventSendParams.builder()
              .addEvent(BetaManagedAgentsUserMessageEventParams.builder()
                  .addTextContent("Describe the repo in one sentence.")
                  .build())
              .build()
      );

      deltas:
      for (var event : stream) {
          switch (event.type().value()) {
              case EVENT_START -> {
                  if (event.asEventStart().event().isAgentMessage()) {
                      var preview = event.asEventStart().event().asAgentMessage();
                      IO.println("event_start             " + preview._type() + " " + preview.id());
                  }
              }
              case EVENT_DELTA -> {
                  var eventDelta = event.asEventDelta();
                  var fragment = eventDelta.delta();
                  var buffer = previews
                      .computeIfAbsent(eventDelta.eventId(), _ -> new HashMap<>())
                      .computeIfAbsent(fragment.index().orElse(0L), _ -> new StringBuilder());
                  buffer.append(fragment.content().text());
                  IO.println("event_delta             preview: " + buffer);
              }
              case AGENT_MESSAGE -> {
                  // 缓冲的事件即为记录：丢弃其预览，渲染其内容
                  var message = event.asAgentMessage();
                  previews.remove(message.id());
                  var text = message.content().stream()
                      .flatMap(block -> block.text().stream())
                      .map(textBlock -> textBlock.text())
                      .collect(Collectors.joining());
                  IO.println("agent.message           " + message.id() + " " + text);
              }
              case SPAN_MODEL_REQUEST_END -> {
                  // 不会再有增量到达。关闭所有缓冲事件始终未到达的预览。
                  previews.keySet().forEach(eventId ->
                      IO.println("span.model_request_end  closing preview for " + eventId));
                  previews.clear();
              }
              case SESSION_STATUS_IDLE -> {
                  break deltas;
              }
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Sessions\BetaManagedAgentsDeltaEvent;
  use Anthropic\Beta\Sessions\BetaManagedAgentsDeltaType;
  use Anthropic\Beta\Sessions\BetaManagedAgentsStartEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionStatusIdleEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSpanModelRequestEndEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock;
  use Anthropic\Lib\Sessions\EventAccumulator;

  // 预览快照，以事件 ID 为键。EventAccumulator::accumulate() 将每个
  // event_start / event_delta 添加到 agent.message 快照中；缓冲的
  // agent.message 会替换它。
  $previews = [];

  // agent.message 快照或事件的文本
  $textOf = fn (ManagedAgentsAgentMessageEvent $message): string => implode(
      '',
      array_map(
          fn ($block) => $block->text,
          array_filter(
              $message->content,
              fn ($block) => $block instanceof ManagedAgentsTextBlock,
          ),
      ),
  );

  // 在此连接上选择启用 agent.message 预览
  $stream = $client->beta->sessions->events->streamStream(
      $session->id,
      eventDeltas: [BetaManagedAgentsDeltaType::AGENT_MESSAGE],
  );

  $client->beta->sessions->events->send(
      $session->id,
      events: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => 'Describe the repo in one sentence.']],
          ],
      ],
  );

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof BetaManagedAgentsStartEvent:
              $snapshot = EventAccumulator::accumulate(null, $event);
              if ($snapshot !== null) {
                  $previews[$event->event->id] = $snapshot;
              }
              printf("event_start             %s %s\n", $event->event->type, $event->event->id);
              break;
          case $event instanceof BetaManagedAgentsDeltaEvent:
              // 跳过已被替换或关闭的预览的迟到增量
              if (!isset($previews[$event->eventID])) {
                  break; // Break switch statement
              }
              $previews[$event->eventID] = EventAccumulator::accumulate($previews[$event->eventID], $event);
              printf("event_delta             preview: %s\n", json_encode($textOf($previews[$event->eventID])));
              break;
          case $event instanceof ManagedAgentsAgentMessageEvent:
              // 缓冲的事件即为正式记录：它会替换并关闭预览
              unset($previews[$event->id]);
              printf("agent.message           %s %s\n", $event->id, json_encode($textOf($event)));
              break;
          case $event instanceof ManagedAgentsSpanModelRequestEndEvent:
              // 不会再有增量到来。关闭所有
              // 缓冲事件始终未到达的预览。
              foreach (array_keys($previews) as $eventID) {
                  printf("span.model_request_end  closing preview for %s\n", $eventID);
              }
              $previews = [];
              break;
          case $event instanceof ManagedAgentsSessionStatusIdleEvent:
              break 2; // Break foreach loop
      }
  }
  $stream->close();
  ```

  ```ruby Ruby
  # 启用事件增量：agent.message 预览以增量片段形式流式传输。
  stream = client.beta.sessions.events.stream_events(
    session.id,
    event_deltas: [Anthropic::Beta::BetaManagedAgentsDeltaType::AGENT_MESSAGE]
  )

  client.beta.sessions.events.send_(
    session.id,
    events: [{
      type: "user.message",
      content: [{type: "text", text: "Give a one-sentence project tagline."}]
    }]
  )

  # 按 (event_id, index) 将预览片段累积到显式可变的
  # （`+""`）缓冲区中，以便 `<<` 原地追加。具有相同 ID 的
  # 已缓冲 agent.message 为权威版本，会替换增量累积的内容。
  buffers = Hash.new do |by_event, event_id|
    by_event[event_id] = Hash.new { |fragments, index| fragments[index] = +"" }
  end

  stream.each do |event|
    case event
    when Anthropic::Beta::BetaManagedAgentsStartEvent
      puts "event_start             #{event.event.type} #{event.event.id}"
    when Anthropic::Beta::BetaManagedAgentsDeltaEvent
      delta = event.delta
      fragment = delta.content.text
      buffers[event.event_id][delta.index || 0] << fragment
      puts "event_delta             preview: #{buffers[event.event_id][delta.index || 0].inspect}"
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
      # 替换：丢弃累积的预览并渲染完整事件。
      buffers.delete(event.id)
      puts "agent.message           #{event.id} #{event.content.map(&:text).join.inspect}"
    when Anthropic::Beta::Sessions::BetaManagedAgentsSpanModelRequestEndEvent
      # 不会再有增量。关闭所有从未被核对的预览。
      buffers.each_key { |event_id| puts "span.model_request_end  closing preview for #{event_id}" }
      buffers.clear
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionStatusIdleEvent
      break
    else
      # 忽略其他事件类型
    end
  end
  ```
</CodeGroup>

## 预览会话线程事件

在[多智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)会话中，每个会话线程都有自己的事件流。它接受相同的 `event_deltas[]` 参数和相同的值。

一个连接只预览它正在读取的线程。会话级流预览主线程，子线程的预览永远不会被交叉发布到会话级流。要在模型生成时观看子智能体的文本，请打开该子智能体的线程流。

线程流的路径以 `/threads/{thread_id}/stream` 结尾。`/events/stream` 仅存在于会话级别，因此不存在 `/threads/{thread_id}/events/stream` 端点。

`event_start` 和 `event_delta` 在线程流上的结构与在会话级流上相同，[累积与协调](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#accumulate-and-reconcile)模式可按原样适用。请为每个流连接运行一个累加器实例。

<CodeGroup>
  ```bash cURL
  # 线程流接受与会话流相同的 event_deltas[] 参数。
  # 对方括号进行百分号编码（%5B%5D）并为 URL 加引号。
  curl --fail-with-body -sS -N \
    "https://api.anthropic.com/v1/sessions/$SESSION_ID/threads/$THREAD_ID/stream?beta=true&event_deltas%5B%5D=agent.message" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "accept: text/event-stream"
  ```

  ```bash CLI
  # 线程流接受与会话流相同的 event_deltas 参数，
  # 每个要预览的事件类型对应一个 --event-delta 标志。
  ant beta:sessions:threads:events stream \
    --session-id "$SESSION_ID" \
    --thread-id "$THREAD_ID" \
    --event-delta agent.message
  ```

  ```python Python
  # 列出会话的线程并选择一个子线程：子线程的
  # parent_thread_id 非 null，而主线程的 parent_thread_id 为 null。
  child_thread = next(
      thread
      for thread in client.beta.sessions.threads.list(session.id)
      if thread.parent_thread_id is not None
  )

  # 子线程的流接受与会话流相同的 event_deltas
  # 参数。
  with client.beta.sessions.threads.events.stream(
      child_thread.id,
      session_id=session.id,
      event_deltas=["agent.message"],
  ) as stream:
      for event in stream:
          match event.type:
              case "event_delta":
                  print(event.delta.content.text, end="")
              case "agent.message":
                  # 缓冲事件是权威记录；渲染其内容。
                  print()
                  for block in event.content:
                      if block.type == "text":
                          print(block.text, end="")
                  print()
              case "session.thread_status_idle":
                  break
  ```

  ```typescript TypeScript
  // 列出会话的线程并选择一个子线程：子线程的
  // parent_thread_id 非 null，主线程的 parent_thread_id 为 null。
  let childThreadId: string | undefined;
  for await (const thread of client.beta.sessions.threads.list(session.id)) {
    if (thread.parent_thread_id !== null) {
      childThreadId = thread.id;
      break;
    }
  }
  if (!childThreadId) {
    throw new Error("No child thread found");
  }

  // 子线程的流接受与会话流相同的
  // event_deltas 参数。
  const stream = await client.beta.sessions.threads.events.stream(childThreadId, {
    session_id: session.id,
    event_deltas: ["agent.message"]
  });

  threadDeltas: for await (const event of stream) {
    switch (event.type) {
      case "event_delta":
        process.stdout.write(event.delta.content.text);
        break;
      case "agent.message": {
        // 缓冲事件是权威记录；渲染其内容。
        process.stdout.write("\n");
        const text = event.content
          .map((block) => (block.type === "text" ? block.text : ""))
          .join("");
        console.log(text);
        break;
      }
      case "session.thread_status_idle":
        break threadDeltas;
    }
  }
  stream.controller.abort();
  ```

  ```csharp C#
  // 列出会话的线程并选择一个子线程：子线程的
  // parent_thread_id 非 null，而主线程的 parent_thread_id 为 null。
  var threads = await client.Beta.Sessions.Threads.List(session.ID);
  var childThread = threads.Items.First(thread => thread.ParentThreadID is not null);

  // 子线程的流接受与会话流相同的 event_deltas
  // 参数。
  using var stream = await client.Beta.Sessions.Threads.Events.WithRawResponse.StreamStreaming(
      childThread.ID,
      new() { SessionID = session.ID, EventDeltas = [BetaManagedAgentsDeltaType.AgentMessage] }
  );

  await foreach (var streamEvent in stream.Enumerate())
  {
      if (streamEvent.TryPickDeltaEvent(out var delta))
      {
          Console.Write(delta.Delta.Content.Text);
      }
      else if (streamEvent.TryPickAgentMessageEvent(out var message))
      {
          // 缓冲的事件是权威记录；渲染其内容。
          Console.WriteLine();
          var text = string.Concat(message.Content.Select(block =>
              block.TryPickBetaManagedAgentsTextBlock(out var textBlock) ? textBlock.Text : ""));
          Console.WriteLine(text);
      }
      else if (streamEvent.TryPickSessionThreadStatusIdleEvent(out _))
      {
          break;
      }
  }
  ```

  ```go Go
  	// 列出会话的线程并选择一个子线程：子线程的 parent_thread_id
  	// 非 null，而主线程的 parent_thread_id 为 null。
  	var childThreadID string
  	threads := client.Beta.Sessions.Threads.ListAutoPaging(ctx, session.ID, anthropic.BetaSessionThreadListParams{})
  	for threads.Next() {
  		if thread := threads.Current(); thread.ParentThreadID != "" {
  			childThreadID = thread.ID
  			break
  		}
  	}
  	if err := threads.Err(); err != nil {
  		panic(err)
  	}

  	// 子线程的流接受与会话流相同的 event_deltas 参数；
  	// 每个流连接运行一个读取循环。
  	stream := client.Beta.Sessions.Threads.Events.StreamEvents(ctx, childThreadID, anthropic.BetaSessionThreadEventStreamParams{
  		SessionID: session.ID,
  		EventDeltas: []anthropic.BetaManagedAgentsDeltaType{
  			anthropic.BetaManagedAgentsDeltaTypeAgentMessage,
  		},
  	})

  threadDeltas:
  	for stream.Next() {
  		switch event := stream.Current().AsAny().(type) {
  		case anthropic.BetaManagedAgentsDeltaEvent:
  			fmt.Print(event.Delta.Content.Text)
  		case anthropic.BetaManagedAgentsAgentMessageEvent:
  			// 缓冲的事件是权威记录；渲染其内容。
  			fmt.Println()
  			// 具体类型列表：BetaManagedAgentsTextBlock
  			for _, block := range event.Content {
  				fmt.Print(block.Text)
  			}
  			fmt.Println()
  		case anthropic.BetaManagedAgentsSessionThreadStatusIdleEvent:
  			break threadDeltas
  		}
  	}
  	if err := stream.Err(); err != nil {
  		panic(err)
  	}
  	stream.Close()
  ```

  ```java Java
  // 列出会话的线程并选择一个子线程：子线程的 parent_thread_id
  // 非 null，而主线程的 parent_thread_id 为 null。
  var childThread = client.beta().sessions().threads().list(session.id()).autoPager().stream()
      .filter(thread -> thread.parentThreadId().isPresent())
      .findFirst()
      .orElseThrow();

  // 子线程的流接受与会话流相同的 event_deltas 参数。
  // 其参数类与会话级参数类的简单名称相同，因此需使用限定名。
  try (var stream = client.beta().sessions().threads().events().streamStreaming(
          childThread.id(),
          com.anthropic.models.beta.sessions.threads.events.EventStreamParams.builder()
              .sessionId(session.id())
              .addEventDelta(BetaManagedAgentsDeltaType.AGENT_MESSAGE)
              .build()
  )) {
      threadDeltas:
      for (var event : stream) {
          switch (event.type().value()) {
              case EVENT_DELTA -> IO.print(event.asEventDelta().delta().content().text());
              case AGENT_MESSAGE -> {
                  // 缓冲的事件是权威记录；渲染其内容。
                  IO.println();
                  event.asAgentMessage().content().forEach(block -> block.text().ifPresent(textBlock -> IO.print(textBlock.text())));
                  IO.println();
              }
              case SESSION_THREAD_STATUS_IDLE -> {
                  break threadDeltas;
              }
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Sessions\BetaManagedAgentsDeltaEvent;
  use Anthropic\Beta\Sessions\BetaManagedAgentsDeltaType;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsSessionThreadStatusIdleEvent;
  use Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock;

  // 列出会话的线程并选择一个子线程：子线程的 parentThreadID 非 null，
  // 而主线程的 parentThreadID 为 null。
  $childThread = null;
  foreach ($client->beta->sessions->threads->list($session->id)->pagingEachItem() as $thread) {
      if ($thread->parentThreadID !== null) {
          $childThread = $thread;
          break;
      }
  }

  // 子线程的流接受与会话流相同的
  // eventDeltas 参数。
  $stream = $client->beta->sessions->threads->events->streamStream(
      $childThread->id,
      sessionID: $session->id,
      eventDeltas: [BetaManagedAgentsDeltaType::AGENT_MESSAGE],
  );

  foreach ($stream as $event) {
      switch (true) {
          case $event instanceof BetaManagedAgentsDeltaEvent:
              echo $event->delta->content->text;
              break;
          case $event instanceof ManagedAgentsAgentMessageEvent:
              // 缓冲的事件是权威记录；渲染其内容。
              echo "\n";
              foreach ($event->content as $block) {
                  if ($block instanceof ManagedAgentsTextBlock) {
                      echo $block->text;
                  }
              }
              echo "\n";
              break;
          case $event instanceof ManagedAgentsSessionThreadStatusIdleEvent:
              break 2; // Break foreach loop
      }
  }
  $stream->close();
  ```

  ```ruby Ruby
  # 列出会话的线程并选择一个子线程：子线程的 parent_thread_id
  # 非 null，而主线程的 parent_thread_id 为 null。
  child_thread = client.beta.sessions.threads.list(session.id).to_enum.find { it.parent_thread_id }

  # 子线程的流接受与会话流相同的 event_deltas
  # 参数。
  stream = client.beta.sessions.threads.events.stream_events(
    child_thread.id,
    session_id: session.id,
    event_deltas: [Anthropic::Beta::BetaManagedAgentsDeltaType::AGENT_MESSAGE]
  )

  stream.each do |event|
    case event
    when Anthropic::Beta::BetaManagedAgentsDeltaEvent
      print event.delta.content.text
    when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
      # 已缓冲的事件是权威记录；渲染其内容。
      puts
      event.content.each { print it.text }
      puts
    when Anthropic::Beta::Sessions::BetaManagedAgentsSessionThreadStatusIdleEvent
      break
    else
      # 忽略其他事件类型
    end
  end
  ```
</CodeGroup>

读取循环在收到 [`session.thread_status_idle`](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types) 时退出，该事件在会话线程的轮次完成且线程进入空闲状态时发出。

## 限制

* **尽力而为：** 在负载下，服务器可能会丢弃某个事件的增量。发生这种情况时，您会收到文本的一个连续前缀，之后不再收到该事件的任何增量。缓冲的 `agent.message` 仍会完整到达。切勿将累积的预览视为最终结果。
* **重新连接时不重放：** 增量仅在选择启用的连接处于打开状态时传递给该连接。这同样适用于会话级流和每个会话线程流。在模型请求开始后才打开的连接不会收到该进行中事件的任何增量。无法重新请求错过的增量。
* **单一线程，仅限文本：** 预览仅涵盖[连接正在读取的线程](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#preview-session-thread-events)上的助手文本。工具使用、工具结果和 MCP 结果永远不会被预览。
* **永不持久化：** `event_start` 和 `event_delta` 仅存在于实时流中。它们不会出现在会话的事件历史（`GET /v1/sessions/{session_id}/events`）或任何会话线程的事件历史中。

## 预览故障排除

| 您看到的情况                                    | 含义                                                                                                                                                                                                 |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 流中有缓冲事件，但没有 `event_start` 或 `event_delta` | 您正在读取的连接没有选择启用，或者该轮次从未涉及您正在流式传输的线程。`event_deltas[]` 按连接生效，而不是按会话生效。要查找运行了哪个线程，请列出会话的线程（`GET /v1/sessions/{session_id}/threads`）。                                                                   |
| 流在预览期间中断                                  | 增量不会被重放。请按照[重新连接流程](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#reconnect-without-missing-events)操作：重新打开流并列出事件历史。该历史包含您断开连接期间发出的所有缓冲事件，包括您的预览正在等待的 `agent.message`。 |
| 流 URL 返回 404                              | 路径或某个 ID 有误，或者请求根本没有携带 managed-agents beta 标头。线程端点受 beta 限制，因此没有该标头时它们不存在。                                                                                                                         |
| 返回指明 `event_deltas` 的 400 错误              | 仅接受 `agent.message` 和 `agent.thinking`。                                                                                                                                                            |

## 后续步骤

<CardGroup cols={2}>
  <Card title="会话事件流" icon="lightning" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
    发送事件、流式传输响应，并在会话运行时中断或重定向会话。
  </Card>

  <Card title="多智能体编排" icon="stack" href="https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration">
    在单个会话中协调多个智能体。
  </Card>
</CardGroup>

## 可用性

事件增量在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台     | 模型                                                             |
| ---- | ------- | -------------------------------------------------------------- |
| Beta | 所有支持的平台 | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                         | HIPAA                             |
| ---- | ------------------------------------------------------------- | --------------------------------- |
| 状态   | 不具备资格                                                         | 不具备资格                             |
| 详细信息 | 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。 | 适用于所有 Managed Agents 子功能，包括自托管沙箱。 |

适用的端点：

* `/v1/agents`
* `/v1/sessions`
* `/v1/environments`
