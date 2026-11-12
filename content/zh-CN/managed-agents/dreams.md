---
title: 梦境
url: https://platform.claude.com/docs/zh-CN/managed-agents/dreams
description: 让 Claude 回顾过去的会话，以整理智能体的记忆并发现新的见解。
featureMetadata:
  status: research-preview
  betaHeader: managed-agents-2026-04-01, dreaming-2026-04-21
  zdr:
    eligibility: not-eligible
    note: 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。
  supportedModels:
    - claude-fable-5-2
    - claude-fable-5-1
    - claude-fable-5
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-sonnet-5
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-sonnet-4-6
  supportedPlatforms:
    Claude API: research-preview
---

<Note>
  梦境处于研究预览阶段。

  * [申请访问权限](https://claude.com/form/claude-managed-agents)
</Note>

<Tip>
  想要对梦境过程进行更多控制？Claude Cookbook 中的[使用动态工作流进行梦境](https://platform.claude.com/cookbook/managed-agents-cma-dreaming-with-a-dynamic-workflow)示例展示了如何使用您选择的提示和模型构建自己的梦境。由您决定它何时运行、读取哪些会话记录，以及如何处理这些记录。
</Tip>

智能体在工作时会写入其[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)，但这些写入是局部且增量的：经过多次会话后，记忆存储会积累重复、矛盾和过时的条目。

**Dreams**（梦境）让 Claude 来清理这些内容。梦境会读取现有的记忆存储以及过去的会话记录，然后生成一个新的、重新组织的记忆存储：合并重复项，用最新值替换过时或相互矛盾的条目，并发现新的见解。

输入存储永远不会被修改，因此您可以审查输出，如果不满意结果可以将其丢弃。

## 工作原理

**梦境**是一个异步作业，它接收：

* 一个预先存在的**记忆存储：** Claude 对其进行验证、去重和重新组织的存储，以及
* 1 到 100 个**会话：** Claude 从中挖掘模式和见解并整合到输出中的过去记录。

梦境会生成另一个**输出记忆存储**，与输入存储分开。在梦境开始 `running` 后不久，一旦工作流克隆了输入存储，输出存储 ID 就会出现在梦境的 `outputs` (csharp, go: `Outputs`) 数组中；处于 `running` 状态的梦境可能会短暂报告一个空的 `outputs` (csharp, go: `Outputs`) 数组。

## 创建梦境

<CodeGroup>
  ```bash cURL
  curl -s https://api.anthropic.com/v1/dreams \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01,dreaming-2026-04-21" \
    -H "content-type: application/json" \
    -d '{
      "inputs": [
        { "type": "memory_store", "memory_store_id": "'"$store_id"'" },
        { "type": "sessions", "session_ids": ["'"$session_a"'", "'"$session_b"'"] }
      ],
      "model": "claude-opus-5-5",
      "instructions": "Focus on coding-style preferences; ignore one-off debugging notes."
    }'
  ```

  ```bash CLI
  ant beta:dreams create <<YAML
  inputs:
    - type: memory_store
      memory_store_id: $store_id
    - type: sessions
      session_ids: [$session_a, $session_b]
  model: claude-opus-5-5
  instructions: Focus on coding-style preferences; ignore one-off debugging notes.
  YAML
  ```

  ```python Python
  dream = client.beta.dreams.create(
      inputs=[
          {"type": "memory_store", "memory_store_id": store_id},
          {"type": "sessions", "session_ids": [session_a, session_b]},
      ],
      model="claude-opus-5-5",
      instructions="Focus on coding-style preferences; ignore one-off debugging notes.",
  )
  print(dream.id)  # drm_01...
  ```

  ```typescript TypeScript
  let dream = await client.beta.dreams.create({
    inputs: [
      { type: "memory_store", memory_store_id: storeId },
      { type: "sessions", session_ids: [sessionA, sessionB] }
    ],
    model: "claude-opus-5-5",
    instructions: "Focus on coding-style preferences; ignore one-off debugging notes."
  });
  console.log(dream.id); // drm_01...
  ```

  ```csharp C#
  var dream = await client.Beta.Dreams.Create(new()
  {
      Inputs =
      [
          new BetaDreamMemoryStoreInput { MemoryStoreID = storeID },
          new BetaDreamSessionsInput { SessionIds = [sessionA, sessionB] },
      ],
      Model = "claude-opus-5-5",
      Instructions = "Focus on coding-style preferences; ignore one-off debugging notes.",
  });
  Console.WriteLine(dream.ID);  // drm_01...
  ```

  ```go Go
  dream, err := client.Beta.Dreams.New(ctx, anthropic.BetaDreamNewParams{
  	Inputs: []anthropic.BetaDreamInputUnionParam{
  		anthropic.BetaDreamInputParamOfMemoryStore(storeID),
  		anthropic.BetaDreamInputParamOfSessions([]string{sessionA, sessionB}),
  	},
  	Model: anthropic.BetaDreamNewParamsModelUnion{
  		OfString: anthropic.String("claude-opus-5-5"),
  	},
  	Instructions: anthropic.String("Focus on coding-style preferences; ignore one-off debugging notes."),
  })
  if err != nil {
  	panic(err)
  }
  fmt.Println(dream.ID) // drm_01...
  ```

  ```java Java
  var dream = client.beta().dreams().create(
      DreamCreateParams.builder()
          .addMemoryStoreInput(storeId)
          .addSessionsInput(List.of(sessionA, sessionB))
          .model("claude-opus-5-5")
          .instructions("Focus on coding-style preferences; ignore one-off debugging notes.")
          .build()
  );
  IO.println(dream.id());  // drm_01...
  ```

  ```php PHP
  $dream = $client->beta->dreams->create(
      inputs: [
          ['type' => 'memory_store', 'memory_store_id' => $storeId],
          ['type' => 'sessions', 'session_ids' => [$sessionA, $sessionB]],
      ],
      model: 'claude-opus-5-5',
      instructions: 'Focus on coding-style preferences; ignore one-off debugging notes.',
  );
  echo "{$dream->id}\n"; // drm_01...
  ```

  ```ruby Ruby
  dream = client.beta.dreams.create(
    inputs: [
      {type: "memory_store", memory_store_id: store_id},
      {type: "sessions", session_ids: [session_a, session_b]}
    ],
    model: "claude-opus-5-5",
    instructions: "Focus on coding-style preferences; ignore one-off debugging notes."
  )
  puts dream.id # drm_01...
  ```
</CodeGroup>

梦境的输入包括预先存在的记忆存储和一个会话数组。您选择的模型负责运行梦境管道。仅支持[部分模型](https://platform.claude.com/docs/zh-CN/managed-agents/dreams#feature-availability)。您可以选择传入 `instructions` (csharp, go: `Instructions`) 来引导梦境过程。请参阅[使用指令进行引导](https://platform.claude.com/docs/zh-CN/managed-agents/dreams#steer-with-instructions)。

响应是完整的 `dream` 资源，其 `status: "pending"`：

```json
{
  "type": "dream",
  "id": "drm_01AbCDefGhIjKlMnOpQrStUv",
  "status": "pending",
  "inputs": [
    { "type": "memory_store", "memory_store_id": "memstore_01Hx..." },
    { "type": "sessions", "session_ids": ["sesn_01...", "sesn_02..."] }
  ],
  "outputs": [],
  "model": { "id": "claude-opus-5-5" },
  "instructions": "Focus on coding-style preferences; ignore one-off debugging notes.",
  "session_id": null,
  "created_at": "2026-04-29T17:04:10Z",
  "ended_at": null,
  "archived_at": null,
  "usage": {
    "input_tokens": 0,
    "output_tokens": 0,
    "cache_read_input_tokens": 0,
    "cache_creation_input_tokens": 0
  },
  "error": null
}
```

<Tip>
  如果您只有会话记录而没有现有存储，请先[创建一个空的记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory#create-a-memory-store)，并将其作为 `memory_store` 输入传入。
</Tip>

### 使用指令进行引导

可选的 `instructions` (csharp, go: `Instructions`) 字段用于引导梦境管道综合哪些内容。它在整个管道中都会生效：仔细阅读哪些内容、合并或丢弃哪些内容，以及如何组织输出存储的结构。

将 `instructions` (csharp, go: `Instructions`) 用于高层次的综合指导，例如重点领域（"专注于编码风格偏好"）、需要原样保留的内容，或您希望在整个存储中应用的输出约定。该管道是对输入进行的综合处理，而不是作用于存储文本的编辑器，因此针对特定行的命令式指令（"将句子 X 改为 Y"、"修正 Z 部分中的计数"）通常不会产生任何变化。如需对单个记忆进行针对性编辑，请直接对输出存储使用 [Memory Stores API](https://platform.claude.com/docs/zh-CN/managed-agents/memory#view-and-edit-memories)。

## 跟踪进度

梦境以异步方式运行，通常需要几分钟到几个小时，具体取决于输入记录的数量。按 ID 轮询梦境以检查状态：

<CodeGroup>
  ```bash cURL
  curl -s "https://api.anthropic.com/v1/dreams/$dream_id" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01,dreaming-2026-04-21"
  ```

  ```bash CLI
  ant beta:dreams retrieve --dream-id "$dream_id"
  ```

  ```python Python
  while dream.status in ("pending", "running"):
      time.sleep(10)
      dream = client.beta.dreams.retrieve(dream.id)
      print(f"status={dream.status} input_tokens={dream.usage.input_tokens}")
  ```

  ```typescript TypeScript
  while (dream.status === "pending" || dream.status === "running") {
    await new Promise((resolve) => setTimeout(resolve, 10_000));
    dream = await client.beta.dreams.retrieve(dream.id);
    console.log(`status=${dream.status} input_tokens=${dream.usage.input_tokens}`);
  }
  ```

  ```csharp C#
  while (dream.Status.Value() is BetaDreamStatus.Pending or BetaDreamStatus.Running)
  {
      await Task.Delay(TimeSpan.FromSeconds(10));
      dream = await client.Beta.Dreams.Retrieve(dream.ID);
      Console.WriteLine($"status={dream.Status.Raw()} input_tokens={dream.Usage.InputTokens}");
  }
  ```

  ```go Go
  for dream.Status == anthropic.BetaDreamStatusPending || dream.Status == anthropic.BetaDreamStatusRunning {
  	time.Sleep(10 * time.Second)
  	dream, err = client.Beta.Dreams.Get(ctx, dream.ID, anthropic.BetaDreamGetParams{})
  	if err != nil {
  		panic(err)
  	}
  	fmt.Printf("status=%s input_tokens=%d\n", dream.Status, dream.Usage.InputTokens)
  }
  ```

  ```java Java
  while (dream.status().equals(BetaDreamStatus.PENDING)
          || dream.status().equals(BetaDreamStatus.RUNNING)) {
      Thread.sleep(10_000);
      dream = client.beta().dreams().retrieve(dream.id());
      IO.println("status=" + dream.status() + " input_tokens=" + dream.usage().inputTokens());
  }
  ```

  ```php PHP
  while (in_array($dream->status, [BetaDreamStatus::PENDING->value, BetaDreamStatus::RUNNING->value], true)) {
      sleep(10);
      $dream = $client->beta->dreams->retrieve($dream->id);
      echo "status={$dream->status} input_tokens={$dream->usage->inputTokens}\n";
  }
  ```

  ```ruby Ruby
  while %i[pending running].include?(dream.status)
    sleep 10
    dream = client.beta.dreams.retrieve(dream.id)
    puts "status=#{dream.status} input_tokens=#{dream.usage.input_tokens}"
  end
  ```
</CodeGroup>

### 生命周期

| `status` (csharp, go: `Status`) | 含义                                                  |
| ------------------------------- | --------------------------------------------------- |
| `pending`                       | 梦境已成功创建并进入队列。                                       |
| `running`                       | 管道正在处理中。`usage` (csharp, go: `Usage`) 会随着工作进展而更新。   |
| `completed`                     | 已成功完成。`outputs` (csharp, go: `Outputs`) 的值即为新的记忆存储。 |
| `failed`                        | 梦境运行因错误而结束。输出记忆存储保持原样，包含失败前已写入的所有内容。                |
| `canceled`                      | 梦境运行已取消。输出记忆存储保持原样。                                 |

### 观察管道运行

一旦梦境处于 `running` 状态，其 `session_id` (java: `sessionId`; php: `sessionID`; csharp, go: `SessionID`) 字段就会指向运行该管道的底层[会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)。您可以对该会话的[事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)进行流式传输，以实时观察梦境正在读取和写入的内容。当梦境达到终止状态时，该会话会被归档（而不是删除），因此之后仍可查看其记录。

## 使用输出

当 `status` (csharp, go: `Status`) 达到 `completed` 时，`outputs` (csharp, go: `Outputs`) 中的 `memory_store` 条目会引用一个已完全填充的存储。它是您工作区中的一个普通记忆存储。使用 [Memory Stores API](https://platform.claude.com/docs/zh-CN/managed-agents/memory#view-and-edit-memories) 或在 Claude Console 中审查它，然后选择：

* **使用它：** 在未来的会话中将其作为 `memory_store` 资源附加，以替代（或搭配）输入记忆存储，或
* **丢弃它：** [删除该记忆存储](https://platform.claude.com/docs/zh-CN/api/beta/memory_stores/delete)或[归档该记忆存储](https://platform.claude.com/docs/zh-CN/api/beta/memory_stores/archive)。

<CodeGroup>
  ```bash cURL
  # $output_store_id 是已完成梦境的 memory_store 输出的 memory_store_id
  curl -s https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$agent_id"'",
      "environment_id": "'"$environment_id"'",
      "resources": [
        { "type": "memory_store", "memory_store_id": "'"$output_store_id"'" }
      ]
    }'
  ```

  ```bash CLI
  # $output_store_id 是已完成梦境的 memory_store 输出的 memory_store_id
  ant beta:sessions create <<YAML
  agent: $agent_id
  environment_id: $environment_id
  resources:
    - type: memory_store
      memory_store_id: $output_store_id
  YAML
  ```

  ```python Python
  # 梦境结束后，输出中保存着重建的记忆存储
  output_store_id = next(
      output.memory_store_id for output in dream.outputs if output.type == "memory_store"
  )

  session = client.beta.sessions.create(
      agent=agent_id,
      environment_id=environment_id,
      resources=[
          {"type": "memory_store", "memory_store_id": output_store_id},
      ],
  )
  ```

  ```typescript TypeScript
  // dream 结束后，输出中保存着重建的内存存储
  const output = dream.outputs.find((entry) => entry.type === "memory_store");
  const outputStoreId = output!.memory_store_id;

  await client.beta.sessions.create({
    agent: agentId,
    environment_id: environmentId,
    resources: [{ type: "memory_store", memory_store_id: outputStoreId }]
  });
  ```

  ```csharp C#
  var output = dream.Outputs.FirstOrDefault(entry => entry.Type.GetString() == "memory_store");
  if (output is { MemoryStoreID: var outputStoreID })
  {
      await client.Beta.Sessions.Create(new()
      {
          Agent = agentID,
          EnvironmentID = environmentID,
          Resources =
          [
              new BetaManagedAgentsMemoryStoreResourceParam { MemoryStoreID = outputStoreID },
          ],
      });
  }
  ```

  ```go Go
  for _, output := range dream.Outputs {
  	if output.Type != "memory_store" {
  		continue
  	}
  	outputStoreID := output.MemoryStoreID

  	session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  		Agent: anthropic.BetaSessionNewParamsAgentUnion{
  			OfString: anthropic.String(agentID),
  		},
  		EnvironmentID: environmentID,
  		Resources: []anthropic.BetaSessionNewParamsResourceUnion{{
  			OfMemoryStore: &anthropic.BetaManagedAgentsMemoryStoreResourceParam{
  				MemoryStoreID: outputStoreID,
  			},
  		}},
  	})
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println(session.ID)
  	break
  }
  ```

  ```java Java
  var output = dream.outputs().stream()
      .filter(entry -> entry._type().equals(JsonValue.from("memory_store")))
      .findFirst();
  if (output.isPresent()) {
      var outputStoreId = output.get().memoryStoreId();

      var session = client.beta().sessions().create(
          SessionCreateParams.builder()
              .agent(agentId)
              .environmentId(environmentId)
              .addMemoryStoreResource(outputStoreId)
              .build()
      );
  }
  ```

  ```php PHP
  $matches = array_filter($dream->outputs, fn($output) => $output->type === 'memory_store');
  $output = $matches ? reset($matches) : null;
  if ($output !== null) {
      $session = $client->beta->sessions->create(
          agent: $agentId,
          environmentID: $environmentId,
          resources: [
              ['type' => 'memory_store', 'memory_store_id' => $output->memoryStoreID],
          ],
      );
  }
  ```

  ```ruby Ruby
  output = dream.outputs.find { it.type == :memory_store }
  if output
    client.beta.sessions.create(
      agent: agent_id,
      environment_id: environment_id,
      resources: [
        {type: "memory_store", memory_store_id: output.memory_store_id}
      ]
    )
  end
  ```
</CodeGroup>

梦境本身永远不会删除或修改其输入。在 `failed` 或 `canceled` 状态下，输出存储会保留部分内容，以便您检查停止前已生成的内容；如果不需要，请通过 Memory Stores API 将其清理。

<Warning>
  当梦境处于 `pending` 或 `running` 状态时，400 保护仅适用于归档梦境本身，而不适用于其存储。在运行期间归档或删除*输入*记忆存储（或删除输入会话）将导致梦境失败，并返回 `input_memory_store_unavailable` 或 `input_session_unavailable`。
</Warning>

## 取消梦境

取消操作会立即将 `pending` 或 `running` 状态的梦境转为 `canceled`。取消一个已处于 `canceled` 状态的梦境是幂等的空操作；取消 `completed` 或 `failed` 状态的梦境会返回 400。

<Note>
  取消后，在进行中的工作逐渐结束期间，梦境的 `usage` (csharp, go: `Usage`) 字段可能会在几秒钟内继续更新。如果您需要最终计数，请轮询梦境直到 `usage` (csharp, go: `Usage`) 稳定下来。
</Note>

<CodeGroup>
  ```bash cURL
  curl -s -X POST "https://api.anthropic.com/v1/dreams/$dream_id/cancel" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01,dreaming-2026-04-21"
  ```

  ```bash CLI
  ant beta:dreams cancel --dream-id "$dream_id"
  ```

  ```python Python
  client.beta.dreams.cancel(dream.id)
  ```

  ```typescript TypeScript
  await client.beta.dreams.cancel(dream.id);
  ```

  ```csharp C#
  await client.Beta.Dreams.Cancel(dream.ID);
  ```

  ```go Go
  dream, err = client.Beta.Dreams.Cancel(ctx, dream.ID, anthropic.BetaDreamCancelParams{})
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().dreams().cancel(dream.id());
  ```

  ```php PHP
  $client->beta->dreams->cancel($dream->id);
  ```

  ```ruby Ruby
  client.beta.dreams.cancel(dream.id)
  ```
</CodeGroup>

## 归档梦境

归档操作会在已达到终止状态（`completed`、`failed` 或 `canceled`）的梦境上设置 `archived_at` (java, php: `archivedAt`; csharp, go: `ArchivedAt`)；`status` (csharp, go: `Status`) 保持不变。已归档的梦境不会出现在默认的列表响应中，但仍可按 ID 读取。归档一个已归档的梦境是幂等的空操作。归档 `pending` 或 `running` 状态的梦境会返回 400；请先取消它。不支持取消归档。

<CodeGroup>
  ```bash cURL
  curl -s -X POST "https://api.anthropic.com/v1/dreams/$dream_id/archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01,dreaming-2026-04-21"
  ```

  ```bash CLI
  ant beta:dreams archive --dream-id "$dream_id"
  ```

  ```python Python
  client.beta.dreams.archive(dream.id)
  ```

  ```typescript TypeScript
  await client.beta.dreams.archive(dream.id);
  ```

  ```csharp C#
  await client.Beta.Dreams.Archive(dream.ID);
  ```

  ```go Go
  dream, err = client.Beta.Dreams.Archive(ctx, dream.ID, anthropic.BetaDreamArchiveParams{})
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().dreams().archive(dream.id());
  ```

  ```php PHP
  $client->beta->dreams->archive($dream->id);
  ```

  ```ruby Ruby
  client.beta.dreams.archive(dream.id)
  ```
</CodeGroup>

归档梦境不会影响其输出记忆存储；请通过 [Memory Stores API](https://platform.claude.com/docs/zh-CN/managed-agents/memory#view-and-edit-memories) 单独管理该存储。

## 列出梦境

返回工作区中所有未归档的梦境，按最新优先排序。使用 `limit` (csharp, go: `Limit`)（默认 20，最大 100）和 `page` (csharp, go: `Page`) 游标进行分页。将 `include_archived` (java, php: `includeArchived`; csharp, go: `IncludeArchived`) 设置为 `true` 以包含已归档的梦境。

<CodeGroup>
  ```bash cURL
  curl -s "https://api.anthropic.com/v1/dreams?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01,dreaming-2026-04-21"
  ```

  ```bash CLI
  ant beta:dreams list --limit 20
  ```

  ```python Python
  for listed_dream in client.beta.dreams.list(limit=20):
      print(listed_dream.id, listed_dream.status)
  ```

  ```typescript TypeScript
  for await (const listedDream of client.beta.dreams.list({ limit: 20 })) {
    console.log(listedDream.id, listedDream.status);
  }
  ```

  ```csharp C#
  var page = await client.Beta.Dreams.List(new() { Limit = 20 });
  await foreach (var listed in page.Paginate())
  {
      Console.WriteLine($"{listed.ID} {listed.Status.Raw()}");
  }
  ```

  ```go Go
  dreams := client.Beta.Dreams.ListAutoPaging(ctx, anthropic.BetaDreamListParams{
  	Limit: anthropic.Int(20),
  })
  for dreams.Next() {
  	listed := dreams.Current()
  	fmt.Println(listed.ID, listed.Status)
  }
  if err := dreams.Err(); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  for (var listedDream : client.beta().dreams().list(
      DreamListParams.builder()
          .limit(20)
          .build()
  ).autoPager()) {
      IO.println(listedDream.id() + " " + listedDream.status());
  }
  ```

  ```php PHP
  foreach ($client->beta->dreams->list(limit: 20)->pagingEachItem() as $dream) {
      echo "{$dream->id} {$dream->status}\n";
  }
  ```

  ```ruby Ruby
  client.beta.dreams.list(limit: 20).auto_paging_each do
    puts "#{it.id} #{it.status}"
  end
  ```
</CodeGroup>

## 错误

以下是可能出现的梦境错误的非详尽列表。

| `error.type`                      | 发生时机                        |
| --------------------------------- | --------------------------- |
| `timeout`                         | 管道超出了其运行时间预算。               |
| `internal_error`                  | 未分类的管道故障。                   |
| `memory_store_org_limit_exceeded` | 在管道配置工作存储时，您的组织达到了记忆存储数量上限。 |
| `input_memory_store_too_large`    | 输入记忆存储超出了管道的大小限制。           |
| `input_memory_store_unavailable`  | 输入记忆存储在梦境创建后被归档或删除。         |
| `input_session_unavailable`       | 某个输入会话在梦境创建后被删除。            |

## 计费

梦境按您所选模型的标准 API 令牌费率计费；资源上的 `usage` (csharp, go: `Usage`) 会报告确切的总量。成本大致随输入会话的数量和长度线性增长。建议先从一小批会话开始，在对整理质量满意后再扩大规模。

## 限制

在此功能处于研究预览期间，梦境创建适用默认的速率限制。如果您需要更高的限制，请[联系支持团队](https://support.claude.com)。

## 可用性

梦境在部分模型上处于研究预览阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台        | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 研究预览 | Claude API | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`
* `dreaming-2026-04-21`

梦境端点需要 `dreaming-2026-04-21` beta 标头。仅凭 `managed-agents-2026-04-01` 标头无法访问梦境。会话调用使用 `managed-agents-2026-04-01`，记忆存储调用使用 `agent-memory-2026-07-22`。SDK 会为每个调用设置正确的标头。

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
