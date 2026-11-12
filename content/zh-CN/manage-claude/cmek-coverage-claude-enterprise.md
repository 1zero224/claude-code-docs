---
title: CMEK 在 Claude Enterprise 上涵盖的内容
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise
description: 查看您的密钥加密哪些 Claude Enterprise 数据、哪些功能会关闭或变更，以及哪些数据仍由 Anthropic 的密钥加密。
---

有关 Claude Platform，请参阅 [CMEK 在 Claude Platform 上涵盖的内容](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform)。

## 使用您的密钥加密

* 聊天内容，包括技能和插件。
* 聊天附件和项目附件。
* Artifacts，包括对话和**Artifacts**标签页中的 Claude Design、Claude Slides 和 Claude Docs。
* CLI 上的 Claude Code，包括消息内容。
* Claude Desktop 中的 Cowork。
* 统一 Claude 体验中的聊天，仅限网页端。**Computer access** 处于关闭状态。
* Office agents，包括 Claude for Outlook（beta）。
* Claude in Chrome。
* Claude Science（处于 beta 阶段）。
* [插件 API](https://platform.claude.com/docs/zh-CN/manage-claude/plugins-api#customer-managed-encryption-keys)（beta）插件版本：每个版本的描述、发行说明、组件和文件。该页面列出了在您的密钥无法使用时仍可正常工作的请求。
* 来自用户机器上会话的 Compliance API [本地会话记录](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-local-sessions)。如果您的密钥无法使用，messages 端点将返回 [503 Service Unavailable](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#local-sessions-temporarily-unavailable)，而不是会话记录内容。会话元数据仍会列出。
* 备份和快照，它们继承该密钥。

## 关闭或变更的功能

Beta 和研究预览功能可能不在客户管理的加密密钥（CMEK）的涵盖范围内，并且在 CMEK 组织中可能会出现故障。

此列表并非详尽无遗。在设置 CMEK 之前，请与您的团队一起查看此列表。

| 功能                                                                                                                  | 用户看到的情况                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 网页版 Claude Code                                                                                                     | 关闭。即使管理员开启该产品，也无法启动新会话。                                                                                                                                                       |
| Claude Code Desktop                                                                                                 | 在管理员开启之前处于关闭状态。在 [claude.ai > 组织设置 > Claude Code](https://claude.ai/admin-settings/claude-code) 下开启后，可用于本地会话。                                                                 |
| [Code Review](https://code.claude.com/docs/zh-CN/code-review)                                                       | 关闭。Code Review 不会在 pull request（拉取请求）上运行。                                                                                                                                     |
| Artifacts                                                                                                           | 已变更。对于使用 Claude Code ZDR 的组织：Artifacts（包括对话和**Artifacts**标签页中的 Claude Design、Claude Slides 和 Claude Docs）无法开启。                                                                |
| Artifact 发布和共享                                                                                                      | 已变更。Artifacts 可以在您的组织内共享，但不能公开共享。                                                                                                                                             |
| 分析：技能和连接器                                                                                                           | 已变更。claude.ai 分析仪表板中以及通过 [Claude Enterprise Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api) 提供的技能和连接器管理员分析不包括聊天中的使用情况，并会隐藏自定义技能、插件和连接器的名称。 |
| 回复评分：点赞和点踩                                                                                                          | 关闭。成员无法对 Claude 的回复进行评分。                                                                                                                                                      |
| [技能和插件安全扫描](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/enterprise#skill-content-scanning) | 关闭。该设置无法开启，技能和插件在安装时不经过扫描。                                                                                                                                                    |
| Claude in Slack（beta）                                                                                               | 关闭。即使管理员开启该产品，Claude 也会拒绝请求。                                                                                                                                                  |
| 分析：智能报告和贡献指标                                                                                                        | 关闭。无论管理员设置如何，某些分析功能都会被禁用：Claude 智能报告（位于 claude.ai/analytics/insights 下）以及来自 GitHub 的 Claude Code 贡献指标（beta，位于 claude.ai/analytics/claude-code 下）。                             |
| Claude Security（beta）                                                                                               | 关闭。这是 beta 功能，无论管理员设置如何都会关闭。                                                                                                                                                  |
| 云端定时任务（研究预览）                                                                                                        | 关闭。管理员可以为统一 Claude 体验开启这些任务。                                                                                                                                                  |
| 独立的 Claude Design 应用（正在弃用）                                                                                          | 不受支持。请参阅[使用 Anthropic 的密钥加密](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#encrypted-with-anthropics-key)。                             |

## 使用 Anthropic 的密钥加密

存储这些数据的功能仍然可用，但这些数据不使用您的密钥加密。您可以在**设置**中关闭其中部分功能。

| 数据                         | 详情                                                                                                                                                                                                                                                   |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 密钥设置之前的现有数据                | 在[您的密钥生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)之前写入的数据。                                                                                                                                 |
| 独立的 Claude Design 应用（正在弃用） | 位于 claude.ai/design 的独立 Claude Design 应用不支持 CMEK，其数据可能不使用您的密钥加密。该独立应用即将关闭。对话和**Artifacts**标签页中的 Claude Design 使用您的密钥加密，如[使用您的密钥加密](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#encrypted-with-your-key)中所列。 |
| 个人偏好和 Cowork 全局指令          | [个人偏好 - Claude 指令部分](https://claude.ai/new#settings/account)以及 Cowork 全局指令。这些在账户级别设置，并在用户所属的所有组织之间共享。                                                                                                                                                |
| 您组织中用户的账户数据                | 包括姓名、电子邮件地址和头像。                                                                                                                                                                                                                                      |
| 保存的记录                      | Anthropic 可以[在三种有限的情况下](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#limited-preservation-outside-your-key)保存特定记录。                                                                                                       |
| Claude Code 遥测             | 当 Claude Code 通过 LLM 网关或代理（自定义的 `ANTHROPIC_BASE_URL`）连接时，CMEK 不适用于 Claude Code 的运营遥测数据。要关闭此遥测，请将 `DISABLE_TELEMETRY` 环境变量设置为 `1`，如 Claude Code 文档中的[遥测服务](https://code.claude.com/docs/zh-CN/data-usage#telemetry-services)部分所述。                     |
