---
title: 速率限制 API
url: https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api
description: 使用速率限制 API 以编程方式读取您组织的 API 速率限制并设置工作区速率限制。
---

<Tip>
  **Admin API 不适用于 claude.ai Team 计划中的组织，也不适用于个人 Claude Console 账户。** 如果您的公司还拥有 Claude Console 组织，您可以通过该组织使用此 API；请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。个人 Console 账户必须先在 Claude Console 中的[组织](https://platform.claude.com/settings/organization)页面（`platform.claude.com/settings/organization`）上设置组织。
</Tip>

速率限制 API 提供对您的组织及其工作区所配置的 “rate limits”（速率限制）的编程访问。这与 [Claude Console 中的速率限制页面](https://platform.claude.com/usage/limits)上显示的信息相同。您还可以使用它来设置和清除工作区的速率限制。

使用此 API 可以：

* **保持网关和代理同步：** 在启动时和按计划读取您当前的限制，而不是硬编码那些在 Anthropic 调整限制后会过时的值。
* **查看剩余额度：** 将来自[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 的过往使用数据与您配置的限制进行比较。
* **审计工作区配置：** 验证工作区覆盖设置是否与您的配置自动化所预期的一致。
* **通过代码管理工作区限制：** 从您的预配自动化中设置和清除每个工作区的限制，而不是在 Console 中编辑它们。

<Check>
  **需要 Admin API 凭证。** 这些端点属于 Admin API 的一部分。您可以使用 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)、具有 `org:admin` 作用域的 OAuth 令牌，或未限定于某个工作区的个人或服务账户密钥来访问它们；工作区 API 密钥无法使用。详情请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)。
</Check>

SDK 的默认客户端和 CLI 从 `ANTHROPIC_API_KEY` 环境变量中读取 Admin API 密钥。SDK 将这些端点公开为 `client.organization.rate_limits` (typescript: `client.organization.rateLimits`; csharp, go: `client.Organization.RateLimits`; java: `client.organization().rateLimits()`; php: `$client->organization->rateLimits`) 和 `client.organization.workspaces.rate_limits` (typescript: `client.organization.workspaces.rateLimits`; csharp, go: `client.Organization.Workspaces.RateLimits`; java: `client.organization().workspaces().rateLimits()`; php: `$client->organization->workspaces->rateLimits`)。每个列表示例中的注释说明了它是会获取更多页面还是仅读取第一页。

## 快速开始

列出为您的组织配置的速率限制：

<CodeGroup>
  ```bash cURL
  # 返回一页结果；省略 `limit` 时，该页包含所有条目。
  curl "https://api.anthropic.com/v1/organizations/rate_limits" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 根据需要自动获取更多页面。
  ant organization:rate-limits list
  ```

  ```python Python
  client = anthropic.Anthropic()

  rate_limits = client.organization.rate_limits.list()

  # 根据需要自动获取更多页面。
  for entry in rate_limits:
      models = f" ({', '.join(entry.models)})" if entry.models else ""
      print(f"{entry.group.type}{models}")
      for limit in entry.limits:
          print(f"  {limit.type}: {limit.value}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rateLimits = await client.organization.rateLimits.list();

  // 根据需要自动获取更多页面。
  for await (const entry of rateLimits) {
    const models = entry.models ? ` (${entry.models.join(", ")})` : "";
    console.log(`${entry.group.type}${models}`);
    for (const limit of entry.limits) {
      console.log(`  ${limit.type}: ${limit.value}`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var rateLimits = await client.Organization.RateLimits.List();

  // 根据需要自动获取更多页面。
  await foreach (var entry in rateLimits.Paginate())
  {
      var models = entry.Models is null ? "" : $" ({string.Join(", ", entry.Models)})";
      Console.WriteLine($"{entry.Group.Type.Raw()}{models}");
      foreach (var limit in entry.Limits)
      {
          Console.WriteLine($"  {limit.Type}: {limit.Value}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rateLimits := client.Organization.RateLimits.ListAutoPaging(context.Background(), anthropic.OrganizationRateLimitListParams{})

  // 根据需要自动获取更多页面。
  for rateLimits.Next() {
  	entry := rateLimits.Current()
  	models := ""
  	if len(entry.Models) > 0 {
  		models = fmt.Sprintf(" (%s)", strings.Join(entry.Models, ", "))
  	}
  	fmt.Printf("%s%s\n", entry.Group.Type, models)
  	for _, limit := range entry.Limits {
  		fmt.Printf("  %s: %d\n", limit.Type, limit.Value)
  	}
  }
  if err := rateLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var rateLimits = client.organization().rateLimits().list();

  // 根据需要自动获取更多页面。
  for (var entry : rateLimits.autoPager()) {
      var models = entry.models()
          .map(modelIds -> " (" + String.join(", ", modelIds) + ")")
          .orElse("");
      IO.println(entry.group().type().asString() + models);
      for (var limit : entry.limits()) {
          IO.println("  " + limit.type() + ": " + limit.value());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rateLimits = $client->organization->rateLimits->list();

  // 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  foreach ($rateLimits->data as $entry) {
      $models = $entry->models ? ' (' . implode(', ', $entry->models) . ')' : '';
      echo "{$entry->group->type}{$models}\n";
      foreach ($entry->limits as $limit) {
          echo "  {$limit->type}: {$limit->value}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rate_limits = client.organization.rate_limits.list

  # 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  rate_limits.data.each do |entry|
    models = entry.models ? " (#{entry.models.join(", ")})" : ""
    puts "#{entry.group.type}#{models}"
    entry.limits.each do |limit|
      puts "  #{limit.type}: #{limit.value}"
    end
  end
  ```
</CodeGroup>

## 组织速率限制

`/v1/organizations/rate_limits` 端点返回在组织级别应用于 Messages API 及其支持资源的速率限制。其他产品（例如 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)）的限制不包括在内。

