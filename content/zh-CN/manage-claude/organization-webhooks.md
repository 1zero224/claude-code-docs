---
title: 组织 webhook
url: https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks
description: 注册 webhook 端点，以接收有关您组织的用户配置文件和客户管理的加密密钥的事件，并响应每个事件。
featureMetadata:
  status: beta
  betaHeader: webhooks-2026-10-07
  supportedPlatforms:
    Claude API: beta
---

组织 webhook 端点是 Anthropic 向其发送有关您组织的事件（例如某个加密密钥停止工作）的 URL。您可以使用本页选择所需的事件、注册端点、响应每个事件，并在事件未到达时补齐遗漏。您可以通过 Admin API 在 `/v1/organizations/webhooks` 下管理端点；对于某些事件类型，也可以[在 Claude Console 中](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#manage-endpoints)管理。

<Note>
  两种[用户配置文件事件](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#user-profile-events)的投递未上线：Anthropic 不会发送它们。您可以为它们注册端点，并根据本页中的事件格式构建您的处理程序。
</Note>

## 开始之前

您需要以下内容：

* 一个位于 Claude API 上、已开启组织 webhook 的组织。如果未开启，[请求会返回 `404`](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#errors)。以下情况中第一个符合您组织的情况适用：

  * 已启用 [HIPAA 就绪](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#hipaa-readiness)：无法使用组织 webhook。
  * 可以使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)：已拥有组织 webhook。
  * 任何其他组织：请联系您的 Anthropic 客户团队申请访问权限。

* 以下凭据之一：

  * 一个 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)。
  * 一个具有 `org:admin` 作用域的 [OAuth bearer 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#oauth-bearer-token)。
  * 一个未限定于某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)。

使用 Admin API 密钥，您可以发出[管理端点](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#manage-endpoints)中的所有请求。使用 `org:admin` 令牌，您可以发出相同的请求。如果该令牌绑定到您组织的[默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#how-workspaces-work)以外的工作区，您将无法发出其中任何请求。使用个人或服务账户密钥时，您可以发出的请求取决于该账户的[组织角色](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#organization-roles-and-permissions)：

* 列出或获取 webhook 端点：`developer` 或 `admin`
* 创建、更新或删除 webhook 端点，或替换签名密钥：`admin`

## 事件类型

端点会接收您在[创建或更新它](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#manage-endpoints)时在其 [`enabled_events`](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#endpoint-fields) 中列出的事件类型。本页涵盖以下几组事件类型：

* [用户配置文件事件](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#user-profile-events)：Anthropic 限制或封禁您的某个[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)，或者该限制或封禁结束。
* [加密密钥事件](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#encryption-key-events)：您的 “key management service”（密钥管理服务），即 KMS，持续拒绝您的某个客户管理的加密密钥，或者该密钥恢复正常工作。

有关其他事件类型，请参阅[创建端点的 API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/create)中的 `enabled_events`。

## 用户配置文件事件

以下事件类型与用户配置文件上的[执行措施](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#check-whether-anthropic-has-restricted-or-banned-a-profile)有关。Anthropic 不会发送它们。

| 事件 | 触发条件 |
| ----- | ----- |
| `user_profile.enforcement_action_opened` | Anthropic 对某个用户配置文件开启一项执行措施。 |
| `user_profile.enforcement_action_resolved` | 一项执行措施结束。Anthropic 解决了它，或者其 `expires_at` 时间已过。 |

删除配置文件会结束其所有当前生效的执行措施，但不会发送 `user_profile.enforcement_action_resolved` 事件。

### 在收到第一个事件之前

在任何 `anthropic-beta` 标头包含 `user-profiles-2026-09-04` 或更早的用户配置文件 beta 的请求中，响应会省略 `enforcement_status` 和 `enforcement_actions` 字段。在此类请求中，`enforcement_status[]` 列表筛选器会返回 `400`。要获取这些字段，请从您的请求中移除该 beta。

如果某项执法措施是在 Anthropic 为您的组织开启事件投递之前，或在您注册端点之前开启的，您不会收到针对它的 `user_profile.enforcement_action_opened` 事件。该措施仍会显示在配置文件上。要查找这些措施，请使用 `enforcement_status[]=restricted` 和 `enforcement_status[]=banned` [列出您的配置文件](https://platform.claude.com/docs/zh-CN/api/user_profiles/list)。在列表结束之前，响应返回的配置文件数量可能少于 `limit`，甚至为零，因此请持续请求分页，直到 `next_page` 为 `null`。

### 事件包含的内容

用户配置文件事件携带的是 ID，而不是措施或配置文件本身。`data.id` 是配置文件的 ID，`data.enforcement_action_id` 是措施的 ID，以 `enac_` 开头：

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

## 加密密钥事件

这些事件与您组织的[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)（CMEK）有关。Admin API 将此类密钥称为外部密钥（external key），因此事件类型以 `external_key` 开头：

| 事件 | 触发条件 |
| ----- | ----- |
| `external_key.failed` | 您的 KMS 持续拒绝 Anthropic 使用某个密钥，例如因为有人禁用了该密钥或移除了 Anthropic 对它的访问权限。 |
| `external_key.recovered` | 在 Anthropic 发送 `external_key.failed` 之后，Anthropic 使用该密钥时它再次正常工作。 |

只有真实的拒绝才会触发这些事件。没有测试事件。

当密钥失败或恢复时，Anthropic 通常还会向您组织中能够管理加密密钥的成员发送电子邮件。失败邮件会列出您的 KMS 最近拒绝过的工作区，但可能不会列出全部工作区。

该事件在 `data.id` 中指明密钥，其以 `ekey_` 开头：

```json
{
  "type": "event",
  "id": "whe_8c1d5e2a9f4b7036d1e8a5c2f9b40d73",
  "created_at": "2026-11-20T09:14:07.482913Z",
  "data": {
    "type": "external_key.failed",
    "id": "ekey_011CZkZFr7KpXc4nMgSvQh3w",
    "organization_id": "org_011CZkZBj9RsYd3wNcHvKq6t"
  }
}
```

* **时间：** `created_at` 是 Anthropic 首次尝试发送该事件的时间，而不是密钥失败或恢复的时间。它可能包含小数秒。
* **工作区：** 该事件不会指明工作区，因为多个工作区可以使用同一个密钥。

### 响应加密密钥事件

在您为任一事件返回 `2xx` 之后，请使用 `POST /v1/organizations/external_keys/{external_key_id}/validate` [验证该密钥](https://platform.claude.com/docs/zh-CN/api/organization/external_keys/validate)。使用 `data.id` 作为 ID，并使用 Admin API 密钥或 `org:admin` 令牌。您接下来要做什么取决于响应中的 `status`：

* **`failure`，且 `error` 消息提示重试：** 按照其提示操作，然后再次验证。
* **`failure`，且为任何其他消息：** 在 [CMEK 故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting)中查找修复方法。
* **`success`：** 请检查使用该密钥的每个工作区，因为验证最多只测试一个工作区。您的 KMS 可能会针对其他工作区拒绝 Anthropic，例如在[列出工作区的 AWS 密钥策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces)的情况下。

要检查这些工作区：

1. 使用 `include_default=true` [列出您的工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)。
2. 找出 `external_key_id` 中包含该密钥 ID 的工作区。
3. 使用可以在该工作区中操作的 API 密钥，在每个工作区中发送一个 Messages API 请求。

| 请求结果 | 对该工作区意味着什么 |
| ----- | ----- |
| [`cmek_key_disabled`](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#cmek-key-disabled) 或 [`cmek_key_network_blocked`](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#cmek-key-network-blocked) 错误 | 您的 KMS 拒绝了 Anthropic。 |
| [`cmek_key_unreachable`](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#cmek-key-unreachable) 错误 | Anthropic 无法连接到您的 KMS。 |
| 成功 | 该密钥可以正常工作，或者它停止工作还不到一小时。 |

在密钥停止工作后，[请求最多可能在 1 小时内仍然成功](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)，因此如果请求成功，请在 `external_key.failed` 一小时后再次发送。修复之后，错误可能会持续几分钟。

修复之后，请重复这些检查。不要等待 `external_key.recovered`，它可能不会到达。

### 加密密钥事件何时到达

如果使用该密钥的每个工作区每分钟大约收到一个 Messages API 请求，这些事件通常会在以下时间到达。流量越少，所需时间越长：

* **`external_key.failed`：** 在有人禁用该密钥或移除 Anthropic 对它的访问权限后的 30 分钟内。该事件到达时，请求可能仍会成功。
* **`external_key.recovered`：** 在该密钥恢复正常工作后约 30 分钟。

### 加密密钥事件何时可能不会到达

不要将这些事件作为您唯一的信号。还请针对 Messages API 返回的三种 `cmek_key` 错误设置告警，并定期[补齐遗漏的事件](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#catch-up-on-missed-events)。

在以下情况下，您可能会错过其中一个或两个事件：

| 情况 | 您可能错过的内容 |
| ----- | ----- |
| 没有请求使用该密钥 | `external_key.failed`，因为只有当请求使用该密钥时，Anthropic 才会察觉到拒绝 |
| 在最后一次拒绝后的大约一天内没有请求使用该密钥 | `external_key.recovered`，因为 Anthropic 会停止等待成功 |
| 您的 KMS 发生中断或超时 | 两个事件，因为中断不属于拒绝 |
| 多个组织的密钥大约在同一时间开始失败 | 两个事件以及电子邮件，因为原因可能在于 Anthropic 或云提供商 |
| [您默认工作区的密钥](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#if-your-default-workspace-uses-an-encryption-key)失败，且它加密了您端点的某个值 | 在该密钥恢复正常工作之前的所有事件，包括其自身的 `external_key.failed` |

由于您可能会错过任一事件，您可能会连续收到两次 `external_key.failed`，或者收到 `external_key.recovered`，而在此之前没有 `external_key.failed`。

## 端点字段

组织 webhook 端点具有以下字段：

| 字段 | 类型 | 描述 |
| ----- | ----- | ----- |
| `id` | string | 端点的 ID，以 `wep_` 开头。由 Anthropic 设置。 |
| `url` | string | Anthropic 发送事件的目标地址。必须是 443 端口上的 HTTPS，并使用公共域名，而不是 IP 地址。您组织中的两个已启用的组织 webhook 端点不能共用同一个 URL。 |
| `enabled_events` | array of strings | 此端点接收的[事件类型](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#event-types)。创建请求必须至少列出一种。 |
| `name` | string | 供您自己使用的可选标签。 |
| `description` | string | 供您自己使用的可选描述。 |
| `state` | string | `enabled` 或 `disabled`。 |
| `disabled_reason` | string | Anthropic 禁用该端点的原因。否则为空。 |
| `created_at` | string | 端点的创建时间。由 Anthropic 设置。 |
| `updated_at` | string | 端点的最后更改时间。由 Anthropic 设置。 |
| `signing_secret` | string | 一个以 `whsec_` 为前缀的密钥，随端点一起创建。只有创建端点或替换密钥的响应才会包含它，因此请妥善保存。 |

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

* **重试：** 如果投递失败，Anthropic 会在接下来的几分钟内重试几次。如果所有尝试都失败，Anthropic 会丢弃该事件，因此即使您的端点短暂中断，也可能丢失事件。

* **自动禁用：** 在以下情况下，Anthropic 会禁用端点：

  * 它返回重定向（`3xx`）。
  * 其 URL 解析为非公共 IP 地址。
  * 其投递持续失败。

  对于已禁用的端点，Anthropic 会跳过事件，并且之后也不会再发送。

### 补齐遗漏的事件

当 Anthropic 丢弃事件、禁用端点或为已禁用的端点跳过事件时，它不会通知您。请定期（例如每天一次）补齐遗漏，并在以下时间进行：

* 首次向端点添加事件类型时
* 您的端点发生中断之后
* 您重新启用端点之后

每次都请检查您端点的 `state` 是否为 `enabled`。然后，检查您所使用的每组事件背后的资源：

* **用户配置文件：** 按执行状态列出您的配置文件，如[在收到第一个事件之前](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#before-your-first-event)中所述。未出现在这些列表中的配置文件没有未结的措施。
* **加密密钥：** [列出您的密钥](https://platform.claude.com/docs/zh-CN/api/organization/external_keys/list)，并[逐一验证](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#respond-to-an-encryption-key-event)。如果您没有收到针对该密钥的事件，除非 Messages API 请求失败，否则 `success` 无需进一步检查。

## 管理端点

通过 Admin API，您可以发出以下请求：

| 任务 | 请求 |
| ----- | ----- |
| 列出您组织的端点 | [`GET /v1/organizations/webhooks`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/list) |
| 创建端点 | [`POST /v1/organizations/webhooks`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/create) |
| 获取单个端点 | [`GET /v1/organizations/webhooks/{webhook_endpoint_id}`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/retrieve) |
| 更新端点的 `url`、`name`、`description`、`enabled_events` 或 `state` | [`POST /v1/organizations/webhooks/{webhook_endpoint_id}`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/update) |
| 删除端点 | [`DELETE /v1/organizations/webhooks/{webhook_endpoint_id}`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/delete) |
| 替换签名密钥 | [`POST /v1/organizations/webhooks/{webhook_endpoint_id}/regenerate_signing_secret`](https://platform.claude.com/docs/zh-CN/api/beta/organization/webhooks/regenerate_signing_secret) |

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

此示例为两种用户配置文件事件（Anthropic 不会发送这些事件）创建一个端点。要接收其他事件（例如 `external_key.failed` 和 `external_key.recovered`），请改为在 `enabled_events` 中列出它们的类型：

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

如果您拥有 `admin` 组织角色，也可以改为在 Claude Console 中管理加密密钥事件的端点。请使用 [Webhook](https://platform.claude.com/settings/webhooks) 页面的**组织**选项卡，该页面会在您创建端点或替换其密钥时显示一次签名密钥。

## 如果您的默认工作区使用加密密钥

Anthropic 将您组织的端点存储在您的[默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#how-workspaces-work)中。如果该工作区使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)，Anthropic 会使用该密钥加密以下值：

| 值 | 对其进行加密的请求 |
| ----- | ----- |
| `name` 和 `description` | 创建端点，或对其发送任何更新 |
| 签名密钥 | 创建端点，或替换密钥 |

Anthropic 不会使用您的密钥加密其他字段，例如 `url`。

对于在[您的密钥生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)之前创建的端点，每个值都会继续由 Anthropic 的密钥保护，直到上述某个请求对其进行加密。没有任何字段会显示哪个密钥保护着某个值。要将端点的值转由您的密钥保护，请更新该端点并替换其密钥。

一旦您的密钥加密了端点的任何值，该端点就依赖于该密钥。当 Anthropic 无法使用该密钥时，需要该密钥的投递和请求都会失败。请求通常会返回 `500` 错误：

| 失败的内容 | 针对哪些端点 |
| ----- | ----- |
| 所有事件类型的投递 | 依赖于该密钥的端点 |
| 获取和删除请求 | 依赖于该密钥的端点 |
| 列出请求 | 全部端点（如果有任何端点依赖于该密钥） |
| 创建、更新和替换密钥请求 | 通常为全部端点 |

Anthropic 会丢弃它无法投递的事件，并且之后也不会再发送。这些端点保持启用状态，一旦 Anthropic 能够使用该密钥，投递就会恢复。然后，请[补齐遗漏的事件](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#catch-up-on-missed-events)。

如果您在 KMS 中删除该密钥，所有这些失败都将是永久性的，并且您将无法再创建端点。有关还有哪些情况会导致 Anthropic 无法使用密钥，请参阅[轮换、禁用或删除您的加密密钥时会发生什么](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle)。

## 错误

在以下情况下，请求会返回 `400`：

* 其 `url` 不符合[端点字段](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#endpoint-fields)中的规则，或者创建请求未列出任何事件类型。错误消息会说明问题所在。
* 您的组织没有默认工作区。
* 该请求会使您的组织拥有超过 32 个组织 webhook 端点。
* 其 `enabled_events` 包含组织 webhook 端点不接受的事件类型。错误消息会列出可接受的类型。

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
