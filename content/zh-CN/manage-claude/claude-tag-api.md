---
title: Claude Tag API
url: https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api
description: 从您自己的系统管理 Claude Enterprise 组织在 Slack 中的 Claude Tag：读取和更改 Slack 频道与工作区的设置，查看哪些设置生效及其原因，并附加访问包。
---

[Claude Tag](https://claude.com/docs/claude-tag/overview) 是在 Slack 中作为团队成员工作的 Claude：有人在频道中提及 Claude，Claude 就会在该线程中回答。借助 Claude Tag API，您可以从自己的系统读取和更改 Claude Tag 设置，而这些设置原本需要组织所有者在 [claude.ai > 组织设置 > Claude 标签](https://claude.ai/admin-settings/claude-tag)中手动设置。您可以用它从审批工作流中逐个频道开启 Claude，将配置保存在版本控制中，或审计每个频道中生效的设置。

Claude Tag 是一款 claude.ai 产品。此 API 为 Claude Enterprise 组织管理该产品，与 Claude Platform 工作区、API 用量或 Claude Console 无关。

<Check>
  **需要具有作用域的 Admin API 密钥**

  这些端点需要具有 `read:claude_tag` 作用域（用于 `GET` 端点）或 `write:claude_tag` 作用域（用于 `POST` 和 `DELETE` 端点）的 Admin API 密钥；请参阅[作用域](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#scopes)。请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization)，了解您的主所有者在何处创建密钥。
</Check>

<Note>
  Claude Tag API 目前处于 **beta** 阶段，仅适用于 Claude Enterprise 组织。它不适用于 Claude Platform（Claude Console）组织。

  每个请求都必须包含 [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers) `anthropic-beta: experimental-claude-tag-2026-10-21`（SDK 和 `ant` CLI 会为您发送）。缺少该标头的请求会返回 `404`，就像该端点不存在一样。
</Note>

## 端点

该 API 在四种资源上提供 14 个端点：

| 资源                                                                 | 端点                                                                                                                                                                                                                                                                                                     |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Slack 频道：**&#x8BBE;置频道、列出工作区的频道或通过 Slack ID 查找频道、读取频道、更改其设置、移除频道 | `POST /v1/organizations/claude_tag/slack_channels`<br />`GET /v1/organizations/claude_tag/slack_channels`<br />`GET /v1/organizations/claude_tag/slack_channels/{id}`<br />`POST /v1/organizations/claude_tag/slack_channels/{id}`<br />`DELETE /v1/organizations/claude_tag/slack_channels/{id}`      |
| **Slack 工作区：**&#x5217;出您的工作区或通过 Slack ID 查找工作区、读取工作区、更改其设置         | `GET /v1/organizations/claude_tag/slack_workspaces`<br />`GET /v1/organizations/claude_tag/slack_workspaces/{id}`<br />`POST /v1/organizations/claude_tag/slack_workspaces/{id}`                                                                                                                       |
| **Bundle：**&#x5217;出您组织的访问包、读取某个 Bundle（只读）                        | `GET /v1/organizations/claude_tag/bundles`<br />`GET /v1/organizations/claude_tag/bundles/{bundle_id}`                                                                                                                                                                                                 |
| **Bundle 目标：**&#x5217;出 Bundle 附加的位置、将其附加到频道或工作区、读取某个附加项、将其分离      | `GET /v1/organizations/claude_tag/bundles/{bundle_id}/targets`<br />`POST /v1/organizations/claude_tag/bundles/{bundle_id}/targets`<br />`GET /v1/organizations/claude_tag/bundles/{bundle_id}/targets/{target_id}`<br />`DELETE /v1/organizations/claude_tag/bundles/{bundle_id}/targets/{target_id}` |

此版本涵盖 Slack。连接和断开 Slack 工作区或 Enterprise Grid、您组织的 Slack 默认值、创建和编辑访问包、支出限额，以及[设置](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#settings)中五项以外的所有 Claude Tag 设置，仍在 claude.ai 中进行。

## 前提条件

* 您的组织必须使用 Claude Enterprise 计划，已[设置 Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview)，并且至少连接了一个 Slack 工作区或 Enterprise Grid。
* 您的主所有者在 [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access) 中创建具有 `read:claude_tag` 作用域、`write:claude_tag` 作用域或两者兼有的 Admin API 密钥。请参阅[为 Claude Enterprise 组织创建密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization)。
* 每个请求都携带三个标头：`x-api-key`、`anthropic-version: 2026-10-21`（请参阅 [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)）和 `anthropic-beta: experimental-claude-tag-2026-10-21`。

在 Python、TypeScript、C#、Go、Java、PHP 和 Ruby 中，SDK 在 `client.beta.organization.claude_tag` (typescript: `client.beta.organization.claudeTag`; csharp, go: `client.Beta.Organization.ClaudeTag`; java: `client.beta().organization().claudeTag()`; php: `$client->beta->organization->claudeTag`) 下提供这些端点，[`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart) 则在 `ant beta:organization:claude-tag` 下提供；它们会为您发送 `anthropic-version` 和 `anthropic-beta` 标头。本页示例从 `ANTHROPIC_API_KEY` 环境变量读取 Admin API 密钥。当 SDK 或 CLI 为您获取更多页面时，`limit` 设置的是每页大小，而不是总数（请参阅[分页](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#pagination)）。

API 密钥属于组织，在创建者离开后仍然有效。请勿共享这些密钥，也不要将其提交到源代码管理中。

## 快速入门

列出您组织在 Claude Tag 中的 Slack 工作区：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/slack_workspaces" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-workspaces list
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspaces = client.beta.organization.claude_tag.slack_workspaces.list()

  # 根据需要自动获取更多页面。
  for workspace in workspaces:
      print(f"{workspace.id}: {workspace.slack.workspace_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const workspaces = await client.beta.organization.claudeTag.slackWorkspaces.list();

  for await (const workspace of workspaces) {
    console.log(`${workspace.id}: ${workspace.slack.workspace_id}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.ClaudeTag.SlackWorkspaces.List();

  await foreach (var workspace in page.Paginate())
  {
      Console.WriteLine($"{workspace.ID}: {workspace.Slack.WorkspaceID}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  workspaces := client.Beta.Organization.ClaudeTag.SlackWorkspaces.ListAutoPaging(
  	context.Background(),
  	anthropic.BetaOrganizationClaudeTagSlackWorkspaceListParams{},
  )

  for workspaces.Next() {
  	workspace := workspaces.Current()
  	fmt.Printf("%s: %s\n", workspace.ID, workspace.Slack.WorkspaceID)
  }
  if err := workspaces.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var workspaces = client.beta().organization().claudeTag().slackWorkspaces().list();

  for (var workspace : workspaces.autoPager()) {
      IO.println(workspace.id() + ": " + workspace.slack().workspaceId());
  }
  ```

  ```php PHP
  $client = new Client();

  $workspaces = $client->beta->organization->claudeTag->slackWorkspaces->list();

  // 仅限当前页；如需下一页，请使用 page: $workspaces->nextPage 再次调用 list()。
  foreach ($workspaces->getItems() as $workspace) {
      echo "{$workspace->id}: {$workspace->slack->workspaceID}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.claude_tag.slack_workspaces.list

  page.auto_paging_each do |workspace|
    puts "#{workspace.id}: #{workspace.slack.workspace_id}"
  end
  ```
</CodeGroup>

```json Response
{
  "data": [
    {
      "type": "slack_workspace",
      "id": "slws_011CZkZnWa7xPc3uJfZe5LgT",
      "slack": {
        "workspace_id": "T01ABCD2EFG"
      },
      "settings": {
        "enabled": null,
        "instructions": "Answer in English unless asked otherwise.",
        "model": null,
        "environment_id": null,
        "guest_policy": null
      },
      "sources": {
        "enabled": null,
        "instructions": [
          {
            "type": "slack_workspace",
            "slack_workspace_id": "slws_011CZkZnWa7xPc3uJfZe5LgT"
          }
        ],
        "model": null,
        "environment_id": null,
        "guest_policy": null
      },
      "created_at": "2026-09-02T17:04:11Z",
      "updated_at": "2026-10-06T09:41:53Z"
    }
  ],
  "next_page": null
}
```

`settings` 中为 `null` 的设置表示该工作区未设置此项，因此工作区会继承它，且该设置没有来源。若要读取实际生效的值，请参阅[存储值和继承值](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#stored-and-inherited-values)。

## 作用域

| 作用域                | 授予的权限                                                    |
| ------------------ | -------------------------------------------------------- |
| `read:claude_tag`  | 本页上的每个 `GET` 端点。                                         |
| `write:claude_tag` | 本页上的每个 `POST` 和 `DELETE` 端点。它不授予读取权限，尽管更改请求的响应中包含已更改的资源。 |

一个密钥可以同时具有两个作用域。先查找频道再更改频道的集成需要两者兼有。没有其他作用域可授予对这些端点的访问权限，在 Claude Console 中创建的密钥在此处无效。

密钥必须属于整个组织。组织始终是密钥所属的组织，任何请求都不指定组织。`anthropic-workspace-id` 标头会被忽略，因为这里没有任何内容属于 Claude Platform 工作区。

## 关键概念

### 本页与 claude.ai 中的名称

| 此 API            | claude.ai 和 Slack                                                                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Slack 工作区        | 一个 Slack 工作区。Slack 自身的 API 称其为 team，因此本页中 Slack 自身的工作区 ID 就是那里的 team ID。它与 Claude Platform 工作区无关，后者的 ID 以 `wrkspc_` 开头。                     |
| Slack 默认值        | **默认 Slack 访问权限**：适用于每个工作区和频道的设置。组织所有者在 claude.ai 中更改它们；此 API 显示频道或工作区在何处继承这些设置。                                                            |
| 设置层级             | 一个[作用域](https://claude.com/docs/claude-tag/admins/customize)：频道、工作区或 Slack 默认值                                                              |
| Bundle           | 一个访问包（[Claude Tag 术语表](https://claude.com/docs/claude-tag/concepts/glossary#access-bundle)）                                                 |
| `enabled`        | **Enable Claude Tag in Slack（在 Slack 中启用 Claude Tag）** 开关                                                                                   |
| `instructions`   | **自定义指令**                                                                                                                                   |
| `model`          | **模型**                                                                                                                                      |
| `environment_id` | **环境**                                                                                                                                      |
| `guest_policy`   | **How should Claude work in channels with guests（Claude 在有访客的频道中应如何工作）**：`restrict` 对应**限制**，`channel_only` 对应**仅限频道**，`allow` 对应**完全访问权限** |

### 已设置的频道和工作区

当 Claude Tag 为某个频道或工作区保存了设置记录时，该频道或工作区即为**已设置**。只有已设置的频道或工作区才有 `id`，也只有它们会被列出：工作区的完整频道列表由 Slack 而非此 API 保存。

* 当组织所有者在 claude.ai 中连接**工作区**时，该工作区即被设置。当所有者连接 Slack Enterprise Grid 时，Grid 的各工作区会在稍后被设置。Claude 在某工作区中的首次使用，或您在其中[设置](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)的第一个频道，会设置尚未设置的工作区。没有其他端点会设置工作区，也没有端点会移除工作区。
* **频道**可在 claude.ai 中设置、通过 Claude 在其中的首次使用设置，或通过[设置频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)设置。

当您的组织与某工作区的连接处于活动状态时，您的组织拥有该工作区及其中的频道。Slack Enterprise Grid 将多个工作区归于同一公司之下。当您的组织与某 Grid 的连接处于活动状态时，它还拥有该 Grid 中未被其他组织连接的每个工作区。这样的工作区与其他工作区没有区别：一旦设置，它就有 `id`，会被列出，可以读取、更改和附加 Bundle，其频道也可以列出。Grid 本身不是此 API 的资源，也没有 `id`。您不拥有的频道或工作区会返回与不存在的频道或工作区相同的 `404`。

您的组织仅通过 Grid 拥有的工作区在被设置之前，凡是请求通过 Slack 自身 ID 指定它时都会返回该 `404`：查找该工作区、列出其频道，或查找其中尚未设置的频道时。您仍然可以在其中[设置频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)，这也会设置该工作区，且不存储任何设置。

与整个 Grid 共享的频道可以存储在 Grid 自身的 Slack ID（以 `E` 开头）下，而不是某个工作区的 ID 下。此时该 ID 就是频道的 `slack.workspace_id`，以它作为 `slack_workspace_id` [列出频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-channels)会返回这些频道。没有任何工作区对象拥有该 ID：如果所有者在 claude.ai 中对 Grid 本身存储了设置，`sources` 可能会将其命名为一个 `slack_workspace`，而读取其 `id` 会返回 `404`。

### ID

您通过 Slack 显示的 ID 来识别频道。使用[按 Slack ID 查找频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#find-a-channel-by-its-slack-ids)查找一次其 `id`，然后存储并使用该 `id`。

| ID              | 前缀                                                                                                                                          | 出现位置                                           |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| Slack 频道资源      | `slch_`                                                                                                                                     | 频道上的 `id`；请求体和响应中的 `slack_channel_id`          |
| Slack 工作区资源     | `slws_`                                                                                                                                     | 工作区上的 `id`；请求体和响应中的 `slack_workspace_id`       |
| Bundle          | `capp_`                                                                                                                                     | Bundle 上的 `id`；路径中的 `{bundle_id}`              |
| Bundle 目标       | `btgt_`                                                                                                                                     | Bundle 目标上的 `id`；路径中的 `{target_id}`            |
| Slack 自身的工作区 ID | `T`，Grid 自身的 ID 则为 `E`（请参阅[已设置的频道和工作区](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-channels-and-workspaces)） | `slack.workspace_id`；`slack_workspace_id` 查询参数 |
| Slack 自身的频道 ID  | `C` 或 `G`                                                                                                                                   | `slack.channel_id`；`slack_channel_id` 查询参数     |

<Warning>
  `slack_channel_id` 和 `slack_workspace_id` 各有两种含义。在**查询参数**中，它们是 Slack 自身的 ID（`C05HJKL6MNP`、`T01ABCD2EFG`）。在**请求体或响应**中，它们是此 API 发放的 `id`（`slch_...`、`slws_...`）。请求体或响应中的 Slack 自身 ID 始终位于 `slack` 下。
</Warning>

查询参数或请求体中不以 `C` 或 `G` 开头的 Slack 频道 ID 会返回 `400`；任何在首字母之后包含大写字母和数字以外字符、或长度超过 32 个字符的 Slack ID 也会返回 `400`。

只要频道或工作区保持设置状态，无论您对其做何更改，其 `id` 都不会改变，唯一的例外见[只读模式](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#read-only-mode)。被移除后再次设置的频道或工作区会获得新的 `id`，旧的 `id` 将持续返回 `404`。当存储的 `id` 返回不带 `error_code` 的 `404` 时，请使用相同的密钥和标头[列出您的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-workspaces)。该请求不指定任何频道或工作区，因此如果它返回 `200`，说明该 `id` 已失效：请通过 Slack ID 重新查找该频道或工作区。任何其他状态都不能说明该 `id` 的情况：请保留您存储的 `id` 值，并参阅[错误响应](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#error-responses)。

### 设置

每个设置层级都存储相同的五项设置，均为可选：

| 设置               | 类型      | 描述                                                                                                                                                                                                                                                                                                                                                          |
| ---------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enabled`        | boolean | Claude 是否开启。                                                                                                                                                                                                                                                                                                                                                |
| `instructions`   | string  | Claude 在每个会话中读取的常设指导。去除首尾空白后最多 8,192 个字符。空白文本会清除该设置。                                                                                                                                                                                                                                                                                                        |
| `model`          | string  | 新会话启动时使用的模型：您的组织可在 Claude Tag 中使用的[模型 ID](https://platform.claude.com/docs/zh-CN/models/overview)。最多 128 个字符。                                                                                                                                                                                                                                               |
| `environment_id` | string  | 会话运行所在的云环境，前缀为 `env_`。它必须处于活动状态并属于您的组织。                                                                                                                                                                                                                                                                                                                     |
| `guest_policy`   | object  | 当 Slack 访客在频道中时，Claude 在该频道中对所有人的行为：`{"type": "allow"}`（Claude 回答）、`{"type": "restrict"}`（Claude 不回答）或 `{"type": "channel_only"}`（Claude 仅使用附加到频道本身的 Bundle 回答；如果您的组织无法使用 `channel_only`，设置它的请求会返回 `400`）。关于它在通过 Slack Connect 或跨 Enterprise Grid 共享的频道中的含义，请参阅[限制访客频道](https://claude.com/docs/claude-tag/admins/restrict-access#restrict-guest-channels)。 |

`model` 和 `environment_id` 仅在写入时检查，之后不再检查：如果模型或环境后来发生变化，响应仍会返回存储的值。

如果组织所有者在 claude.ai 中将某个设置层级置于自托管运行器池上，则其 `environment_id` 会返回该池的 ID，以 `ccpool_` 开头。此 API 无法设置运行器池，对于在存储了运行器池的设置层级上更改或清除 `environment_id` 的请求，会返回 `400`。

### 存储值和继承值

设置存在于三个层级：频道、频道所在的工作区，以及由组织所有者在 claude.ai 中设置的您组织的 Slack 默认值。每个层级只存储在其上设置的内容。频道或工作区对象的 `settings` 可以包含以下两者之一：

* **它存储的内容。**&#x9664;非您另行要求，响应中包含的就是这些内容。未在频道或工作区上设置的设置为 `null`。可用它与您在其他地方保存的配置进行比较。
* **在其中实际生效的内容**，即综合考虑所有层级之后的结果。要获取此内容，请在任何返回频道或工作区的 `GET` 请求（包括这两个列表请求）中添加查询参数 `include_inherited=true`。您可以用它进行审计，或了解 Claude 为何在某个频道中表现出当前的行为。该参数默认为 `false`，非布尔值会返回 `400`。设置频道或更改设置的请求不接受该参数，如果您发送该参数则会返回 `400`；其响应包含的是存储的内容。

在这两种情况下，`sources` 都会为每项设置指明 `settings` 中的值来自哪个层级。不带 `include_inherited` 时，频道或工作区存储的设置，其来源指向该频道或工作区本身；它未存储的设置没有来源：为 `null`，对于 `instructions` 则为空列表。使用 `include_inherited=true` 时，除 `instructions` 外的每项设置都有来源，没有任何层级设置的设置，其来源为 `{"type": "default"}`；当没有层级设置 `instructions` 时，它为 `null`，其来源列表为空。因此，要区分频道或工作区自身设置的值与继承的值，请查看其来源，而不是看该值是否为 `null`。

每项设置单独解析：

| 设置               | 规则                                                            | 没有层级设置时                 |
| ---------------- | ------------------------------------------------------------- | ----------------------- |
| `enabled`        | 由设置它的最近层级决定值，因此频道上的 `true` 会在该频道开启 Claude，即使其工作区存储的是 `false`。 | `true`                  |
| `instructions`   | 累加。每个设置了它的层级的文本会从最宽泛的层级开始依次拼接，中间以空行分隔。                        | `null`                  |
| `model`          | 由设置它的最近层级决定值。                                                 | `null`：Claude 使用产品的默认模型 |
| `environment_id` | 由设置它的最近层级决定值。                                                 | `null`                  |
| `guest_policy`   | 由设置它的最近层级决定值。                                                 | `restrict`              |

例如，某组织在其 Slack 默认值上存储了指令，在一个工作区上关闭了 Claude，并在该工作区的一个频道上设置了模型。使用 `include_inherited=true` 读取时，该频道会显示这三个值及各自的来源：

```json
{
  "type": "slack_channel",
  "id": "slch_011CZkZmRt4vNq8sHbYw2KdX",
  "slack": {
    "workspace_id": "T01ABCD2EFG",
    "channel_id": "C05HJKL6MNP"
  },
  "settings": {
    "enabled": false,
    "instructions": "Be brief.",
    "model": "claude-opus-5-5",
    "environment_id": null,
    "guest_policy": { "type": "restrict" }
  },
  "sources": {
    "enabled": {
      "type": "slack_workspace",
      "slack_workspace_id": "slws_011CZkZnWa7xPc3uJfZe5LgT"
    },
    "instructions": [{ "type": "slack_default" }],
    "model": {
      "type": "slack_channel",
      "slack_channel_id": "slch_011CZkZmRt4vNq8sHbYw2KdX"
    },
    "environment_id": { "type": "default" },
    "guest_policy": { "type": "default" }
  },
  "created_at": "2026-10-19T16:20:05Z",
  "updated_at": "2026-10-19T16:24:38Z"
}
```

不带该参数读取时，同一频道的 `settings` 中包含其 `model` 和四个 `null` 设置；在 `sources` 中，`model` 指向该频道，`instructions` 为空列表，其余三项为 `null`。

来源为以下之一：

| `type`                 | 含义                                                                                                                                                   |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `slack_channel`        | 频道，在 `slack_channel_id` 中指明。                                                                                                                         |
| `slack_workspace`      | 工作区，在 `slack_workspace_id` 中指明。                                                                                                                      |
| `slack_parent_channel` | 代码频道的父频道，在 `slack_channel_id` 中指明。代码频道是从另一个频道发起的频道。当父频道是同一工作区中的公共频道时，它位于该频道与工作区之间，且仅提供 `instructions` 和 `environment_id`。                            |
| `slack_grid_channel`   | 在其 Slack Enterprise Grid 的另一个工作区下设置的同一 Slack 频道，通过 `slack` 下的 Slack 自身 ID 指明。它仅提供 `enabled`：当频道及其工作区都未设置 `enabled` 时，该频道在 Grid 另一个工作区下存储的 `true` 生效。 |
| `slack_default`        | 您组织的 Slack 默认值，没有 ID。                                                                                                                                |
| `default`              | 没有层级设置值，因此应用内置默认值。                                                                                                                                   |

`sources.instructions` 列出每个提供文本的层级，从最宽泛的开始，这也是它们的文本在 `settings.instructions` 中的顺序；若没有层级提供文本则为空。它说明谁的文本在前，而不说明某一层级的文本在何处结束：要获取某一层级自己的文本，请不带 `include_inherited` 读取该频道或工作区；对于 Slack 默认值，请在 claude.ai 中查看。指向频道或工作区的来源始终携带其 ID，且绝不为 `null`：在 `slack_channel_id` 或 `slack_workspace_id` 中为此 API 的 `id`，或者在 `slack_grid_channel` 上，像频道对象一样在 `slack` 下为 Slack 自身的 ID。

使用 `include_inherited=true` 时，如果某个层级无法读取，请求会以 `503` 失败，而不是在缺少该层级的情况下作答。

### 更改设置

两个更新端点和[设置频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)都按相同规则接受 `settings` 对象：

* **您省略的设置保持不变。**`{"settings": {"enabled": false}}` 会关闭 Claude，其余四项设置保持原样。
* **您以 `null` 发送的设置会被清除**，因此该层级会重新继承它。
* **全有或全无。**&#x5982;果请求中的任何值被拒绝，则请求中的任何设置都不会被存储。
* **可安全重复。**&#x6BCF;次更改都会使资源保持为第一次相同请求所留下的状态，因此没有端点接受 `Idempotency-Key`。未改变任何内容的请求不会写入任何内容，也不会记录活动源事件。
* **最后写入者获胜。**&#x6CA1;有请求接受前置条件。两个调用方各自读取资源后更改同一设置，会相互覆盖。仅当另一更改在您的请求运行期间与之重叠时才会返回 `409`；您的请求要求更改或删除的内容均未执行，但设置请求可能已经设置了频道，因此请重新读取资源，如仍需要则重新发送更改。

更改频道上的 `guest_policy` 可能会结束该频道中正在运行的会话。这些会话在后台结束，响应不会等待其完成。

### 只读模式

**API / Terraform read-only mode（API / Terraform 只读模式）** 是组织所有者在 claude.ai 的 Claude Tag 设置中开启和关闭的一项设置。该名称描述的是 claude.ai 和 Slack：当该模式开启时，它们会拒绝个人对此 API 所管理内容的更改，因此只有密钥能进行这些更改。它们还会拒绝少数没有密钥能进行的相关更改，例如更改 Slack 默认值上的设置，因此在所有者关闭该模式之前，任何人都无法进行这些更改。它不会使此 API 变为只读。无论该模式开启与否，本页的每个端点都以相同方式响应，且没有端点读取或更改它。

此模式关闭时，个人和密钥会更改相同的设置、附加和分离相同的 Bundle，每次更改都会替换已存储的内容。如果您的流水线应成为唯一的更改来源，请开启此模式。

此模式开启时，claude.ai 和 Slack 会拒绝：

* 更改频道、工作区或 Slack 默认值（密钥也无法更改）上五项设置中的任何一项
* 设置频道、移除频道、添加已连接 Enterprise Grid 的工作区，或在 claude.ai 中断开工作区或 Grid
* 将 Bundle 附加到频道或工作区、分离 Bundle，或删除已附加到某处的 Bundle

无论哪种情况，有些操作仍然可行：所有者仍可连接新的工作区或 Grid、创建和编辑 Bundle，以及更改此 API 不管理的 Claude Tag 设置。Claude 的首次使用仍会设置频道或尚未设置的 Grid 工作区，在 Slack 中删除频道仍会将其移除，卸载 Claude 的 Slack 应用仍会断开该工作区或 Grid。

当该模式关闭时，在 claude.ai 中设置一个已设置的频道可能会替换该频道的记录，但仅限于尚未包含任何内容的频道：即由 Claude 首次使用设置、或由此 API 在不带 `settings` 的情况下设置，且此后无人存储设置或附加 Bundle 的频道。此时该频道会获得新的 `id`，旧的 `id` 返回 `404`。任何其他频道在被移除之前都保持其 `id`。

### 群组私信

Slack 群组私信的 ID 与频道格式相同，因此 Claude 在其中的使用会像频道一样设置它：它有 `id`，会被列出，可以读取。通过此 API 设置群组私信、更改其设置、向其附加 Bundle 以及移除它，都会返回 `400`，但对于已设置的群组私信，省略 `settings` 的设置请求会像读取一样以 `200` 返回它。允许从群组私信分离 Bundle。

## 示例工作流

### 仅在已批准的频道中开启 Claude

任何成员都可以邀请 Claude 加入公共频道。要仅在您已批准的地方开启 Claude：

1. 请组织所有者在 claude.ai 中通过**默认 Slack 访问权限**下的 **Enable Claude Tag in Slack** 开关在所有地方关闭 Claude，这会在您组织的 Slack 默认值上将 `enabled` 存储为 `false`。在此期间，[只读模式](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#read-only-mode)必须处于关闭状态。
2. 当您的审批系统批准某个频道时，使用 `"settings": {"enabled": true}` [设置该频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)。该请求接受 Slack 自身的 ID，即使频道已设置也会返回它，因此可以安全重复。
3. 在同一次运行中，[附加](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#attach-a-bundle)该团队获批的 Bundle，使频道的第一个会话就已拥有其工具。

### 将配置保存在版本控制中

1. 以 Slack 为每个频道显示的 ID 作为配置的键，并通过这些 ID [查找每个频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#find-a-channel-by-its-slack-ids)。
2. 不带 `include_inherited` 读取每个频道，并将其 `settings` 与您的存储库声明的内容进行比较。带上该参数会混入继承的值。
3. 只发送不同的设置，并为存储库不再声明的设置发送 `null`。
4. 请组织所有者开启[只读模式](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#read-only-mode)，以免有人手动更改相同的设置。

### 审计所有地方生效的设置

1. [列出您的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-workspaces)，然后[列出每个工作区的频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-channels)，两者都使用 `include_inherited=true`，使每一项都包含在其中生效的内容。如果您的组织连接了 Enterprise Grid，还要列出存储在 Grid 自身 Slack ID 下的频道。请参阅[已设置的频道和工作区](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-channels-and-workspaces)。
2. 标记您的策略所禁止的情况，例如 `enabled` 为 `true` 但没有存档审批的频道，或 `guest_policy` 为 `allow` 的频道。
3. [列出您的 Bundle](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-bundles)，然后列出[每个 Bundle 的目标](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-a-bundles-targets)，以查看对敏感系统的访问附加在何处。

## Slack 频道

频道对象描述一个 Slack 频道：

| 字段           | 描述                                                                                                                                                                                                                                                                            |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`       | `slack_channel`。                                                                                                                                                                                                                                                              |
| `id`         | 前缀为 `slch_`。请参阅 [ID](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#ids)。                                                                                                                                                                                |
| `slack`      | Slack 自身的 ID：`workspace_id` 为频道所在的工作区，`channel_id` 为频道。                                                                                                                                                                                                                       |
| `settings`   | 频道本身存储的内容，或者在使用 `include_inherited=true` 时，在其中生效的内容。始终存在，包含全部五项[设置](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#settings)；未设置的为 `null`。请参阅[存储值和继承值](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#stored-and-inherited-values)。 |
| `sources`    | 每项设置对应 `settings` 中的值所来自的层级。始终存在。                                                                                                                                                                                                                                             |
| `created_at` | 频道被设置的时间。                                                                                                                                                                                                                                                                     |
| `updated_at` | Claude Tag 对该频道的记录最后一次变更的时间。每次设置发生变化时它都会更新，无论是在此处、在 claude.ai 中还是在 Slack 中，也可能因此处任何响应都未显示的变化而更新。请将较新的值视为重新读取频道的理由，而不是设置已变化的证据。附加或分离 Bundle 本身不会使其更新。                                                                                                                          |

不是有效 `slch_` 标识符的 `id` 会返回 `400`。有效但不指向您组织任何频道的 `id` 会返回 `404`。

### 设置频道

`POST /v1/organizations/claude_tag/slack_channels` 设置一个 Slack 频道并返回它。`slack.workspace_id` 和 `slack.channel_id` 为必填项，且是 Slack 自身的 ID。`settings` 为可选项，遵循[更改设置](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#changing-settings)。如果频道已设置，请求会返回现有频道并应用 `settings`；两种情况下响应均为 `200`。需要 `write:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21" \
    -d '{
      "slack": {"workspace_id": "T01ABCD2EFG", "channel_id": "C05HJKL6MNP"},
      "settings": {"instructions": "Answer in the language of the question."}
    }'
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels create \
    --slack '{workspace_id: T01ABCD2EFG, channel_id: C05HJKL6MNP}' \
    --settings '{instructions: "Answer in the language of the question."}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  channel = client.beta.organization.claude_tag.slack_channels.create(
      slack={"workspace_id": "T01ABCD2EFG", "channel_id": "C05HJKL6MNP"},
      settings={"instructions": "Answer in the language of the question."},
  )

  print(channel.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const channel = await client.beta.organization.claudeTag.slackChannels.create({
    slack: { workspace_id: "T01ABCD2EFG", channel_id: "C05HJKL6MNP" },
    settings: { instructions: "Answer in the language of the question." }
  });

  console.log(channel.id);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var channel = await client.Beta.Organization.ClaudeTag.SlackChannels.Create(
      new()
      {
          Slack = new() { WorkspaceID = "T01ABCD2EFG", ChannelID = "C05HJKL6MNP" },
          Settings = new() { Instructions = "Answer in the language of the question." },
      }
  );

  Console.WriteLine(channel.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  channel, err := client.Beta.Organization.ClaudeTag.SlackChannels.New(
  	context.Background(),
  	anthropic.BetaOrganizationClaudeTagSlackChannelNewParams{
  		Slack: anthropic.BetaClaudeTagSlackChannelIdentifiersParam{
  			WorkspaceID: "T01ABCD2EFG",
  			ChannelID:   "C05HJKL6MNP",
  		},
  		Settings: anthropic.BetaClaudeTagSettingsCreateParam{
  			Instructions: anthropic.String("Answer in the language of the question."),
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(channel.ID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.BetaClaudeTagSettingsCreate;
  import com.anthropic.models.beta.organization.claudetag.BetaClaudeTagSlackChannelIdentifiers;
  import com.anthropic.models.beta.organization.claudetag.slackchannels.SlackChannelCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SlackChannelCreateParams.builder()
          .slack(BetaClaudeTagSlackChannelIdentifiers.builder()
              .workspaceId("T01ABCD2EFG")
              .channelId("C05HJKL6MNP")
              .build())
          .settings(BetaClaudeTagSettingsCreate.builder()
              .instructions("Answer in the language of the question.")
              .build())
          .build();
      var channel = client.beta().organization().claudeTag().slackChannels().create(params);

      IO.println(channel.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $channel = $client->beta->organization->claudeTag->slackChannels->create(
      slack: ['workspaceID' => 'T01ABCD2EFG', 'channelID' => 'C05HJKL6MNP'],
      settings: ['instructions' => 'Answer in the language of the question.'],
  );

  echo "{$channel->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  channel = client.beta.organization.claude_tag.slack_channels.create(
    slack: {workspace_id: "T01ABCD2EFG", channel_id: "C05HJKL6MNP"},
    settings: {instructions: "Answer in the language of the question."}
  )

  puts channel.id
  ```
</CodeGroup>

```json Response
{
  "type": "slack_channel",
  "id": "slch_011CZkZmRt4vNq8sHbYw2KdX",
  "slack": {
    "workspace_id": "T01ABCD2EFG",
    "channel_id": "C05HJKL6MNP"
  },
  "settings": {
    "enabled": null,
    "instructions": "Answer in the language of the question.",
    "model": null,
    "environment_id": null,
    "guest_policy": null
  },
  "sources": {
    "enabled": null,
    "instructions": [
      {
        "type": "slack_channel",
        "slack_channel_id": "slch_011CZkZmRt4vNq8sHbYw2KdX"
      }
    ],
    "model": null,
    "environment_id": null,
    "guest_policy": null
  },
  "created_at": "2026-10-19T16:20:05Z",
  "updated_at": "2026-10-19T16:20:05Z"
}
```

在设置频道之前，Claude 的 Slack 应用会向 Slack 确认该频道在该工作区中、未被归档且对应用可见。未通过该检查的频道会返回 `404`，未安装 Claude 的 Slack 应用的工作区也会返回 `404`。对于您的组织通过 Enterprise Grid 拥有的工作区，该应用必须安装在 Grid 上。

设置频道和应用 `settings` 是两个步骤。当此请求设置了频道，而 `settings` 随后以 `400`、`404` 或 `409` 被拒绝时，响应即为该拒绝，频道保持已设置状态且不存储任何内容（在 `409` 之后，除非重叠的更改移除或替换了它）。该拒绝带有以 `slack_channel_set_up_` 开头的 `error_code`，并在 `error.details.slack_channel_id` 中包含新频道的 `id`（请参阅[错误响应](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#error-responses)）。请更正 `settings` 并通过[更新](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#update-a-channel)发送到该 `id`，或再次发送设置请求。如果频道之前已设置，被拒绝的 `settings` 会返回与[更新](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#update-a-channel)相同的错误，频道保留其存储的内容。两种情况下的 `500` 或 `503` 都不带 `error_code`，也不说明频道是否已设置或 `settings` 是否已存储：请再次发送设置请求。

### 列出频道

`GET /v1/organizations/claude_tag/slack_channels` 列出某个工作区中已设置的频道，按 `created_at` 从新到旧排序。`slack_workspace_id` 为必填项，是 Slack 自身的工作区 ID。添加 `include_inherited=true` 可使每个频道包含在其中生效的内容（请参阅[存储值和继承值](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#stored-and-inherited-values)）。一个页面可能包含少于 `limit` 个频道，甚至没有频道，而 `next_page` 仍被设置，因此请持续请求页面直到 `next_page` 为 `null`（请参阅[分页](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#pagination)）。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels?slack_workspace_id=T01ABCD2EFG" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels list --slack-workspace-id T01ABCD2EFG
  ```

  ```python Python
  client = anthropic.Anthropic()

  channels = client.beta.organization.claude_tag.slack_channels.list(
      slack_workspace_id="T01ABCD2EFG"
  )

  # 根据需要自动获取更多页面。
  for channel in channels:
      print(f"{channel.id}: {channel.slack.channel_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const channels = await client.beta.organization.claudeTag.slackChannels.list({
    slack_workspace_id: "T01ABCD2EFG"
  });

  for await (const channel of channels) {
    console.log(`${channel.id}: ${channel.slack.channel_id}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.ClaudeTag.SlackChannels.List(
      new() { SlackWorkspaceID = "T01ABCD2EFG" }
  );

  await foreach (var channel in page.Paginate())
  {
      Console.WriteLine($"{channel.ID}: {channel.Slack.ChannelID}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  channels := client.Beta.Organization.ClaudeTag.SlackChannels.ListAutoPaging(
  	context.Background(),
  	anthropic.BetaOrganizationClaudeTagSlackChannelListParams{
  		SlackWorkspaceID: "T01ABCD2EFG",
  	},
  )

  for channels.Next() {
  	channel := channels.Current()
  	fmt.Printf("%s: %s\n", channel.ID, channel.Slack.ChannelID)
  }
  if err := channels.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.slackchannels.SlackChannelListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SlackChannelListParams.builder()
          .slackWorkspaceId("T01ABCD2EFG")
          .build();
      var channels = client.beta().organization().claudeTag().slackChannels().list(params);

      for (var channel : channels.autoPager()) {
          IO.println(channel.id() + ": " + channel.slack().channelId());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $channels = $client->beta->organization->claudeTag->slackChannels->list(
      slackWorkspaceID: 'T01ABCD2EFG',
  );

  // 仅限当前页；如需下一页，请使用 page: $channels->nextPage 再次调用 list()。
  foreach ($channels->getItems() as $channel) {
      echo "{$channel->id}: {$channel->slack->channelID}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.claude_tag.slack_channels.list(slack_workspace_id: "T01ABCD2EFG")

  page.auto_paging_each do |channel|
    puts "#{channel.id}: #{channel.slack.channel_id}"
  end
  ```
</CodeGroup>

响应为 `{"data": [...], "next_page": ...}`，其中每一项都是一个频道对象。指向您组织不拥有的工作区的 `slack_workspace_id` 会返回 `404`；指向您组织仅通过 Enterprise Grid 拥有的工作区的也会返回 `404`，直到该工作区被设置（请参阅[已设置的频道和工作区](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-channels-and-workspaces)）。

### 按 Slack ID 查找频道

添加 `slack_channel_id`（Slack 自身的频道 ID）可查找单个频道。`data` 包含该频道，如果频道未设置则为空。在尚未设置且您的组织仅通过 Enterprise Grid 拥有的工作区中，未设置的频道会改为返回 `404`。查找不分页：它会忽略 `page` 以及 1 到 100 之间的任何 `limit`（超出该范围的 `limit` 仍返回 `400`），其 `next_page` 为 `null`。这就是查找频道 `id` 的方法。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels?slack_workspace_id=T01ABCD2EFG&slack_channel_id=C05HJKL6MNP" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels list \
    --slack-workspace-id T01ABCD2EFG \
    --slack-channel-id C05HJKL6MNP
  ```

  ```python Python
  client = anthropic.Anthropic()

  page = client.beta.organization.claude_tag.slack_channels.list(
      slack_workspace_id="T01ABCD2EFG",
      slack_channel_id="C05HJKL6MNP",
  )

  for channel in page.data:
      print(channel.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const page = await client.beta.organization.claudeTag.slackChannels.list({
    slack_workspace_id: "T01ABCD2EFG",
    slack_channel_id: "C05HJKL6MNP"
  });

  for (const channel of page.data) {
    console.log(channel.id);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.ClaudeTag.SlackChannels.List(
      new() { SlackWorkspaceID = "T01ABCD2EFG", SlackChannelID = "C05HJKL6MNP" }
  );

  foreach (var channel in page.Items)
  {
      Console.WriteLine(channel.ID);
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  page, err := client.Beta.Organization.ClaudeTag.SlackChannels.List(
  	context.Background(),
  	anthropic.BetaOrganizationClaudeTagSlackChannelListParams{
  		SlackWorkspaceID: "T01ABCD2EFG",
  		SlackChannelID:   anthropic.String("C05HJKL6MNP"),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  for _, channel := range page.Data {
  	fmt.Println(channel.ID)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.slackchannels.SlackChannelListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SlackChannelListParams.builder()
          .slackWorkspaceId("T01ABCD2EFG")
          .slackChannelId("C05HJKL6MNP")
          .build();
      var page = client.beta().organization().claudeTag().slackChannels().list(params);

      for (var channel : page.items()) {
          IO.println(channel.id());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $channels = $client->beta->organization->claudeTag->slackChannels->list(
      slackWorkspaceID: 'T01ABCD2EFG',
      slackChannelID: 'C05HJKL6MNP',
  );

  foreach ($channels->getItems() as $channel) {
      echo "{$channel->id}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.claude_tag.slack_channels.list(
    slack_workspace_id: "T01ABCD2EFG",
    slack_channel_id: "C05HJKL6MNP"
  )

  page.data&.each do |channel|
    puts channel.id
  end
  ```
</CodeGroup>

将 `slch_` ID 作为 `slack_channel_id` 传递会返回 `400`。

### 获取频道

`GET /v1/organizations/claude_tag/slack_channels/{id}` 返回频道对象，其中包含频道存储的内容。若要改为读取在其中生效的内容，请参阅[读取频道中生效的内容](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#read-what-applies-in-a-channel)。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels/slch_011CZkZmRt4vNq8sHbYw2KdX" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels retrieve \
    --id slch_011CZkZmRt4vNq8sHbYw2KdX
  ```

  ```python Python
  client = anthropic.Anthropic()

  channel = client.beta.organization.claude_tag.slack_channels.retrieve(
      "slch_011CZkZmRt4vNq8sHbYw2KdX"
  )

  print(channel.settings.enabled)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const channel = await client.beta.organization.claudeTag.slackChannels.retrieve(
    "slch_011CZkZmRt4vNq8sHbYw2KdX"
  );

  console.log(channel.settings.enabled);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var channel = await client.Beta.Organization.ClaudeTag.SlackChannels.Retrieve(
      "slch_011CZkZmRt4vNq8sHbYw2KdX"
  );

  Console.WriteLine(channel.Settings.Enabled);
  ```

  ```go Go
  client := anthropic.NewClient()

  channel, err := client.Beta.Organization.ClaudeTag.SlackChannels.Get(
  	context.Background(),
  	"slch_011CZkZmRt4vNq8sHbYw2KdX",
  	anthropic.BetaOrganizationClaudeTagSlackChannelGetParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(channel.Settings.Enabled)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var channel = client.beta().organization().claudeTag().slackChannels()
      .retrieve("slch_011CZkZmRt4vNq8sHbYw2KdX");

  IO.println(channel.settings().enabled().orElse(null));
  ```

  ```php PHP
  $client = new Client();

  $channel = $client->beta->organization->claudeTag->slackChannels->retrieve(
      id: 'slch_011CZkZmRt4vNq8sHbYw2KdX',
  );

  echo json_encode($channel->settings->enabled) . "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  channel_id = "slch_011CZkZmRt4vNq8sHbYw2KdX"
  channel = client.beta.organization.claude_tag.slack_channels.retrieve(channel_id)

  puts channel.settings.enabled
  ```
</CodeGroup>

### 读取频道中生效的内容

带有 `include_inherited=true` 的同一请求会返回频道对象，其中 `settings` 包含频道中生效的内容，`sources` 说明每个值的来源。[存储值和继承值](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#stored-and-inherited-values)展示了完整的响应；其中 `enabled` 和 `guest_policy` 永远不会为 `null`。如果某个层级无法读取，请求会以 `503` 失败，而不是在缺少该层级的情况下作答。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels/slch_011CZkZmRt4vNq8sHbYw2KdX?include_inherited=true" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels retrieve \
    --id slch_011CZkZmRt4vNq8sHbYw2KdX \
    --include-inherited
  ```

  ```python Python
  client = anthropic.Anthropic()

  channel = client.beta.organization.claude_tag.slack_channels.retrieve(
      "slch_011CZkZmRt4vNq8sHbYw2KdX",
      include_inherited=True,
  )

  print(channel.settings.instructions)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const channel = await client.beta.organization.claudeTag.slackChannels.retrieve(
    "slch_011CZkZmRt4vNq8sHbYw2KdX",
    { include_inherited: true }
  );

  console.log(channel.settings.instructions);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var channel = await client.Beta.Organization.ClaudeTag.SlackChannels.Retrieve(
      "slch_011CZkZmRt4vNq8sHbYw2KdX",
      new() { IncludeInherited = true }
  );

  Console.WriteLine(channel.Settings.Instructions);
  ```

  ```go Go
  client := anthropic.NewClient()

  channel, err := client.Beta.Organization.ClaudeTag.SlackChannels.Get(
  	context.Background(),
  	"slch_011CZkZmRt4vNq8sHbYw2KdX",
  	anthropic.BetaOrganizationClaudeTagSlackChannelGetParams{
  		IncludeInherited: anthropic.Bool(true),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(channel.Settings.Instructions)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.slackchannels.SlackChannelRetrieveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SlackChannelRetrieveParams.builder()
          .includeInherited(true)
          .build();
      var channel = client.beta().organization().claudeTag().slackChannels()
          .retrieve("slch_011CZkZmRt4vNq8sHbYw2KdX", params);

      IO.println(channel.settings().instructions().orElse(null));
  }
  ```

  ```php PHP
  $client = new Client();

  $channel = $client->beta->organization->claudeTag->slackChannels->retrieve(
      id: 'slch_011CZkZmRt4vNq8sHbYw2KdX',
      includeInherited: true,
  );

  echo "{$channel->settings->instructions}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  channel_id = "slch_011CZkZmRt4vNq8sHbYw2KdX"
  channel = client.beta.organization.claude_tag.slack_channels.retrieve(channel_id, include_inherited: true)

  puts channel.settings.instructions
  ```
</CodeGroup>

### 更新频道

`POST /v1/organizations/claude_tag/slack_channels/{id}` 更改频道的设置并返回频道对象。`settings` 为必填项，并遵循[更改设置](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#changing-settings)中的规则。需要 `write:claude_tag` 作用域。

在一个频道中关闭 Claude，并保持其他设置不变：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels/slch_011CZkZmRt4vNq8sHbYw2KdX" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21" \
    -d '{"settings": {"enabled": false}}'
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels update \
    --id slch_011CZkZmRt4vNq8sHbYw2KdX \
    --settings '{enabled: false}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  channel = client.beta.organization.claude_tag.slack_channels.update(
      "slch_011CZkZmRt4vNq8sHbYw2KdX",
      settings={"enabled": False},
  )

  print(channel.settings.enabled)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const channel = await client.beta.organization.claudeTag.slackChannels.update(
    "slch_011CZkZmRt4vNq8sHbYw2KdX",
    { settings: { enabled: false } }
  );

  console.log(channel.settings.enabled);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var channel = await client.Beta.Organization.ClaudeTag.SlackChannels.Update(
      "slch_011CZkZmRt4vNq8sHbYw2KdX",
      new() { Settings = new() { Enabled = false } }
  );

  Console.WriteLine(channel.Settings.Enabled);
  ```

  ```go Go
  client := anthropic.NewClient()

  channel, err := client.Beta.Organization.ClaudeTag.SlackChannels.Update(
  	context.Background(),
  	"slch_011CZkZmRt4vNq8sHbYw2KdX",
  	anthropic.BetaOrganizationClaudeTagSlackChannelUpdateParams{
  		Settings: anthropic.BetaClaudeTagSettingsUpdateParam{
  			Enabled: anthropic.Bool(false),
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(channel.Settings.Enabled)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.BetaClaudeTagSettingsUpdate;
  import com.anthropic.models.beta.organization.claudetag.slackchannels.SlackChannelUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SlackChannelUpdateParams.builder()
          .settings(BetaClaudeTagSettingsUpdate.builder()
              .enabled(false)
              .build())
          .build();
      var channel = client.beta().organization().claudeTag().slackChannels()
          .update("slch_011CZkZmRt4vNq8sHbYw2KdX", params);

      IO.println(channel.settings().enabled().orElse(null));
  }
  ```

  ```php PHP
  $client = new Client();

  $channel = $client->beta->organization->claudeTag->slackChannels->update(
      id: 'slch_011CZkZmRt4vNq8sHbYw2KdX',
      settings: ['enabled' => false],
  );

  echo json_encode($channel->settings->enabled) . "\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  channel_id = "slch_011CZkZmRt4vNq8sHbYw2KdX"
  channel = client.beta.organization.claude_tag.slack_channels.update(
    channel_id,
    settings: {enabled: false}
  )

  puts channel.settings.enabled
  ```
</CodeGroup>

清除频道的指令，使只有继承的指令生效：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels/slch_011CZkZmRt4vNq8sHbYw2KdX" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21" \
    -d '{"settings": {"instructions": null}}'
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels update \
    --id slch_011CZkZmRt4vNq8sHbYw2KdX \
    --settings '{instructions: null}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  channel = client.beta.organization.claude_tag.slack_channels.update(
      "slch_011CZkZmRt4vNq8sHbYw2KdX",
      settings={"instructions": None},
  )

  print(channel.settings.instructions)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const channel = await client.beta.organization.claudeTag.slackChannels.update(
    "slch_011CZkZmRt4vNq8sHbYw2KdX",
    { settings: { instructions: null } }
  );

  console.log(channel.settings.instructions);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var channel = await client.Beta.Organization.ClaudeTag.SlackChannels.Update(
      "slch_011CZkZmRt4vNq8sHbYw2KdX",
      new() { Settings = new() { Instructions = null } }
  );

  Console.WriteLine(channel.Settings.Instructions);
  ```

  ```go Go
  client := anthropic.NewClient()

  channel, err := client.Beta.Organization.ClaudeTag.SlackChannels.Update(
  	context.Background(),
  	"slch_011CZkZmRt4vNq8sHbYw2KdX",
  	anthropic.BetaOrganizationClaudeTagSlackChannelUpdateParams{
  		Settings: anthropic.BetaClaudeTagSettingsUpdateParam{
  			Instructions: param.Null[string](),
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(channel.Settings.Instructions)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.BetaClaudeTagSettingsUpdate;
  import com.anthropic.models.beta.organization.claudetag.slackchannels.SlackChannelUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SlackChannelUpdateParams.builder()
          .settings(BetaClaudeTagSettingsUpdate.builder()
              .instructions(Optional.empty())
              .build())
          .build();
      var channel = client.beta().organization().claudeTag().slackChannels()
          .update("slch_011CZkZmRt4vNq8sHbYw2KdX", params);

      IO.println(channel.settings().instructions().orElse(null));
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\ClaudeTag\ClaudeTagSettingsUpdate;
  // ...

  $client = new Client();

  // update() 会丢弃数组中的 null 值，因此需构建 settings 对象以发送 null。
  $channel = $client->beta->organization->claudeTag->slackChannels->update(
      id: 'slch_011CZkZmRt4vNq8sHbYw2KdX',
      settings: (new ClaudeTagSettingsUpdate())->withInstructions(null),
  );

  echo "{$channel->settings->instructions}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  channel_id = "slch_011CZkZmRt4vNq8sHbYw2KdX"
  channel = client.beta.organization.claude_tag.slack_channels.update(
    channel_id,
    settings: {instructions: nil}
  )

  puts channel.settings.instructions
  ```
</CodeGroup>

### 移除频道

`DELETE /v1/organizations/claude_tag/slack_channels/{id}` 将频道从 Claude Tag 中移除。它与组织所有者在 claude.ai 中执行的移除操作相同，效果也相同。它不会删除、归档或更改 Slack 中的频道，也不会影响其中的任何消息。需要 `write:claude_tag` 作用域。

<Warning>
  移除操作无法撤销。它会删除 Claude Tag 为该频道保存的内容：

  * 其存储的设置及其 Bundle 目标
  * 其会话及会话记录，包括正在运行的会话
  * 其记忆，即 Claude 在不同会话之间保留的关于该频道的信息
  * 其例程以及从中发布的 artifacts
</Warning>

**只有在频道中 Claude 处于关闭状态时才能移除该频道。**&#x5F53; `enabled` 对该频道解析为 `true`（如[读取频道中生效的内容](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#read-what-applies-in-a-channel)所返回的那样）时，请求会返回 `400`，并带有 `error_code: "slack_channel_enabled"`。请先在频道上将 `enabled` 设置为 `false`，然后再次发送请求。唯一的例外是具有[受限合规配置](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#error-responses)的组织：它无法进行该更改，因此无论 Claude 是否在频道中开启，其请求都会移除该频道。如果此检查所需的某个层级无法读取，请求会返回 `503` 且不移除任何内容。

该检查用于防止误删正在使用的频道。它不会在之后阻止 Claude 进入该频道。移除会删除频道自身的设置，包括值为 `false` 的 `enabled`，因此此后只有频道继承的内容决定 Claude 是否在其中开启。如果继承的内容开启了 Claude，Claude 会在下次被提及时再次在该频道中回答，而这次使用会重新设置该频道：新的 `id`、没有自己的设置，也没有被移除频道的任何记忆。

* 若要让 Claude 远离某个频道，请保持该频道的设置状态并将 `enabled` 设为 `false`，而不是移除它；或者确保它将继承的内容（移除操作不会触及这些内容）使 Claude 保持关闭。
* 若要让频道不存储任何内容但保留其会话和记忆，请改为清除每项设置并分离每个 Bundle。

有两类频道无论 Claude 在其中开启还是关闭，都完全无法在此处移除，会返回不带 `error_code` 的 `400`：[群组私信](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#group-direct-messages)，以及包含此 API 不管理的配置（例如运行器池或仅 claude.ai 拥有的设置）的频道。这些检查在检查 `enabled` 之前进行，因此关闭 Claude 无济于事；第二类频道由组织所有者在 claude.ai 中移除。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/claude_tag/slack_channels/slch_011CZkZmRt4vNq8sHbYw2KdX" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-channels delete \
    --id slch_011CZkZmRt4vNq8sHbYw2KdX
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted_channel = client.beta.organization.claude_tag.slack_channels.delete(
      "slch_011CZkZmRt4vNq8sHbYw2KdX"
  )

  print(deleted_channel.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deletedChannel = await client.beta.organization.claudeTag.slackChannels.delete(
    "slch_011CZkZmRt4vNq8sHbYw2KdX"
  );

  console.log(deletedChannel.id);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deletedChannel = await client.Beta.Organization.ClaudeTag.SlackChannels.Delete(
      "slch_011CZkZmRt4vNq8sHbYw2KdX"
  );

  Console.WriteLine(deletedChannel.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  deletedChannel, err := client.Beta.Organization.ClaudeTag.SlackChannels.Delete(
  	context.Background(),
  	"slch_011CZkZmRt4vNq8sHbYw2KdX",
  	anthropic.BetaOrganizationClaudeTagSlackChannelDeleteParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(deletedChannel.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var deletedChannel = client.beta().organization().claudeTag().slackChannels()
      .delete("slch_011CZkZmRt4vNq8sHbYw2KdX");

  IO.println(deletedChannel.id());
  ```

  ```php PHP
  $client = new Client();

  $deletedChannel = $client->beta->organization->claudeTag->slackChannels->delete(
      id: 'slch_011CZkZmRt4vNq8sHbYw2KdX',
  );

  echo "{$deletedChannel->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  channel_id = "slch_011CZkZmRt4vNq8sHbYw2KdX"
  deleted_channel = client.beta.organization.claude_tag.slack_channels.delete(channel_id)

  puts deleted_channel.id
  ```
</CodeGroup>

```json Response
{
  "type": "slack_channel_deleted",
  "id": "slch_011CZkZmRt4vNq8sHbYw2KdX"
}
```

请求返回后，该 `id` 立即返回 `404`，其余内容在后台删除。重复移除会返回 `404`，因此如果您在 `503` 之后重新发送移除请求，随后的 `404` 表示该频道已被移除。

## Slack 工作区

工作区对象描述一个 Slack 工作区。它具有与[频道对象](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#slack-channels)相同的字段，只是 `type` 为 `slack_workspace`，`id` 前缀为 `slws_`，`slack` 仅包含 `workspace_id`。[快速入门](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#quick-start)的响应展示了一个完整的工作区对象。使用 `include_inherited=true` 读取时，工作区会显示由工作区和 Slack 默认值解析出的每项设置，这也是工作区中的频道所继承的内容，除非更近的层级设置了该项。

不是有效 `slws_` 标识符的 `id` 会返回 `400`。有效但不指向您组织任何工作区的 `id` 会返回 `404`。

### 列出工作区

`GET /v1/organizations/claude_tag/slack_workspaces` 列出您组织拥有的已设置工作区，按 `created_at` 从新到旧排序。[快速入门](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#quick-start)展示了请求和响应。若要通过 Slack 自身的 ID 查找某个工作区，请添加 `slack_workspace_id`：`data` 包含该工作区，如果未设置则为空；您的组织不拥有的工作区会返回 `404`。您的组织仅通过 Enterprise Grid 拥有的工作区在被设置之前也会返回 `404`。添加 `include_inherited=true` 可使每个工作区包含在其中生效的内容（请参阅[存储值和继承值](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#stored-and-inherited-values)）。需要 `read:claude_tag` 作用域。

### 获取工作区

`GET /v1/organizations/claude_tag/slack_workspaces/{id}` 返回工作区对象，其中包含工作区存储的内容；或者如[读取频道中生效的内容](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#read-what-applies-in-a-channel)那样在查询中使用 `include_inherited=true` 时，包含在其中生效的内容。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/slack_workspaces/slws_011CZkZnWa7xPc3uJfZe5LgT" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-workspaces retrieve \
    --id slws_011CZkZnWa7xPc3uJfZe5LgT
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspace = client.beta.organization.claude_tag.slack_workspaces.retrieve(
      "slws_011CZkZnWa7xPc3uJfZe5LgT"
  )

  print(workspace.settings.instructions)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const workspace = await client.beta.organization.claudeTag.slackWorkspaces.retrieve(
    "slws_011CZkZnWa7xPc3uJfZe5LgT"
  );

  console.log(workspace.settings.instructions);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var workspace = await client.Beta.Organization.ClaudeTag.SlackWorkspaces.Retrieve(
      "slws_011CZkZnWa7xPc3uJfZe5LgT"
  );

  Console.WriteLine(workspace.Settings.Instructions);
  ```

  ```go Go
  client := anthropic.NewClient()

  workspace, err := client.Beta.Organization.ClaudeTag.SlackWorkspaces.Get(
  	context.Background(),
  	"slws_011CZkZnWa7xPc3uJfZe5LgT",
  	anthropic.BetaOrganizationClaudeTagSlackWorkspaceGetParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(workspace.Settings.Instructions)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var workspace = client.beta().organization().claudeTag().slackWorkspaces()
      .retrieve("slws_011CZkZnWa7xPc3uJfZe5LgT");

  IO.println(workspace.settings().instructions().orElse(null));
  ```

  ```php PHP
  $client = new Client();

  $workspace = $client->beta->organization->claudeTag->slackWorkspaces->retrieve(
      id: 'slws_011CZkZnWa7xPc3uJfZe5LgT',
  );

  echo "{$workspace->settings->instructions}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace_id = "slws_011CZkZnWa7xPc3uJfZe5LgT"
  workspace = client.beta.organization.claude_tag.slack_workspaces.retrieve(workspace_id)

  puts workspace.settings.instructions
  ```
</CodeGroup>

### 更新工作区

`POST /v1/organizations/claude_tag/slack_workspaces/{id}` 更改工作区的设置并返回工作区对象。`settings` 为必填项，遵循[更改设置](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#changing-settings)。您在此处设置的内容适用于工作区中未设置自身值的每个频道，但 `instructions` 除外：每个频道会在其上追加自己的指令，而不是替换它。需要 `write:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/claude_tag/slack_workspaces/slws_011CZkZnWa7xPc3uJfZe5LgT" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21" \
    -d '{"settings": {"guest_policy": {"type": "restrict"}}}'
  ```

  ```bash CLI
  ant beta:organization:claude-tag:slack-workspaces update \
    --id slws_011CZkZnWa7xPc3uJfZe5LgT \
    --settings '{guest_policy: {type: restrict}}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  workspace = client.beta.organization.claude_tag.slack_workspaces.update(
      "slws_011CZkZnWa7xPc3uJfZe5LgT",
      settings={"guest_policy": {"type": "restrict"}},
  )

  print(workspace.settings.guest_policy)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const workspace = await client.beta.organization.claudeTag.slackWorkspaces.update(
    "slws_011CZkZnWa7xPc3uJfZe5LgT",
    { settings: { guest_policy: { type: "restrict" } } }
  );

  console.log(workspace.settings.guest_policy?.type);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.ClaudeTag;

  AnthropicClient client = new();

  var workspace = await client.Beta.Organization.ClaudeTag.SlackWorkspaces.Update(
      "slws_011CZkZnWa7xPc3uJfZe5LgT",
      new() { Settings = new() { GuestPolicy = new BetaClaudeTagGuestPolicyRestrict() } }
  );

  Console.WriteLine(workspace.Settings.GuestPolicy?.Type.Raw());
  ```

  ```go Go
  client := anthropic.NewClient()

  workspace, err := client.Beta.Organization.ClaudeTag.SlackWorkspaces.Update(
  	context.Background(),
  	"slws_011CZkZnWa7xPc3uJfZe5LgT",
  	anthropic.BetaOrganizationClaudeTagSlackWorkspaceUpdateParams{
  		Settings: anthropic.BetaClaudeTagSettingsUpdateParam{
  			GuestPolicy: anthropic.BetaClaudeTagGuestPolicyUnionParam{
  				OfRestrict: &anthropic.BetaClaudeTagGuestPolicyRestrictParam{},
  			},
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(workspace.Settings.GuestPolicy.Type)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.BetaClaudeTagGuestPolicyRestrict;
  import com.anthropic.models.beta.organization.claudetag.BetaClaudeTagSettingsUpdate;
  import com.anthropic.models.beta.organization.claudetag.slackworkspaces.SlackWorkspaceUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SlackWorkspaceUpdateParams.builder()
          .settings(BetaClaudeTagSettingsUpdate.builder()
              .guestPolicy(BetaClaudeTagGuestPolicyRestrict.builder().build())
              .build())
          .build();
      var workspace = client.beta().organization().claudeTag().slackWorkspaces()
          .update("slws_011CZkZnWa7xPc3uJfZe5LgT", params);

      IO.println(workspace.settings().guestPolicy().map(policy -> policy.type().asString()).orElse(null));
  }
  ```

  ```php PHP
  $client = new Client();

  $workspace = $client->beta->organization->claudeTag->slackWorkspaces->update(
      id: 'slws_011CZkZnWa7xPc3uJfZe5LgT',
      settings: ['guestPolicy' => ['type' => 'restrict']],
  );

  echo "{$workspace->settings->guestPolicy?->type}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  workspace_id = "slws_011CZkZnWa7xPc3uJfZe5LgT"
  workspace = client.beta.organization.claude_tag.slack_workspaces.update(
    workspace_id,
    settings: {guest_policy: {type: :restrict}}
  )

  puts workspace.settings.guest_policy&.type
  ```
</CodeGroup>

## Bundle

Bundle 即 claude.ai 中所称的[访问包](https://claude.com/docs/claude-tag/admins/add-connections)：一组具名的访问权限，例如连接器、代码仓库和凭据，Claude 可以在频道或工作区中使用它们。组织所有者在 claude.ai 中创建、编辑和删除 Bundle。此 API 用于列出和附加 Bundle。它会返回 Bundle 的名称和描述，但绝不会返回 Bundle 所授予的内容。

| 字段                        | 描述                        |
| ------------------------- | ------------------------- |
| `type`                    | `bundle`。                 |
| `id`                      | 前缀为 `capp_`。              |
| `display_name`            | Bundle 的名称。               |
| `description`             | Bundle 的用途。没有描述时为 `null`。 |
| `created_at`、`updated_at` | Bundle 的创建时间和最后更改时间。      |

不是有效 `capp_` 标识符的 `bundle_id` 会返回 `400`。有效但不存在或属于其他组织的 `bundle_id` 会返回 `404`，无论请求中包含什么其他内容。Claude Tag 为自身管理的 Bundle 不会被列出，并会返回 `404`。

### 列出 Bundle

`GET /v1/organizations/claude_tag/bundles` 列出您组织的 Bundle，按 `created_at` 从新到旧排序。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/bundles" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:bundles list
  ```

  ```python Python
  client = anthropic.Anthropic()

  bundles = client.beta.organization.claude_tag.bundles.list()

  # 根据需要自动获取更多页面。
  for bundle in bundles:
      print(f"{bundle.id}: {bundle.display_name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const bundles = await client.beta.organization.claudeTag.bundles.list();

  for await (const bundle of bundles) {
    console.log(`${bundle.id}: ${bundle.display_name}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.ClaudeTag.Bundles.List();

  await foreach (var bundle in page.Paginate())
  {
      Console.WriteLine($"{bundle.ID}: {bundle.DisplayName}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  bundles := client.Beta.Organization.ClaudeTag.Bundles.ListAutoPaging(
  	context.Background(),
  	anthropic.BetaOrganizationClaudeTagBundleListParams{},
  )

  for bundles.Next() {
  	bundle := bundles.Current()
  	fmt.Printf("%s: %s\n", bundle.ID, bundle.DisplayName)
  }
  if err := bundles.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var bundles = client.beta().organization().claudeTag().bundles().list();

  for (var bundle : bundles.autoPager()) {
      IO.println(bundle.id() + ": " + bundle.displayName());
  }
  ```

  ```php PHP
  $client = new Client();

  $bundles = $client->beta->organization->claudeTag->bundles->list();

  // 仅限本页；如需下一页，请使用 page: $bundles->nextPage 再次调用 list()。
  foreach ($bundles->getItems() as $bundle) {
      echo "{$bundle->id}: {$bundle->displayName}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.claude_tag.bundles.list

  page.auto_paging_each do |bundle|
    puts "#{bundle.id}: #{bundle.display_name}"
  end
  ```
</CodeGroup>

```json Response
{
  "data": [
    {
      "type": "bundle",
      "id": "capp_011CZkZjLP1cWxj5EtMmdgbq",
      "display_name": "Support tools",
      "description": "Issue tracker and the support runbooks repository.",
      "created_at": "2026-09-12T08:15:30Z",
      "updated_at": "2026-09-30T13:02:47Z"
    }
  ],
  "next_page": null
}
```

### 获取 Bundle

`GET /v1/organizations/claude_tag/bundles/{bundle_id}` 返回 Bundle 对象。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/bundles/capp_011CZkZjLP1cWxj5EtMmdgbq" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:bundles retrieve \
    --bundle-id capp_011CZkZjLP1cWxj5EtMmdgbq
  ```

  ```python Python
  client = anthropic.Anthropic()

  bundle = client.beta.organization.claude_tag.bundles.retrieve(
      "capp_011CZkZjLP1cWxj5EtMmdgbq"
  )

  print(bundle.display_name)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const bundle = await client.beta.organization.claudeTag.bundles.retrieve(
    "capp_011CZkZjLP1cWxj5EtMmdgbq"
  );

  console.log(bundle.display_name);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var bundle = await client.Beta.Organization.ClaudeTag.Bundles.Retrieve(
      "capp_011CZkZjLP1cWxj5EtMmdgbq"
  );

  Console.WriteLine(bundle.DisplayName);
  ```

  ```go Go
  client := anthropic.NewClient()

  bundle, err := client.Beta.Organization.ClaudeTag.Bundles.Get(
  	context.Background(),
  	"capp_011CZkZjLP1cWxj5EtMmdgbq",
  	anthropic.BetaOrganizationClaudeTagBundleGetParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(bundle.DisplayName)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var bundle = client.beta().organization().claudeTag().bundles()
      .retrieve("capp_011CZkZjLP1cWxj5EtMmdgbq");

  IO.println(bundle.displayName());
  ```

  ```php PHP
  $client = new Client();

  $bundle = $client->beta->organization->claudeTag->bundles->retrieve(
      bundleID: 'capp_011CZkZjLP1cWxj5EtMmdgbq',
  );

  echo "{$bundle->displayName}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  bundle_id = "capp_011CZkZjLP1cWxj5EtMmdgbq"
  bundle = client.beta.organization.claude_tag.bundles.retrieve(bundle_id)

  puts bundle.display_name
  ```
</CodeGroup>

## Bundle 目标

Bundle 目标是一个 Bundle 附加到一个频道或一个工作区的记录。附加到工作区的 Bundle 可以在该工作区及其中的每个频道中使用。附加到频道的 Bundle 只能在该频道中使用。

| 字段                   | 描述                                                  |
| -------------------- | --------------------------------------------------- |
| `type`               | `slack_channel` 或 `slack_workspace`。                |
| `id`                 | 前缀为 `btgt_`。它是路径中的 `{target_id}`，绝不是频道或工作区自身的 `id`。 |
| `slack_channel_id`   | 在 `slack_channel` 目标上：频道的 `id`，前缀为 `slch_`。         |
| `slack_workspace_id` | 在 `slack_workspace` 目标上：工作区的 `id`，前缀为 `slws_`。      |
| `created_at`         | Bundle 的附加时间。                                       |

一个 Bundle 最多只能附加到某个频道或工作区一次，并且同一组配对始终具有相同的 `id`，即使在分离后重新附加也是如此。该 `id` 是不透明的：请从附加操作的响应或列表中获取它。Bundle 目标不包含其 Bundle 的 `id`，而其路径同样需要该 `id`，因此请将两者一起保存。

附加或分离不会更改任何存储的设置。正在该频道或工作区中运行的会话会收到其访问权限已变化的通知。

claude.ai 在值班（on-call）设置中附加的 Bundle 会被列出且可以读取，但无法在此处附加或分离；对于由值班设置管理的频道或工作区，也无法在其上附加或分离任何内容。这些请求均会返回 `400`。

### 列出 Bundle 的目标

`GET /v1/organizations/claude_tag/bundles/{bundle_id}/targets` 列出该 Bundle 所附加到的频道和工作区，按 `created_at` 从新到旧排序。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/bundles/capp_011CZkZjLP1cWxj5EtMmdgbq/targets" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:bundles:targets list \
    --bundle-id capp_011CZkZjLP1cWxj5EtMmdgbq
  ```

  ```python Python
  client = anthropic.Anthropic()

  targets = client.beta.organization.claude_tag.bundles.targets.list(
      "capp_011CZkZjLP1cWxj5EtMmdgbq"
  )

  # 根据需要自动获取更多页面。
  for target in targets:
      print(f"{target.id}: {target.type}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const targets = await client.beta.organization.claudeTag.bundles.targets.list(
    "capp_011CZkZjLP1cWxj5EtMmdgbq"
  );

  for await (const target of targets) {
    console.log(`${target.id}: ${target.type}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.ClaudeTag.Bundles.Targets.List(
      "capp_011CZkZjLP1cWxj5EtMmdgbq"
  );

  await foreach (var target in page.Paginate())
  {
      Console.WriteLine($"{target.ID}: {target.Type.Raw()}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  targets := client.Beta.Organization.ClaudeTag.Bundles.Targets.ListAutoPaging(
  	context.Background(),
  	"capp_011CZkZjLP1cWxj5EtMmdgbq",
  	anthropic.BetaOrganizationClaudeTagBundleTargetListParams{},
  )

  for targets.Next() {
  	target := targets.Current()
  	fmt.Printf("%s: %s\n", target.ID, target.Type)
  }
  if err := targets.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var targets = client.beta().organization().claudeTag().bundles().targets()
      .list("capp_011CZkZjLP1cWxj5EtMmdgbq");

  for (var target : targets.autoPager()) {
      IO.println(target.id() + ": " + target.type().asString());
  }
  ```

  ```php PHP
  $client = new Client();

  $targets = $client->beta->organization->claudeTag->bundles->targets->list(
      bundleID: 'capp_011CZkZjLP1cWxj5EtMmdgbq',
  );

  // 仅限本页；如需下一页，请使用 page: $targets->nextPage 再次调用 list()。
  foreach ($targets->getItems() as $target) {
      echo "{$target->id}: {$target->type}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  bundle_id = "capp_011CZkZjLP1cWxj5EtMmdgbq"
  page = client.beta.organization.claude_tag.bundles.targets.list(bundle_id)

  page.auto_paging_each do |target|
    puts "#{target.id}: #{target.type}"
  end
  ```
</CodeGroup>

```json Response
{
  "data": [
    {
      "type": "slack_channel",
      "id": "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
      "slack_channel_id": "slch_011CZkZmRt4vNq8sHbYw2KdX",
      "created_at": "2026-10-19T16:21:44Z"
    },
    {
      "type": "slack_workspace",
      "id": "btgt_011CZkZuMc3dVk7xQpGw9StZ1CZkZxRe6gTk9uNyVs4adJ",
      "slack_workspace_id": "slws_011CZkZnWa7xPc3uJfZe5LgT",
      "created_at": "2026-09-12T08:17:02Z"
    }
  ],
  "next_page": null
}
```

仅列出附加到频道和工作区的记录。在 claude.ai 中附加到其他对象（例如**默认 Slack 访问权限**）的 Bundle 没有对应的 Bundle 目标，将某个 Bundle 同时应用于多个频道的规则也没有。因此，Claude 在某个频道中可以使用的 Bundle 可能多于指向该频道的目标。没有任何端点可以列出附加到某个频道或工作区的 Bundle。

### 附加 Bundle

`POST /v1/organizations/claude_tag/bundles/{bundle_id}/targets` 附加该 Bundle 并返回 Bundle 目标。请求体指定 `type` 及相应的 `id`。频道或工作区必须已经完成设置：附加操作永远不会设置频道。附加一个已附加的配对会返回 `200` 以及现有的 Bundle 目标。需要 `write:claude_tag` 作用域。

将 Bundle 附加到频道：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/claude_tag/bundles/capp_011CZkZjLP1cWxj5EtMmdgbq/targets" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21" \
    -d '{
      "type": "slack_channel",
      "slack_channel_id": "slch_011CZkZmRt4vNq8sHbYw2KdX"
    }'
  ```

  ```bash CLI
  ant beta:organization:claude-tag:bundles:targets create \
    --bundle-id capp_011CZkZjLP1cWxj5EtMmdgbq \
    --target '{type: slack_channel, slack_channel_id: slch_011CZkZmRt4vNq8sHbYw2KdX}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  target = client.beta.organization.claude_tag.bundles.targets.create(
      "capp_011CZkZjLP1cWxj5EtMmdgbq",
      target={
          "type": "slack_channel",
          "slack_channel_id": "slch_011CZkZmRt4vNq8sHbYw2KdX",
      },
  )

  print(target.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const target = await client.beta.organization.claudeTag.bundles.targets.create(
    "capp_011CZkZjLP1cWxj5EtMmdgbq",
    { target: { type: "slack_channel", slack_channel_id: "slch_011CZkZmRt4vNq8sHbYw2KdX" } }
  );

  console.log(target.id);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.ClaudeTag.Bundles.Targets;

  AnthropicClient client = new();

  var target = await client.Beta.Organization.ClaudeTag.Bundles.Targets.Create(
      "capp_011CZkZjLP1cWxj5EtMmdgbq",
      new()
      {
          Target = new BetaClaudeTagSlackChannelBundleTargetCreate
          {
              SlackChannelID = "slch_011CZkZmRt4vNq8sHbYw2KdX",
          },
      }
  );

  Console.WriteLine(target.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  target, err := client.Beta.Organization.ClaudeTag.Bundles.Targets.New(
  	context.Background(),
  	"capp_011CZkZjLP1cWxj5EtMmdgbq",
  	anthropic.BetaOrganizationClaudeTagBundleTargetNewParams{
  		Target: anthropic.BetaClaudeTagBundleTargetCreateUnionParam{
  			OfSlackChannel: &anthropic.BetaClaudeTagSlackChannelBundleTargetCreateParam{
  				SlackChannelID: "slch_011CZkZmRt4vNq8sHbYw2KdX",
  			},
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(target.ID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.bundles.targets.TargetCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = TargetCreateParams.builder()
          .slackChannelTarget("slch_011CZkZmRt4vNq8sHbYw2KdX")
          .build();
      var target = client.beta().organization().claudeTag().bundles().targets()
          .create("capp_011CZkZjLP1cWxj5EtMmdgbq", params);

      IO.println(target.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $target = $client->beta->organization->claudeTag->bundles->targets->create(
      bundleID: 'capp_011CZkZjLP1cWxj5EtMmdgbq',
      target: [
          'type' => 'slack_channel',
          'slackChannelID' => 'slch_011CZkZmRt4vNq8sHbYw2KdX',
      ],
  );

  echo "{$target->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  bundle_id = "capp_011CZkZjLP1cWxj5EtMmdgbq"
  target = client.beta.organization.claude_tag.bundles.targets.create(
    bundle_id,
    target: {
      type: :slack_channel,
      slack_channel_id: "slch_011CZkZmRt4vNq8sHbYw2KdX"
    }
  )

  puts target.id
  ```
</CodeGroup>

```json Response
{
  "type": "slack_channel",
  "id": "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
  "slack_channel_id": "slch_011CZkZmRt4vNq8sHbYw2KdX",
  "created_at": "2026-10-19T16:21:44Z"
}
```

将 Bundle 附加到工作区：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/claude_tag/bundles/capp_011CZkZjLP1cWxj5EtMmdgbq/targets" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21" \
    -d '{
      "type": "slack_workspace",
      "slack_workspace_id": "slws_011CZkZnWa7xPc3uJfZe5LgT"
    }'
  ```

  ```bash CLI
  ant beta:organization:claude-tag:bundles:targets create \
    --bundle-id capp_011CZkZjLP1cWxj5EtMmdgbq \
    --target '{type: slack_workspace, slack_workspace_id: slws_011CZkZnWa7xPc3uJfZe5LgT}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  target = client.beta.organization.claude_tag.bundles.targets.create(
      "capp_011CZkZjLP1cWxj5EtMmdgbq",
      target={
          "type": "slack_workspace",
          "slack_workspace_id": "slws_011CZkZnWa7xPc3uJfZe5LgT",
      },
  )

  print(target.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const target = await client.beta.organization.claudeTag.bundles.targets.create(
    "capp_011CZkZjLP1cWxj5EtMmdgbq",
    { target: { type: "slack_workspace", slack_workspace_id: "slws_011CZkZnWa7xPc3uJfZe5LgT" } }
  );

  console.log(target.id);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.ClaudeTag.Bundles.Targets;

  AnthropicClient client = new();

  var target = await client.Beta.Organization.ClaudeTag.Bundles.Targets.Create(
      "capp_011CZkZjLP1cWxj5EtMmdgbq",
      new()
      {
          Target = new BetaClaudeTagSlackWorkspaceBundleTargetCreate
          {
              SlackWorkspaceID = "slws_011CZkZnWa7xPc3uJfZe5LgT",
          },
      }
  );

  Console.WriteLine(target.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  target, err := client.Beta.Organization.ClaudeTag.Bundles.Targets.New(
  	context.Background(),
  	"capp_011CZkZjLP1cWxj5EtMmdgbq",
  	anthropic.BetaOrganizationClaudeTagBundleTargetNewParams{
  		Target: anthropic.BetaClaudeTagBundleTargetCreateUnionParam{
  			OfSlackWorkspace: &anthropic.BetaClaudeTagSlackWorkspaceBundleTargetCreateParam{
  				SlackWorkspaceID: "slws_011CZkZnWa7xPc3uJfZe5LgT",
  			},
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(target.ID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.bundles.targets.TargetCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = TargetCreateParams.builder()
          .slackWorkspaceTarget("slws_011CZkZnWa7xPc3uJfZe5LgT")
          .build();
      var target = client.beta().organization().claudeTag().bundles().targets()
          .create("capp_011CZkZjLP1cWxj5EtMmdgbq", params);

      IO.println(target.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $target = $client->beta->organization->claudeTag->bundles->targets->create(
      bundleID: 'capp_011CZkZjLP1cWxj5EtMmdgbq',
      target: [
          'type' => 'slack_workspace',
          'slackWorkspaceID' => 'slws_011CZkZnWa7xPc3uJfZe5LgT',
      ],
  );

  echo "{$target->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  bundle_id = "capp_011CZkZjLP1cWxj5EtMmdgbq"
  target = client.beta.organization.claude_tag.bundles.targets.create(
    bundle_id,
    target: {
      type: :slack_workspace,
      slack_workspace_id: "slws_011CZkZnWa7xPc3uJfZe5LgT"
    }
  )

  puts target.id
  ```
</CodeGroup>

附加到频道的记录带有其生效条件，例如仅在频道中没有 Slack 访客时生效。条件在 claude.ai 中设置，不会在此处返回。通过此 API 附加时使用默认条件，而附加一个已附加的配对不会更改其条件。

### 获取 Bundle 目标

`GET /v1/organizations/claude_tag/bundles/{bundle_id}/targets/{target_id}` 返回 Bundle 目标。不是有效 `btgt_` 标识符的 `target_id` 会返回 `400`。有效但属于另一个 Bundle、已被分离，或指向不再处于设置状态的频道或工作区的 `target_id` 会返回 `404`。需要 `read:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/claude_tag/bundles/capp_011CZkZjLP1cWxj5EtMmdgbq/targets/btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:bundles:targets retrieve \
    --bundle-id capp_011CZkZjLP1cWxj5EtMmdgbq \
    --target-id btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH
  ```

  ```python Python
  client = anthropic.Anthropic()

  target = client.beta.organization.claude_tag.bundles.targets.retrieve(
      "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
      bundle_id="capp_011CZkZjLP1cWxj5EtMmdgbq",
  )

  print(target.type)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const target = await client.beta.organization.claudeTag.bundles.targets.retrieve(
    "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
    { bundle_id: "capp_011CZkZjLP1cWxj5EtMmdgbq" }
  );

  console.log(target.type);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var target = await client.Beta.Organization.ClaudeTag.Bundles.Targets.Retrieve(
      "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
      new() { BundleID = "capp_011CZkZjLP1cWxj5EtMmdgbq" }
  );

  Console.WriteLine(target.Type.Raw());
  ```

  ```go Go
  client := anthropic.NewClient()

  target, err := client.Beta.Organization.ClaudeTag.Bundles.Targets.Get(
  	context.Background(),
  	"btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
  	anthropic.BetaOrganizationClaudeTagBundleTargetGetParams{
  		BundleID: "capp_011CZkZjLP1cWxj5EtMmdgbq",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(target.Type)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.bundles.targets.TargetRetrieveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = TargetRetrieveParams.builder()
          .bundleId("capp_011CZkZjLP1cWxj5EtMmdgbq")
          .build();
      var target = client.beta().organization().claudeTag().bundles().targets()
          .retrieve("btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH", params);

      IO.println(target.type().asString());
  }
  ```

  ```php PHP
  $client = new Client();

  $target = $client->beta->organization->claudeTag->bundles->targets->retrieve(
      targetID: 'btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH',
      bundleID: 'capp_011CZkZjLP1cWxj5EtMmdgbq',
  );

  echo "{$target->type}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  bundle_id = "capp_011CZkZjLP1cWxj5EtMmdgbq"
  target_id = "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH"
  target = client.beta.organization.claude_tag.bundles.targets.retrieve(
    target_id,
    bundle_id: bundle_id
  )

  puts target.type
  ```
</CodeGroup>

### 分离 Bundle

`DELETE /v1/organizations/claude_tag/bundles/{bundle_id}/targets/{target_id}` 分离 Bundle。重复分离会返回 `404`，因此在 `503` 之后，重新发送的分离请求若返回 `404`，表示该 Bundle 目标已不存在。需要 `write:claude_tag` 作用域。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/claude_tag/bundles/capp_011CZkZjLP1cWxj5EtMmdgbq/targets/btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: experimental-claude-tag-2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:claude-tag:bundles:targets delete \
    --bundle-id capp_011CZkZjLP1cWxj5EtMmdgbq \
    --target-id btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted_target = client.beta.organization.claude_tag.bundles.targets.delete(
      "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
      bundle_id="capp_011CZkZjLP1cWxj5EtMmdgbq",
  )

  print(deleted_target.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deletedTarget = await client.beta.organization.claudeTag.bundles.targets.delete(
    "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
    { bundle_id: "capp_011CZkZjLP1cWxj5EtMmdgbq" }
  );

  console.log(deletedTarget.id);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deletedTarget = await client.Beta.Organization.ClaudeTag.Bundles.Targets.Delete(
      "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
      new() { BundleID = "capp_011CZkZjLP1cWxj5EtMmdgbq" }
  );

  Console.WriteLine(deletedTarget.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  deletedTarget, err := client.Beta.Organization.ClaudeTag.Bundles.Targets.Delete(
  	context.Background(),
  	"btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH",
  	anthropic.BetaOrganizationClaudeTagBundleTargetDeleteParams{
  		BundleID: "capp_011CZkZjLP1cWxj5EtMmdgbq",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(deletedTarget.ID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.claudetag.bundles.targets.TargetDeleteParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = TargetDeleteParams.builder()
          .bundleId("capp_011CZkZjLP1cWxj5EtMmdgbq")
          .build();
      var deletedTarget = client.beta().organization().claudeTag().bundles().targets()
          .delete("btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH", params);

      IO.println(deletedTarget.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $deletedTarget = $client->beta->organization->claudeTag->bundles->targets->delete(
      targetID: 'btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH',
      bundleID: 'capp_011CZkZjLP1cWxj5EtMmdgbq',
  );

  echo "{$deletedTarget->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  bundle_id = "capp_011CZkZjLP1cWxj5EtMmdgbq"
  target_id = "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH"
  deleted_target = client.beta.organization.claude_tag.bundles.targets.delete(
    target_id,
    bundle_id: bundle_id
  )

  puts deleted_target.id
  ```
</CodeGroup>

```json Response
{
  "type": "bundle_target_deleted",
  "id": "btgt_011CZkZpBv9yQd6wKhAr8MnU1CZkZwPd5fSj8tMxUr3ZcH"
}
```

## 无法识别的值

`guest_policy.type`、来源的 `type`、Bundle 目标的 `type` 以及错误代码随时可能新增取值。对于您无法识别的值，请像对待任何未知字符串一样处理，而不是直接失败。

## 速率限制

请求按组织进行限制，分为两个预算，由您组织的所有密钥共享，并与您组织的其他 Admin API 限制相互独立。每个预算允许一次突发，然后以稳定速率补充：

| 预算 | 请求                   | 突发  | 补充     |
| -- | -------------------- | --- | ------ |
| 读取 | 每个 `GET`             | 120 | 每秒 2 个 |
| 更改 | 每个 `POST` 和 `DELETE` | 60  | 每秒 1 个 |

超出预算的请求会返回 **429 Too Many Requests**，并带有 `retry-after` 标头：等待相应的秒数，然后再次发送请求。已到达其预算检查阶段的请求，无论结果如何，都会获得该预算的 `anthropic-ratelimit-requests-limit`、`anthropic-ratelimit-requests-remaining` 和 `anthropic-ratelimit-requests-reset` 标头；在此之前就被拒绝的请求（例如因其密钥、作用域或标头，或因 API、更改或新存储未对您的组织启用）则不会获得这些标头。

## 分页

列表端点使用**不透明游标**。第一个请求最多返回 `limit` 行以及一个 `next_page` 游标；在下一个请求中将该游标原样作为 `page` 参数传递，其他查询参数保持不变，并重复此过程直到 `next_page` 为 `null`。如果发送游标时使用的 `include_inherited` 值与返回该游标的请求不同，会返回 `400`。请将游标字符串视为不透明：不要自行解析、修改或构造它。[列出频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-channels)可能返回少于 `limit` 个频道甚至没有频道的页面，而 `next_page` 仍被设置，因此请持续请求页面直到 `next_page` 为 `null`。Python、TypeScript、C#、Go、Java 和 Ruby SDK 的列表迭代器以及 CLI（使用 `--format raw` 时除外）会为您完成此操作。在 PHP 和 curl 中，请自行请求每个页面，并将其 `next_page` 作为 `page` 传递。

`limit` 默认为 20，最小为 1，最大为 100；超出该范围的值在每个列表上都会返回 `400`。每个列表都按 `created_at` 从新到旧排序，且不接受其他排序方式。

## 错误响应

错误响应遵循 [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)中记录的标准格式。联系支持时，请提供响应正文中的 `request_id`。

| 状态  | 含义                                                                                                                                                                |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400 | 输入无效，或该操作不适用于此频道、工作区或 Bundle（请参阅各端点的章节）。                                                                                                                          |
| 401 | 缺少 `x-api-key` 标头，或密钥无法识别。                                                                                                                                        |
| 403 | 密钥不是组织级别的作用域密钥、缺少所需的作用域，或您的组织无法进行此更改。                                                                                                                             |
| 404 | 未找到资源，或资源不属于您的组织。当请求缺少 `anthropic-beta` 值或 API 未对您的组织启用时也会返回，因此这些端点看起来不存在。                                                                                        |
| 409 | 在请求运行期间，其他操作更改或移除了该资源。请求所要求的内容均未执行：没有存储设置，没有附加、分离或移除任何内容。唯一的例外是设置请求，它可能在其 `settings` 遇到冲突之前已经设置了频道（`slack_channel_set_up_settings_conflict`）。请重新读取资源，如仍需要则重新发送请求。 |
| 429 | 超出速率限制。请参阅[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#rate-limiting)。                                                              |
| 500 | 内部错误。                                                                                                                                                             |
| 503 | 临时错误。对于更改操作，它不说明更改是否已存储。每次更改都可以安全重复，因此请原样再次发送请求。如果重新发送的 `DELETE` 随后返回 `404`，则该频道或 Bundle 目标已不存在。                                                                  |

当同一状态有多种需要您以不同方式处理的原因时，错误中还会包含 `error.details.error_code`：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "...",
    "details": {
      "error_code": "slack_channel_set_up_settings_refused",
      "slack_channel_id": "slch_011CZkZmRt4vNq8sHbYw2KdX"
    }
  },
  "request_id": "req_018EeWyXxfu5pfWkrYcMdjWG"
}
```

| `error_code`                                 | 状态  | 含义及处理方法                                                                                                                                                                                                                     |
| -------------------------------------------- | --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `organization_level_scoped_api_key_required` | 403 | 凭据不是属于整个组织的作用域 Admin API 密钥。请向您的主所有者申请一个。                                                                                                                                                                                   |
| `api_key_scope_missing`                      | 403 | 密钥缺少端点所需的 `read:claude_tag` 或 `write:claude_tag`。                                                                                                                                                                           |
| `claude_tag_writes_not_enabled`              | 403 | 您的组织可以通过此 API 读取，但未启用更改。                                                                                                                                                                                                    |
| `compliance_configuration_restricted`        | 403 | 您的组织具有使 Claude Tag 不存储任何新内容的合规配置，例如零数据保留。设置频道、更新设置（即使是清除设置）以及附加 Bundle 都会被拒绝。分离 Bundle 和移除频道不会被拒绝，并且此类组织即使在 Claude 在频道中开启时也可以[移除](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#remove-a-channel)该频道。 |
| `environment_not_found`                      | 404 | `environment_id` 未指向您的组织可在此处使用的任何环境。路径中的 `id` 没有问题。                                                                                                                                                                         |
| `environment_archived`                       | 400 | `environment_id` 指向已归档的环境。                                                                                                                                                                                                  |
| `slack_channel_set_up_settings_refused`      | 400 | 请求设置了频道，随后 `settings` 被拒绝。`details.slack_channel_id` 是该频道。请参阅[设置频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)。                                                                  |
| `slack_channel_set_up_environment_not_found` | 404 | 请求设置了频道，随后未找到 `environment_id`。`details.slack_channel_id` 是该频道。请参阅[设置频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)。                                                             |
| `slack_channel_set_up_settings_conflict`     | 409 | 请求设置了频道，随后另一更改与 `settings` 重叠。`settings` 中的任何内容都未存储。`details.slack_channel_id` 是该频道。请参阅[设置频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#set-up-a-channel)。                                        |
| `slack_channel_enabled`                      | 400 | Claude 在该频道中处于开启状态（`enabled` 解析为 `true`），因此无法移除该频道。请先将 `enabled` 设置为 `false`。请参阅[移除频道](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#remove-a-channel)。                                               |

可能会添加新的错误代码。对于无法识别的代码，请按其状态进行处理。不带 `error_code` 的 `404` 有两种可能的原因。要么是请求指定的某项内容未找到（路径中的 `id`，或请求体或查询中的 Slack 频道或工作区），要么是因为请求缺少 `anthropic-beta` 值或 API 未对您的组织启用，导致每个端点看起来都不存在。要区分两者，请使用相同的密钥和标头[列出您的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/claude-tag-api#list-workspaces)：该请求不指定任何内容，因此只有在第二种情况下才会返回 `404`。

## 活动源事件

通过此 API 进行的每项更改都会记录在您组织的 [Compliance API 活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)中，并归属于该 API 密钥，以携带其 `apikey_` ID 的 `api_actor` 形式记录。这些事件与在 claude.ai 中进行相同更改时活动源所记录的事件相同，因此无论更改是通过哪种方式进行的，一次查询即可找到它。

| 事件                                | 触发时机                            |
| --------------------------------- | ------------------------------- |
| `ccr_agent_created`               | 频道被设置。                          |
| `ccr_agent_updated`               | 频道或工作区上的设置发生变化。                 |
| `ccr_agent_deleted`               | 频道被移除。其 Bundle 目标随之移除，不会单独触发事件。 |
| `ccr_agent_proxy_profile_bound`   | Bundle 被附加。                     |
| `ccr_agent_proxy_profile_unbound` | Bundle 被分离。                     |

这些事件通过 Feed 自身的标识符 `agent_id`（前缀为 `cagt_`）指明频道或工作区，而此 API 的任何响应都不包含该标识符，因此您目前还无法将事件与频道或工作区的 `id` 对应起来。两个 Bundle 事件在 `profile_id` 中指明 Bundle，并在 `scope_id` 中以相同的 `agent_id` 指明频道或工作区。

`ccr_agent_updated` 事件会在 `updated_fields` 中列出所更改的内容，使用的是活动源自己的名称：

| 设置               | 在 `updated_fields` 中的名称  |
| ---------------- | ------------------------ |
| `enabled`        | `slackbot_version`       |
| `instructions`   | `system_prompt_addendum` |
| `model`          | `default_model`          |
| `environment_id` | `default_environment_id` |
| `guest_policy`   | `guest_policy`           |

`updated_fields` 还可能包含 `is_virtual`，它不对应任何设置。当通过此 API 进行的更改是任何人首次在频道或工作区上存储内容时，它会与已更改设置的名称一起出现在该事件中：即由 Claude 首次使用设置、或由此 API 在不带 `settings` 的情况下设置的频道，或从未使用过 Claude 的工作区，且尚无人在此处或 claude.ai 中对其存储设置或附加 Bundle。代码频道启动时使用的 `model` 不算作已存储。附加 Bundle 仅记录 `ccr_agent_proxy_profile_bound`。

读取操作、未更改任何内容的请求，以及在存储任何内容之前即被拒绝的请求，都不会记录任何事件。如果一个设置请求设置了频道，但其 `settings` 随后被拒绝，则会记录 `ccr_agent_created`。

## 另请参阅

<CardGroup cols={2}>
  <Card title="创建 Admin API 密钥" href="https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys">
    您的主要所有者在此创建具有作用域的密钥。
  </Card>

  <Card title="Claude Tag 文档" href="https://claude.com/docs/claude-tag/overview">
    了解 Claude Tag 是什么、如何设置，以及每项设置在 Slack 中的作用。
  </Card>

  <Card title="Compliance API 活动源" href="https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed">
    在此处和 claude.ai 中所做的每项更改的审计记录。
  </Card>

  <Card title="插件 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api">
    管理您的 Claude Enterprise 组织中的插件。
  </Card>
</CardGroup>
