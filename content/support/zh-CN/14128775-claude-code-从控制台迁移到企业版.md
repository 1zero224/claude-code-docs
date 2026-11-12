# Claude Code 从控制台迁移到企业版

## 概述

本指南适用于从基于控制台的 Claude Code 访问迁移到 Claude 企业版的团队。请注意，个人用户无需迁移 CLI 会话历史记录，因为这些记录存储在本地。您只需为每个用户配置 Claude 企业版账户，然后用户应将其登录方法从控制台切换到 Claude 企业版。

## 为什么要迁移到 Claude 企业版

控制台 API 上的 Claude Code 是让开发人员快速入门的方式，但它将治理权留给每台机器。Claude 企业版将开发人员已在使用的相同 Claude Code 置于安全、IT 和财务团队需要的控制措施之后，以便大规模运行。您可以通过现有 IdP 获得集中式身份和访问管理，通过服务器管理的设置在每个客户端上强制执行工具、文件和 MCP 策略而无需 MDM，以及从组织级别级联到个人用户的粒度支出上限。

Claude 企业版增加了丰富的分析和审计日志（包括贡献指标，如 PR 和提交的代码行数）、自定义数据保留控制，以及其他产品功能——Claude Code 云会话、Claude Mobile 和 Slack（Claude Tag），以及代码审查——这些在控制台上不可用。由于 Claude 企业版可通过 AWS Marketplace 获得，采购可以通过您已有的渠道进行。

---

## Claude 企业版与控制台的对比

