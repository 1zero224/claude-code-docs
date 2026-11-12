---
title: Claude Platform on AWS 的 IAM 操作
url: https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions
description: 通过 AWS 策略控制对 Claude Platform on AWS 访问权限的 IAM 操作参考。
---

Claude Platform on AWS 使用 AWS IAM 进行访问控制。每个 API 路由都映射到 `aws-external-anthropic` 命名空间中的一个 IAM 操作。本页面列出了所有操作、每个操作所授权的路由，以及适用于常见访问模式的托管策略。有关平台设置和身份验证，请参阅 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)。

## 服务详情

| 属性           | 值                        |
| ------------ | ------------------------ |
| **IAM 服务前缀** | `aws-external-anthropic` |
| **资源类型**     | `workspace`              |

工作区 ARN 格式：

```text wrap
arn:aws:aws-external-anthropic:{region}:{account-id}:workspace/{workspace-id}
```

ARN 中的区域始终会被填充，并与工作区所绑定的区域一致。资源段是带标签的工作区 ID（`wrkspc_...`），与您在 `anthropic-workspace-id` 标头中传递的值相同。

## 操作

操作遵循 AWS 的 `VerbNoun` 命名约定，并严格规范动词的使用，使 `Get*` 和 `List*` “wildcard”（通配符）能够形成清晰的只读边界。

### 推理

| 操作                | 授权的路由                            |
| ----------------- | -------------------------------- |
| `CreateInference` | `POST /v1/messages`              |
| `CountTokens`     | `POST /v1/messages/count_tokens` |

### 批处理

| 操作                     | 授权的路由                                                                        |
| ---------------------- | ---------------------------------------------------------------------------- |
| `CreateBatchInference` | `POST /v1/messages/batches`                                                  |
| `GetBatchInference`    | `GET /v1/messages/batches/{id}`<br />`GET /v1/messages/batches/{id}/results` |
| `ListBatchInferences`  | `GET /v1/messages/batches`                                                   |
| `CancelBatchInference` | `POST /v1/messages/batches/{id}/cancel`                                      |
| `DeleteBatchInference` | `DELETE /v1/messages/batches/{id}`                                           |

<Note>
  `GetBatchInference` 同时授权读取批处理元数据和下载批处理结果。`AnthropicReadOnlyAccess`、`AnthropicInferenceAccess` 和 `AnthropicLimitedAccess` 策略的 `Get*` 通配符包含此操作。
</Note>

### 模型

| 操作           | 授权的路由                 |
| ------------ | --------------------- |
| `GetModel`   | `GET /v1/models/{id}` |
| `ListModels` | `GET /v1/models`      |

### 文件

| 操作           | 授权的路由                                                  |
| ------------ | ------------------------------------------------------ |
| `CreateFile` | `POST /v1/files`                                       |
| `GetFile`    | `GET /v1/files/{id}`<br />`GET /v1/files/{id}/content` |
| `ListFiles`  | `GET /v1/files`                                        |
| `DeleteFile` | `DELETE /v1/files/{id}`                                |

<Note>
  `GetFile` 同时授权元数据读取和内容下载。拥有只读访问权限的主体可以下载文件字节，而不仅仅是列出文件。
</Note>

### 技能

| 操作            | 授权的路由                                                                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CreateSkill` | `POST /v1/skills`                                                                                                                                             |
| `GetSkill`    | `GET /v1/skills/{id}`<br />`GET /v1/skills/{id}/versions`<br />`GET /v1/skills/{id}/versions/{version}`<br />`GET /v1/skills/{id}/versions/{version}/content` |
| `ListSkills`  | `GET /v1/skills`                                                                                                                                              |
| `UpdateSkill` | `POST /v1/skills/{id}/versions`<br />`DELETE /v1/skills/{id}/versions/{version}`                                                                              |
| `DeleteSkill` | `DELETE /v1/skills/{id}`                                                                                                                                      |

<Note>
  `GetSkill` 同时授权技能元数据读取和技能内容下载。拥有只读访问权限的主体可以下载技能字节，而不仅仅是列出技能。
</Note>

<Note>
  创建或删除单个技能版本映射到 `UpdateSkill`，而非 `CreateSkill` 或 `DeleteSkill`。拒绝 `aws-external-anthropic:Delete*` 的策略仍然允许删除版本，拒绝 `aws-external-anthropic:Create*` 的策略仍然允许创建版本。要阻止 `/v1/skills` 下的所有写入路由，请拒绝 `CreateSkill`、`UpdateSkill` 和 `DeleteSkill`。
</Note>

### 代理

| 操作             | 授权的路由                                                     |
| -------------- | --------------------------------------------------------- |
| `CreateAgent`  | `POST /v1/agents`                                         |
| `GetAgent`     | `GET /v1/agents/{id}`<br />`GET /v1/agents/{id}/versions` |
| `ListAgents`   | `GET /v1/agents`                                          |
| `UpdateAgent`  | `POST /v1/agents/{id}`                                    |
| `ArchiveAgent` | `POST /v1/agents/{id}/archive`                            |

<Note>
  代理仅支持归档，不支持硬删除。拒绝 `aws-external-anthropic:Delete*` 的策略不会阻止 `ArchiveAgent`。要阻止 `/v1/agents` 下的所有写入路由，请拒绝 `CreateAgent`、`UpdateAgent` 和 `ArchiveAgent`。
</Note>

### 会话

| 操作               | 授权的路由                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CreateSession`  | `POST /v1/sessions`                                                                                                                                                                                                                                                                                                                                                        |
| `GetSession`     | `GET /v1/sessions/{id}`<br />`GET /v1/sessions/{id}/events`<br />`GET /v1/sessions/{id}/events/stream`<br />`GET /v1/sessions/{id}/resources`<br />`GET /v1/sessions/{id}/resources/{id}`<br />`GET /v1/sessions/{id}/threads`<br />`GET /v1/sessions/{id}/threads/{id}`<br />`GET /v1/sessions/{id}/threads/{id}/events`<br />`GET /v1/sessions/{id}/threads/{id}/stream` |
| `ListSessions`   | `GET /v1/sessions`                                                                                                                                                                                                                                                                                                                                                         |
| `UpdateSession`  | `POST /v1/sessions/{id}`<br />`POST /v1/sessions/{id}/events`<br />`POST /v1/sessions/{id}/resources`<br />`POST /v1/sessions/{id}/resources/{id}`<br />`DELETE /v1/sessions/{id}/resources/{id}`<br />`POST /v1/sessions/{id}/threads/{id}/archive`                                                                                                                       |
| `ArchiveSession` | `POST /v1/sessions/{id}/archive`                                                                                                                                                                                                                                                                                                                                           |
| `DeleteSession`  | `DELETE /v1/sessions/{id}`                                                                                                                                                                                                                                                                                                                                                 |

