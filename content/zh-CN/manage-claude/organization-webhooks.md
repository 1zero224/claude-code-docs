---
title: 组织 webhook
url: https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks
description: 注册 webhook 端点，以接收有关您组织的用户配置文件所受执行措施的事件。
featureMetadata:
  status: beta
  betaHeader: webhooks-2026-10-07
  supportedPlatforms:
    Claude API: beta
---

当 Anthropic 限制或封禁您组织的某个[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)时，以及当该限制或封禁结束时，组织 webhook 端点都会收到一个事件。您可以通过 Admin API 在 `/v1/organizations/webhooks` 下管理它。

<Note>
  事件投递尚未上线：Anthropic 不会发送这两种事件，因此您注册的端点不会收到任何内容。您可以注册端点，并根据本页中的事件格式构建您的处理程序。
</Note>

您组织的任何成员都可以通过 API 列出和查看组织 webhook 端点，但只有具有管理员、所有者或主要所有者角色的成员才能创建、更改或删除它们，或替换其签名密钥。

## 开始之前

您需要以下内容：

* 一个位于 Claude API 上、已开启组织 webhook 且未启用 [HIPAA 就绪](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#hipaa-readiness)的组织。如需申请访问权限，请联系您的 Anthropic 客户团队。

* 以下凭据之一：

  * 一个 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。
  * 一个具有 `org:admin` 作用域的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。
  * 一个未限定于某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)。

## 事件类型

