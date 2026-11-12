---
title: Claude Models on Google Cloud (Anthropic managed) 的 IAM 权限
url: https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions
description: 查阅 Claude Models on Google Cloud (Anthropic managed) 的 Google Cloud IAM 权限、预定义角色和审计日志类型。
---

Claude Models on Google Cloud (Anthropic managed) 使用 Google Cloud 的 “Identity and Access Management”（身份和访问管理），即 IAM，进行访问控制。每个 API 路由都需要 `claude.googleapis.com` 服务中的一项权限。如果调用方不具备该权限，Google Cloud 会返回 HTTP 403 错误。三个 “predefined role”（预定义角色）将这些权限打包在一起。有关设置和身份验证，请参阅 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)。

## 服务详情

下表描述了 Claude Models on Google Cloud (Anthropic managed) 的 Google Cloud 服务。

| 属性                      | 值                                                                                                                                                                                                              |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **服务**                  | `claude.googleapis.com`                                                                                                                                                                                        |
| **权限检查位置**              | 工作区或 Anthropic 组织                                                                                                                                                                                              |
| **“Audit log”（审计日志）类型** | Data Access（数据访问）、Admin Activity（管理员活动）<br />要开启 Data Access 日志，请参阅[监控和日志记录](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#monitoring-and-logging)。 |

请在 Google Cloud 项目上授予角色。有关项目、组织和工作区之间的关系，请参阅[工作区](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#workspaces)。

本页中的权限采用简写形式 `resource.verb`，例如 `messages.create`。

## 预定义角色

下表列出了 Claude Models on Google Cloud (Anthropic managed) 的预定义角色。

| 角色                    | 显示名称          | 权限                                                                                                                           |
| --------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `roles/claude.viewer` | Claude Viewer | 工作区资源上的所有 `get` 和 `list` 权限。                                                                                                 |
| `roles/claude.user`   | Claude User   | `roles/claude.viewer` 中的所有权限，以及工作区资源上的所有其他权限。<br />例如 `messages.create`、`messages.countTokens` 和 `environments.processWork`。 |
| `roles/claude.admin`  | Claude Admin  | `roles/claude.user` 中的所有权限，以及在您的 Anthropic 组织上检查的权限。<br />例如 `externalKeys.*` 和 `usageAnalytics.list`。                       |

在以下各表中，**Admin**、**User** 和 **Viewer** 分别代表这三个角色。

## 权限

以下各表列出了涵盖 113 个路由的 69 项权限。

`get` 和 `list` 权限是只读权限。

除非某一节另有说明，否则每个路由的审计日志类型均为数据访问。

`roles/claude.viewer` 无法发送消息或计算令牌数。发送消息需要 `messages.create`。计算令牌数需要 `messages.countTokens`。

以下各表中的某些路由可能对您的组织不可用。要查看哪些功能可用，请参阅[功能支持](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#feature-support)。

### 消息

| 权限                     | 角色          | 授权的路由                            |
| ---------------------- | ----------- | -------------------------------- |
| `messages.create`      | Admin, User | `POST /v1/messages`              |
| `messages.countTokens` | Admin, User | `POST /v1/messages/count_tokens` |

### 消息批处理

| 权限                      | 角色                  | 授权的路由                                                                                                    |
| ----------------------- | ------------------- | -------------------------------------------------------------------------------------------------------- |
| `messageBatches.create` | Admin, User         | `POST /v1/messages/batches`                                                                              |
| `messageBatches.get`    | Admin, User, Viewer | `GET /v1/messages/batches/{message_batch_id}`<br />`GET /v1/messages/batches/{message_batch_id}/results` |
| `messageBatches.list`   | Admin, User, Viewer | `GET /v1/messages/batches`                                                                               |
| `messageBatches.delete` | Admin, User         | `DELETE /v1/messages/batches/{message_batch_id}`                                                         |
| `messageBatches.cancel` | Admin, User         | `POST /v1/messages/batches/{message_batch_id}/cancel`                                                    |

`roles/claude.viewer` 可以读取批处理结果。`messageBatches.get` 既授权读取批处理的元数据，也授权下载其结果。

### 模型

| 权限            | 角色                  | 授权的路由                       |
| ------------- | ------------------- | --------------------------- |
| `models.get`  | Admin, User, Viewer | `GET /v1/models/{model_id}` |
| `models.list` | Admin, User, Viewer | `GET /v1/models`            |

### 文件

| 权限             | 角色                  | 授权的路由                                                            |
| -------------- | ------------------- | ---------------------------------------------------------------- |
| `files.create` | Admin, User         | `POST /v1/files`                                                 |
| `files.get`    | Admin, User, Viewer | `GET /v1/files/{file_id}`<br />`GET /v1/files/{file_id}/content` |
| `files.list`   | Admin, User, Viewer | `GET /v1/files`                                                  |
| `files.delete` | Admin, User         | `DELETE /v1/files/{file_id}`                                     |

`roles/claude.viewer` 可以下载文件。`files.get` 既授权读取文件的元数据，也授权下载其内容。

### 技能

| 权限              | 角色                  | 授权的路由                                                                                                                                                                                 |
| --------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `skills.create` | Admin, User         | `POST /v1/skills`                                                                                                                                                                     |
| `skills.get`    | Admin, User, Viewer | `GET /v1/skills/{skill_id}`<br />`GET /v1/skills/{skill_id}/versions`<br />`GET /v1/skills/{skill_id}/versions/{version}`<br />`GET /v1/skills/{skill_id}/versions/{version}/content` |
| `skills.list`   | Admin, User, Viewer | `GET /v1/skills`                                                                                                                                                                      |
| `skills.update` | Admin, User         | `POST /v1/skills/{skill_id}/versions`<br />`DELETE /v1/skills/{skill_id}/versions/{version}`                                                                                          |
| `skills.delete` | Admin, User         | `DELETE /v1/skills/{skill_id}`                                                                                                                                                        |

创建或删除技能版本需要 `skills.update`，而不是 `skills.create` 或 `skills.delete`。

### 代理

| 权限               | 角色                  | 授权的路由                                                                 |
| ---------------- | ------------------- | --------------------------------------------------------------------- |
| `agents.create`  | Admin, User         | `POST /v1/agents`                                                     |
| `agents.get`     | Admin, User, Viewer | `GET /v1/agents/{agent_id}`<br />`GET /v1/agents/{agent_id}/versions` |
| `agents.list`    | Admin, User, Viewer | `GET /v1/agents`                                                      |
| `agents.update`  | Admin, User         | `POST /v1/agents/{agent_id}`                                          |
| `agents.archive` | Admin, User         | `POST /v1/agents/{agent_id}/archive`                                  |

您可以归档代理，但不能删除代理。

### 会话

| 权限                 | 角色                  | 授权的路由                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------ | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sessions.create`  | Admin, User         | `POST /v1/sessions`                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `sessions.get`     | Admin, User, Viewer | `GET /v1/sessions/{session_id}`<br />`GET /v1/sessions/{session_id}/events`<br />`GET /v1/sessions/{session_id}/events/stream`<br />`GET /v1/sessions/{session_id}/resources`<br />`GET /v1/sessions/{session_id}/resources/{resource_id}`<br />`GET /v1/sessions/{session_id}/threads`<br />`GET /v1/sessions/{session_id}/threads/{thread_id}`<br />`GET /v1/sessions/{session_id}/threads/{thread_id}/events`<br />`GET /v1/sessions/{session_id}/threads/{thread_id}/stream` |
| `sessions.list`    | Admin, User, Viewer | `GET /v1/sessions`                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `sessions.update`  | Admin, User         | `POST /v1/sessions/{session_id}`<br />`POST /v1/sessions/{session_id}/events`<br />`POST /v1/sessions/{session_id}/resources`<br />`POST /v1/sessions/{session_id}/resources/{resource_id}`<br />`DELETE /v1/sessions/{session_id}/resources/{resource_id}`                                                                                                                                                                                                                      |
| `sessions.archive` | Admin, User         | `POST /v1/sessions/{session_id}/archive`<br />`POST /v1/sessions/{session_id}/threads/{thread_id}/archive`                                                                                                                                                                                                                                                                                                                                                                       |
| `sessions.delete`  | Admin, User         | `DELETE /v1/sessions/{session_id}`                                                                                                                                                                                                                                                                                                                                                                                                                                               |

`sessions.get` 授权读取会话、其事件历史和事件流、其资源以及其线程。因此，`roles/claude.viewer` 可以读取每个会话的完整事件历史。

其他会话操作需要以下权限：

* 发送事件：`sessions.update`
* 添加、更改或移除会话资源：`sessions.update`
* 归档线程：`sessions.archive`

### 环境

| 权限                         | 角色                  | 授权的路由                                                                                                                                                                                                                                                                                                                    |
| -------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `environments.create`      | Admin, User         | `POST /v1/environments`                                                                                                                                                                                                                                                                                                  |
| `environments.get`         | Admin, User, Viewer | `GET /v1/environments/{environment_id}`<br />`GET /v1/environments/{environment_id}/work`<br />`GET /v1/environments/{environment_id}/work/stats`<br />`GET /v1/environments/{environment_id}/work/{work_id}`                                                                                                            |
| `environments.list`        | Admin, User, Viewer | `GET /v1/environments`                                                                                                                                                                                                                                                                                                   |
| `environments.update`      | Admin, User         | `POST /v1/environments/{environment_id}`                                                                                                                                                                                                                                                                                 |
| `environments.archive`     | Admin, User         | `POST /v1/environments/{environment_id}/archive`                                                                                                                                                                                                                                                                         |
| `environments.delete`      | Admin, User         | `DELETE /v1/environments/{environment_id}`                                                                                                                                                                                                                                                                               |
| `environments.processWork` | Admin, User         | `GET /v1/environments/{environment_id}/work/poll`<br />`POST /v1/environments/{environment_id}/work/{work_id}`<br />`POST /v1/environments/{environment_id}/work/{work_id}/ack`<br />`POST /v1/environments/{environment_id}/work/{work_id}/heartbeat`<br />`POST /v1/environments/{environment_id}/work/{work_id}/stop` |

`roles/claude.user` 和 `roles/claude.admin` 包含[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)工作器所需的全部权限。`environments.processWork` 授权以下工作器操作：

* 轮询工作
* 确认工作项
* 为工作项发送心跳
* 停止工作项
* 更新工作项

此权限独立于 `environments.update`。读取工作队列及其统计信息只需要 `environments.get`。

除 `environments.processWork` 之外，工作进程还需要其他权限才能运行会话，例如用于读取会话及其事件的 `sessions.get`、用于发送工具结果的 `sessions.update`，以及用于下载智能体技能的 `skills.get`。

### 保管库

Google Cloud 会将这些路由记录在管理员活动审计日志中，但 `vaults.verify` 的路由除外，其审计日志类型为数据访问。

| 权限               | 角色                  | 授权的路由                                                                                                                                                                                                    |
| ---------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `vaults.create`  | Admin, User         | `POST /v1/vaults`                                                                                                                                                                                        |
| `vaults.get`     | Admin, User, Viewer | `GET /v1/vaults/{vault_id}`<br />`GET /v1/vaults/{vault_id}/credentials`<br />`GET /v1/vaults/{vault_id}/credentials/{credential_id}`                                                                    |
| `vaults.list`    | Admin, User, Viewer | `GET /v1/vaults`                                                                                                                                                                                         |
| `vaults.update`  | Admin, User         | `POST /v1/vaults/{vault_id}`<br />`POST /v1/vaults/{vault_id}/credentials`<br />`POST /v1/vaults/{vault_id}/credentials/{credential_id}`<br />`DELETE /v1/vaults/{vault_id}/credentials/{credential_id}` |
| `vaults.archive` | Admin, User         | `POST /v1/vaults/{vault_id}/archive`<br />`POST /v1/vaults/{vault_id}/credentials/{credential_id}/archive`                                                                                               |
| `vaults.delete`  | Admin, User         | `DELETE /v1/vaults/{vault_id}`                                                                                                                                                                           |
| `vaults.verify`  | Admin, User         | `POST /v1/vaults/{vault_id}/credentials/{credential_id}/mcp_oauth_validate`                                                                                                                              |

凭据操作需要以下权限：

* 创建、更改或删除凭据：`vaults.update`
* 归档凭据：`vaults.archive`
* 读取凭据：`vaults.get`
* 检查已存储的 MCP OAuth 凭据：`vaults.verify`

借助 `vaults.get`，`roles/claude.viewer` 可以列出保管库的凭据。Claude Models on Google Cloud (Anthropic managed) 从不返回凭据的机密字段。要了解更多信息，请参阅[使用保管库进行身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)。

### 记忆存储

| 权限                     | 角色                  | 授权的路由                                                                                                                                                                                                                                                                                                                                      |
| ---------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `memoryStores.create`  | Admin, User         | `POST /v1/memory_stores`                                                                                                                                                                                                                                                                                                                   |
| `memoryStores.get`     | Admin, User, Viewer | `GET /v1/memory_stores/{memory_store_id}`<br />`GET /v1/memory_stores/{memory_store_id}/memories`<br />`GET /v1/memory_stores/{memory_store_id}/memories/{memory_id}`<br />`GET /v1/memory_stores/{memory_store_id}/memory_versions`<br />`GET /v1/memory_stores/{memory_store_id}/memory_versions/{memory_version_id}`                    |
| `memoryStores.list`    | Admin, User, Viewer | `GET /v1/memory_stores`                                                                                                                                                                                                                                                                                                                    |
| `memoryStores.update`  | Admin, User         | `POST /v1/memory_stores/{memory_store_id}`<br />`POST /v1/memory_stores/{memory_store_id}/memories`<br />`POST /v1/memory_stores/{memory_store_id}/memories/{memory_id}`<br />`DELETE /v1/memory_stores/{memory_store_id}/memories/{memory_id}`<br />`POST /v1/memory_stores/{memory_store_id}/memory_versions/{memory_version_id}/redact` |
| `memoryStores.archive` | Admin, User         | `POST /v1/memory_stores/{memory_store_id}/archive`                                                                                                                                                                                                                                                                                         |
| `memoryStores.delete`  | Admin, User         | `DELETE /v1/memory_stores/{memory_store_id}`                                                                                                                                                                                                                                                                                               |

记忆存储操作需要以下权限：

* 读取存储、其记忆或其版本历史：`memoryStores.get`
* 创建、更改或删除记忆：`memoryStores.update`
* 编辑（redact）记忆版本：`memoryStores.update`

### 部署

| 权限                    | 角色                  | 授权的路由                                                                                                                   |
| --------------------- | ------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `deployments.create`  | Admin, User         | `POST /v1/deployments`                                                                                                  |
| `deployments.get`     | Admin, User, Viewer | `GET /v1/deployment_runs`<br />`GET /v1/deployment_runs/{deployment_run_id}`<br />`GET /v1/deployments/{deployment_id}` |
| `deployments.list`    | Admin, User, Viewer | `GET /v1/deployments`                                                                                                   |
| `deployments.update`  | Admin, User         | `POST /v1/deployments/{deployment_id}`                                                                                  |
| `deployments.archive` | Admin, User         | `POST /v1/deployments/{deployment_id}/archive`                                                                          |
| `deployments.run`     | Admin, User         | `POST /v1/deployments/{deployment_id}/run`                                                                              |
| `deployments.pause`   | Admin, User         | `POST /v1/deployments/{deployment_id}/pause`                                                                            |
| `deployments.unpause` | Admin, User         | `POST /v1/deployments/{deployment_id}/unpause`                                                                          |

读取部署运行需要 `deployments.get`。

### 梦境

| 权限               | 角色                  | 授权的路由                                |
| ---------------- | ------------------- | ------------------------------------ |
| `dreams.create`  | Admin, User         | `POST /v1/dreams`                    |
| `dreams.get`     | Admin, User, Viewer | `GET /v1/dreams/{dream_id}`          |
| `dreams.list`    | Admin, User, Viewer | `GET /v1/dreams`                     |
| `dreams.archive` | Admin, User         | `POST /v1/dreams/{dream_id}/archive` |
| `dreams.cancel`  | Admin, User         | `POST /v1/dreams/{dream_id}/cancel`  |

Dreams 是一项研究预览功能。要申请访问权限，请参阅[梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)。

### 加密密钥

Google Cloud 会将这些路由记录在管理员活动审计日志中。

| 权限                    | 角色    | 授权的路由                                                      |
| --------------------- | ----- | ---------------------------------------------------------- |
| `externalKeys.create` | Admin | `POST /v1/organizations/external_keys`                     |
| `externalKeys.get`    | Admin | `GET /v1/organizations/external_keys/{external_key_id}`    |
| `externalKeys.list`   | Admin | `GET /v1/organizations/external_keys`                      |
| `externalKeys.update` | Admin | `POST /v1/organizations/external_keys/{external_key_id}`   |
| `externalKeys.delete` | Admin | `DELETE /v1/organizations/external_keys/{external_key_id}` |

这些权限用于管理您组织的加密密钥注册。在三个预定义角色中，只有 `roles/claude.admin` 包含这些权限。

### 使用情况报告

| 权限                    | 角色    | 授权的路由                                                                                      |
| --------------------- | ----- | ------------------------------------------------------------------------------------------ |
| `usageAnalytics.list` | Admin | `GET /v1/organizations/usage_report/messages`<br />`GET /v1/organizations/analytics/usage` |
| `costAnalytics.list`  | Admin | `GET /v1/organizations/analytics/cost`                                                     |

在三个预定义角色中，只有 `roles/claude.admin` 包含 `usageAnalytics.list` 和 `costAnalytics.list`。

## 另请参阅

* [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)
* [IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)
* [监控和日志记录](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#monitoring-and-logging)
