---
title: 会话操作
url: https://platform.claude.com/docs/zh-CN/managed-agents/session-operations
description: 检索、列出、更新、归档和删除 Claude Managed Agents 会话。
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

会话存在后，可使用这些操作来读取、更新、归档或删除它。有关创建会话并向其发送工作的信息，请参阅[启动会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)。

## 会话状态

会话会经历以下状态。有关会话生命周期，请参阅[启动会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)。

| 状态             | 描述                                                                                                                                                                                                                                                                                                                                                                                        |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `idle`         | 智能体正在等待输入，包括用户消息或工具确认。未使用 `initial_events` (java, php: `initialEvents`; csharp, go: `InitialEvents`) 创建的会话以 `idle` 状态开始。当[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#while-a-run-is-open)仍处于打开状态时，会话也可能处于 `idle` 状态，因此仅凭 `idle` 并不意味着工作已完成。请参阅[了解工作何时完成](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#know-when-the-work-is-done)。 |
| `running`      | 智能体正在主动执行。                                                                                                                                                                                                                                                                                                                                                                                |
| `rescheduling` | 发生了瞬时错误，正在自动重试。                                                                                                                                                                                                                                                                                                                                                                           |
| `terminated`   | 会话已结束，原因是发生了不可恢复的错误或会话已被归档。完成工作的会话会进入 `idle`，而不是 `terminated`。                                                                                                                                                                                                                                                                                                                            |

## 更新智能体配置

您可以在会话中途更新会话的 `agent.tools` (csharp, go: `Agent.Tools`) 和 `agent.mcp_servers` (java, php: `agent.mcpServers`; csharp: `Agent.McpServers`; go: `Agent.MCPServers`)，包括权限策略和每个工具的 Web 设置（例如[域名过滤器](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions#change-the-lists-mid-session)），而无需创建新的智能体版本。更新仅作用于当前会话，不会应用于底层智能体。更新后的 `allowed_domains` (java, php: `allowedDomains`; csharp, go: `AllowedDomains`) 和 `blocked_domains` (java, php: `blockedDomains`; csharp, go: `BlockedDomains`) 将应用于会话的剩余部分。

会话创建后，只有智能体的 `tools` (csharp, go: `Tools`) 和 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 可以更改。若要使用与智能体不同的 `model` (csharp, go: `Model`)、`system` (csharp, go: `System`; ruby: `system_`) 或 `skills` (csharp, go: `Skills`) 值运行会话，请在创建会话时使用[智能体配置覆盖](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)。智能体的模型配置（包括其 [`inference_geo`](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency) 固定设置）同样无法在会话进行中更改：请在保存智能体时设置该固定设置，或在创建会话时通过 `model` (csharp, go: `Model`) 覆盖为单个会话设置或清除它。智能体配置的 `system` (csharp, go: `System`; ruby: `system_`) 字段在会话的整个生命周期内保持不变。在支持该功能的模型上，您仍然可以通过发送 [`system.message` 事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#send-system-messages)在会话进行中追加系统级指导。

`tools` (csharp, go: `Tools`) 或 `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 更新的语义是完全替换：提供的数组即为新值。若要保留现有条目，请先 `GET` 会话，修改数组，然后再 `POST` 回去。

会话必须处于 `idle` 状态才能更新智能体。要在会话运行时更新智能体，请单独发送一个 [`user.interrupt` 事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#interrupt-the-agent)，并等待会话变为 `idle`。如果有[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#while-a-run-is-open)处于打开状态，无论它正在运行还是已暂停，更新都会返回 400 错误，即使会话处于 `idle` 状态也是如此。中断不会结束运行，因此在您中断会话后，更新仍会失败。请发送一条 `user.message` [要求智能体停止其运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#interrupt-a-session-with-runs-open)，或等待所有运行结束。

<CodeGroup>
  ```bash cURL
  curl -sS --fail-with-body "https://api.anthropic.com/v1/sessions/$SESSION_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": {
        "tools": [
          {"type": "agent_toolset_20260401"},
          {"type": "mcp_toolset", "mcp_server_name": "linear"}
        ],
        "mcp_servers": [
          {"type": "url", "name": "linear", "url": "https://mcp.linear.app/sse"}
        ]
      }
    }'
  ```

  ```bash CLI
  ant beta:sessions update --session-id "$SESSION_ID" <<'YAML'
  agent:
    tools:
      - type: agent_toolset_20260401
      - type: mcp_toolset
        mcp_server_name: linear
    mcp_servers:
      - type: url
        name: linear
        url: https://mcp.linear.app/sse
  YAML
  ```

  ```python Python
  client.beta.sessions.update(
      session.id,
      agent={
          "tools": [
              {"type": "agent_toolset_20260401"},
              {"type": "mcp_toolset", "mcp_server_name": "linear"},
          ],
          "mcp_servers": [
              {"type": "url", "name": "linear", "url": "https://mcp.linear.app/sse"}
          ],
      },
  )
  ```

  ```typescript TypeScript
  await client.beta.sessions.update(session.id, {
    agent: {
      tools: [
        { type: "agent_toolset_20260401" },
        { type: "mcp_toolset", mcp_server_name: "linear" }
      ],
      mcp_servers: [{ type: "url", name: "linear", url: "https://mcp.linear.app/sse" }]
    }
  });
  ```

  ```csharp C#
  await client.Beta.Sessions.Update(session.ID, new()
  {
      Agent = new()
      {
          Tools =
          [
              new BetaManagedAgentsAgentToolset20260401Params(),
              new BetaManagedAgentsMcpToolsetParams { McpServerName = "linear" },
          ],
          McpServers =
          [
              new()
              {
                  Name = "linear",
                  Url = "https://mcp.linear.app/sse",
              },
          ],
      },
  });
  ```

  ```go Go
  _, err = client.Beta.Sessions.Update(ctx, session.ID, anthropic.BetaSessionUpdateParams{
  	Agent: anthropic.BetaManagedAgentsSessionAgentUpdateParam{
  		Tools: []anthropic.BetaManagedAgentsSessionAgentUpdateToolUnionParam{
  			{
  				OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{},
  			},
  			{
  				OfMCPToolset: &anthropic.BetaManagedAgentsMCPToolsetParams{
  					MCPServerName: "linear",
  				},
  			},
  		},
  		MCPServers: []anthropic.BetaManagedAgentsURLMCPServerParams{
  			{
  				Name: "linear",
  				URL:  "https://mcp.linear.app/sse",
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().sessions().update(
      session.id(),
      SessionUpdateParams.builder()
          .agent(BetaManagedAgentsSessionAgentUpdate.builder()
              .addTool(BetaManagedAgentsAgentToolset20260401Params.builder().build())
              .addTool(BetaManagedAgentsMcpToolsetParams.builder()
                  .mcpServerName("linear")
                  .build())
              .addMcpServer(BetaManagedAgentsUrlMcpServerParams.builder()
                  .name("linear")
                  .url("https://mcp.linear.app/sse")
                  .build())
              .build())
          .build()
  );
  ```

  ```php PHP
  $client->beta->sessions->update(
      $session->id,
      agent: BetaManagedAgentsSessionAgentUpdate::with(
          tools: [
              BetaManagedAgentsAgentToolset20260401Params::with(),
              BetaManagedAgentsMCPToolsetParams::with(mcpServerName: 'linear'),
          ],
          mcpServers: [
              BetaManagedAgentsURLMCPServerParams::with(
                  name: 'linear',
                  url: 'https://mcp.linear.app/sse',
              ),
          ],
      ),
  );
  ```

  ```ruby Ruby
  client.beta.sessions.update(
    session.id,
    agent: {
      tools: [
        {type: :agent_toolset_20260401},
        {type: :mcp_toolset, mcp_server_name: "linear"}
      ],
      mcp_servers: [
        {type: :url, name: "linear", url: "https://mcp.linear.app/sse"}
      ]
    }
  )
  ```
</CodeGroup>

## 更新会话预算

[使用预算创建的](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#set-a-session-budget)会话接受两种预算更新：使用新的 `max_list_cost` (java, php: `maxListCost`; csharp, go: `MaxListCost`) 替换上限，以及通过传递 `"budget": null` (python: `budget=None`; typescript: `budget: null`; ruby: `budget: nil`; go: `Budget: param.NullStruct[anthropic.BetaManagedAgentsBudgetLimitParam]()`; java: `.budget(Optional.empty())`; csharp: `Budget = null`; php: `requestOptions: ['extraBodyParams' => ['budget' => null]]`; cli: `--budget null`) 移除上限。这两种方式都会自动恢复因会话达到上限而暂停的工作。替换后的上限可以高于或低于当前上限，但必须严格大于会话已消耗的标价成本。移除是单向的：只有当前拥有预算的会话才接受非 null 的 `budget` (csharp, go: `Budget`)。因此，您无法重新添加已移除的预算，也无法为创建时没有预算的会话添加预算。有关请求示例、错误行为以及哪些内容计入标价成本，请参阅[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#resume-a-session-at-its-budget)。

## 检索会话

<CodeGroup>
  ```bash cURL
  curl -fsSL "https://api.anthropic.com/v1/sessions/$SESSION_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:sessions retrieve --session-id "$SESSION_ID"
  ```

  ```python Python
  retrieved = client.beta.sessions.retrieve(session.id)
  print(f"Status: {retrieved.status}")
  ```

  ```typescript TypeScript
  const retrieved = await client.beta.sessions.retrieve(session.id);
  console.log(`Status: ${retrieved.status}`);
  ```

  ```csharp C#
  var retrieved = await client.Beta.Sessions.Retrieve(session.ID);
  Console.WriteLine($"Status: {retrieved.Status.Raw()}");
  ```

  ```go Go
  retrieved, err := client.Beta.Sessions.Get(ctx, session.ID, anthropic.BetaSessionGetParams{})
  if err != nil {
  	panic(err)
  }
  fmt.Printf("Status: %s\n", retrieved.Status)
  ```

  ```java Java
  var retrieved = client.beta().sessions().retrieve(session.id());
  IO.println("Status: " + retrieved.status());
  ```

  ```php PHP
  $retrieved = $client->beta->sessions->retrieve($session->id);
  echo "Status: {$retrieved->status}\n";
  ```

  ```ruby Ruby
  retrieved = client.beta.sessions.retrieve(session.id)
  puts "Status: #{retrieved.status}"
  ```
</CodeGroup>

## 列出会话

`GET /v1/sessions` 的结果是分页的。使用 `limit` 查询参数控制页面大小。每个响应都包含一个 `next_page` 游标；在下一次请求中将其作为 `page` 参数传递即可获取下一页。当没有更多结果时，`next_page` 为 `null`。

若要返回上一页，请将 `prev_page` 作为 `page` 参数传递。当您位于第一页时，`prev_page` 为 `null`。

`page` 游标是不透明的，并编码了生成它的请求的 `order`。`order` 查询参数设置结果的排序方向，按创建时间 `asc` 或 `desc`；默认为 `desc`（最新的在前）。以不同的 `order` 重用游标会返回 400 错误，更改 `created_at` 过滤器使其排除游标所在位置也会如此。其他查询参数（包括其余过滤器和 `limit`）可以在分页请求之间更改。有关各列表端点共用的分页字段，请参阅[分页](https://platform.claude.com/docs/zh-CN/api/overview#pagination)。

<CodeGroup>
  ```bash cURL
  curl -sS --fail-with-body \
    "https://api.anthropic.com/v1/sessions?agent_id=$AGENT_ID&limit=1" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  # 将 next_page 作为 page 查询参数传回以获取下一页，
  # 或传回 prev_page 以返回上一页。
  ```

  ```bash CLI
  # --format raw 返回一个包含 prev_page 和 next_page 游标的
  # 分页封装；默认输出会自动分页且仅输出会话。
  ant beta:sessions list --agent-id "$AGENT_ID" --limit 1 --format raw
  # 将 next_page 作为 --page 传回以获取下一页，或传回 prev_page 以返回上一页。
  ```

  ```python Python
  # 将 `limit` 设得较低，使结果跨越多页。
  first_page = client.beta.sessions.list(limit=1, agent_id=agent.id)
  # 第一页的 `prev_page` 为 None；最后一页的 `next_page` 为 None。
  print(f"prev_page: {first_page.prev_page}")
  print(f"next_page: {first_page.next_page}")

  # 将 `next_page` 作为 `page` 传回以获取下一页。
  second_page = client.beta.sessions.list(
      limit=1, agent_id=agent.id, page=first_page.next_page
  )
  for listed_session in second_page.data:
      print(f"{listed_session.id}: {listed_session.status}")

  # 将 `prev_page` 作为 `page` 传回以返回上一页。
  previous_page = client.beta.sessions.list(
      limit=1, agent_id=agent.id, page=second_page.prev_page
  )
  for listed_session in previous_page.data:
      print(f"{listed_session.id}: {listed_session.status}")
  # 对于仅向前的迭代，页面对象也可以直接迭代。
  ```

  ```typescript TypeScript
  const firstPage = await client.beta.sessions.list({ limit: 1, agent_id: agent.id });
  // 第一页的 prev_page 为 null；当存在更多会话时会设置 next_page。
  console.log(`prev_page: ${firstPage.prev_page}`);
  console.log(`next_page: ${firstPage.next_page}`);

  // 将 next_page 作为 `page` 游标传入以获取第二页。
  const secondPage = await client.beta.sessions.list({
    limit: 1,
    agent_id: agent.id,
    page: firstPage.next_page
  });
  for (const listedSession of secondPage.data) {
    console.log(`Page 2 has ${listedSession.id}: ${listedSession.status}`);
  }

  // 传入第二页的 prev_page 游标以回退到第一页。
  const previousPage = await client.beta.sessions.list({
    limit: 1,
    agent_id: agent.id,
    page: secondPage.prev_page
  });
  for (const listedSession of previousPage.data) {
    console.log(`Back on page 1: ${listedSession.id} is ${listedSession.status}`);
  }
  // 若只需向前迭代，page 对象本身也可直接迭代。
  ```

  ```csharp C#
  // `List` 返回的 SessionListPage 公开了条目，但不公开
  // 分页游标。要读取 `prev_page` / `next_page`，请改为将原始
  // 响应反序列化为 SessionListPageResponse。
  using var page1Response = await client.Beta.Sessions.WithRawResponse.List(
      new SessionListParams { Limit = 1, AgentID = agent.ID }
  );
  var page1 = await page1Response.Deserialize<SessionListPageResponse>();
  Console.WriteLine($"prev_page: {page1.PrevPage ?? "null"}");
  Console.WriteLine($"next_page: {page1.NextPage ?? "null"}");

  // 前进：将第 1 页的 `next_page` 作为 `page` 游标传入。
  using var page2Response = await client.Beta.Sessions.WithRawResponse.List(
      new SessionListParams { Limit = 1, AgentID = agent.ID, Page = page1.NextPage }
  );
  var page2 = await page2Response.Deserialize<SessionListPageResponse>();
  foreach (var listedSession in page2.Data ?? [])
  {
      Console.WriteLine($"Page 2: {listedSession.ID}: {listedSession.Status.Raw()}");
  }

  // 后退：将第 2 页的 `prev_page` 作为同一个 `page` 游标传入。
  using var previousPageResponse = await client.Beta.Sessions.WithRawResponse.List(
      new SessionListParams { Limit = 1, AgentID = agent.ID, Page = page2.PrevPage }
  );
  var previousPage = await previousPageResponse.Deserialize<SessionListPageResponse>();
  foreach (var listedSession in previousPage.Data ?? [])
  {
      Console.WriteLine($"Back to page 1: {listedSession.ID}: {listedSession.Status.Raw()}");
  }
  // 对于仅向前的迭代，(await client.Beta.Sessions.List(...)).Paginate() 会返回一个自动跟随 next_page 的 IAsyncEnumerable。
  ```

  ```go Go
  // 第 1 页：prev_page 为空，因为第一页之前没有内容。
  firstPage, err := client.Beta.Sessions.List(ctx, anthropic.BetaSessionListParams{
  	AgentID: anthropic.String(agent.ID),
  	Limit:   anthropic.Int(1),
  })
  if err != nil {
  	panic(err)
  }
  fmt.Printf("Page 1 prev_page: %q\n", firstPage.PrevPage)
  fmt.Printf("Page 1 next_page: %q\n", firstPage.NextPage)

  // 前进：将 next_page 作为 Page 游标传入以获取第 2 页。
  secondPage, err := client.Beta.Sessions.List(ctx, anthropic.BetaSessionListParams{
  	AgentID: anthropic.String(agent.ID),
  	Limit:   anthropic.Int(1),
  	Page:    anthropic.String(firstPage.NextPage),
  })
  if err != nil {
  	panic(err)
  }
  for _, listedSession := range secondPage.Data {
  	fmt.Printf("Page 2: %s: %s\n", listedSession.ID, listedSession.Status)
  }

  // 后退：第 2 页的 prev_page 是其前一页的游标。
  previousPage, err := client.Beta.Sessions.List(ctx, anthropic.BetaSessionListParams{
  	AgentID: anthropic.String(agent.ID),
  	Limit:   anthropic.Int(1),
  	Page:    anthropic.String(secondPage.PrevPage),
  })
  if err != nil {
  	panic(err)
  }
  for _, listedSession := range previousPage.Data {
  	fmt.Printf("Back to page 1: %s: %s\n", listedSession.ID, listedSession.Status)
  }
  // 如只需向前迭代，请使用 ListAutoPaging 自动跟随 next_page。
  ```

  ```java Java
  var params = SessionListParams.builder()
      .agentId(agent.id())
      .limit(1)
      .build();
  var firstPage = client.beta().sessions().list(params);
  for (var listedSession : firstPage.data()) {
      IO.println(listedSession.id() + ": " + listedSession.status());
  }
  // 在第一页上 prev_page 是空的 Optional；next_page 指向第 2 页。
  IO.println("prev_page: " + firstPage.response().prevPage());
  IO.println("next_page: " + firstPage.response().nextPage());

  // 通过将 next_page 作为页面游标传入来前进。
  var nextCursor = firstPage.response().nextPage().orElseThrow();
  var secondPage = client.beta().sessions().list(params.toBuilder()
      .page(nextCursor)
      .build());

  // 通过将 prev_page 作为同一页面游标传入来后退。
  var prevCursor = secondPage.response().prevPage().orElseThrow();
  var previousPage = client.beta().sessions().list(params.toBuilder()
      .page(prevCursor)
      .build());
  // 回到第一页，因此 prev_page 再次为空。
  IO.println("prev_page: " + previousPage.response().prevPage());
  // 对于仅向前的迭代，page.autoPager() 返回一个自动跟随 next_page 的 Iterable。
  ```

  ```php PHP
  // 第 1 页：prevPage 为 null，因为第一页之前没有任何内容。
  $firstPage = $client->beta->sessions->list(agentID: $agent->id, limit: 1);
  echo 'Page 1 prev_page: ' . ($firstPage->prevPage ?? 'null') . "\n";
  echo 'Page 1 next_page: ' . ($firstPage->nextPage ?? 'null') . "\n";

  // 前进：将 nextPage 作为 `page` 游标传回以获取第 2 页。
  $secondPage = $client->beta->sessions->list(
      agentID: $agent->id,
      limit: 1,
      page: $firstPage->nextPage,
  );
  foreach ($secondPage->getItems() as $listedSession) {
      echo "Page 2: {$listedSession->id}: {$listedSession->status}\n";
  }

  // 后退：第 2 页的 prevPage 是其前一页的游标。
  $previousPage = $client->beta->sessions->list(
      agentID: $agent->id,
      limit: 1,
      page: $secondPage->prevPage,
  );
  foreach ($previousPage->getItems() as $listedSession) {
      echo "Back to page 1: {$listedSession->id}: {$listedSession->status}\n";
  }
  // 若只需向前迭代，$page->pagingEachItem() 会逐页产出每个会话。
  ```

  ```ruby Ruby
  first_page = client.beta.sessions.list(agent_id: agent.id, limit: 1)
  first_page.data.each do |listed_session|
    puts "#{listed_session.id}: #{listed_session.status}"
  end

  # 第一页上 `prev_page` 为 nil。下一页游标以
  # `next_page_`（带尾部下划线）的形式暴露，因为普通的 `next_page` 是
  # 为您获取下一页对象的辅助方法。
  puts "prev_page: #{first_page.prev_page.inspect}"
  puts "next_page: #{first_page.next_page_.inspect}"

  # 将任一游标作为 `page` 传回，即可在列表中双向移动。
  second_page = client.beta.sessions.list(
    agent_id: agent.id,
    limit: 1,
    page: first_page.next_page_
  )
  back_to_first = client.beta.sessions.list(
    agent_id: agent.id,
    limit: 1,
    page: second_page.prev_page
  )
  back_to_first.data.each do |listed_session|
    puts "#{listed_session.id}: #{listed_session.status}"
  end
  # 若只需向前迭代，page.auto_paging_each 会自动跟随 next_page。
  ```
</CodeGroup>

## 归档会话

归档会话可以阻止向其发送新事件，同时保留其历史记录。处于 `running` 状态的会话无法归档；要归档此类会话，请单独发送一个 [`user.interrupt` 事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#interrupt-the-agent)，并等待会话变为 `idle`。如果有[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#while-a-run-is-open)处于打开状态，即使会话处于 `idle` 状态，归档也可能返回 400 错误，或者可能成功并结束该运行。在归档之前，请发送一条 `user.message` [要求智能体停止其运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#interrupt-a-session-with-runs-open)，或等待每个运行结束。然后在会话变为 `idle` 后将其归档。

<CodeGroup>
  ```bash cURL
  curl -fsSL -X POST "https://api.anthropic.com/v1/sessions/$SESSION_ID/archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:sessions archive --session-id "$SESSION_ID"
  ```

  ```python Python
  client.beta.sessions.archive(session.id)
  ```

  ```typescript TypeScript
  await client.beta.sessions.archive(session.id);
  ```

  ```csharp C#
  await client.Beta.Sessions.Archive(session.ID);
  ```

  ```go Go
  _, err = client.Beta.Sessions.Archive(ctx, session.ID, anthropic.BetaSessionArchiveParams{})
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().sessions().archive(session.id());
  ```

  ```php PHP
  $client->beta->sessions->archive($session->id);
  ```

  ```ruby Ruby
  client.beta.sessions.archive(session.id)
  ```
</CodeGroup>

## 删除会话

删除会话是永久性的。它会删除会话的记录、事件以及关联的沙箱。处于 `running` 状态的会话无法删除；要删除此类会话，请单独发送一个 [`user.interrupt` 事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#interrupt-the-agent)，并等待会话变为 `idle`。如果有[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#while-a-run-is-open)处于打开状态，即使会话处于 `idle`，删除也可能返回 400 错误，或者可能成功。删除成功后，不会有 `workflow_run` 事件报告该会话运行的结束。在删除之前，请发送一条 `user.message` [要求智能体停止其运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#interrupt-a-session-with-runs-open)，或等待每个运行结束。然后在会话处于 `idle` 后将其删除。

记忆存储、保管库、技能、环境和智能体是独立的资源，不受会话删除的影响。您通过 Files API 上传的文件同样不受影响，但会话自身生成的文件的作用域限定于该会话，会随其文件系统一起被永久删除。在删除会话之前，请下载您需要保留的所有内容。在最后一轮结束时写入的输出文件，可能需要在会话进入空闲状态后几秒钟才会出现在[会话的文件列表](https://platform.claude.com/docs/zh-CN/managed-agents/files#listing-and-downloading-session-files)中，因此请先确认您期望的文件已列出。

<CodeGroup>
  ```bash cURL
  curl -fsSL -X DELETE "https://api.anthropic.com/v1/sessions/$SESSION_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:sessions delete --session-id "$SESSION_ID"
  ```

  ```python Python
  client.beta.sessions.delete(session.id)
  ```

  ```typescript TypeScript
  await client.beta.sessions.delete(session.id);
  ```

  ```csharp C#
  await client.Beta.Sessions.Delete(session.ID);
  ```

  ```go Go
  _, err = client.Beta.Sessions.Delete(ctx, session.ID, anthropic.BetaSessionDeleteParams{})
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().sessions().delete(session.id());
  ```

  ```php PHP
  $client->beta->sessions->delete($session->id);
  ```

  ```ruby Ruby
  client.beta.sessions.delete(session.id)
  ```
</CodeGroup>
