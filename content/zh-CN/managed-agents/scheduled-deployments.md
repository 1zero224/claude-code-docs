---
title: 计划部署
url: https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments
description: 使用 Claude API 创建和管理部署：按周期性 cron 计划运行智能体并查看其运行历史。
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

**scheduled deployment**（计划部署）允许[智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)自主启动[会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)，从而使智能体能够按可预测的节奏完成任务。您可以使用 Deployments API（Claude API 的一部分）来创建和管理部署。

有关发布背景以及团队按计划运行的任务示例，请参阅博客上的 [Claude Managed Agents 中的计划部署和保管库](https://claude.com/blog/whats-new-in-claude-managed-agents)。

## 创建计划部署

创建部署时，除了 `schedule` (csharp, go: `Schedule`) 之外，您还需要传入执行所需的[会话配置](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)。

* 部署需要[智能体配置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)和[环境配置](https://platform.claude.com/docs/zh-CN/managed-agents/environments)，并可选择接受[文件](https://platform.claude.com/docs/zh-CN/managed-agents/files)、[GitHub](https://platform.claude.com/docs/zh-CN/managed-agents/github)、[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)和[保管库](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)。以[自托管环境](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)为目标的部署可以附加记忆存储；`file` 和 `github_repository` 资源需要云环境。Claude Console 的部署表单目前不为自托管环境提供记忆存储选项；请改为通过 API 或 SDK 附加它们。
* 部署还需要至少一个初始事件，即 `user.message` 或 `user.define_outcome`，用于启动每个会话的工作。在用于 `ant apply` 的部署文件中，frontmatter 下方的文本会成为该 `user.message`。
* 在 `schedule` (csharp, go: `Schedule`) 中，您需要定义 cron `expression` (csharp, go: `Expression`) 和 `timezone` (csharp, go: `Timezone`)。支持的最大粒度为分钟级。

<Note>
  在 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上，每次运行都会以您 AWS 账户中的 IAM 角色身份创建其会话。在创建部署之前，请将智能体上的 `execution_identity` 设置为 `{"type": "aws_role", "role_arn": "arn:aws:iam::123456789012:role/example-role"}`。为没有此字段的智能体创建部署将返回 400 错误。

  * **信任策略：** 允许 `aws-external-anthropic.amazonaws.com` 服务主体调用 `sts:AssumeRole` 和 `sts:SetSourceIdentity`。
  * **角色权限：** 允许 `aws-external-anthropic:CreateSession`、`sts:GetWebIdentityToken` 和 `sts:TagGetWebIdentityToken`。
  * **调用方权限：** 在智能体上设置该角色的 IAM 主体需要对该角色拥有 `iam:PassRole` 权限。该角色必须位于同一 AWS 账户中。没有任何[托管策略](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#managed-policies)授予 `iam:PassRole`。
</Note>

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/deployments?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Weekly compliance scan",
      "agent": "'"$AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'",
      "initial_events": [
        {"type": "user.message", "content": [{"type": "text", "text": "Run the weekly compliance scan."}]}
      ],
      "schedule": {
        "type": "cron",
        "expression": "0 20 * * 5",
        "timezone": "America/New_York"
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply deployment.md
    ```

    <File filename="deployment.md">
      ```markdown
      ---
      name: Weekly compliance scan
      agent: agent_011CYm1BLqPXpQRk5khsSXrs
      environment_id: env_01595EKxaaTTGwwY3kyXdtbs
      schedule:
        type: cron
        expression: "0 20 * * 5"
        timezone: America/New_York
      ---

      Run the weekly compliance scan.
      ```
    </File>

    [`ant apply`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/apply) 会打印新部署的 ID 并将其记录在 `claude-lock.json` 中。要查看部署对象，请运行 `ant beta:deployments retrieve`。
  </CodeGroupItem>

  ```python Python
  deployment = client.beta.deployments.create(
      name="Weekly compliance scan",
      agent=agent.id,
      environment_id=environment.id,
      initial_events=[
          {
              "type": "user.message",
              "content": [{"type": "text", "text": "Run the weekly compliance scan."}],
          },
      ],
      schedule={
          "type": "cron",
          "expression": "0 20 * * 5",
          "timezone": "America/New_York",
      },
  )
  ```

  ```typescript TypeScript
  const deployment = await client.beta.deployments.create({
    name: "Weekly compliance scan",
    agent: agent.id,
    environment_id: environment.id,
    initial_events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "Run the weekly compliance scan." }]
      }
    ],
    schedule: {
      type: "cron",
      expression: "0 20 * * 5",
      timezone: "America/New_York"
    }
  });
  ```

  ```csharp C#
  var deployment = await client.Beta.Deployments.Create(new()
  {
      Name = "Weekly compliance scan",
      Agent = agent.ID,
      EnvironmentID = environment.ID,
      InitialEvents =
      [
          new BetaManagedAgentsUserMessageEventParams
          {
              Content = [new BetaManagedAgentsTextBlock { Text = "Run the weekly compliance scan." }],
          },
      ],
      Schedule = new BetaManagedAgentsScheduleParams
      {
          Expression = "0 20 * * 5",
          Timezone = "America/New_York",
      },
  });
  ```

  ```go Go
  deployment, err := client.Beta.Deployments.New(ctx, anthropic.BetaDeploymentNewParams{
  	Name:          "Weekly compliance scan",
  	Agent:         anthropic.BetaDeploymentNewParamsAgentUnion{OfString: anthropic.String(agent.ID)},
  	EnvironmentID: environment.ID,
  	InitialEvents: []anthropic.BetaManagedAgentsDeploymentInitialEventParamsUnion{{
  		OfUserMessage: &anthropic.BetaManagedAgentsUserMessageEventParams{
  			Content: []anthropic.BetaManagedAgentsUserMessageEventParamsContentUnion{{
  				OfText: &anthropic.BetaManagedAgentsTextBlockParam{
  					Text: "Run the weekly compliance scan.",
  				},
  			}},
  		},
  	}},
  	Schedule: anthropic.BetaManagedAgentsScheduleParams{
  		Expression: "0 20 * * 5",
  		Timezone:   "America/New_York",
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var deployment = client.beta().deployments().create(
      DeploymentCreateParams.builder()
          .name("Weekly compliance scan")
          .agent(agent.id())
          .environmentId(environment.id())
          .addInitialEvent(
              BetaManagedAgentsUserMessageEventParams.builder()
                  .addTextContent("Run the weekly compliance scan.")
                  .build()
          )
          .schedule(
              BetaManagedAgentsScheduleParams.builder()
                  .expression("0 20 * * 5")
                  .timezone("America/New_York")
                  .build()
          )
          .build()
  );
  ```

  ```php PHP
  $deployment = $client->beta->deployments->create(
      name: 'Weekly compliance scan',
      agent: $agent->id,
      environmentID: $environment->id,
      initialEvents: [
          [
              'type' => 'user.message',
              'content' => [['type' => 'text', 'text' => 'Run the weekly compliance scan.']],
          ],
      ],
      schedule: [
          'type' => 'cron',
          'expression' => '0 20 * * 5',
          'timezone' => 'America/New_York',
      ],
  );
  ```

  ```ruby Ruby
  deployment = client.beta.deployments.create(
    name: "Weekly compliance scan",
    agent: agent.id,
    environment_id: environment.id,
    initial_events: [
      {
        type: "user.message",
        content: [{type: "text", text: "Run the weekly compliance scan."}]
      }
    ],
    schedule: {
      type: "cron",
      expression: "0 20 * * 5",
      timezone: "America/New_York"
    }
  )
  ```
</CodeGroup>

响应包含一个部署对象，其中 `schedule.upcoming_runs_at` 已填充接下来的触发时间，以便您确认计划设置正确。

```json
{
  "id": "depl_01xyz",
  "status": "active",
  "paused_reason": null,
  "schedule": {
    "type": "cron",
    "expression": "0 20 * * 5",
    "timezone": "America/New_York",
    "last_run_at": null,
    "upcoming_runs_at": [
      "2026-05-09T00:00:00Z",
      "2026-05-16T00:00:00Z",
      "2026-05-23T00:00:00Z"
    ]
  }
}
```

即将运行的时间戳反映了所配置的确切计划。但是，为了分散负载，实际执行会应用最多为运行间隔 15% 的 jitter（抖动），最短 5 秒，最长 9 分钟。

每个组织最多支持 **1,000 个计划部署**。如果您需要更多，请联系 Anthropic 支持团队。

有关完整参数和响应架构，请参阅[创建部署](https://platform.claude.com/docs/zh-CN/api/beta/deployments/create)参考文档。

### Cron 和时区语义

* **表达式：** 标准 POSIX cron（`minute hour day-of-month month day-of-week`）。您可以在 [Claude Console](https://platform.claude.com/workspaces/default/deployments) 中生成和验证这些 cron 表达式。
* **时区：** IANA 时区标识符（例如 `"America/Los_Angeles"`）。
* **夏令时：** Cron 计划使用字面挂钟时间匹配，因此 `America/New_York` 中的 `"0 20 * * *"` 会在当地时间晚上 8:00 触发，无论当前实行的是 EST 还是 EDT。

<Note>
  在夏令时开始（时钟拨快）当天不存在的挂钟时间（例如凌晨 2 点）不会被触发。在夏令时结束（时钟拨回）当天出现两次的挂钟时间会触发两次。如果无法接受遗漏或重复执行，请将计划安排在当地时间凌晨 1–3 点窗口之外，或使用 UTC。
</Note>

### 为每次运行设置预算

在创建或更新部署时，传入可选的 `budget` (csharp, go: `Budget`) 对象。它采用与[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)相同的结构。部署会将上限复制到其启动的每个会话上，因此预算分别限制每次运行，而不是作为跨所有运行的累计上限：上限为 `"2000"` 的部署每次运行最多可花费约 20 美元。

由部署启动的会话与任何其他设有预算的会话行为完全相同：当其自身的标价成本[达到上限](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#when-a-session-reaches-its-budget)时，它会以 `budget_reached` 暂停。更改部署的预算会应用于之后启动的运行；已在运行的会话保留其启动时的上限，您可以[通过会话本身更改](https://platform.claude.com/docs/zh-CN/managed-agents/session-operations#updating-the-session-budget)该上限。与会话预算不同，部署的预算可以通过 `"budget": null` (python: `budget=None`; typescript: `budget: null`; ruby: `budget: nil`; go: `Budget: param.NullStruct[anthropic.BetaManagedAgentsBudgetLimitParam]()`; java: `.budget(Optional.empty())`; csharp: `Budget = null`; php: `requestOptions: ['extraBodyParams' => ['budget' => null]]`; cli: `--budget null`) 移除，并在之后重新设置。

以下示例为现有部署设置预算：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/deployments/$DEPLOYMENT_ID?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "budget": {
        "type": "limit",
        "max_list_cost": {"amount": "2000", "currency": "USD"}
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply deployment.md
    ```

    <File filename="deployment.md">
      ```markdown
      ---
      name: Weekly compliance scan
      agent: agent_011CYm1BLqPXpQRk5khsSXrs
      environment_id: env_01595EKxaaTTGwwY3kyXdtbs
      schedule:
        type: cron
        expression: "0 20 * * 5"
        timezone: America/New_York
      budget:
        type: limit
        max_list_cost:
          amount: "2000"
          currency: USD
      ---

      Run the weekly compliance scan.
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  updated_deployment = client.beta.deployments.update(
      deployment.id,
      budget={
          "type": "limit",
          "max_list_cost": {"amount": "2000", "currency": "USD"},
      },
  )
  print(updated_deployment.budget.max_list_cost.amount)  # 2000
  ```

  ```typescript TypeScript
  const updatedDeployment = await client.beta.deployments.update(deployment.id, {
    budget: {
      type: "limit",
      max_list_cost: { amount: "2000", currency: "USD" }
    }
  });
  console.log(updatedDeployment.budget?.max_list_cost.amount); // 2000
  ```

  ```csharp C#
  var updatedDeployment = await client.Beta.Deployments.Update(deployment.ID, new()
  {
      Budget = new()
      {
          MaxListCost = new() { Amount = "2000", Currency = BetaCurrency.Usd },
      },
  });
  Console.WriteLine(updatedDeployment.Budget?.MaxListCost.Amount);  // 2000
  ```

  ```go Go
  updatedDeployment, err := client.Beta.Deployments.Update(ctx, deployment.ID, anthropic.BetaDeploymentUpdateParams{
  	Budget: anthropic.BetaManagedAgentsBudgetLimitParam{
  		MaxListCost: anthropic.BetaMonetaryAmountParam{
  			Amount:   "2000",
  			Currency: anthropic.BetaCurrencyUsd,
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  fmt.Println(updatedDeployment.Budget.MaxListCost.Amount) // 2000
  ```

  ```java Java
  var updatedDeployment = client.beta().deployments().update(
      deployment.id(),
      DeploymentUpdateParams.builder()
          .budget(BetaManagedAgentsBudgetLimit.builder()
              .maxListCost(BetaMonetaryAmount.builder()
                  .amount("2000")
                  .currency(BetaCurrency.USD)
                  .build())
              .build())
          .build()
  );
  IO.println(updatedDeployment.budget().orElseThrow().maxListCost().amount());  // 2000
  ```

  ```php PHP
  $updatedDeployment = $client->beta->deployments->update(
      $deployment->id,
      budget: [
          'type' => 'limit',
          'max_list_cost' => ['amount' => '2000', 'currency' => 'USD'],
      ],
  );
  echo "{$updatedDeployment->budget->maxListCost->amount}\n"; // 2000
  ```

  ```ruby Ruby
  updated_deployment = client.beta.deployments.update(
    deployment.id,
    budget: {
      type: "limit",
      max_list_cost: {amount: "2000", currency: "USD"}
    }
  )
  puts updated_deployment.budget.max_list_cost.amount # 2000
  ```
</CodeGroup>

## 部署运行

部署可能因多种原因而无法触发：例如，`environment` 资源已被归档，或会话创建受到速率限制。每次尝试都会生成一条 **deployment run**（部署运行）记录，使您能够独立于会话生命周期跟踪成功和失败情况。

成功的部署会生成活动会话，成功的部署运行包含关联的 `session_id` (java: `sessionId`; php: `sessionID`; csharp, go: `SessionID`)。要跟踪会话的生命周期，请通过[事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)或 [webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks) 跟踪会话事件。部署生命周期变更和每次计划运行的结果也会作为 webhook 事件传递，列在[支持的事件类型](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#supported-event-types)的**部署事件**和**部署运行事件**选项卡中。

按如下方式列出某个部署的所有部署运行：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/deployment_runs?beta=true&deployment_id=$DEPLOYMENT_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:deployment-runs list --deployment-id "$DEPLOYMENT_ID"
  ```

  ```python Python
  for run in client.beta.deployment_runs.list(
      deployment_id=deployment.id,
  ):
      print(run.created_at, run.session_id or run.error.type)
  ```

  ```typescript TypeScript
  for await (const run of client.beta.deploymentRuns.list({
    deployment_id: deployment.id
  })) {
    console.log(run.created_at, run.session_id ?? run.error?.type);
  }
  ```

  ```csharp C#
  var runs = await client.Beta.DeploymentRuns.List(
      new() { DeploymentID = deployment.ID }
  );
  await foreach (var run in runs.Paginate())
  {
      var outcome = run.SessionID ?? run.Error!.Type.Raw();
      Console.WriteLine($"{run.CreatedAt} {outcome}");
  }
  ```

  ```go Go
  runs := client.Beta.DeploymentRuns.ListAutoPaging(ctx, anthropic.BetaDeploymentRunListParams{
  	DeploymentID: anthropic.String(deployment.ID),
  })
  for runs.Next() {
  	run := runs.Current()
  	if run.SessionID != "" {
  		fmt.Println(run.CreatedAt.Format(time.RFC3339), run.SessionID)
  	} else {
  		fmt.Println(run.CreatedAt.Format(time.RFC3339), run.Error.Type)
  	}
  }
  if err := runs.Err(); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  for (var run : client.beta().deploymentRuns().list(
          DeploymentRunListParams.builder()
              .deploymentId(deployment.id())
              .build()).autoPager()) {
      IO.println(run.createdAt() + " "
          + run.sessionId().orElseGet(() -> run.error().orElseThrow().type().asString()));
  }
  ```

  ```php PHP
  foreach ($client->beta->deploymentRuns->list(
      deploymentID: $deployment->id,
  )->pagingEachItem() as $run) {
      $outcome = $run->sessionID ?? $run->error->type;
      echo "{$run->createdAt->format(DATE_ATOM)} {$outcome}\n";
  }
  ```

  ```ruby Ruby
  client.beta.deployment_runs.list(
    deployment_id: deployment.id
  ).auto_paging_each do
    puts "#{it.created_at} #{it.session_id || it.error.type}"
  end
  ```
</CodeGroup>

您还可以筛选出带有错误的部署运行：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/deployment_runs?beta=true&deployment_id=$DEPLOYMENT_ID&has_error=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:deployment-runs list --deployment-id "$DEPLOYMENT_ID" --has-error
  ```

  ```python Python
  for run in client.beta.deployment_runs.list(
      deployment_id=deployment.id,
      has_error=True,
  ):
      print(run.created_at, run.error.type, run.error.message)
  ```

  ```typescript TypeScript
  for await (const run of client.beta.deploymentRuns.list({
    deployment_id: deployment.id,
    has_error: true
  })) {
    console.log(run.created_at, run.error?.type, run.error?.message);
  }
  ```

  ```csharp C#
  var failedRuns = await client.Beta.DeploymentRuns.List(
      new() { DeploymentID = deployment.ID, HasError = true }
  );
  await foreach (var failedRun in failedRuns.Paginate())
  {
      var error = failedRun.Error!;
      var errorType = error.Type.Raw();
      Console.WriteLine($"{failedRun.CreatedAt} {errorType} {error.Message}");
  }
  ```

  ```go Go
  failedRuns := client.Beta.DeploymentRuns.ListAutoPaging(ctx, anthropic.BetaDeploymentRunListParams{
  	DeploymentID: anthropic.String(deployment.ID),
  	HasError:     anthropic.Bool(true),
  })
  for failedRuns.Next() {
  	failedRun := failedRuns.Current()
  	fmt.Println(failedRun.CreatedAt.Format(time.RFC3339), failedRun.Error.Type, failedRun.Error.Message)
  }
  if err := failedRuns.Err(); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  for (var run : client.beta().deploymentRuns().list(
          DeploymentRunListParams.builder()
              .deploymentId(deployment.id())
              .hasError(true)
              .build()).autoPager()) {
      var error = run.error().orElseThrow();
      IO.println(run.createdAt() + " " + error.type().asString() + " " + error.message());
  }
  ```

  ```php PHP
  foreach ($client->beta->deploymentRuns->list(
      deploymentID: $deployment->id,
      hasError: true,
  )->pagingEachItem() as $run) {
      echo "{$run->createdAt->format(DATE_ATOM)} {$run->error->type} {$run->error->message}\n";
  }
  ```

  ```ruby Ruby
  client.beta.deployment_runs.list(
    deployment_id: deployment.id,
    has_error: true
  ).auto_paging_each do
    puts "#{it.created_at} #{it.error.type} #{it.error.message}"
  end
  ```
</CodeGroup>

失败的运行包含一个 `error`，其 `type` 描述了会话创建被拒绝的原因（例如 `environment_archived_error`、`agent_archived_error` 或 `session_rate_limited_error`）。有关所有筛选参数和响应架构，请参阅[列出部署运行](https://platform.claude.com/docs/zh-CN/api/beta/deployment_runs/list)参考文档。

```json
{
  "type": "deployment_run",
  "id": "drun_01abc124",
  "deployment_id": "depl_01xyz",
  "trigger_context": { "type": "schedule", "scheduled_at": "2026-05-09T00:00:00Z" },
  "session_id": null,
  "error": {
    "type": "environment_archived_error",
    "message": "environment `env_01abc` is archived"
  },
  "agent": { "type": "agent", "id": "agent_01ghi789", "version": 3 },
  "created_at": "2026-05-09T00:00:01Z"
}
```

要按 ID 检索单个运行，请调用 [`GET /v1/deployment_runs/{deployment_run_id}`](https://platform.claude.com/docs/zh-CN/api/beta/deployment_runs/retrieve)。[`deployment_run` webhook 事件](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#supported-event-types)会将运行 ID 作为其 `data.id` 携带。

## 管理部署生命周期

每次生命周期变更都会发出一个 [webhook 事件](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#supported-event-types)，因此您无需轮询即可对部署的暂停、取消暂停或归档做出响应；请参阅**部署事件**选项卡。

**暂停**会从该时间点起阻止计划触发；先前部署运行中已在运行的会话会继续运行。暂停期间仍允许通过 `run` 端点进行手动运行。暂停会将 `paused_reason` 设置为 `{"type": "manual"}`；取消暂停会将其清除。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS -X POST "https://api.anthropic.com/v1/deployments/$DEPLOYMENT_ID/pause?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:deployments pause --deployment-id "$DEPLOYMENT_ID"
  ```

  ```python Python
  client.beta.deployments.pause(deployment.id)
  ```

  ```typescript TypeScript
  await client.beta.deployments.pause(deployment.id);
  ```

  ```csharp C#
  await client.Beta.Deployments.Pause(deployment.ID);
  ```

  ```go Go
  if _, err := client.Beta.Deployments.Pause(ctx, deployment.ID, anthropic.BetaDeploymentPauseParams{}); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().deployments().pause(deployment.id());
  ```

  ```php PHP
  $client->beta->deployments->pause($deployment->id);
  ```

  ```ruby Ruby
  client.beta.deployments.pause(deployment.id)
  ```
</CodeGroup>

**取消暂停**会从下一个计划时间点恢复计划。错过的触发不会被补执行。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS -X POST "https://api.anthropic.com/v1/deployments/$DEPLOYMENT_ID/unpause?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:deployments unpause --deployment-id "$DEPLOYMENT_ID"
  ```

  ```python Python
  client.beta.deployments.unpause(deployment.id)
  ```

  ```typescript TypeScript
  await client.beta.deployments.unpause(deployment.id);
  ```

  ```csharp C#
  await client.Beta.Deployments.Unpause(deployment.ID);
  ```

  ```go Go
  if _, err := client.Beta.Deployments.Unpause(ctx, deployment.ID, anthropic.BetaDeploymentUnpauseParams{}); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().deployments().unpause(deployment.id());
  ```

  ```php PHP
  $client->beta->deployments->unpause($deployment->id);
  ```

  ```ruby Ruby
  client.beta.deployments.unpause(deployment.id)
  ```
</CodeGroup>

与**暂停**不同，**归档**是终态操作：计划结束，且部署无法再被修改。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS -X POST "https://api.anthropic.com/v1/deployments/$DEPLOYMENT_ID/archive?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:deployments archive --deployment-id "$DEPLOYMENT_ID"
  ```

  ```python Python
  client.beta.deployments.archive(deployment.id)
  ```

  ```typescript TypeScript
  await client.beta.deployments.archive(deployment.id);
  ```

  ```csharp C#
  await client.Beta.Deployments.Archive(deployment.ID);
  ```

  ```go Go
  if _, err := client.Beta.Deployments.Archive(ctx, deployment.ID, anthropic.BetaDeploymentArchiveParams{}); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  client.beta().deployments().archive(deployment.id());
  ```

  ```php PHP
  $client->beta->deployments->archive($deployment->id);
  ```

  ```ruby Ruby
  client.beta.deployments.archive(deployment.id)
  ```
</CodeGroup>

### 失败行为

会话创建的速率限制响应会立即记录为 `session_rate_limited_error` 运行，且不会重试；计划会在下一个计划时间点再次尝试。会话内底层 API 调用的速率限制由会话本身处理。

如果部署的智能体已被归档，该部署会在同一操作中被自动归档。如果智能体已被删除，下一次计划触发会检测到智能体缺失并自动归档该部署。在这两种情况下都不会记录部署运行。如果智能体引用的子智能体已被归档，下一次触发会记录一个带有 `error.type: "agent_archived_error"` 的失败运行，并且部署会被自动暂停，以便您更新智能体后恢复。其他不可恢复的会话创建错误（例如已归档的环境或保管库）的行为相同：触发会记录一个失败运行，并且部署会被自动暂停。部署的 `paused_reason.error.type` 与失败运行的 `error.type` 一致。

## 触发手动运行

要在计划之外运行部署，请调用 [`run` 端点](https://platform.claude.com/docs/zh-CN/api/beta/deployments/run)。这会立即创建一个会话，并写入一条带有 `trigger_context.type: "manual"` 的部署运行。这使您可以在正式启用计划之前测试部署。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS -X POST "https://api.anthropic.com/v1/deployments/$DEPLOYMENT_ID/run?beta=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:deployments run --deployment-id "$DEPLOYMENT_ID"
  ```

  ```python Python
  run = client.beta.deployments.run(deployment.id)
  ```

  ```typescript TypeScript
  const run = await client.beta.deployments.run(deployment.id);
  ```

  ```csharp C#
  var manualRun = await client.Beta.Deployments.Run(deployment.ID);
  ```

  ```go Go
  manualRun, err := client.Beta.Deployments.Run(ctx, deployment.ID, anthropic.BetaDeploymentRunParams{})
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var run = client.beta().deployments().run(deployment.id());
  ```

  ```php PHP
  $run = $client->beta->deployments->run($deployment->id);
  ```

  ```ruby Ruby
  run = client.beta.deployments.run(deployment.id)
  ```
</CodeGroup>

## 可用性

计划部署在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                                                                                                                                                                                  | 模型                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

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