组织 webhook 端点接受两种事件类型。两者都与用户配置文件上的[执法措施](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#check-whether-anthropic-has-restricted-or-banned-a-profile)有关。

| 事件                                         | 触发条件                                           |
| ------------------------------------------ | ---------------------------------------------- |
| `user_profile.enforcement_action_opened`   | Anthropic 对某个用户配置文件开启一项执行措施。                   |
| `user_profile.enforcement_action_resolved` | 一项执行措施结束。Anthropic 解决了它，或者其 `expires_at` 时间已过。 |

删除配置文件会结束其所有当前生效的执行措施，但不会发送 `user_profile.enforcement_action_resolved` 事件。

### 在收到第一个事件之前

在任何 `anthropic-beta` 标头包含 `user-profiles-2026-09-04` 或更早值的请求中，响应会省略 `enforcement_status` 和 `enforcement_actions` 字段，并且 `enforcement_status[]` 列表筛选器会返回 `400`。要获取这些字段，请从您的请求中移除该值。

如果某项执法措施是在 Anthropic 为您的组织开启事件投递之前，或在您注册端点之前开启的，您不会收到针对它的 `user_profile.enforcement_action_opened` 事件。该措施仍会显示在配置文件上。要查找这些措施，请使用 `enforcement_status[]=restricted` 和 `enforcement_status[]=banned` [列出您的配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/list)。在列表结束之前，响应返回的配置文件数量可能少于 `limit`，甚至为零，因此请持续请求分页，直到 `next_page` 为 `null`。

### 事件包含的内容

事件携带的是 ID，而不是措施或配置文件本身。`data.id` 是配置文件的 ID，`data.enforcement_action_id` 是措施的 ID，以 `enac_` 开头：

```json
{
  "type": "event",
  "id": "whe_4f2a9c1e7b3d5a8f0e6c2b9d1a7f3e5c",
  "created_at": "2026-10-15T17:02:41Z",
  "data": {
    "type": "user_profile.enforcement_action_opened",
    "id": "uprof_011CZkZCu8hGbp5mYRQgUmz9",
    "enforcement_action_id": "enac_011CZkZEq4TnWb7xLfRgPm2v",
    "organization_id": "org_011CZkZBj9RsYd3wNcHvKq6t"
  }
}
```

要查看发生了什么变化，请使用 `data.id`，通过 `GET /v1/user_profiles/{id}` (python, ruby: `client.user_profiles.retrieve()`; typescript: `client.userProfiles.retrieve()`; go: `client.UserProfiles.Get()`; java: `client.userProfiles().retrieve()`; csharp: `client.UserProfiles.Retrieve()`; php: `$client->userProfiles->retrieve()`; cli: `ant user-profiles retrieve`) [检索该配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/retrieve)。其 `enforcement_actions` 列表显示其当前生效的措施。每项措施都有一个 `type`（`restriction` 或 `ban`）、一个 `id`、一个 `reason`、一个 `opened_at` 时间，以及一个 `expires_at` 时间或 `null`。`enforcement_status` 对它们进行汇总：有封禁生效时为 `banned`，有任何其他措施生效时为 `restricted`，没有任何措施生效时为 `unrestricted`。请将任何其他值视为 `restricted`。

Anthropic 仅在配置文件显示出变化之后才发送事件，因此您检索到的配置文件会显示该变化或更晚的变化。事件可能延迟到达或乱序到达，因此请以配置文件的当前状态为准，而不是以事件到达的顺序为准。在 `user_profile.enforcement_action_resolved` 之后，已解决的措施将不再出现在列表中。`404` 表示该配置文件已被删除。

## 端点字段

组织 webhook 端点具有以下字段：

| 字段                | 类型               | 描述                                                                                                                 |
| ----------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------ |
| `id`              | string           | 端点的 ID，以 `wep_` 开头。由 Anthropic 设置。                                                                                 |
| `url`             | string           | Anthropic 发送事件的目标地址。必须是 443 端口上的 HTTPS，并使用公共域名，而不是 IP 地址。您组织中的两个已启用的组织 webhook 端点不能共用同一个 URL。                      |
| `enabled_events`  | array of strings | 此端点接收的[事件类型](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#event-types)。创建请求必须至少列出一种。 |
| `name`            | string           | 供您自己使用的可选标签。                                                                                                       |
| `description`     | string           | 供您自己使用的可选描述。                                                                                                       |
| `state`           | string           | `enabled` 或 `disabled`。                                                                                            |
| `disabled_reason` | string           | Anthropic 禁用该端点的原因。否则为空。                                                                                           |
| `created_at`      | string           | 端点的创建时间。由 Anthropic 设置。                                                                                            |
| `updated_at`      | string           | 端点的最后更改时间。由 Anthropic 设置。                                                                                          |
| `signing_secret`  | string           | 一个以 `whsec_` 为前缀的密钥，随端点一起创建。只有创建端点或替换密钥的响应才会包含它，因此请妥善保存。                                                           |

```json
{
  "id": "wep_011CZkZDf8HnUq2mTxRbVn4s",
  "url": "https://example.com/anthropic-webhooks",
  "name": "Enforcement actions",
  "description": "",
  "enabled_events": [
    "user_profile.enforcement_action_opened",
    "user_profile.enforcement_action_resolved"
  ],
  "state": "enabled",
  "disabled_reason": "",
  "created_at": "2026-10-20T19:00:00Z",
  "updated_at": "2026-10-20T19:00:00Z"
}
```

一个组织最多可以拥有 32 个组织 webhook 端点。已禁用的 webhook 端点仍计入此总数。

## 投递

投递遵循 [Standard Webhooks](https://www.standardwebhooks.com/) 规范。每次投递都携带 `webhook-id`、`webhook-timestamp` 和 `webhook-signature` 标头。在信任某个事件之前，请使用任意 Standard Webhooks 库，通过端点的签名密钥验证其签名。

请在 5 秒内返回任意 `2xx` 状态码，并在之后再执行耗时的工作。任何其他响应或无响应都会被视为一次失败的尝试。

* **重复：** 端点可能会多次收到同一事件，其事件 `id` 相同（与 `webhook-id` 标头的值相同）。请据此进行去重。

* **重试：** 如果投递失败，Anthropic 会在接下来的几分钟内重试几次。如果所有尝试都失败，Anthropic 会丢弃该事件。要补齐遗漏，请按执行状态列出您的配置文件，如[在收到第一个事件之前](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#before-your-first-event)中所述。

* **自动禁用：** 在以下情况下，Anthropic 会禁用端点：

  * 它返回重定向（`3xx`）。
  * 其 URL 解析为非公共 IP 地址。
  * 其投递持续失败。

  对于已禁用的端点，Anthropic 会跳过事件，并且之后也不会再发送。

## 管理端点

通过 Admin API，您可以发出以下请求：

| 任务                                                          | 请求                                                                                                                                                                                   |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 列出您组织的端点                                                    | [`GET /v1/organizations/webhooks`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/list)                                                                       |
| 创建端点                                                        | [`POST /v1/organizations/webhooks`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/create)                                                                    |
| 获取单个端点                                                      | [`GET /v1/organizations/webhooks/{webhook_endpoint_id}`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/retrieve)                                             |
| 更新端点的 `url`、`name`、`description`、`enabled_events` 或 `state` | [`POST /v1/organizations/webhooks/{webhook_endpoint_id}`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/update)                                              |
| 删除端点                                                        | [`DELETE /v1/organizations/webhooks/{webhook_endpoint_id}`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/delete)                                            |
| 替换签名密钥                                                      | [`POST /v1/organizations/webhooks/{webhook_endpoint_id}/regenerate_signing_secret`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/regenerate_signing_secret) |

在更新中，`enabled_events` 会替换端点的整个事件类型列表。空列表不会清空该列表：端点会保留其原有的事件类型。要停止投递，请发送 `"state": "disabled"` 或删除该端点。

要重新启用端点（包括在 Anthropic 禁用它之后），请在更新请求中发送 `"state": "enabled"`。

替换签名密钥后，旧密钥会立即失效。此后的投递（包括对先前事件的重试）都将使用新密钥签名，因此请立即更新您的接收端。

请在每个请求中发送 `anthropic-beta: webhooks-2026-10-07` 标头。SDK 和 CLI 会为您发送它。此示例使用来自 `ANTHROPIC_API_KEY` 的 Admin API 密钥列出您组织的端点。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/webhooks" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: webhooks-2026-10-07"
  ```

  ```bash CLI
  ant beta:organization:webhooks list
  ```

  ```python Python
  client = anthropic.Anthropic()

  endpoints = client.beta.organization.webhooks.list()

  for endpoint in endpoints:
      print(f"{endpoint.id}: {endpoint.url} ({endpoint.state})")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const endpoints = await client.beta.organization.webhooks.list();

  for await (const endpoint of endpoints) {
    console.log(`${endpoint.id}: ${endpoint.url} (${endpoint.state})`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.Webhooks.List();

  await foreach (var endpoint in page.Paginate())
  {
      Console.WriteLine($"{endpoint.ID}: {endpoint.Url} ({endpoint.State.Raw()})");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  endpoints := client.Beta.Organization.Webhooks.ListAutoPaging(context.Background(), anthropic.BetaOrganizationWebhookListParams{})

  for endpoints.Next() {
  	endpoint := endpoints.Current()
  	fmt.Printf("%s: %s (%s)\n", endpoint.ID, endpoint.URL, endpoint.State)
  }
  if err := endpoints.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var endpoints = client.beta().organization().webhooks().list();

  for (var endpoint : endpoints.autoPager()) {
      IO.println(endpoint.id() + ": " + endpoint.url() + " (" + endpoint.state().asString() + ")");
  }
  ```

  ```php PHP
  $client = new Client();

  $endpoints = $client->beta->organization->webhooks->list();

  foreach ($endpoints->pagingEachItem() as $endpoint) {
      echo "{$endpoint->id}: {$endpoint->url} ({$endpoint->state})\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  endpoints = client.beta.organization.webhooks.list

  endpoints.auto_paging_each do |endpoint|
    puts "#{endpoint.id}: #{endpoint.url} (#{endpoint.state})"
  end
  ```
</CodeGroup>

如果没有端点，API 会返回一个空列表：

```json
{
  "data": [],
  "has_more": false
}
```

列表请求默认最多返回 20 个端点。设置 `limit` 最多可获取 100 个。如果 `has_more` 为 `true`，请将响应中的 `last_id` 作为 `after_id` 传入以获取下一页。

此示例创建一个接收两种事件类型的端点：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/webhooks" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: webhooks-2026-10-07" \
    -H "content-type: application/json" \
    -d '{
      "url": "https://example.com/anthropic-webhooks",
      "name": "Enforcement actions",
      "enabled_events": [
        "user_profile.enforcement_action_opened",
        "user_profile.enforcement_action_resolved"
      ]
    }'
  ```

  ```bash CLI
  ant beta:organization:webhooks create \
    --url https://example.com/anthropic-webhooks \
    --name "Enforcement actions" \
    --enabled-event user_profile.enforcement_action_opened \
    --enabled-event user_profile.enforcement_action_resolved
  ```

  ```python Python
  client = anthropic.Anthropic()

  endpoint = client.beta.organization.webhooks.create(
      url="https://example.com/anthropic-webhooks",
      name="Enforcement actions",
      enabled_events=[
          "user_profile.enforcement_action_opened",
          "user_profile.enforcement_action_resolved",
      ],
  )

  print(endpoint.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const endpoint = await client.beta.organization.webhooks.create({
    url: "https://example.com/anthropic-webhooks",
    name: "Enforcement actions",
    enabled_events: [
      "user_profile.enforcement_action_opened",
      "user_profile.enforcement_action_resolved"
    ]
  });

  console.log(endpoint.id);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Webhooks;

  AnthropicClient client = new();

  var endpoint = await client.Beta.Organization.Webhooks.Create(new WebhookCreateParams
  {
      UrlValue = "https://example.com/anthropic-webhooks",
      Name = "Enforcement actions",
      EnabledEvents =
      [
          EnabledEvent.UserProfileEnforcementActionOpened,
          EnabledEvent.UserProfileEnforcementActionResolved,
      ],
  });

  Console.WriteLine(endpoint.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  endpoint, err := client.Beta.Organization.Webhooks.New(context.Background(), anthropic.BetaOrganizationWebhookNewParams{
  	URL:  "https://example.com/anthropic-webhooks",
  	Name: anthropic.String("Enforcement actions"),
  	EnabledEvents: []string{
  		"user_profile.enforcement_action_opened",
  		"user_profile.enforcement_action_resolved",
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(endpoint.ID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.webhooks.WebhookCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = WebhookCreateParams.builder()
          .url("https://example.com/anthropic-webhooks")
          .name("Enforcement actions")
          .addEnabledEvent(WebhookCreateParams.EnabledEvent.USER_PROFILE_ENFORCEMENT_ACTION_OPENED)
          .addEnabledEvent(WebhookCreateParams.EnabledEvent.USER_PROFILE_ENFORCEMENT_ACTION_RESOLVED)
          .build();

      var endpoint = client.beta().organization().webhooks().create(params);

      IO.println(endpoint.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $endpoint = $client->beta->organization->webhooks->create(
      url: 'https://example.com/anthropic-webhooks',
      name: 'Enforcement actions',
      enabledEvents: [
          'user_profile.enforcement_action_opened',
          'user_profile.enforcement_action_resolved',
      ],
  );

  echo $endpoint->id, "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  endpoint = client.beta.organization.webhooks.create(
    url: "https://example.com/anthropic-webhooks",
    name: "Enforcement actions",
    enabled_events: [
      "user_profile.enforcement_action_opened",
      "user_profile.enforcement_action_resolved"
    ]
  )

  puts endpoint.id
  ```
</CodeGroup>

响应为新端点（如[端点字段](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#endpoint-fields)中所示），外加其 `signing_secret`。

## 错误

在以下情况下，请求会返回 `400`：

* 其 `url` 不符合[端点字段](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#endpoint-fields)中的规则，或者创建请求未列出任何事件类型。错误消息会说明问题所在。
* 您的组织没有默认工作区。
* 由于您组织的默认工作区使用了[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)，组织 webhook 对您的组织不可用。错误消息会说明这一点。
* 该请求会使您的组织拥有超过 32 个组织 webhook 端点。
* 它在 `enabled_events` 中放入了[事件类型](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#event-types)中两种类型以外的事件类型。错误消息会列出可接受的类型。

如果您账户的角色不包含请求所需的权限，请求会返回 `403`。错误会指明该权限。

在以下情况下，请求会返回 `404`：

* 它未发送 `anthropic-beta: webhooks-2026-10-07` 标头。
* Anthropic 尚未为您的组织开启组织 webhook。
* 您的组织已启用 HIPAA 就绪。
* 它使用了这些请求不接受的凭据类型。
* 其 `webhook_endpoint_id` 与您组织中的任何端点都不匹配。

在以下情况下，请求会返回 `409`：

* 它会使两个已启用的组织 webhook 端点共用同一个 URL。
* 另一个请求同时更改了您组织的端点。请重试该请求。
