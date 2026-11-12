---
title: 用量和成本分析 API
url: https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api
description: 按模型、工作区、API 密钥、操作者等维度查询您的 Claude Console 或 Claude Enterprise 组织的令牌用量和成本。
---

<Tip>
  **Admin API 不适用于 claude.ai Team 计划中的组织，也不适用于个人 Claude Console 账户。** 如果您的公司还拥有 Claude Console 组织，您可以通过该组织使用此 API；请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。个人 Console 账户必须先在 Claude Console 中的[组织](https://platform.claude.com/settings/organization)页面（`platform.claude.com/settings/organization`）上设置组织。
</Tip>

用量和成本分析 API 报告您组织的历史令牌用量和成本，并按您选择的维度进行细分，例如模型、工作区、API 密钥，或每个请求背后的用户或服务账户。对于 Claude Console 组织，这些数据与 Claude Console 的[用量](https://platform.claude.com/usage)和[费用](https://platform.claude.com/cost)页面所显示的内容类似。

该 API 有两个端点：

* `GET /v1/organizations/analytics/usage` 报告令牌数和请求数。
* `GET /v1/organizations/analytics/cost` 报告以美元计的成本。

用量和成本分析 API 目前处于 beta 阶段。请在每个请求中发送 `anthropic-beta: analytics-usage-cost-2026-08-24` 标头和 [`anthropic-version`](https://platform.claude.com/docs/zh-CN/api/versioning) 标头。SDK 和 `ant` CLI 会为您发送这两个标头，并在您读取结果时获取剩余页面，因此 `limit` 仅设置页面大小。

使用此 API 可以：

* **归因支出：** 按工作区、API 密钥、模型、服务层级或发出每个请求的身份细分成本和令牌用量。
* **找出最大消耗者：** 按令牌数、请求数或成本对结果排序，以回答诸如"本月哪些 API 密钥支出最多？"之类的问题。
* **构建时间序列：** 将结果拆分为每日、每小时或每分钟的时间桶，用于仪表板和财务报告。
* **一次集成：** 相同的端点、参数和响应字段同时适用于 Claude Console 和 Claude Enterprise 组织。

<Check>
  **身份验证**

  Claude Console 组织使用 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-console-organization)或其他 [Admin API 凭据](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)，例如具有 `org:admin` 范围的 OAuth 令牌。请在 `authorization: Bearer` 标头（而不是 `x-api-key` 标头）中发送 OAuth 令牌；请参阅[获取 OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。

  Claude Enterprise 组织使用具有 `read:analytics` 范围的密钥，该密钥由主要所有者在组织启用 Analytics API 后于 claude.ai 中创建；请参阅[获取 Claude Enterprise Analytics API 的访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api#get-access-to-the-claude-enterprise-analytics-api)。

  本页中的示例会从 `ANTHROPIC_API_KEY` 环境变量读取密钥，因此请在运行示例之前将该变量设置为您的密钥。

  如果您使用 Admin API 密钥调用这些端点，请为收集这些数据的服务单独分配一个密钥，将其存储在密钥管理器中，并定期轮换；在创建密钥时设置[密钥过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)可以强制执行轮换。在可能的情况下，请改用具有 [`org:admin` 范围](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#oauth-scopes)的 [Workload Identity Federation](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)（工作负载身份联合）规则，这样服务会获得短期令牌，也就不存在可能泄露的长期密钥。
</Check>

<Note>
  **Claude Platform on AWS：** 两个端点均可用。请求使用 Claude Platform on AWS 的基础 URL 和身份验证来代替 Admin API 密钥（请参阅[发出请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#making-requests)），并由 AWS IAM 通过 [`ListAccountUsage` 和 `ListAccountCost` 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#usage-and-cost)进行授权。每个请求仅涵盖您所调用端点所在 AWS 区域中的工作区，因此请调用每个区域的端点以涵盖整个组织。`user_profile_id`、`claude_tag_category`、`claude_tag_user_id` 以及 Claude Managed Agents 维度（`agent_id`、`environment_id`、`deployment_id` 和 `session_id`）在每一行中均为 `null`。
</Note>

<Note>
  **Claude Models on Google Cloud (Anthropic managed)：** 两个端点均可用。请通过[组织级基础 URL](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#making-requests) 使用 Google Cloud 凭据（而非 Admin API 密钥）调用这些端点。`/usage` 端点需要 `usageAnalytics.list` [IAM 权限](https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions)，`/cost` 端点需要 `costAnalytics.list`。
</Note>

## 快速入门

获取 9 月第一周的每日令牌用量，每个模型每天一行：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/usage?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-09-08T00:00:00Z&\
  time_bucket_width=1d&\
  group_by[]=model&\
  limit=10" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:usage list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-09-08T00:00:00Z \
    --time-bucket-width 1d \
    --group-by model \
    --limit 10
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.usage.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-09-08T00:00:00Z",
      time_bucket_width="1d",
      group_by=["model"],
      limit=10,
  )

  for row in rows:
      print(f"{row.starting_at} {row.group.model}: {row.usage.total_tokens}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.usage.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-09-08T00:00:00Z",
    time_bucket_width: "1d",
    group_by: ["model"],
    limit: 10
  });

  for await (const row of rows) {
    console.log(`${row.starting_at} ${row.group.model}: ${row.usage.total_tokens}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics;
  using Anthropic.Models.Beta.Organization.Analytics.Usage;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Usage.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-09-08T00:00:00Z"),
          TimeBucketWidth = BetaAnalyticsTimeBucketWidth.Day,
          GroupBy = [BetaAnalyticsUsageGroupBy.Model],
          Limit = 10,
      }
  );

  await foreach (var row in page.Paginate())
  {
      Console.WriteLine($"{row.StartingAt} {row.Group.Model}: {row.Usage.TotalTokens}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Usage.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsUsageListParams{
  	StartingAt:      time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:        anthropic.Time(time.Date(2026, time.September, 8, 0, 0, 0, 0, time.UTC)),
  	TimeBucketWidth: anthropic.BetaAnalyticsTimeBucketWidthDay,
  	GroupBy:         []anthropic.BetaAnalyticsUsageGroupBy{anthropic.BetaAnalyticsUsageGroupByModel},
  	Limit:           anthropic.Int(10),
  })

  for rows.Next() {
  	row := rows.Current()
  	fmt.Printf("%s %s: %d\n", row.StartingAt.Format(time.RFC3339), row.Group.Model, row.Usage.TotalTokens)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.BetaAnalyticsTimeBucketWidth;
  import com.anthropic.models.beta.organization.analytics.usage.BetaAnalyticsUsageGroupBy;
  import com.anthropic.models.beta.organization.analytics.usage.UsageListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UsageListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-09-08T00:00:00Z"))
          .timeBucketWidth(BetaAnalyticsTimeBucketWidth.DAY)
          .addGroupBy(BetaAnalyticsUsageGroupBy.MODEL)
          .limit(10)
          .build();
      var rows = client.beta().organization().analytics().usage().list(params);

      for (var row : rows.autoPager()) {
          IO.println(row.startingAt() + " " + row.group().model().orElse(null) + ": " + row.usage().totalTokens());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->usage->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-09-08T00:00:00Z'),
      timeBucketWidth: '1d',
      groupBy: ['model'],
      limit: 10,
  );

  foreach ($rows->pagingEachItem() as $row) {
      $startingAt = $row->startingAt->format(\DateTimeInterface::ATOM);
      echo "{$startingAt} {$row->group->model}: {$row->usage->totalTokens}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.usage.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 9, 8),
    time_bucket_width: :"1d",
    group_by: [:model],
    limit: 10
  )

  page.auto_paging_each do |row|
    puts "#{row.starting_at} #{row.group.model}: #{row.usage.total_tokens}"
  end
  ```
</CodeGroup>

响应中每个有用量的模型每天一行，`total_tokens` 最大的排在最前。此示例显示单行：

```json
{
  "data": [
    {
      "starting_at": "2026-09-03T00:00:00Z",
      "ending_at": "2026-09-04T00:00:00Z",
      "group": {
        "product": null,
        "model": "claude-sonnet-5",
        "context_window": null,
        "inference_geo": null,
        "speed": null,
        "slack_channel_id": null,
        "claude_tag_category": null,
        "claude_tag_user_id": null,
        "rbac_group_id": null,
        "workspace_id": null,
        "service_tier": null,
        "user_profile_id": null,
        "agent_id": null,
        "environment_id": null,
        "deployment_id": null,
        "session_id": null,
        "actor": null,
        "actor_type": null,
        "api_key_id": null
      },
      "usage": {
        "uncached_input_tokens": 1284500,
        "cache_creation": {
          "ephemeral_1h_input_tokens": 0,
          "ephemeral_5m_input_tokens": 215300
        },
        "cache_read_input_tokens": 3200000,
        "output_tokens": 891000,
        "total_tokens": 5590800,
        "server_tool_use": {
          "web_search_requests": 12
        },
        "requests": 4210
      }
    }
  ],
  "has_more": true,
  "next_page": "page_V2mQ4dkt9HcbX3pLr...",
  "data_refreshed_at": "2026-09-18T16:02:11Z"
}
```

<Tip>
  **为集成设置 User-Agent 标头**

  如果您正在构建集成，请设置 User-Agent 标头，以帮助 Anthropic 了解使用模式：

  ```text wrap
  User-Agent: YourApp/1.0.0 (https://yourapp.com)
  ```
</Tip>

## 结果的结构

* **每一行都有三个部分：** `starting_at` 和 `ending_at` 给出该行涵盖的时间段，`group` 包含所有维度，`usage`（在成本端点上为 `cost`）包含指标。
* **未分组的维度为 null：** `group` 中的维度为 `null`，除非您在 `group_by[]` 中列出它。对于没有该维度值的行，已分组的维度也为 `null`，例如代码执行用量中的 `model`。`group_by[]` 最多接受五个维度。
* **每种组合一行：** 每一行是已分组维度的一种组合，每个有用量或成本的组合在一次查询的所有页面中恰好出现一次。如果没有 `group_by[]` 且没有 `time_bucket_width`，响应将是汇总整个范围的单行。
* **时间桶是可选的：** 将 `time_bucket_width` 设置为 `1m`、`1h` 或 `1d`，即可按 UTC 时间桶拆分每种组合；此时每行的 `starting_at` 和 `ending_at` 即为其时间桶的边界。没有用量或成本的时间桶不会产生任何行。
* **默认按排名排序：** 行按 `order_by` 指标在整个时间范围内排序，默认从大到小（`order=asc` 可反转顺序）。要构建时间序列，请设置 `time_bucket_width` 和 `order_by=time`。此时行将按时间桶依次返回，默认最新的时间桶在前（`order=asc` 会将最早的放在前面），每个时间桶内数值最大的行在前。请用零填充空的时间桶。
* **筛选器可与分组配合使用：** 大多数可用于分组的维度都有对应的方括号表示法列表筛选器，例如 `models[]=claude-opus-5-5`。重复该参数即可传递多个值，最多 100 个。同一筛选器的多个值以 OR 组合，不同筛选器以 AND 组合。筛选器和 `group_by[]` 相互独立；您可以按任何维度进行筛选，无论是否按其分组，反之亦然。

`model` 和 `actor` 等维度适用于所有组织。有些维度（例如 `api_key_id`）仅适用于 Claude Console 组织，另一些（例如 `slack_channel_id`）仅适用于 Claude Enterprise 组织。不适用于您组织的维度在每一行中都为 `null`，其筛选器不匹配任何内容；例外是 `rbac_group_id`，当您的组织无法使用团队功能时，它会返回错误。[用量](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/usage/list) 和 [成本](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/cost/list) 的 API 参考列出了所有维度、筛选器和值。

### 操作者

`actor` 维度表示请求归属于谁：用户、服务账户、联合云身份（对于通过其云平台访问 Claude 的组织，即 AWS 或 Google Cloud 主体），或者当请求背后没有上述任何身份时，即 API 密钥本身。当您按 `actor` 分组时，每行的 `actor` 是一个对象，其 `type` 表明它属于哪种类型。无法归属于任何身份的用量具有 `actor: null`。

[个人密钥和服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)以其关联的用户或服务账户身份运行，因此它们的请求会将该身份显示为 `actor`，并将密钥显示在 `api_key_id` 中。旧版工作区密钥未关联任何身份，因此该密钥同时显示为 `actor` 和 `api_key_id`。

不使用 API 密钥发出的请求（例如来自 Claude Console 或使用[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)令牌的请求）具有 `api_key_id: null`。使用工作负载身份联合令牌时，`actor` 是该令牌以其身份运行的服务账户。

无论成员以何种方式使用其 Claude Console 账户登录 Claude Code，`actor` 都是该成员。如果 Claude Code 在登录时创建了 API 密钥，则 `api_key_id` 是该密钥，它位于 [Claude Code 工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#claude-code-workspace)中。[不使用 API 密钥登录](https://code.claude.com/docs/en/authentication#sign-in-without-an-api-key)后，`api_key_id` 为 `null`。

部分 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 会话的令牌用量归属于一个由 Anthropic 管理的服务账户（`svac_...`）。对于此类用量，`api_key_id` 为 `null`。该账户不会与您创建的服务账户一起列出。请参阅[查找整个组织的会话用量](https://platform.claude.com/docs/zh-CN/managed-agents/session-observability#find-session-usage-across-your-organization)。

`api_key_ids[]` 和 `actor_ids[]` 筛选器回答的是不同的问题。`api_key_ids[]=apikey_...` 匹配该密钥认证的所有请求，无论其归属于谁。`actor_ids[]=apikey_...` 仅匹配归属于该密钥本身的请求。要在 `api_key_ids[]` 筛选器中包含没有 API 密钥的用量，请添加一个空条目：`api_key_ids[]=`。

## 用量端点

`GET /v1/organizations/analytics/usage` 按类型返回令牌计数。`order_by` 接受 `total_tokens`（默认值）、`output_tokens`、`uncached_input_tokens`、`requests` 或 `time`。

例如，要查看某个工作区和某个 API 密钥一周的用量，并按服务层级细分（每个层级一行，涵盖整周）：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/usage?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-09-08T00:00:00Z&\
  workspace_ids[]=wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ&\
  api_key_ids[]=apikey_01Rj2N8SVvo6BePZj99NhmiT&\
  group_by[]=service_tier" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:usage list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-09-08T00:00:00Z \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --api-key-id apikey_01Rj2N8SVvo6BePZj99NhmiT \
    --group-by service_tier
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.usage.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-09-08T00:00:00Z",
      workspace_ids=["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
      api_key_ids=["apikey_01Rj2N8SVvo6BePZj99NhmiT"],
      group_by=["service_tier"],
  )

  for row in rows:
      print(f"{row.group.service_tier}: {row.usage.total_tokens}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.usage.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-09-08T00:00:00Z",
    workspace_ids: ["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
    api_key_ids: ["apikey_01Rj2N8SVvo6BePZj99NhmiT"],
    group_by: ["service_tier"]
  });

  for await (const row of rows) {
    console.log(`${row.group.service_tier}: ${row.usage.total_tokens}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics.Usage;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Usage.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-09-08T00:00:00Z"),
          WorkspaceIds = ["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
          ApiKeyIds = ["apikey_01Rj2N8SVvo6BePZj99NhmiT"],
          GroupBy = [BetaAnalyticsUsageGroupBy.ServiceTier],
      }
  );

  await foreach (var row in page.Paginate())
  {
      Console.WriteLine($"{row.Group.ServiceTier?.Raw()}: {row.Usage.TotalTokens}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Usage.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsUsageListParams{
  	StartingAt:   time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:     anthropic.Time(time.Date(2026, time.September, 8, 0, 0, 0, 0, time.UTC)),
  	WorkspaceIDs: []string{"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"},
  	APIKeyIDs:    []string{"apikey_01Rj2N8SVvo6BePZj99NhmiT"},
  	GroupBy:      []anthropic.BetaAnalyticsUsageGroupBy{anthropic.BetaAnalyticsUsageGroupByServiceTier},
  })

  for rows.Next() {
  	row := rows.Current()
  	fmt.Printf("%s: %d\n", row.Group.ServiceTier, row.Usage.TotalTokens)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.usage.BetaAnalyticsUsageGroupBy;
  import com.anthropic.models.beta.organization.analytics.usage.UsageListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UsageListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-09-08T00:00:00Z"))
          .addWorkspaceId("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
          .addApiKeyId("apikey_01Rj2N8SVvo6BePZj99NhmiT")
          .addGroupBy(BetaAnalyticsUsageGroupBy.SERVICE_TIER)
          .build();
      var rows = client.beta().organization().analytics().usage().list(params);

      for (var row : rows.autoPager()) {
          IO.println(row.group().serviceTier().orElse(null) + ": " + row.usage().totalTokens());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->usage->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-09-08T00:00:00Z'),
      workspaceIDs: ['wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ'],
      apiKeyIDs: ['apikey_01Rj2N8SVvo6BePZj99NhmiT'],
      groupBy: ['service_tier'],
  );

  foreach ($rows->pagingEachItem() as $row) {
      echo "{$row->group->serviceTier}: {$row->usage->totalTokens}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.usage.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 9, 8),
    workspace_ids: ["wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"],
    api_key_ids: ["apikey_01Rj2N8SVvo6BePZj99NhmiT"],
    group_by: [:service_tier]
  )

  page.auto_paging_each do |row|
    puts "#{row.group.service_tier}: #{row.usage.total_tokens}"
  end
  ```
</CodeGroup>

要查找 API 密钥和工作区 ID，请使用[列出 API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/list)和[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)端点或 Claude Console。

要找出一个月内发出请求最多的用户、服务账户和 API 密钥：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/usage?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-10-01T00:00:00Z&\
  group_by[]=actor&\
  order_by=requests&\
  limit=25" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:usage list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-10-01T00:00:00Z \
    --group-by actor \
    --order-by requests \
    --limit 25
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.usage.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-10-01T00:00:00Z",
      group_by=["actor"],
      order_by="requests",
      limit=25,
  )

  for row in rows:
      print(f"{row.group.actor}: {row.usage.requests}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.usage.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-10-01T00:00:00Z",
    group_by: ["actor"],
    order_by: "requests",
    limit: 25
  });

  for await (const row of rows) {
    console.log(`${JSON.stringify(row.group.actor)}: ${row.usage.requests}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics.Usage;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Usage.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-10-01T00:00:00Z"),
          GroupBy = [BetaAnalyticsUsageGroupBy.Actor],
          OrderBy = BetaAnalyticsUsageOrderBy.Requests,
          Limit = 25,
      }
  );

  await foreach (var row in page.Paginate())
  {
      Console.WriteLine($"{row.Group.Actor?.Json}: {row.Usage.Requests}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Usage.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsUsageListParams{
  	StartingAt: time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:   anthropic.Time(time.Date(2026, time.October, 1, 0, 0, 0, 0, time.UTC)),
  	GroupBy:    []anthropic.BetaAnalyticsUsageGroupBy{anthropic.BetaAnalyticsUsageGroupByActor},
  	OrderBy:    anthropic.BetaAnalyticsUsageOrderByRequests,
  	Limit:      anthropic.Int(25),
  })

  for rows.Next() {
  	row := rows.Current()
  	fmt.Printf("%s: %d\n", row.Group.Actor.RawJSON(), row.Usage.Requests)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.usage.BetaAnalyticsUsageGroupBy;
  import com.anthropic.models.beta.organization.analytics.usage.BetaAnalyticsUsageOrderBy;
  import com.anthropic.models.beta.organization.analytics.usage.UsageListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UsageListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-10-01T00:00:00Z"))
          .addGroupBy(BetaAnalyticsUsageGroupBy.ACTOR)
          .orderBy(BetaAnalyticsUsageOrderBy.REQUESTS)
          .limit(25)
          .build();
      var rows = client.beta().organization().analytics().usage().list(params);

      for (var row : rows.autoPager()) {
          IO.println(row.group().actor().orElse(null) + ": " + row.usage().requests().orElse(null));
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->usage->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-10-01T00:00:00Z'),
      groupBy: ['actor'],
      orderBy: 'requests',
      limit: 25,
  );

  foreach ($rows->pagingEachItem() as $row) {
      $actor = json_encode($row->group->actor);
      echo "{$actor}: {$row->usage->requests}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.usage.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 10, 1),
    group_by: [:actor],
    order_by: :requests,
    limit: 25
  )

  page.auto_paging_each do |row|
    puts "#{row.group.actor}: #{row.usage.requests}"
  end
  ```
</CodeGroup>

## 成本端点

`GET /v1/organizations/analytics/cost` 接受与 `/usage` 端点相同的筛选器、维度和时间桶。每行的 `cost` 对象包含 `amount`（折扣后、应用任何抵扣额度之前的成本）、`list_amount`（按标价计算的成本）、`currency`（始终为 `"USD"`）以及 `requests`。金额是以美分为单位、保留六位小数的十进制字符串：`"41280.000000"` 即 412.80 美元。`order_by` 接受 `amount`（默认值）、`list_amount` 或 `time`。

有两个额外维度仅存在于此端点上，且不计入五个维度的限制：`group_by[]=cost_type` 将每一行拆分为每种成本类型各一行，`group_by[]=token_type` 将令牌成本拆分为每种令牌类型各一行。`limit` 同样不适用于这两个维度，因此一个页面可以包含超过 `limit` 的行数。例如，按模型、成本类型和令牌类型查看一个月的成本：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/cost?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-10-01T00:00:00Z&\
  group_by[]=model&\
  group_by[]=cost_type&\
  group_by[]=token_type" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:cost list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-10-01T00:00:00Z \
    --group-by model \
    --group-by cost_type \
    --group-by token_type
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.cost.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-10-01T00:00:00Z",
      group_by=["model", "cost_type", "token_type"],
  )

  for row in rows:
      print(
          f"{row.group.model} {row.group.cost_type} {row.group.token_type}: {row.cost.amount}"
      )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.cost.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-10-01T00:00:00Z",
    group_by: ["model", "cost_type", "token_type"]
  });

  for await (const row of rows) {
    console.log(
      `${row.group.model} ${row.group.cost_type} ${row.group.token_type}: ${row.cost.amount}`
    );
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics.Cost;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Cost.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-10-01T00:00:00Z"),
          GroupBy = [BetaAnalyticsCostGroupBy.Model, BetaAnalyticsCostGroupBy.CostType, BetaAnalyticsCostGroupBy.TokenType],
      }
  );

  await foreach (var row in page.Paginate())
  {
      Console.WriteLine(
          $"{row.Group.Model} {row.Group.CostType?.Raw()} {row.Group.TokenType?.Raw()}: {row.Cost.Amount}"
      );
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Cost.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsCostListParams{
  	StartingAt: time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:   anthropic.Time(time.Date(2026, time.October, 1, 0, 0, 0, 0, time.UTC)),
  	GroupBy:    []anthropic.BetaAnalyticsCostGroupBy{anthropic.BetaAnalyticsCostGroupByModel, anthropic.BetaAnalyticsCostGroupByCostType, anthropic.BetaAnalyticsCostGroupByTokenType},
  })

  for rows.Next() {
  	row := rows.Current()
  	fmt.Printf("%s %s %s: %s\n", row.Group.Model, row.Group.CostType, row.Group.TokenType, row.Cost.Amount)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.cost.BetaAnalyticsCostGroupBy;
  import com.anthropic.models.beta.organization.analytics.cost.CostListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = CostListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-10-01T00:00:00Z"))
          .addGroupBy(BetaAnalyticsCostGroupBy.MODEL)
          .addGroupBy(BetaAnalyticsCostGroupBy.COST_TYPE)
          .addGroupBy(BetaAnalyticsCostGroupBy.TOKEN_TYPE)
          .build();
      var rows = client.beta().organization().analytics().cost().list(params);

      for (var row : rows.autoPager()) {
          var group = row.group();
          IO.println(
              group.model().orElse(null) + " " + group.costType().orElse(null) + " "
                  + group.tokenType().orElse(null) + ": " + row.cost().amount()
          );
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->cost->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-10-01T00:00:00Z'),
      groupBy: ['model', 'cost_type', 'token_type'],
  );

  foreach ($rows->pagingEachItem() as $row) {
      $group = $row->group;
      echo "{$group->model} {$group->costType} {$group->tokenType}: {$row->cost->amount}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.cost.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 10, 1),
    group_by: [:model, :cost_type, :token_type]
  )

  page.auto_paging_each do |row|
    group = row.group
    puts "#{group.model} #{group.cost_type} #{group.token_type}: #{row.cost.amount}"
  end
  ```
</CodeGroup>

对于每个模型，响应包含五个 `tokens` 行（每种令牌类型一行），外加 `web_search`、`code_execution` 和 `session_usage` 各一行。其中一行如下：

```json
{
  "starting_at": "2026-09-01T00:00:00Z",
  "ending_at": "2026-10-01T00:00:00Z",
  "group": {
    "model": "claude-opus-5",
    ...,
    "cost_type": "tokens",
    "token_type": "cache_read_input_tokens"
  },
  "cost": {
    "currency": "USD",
    "amount": "41280.000000",
    "list_amount": "51600.000000",
    "requests": null
  }
}
```

`group` 对象包含所有维度，此处已省略部分内容。指定 `group_by[]=cost_type` 或 `token_type` 时，每行的 `requests` 均为 `null`。

<Warning>
  在 Priority Tier 承诺下处理的请求会以 `service_tier: "priority"` 出现在用量端点上，但对 `amount` 和 `list_amount` 贡献的令牌成本为零，因为您的承诺已涵盖这些请求；承诺费用单独计费，不包含在这些数据中。这些请求中的网络搜索按标准费率计费。超出您承诺容量的请求将按标准层级处理和计价。
</Warning>

## 日期范围和时间桶

* 时间戳采用带 UTC 偏移量的 RFC 3339 格式，例如 `2026-09-01T00:00:00Z`。`starting_at` 包含在内，`ending_at` 不包含在内。
* `starting_at` 必须在 2026 年 1 月 1 日或之后，且在过去 365 天内。单个请求最多可跨越 31 天；`ending_at` 默认为 `starting_at` 之后 31 天或当前时间，以较早者为准。
* 只要设置了 `time_bucket_width`，就必须提供 `ending_at`；使用 `time_bucket_width=1m` 时，单个请求最多可跨越 24 小时。
* 范围的两端都会向下取整到 UTC 时间桶边界，未设置 `time_bucket_width` 时则取整到整点。例如，使用 `time_bucket_width=1h` 时，`2026-09-01T10:43:00Z` 的 `starting_at` 会被视为 `2026-09-01T10:00:00Z`，`2026-09-01T15:43:00Z` 的 `ending_at` 会被视为 `2026-09-01T15:00:00Z`，因此响应覆盖 UTC 时间 10:00 至 15:00。

## 数据新鲜度

每个响应都包含一个 `data_refreshed_at` 时间戳，该时间之后的用量数据缺失或不完整。当请求范围的任何部分都尚无可用数据时，`data_refreshed_at` 为 `null`，`data` 为空。数据通常在底层用量发生后 4 小时内刷新，最长可能需要 24 小时。随着延迟事件的到达和对账的运行，数值可能会被修订，直到日历月结束后约 7 天为止。例如，10 月 1 日的数值可能会变化到 11 月 7 日左右。为了在重复调用中获得稳定的结果，请将 `ending_at` 设置为不晚于您已看到的某个 `data_refreshed_at`；如需最终总计，请仅查询至少 7 天前已结束的月份。分页查询的每一页都读取相同的数据快照，因此 `data_refreshed_at` 在各页之间完全相同。

## 从用量和成本 API 迁移

本节适用于调用旧版[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)（`/v1/organizations/usage_report/messages` 和 `/v1/organizations/cost_report`）的 Claude Console 组织。您的 Admin API 密钥、`anthropic-version` 标头、`starting_at` 和 `ending_at`、时间桶宽度值（`1m`、`1h`、`1d`）、方括号表示法、分页字段名称以及令牌和成本字段名称均保持不变。变化如下：

| 用量和成本 API                                                                                       | 用量和成本分析 API                                                                                                                                                                                                                                                      |
| ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 无 beta 标头                                                                                       | 每个请求都需要 `anthropic-beta: analytics-usage-cost-2026-08-24`                                                                                                                                                                                                        |
| 数据通常在 5 分钟内出现                                                                                   | 数据通常在 4 小时内刷新，最长可能需要 24 小时（请参阅[数据新鲜度](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#data-freshness)）                                                                                                                             |
| `data` 是时间桶列表，最早的在前，包括空时间桶，每个时间桶都有一个 `results` 列表，其条目将维度和指标并列存放                                 | `data` 是按 `order_by` 排序的行列表，每行都有自己的 `starting_at` 和 `ending_at`，维度位于 `group` 中，指标位于 `usage` 或 `cost` 中。没有用量或成本的组合不会有对应的行。                                                                                                                                        |
| `bucket_width`，默认为 `1d`                                                                         | `time_bucket_width`，无默认值。传递 `time_bucket_width=1d` 以保留每日行；省略它则获取整个范围的总计。成本端点还接受所有筛选器以及 `1m` 和 `1h` 时间桶。                                                                                                                                                          |
| `limit` 计算每页的时间桶数                                                                               | `limit` 计算每页的行数（1 到 1,000，默认为 20），因此页数取决于有多少组合有数据                                                                                                                                                                                                                |
| 任意日期范围，按时间桶向前分页                                                                                 | `starting_at` 不早于 2026 年 1 月 1 日且在过去 365 天内，每个请求最多 31 天，使用 `time_bucket_width` 时必须提供 `ending_at`，使用 `1m` 时间桶时最多 24 小时（请参阅[日期范围和时间桶](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#date-ranges-and-time-buckets)）。请将更长的范围拆分为多个请求。 |
| `account_ids[]`、`service_account_ids[]`、`group_by[]=account_id`、`group_by[]=service_account_id` | 使用带有相同标记 ID 的 `actor_ids[]`，以及 `group_by[]=actor` 或 `actor_type`；读取 `actor.user_id` 或 `actor.service_account_id`。请参阅[操作者](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#actors)。                                                 |
| `context_window[]`                                                                              | `context_windows[]`                                                                                                                                                                                                                                              |
| 成本报告上的 `group_by[]=description`，带有 `description` 字符串                                            | 没有 `description`。请改为按结构化维度分组：`model`、`cost_type`、`token_type`、`context_window`、`service_tier`、`inference_geo` 和 `speed` 中的任意维度，它们都可以放在一个请求中。                                                                                                                     |
| 未知的 API 密钥或工作区 ID 会返回错误                                                                         | 不匹配任何内容，包括拼写错误的 `models[]` 名称。每个列表最多接受 100 个值。                                                                                                                                                                                                                   |
| 默认工作区的 `workspace_id: null`                                                                     | 默认工作区的真实 `workspace_id`（请参阅[默认工作区如何表示？](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#how-is-the-default-workspace-represented)）                                                                                                 |
| 没有地理位置或层级的用量为 `inference_geo: "not_available"` 和 `service_tier: "standard"`                     | 两者均为 `null`。`inference_geos[]=not_available` 仍会匹配没有推理地理位置的行。                                                                                                                                                                                                     |
| 成本的 `service_tier` 为 `standard` 或 `batch`；`amount` 形如 `"123.45"`                                | 实际层级；`amount` 保留六位小数，形如 `"123.450000"`，另加 `list_amount`。                                                                                                                                                                                                         |
| 代码执行不在用量报告中；没有请求计数                                                                              | 代码执行以 `model: null` 的用量行出现；两个端点都新增 `requests`，用量行新增 `total_tokens`                                                                                                                                                                                               |

要重现旧版每日报告，请使用相同的 `group_by[]` 请求 `time_bucket_width=1d`，使用 `limit=1000` 逐页获取，然后按 `starting_at` 对行进行分组，并用零填充缺失的日期。添加 `order_by=time&order=asc` 可按顺序获取各天数据。以下是按模型统计 1 周每日用量的旧写法：

```bash cURL
curl --globoff "https://api.anthropic.com/v1/organizations/usage_report/messages?\
starting_at=2026-09-01T00:00:00Z&\
ending_at=2026-09-08T00:00:00Z&\
group_by[]=model&\
bucket_width=1d" \
  -H "anthropic-version: 2026-10-21" \
  -H "x-api-key: $ANTHROPIC_API_KEY"
```

迁移后：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/usage?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-09-08T00:00:00Z&\
  group_by[]=model&\
  time_bucket_width=1d&\
  limit=1000" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:usage list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-09-08T00:00:00Z \
    --time-bucket-width 1d \
    --group-by model \
    --limit 1000
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.usage.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-09-08T00:00:00Z",
      time_bucket_width="1d",
      group_by=["model"],
      limit=1000,
  )

  for row in rows:
      print(f"{row.starting_at} {row.group.model}: {row.usage.total_tokens}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.usage.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-09-08T00:00:00Z",
    time_bucket_width: "1d",
    group_by: ["model"],
    limit: 1000
  });

  for await (const row of rows) {
    console.log(`${row.starting_at} ${row.group.model}: ${row.usage.total_tokens}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics;
  using Anthropic.Models.Beta.Organization.Analytics.Usage;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Usage.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-09-08T00:00:00Z"),
          TimeBucketWidth = BetaAnalyticsTimeBucketWidth.Day,
          GroupBy = [BetaAnalyticsUsageGroupBy.Model],
          Limit = 1000,
      }
  );

  await foreach (var row in page.Paginate())
  {
      Console.WriteLine($"{row.StartingAt} {row.Group.Model}: {row.Usage.TotalTokens}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Usage.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsUsageListParams{
  	StartingAt:      time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:        anthropic.Time(time.Date(2026, time.September, 8, 0, 0, 0, 0, time.UTC)),
  	TimeBucketWidth: anthropic.BetaAnalyticsTimeBucketWidthDay,
  	GroupBy:         []anthropic.BetaAnalyticsUsageGroupBy{anthropic.BetaAnalyticsUsageGroupByModel},
  	Limit:           anthropic.Int(1000),
  })

  for rows.Next() {
  	row := rows.Current()
  	fmt.Printf("%s %s: %d\n", row.StartingAt.Format(time.RFC3339), row.Group.Model, row.Usage.TotalTokens)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.BetaAnalyticsTimeBucketWidth;
  import com.anthropic.models.beta.organization.analytics.usage.BetaAnalyticsUsageGroupBy;
  import com.anthropic.models.beta.organization.analytics.usage.UsageListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UsageListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-09-08T00:00:00Z"))
          .timeBucketWidth(BetaAnalyticsTimeBucketWidth.DAY)
          .addGroupBy(BetaAnalyticsUsageGroupBy.MODEL)
          .limit(1000)
          .build();
      var rows = client.beta().organization().analytics().usage().list(params);

      for (var row : rows.autoPager()) {
          IO.println(row.startingAt() + " " + row.group().model().orElse(null) + ": " + row.usage().totalTokens());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->usage->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-09-08T00:00:00Z'),
      timeBucketWidth: '1d',
      groupBy: ['model'],
      limit: 1000,
  );

  foreach ($rows->pagingEachItem() as $row) {
      $startingAt = $row->startingAt->format(\DateTimeInterface::ATOM);
      echo "{$startingAt} {$row->group->model}: {$row->usage->totalTokens}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.usage.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 9, 8),
    time_bucket_width: :"1d",
    group_by: [:model],
    limit: 1000
  )

  page.auto_paging_each do |row|
    puts "#{row.starting_at} #{row.group.model}: #{row.usage.total_tokens}"
  end
  ```
</CodeGroup>

按工作区、模型、服务层级、上下文窗口和推理地理位置划分的每日成本，并按成本类型和令牌类型拆分：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/cost?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-09-08T00:00:00Z&\
  time_bucket_width=1d&\
  group_by[]=workspace_id&\
  group_by[]=model&\
  group_by[]=service_tier&\
  group_by[]=context_window&\
  group_by[]=inference_geo&\
  group_by[]=cost_type&\
  group_by[]=token_type&\
  limit=1000" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:cost list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-09-08T00:00:00Z \
    --time-bucket-width 1d \
    --group-by workspace_id \
    --group-by model \
    --group-by service_tier \
    --group-by context_window \
    --group-by inference_geo \
    --group-by cost_type \
    --group-by token_type \
    --limit 1000
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.cost.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-09-08T00:00:00Z",
      time_bucket_width="1d",
      group_by=[
          "workspace_id",
          "model",
          "service_tier",
          "context_window",
          "inference_geo",
          "cost_type",
          "token_type",
      ],
      limit=1000,
  )

  for row in rows:
      group = row.group
      print(
          f"{row.starting_at} {group.workspace_id} {group.model} {group.service_tier} "
          f"{group.context_window} {group.inference_geo} {group.cost_type} {group.token_type}: "
          f"{row.cost.amount}"
      )
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.cost.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-09-08T00:00:00Z",
    time_bucket_width: "1d",
    group_by: [
      "workspace_id",
      "model",
      "service_tier",
      "context_window",
      "inference_geo",
      "cost_type",
      "token_type"
    ],
    limit: 1000
  });

  for await (const row of rows) {
    const { group } = row;
    console.log(
      `${row.starting_at} ${group.workspace_id} ${group.model} ${group.service_tier} ` +
        `${group.context_window} ${group.inference_geo} ${group.cost_type} ${group.token_type}: ` +
        `${row.cost.amount}`
    );
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics;
  using Anthropic.Models.Beta.Organization.Analytics.Cost;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Cost.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-09-08T00:00:00Z"),
          TimeBucketWidth = BetaAnalyticsTimeBucketWidth.Day,
          GroupBy =
          [
              BetaAnalyticsCostGroupBy.WorkspaceID,
              BetaAnalyticsCostGroupBy.Model,
              BetaAnalyticsCostGroupBy.ServiceTier,
              BetaAnalyticsCostGroupBy.ContextWindow,
              BetaAnalyticsCostGroupBy.InferenceGeo,
              BetaAnalyticsCostGroupBy.CostType,
              BetaAnalyticsCostGroupBy.TokenType,
          ],
          Limit = 1000,
      }
  );

  await foreach (var row in page.Paginate())
  {
      var group = row.Group;
      Console.WriteLine(
          $"{row.StartingAt} {group.WorkspaceID} {group.Model} {group.ServiceTier?.Raw()} "
              + $"{group.ContextWindow?.Raw()} {group.InferenceGeo?.Raw()} {group.CostType?.Raw()} "
              + $"{group.TokenType?.Raw()}: {row.Cost.Amount}"
      );
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Cost.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsCostListParams{
  	StartingAt:      time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:        anthropic.Time(time.Date(2026, time.September, 8, 0, 0, 0, 0, time.UTC)),
  	TimeBucketWidth: anthropic.BetaAnalyticsTimeBucketWidthDay,
  	GroupBy: []anthropic.BetaAnalyticsCostGroupBy{
  		anthropic.BetaAnalyticsCostGroupByWorkspaceID,
  		anthropic.BetaAnalyticsCostGroupByModel,
  		anthropic.BetaAnalyticsCostGroupByServiceTier,
  		anthropic.BetaAnalyticsCostGroupByContextWindow,
  		anthropic.BetaAnalyticsCostGroupByInferenceGeo,
  		anthropic.BetaAnalyticsCostGroupByCostType,
  		anthropic.BetaAnalyticsCostGroupByTokenType,
  	},
  	Limit: anthropic.Int(1000),
  })

  for rows.Next() {
  	row := rows.Current()
  	group := row.Group
  	fmt.Printf("%s %s %s %s %s %s %s %s: %s\n",
  		row.StartingAt.Format(time.RFC3339), group.WorkspaceID, group.Model, group.ServiceTier,
  		group.ContextWindow, group.InferenceGeo, group.CostType, group.TokenType, row.Cost.Amount)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.BetaAnalyticsTimeBucketWidth;
  import com.anthropic.models.beta.organization.analytics.cost.BetaAnalyticsCostGroupBy;
  import com.anthropic.models.beta.organization.analytics.cost.CostListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = CostListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-09-08T00:00:00Z"))
          .timeBucketWidth(BetaAnalyticsTimeBucketWidth.DAY)
          .addGroupBy(BetaAnalyticsCostGroupBy.WORKSPACE_ID)
          .addGroupBy(BetaAnalyticsCostGroupBy.MODEL)
          .addGroupBy(BetaAnalyticsCostGroupBy.SERVICE_TIER)
          .addGroupBy(BetaAnalyticsCostGroupBy.CONTEXT_WINDOW)
          .addGroupBy(BetaAnalyticsCostGroupBy.INFERENCE_GEO)
          .addGroupBy(BetaAnalyticsCostGroupBy.COST_TYPE)
          .addGroupBy(BetaAnalyticsCostGroupBy.TOKEN_TYPE)
          .limit(1000)
          .build();
      var rows = client.beta().organization().analytics().cost().list(params);

      for (var row : rows.autoPager()) {
          var group = row.group();
          IO.println(
              row.startingAt() + " " + group.workspaceId().orElse(null) + " " + group.model().orElse(null) + " "
                  + group.serviceTier().orElse(null) + " " + group.contextWindow().orElse(null) + " "
                  + group.inferenceGeo().orElse(null) + " " + group.costType().orElse(null) + " "
                  + group.tokenType().orElse(null) + ": " + row.cost().amount()
          );
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->cost->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-09-08T00:00:00Z'),
      timeBucketWidth: '1d',
      groupBy: [
          'workspace_id',
          'model',
          'service_tier',
          'context_window',
          'inference_geo',
          'cost_type',
          'token_type',
      ],
      limit: 1000,
  );

  foreach ($rows->pagingEachItem() as $row) {
      $startingAt = $row->startingAt->format(\DateTimeInterface::ATOM);
      $group = $row->group;
      echo "{$startingAt} {$group->workspaceID} {$group->model} {$group->serviceTier} "
          . "{$group->contextWindow} {$group->inferenceGeo} {$group->costType} {$group->tokenType}: "
          . "{$row->cost->amount}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.cost.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 9, 8),
    time_bucket_width: :"1d",
    group_by: [
      :workspace_id,
      :model,
      :service_tier,
      :context_window,
      :inference_geo,
      :cost_type,
      :token_type
    ],
    limit: 1000
  )

  page.auto_paging_each do |row|
    group = row.group
    puts "#{row.starting_at} #{group.workspace_id} #{group.model} #{group.service_tier} " \
      "#{group.context_window} #{group.inference_geo} #{group.cost_type} #{group.token_type}: " \
      "#{row.cost.amount}"
  end
  ```
</CodeGroup>

## 从 Claude Enterprise 用量和成本报告迁移

本节适用于调用 [Claude Enterprise Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api) 中 `/v1/organizations/analytics/` 下的报告端点 `usage_report`、`cost_report`、`user_usage_report` 和 `user_cost_report` 的 Claude Enterprise 组织。这些端点为旧版，仍受支持。当您需要以下任一功能时，请迁移到 `/analytics/usage` 和 `/analytics/cost`：

* 每个时间桶中的每个分组，而不是每个时间桶中的前 100 个分组
* 在用量和成本上，均可将 `actor` 或 `actor_type` 与其他维度一起分组
* 按您选择的指标在整个时间范围内对行进行排名，或按时间桶排序
* 一次集成即可同时服务 Claude Console 组织

您的密钥、最早日期、365 天回溯期和 31 天跨度、筛选器和维度的名称及值（`user_ids[]` 和 Claude Tag 产品值除外）、用户 `actor` 字段、指标和金额字段、`data_refreshed_at`、游标规则以及[组织级速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api#working-with-the-api)均保持不变。唯一新增的标头是 `analytics-usage-cost-2026-08-24` beta 标头。变化如下：

| Claude Enterprise 报告                                                                                         | 用量和成本分析 API                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `usage_report` 和 `cost_report` 返回时间桶，最早的在前，包括空时间桶，每个时间桶最多包含其前 100 个分组；`bucket_width` 默认为 `1d`，`limit` 计算时间桶数 | 涵盖每个有数据的组合的行，按 `order_by` 排序，每行都有自己的 `starting_at` 和 `ending_at`，维度位于 `group` 中，指标位于 `usage` 或 `cost` 中。`time_bucket_width` 是可选的，`limit` 计算行数。 |
| 时间桶参数为 `bucket_width`                                                                                        | 参数为 `time_bucket_width`，取值相同                                                                                                                   |
| 在 `usage_report` 和 `cost_report` 上，使用 `bucket_width` 时 `ending_at` 是可选的，`1m` 时间桶最多可跨越 31 天                   | 使用 `time_bucket_width` 时必须提供 `ending_at`；每个请求中 `1m` 时间桶最多跨越 24 小时                                                                              |
| `user_usage_report` 和 `user_cost_report` 每个用户返回一行                                                            | `group_by[]=actor&actor_types[]=user_actor`。如果不使用该筛选器，您还会获得非用户身份的行，以及一个用于未归因用量的 `actor: null` 行。                                               |
| Claude Tag 用量的 `product: "claude-tag"`                                                                       | `product: "claude_tag"`；`products[]` 接受任一拼写                                                                                                    |
| 无法识别的 `models[]` 名称会返回 400 错误                                                                                | 不匹配任何内容                                                                                                                                        |
| `user_ids[]`                                                                                                 | 使用带有相同标记 `user_...` ID 的 `actor_ids[]`                                                                                                         |
| 每用户报告上的 `exclude_deleted_users`                                                                              | 需要 `group_by[]=actor`；否则请求会返回 400 错误。它仅删除 `actor` 为 `deleted: true` 用户的行；其他身份的行和 `actor: null` 的行会保留。                                          |

一个月的每用户成本，迁移前：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/analytics/user_cost_report?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-10-01T00:00:00Z&\
  limit=100" \
    -H "anthropic-version: 2026-10-21" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:user-cost-report list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-10-01T00:00:00Z \
    --limit 100
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.user_cost_report.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-10-01T00:00:00Z",
      limit=100,
  )

  for row in rows:
      print(f"{row.actor.user_id}: {row.amount}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.userCostReport.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-10-01T00:00:00Z",
    limit: 100
  });

  for await (const row of rows) {
    console.log(`${row.actor.user_id}: ${row.amount}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.UserCostReport.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-10-01T00:00:00Z"),
          Limit = 100,
      }
  );

  await foreach (var row in page.Paginate())
  {
      Console.WriteLine($"{row.Actor.UserID}: {row.Amount}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.UserCostReport.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsUserCostReportListParams{
  	StartingAt: time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:   anthropic.Time(time.Date(2026, time.October, 1, 0, 0, 0, 0, time.UTC)),
  	Limit:      anthropic.Int(100),
  })

  for rows.Next() {
  	row := rows.Current()
  	fmt.Printf("%s: %s\n", row.Actor.UserID, row.Amount)
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.usercostreport.UserCostReportListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserCostReportListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-10-01T00:00:00Z"))
          .limit(100)
          .build();
      var rows = client.beta().organization().analytics().userCostReport().list(params);

      for (var row : rows.autoPager()) {
          IO.println(row.actor().userId() + ": " + row.amount());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rows = $client->beta->organization->analytics->userCostReport->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-10-01T00:00:00Z'),
      limit: 100,
  );

  foreach ($rows->pagingEachItem() as $row) {
      echo "{$row->actor->userID}: {$row->amount}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.user_cost_report.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 10, 1),
    limit: 100
  )

  page.auto_paging_each do |row|
    puts "#{row.actor.user_id}: #{row.amount}"
  end
  ```
</CodeGroup>

迁移后，并增加按产品的细分：

<CodeGroup>
  ```bash cURL
  curl --globoff "https://api.anthropic.com/v1/organizations/analytics/cost?\
  starting_at=2026-09-01T00:00:00Z&\
  ending_at=2026-10-01T00:00:00Z&\
  group_by[]=actor&\
  group_by[]=product&\
  actor_types[]=user_actor&\
  limit=100" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
    -H "x-api-key: $ANTHROPIC_API_KEY"
  ```

  ```bash CLI
  ant beta:organization:analytics:cost list \
    --starting-at 2026-09-01T00:00:00Z \
    --ending-at 2026-10-01T00:00:00Z \
    --actor-type user_actor \
    --group-by actor \
    --group-by product \
    --limit 100
  ```

  ```python Python
  client = anthropic.Anthropic()

  rows = client.beta.organization.analytics.cost.list(
      starting_at="2026-09-01T00:00:00Z",
      ending_at="2026-10-01T00:00:00Z",
      actor_types=["user_actor"],
      group_by=["actor", "product"],
      limit=100,
  )

  for row in rows:
      actor = row.group.actor
      if actor is not None and actor.type == "user_actor":
          print(f"{actor.user_id} {row.group.product}: {row.cost.amount}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rows = await client.beta.organization.analytics.cost.list({
    starting_at: "2026-09-01T00:00:00Z",
    ending_at: "2026-10-01T00:00:00Z",
    actor_types: ["user_actor"],
    group_by: ["actor", "product"],
    limit: 100
  });

  for await (const row of rows) {
    const { actor } = row.group;
    if (actor?.type === "user_actor") {
      console.log(`${actor.user_id} ${row.group.product}: ${row.cost.amount}`);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Analytics;
  using Anthropic.Models.Beta.Organization.Analytics.Cost;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Analytics.Cost.List(
      new()
      {
          StartingAt = DateTimeOffset.Parse("2026-09-01T00:00:00Z"),
          EndingAt = DateTimeOffset.Parse("2026-10-01T00:00:00Z"),
          ActorTypes = [BetaAnalyticsActorType.UserActor],
          GroupBy = [BetaAnalyticsCostGroupBy.Actor, BetaAnalyticsCostGroupBy.Product],
          Limit = 100,
      }
  );

  await foreach (var row in page.Paginate())
  {
      if (row.Group.Actor is { } actor && actor.TryPickReportUser(out var user))
      {
          Console.WriteLine($"{user.UserID} {row.Group.Product}: {row.Cost.Amount}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rows := client.Beta.Organization.Analytics.Cost.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsCostListParams{
  	StartingAt: time.Date(2026, time.September, 1, 0, 0, 0, 0, time.UTC),
  	EndingAt:   anthropic.Time(time.Date(2026, time.October, 1, 0, 0, 0, 0, time.UTC)),
  	ActorTypes: []anthropic.BetaAnalyticsActorType{anthropic.BetaAnalyticsActorTypeUserActor},
  	GroupBy:    []anthropic.BetaAnalyticsCostGroupBy{anthropic.BetaAnalyticsCostGroupByActor, anthropic.BetaAnalyticsCostGroupByProduct},
  	Limit:      anthropic.Int(100),
  })

  for rows.Next() {
  	row := rows.Current()
  	if user, ok := row.Group.Actor.AsAny().(anthropic.BetaAnalyticsReportUserActor); ok {
  		fmt.Printf("%s %s: %s\n", user.UserID, row.Group.Product, row.Cost.Amount)
  	}
  }
  if err := rows.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.analytics.BetaAnalyticsActorType;
  import com.anthropic.models.beta.organization.analytics.cost.BetaAnalyticsCostGroupBy;
  import com.anthropic.models.beta.organization.analytics.cost.CostListParams;
  import java.time.OffsetDateTime;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = CostListParams.builder()
          .startingAt(OffsetDateTime.parse("2026-09-01T00:00:00Z"))
          .endingAt(OffsetDateTime.parse("2026-10-01T00:00:00Z"))
          .addActorType(BetaAnalyticsActorType.USER_ACTOR)
          .addGroupBy(BetaAnalyticsCostGroupBy.ACTOR)
          .addGroupBy(BetaAnalyticsCostGroupBy.PRODUCT)
          .limit(100)
          .build();
      var rows = client.beta().organization().analytics().cost().list(params);

      for (var row : rows.autoPager()) {
          row.group().actor().flatMap(actor -> actor.user()).ifPresent(user ->
              IO.println(user.userId() + " " + row.group().product().orElse(null) + ": " + row.cost().amount())
          );
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\Analytics\AnalyticsReportUserActor;
  // ...

  $client = new Client();

  $rows = $client->beta->organization->analytics->cost->list(
      startingAt: new \DateTimeImmutable('2026-09-01T00:00:00Z'),
      endingAt: new \DateTimeImmutable('2026-10-01T00:00:00Z'),
      actorTypes: ['user_actor'],
      groupBy: ['actor', 'product'],
      limit: 100,
  );

  foreach ($rows->pagingEachItem() as $row) {
      $actor = $row->group->actor;
      if ($actor instanceof AnalyticsReportUserActor) {
          echo "{$actor->userID} {$row->group->product}: {$row->cost->amount}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.analytics.cost.list(
    starting_at: Time.utc(2026, 9, 1),
    ending_at: Time.utc(2026, 10, 1),
    actor_types: [:user_actor],
    group_by: [:actor, :product],
    limit: 100
  )

  page.auto_paging_each do |row|
    actor = row.group.actor
    if actor.is_a?(Anthropic::Models::Beta::Organization::BetaAnalyticsReportUserActor)
      puts "#{actor.user_id} #{row.group.product}: #{row.cost.amount}"
    end
  end
  ```
</CodeGroup>

## 常见问题

### 默认工作区如何表示？

与其他所有工作区一样，Claude Console 组织的默认工作区也有自己的 `wrkspc_` ID。默认工作区的行在 `workspace_id` 中包含该 ID，您可以使用 `workspace_ids[]` 按其进行筛选。除非您传递 `include_default=true`，否则[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)不会包含默认工作区。要检查某个不熟悉的 `workspace_id` 是否为默认工作区，请将其传递给[获取工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/retrieve)：默认工作区会返回 `"name": "Default"`。

### 如何获取 Claude Code 的每用户成本？

使用 `products[]=claude_code&group_by[]=actor` 调用成本端点。Claude Console 组织还可以使用 [Claude Code 分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-code-analytics-api)，它报告每用户的 Claude Code 活动和估算成本。

## 另请参阅

* [用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)（旧版）
* [Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api)：您的组织需要哪种分析 API 和密钥类型
* [Claude Code 分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-code-analytics-api)
* [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization)
* [定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)
