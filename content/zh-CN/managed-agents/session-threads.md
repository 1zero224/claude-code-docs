---
title: 会话线程
url: https://platform.claude.com/docs/zh-CN/managed-agents/session-threads
description: 列出、中断和归档多智能体会话的线程，读取其事件，并跨线程处理工具权限。
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

在[多智能体会话](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)中，每个智能体都在自己的 **“session thread”（会话线程）** 中工作。本页介绍如何列出、中断和归档线程，线程发送的事件，以及工具权限如何跨线程运作。[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)也会创建会话线程。

## 主线程和会话线程

**会话级事件流**（`/v1/sessions/{session_id}/events/stream`）被视为 **“primary thread”（主线程）**，其中包含所有线程中全部活动的精简视图。您看不到[子智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#delegate-to-subagents)的完整活动，但可以看到其工作的开始和结束，以及工具权限请求等阻塞事件。

**会话线程**是您查看特定智能体活动的地方。

会话的 `status` (csharp, go: `Status`) 是所有智能体活动的汇总；如果至少有一个线程处于 `running` 状态，则整个会话的状态也为 `running`。正在运行的[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)也可以使会话保持 `running`，即使其所有线程都没有在工作。当没有线程在工作且有线程在等待您的客户端时，会话为 `idle`；请参阅[了解工作何时完成](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#know-when-the-work-is-done)。

[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)是会话所有线程共享的单一上限。达到上限时，各线程会独立暂停，每个线程的成本按该线程自身所用的模型计价。

<Note>
  一个会话同时最多可以有 25 个子线程。空闲线程在您归档之前都会计入，主线程不计入。主线程的智能体可以调用单个子智能体的多个副本，从而创建与同一个 `agent` 关联的多个线程。[顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)咨询线程不受此限制。[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)的线程也不受此限制。
</Note>

## 列出线程

按如下方式列出与会话关联的所有线程：

<CodeGroup>
  ```bash cURL
  curl -fsS "https://api.anthropic.com/v1/sessions/$SESSION_ID/threads" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:sessions:threads list --session-id "$SESSION_ID"
  ```

  ```python Python
  for thread in client.beta.sessions.threads.list(session.id):
      agent = thread.agent
      label = agent.type if agent.type == "advisor" else agent.name
      print(f"[{label}] {thread.status}")
  ```

  ```typescript TypeScript
  for await (const thread of client.beta.sessions.threads.list(session.id)) {
    const label = thread.agent.type === "advisor" ? thread.agent.type : thread.agent.name;
    console.log(`[${label}] ${thread.status}`);
  }
  ```

  ```csharp C#
  await foreach (var sessionThread in (await client.Beta.Sessions.Threads.List(session.ID)).Paginate())
  {
      var label = sessionThread.Agent.Name ?? sessionThread.Agent.Type.Raw();
      Console.WriteLine($"[{label}] {sessionThread.Status.Raw()}");
  }
  ```

  ```go Go
  threads := client.Beta.Sessions.Threads.ListAutoPaging(ctx, session.ID, anthropic.BetaSessionThreadListParams{})
  for threads.Next() {
  	thread := threads.Current()
  	label := cmp.Or(thread.Agent.Name, thread.Agent.Type)
  	fmt.Printf("[%s] %s\n", label, thread.Status)
  }
  if err := threads.Err(); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  for (var thread : client.beta().sessions().threads().list(session.id()).autoPager()) {
      var agent = thread.agent();
      var label = agent.isAgent() ? agent.asAgent().name() : agent.type().asString();
      IO.println("[" + label + "] " + thread.status());
  }
  ```

  ```php PHP
  foreach ($client->beta->sessions->threads->list($session->id)->pagingEachItem() as $thread) {
      $label = $thread->agent instanceof \Anthropic\Beta\Agents\BetaManagedAgentsAdvisor
          ? $thread->agent->type
          : $thread->agent->name;
      echo "[{$label}] {$thread->status}\n";
  }
  ```

  ```ruby Ruby
  client.beta.sessions.threads.list(session.id).auto_paging_each do |thread|
    agent = thread.agent
    label = agent.type == :advisor ? agent.type : agent.name
    puts "[#{label}] #{thread.status}"
  end
  ```
</CodeGroup>

完整列表包含主线程。主线程的 `parent_thread_id` (java: `parentThreadId`; php: `parentThreadID`; csharp, go: `ParentThreadID`) 为 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `""`)。其他所有线程都是子线程。除[运行的线程](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#a-runs-threads)外，`workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`) 均为 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `""`)。

要仅列出具有特定状态的线程，请在请求中添加 `statuses[]`，并重复该参数以指定多个状态，例如 `?statuses[]=running&statuses[]=idle`。省略该参数则返回所有状态的线程。

## 中断会话线程

发送带有 `session_thread_id` (java: `sessionThreadId`; php: `sessionThreadID`; csharp, go: `SessionThreadID`) 的 `user.interrupt` 以停止特定线程。省略 `session_thread_id` (java: `sessionThreadId`; php: `sessionThreadID`; csharp, go: `SessionThreadID`) 会中断会话中所有未归档的线程，包括主线程。在使用动态工作流的会话中，中断不会结束任何运行，而指定某个运行线程的中断不会停止任何内容。中断会关闭其他子线程的待处理工具调用，但不要依赖它来关闭运行线程的待处理工具调用。请参阅[在有打开的运行时中断会话](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#interrupt-a-session-with-runs-open)。

<CodeGroup>
  ```bash cURL
  curl -fsS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {"type": "user.interrupt", "session_thread_id": "'"$THREAD_ID"'"}
      ]
    }'
  ```

  ```bash CLI
  ant beta:sessions:events send \
    --session-id "$SESSION_ID" \
    --event "{type: user.interrupt, session_thread_id: $THREAD_ID}"
  ```

  ```python Python
  client.beta.sessions.events.send(
      session.id,
      events=[{"type": "user.interrupt", "session_thread_id": thread.id}],
  )
  ```

  ```typescript TypeScript
  await client.beta.sessions.events.send(session.id, {
    events: [{ type: "user.interrupt", session_thread_id: thread.id }]
  });
  ```

  ```csharp C#
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events = [new BetaManagedAgentsUserInterruptEventParams { SessionThreadID = thread.ID }],
  });
  ```

  ```go Go
  if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfUserInterrupt: &anthropic.BetaManagedAgentsUserInterruptEventParams{
  			SessionThreadID: anthropic.String(thread.ID),
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
          .addEvent(BetaManagedAgentsUserInterruptEventParams.builder()
              .sessionThreadId(thread.id())
              .build())
          .build());
  ```

  ```php PHP
  $client->beta->sessions->events->send(
      $session->id,
      events: [
          ['type' => 'user.interrupt', 'session_thread_id' => $thread->id],
      ],
  );
  ```

  ```ruby Ruby
  client.beta.sessions.events.send_(
    session.id,
    events: [{type: "user.interrupt", session_thread_id: thread.id}]
  )
  ```
</CodeGroup>

对于阻塞在 `requires_action` 上的子智能体的线程，中断会以错误工具结果（"Tool execution was interrupted before completion. Please retry."）关闭每个待处理的工具调用，并直接重新发出带有 `stop_reason: end_turn` 的 `session.thread_status_idle`；不会对模型进行采样。对于以 `end_turn` 或 `budget_reached` 处于空闲状态的子线程，中断不执行任何操作。指定已终止线程的中断会返回 400 错误。被中断的子线程不会向主线程的智能体发送其在轮次结束时通常发送的报告。当该智能体等待子线程时，在有其他内容到达之前（例如 `user.message` 或另一个线程的报告），它不会开始新的轮次。

## 归档会话线程

会话线程完成工作后，您可以选择将其归档。归档线程会释放其在 25 个子线程限制中占用的名额。服务器会自行归档[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)的线程。您无需归档它们，并且在运行处于打开状态时也无法归档。

<CodeGroup>
  ```bash cURL
  curl -fsS -X POST "https://api.anthropic.com/v1/sessions/$SESSION_ID/threads/$THREAD_ID/archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:sessions:threads archive \
    --session-id "$SESSION_ID" \
    --thread-id "$THREAD_ID"
  ```

  ```python Python
  archived = client.beta.sessions.threads.archive(thread.id, session_id=session.id)
  print(archived.status, archived.archived_at)
  ```

  ```typescript TypeScript
  const archived = await client.beta.sessions.threads.archive(thread.id, {
    session_id: session.id
  });
  console.log(archived.status, archived.archived_at);
  ```

  ```csharp C#
  var archived = await client.Beta.Sessions.Threads.Archive(thread.ID, new() { SessionID = session.ID });
  Console.WriteLine($"{archived.Status} {archived.ArchivedAt}");
  ```

  ```go Go
  archived, err := client.Beta.Sessions.Threads.Archive(ctx, thread.ID, anthropic.BetaSessionThreadArchiveParams{
  	SessionID: session.ID,
  })
  if err != nil {
  	panic(err)
  }
  fmt.Println(archived.Status, archived.ArchivedAt)
  ```

  ```java Java
  var archived = client.beta().sessions().threads().archive(
      thread.id(),
      ThreadArchiveParams.builder()
          .sessionId(session.id())
          .build());
  IO.println(archived.status() + " " + archived.archivedAt().orElseThrow());
  ```

  ```php PHP
  $archived = $client->beta->sessions->threads->archive($thread->id, sessionID: $session->id);
  echo "{$archived->status} {$archived->archivedAt->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  archived = client.beta.sessions.threads.archive(thread.id, session_id: session.id)
  puts "#{archived.status} #{archived.archived_at}"
  ```
</CodeGroup>

只有当线程处于 `idle` 状态时，归档才会成功。停留在 `requires_action` 上的线程被视为空闲，因此您可以直接将其归档。要归档正在运行的线程，请先中断它：

<CodeGroup>
  ```bash cURL
  # 中断该线程，然后将其归档
  curl -fsS "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "events": [
        {"type": "user.interrupt", "session_thread_id": "'"$THREAD_ID"'"}
      ]
    }'

  curl -fsS -X POST "https://api.anthropic.com/v1/sessions/$SESSION_ID/threads/$THREAD_ID/archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:sessions:events send \
    --session-id "$SESSION_ID" \
    --event "{type: user.interrupt, session_thread_id: $THREAD_ID}"

  ant beta:sessions:threads archive \
    --session-id "$SESSION_ID" \
    --thread-id "$THREAD_ID"
  ```

  ```python Python
  client.beta.sessions.events.send(
      session.id,
      events=[{"type": "user.interrupt", "session_thread_id": thread.id}],
  )
  archived = client.beta.sessions.threads.archive(thread.id, session_id=session.id)
  print(archived.status, archived.archived_at)
  ```

  ```typescript TypeScript
  await client.beta.sessions.events.send(session.id, {
    events: [{ type: "user.interrupt", session_thread_id: thread.id }]
  });
  const archived = await client.beta.sessions.threads.archive(thread.id, {
    session_id: session.id
  });
  console.log(archived.status, archived.archived_at);
  ```

  ```csharp C#
  await client.Beta.Sessions.Events.Send(session.ID, new()
  {
      Events = [new BetaManagedAgentsUserInterruptEventParams { SessionThreadID = thread.ID }],
  });
  archived = await client.Beta.Sessions.Threads.Archive(thread.ID, new() { SessionID = session.ID });
  Console.WriteLine($"{archived.Status} {archived.ArchivedAt}");
  ```

  ```go Go
  if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  	Events: []anthropic.BetaManagedAgentsEventParamsUnion{{
  		OfUserInterrupt: &anthropic.BetaManagedAgentsUserInterruptEventParams{
  			SessionThreadID: anthropic.String(thread.ID),
  		},
  	}},
  }); err != nil {
  	panic(err)
  }

  archived, err := client.Beta.Sessions.Threads.Archive(ctx, thread.ID, anthropic.BetaSessionThreadArchiveParams{
  	SessionID: session.ID,
  })
  if err != nil {
  	panic(err)
  }
  fmt.Println(archived.Status, archived.ArchivedAt)
  ```

  ```java Java
  client.beta().sessions().events().send(
      session.id(),
      EventSendParams.builder()
          .addEvent(BetaManagedAgentsUserInterruptEventParams.builder()
              .sessionThreadId(thread.id())
              .build())
          .build());

  archived = client.beta().sessions().threads().archive(
      thread.id(),
      ThreadArchiveParams.builder()
          .sessionId(session.id())
          .build());
  IO.println(archived.status() + " " + archived.archivedAt().orElseThrow());
  ```

  ```php PHP
  $client->beta->sessions->events->send(
      $session->id,
      events: [['type' => 'user.interrupt', 'session_thread_id' => $thread->id]],
  );
  $archived = $client->beta->sessions->threads->archive($thread->id, sessionID: $session->id);
  echo "{$archived->status} {$archived->archivedAt->format(DATE_ATOM)}\n";
  ```

  ```ruby Ruby
  client.beta.sessions.events.send_(
    session.id,
    events: [{type: "user.interrupt", session_thread_id: thread.id}]
  )
  archived = client.beta.sessions.threads.archive(thread.id, session_id: session.id)
  puts "#{archived.status} #{archived.archived_at}"
  ```
</CodeGroup>

## 主线程事件

这些事件在 `/v1/sessions/{session_id}/events/stream` 的主线程上呈现多智能体活动。消息方向类事件的命名是相对于其所在流的线程而言的：`agent.thread_message_received` 表示有消息从另一个线程到达此线程，`agent.thread_message_sent` 表示此线程发送了一条消息。例如，主线程的智能体委派的任务会以 `agent.thread_message_received` 事件的形式到达子线程自己的流。

| 类型                                 | 描述                                                                                                                                                                                                                                                                   |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `session.thread_created`           | 创建了一个线程。包含 `session_thread_id` (java: `sessionThreadId`; php: `sessionThreadID`; csharp, go: `SessionThreadID`) 和 `agent_name` (java, php: `agentName`; csharp, go: `AgentName`)。                                                                                    |
| `session.thread_status_running`    | 某个线程开始活动。                                                                                                                                                                                                                                                            |
| `session.thread_status_idle`       | 与该线程关联的智能体正在等待输入。包含一个 `stop_reason` (java, php: `stopReason`; csharp, go: `StopReason`)，指示智能体停止的原因。                                                                                                                                                                  |
| `session.thread_status_terminated` | 某个线程已终止且不再接受任何输入，例如因为它已被归档或遇到了不可恢复的错误。顾问线程在其咨询结束时也会终止。                                                                                                                                                                                                               |
| `agent.thread_message_received`    | 在主线程上，子智能体向主线程的智能体发送了报告或问题。包含 `from_session_thread_id` (java: `fromSessionThreadId`; php: `fromSessionThreadID`; csharp, go: `FromSessionThreadID`)、`from_agent_name` (java, php: `fromAgentName`; csharp, go: `FromAgentName`) 和 `content` (csharp, go: `Content`)。 |
| `agent.thread_message_sent`        | 在主线程上，主线程的智能体向子智能体发送了任务或后续消息。包含 `to_session_thread_id` (java: `toSessionThreadId`; php: `toSessionThreadID`; csharp, go: `ToSessionThreadID`)、`to_agent_name` (java, php: `toAgentName`; csharp, go: `ToAgentName`) 和 `content` (csharp, go: `Content`)。             |

顾问咨询会以保留名称 `anthropic.advisor` 发出这些相同的线程事件（在线程生命周期事件中作为 `agent_name` (java, php: `agentName`; csharp, go: `AgentName`)，在建议传递中作为 `from_agent_name` (java, php: `fromAgentName`; csharp, go: `FromAgentName`)）；有关事件顺序，请参阅[为会话配置顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。

[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)的线程在主事件流上的呈现方式如下：

* **生命周期事件：** 每个运行线程都会发送带有该运行的 `workflow_run_id` (java: `workflowRunId`; php: `workflowRunID`; csharp, go: `WorkflowRunID`) 的 `session.thread_created`，以及其 `session.thread_status_running`、`session.thread_status_idle` 和 `session.thread_status_terminated` 事件。
* **消息事件：** 运行线程的提示（一个 `agent.thread_message_received` 事件）保留在其自己的流上。
* **运行事件：** `workflow_run.*` 事件也会到达此流；请参阅[运行事件](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#run-events)。
* **等待您处理的工具调用：** 与任何子线程一样，运行线程中需要您的客户端处理的工具调用会被交叉发布到此流。请参阅[工具权限和自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads#tool-permissions-and-custom-tools)。

## 会话线程事件

关键事件会被代理转发到主线程。不过，您可能仍希望调查特定智能体的推理和工具调用。为此，请对关联的会话线程的事件进行流式传输或列出。

每个会话线程在 `/v1/sessions/{session_id}/threads/{thread_id}/stream` 都有自己的事件流，并且它接受与会话级流相同的 `event_deltas[]` 参数，因此您可以在模型生成子智能体文本时预览该文本。一个连接只预览它正在读取的线程：子线程的预览永远不会出现在会话级流上，因此要实时观察子智能体，请打开其自己的线程流。有关选择启用、累积和协调预览的信息，请参阅[预览会话线程事件](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas#preview-session-thread-events)。

在工作流运行中，服务器运行一个工作流：即由主线程的智能体编写的程序。在该运行的每个线程上，第一个 `agent.thread_message_received` 是工作流编写的提示。其 `from_session_thread_id` (java: `fromSessionThreadId`; php: `fromSessionThreadID`; csharp, go: `FromSessionThreadID`) 是主线程的 ID，并且 API 不会设置其 `from_agent_name` (java, php: `fromAgentName`; csharp, go: `FromAgentName`)。API 不保证提示的文本内容，因此请勿解析它。主线程流上该线程的 `session.thread_status_terminated` 事件会告诉您该线程已完成。没有任何事件记录它返回给工作流的结果。

线程的流不会重放较早的事件。在 `session.thread_created` 之后立即查看时，运行线程的事件列表可能为空，因为服务器在此之后才写入该线程的第一个事件。因此，请先打开线程的流，然后列出线程的事件，并跳过 `id` (csharp, go: `ID`) 已在列表中返回的每个流式事件。

<Tabs>
  <Tab title="流式传输会话线程事件">
    <CodeGroup>
      ```bash cURL
      curl -fsSN "https://api.anthropic.com/v1/sessions/$SESSION_ID/threads/$THREAD_ID/stream?beta=true" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01"
      ```

      ```bash CLI
      ant beta:sessions:threads:events stream \
        --session-id "$SESSION_ID" \
        --thread-id "$THREAD_ID"
      ```

      ```python Python
      with client.beta.sessions.threads.events.stream(
          thread.id,
          session_id=session.id,
      ) as stream:
          for event in stream:
              match event.type:
                  case "agent.message":
                      for block in event.content:
                          if block.type == "text":
                              print(block.text, end="")
                  case "session.thread_status_idle":
                      break
      ```

      ```typescript TypeScript
      const stream = await client.beta.sessions.threads.events.stream(thread.id, {
        session_id: session.id
      });

      loop: for await (const event of stream) {
        switch (event.type) {
          case "agent.message":
            for (const block of event.content) {
              if (block.type === "text") {
                process.stdout.write(block.text);
              }
            }
            break;
          case "session.thread_status_idle":
            break loop;
        }
      }
      ```

      ```csharp C#
      await foreach (var evt in client.Beta.Sessions.Threads.Events.StreamStreaming(thread.ID, new() { SessionID = session.ID }))
      {
          if (evt.Value is BetaManagedAgentsAgentMessageEvent message)
          {
              foreach (var block in message.Content)
              {
                  if (block.Value is BetaManagedAgentsTextBlock textBlock)
                  {
                      Console.Write(textBlock.Text);
                  }
              }
          }
          else if (evt.Value is BetaManagedAgentsSessionThreadStatusIdleEvent)
          {
              break;
          }
      }
      ```

      ```go Go
      	stream := client.Beta.Sessions.Threads.Events.StreamEvents(ctx, thread.ID, anthropic.BetaSessionThreadEventStreamParams{
      		SessionID: session.ID,
      	})
      	defer stream.Close()

      loop:
      	for stream.Next() {
      		event := stream.Current()
      		switch event.Type {
      		case "agent.message":
      			for _, block := range event.AsAgentMessage().Content {
      				if block.Type == "text" {
      					fmt.Print(block.Text)
      				}
      			}
      		case "session.thread_status_idle":
      			break loop
      		}
      	}
      	if err := stream.Err(); err != nil {
      		panic(err)
      	}
      ```

      ```java Java
      try (var streamResponse = client.beta().sessions().threads().events().streamStreaming(
          thread.id(),
          EventStreamParams.builder()
              .sessionId(session.id())
              .build()
      )) {
          loop:
          for (var event : streamResponse) {
              switch (event.type().value()) {
                  case AGENT_MESSAGE -> {
                      for (var block : event.asAgentMessage().content()) {
                          block.text().ifPresent(textBlock -> IO.print(textBlock.text()));
                      }
                  }
                  case SESSION_THREAD_STATUS_IDLE -> {
                      break loop;
                  }
              }
          }
      }
      ```

      ```php PHP
      $stream = $client->beta->sessions->threads->events->streamStream(
          $thread->id,
          sessionID: $session->id,
      );

      foreach ($stream as $event) {
          switch (true) {
              case $event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsAgentMessageEvent:
                  foreach ($event->content as $block) {
                      if ($block instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock) {
                          echo $block->text;
                      }
                  }
                  break;
              case $event instanceof \Anthropic\Beta\Sessions\Events\ManagedAgentsSessionThreadStatusIdleEvent:
                  break 2;
          }
      }
      ```

      ```ruby Ruby
      client.beta.sessions.threads.events.stream_events(thread.id, session_id: session.id).each do |event|
        case event
        when Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent
          event.content.each do |block|
            print block.text if block.is_a?(Anthropic::Beta::Sessions::BetaManagedAgentsTextBlock)
          end
        when Anthropic::Beta::Sessions::BetaManagedAgentsSessionThreadStatusIdleEvent
          break
        end
      end
      ```
    </CodeGroup>
  </Tab>

  <Tab title="列出会话线程事件">
    列出所有过去的会话线程事件以获取完整历史记录。

    <CodeGroup>
      ```bash cURL
      curl -fsS "https://api.anthropic.com/v1/sessions/$SESSION_ID/threads/$THREAD_ID/events" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01"
      ```

      ```bash CLI
      ant beta:sessions:threads:events list \
        --session-id "$SESSION_ID" \
        --thread-id "$THREAD_ID"
      ```

      ```python Python
      for event in client.beta.sessions.threads.events.list(
          thread.id,
          session_id=session.id,
      ):
          print(f"[{event.type}] {event.processed_at}")
      ```

      ```typescript TypeScript
      for await (const event of client.beta.sessions.threads.events.list(thread.id, {
        session_id: session.id
      })) {
        console.log(`[${event.type}] ${event.processed_at}`);
      }
      ```

      ```csharp C#
      var page = await client.Beta.Sessions.Threads.Events.List(thread.ID, new() { SessionID = session.ID });
      await foreach (var evt in page.Paginate())
      {
          Console.WriteLine($"[{evt.Type.Raw()}] {evt.ProcessedAt}");
      }
      ```

      ```go Go
      pager := client.Beta.Sessions.Threads.Events.ListAutoPaging(ctx, thread.ID, anthropic.BetaSessionThreadEventListParams{
      	SessionID: session.ID,
      })
      for pager.Next() {
      	event := pager.Current()
      	fmt.Printf("[%s] %s\n", event.Type, event.ProcessedAt)
      }
      if err := pager.Err(); err != nil {
      	panic(err)
      }
      ```

      ```java Java
      for (var event : client.beta().sessions().threads().events().list(
              thread.id(),
              EventListParams.builder()
                  .sessionId(session.id())
                  .build()
      ).autoPager()) {
          var processedAt = event.processedAt().map(OffsetDateTime::toString).orElse("pending");
          IO.println("[" + event.type().asString() + "] " + processedAt);
      }
      ```

      ```php PHP
      foreach (
          $client->beta->sessions->threads->events->list(
              $thread->id,
              sessionID: $session->id,
          )->pagingEachItem() as $event
      ) {
          echo "[{$event->type}] {$event->processedAt->format(DATE_RFC3339)}\n";
      }
      ```

      ```ruby Ruby
      client.beta.sessions.threads.events.list(
        thread.id,
        session_id: session.id
      ).auto_paging_each do |event|
        puts "[#{event.type}] #{event.processed_at}"
      end
      ```
    </CodeGroup>
  </Tab>
</Tabs>

## 工具权限和自定义工具

如果子智能体需要从您的客户端获取某些内容，例如运行工具调用的[权限](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#tool-confirmation)或[自定义工具的结果](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#handling-custom-tool-calls)，该事件会被交叉发布到**主线程**，并带有标识来源会话线程的 `session_thread_id`。在 `always_ask` 下，或在 [`auto`](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#let-the-server-evaluate-each-call-with-auto) 下服务器未能作出判定时，工具调用需要您的许可。

```json
{
  "type": "session.thread_status_idle",
  "id": "sevt_01ABC...",
  "session_thread_id": "sthr_01DEF...",
  "agent_name": "code-reviewer",
  "stop_reason": {
    "type": "requires_action",
    "event_ids": ["sevt_01XYZ..."]
  }
}
```

发布 `user.tool_confirmation`（带有 `tool_use_id` (java: `toolUseId`; php: `toolUseID`; csharp, go: `ToolUseID`)）或 `user.custom_tool_result`（带有 `custom_tool_use_id` (java: `customToolUseId`; php: `customToolUseID`; csharp, go: `CustomToolUseID`)）；服务器会自动将响应路由到正确的线程。该响应可能以不同的 `id` (csharp, go: `ID`) 值同时出现在主线程和子智能体的线程上。要匹配这两个副本，请比较 `type` (csharp, go: `Type`) 和 `tool_use_id` (java: `toolUseId`; php: `toolUseID`; csharp, go: `ToolUseID`)（或 `custom_tool_use_id` (java: `customToolUseId`; php: `customToolUseID`; csharp, go: `CustomToolUseID`)），而不是 `id` (csharp, go: `ID`)。

只有当没有线程处于 `running` 状态时，会话才会变为 `idle`，因此 `session.status_idle` 可能在子智能体发起调用很久之后才到达。您不必等待它：一旦交叉发布的 `agent.custom_tool_use` 事件到达，就立即发送 `user.custom_tool_result`。

在 `auto` 下，您的 `user.message` 事件可能使服务器允许原本会拒绝的调用。子智能体线程中的任何内容都不算作您的意图。您的客户端不会在那里发布消息，而主线程的智能体发送给子智能体的消息也不算数。当服务器在 `auto` 下拒绝某个调用时，不会交叉发布任何内容：该事件和错误工具结果只出现在子智能体自己的[线程流](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads#session-thread-events)上，并且子智能体会继续运行。

以下示例位于[工具确认处理程序](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#tool-confirmation)的事件循环内。对于 `stop_reason.event_ids` 中的每个 ID，它都会发送一个允许该调用的 `user.tool_confirmation`。相同的模式也适用于 `user.custom_tool_result`，您也可以在其 `agent.custom_tool_use` 事件到达后立即发送它。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  for event_id in stop.event_ids:
      client.beta.sessions.events.send(
          session.id,
          events=[
              {
                  "type": "user.tool_confirmation",
                  "tool_use_id": event_id,
                  "result": "allow",
              }
          ],
      )
  ```

  ```typescript TypeScript
  for (const eventId of stop.event_ids) {
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
  ```

  ```csharp C#
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
  ```

  ```go Go
  for _, eventID := range stopReason.EventIDs {
  	params := anthropic.BetaManagedAgentsUserToolConfirmationEventParams{
  		ToolUseID: eventID,
  		Result:    anthropic.BetaManagedAgentsUserToolConfirmationEventParamsResultAllow,
  	}
  	if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, anthropic.BetaSessionEventSendParams{
  		Events: []anthropic.BetaManagedAgentsEventParamsUnion{{OfUserToolConfirmation: &params}},
  	}); err != nil {
  		panic(err)
  	}
  }
  ```

  ```java Java
  for (var eventId : pendingToolUseIds) {
      client.beta().sessions().events().send(
          session.id(),
          EventSendParams.builder()
              .addEvent(BetaManagedAgentsUserToolConfirmationEventParams.builder()
                  .toolUseId(eventId)
                  .result(BetaManagedAgentsUserToolConfirmationEventParams.Result.ALLOW)
                  .build())
              .build()
      );
  }
  ```

  ```php PHP
  foreach ($event->stopReason->eventIDs as $eventId) {
      $client->beta->sessions->events->send($session->id, events: [[
          'type' => 'user.tool_confirmation',
          'tool_use_id' => $eventId,
          'result' => 'allow',
      ]]);
  }
  ```

  ```ruby Ruby
  event_ids.each do |event_id|
    client.beta.sessions.events.send_(session.id, events: [{
      type: "user.tool_confirmation",
      tool_use_id: event_id,
      result: "allow"
    }])
  end
  ```
</CodeGroup>

上述模式响应的是空闲事件所列出的调用。在主事件流上，子智能体的 `session.thread_status_idle` 事件可能会先于其 `stop_reason.event_ids` 所列出的 `agent.tool_use` 或 `agent.mcp_tool_use` 事件到达。针对事件尚未到达的调用发送 `user.tool_confirmation` 可能会返回 400。为避免这种情况，请在每个 [`evaluated_permission` (java, php: `evaluatedPermission`; csharp, go: `EvaluatedPermission`)](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#see-how-each-call-was-evaluated) 为 `ask` 的调用自身的事件到达主事件流时再对其作出响应。

## 可用性

会话线程在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启

| 状态   | 云平台     |
| ---- | ------- |
| Beta | 所有支持的平台 |

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