### 关键概念

* **速率限制组：** 响应中的每个条目代表一个速率限制组。模型速率限制经过分组，使多个模型版本共享同一组限制。其他组涵盖 Message Batches API 和网络搜索工具。
* **`group` 对象：** 存在于每个条目中，用于标识该条目所适用的速率限制组。它始终包含 `type`（取值为 `group_type` 值之一）和 `id`（一个带有 `rlg_` 前缀的不透明标识符）。在 `model_group` 条目中，它还包含 `display_name`，即 Anthropic 当前为该组使用的标签，例如 `Claude Sonnet 4.x`。该标签仅用于显示，可能会发生变化。其他组类型没有 `display_name`。
* **`group` 内的 `id`：** 一个组在每个组织和每个工作区覆盖中都具有相同的 `id`，且永远不会改变。使用它可以跨组织匹配条目，或与您自己的目录进行匹配。条目自身的 `id` 因组织而异，而当 Anthropic 在组之间移动模型时，`models` 也会变化。两者都不是该组的稳定键。
* **`group_type`：** 用于将列表限制为单个组类型的查询参数。有关取值，请参阅[按组类型筛选](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api#filtering-by-group-type)。
* **`models` 列表：** 对于 `model_group` 条目，`models` 字段列出计入该组限制的每个模型 ID 和别名。使用此列表可以查找任意模型字符串所属的组。对于其他组类型，`models` 为 `null`。
* **`limits` 列表：** 每个组都带有一个 `{type, value}` 对列表。`type` 字段标识限制器（例如 `requests_per_minute`、`input_tokens_per_minute` 或 `output_tokens_per_minute`），`value` 是配置的限制值。有关每个限制器的衡量和执行方式，请参阅[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)。
* **工作区限制上的 `source`：** 在工作区端点上，每个限制值还带有 `source`，用于说明 `value` 的来源：`{"type": "workspace"}` 表示存储在工作区上的覆盖设置，`{"type": "organization"}` 表示从组织继承的值。默认情况下，工作区端点仅返回覆盖设置，因此其中的 `source.type` 始终为 `workspace`；当您传入 `include_inherited=true` 时，继承的值才会出现。请参阅[工作区速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api#workspace-rate-limits)。

有关完整的参数详情和响应模式，请参阅[组织速率限制 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/rate_limits/list)。

### 列出所有组织速率限制

<CodeGroup>
  ```bash cURL
  # 返回一页结果；省略 `limit` 时，该页包含所有条目。
  curl "https://api.anthropic.com/v1/organizations/rate_limits" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 根据需要自动获取更多页面。
  ant organization:rate-limits list
  ```

  ```python Python
  client = anthropic.Anthropic()

  rate_limits = client.organization.rate_limits.list()

  # 根据需要自动获取更多页面。
  for entry in rate_limits:
      models = f" ({', '.join(entry.models)})" if entry.models else ""
      print(f"{entry.group.type}{models}")
      for limit in entry.limits:
          print(f"  {limit.type}: {limit.value}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rateLimits = await client.organization.rateLimits.list();

  // 根据需要自动获取更多页面。
  for await (const entry of rateLimits) {
    const models = entry.models ? ` (${entry.models.join(", ")})` : "";
    console.log(`${entry.group.type}${models}`);
    for (const limit of entry.limits) {
      console.log(`  ${limit.type}: ${limit.value}`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var rateLimits = await client.Organization.RateLimits.List();

  // 根据需要自动获取更多页面。
  await foreach (var entry in rateLimits.Paginate())
  {
      var models = entry.Models is null ? "" : $" ({string.Join(", ", entry.Models)})";
      Console.WriteLine($"{entry.Group.Type.Raw()}{models}");
      foreach (var limit in entry.Limits)
      {
          Console.WriteLine($"  {limit.Type}: {limit.Value}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rateLimits := client.Organization.RateLimits.ListAutoPaging(context.Background(), anthropic.OrganizationRateLimitListParams{})

  // 根据需要自动获取更多页面。
  for rateLimits.Next() {
  	entry := rateLimits.Current()
  	models := ""
  	if len(entry.Models) > 0 {
  		models = fmt.Sprintf(" (%s)", strings.Join(entry.Models, ", "))
  	}
  	fmt.Printf("%s%s\n", entry.Group.Type, models)
  	for _, limit := range entry.Limits {
  		fmt.Printf("  %s: %d\n", limit.Type, limit.Value)
  	}
  }
  if err := rateLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var rateLimits = client.organization().rateLimits().list();

  // 根据需要自动获取更多页面。
  for (var entry : rateLimits.autoPager()) {
      var models = entry.models()
          .map(modelIds -> " (" + String.join(", ", modelIds) + ")")
          .orElse("");
      IO.println(entry.group().type().asString() + models);
      for (var limit : entry.limits()) {
          IO.println("  " + limit.type() + ": " + limit.value());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rateLimits = $client->organization->rateLimits->list();

  // 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  foreach ($rateLimits->data as $entry) {
      $models = $entry->models ? ' (' . implode(', ', $entry->models) . ')' : '';
      echo "{$entry->group->type}{$models}\n";
      foreach ($entry->limits as $limit) {
          echo "  {$limit->type}: {$limit->value}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rate_limits = client.organization.rate_limits.list

  # 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  rate_limits.data.each do |entry|
    models = entry.models ? " (#{entry.models.join(", ")})" : ""
    puts "#{entry.group.type}#{models}"
    entry.limits.each do |limit|
      puts "  #{limit.type}: #{limit.value}"
    end
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "id": "rlt_01NwFawrTvAijwqn8AjKVBPS",
      "type": "rate_limit",
      "group": {
        "type": "model_group",
        "id": "rlg_01Hq7YkP3mZ9dTwRx4cVbN2s",
        "display_name": "Claude Opus 5.5"
      },
      "models": ["claude-opus-5-5"],
      "limits": [
        { "type": "requests_per_minute", "value": 4000 },
        { "type": "input_tokens_per_minute", "value": 10000000 },
        { "type": "output_tokens_per_minute", "value": 800000 }
      ]
    },
    {
      "id": "rlt_016qriyigbdebqkzhMujdHkX",
      "type": "rate_limit",
      "group": {
        "type": "model_group",
        "id": "rlg_01Kd5wMv8nSq2LcXy6tRfJ4b",
        "display_name": "Claude Opus 4.x"
      },
      "models": [
        "claude-opus-4-5",
        "claude-opus-4-5-20251101",
        "claude-opus-4-6",
        "claude-opus-4-7",
        "claude-opus-4-8"
      ],
      "limits": [
        { "type": "requests_per_minute", "value": 4000 },
        { "type": "input_tokens_per_minute", "value": 10000000 },
        { "type": "output_tokens_per_minute", "value": 800000 }
      ]
    },
    {
      "id": "rlt_01EyY8kq14eviKJPgepbgBH4",
      "type": "rate_limit",
      "group": { "type": "batch", "id": "rlg_01Wn3pBz6kCg9vHtQ7mLxD5a" },
      "models": null,
      "limits": [{ "type": "enqueued_batch_requests", "value": 500000 }]
    }
  ],
  "next_page": null
}
```

### 查找特定模型的限制

将任意模型 ID 或别名作为 `model` 查询参数传入，即可仅返回包含该模型的条目：

<CodeGroup>
  ```bash cURL
  # 返回一页结果；省略 `limit` 时，该页包含所有条目。
  curl "https://api.anthropic.com/v1/organizations/rate_limits?model=claude-opus-5-5" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 根据需要自动获取更多页面。
  ant organization:rate-limits list --model claude-opus-5-5
  ```

  ```python Python
  client = anthropic.Anthropic()

  rate_limits = client.organization.rate_limits.list(model="claude-opus-5-5")

  # 根据需要自动获取更多页面。
  for entry in rate_limits:
      models = f" ({', '.join(entry.models)})" if entry.models else ""
      print(f"{entry.group.type}{models}")
      for limit in entry.limits:
          print(f"  {limit.type}: {limit.value}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rateLimits = await client.organization.rateLimits.list({
    model: "claude-opus-5-5"
  });

  // 根据需要自动获取更多页面。
  for await (const entry of rateLimits) {
    const models = entry.models ? ` (${entry.models.join(", ")})` : "";
    console.log(`${entry.group.type}${models}`);
    for (const limit of entry.limits) {
      console.log(`  ${limit.type}: ${limit.value}`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var rateLimits = await client.Organization.RateLimits.List(new()
  {
      Model = "claude-opus-5-5"
  });

  // 根据需要自动获取更多页。
  await foreach (var entry in rateLimits.Paginate())
  {
      var models = entry.Models is null ? "" : $" ({string.Join(", ", entry.Models)})";
      Console.WriteLine($"{entry.Group.Type.Raw()}{models}");
      foreach (var limit in entry.Limits)
      {
          Console.WriteLine($"  {limit.Type}: {limit.Value}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rateLimits := client.Organization.RateLimits.ListAutoPaging(context.Background(), anthropic.OrganizationRateLimitListParams{
  	Model: anthropic.String(anthropic.ModelClaudeOpus5_5),
  })

  // 根据需要自动获取更多页面。
  for rateLimits.Next() {
  	entry := rateLimits.Current()
  	models := ""
  	if len(entry.Models) > 0 {
  		models = fmt.Sprintf(" (%s)", strings.Join(entry.Models, ", "))
  	}
  	fmt.Printf("%s%s\n", entry.Group.Type, models)
  	for _, limit := range entry.Limits {
  		fmt.Printf("  %s: %d\n", limit.Type, limit.Value)
  	}
  }
  if err := rateLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.ratelimits.RateLimitListParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RateLimitListParams.builder()
          .model(Model.CLAUDE_OPUS_5_5.asString())
          .build();
      var rateLimits = client.organization().rateLimits().list(params);

      // 根据需要自动获取更多页面。
      for (var entry : rateLimits.autoPager()) {
          var models = entry.models()
              .map(modelIds -> " (" + String.join(", ", modelIds) + ")")
              .orElse("");
          IO.println(entry.group().type().asString() + models);
          for (var limit : entry.limits()) {
              IO.println("  " + limit.type() + ": " + limit.value());
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;

  $client = new Client();

  $rateLimits = $client->organization->rateLimits->list(
      model: Model::CLAUDE_OPUS_5_5->value,
  );

  // 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  foreach ($rateLimits->data as $entry) {
      $models = $entry->models ? ' (' . implode(', ', $entry->models) . ')' : '';
      echo "{$entry->group->type}{$models}\n";
      foreach ($entry->limits as $limit) {
          echo "  {$limit->type}: {$limit->value}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rate_limits = client.organization.rate_limits.list(model: Anthropic::Model::CLAUDE_OPUS_5_5)

  # 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  rate_limits.data.each do |entry|
    models = entry.models ? " (#{entry.models.join(", ")})" : ""
    puts "#{entry.group.type}#{models}"
    entry.limits.each do |limit|
      puts "  #{limit.type}: #{limit.value}"
    end
  end
  ```
</CodeGroup>

如果模型字符串不匹配任何组，端点将返回 404 错误。您的组织有权访问的任何模型都会匹配到其所属的组。`model` 参数仅在组织端点上受支持；工作区端点不接受该参数。

## 工作区速率限制

`/v1/organizations/workspaces/{workspace_id}/rate_limits` 端点返回为单个工作区配置的速率限制覆盖值。若还要查看工作区继承的值，请传入 `include_inherited=true`（请参阅[包含继承的值](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api#include-inherited-values)）。若要更改覆盖值，请参阅[设置工作区速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api#set-workspace-rate-limits)。

默认情况下，响应仅包含覆盖设置，因此其中缺失的任何内容都继承自组织：

* 在 `data` 中不存在的组完全没有工作区覆盖设置。该工作区继承该组的组织级限制（并非无限制）。
* 在存在的组中，`limits[]` 中不存在的限制器类型没有针对该限制器的工作区覆盖设置。该工作区继承其组织值。
* 对于每个存在的限制器，`org_limit` 是同一限制器的组织级值；如果组织没有为该限制器类型配置限制，则为 `null`；`source` 为 `{"type": "workspace"}`。

有关完整的参数详情和响应模式，请参阅[工作区速率限制 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/rate_limits/list)。

<Tip>
  要获取您组织的工作区 ID，请使用[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)端点，或在 [Claude Console 的工作区](https://platform.claude.com/settings/workspaces)下查找。仅当您传入 `include_default=true` 时，列出工作区端点才会包含默认工作区。
</Tip>

<CodeGroup>
  ```bash cURL
  # 返回一页结果；省略 `limit` 时，该页包含所有条目。
  curl "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/rate_limits" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 根据需要自动获取更多页面。
  ant organization:workspaces:rate-limits list \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ
  ```

  ```python Python
  client = anthropic.Anthropic()

  rate_limits = client.organization.workspaces.rate_limits.list(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  )

  # 根据需要自动获取更多页面。
  for entry in rate_limits:
      models = f" ({', '.join(entry.models)})" if entry.models else ""
      print(f"{entry.group.type}{models}")
      for limit in entry.limits:
          print(f"  {limit.type}: {limit.value}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rateLimits = await client.organization.workspaces.rateLimits.list(
    "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  );

  // 根据需要自动获取更多页面。
  for await (const entry of rateLimits) {
    const models = entry.models ? ` (${entry.models.join(", ")})` : "";
    console.log(`${entry.group.type}${models}`);
    for (const limit of entry.limits) {
      console.log(`  ${limit.type}: ${limit.value}`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var rateLimits = await client.Organization.Workspaces.RateLimits.List(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  );

  // 根据需要自动获取更多页面。
  await foreach (var entry in rateLimits.Paginate())
  {
      var models = entry.Models is null ? "" : $" ({string.Join(", ", entry.Models)})";
      Console.WriteLine($"{entry.Group.Type.Raw()}{models}");
      foreach (var limit in entry.Limits)
      {
          Console.WriteLine($"  {limit.Type}: {limit.Value}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rateLimits := client.Organization.Workspaces.RateLimits.ListAutoPaging(
  	context.Background(),
  	"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  	anthropic.OrganizationWorkspaceRateLimitListParams{},
  )

  // 根据需要自动获取更多页面。
  for rateLimits.Next() {
  	entry := rateLimits.Current()
  	models := ""
  	if len(entry.Models) > 0 {
  		models = fmt.Sprintf(" (%s)", strings.Join(entry.Models, ", "))
  	}
  	fmt.Printf("%s%s\n", entry.Group.Type, models)
  	for _, limit := range entry.Limits {
  		fmt.Printf("  %s: %d\n", limit.Type, limit.Value)
  	}
  }
  if err := rateLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var rateLimits = client.organization().workspaces().rateLimits()
      .list("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ");

  // 根据需要自动获取更多页面。
  for (var entry : rateLimits.autoPager()) {
      var models = entry.models()
          .map(modelIds -> " (" + String.join(", ", modelIds) + ")")
          .orElse("");
      IO.println(entry.group().type().asString() + models);
      for (var limit : entry.limits()) {
          IO.println("  " + limit.type() + ": " + limit.value());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rateLimits = $client->organization->workspaces->rateLimits->list(
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
  );

  // 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  foreach ($rateLimits->data as $entry) {
      $models = $entry->models ? ' (' . implode(', ', $entry->models) . ')' : '';
      echo "{$entry->group->type}{$models}\n";
      foreach ($entry->limits as $limit) {
          echo "  {$limit->type}: {$limit->value}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace_id = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  rate_limits = client.organization.workspaces.rate_limits.list(workspace_id)

  # 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  rate_limits.data.each do |entry|
    models = entry.models ? " (#{entry.models.join(", ")})" : ""
    puts "#{entry.group.type}#{models}"
    entry.limits.each do |limit|
      puts "  #{limit.type}: #{limit.value}"
    end
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "workspace_id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      "rate_limit_id": "rlt_01NwFawrTvAijwqn8AjKVBPS",
      "type": "workspace_rate_limit",
      "group": {
        "type": "model_group",
        "id": "rlg_01Hq7YkP3mZ9dTwRx4cVbN2s",
        "display_name": "Claude Opus 5.5"
      },
      "models": ["claude-opus-5-5"],
      "limits": [
        {
          "type": "requests_per_minute",
          "value": 1000,
          "org_limit": 4000,
          "source": { "type": "workspace" }
        },
        {
          "type": "input_tokens_per_minute",
          "value": 500000,
          "org_limit": 10000000,
          "source": { "type": "workspace" }
        }
      ]
    },
    {
      "workspace_id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      "rate_limit_id": "rlt_016qriyigbdebqkzhMujdHkX",
      "type": "workspace_rate_limit",
      "group": {
        "type": "model_group",
        "id": "rlg_01Kd5wMv8nSq2LcXy6tRfJ4b",
        "display_name": "Claude Opus 4.x"
      },
      "models": [
        "claude-opus-4-5",
        "claude-opus-4-5-20251101",
        "claude-opus-4-6",
        "claude-opus-4-7",
        "claude-opus-4-8"
      ],
      "limits": [
        {
          "type": "requests_per_minute",
          "value": 1000,
          "org_limit": 4000,
          "source": { "type": "workspace" }
        },
        {
          "type": "input_tokens_per_minute",
          "value": 500000,
          "org_limit": 10000000,
          "source": { "type": "workspace" }
        }
      ]
    }
  ],
  "next_page": null
}
```

### 包含继承的值

若要在一次请求中读取工作区的适用限制，请添加可选的 `include_inherited` 查询参数。其默认值为 `false`；非布尔值会返回 400 错误。使用 `include_inherited=true` 时：

* 对于每个适用于该工作区且具有组织级值的组，`data` 中都有一个条目，即使工作区没有覆盖其中任何内容。当工作区可以访问某个模型组中的至少一个模型时，该模型组即适用。
* 每个条目的 `limits[]` 列出组织在该组上具有值的每种限制器类型，以及工作区覆盖的任何类型。每种类型只出现一次：存储了工作区覆盖设置时使用该覆盖值，否则使用组织的值，因此一个条目可能混合两者。
* 每个值的 `source` 用于区分两者：`{"type": "workspace"}` 表示存储的覆盖设置，`{"type": "organization"}` 表示继承的值，此时 `value` 等于 `org_limit`。

<CodeGroup>
  ```bash cURL
  # 返回一页；省略 `limit` 时，该页包含所有条目。
  curl "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/rate_limits?include_inherited=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 根据需要自动获取更多页面。
  ant organization:workspaces:rate-limits list \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --include-inherited
  ```

  ```python Python
  client = anthropic.Anthropic()

  rate_limits = client.organization.workspaces.rate_limits.list(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      include_inherited=True,
  )

  # 根据需要自动获取更多页面。
  for entry in rate_limits:
      models = f" ({', '.join(entry.models)})" if entry.models else ""
      print(f"{entry.group.type}{models}")
      for limit in entry.limits:
          print(f"  {limit.type}: {limit.value} ({limit.source.type})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rateLimits = await client.organization.workspaces.rateLimits.list(
    "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    { include_inherited: true }
  );

  // 根据需要自动获取更多页面。
  for await (const entry of rateLimits) {
    const models = entry.models ? ` (${entry.models.join(", ")})` : "";
    console.log(`${entry.group.type}${models}`);
    for (const limit of entry.limits) {
      console.log(`  ${limit.type}: ${limit.value} (${limit.source.type})`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var rateLimits = await client.Organization.Workspaces.RateLimits.List(
      "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      new() { IncludeInherited = true }
  );

  // 根据需要自动获取更多页面。
  await foreach (var entry in rateLimits.Paginate())
  {
      var models = entry.Models is null ? "" : $" ({string.Join(", ", entry.Models)})";
      Console.WriteLine($"{entry.Group.Type.Raw()}{models}");
      foreach (var limit in entry.Limits)
      {
          Console.WriteLine($"  {limit.Type}: {limit.Value} ({limit.Source.Type.Raw()})");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rateLimits := client.Organization.Workspaces.RateLimits.ListAutoPaging(
  	context.Background(),
  	"wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  	anthropic.OrganizationWorkspaceRateLimitListParams{
  		IncludeInherited: anthropic.Bool(true),
  	},
  )

  // 根据需要自动获取更多页面。
  for rateLimits.Next() {
  	entry := rateLimits.Current()
  	models := ""
  	if len(entry.Models) > 0 {
  		models = fmt.Sprintf(" (%s)", strings.Join(entry.Models, ", "))
  	}
  	fmt.Printf("%s%s\n", entry.Group.Type, models)
  	for _, limit := range entry.Limits {
  		fmt.Printf("  %s: %d (%s)\n", limit.Type, limit.Value, limit.Source.Type)
  	}
  }
  if err := rateLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.ratelimits.RateLimitListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RateLimitListParams.builder()
          .includeInherited(true)
          .build();
      var rateLimits = client.organization().workspaces().rateLimits()
          .list("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ", params);

      // 根据需要自动获取更多页面。
      for (var entry : rateLimits.autoPager()) {
          var models = entry.models()
              .map(modelIds -> " (" + String.join(", ", modelIds) + ")")
              .orElse("");
          IO.println(entry.group().type().asString() + models);
          for (var limit : entry.limits()) {
              IO.println("  " + limit.type() + ": " + limit.value()
                  + " (" + limit.source().type().asString() + ")");
          }
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $rateLimits = $client->organization->workspaces->rateLimits->list(
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
      includeInherited: true,
  );

  // 仅读取第一页；省略 `limit` 时，第一页包含所有条目。
  foreach ($rateLimits->data as $entry) {
      $models = $entry->models ? ' (' . implode(', ', $entry->models) . ')' : '';
      echo "{$entry->group->type}{$models}\n";
      foreach ($entry->limits as $limit) {
          echo "  {$limit->type}: {$limit->value} ({$limit->source->type})\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace_id = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ"
  rate_limits = client.organization.workspaces.rate_limits.list(workspace_id, include_inherited: true)

  # 仅读取第一页；省略 `limit` 时，第一页即包含全部条目。
  rate_limits.data.each do |entry|
    models = entry.models ? " (#{entry.models.join(", ")})" : ""
    puts "#{entry.group.type}#{models}"
    entry.limits.each do |limit|
      puts "  #{limit.type}: #{limit.value} (#{limit.source.type})"
    end
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "workspace_id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      "rate_limit_id": "rlt_01NwFawrTvAijwqn8AjKVBPS",
      "type": "workspace_rate_limit",
      "group": {
        "type": "model_group",
        "id": "rlg_01Hq7YkP3mZ9dTwRx4cVbN2s",
        "display_name": "Claude Opus 5.5"
      },
      "models": ["claude-opus-5-5"],
      "limits": [
        {
          "type": "requests_per_minute",
          "value": 1000,
          "org_limit": 4000,
          "source": { "type": "workspace" }
        },
        {
          "type": "input_tokens_per_minute",
          "value": 500000,
          "org_limit": 10000000,
          "source": { "type": "workspace" }
        },
        {
          "type": "output_tokens_per_minute",
          "value": 800000,
          "org_limit": 800000,
          "source": { "type": "organization" }
        }
      ]
    },
    {
      "workspace_id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      "rate_limit_id": "rlt_016qriyigbdebqkzhMujdHkX",
      "type": "workspace_rate_limit",
      "group": {
        "type": "model_group",
        "id": "rlg_01Kd5wMv8nSq2LcXy6tRfJ4b",
        "display_name": "Claude Opus 4.x"
      },
      "models": [
        "claude-opus-4-5",
        "claude-opus-4-5-20251101",
        "claude-opus-4-6",
        "claude-opus-4-7",
        "claude-opus-4-8"
      ],
      "limits": [
        {
          "type": "requests_per_minute",
          "value": 1000,
          "org_limit": 4000,
          "source": { "type": "workspace" }
        },
        {
          "type": "input_tokens_per_minute",
          "value": 500000,
          "org_limit": 10000000,
          "source": { "type": "workspace" }
        },
        {
          "type": "output_tokens_per_minute",
          "value": 800000,
          "org_limit": 800000,
          "source": { "type": "organization" }
        }
      ]
    },
    {
      "workspace_id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      "rate_limit_id": "rlt_01EyY8kq14eviKJPgepbgBH4",
      "type": "workspace_rate_limit",
      "group": { "type": "batch", "id": "rlg_01Wn3pBz6kCg9vHtQ7mLxD5a" },
      "models": null,
      "limits": [
        {
          "type": "enqueued_batch_requests",
          "value": 500000,
          "org_limit": 500000,
          "source": { "type": "organization" }
        }
      ]
    }
  ],
  "next_page": null
}
```

### 设置工作区速率限制

要为某个 rate limit group（速率限制组）设置或清除工作区的速率限制，请向 `/v1/organizations/workspaces/{workspace_id}/rate_limits/{rate_limit_id}` 发送 `POST` 请求。对于 `rate_limit_id`，请传入组织端点中该组条目的 `id`，或工作区列表端点中该组条目的 `rate_limit_id`。要查找工作区尚未覆盖的组的 ID，请使用 `include_inherited=true` 调用工作区列表端点，该端点会返回您可以为其设置限制的组。这些 ID 在不同组织之间有所不同，因此请在您管理的每个组织中分别查找。

请求体包含一个 `limits` 数组，其中有一个或多个 `{type, value}` 条目，每个条目对应不同的限制器类型：

* 当 `value` 为零或更大的整数时，会为该限制器类型设置工作区的覆盖值。
* 当 `value` 为 `null` 时，会清除覆盖值，使工作区重新继承组织的值。清除没有覆盖值的限制器类型不会产生任何更改。
* 请求中未指定的限制器类型保持其当前值。

您可以设置以下限制器类型：`requests_per_minute`、`input_tokens_per_minute`、`output_tokens_per_minute`、`input_tokens_per_minute_cache_aware`、`tokens_per_minute`、`tokens_per_day`、`tool_uses_per_second`，以及在列表端点为您的组织显示时的[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)限制 `fast_itpmca`（每分钟输入令牌数，缓存感知）和 `fast_otpm`（每分钟输出令牌数）。一个组接受其中哪些类型取决于其 `group.type`：

* `model_group`：除 `tool_uses_per_second` 之外的所有类型
* `web_search`：仅 `tool_uses_per_second`
* `batch`：仅 `requests_per_minute`

您可以在这些组中的任何一个上清除前面列出的任何限制器类型。列表端点可能会返回您无法设置的限制器类型，例如 `enqueued_batch_requests`；发送此类类型会返回 400 错误。

API 会存储高于同一限制器类型组织限制的值，但会执行两者中较低的那个。响应中的 `org_limit` 显示组织的值。

更改会在大约一小时内生效。在更改生效之前，工作区的请求仍受先前值的限制。

API 会在进行任何更改之前检查整个请求，因此无效条目会导致请求失败并返回 400 错误，且不会更改任何内容。如果请求通过验证，API 随后会逐一应用各个条目。如果某个条目应用失败，请求会返回错误，而在其之前的条目将保持已应用状态。每个条目设置或清除的是一个绝对值，因此您可以安全地再次发送相同的请求。重新发送会替换其他人在此期间对该请求所指定的限制器类型所做的任何更改。

<Note>
  [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 上不支持设置工作区速率限制。
</Note>

将工作区在 Claude Opus 5.5 组的每分钟请求数提高到 2,000，并清除其每分钟输入令牌数覆盖值：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/workspaces/wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ/rate_limits/rlt_01NwFawrTvAijwqn8AjKVBPS" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "limits": [
        {"type": "requests_per_minute", "value": 2000},
        {"type": "input_tokens_per_minute", "value": null}
      ]
    }'
  ```

  ```bash CLI
  ant organization:workspaces:rate-limits update \
    --workspace-id wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ \
    --rate-limit-id rlt_01NwFawrTvAijwqn8AjKVBPS <<'YAML'
  limits:
    - type: requests_per_minute
      value: 2000
    - type: input_tokens_per_minute
      value: null
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  entry = client.organization.workspaces.rate_limits.update(
      "rlt_01NwFawrTvAijwqn8AjKVBPS",
      workspace_id="wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      limits=[
          {"type": "requests_per_minute", "value": 2000},
          {"type": "input_tokens_per_minute", "value": None},
      ],
  )

  for limit in entry.limits:
      print(f"{limit.type}: {limit.value}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const entry = await client.organization.workspaces.rateLimits.update(
    "rlt_01NwFawrTvAijwqn8AjKVBPS",
    {
      workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
      limits: [
        { type: "requests_per_minute", value: 2000 },
        { type: "input_tokens_per_minute", value: null }
      ]
    }
  );

  for (const limit of entry.limits) {
    console.log(`${limit.type}: ${limit.value}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var entry = await client.Organization.Workspaces.RateLimits.Update(
      "rlt_01NwFawrTvAijwqn8AjKVBPS",
      new()
      {
          WorkspaceID = "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
          Limits =
          [
              new() { Type = "requests_per_minute", Value = 2000 },
              new() { Type = "input_tokens_per_minute", Value = null },
          ],
      }
  );

  foreach (var limit in entry.Limits)
  {
      Console.WriteLine($"{limit.Type}: {limit.Value}");
  }
  ```

  ```go Go
  import (
  	"context"
  	"fmt"
  	"log"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"github.com/anthropics/anthropic-sdk-go/v2/packages/param"
  )

  func main() {
  	client := anthropic.NewClient()

  	entry, err := client.Organization.Workspaces.RateLimits.Update(
  		context.Background(),
  		"rlt_01NwFawrTvAijwqn8AjKVBPS",
  		anthropic.OrganizationWorkspaceRateLimitUpdateParams{
  			WorkspaceID: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  			Limits: []anthropic.WorkspaceRateLimitUpdateValueParam{
  				{Type: "requests_per_minute", Value: anthropic.Int(2000)},
  				{Type: "input_tokens_per_minute", Value: param.Null[int64]()},
  			},
  		},
  	)
  	if err != nil {
  		log.Fatal(err)
  	}

  	for _, limit := range entry.Limits {
  		fmt.Printf("%s: %d\n", limit.Type, limit.Value)
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.organization.workspaces.ratelimits.RateLimitUpdateParams;
  import com.anthropic.models.organization.workspaces.ratelimits.WorkspaceRateLimitType;
  import com.anthropic.models.organization.workspaces.ratelimits.WorkspaceRateLimitUpdateValue;
  import java.util.Optional;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var entry = client.organization().workspaces().rateLimits().update(
          "rlt_01NwFawrTvAijwqn8AjKVBPS",
          RateLimitUpdateParams.builder()
              .workspaceId("wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ")
              .addLimit(WorkspaceRateLimitUpdateValue.builder()
                  .type(WorkspaceRateLimitType.REQUESTS_PER_MINUTE)
                  .value(2000L)
                  .build())
              .addLimit(WorkspaceRateLimitUpdateValue.builder()
                  .type(WorkspaceRateLimitType.INPUT_TOKENS_PER_MINUTE)
                  .value(Optional.empty())
                  .build())
              .build()
      );

      for (var limit : entry.limits()) {
          IO.println(limit.type() + ": " + limit.value());
      }
  }
  ```

  ```php PHP
  use Anthropic\Organization\Workspaces\RateLimits\WorkspaceRateLimitUpdateValue;

  $client = new Client();

  $entry = $client->organization->workspaces->rateLimits->update(
      'rlt_01NwFawrTvAijwqn8AjKVBPS',
      workspaceID: 'wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ',
      limits: [
          WorkspaceRateLimitUpdateValue::with(type: 'requests_per_minute', value: 2000),
          WorkspaceRateLimitUpdateValue::with(type: 'input_tokens_per_minute', value: null),
      ],
  );

  foreach ($entry->limits as $limit) {
      echo "{$limit->type}: {$limit->value}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  entry = client.organization.workspaces.rate_limits.update(
    "rlt_01NwFawrTvAijwqn8AjKVBPS",
    workspace_id: "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
    limits: [
      { type: :requests_per_minute, value: 2000 },
      { type: :input_tokens_per_minute, value: nil }
    ]
  )

  entry.limits.each do |limit|
    puts "#{limit.type}: #{limit.value}"
  end
  ```
</CodeGroup>

响应是该组的条目，其形式与工作区列表端点 `data` 列表中的一个条目相同。其 `limits` 列出更改后工作区在该组上的覆盖值：已清除的限制器类型不会列出，当没有剩余覆盖值时 `limits` 为空。即使紧随其后发出的列表请求尚未显示这些更改，响应中也会包含该请求的更改。

```json
{
  "workspace_id": "wrkspc_01JwQvzr7rXLA5AGx3HKfFUJ",
  "rate_limit_id": "rlt_01NwFawrTvAijwqn8AjKVBPS",
  "type": "workspace_rate_limit",
  "group": {
    "type": "model_group",
    "id": "rlg_01Hq7YkP3mZ9dTwRx4cVbN2s",
    "display_name": "Claude Opus 5.5"
  },
  "models": ["claude-opus-5-5"],
  "limits": [
    {
      "type": "requests_per_minute",
      "value": 2000,
      "org_limit": 4000,
      "source": { "type": "workspace" }
    }
  ]
}
```

对于无效请求或已归档的工作区，端点返回 400 错误。当找不到工作区，或 `rate_limit_id` 指定了工作区无法使用的组时，返回 404 错误。

## 按组类型筛选

两个列表端点都接受一个可选的 `group_type` 查询参数，用于将响应限制为单个类别：

<CodeGroup>
  ```bash cURL
  # 返回一页结果；若省略 `limit`，该页将包含所有条目。
  curl "https://api.anthropic.com/v1/organizations/rate_limits?group_type=batch" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  # 根据需要自动获取更多页面。
  ant organization:rate-limits list --group-type batch
  ```

  ```python Python
  client = anthropic.Anthropic()

  rate_limits = client.organization.rate_limits.list(group_type="batch")

  # 根据需要自动获取更多页面。
  for entry in rate_limits:
      models = f" ({', '.join(entry.models)})" if entry.models else ""
      print(f"{entry.group.type}{models}")
      for limit in entry.limits:
          print(f"  {limit.type}: {limit.value}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const rateLimits = await client.organization.rateLimits.list({ group_type: "batch" });

  // 根据需要自动获取更多页面。
  for await (const entry of rateLimits) {
    const models = entry.models ? ` (${entry.models.join(", ")})` : "";
    console.log(`${entry.group.type}${models}`);
    for (const limit of entry.limits) {
      console.log(`  ${limit.type}: ${limit.value}`);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Organization.RateLimits;

  AnthropicClient client = new();

  var rateLimits = await client.Organization.RateLimits.List(new()
  {
      GroupType = GroupType.Batch
  });

  // 根据需要自动获取更多页面。
  await foreach (var entry in rateLimits.Paginate())
  {
      var models = entry.Models is null ? "" : $" ({string.Join(", ", entry.Models)})";
      Console.WriteLine($"{entry.Group.Type.Raw()}{models}");
      foreach (var limit in entry.Limits)
      {
          Console.WriteLine($"  {limit.Type}: {limit.Value}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  rateLimits := client.Organization.RateLimits.ListAutoPaging(context.Background(), anthropic.OrganizationRateLimitListParams{
  	GroupType: anthropic.OrganizationRateLimitListParamsGroupTypeBatch,
  })

  // 根据需要自动获取更多页面。
  for rateLimits.Next() {
  	entry := rateLimits.Current()
  	models := ""
  	if len(entry.Models) > 0 {
  		models = fmt.Sprintf(" (%s)", strings.Join(entry.Models, ", "))
  	}
  	fmt.Printf("%s%s\n", entry.Group.Type, models)
  	for _, limit := range entry.Limits {
  		fmt.Printf("  %s: %d\n", limit.Type, limit.Value)
  	}
  }
  if err := rateLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.organization.ratelimits.RateLimitListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = RateLimitListParams.builder()
          .groupType(RateLimitListParams.GroupType.BATCH)
          .build();
      var rateLimits = client.organization().rateLimits().list(params);

      // 根据需要自动获取更多页面。
      for (var entry : rateLimits.autoPager()) {
          var models = entry.models()
              .map(modelIds -> " (" + String.join(", ", modelIds) + ")")
              .orElse("");
          IO.println(entry.group().type().asString() + models);
          for (var limit : entry.limits()) {
              IO.println("  " + limit.type() + ": " + limit.value());
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Organization\RateLimits\RateLimitListParams\GroupType;
  // ...

  $client = new Client();

  $rateLimits = $client->organization->rateLimits->list(
      groupType: GroupType::BATCH,
  );

  // 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  foreach ($rateLimits->data as $entry) {
      $models = $entry->models ? ' (' . implode(', ', $entry->models) . ')' : '';
      echo "{$entry->group->type}{$models}\n";
      foreach ($entry->limits as $limit) {
          echo "  {$limit->type}: {$limit->value}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  rate_limits = client.organization.rate_limits.list(group_type: :batch)

  # 仅读取第一页；省略 `limit` 时，第一页即包含所有条目。
  rate_limits.data.each do |entry|
    models = entry.models ? " (#{entry.models.join(", ")})" : ""
    puts "#{entry.group.type}#{models}"
    entry.limits.each do |limit|
      puts "  #{limit.type}: #{limit.value}"
    end
  end
  ```
</CodeGroup>

有效值为 `model_group`、`batch`、`token_count`、`files`、`skills` 和 `web_search`。`token_count`、`files` 和 `skills` 值会返回空的 `data` 列表，因为两个列表端点都不返回 Token Counting API、Files API 或 Skills API 的限制。

## 分页

两个列表端点都接受可选的 `limit` 和 `page` 查询参数，并返回 `next_page` 字段。`limit` 的取值范围为 1 到 1,000。如果省略该参数，响应将包含所有剩余条目，且 `next_page` 为 `null`。如果 `limit` 截断了列表，`next_page` 会包含一个游标：将其作为 `page` 传入，并使用相同的筛选条件，即可获取后续条目。请基于 `next_page` 进行循环，以便您的客户端获取所有条目。

## 常见问题

### 哪些模型字符串会出现在 `models` 列表中？

计入该组的每个模型 ID 和别名，包括带日期的 ID（例如 `claude-sonnet-4-5-20250929`）和不带日期的别名（例如 `claude-sonnet-4-5`）。查找您传递给 Messages API 的任意模型字符串，您都会在恰好一个 `model_group` 条目中找到它。

该列表涵盖您的组织有权访问的模型，因此不同组织之间可能有所不同。

### 如果工作区响应中缺少某个组，这意味着什么？

默认情况下，工作区响应仅列出覆盖设置，因此缺失的组没有工作区覆盖设置，并继承组织级限制。传入 `include_inherited=true` 可在工作区端点上查看继承的值，或者查询组织端点。使用 `include_inherited=true` 时，只有当某个组不适用于该工作区或组织没有为其设置限制时，该组才会缺失。

### 您可以使用此 API 更新速率限制吗？

**对于工作区：** 可以。[设置工作区速率限制](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/rate_limits/update)端点可为某个速率限制组设置或清除工作区的覆盖值。请将[组织端点响应](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api#list-all-organization-rate-limits)中匹配条目的顶层 `id` 作为 `rate_limit_id` 传入，而不是其 `group` 内部的 `rlg_` `id`。工作区覆盖值与组织级限制中较低的那个生效，更改会在大约一小时内生效。有关请求体和示例，请参阅[设置工作区速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/rate-limits-api#set-workspace-rate-limits)。您还可以在 [Claude Console 中的速率限制页面](https://platform.claude.com/usage/limits)上设置工作区速率限制：从**工作区**下拉菜单中选择工作区，然后点击模型旁边的**编辑**。

**对于组织：** 不可以。如需申请提高限制，请参阅[申请更高的限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#requesting-higher-limits)。

## 另请参阅

* [速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)
* [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)
* [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/organization)
* [工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)
* [用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)