<Note>
  `GetSession` 授权读取会话元数据、事件流（对话历史）、会话资源以及[会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)，包括每个线程的事件。`AnthropicReadOnlyAccess`、`AnthropicInferenceAccess` 和 `AnthropicLimitedAccess` 策略的 `Get*` 通配符包含此操作，而 `AnthropicSelfHostedEnvironmentAccess` 策略则显式授予此操作。
</Note>

<Note>
  所有写入现有会话的事件、资源或线程的路由都映射到 `UpdateSession`，而非 `CreateSession`、`DeleteSession` 或 `ArchiveSession`。拒绝 `aws-external-anthropic:Create*`、`Delete*` 或 `Archive*` 的策略仍然允许发送事件、添加或删除会话资源以及归档会话线程。要阻止 `/v1/sessions` 下的所有写入路由，请拒绝 `CreateSession`、`UpdateSession`、`ArchiveSession` 和 `DeleteSession`。拥有 `ProcessEnvironmentWork` 的主体仍然可以[停止在自托管环境中运行的会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-operations#stop-a-session-gracefully)。
</Note>

### 环境

| 操作                       | 授权的路由                                                                                                                                                                                                                                                        |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `CreateEnvironment`      | `POST /v1/environments`                                                                                                                                                                                                                                      |
| `GetEnvironment`         | `GET /v1/environments/{id}`<br />`GET /v1/environments/{id}/work`<br />`GET /v1/environments/{id}/work/{work_id}`<br />`GET /v1/environments/{id}/work/stats`                                                                                                |
| `ListEnvironments`       | `GET /v1/environments`                                                                                                                                                                                                                                       |
| `UpdateEnvironment`      | `POST /v1/environments/{id}`                                                                                                                                                                                                                                 |
| `ArchiveEnvironment`     | `POST /v1/environments/{id}/archive`                                                                                                                                                                                                                         |
| `DeleteEnvironment`      | `DELETE /v1/environments/{id}`                                                                                                                                                                                                                               |
| `ProcessEnvironmentWork` | `GET /v1/environments/{id}/work/poll`<br />`POST /v1/environments/{id}/work/{work_id}`<br />`POST /v1/environments/{id}/work/{work_id}/ack`<br />`POST /v1/environments/{id}/work/{work_id}/heartbeat`<br />`POST /v1/environments/{id}/work/{work_id}/stop` |

<Note>
  拒绝 `aws-external-anthropic:Delete*` 的策略不会阻止 `ArchiveEnvironment`。`ProcessEnvironmentWork` 不会被 `Create*`、`Update*`、`Delete*` 或 `Archive*` 通配符匹配。要阻止 `/v1/environments` 下的所有写入路由，请拒绝 `CreateEnvironment`、`UpdateEnvironment`、`ArchiveEnvironment`、`DeleteEnvironment` 和 `ProcessEnvironmentWork`。
</Note>

<Note>
  `ProcessEnvironmentWork` 授权[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)工作进程对环境工作项进行轮询、确认、发送心跳、停止以及提交结果。请仅将其授予运行自托管环境工作进程的主体。`AnthropicSelfHostedEnvironmentAccess` 托管策略包含此操作。
</Note>

### 保管库

| 操作             | 授权的路由                                                                                                                                                                                                           |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CreateVault`  | `POST /v1/vaults`                                                                                                                                                                                               |
| `GetVault`     | `GET /v1/vaults/{id}`<br />`GET /v1/vaults/{id}/credentials`<br />`GET /v1/vaults/{id}/credentials/{id}`                                                                                                        |
| `ListVaults`   | `GET /v1/vaults`                                                                                                                                                                                                |
| `UpdateVault`  | `POST /v1/vaults/{id}`<br />`POST /v1/vaults/{id}/credentials`<br />`POST /v1/vaults/{id}/credentials/{id}`<br />`POST /v1/vaults/{id}/credentials/{id}/archive`<br />`DELETE /v1/vaults/{id}/credentials/{id}` |
| `ArchiveVault` | `POST /v1/vaults/{id}/archive`                                                                                                                                                                                  |
| `DeleteVault`  | `DELETE /v1/vaults/{id}`                                                                                                                                                                                        |

<Note>
  创建、更新、归档或删除单个保管库凭证映射到 `UpdateVault`。读取凭证映射到 `GetVault`。保管库凭证密钥不会被暴露：密钥字段为只写字段，`GetVault` 永远不会返回它们（请参阅[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)）。拒绝 `aws-external-anthropic:Create*`、`Delete*` 或 `Archive*` 的策略仍然允许创建、删除和归档凭证。要阻止 `/v1/vaults` 下的所有写入路由，请拒绝 `CreateVault`、`UpdateVault`、`ArchiveVault` 和 `DeleteVault`。
</Note>

### 记忆存储

| 操作                   | 授权的路由                                                                                                                                                                                                                                        |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CreateMemoryStore`  | `POST /v1/memory_stores`                                                                                                                                                                                                                     |
| `GetMemoryStore`     | `GET /v1/memory_stores/{id}`<br />`GET /v1/memory_stores/{id}/memories`<br />`GET /v1/memory_stores/{id}/memories/{id}`<br />`GET /v1/memory_stores/{id}/memory_versions`<br />`GET /v1/memory_stores/{id}/memory_versions/{id}`             |
| `ListMemoryStores`   | `GET /v1/memory_stores`                                                                                                                                                                                                                      |
| `UpdateMemoryStore`  | `POST /v1/memory_stores/{id}`<br />`POST /v1/memory_stores/{id}/memories`<br />`POST /v1/memory_stores/{id}/memories/{id}`<br />`DELETE /v1/memory_stores/{id}/memories/{id}`<br />`POST /v1/memory_stores/{id}/memory_versions/{id}/redact` |
| `ArchiveMemoryStore` | `POST /v1/memory_stores/{id}/archive`                                                                                                                                                                                                        |
| `DeleteMemoryStore`  | `DELETE /v1/memory_stores/{id}`                                                                                                                                                                                                              |

<Note>
  `GetMemoryStore` 授权读取存储元数据、所有记忆以及记忆版本历史。`AnthropicReadOnlyAccess`、`AnthropicInferenceAccess` 和 `AnthropicLimitedAccess` 策略的 `Get*` 通配符包含此操作。
</Note>

<Note>
  创建、更新或删除单个记忆以及编辑（redact）记忆版本均映射到 `UpdateMemoryStore`，而非 `CreateMemoryStore` 或 `DeleteMemoryStore`。拒绝 `aws-external-anthropic:Delete*` 的策略仍然允许删除单个记忆和编辑记忆版本，拒绝 `aws-external-anthropic:Create*` 的策略仍然允许创建单个记忆。要阻止 `/v1/memory_stores` 下的所有写入路由，请拒绝 `CreateMemoryStore`、`UpdateMemoryStore`、`ArchiveMemoryStore` 和 `DeleteMemoryStore`。拥有 `CreateSession` 的主体仍然可以使用默认的 `read_write` 访问权限[将存储附加到会话](https://platform.claude.com/docs/zh-CN/managed-agents/memory#attach-a-memory-store-to-a-session)。随后该会话的代理便可以写入该存储。
</Note>

### Webhook

| 操作                    | 授权的路由                                              |
| --------------------- | -------------------------------------------------- |
| `CreateWebhook`       | `POST /v1/webhooks`                                |
| `GetWebhook`          | `GET /v1/webhooks/{id}`                            |
| `ListWebhooks`        | `GET /v1/webhooks`                                 |
| `UpdateWebhook`       | `POST /v1/webhooks/{id}`                           |
| `DeleteWebhook`       | `DELETE /v1/webhooks/{id}`                         |
| `RotateWebhookSecret` | `POST /v1/webhooks/{id}/regenerate_signing_secret` |

<Note>
  Webhook 签名密钥为只写字段。`GetWebhook` 仅返回 webhook 元数据；不会返回签名密钥。
</Note>

<Note>
  `RotateWebhookSecret` 不会被 `aws-external-anthropic:Create*`、`Update*` 或 `Delete*` 通配符匹配。拒绝这些模式的策略仍然允许轮换密钥。要阻止 `/v1/webhooks` 下的所有写入路由，请拒绝 `CreateWebhook`、`UpdateWebhook`、`DeleteWebhook` 和 `RotateWebhookSecret`。
</Note>

### 用户配置文件

| 操作                               | 授权的路由                                        |
| -------------------------------- | -------------------------------------------- |
| `CreateUserProfile`              | `POST /v1/user_profiles`                     |
| `GetUserProfile`                 | `GET /v1/user_profiles/{id}`                 |
| `ListUserProfiles`               | `GET /v1/user_profiles`                      |
| `UpdateUserProfile`              | `POST /v1/user_profiles/{id}`                |
| `DeleteUserProfile`              | `DELETE /v1/user_profiles/{id}`              |
| `CreateUserProfileEnrollmentUrl` | `POST /v1/user_profiles/{id}/enrollment_url` |

<Note>
  与 `ListWorkspaces` 一样，用户配置文件操作是账户范围的。`Resource` 为工作区 ARN 的语句不会授予这些操作；请使用 `Resource: "*"`。
</Note>

<Warning>
  IAM 操作匹配不区分大小写。通配符 `aws-external-anthropic:*File` 匹配 `CreateFile`、`GetFile` 和 `DeleteFile`，但不匹配 `ListFiles`（其以 “files” 结尾，而非 “file”）。它还会过度匹配 `CreateUserProfile`、`GetUserProfile`、`UpdateUserProfile` 和 `DeleteUserProfile`，因为 “Profile” 以 “file” 结尾。如果您只想授予或拒绝 Files API 操作，请显式枚举它们（`CreateFile`、`GetFile`、`ListFiles`、`DeleteFile`），而不要使用 `*File` 后缀模式。
</Warning>

### 工作区

| 操作                 | 授权的路由                                            |
| ------------------ | ------------------------------------------------ |
| `CreateWorkspace`  | `POST /v1/organizations/workspaces`              |
| `GetWorkspace`     | `GET /v1/organizations/workspaces/{id}`          |
| `ListWorkspaces`   | `GET /v1/organizations/workspaces`               |
| `UpdateWorkspace`  | `POST /v1/organizations/workspaces/{id}`         |
| `ArchiveWorkspace` | `POST /v1/organizations/workspaces/{id}/archive` |

<Note>
  工作区仅支持归档，不支持硬删除。拒绝 `aws-external-anthropic:Delete*` 的策略不会阻止 `ArchiveWorkspace`。要阻止 `/v1/organizations/workspaces` 下的所有写入路由，请拒绝 `CreateWorkspace`、`UpdateWorkspace` 和 `ArchiveWorkspace`。
</Note>

### 加密密钥

| 操作            | 授权的路由                                         |
| ------------- | --------------------------------------------- |
| `RegisterKey` | `POST /v1/organizations/external_keys`        |
| `GetKey`      | `GET /v1/organizations/external_keys/{id}`    |
| `ListKeys`    | `GET /v1/organizations/external_keys`         |
| `UpdateKey`   | `POST /v1/organizations/external_keys/{id}`   |
| `DisableKey`  | `DELETE /v1/organizations/external_keys/{id}` |

<Note>
  这些操作管理您组织的[客户管理加密密钥（CMEK）](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws)注册，即记录哪些 AWS KMS 密钥 ARN 已被注册。它们不会在 AWS KMS 中创建、更改或禁用密钥。`DisableKey` 会移除一项注册，并且在任何工作区仍在使用该密钥时会被拒绝。`RegisterKey` 和 `DisableKey` 不会被 `Create*`、`Update*` 或 `Delete*` 通配符匹配。要阻止 `/v1/organizations/external_keys` 下的所有写入路由，请拒绝 `RegisterKey`、`UpdateKey` 和 `DisableKey`。在这些路由中，`{id}` 是经过 URL 编码的 KMS 密钥 ARN。将已注册的密钥附加到工作区属于工作区操作，由 `CreateWorkspace` 或 `UpdateWorkspace` 授权；附加密钥的主体还需要对该密钥拥有 `kms:DescribeKey`、`kms:Encrypt` 和 `kms:Decrypt` 权限（请参阅[先决条件](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws#prerequisites)）。外部密钥操作是账户范围的：在这些操作上指定工作区 ARN 不会产生任何效果；请使用 `Resource: "*"`。
</Note>

### 合规

| 操作                         | 授权的路由                           |
| -------------------------- | ------------------------------- |
| `ListComplianceActivities` | `GET /v1/compliance/activities` |

<Note>
  `ListComplianceActivities` 授权读取 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的[活动源（Activity Feed）](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)，即包含访问透明度事件的组织级审计日志。在 Compliance API [为您的组织启用](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)之前，该路由会返回错误；启用需通过您的 Anthropic 客户团队申请。`AnthropicReadOnlyAccess`、`AnthropicInferenceAccess` 和 `AnthropicLimitedAccess` 策略的 `List*` 通配符包含此操作。
</Note>

<Note>
  `ListComplianceActivities` 与 `ListWorkspaces` 一样是账户范围的。在此操作上指定工作区 ARN 不会产生任何效果；请使用 `Resource: "*"`。
</Note>

### 遥测目标

| 操作                                          | 授权的路由                                                                            |
| ------------------------------------------- | -------------------------------------------------------------------------------- |
| `CreateAccountTelemetryDestination`         | `POST /v1/organizations/telemetry_destinations`                                  |
| `GetAccountTelemetryDestination`            | `GET /v1/organizations/telemetry_destinations/{id}`                              |
| `ListAccountTelemetryDestinations`          | `GET /v1/organizations/telemetry_destinations`                                   |
| `UpdateAccountTelemetryDestination`         | `POST /v1/organizations/telemetry_destinations/{id}`                             |
| `DeleteAccountTelemetryDestination`         | `DELETE /v1/organizations/telemetry_destinations/{id}`                           |
| `TestAccountTelemetryDestination`           | `POST /v1/organizations/telemetry_destinations/{id}/test`                        |
| `ListAccountTelemetryDestinationWorkspaces` | `GET /v1/organizations/telemetry_destinations/{id}/workspaces`                   |
| `AddTelemetryDestinationWorkspace`          | `POST /v1/organizations/telemetry_destinations/{id}/workspaces/{workspace_id}`   |
| `RemoveTelemetryDestinationWorkspace`       | `DELETE /v1/organizations/telemetry_destinations/{id}/workspaces/{workspace_id}` |

<Note>
  除 `AddTelemetryDestinationWorkspace` 和 `RemoveTelemetryDestinationWorkspace` 之外，这些操作都是账户范围的，因此在这些操作上指定工作区 ARN 不会产生任何效果。请使用 `Resource: "*"`。一个目标可以导出组织在所有区域中每个工作区的遥测数据。请仅将 `CreateAccountTelemetryDestination` 和 `UpdateAccountTelemetryDestination` 授予您信任其访问全部这些遥测数据的主体。
</Note>

<Note>
  `AddTelemetryDestinationWorkspace` 和 `RemoveTelemetryDestinationWorkspace` 作用于路由中的工作区，并且仅适用于**工作区**选项为**所选**的目标。要更改**除所选外的全部**列表，请使用 `UpdateAccountTelemetryDestination`。拒绝 `aws-external-anthropic:Create*`、`Update*` 或 `Delete*` 的策略仍然允许添加和移除工作区。要阻止 `/v1/organizations/telemetry_destinations` 下的所有写入路由，请拒绝 `CreateAccountTelemetryDestination`、`UpdateAccountTelemetryDestination`、`DeleteAccountTelemetryDestination`、`TestAccountTelemetryDestination`、`AddTelemetryDestinationWorkspace` 和 `RemoveTelemetryDestinationWorkspace`。
</Note>

### 用量和成本

| 操作                 | 授权的路由                                   |
| ------------------ | --------------------------------------- |
| `ListAccountUsage` | `GET /v1/organizations/analytics/usage` |
| `ListAccountCost`  | `GET /v1/organizations/analytics/cost`  |

<Note>
  这些操作授权[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)。两者均为账户范围的操作。在这些操作上指定工作区 ARN 不会产生任何效果；请使用 `Resource: "*"`。
</Note>

### 身份验证

| 操作                    | 授权的路由 |
| --------------------- | ----- |
| `CallWithBearerToken` | （无）   |

`CallWithBearerToken` 是一项身份验证层权限，授权主体通过 API 密钥（bearer token，持有者令牌）而非 AWS SigV4 进行身份验证。它不映射到任何路由。请将其与您希望 API 密钥持有者能够执行的路由映射操作一起授予。

### 控制台访问

| 操作              | 授权的路由 |
| --------------- | ----- |
| `AssumeConsole` | （无）   |

`AssumeConsole` 授权主体通过 AWS Console 联合登录流程为 Claude Platform on AWS 工作区打开 Claude Console。它不映射到任何路由。请将其授予应当能够在 AWS Console 中 Claude Platform on AWS 服务页面上点击 **Open Claude Console** 的主体。Claude Console 角色（Admin 或 Developer）由您的 Anthropic 客户代表单独分配；它并非从主体的 IAM 权限派生而来。有关登录流程和角色说明，请参阅[使用 Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#using-the-claude-console)。

## 路由到操作映射

下表列出了 Claude Platform on AWS 上的每个路由以及调用它所需的 IAM 操作。每个 IAM 操作同时也授权使用 `anthropic-beta` 标头的请求；路由的 beta 变体不需要单独的 IAM 操作。CloudTrail 将每个操作分类为 Data 事件（高容量的数据平面操作）或 Management 事件（控制平面操作）。保管库和 webhook 操作被分类为 Management 事件，因为它们持有密钥（保管库凭证和 webhook 签名密钥），并受益于默认开启的审计日志记录。工作区、外部密钥、遥测目标和合规操作也被分类为 Management 事件，因为它们是组织范围的控制平面操作。所有其他操作，包括推理、批处理、模型、文件、技能、用户配置文件、用量和成本以及其余的 Claude Managed Agents 操作，均被分类为 Data 事件。

| 方法       | 路由                                                                        | IAM 操作                                      | CloudTrail 事件类型 |
| -------- | ------------------------------------------------------------------------- | ------------------------------------------- | --------------- |
| `POST`   | `/v1/messages`                                                            | `CreateInference`                           | Data            |
| `POST`   | `/v1/messages/count_tokens`                                               | `CountTokens`                               | Data            |
| `POST`   | `/v1/messages/batches`                                                    | `CreateBatchInference`                      | Data            |
| `GET`    | `/v1/messages/batches`                                                    | `ListBatchInferences`                       | Data            |
| `GET`    | `/v1/messages/batches/{id}`                                               | `GetBatchInference`                         | Data            |
| `GET`    | `/v1/messages/batches/{id}/results`                                       | `GetBatchInference`                         | Data            |
| `POST`   | `/v1/messages/batches/{id}/cancel`                                        | `CancelBatchInference`                      | Data            |
| `DELETE` | `/v1/messages/batches/{id}`                                               | `DeleteBatchInference`                      | Data            |
| `GET`    | `/v1/models`                                                              | `ListModels`                                | Data            |
| `GET`    | `/v1/models/{id}`                                                         | `GetModel`                                  | Data            |
| `POST`   | `/v1/files`                                                               | `CreateFile`                                | Data            |
| `GET`    | `/v1/files`                                                               | `ListFiles`                                 | Data            |
| `GET`    | `/v1/files/{id}`                                                          | `GetFile`                                   | Data            |
| `GET`    | `/v1/files/{id}/content`                                                  | `GetFile`                                   | Data            |
| `DELETE` | `/v1/files/{id}`                                                          | `DeleteFile`                                | Data            |
| `POST`   | `/v1/skills`                                                              | `CreateSkill`                               | Data            |
| `GET`    | `/v1/skills`                                                              | `ListSkills`                                | Data            |
| `GET`    | `/v1/skills/{id}`                                                         | `GetSkill`                                  | Data            |
| `DELETE` | `/v1/skills/{id}`                                                         | `DeleteSkill`                               | Data            |
| `POST`   | `/v1/skills/{id}/versions`                                                | `UpdateSkill`                               | Data            |
| `GET`    | `/v1/skills/{id}/versions`                                                | `GetSkill`                                  | Data            |
| `GET`    | `/v1/skills/{id}/versions/{version}`                                      | `GetSkill`                                  | Data            |
| `GET`    | `/v1/skills/{id}/versions/{version}/content`                              | `GetSkill`                                  | Data            |
| `DELETE` | `/v1/skills/{id}/versions/{version}`                                      | `UpdateSkill`                               | Data            |
| `POST`   | `/v1/user_profiles`                                                       | `CreateUserProfile`                         | Data            |
| `GET`    | `/v1/user_profiles`                                                       | `ListUserProfiles`                          | Data            |
| `GET`    | `/v1/user_profiles/{id}`                                                  | `GetUserProfile`                            | Data            |
| `POST`   | `/v1/user_profiles/{id}`                                                  | `UpdateUserProfile`                         | Data            |
| `DELETE` | `/v1/user_profiles/{id}`                                                  | `DeleteUserProfile`                         | Data            |
| `POST`   | `/v1/user_profiles/{id}/enrollment_url`                                   | `CreateUserProfileEnrollmentUrl`            | Data            |
| `POST`   | `/v1/organizations/workspaces`                                            | `CreateWorkspace`                           | Management      |
| `GET`    | `/v1/organizations/workspaces`                                            | `ListWorkspaces`                            | Management      |
| `GET`    | `/v1/organizations/workspaces/{id}`                                       | `GetWorkspace`                              | Management      |
| `POST`   | `/v1/organizations/workspaces/{id}`                                       | `UpdateWorkspace`                           | Management      |
| `POST`   | `/v1/organizations/workspaces/{id}/archive`                               | `ArchiveWorkspace`                          | Management      |
| `POST`   | `/v1/organizations/external_keys`                                         | `RegisterKey`                               | Management      |
| `GET`    | `/v1/organizations/external_keys`                                         | `ListKeys`                                  | Management      |
| `GET`    | `/v1/organizations/external_keys/{id}`                                    | `GetKey`                                    | Management      |
| `POST`   | `/v1/organizations/external_keys/{id}`                                    | `UpdateKey`                                 | Management      |
| `DELETE` | `/v1/organizations/external_keys/{id}`                                    | `DisableKey`                                | Management      |
| `POST`   | `/v1/organizations/telemetry_destinations`                                | `CreateAccountTelemetryDestination`         | Management      |
| `GET`    | `/v1/organizations/telemetry_destinations`                                | `ListAccountTelemetryDestinations`          | Management      |
| `GET`    | `/v1/organizations/telemetry_destinations/{id}`                           | `GetAccountTelemetryDestination`            | Management      |
| `POST`   | `/v1/organizations/telemetry_destinations/{id}`                           | `UpdateAccountTelemetryDestination`         | Management      |
| `DELETE` | `/v1/organizations/telemetry_destinations/{id}`                           | `DeleteAccountTelemetryDestination`         | Management      |
| `POST`   | `/v1/organizations/telemetry_destinations/{id}/test`                      | `TestAccountTelemetryDestination`           | Management      |
| `GET`    | `/v1/organizations/telemetry_destinations/{id}/workspaces`                | `ListAccountTelemetryDestinationWorkspaces` | Management      |
| `POST`   | `/v1/organizations/telemetry_destinations/{id}/workspaces/{workspace_id}` | `AddTelemetryDestinationWorkspace`          | Management      |
| `DELETE` | `/v1/organizations/telemetry_destinations/{id}/workspaces/{workspace_id}` | `RemoveTelemetryDestinationWorkspace`       | Management      |
| `GET`    | `/v1/compliance/activities`                                               | `ListComplianceActivities`                  | Management      |
| `POST`   | `/v1/agents`                                                              | `CreateAgent`                               | Data            |
| `GET`    | `/v1/agents`                                                              | `ListAgents`                                | Data            |
| `GET`    | `/v1/agents/{id}`                                                         | `GetAgent`                                  | Data            |
| `POST`   | `/v1/agents/{id}`                                                         | `UpdateAgent`                               | Data            |
| `POST`   | `/v1/agents/{id}/archive`                                                 | `ArchiveAgent`                              | Data            |
| `GET`    | `/v1/agents/{id}/versions`                                                | `GetAgent`                                  | Data            |
| `POST`   | `/v1/sessions`                                                            | `CreateSession`                             | Data            |
| `GET`    | `/v1/sessions`                                                            | `ListSessions`                              | Data            |
| `GET`    | `/v1/sessions/{id}`                                                       | `GetSession`                                | Data            |
| `POST`   | `/v1/sessions/{id}`                                                       | `UpdateSession`                             | Data            |
| `POST`   | `/v1/sessions/{id}/archive`                                               | `ArchiveSession`                            | Data            |
| `DELETE` | `/v1/sessions/{id}`                                                       | `DeleteSession`                             | Data            |
| `GET`    | `/v1/sessions/{id}/events`                                                | `GetSession`                                | Data            |
| `POST`   | `/v1/sessions/{id}/events`                                                | `UpdateSession`                             | Data            |
| `GET`    | `/v1/sessions/{id}/events/stream`                                         | `GetSession`                                | Data            |
| `GET`    | `/v1/sessions/{id}/resources`                                             | `GetSession`                                | Data            |
| `GET`    | `/v1/sessions/{id}/resources/{id}`                                        | `GetSession`                                | Data            |
| `POST`   | `/v1/sessions/{id}/resources`                                             | `UpdateSession`                             | Data            |
| `POST`   | `/v1/sessions/{id}/resources/{id}`                                        | `UpdateSession`                             | Data            |
| `DELETE` | `/v1/sessions/{id}/resources/{id}`                                        | `UpdateSession`                             | Data            |
| `GET`    | `/v1/sessions/{id}/threads`                                               | `GetSession`                                | Data            |
| `GET`    | `/v1/sessions/{id}/threads/{id}`                                          | `GetSession`                                | Data            |
| `GET`    | `/v1/sessions/{id}/threads/{id}/events`                                   | `GetSession`                                | Data            |
| `GET`    | `/v1/sessions/{id}/threads/{id}/stream`                                   | `GetSession`                                | Data            |
| `POST`   | `/v1/sessions/{id}/threads/{id}/archive`                                  | `UpdateSession`                             | Data            |
| `POST`   | `/v1/environments`                                                        | `CreateEnvironment`                         | Data            |
| `GET`    | `/v1/environments`                                                        | `ListEnvironments`                          | Data            |
| `GET`    | `/v1/environments/{id}`                                                   | `GetEnvironment`                            | Data            |
| `POST`   | `/v1/environments/{id}`                                                   | `UpdateEnvironment`                         | Data            |
| `POST`   | `/v1/environments/{id}/archive`                                           | `ArchiveEnvironment`                        | Data            |
| `DELETE` | `/v1/environments/{id}`                                                   | `DeleteEnvironment`                         | Data            |
| `GET`    | `/v1/environments/{id}/work`                                              | `GetEnvironment`                            | Data            |
| `GET`    | `/v1/environments/{id}/work/poll`                                         | `ProcessEnvironmentWork`                    | Data            |
| `GET`    | `/v1/environments/{id}/work/{work_id}`                                    | `GetEnvironment`                            | Data            |
| `GET`    | `/v1/environments/{id}/work/stats`                                        | `GetEnvironment`                            | Data            |
| `POST`   | `/v1/environments/{id}/work/{work_id}`                                    | `ProcessEnvironmentWork`                    | Data            |
| `POST`   | `/v1/environments/{id}/work/{work_id}/ack`                                | `ProcessEnvironmentWork`                    | Data            |
| `POST`   | `/v1/environments/{id}/work/{work_id}/heartbeat`                          | `ProcessEnvironmentWork`                    | Data            |
| `POST`   | `/v1/environments/{id}/work/{work_id}/stop`                               | `ProcessEnvironmentWork`                    | Data            |
| `POST`   | `/v1/vaults`                                                              | `CreateVault`                               | Management      |
| `GET`    | `/v1/vaults`                                                              | `ListVaults`                                | Management      |
| `GET`    | `/v1/vaults/{id}`                                                         | `GetVault`                                  | Management      |
| `POST`   | `/v1/vaults/{id}`                                                         | `UpdateVault`                               | Management      |
| `POST`   | `/v1/vaults/{id}/archive`                                                 | `ArchiveVault`                              | Management      |
| `DELETE` | `/v1/vaults/{id}`                                                         | `DeleteVault`                               | Management      |
| `GET`    | `/v1/vaults/{id}/credentials`                                             | `GetVault`                                  | Management      |
| `POST`   | `/v1/vaults/{id}/credentials`                                             | `UpdateVault`                               | Management      |
| `GET`    | `/v1/vaults/{id}/credentials/{id}`                                        | `GetVault`                                  | Management      |
| `POST`   | `/v1/vaults/{id}/credentials/{id}`                                        | `UpdateVault`                               | Management      |
| `POST`   | `/v1/vaults/{id}/credentials/{id}/archive`                                | `UpdateVault`                               | Management      |
| `DELETE` | `/v1/vaults/{id}/credentials/{id}`                                        | `UpdateVault`                               | Management      |
| `POST`   | `/v1/memory_stores`                                                       | `CreateMemoryStore`                         | Data            |
| `GET`    | `/v1/memory_stores`                                                       | `ListMemoryStores`                          | Data            |
| `GET`    | `/v1/memory_stores/{id}`                                                  | `GetMemoryStore`                            | Data            |
| `POST`   | `/v1/memory_stores/{id}`                                                  | `UpdateMemoryStore`                         | Data            |
| `POST`   | `/v1/memory_stores/{id}/archive`                                          | `ArchiveMemoryStore`                        | Data            |
| `DELETE` | `/v1/memory_stores/{id}`                                                  | `DeleteMemoryStore`                         | Data            |
| `POST`   | `/v1/memory_stores/{id}/memories`                                         | `UpdateMemoryStore`                         | Data            |
| `GET`    | `/v1/memory_stores/{id}/memories`                                         | `GetMemoryStore`                            | Data            |
| `GET`    | `/v1/memory_stores/{id}/memories/{id}`                                    | `GetMemoryStore`                            | Data            |
| `POST`   | `/v1/memory_stores/{id}/memories/{id}`                                    | `UpdateMemoryStore`                         | Data            |
| `DELETE` | `/v1/memory_stores/{id}/memories/{id}`                                    | `UpdateMemoryStore`                         | Data            |
| `GET`    | `/v1/memory_stores/{id}/memory_versions`                                  | `GetMemoryStore`                            | Data            |
| `GET`    | `/v1/memory_stores/{id}/memory_versions/{id}`                             | `GetMemoryStore`                            | Data            |
| `POST`   | `/v1/memory_stores/{id}/memory_versions/{id}/redact`                      | `UpdateMemoryStore`                         | Data            |
| `GET`    | `/v1/webhooks`                                                            | `ListWebhooks`                              | Management      |
| `GET`    | `/v1/webhooks/{id}`                                                       | `GetWebhook`                                | Management      |
| `POST`   | `/v1/webhooks`                                                            | `CreateWebhook`                             | Management      |
| `POST`   | `/v1/webhooks/{id}`                                                       | `UpdateWebhook`                             | Management      |
| `DELETE` | `/v1/webhooks/{id}`                                                       | `DeleteWebhook`                             | Management      |
| `POST`   | `/v1/webhooks/{id}/regenerate_signing_secret`                             | `RotateWebhookSecret`                       | Management      |

| 方法    | 路由                                  | IAM 操作             | CloudTrail 事件类型 |
| ----- | ----------------------------------- | ------------------ | --------------- |
| `GET` | `/v1/organizations/analytics/usage` | `ListAccountUsage` | Data            |
| `GET` | `/v1/organizations/analytics/cost`  | `ListAccountCost`  | Data            |

不在这些表中的路由在 Claude Platform on AWS 上不可用。网关默认拒绝此处未列出的任何路由。

<Note>
  工作区、外部密钥、遥测目标以及用量和成本分析 API 路由是 Claude Platform on AWS 上唯一可用的 Admin API 路由。您也可以在 AWS Console 中创建、更新或归档工作区，或者以 Admin 角色在 Claude Console 中进行这些操作。加密密钥也可以在 Claude Console 中注册和附加。
</Note>

## 托管策略

AWS 为 Claude Platform on AWS 提供五个托管策略。所有托管策略均应用于 `Resource: "*"`。

| 策略                                     | 授予                                                                                                                                        |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `AnthropicFullAccess`                  | `aws-external-anthropic:*`                                                                                                                |
| `AnthropicReadOnlyAccess`              | `Get*`、`List*`、`CallWithBearerToken`                                                                                                      |
| `AnthropicInferenceAccess`             | `Get*`、`List*`、`CreateInference`、`CreateBatchInference`、`CancelBatchInference`、`DeleteBatchInference`、`CountTokens`、`CallWithBearerToken` |
| `AnthropicLimitedAccess`               | 所有 `AnthropicInferenceAccess` 操作，加上所有 Claude Managed Agents 操作（代理、会话、环境、保管库、记忆存储、webhook 以及自托管环境工作）                                       |
| `AnthropicSelfHostedEnvironmentAccess` | `GetEnvironment`、`ProcessEnvironmentWork`、`GetSession`、`UpdateSession`、`GetSkill`、`CallWithBearerToken`                                   |

`AnthropicInferenceAccess` 是足以运行推理的最窄托管策略。它涵盖同步推理和批量推理，并通过 `Get*` 和 `List*` 通配符授予对命名空间中每个 API 资源的读取访问权限，包括 Claude Managed Agents（CMA）资源（代理、会话、环境、保管库、记忆存储和 webhook）。这包括通过 `GetFile` 下载文件内容（请参阅[文件](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#files)注释）、通过 `GetSkill` 下载技能内容（请参阅[技能](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#skills)注释）以及通过 `GetMemoryStore` 读取记忆内容。保管库凭证密钥和 webhook 签名密钥不会被暴露：这些字段为只写字段，`GetVault` 或 `GetWebhook` 永远不会返回它们（请参阅[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)）。`AnthropicInferenceAccess` 不授予文件创建或删除、技能管理、用户配置文件管理、工作区变更、加密密钥管理、遥测目标管理或任何 Claude Managed Agents 写入操作（创建、更新、归档、删除、处理或轮换）。若要排除 CMA 读取权限，请将 `AnthropicInferenceAccess` 替换为仅枚举您所需的特定非 CMA 操作的自定义策略。

<Note>
  `AnthropicReadOnlyAccess`、`AnthropicInferenceAccess` 和 `AnthropicLimitedAccess` 都带有 `Get*` 和 `List*` 通配符，这些通配符授予对工作区中所有内容的读取权限：文件字节、技能内容、批处理结果、会话对话历史和记忆内容。这些通配符还授予 `GetKey` 和 `ListKeys`，用于读取组织已注册的加密密钥配置（密钥 ARN 和元数据，绝不包括密钥材料）。它们还授予账户范围的 `GetUserProfile` 和 `ListUserProfiles`，用于读取用户配置文件。`List*` 通配符还授予 `ListComplianceActivities`，在组织启用 Compliance API 后，可用于读取组织的合规[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)（请参阅[合规](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#compliance)）。它还授予 `ListAccountUsage` 和 `ListAccountCost`，用于读取组织的用量和成本数据（请参阅[用量和成本](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#usage-and-cost)）。这些通配符还授予 `GetAccountTelemetryDestination`、`ListAccountTelemetryDestinations` 和 `ListAccountTelemetryDestinationWorkspaces`，用于读取组织的遥测目标，但绝不会读取其标头值。保管库凭证密钥和 webhook 签名密钥不会被暴露；这些字段是只写的，`GetVault` 或 `GetWebhook` 永远不会返回它们。如果您的主体不应读取现有内容，请使用仅列举您所需操作的自定义策略。
</Note>

`AnthropicLimitedAccess` 除推理操作外，还包含所有 Claude Managed Agents 操作。

`AnthropicSelfHostedEnvironmentAccess` 是足以运行[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)工作进程的最窄托管策略。请将其附加到您的环境工作进程进行身份验证时所使用的主体。

`AssumeConsole` 不包含在 `AnthropicReadOnlyAccess`、`AnthropicInferenceAccess`、`AnthropicLimitedAccess` 或 `AnthropicSelfHostedEnvironmentAccess` 中。需要 Claude Console 访问权限的主体需要 `AnthropicFullAccess` 或授予 `aws-external-anthropic:AssumeConsole` 的自定义策略。请参阅[控制台访问](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#console-access)。

<Note>
  `CreateInference` 和 `CreateBatchInference` 是独立的操作。拒绝其中一个不会阻止另一个。如果您想阻止所有模型调用，请同时拒绝两者。
</Note>

## 示例策略

### 在单个工作区上进行同步推理

为针对一个生产工作区运行推理的 IAM 主体授予最小权限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "aws-external-anthropic:CreateInference",
        "aws-external-anthropic:CountTokens",
        "aws-external-anthropic:GetModel",
        "aws-external-anthropic:ListModels",
        "aws-external-anthropic:GetWorkspace"
      ],
      "Resource": "arn:aws:aws-external-anthropic:us-west-2:123456789012:workspace/wrkspc_01AbCdEf23GhIj"
    }
  ]
}
```

<Note>
  `ListWorkspaces` 是账户范围的（请参阅[预配自动化](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#provisioning-automation)）。如果您的服务账户需要枚举工作区，请为 `ListWorkspaces` 添加一条单独的 `Allow` 语句，并使用 `Resource: "*"`。

  此策略假定使用 AWS SigV4 身份验证。如果主体使用 API 密钥进行身份验证，请为 `aws-external-anthropic:CallWithBearerToken` 添加一条单独的 `Allow` 语句，并使用 `Resource: "*"`。`CallWithBearerToken` 是一个无路由操作，不绑定到工作区 ARN。有关双语句模式，请参阅[按客户隔离工作区](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#per-customer-workspace-isolation)。
</Note>

### 按客户隔离工作区

将角色限制在单个工作区：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "aws-external-anthropic:*",
      "Resource": "arn:aws:aws-external-anthropic:us-west-2:123456789012:workspace/wrkspc_01AbCdEf23GhIj"
    },
    {
      "Effect": "Allow",
      "Action": [
        "aws-external-anthropic:CallWithBearerToken",
        "aws-external-anthropic:AssumeConsole"
      ],
      "Resource": "*"
    }
  ]
}
```

<Note>
  第一条语句中的 `aws-external-anthropic:*` 通配符包含账户范围的操作（`CreateWorkspace`、`ListWorkspaces`、`ListComplianceActivities`、外部密钥操作、账户范围的遥测目标操作、用户配置文件操作以及用量和成本操作），这些操作会被工作区 ARN 约束静默过滤掉。这与"隔离"的意图一致（该角色无法创建工作区、枚举工作区、管理加密密钥注册、读取合规活动源、读取用量或成本报告、读取或管理用户配置文件，也无法创建、读取或更改遥测目标；但它仍然可以通过 `UpdateWorkspace` 将已注册的密钥附加到自己的工作区，并且在给定目标 ID 的情况下，可以将自己的工作区添加到目标或从目标中移除），但该策略包含了不产生任何效果的权限。如果该角色需要用户配置文件或用量和成本报告，请为这些操作添加一条单独的 `Allow` 语句，并使用 `Resource: "*"`。有关账户范围的模式，请参阅[预配自动化](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#provisioning-automation)。

  `CallWithBearerToken` 和 `AssumeConsole` 是无路由操作，不绑定到工作区 ARN。第二条语句在 `Resource: "*"` 上授予它们，以便该角色可以使用 API 密钥进行身份验证并打开 Claude Console。如果该角色仅使用 SigV4 且不需要 Claude Console 访问权限，请省略此语句。
</Note>

### 针对 ZDR 敏感工作区的功能锁定

在特定工作区上阻止批处理和文件上传，同时保留同步推理可用。当工作区处理不得在服务器端持久化的[零数据保留（ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)数据时非常有用。请将此策略与 Allow 策略（例如 `AnthropicInferenceAccess` 或[单工作区示例](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#synchronous-inference-on-a-single-workspace)）一起附加；仅包含 Deny 的策略本身不授予任何权限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "aws-external-anthropic:CreateBatchInference",
        "aws-external-anthropic:CreateFile"
      ],
      "Resource": "arn:aws:aws-external-anthropic:us-west-2:123456789012:workspace/wrkspc_01AbCdEf23GhIj"
    }
  ]
}
```

<Note>
  此拒绝仅阻止创建。其他文件和批处理操作不会被拒绝，除非您也将它们列出。若要实现工作区绝不持有文件或批处理的完全锁定，还需拒绝 `aws-external-anthropic:GetFile`、`aws-external-anthropic:ListFiles`、`aws-external-anthropic:DeleteFile`、`aws-external-anthropic:GetBatchInference`、`aws-external-anthropic:ListBatchInferences`、`aws-external-anthropic:CancelBatchInference` 和 `aws-external-anthropic:DeleteBatchInference`。
</Note>

### 配置自动化

<Note>
  除了 Admin API 之外，您还可以在 AWS 控制台中创建、更新或归档工作区，或者以 Admin 角色在 Claude Console 中执行这些操作。
</Note>

授予 CI/CD 角色创建和管理工作区所需的操作权限，而不包含任何推理权限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "aws-external-anthropic:CreateWorkspace",
        "aws-external-anthropic:GetWorkspace",
        "aws-external-anthropic:ListWorkspaces",
        "aws-external-anthropic:UpdateWorkspace",
        "aws-external-anthropic:ArchiveWorkspace"
      ],
      "Resource": "*"
    }
  ]
}
```

`CreateWorkspace` 和 `ListWorkspaces` 是账户范围的操作。在这些操作上指定工作区 ARN 不会产生任何效果；请使用 `Resource: "*"`。

## 另请参阅

* [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)：了解设置、身份验证和平台概述
* [AWS IAM 用户指南](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)：了解 IAM 策略语法和评估逻辑
* [AWS CloudTrail 用户指南](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)：了解审计日志配置