| **功能**                                 | **Claude 控制台/API**                                                                                              | **Claude 企业版**                                                                                                                                                                                                                   |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code 云会话，包括来自 Claude Mobile 的会话 | ❌                                                                                                               | ✅                                                                                                                                                                                                                                |
| Claude Tag                             | ❌                                                                                                               | ✅                                                                                                                                                                                                                                |
| 代码审查                                   | ❌                                                                                                               | ✅                                                                                                                                                                                                                                |
| 强制执行 Claude Code 可以使用的工具               | ❌ 无原生 UI<br>*\*设置可以通过 MDM (Jamf, Intune) 或配置管理 (Ansible, Puppet 等) 推送到开发人员机器。*                                  | ✅ 原生 UI 用于设置 managed-settings.json 允许/拒绝规则                                                                                                                                                                                       |
| 阻止特定的 bash 命令                          | ❌ 无原生 UI\*                                                                                                      | ✅ Bash(curl:\*), Bash(sudo:\*) 等                                                                                                                                                                                                 |
| 防止访问敏感文件                               | ❌ 无原生 UI\*                                                                                                      | ✅ Read(.env), Read(./secrets/\*\*)                                                                                                                                                                                               |
| 控制允许的 MCP 服务器                          | ❌ 无原生 UI\*                                                                                                      | ✅ allowedMcpServers / deniedMcpServers                                                                                                                                                                                           |
| 在整个组织范围内部署预先批准的 MCP 服务器                | ❌ 无原生 UI\*                                                                                                      | ✅ managed-mcp.json                                                                                                                                                                                                               |
| 强制沙箱模式                                 | ❌ 无原生 UI\*                                                                                                      | ✅ sandbox.enabled                                                                                                                                                                                                                |
| 禁用 --dangerously-skip-permissions      | ❌ 无原生 UI\*                                                                                                      | ✅ permissions.disableBypassPermissionsMode: disable                                                                                                                                                                              |
| 禁用自动模式                                 | ❌ 无原生 UI\*                                                                                                      | ✅ disableAutoMode: disable                                                                                                                                                                                                       |
| 自定义角色 (RBAC)                           | ❌                                                                                                               | ✅ 按组限制功能访问，并委派特定的管理区域（如计费、用户管理和身份管理），无需授予所有者角色。**[在企业计划上管理自定义角色](https://support.claude.com/en/articles/13930452-manage-custom-roles-on-enterprise-plans)**                                                                      |
| 模型治理                                   | ❌ 无原生 UI\*                                                                                                      | ✅ 从管理控制台为聊天和 Cowork 设置组织默认模型，并通过托管设置中的 `availableModels` 限制 Claude Code 模型选择。**[为您的组织设置默认模型](https://support.claude.com/en/articles/15330088-set-a-default-model-for-your-organization)**                                        |
| 审计日志                                   | ❌ 不支持审计日志<br>🟡 Claude Code 支持 **[OpenTelemetry](https://code.claude.com/docs/en/monitoring-usage)**             | 🟡 审计日志和 **[合规 API](https://support.claude.com/en/articles/13015708-access-the-compliance-api)**（包括审计日志事件）。企业计划可通过合规 API 获取本地 CLI 和桌面会话的记录。                                                                                       |
| 使用分析                                   | ✅ 代码行数、接受率、日活跃用户、日支出。<br>**[文档](https://code.claude.com/docs/en/analytics#access-analytics-for-api-customers)** | ✅ 在**组织设置**→**使用情况**中查看代码行数、接受率、日活跃用户和按成员计算的本月累计支出。<br>**[文档](https://code.claude.com/docs/en/analytics#access-analytics-for-teams-and-enterprise)**                                                                             |
| 程序化使用情况和成本报告                           | ✅ **[Claude Code 分析 API](https://platform.claude.com/docs/en/manage-claude/claude-code-analytics-api)**         | ✅ **[Claude 企业分析 API](https://platform.claude.com/docs/en/manage-claude/analytics-api)** 返回按用户的参与度和 Claude Code 指标（提交、拉取请求、代码行数）以及使用情况和成本端点。**[管理 API](https://platform.claude.com/docs/en/manage-claude/admin-api)** 涵盖程序化组织管理。 |
| 贡献指标                                   | ✅ 通过 Claude Code 分析 API                                                                                         | ✅ 使用 Claude Code 协助创建的拉取请求和提交的代码行数。                                                                                                                                                                                              |
| 细粒度支出控制                                | ✅ 组织和工作区限制，以及 Claude Code 工作区中的每个开发者限制                                                                          | ✅ 组织 → 组 → 个人，与 RBAC 组集成                                                                                                                                                                                                         |

**文档：[角色和权限](https://support.claude.com/en/articles/9267276-roles-and-permissions)**、**[在企业计划上购买和管理席位](https://support.claude.com/en/articles/13393991-purchasing-and-managing-seats-on-enterprise-plans)、[我的企业计划如何计费？](https://support.claude.com/en/articles/11526368-how-am-i-billed-for-my-enterprise-plan)**、**[在企业计划中使用 Claude Code](https://support.claude.com/en/articles/11845131-using-claude-code-with-your-team-or-enterprise-plan)**

---

## SCIM 配置

以下是您的身份团队需要执行的操作：

1. **更新 IdP 组映射** — 创建或重新利用 IdP 组以将用户映射到 Claude 企业组织。启用组映射以自动分配席位。

2. **确保购买了足够的席位** — 在触发 SCIM 同步之前，验证 Claude 企业组织有足够的可用席位。没有可用席位的用户将被设置为"未分配"状态。

3. **确认已启用 Claude Code 访问权限** — 确认在管理设置中为企业组织启用了 Claude Code，并且 — 如果您使用自定义角色或组来限定功能范围 — 您的开发者组被授予了 Claude Code 访问权限。

4. **触发同步** — 导航到**[组织设置 → 组织和访问权限](https://claude.ai/admin-settings/organization)**，找到**目录同步 (SCIM)**，然后点击"同步"。请注意，这可能会很慢，因为这可能会触发大型同步。

5. **验证席位分配** — 同步后，检查**[组织设置 → 组织和访问权限](https://claude.ai/admin-settings/organization)**以确认用户分配到了正确的席位类型。

6. **可选：调整控制台组织映射** — 如果某些用户应该保留控制台 API 访问权限，请将其组保持映射到控制台组织。用户可以同时属于两个组织。

**注意：**Microsoft Entra SCIM 更改每约 40 分钟同步一次。在组更改后使用"同步"按钮触发按需同步。

如果您的企业组织的 SSO 配置为仅登录，登录不会创建账户 — 用户必须手动邀请。在指导开发者登录之前启用 JIT 或 SCIM 配置。请参阅**[设置 JIT 或 SCIM 配置](https://support.claude.com/en/articles/13133195-set-up-jit-or-scim-provisioning)**。

---

## 支出限制

Claude 企业计划提供分层支出控制系统。限制级联 — 用户永远不能超过组织限制。

| **级别** | **范围**     | **谁设置**     | **它控制什么**        |
| ------ | ---------- | ----------- | ---------------- |
| 1. 组织  | 整个企业组织     | 主要所有者 / 所有者 | 所有席位和使用情况的最大月度支出 |
| 2. 组控制 | 具有 RBAC 的组 | 主要所有者 / 所有者 | 使用 RBAC 的组的支出限制  |
| 3. 个人  | 特定用户       | 主要所有者 / 所有者 | 单个团队成员的支出限制      |

### 如何设置和编辑支出限制

1. 以所有者或主要所有者身份登录。

2. 导航至 **[组织设置 → 使用情况](https://claude.ai/admin-settings/usage)**。

3. 设置组织级别限制 — 这是所有月度支出的全局上限。

4. 设置组级别限制 — 在

5. 设置个人级别限制 — 在"按成员"选项卡下的 **支出默认值** 部分中查找特定用户。

所有者可以将限制设置为"无限制"，但所有消费仍会被计费。如果消费座位上的用户达到其限制，他们将无法使用 Claude 或 Claude Code，直到下一个计费周期或管理员提高其限制。

**文档：[配置支出限制](https://support.claude.com/en/articles/11526368-how-am-i-billed-for-my-enterprise-plan)**、**[在企业计划中管理组和组支出限制](https://support.claude.com/en/articles/13799932-manage-groups-and-group-spend-limits-on-enterprise-plans)**

---

## 发放新座位和重新身份验证

### 添加座位

1. 以主所有者或所有者身份登录。

2. 转到 **[组织设置 → 账单](https://claude.ai/admin-settings/billing)**。

3. 单击"座位"下的铅笔图标。

4. 输入新的座位数。

5. 审查并单击"升级"以确认。新座位按比例计费。

### 将用户分配到座位

1. 转到 **[组织设置 → 成员](https://claude.ai/admin-settings/members)**。

2. 单击"添加成员"（或"批量添加"以添加多个）。

3. 输入用户的 @ 电子邮件地址。

4. 设置角色（用户、管理员、所有者）并发送邀请。

对于 SCIM 配置的用户，座位分配会根据组映射自动进行。如果未配置组映射，用户将默认使用最高可用座位层级。

### Claude Code 用户的重新身份验证

当前针对 Console 组织进行身份验证的开发人员需要针对 Claude 企业组织重新进行身份验证：

1. 登录前删除任何剩余的 Console 凭据。检查 shell 配置文件、dotfiles 和 CI 配置中的 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 环境变量，并删除任何 `apiKeyHelper` 设置。还要检查 `CLAUDE_CODE_OAUTH_TOKEN`、`ANTHROPIC_BASE_URL` 和 `CLAUDE_CODE_USE_BEDROCK / _VERTEX / _FOUNDRY` 标志 — 这些也会覆盖或绕过 `/login`。凭据也可能隐藏在 Claude Code 自己的设置文件的 env 块中（~/.claude/settings.json 和存储库中的 .claude/settings*.json 文件）。如果这些仍然存在，它们会默认优先于新登录：`/login` 看起来会成功，但使用情况继续计费到旧的 Console 组织。

2. 在终端中，运行 `claude`，然后运行 `/login` 以切换身份验证方法。

3. 选择"带订阅的 Claude 账户"作为登录方法。

4. 选择 Claude 企业组织（不是 Console 组织或个人账户）。

5. 授权，返回终端，并运行 `/status` 以确认 Claude Code 显示您的企业组织。

**CI 和自动化**

管道和脚本不使用 `/login`。要么保留 Console 组织（及其 API 密钥）用于自动化并仅迁移交互式开发人员座位，要么将 CI 切换到企业凭据：在登录企业组织时运行 `claude setup-token`，并将打印的令牌设置为 CI 环境中的 `CLAUDE_CODE_OAUTH_TOKEN`。在其中一个就位之前，不要删除 CI 凭据。

**IT 提示：**部署托管设置，将 `forceLoginOrgUUID` 设置为您的企业组织 UUID，作为每次迁移的标准部分。将 pin 作为端点管理的文件或 MDM 策略提供 — 服务器管理的设置仅在用户登录后才会到达，因此无法捕获错误的首次登录。如果您也使用服务器管理的设置，也在那里设置 `forceLoginOrgUUID`：这两个通道不会合并。这会阻止登录到任何其他组织，并将上述剩余凭据故障模式从无声计费错误转变为明显的错误：Claude Code 拒绝启动并告诉用户必须删除剩余凭据（`ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN` 或 `apiKeyHelper`）。

**验证迁移后**

从 Console 组织中删除已迁移的开发人员（或轮换其密钥）。删除会撤销其 Console 登录令牌并禁用其 Claude Code 工作区密钥，停止来自交互式使用的进一步 Console 计费。他们在其他 Console 工作区中创建的 API 密钥不会因删除而被禁用 — 单独审查并禁用这些密钥。在确认企业访问有效之前，不要删除成员；没有自动回滚。

**文档：[在企业计划中购买和管理座位](https://support.claude.com/en/articles/13393991-purchasing-and-managing-seats-on-enterprise-plans)**

---

## 报告改进

从 Console API 迁移到 Claude 企业可解锁 Claude Code 使用情况的更丰富分析：

| **指标**                  | **Console API**                | **Claude 企业**                                                                                                                                  |
| ----------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 令牌消耗                    | ✅                              | ✅                                                                                                                                              |
| 接受的代码行                  | ✅                              | ✅                                                                                                                                              |
| 建议接受率                   | ✅                              | ✅                                                                                                                                              |
| 使用 Claude Code 协助创建的 PR | ✅ 通过 Claude Code 分析 API        | ✅                                                                                                                                              |
| 使用 Claude Code 协助提交的代码行 | ✅ 通过 Claude Code Analytics API | ✅                                                                                                                                              |
| 支出限额通知                  | ✅ 在达到限额时发送电子邮件通知，支持配置收件人       | ✅<br>阈值警报                                                                                                                                      |
| 每个成员的本月累计支出             | ❌                              | ✅<br>管理员设置 → 使用情况                                                                                                                              |
| 合规审计跟踪                  | ❌                              | ✅ **[合规 API](https://support.claude.com/en/articles/13015708-access-the-compliance-api)** 包含审计日志事件。本地 CLI 和 Desktop 会话的记录可通过合规 API 获取，适用于企业计划。 |

所有报告都可从 Claude 管理面板中的 **[分析](https://claude.ai/analytics/activity)** 访问。如需以编程方式访问，**[Claude 企业分析 API](https://platform.claude.com/docs/en/manage-claude/analytics-api)** 可返回每个用户的参与度指标、Claude Code 活动（提交、拉取请求、代码行数）以及使用情况和成本数据。分析不会迁移：企业组织从全新的报告历史记录开始，其分析/管理 API 需要在企业组织中创建新的 API 密钥——Console 密钥不会转移。在切换前，请导出您需要的任何 Console 分析数据以用于历史仪表板。在 Console API 密钥上保留的流量（例如 CI）将继续仅在 Console 报告中显示。

**文档：[Claude Code 使用分析](https://support.claude.com/en/articles/12157520-claude-code-usage-analytics)**

---

## 托管策略设置——深入探讨

这是安全和 IT 团队最重要的升级。**[服务器托管设置](https://code.claude.com/docs/en/server-managed-settings)** 允许对每个开发者机器上 Claude Code 的功能进行集中的、可强制执行的控制。有两种方式可以实现，您不需要 MDM 来使用第一种方式。

### 工作原理

Claude Code 从设置源的层次结构中读取配置。managed-settings.json 文件位于最顶层，无法被用户或项目设置覆盖：

| **优先级** | **来源**                      | **范围**         | **谁控制它**                                               |
| ------- | --------------------------- | -------------- | ------------------------------------------------------ |
| 1（最高）   | 托管设置（服务器托管或端点托管）            | 企业范围           | 所有者/主要所有者（从 claude.ai 管理控制台推送时）或您的 IT/MDM 团队（作为文件部署时）。 |
| 2       | 命令行参数                       | 会话             | 开发者                                                    |
| 3       | .claude/settings.local.json | 项目（个人）         | 开发者                                                    |
| 4       | .claude/settings.json       | 项目（共享，在 Git 中） | 团队                                                     |
| 5（最低）   | ~/.claude/settings.json     | 用户（全局）         | 开发者                                                    |

### 两种方式来实现托管设置

**服务器托管设置（无需 MDM）：**所有者或主要所有者在管理控制台的 **组织设置** → **Claude Code** → **托管设置** 中定义设置。每个登录到该组织的 Claude Code 客户端在启动时自动获取这些设置，并每小时轮询一次更新。这是适合没有设备管理基础设施或用户在非托管设备上的组织的选择。了解更多关于 **[服务器托管设置](https://code.claude.com/docs/en/server-managed-settings)**。

**端点托管设置：**IT 直接将设置部署到设备，可以通过本机操作系统策略（macOS 托管偏好设置或 Windows 注册表，通过 Jamf、Intune、Group Policy 等）或作为推送到以下系统路径的 managed-settings.json 文件。使用操作系统级权限保护该文件，以防最终用户修改。在已注册的设备上，这比服务器托管交付提供更强的保证，因为操作系统可防止用户篡改。

**注意：**两个通道占据相同的最高优先级层，使用相同的 JSON 格式，但它们不会合并。第一个提供非空配置的来源获胜：首先检查服务器托管设置，如果它们提供任何密钥，端点托管设置将被完全忽略。运行 /status 查看哪个托管来源处于活动状态。

### 文件位置（端点托管）

| **操作系统** | **路径**                                                        |
| -------- | ------------------------------------------------------------- |
| macOS    | /Library/Application Support/ClaudeCode/managed-settings.json |
| Linux    | /etc/claude-code/managed-settings.json                        |
| Windows  | C:\Program Files\ClaudeCode\managed-settings.json             |

### 关键设置参考

这些是与控制台到企业迁移最相关的设置。Claude Code 支持更多设置：如需完整的当前列表，请参阅 **[Claude Code 设置](https://code.claude.com/docs/en/settings)**。

| **设置**                                     | **用途**                                                                                       | **示例**                         |
| ------------------------------------------ | -------------------------------------------------------------------------------------------- | ------------------------------ |
| `permissions.deny`                         | 在整个组织范围内阻止特定工具/命令                                                                            | Bash(curl:\*), Read(.env)      |
| `permissions.allow`                        | 明确允许受信任的命令                                                                                   | Bash(npm run test:\*)          |
| `permissions.ask`                          | 每次都需要用户批准                                                                                    | Bash(rm:\*)                    |
| `permissions.disableBypassPermissionsMode` | 防止 --dangerously-skip-permissions                                                            | "disable"                      |
| `forceLoginOrgUUID`                        | 限制登录到特定企业组织。接受一个组织 UUID 或数组；阻止登录到任何其他组织。单个 UUID 也会在登录时自动选择该组织。                               | "298e7cb2…"                    |
| `forceLoginMethod`                         | 强制登录方法（claudeai、console 或 gateway）。建议与 `forceLoginOrgUUID` 一起使用，以便用户跳过账户类型选择器，但组织限制的应用不需要此项。 | "claudeai"                     |
| `allowedMcpServers`                        | 仅允许已批准的 MCP 服务器，按名称、命令或 URL 模式匹配                                                             | [{"serverName": "github"}]     |
| `deniedMcpServers`                         | 黑名单特定 MCP 服务器                                                                                | [{"serverName": "filesystem"}] |
| `allowManagedPermissionRulesOnly`          | 仅应用托管设置中的权限规则；用户和项目无法添加自己的规则                                                                 | true                           |
| `allowManagedMcpServersOnly`               | 仅应用托管 MCP 允许列表；拒绝列表仍从所有来源合并                                                                  | true                           |
| `allowManagedHooksOnly`                    | 仅运行托管和管理员批准的钩子                                                                               | true                           |
| `enforceAvailableModels`                   | 将 `availableModels` 允许列表扩展到默认模型选项                                                            | true                           |
| `sandbox.enabled`                          | 强制沙箱模式                                                                                       | true                           |
| `sandbox.allowUnsandboxedCommands`         | 阻止未沙箱化执行                                                                                     | false                          |
| `cleanupPeriodDays`                        | 本地会话数据（记录和其他应用程序文件）的保留期。默认值：30 天                                                             | 7                              |
| `companyAnnouncements`                     | 向所有 Claude Code 用户显示消息                                                                       | 字符串数组                          |

**部署：**使用您的 MDM 工具（Intune、Jamf、SCCM、Puppet 等）将 managed-settings.json 推送到所有开发者机器。使用操作系统级权限保护该文件，以防最终用户修改。

**文档：[Claude Code 设置](https://code.claude.com/docs/en/settings)**、**[托管设置](https://code.claude.com/docs/en/permissions#managed-settings)**、**[MCP 安装范围](https://code.claude.com/docs/en/mcp#managed-mcp-configuration)**