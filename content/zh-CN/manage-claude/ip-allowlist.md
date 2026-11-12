---
title: IP 允许列表
url: https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist
description: 通过将访问限制为您信任的特定 IP 地址来保护您的组织。
featureMetadata:
  status: beta
---

“IP allowlisting”（IP 允许列表）通过仅允许[受覆盖请求](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#where-the-allowlist-applies)来自特定 IP 范围来保护您的组织。这意味着，如果 API 密钥泄露，攻击者将更难使用该密钥。这可以防止数据泄露，并降低因滥用而被收费的可能性。

允许列表限制的是密钥可以在哪里使用，但不限制可以使用多长时间。您还应设置较短的[密钥过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)，或改用[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)以停止使用静态密钥。这比单独使用允许列表更安全。

<Note>
  IP 允许列表目前以 beta 形式提供，受您与 Anthropic 之间协议的约束，例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[服务特定条款](https://www.anthropic.com/legal/service-specific-terms)。在 beta 期间，设置和行为可能会发生变化。
</Note>

## 工作原理

组织管理员或所有者在设置中设置 IP 范围列表。此范围列表涵盖您的整个组织。对于每个受覆盖的请求，Anthropic 会将请求的公共 IP 地址与您的列表进行比较，如果该地址不被允许，则会拒绝该请求。

* 从列表之外的地址向[受覆盖端点](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#where-the-allowlist-applies)发出的 **API 请求**会被拒绝。例如，Messages API 会返回 403 `permission_error`，消息为 `Your IP address isn't in this organization's allowed range.`
* 从不被允许的地址连接的**人员**会看到一个标题为 **Your current network can’t access this organization** 的页面。组织管理员也会被阻止，但您可以选择[为管理员保留一条返回途径](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#avoid-locking-yourself-out)以避免被锁在外面，在这种情况下，他们只能访问允许列表设置。

## Claude Platform 组织

### 允许列表的适用范围

* **Claude Console：**&#x55;I 访问受到限制。
* **以下 Claude API 端点：**&#x4F7F;用 API 密钥或工作负载身份联合令牌访问的 [Messages](https://platform.claude.com/docs/zh-CN/api/messages)、[Message Batches](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)、[Token Counting](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)、[Models](https://platform.claude.com/docs/zh-CN/api/models/list)、[Files](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 和 [Skills](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide)。
* **以下 Claude Managed Agents 端点：**&#x4F7F;用 API 密钥或工作负载身份联合令牌访问的 [Agents](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)、[会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)、[环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments)、[Vaults](https://platform.claude.com/docs/zh-CN/managed-agents/vaults) 和 [Memory stores](https://platform.claude.com/docs/zh-CN/managed-agents/memory)。
* **以下组织 API：**&#x4F7F;用[仅覆盖此组织的密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#which-key-do-you-need)访问的 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 和 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)。

另请参阅[限制](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#limitations)。

## Claude Enterprise 组织

Claude Enterprise 组织有其自己的 IP 允许列表，仅允许从特定 IP 范围访问[其覆盖的应用](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#apps-that-the-allowlist-covers)。这意味着，如果用户的登录凭据被盗，攻击者将更难使用它。此列表涵盖您的整个组织，包括所有者。所有者和主要所有者可以设置该列表。在您[开启允许列表](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#turn-on-the-allowlist)之前，请阅读[避免把自己锁在外面](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#avoid-locking-yourself-out)。

### 允许列表覆盖的应用

列表之外的用户将无法在以下应用中访问您的组织：

* 浏览器中的 Claude、桌面应用和移动应用
* [Claude Code](https://code.claude.com/docs/zh-CN/overview)（当用户使用其 Claude 账户登录时）
* Claude in Chrome、[Office 加载项](https://claude.com/docs/office-agents/overview)、[Cowork](https://claude.com/docs/cowork/overview) 和 Remote Tools

这包括这些应用中的连接器和 artifacts，但用户在组织外部共享的 artifacts 除外。关于 Claude Science，请参阅 [Claude Science 管理控制中的身份和访问](https://claude.com/docs/claude-science/admin-controls#identity-and-access)。

允许列表还覆盖使用[仅覆盖此组织的密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#which-key-do-you-need)访问的 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 和 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)。

另请参阅[限制](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#limitations)。

### 用户可能遇到的情况

在网页和桌面应用中，被阻止的用户会看到一个标题为 **Your current network can’t access this organization** 的页面。所有者也会被阻止，但您可以选择[为所有者保留一条返回途径](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#avoid-locking-yourself-out)以避免被锁在外面，在这种情况下，他们只能访问允许列表设置。在 Claude Code 中，被阻止的用户会看到一个 403 API 错误，消息为 `Your IP address isn't in this organization's allowed range.`

* **其他账户：**&#x88AB;阻止的用户仍然可以登录并使用其他组织或个人账户。
* **远程用户：**&#x4F7F;用移动数据的手机、家庭 Wi-Fi 以及未连接 VPN 的笔记本电脑会被阻止，直到它们通过列表中的网络进行连接。
* **代理：**&#x90E8;分功能通过 `bridge.claudeusercontent.com` 进行连接：Claude in Chrome、Cowork、Remote Tools 以及 Office 加载项中的[跨应用工作](https://claude.com/docs/office-agents/work-across-apps)功能。如果您的代理通过与 `claude.ai` 和 `api.anthropic.com` 不同的地址发送该主机的流量，请同时列出该地址。

## 限制

* **Claude Platform：**&#x5141;许列表覆盖的是对 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 端点的请求，而不是智能体本身。它不适用于 Claude Managed Agents 运行时中的任何网络活动。允许列表不适用于 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、[Claude Models on Google Cloud](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/overview) 或 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)。
* **Claude Enterprise：**&#x43;laude Design、Claude in Slack 和 Claude Academy 不在覆盖范围内。如果用户被阻止时 Claude 正在聊天中处理任务，该任务最多可以继续运行 24 小时。用户无法从列表之外打开或控制该聊天。

## 开启允许列表

<Steps>
  <Step title="打开开关">
    前往您组织的设置页面：

    * **Claude Platform：**&#x43;laude Console 中的[身份与访问](https://platform.claude.com/settings/identity)页面
    * **Claude Enterprise：**[claude.ai > 组织设置 > 组织与访问权限](https://claude.ai/admin-settings/organization#ip-allowlist)

    打开 **IP allowlist** 开关。在您保存之前，不会有任何更改。
  </Step>

  <Step title="添加您的范围并保存">
    点击**添加条目**（Claude Console）或**添加 IP 范围**（claude.ai），输入一个或多个地址或范围（以逗号分隔），然后按 Enter。列出您的服务器、CI 和用户连接所用的每个网络，包括您当前连接所用的网络。Anthropic 建议确保已选中**允许管理员从任意 IP 地址访问此设置页面**（Claude Console）或**允许所有者从任意 IP 地址访问此设置页面**（claude.ai）复选框，以[避免被锁在外面](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#avoid-locking-yourself-out)。然后点击**保存**并确认。
  </Step>

  <Step title="检查您的流量">
    更改最多可能需要 15 分钟才能生效。留意您的应用程序是否出现 403 `permission_error` 响应，包括仅每天或每周运行的作业，并添加您遗漏的任何地址。在 Claude Enterprise 组织中，请每个网络上的用户确认他们仍然可以打开 Claude。在此类组织中，更改传达到已打开的连接、Office 加载项和 Remote Tools 可能需要更长时间。
  </Step>
</Steps>

### 条目格式

* **格式：**&#x60A8;可以将 IPv4 或 IPv6 地址加入允许列表。每个条目可以是单个地址（`203.0.113.5`）或 CIDR 范围（`203.0.113.0/24`、`2001:db8::/32`），IPv4 不得宽于 `/8`，IPv6 不得宽于 `/32`。列表最多可容纳 1,000 个条目。
* **仅限公共地址：**&#x5217;出您的流量离开您网络时所用的地址，而不是 `10.0.0.0/8` 等私有范围。如果您添加了共享代理地址，请注意，持有泄露密钥或被盗登录凭据的攻击者可能会通过该代理使用它。

## 避免把自己锁在外面

保存时，设置页面会将您当前的地址与列表进行比对，以确保您不会把自己锁在外面。但它不会检查您的服务器、您的队友或您明天可能用于连接的地址。

* **务必先列出每个网络：**&#x5305;括数据中心、CI 系统、办公室、VPN、代理，以及任何[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)的 worker，并附上它们的公共 IPv4 和 IPv6 地址。请注意，家庭和移动网络的地址经常会在不通知的情况下发生变化。
* **保留一条返回途径：**&#x41;nthropic 建议选中**允许管理员从任意 IP 地址访问此设置页面**（Claude Console）或**允许所有者从任意 IP 地址访问此设置页面**（claude.ai）复选框。如此一来，管理员（在 claude.ai 中为所有者）可以从任何地方修复列表。这样做的缺点是，如果管理员或所有者的会话遭到入侵，攻击者可能会从远程位置更新允许列表。如果您为组织[设置了 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)，其[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)会记录对允许列表的更改，以便您进行审计。

### 如果您被锁在外面

在 Claude Enterprise 组织中，管理员无法更改列表，因此以下步骤中的"管理员或所有者"是指所有者或主要所有者。

1. **从列表中的网络重新连接：**&#x8FDE;接到您的办公室网络或 VPN，然后点击 **Try again**。如果没有 **Try again** 按钮，请重新加载页面。
2. **请求其他管理员或所有者：**&#x4F4D;于列表中网络上的管理员或所有者可以添加您的地址。
3. **在当前位置修复列表：**&#x5982;果您是管理员或所有者，并且已选中[保留一条返回途径](https://platform.claude.com/docs/zh-CN/manage-claude/ip-allowlist#avoid-locking-yourself-out)中所述的复选框，请在浏览器中打开 Claude Console 或 Claude 并点击**打开 IP 允许列表**，或前往[身份与访问](https://platform.claude.com/settings/identity)页面（在 claude.ai 中为[组织与访问权限](https://claude.ai/admin-settings/organization#ip-allowlist)）。点击**添加**添加您当前的地址，或关闭允许列表，然后点击**保存**并确认。访问权限最多可能需要 15 分钟才能恢复。
4. **联系 Anthropic：**&#x5982;果没有管理员或所有者能够访问该页面，请联系 [Anthropic 支持](https://support.claude.com)关闭允许列表。
