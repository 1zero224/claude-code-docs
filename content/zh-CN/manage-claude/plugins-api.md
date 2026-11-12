---
title: 插件 API
url: https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api
description: 清点和管理您的 Claude Enterprise 组织中的插件：上传插件和版本、选择向成员提供的版本、控制谁可以使用每个插件、下载插件文件以供审查，以及在连接市场之前对其进行验证。
---

插件 API 让您可以执行以下操作：

* 盘点您的 Claude Enterprise 组织中的插件
* 从您自己的流水线发布插件和新版本
* 选择向成员提供哪个版本
* 控制谁可以使用每个插件
* 下载插件文件以供审查
* 在连接 Git 市场之前对其进行检查

有关插件*使用情况*报告（成员使用哪些插件和技能，以及使用频率），请参阅 [Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api)。

<Check>
  **需要带作用域的 Admin API 密钥**

  这些端点需要具有 `read:plugins` “scope”（作用域）的 Admin API 密钥（用于 `GET` 端点，包括归档下载），或具有 `write:plugins` 作用域的 Admin API 密钥（用于 `POST` 和 `DELETE` 端点，但市场验证除外，两种作用域均可用于市场验证）；[作用域](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#scopes)部分提供了详细信息，包括另外两个同样适用的读取作用域。有关主所有者在何处创建密钥，请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization)。请在每个请求的 `x-api-key` 标头中传递该密钥，同时附带 [`anthropic-version`](https://platform.claude.com/docs/zh-CN/api/versioning) 标头以及下方说明中所示的 beta 标头。
</Check>

<Note>
  插件 API 目前处于 **beta** 阶段，仅适用于 Claude Enterprise 组织。它不适用于 Claude Platform（Claude Console）组织，也不适用于已启用 HIPAA 就绪的组织。

  每个请求都必须包含 [beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers) `anthropic-beta: ce-plugins-2026-09-01`（SDK 和 `ant` CLI 会为您发送它）。缺少该标头的请求会返回 `404`，就像该端点不存在一样。
</Note>

## 端点

该 API 在五种资源上共提供 18 个端点：

| 资源                                                   | 端点                                                                                                                                                                                                                                                                                                                        |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **插件：** 列出组织中的插件、上传新插件、查询某个插件、选择向成员提供的版本（回滚或升级）、删除插件 | `GET /v1/organizations/plugins`<br />`POST /v1/organizations/plugins`<br />`GET /v1/organizations/plugins/{plugin_id}`<br />`POST /v1/organizations/plugins/{plugin_id}`<br />`DELETE /v1/organizations/plugins/{plugin_id}`                                                                                              |
| **插件版本：** 列出插件的版本历史、上传新版本、查询某个版本、下载某个版本的文件           | `GET /v1/organizations/plugins/{plugin_id}/versions`<br />`POST /v1/organizations/plugins/{plugin_id}/versions`<br />`GET /v1/organizations/plugins/{plugin_id}/versions/{version}`<br />`GET /v1/organizations/plugins/{plugin_id}/versions/{version}/content`                                                           |
| **安装设置：** 读取谁可以使用组织拥有的插件、为整个组织或某个组设置该项、移除任一设置        | `GET /v1/organizations/plugins/{plugin_id}/installation_settings`<br />`POST /v1/organizations/plugins/{plugin_id}/installation_settings/{target}`<br />`DELETE /v1/organizations/plugins/{plugin_id}/installation_settings/{target}`                                                                                     |
| **共享：** 读取成员将自己的插件共享给了谁（只读）                          | `GET /v1/organizations/plugins/{plugin_id}/shares`                                                                                                                                                                                                                                                                        |
| **插件市场：** 查找市场的 ID、查询某个市场、为其插件设置默认安装设置、在连接市场之前检查其内容  | `GET /v1/organizations/plugin_marketplaces`<br />`GET /v1/organizations/plugin_marketplaces/{marketplace_id}`<br />`POST /v1/organizations/plugin_marketplaces/{marketplace_id}`<br />`POST /v1/organizations/plugin_marketplaces/validate_repository`<br />`POST /v1/organizations/plugin_marketplaces/validate_archive` |

Anthropic 发布的插件不会被清点；其使用情况由 [Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api) 报告。市场的创建、与代码仓库的连接以及删除均在 claude.ai 中进行，而不是通过此 API。

## 前提条件

* 您的组织必须使用 Claude Enterprise 计划。
* 您的主要所有者在 [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access) 中创建具有 `read:plugins` 作用域、`write:plugins` 作用域或两者兼有的 Admin API 密钥。请参阅[为 Claude Enterprise 组织创建密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization)。
* 每个请求都携带三个标头：`x-api-key`、`anthropic-version: 2026-10-21` 和 `anthropic-beta: ce-plugins-2026-09-01`。

在 Python、TypeScript、C#、Go、Java、PHP 和 Ruby 中，SDK 在 `client.beta.organization` (csharp, go: `client.Beta.Organization`; java: `client.beta().organization()`; php: `$client->beta->organization`) 下提供这些端点，[`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart) 则在 `ant beta:organization` 下提供；它们会为您发送 `anthropic-version` 和 `anthropic-beta` 标头。本页面上的示例从 `ANTHROPIC_API_KEY` 环境变量读取 Admin API 密钥。当 SDK 或 CLI 为您获取更多页面时，`limit` 设置的是页面大小，而不是总数（请参阅[分页](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#pagination)）。

API 密钥属于组织，在创建者离开后仍然有效。请勿共享这些密钥，也不要将其提交到源代码管理中。

## 快速入门

列出您组织自有市场中的插件，最新的排在最前：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins?owner_type=organization&limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins list --owner-type organization --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  plugins = client.beta.organization.plugins.list(owner_type="organization", limit=20)

  # 根据需要自动获取更多页面。
  for plugin in plugins:
      print(f"{plugin.id}: {plugin.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const plugins = await client.beta.organization.plugins.list({
    owner_type: "organization",
    limit: 20
  });

  for await (const plugin of plugins) {
    console.log(`${plugin.id}: ${plugin.name}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Plugins;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Plugins.List(
      new() { OwnerType = OwnerType.Organization, Limit = 20 }
  );

  await foreach (var plugin in page.Paginate())
  {
      Console.WriteLine($"{plugin.ID}: {plugin.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  plugins := client.Beta.Organization.Plugins.ListAutoPaging(context.Background(), anthropic.BetaOrganizationPluginListParams{
  	OwnerType: anthropic.BetaOrganizationPluginListParamsOwnerTypeOrganization,
  	Limit:     anthropic.Int(20),
  })

  for plugins.Next() {
  	plugin := plugins.Current()
  	fmt.Printf("%s: %s\n", plugin.ID, plugin.Name)
  }
  if err := plugins.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.plugins.PluginListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = PluginListParams.builder()
          .ownerType(PluginListParams.OwnerType.ORGANIZATION)
          .limit(20)
          .build();
      var plugins = client.beta().organization().plugins().list(params);

      for (var plugin : plugins.autoPager()) {
          IO.println(plugin.id() + ": " + plugin.name());
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\Plugins\PluginListParams\OwnerType;
  // ...

  $client = new Client();

  $plugins = $client->beta->organization->plugins->list(
      limit: 20,
      ownerType: OwnerType::ORGANIZATION,
  );

  // 仅限当前页；如需下一页，请使用 page: $plugins->nextPage 再次调用 list()。
  foreach ($plugins->getItems() as $plugin) {
      echo "{$plugin->id}: {$plugin->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.plugins.list(owner_type: :organization, limit: 20)

  page.auto_paging_each do |plugin|
    puts "#{plugin.id}: #{plugin.name}"
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "type": "plugin",
      "id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
      "name": "sales-toolkit",
      "display_name": "Sales Toolkit",
      "description": "Account research and call prep for the sales team.",
      "served_version_id": "pluginver_01Km7tL4pR9xF5sU2zV3jP6q",
      "served_version_pinned": true,
      "latest_version_id": "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p",
      "manifest_version": "1.4.0",
      "owner": { "type": "organization" },
      "marketplace_id": "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
      "created_by": { "type": "api_actor", "api_key_id": "apikey_01Nq9vN6rT2zH7uW4bX5mR8s" },
      "organization_installation_preference": "available",
      "organization_installation_preference_inherited": true,
      "content_scan": { "status": "completed", "assessment": "pass", "reason": null },
      "components": [
        {
          "type": "skill",
          "name": "account-research",
          "description": "Researches a customer account before a call."
        },
        { "type": "mcp_server", "name": "crm", "description": null }
      ],
      "reach": "remote",
      "created_at": "2026-09-01T17:04:11Z",
      "updated_at": "2026-09-15T14:12:30Z"
    }
  ],
  "next_page": "page_xK9f2LqT7vNw3pRzBd8sHy"
}
```

在此示例中，插件被固定到较早的版本：一个较新的版本（`latest_version_id`）已存储，但尚未提供给成员。

## 作用域

| 作用域                        | 授予的权限                                                                                                                                                                                                                               |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `read:plugins`             | 本页上的每个 `GET` 端点（包括归档下载），以及市场验证。                                                                                                                                                                                                     |
| `write:plugins`            | 本页上的每个 `POST` 和 `DELETE` 端点：创建插件、创建版本、更改提供的版本、删除插件、设置和移除安装设置、设置市场的默认值，以及市场验证。它不授予读取权限。                                                                                                                                              |
| `read:org_audit`           | 用于安全审计集成的只读作用域：本页上的每个 `GET` 端点（包括归档下载），以及[用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management)和 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的读取端点。它不授予市场验证或任何写入权限。 |
| `read:compliance_org_data` | Compliance API 用于组织元数据（名称、类型、角色和组）及有效设置的作用域。它授予本页上的每个 `GET` 端点，与 `read:org_audit` 完全相同，因此 Compliance Access Key 无需第二个密钥即可读取插件。它不授予市场验证或任何写入权限。                                                                                      |

一个密钥可以携带多个作用域。上传插件后再将其读回的集成需要同时具有 `read:plugins` 和 `write:plugins`。本页中凡是提到某个端点需要 `read:plugins` 作用域的地方，具有 `read:org_audit` 或 `read:compliance_org_data` 的密钥同样适用。

### 访问成员的插件文件

这些读取作用域（`read:plugins`、`read:org_audit` 和 `read:compliance_org_data`）中的每一个都可以下载成员个人市场中插件的文件，包括 claude.ai 管理设置中不显示的文件；而绑定到您的父组织的 `read:org_audit` 或 `read:compliance_org_data` 密钥，可以通过传递 `organization_id`，在其下任何有权访问此 API 的组织中执行此操作（请参阅[读取同一父组织下的其他组织](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#reading-another-organization-under-the-same-parent)）。每次此类下载都会在 [Compliance API 活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)上记录一个 `claude_plugin_archive_accessed` 事件，标识密钥、插件、版本和成员（请参阅[活动源事件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#activity-feed-events)）。下载组织拥有的插件不会被记录。

### 读取同一父组织下的其他组织

`read:plugins` 和 `write:plugins` 密钥只能读写创建它们的组织。有些公司有多个 Claude 组织链接在一个父组织下。[为所有链接的组织创建的](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization) `read:org_audit` 或 `read:compliance_org_data` 密钥还可以读取任何有权访问此 API 的链接组织。要读取某个组织，请在本页面上任何 `GET` 端点的 `organization_id` 查询参数中传递其 ID。该 ID 是 claude.ai 设置中显示的组织 UUID（也接受其带 `org_` 前缀的形式）。如果不带该参数，密钥将读取创建它的组织。没有自身组织的密钥必须始终传递 `organization_id`。`404` 表示指定的组织不在密钥的父组织下，或者该组织无法使用此 API；不是 UUID 或 `org_` ID 的值会返回 `400`。任何其他密钥如果指定了非其自身的组织，都会得到 `404`。写入操作不接受 `organization_id`。

## 关键概念

### 插件和组件

**插件**是为您组织的成员扩展 Claude 功能的软件包。它包含以下组件的任意组合：

| 组件                  | 说明                                                 |
| ------------------- | -------------------------------------------------- |
| Skill（技能）           | Claude 在任务需要时加载的指令和文件。                             |
| Command（命令）         | 成员通过输入 `/` 加命令名称来运行的已保存提示。                         |
| Agent（代理）           | 拥有自己指令的辅助助手，Claude 可以将部分任务交给它处理。                   |
| 钩子                  | 在会话中发生某个事件时（例如在 Claude 使用工具之前）自动运行的命令。             |
| MCP server（MCP 服务器） | 从 Claude 到另一个系统中的工具和数据的连接（Model Context Protocol）。 |
| CLI                 | 插件允许 Claude 运行的命令行程序。                              |

每个插件在 `.claude-plugin/plugin.json` 处都有一个清单。清单中的 `name` 会成为插件的 `name`：一个在其市场内唯一的小写标识符。

### 市场

**市场**是插件的容器。每个市场都有一个所有者和一个来源。

* **所有者。** 组织拥有其市场。每个成员也可以拥有个人市场。
* **来源。** `manual` 表示插件是上传的，可以在 claude.ai 中上传，对于组织市场也可以通过此 API 上传。`github`、`gitlab` 和 `public_git` 表示插件从所有者连接的 Git 仓库同步而来。同步的市场无法上传任何内容，此 API 也无法删除其中的插件，因为下一次同步会撤销这两种更改。请改为修改仓库。

您组织的**库市场**是组织拥有的 `manual` 市场，当您未指定市场时，上传内容会进入该市场。它会在首次有内容上传到其中时创建。

### 组织拥有的插件和成员拥有的插件

插件的 `owner.type` 表明它位于谁的市场中：

* `organization`：您可以通过此 API 管理它，但位于从 Git 同步的市场中的插件无法在此处接收上传或被删除。
* `user`：它位于某个成员的个人市场中。您可以读取其详细信息并下载其文件，如果其市场为 `manual`，还可以删除它。上传版本和选择提供的版本会返回 `403`。共享仅由该成员在 claude.ai 中管理。

将成员从组织中移除不会移除其插件。这些插件会以该成员的 `user_id` 保留在清单中，`owner_user_id` 筛选器仍能找到它们，因此您可以审查和移除已离开成员的内容。当该成员的账户被删除时，它们会从清单中移除。

### 版本和提供的版本

每次上传都会创建一个新的、不可变的 “version”（版本），无论它来自此 API、claude.ai 还是 Git 同步。插件有两个指向其版本的指针：

* `latest_version_id`：最新版本
* `served_version_id`：向成员提供的版本

默认情况下 `served_version_pinned` 为 `false`：提供的版本通常跟随最新版本。当内容扫描开启时，在 claude.ai 中保存的版本可能会**等待**其内容扫描：它会被存储，但不会立即提供。本页面中的"等待"仅用于在 claude.ai 中保存的版本，而不用于成员或管理员发布到插件中的版本。在扫描开启期间，只有当等待中的版本的 `content_scan` 显示为 `completed` 且结果为 `pass` 或 `warn` 时，它才会被提供。当某个版本处于等待状态时，`served_version_id` 指向的版本比 `latest_version_id` 更早。不要轮询直到两个 ID 匹配：它们可能一直不同，例如在扫描失败时。

使用 `POST /v1/organizations/plugins/{plugin_id}` 选择版本会**固定**（pin）该插件（`served_version_pinned: true`）。管理员在 claude.ai 中选择版本，或接受成员发布到该插件的请求，也会产生同样的效果。此后，新上传的内容会被存储并推进 `latest_version_id`，但成员会一直使用固定的版本，直到您将 `served_version_id` 指向另一个版本。两个指针不同的插件，表示有一个已存储但未被提供的版本。

这使发布流水线可以上传每个构建、对其进行测试，然后再提升它。要让您的流水线决定每个构建何时被提供，请通过将 `served_version_id` 设置为其当前版本来固定插件一次；此后，提升您想要提供的每个构建。在内容扫描开启的情况下，当前版本的 `content_scan` 显示为 `processing` 时，首次固定会返回 `409 scan_pending`；如果扫描显示为 `fail`、`unknown` 或 `errored`，则返回 `400 scan_failed`（`warn` 会被接受）。已固定的插件无法取消固定，无论是在此处还是在 claude.ai 中。

要回滚，请将 `served_version_id` 设置为较早的版本。前滚的方式相同。

这些规则描述的是组织拥有的插件。成员拥有的插件的提供版本由其所有者在 claude.ai 中控制。

### 安装设置

**安装设置**决定谁可以使用组织拥有的插件。每个设置有四个值之一，承载在名为 `installation_preference` 的字段中（在插件和市场对象上，则是 `organization_installation_preference` 和 `default_installation_preference`）：

| 值               | 成员看到的情况     |
| --------------- | ----------- |
| `required`      | 插件已安装且无法移除。 |
| `auto_install`  | 插件已安装且可以移除。 |
| `available`     | 插件可按需安装。    |
| `not_available` | 插件被隐藏。      |

一个插件可以拥有一个组织范围的设置，以及每个组各一个设置（即[在用户管理中管理的组](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#groups)，基于角色的访问控制组）。成员按以下规则获得一个值：

1. 组织范围的值为：如果插件有自己的组织范围设置，则取该设置；否则取其市场的默认值；否则为 `not_available`。插件在 `organization_installation_preference` 中报告此值，当该值来自市场默认值时，`organization_installation_preference_inherited: true`。
2. 不属于任何持有该插件设置的组的成员，获得组织范围的值。
3. 属于一个或多个持有设置的组的成员，则改为获得这些组设置中最宽松的一个，排序为 `required`、`auto_install`、`available`、`not_available`。

组的设置会为其成员替换组织范围的值，而不是在其基础上叠加。例如，如果组织范围的值为 `required`，而试点组持有 `available`，则试点成员获得 `available`。当您将插件从试点组推广到整个组织时，请设置组织范围的值，然后移除该组的设置。设置组织范围的值会使插件不再继承其市场默认值；[移除组织范围的设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#remove-an-installation-setting)会使插件恢复为该默认值。

通过此 API 创建的插件一开始没有自己的设置，因此会继承其市场的默认值：除非有人设置了默认值，否则为 `not_available`。删除一个组会从每个插件中移除该组的设置。

### 共享

**共享**决定谁可以使用成员拥有的插件。所有者在 claude.ai 中将其共享给所有成员、某个组或指定成员。此 API 可以列出共享，但无法更改它们。

如果您的组织在其 claude.ai 设置中关闭了某种共享方式，该类共享仍会出现在列表中，但在该设置关闭期间不会向任何人授予访问权限；列表本身不会显示该设置是否关闭。

### 内容扫描

内容扫描是 claude.ai 中的一项组织设置。开启后，新存储的版本会被扫描（claude.ai 会豁免少数版本），结果报告在 `content_scan` 中；未被扫描的版本（例如在开启扫描之前存储的版本）的 `content_scan: null`。使用客户管理的加密密钥或零数据保留的组织无法使用扫描功能。

在扫描开启期间，只有当插件所提供版本的扫描为 `completed` 且结果为 `pass` 或 `warn` 时，才会向成员提供该插件。当扫描处于 `processing` 或 `errored` 状态，或结果为 `fail` 或 `unknown` 时，插件会对成员隐藏，也不会用较早的版本代替它。从未被扫描的版本（`content_scan: null`）会正常提供。

您通过此 API 上传的版本从不等待其内容扫描。当它成为提供的版本时，成员会失去该插件，直到其 `content_scan` 显示为 `completed` 且结果为 `pass` 或 `warn`。如果希望成员在新版本扫描期间继续使用当前版本，请先固定插件（请参阅[版本和提供的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#versions-and-the-served-version)）。

当在 claude.ai 中保存的版本等待其扫描时，提供的版本不会改变。当您的 API 上传成为提供的版本时，任何等待都会结束：等待中的版本只有在有人选择它时才会提供。要查看等待中版本的扫描，请读取该版本（请参阅[列出插件的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#list-a-plugins-versions)）。

上传后，`content_scan.status` 为 `processing`，结论会异步到达。读取版本即可查看；插件对象只显示其所提供版本的扫描结果。扫描开启期间，将提供的版本更改为 `content_scan` 显示为 `processing` 的版本会返回 `409 scan_pending`，更改为扫描显示为 `fail`、`unknown` 或 `errored` 的版本会返回 `400 scan_failed`。

#### 在 claude.ai 中允许的失败扫描

在 claude.ai 中，组织所有者可以允许扫描失败的版本。其[自定义角色](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#custom-roles)可以管理**库**的成员也可以这样做。此 API 随后可能会将该扫描报告为 `completed`，结果为 `warn`，并附带失败扫描的 `reason`。在此期间，适用 `warn` 的规则：该版本可以提供给成员，您也可以将其设为提供的版本。响应中没有任何内容表明它是被允许的，因此 `warn` 既可能是警告，也可能是被允许的失败扫描。允许扫描显示为 `warn` 的版本不会改变其 `content_scan`。当某个版本不再被允许时，此 API 会报告其最新完成扫描的结论。此后扫描显示为 `fail` 或 `unknown` 的已提供版本会对成员隐藏。

### 影响范围

`reach`（影响范围）用一个值概括某个版本在成员计算机上及更远范围内的影响程度：

| 值            | 含义                                                                                                                                                                                                                                                         |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `remote`     | 声明了 MCP 服务器或 CLI，无论它还声明了什么。                                                                                                                                                                                                                                |
| `privileged` | 未声明 MCP 服务器或 CLI，但声明了钩子、监视器（在会话期间持续运行的后台命令）、"Language Server Protocol"（语言服务器协议），即 LSP 服务器，工作流（Claude Code 可以运行的脚本；插件根目录下 `workflows/` 文件夹中的任何文件都算作一个工作流），或插件应用于成员应用的设置，或包含其 frontmatter 预先批准工具（`allowed-tools`）的技能、命令或任何其他 Markdown 文件。这些会在成员自己的计算机上运行或生效。 |
| `contained`  | 未声明 MCP 服务器、CLI、钩子、监视器、LSP 服务器、工作流或应用设置，且其 Markdown 文件均未预先批准工具（例如，仅包含技能、命令和代理且均无 `allowed-tools` 的插件）。                                                                                                                                                     |

`reach` 会计入版本声明的所有内容，包括 `components` 未列出的监视器、LSP 服务器、工作流和应用设置，因此 `components` 列表为空的版本仍可能是 `privileged`。对于在记录组件之前存储的版本，以及由于其某个技能的 `SKILL.md` 或某个命令文件无法读取而无法确定影响范围的版本，它为 `null`；请将 `null` 视为未分类。该值通常在存储版本时根据当时的规则计算得出。因此，较旧的版本可能是 `contained`，而如今存储的相同文件则会是 `privileged`。

### 上传要求

上传遵循与 claude.ai 中插件上传相同的规则，因此两处接受相同的归档文件。

* 上传内容可以是一个 `.zip` 或 `.plugin` 归档文件，也可以是一组单独的文件。归档文件可以将所有内容包裹在一个顶层文件夹中。
* 它必须恰好包含一个清单，位于 `.claude-plugin/plugin.json`，且该清单必须声明 `name`。没有清单的单独 `SKILL.md` 会被拒绝。
* 如果顶层 `SKILL.md` 的 frontmatter 声明了插件组件，则会将其合并到清单中；当两者都设置了某个值时，以 `plugin.json` 为准。清单从该 `SKILL.md` 获取的 `description` 会被截断为前 500 个字符。
* `name` 可以包含小写字母（任何字母表）、数字和连字符，最多 64 个字符。大写字母、空格、下划线和其他标点符号会被拒绝。
* `displayName` 最多 64 个字符，`description` 最多 500 个字符。
* 每个 `SKILL.md` 都需要包含 `name` 和 `description` 的有效 YAML frontmatter，且两者都不能包含 XML 标签（例如 `<example>`）。两个技能或两个命令不能同名。
* 任何文件都不能位于顶层 `bin/` 目录下。
* 不允许嵌套 `.zip` 文件。允许打包的 MCP 服务器（`.mcpb`、`.dxt`）。
* 文件路径必须是相对路径，不能包含 `..`，并且只能使用字母、数字、空格和 `_ . - / ( ) ,`。
* 请求体和未压缩的归档各自最多 200 MB；超过限制的请求体返回 `413`（`request_too_large`）而不是 `400`。一次上传最多包含 5,000 个文件，路径深度最多 12 层，路径最多 472 个字符，文件或文件夹名称最多 255 个字符。
* ZIP 归档文件必须使用 DEFLATE 或 STORE 压缩，且不能加密或包含符号链接。
* 当市场已容纳 1,000 个插件时，创建插件的上传会返回 `400`。此 API 未列出的插件也会计入，因此即使[列出插件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#list-plugins)显示该市场中的插件少于 1,000 个，也可能达到此限制。Anthropic 可以提高 5,000 个文件的限制。

## 示例工作流

### 从发布流水线发布每个构建

从 CI 上传每个带标签的构建，并让流水线决定构建何时被提供。

1. 使用 `GET /v1/organizations/plugin_marketplaces?owner_type=organization` 查找要上传到的市场，或省略 `marketplace_id` 以使用库市场。
2. 首次发布时，使用 `POST /v1/organizations/plugins` [创建插件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#create-a-plugin)。之后的每次发布，先记录插件的 `latest_version_id`，然后使用 `POST /v1/organizations/plugins/{plugin_id}/versions` [上传版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#create-a-version)。如果上传的响应丢失，请读取插件，仅当 `latest_version_id` 未变化时才重试（请参阅[重试上传](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#retrying-uploads)）。
3. 若要在检查每个新构建期间让成员继续使用当前版本，请将 `served_version_id` 设置为插件的当前版本，从而一次性固定该插件。此后每次上传都会被存储但不会被提供，并且固定无法撤销：您希望提供的每个构建都需要执行第 5 步。
4. 启用内容扫描时，轮询 `GET /v1/organizations/plugins/{plugin_id}/versions/{version}`，直到 `content_scan.status` 不再是 `processing`，并且仅在其为 `completed` 且结果为 `pass` 或 `warn` 时才进行升级。
5. 使用 `POST /v1/organizations/plugins/{plugin_id}` 和 `{"served_version_id": "<the new version's ID>"}` [升级该构建](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#change-the-served-version)。要回滚，请以相同方式发送上一个版本的 ID。

### 先向试点组推出插件，再推广到所有人

1. 使用 `GET /v1/organizations/rbac_groups` 查找试点组的 ID。该调用需要 `read:rbac_groups` 作用域（请参阅[用户管理中的群组和角色](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#groups-and-roles)）。后续步骤需要 `write:plugins`，它仅作用于创建其密钥的组织，因此在有多个链接组织的企业中，请在持有该插件的组织中创建一个密钥并赋予其这两个作用域，或者在这些步骤中使用在该组织中创建的第二个密钥。

2. 使用 `POST /v1/organizations/plugins/{plugin_id}/installation_settings/{target}` [为该组设置其自己的设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#set-an-installation-setting)，例如 `auto_install`，其中 `{target}` 是该组的 `rbac_group_` ID，同时组织范围的值保持为 `not_available`。只有该组的成员会获得该插件。

3. 试点结束后，使用以下调用设置组织范围的值，然后移除该组的设置，使该组重新跟随组织。设置组织范围的值会使插件不再继承其市场默认值；之后[移除组织范围的设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#remove-an-installation-setting)会使插件恢复为该默认值。

   <CodeGroup>
     ```bash cURL
     curl -X POST "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/installation_settings/organization" \
       -H "content-type: application/json" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21" \
       -H "anthropic-beta: ce-plugins-2026-09-01" \
       -d '{"installation_preference": "required"}'
     ```

     ```bash CLI
     ant beta:organization:plugins:installation-settings set \
       --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
       --target organization \
       --installation-preference required
     ```

     ```python Python
     client = anthropic.Anthropic()

     setting = client.beta.organization.plugins.installation_settings.set(
         "organization",
         plugin_id="plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
         installation_preference="required",
     )

     print(f"plugin_id: {setting.plugin_id}")
     print(f"installation_preference: {setting.installation_preference}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const setting = await client.beta.organization.plugins.installationSettings.set(
       "organization",
       {
         plugin_id: "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
         installation_preference: "required"
       }
     );

     console.log(`plugin_id: ${setting.plugin_id}`);
     console.log(`installation_preference: ${setting.installation_preference}`);
     ```

     ```csharp C#
     using Anthropic.Models.Beta.Organization.Plugins.InstallationSettings;

     AnthropicClient client = new();

     var setting = await client.Beta.Organization.Plugins.InstallationSettings.Set(
         "organization",
         new()
         {
             PluginID = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
             InstallationPreference = InstallationPreference.Required,
         }
     );

     Console.WriteLine($"plugin_id: {setting.PluginID}");
     Console.WriteLine($"installation_preference: {setting.InstallationPreference.Raw()}");
     ```

     ```go Go
     client := anthropic.NewClient()

     setting, err := client.Beta.Organization.Plugins.InstallationSettings.Set(
     	context.Background(),
     	"organization",
     	anthropic.BetaOrganizationPluginInstallationSettingSetParams{
     		PluginID:               "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
     		InstallationPreference: anthropic.BetaOrganizationPluginInstallationSettingSetParamsInstallationPreferenceRequired,
     	},
     )
     if err != nil {
     	log.Fatal(err)
     }

     fmt.Printf("plugin_id: %s\n", setting.PluginID)
     fmt.Printf("installation_preference: %s\n", setting.InstallationPreference)
     ```

     ```java Java
     import com.anthropic.models.beta.organization.plugins.installationsettings.InstallationSettingSetParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = InstallationSettingSetParams.builder()
             .pluginId("plugin_01Hq3vX8kZcN2mB7pR4tY9wL")
             .installationPreference(InstallationSettingSetParams.InstallationPreference.REQUIRED)
             .build();
         var setting = client.beta().organization().plugins().installationSettings()
             .set("organization", params);

         IO.println("plugin_id: " + setting.pluginId());
         IO.println("installation_preference: " + setting.installationPreference().asString());
     }
     ```

     ```php PHP
     use Anthropic\Beta\Organization\Plugins\InstallationSettings\InstallationSettingSetParams\InstallationPreference;
     // ...

     $client = new Client();

     $setting = $client->beta->organization->plugins->installationSettings->set(
         target: 'organization',
         pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
         installationPreference: InstallationPreference::REQUIRED,
     );

     echo "plugin_id: {$setting->pluginID}\n";
     echo "installation_preference: {$setting->installationPreference}\n";
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
     setting = client.beta.organization.plugins.installation_settings.set(
       "organization",
       plugin_id: plugin_id,
       installation_preference: :required
     )

     puts "plugin_id: #{setting.plugin_id}"
     puts "installation_preference: #{setting.installation_preference}"
     ```
   </CodeGroup>

   然后使用 `DELETE /v1/organizations/plugins/{plugin_id}/installation_settings/{target}` 移除该组的设置，其中 `{target}` 是该组的 ID。组的设置会为其成员替换组织范围的值，而不是在其基础上叠加，因此残留的 `available` 组设置会使这些成员停留在 `available`。

### 保持安全清单同步

运行一个夜间作业，标记影响范围超出成员会话的插件，或内容扫描结果为 `warn`、`fail` 或 `unknown` 的插件。

1. 分页遍历 [`GET /v1/organizations/plugins?limit=100`](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#list-plugins)，直到 `next_page` 为 `null`（请参阅[分页](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#pagination)）。每次运行时都从该列表中读取每个插件的 `reach` 和 `content_scan`：仅 `content_scan` 的变化（例如稍后到达的结论）不会更新 `updated_at`。`updated_at` 通常可以告诉您自上次运行以来哪些插件有新内容或新的提供版本（值得重新下载归档）。等待中的版本（请参阅[版本和提供的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#versions-and-the-served-version)）在成为提供的版本时才会更新 `updated_at`，而不一定是在存储时。要发现这类版本，请将 `latest_version_id` 与上次运行时的值进行比较。完整的重新列出也能捕获移除情况，因为被 Git 同步或账户删除所移除的插件会在没有事件的情况下消失。
2. 标记 `reach` 为 `remote`（声明了 MCP 服务器或 CLI）或 `content_scan.assessment` 为 `warn`、`fail` 或 `unknown` 的每个插件。`warn` 也可能是[在 claude.ai 中被允许的失败扫描](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#failed-scans-allowed-in-claude-ai)。仅当您接受在那里被允许的每个失败扫描时，才可以排除 `warn`。这些字段描述的是提供的版本。如果还要检查未被提供的较新版本，请在 `latest_version_id` 与 `served_version_id` 不同时列出插件的版本（请参阅[列出插件的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#list-a-plugins-versions)）。
3. 对于每个被标记的插件，使用 `GET /v1/organizations/plugins/{plugin_id}/versions/{served_version_id}/content` 下载提供版本的归档以供审查（请参阅[下载版本的文件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#download-a-versions-files)）。同时按 ID 下载任何符合第 2 步检查条件的未提供版本。
4. 若要在审查期间让成员无法使用某个插件，请参阅[删除插件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#delete-a-plugin)，了解可逆（组织拥有的插件）和永久的选项。

## 插件

插件对象描述位于您组织的某个市场或某个成员个人市场中的插件（[快速入门](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#quick-start)的响应展示了一个完整的插件对象）。其 `display_name`、`description`、`manifest_version`、`content_scan`、`components` 和 `reach` 描述的是其**提供的**版本，因此一次列表调用即可显示向成员提供的内容。

| 字段                                                                                      | 描述                                                                                                                                                                                                                                                                                                                                                     |
| --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `id`                                                                                    | 前缀为 `plugin_`。                                                                                                                                                                                                                                                                                                                                         |
| `name`                                                                                  | 来自清单。在其市场内唯一，而非在整个组织内唯一。对于组织拥有的插件是固定的；如果成员在 claude.ai 中重命名自己的插件，则会发生变化。                                                                                                                                                                                                                                                                                |
| `display_name`、`description`、`manifest_version`                                         | 所提供版本的清单中的 `displayName`、`description` 和 `version`；当清单未声明时，各自为 `null`。`description` 可能是某个技能的 `description`，而不是清单的（在此表之后说明）。`manifest_version` 会为显示进行规范化：去掉一个前导 `v` 或 `V`，因此清单 `version` 为 `"v1.4.0"` 时返回 `"1.4.0"`。对于看起来不像版本号的值（例如 `"latest"`），以及在 claude.ai 于 2026 年 8 月开始记录此字段之前创建的插件版本，它也为 `null`。上传绝不会因为其 `version` 而被拒绝，且 `manifest_version` 不唯一。 |
| `served_version_id`、`latest_version_id`                                                 | 前缀为 `pluginver_`：分别为向成员提供的版本和最新版本。请参阅[版本和提供的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#versions-and-the-served-version)。                                                                                                                                                                                                     |
| `served_version_pinned`                                                                 | 在插件被固定之前为 `false`（请参阅[版本和提供的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#versions-and-the-served-version)）；此后为 `true`。                                                                                                                                                                                                         |
| `owner`                                                                                 | `{"type": "organization"}`，或对于成员的个人市场为 `{"type": "user", "user_id": "user_..."}`。                                                                                                                                                                                                                                                                      |
| `marketplace_id`                                                                        | 前缀为 `marketplace_`。                                                                                                                                                                                                                                                                                                                                    |
| `created_by`                                                                            | 插件的创建者：对于在 claude.ai 中的用户为 `{"type": "user_actor", "user_id": "user_...", "email_address": "..."}`（`email_address` 可能为 `null`），对于 API 密钥为 `{"type": "api_actor", "api_key_id": "apikey_..."}`。可能会出现其他操作者类型。未记录创建者时为 `null`，例如从 Git 同步的插件。                                                                                                              |
| `organization_installation_preference`、`organization_installation_preference_inherited` | 组织拥有的插件：组织范围的值，以及该值是否来自市场的默认值（请参阅[安装设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#installation-settings)）。成员拥有的插件：两者均为 `null`。                                                                                                                                                                                                  |
| `content_scan`                                                                          | 提供版本的扫描结果，一个包含 `status`、`assessment` 和 `reason` 的对象（在此表之后说明）。从未扫描时为 `null`。                                                                                                                                                                                                                                                                            |
| `components`                                                                            | 提供版本的组件，每个组件为 `{"type", "name", "description"}`，其中 `type` 为 `skill`、`mcp_server`、`command`、`agent`、`hook` 或 `cli` 之一，先按该类型顺序排列，再按名称排列。对于 MCP 服务器，`name` 是其在清单中的键；对于钩子，是其运行时所对应的事件；对于 CLI，是可执行文件的名称。对于 MCP 服务器、钩子和 CLI，`description` 始终为 `null`。未记录时为 `null`。                                                                                           |
| `reach`                                                                                 | `contained`、`privileged` 或 `remote`。请参阅[影响范围](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#reach)。                                                                                                                                                                                                                                 |
| `updated_at`                                                                            | 在所提供的版本变更时、插件被固定时更改，通常在存储新版本时也会更改。等待中的版本（请参阅[版本与提供的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#versions-and-the-served-version)）在其成为所提供的版本时更改此值，而不一定在其被存储时更改。安装设置、共享或仅 `content_scan` 的变更不会更改此值。                                                                                                                              |

`description` 并不总是清单中的描述。清单从顶层 `SKILL.md` 获取的 `description` 会被截断为前 500 个字符（请参阅[上传要求](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#upload-requirements)）。如果 `components` 的技能 `description` 中（内容相同的只算一个）恰好只有一个以清单的 `description` 作为前 500 个字符，API 将改为返回该技能的 `description`。因此：

* 您读取到的 `description` 可能超过 500 个字符，即上传清单的限制。如果上传的 `plugin.json` 设置了这样的 `description`，将返回 `400`。
* 在[您下载的文件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#download-a-versions-files)中，`.claude-plugin/plugin.json` 可能具有不同的 `description`，或者没有。

`content_scan` 对象：

| 字段           | 描述                                                                                                                                                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `status`     | 扫描运行期间为 `processing`，完成时为 `completed`，无法完成时为 `errored`（或者偶尔在本次响应无法读取其结果时也为此值，这种情况下之后的读取可能会报告结果）。如果存在已完成的扫描，则报告最新的一次。`status` 为 `processing` 或 `errored` 的版本不会提供给成员；将内容作为新版本再次上传会获得一次新的扫描。                                                  |
| `assessment` | 当 `status` 为 `completed` 时设置：`pass`（未发现问题）、`warn`（发现了不妨碍使用的问题，或[在 claude.ai 中被允许的失败扫描](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#failed-scans-allowed-in-claude-ai)）、`fail`（发现了阻止使用的问题）或 `unknown`（无结论）。否则为 `null`。 |
| `reason`     | 对于 `warn` 和 `fail`，为主要问题，取值见下表。否则为 `null`；对于早于原因记录功能的旧扫描，也为 `null`。                                                                                                                                                                        |

<Accordion title="内容扫描原因值">
  | `reason`                          | 含义                                                   |
  | --------------------------------- | ---------------------------------------------------- |
  | `covert-usage-telemetry`          | 指示 Claude 在不告知成员的情况下，将有关成员或其使用情况的信息发送到外部地址。          |
  | `undisclosed-data-destination`    | 将文件、电子邮件、文档或其他内容发送到成员看不到且无法控制的固定外部目标。                |
  | `remote-code-instruction-loader`  | 指示 Claude 下载并运行外部内容，或遵循来自外部内容的指令，而这些内容在插件安装后可能会发生变化。 |
  | `credential-exposure`             | 包含有效凭据，或从成员的环境中收集凭据或令牌。                              |
  | `guardrail-tampering`             | 削弱成员的安全防护，例如预先批准每个权限提示。                              |
  | `system-prompt-spoofing`          | 模仿或试图替换 Claude 的系统指令。                                |
  | `covert-record-tampering`         | 悄悄更改、隐藏或删除成员原本会看到的信息。                                |
  | `covert-behavior-override`        | 在插件用途之外更改 Claude 的行为，并向成员隐藏该更改。                      |
  | `hidden-code-execution`           | 运行捆绑的代码，同时指示 Claude 不要透露其作用。                         |
  | `undisclosed-promotion-injection` | 在 Claude 的输出中插入未披露的推广内容。                             |
  | `hidden-identity-gate`            | 根据运行它的账户改变或停止其行为，且不说明原因。                             |
  | `destructive-persistence`         | 可能删除或损坏成员的文件，或安装在插件移除后仍然保留的程序。                       |
  | `unanalyzable-binary`             | 包含已编译或无法读取的程序，因此扫描无法验证其作用。                           |
  | `other`                           | 任何其他问题，包括比此列表更新的问题。                                  |
</Accordion>

缺少 `plugin_` 前缀的 `plugin_id` 会返回 `400`。带有该前缀但不在您组织清单中的 `plugin_id` 会返回 `404`。

### 列出插件

`GET /v1/organizations/plugins` 列出您组织中的插件，包括组织市场中的插件和成员个人市场中的插件，按 `created_at` 降序排列。可按 `owner_type`（`organization` 或 `user`）、`owner_user_id`（以 `user_` 为前缀的 ID，用于查找某位成员的插件，包括该成员离开组织之后）、`marketplace_id` 以及 `created_at[gte]`、`created_at[gt]`、`created_at[lte]`、`created_at[lt]`（RFC 3339 时间戳）进行筛选。筛选条件以 AND 组合。在您的组织中不匹配任何内容的 `marketplace_id` 或 `owner_user_id` 返回空页面，而不是错误。响应的结构如[快速入门](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#quick-start)中所示。需要 `read:plugins` 作用域。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins?owner_type=organization&limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins list --owner-type organization --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  plugins = client.beta.organization.plugins.list(owner_type="organization", limit=20)

  # 根据需要自动获取更多页面。
  for plugin in plugins:
      print(f"{plugin.id}: {plugin.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const plugins = await client.beta.organization.plugins.list({
    owner_type: "organization",
    limit: 20
  });

  for await (const plugin of plugins) {
    console.log(`${plugin.id}: ${plugin.name}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Plugins;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.Plugins.List(
      new() { OwnerType = OwnerType.Organization, Limit = 20 }
  );

  await foreach (var plugin in page.Paginate())
  {
      Console.WriteLine($"{plugin.ID}: {plugin.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  plugins := client.Beta.Organization.Plugins.ListAutoPaging(context.Background(), anthropic.BetaOrganizationPluginListParams{
  	OwnerType: anthropic.BetaOrganizationPluginListParamsOwnerTypeOrganization,
  	Limit:     anthropic.Int(20),
  })

  for plugins.Next() {
  	plugin := plugins.Current()
  	fmt.Printf("%s: %s\n", plugin.ID, plugin.Name)
  }
  if err := plugins.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.plugins.PluginListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = PluginListParams.builder()
          .ownerType(PluginListParams.OwnerType.ORGANIZATION)
          .limit(20)
          .build();
      var plugins = client.beta().organization().plugins().list(params);

      for (var plugin : plugins.autoPager()) {
          IO.println(plugin.id() + ": " + plugin.name());
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\Plugins\PluginListParams\OwnerType;
  // ...

  $client = new Client();

  $plugins = $client->beta->organization->plugins->list(
      limit: 20,
      ownerType: OwnerType::ORGANIZATION,
  );

  // 仅限当前页；如需下一页，请使用 page: $plugins->nextPage 再次调用 list()。
  foreach ($plugins->getItems() as $plugin) {
      echo "{$plugin->id}: {$plugin->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.plugins.list(owner_type: :organization, limit: 20)

  page.auto_paging_each do |plugin|
    puts "#{plugin.id}: #{plugin.name}"
  end
  ```
</CodeGroup>

### 创建插件

`POST /v1/organizations/plugins` 在一次调用中创建一个组织拥有的插件及其第一个版本；该版本成为所提供的版本。请求体为 `multipart/form-data`：`files[]` 要么是一个 `.zip` 或 `.plugin` 归档文件，要么是每个文件对应一个部分，其中每个部分的文件名是该文件在插件中的路径（例如 `.claude-plugin/plugin.json`）。可选字段为 `marketplace_id`（组织拥有的 `manual` 市场，默认为您的库市场，该市场在首次使用时创建）和 `release_notes`（最多 5,000 个字符，显示在 claude.ai 的版本历史中，并随版本返回）。插件的 `name`、`display_name`、`description` 和 `manifest_version` 来自上传的文件，且上传必须满足[上传要求](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#upload-requirements)。启用内容扫描时，响应中的 `content_scan.status` 为 `processing`，结论会异步返回。返回该插件。需要 `write:plugins` 作用域。

上传归档文件：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugins" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -F "files[]=@dist/sales-toolkit.zip" \
    -F "release_notes=First release"
  ```

  ```bash CLI
  ant beta:organization:plugins create \
    --file dist/sales-toolkit.zip \
    --release-notes "First release"
  ```

  ```python Python
  client = anthropic.Anthropic()

  with open("dist/sales-toolkit.zip", "rb") as archive:
      plugin = client.beta.organization.plugins.create(
          files=[archive],
          release_notes="First release",
      )

  print(f"id: {plugin.id}")
  print(f"served_version_id: {plugin.served_version_id}")
  ```

  ```typescript TypeScript
  import fs from "node:fs";

  const client = new Anthropic();

  const plugin = await client.beta.organization.plugins.create({
    files: [fs.createReadStream("dist/sales-toolkit.zip")],
    release_notes: "First release"
  });

  console.log(`id: ${plugin.id}`);
  console.log(`served_version_id: ${plugin.served_version_id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var plugin = await client.Beta.Organization.Plugins.Create(
      new()
      {
          Files = [File.OpenRead("dist/sales-toolkit.zip")],
          ReleaseNotes = "First release",
      }
  );

  Console.WriteLine($"id: {plugin.ID}");
  Console.WriteLine($"served_version_id: {plugin.ServedVersionID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  archive, err := os.Open("dist/sales-toolkit.zip")
  if err != nil {
  	log.Fatal(err)
  }
  defer archive.Close()

  plugin, err := client.Beta.Organization.Plugins.New(context.Background(), anthropic.BetaOrganizationPluginNewParams{
  	Files:        []io.Reader{archive},
  	ReleaseNotes: anthropic.String("First release"),
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", plugin.ID)
  fmt.Printf("served_version_id: %s\n", plugin.ServedVersionID)
  ```

  ```java Java
  import com.anthropic.core.MultipartField;
  import com.anthropic.models.beta.organization.plugins.PluginCreateParams;

  void main() throws Exception {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      // 手动构建 `files[]` 部分，以便归档文件连同其文件名一起发送，
      // 这是上传所必需的。
      var archive = MultipartField.<List<InputStream>>builder()
          .value(List.of(Files.newInputStream(Path.of("dist/sales-toolkit.zip"))))
          .filename("sales-toolkit.zip")
          .contentType("application/octet-stream")
          .build();
      var params = PluginCreateParams.builder()
          .files(archive)
          .releaseNotes("First release")
          .build();
      var plugin = client.beta().organization().plugins().create(params);

      IO.println("id: " + plugin.id());
      IO.println("served_version_id: " + plugin.servedVersionId());
  }
  ```

  ```php PHP
  use Anthropic\Core\FileParam;

  $client = new Client();

  $plugin = $client->beta->organization->plugins->create(
      files: [
          FileParam::fromResource(fopen('dist/sales-toolkit.zip', 'r')),
      ],
      releaseNotes: 'First release',
  );

  echo "id: {$plugin->id}\n";
  echo "served_version_id: {$plugin->servedVersionID}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin = client.beta.organization.plugins.create(
    files: [Pathname("dist/sales-toolkit.zip")],
    release_notes: "First release"
  )

  puts "id: #{plugin.id}"
  puts "served_version_id: #{plugin.served_version_id}"
  ```
</CodeGroup>

```json
{
  "type": "plugin",
  "id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  "name": "sales-toolkit",
  "display_name": "Sales Toolkit",
  "description": "Account research and call prep for the sales team.",
  "served_version_id": "pluginver_01Km7tL4pR9xF5sU2zV3jP6q",
  "served_version_pinned": false,
  "latest_version_id": "pluginver_01Km7tL4pR9xF5sU2zV3jP6q",
  "manifest_version": "1.4.0",
  "owner": { "type": "organization" },
  "marketplace_id": "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
  "created_by": { "type": "api_actor", "api_key_id": "apikey_01Nq9vN6rT2zH7uW4bX5mR8s" },
  "organization_installation_preference": "available",
  "organization_installation_preference_inherited": true,
  "content_scan": { "status": "processing", "assessment": null, "reason": null },
  "components": [
    {
      "type": "skill",
      "name": "account-research",
      "description": "Researches a customer account before a call."
    },
    { "type": "mcp_server", "name": "crm", "description": null }
  ],
  "reach": "remote",
  "created_at": "2026-09-01T17:04:11Z",
  "updated_at": "2026-09-01T17:04:11Z"
}
```

将单个文件上传到指定的市场。将每个文件附加在其在插件中的路径下（cURL 示例中的 `;filename=` 后缀，SDK 示例中的文件名参数）；仅以基本名称发送的文件会导致找不到清单。TypeScript 和 Java SDK 以及 `ant` CLI 目前还无法在路径下附加文件，因此这些示例改为将插件作为一个归档文件上传到市场：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugins" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -F "files[]=@.claude-plugin/plugin.json;filename=.claude-plugin/plugin.json" \
    -F "files[]=@skills/account-research/SKILL.md;filename=skills/account-research/SKILL.md" \
    -F "marketplace_id=marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  ```

  ```bash CLI
  # CLI 仅以基本文件名发送每个文件，
  # 因此请将插件打包为单个归档文件上传。
  ant beta:organization:plugins create \
    --file dist/sales-toolkit.zip \
    --marketplace-id marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 使用 (filename, file) 元组可保留每个文件在插件内的路径；
  # 若仅传入文件对象，则只会以其基本文件名发送。
  with (
      open(".claude-plugin/plugin.json", "rb") as manifest,
      open("skills/account-research/SKILL.md", "rb") as skill_md,
  ):
      plugin = client.beta.organization.plugins.create(
          files=[
              (".claude-plugin/plugin.json", manifest),
              ("skills/account-research/SKILL.md", skill_md),
          ],
          marketplace_id="marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
      )

  print(f"id: {plugin.id}")
  print(f"served_version_id: {plugin.served_version_id}")
  ```

  ```typescript TypeScript
  import fs from "node:fs";

  const client = new Anthropic();

  // TypeScript SDK 会以基本文件名（base name）发送每个文件，
  // 因此请将插件打包为单个归档文件上传。
  const plugin = await client.beta.organization.plugins.create({
    files: [fs.createReadStream("dist/sales-toolkit.zip")],
    marketplace_id: "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  });

  console.log(`id: ${plugin.id}`);
  console.log(`served_version_id: ${plugin.served_version_id}`);
  ```

  ```csharp C#
  using Anthropic.Core;

  AnthropicClient client = new();

  // FileName 会保留每个文件在插件内的路径；若不设置，FileStream
  // 将仅以其基本文件名（base name）发送。
  var plugin = await client.Beta.Organization.Plugins.Create(
      new()
      {
          Files =
          [
              new BinaryContent
              {
                  Stream = File.OpenRead(".claude-plugin/plugin.json"),
                  FileName = ".claude-plugin/plugin.json",
              },
              new BinaryContent
              {
                  Stream = File.OpenRead("skills/account-research/SKILL.md"),
                  FileName = "skills/account-research/SKILL.md",
              },
          ],
          MarketplaceID = "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
      }
  );

  Console.WriteLine($"id: {plugin.ID}");
  Console.WriteLine($"served_version_id: {plugin.ServedVersionID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  manifest, err := os.Open(".claude-plugin/plugin.json")
  if err != nil {
  	log.Fatal(err)
  }
  defer manifest.Close()

  skillMd, err := os.Open("skills/account-research/SKILL.md")
  if err != nil {
  	log.Fatal(err)
  }
  defer skillMd.Close()

  // anthropic.File 会将每个文件在插件中的路径保留为该部分的文件名；
  // 若直接使用 *os.File，则会以其基本名称发送，导致找不到清单文件。
  plugin, err := client.Beta.Organization.Plugins.New(context.Background(), anthropic.BetaOrganizationPluginNewParams{
  	Files: []io.Reader{
  		anthropic.File(manifest, ".claude-plugin/plugin.json", "application/json"),
  		anthropic.File(skillMd, "skills/account-research/SKILL.md", "text/markdown"),
  	},
  	MarketplaceID: anthropic.String("marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"),
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", plugin.ID)
  fmt.Printf("served_version_id: %s\n", plugin.ServedVersionID)
  ```

  ```java Java
  import com.anthropic.core.MultipartField;
  import com.anthropic.models.beta.organization.plugins.PluginCreateParams;

  void main() throws Exception {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      // PluginCreateParams 上的 `files` setter 只为整个
      // 列表接受一个文件名，因此改为将插件作为单个归档上传：即一个手动构建的
      // 部分，携带上传所需的归档文件名。
      var archive = MultipartField.<List<InputStream>>builder()
          .value(List.of(Files.newInputStream(Path.of("dist/sales-toolkit.zip"))))
          .filename("sales-toolkit.zip")
          .contentType("application/octet-stream")
          .build();
      var params = PluginCreateParams.builder()
          .files(archive)
          .marketplaceId("marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r")
          .build();
      var plugin = client.beta().organization().plugins().create(params);

      IO.println("id: " + plugin.id());
      IO.println("served_version_id: " + plugin.servedVersionId());
  }
  ```

  ```php PHP
  use Anthropic\Core\FileParam;

  $client = new Client();

  $plugin = $client->beta->organization->plugins->create(
      files: [
          FileParam::fromResource(
              fopen('.claude-plugin/plugin.json', 'r'),
              filename: '.claude-plugin/plugin.json',
          ),
          FileParam::fromResource(
              fopen('skills/account-research/SKILL.md', 'r'),
              filename: 'skills/account-research/SKILL.md',
          ),
      ],
      marketplaceID: 'marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r',
  );

  echo "id: {$plugin->id}\n";
  echo "served_version_id: {$plugin->servedVersionID}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 带 filename: 的 FilePart 会保留每个文件在插件内的路径；
  # 而裸 Pathname 只会以其基本文件名发送。
  plugin = client.beta.organization.plugins.create(
    files: [
      Anthropic::FilePart.new(
        Pathname(".claude-plugin/plugin.json"),
        filename: ".claude-plugin/plugin.json"
      ),
      Anthropic::FilePart.new(
        Pathname("skills/account-research/SKILL.md"),
        filename: "skills/account-research/SKILL.md"
      )
    ],
    marketplace_id: "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  )

  puts "id: #{plugin.id}"
  puts "served_version_id: #{plugin.served_version_id}"
  ```
</CodeGroup>

除了违反[上传要求](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#upload-requirements)的上传返回 `400`（请求体超过 200 MB 时返回 `413`）以及通用响应（当 `marketplace_id` 是成员的个人市场时返回 `403`；请参阅[错误响应](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#error-responses)）之外，创建操作还可能因以下原因失败：

| 状态                         | 原因                                                                                                                           | 处理方法                                                                                                                                     |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 404                        | `marketplace_id` 不是您组织的市场。                                                                                                   | 从[列出市场](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#list-marketplaces)获取 ID。                                        |
| 400                        | 该市场从 Git 同步，或已包含 1,000 个插件（请参阅[上传要求](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#upload-requirements)）。 | 上传到另一个 `manual` 市场，或改为更改存储库。                                                                                                             |
| 409 `plugin_name_taken`    | 该名称在该市场中已被占用。                                                                                                                | 仅当 `details.plugin_id` 是您打算更新的插件时，才向其上传版本，因为它可能是其他人创建的。否则，或者当 `details.plugin_id` 不存在时，请使用其他名称上传（更改清单的 `name`）。                          |
| 409 `skill_name_taken`     | 该插件将进入库市场，且其某个技能与某个组织技能同名。                                                                                                   | 重命名该技能，或在 claude.ai 中移除该组织技能。                                                                                                            |
| 409（无 `error_code`）        | 另一个向同一市场上传同名插件的操作正在进行中，或者此 API 未列出的某个插件占用了该名称。                                                                               | 使用退避策略重试。如果 `409` 持续超过几分钟，请使用其他名称上传（更改清单的 `name`）。                                                                                       |
| 503 `registration_pending` | 插件已创建，但其注册未完成。                                                                                                               | 不要重新发送；将相同的文件作为 `details.plugin_id` 的版本上传（请参阅[重试上传](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#retrying-uploads)）。 |

### 获取插件

`GET /v1/organizations/plugins/{plugin_id}` 返回一个插件。需要 `read:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins retrieve \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL
  ```

  ```python Python
  client = anthropic.Anthropic()

  plugin = client.beta.organization.plugins.retrieve("plugin_01Hq3vX8kZcN2mB7pR4tY9wL")

  print(f"id: {plugin.id}")
  print(f"name: {plugin.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const plugin = await client.beta.organization.plugins.retrieve(
    "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  );

  console.log(`id: ${plugin.id}`);
  console.log(`name: ${plugin.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var plugin = await client.Beta.Organization.Plugins.Retrieve("plugin_01Hq3vX8kZcN2mB7pR4tY9wL");

  Console.WriteLine($"id: {plugin.ID}");
  Console.WriteLine($"name: {plugin.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  plugin, err := client.Beta.Organization.Plugins.Get(
  	context.Background(),
  	"plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	anthropic.BetaOrganizationPluginGetParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", plugin.ID)
  fmt.Printf("name: %s\n", plugin.Name)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var plugin = client.beta().organization().plugins()
      .retrieve("plugin_01Hq3vX8kZcN2mB7pR4tY9wL");

  IO.println("id: " + plugin.id());
  IO.println("name: " + plugin.name());
  ```

  ```php PHP
  $client = new Client();

  $plugin = $client->beta->organization->plugins->retrieve(
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
  );

  echo "id: {$plugin->id}\n";
  echo "name: {$plugin->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  plugin = client.beta.organization.plugins.retrieve(plugin_id)

  puts "id: #{plugin.id}"
  puts "name: #{plugin.name}"
  ```
</CodeGroup>

### 更改提供版本

`POST /v1/organizations/plugins/{plugin_id}` 更改组织拥有的插件向成员提供的版本。传入较早的版本可回滚，传入较新的版本可推广一个已存储但未提供的构建。此操作会固定该插件，而已固定的插件无法取消固定，无论在此处还是在 claude.ai 中（请参阅[版本和提供的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#versions-and-the-served-version)）。唯一可更新的字段是 `served_version_id`，且为必填项。更改会在响应返回之前到达成员，且不会创建版本。启用内容扫描时，该版本必须是可以提供给成员的版本（请参阅[内容扫描](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#content-scanning)）。对已固定的插件传入当前已提供的版本不会产生任何变化；对未固定的插件传入该版本会将插件固定在该版本，因此之后的上传将不再自动提供。固定也会结束任何等待：正在等待内容扫描的版本仍保持存储状态，只有在有人选择它时才会被提供。返回该插件。需要 `write:plugins` 作用域。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -d '{"served_version_id": "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p"}'
  ```

  ```bash CLI
  ant beta:organization:plugins update \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
    --served-version-id pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p
  ```

  ```python Python
  client = anthropic.Anthropic()

  plugin = client.beta.organization.plugins.update(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
      served_version_id="pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p",
  )

  print(f"id: {plugin.id}")
  print(f"served_version_id: {plugin.served_version_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const plugin = await client.beta.organization.plugins.update(
    "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
    { served_version_id: "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p" }
  );

  console.log(`id: ${plugin.id}`);
  console.log(`served_version_id: ${plugin.served_version_id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var plugin = await client.Beta.Organization.Plugins.Update(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
      new() { ServedVersionID = "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p" }
  );

  Console.WriteLine($"id: {plugin.ID}");
  Console.WriteLine($"served_version_id: {plugin.ServedVersionID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  plugin, err := client.Beta.Organization.Plugins.Update(
  	context.Background(),
  	"plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	anthropic.BetaOrganizationPluginUpdateParams{
  		ServedVersionID: "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", plugin.ID)
  fmt.Printf("served_version_id: %s\n", plugin.ServedVersionID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.plugins.PluginUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = PluginUpdateParams.builder()
          .servedVersionId("pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p")
          .build();
      var plugin = client.beta().organization().plugins()
          .update("plugin_01Hq3vX8kZcN2mB7pR4tY9wL", params);

      IO.println("id: " + plugin.id());
      IO.println("served_version_id: " + plugin.servedVersionId());
  }
  ```

  ```php PHP
  $client = new Client();

  $plugin = $client->beta->organization->plugins->update(
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
      servedVersionID: 'pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p',
  );

  echo "id: {$plugin->id}\n";
  echo "served_version_id: {$plugin->servedVersionID}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  plugin = client.beta.organization.plugins.update(
    plugin_id,
    served_version_id: "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p"
  )

  puts "id: #{plugin.id}"
  puts "served_version_id: #{plugin.served_version_id}"
  ```
</CodeGroup>

```json
{
  "type": "plugin",
  "id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  "name": "sales-toolkit",
  "display_name": "Sales Toolkit",
  "description": "Account research and call prep for the sales team.",
  "served_version_id": "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p",
  "served_version_pinned": true,
  "latest_version_id": "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p",
  "manifest_version": "1.5.0",
  "owner": { "type": "organization" },
  "marketplace_id": "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
  "created_by": { "type": "api_actor", "api_key_id": "apikey_01Nq9vN6rT2zH7uW4bX5mR8s" },
  "organization_installation_preference": "available",
  "organization_installation_preference_inherited": true,
  "content_scan": { "status": "completed", "assessment": "pass", "reason": null },
  "components": [
    {
      "type": "skill",
      "name": "account-research",
      "description": "Researches a customer account before a call."
    },
    { "type": "mcp_server", "name": "crm", "description": null },
    {
      "type": "command",
      "name": "call-prep",
      "description": "Builds a one-page brief for an upcoming call."
    }
  ],
  "reach": "remote",
  "created_at": "2026-09-01T17:04:11Z",
  "updated_at": "2026-09-16T10:02:45Z"
}
```

除了共享响应（成员所有的插件返回 `403`，无法提供给成员的版本返回 `409 scan_pending` 或 `400 scan_failed`；请参阅[错误响应](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#error-responses)）之外，该请求还可能因以下原因失败：

| 状态                     | 原因                                                                                   | 处理方法                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| 400                    | 请求体省略了 `served_version_id`、将其设置为 `null` 或包含任何其他字段；或者该值缺少 `pluginver_` 前缀或为 `latest`。 | 仅发送 `{"served_version_id": "pluginver_..."}`。                                                              |
| 404                    | `served_version_id` 不是此插件的版本。                                                        | 从[列出插件的版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#list-a-plugins-versions)获取 ID。 |
| 409（无 `error_code`）    | 对此插件的上传或另一个提供版本更改仍在进行中。                                                              | 稍后重试。                                                                                                      |
| 409 `skill_name_taken` | 该插件位于库市场中，且该版本包含一个名称现已被某个组织技能使用的技能。                                                  | 选择另一个版本，或重命名其中一个技能。                                                                                        |

### 删除插件

`DELETE /v1/organizations/plugins/{plugin_id}` 永久删除插件及其包含的每个版本，效果与管理员在 claude.ai 中执行删除相同。它适用于 `manual` 市场中的任何插件，包括成员的插件，即使该成员此后已离开组织。删除返回时，该插件、其版本及其文件将从所有读取结果中消失，并且不再向成员提供。组织所有插件的安装设置会随之移除；成员所有插件的共享会被撤回，并且该插件对其所有者也会消失。从 Git 同步的市场中的插件会返回 `400`：请从存储库中移除它，或在 claude.ai 中移除该市场。需要 `write:plugins` 范围。

删除无法撤销，且不支持按版本删除。如果要以可逆方式停止提供组织拥有的插件，请将其组织范围的安装设置设为 `not_available`（原本继承其市场默认值的插件将获得自己的设置，之后[移除该组织范围设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#remove-an-installation-setting)即可让插件恢复为默认值），并移除（或设为 `not_available`）`GET /v1/organizations/plugins/{plugin_id}/installation_settings` 列出的每个组设置，因为组的设置会为其成员覆盖组织范围的值。请依次发送这些写入请求，而不是并行发送（请参阅[设置安装设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#set-an-installation-setting)）。成员拥有的插件无法通过此 API 停止提供，只能将其删除，且仅当其市场为 `manual` 时才可删除。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins delete --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted_plugin = client.beta.organization.plugins.delete(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  )

  print(f"id: {deleted_plugin.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deletedPlugin = await client.beta.organization.plugins.delete(
    "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  );

  console.log(`id: ${deletedPlugin.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deletedPlugin = await client.Beta.Organization.Plugins.Delete(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  );

  Console.WriteLine($"id: {deletedPlugin.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  deletedPlugin, err := client.Beta.Organization.Plugins.Delete(
  	context.Background(),
  	"plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	anthropic.BetaOrganizationPluginDeleteParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", deletedPlugin.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var deletedPlugin = client.beta().organization().plugins()
      .delete("plugin_01Hq3vX8kZcN2mB7pR4tY9wL");

  IO.println("id: " + deletedPlugin.id());
  ```

  ```php PHP
  $client = new Client();

  $deletedPlugin = $client->beta->organization->plugins->delete(
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
  );

  echo "id: {$deletedPlugin->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  deleted_plugin = client.beta.organization.plugins.delete(plugin_id)

  puts "id: #{deleted_plugin.id}"
  ```
</CodeGroup>

```json
{ "type": "plugin_deleted", "id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL" }
```

## 插件版本

插件版本是某次上传中插件文件的不可变快照（[创建版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#create-a-version)的响应展示了一个完整对象）。其字段对应插件所提供版本的字段（`display_name`、`description`、`manifest_version`、`content_scan`、`components`、`reach`），但针对的是此版本，此外还有 `release_notes`（随上传提供并显示在 claude.ai 的版本历史中）和 `created_by`（上传者）。

缺少 `pluginver_` 前缀的 `{version}` 会返回 `400`（注明可使用字面值 `latest` 的情况除外）。带有该前缀但不标识该插件某个版本的值会返回 `404`。

### 列出插件的版本

`GET /v1/organizations/plugins/{plugin_id}/versions` 列出插件的版本，按 `created_at` 降序排列；第一项是 `latest_version_id` 所标识的版本。`limit` 为 1 到 1,000。需要 `read:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/versions?limit=50" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins:versions list \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
    --limit 50
  ```

  ```python Python
  client = anthropic.Anthropic()

  versions = client.beta.organization.plugins.versions.list(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL", limit=50
  )

  # 根据需要自动获取更多页面。
  for version in versions:
      print(f"{version.id}: {version.manifest_version}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const versions = await client.beta.organization.plugins.versions.list(
    "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
    { limit: 50 }
  );

  for await (const version of versions) {
    console.log(`${version.id}: ${version.manifest_version}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.Plugins.Versions.List(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
      new() { Limit = 50 }
  );

  await foreach (var version in page.Paginate())
  {
      Console.WriteLine($"{version.ID}: {version.ManifestVersion}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  versions := client.Beta.Organization.Plugins.Versions.ListAutoPaging(
  	context.Background(),
  	"plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	anthropic.BetaOrganizationPluginVersionListParams{
  		Limit: anthropic.Int(50),
  	},
  )

  for versions.Next() {
  	version := versions.Current()
  	fmt.Printf("%s: %s\n", version.ID, version.ManifestVersion)
  }
  if err := versions.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.plugins.versions.VersionListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = VersionListParams.builder()
          .limit(50)
          .build();
      var versions = client.beta().organization().plugins().versions()
          .list("plugin_01Hq3vX8kZcN2mB7pR4tY9wL", params);

      for (var version : versions.autoPager()) {
          IO.println(version.id() + ": " + version.manifestVersion().orElse(null));
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $versions = $client->beta->organization->plugins->versions->list(
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
      limit: 50,
  );

  // 仅限当前页；如需获取下一页，请使用 page: $versions->nextPage 再次调用 list()。
  foreach ($versions->getItems() as $version) {
      echo "{$version->id}: {$version->manifestVersion}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  page = client.beta.organization.plugins.versions.list(plugin_id, limit: 50)

  page.auto_paging_each do |version|
    puts "#{version.id}: #{version.manifest_version}"
  end
  ```
</CodeGroup>

### 创建版本

`POST /v1/organizations/plugins/{plugin_id}/versions` 向 `manual` 市场中组织拥有的插件添加一个版本。请求体为 `multipart/form-data`，其 `files[]` 和 `release_notes` 字段、[上传要求](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#upload-requirements)以及文件、清单、归档和大小错误均与[创建插件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#create-a-plugin)相同。上传的名称（清单中的 `name`）必须与插件的 `name` 相同。如果插件未固定，新版本通常在存储后立即提供；如果已固定，该版本会被存储但不会提供，直到您将提供的版本更改为它。要进行检查，请将响应的 `id` 与插件的 `served_version_id` 进行比较。返回该版本。需要 `write:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/versions" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -F "files[]=@dist/sales-toolkit.zip" \
    -F "release_notes=Adds the call-prep command."
  ```

  ```bash CLI
  ant beta:organization:plugins:versions create \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
    --file dist/sales-toolkit.zip \
    --release-notes "Adds the call-prep command."
  ```

  ```python Python
  client = anthropic.Anthropic()

  with open("dist/sales-toolkit.zip", "rb") as archive:
      version = client.beta.organization.plugins.versions.create(
          "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
          files=[archive],
          release_notes="Adds the call-prep command.",
      )

  print(f"id: {version.id}")
  print(f"manifest_version: {version.manifest_version}")
  ```

  ```typescript TypeScript
  import fs from "node:fs";

  const client = new Anthropic();

  const version = await client.beta.organization.plugins.versions.create(
    "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
    {
      files: [fs.createReadStream("dist/sales-toolkit.zip")],
      release_notes: "Adds the call-prep command."
    }
  );

  console.log(`id: ${version.id}`);
  console.log(`manifest_version: ${version.manifest_version}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var version = await client.Beta.Organization.Plugins.Versions.Create(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
      new()
      {
          Files = [File.OpenRead("dist/sales-toolkit.zip")],
          ReleaseNotes = "Adds the call-prep command.",
      }
  );

  Console.WriteLine($"id: {version.ID}");
  Console.WriteLine($"manifest_version: {version.ManifestVersion}");
  ```

  ```go Go
  client := anthropic.NewClient()

  archive, err := os.Open("dist/sales-toolkit.zip")
  if err != nil {
  	log.Fatal(err)
  }
  defer archive.Close()

  version, err := client.Beta.Organization.Plugins.Versions.New(
  	context.Background(),
  	"plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	anthropic.BetaOrganizationPluginVersionNewParams{
  		Files:        []io.Reader{archive},
  		ReleaseNotes: anthropic.String("Adds the call-prep command."),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", version.ID)
  fmt.Printf("manifest_version: %s\n", version.ManifestVersion)
  ```

  ```java Java
  import com.anthropic.core.MultipartField;
  import com.anthropic.models.beta.organization.plugins.versions.VersionCreateParams;

  void main() throws Exception {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      // 手动构建 `files[]` 部分，以便归档文件连同其文件名一起发送，
      // 这是上传所必需的。
      var archive = MultipartField.<List<InputStream>>builder()
          .value(List.of(Files.newInputStream(Path.of("dist/sales-toolkit.zip"))))
          .filename("sales-toolkit.zip")
          .contentType("application/octet-stream")
          .build();
      var params = VersionCreateParams.builder()
          .files(archive)
          .releaseNotes("Adds the call-prep command.")
          .build();
      var version = client.beta().organization().plugins().versions()
          .create("plugin_01Hq3vX8kZcN2mB7pR4tY9wL", params);

      IO.println("id: " + version.id());
      IO.println("manifest_version: " + version.manifestVersion().orElse(null));
  }
  ```

  ```php PHP
  use Anthropic\Core\FileParam;

  $client = new Client();

  $version = $client->beta->organization->plugins->versions->create(
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
      files: [
          FileParam::fromResource(fopen('dist/sales-toolkit.zip', 'r')),
      ],
      releaseNotes: 'Adds the call-prep command.',
  );

  echo "id: {$version->id}\n";
  echo "manifest_version: {$version->manifestVersion}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  version = client.beta.organization.plugins.versions.create(
    plugin_id,
    files: [Pathname("dist/sales-toolkit.zip")],
    release_notes: "Adds the call-prep command."
  )

  puts "id: #{version.id}"
  puts "manifest_version: #{version.manifest_version}"
  ```
</CodeGroup>

```json
{
  "type": "plugin_version",
  "id": "pluginver_01Jd5sK2nQ8wE4rT6yU1iO3p",
  "plugin_id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  "display_name": "Sales Toolkit",
  "description": "Account research and call prep for the sales team.",
  "manifest_version": "1.5.0",
  "release_notes": "Adds the call-prep command.",
  "created_by": { "type": "api_actor", "api_key_id": "apikey_01Nq9vN6rT2zH7uW4bX5mR8s" },
  "content_scan": { "status": "processing", "assessment": null, "reason": null },
  "components": [
    {
      "type": "skill",
      "name": "account-research",
      "description": "Researches a customer account before a call."
    },
    { "type": "mcp_server", "name": "crm", "description": null },
    {
      "type": "command",
      "name": "call-prep",
      "description": "Builds a one-page brief for an upcoming call."
    }
  ],
  "reach": "remote",
  "created_at": "2026-09-15T14:12:30Z"
}
```

除了违反[上传要求](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#upload-requirements)的上传返回 `400`（请求体超过 200 MB 时返回 `413`）以及通用响应（对于成员拥有的插件返回 `403`；请参阅[错误响应](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#error-responses)）之外，请求还可能因以下原因失败：

| 状态                         | 原因                                | 处理方法                                                                                                                                    |
| -------------------------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| 400                        | 该插件位于从 Git 同步的市场中，或上传的名称与插件的名称不同。 | 改为更改存储库，或修正清单的 `name`。                                                                                                                  |
| 409（无 `error_code`）        | 对此插件的另一个上传或提供版本更改仍在进行中。           | 稍后重试。                                                                                                                                   |
| 409 `skill_name_taken`     | 该插件位于库市场中，且该版本添加了一个与某个组织技能同名的技能。  | 重命名该技能，或在 claude.ai 中移除该组织技能。                                                                                                           |
| 503 `registration_pending` | 版本已存储，但其注册未完成。                    | 当响应带有 `x-should-retry: true` 时，重新发送相同的请求（请参阅[重试上传](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#retrying-uploads)）。 |

### 获取版本

`GET /v1/organizations/plugins/{plugin_id}/versions/{version}` 返回一个版本。`{version}` 是版本 ID，或者是 `latest`，表示请求时 `latest_version_id` 所标识的版本。需要 `read:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/versions/latest" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins:versions retrieve \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
    --version latest
  ```

  ```python Python
  client = anthropic.Anthropic()

  version = client.beta.organization.plugins.versions.retrieve(
      "latest",
      plugin_id="plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  )

  print(f"id: {version.id}")
  print(f"manifest_version: {version.manifest_version}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const version = await client.beta.organization.plugins.versions.retrieve("latest", {
    plugin_id: "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  });

  console.log(`id: ${version.id}`);
  console.log(`manifest_version: ${version.manifest_version}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var version = await client.Beta.Organization.Plugins.Versions.Retrieve(
      "latest",
      new() { PluginID = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL" }
  );

  Console.WriteLine($"id: {version.ID}");
  Console.WriteLine($"manifest_version: {version.ManifestVersion}");
  ```

  ```go Go
  client := anthropic.NewClient()

  version, err := client.Beta.Organization.Plugins.Versions.Get(
  	context.Background(),
  	"latest",
  	anthropic.BetaOrganizationPluginVersionGetParams{
  		PluginID: "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", version.ID)
  fmt.Printf("manifest_version: %s\n", version.ManifestVersion)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.plugins.versions.VersionRetrieveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = VersionRetrieveParams.builder()
          .pluginId("plugin_01Hq3vX8kZcN2mB7pR4tY9wL")
          .build();
      var version = client.beta().organization().plugins().versions()
          .retrieve("latest", params);

      IO.println("id: " + version.id());
      IO.println("manifest_version: " + version.manifestVersion().orElse(null));
  }
  ```

  ```php PHP
  $client = new Client();

  $version = $client->beta->organization->plugins->versions->retrieve(
      version: 'latest',
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
  );

  echo "id: {$version->id}\n";
  echo "manifest_version: {$version->manifestVersion}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  version = client.beta.organization.plugins.versions.retrieve("latest", plugin_id: plugin_id)

  puts "id: #{version.id}"
  puts "manifest_version: #{version.manifest_version}"
  ```
</CodeGroup>

### 下载版本的文件

`GET /v1/organizations/plugins/{plugin_id}/versions/{version}/content` 以存储的 `.zip` 归档文件形式（`Content-Type: application/zip`）下载版本的文件。无论内容扫描结果如何，都会返回该归档文件，因此您可以检查未向成员提供的版本。它按存储的原样提供，因此对于 `manual` 市场中组织拥有的插件，只要它满足当前的上传要求，您就可以将其原封不动地作为新版本重新上传。`{version}` 必须是版本 ID，而不是 `latest`：请先读取插件的 `served_version_id` 或 `latest_version_id`，或使用 `GET /v1/organizations/plugins/{plugin_id}/versions/latest` 解析 `latest`。`Content-Disposition` 文件名源自插件名称，且不唯一；请按插件和版本 ID 为保存的文件命名。需要 `read:plugins` 作用域。

下载**成员所有**插件的归档文件会在 Compliance API 活动源上记录一个 `claude_plugin_archive_accessed` 事件，通过 ID 标识密钥（作为 `api_actor`）、插件及其市场、版本以及所属成员；该事件不包含任何名称。下载组织所有插件的归档文件不会记录任何内容。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/versions/pluginver_01Km7tL4pR9xF5sU2zV3jP6q/content" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -o plugin_01Hq3vX8kZcN2mB7pR4tY9wL_pluginver_01Km7tL4pR9xF5sU2zV3jP6q.zip
  ```

  ```bash CLI
  ant beta:organization:plugins:versions download \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
    --version pluginver_01Km7tL4pR9xF5sU2zV3jP6q \
    --output plugin_01Hq3vX8kZcN2mB7pR4tY9wL_pluginver_01Km7tL4pR9xF5sU2zV3jP6q.zip
  ```

  ```python Python
  client = anthropic.Anthropic()

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  version_id = "pluginver_01Km7tL4pR9xF5sU2zV3jP6q"

  with client.beta.organization.plugins.versions.with_streaming_response.download(
      version_id,
      plugin_id=plugin_id,
  ) as response:
      response.stream_to_file(f"{plugin_id}_{version_id}.zip")
  ```

  ```typescript TypeScript
  import { writeFile } from "node:fs/promises";

  const client = new Anthropic();

  const pluginId = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL";
  const versionId = "pluginver_01Km7tL4pR9xF5sU2zV3jP6q";

  const response = await client.beta.organization.plugins.versions.download(versionId, {
    plugin_id: pluginId
  });
  if (!response.body) {
    throw new Error("The download returned no body");
  }

  await writeFile(`${pluginId}_${versionId}.zip`, response.body);
  ```

  ```csharp C#
  AnthropicClient client = new();

  using var response = await client.Beta.Organization.Plugins.Versions.Download(
      "pluginver_01Km7tL4pR9xF5sU2zV3jP6q",
      new() { PluginID = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL" }
  );

  using var content = await response.ReadAsStream();
  using var file = File.Create(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL_pluginver_01Km7tL4pR9xF5sU2zV3jP6q.zip"
  );
  await content.CopyToAsync(file);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Beta.Organization.Plugins.Versions.Download(
  	context.Background(),
  	"pluginver_01Km7tL4pR9xF5sU2zV3jP6q",
  	anthropic.BetaOrganizationPluginVersionDownloadParams{
  		PluginID: "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }
  defer response.Body.Close()

  out, err := os.Create("plugin_01Hq3vX8kZcN2mB7pR4tY9wL_pluginver_01Km7tL4pR9xF5sU2zV3jP6q.zip")
  if err != nil {
  	log.Fatal(err)
  }
  defer out.Close()

  if _, err := io.Copy(out, response.Body); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.core.http.HttpResponse;
  import com.anthropic.models.beta.organization.plugins.versions.VersionDownloadParams;

  void main() throws Exception {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = VersionDownloadParams.builder()
          .pluginId("plugin_01Hq3vX8kZcN2mB7pR4tY9wL")
          .build();
      try (HttpResponse response = client.beta().organization().plugins().versions()
              .download("pluginver_01Km7tL4pR9xF5sU2zV3jP6q", params)) {
          Files.copy(
              response.body(),
              Path.of("plugin_01Hq3vX8kZcN2mB7pR4tY9wL_pluginver_01Km7tL4pR9xF5sU2zV3jP6q.zip"),
              StandardCopyOption.REPLACE_EXISTING);
      }
  }
  ```

  ```php PHP
  $client = new Client();

  // download() 会将整个归档作为单个字符串返回，因此改为获取原始
  // 响应，并将其响应体分块复制到磁盘。
  $response = $client->beta->organization->plugins->versions->raw->download(
      version: 'pluginver_01Km7tL4pR9xF5sU2zV3jP6q',
      params: ['pluginID' => 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL'],
  );

  $archive = $response->getBody();
  $file = fopen('plugin_01Hq3vX8kZcN2mB7pR4tY9wL_pluginver_01Km7tL4pR9xF5sU2zV3jP6q.zip', 'wb');
  while (!$archive->eof()) {
      fwrite($file, $archive->read(1024 * 1024));
  }
  fclose($file);
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  version_id = "pluginver_01Km7tL4pR9xF5sU2zV3jP6q"
  archive = client.beta.organization.plugins.versions.download(version_id, plugin_id: plugin_id)

  # SDK 返回的归档已读入内存，形式为 StringIO。
  File.binwrite("#{plugin_id}_#{version_id}.zip", archive.string)
  ```
</CodeGroup>

## 插件安装设置

这些端点适用于组织拥有的插件。对于成员拥有的插件，它们返回 `404`，成员拥有的插件使用共享（请参阅[插件共享](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#plugin-shares)）。`{target}` 为字面值 `organization` 时表示插件的组织范围设置，为群组的 `rbac_group_` ID 时表示该群组的设置；任何其他值返回 `400`。群组 ID 来自 `GET /v1/organizations/rbac_groups`（作用域 `read:rbac_groups`；请参阅[用户管理中的群组](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#groups)）。设置本身没有 `id`：它通过 `(plugin_id, target)` 寻址，且不在其上记录操作者（操作者记录在其 `plugin_installation_preference_updated` 活动事件上）。

### 列出插件的安装设置

`GET /v1/organizations/plugins/{plugin_id}/installation_settings` 列出组织所有插件持有的设置，按 `created_at` 降序排列：其自身的组织范围设置（在继承其市场默认值期间不存在）以及每个组的设置。可按 `target_type`（`organization` 或 `rbac_group`）筛选。需要 `read:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/installation_settings" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins:installation-settings list \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL
  ```

  ```python Python
  client = anthropic.Anthropic()

  settings = client.beta.organization.plugins.installation_settings.list(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  )

  # 根据需要自动获取更多页面。
  for setting in settings:
      print(f"{setting.plugin_id}: {setting.installation_preference}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const settings = await client.beta.organization.plugins.installationSettings.list(
    "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  );

  for await (const setting of settings) {
    console.log(`${setting.plugin_id}: ${setting.installation_preference}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.Plugins.InstallationSettings.List(
      "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  );

  await foreach (var setting in page.Paginate())
  {
      Console.WriteLine($"{setting.PluginID}: {setting.InstallationPreference.Raw()}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  settings := client.Beta.Organization.Plugins.InstallationSettings.ListAutoPaging(
  	context.Background(),
  	"plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	anthropic.BetaOrganizationPluginInstallationSettingListParams{},
  )

  for settings.Next() {
  	setting := settings.Current()
  	fmt.Printf("%s: %s\n", setting.PluginID, setting.InstallationPreference)
  }
  if err := settings.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var settings = client.beta().organization().plugins().installationSettings()
      .list("plugin_01Hq3vX8kZcN2mB7pR4tY9wL");

  for (var setting : settings.autoPager()) {
      IO.println(setting.pluginId() + ": " + setting.installationPreference().asString());
  }
  ```

  ```php PHP
  $client = new Client();

  $settings = $client->beta->organization->plugins->installationSettings->list(
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
  );

  // 仅限当前页；如需下一页，请使用 page: $settings->nextPage 再次调用 list()。
  foreach ($settings->getItems() as $setting) {
      echo "{$setting->pluginID}: {$setting->installationPreference}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  page = client.beta.organization.plugins.installation_settings.list(plugin_id)

  page.auto_paging_each do |setting|
    puts "#{setting.plugin_id}: #{setting.installation_preference}"
  end
  ```
</CodeGroup>

### 设置安装设置

`POST /v1/organizations/plugins/{plugin_id}/installation_settings/{target}` 为组织拥有的插件设置某个目标的安装设置，可创建该设置或更改其已有的值。请求体的唯一字段是 `installation_preference`（`required`、`auto_install`、`available` 或 `not_available`），且为必填。设置为目标已有的值不会产生任何更改。设置 `organization` 目标会使插件停止继承其市场的默认值（`organization_installation_preference_inherited` 变为 `false`），即使该值与默认值相同也是如此，因此插件将不再跟随市场默认值的后续更改。要让插件恢复为其市场的默认值，请[移除组织范围设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#remove-an-installation-setting)；组设置不受影响。组目标必须是您的组织在 `GET /v1/organizations/rbac_groups` 中可见的组，否则请求返回 `404`。该更改不会改变插件的 `updated_at`；它会记录在活动源上。返回该设置。需要 `write:plugins` 范围。

请逐个发送同一插件的安装设置写入请求。如果同一插件的多个写入请求同时到达，服务器会依次处理它们，并可能对其中一些返回 `503` 而不应用它们。该 `503` 带有 `x-should-retry: true`，并且该写入可以安全地重复：等待一两秒，然后再次发送。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/installation_settings/rbac_group_01F3xQqQzXyWvUtSrQpOnMlK" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -d '{"installation_preference": "available"}'
  ```

  ```bash CLI
  ant beta:organization:plugins:installation-settings set \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
    --target rbac_group_01F3xQqQzXyWvUtSrQpOnMlK \
    --installation-preference available
  ```

  ```python Python
  client = anthropic.Anthropic()

  setting = client.beta.organization.plugins.installation_settings.set(
      "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
      plugin_id="plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
      installation_preference="available",
  )

  print(f"plugin_id: {setting.plugin_id}")
  print(f"installation_preference: {setting.installation_preference}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const setting = await client.beta.organization.plugins.installationSettings.set(
    "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
    {
      plugin_id: "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
      installation_preference: "available"
    }
  );

  console.log(`plugin_id: ${setting.plugin_id}`);
  console.log(`installation_preference: ${setting.installation_preference}`);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.Plugins.InstallationSettings;

  AnthropicClient client = new();

  var setting = await client.Beta.Organization.Plugins.InstallationSettings.Set(
      "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
      new()
      {
          PluginID = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
          InstallationPreference = InstallationPreference.Available,
      }
  );

  Console.WriteLine($"plugin_id: {setting.PluginID}");
  Console.WriteLine($"installation_preference: {setting.InstallationPreference.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  setting, err := client.Beta.Organization.Plugins.InstallationSettings.Set(
  	context.Background(),
  	"rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
  	anthropic.BetaOrganizationPluginInstallationSettingSetParams{
  		PluginID:               "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  		InstallationPreference: anthropic.BetaOrganizationPluginInstallationSettingSetParamsInstallationPreferenceAvailable,
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("plugin_id: %s\n", setting.PluginID)
  fmt.Printf("installation_preference: %s\n", setting.InstallationPreference)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.plugins.installationsettings.InstallationSettingSetParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = InstallationSettingSetParams.builder()
          .pluginId("plugin_01Hq3vX8kZcN2mB7pR4tY9wL")
          .installationPreference(InstallationSettingSetParams.InstallationPreference.AVAILABLE)
          .build();
      var setting = client.beta().organization().plugins().installationSettings()
          .set("rbac_group_01F3xQqQzXyWvUtSrQpOnMlK", params);

      IO.println("plugin_id: " + setting.pluginId());
      IO.println("installation_preference: " + setting.installationPreference().asString());
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\Plugins\InstallationSettings\InstallationSettingSetParams\InstallationPreference;
  // ...

  $client = new Client();

  $setting = $client->beta->organization->plugins->installationSettings->set(
      target: 'rbac_group_01F3xQqQzXyWvUtSrQpOnMlK',
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
      installationPreference: InstallationPreference::AVAILABLE,
  );

  echo "plugin_id: {$setting->pluginID}\n";
  echo "installation_preference: {$setting->installationPreference}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  group_id = "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK"
  setting = client.beta.organization.plugins.installation_settings.set(
    group_id,
    plugin_id: plugin_id,
    installation_preference: :available
  )

  puts "plugin_id: #{setting.plugin_id}"
  puts "installation_preference: #{setting.installation_preference}"
  ```
</CodeGroup>

```json
{
  "type": "plugin_installation_setting",
  "plugin_id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  "target": { "type": "rbac_group", "rbac_group_id": "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK" },
  "installation_preference": "available",
  "created_at": "2026-09-02T10:00:00Z",
  "updated_at": "2026-09-02T10:00:00Z"
}
```

### 移除安装设置

`DELETE /v1/organizations/plugins/{plugin_id}/installation_settings/{target}` 移除组织拥有插件的某个目标的安装设置。当 `{target}` 为 `organization` 时，它会移除插件自身的组织范围设置。之后插件将再次继承其市场的默认值（`organization_installation_preference_inherited` 变为 `true`），并跟随该默认值的后续更改；组设置不受影响。当使用组的 `rbac_group_` ID 时，它会移除该组的设置：该组的成员将回退到组织范围的值，或回退到其所属其他组的设置。没有自身设置的目标会返回 `404`；这包括已继承其市场默认值的插件的 `organization` 目标。该更改不会改变插件的 `updated_at`；它会记录在活动源上。响应以复合键代替 `id`；对于 `organization` 目标，其 `target` 为 `{"type": "organization"}`。需要 `write:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/plugins/plugin_01Hq3vX8kZcN2mB7pR4tY9wL/installation_settings/rbac_group_01F3xQqQzXyWvUtSrQpOnMlK" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins:installation-settings remove \
    --plugin-id plugin_01Hq3vX8kZcN2mB7pR4tY9wL \
    --target rbac_group_01F3xQqQzXyWvUtSrQpOnMlK
  ```

  ```python Python
  client = anthropic.Anthropic()

  removed_setting = client.beta.organization.plugins.installation_settings.remove(
      "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
      plugin_id="plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  )

  print(f"plugin_id: {removed_setting.plugin_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const removedSetting = await client.beta.organization.plugins.installationSettings.remove(
    "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
    { plugin_id: "plugin_01Hq3vX8kZcN2mB7pR4tY9wL" }
  );

  console.log(`plugin_id: ${removedSetting.plugin_id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var removedSetting = await client.Beta.Organization.Plugins.InstallationSettings.Remove(
      "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
      new() { PluginID = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL" }
  );

  Console.WriteLine($"plugin_id: {removedSetting.PluginID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  removedSetting, err := client.Beta.Organization.Plugins.InstallationSettings.Remove(
  	context.Background(),
  	"rbac_group_01F3xQqQzXyWvUtSrQpOnMlK",
  	anthropic.BetaOrganizationPluginInstallationSettingRemoveParams{
  		PluginID: "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("plugin_id: %s\n", removedSetting.PluginID)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.plugins.installationsettings.InstallationSettingRemoveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = InstallationSettingRemoveParams.builder()
          .pluginId("plugin_01Hq3vX8kZcN2mB7pR4tY9wL")
          .build();
      var removedSetting = client.beta().organization().plugins().installationSettings()
          .remove("rbac_group_01F3xQqQzXyWvUtSrQpOnMlK", params);

      IO.println("plugin_id: " + removedSetting.pluginId());
  }
  ```

  ```php PHP
  $client = new Client();

  $removedSetting = $client->beta->organization->plugins->installationSettings->remove(
      target: 'rbac_group_01F3xQqQzXyWvUtSrQpOnMlK',
      pluginID: 'plugin_01Hq3vX8kZcN2mB7pR4tY9wL',
  );

  echo "plugin_id: {$removedSetting->pluginID}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
  group_id = "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK"
  removed_setting = client.beta.organization.plugins.installation_settings.remove(
    group_id,
    plugin_id: plugin_id
  )

  puts "plugin_id: #{removed_setting.plugin_id}"
  ```
</CodeGroup>

```json
{
  "type": "plugin_installation_setting_deleted",
  "plugin_id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL",
  "target": { "type": "rbac_group", "rbac_group_id": "rbac_group_01F3xQqQzXyWvUtSrQpOnMlK" }
}
```

## 插件共享

共享仅存在于成员所有的插件上，并且在此 API 中为只读（请参阅[共享](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#shares)）。

### 列出插件的共享

`GET /v1/organizations/plugins/{plugin_id}/shares` 列出成员所有插件的所有者已将其共享给了谁，按 `granted_at` 降序排列：所有成员（`organization`）、某个组（`rbac_group`）或指定成员（`organization_member`）。可按 `target_type` 筛选。所有者未共享的插件返回空列表；组织所有的插件返回 `404`。共享在此 API 上为只读，并且列出的共享仅在您的组织在 claude.ai 中启用了该类共享时才授予访问权限（请参阅[共享](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#shares)）。`granted_at` 是授予共享的时间；如果所有者之后在 claude.ai 中更改了该共享，则为该更改的时间。需要 `read:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugins/plugin_01Mr2wP7sU3aJ8vX5cY6nS9t/shares" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugins:shares list \
    --plugin-id plugin_01Mr2wP7sU3aJ8vX5cY6nS9t
  ```

  ```python Python
  client = anthropic.Anthropic()

  shares = client.beta.organization.plugins.shares.list("plugin_01Mr2wP7sU3aJ8vX5cY6nS9t")

  # 根据需要自动获取更多页面。
  for share in shares:
      print(f"plugin_id: {share.plugin_id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const shares = await client.beta.organization.plugins.shares.list(
    "plugin_01Mr2wP7sU3aJ8vX5cY6nS9t"
  );

  for await (const share of shares) {
    console.log(`plugin_id: ${share.plugin_id}`);
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.Plugins.Shares.List("plugin_01Mr2wP7sU3aJ8vX5cY6nS9t");

  await foreach (var share in page.Paginate())
  {
      Console.WriteLine($"plugin_id: {share.PluginID}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  shares := client.Beta.Organization.Plugins.Shares.ListAutoPaging(
  	context.Background(),
  	"plugin_01Mr2wP7sU3aJ8vX5cY6nS9t",
  	anthropic.BetaOrganizationPluginShareListParams{},
  )

  for shares.Next() {
  	share := shares.Current()
  	fmt.Printf("plugin_id: %s\n", share.PluginID)
  }
  if err := shares.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var shares = client.beta().organization().plugins().shares()
      .list("plugin_01Mr2wP7sU3aJ8vX5cY6nS9t");

  for (var share : shares.autoPager()) {
      IO.println("plugin_id: " + share.pluginId());
  }
  ```

  ```php PHP
  $client = new Client();

  $shares = $client->beta->organization->plugins->shares->list(
      pluginID: 'plugin_01Mr2wP7sU3aJ8vX5cY6nS9t',
  );

  // 仅限当前页；如需下一页，请使用 page: $shares->nextPage 再次调用 list()。
  foreach ($shares->getItems() as $share) {
      echo "plugin_id: {$share->pluginID}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  plugin_id = "plugin_01Mr2wP7sU3aJ8vX5cY6nS9t"
  page = client.beta.organization.plugins.shares.list(plugin_id)

  page.auto_paging_each do |share|
    puts "plugin_id: #{share.plugin_id}"
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "type": "plugin_share",
      "plugin_id": "plugin_01Mr2wP7sU3aJ8vX5cY6nS9t",
      "target": { "type": "organization_member", "user_id": "user_01WCz9BvGLdMYRUMmcAxMWvW" },
      "granted_at": "2026-08-20T15:12:00Z"
    }
  ],
  "next_page": null
}
```

## 插件市场

此 API 读取[市场](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#marketplaces)并设置组织市场的默认安装设置；市场本身的创建、与存储库的连接以及删除均在 claude.ai 中进行。

```json
{
  "type": "plugin_marketplace",
  "id": "marketplace_01VbNcMxZaSdFgHjKlQwErTy",
  "name": "engineering-tools",
  "owner": { "type": "organization" },
  "source": "github",
  "sync_status": "success",
  "last_sync_ended_at": "2026-09-10T22:15:03Z",
  "last_sync_read_sha": "9fceb02d0ae598e95dc970b74767f19372d61af8",
  "default_installation_preference": "available",
  "created_at": "2026-06-12T08:45:00Z"
}
```

| 字段                                | 描述                                                                                                                                                   |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                            | 市场的名称。在其整个生命周期内固定不变。                                                                                                                                 |
| `owner`                           | 结构与插件上的相同。                                                                                                                                           |
| `source`                          | `manual`、`github`、`gitlab` 或 `public_git`。请参阅[市场](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#marketplaces)。                    |
| `sync_status`                     | 最近一次同步的结果：`success`、`in_progress`、`failed_content`、`failed_transient`、`failed_auth` 或 `failed_limits`。在首次尝试同步之前为 `null`，对于来源为 `manual` 的市场，永远不会进行同步。 |
| `last_sync_ended_at`              | 最近一次同步尝试完成的时间，无论其结果如何；对于尚未同步的已连接存储库，为市场的创建时间。对于不进行同步的市场为 `null`。                                                                                     |
| `last_sync_read_sha`              | 上次同步从存储库读取的提交。不一定是提供版本所来自的提交。对于不进行同步的市场为 `null`。                                                                                                     |
| `default_installation_preference` | 组织市场：其中每个没有自身设置的插件所采用的组织范围值（如果从未设置，则为 `not_available`）。个人市场：`null`。                                                                                  |

缺少 `marketplace_` 前缀的 `marketplace_id` 返回 `400`。带有该前缀但无法解析或属于其他组织的值返回 `404`。

### 列出市场

`GET /v1/organizations/plugin_marketplaces` 列出您组织的市场和成员的个人市场，按 `created_at` 降序排列。您可以使用它在市场包含任何插件之前找到其 ID，以便按该市场筛选插件列表或向其上传。库市场会在首次在其中创建内容（无论是在 claude.ai 中还是通过此 API）后出现。可按 `owner_type`（`organization` 或 `user`）和 `source` 筛选。`limit` 为 1 到 1,000。需要 `read:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugin_marketplaces?owner_type=organization" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugin-marketplaces list --owner-type organization
  ```

  ```python Python
  client = anthropic.Anthropic()

  marketplaces = client.beta.organization.plugin_marketplaces.list(
      owner_type="organization"
  )

  # 根据需要自动获取更多页面。
  for marketplace in marketplaces:
      print(f"{marketplace.id}: {marketplace.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const marketplaces = await client.beta.organization.pluginMarketplaces.list({
    owner_type: "organization"
  });

  for await (const marketplace of marketplaces) {
    console.log(`${marketplace.id}: ${marketplace.name}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.PluginMarketplaces;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.PluginMarketplaces.List(
      new() { OwnerType = OwnerType.Organization }
  );

  await foreach (var marketplace in page.Paginate())
  {
      Console.WriteLine($"{marketplace.ID}: {marketplace.Name}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  marketplaces := client.Beta.Organization.PluginMarketplaces.ListAutoPaging(
  	context.Background(),
  	anthropic.BetaOrganizationPluginMarketplaceListParams{
  		OwnerType: anthropic.BetaOrganizationPluginMarketplaceListParamsOwnerTypeOrganization,
  	},
  )

  for marketplaces.Next() {
  	marketplace := marketplaces.Current()
  	fmt.Printf("%s: %s\n", marketplace.ID, marketplace.Name)
  }
  if err := marketplaces.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.pluginmarketplaces.PluginMarketplaceListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = PluginMarketplaceListParams.builder()
          .ownerType(PluginMarketplaceListParams.OwnerType.ORGANIZATION)
          .build();
      var marketplaces = client.beta().organization().pluginMarketplaces().list(params);

      for (var marketplace : marketplaces.autoPager()) {
          IO.println(marketplace.id() + ": " + marketplace.name());
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\PluginMarketplaces\PluginMarketplaceListParams\OwnerType;
  // ...

  $client = new Client();

  $marketplaces = $client->beta->organization->pluginMarketplaces->list(
      ownerType: OwnerType::ORGANIZATION,
  );

  // 仅限当前页；如需下一页，请使用 page: $marketplaces->nextPage 再次调用 list()。
  foreach ($marketplaces->getItems() as $marketplace) {
      echo "{$marketplace->id}: {$marketplace->name}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.plugin_marketplaces.list(owner_type: :organization)

  page.auto_paging_each do |marketplace|
    puts "#{marketplace.id}: #{marketplace.name}"
  end
  ```
</CodeGroup>

### 获取市场

`GET /v1/organizations/plugin_marketplaces/{marketplace_id}` 返回一个市场。需要 `read:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/plugin_marketplaces/marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01"
  ```

  ```bash CLI
  ant beta:organization:plugin-marketplaces retrieve \
    --marketplace-id marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r
  ```

  ```python Python
  client = anthropic.Anthropic()

  marketplace = client.beta.organization.plugin_marketplaces.retrieve(
      "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  )

  print(f"id: {marketplace.id}")
  print(f"name: {marketplace.name}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const marketplace = await client.beta.organization.pluginMarketplaces.retrieve(
    "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  );

  console.log(`id: ${marketplace.id}`);
  console.log(`name: ${marketplace.name}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var marketplace = await client.Beta.Organization.PluginMarketplaces.Retrieve(
      "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  );

  Console.WriteLine($"id: {marketplace.ID}");
  Console.WriteLine($"name: {marketplace.Name}");
  ```

  ```go Go
  client := anthropic.NewClient()

  marketplace, err := client.Beta.Organization.PluginMarketplaces.Get(
  	context.Background(),
  	"marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
  	anthropic.BetaOrganizationPluginMarketplaceGetParams{},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", marketplace.ID)
  fmt.Printf("name: %s\n", marketplace.Name)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var marketplace = client.beta().organization().pluginMarketplaces()
      .retrieve("marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r");

  IO.println("id: " + marketplace.id());
  IO.println("name: " + marketplace.name());
  ```

  ```php PHP
  $client = new Client();

  $marketplace = $client->beta->organization->pluginMarketplaces->retrieve(
      marketplaceID: 'marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r',
  );

  echo "id: {$marketplace->id}\n";
  echo "name: {$marketplace->name}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  marketplace_id = "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  marketplace = client.beta.organization.plugin_marketplaces.retrieve(marketplace_id)

  puts "id: #{marketplace.id}"
  puts "name: #{marketplace.name}"
  ```
</CodeGroup>

### 设置市场的默认安装设置

`POST /v1/organizations/plugin_marketplaces/{marketplace_id}` 设置组织所有市场的默认安装设置。市场中每个没有自身组织范围设置的插件都会将此默认值报告为其 `organization_installation_preference`，包括之后添加的插件。它适用于 `manual` 市场和同步市场；成员的个人市场返回 `403`。唯一可更新的字段是 `default_installation_preference`，且为必填。它无法设回 `null`：市场一旦有了默认值，就会一直保留，这与 claude.ai 中相同。更改会记录为一个 `marketplace_updated` 事件，不会产生按插件的事件，也不会改变任何插件的 `updated_at`。设置为已设置的值不会产生任何更改，但有一个例外：从未设置默认值的市场会报告 `not_available`，但实际并不持有设置，因此其第一次写入（即使是 `not_available`）也算作更改。返回该市场。需要 `write:plugins` 范围。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugin_marketplaces/marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -d '{"default_installation_preference": "available"}'
  ```

  ```bash CLI
  ant beta:organization:plugin-marketplaces update \
    --marketplace-id marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r \
    --default-installation-preference available
  ```

  ```python Python
  client = anthropic.Anthropic()

  marketplace = client.beta.organization.plugin_marketplaces.update(
      "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
      default_installation_preference="available",
  )

  print(f"id: {marketplace.id}")
  print(f"default_installation_preference: {marketplace.default_installation_preference}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const marketplace = await client.beta.organization.pluginMarketplaces.update(
    "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
    { default_installation_preference: "available" }
  );

  console.log(`id: ${marketplace.id}`);
  console.log(`default_installation_preference: ${marketplace.default_installation_preference}`);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.PluginMarketplaces;

  AnthropicClient client = new();

  var marketplace = await client.Beta.Organization.PluginMarketplaces.Update(
      "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
      new() { DefaultInstallationPreference = DefaultInstallationPreference.Available }
  );

  Console.WriteLine($"id: {marketplace.ID}");
  Console.WriteLine(
      $"default_installation_preference: {marketplace.DefaultInstallationPreference?.Raw()}"
  );
  ```

  ```go Go
  client := anthropic.NewClient()

  marketplace, err := client.Beta.Organization.PluginMarketplaces.Update(
  	context.Background(),
  	"marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r",
  	anthropic.BetaOrganizationPluginMarketplaceUpdateParams{
  		DefaultInstallationPreference: anthropic.BetaOrganizationPluginMarketplaceUpdateParamsDefaultInstallationPreferenceAvailable,
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", marketplace.ID)
  fmt.Printf("default_installation_preference: %s\n", marketplace.DefaultInstallationPreference)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.pluginmarketplaces.PluginMarketplaceUpdateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = PluginMarketplaceUpdateParams.builder()
          .defaultInstallationPreference(PluginMarketplaceUpdateParams.DefaultInstallationPreference.AVAILABLE)
          .build();
      var marketplace = client.beta().organization().pluginMarketplaces()
          .update("marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r", params);

      IO.println("id: " + marketplace.id());
      IO.println("default_installation_preference: "
          + marketplace.defaultInstallationPreference().map(preference -> preference.asString()).orElse(null));
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\PluginMarketplaces\PluginMarketplaceUpdateParams\DefaultInstallationPreference;
  // ...

  $client = new Client();

  $marketplace = $client->beta->organization->pluginMarketplaces->update(
      marketplaceID: 'marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r',
      defaultInstallationPreference: DefaultInstallationPreference::AVAILABLE,
  );

  echo "id: {$marketplace->id}\n";
  echo "default_installation_preference: {$marketplace->defaultInstallationPreference}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  marketplace_id = "marketplace_01Lp8uM5qS1yG6tV3aW4kQ7r"
  marketplace = client.beta.organization.plugin_marketplaces.update(
    marketplace_id,
    default_installation_preference: :available
  )

  puts "id: #{marketplace.id}"
  puts "default_installation_preference: #{marketplace.default_installation_preference}"
  ```
</CodeGroup>

### 验证市场内容

有两个端点可以报告对给定市场内容执行同步会产生什么结果，而不会连接或存储任何内容：`POST /v1/organizations/plugin_marketplaces/validate_repository` 读取公开的 GitHub 仓库，`POST /v1/organizations/plugin_marketplaces/validate_archive` 读取您上传的市场目录 `.zip` 文件。两者返回相同的报告：`marketplace.json` 格式是否正确、哪些插件会被跳过及其原因，以及哪些插件在同步时会遗漏部分内容。这些检查与实际同步所执行的检查相同。内容问题会在报告中返回，而不是作为 HTTP 错误返回：即使仓库或归档完全无法读取，请求也会成功并返回 `valid: false`。一次验证计为一次读取，并且这两个端点合计还额外限制为每个组织每分钟 10 次验证（请参阅[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#rate-limiting)）；它们不会在活动源上记录任何内容。一次验证最多可能需要 120 秒才会返回，因此请将客户端的超时时间设置为高于该值。两个端点都需要 `read:plugins` 或 `write:plugins` 作用域（`read:org_audit` 和 `read:compliance_org_data` 不授予这些作用域）。

验证以匿名方式读取仓库。对于位于 GitHub 上但在该仓库之外的任何插件源，验证也采用相同方式。因此，验证会将私有仓库或私有插件源报告为未找到或需要登录 GitHub。当验证报告需要登录 GitHub 时，报告代码为 `marketplace_repo_sign_in_required`。

验证不会获取 GitHub 以外主机上的插件源。具有此类源的插件通常会收到 `marketplace_validate_source_not_checked` 警告，并会在市场实际同步时进行检查。

对于 Anthropic 同步到每个组织中的市场，验证会应用更严格的规则。当仓库是此类市场时，`validate_repository` 会应用这些规则。当归档指定了此类市场时，`validate_archive` 会应用这些规则。更严格的规则如下：

* 该市场之外的每个插件源都必须固定到完整的提交 SHA。
* 验证会将未固定的源和位于不受支持主机上的源报告为插件错误。
* 对于 `validate_repository`，验证读取的分支默认为该市场进行同步时所用的分支。

`validate_repository` 接受包含两个字段的 JSON 请求体：`repository_url`，即 github.com 上公开仓库的 `https://` URL（必填）；以及 `ref`，即分支名称或完整的 40 字符提交 SHA（可选）。当省略 `ref` 或其为 `null` 时，默认使用同步时会读取的分支，通常是仓库的默认分支。`validate_archive` 接受 `multipart/form-data`，其中恰好包含一个部分 `archive`，以带文件名的文件部分形式发送。该文件是市场目录的 `.zip`，最大 32 MB，且仅使用 DEFLATE 或 STORE 压缩。归档内容位于根目录，或包裹在一个文件夹中（如 Git 托管平台下载所生成的那样）。不接受其他表单字段。

验证某个分支上的公共存储库：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugin_marketplaces/validate_repository" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -d '{"repository_url": "https://github.com/example-org/claude-plugins", "ref": "release-candidate"}'
  ```

  ```bash CLI
  ant beta:organization:plugin-marketplaces validate-repository \
    --repository-url https://github.com/example-org/claude-plugins \
    --ref release-candidate
  ```

  ```python Python
  client = anthropic.Anthropic()

  report = client.beta.organization.plugin_marketplaces.validate_repository(
      repository_url="https://github.com/example-org/claude-plugins",
      ref="release-candidate",
  )

  print(f"valid: {str(report.valid).lower()}")
  print(f"total_plugin_count: {report.total_plugin_count}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const report = await client.beta.organization.pluginMarketplaces.validateRepository({
    repository_url: "https://github.com/example-org/claude-plugins",
    ref: "release-candidate"
  });

  console.log(`valid: ${report.valid}`);
  console.log(`total_plugin_count: ${report.total_plugin_count}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var report = await client.Beta.Organization.PluginMarketplaces.ValidateRepository(
      new()
      {
          RepositoryUrl = "https://github.com/example-org/claude-plugins",
          Ref = "release-candidate",
      }
  );

  Console.WriteLine($"valid: {report.Valid.ToString().ToLowerInvariant()}");
  Console.WriteLine($"total_plugin_count: {report.TotalPluginCount}");
  ```

  ```go Go
  client := anthropic.NewClient()

  report, err := client.Beta.Organization.PluginMarketplaces.ValidateRepository(
  	context.Background(),
  	anthropic.BetaOrganizationPluginMarketplaceValidateRepositoryParams{
  		RepositoryURL: "https://github.com/example-org/claude-plugins",
  		Ref:           anthropic.String("release-candidate"),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("valid: %t\n", report.Valid)
  fmt.Printf("total_plugin_count: %d\n", report.TotalPluginCount)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.pluginmarketplaces.PluginMarketplaceValidateRepositoryParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = PluginMarketplaceValidateRepositoryParams.builder()
          .repositoryUrl("https://github.com/example-org/claude-plugins")
          .ref("release-candidate")
          .build();
      var report = client.beta().organization().pluginMarketplaces().validateRepository(params);

      IO.println("valid: " + report.valid());
      IO.println("total_plugin_count: " + report.totalPluginCount());
  }
  ```

  ```php PHP
  $client = new Client();

  $report = $client->beta->organization->pluginMarketplaces->validateRepository(
      repositoryURL: 'https://github.com/example-org/claude-plugins',
      ref: 'release-candidate',
  );

  echo 'valid: ' . ($report->valid ? 'true' : 'false') . "\n";
  echo "total_plugin_count: {$report->totalPluginCount}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  report = client.beta.organization.plugin_marketplaces.validate_repository(
    repository_url: "https://github.com/example-org/claude-plugins",
    ref: "release-candidate"
  )

  puts "valid: #{report.valid}"
  puts "total_plugin_count: #{report.total_plugin_count}"
  ```
</CodeGroup>

```json
{
  "type": "plugin_marketplace_validation_report",
  "valid": false,
  "ref": "release-candidate",
  "commit_sha": "9fceb02d0ae598e95dc970b74767f19372d61af8",
  "total_plugin_count": 3,
  "manifest_error": null,
  "manifest_error_code": null,
  "plugin_errors": [
    {
      "name": "deploy-helper",
      "error": "The plugin has a top-level bin/ directory.",
      "error_code": "marketplace_sync_bin_directory_not_allowed"
    }
  ],
  "plugin_warnings": [
    {
      "name": "release-notes",
      "warnings": [
        {
          "message": "plugin.json has unrecognized top-level keys: owners",
          "error_code": "marketplace_sync_plugin_unrecognized_keys"
        }
      ]
    }
  ]
}
```

| 字段                                     | 描述                                                                                                                                       |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `valid`                                | 当 `marketplace.json` 格式正确且没有插件会被跳过时为 `true`。警告不会使其变为 `false`。                                                                            |
| `ref`                                  | 所读取分支的名称；当未指定分支而读取了默认分支、指定的是提交 SHA 或验证的是归档文件时为 `null`。                                                                                   |
| `commit_sha`                           | 所验证的提交。对于从 Git 主机下载的归档文件，为主机记录在 ZIP 文件注释字段中的提交（如果有；未经验证）。                                                                                |
| `total_plugin_count`                   | `marketplace.json` 声明的插件数量；无法读取时为 `0`。                                                                                                   |
| `manifest_error`、`manifest_error_code` | 在无法验证任何内容时设置：来源无法读取，或 `marketplace.json` 缺失、格式错误或超出限制。未在 120 秒内完成的验证会报告 `manifest_error_code: "marketplace_validate_deadline_exceeded"`。 |
| `plugin_errors`                        | 同步会跳过的每个插件各对应一个 `{name, error, error_code}`。                                                                                             |
| `plugin_warnings`                      | 同步时会遗漏部分内容的每个插件各对应一个 `{name, warnings: [{message, error_code}]}`。                                                                        |

您也可以改为以 `.zip` 形式验证市场目录的本地副本；响应是相同的报告：

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/plugin_marketplaces/validate_archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: ce-plugins-2026-09-01" \
    -F "archive=@marketplace.zip"
  ```

  ```bash CLI
  ant beta:organization:plugin-marketplaces validate-archive \
    --archive marketplace.zip
  ```

  ```python Python
  client = anthropic.Anthropic()

  with open("marketplace.zip", "rb") as archive:
      report = client.beta.organization.plugin_marketplaces.validate_archive(
          archive=archive
      )

  print(f"valid: {str(report.valid).lower()}")
  print(f"total_plugin_count: {report.total_plugin_count}")
  ```

  ```typescript TypeScript
  import fs from "node:fs";

  const client = new Anthropic();

  const report = await client.beta.organization.pluginMarketplaces.validateArchive({
    archive: fs.createReadStream("marketplace.zip")
  });

  console.log(`valid: ${report.valid}`);
  console.log(`total_plugin_count: ${report.total_plugin_count}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var report = await client.Beta.Organization.PluginMarketplaces.ValidateArchive(
      new() { Archive = File.OpenRead("marketplace.zip") }
  );

  Console.WriteLine($"valid: {report.Valid.ToString().ToLowerInvariant()}");
  Console.WriteLine($"total_plugin_count: {report.TotalPluginCount}");
  ```

  ```go Go
  client := anthropic.NewClient()

  archive, err := os.Open("marketplace.zip")
  if err != nil {
  	log.Fatal(err)
  }
  defer archive.Close()

  report, err := client.Beta.Organization.PluginMarketplaces.ValidateArchive(
  	context.Background(),
  	anthropic.BetaOrganizationPluginMarketplaceValidateArchiveParams{
  		Archive: archive,
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("valid: %t\n", report.Valid)
  fmt.Printf("total_plugin_count: %d\n", report.TotalPluginCount)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.pluginmarketplaces.PluginMarketplaceValidateArchiveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      // 传入 Path，以便发送 `archive` 部分时附带文件名，此端点要求提供文件名。
      var params = PluginMarketplaceValidateArchiveParams.builder()
          .archive(Path.of("marketplace.zip"))
          .build();
      var report = client.beta().organization().pluginMarketplaces().validateArchive(params);

      IO.println("valid: " + report.valid());
      IO.println("total_plugin_count: " + report.totalPluginCount());
  }
  ```

  ```php PHP
  use Anthropic\Core\FileParam;

  $client = new Client();

  $report = $client->beta->organization->pluginMarketplaces->validateArchive(
      archive: FileParam::fromResource(fopen('marketplace.zip', 'r')),
  );

  echo 'valid: ' . ($report->valid ? 'true' : 'false') . "\n";
  echo "total_plugin_count: {$report->totalPluginCount}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  report = client.beta.organization.plugin_marketplaces.validate_archive(
    archive: Pathname("marketplace.zip")
  )

  puts "valid: #{report.valid}"
  puts "total_plugin_count: #{report.total_plugin_count}"
  ```
</CodeGroup>

内容问题永远不会导致请求失败。除了所有端点共享的响应（仅具有 `read:org_audit` 或 `read:compliance_org_data` 的密钥返回 `403`；请参阅[错误响应](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#error-responses)和[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#rate-limiting)）之外，请求本身还可能因以下原因失败：

| 状态  | 原因                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 处理方法                    |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| 400 | 对于 `validate_repository`：请求体不是 JSON 对象；`repository_url` 缺失、长度超过 2,048 个字符、包含凭据，或不符合 `https://github.com/{owner}/{repo}` 的格式（接受 `.git` 后缀；不接受其他主机、更长的路径（例如分支页面的 `/tree/main`）或 443 或 80 以外的端口）；`ref` 为空、长度超过 255 个字符、包含 `..`，或包含 ASCII 字母、数字、`.`、`_`、`-`、`+` 和 `/` 以外的字符；或者存在其他字段。通过这些检查但指向仓库中不存在分支的 `ref` 不会被拒绝：请求会成功并返回 `valid: false`，且 `manifest_error` 会说明未找到该分支。对于 `validate_archive`：请求体不是 `multipart/form-data`，`archive` 部分缺失、重复或未以带文件名的文件部分形式发送，或者存在其他表单字段。 | 修正请求后重新发送。              |
| 413 | 对于 `validate_archive`：`archive` 部分或请求声明的请求体长度超过 32 MB。                                                                                                                                                                                                                                                                                                                                                                                                               | 改为通过 URL 验证存储库，或精简归档文件。 |

#### 报告代码

报告中的每项发现都有一个稳定的代码：无法验证任何内容时为 `manifest_error_code`，每个 `plugin_errors` 条目上为 `error_code`，每个警告上也为 `error_code`。当一个插件存在多个问题时，`error_code` 是第一个问题的代码，`error` 则合并了所有问题的消息。可能会添加新代码；无法识别的 `manifest_error_code` 仍表示内容无法验证，`plugin_errors` 条目上无法识别的代码仍表示该插件会被跳过，警告上无法识别的代码仍表示该插件会被同步。以下代码表示临时状况，因此相同的请求稍后可能会成功：`marketplace_host_rate_limited`、`marketplace_host_server_error`、`marketplace_host_timeout`、`marketplace_host_unreachable`、`marketplace_repo_access_denied`、`marketplace_sync_transient_fetch_budget_exhausted`、`marketplace_validate_network_error`，以及通常情况下的 `marketplace_validate_deadline_exceeded`。

## 无法识别的值

此页面上的每个字符串值（组件类型、`reach`、扫描字段、市场 `source`、错误代码）都可能随时增加新值。遇到无法识别的值时，请像对待任何未知字符串一样处理，而不要因此失败。

## 速率限制

读取请求（本页上的每个 `GET` 端点）共享每个组织**每分钟 300 次请求**的限制。写入请求（创建插件或版本、更改提供的版本、删除、设置或移除安装设置，以及更新市场）共享每个组织**每分钟 60 次请求**的限制。市场验证（任一端点）计为一次读取，并且这两个端点的验证合计还额外限制为每个组织每分钟 10 次；这两项限制都会在读取请求体之前进行检查。这些限制按您组织的所有密钥合计计算，并且独立于您组织的其他 Admin API 限制。传递 `organization_id` 的读取请求计入创建该密钥的组织的限制；如果密钥没有自身组织，则计入父组织的限制。超出限制的请求会返回 **429 Too Many Requests**，并附带 `retry-after` 标头。响应包含适用限制的 `anthropic-ratelimit-requests-*` 标头（对于市场验证，为其每分钟 10 次的限制；对于 `429`，为拒绝该请求的那项限制）。

当服务暂时没有容量处理另一个上传、提供版本更改或验证时，这些操作也可能返回带有 `retry-after` 的 `429`；当您的组织超出其内容扫描速率时，上传也会返回 `429`。请以相同方式处理所有这些情况：等待 `retry-after` 指定的时间，然后重试。除这些限制外，请逐个发送同一插件的安装设置写入请求：当多个请求同时到达时，其中一些可能会收到带有 `x-should-retry: true` 的 `503`，这些请求可以在一两秒后安全地再次发送（请参阅[设置安装设置](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#set-an-installation-setting)）。

## 分页

列表端点使用 **opaque cursor**（不透明游标）。第一个请求最多返回 `limit` 行以及一个 `next_page` 游标；在下一个请求中将该游标原样作为 `page` 参数传递，并重复此操作，直到 `next_page` 为 `null`。请将游标字符串视为不透明的：不要自行解析、修改或构造它。[列出插件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#list-plugins)可能会返回少于 `limit` 个插件的页面，甚至是空页面，而 `next_page` 仍然被设置，因此请持续请求页面，直到 `next_page` 为 `null`。Python、TypeScript、C#、Go、Java 和 Ruby SDK 的列表迭代器以及 CLI（使用 `--format raw` 时除外）会为您完成此操作。只要设置了 `next_page`，它们就会请求下一页，因此较短的页面或空页面不会使其终止。在 PHP 和 curl 中，请自行请求每一页，并将其 `next_page` 作为 `page` 传递。

`limit` 默认为 20，最小值为 1。插件、安装设置和共享的最大值为 100，版本和市场的最大值为 1,000。每个列表都按从新到旧的顺序排列。

## 错误响应

错误响应遵循 [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)中记录的标准格式。联系支持团队时，请提供响应体中的 `request_id`。

| 状态  | 含义                                                                                                                                                  |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400 | 输入无效，或该操作不适用于此插件或市场（请参阅各端点的相应章节）。当请求包含端点无法识别的查询参数，或组织不是 Claude Enterprise 组织时（`this endpoint is not supported for this organization type`），也会返回此状态。 |
| 401 | 缺少 `x-api-key` 标头，或无法识别该密钥。                                                                                                                         |
| 403 | 密钥缺少所需的 scope（作用域），或者请求试图向成员的插件或个人市场上传内容、更改其提供的版本或为其设置默认值。（允许删除成员的插件。）                                                                              |
| 404 | 未找到资源。当请求省略 `anthropic-beta` 值，或您的组织未启用该 API 时，也会返回此状态，使这些端点看起来不存在。                                                                                 |
| 409 | 名称已被占用、某个版本的 `content_scan` 为 `processing`，或有冲突的上传正在进行中。                                                                                            |
| 413 | 请求正文超出大小限制：上传为 200 MB，市场验证为 32 MB。                                                                                                                  |
| 429 | 超出速率限制。请参阅[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#rate-limiting)。                                                   |
| 500 | 内部错误。                                                                                                                                               |
| 503 | 临时错误。当针对同一插件的多个安装设置写入同时到达时，也会返回此状态；请逐个发送这些请求。请使用 backoff（退避）策略重试，但 `registration_pending` 除外（请参阅下表）。                                                |

当同一状态有多种需要您分别处理的原因时，错误还会携带 `error.details.error_code`，并在原因涉及某个插件或版本时携带 `error.details.plugin_id` 或 `error.details.plugin_version_id`：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "...",
    "details": {
      "error_code": "plugin_name_taken",
      "plugin_id": "plugin_01Hq3vX8kZcN2mB7pR4tY9wL"
    }
  },
  "request_id": "req_018EeWyXxfu5pfWkrYcMdjWG"
}
```

| `error_code`                                   | 状态  | 含义及处理方式                                                                                                                                                                                                                                          |
| ---------------------------------------------- | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `plugin_name_taken`                            | 409 | 市场中已存在该名称。`details.plugin_id` 即为占用该名称的插件，它可能是其他人创建的。要查看创建者，请[获取该插件](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#get-a-plugin)并读取其 `created_by`，如果未记录创建者，则该值为 `null`。仅当它是您打算更新的插件时，才向其上传版本。否则，或当 `plugin_id` 不存在时，请使用其他名称上传。 |
| `skill_name_taken`                             | 409 | 该插件位于库市场中，且其某个技能与某个组织技能（管理员在 claude.ai 中为整个组织上传的技能）同名。`details.skill_name` 给出了该名称。请重命名或移除其中之一。                                                                                                                                                   |
| `registration_pending`                         | 503 | 文件已存储，但插件的技能尚未能提供给成员使用。请参阅[重试上传](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#retrying-uploads)。                                                                                                                             |
| `scan_pending`                                 | 409 | 该版本的 `content_scan` 为 `processing`。请在扫描完成后重试。                                                                                                                                                                                                    |
| `scan_failed`                                  | 400 | 该版本的 `content_scan` 为 `fail`、`unknown` 或 `errored`，因此无法提供。请选择其他版本。                                                                                                                                                                               |
| `cmek_key_disabled`、`cmek_key_network_blocked` | 400 | 您组织的客户管理加密密钥不可用。请参阅[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#customer-managed-encryption-keys)。                                                                                                               |
| `unsupported_charset`                          | 400 | 在上传或 `validate_archive` 时，请求的 `Content-Type` 标头指定了 UTF-8、ISO-8859-1 或 US-ASCII 以外的字符集。请在不指定字符集或指定上述字符集之一的情况下发送请求。                                                                                                                                |

未来可能会添加新的代码。对于无法识别的代码，请按照其对应状态的方式处理。

### 重试上传

没有端点接受 `Idempotency-Key`。更改所提供的版本、设置安装设置以及设置市场默认值都可以安全地重复执行。重复删除或重复移除安装设置会返回 `404`。

返回错误的上传不会存储任何内容，但有一个例外：带有 `error_code: "registration_pending"` 的 `503`。在存储上传的文件后，服务器会向 claude.ai 注册新版本的技能，这一步使成员能够使用这些技能；`registration_pending` 表示文件已存储，但最后这一步未完成。再次上传相同的文件即可完成该步骤（并会再存储一个相同的版本）：

* 对于 `POST /v1/organizations/plugins`，插件*已经*创建，且响应携带 `x-should-retry: false`：请勿重新发送创建请求（重新发送会返回 `409 plugin_name_taken`）；而应将相同的文件作为 `details.plugin_id` 的一个版本上传。
* 对于 `POST /v1/organizations/plugins/{plugin_id}/versions`，版本*已经*存储（`details.plugin_version_id`）；当响应携带 `x-should-retry: true` 时，请重新发送相同的请求，当携带 `false` 时则不要重新发送。

如果创建请求的响应丢失，请再次发送创建请求。`409 plugin_name_taken` 表示第一次尝试已存储了该插件，或者该名称此前已被占用。仅当 `details.plugin_id` 中插件的 `created_by` 是带有您密钥的 `api_key_id` 的 `api_actor` 时，该插件才是由您丢失响应的创建请求所创建的。在向其上传之前，请先检查这一点（请参阅上表中的 `plugin_name_taken`）。重试一个丢失了响应的版本创建请求会存储第二个相同的版本。为避免这种情况，请在每次上传前记录插件的 `latest_version_id`；如果响应丢失，请读取该插件，仅当 `latest_version_id` 未发生变化时才重试。

## 活动源事件

通过此 API 进行的每次写入都会记录在您组织的 [Compliance API Activity Feed（活动源）](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)中，并归属于该 API 密钥，表示为携带其 `apikey_` ID 的 `api_actor`。同一执行者也会出现在该密钥所创建的插件和版本的 `created_by` 中。

| 事件                                       | 触发时机                                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------------------ |
| `claude_plugin_created`                  | 通过上传（在此处或在 claude.ai 中）或通过已接受的发布请求创建插件时。通过 Git 同步创建的插件仅触发 `claude_plugin_version_created`。 |
| `claude_plugin_version_created`          | 存储版本时。通过 Git 同步存储的版本归属于 `system_actor`。                                                    |
| `claude_plugin_updated`                  | 向现有插件上传新版本时。                                                                               |
| `claude_plugin_served_version_updated`   | 提供的版本发生变化时。                                                                                |
| `claude_plugin_deleted`                  | 插件被单独删除时（在此处或在 claude.ai 中）。                                                               |
| `plugin_installation_preference_updated` | 设置或移除安装设置时。                                                                                |
| `marketplace_created`                    | 首次上传创建库市场时。                                                                                |
| `marketplace_updated`                    | 市场的默认安装设置发生变化，或管理员或所有者在 claude.ai 中启动同步时。                                                  |
| `marketplace_deleted`                    | 在 claude.ai 中将市场连同其插件一起删除时（不会触发针对单个插件的事件）。                                                 |
| `claude_plugin_archive_accessed`         | 下载成员拥有的插件的归档文件时。                                                                           |
| `claude_plugin_security_scan_completed`  | 内容扫描完成时。                                                                                   |

这些事件中的插件、版本和市场 ID 与此 API 返回的 ID 相同。事件也可能指向此 API 未列出的市场，例如由 Anthropic 发布的插件组成的市场。`plugin_installation_preference_updated` 通过插件的 `name` 和 `marketplace_id` 而不是其 `id` 来标识插件。在 `claude_plugin_security_scan_completed` 中，`plugin_version` 不是版本 ID，[插件版本](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#plugin-versions)中也没有与之对应的字段。

更改市场的默认值只会记录一个 `marketplace_updated` 事件，不会记录针对单个插件的事件，即使这会改变所有继承该默认值的插件的值。读取操作不会被记录，但下载成员拥有的插件的归档文件除外。未产生任何更改的写入不会被记录。

在 claude.ai 中授予或撤回的共享会以 `role_assignment_granted` 和 `role_assignment_revoked` 事件的形式出现在活动源中。此 API 不会报告删除操作：已删除的插件不会出现在下一次列表中。通过 Git 同步移除的插件、因删除其所在市场而移除的插件（一个 `marketplace_deleted` 事件），或因删除成员账户或组织而移除的插件，都不会触发针对单个插件的事件，因此请定期重新列出完整清单以发现移除情况。

## 客户管理的加密密钥

如果您的组织使用 [customer-managed encryption key（客户管理的加密密钥）](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)，则版本的 `description`、`release_notes`、`components` 和文件都会使用该密钥加密。当该密钥不可用时：

* 读取和列出操作仍会成功，但 `description`、`release_notes` 和 `components` 会返回 `null`。
* 归档文件下载、创建、版本创建以及提供版本的更改会返回 `400`，并带有 `cmek_key_disabled` 或 `cmek_key_network_blocked`。
* 删除库市场中的插件会返回 `400 cmek_key_disabled` 且不会删除任何内容，因为必须先从 claude.ai 撤回其技能，而这需要该密钥。其他删除操作、安装设置和市场默认值均可正常使用。

恢复密钥后，上述所有限制都会解除。

## 另请参阅

<CardGroup cols={2}>
  <Card title="创建 Admin API 密钥" href="https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys">
    您的主要所有者在此创建具有作用域的密钥。
  </Card>

  <Card title="用户管理" href="https://platform.claude.com/docs/zh-CN/manage-claude/user-management">
    提供安装设置中所用 `rbac_group_` ID 的组端点。
  </Card>

  <Card title="Compliance API 活动源" href="https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed">
    记录插件写入和成员归档文件下载的位置。
  </Card>

  <Card title="分析 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api">
    Claude Enterprise 的插件和技能使用情况报告。
  </Card>
</CardGroup>
